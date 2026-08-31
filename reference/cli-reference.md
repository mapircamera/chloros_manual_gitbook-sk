# Referenčná príručka k príkazu „Chloros“ (CLI)

**Verzia:**

1.2.0**Vytvorené:**

29. 7. 2026 o 19:19 ·**Revidované:**

30. 8. 2026**Cieľová skupina:** Optimalizované pre využitie veľkými jazykovými modelmi (LLM); zrozumiteľné pre ľudí.**Rozsah:** Všetky podpríkazy nástroja `chloros-cli` určené pre používateľov, vrátane volieb a príkladov, ktoré je možné skopírovať a vložiť.

Tento dokument je kompletnou referenciou pre nástroj príkazového riadku `chloros-cli`, ktorý je súčasťou balíka MAPIR Chloros. Je zámerne vyčerpávajúci, aby LLM (alebo človek) mohol zostaviť akýkoľvek podporovaný pracovný postup z nižšie uvedených zoznamov bez nutnosti prezerať zdrojový kód.

Ak potrebujete len to najdôležitejšie, prejdite na:
- [Päťminútový rýchly štart](#päťminútový-rýchly-začiatok)
- [Pracovný postup prvého pripojenia kamery LATTICE](#lattice-camera-first-connect-workflow)
- [Pracovný postup prvého pripojenia snímača DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Režimy snímania, záznamníky a offline spracovanie](#capture-modes-recorders--offline-reprocess)

---

## Konvencie

- Všetky príkazy začínajú predponou `chloros-cli`. Na zariadení Windows je binárny súbor `chloros-cli.exe`; na zariadení Linux /Jetson je to `chloros-cli`.
- Voliteľné argumenty sú zobrazené ako `--flag`. Povinné pozičné argumenty sú uvedené bez zátvoriek.
- Ak je uvedená predvolená hodnota, vynechaním príznaku sa použije táto hodnota.
- CLI je tenký klient typu „HTTP“ nad backendom Chloros (server Flask na `127.0.0.1:5000`). Backend sa automaticky spúšťa väčšinou príkazov. `CHLOROS_BACKEND_URL=<url>` smeruje na rodiny príkazov **`lattice`**,**`project`**a**`daq pool-*`** na vzdialený backend — základné príkazy (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) zámerne fixujú `http://127.0.0.1:<port>` a ignorujú ho (literál IPv4 zabraňuje penalizácii ~Windows s na požiadavku pre `localhost`→`::1` ~2 s penalizácie na žiadosť). Pozri [Premenné prostredia](#environment-variables).
- Pre všetky volania SDK / CLI je potrebné prihlásenie sa pod účtom Chloros+ (spusti `chloros-cli login` raz na každom počítači; uložené v pamäti cache v `~/.chloros/`).
- Príklady používajú cesty Linux; na Windows nahraďte `/home/user/...` za `C:/Users/.../...`.

---

## Prehľad na najvyššej úrovni

```
chloros-cli [global options] COMMAND [command options]
```

### Globálne možnosti

| Príznak | Popis |
| --- | --- |
| `--backend-exe PATH` | Prepísanie automaticky zisteného spustiteľného súboru backendu. |
| `--port N` | Port backendu HTTP (predvolené: `5000`). |
| `-v, --verbose` | Zapnúť podrobný výstup. |
| `--restart` | Vynútiť reštart backendu (ukončí všetky spustené procesy `backend_server.py`). |
| `--version` | Vypísať verziu (`Chloros CLI 1.2.0`). |
| `--help` | Zobraziť pomoc najvyššej úrovne. |

### Zoznam príkazov

| Príkaz | Účel |
| --- | --- |
| [`process`](#chloros-cli-process) | Spracovať priečinok so záznamami typu „Survey3“ alebo „LATTICE“ od začiatku do konca. |
| [`login`](#chloros-cli-login) | Overiť toto zariadenie pomocou účtu Chloros+. |
| [`logout`](#chloros-cli-logout) | Vymazať uložené prihlasovacie údaje z vyrovnávacej pamäte. |
| [`status`](#chloros-cli-status) | Zobraziť aktuálny stav licencie / overenia. |
| [`export-status`](#chloros-cli-export-status) | Zobraziť priebeh exportu Live Thread-4 počas behu `process`. |
| [`language`](#chloros-cli-language) | Nastavenie alebo zobrazenie zoznamu jazykov zobrazenia vCLIi (podporovaných je 38). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Predvolaná zložka projektu (zdieľaná s grafickým rozhraním). |
| [`update`](#chloros-cli-update) | Vyhľadať a nainštalovať aktualizácie CLI (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnostika systému + základné testy. |
| [`time-sync`](#chloros-cli-time-sync) | Stav a ovládanie PTP grandmaster. |
| [`lattice`](#chloros-cli-lattice) | Ovládanie a snímanie kamery LATTICE (viac ako 45 podpríkazov). |
| [`daq`](#chloros-cli-daq) | Ovládanie spektrálnych senzorov DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Otvorenie a spustenie uloženého projektu Chloros (kamery + DAQ). |

---

## Inštalácia

`chloros-cli` je súčasťou inštalačného balíka pre stolné počítače Chloros na všetkých podporovaných platformách — nie je k dispozícii samostatné stiahnutie CLI. Inštaláciou balíka pre danú platformu sa `chloros-cli` pridá do vášho `PATH` spolu s aplikáciou pre stolné počítače a binárnym súborom backendu, ktorý táto aplikácia ovláda.

Najnovšie súbory na stiahnutie: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Inštalačný program obsahuje aj skripty na pohodlné spúšťanie (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), ktoré otvárajú pripravené prostredie CLI; sú popísané v [CLI User Guide](../CLI.md) a nebudú tu opakované.

### Windows (.exe)

1. Stiahnite si inštalačný program Windows zo stránky na stiahnutie.
2. Spustite súbor `Chloros-Setup-x.y.z.exe` a postupujte podľa pokynov sprievodcu. Predvolená inštalačná cesta je `C:\Program Files\Chloros\` (CLIa sa nachádza v adresári `C:\Program Files\Chloros\cli\`, ktorý inštalačný program pridá do premennej PATH).
3. Otvorte nový terminál (`cmd.exe`, PowerShell alebo terminál systému Windows), aby sa načíta aktualizovaný súbor `PATH`.

```powershell
chloros-cli --version
```

Inštalačný program automaticky pridá súbor `chloros-cli.exe` do vášho systémového prostredia `PATH` a zahrnie do balíka runtime prostredie Arena SDK potrebné pre kamery LATTICE.

### Linux amd64 (.deb)

Pre pracovné stanice s Ubuntu 22.04 LTS alebo novším / založené na Debiane s architektúrou x86_64.

> **Ubuntu 20.04 nie je podporované.** Zoznam závislostí balíka je odvodený od
> toho, na čo sa backend skutočne odkazuje, a to zahŕňa `libc6 (>= 2.34)`;
> spoločnosť Focal dodáva glibc 2.31. `apt` odmietne inštaláciu, namiesto toho, aby ju nechal zlyhať počas
> behu programu.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

Balík .deb nainštaluje:
- `chloros-cli` až po `/usr/bin/chloros-cli`
- Kompilovaný backend na verziu `/usr/lib/chloros/chloros-backend`
- Runtime pre Arena SDK (pre kamery LATTICE)
- Modely odšumovača, kalibračné balíky a konfigurácia kanála aktualizácií

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Rovnaká štruktúra ako pri balíku .deb pre amd64, s verziou CUDA vyladenou pre Jetson Orin / Orin NX / Orin Nano.

### Jednorazová autentifikácia na každom zariadení

Každá platforma vyžaduje jednorazové prihlásenie na Chloros+ predtým, ako budú fungovať volania SDK / CLI:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Prihlasovacie údaje sa ukladajú do vyrovnávacej pamäte v súbore `~/.chloros/user_session.json`.

### Overenie inštalácie

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Je potrebné predplatné Chloros+.**Služba CLI vyžaduje aktívny plán Chloros+.**Copper**je základná úroveň Chloros+ — každá platená úroveň Chloros+ má prístup k CLI / SDK; iba bezplatná úroveň**Iron** ho nemá. (Mapovanie ID plánov: `0`=Iron/free, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Upgradujte na [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Táto minimálna hranica je vynucovaná na strane backendu, nielen prostredníctvom CLI: žiadosť s príznakmi SDK / CLI bez plateného plánu je odmietnutá s kódom `403 PLAN_UPGRADE_REQUIRED`, bez ohľadu na to, či pochádza z `chloros-cli`, Python SDK, alebo z vlastnoručne vytvoreného klienta HTTP. Odhlásený volajúci dostane namiesto toho kód `401 AUTH_REQUIRED`. Prístup funguje offline počas ochranného obdobia plánu (30 dní mesačne, v prípade ročného predplatného do konca platnosti) a po jej uplynutí sa zastaví; kód `chloros-cli status` naďalej funguje, aby bol dôvod viditeľný (ide o jedinú trasu SDK / CLI, ktorá je vyňatá z obmedzenia podľa úrovne — `GET /api/license-status`).

---

## Päťminútový rýchly štart

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Spracujte priečinok s obrázkami prostredníctvom kompletného spracovateľského reťazca Chloros (detekcia cieľa → kalibrácia → vineta → odrazivosť → export indexu).

### Prehľad

```
chloros-cli process INPUT [OPTIONS]
```

### Polohové argumenty

| Argument | Popis |
| --- | --- |
| `INPUT` | Cesta k vstupnému priečinku obsahujúcemu súbory `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) alebo `.dng`. |

### Bežné možnosti

| Príznak | Predvolené nastavenie | Popis |
| --- | --- | --- |
| `-o, --output PATH` | nový priečinok s časovou pečiatkou v predvolenej ceste projektu (`~/Chloros Projects`, ak nie je nakonfigurované inak) | Priečinok projektu, ktorý sa má vytvoriť alebo znovu použiť. Ak priečinok už obsahuje súbor `project.json`, namiesto prepísania sa vytvorí súrodenecký priečinok `_1`/`_2`. |
| `-n, --project-name NAME` | auto (časová pečiatka) | Názov projektu. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` používa neurálny debayer Chloros+; pomalší, ale s vyššou kvalitou. |
| `--vignette / --no-vignette` | `--vignette` | Korekcia vinety. |
| `--reflectance / --no-reflectance` | `--reflectance` | Kalibrácia odrazivosti (používa cieľový panel, ak je k dispozícii, alebo sériovú kalibráciu NIST pre LATTICE). V prípade multispektrálneho snímača LATTICE slúži zároveň ako prepínač **produktu** odrazivosti prepínač — pozri [Prepínače exportu podľa produktu](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | vypnuté | Použiť korekcie PPK GNSS zo súborov sidecar. |
| `--exposure-pin-1 MODEL` | vypnuté | Upevniť model „pin-1“ dvojkamery typu „Survey3“. |
| `--exposure-pin-2 MODEL` | vypnuté | Upevniť model „pin-2“. |
| `--recal-interval SECONDS` | 0 | Vynútiť opätovné spustenie kalibračných výpočtov každých N sekúnd počas snímkovania. |
| `--timezone-offset HOURS` | lokálne | Prepísať časové posunutie zakódované vo výstupných metadátach. |
| `--format FORMAT` | `TIFF (16-bit)` | Jeden z modelov `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | žiadne | Vegetácia indexy (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Vynútiť vstupný bod potrubia pre súbory LATTICE TIFF (súbory Survey3 .raw to neovplyvní). Takisto únikový mechanizmus, ktorý umožňuje, aby sa zachytenie **bez surových dát** vôbec spracovať — pozri [Ako vyzerá priečinok so záznamami](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | zapnuté | Vygenerovať lineárny produkt po odstránení bayera (`Debayered_Images`). Pozri [Prepínače exportu pre jednotlivé produkty](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | zapnuté | Vysiela náhľad na displeji (`Preview_Images`): RGB = vyváženie bielej (svetelný zdroj DAQ, ak je k dispozícii, inak šedý svet) + gama; multispec = rozťahovanie falošných farieb. |
| `--radiance / --no-radiance` | zapnuté | Vysiela žiarivosť typu float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referencia pre produkt odrazivosti LATTICE: `auto` = cieľ v rámci snímky, ktorý prešiel kontrolou kvality (QA), je absolútnym referenčným bodom, záložná hodnota pre DAQ smerujúce nadol (ρ = π·L/E); `target` = prísne (bez nahradenia DAQ); `daq` = autoritatívne podľa DAQ. Pozri [Prepínače exportu pre jednotlivé produkty](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | žiadne | Adresár skenov **nameraných** odrazivostí cieľa na jednotku (`<serial>.csv`); v prípade chýbajúcich údajov sa použijú nominálne spektrá T3/T4P. |
| `--array-alignment / --no-array-alignment` | zapnuté | LATTICE polia: na každý spracovaný produkt sa uplatní zarovnanie medzi modulmi, ktoré je zaznamenané v súbore XMP každého záznamu `Chloros:Alignment*` (odstránenie matice / náhľad / žiarivosť / odrazivosť / index). Žiadna operácia pre snímky bez týchto značiek. |
| `--array-alignment-crop / --no-array-alignment-crop` | orezanie | Orezanie zarovnaných výstupov na oblasť spoločného prekrývania poľa tak, aby všetky moduly zdieľali jednu stopu; `--no-…` zachováva plnú plochu snímača (čierne vyplnenie mimo zdrojového obrazu). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Prepočítanie vzorky pre deformáciu pri zarovnaní. `nearest` zachováva presné zdrojové hodnoty DN (bez zmiešavania radiometrických hodnôt medzi pixelmi). |

### Možnosti detekcie cieľa

| Priepínač | Popis |
| --- | --- |
| `--min-target-size PIXELS` | Minimálna veľkosť cieľa na paneli (px) pre detektor. |
| `--target-clustering 0-100` | Citlivosť zhlukovania. |
| `--target / --targets` | Spracovať vstupný priečinok ako obsahujúci iba cieľové panely (preskočiť detekciu prieskumu). |

### Príklady

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Prepínače exportu pre jednotlivé produkty (multispektrálny LATTICE)

Spracovanie LATTICE sa rozvetvuje na **všetky príslušné produkty v jednom cykle**. Štyri prepínače pre jednotlivé typy — `--debayered`, `--preview`, `--radiance`, `--reflectance` — sú všetky**štandardne zapnuté**; na vypnutie jedného z nich použite formulár `--no-<type>`. Hlavné kamery RGB vysielajú vždy len debayered + náhľad (žiadna žiarivosť/odrazivosť podľa pásma), takže `--radiance`/`--reflectance` pre ne nie súpre ne žiadne operácie. Prepínače sa ignorujú pre Survey3 `.raw` (ktorý sleduje štandardnú cestu odrazivosti/cieľa). *(Starý príznak `--radiometric-output {reflectance,radiance,sensor-response}` bol **odstránený** a nahradený týmito prepínačmi; už neexistuje úroveň `sensor-response`.)*

| Produkt | Výstup | Je potrebné zber údajov smerom nadol? |
| --- | --- | --- |
| `--debayered` | Lineárna demosaika (`Debayered_Images`). | Nie. |
| `--preview` | Náhľad zobrazenia (`Preview_Images`): RGB = WB + gama; multispec = rozloženie falošných farieb. | Nie. |
| `--radiance` | float32 W/m²/sr/nm z úplného radiometrického reťazca (`Radiance_Images`). | Č. |
| `--reflectance` | uint16 odrazivosť ρ (`32768` = 1,0), pripravené pre Pix4D. | **Áno**, pokiaľ ho neukotví cieľ v rámci snímky prechádzajúci kontrolou kvalityv rámci snímky, ktorý spĺňa požiadavky QA, ho ukotví (pozri nižšie). |

`--reflectance-source` vyberá referenciu odrazivosti:**`auto`**(predvolené) stanoví cieľ v rámci snímky, ktorý spĺňa požiadavky QA, za**absolútnu referenciu**— reťazceukotvené reťazce empirických čiar sú krížovo porovnávané na vyňatých paneloch a uplatňuje sa nameraný víťaz — s prechodom na DAQ delenie smerom nadol (ρ = π·L/E), ak nie je prítomný žiadny cieľ alebo zlyhá kontrola kvality;**`target`**je prísny (bez nahradenia DAQ);**`daq`**uprednostňuje správanie riadené DAQ. Geometria cieľa (ArUco / pevná oblasť záujmu / pás) pochádza z konfigurácie cieľa projektu; `--target-reflectance-dir DIR` uchováva**zmerané** skeny na jednotku (`<serial>.csv`), ktoré sa vyhľadávajú podľa sériového čísla/QR kódu cieľovej jednotkysériovým číslom/QR kódom cieľovej jednotky, pričom ako záložné riešenie slúžia nominálne spektrá T3/T4P.

Cesta odrazivosti DAQ automaticky vyrieši **zhodu časovej pečiatky s dopadajúcim žiarením**na základe zaznamenaného**`.daq`**(DAQ-U/M/E)**alebo natívneho súboru DAQ-M typu `.csv`**, ktorý sa nachádza spolu so snímkami. Ak nie je kalibračný balík pre konkrétnu kameru alebo DAQ lokálne uložený v cache, spracovateľský reťazec**ho automaticky stiahne z AWS** pri prvom použití (vyžaduje jednorazové pripojenie k internetu; uloží sa do vyrovnávacej pamäte pod názvom `~/.chloros/`).

#### Čítanie pixelov odrazivosti (Pix4D / Metashape / vaše vlastné skripty)

Odrazivosť je uložená ako celé číslo DN a **hodnota DN, ktorá zodpovedá ρ = 1,0, závisí od zdrojovej kamery**:

| Zdroj | ρ = 1,0 je | Ako zistiť |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (rozsah až po ρ 2,0) | V súbore je označený XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (orezané pri ρ 1,0) | Žiadne značky `Chloros:*` XMP — táto absencia *je* signálom. |

**Prečítajte si hodnotu `Chloros:PixelScale` a vydelte ňou** namiesto toho, aby ste predpokladali konštantu. Značka je definovaná v doméne uint16, takže zostáva `32768` vo všetkých výstupných formátoch, ktoré menia mierku — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` a `TIFF (32-bit, Percent)` sú všetky samopopisné (najprv normalizujú uložený dátový typ späť na uint16: ×257 z 8-bitového, ×65535 z float).

> **Jeden prípad podľa návrhu neobsahuje žiadne mierky.** Keď sa záznam z 8-bitového zdroja (BayerRG8) zapíše ako 8-bitový TIFF, potrubie *oreže* na rozsah 0..255 namiesto preškálovania, takže každá hodnota nad ρ≈0,008 sa zjednotí na 255 a súbor nemá žiadne škárovanie. Chloros zámerne vynecháva ako `Chloros:PixelScale`, tak aj `MicaSense:RadiometricCalibration` a zaznamenáva dôvod. **Ak v súbore odrazivosti LATTICE chýba táto značka, nepredpokladajte žiadne mierky – znovu exportujte v 16-bitovom alebo 32-bitovom formáte** namiesto delenia pixelov, ktoré nikdy neboli deliteľné.

#### EXIF prenesené do exportu

`process` skopíruje **blok GPS a jeho ExifIFD** zo zdrojového záznamu do každého produktu, takže
export obsahuje `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` a
`CameraSerialNumber` spolu s georeferencovaním.

**`FocalLength` nie je voliteľný pre fotogrametriu.** Pix4D vypočíta vzdialenosť vzorky na zemi na základe
ohniskovej vzdialenosti a nadmorskej výšky; ak táto značka chýba, program použije nesprávne mierku. Pri jednom
lete nad pomarančovým hájom s 49 zábermi chýbajúca značka premenila lokalitu s rozmermi 411 m × 160 m na rekonštruovanú
plochu s rozmermi 47,8 km × 13 km — ortofotomapu s rozlíšením 455 MP, ktorá obsahovala prevažne údaje typu „nodata“, čo sa následne interpretovalo ako problém s dlaždicovaním a
problém s formátom BigTIFF, kým niekto skontroloval GSD. Ak vaša ortofotomapa vyjde v nepravdepodobnom
mierke, najprv spustite `exiftool -FocalLength` na exportovaný produkt.

Kópia zámerne **nie je** `-all:all`: štrukturálne tagy IFD0 narušujú výstup LATTICE pri
kopírovaní a `ExifImageWidth` / `ExifImageHeight` sú vylúčené, pretože popisujú *zdrojové*
zachytenie — export, ktorého rozmery boli kedykoľvek zmenené, by inak obsahoval rozmery
v rozpore s vlastným rastrom. XMP sa zapisuje priamo namiesto kopírovania, pretože ExifTool
pri kopírovaní bloku XMP vyhadzuje XMP značky z rovnakého volania (čo by viedlo k strate kalibračných značiek MAPIR
).

### Kam sa ukladajú výstupy

Výstupy sa ukladajú **do projektovej zložky, zoskupené podľa fotoaparátu a následne podľa formátu súboru**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Zložka fotoaparátu je `LATT-<sensor>-<lens>-F<filter>` pre LATTICE (zodpovedá zachyteniuEXIF
`Model`) a `<model>_<filter>` pre Survey3 — dve kamery, ktoré zdieľajú senzor a filter, ale líšia sa
objektívom, majú oddelené stromy, pretože sa líšia vinetáciou, zorným poľom a skreslením. Formát
zložky je nasledovný: `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` alebo `tiff32` pre
`TIFF (32-bit, Percent)`.

> **Každý exportovaný produkt si zachováva názov zdrojového súboru.** Export radiancie
> `capture_…_raw.tif` sa stále nazýva `capture_…_raw.tif` — len sa nachádza v
> `tiff32/Radiance_Images/`. **Produkt identifikuje zložka, nie názov súboru**, takže globálny
> na `*radiance*.tif` nič nenájde; namiesto toho použite zhodu s adresárom.

### Záznamy zo svetelného senzora — kalibrované `.daq` + `.csv`

`process` spracováva aj záznamy `.daq` vo vašom vstupnej zložke a na to **nepotrebuje**
žiadne snímky: samostatne letiaci DAQ-U / DAQ-M / DAQ-E predstavuje kompletný
záznam a zložka obsahujúca iba súbory `.daq` je platným vstupom.

Záznam z DAQ je možné vykonať **bez** jeho kalibrácie — to je to, čo verejné
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) záznamníky
(`record_daq.py`) robia štandardne: zapisujú surové hodnoty zo senzorov a označujú súbor tak, aby
Chloros načítalo továrenskú kalibráciu daného senzora **cez sériový port** (najprv z lokálnej cache,
potom z cloudu MAPIR) a aplikovalo ju. `process` zapíše výsledok späť:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` obsahuje jeden riadok na jedno meranie: časovú pečiatku UTC, integračný čas, celkový výkon,
fotopický/skotopický lux, PPFD (a jeho rozdelenie na modrú/zelenú/červenú), špičkovú vlnovú dĺžku, potom
celé spektrum na vlastnej vlnovej mriežke senzora. `.daq` sa znovu importuje bez toho, aby bol
kalibrovaný druhýkrát.

V prípade úspechu hlásenie behu vygeneruje `Light-sensor products written: N (calibrated .daq + .csv)`.
Text v zátvorkách popisuje, čo bolo skutočne zapísané, takže znie
`(RAW COUNTS — this sensor has no calibration bundle)` pre senzor bez zväzku a
`(N calibrated, M raw counts)` pre priečinok obsahujúci oboje. Vlastné nadpisy backendu
`[DAQ-EXPORT]` a `[RUN-SUMMARY]` sú odvodené rovnakým spôsobom – žiadny z
týchto troch nemôže označiť surový export za kalibrovaný.

Záznam DAQ-U / DAQ-M / DAQ-E, ktorého kalibračný balík nie je možné načítať – ste
offline alebo daný senzor nemá v súbore žiadnu kalibráciu – je **preskočený s uvedením dôvodu** v riadku
`[DAQ-EXPORT]`, nikdy sa nezapíše ako „kalibrovaný“ súbor obsahujúci surové počty.
Pripojte sa k internetu a spustite proces znovu. Dôvodom je ten, ktorý čítač skutočne
zistil pre daný súbor (nečitateľná schéma, chýbajúci balík, chyba zápisu), a v súhrne
behu sú uvedené **odlišné** dôvody – dvadsať preskočených súborov z jedného dôvodu sa vyhodnotí ako jeden
príčinu, nie ako dvadsať opakovaní tej istej chyby.

#### Export záznamov DAQ-A ako surové počty

Rodina **DAQ-A** predchádza systému zväzkov podľa sériového čísla a nemá žiadny kalibračný zväzok,
ktorý by sa dal načítať – namiesto toho sa kalibruje v teréne pomocou cieľa odrazivosti, čo je
dôvod, prečo ho nikdy nepotrebovala. Odmietnutie týchto záznamov im znemožnilo získať ich
čísla vôbec získať, preto ich exportujú pod **iným názvom**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Ide o iný názov súboru namiesto príznaku vnútri súboru, pretože údaj musí prežiť
odoslanie e-mailom ako holý názov. Hlavička súboru `.csv` uvádza
`raw spectral sensor counts (NOT irradiance)` a upozorňuje, že hodnoty sú porovnateľné
**v rámci** súboru – čo je presne to, na čo ich využíva kalibrácia založená na cieľoch – a
nie medzi senzormi. Fotometrické stĺpce závislé od výkonu (celkový výkon, fotopický a
skotopický lux, PPFD) sú zapísané ako **NULL** namiesto toho, aby boli integrované z počtov, a v súhrne
behu je uvedené `RAW COUNTS`, takže údaje „exportované“ do protokolu nemožno interpretovať ako ožiarenie.

Staršie **v1.01 / v1.02** (zapisuje ich DAQ-A-SD) neobsahujú epochu pre každé meranie,
iba čas zápisu súboru. Porovnávač obrazu↔svetelného toku ich stále odmieta — porovnanie
s časom zápisu by bolo nesprávne a nebolo by to viditeľné — ale exportér ich číta a
CSV vytlačí `clock=daq_created_on`, takže produkt uvádza, na akých hodinách beží.

### Poznámky

- `process` automaticky zistí, či je vaša zložka typu „Survey3“, „LATTICE“ alebo zmiešaná.
- Priebeh spracovania sa prenáša prostredníctvom udalostí Server-Sent Events; stránka CLI zobrazuje priebeh spracovania v reálnom čase pre jednotlivé vlákna (Detekcia, Analýza, Spracovanie, Export).
- V prípade zariadení Linux /Jetson nástroj CLI kontroluje výmennú pamäť a môže zobraziť varovanie pred spracovaním veľkých zložiek. Funkcia debayeringu zohľadňujúca textúry tiež automaticky uplatňuje obmedzenie frekvencie GPU na zariadeniach Jetson s nízkou spotrebou energie (Nano, Orin Nano).
- V prípade úspechu spustenie nahlási, koľko obrazových produktov zapísalo (`Image products written: N`).

#### Spustenie, pri ktorom sa nezapíšu žiadne obrázky, zlyhá

Ak ste požiadali o výstupy a spustenie nezapísalo **žiadne** — iba `project.json` a
`calibration_data.json` — `process` to považuje za zlyhanie: vypíše
`Processing finished but wrote no image products.` a **ukončí sa s nenulovým kódom**, takže skript to môže
zistiť. Správa uvádza názov projektovej zložky a bežné príčiny:

- vstupná zložka nebola rozpoznaná ako záznam (skontrolujte rozloženie a `--input-level`), alebo
- každý požadovaný produkt bol vynechaný, pretože bol pre dané kamery nepoužiteľný (napr. požiadavka na
  radianciu/reflexiu z kamier, ktoré podporujú iba RGB).

Spustite znovu s `--verbose` a skontrolujte protokol backendu, či sa v ňom nachádzajú riadky `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
ktoré vysvetľujú, ktorékamery, ktoré by inak neboli zahrnuté vo výstupe CLI.

Úmyselné spustenie iba s metadátami — všetky produkty vypnuté a bez `--indices` — je stále
**úspešné**, pretože prázdny obrazový výstup je v tomto prípade správnym výsledkom.

Rovnako tak aj **spustenie iba so svetelnými senzormi**: zložka so záznamami `.daq` neobsahuje žiadne snímky na export
už zo samotnej definície a beh sa hodnotí na základe kalibrovaných súborov `.daq` / `.csv`, ktoré namiesto toho vytvoril.

---

## `chloros-cli login`

Overte toto zariadenie pomocou cloudového účtu Chloros+. Prihlasovacie údaje sú bezpečne uložené v cache v súbore `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Príklady

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (odstránením časti hesla alebo duplikovaním jeho častí). Pri chybe 401 služba CLI automaticky opakuje pokus s opätovným pridaním `$$`, potom s polovicou hesla bez duplicitných znakov; ak je opakovaný pokus úspešný, prihlási vás a vypíše správnu-uvedie správnu syntax, ktorú treba použiť nabudúce.

> **Použitie bez grafického rozhrania/prostredníctvom skriptu: žiadna uložená relácia znamená interaktívnu výzvu, nie rýchlu chybu.** Akýkoľvek príkaz na spustenie backenduspúšťajúci proces (`process`, `status`, `export-status`, `time-sync`, …) beží bez uloženého licenčného/relácie sa pred pokračovaním dostane do interaktívneho riadku príkazov `Email:` / `Password:` na štandardnom vstupe. Úloha bežiaca bez obsluhy bez uloženého relácie sa preto zablokuje a bude čakať na vstup — pred naplánovaním úlohy v režime bez grafického rozhrania spustite raz na počítač pred naplánovaním úlohy bez grafického rozhrania.

---

## `chloros-cli logout`

Vymaže uloženú reláciu a vynúti nové prihlásenie pri nasledujúcom volaní.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Zobrazí aktuálnu úroveň licencie (Iron/Copper/Bronze/Silver/Gold), overeného používateľa a počet priradení k zariadeniam.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Zisťuje aktuálny priebeh exportu Thread-4. Je bezpečné volať **počas** behu `process` z iného shellu.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Nastavte jazyk zobrazenia v nástroji „CLI“ (podporovaných je 38, vrátane CJK, RTL a indických písem). Na starších konzolách, ktoré nedokážu zobraziť skript, sa hladko prepne na angličtinu.

```
chloros-cli language [LANG_CODE] [--list]
```

### Príklady

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Príkazy pre priečinok projektu

Tieto príkazy spravujú umiestnenie predvoleného priečinka projektu (zdieľané s grafickým rozhraním).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Iba pre Jetson. Skontroluje súbor `version_url` z `/etc/chloros/update.conf` a ponúkne stiahnutie a inštaláciu zodpovedajúceho súboru `.deb`.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Na Linux / Jetson spúšťa CLI tiež **automatickú kontrolu aktualizácií pri každom spustení** (neblokujúcu, nikdy nespomaľuje príkaz): prečíta hodnotu `/etc/chloros/update.conf`, výsledok uloží do vyrovnávacej pamäte na 1 hodinu v súbore `~/.chloros/update_cache.json` a v prípade existencie novej verzie vypíše hodnotu `Update available: vX.Y.Z / Run: chloros-cli update`. Pri akejkoľvek chybe a na adrese Windows sa táto kontrola ticho preskočí.

---

## `chloros-cli selftest`

Spustí 7krokový rýchly test: verzia, dostupnosť portu, spustenie backendu, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), prítomnosť modelu odšumovača, pripravenosť CUDA+odšumovača.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Stav a ovládanie PTP grandmastera. Hostiteľ „Chloros“ spúšťa PTP grandmaster; kamery LATTICE a jednotky DAQ-E sú k nemu pripojené ako podriadené zariadenia na účely časových značiek medzi zariadeniami.

| Podpríkaz | Popis |
| --- | --- |
| `status` | Zobraziť stav grandmastera, priority BMCA, identitu hodín. |
| `peers` | Zoznam podriadených zariadení detekovaných prostredníctvom Delay_Req (kamery + senzory DAQ-E). |
| `cameras` | Stav PTP pre jednotlivé kamery (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Reštartovanie procesu grandmaster. |
| `set-priority --priority1 N --priority2 N` | Prepisovanie priorít BMCA. |

### Príklady

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Ovládanie kamery LATTICE. Každý podpríkaz prechádza cez backend Chloros; tento backend spravuje skupinu kamier, takže nasledujúce volania CLI opätovne používajú rovnakú otvorenú referenciu.

### Bežné možnosti (spoločné pre väčšinu podpríkazov)

| Príznak | Popis |
| --- | --- |
| `-d, --device N` | Index kamery (predvolené: 0). |
| `-s, --serial SN` | Špecifické sériové číslo; prepisuje `--device`. |
| `--serials SN1,SN2,…` | Sériové čísla oddelené čiarkami pre prevádzku s viacerými kamerami. |
| `--all` | Prevádzka na každej zistené kamere. |
| `--exposure US` | Doba expozície v mikrosekundách. |
| `--gain DB` | Zosilnenie v dB. |
| `--pixel-format FMT` | napr. `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Rozmery obrazu. |
| `--preset {default,high_quality,high_speed,triggered}` | Použiť prednastavenie. Všetky sú vo voľnom režime okrem `triggered`, ktorý aktivuje kameru na hardvérový signál na linke 2 — ak táto linka nie je aktivovaná, bude čakať donekonečna namiesto toho, aby snímal. |
| `-o, --output DIR` | Výstupný adresár (predvolené: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Veľkosť paketu GVSP. `auto` spúšťa sondy ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Postup prvého pripojenia kamery LATTICE

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referencia podpríkazov

#### Zisťovanie a informácie

| Podpríkaz | Účel |
| --- | --- |
| `lattice info` | Zoznam pripojených kamier (výrobca, model, sériové číslo, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analýza hostiteľského systému na účely optimálnej konfigurácie kamery. `--no-discover` preskočí vyhľadávanie kamery (rýchlejšie, analýza iba na úrovni sieťovej karty). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Kontrola/oprava nastavení sieťovej karty; odhad šírky pásma/FPS. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Sieťové možnosti backendu so stabilnou schémou + odporúčanie poľa (vráti `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` zachová požadované rozlíšenie, ale obmedzí cieľovú hodnotu snímok za sekundu — prečítajte `recommended.recommended_target_fps` a odovzdajte ho ako cieľ pripojenia; považujte to za úspech, nie za chybu. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Analýza „čo keby“ bez spustenia kamier. **`--n-active` je celkový počet kamier v sieti, nie len v tomto poli**— zvýšte ho, ak samostatné kamery vysielajú súčasne, alebo ak je kapacita siete vypočítaná na základe požiadavky, ktorá ich podhodnocuje (predvolené: `len(--models)`). Vždy vypíše súhrnné riadky `Wire budget:` (požadovaných MB/s vs. strop bezpečný voči kolíziám) a `Max cameras:` a označí `** OVER-SUBSCRIBED**`, ak pole prekročí kapacitu siete — pozri [Model fps a burstového prenosu poľa](#array-fps--burst-model). |
| `lattice gpu` | Zobraziť stav GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Skontrolovať alebo aktualizovať firmvér kamery. Miestny výber `.fwa` je zafixovaný: súbor v `firmware/<MODEL_PREFIX>/` zodpovedajúci verzii zostavenia`MIN_FIRMWARE_VERSION` sa nahraje, ak je k dispozícii (najnovšia verzia slúži iba ako záložná možnosť), takže novší obraz od dodávateľa uložený na disku zostáva neaktívny, kým sa tento výstup neaktualizuje — zámerne novšie verzie sa dostávajú do zariadení prostredníctvom podpísaného manifestu AWS, ktorý sa uprednostňuje, ak je novší. |
| `lattice presets [--apply NAME]` | Zobraziť alebo použiť prednastavenia kamery. |
| `lattice status` | Stav kamery v reálnom čase. |

#### Zachytenie

| Podpríkaz | Účel |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Jednotlivý snímok. **Štandardne ukladá všetky typy exportu** (`--processing all`); pozri [Úrovne exportu záznamov](#úrovne-exportu-zachytávania-všetko-predvolené). `--levels` uloží explicitnú podmnožinu (prepíše `--processing`); `--force-daq` zapíše priradenú hodnotu z DAQ ako vedľajšíaj pri zachytení iba surových údajov. `--jpeg-quality` = JPEG kvalita 1–100 (predvolené nastavenie 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Ukladanie na disk až do stlačenia Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Živé náhľady MJPEG v prehliadači. `--ae-damping` nastavuje tlmenie automatickej expozície (0,4–100). |

#### Nastavenie snímača

| Podpríkaz | Účel |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Čítanie/zápis ľubovoľného uzla GenICam. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Expozícia a automatická expozícia (AE). |
| `lattice gain [--auto] [--off] [--set DB]` | Zosilnenie a automatické zosilnenie. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Oblasť záujmu (ROI) snímača a binning. |
| `lattice format [--set FMT] [--list]` | Formát pixelov. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Hardvérová/softvérová spúšť. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (žiadne príznaky = jednorazová kompenzácia bielej) | Operácie kompenzácie bielej. Iba kamery typu „RGB“ /Bayer; na monochromatických kamerách M3M sa táto operácia neuskutoční (preskočí sa). |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | Farebný procesRGBného zobrazenia. `natural` (predvolené) je lacná úprava v reálnom čase; `enhanced` pridáva odstránenie farebných okrajov + živosť + lokálny kontrast CLAHE pre plný vzhľad s paritou v strede pri ~2× vyšších nákladoch na úpravu na snímku, takže nižšia **živá** snímková frekvencia — uložené zábery vždy dostanú plnú úpravu v oboch prípadoch. RGB /Blen pre kamery s viacerými kanálmi; pri mono M3M sa preskočí. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Zobraziť sýtosť/kontrast (kamery s filtrom RGB). Pri mono M3M sa preskočí. |
| `lattice filter [--set NAME] [--list]` | Nastaviť model filtra kamery (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Zisťuje napájacie/tepelné uzly; prepnutie do režimu nízkej spotreby v kľude. |

#### Kalibrácia a senzory

| Podpríkaz | Účel |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibrovať pomocou cieľa odrazivosti. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Vstavanépríkazy pre senzor dopadajúceho svetla. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Uplatniť korekciu vinetácie na existujúce snímky. |

#### Viac kamier (prechodné relácie)

| Podpríkaz | Účel |
| --- | --- |
| `lattice multi-info` | Zoznam všetkých kamier so synchronizačnými úlohami. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Jeden synchronizovaný snímok z každej kamery. Pri pripojení trvalého poľa sa **štandardne ukladajú všetky typy exportu**; dočasná náhradná možnosť bez poľa je**len bez debayeringu** (pre ostatné najprv spustite `array-connect`). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Streamovanie synchronizovaných snímok (dočasné). |
| `lattice multi-test [--count N]` | Test časovania synchronizácie GPIO. |
| `lattice multi-detect [--line LINE] [--json]` | Automatická detekcia zapojenia GPIO master/slave. |

#### Zarovnanie

| Podpríkaz | Účel |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — plus ovládače detektora/porovnávača `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, ovládače RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombinácia viacerých snímok `[--averaging mean\|median\|inlier_weighted]`, geometrické obmedzenia `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, priestorové obmedzenie `[--roi X0,Y0,X1,Y1] [--mask PATH]` a prepisovanie nastavení pre jednotlivé podriadené kamery `[--per-cam-override SN:KEY=VALUE]` (opakovateľné) | Vypočíta profil zarovnania z živých kamier. `--prefilter` je predvolene nastavené na `gradient` (mapa hrán; zodpovedá vyrovnávaciemu nástroju v grafickom rozhraní/pole — hrany pretrvávajú naprieč spektrálnymi pásmami). `--matcher flann` sa oplatí pri počte charakteristík nad ~5000; `--averaging median` je odolný voči jednému chybnému záberu, `inlier_weighted` váži podľa počtu zhôd; `--lock-scale` premieta na najbližšiu rotáciu (bez mierky), `--lock-axis` vynuluje jednu zložku posunu; `--mask` sa uplatňuje na každú kameru (pre nastavenia pre jednotlivé kamery použite `--per-cam-override`, napr. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` odmietne uloženie kalibrácie, ktorej RMS reprojekcie prekračuje limit. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Zachytí jeden zarovnaný viacpásmový snímok. `--bit-depth` je prednastavený na prispôsobenie kamere; `--no-crop` zachováva celý snímok (vyplní ho čiernou farbou); `--interpolation` (predvolené nastavenie `linear`) a `--border-mode`/`--border-value` (predvolené nastavenie `constant`/0) riadia warp procesora — cesta grafického procesora je v každom prípade bilineárna. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Streamovo zarovnané viacpásmové snímky (rovnaké ovládače warpu ako `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Zobraziť podrobnosti profilu. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Zmeniť poradie vrstiev. |

#### Index / Matematika vegetácie

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Úplná sada príznakov: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (opakovateľné), `--capture-level raw|debayered|radiance|reflectance|unknown` (prepíše úroveň zachytávania zaznamenanú v zdrojovom TIFF; predvolené nastavenie: načítanie z metadát TIFF), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Pri `--live` sa uplatňujú aj ovládače deformácie zarovnania: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **Pri symboloch `--channel` sa rozlišujú veľké a malé písmená.** Symboly musia presne zodpovedať názvom kanálov v predvolbách (predvolby používajú malé písmená, napr. NDVI = `red`,`nir` — skontrolujte `--list-presets`), a časť s pásmom sa musí zhodovať s názvom pásma v zarovnanom zozname (alebo byť 0index pásma v režime offline). `--channel red=Red_660 --channel nir=NIR_850` funguje; `--channel RED=660` zlyhá s chybou `channel_map missing entries`.

#### Trvalé pripojenia (Smart-Prep, tok ekvivalentný GUI)

Tieto príkazy udržujú kamery otvorené v backendovom poole aj po viacerých volaniach CLI.

| Podpríkaz | Účel |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Pridať jednu kameru do poola (jedna kamera, bez poľa). |
| `lattice cam-disconnect [--serial SN] [--all]` | Uvoľniť. |
| `lattice cam-list` | Zobraziť zoznam kamier v poole. |
| **`lattice array-connect`**|**Pripojenie trvalého synchronizovaného poľa (ODPORÚČANÝ vstupný bod).** Spustí kompletný proces prípravy inteligentného grafického rozhraniaprípravný postup. |
| `lattice array-disconnect [--array-id ID] [--all]` | Uvoľniť pole. |
| `lattice array-list` | Zobraziť zoznam pripojených polí. |
| `lattice array-status [--array-id ID]` | Aktuálne fps, PTP, posledná chyba. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Jedno synchronizované zachytenie z aktívneho poľa — Jednorazové / Nepretržité / Intervalové / Najrýchlejšie. **Predvolené nastavenie je `all`** (jeden súbor na príslušný typ exportu na jednu kameru). Vynechané kamery (napr. RGB vylúčené z odrazivosti/odrazivosti) sú hlásené s kódom `Skipped: SN:<serial> (<reason>)`; hodnota DAQ použitá pre odrazivosť sa ukladá spolu s nimi a hlási sa s kódom `DAQ: <path>`. Pozri [Režimy snímania, záznamníky a offline spracovanie](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Nahrávanie živého zobrazenia kombinovaného indexu do videa/GIF (na účely monitorovania; vyžaduje otvorený kombinovaný prúd). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Séria surových snímok Bayer s vysokou snímkovou frekvenciou (na účely analýzy; offline spracovanie). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Opätovné spracovanie uloženej surovej série snímok na kalibrované video(-á). |

##### Možnosti `array-connect`

| Príznak | Predvolené | Popis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatické vyhľadanie všetkých kamier LATTICE (vyžaduje ≥2) | Prvá v poradí je MASTER. Ak sa neuvádza, vyhľadávanie filtruje na modely LATTICE (`TRI032*`) a pripojí ich všetky. |
| `--line {Line0,Line2,Line3}` | `Line2` | Synchronizačná linka GPIO. |
| `--target-fps F` | auto | Rýchlosť spúšťania hlavnej kamery. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Prepis výberu úrovne. |
| `--wire-ceiling-mbps MB_PER_S` | automaticky detekované | **Trvalý rozpočet hostiteľa na prenosovú šírku pásma, v MB/s — číslo, od ktorého závisí alokácia celého poľa.** Znížte ho, ak pole hlási rámce poškodené GVSP: automatická hodnota je odvodená od deklarovanej rýchlosti spojenia sieťovej karty, ktorá nadhodnocuje USB adaptéry, úzke PCIe kanály a vyťažené zdieľané štruktúry. Ukladá sa do bloku zachytávania poľa projektu, takže opätovné otvorenie / CLI / SDK opätovné pripojenie . Pozri [Stav poľa](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Hardvérové zoskupovanie. |
| `--no-recommend` | vypnuté | Preskočiť krok analýzy siete. |
| `--no-ptp` | vypnuté | Zakázať PTP (časové značky medzi kamerami potom **nie sú** porovnateľné). |

### Smart-AE / Smart-Capture

Polia LATTICE spúšťajú nepretržité automatické zaostrovanie (AE) na pozadí hneď po pripojení, ale scéna s čerstvonastavená scéna chvíľu trvá, kým sa ustáli. `array-capture --smart` je **pripravené praktické riešenie**: počká, kým sa AE ustáli na všetkých kamerách v poli, a potom spustí snímanie. Použite ho, keď meníte scény uprostred relácie.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Politika ustálenia je štandardne konzervatívna: časový limit 5 s, okno stability 1,5 s, tolerancia rozptylu expozície ±5 %. Nastavte prostredníctvom súboru SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) , ak potrebujete od automatizácie iné správanie.

### Úrovne exportu snímok (predvolené nastavenie `all`)

Od tejto verzie sú `lattice capture`, `lattice multi-capture` a `lattice array-capture` **prednastavené na `--processing all`** — jeden uložený súbor na typ exportu , ktorý sa vzťahuje na každú kameru, čo zodpovedá správaniu funkcie „Zachytiť všetko“ v grafickom rozhraní. Úrovne sú:

| Úroveň | Výstup | Vzťahuje sa na |
| --- | --- | --- |
| `raw` | Jednokanálový Bayer (mono kamery: jeden pásmo) priamo zo snímača. | Všetky kamery. |
| `debayered` | 3-kanálová demosaika BGR (mono kamery: 1-kanálová škála sivej). | Všetky kamery. |
| `radiance` | float32 W/m²/sr/nm cez kompletný radiometrický reťazec. | Iba multispektrálne (M3C/M3M) — **vynechané pre kamery s filtrom RGB**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), pripravené pre Pix4D. | Iba multispektrálne a **len ak je priradené zariadenie DAQ a kamera je kalibrovaná**; inak sa preskočí. |
| `preview` / `display` | Kompletný reťazec náhľadov v grafickom rozhraní (CCM + WB + gama podľa profilu kamery). `lattice capture` nazýva tento `preview`; `array-capture`/`multi-capture` používajú `display`. | Všetky kamery. |

Zadajte jednu úroveň, aby sa uložila len táto jedna (`--processing debayered`). Keď požiadate o `all`, úrovne, ktoré sa nevzťahujú na danú kameru, sa preskočia (a nahlásia), bez hlásenia chyby — nepripojená alebo nekalibrovaná kamera stále dostane `raw` / `debayered` / `preview`.

Pre každý snímok odrazivosti sa skutočne použitá hodnota DAQ pre zosilnenie v smere nadol zapíše do súboru **`.daq`** vedľa snímky (aby bolo možné snímku neskôr opätovne spracovať) a nahlási sa v riadku `DAQ:`.

### Ako vyzerá priečinok so záznamami

Každý typ exportu sa ukladá do **vlastnej podzložky** pod `-o`, takže pri viacúrovňovom zázname sa typy nikdy nemiešajú:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` je časová pečiatka záznamu časová pečiatka a `<serial>` sériové číslo kamery, takže jedna synchronizovaná skupina zdieľa
časovú pečiatku medzi kamerami. **Všimnite si jednu asymetriu:** úroveň `display` je uložená v priečinku
s názvom `preview/`, zatiaľ čo samotné súbory si v názve zachovávajú `_display` — názov priečinka a prípon sa líšia
len pre túto úroveň. Neznáme úrovne sa ukladajú do zložky s vlastným názvom a ak sa podzložka
nedá vytvoriť, súbor sa zapíše do koreňového adresára výstupu, namiesto toho, aby sa stratil.

**Opätovné spracovanie zložky s nahrávkami:**nasmerujte `chloros-cli process` na**koreňový adresár zachytených snímok**
(`output/`). `process` zvyčajne importuje len zložku, ktorú určíte, ale ak táto zložka neobsahuje žiadne
obrázky a má podzložky, automaticky prechádza do nižších úrovní – takže podadresáre na úrovni koreňového adresára a samotný
koreňový adresár `.daq` sa načítajú naraz. Každá úroveň záznamu sa importuje ako jeden obrázok, pričom
ostatné úrovne sú k dispozícii ako režimy, namiesto toho, aby bol každý obrázok na samostatnej úrovni.

Priamo pomenovať **podzložku na úrovni** (napr. `output/raw/`) tiež funguje. Týmto spôsobom sa koreňová zložka
`.daq` vynechá, preto skopírujte alebo nasmerujte čítanie DAQ vedľa keď znovu odvádzate radiometrický
produkt z `raw/` – inak sa časová pečiatka nebude mať s čím porovnať.

**Spracovanie vždy začína od `raw`.** V rámci každého záznamu je zdrojom spracovateľského reťazca surový snímok;
`debayered`, `radiance`, `reflectance` a `preview` sa objavujú ako režimy na prezeranie, ale nikdy sa nevracajú
späť do spracovateľského reťazca. Opätovné spracovanie odvodeného produktu by znamenalo opätovné uplatnenie vinetácie, CCM a
výpočty žiarivosti, ktoré sú už zapracované do jeho pixelov, preto sa funkcia „Chloros“ (Zrušiť spracovanie) radšej vypne, ako aby
sa vykonalo dvojité spracovanie. Dva dôsledky, o ktorých stojí za to vedieť:

- Rendery `index/` a `composite/` sa **nikdy** nespracúvajú. Sú to výstupy, nie zachytenia —
  render s LUT „NDVI“ nemá žiadny zmysluplný výklad radiancie.
- Zložka so záznamami exportovaná **bez** súboru `raw` (napr. `array-capture --processing reflectance`) nemá
  žiadny platný zdroj v spracovateľskom reťazci. Tieto záznamy sa importujú a zobrazujú normálne, ale `process` ich preskočí
  a oznámi to takto:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Ak skutočne potrebujete prejsť odvodeným produktom – reláciou hubu zachytenou s
  zapnutým `demosaic` alebo starším priečinkom – `--input-level {raw,debayered,processed}` vynúti vstupný
  bod a prepisuje preskočenie. Táto vlajka je zámerným únikovým východom; `auto` (predvolené nastavenie)
  nikdy nespracuje záznam, ktorý neobsahuje surové dáta.

### Preskočené záznamy v poliach so zmiešanými filtrami

Ak v jednom poli kombinujete kamery typu „RGB“ a multispektrálne kamery, `array-capture --processing radiance` (alebo `reflectance`) uloží multispektrálne snímky a **preskočí** kamery typu „RGB“ — podľa-Bayerov radiančný signál nemá pre širokopásmový senzor zmysel. Súbor CLI explicitne vypíše každý uložený súbor (s úrovňou exportu), každý zapísaný súbor `.daq` a každé preskočenie, takže počet súborov nie je prekvapujúci:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Značky dôvodu preskočenia majú formát `<level>-not-applicable-to-rgb-cam`. Odrazivosť sa môže preskočiť aj s `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` a pomocou `dls-uncalibrated-band-<nm>`, ak sa pásmo nachádza prevažne mimo radiometricky kalibrovaného rozsahu svetelného senzora DAQ (~374–974 nm) — spomedzi dodávaných SKU iba F988, ktorého podporovanou cestou je pracovný postup s panelom odrazivosti.

Použite `--processing debayered` (alebo `display`) na zahrnutie každej kamery bez ohľadu na typ filtra, alebo predvolený `all`, aby ste získali všetky príslušné úrovne pre každú kameru naraz.

---

## Režimy snímania, záznamníky a offline spracovanie

Všetky tieto funkcie pracujú s **trvalým poľom** (najskôr spustite `array-connect`). Zrkadlia panel snímania v grafickom rozhraní.

### Režimy `array-capture`

`array-capture` je jediný príkaz so štyrmi režimami uzávierky a súborom prepínačov exportu:

| Režim | Príznak | Správanie |
| --- | --- | --- |
| **Jednorazový** *(predvolený)* | (žiadny) | Jedna synchronizovaná skupina snímok, potom ukončenie. |
| **Neprerušované** | `--continuous` | Po sebe idúce cykly až do `Ctrl+C`, `--count N` alebo `--duration S`. |
| **Interval** | `--interval S` | Jeden prechod každých `S` sekúnd (merané od začiatku každého prechodu), rovnaké hranice. |
| **Najrýchlejšie** | `--fastest` | Iba surové údaje + priradená hodnota z DAQ + kompozitný index; preskakuje výpočty odrazivosti/odrazivosti/zobrazenia, aby sa snímka načítala rýchlo. Predpokladá `--processing raw --force-daq`. Uložené údaje `.daq` neskôr spracujte na kalibrované produkty. |

Prepínače exportu (možno kombinovať s ľubovoľným režimom; všetky zdieľajú grafické rozhranie a koncový bod SDK):

| Priepínač | Účinok |
| --- | --- |
| `--processing LEVEL` | Jedna úroveň exportu alebo `all` (predvolené). |
| `--levels L1,L2,…` | Explicitná podmnožina typov exportu (napr. `raw,radiance,reflectance`); **prepíše `--processing`**. |
| `--aligned` / `--no-aligned` | Deformuje ne-surový export každého člena podľa [profil zarovnania](#alignment) (spoluregistrované). Surové údaje zostávajú nezarovnané, ale transformáciu nesú v metadátach. Vráti sa k nezarovnanému (s varovaním), ak pole nemá profil. |
| `--index` / `--no-index` | Uloží / preskočí prekrývací obrazec indexu vegetácie pre každú kameru, ak je nakonfigurovaný. Predvolené nastavenie: vykresliť ho. |
| `--force-daq` | Uložiť priradené čítanie DAQ/DLS ako súbor typu sidecar `.daq`, aj keď to žiadna zvolená úroveň nevyžaduje (napr. snímka obsahujúca len surové dáta), aby bolo možné snímky offline spracovať na odrazivosť/index. |
| `--smart` | Pred spustením počkať, kým sa AE ustáli na všetkých kamerách (pozri [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Kompresia pixelov typu „TIFF“. `deflate` (predvolené) = bezstratová kompresia zlib L1 + horizontálny prediktor, ~4,1 MB na snímku v plnom rozlíšení; `none` = bez kompresie, ~5× rýchlejšie zapisovanie pri ~6,3 MB na snímku — použite pre maximálnu trvalú rýchlosť, ak to disk umožňuje. Oba sú bezstratové a pri importe sa čítajú identicky. |

> **TIFF s jednorazovým zápisom + model s trvalou rýchlosťou.**Záznamy sa zapisujú v**jednom**prechode do súboru TIFF, ktorý obsahuje pixely + XMP + IFD0 Make/Model (merané na Mono12 v plnom rozlíšení: 36 ms komprimované / 6,5 ms nekomprimované, oproti ~148 ms pri starom zápise a následnom-ExifTool); jediná zostávajúca úloha ExifToolu (dolaďovanie EXIF sub-IFD) beží v asynchrónnom pozadí a snímka je kompletná a pripravená na import aj v prípade, že sa táto úloha nikdy nespustí. Všimnite si, že kompresia DEFLATE drží GIL (Python), takže komprimované zápisy sa**ne**paralelizujú naprieč vláknami zapisovača pre jednotlivé kamery — nepretržité snímanie v plnom rozlíšení s 8 kamerami pri rýchlosti snímača (~10,4 fps) vyžaduje `--compression none`**a** disk triedy NVMe (~500 MB/s pri nepretržitom zápise). Ten istý ovládač je dostupný ako `compression` na `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — kombinovaný index video/GIF (monitorovacia kvalita)

Zaznamenáva všetko, čo **živý kombinovaný indexový pohľad** zobrazuje do `.avi` (a voliteľne do `.gif`). Keďže čerpá z živého kompozitného signálu, kombinovaný prúd musí byť otvorený (t.napr. pole sa prezerá v grafickom rozhraní), aby sa mohli snímky ukladať. Každé 2 s kontroluje priebeh a zastaví sa na zariadeniach `--duration`, `Ctrl+C` alebo keď sa nahrávač sámukončí.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Príznak | Predvolené | Popis |
| --- | --- | --- |
| `--array-id ID` | iba pole | Cieľové pole (vynechajte, ak je pripojené len jedno). |
| `-o, --output DIR` | `output` | Výstupný adresár (backend-local). |
| `--fps F` | `10` | Rýchlosť záznamu snímok. |
| `--duration S` | až do stlačenia Ctrl+C | Automatické zastavenie po `S` sekundách. |
| `--gif` | vypnuté | Zapísať aj animovaný súbor GIF. |
| `--gif-only` | vypnuté | Zapísať len súbor GIF (bez `.avi`). |

### `array-burst` — séria snímok v surovom formáte Bayer s vysokou snímkovou frekvenciou (na analytické účely)

Priamo číta vyrovnávaciu pamäť synchronizovanej skupiny snímkovacej slučky — **nie je potrebný kalibračný reťazec, exiftool ani živý náhľad** — takže beží pri plnej rýchlosti snímania fotoaparátu. Ukladá surové snímky + manifest nasnímku a jeden súbor `.daq` pre každé odlišné čítanie DLS pod `<output>/bursts/<base>/`. Opätovne spracujte offline (ďalší príkaz)alebo odovzdajte `--build`, aby sa to vykonalo ihneď po zastavení.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Prieznak | Predvolené | Popis |
| --- | --- | --- |
| `--array-id ID` | len pole | Cieľové pole. |
| `-o, --output DIR` | `output` | Výstupný adresár (výstup sa ukladá do `<DIR>/bursts/<base>/`). |
| `--duration S` | až do Ctrl+C | Automatické zastavenie po `S` sekundách. |
| `--max-frames N` | neobmedzené | Automatickézastavenie po `N` surových snímok. |
| `--build` | vypnuté | Po zastavení ihneď opätovne spracovať sériu (rovnako ako `array-build-video`). |
| `--products …` | `combined:index` | S `--build`: ktoré video(-á) vytvoriť (pozri nižšie). |
| `--fps F` | `10` | S `--build`: počet snímok za sekundu (fps) výstupného videa. |
| `--save-tiffs` | vypnuté | S `--build`: ukladať aj kalibrované súbory TIFF pre každý snímok. |
| `--gif` | vypnuté | S `--build`: ukladať aj animované súbory GIF. |

### `array-build-video` — offline spracovanie uloženej série snímok

Časovo zosúladí každý surový snímok s najbližšou uloženou hodnotou `.daq` a prejde ho cez **rovnaký reťazec radiancie / odrazivosti / indexu ako importný proces**, čím vytvorí jedno alebo viac videí.

`--products` je zoznam oddelený čiarkami obsahujúci `kind:level` položiek, kde `kind` ∈ `per_cam` | `combined` a `level` ∈ `radiance` | `reflectance` | `index`. Samotný `level` (bez `kind:`) má predvolenú hodnotu `per_cam`. Predvolená hodnota je `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Prieznak | Predvolené | Popis |
| --- | --- | --- |
| `--burst-dir DIR` | (povinné) | Cesta k zložke sériových záberov (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Zoznam `kind:level`, ako vyššie. |
| `--fps F` | `10` | Počet snímok za sekundu výstupného videa. |
| `--save-tiffs` | vypnuté | Okrem videa (videí) ukladať aj kalibrované súbory TIFF pre každý snímok. |
| `--gif` | vypnuté | Okrem toho zapisovať aj animované súbory GIF. |

> **Vyberte správny záznamník.** `array-record` je *monitorovacej triedy* — zaznamenáva živý kompozitný obraz tak, ako sa zobrazuje, a vyžaduje otvorený prúd. `array-burst` → `array-build-video` je *analytickej* — ukladá surové údaje zo senzorov pri plnej rýchlosti a následne rekonštruuje kalibrované videá žiarivosti/odrazivosti/indexu, pričom nie je potrebný živý náhľad.

### Mono (M3M) jednopásmové kamery

Rad **M3M**je monochromatickým ekvivalentom radu Bayer**M3C**: jedna úzkopásmová interferenčná filtračná mriežka na kameru (napr. `M3M-<lens>-F<wavelength>`, `M3M-L87-F685`), takže snímač poskytuje**jedno pásmo v odtieňoch šedej** bez Bayerovej mozaiky. Nie je potrebné vykonávať demosaic, odstraňovať presluchy medzi kanálmi ani nastavovať vyváženie bielej – celý RGB -farebný proces spracovania obrazu sa jednoducho neuplatňuje.

Čo to znamená pre CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**rozpoznajú monochromatickú kameru a**preskočia s jednoriadkovou správou** namiesto toho, aby aplikovali bezvýznamné nastavenia. V tej istej relácii stále fungujú normálne s kamerou typu RGB /Bayer M3C.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** stále fungujú — žiarivosť a odrazivosť sú *na pásmo* radiometrické mapy a sú pre jedno pásmo dokonale definované. Mono snímky obsahujú **identitnú** (bez 3×3 unmixu), takže prechádzajú kalibračnými výpočtami bez zmien.
- **Jedna monochromatická kamera nedokáže vygenerovať vegetačný index.**NDVI / NDRE /atď. Potrebujete aspoň dve pásma (napr. Red + NIR). Ak chcete získať index z monochromatického hardvéru, nasmerujte**niekoľko** kamier M3M na rôzne vlnové dĺžky, zosúladte ich do jedného viacpásmového súboru a vypočítajte index *z toho*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` – symboly sa musia **presne** zhodovať s názvami kanálov v predvolbe (rozlišuje sa veľkosť písmen; NDVI sú písané malými písmenami `red`,`nir` — pozri `--list-presets`), a názvy strán pásma označujú pásmo v zarovnanom stohu (režim offline akceptuje aj indexy pásiem počítané od 0, napr. `--channel red=0 --channel nir=1`).

Rozlišovacím znakom v celom stohu je token `M3M` v reťazci modelu (nikdy sa neobjavuje v reťazci `M3C`), ktorý sa v grafickom rozhraní (GUI) / SDK zobrazuje ako `is_mono`.

---

## Nastavenie a ladenie sieťovej karty hostiteľa (súbory kamier LATTICE)

Kamery LATTICE prenášajú GVSP cez ethernetovú kartu hostiteľa, takže pri súboroch s viacerými kamerami má **ovládač**a**veľkosť prijímacieho prstenca** sú rovnako dôležité ako rýchlosť spojenia. Nesprávne nastavenia sa prejavujú ako brána `FRAMES WILL DROP` / `Reduce ROI to enable` v paneli Nastavenia poľa (a v `lattice network-analysis` / v `analyze_array_network()` v nástroji „SDK“), a to aj vtedy, keď sú samotné kamery v poriadku.

### USB 10GbE adaptéry — Realtek RTL8157 („Realtek USB 10GbE Family Controller“)

| Položka | Požadovaná hodnota | Prečo je to dôležité |
| --- | --- | --- |
| **Verzia ovládača**|**≥ v10.67 (január 2026)**, INF `rtump64x64sta.inf` | Starší ovládač z roku**2016**(v10.65, `rtump64x64.inf`) nesprávne spracováva vypnutie a vyvoláva chyby typu**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**pri vypínaní/reštarte/prechode do režimu spánku. Prechod sa zasekne (časový limit ~5 minút), používateľ zariadenie násilne vypne a opakované nečisté vypnutia**poškodia úložisko WMI**(PowerShell a nástroje začínajú vykazovať chyby s kódom `Invalid class`) a**blokujú USB stack** pri nasledujúcom spustení (sieťová karta sa neaktivuje; USB zariadenia prestanú byť rozpoznávané). Pred spustením čistých reštartov si stiahnite aktualizáciu z realtek.com (alebo od výrobcu hardvérového kľúča). |
| **Prijímacie vyrovnávacie pamäte**— kľúčové slovo `ReceiveBufferLen` |**256**(maximum ovládača) | Prijímací krúžok sieťovej karty. Východisková hodnota ovládača**32**ponecháva len ~0,26 MB použiteľného priestoru v krúžku — čo je príliš málo na burst z viacerých kamier — preto panel poľa hlási chybu `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` a blokuje pripojenie. Pri hodnote**256**je kruh veľký (**~13,5 MB namerané na laboratórnom 10GbE hostiteľovi**), čo poskytuje prijímacej (RX) potrubnej linke skutočnú rezervu pre sériové prenosy GVSP z viacerých kamier. (O tom, či daná konfigurácia skutočne *vytvorí spojenie*, rozhodujú dve kontroly — kontrola priepustnosti **zohľadňujúca vyprázdňovanie**a kontrola**agregátneho prekročenia kapacity** — nie na základe surového porovnania burstu s krúžkom; pozri [Model fps a burstu poľa](#array-fps--burst-model).) |
| **Prijímacie URB**— kľúčové slovo `PendingReceives` |**64** (max.) | Bloky požiadaviek USB v prenose; zvýšte spolu s prijímacími vyrovnávacími pamäťami na absorpciu sériových prenosov. |
| **Jumbo rámec** — kľúčové slovo `*JumboPacket` | **9014** | Potrebné pre pakety GVSP s veľkosťou 9000 bajtov (6× menej paketov na rámec ako pri 1500). |

> ⚠️ **Aktualizácia ovládača sieťovej karty RESETUJE tieto pokročilé vlastnosti na predvolené hodnoty.**Po aktualizácii alebo výmene ovládača adaptéra**znova nastavte** `ReceiveBufferLen=256` a `PendingReceives=64`, inak sa panel poľa opäť zablokuje, hoci sa „na hardvéri nič nezmenilo“. Toto je hlavná príčina toho, že zariadenie, ktoré predtým fungovalo, sa náhle odmietne pripojiť.

Nastavte z **zvýšeného**PowerShell** (nahraďte názov svojho adaptéra, napr. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` sa vzťahuje na USB 10GbE adaptéry.** Teraz rozpozná typ adaptéra a nastaví správne kľúčové slovo prijímacieho prstenca: `*ReceiveBuffers`→2048 pre sieťové karty PCIe (Intel I219 atď.) alebo `ReceiveBufferLen`→256 + `PendingReceives`→64 pre radič Realtek **USB** 10GbE (ktorý nezverejňuje `*ReceiveBuffers`). Cieľové hodnoty sú obmedzené na maximálnu hodnotu hlásenú jednotlivými ovládačmi (`NumericParameterMaxValue`), takže sa nikdy nezapíše hodnota mimorozsahu. Spustite to z terminálu s **zvýšenými oprávneniami**; rovnako ako pri každom ladení založenom na registri, zmena nadobudne platnosť až po reštarte adaptéra alebo reštarte počítača. Manuálne príkazy `Set-NetAdapterAdvancedProperty` uvedené vyššie zostávajú dobrou alternatívou — uplatňujú sa za behu (opätovné priradenie adaptéra) bez reštartu.

### Základy siete (všetky prepojenia LATTICE)

- **Adresovanie:** link-local `169.254.0.0/16` (GigE Vision LLA). Hostiteľ používa statickú adresu `169.254.x.x/16`; kamery + DAQ-E si priraďujú adresy v tom istom rozsahu. Nie je potrebný DHCP ani brána.
- **Veľkosť paketu:**uprednostňujte jumbo (9000), ale nechajte to zistiť automatickou sondou — pri každom pripojení vykonáva nové meranie a prostredníctvom sondy GVSP už prekonáva 1500-bajtový limit ICMP kamery, takže sa nastaví na jumbo všade tam, kde to kábel skutočne prenáša. Nastavte pomocou `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` len vtedy, ak viete viac ako test, a uprednostňujte nastavenie na príkaz pred trvalým: toto nastavenie preskočí test, takže ak trasa nedokáže skutočne prenášať 9000**každý** zachytený paket vyprší s `SC_ERR_TIMEOUT -1011` (pozri [Premenné prostredia](#environment-variables)).
- **Veľkosť prstena RX sa mení s `ReceiveBufferLen`:**pri predvolenom nastavení `32` je využiteľný krúžok ~0,26 MB (príliš malý na akúkoľvek sériu záznamov z viacerých kamier); pri maximálnej hodnote `256` je veľký (~13,5 MB namerané na laboratórnom 10GbE hostiteľovi), čo poskytuje skutočnú rezervu. O tom, či sa konfigurácia pripojí, rozhoduje kontrola pripustnosti zohľadňujúca odber**a** súhrnná kontrolapredplatného nižšie — nie na základe priameho porovnania burstu s prstencom.

### Model fps a burstu poľa

Ako čítať panel Nastavenia poľa (a `lattice analyze-array` / `analyze_array_network` v SDK):

- **Burst sa sčítava pre každú kameru v jej skutočnom formáte pixelov.**Mono kamery**M3M**vysielajú**Mono12 (2 B/px)**;**M3C**kamery Bayer vysielajú 8- alebo 12-bitový signál (TRI032S ticho vysiela BayerRG12 aj v prípade, že je požadovaný BayerRG8). Takže snímka v plnom rozlíšení zo 4 kamier má veľkosť**~12,6 MB, ak sú všetky 8-bitové, ale ~25 MB s tromi 12-bitovými mono kamerami**. Projekcia určuje formát každej kamery na základe jej modelu (identifikačná cache), takže prenos zodpovedá tomu, čo skutočne prenáša kábel — nie je to predpoklad jednotného formátu BayerRG8.
- **Rýchlosť USB ethernetového adaptéra je obmedzená na 200 MB/s bez ohľadu na údaje na typovom štítku.** Tabuľka účinnosti, ktorá prevádza rýchlosť spojenia na trvalú hodnotu, je odvodená od PCIe; USB sieťová karta uvádza svoju *ethernetovú* rýchlosť spojenia, je však obmedzená zbernicou USB a jej ovládačom. USB 10GbE adaptér dosahoval „trvalú“ rýchlosť ~1063 MB/s — číslo, ktoré nebolo nikdy overené — a výsledné kolísanie poškodilo 6–18 % snímok, pričom stále hlásilo správnu cieľovú hodnotu fps. Sieťové karty pripojené cez USB sú teraz obmedzené na **200 MB/s** ako absolútna hodnota (obmedzením je zbernica, takže sa to nesúvisí s menovkou; USB 1 GbE adaptér dosahuje ~80 MB/s a nie je tým ovplyvnený). `wire_ceiling_source` v zázname o schopnostiach to uvádza slovne a `nic_is_usb` to označuje. Obe nastavenia je možné prepisovať pomocou `--wire-ceiling-mbps`.
- **Admittance zohľadňuje odber, nie na základe porovnania celého burstu s prsteňom.** Súčasný burst sa musí zmestiť len do *prechodného backlogu* = `max(0, Σ per-cam arrival − host drain) × emit_window`, nie do celého burstu. V štruktúre s rýchlym hostiteľom a pomalými kamerami (**PCIe**10G hostiteľ + 4× 1 GbE kamery: príchod ≈ 320 MB/s, odvod ≈ 1063 MB/s) sa hostiteľ vyprázdňuje rýchlejšie, než sa kamery napĺňajú, backlog ≈ 0, takže fullsimulácia vysielania v plnom rozlíšení**prepustí**dáta, hoci burst 25 MB presahuje kapacitu prstena 13,5 MB. Ak umiestnite tie isté štyri kamery za**USB**10GbE adaptér a odosielanie je 200 MB/s, nie 1063 — prichádzajúci tok ho predbieha a strata sa prejaví ako poškodené rámce, nie ako nižšia snímková frekvencia. Na 1 GbE hostiteľovi dolná hranica DLThr kamier 31,25 MB/s spôsobuje, že prichádzajúci tok predbehne odber → správne**blokuje** (pre *túto* triedu blokov znížte ROI alebo použite binning ≥ 2). Priepustnosť je jednou z **dvoch** kontrolných brán – druhou je nižšie uvedená kontrola agregátneho nadmerného predplatného.
- **Predpokladaná snímková frekvencia (fps) je konzervatívny strop pre sériové načítavanie.**Hostiteľský v súčasnosti sťahuje vyrovnávaciu pamäť každej kamery**sériovo**(~jedna naokno vysielania kamery), takže cyklus je ohraničený `max(readout+emit, N × emit)`, pričom vysielanie na kameru je obmedzené na**prístupový kanál**kamery (1 GbE ≈ 80 MB/s), nie na hostiteľský uplink. Pre 4-kamerové pole s plným rozlíšením s plným rozlíšením je to**~2,8 fps**, čo zodpovedá nameraným ~2,7–3,0 fps je zámerne**nezávislá od expozície**, takže v tmavých scénach môže skutočná hodnota mierne klesnúť pod hornú hranicu, keď sa expozícia predĺži. Sériové načítavanie je skutočným obmedzovačom fps; jeho paralelizácia by zvýšila hornú hranicu smerom k rýchlosti jedného vysielania.
- **Agregovaná nadmerná rezervácia je tvrdým blokátorom pripojenia.**Minimálna alokácia šírky pásma na jednu kameru je**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), takže akonáhle sa dosiahne táto minimálna hodnota, súhrnný dopyt (`per_cam × N`) môže prekročiť**strop bezpečný voči kolíziám**(`sustained × sim_emit_factor`). Praktické stropy pri plnom rozlíšení na 1 GbE:**6 kamier pri 1500 MTU, 9 pri jumbo**. Tento limit je vlastnosťou samotného kábla a spodnej hranice – je**nezávislý od veľkosti rámca**, takže**zlučovanie a menšia oblasť záujmu (ROI) nepomáhajú** (znižujú počet bajtov na *rámec*, nie počet bajtov za *sekundu* riadený GevSCPD); jedinými riešeniami je menej kamier, použitie jumbo rámcov v celom reťazci alebo rýchlejšia sieťová karta. Príznakom by bola strata paketov GVSP, nie plynulé zníženie fps, preto `analyze-array` vynuluje hodnoty dosiahnuteľných fps a vypíše `**OVER-SUBSCRIBED**`, a `array-connect` s pevne nastaveným rozlíšením **odmietne nadviazať spojenie** (inak by proces „walk-down“ zlučoval snímky do menších skupín, čo by túto triedu blokov tiež nevyriešilo). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` zníži úroveň odmietnutia na hlasné varovanie pre prácu na testovacej platforme — pozri [Premenné prostredia](#environment-variables).

### Stav poľa — ktorý subsystém stráca snímky

`GET /api/camera/array/<array_id>/capability` pripojeného poľa obsahuje aktívny
blok `health`, ktorý sa prehodnocuje v priebežnom **10-sekundovom** okne. Rozdeľuje stratu rámcov
na dve príčiny, ktoré si vyžadujú opačné riešenia, namiesto hlásenia jednej „neúplnej“
miera, ktorá nešpecifikuje ani jednu z nich:

| Pole | Čo to znamená | Ktorý subsystém |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (na sériové číslo) | Rámec **dorazil a bol štrukturálne poškodený**— strata paketov GVSP. |**Sieť**: kapacita linky, tempo, prstenec prijímača sieťovej karty, MTU |
| `never_arrived_rate_pct` (na sériové číslo) | Rámec **vôbec neprišiel**— kamera sa nespustila alebo z nej nič nevyšlo. |**Spúšťač / synchronizácia**: kábel M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Najhoršia miera neúspešnosti pre každú kameru. | — |
| `per_cam_rate_pct` | Kombinovaná miera neúspešnosti na kameru (obe príčiny spolu). | — |
| `stable_for_seconds` | Ako dlho zostala každá kamera pod 0,01 %. | — |

Pri hodnote nad 5 % backend zaznamená do protokolu riadok `[array-health <id>] WARN` s názvom rozdelenia — pri
prvom porušení, pri zmene pásma závažnosti, raz za minútu, pokiaľ stav pretrváva, a raz, keď
sa stav vyrieši. Poškodená polovica vypíše `[gvsp-corrupt <SN>]` pri prvom dosiahnutí prahu na kameru a
dôvod, potom súhrn každých 60 s. Každé vyhodnotenie sa stále zaznamenáva do súboru protokolu backendu;
počítadlá sa posúvajú pri každom vyrovnávacom pamäti bez ohľadu na to, čo sa vypíše.

Ten istý záznam uvádza číslo, na ktorom závisí celá alokácia:

| Pole | Význam |
| --- | --- |
| `wire_ceiling_mbps` | Aktuálny trvalý rozpočet šírky pásma hostiteľa, MB/s. |
| `wire_ceiling_source` | Odkiaľ pochádza toto číslo, slovami — napr.g. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` alebo `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, keď ho nastavil `--wire-ceiling-mbps` (alebo pole **Wire Budget** v grafickom rozhraní) ho nastaví. |
| `nic_is_usb` | `true` pre USB ethernetový adaptér — pozri obmedzenie 200 MB/s vyššie. |

**Význam hodnoty:** nenulová hodnota `gvsp_corrupt_rate_pct` pri hodnote `never_arrived_rate_pct` rovnej 0
znamená, že spúšťanie a synchronizácia kábla sú v poriadku a 100 % strát je na sieťovej
trase — znížte hodnotu `--wire-ceiling-mbps` a znovu pripojte. Opačný vzor poukazuje skôr na
synchronizačný kábel alebo spúšťaciu linku.

> **Hodnota `--target-fps` nie je rozhodujúcim faktorom pre poškodené rámce.** Frekvencia GevSCPD sa zapisuje
> raz pri pripojení, takže zníženie spúšťacej frekvencie mení pracovný cyklus, nie však
> rýchlosť simultánneho vysielania burstov . Zmerané zníženie požiadaviek o 5× neprinieslo žiadne zlepšenie;
> zníženie maximálnej rýchlosti z 240 na 200 MB/s znížilo u toho istého zariadenia podiel
> poškodených rámcov z 10,4 % na 0,00 %.

> **Automatické zmenšenie v priebehu prenosu nie je k dispozícii vo firmvéri TRI032S.** Spustené pole
> to nedokáže opraviť samo; odpojte a znovu pripojte, aby nástroj na výber času pripojenia mohol
> znovu naplánovať s novým limitom.

### Príznak → riešenie

| Príznak (Nastavenia poľa / pripojenie / `analyze_array_network`) | Príčina | Riešenie |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` sa resetuje na 32 (zvyčajne po aktualizácii ovládača) | Nastavte `ReceiveBufferLen`→256, `PendingReceives`→64; znovu otvorte panel (ak backend uložil do vyrovnávacej pamäte starú veľkosť prstenca, reštartujte ho) |
| Zavesenie pri reštarte/vypnutí; neskôr chyby WMI `Invalid class`, sieťová karta sa nedá aktivovať, chýbajú USB disky | Starý ovládač Realtek USB 10GbE z roku 2016 → BSOD `0x9F` → vynútené vypnutia | Aktualizujte ovládač adaptéra na verziu ≥ v10.67 (2026), potom znovu použite vyššie uvedené nastavenia prijímacej slučky |
| Pripojenie prebehne úspešne, ale vráti nižšie rozlíšenie, ako je natívne | Funkcia Smart-prep automatickyzmenšil rámec, aby sa zmestil do linky | Aktualizujte spojenie / prijmite zmenšenie / `--force-tier slip-emit-and-capture` |
| Pole hlási správnu cieľovú hodnotu fps, ale poskytuje len jej zlomok; `health.gvsp_corrupt_rate_pct` nenulová, `never_arrived_rate_pct` 0 | Odhadovaný rozpočet šírky pásma hostiteľa nadhodnocuje skutočnú kapacitu (typické pre USB ethernetový adaptér, úzku PCIe linku alebo zdieľanú sieťovú štruktúru) | Znovu sa pripojte s nižšou hodnotou `--wire-ceiling-mbps` a znovu skontrolujte blok stavu. **Nie** `--target-fps` — tempo GevSCPD je pri pripojení pevne nastavené |
| V zverejnených skupinách chýbajú kamery; `health.never_arrived_rate_pct` nie je nulová, `gvsp_corrupt_rate_pct` 0 | Spúšťacia/synchronizačná cesta — kamery sa nespúšťajú, nejde o sieťový problém | Skontrolujte synchronizačný kábel M8 a hodnotu `--line`; overte, či sú všetky kamery aktivované (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` prekročené v `analyze-array`, alebo odmietnutie pripojenia s pevne nastaveným rozlíšením (`array over-subscribes the wire`) | Celková požiadavka na jednu kameru (minimálne 8 MB/s × N kamier) prekračuje hornú hranicu bezpečnú voči kolíziám — 6 kamier v plnom rozlíšení na 1 GbE pri 1500 MTU, 9 s jumbo rámcami | Menší počet kamier, jumbo rámce v celom smere alebo rýchlejšia sieťová karta. **ROI/binning NEPOMÔŽE** (horná hranica je nezávislá od veľkosti rámca). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` prepisuje nastavenia na testovacom stole (akceptuje stratu paketov) |

---

## `chloros-cli daq`

Príkazy spektrálneho senzora. Dve triedy:
- **`pool-*`**— tenké klienti typu „HTTP“, ktorí ovládajú senzor prostredníctvom trvalého poola backendu.**Toto je podporovaná cesta a jediná, ktorá je prítomná v dodávanom balíku CLI.** Backend vlastní prenos, takže GUI, skripty CLI a SDK zdieľajú jedno aktívne identifikačné číslo namiesto toho, aby sa bili o sériový port.
- **Všetko ostatné**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — priamy prístup k hardvéru, pre úplnosť zdokumentovaný nižšie. Na ich použitie je potrebný balík `daq` Python, ktorý**nie je súčasťou žiadneho dodávaného artefaktu**: kompilovaná verzia CLI ho neobsahuje (`scripts/Build-CLI.ps1` nastavuje `--nofollow-import-to=daq` a transporty `pyserial` / `bleak` / `zeroconf` ho obsahujú), a balík PyPI SDK ho tiež neobsahuje. Fungujú iba zo zdrojového kódu, takže ich považujte skôr za internú vývojovú cestu MAPIR než za niečo, po čom by ste mali siahať.
- **`discover` / `list`** sa nachádzajú niekde uprostred: sú to priame hardvérové príkazy zo zdrojového kódu, ale v dodanej verzii sa prepnú na `pool-discover` a skenovanie vykonáva backend. Skenovanie teda funguje všade — čo je dôležité, pretože je to jediný spôsob, ako zistiť BLE MAC zariadenia DAQ-M.

> **`chloros-cli daq --help`** (a `-h` / `help`) uvádza zoznam podpríkazov `pool-*` — nápoveda je zámerne smerovaná na klienta poolu, aby odrážala príkazy, ktoré sa skutočne spúšťajú. Ak v dodanej verzii spustíte podpríkaz na priamy prístup k hardvéru, program sa ukončí s explicitnou chybou, v ktorej sa uvedie chýbajúci balík a odkáže vás späť na `pool-*`; nič sa nekončí bez upozornenia. (`discover` / `list` sú výnimkou — presmerujú na `pool-discover` a jednoducho fungujú.)
>
> **Všetko, čo zákazník potrebuje, je dostupné prostredníctvom `pool-*`** — pripojenie, streamovanie, nahrávanie kalibrovaných súborov `.daq` a výmena profilov kondenzátorov. DAQ je možné ovládať aj z Python pomocou `chloros_sdk.connect_daq_sensor()`, ktorý využíva tú istú zdieľanú cestu.

### Postup prvého pripojenia senzora DAQ

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referencia `pool-*`

| Podpríkaz | Účel |
| --- | --- |
| `daq pool-connect` (smart-detect) | Otvorenie snímača v backendovom poole. |
| `daq pool-connect --port PORT` | DAQ-U na konkrétnom sériovom porte. |
| `daq pool-connect --ble` | DAQ-M cez BLE, automatické vyhľadávanie MAC. |
| `daq pool-connect --mac MAC` | DAQ-M cez BLE na známej adrese MAC (predpokladá `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E cez Ethernet na známom hostiteľovi. |
| `daq pool-connect --eth` | DAQ-E cez Ethernet, hostiteľ automaticky zistený (mDNS + záložný režim ARP; funguje z prázdnej vyrovnávacej pamäte ARP na adresách Windows a Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Nastaviť integračný okno / stav AE. |
| `daq pool-connect --no-stream` | Pripojiť sa, ale zatiaľ nespustiť prenos (pokračovať pomocou `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil korekcie limitu. Predvolená hodnota na strane backendu je `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Prehľadávajte všetky transportné kanály a hľadajte senzory, ku ktorým by ste sa mohli pripojiť, bez toho, aby ste sa pripájali. **Takto nájdete BLE MAC adresu zariadenia DAQ-M.** `daq discover` / `daq list` sa v dodávaných zostavách automaticky smerujú sem. Senzory, ktoré sú už otvorené v skupine, sa nezobrazujú – pripojený DAQ-M prestane vysielať reklamu – preto pre ne použite `pool-list`. |
| `daq pool-list` | Zobraziť všetky senzory v backendovom poole. |
| `daq pool-disconnect --sensor-id ID [--all]` | Uvoľniť. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Najnovšienajnovších rámcov spektra N. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Obnoviť / pozastaviť streamovanie. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Spustiť / zastaviť nahrávanie .daq. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Zmena profilu korekcie kapacity počas behu. |

### Podpríkazy pre priamy prístup k hardvéru (len v zdrojovom kóde – nie sú súčasťou dodávaných zostáv)

> Uvedené pre úplnosť. Tieto podpríkazy vyžadujú balík `daq` Python a podpríkazy `pyserial` / `bleak` / `zeroconf`, z ktorých žiadny nie je súčasťou kompilovaných verzií CLI ani repozitára PyPI SDK — fungujú iba po stiahnutí zdrojového kódu z MAPIR. **Ak používate vydanú verziu Chloros, použite namiesto toho vyššie uvedené príkazy `pool-*`**; pokrývajú pripojenie, streamovanie, nahrávanie a výber kap.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Otvorenie, pripojenie a spustenie uloženého projektu Chloros (zložka obsahujúca súbory `cameras.json` + `sensors.json` + `project.json`). Všetko prechádza cez backend, takže grafické rozhranie (GUI) a príkazy v príkazovom riadku (CLI) vytvárajú identický stav hardvéru.

### Prehľad podpříkazov

| Podpríkaz | Účel |
| --- | --- |
| `project open PATH` | Vytlačiť zoznam zariadení projektu (kamery, pole, senzory). |
| `project devices PATH [--reconnect]` | Zobraziť zoznam alebo znova spustiť vyhľadávanie. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Pripojiť všetky uložené kamery / pole / senzory. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Jednorazové zachytenie z určenej kamery alebo poľa. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Séria N snímok z určenej kamery alebo skupiny (`-n/--count` predvolené 5; `-i/--interval` sekundy medzi snímkami, predvolené 0). Séria snímok zo skupiny odstraňuje duplicitné opakované synchronizované skupiny (kontrolný mechanizmus zastaralosti), takže skupina s čiastočným cyklom nemôže vrátiť N kópií jedného snímku; vypíše výsledky za každú iteráciu. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Prúd na disk prostredníctvom úlohy na pozadí. `--poll-interval` = interval v sekundách medzi dotazmi `/stats` (predvolené nastavenie 2,0). |
| `project sensor read PATH NAME [--json]` | Najnovší rámec spektra. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Zaznamenať .daq. |
| `project run PATH RECIPE.yaml` | Spustiť recept na zachytávanie YAML/JSON. `--dry-run` overuje bez spustenia. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Vypočítať zarovnanie pre pole — pozri [tabuľku príkazov nižšie](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Vypíše aktuálny profil zarovnania. |
| `project align clear PATH NAME` | Vymaže profil z vyrovnávacej pamäte. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Posun transformácie jedného podriadeného. |
| `project align export PATH NAME --to FILE` | Uloženie profilu do súboru JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Načítanie uloženého profilu. |

#### Možnosti `project align calibrate`

| Priepínač | Predvolené | Popis |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Metóda zarovnania. **Tieto názvy sa líšia od `lattice align-calibrate`**, ktoré používa skrátené formy `orb` / `akaze` / `phase`; tieto dva príkazy nie sú v prípade tohto parametra zameniteľné. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformácia modelu na prispôsobenie. |
| `--frames N` | `1` | Synchronizované snímky snímok na priemer. |
| `--reference SN` | hlavná | Sériové číslo referenčnej kamery; všetky ostatné členy sú na ňu deformované. |
| `--max-features N` | `5000` | Limit počtu charakteristík ORB. |
| `--ratio-threshold F` | `0.75` | Loweho pomer . |
| `--ransac-threshold-px F` | `3.0` | Prahová hodnota vnútorných bodov RANSAC. |
| `--min-matches N` | `15` | **Kritérium kvality** — odmietne riešenie s menším počtom zhodných vnútorných bodov, ako je tento počet. |
| `--max-reproj-err-px F` | `4.0` | **Kontrola kvality** — odmietne riešenie s chybou reprojekcie RMS vyššou ako táto hodnota. |
| `--checkerboard RxC` | — | Geometria dosky pre `--method checkerboard`, napr. `9x6`. |
| `--name PROFILE` | prázdne | Názov profilu vložený do uloženého súboru JSON. **Nie je to názov poľa** — to je pozičný `NAME`. |

Tieto dve kontroly kvality sú dôvodom, prečo môže kalibrácia úspešne vyriešiť problém, ale napriek tomu
odmietnuť uloženie: profil, ktorý nesplní aspoň jednu z nich, by bez upozornenia nesprávne zaregistroval každé
neskoršie zachytenie, preto je odmietnutý namiesto toho, aby bol uložený.

### Príklady

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL receptu

`project run RECIPE.yaml` akceptuje súbor YAML alebo JSON opisujúci postupnosť akcií:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Podporované akcie: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Akcia `burst` prijíma `name` (povinné), `count` (predvolené 5), `interval` (sekúnd, predvolené nastavenie 0), `output`, `format`, a `settings` (rovnaký tvar nastavení pre každú kameru ako `apply`); séria snímok v poli používa rovnaký strážny mechanizmus pre čerstvo synchronizovanú skupinu ako `project burst`.

Spustite to:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Premenné prostredia

| Premenná | Účinok |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Prepíše backend URL (predvolené nastavenie `http://127.0.0.1:5000`) — **rešpektované iba rodinami príkazov `lattice`, `project` a `daq pool-*`.** Základné príkazy (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) pripájajú `http://127.0.0.1:<port>` a ignorujú túto premennú (literál IPv4 obchádza Windows `localhost`→`::1` ~2 s nažiadosť), takže sa vždy zameriavajú na lokálny počítač. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` zníži úroveň odmietnutia pripojenia v dôsledku nadmerného predplatného poľa (agregovaná požiadavka na CAM &gt; strop vedenia odolný voči kolíziám s `pin_resolution`) na hlasné varovanie a pokračovanie, pričom akceptuje stratu paketov GVSP. Iba na testovacie účely — pozri [Model fps a burst pole](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Nastavuje samotný CLI; nariaďuje backendu, aby povolil paralelné spracovanie. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` preskakuje záložnú kontrolu GVSP (len výsledky ICMP). **Týmto sa vypne jumbo, nielen že sa utlmí protokol** — kamera odpovedá na DF pingy len do 1500 na každej trase, takže táto kontrola je jediná, ktorá dokáže detekovať jumbo. Ušetrí ~1 s na kameru na jedno pripojenie; stojí ~1,45× maximálnu šírku pásma, ak by sieť *mohla* prenášať jumbo. Funkcia „SDK“ vás upozorní, keď to nastavíte. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fixuje veľkosť paketu GVSP na N bajtov; úplne vynecháva testovanie. Uprednostňujte nastavenie na jednotlivé príkazy (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) pred trvalým nastavením: fixná veľkosť prestane prispôsobovať sa sieti pred ňou a fixácia hodnotu 9000 na trase, ktorá nepodporuje jumbo pakety, spôsobí, že **každé** zachytávanie vyprší s chybou `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Prepísanie adresára na extrakciu súborov Nuitka. Príkaz CLI automaticky použije `/mnt/ssd/tmp`, ak je prítomný. |

---

## Kódy ukončenia

| Kód | Význam |
| --- | --- |
| `0` | Úspech. |
| `1` | Všeobecná chyba (väčšina chýb podpríkazov). |
| `2` | Chyba argumentu. |
| `130` | Prerušené klávesovou skratkou Ctrl+C. |

---

## Tipy na riešenie problémov

- **„Vyžaduje sa prihlásenie“** → Spustite na tomto počítači raz príkaz `chloros-cli login EMAIL PASSWORD`.
- **„Backend je nedostupný“** → Spustite desktopovú aplikáciu Chloros alebo spustite binárny súbor backendu priamo (`chloros-backend`), alebo v prípade vzdialeného prístupu skontrolujte `CHLOROS_BACKEND_URL`.
- **Príkazy `lattice` zlyhávajú s chybou „LATTICE camera drivers not found“** → Nie je nainštalované runtime prostredie Arena SDK; balík CLI je dodávaný spolu s `win32api` na stránke Windows, ale runtime prostredie C je súčasťou inštalátora s grafickým rozhraním.
- **V okne „Array connect“ / „Array Settings“ sa zobrazuje hlásenie „FRAMES WILL DROP“ alebo „Reduce ROI to enable“** → Príjemný krúžok sieťovej karty hostiteľa je príliš malý (zvyčajne sa po aktualizácii ovládača sieťovej karty resetuje na hodnotu 32). Pozrite si [Nastavenie a ladenie sieťovej karty hostiteľa](#host-nic-setup--tuning-lattice-arrays) — nastavte `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Počítač sa zasekne pri reštarte/vypnutia, následne WMI `Invalid class` / sieťová karta sa nedá aktivovať / chýbajú USB disky** → Zastaraný ovládač USB 10GbE adaptéra spôsobuje chybu `DRIVER_POWER_STATE_FAILURE` (modrá obrazovka smrti `0x9F`). Aktualizujte ovládač adaptéra — pozrite [Nastavenie a ladenie sieťovej karty hostiteľa](#host-nic-setup--tuning-lattice-arrays).
- **Upozornenie na swap v Jetsone** → Pridajte swap založený na súboroch; skript CLI vypíše presné príkazy `fallocate` / `swapon`.
- **Chýbajú priame príkazy DAQ** → Očakávané: dodávaný balík `chloros-cli` zámerne vylučuje balík `daq`, takže je prítomný len `pool-*` (ani repozitár PyPI SDK ho neobsahuje). Použite balík `pool-*`, ktorý ovláda ten istý senzor prostredníctvom backendu, alebo balík `chloros_sdk.connect_daq_sensor()` z adresy Python.

---

## Pozri tiež

- [Python Referencia k SDK](sdk-reference.md) — programový ekvivalent každého príkazu CLI.
- [Príručka k snímačom DAQ](../daq/README.md) — zapojenie a kalibrácia konkrétnych snímačov.
- Online dokumentácia: `https://mapir.gitbook.io/chloros/cli`

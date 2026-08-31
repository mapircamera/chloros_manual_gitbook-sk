# CLI : Príkazový riadok

> **Kompletná referenčná príručka:**[CLI Reference](reference/cli-reference.md) dokumentuje**všetky parametre všetkých podpovelov** a je optimalizovaná pre AI asistentov — vložte jej URL do svojho asistenta a požiadajte o funkčný príkaz: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tip pre nástroje umelej inteligencie:** akákoľvek stránka tejto príručky je dostupná vo formáte surového Markdownu pridaním `.md` k jej URL (napr. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), a `https://mapir.gitbook.io/chloros/llms.txt` indexuje celú príručku pre použitie v LLM.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## Čo je toCLI


`chloros-cli` je rozhranie príkazového riadku pre ten istý spracovateľský engine, ktorý používa desktopová aplikáciaChloros
. Ide o tenkého klienta typu „HTTP
“ nad backendomChloros
(lokálny server na `127.0.0.1:5000`) — väčšina príkazov spustí backend automaticky, takže skript potrebuje len jedno volanie `chloros-cli process …`.

Beží na **Windows
10/11 (x64)**a**Linux
(x86_64 a NVIDIA Jetson arm64 na JetPack 6)**, v ľubovoľnom termináli, bez potreby grafického rozhrania. Overte si inštaláciu pomocou:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Prehľad skupín príkazov:

* **Spracovanie a účet** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 jazykov — pozri [Podporované jazyky](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (len preLinux
/Jetson)
* **Hardvér v reálnom čase** — `lattice` (ovládanie kamery LATTICE, viac ako 45 podpríkazov), `daq pool-*` (svetelné senzory DAQ), `time-sync` (PTP)
* **Automatizácia** — `project` (spustenie uloženého projektuChloros
bez grafického rozhrania, vrátane receptov na zachytávanie vo formáte YAML)

Globálne možnosti, ktoré stojí za to poznať: `--port N` (port backendu, predvolené nastavenie `5000`), `-v/--verbose`, `--restart` (vynútené reštartovanie backendu), `--backend-exe PATH`. Úplný zoznam nájdete v [ReferenciiCLI
](reference/cli-reference.md).

***

## Inštalácia

CLI
**je súčasťou inštalátoraChloros** na všetkých platformách — neexistuje žiadne samostatné stiahnutieCLI
. Inštalátor si stiahnite zo stránky [Stiahnutie](download.md).

###Windows


Inštalátor umiestni súborCLI
do adresára:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

a pridá tento priečinok do vášho systému `PATH` — po inštalácii **otvorte nový terminál**, aby sa načítal aktualizovaný súbor `PATH`. Inštalátor tiež umiestni spúšťacie skripty (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) do koreňového adresára inštalácie a vytvorí**skratku v ponuke Štart (ChlorosCLI
)** skratku v ponuke Štart, z ktorých každá otvorí terminál s programom `chloros-cli` pripraveným na použitie.

###Linux


Nainštalujte verziu `.deb` pre vašu architektúru:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Týmto sa nainštaluje `chloros-cli` až `/usr/bin/chloros-cli` (už na verzii `PATH`) a backend na verziu `/usr/lib/chloros/chloros-backend`, spolu s runtime prostredím ArenaSDK
potrebným pre kamery LATTICE. Podrobnosti nájdete v [InštaláciiLinux
](linux/linux-installation.md).

### Overenie

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Prihlásenie a licencovanie

CLI
(aPython
SDK
) vyžaduje **platený plánChloros
+**— je k dispozícii v každej platenej úrovni; bezplatná úroveň ho neposkytuje. Toto obmedzenie je vynucované**na strane servera** prostredníctvom backendu, nie binárnym súboromCLI
: volanie bez prihlásenia je odmietnuté s kódom chyby `401 AUTH_REQUIRED`, a prihlásené volanie v bezplatnej verzii s chybou `403 PLAN_UPGRADE_REQUIRED`, bez ohľadu na to, či pochádza z `chloros-cli`,SDK
alebo z vlastného klientaHTTP
. Aktualizujte na [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Prihláste sa **raz na každom zariadení**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Heslá so špeciálnymi znakmi**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` je shellom skreslené (CLI
to zistí na základe chyby 401 a automaticky to skúša znova, ale použitie jednoduchých úvodzoviek tento problém úplne vyrieši).
{% endhint %}

Relácia je uložená v cache pod názvom `~/.chloros/user_session.json` a funguje offline počas tolerančného obdobia daného plánu (30 dní pri mesačných plánoch, do vypršania platnosti pri ročných plánoch). `chloros-cli status` funguje aj bez plateného plánu, takže dôvod odmietnutia je vždy viditeľný.

{% hint style="danger" %}
**Plánujete úlohy bez grafického rozhrania? Najprv sa prihláste.**Príkazy na spustenie backendu (`process`, `status`, `export-status`, …) spustený**bez uloženej relácie v cache**neskončí rýchlo s chybou — prejde do interaktívneho riadku príkazov `Email:` / `Password:` na stdin. Automatizovaná úloha cron alebo krok CI sa preto**zasekne a bude čakať na vstup**. Pred naplánovaním akejkoľvek úlohy najprv raz spustite `chloros-cli login EMAIL 'PASSWORD'` na danom počítači.
{% endhint %}

***

## Vaše prvé spustenie spracovania

Nasmerujte `process` na priečinok so zachytenými údajmi — automaticky detekujeSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` alebo ich kombináciu:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Priebeh sa zobrazuje v reálnom čase pre každé vlákno potrubia (detekcia, analýza, spracovanie, export) a úspešné spustenie končí hlásením počtu zapísaných obrazových produktov (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Kam sa ukladajú výstupy

`process` zapisuje do **priečinka projektu**, nie do vášho vstupného priečinka:

* Ak nie je zadané `-o`: projekt sa vytvorí vo vašej predvolenej projektovej zložke (zdieľanej s grafickým rozhraním; spravujte ju pomocou `get-project-folder` / `set-project-folder`, záložný `~/Chloros Projects`), pomenovaný podľa `-n/--project-name` alebo časovej pečiatky (`YYYYMMDD_HHMMSS`), ak je vynechaný.
* S `-o PATH`: tento priečinok **je** priečinkom projektu. Ak už obsahuje súbor `project.json`, namiesto prepísania sa vytvorí súbor s príponou `_1`/`_2`…

V rámci projektu sú produkty zoskupené **podľa fotoaparátu a potom podľa formátu súboru**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Zložka fotoaparátu je `LATT-<sensor>-<lens>-F<filter>` pre LATTICE (zodpovedá EXIF záznamu `Model`) a `<model>_<filter>` (napr. `Survey3N_RGN`) preSurvey3
. Zložka formátu nasleduje po `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` alebo `tiff32` pre `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Každý exportovaný produkt si zachováva názov zdrojového súboru.**Export radiancie súboru `capture_..._raw.tif` sa stále nazýva `capture_..._raw.tif` — len sa nachádza v adresári `tiff32/Radiance_Images/`.**Produkt identifikuje zložka, nie názov súboru**, preto používajte globálny výraz pre zložku, nie pre príponu `*radiance*`.
{% endhint %}

### Možnosti, ktoré skutočne použijete

| Príznak | Predvolené nastavenie | Čo robí |
| --- | --- | --- |
| `-o, --output PATH` | predvolený priečinok projektu | Umiestnenie priečinka projektu (pozri vyššie). |
| `-n, --project-name NAME` | časová pečiatka | Názov projektu. |
| `--format FMT` | `TIFF (16-bit)` | Jedna z možností: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | žiadne | Vegetácia indexy na export (pozri [Vegetácia indexy](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neurálny debayer, pomalší, najvyššia kvalita (Chloros
+, grafická karta NVIDIA). |
| `--vignette / --no-vignette` | zapnuté | Korekcia vinety. |
| `--reflectance / --no-reflectance` | zapnuté | Kalibrácia odrazivosti; pre LATTICE slúži aj na zapnutie/vypnutie produktu odrazivosti. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Vynútiť vstupný bod spracovateľského reťazca pre súbory LATTICE TIFF. |

Všetko ostatné — ladenie detekcie cieľa, PPK, expozičné body, príznaky zarovnania poľa — nájdete v [časti `process` referenčnej príručkyCLI
](reference/cli-reference.md).

***

## Výber toho, čo exportovať (produkty LATTICE)

Spracovanie LATTICE sa rozvetvuje na **všetky príslušné produkty v jednom cykle**. Štyri prepínače pre každý produkt sú**štandardne zapnuté**; formulár `--no-` použite na vypnutie jedného z nich:

| Prepínač | Produkt |
| --- | --- |
| `--debayered` | Lineárna demosaika → `Debayered_Images/` |
| `--preview` | Náhľad na displeji (vyváženie bielej + gama; rozťahovanie falošných farieb pre multispektrálne snímky) → `Preview_Images/` |
| `--radiance` | žiarivosť typu float32, W/m²/sr/nm → `Radiance_Images/` (vždy `tiff32/`) |
| `--reflectance` | uint16 odrazivosť, pripravené pre Pix4D → `Reflectance_Calibrated_Images/` |

RGB
hlavné kamery vždy vysielajú iba údaje po odstránení Bayerovho vzoru + náhľad — žiarivosť/odrazivosť na jednotlivé pásma nemá pre širokopásmový senzor zmysel, preto sú tieto prepínače pre ne bez účinku.Survey3
`.raw` ignoruje prepínače a riadi sa štandardnou cestou odrazivosti/cieľa.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (predvolené nastavenie `auto`) vyberá referenčnú hodnotu odrazivosti: `auto` vytvorí [kalibračný cieľ](calibration-targets.md) v rámci záberu, ktorý spĺňa požiadavky kontroly kvality ako absolútnu referenciu a v prípade, že nie je prítomný žiadny cieľ, prejde na rozdelenie dopadajúceho svetla svetelného senzora DAQ (ρ = π·L/E); `target` je prísny (bez nahradenia DAQ); `daq` sa riadi údajmi z DAQ. Skeny meraných cieľov v jednotkách na jednotku možno poskytnúť pomocou `--target-reflectance-dir`.

{% hint style="info" %}
**Čítanie pixelov odrazivosti:**hodnota DN znamenajúca ρ = 1,0 je**na zdroj** — Súbory LATTICE vkladajú do XMP značku `Chloros:PixelScale=32768`; súborySurvey3
používajú hodnotu 65535 (a neobsahujú žiadne značky `Chloros:*`). Prečítajte značku a vydelte ňou, namiesto toho, aby ste predpokladali konštantu. Podrobnosti a jeden zámerný okrajový prípad bez mierky sú uvedené v [CLI
Referencii](reference/cli-reference.md).
{% endhint %}

**Spracovanie sa vždy začína od `raw`.** Odvodené produkty (exporty bez debayeringu/žiarivosti/odrazivosti) sa nikdy nevracajú späť do spracovateľského reťazca – ich opätovný import a spracovanie by znamenalo dvojité uplatnenie kalibračných výpočtov, preto ichChloros
preskočí a oznámi to. `--input-level` je zámerný únikový východ pre prípady, keď skutočne potrebujete vynútiť vstupný bod.

***

## Keď spracovanie zlyhá

Od verzie 1.2.0 program `process` jasne signalizuje zlyhanie namiesto toho, aby „úspešne“ skončil bez zobrazenia výsledkov:

* Spustenie, ktoré **požadovalo produkty, ale žiadny nezapísalo**— iba `project.json` a `calibration_data.json` — vypíše `Processing finished but wrote no image products.` a**ukončí sa s nenulovým kódom**, takže skripty to dokážu zistiť. Bežné príčiny: vstupná zložka nebola rozpoznaná ako záznam (skontrolujte rozloženie a `--input-level`), alebo žiadny z požadovaných produktov nebol pre dané kamery použiteľný (napr. požiadavka na radianciu/odrazivosť z kamier, ktoré podporujú ibaRGB
).
* **Úmyselné spustenie len s metadátami** (všetky produkty vypnuté, bez `--indices`) je stále úspešné — prázdny výstupný obrázok je v tomto prípade správnym výsledkom.
* Spustite proces znovu s parametrom `--verbose` a skontrolujte protokol backendu, či sa v ňom nachádzajú riadky s kódmi `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, ktoré vysvetľujú vynechania jednotlivých kamier.

Kódy ukončenia: `0` úspech · `1` všeobecná chyba · `2` chyba argumentu · `130` prerušené klávesovou skratkou Ctrl+C.

***

## Indexy vegetácie

Spustite `--indices` s jedným alebo viacerými názvami predvolieb; každý index sa uloží do vlastnej zložky `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

22 prednastavených mien, ktoré `process --indices` akceptuje:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Existujú tri zoznamy indexov — nezamieňajte ich.**V roletovom menu „Nastavenia projektu“ v grafickom rozhraní je 27 vzorcov (pridáva sa `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` – týchto päť je určených len pre grafické rozhranie a**neplatia** pre `--indices`). Príkaz live/offline `lattice index --preset` používa vlastný samostatný zoznam s 22 prednastaveniami. Vzorce a výpočty v pásmach sú zdokumentované v [Vzorce multispektrálnych indexov](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Svetelné senzory DAQ: Stručný prehľad

Rodina `daq pool-*` ovláda spektrálne senzory DAQ typu „MAPIR
“ (DAQ-U cez USB, DAQ-M cez BLE, DAQ-E cez Ethernet) prostredníctvom trvalého poola backendu — grafické rozhranie (GUI),CLI
aSDK
zdieľajú jeden aktívny identifikátor. **`pool-*` je podporovaná cesta DAQ v dodávanom balíkuCLI
**; ostatné podpríkazy typu `daq`, na ktoré sa môžete stretnúť, sú iba interným zdrojomMAPIR
a ukončia sa s explicitnou chybou, ktorá vás nasmeruje na `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` bez `--duration` beží až do `pool-record --stop`; predvolený výstupný adresár je `~/Documents/DAQ Live View/` **na počítači backendu**. Profil korekcie kapacity sa volí v čase pripojenia (`--cap-id`, predvolený pre backend `sunshine_cosine`) a je možné ho za behu zmeniť pomocou `pool-set-cap` — profily obmedzenia a kalibrovaný rozsah senzora sú popísané v kapitolách o DAQ v tejto príručke.

{% hint style="warning" %}
**DAQ-E na hostiteľskom počítači s viacerými sieťovými kartami:** prvé automatické vyhľadávanie `pool-connect --eth` po spustení systému môže zlyhať aj pri funkčnom senzore. `--eth-host <ip-or-hostname>` je spoľahlivá alternatíva — použite ju vždy, keď vyhľadávanie neprinesie žiadne výsledky.
{% endhint %}

***

## Kamery LATTICE, PTP a automatizácia projektov

Rodina príkazov `lattice` (viac ako 45 podpríkazov) pokrýva prácu s kamerami LATTICE od začiatku do konca: vyhľadávanie, jednotlivé snímky, trvalé synchronizované polia s tokom pripojenia „smart-prep“ v grafickom rozhraní, živý náhľad v prehliadači, zarovnanie, výpočty indexov a diagnostika sieťových kariet hostiteľa. Ukážka:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Spolu s tým: `chloros-cli time-sync` podáva správy o PTP grandmasterovi, ktorý beží na hostiteľskom počítači typu „Chloros
“ (kamery LATTICE a senzory DAQ-E sú k nemu pripojené ako slave zariadenia na účely časových pečiatok medzi zariadeniami), a `chloros-cli project` otvára uložený projektChloros
a bez grafického rozhrania ovláda jeho kamery, polia a senzory — vrátane skriptovaných postupov snímania v YAML.

Tieto tri rodiny (`lattice`, `project`, `daq pool-*`) sú zároveň jediné, ktoré podporujú príkaz `CHLOROS_BACKEND_URL` na ovládanie **vzdialeného** backendu; základné príkazy sú vždy zamerané na lokálny počítač.

Úplné návody nájdete v kapitolách venovaných LATTICE v tejto príručke; všetky parametre sú uvedené v [ReferenciiCLI
](reference/cli-reference.md).

***

## Riešenie problémov: Top 5

| Príznak | Riešenie |
| --- | --- |
| `Login required` alebo naplánovaná úloha sa zasekne na výzve `Email:` | Spustite na tomto počítači raz `chloros-cli login EMAIL 'PASSWORD'` — príkazy bez uloženého relácie sa budú spúšťať interaktívne namiesto toho, aby sa rýchlo skončili neúspechom. |
| `backend unreachable` | Spustite desktopovú aplikáciuChloros
alebo priamo spustite binárny súbor backendu (`chloros-backend`). Ak nasmerujete `lattice`/`project`/`daq pool-*` na vzdialený backend, skontrolujte `CHLOROS_BACKEND_URL`. |
| Pripojenie k poľu je blokované: `FRAMES WILL DROP` / `Reduce ROI to enable` | Príjemný prstenec sieťovej karty hostiteľa bol resetovaný na predvolené nastavenia — najčastejšia príčina toho, že zariadenie, ktoré predtým fungovalo, odmieta pripojenie, zvyčajne po aktualizácii ovládača sieťovej karty. Spustite príkaz `chloros-cli lattice network --fix` z terminálu s **zvýšenými oprávneniami** (alebo nastavte `ReceiveBufferLen=256`, `PendingReceives=64`); pozrite si časť *Nastavenie a ladenie sieťovej karty hostiteľa* v referenčnej príručke. |
| Podpríkaz `daq` sa ukončí s hlásením: „vyžaduje kompletný balík daq…“ | Očakávané v dodávaných zostavách — skompilovaný balíkCLI
obsahuje iba rodinu príkazov `daq pool-*`, ktorá pokrýva pripojenie, prenos, záznam a výber kap. Použite `pool-*` (alebo `chloros_sdk.connect_daq_sensor()` zPython
). |
| Jetson zobrazuje varovanie o výmene pred spracovaním veľkých zložiek | Pridajte výmenu na báze súborov — súborCLI
vytlačí presné príkazy `fallocate`/`swapon`, ktoré sa majú spustiť. |

***

## Pomoc

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Všetky príznaky, všetky podpríkazy:** [CLI
Referencia](reference/cli-reference.md)
* **Ekvivalent vPython
:** [Python
SDK
](api-python-sdk.md) a [SDK
Referencia](reference/sdk-reference.md)
* **Podpora:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)

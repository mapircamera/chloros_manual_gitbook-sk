# CLI: Príkazový riadok

> **Kompletná príručka:**[Príručka k CLI](reference/cli-reference.md) dokumentuje**všetky parametre všetkých podpríkazov** a je optimalizovaná pre AI asistentov — vložte jej URL do svojho asistenta a požiadajte o funkčný príkaz: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tip pre nástroje umelej inteligencie:** akákoľvek stránka tejto príručky je dostupná vo formáte surového Markdownu pridaním `.md` k jej URL-u (napr. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), a `https://mapir.gitbook.io/chloros/llms.txt` indexuje celú príručku pre použitie v LLM.

> **Chcete, aby váš asistent spúšťal samotný Chloros?** `chloros-mcp`, server MCP nainštalovaný spolu s Python SDK, funguje s aplikáciami Claude Desktop, Claude Code, Cursor a ďalšími klientmi MCP: pozrite si [Používanie Chloros s AI asistentmi](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Čo je to CLI


`chloros-cli` je rozhranie príkazového riadku pre ten istý spracovávací engine, ktorý používa desktopová aplikácia Chloros
. Je to jednoduchý HTTP
-klient nad backendom Chloros
(lokálny server na `127.0.0.1:5000`) — väčšina príkazov spustí backend automaticky, takže skript potrebuje len jedno volanie `chloros-cli process …`. Backend potom zostane spustený pre nasledujúci príkaz: zastaví sa pri `chloros-cli logout` (pokiaľ nie je aplikácia otvorená alebo prebieha spustenie či kompilácia), pri `chloros-cli backend stop` alebo sám od seba po 30 minútach bez klienta, spustenia alebo pripojeného zariadenia. `chloros-cli backend status` to ukazuje.

Beží na **Windows
10/11 (x64)**a**Linux
(x86_64 a NVIDIA Jetson arm64 na JetPack 6)**, v akomkoľvek termináli, bez potreby grafického rozhrania. Overte si inštaláciu pomocou:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

Prehľad skupín príkazov:

* **Spracovanie a účet** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 jazykov — pozri [Podporované jazyky](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (len Linux
/Jetson)
* **Hardvér v prevádzke** — `lattice` (ovládanie kamery LATTICE, viac ako 45 podpríkazov), `daq pool-*` (svetelné senzory DAQ), `time-sync` (PTP)
* **Automatizácia** — `project` (spustenie uloženého projektu Chloros
bez grafického rozhrania, vrátane receptov na zachytávanie v YAML)

Globálne možnosti, ktoré stojí za to poznať: `--port N` (port backendu, predvolené nastavenie `5000`), `-v/--verbose`, `--restart` (vynútený reštart backendu), `--backend-exe PATH`. Úplný zoznam nájdete v [Referencii CLI
](reference/cli-reference.md).

***

## Inštalácia

CLI
**je súčasťou inštalačného programu Chloros** na všetkých platformách — nie je k dispozícii samostatné stiahnutie CLI
. Inštalačný program si stiahnite zo stránky [Stiahnutie](download.md).

### Windows


Inštalačný program umiestni CLI
do:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

a tento priečinok raz pridá do vášho systému `PATH` (aktualizácia tiež odstráni duplicitné položky zanechané predchádzajúcimi inštalátormi) — po inštalácii **otvorte nový terminál**, aby sa načíta aktualizovaný `PATH`. Inštalátor tiež umiestni spúšťacie skripty (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) do koreňového adresára inštalácie a vytvorí skratku**Chloros
CLI
** v ponuke Štart, z ktorých každá otvorí terminál s programom `chloros-cli` pripraveným na použitie.

### Linux


Nainštalujte `.deb` pre vašu architektúru pomocou `apt` (nie `dpkg -i`, ktorý vynecháva odporúčané požiadavky Python
SDK
):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Týmto sa nainštaluje `chloros-cli` až `/usr/bin/chloros-cli` (už na verzii `PATH`), backend a programy CLI
rozbalené v `/usr/lib/chloros/backend/` a `/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` je spúšťač, ktorý spúšťa backend), runtime prostredie Arena SDK
, ktoré backend potrebuje pre kamery LATTICE, a Python
SDK
pre systém `python3`. Podrobnosti nájdete v časti [Inštalácia v systéme Linux
](linux/linux-installation.md).

### Overenie

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Prihlásenie a licencovanie

Prístup k CLI
(a Python
SDK
) vyžaduje **platený plán Chloros
+**— ten je k dispozícii v každej platenej úrovni; bezplatná úroveň ho neposkytuje. Toto obmedzenie je vynucované**na strane servera** backendom, nie binárnym súborom CLI
: prihlásené volanie je odmietnuté s kódom `401 AUTH_REQUIRED`, a prihlásené volanie v bezplatnej úrovni s kódom `403 PLAN_UPGRADE_REQUIRED`, bez ohľadu na to, či pochádza z `chloros-cli`, SDK
alebo vlastného HTTP
klienta. Upgradujte na [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Prihláste sa **raz na každom zariadení**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Heslá so špeciálnymi znakmi**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` je shellom skreslené (CLI
to zistí na základe chyby 401 a automaticky to skúsi znova, ale použitím jednoduchých úvodzoviek sa tomuto problému úplne vyhnete).
{% endhint %}

Relácia je uložená v cache pod `~/.chloros/user_session.json` a funguje offline počas tolerančného obdobia plánu (30 dní pri mesačných plánoch, do vypršania platnosti pri ročných). `chloros-cli status` funguje aj bez plateného plánu, takže dôvod odmietnutia je vždy viditeľný.

{% hint style="danger" %}
**Plánujete úlohy bez grafického rozhrania? Najprv sa prihláste.**Príkazy na spustenie backendu (`process`, `status`, `export-status`, …) spustený**bez uloženého relácie v cache**nekončí rýchlou chybou — prejde do interaktívneho riadku `Email:` / `Password:` na stdin. Automatizovaná úloha cron alebo krok CI sa preto**zasekne a bude čakať na vstup**. Pred naplánovaním akejkoľvek úlohy spustite na počítači raz príkaz `chloros-cli login EMAIL 'PASSWORD'`.
{% endhint %}

***

## Vaše prvé spustenie spracovania

Nasmerujte `process` na priečinok so záznamami — automaticky detekuje Survey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` alebo ich kombináciu:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Priebeh sa prenáša v reálnom čase pre každé vlákno potrubia (detekcia, analýza, spracovanie, export) a úspešné spustenie končí hlásením počtu vytvorených obrazových produktov: `Image products written: N`, po ktorom nasleduje `(plus M clamp masks)`, ak sa počas behu zapísali aj [clamp masky](output-image-formats.md#clamp-masks) (tie sa nezapočítavajú ako produkty).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Ak už beží backend Chloros
(napríklad ten z desktopovej aplikácie), `process` ho použije a otvorí v ňom svoj projekt. Ak je aplikácia otvorená na projekte, ktorý otvorila, proces z aplikácie odmietne, alebo tento projekt zatvorí. Ak sa predchádzajúce spustenie ešte dokončuje, počká naň až 15 sekúnd, než odmietne. `--restart` vynúti spustenie nového backendu. V systéme Linux
sa backend, ktorý beží ako služba systemd, nikdy nerestartuje: `process` ho používa tak, ako je, a tam, kde by ho inak reštartoval (`--restart` alebo import, ktorý nenašiel nič), vypíše `sudo systemctl restart <unit>` a ukončí sa `1`.

### Kam sa ukladajú výstupy

`process` zapisuje do **projektovej zložky**, nie do vašej vstupnej zložky:

* Ak nie je nastavený `-o`: projekt sa vytvorí vo vašej predvolenej projektovej zložke (zdieľanej s grafickým rozhraním; spravujte ju pomocou `get-project-folder` / `set-project-folder`; ak nie je nastavený žiadny, `Documents/MAPIR/Chloros Projects` vo vašom užívateľskom priečinku), pomenovaný podľa `-n/--project-name` alebo časovej pečiatky (`YYYYMMDD_HHMMSS`), ak je vynechaný.
* V prípade `-o PATH`: tento priečinok **je** priečinkom projektu. Ak už obsahuje súbor `project.json`, namiesto prepísania sa vytvorí súbor s príponou `_1`/`_2`… namiesto prepísania. Relatívna cesta `PATH` sa berie vzhľadom na zložku, z ktorej spúšťate `chloros-cli`.

{% hint style="info" %}
**Relatívne cesty (zmenené vo verzii 1.3.0).** Každá cesta, ktorú backend otvorí — `-o`/`--output` pri akomkoľvek príkaze (vrátane predvoleného `output` pre príkazy zachytávania), `--input`, `--burst-dir`, `--profile`, `--save` a `--mask` — sa vyriešia vo vzťahu k priečinku, z ktorého spúšťate `chloros-cli`, takže `-o captures/` zapisuje do `<current folder>/captures/`. Vo verzii 1.2.0 ich backend riešil vo vlastnej inštalačnej zložke, čo skončilo chybou „Prístup je zamietnutý“. V prípade vzdialeného backendu (`CHLOROS_BACKEND_URL`) sa cesty odosielajú tak, ako boli zadané, a riešia sa na danom počítači.
{% endhint %}

{% hint style="warning" %}
**Windows
: zložku projektu udržiavajte krátku, pokiaľ nie sú zapnuté dlhé cesty.** Ak sú vo Windows
vypnuté dlhé cesty (predvolené nastavenie Windows
), žiadna cesta k súboru nesmie byť dlhšia ako 260 znakov a produkty sa nachádzajú až približne 130 znakov pod zložkou projektu. `process` to skontroluje pred spustením: príliš hlboký priečinok projektu je odmietnutý s hlásením **„Priečinok projektu je pre tento počítač príliš hlboký“**, najdlhší priečinok projektu, ktorý sa zmestí, a pokyny na zapnutie dlhých ciest — nič sa nespracuje. Použite kratší `-o` (alebo `set-project-folder`) alebo zapnite dlhé cesty v systéme Windows
(voľba inštalátora **Povoliť dlhé cesty v systéme Windows**, ktorá je štandardne zaškrtnutá) a reštartujte program Chloros
.
{% endhint %}

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

Zložka fotoaparátu je `LATT-<sensor>-<lens>-F<filter>` pre LATTICE (zodpovedá EXIF záznamu `Model`) a `<model>_<filter>` (napr. `Survey3N_RGN`) pre Survey3
. Zložka formátu nadväzuje na `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` alebo `tiff32` pre `TIFF (32-bit, Percent)`.

V prípade multispektrálnej kamery M3C sa v priečinku formátu môže vedľa priečinkov produktov objaviť aj priečinok `clamp_masks/` (napríklad `tiff16/clamp_masks/Reflectance_Calibrated_Images/` a `tiff32/clamp_masks/Radiance_Images/`). Obsahuje masky žiarivosti a odrazivosti na úrovni jednotlivých pixelov, v ktorých proces oddelenia pásiem obmedzil záporné hodnoty na nulu — pozri [Masky obmedzenia](output-image-formats.md#clamp-masks). Žiadna zložka s produktmi nikdy neobsahuje masku.

{% hint style="info" %}
**Každý exportovaný produkt si zachováva názov zdrojového súboru.**Export žiarivosti z `capture_..._raw.tif` sa stále nazýva `capture_..._raw.tif` — len sa nachádza v `tiff32/Radiance_Images/`.**Produkt identifikuje zložka, nie názov súboru**, preto používajte globálny výraz pre adresár, nie pre príponu `*radiance*`.
{% endhint %}

### Možnosti, ktoré skutočne použijete

| Príznak | Predvolené | Čo robí |
| --- | --- | --- |
| `-o, --output PATH` | predvolený priečinok projektu | Umiestnenie priečinka projektu (pozri vyššie). |
| `-n, --project-name NAME` | časová pečiatka | Názov projektu. |
| `--format FMT` | `TIFF (16-bit)` | Jedna z možností: `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros
nevykonáva zväčšovanie: pri všetkých 8-bitových snímkach sa hlbší formát zapisuje ako `TIFF (8-bit)` a uvádza sa to len raz. |
| `--indices NAME [NAME ...]` | žiadne | Vegetácia indexy na export (pozri [Vegetácia indexy](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neurálny debayer, pomalší, najvyššia kvalita (Chloros
+, vyžaduje grafickú kartu s podporou CUDA: bez nej sa používa `standard`; odmietnuté pre snímky LATTICE, ktoré vždy používajú `standard`). |
| `--vignette / --no-vignette` | zapnuté | Korekcia vinety. |
| `--reflectance / --no-reflectance` | zapnuté | Kalibrácia odrazivosti; pre LATTICE slúži aj ako prepínač produktu odrazivosti. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Vynútiť vstupný bod spracovateľského reťazca pre súbory TIFF z LATTICE. |

Všetko ostatné — ladenie detekcie cieľov, PPK, expozičné body, príznaky zarovnania polí — nájdete v [sekcii `process` v referenčnej príručke CLI
](reference/cli-reference.md).

***

## Výber toho, čo sa má exportovať (produkty LATTICE)

Spracovanie LATTICE sa rozvetvuje na **všetky príslušné produkty v jednom prechode**. Štyri prepínače pre každý produkt sú**štandardne zapnuté**; na vypnutie jedného z nich použite formulár `--no-`:

| Prepínač | Produkt |
| --- | --- |
| `--debayered` | Lineárna demosaika → `Debayered_Images/`: odstránenie tmavého posunu z kalibračného balíka, potom demosaik s ohľadom na okraje, bez korekcie linearity, vyrovnania poľa alebo oddelenia kanálov — ten istý súbor, aký zachytáva na úrovni `debayered`. Zachovaná pôvodná orientácia snímača (zaznamenaná v XMP). |
| `--preview` | Náhľad na displeji (vyváženie bielej + gama; rozloženie falošných farieb pre multispektrálne snímky) → `Preview_Images/` |
| `--radiance` | radiancia typu float32, W/m²/sr/nm → `Radiance_Images/` (vždy `tiff32/`) |
| `--reflectance` | uint16 odrazivosť, pripravené pre Pix4D → `Reflectance_Calibrated_Images/` |

Hlavné kamery RGB
vždy vysielajú iba údaje po odstránení Bayerovho filtra + náhľad — žiarivosť/odrazivosť na jednotlivé pásma nemá pre širokopásmový senzor zmysel, takže tieto prepínače sú pre ne bezvýznamné. Survey3
`.raw` ignoruje prepínače a riadi sa štandardnou cestou odrazivosti/cieľa.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (predvolené nastavenie `auto`) vyberá referenčnú hodnotu odrazivosti: `auto` stanovuje [kalibračný cieľ](calibration-targets.md) v rámci snímky, ktorý prešiel kontrolou kvality (QA), za absolútnu referenciu a v prípade, že nie je prítomný žiadny cieľ, prejde na rozdelenie svetla smerujúceho nadol zo svetelného senzora DAQ (ρ = π·L/E), ak nie je prítomný žiadny cieľ (cieľ, ktorý prešiel kontrolou kvality s označením, súťaží s DAQ o nameraný výsledok); `target` je prísny (bez nahradenia DAQ); `daq` je autoritatívny voči DAQ. Skeny cieľa merané na jednotku môžu byť dodané s `--target-reflectance-dir`.

{% hint style="info" %}
**Čítanie pixelov odrazivosti:**hodnota DN, ktorá znamená ρ = 1,0, je**na zdroj** — súbory LATTICE vkladajú do XMP značku `Chloros:PixelScale=32768`; súbory Survey3
používajú hodnotu 65535 a neobsahujú značku `Chloros:PixelScale` (obsahujú však iné značky spracovania typu `Chloros:*`). Prečítajte značku a vydelte ňou, namiesto toho, aby ste predpokladali konštantu. Podrobnosti a jeden zámerný okrajový prípad bez mierky sú uvedené v [Referencii CLI
](reference/cli-reference.md).
{% endhint %}

**Spracovanie sa vždy začína od `raw`.** Odvodené produkty (exporty bez debayeringu/žiarivosti/odrazivosti) sa nikdy nevracajú späť do spracovateľského reťazca — ich opätovný import a spracovanie by znamenalo dvojité uplatnenie kalibračných výpočtov, preto ich Chloros
preskočí a upozorní na to. `--input-level` je zámerný únikový východ, ak skutočne potrebujete vynútiť vstupný bod.

***

## Keď spracovanie zlyhá

Od verzie 1.2.0 sa `process` jasne hlási ako zlyhanie namiesto toho, aby „úspešne“ skončil bez zobrazenia výsledkov:

* Spustenie, ktoré **požadovalo produkty, ale žiadny nezapísalo**— iba `project.json` a `calibration_data.json` — vypíše `Processing finished but wrote no image products.` a**ukončí sa s nenulovým kódom**, takže skripty to dokážu zistiť. Bežné príčiny: vstupná zložka nebola rozpoznaná ako záznam (skontrolujte rozloženie a `--input-level`), alebo žiadny z požadovaných produktov nebol pre dané kamery použiteľný (napr. požiadavka na radianciu/odrazivosť z kamier, ktoré podporujú iba RGB
).
* **Úmyselné spustenie iba s metadátami** (všetky produkty vypnuté, bez `--indices`) sa stále považuje za úspešné — prázdny výstupný obrázok je v tomto prípade správnym výsledkom.
* Spustite späť s `--verbose` a skontrolujte protokol backendu, či sa v ňom nachádzajú riadky `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, ktoré vysvetľujú vynechania jednotlivých kamier.
* V systéme Windows
, kde sú vypnuté dlhé cesty, bude projektová zložka, ktorá je príliš hlboká na to, aby sa zmestila do limitu 260 znakov, odmietnutá **ešte pred** začatím spracovania, pričom sa použije najdlhšia projektová zložka, ktorá sa zmestí (pozri [Kam sa ukladajú výstupy](#where-the-outputs-land)).
* Príkazy na zachytávanie (`lattice capture`, `lattice array-capture` a ostatné skupiny príkazov na zachytávanie) vymenujú každý súbor, ktorý sa nepodarilo zapísať, a ukončia sa s kódom `1`; opakované zachytávanie sa zastaví, keď je disk plný.

Kódy ukončenia: `0` úspech · `1` všeobecná chyba · `2` chyba argumentu · `130` prerušené klávesovou skratkou Ctrl+C.

***

## Vegetácia – indexy

Spustite `--indices` s jedným alebo viacerými prednastavenými názvami; každý index sa uloží do vlastnej zložky `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

22 prednastavených mien, ktoré `process --indices` akceptuje:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Do zoznamu sa pridá názov, ktorý v projekte ešte nie je uvedený. Názov, ktorý nie je prednastavený alebo ktorý nie je možné vytvoriť pomocou pásiem kamier, sa preskočí a zobrazí sa varovanie s jeho názvom. Ak rozhranie CLI
nedokáže prečítať aktuálny zoznam indexov projektu, ponechá zoznam nezmenený a namiesto jeho nahradenia zobrazí varovanie.

{% hint style="warning" %}
**Existujú tri zoznamy indexov – nezamieňajte ich.**Rozbaľovacie menu „Nastavenia projektu“ v grafickom rozhraní obsahuje 27 vzorcov (pridáva `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` – týchto päť je k dispozícii iba v grafickom rozhraní a**neplatí** pre `--indices`). Príkaz live/offline `lattice index --preset` používa vlastný samostatný zoznam s 22 prednastaveniami. Vzorce a výpočty pásiem sú zdokumentované v [Vzorce multispektrálnych indexov](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Svetelné senzory DAQ: Stručný prehľad

Rodina `daq pool-*` ovláda spektrálne senzory DAQ spoločnosti MAPIR
(DAQ-U cez USB, DAQ-M cez BLE, DAQ-E cez Ethernet) prostredníctvom trvalého poola backendu — grafické rozhranie (GUI), rozhranie príkazového riadku (CLI
) a súbor nástrojov pre vývojárov (SDK
) zdieľajú jeden živý identifikátor. **`pool-*` je podporovaná DAQ cesta v dodávanom CLI
**; ostatné podpríkazy `daq`, na ktoré sa môžete stretnúť, sú iba interným rozhraním MAPIR
určeným len pre zdroj a ukončia sa s explicitnou chybou, ktorá vás nasmeruje na `pool-*`.

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

`pool-record` bez `--duration` beží až do `pool-record --stop`; predvolený výstupný adresár je `~/Documents/DAQ Live View/` **na počítači backendu**. Profil korekcie kapacity sa volí v čase pripojenia (`--cap-id`, predvolený profil backendu `sunshine_cosine`) a je možné ho za behu zmeniť pomocou `pool-set-cap` — profily obmedzenia a kalibrovaný rozsah senzora sú popísané v kapitolách o DAQ v tejto príručke.

{% hint style="warning" %}
**DAQ-E na hostiteľskom počítači s viacerými sieťovými kartami:** Prvé automatické vyhľadávanie `pool-connect --eth` po spustení systému môže zlyhať aj pri funkčnom senzore. `--eth-host <ip-or-hostname>` je spoľahlivá alternatíva — použite ju vždy, keď vyhľadávanie neprinesie žiadne výsledky.
{% endhint %}

***

## Kamery LATTICE, PTP a automatizácia projektov

Rodina príkazov `lattice` (viac ako 45 podpríkazov) pokrýva prácu s kamerami LATTICE od začiatku do konca: vyhľadávanie, jednotlivé snímky, trvalé synchronizované polia s procesom pripojenia „smart-prep“ v grafickom rozhraní, živý náhľad v prehliadači, zarovnanie, výpočty indexov a diagnostika sieťových kariet hostiteľa. Ukážka:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Okrem toho: `chloros-cli time-sync` podáva správy o grandmasteri PTP, ktorý beží na hostiteľskom počítači Chloros
(kamery LATTICE a senzory DAQ-E sú k nemu pripojené ako slave zariadenia na účely časových značiek medzi zariadeniami), a `chloros-cli project` otvára uložený projekt Chloros
a bez grafického rozhrania ovláda jeho kamery, polia a senzory — vrátane skriptovaných postupov snímania v YAML.

Tieto tri rodiny (`lattice`, `project`, `daq pool-*`) sú zároveň jediné, ktoré podporujú použitie `CHLOROS_BACKEND_URL` na ovládanie **vzdialeného** backendu; základné príkazy sú vždy určené pre lokálny počítač.

Úplné návody nájdete v kapitolách venovaných LATTICE v tejto príručke; všetky príznaky sú uvedené v [Referencii CLI
](reference/cli-reference.md).

***

## Riešenie problémov: Top 5

| Príznak | Riešenie |
| --- | --- |
| `Login required` alebo naplánovaná úloha sa zasekne na výzve `Email:` | Spustite na tomto počítači raz príkaz `chloros-cli login EMAIL 'PASSWORD'` — príkazy bez uloženého relácie sa budú spúšťať interaktívne namiesto toho, aby sa okamžite skončili neúspechom. |
| `backend unreachable` | Spustite desktopovú aplikáciu Chloros
alebo spustite backend ručne: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` v systéme Windows
(z priečinka `backend` vo vašom inštalačnom priečinku; v systéme `PATH` sa nenachádza), `chloros-backend` v systéme Linux
. `chloros-cli backend status` ukazuje, či niektorý z nich beží; `lattice`/`project`/`daq pool-*` nikdy žiadny nespúšťajú. Ak ich nasmerujete na vzdialený backend, skontrolujte `CHLOROS_BACKEND_URL`. |
| Pripojenie k poľu je blokované: `FRAMES WILL DROP` / `Reduce ROI to enable` | Príjemný prstenec sieťovej karty hostiteľa bol resetovaný na predvolené nastavenia — hlavná príčina toho, že zariadenie, ktoré predtým fungovalo, odmieta pripojenie, zvyčajne po aktualizácii ovládača sieťovej karty. Spustite príkaz `chloros-cli lattice network --fix` z terminálu s **zvýšenými oprávneniami** (alebo nastavte `ReceiveBufferLen=256`, `PendingReceives=64`); pozrite si časť *Nastavenie a ladenie sieťovej karty hostiteľa* v referenčnom materiáli. |
| Podpríkaz `daq` sa ukončí s hlásením: „vyžaduje kompletný balík DAQ…“ | Použite príkazy `daq pool-*`, ktoré ovládajú ten istý senzor prostredníctvom backendu: pripojenie, streamovanie, nahrávanie, výber kapacity a v prípade DAQ-E aj jeho sieťové nastavenia, reštart a aktualizácie firmvéru (alebo `chloros_sdk.connect_daq_sensor()` z Python
u). |
| Jetson zobrazuje varovanie o swapovaní pred spracovaním veľkých zložiek | Pridajte swapovanie založené na súboroch — CLI
zobrazí presné príkazy `fallocate`/`swapon`, ktoré je potrebné spustiť. |

***

## Pomoc

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Všetky príznaky, všetky podpríkazy:** [Referencia CLI
](reference/cli-reference.md)
* **Python
ekvivalent:** [Python
SDK
](api-python-sdk.md) a [Referencia SDK
](reference/sdk-reference.md)
* **Podpora:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)

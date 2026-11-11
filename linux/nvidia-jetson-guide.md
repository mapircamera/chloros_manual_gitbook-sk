# Príručka k NVIDIA Jetson –

Chloros
na platforme NVIDIA Jetson umožňuje multispektrálne spracovanie obrazu na okraji siete – v teréne, na bezpilotných lietadlách (UAV) a vo vzdialených zariadeniach. Chloros
1.3.0 pri spustení rozpozná váš model Jetson a optimalizuje stratégiu spracovania pre zistený hardvér. **Nie je potrebné žiadne ručné nastavovanie.**
{% hint style="info" %}
**Balík 1.3.0 pre Jetson spracováva údaje na grafickom procesore (GPU) zariadenia Jetson.** `chloros_1.3.0_arm64_jp6.deb` je vytvorený pre JetPack 6 a využíva CUDA verziu PyTorch z JetPack 6, takže spracovanie, vrátane debayera s podporou textúr, prebieha na GPU zariadenia Orin. Balík neobsahuje samotnú CUDA: využíva knižnice CUDA a cuDNN, ktoré inštaluje JetPack 6 (`sudo apt install nvidia-jetpack` na obraze, kde chýbajú). Ak nie je k dispozícii žiadne GPU s podporou CUDA, spracovanie prebieha na CPU a Texture Aware používa štandardný debayer.
{% endhint %}

***

## Podporované modely Jetson

| Model                | RAM            | Stratégia spracovania                                     | Odporúčané použitie                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB zdieľanej pamäte | `GPU_PARALLEL` (2 pracovné procesy)                              | Maximálny výkon, veľké dátové súbory                      |
| **Jetson Orin NX**   | 8–16 GB zdieľanej | `GPU_PARALLEL` (2 pracovné procesy, 16 GB) / `GPU_SINGLE` (8 GB)   | Hlavné odporúčanie pre nasadenie v teréne              |
| **Jetson Orin Nano** | 8 GB zdieľanej pamäte     | `GPU_SINGLE` (1 pracovný proces, sekvenčné spracovanie)                     | Základná úroveň výpočtov na okraji siete                                 |

{% hint style="info" %}
Balík pre Linux
arm64 vyžaduje **JetPack 6**, ktorý je k dispozícii pre rodinu zariadení Jetson Orin. Staršie modely (Nano, TX2, Xavier NX) nemôžu spúšťať JetPack 6 a nie sú podporované aktuálnym balíkom.
{% endhint %}

***

## Požiadavky

* **JetPack 6.x** (odporúča sa najnovšia verzia)
* **NVIDIA CUDA a cuDNN** z balíka JetPack 6 (`sudo apt install nvidia-jetpack`, ak ich váš obraz neobsahuje)
* **Platený plán Chloros
+** — úroveň Copper alebo vyššia (vyžaduje sa pre všetky prístupy cez CLI
/SDK
; vynucované na strane servera)

## Inštalácia

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

`.deb` nainštaluje Python
SDK
pre systém `python3`, ktorý zodpovedá tejto verzii. Iba do prostredia virtualenv je potrebné nainštalovať priložený balík wheel: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (plne offline). Všeobecné podrobnosti o inštalácii v systéme Linux
, umiestnenie súborov a riešenie problémov nájdete v článku [Inštalácia v systéme Linux
](linux-installation.md).

{% hint style="info" %}
**Pri spustení už nedochádza k rozbaľovaniu.** Od verzie 1.3.0 sa backend a CLI
inštalujú v nezbalenom stave do adresára `/usr/lib/chloros/`, takže sa už pri každom spustení nerozbaľujú do dočasného adresára. Dočasné súbory sa naďalej automaticky ukladajú do adresára `/mnt/ssd/tmp`, ak tento adresár existuje; v opačnom prípade môžete nastaviť `TMPDIR` na cestu na vašom NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dynamická adaptácia výpočtového výkonu na Jetsone

### Ako to funguje

Pri spustení systém Chloros
vytvorí profil vášho systému:

1. **Zistí model Jetsonu** prostredníctvom `/proc/device-tree/model`
2. **Načíta dostupnú zdieľanú pamäť GPU/CPU** (Jetson používa zjednotenú pamäť)
3. **Opýta sa PyTorch, či môže používať CUDA** — ak je nainštalovaný JetPack 6 s CUDA a cuDNN, môže
4. **Automaticky vyberie stratégiu spracovania** (`GPU_PARALLEL`, `GPU_SINGLE` alebo `CPU_PARALLEL`) a počet pracovných procesov

Rozhodnutie závisí od **celkovej veľkosti zdieľanej RAM**, nie od názvu modelu:

* **Pri celkovej RAM menej ako 12 GB**(všetky zariadenia Jetson s 8 GB):**1 pracovník — zámerné sekvenčné spracovanie**. Pamäť je príliš obmedzená na súbežné spustenie viacerých pracovníkov, takže obrázky sa spracúvajú po jednom. Na zariadeniach Jetson s**8 GB alebo menej** vlákno 3 úplne vynecháva skupinu pracovných procesov a spracováva úlohy pre jednotlivé obrázky priamo v rámci procesu. Backendový protokol označuje túto stratégiu ako `GPU_SINGLE`.
* **12 GB alebo viac**(Orin NX 16 GB, AGX Orin): zjednotená pamäť spĺňa podmienky pre `GPU_PARALLEL`, ale počet pracovných procesov je**na zariadeniach Jetson obmedzený na 2** — GPU, RAM pracovných procesov a ich kontexty CUDA pre jednotlivých pracovníkov čerpajú všetky z rovnakého zdieľaného fondu, takže pri väčšom počte pracovníkov hrozí riziko zlyhania z dôvodu nedostatku pamäte.
* **Žiadna použiteľná CUDA** (chýbajú knižnice CUDA pre JetPack): `CPU_PARALLEL` pri 12 GB alebo viac, nižšie je uvedený jeden obraz naraz.

Automatický výber môžete prepisovať prostredníctvom premennej prostredia `CHLOROS_STRATEGY` — pozrite si [Dynamická adaptácia výpočtov](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Správanie podľa modelu

| Model Jetson                | Stratégia       | Pracovníci | Vykonávanie                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Sekvenčná slučka v rámci procesu (`tiled_gpu` pri nedostatku pamäte) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Sekvenčná slučka v rámci procesu                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Súbežné pracovné procesy, cesta `fused_gpu`  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Súbežné pracovné procesy, cesta `fused_gpu`  |

Kľúčovým rozdielom medzi platformami je **pamäť**. Jetson s 8 GB musí pri vysokom zaťažení spracovávať obrázky po jednom pomocou pamäťovo efektívneho prístupu s rozdelením na dlaždice, zatiaľ čo Orin s 16 GB a viac dokáže spracovávať 2 obrázky súčasne cez GPU pomocou fúzovaného potrubia s vyššou priepustnosťou.

### Rozpočet GPU na model

Každý model Jetson má tiež hardvérový profil, ktorý obmedzuje, koľko prostriedkov zo zdieľaného fondu môže spracovanie obsadiť, a škaluje veľkosti dávok:

| Model | Maximálny rozpočet GPU | Multiplikátor veľkosti dávky | Rezervované pre systém/displej |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70 % | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75 % | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80 % | ×1,5 | 4,0 GB |

Profil sa prispôsobuje podľa zistenej veľkosti pamäte RAM: v prípade zariadenia Jetson s **16 GB alebo viac** sa násobiteľ veľkosti dávky zvýši na ×1,2. Základná veľkosť dávky pred uplatnením násobiteľov je 8 obrázkov.

Úplné informácie o prispôsobení výpočtovej kapacity nájdete v článku [Dynamické prispôsobenie výpočtovej kapacity](../processing-architecture/dynamic-compute-adaptation.md).

***

## Obmedzenie frekvencie GPU pre funkciu Texture Aware na modeloch Nano a Orin Nano

Algoritmus Texture Aware debayer vykonáva inferenciu neurónovej siete na GPU, čo môže pri plnej taktovacej frekvencii GPU vyvolať **varovania pred nadmerným prúdom**na modeloch Jetson s nízkou spotrebou energie (trieda 10–15 W). Ak zadáte `--debayer texture-aware` na**Jetson Nano alebo Orin Nano**, CLI
skontroluje maximálnu frekvenciu GPU a obmedzí ju na **510 MHz** (510000000), ak je aktuálne vyššia:

* Ak môže CLI
zapísať do uzla sysfs frekvenciu GPU, obmedzenie sa **uplatní automaticky** a zobrazí sa potvrdenie.
* Ak nie (vyžaduje oprávnenie root), CLI
vypíše presný príkaz `sudo` na ručné nastavenie obmedzenia, chvíľu počká, aby ste si ho mohli prečítať, a potom pokračuje — spracovanie stále beží, ale môžu sa zobrazovať varovania o nadmernom prúde.

Ak chcete obmedzenie nastaviť sami pred spracovaním:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modely s vyšším výkonom (Orin NX 25 W, AGX Orin 60 W) bežia na plnej rýchlosti GPU; obmedzenie sa neuplatňuje. Štandardný debayer nikdy nespustí obmedzenie na žiadnom modeli.

{% hint style="info" %}
**Funkcia „Texture Aware“ na Jetsone vždy spracováva jeden obrázok naraz.** Každý worker by potreboval vlastný kontext CUDA (~1 GB) plus vlastnú kópiu modelu odšumovača, čo si zjednotená pamäť nemôže dovoliť — preto je na Jetsone cesta Texture Aware viazaná na jediného workera so sériovým prístupom k GPU. Očakávajte, že Texture Aware bude na akomkoľvek Jetsone výrazne pomalší ako Standard. Texture Aware sa vzťahuje na obrázky typu Survey3
: snímky LATTICE vždy používajú debayer v režime Standard. Bez použiteľného CUDA GPU sa spustí režim Standard a backend zaznamená jeden riadok `[DEBAYER] … needs a CUDA GPU …`, v ktorom vysvetlí dôvod (zobrazené s `chloros-cli --verbose`).
{% endhint %}

***

## Riadenie teploty

Zariadenia Jetson majú obmedzenú tepelnú rezervu, najmä pri inštaláciách v uzavretých priestoroch alebo vo vzduchu. Chloros
monitoruje teplotu SoC a automaticky obmedzuje veľkosť dávok:

| Teplota         | Akcia                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normálna prevádzka — plná rýchlosť spracovania          |
| **70 °C** (Varovanie)  | Veľkosť dávky sa postupne znižuje (100 % → 50 % v rozmedzí 70 °C až 80 °C) |
| **80 °C** (Kritické) | Agresívne obmedzovanie (50 % → 0 % v rozmedzí 80 °C až 90 °C) |
| **90 °C** (Vypnutie) | Úplné zastavenie spracovania GPU — je potrebné ochladenie |

{% hint style="warning" %}
**Zabezpečte dostatočné vetranie a odvod tepla** pre nepretržité spracovanie, najmä v uzavretých poľných skriniach alebo leteckých systémoch. Tepelné obmedzenie zníži výkonnosť spracovania s cieľom chrániť hardvér.
{% endhint %}

***

## Správa pamäte

Zariadenia Jetson používajú **unifikovanú pamäť** — GPU a CPU zdieľajú rovnakú fyzickú RAM. Uvádzaná VRAM (napr. ~15,3 GB na Orin NX 16 GB) nie je vyhradená pamäť GPU; ide o tú istú RAM, ktorú používa operačný systém a všetky ostatné procesy.

### Upozornenie a odporúčania týkajúce sa výmennej pamäte

Pred spracovaním na zariadení Jetson CLI
spočíta počet súborov RAW vo vašej vstupnej zložke (`.tif`, `.tiff`, `.raw`, `.dng` — náhľady vo formáte JPG sa nezapočítavajú), odhadne maximálnu pamäťovú náročnosť spustenia a **varuje pred spustením**, ak je pravdepodobné, že RAM + swapová pamäť nebudú stačiť. Varovanie má názov `LOW MEMORY WARNING - Jetson Detected`, vypíše počet vašich obrázkov, veľkosť RAM, aktuálny swap a odhadovanú maximálnu potrebu, a potom poskytne presné príkazy `fallocate` / `chmod` / `mkswap` / `swapon` prispôsobené veľkosti váš projekt (nikdy menšie ako 8 GB). Na niekoľko sekúnd sa pozastaví, aby sa správa nestratila v histórii posúvania, a potom spracovanie pokračuje.**Odhady pamäte použité v upozornení:**

| Režim debayeringu | Základ | Na obrázok |
| --- | --- | --- |
| Štandardný | ~1,5 GB | ~10 MB |
| S ohľadom na textúry | ~2,5 GB (model + runtime Pythonu) | ~15 MB |

Varovanie sa spustí, keď odhadovaná špičková hodnota prekročí RAM + swap mínus 1 GB bezpečnostnú rezervu, pričom sa počíta iba **swap založený na súboroch** — konfigurácia používajúca výlučne zram bude aj tak označená.

Ak chcete pridať výmenný priestor ručne (príklad: 8 GB):

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->


### Spracovanie situácie OOM (Out of Memory)

Počas spracovania Chloros
monitoruje pamäť GPU a namiesto zrútenia sa plynule spomaľuje:

1. Keď využitie pamäte GPU prekročí **85 %**, veľkosti dávok sa preventívne znížia
2. Ak aj napriek tomu dôjde k udalosti nedostatku pamäte, veľkosť dávky sa **zníži na polovicu** a pri každom ďalšom OOM sa zníži opäť na polovicu; každá nasledujúca úspešná dávka posunie túto penalizáciu o jeden krok späť
3. Pri trvalom zaťažení sa spracovateľský reťazec prepne z cesty `fused_gpu` na pamäťovo úspornú cestu `tiled_gpu` a v krajnom prípade na spracovanie na CPU

Keďže GPU zdieľa RAM zariadenia Jetson, RAM spolu so swapom stále obmedzujú beh — vyššie uvedené varovanie o swape je kontrolou, ktorú treba brať do úvahy.

***

## Nasadenie v teréne

### Úvahy o spotrebe energie

| Model Jetson     | Typická spotreba energie | Poznámky                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | DC konektor typu „barrel“          |
| Jetson Orin NX   | 10–25 W             | DC konektor typu „barrel“          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD alebo konektor typu „barrel“ |

Naplánujte si spotrebu energie pre nepretržité spracovanie — najvyšší odber energie nastáva počas vlákna 3 (Spracovanie), ktoré intenzívne využíva GPU.

### Odporúčania týkajúce sa úložiska

* **NVMe SSD** sa dôrazne odporúča pre nasadenia s architektúrou arm64
* SD karty sú na spracovanie príliš pomalé — používajte ich iba ako spúšťacie médiá
* Počítajte s 2–3-násobkom veľkosti surových obrazových dát pre spracovaný výstup

### Prevádzka bez grafického rozhrania cez SSH


Chloros
je ideálny pre nasadenia Jetsonu bez grafického rozhrania:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Neustále aktívny backend pre synchronizáciu času LATTICE / DAQ-E

Ak váš Jetson bezmonitorovo ovláda kamery LATTICE alebo svetelné senzory DAQ-E, aktivujte backendovú službu systemd **ako váš používateľ**, aby grandmaster PTP bežal nepretržite a skripty SDK
našli bežiaci backend (jednotka je nainštalovaná, ale štandardne nie je aktivovaná):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

Backend potom beží pod vaším účtom: záznamy, ktoré zapisuje, sú vaše a používa vaše `chloros-cli login`. Podrobnosti, vrátane súboru root `chloros-backend.service` a toho, ako balík umožňuje viazanie portov PTP 319/320 bez oprávnení root, nájdete v [Inštalácii pre Linux
](linux-installation.md#always-on-ptp-for-headless-hosts).

### Automatizované spracovanie pomocou systemd

Vytvorte službu systemd pre automatizované spracovanie:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

`chloros-cli process` ukončí beh s nenulovým kódom, ak spustenie, ktoré požiadalo o produkty, nezapíše žiadne snímky, takže stav chyby v systemd má význam pre monitorovanie.

Prepojte so systémovým časovačom systemd na plánované spracovanie:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Príklady pracovných postupov

### Základné spracovanie na Jetsone

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI
```

### PythonSDK
na Jetsone

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Hromadné spracovanie viacerých letov

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## Odporúčané systémy Jetson pre použitie v teréne

Pre nasadenie v teréne a vo vzduchu zvážte tieto možnosti nosných dosiek Jetson Orin NX 16 GB:

* **Vo vzduchu/drony**: Systémy s odolnosťou proti vibráciám (MIL-STD), nízka hmotnosť (menej ako 300 g), pasívne chladenie
* **Náročné terénne podmienky**: Vodotesné kryty s krytím IP67/IP69K a pripojením kamery cez PoE GigE
* **Minimálne náklady/rozpočet**: Vývojárske sady s doplnkovými krytmi

Obráťte sa na [podporu MAPIR
](https://www.mapir.camera/community/contact), kde získate konkrétne odporúčania týkajúce sa hardvéru pre váš scenár nasadenia.

***

## Ďalšie kroky

* [Inštalácia Linux
u](linux-installation.md) — Všeobecné informácie o inštalácii Linux
u
* [Dynamická adaptácia výpočtového výkonu](../processing-architecture/dynamic-compute-adaptation.md) — Kompletný prehľad stratégií výpočtového výkonu
* [Spracovateľský pipeline](../processing-architecture/processing-pipeline.md) — Vysvetlenie 4-vláknového pipeline
* [CLI
: Príkazový riadok](../CLI.md) — Príručka k CLI

* [API
: Python
SDK
](../api-python-sdk.md) — Príručka k SDK

* [Referenčná príručka CLI
](../reference/cli-reference.md) a [Referenčná príručka SDK
](../reference/sdk-reference.md) — Úplný zoznam príkazov a API
pre verziu 1.3.0

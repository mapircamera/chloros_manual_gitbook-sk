# Príručka k NVIDIA Jetson

Chloros na platforme NVIDIA Jetson umožňuje multispektrálne spracovanie obrazu na okraji siete – v teréne, na bezpilotných lietadlách (UAV) a vo vzdialených zariadeniach. Chloros 1.2.0 pri spustení rozpozná váš model Jetson a optimalizuje stratégiu spracovania pre zistený hardvér. **Nie je potrebné žiadne ručné nastavovanie.**

***

## Podporované modely Jetson

| Model                | RAM            | Stratégia spracovania                                     | Odporúčané použitie                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB zdieľanej | `GPU_PARALLEL` (2 pracovné procesy)                              | Maximálny výkon, veľké dátové súbory                      |
| **Jetson Orin NX**   | 8–16 GB zdieľaná  | `GPU_PARALLEL` (2 pracovné procesy, 16 GB) / `GPU_SINGLE` (8 GB)   | Hlavné odporúčanie pre nasadenie vo vzduchu a v teréne |
| **Jetson Orin Nano** | 8 GB zdieľanej pamäte     | `GPU_SINGLE` (1 pracovný modul, sekvenčný)                     | Vstupná úroveň výpočtov na okraji siete                                 |

{% hint style="info" %}
Balík Linux pre architektúru arm64 vyžaduje **JetPack 6**, ktorý je k dispozícii pre produktovú radu Jetson Orin. Staršie modely (Nano, TX2, Xavier NX) nemôžu spúšťať JetPack 6 a nie sú podporované aktuálnym balíkom.
{% endhint %}

***

## Požiadavky

* **JetPack 6.x** (odporúča sa najnovšia verzia)
* **NVIDIA CUDA** (súčasť balíka JetPack)
* **Platený plán Chloros+** — úroveň Copper alebo vyššia (vyžaduje sa pre všetky prístupy k CLI/SDK; vynucované na strane servera)

## Inštalácia

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

Všeobecné informácie o inštalácii Linux, umiestnenie súborov a riešenie problémov nájdete v článku [Inštalácia Linux](linux-installation.md).

{% hint style="info" %}
**Umiestnite adresár na rozbalenie na rýchle úložisko.** Kompilované binárne súbory sa pri každom spustení samy rozbalia do dočasného adresára — čo je z SD karty mimoriadne pomalé. Chloros automaticky používa `/mnt/ssd/tmp`, ak existuje; v opačnom prípade nastavte `TMPDIR` na cestu na vašom NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dynamická adaptácia výpočtov na Jetsone

### Ako to funguje

Pri spustení Chloros vytvorí profil vášho systému:

1. **Zistí model Jetsonu** prostredníctvom `/proc/device-tree/model`
2. **Zistí dostupnú zdieľanú pamäť GPU/CPU** (Jetson používa zjednotenú pamäť)
3. **Vyberie stratégiu spracovania** (`GPU_PARALLEL`, `GPU_SINGLE` alebo `CPU_PARALLEL`)
4. **Automaticky nastaví počet pracovných procesov, typ potrubia a alokáciu pamäte**Rozhodnutie závisí od**celkovej veľkosti zdieľanej RAM**, nie od názvu modelu:

* **Pri celkovej RAM menšej ako 12 GB**(všetky zariadenia Jetson s 8 GB): `GPU_SINGLE` s**1 pracovným procesom – zámerné sekvenčné spracovanie**. Pamäť je príliš obmedzená na súbežné pracovné procesy, takže sa obrázky spracúvajú po jednom. Na zariadeniach Jetson s**8 GB alebo menej** vlákno 3 úplne vynecháva skupinu pracovných procesov a spracováva úlohy pre jednotlivé obrázky priamo v procese.
* **12 GB alebo viac**(Orin NX 16 GB, AGX Orin): zjednotená pamäť spĺňa podmienky pre `GPU_PARALLEL`, ale počet pracovných procesov je**na zariadeniach Jetson obmedzený na 2** — grafický procesor (GPU), pamäť RAM pracovných procesov a ich kontexty CUDA pre jednotlivých pracovníkov čerpajú všetky z rovnakého zdieľaného fondu, takže väčší počet pracovníkov zvyšuje riziko zlyhania z dôvodu nedostatku pamäte.

Automatický výber môžete prepisovať prostredníctvom premennej prostredia `CHLOROS_STRATEGY` — pozrite si [Dynamická adaptácia výpočtov](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Správanie podľa modelu

| Model Jetson                | Stratégia       | Pracovné procesy | Vykonávanie                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Sekvenčná slučka v rámci procesu (`tiled_gpu` pri nedostatku pamäte) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Sekvenčná slučka v rámci procesu                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Súbežné pracovné procesy, cesta `fused_gpu`  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Súbežné pracovné procesy, cesta `fused_gpu`  |

Kľúčovým rozdielom medzi platformami je **pamäť**. Jetson s 8 GB musí pri vysokom zaťažení spracovávať obrázky po jednom pomocou pamäťovo efektívneho prístupu s rozdelením na dlaždice, zatiaľ čo Orin s 16 GB a viac dokáže spracovávať 2 obrázky súčasne cez GPU pomocou fúzovaného potrubia s vyššou priepustnosťou.

### Rozpočet GPU na model

Každý model Jetson má tiež hardvérový profil, ktorý obmedzuje, koľko prostriedkov zo zdieľaného fondu môže spracovanie využiť, a škaluje veľkosti dávok:

| Model | Horná hranica rozpočtu GPU | Multiplikátor veľkosti dávky | Vyhradené pre systém/displej |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70 % | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75 % | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80 % | ×1,5 | 4,0 GB |

Zistená veľkosť pamäte RAM upravuje profil: v prípade zariadenia Jetson s **16 GB alebo viac** sa násobiteľ veľkosti dávky zvýši na ×1,2. Základná veľkosť dávky pred vynásobením je 8 obrázkov.

Úplný popis prispôsobenia výpočtov nájdete v časti [Dynamické prispôsobenie výpočtov](../processing-architecture/dynamic-compute-adaptation.md).

***

## Obmedzenie frekvencie GPU pre funkciu Texture Aware na modeloch Nano a Orin Nano

Funkcia Texture Aware debayer spúšťa inferenciu neurónovej siete na GPU, čo môže pri plnej taktovacej frekvencii GPU vyvolať **varovania pred nadmerným prúdom**na modeloch Jetson s nízkou spotrebou (trieda 10–15 W). Pred spracovaním funkciou Texture Aware na**Jetson Nano alebo Orin Nano**skontroluje Chloros maximálnu frekvenciu GPU a obmedzí ju na**510 MHz** (510000000), ak je aktuálne vyššia:

* Ak môže príkaz CLI zapísať do uzla sysfs frekvenciu GPU, obmedzenie sa **uplatní automaticky** a zobrazí sa potvrdenie.
* Ak nie (vyžaduje sa oprávnenie root), CLI vypíše presný príkaz `sudo` na ručné nastavenie obmedzenia, chvíľu počká, aby ste si ho mohli prečítať, a potom pokračuje — spracovanie stále beží, ale môžu sa zobrazovať varovania o nadmernom prúde.

Ak chcete obmedzenie nastaviť sami pred spracovaním:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modely s vyšším výkonom (Orin NX 25 W, AGX Orin 60 W) bežia na plnej rýchlosti grafického procesora; obmedzenie sa neuplatňuje. Štandardný debayer nikdy nespustí obmedzenie na žiadnom modeli.

{% hint style="info" %}
**Funkcia Texture Aware na Jetsone vždy spracováva jeden obrázok naraz.** Každý worker by potreboval vlastný kontext CUDA (~1 GB) plus vlastnú kópiu modelu odšumovača, čo si zjednotená pamäť nemôže dovoliť – preto je na zariadeniach Jetson cesta „Texture Aware“ pripútaná k jedinému workeru so sériovým prístupom k GPU. Očakávajte, že funkcia „Texture Aware“ bude na akomkoľvek zariadení Jetson výrazne pomalšia ako štandardná verzia.
{% endhint %}

***

## Riadenie teploty

Zariadenia Jetson majú obmedzenú teplotnú rezervu, najmä pri inštaláciách v uzavretých priestoroch alebo vo vzduchu. Chloros monitoruje teplotu SoC a automaticky obmedzuje veľkosť dávok:

| Teplota         | Akcia                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normálna prevádzka – plná rýchlosť spracovania          |
| **70 °C** (Varovanie) | Veľkosť dávky sa postupne znižuje (100 % → 50 % v rozmedzí 70 °C až 80 °C) |
| **80 °C** (Kritické) | Agresívne obmedzovanie (z 50 % na 0 % v rozmedzí 80 °C až 90 °C) |
| **90 °C** (Vypnutie) | Úplné zastavenie spracovania GPU — je potrebné vychladenie |

{% hint style="warning" %}
**Zabezpečte dostatočné vetranie a odvod tepla** pre nepretržité spracovanie, najmä v uzavretých poľných skriniach alebo leteckých systémoch. Tepelné obmedzenie zníži výkonnosť spracovania s cieľom chrániť hardvér.
{% endhint %}

***

## Správa pamäte

Zariadenia Jetson používajú **unifikovanú pamäť** — GPU a CPU zdieľajú tú istú fyzickú RAM. Uvádzaná VRAM (napr. ~15,3 GB na Orin NX 16 GB) nie je vyhradenou pamäťou GPU; ide o tú istú RAM, ktorú používa operačný systém a všetky ostatné procesy.

### Upozornenie a odporúčania týkajúce sa výmennej pamäte

Pred spracovaním na zariadení Jetson program CLI spočíta počet surových snímok vo vašej vstupnej zložke (`.tif`, `.tiff`, `.raw`, `.dng` — náhľady vo formáte JPG sa nezapočítavajú), odhadne maximálnu pamäťovú náročnosť spustenia a **varuje pred spustením**, ak je pravdepodobné, že RAM + výmenná pamäť nebudú postačovať. Varovanie má názov `LOW MEMORY WARNING - Jetson Detected`, zobrazí počet vašich snímok, veľkosť RAM, aktuálny swap a odhadovanú maximálnu spotrebu, a potom poskytne presné príkazy `fallocate` / `chmod` / `mkswap` / `swapon` prispôsobené veľkosti vášho projektu (nikdy menšie ako 8 GB). Na niekoľko sekúnd sa pozastaví, aby sa správa nestratila v zrolovanom výpise, a potom pokračuje v spracovaní.**Odhady pamäte použité v tomto varovaní:**

| Režim odbayeringu | Základ | Na obrázok |
| --- | --- | --- |
| Štandardný | ~1,5 GB | ~10 MB |
| S ohľadom na textúry | ~2,5 GB (model + beh programu Python) | ~15 MB |

Varovanie sa spustí, keď odhadovaná špičková hodnota prekročí veľkosť RAM + výmennej pamäte mínus bezpečnostnú rezervu 1 GB, pričom sa počíta iba výmenná pamäť **podporovaná súbormi** — konfigurácia používajúca výlučne zram bude aj tak označená.

Ručné pridanie výmenného priestoru (príklad: 8 GB):



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

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
```### Spracovanie situácie OOM (Out of Memory)

Počas spracovania Chloros monitoruje pamäť GPU a namiesto zrútenia sa plynule obmedzuje výkon:

1. Keď využitie pamäte GPU prekročí **85 %**, veľkosti dávok sa preventívne znížia
2. Ak sa napriek tomu vyskytne udalosť nedostatku pamäte, veľkosť dávky sa **zníži na polovicu** a pri každom ďalšom OOM sa zníži opäť na polovicu; každá nasledujúca úspešná dávka posunie toto obmedzenie o jeden krok späť
3. Pri dlhodobom zaťažení sa spracovacia línia prepne z režimu `fused_gpu` na pamäťovo úspornú cestu `tiled_gpu` a ako posledná možnosť na spracovanie na CPU

***

## Nasadenie v teréne

### Úvahy o spotrebe energie

| Model Jetson     | Typická spotreba energie | Poznámky                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | DC konektor typu „barrel“          |
| Jetson Orin NX   | 10–25 W             | DC konektor typu „barrel“          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD alebo konektor typu „barrel“ |

Naplánujte si spotrebu energie pre nepretržité spracovanie — špičkový odber energie nastáva počas vlákna 3 (Spracovanie), ktoré intenzívne zaťažuje GPU.

### Odporúčania týkajúce sa úložiska

* **NVMe SSD** sa dôrazne odporúča pre nasadenia s architektúrou arm64
* SD karty sú na spracovanie príliš pomalé — používajte ich iba ako spúšťacie médium
* Počítajte s 2–3-násobkom veľkosti surových obrazových dát pre spracovaný výstup

### Prevádzka bez monitoru prostredníctvom SSH

Chloros CLI je ideálny pre nasadenia Jetson bez obrazovky:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Neustále aktívny backend pre synchronizáciu času LATTICE / DAQ-E

Ak váš Jetson bezmonitorovo ovláda kamery LATTICE alebo svetelné senzory DAQ-E, aktivujte backendovú službu systemd, aby grandmaster PTP bežal nepretržite (služba je nainštalovaná, ale štandardne nie je aktivovaná):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Podrobnosti, vrátane toho, ako balík umožňuje viazať porty PTP 319/320 bez oprávnení root, nájdete v [Inštalácia Linux](linux-installation.md#always-on-ptp-for-headless-hosts).

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

`chloros-cli process` ukončí činnosť s nenulovým kódom, ak beh, ktorý požadoval výstupy, nezapíše žiadne obrázky, takže stav chyby v systemd má význam pre monitorovanie.

Pre naplánované spracovanie ju skombinujte s časovačom systemd:

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
    --indices NDVI NDRE GNDVI
```

### Python SDK na Jetsone

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
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
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Odporúčané systémy Jetson pre použitie v teréne

Pre nasadenie v teréne a vo vzduchu zvážte tieto možnosti nosných dosiek Jetson Orin NX 16 GB:

* **Vo vzduchu/drony**: Systémy odolné voči vibráciám (MIL-STD), ľahké (do 300 g), s pasívnym chladením
* **Odolné terénne použitie**: Vodotesné kryty s certifikáciou IP67/IP69K a pripojením kamery cez PoE GigE
* **Minimálne/cenovo výhodné**: Vývojárske sady s doplnkovými krytmi

Obráťte sa na [podporu MAPIR](https://www.mapir.camera/community/contact) s požiadavkou o konkrétne odporúčania týkajúce sa hardvéru pre váš scenár nasadenia.

***

## Ďalšie kroky

* [Inštalácia Linux](linux-installation.md) — Všeobecné podrobnosti o inštalácii Linux
* [Dynamická adaptácia výpočtového výkonu](../processing-architecture/dynamic-compute-adaptation.md) — Kompletný prehľad stratégií výpočtového výkonu
* [Spracovateľský potrubie](../processing-architecture/processing-pipeline.md) — Vysvetlenie 4-vláknového potrubia
* [CLI : Príkazový riadok](../CLI.md) — Príručka k CLI
* [API : Python SDK](../api-python-sdk.md) — Príručka k SDK
* [Referenčná príručka k CLI](../reference/cli-reference.md) a [Referenčná príručka k SDK](../reference/sdk-reference.md) — Úplný zoznam príkazov/API pre verziu 1.2.0

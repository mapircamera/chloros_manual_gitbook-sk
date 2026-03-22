# Príručka k NVIDIA Jetson

Chloros na platforme NVIDIA Jetson umožňuje spracovanie multispektrálnych snímok na okraji siete – v teréne, na bezpilotných lietadlách (UAV) a vo vzdialených zariadeniach. Chloros automaticky rozpozná váš model Jetson a optimalizuje stratégiu spracovania pre váš hardvér.

***

## Podporované modely Jetson

| Model                | RAM            | Stratégia spracovania                                   | Odporúčané použitie                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB zdieľanej | `GPU_PARALLEL` (4 pracovníci)                            | Maximálny výkon, veľké dátové súbory                      |
| **Jetson Orin NX**   | 8–16 GB zdieľanej  | `GPU_PARALLEL` (3 pracovné procesy, 16 GB) / `GPU_SINGLE` (8 GB) | Primárne odporúčanie pre nasadenie vo vzduchu a v teréne |
| **Jetson Orin Nano** | 8 GB zdieľanej     | `GPU_SINGLE` (1 pracovník)                               | Vstupná úroveň výpočtov na okraji siete                                 |
| **Jetson Nano**      | 4–8 GB zdieľanej   | `GPU_SINGLE` (1 pracovník)                               | Vstupná úroveň, obmedzená pamäť                          |

{% hint style="info" %}
**Staršie modely Jetson** (TX2, TX1, Xavier NX) nemusia byť podporované. Výkon sa bude líšiť v závislosti od dostupnej pamäte GPU a schopností CUDA.
{% endhint %}

***

## Požiadavky

* **JetPack 6.x** (odporúča sa najnovšia verzia)
* **NVIDIA CUDA** (súčasť balíka JetPack)
* **Licencia Chloros+** (vyžadovaná pre prístup k CLI/SDK)

## Inštalácia

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Všeobecné informácie o inštalácii Linux nájdete v [Inštalácia Linux](linux-installation.md).

***

## Dynamická adaptácia výpočtov na Jetsone

Chloros automaticky detekuje váš model Jetson a vyberie optimálnu stratégiu spracovania. **Nie je potrebné žiadne ručné nastavovanie.**

### Ako to funguje

Pri spustení Chloros vytvorí profil vášho systému:

1. **Detekuje model Jetson** prostredníctvom `/proc/device-tree/model`
2. **Načíta dostupnú GPU/zdieľanú pamäť**

3.**Vyberie stratégiu spracovania** (`GPU_PARALLEL`, `GPU_SINGLE` alebo `CPU_PARALLEL`)
4. **Automaticky nastaví počet pracovníkov, typ potrubia a alokáciu pamäte**

### Správanie podľa modelu

| Model Jetson                | Stratégia       | Pracovníci | Potrubie                       | Súbežnosť |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8 GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (úsporné využitie pamäte) | Sériové  |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Sériové  |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Sériové  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (plná cesta GPU)    | Súbežné  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Súbežné  |

{% hint style="success" %}
**Jetson Orin NX 16 GB** je ideálnou voľbou pre nasadenie na okraji siete – využíva stratégiu `GPU_PARALLEL` s 3 súbežnými pracovníkmi, čím poskytuje skutočné paralelné spracovanie na GPU v kompaktnom formáte.
{% endhint %}

Kľúčovým rozdielom medzi platformami je **pamäť**. Jetson Nano s 8 GB zdieľanej pamäte musí spracovávať obrázky po jednom pomocou pamäťovo efektívneho prístupu s dlaždicami, zatiaľ čo Orin NX s 16 GB môže spracovávať 3 obrázky súčasne cez GPU pomocou fúzovaného potrubia s vyššou priepustnosťou.

Kompletnú referenciu o prispôsobení výpočtov nájdete v [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

***

## Riadenie teploty

Zariadenia Jetson majú obmedzenú tepelnú rezervu, najmä pri inštaláciách v uzavretých priestoroch alebo vo vzduchu. Chloros obsahuje automatické monitorovanie teploty a obmedzovanie výkonu:

| Teplota         | Akcia                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normálna prevádzka — plná rýchlosť spracovania          |
| **70 °C** (Varovanie)  | Automatické zníženie veľkosti dávky                   |
| **80 °C** (Kritické) | Agresívne obmedzenie výkonu — nižšia súbežnosť         |
| **90 °C** (Vypnutie) | Úplné zastavenie spracovania GPU — vyžaduje sa ochladenie |

{% hint style="warning" %}
**Zabezpečte dostatočné vetranie a odvod tepla** pre nepretržité spracovanie, najmä v uzavretých poľných skriniach alebo leteckých systémoch. Tepelné obmedzenie výkonu zníži priepustnosť spracovania s cieľom chrániť hardvér.
{% endhint %}

***

## Správa pamäte

Zariadenia Jetson používajú **unifikovanú pamäť** — GPU a CPU zdieľajú rovnakú fyzickú RAM. To znamená, že uvádzaná VRAM (napr. 15,3 GB na Orin NX 16 GB) nie je vyhradená pamäť GPU; je zdieľaná s operačným systémom a inými procesmi.

### Odporúčania týkajúce sa výmennej pamäte

V prípade veľkých dátových súborov alebo spracovania debayera s podporou textúr môže Chloros odporučiť vytvorenie výmenného priestoru:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Odhadovaná pamäť na jeden obrázok:**

* Štandardný debayer: ~10 MB na obrázok
* Texture Aware debayer: ~15 MB na obrázok

Chloros automaticky vypočíta potrebnú pamäť na základe veľkosti vášho dátového súboru a upozorní vás, ak sa odporúča swap.

### Záložný plán pri nedostatku pamäte (OOM)

Ak sa počas spracovania zistí stav nedostatku pamäte:

1. Chloros automaticky zníži počet GPU workerov
2. Prejde z potrubia `fused_gpu` na potrubie `tiled_gpu` (úspornejšie z hľadiska pamäte)
3. Pokračuje v spracovaní so zníženou priepustnosťou namiesto toho, aby došlo k zlyhaniu

***

## Nasadenie v teréne

### Úvahy o napájaní

| Model Jetson     | Typický príkon | Poznámky                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5–10 W              | USB-C alebo konektor typu barrel    |
| Jetson Orin Nano | 7–15 W              | Konektor typu barrel na jednosmerný prúd          |
| Jetson Orin NX   | 10–25 W             | Konektor typu barrel na jednosmerný prúd          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD alebo konektor typu barrel |

Naplánujte si spotrebu energie pre nepretržité spracovanie — najvyšší odber energie nastáva počas GPU-náročného vlákna 3 (Spracovanie).

### Odporúčania týkajúce sa úložiska

* **NVMe SSD** sa dôrazne odporúča pre nasadenia arm64
* SD karty sú na spracovanie príliš pomalé — používajte ich iba ako spúšťacie médiá
* Naplánujte si 2–3-násobok veľkosti surových obrazových dát pre spracovaný výstup

### Prevádzka bez grafického rozhrania prostredníctvom SSH

Chloros CLI je ideálny pre nasadenia Jetson bez grafického rozhrania:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Automatizované spracovanie so systémom systemd

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

Spojte so systémovým časovačom pre plánované spracovanie:

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
    --format tiff-32 \
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
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Odporúčané systémy Jetson pre použitie v teréne

Pre nasadenie v teréne a vo vzduchu zvážte tieto možnosti nosných dosiek Jetson Orin NX 16 GB:

* **Vo vzduchu/drony**: Systémy s odolnosťou proti vibráciám (MIL-STD), ľahké (do 300 g), pasívne chladenie
* **Náročné terénne podmienky**: Vodotesné kryty IP67/IP69K s pripojením kamery PoE GigE
* **Minimálne/cenovo výhodné**: Vývojárske sady s doplnkovými krytmi

Kontaktujte [MAPIR podporu](https://www.mapir.camera/community/contact) pre konkrétne odporúčania týkajúce sa hardvéru pre váš scenár nasadenia.

***

## Ďalšie kroky

* [Linux Inštalácia](linux-installation.md) — Všeobecné podrobnosti o inštalácii Linux
* [Dynamická adaptácia výpočtov](../processing-architecture/dynamic-compute-adaptation.md) — Kompletný referenčný materiál k stratégii výpočtov
* [Spracovateľský potrubie](../processing-architecture/processing-pipeline.md) — Vysvetlenie 4-vláknového potrubia
* [CLI : Príkazový riadok](../CLI.md) — Úplný referenčný materiál k CLI
* [API : Python SDK](../api-python-sdk.md) — Úplný referenčný materiál k SDK

# API : Python SDK

**Chloros Python SDK** poskytuje programový prístup k obrazovému procesorovému jadru Chloros, čo umožňuje automatizáciu, vlastné pracovné postupy a bezproblémovú integráciu s vašimi aplikáciami Python a výskumnými procesmi.

### Kľúčové vlastnosti

* 🐍 **Natívny Python** - Čistý, pythonský API pre spracovanie obrazu
* 🔧 **Plný prístup k API** - Kompletná kontrola nad spracovaním Chloros
* 🚀 **Automatizácia** - Vytvorte vlastné pracovné postupy hromadného spracovania
* 🔗 **Integrácia** – Vložte Chloros do existujúcich aplikácií Python
* 📊 **Pripravené na výskum** – Ideálne pre vedecké analytické procesy
* ⚡ **Paralelné spracovanie** – Škálovateľné podľa počtu jadier vášho procesora (Chloros+)

### Požiadavky

| Požiadavka          | Podrobnosti                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | Musí byť nainštalovaný lokálne                                           |
| **Licencia**          | Chloros+ ([vyžaduje sa platený plán](https://cloud.mapir.camera/pricing)) |
| **Operačný systém** | Windows 10/11 (64-bit)                                              |
| **Python**           | Python 3.7 alebo vyšší                                                |
| **Pamäť**           | Minimálne 8 GB RAM (odporúča sa 16 GB)                                  |
| **Internet**         | Potrebný na aktiváciu licencie                                     |

{% hint style=&quot;warning&quot; %}
**Požiadavky na licenciu**: Python SDK vyžaduje platené predplatné Chloros+ pre prístup k API. Štandardné (bezplatné) plány nemajú prístup k API/SDK. Prejdite na [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) a vykonajte upgrade.
{% endhint %}

## Rýchly štart

### Inštalácia

Inštalácia prostredníctvom pip:

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**Prvé nastavenie**: Pred použitím SDK aktivujte svoju licenciu Chloros+ otvorením Chloros, Chloros (prehliadač) alebo Chloros CLI a prihlásením sa pomocou svojich prihlasovacích údajov. Toto je potrebné urobiť len raz.
{% endhint %}

### Základné použitie

Spracujte priečinok s niekoľkými riadkami:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Plná kontrola

Pre pokročilé pracovné postupy:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Návod na inštaláciu

### Predpoklady

Pred inštaláciou SDK sa uistite, že máte:

1. **Chloros Desktop** nainštalovaný ([stiahnuť](download.md))
2. **Python 3.7+** nainštalovaný ([python.org](https://www.python.org))
3. **Aktívna licencia Chloros+** ([upgrade](https://cloud.mapir.camera/pricing))

### Inštalácia prostredníctvom pip

**Štandardná inštalácia:**

```bash
pip install chloros-sdk
```

**S podporou monitorovania priebehu:**

```bash
pip install chloros-sdk[progress]
```

**Inštalácia pre vývoj:**

```bash
pip install chloros-sdk[dev]
```

### Overenie inštalácie

Otestujte, či je SDK nainštalovaný správne:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Prvé nastavenie

### Aktivácia licencie

SDK používa rovnakú licenciu ako Chloros, Chloros (prehliadač) a Chloros CLI. Aktivujte raz prostredníctvom GUI alebo CLI:

1. Otvorte **Chloros alebo Chloros (prehliadač)** a prihláste sa na karte Používateľ <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> . Alebo otvorte **CLI**.
2. Zadajte svoje prihlasovacie údaje Chloros+ a prihláste sa
3. Licencia je uložená v lokálnej pamäti (zostáva zachovaná aj po reštartovaní)

{% hint style=&quot;success&quot; %}
**Jednorazové nastavenie**: Po prihlásení prostredníctvom GUI alebo CLI, SDK automaticky používa uloženú licenciu. Nie je potrebné žiadne ďalšie overenie!
{% endhint %}

### Testovanie pripojenia

Overte, či sa SDK môže pripojiť k Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## API Referencia

### Trieda ChlorosLocal

Hlavná trieda pre lokálne spracovanie obrazu Chloros.

#### Konštruktor

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parametre:**

| Parameter                 | Typ | Predvolené nastavenie                   | Popis                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL lokálneho Chloros backendu          |
| `auto_start_backend`      | bool | `True`                    | V prípade potreby automaticky spustiť backend |
| `backend_exe`             | str  | `None` (automatická detekcia)      | Cesta k spustiteľnému súboru backendu            |
| `timeout`                 | int  | `30`                      | Časový limit požiadavky v sekundách            |
| `backend_startup_timeout` | int  | `60`                      | Časový limit na spustenie backendu (sekundy) |

**Príklady:**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### Metódy

#### `create_project(project_name, camera=None)`

Vytvorenie nového projektu Chloros.

**Parametre:**

| Parameter      | Typ | Povinné | Popis                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Áno      | Názov projektu                                     |
| `camera`       | str  | Nie       | Šablóna kamery (napr. „Survey3N\_RGN“, „Survey3W\_OCN“) |

**Vrátené hodnoty:** `dict` - Odpoveď na vytvorenie projektu

**Príklad:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Importujte obrázky zo zložky.

**Parametre:**

| Parameter     | Typ     | Povinné | Popis                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Áno      | Cesta k priečinku s obrázkami         |
| `recursive`   | bool     | Nie       | Vyhľadávať podpriečinky (predvolené: False) |

**Vrátené hodnoty:** `dict` – Výsledky importu s počtom súborov

**Príklad:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Konfigurácia nastavení spracovania.

**Parametre:**

| Parameter                 | Typ | Predvolené nastavenie                 | Popis                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | „Vysoká kvalita (rýchlejšie)“ | Metóda Debayer                  |
| `vignette_correction`     | bool | `True`                  | Povoliť korekciu vinetácie      |
| `reflectance_calibration` | bool | `True`                  | Povoliť kalibráciu odrazivosti  |
| `indices`                 | list | `None`                  | Vegetačné indexy na výpočet |
| `export_format`           | str  | „TIFF (16-bitový)“         | Výstupný formát                   |
| `ppk`                     | bool | `False`                 | Povoliť PPK korekcie          |
| `custom_settings`         | dict | `None`                  | Pokročilé vlastné nastavenia        |

**Formáty exportu:**

* `"TIFF (16-bit)"` – odporúčané pre GIS/fotogrametriu
* `"TIFF (32-bit, Percent)"` – vedecká analýza
* `"PNG (8-bit)"` – vizuálna kontrola
* `"JPG (8-bit)"` – Komprimovaný výstup

**Dostupné indexy:**

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 a ďalšie.

**Príklad:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Spracujte obrázky projektu.

**Parametre:**

| Parameter           | Typ     | Predvolené nastavenie      | Popis                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Režim spracovania: „parallel“ alebo „serial“   |
| `wait`              | bool     | `True`       | Čakanie na dokončenie                       |
| `progress_callback` | callable | `None`       | Funkcia spätného volania pokroku (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Interval opakovania pre pokrok (sekundy)   |

**Vrátené hodnoty:** `dict` – Výsledky spracovania

{% hint style=&quot;warning&quot; %}
**Paralelný režim**: Vyžaduje licenciu Chloros+. Automaticky sa prispôsobí jadrám vášho procesora (až 16 pracovníkov).
{% endhint %}

**Príklad:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Získa aktuálnu konfiguráciu projektu.

**Vrátené:** `dict` – Aktuálna konfigurácia projektu

**Príklad:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Získať informácie o stave backendu.

**Vrátené hodnoty:** `dict` – stav backendu

**Príklad:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Vypne backend (ak bol spustený pomocou SDK).

**Príklad:**

```python
chloros.shutdown_backend()
```

***

### Pomocné funkcie

#### `process_folder(folder_path, **options)`

Jednoradová pomocná funkcia na spracovanie priečinka.

**Parametre:**

| Parameter                 | Typ     | Predvolené         | Popis                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Povinné        | Cesta k priečinku s obrázkami     |
| `project_name`            | str      | Automaticky generované  | Názov projektu                   |
| `camera`                  | str      | `None`          | Šablóna kamery                |
| `indices`                 | list     | `["NDVI"]`      | Indexy na výpočet           |
| `vignette_correction`     | bool     | `True`          | Povoliť korekciu vinetácie     |
| `reflectance_calibration` | bool     | `True`          | Povoliť kalibráciu odrazivosti |
| `export_format`           | str      | „TIFF (16-bitový)“ | Výstupný formát                  |
| `mode`                    | str      | `"parallel"`    | Režim spracovania                |
| `progress_callback`       | callable | `None`          | Spätné volanie priebehu              |

**Vrátené hodnoty:** `dict` – Výsledky spracovania

**Príklad:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Podpora správcu kontextu

SDK podporuje správcov kontextu pre automatické čistenie:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Kompletné príklady

### Príklad 1: Základné spracovanie

Spracovanie priečinka s predvolenými nastaveniami:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Príklad 2: Vlastný pracovný postup

Plná kontrola nad spracovateľským potrubím:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Príklad 3: Hromadné spracovanie viacerých priečinkov

Spracovanie viacerých súborov údajov o letoch:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Príklad 4: Integrácia výskumného potrubia

Integrácia Chloros s analýzou údajov:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Príklad 5: Vlastné sledovanie priebehu

Pokročilé sledovanie priebehu s protokolovaním:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Príklad 6: Spracovanie chýb

Robustné spracovanie chýb pre produkčné použitie:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Príklad 7: Nástroj príkazového riadku

Vytvorte vlastný nástroj CLI pomocou SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    
    args = parser.parse_args()
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Použitie:**

```bash
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
```

***

## Spracovanie výnimiek

SDK poskytuje špecifické triedy výnimiek pre rôzne typy chýb:

### Hierarchia výnimiek

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Príklady výnimiek

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Pokročilé témy

### Vlastná konfigurácia backendu

Použite vlastnú polohu alebo konfiguráciu backendu:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Nezablokované spracovanie

Začnite spracovanie a pokračujte v ďalších úlohách:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Správa pamäte

V prípade veľkých dátových súborov spracúvajte v dávkach:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Riešenie problémov

### Backend sa nespustí

**Problém:** SDK sa nepodarí spustiť backend.

**Riešenia:**

1. Overte, či je nainštalovaný Chloros Desktop:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Skontrolujte, či Windows Firewall neblokuje
3. Skúste manuálnu cestu backendu:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### Licencia nebola detegovaná

**Problém:** SDK varuje o chýbajúcej licencii

**Riešenia:**

1. Otvorte Chloros, Chloros (prehliadač) alebo Chloros CLI a prihláste sa.
2. Overte, či je licencia uložená v pamäti cache:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Kontaktujte podporu: info@mapir.camera

***

### Chyby importu

**Problém:** `ModuleNotFoundError: No module named 'chloros_sdk'`

**Riešenia:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Časový limit spracovania

**Problém:** Časový limit spracovania

**Riešenia:**

1. Zvýšte časový limit:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Spracujte menšie dávky
3. Skontrolujte dostupný priestor na disku
4. Monitorujte systémové zdroje

***

### Port je už používaný

**Problém:** Backend port 5000 je obsadený

**Riešenia:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Alebo vyhľadajte a zatvorte konfliktný proces:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## Tipy na zvýšenie výkonu

### Optimalizácia rýchlosti spracovania

1. **Použite paralelný režim** (vyžaduje Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Znížte výstupné rozlíšenie** (ak je to prijateľné)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Deaktivujte nepotrebné indexy**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Spracúvajte na SSD** (nie HDD)

***

### Optimalizácia pamäte

Pre veľké súbory údajov:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Spracovanie na pozadí

Uvoľnite Python pre iné úlohy:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Príklady integrácie

### Integrácia Django

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## Často kladené otázky

### Otázka: Vyžaduje SDK pripojenie k internetu?

**Odpoveď:** Iba na počiatočnú aktiváciu licencie. Po prihlásení prostredníctvom Chloros, Chloros (prehliadač) alebo Chloros CLI sa licencia uloží do lokálnej vyrovnávacej pamäte a funguje offline počas 30 dní.

***

### Otázka: Môžem používať SDK na serveri bez grafického používateľského rozhrania?

**Odpoveď:** Áno! Požiadavky:

* Windows Server 2016 alebo novší
* Chloros nainštalovaný (jednorazovo)
* Licencia aktivovaná na akomkoľvek počítači (licencia uložená v cache skopírovaná na server)

***

### Otázka: Aký je rozdiel medzi Desktop, CLI a SDK?

| Funkcia         | Desktop GUI | CLI Príkazový riadok | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Rozhranie**   | Ukazovateľom myši | Príkazový riadok          | Python API  |
| **Najvhodnejšie pre**    | Vizuálnu prácu | Skriptovanie        | Integráciu |
| **Automatizácia**  | Obmedzená     | Dobrá             | Vynikajúca   |
| **Flexibilita** | Základná       | Dobrá             | Maximálna     |
| **Licencia**     | Chloros+    | Chloros+         | Chloros+    |

***

### Otázka: Môžem distribuovať aplikácie vytvorené pomocou SDK?

**Odpoveď:** Kód SDK je možné integrovať do vašich aplikácií, ale:

* Koncoví používatelia musia mať nainštalovaný Chloros.
* Koncoví používatelia musia mať aktívne licencie Chloros+.
* Komerčná distribúcia vyžaduje OEM licencie.

V prípade otázok týkajúcich sa OEM kontaktujte info@mapir.camera.

***

### Otázka: Ako aktualizujem SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### Otázka: Kde sa ukladajú spracované obrázky?

V predvolenom nastavení v ceste projektu:

```
Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### Otázka: Môžem spracovávať obrázky zo skriptov Python bežiacich podľa plánu?

**Odpoveď:** Áno! Použite plánovač úloh Windows so skriptami Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Naplánujte prostredníctvom plánovača úloh, aby sa spúšťal každý deň.

***

### Otázka: Podporuje SDK async/await?

**Odpoveď:** Aktuálna verzia je synchronná. Pre asynchrónne správanie použite `wait=False` alebo spustite v samostatnom vlákne:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## Pomoc

### Dokumentácia

* **Referencia API**: Táto stránka

### Kanály podpory

* **E-mail**: info@mapir.camera
* **Webová stránka**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Ceny**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Ukážkový kód

Všetky uvedené príklady sú otestované a pripravené na použitie. Skopírujte ich a prispôsobte svojim potrebám.

***

## Licencia

**Vlastnícky softvér** – Copyright (c) 2025 MAPIR Inc.

SDK vyžaduje aktívne predplatné Chloros+. Neoprávnené používanie, distribúcia alebo úprava sú zakázané.

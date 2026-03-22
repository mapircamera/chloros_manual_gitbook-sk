# API : Python SDK

**Chloros Python SDK** poskytuje programový prístup k enginu na spracovanie obrazu Chloros, čo umožňuje automatizáciu, vlastné pracovné postupy a hladkú integráciu s vašimi aplikáciami Python a výskumnými procesmi.

### Kľúčové funkcie

* 🐍 **Natívne Python** – Čistý, Pythonský API pre spracovanie obrazu
* 🔧 **Plný prístup k API** – Úplná kontrola nad spracovaním Chloros
* 🚀 **Automatizácia** – Vytvárajte vlastné pracovné postupy hromadného spracovania
* 🔗 **Integrácia** – Vložte Chloros do existujúcich aplikácií Python
* 📊 **Pripravené na výskum** – Ideálne pre vedecké analytické procesy
* ⚡ **Paralelné spracovanie** – Škálovateľné podľa počtu jadier vášho procesora (Chloros+)

### Požiadavky

| Požiadavka          | Podrobnosti                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros nainštalovaný** | Windows: Inštalačný program pre stolné počítače; Linux: balík `.deb`                  |
| **Licencia**          | Chloros+ ([vyžaduje sa platený plán](https://cloud.mapir.camera/pricing)) |
| **Operačný systém** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 alebo novší                                                |
| **Pamäť**           | Minimálne 8 GB RAM (odporúča sa 16 GB)                                  |
| **Internet**         | Potrebný na aktiváciu licencie                                     |

{% hint style="warning" %}
**Požiadavky na licenciu**: Python SDK vyžaduje platené predplatné Chloros+ pre prístup k API. Štandardné (bezplatné) plány nemajú prístup k API/SDK. Pre upgrade navštívte [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).
{% endhint %}

## Rýchly štart

### Inštalácia

Inštalácia cez pip:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**Prvé nastavenie**: Pred použitím SDK aktivujte svoju licenciu Chloros+ otvorením Chloros, Chloros (prehliadač) alebo Chloros CLI a prihlásením sa pomocou svojich prihlasovacích údajov. Toto je potrebné urobiť len raz. V Linux (bez grafického rozhrania) použite: `chloros-cli login user@example.com 'password'`
{% endhint %}

### Základné použitie

Spracujte priečinok len niekoľkými riadkami:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**Cesty pre viaceré platformy**: Príklady kódu na tejto stránke používajú cesty v štýle Windows (napr. `C:\\DroneImages\\Flight001`). V systéme Linux použite namiesto toho cesty typu Linux (napr. `/home/user/drone_images/flight001` alebo `~/drone_images/flight001`). SDK funguje identicky na oboch platformách.
{% endhint %}

### Plná kontrola

Pre pokročilé pracovné postupy:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

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

## Inštalačná príručka

### Predpoklady

Pred inštaláciou SDK sa uistite, že máte:

1. **Chloros nainštalovaný** — Windows: Inštalačný program pre počítače ([stiahnuť](download.md)); Linux: balík `.deb` ([Inštalácia](linux/linux-installation.md))
2. **Python 3.7+** nainštalované ([python.org](https://www.python.org))
3. **Aktívna licencia Chloros+** ([aktualizácia](https://cloud.mapir.camera/pricing))

### Inštalácia cez pip

**Štandardná inštalácia:**

```bash
pip install chloros-sdk
```

**S podporou sledovania priebehu:**

```bash
pip install chloros-sdk[progress]
```

**Vývojová inštalácia:**

```bash
pip install chloros-sdk[dev]
```

### Overenie inštalácie

Otestujte, či je SDK správne nainštalovaný:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Prvé nastavenie

### Aktivácia licencie

SDK používa rovnakú licenciu ako Chloros, Chloros (prehliadač) a Chloros CLI. Aktivujte raz prostredníctvom grafického rozhrania alebo CLI:**Windows:**Otvorte**Chloros alebo Chloros (prehliadač)** a prihláste sa na karte <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> alebo použite CLI.**Linux:** Použite CLI (GUI nie je k dispozícii):

```bash
chloros-cli login user@example.com 'your_password'
```

Licencia je uložená v lokálnej cache a pretrváva aj po reštarte.

{% hint style="success" %}
**Jednorazové nastavenie**: Po prihlásení cez grafické rozhranie alebo CLI, SDK automaticky použije uloženú licenciu. Nie je potrebné žiadne ďalšie overenie!
{% endhint %}

{% hint style="info" %}
**Odhlásenie**: Používatelia SDK môžu programovo vymazať uložené prihlasovacie údaje pomocou metódy `logout()`. Pozrite si [metódu logout()](#logout) v referencii API.
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

## Referencia API

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

| Parameter                 | Typ | Predvolené                   | Popis                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL lokálneho backendu Chloros          |
| `auto_start_backend`      | bool | `True`                    | Automaticky spustiť backend v prípade potreby |
| `backend_exe`             | str  | `None` (automatická detekcia)      | Cesta k spustiteľnému súboru backendu            |
| `timeout`                 | int  | `30`                      | Časový limit požiadavky v sekundách            |
| `backend_startup_timeout` | int  | `60`                      | Časový limit na spustenie backendu (sekúnd) |

**Príklady:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**Automatická detekcia platformy**: SDK automaticky vyskúša správnu cestu k backendu pre vašu platformu:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (ručné)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### Metódy

#### `create_project(project_name, camera=None)`

Vytvorí nový projekt Chloros.

**Parametre:**

| Parameter      | Typ | Povinné | Popis                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Áno      | Názov projektu                                     |
| `camera`       | str  | Nie       | Šablóna kamery (napr. „Survey3N\_RGN“, „Survey3W\_OCN“) |

**Vrátené hodnoty:** `dict` – Odpoveď na vytvorenie projektu**Príklad:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Import obrázkov zo zložky.

**Parametre:**

| Parameter     | Typ     | Povinné | Popis                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Áno      | Cesta k zložke s obrázkami         |
| `recursive`   | bool     | Nie       | Vyhľadávať podzložky (predvolené: False) |

**Vrátené hodnoty:** `dict` – Výsledky importu s počtom súborov**Príklad:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Nastavenie parametrov spracovania.

**Parametre:**

| Parameter                 | Typ | Predvolené                 | Popis                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;Štandardné (rýchle, stredná kvalita)&quot; | Metóda debayeringu            |
| `vignette_correction`     | bool | `True`                  | Povoliť korekciu vinetácie      |
| `reflectance_calibration` | bool | `True`                  | Povoliť kalibráciu odrazivosti  |
| `indices`                 | zoznam | `None`                  | Vegetácia indexy na výpočet |
| `export_format`           | reťazec  | &quot;TIFF (16-bit)&quot;         | Výstupný formát                   |
| `ppk`                     | bool | `False`                 | Povoliť korekcie PPK          |
| `custom_settings`         | dict | `None`                  | Pokročilé vlastné nastavenia        |

**Formáty exportu:**

* `"TIFF (16-bit)"` - Odporúčané pre GIS/fotogrametriu
* `"TIFF (32-bit, Percent)"` - Vedecká analýza
* `"PNG (8-bit)"` - Vizuálna kontrola
* `"JPG (8-bit)"` - Komprimovaný výstup

**Dostupné indexy:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 a ďalšie.**Príklad:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
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

| Parameter           | Typ     | Predvolené      | Popis                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Režim spracovania: „parallel“ alebo „serial“   |
| `wait`              | bool     | `True`       | Čakanie na dokončenie                       |
| `progress_callback` | callable | `None`       | Funkcia spätného volania priebehu (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Interval dotazovania na priebeh (sekundy)   |

**Vrátené hodnoty:** `dict` - Výsledky spracovania

{% hint style="warning" %}
**Paralelný režim**: Vyžaduje licenciu Chloros+. Automaticky sa prispôsobí vašim jadrám CPU (až 16 pracovných procesov).
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

**Vráti:** `dict` - Aktuálna konfigurácia projektu**Príklad:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Získa informácie o stave backendu vrátane priebehu spracovania na úrovni jednotlivých vlákien.

**Vrátené hodnoty:** `dict` – Stav backendu s nasledujúcou štruktúrou:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**Príklad:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

Vypne backend (ak bol spustený pomocou SDK).

**Príklad:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

Vymazanie uložených prihlasovacích údajov z lokálneho systému.

**Popis:**

Programovo sa odhlási odstránením overovacích údajov uložených v cache. To je užitočné pre:
* Prepínanie medzi rôznymi účtami Chloros+
* Vymazanie overovacích údajov v automatizovaných prostrediach
* Bezpečnostné účely (napr. odstránenie overovacích údajov pred odinštalovaním)

**Vrátené hodnoty:** `dict` – Výsledok operácie odhlásenia**Príklad:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**Vyžaduje sa opätovná autentifikácia**: Po volaní `logout()` sa musíte znovu prihlásiť prostredníctvom Chloros, Chloros (prehliadač) alebo Chloros CLI pred použitím SDK.
{% endhint %}

***

### Pomocné funkcie

#### `process_folder(folder_path, **options)`

Jednořádková pomocná funkcia na spracovanie priečinka.

**Parametre:**

| Parameter                 | Typ     | Predvolené         | Popis                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Povinné        | Cesta k zložke s obrázkami     |
| `project_name`            | str      | Automaticky generované  | Názov projektu                   |
| `camera`                  | str      | `None`          | Šablóna fotoaparátu                |
| `indices`                 | zoznam     | `["NDVI"]`      | Indexy na výpočet           |
| `vignette_correction`     | bool     | `True`          | Povoliť korekciu vinetácie     |
| `reflectance_calibration` | bool     | `True`          | Povoliť kalibráciu odrazivosti |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | Formát výstupu                  |
| `mode`                    | str      | `"parallel"`    | Režim spracovania                |
| `progress_callback`       | volateľný | `None`          | Spätné volanie priebehu              |

**Vrátené hodnoty:** `dict` - Výsledky spracovania**Príklad:**

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

{% hint style="info" %}
**Používatelia Linux**: Všetky nižšie uvedené príklady používajú cesty Windows. Nahraďte cesty `C:\\...` svojimi cestami Linux (napr. `/home/user/...` alebo `~/...`). Všetky funkcie SDK sú na všetkých platformách identické.
{% endhint %}

### Príklad 1: Základné spracovanie

Spracujte priečinok s predvolenými nastaveniami:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Príklad 2: Vlastný pracovný postup

Úplná kontrola nad spracovateľským potrubím:

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
    debayer="Standard (Fast, Medium Quality)",
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

### Príklad 3: Hromadné spracovanie viacerých zložiek

Spracovanie viacerých súborov údajov z letov:

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
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
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

### Príklad 7: Správa účtov a odhlásenie

Programové spravovanie prihlasovacích údajov:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### Príklad 8: Nástroj pre príkazový riadok

Vytvorenie vlastného nástroja CLI pomocou SDK:

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
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
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
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
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
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

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

### Nezablokujúce spracovanie

Spustite spracovanie a pokračujte v iných úlohách:

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

Pri veľkých dátových súboroch spracúvajte v dávkach:

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

**Problém:** SDK nedokáže spustiť backend**Riešenia:**

1. Overte, či je nainštalovaný Chloros:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Skontrolujte firewall (Windows) alebo dostupnosť portu (Linux: `lsof -i :5000`)
3. Skúste manuálnu cestu k backendu:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### Licencia nebola zistená**Problém:** SDK upozorňuje na chýbajúcu licenciu**Riešenia:**

1. Otvorte Chloros, Chloros (prehliadač) alebo Chloros CLI a prihláste sa.
2. Overte, či je licencia uložená v pamäti cache:

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. Ak máte problémy s prihlasovacími údajmi, vymažte uložené prihlasovacie údaje a prihláste sa znovu:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Kontaktujte podporu: info@mapir.camera

***

### Chyby importu**Problém:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Riešenia:**

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

### Časový limit spracovania**Problém:** Časový limit spracovania vypršal**Riešenia:**

1. Zvýšte časový limit:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Spracúvajte menšie dávky
3. Skontrolujte voľné miesto na disku
4. Sledujte systémové zdroje

***

### Port je už obsadený**Problém:** Port 5000 na strane servera je obsadený**Riešenia:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Alebo vyhľadajte a ukončite konfliktný proces:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
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

4. **Spracúvajte na SSD** (nie na HDD)***

### Optimalizácia pamäte

Pre veľké dátové súbory:

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

### Integrácia s Djangom

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

**Odpoveď:** Iba na počiatočnú aktiváciu licencie. Po prihlásení cez Chloros, Chloros (prehliadač) alebo Chloros CLI sa licencia uloží do lokálnej cache a funguje offline po dobu 30 dní.***

### Otázka: Môžem používať SDK na serveri bez grafického rozhrania?**Odpoveď:** Áno! SDK funguje bez grafického rozhrania na serveroch Windows aj Linux.**Linux (odporúčané pre bezhlavý režim):**
* Inštalácia prostredníctvom balíka `.deb`
* Aktivácia licencie: `chloros-cli login user@example.com 'password'`

**Server Windows:**
* Server Windows 2016 alebo novší
* Nainštalovaný Chloros (jednorazovo)
* Licencia aktivovaná prostredníctvom CLI alebo na akomkoľvek počítači

***

### Otázka: Aký je rozdiel medzi Desktop, CLI a SDK?

| Funkcia         | Grafické rozhranie Desktop | Príkazový riadok CLI | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Rozhranie**   | Ukazovateľom | Príkazový riadok | Python API  |
| **Najvhodnejšie pre**    | Vizuálnu prácu | Skriptovanie        | Integráciu |
| **Automatizácia**  | Obmedzená     | Dobrá             | Vynikajúca   |
| **Flexibilita** | Základná       | Dobrá             | Maximálna     |
| **Licencia**     | Chloros+    | Chloros+         | Chloros+    |***

### Otázka: Môžem distribuovať aplikácie vytvorené pomocou SDK?**Odpoveď:** Kód SDK je možné integrovať do vašich aplikácií, ale:

* Koneční používatelia musia mať nainštalovaný Chloros
* Koneční používatelia potrebujú aktívne licencie Chloros+
* Komerčná distribúcia vyžaduje OEM licenciu

V prípade otázok týkajúcich sa OEM kontaktujte info@mapir.camera.

***

### Otázka: Ako aktualizujem SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### Otázka: Kam sa ukladajú spracované obrázky?

V predvolenom nastavení do projektovej cesty:

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### Otázka: Môžem spracovávať obrázky zo skriptov Python spúšťaných podľa plánu?**Odpoveď:** Áno! Použite plánovač vášho operačného systému so skriptmi Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** Naplánujte prostredníctvom Plánovača úloh tak, aby sa spúšťal denne.**Linux:** Naplánujte prostredníctvom cron:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### Otázka: Podporuje SDK async/await?**Odpoveď:** Aktuálna verzia je synchronná. Pre asynchrónne správanie použite `wait=False` alebo spustite v samostatnom vlákne:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### Otázka: Ako môžem prepínať medzi rôznymi účtami Chloros+?**Odpoveď:** Použite metódu `logout()` na vymazanie uložených prihlasovacích údajov v cache a potom sa znovu prihláste s novým účtom:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

Po odhlásení sa overte pomocou nového účtu cez grafické rozhranie, prehliadač alebo CLI, než znovu použijete SDK.

***

## Pomoc

### Dokumentácia

* **Referencia API**: Táto stránka

### Kanály podpory

* **E-mail**: info@mapir.camera
* **Webová stránka**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Ceny**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Ukážkový kód

Všetky tu uvedené príklady sú otestované a pripravené na použitie v produkčnom prostredí. Skopírujte ich a prispôsobte svojim potrebám.

***

## Licencia**Proprietárny softvér** – Copyright (c) 2025 MAPIR Inc.

SDK vyžaduje aktívne predplatné Chloros+. Neoprávnené používanie, distribúcia alebo úprava sú zakázané.

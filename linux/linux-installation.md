# Inštalácia Linux

Chloros sa distribuuje pre Linux vo forme balíkov `.deb`, ktoré inštalujú CLI a backend. Python SDK sa inštaluje samostatne prostredníctvom pip.

***

## Linux amd64 (x86_64)

### Systémové požiadavky

| Požiadavka | Minimálna | Odporúčaná |
| --- | --- | --- |
| **Distribúcia** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Procesor** | x86_64 (Intel/AMD) | Intel Core i7 alebo lepší |
| **Pamäť (RAM)** | 8 GB | 16 GB alebo viac |
| **Grafická karta** | Žiadna (spracovanie CPU) | NVIDIA GPU s 4 GB+ VRAM |
| **Úložisko** | 2 GB voľného miesta | SSD s 10 GB+ voľného miesta |
| **Python** | Python 3.7+ (pre SDK) | Python 3.10+ |

### Inštalácia

Stiahnite si balík `.deb` a nainštalujte ho:

```bash
sudo dpkg -i chloros-amd64.deb
```

Overte inštaláciu:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Systémové požiadavky

| Požiadavka | Minimálna | Odporúčaná |
| --- | --- | --- |
| **Platforma** | NVIDIA Jetson s JetPack 6 | Jetson Orin NX 16 GB alebo AGX Orin |
| **JetPack** | JetPack 6.x | Najnovší JetPack 6 |
| **Pamäť (RAM)** | 8 GB (zdieľaná GPU/CPU) | 16 GB+ zdieľaná |
| **Úložisko** | 2 GB voľného miesta | NVMe SSD s 10 GB+ voľného miesta |
| **Python** | Python 3.7+ (pre SDK) | Python 3.10+ |

### Inštalácia

Stiahnite si balík JetPack 6 `.deb` a nainštalujte ho:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Overte inštaláciu:

```bash
chloros-cli --version
```

Podrobné informácie o nastavení Jetsonu vrátane riadenia teploty a nasadenia v teréne nájdete v [príručke NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Inštalácia Python SDK (všetko Linux)

Python SDK sa inštaluje samostatne prostredníctvom pip a funguje na architektúrach amd64 aj arm64:

```bash
pip install chloros-sdk
```

Ak chcete zahrnúť voliteľnú podporu streamovania priebehu:

```bash
pip install chloros-sdk[progress]
```

Overte SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Balík `.deb` inštaluje Chloros, CLI a backend. Python SDK je samostatný balík pip, ktorý komunikuje s backendom prostredníctvom lokálneho HTTP API.
{% endhint %}

***

## Konfiguračné adresáre

Chloros na Linux sa riadi [špecifikáciou základného adresára XDG](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Účel | Linux Cesta | Windows Ekvivalent |
| --- | --- | --- |
| **Konfigurácia** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Dáta / Projekty** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Cache / Prihlasovacie údaje** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Umiestnenia spustiteľných súborov backendu

Balík `.deb` inštaluje backend do štandardného umiestnenia. Balíky CLI a SDK automaticky zistia cestu k backendu:

| Spôsob inštalácie | Cesta k backendu |
| --- | --- |
| Balík `.deb` | `/usr/lib/chloros/chloros-backend` |
| Ručná / vlastná | `/opt/mapir/chloros/backend/chloros-backend` |

Cestu k backendu môžete prepisovať pomocou príznaku `--backend-exe` CLI alebo parametra konštruktora `backend_exe` SDK.

***

## Prvé nastavenie

### 1. Aktivujte svoju licenciu

Pre prístup k CLI a SDK je potrebná licencia Chloros+:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Skontrolujte stav svojej licencie

```bash
chloros-cli status
```

### 3. Spracujte svoj prvý dátový súbor

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Spustite diagnostiku systému

Overte, či je váš systém správne nakonfigurovaný:

```bash
chloros-cli selftest
```

Toto spustí 7 diagnostických kontrol vrátane verzie, spustenia backendu, pripojiteľnosti API a dostupnosti CUDA/GPU.

***

## Príklady skriptov v Bash

### Spracovanie viacerých dátových súborov

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Spracovanie s vlastnými nastaveniami

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Automatizované spracovanie pomocou Cron

Pridajte do svojho crontabu (`crontab -e`) na automatické spracovanie nových dátových súborov:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Príklad

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Riešenie problémov

### CLI Nenašiel sa po inštalácii

Ak sa po inštalácii balíka `.deb` nenájde `chloros-cli`:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Odmietnuté povolenie

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### Spustenie backendu zlyhalo

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### CUDA nebola zistená

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Chýbajúce zdieľané knižnice

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Aktualizácia Chloros na Linux

Použite integrovaný príkaz na aktualizáciu, aby ste skontrolovali a nainštalovali aktualizácie:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Ďalšie kroky

* [Príručka NVIDIA Jetson](nvidia-jetson-guide.md) — Optimalizácia a nasadenie špecifické pre Jetson
* [CLI : Príkazový riadok](../CLI.md) — Úplný referenčný zoznam príkazov CLI
* [API : Python SDK](../api-python-sdk.md) — Úplný referenčný zoznam SDK
* [Dynamická adaptácia výpočtov](../processing-architecture/dynamic-compute-adaptation.md) — Ako sa Chloros prispôsobuje vášmu hardvéru

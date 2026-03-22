# CLI : Príkazový riadok

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** poskytuje výkonný prístup cez príkazový riadok k modulu na spracovanie obrazu Chloros, čím umožňuje automatizáciu, skriptovanie a bezobslužnú prevádzku vašich pracovných postupov v oblasti spracovania obrazu.

### Kľúčové funkcie

* 🚀 **Automatizácia** – Skriptové dávkové spracovanie viacerých dátových súborov
* 🔗 **Integrácia** – Vloženie do existujúcich pracovných postupov a potrubí
* 💻 **Prevádzka bez grafického rozhrania** – Spustenie bez grafického rozhrania
* 🌍 **Viacjazyčnosť** – Podpora 38 jazykov
* ⚡ **Paralelné spracovanie** – [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) sa automaticky optimalizuje pre váš hardvér

### Požiadavky

| Požiadavka          | Podrobnosti                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operačný systém** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Licencia**          | Chloros+ ([vyžaduje sa platený plán](https://cloud.mapir.camera/pricing)) |
| **Pamäť**           | Minimálne 8 GB RAM (odporúča sa 16 GB)                                  |
| **Internet**         | Vyžaduje sa na aktiváciu licencie                                     |
| **Miesto na disku**       | Záleží od veľkosti projektu                                              |

{% hint style="warning" %}
**Požiadavky na licenciu**: CLI vyžaduje platené predplatné Chloros+. Štandardné (bezplatné) plány nemajú prístup k CLI. Pre upgrade navštívte [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).
{% endhint %}

## Rýchly štart

### Inštalácia

#### Windows

CLI je automaticky súčasťou inštalačného programu Chloros:

1. Stiahnite a spustite **Chloros Installer.exe**

2. Dokončite inštalačného sprievodcu
3. CLI nainštalovaný do: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Inštalátor automaticky pridá `chloros-cli` do systémovej premennej PATH. Po inštalácii reštartujte terminál.
{% endhint %}

#### Linux

Nainštalujte balík `.deb` pre vašu architektúru:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Podrobné informácie o nastavení Linux nájdete v [Inštalácia Linux](linux/linux-installation.md).

### Prvé nastavenie

Pred použitím CLI aktivujte svoju licenciu Chloros+:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Základné použitie

Spracujte priečinok s predvolenými nastaveniami:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Referencia príkazov

### Všeobecná syntax

```
chloros-cli [global-options] <command> [command-options]
```

***

## Príkazy

### `process` – Spracovanie obrázkov

Spracovanie obrázkov v priečinku s kalibráciou.

**Syntax:**

```bash
chloros-cli process <input-folder> [options]
```

**Príklady:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Možnosti príkazu spracovania

| Možnosť                | Typ    | Predvolené nastavenie        | Popis                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Cesta    | _Povinné_     | Zložka obsahujúca multispektrálne obrázky vo formáte RAW/JPG                                         |
| `-o, --output`        | Cesta    | Rovnaká ako vstup  | Výstupná zložka pre spracované obrázky                                                     |
| `-n, --project-name`  | Reťazec  | Automaticky generované | Vlastný názov projektu                                                                    |
| `--vignette`          | Príznak    | Zapnuté        | Zapnúť korekciu vinetácie                                                             |
| `--no-vignette`       | Príznak    | -              | Vypnúť korekciu vinetácie                                                            |
| `--reflectance`       | Príznak    | Zapnuté        | Zapnúť kalibráciu odrazivosti                                                         |
| `--no-reflectance`    | Príznak    | -              | Vypnúť kalibráciu odrazivosti                                                        |
| `--ppk`               | Príznak    | Vypnuté       | Použiť korekcie PPK z údajov svetelného senzora .daq                                      |
| `--format`            | Voľba  | TIFF (16-bit)  | Výstupný formát: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Celé číslo | Automaticky           | Minimálna veľkosť cieľa v pixeloch pre detekciu kalibračného panelu                          |
| `--target-clustering` | Celé číslo | Automaticky           | Prah zhlukovania cieľov (0–100)                                                    |
| `--debayer`           | Voľba  | `standard`     | Metóda debayeringu: `standard` alebo `texture-aware` (len Chloros+)                          |
| `--target`, `--targets` | Príznak  | Vypnuté       | Hľadať kalibračné ciele iba v podpriečinku „target“ alebo „targets“ (urýchľuje spracovanie) |
| `--indices`           | Zoznam    | Žiadne           | Vegetácia indexy na výpočet (napr. `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Reťazec  | Žiadne           | Zamknúť expozíciu pre model fotoaparátu (Pin 1)                                                 |
| `--exposure-pin-2`    | Reťazec  | Žiadne           | Uzamknutie expozície pre model kamery (pin 2)                                                 |
| `--recal-interval`    | Celé číslo | Automaticky           | Interval rekalibrácie v sekundách                                                      |
| `--timezone-offset`   | Celé číslo | 0              | Časový posun v hodinách                                                               |

***

### `login` – Overenie účtu

Prihláste sa pomocou svojich prihlasovacích údajov Chloros+, aby ste povolili spracovanie CLI.

**Syntax:**

```bash
chloros-cli login <email> <password>
```

**Príklad:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Špeciálne znaky**: Použite jednoduché úvodzovky okolo hesiel obsahujúcich znaky ako `$`, `!` alebo medzery.
{% endhint %}

**Výstup:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` – Vymazať prihlasovacie údaje

Vymazanie uložených prihlasovacích údajov a odhlásenie z vášho účtu.

**Syntax:**

```bash
chloros-cli logout
```

**Príklad:**

```bash
chloros-cli logout
```

**Výstup:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**SDK Používatelia**: Python SDK poskytuje aj programovú metódu `logout()` na vymazanie prihlasovacích údajov v skriptoch Python. Podrobnosti nájdete v [dokumentácii k Python SDK](api-python-sdk.md#logout).
{% endhint %}

***

### `status` – Kontrola stavu licencie

Zobrazí aktuálny stav licencie a overenia.

**Syntax:**

```bash
chloros-cli status
```

**Príklad:**

```bash
chloros-cli status
```

**Výstup:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` – Kontrola priebehu exportu

Monitoruje priebeh exportu vlákna 4 počas alebo po spracovaní.

**Syntax:**

```bash
chloros-cli export-status
```

**Príklad:**

```bash
chloros-cli export-status
```

**Prípad použitia:** Tento príkaz volajte počas spustenej operácie, aby ste skontrolovali priebeh exportu.***

### `language` – Správa jazyka rozhrania

Zobrazenie alebo zmena jazyka rozhrania CLI.

**Syntax:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Príklady:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Podporované jazyky (celkom 38)

| Kód    | Jazyk              | Názov v pôvodnom jazyku      |
| ------- | --------------------- | ---------------- |
| `en`    | Angličtina               | English          |
| `es`    | Španielčina               | Español          |
| `pt`    | Portugalčina            | Português        |
| `fr`    | francúzština                | Français         |
| `de`    | nemčina                | Deutsch          |
| `it`    | taliančina               | Italiano         |
| `ja`    | japončina              | 日本語              |
| `ko`    | Kórejčina                | 한국어              |
| `zh`    | Čínština (zjednodušená)  | 简体中文             |
| `zh-TW` | Čínština (tradičná) | 繁體中文             |
| `ru`    | ruština               | Русский          |
| `nl`    | holandčina                | Nederlands       |
| `ar`    | arabčina                | العربية          |
| `pl`    | poľština                | Polski           |
| `tr`    | Turečtina               | Türkçe           |
| `hi`    | Hindčina                 | हिंदी            |
| `id`    | Indonézština            | Bahasa Indonesia |
| `vi`    | Vietnamčina            | Tiếng Việt       |
| `th`    | Thajčina                  | ไทย              |
| `sv`    | Švédčina               | Svenska          |
| `da`    | Dánčina                | Dansk            |
| `no`    | Nórčina             | Norsk            |
| `fi`    | Fínčina               | Suomi            |
| `el`    | Gréčtina                 | Ελληνικά         |
| `cs`    | čeština                | Čeština          |
| `hu`    | maďarčina             | Magyar           |
| `ro`    | rumunčina              | Română           |
| `uk`    | ukrajinčina             | Українська       |
| `pt-BR` | brazílska portugalčina  | Português Brasileiro |
| `zh-HK` | kantónčina             | 粵語             |
| `ms`    | malajčina                 | Bahasa Melayu    |
| `sk`    | Slovenská                | Slovenčina       |
| `bg`    | Bulharská             | Български        |
| `hr`    | Chorvátska              | Hrvatski         |
| `lt`    | Litovčina            | Lietuvių         |
| `lv`    | Lotyština               | Latviešu         |
| `et`    | Estónčina              | Eesti            |
| `sl`    | Slovinčina             | Slovenščina      |

{% hint style="success" %}
**Automatické uchovanie**: Vaše jazykové nastavenie sa uloží do súboru `~/.chloros/cli_language.json` a zostane zachované počas všetkých relácií.
{% endhint %}

***

### `set-project-folder` - Nastaviť predvolenú zložku projektu

Zmeňte umiestnenie predvolenej zložky projektu (zdieľané s GUI na Windows).

**Syntax:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Príklady:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` – Zobraziť priečinok projektu

Zobrazí aktuálne umiestnenie predvoleného priečinka projektu.

**Syntax:**

```bash
chloros-cli get-project-folder
```

**Príklad:**

```bash
chloros-cli get-project-folder
```

**Výstup:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` – Obnoviť predvolené nastavenia

Obnoví predvolenú polohu priečinka projektu.

**Syntax:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` – Spustiť diagnostiku systému

Spustí 7 diagnostických kontrol na overenie konfigurácie systému.

**Syntax:**

```bash
chloros-cli selftest
```

**Vykonané diagnostické kontroly:**

1. Kontrola verzie
2. Dostupnosť portu (5000)
3. Spustenie backendu
4. Test pripojiteľnosti API
5. Informácie o systéme a detekcia GPU
6. Overenie modelov odšumovača
7. Kontrola dostupnosti CUDA

{% hint style="info" %}
**Užitočné pri riešení problémov**: Po inštalácii spustite `selftest`, aby ste overili, či je váš systém správne nakonfigurovaný, najmä na Linux/Jetson, kde môže byť potrebné overiť nastavenie GPU a CUDA.
{% endhint %}

***

### `update` – Kontrola aktualizácií (len Linux)

Skontrolujte a nainštalujte aktualizácie CLI na systémoch Linux.

**Syntax:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Možnosť    | Popis                        |
| --------- | ---------------------------------- |
| `--check` | Len vyhľadať aktualizácie, neinštalovať |

{% hint style="info" %}
Tento príkaz je k dispozícii iba v systéme Linux. V systéme Windows sa aktualizácie dodávajú prostredníctvom inštalátora.
{% endhint %}

***

## Globálne možnosti

Tieto možnosti platia pre všetky príkazy:

| Možnosť            | Typ    | Predvolené       | Popis                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Cesta    | Automaticky zistené | Cesta k spustiteľnému súboru backendu                       |
| `--port`          | Celé číslo | 5000          | Číslo portu backendu API                          |
| `--restart`       | Príznak    | -             | Vynútiť reštart backendu (ukončí existujúce procesy) |
| `--version`       | Príznak    | -             | Zobraziť informácie o verzii a ukončiť                |
| `--help`          | Príznak    | -             | Zobraziť informácie o pomoci a ukončiť                   |

{% hint style="info" %}
**Automatická detekcia backendu**: Cesta `--backend-exe` sa automaticky detekuje podľa platformy:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (ručné)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Príklad s globálnymi možnosťami:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Príručka nastavení spracovania

### Paralelné spracovanie a dynamická adaptácia výpočtov

Chloros 1.1.0 obsahuje [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) — spracovateľský engine **automaticky detekuje váš hardvér** a vyberie optimálnu stratégiu:

| Platforma | Stratégia | Pracovníci | Potrubie | Poznámky |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Úsporné na pamäť, sériové |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Súbežné spracovanie na GPU |
| **Stolný počítač s 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Dobrý výkon stolného počítača |
| **Stolný počítač s 12 GB+ GPU** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Optimálny výkon stolného počítača |
| **Systém len s procesorom** | `CPU_PARALLEL` | jadrá – 1 | `cpu_fallback` | Grafická karta nie je potrebná |

{% hint style="success" %}
**Nie je potrebná žiadna manuálna konfigurácia!** Chloros automaticky detekuje váš procesor, grafickú kartu, pamäť RAM a (na Jetsone) teplotné senzory, a následne automaticky nakonfiguruje optimálny spracovateľský reťazec.
{% endhint %}

### Metódy debayeringu

| Metóda | CLI Príznak | Kvalita | Rýchlosť | Licencia |
| --- | --- | --- | --- | --- |
| **Štandardná (rýchla, stredná kvalita)** | `--debayer standard` | Dobrá | Rýchla | Bezplatná / Chloros+ |
| **S ohľadom na textúru (pomalá, najvyššia kvalita)** | `--debayer texture-aware` | Najvyššia | Pomalá | Iba Chloros+ |

Predvolená metóda debayeringu je **Štandardná**. Metóda**S ohľadom na textúru** využíva model odšumovania AI/ML pre výstup v najvyššej kvalite, vyžaduje však licenciu Chloros+ a grafickú kartu NVIDIA.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Korekcia vinetácie

**Čo robí:** Koriguje pokles intenzity svetla na okrajoch obrazu (tmavšie rohy bežné na snímkach z fotoaparátov).

* **V predvolenom nastavení zapnuté** – Väčšina používateľov by mala túto funkciu nechať zapnutú
* Použite `--no-vignette` na vypnutie

{% hint style="success" %}
**Odporúčanie**: Vždy zapínajte korekciu vinetácie, aby ste zabezpečili rovnomernú jasnosť v celom zábere.
{% endhint %}

### Kalibrácia odrazivosti

Prevádza surové hodnoty senzora na štandardizované percentá odrazivosti pomocou kalibračných panelov.

* **Zapnuté štandardne** – nevyhnutné pre analýzu vegetácie
* Vyžaduje kalibračné cieľové panely v snímkach
* Použite `--no-reflectance` na vypnutie

{% hint style="info" %}
**Požiadavky**: Uistite sa, že kalibračné panely sú správne exponované a viditeľné vo vašich snímkach, aby bola konverzia odrazivosti presná.
{% endhint %}

### Korekcie PPK

**Čo robí:** Uplatňuje kinematické korekcie po spracovaní pomocou údajov z protokolu DAQ-A-SD na zlepšenie presnosti GPS.

* **V predvolenom nastavení je vypnutá**
* Použite `--ppk` na zapnutie
* Vyžaduje súbory .daq v projektovej zložke zo senzora osvetlenia DAQ-A-SD MAPIR.

### Výstupné formáty

<table><thead><tr><th width="197">Formát</th><th width="130.20001220703125">Hĺbka bitov</th><th width="116.5999755859375">Veľkosť súboru</th><th>Najvhodnejšie pre</th></tr></thead><tbody><tr><td><strong>TIFF (16-bitový)</strong> ⭐</td><td>16-bitové celé číslo</td><td>Veľké</td><td>GIS analýza, fotogrametria (odporúčané)</td></tr><tr><td><strong>TIFF (32-bitové, percentá)</strong></td><td>32-bitové číslo s pohyblivou desatinnou čiarkou</td><td>Veľmi veľké</td><td>Vedecká analýza, výskum</td></tr><tr><td><strong>PNG (8-bitový)</strong></td><td>8-bitové celé číslo</td><td>Stredná</td><td>Vizuálna kontrola, zdieľanie na webe</td></tr><tr><td><strong>JPG (8-bitové)</strong></td><td>8-bitové celé číslo</td><td>Malé</td><td>Rýchly náhľad, komprimovaný výstup</td></tr></tbody></table>***

## Automatizácia a skriptovanie

### Hromadné spracovanie v PowerShell (Windows)

Automatické spracovanie viacerých zložiek s dátovými súbormi v Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Dávkový skript Windows (Windows)

Jednoduchá slučka pre dávkové spracovanie na Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Dávkové spracovanie v Bash (Linux)

Spracovanie viacerých zložiek s dátovými súbormi na Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Automatizačný skript Python (multiplatformový)

Pokročilá automatizácia s riešením chýb (funguje na Windows a Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Pracovný postup spracovania

### Štandardný pracovný postup

1. **Vstup**: Zložka obsahujúca páry obrázkov vo formátoch RAW/JPG
2. **Vyhľadávanie**: CLI automaticky vyhľadáva podporované obrazové súbory
3. **Spracovanie**: Paralelný režim sa prispôsobuje počtu jadier vášho procesora (Chloros+)
4. **Výstup**: Vytvorí podzložky podľa modelov fotoaparátov so spracovanými obrázkami

### Príklad štruktúry výstupu

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Odhady času spracovania

Typické časy spracovania pre 100 obrázkov (každý s rozlíšením 12 MP):

| Platforma | Režim | Odhadovaný čas | Poznámky |
| --- | --- | --- | --- |
| **Stolný počítač s GPU 12 GB+** | `GPU_PARALLEL` | 5–10 min | Najrýchlejšia možnosť |
| **Stolný počítač s GPU 8 GB** | `GPU_SINGLE` | 10–15 min | Dobrý výkon |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 15–25 min | Edge computing |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 30–60 min | Obmedzená pamäť |
| **Iba CPU** | `CPU_PARALLEL` | 20–40 min | Nie je potrebná GPU |

{% hint style="info" %}
**Tip na zvýšenie výkonu**: Doba spracovania sa líši v závislosti od počtu obrázkov, rozlíšenia, metódy debayeringu a hardvéru. Debayering s ohľadom na textúru trvá podstatne dlhšie ako štandardný. Podrobnosti nájdete v časti [Dynamická adaptácia výpočtov](processing-architecture/dynamic-compute-adaptation.md).
{% endhint %}

***

## Riešenie problémov

### CLI nenájdené

**Windows Chyba:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Riešenia:**

1. Overte umiestnenie inštalácie:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Ak nie je v PATH, použite úplnú cestu:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Pridajte do PATH ručne:
   * Otvorte Vlastnosti systému → Premenné prostredia
   * Upravte premennú PATH
   * Pridajte: `C:\Program Files\Chloros\resources\cli`
   * Reštartujte terminál

**Linux Chyba:**

```
chloros-cli: command not found
```

**Linux Riešenia:**

1. Overte inštaláciu:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Načítajte shell znovu:

```bash
source ~/.bashrc
```

3. Skontrolujte oprávnenia:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Spustenie backendu zlyhalo**Chyba:**

```

Backend failed to start within 30 seconds
```

**Riešenia:**

1. Skontrolujte, či backend už beží (najprv ho zatvorte)
2. Skontrolujte, či ho neblokuje firewall (Windows) alebo skontrolujte dostupnosť portu (Linux: `lsof -i :5000`)
3. Skúste iný port:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Vynúťte reštart backendu:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. Na Linux skontrolujte, či existuje spustiteľný súbor backendu:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Problémy s licenciou / overovaním**Chyba:**

```

Chloros+ license required for CLI access
```

**Riešenia:**

1. Overte si, či máte aktívne predplatné Chloros+
2. Prihláste sa pomocou svojich prihlasovacích údajov:

```bash
chloros-cli login user@example.com 'password'
```

3. Skontrolujte stav licencie:

```bash
chloros-cli status
```

4. Kontaktujte podporu: info@mapir.camera

***

### Nenašli sa žiadne obrázky**Chyba:**

```

No images found in the specified folder
```

**Riešenia:**

1. Overte, či priečinok obsahuje podporované formáty (.RAW, .TIF, .JPG)
2. Skontrolujte, či je cesta k priečinku správna (pri cestách s medzerami použite úvodzovky)
3. Uistite sa, že máte oprávnenie na čítanie priečinka
4. Skontrolujte, či sú prípony súborov správne

***

### Spracovanie sa zastaví alebo zamrzne**Riešenia:**

1. Skontrolujte voľné miesto na disku (uistite sa, že je dostatočné pre výstup)
2. Zatvorte ostatné aplikácie, aby ste uvoľnili pamäť
3. Znížte počet obrázkov (spracúvajte v dávkach)

***

### Port je už používaný**Chyba:**

```

Port 5000 is already in use
```

**Riešenia:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## Často kladené otázky

### Otázka: Potrebujem licenciu pre CLI?

**Odpoveď:**Áno! CLI vyžaduje platenú**licenciu Chloros+**.

* ❌ Štandardný (bezplatný) plán: CLI je deaktivovaný
* ✅ Plány Chloros+ (platené): CLI je plne aktivovaný

Prihláste sa na: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Otázka: Môžem používať CLI na serveri bez grafického rozhrania?**Odpoveď:** Áno! CLI beží úplne bez grafického rozhrania. Toto je hlavný prípad použitia na Linux.**Windows Server:**
* Windows Server 2016 alebo novší
* Nainštalovaný Visual C++ Redistributable

**Linux Server:**
* Ubuntu 20.04+ / Debian 11+ (amd64) alebo JetPack 6 (arm64)
* Inštalácia prostredníctvom balíka `.deb`

**Obe platformy:**
* Minimálne 8 GB RAM (odporúča sa 16 GB)
* Jednorazová aktivácia licencie: `chloros-cli login user@example.com 'password'`

***

### Otázka: Kam sa ukladajú spracované obrázky?**Odpoveď:**V predvolenom nastavení sa spracované obrázky ukladajú do**tej istej zložky ako vstupné súbory** v podzložkách podľa modelu fotoaparátu (napr. `Survey3N_RGN/`).

Pomocou možnosti `-o` môžete určiť iný výstupný priečinok:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### Otázka: Môžem spracovať viacero priečinkov naraz?**A:** Nie priamo jedným príkazom, ale môžete použiť skriptovanie na postupné spracovanie zložiek. Pozrite si časť [Automatizácia a skriptovanie](CLI.md#automation--scripting).***

### Otázka: Ako uložím výstup CLI do súboru protokolu?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### Otázka: Čo sa stane, ak počas spracovania stlačím Ctrl+C?**Odpoveď:** CLI:

1. Správne zastaví spracovanie
2. Vypne backend
3. Ukončí sa s kódom 130

Čiastočne spracované obrázky môžu zostať vo výstupnej zložke.

***

### Otázka: Môžem automatizovať spracovanie CLI?**Odpoveď:** Samozrejme! CLI je navrhnutý pre automatizáciu. Pozrite si [Automatizácia a skriptovanie](CLI.md#automation--scripting) pre PowerShell (Windows), Batch (Windows), Bash (Linux) a Python (multiplatformové).***

### Otázka: Ako skontrolujem verziu CLI?**Odpoveď:**

```bash
chloros-cli --version
```

**Výstup:**

```

Chloros CLI 1.1.0
```

***

## Získanie pomoci

### Pomoc na príkazovom riadku

Zobrazenie informácií o pomoci priamo v CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Kanály podpory

* **E-mail**: info@mapir.camera
* **Webová stránka**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Ceny**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Kompletné príklady

### Príklad 1: Základné spracovanie

Spracovanie s predvolenými nastaveniami (vignette, reflectance):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Príklad 2: Vysoko kvalitný vedecký výstup

32-bitové číslo s pohyblivou desatinnou čiarkou TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Príklad 3: Rýchle spracovanie náhľadu

8-bitový PNG bez kalibrácie pre rýchle prehliadanie:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Príklad 4: Spracovanie s korekciou PPK

Uplatnenie korekcií PPK s odrazivosťou:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Príklad 5: Vlastné umiestnenie výstupu

Spracujte do iného umiestnenia s konkrétnym formátom:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Príklad 6: Pracovný postup overovania

Kompletný postup overovania (rovnaký na všetkých platformách):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Príklad 7: Použitie viacerých jazykov

Zmena jazyka rozhrania (rovnaká na všetkých platformách):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```

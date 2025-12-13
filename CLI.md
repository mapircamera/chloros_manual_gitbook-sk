# CLI : Príkazový riadok

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** poskytuje výkonný prístup cez príkazový riadok k obrazovému procesorovému jadru Chloros, čo umožňuje automatizáciu, skriptovanie a bezhlavú prevádzku vašich pracovných postupov spracovania obrazu.

### Kľúčové funkcie

* 🚀 **Automatizácia** – skriptové hromadné spracovanie viacerých dátových súborov
* 🔗 **Integrácia** – vloženie do existujúcich pracovných postupov a potrubí
* 💻 **Prevádzka bez grafického rozhrania** – prevádzka bez grafického rozhrania
* 🌍 **Viacjazyčnosť** – podpora 38 jazykov
* ⚡ **Paralelné spracovanie** – dynamické škálovanie podľa výkonu vášho procesora (až 16 paralelných pracovných procesov)

### Požiadavky

| Požiadavka          | Podrobnosti                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operačný systém** | Windows 10/11 (64-bit)                                              |
| **Licencia**          | Chloros+ ([vyžaduje sa platený plán](https://cloud.mapir.camera/pricing)) |
| **Pamäť**           | Minimálne 8 GB RAM (odporúča sa 16 GB)                                  |
| **Internet**         | Vyžaduje sa na aktiváciu licencie                                     |
| **Miesto na disku**       | Závisí od veľkosti projektu                                              |

{% hint style=&quot;warning&quot; %}
**Požiadavky na licenciu**: CLI vyžaduje platené predplatné Chloros+. Štandardné (bezplatné) plány nemajú prístup k CLI. Prejdite na stránku [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), aby ste vykonali aktualizáciu.
{% endhint %}

## Rýchly štart

### Inštalácia

CLI je automaticky súčasťou inštalačného programu Chloros:

1. Stiahnite a spustite **Chloros Installer.exe**
2. Dokončite inštalačného sprievodcu
3. CLI nainštalovaný do: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Inštalačný program automaticky pridá `chloros-cli` do systémovej cesty PATH. Po inštalácii reštartujte terminál.
{% endhint %}

### Prvé nastavenie

Pred použitím CLI aktivujte svoju licenciu Chloros+:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Základné použitie

Spracujte priečinok s predvolenými nastaveniami:

```powershell
chloros-cli process "C:\Images\Dataset001"
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

**Príklad:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Možnosti príkazu spracovania

| Možnosť                | Typ    | Predvolené        | Popis                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Cesta    | _Povinné_     | Zložka obsahujúca multispektrálne obrázky RAW/JPG                                         |
| `-o, --output`        | Cesta    | Rovnaká ako vstup  | Výstupná zložka pre spracované obrázky                                                     |
| `-n, --project-name`  | Reťazec  | Automaticky generované | Vlastný názov projektu                                                                    |
| `--vignette`          | Príznak    | Povolené        | Povoliť korekciu vinetácie                                                             |
| `--no-vignette`       | Príznak    | -              | Zakázať korekciu vinetácie                                                            |
| `--reflectance`       | Príznak    | Povolené        | Povoliť kalibráciu odrazivosti                                                         |
| `--no-reflectance`    | Príznak    | -              | Zakázať kalibráciu odrazivosti                                                        |
| `--ppk`               | Príznak    | Zakázané       | Použiť korekcie PPK z údajov svetelného senzora .daq                                      |
| `--format`            | Voľba  | TIFF (16-bit)  | Výstupný formát: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Celé číslo | Automaticky           | Minimálna cieľová veľkosť v pixeloch pre detekciu kalibračného panela                          |
| `--target-clustering` | Celé číslo | Automaticky           | Prah zhlukovania cieľov (0-100)                                                    |
| `--exposure-pin-1`    | Reťazec  | Žiadne           | Uzamknutie expozície pre model kamery (pin 1)                                                 |
| `--exposure-pin-2`    | Reťazec  | Žiadne           | Uzamknutie expozície pre model kamery (pin 2)                                                 |
| `--recal-interval`    | Celé číslo | Automaticky           | Interval rekalibrácie v sekundách                                                      |
| `--timezone-offset`   | Celé číslo | 0              | Posun časového pásma v hodinách                                                               |

***

### `login` – Overenie účtu

Prihláste sa pomocou svojich prihlasovacích údajov Chloros+, aby ste povolili spracovanie CLI.

**Syntax:**

```bash
chloros-cli login <email> <password>
```

**Príklad:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Špeciálne znaky**: Použite jednoduché úvodzovky okolo hesiel obsahujúcich znaky ako `$`, `!` alebo medzery.
{% endhint %}

**Výstup:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` – Vymazať prihlasovacie údaje

Vymazanie uložených prihlasovacích údajov a odhlásenie z vášho účtu.

**Syntax:**

```bash
chloros-cli logout
```

**Príklad:**

```powershell
chloros-cli logout
```

**Výstup:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` – Skontrolujte stav licencie

Zobrazí aktuálny stav licencie a overenia.

**Syntax:**

```bash
chloros-cli status
```

**Príklad:**

```powershell
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

Monitorovanie priebehu exportu vlákna 4 počas alebo po spracovaní.

**Syntax:**

```bash
chloros-cli export-status
```

**Príklad:**

```powershell
chloros-cli export-status
```

**Prípad použitia:** Vyvolajte tento príkaz počas spracovania, aby ste skontrolovali priebeh exportu.

***

### `language` – Správa jazyka rozhrania

Zobraziť alebo zmeniť jazyk rozhrania CLI.

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

```powershell
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

| Kód    | Jazyk              | Rodný názov      |
| ------- | --------------------- | ---------------- |
| `en`    | Angličtina               | English          |
| `es`    | Španielčina               | Español          |
| `pt`    | Portugalčina            | Português        |
| `fr`    | Francúzština                | Français         |
| `de`    | Nemčina                | Deutsch          |
| `it`    | Taliančina               | Italiano         |
| `ja`    | Japončina              | 日本語              |
| `ko`    | Kórejčina                | 한국어              |
| `zh`    | Čínština (zjednodušená)  | 简体中文             |
| `zh-TW` | Čínština (tradičná) | 繁體中文             |
| `ru`    | Ruština               | Русский          |
| `nl`    | Holandčina                 | Nederlands       |
| `ar`    | Arabčina                | العربية          |
| `pl`    | Poľština                | Polski           |
| `tr`    | Turečtina               | Türkçe           |
| `hi`    | Hindčina                 | हिंदी            |
| `id`    | Indonézština            | Bahasa Indonesia |
| `vi`    | Vietnamčina            | Tiếng Việt       |
| `th`    | Thajčina                  | ไทย              |
| `sv`    | Švédčina               | Svenska          |
| `da`    | Dánčina                | Dansk            |
| `no`    | Nórčina             | Norsk            |
| `fi`    | fínčina               | Suomi            |
| `el`    | gréčtina                 | Ελληνικά         |
| `cs`    | čeština                 | Čeština          |
| `hu`    | Maďarčina             | Magyar           |
| `ro`    | Rumunčina              | Română           |
| `uk`    | Ukrajinčina             | Українська       |
| `pt-BR` | Brazílska portugalčina  | Português Brasileiro |
| `zh-HK` | Kantónčina             | 粵語             |
| `ms`    | Malajčina                 | Bahasa Melayu    |
| `sk`    | Slovenčina                | Slovenčina       |
| `bg`    | Bulharčina             | Български        |
| `hr`    | Chorvátčina              | Hrvatski         |
| `lt`    | Litovčina            | Lietuvių         |
| `lv`    | Lotyšský               | Latviešu         |
| `et`    | Estónsky              | Eesti            |
| `sl`    | Slovinčtina             | Slovenščina      |

{% hint style=&quot;success&quot; %}
**Automatická perzistencia**: Vaše jazykové preferencie sú uložené v `~/.chloros/cli_language.json` a pretrvávajú vo všetkých reláciách.
{% endhint %}

***

### `set-project-folder` – Nastavenie predvoleného priečinka projektu

Zmeňte umiestnenie predvoleného priečinka projektu (zdieľaného s GUI).

**Syntax:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Príklad:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` – Zobraziť priečinok projektu

Zobrazí aktuálne umiestnenie predvoleného priečinka projektu.

**Syntax:**

```bash
chloros-cli get-project-folder
```

**Príklad:**

```powershell
chloros-cli get-project-folder
```

**Výstup:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` – Obnoviť predvolené nastavenia

Obnoví predvolenú polohu priečinka projektu.

**Syntax:**

```bash
chloros-cli reset-project-folder
```

***

## Globálne možnosti

Tieto možnosti sa vzťahujú na všetky príkazy:

| Možnosť          | Typ    | Predvolené nastavenie       | Popis                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Cesta    | Automaticky detegovaná | Cesta k spustiteľnému súboru backendu                       |
| `--port`        | Celé číslo | 5000          | Číslo portu backendu API                          |
| `--restart`     | Príznak    | -             | Vynútiť reštart backendu (ukončí existujúce procesy) |
| `--version`     | Príznak    | -             | Zobraziť informácie o verzii a ukončiť                |
| `--help`        | Príznak    | -             | Zobraziť informácie o pomoci a ukončiť                   |

**Príklad s globálnymi možnosťami:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Príručka nastavení spracovania

### Paralelné spracovanie

Chloros+ CLI **automaticky škaluje** paralelné spracovanie tak, aby zodpovedalo schopnostiam vášho počítača:

**Ako to funguje:**

* Detekuje jadrá procesora a pamäť RAM
* Prideli pracovníkov: **2× jadrá procesora** (využíva hyperthreading)
* **Maximálne: 16 paralelných pracovníkov** (pre stabilitu)

**Úrovne systému:**

| Typ systému   | Procesor        | Pamäť RAM      | Pracovníci  | Výkon     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **High-End**  | 16+ jadier  | 32+ GB   | Až 16 | Maximálna rýchlosť   |
| **Stredná trieda** | 8–15 jadier | 16–31 GB | 8–16     | Vynikajúca rýchlosť |
| **Nízka trieda**   | 4–7 jadier  | 8–15 GB  | 4–8      | Dobrá rýchlosť      |

{% hint style=&quot;success&quot; %}
**Automatická optimalizácia**: CLI automaticky detekuje špecifikácie vášho systému a nakonfiguruje optimálne paralelné spracovanie. Nie je potrebná žiadna manuálna konfigurácia!
{% endhint %}

### Metódy debayer

CLI používa ako predvolený a odporúčaný debayer algoritmus **Vysoká kvalita (rýchlejší)**:

| Metóda                      | Kvalita | Rýchlosť | Popis                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Vysoká kvalita (rýchlejšia)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Algoritmus zohľadňujúci okraje (predvolený, odporúčaný) |

### Korekcia vinetácie

**Čo robí:** Koriguje pokles jasu na okrajoch obrazu (tmavšie rohy bežné v snímkach z fotoaparátu).

* **Predvolene zapnuté** – väčšina používateľov by mala túto funkciu nechať zapnutú.
* Na vypnutie použite `--no-vignette`.

{% hint style=&quot;success&quot; %}
**Odporúčanie**: Vždy zapnite korekciu vinietovania, aby ste zabezpečili rovnomernú jasnosť v celom ráme.
{% endhint %}

### Kalibrácia odrazivosti

Prevádza surové hodnoty senzora na štandardizované percentá odrazivosti pomocou kalibračných panelov.

* **Povolené štandardne** – nevyhnutné pre analýzu vegetácie.
* Vyžaduje kalibračné panely v obrazoch.
* Na deaktiváciu použite `--no-reflectance`.

{% hint style=&quot;info&quot; %}
**Požiadavky**: Pre presnú konverziu odrazivosti sa uistite, že kalibračné panely sú správne exponované a viditeľné vo vašich obrázkoch.
{% endhint %}

### PPK korekcie

**Čo robí:** Používa postprocesné kinematické korekcie pomocou údajov z protokolu DAQ-A-SD pre zlepšenie presnosti GPS.

* **V predvolenom nastavení je táto funkcia vypnutá.
* Na jej zapnutie použite `--ppk`.
* Vyžaduje súbory .daq v projektovej zložke z MAPIR DAQ-A-SD svetelného senzora.

### Výstupné formáty

<table><thead><tr><th width="197">Formát</th><th width="130.20001220703125">Bitová hĺbka</th><th width="116.5999755859375">Veľkosť súboru</th><th>Najvhodnejšie pre</th></tr></thead><tbody><tr><td><strong>TIFF (16-bitový)</strong> ⭐</td><td>16-bitové celé číslo</td><td>Veľké</td><td>GIS analýza, fotogrametria (odporúčané)</td></tr><tr><td><strong>TIFF (32-bitové, percentá)</strong></td><td>32-bitové plávajúce číslo</td><td>Veľmi veľké</td><td>Vedecká analýza, výskum</td></tr><tr><td><strong>PNG (8-bitový)</strong></td><td>8-bitové celé číslo</td><td>Stredné</td><td>Vizuálna kontrola, zdieľanie na webe</td></tr><tr><td><strong>JPG (8-bitové)</strong></td><td>8-bitové celé číslo</td><td>Malé</td><td>Rýchly náhľad, komprimovaný výstup</td></tr></tbody></table>***

## Automatizácia a skriptovanie

### Hromadné spracovanie v PowerShell

Automatické spracovanie viacerých priečinkov s dátovými sadami:

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

### Windows Hromadný skript

Jednoduchá slučka pre hromadné spracovanie:

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

### Python Automatizačný skript

Pokročilá automatizácia s riešením chýb:

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

1. **Vstup**: Zložka obsahujúca páry obrázkov RAW/JPG
2. **Vyhľadávanie**: CLI automaticky vyhľadáva podporované obrazové súbory
3. **Spracovanie**: Paralelný režim sa prispôsobuje jadrám vášho procesora (Chloros+)
4. **Výstup**: Vytvára podadresáre podľa modelov fotoaparátov so spracovanými obrázkami

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

### Odhadované časy spracovania

Typické časy spracovania pre 100 obrázkov (každý s rozlíšením 12 MP):

| Režim              | Čas      | Hardvér                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Paralelný režim** | 5–10 minút  | i7/Ryzen 7, 16 GB RAM, SSD (až 16 pracovníkov) |
| **Paralelný režim** | 10–15 minút | i5/Ryzen 5, 8 GB RAM, HDD (až 8 pracovníkov)   |

{% hint style=&quot;info&quot; %}
**Tip na zvýšenie výkonu**: Doba spracovania sa líši v závislosti od počtu obrázkov, rozlíšenia a špecifikácií počítača.
{% endhint %}

***

## Riešenie problémov

### CLI nenájdené

**Chyba:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Riešenia:**

1. Overte umiestnenie inštalácie:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Ak nie je v PATH, použite úplnú cestu:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Ručne pridajte do PATH:
   * Otvorte Vlastnosti systému → Premenné prostredia.
   * Upravte premennú PATH.
   * Pridajte: `C:\Program Files\Chloros\resources\cli`
   * Reštartujte terminál.

***

### Backend sa nepodarilo spustiť.

**Chyba:**

```
Backend failed to start within 30 seconds
```

**Riešenia:**

1. Skontrolujte, či backend už beží (najskôr ho zatvorte).
2. Skontrolujte, či Windows Firewall neblokuje.
3. Skúste iný port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Vynúťte reštartovanie backendu:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Problémy s licenciou/overením

**Chyba:**

```
Chloros+ license required for CLI access
```

**Riešenia:**

1. Overte, či máte aktívne predplatné Chloros+.
2. Prihláste sa pomocou svojich prihlasovacích údajov:

```powershell
chloros-cli login user@example.com 'password'
```

3. Skontrolujte stav licencie:

```powershell
chloros-cli status
```

4. Kontaktujte podporu: info@mapir.camera

***

### Nenašli sa žiadne obrázky

**Chyba:**

```
No images found in the specified folder
```

**Riešenia:**

1. Overte, či priečinok obsahuje podporované formáty (.RAW, .TIF, .JPG).
2. Skontrolujte, či je cesta k priečinku správna (pre cesty s medzerami použite úvodzovky).
3. Uistite sa, že máte oprávnenie na čítanie priečinka.
4. Skontrolujte, či sú príponami súborov správne.

***

### Spracovanie sa zastaví alebo zamrzne

**Riešenia:**

1. Skontrolujte voľné miesto na disku (uistite sa, že je dostatočné na výstup).
2. Zatvorte ostatné aplikácie, aby ste uvoľnili pamäť.
3. Znížte počet obrázkov (spracovávajte ich po častiach).

***

### Port je už používaný

**Chyba:**

```
Port 5000 is already in use
```

**Riešenie:**

Určite iný port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## Často kladené otázky

### Otázka: Potrebujem licenciu pre CLI?

**Odpoveď:** Áno! CLI vyžaduje platenú **licenciu Chloros+**.

* ❌ Štandardný (bezplatný) plán: CLI deaktivovaný
* ✅ Plány Chloros+ (platené): CLI plne povolené

Prihláste sa na: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Otázka: Môžem používať CLI na serveri bez grafického rozhrania?

**Odpoveď:** Áno! CLI beží úplne bez grafického rozhrania. Požiadavky:

* Windows Server 2016 alebo novší
* Nainštalovaný Visual C++ Redistributable
* Dostatočná pamäť RAM (minimálne 8 GB, odporúča sa 16 GB)
* Jednorazová aktivácia licencie GUI na akomkoľvek počítači

***

### Otázka: Kde sa ukladajú spracované obrázky?

**Odpoveď:** Spracované obrázky sa štandardne ukladajú do **rovnakého priečinka ako vstupné** v podpriečinkoch modelu fotoaparátu (napr. `Survey3N_RGN/`).

Na určenie iného výstupného priečinka použite možnosť `-o`:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### Otázka: Môžem spracovať viacero priečinkov naraz?

**A:** Nie priamo v jednom príkaze, ale môžete použiť skriptovanie na postupné spracovanie priečinkov. Pozrite si časť [Automatizácia a skriptovanie](CLI.md#automation--scripting).

***

### Q: Ako uložím výstup CLI do súboru protokolu?

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### Otázka: Čo sa stane, ak počas spracovania stlačím klávesy Ctrl+C?

**Odpoveď:** CLI:

1. Plynule zastaví spracovanie
2. Vypne backend
3. Ukončí sa s kódom 130

Čiastočne spracované obrázky môžu zostať vo výstupnej zložke.

***

### Otázka: Môžem automatizovať spracovanie CLI?

**Odpoveď:** Samozrejme! CLI je navrhnutý pre automatizáciu. Príklady pre PowerShell, Batch a Python nájdete v časti [Automatizácia a skriptovanie](CLI.md#automation--scripting).

***

### Otázka: Ako skontrolujem verziu CLI?

**Odpoveď:**

```powershell
chloros-cli --version
```

**Výstup:**

```
Chloros CLI 1.0.2
```

***

## Pomoc

### Pomoc v príkazovom riadku

Informácie o pomoci si môžete zobraziť priamo v CLI:

```powershell
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
* **Ceny**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Kompletné príklady

### Príklad 1: Základné spracovanie

Spracovanie s predvolenými nastaveniami (vinetácia, odrazivosť):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Príklad 2: Vysoko kvalitný vedecký výstup

32-bitové plávajúce desatinné číslo TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Príklad 3: Rýchle spracovanie náhľadu

8-bitové PNG bez kalibrácie pre rýchle prezeranie:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Príklad 4: Spracovanie s korekciou PPK

Uplatnenie korekcií PPK s odrazivosťou:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Príklad 5: Vlastné umiestnenie výstupu

Spracovanie na inom disku s konkrétnym formátom:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Príklad 6: Pracovný postup overovania

Dokončenie overovania:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Príklad 7: Viacjazyčné použitie

Zmena jazyka rozhrania:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```

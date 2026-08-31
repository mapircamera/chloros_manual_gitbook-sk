# Inštalácia Linux

Chloros sa distribuuje pre Linux ako balíky `.deb`, ktoré inštalujú CLI a backendový server. Python SDK je samostatný balík pip (zahrnutý aj v balíku `.deb` ako wheel s zodpovedajúcou verziou).

Názvy súborov balíkov obsahujú verziu a architektúru: `chloros_1.2.0_amd64.deb` pre x86_64 a `chloros_1.2.0_arm64_jp6.deb` pre zostavenia JetPack 6 pre Jetson. V nižšie uvedených príkazoch nahraďte názov súboru, ktorý ste si skutočne stiahli.

***

## Linux amd64 (x86_64)

### Systémové požiadavky

| Požiadavka | Minimálna | Odporúčaná |
| --- | --- | --- |
| **Distribúcia** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Procesor** | x86_64 (Intel/AMD) | Intel Core i7 alebo lepší |
| **Pamäť (RAM)** | 8 GB | 16 GB alebo viac |
| **Grafická karta** | Žiadna (spracovanie na CPU) | Grafická karta NVIDIA s 4 GB a viac VRAM (12 GB a viac odomkne `GPU_PARALLEL`, 7 GB a viac zabráni tomu, aby sa Texture Aware používalo v režime jedného obrázka) |
| **Úložný priestor** | 2 GB voľného miesta | SSD s 10 GB a viac voľného miesta |
| **Python** | Python 3.7+ (pre SDK) | Python 3.10+ |

> **Ubuntu 20.04 a Debian 11 nie sú podporované.** Zoznam závislostí `.deb` je
> odvodený od toho, na čo sa backend Chloros skutočne odkazuje, a to zahŕňa
> `libc6 (>= 2.34)`. Verzie Focal aj Bullseye obsahujú glibc 2.31, takže `apt`
> inštaláciu priamo odmietne, namiesto toho, aby ju neskôr počas behu programu nechal zlyhať.

### Inštalácia

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` nevyrieši závislosti. Ak nahlási chýbajúce balíky, `sudo apt-get install -f` (alebo `sudo apt --fix-broken install`) dokončí inštaláciu — ide o bežný postup, nie o chybu.
{% endhint %}

Overte inštaláciu:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Systémové požiadavky

| Požiadavka | Minimálna | Odporúčaná |
| --- | --- | --- |
| **Platforma** | NVIDIA Jetson s JetPack 6 | Jetson Orin NX 16 GB alebo AGX Orin |
| **JetPack** | JetPack 6.x | Najnovšia verzia JetPack 6 |
| **Pamäť (RAM)** | 8 GB (zdieľaná medzi GPU a CPU) | 16 GB+ zdieľanej pamäte (12 GB+ je prahová hodnota pre paralelné GPU pracovníky) |
| **Úložisko** | 2 GB voľného miesta | NVMe SSD s 10 GB+ voľného miesta |
| **Python** | Python 3.7+ (pre SDK) | Python 3.10+ |

### Inštalácia

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Rovnaká štruktúra ako v prípade amd64 `.deb`, s verziou CUDA vyladenou pre Jetson Orin / Orin NX / Orin Nano. Informácie o správaní Jetsonu v oblasti pamäte, tepelného správania a nasadenia v teréne nájdete v [príručke NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Inštalácia Python a SDK (všetky verzie Linux)

SDK je čistý klient Python pre backend, takže ten istý balík funguje na architektúrach amd64 aj arm64. Dva zdroje:**Z PyPI** — uverejnená stabilná verzia:

```bash
pip install chloros-sdk
```

**Z priloženého súboru wheel** — zaručene kompatibilný s CLI/backendom, ktorý ste práve nainštalovali (použite to, ak je váš `.deb` novší ako verzia na PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**Distribúcie podľa PEP 668** (Ubuntu 23.10+, Debian 12+) odmietajú inštalácie pip v celom systéme. Použite `pip install --user …`, virtuálne prostredie alebo `sudo pip install --break-system-packages …`. Inštalátor balíkov nikdy automaticky neinštaluje SDK do vášho systémového prostredia Python — toto rozhodnutie je na vás.
{% endhint %}

Voliteľné doplnky:

| Prídavok | Príkaz | Pridáva |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` pre živé streamovanie priebehu |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` pre prenos cez BLE (DAQ-M) |

Overte SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` nainštaluje Chloros, CLI a backend. Python a SDK komunikujú s týmto backendom cez lokálnu sieť HTTP a API (`http://127.0.0.1:5000`) a v prípade potreby ho automaticky spustí. Vždy používajte doslovnú adresu IPv4 namiesto `localhost` — `localhost` sa môže preložiť na `::1` a jedna požiadavka môže trvať približne dve sekundy.
{% endhint %}

***

## Prvé nastavenie

### 1. Prihlásenie

Prístup k adresám CLI a SDK vyžaduje platenú úroveň Chloros+ (**Copper** alebo vyššiu), čo sa vynucuje na strane servera: odhlásený volajúci dostane `401 AUTH_REQUIRED` a volajúci s bezplatným balíkom (Iron) dostane `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Prihlasovacie údaje sa ukladajú do vyrovnávacej pamäte v súbore `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Po každej inštalácii alebo aktualizácii sa musíte znovu prihlásiť.** Skript balíka `prerm` zámerne vymaže `~/.chloros/user_session.json` a uloženú licenciu pre každého používateľa v systéme, aby nová verzia vždy znovu overila platnosť licencie namiesto toho, aby sa spoliehala na neaktuálnu vyrovnávaciu pamäť.
{% endhint %}

### 2. Skontrolujte stav svojej licencie

```bash
chloros-cli status
```

`chloros-cli status` funguje na akejkoľvek úrovni (vrátane bezplatnej), takže vždy môžete zistiť, prečo je alebo nie je prístup dostupný.

### 3. Spustite diagnostiku systému

```bash
chloros-cli selftest
```

Postupne sa spustí sedem kontrol a príkaz skončí s nenulovým výstupom, ak niektorá z nich zlyhá:

| # | Kontrola | Čo overuje |
| --- | --- | --- |
| 1 | **Verzia** | CLI nahlási svoju verziu (`v1.2.0`). |
| 2 | **Port dostupný** | Port 5000 je voľný, *alebo* naň už odpovedal funkčný backend Chloros (čo sa považuje za úspešnú kontrolu). |
| 3 | **Spustenie backendu** | Spustí sa binárny súbor backendu. |
| 4 | **Test API (`/api/test`)** | Backend odpovedá `status: ok`. |
| 5 | **Informácie o systéme** | Vypíše `GPU: <name>, CUDA: <bool>, PyTorch: <version>` z `/api/system-info`. |
| 6 | **Modely odšumovača** | Nájde modely `*.pth.enc` (na Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + odšumovač**| Funkcia Texture Aware je skutočne použiteľná — vyžaduje CUDA**a** aspoň jeden súbor modelu. |

Spustenie končí modelom `N/7 checks passed`, pričom sa vypisujú všetky chyby podľa názvu.

### 4. Spracujte svoj prvý súbor údajov

```bash
chloros-cli process ~/datasets/flight001
```

***

## Súbory a adresáre

### Pre každého používateľa

Chloros uchováva svoje prihlasovacie údaje a konfiguráciu CLI v jednom medziplatformovom adresári, **`~/.chloros/`** (na Windows, `%USERPROFILE%\.chloros\`). Dve vyrovnávacie pamäte špecifické pre Linux sa namiesto toho riadia konvenciami XDG — tieto rešpektujú nastavenia `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`, ak sú nastavené.

| Cesta | Účel |
| --- | --- |
| `~/.chloros/user_session.json` | Cache prihlasovacej relácie zapísaná súborom `chloros-cli login` (vyprázdňuje sa pri každej inštalácii/aktualizácii balíka) |
| `~/.chloros/working_directory.txt` | Prepis predvoleného priečinka projektu (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Nastavenie jazyka CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Nastavenie jazyka zdieľané s grafickým rozhraním Windows — hodnota `language` má v tomto prípade prednosť pred hodnotou `cli_language.json` |
| `~/.chloros/update_cache.json` | Hodinová vyrovnávacia pamäť pre kontrolu aktualizácií pri spustení Linux/Jetson |
| `~/.chloros/backend.log` | Protokol backendu pri spustení backendu prostredníctvom CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Uložené kalibračné balíky LATTICE pre jednotlivé kamery, identifikované sériovým číslom a hash kódom balíka |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Voliteľné užívateľské prepisovania profilov korekcie DAQ |
| `~/.config/chloros/system_config.json` | Uložený hardvérový profil z Dynamic Compute Adaptation – jeho odstránením vynútite nové rozpoznanie hardvéru |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Protokoly backendového servera, jeden súbor na každé spustenie |
| `~/Chloros Projects/` | Predvolaná zložka projektu, ak nie je nastavené žiadne prepisovanie |

### Systémové

| Cesta | Účel |
| --- | --- |
| `/usr/bin/chloros-cli` | Skript wrapperu — nastaví `LD_LIBRARY_PATH` pre priložené natívne knižnice a následne spustí skutočný binárny súbor |
| `/usr/bin/chloros-backend` | Obalový skript — to isté, plus `CHLOROS_PRODUCTION=1`, aby sa autorizačná brána backendu nikdy nemohla ticho deaktivovať |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Zkompilované binárne súbory |
| `/usr/lib/chloros/arena_runtime/` | Runtime prostredie Arena SDK požadované kamerami LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Šifrované modely odšumovača používané debayerom Texture Aware |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK balík zodpovedajúci presne tejto verzii |
| `/usr/lib/chloros/exiftool` | Priložený exiftool (symbolický odkaz na `/usr/local/bin/exiftool` iba v prípade, že neexistuje systémový exiftool) |
| `/etc/chloros/update.conf` | Aktualizáciakonfigurácie kanála načítanej programom `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Nastaví `net.ipv4.ip_unprivileged_port_start = 319` tak, aby backend mohol viazať porty PTP bez oprávnení root |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Nasmeruje dynamický zavádzač na `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Udeľuje prihlásenému používateľovi prístup k sériovému mostu DAQ-U USB (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Zapnutie stále aktívnej služby na pozadí (nainštalovaná, **nie je povolená**) |
| `/usr/share/applications/chloros-cli.desktop` | Položka v ponuke aplikácií „Chloros CLI“, ktorá otvorí terminál |

## Umiestnenie spustiteľného súboru backendu

CLI a SDK automaticky detekujú backend:

| Komponent | Cesta |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Cestu k backendu môžete prepisovať pomocou príznaku `--backend-exe` CLI alebo parametra konštruktora `backend_exe` SDK, a port pomocou `--port` (predvolené nastavenie je `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` smeruje na **`lattice`**,**`project`**a**`daq pool-*`** na vzdialenom backende. Základné príkazy (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ho zámerne ignorujú a vždy sa zameriavajú na `http://127.0.0.1:<port>`.
{% endhint %}

***

## Kamery LATTICE a svetelné senzory DAQ na Linux

Všetky skupiny príkazov pre živý hardvér fungujú na Linux (amd64 a Jetson):

* **`chloros-cli lattice`** — vyhľadávanie, pripájanie, konfigurácia a snímanie z kamier LATTICE a synchronizovaných polí. `.deb` obsahuje runtime prostredie Arena SDK, ktoré tieto zariadenia vyžadujú, a zaregistruje ho v dynamickom zavádzači.
* **`chloros-cli daq pool-*`** — pripojenie svetelných senzorov DAQ-U/M/E prostredníctvom backendového poola, streamovanie kalibrovaných spektier a zaznamenávanie súborov `.daq`. Kompilovaný balík CLI obsahuje iba rodinu `pool-*`: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — spustite uložený projekt (jeho kamery, senzory a nastavenia spracovania) bez grafického rozhrania.
* **`chloros-cli time-sync`** — skontroluje hlavný PTP server, na ktorom beží backend Chloros pre kamery LATTICE a senzory DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` je potrebný pre `pool-latest`, `pool-stream`, `pool-record` a `pool-set-cap`; `pool-list` zobrazuje ID, ktoré sa momentálne nachádzajú v poole.

{% hint style="info" %}
**Pre prvé pripojenie DAQ-E na zariadení s viacerými sieťovými kartami uprednostnite `--eth-host`.** Funkcia automatického vyhľadávania prehľadáva mDNS a môže prehliadnuť rozhranie senzora kvôli prázdnej ARP cache, takže prvé pripojenie `pool-connect --eth` po spustení systému môže zlyhať, aj keď je senzor úplne v poriadku. Zadanie IP adresy alebo hostiteľa senzora úplne preskočí proces vyhľadávania.
{% endhint %}

**Oprávnenia sériového rozhrania DAQ-U** sú spravované nainštalovaným pravidlom udev (`uaccess` + skupina `dialout`). Ak senzor, ktorý bol už pripojený, zostáva nedostupný, načítajte pravidlá znovu alebo ho znovu pripojte:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Úplný zoznam príkazov nájdete v [referencii CLI](../CLI.md).

### Neustále aktívny PTP pre hostiteľa bez grafického rozhrania

Pri prvej inštalácii sa vygeneruje jednotka systemd `chloros-backend.service`, ktorá však **nie je aktivovaná**. Na bezmonitorovom zariadení Jetson alebo serveri, kde by mala synchronizácia času PTP pre senzory DAQ-E a kamery LATTICE bežať nepretržite, ju aktivujte:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Bez nej beží PTP len počas behu backendu Chloros — teda počas aktívnej relácie CLI/SDK.

Zariadenie viaže backend na `127.0.0.1:5000` (nastavenia prostredia `CHLOROS_HOST` / `CHLOROS_PORT` vnútri zariadenia; prepisujte pomocou `sudo systemctl edit chloros-backend.service`) a v prípade zlyhania ho po 5 sekundách reštartuje.

**Ako PTP získava svoje porty.** PTP používa porty UDP 319/320, ktoré sú oba pod bežnou hranicou 1024 privilegovaných portov. Balík `postinst` zapíše `/etc/sysctl.d/60-chloros-ptp.conf` pomocou `net.ipv4.ip_unprivileged_port_start = 319`, čo umožňuje backendu viazať sa na ne pri behu pod vaším používateľským účtom. Ako dodatočné bezpečnostné opatrenie sa na binárny súbor backendu aplikuje aj `setcap cap_net_bind_service,cap_net_raw=+ep` – preto je `libcap2-bin` deklarovanou závislosťou balíka.***

## Príklady skriptov v Bash

{% hint style="info" %}
**Kódy ukončenia prispôsobené skriptovaniu.**`chloros-cli process` ukončí `0` v prípade úspechu a**v prípade zlyhania vráti hodnotu odlišnú od nuly — vrátane behu, ktorý požiadal o obrazové produkty, ale žiadne nezapísal** (vypíše `Processing finished but wrote no image products.` a uvedie názov projektovej zložky a bežné príčiny). Úspešné spustenia hlásia, koľko obrazových produktov bolo zapísaných (`Image products written: N`). Kódy ukončenia: `0` úspech, `1` zlyhanie, `2` chyba argumentu, `130` prerušené.
{% endhint %}

### Spracovanie viacerých dátových súborov

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Spracovanie s vlastnými nastaveniami

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Platných hodnôt `--format` je presne štyri a obsahujú medzery – vždy ich uvádzajte v úvodzovkách:

| Hodnota `--format` | Výstupná zložka |
| --- | --- |
| `TIFF (16-bit)` *(predvolené)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` prijíma `standard` (predvolené) alebo `texture-aware` (Chloros+).

### Automatizované spracovanie pomocou Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Príklad Python SDK

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

### CLI sa po inštalácii nenašiel

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Oprávnenie zamietnuté

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### Chyba „setcap failed“ počas inštalácie

`.deb` aplikuje `cap_net_bind_service` na `/usr/lib/chloros/chloros-backend`, aby mohol viazať PTP porty 319/320 bez oprávnení root. Ak chýbal `libcap2-bin` v čase inštalácie, volanie sa preskočí. Nainštalujte ho a balík nainštalujte znovu:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP sa nespustí / nemôže priradiť port 319

Overte, či bola znížená hranica pre porty bez oprávnení, a ak nie, znovu ju uplatnite pre aktuálne spustenie:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Potom skontrolujte grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### „Nenašli sa ovládače kamery LATTICE“

Runtime Arena SDK sa nenašiel. Overte, či je prítomná a aktualizovaná konfigurácia zavádzača, ktorú balík zapisuje:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
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

Protokoly backendu týkajúce sa zlyhania spustenia sa nachádzajú v súbore `~/.cache/chloros/logs/`.

### CUDA nebola zistená

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` hlási to isté v jednom riadku: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Chýbajúce zdieľané knižnice

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Pomalé spúšťanie na systémoch s SD kartou

Zkompilované binárne súbory sa pri každom spustení samy rozbalia do dočasného adresára. Ak existuje súbor `/mnt/ssd/tmp`, program Chloros ho automaticky použije; v opačnom prípade nastavte súbor `TMPDIR` na rýchly súborový systém:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Aktualizácia súboru Chloros na systéme Linux

Príkaz `update` je k dispozícii iba na systéme Linux/Jetson. Overuje verziu uverejnenú v kanáli aktualizácií nakonfigurovanom v `/etc/chloros/update.conf` a ponúka stiahnutie a inštaláciu zodpovedajúceho `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Na Linux/Jetson vykonáva CLI pri každom spustení aj neblokujúcu kontrolu aktualizácií (výsledok je v `~/.chloros/update_cache.json` uložený v cache po dobu jednej hodiny) a v prípade, že existuje novšia verzia, vypíše `Update available: vX.Y.Z`. Vaše nastavenia a projekty zostanú po aktualizácii zachované; po jej dokončení sa budete musieť znovu prihlásiť.

## Odinštalovanie

```bash
sudo apt remove chloros
```

Odstránenie zastaví proces `chloros-backend.service`, obnoví predvolenú minimálnu hodnotu pre neprivilegované porty (1024), odstráni symbolický odkaz na nástroj exiftool a konfiguráciu zavádzača Arena a vymaže uložené prihlasovacie údaje. Vaše projekty a dátové súbory `~/.chloros/` zostanú nedotknuté.

***

## Ďalšie kroky

* [Príručka pre NVIDIA Jetson](nvidia-jetson-guide.md) — optimalizácia a nasadenie špecifické pre Jetson
* [CLI : Príkazový riadok](../CLI.md) — sprievodca CLI
* [API : Python SDK](../api-python-sdk.md) — príručka k SDK
* [Referencia k CLI](../reference/cli-reference.md) a [Referencia k SDK](../reference/sdk-reference.md) — úplný zoznam príkazov/API pre verziu 1.2.0
* [Dynamická adaptácia výpočtov](../processing-architecture/dynamic-compute-adaptation.md) — ako sa Chloros prispôsobuje vášmu hardvéru

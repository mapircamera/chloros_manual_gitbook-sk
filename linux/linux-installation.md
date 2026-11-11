# Inštalácia v systéme Linux

Chloros sa pre systém Linux distribuuje vo forme balíkov `.deb`, ktoré inštalujú CLI, backendový server a Python SDK (pre systém `python3`, plus wheel zodpovedajúci danej verzii pre virtualenvs).

Názvy súborov balíkov obsahujú verziu a architektúru: `chloros_1.3.0_amd64.deb` pre x86_64, a `chloros_1.3.0_arm64_jp6.deb` pre arm64 (NVIDIA Jetson, JetPack 6). V nižšie uvedených príkazoch nahraďte názov súboru, ktorý ste skutočne stiahli.

{% hint style="info" %}
**Novinka vo verzii 1.3.0: programy sa dodávajú v rozbalenom stave.** Backend a CLI sa inštalujú ako priečinky, `/usr/lib/chloros/backend/` a `/usr/lib/chloros/cli/`, namiesto jednotlivých samorozbaľovacích súborov. Už sa pri každom spustení nerozbaľujú do dočasnej zložky, takže sa spúšťajú oveľa rýchlejšie (na x86_64 sa backend pri každom spustení rozbaľoval približne 100 sekúnd) a už nepotrebujú niekoľko GB voľného miesta v `/tmp`. Verzia pre amd64 `.deb` má tiež približne polovičnú veľkosť oproti predchádzajúcej verzii.
{% endhint %}

***

## Linux amd64 (x86_64)

### Systémové požiadavky

| Požiadavka | Minimálna | Odporúčaná |
| --- | --- | --- |
| **Distribúcia** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Procesor** | x86_64 (Intel/AMD) | Intel Core i7 alebo lepší |
| **Pamäť (RAM)** | 8 GB | 16 GB alebo viac |
| **Grafická karta** | Žiadna (spracovanie na procesore) | Grafická karta NVIDIA s 4 GB+ VRAM (12 GB+ odomkne `GPU_PARALLEL`, 7 GB+ udrží Texture Aware mimo cesty s jedným obrázkom) |
| **Úložisko** | 2 GB voľného miesta | SSD s 10 GB+ voľného miesta |
| **Python** | Python 3.7+ (pre SDK) | Python 3.10+ |

> **Ubuntu 20.04 a Debian 11 nie sú podporované.** Zoznam závislostí `.deb` je
> odvodený od toho, na čo sa backend Chloros skutočne odkazuje, a to zahŕňa
> `libc6` novší ako glibc 2.31, s ktorou sa dodávajú verzie focal a bullseye, takže `apt` inštaláciu
> priamo odmietne, namiesto toho, aby ju neskôr počas behu programu nechal zlyhať.

### Inštalácia

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Použite `apt install ./…`, nie `dpkg -i`.** `apt` vyrieši závislosti balíka a nainštaluje aj to, čo odporúča: požiadavky Python SDK, `python3-numpy` a `python3-requests`. `dpkg -i` neinštaluje ani jedno z nich. CLI a backend sú samostatné a fungujú aj bez nich, ale `import chloros_sdk` ich potrebuje; výstup inštalácie uvádza všetky chýbajúce súčasti. Zachovajte `./`, aby balík `apt` nainštaloval tento súbor namiesto toho, aby hľadal balík s týmto názvom. Balík tiež deklaruje súbor `libgl1`, ktorý načíta súčasťou balíka OpenCV a v minimálnom obraze servera môže chýbať.
{% endhint %}

Overte inštaláciu:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Systémové požiadavky

| Požiadavka | Minimálna | Odporúčaná |
| --- | --- | --- |
| **Platforma** | NVIDIA Jetson s JetPack 6 | Jetson Orin NX 16 GB alebo AGX Orin |
| **JetPack** | JetPack 6.x | Najnovšia verzia JetPack 6 |
| **Pamäť (RAM)** | 8 GB (zdieľaná medzi GPU a CPU) | 16 GB a viac zdieľanej pamäte (12 GB a viac je prahová hodnota pre paralelné spracovanie) |
| **Úložisko** | 2 GB voľného miesta | NVMe SSD s 10 GB+ voľného miesta |
| **Python** | Python 3.7+ (pre SDK) | Python 3.10+ |

### Inštalácia

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Rovnaká štruktúra ako pri amd64 `.deb` (vrátane Python SDK, prijímacích vyrovnávacích pamätí a backendových služieb), pre Jetson Orin / Orin NX / Orin Nano na **JetPack 6** (L4T R36). Jeho PyTorch je CUDA verzia pre Orin z JetPack 6, takže spracovanie na GPU, vrátane debayeringu s podporou textúr, prebieha na GPU zariadenia Jetson. Balík neobsahuje samotnú CUDA: využíva knižnice CUDA a cuDNN, ktoré inštaluje JetPack 6 (`sudo apt install nvidia-jetpack` na obraze, kde chýbajú). Ak nie je k dispozícii žiadne GPU s podporou CUDA, spracovanie prebieha na CPU a `--debayer texture-aware` používa štandardný debayer (backend zaznamená jeden riadok `[DEBAYER] … needs a CUDA GPU …`, v ktorom vysvetľuje dôvod). Informácie o správaní Jetsonu v oblasti pamäte, tepelného správania a nasadenia v teréne nájdete v [Príručke NVIDIA Jetson](nvidia-jetson-guide.md).***

## Inštalácia Python SDK (všetky verzie Linuxu)**`.deb` nainštaluje SDK za vás.** Oba balíky (amd64 a Jetson) ho nainštalujú pre systém `python3`, takže `import chloros_sdk` funguje ihneď po inštalácii, bez potreby pip a bez pripojenia k sieti. Jeho balíky (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` a projektové načítače) sa umiestňujú do `/usr/lib/python3/dist-packages`, ktorý patrí k balíku: aktualizácia ich nahradí a odstránenie balíka ich odstráni. Verzia vždy zodpovedá nainštalovanému rozhraniu CLI a backendu.

Vlastné požiadavky SDK, `python3-numpy` a `python3-requests`, sú **odporúčané, nie povinné** balíkom: `sudo apt install ./chloros_<version>_<arch>.deb` ich nainštaluje, `dpkg -i` nie. Výstup inštalácie uvádza verziu SDK, ktorú `python3` importuje, akékoľvek chýbajúce požiadavky a akýkoľvek súbor `chloros_sdk` nachádzajúci sa inde v ceste balíka `python3`, ktorý prekrýva kópiu balíka (napríklad ten, ktorý bol nainštalovaný skôr pomocou `pip install --user`).**Vo virtuálnom prostredí**, ktoré nevidí systémové balíky, nainštalujte do neho priložený súbor wheel (úplne offline):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**Z PyPI** — iba pre počítač, na ktorom nie je nainštalovaný balík Chloros a ktorého SDK riadi backend na inom počítači (pozri [Režim vzdialeného backendu](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Voliteľné doplnky pre inštaláciu pomocou pip (v prostredí virtualenv alebo z PyPI):

| Doplnok | Príkaz | Pridá |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` pre živé streamovanie priebehu |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` pre prenos cez BLE (DAQ-M) |

Overte SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Pythonové SDK komunikuje s backendom, ktorý `.deb` inštaluje prostredníctvom lokálneho HTTP API (`http://127.0.0.1:5000`). `ChlorosLocal` a pomocné programy na pripojenie ho v prípade potreby automaticky spustia; uložený projekt `connect_all()` to nikdy nerobí, preto na počítači bez grafického rozhrania spúšťajte backend ako [službu](#always-on-ptp-for-headless-hosts). Vždy používajte doslovnú adresu IPv4 namiesto `localhost` — `localhost` sa môže preložiť na `::1` a spomaliť spracovanie o približne dve sekundy na žiadosť.
{% endhint %}

***

## Prvé nastavenie

### 1. Prihlásenie

Prístup k CLI a SDK vyžaduje platenú úroveň služby Chloros+ (**Copper** alebo vyššiu), čo sa vynucuje na strane servera: odhlásený volajúci dostane adresu `401 AUTH_REQUIRED` a volajúci s bezplatnou úrovňou (Iron) dostane adresu `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Prihlasovacie údaje sú uložené v cache v `~/.chloros/user_session.json`.

{% hint style="info" %}
**Pri aktualizácii sa vaše prihlasovacie údaje zachovajú; ich odstránenie ich vymaže.** Inštalácia novšej verzie `.deb` na existujúcu inštaláciu (vrátane inštalácie cez `chloros-cli update`) zachová `~/.chloros/user_session.json` a uloženú licenciu v cache, takže úlohy CLI/SDK bez grafického rozhrania budú naďalej fungovať. Odstránenie balíka (`sudo apt remove chloros`) vymaže uložené prihlasovacie údaje a vyrovnávaciu pamäť licencie pre každého používateľa na počítači, takže nová inštalácia si vyžaduje opätovnú inštaláciu verzie `chloros-cli login`.
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

Postupne sa vykoná sedem kontrol a príkaz skončí s nenulovým výstupom, ak niektorá z nich zlyhá:

| # | Kontrola | Čo overuje |
| --- | --- | --- |
| 1 | **Verzia** | CLI nahlási svoju verziu (`v1.3.0`). |
| 2 | **Port je dostupný** | Port 5000 je voľný, *alebo* naň už odpovedal funkčný backend Chloros (čo sa považuje za úspešné). |
| 3 | **Spustenie backendu** | Spustí sa binárny súbor backendu. |
| 4 | **Test API (`/api/test`)** | Backend odpovie `status: ok`. |
| 5 | **Informácie o systéme** | Vypíše `GPU: <name>, CUDA: <bool>, PyTorch: <version>` z `/api/system-info`. |
| 6 | **Modely odšumovača** | Nájde modely `*.pth.enc` (v Linuxe: `/usr/lib/chloros/models`). |
| 7 | **CUDA + odšumovač**| Funkcia Texture Aware je skutočne použiteľná — vyžaduje CUDA**a** aspoň jeden súbor modelu. |

Spustenie končí súborom `N/7 checks passed`, v ktorom sú uvedené všetky chyby podľa názvu.

### 4. Spracujte svoj prvý súbor údajov

```bash
chloros-cli process ~/datasets/flight001
```

***

## Súbory a adresáre

### Pre každého používateľa si

Chloros uchováva svoje prihlasovacie údaje a konfiguráciu CLI v jedinom adresári pre všetky platformy, **`~/.chloros/`** (v systéme Windows, `%USERPROFILE%\.chloros\`). Dve vyrovnávacie pamäte špecifické pre Linux naopak dodržiavajú konvencie XDG — ak sú nastavené, používajú adresáre `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`.

| Cesta | Účel |
| --- | --- |
| `~/.chloros/user_session.json` | Cache prihlasovacej relácie zapísaná súborom `chloros-cli login` (zachováva sa aj pri aktualizáciách balíka; vymaže sa pri odstránení balíka) |
| `~/.chloros/working_directory.txt` | Prepis predvoleného priečinka projektu (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Nastavenie jazyka rozhrania CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Nastavenie jazyka zdieľané s grafickým rozhraním Windows — hodnota `language` má v tomto prípade prednosť pred hodnotou `cli_language.json` |
| `~/.chloros/update_cache.json` | Jednohodinová vyrovnávacia pamäť pre kontrolu aktualizácií pri spustení systému Linux/Jetson |
| `~/.chloros/backend.log` | Protokol backendu pri spustení backendu prostredníctvom CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Uložené kalibračné balíky LATTICE pre jednotlivé kamery, indexované podľa sériového čísla a hash kódu balíka |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Voliteľné používateľské prepisovania profilov korekcie DAQ |
| `~/.config/chloros/system_config.json` | Uložený hardvérový profil z Dynamic Compute Adaptation — jeho vymazaním vynútite nové rozpoznanie hardvéru |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Protokoly backendového servera, jeden súbor na každé spustenie |
| `~/Documents/MAPIR/Chloros Projects/` | Predvolaná zložka projektu, ak nie je nastavené žiadne prepisovanie |

### Systémové

| Cesta | Účel |
| --- | --- |
| `/usr/bin/chloros-cli` | Skript obálky — nastaví `LD_LIBRARY_PATH` pre priložené natívne knižnice (a `TMPDIR` na `/mnt/ssd/tmp`, ak táto zložka existuje), potom spustí skutočný binárny súbor |
| `/usr/bin/chloros-backend` | Skript obálky — to isté pre backend, ktorý označí ako nainštalovanú produkčnú verziu |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | Zkompilované rozhranie CLI a backend, inštalované v rozbalenej podobe (novinka vo verzii 1.3.0); programy sú `cli/chloros-cli` a `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Dvojriadkové spúšťacie skripty, ktoré spúšťajú vyššie uvedené skutočné programy — sú uložené v týchto cestách, pretože SDK, CLI a staršie skripty ich hľadajú práve tu |
| `/usr/lib/chloros/arena_runtime/` | Runtime prostredie SDKu Arena, ktoré vyžadujú kamery LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Šifrované modely odšumovača používané nástrojom Texture Aware debayer |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (a `lattice_sdk`, `lattice_cam`, `arena_api`, načítavače projektov) | Python SDK, nainštalovaný pre systém `python3` a patriaci k balíku |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK wheel zodpovedajúci presne tejto verzii, pre virtualenv |
| `/usr/lib/chloros/exiftool` | Súčasťou balíka je spúšťač ExifTool (s symbolickým odkazom na `/usr/local/bin/exiftool` iba v prípade, že neexistuje systémový exiftool). Od verzie 1.3.0 sa skript ExifTool dodáva s vlastnou knižnicou Perl v balíku `/usr/lib/chloros/exiftool-dist/`, takže beží bez systémového balíka `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Konfigurácia kanála aktualizácií načítaná programom `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Nastaví `net.ipv4.ip_unprivileged_port_start = 319` tak, aby backend mohol viazať porty PTP bez oprávnení root |
| `/etc/sysctl.d/60-chloros-gige.conf` | Zväčší vyrovnávacie pamäte prijímania socketov jadra (`net.core.rmem_max`, `net.core.rmem_default`) na 32 MB pre prúdy z kamery LATTICE a uplatňuje ich pri inštalácii (ak je už nastavená vyššia hodnota, tá zostane zachovaná). Pri predvolenom nastavení jadra sú pakety z kamery v jadre zahodené a pole stráca snímky a snímkovú frekvenciu; `chloros-cli lattice network` hlási stav vyrovnávacích pamätí, keď sú nízke. Pozri [Hostitelia Linuxu — vyrovnávacie pamäte pre príjem cez socket](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Nasmeruje dynamický zavádzač na `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Udeľuje prihlásenému používateľovi prístup k sériovému mostu DAQ-U USB (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Služba „Opt-in always-on backend“, ktorá spúšťa backend ako používateľ, ktorého určíte (nainštalovaná, **nie je povolená**) — túto službu je potrebné povoliť, pozri [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Tá istá služba, ktorá spúšťa backend ako **root**(nainštalovaná,**nie je povolená**), ponechaná tak, aby stroj, na ktorom je povolená, naďalej bežal |
| `/var/log/chloros/` | Podrobný protokol služby root, čitateľný pre skupinu `adm` (odstránené pri úplnom odstránení balíka) |
| `/usr/share/applications/chloros-cli.desktop` | Položka v ponuke aplikácií „Chloros CLI“, ktorá otvorí terminál |

**Firewall (novinka vo verzii 1.3.0).** Ak je v čase inštalácie aktívna služba `ufw` alebo `firewalld`, balík otvorí UDP porty `5002` (dáta DAQ-E), `5353` (vyhľadávanie mDNS), `319` a `320` (PTP) a pri odstránení balíka tieto pravidlá zruší. Ak je firewall nainštalovaný, ale nie je aktívny, nič nepridáva a vypíše porty, ktoré je potrebné povoliť, ak ho neskôr aktivujete. Neotvára žiadny port TCP: aktualizácia firmvéru DAQ-E, pri ktorej sa senzor pripája späť k tomuto počítaču cez TCP na zber obrazu, zlyhá s chybou `… device never connected back on TCP <port> …`, ak aktívny firewall toto pripojenie blokuje.

## Umiestnenie spustiteľného súboru backendu

CLI a SDK automaticky detekujú backend:

| Komponent | Cesta |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (spúšťač pre `/usr/lib/chloros/backend/chloros-backend`) |

Cestu k backendu môžete prepisovať pomocou príznaku CLIu `--backend-exe` alebo parametra konštruktora SDKu `backend_exe`, a port pomocou `--port` (predvolené nastavenie je `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` smeruje rodiny príkazov **`lattice`**,**`project`**a**`daq pool-*`** na vzdialenom backende. Základné príkazy (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ho zámerne ignorujú a vždy sa zameriavajú na `http://127.0.0.1:<port>`.
{% endhint %}

***

## Kamery LATTICE a svetelné senzory DAQ v systéme Linux

Všetky skupiny príkazov live-hardware fungujú v systéme Linux (amd64 a Jetson):

* **`chloros-cli lattice`** — vyhľadávanie, pripájanie, konfigurácia a snímanie z kamier LATTICE a synchronizovaných polí. `.deb` obsahuje runtime prostredie Arena SDK, ktoré tieto zariadenia vyžadujú, a zaregistruje ho v dynamickom zavádzači.
* **`chloros-cli daq pool-*`** — pripojenie svetelných senzorov DAQ-U/M/E prostredníctvom backendového poola, streamovanie kalibrovaných spektier, nahrávanie súborov `.daq` a výmena profilov krytov, a v prípade DAQ-E zmena jeho sieťových nastavení, jeho reštartovanie a aktualizácia firmvéru. `chloros-cli daq discover` vyhľadáva senzory, ktoré je možné pripojiť, a `chloros-cli daq --help` vypisuje podpríkazy `pool-*`.
* **`chloros-cli project`** — spustí uložený projekt (jeho kamery, senzory a nastavenia spracovania) bez grafického rozhrania.
* **`chloros-cli time-sync`** — skontrolujte PTP grandmaster, na ktorom beží backend Chloros pre kamery LATTICE a senzory DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` je nevyhnutný pre `pool-latest`, `pool-stream`, `pool-record` a `pool-set-cap`; `pool-list` zobrazuje ID, ktoré sa momentálne nachádzajú v poole.

{% hint style="info" %}
**`--eth-host` preskočí vyhľadávanie.** `pool-connect --eth` nájde zariadenie DAQ-E pomocou mDNS s náhradným riešením ARP, ktoré funguje aj z prázdnej vyrovnávacej pamäte ARP; odovzdaním IP adresy alebo názvu hostiteľa senzora pomocou `--eth-host` sa k nemu priamo pripojí.
{% endhint %}

**Sériové oprávnenia pre DAQ-U** sú spravované nainštalovaným pravidlom udev (`uaccess` + skupina `dialout`). Ak senzor, ktorý bol už pripojený, zostáva nedostupný, načítajte pravidlá znovu alebo ho znovu pripojte:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Úplný zoznam príkazov nájdete v [referencii CLI](../CLI.md).

### Neustále aktívny PTP pre hostiteľa bez grafického rozhrania

Balík inštaluje dve služby systemd pre backend, z ktorých ani jedna nie je povolená. Na zariadení Jetson bez grafického rozhrania alebo na serveri, kde by mala synchronizácia času PTP pre senzory DAQ-E a kamery LATTICE bežať nepretržite, alebo kde skripty SDK pripájajú uložené projekty (`connect_all()` nikdy nespustí backend), aktivujte službu **ako svoj používateľ**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` spúšťa backend ako tento používateľ: záznamy, ktoré zapisuje, patria tomuto používateľovi, jeho podrobný protokol sa nachádza v súbore `~/.cache/chloros/logs` a používa jeho `chloros-cli login` (`~/.chloros`), preto sa najprv raz prihláste ako tento používateľ. Inštalácia balíka vypíše tento príkaz s vyplneným vaším používateľským menom.

Bez služby beží backend (a s ním aj PTP grandmaster) len dovtedy, kým ho niečo nespustí — príkaz `chloros-cli` alebo skript SDK.

Modul viaže backend na nastavenia prostredia `127.0.0.1:5000` (nastavenia prostredia `CHLOROS_HOST` / `CHLOROS_PORT` vnútri jednotky; prepisujte pomocou `sudo systemctl edit chloros-backend@.service`) a v prípade zlyhania ho po 5 sekundách reštartuje.

**Služba root.**`chloros-backend.service` spúšťa backend ako**root**. Stále je nainštalovaná, aby stroj, na ktorom je povolená, naďalej bežal, ale všetko, čo backend s oprávneniami root zapíše, patrí používateľovi root, takže používateľ nemôže zmazať svoje vlastné zachytené údaje; jeho protokol sa nachádza v súbore `/var/log/chloros`, ktorý je čitateľný pre skupinu `adm`. Pre prepnutie na vášho používateľa:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Backend služby je zdieľaný a klient ho nikdy nezastaví.** Skripty `chloros-cli` a SDK ho používajú tak, ako ho nájdu: `process` na ňom beží a načíta svoj vlastný projekt namiesto toho posledného. Tam, kde by inak CLI reštartovalo backend (`--restart` alebo `process`, ktorého import nenašiel nič), označí jednotku, vypíše `sudo systemctl restart <unit>` a ukončí `1`, pretože backend zastavený týmto spôsobom by zostal neaktívny. Reštartujte ho sami pomocou `sudo systemctl restart chloros-backend@$USER.service`. Backend služby sa nikdy sám nezastaví v stave nečinnosti a `chloros-cli backend stop` to odmietne, pričom uvedie názov príkazu `systemctl stop`.**Ako PTP získava svoje porty.** PTP používa porty UDP 319/320, ktoré sú oba pod bežnou hranicou 1024 pre privilegované porty. `postinst` balíka zapisuje `/etc/sysctl.d/60-chloros-ptp.conf` s `net.ipv4.ip_unprivileged_port_start = 319`, čo umožňuje backendu viazať sa na ne pri behu pod vaším používateľom. Od verzie 1.3.0 balík nenastavuje na backende žiadne oprávnenia súborov (`setcap`) a `libcap2-bin` už nie je závislosťou: na rozbalenom backende by oprávnenie súboru zabránilo jeho spusteniu, takže ju nepridávajte sami.***

## Príklady skriptov v Bash

{% hint style="info" %}
**Kódy ukončenia vhodné pre skriptovanie.**`chloros-cli process` ukončí `0` v prípade úspechu a**vydá nenulovú hodnotu v prípade zlyhania — vrátane behu, ktorý požiadal o obrazové produkty, ale žiadne nezapísal** (vypíše kód `Processing finished but wrote no image products.` a uvedie názov projektovej zložky a bežné príčiny). Úspešné spustenia hlásia, koľko obrazových výstupov bolo zapísaných (`Image products written: N`, plus `(plus M clamp masks)`, ak boli zapísané masky). Kódy ukončenia: `0` úspech, `1` zlyhanie, `2` chyba argumentu, `130` prerušenie.
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Platných hodnôt `--format` je presne štyri a obsahujú medzery — vždy ich uvádzajte v úvodzovkách:

| Hodnota `--format` | Výstupná zložka |
| --- | --- |
| hodnota `TIFF (16-bit)` *(predvolená)* | `tiff16` |
| hodnota `TIFF (8-bit)` | hodnota `tiff8` |
| hodnota `TIFF (32-bit, Percent)` | hodnota `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` prijíma `standard` (predvolené) alebo `texture-aware` (Chloros+).

### Automatizované spracovanie pomocou Cronu

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Príklad PythonSDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Riešenie problémov

### Po inštalácii sa nenašlo rozhranie CLI

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Prístup zamietnutý

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Backend sa nespustí po pridaní oprávnení pre súbory

Nespúšťajte `setcap` na backende. Od verzie 1.3.0 balík nenastavuje žiadne oprávnenia súborov a porty PTP 319/320 pochádzajú z nastavenia `sysctl` opísaného v časti [Ako PTP získava svoje porty](#always-on-ptp-for-headless-hosts). Funkcia súboru nastaví rozbalený backend do režimu bezpečného spustenia, v ktorom nedokáže nájsť svoje vlastné knižnice, a preto sa nespustí. Ak ste takúto funkciu pridali, odstráňte ju:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP sa nespustí / nemôže sa pripojiť na port 319

Overte, či bola znížená minimálna hodnota pre neprivilegované porty, a ak nie, znovu ju nastavte pre aktuálne spustenie:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Potom skontrolujte grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### Príkazy `lattice` zlyhávajú s chybou „LATTICE camera drivers not found“

Backend nedokázal načítať runtime kamery LATTICE, s ktorým je nainštalovaný. Každý príkaz `lattice` beží na backende; samotné CLI neobsahuje žiadne ovládače kamery. Preinštalujte Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): `.deb` nainštaluje runtime vedľa backendu v `/usr/lib/chloros/arena_runtime` a zaregistruje ho v dynamickom zavádzači. Ak chcete skontrolovať túto registráciu:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
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

Protokoly backendu týkajúce sa neúspešného spustenia sa nachádzajú v súbore `~/.cache/chloros/logs/`. Ak port 5000 patrí službe backendu (`systemctl status chloros-backend@$USER.service`), radšej službu reštartujte, ako aby ste jej proces ukončili.

### CUDA nebola zistená

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` hlási to isté v jednom riadku: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Chýbajúce zdieľané knižnice

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Pomalé spúšťanie na systémoch s SD kartou

Od verzie 1.3.0 sa backend a CLI inštalujú v rozbalenom stave, takže sa už pri každom spustení nerozbaľujú do dočasného adresára. Dočasné súbory sa automaticky ukladajú do adresára `/mnt/ssd/tmp`, ak tento adresár existuje; v opačnom prípade ich môžete nasmerovať do rýchleho súborového systému:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Aktualizácia Chlorosu na Linuxu

Príkaz `update` je dostupný iba pre Linux/Jetson. Skontroluje verziu uverejnenú v kanáli aktualizácií nakonfigurovanom v `/etc/chloros/update.conf` a ponúkne stiahnutie a inštaláciu zodpovedajúceho `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

V systéme Linux/Jetson vykonáva CLI pri každom spustení aj neblokujúcu kontrolu aktualizácií (výsledok je uložený v cache na jednu hodinu v `~/.chloros/update_cache.json`) a v prípade existencie novej verzie vypíše `Update available: vX.Y.Z`. Vaše nastavenia, projekty a prihlásenie zostanú po aktualizácii zachované.

Aktualizácia balíka počas inštalácie zastaví všetky backendové služby, takže bežiaci skript stratí prístup ku kamerám a senzorom a následne znovu spustí všetky, ktoré sú povolené (služba spustená ručne, ale nikdy nepovolená, zostane zastavená, rovnako ako po reštarte).

## Odinštalovanie

```bash
sudo apt remove chloros
```

Odstránenie zastaví a deaktivuje všetky backendové služby, obnoví predvolenú minimálnu hodnotu pre neprivilegované porty (1024), odstráni konfiguračný súbor prijímacieho vyrovnávacieho pamäťového priestoru (aktuálne hodnoty zostanú zachované až do reštartu), symbolický odkaz na nástroj exiftool, konfiguráciu zavádzača Arena, pravidlá firewallu pridané balíkom a nainštalované Python SDK a vymaže uložené prihlasovacie údaje z vyrovnávacej pamäte. Vaše projekty a dátové súbory `~/.chloros/` zostanú nedotknuté. `sudo apt purge chloros` tiež odstráni `/var/log/chloros`.

***

## Ďalšie kroky

* [Príručka NVIDIA Jetson](nvidia-jetson-guide.md) — Optimalizácia a nasadenie špecifické pre Jetson
* [CLI: Príkazový riadok](../CLI.md) — sprievodca CLI
* [API: Python SDK](../api-python-sdk.md) — sprievodca SDK
* [Referencia CLI](../reference/cli-reference.md) a [Referencia SDK](../reference/sdk-reference.md) — vyčerpávajúce zoznamy príkazov a API pre verziu 1.3.0
* [Dynamická adaptácia výpočtového výkonu](../processing-architecture/dynamic-compute-adaptation.md) — ako sa Chloros prispôsobuje vášmu hardvéru

# Chloros Python SDK Referencia

**Verzia:**

1.2.0**Vytvorené:**

29. 7. 2026 19:19 ·**Revidované:**

30. 8. 2026**Balík:** `chloros-sdk` (PyPI)**Cieľová skupina:** Optimalizované pre použitie v LLM; čitateľné pre človeka.**Rozsah:** Všetky verejné triedy, funkcie a pomocné funkcie sprístupnené balíkom `import chloros_sdk`, s príkladmi, ktoré je možné skopírovať a vložiť, pokrývajúce spracovanie obrazu, ovládanie jednej kamery, synchronizované polia, senzory DAQ a automatizáciu projektov.

Ak potrebujete len to najdôležitejšie, prejdite na:
- [Inštalácia a rýchly štart](#installation)
- [Smart-Connect pre pole LATTICE](#smart-connect-for-lattice-cameras)
- [Relácie senzorov DAQ](#daq-sensor-sessions)
- [Automatizácia projektov](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Architektúra za 60 sekúnd

SDK je tenká vrstva typu „Python“ nad backendom Chloros (ten istý server Flask, ktorý používajú aj desktopové grafické rozhranie a CLI). Na účely automatizácie importujete `chloros_sdk` a voláte metódy vyššej úrovne; v pozadí sa každé volanie premení na požiadavku typu HTTP smerujúcu na lokálny backend na porte 5000 — `http://127.0.0.1:5000/api/...` (úmyselneúmyselne nie `localhost`, ktorý sa najskôr prekladá na `::1` na Windows a stojí ~2 s na požiadavku pri backende podporujúcom iba IPv4). Backend spravuje hardvérový fond — kamery, senzory DAQ, profily zarovnania, rámcové vyrovnávacie pamäte — takže skripty SDK môžu ko- koexistovať s grafickým rozhraním bez toho, aby sa museli deliť o sériové porty alebo šírku pásma sieťovej karty.

K dispozícii sú tri rozhrania, ktoré budete používať:

1. **`ChlorosLocal` + voľné funkcie** (`process_folder`, `process_lattice_capture`) — Spracovateľský reťazec obrazu. Spustite spracovanie celého priečinka vrátane kalibrácie, debayeringu a exportu indexu pomocou jediného volania Python.
2. **Smart-connect handles** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Otvorenie trvalej relácie backendu pre živý hardvér. Rovnaký postup „smart-prep“ ako v grafickom rozhraní: sieťová sonda, automatický výber vrstvy, PTP, inicializácia AE, konfigurácia spúšťača GPIO.
3. **`ChlorosProject` / `open_project`** — Načítanie uloženého projektu (zložka s `cameras.json` + `sensors.json` + `project.json`), pripojenie všetkého naraz a snímanie pomocou pomenovaných rukovätí.

Rozhrania 1 a 2 **automaticky spustia lokálny backend**, ak ešte žiadny nepočúva (ide o ten istý balík binárnych súborov, ktorý spúšťa GUI/CLI) — takže holý skript funguje z nového shellu bez toho, aby ste museli najprv spustiť backend. Ak sa chcete odhlásiť, odovzdajte `auto_start_backend=False` (napr. pri smerovaní na vzdialený backend, ktorý sa nikdy nespustí). Pozrite si [Automatické spustenie backendu](#backend-auto-start). Surface 3 sa správa inak: `open_project()` neprijíma žiadny parameter `auto_start_backend` a `connect_all()` nikdy nespustí backend — raz skontroluje `http://127.0.0.1:5000` a ak nič neodpovie, ticho prejde na priame (bez backendu) ovládanie zariadenia `lattice_sdk`. Iba `proj.process()` a `stream(..., overlays=True)` odložene vytvoria `ChlorosLocal()` (ktorý vykonáva automatickýštart).

Všetky tri sú chránené autentifikáciou: spustite `chloros-cli login` raz na danom počítači alebo sa prihláste cez grafické rozhranie pracovnej plochy. Volania SDK bez platnej relácie vyvolajú chybu `ChlorosAuthenticationError`.

Požiadavky:
- Python 3.7+ (ako uvádza balík; vyvinuté/testované na verzii 3.10)
- Lokálne nainštalovaný program Chloros Desktop (binárny súbor backendu je súčasťou inštalátora)
- Aktívne prihlásenie na Chloros+. Minimálna úroveň na stránkach SDK / CLI je **Copper**alebo vyššia (Copper / Bronze / Silver / Gold); bezplatná úroveň**Iron**nemá prístup na SDK / CLI. Toto sa vynucuje**na strane servera**: každá požiadavka s príznakom SDK / CLI musí obsahovať aktívnu reláciu aj platený plán, inak backend vráti kód chyby `403` s kódom `error_code: PLAN_UPGRADE_REQUIRED` (zobrazený ako `ChlorosLicenseError` pomocnými funkciami `ChlorosLocal` a ako `ChlorosConnectError` pomocnými funkciami `connect_*`). Odhlásený volajúci namiesto toho dostane chybu `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) – tieto dva sú odlišné, pretože opätovné spustenie `chloros-cli login` vyrieši prvý problém, ale druhý vyriešiť nedokáže.
- Offline používanie je podporované v rámci: úroveň sa číta z vyrovnávacej pamäte overovania serverom (5 minút) alebo z vyrovnávacej pamäte podpísanej licencie viazanej na zariadenie (30 dní pre mesačné plány, do vypršania predplatného v prípade ročných plánov). Po uplynutí tohto obdobia odkladu sa plán zmení na bezplatný a prístup na SDK / CLI sa pozastaví, kým sa zariadenie aspoň raz nepripojí k serveru. Kód `chloros-cli status` (`GET /api/license-status`) zostáva dostupný v rámci bezplatnej úrovne, takže dôvod je zrejmý — je to jediná trasa SDK / CLI, ktorá je vyňatá z obmedzenia úrovne.
- Windows 10/11 64-bit, **Ubuntu 22.04 LTS alebo novší**, alebo Jetson (JetPack 6). Ubuntu 20.04**nie je** podporované: závislosti `.deb` sú odvodené od toho, na čo sa odkazuje backend, vrátane `libc6 (>= 2.34)`, a verzia Focal obsahuje glibc 2.31.

---

## Inštalácia

Python SDK je tenká vrstva Python nad backendom Chloros. Pre všetko, čo presahuje niekoľko pracovných postupov zameraných výlučne na DAQ, potrebujete **lokálne nainštalovaný balík Chloros pre pracovnú plochu** (inštalačný program Windows alebo Linux `.deb`) — ten poskytuje binárny súbor backendu, runtime prostredie Arena SDK pre kamery LATTICE a kalibračné balíky.

Najnovšie súbory na stiahnutie: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Krok 1 — Nainštalujte balík platformy Chloros

#### Windows (.exe)

1. Stiahnite si súbor `Chloros-Setup-x.y.z.exe` zo stránky na stiahnutie.
2. Spustite inštalačný program a postupujte podľa pokynov sprievodcu. Predvolená inštalačná cesta je `C:\Program Files\MAPIR\Chloros\`.
3. Spustite aspoň raz stránku Chloros a prihláste sa pomocou svojho účtu Chloros+.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Krok 2 — Inštalácia balíka „Python“ SDK

**Inštalátor Chloros obsahuje zodpovedajúci balík „SDK“ (wheel).** Každý inštalátor Windows a balík Linux .deb umiestni na disk súbor `chloros_sdk-X.Y.Z-py3-none-any.whl`, ktorý presne zodpovedá verzii GUI / CLI / backendu. Nemusíte sledovať PyPI, aby ste zostali v synchronizácii.

#### Windows

Inštalátor automatickyspustí `pip install` na dodanom súbore wheel pomocou vášho systémového Python (uprednostňuje sa spúšťač `py.exe`, v prípade potreby sa použije `python -m pip`). Nie je potrebná žiadna akcia — `import chloros_sdk` funguje vo vašom prostredí Python po úspešnej inštalácii. Ak na počítači nie je nainštalovaný Python, inštalátor tento krok ticho preskočí a grafické rozhranie + CLI naďalej fungujú.

#### Linux (.deb)

Súbor .deb umiestni balík wheel do adresára `/usr/lib/chloros/sdk/`. Príkaz `postinst` vypíše presný príkaz — Distribúcie podľa PEP 668 štandardne odmietajú globálne zápisy pip, preto neprebieha automatická inštalácia:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

V prípade nasadenia Jetsonu v izolovanom prostredí (air-gapped) prebieha tento proces úplne offline — balík wheel je už na disku.

#### Verejný PyPI

Pre hostiteľa používajúceho iba pip (bez nainštalovaného balíka Chloros pre desktop; pracovné postupy so vzdialeným backendom alebo iba s DAQ):

```bash
pip install chloros-sdk
```

PyPI sa aktualizuje pri zostavovaní inštalátorov pre release verzie, takže zverejnený balík wheel zodpovedá najnovšej stabilnej verzii. Vývojárske zostavenia (napr. `1.1.4.dev1`) sa dodávajú iba prostredníctvom balíka s inštalátorom.

#### Overenie

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Vyžaduje sa predplatné Chloros+.** Všetky volania na SDK vyžadujú aktívne prihlásenie na Chloros+. Spustite `chloros-cli login user@example.com 'YourPassword'` raz na každom počítači; prihlasovacie údaje sa ukladajú do vyrovnávacej pamäte v súbore `~/.chloros/`.

### Potrebujem balík pre stolné počítače?

Samotný balík pip **nie je** pre väčšinu pracovných postupov dostatočný. Tu je prehľad toho, čo potrebuje každý povrch SDK:

| PovrchSDKu | Potrebuje balík Desktop Package? | Prečo |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Áno** | Automaticky spúšťa binárny súbor backendu na `/usr/lib/chloros/chloros-backend` (Linux) alebo `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Áno**(lokálne)**/ Nie**(vzdialený) | Čisto HTTP klienti cez backend. Lokálny backend → vyžaduje sa balík pre desktop. Vzdialený backend → `backend_url=`**cez tunel** (pozri Režim vzdialeného backendu — dodávané backendy viažu len loopback). |
| `ChlorosProject` / `open_project` | **Áno** | Umožňuje prácu s uloženými projektmi cez backend. |
| Priame triedy LATTICE (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Áno** | Vyžaduje natívne runtime prostredie Arena SDK, ktoré je súčasťou balíka pre stolné počítače. V opačnom prípade je `CAMERA_AVAILABLE` rovnaké ako `False` pri importe. |
| Triedy priameho DAQ (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Nie** | Čistý Python cez pyserial/bleak/zeroconf. Prostredie založené výlučne na pip môže riadiť DAQ zariadenia od začiatku do konca. |

### Režim vzdialeného backendu (hostiteľ len s pip, cez tunel)

> **Dodávaný backend nie je dostupný cez LAN.** Produkčná verzia
> zostavenia viažu iba loopback (obe rodiny loopback) a kategoricky odmietajú
> jediný režim bez loopbacku (`CHLOROS_CLOUD_MODE`), takže
> `backend_url="http://<lan-ip>:5000"` **nemôže fungovať s nainštalovaným
> Chloros** — tento vzor fungoval vždy len s backendom typu source/dev
> . Ak chcete ovládať backend na inom počítači, presmerujte jeho loopback
> port sami a nasmerujte SDK na tunel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Hostitelia bez grafického rozhrania / CI / robotika môžu mať jeden počítač s plnou inštaláciou pracovnej plochy ako „Chloros server“ a `pip install chloros-sdk` všade inde — prenos medzi nimi však zabezpečuje vyššie uvedený tunel nastavený používateľom, nie priama LAN URL.

> **Známe obmedzenie — `ChlorosLocal` nepodporuje výhradne pip.** `ChlorosLocal(backend_url=BACKEND)` v súčasnosti vyhľadáva lokálny binárny súbor backendu vo svojom konštruktore *predtým*, ako prehľadáva URL, a vygeneruje chybu `ChlorosBackendError` („Nenašiel sa backend Chloros…“), ak nie je nainštalovaný žiadny balík pre pracovnú plochu — a to aj v prípade, že je dostupný vzdialený backend. Iba vyššie uvedené rozhranie smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`, plus pomocné programy `analyze_array_network` a `list_*` / `discover_*`) fungujú z hostiteľa, ktorý používa iba pip.

### Pracovný postup iba s DAQ (hostiteľ iba s pip)

Ak potrebujete iba senzory DAQ a nevyužívate kamery LATTICE ani spracovanie obrazu, balík pip je samostatný:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Na prácu s DAQ priamo na hardvéri nie je potrebný žiadny backend, žiadny súbor .deb ani prihlásenie na Chloros+.

---

## Rýchly štart

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Index najvyššej úrovne API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Spracovanie obrazu — `ChlorosLocal`

Hlavná trieda spracovateľského reťazca. Pri prvom použití spustí backend, vytvorí a nakonfiguruje projekty, monitoruje priebeh a vráti súhrny po ukončení spustenia.

### Konštruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metódy

| Metóda | Popis |
| --- | --- |
| `create_project(project_name, camera=None)` | Vytvorenie nového projektu (voliteľne s šablónou fotoaparátu, napr. `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Importuje obrázky vo formátoch RAW/TIF/JPG/DNG **a záznamy zo svetelného senzora `.daq`**. Vráti `count` (obrázky) a `scan_count` (záznamy). Upozorní len v prípade, ak zložka neobsahuje ani jedno z nich. |
| `export_light_sensor(daq=True, csv=True)` | Zapíše kalibrované súbory `.daq` + `.csv` pre každý záznam zo svetelnéhosvetelného senzora v projekte do súboru `<project>/Light Sensor/`. Pozri [Záznamy svetelného senzora](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Nastavte ovládače spracovania. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Spustite spracovateľský reťazec. Vráti `{"status": "complete", "async": False}` a kľúč `summary`, ak ho backend poskytne — pozri [Súhrn po spustení a Tipy](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Skontroluje stav backendu. |
| `logout()` | Vymaže uložené prihlasovacie údaje z vyrovnávacej pamäte. |
| `shutdown_backend()` | Ukončí backend (ak bol spustený pomocou príkazu `SDK`). |
| `discover_cameras()` | Vyhľadaj kamery LATTICE **prostredníctvom backendu tejto inštancie** (`/api/camera/discover`). Vráti zoznam slovníkov (`serial`, `model`, `ip`, …) — v rovnakom formáte, aký vidí GUI/CLI. Prázdny zoznam, ak sa nenašli žiadne kamery alebo backend nie je dostupný. |
| `camera_capture(output_dir, format="tiff", **settings)` | Zachytí jeden snímok**prostredníctvom backendu**(automaticky spusteného týmto identifikátorom), aby mal rovnakú prípravu ako v GUI/ CLI (predvolene 12-bitové, opätovné použitie zásobníka, vložené kalibračné metadáta). Cieľ spracujte pomocou `serial=` alebo `device_index=`; odovzdajte `exposure`/`gain`/`pixel_format`/`preset` ako `**settings`. Vráti slovník starších metadát (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Vytvára prekrývané kompozitné náhľadové snímky zo združenej kamery — tenký MJPEG klient cez trasu `/api/camera/<serial>/stream-annotated` na strane backendu (zebra / mriežka / nitkový kríž / histogram / peaking / bod nakreslený na strane servera). `decode=True` poskytuje polia BGR; `False` poskytuje surové bajty JPEG. Dostupné aj na úrovni projektu ako `ChlorosProject.stream(overlays=True)`. |

Použite ako správcu kontextu pre zaručené vyčistenie:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Záznamy zo svetelných senzorov — kalibrované `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E je možné zaznamenávať **bez** jeho kalibračného balíka. To je to,
čo verejné [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`): zapisujú surové hodnoty zo senzorov a označujú
súbor tak, aby stránka Chloros najprv načítala továrenskú kalibráciu daného senzora **podľa sériového čísla** — najskôr z lokálnej cache
, potom z cloudu MAPIR — a aplikuje ju pri importe.

Nástroj Chloros zapíše výsledok späť ako dva produkty na jeden záznam pod
`<project>/Light Sensor/`:

| Produkt | Čo to je |
| --- | --- |
| `<name>_calibrated.daq` | Archív, ktorý je možné opätovne spracovať — rovnaká schéma ako pri živom zázname, teraz však uvádza balík, ktorý ho vytvoril. Jeho opätovný import ho **nekalibruje** druhýkrát. |
| `<name>_calibrated.csv` | Spektrálna intenzita žiarenia v W/m²/nm na vlastnej vlnovej mriežke senzora, jeden riadok na jedno meranie, plus fotometrické stĺpce (celkový výkon, fotopický/skotopický lux, PPFD a jeho rozdelenie na modrú, zelenú a červenú zložku, špičková vlnová dĺžka). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Iba senzory bez balíka (DAQ-A).** Surové spektrálne počty senzora — *nie* ožiarenie. Pozri nižšie. |

`process()` vykonáva tento export ako jednu zo svojich fáz. **Nevyžaduje** snímky:
samostatne lietajúci svetelný senzor predstavuje prvotriedny pracovný postup a takýto projekt má konštrukčne nulový
počet snímok.

**Záznamy DAQ-A sa exportujú ako surové počty.** Rodina DAQ-A predchádza systému zväzkov na sériové čísla
a nemá žiadny zväzok, ktorý by bolo potrebné načítať — namiesto toho sa kalibruje v teréne voči
reflexnému terču, a preto ho nikdy nepotreboval. Tieto záznamy sa exportujú
pod kmeňom `_raw` namiesto `_calibrated`: ide o odlišný názov súboru namiesto príznaku
v súbore, pretože údaj musí prežiť odoslanie e-mailom ako holý názov. Hlavička
`.csv` uvádza `raw spectral sensor counts (NOT irradiance)` a upozorňuje, že
hodnoty sú porovnateľné **v rámci** súboru — presne to, na čo sa používa kalibrácia na základe cieľa
— a nie medzi senzormi. Fotometrické stĺpce závislé od výkonu (celkový výkon,
fotopický/skotopický lux, PPFD) vracajú hodnotu **NULL** namiesto toho, aby boli integrované z počtov.

DAQ-U / DAQ-M / DAQ-E, ktorých balík sa jednoducho nedal načítať, sa stále **preskočia**,
nezaznamenávajú sa v surovom formáte: v takom prípade balík existuje a „opätovné pripojenie a opätovné spracovanie“ je skutočná rada.

Staršie záznamy **v1.01 / v1.02** (zapisuje ich DAQ-A-SD) neobsahujú epochu pre každé odčítanie,
iba čas zápisu súboru. Porovnávač obrazu ↔ downwelling ich stále odmieta — porovnávanie
snímky s časom zápisu by bolo neviditeľne nesprávne — ale exportér ich číta a
CSV vytlačí `clock=daq_created_on`, takže produkt uvádza, na akých hodinách beží.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Záznam, ktorého kalibračný balík nie je možné načítať (offline alebo senzor bez
kalibrácie v súbore), sa nahlási pod kódom `skipped` **s uvedením dôvodu**. Nikdy sa
nezapíše ako „kalibrovaný“ súbor obsahujúci surové počty — pripojte sa k internetu a
spustite proces znovu, čím sa export dokončí.

### Spätné volania o priebehu

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Súhrn a tipy po spustení

Po dokončení `process()` načíta `GET /api/processing-summary` a pripojí telo ako `result["summary"]`. Načítanie prebieha podľa najlepších možností a nikdy neblokuje úspešný návrat – ak nie je súhrn k dispozícii, `process()` sa vráti k jednoduchému formátu `{"status": "complete", "async": False}`. Každý záznam v `summary["hints"]` – celé vety s navrhovaným riešením, napr. prečo beh vyprodukoval nulový výstup — sa tiež opätovne odošle ako Python `UserWarning`, takže behy s nulovým výstupom sú samodiagnostické, aj keď slovník nikdy nepreveríte:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` je strojovo čitateľná polovica:

| Kľúč | Čo počíta |
| --- | --- |
| `models` | Skupiny kamier v behu. |
| `images_in_groups` | Zdrojové obrázky v týchto skupinách. |
| `targets_found` | Zistené ciele odrazivosti. |
| `images_calibrated` | Snímky, ktoré boli v priebehu kalibrované. |
| `exported_files` | **Súbory obrazových výstupov, ktoré beh vytvoril.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Záznamy svetelných senzorov, úmyselne spočítané samostatne — pochádzajú z inej fázy a existujú aj pre behy bez akýchkoľvek snímok, takže ich zahrnutie by spôsobilo, že beh len s DAQ by vyzeral, ako keby exportoval snímky. |

Okrem nich: `summary["output_dirs"]` (každý adresár, do ktorého sa zapisovalo),
`summary["light_sensor_export"]`, `summary["stopped"]` (hodnota „true“, ak používateľ prerušil
beh, takže čiastočné počty sa neinterpretujú ako dokončený beh s nedostatočnou produkciou) a
`summary["groups"]` (rozpis podľa skupín).

`exported_files` sa zaznamenáva v procese **počas zápisu**, nie sa neskôr skenuje z
objektov obrázkov projektuobjektov obrazu. Paralelné a GPU stratégie vytvárajú svoje vlastné obrazové
objekty (v pracovných podprocesoch pre GPU cesty), takže staré skenovanie hlásilo
`0 file(s) written` pre každé takéto spustenie a následne vysielalo upozornenie o nulových exportoch — pri spusteniach,
pri ktorých všetko fungovalo. Ak vytvoríte skript na základe tohto čísla, zdravé paralelné spustenie teraz
hlási nenulový počet.

Preskočenia detekcie ľahkých súborov hlásia dôvod, ktorý čítač skutočne zistil pre každý súbor —
nečitateľná schéma, chýbajúci balík, chyba zápisu — **duplikáty sa odstránili**, takže dvadsať súborov
preskočených z jedného dôvodu sa vyhodnotí ako jeden dôvod, a nie ako dvadsať opakovaní tohto dôvodu.

> **`process()` sa nevygeneruje, ak spustenie nevytvorí žiadne obrázky.** Toto je jediné miesto, kde sa SDK a
> CLI zámerne líšia: `chloros-cli process` považuje „boli požadované produkty, žiadne neboli
> zapísané“ ako zlyhanie a ukončí sa s nenulovým kódom, zatiaľ čo SDK sa vráti normálne a nahlási tento
> stav prostredníctvom `summary` / hints. Ak by sa váš pipeline mal zastaviť pri prázdnom behu, skontrolujte to
> sami — skontrolujte `summary` (alebo spočítajte súbory v projektovej zložke) namiesto toho, aby ste sa spoliehali na
> absenciu výnimky. Bežnými príčinami sú vstupná zložka, ktorá nebola rozpoznaná ako
> zachytenie a produkty, ktoré boli vynechané ako nevhodné pre prítomné kamery (napr. jas z kamier podporujúcich iba RGB
>).

### Pomocné funkcie

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Podporované hodnoty

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrický výstup (multispektrálny spracovateľský reťazec LATTICE)

Úroveň exportu multispektrálnych údajov (M3C/M3M) v spracovateľskom reťazci `process` — `reflectance` (predvolené), `radiance`, `sensor-response` alebo `all` (každý príslušný režim pre každý obrázok) — zodpovedá nastaveniu spracovania **„Radiometric output“** v projekte. `configure()` má na to vyhradené kľúčové slovo:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Pokročilá úniková možnosť — zápis kľúča projektu `"Radiometric output"` prostredníctvom `custom_settings` — stále funguje, ale pamätajte, že nahrádza celý blok nastavení (pozri varovanie nižšie):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (predvolené nastavenie) delí žiarivosť kamery **s časovou pečiatkou zosúladeným DAQ downwellingom**, automaticky vypočítaným zo zaznamenaného `.daq` (DAQ-U/M/E)**alebo natívneho `.csv` z DAQ-M**, ktorý sa nachádza spolu so snímkami; akýkoľvek kalibračný balík pre jednotlivé kamery alebo DAQ, ktorý lokálne chýba, sa**automaticky stiahne z AWS** pri prvom použití. Služba CLI to sprístupňuje vo forme prepínačov pre jednotlivé typy produktov v súbore `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **nahrádza** celý blok vypočítaných nastavení (podľa návrhu obchádza ostatné kľúčové slová a validáciu `configure()`). Ak ho použijete, zahrňte všetky kľúče `Project Settings`, ktoré sú pre vás dôležité, ako je uvedené v príklade vyššie.

---

## Smart-Connect pre kamery LATTICE

Trvalé relácie backendu pre živý hardvér. Používajú sa rovnaké koncové body ako v grafickom rozhraní, takže správanie je identické na stránkach SDK / CLI / v grafickom rozhraní.

### Jedna kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### `connect_camera()` Signatúra

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession` Metódy

| Metóda | Popis |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Načíta uzly GenICam; vráti `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Zapíše uzly podľa priateľského názvu (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Zachytí **jeden** snímok. Vráti zoznam s jedným prvkom obsahujúci slovníky metadát snímku. (Zachytávanie sériových/viacerých snímok bolo odstránené — ak potrebujete sériu, volajte `capture()` v slučke.) |
| `disconnect()` | Uvoľnenie z fondu. Žiadna operácia, ak sme sa pripojili k už otvorenej relácii. |

`capture()` – kontroly exportu (rovnaký model ako pole + grafické rozhranie):

- `processing` / `levels` — `processing="all"` ukladá všetky príslušné typy exportu; `levels=["raw","radiance"]` ukladá len tie (prepíše `processing`). Ak chcete použiť predvolené nastavenie backendu, vynechajte obe.
- `force_daq=True` — uloží priradené meranie DAQ/DLS ako `.daq` sidecar aj pri zachytení iba surových dát, aby bolo možné snímku neskôr opätovne spracovať na odrazivosť/index. Žiadna operácia, ak nie je prepojený žiadny DAQ.

### Synchronizované pole — `ArraySession` (Smart-Prep)

`connect_array` je **odporúčaný vstupný bod** pre konfigurácie s viacerými kamerami. V pozadí spúšťa kompletný proces smart-prep cez grafické rozhranie:

1. **Analýza siete** (`/api/camera/array/recommend`) — nájde najväčšiu veľkosť rámca, ktorá sa zmestí do úrovne sim-emit bez straty rámcov.
2. **Automatický výber úrovne** — `sim-capture-sim-emit`, ak to kábel zvládne; inak `sim-capture-ftd-stagger` alebo `slip-emit-and-capture`.
3. **Automatické zmenšenie**— bez upozornenia zmenší veľkosť rámca / zvýši zlučovanie, ak linka nedokáže udržať požadované rozlíšenie.**Táto bezpečnostná sieť nezahŕňa agregované preťaženie**: príliš veľa kamier pre sieť nie je možné vyriešiť zmenšením snímok — pozri [Preťaženie](#over-subscription-the-per-cam-floor).
4. **PTP je povolené**štandardne — časové značky medzi kamerami sa zosúladia na jednom spoločnom časovači s presnosťou**~1 ms**. Súčasná expozícia je riadená hardvérovým spúšťačom M8 (**&lt; 100 µs** medzi modulmi), nie prostredníctvom PTP: PTP synchronizuje *časové značky*, nie expozície.
5. **Automatický výber formátu pixelov pre každú kameru** — RGB kamery → `BayerRG8`, multispektrálne → `BayerRG12`.
6. **Inicializácia AE** — zaznamenáva aktuálny stav AE každej kamery, aby sa expozícia počas prevádzky pri pripojení nezresetovala.
7. **Konfigurácia spúšťača GPIO** — `connect_array` aktivuje každú kameru (`TriggerMode=On`, `TriggerSource=Line2`), aby impulz hlavnej kamery riadil podriadené kamery cez kábel M8. Ide o krok určený výlučne pre pole: samostatná kamera otvorená pomocou `LatticeCamera` namiesto toho beží vo voľnom režime.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()` Podpis

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

Hodnoty `force_tier`:
- `"sim-capture-sim-emit"` — skutočná simultánnosť (všetky kamery vysielajú na rovnakom okraji taktu).
- `"sim-capture-ftd-stagger"` — flexibilné časové posunutie (kamery vysielajú s miernym časovým posunom, takže pakety sa na vedení sériovo usporiadajú).
- `"slip-emit-and-capture"` — sekvenčné zachytávanie podľa kamery (bez časovej synchronizácie; jediná možnosť, ak žiadna veľkosť rámca nezodpovedá simultánnemu režimu).

`wire_ceiling_mbps` prepisuje **trvalý rozpočet hostiteľa na prenosovú šírku pásma** v MB/s — jediné
číslo, od ktorého závisí alokácia celého poľa. Nechajte nastavenie na `None`, aby sa použila automaticky zistená
hodnota. Znížte ju, ak pole hlási rámce poškodené podľa GVSP: automatická hodnota je odvodená
od udávanej rýchlosti spojenia sieťovou kartou, ktorá nadhodnocuje USB adaptéry, úzke PCIe kanály a
vyťažené zdieľané štruktúry — a toto nadhodnotenie sa prejavuje skôr ako poškodené rámce než ako
viditeľne pomalé pripojenie. Hodnota sa ukladá do bloku zachytávania poľa v projekte, takže jej
opätovné otvorenie alebo neskoršie nastavenie na hodnotu `connect_array` ju obnoví rovnako ako akékoľvek iné nastavenie poľa.
Pozri [Stav poľa](#array-health--which-subsystem-is-losing-frames).

#### Nadmerné predplatné (minimálna hodnota na kameru)

Pacing simulovaného vysielania prideľuje každej kamere podiel z rozpočtu prepojenia odolného voči kolíziám, s minimálnou hodnotou **8 MB/s na kameru**(`per_cam_floor_bps`). Akonáhle `N × floor` prekročí hornú hranicu zabezpečujúcu ochranu proti kolíziám, pole**prekročí kapacitu kanála**— režimom zlyhania je strata paketov GVSP, nie nižšia snímková frekvencia — a neexistuje žiadne riešenie týkajúce sa veľkosti snímky:**zlučovanie a ROI znižujú počet bajtov na snímku, nie počet bajtov za sekundu regulovaný algoritmom**, ktoré porovnáva agregovaná kontrola. Praktické stropy pri plnom rozlíšení na hostiteľskom zariadení s 1 GbE:**6 kamier pri 1500 MTU, 9 s jumbo rámcami** (`max_cams_collision_safe` v odpovedi analýzy uvádza limit pre vaše pripojenie). Riešenia: menej kamier, jumbo rámce v celom reťazci alebo rýchlejšia sieťová karta.

- Odpovede `analyze_array_network()` a `/api/camera/array/connect` obsahujú `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` a `per_cam_floor_bps`. Ak je hodnota `oversubscribed` pravdivá, projekcia **nuluje polia fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) namiesto toho, aby hlásila zavádzajúcu hodnotu rýchlosti, ktorá je síce nízka, ale funkčná.
- `POST /api/camera/array/connect` prijíma parameter tela `pin_resolution` (**len HTTP — nie je to kľúčový argument SDK**; `connect_array` ho nezverejňuje). Pripútanie odstraňuje bezpečnostnú sieť zníženia rýchlosti, takže preťažené pripojenie s nastaveným `pin_resolution` je**tvrdo odmietnuté** s chybou, ktorá uvádza všetky možné riešenia. Bez fixácie pripojenie pokračuje v znižovaní, ale varuje, že zmenšenie nemôže vyčistiť agregát.
- Úniková cesta pre testovanie: nastavte `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` v prostredí backendu, čím znížite úroveň odmietnutia na výrazné varovanie — pripojíte sa aj tak a akceptujete stratu paketov.

#### Stav poľa — ktorý subsystém stráca rámce

`GET /api/camera/array/<array_id>/capability` obsahuje aktívny blok `health` na
pripojenom poli, ktorý sa prehodnocuje v priebežnom **10-sekundovom** okne. Rozdeľuje stratu rámcov
na dve príčiny, ktoré si vyžadujú opačné riešenia, namiesto jednej miery „neúplnosti“, ktorá
nešpecifikuje ani jednu z nich:

| Pole | Čo to znamená | Ktorý subsystém |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (na sériový port) | Rámec **dorazil, ale bol štrukturálne poškodený**— strata paketov GVSP. |**Sieť**: kapacita linky, tempo, prstenec prijímača NIC, MTU |
| `never_arrived_rate_pct` (podľa sériového čísla) | Rámec **vôbec neprišiel**— kamera sa nespustila alebo z nej nič nevyšlo. |**Spúšť / synchronizácia**: kábel M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Najhoršia rýchlosť kamery pre každý z nich. | — |
| `per_cam_rate_pct` | Kombinovaná miera neúplnosti na fotoaparát (obe príčiny spolu). | — |
| `stable_for_seconds` | Ako dlho každý fotoaparát zostal pod 0,01 %. | — |

Okrem `health` uvádza ten istý záznam aj číslo, na ktorom visí celá alokácia:

| Pole | Čo to znamená |
| --- | --- |
| `wire_ceiling_mbps` | Platný trvalý rozpočet šírky pásma hostiteľa, MB/s. |
| `wire_ceiling_source` | Odkiaľ táto hodnota pochádza, slovami — napr. `USB-capped 200 MB/s (was theoretical 1062; …)` alebo `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, keď ho nastavil `wire_ceiling_mbps=`. |
| `nic_is_usb` | `true` pre USB ethernetový adaptér. |

Pre tento koncový bod neexistuje žiadny SDK wrapper — prečítajte ho priamo:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Čítanie:** hodnota odlišná od nuly `gvsp_corrupt_rate_pct` s hodnotou `never_arrived_rate_pct` rovnou 0 znamená,
že spúšťanie a synchronizácia kábla sú bezchybné a 100 % strát je na sieťovej trase – nižšia
`wire_ceiling_mbps` a znovu sa pripojte. Opačný vzor poukazuje skôr na synchronizačný kábel alebo
spúšťaciu linku.

> **`target_fps` nie je faktorom spôsobujúcim poškodené rámce.** Frekvencia GevSCPD sa nastaví jednorazovo pri
> pripojení, takže zníženie spúšťacej frekvencie mení pracovný cyklus, nie však
> rýchlosť súbežného vysielania. Zmerané 5-násobné zníženie požiadaviek neprinieslo žiadne zlepšenie, zatiaľ čo
> zníženie maximálnej rýchlosti z 240 na 200 MB/s znížilo podiel poškodených rámcov na tej istej zostave z 10,4 % na
> 0,00 %.

> **Automatické zmenšovanie v priebehu prenosu nie je k dispozícii vo firmvéri TRI032S.** Bežiace pole to nedokáže
> samo opraviť; odpojte a znovu pripojte, aby nástroj na výber času pripojenia preplánoval na základe
> nového stropu.

**USB ethernetový adaptér je obmedzený na 200 MB/s** sondou bez ohľadu na jeho
špecifikácie: tabuľka účinnosti, ktorá premieňa rýchlosť spojenia na trvalú hodnotu, je
odvodená od PCIe, a sieťová karta USB síce udáva rýchlosť ethernetového spojenia, je však obmedzená
zbernicou USB a jej ovládačom. Obmedzenie je absolútne, nie percentuálne – adaptér USB 1 GbE
dosahuje rýchlosť ~80 MB/s a nie je týmto obmedzením ovplyvnený.

#### Metódy `ArraySession`

| Metóda | Popis |
| --- | --- |
| `status(timeout=10.0)` | Live `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Jedna synchronizovaná skupina zachytení. Vráti `CaptureResult` (zoznam slovníkov snímok + `.skipped`). Ovládacie prvky exportu sú uvedené nižšie. |
| `capture(..., smart=True)` | **Inteligentné snímanie** — čaká, kým sa AE ustáli na všetkých kamerách, a potom spustí snímanie. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Najrýchlejšie snímanie: iba surové dáta + priradená hodnota DAQ (+ voľný kombinovaný index). Zodpovedá tlačidlu „Najrýchlejšie snímanie“ v grafickom rozhraní. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Jednorazové / nepretržité / intervalové snímanie v jednej ohraničenej slučke. Vráti `list[CaptureResult]`.**Vyžaduje `count` a/alebo `duration_s`**, aby sa proces ukončil (SDKk nemá funkciu Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Spustí nahrávanie živého zobrazenia kombinovaného indexu do formátu video/GIF → `RecorderHandle`. Jeden kompozitný záznamník na jedno pole. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Spustí sériové snímanie s vysokou snímkovou frekvenciou-Bayer sériového snímania s vysokou snímkovou frekvenciou → `RecorderHandle`. Offline spracovanie pomocou `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Offline spracujte uloženú sériu snímok vo formáte RAW na kalibrované videá. Blokuje sa, kým sa úloha nedokončí (`wait=True`) a vráti `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Kontroluje stav offline úlohy zostavovania: `{running, result, error, burst_dir}`. |
| `disconnect()` | Uvoľnenie celého poľa. |

`capture()` – kontroly exportu (rovnaký koncový bod, aký používa grafické rozhranie/CLI):

- `processing` / `levels` — `processing="all"` (alebo `levels=["raw","radiance",…]`) ukladá každý príslušný typ exportu pre každú kameru; jediná hodnota `processing` ukladá len túto úroveň.
- `aligned=True` — deformuje ne-surový export každého člena podľa [profilu zarovnania](#array-alignment) poľa (spoluregistrovaný); surové údaje zostávajú bez deformácie, ale transformácia je zaznamenaná v metadátach. Ak pole nemá profil, prejde sa na nezarovnané (s varovaním zobrazeným v `alignment` výsledku).
- `render_index=False` — preskočiť prekrývanie indexu vegetácie pre každú kameru; predvolene sa vykresľuje tam, kde je to nakonfigurované.
- `force_daq=True` — uložiť priradené hodnoty DAQ/DLS ako súbor typu sidecar `.daq`, aj keď to žiadna zvolená úroveň nevyžaduje.

**Kompresia TIFF (ovládač HTTP -only):**`ArraySession.capture()` neodosiela kľúč `compression`, takže sa uplatňuje predvolené nastavenie backendu — `POST /api/camera/array/capture` číta parameter tela `compression`, štandardne `"deflate"` (bezstratný zlib L1 + horizontálny prediktor, ~4,1 MB na snímku v plnom rozlíšení). `"none"` zapisuje nekomprimované údaje (~6,3 MB/snímka) s**~5× rýchlejším zápisom** — obidva formáty sú bezstratové a pri importe sa čítajú identicky. Rozhranie SDK pre to neposkytuje žiadny kwarg; únikovou cestou je `chloros-cli lattice array-capture --compression none` alebo surové HTTP. DEFLATE tiež drží GIL Python, takže komprimované zápisy sa neparalelizujú medzi vláknami zapisovača pre jednotlivé kamery — pri nepretržitom 8kamier v plnom rozlíšení pri rýchlosti snímača vyžaduje `compression: "none"`. Podrobnosti: [CLI Referencia → array-capture](cli-reference.md).**Prepísanie exportu na úrovni jednotlivých členov (len HTTP):**ten istý koncový bod prijíma aj `exclude_serials` (zoznam — vynechanie členov z uloženej sady; pole sa naďalej spúšťa ako jedna synchronizovaná skupina a vylúčení členovia sa vracajú v `excluded`), `serial_levels` (prepísania na úrovni kamery v súbore `{serial: [level tokens]}`)a `serial_index` (prepísania indexového prekrytia na úrovni jednotlivých kamier v `{serial: bool}`). Ide o parametre telesa zodpovedajúce grafickému rozhraniu (GUI) a**zatiaľ nie sú kľúčovo-hodnotové argumenty typu „SDK“**; členovia, ktorí chýbajú v mapách, sa nahradia hodnotami z celého poľa `levels` / `render_index`.

##### Kontrola preskočených kamier — `CaptureResult.skipped`

`ArraySession.capture()` vráti `CaptureResult`, čo je podtrieda `list`: prechádzajte ju, indexujte ho, spracujte ho pomocou `len()` — všetky existujúce vzory naďalej fungujú. Nový kód môže skontrolovať atribút `.skipped`, aby zistil, ktoré snímky boli vylúčené a prečo. Najbežnejším prípadom sú snímky typu „RGB“ v poli so zmiešanými filtrami, keď požiadate o `processing="radiance"` alebo `"reflectance"` — žiarivosť na jeden Bayerový pixel nemá pre širokopásmový senzor žiadny význam, takže backend tieto kamery preskočí, namiesto toho, aby produkoval nezmyselné údaje.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Tokeny dôvodov majú formát `<level>-not-applicable-to-rgb-cam` (jeden záznam na každú preskočenú úroveň, pričom každý obsahuje `level`). Preskočenia špecifické pre odrazivosť sú `reflectance-skipped-no-fresh-dls` (nie je k dispozícii žiadne nové meranie dopadajúceho žiarenia), `reflectance-skipped-bound-daq-unavailable (…)` (nebolo možné dosiahnuť pripojené zariadenie DAQ) a `dls-uncalibrated-band-<nm>` — pásmo leží prevažne mimo radiometricky kalibrovaného rozsahu svetelného senzora zariadenia DAQ (~374–974 nm), takže absolútne rozdelenie odrazivosti na základe zariadenia DAQ je odmietnuté a snímka sa výrazne zníži na senzorovúodozvy. Zo všetkých dodávaných modelov to spúšťa iba F988; podporovanou cestou pre túto kameru je pracovný postup s panelom odrazivosti.

Úrovne `processing`:

| Úroveň | Výstup |
| --- | --- |
| `"raw"` | Jednokanálový Bayer (mono kamery: jedno pásmo) priamo zo senzora. |
| `"debayered"` *(predvolené nastavenie SDK)* | 3-kanálový BGR prostredníctvom bilineárneho demosaicu (mono kamery: 1-kanálová škála šedých). |
| `"radiance"` | float32 W/m²/sr/nm cez kompletný radiometrický reťazec. Iba multispektrálne — kamery typu „RGB“ sa preskakujú. |
| `"reflectance"` | uint16 0..32768 (pripravené pre Pix4D); vyžaduje spárovanie s živým zberom dát (DAQ) pre absolútnu referenciu. Iba multispektrálne. |
| `"display"` | Úplný reťazec zodpovedajúci náhľadu v grafickom rozhraní (CCM + WB + gama podľa profilu kamery). |
| `"all"` | **Jeden súbor na príslušnú úroveň** pre každú kameru (zodpovedá predvolenému nastaveniu v grafickom rozhraní „Capture All“ / predvolené nastavenie „CLI“). Vrátený súbor `CaptureResult` potom obsahuje jeden slovník snímok na každý súbor `(cam, level)`, pričom úroveň je uvedená v každom slovníku; neuplatniteľné úrovne sa nachádzajú v súbore `.skipped`. Hodnota DAQ použitá pre akýkoľvek snímok odrazivosti sa uloží ako doplnkový súbor `.daq`. |

> **Poznámka — predvolené nastavenie sa líši od CLI.** `ArraySession.capture()` má predvolené nastavenie na `processing="debayered"`; príkaz `chloros-cli lattice array-capture` má predvolené nastavenie na `processing="all"`. Explicitne zadajte `processing="all"` z SDK, aby ste zrkadlili viacúrovňové ukladanie z CLI /GUI.

### Režimy snímania a záznamové zariadenia

Povrch poľa zrkadlí panel snímania v grafickom rozhraní: režimy Jednorazový / Nepretržitý / Intervalový / Najrýchlejší uzáver, plus dve záznamové zariadenia (kompozitné video v reálnom čase a surové snímky v sérii → offline spracovanie).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**je cyklus nepretržitého/intervalového snímania vSDK. Keďže neexistuje `Ctrl+C`, ktorý by ho prerušil zo skriptu,**musíte** odovzdať `count` a/alebo `duration_s` (zastaví sa, keď sa dosiahne niektorý z nich). `interval_s` sa meria od začiatku každého prechodu (zodpovedá grafickému rozhraniu). Zostávajúce kwargs prechádzajú priamo do `capture()`.
- **`record`** je určený na *monitorovanie*: zachytáva živý kompozitný index tak, ako sa zobrazuje, takže kombinovaný tok musí byť otvorený, aby sa mohli snímky ukladať. Jeden záznamník kompozitných údajov na pole (vyvolá výnimku, ak už jeden beží).
- **`burst` → `build_video`** je určený na *analýzu*: `burst` zapisuje surové snímky + manifest pre každý snímok + jeden `.daq` na každé odlišné čítanie DLS pod `<output>/bursts/<base>/` pri plnej rýchlosti snímacej slučky (bez reťazca, bez exiftool, bez živého náhľadu). `build_video` časovo zosúladí každý snímok s najbližším `.daq` a opätovne spustí reťazec radiancie/odrazivosti/indexu v importnom potrubí. `products` je zoznam `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (predvolené: kombinovaný index). `burst().stop()` tiež automatickyspustí kompiláciu kombinovaného indexu s maximálnym úsilím, ktorá sa vráti ako `build_job` vo výsledku zastavenia.

#### `RecorderHandle`

Vrátené funkciami `ArraySession.record()` a `ArraySession.burst()`. Použite ho ako správcu kontextu na automatické zastavenie pri opustení rozsahu alebo ho ovládajte manuálne.

| Člen | Popis |
| --- | --- |
| `job_id` | ID úlohy backendu (reťazec). |
| `kind` | `"composite"` (z `record`) alebo `"raw"` (z `burst`). |
| `start_stats` | Slovník vrátený volaním `start`. |
| `result` | `None` počas behu; konečný slovník výsledkov zastavenia po zastavení. |
| `stats(timeout=10.0)` | Štatistiky úlohy v reálnom čase (počet zapísaných snímok, dosiahnutá snímková frekvencia, uplynutý čas). |
| `stop(timeout=60.0)` | Zastaví záznam; vráti a uloží do vyrovnávacej pamäte konečný výsledok. Idempotentná funkcia (druhé volanie vráti výsledok uložený vo vyrovnávacej pamäti). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Pripojenie k už pripojenému poľu — `attach_array`

Ak je poľ už spustené (otvorilo ho grafické rozhranie, alebo ho vyvolala predchádzajúca relácia SDK volaním `connect_array`), použite `attach_array` na získanie rukoväte k nemu namiesto opätovného pripájania. `connect_array` v takejto situácii vždy vyhodí chybu „Kamera  <sn>je už v poli<id>“, pretože odoslanie príkazu `/array/connect` pre člena v poole nie je idempotentné; `attach_array` číta `/api/camera/array/list` a porovnáva buď podľa array_id, alebo podľa sériových čísel.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Vzor: SDK Skripty, ktoré sú súčasťou grafického používateľského rozhrania, by mali najskôr vyskúšať `attach_array` a ak v skupine ešte nie je žiadne pole, prejsť na `connect_array`.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Dôležité — ukončenie kontextového manažéra skutočne spôsobí odpojenie.**`ArraySession.disconnect()` vždy odošle POST na `/array/disconnect`; neexistuje žiadna ochrana typu „attached-not-owned“, ako je to v prípade `CameraSession` / `DAQSensorSession`. Ak spolupracujetes grafickým rozhraním a nechcete pri ukončení rozsahu rozobrať pole,**nepoužívajte blok `with`** — uchovajte referenciu v bežnej premennej a vynechajte explicitné `disconnect()`:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Pomocník pre analýzu siete

Užitočné pred otvorením poľa — odhadne, či vaše navrhované nastavenia budú vyhovovať:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` je jedným z `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (inak `error`). `auto_capped_fps` znamená, že požadované rozlíšenie vyhovuje prstencu RX iba pri obmedzenej frekvencii spúšťania — zachovajte rozlíšenie a prejdite z `target_fps=result["recommended"]["recommended_target_fps"]` na `connect_array` (pozri [Príklad 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Ako čítať projekciu** (rovnaký model ako v paneli Nastavenia poľa v grafickom rozhraní):

- **Séria snímok (`frame_bytes_total`) sa sčítava za každú kameru v reálnom formáte pixelov danej kamery.**Mono**M3M**kamery vysielajú Mono12 (2 B/px) bez ohľadu na to, akú hodnotu `pixel_format` zadáte, takže snímka v plnom rozlíšení zo 4 kamier má veľkosť**~25 MB** pri troch mono kamerách, nie ~12,6 MB, ako by vyplývalo z predpokladu, že všetky sú 8-bitové. Backend určí formát každej kamery na základe jej modelu.
- **Priepustnosť (`burst_fits_nic_ring`) zohľadňuje vyprázdňovanie**, nie porovnanie celého burstu s krúžkom: sim-emit sa uplatňuje, keď hostiteľ vyprázdňuje prijímací krúžok rýchlejšie, ako ho kamery napĺňajú. Hostiteľ 10G + kamery 1 GbE**prijímajú** plné rozlíšenie aj vtedy, keď burst presahuje kapacitu prstena; 1 GbE hostiteľ blokuje (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` predstavuje konzervatívny strop pre sériové načítanie** — `max(readout+emit, N×emit)` s vysielaním na kameru obmedzeným na 1 GbE kamerové prepojenie, nezávisle od expozície. Napr. ~2,8 fps pre 4-kamerové pole s plným rozlíšením a 12-bitovou hĺbkou (zodpovedá nameranej hodnote ~2,7–3,0 v runtime). Úplný model: [CLI Referencia → Model fps a burst poľa](cli-reference.md#array-fps--burst-model).
- **Prekročenie kapacity (`oversubscribed: true`) znamená, že minimálna hodnota N × na kameru prekračuje strop bezpečný voči kolíziám** — polia fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) majú hodnotu 0 a automatické zmenšovanie/zlučovanie to nedokáže vyriešiť (znižujú počet bajtov na snímku, nie počet bajtov za sekundu). Riešením je zníženie počtu kamier, použitie jumbo rámcov alebo rýchlejšia sieťová karta; kód `max_cams_collision_safe` hlási hornú hranicu (6 kamier s plným rozlíšením na 1 GbE pri 1500 MTU, 9 s jumbo rámcami). Odpoveď obsahuje aj kódy `aggregate_demand_bps`, `collision_safe_ceiling_bps` a `per_cam_floor_bps` (8 MB/s). Pozri [Prekročenie kapacity](#over-subscription-the-per-cam-floor).

### Zisťovanie a zoznam

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

Polia LATTICE spúšťajú nepretržité automatické zaostrovanie (AE) na pozadí hneď po pripojení, ale u novo nasmerovanej scény trvá chvíľu, kým sa zaostrenie ustáli. **Smart-capture** je praktická funkcia v balíku: skontroluje expozíciu každej kamery, počká, kým sa pole v rámci okna ustáli, a potom spustí snímanie. Je to ekvivalent funkcie v grafickom rozhraní: tlačidlo „smart“ v desktopovej aplikácii volá ten istý backendový endpoint.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Pri ovládaní cez `ChlorosProject` (nasledujúca časť) máte k dispozícii viac nastavení:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Politika smart-AE je štandardne konzervatívna. Pre náročnú radiometrickú prácu nastavte prísnejšie kritériá pomocou `exposure_tolerance_pct`; pre rýchlo sa meniace scény, kde vám stačí „dostatočná presnosť“, nastavte voľnejšie kritériá.

---

## Sesií senzorov DAQ

Trvalá skupina backendov pre spektrálne senzory (DAQ-U cez USB, DAQ-M cez BLE, DAQ-E cez Ethernet). Zrkadlí povrch kamery: inteligentná detekcia, opätovné použitie fondu, idempotentné pripojenie.

### Inteligentná detekcia (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Priorita: Ethernet → BLE → USB. Odovzdajte akýkoľvek explicitný pokyn na fixáciu prenosu.

### Fixovaný prenos

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### Metódy `DAQSensorSession`

| Metóda | Popis |
| --- | --- |
| `status(timeout=10.0)` | Súhrn položky v poole (stav streamovania/nahrávania, rozsah vlnových dĺžok, kalibračný SHA, integračný čas, frame_avg, stav AE). |
| `latest(n=1, timeout=10.0)` | Vráti až N najnovších rámcov spektra. |
| `stream_start()` / `stream_stop()` | Obnovenie / pozastavenie streamovania (identifikátor zostáva otvorený). |
| `record_start(output_dir=None, device_name=None)` | Spustí nahrávanie súboru .daq. Vráti cestu k súboru. Nefunguje pre DAQ-U/M bez kalibračného balíka AWS (DAQ-E je výnimkou). |
| `record_stop()` | Zastaví nahrávanie. Vráti `{path, rows}`. |
| `disconnect()` | Uvoľní z fondu. Žiadna operácia pre pripojenénevlastnených rukovätí. |

> **Profily korekcie kapacity (`cap_id`) nie sú ovládačom typu „SDK“.** `connect_daq_sensor()` / `DAQSensorSession` neexponujú žiadny parameter `cap_id` ani metódu `set_cap`. Vyberte profil korekcie limitu flotily prostredníctvom CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) alebo prostredníctvom ciest `/api/daq` HTTP v backende (`/api/daq/connect` a `/api/daq/<id>/cap-id` prijímajú `cap_id`).

### Vyhľadávanie — nájdenie adresy na pripojenie

`discover_daq_sensors()` vyhľadáva cez USB / BLE / ETH senzory, ktoré *by ste mohli* otvoriť. Je to ekvivalent `discover_lattice_cameras()` pre DAQ a jediný spôsob, ako získať **BLE MAC DAQ-M** — DAQ-E má názov hostiteľa a DAQ-U port COM, ale MAC nie je ani vyznačený na zariadení, ani uvedený v operačnom systéme.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Pole | Popis |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM port / BLE MAC / názov hostiteľa — odovzdať do `connect_daq_sensor` ako `port=` / `mac=` / `eth_host=`. |
| `display` | Čitateľný názov. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` alebo `None` pre port, ktorý skenovanie nedokáže identifikovať (sériové adaptéry USB nie je možné bez sondy rozlíšiť, preto sa neznáme položky zobrazujú, nie skryjú). |
| `extra` | Podrobnosti o jednotlivých typoch prenosu (inzerovaný názov BLE, výrobca USB, IP/fw/… zariadenia DAQ-E). Prázdne hodnoty sa vynechávajú. |

| Parameter | Predvolená hodnota | Popis |
| --- | --- | --- |
| `transports` | všetky tri | Sekvencia (alebo reťazec vo formáte CSV) obmedzujúca skenovanie. Stojí za to ju odovzdať, ak viete, čo chcete — BLE je pomalá časť. |
| `scan_timeout` | 5 | Okno skenovania pre jednotlivé transportné protokoly v sekundách; backend obmedzuje hodnotu na 1–20. |
| `timeout` | 60,0 | Maximálna hodnota pre „HTTP“ pre celé volanie (rovnako ako inde v súbore „SDK“). |
| `auto_start_backend` | `True` | Spustí lokálny backend, ak žiadny nebeží. Nikdy sa nespustí pre vzdialený `backend_url`. |

> **Senzory, ktoré sú už otvorené v poole, sa nezobrazujú.** Pripojené periférne zariadenie BLE prestane vysielať a otvorený port COM nemôženemôže byť skenovaný, takže vyhľadávanie zobrazuje len to, čo je *dostupné na pripojenie*. Hneď po pripojení zariadenia je očakávaný prázdny výsledok — pre zariadenia, ktoré už máte pripojené, použite `list_daq_sensors()`. Transporty, ktorých skenovanie nemôže prebehnúť (nie je nainštalovaný bleak / zeroconf), sa preskočia namiesto vyvolania chyby, takže zariadenie bez Bluetooth stále dostáva odpovede pre USB a ETH.

### Zoznam

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Spolupráca s GUI / CLI

Ak má GUI už otvorený senzor, volanie `connect_daq_sensor(port="COM3")` z Python vráti identifikátor označený ako `already_connected=True`. `disconnect()` danej relácie je potom bez-op, takže váš skript SDK neodpojí senzor od grafického rozhrania pri ukončení programu.

### Triedy priameho hardvéru (bez backendu)

`daq_sdk` je opätovne exportovaný prostredníctvom `chloros_sdk`, takže senzory môžete ovládať aj end-to-end priamo v procese bez backendu:

> **Dostupnosť:**`daq_sdk` je súčasťou inštalácie pre stolné počítače z Chloros,**nie** s balíkom PyPI — `pip install chloros-sdk` vám poskytuje `lattice_sdk`, ale vynecháva `chloros_sdk.DAQ_AVAILABLE == False`. Pred použitím týchto tried skontrolujte tento príznak; pri inštalácii cez pipovládajte senzor pomocou [`connect_daq_sensor()`](#daq-sensor-sessions), ktorý nevyžaduje žiadne lokálne transportné knižnice.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Ak chcete zdieľané vlastníctvo so grafickým rozhraním, uprednostnite cestu smart-connect (`connect_daq_sensor`); pri skriptoch bez grafického rozhrania, ktoré majú senzor výhradne vo vlastníctve, použite priame triedy.

---

## Automatizácia projektu — `ChlorosProject`

Uložený projekt typu „Chloros“ je priečinok obsahujúci súbory `cameras.json` + `sensors.json` + `project.json`. `open_project` načíta manifest a `connect_all` pripojí každé uložené zariadenie online s jeho uloženými nastaveniami – v rovnakom stave hardvéru, aký by vytvorilo grafické používateľské rozhranie.

### Minimálny príklad

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Alebo ako správca kontextu:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Metódy `ChlorosProject`

| Metóda | Popis |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Zistí a pripojí každé uložené zariadenie. Vráti správu o pripojení pre každútriedy. Používa spustený backend, ak nejaký počúva na `127.0.0.1:5000`; inak sa bez upozornenia vráti k priamemu (bez backendu) `lattice_sdk` ovládaniu zariadení — nikdy nespustí backend. |
| `disconnect_all()` | Ukončí všetko. |
| `capture_all(output_dir=".")` | Jeden snímok z každej kamery + pole + spektrum z každého senzora. |
| `stream(camera, overlays=False, fps=10.0)` | Generátor poskytujúci snímky BGR `numpy` z pomenovanej kamery (alebo poľa). `overlays=False` je priama `lattice_sdk` slučka na zachytávanie (polia generujú `{serial: frame}` slovníky). `overlays=True` smeruje cez `ChlorosLocal.camera_stream()` → MJPEG kanál `/api/camera/<serial>/stream-annotated` backendu, pričomuložený blok kamery sa odovzdáva ako parametre dotazu. Vyžaduje režim backendu a **samostatnú kameru**: kamera v priamom režime vyvolá `RuntimeError` (backend nemôže uchopiť kameru, ktorú tento proces vlastní) a pole vyvolá `NotImplementedError` (prekrýva kompozitný obraz podľa kamery — streamuje člena podľa mena). Ekvivalent jednorazového spustenia: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Spustiť zarovnanie na každom aktuálne pripojenom poli. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Spustiť kalibračný / indexovací proces na obrázkoch projektuobrazov (zahŕňa `ChlorosLocal.process`; tieto štyri sú **jediné** prijímané kľúčové argumenty — `indices=` atď. vyvolajú výnimku `TypeError`; nastavte indexy prostredníctvom `ChlorosLocal.configure()`). Lenošne vytvorí `ChlorosLocal()`, ktorý automaticky spustí backend. |

Atribúty:
- `proj.cameras` — `Dict[str, CameraHandle]` s kľúčom podľa názvu A sériového čísla.
- `proj.arrays` — `Dict[str, ArrayHandle]` s kľúčom podľa názvu A array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` s kľúčom podľa názvu A slot_id.
- `proj.config` — `project.json["config"]` slovník.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Úrovne spracovania.** `capture()`, `grab()`, a `frame_stream()` všetky prijímajú rovnaký token `processing`
a reťazec je kumulatívny — každá úroveň spúšťa všetko, čo je nad ňou:

| Úroveň | Výstup | Poznámky |
| --- | --- | --- |
| `raw` | 1-kanálový Bayer, natívny pre senzor | Bez demosaikácie. Prekrytia nie sú na tejto úrovni k dispozícii. |
| `debayered` | 3-kanálový BGR (**predvolené**) | Bilineárna demosaika. Jediná úroveň, ktorá funguje bez režimu backend. |
| `radiance` | float32, W/m²/sr/nm | Úplný radiometrický reťazec: demosaic + 3×3 unmix (multispec) + DSNU + flat-field + NIST scale, s odpočítaním expozície × zosilnenia, takže hodnoty sú absolútne. |
| `reflectance` | uint16, 32768 = 1,0 | Žiarivosť vydelená dopadajúcou intenzitou žiarenia (ρ = π·L/E). Vyžaduje sa hodnota z DLS/DAQ — pozri poznámku nižšie. |
| `display` | 8-bitový sRGB-podobný | Vykreslenie zodpovedajúce GUI: CCM + vyváženie bielej + gama prostredníctvom aktívneho farebného profilu kamery. |

Všetko okrem `debayered` vyžaduje režim backendu; kamera v priamom režime vyvolá
`NotImplementedError`. `reflectance` vyžaduje použiteľnú hodnotu dopadajúceho žiarenia — koncový bod snímky automaticky vtiahne
zhromaždené DAQ do slotu DLS kamery, ale ak nie je viazané žiadne DAQ, reťaz odmietne
výstup odrazivosti a úprimne označí zníženie kvality vo vrátených metadátach, namiesto toho, aby ticho
vrátila produkt nižšej kvality.

> **Škála DN odrazivosti — neprogramujte ju pevne.** Odrazivosť LATTICE používa `32768` = ρ 1.0 a označuje
> XMP `Chloros:PixelScale=32768`; odrazivosť Survey3 používa `65535` = ρ 1,0 a neobsahuje žiadne
> `Chloros:*` značky. Prečítajte značku a vydelte ňou. Je definovaná v doméne uint16, takže zostáva
> `32768` pre každý formát, ktorý mení mierku (16-bitové TIFF, 8-bitové PNG /JPG, 32-bitové percent) — najprv normalizujte
> uložený dátový typ späť na uint16 (×257 z 8-bitového, ×65535 z plávajúcej desatinnej čiarky). Jediná výnimka:
> záznam z 8-bitového zdroja zapísaný ako 8-bitový TIFF je *orezaný*, nie zmenšený, takže ho žiadne mierka nepopisuje
> — Chloros v takom prípade úplne vynecháva `PixelScale` a tupel MicaSense. Chýbajúcu
> značku v súbore odrazivosti LATTICE považujte za „žiadne platné mierky“, nie za predvolenú hodnotu.

> **EXIF prenesené do exportu.** `process()` skopíruje blok GPS zo zdrojového záznamu
> **a jeho ExifIFD** do každého produktu, takže exporty obsahujú `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` a `CameraSerialNumber`, ako aj
> georeferenciu. Na základe kódu `FocalLength` program Pix4D vypočíta vzdialenosť medzi bodmi na zemi – bez neho
> sa rekonštrukcia vráti k úplne nesprávnemu meradlu (v jednom nameranom prípade sa lokalita s rozlohou 411 m
> zmenila na lokalitu s rozlohou 47,8 km). Kópia zámerne nie je `-all:all`: štrukturálne tagy IFD0 narúšajú
> výstup LATTICE a súbory `ExifImageWidth`/`Height` sú vylúčené, pretože opisujú zdrojové
> snímanie, a nie exportovaný raster.

Podvlajky fázy snímania (platia pre radiometrické úrovne — `radiance`, `reflectance`, `display`):

| Priepínač | Predvolené | Význam |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + 3x3 unmix + radiometrická stupnica NIST. |
| `apply_white_balance` | `True` | WB LUT. DLS- zohľadňuje, keď je DAQ pripojený k kamere. |
| `apply_index` | `False` | Vyhodnotenie vegetačného indexu. |
| `index_expression` | `None` | Prepis vzorca. Ak nie je prázdny → automaticky aktivuje index. |
| `annotated` | `False` | Prekrytie dekorácií grafického rozhrania (zebra/mriežka/vrcholy). Nedostupné pre `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Typ návratovej hodnoty je `CapturePathMap`, nie `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` je `Dict[str, Union[str, List[str]]]`: jednovrstvový
> `processing` priraďuje každému sériovému číslu jednu cestu, zatiaľ čo viacvrstvovýúrovňová (`"all"` alebo
> explicitný zoznam `levels`) mu poskytuje **usporiadaný zoznam** všetkých produktov uložených pre danú
> kameru. Kombinovaný kompozit v reálnom čase, ak sa nejaký streamuje, prichádza pod dodatočným
> kľúčom `"combined"`, a nie pod sériovým číslom. Kód, ktorý predpokladá `str`, sa pokazí pri
> zozname bez toho, aby proti tomu namietal akýkoľvek typový kontrolór — poznámka uvádzala `Dict[str, str]`
> ešte nejaký čas po vydaní zoznamového formátu, preto tento alias existuje. Normalizujte
> v prípade, že chcete plochý formát:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Zarovnanie poľa

`ArrayHandle` sprístupňuje celú plochu zarovnania. Profily sú štandardne platné len pre danú reláciu — na ich trvalé uloženie explicitne volajte `export_alignment()`.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Zarovnanie pri pripojení

`connect_all(align=...)` dokáže automaticky zarovnať každé pole pri pripojení:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Ak nie je špecifikované, použije sa ako náhrada `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Priamy hardvér (bez backendu)

Ak chcete nulovú závislosť na backende (CI, bezhlavé roboty, vstavané systémy), importujte priamo `lattice_sdk` a `daq_sdk` — obe sú opätovne exportované prostredníctvom `chloros_sdk`. Ochrana na `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` je súčasťou balíka PyPI (vyžaduje však prítomnosť runtime prostredia Arena SDK), zatiaľ čo `daq_sdk` je súčasťou iba inštalácie pre stolné počítače.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Prednastavenia a spúšťač

Tri zo štyroch prednastavení **free-run**: kamera nepretržite exponuje a
`capture()` vráti ďalší snímok. Výnimkou je `triggered` — táto prednastavená hodnota aktivuje
kameru pre hardvérový signál na linke 2, takže nezaznamená nič, kým takýto signál nepríde.

| Prednastavená hodnota| Spúšť | Použite, keď |
| --- | --- | --- |
| `default` | voľný chod | všeobecné použitie |
| `high_speed` | voľný chod | 8-bitové, obmedzenie na 60 fps, krátka expozícia |
| `high_quality` | voľný chod | 12-bitové, bez obmedzenia fps — bežná voľba pre fotografie |
| `triggered` | **aktivovaný, linka 2** | fotoaparát je pripojený k synchronizačnému káblu M8 a spúšťa ho niečo iné |

Ak zvolíte `triggered` (alebo si sami nastavíte `trigger_mode="On"`) bez toho, aby
niečo riadilo linku 2, každý `capture()` vyprší – správne, keďže ste
požiadali kameru, aby čakala. Chyba „SDK“ to vysvetľuje, keď k tomu dôjde; pozri
[SC_ERR_TIMEOUT počas snímania](#direct-hardware-backend-free).

> **Poznámka — Správy „GVSP probe“ / `SC_ERR_TIMEOUT -1011` pri pripojení nie sú chybami.**&gt; Pri pripájaní sa rozhranie SDK pokúša dohodnúť**jumbo rámce** (9000-bajtové pakety GVSP) pre vyššiu priepustnosť. Na priamom spojení sieťovej karty typu point-to-point (napr. adresa `169.254.x.x` platná len pre dané pripojenie) sieť zvyčajne nedokáže prenášať jumbo rámce, takže táto kontrola vyprší a do protokolu sa zapíšu riadky ako:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Toto je **predpokladaná záložná možnosť**: modul SDK sa automaticky vráti k štandardným 1500-bajtovým paketom a kamera pokračuje v pripájaní normálne (nasledujúce riadky typu `[chunk-enable …]` sú súčasťou bežnej sekvencie pripájania). Zachytávanie stále funguje.
>
> Túto kontrolu môžete preskočiť, ale **nejde len o potlačenie záznamov v protokole — vypína jumbo rámce.** Kamera odpovedá na pingy s príkazom „Don&#x27;t-Fragment“ len do veľkosti 1500 bajtov bez ohľadu na to, aká dobrá je vaša sieť, takže samotný ping test nikdy nemôže zistiť prítomnosť jumbo rámcov; táto kontrola je jediná, ktorá to dokáže. Ak ju deaktivujete, kamera bude navždy používať štandardné 1500-bajtové pakety v akejkoľvek sieti:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Oplatí sa to len v sieti, o ktorej *viete*, že nepodporuje jumbo rámce, kde to ušetrí približne jednu sekundu času pripájania na jednu kameru. Keďže ide o skutočný kompromis a nie len o kozmetickú zmenu, správa „SDK“ to teraz pri použití uvádza:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Nechajte to tak, pokiaľ nemáte dôvod.** Ak to necháte zapnuté, pri každom pripojení sa znovu zmeria vaša aktuálna sieť: pripojte sa k prepínaču podporujúcemu jumbo pakety a pri ďalšom pripojení sa jumbo pakety aktivujú automaticky, bez akejkoľvek konfigurácie a bez reštartu.
>
> Ak *chcete* priepustnosť jumbo, povolte jumbo end-to-end (MTU sieťovej karty 9000 + prepínač, ktorý ich prenáša), alebo ju zafixujte pomocou `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, ak viete, že spojenie ju prenáša — hoci je lepšie použiť `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` pre jednotlivé príkazy ako trvalé nastavenie, pretože pevne nastavená veľkosť preskočí testovanie a zabráni prispôsobovaniu sa sieti pred ňou. **Každé** zariadenie na trase musí prenášať jumbo pakety — vrátane akéhokoľvek PoE rozdeľovača alebo injektora, čo je bežný dôvod, prečo inak jumbo-kompatibilná konfigurácia nedokáže prenášať.

> **`SC_ERR_TIMEOUT -1011` počas `capture()` / `grab*()` je iný problém — ten je skutočnou chybou.**&gt; Vyššie uvedená poznámka sa týka iba chyby `-1011` zaznamenanej sondou**connect-time**. Ak sa tá istá chyba vyskytne pri**zachytávaní**, znamená to, že kamera sa pripojila v poriadku, ale neposiela žiadne snímky:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Rozhodujúcim znakom je kamera, ktorej *ovládací* kanál funguje správne – vyhľadávanie prebieha bez problémov, nastavenia a zápisy `[chunk-enable …]` sú úspešné –, zatiaľ čo *každý* snímok vyprší.
>
> **Bežnou príčinou je, že kamera je nastavená na hardvérové spúšťanie.** Pri chybách `trigger_mode="On"` a `trigger_source="Line2"` kamera nevysiela vôbec nič, kým na synchronizačnom kábli M8 nedôjde k elektrickej zmene stavu. Ak nemáte kábel, ktorý by túto linku riadil, každé zachytenie snímky čaká donekonečna. Kamera nie je pokazená a sieť funguje správne – robí presne to, čo má.
>
> Prednastavenia `CameraSettings()` a `default` / `high_speed` / `high_quality` umožňujú voľný chod a zachytenie, ktoré vyprší počas aktivácii, sa vysvetľuje sám, namiesto toho, aby sa zobrazilo len `-1011`. `PRESETS["triggered"]` aktivuje Line2, podľa návrhu.
>
> Ak chcete vynútiť voľný chod akejkoľvek kamery:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Ak aj pri použití `trigger_mode="Off"` stále dochádza k časovému limitu, kamera skutočne neposiela údaje — pošlite nám protokol a `ip link show`.

#### Farebné profily (živý náhľad RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` vyberá farebný profil zobrazenia pre **živý náhľad** na kamerách typu „RGB“ (kamery typu „multispec“ toto nastavenie ignorujú):

| Profil | Význam |
| --- | --- |
| `raw` | Úplné obídenie radiometrického reťazca. |
| `linear` | DSNU + flat + WB, bez CCM, bez gama. |
| `natural` | Lineárne + namerané CCM + gama sRGB, iba s lacnou úpravou (vyhladenie chrominancie + desaturácia svetiel) — realistické predvolené nastavenie. |
| `enhanced` | `natural` plus kompletná úprava s paritou (odstránenie farebných okrajov, živost, lokálny kontrast CLAHE). Bohatší vzhľad pri približne **dvojnásobných nákladoch na spracovanie každého snímku**, teda nižšia snímková frekvencia v režime LIVE. |
| `custom_temp` | `natural`, ale vyváženie bielej (WB) je zafixované na `custom_cct_k` Kelvin (DLS ignorované; obmedzené na 2000–10000 K na strane backendu). |

Tento profil slúži **len na živý náhľad** ako ovládač rýchlosti/vzhľadu: uložené snímky vždy získajú plnú bohatú úpravu bez ohľadu na vybraný profil, takže výber `natural` s cieľom získať späť čas snímky neznižuje kvalitu toho, čo sa uloží na disk. Neznámy profil zvýši hodnotu `ValueError`; keď je dostupný backend chloros, zmena sa odošle aj do neho, takže nasledujúci náhľadový snímok ju zohľadní (užívatelia s priamym SDK bez backendu stále dostanú zmenu nastavení).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Mono (M3M) kamery a `Calibration`

Mono kamera **M3M** (`M3M-<lens>-F<wavelength>`) je jednopásmová: jedna rovina v odtieňoch šedej, bez Bayerovej mozaiky, bez spektrálnej matice 3×3. `Calibration` ju rozpozná a sprístupní príznak `is_mono`. Odrazivosť sa naďalej uplatňuje ako radiometrická mapa pre každé pásmo (demix je identitná matica), ale viacpásmové výpočty na jednej kamere skôr zvyšujú hodnotu, než aby vracali nezmysel:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Na vytvorenie vegetačného indexu z monokromatického hardvéru skombinujte niekoľko kamier M3M s rôznymi vlnovými dĺžkami do zarovnaného viacpásmového súboru (pozri [Zarovnanie poľa](#array-alignment)) a vypočítajte index pre celý tento súbor namiesto jednej kamery.

Priamy režim DAQ:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **`apply_sensor_settings` akceptované kľúče**— presne `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; nahradené kľúčmi `cap_id`), `filter_model` (DAQ-M) a `cap_id` (všetky typy DAQ; `None`/`""`/`"none"` = holý senzor, bez korekcie krytu). Neznáme kľúče sú**ticho ignorované** — napr. `{"integration_time": 64}` nevykoná nič (musí to byť `integration_time_ms`). Vráti `{"applied": [...], "errors": {...}}` a nikdy nevyvolá výnimku.

`chloros_sdk` opätovne exportuje iba základnú plochu použitú vyššie. Úplný verejný API `daq_sdk` (22 mien) pridáva nasledujúce položky — importujte ich priamo z `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Výnimky

Zachyťte základnú triedu na spracovanie „všetkého, čo sa pokazilo v Chloros“:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` a `ChlorosConfigurationError` sú exportované na najvyššej úrovni spolu s ostatnými; ako je znázornené, je možné ich importovať aj z `chloros_sdk.exceptions`.

Hierarchia:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Príklady od začiatku do konca

### 1. Spracovanie priečinka s vlastným ukazovateľom priebehu

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Pole LATTICE v reálnom čase → Odrazivosť + referenčné údaje DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Zber údajov riadený projektom

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Prúd snímok z viacerých kamier → NumPy pipeline

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Skript na snímanie bez grafického rozhrania priamo z hardvéru (bez backendu)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Testovanie schopností pred pripojením 4-kamery

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Ekvivalent receptu na snímanie (čistý Python)

Receptový DSL v CLI má priamy ekvivalent v Python:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Automatické spustenie backendu

Vstupné body smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor` a `discover_lattice_cameras` — sú tenké klienti typu „HTTP“, ktorí predpokladajú, že backend načúva na `127.0.0.1:5000` (predvolený URL rozhrania Smart-Connect). Ak je už spustené grafické rozhranie (GUI) alebo webová aplikácia (CLI), jeden z nich už beží. Pri spustení z holého skriptu však nemusí byť spustený žiadny — preto tieto funkcie **automaticky spustia priložený binárny súbor backendu** (bez okna, rovnako ako to robí `ChlorosLocal`) pred ich prvým volaním a potom čakajú až do `backend_startup_timeout`, kým sa spustí.

Pravidlá:

- **Spúšťa sa vždy len lokálny URL.** `backend_url` smerujúci na `localhost` / `127.0.0.1` / `[::1]` je oprávnený; akýkoľvek iný hostiteľ sa považuje za stroj niekoho iného a nikdy sa nespustí.
- **Backend zostáva spustený na opätovné použitie** (rovnako ako v prípade CLI) – pri ukončení skriptu nedochádza k implicitnému vypnutiu. Opätovné spustenie skriptu znovu použije aktívny backend.
- **Vypnutie je možné pomocou `auto_start_backend=False`** pri ktoromkoľvek z týchto volaní (napr. ak ste nasmerovali na vzdialený backend alebo si životný cyklus backendu spravujete sami).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Ak sa nedá nájsť alebo spustiť priložený binárny súbor, nasledujúce volanie HTTP vyvolá akčné, **platformovo špecifické** upozornenie `ChlorosConnectError` namiesto holého záznamu o odmietnutí pripojenia — na Windows vás nasmeruje na desktopovú aplikáciu alebo príkaz `chloros-cli`; na Linux (bez grafického rozhrania) vás nasmeruje na príkaz `chloros-cli` alebo na `.deb`.

---

## Prostredie a hlavičky

SDK označuje každé volanie backendu HTTP pomocou `X-Chloros-Client: sdk`. Backend uplatňuje licenčné pravidlá SDK / CLI (vyžaduje sa prihlásenie **a** platený plán Chloros+), na rozdiel od bezplatnej verzie s grafickým rozhraním. Toto sa nastaví automaticky pri importe — nemusíte robiť nič.

`http://localhost` a `http://127.0.0.1` sú detekované ako lokálny backend. Volania na iné hostiteľa (napr. vaša vlastná analytická služba) zostávajú nezmenené.

Prepisujte backend URL zadaním `backend_url=` (alebo `api_url=` na `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(`backend_url` bez loopbacku dosiahne iba backend source/dev — dodávané backendy viažu iba loopback; pozrite si Režim vzdialeného backendu pre vzor tunela.)

---

## Verzie a kompatibilita

- Verzia „SDK“ je dostupná ako `chloros_sdk.__version__`.
- Verzia „SDK“ viaže správanie na verziu dodávaného backendu. Kombinácia staršieho „SDK“ s novším backendom zvyčajne funguje (koncové body sú kompatibilné s novšími verziami), avšak kombinácia novšieho SDK so starším backendom môže spôsobiť chyby typu `404` na nových koncových bodoch — aktualizujte desktopovú aplikáciu tak, aby bola zosúladená.
- Rozhranie Smart-Connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) a koncový bod analýzy siete vracajú stabilné schémy JSON; nové polia sú doplnkové.

---

## Tipy na riešenie problémov

- **`ChlorosAuthenticationError: Login required`** → Spustite na tomto počítači raz skript `chloros-cli login EMAIL PASSWORD` alebo sa prihláste prostredníctvom desktopovej aplikácie Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → Funkcia Smart-Connect automaticky spúšťa lokálny backend, takže táto chyba sa objaví len vtedy, ak sa nedá nájsť alebo spustiť priložený binárny súbor (napr. na hostiteľskom počítači, kde je nainštalovaný len pip a chýba balík pre pracovnú plochu). Správa závisí od platformy: na Windows otvorte desktopovú aplikáciu alebo spustite ľubovoľný príkaz `chloros-cli`; na Linux spustite príkaz `chloros-cli` (neexistuje grafické rozhranie) alebo nainštalujte `.deb`. V prípade vzdialeného backendu zadajte `backend_url=` (a `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** pri importe → Načítanie `lattice_sdk` zlyhalo (zvyčajne nie sú nainštalované runtime DLL súbory Arena SDK). Povrch bez kamery stále funguje.
- **Funkcia Array connect vráti rozlíšenie nižšie ako natívne**→ Funkcia smart-prep na strane backendu automaticky zmenšuje veľkosť snímky, aby sa zmestila do kanála. Použite `analyze_array_network()`, aby ste zistili dôvod, a potom buď vylepšite prepojenie, akceptujte zmenšenie alebo odovzdajte `force_tier="slip-emit-and-capture"` pre sekvenčné snímanie. Bezpečnostný mechanizmus proti zmenšeniu**nezahŕňa** agregované prekročenie kapacity (`oversubscribed: true`, polia fps 0): príliš veľa kamier pre linku nemožno vyriešiť zlučovaním snímok/ROI — znížte počet kamier, aktivujte jumbo rámce alebo prejdite na rýchlejšiu sieťovú kartu (pozri [Prekročenie kapacity](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` hlási, že prijímací krúžok sieťovej karty je veľmi malý (~0,26 MB) / pripojovacie brány s hlásením „FRAMES WILL DROP“** → Príjmový prstenec sieťovej karty hostiteľa je nastavený na predvolenú hodnotu (často sa po aktualizácii ovládača sieťovej karty resetuje na 32). Na adaptéri Realtek USB 10GbE nastavte `ReceiveBufferLen=256` a `PendingReceives=64` (s zvýšenými oprávneniami), potom reštartujte systém, aby znovu prečítal kruh. Úplný postup: [CLI Referencia → Nastavenie a ladenie hostiteľskej sieťovej karty](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Hostiteľ sa zavesí pri reštarte/vypnutia, neskôr chyby WMI `Invalid class` / sieťová karta sa nedá aktivovať** → Zastaraný ovládač USB 10GbE spôsobuje chybu `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Aktualizujte ovládač adaptéra na aktuálnu verziu (≥ 2026) a znovu aplikujte nastavenia prijímacieho prstenca. Pozrite si [Referenčné informácie k CLI → Nastavenie a ladenie sieťovej karty hostiteľa](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Odrazivosť odmietnutá** → Pre odrazivosť v absolútnej stupnici musí byť k kamere (alebo sústave) priradený aktívny DAQ. Priraďte ho buď prostredníctvom grafického rozhrania, alebo použite `processing="radiance"` (W/m²/sr/nm) , ktorý nevyžaduje spárovaný senzor.
- **Zaznamenávanie pomocou `smart=True` trvá dlhšie, ako sa očakávalo** → Konvergencia AE závisí od dynamiky scény; ak chcete rýchlejšie (menej stabilné) spustenie.

---

## Pozri tiež

- [Referenčná príručka CLI](cli-reference.md) — každý podpríkaz CLI zodpovedá volaniu SDK.
- [Príručka k senzorom DAQ](../daq/README.md) — pravidlá zapojenia, kalibrácie a záznamu špecifické pre jednotlivé senzory.
- Online dokumentácia: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>

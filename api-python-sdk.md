# API : Python SDK

{% hint style="info" %}
**Hľadáte kompletný API?** Táto stránka je praktickým návodom. Všetky verejné triedy, metódy, presné podpisy a príklady, ktoré je možné skopírovať a vložiť, sa nachádzajú v [Referencii SDK](reference/sdk-reference.md), ktorá je optimalizovaná pre AI asistentov.**Pracujete s AI asistentom?** Vložte tento text URL do chatu, aby mal k dispozícii úplnú a aktuálnu verziu Chloros 1.2.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Každá stránka tejto príručky je dostupná ako surový markdown pod názvom v malých písmenách + `.md` a celá príručka je indexovaná na `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` na PyPI) riadi všetko, čo dokáže desktopová aplikácia od Python: hromadné spracovanie obrázkov, ovládanie kamery LATTICE a polia v reálnom čase, relácie DAQ so svetelnými senzormi a automatizáciu uložených projektov. Ide o tenkú vrstvu nad tým istým lokálnym backendom, ktorý používajú grafické rozhranie (GUI) a CLI (HTTP na `127.0.0.1:5000`), takže správanie je identické vo všetkých troch prostrediach.

## Inštalácia

Inštalácia prebieha v dvoch krokoch: najskôr nainštalujte balík Chloros pre pracovnú plochu (poskytuje backend na spracovanie a hardvérové runtime prostredia), potom balík Python.

**Krok 1 — Inštalácia balíka Chloros.** Windows: spustite inštalačný program pre stolné počítače (predvolená cesta `C:\Program Files\MAPIR\Chloros\`) zo stránky [Stiahnuť](download.md). Linux: nainštalujte balík `.deb` ([Inštalácia Linux](linux/linux-installation.md)).**Krok 2 — Nainštalujte SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Možno ani nebudete potrebovať pip: každý inštalátor obsahuje zodpovedajúce koleso SDK. Inštalátor Windows ho automaticky nainštaluje do vášho systému Python; inštalátor Linux `.deb` ho umiestni do adresára `/usr/lib/chloros/sdk/` a vypíše presný príkaz `pip install --user`. PyPI sa aktualizuje pri vydaní nových verzií, takže `pip install chloros-sdk` zodpovedá najnovšej stabilnej verzii.

**Krok 3 — Prihláste sa raz na každom počítači:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Prihlasovacie údaje sú uložené v pamäti cache v `~/.chloros/` (na oboch platformách). V Windows sa môžete rovnako prihlásiť cez kartu „User“ (Používateľ)<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> v aplikácii pre počítače. SDK vyžaduje platený plán Chloros+ — pozri [Požiadavky na licenciu](#license-requirement) nižšie.

| Požiadavka | Podrobnosti |
| --- | --- |
| **Chloros nainštalovaný** | Windows: inštalačný program pre počítač; Linux: balík `.deb` (poskytuje binárny súbor backendu) |
| **Python** | 3.7 alebo vyššia (vyvinuté/testované na verzii 3.10) |
| **Operačný systém** | Windows 10/11 64-bit, Ubuntu 22.04 LTS alebo novší, alebo NVIDIA Jetson (JetPack 6) |
| **Licencia** | Aktívne prihlásenie do Chloros+, akákoľvek platená úroveň (Copper alebo vyššia) |

## Úspech za 60 sekúnd

Jedno volanie vytvorí projekt, načíta priečinok, nakonfiguruje spracovanie a spustí pipeline — pričom automaticky spustí backend, ak ešte nebeží:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(V Linux použite cesty Linux: `/home/user/drone_images/flight001`. SDK funguje identicky na oboch platformách.)

Spracovávate zložku so záznamami z LATTICE? Použite wrapper prispôsobený pre LATTICE — uplatňuje správne predvolené nastavenia (bez detekcie cieľa panelu, štandardný debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — plná kontrola nad spracovateľským reťazcom

Pre akékoľvek úlohy, ktoré presahujú rámec jedného riadku, použite `ChlorosLocal`. Pri prvom použití spustí backend (`auto_start_backend=True`), vytvorí a nakonfiguruje projekty, monitoruje priebeh a po dokončení vráti súhrn.

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

{% hint style="info" %}
Používajte predvolený príkaz `http://127.0.0.1:5000` namiesto nahradenia príkazom `localhost` — v prípade Windows `localhost` sa najprv vyrieši na `::1` a pri backende podporujúcom iba IPv4 trvá spracovanie jednej požiadavky približne 2 sekundy.
{% endhint %}

Použite ho ako správcu kontextu pre zaručené uvoľnenie pamäte:

```python
import chloros_sdk

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

`configure()` akceptuje tieto kľúčové slová: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` a `custom_settings`. Hlavné hodnoty:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Ovládacie prvky špecifické pre LATTICE (`input_level`, `radiometric_output`, rodina `array_alignment*`) sú zdokumentované spolu s tabuľkami všetkých ich hodnôt v [Referencii SDK](reference/sdk-reference.md#supported-values).

### Sledovanie priebehu

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Čítanie súhrnu po spustení — a zachytávanie prázdnych behov

Po dokončení `process()` pripojí súhrn spracovania backendu ako `result["summary"]`. Každý záznam v `summary["hints"]` je úplná veta vysvetľujúca čokoľvek pozoruhodné — napríklad, prečo beh nevytvoril žiadny výstup — a každý tip sa tiež opätovne odošle ako Python `UserWarning`, takže prázdne behy sa diagnostikujú samy, aj keď slovník nikdy nekontrolujete:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` sa nevygeneruje, ak beh nevytvorí žiadne obrázky.** Toto je jediné miesto, kde sa SDK a CLI zámerne líšia: `chloros-cli process` považuje situáciu „boli požiadané výstupy, žiadne neboli zapísané“ za zlyhanie a ukončí sa s nenulovým kódom, zatiaľ čo SDK sa vráti normálne a túto situáciu nahlási prostredníctvom `summary` / hints. Ak by sa váš pipeline mal zastaviť pri prázdnom behu, skontrolujte to sami — skontrolujte skript `summary` (alebo spočítajte súbory v projektovej zložke) namiesto toho, aby ste sa spoliehali na výnimku.
{% endhint %}

## Smart Connect — aktívny hardvér

Tri pomocné skripty otvárajú trvalé relácie vo hardvérovom poole backendu – v tom istom poole, ktorý používa grafické používateľské rozhranie, takže skripty SDK koexistujú s desktopovou aplikáciou bez toho, aby si navzájom prekážali v používaní sériových portov alebo sieťovej šírky pásma. Všetky tri automaticky spustia lokálny backend, ak žiadny nebeží.

### Jedna kamera LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Synchronizované pole — `connect_array`

`connect_array` je odporúčaným východiskovým bodom pre zostavy s viacerými kamerami. Spúšťa rovnaký proces inteligentnej prípravy ako grafické používateľské rozhranie: analýzu siete, automatický výber synchronizačnej úrovne, časovú synchronizáciu PTP, výber formátu pixelov pre každú kameru, inicializáciu AE a aktiváciu spúšťača GPIO. **Prvá séria je hlavná** (vysiela hardvérový spúšťací impulz); ostatné sú podriadené.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Pridajte `smart=True` k akémukoľvek snímaniu poľa, aby ste pred spustením počkali na ustálenie automatickej expozície vo všetkých kamerách. Informácie o režimoch snímania (jednotlivé / sériové / intervalové / najrýchlejšie), záznamníkoch, burst-to-video a zarovnaní poľa nájdete v [Referencii SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Svetelný senzor DAQ — `connect_daq_sensor`

Bez argumentov funkcia `connect_daq_sensor()` automaticky detekuje prenosový protokol (v poradí: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Každý rámec obsahuje hodnotu 135 bodov `spectrum` (W/m²/nm po kalibrácii), príznak `is_saturated` a CIE `x`, `y`, `z`. Ak chcete priradiť konkrétny senzor alebo prenosový protokol — čo je spoľahlivá voľba na hostiteľských zariadeniach s viacerými sieťovými rozhraniami, kde automatické vyhľadávanie cez Ethernet môže pri prvom pokuse prehliadnuť funkčný DAQ-E — zadajte jeden explicitný údaj:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Upozorňujeme, že profily korekcie stropu (`cap_id`) **nie sú** nastavovacím prvkom typu SDK — namiesto toho ich vyberte cez `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Uložené projekty — `open_project`

Uložený projekt Chloros si zachováva pripojený hardvér (`cameras.json` + `sensors.json` spolu s `project.json`), a `chloros_sdk.open_project(path)` dokáže všetko znovu pripojiť naraz a riadiť zachytávanie podľa názvu zariadenia. Pozrite si [Automatizácia projektov](reference/sdk-reference.md#project-automation--chlorosproject) v referenčnej príručke.

## Čo získate pri inštalácii iba prostredníctvom pip

Pred použitím hardvérových plôch skontrolujte príznaky dostupnosti na úrovni modulov:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Na hostiteľskom počítači s **len** balíkom `pip install chloros-sdk` a bez balíka Chloros pre pracovnú plochu:

* `ChlorosLocal`, `process_folder` a `process_lattice_capture` **nefungujú** — potrebujú binárny súbor backendu, ktorý je súčasťou inštalátora pre pracovnú plochu.
* Pomocné programy smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) sú čistými klientmi typu HTTP, takže fungujú s backendom na inom počítači — dodávané backendy sa však viažu iba na loopback, takže port musíte presmerovať sami (napr. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) a odovzdať `backend_url="http://127.0.0.1:5000"` spolu s `auto_start_backend=False`. Pozrite si [Režim vzdialeného backendu](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Triedy LATTICE pre priamy prístup k hardvéru (`LatticeCamera`, `CameraPool`, …) sa dajú importovať, ale vyžadujú si runtime prostredie Arena SDK z balíka pre stolné počítače — bez neho je `CAMERA_AVAILABLE` rovnaký ako `False`.
* `daq_sdk` (triedy pre priame DAQ) je súčasťou inštalácie pre stolné počítače, nie balíka PyPI, takže `DAQ_AVAILABLE` je na hostiteľovi, kde sa používa iba pip, rovnaký ako `False` — namiesto toho ovládať senzory DAQ prostredníctvom `connect_daq_sensor()` voči (tunelovanému) backendu.

## Licenčné požiadavky

Prístup k SDK vyžaduje aktívne prihlásenie cez Chloros+ v akejkoľvek platenej úrovni — **Copper alebo vyššej**(Copper / Bronze / Silver / Gold); bezplatná úroveň Iron nemá prístup k službám SDK ani CLI. Kontrola sa vykonáva**na strane servera**: každá požiadavka typu SDK musí obsahovať aktívnu reláciu aj platený plán, inak backend vráti chybu `403` / `PLAN_UPGRADE_REQUIRED` (vygenerované ako `ChlorosLicenseError` pomocou `ChlorosLocal` a ako `ChlorosConnectError` pomocnými funkciami `connect_*`). Odhlásenému volajúcemu sa namiesto toho zobrazí `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) – opätovné spustenie `chloros-cli login` vyrieši prvý prípad, ale nie druhý.

Offline používanie funguje počas tolerančného obdobia plánu: úroveň sa číta z vyrovnávacej pamäte overovania serverom (5 minút) alebo z vyrovnávacej pamäte podpísanej licencie viazanej na zariadenie (30 dní pre mesačné plány; do vypršania predplatného pre ročné plány). Po uplynutí tolerančného obdobia sa plán zmení na bezplatný a prístup prostredníctvom SDK sa pozastaví, kým sa zariadenie aspoň raz nepripojí k serveru. `chloros-cli status` zostáva dostupný v rámci bezplatnej úrovne, takže dôvod je vždy viditeľný. Pozrite si [Chloros+ Prihlásenie](chloros+-login.md).

## Výnimky

Zachyťte základnú triedu na spracovanie „všetkého, čo sa pokazilo“ (Chloros):

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

Všetky výnimky potrubia (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) pochádzajú z `ChlorosError`. Jedna úskalia: `ChlorosConnectError` — vyvoláva iba `connect_camera` / `connect_array` / `connect_daq_sensor` — pochádza z bežnej chyby `Exception`, **nie** z `ChlorosError`, takže `except ChlorosError` ju nezachytí. Úplná hierarchia je uvedená v [Referencii SDK](reference/sdk-reference.md#exceptions).

## Pozri tiež

* [Referencia k SDK](reference/sdk-reference.md) — kompletná rozhranie API, optimalizované pre AI asistentov.
* [Referencia CLI](reference/cli-reference.md) — každý podpríkaz CLI zodpovedá volaniu SDK.
* [Stiahnutie](download.md) — inštalačné súbory pre Windows a Linux.

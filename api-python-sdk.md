# API: Python SDK

{% hint style="info" %}
**Hľadáte kompletné API?** Táto stránka je praktickým návodom. Všetky verejné triedy, metódy, presné podpisy a príklady, ktoré si môžete skopírovať a vložiť, nájdete v [Referencii SDK](reference/sdk-reference.md), ktorá je optimalizovaná pre AI asistentov.**Pracujete s AI asistentom?** Vložte túto URL do chatu, aby mal k dispozícii úplné a aktuálne API Chloros 1.3.0:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Každá stránka tejto príručky je k dispozícii ako surový markdown pod názvom s malými písmenami + `.md` a celá príručka je indexovaná na adrese `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` na PyPI) riadi všetko, čo dokáže desktopová aplikácia z jazyka Python: hromadné spracovanie obrázkov, ovládanie kamery LATTICE a snímačov v reálnom čase, relácie DAQ so svetelnými senzormi a automatizáciu uložených projektov. Ide o tenkú vrstvu nad tým istým lokálnym backendom, ktorý používajú grafické rozhranie (GUI) aj príkazový riadok (CLI) (HTTP na `127.0.0.1:5000`), takže správanie je identické vo všetkých troch prostrediach.

## Inštalácia

SDK je súčasťou balíka Chloros, ktorý obsahuje aj backend na spracovanie a hardvérové runtime prostredia a vždy zodpovedá jeho verzii.

**Krok 1 — Inštalácia Chlorosu.** Windows: spustite inštalačný program pre desktop (predvolená cesta `C:\Program Files\Chloros\`) zo stránky [Stiahnuť](download.md). Linux: nainštalujte balík `.deb` pomocou `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Inštalácia v Linuxe](linux/linux-installation.md)).**Krok 2 — Skontrolujte SDK** (Python 3.7+). Zvyčajne nie je potrebné nič inštalovať:

* **Windows:** inštalačný program nainštaluje balík SDK wheel do vášho systémového Pythonu (ak nie je nainštalovaný žiadny Python, tento krok preskočí).
* **Linux:** balík `.deb` nainštaluje SDK pre systémový balík `python3` do balíka `/usr/lib/python3/dist-packages`. Požiadavky na balík, `python3-numpy` a `python3-requests`, sú odporúčané balíkom: `apt install` ich obsahuje, `dpkg -i` nie.
* **Prostredie virtualenv** nevidí systémové balíky; nainštalujte do neho priložený súbor wheel: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` v systéme Linux alebo súbor `chloros_sdk-*.whl` v priečinku `sdk` v inštalačnom priečinku systému Windows (štandardne `C:\Program Files\Chloros\sdk\`).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Na počítači bez balíka Chloros, ktorého SDK ovláda backend na inom počítači (pozri [Čo prináša inštalácia iba cez pip](#what-a-pip-only-install-gets)), nainštalujte ho z PyPI: `pip install chloros-sdk`. PyPI sa aktualizuje pri vydaní nových verzií, takže zodpovedá najnovšej stabilnej verzii.

**Krok 3 — Prihláste sa raz na každom počítači:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Prihlasovacie údaje sa ukladajú do vyrovnávacej pamäte v `~/.chloros/` (na oboch platformách). V systéme Windows sa môžete prihlásiť aj prostredníctvom karty „User“ <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> v desktopovej aplikácii. SDK vyžaduje platený plán Chloros+ — pozrite si [Požiadavky na licenciu](#license-requirement) nižšie.

| Požiadavka | Podrobnosti |
| --- | --- |
| **Nainštalovaný Chloros** | Windows: inštalačný program pre počítač; Linux: balík `.deb` (obsahuje binárny súbor backendu a SDK) |
| **Jazyk Python** | 3.7 alebo novší (vyvinuté/testované na verzii 3.10) |
| **Operačný systém** | Windows 10/11 (64-bitová verzia), Ubuntu 22.04 LTS alebo novší, alebo NVIDIA Jetson (JetPack 6) |
| **Licencia** | Aktívne prihlásenie do Chloros+, akákoľvek platená úroveň (Copper alebo vyššia) |

## Úspech za 60 sekúnd

Jedným volaním vytvoríte projekt, naimportujete zložku, nakonfigurujete spracovanie a spustíte pipeline — pričom sa automaticky spustí backend, ak ešte nebeží:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(V systéme Linux použite Linuxové cesty: `/home/user/drone_images/flight001`. SDK funguje na oboch platformách rovnako.)

Spracovávate zložku so záznamami z LATTICE? Použite wrapper prispôsobený pre LATTICE – uplatňuje správne predvolené nastavenia (bez detekcie cieľového panelu, štandardný debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — plná kontrola nad spracovacím reťazcom

Pre akúkoľvek úlohu, ktorá presahuje jeden riadok, použite `ChlorosLocal`. Využíva backend, ktorý už beží (backend desktopovej aplikácie), alebo pri prvom použití spustí nainštalovaný backend (`auto_start_backend=True`; v systéme Windows zo zložky `backend` v inštalačnej zložke Chloros, štandardne `C:\Program Files\Chloros\backend\`), vytvára a konfiguruje projekty, monitoruje priebeh a vráti súhrn po spustení.

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
Používajte predvolený názov `http://127.0.0.1:5000` namiesto nahradenia názvom `localhost` — v systéme Windows sa `localhost` najskôr premenuje na `::1` a pri backende podporujúcom iba IPv4 to trvá približne 2 sekundy na jednu požiadavku.
{% endhint %}

Použite ho ako správcu kontextu. Opustenie bloku ukončí používanie backendu touto inštanciou, nie samotný backend: ten zostane spustený pre ďalší skript a zastaví sa sám po 30 minútach nečinnosti. `shutdown_backend()` zastaví túto inštanciu po jej spustení.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` akceptuje tieto kľúčové slová: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` a `custom_settings`. Hlavné hodnoty:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Ovládacie prvky špecifické pre LATTICE (`input_level`, `radiometric_output`, rodina `array_alignment*`) sú zdokumentované spolu s tabuľkami ich úplných hodnôt v [Referencii SDK](reference/sdk-reference.md#supported-values).

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

### Čítanie súhrnu po spustení — a zachytávanie prázdnych spustení

Po dokončení `process()` pripojí súhrn spracovania backendu ako `result["summary"]`. Každá položka v `summary["hints"]` je úplná veta vysvetľujúca všetko dôležité — napríklad, prečo beh nevytvoril žiadny výstup — a každý tip sa tiež opätovne vygeneruje ako Pythonový súbor `UserWarning`, takže prázdne behy sa diagnostikujú automaticky, aj keď slovník nikdy nepreveríte:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` sa nevyvolá, ak spustenie nevytvorí žiadne obrázky.** Toto je jediné miesto, kde sa SDK a CLI zámerne líšia: `chloros-cli process` považuje situáciu „boli požiadané výstupy, žiadne neboli vytvorené“ za zlyhanie a ukončí sa s nenulovým kódom, zatiaľ čo SDK sa vráti normálne a nahlási tento stav prostredníctvom `summary` / hints. Ak má váš pipeline zastaviť pri prázdnom behu, skontrolujte to sami — skontrolujte `summary` (alebo spočítajte súbory v zložke projektu) namiesto toho, aby ste sa spoliehali na výnimku.
{% endhint %}

## Smart Connect — aktívny hardvér

Tri pomocné skripty otvárajú trvalé relácie v hardvérovom poole backendu — v tom istom poole, ktorý používajú CLI, ostatné skripty a karty kamery v grafickom rozhraní, takže skripty SDK s nimi koexistujú bez súperenia o sériové porty alebo šírku pásma siete. Všetky tri automaticky spustia lokálny backend, ak žiadny nebeží. Panel DAQ v grafickom rozhraní nie je súčasťou tohto poola: pred otvorením svetelného senzora zo skriptu ho najprv zatvorte v grafickom rozhraní.

### Jedna kamera LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Relatívne cesty smerujú do adresára, kde sa spúšťa váš skript (zmenené vo verzii 1.3.0).** Pri lokálnom backende je relatívna cesta `output_dir` (vrátane predvolenej `"output"`) — a ostatné cesty, ktoré backend otvára (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — sa prekladajú voči pracovnému adresáru vášho skriptu, takže `cam.capture("output/")` zapisuje do `<current folder>/output/`. Vo verzii 1.2.0 ich nainštalovaný backend riešil vo vlastnej inštalačnej zložke a došlo k chybe s hlásením „Prístup je zamietnutý“. V prípade vzdialeného backendu sa cesty odosielajú tak, ako boli zadané.
{% endhint %}

### Synchronizované pole — `connect_array`

`connect_array` je odporúčaný vstupný bod pre zostavy s viacerými kamerami. Spúšťa rovnaký postup inteligentnej prípravy ako grafické používateľské rozhranie: analýzu siete, automatický výber synchronizačnej vrstvy, časovú synchronizáciu PTP, výber formátu pixelov pre každú kameru, inicializáciu AE a aktiváciu spúšťača GPIO. **Prvá séria je hlavná** (vysiela hardvérový spúšťací impulz); ostatné sú podriadené.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Pridajte `smart=True` k akémukoľvek zachyteniu poľa, aby ste pred spustením počkali na ustálenie automatickej expozície vo všetkých kamerách.

Novinkou vo verzii 1.3.0 je, že `ArraySession` vám tiež priamo poskytuje synchronizované snímky bez zapisovania súborov:

* `arr.grab()` vráti jeden snímok na kameru, všetky z jedného spúšťacieho impulzu, vo forme numpy polí (`{serial: array}`); `.group_id` je zdieľané spustenie a `.errors` označuje akúkoľvek vynechanú kameru.
* `arr.frame_stream(fps=5.0)` vráti jednu takúto sadu za každý tik na účely spracovania v reálnom čase vo vašom vlastnom spracovateľskom reťazci.
* `arr.continuous("output/")` spúšťa vlastné nepretržité snímanie backendu (režim Continuous v grafickom rozhraní) a vráti `ContinuousHandle` spolu s `status()`, `wait()` a `stop()` — použite ho namiesto cyklu snímania, ak potrebujete, aby sa každý spúšťací signál ukladal s vysokou frekvenciou.

Informácie o režimoch snímania (Jednorazové / Nepretržité / Intervalové / Najrýchlejšie), záznamníkoch, burst-to-video a zarovnaní poľa nájdete v [Referencii SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Senzor osvetlenia DAQ — `connect_daq_sensor`

Bez argumentov `connect_daq_sensor()` automaticky rozpozná prenosový protokol (v poradí: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Každý rámec obsahuje 135-bodovú hodnotu `spectrum` (W/m²/nm po kalibrácii), príznak `is_saturated` a CIE `x`, `y`, `z`. Ak chcete priradiť konkrétny senzor alebo spôsob prenosu, zadajte jeden explicitný údaj:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Upozorňujeme, že profily korekcie maximálnej hodnoty (`cap_id`) **nie sú** nastavením v rámci SDK — vyberte ich namiesto toho prostredníctvom `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Uložené projekty — `open_project`

Uložený projekt Chloros si zachováva pripojený hardvér (`cameras.json` + `sensors.json` spolu s `project.json`), a `chloros_sdk.open_project(path)` môže naraz opätovne pripojiť svoje kamery, pole a svetelné senzory a riadiť snímky podľa názvu zariadenia. Svetelné senzory sa opätovne pripájajú prostredníctvom bežiaceho backendu Chlorosu so svojimi uloženými nastaveniami a limitmi. Pozrite si časť [Automatizácia projektu](reference/sdk-reference.md#project-automation--chlorosproject) v referenčnom materiáli.

## Čo prináša inštalácia iba s pip

Pred použitím hardvérových plôch skontrolujte príznaky dostupnosti na úrovni modulov:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Na hostiteľskom počítači s **len** `pip install chloros-sdk` a bez balíka Chloros pre pracovnú plochu:

* `ChlorosLocal`, `process_folder` a `process_lattice_capture` **nefungujú** — potrebujú binárny súbor backendu, ktorý je súčasťou inštalátora pre pracovnú plochu.
* Pomocné programy smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) sú čisté HTTP klienti, takže fungujú s backendom na inom počítači — dodávané backendy sa však viažu iba na loopback, takže port musíte presmerovať sami (napr. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) a odovzdať `backend_url="http://127.0.0.1:5000"` spolu s `auto_start_backend=False`. Pozrite si [Režim vzdialeného backendu](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Triedy LATTICE pre priamy hardvér (`LatticeCamera`, `CameraPool`, …) sa dajú importovať, ale vyžadujú si runtime prostredie Arena SDK z balíka pre stolné počítače — bez neho je `CAMERA_AVAILABLE` rovnaké ako `False`.
* Senzory DAQ sú riadené iba prostredníctvom (tunelovaného) backendu — `connect_daq_sensor()` alebo projektového `proj.sensors`; backend otvorí senzor na svojom vlastnom počítači.

## Asistenti umelej inteligencie — `chloros-mcp`

SDK obsahuje `chloros-mcp`, server MCP, ktorý umožňuje aplikáciám Claude Desktop, Claude Code, Cursor a ďalším klientom MCP ovládať Chloros: spracovávať obrazové údaje a ovládať kamery LATTICE, senzorové polia a svetelné senzory DAQ. Jeho nástroje pokrývajú funkcie tohto SDK. Inštaluje sa spolu s SDK, takže v prípade aplikácie Claude Code stačí na nastavenie jeden riadok:

```bash
claude mcp add chloros -- chloros-mcp
```

Ostatní klienti: [Používanie Chloros s AI asistentmi](ai-assistants.md). Možnosti a všetky nástroje: [MCP Server (AI asistenti)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Licenčné požiadavky

Prístup k SDK vyžaduje aktívne prihlásenie do Chloros+ v akejkoľvek platenej úrovni — **Copper alebo vyššej**(Copper / Bronze / Silver / Gold); bezplatná úroveň Iron nemá prístup k SDK/CLI. Kontrola sa vykonáva**na strane servera**: každá požiadavka SDK musí obsahovať aktívnu reláciu aj platený plán, inak backend vráti chybu `403` / `PLAN_UPGRADE_REQUIRED` (generované ako `ChlorosLicenseError` pomocnou funkciou `ChlorosLocal` a ako `ChlorosConnectError` pomocnou funkciou `connect_*`). Odhlásený volajúci dostane namiesto toho chyby `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — opätovné spustenie `chloros-cli login` vyrieši prvý prípad, ale nie druhý.

Offline používanie funguje počas tolerančného obdobia plánu: úroveň sa číta z vyrovnávacej pamäte overovania serverom (5 minút) alebo z vyrovnávacej pamäte podpísanej licencie viazanej na zariadenie (30 dní pre mesačné plány; do vypršania predplatného pre ročné plány). Po uplynutí tolerančného obdobia sa plán zmení na bezplatný a prístup k SDK sa pozastaví, kým sa zariadenie aspoň raz nepripojí k serveru. Kód chyby `chloros-cli status` zostáva dostupný aj v bezplatnej úrovni, takže dôvod je vždy viditeľný. Pozrite si [Prihlásenie do Chloros+](chloros+-login.md).

## Výnimky

Zachyťte základnú triedu na spracovanie „akýchkoľvek chýb v Chloros“:

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

Všetky výnimky v potrubí (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) pochádzajú z `ChlorosError`. Jedna úskalia: `ChlorosConnectError` — vyvoláva iba `connect_camera` / `connect_array` / `connect_daq_sensor` — pochádza z bežného `Exception`, **nie** z `ChlorosError`, takže `except ChlorosError` ju nezachytí. Úplná hierarchia je uvedená v [Referencii SDK](reference/sdk-reference.md#exceptions).

## Pozri tiež

* [Referencia SDK](reference/sdk-reference.md) — kompletná rozhranie API, optimalizované pre AI asistentov.
* [Referenčná príručka CLI](reference/cli-reference.md) — každý podpríkaz CLI zodpovedá volaniu SDK.
* [Používanie Chloros s AI asistentmi](ai-assistants.md) — návody na nastavenie a príkazové riadky pre `chloros-mcp`.
* [Stiahnutie](download.md) — inštalačné súbory pre Windows a Linux.

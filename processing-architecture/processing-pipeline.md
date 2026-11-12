# Spracovateľský reťazec

Chloros 1.3.0 využíva 4-vláknový spracovateľský reťazec, ktorý funguje ako postupná montážna linka. Každé vlákno spracováva samostatnú fázu pracovného toku, takže v rôznych fázach môže byť súčasne spracovávaných viacero obrázkov.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Architektúra spracovateľského potrubia

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Každý obrázok prechádza postupne všetkými štyrmi vláknami. Vďaka viacvláknovému spracovaniu v Chloros+ môže viacero obrázkov súčasne obsadzovať rôzne vlákna — zatiaľ čo vlákno 3 spracováva jeden obrázok, vlákno 1 môže detekovať ďalší, vlákno 2 kalibrovať iný a vlákno 4 zapisovať hotový obrázok na disk.

Priebeh spracovania sa hlási pre každé vlákno a prenáša sa prostredníctvom udalostí Server-Sent Events (backend ich publikuje na `/api/events`). V živom zobrazení priebehu v CLI sú štyri fázy označené ako **Detekcia, Analýza, Spracovanie, Export**.

***

## Podrobnosti o vláknach

### Vlákno 1: Detekcia

**Účel**: Načítanie obrázkov a detekcia kalibračných cieľov.

* Načíta obrazové súbory z disku — páry Survey3 `.raw`+`.jpg`, zábbery LATTICE `.tif`/`.tiff` a `.dng`
* Extrahuje metadáta EXIF (GPS, model fotoaparátu, časové značky, expozícia)
* Detekuje kalibračné ciele: geometrie cieľov označené ArUco pre snímky LATTICE a klasický panelový detektor pre fotografie kalibračných cieľov programu Survey3
* Výstupy: obrazové údaje + metadáta + výsledky detekcie cieľov

Ide predovšetkým o vlákno závislé od vstupov/výstupov a procesora.

### Vlákno 2: Kalibrácia

**Účel**: Výpočet kalibračných parametrov na základe detegovaných cieľov.

* Vypočíta koeficienty kalibrácie odrazivosti z obrázkov cieľov
* Vypočíta parametre korekcie vinetácie
* Určí kalibračné krivky pre jednotlivé pásma
* Výstupy: kalibračné parametre pre každý obrázok

Výpočtové vlákno závislé od CPU. Pri spustení programu Survey3 alebo zmiešanom spustení naň čaká vlákno 3, ak je povolená kalibrácia odrazivosti, takže jeho koeficienty sú pripravené skôr, ako sa spracuje akýkoľvek obrázok. Pri spustení výlučne s programom LATTICE beží vlákno 3 počas detekcie: každý snímok sa spustí, akonáhle je jeho cieľová kotva definitívna. Snímka berie do úvahy časovo najbližší cieľ na oboch stranách, takže čaká, kým sa naskenujú všetky snímky z jej kamery, ktoré sú časovo bližšie ako jej aktuálny najbližší cieľ. Výstupy sú rovnaké, len skôr.

### Vlákno 3: Spracovanie (GPU)

**Účel**: Uplatniť korekcie a vypočítať vegetačné indexy.**Toto je vlákno s najvyššou výpočtovou náročnosťou.*** **Debayering**: prevádza surové Bayerove údaje na viackanálové snímky
  * Štandardné (rýchle, stredná kvalita) — predvolené, `--debayer standard`
  * S ohľadom na textúru (pomalé, najvyššia kvalita) — iba Chloros+, `--debayer texture-aware`, používa model odšumovania založený na AI/ML
  * Snímky LATTICE mono (M3M) sú jednopásmové: kroky demosaic a vyváženia bielej sa pri nich preskakujú (s jednoriadkovou správou v protokole), zatiaľ čo všetky snímky M3C/Bayer v tom istom behu ich stále prechádzajú
* **Korekcia vinetácie**: aplikuje korekciu vinetácie objektívu na celý obraz
* **Kalibrácia odrazivosti**: aplikuje kalibračné koeficienty na prevod na hodnoty odrazivosti
* **Výpočet indexov**: vypočítava vegetačné indexy (NDVI, NDRE, GNDVI, …)
* Výstupy: spracované obrazové údaje pripravené na export

Toto vlákno najviac ťaží z akcelerácie GPU a je to práve vlákno, ktoré ladí [Dynamic Compute Adaptation](dynamic-compute-adaptation.md).

### Vlákno 4: Export

**Účel**: Zapisuje spracované snímky na disk.

* Zapisuje výstupné súbory vo vybranom formáte — `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Do výstupných súborov vkladá metadáta (GPS, časové značky, parametre spracovania) — vrátane značiek spracovania `Chloros:*`, ktoré sa od verzie 1.3.0 zapisujú v každom režime spracovania, a to ako pre Survey3, tak aj pre LATTICE
* Usporiadava výstup do projektovej zložky ako `<camera>/<format>/<Product>_Images/` — napríklad `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Exportované súbory si zachovávajú názov zdrojového súboru; zložka identifikuje produkt.**
* V prípade snímok z LATTICE sa jeden zdrojový snímok môže rozvetviť na viacero produktov (Debayered, Preview, Radiance, Reflectance, Index), z ktorých každý je umiestnený vo vlastnej produktovej zložke. Indexové produkty sa počítajú na základe produktu odrazivosti, ktorý spustenie vytvorilo (ak žiadny neexistuje, na základe iného produktu, ktorý spustenie vytvorilo, napríklad radiancie). Produkty jasu a odrazivosti multispektrálnych kamier M3C môžu tiež získať masku obmedzenia na úrovni jednotlivých pixelov v súbore `<format>/clamp_masks/` (pozri [Formáty výstupných obrázkov](../output-image-formats.md#clamp-masks))
* Výstupy: konečné súbory na disku

Ide predovšetkým o vlákno obmedzené vstupom a výstupom — úložisko SSD výrazne zlepšuje jeho výkon.

***

## Pod kapotou: vykonávacie procesy

V rámci vlákna 3 je práca na jednotlivých snímkach paralelizovaná nasledovne:

* **Stratégie GPU**(`GPU_SINGLE`, `GPU_PARALLEL`) využívajú skupinu**pracovných procesov**, z ktorých každý je samostatným procesom operačného systému s vlastným kontextom CUDA. Chloros spúšťa každého pracovného procesa opätovným spustením vlastného nainštalovaného backendového programu — nie prostredníctvom Pythonu `multiprocessing`, ktorého spúšťanie pracovných procesov nefunguje vnútri balíkovej aplikácie
* **`CPU_PARALLEL`** využíva fond vlákien (funkcia Pythonu `ThreadPoolExecutor`) — NumPy a OpenCV uvoľňujú GIL, takže vlákna stačia
* Zariadenia Jetson so zdieľanou pamäťou RAM 8 GB alebo menej úplne vynechávajú vykonávaciu jednotku a spracovávajú v rámci procesu, sekvenčne
* Funkcia Texture Aware na GPU s menej ako 7 GB VRAM tiež beží sekvenčne — model odšumovača sa nemôže zmestiť viac ako raz

Chloros nepoužíva žiadny distribuovaný framework tretej strany (napríklad Ray). Pozrite si [Dynamic Compute Adaptation](dynamic-compute-adaptation.md), kde sa dozviete, ako sa volí stratégia a počet pracovníkov.

***

## Sekvenčné vs. potrubné spracovanie

### Voľný režim (sekvenčný)

V bezplatnej verzii programu Chloros sa obrázky spracúvajú **po jednom**, postupne vo všetkých štyroch fázach:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI zobrazuje v bezplatnom režime zjednodušený ukazovateľ priebehu; jeho sériové fázy sú označené ako **Detekcia cieľa**a následne**Spracovanie**.

### Režim Chloros+ (sériovo-paralelné spracovanie)

S licenciou Chloros+ všetky štyri vlákna pracujú **súbežne** na rôznych snímkach:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Indikátor priebehu v grafickom rozhraní zobrazuje štyri fázy; prejdite nad ním kurzorom, aby ste videli priebeh jednotlivých vlákien. V CLI sa tie isté štyri fázy zobrazujú v reálnom čase ako **Detekcia, Analýza, Spracovanie, Export**.

{% hint style="info" %}
**Jedna značka, dva názvy.** CLI nazýva fázu 3 _Spracovanie_. Prúd priebehu v prémiovom režime backendu — ten, ktorý zobrazuje indikátor priebehu v grafickom rozhraní — označuje tú istú fázu ako _Kalibrácia_. Ide o to isté vlákno, ktoré vykonáva tú istú prácu (Vlákno 3: debayer, korekcie, indexy).
{% endhint %}

{% hint style="success" %}
**Spracovanie v potrubí s Chloros+** môže byť 3–5-krát rýchlejšie ako sekvenčné spracovanie, v závislosti od vášho hardvéru a veľkosti dátového súboru. Zrýchlenie je najväčšie na systémoch s rýchlymi grafickými kartami a SSD diskami.
{% endhint %}

***

## Priebeh exportu v vlákne 4

Exportné vlákno má vlastné sledovanie priebehu, ktoré môžete kontrolovať samostatne:**Príkazový riadok (CLI):**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Spracovanie je dokončené, keď vlákno 4 dosiahne 100 %.

{% hint style="info" %}
**Spustenie, pri ktorom sa nezapíšu žiadne obrazy, je neúspešné.**V prípade úspechu hlási `chloros-cli process`, koľko obrazových produktov zapísalo (`Image products written: N`). Ak boli požiadané produkty a**žiadny**nebol zapísaný — iba `project.json` a `calibration_data.json` — CLI vypíše `Processing finished but wrote no image products.` a**ukončí sa s nenulovým kódom**, pričom uvedie názov projektovej zložky a bežné príčiny (vstupná zložka nebola rozpoznaná ako záznam – skontrolujte rozloženie a `--input-level` – alebo žiadny z požadovaných produktov nebol pre dané kamery použiteľný). Skripty sa môžu spoliehať na kód ukončenia.
{% endhint %}

***

## Vzťah k dynamickej adaptácii výpočtov

[Dynamická adaptácia výpočtov](dynamic-compute-adaptation.md) ovplyvňuje predovšetkým **vlákno 3 (spracovanie)**:

* **`GPU_PARALLEL`**: Vlákno 3 spracováva viacero obrázkov súčasne prostredníctvom GPU pomocou potrubia `fused_gpu`
* **`GPU_SINGLE`**: Vlákno 3 serializuje prístup k GPU pomocou semaforu, zatiaľ čo pracovné procesy prekrývajú vstupno-výstupné operácie, pričom využíva potrubie `fused_gpu` alebo potrubie `tiled_gpu` s efektívnym využívaním pamäte
* **`CPU_PARALLEL`**: Vlákno 3 využíva spracovanie na CPU s viacvláknovou paralelnosťou

Alokácia pamäte GPU vlákna 3 sa tiež zvyšuje, keď vlákna 1 a 2 dokončia svoju činnosť — pozri [Dynamická alokácia pamäte GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Ďalšie kroky

* [Dynamická adaptácia výpočtov](dynamic-compute-adaptation.md) — Ako Chloros vyberá optimálnu stratégiu pre váš hardvér
* [Príručka k NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Správanie potrubia špecifické pre platformu Jetson
* [Monitorovanie spracovania](../processing-images-gui/monitoring-the-processing.md) — Monitorovanie priebehu prostredníctvom grafického používateľského rozhrania
* [Referenčná príručka CLI](../reference/cli-reference.md) — `process`, `export-status`, kódy ukončenia a rozloženie výstupu

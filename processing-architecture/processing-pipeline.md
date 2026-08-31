# Spracovateľský kanál

ChlorosVerzia 1.2.0 využíva 4-vláknový spracovateľský kanál, ktorý funguje ako postupná montážna linka. Každé vlákno spracováva samostatnú fázu pracovného toku, takže v rôznych fázach môže byť súčasne spracovávaných viacero obrázkov.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Architektúra spracovateľského potrubia

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Každý obrázok prechádza postupne všetkými štyrmi vláknami. Vďaka viacvláknovému spracovaniu v Chloros+ môže viacero obrázkov súčasne obsadzovať rôzne vlákna – zatiaľ čo vlákno 3 spracováva jeden obrázok, vlákno 1 môže detekovať ďalší, vlákno 2 kalibrovať iný a vlákno 4 zapisovať hotový obrázok na disk.

Priebeh spracovania sa hlási pre každé vlákno a prenáša sa prostredníctvom udalostí Server-Sent Events (backend ich publikuje na `/api/events`). V živom zobrazení priebehu v nástroji CLI sú štyri fázy označené ako **Detekcia, Analýza, Spracovanie, Export**.***

## Podrobnosti o vláknach

### Vlákno 1: Detekcia

**Účel**: Načítať obrázky a detekovať kalibračné ciele.

* Číta obrazové súbory z disku — páry Survey3 `.raw`+`.jpg`, zachytenia LATTICE `.tif`/`.tiff` a `.dng`
* Extrahuje metadáta EXIF (GPS, model fotoaparátu, časové značky, expozícia)
* Detekuje kalibračné ciele: geometrie cieľov označené ArUco pre snímky LATTICE a klasický panelový detektor pre fotografie kalibračných cieľov Survey3
* Výstupy: obrazové údaje + metadáta + výsledky detekcie cieľov

Ide predovšetkým o vlákno závislé od vstupov/výstupov a CPU.

### Vlákno 2: Kalibrácia

**Účel**: Výpočet kalibračných parametrov na základe detegovaných cieľov.

* Vypočíta kalibračné koeficienty odrazivosti z obrázkov cieľov
* Vypočíta parametre korekcie vinetácie
* Určí kalibračné krivky pre jednotlivé pásma
* Výstupy: kalibračné parametre pre každý obrázok

Výpočtové vlákno závislé od CPU. Vlákno 3 naň čaká, ak je povolená kalibrácia odrazivosti, aby boli jeho koeficienty pripravené ešte pred spracovaním akéhokoľvek obrázku.

### Vlákno 3: Spracovanie (GPU)

**Účel**: Uplatniť korekcie a vypočítať vegetačné indexy.**Toto je výpočtovo najnáročnejšie vlákno.*** **Debayering**: konvertuje surové Bayerove údaje na viackanálové snímky
  * Štandardné (rýchle, stredná kvalita) — predvolené, `--debayer standard`
  * S ohľadom na textúru (pomalé, najvyššia kvalita) — iba pre Chloros+, `--debayer texture-aware`, používa model odšumovania založený na AI/ML
  * Snímky LATTICE mono (M3M) sú jednopásmové: kroky demosaic a vyváženia bielej sa pri nich preskakujú (s jednoradkovou správou v protokole), zatiaľ čo všetky snímky M3C/Bayer v tom istom cykle ich stále prechádzajú
* **Korekcia vinetácie**: aplikuje korekciu vinetácie objektívu na celý snímok
* **Kalibrácia odrazivosti**: aplikuje kalibračné koeficienty na prevod na hodnoty odrazivosti
* **Výpočet indexov**: vypočítava vegetačné indexy (NDVI, NDRE, GNDVI, …)
* Výstupy: spracované obrazové údaje pripravené na export

Toto vlákno najviac ťaží z akcelerácie GPU a je to práve vlákno, ktoré optimalizuje funkcia [Dynamic Compute Adaptation](dynamic-compute-adaptation.md).

### Vlákno 4: Export

**Účel**: Zapisuje spracované snímky na disk.

* Ukladá výstupné súbory vo vybranom formáte — `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Vkladá metadáta do výstupných súborov (GPS, časové značky, parametre spracovania)
* Usporiadava výstup do zložky projektu ako `<camera>/<format>/<Product>_Images/` — napríklad `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Exportované súbory si zachovávajú názov zdrojového súboru; produkt identifikuje zložka.**
* V prípade snímok LATTICE sa jeden zdrojový snímok môže rozvetviť na viacero produktov (Debayered, Preview, Radiance, Reflectance, Index), z ktorých každý je umiestnený vo vlastnej zložke produktu
* Výstupy: finálne súbory na disku

Ide predovšetkým o vlákno obmedzené vstupom a výstupom — úložisko SSD ho výrazne zrýchľuje.

***

## Pod kapotou: Exekútory

V rámci vlákna 3 je práca na jednotlivých snímkach paralelizovaná pomocou štandardného `concurrent.futures` z balíka „Python“:

* **Stratégie GPU**(`GPU_SINGLE`, `GPU_PARALLEL`) používajú metódu**spawn** — každý pracovník je samostatný proces s vlastným kontextom CUDA (`fork` by zdedil inicializovaný stav CUDA rodičovského procesu a poškodil by potomkov)
* **`CPU_PARALLEL`** používa `ThreadPoolExecutor` — NumPy a OpenCV uvoľňujú GIL, takže stačia vlákna
* Zariadenia Jetson so zdieľanou RAM s kapacitou 8 GB alebo menej úplne vynechávajú vykonávaciu jednotku a spracovávajú v rámci procesu sekvenčne
* Funkcia Texture Aware na GPU s VRAM menšou ako 7 GB tiež beží sekvenčne — model odšumovača sa nemôže zmestiť viac ako raz

Chlorosnepoužíva žiadny distribuovaný framework tretej strany (napríklad Ray). Pozrite si [Dynamic Compute Adaptation](dynamic-compute-adaptation.md), kde sa dozviete, ako sa volí stratégia a počet pracovníkov.

***

## Sekvenčné vs. potrubné spracovanie

### Voľný režim (sekvenčný)

V bezplatnej verzii Chloros sa obrázky spracúvajú **po jednom**, postupne vo všetkých štyroch fázach:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI zobrazuje v bezplatnom režime zjednodušený ukazovateľ priebehu; jeho sériové fázy sú označené ako **Detekcia cieľa**a potom**Spracovanie**.

### Režim „Chloros“ (pipelined)

S licenciou „Chloros“ všetky štyri vlákna pracujú **súbežne** na rôznych snímkach:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Indikátor priebehu v grafickom rozhraní zobrazuje štyri fázy; prejdite nad ním kurzorom, aby ste videli priebeh jednotlivých vlákien. V nástroji CLI sa tie isté štyri fázy zobrazujú v reálnom čase ako **Detekcia, Analýza, Spracovanie, Export**.

{% hint style="info" %}
**Jeden názov, dve označenia.** V súbore `CLI` sa fáza 3 nazýva _Spracovanie_. Kanál priebehu v prémiovom režime backendu — ten, ktorý zobrazuje indikátor priebehu v grafickom rozhraní — označuje tú istú fázu ako _Kalibrácia_. Ide o to isté vlákno, ktoré vykonáva tú istú prácu (vlákn 3: debayer, korekcie, indexy).
{% endhint %}

{% hint style="success" %}
**Sériové spracovanie s funkciou „Chloros“** môže byť 3–5-krát rýchlejšie ako sekvenčné spracovanie, v závislosti od vášho hardvéru a veľkosti dátového súboru. Zrýchlenie je najväčšie na systémoch s rýchlymi grafickými kartami (GPU) a SSD diskami.
{% endhint %}

***

## Priebeh exportu v vlákne 4

Vlákno exportu má vlastné sledovanie priebehu, ktoré môžete kontrolovať samostatne:**CLI:**

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
**Spustenie, pri ktorom sa nezapíšu žiadne obrázky, je neúspešné.**V prípade úspechu `chloros-cli process` nahlási, koľko obrazových produktov zapísal (`Image products written: N`). Ak boli požiadané produkty a**žiadny**nebol zapísaný — iba `project.json` a `calibration_data.json` — program CLI vypíše `Processing finished but wrote no image products.` a**ukončí sa s nenulovým kódom**, pričom uvedie názov projektovej zložky a bežné príčiny (vstupná zložka nebola rozpoznaná ako záznam – skontrolujte rozloženie a `--input-level` – alebo žiadny zo žiadaných produktov nebol pre dané kamery použiteľný). Skripty sa môžu spoliehať na kód ukončenia.
{% endhint %}

***

## Vzťah k dynamickej adaptácii výpočtov

[Dynamická adaptácia výpočtov](dynamic-compute-adaptation.md) ovplyvňuje predovšetkým **vlákno 3 (spracovanie)**:

* **`GPU_PARALLEL`**: Vlákno 3 spracováva viacero obrázkov súčasne prostredníctvom GPU pomocou potrubia `fused_gpu`
* **`GPU_SINGLE`**: Vlákn 3 serializuje prístup k GPU pomocou semaforu, zatiaľ čo pracovné procesy prekrývajú vstupy a výstupy, pričom využíva potrubie `fused_gpu` alebo potrubie `tiled_gpu` šetrné k pamäti
* **`CPU_PARALLEL`**: Vlákno 3 využíva spracovanie na báze CPU s viacvláknovou paralelnosťou

Alokácia pamäte GPU vlákna 3 sa tiež zvyšuje, keď vlákna 1 a 2 dokončia svoju činnosť — pozri [Dynamická alokácia pamäte GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Ďalšie kroky

* [Dynamická adaptácia výpočtov](dynamic-compute-adaptation.md) — Ako Chloros vyberá optimálnu stratégiu pre váš hardvér
* [Príručka k NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Správanie potrubia špecifické pre platformu Jetson
* [Monitorovanie spracovania](../processing-images-gui/monitoring-the-processing.md) — Monitorovanie priebehu prostredníctvom grafického používateľského rozhrania
* [Referenčné údaje o CLI](../reference/cli-reference.md) — `process`, `export-status`, kódy ukončenia a rozloženie výstupu

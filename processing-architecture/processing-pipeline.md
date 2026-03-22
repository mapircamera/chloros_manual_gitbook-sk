# Spracovateľský reťazec

Chloros 1.1.0 využíva 4-vláknový spracovateľský reťazec, ktorý funguje ako postupná montážna linka. Každé vlákno spracováva samostatnú fázu pracovného postupu, čo umožňuje súbežné spracovanie viacerých obrázkov v rôznych fázach.

***

## Architektúra reťazca

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Každý obrázok prechádza všetkými štyrmi vláknami v poradí. Vďaka viacvláknovému spracovaniu v Chloros+ môže byť viacero obrázkov súčasne v rôznych vláknach – zatiaľ čo vlákno 3 spracováva jeden obrázok, vlákno 1 môže detekovať ďalší, vlákno 2 môže kalibrovať iný a vlákno 4 môže zapisovať predtým spracovaný obrázok na disk.

***

## Podrobnosti o vláknach

### Vlákno 1: Detekcia

**Účel**: Načítanie obrázkov a detekcia kalibračných cieľov.

* Číta obrazové súbory z disku (RAW, JPG)
* Extrahuje metadáta EXIF (GPS, model fotoaparátu, časové značky, expozícia)
* Detekuje kalibračné ciele ArUco v označených cieľových obrázkoch
* Výstupy: obrazové dáta + metadáta + výsledky detekcie cieľov

Ide primárne o vlákno viazané na I/O a CPU.

### Vlákno 2: Kalibrácia

**Účel**: Vypočítať kalibračné parametre z detekovaných cieľov.

* Vypočíta kalibračné koeficienty odrazivosti z obrázkov cieľov
* Vypočíta parametre korekcie vinetácie
* Určí kalibračné krivky pre jednotlivé pásma
* Výstupy: kalibračné parametre pre každý obrázok

Toto je výpočtové vlákno závislé od CPU.

### Vlákno 3: Spracovanie (GPU)

**Účel**: Uplatniť korekcie a vypočítať vegetačné indexy.**Toto je vlákno s najvyššou výpočtovou náročnosťou.*** **Debayering**: Konvertuje surové dáta vzoru Bayer na viackanálové snímky
  * Štandardné (rýchle, stredná kvalita) — predvolené
  * S ohľadom na textúru (pomalé, najvyššia kvalita) — iba Chloros+, používa odšumovanie AI/ML
* **Korekcia vinetácie**: Uplatňuje korekciu vinetácie objektívu na celom snímku
* **Kalibrácia odrazivosti**: Uplatňuje kalibračné koeficienty na konverziu na hodnoty odrazivosti
* **Výpočet indexov**: Vypočíta vegetačné indexy (NDVI, NDRE, GNDVI atď.)
* Výstupy: spracované obrazové údaje pripravené na export

Toto vlákno najviac ťaží z akcelerácie GPU. Systém [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) primárne optimalizuje správanie tohto vlákna.

### Vlákno 4: Export

**Účel**: Zapisovanie spracovaných obrázkov na disk.

* Zapisuje výstupné súbory vo vybranom formáte (TIFF 16-bit, TIFF 32-bit %, PNG, JPG)
* Vkladá metadáta EXIF do výstupných súborov (GPS, časové značky, parametre spracovania)
* Usporiadava výstup do podpriečinkov podľa modelov fotoaparátov
* Výstupy: finálne súbory na disku

Ide predovšetkým o vlákno viazané na vstupno-výstupné operácie. SSD úložisko výrazne zlepšuje výkon vlákna 4.

***

## Sekvenčné vs. potrubné spracovanie

### Voľný režim (sekvenčný)

Vo voľnej verzii Chloros sa obrázky spracúvajú **po jednom**, sekvenčne cez všetky štyri fázy:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

Indikátor priebehu v grafickom rozhraní zobrazuje 2 fázy: Detekcia cieľa a Spracovanie.

### Režim Chloros+ (sériové spracovanie)

S licenciou Chloros+ všetky štyri vlákna pracujú **súčasne** na rôznych obrázkoch:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Indikátor priebehu v grafickom rozhraní zobrazuje 4 fázy: Detekcia, Analýza, Kalibrácia, Export. Nabehnite kurzorom na indikátor priebehu, aby ste videli priebeh jednotlivých vlákien.

{% hint style="success" %}
**Sériové spracovanie s Chloros+** môže byť 3-5x rýchlejšie ako sekvenčné spracovanie, v závislosti od vášho hardvéru a veľkosti dátového súboru. Zrýchlenie je najväčšie na systémoch s rýchlymi GPU a SSD.
{% endhint %}

***

## Priebeh exportu vlákna 4

V Chloros 1.1.0 má exportné vlákno (vlákno 4) vlastné vyhradené sledovanie priebehu. Priebeh exportu môžete sledovať samostatne:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Spracovanie je dokončené, keď vlákno 4 dosiahne 100 %.

***

## Vzťah k dynamickej adaptácii výpočtov

Systém [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) ovplyvňuje predovšetkým **vlákno 3 (spracovanie)**:

* **`GPU_PARALLEL`** stratégia: Vlákno 3 spracováva viacero obrazov súčasne cez GPU pomocou potrubia `fused_gpu`
* **`GPU_SINGLE`** stratégia: Vlákn 3 spracováva jeden obraz po druhom pomocou pamäťovo efektívneho potrubia `tiled_gpu`
* **`CPU_PARALLEL`** stratégia: Vlákn 3 používa spracovanie založené na CPU s viacvláknovou paralelnosťou

Alokácia pamäte GPU vlákna 3 sa tiež dynamicky mení v závislosti od dokončenia vlákien 1 a 2 — pozri [Dynamická alokácia pamäte GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Ďalšie kroky

* [Dynamická adaptácia výpočtov](dynamic-compute-adaptation.md) — Ako Chloros vyberá optimálnu stratégiu pre váš hardvér
* [Príručka NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Správanie potrubia špecifické pre platformu na Jetsone
* [Monitorovanie spracovania](../processing-images-gui/monitoring-the-processing.md) — Monitorovanie priebehu prostredníctvom grafického používateľského rozhrania

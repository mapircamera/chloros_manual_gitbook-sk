# Otvorenie obrázku na celej obrazovke

Prehliadač obrázkov Chloros poskytuje špecializované rozhranie na celej obrazovke na prezeranie, analýzu a manipuláciu s multispektrálnymi obrázkami. Nech už prezeráte originálne obrázky alebo spracované výstupy, prehliadač obrázkov ponúka výkonné nástroje na kontrolu a analýzu.

## Prístup k prehliadaču obrázkov

### Z prehliadača súborov

Najbežnejší spôsob otvorenia obrázku v prehliadači obrázkov:

1. Uistite sa, že ste na karte **Prehliadač súborov**. <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Kliknite na ľubovoľnú **miniaturu obrázka** v mriežke obrázkov.
3. Obrázok sa otvorí v **hlavnej oblasti náhľadu** (v strede obrazovky).
4. Obrázok je teraz načítaný a pripravený na zobrazenie na celej obrazovke.

### Otvorenie karty Image Viewer

Po načítaní obrázka do oblasti náhľadu:

1. Kliknite na ikonu **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> v ľavom bočnom paneli.
2. Otvorí sa karta Prehliadač obrázkov, kde sa vybraný obrázok zobrazí na celej obrazovke.
3. V ľavom bočnom paneli sa sprístupnia pokročilé nástroje na prezeranie a analýzu.

***

## Prehľad rozhrania prehliadača obrázkov

### Hlavná oblasť zobrazenia

Najväčšia časť obrazovky zobrazuje váš obrázok:

* **Plné rozlíšenie**: Obrázky sa zobrazujú v natívnom rozlíšení.
* **Zväčšiteľný**: Na zväčšenie použite ovládacie prvky alebo koliesko myši
* **Posuvný**: Po zväčšení kliknite a ťahajte, aby ste sa pohybovali
* **Zachované pomerové rozmery**: Obrázky sa zväčšujú proporcionálne

***

## Možnosti zobrazenia

### Základná navigácia v obrázkoch

#### Prehliadanie obrázkov

V sade obrázkov sa pohybujte pomocou klávesových skratiek alebo tlačidiel:

* **Ďalší obrázok**: Kliknite na tlačidlo → alebo stlačte klávesu **→** (šipka doprava)
* **Predchádzajúci obrázok**: Kliknite na tlačidlo ← alebo stlačte klávesu **←** (šipka doľava)
* **Preskočiť na konkrétny obrázok**: Vráťte sa do prehliadača súborov a kliknite na požadovanú miniatúru

#### Ovládacie prvky priblíženia

Upravte zväčšenie, aby ste mohli prezrieť detaily obrázku:

**Zväčšenie:**

* Kliknite na tlačidlo **+** (plus)
* Stlačte kláves **+** alebo **=**
* Posuňte koliesko myši **nahor**

**Zmenšenie:**

* Kliknite na tlačidlo **−** (mínus)
* Stlačte kláves **−** (mínus)
* Posuňte koliesko myši **nadol**

**Prispôsobiť obrazovke:**

* Kliknite na tlačidlo **↔** (Prispôsobiť)
* Stlačte kláves **0** (Nula)
* Dvakrát kliknite na obrázok

#### Posúvanie pri zväčšení

Pri zväčšení nad veľkosť obrazovky:

1. Presuňte kurzor myši nad obrázok
2. Kliknite a **podržte ľavé tlačidlo myši**
3. **Ťahajte**, aby ste posunuli obrázok
4. Uvoľnite tlačidlo, aby ste zastavili posúvanie

**Alternatíva**: Použite klávesy so šípkami na posúvanie v malých krokoch

***

## Kontrola hodnoty pixelov

### Zobrazenie hodnôt pixelov na kurzore

Keď presuniete kurzor myši nad obrázok, hodnoty pixelov sa zobrazia v reálnom čase:

**Umiestnenie zobrazenia hodnoty:**

* **Plávajúce číslo a červená čiara v pravej časti indexu LUT gradientnej legendy**
* **Pri ďalšom zväčšení sa zobrazí plávajúca hodnota v blízkosti kurzora a zvýrazneného pixelu**
* Zobrazuje hodnoty pre pixel **pod kurzorom alebo zvýraznený**
* Aktualizuje sa pri pohybe myši

***

## Typy obrázkov, ktoré môžete zobraziť

### Pôvodné obrázky (pred spracovaním)

**Obrázky RAW + JPG z fotoaparátu:**

* Zobrazenie údajov RAW ako náhľad
* Zobrazenie pôvodných, neopravených hodnôt
* Užitočné na kontrolu kvality obrázku pred spracovaním

### Kalibrované obrázky odrazivosti

**Po spracovaní:**

* Korekcia vinetácie
* Kalibrovaná odrazivosť
* Viacpásmové TIFF (Red, Green, NIR atď.)
* Vedecké údaje pripravené na analýzu

### Indexové obrázky

**NDVI, NDRE, GNDVI atď. (súbory \_NDVI.tif):**

* Jednopásmové obrázky v odtieňoch sivej
* Hodnoty pixelov predstavujú výsledky výpočtu indexu
* Rozsah typicky od -1 do +1 pre normalizované indexy
* Na vizualizáciu je možné použiť farebné LUT

***

## Použitie indexu a LUT

Použite multispektrálne indexy a farebné vyhľadávacie tabuľky:

1. Vyhľadajte **Index/LUT Sandbox** v **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> bočnom paneli
2. Vyberte index vegetácie (NDVI, NDRE atď.)
3. Vyberte multispektrálnu formulu alebo vytvorte vlastnú (len Chloros+)
4. Použite farebný LUT gradient na vizualizáciu
5. Upravte rozsahy hodnôt a prahové hodnoty

Podrobné pokyny nájdete v časti [Index/LUT Sandbox](index-lut-sandbox.md).

***

## Klávesové skratky

### Navigácia

* **→** (šipka doprava): Ďalší obrázok
* **←** (šipka doľava): Predchádzajúci obrázok
* **Home**: Prvý obrázok v zozname
* **End**: Posledný obrázok v zozname

### Zväčšenie

* **+** alebo **=**: Zväčšiť
* **−**: Zmenšiť
* **0** (Nula): Prispôsobiť obrazovke
* **Koliesko myši**: Zväčšiť/zmenšiť

### Ovládacie prvky zobrazenia

* **P**: Prepnúť režim percentuálneho zobrazenia pixelov
* **L**: Prepnúť panel vrstiev
* **Esc**: Zatvoriť celú obrazovku alebo vrátiť sa do prehliadača súborov

### Ostatné

* **Ctrl+S**: Uložiť aktuálny obrázok
* **F**: Režim celej obrazovky (ak je k dispozícii)

***

### Overenie výpočtov indexov

Skontrolujte, či sú indexy vypočítané správne:

1. Otvorte NDVI alebo iný indexový obrázok.
2. Skontrolujte oblasti vegetácie:
   * **NDVI**: Malo by sa zobraziť 0,4–0,9 pre zdravé rastliny.
   * **NDRE**: Vyššie hodnoty pre silný rast
   * **GNDVI**: Podobné ako NDVI, ale citlivé na chlorofyl
3. Skontrolujte oblasti bez vegetácie:
   * **Pôda**: Blízko 0 alebo mierne záporná
   * **Voda**: Záporné hodnoty (-0,5 až 0)

***

## Riešenie problémov so zobrazením

### Obraz sa neotvorí

**Možné príčiny:**

* Poškodený súbor počas spracovania
* Nepodporovaný formát súboru
* Nedostatočná pamäť pre veľký obraz

**Riešenia:**

1. Skúste otvoriť v externom prehliadači, aby ste overili integritu súboru.
2. Skontrolujte, či formát súboru zodpovedá očakávanému typu.
3. Zatvorte ostatné aplikácie, aby ste uvoľnili pamäť.
4. Skúste menší/iný obrázok.

### Čierny alebo biely obraz

**Možné príčiny:**

* Rozsah hodnôt mimo možnosti zobrazenia.
* 32-bitový plávajúci obrázok s neobvyklými hodnotami.
* Chyba výpočtu indexu.

**Riešenia:**

1. Skontrolujte hodnoty pixelov – ak sú všetky veľmi nízke alebo veľmi vysoké, upravte rozsah zobrazenia.
2. Skúste otvoriť v QGIS alebo podobnom programe s automatickým nastavením rozsahu.
3. Skontrolujte protokol ladenia zo spracovania, či neobsahuje chyby.

### Hodnoty pixelov sa zdajú nesprávne

**Možné príčiny:**

* Zobrazenie nesprávneho obrázku (originál vs. spracovaný)
* Kalibrácia nebola správne aplikovaná
* Dáta svetelného senzora neboli zahrnuté vo vstupe
* Režim percent bol nesprávne prepnutý

**Riešenia:**

1. Overte, či prezeráte spracovaný výstup (skontrolujte príponu súboru)
2. Skontrolujte stav tlačidla režimu percent
3. Porovnajte so známymi dobrými obrázkami z rovnakého súboru údajov

***

## Ďalšie kroky

Teraz, keď môžete zobraziť obrázky na celej obrazovke:

* [**Vrstvy obrázkov**](image-layers.md) – Informácie o vizualizácii viacerých pásiem
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Použite vlastné indexy a mapovanie farieb
* [**Vzorec multispektrálneho indexu**](../project-settings/multispectral-index-formulas.md) – Porozumenie dostupným indexom

Pre spracovanie pracovného postupu pozrite:

* [**Spracovanie obrázkov (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Kompletný sprievodca spracovaním

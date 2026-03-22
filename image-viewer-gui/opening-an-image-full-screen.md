# Otvorenie obrázku na celú obrazovku

Prehliadač obrázkov Chloros ponúka špecializované rozhranie na celú obrazovku určené na prezeranie, analýzu a úpravu vašich multispektrálnych obrázkov. Nech už prezeráte originálne obrázky alebo spracované výstupy, prehliadač obrázkov ponúka výkonné nástroje na kontrolu a analýzu.

## Prístup k prehliadaču obrázkov

### Z prehliadača súborov

Najbežnejší spôsob otvorenia obrázku v prehliadači obrázkov:

1. Uistite sa, že ste na karte **Prehliadač súborov** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Kliknite na ľubovoľnú **miniatúru snímky** v mriežke snímok
3. Snímka sa otvorí v **hlavnej oblasti náhľadu** (v strede obrazovky)
4. Snímka je teraz načítaná a pripravená na prezeranie na celej obrazovke

### Otvorenie karty prehliadača snímok

Akonáhle je snímka načítaná v oblasti náhľadu:

1. Kliknite na ikonu **Prehliadač snímok** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> v ľavom bočnom paneli
2. Otvorí sa karta Prehliadač obrázkov, ktorá zobrazí vybraný obrázok na celej obrazovke
3. V ľavom bočnom paneli sa sprístupnia pokročilé nástroje na prezeranie a analýzu

***

## Prehľad rozhrania prehliadača obrázkov

### Hlavná zobrazovacia oblasť

Najväčšia časť obrazovky zobrazuje váš obrázok:

* **Plné rozlíšenie**: Obrázky sa zobrazujú v natívnom rozlíšení
* **Možnosť priblíženia**: Na priblíženie použite ovládacie prvky alebo koliesko myši
* **Možnosť posúvania**: Po priblížení kliknite a ťahajte kurzorom na posúvanie
* **Zachovanie pomeru strán**: Obrázky sa proporcionálne zmenšujú alebo zväčšujú***

## Možnosti zobrazenia

### Základná navigácia v obrázkoch

#### Prechádzanie obrázkami

V sade obrázkov sa pohybujte pomocou klávesových skratiek alebo tlačidiel:

* **Ďalší obrázok**: Kliknite na tlačidlo → alebo stlačte klávesu**→** (šípka doprava)
* **Predchádzajúci obrázok**: Kliknite na tlačidlo ← alebo stlačte klávesu**←** (šípka doľava)
* **Preskočiť na konkrétny obrázok**: Vráťte sa do prehliadača súborov a kliknite na požadovanú miniatúru

#### Ovládacie prvky priblíženia

Upravte zväčšenie, aby ste si prezreli detaily obrázku:

**Zväčšenie:*** Kliknite na tlačidlo **+** (plus)
* Stlačte kláves **+**alebo**=*** Posuňte koliesko myši **nahor**

**Zmenšenie:*** Kliknite na tlačidlo **−** (mínus)
* Stlačte kláves **−** (mínus)
* Posuňte koliesko myši **nadol**

#### Posúvanie pri zväčšení

Pri zväčšení nad rámec veľkosti obrazovky:

1. Presuňte kurzor myši nad obrázok
2. Kliknite a **podržte ľavé tlačidlo myši**

3.**Ťahajte**, aby ste posúvali obrázok
4. Uvoľnite tlačidlo, aby ste zastavili posúvanie

**Alternatíva**: Použite šípky na posúvanie v malých krokoch***

## Kontrola hodnôt pixelov

### Zobrazenie hodnôt pixelov pri kurzore

Keď presúvate kurzor myši nad obrázkom, hodnoty pixelov sa zobrazujú v reálnom čase:**Umiestnenie zobrazenia hodnôt:*** **Plávajúce číslo a červená čiara v pravej časti legendy gradientu indexovej LUT*** **Pri ďalšom zväčšení plávajúca hodnota v blízkosti kurzora a zvýrazneného pixela*** Zobrazuje hodnoty pre pixel **pod kurzorom alebo zvýraznený*** Aktualizuje sa pri pohybe myši

***

## Typy obrázkov, ktoré môžete zobraziť

### JPG

**Obrázky JPG z fotoaparátu:**

* Zobrazenie údajov JPG ako v náhľade
* Zobrazenie pôvodných, neopravených hodnôt
* Užitočné na kontrolu kvality obrázku pred spracovaním

### RAW (pôvodný)

### RAW (odrazivosť)

**Po spracovaní:**

* Opravená vineta
* Kalibrovaná odrazivosť
* Viacpásmový TIFF (Red, Green, NIR atď.)
* Vedecké údaje pripravené na analýzu

### RAW (Index)

**NDVI, NDRE, GNDVI atď. (súbory \_NDVI.tif):**

* Jednopásmové snímky v odtieňoch šedej
* Hodnoty pixelov predstavujú výsledky výpočtu indexu
* Rozsah typicky od -1 do +1 pre normalizované indexy
* Na vizualizáciu je možné použiť farebné LUT

***

## Použitie indexov a LUT

Použite multispektrálne indexy a farebné vyhľadávacie tabuľky (LUT):

1. V **Prehliadači obrázkov**vyhľadajte**Index/LUT Sandbox** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> bočnom paneli
2. Vyberte index vegetácie (NDVI, NDRE atď.)
3. Vyberte multispektrálny vzorec alebo vytvorte vlastný (len Chloros+)
4. Použite farebný LUT gradient na vizualizáciu
5. Upravte rozsahy hodnôt a prahové hodnoty

Podrobné pokyny nájdete v [Index/LUT Sandbox](index-lut-sandbox.md).

***

## Klávesové skratky

### Navigácia

* **→** (šípka doprava): Ďalší obrázok
* **←** (šípka doľava): Predchádzajúci obrázok
* **Home**: Prvý obrázok v zozname
* **End**: Posledný obrázok v zozname

### Zväčšenie

* **+**alebo**=**: Zväčšiť
* **−**: Zmenšiť
* **Koleso myši**: Zväčšiť/zmenšiť***

### Overenie výpočtov indexov

Skontrolujte, či sú indexy vypočítané správne:

1. Otvorte NDVI alebo iný obrázok s indexom
2. Skontrolujte oblasti vegetácie:
   * **NDVI**: U zdravých rastlín by mala byť hodnota 0,4–0,9
   * **NDRE**: Vyššie hodnoty pri bujnom raste
   * **GNDVI**: Podobné ako NDVI, ale citlivé na chlorofyl
3. Skontrolujte neporast:
   * **Pôda**: Blízko 0 alebo mierne záporné
   * **Voda**: Záporné hodnoty (-0,5 až 0)***

## Riešenie problémov so zobrazením

### Obrázok sa neotvorí

**Možné príčiny:**

* Súbor bol poškodený počas spracovania
* Nepodporovaný formát súboru
* Nedostatočná pamäť pre veľký obrázok

**Riešenia:**

1. Skúste otvoriť v externom prehliadači, aby ste overili integritu súboru
2. Skontrolujte, či formát súboru zodpovedá očakávanému typu
3. Zatvorte ostatné aplikácie, aby ste uvoľnili pamäť
4. Skúste menší/iný obrázok

### Zobrazenie čierneho alebo bieleho obrázku

**Možné príčiny:**

* Rozsah hodnôt mimo zobrazovacích možností
* 32-bitový obrázok s neobvyklými hodnotami
* Chyba výpočtu indexu

**Riešenia:**

1. Skontrolujte hodnoty pixelov – ak sú všetky veľmi nízke alebo veľmi vysoké, upravte rozsah zobrazenia
2. Skúste otvoriť v QGIS alebo podobnom programe s automatickým nastavením rozsahu
3. Skontrolujte protokol ladenia zo spracovania, či neobsahuje chyby

### Hodnoty pixelov sa zdajú nesprávne

**Možné príčiny:**

* Prezeranie nesprávneho obrázku (originál vs. spracovaný)
* Kalibrácia nebola správne aplikovaná
* Údaje zo svetelného senzora neboli zahrnuté vo vstupe
* Režim percentuálneho zobrazenia bol nesprávne prepnutý

**Riešenia:**

1. Overte, či prezeráte spracovaný výstup (skontrolujte príponu súboru)
2. Skontrolujte stav tlačidla režimu percentuálneho zobrazenia
3. Porovnajte s obrázkami, o ktorých viete, že sú v poriadku, z rovnakého súboru údajov

***

## Ďalšie kroky

Teraz, keď môžete zobraziť obrázky na celej obrazovke:

* [**Vrstvy obrázkov**](image-layers.md) – Zoznámte sa s vizualizáciou viacerých pásiem
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Použite vlastné indexy a farebné mapovanie
* [**Vzorce multispektrálnych indexov**](../project-settings/multispectral-index-formulas.md) – Zoznámte sa s dostupnými indexmi

Informácie o pracovnom postupe spracovania nájdete tu:

* [**Spracovanie obrázkov (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Kompletný sprievodca spracovaním

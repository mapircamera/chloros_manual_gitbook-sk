# Nastavenia projektu

Bočný panel Nastavenia projektu <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v programe Chloros vám umožňuje konfigurovať všetky aspekty spracovania obrazu, detekciu kalibračného cieľa, výpočty multispektrálnych indexov a možnosti exportu pre váš projekt. Tieto nastavenia sa ukladajú spolu s vaším projektom a môžu sa uložiť ako šablóny na opätovné použitie vo viacerých projektoch.

## Prístup k nastaveniam projektu

Prístup k nastaveniam projektu:

1. Otvorte projekt v Chloros
2. Kliknite na kartu **Nastavenia projektu**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v ľavom bočnom paneli.
3. Na paneli nastavení sa zobrazia všetky dostupné možnosti konfigurácie usporiadané podľa kategórií.

***

## Detekcia cieľov

Tieto nastavenia ovládajú spôsob, akým Chloros detekuje a spracováva kalibračné ciele vo vašich obrázkoch.

### Minimálna plocha kalibračnej vzorky (px)

* **Typ**: Číslo
* **Rozsah**: 0 až 10 000 pixelov
* **Predvolené nastavenie**: 25 pixelov
* **Popis**: Nastavuje minimálnu plochu (v pixeloch) potrebnú na to, aby bola detegovaná oblasť považovaná za platnú vzorku kalibračného cieľa. Menšie hodnoty detegujú menšie ciele, ale môžu zvýšiť počet falošných pozitív. Väčšie hodnoty vyžadujú na detegáciu väčšie a jasnejšie cieľové oblasti.
* **Kedy upraviť**:
  * Zvýšte, ak sa vám zobrazujú falošné detekcie na malých artefaktoch obrázkov.
  * Znížte, ak sa vaše kalibračné ciele zobrazujú na obrázkoch ako malé a nie sú detegované.

### Minimálne zhlukovanie cieľov (0–100)

* **Typ**: Číslo
* **Rozsah**: 0 až 100
* **Predvolené nastavenie**: 60
* **Popis**: Ovláda prah zhlukovania pre zoskupovanie podobných farebných oblastí pri detekcii kalibračných cieľov. Vyššie hodnoty vyžadujú zoskupenie podobnejších farieb, čo vedie k konzervatívnejšej detekcii cieľov. Nižšie hodnoty umožňujú väčšiu farebnú variabilitu v rámci cieľovej skupiny.
* **Kedy nastaviť**:
  * Zvýšte, ak sa kalibračné ciele rozdeľujú na viacero detekcií.
  * Znížte, ak sa kalibračné ciele s farebnými variáciami nedetekujú úplne.

***

## Spracovanie

Tieto nastavenia riadia spôsob, akým Chloros spracováva a kalibruje vaše obrázky.

### Korekcia vinetácie

* **Typ**: Začiarkavacie políčko
* **Predvolené nastavenie**: Povolené (začiarknuté)
* **Popis**: Uplatňuje korekciu vinetácie na kompenzáciu stmavnutia objektívu na okrajoch obrázkov. Vinetácia je bežný optický jav, pri ktorom rohy a okraje obrázka vyzerajú tmavšie ako stred v dôsledku vlastností objektívu.
* **Kedy vypnúť**: Vypnite len v prípade, ak vaša kombinácia fotoaparátu a objektívu už použila korekciu vinetácie, alebo ak chcete vinetáciu manuálne korigovať v postprocesingu.

### Kalibrácia odrazivosti / vyváženie bielej

* **Typ**: Začiarkavacie políčko
* **Predvolené nastavenie**: Zapnuté (zaškrtnuté)
* **Popis**: Aktivuje automatickú kalibráciu odrazivosti pomocou detegovaných kalibračných cieľov vo vašich snímkach. Tým sa normalizujú hodnoty odrazivosti v celom súbore údajov a zabezpečia sa konzistentné merania bez ohľadu na svetelné podmienky.
* **Kedy vypnúť**: Vypnite len v prípade, ak chcete spracovávať surové, nekalibrované snímky alebo ak používate iný kalibračný pracovný postup.

### Metóda Debayer

* **Typ**: Rozbaľovacie menu
* **Možnosti**:
  * Vysoká kvalita (rýchlejšie) – v súčasnosti jediná dostupná možnosť
* **Predvolené nastavenie**: Vysoká kvalita (rýchlejšie)
* **Popis**: Vyberá algoritmus demosaicingu, ktorý sa používa na konverziu surových údajov snímača Bayerovho vzoru na farebné obrázky. Metóda „Vysoká kvalita (rýchlejšie)“ poskytuje optimálnu rovnováhu medzi rýchlosťou spracovania a kvalitou obrázkov.
* **Poznámka**: V budúcich verziách Chloros môžu byť pridané ďalšie metódy debayer.

### Minimálny interval rekalibrácie

* **Typ**: Číslo
* **Rozsah**: 0 až 3 600 sekúnd
* **Predvolené nastavenie**: 0 sekúnd
* **Popis**: Nastavuje minimálny časový interval (v sekundách) medzi použitím kalibračných cieľov. Ak je nastavená hodnota 0, Chloros použije každý zistený kalibračný cieľ. Ak je nastavená vyššia hodnota, Chloros použije len kalibračné ciele, ktoré sú od seba vzdialené aspoň o tento počet sekúnd, čím sa skráti čas spracovania pre súbory údajov s častým zachytávaním kalibračných cieľov.
* **Kedy upraviť**:
  * Nastavte na 0 pre maximálnu presnosť kalibrácie, ak sa menia svetelné podmienky.
  * Zvýšte (napr. na 60–300 sekúnd) pre rýchlejšie spracovanie, ak je osvetlenie konzistentné a máte časté obrazy kalibračných cieľov.

### Časový posun svetelného senzora

* **Typ**: Číslo
* **Rozsah**: -12 až +12 hodín
* **Predvolené nastavenie**: 0 hodín
* **Popis**: Určuje posun časového pásma (v hodinách od UTC) pre časové značky údajov svetelného senzora. Používa sa pri spracovaní súborov údajov PPK (Post-Processed Kinematic) na zabezpečenie správnej synchronizácie času medzi zachytenými snímkami a údajmi GPS.
* **Kedy nastaviť**: Nastavte na posun miestneho časového pásma, ak vaše údaje PPK používajú miestny čas namiesto UTC. Napríklad:
  * Tichomorský čas: -8 alebo -7 (v závislosti od letného času)
  * Východný čas: -5 alebo -4 (v závislosti od letného času)
  * Stredoeurópsky čas: +1 alebo +2 (v závislosti od letného času)

### Použiť korekcie PPK

* **Typ**: Začiarkavacie políčko
* **Predvolené nastavenie**: Vypnuté (nezačiarknuté)
* **Popis**: Umožňuje použitie korekcií Post-Processed Kinematic (PPK) z MAPIR DAQ záznamníkov obsahujúcich GPS (GNSS). Ak je táto funkcia povolená, Chloros použije všetky súbory protokolu .daq obsahujúce údaje o expozícií v adresári vášho projektu a uplatní presné korekcie geolokácie na vaše obrázky.
* **Požiadavka**: V adresári projektu musí byť prítomný log súbor .daq s údajmi o expozícií.
* **Kedy povoliť**: Odporúča sa vždy povoliť korekciu PPK, ak máte v log súbore .daq údaje o expozícii.

### Expozíciou pin 1

* **Typ**: Výber z roletového menu
* **Viditeľnosť**: Viditeľné len vtedy, ak je povolená funkcia „Použiť korekcie PPK“ A sú k dispozícii údaje o expozícii pre pin 1
* **Možnosti**:
  * Názvy modelov kamier zistené v projekte
  * „Nepoužívať“ – ignorovať tento expozičný pin
* **Predvolené nastavenie**: Automaticky vybrané na základe konfigurácie projektu
* **Popis**: Priraďuje konkrétnu kameru k expozičnému pinu 1 pre synchronizáciu času PPK. Expozičný pin zaznamenáva presný čas spustenia uzávierky fotoaparátu, čo je kritické pre presnú geolokalizáciu PPK.
* **Správanie automatického výberu**:
  * Jedna kamera + jeden pin: Automaticky vyberie kameru
  * Jedna kamera + dva piny: Pin 1 automaticky priradený k kamere
  * Viac kamier: Vyžaduje sa ručný výber

### Expozíciou pin 2

* **Typ**: Výber z roletového menu
* **Viditeľnosť**: Viditeľné len vtedy, ak je povolená funkcia „Použiť korekcie PPK“ A sú k dispozícii údaje o expozícii pre pin 2
* **Možnosti**:
  * Názvy modelov kamier zistené v projekte
  * „Nepoužívať“ – ignorovať tento pin expozície
* **Predvolené**: Automaticky vybrané na základe konfigurácie projektu
* **Popis**: Pri použití konfigurácie s dvoma kamerami priraďuje konkrétnu kameru k pinom expozície 2 na synchronizáciu času PPK.
* **Správanie automatického výberu**:
  * Jedna kamera + jeden pin: Pin 2 automaticky nastavený na „Nepoužívať“
  * Jedna kamera + dva piny: Pin 2 sa automaticky nastaví na „Nepoužívať“
  * Viac kamier: Vyžaduje sa ručný výber
* **Poznámka**: Tú istú kameru nemožno priradiť súčasne k pinu 1 aj k pinu 2.

***

## Index

Tieto nastavenia vám umožňujú konfigurovať multispektrálne indexy na analýzu a vizualizáciu.

### Pridať index

* **Typ**: Špeciálny panel konfigurácie indexov
* **Popis**: Otvorí interaktívny panel, kde môžete vybrať a nakonfigurovať multispektrálne vegetačné indexy (NDVI, NDRE, EVI atď.) na výpočet počas spracovania obrazu. Môžete pridať viacero indexov, každý s vlastnými nastaveniami vizualizácie.
* **Dostupné indexy**: Systém obsahuje viac ako 30 preddefinovaných multispektrálnych indexov, vrátane:
  * NDVI (normalizovaný rozdielový vegetačný index)
  * NDRE (normalizovaný rozdielový RedEdge)
  * EVI (vylepšený index vegetácie)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * A mnoho ďalších (úplný zoznam nájdete v časti [Vzorec multispektrálneho indexu](multispectral-index-formulas.md))
* **Funkcie**:
  * Výber z preddefinovaných vzorcov indexu
  * Konfigurácia farebných prechodov vizualizácie (LUT – vyhľadávacie tabuľky)
  * Nastavenie prahových hodnôt pre analýzu
  * Vytvorenie vlastných vzorcov indexu

### Vlastné vzorce (funkcia Chloros+)

* **Typ**: Pole definícií vlastných vzorcov
* **Popis**: Umožňuje vytvárať a ukladať vlastné multispektrálne indexové vzorce pomocou matematických operácií s pásmami. Vlastné vzorce sa ukladajú spolu s nastaveniami projektu a môžu sa používať rovnako ako vstavané indexy.
* **Ako vytvoriť**:
  1. V paneli konfigurácie indexu vyhľadajte možnosť vlastného vzorca.
  2. Definujte svoj vzorec pomocou identifikátorov pásiem (napr. NIR, Red, Green, Blue).
  3. Vzorec uložte pod popisným názvom.
* **Syntax vzorca**: Podporované sú štandardné matematické operácie, vrátane:
  * Aritmetika: `+`, `-`, `*`, `/`
  * Zátvorky pre poradie operácií
  * Odkazy na pásma: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Export

Tieto nastavenia ovládajú formát a kvalitu exportovaných spracovaných obrázkov.

### Kalibrovaný formát obrázku

* **Typ**: Výber z roletového menu
* **Možnosti**:
  * **TIFF (16-bit)** – Nekomprimovaný 16-bitový formát TIFF
  * **TIFF (32-bitový, percentuálny)** – 32-bitový formát TIFF s plávajúcou desatinnou čiarkou s hodnotami odrazivosti vyjadrenými v percentách
  * **PNG (8-bitový)** - Komprimovaný 8-bitový formát PNG
  * **JPG (8-bitový)** - Komprimovaný 8-bitový formát JPEG
* **Predvolené**: TIFF (16-bitový)
* **Popis**: Vyberá formát súboru na uloženie spracovaných a kalibrovaných obrázkov.
* **Odporúčania týkajúce sa formátu**:
  * **TIFF (16-bitový)**: Odporúča sa pre vedecké analýzy a profesionálne pracovné postupy. Zachováva maximálnu kvalitu údajov bez kompresných artefaktov. Najvhodnejší pre multispektrálnu analýzu a ďalšie spracovanie v softvéri GIS.
  * **TIFF (32-bitový, percentuálny)**: Najvhodnejší pre pracovné postupy, ktoré vyžadujú hodnoty odrazivosti v percentách (0–100 %). Ponúka maximálnu presnosť pre radiometrické merania.
  * **PNG (8-bitový)**: Vhodný pre prezeranie na webe a všeobecnú vizualizáciu. Menšie veľkosti súborov s bezstratovou kompresiou, ale znížený dynamický rozsah.
  * **JPG (8-bitový)**: Najmenšie veľkosti súborov, najvhodnejšie len pre náhľady a zobrazenie na webe. Používa stratovú kompresiu, ktorá nie je vhodná pre vedecké analýzy.

***

## Uložiť šablónu projektu

Táto funkcia vám umožňuje uložiť aktuálne nastavenia projektu ako šablónu, ktorú môžete opakovane používať.

* **Typ**: Zadávanie textu + tlačidlo Uložiť
* **Popis**: Zadajte popisný názov pre šablónu nastavení a kliknite na ikonu Uložiť. Šablóna uloží všetky aktuálne nastavenia projektu (detekcia cieľa, možnosti spracovania, indexy a formát exportu) pre jednoduché opätovné použitie v budúcich projektoch.
* **Prípady použitia**:
  * Vytvorte šablóny pre rôzne kamerové systémy (RGB, multispektrálny, NIR)
  * Uložte štandardné konfigurácie pre konkrétne typy plodín alebo pracovné postupy analýzy
  * Zdieľajte konzistentné nastavenia v rámci tímu
* **Ako používať**:
  1. Nakonfigurujte všetky požadované nastavenia projektu
  2. Zadajte názov šablóny (napr. „RedEdge Survey3 NDVI Standard“).
  3. Kliknite na ikonu Uložiť.
  4. Šablónu je teraz možné načítať pri vytváraní nových projektov.

***

## Uložiť priečinok projektu

Toto nastavenie určuje, kde sa nové projekty ukladajú ako predvolené.

* **Typ**: Zobrazenie cesty k adresáru + tlačidlo Upraviť
* **Predvolené**: `C:\Users\[Username]\Chloros Projects`
* **Popis**: Zobrazuje aktuálny predvolený adresár, kde sa vytvárajú nové projekty Chloros. Kliknutím na ikonu úpravy vyberte iný adresár.
* **Kedy zmeniť**:
  * Nastavte sieťový disk pre spoluprácu v tíme.
  * Zmeňte disk s väčším úložným priestorom pre veľké súbory údajov.
  * Usporiadajte projekty podľa roka, klienta alebo typu projektu do rôznych priečinkov.
* **Poznámka**: Zmena tohto nastavenia ovplyvní len NOVÉ projekty. Existujúce projekty zostanú na svojich pôvodných miestach.

***

## Trvalosť nastavení

Všetky nastavenia projektu sa automaticky uložia spolu s projektovým súborom (formát projektu `.mapir`). Keď projekt znovu otvoríte, všetky nastavenia sa obnovia presne tak, ako ste ich nechali.

### Hierarchia nastavení

Nastavenia sa uplatňujú v nasledujúcom poradí:

1. **Východiskové nastavenia systému** – Vstavané východiskové nastavenia definované Chloros
2. **Nastavenia šablóny** – ak pri vytváraní projektu načítajte šablónu
3. **Uložené nastavenia projektu** – nastavenia uložené spolu s projektovým súborom
4. **Ručné úpravy** – akékoľvek zmeny, ktoré vykonáte počas aktuálnej relácie

### Nastavenia a spracovanie obrázkov

Väčšina zmien nastavení (najmä v kategóriách Spracovanie a Export) spustí opätovné spracovanie obrázkov, aby sa zohľadnili nové nastavenia. Niektoré nastavenia sú však „len na export“ a nevyžadujú okamžité opätovné spracovanie:

* Uložiť šablónu projektu
* Pracovný adresár
* Kalibrovaný formát obrázku (platí pri exporte)

***

## Osvedčené postupy

1. **Začnite s predvolenými nastaveniami**: Predvolené nastavenia fungujú dobre pre väčšinu kamerových systémov MAPIR a typické pracovné postupy.
2. **Vytvorte šablóny**: Po optimalizácii nastavení pre konkrétny pracovný postup alebo kameru ich uložte ako šablónu, aby ste zabezpečili konzistentnosť medzi projektmi.
3. **Pred úplným spracovaním otestujte**: Pri experimentovaní s novými nastaveniami otestujte malú podmnožinu obrázkov pred spracovaním celého súboru údajov.
4. **Zaznamenajte svoje nastavenia**: Používajte popisné názvy šablón, ktoré označujú kamerový systém, typ spracovania a zamýšľané použitie (napr. „Survey3\_RGB\_NDVI\_Agriculture“).
5. **Výber formátu exportu**: Vyberte formát exportu na základe konečného použitia:
   * Vedecká analýza → TIFF (16-bitový alebo 32-bitový)
   * Spracovanie GIS → TIFF (16-bitový)
   * Rýchla vizualizácia → PNG (8-bitový)
   * Zdieľanie na webe → JPG (8-bitový)

***

Ďalšie informácie o multispektrálnych indexoch v Chloros nájdete na stránke [Vzorce multispektrálnych indexov](multispectral-index-formulas.md).

# Nastavenia projektu

Bočný panel „Nastavenia projektu“ <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v Chloros vám umožňuje konfigurovať všetky aspekty spracovania obrazu, detekcie kalibračných cieľov, výpočtov multispektrálnych indexov a možností exportu pre váš projekt. Tieto nastavenia sa ukladajú spolu s vaším projektom a môžu sa uložiť ako šablóny na opätovné použitie vo viacerých projektoch.

## Prístup k nastaveniam projektu

Prístup k nastaveniam projektu:

1. Otvorte projekt v Chloros
2. Kliknite na kartu **Nastavenia projektu**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v ľavom bočnom paneli
3. V paneli nastavení sa zobrazia všetky dostupné možnosti konfigurácie zoradené podľa kategórií

***

## Detekcia cieľov

Tieto nastavenia ovplyvňujú spôsob, akým Chloros detekuje a spracováva kalibračné ciele vo vašich snímkach.

### Minimálna plocha kalibračnej vzorky (px)

* **Typ**: Číslo
* **Rozsah**: 0 až 10 000 pixelov
* **Predvolené**: 25 pixelov
* **Popis**: Nastavuje minimálnu plochu (v pixeloch) potrebnú na to, aby bola detegovaná oblasť považovaná za platnú vzorku kalibračného cieľa. Menšie hodnoty detegujú menšie ciele, ale môžu zvýšiť počet falošných pozitív. Väčšie hodnoty vyžadujú väčšie a jasnejšie oblasti cieľov na detekciu.
* **Kedy nastaviť**:
  * Zvýšte, ak dochádza k falošným detekciám na malých artefaktoch v obrazoch
  * Znížte, ak sa vaše kalibračné ciele v obrazoch javia ako malé a nie sú detekované

### Minimálne zhlukovanie cieľov (0–100)

* **Typ**: Číslo
* **Rozsah**: 0 až 100
* **Predvolené nastavenie**: 60
* **Popis**: Ovláda prah zhlukovania pre zoskupovanie podobne sfarbených oblastí pri detekcii kalibračných cieľov. Vyššie hodnoty vyžadujú zoskupenie väčšieho počtu podobných farieb, čo vedie k konzervatívnejšej detekcii cieľov. Nižšie hodnoty umožňujú väčšiu farebnú variabilitu v rámci skupiny cieľov.
* **Kedy nastaviť**:
  * Zvýšte, ak sa kalibračné ciele rozdeľujú na viacero detekcií
  * Znížte, ak sa kalibračné ciele s farebnou variabilitou nedetekujú úplne

***

## Spracovanie

Tieto nastavenia riadia spôsob, akým Chloros spracováva a kalibruje vaše snímky.

### Korekcia vinetácie

* **Typ**: Zaškrtávacie políčko
* **Predvolené nastavenie**: Zapnuté (zaškrtnuté)
* **Popis**: Použije korekciu vinetácie na kompenzáciu stmavnutia objektívu na okrajoch obrázkov. Vinetácia je bežný optický jav, pri ktorom sa rohy a okraje obrázku javia tmavšie ako stred v dôsledku vlastností objektívu.
* **Kedy vypnúť**: Vypnite len v prípade, ak vaša kombinácia fotoaparátu a objektívu už použila korekciu vinetácie, alebo ak chcete vinetáciu korigovať ručne v post-spracovaní.

### Kalibrácia odrazivosti / vyváženie bielej

* **Typ**: Zaškrtávacie políčko
* **Predvolené nastavenie**: Zapnuté (zaškrtnuté)
* **Popis**: Zapína automatickú kalibráciu odrazivosti pomocou detegovaných kalibračných cieľov vo vašich snímkach. Tým sa normalizujú hodnoty odrazivosti v celom vašom súbore údajov a zabezpečia sa konzistentné merania bez ohľadu na svetelné podmienky.
* **Kedy vypnúť**: Vypnite len vtedy, ak chcete spracovávať surové, nekalibrované snímky, alebo ak používate iný kalibračný pracovný postup.

### Metóda debayeringu

* **Typ**: Rozbaľovacie menu
* **Možnosti**:
  * Štandardná (rýchla, stredná kvalita)
  * S ohľadom na textúru (pomalá, najvyššia kvalita) \[Chloros+]
* **Predvolené**: Štandardné (rýchle, stredná kvalita)
* **Popis**: Vyberá algoritmus demosaicingu používaný na konverziu surových údajov zo senzora s Bayerovým vzorom na plnofarebné snímky. Metóda „Štandardné (rýchle, stredná kvalita)“ poskytuje optimálnu rovnováhu medzi rýchlosťou spracovania a kvalitou snímky. Metóda „S ohľadom na textúru (pomalé, najvyššia kvalita)“ \[Chloros+] používa vysoko kvalitný algoritmus demosaicingu zohľadňujúci hrany v kombinácii s modelom odšumovania AI/ML, ktorý odstraňuje takmer všetok šum spôsobený demosaicingom. Model zohľadňujúci textúru vyžaduje na spustenie pamäť GPU (VRAM). Odporúčame ho používať, ak máte k dispozícii &gt;4 GB VRAM pre rýchlejšie spracovanie.
* **Poznámka**: V budúcich verziách Chloros môžu byť pridané ďalšie metódy odstraňovania šumu.

### Minimálny interval rekalibrácie

* **Typ**: Číslo
* **Rozsah**: 0 až 3 600 sekúnd
* **Predvolené**: 0 sekúnd
* **Popis**: Nastavuje minimálny časový interval (v sekundách) medzi použitím kalibračných cieľov. Ak je nastavené na 0, Chloros použije každý detekovaný kalibračný cieľ. Ak je nastavené na vyššiu hodnotu, Chloros použije len kalibračné ciele, ktoré sú od seba vzdialené aspoň o tento počet sekúnd, čím sa skráti čas spracovania pre dátové súbory s častým zachytením kalibračných cieľov.
* **Kedy nastaviť**:
  * Nastavte na 0 pre maximálnu presnosť kalibrácie, ak sa menia svetelné podmienky
  * Zvýšte (napr. na 60–300 sekúnd) pre rýchlejšie spracovanie, ak je osvetlenie konzistentné a máte časté snímky kalibračných cieľov

### Časový posun svetelného senzora

* **Typ**: Číslo
* **Rozsah**: -12 až +12 hodín
* **Predvolené nastavenie**: 0 hodín
* **Popis**: Určuje časové posunutie (v hodinách od UTC) pre časové značky údajov svetelného senzora. Používa sa pri spracovaní súborov údajov PPK (Post-Processed Kinematic) na zabezpečenie správnej časovej synchronizácie medzi zachytenými snímkami a údajmi GPS.
* **Kedy nastaviť**: Nastavte to na časové posunutie vášho miestneho časového pásma, ak vaše údaje PPK používajú miestny čas namiesto UTC. Napríklad:
  * Tichomorský čas: -8 alebo -7 (v závislosti od letného času)
  * Východný čas: -5 alebo -4 (v závislosti od letného času)
  * Stredoeurópsky čas: +1 alebo +2 (v závislosti od letného času)

### Použiť korekcie PPK

* **Typ**: Zaškrtávacie políčko
* **Predvolené nastavenie**: Vypnuté (nezaškrtnuté)
* **Popis**: Umožňuje použitie korekcií Post-Processed Kinematic (PPK) z DAQ záznamníkov MAPIR obsahujúcich GPS (GNSS). Ak je táto funkcia povolená, Chloros použije všetky súbory protokolu .daq obsahujúce údaje o expozícii v adresári vášho projektu a na vaše snímky uplatní presné korekcie geolokácie.
* **Požiadavka**: V adresári projektu musí byť prítomný súbor protokolu .daq so záznamami o expozícii
* **Kedy povoliť**: Odporúča sa vždy povoliť korekciu PPK, ak máte v súbore protokolu .daq záznamy o spätnej väzbe expozície.

### Expozícia Pin 1

* **Typ**: Výber z roletového menu
* **Viditeľnosť**: Viditeľné len vtedy, ak je povolená funkcia „Použiť korekcie PPK“ A sú k dispozícii údaje o expozícii pre Pin 1
* **Možnosti**:
  * Názvy modelov fotoaparátov zistené v projekte
  * „Nepoužívať“ – Ignorovať tento expozíciou pin
* **Predvolené**: Automaticky vybrané na základe konfigurácie projektu
* **Popis**: Priradí konkrétnu kameru k konektoru expozície 1 na účely časovej synchronizácie PPK. Konektor expozície zaznamenáva presný čas spustenia uzávierky kamery, čo je kľúčové pre presnú geolokalizáciu PPK.
* **Správanie automatického výberu**:
  * Jedna kamera + jeden konektor: Automaticky vyberie kameru
  * Jedna kamera + dva piny: Pin 1 sa automaticky priradí k kamere
  * Viac kamier: Vyžaduje sa ručný výber

### Expozíciový pin 2

* **Typ**: Výber z roletového menu
* **Viditeľnosť**: Viditeľné len vtedy, ak je povolená možnosť „Použiť korekcie PPK“ A sú k dispozícii údaje o expozícii pre pin 2
* **Možnosti**:
  * Názvy modelov kamier zistené v projekte
  * „Nepoužívať“ – tento pin expozície ignorovať
* **Predvolené**: Automaticky vybrané na základe konfigurácie projektu
* **Popis**: Priradí konkrétnu kameru k pinu expozície 2 na synchronizáciu času PPK pri použití konfigurácie s dvoma kamerami.
* **Správanie automatického výberu**:
  * Jedna kamera + jeden pin: Pin 2 sa automaticky nastaví na „Nepoužívať“
  * Jedna kamera + dva piny: Pin 2 sa automaticky nastaví na „Nepoužívať“
  * Viac kamier: Vyžaduje sa ručný výber
* **Poznámka**: Tú istú kameru nemožno priradiť súčasne k pinom 1 aj 2.***

## Index

Tieto nastavenia vám umožňujú konfigurovať multispektrálne indexy na analýzu a vizualizáciu.

### Pridať index

* **Typ**: Špeciálny panel konfigurácie indexov
* **Popis**: Otvorí interaktívny panel, kde môžete vybrať a nakonfigurovať multispektrálne vegetačné indexy (NDVI, NDRE, EVI atď.) na výpočet počas spracovania snímok. Môžete pridať viacero indexov, každý s vlastnými nastaveniami vizualizácie.
* **Dostupné indexy**: Systém obsahuje viac ako 30 preddefinovaných multispektrálnych indexov, vrátane:
  * NDVI (Normalizovaný rozdielový vegetačný index)
  * NDRE (Normalizovaný rozdiel RedEdge)
  * EVI (Vylepšený index vegetácie)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * A mnoho ďalších (kompletný zoznam nájdete v [Vzorce multispektrálnych indexov](multispectral-index-formulas.md))
* **Funkcie**:
  * Výber z preddefinovaných vzorcov indexov
  * Konfigurácia farebných prechodov vizualizácie (LUT – vyhľadávacie tabuľky)
  * Nastavenie prahových hodnôt pre analýzu
  * Vytvorenie vlastných vzorcov indexov

### Vlastné vzorce (Funkcia Chloros+)

* **Typ**: Pole definícií vlastných vzorcov
* **Popis**: Umožňuje vytvárať a ukladať vlastné multispektrálne indexové vzorce pomocou matematických operácií s pásmami. Vlastné vzorce sa ukladajú spolu s nastaveniami projektu a dajú sa používať rovnako ako vstavané indexy.
* **Ako vytvoriť**:
  1. V paneli konfigurácie indexu vyhľadajte možnosť vlastného vzorca
  2. Definujte svoj vzorec pomocou identifikátorov pásiem (napr. NIR, Red, Green, Blue)
  3. Uložte vzorec pod výstižným názvom
* **Syntax vzorca**: Podporované sú štandardné matematické operácie, vrátane:
  * Aritmetika: `+`, `-`, `*`, `/`
  * Zátvorky pre poradie operácií
  * Odkazy na pásma: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Export

Tieto nastavenia ovplyvňujú formát a kvalitu exportovaných spracovaných obrázkov.

### Formát kalibrovaného obrázku

* **Typ**: Výber z roletového menu
* **Možnosti**:
  * **TIFF (16-bitový)** – Nekomprimovaný 16-bitový formát TIFF
  * **TIFF (32-bitový, percentuálny)** – 32-bitový formát TIFF s plávajúcou desatinnou čiarkou a hodnotami odrazivosti vyjadrenými v percentách
  * **PNG (8-bitový)** - Komprimovaný 8-bitový formát PNG
  * **JPG (8-bitový)** - Komprimovaný 8-bitový formát JPEG
* **Predvolené**: TIFF (16-bitový)
* **Popis**: Vyberá formát súboru na ukladanie spracovaných a kalibrovaných snímok.
* **Odporúčania týkajúce sa formátu**:
  * **TIFF (16-bitový)**: Odporúča sa pre vedeckú analýzu a profesionálne pracovné postupy. Zachováva maximálnu kvalitu údajov bez kompresných artefaktov. Najvhodnejší pre multispektrálnu analýzu a ďalšie spracovanie v softvéri GIS.
  * **TIFF (32-bitový, percentuálny)**: Najvhodnejší pre pracovné postupy, ktoré vyžadujú hodnoty odrazivosti v percentách (0–100 %). Ponúka maximálnu presnosť pre radiometrické merania.
  * **PNG (8-bitový)**: Vhodný pre prezeranie na webe a všeobecnú vizualizáciu. Menšie veľkosti súborov s bezstratovou kompresiou, ale zníženým dynamickým rozsahom.
  * **JPG (8-bit)**: Najmenšie veľkosti súborov, najlepšie len pre náhľady a zobrazenie na webe. Používa stratovú kompresiu, ktorá nie je vhodná pre vedeckú analýzu.***

## Uložiť šablónu projektu

Táto funkcia vám umožňuje uložiť aktuálne nastavenia projektu ako šablónu, ktorú môžete opakovane používať.

* **Typ**: Zadávanie textu + tlačidlo Uložiť
* **Popis**: Zadajte popisný názov pre šablónu nastavení a kliknite na ikonu uloženia. Šablóna uloží všetky vaše aktuálne nastavenia projektu (detekcia cieľa, možnosti spracovania, indexy a formát exportu) pre jednoduché opätovné použitie v budúcich projektoch.
* **Príklady použitia**:
  * Vytvorte šablóny pre rôzne kamerové systémy (RGB, multispektrálny, NIR)
  * Uložte štandardné konfigurácie pre konkrétne typy plodín alebo pracovné postupy analýzy
  * Zdieľajte konzistentné nastavenia v rámci tímu
* **Ako používať**:
  1. Nakonfigurujte všetky požadované nastavenia projektu
  2. Zadajte názov šablóny (napr. „RedEdge Survey3 NDVI Štandardná“)
  3. Kliknite na ikonu uloženia
  4. Šablónu je teraz možné načítať pri vytváraní nových projektov

***

## Uložiť priečinok projektu

Toto nastavenie určuje, kam sa nové projekty štandardne ukladajú.

* **Typ**: Zobrazenie cesty k adresáru + tlačidlo Upraviť
* **Predvolené (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Predvolené (Linux)**: `~/.local/share/chloros/projects`
* **Popis**: Zobrazuje aktuálny predvolený adresár, kde sa vytvárajú nové projekty Chloros. Kliknutím na ikonu úpravy vyberte iný adresár.
* **Kedy zmeniť**:
  * Nastavte na sieťovú jednotku pre spoluprácu v tíme
  * Zmeňte na disk s väčším úložným priestorom pre veľké súbory údajov
  * Usporiadajte projekty podľa roka, klienta alebo typu projektu do rôznych zložiek
* **Poznámka**: Zmena tohto nastavenia ovplyvní len NOVÉ projekty. Existujúce projekty zostanú na svojich pôvodných miestach.***

## Trvalosť nastavení

Všetky nastavenia projektu sa automaticky ukladajú spolu s vaším projektovým súborom (formát projektu `.mapir`). Keď projekt znovu otvoríte, všetky nastavenia sa obnovia presne tak, ako ste ich nechali.

### Hierarchia nastavení

Nastavenia sa uplatňujú v nasledujúcom poradí:

1. **Systémové predvolby** – Vstavané predvolby definované v Chloros
2. **Nastavenia šablóny** – Ak pri vytváraní projektu načítajte šablónu
3. **Uložené nastavenia projektu** – Nastavenia uložené spolu so súborom projektu
4. **Ručné úpravy** – Akékoľvek zmeny, ktoré vykonáte počas aktuálnej relácie

### Nastavenia a spracovanie obrázkov

Väčšina zmien nastavení (najmä v kategóriách Spracovanie a Export) spustí opätovné spracovanie obrázkov, aby odzrkadľovali nové nastavenia. Niektoré nastavenia sú však určené „len na export“ a nevyžadujú okamžité opätovné spracovanie:

* Uložiť šablónu projektu
* Pracovný adresár
* Kalibrovaný formát obrázkov (platí pri exporte)

***

## Osvedčené postupy

1. **Začnite s predvolenými nastaveniami**: Predvolené nastavenia fungujú dobre pre väčšinu kamerových systémov MAPIR a typických pracovných postupov.
2. **Vytvorte šablóny**: Akonáhle optimalizujete nastavenia pre konkrétny pracovný postup alebo kameru, uložte ich ako šablónu, aby ste zabezpečili konzistentnosť medzi projektmi.
3. **Otestujte pred úplným spracovaním**: Pri experimentovaní s novými nastaveniami otestujte na malej podmnožine obrázkov pred spracovaním celého súboru údajov.
4. **Zaznamenajte si svoje nastavenia**: Používajte popisné názvy šablón, ktoré uvádzajú kamerový systém, typ spracovania a zamýšľané použitie (napr. „Survey3\_RGB\_NDVI\_Agriculture“).
5. **Výber formátu exportu**: Vyberte formát exportu na základe konečného použitia:
   * Vedecká analýza → TIFF (16-bitový alebo 32-bitový)
   * Spracovanie GIS → TIFF (16-bitový)
   * Rýchla vizualizácia → PNG (8-bitový)
   * Zdieľanie na webe → JPG (8-bitový)

***

Ďalšie informácie o multispektrálnych indexoch v Chloros nájdete na stránke [Vzorce multispektrálnych indexov](multispectral-index-formulas.md).

# Obrazové vrstvy

Rozbaľovacie menu „Obrazové vrstvy“ v prehliadači obrázkov Chloros vám umožňuje rýchlo prepínať medzi rôznymi verziami toho istého obrázku – od pôvodných snímok až po spracované výstupy odrazivosti a vypočítané indexové obrázky.

## Čo sú vrstvy obrázkov?

V programe Chloros sa **vrstvy** vzťahujú na rôzne výstupy obrázkov, ktoré sú k dispozícii pre jeden zdrojový obrázok. Pri spracovaní obrázkov program Chloros vytvorí viacero verzií:

* **Pôvodné obrázky** (súbory JPG a RAW z vášho fotoaparátu)
* **Výstupy s kalibrovanou odrazivosťou** (ak bola povolená kalibrácia odrazivosti)
* **Cieľové obrázky** (ak obrázok obsahuje kalibračné ciele)
* **Indexové obrázky** (NDVI, NDRE, GNDVI atď., ak boli nakonfigurované indexy)**Rozbaľovacie menu Výber vrstvy** v pravom hornom rohu prehliadača obrázkov vám umožňuje okamžite prepínať medzi týmito verziami bez opustenia prehliadača.***

## Dostupné typy vrstiev

### JPG

* Pôvodný náhľadový obrázok vo formáte JPG z vášho fotoaparátu
* Vždy k dispozícii pre všetky obrázky
* Nespracovaný, tak ako bol zachytený fotoaparátom
* Najrýchlejšie sa načíta a zobrazí

**Kedy zobraziť:**

* Rýchly náhľad pôvodného záberu
* Kontrola kompozície a rámovania snímky
* Overenie kvality záberu pred spracovaním

### RAW (pôvodný)

* Pôvodné údaje zo senzora vo formáte RAW z vášho fotoaparátu
* Odstránené prekrývanie bez použitia následného spracovania
* Vyššia bitová hĺbka ako pri JPG (zvyčajne 12-bitové alebo 14-bitové údaje zo senzora)

**Kedy zobraziť:**

* Kontrola kvality pôvodných údajov zo senzora
* Kontrola problémov so snímačom alebo artefaktov
* Porovnanie výsledkov pred a po spracovaní

### RAW (Cieľ)

* Zobrazuje sa len pri snímkach identifikovaných ako obsahujúce kalibračné ciele
* Zobrazuje pôvodný RAW snímok s detegovaným cieľom
* Slúži na overenie, či bola detekcia cieľa úspešná

**Kedy zobraziť:**

* Potvrdenie, že kalibračné ciele boli správne detekované
* Kontrola kvality snímky cieľa
* Riešenie problémov s kalibráciou

{% hint style="info" %}
**Vrstva cieľa**: Táto vrstva sa zobrazuje v roletovom menu iba pri snímkach, ktoré obsahujú kalibračné ciele. Bežné snímky túto možnosť nemajú.
{% endhint %}

### RAW (Odrazivosť)

* Kalibrovaný výstupný obrázok odrazivosti
* S korekciou vinetácie (ak je povolená pri spracovaní)
* Odrazivosť kalibrovaná pomocou údajov cieľa (ak je povolená)
* Viacpásmový TIFF so všetkými kanálmi kamery
* Hodnoty pixelov predstavujú percentuálnu odrazivosť (pri použití percentuálneho režimu)
* Pripravené na manipuláciu s [Index/LUT Sandbox](index-lut-sandbox.md)

**Kedy zobraziť:**

* Kontrola kalibrovaných výsledkov
* Overenie kvality kalibrácie
* Kontrola hodnôt pixelov z hľadiska vedeckej presnosti
* Porovnanie s originálom na posúdenie efektov kalibrácie

{% hint style="success" %}
**Odporúčané**: Pri kontrole hodnôt pixelov pre vedecké merania a analýzy použite vrstvu RAW (Reflectance).
{% endhint %}

### RAW (NDVI Index)... a podobné

* Vypočítaný obraz vegetačného indexu (v tomto príklade NDVI)
* Názov indexu sa mení v závislosti od toho, ktorý index bol nakonfigurovaný počas spracovania
* Príklady: RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI Index) atď.
* Jednopásmový obrázok v odtieňoch šedej zobrazujúci výsledky výpočtu indexu
* Pre každý index nakonfigurovaný v nastaveniach projektu sa zobrazí jedna vrstva

**Možné názvy indexov:**

* RAW (NDVI Index)
* RAW (NDRE Index)
* RAW (GNDVI Index)
* RAW (OSAVI Index)
* RAW (EVI Index)
* RAW (SAVI Index)
* A mnoho ďalších... (pozri [Vzorce multispektrálnych indexov](../project-settings/multispectral-index-formulas.md))

**Kedy zobraziť:**

* Preskúmanie výsledkov výpočtu indexu
* Kontrola rozsahov hodnôt indexu
* Identifikácia oblastí záujmu
* Overenie indexových snímok pred použitím v GIS alebo analýze

***

## Používanie výberu vrstiev

### Otvorenie roletového menu

1. Otvorte snímku v režime celej obrazovky (kliknite na ľubovoľnú miniatúru v prehliadači snímok)
2. Vyhľadajte **roztváraciu ponuku vrstiev** v pravom hornom rohu prehliadača
3. Roztváracia ponuka zobrazuje aktuálne vybranú vrstvu (napr. „JPG“)
4. Kliknite na roztváraciu ponuku, aby ste videli všetky dostupné vrstvy

### Prepínanie vrstiev

1. Kliknite na roztváraciu ponuku vrstiev, aby ste otvorili zoznam
2. Zobrazia sa všetky dostupné vrstvy pre aktuálny obrázok
3. Kliknite na akýkoľvek názov vrstvy, aby ste prešli na túto verziu
4. Obrázok sa okamžite aktualizuje a zobrazí vybranú vrstvu

**Rýchle prepínanie:**

* Rozbaľovacie menu si pamätá váš posledný výber
* Pri prechode na ďalší obrázok sa Chloros pokúsi zobraziť rovnaký typ vrstvy
* Ak táto vrstva na ďalšom obrázku neexistuje, predvolene sa nastaví na JPG

### Dostupnosť vrstiev

Nie všetky vrstvy sú k dispozícii pre každý obrázok:

**Vždy k dispozícii:*** ✅ JPG (každý obrázok má náhľad vo formáte JPG)

**Podmienečne k dispozícii:**

* ⚠️ RAW (Originál) – Iba ak bol obrázok nasnímaný v režime RAW alebo RAW+JPG
* ⚠️ RAW (Cieľ) – Iba ak obrázok obsahuje detegované kalibračné ciele
* ⚠️ RAW (Reflectance) – Iba po spracovaní s povolenou kalibráciou odrazivosti
* ⚠️ RAW (\[Index] Index) – Iba po spracovaní s nakonfigurovanými indexmi

***

## Trvácnosť vrstiev

### Prechádzanie medzi obrázkami

Keď prejdete na iný obrázok (pomocou klávesov so šípkami alebo kliknutím na miniatúry):**Predvoľba vrstvy zostane zachovaná:**

* Ak prezeráte „RAW (Reflectance)“, ďalší obrázok zobrazí „RAW (Reflectance)“ (ak je k dispozícii)
* Ak prezeráte „RAW (NDVI Index)“, ďalší obrázok zobrazí „RAW (NDVI Index)“ (ak je k dispozícii)
* Ak tá istá vrstva neexistuje, predvolene sa nastaví JPG

**Príklad pracovného postupu:**

1. Otvorte obrázok 1, prepnite na RAW (NDVI Index)
2. Stlačte → pre zobrazenie obrázku 2
3. Obrázok 2 automaticky zobrazí vrstvu RAW (NDVI Index)
4. Pokračujte v navigácii – všetky obrázky zobrazujú vrstvu NDVI
5. Veľmi efektívne pre prezeranie výsledkov indexovania na mnohých obrázkoch

***

## Bežné pracovné postupy

### Pracovný postup 1: Porovnanie pred/po

**Cieľ**: Porovnať pôvodný a kalibrovaný obrázok

1. Otvorte spracovaný obrázok v prehliadači obrázkov
2. Z roletového menu vyberte **RAW (Original)**

3. Všímajte si vinetáciu a nekalibrované hodnoty
4. Z roletového menu prejdite na **RAW (Reflectance)**

5. Porovnajte – vinetácia odstránená, hodnoty kalibrované

### Pracovný postup 2: Prehľad indexu

**Cieľ**: Rýchlo skontrolujte výsledky NDVI v celom súbore údajov

1. Otvorte prvý spracovaný obrázok
2. Z roletového menu vyberte **RAW (NDVI Index)**

3. Pomocou šípky → prejdite na ďalší obrázok
4. Vrstva NDVI sa automaticky zachová
5. Pokračujte cez všetky obrázky a kontrolujte vzory NDVI
6. Prepnite na **RAW (NDRE Index)** na porovnanie

### Pracovný postup 3: Overenie cieľa

**Cieľ**: Overiť, či boli všetky cieľové obrázky správne detekované

1. Prejdite na cieľový obrázok
2. Z roletového menu vyberte **RAW (Cieľ)**

3. Overte, či sú kalibračné ciele jasne viditeľné a detekované
4. Prejdite na ďalší cieľový obrázok
5. Opakujte overenie pre všetky ciele

### Pracovný postup 4: Kontrola hodnôt pixelov

**Cieľ**: Skontrolujte hodnoty odrazivosti z hľadiska vedeckej presnosti

1. Otvorte spracovaný obrázok
2. Vyberte vrstvu **RAW (Reflectance)**

3. Zapnite režim**Pixel Percent** (tlačidlo v pravom hornom rohu panela nástrojov)
4. Presuňte kurzor nad oblasti vegetácie
5. Overte, či sú hodnoty pixelov v očakávaných rozsahoch (30–70 % pre NIR, 5–15 % pre Red)
6. Skontrolujte, či oblasti pôdy a vody majú správne hodnoty

***

## Porozumenie hodnotám pixelov podľa vrstvy

Rôzne vrstvy zobrazujú rôzne rozsahy hodnôt pixelov:

### Vrstva JPG

* **Rozsah**: 0–255 (8-bitový)
* **Význam**: Zobrazené hodnoty, s korekciou gama
* **Použitie**: Iba vizuálna kontrola, nie na vedecké meranie

### RAW (pôvodný)

* **Rozsah**: 0–65535 (16-bitový)
* **Význam**: Surové digitálne hodnoty snímača
* **Použitie**: Kontrola výkonu snímača, nekalibrované

### RAW (odrazivosť)

* **Rozsah**: 0–65 535 (16-bitový TIFF) alebo 0,0–1,0 (32-bitové percento)
* **Význam**: Kalibrovaná percentuálna odrazivosť
* **Použitie**: Vedecké merania a analýzy**Pre 16-bitový TIFF:**Vydelením číslom 65 535 získate percentuálnu odrazivosť**Pre 32-bitový percentuálny údaj:** Hodnoty priamo predstavujú percentá (0,5 = 50 % odrazivosť)

### RAW (indexové snímky)

* **Rozsah**: Líši sa podľa indexu (zvyčajne -1,0 až +1,0 pre normalizované indexy)
* **Význam**: Výsledok výpočtu indexu
* **Príklady**:
  * NDVI: -1 až +1 (vegetácia zvyčajne 0,4 až 0,9)
  * NDRE: -1 až +1 (detekcia stresu)
  * EVI: 0 až 1 (vylepšená vegetácia)

***

## Tipy a osvedčené postupy

### Efektívne prepínanie vrstiev

* **Informácie o klávesových skratkách**: Hoci pre vrstvy neexistujú klávesové skratky, navigačné šípky (←/→) fungujú vo všetkých vrstvách
* **Konzistentné pracovné postupy**: Vyberte jednu vrstvu (napr. NDVI) a skontrolujte celý súbor údajov, než prejdete na inú
* **Rýchle porovnania**: Prepínajte medzi vrstvami Original a Reflectance, aby ste overili kvalitu spracovania

### Úvahy o výkone

* **JPG sa načíta najrýchlejšie**: Použite na rýchlu navigáciu medzi mnohými obrázkami
* **Vrstvy RAW sa načítajú pomalšie**: Vyššie rozlíšenie a bitová hĺbka
* **Indexové vrstvy**: Podobná rýchlosť ako vrstvy odrazivosti
* **Prvé načítanie je najpomalšie**: Nasledujúce zobrazenia tej istej vrstvy sú uložené v cache a sú rýchlejšie

### Overenie kvality

* **Vždy skontrolujte RAW (pôvodné)**: Overte kvalitu zdrojových údajov, než budete dôverovať spracovaným výstupom
* **Porovnajte vrstvy**: Použite prepínanie vrstiev na overenie, či spracovanie prebehlo správne
* **Skontrolujte rozsahy indexu**: Použite režim Pixel Percent s indexovými vrstvami na overenie, či sú hodnoty primerané***

## Riešenie problémov

### Vrstva nie je k dispozícii

**Problém**: Očakávaná vrstva sa nezobrazuje v roletovom menu**Možné príčiny:**

* Obraz nebol spracovaný (k dispozícii sú len formáty JPG a RAW (originál))
* Kalibrácia odrazivosti bola počas spracovania deaktivovaná
* Konkrétny index nebol nakonfigurovaný v nastaveniach projektu
* Obraz je obrazom len pre ciele (pre ciele sa negenerujú žiadne indexy)

**Riešenia:**

1. Overte, či bol obraz spracovaný (skontrolujte výstupnú zložku, či obsahuje spracované súbory)
2. Skontrolujte nastavenia projektu, aby ste sa uistili, že boli nakonfigurované indexy
3. Spracujte znovu s povolenými požadovanými indexmi

### Zobrazená nesprávna vrstva

**Problém**: Obrázok sa otvorí v neočakávanej vrstve**Príčina**: Predvolená vrstva z predchádzajúceho obrázku bola prenesená, ale táto vrstva na aktuálnom obrázku neexistuje**Riešenie**: Chloros automaticky prejde na formát JPG, ak nie je k dispozícii preferovaná vrstva – ide o normálne správanie

### Nevidím kalibračné ciele

**Problém**: Vrstva RAW (Cieľ) nezobrazuje detekciu cieľov**Možné príčiny:**

* Ciele neboli detekované počas spracovania
* Obraz v skutočnosti neobsahuje ciele
* Nastavenia detekcie cieľov sú príliš prísne

**Riešenia:**

1. Skontrolujte protokol ladenia, či sa v ňom nachádzajú správy „Cieľ nájdený“
2. Overte, či obrázok skutočne obsahuje viditeľné kalibračné ciele
3. Upravte nastavenia detekcie cieľov v nastaveniach projektu
4. Pozrite si [Výber obrázkov cieľov](../processing-images-gui/choosing-target-images.md)

***

## Súvisiace funkcie

### Nástroje prehliadača obrázkov

Pri prezeraní akejkoľvek vrstvy môžete použiť:

* **Ovládacie prvky priblíženia**: Zväčšite obrázok, aby ste mohli skontrolovať detaily
* **Posúvanie**: Kliknutím a ťahaním sa pohybujete po zväčšenom obrázku
* **Kontrola hodnoty pixelov**: Zobrazenie hodnôt v mieste kurzora
* **Navigačné šípky**: Prechod medzi obrázkami pri zachovaní vrstvy
* **Režim percentuálnej hodnoty pixelov**: Prepínanie medzi zobrazením v DN a percentách

Kompletnú dokumentáciu k prehliadaču obrázkov nájdete v časti [Otvorenie obrázka na celú obrazovku](opening-an-image-full-screen.md).

### Index/LUT Sandbox

Pre interaktívne testovanie a vizualizáciu indexov:

* **Výpočet indexu v reálnom čase**: Vyskúšajte rôzne vzorce indexov
* **Mapovanie farieb LUT**: Použite farebné prechody na indexy v odtieňoch šedej
* **Export vizualizácií**: Uložte farebné obrázky indexov

Podrobnosti nájdete v [Index/LUT Sandbox](index-lut-sandbox.md).

***

## Ďalšie kroky

Teraz, keď už rozumiete vrstvám obrázkov:

* [**Otvorenie obrázku na celú obrazovku**](opening-an-image-full-screen.md) – Kompletný sprievodca prehliadačom obrázkov
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Interaktívna vizualizácia indexov
* [**Vzorce multispektrálnych indexov**](../project-settings/multispectral-index-formulas.md) – Referenčný zoznam dostupných indexov
* [**Dokončenie spracovania**](../processing-images-gui/finishing-the-processing.md) – Porozumenie spracovaným výstupom

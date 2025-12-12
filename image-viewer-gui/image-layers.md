# Vrstvy obrázkov

Rozbaľovacie menu Vrstvy obrázkov v prehliadači obrázkov Chloros vám umožňuje rýchlo prepínať medzi rôznymi verziami toho istého obrázku – od pôvodných snímok po spracované výstupy odrazivosti a vypočítané indexové obrázky.

## Čo sú obrazové vrstvy?

V programe Chloros sa **vrstvy** vzťahujú na rôzne výstupy obrazu, ktoré sú k dispozícii pre jeden zdrojový obraz. Pri spracovaní obrazov program Chloros vytvorí viacero verzií:

* **Pôvodné obrazy** (súbory JPG a RAW z fotoaparátu)
* Výstupy s **kalibrovanou odrazivosťou** (ak bola povolená kalibrácia odrazivosti)
* **Cieľové obrázky** (ak obrázok obsahuje kalibračné ciele)
* **Indexové obrázky** (NDVI, NDRE, GNDVI atď., ak boli nakonfigurované indexy)

**Rozbaľovacie menu výberu vrstiev** v pravom hornom rohu prehliadača obrázkov vám umožňuje okamžite prepínať medzi týmito verziami bez opustenia prehliadača.

***

## Dostupné typy vrstiev

### JPG

* Pôvodný náhľadový obrázok JPG z fotoaparátu
* Vždy k dispozícii pre všetky obrázky
* Nespracovaný, tak ako bol zachytený fotoaparátom
* Najrýchlejšie sa načíta a zobrazí

**Kedy zobraziť:**

* Rýchly náhľad pôvodného záberu
* Kontrola kompozície a rámovania obrázku
* Overenie kvality záberu pred spracovaním

### RAW (pôvodný)

* Pôvodné údaje snímača RAW z vášho fotoaparátu
* Bez postprocesingu
* Vyššia bitová hĺbka ako JPG (typicky 12-bitové alebo 14-bitové údaje snímača)

**Kedy zobraziť:**

* Kontrola kvality pôvodných údajov snímača
* Kontrola problémov so senzorom alebo artefaktov
* Porovnanie výsledkov pred a po spracovaní

### RAW (cieľ)

* Zobrazuje sa len pre obrázky identifikované ako obsahujúce kalibračné ciele
* Zobrazuje originálny obrázok RAW s detegovaným cieľom
* Používa sa na overenie, či bola detekcia cieľa úspešná

**Kedy zobraziť:**

* Potvrdenie, že kalibračné ciele boli detegované správne
* Kontrola kvality cieľového obrázku
* Riešenie problémov s kalibráciou

{% hint style=&quot;info&quot; %}
**Cieľová vrstva**: Táto vrstva sa zobrazuje iba v roletovom menu pre obrázky, ktoré obsahujú kalibračné ciele. Bežné zachytené obrázky túto možnosť nemajú.
{% endhint %}

### RAW (odrazivosť)

* Kalibrovaný výstupný obraz odrazivosti
* Korekcia vinetácie (ak je povolená v spracovaní)
* Odrazivosť kalibrovaná pomocou cieľových údajov (ak je povolená)
* Viacpásmový TIFF so všetkými kanálmi fotoaparátu
* Hodnoty pixelov predstavujú percentuálnu odrazivosť (pri použití percentuálneho režimu)
* Pripravené na manipuláciu s [Index/LUT Sandbox](index-lut-sandbox.md)

**Kedy zobraziť:**

* Kontrola kalibrovaných výsledkov
* Overenie kvality kalibrácie
* Kontrola hodnôt pixelov z hľadiska vedeckej presnosti
* Porovnanie s originálom, aby ste videli účinky kalibrácie

{% hint style=&quot;success&quot; %}
**Odporúčané**: Pri kontrole hodnôt pixelov pre vedecké merania a analýzy používajte vrstvu RAW (Reflectance).
{% endhint %}

### RAW (NDVI Index)... a podobné

* Vypočítaný obraz vegetačného indexu (NDVI v tomto príklade)
* Názov indexu sa mení na základe toho, ktorý index bol nakonfigurovaný počas spracovania.
* Príklady: RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI Index) atď.
* Jednopásmový obrázok v odtieňoch sivej zobrazujúci výsledky výpočtu indexu
* Pre každý index nakonfigurovaný v nastaveniach projektu sa zobrazí jedna vrstva

**Možné názvy indexov:**

* RAW (index NDVI)
* RAW (index NDRE)
* RAW (GNDVI Index)
* RAW (OSAVI Index)
* RAW (EVI Index)
* RAW (SAVI Index)
* A mnoho ďalších... (pozri [Vzorec multispektrálneho indexu](../project-settings/multispectral-index-formulas.md))

**Kedy zobraziť:**

* Kontrola výsledkov výpočtu indexu
* Kontrola rozsahov hodnôt indexu
* Identifikácia oblastí záujmu
* Overenie obrazov indexu pred použitím v GIS alebo analýze

***

## Použitie výberu vrstiev

### Otvorenie roletového menu

1. Otvorte obrázok v režime celej obrazovky (kliknite na ľubovoľnú miniatúru v prehliadači obrázkov).
2. Vyhľadajte **roletové menu vrstiev** v pravom hornom rohu prehliadača.
3. Roletové menu zobrazuje aktuálne vybranú vrstvu (napr. „JPG“).
4. Kliknutím na roletové menu zobrazíte všetky dostupné vrstvy.

### Prepínanie vrstiev

1. Kliknite na roletové menu vrstiev, aby sa otvoril zoznam.
2. Zobrazia sa všetky dostupné vrstvy pre aktuálny obrázok.
3. Kliknutím na názov akejkoľvek vrstvy prejdite na túto verziu.
4. Obrázok sa okamžite aktualizuje a zobrazí vybranú vrstvu.

**Rýchle prepínanie:**

* Roletové menu si pamätá vašu poslednú voľbu.
* Pri prechode na ďalší obrázok sa Chloros pokúsi zobraziť rovnaký typ vrstvy.
* Ak táto vrstva na ďalšom obrázku neexistuje, predvolene sa použije JPG.

### Dostupnosť vrstiev

Nie všetky vrstvy sú dostupné pre každý obrázok:

**Vždy dostupné:**

* ✅ JPG (každý obrázok má náhľad JPG)

**Podmienečne dostupné:**

* ⚠️ RAW (originál) – iba ak bol obrázok zachytený v režime RAW alebo RAW+JPG
* ⚠️ RAW (cieľ) – iba ak obrázok obsahuje detegované kalibračné ciele
* ⚠️ RAW (odrazivosť) – iba po spracovaní s povolenou kalibráciou odrazivosti
* ⚠️ RAW (\[Index] Index) – iba po spracovaní s nakonfigurovanými indexmi

***

## Trvalosť vrstiev

### Navigácia medzi obrázkami

Keď prechádzate na iný obrázok (pomocou šípok alebo kliknutím na miniatúry):

**Preferencia vrstvy zostáva zachovaná:**

* Ak sa zobrazuje „RAW (Reflexia)“, ďalší obrázok zobrazuje „RAW (Reflexia)“ (ak je k dispozícii)
* Ak sa zobrazuje „RAW (NDVI Index)“, ďalší obrázok zobrazuje „RAW (NDVI Index)“ (ak je k dispozícii)
* Ak rovnaká vrstva neexistuje, predvolené nastavenie je JPG

**Príklad pracovného postupu:**

1. Otvorte obrázok 1, prepnite na RAW (NDVI Index)
2. Stlačte → pre zobrazenie obrázku 2
3. Obrázok 2 automaticky zobrazí vrstvu RAW (NDVI Index)
4. Pokračujte v navigácii – všetky obrázky zobrazujú vrstvu NDVI
5. Veľmi efektívne pre kontrolu výsledkov indexu na mnohých obrázkoch

***

## Bežné pracovné postupy

### Pracovný postup 1: Porovnanie pred a po

**Cieľ**: Porovnať pôvodný a kalibrovaný obrázok.

1. Otvorte spracovaný obrázok v prehliadači obrázkov.
2. Z roletového menu vyberte **RAW (pôvodný)**.
3. Poznámka: vinetácia a nekalibrované hodnoty.
4. Z roletového menu prejdite na **RAW (odrazivosť)**.
5. Porovnajte – vinetácia bola odstránená, hodnoty boli kalibrované.

### Pracovný postup 2: Kontrola indexu

**Cieľ**: Rýchla kontrola výsledkov NDVI v rámci súboru údajov.

1. Otvorte prvý spracovaný obrázok.
2. Z roletového menu vyberte **RAW (NDVI Index)**.
3. Pomocou klávesy → prejdite na ďalší obrázok
4. Vrstva NDVI zostane automaticky zachovaná
5. Pokračujte cez všetky obrázky a skontrolujte vzory NDVI
6. Prejdite na **RAW (NDRE Index)**, aby ste mohli porovnať

### Pracovný postup 3: Overenie cieľa

**Cieľ**: Overiť, či boli všetky cieľové obrázky správne detegované

1. Prejdite na cieľový obrázok
2. Z roletového menu vyberte **RAW (cieľ)**
3. Overte, či sú kalibračné ciele jasne viditeľné a detegované
4. Prejdite na ďalší cieľový obrázok
5. Opakujte overenie pre všetky ciele

### Pracovný postup 4: Kontrola hodnoty pixelov

**Cieľ**: Skontrolujte hodnoty odrazivosti z hľadiska vedeckej presnosti.

1. Otvorte spracovaný obrázok.
2. Vyberte vrstvu **RAW (Odrazivosť)**.
3. Aktivujte režim **Pixel Percent** (tlačidlo v pravom hornom rohu panela nástrojov).
4. Presuňte kurzor nad oblasti vegetácie.
5. Overte, či sú hodnoty pixelov v očakávaných rozsahoch (30–70 % pre NIR, 5–15 % pre Red).
6. Skontrolujte, či sú hodnoty pôdy a vody primerané.

***

## Porozumenie hodnotám pixelov podľa vrstiev

Rôzne vrstvy zobrazujú rôzne rozsahy hodnôt pixelov:

### Vrstva JPG

* **Rozsah**: 0–255 (8-bitový)
* **Význam**: Zobrazené hodnoty, korigované gama
* **Použitie**: Iba vizuálna kontrola, nie pre vedecké merania

### RAW (originál)

* **Rozsah**: 0–65535 (16-bitový)
* **Význam**: Surové digitálne čísla senzora
* **Použitie**: Kontrola výkonu senzora, nekalibrované

### RAW (odrazivosť)

* **Rozsah**: 0–65 535 (16-bitové TIFF) alebo 0,0–1,0 (32-bitové percento)
* **Význam**: Kalibrovaná percentuálna odrazivosť
* **Použitie**: Vedecké merania a analýzy

**Pre 16-bitový TIFF:** Vydelite číslom 65 535, aby ste získali percentuálnu odrazivosť **Pre 32-bitové Percentá:** Hodnoty priamo predstavujú percentá (0,5 = 50 % odrazivosť)

### RAW (indexové obrázky)

* **Rozsah**: Líši sa podľa indexu (typicky -1,0 až +1,0 pre normalizované indexy)
* **Význam**: Výsledok výpočtu indexu
* **Príklady**:
  * NDVI: -1 až +1 (vegetácia typicky 0,4 až 0,9)
  * NDRE: -1 až +1 (detekcia stresu)
  * EVI: 0 až 1 (vylepšená vegetácia)

***

## Tipy a osvedčené postupy

### Efektívne prepínanie vrstiev

* **Znalosť klávesových skratiek**: Hoci pre vrstvy neexistujú klávesové skratky, navigačné šípky (←/→) fungujú vo všetkých vrstvách
* **Konzistentné pracovné postupy**: Vyberte jednu vrstvu (napr. NDVI) a skontrolujte celý súbor údajov, než prejdete na inú
* **Rýchle porovnania**: Prepínajte medzi originálom a odrazom, aby ste overili kvalitu spracovania

### Úvahy o výkone

* **JPG sa načíta najrýchlejšie**: Použite na rýchlu navigáciu medzi mnohými obrázkami.
* **Vrstvy RAW sa načítajú pomalšie**: Vyššie rozlíšenie a bitová hĺbka.
* **Indexové vrstvy**: Podobná rýchlosť ako vrstvy odrazu.
* **Prvé načítanie je najpomalšie**: Nasledujúce zobrazenia tej istej vrstvy sa ukladajú do vyrovnávacej pamäte a sú rýchlejšie.

### Overenie kvality

* **Vždy skontrolujte RAW (originál)**: Overte kvalitu zdrojových údajov, než budete dôverovať spracovaným výstupom.
* **Porovnajte vrstvy**: Použite prepínanie vrstiev na overenie správnosti spracovania.
* **Skontrolujte rozsahy indexov**: Použite režim Pixel Percent s indexovými vrstvami na overenie správnosti hodnôt.

***

## Riešenie problémov

### Vrstva nie je k dispozícii

**Problém**: Očakávaná vrstva sa nezobrazuje v roletovom menu.

**Možné príčiny:**

* Obrázok nebol spracovaný (k dispozícii sú len formáty JPG a RAW (originál))
* Kalibrácia odrazivosti bola počas spracovania deaktivovaná
* V nastaveniach projektu nebol nakonfigurovaný konkrétny index
* Obrázok je iba cieľový obrázok (pre ciele neboli generované žiadne indexy)

**Riešenia:**

1. Overte, či bol obrázok spracovaný (skontrolujte výstupnú zložku so spracovanými súbormi)
2. Skontrolujte nastavenia projektu, aby ste potvrdili, že indexy boli nakonfigurované
3. Opätovne spracujte s povolenými požadovanými indexmi.

### Zobrazená nesprávna vrstva

**Problém**: Obrázok sa otvorí v neočakávanej vrstve.

**Príčina**: Predvoľba vrstvy z predchádzajúceho obrázku bola prenesená, ale táto vrstva neexistuje v aktuálnom obrázku.

**Riešenie**: Chloros automaticky prejde na formát JPG, ak preferovaná vrstva nie je k dispozícii – ide o normálne správanie.

### Neviditeľné kalibračné ciele

**Problém**: Vrstva RAW (cieľ) nezobrazuje detekciu cieľa.

**Možné príčiny:**

* Ciele neboli detegované počas spracovania.
* Obrázok v skutočnosti neobsahuje ciele.
* Nastavenia detekcie cieľa sú príliš prísne.

**Riešenia:**

1. Skontrolujte protokol ladenia, či neobsahuje správy „Cieľ nájdený“.
2. Overte, či obrázok skutočne obsahuje viditeľné kalibračné ciele.
3. Upravte nastavenia detekcie cieľov v nastaveniach projektu.
4. Pozrite si [Výber cieľových obrázkov](../processing-images-gui/choosing-target-images.md).

***

## Súvisiace funkcie

### Nástroje prehliadača obrázkov

Pri prezeraní akejkoľvek vrstvy môžete použiť:

* **Ovládacie prvky zväčšenia**: Zväčšite obrázok, aby ste mohli skontrolovať detaily.
* **Posun**: Kliknutím a ťahaním presúvajte zväčšený obrázok.
* **Kontrola hodnoty pixelov**: Zobrazenie hodnôt v mieste kurzora.
* **Navigačné šípky**: Presúvajte sa medzi obrázkami bez zmeny vrstvy.
* **Režim percent pixelov**: Prepínanie medzi zobrazením DN a percentami.

Kompletnú dokumentáciu k prehliadaču obrázkov nájdete v časti [Otvorenie obrázka na celú obrazovku](opening-an-image-full-screen.md).

### Index/LUT Sandbox

Pre interaktívne testovanie a vizualizáciu indexov:

* **Výpočet indexu v reálnom čase**: Testujte rôzne vzorce indexov.
* **Mapovanie farieb LUT**: Použite farebné prechody na indexy v odtieňoch sivej.
* **Export vizualizácií**: Uložte farebné obrázky indexov.

Podrobnosti nájdete v časti [Index/LUT Sandbox](index-lut-sandbox.md).

***

## Ďalšie kroky

Teraz, keď už rozumiete vrstvám obrázkov:

* [**Otvorenie obrázku na celú obrazovku**](opening-an-image-full-screen.md) – Kompletný sprievodca prehliadačom obrázkov
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Interaktívna vizualizácia indexov
* [**Vzorec multispektrálneho indexu**](../project-settings/multispectral-index-formulas.md) – Referenčné dostupné indexy
* [**Dokončenie spracovania**](../processing-images-gui/finishing-the-processing.md) – Porozumenie spracovaným výstupom

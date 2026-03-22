# Nastavenie parametrov projektu

Pred spracovaním snímok je dôležité nakonfigurovať nastavenia projektu tak, aby zodpovedali požiadavkám vášho pracovného postupu. Panel Nastavenia projektu <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> poskytuje komplexnú kontrolu nad kalibráciou, možnosťami spracovania, multispektrálnymi indexmi a formátmi exportu.

## Prístup k nastaveniam projektu

1. Otvorte svoj projekt v Chloros
2. Kliknite na ikonu **Nastavenia projektu** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v ľavom bočnom paneli
3. Panel Nastavenia projektu zobrazí všetky možnosti konfigurácie

{% hint style="info" %}
**Nastavenia sa automaticky ukladajú** spolu s vaším projektom. Keď projekt znovu otvoríte, všetky nastavenia sa obnovia.
{% endhint %}

***

## Rýchle nastavenie pre bežné pracovné postupy

### Predvolené nastavenia (odporúčané pre väčšinu používateľov)

Pre typické pracovné postupy s kamerami MAPIR a Survey3 fungujú predvolené nastavenia dobre:

* ✅ **Korekcia vinetácie**: Zapnutá
* ✅ **Kalibrácia odrazivosti**: Zapnutá (vyžaduje snímky cieľov MAPIR)
* ✅ **Metóda debayeringu**: Štandardná (rýchla, stredná kvalita)
* ✅ **Formát exportu**: TIFF (16-bitový)

Stačí importovať snímky a začať spracovanie s týmito predvolenými nastaveniami.

***

## Prehľad nastavení projektu

Panel Nastavenia projektu je rozdelený do niekoľkých kategórií. Nižšie je zhrnutie každej sekcie. Kompletnú dokumentáciu nájdete v [Nastaveniach projektu](../project-settings/project-settings.md).

### Detekcia cieľov

Ovláda spôsob, akým Chloros identifikuje kalibračné ciele vo vašich obrázkoch.

**Kľúčové nastavenia:*** **Minimálna plocha kalibračnej vzorky**: Prahová hodnota veľkosti pre detekciu cieľov (predvolené: 25 pixelov)
* **Minimálne zoskupovanie cieľov**: Prahová hodnota podobnosti pre zoskupovanie oblastí cieľov (predvolené: 60)**Kedy upraviť:**

* Zväčšite plochu vzorky, ak dochádza k falošným detekciám
* Znížte ju, ak sa ciele nedetekujú
* Upravte zoskupovanie, ak sa ciele rozdeľujú na viacero detekcií

### Spracovanie

Hlavné možnosti spracovania obrazu a kalibrácie.

**Kľúčové nastavenia:*** **Korekcia vinety**: Kompenzuje stmavnutie objektívu na okrajoch ✅ Odporúčané
* **Kalibrácia odrazivosti**: Normalizuje hodnoty pomocou kalibračných cieľov ✅ Odporúčané
* **Metóda debayeringu**: Algoritmus na konverziu formátu RAW na 3-kanálový multispektrálny formát
* **Minimálny interval rekalibrácie**: Čas medzi použitím kalibračných cieľov (0 = použiť všetky)**Pokročilé nastavenia:*** **Posun časového pásma svetelného senzora**: Pre synchronizáciu času PPK (predvolené: 0)
* **Použiť korekcie PPK**: Používa údaje GPS/expozície z súborov .daq
* **Expozícia 1/2**: Priraďuje kamery k expozíciám pre konfigurácie s dvoma kamerami

### Metóda debayeringu

V súčasnosti ponúkame v Chloros 2 metódy debayeringu:

#### Štandardná (rýchla, stredná kvalita)

Štandardný debayer spracováva rýchlo, ale vykazuje farebný šum pri debayeringu, čo má za následok menej presné a šumovejšie obrázky.

#### Texture Aware (pomalé, najvyššia kvalita) \[Iba Chloros+]

Texture Aware používa vysoko kvalitný debayer citlivý na hrany v kombinácii s modelom odšumovania AI/ML, ktorý odstraňuje takmer všetok šum pri debayeringu. Model Texture Aware vyžaduje na spustenie pamäť GPU (VRAM). Odporúčame ho používať, ak máte k dispozícii &gt;4 GB VRAM pre rýchlejšie spracovanie.

### Index (multispektrálne indexy)

Nakonfigurujte, ktoré vegetačné indexy sa majú vypočítať a exportovať.

**Ako pridať indexy:**

1. Kliknite na tlačidlo**„Pridať index“**

2. Vyberte index z roletového menu (NDVI, NDRE, GNDVI atď.)
3. Nastavte vizualizačné nastavenia (farby LUT, rozsahy hodnôt)
4. Pridajte podľa potreby viacero indexov

**Populárne indexy:*** **NDVI**: Všeobecný zdravotný stav vegetácie (najbežnejší)
* **NDRE**: Včasná detekcia stresu s RedEdge
* **GNDVI**: Citlivý na koncentráciu chlorofylu
* **OSAVI**: Funguje dobre s viditeľnou pôdou
* **EVI**: Oblasti s vysokým indexom listovej plochy (LAI)**Vlastné vzorce (len Chloros+):**

* Vytvorte vlastné vzorce multispektrálnych indexov
* Použite matematické operácie s pásmami vo všetkých kanáloch obrazu
* Uložte vlastné vzorce na opätovné použitie

Všetky dostupné indexy a vzorce nájdete v [Vzorce multispektrálnych indexov](../project-settings/multispectral-index-formulas.md).

### Export

Ovláda formát a kvalitu výstupného súboru.

**Dostupné formáty:*** **TIFF (16-bitový)**: Odporúčaný pre GIS a vedeckú analýzu (rozsah 0–65 535)
* **TIFF (32-bitový, percentuálny)**: Hodnoty odrazivosti s pohyblivou desatinnou čiarkou (rozsah 0,0–1,0)
* **PNG (8-bit)**: Bezstratová kompresia pre vizualizáciu (rozsah 0–255)
* **JPG (8-bit)**: Najmenšie súbory, stratová kompresia (rozsah 0–255)***

## Ukladanie a načítanie nastavení

### Uloženie šablóny projektu

Vytvorte opakovane použiteľné šablóny pre konzistentné pracovné postupy:

1. Nakonfigurujte všetky požadované nastavenia v paneli Nastavenia projektu
2. Prejdite do sekcie **„Uložiť šablónu projektu“** v dolnej časti
3. Zadajte popisný názov šablóny (napr. „Survey3N\_RGN\_Agriculture“)
4. Kliknite na ikonu uloženia

**Výhody:**

* Použite identické nastavenia vo viacerých projektoch
* Zdieľajte konfigurácie s členmi tímu
* Zachovajte konzistentnosť pri opakovaných prieskumoch

### Načítanie šablóny do nového projektu

Pri vytváraní nového projektu:

1. Z hlavného menu vyberte **„Nový projekt“**

2. Zvoľte možnosť**„Načítať zo šablóny“**

3. Vyberte uloženú šablónu
4. Všetky nastavenia sa automaticky uplatnia

### Pracovný adresár

Nastavenie **„Adresár na uloženie projektu“** určuje, kde sa nové projekty vytvárajú štandardne:

* **Štandardné umiestnenie**: `C:\Users\[Username]\Chloros Projects`
* **Zmeniť umiestnenie**: Kliknite na ikonu úprav a vyberte nový adresár
* **Kedy zmeniť**:
  * Sieťový disk pre tímovú spoluprácu
  * Iný disk s väčším úložným priestorom
  * Usporiadaná štruktúra zložiek podľa roka/klienta

***

## Nastavenie PPK (Post-Processed Kinematic)

Ak používate záznamníky MAPIR DAQ s GPS pre presnú geolokalizáciu:

### Predpoklady

* DAQ MAPIR s modulom GPS (GNSS)
* Protokolový súbor .daq so záznamami o expozícii
* Kamera pripojená k expozíciám DAQ počas snímacej relácie

### Kroky konfigurácie

1. Umiestnite protokolový súbor .daq do priečinka projektu
2. V nastaveniach projektu zaškrtnite políčko **„Použiť korekcie PPK“**

3. V prípade potreby nastavte**„Časový posun svetelného senzora“** (predvolené nastavenie: 0 pre UTC)
4. Priraďte kamery k pinom expozície:
   * **Jedna kamera**: Automaticky priradená k pinu 1
   * **Dve kamery**: Ručne priraďte každú kameru k správnemu pinu**Priradenie pinov expozície:*** **Expozičný pin 1**: Vyberte model kamery z roletového menu
* **Expozičný pin 2**: Vyberte druhú kameru alebo „Nepoužívať“
* Tú istú kameru nemožno priradiť k obom pinom

{% hint style="warning" %}
**Dôležité**: Expozičné piny musia byť správne priradené k príslušným kamerám. Nesprávne priradenie bude mať za následok nesprávne údaje o geolokácii.
{% endhint %}

***

## Pokročilé scenáre

### Projekty s viacerými kamerami

Pri spracovaní snímok z viacerých kamier MAPIR v jednom projekte:

1. Chloros automaticky rozpozná každý model kamery
2. Každá kamera dostane príslušný profil spracovania
3. PPK: Ručne priraďte každú kameru k správnemu expozíciovému bodu
4. Všetky kamery používajú rovnaký formát exportu a indexy

**Príklad**: Survey3W RGN + Survey3N OCN zostava s dvoma kamerami

### Časozberné alebo viacdňové merania

Pre opakované merania tej istej oblasti v priebehu času:

1. Vytvorte šablónu so štandardnými nastaveniami
2. Pri každej relácii používajte konzistentné nastavenie kalibračného terča
3. Spracujte každý dátum ako samostatný projekt
4. Používajte identické nastavenia pre porovnateľné výsledky
5. Exportujte v rovnakom formáte pre časovú analýzu

### Veľké dátové súbory

Pre projekty s veľkým množstvom snímok (500+):

* Zvážte rozdelenie na menšie projekty podľa dátumu alebo oblasti
* Použite paralelné spracovanie Chloros+ pre rýchlejšie výsledky
* Zvážte použitie CLI alebo API pre automatizáciu dávok
* Upravte minimálny interval rekalibrácie, aby ste skrátili čas detekcie cieľa

***

## Overenie nastavení

Pred začatím spracovania skontrolujte tieto kľúčové nastavenia:

* [ ] Model fotoaparátu správne detekovaný v prehliadači súborov
* [ ] Korekcia vinety zapnutá
* [ ] Kalibrácia odrazivosti zapnutá
* [ ] Importovaný aspoň jeden obrázok kalibračného cieľa
* [ ] Pridané požadované multispektrálne indexy
* [ ] Formát exportu vhodný pre váš pracovný postup
* [ ] Nastavenia PPK nakonfigurované (ak používate .daq s udalosťami expozície)

***

## Ďalšie kroky

Po nakonfigurovaní nastavení:

1. **Označte kalibračné cieľové obrázky** – pozrite si [Výber cieľových obrázkov](choosing-target-images.md)
2. **Spustite spracovanie** – pozrite si [Spustenie spracovania](starting-the-processing.md)
3. **Sledujte priebeh** – pozrite si [Sledovanie spracovania](monitoring-the-processing.md)

Úplné podrobnosti o všetkých dostupných nastaveniach nájdete v referenčnej dokumentácii [Nastavenia projektu](../project-settings/project-settings.md).

# Úprava nastavení projektu

Pred spracovaním obrázkov je dôležité nakonfigurovať nastavenia projektu tak, aby zodpovedali požiadavkám vášho pracovného postupu. Panel Nastavenia projektu <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> poskytuje komplexnú kontrolu nad kalibráciou, možnosťami spracovania, multispektrálnymi indexmi a formátmi exportu.

## Prístup k nastaveniam projektu

1. Otvorte svoj projekt v Chloros
2. Kliknite na ikonu **Nastavenia projektu** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v ľavom bočnom paneli
3. Panel Nastavenia projektu zobrazuje všetky možnosti konfigurácie

{% hint style=&quot;info&quot; %}
**Nastavenia sa automaticky ukladajú** spolu s projektom. Pri opätovnom otvorení projektu sa všetky nastavenia obnovia.
{% endhint %}

***

## Rýchle nastavenie pre bežné pracovné postupy

### Predvolené nastavenia (odporúčané pre väčšinu používateľov)

Pre typické pracovné postupy s kamerou MAPIR Survey3 sú predvolené nastavenia vhodné:

* ✅ **Korekcia vinetácie**: Povolená
* ✅ **Kalibrácia odrazivosti**: Povolená (vyžaduje obrázky cieľov MAPIR)
* ✅ **Metóda Debayer**: Vysoká kvalita (rýchlejšia)
* ✅ **Formát exportu**: TIFF (16-bitový)

Stačí importovať obrázky a začať spracúvanie s týmito predvolenými nastaveniami.

***

## Prehľad nastavení projektu

Panel Nastavenia projektu je rozdelený do niekoľkých kategórií. Nižšie je uvedený prehľad jednotlivých častí. Kompletnú dokumentáciu nájdete v časti [Nastavenia projektu](../project-settings/project-settings.md).

### Detekcia cieľov

Ovláda spôsob, akým Chloros identifikuje kalibračné ciele vo vašich obrázkoch.

**Kľúčové nastavenia:**

* **Minimálna plocha vzorky kalibrácie**: Prahová hodnota veľkosti pre detekciu cieľov (predvolené nastavenie: 25 pixelov)
* **Minimálne zhlukovanie cieľov**: Prahová hodnota podobnosti pre zoskupovanie cieľových oblastí (predvolené nastavenie: 60)

**Kedy upraviť:**

* Zväčšite plochu vzorky, ak dochádza k falošným detekciám.
* Zmenšite ju, ak sa ciele nedetekujú.
* Upravte zhlukovanie, ak sa ciele rozdeľujú na viacero detekcií.

### Spracovanie

Hlavné možnosti spracovania a kalibrácie obrázkov.

**Kľúčové nastavenia:**

* **Korekcia vinetácie**: Kompenzuje stmavnutie objektívu na okrajoch ✅ Odporúčané
* **Kalibrácia odrazivosti**: Normalizuje hodnoty pomocou kalibračných cieľov ✅ Odporúčané
* **Metóda Debayer**: Algoritmus na konverziu formátu RAW na 3-kanálový multispektrálny
* **Minimálny interval rekalibrácie**: Čas medzi použitím kalibračných cieľov (0 = použiť všetky)

**Pokročilé nastavenia:**

* **Posun časového pásma svetelného senzora**: Pre synchronizáciu času PPK (predvolené: 0)
* **Použiť korekcie PPK**: Používa údaje GPS/expozície z .daq súborov
* **Expozičný pin 1/2**: Priraďuje kamery k expozičným pinom pre konfigurácie s dvoma kamerami

### Index (multispektrálne indexy)

Nakonfigurujte, ktoré vegetačné indexy sa majú vypočítať a exportovať.

**Ako pridať indexy:**

1. Kliknite na tlačidlo **„Pridať index“**
2. Vyberte index z roletového menu (NDVI, NDRE, GNDVI atď.)
3. Nakonfigurujte nastavenia vizualizácie (farby LUT, rozsahy hodnôt)
4. Pridajte podľa potreby viacero indexov

**Populárne indexy:**

* **NDVI**: Všeobecný zdravotný stav vegetácie (najbežnejší)
* **NDRE**: Včasná detekcia stresu s RedEdge
* **GNDVI**: Citlivý na koncentráciu chlorofylu
* **OSAVI**: Funguje dobre s viditeľnou pôdou
* **EVI**: Oblasti s vysokým indexom listovej plochy (LAI)

**Vlastné vzorce (len Chloros+):**

* Vytvorte vlastné multispektrálne indexové vzorce
* Použite matematiku pásma so všetkými obrazovými kanálmi
* Uložte vlastné vzorce na opätovné použitie

Všetky dostupné indexy a vzorce nájdete v časti [Multispektrálne indexové vzorce](../project-settings/multispectral-index-formulas.md).

### Export

Ovláda formát a kvalitu výstupného súboru.

**Dostupné formáty:**

* **TIFF (16-bitový)**: Odporúčaný pre GIS a vedeckú analýzu (rozsah 0–65 535)
* **TIFF (32-bitový, percentuálny)**: Hodnoty odrazivosti s pohyblivou desatinnou čiarkou (rozsah 0,0–1,0)
* **PNG (8-bitový)**: Bezstratová kompresia pre vizualizáciu (rozsah 0–255)
* **JPG (8-bitový)**: Najmenšie súbory, stratová kompresia (rozsah 0–255)

***

## Ukladanie a načítanie nastavení

### Uloženie šablóny projektu

Vytvorte šablóny, ktoré môžete opakovane používať pre konzistentné pracovné postupy:

1. Nakonfigurujte všetky požadované nastavenia v paneli Nastavenia projektu.
2. Prejdite do sekcie **„Uložiť šablónu projektu“** v dolnej časti.
3. Zadajte popisný názov šablóny (napr. „Survey3N\_RGN\_Agriculture“).
4. Kliknite na ikonu uloženia.

**Výhody:**

* Použite identické nastavenia vo viacerých projektoch.
* Zdieľajte konfigurácie s členmi tímu.
* Zachovajte konzistentnosť pri opakovaných prieskumoch.

### Načítanie šablóny do nového projektu

Pri vytváraní nového projektu:

1. V hlavnom menu vyberte **„Nový projekt“**.
2. Zvoľte možnosť **„Načítať zo šablóny“**.
3. Vyberte uloženú šablónu.
4. Všetky nastavenia sa automaticky použijú.

### Pracovný adresár

Nastavenie **„Uložiť priečinok projektu“** určuje, kde sa nové projekty vytvárajú štandardne:

* **Štandardná poloha**: `C:\Users\[Username]\Chloros Projects`
* **Zmeniť polohu**: Kliknite na ikonu úprav a vyberte nový priečinok.
* **Kedy zmeniť**:
  * Sieťový disk pre spoluprácu tímu.
  * Iný disk s väčším úložným priestorom.
  * Usporiadaná štruktúra priečinkov podľa roka/klienta.

***

## Nastavenie PPK (Post-Processed Kinematic)

Ak používate záznamníky MAPIR DAQ s GPS pre presnú geolokalizáciu:

### Predpoklady

* MAPIR DAQ s modulom GPS (GNSS)
* .daq log súbor s údajmi o expozícií
* Kamera pripojená k expozičným pinom DAQ počas snímania

### Konfiguračné kroky

1. Umiestnite log súbor .daq do priečinka projektu.
2. V nastaveniach projektu zaškrtnite políčko **„Použiť PPK korekcie“**.
3. V prípade potreby nastavte **„Časový posun svetelného senzora“** (predvolené nastavenie: 0 pre UTC).
4. Priraďte kamery k expozičným pinom:
   * **Jedna kamera**: Automaticky priradená k pinu 1.
   * **Dve kamery**: Ručne priraďte každú kameru k správnemu pinu.

**Priradenie expozičných pinov:**

* **Expozičný pin 1**: Vyberte model kamery z roletového menu.
* **Expozičný pin 2**: Vyberte druhú kameru alebo „Nepoužívať“.
* Tú istú kameru nemožno priradiť k obom pinom.

{% hint style=&quot;warning&quot; %}
**Dôležité**: Kolíky expozície musia byť správne priradené k príslušným kamerám. Nesprávne priradenie bude mať za následok nesprávne údaje o geolokácii.
{% endhint %}

***

## Pokročilé scenáre

### Projekty s viacerými kamerami

Pri spracovaní obrázkov z viacerých kamier MAPIR v jednom projekte:

1. Chloros automaticky detekuje každý model kamery
2. Každá kamera dostane príslušný profil spracovania
3. PPK: Ručne priraďte každej kamere správny expozičný pin.
4. Všetky kamery používajú rovnaký formát exportu a indexy.

**Príklad**: Survey3W RGN + Survey3N OCN dvojitá kamera

### Časozberné alebo viacdňové prieskumy

Pre opakované prieskumy tej istej oblasti v priebehu času:

1. Vytvorte šablónu so štandardnými nastaveniami.
2. Používajte konzistentné nastavenie kalibračného cieľa v každej relácii.
3. Spracujte každý deň ako samostatný projekt.
4. Používajte identické nastavenia pre porovnateľné výsledky.
5. Exportujte v rovnakom formáte pre časovú analýzu.

### Veľké dátové súbory

Pre projekty s veľkým množstvom obrázkov (500+):

* Zvážte rozdelenie na menšie projekty podľa dátumu alebo oblasti.
* Použite paralelné spracovanie Chloros+ pre rýchlejšie výsledky.
* Zvážte použitie CLI alebo API pre automatizáciu dávok.
* Upravte minimálny interval rekalibrácie, aby ste skrátili čas detekcie cieľa.

***

## Overenie nastavení

Pred začatím spracovania skontrolujte tieto kľúčové nastavenia:

* [ ] Model kamery správne detegovaný v prehliadači súborov
* [ ] Korekcia vinetácie povolená
* [ ] Kalibrácia odrazivosti povolená
* [ ] Importovaný aspoň jeden kalibračný cieľový obrázok
* [ ] Pridané požadované multispektrálne indexy
* [ ] Exportný formát vhodný pre váš pracovný postup
* [ ] Konfigurované nastavenia PPK (ak používate .daq s expozíciou udalostí)

***

## Ďalšie kroky

Po nakonfigurovaní nastavení:

1. **Označte kalibračné cieľové obrázky** – pozrite si [Výber cieľových obrázkov](choosing-target-images.md)
2. **Spustite spracovanie** – pozrite si [Spustenie spracovania](starting-the-processing.md)
3. **Sledujte priebeh** – pozrite si [Sledovanie spracovania](monitoring-the-processing.md)

Úplné podrobnosti o všetkých dostupných nastaveniach nájdete v referenčnej dokumentácii [Nastavenia projektu](../project-settings/project-settings.md).

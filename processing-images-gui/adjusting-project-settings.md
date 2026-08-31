# Nastavenie parametrov projektu

Pred spracovaním snímok je dôležité nakonfigurovať nastavenia projektu tak, aby zodpovedali požiadavkám vášho pracovného postupu. Panel „Nastavenia projektu“ (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">) poskytuje komplexnú kontrolu nad kalibráciou, možnosťami spracovania, multispektrálnymi indexmi a formátmi exportu.

## Otvorenie nastavení projektu

1. Otvorte svoj projekt v programe Chloros
2. Kliknite na ikonu **Nastavenia projektu** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v ľavom bočnom paneli
3. Panel „Nastavenia projektu“ zobrazuje všetky možnosti konfigurácie

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Panel Nastavenia projektu — Zobrazenie, detekcia cieľov a spracovanie</p></figcaption></figure>{% hint style="info" %}
**Nastavenia sa automaticky ukladajú** spolu s vaším projektom. Keď projekt znovu otvoríte, všetky nastavenia sa obnovia.
{% endhint %}

***

## Rýchle nastavenie pre bežné pracovné postupy

### Predvolené nastavenia (odporúčané pre väčšinu používateľov)

Predvolené nastavenia fungujú dobre pre typické pracovné postupy s Survey3 a LATTICE:

* ✅ **Korekcia vinety**: Zapnutá
* ✅ **Kalibrácia odrazivosti / vyváženie bielej**: Zapnutá (využíva ciele MAPIR a/alebo údaje zo svetelného senzora DAQ)
* ✅ **Metóda debayeringu**: Štandardná (rýchla, stredná kvalita)
* ✅ **Formát exportu**: TIFF (16-bitový)
* ✅ **Všetky exportované výstupy**: Zapnuté (LATTICE automaticky exportuje fan-out do formátov debayered, náhľad, radiancia a odrazivosť)

Stačí importovať vaše snímky a začať spracovanie s týmito predvolenými nastaveniami.

***

## Prehľad nastavení projektu

Panel Nastavenia projektu je rozdelený do nižšie uvedených sekcií. Dve ďalšie sekcie — **Svetelný senzor DAQ**a**Vyrovnanie poľa** — sa zobrazia automaticky, ak váš projekt obsahuje príslušné súbory. Úplnú dokumentáciu nájdete v [Nastaveniach projektu](../project-settings/project-settings.md).

### Zobrazenie

* **Rozlíšenie miniatúr snímok**: Rozlíšenie miniatúr v mriežke snímok. Možnosti:**Predvolené (512 px)**,**1024 px**,**2048 px**,**Plné rozlíšenie**. Iba na zobrazenie — nikdy neovplyvňuje spracovanie. Vyššie hodnoty vyzerajú pri zväčšení ostrejšie, ale načítajú sa pomalšie.

### Detekcia cieľov

Ovláda spôsob, akým Chloros identifikuje kalibračné ciele vo vašich obrázkoch.

**Kľúčové nastavenia:*** **Minimálna plocha vzorky na kalibráciu (px)**: Prahová hodnota veľkosti pre detekciu cieľov (predvolené:**25**, rozsah 0–10000)
* **Minimálne zhlukovanie cieľov (0–100)**: Prah podobnosti pre zoskupovanie oblastí cieľov (predvolené:**60**)**Kedy nastaviť:**

* Zväčšite vzorkovú oblasť, ak dochádza k falošným detekciám
* Zmenšite ju, ak sa ciele nedetekujú
* Upravte zoskupovanie, ak sa ciele rozdeľujú na viacero detekcií

{% hint style="info" %}
Tieto nastavenia sú sivé, ak je vypnutá funkcia **Kalibrácia odrazivosti / vyváženie bielej** – ak je táto funkcia vypnutá, detekcia cieľov sa vôbec nespustí.
{% endhint %}

### Spracovanie

Hlavné možnosti spracovania obrazu a kalibrácie.

**Kľúčové nastavenia:*** **Korekcia vinetácie**: Kompenzuje stmavnutie objektívu na okrajoch ✅ Odporúčané
* **Kalibrácia odrazivosti / vyváženie bielej**: Kalibruje snímky pomocou detekovaných cieľov (Survey3) a/alebo údajov zo svetelného senzora DAQ (LATTICE) ✅ Odporúčané
* **Metóda debayeringu**: Algoritmus na konverziu formátu RAW na 3-kanálový multispektrálny formát
* **Minimálny interval rekalibrácie**: Minimálny čas v sekundách medzi použitím kalibračného cieľa (predvolené:**0** = použiť všetky, rozsah 0–3600)**Nekalibrované náhradné produkty:**Ak nie je možné kalibrovať odrazivosť snímky (nie je k dispozícii žiadny cieľ alebo je kalibrácia vypnutá), exportuje sa ako jeden z dvoch náhradných produktov —**na jedno spustenie existuje presne jeden z tejto dvojice**, vybraný prepínačom korekcie vinety:

* **Exportovať odozvu senzora**: zapíše `Sensor_Response_Images` — používa sa, keď je korekcia vinetácie**vypnutá*** **Export s korekciou vinetácie**: zapíše súbor `Vignette_Corrected_Images` — používa sa, keď je korekcia vinetácie**zapnutá**Zaškrtávacie políčko, ktoré nie je aktívne, je sivé. Zrušením zaškrtnutia aktívneho políčka sa úplne zastaví zapisovanie daného súboru.**Produkty exportu LATTICE** (zobrazujú sa pri každom projekte; vzťahujú sa na snímky LATTICE):

* **Export s odstránením bayera**: lineárny obraz bez bayera (`Debayered_Images`). Platí pre RGB a multispektrálne moduly.
* **Export náhľadu**: náhľad na displeji (`Preview_Images`). RGB = vyváženie bielej (svetelný zdroj DAQ, ak je k dispozícii, inak šedý svet) + gama; multispektrálne = rozloženie falošných farieb.
* **Export radiancie**: spektrálna radiancia typu float32 (`Radiance_Images`, W/m²/sr/nm). Iba multispektrálne moduly — neplatí pre hlavné moduly RGB.
* ****Export odrazivosti**: odrazivosť typu uint16 (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0), ak snímku pokrýva hodnota smerujúca nadol z `.daq` alebo cieľ v rámci snímky. Iba multispektrálne moduly.

Všetky štyri sú **štandardne zapnuté**– jeden importovaný surový snímok LATTICE sa v jednom spracovateľskom cykle rozdelí do všetkých povolených a príslušných produktov. Zaškrtávacie políčko**Exportovať odrazivosť** je sivé, ak je kalibrácia odrazivosti vypnutá. Nastavenia, ktorých aktivácia je znemožnená nadradeným prepínačom, sú vždy sivé a obsahujú popisok s názvom prepínača, ktorý je potrebné zmeniť.**Pokročilé nastavenia:*** **Posun časového pásma svetelného senzora**: Počet hodín od UTC pre zosúladenie času svetelného senzora (predvolené: 0, rozsah −12 až +12)
* **Použiť korekcie PPK**: Používa údaje GPS/expozičných pinov zo súborov `.daq` (predvolené nastavenie: vypnuté)
* **Expozičný pin 1/2**: Priraďuje kamery k expozičným pinom pre konfigurácie s dvoma kamerami

{% hint style="info" %}
**Úroveň vstupu LATTICE je automatická.** Záznamy LATTICE obsahujú svoju úroveň spracovania v metadátach XMP a spracovanie vždy vstupuje do spracovateľského reťazca na úrovni surového snímku — v grafickom rozhraní nie je potrebné nič konfigurovať. (Prieznak CLI `--input-level` slúži ako pokročilá možnosť pre používateľov na prepisovanie nastavení pri záznamoch so stratenými metadátami; pozri [Referenciu CLI](../reference/cli-reference.md).)
{% endhint %}

### Metóda odstraňovania Bayerovho vzoru

V súčasnosti ponúkame v Chloros 2 metódy odstraňovania Bayerovho vzoru:

#### Štandardná (rýchla, stredná kvalita)

Štandardná metóda odstraňovania Bayerovho vzoru spracováva rýchlo, ale vykazuje farebný šum spôsobený odstraňovaním vzoru, čo má za následok menej presné a šumovejšie obrázky.

#### S ohľadom na textúru (pomalé, najvyššia kvalita) \[Iba Chloros+]

Metóda s ohľadom na textúru využíva vysokokvalitný debayering zohľadňujúci hrany v kombinácii s modelom odšumovania založeným na AI/ML, ktorý odstraňuje takmer všetok šum spôsobený debayeringom. Na spustenie modelu je potrebná pamäť GPU (VRAM): pri **7 GB alebo viac VRAM** dokáže spracovávať viacero obrázkov súčasne; pri menej ako 7 GB spracováva jeden obrázok za druhým (čo je výrazne pomalšie). Pozrite si [Dynamickú adaptáciu výpočtov](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**Zábery LATTICE vždy používajú štandardné demosaikovanie.** Neexistuje žiadny model Texture Aware vytrénovaný pre LATTICE, preto táto možnosť nie je k dispozícii pre snímky LATTICE — snímky Survey3 v tom istom projekte ju však môžu naďalej používať.
{% endhint %}

### Index (multispektrálne indexy)

Nastavte, ktoré vegetačné indexy sa majú vypočítať a exportovať. Rozbaľovacie menu v grafickom rozhraní ponúka **27 preddefinovaných vzorcov indexov**.**Ako pridať indexy:**

1. Kliknite na tlačidlo**„Pridať index“**

2. Z roletového menu vyberte index (NDVI, NDRE, GNDVI atď.)
3. Nastavte parametre vizualizácie (farby LUT, rozsahy hodnôt)
4. Podľa potreby pridajte viacero indexov

**Populárne indexy:*** **NDVI**: Všeobecný zdravotný stav vegetácie (najbežnejší)
* **NDRE**: Včasná detekcia stresu v kombinácii s RedEdge
* **GNDVI**: Citlivý na koncentráciu chlorofylu
* **OSAVI**: Funguje dobre pri viditeľnej pôde
* **EVI**: Oblasti s vysokým indexom listovej plochy (LAI)**Vlastné vzorce:**

* Vytvorte vlastné vzorce multispektrálnych indexov pomocou matematických operácií s pásmami vo všetkých kanáloch snímky
* Uložte vlastné vzorce na opätovné použitie
* Vlastné vzorce sú funkciou Chloros+; ich dostupnosť závisí od úrovne vášho plánu

Všetky dostupné indexy a vzorce – vrátane toho, ktoré názvy sú k dispozícii iba v grafickom rozhraní a ktoré fungujú aj v aplikáciách CLI/SDK – nájdete v [Vzorce multispektrálnych indexov](../project-settings/multispectral-index-formulas.md).

### Export

Ovláda formát výstupného súboru.

**Dostupné formáty**(nastavenie:**Formát kalibrovaného obrazu**, predvolené**TIFF (16-bitový)**):

* **TIFF (16-bit)**: Odporúčané pre GIS a vedeckú analýzu
* **TIFF (32-bit, percentá)**: Hodnoty s plávajúcou desatinnou čiarkou
* **PNG (8-bit)**: Bezstratová kompresia pre vizualizáciu
* **JPG (8-bit)**: Najmenšie súbory, stratová kompresia

Výstupy sa ukladajú do priečinka projektu, zoskupené podľa kamery a formátu: `<project>/<camera>/<format>/<Product>_Images/`. Hodnota jasu (Radiance) sa **vždy** ukladá ako float32 do priečinka `tiff32` bez ohľadu na toto nastavenie. Exportované súbory si zachovávajú názov zdrojového súboru — produkt identifikuje zložka. Úplnú štruktúru výstupných súborov nájdete v časti [Dokončenie spracovania](finishing-the-processing.md).

{% hint style="warning" %}
**Čítanie hodnôt odrazivosti**: DN, pri ktorom platí ρ = 1,0, závisí od zdrojovej kamery — LATTICE používa hodnotu 32768 (označenú ako XMP `Chloros:PixelScale`), Survey3 používa hodnotu 65535. Prečítajte si značku namiesto predpokladania konštanty. Pozrite si [Formáty výstupných obrázkov](../output-image-formats.md).
{% endhint %}

### Svetelný senzor DAQ

V tejto časti je uvedený zoznam všetkých súborov DAQ s údajmi o dopadajúcom slnečnom žiarení (`.daq` / `.csv`) vo vašom projekte, jeden riadok na súbor, s uvedením modelu senzora, názvu súboru a korekcie **krytu** difúzora platnej pre daný súbor.

* **Prepísanie limitu (všetky súbory)**: jediné roletové menu pre celý projekt. Možnosť**Auto** (predvolené) používa limit zaznamenaný v každom súbore — ak nebolo nič zaznamenané, predpokladá sa slnečné žiarenie, keďže všetky DAQ súbory typu MAPIR sú dodávané s korektorom slnečného žiarenia. Výber limitu prepisuje všetky súbory: surové záznamy sa ním korigujú a záznamy, ktoré už obsahujú limit, sa prepočítajú (zaznamenaná korekcia sa zruší a uplatní sa vybraný limit).
* Riadky upozorňujú, ak bol zaznamenaný limit predpokladaným predvoleným nastavením rozbočovača a nie potvrdeným obsluhou, a ak vybraný limit nemá profil pre daný model zariadenia (pre daný súbor sa prepis odmietne).

Záznamy z DAQ vytvorené na karte „Light Sensors“ (Svetelné senzory) sa automaticky pridávajú do otvoreného projektu a importované súbory `.daq` / `.csv` sa tu zobrazia hneď po ich pridaní.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Dolné nastavenia projektu — Index, Formát exportu, sekcia DAQ svetelných senzorov a ovládacie prvky šablóny/zložky projektu</p></figcaption></figure>### Zarovnanie poľa

Táto sekcia sa zobrazí **len** vtedy, ak aspoň jeden obrázok v projekte obsahuje transformáciu zarovnania medzi modulmi, ktorú polia LATTICE vtlačia pri snímaní (`Chloros:Alignment*` XMP). Ukazuje, koľko snímok obsahuje značky a ktorá kamera slúži ako referenčná, s týmito ovládacími prvkami:

* **Použiť zarovnanie poľa** (predvolené: zapnuté): deformuje každý spracovaný výstup (debayering / náhľad / radiancia / odrazivosť / index) do spoločnej referenčnej geometrie poľa. Vypnuté = export v pôvodnej geometrii senzora.
* **Orezanie na spoločný prekrývací priestor** (predvolené: zapnuté): oreže zarovnané exporty na oblasť, ktorú zdieľajú všetky moduly, takže každé pásmo má rovnakú plochu. Vypnuté zachováva plnú plochu snímača (čierna výplň mimo zdrojového obrazu).
* **Prevzorkovanie**:**Bilineárne (hladké, predvolené)**,**Najbližšie (zachovanie presných hodnôt)**— bez miešania medzi pixelmi, pre prísnu radiometrickú analýzu — alebo**Kubické (najostrejšie)**.***

## Ukladanie a načítanie nastavení

### Uloženie šablóny projektu

Vytvorte opakovane použiteľné šablóny pre konzistentné pracovné postupy:

1. Nakonfigurujte všetky požadované nastavenia v paneli Nastavenia projektu
2. Prejdite do sekcie **„Uložiť šablónu projektu“** v spodnej časti
3. Zadajte výstižný názov šablóny (napr. „Survey3N\_RGN\_Agriculture“)
4. Kliknite na ikonu uloženia

**Výhody:**

* Používanie identických nastavení vo viacerých projektoch
* Zdieľajte konfigurácie s členmi tímu
* Zabezpečte konzistentnosť pri opakovaných prieskumoch

### Načítanie šablóny do nového projektu

Pri vytváraní nového projektu:

1. V hlavnom menu vyberte **„Nový projekt“**

2. V voliteľnom výbere šablón vyberte šablónu projektu
3. Všetky nastavenia zo šablóny sa automaticky uplatnia

### Pracovný adresár

Nastavenie **„Pracovný adresár“** určuje, kde sa nové projekty štandardne vytvárajú:

* **Štandardné umiestnenie**: `C:\Users\[Username]\Chloros Projects`
* **Zmena umiestnenia**: Kliknite na ikonu úprav a vyberte nový priečinok
* **Zdieľané s CLI**: `chloros-cli` používa rovnaké predvolené nastavenie priečinka pre projekty
* **Kedy zmeniť**:
  * Sieťový disk pre tímovú spoluprácu
  * Iná jednotka s väčším úložným priestorom
  * Usporiadaná štruktúra zložiek podľa roka/klienta

***

## Nastavenie PPK (Post-Processed Kinematic)

Ak používate záznamníky DAQ MAPIR s GPS na presnú geolokalizáciu:

### Predpoklady

* DAQ MAPIR s modulom GPS (GNSS)
* Protokolový súbor .daq so záznamami o expozičných pinoch
* Fotoaparát pripojený k expozičným pinom DAQ počas snímacej relácie

### Kroky konfigurácie

1. Umiestnite súbor protokolu .daq do priečinka projektu
2. V nastaveniach projektu zaškrtnite políčko **„Použiť korekcie PPK“**

3. V prípade potreby nastavte**„Časový posun svetelného senzora“** (predvolené: 0 pre UTC)
4. Priraďte kamery k pinom expozície:
   * **Jedna kamera**: Automaticky priradená k pinu 1
   * **Dve kamery**: Každú kameru ručne priraďte k správnemu pinu**Priradenie pinov expozície:*** **Expozičný pin 1**: Vyberte model kamery z roletového menu
* **Expozičný pin 2**: Vyberte druhú kameru alebo možnosť „Nepoužívať“
* Jednu kameru nemožno priradiť k obom pinom

{% hint style="warning" %}
**Dôležité**: Piny expozície musia byť správne priradené k príslušným kamerám. Nesprávne priradenie bude mať za následok nesprávne údaje o geolokácii.
{% endhint %}

***

## Pokročilé scenáre

### Projekty s viacerými kamerami

Pri spracovaní snímok z viacerých kamier MAPIR v jednom projekte:

1. Chloros automaticky rozpozná každý model kamery (Survey3 aj LATTICE)
2. Každá kamera dostane príslušné profily spracovania a každá kamera má vlastnú štruktúru výstupných zložiek
3. PPK: Ručne priraďte každú kameru typu Survey3 k správnemu pinu expozície
4. Všetky kamery používajú rovnaký formát exportu a indexy

**Príklady**: Survey3W RGN + Survey3N OCN konfigurácia s dvoma kamerami, alebo sústava LATTICE kombinujúca hlavnú kameru typu RGB s úzkopásmovými modulmi

### Časozberné alebo viacdňové merania

Pre opakované merania tej istej oblasti v priebehu času:

1. Vytvorte šablónu so štandardnými nastaveniami
2. Pri každej relácii používajte konzistentné nastavenie kalibračných cieľov
3. Každý dátum spracujte ako samostatný projekt
4. Používajte identické nastavenia pre porovnateľné výsledky
5. Exportujte v rovnakom formáte pre časovú analýzu

### Veľké súbory údajov

Pre projekty s veľkým počtom snímok (500 a viac):

* Zvážte rozdelenie na menšie projekty podľa dátumu alebo oblasti
* Použite paralelné spracovanie Chloros+ pre rýchlejšie výsledky
* Zvážte použitie CLI alebo API pre automatizáciu hromadného spracovania
* Upravte minimálny interval rekalibrácie, aby ste skrátili čas detekcie cieľa

***

## Overenie nastavení

Pred začatím spracovania skontrolujte tieto kľúčové nastavenia:

* [ ] Model kamery je správne rozpoznaný v prehliadači súborov
* [ ] Korekcia vinety je zapnutá
* [ ] Kalibrácia odrazivosti je zapnutá
* [ ] Pre Survey3: bol importovaný a skontrolovaný aspoň jeden obrázok kalibračného cieľa; pre LATTICE: je k dispozícii cieľ a/alebo záznam smerujúci nadol z `.daq`
* [ ] Pridané požadované multispektrálne indexy
* [ ] Formát exportu vhodný pre váš pracovný postup
* [ ] Nastavenia PPK nakonfigurované (ak používate súbory .daq s udalosťami expozície)

***

## Ďalšie kroky

Po nakonfigurovaní nastavení:

1. **Označte kalibračné cieľové snímky** – pozri [Výber cieľových snímok](choosing-target-images.md)
2. **Spustite spracovanie** – pozri [Spustenie spracovania](starting-the-processing.md)
3. **Sledujte priebeh** – pozrite si [Sledovanie spracovania](monitoring-the-processing.md)

Úplné podrobnosti o všetkých dostupných nastaveniach nájdete v referenčnej dokumentácii [Nastavenia projektu](../project-settings/project-settings.md).

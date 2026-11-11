# Nastavenie parametrov projektu

Pred spracovaním obrázkov je dôležité nakonfigurovať nastavenia projektu tak, aby zodpovedali požiadavkám vášho pracovného postupu. Panel „Nastavenia projektu“ <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> ponúka komplexnú kontrolu nad kalibráciou, možnosťami spracovania, multispektrálnymi indexmi a formátmi exportu.

## Otvorenie nastavení projektu

1. Otvorte svoj projekt v programe Chloros
2. Kliknite na ikonu **Nastavenia projektu** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> v ľavom bočnom paneli
3. Panel „Nastavenia projektu“ zobrazuje všetky možnosti konfigurácie

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Panel Nastavenia projektu — Zobrazenie, detekcia cieľov a spracovanie</p></figcaption></figure>

{% hint style="info" %}
**Nastavenia sa automaticky ukladajú** spolu s vaším projektom. Keď projekt opäť otvoríte, všetky nastavenia sa obnovia.
{% endhint %}

***

## Rýchle nastavenie pre bežné pracovné postupy

### Predvolené nastavenia (odporúčané pre väčšinu používateľov)

Predvolené nastavenia fungujú dobre pre typické pracovné postupy v programoch Survey3 a LATTICE:

* ✅ **Korekcia vinety**: Zapnutá
* ✅ **Kalibrácia odrazivosti / vyváženie bielej**: Zapnuté (využíva ciele MAPIR a/alebo údaje zo svetelného senzora DAQ)
* ✅ **Metóda debayeringu**: Štandardná (rýchla, stredná kvalita)
* ✅ **Formát exportu**: TIFF (16-bitový) (TIFF (8-bitový) pre projekty s 8-bitovým rozsahom)
* ✅ **Všetky exportované výstupy**: Zapnuté (LATTICE automaticky exportuje rozložené snímky, náhľad, radianciu a odrazivosť)

Stačí importovať obrázky a začať spracovanie s týmito predvolenými nastaveniami.

***

## Prehľad nastavení projektu

Panel Nastavenia projektu je rozdelený do nižšie uvedených sekcií. Dve ďalšie sekcie — **Svetelný senzor DAQ**a**Vyrovnanie poľa** — sa zobrazia automaticky, ak váš projekt obsahuje príslušné súbory. Úplnú dokumentáciu nájdete v [Nastaveniach projektu](../project-settings/project-settings.md).

### Zobrazenie

* **Rozlíšenie miniatúr obrázkov**: Rozlíšenie miniatúr v mriežke obrázkov. Možnosti:**Predvolené (512 px)**,**1024 px**,**2048 px**,**Plné rozlíšenie**. Slúži len na zobrazenie – nemá vplyv na spracovanie. Vyššie hodnoty vyzerajú pri zväčšení ostrejšie, ale načítajú sa pomalšie.

### Detekcia cieľov

Určuje, ako program Chloros identifikuje kalibračné ciele na vašich obrázkoch.

**Kľúčové nastavenia:*** **Minimálna plocha kalibračnej vzorky (px)**: Prahová hodnota veľkosti pre detekciu cieľov (predvolené:**25**, rozsah 0–10 000)
* **Minimálne zoskupovanie cieľov (0–100)**: Prah podobnosti pre zoskupovanie oblastí cieľov (predvolené:**60**)**Kedy nastaviť:**

* Zväčšite vzorkovú oblasť, ak dochádza k falošným detekciám
* Zmenšite ju, ak sa ciele nedetekujú
* Upravte zoskupovanie, ak sa ciele rozdeľujú na viacero detekcií

{% hint style="info" %}
Tieto nastavenia sú sivé, ak je vypnutá funkcia **Kalibrácia odrazivosti / vyváženie bielej** — ak je vypnutá, detekcia cieľov sa vôbec nespustí.
{% endhint %}

### Spracovanie

Hlavné možnosti spracovania obrazu a kalibrácie.

**Kľúčové nastavenia:*** **Korekcia vinetácie**: Kompenzuje stmavnutie objektívu na okrajoch ✅ Odporúčané
* **Kalibrácia odrazivosti / vyváženie bielej**: Kalibruje snímky pomocou detekovaných cieľov (systém Survey3) a/alebo údajov zo svetelného senzora DAQ (LATTICE) ✅ Odporúčané
* **Metóda debayeringu**: Algoritmus na konverziu formátu RAW na 3-kanálový multispektrálny formát
* **Minimálny interval rekalibrácie**: Minimálny čas v sekundách medzi použitím kalibračného cieľa (predvolené:**0** = použiť všetky, rozsah 0–3600)**Nekalibrované náhradné produkty:**Ak nie je možné kalibrovať odrazivosť snímky (nie je k dispozícii žiadny cieľ alebo je kalibrácia vypnutá), exportuje sa ako jeden z dvoch náhradných produktov —**na jedno spustenie existuje presne jeden z tejto dvojice**, vybraný prepínačom korekcie vinetácie:

* **Exportovať odozvu senzora**: zapíše `Sensor_Response_Images` — používa sa, keď je korekcia vinetácie**vypnutá*** **Export s korekciou vinetácie**: zapíše `Vignette_Corrected_Images` — používa sa, keď je korekcia vinetácie**zapnutá**Zaškrtávacie políčko, ktoré nie je aktívne, je sivé. Odškrtnutím aktívneho políčka sa úplne zastaví zapisovanie daného súboru.**Výstupné produkty LATTICE** (zobrazujú sa pri každom projekte; vzťahujú sa na snímky z LATTICE):

* **Exportovať bez demosaicingu**: lineárny obraz bez debayeringu (`Debayered_Images`): odstránený tmavý posun kalibračného balíka, následne demosaicovaný, bez flat fieldu alebo unmixu — ten istý súbor, aký sa uloží pri snímaní na úrovni bez debayeringu — v natívnej orientácii senzora. Platí pre RGB a multispektrálne moduly.
* **Export náhľadu**: náhľad na displeji (`Preview_Images`). RGB = vyváženie bielej (DAQ-svetelný zdroj, ak je k dispozícii, inak šedý svet) + gama; multispektrálne = rozloženie falošných farieb.
* **Export žiarivosti**: spektrálna žiarivosť typu float32 (`Radiance_Images`, W/m²/sr/nm). Iba multispektrálne moduly – neplatí pre RGB mastery.
* **Export odrazivosti**: odrazivosť typu uint16 (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0), ak snímka obsahuje hodnotu `.daq` smerujúcu nadol alebo cieľ v rámci snímky. Iba multispektrálne moduly.

Všetky štyri sú **štandardne zapnuté**— jeden importovaný surový snímok LATTICE sa v rámci jedného spracovateľského cyklu rozdelí do všetkých povolených a príslušných produktov. Zaškrtávacie políčko**Exportovať odrazivosť** je sivé, ak je kalibrácia odrazivosti vypnutá. Nastavenia, ktorých aktivácia je znemožnená nadradeným prepínačom, sú vždy sivé a obsahujú popisok s názvom prepínača, ktorý je potrebné zmeniť.**Pokročilé nastavenia:*** **Posun časového pásma svetelného senzora**: Počet hodín od UTC pre zosúladenie času svetelného senzora (predvolené: 0, rozsah −12 až +12)
* **Použiť korekcie PPK**: Používa údaje GPS/expozičných pinov zo súborov `.daq` (predvolené nastavenie: vypnuté)
* **Expozičný pin 1/2**: Priraďuje kamery k expozičným pinom pre konfigurácie s dvoma kamerami

{% hint style="info" %}
**Úroveň vstupu LATTICE je automatická.** Snímky zachytené systémom LATTICE obsahujú svoju úroveň spracovania v metadátach XMP a spracovanie vždy vstupuje do spracovateľského reťazca na úrovni surového snímku — v grafickom rozhraní nie je potrebné nič konfigurovať. (Príznak CLI `--input-level` slúži ako prekonfigurácia pre pokročilých používateľov v prípade záznamov so stratenými metadátami; pozrite si [Referenciu CLI](../reference/cli-reference.md).)
{% endhint %}

### Metóda odstránenia bayera

V súčasnosti ponúkame v programe Chloros 2 metódy odstránenia bayera:

#### Štandardná (rýchla, stredná kvalita)

Štandardná metóda odstránenia bayera spracováva rýchlo, ale vykazuje farebný šum spôsobený odstránením bayera, čo vedie k menej presným a šumovejším obrazom.

#### S ohľadom na textúru (pomalá, najvyššia kvalita) \[Iba Chloros+]

Metóda s ohľadom na textúru využíva vysokokvalitný debayer zohľadňujúci hrany v kombinácii s modelom odšumovania založeným na AI/ML, ktorý odstraňuje takmer všetok šum spôsobený debayerom. Na spustenie modelu je potrebná pamäť GPU (VRAM): s **7 GB alebo viac VRAM** dokáže spracovávať viacero obrázkov súčasne; pri menej ako 7 GB spracováva jeden obrázok po druhom (čo je citeľne pomalšie). Pozrite si [Dynamickú adaptáciu výpočtov](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**Zábery LATTICE vždy používajú štandardnú demosaic.** Neexistuje žiadny model Texture Aware vytrénovaný pre LATTICE, preto táto možnosť nie je k dispozícii pre snímky LATTICE — snímky Survey3 v tom istom projekte ju však môžu naďalej používať.
{% endhint %}

### Index (multispektrálne indexy)

Nastavte, ktoré vegetačné indexy sa majú vypočítať a exportovať. Rozbaľovacie menu v grafickom rozhraní ponúka **27 preddefinovaných vzorcov indexov**.**Ako pridať indexy:**

1. Kliknite na tlačidlo**„Pridať index“**

2. Z roletového menu vyberte index (NDVI, NDRE, GNDVI atď.)
3. Nastavte parametre vizualizácie (farby LUT, rozsahy hodnôt)
4. Podľa potreby pridajte viacero indexov

**Populárne indexy:*** **NDVI**: Všeobecný zdravotný stav vegetácie (najbežnejší)
* **NDRE**: Včasná detekcia stresu pomocou RedEdge
* **GNDVI**: Citlivý na koncentráciu chlorofylu
* **OSAVI**: Funguje dobre pri viditeľnej pôde
* **EVI**: Oblasti s vysokým indexom listovej plochy (LAI)

**Vlastné vzorce:**

* Vytvorte vlastné vzorce multispektrálnych indexov pomocou matematických operácií so všetkými kanálmi snímky
* Uložte si vlastné vzorce na opätovné použitie
* Vlastné vzorce sú funkciou Chloros+; ich dostupnosť závisí od úrovne vášho plánu

Všetky dostupné indexy a vzorce — vrátane informácií o tom, ktoré názvy sú dostupné len v grafickom rozhraní (GUI) a ktoré fungujú aj v CLI alebo SDK — nájdete v [Vzorcoch multispektrálnych indexov](../project-settings/multispectral-index-formulas.md).

### Export

Určuje formát výstupného súboru.

**Dostupné formáty**(nastavenie:**Formát kalibrovaného snímku**, predvolené nastavenie**TIFF (16-bit)**):

* **TIFF (16-bit)**: Odporúčané pre GIS a vedeckú analýzu
* **TIFF (8-bit)**: 8-bitový TIFF
* **TIFF (32-bit, Percent)**: Hodnoty s plávajúcou desatinnou čiarkou
* **PNG (8-bit)**: Bezstratová kompresia pre vizualizáciu
* **JPG (8-bit)**: Najmenšie súbory, stratová kompresia

Zoznam uvádza, aké formáty môžu obsahovať obrázky v projekte: Chloros nezvyšuje bitovú hĺbku. Všetky 8-bitové obrázky ponúkajú iba formáty TIFF (8-bit), PNG (8-bit) a JPG (8-bit), pričom predvoleným formátom je TIFF (8-bit). V zmiešanom projekte sa každý záznam exportuje vo svojej vlastnej hĺbke alebo v nižšej. Radiance zostáva 32-bitové s plávajúcou desatinnou čiarkou.

Výstupy sa ukladajú do projektovej zložky, zoskupené podľa kamery a formátu: `<project>/<camera>/<format>/<Product>_Images/`. Radiance sa **vždy** ukladá ako float32 do zložky `tiff32` bez ohľadu na toto nastavenie. Exportované súbory si zachovávajú názov zdrojového súboru – produkt identifikuje zložka. Multispektrálne exporty LATTICE môžu tiež zapisovať masky obmedzenia na úrovni jednotlivých pixelov do zložky `clamp_masks/` vedľa zložiek s produktmi. Úplnú štruktúru výstupných súborov nájdete v časti [Dokončenie spracovania](finishing-the-processing.md).

{% hint style="warning" %}
**Čítanie hodnôt odrazivosti**: hodnota DN, ktorá zodpovedá ρ = 1,0, závisí od zdrojovej kamery — LATTICE používa 32768 (označené ako XMP `Chloros:PixelScale`), Survey3 používa 65535. Prečítajte si túto značku namiesto predpokladania konštantnej hodnoty. Pozrite si [Formáty výstupných obrázkov](../output-image-formats.md).
{% endhint %}

### Svetelný senzor DAQ

V tejto časti je uvedený zoznam všetkých súborov DAQ s dopadajúcim svetlom (`.daq` / `.csv`) vo vašom projekte, jeden riadok na súbor, s uvedením modelu senzora, názvu súboru a platnej korekcie difúzora **cap** platnú pre daný súbor.

* **Prepísanie krytu (všetky súbory)**: jediné roletové menu pre celý projekt.**Auto** (predvolené nastavenie) používa zaznamenanú hodnotu cap pre každý súbor — ak nebolo nič zaznamenané, predpokladá sa slnečné žiarenie, avšak modely DAQ-U, DAQ-M a DAQ-E sa dodávajú bez hodnoty cap a program Chloros nedokáže zistiť, aká hodnota bola nastavená, preto spracovanie vydá varovanie v prípade predpokladanej hodnoty cap: vyberte hodnotu cap, ktorá bola nastavená. Výber limitu prepisuje všetky súbory okrem súborov z DAQ-E-S, ktorých difúzor je súčasťou jeho kalibrácie: surové záznamy sa ním korigujú a záznamy, ktoré už obsahujú limit, sa prekalibrujú (zaznamenaná korekcia sa zruší a uplatní sa vybraný limit).
* Riadky upozorňujú, ak bol zaznamenaný kryt predpokladaným predvoleným nastavením rozbočovača a nie potvrdeným obsluhou, a ak vybraný kryt nemá profil pre daný model zariadenia (pre daný súbor sa prepis odmietne).

Záznamy DAQ vytvorené na karte Svetelné senzory sa automaticky pridajú do otvoreného projektu a importované súbory `.daq` / `.csv` sa tu zobrazia hneď po ich pridaní.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Dolné nastavenia projektu — Index, formát exportu, sekcia DAQ svetelných senzorov a ovládacie prvky šablóny/zložky projektu</p></figcaption></figure>### Zarovnanie poľa

Táto sekcia sa zobrazí **len** vtedy, ak aspoň jeden obrázok v projekte obsahuje transformáciu zarovnania medzi modulmi, ktorú polia LATTICE vtlačia pri snímaní (`Chloros:Alignment*` XMP). Ukazuje, koľko snímok obsahuje značky a ktorá kamera slúži ako referenčná, s týmito ovládacími prvkami:

* **Použiť zarovnanie poľa** (predvolené: zapnuté): deformuje každý spracovaný výstup (odstránenie matice / náhľad / radiancia / odrazivosť / index) do spoločnej referenčnej geometrie poľa. Vypnuté = export v natívnej geometrii senzora.
* **Orezanie na spoločný prekrývací úsek** (predvolené: zapnuté): oreže zarovnané exporty na oblasť, ktorú zdieľajú všetky moduly, takže každé pásmo má rovnakú stopu. Vypnuté zachováva plnú plochu snímača (čierna výplň mimo zdrojového obrazu).
* **Prevzorkovanie**:**Bilineárne (hladké, predvolené)**,**Najbližšie (zachovanie presných hodnôt)**— bez miešania medzi pixelmi, pre prísnu radiometrickú analýzu — alebo**Kubické (najostrejšie)**.***

## Ukladanie a načítanie nastavení

### Uloženie šablóny projektu

Vytvorte opakovane použiteľné šablóny pre konzistentné pracovné postupy:

1. Nakonfigurujte všetky požadované nastavenia v paneli Nastavenia projektu
2. Prejdite do sekcie **„Uložiť šablónu projektu“** v spodnej časti
3. Zadajte výstižný názov šablóny (napr. „Survey3N_RGN_Agriculture“)
4. Kliknite na ikonu uloženia

**Výhody:**

* Používajte identické nastavenia vo viacerých projektoch
* Zdieľajte konfigurácie s členmi tímu
* Zachovajte konzistentnosť pri opakovaných prieskumoch

### Načítanie šablóny do nového projektu

Pri vytváraní nového projektu:

1. V hlavnom menu vyberte **„Nový projekt“**

2. V voliteľnom výbere šablón vyberte šablónu projektu
3. Všetky nastavenia zo šablóny sa automaticky uplatnia

### Pracovný adresár

Nastavenie **„Pracovný adresár“** určuje, kde sa nové projekty vytvárajú štandardne:

* **Predvolené umiestnenie**: `Documents\MAPIR\Chloros Projects` vo vašej zložke Dokumenty v systéme Windows (zvyčajne `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`; sleduje zložku Dokumenty, ak ju presunie OneDrive alebo roamingový profil)
* **Zmeniť umiestnenie**: Kliknite na ikonu úpravy a vyberte nový priečinok
* **Zdieľané s CLI**: `chloros-cli` používa rovnaké predvolené nastavenie priečinka projektu
* **V systéme Windows používajte krátke názvy**: pokiaľ nie sú v systéme Windows povolené dlhé cesty, zložka projektu s názvom dlhším ako približne 130 znakov bude pred spracovaním odmietnutá — pozri [Spustenie spracovania](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)
* **Kedy zmeniť**:
  * Sieťová jednotka pre tímovú spoluprácu
  * Iná jednotka s väčším úložným priestorom
  * Prehľadná štruktúra zložiek podľa roka/klienta

***

## Nastavenie PPK (Post-Processed Kinematic)

Ak používate záznamníky MAPIR DAQ s GPS na presnú geolokalizáciu:

### Predpoklady

* Zariadenie MAPIR DAQ s modulom GPS (GNSS)
* Protokolový súbor .daq so záznamami o expozičných pinách
* Fotoaparát pripojený k expozičným pinom zariadenia DAQ počas snímacej relácie

### Kroky konfigurácie

1. Umiestnite protokolový súbor .daq do zložky projektu
2. V nastaveniach projektu zaškrtnite políčko **„Použiť korekcie PPK“**

3. V prípade potreby nastavte**„Časový posun svetelného senzora“** (predvolené nastavenie: 0 pre UTC)
4. Priraďte kamery k expozícijným pinom:
   * **Jedna kamera**: Automaticky priradená k pinu 1
   * **Dve kamery**: Ručne priraďte každú kameru k správnemu pinu**Priradenie expozícií:*** **Expozícia 1**: Z roletového menu vyberte model kamery
* **Expozícia 2**: Vyberte druhú kameru alebo možnosť „Nepoužívať“
* Na oba piny nemožno priradiť tú istú kameru

{% hint style="warning" %}
**Dôležité**: Piny expozície musia byť správne priradené k príslušným kamerám. Nesprávne priradenie bude mať za následok nesprávne údaje o geolokácii.
{% endhint %}

***

## Pokročilé scenáre

### Projekty s viacerými kamerami

Pri spracovaní snímok z viacerých kamier MAPIR v jednom projekte:

1. Chloros automaticky rozpozná každý model kamery (Survey3 aj LATTICE)
2. Každá kamera dostane príslušné profily spracovania a každá kamera má vlastnú štruktúru výstupných priečinkov
3. PPK: Ručne priraďte každú kameru Survey3 k správnemu expozíciovému pinu
4. Všetky kamery používajú rovnaký formát exportu a indexy

**Príklady**: Dvojkamera Survey3W RGN + Survey3N OCN, alebo sústava LATTICE kombinujúca hlavnú RGB kameru s úzkopásmovými modulmi

### Časozberné snímky alebo merania z viacerých dátumov

Pri opakovaných meraniach tej istej oblasti v priebehu času:

1. Vytvorte šablónu so svojimi štandardnými nastaveniami
2. Pri každej relácii používajte konzistentné nastavenie kalibračných cieľov
3. Každý dátum spracujte ako samostatný projekt
4. Pre porovnateľné výsledky používajte identické nastavenia
5. Exportujte v rovnakom formáte pre časovú analýzu

### Veľké súbory údajov

Pre projekty s veľkým počtom snímok (500 a viac):

* Zvážte rozdelenie na menšie projekty podľa dátumu alebo oblasti
* Pre rýchlejšie výsledky využite paralelné spracovanie v programe Chloros+
* Zvážte použitie rozhrania CLI alebo API na automatizáciu hromadného spracovania
* Upravte minimálny interval rekalibrácie, aby ste skrátili čas detekcie kalibračných cieľov

***

## Overenie nastavení

Pred začatím spracovania skontrolujte tieto kľúčové nastavenia:

* [ ] Model fotoaparátu bol správne rozpoznaný v prehliadači súborov
* [ ] Korekcia vinety je zapnutá
* [ ] Kalibrácia odrazivosti je zapnutá
* [ ] Pre Survey3: bol importovaný a skontrolovaný aspoň jeden obrázok kalibračného cieľa; pre LATTICE: je prítomný cieľ a/alebo záznam smerujúci nadol typu `.daq`
* [ ] Pridané požadované multispektrálne indexy
* [ ] Formát exportu vhodný pre váš pracovný postup
* [ ] Nastavenia PPK nakonfigurované (ak používate súbory .daq s udalosťami expozície)

***

## Ďalšie kroky

Po nakonfigurovaní nastavení:

1. **Označte kalibračné cieľové snímky** – pozri [Výber cieľových snímok](choosing-target-images.md)
2. **Spustite spracovanie** – pozri [Spustenie spracovania](starting-the-processing.md)
3. **Sledujte priebeh spracovania** – pozrite si [Sledovanie spracovania](monitoring-the-processing.md)

Úplné podrobnosti o všetkých dostupných nastaveniach nájdete v referenčnej dokumentácii [Nastavenia projektu](../project-settings/project-settings.md).

# Mriežka obrázkov

Po importovaní obrázkov do projektu sa vám v hlavnej oblasti zobrazia usporiadané do mriežky. V tejto mriežke si vyberáte, **ktorú verziu každého obrázka si práve prezeráte** — tlačidlá nad ňou prepínajú všetky náhľady naraz medzi zdrojovými súbormi a jednotlivými spracovanými výstupmi.

## Veľkosť náhľadov

Pomocou posuvníka zväčšenia v pravom hornom rohu môžete nastaviť veľkosť náhľadov obrázkov. Rozsah posuvníka je od **64 px do 1200 px**.

* **Kombinácia klávesov Ctrl + koliesko myši** tiež umožňuje zmeniť mierku náhľadov.
* **Ctrl + `+`**/**Ctrl + `=`**a**Ctrl + `−`** menia veľkosť o 4 px pri každom stlačení. Rozsah klávesových skratiek končí na 64 px na spodnej hranici a na hornej hranici pri veľkosti, ktorá presne pojme dva náhľady v jednom riadku v aktuálnom okne.
* Veľkosť, ktorú si zvolíte, sa uloží spolu s projektom (`UI → Grid thumbnail size` v `project.json`, predvolené nastavenie `160`), takže pri opätovnom otvorení projektu sa obnoví.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>*Rozlíšenie* náhľadov je odlišné nastavenie od *veľkosti* náhľadov: pozri **Zobrazenie → Rozlíšenie náhľadov obrázkov** v [Nastaveniach projektu](../project-settings/project-settings.md) (predvolené 512 px na dlhšej strane). Veľkosť určuje, ako veľká sa dlaždica nakreslí; rozlíšenie určuje, koľko detailov sa na jej vyplnenie načíta.***

## Nástrojová lišta mriežky

Rad tlačidiel nad mriežkou má až tri skupiny, zľava doprava:

1. **Podľa spúšťača / Podľa kamery** — režim zoskupovania. Zobrazuje sa len v projektoch obsahujúcich záznamy z LATTICE.
2. **Tlačidlá filtrov kamier** — po jednom pre každú kameru LATTICE. Zobrazujú sa len v režime *Per Camera*.
3. **Tlačidlá režimu exportu/zobrazenia** — určujú, ktorý produkt zobrazuje každá miniatúra.

Ak je okno príliš úzke na to, aby sa vošli všetky, skupiny sa zľava doprava zložia do rozbaľovacích ponúk pri nabehnutí kurzora: najskôr sa zložia tlačidlá exportu/zobrazenia, potom tlačidlá kamier. Zložená skupina zanechá jedno tlačidlo označené aktuálne aktívnou voľbou a pri nabehnutí kurzora sa celá sada posunie nadol. **Režimy „Podľa spúšťača“ a „Podľa kamery“ sa nikdy nezatvoria.

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

*****

## Tlačidlá pre export a zobrazenie

Tieto tlačidlá prepínajú miniatúry v mriežke medzi typmi obrázkov. **Tlačidlo sa zobrazí hneď, ako existuje produkt, ktorého názov nesie** — čo v prípade zdrojových súborov znamená ihneď pri importe, nie až po spracovaní. Chloros počas prebiehajúceho spracovania opätovne skenuje produkty projektu, takže tlačidlá sa objavujú počas spracovania, keď sa každý produkt začne ukladať na disk.

### Základné tlačidlo

Tlačidlo exportu úplne vľavo je označené podľa toho, **čo ste skutočne importovali**:

| Čo ste importovali | Názov tlačidla |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| Zábery z LATTICE s náhľadom na displeji vedľa surového snímku | `PNG` alebo `TIFF`, podľa toho, ktoré sú náhľady |
| Snímky LATTICE, kde základným súborom **je** surový snímok | *žiadne tlačidlo* — `RAW (Original)` už tento súbor zobrazuje |

V zmiešanom projekte sa názov riadi príponou, ktorú používa väčšina obrázkov.

### Tlačidlá produktov

| Tlačidlo | Zobrazuje | Kedy sa zobrazí |
| --- | --- | --- |
| **Ciele** | Obrázky s detekovaným kalibračným cieľom | Po spustení, ktoré detekovalo ciele |
| **Odrazivosť** | Kalibrované snímky odrazivosti | Iba v projektoch Survey3 — projekty LATTICE používajú namiesto toho `RAW (Reflectance)`, takže mriežka nikdy nezobrazuje dve tlačidlá odrazivosti |
| **Vyváženie bielej** | Výstup s vyváženou bielou (kamery RGB) | Po spracovaní |
| **Korekcia vinety** | Neokalibrovaná záložná hodnota s korekciou vinety | Po spustení, pri ktorom nebolo možné použiť kalibráciu odrazivosti a bola zapnutá *korekcia vinety* |
| **Odozva snímača** | Nekalibrovaná náhradná verzia s odozvou snímača | To isté, ale s vypnutou *korekciou vinetácie* |
| **`RAW (<INDEX> Index)`** | Jedno tlačidlo na každý vypočítaný index | Po spustení s nakonfigurovanými indexmi |
| **`<INDEX> LUT`** | Jedno tlačidlo na každý index s farebným mapovaním | Po spustení s nakonfigurovanou LUT |
| **`<Index> <Index\|LUT> <NNN>`** | Jedno tlačidlo na každý beh exportu [Index/LUT Sandbox](index-lut-sandbox.md) | V okamihu dokončenia exportu do sandboxu |

### Tlačidlá úrovne LATTICE

Projekty obsahujúce zábery z LATTICE pridávajú tieto tlačidlá, ktoré sú označené názvom úrovne namiesto názvu produktu:

| Tlačidlo | Úroveň |
| --- | --- |
| **RAW (Pôvodný)** | Zdrojový surový snímok, tak ako bol importovaný |
| **RAW (Radiance)** | Spektrálna žiarivosť typu Float32, W/m²/sr/nm |
| **RAW (Reflectance)** | Odrazivosť typu uint16, 32768 = ρ 1,0 |

`RAW (Original)` je k dispozícii ihneď po importe — nevyžaduje žiadne spracovanie. Ak import LATTICE nemá žiadne základné tlačidlo (základným súborom každého záznamu je jeho surový snímok), mriežka sa presunie na prvé dostupné tlačidlo úrovne, aby zvýraznenie na paneli nástrojov zodpovedalo tomu, čo vidíte.

Dvojúrovňové exporty Chloros nemajú **žiadne vlastné tlačidlo mriežky**:

* **Debayered** — zobrazenie `RAW (Original)` sa už vykresľuje bez bayeresového filtra, takže druhé tlačidlo na vizuálne identickom obrázku by bolo zbytočné. Produkt `RAW (Debayered)` sa stále ukladá na disk a je stále možné ho vybrať z roletového menu vrstiev na celej obrazovke.
* **Náhľad** — na kamerách typu RGB je náhľad zaregistrovaný ako vrstva `White Balanced`, ktorá má vlastné tlačidlo. Na multispektrálnych kamerách je zaregistrovaný ako `RAW (Preview)` a je dostupný z roletového menu vrstiev na celej obrazovke.

{% hint style="info" %}
Tieto tlačidlá úrovne sa zobrazia len v projektoch, ktoré skutočne obsahujú snímky LATTICE. Projekty typu Survey3 registrujú niektoré z tých istých interných názvov vrstiev a tlačidlá sú pre ne odfiltrované, takže mriežka typu Survey3 si zachováva svoju známu sadu `JPG / Targets / Reflectance`.
{% endhint %}

Kliknutím na miniatúru mriežky otvoríte [Prehliadač obrázkov](opening-an-image-full-screen.md) na celej obrazovke pre **ten istý produkt, ktorý mriežka zobrazuje** — ak je mriežka nastavená na `Targets`, miniatúra otvorí exportovaný cieľový obrázok.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## Zoskupovanie projektu LATTICE: Podľa spúšťača vs. podľa kamery

Záznamy z matice vytvárajú niekoľko obrázkov toho istého okamihu z rôznych kamerových modulov. Zoskupovanie určuje, ako ich mriežka usporiada. V oboch režimoch sa zobrazujú zrolovateľné hlavičkové lišty v plnej šírke; **každá skupina sa otvára v rozbalenom stave** a Chloros si pamätá tie, ktoré ste zatvorili. Stav zrolovania sa sleduje samostatne pre každý režim, takže zatvorenie skupiny v režime „Podľa kamery“ nezatvorí nič v režime „Podľa spúšťača“.

### Podľa kamery (predvolené)

Jedna skupina na jeden kamerový modul. V hlavičke sa zobrazuje model kamery, sériové číslo (`LATT-M3M — <serial>`) a počet fotografií. Obrazy v rámci skupiny sú zoradené chronologicky podľa času snímania.

V tomto režime sa na paneli nástrojov zobrazí aj jedno **tlačidlo filtrovania kamery na každú kameru** s označením `MODEL (SERIAL)`. Všetky kamery sú spočiatku vybrané; kliknutím na tlačidlo zrušíte výber danej kamery a odstránite jej skupinu z mriežky. Ide o rýchly spôsob prehliadania jedného pásma počas celého letu.

### Podľa spúšťača

Jedna skupina na jednu udalosť zachytenia — súbor snímok, ktoré všetky moduly nasnímali pri rovnakom spúšťači. V hlavičke sa zobrazuje čas zachytenia, počet kamier, ktoré sa na ňom podieľali, a ikona pre každý model kamery v skupine. Dlaždice v rámci skupiny sú zoradené podľa sériového čísla kamery, takže rovnaké spektrálne pásmo sa nachádza v rovnakom stĺpci pre každý spúšťací impulz.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
Snímky, ktoré nie sú v formáte LATTICE, v zmiešanom projekte nie sú zoskupené — zobrazujú sa ako bežné dlaždice za skupinami.

***

## Miniatúry v mriežke zodpovedajú veľkosti bloku GSD

Ak ste v bočnom paneli na karte „obrázok“ nastavili veľkosť bloku **GSD (px)**, miniatúry v mriežke sa zobrazujú v tom istom rozlíšení na zemi – nielen v zobrazení na celú obrazovku. Veľkosť bloku 8 znamená, že každý zobrazený pixel je priemerom bloku 8 × 8 zdrojových pixelov, a to všade v aplikácii, kde sa obrázok zobrazuje.

Keďže dlaždica má už od začiatku šírku len niekoľko sto pixelov, hrubé veľkosti blokov prestávajú mať viditeľný vplyv na mriežku oveľa skôr, ako je to v režime celej obrazovky: rámec s rozmerom 4000 px nakreslený do dlaždice s rozmerom 160 px už predstavuje približne 25 zdrojových pixelov na jeden zobrazený pixel. Pozrite si [Otvorenie obrázku na celej obrazovke](opening-an-image-full-screen.md#gsd-block-size) pre samotné ovládacie prvky.

***

## Súvisiace stránky

* [**Otvorenie obrázku na celú obrazovku**](opening-an-image-full-screen.md) — prehliadač na celú obrazovku, hodnoty kurzora a histogram
* [**Vrstvy obrázku**](image-layers.md) — roletové menu vrstiev v prehliadači na celej obrazovke
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — vytváranie a export vizualizácií indexov
* [**Nastavenia projektu**](../project-settings/project-settings.md) — prepínače exportu, ktoré určujú, ktoré produkty vôbec existujú

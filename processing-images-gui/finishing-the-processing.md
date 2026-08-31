# Dokončenie spracovania

Akonáhle proces Chloros dokončí spracovanie, je čas skontrolovať výsledky, overiť kvalitu výstupu a pripraviť spracované obrázky na použitie vo vašom pracovnom postupe. Táto stránka vás prevedie záverečnými krokmi a ďalšími úkonmi.

## Upozornenie na dokončenie spracovania

Po úspešnom dokončení spracovania uvidíte niekoľko indikátorov:

* ✅ **Indikátor priebehu**: Dosiahne 100 % dokončenia
* ✅ **Log ladiaceho režimu**: Zobrazuje posledné riadky programu `[RUN-SUMMARY]` s počtami (obrázky, skupiny kamier, ciele, kalibrované obrázky, zapísané súbory)
* ✅ **Tlačidlo Štart**: Znovu sa aktivuje (pripravené na ďalšie spustenie spracovania)
* ✅ **Výstupné súbory**: Všetky spracované snímky sú uložené do výstupnej zložky projektu (nižšie)

{% hint style="warning" %}
**Spustenie, pri ktorom sa nezapíšu žiadne snímky, sa považuje za zlyhanie.** Ak ste požiadali o obrazové produkty a beh nezaznamenal žiadny, Chloros to nahlási ako zlyhanie — `[RUN-SUMMARY]` naznačuje v názve protokolu pravdepodobnú príčinu (nebolo importované nič, nebol zistený žiadny cieľ alebo bol každý požadovaný produkt vynechaný ako nepoužiteľný). Ekvivalent CLI sa ukončí s nenulovým výsledkom. Úmyselné spustenie iba s metadátami (všetky exportované produkty vypnuté, žiadne indexy) sa stále považuje za úspešné. Pozrite si [Referenciu CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Vyhľadanie spracovaných obrázkov

### Otvorenie výstupného priečinka

1. Kliknite na ikonu **Hlavné menu** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (vľavo hore)
2. Zvoľte **„Otvoriť priečinok projektu“**

3. Otvorí sa vám prehliadač súborov s adresárom projektu
4. Vyhľadajte svoj projekt podľa názvu

### Výstupná štruktúra

Výstupné súbory sa ukladajú **do zložky projektu, zoskupené podľa fotoaparátu a následne podľa formátu súboru**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Zložka fotoaparátu**: `LATT-<sensor>-<lens>-F<filter>` pre LATTICE (zodpovedá EXIF záznamu `Model`), `<model>_<filter>` pre Survey3 (napr. `Survey3N_RGN`). Dve kamery, ktoré majú spoločný snímač a filter, ale líšia sa objektívom, majú oddelené stromy — líšia sa vinetáciou, zorným poľom a skreslením.
* **Zložka formátu**: riadi sa nastavením formátu exportu — `tiff16`, `tiff8`, `png8`, `jpg8` alebo `tiff32` pre TIFF (32-bitové, v percentách). Žiarivosť je vždy typu float32 a vždy sa ukladá pod `tiff32`.
* **Zložky produktov**:
  * `Reflectance_Calibrated_Images/` — kalibrovaná odrazivosť
  * `Debayered_Images/` — lineárne odstránenie bayeresovho efektu (LATTICE)
  * `Preview_Images/` — náhľad na displeji (LATTICE)
  * `Radiance_Images/` — spektrálna žiarivosť typu float32, W/m²/sr/nm (multispektrálna technológia LATTICE)
  * `Vignette_Corrected_Images/` **alebo** `Sensor_Response_Images/` — nekalibrovaná náhradná hodnota pre snímky bez referenčnej odrazivosti; v každom behu existuje presne jedna z týchto dvoch možností, vybraná nastavením korekcie Vignette
  * `<INDEX>_Index_Images/` — jeden priečinok na každý vybraný index (napr. `NDVI_Index_Images`)

{% hint style="info" %}
**Každý exportovaný produkt si zachováva názov zdrojového súboru.**Export radiancie súboru `capture_..._raw.tif` sa stále nazýva `capture_..._raw.tif` — len sa nachádza v priečinku `tiff32/Radiance_Images/`.**Produkt identifikuje zložka, nie názov súboru**, takže vyhľadávanie súboru `*radiance*.tif` nič nenájde; namiesto toho vyhľadávajte podľa zložky.
{% endhint %}



<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-… camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images) and tiff32 (Radiance_Images) subfolders visible -->### Koľko súborov by malo byť?

Nepočítajte podľa vzorca – počet výstupov závisí od toho, ktoré produkty boli aktivované a ktoré sa vzťahujú na jednotlivé kamery (napr. kamery RGB nemajú údaje o žiarivosti/odrazivosti). Konečný počet je uvedený v protokole: posledný riadok `[RUN-SUMMARY]` uvádza presný počet súborov, ktoré boli zapísané, a riadky s tipmi vysvetľujú všetko, čo bolo vynechané.

***

## Prezeranie spracovaných obrázkov

### Rýchly náhľad v Průzkumníku súborov

**Vstavaný náhľad Windows:**

1. Prejdite do zložky produktu (napr. `tiff16/Reflectance_Calibrated_Images/`)
2. Vyberte súbor s obrázkom
3. Náhľad sa zobrazí v paneli náhľadu Průzkumníka Windows
4. Pomocou klávesov so šípkami prechádzajte medzi obrázkami

### Náhľad v externých prehliadačoch obrázkov

**Odporúčané prehliadače:*** **QGIS** – bezplatný GIS softvér (najlepší pre georeferencovanú multispektrálnu analýzu)
* **IrfanView** – rýchly a nenáročný prehliadač obrázkov (podporuje TIFF)
* **Adobe Photoshop** – profesionálna úprava (podpora TIFF)
* **GIMP** – bezplatná alternatíva k programu Photoshop
* **Windows Photos** – základné prezeranie (nemusí podporovať 16-bitový formát TIFF)

### Náhľad v prehliadači obrázkov Chloros

Na pokročilú vizualizáciu použite integrovaný prehliadač obrázkov Chloros:

1. Kliknite na miniatúru obrázku v prehliadači súborov
2. Obrázok sa otvorí v hlavnej oblasti náhľadu
3. Kliknite na kartu **Prehliadač obrázkov** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> v ľavom bočnom paneli
4. Na interaktívnu analýzu použite [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md)

Podrobné pokyny nájdete v časti [Prehliadač obrázkov](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Čítanie hodnôt odrazivosti pixelov (GIS / Pix4D / skripty)

Odrazivosť je uložená ako celé číslo DN a **hodnota DN, ktorá zodpovedá ρ = 1,0, závisí od zdrojovej kamery**:

| Zdroj          | ρ = 1,0 je | Ako zistiť                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (rezerva až do ρ 2,0) | Do súboru je vložená značka XMP `Chloros:PixelScale=32768` |
| Survey3         | **65535** (obmedzené na ρ 1,0)     | Žiadne XMP tagy `Chloros:*` — táto absencia je signálom |

**Prečítajte značku `Chloros:PixelScale` a vydelite ňou** namiesto toho, aby ste predpokladali paušálnu hodnotu 65535 — vydelením odrazivosti LATTICE číslom 65535 sa každá hodnota bez upozornenia zníži na polovicu. Jeden okrajový prípad nemá podľa návrhu žiadnu mierku: 8-bitový záznam zo zdroja zapísaný ako 8-bitový výstup je orezaný, nie preškálovaný, a zámerne nemá žiadnu značku mierky — namiesto delenia ho znovu exportujte v 16-bitovom alebo 32-bitovom formáte. Úplné vysvetlenie nájdete v časti [Formáty výstupných obrázkov](../output-image-formats.md).***

## Metadáta prenášané do exportov

Každý produkt zachováva **GPS blok**zdrojového záznamu a jeho**EXIF sub-IFD**, takže
export obsahuje `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` a
`CameraSerialNumber`, ako aj georeferencovanie.

{% hint style="warning" %}
**Ak sa ortomozaika vygeneruje v absurdnom meradle, najskôr skontrolujte `FocalLength`.**
Pix4D vypočíta vzdialenosť vzorky od zeme na základe ohniskovej vzdialenosti a nadmorskej výšky. Bez tejto značky sa
vráti k úplne nesprávnemu mierku — pri jednom meranom lete so 49 zábermi bol pomarančový háj s rozmermi 411 m × 160 m
rekonštruovaný ako 47,8 km × 13 km, čo viedlo k vytvoreniu ortofotomapy s rozlíšením 455 megapixelov, ktorá zobrazovala prevažne
prázdny priestor. Pomalé vytváranie dlaždíc a neočakávane veľký súbor sú príznakmi tohto problému, nie samostatnými
problémami.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Nie *všetky* tagy sa kopírujú. Štrukturálne tagy IFD0 sa zámerne vynechávajú (ich kopírovanie
poškodzuje výstup LATTICE) a `ExifImageWidth` / `ExifImageHeight` sú vylúčené,
pretože opisujú pôvodný záznam – export, ktorého rozmery boli zmenené, by inak
uvádzal rozmery, ktoré sú v rozpore s jeho vlastným rastrovým obrazom.

***

## Prezeranie ladiaceho protokolu

### Kontrola varovaní alebo chýb

1. Otvorte kartu **Log ladenia** v<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">
2. Prejdite správami
3. Hľadajte žlté varovania alebo červené chyby
4. Prečítajte si riadky `[RUN-SUMMARY]` a všetky rady
5. Kontaktujte podporu MAPIR a požiadajte o pomoc

### Uloženie protokolu

Ak chcete uchovať záznam o spracovaní alebo ho odoslať technickej podpore MAPIR:

1. Kliknite na tlačidlo **„Kopírovať“**alebo**„Stiahnuť“**

2. Uložte ho ako textový súbor do priečinka projektu
3. Priložte ho k dokumentácii projektu
4. V prípade problémov ho pošlite technickej podpore MAPIR

***

## Bežné problémy s výstupom a ich riešenia

### Problém: Chýbajúce výstupné súbory

**Možné príčiny:**

* Produkt sa na danú kameru nevzťahuje (napr. radiancia/reflexia pre kamery RGB – uvádza to protokol)
* Chýbala povinná referenčná hodnota (napr. odrazivosť bez cieľa a bez dopadajúceho žiarenia `.daq`)
* Zaškrtávacie políčko pre export produktu bolo v nastaveniach projektu deaktivované
* Počas exportu došlo k vyčerpaniu miesta na disku

**Riešenia:**

1. Skontrolujte rady v lógovom súbore Debug Log s označením `[RUN-SUMMARY]` a riadky s označením `[EXPORT-CHECK]` – vysvetľujú vynechania jednotlivých kamier
2. Skontrolujte zaškrtnutie políčok pre export produktov v [Nastaveniach projektu](adjusting-project-settings.md)
3. Skontrolujte, či bolo na disku dostatok miesta
4. Po odstránení príčiny spustite spracovanie znova

### Problém: Tmavé alebo svetlé okraje (vignettovanie je stále viditeľné)

**Možné príčiny:**

* Korrekcia vinetácie je vypnutá
* Kamera/objektív nie je v databáze profilov Chloros
* Extrémna vinetácia presahujúca možnosti korekcie

**Riešenia:**

1. Overte, či bola korekcia vinetácie zapnutá v nastaveniach projektu
2. Skontrolujte, či bol správne rozpoznaný model kamery
3. Ak vinetácia pretrváva, kontaktujte podporu MAPIR

### Problém: Nesprávne farby alebo hodnoty

**Možné príčiny:**

* Neboli zistené kalibračné ciele
* Bol vybraný nesprávny model kalibračného cieľa
* Kalibrácia odrazivosti je vypnutá
* Nízka kvalita obrázkov cieľov

**Riešenia:**

1. Overte, či bola povolená kalibrácia odrazivosti
2. Skontrolujte správy „Cieľ nájdený“ v ladiacom protokole
3. Skontrolujte kvalitu obrázkov cieľov
4. Spustite spracovanie znovu so správne označenými cieľmi

### Problém: Hodnoty NDVI sa zdajú byť nesprávne

**Očakávané rozsahy hodnôt NDVI:*** **Voda, skaly, pôda**: -0,1 až 0,2
* **Riedka/nezdravá vegetácia**: 0,2 až 0,4
* **Stredná vegetácia**: 0,4 až 0,6
* **Zdravá, hustá vegetácia**: 0,6 až 0,9**Ak sú hodnoty mimo týchto rozsahov:**

1. Overte, či bola použitá kalibrácia odrazivosti
2. Overte, či bol zahrnutý protokol svetelného senzora
3. Skontrolujte, či boli detekované kalibračné ciele
4. Uistite sa, že bol detekovaný správny model kamery
5. Skontrolujte načasovanie a podmienky snímania cieľového obrazu
6. Ak indexy vypočítavate sami zo súborov odrazivosti, uistite sa, že ste hodnotu vydelili hodnotou `Chloros:PixelScale` uvedenou v súbore (pozri vyššie)

***

## Používanie spracovaných snímok

### Pre fotogrametriu / tvorbu ortomozaiky

**Odporúčaný pracovný postup:**

1.**Importujte kalibrované snímky odrazivosti** do fotogrametrického softvéru:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Zachovajte metadáta EXIF**: Uistite sa, že sú zachované údaje GPS pre geotagovanie
3. **Kalibrované pracovné postupy**: Používajte odrazové snímky pre vedeckú presnosť — odrazové snímky LATTICE obsahujú kalibračné značky XMP, ktoré Pix4D číta
4. **Spracovanie indexových mozaík**: Vytvorte ortomozaiky NDVI z jednotlivých indexových snímok
5. **Exportujte georeferencované snímky GeoTIFF**: Na použitie v GIS aplikáciách

### Pre GIS analýzu

**Odporúčaný pracovný postup:**

1.**Načítajte do QGIS, ArcGIS alebo podobného programu**

2.**Použite 16-bitové snímky odrazivosti TIFF** na multispektrálnu analýzu (vydelené hodnotou `Chloros:PixelScale` uvedenou v súbore)
3. **Použite indexové snímky** (NDVI, NDRE) ako pripravené vrstvy vegetácie
4. **Rastrový kalkulátor**: Kombinujte pásma pre vlastnú analýzu
5. **Export**: Vytvorte klasifikačné mapy, detekciu zmien, mapy zdravotného stavu vegetácie

### Pre priamu analýzu / vykazovanie

**Odporúčaný pracovný postup:**

1.**Použite indexové snímky s farbami LUT** pre vizuálne správy
2. **Extrahujte štatistiky**: Priemerná hodnota NDVI na pole/parcelu
3. **Časové rady**: Porovnajte indexy z viacerých sedení
4. **Vytvorte správy**: Zahrňte mapy, štatistiky a vizualizácie***

## Archivácia a zálohovanie

### Odporúčaná stratégia zálohovania

**Čo uložiť:*** ✅ **Pôvodné snímky vo formáte RAW/JPG alebo surové záznamy LATTICE** – archivujte na samostatný disk/do cloudu; surové údaje sú zdrojom spracovateľského reťazca a všetko ostatné sa z nich dá znovu vygenerovať
* ✅ **Súbory zo svetelných senzorov `.daq` / `.csv`** – potrebné na neskoršie opätovné odvodenie odrazivosti
* ✅ **Spracované výstupy** – Uložte si kalibrované snímky a indexy
* ✅ **Zložka projektu** (`project.json` a súvisiace súbory) – Obsahuje všetky nastavenia potrebné na opätovné spracovanie v prípade potreby
* ✅ **Log ladiaceho procesu** – Dokumentuje podrobnosti spracovania
* ✅ **Obrázky kalibračných terčov** – Na overenie a opätovné spracovanie**Odporúčania pre ukladanie:*** **Okamžitá záloha**: Externý pevný disk
* **Dlhodobý archív**: Úložisko v cloude (Google Drive, Dropbox atď.)
* **Kritické údaje**: Uložte 2–3 kópie na rôznych miestach***

## Ďalšie spracovania

### Opätovné použitie nastavení projektu

Ak budete v budúcnosti spracovávať podobné súbory údajov:

1. **Uložte šablónu projektu** (ak ste tak ešte neurobili)
2. **Vytvorte nový projekt** pomocou uloženej šablóny
3. **Importujte nové obrázky**

4.**Spracujte**s identickými nastaveniami pre zachovanie konzistencie

### Hromadné spracovanie viacerých sedení

V prípade viacerých sedení/datasetov:**Možnosť 1: Grafické rozhranie – viaceré projekty**

* Vytvorte samostatný projekt pre každé sedenie
* Použite konzistentné nastavenia šablóny
* Spracúvajte po jednom

**Možnosť 2: Chloros CLI (len Chloros+)**

* Automatizujte hromadné spracovanie
* Spracujte viacero zložiek pomocou skriptov
* Pozrite si [Dokumentáciu k CLI](../CLI.md) a [Referenčnú príručku k CLI](../reference/cli-reference.md)

**Možnosť 3: Python SDK (len pre verziu Chloros a vyššie)**

* Programové ovládanie
* Integrácia s analytickými procesmi
* Pozrite si [API dokumentáciu](../api-python-sdk.md) a [SDK referenčnú príručku](../reference/sdk-reference.md)

***

## Riešenie problémov pri následnom spracovaní

### Opätovné spracovanie s odlišnými nastaveniami

Ak výsledky nie sú uspokojivé:

1. Zachovajte pôvodné obrázky (nikdy ich nemazajte)
2. Otvorte ten istý projekt v Chloros
3. Upravte nastavenia v paneli Nastavenia projektu
4. Spracujte znovu — výstupy sa uložia do rovnakých produktových priečinkov, takže súbory s rovnakým názvom z predchádzajúceho spustenia budú nahradené

### Spracovanie podmnožiny snímok

Ak chcete opätovne spracovať len konkrétne snímky:

1. Vytvorte nový projekt
2. Importujte len obrázky, ktoré je potrebné znovu spracovať
3. Použite rovnakú šablónu nastavení
4. Spracujte menší súbor údajov

### Pomoc

Ak narazíte na problémy:

* 📧 **E-mail**: info@mapir.camera (priložte ladiaci protokol)
* 🌐 **Podpora**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Často kladené otázky**: [Často kladené otázky](../faq.md)
* 📖 **Dokumentácia**: [Chloros Príručka](../)***

## Zhrnutie: Kompletný pracovný postup

Teraz ste dokončili celý pracovný postup spracovania Chloros:

1. ✅ **Vytvorený projekt** – Pozrite si [Projekty](../projects.md)
2. ✅ **Pridali ste súbory** – pozri [Pridávanie súborov](adding-files-to-a-project.md)
3. ✅ **Upravili ste nastavenia** – pozrite si [Úprava nastavení projektu](adjusting-project-settings.md)
4. ✅ **Označili ste ciele** – pozrite si [Výber cieľových obrázkov](choosing-target-images.md)
5. ✅ **Spustilo sa spracovanie** – pozri [Spustenie spracovania](starting-the-processing.md)
6. ✅ **Sledoval sa priebeh** – pozri [Sledovanie priebehu spracovania](monitoring-the-processing.md)
7. ✅ **Skontrolované výsledky** – Táto stránka**Vaše kalibrované multispektrálne snímky s korekciou odrazivosti sú pripravené na analýzu!**

***

## Ďalšie zdroje

### Pokročilé funkcie

* [**Prehliadač snímok**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktívna vizualizácia a analýza
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – Testovanie vlastných indexov
* [**Vzorce multispektrálnych indexov**](../project-settings/multispectral-index-formulas.md) – Kompletný zoznam indexov

### Automatizácia a integrácia

* [**Dokumentácia k CLI**](../CLI.md) – Hromadné spracovanie z príkazového riadku
* [**Python SDK**](../api-python-sdk.md) – Programová automatizácia
* [**Chloros+ Funkcie**](../#chloros) – Pokročilé možnosti spracovania

### Podpora a vzdelávanie

* [**Často kladené otázky**](../faq.md) – Odpovede na bežné otázky
* [**Kalibračné terče**](../calibration-targets.md) – Vysvetlenie kalibrácie odrazivosti
* [**Podporované kamery**](../supported-cameras.md) – Kompatibilný hardvér

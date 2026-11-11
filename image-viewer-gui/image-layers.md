# Vrstvy obrázkov

**Rozbaľovacie menu vrstiev** v pravom hornom rohu prehliadača obrázkov umožňuje prepínať medzi všetkými verziami obrázku, ktorý si práve prezeráte – od zdrojového záznamu cez každý spracovaný produkt až po vypočítané indexové obrázky – bez toho, aby ste museli opustiť prehliadač.

## Čo sú vrstvy obrázkov?

„Vrstva“ v programe Chloros

je jeden **súbor produktu**priradený k jednému zdrojovému obrázku. Importom získate zdrojové súbory; spracovaním sa pridá vrstva pre každý produkt, ktorý bol vytvorený v rámci daného cyklu. Exportované súbory si zachovávajú názov zdrojového súboru — produkt identifikuje**zložka** a názov vrstvy je označenie tejto zložky v programe Chloros

.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Zoznam vrstiev

### Vždy prítomné

| Vrstva | Čo to je |
| --- | --- |
| **JPG**(alebo**PNG

**/**TIFF

**) | Základný súbor, ktorý bol získaný pri zachytení. Survey3

importuje súbor `.JPG` vedľa každého súboru `.RAW`; zachytenia LATTICE prinášajú náhľad zobrazenia vo formáte PNG

alebo TIFF

. Označené podľa toho, čo bolo skutočne importované |
| **RAW (originál)** | Zdrojový surový snímok, dekódovaný pre zobrazenie bez aplikovaných korekcií. K dispozícii od momentu importu — nevyžaduje žiadne spracovanie |

Záznam z LATTICE, ktorého základným súborom **je** jeho surový snímok, nemá samostatný základný záznam: `RAW (Original)` ho už pokrýva.

### Výstupy spracovania v programe Survey3



| Vrstva | Zapísané do | Existuje, keď |
| --- | --- | --- |
| **RAW (cieľ)** | — | Snímka bola identifikovaná ako obsahujúca kalibračný cieľ |
| **RAW (odrazivosť)** | `Reflectance_Calibrated_Images/` | Na tejto snímke bola úspešne vykonaná kalibrácia odrazivosti |
| **S korekciou vinetácie**| `Vignette_Corrected_Images/` | Snímku nebolo možné kalibrovať podľa odrazivosti**a** bola zapnutá *korekcia vinetácie* |
| **Odozva snímača**| `Sensor_Response_Images/` | Snímku nebolo možné kalibrovať podľa odrazivosti**a** *korekcia vinetácie* bola vypnutá |
| **Vyváženie bielej** | `White_Balanced_Images/` | Bol vytvorený produkt s vyvážením bielej |

{% hint style="info" %}
**Korekcia vinetácie a odozva snímača sú alternatívy, nikdy nie obe naraz.** Na každý model fotoaparátu existuje presne jeden nekalibrovaný náhradný produkt na jedno spustenie a prepínač *Korekcia vinetácie* určuje, ktorý z nich sa použije. Pozri [Nastavenia projektu](../project-settings/project-settings.md).
{% endhint %}

### Úrovne LATTICE

LATTICE zachytáva rozvetvenie do týchto úrovní v jedinom kroku spracovania. Ktoré z nich existujú, závisí od prepínačov exportu pre jednotlivé produkty v Nastaveniach projektu a od toho, čo platí pre daný fotoaparát.

| Vrstva | Zapisuje sa do | Platí pre |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB

a multispektrálne |
| **RAW (náhľad)** | `Preview_Images/` | Multispektrálne (rozloženie falošných farieb) |
| **Vyváženie bielej** | `Preview_Images/` | Hlavné kamery RGB

— náhľad RGB

je zaregistrovaný pod týmto názvom, aby sa zhodoval s vrstvou Survey3

s rovnakým názvom |
| **RAW (žiarivosť)** | `Radiance_Images/` | Iba multispektrálne |
| **RAW (odrazivosť)** | `Reflectance_Calibrated_Images/` | Iba multispektrálne a iba vtedy, ak snímku pokrýva zodpovedajúci záznam smerujúci nadol `.daq` alebo cieľ v rámci snímky, ktorý prešiel kontrolou kvality |

Hlavné kamery RGB

nemajú radiometriu na jednotlivé pásma, preto sa u nich žiarivosť a odrazivosť vynechávajú ako **neaplikovateľné** — protokol to uvádza, namiesto toho, aby došlo k tichému zlyhaniu.

{% hint style="info" %}
**Surové a debayered vrstvy sú zosúladené s produktmi (novinka vo verzii 1.3.0).**Surové a debayered súbory LATTICE sa ukladajú v natívnej orientácii senzora a do XMP zaznamenávajú orientáciu, ktorá k nim patrí; ostatné produkty sa ukladajú s touto orientáciou už aplikovanou. Prehliadač a mriežka obrázkov pre zobrazenie prepínajú medzi**RAW (pôvodný)**a**RAW (s odstráneným bayesovým efektom)** podľa tohto záznamu, takže sa snímka už neotočí, keď prepínate medzi vrstvou produktu a surovou vrstvou. Ide len o zobrazenie — súbory sa nikdy neprepisujú. Súbory bez záznamu o orientácii (zaznamenané pred verziou 1.3.0 alebo vytvorené iným softvérom) sa zobrazujú tak, ako sú uložené. Pozrite si [Formáty výstupných obrázkov](../output-image-formats.md#orientation).
{% endhint %}

Masky obmedzenia (`*_clampmask.tif` v priečinku `clamp_masks/`) nie sú vrstvami: program Chloros

ich pri importe preskakuje. Pozrite si [Formáty výstupných obrázkov](../output-image-formats.md#clamp-masks).

### Vrstvy indexu, LUT a sandboxu

| Vzor vrstvy | Príklad | Odkiaľ pochádza |
| --- | --- | --- |
| **RAW (`<INDEX>` Index)** | `RAW (NDVI Index)` | Jedna na každý index nakonfigurovaný v nastaveniach projektu, vypočítaná počas spracovania |
| **`<INDEX>` LUT** | `NDVI LUT` | Verzia indexu s farebným mapovaním |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Jeden na každý beh exportu [Index/LUT Sandbox](index-lut-sandbox.md) export |

Ak je rovnaký názov indexu nakonfigurovaný viac ako raz s rôznymi nastaveniami, druhý a ďalšie dostanú v názve číslo (`RAW (NDVI2 Index)`), aby sa vrstvy dali rozlíšiť.

***

## Používanie výberu vrstiev

1. Otvorte obrázok na celú obrazovku kliknutím na miniatúru v mriežke
2. Kliknite na **roztiahnuteľné menu vrstiev** v pravom hornom rohu prehliadača
3. Vyberte vrstvu — obrázok sa okamžite aktualizuje

V roletovom menu sú na prvých miestach v uvedenom poradí uvedené formáty **JPG, RAW (originál), RAW (cieľ), RAW (odrazivosť)** a za nimi sú uvedené všetky ostatné formáty v poradí, v akom boli produkty zaregistrované.

### Predvoľba vrstvy pri navigácii

Stlačením klávesov **←**/**→** prejdete na ďalší obrázok a systém sa pokúsi zachovať tú istú vrstvu:

1. **Najskôr presná zhoda** — ak má ďalší obrázok vrstvu s rovnakým názvom, zobrazí sa táto vrstva. Vďaka tomu zostanete na vrstve `RAW (NDVI Index)` aj pri prechádzaní celej sady
2. **Potom zhoda podľa typu** — indexová vrstva hľadá akúkoľvek indexovú vrstvu, LUT akúkoľvek LUT, odrazovú akúkoľvek odrazovú, cieľovú akúkoľvek cieľovú, originálnu akúkoľvek originálnu, základnú akúkoľvek základnú
3. **Potom, iba pre exportné vrstvy** — názov sa zachová aj v prípade, že zoznam vrstiev ešte nie je synchronizovaný, pretože súbor už existuje na disku. Vďaka tomu môžete prehliadať výstupy, zatiaľ čo proces ich ešte zapisuje
4. **V ostatných prípadoch** — prvá dostupná vrstva, ktorou je zvyčajne základný obrázok

Súbory sidecar `.daq` a `.csv` v projekte sa pri navigácii pomocou šípok preskakujú, takže pri prechádzaní obrázkov sa nikdy nedostanete k záznamu zo svetelného senzora.

Zväčšovanie a posúvanie sa prenáša aj medzi obrazmi, čo uľahčuje porovnanie tej istej polohy poľa pred a po.

***

## Porozumenie hodnotám pixelov podľa vrstvy

[Panel Hodnoty kurzora](opening-an-image-full-screen.md#cursor-values) zobrazuje skutočnú hodnotu na kanál pod kurzorom v jednotke, v ktorej je daná vrstva uložená. Jeho stĺpce sa menia v závislosti od vrstvy:

| Vrstva | Zobrazená jednotka | Poznámky |
| --- | --- | --- |
| Základná (náhľad JPG / PNG

/ TIFF

) | DN, 0–255 | Zobrazené hodnoty, s korekciou gama pre RGB

. Iba na vizuálnu kontrolu |
| RAW (originál) | DN | Surové digitálne hodnoty zo senzora. Os histogramu udáva hĺbku: 255 (8-bitová), 4095 (12-bitová) alebo 65535 (16-bitová) |
| RAW (bez bayerského filtra) | DN | Lineárne, bez rozťahovania zobrazenia |
| RAW (náhľad) / Vyváženie bielej | DN | Výsledok zobrazenia — rozťahaný alebo s korekciou gama. Nie je určené na meranie |
| RAW (žiarivosť) | **W/m²/sr/nm** | Fyzikálna žiarivosť typu Float32. Bez stĺpca DN |
| RAW (odrazivosť) | DN **a %** | Percento vypočítané podľa vlastnej stupnice daného súboru — pozri nižšie |
| Exporty indexu / LUT / sandbox | Hodnota indexu alebo zložky RGB

| Jednokanálový indexový súbor uvádza hodnotu indexu; súbor LUT s farebným mapovaním uvádza zložky Red

/Green

/Blue

|

### Odrazivosť: mierka je špecifická pre každý súbor

{% hint style="warning" %}
**„Vydeliť 65 535“ platí len pre Survey3

.** Odrazivosť LATTICE je uložená v inom meradle a zmiešanie týchto dvoch deliteľov je najbežnejším spôsobom, ako získať hodnoty odrazivosti, ktoré sú presne polovičné oproti tomu, aké by mali byť.
{% endhint %}

| Zdroj | DN, ktorý zodpovedá odrazivosti 1,0 | Identifikované podľa |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | XMP značka `Chloros:PixelScale=32768` vložená do každého exportu odrazivosti LATTICE. Dvojnásobná rezerva znamená, že hodnoty ρ nad 1,0 sú reprezentovateľné a nie sú orezané |
| **Survey3**| **65535** | Chýba XMP značka mierky Chloros

— kalibrácia Survey3

zapisuje ρ × dtype-max a orezáva pri hodnote 1,0 |

Pre GIS a skriptovanie: prečítajte z súboru hodnotu `Chloros:PixelScale` a vydelite ňou. Ak značka chýba, súbor má mierku Survey3

(65535). Prehliadač, index/LUT sandbox a export indexu všetky určujú mierku týmto rovnakým spôsobom, takže číslo, ktoré vidíte pri kurzore, je číslo, ktoré použil indexový výpočet.

Ukladanie špecifické pre formát nad rámec tohto meradla:

* **Formát TIFF

(32-bitový, percentuálny)** ukladá hodnotu DN / 65535 ako číslo typu float
* **Formát PNG

(8-bitový)**a**formát JPG (8-bitový)** ukladajú hodnotu DN × 255 / 65535
* **8-bitový DN získaný z 8-bitového snímania** alebo súbor odrazivosti vytvorený pred verziou 1.3.0 je obmedzený na rozsah 0–255 a neobsahuje žiadnu značku mierky. Panel pre tieto súbory zobrazuje iba hodnotu DN, bez stĺpca s percentami

### Rozsahy hodnôt indexov

| Rodina indexov | Typický rozsah | Hodnota |
| --- | --- | --- |
| Normalizovaný rozdiel (NDVI

, GNDVI

, NDRE

, ENDVI…) | −1 až +1 | Zdravá vegetácia zvyčajne 0,4–0,9; holá pôda blízko 0; voda záporná |
| Upravené o pôdu (SAVI

, OSAVI

, MSAVI2…) | približne −1 až +1,5 | Podobné hodnoty ako NDVI

s potlačeným pôdnym pozadím |
| Pomer (GRVI

, GCI

, MSR, CIRE…) | neobmedzené nad | Pomery rastú bez obmedzenia, keď sa pásmo v menovateli blíži k nule |
| EVI

/ LAI

| 0 až ~1, 0 až ~3,5 | Oblaky a iné nasýtené pixely posúvajú obe hodnoty mimo rozsah — najprv ich zamaskujte |

Presné vzorce pre každé prednastavenie nájdete v [Vzorcoch multispektrálnych indexov](../project-settings/multispectral-index-formulas.md).

***

## Bežné pracovné postupy

### Porovnanie pred a po

1. Vyberte **RAW (originál)** a všimnite si vinetáciu a nekalibrované hodnoty
2. Prepnite na **RAW (Reflectance)**

3. Porovnajte — vinetácia je odstránená, hodnoty sú kalibrované. Zväčšovanie a posúvanie zostávajú zachované, takže sa pozeráte na tú istú oblasť

### Prehliadnite jeden index v celej sade

1. Otvorte prvý spracovaný snímok a vyberte indexovú vrstvu
2. Opakovane stláčajte **→** — indexová vrstva vás sleduje zo snímku na snímok
3. Počas prehliadania sledujte histogram v bočnom paneli: snímok, na ktorom dochádza k náhlemu skoku v rozložení, si zaslúži bližšie preskúmanie

### Overenie kalibračných cieľov

1. Vyberte **RAW (Cieľ)** na cieľovom snímku
2. Uistite sa, že cieľ je jasne viditeľný a detekovaný
3. Prejdite na ďalší cieľový snímok — vrstva cieľov vás nasleduje

### Skontrolujte presnosť hodnôt odrazivosti

1. Vyberte **RAW (Odrazivosť)**

2. Prečítajte si stĺpec**%** v paneli Hodnoty kurzora — pre daný súbor je už správne škálovaný
3. Overte správnosť na základe známych materiálov v snímke: zdravá vegetácia má vysokú hodnotu v NIR

a nízku v červenej; kalibračný cieľ by mal vykazovať hodnotu blízku svojej uverejnenej odrazivosti

***

## Riešenie problémov

### Vrstva, ktorú som očakával, nie je v roletovom menu

**Možné príčiny**

* Snímka nebola nikdy spracovaná — existujú len základná vrstva a `RAW (Original)`
* V nastaveniach projektu nie je zaškrtnuté políčko pre export produktu
* Produkt sa na túto kameru nevzťahuje (radiancia a odrazivosť na RGB

master; akýkoľvek index na jednopásmovej monochromatickej kamere M3M)
* Kalibrácia odrazivosti nemala s čím pracovať — chýbalo pokrytie smerom nadol (`.daq`) a v snímke nebol cieľ, ktorý by prešiel kontrolou kvality (QA) — preto sa snímka vrátila k režimu „Vignette Corrected“ alebo „Sensor Response“

**Postup**

1. Skontrolujte protokol spustenia: Chloros

uvádza, kedy nebolo možné vygenerovať požadovaný exportný produkt a prečo
2. Skontrolujte prepínače exportu pre jednotlivé produkty v [Nastaveniach projektu](../project-settings/project-settings.md)
3. Overte, či zložka produktu existuje vo výstupnej štruktúre projektu
4. Spustite spracovanie znovu s povoleným produktom

### Zoznam vrstiev vyzerá neaktuálne

Program Chloros

počas prebiehajúceho behu opätovne skenuje zložky produktov projektu a opravuje chýbajúce registrácie vrstiev na základe toho, čo sa skutočne nachádza na disku, takže vrstva, ktorej export sa úspešne dokončil, sa v prieskume objaví sama od seba. Prechod preč od obrázku a späť vynúti nové vyhodnotenie.

### Hodnoty odrazivosti sa zdajú byť polovičné oproti tomu, aké by mali byť

Takmer s určitosťou delíte súbor LATTICE číslom 65535. Použite `Chloros:PixelScale` (32768) alebo si prečítajte stĺpec **%**, v ktorom je táto hodnota už aplikovaná.

### Indexová vrstva existuje, ale obrázok je prázdny

Index vyžaduje pásma, ktoré vaša vrstva nemá — napríklad index, ktorý číta tretí kanál, aplikovaný na jedno- alebo dvojkanálový súbor. Prejdite na viacpásmovú vrstvu (odrazivosť alebo debayered) alebo vyberte index, ktorý zodpovedá filtru kamery.

***

## Ďalšie kroky

* [**Otvorenie obrázka na celú obrazovku**](opening-an-image-full-screen.md) — zobrazenie kurzora, histogram a ovládanie GSD
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — interaktívna vizualizácia indexu a export
* [**Vzorce multispektrálnych indexov**](../project-settings/multispectral-index-formulas.md) — referenčné údaje k indexom
* [**Dokončenie spracovania**](../processing-images-gui/finishing-the-processing.md) — strom výstupných priečinkov, na ktoré tieto vrstvy odkazujú

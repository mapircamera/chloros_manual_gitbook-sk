# Vrstvy obrázkov

Pomocou **roztiahnutej ponuky vrstiev** v pravom hornom rohu prehliadača obrázkov môžete prepínať medzi všetkými verziami obrázku, ktorý si práve prezeráte – od zdrojového záberu cez jednotlivé spracované výstupy až po vypočítané indexové obrázky – bez toho, aby ste museli prehliadač opustiť.

## Čo sú vrstvy obrázkov?

„Vrstva“ v programe Chloros je jeden **súbor produktu**priradený k jednému zdrojovému obrázku. Importom získate zdrojové súbory; spracovaním sa pridá jedna vrstva pre každý produkt vytvorený v danom cykle. Exportované súbory si zachovávajú názov zdrojového súboru — produkt identifikuje**zložka** a názov vrstvy je označenie tejto zložky v systéme Chloros.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Zoznam vrstiev

### Vždy prítomné

| Vrstva | Čo to je |
| --- | --- |
| **JPG**(alebo**PNG**/**TIFF**) | Základný súbor, ktorý bol súčasťou zachytenia. Survey3 importuje súbor `.JPG` vedľa každého súboru `.RAW`; Záznamy LATTICE obsahujú náhľad na displeji vo formáte PNG alebo TIFF. Označené podľa toho, čo bolo skutočne importované |
| **RAW (originál)** | Zdrojový surový snímok, dekódovaný pre zobrazenie bez aplikovaných korekcií. Dostupné od okamihu importu — nevyžaduje žiadne spracovanie |

Záznam LATTICE, ktorého základným súborom **je** surový snímok, nemá samostatný základný záznam: `RAW (Original)` ho už pokrýva.

### Výstupy spracovania Survey3

| Vrstva | Zapísané do | Existuje, keď |
| --- | --- | --- |
| **RAW (Cieľ)** | — | Snímka bola identifikovaná ako obsahujúca kalibračný cieľ |
| **RAW (odrazivosť)** | `Reflectance_Calibrated_Images/` | Na tomto snímku prebehla úspešná kalibrácia odrazivosti |
| **Korekcia vinety**| `Vignette_Corrected_Images/` | Snímku nebolo možné kalibrovať podľa odrazivosti**a** bola zapnutá *korekcia vinety* |
| **Odpoveď snímača**| `Sensor_Response_Images/` | Snímku nebolo možné kalibrovať podľa odrazivosti**a** *korekcia vinety* bola vypnutá |
| **Vyváženie bielej** | `White_Balanced_Images/` | Bol vytvorený produkt s vyvážením bielej |

{% hint style="info" %}
**Korekcia vinetácie a odozva snímača sú alternatívy, nikdy nie obe naraz.** Na každý model fotoaparátu existuje presne jeden nekalibrovaný záložný produkt na jedno spustenie a prepínač *Korekcia vinetácie* určuje, ktorý to bude. Pozrite si [Nastavenia projektu](../project-settings/project-settings.md).
{% endhint %}

### Úrovne LATTICE

LATTICE zachytáva rozvetvenie do týchto úrovní v jednom spracovateľskom cykle. Ktoré z nich existujú, závisí od prepínačov exportu pre jednotlivé produkty v Nastaveniach projektu a od toho, čo platí pre daný fotoaparát.

| Vrstva | Zapísané do | Platí pre |
| --- | --- | --- |
| **RAW (bez bayesovho filtra)** | `Debayered_Images/` | RGB a multispektrálne |
| **RAW (náhľad)** | `Preview_Images/` | Multispektrálne (rozloženie falošných farieb) |
| **Vyváženie bielej** | `Preview_Images/` | Hlavné kamery RGB — náhľad RGB je zaregistrovaný pod týmto názvom, aby sa zhodoval s rovnomennou vrstvou Survey3 |
| **RAW (žiarivosť)** | `Radiance_Images/` | Iba multispektrálne |
| **RAW (odrazivosť)** | `Reflectance_Calibrated_Images/` | Iba multispektrálne, a to len vtedy, ak snímku pokrýva zodpovedajúci záznam smerujúci nadol `.daq` alebo cieľ v rámci snímky, ktorý prešiel kontrolou kvality |

Hlavné kamery RGB nemajú radiometriu na jednotlivé pásma, takže žiarivosť a odrazivosť sa u nich vynechávajú ako **neaplikovateľné** — protokol to uvádza, namiesto toho, aby došlo k tichému zlyhaniu.

### Vrstvy indexu, LUT a sandboxu

| Vzor vrstvy | Príklad | Odkiaľ pochádza |
| --- | --- | --- |
| **RAW (`<INDEX>` Index)** | `RAW (NDVI Index)` | Jedna na každý index nakonfigurovaný v nastaveniach projektu, vypočítaná počas spracovania |
| **`<INDEX>` LUT** | `NDVI LUT` | Verzia indexu s farebným mapovaním |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Jeden na každý beh exportu [Index/LUT Sandbox](index-lut-sandbox.md) |

Ak je rovnaký názov indexu nakonfigurovaný viac ako raz s rôznymi nastaveniami, druhý a ďalšie dostanú v názve číslo (`RAW (NDVI2 Index)`), aby bolo možné vrstvy od seba odlíšiť.

***

## Používanie výberu vrstiev

1. Otvorte obrázok na celú obrazovku kliknutím na miniatúru v mriežke
2. Kliknite na **rozbaľovacie menu vrstiev** v pravom hornom rohu prehliadača
3. Vyberte vrstvu — obrázok sa okamžite aktualizuje

V roletovom menu sú na prvých miestach v uvedenom poradí uvedené formáty **JPG, RAW (Original), RAW (Target), RAW (Reflectance)** a všetky ostatné sú uvedené za nimi v poradí, v akom boli produkty zaregistrované.

### Predvoľba vrstvy pri navigácii

Stlačením klávesov **←**/**→** prejdete na ďalší obrázok a systém sa pokúsi zachovať tú istú vrstvu:

1. **Najskôr presná zhoda** — ak má nasledujúci obrázok vrstvu s rovnakým názvom, zobrazí sa táto vrstva. Vďaka tomu zostanete na vrstve `RAW (NDVI Index)` aj pri prechádzaní celej sady
2. **Potom zhoda podľa typu** — indexová vrstva hľadá akúkoľvek indexovú vrstvu, LUT akúkoľvek LUT, odrazivosť akúkoľvek odrazivosť, cieľovú vrstvu akúkoľvek cieľovú vrstvu, originál akýkoľvek originál, základnú akúkoľvek základnú
3. **Potom, iba pre exportné vrstvy** — názov sa zachová aj v prípade, že zoznam vrstiev ešte nedobehol, pretože súbor už existuje na disku. To vám umožňuje prezerať produkty, zatiaľ čo ich spustenie stále zapisuje
4. **V ostatných prípadoch** — prvá dostupná vrstva, ktorou je zvyčajne základný obrázok

Súbory sidecar `.daq` a `.csv` v projekte sa pri navigácii pomocou šípok preskakujú, takže pri prechádzaní obrázkov sa nikdy nedostanete k záznamu svetelného senzora.

Zväčšovanie a posúvanie sa prenáša aj medzi snímkami, čo uľahčuje porovnanie tej istej polohy poľa pred a po.

***

## Porozumenie hodnotám pixelov podľa vrstvy

[Panel Hodnoty kurzora](opening-an-image-full-screen.md#cursor-values) zobrazuje skutočnú hodnotu na kanál pod kurzorom v jednotke, v ktorej je daná vrstva uložená. Jeho stĺpce sa menia v závislosti od vrstvy:

| Vrstva | Zobrazená jednotka | Poznámky |
| --- | --- | --- |
| Základná (JPG / náhľad PNG / TIFF) | DN, 0–255 | Zobrazené hodnoty, s korekciou gama v programe RGB. Iba vizuálna kontrola |
| RAW (originál) | DN | Surové digitálne hodnoty snímača. Os histogramu udáva hĺbku: 255 (8-bitová), 4095 (12-bitová) alebo 65535 (16-bitová) |
| RAW (bez bayeresovania) | DN | Lineárne, bez rozťahovania zobrazenia |
| RAW (Náhľad) / Vyváženie bielej | DN | Zobrazený výsledok — rozťahaný alebo s korekciou gama. Nie je určené na meranie |
| RAW (Žiarivosť) | **W/m²/sr/nm** | Fyzikálna žiarivosť typu Float32. Bez stĺpca DN |
| RAW (odrazivosť) | DN **a %** | Percento vypočítané podľa vlastnej stupnice daného súboru — pozri nižšie |
| Exporty indexu / LUT / sandbox | Hodnota indexu alebo zložky RGB | Jednokanálový indexový súbor uvádza hodnotu indexu; súbor LUT s farebným mapovaním uvádza komponenty Red/Green/Blue |

### Odrazivosť: mierka je pre každý súbor individuálna

{% hint style="warning" %}
**„Vydelenie číslom 65 535“ je správne iba pre Survey3.** Odrazivosť LATTICE je uložená v inom meradle a zmiešanie týchto dvoch deliteľov je najbežnejší spôsob, ako získať hodnoty odrazivosti, ktoré sú presne polovičné oproti tomu, aké by mali byť.
{% endhint %}

| Zdroj | DN, ktorý zodpovedá odrazivosti 1,0 | Identifikované podľa |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | Značka XMP `Chloros:PixelScale=32768` vložená do každého exportu odrazivosti LATTICE. Dvojnásobná rezerva znamená, že hodnota ρ nad 1,0 je reprezentovateľná a nie je orezaná |
| **Survey3**|**65535** | Ak chýba značka mierky XMP Chloros — kalibrácia Survey3 zapíše ρ × dtype-max a orezáva pri hodnote 1,0 |

Pre GIS a skriptovanie: prečítajte hodnotu `Chloros:PixelScale` zo súboru a vydelte ňou. Ak táto značka chýba, súbor je v mierke Survey3 (65535). Prehliadač, index/LUT sandbox a export indexu všetky určujú mierku týmto rovnakým spôsobom, takže číslo, ktoré vidíte pri kurzore, je číslo, ktoré použila matematika indexu.

Ukladanie špecifické pre formát nad rámec tohto meradla:

* **TIFF (32-bitové, percentá)** ukladá hodnotu DN / 65535 ako číslo s plávajúcou desatinnou čiarkou
* **PNG (8-bitové)**a**JPG (8-bitové)** ukladajú DN × 255 / 65535
* **8-bitový export TIFF zo záznamu s 8-bitovým zdrojom** je orezaný na rozsah 0–255 namiesto preškálovania a zámerne neobsahuje žiadnu značku mierky. Panel zobrazuje hodnotu DN len pre tieto súbory, bez stĺpca s percentami

### Rozsahy indexových hodnôt

| Rodina indexov | Typický rozsah | Hodnota |
| --- | --- | --- |
| Normalizovaný rozdiel (NDVI, GNDVI, NDRE, ENDVI…) | −1 až +1 | Zdravá vegetácia zvyčajne 0,4–0,9; holá pôda blízko 0; voda záporná |
| Upravené podľa pôdy (SAVI, OSAVI, MSAVI2…) | približne −1 až +1,5 | Hodnota podobná NDVI s potlačeným pôdnym pozadím |
| Pomer (GRVI, GCI, MSR, CIRE…) | neobmedzený smerom nahor | Pomery rastú bez obmedzenia, keď sa pásmo v menovateli blíži k nule |
| EVI / LAI | 0 až ~1, 0 až ~3,5 | Oblaky a iné nasýtené pixely posúvajú obe hodnoty mimo rozsah — najskôr ich zamaskujte |

Presné vzorce pre každé prednastavenie nájdete v [Vzorcov multispektrálnych indexov](../project-settings/multispectral-index-formulas.md).

***

## Bežné pracovné postupy

### Porovnanie pred a po

1. Vyberte **RAW (Originál)** a všimnite si vinetáciu a nekalibrované hodnoty
2. Prepnite na **RAW (Reflexia)**

3. Porovnajte — vinetácia odstránená, hodnoty kalibrované. Zväčšovanie a posúvanie zostávajú zachované, takže sa pozeráte na rovnakú oblasť

### Prehliadnite jeden index v celej sade

1. Otvorte prvý spracovaný obrázok a vyberte indexovú vrstvu
2. Opakovane stlačte **→** — indexová vrstva vás sleduje z obrázku na obrázok
3. Počas prehliadania sledujte histogram v bočnom paneli: snímka, na ktorej dochádza k skokom v rozložení, si zaslúži bližšiu pozornosť

### Overte kalibračné ciele

1. Na cieľovom snímku vyberte **RAW (Target)**

2. Uistite sa, že cieľ je jasne viditeľný a detekovaný
3. Prejdite na ďalší cieľový snímok — vrstva cieľov vás nasleduje

### Skontrolujte presnosť hodnôt odrazivosti

1. Vyberte **RAW (Reflectance)**

2. Prečítajte si stĺpec**%** v paneli Hodnoty kurzora — pre daný súbor je už správne škálovaný
3. Overte správnosť na základe známych materiálov v snímke: zdravá vegetácia má vysokú hodnotu NIR a nízku hodnotu červenej; kalibračný cieľ by mal vykazovať hodnotu blízku svojej uverejnenej odrazivosti

***

## Riešenie problémov

### Vrstva, ktorú som očakával, nie je v roletovom menu

**Možné príčiny**

* Snímka nebola nikdy spracovaná — existujú len základná vrstva a vrstva `RAW (Original)`
* V nastaveniach projektu nie je zaškrtnuté políčko pre export produktu
* Produkt sa na danú kameru nevzťahuje (radiancia a odrazivosť na hlavnej kamere typu RGB; akýkoľvek index na jednopásmovej monochromatickej kamere M3M)
* Kalibrácia odrazivosti nemala s čím pracovať — chýbalo pokrytie smerom nadol pre `.daq` a nebol k dispozícii cieľ v snímke, ktorý by prešiel kontrolou kvality — preto sa snímka vrátila k režimu „Vignette Corrected“ alebo „Sensor Response“

**Postup**

1. Skontrolujte protokol behu: Chloros uvádza, kedy nebolo možné vygenerovať požadovaný exportný produkt a prečo
2. Skontrolujte prepínače exportu pre jednotlivé produkty v [Nastaveniach projektu](../project-settings/project-settings.md)
3. Overte, či zložka produktu existuje v stromovej štruktúre výstupov projektu
4. Spustite spracovanie znova s povoleným produktom

### Zoznam vrstiev vyzerá, že nie je aktuálny

Chloros počas prebiehajúceho behu opätovne prehľadáva zložky produktov projektu a opravuje chýbajúce registrácie vrstiev na základe toho, čo sa skutočne nachádza na disku, takže vrstva, ktorej export sa úspešne dokončil, sa pri ďalšom prehľade zobrazí sama od seba. Prechod na inú obrazovku a späť vynúti nové vyhodnotenie.

### Hodnoty odrazivosti vyzerajú ako polovica toho, čo by mali byť

Takmer určite delíte súbor LATTICE číslom 65535. Použite `Chloros:PixelScale` (32768) alebo si prečítajte stĺpec **%**, v ktorom je táto hodnota už aplikovaná.

### Indexová vrstva existuje, ale obrázok je prázdny

Index vyžaduje pásma, ktoré vaša vrstva nemá — napríklad index, ktorý číta tretí kanál, aplikovaný na jedno- alebo dvojkanálový súbor. Prepnite na viacpásmovú vrstvu (odrazivosť alebo debayered) alebo vyberte index, ktorý zodpovedá filtru kamery.

***

## Ďalšie kroky

* [**Otvorenie obrázka na celú obrazovku**](opening-an-image-full-screen.md) — zobrazenie kurzora, histogram a ovládanie GSD
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — interaktívna vizualizácia a export indexu
* [**Vzorce multispektrálnych indexov**](../project-settings/multispectral-index-formulas.md) — referenčné údaje o indexoch
* [**Dokončenie spracovania**](../processing-images-gui/finishing-the-processing.md) — strom výstupných zložiek, na ktoré tieto vrstvy odkazujú

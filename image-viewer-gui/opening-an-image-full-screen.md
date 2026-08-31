# Otvorenie obrázku na celú obrazovku

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Obrázok otvorený na celej obrazovke s voličom vrstiev v pravom hornom rohu</p></figcaption></figure>

Prehliadač obrázkov Chloros je rozhranie na celej obrazovke určené na prezeranie, kontrolu a meranie vašich obrázkov. Tu môžete čítať **skutočné hodnoty pixelov** — DN na kanál, percentuálnu odrazivosť alebo žiarivosť v W/m²/sr/nm — namiesto roztiahnutého náhľadu, ktorý zobrazuje obrazovka.

## Spustenie prehliadača obrázkov

### Z prehliadača súborov

1. Otvorte kartu **Prehliadač súborov** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Kliknite na ľubovoľnú **miniatúru** v [mriežke obrázkov](image-grid.md)
3. Obrázok sa otvorí na celej obrazovke na karte **Prehliadač obrázkov**

Obrázok sa otvorí v tom produkte, ktorý bol zobrazený v mriežke. Ak je mriežka nastavená na `RAW (Reflectance)`, otvorí sa práve táto vrstva.

### Otvorenie bočného panela prehliadača obrázkov

Kliknutím na ikonu **Prehliadač obrázkov** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> v ľavom bočnom paneli rozbalíte panel analýzy. Obsahuje zhora nadol:

* názov snímky a model kamery
* tlačidlo **Exportovať/Uložiť snímky** (len ak je aktívny index alebo LUT)
* začiarkavacie políčka **Index**a**LUT** a panel konfigurácie indexu — pozri [Index/LUT Sandbox](index-lut-sandbox.md)
* panel **Hodnoty kurzora**: odčítanie údajov za každý kanál, histogram vrstvy a ovládací prvok GSD***

## Navigácia a priblíženie

### Prehliadanie obrázkov

* **Ďalší obrázok**: tlačidlo → alebo klávesa**→** (šípka doprava)
* **Predchádzajúci obrázok**: tlačidlo ← alebo klávesa**←** (šípka doľava)
* **Preskočiť na konkrétny obrázok**: vráťte sa do mriežky a kliknite na jeho miniatúru

Zväčšenie a posun zostávajú zachované aj pri prechádzaní medzi obrázkami, takže môžete prechádzať sadou obrázkov a zároveň zostať na tej istej časti snímky.

### Zväčšenie

Zväčšenie sa ovláda pomocou **kolieska myši** v 15 % krokoch a je ukotvené na kurzore — bod pod ukazovateľom zostáva pod ukazovateľom. Rozsah je ohraničený veľkosťou obrázku a okna: nemôžete zmenšiť obraz viac, ako je veľkosť okna, a horná hranica je daná natívnym rozlíšením obrázku.

V prehliadači na celú obrazovku nie sú žiadne špeciálne klávesy na priblíženie. (V mriežke klávesová skratka **Ctrl + `+` / `−`** mení veľkosť náhľadov – ide o iný ovládací prvok.)

### Posúvanie pri zväčšení

Kliknite a podržte ľavé tlačidlo myši nad obrázkom a ťahajte. Posúvanie je obmedzené, takže obrázok nemožno vytiahnuť mimo obrazovku.

### Kontrola jednotlivých pixelov pri vysokom zväčšení

Akonáhle efektívne zväčšenie prekročí hodnotu **60×**, Chloros nakreslí zvýrazňujúci rámček okolo jednotlivého zobrazeného pixelu pod kurzorom a vedľa neho sa zobrazí plávajúca hodnota.

„Efektívne“ zväčšenie zohľadňuje veľkosť bloku GSD: pri veľkosti bloku 8 sa zvýraznenie objaví pri zväčšení 7,5× namiesto 60×, pretože jeden zobrazený pixel už zodpovedá 8 × 8 zdrojovým pixelom. Ak zväčšenie zmenšíte pod túto hranicu, zvýraznenie zmizne.

### Klávesové skratky

| Klávesa                             | Kde       | Akcia                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Celá obrazovka | Ďalší obrázok                          |
| **←**                           | Celá obrazovka | Predchádzajúci obrázok                      |
| **Ctrl + R**                    | Celá obrazovka | Obnoviť index/LUT sandbox         |
| **Ctrl + `+`**/**Ctrl + `=`** | Mriežka        | Väčšie náhľady (4 px na stlačenie)  |
| **Ctrl + `−`**                  | Mriežka        | Menšie náhľady (4 px za stlačenie) |***

## Hodnoty kurzora

Presuňte kurzor nad obrázok a panel **Hodnoty kurzora** zobrazí hodnotu každého kanála pod ním.

{% hint style="success" %}
**Toto sú skutočné čísla súboru.** Plocha na obrazovke je 8-bitový roztiahnutý náhľad a nemôže ich poskytnúť, preto Chloros na účely zobrazenia odoberá vzorky zo skutočného súboru produktu. Preto 12-bitový surový snímok uvádza hodnoty nad 255 a vrstva radiancie typu float32 uvádza fyzikálne jednotky.
{% endhint %}

### Význam stĺpcov

Panel sa prispôsobuje vrstve, ktorú práve prezeráte:

| Prezeraná vrstva              | Zobrazené stĺpce    | Poznámky                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Odrazivosť                        | **DN**a**%** | Percento sa počíta podľa vlastného meradla daného súboru — pozri nižšie                                      |
| Žiarivosť                           | **W/m²/sr/nm**   | Fyzikálne hodnoty typu float; stĺpec DN chýba, pretože DN tu nemá zmysel                           |
| Surové / po odstránení bayeringu / náhľad / JPG    | **DN**           | Celé digitálne čísla                                                                         |
| Export 32-bitovej percentuálnej odrazivosti | iba **%**       | Uložená hodnota s plávajúcou desatinnou čiarkou nie je DN, takže zaokrúhlenie na celé číslo by viedlo k nezmyselnému výstupu typu `0` alebo `1` |

Každý riadok je označený názvom kanála filtra vášho fotoaparátu — `Red / Green / NIR` pre RGN, `Orange / Cyan / NIR` pre OCN, `NIR / Green / Blue` pre NGB, `Red / Green / Blue` pre RGB a názov jediného pásma pre kamery RE, NIR a mono M3M. Každá etiketa obsahuje farebnú bodku, ktorá zodpovedá kruhom kanálov používaným v editore indexových vzorcov.

Uložené obrázky **indexu a LUT** predstavujú špeciálny prípad: obsahujú komponenty farebnej mapy namiesto spektrálnych pásiem, preto sú ich riadky označené ako `Red / Green / Blue` (alebo `Index` v prípade jedno-kanálového indexového súboru) namiesto názvov filtrov kamery.

Keď je index aktívny v testovacom prostredí, pod kanálmi sa zobrazí dodatočný riadok s **hodnotou indexu** v mieste kurzora, spolu s názvom indexu a bielou bodkou, ktorá zodpovedá jeho značke na histograme.

### Percento odrazivosti používa vlastnú stupnicu každého súboru

{% hint style="warning" %}
**Nepredpokladajte, že 65535 = 100 %.** Chloros ukladá odrazivosť v rôznych stupniciach v závislosti od toho, ktorá kamera ju vytvorila, a prehliadač určí správnu stupnicu pre každý súbor.
{% endhint %}

| Zdroj                  | DN, ktorý zodpovedá odrazivosti 1,0 | Ako sa identifikuje                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | XMP tag `Chloros:PixelScale=32768` zapísaný do každého exportu odrazivosti v formáte LATTICE. Dvojnásobná rezerva umožňuje, aby súbor obsahoval hodnotu ρ vyššiu ako 1,0 bez orezania |
| **Survey3**|**65535**                      | Chýba XMP značka mierky Chloros — kalibrácia Survey3 zapisuje ρ × dtype-max a orezáva pri hodnote 1,0                                                               |

Prehliadač, sandbox indexu/LUT a export indexu všetky riešia mierku prostredníctvom tej istej jedinej implementácie, takže hodnota, ktorú prečítate na kurzore, je tá istá hodnota, ktorú použila matematika indexu.

Dva dôsledky, o ktorých stojí za to vedieť:

* **32-bitové percentuálne**TIFF ukladá hodnotu DN/65535 ako číslo typu float a**8-bitový** export PNG/JPG ukladá hodnotu DN × 255/65535 — prehliadač obidve hodnoty pred zobrazením percenta opäť prevádza späť.
* Jeden prípad nie je možné obnoviť: **8-bitový export TIFF zo 8-bitového zdrojového záznamu** je orezaný na rozsah 0–255 namiesto preškálovania a zámerne neobsahuje žiadnu značku mierky. Pri týchto súboroch panel tlačí iba hodnotu DN, bez stĺpca s percentami. Toto je úprimná odpoveď, nie chyba.***

## Histogram vrstvy

Pod riadkami kurzora sa nachádza živý histogram vrstvy, ktorú práve prezeráte, v **256 intervaloch**. Štandardne sa kreslí jedna kombinovaná krivka, vážená `(R + 2G + B) / 4` — ten istý merací priestor, aký používajú histogramy kamery LATTICE. Zapnutím**RGB** sa nahradí krivkami pre jednotlivé kanály v farbách kanálov, ktoré sú aditívne zmiešané, aby prekrývania zostali čitateľné. Mono vrstvy vždy vykresľujú jedinú krivku.

Vodorovná os je v jednotkách danej vrstvy:

| Vrstva       | Jednotka osi  | Maximálna hodnota osi                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Odrazivosť | percent    | 125 % — rezerva produktu umožňuje ρ nad 1,0           |
| Žiarivosť    | W/m²/sr/nm | Vlastný vrchol snímky, zaokrúhlený nahor na dve významné číslice |
| 8-bitové údaje | DN         | 255                                                        |
| 12-bitové údaje | DN         | 4095                                                       |
| 16-bitové údaje | DN         | 65535                                                      |

Keď je os v jednotkách DN a dosiahne jednu z týchto troch horných hraníc, Chloros zároveň zistí bitovú hĺbku zobrazeného obrazu.

Nad histogramom sa nachádzajú tri tlačidlá:

| Tlačidlo     | Predvolené nastavenie | Účinok                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **KURZOR** | Zapnuté      | Nakreslí na histograme značenia v presných hodnotách uvedených v riadkoch vyššie, takže môžete vidieť, kde sa pixel pod kurzorom nachádza v rozložení snímky. V režime RGB je na každý kanál jedna značka vo vlastnej farbe; inak je to jediná biela značka pri kombinovanej hodnote |
| **INDEX**| Zapnuté      | Zobrazuje sa len vtedy, keď je aktívny index. Prepne histogram zo zdrojových pásiem na**rozdelenie indexových hodnôt**, pričom dve prahové hodnoty orezania sú znázornené ako oranžové prerušované čiary a indexová hodnota kurzora ako biela čiara                                                          |
| **RGB**| Vypnuté     | Prepne z kombinovanej krivky na krivky pre jednotlivé kanály. Na monochromatickom senzore sa na tomto tlačidle zobrazuje nápis**MONO** a je deaktivované — je tu len jeden kanál, ktorý sa má zobraziť                                                                                                                                  |

Histogram sa počíta na základe **viditeľných blokov**, nie na základe zdrojových pixelov za nimi: zmenou veľkosti bloku GSD sa rozloženie prepočíta, takže histogram, značka kurzora a zobrazený obrázok sa vždy zhodujú.***

## Veľkosť bloku GSD

V spodnej časti panela sa nachádza ovládací prvok **GSD (px)**: číselné pole, posuvník v rozsahu od**1 do 256**a tlačidlo**RESET**.

Zhrubňuje _zobrazený_ obrázok tým, že zoskupuje blok zdrojových pixelov s rozmermi N × N do jedného zobrazeného pixela. `1` je natívne rozlíšenie.

* Ovplyvňuje **zobrazenie na celej obrazovke, miniatúry mriežky, zobrazenie kurzora a oba histogramy** — všetko, čo zobrazuje obrázok, sa zhoduje na rovnakom základnom rozlíšení.
* Ide **len o zobrazenie**. Spracovanie a export zostávajú nedotknuté. Jediná výnimka je zámerná: export cez [Index/LUT Sandbox](index-lut-sandbox.md) uloží to, na čo sa práve pozeráte, takže zachová aktuálnu veľkosť bloku, a panel exportu vás upozorní, ak je veľkosť bloku väčšia ako 1.
* Hodnota sa ukladá **pre každý projekt** ako `viewer_display.gsd_bin` v `project.json`, takže pretrváva aj po zatvorení a opätovnom otvorení.
* Hodnota zobrazená na kurzore udáva informácie o bloku, nie o zdrojovom pixeli, vždy keď je veľkosť bloku väčšia ako 1 — zobrazená hodnota je priemer bloku pod vaším kurzorom.

{% hint style="info" %}
**Prečo „veľkosť bloku“ a nie centimetre na pixel?** Hodnota cm/px vyžaduje výšku nad zemou. EXIF údaj jedného snímku obsahuje nadmorskú výšku GPS nad priemernou hladinou mora, nie nad terénom, na ktorý bol fotoaparát nasmerovaný, takže Chloros nevytlačí vzdialenosť od zeme, ktorú nemôže spoľahlivo odvodiť. Veľkosť bloku v zdrojových pixloch je rovnakým náhradným riešením, aké používajú nástroje pre spracovanie snímok MAPIR, keď nie je známa vzdialenosť vzorkovania od povrchu.
{% endhint %}

***

## Typy obrázkov, ktoré si môžete prezerať

Rozbaľovacie menu vrstiev v pravom hornom rohu prehliadača obsahuje zoznam všetkých verzií aktuálneho obrázku. Ktoré položky sa zobrazia, závisí od kamery a od toho, čo bolo spracované — pozrite si [Obrazové vrstvy](image-layers.md), kde nájdete úplný zoznam a informácie o fungovaní roletového menu.

### Survey3

* **JPG** — vlastný náhľadový súbor kamery
* **RAW (originál)** — zdrojový súbor `.RAW`, zbavený bayeresovania pre zobrazenie, bez korekcií
* **RAW (Cieľ)** — snímka identifikovaná ako obsahujúca kalibračný cieľ
* **RAW (Odrazivosť)** — kalibrovaný produkt odrazivosti (65535 = ρ 1,0)
* **Korigovaná vineta**/**Odpoveď snímača** — nekalibrovaný náhradný produkt
* **Vyváženie bielej** — produkt s vyváženou bielou
* **RAW (`<INDEX>` Index)**a**`<INDEX>` LUT** — vypočítané indexové snímky

### LATTICE

Zábery LATTICE používajú rovnaké roletové menu s názvami úrovní spracovateľského reťazca:

| Vrstva                 | Čo obsahuje                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (pôvodný)**    | Zdrojový surový snímok tak, ako bol zachytený                                     |
| **RAW (bez bayeresovania)**   | Lineárny obraz bez bayeresovania                                           |
| **RAW (Náhľad)**     | Náhľad na displeji — rozloženie falošných farieb pre multispektrálne kamery |
| **Vyváženie bielej**    | Náhľad na displeji pre hlavné kamery RGB (vyváženie bielej + gama)   |
| **RAW (žiarivosť)**    | Spektrálna žiarivosť typu Float32 v W/m²/sr/nm                              |
| **RAW (odrazivosť)** | Odrazivosť typu uint16, 32768 = ρ 1,0                                    |

Žiarivosť a odrazivosť sú k dispozícii iba v multispektrálnom režime: hlavná kamera typu RGB nemá radiometriu pre jednotlivé pásma, preto sa pre ňu tieto vrstvy nevytvárajú.

***

## Použitie indexov a LUT

Aplikujte multispektrálne indexy a farebné vyhľadávacie tabuľky (LUT) z bočného panela:

1. Otvorte bočný panel **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Zaškrtnite **Index**

3. Vyberte filter vašej kamery a vzorec indexu, potom pretiahnite kruhy kanálov na príslušné polia vo vzorci
4. Pridajte LUT a vyberte prechod, prahové hodnoty a režim orezania
5. Prečítajte si hodnoty v mieste kurzora a výsledok uložte pomocou **Export/Save Image(s)**Úplný návod nájdete v [Index/LUT Sandbox](index-lut-sandbox.md).***

## Riešenie problémov

### Obrázok sa neotvorí

**Možné príčiny**: súbor bol po importe presunutý alebo vymazaný; produkt nebol nikdy zapísaný; nedostatočná pamäť pre veľmi veľký obrázok.**Čo robiť**:

1. Skontrolujte, či súbor vrstvy stále existuje vo výstupnej štruktúre projektu
2. Otvorte súbor v externom prehliadači, aby ste sa uistili, že je nepoškodený
3. Zatvorte ostatné aplikácie, aby ste uvoľnili pamäť

### Obraz je čierny, biely alebo má divoké farby

**Možné príčiny**: funkcia roztiahnutia obrazu nemá s čím pracovať (takmer konštantný snímok); vrstva typu float32 s nezvyčajnými hodnotami; index, ktorý nevytvoril žiadne platné údaje.**Čo robiť**:

1. Prečítajte hodnoty kurzora — ak je každý kanál na nule alebo blízko nuly, problém je v údajoch, nie v zobrazení
2. Skontrolujte histogram: jediný výkyv na jednom konci naznačuje, že snímka je orezaná alebo prázdna
3. Skontrolujte protokol spracovania pre beh, ktorý vytvoril vrstvu

### Hodnoty vyzerajú nesprávne

**Možné príčiny**: nachádzate sa na inej vrstve, ako si myslíte; porovnávate percento s surovou hodnotou DN; porovnávate súbor LATTICE so súborom Survey3 s použitím rovnakého deliteľa.**Čo robiť**:

1. Overte si vybranú vrstvu v roletovom menu – jednotky v paneli sa riadia vrstvou
2. V prípade odrazivosti použite stĺpec **%** namiesto toho, aby ste hodnotu DN delili sami; ak musíte deliť, použite hodnotu `Chloros:PixelScale` daného súboru (32768 pre LATTICE, ak chýba, znamená to 65535 pre Survey3)
3. Nastavte veľkosť bloku GSD späť na 1 — pri hodnote nad 1 čítate priemernú hodnotu bloku, nie pixel
4. Skontrolujte, či sa pre daný snímok skutočne vykonala kalibrácia odrazivosti; nekalibrovaný náhradný produkt (Sensor Response / Vignette Corrected) nie je odrazivosťou

***

## Ďalšie kroky

* [**Obrázkové vrstvy**](image-layers.md) — názvy všetkých vrstiev (ak existujú) a význam ich hodnôt
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — vytvárajte, ladte a exportujte vizualizácie indexov
* [**Značky na mape**](map-markers.md) — tá istá sada obrázkov na mape
* [**Vzorce multispektrálnych indexov**](../project-settings/multispectral-index-formulas.md) — referenčné informácie o indexoch

Postup spracovania nájdete v časti [Spracovanie snímok (GUI)](../processing-images-gui/adding-files-to-a-project.md).

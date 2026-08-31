# Nastavenia projektu

Bočný panel „Nastavenia projektu“ (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

) v programeChloros

vám umožňuje konfigurovať všetky aspekty spracovania obrazu, detekcie kalibračných cieľov, výpočtov multispektrálnych indexov a možností exportu pre váš projekt. Tieto nastavenia sa ukladajú spolu s vaším projektom a je možné ich uložiť ako šablóny na opätovné použitie vo viacerých projektoch.

## Otvorenie nastavení projektu

Postup otvorenia nastavení projektu:

1. Otvorte projekt na stránkeChloros


2. Kliknite na kartu **Nastavenia projektu**<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

v ľavom bočnom paneli
3. V paneli nastavení sa zobrazia všetky dostupné konfiguračné možnosti zoradené podľa kategórií



<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Nastavenia, ktoré závisia od iných nastavení, sú sivé.** Ak nadradené prepnutie znemožňuje určité nastavenie (napríklad zrušenie zaškrtnutia položky *Kalibrácia odrazivosti / vyváženie bielej* znemožňuje *Export odrazivosti*), závislé ovládacie prvky sú deaktivované a v ich popise je uvedené, ktoré prepnutie je potrebné zmeniť.
{% endhint %}

***

## Zobrazenie

### Rozlíšenie miniatúr obrázkov

* **Typ**: Výber z roletového menu
* **Možnosti**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Predvolené**: Predvolené (512 px)
* **Popis**: Rozlíšenie (najdlhšia hrana, v pixeloch), pri ktorom sa vykresľujú miniatúry v mriežke obrázkov. Vyššie hodnoty vyzerajú pri zväčšení ostrejšie, ale načítajú sa pomalšie a spotrebúvajú viac pamäte. Plné rozlíšenie zodpovedá pôvodnej veľkosti obrázku.
* **Poznámka**: Iba na zobrazenie — toto nikdy neovplyvňuje spracovanie ani exportované súbory.***

## Detekcia cieľov

Tieto nastavenia určujú, ako modul „Chloros

“ detekuje a spracováva kalibračné ciele vo vašich obrázkoch. Obe sú aktívne len vtedy, keď je zapnutá funkcia **Kalibrácia odrazivosti / vyváženie bielej** (inak sú sivé, pretože detekcia cieľov sa úplne preskočí).

### Minimálna plocha kalibračnej vzorky (px)

* **Typ**: Číslo
* **Rozsah**: 0 až 10 000 pixelov
* **Predvolené nastavenie**: 25 pixelov
* **Popis**: Nastavuje minimálnu plochu (v pixeloch), ktorá je potrebná na to, aby bola detegovaná oblasť považovaná za platnú vzorku kalibračného cieľa. Menšie hodnoty detegujú menšie ciele, ale môžu zvýšiť počet falošných pozitív. Väčšie hodnoty vyžadujú na detekciu väčšie a zreteľnejšie oblasti cieľov.
* **Kedy upraviť**:
  * Zvýšte hodnotu, ak sa vyskytujú falošné detekcie na malých artefaktoch v obraze
  * Znížte hodnotu, ak sa vaše kalibračné ciele na obrázkoch javia ako malé a nie sú detekované

### Minimálne zhlukovanie cieľov (0–100)

* **Typ**: Číslo
* **Rozsah**: 0 až 100
* **Predvolená hodnota**: 60
* **Popis**: Ovláda prah zoskupovania podobných farebných oblastí pri detekcii kalibračných cieľov. Vyššie hodnoty vyžadujú zoskupenie väčšieho počtu podobných farieb, čo vedie k konzervatívnejšej detekcii cieľov. Nižšie hodnoty umožňujú väčšiu farebnú variabilitu v rámci jednej skupiny cieľov.
* **Kedy nastaviť**:
  * Zvýšte hodnotu, ak sa kalibračné ciele rozdeľujú na viacero detekcií
  * Znížte hodnotu, ak sa kalibračné ciele s farebnými variáciami nedetekujú v plnom rozsahu

***

## Spracovanie

Tieto nastavenia určujú, ako programChloros

spracováva a kalibruje vaše snímky.

### Korekcia vinetácie

* **Typ**: Zaškrtávacie políčko
* **Predvolené nastavenie**: Zapnuté (zaškrtnuté)
* **Popis**: Uplatňuje korekciu vinetácie na kompenzáciu stmavnutia objektívu na okrajoch snímok. Vinetácia je bežný optický jav, pri ktorom rohy a okraje snímky vyzerajú tmavšie ako stred v dôsledku vlastností objektívu.
* **Vedľajší účinok**: Týmto prepínačom sa tiež vyberá, ktorý *nekalibrovaný náhradný produkt* sa zapíše pri spustení (pozri nižšie).

### Kalibrácia odrazivosti / vyváženie bielej

* **Typ**: Zaškrtávacie políčko
* **Predvolené nastavenie**: Zapnuté (zaškrtnuté)
* **Popis**: Aktivuje kalibráciu odrazivosti — na základe detegovaných kalibračných cieľov v zábere a/alebo údajov o dopadajúcom svetle zo svetelného senzora DAQ, v závislosti od kamery a dostupných možností. Tým sa normalizujú hodnoty odrazivosti v celom vašom súbore údajov a zabezpečia sa konzistentné merania bez ohľadu na svetelné podmienky.
* **Keď je deaktivované**: Detekcia cieľov sa úplne preskočí a**žiadna kamera nemôže vytvoriť žiadny produkt odrazivosti** — či už ide o systém riadený cieľmi typu „Survey3

“ alebo systém riadený zariadením DAQ typu LATTICE. Súvisiace nastavenia (*Exportovať odrazivosť*, *Minimálny interval rekalibrácie* a prahové hodnoty detekcie cieľov) sú sivé.

### Nekalibrované náhradné výstupy: Exportovať odozvu senzora / Exportovať s korekciou vinietovania

* **Typ**: Dve začiarkavacie políčka
* **Predvolené nastavenia**: Obe sú začiarknuté (zaškrtnuté)
* **Popis**: Ak nie je možné kalibrovať odrazivosť snímky (nebol nájdený žiadny kalibračný cieľ alebo je kalibrácia odrazivosti vypnutá), uloží sa namiesto toho ako *nekalibrovaný náhradný produkt*. **Na jedno spustenie existuje presne jeden z dvoch náhradných produktov pre každý model kamery**, vybraný prepínačom *Korekcia vinetácie*:
  * Korekcia vinetácie **zapnutá**→ `Vignette_Corrected_Images/` (riadené nastavením**Exportovať s korekciou vinetácie**)
  * Korekcia vinetácie **vypnutá**→ `Sensor_Response_Images/` (riadi sa nastavením**Exportovať odozvu snímača**)
* Náhradný produkt, ktorý nie je aktívny, je sivý. Zrušením začiarknutia aktívneho produktu sa úplne zabráni zapísaniu daného súboru.

### Exportné produkty LATTICE

V prípade projektov obsahujúcich snímky LATTICE sa každý importovaný snímok LATTICE rozdelí do všetkých povolených **a príslušných**produktov v rámci jedného cyklu spracovania. Rozdelenie riadia štyri zaškrtávacie políčka (všetky sú predvolene**zapnuté**):

| Nastavenie | Výstupná zložka | Čo sa exportuje |
| --- | --- | --- |
| **Exportovať po odstránení bayera** | `Debayered_Images/` | Lineárny obraz po odstránení bayera. Platí pre kamery typu „RGB

“ a multispektrálne kamery. |
| **Exportovať náhľad** | `Preview_Images/` | Náhľad na displeji.RGB

= vyváženie bielej (DAQ-svetelný zdroj, ak je k dispozícii, inak šedý svet) + gama; multispektrálny = rozloženie falošných farieb. |
| **Export žiarivosti** | `Radiance_Images/` | Spektrálna žiarivosť typu Float32 v W/m²/sr/nm. Iba multispektrálne (M3C/M3M) — neplatí pre mastery typu „RGB

“. Vždy sa zapisuje ako 32-bitovéTIFF

bez ohľadu na nastavenie *Formát kalibrovaného obrazu*. |
| **Export odrazivosti**| `Reflectance_Calibrated_Images/` | Odrazivosť typu Uint16, škálovaná tak, že**32768 = odrazivosť 1,0** (označená ako XMP `Chloros:PixelScale`). Iba multispektrálne, zapísané, ak snímku pokrýva zodpovedajúci záznam `.daq` smerujúci nadol (alebo cieľ v rámci snímky, ktorý prešiel kontrolou kvality). |

* Hlavné kamery typu „RGB

“ vysielajú obraz bez bayeresového filtra + náhľad; žiarivosť/odrazivosť sa pre ne vynechávajú, pretože sa na ne nevzťahujú.
* Bitová hĺbka debayered/preview sa riadi nastavením *Calibrated image format*; radiancia je vždy float32.
* Spracovanie typu „Survey3

“ nie je ovplyvnené týmito štyrmi prepínačmi.

Tie isté štyri prepínače existujú aj bez hlavičky ako `chloros-cli process --debayered / --preview / --radiance / --reflectance` a ako zodpovedajúce parametre v súbore „SDK

“. Nahradili starý príznak `--radiometric-output`, ktorý už neexistuje.

{% hint style="warning" %}
**Vypnutie všetkých príslušných produktov spôsobí zlyhanie spustenia.** Od verzie 1.2.0 spracovávacia úloha, ktorá bola požiadaná o produkty, ale nezapísala žiadny obrazový produkt, nahlási zlyhanie a súborCLI

sa ukončí s nenulovou hodnotou namiesto tichého hlásenia úspechu. Protokol uvádza názov produktu, ktorý sa nepodarilo zapísať, a dôvod. Úloha zámerne zameraná len na metadáta (bez žiadnych požiadaviek) sa stále považuje za úspešnú.
{% endhint %}

### Zdroj odrazivosti (nastavenie projektu, nastavuje sa cezCLI

/SDK

)

Projekt tiež ukladá informáciu o tom, akú **referenciu odrazivosti** používa produkt odrazivosti LATTICE. V paneli nastavení nie je k dispozícii žiadny špeciálny ovládací prvok; hodnota je uložená v konfigurácii projektu ako `Processing → "Target reflectance source"` a nastavuje sa pomocou `chloros-cli process --reflectance-source {auto,target,daq}` alebo parametra `reflectance_source` v nástroji „SDK

“:

* **`auto`** (predvolené): kalibračný cieľ v rámci snímky, ktorý prešiel kontrolou kvality (QA), sa stáva absolútnou referenciou; ak cieľ nie je prítomný alebo neprejde kontrolou kvality, systém sa vráti k deleniu smerom nadol z DAQ (ρ = πL/E).
* **`target`**: prísna odrazivosť riadená cieľom — bez nahradenia údajmi z DAQ.
* **`daq`**: odrazivosť určovaná údajmi z DAQ; ciele v rámci snímky sa nepoužívajú ako referenčné hodnoty.

Uložená hodnota sa porovnáva bez ohľadu na veľkosť písmen a niekoľko variantov písania sa akceptuje ako synonymá: `target`, `target_image`, `empirical` a `empirical_line` všetky znamenajú **cieľ**; `daq`, `dls`, `light_sensor` a `sensor` všetky znamenajú**daq**. Všetko ostatné – vrátane chýbajúceho kľúča – sa premení na**auto**.

Skeny **meraných** cieľov na jednotku sa vyhľadávajú podľa sériového čísla/QR kódu cieľovej jednotky, napríklad `<serial>.csv`, na troch miestach: v adresári uvedenom s `--target-reflectance-dir` (uložený ako `Processing → "Target reflectance dir"`), vo vlastnej zložke projektu `target_reflectance/` a v ceste uvedenej v premennej prostredia `CHLOROS_TARGET_REFLECTANCE_DIR`. Ak pre danú jednotku neexistuje žiadne namerané skenovanie, namiesto toho sa použije nominálna uverejnená krivka pre cieľový model.

### Metóda demosaicingu

* **Typ**: Výber z roletového menu
* **Možnosti**:
  * Štandardná (rýchla, stredná kvalita)
  * S ohľadom na textúru (pomalá, najvyššia kvalita) \[Chloros

+]
* **Predvolené nastavenie**: Štandardná (rýchla, stredná kvalita)
* **Popis**: Vyberá algoritmus demosaicingu, ktorý sa používa na konverziu surových údajov zo senzora s Bayerovým vzorom na plnofarebné obrázky. Metóda „Štandardná (rýchla, stredná kvalita)“ poskytuje optimálnu rovnováhu medzi rýchlosťou spracovania a kvalitou obrazu. Metóda „Zohľadňujúca textúru (pomalá, najvyššia kvalita)“ \[Chloros

+] využíva vysokokvalitný algoritmus demosaicingu zohľadňujúci hrany v kombinácii s modelom odšumovania založeným na AI/ML, ktorý odstraňuje takmer všetok šum vznikajúci pri demosaicingu. Model „Texture Aware“ vyžaduje na spustenie pamäť GPU (VRAM). Odporúčame ho používať, ak máte k dispozícii &gt;4 GB VRAM pre rýchlejšie spracovanie.
* **V prípade, že je riadok roletovým menu**: roletové menu s dvoma možnosťami sa zobrazí len vtedy, ak platia**obe** podmienky — ste prihlásení s oprávneným predplatnýmChloros

+ a **projekt** neobsahuje žiadne snímky LATTICE. V opačnom prípade sa riadok zobrazí ako obyčajný text s textom `Standard (Fast, Medium Quality)` bez možnosti výberu.
* **Poznámka k LATTICE**: Neexistuje žiadny model LATTICEa pipeline vynúti štandardnú demosaic pre snímky LATTICE bez ohľadu na uloženú hodnotu. Ak pridáte priečinok LATTICE do projektu, v ktorom už bola zvolená možnosť Texture Aware, aplikáciaChloros

prepíše nastavenie späť na Štandardné namiesto toho, aby ponechala neaktuálnu hodnotu v `project.json`.

### Minimálny interval rekalibrácie

* **Typ**: Číslo
* **Rozsah**: 0 až 3 600 sekúnd
* **Predvolené nastavenie**: 0 sekúnd
* **Popis**: Nastavuje minimálny časový interval (v sekundách) medzi použitím kalibračných cieľov. Ak je nastavená hodnota 0, funkcia „Chloros

“ použije každý detekovaný kalibračný cieľ. Ak je nastavená vyššia hodnota, funkcia „Chloros

“ použije len kalibračné ciele, ktoré sú od seba vzdialené aspoň o tento počet sekúnd, čím sa skráti čas spracovania dátových súborov s častým zachytením kalibračných cieľov.
* **Kedy upraviť**:
  * Nastavte na 0 pre maximálnu presnosť kalibrácie pri meniacich sa svetelných podmienkach
  * Zvýšte hodnotu (napr. na 60–300 sekúnd) pre rýchlejšie spracovanie, ak sú svetelné podmienky stabilné a máte časté snímky kalibračných cieľov

### Posun časového pásma svetelného senzora

* **Typ**: Číslo
* **Rozsah**: -12 až +12 hodín
* **Predvolená hodnota**: 0 hodín
* **Popis**: Určuje posun časového pásma (v hodinách od UTC) pre časové značky údajov svetelného senzora, ktorý sa používa pri priraďovaní záznamov svetelného senzora k časom zachytenia snímok. Novšie záznamy `.daq` obsahujú vlastný časový pás, takže toto nastavenie je potrebné hlavne pre staršie záznamy zaznamenané v miestnom čase.

### Použiť korekcie PPK

* **Typ**: Zaškrtávacie políčko
* **Predvolené nastavenie**: Vypnuté (nezaškrtnuté)
* **Popis**: Umožňuje použitie korekcií Post-Processed Kinematic (PPK) z DAQ záznamníkovMAPIR

vybavených GPS (GNSS). Ak je táto funkcia zapnutá, aplikáciaChloros

použije všetky súbory protokolov .daq obsahujúce údaje o expozícii v adresári vášho projektu a uplatní na vaše snímky presné geolokačné korekcie.
* **Požiadavka**: V adresári projektu musí byť prítomný súbor protokolu .daq so záznamami o expozícii
* **Kedy povoliť**: Odporúča sa vždy povoliť korekciu PPK, ak máte v súbore protokolu .daq záznamy o expozícii.

### Expozícia Pin 1

* **Typ**: Výber z roletového menu
* **Viditeľnosť**: Viditeľné len vtedy, ak je povolená možnosť „Použiť korekcie PPK“ A sú k dispozícii údaje o expozícii pre pin 1
* **Možnosti**:
  * Názvy modelov kamier zistené v projekte
  * „Nepoužívať“ – tento expozičný pin ignorovať
* **Predvolené nastavenie**: Automaticky vybraný na základe konfigurácie projektu
* **Popis**: Priradí konkrétnu kameru k pinu expozície 1 na účely časovej synchronizácie PPK. Pin expozície zaznamenáva presný čas spustenia uzávierky kamery, čo je kľúčové pre presnú geolokalizáciu PPK.
* **Správanie automatického výberu**:
  * Jedna kamera + jeden pin: Automaticky vyberie kameru
  * Jedna kamera + dva piny: Pin 1 sa automaticky priradí ku kamere
  * Viac kamier: Je potrebný ručný výber

### Pin expozície 2

* **Typ**: Výber z roletového menu
* **Viditeľnosť**: Viditeľné len vtedy, ak je povolená možnosť „Použiť korekcie PPK“ A sú k dispozícii údaje o expozícii pre pin 2
* **Možnosti**:
  * Názvy modelov kamier zistené v projekte
  * „Nepoužívať“ – tento pin expozície ignorovať
* **Predvolené**: Automaticky vybraný na základe konfigurácie projektu
* **Popis**: Priradí konkrétnu kameru k pinu expozície 2 na účely časovej synchronizácie PPK pri použití konfigurácie s dvoma kamerami.
* **Správanie automatického výberu**:
  * Jedna kamera + jeden pin: Pin 2 sa automaticky nastaví na „Nepoužívať“
  * Jedna kamera + dva piny: Pin 2 sa automaticky nastaví na „Nepoužívať“
  * Viac kamier: Je potrebný ručný výber
* **Poznámka**: Tú istú kameru nie je možné priradiť súčasne k pinu 1 aj k pinu 2.***

## Svetelný senzor DAQ

Táto sekcia sa nachádza v nastaveniach projektu a obsahuje zoznam všetkých súborov DAQ s údajmi o dopadajúcom svetle v projekte — záznamy `.daq` a protokoly DAQ-M `.csv` o dopade svetla. Záznamy vytvorené na karte Svetelné senzory sa automaticky pridajú do otvoreného projektu.



<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->

Každý riadok zobrazuje súbor, model senzora a korekciu difúzneho krytu, ktorá sa pre daný súbor skutočne uplatňuje. Nad riadkami sa nachádza jediné ovládacie prvok platný pre celý projekt:

### Prepisovanie krytu (všetky súbory)

* **Typ**: Výber z roletového menu
* **Možnosti**: `Auto` plus profily korekcie difúzora platné pre typy senzorov prítomné v projekte
* **Predvolené**: Auto
* **Uložené ako**: `Processing → "DAQ cap id"` (predvolené `auto`)
* **Popis**: `Auto` používa zaznamenanú korekciu slnečného žiarenia z každého súboru (ak nebolo nič zaznamenané, predpokladá sa korekcia slnečného žiarenia – všetky zariadenia na zber údajovMAPIR

sa dodávajú s korektorom slnečného žiarenia). Výber konkrétneho krytu prepisuje **všetky** súbory s dopadajúcim žiarením v projekte: surové záznamy sa ním korigujú a záznamy, ktoré už obsahujú kryt, sa prereferencujú (zaznamenaná korekcia sa zruší a uplatní sa vybraná).
* **Dôležité**: Vybraný kryt sa musí zhodovať s krytom, ktorý bol fyzicky nasadený počas záznamu. Ani senzor, ani softvér nedokážu detekovať fyzický kryt – nezhoda ID krytu vedie k nesprávnej korekcii spektier.

Zámere je k dispozícii **jedno** ovládacie prvok pre celý projekt namiesto roletových menu pre každý súbor: toto nastavenie sa vzťahuje na každý zdroj smerujúci nadol v projekte.***

## Zarovnanie poľa

Táto sekcia sa zobrazí **len** vtedy, ak aspoň jeden obrázok v projekte obsahuje transformáciu zarovnania medzi modulmi, ktorú polia LATTICE vkladajú v čase zachytenia (značky XMP `Chloros:Alignment*`). Ukazuje, koľko snímok obsahuje značky zarovnania, ktorá kamera slúži ako referenčná (označenie `REF`), a tabuľku počtu snímok podľa jednotlivých kamier.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Použiť zarovnanie polí

* **Typ**: Zaškrtávacie políčko
* **Predvolené nastavenie**: Zapnuté (zaškrtnuté)
* **Uložené ako**: `Processing → "Array alignment"`
* **Popis**: Deformuje každý spracovaný produkt (debayered / náhľad / radiancia / odrazivosť / index) do spoločnej referenčnej geometrie poľa pomocou transformácie zaznamenanej v čase zachytenia. Vypnuté = export v natívnej geometrii jednotlivých senzorov.

### Orezať na spoločný prekrývací priestor

* **Typ**: Zaškrtávacie políčko (aktívne len pri zapnutej možnosti *Použiť zarovnanie poľa*)
* **Predvolené nastavenie**: Zapnuté (zaškrtnuté)
* **Uložené ako**: `Processing → "Array alignment crop"`
* **Popis**: Orezanie zarovná exportované údaje do oblasti, ktorú zdieľajú všetky kamerové moduly, takže každé pásmo má rovnakú stopu. Vypnutie zachováva plnú plochu snímača (čierna výplň mimo zdroj).

### Prevzorkovanie

* **Typ**: Výber z roletového menu (aktívne len pri zapnutej funkcii *Použiť zarovnanie poľa*)
* **Možnosti**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Predvolené**: Bilineárne
* **Uložené ako**: `Processing → "Array alignment interpolation"`
* **Popis**: Interpolácia používaná pri deformácii zarovnania. Najbližšia zachováva presné hodnoty zdroja (bez zmiešavania medzi pixelmi) pre prísnu radiometrickú analýzu; Bilineárna je najvhodnejšia pre mapovanie a vizuálne použitie.

Tie isté tri možnosti existujú aj bez predpony ako `chloros-cli process --array-alignment`, `--array-alignment-crop` a `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Index

Tieto nastavenia vám umožňujú konfigurovať multispektrálne indexy pre analýzu a vizualizáciu.

### Pridať index

* **Typ**: Špeciálny konfiguračný panel indexov
* **Popis**: Otvorí interaktívny panel, v ktorom môžete vybrať a nakonfigurovať multispektrálne vegetačné indexy (NDVI

,NDRE

,EVI

atď.), ktoré sa majú vypočítať počas spracovania snímky. Môžete pridať viacero indexov, z ktorých každý má vlastné nastavenia vizualizácie.
* **Dostupné indexy**: Rozbaľovacia ponuka v grafickom rozhraní obsahuje**27** preddefinovaných vzorcov multispektrálnych indexov (úplný zoznam vrátane názvov, ktoré akceptuje aj voľbaCLI

/SDK

`--indices`, nájdete v [Vzorcoch multispektrálnych indexov](multispectral-index-formulas.md)).
* **Funkcie**:
  * Výber z preddefinovaných vzorcov indexov
  * Presúvanie kanálov filtrov vašej kamery do pozícií pásiem vo vzorci
  * Konfigurácia farebných prechodov vizualizácie (LUT – vyhľadávacie tabuľky)
  * Nastavte prahové hodnoty a režimy orezávania
  * Vytvorte vlastné indexové vzorce
* **Poznámka**: Pre jednopásmové monochromatické kamery LATTICE M3M sa indexy nepočítajú — viacpásmové indexy sú na jednom pásme nedefinované. KamerySurvey3

a LATTICE M3C tým nie sú ovplyvnené.



<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->

Každý pridaný index vykresľuje svoj vzorec ako matematický výraz s farebným kruhom pre každý slot pásma: červená =Red

, zelená =Green

, modrá =Blue

, oranžová =Orange

, tyrkysová =Cyan

, fialová =NIR

, magenta = RE. Kruh z riadku nad vzorcom pretiahnite na slot, aby ste ho priradili; dvojitým kliknutím na priradený slot ho vymažete. Index sa vypočíta len vtedy, ak každý slot, ktorý vzorec používa, obsahuje kanál.

### Vlastné vzorce (funkciaChloros

+)

* **Typ**: Pole definícií vlastných vzorcov
* **Dostupnosť**: Vyžaduje prihlásenie sa pomocou platného predplatného službyChloros

+.
* **Popis**: Umožňuje vytvárať a ukladať vlastné multispektrálne indexové vzorce pomocou matematických operácií s pásmami. Vlastné vzorce sa ukladajú spolu s nastaveniami vášho projektu a dajú sa používať rovnako ako vstavané indexy.
* **Ako vytvoriť**:
  1. V paneli konfigurácie indexu otvorte kalkulátor vlastných vzorcov
  2. Napíšte vzorec pomocou **symbolov pásmových slotov**, nie názvov pásiem
  3. Uložte vzorec pod výstižným názvom – následne sa zobrazí v spodnej časti roletového menu vzorcov a kruhy kanálov vašej kamery môžete pretiahnuť do jeho slotov presne tak, ako pri vstavanej predvolbe
* **Syntax vzorca**:
  * Sloty pásiem: `x`, `y`, `z`, `a`, `b`, `c` — šesť pozícií, ktoré priradíte k skutočným kanálom pretiahnutím
  * Operátory: `+`, `-`, `*`, `/`, `^`, a `()` na zoskupovanie
  * Funkcie: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Prečo symboly a nie názvy pásiem**: vzorec zapísaný ako `(y-x)/(y+x)` funguje na akomkoľvek fotoaparáte, pretože mapovanie typu „drag-andurčuje, či je `y` 850 nmNIR

filtraRGN

alebo 808 nmNIR

filtraOCN

. Vstavané prednastavenia sú uložené rovnakým spôsobom — presnú symbolickú formu všetkých 27 nájdete v [Vzorcoch multispektrálnych indexov](multispectral-index-formulas.md).
* **Kde fungujú**: vlastné vzorce sa ukladajú spolu s nastaveniami projektu a dajú sa použiť v [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md), ako aj pri spracovaní. Nie sú**akceptované** zoznamom mienCLI

/SDK

`--indices`, ktorý rozširuje iba 22 názvov vstavaných predvolieb.

***

## Export

Tieto nastavenia riadia formát a kvalitu exportovaných spracovaných obrázkov.

### Formát kalibrovaného obrázku

* **Typ**: Výber z roletového menu
* **Možnosti**:
  * **TIFF

(16-bit)** – Nekomprimovaný 16-bitový formátTIFF


  * **TIFF

(32-bit, Percent)** – 32-bitový formát s plávajúcou desatinnou čiarkouTIFF

s hodnotami odrazivosti vyjadrenými v percentách
  * **PNG

(8-bit)** – Komprimovaný 8-bitový formátPNG


  * **JPG (8-bit)** – Komprimovaný 8-bitový formátJPEG


* **Predvolené**:TIFF

(16-bit)
* **Popis**: Vyberá formát súboru na ukladanie spracovaných a kalibrovaných snímok. Exportované súbory sa ukladajú do podpriečinkov podľa formátu v rámci priečinka každej kamery (`tiff16`, `tiff32`, `png8`, `jpg8`), pričom na každý produkt pripadá jeden priečinok `<Product>_Images/`. Exportované súbory si zachovávajú názov zdrojového súboru – produkt identifikuje priečinok, nie prípona súboru.
* **Odporúčania týkajúce sa formátu**:
  * **TIFF

(16-bit)**: Odporúča sa pre vedeckú analýzu a profesionálne pracovné postupy. Zachováva maximálnu kvalitu údajov bez kompresných artefaktov. Najvhodnejší pre multispektrálnu analýzu a ďalšie spracovanie v softvéri GIS.
  * **FormátTIFF

(32-bit, Percent)**: Najvhodnejší pre pracovné postupy, ktoré vyžadujú hodnoty odrazivosti vyjadrené v percentách (0–100 %). Ponúka maximálnu presnosť pre radiometrické merania.
  * **PNG

(8-bit)**: Vhodný pre prezeranie na webe a všeobecnú vizualizáciu. Menšie veľkosti súborov vďaka bezstratovej kompresii, avšak so zníženým dynamickým rozsahom.
  * **JPG (8-bit)**: Najmenšie veľkosti súborov, vhodné iba na náhľady a zobrazenie na webe. Používa kompresiu so stratou, ktorá nie je vhodná na vedeckú analýzu.
* **Poznámka**: Radiancia LATTICE sa vždy exportuje ako 32-bitové plávajúce desatinné čísTIFF

, bez ohľadu na toto nastavenie.

***

## Uloženie šablóny projektu

Táto funkcia vám umožňuje uložiť aktuálne nastavenia projektu ako šablónu, ktorú môžete opakovane používať.

* **Typ**: Textové pole + tlačidlo Uložiť
* **Popis**: Zadajte popisný názov pre šablónu nastavení a kliknite na ikonu uloženia. Šablóna uloží všetky vaše aktuálne nastavenia projektu (detekcia cieľov, možnosti spracovania, indexy, a formát exportu) pre jednoduché opätovné použitie v budúcich projektoch. Šablóny sa ukladajú do zložky `Project Templates/` vo vnútri zložky na ukladanie projektov a je možné ich vybrať alebo exportovať aj z hlavného menu (*Vybrať šablónu* / *Uložiť šablónu* / *Exportovať šablónu*).
* **Príklady použitia**:
  * Vytvorte šablóny pre rôzne kamerové systémy (RGB

ké, multispektrálne,NIR

)
  * Uložte štandardné konfigurácie pre konkrétne druhy plodín alebo pracovné postupy analýzy
  * Zdieľajte jednotné nastavenia v rámci tímu
* **Ako používať**:
  1. Nakonfigurujte všetky požadované nastavenia projektu
  2. Zadajte názov šablóny (napr. „RedEdge

Survey3

NDVI

Standard“)
  3. Kliknite na ikonu uloženia
  4. Šablónu je teraz možné načítať pri vytváraní nových projektov

***

## Uložiť priečinok projektu

Toto nastavenie určuje, kam sa nové projekty štandardne ukladajú.

* **Typ**: Zobrazenie cesty k adresáru + tlačidlo Upraviť
* **Predvolené (Windows

)**: `C:\Users\[Username]\Chloros Projects`
* **Predvolené (Linux

)**: `~/Chloros Projects`
* **Popis**: Zobrazuje aktuálny predvolený adresár, v ktorom sa vytvárajú nové projektyChloros

. Kliknutím na ikonu úpravy vyberte iný adresár. Prepis sa ukladá ako jeden riadok textu v súbore `~/.chloros/working_directory.txt` – na serveriWindows

je to súbor `C:\Users\<Username>\.chloros\working_directory.txt`. Ak tento súbor chýba, alebo obsahuje cestu, ktorá už neexistuje, aplikáciaChloros

sa vráti k vyššie uvedenému predvolenému nastaveniu. Spustiteľný súborCLI

číta a zapisuje do toho istého súboru, takže súbor `chloros-cli` a grafické rozhranie sa vždy zhodujú v tom, kde sa projekty nachádzajú.
* **Šablóny projektov** sa nachádzajú v podpriečinku `Project Templates/` tohto adresára.
* **Kedy zmeniť**:
  * Nastavte na sieťovú jednotku pre tímovú spoluprácu
  * Zmeňte na disk s väčším úložným priestorom pre veľké súbory údajov
  * Usporiadajte projekty podľa roka, klienta alebo typu projektu do rôznych zložiek
* **Poznámka**: Zmena tohto nastavenia ovplyvní len NOVÉ projekty. Existujúce projekty zostanú na svojich pôvodných miestach.***

## Trvalosť nastavení

ProjektChloros

je **zložka**. Všetky nastavenia projektu sa ukladajú v zložke `project.json` v jej vnútri; pripojený hardvér sa ukladá spolu s nimi v zložkách `cameras.json` a `sensors.json`, takže pri opätovnom otvorení projektu opätovne pripojí aj jeho kamery a svetelné senzory. Pri opätovnom otvorení projektu sa všetky nastavenia obnovia presne tak, ako ste ich nechali. Uložené projekty je možné ovládať aj bez grafického rozhrania pomocou súboru `chloros-cli project` alebo súboru `open_project` v zariadení „SDK

“.

### Hierarchia nastavení

Nastavenia sa uplatňujú v nasledujúcom poradí:

1. **Predvolené nastavenia systému** – Vstavané predvolené nastavenia definované v súboreChloros

2. **Nastavenia šablóny** – Ak pri vytváraní projektu načítajte šablónu
3. **Nastavenia uloženého projektu** – Nastavenia uložené spolu so súborom projektu
4. **Ručné úpravy** – Akékoľvek zmeny, ktoré vykonáte počas aktuálnej relácie

### Nastavenia a spracovanie obrázkov

Nastavenia spracovania sa načítajú pri spustení spracovania. Zmena nastavenia nemá spätný vplyv na výstupy, ktoré sú už uložené na disku — na uplatnenie nových nastavení je potrebné spustiť spracovanie znovu. Niektoré nastavenia nemajú na spracovanie žiadny vplyv:

* Rozlíšenie miniatúr obrázkov (len na zobrazenie)
* Uložiť šablónu projektu
* Uložiť priečinok projektu

***

## Referenčný zoznam konfiguračných kľúčov

Pre automatizáciu (CLI

`--config`,SDK

`configure` alebo pri priamom načítaní `project.json`) sú toto presné kľúče pod `Project Settings`:

| Cesta kľúča | Typ | Predvolená hodnota |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | číslo 0–10000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | číslo 0–100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | bool | `true` |
| `Processing → Export sensor response` | bool | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Štandardný |
| `Processing → Minimum recalibration interval` | číslo 0–3600 | `0` |
| `Processing → Light sensor timezone offset` | číslo -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | identifikátor profilu obmedzenia alebo `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | zoznam konfigurácií indexov | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Kľúče `Array alignment` sa zapíšu pri prvom vykreslení sekcie „Array Alignment“ alebo keď ich nastaví volanie automatizácie. Ak chýbajú, potrubie používa rovnaké hodnoty ako uvedené vyššie (`true`, `true`, bilineárne), takže projekt.json bez nich sa správa rovnako ako projekt s nimi.

### Kľúče uložené v `project.json` bez možnosti ovládania v paneli nastavení

Tieto sa nachádzajú v rovnakom strome `Project Settings` a sú načítané počas spracovania, avšak v bočnom paneli nenájdete žiadny ovládací prvok pre ne:

| Cesta kľúča | Typ | Predvolené | Nastavené |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`,SDK

`input_level=`. Prepíše spôsob interpretácie vstupných súborov TIFF v LATTICE; `auto` odvodí hodnotu z XMP značky každého súboru `Chloros:ProcessingLevel` a počtu kanálov. Ignoruje sa pri zachyteniachSurvey3

`.raw`. Úmyselne nie je nastavením v grafickom rozhraní — vo všetkých bežných prípadoch je správne nastavenie „auto“. |
| `Processing → Target reflectance dir` | reťazec cesty | `""` | `chloros-cli process --target-reflectance-dir`, alebo cieľ projektuAPI

|
| `Processing → Target reflectance config` | slovník s kľúčom podľa sériového čísla kamery | `{}` | Registrácia cieľa v rámci snímky (režim `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | reťazec cesty | `""` |SDK

`process_folder(daq_log_path=…)`. Odkazuje na záznam `.daq` alebo na zložku s takýmito záznamami |
| `Target Detection → Minimum calibration target squares` | číslo | `4` | Staré predvolené nastavenie; bez ovládacích prvkov a bez príznaku „CLI

“ |
| `UI → Grid thumbnail size` | číslo | `160` | Posúvač priblíženia miniatúr v mriežke obrázkov |

Dve nastavenia prehliadača sú uložené **na najvyššejúrovni v `project.json`**, úplne mimo `Project Settings`, pretože ide o stav zobrazenia a nie o nastavenia spracovania:

| Kľúčová cesta | Typ | Predvolená hodnota | Nastavuje |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | celé číslo 1–256 | `1` | Ovládací prvok GSD (px) na karte obrázku — pozri [Otvorenie obrázku na celú obrazovku](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Osvedčené postupy

1. **Začnite s predvolenými nastaveniami**: Predvolené nastavenia fungujú dobre pre väčšinu kamerových systémovMAPIR

a typických pracovných postupov.
2. **Vytvorte šablóny**: Akonáhle optimalizujete nastavenia pre konkrétny pracovný postup alebo kameru, uložte ich ako šablónu, aby ste zabezpečili konzistentnosť vo všetkých projektoch.
3. **Pred úplným spracovaním otestujte**: Keď experimentujete s novými nastaveniami, otestujte ich na malej podmnožine obrázkov, než spracujete celý súbor údajov.
4. **Zaznamenajte si svoje nastavenia**: Používajte popisné názvy šablón, ktoré uvádzajú kamerový systém, typ spracovania a zamýšľané použitie (napr. „Survey3

\_RGB\_NDVI\_Agriculture“).
5. **Výber formátu exportu**: Formát exportu si vyberte podľa konečného použitia:
   * Vedecká analýza →TIFF

(16-bitový alebo 32-bitový)
   * Spracovanie v GIS →TIFF

(16-bitový)
   * Rýchla vizualizácia →PNG

(8-bitový)
   * Zdieľanie na webe → JPG (8-bitový)

***

Ďalšie informácie o multispektrálnych indexoch v nástroji „Chloros

“ nájdete na stránke [Vzorce multispektrálnych indexov](multispectral-index-formulas.md).

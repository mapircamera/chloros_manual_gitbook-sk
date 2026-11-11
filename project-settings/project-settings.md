# Nastavenia projektu

Bočný panel „Nastavenia projektu“ <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

v programe Chloros

vám umožňuje konfigurovať všetky aspekty spracovania snímok, detekcie kalibračných cieľov, výpočtov multispektrálnych indexov a možností exportu pre váš projekt. Tieto nastavenia sa ukladajú spolu s vaším projektom a je možné ich uložiť ako šablóny na opätovné použitie vo viacerých projektoch.

## Prístup k nastaveniam projektu

Prístup k nastaveniam projektu:

1. Otvorte projekt v programe Chloros


2. Kliknite na kartu **Nastavenia projektu** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

v ľavom bočnom paneli
3. V paneli nastavení sa zobrazia všetky dostupné konfiguračné možnosti zoradené podľa kategórií

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->



{% hint style="info" %}
**Nastavenia, ktoré závisia od iných nastavení, sú sivé.** Ak nadradené prepnutie znemožňuje určité nastavenie (napríklad zrušenie zaškrtnutia položky *Kalibrácia odrazivosti / vyváženie bielej* znemožňuje *Export odrazivosti*), závislé ovládacie prvky sú deaktivované a v ich popise je uvedený názov prepnutia, ktoré je potrebné zmeniť.
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

Tieto nastavenia riadia, ako program Chloros

detekuje a spracováva kalibračné ciele vo vašich obrázkoch. Obe sú aktívne iba vtedy, keď je zapnutá funkcia **Kalibrácia odrazivosti / vyváženie bielej** (inak sú sivé, pretože detekcia cieľov sa úplne preskočí).

### Minimálna plocha kalibračnej vzorky (px)

* **Typ**: Číslo
* **Rozsah**: 0 až 10 000 pixelov
* **Predvolené nastavenie**: 25 pixelov
* **Popis**: Nastavuje minimálnu plochu (v pixeloch) potrebnú na to, aby bola detegovaná oblasť považovaná za platnú vzorku kalibračného cieľa. Menšie hodnoty detegujú menšie ciele, ale môžu zvýšiť počet falošných pozitív. Väčšie hodnoty vyžadujú na detekciu väčšie a jasnejšie oblasti cieľov.
* **Kedy upraviť**:
  * Zvýšte hodnotu, ak sa vyskytujú falošné detekcie na malých artefaktoch v obraze
  * Znížte hodnotu, ak sa vaše kalibračné ciele na obrazoch javia ako malé a nie sú detekované

### Minimálne zhlukovanie cieľov (0–100)

* **Typ**: Číslo
* **Rozsah**: 0 až 100
* **Predvolená hodnota**: 60
* **Popis**: Ovláda prah zhlukovania pre zoskupovanie podobne sfarbených oblastí pri detekcii kalibračných cieľov. Vyššie hodnoty vyžadujú, aby sa zoskupovali podobnejšie farby, čo vedie k konzervatívnejšej detekcii cieľov. Nižšie hodnoty umožňujú väčšiu farebnú variabilitu v rámci jednej skupiny cieľov.
* **Kedy nastaviť**:
  * Zvýšte hodnotu, ak sa kalibračné ciele rozdeľujú na viacero detekcií
  * Znížte hodnotu, ak sa kalibračné ciele s farebnými variáciami nedetekujú v plnom rozsahu

***

## Spracovanie

Tieto nastavenia určujú, ako program Chloros

spracováva a kalibruje vaše snímky.

### Korekcia vinetácie

* **Typ**: Zaškrtávacie políčko
* **Predvolené nastavenie**: Zapnuté (zaškrtnuté)
* **Popis**: Uplatňuje korekciu vinetácie na kompenzáciu stmavnutia objektívu na okrajoch snímok. Vinetácia je bežný optický jav, pri ktorom sa rohy a okraje snímky javia tmavšie ako stred v dôsledku vlastností objektívu.
* **Vedľajší účinok**: Týmto prepínačom sa tiež vyberá, ktorý *nekalibrovaný náhradný produkt* sa pri spracovaní uloží (pozri nižšie).

### Kalibrácia odrazivosti / vyváženie bielej

* **Typ**: Zaškrtávacie políčko
* **Predvolené nastavenie**: Zapnuté (zaškrtnuté)
* **Popis**: Aktivuje kalibráciu odrazivosti — na základe detekovaných kalibračných cieľov v zábere a/alebo údajov o dopadajúcom svetle zo svetelného senzora DAQ, v závislosti od kamery a dostupných údajov. Tým sa normalizujú hodnoty odrazivosti v celom súbore údajov a zabezpečia sa konzistentné merania bez ohľadu na svetelné podmienky.
* **Keď je vypnutá**: Detekcia cieľov sa úplne preskočí a**žiadna kamera nemôže vytvoriť žiadny produkt odrazivosti** — či už ide o systém riadený cieľmi Survey3

, alebo systém riadený DAQ LATTICE. Závislé nastavenia (*Exportovať odrazivosť*, *Minimálny interval rekalibrácie* a prahové hodnoty detekcie cieľov) sú sivé.

### Náhradné produkty bez kalibrácie: Exportovať odozvu senzora / Exportovať s korekciou vinety

* **Typ**: Dve zaškrtávacie políčka
* **Predvolené nastavenia**: Obe povolené (zaškrtnuté)
* **Popis**: Ak nie je možné kalibrovať odrazivosť snímky (nebol nájdený žiadny kalibračný cieľ alebo je kalibrácia odrazivosti vypnutá), je namiesto toho uložená ako *nekalibrovaný náhradný produkt*. **Na každý beh existuje presne jeden z dvoch náhradných produktov pre každý model kamery**, vybraný prepínačom *Korekcia vinety*:
  * Korekcia vinetácie **zapnutá**→ `Vignette_Corrected_Images/` (riadi sa nastavením**Exportovať s korekciou vinetácie**)
  * Korekcia vinetácie **vypnutá**→ `Sensor_Response_Images/` (riadi sa nastavením**Exportovať odozvu senzora**)
* Náhradný produkt, ktorý nie je aktívny, je sivý. Odškrtnutím aktívneho produktu sa zabráni zapísaniu daného súboru.

### Výstupné produkty LATTICE

V prípade projektov obsahujúcich zábery z LATTICE sa každý importovaný snímok z LATTICE rozdelí do všetkých povolených **a príslušných**produktov v rámci jedného spracovateľského cyklu. Rozdelenie riadia štyri zaškrtávacie políčka (všetky sú predvolene**zapnuté**):

| Nastavenie | Výstupná zložka | Čo sa exportuje |
| --- | --- | --- |
| **Exportovať bez debayeringu** | `Debayered_Images/` | Lineárny obraz bez debayeringu: odstránený tmavý posun kalibračného balíka, následne demosaik s ohľadom na okraje, bez korekcie linearity, plochého poľa alebo zmiešavania — ten istý súbor, aký záznam ukladá na úrovni bez debayeringu. Zachovaná natívna orientácia snímača (zaznamenaná v XMP). Platí pre RGB

a multispektrálne kamery. |
| **Export náhľadu** | `Preview_Images/` | Náhľad na displeji. RGB

= vyváženie bielej (svetelný zdroj DAQ, ak je k dispozícii, inak šedý svet) + gama; multispektrálny = rozloženie falošných farieb. |
| **Export žiarivosti** | `Radiance_Images/` | Spektrálna žiarivosť typu Float32 v jednotkách W/m²/sr/nm. Iba pre multispektrálne snímky (M3C/M3M) — neplatí pre RGB

originály. Vždy sa zapisuje ako 32-bitový TIFF

bez ohľadu na nastavenie *Kalibrovaný formát obrázku*. |
| **Export odrazivosti**| `Reflectance_Calibrated_Images/` | Odrazivosť typu Uint16, škálovaná tak, že**32768 = odrazivosť 1,0** (označené ako XMP `Chloros:PixelScale`). Iba multispektrálne, zapísané vtedy, keď snímku pokrýva zodpovedajúci záznam `.daq` smerujúci nadol (alebo cieľ v rámci snímky, ktorý prešiel kontrolou kvality). |

* Hlavné kamery RGB

vysielajú údaje po odstránení bayesovského vzoru + náhľad; žiarivosť/odrazivosť sa u nich vynechávajú, pretože sa na ne nevzťahujú.
* Bitová hĺbka údajov po odstránení bayesovského vzoru/náhľadu sa riadi nastavením *Formát kalibrovaného obrazu*; žiarivosť je vždy typu float32.
* Tieto štyri prepínače nemajú vplyv na spracovanie typu *Survey3

*.
* Produkty žiarivosti a odrazivosti multispektrálnych kamier M3C môžu tiež zapísať **masku obmedzenia** do priečinka `clamp_masks/` vedľa priečinkov s produktmi, čím zaznamenávajú miesta, kde sa pri oddeľovaní pásiem záporná hodnota obmedzila na nulu. Pozri [Formáty výstupných obrázkov](../output-image-formats.md#clamp-masks).

Tie isté štyri prepínače existujú aj v bezhlavom režime ako `chloros-cli process --debayered / --preview / --radiance / --reflectance`. Nahradili starý príznak CLI

u `--radiometric-output`, ktorý už neexistuje.

{% hint style="warning" %}
**Vypnutie všetkých príslušných výstupov spôsobí zlyhanie spracovania.** Od verzie 1.2.0 spracovanie, pri ktorom boli požadované výstupy, ale neboli zapísané žiadne obrazové výstupy, hlási zlyhanie a CLI

sa ukončí s nenulovým kódom, namiesto toho, aby hlásilo tiché úspešné dokončenie. Protokol uvádza názov výstupu, ktorý sa nepodarilo zapísať, a dôvod. Úmyselné spustenie len s metadátami (nebolo požiadané o nič) sa stále považuje za úspešné.
{% endhint %}

### Zdroj odrazivosti (nastavenie projektu, nastavuje sa cez CLI

)

Projekt tiež ukladá informáciu o tom, akú **referenciu odrazivosti** používa produkt odrazivosti LATTICE. V paneli nastavení nie je k dispozícii žiadny špeciálny ovládací prvok; hodnota je uložená v konfigurácii projektu ako `Processing → "Target reflectance source"` a nastavuje sa pomocou `chloros-cli process --reflectance-source {auto,target,daq}` (SDK

v položke `configure()` nemá pre to žiadne kľúčové slovo):

* **`auto`** (predvolené): cieľ kalibrácie v rámci snímky, ktorý prešiel kontrolou kvality (QA), sa stáva absolútnou referenciou; ak nie je k dispozícii žiadny cieľ alebo ak kontrola kvality zlyhá, systém sa vráti k deleniu dopadajúceho žiarenia z DAQ (ρ = πL/E). Cieľ, ktorý prešiel kontrolou kvality s výhradou (vypadnutý panel alebo nefunkčné pásmo), nie je automaticky uprednostňovaný: reťazce cieľ, cieľ+DAQ a iba DAQ sa vyskúšajú v poradí podľa ich nameraného skóre na vynechaných paneloch.
* **`target`**: prísna odrazivosť riadená cieľom — bez nahradenia DAQ.
* **`daq`**: odrazivosť s prioritou DAQ; ciele v rámci snímky sa nepoužívajú ako referenčné.

Uložená hodnota sa porovnáva bez ohľadu na veľkosť písmen a niekoľko variantov písania sa akceptuje ako synonymá: `target`, `target_image`, `empirical` a `empirical_line` – všetky znamenajú **cieľ**; `daq`, `dls`, `light_sensor` a `sensor` všetky znamenajú**daq**. Všetko ostatné — vrátane chýbajúceho kľúča — sa premení na**auto**.

Skeny **meraných** cieľov na jednotku sa vyhľadávajú podľa sériového čísla/QR kódu cieľovej jednotky, napríklad `<serial>.csv`, na troch miestach: v adresári zadanom pomocou `--target-reflectance-dir` (uloženom ako `Processing → "Target reflectance dir"`), vo vlastnej zložke projektu `target_reflectance/`, a v ceste uvedenej v premennej prostredia `CHLOROS_TARGET_REFLECTANCE_DIR`. Ak pre danú jednotku neexistuje žiadny nameraný sken, namiesto toho sa použije nominálna uverejnená krivka pre cieľový model.

### Metóda odstránenia bayerského filtra

* **Typ**: Výber z roletového menu
* **Možnosti**:
  * Štandardná (rýchla, stredná kvalita)
  * S ohľadom na textúru (pomalá, najvyššia kvalita) \[Chloros

+]
* **Predvolené**: Štandardná (rýchla, stredná kvalita)
* **Popis**: Vyberá algoritmus demosaicingu používaný na konverziu surových údajov zo senzora s Bayerovým vzorom na plnofarebné obrázky. Metóda „Štandardná (rýchla, stredná kvalita)“ poskytuje optimálnu rovnováhu medzi rýchlosťou spracovania a kvalitou obrazu. Metóda „S ohľadom na textúru (pomalá, najvyššia kvalita)“ \[Chloros

+] využíva vysokokvalitný algoritmus demosaicingu zohľadňujúci hrany v kombinácii s modelom odšumovania založeným na AI/ML, ktorý odstraňuje takmer všetok šum vznikajúci pri demosaicingu. Model zohľadňujúci textúru vyžaduje na spustenie pamäť GPU (VRAM). Odporúčame ho používať, ak máte k dispozícii &gt;4 GB VRAM, aby bolo spracovanie rýchlejšie.
* **Ak je riadok vôbec roletovým menu**: roletové menu s dvoma možnosťami sa zobrazí len vtedy, ak**obe** podmienky sú splnené — ste prihlásení s platným predplatným Chloros

+ a **projekt** neobsahuje žiadne zábery LATTICE. V opačnom prípade sa riadok zobrazí ako obyčajný text s textom `Standard (Fast, Medium Quality)` bez možnosti výberu.
* **Poznámka k LATTICE**: Neexistuje žiadny model Texture Aware vytrénovaný pre LATTICE a spracovateľský reťazec vynúti štandardnú demosaiku pre snímky LATTICE bez ohľadu na uloženú hodnotu. Ak pridáte priečinok LATTICE do projektu, v ktorom už bola zvolená možnosť Texture Aware, Chloros

prepíše toto nastavenie späť na Štandardné, namiesto toho, aby ponechal neaktuálnu hodnotu v `project.json`.

### Minimálny interval rekalibrácie

* **Typ**: Číslo
* **Rozsah**: 0 až 3 600 sekúnd
* **Predvolené nastavenie**: 0 sekúnd
* **Popis**: Nastavuje minimálny časový interval (v sekundách) medzi použitím kalibračných cieľov. Ak je nastavená hodnota 0, Chloros

použije každý detegovaný kalibračný cieľ. Ak je nastavená vyššia hodnota, Chloros

použije len kalibračné ciele, ktoré sú od seba vzdialené aspoň o tento počet sekúnd, čím sa skráti čas spracovania dátových súborov s častým zachytením kalibračných cieľov.
* **Kedy nastaviť**:
  * Nastavte na 0 pre maximálnu presnosť kalibrácie pri meniacich sa svetelných podmienkach
  * Zvýšte hodnotu (napr. na 60–300 sekúnd) pre rýchlejšie spracovanie, ak sú svetelné podmienky stabilné a máte časté snímky kalibračných cieľov

### Časové posunutie svetelného senzora

* **Typ**: Číslo
* **Rozsah**: -12 až +12 hodín
* **Predvolené nastavenie**: 0 hodín
* **Popis**: Určuje časový posun (v hodinách od UTC) pre časové značky údajov svetelného senzora, ktorý sa používa pri priraďovaní záznamov svetelného senzora k časom zachytenia snímok. Novšie záznamy `.daq` obsahujú vlastný časový posun, takže toto nastavenie je potrebné hlavne pre staršie záznamy zaznamenané v miestnom čase.

### Použiť korekcie PPK

* **Typ**: Zaškrtávacie políčko
* **Predvolené nastavenie**: Vypnuté (nezatrhnuté)
* **Popis**: Umožňuje použitie korekcií Post-Processed Kinematic (PPK) zo záznamníkov MAPIR

DAQ vybavených GPS (GNSS). Ak je táto funkcia zapnutá, Chloros

použije všetky súbory protokolov .daq obsahujúce údaje o expozícii vo vašom projektovom adresári a na vaše snímky uplatní presné korekcie geolokácie.
* **Požiadavka**: V adresári projektu musí byť prítomný súbor protokolu .daq so záznamami o expozícii
* **Kedy povoliť**: Odporúča sa vždy povoliť korekciu PPK, ak máte v súbore protokolu .daq záznamy o expozícii.

### Expozícia – pin 1

* **Typ**: Výber z roletového menu
* **Viditeľnosť**: Viditeľné len vtedy, keď je povolená voľba „Použiť korekcie PPK“ A sú k dispozícii údaje o expozícii pre bod 1
* **Možnosti**:
  * Názvy modelov fotoaparátov zistené v projekte
  * „Nepoužívať“ – tento bod expozície ignorovať
* **Predvolené nastavenie**: Automaticky vybrané na základe konfigurácie projektu
* **Popis**: Priradí konkrétnu kameru k kolíku expozície 1 na účely časovej synchronizácie PPK. Kolík expozície zaznamenáva presný čas spustenia uzávierky kamery, čo je kľúčové pre presnú geolokalizáciu PPK.
* **Správanie automatického výberu**:
  * Jedna kamera + jeden pin: Automaticky vyberie kameru
  * Jedna kamera + dva piny: Pin 1 sa automaticky priradí ku kamere
  * Viac kamier: Je potrebný ručný výber

### Expozíciový pin 2

* **Typ**: Výber z roletového menu
* **Viditeľnosť**: Viditeľné len vtedy, ak je povolená možnosť „Použiť korekcie PPK“ A sú k dispozícii údaje o expozícii pre pin 2
* **Možnosti**:
  * Názvy modelov kamier zistené v projekte
  * „Nepoužívať“ – tento pin expozície ignorovať
* **Predvolené nastavenie**: Automaticky vybraný na základe konfigurácie projektu
* **Popis**: Priradí konkrétnu kameru k pinu expozície 2 na účely časovej synchronizácie PPK pri použití konfigurácie s dvoma kamerami.
* **Správanie automatického výberu**:
  * Jedna kamera + jeden pin: Pin 2 sa automaticky nastaví na „Nepoužívať“
  * Jedna kamera + dva piny: Pin 2 sa automaticky nastaví na „Nepoužívať“
  * Viac kamier: Je potrebný ručný výber
* **Poznámka**: Tú istú kameru nie je možné priradiť súčasne k pinu 1 aj k pinu 2.***

## Svetelný senzor DAQ

Táto sekcia sa nachádza v nastaveniach projektu a obsahuje zoznam všetkých súborov DAQ s údajmi o dopadajúcom slnečnom žiarení v projekte — záznamy `.daq` a DAQ-M `.csv`. Záznamy vytvorené na karte Svetelné senzory sa automaticky pridajú do otvoreného projektu.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->



Každý riadok zobrazuje súbor, model senzora a korekciu difúzneho krytu, ktorá sa pre daný súbor skutočne uplatňuje. Nad riadkami sa nachádza jediné ovládacie prvok platný pre celý projekt:

### Prepisovanie nastavení krytu (všetky súbory)

* **Typ**: Výber z roletového menu
* **Možnosti**: `Auto` plus profily korekcie difúzora platné pre typy senzorov prítomné v projekte
* **Predvolené**: Auto
* **Uložené ako**: `Processing → "DAQ cap id"` (predvolené `auto`)
* **Popis**: `Auto` používa kryt zaznamenaný v každom súbore (ak nebolo zaznamenané nič, predpokladá sa kryt Sunshine, avšak zariadenia DAQ-U, DAQ-M a DAQ-E sa dodávajú bez krytu a program Chloros

nedokáže zistiť, aký kryt je namontovaný, preto spracovanie varuje pred predpokladaným krytom: vyberte kryt, ktorý bol namontovaný). Výber konkrétneho krytu prepisuje **všetky** súbory smerom nadol v projekte okrem súborov z DAQ-E-S, ktorých difúzor je súčasťou ich kalibrácie; projekt obsahujúci iba DAQ-E-S nezobrazuje roletové menu: surové záznamy sa korigujú pomocou neho, a záznamy, ktoré už obsahujú krytku, sa prekalibrujú (zaznamenaná korekcia sa zruší a uplatní sa vybraná).
* **Dôležité**: Vybraná krytka sa musí zhodovať s krytkou, ktorá bola fyzicky namontovaná počas záznamu. Ani senzor, ani softvér nedokážu detekovať fyzickú krytku — nezhodné ID krytky nesprávne koriguje spektrá.

Zámere je tu **jedno** nastavenie pre celý projekt namiesto roletových menu pre jednotlivé súbory: toto nastavenie sa vzťahuje na každý zdroj smerujúci nadol v projekte.***

## Zarovnanie poľa

Táto sekcia sa zobrazí **len** vtedy, ak aspoň jeden obrázok v projekte obsahuje transformáciu zarovnania medzi modulmi, ktorú matice LATTICE vkladajú v čase zachytenia (značky XMP `Chloros:Alignment*`). Ukazuje, koľko snímok obsahuje značky zarovnania, ktorá kamera slúži ako referenčná (značka `REF`), a tabuľku počtu snímok pre každú kameru.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Použiť zarovnanie poľa

* **Typ**: Zaškrtávacie políčko
* **Predvolené nastavenie**: Zapnuté (zaškrtnuté)
* **Uložené ako**: `Processing → "Array alignment"`
* **Popis**: Deformuje každý spracovaný výstup (debayering / náhľad / radiancia / odrazivosť / index) do spoločnej referenčnej geometrie poľa pomocou transformácie zaznamenanej pri snímaní. Vypnuté = export v natívnej geometrii jednotlivých senzorov.

### Orezať na spoločný prekrývajúci sa úsek

* **Typ**: Zaškrtávacie políčko (aktívne len pri zapnutej funkcii *Použiť zarovnanie poľa*)
* **Predvolené nastavenie**: Zapnuté (zaškrtnuté)
* **Uložené ako**: `Processing → "Array alignment crop"`
* **Popis**: Orezanie zarovná exportované údaje do oblasti, ktorú zdieľajú všetky kamerové moduly, takže každé pásmo má rovnakú plochu. Vypnuté zachováva plnú plochu snímača (čierna výplň mimo zdroj).

### Prevzorkovanie

* **Typ**: Rozbaľovacie menu (aktívne len pri zapnutej možnosti *Použiť zarovnanie poľa*)
* **Možnosti**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Predvolené nastavenie**: Bilineárne
* **Uložené ako**: `Processing → "Array alignment interpolation"`
* **Popis**: Interpolácia používaná pri deformácii zarovnania. Možnosť „Najbližšia“ zachováva presné hodnoty zdroja (bez zmiešavania medzi pixelmi) pre prísnu radiometrickú analýzu; možnosť „Bilineárna“ je najvhodnejšia pre mapovanie a vizuálne použitie.

Tie isté tri možnosti existujú aj bez predpony ako `chloros-cli process --array-alignment`, `--array-alignment-crop` a `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Index

Tieto nastavenia vám umožňujú konfigurovať multispektrálne indexy na účely analýzy a vizualizácie.

### Pridať index

* **Typ**: Špeciálny konfiguračný panel indexov
* **Popis**: Otvorí interaktívny panel, v ktorom môžete vybrať a nakonfigurovať multispektrálne vegetačné indexy (NDVI

, NDRE

, EVI

atď.), ktoré sa majú vypočítať počas spracovania snímok. Môžete pridať viacero indexov, z ktorých každý má svoje vlastné nastavenia vizualizácie.
* **Dostupné indexy**: Rozbaľovacie menu v grafickom rozhraní obsahuje**27** preddefinovaných vzorcov multispektrálnych indexov (úplný zoznam vrátane názvov, ktoré akceptuje aj CLI

/SDK

`--indices`).
* **Funkcie**:
  * Výber z preddefinovaných vzorcov indexov
  * Presúvanie filtračných kanálov vašej kamery do pozícií pásiem vo vzorci
  * Konfigurácia farebných prechodov vizualizácie (LUT – vyhľadávacie tabuľky)
  * Nastavte prahové hodnoty a režimy orezávania
  * Vytvorte vlastné vzorce indexov
* **Poznámka**: Pre jednopásmové monochromatické kamery LATTICE M3M sa indexy nepočítajú — viacpásmové indexy sú na jednom pásme nedefinované. Kamery Survey3

a LATTICE M3C tým nie sú ovplyvnené.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->



Každý pridaný index zobrazuje svoj vzorec ako matematický výraz s farebným kruhom pre každý slot pásma: červená = Red

, zelená = Green

, modrá = Blue

, oranžová = Orange

, cyan = Cyan

, fialová = NIR

, magenta = RE. Kruh z riadku nad vzorcom pretiahnite na slot, aby ste ho priradili; dvojitým kliknutím na priradený slot ho vymažete. Index sa vypočíta len vtedy, ak každý slot, ktorý vzorec používa, obsahuje kanál.

### Vlastné vzorce (funkcia Chloros

+)

* **Typ**: Pole definícií vlastných vzorcov
* **Dostupnosť**: Vyžaduje prihlásenie s oprávneným predplatným Chloros

+.
* **Popis**: Umožňuje vytvárať a ukladať vlastné vzorce multispektrálnych indexov pomocou matematických operácií s pásmami. Vlastné vzorce sa ukladajú spolu s nastaveniami projektu a dajú sa používať rovnako ako vstavané indexy.
* **Ako vytvoriť**:
  1. V paneli konfigurácie indexu otvorte kalkulátor vlastných vzorcov
  2. Napíšte vzorec pomocou **symbolov pásiem a slotov**, nie názvov pásiem
  3. Vzorec uložte pod výstižným názvom – následne sa zobrazí v spodnej časti roletového menu vzorcov a kruhy kanálov vašej kamery môžete pretiahnuť do príslušných slotov presne tak, ako pri vstavanej predvolbe
* **Syntax vzorca**:
  * Sloty pásiem: `x`, `y`, `z`, `a`, `b`, `c` — šesť pozícií, ktoré priradíte k skutočným kanálom ťahaním
  * Operátory: `+`, `-`, `*`, `/`, `^` a `()` na zoskupovanie
  * Funkcie: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Prečo symboly a nie názvy kapiel**: vzorec zapísaný ako `(y-x)/(y+x)` funguje na akomkoľvek fotoaparáte, pretože priradenie pomocou drag-and-drop rozhoduje, či `y` je 850 nm NIR

filtra RGN

, alebo 808 nm NIR

filtra OCN

. Vstavané prednastavenia sú uložené rovnakým spôsobom — presnú symbolickú formu všetkých 27 nájdete v [Vzorcoch multispektrálnych indexov](multispectral-index-formulas.md).
* **Kde fungujú**: vlastné vzorce sa ukladajú spolu s nastaveniami projektu a dajú sa použiť v [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) aj pri spracovaní. CLI

/SDK

`--indices` ich **neakceptuje** v zozname mien, ktorý rozširuje iba 22 mien vstavaných predvolieb.***

## Export

Tieto nastavenia riadia formát a kvalitu exportovaných spracovaných obrázkov.

### Formát kalibrovaného obrázku

* **Typ**: Výber z roletového menu
* **Možnosti**:
  * **TIFF

(16-bit)** – Nekomprimovaný 16-bitový formát TIFF


  * **TIFF

(8-bit)** – nekomprimovaný 8-bitový formát TIFF


  * **TIFF

(32-bit, Percent)** – 32-bitový formát TIFF

s plávajúcou desatinnou čiarkou, kde sú hodnoty odrazivosti vyjadrené v percentách
  * **PNG

(8-bit)** – komprimovaný 8-bitový formát PNG


  * **JPG (8-bitový)** – Komprimovaný 8-bitový formát JPEG


* **Predvolené**: TIFF

(16-bitový); projekt, v ktorom sú všetky údaje v 8-bitovom formáte, zobrazuje TIFF

(8-bitový) a ponúka iba 8-bitové formáty
* **Popis**: Slúži na výber formátu súboru na uloženie spracovaných a kalibrovaných snímok. Exportované súbory sa ukladajú do podpriečinkov podľa formátu v rámci priečinka každej kamery (`tiff16`, `tiff32`, `png8`, `jpg8`), pričom na každý produkt pripadá jeden priečinok `<Product>_Images/`. Exportované súbory si zachovávajú názov zdrojového súboru – produkt identifikuje priečinok, nie prípona súboru.
* **Odporúčania týkajúce sa formátov**:
  * **TIFF

(16-bitový)**: Odporúča sa pre vedeckú analýzu a profesionálne pracovné postupy. Zachováva maximálnu kvalitu údajov bez kompresných artefaktov. Najvhodnejší pre multispektrálnu analýzu a ďalšie spracovanie v softvéri GIS.
  * **Formát TIFF

(32-bitový, percentuálny)**: Najvhodnejší pre pracovné postupy, ktoré vyžadujú hodnoty odrazivosti vyjadrené v percentách (0–100 %). Ponúka maximálnu presnosť pre radiometrické merania.
  * **PNG

(8-bit)**: Vhodný na prezeranie na webe a všeobecnú vizualizáciu. Menšie veľkosti súborov vďaka bezstratovej kompresii, avšak so zníženým dynamickým rozsahom.
  * **JPG (8-bit)**: Najmenšie veľkosti súborov, vhodné iba na náhľady a zobrazenie na webe. Používa kompresiu so stratou, ktorá nie je vhodná na vedeckú analýzu.
* **Poznámka**: Radiancia LATTICE sa vždy exportuje ako 32-bitový TIFF

s plávajúcou desatinnou čiarkou bez ohľadu na toto nastavenie.

***

## Uloženie šablóny projektu

Táto funkcia vám umožňuje uložiť aktuálne nastavenia projektu ako šablónu, ktorú môžete opakovane používať.

* **Typ**: Textové pole + tlačidlo Uložiť
* **Popis**: Zadajte popisný názov pre šablónu nastavení a kliknite na ikonu uloženia. Šablóna uloží všetky vaše aktuálne nastavenia projektu (detekcia cieľov, možnosti spracovania, indexy, a formát exportu) pre jednoduché opätovné použitie v budúcich projektoch. Šablóny sa ukladajú do zložky `Project Templates/` vo vnútri zložky na ukladanie projektov a je možné ich vybrať alebo exportovať aj z hlavného menu (*Vybrať šablónu* / *Uložiť šablónu* / *Exportovať šablónu*).
* **Príklady použitia**:
  * Vytvorte šablóny pre rôzne kamerové systémy (RGB

, multispektrálne, NIR

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

## Zložka na ukladanie projektov

Toto nastavenie určuje, kam sa nové projekty štandardne ukladajú.

* **Typ**: Zobrazenie cesty k adresáru + tlačidlo Upraviť
* **Predvolené (systém Windows

)**: `Documents\MAPIR\Chloros Projects` vo vašom priečinku Dokumenty — zvyčajne `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`, alebo kdekoľvek, kam OneDrive alebo roamingový profil presunul priečinok Dokumenty
* **Predvolené (pre Linux

)**: `~/Documents/MAPIR/Chloros Projects`
* **Popis**: Zobrazuje aktuálny predvolený adresár, v ktorom sa vytvárajú nové projekty Chloros

. Kliknutím na ikonu úpravy vyberte iný adresár. Prepis sa ukladá ako jeden riadok textu v súbore `~/.chloros/working_directory.txt` — v systéme Windows

je to súbor `C:\Users\<Username>\.chloros\working_directory.txt`. Ak tento súbor chýba alebo uvádza cestu, ktorá už neexistuje, Chloros

sa vráti k vyššie uvedenému predvolenému nastaveniu. CLI

číta a zapisuje do toho istého súboru, takže `chloros-cli` a grafické rozhranie sa vždy zhodujú v tom, kde sa projekty nachádzajú.
* **Šablóny projektov** sa nachádzajú v podadresári `Project Templates/` tohto adresára.
* **Kedy zmeniť**:
  * Nastavte na sieťovú jednotku pre tímovú spoluprácu
  * Zmeňte na jednotku s väčším úložným priestorom pre veľké dátové súbory
  * Usporiadajte projekty podľa roka, klienta alebo typu projektu do rôznych zložiek
* **Poznámka**: Zmena tohto nastavenia ovplyvní len NOVÉ projekty. Existujúce projekty zostanú na svojich pôvodných miestach.
* **Dĺžka cesty v systéme Windows**: pokiaľ nie sú zapnuté dlhé cesty v systéme Windows

(možnosť **Povoliť dlhé cesty v systéme Windows** v inštalačnom programe), zložka projektu s dĺžkou cesty presahujúcou približne 130 znakov bude pred spracovaním odmietnutá, pretože jej výstupy by prekročili limit 260 znakov pre cestu v systéme Windows

. Pozrite si [Spustenie spracovania](../processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

***

## Ukladanie nastavení

Projekt Chloros

je **zložka**. Všetky nastavenia projektu sa ukladajú v súbore `project.json` v nej; pripojené zariadenia sa spolu s nimi ukladajú do súborov `cameras.json` a `sensors.json`, takže pri opätovnom otvorení projektu sa opäť pripoja aj jeho kamery a svetelné senzory. Keď projekt opäť otvoríte, všetky nastavenia sa obnovia presne tak, ako ste ich nechali. Uložené projekty je možné ovládať aj bez grafického rozhrania pomocou súboru `chloros-cli project` alebo súboru `open_project` z balíka SDK

.

### Hierarchia nastavení

Nastavenia sa uplatňujú v nasledujúcom poradí:

1. **Systémové predvolby** – Vstavané predvolby definované spoločnosťou Chloros

2. **Nastavenia šablóny** – Ak pri vytváraní projektu načítajte šablónu
3. **Nastavenia uloženého projektu** – Nastavenia uložené spolu so súborom projektu
4. **Ručné úpravy** – akékoľvek zmeny, ktoré vykonáte počas aktuálnej relácie

### Nastavenia a spracovanie obrazu

Nastavenia spracovania sa načítajú pri spustení spracovania. Zmena nastavenia nemá retroaktívny vplyv na výstupy, ktoré sú už na disku – na uplatnenie nových nastavení je potrebné spustiť spracovanie znovu. Niektoré nastavenia nemajú na spracovanie žiadny vplyv:

* Rozlíšenie miniatúr obrázkov (len na zobrazenie)
* Uložiť šablónu projektu
* Uložiť priečinok projektu

***

## Prehľad konfiguračných kľúčov

Pre automatizáciu (CLI

: `--config`, SDK

: `configure` alebo pri priamom čítaní `project.json`) sú toto presné kľúče v rámci `Project Settings`:

| Cesta ku kľúču | Typ | Predvolená hodnota |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | číslo 0–10 000 | `25` |
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
| `Processing → Minimum recalibration interval` | číslo 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | číslo -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | ID profilu obmedzenia alebo `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | zoznam konfigurácií indexov | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (8-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Kľúče `Array alignment` sa zapíšu pri prvom vykreslení sekcie „Array Alignment“ alebo keď ich nastaví volanie automatizácie. Ak chýbajú, potrubie používa rovnaké hodnoty ako je uvedené vyššie (`true`, `true`, bilineárne), takže projekt bez nich sa správa rovnako ako projekt s nimi.

### Kľúče uložené v `project.json` bez ovládacích prvkov v paneli nastavení

Tieto sa nachádzajú v rovnakom strome `Project Settings` a sú načítavané pri spracovaní, ale v bočnom paneli pre ne nenájdete žiadny ovládací prvok:

| Cesta kľúča | Typ | Predvolené | Nastavené |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK

`input_level=`. Prepisuje spôsob interpretácie vstupných súborov TIFF

v programe LATTICE; `auto` odvodzuje z XMP značky každého súboru `Chloros:ProcessingLevel` a počtu kanálov. Pri snímkach Survey3

`.raw` sa ignoruje. Zámerne nie je nastavením v grafickom rozhraní — automatické nastavenie je správne vo všetkých bežných prípadoch. |
| `Processing → Target reflectance dir` | reťazec cesty | `""` | `chloros-cli process --target-reflectance-dir`, alebo cieľové API

projektu |
| `Processing → Target reflectance config` | slovník s kľúčom podľa sériového čísla kamery | `{}` | Registrácia cieľa v rámci snímky (režim `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | reťazec cesty | `""` | SDK

`process_folder(daq_log_path=…)`. Odkazuje na záznam `.daq` alebo na zložku s takýmito záznamami |
| `Target Detection → Minimum calibration target squares` | číslo | `4` | Staré predvolené nastavenie; bez ovládacích prvkov a bez príznaku CLI

|
| `UI → Grid thumbnail size` | číslo | `160` | Vlastný posúvač zväčšenia miniatúr mriežky obrázkov |

Dve nastavenia prehliadača sú uložené **na najvyššej úrovni v `project.json`**, úplne mimo `Project Settings`, pretože ide o stav zobrazenia a nie o nastavenia spracovania:

| Kľúčová cesta | Typ | Predvolená hodnota | Nastavené |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | celé číslo 1–256 | `1` | Ovládací prvok GSD (px) na karte obrázku — pozri [Otvorenie obrázku na celú obrazovku](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Osvedčené postupy

1. **Začnite s predvolenými nastaveniami**: Predvolené nastavenia fungujú dobre pre väčšinu kamerových systémov MAPIR

a typických pracovných postupov.
2. **Vytvorte šablóny**: Akonáhle optimalizujete nastavenia pre konkrétny pracovný postup alebo kameru, uložte ich ako šablónu, aby ste zabezpečili konzistentnosť vo všetkých projektoch.
3. **Pred úplným spracovaním otestujte**: Pri experimentovaní s novými nastaveniami najprv otestujte na malej vzorke obrázkov, než spracujete celý súbor údajov.
4. **Zaznamenajte si svoje nastavenia**: Používajte popisné názvy šablón, ktoré uvádzajú kamerový systém, typ spracovania a zamýšľané použitie (napr. „Survey3

\_RGB\_NDVI\_Agriculture“).
5. **Výber formátu exportu**: Formát exportu vyberte na základe konečného použitia:
   * Vedecká analýza → TIFF (16-bitový alebo 32-bitový)
   * Spracovanie v GIS → TIFF

(16-bitový)
   * Rýchla vizualizácia → PNG

(8-bitový)
   * Zdieľanie na webe → JPG (8-bitový)

***

Ďalšie informácie o multispektrálnych indexoch v programe Chloros

nájdete na stránke [Vzorce multispektrálnych indexov](multispectral-index-formulas.md).

# Karta DAQ v programe Chloros

Karta DAQ — v bočnom paneli programu Chloros označená ako **Svetelné senzory** — slúži ako ovládací panel v reálnom čase pre [svetelné senzory DAQ-U, DAQ-M a DAQ-E](README.md): pripojte senzory cez ľubovoľný prenosový protokol, sledujte kalibrované spektrá v reálnom čase, vypočítavajte odrazivosť v reálnom čase z dvojice senzorov a zaznamenávajte súbory `.daq` priamo do vášho projektu.

Karta sa sprístupní hneď po dokončení spustenia backendu Chloros. Grafy na karte sú napájané službou DAQ Chloros prostredníctvom živého pripojenia, ktoré sa v prípade prerušenia automaticky opätovne pripojí (s odkladom 2–10 s); pokiaľ je služba nedostupná, v riadku Stav senzora sa zobrazuje **Žiadny server**.

Rozloženie pozostáva z **bočného panela senzorov**(jeden riadok na každý pripojený senzor) a**oblasti grafov** (jedna dlaždica grafu na každý senzor alebo skupinu).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Pripojenie senzora

Kliknite na **Pripojiť senzor** v hornej časti bočného panela. V hlavnej oblasti sa otvorí dialógové okno pripojenia (alebo ako prekrývacie okno pri pridávaní ďalšieho senzora – v takom prípade sa zobrazí tlačidlo Zrušiť).

| Ovládací prvok | Správanie |
| --- | --- |
| **Typ zariadenia** | `DAQ-U (USB)` (predvolené), `DAQ-M (Bluetooth)` alebo `DAQ-E (Ethernet)`. Zmena spustí opätovné vyhľadávanie pre novo vybraný prenosový protokol. |
| **Port / Zariadenie BLE / Názov hostiteľa / IP** | Zobrazuje zoznam nájdených zariadení ako `device - description`; automaticky sa vyberie prvý záznam rozpoznaný ako senzor. Počas vyhľadávania sa zobrazuje `Scanning...` (USB), `Scanning (N)...` s 8-sekundovým odpočítavaním (BLE) alebo `Discovering ethernet sensors (N)...` s 5-sekundovým odpočítavaním (Ethernet). Prázdne výsledky sa zobrazujú ako `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Obnoviť** | Okamžite opätovne skenuje vybraný prenosový protokol (vypnuté uprostred skenovania BLE/Ethernet). |
| **Pripojiť senzor** | Aktivuje sa po výbere zariadenia; počas vytvárania pripojenia sa označenie zmení na `Connecting...`. |

Vyhľadávanie prebieha iba **počas zobrazenia dialógového okna pripojenia** a opakuje sa každých 15 sekúnd iba pre vybraný prenosový kanál — samotné otvorenie karty nespustí vyhľadávanie. V prípade zlyhania sa v dialógovom okne zobrazí: *„Pripojenie sa nepodarilo. Skúste odpojiť a znovu pripojiť senzor, potom kliknite znovu na Pripojiť.“*

Bočný panel sa automaticky otvorí, keď sa pripojí váš prvý senzor.

{% hint style="info" %}
**DAQ-E sa nezobrazuje?**DAQ-E nemá stavovú LED — skontrolujte indikátor PoE/pripojenia na prepínači alebo porte injektora, do ktorého je zapojený, a po zapnutí počkajte niekoľko sekúnd, kým sa naštartuje. Zariadenie Chloros musí byť v tej istej vysielacej doméne (mDNS neprechádza cez smerovače). Pravidlo firewallu**Chloros Backend (UDP-In)** v inštalačnom programe povoľuje mDNS (UDP 5353), dáta z DAQ-E (UDP 5002) a PTP (UDP 319/320); v prípade firewallu tretej strany je potrebné povoliť rovnaké porty. Dve jednotky DAQ-E v jednej sieti LAN sa detekujú samostatne, každá pod vlastným názvom hostiteľa `daq-e-<id>.local`.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Typ zariadenia ponúka DAQ-U (USB), DAQ-M (Bluetooth) a DAQ-E (Ethernet)</figcaption></figure>

***

## Bočný panel senzorov

Každý pripojený senzor má jeden riadok (plus jeden riadok na skupinu „Okolie + Objekt“). Riadky je možné presúvať ťahaním a ich poradie zároveň mení poradie dlaždíc grafu. Kliknutím na riadok nastavíte daný senzor/skupinu ako aktívny graf v zobrazení zoznamu.

| Prvok | Význam |
| --- | --- |
| Farebný ľavý okraj | Farba grafu senzora. |
| Ikona prenosu | `DAQ-U` / `DAQ-M` / `DAQ-E` alebo zelený štítok `REF` pre skupinu odrazivosti Ambient+Object. |
| Názov zariadenia | Predvolene je to sériové číslo senzora (jeho stabilná identita pre kalibráciu, názvy súborov `.daq` a zhodu pri importe); vlastné názvy zostávajú zachované pre každý projekt. |
| Ikona **Kalibrované** (zelená) | Zobrazuje sa, keď je načítaný továrenský kalibračný balík senzora, t. j. spektrá sú v skutočných jednotkách W/m²/nm. |
| Ikona **K dispozícii aktualizácia** (jantárová, len DAQ-E) | Bežiaci firmvér je starší ako obraz dodávaný s touto verziou Chloros. Počas aktualizácie zobrazuje priebeh v reálnom čase (`Flashing… N%`, `Restarting sensor…`, potom `Updated X → Y` alebo `Failed`). |
| Oko | Prepína viditeľnosť tohto senzora na grafe. |
| Ozubené koliesko | Otvorí modálne okno nastavení pre jednotlivé senzory (nižšie). |
| ✕ (červené) | Odpojí senzor alebo odstráni skupinu „Ambient+Object“. |

Nad riadkami sa nachádzajú dve tlačidlá:

* **Pripojiť senzor** — otvorí dialógové okno pripojenia (počas činnosti sa premenuje na `Connecting...`).
* **Zaznamenať všetko / Zastaviť všetko**— spustí alebo zastaví záznam `.daq` na**všetkých**pripojených senzoroch. Vyžaduje aspoň jeden senzor**a otvorený projekt** (popis: „Otvorte projekt na nahrávanie“); počas prebiehajúceho nahrávania sa tlačidlo zafarbí na červeno.

V prázdnom stave sa zobrazuje text „Žiadne pripojené senzory“.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Nastavenia jednotlivých senzorov (modálne okno s ozubeným kolieskom)

Otvoríte kliknutím na ikonu ozubeného kolesa v riadku senzora. Obsah v poradí:

* **Informačné riadky** — Typ zariadenia (DAQ-U/M/E), Pripojenie (`Serial (USB)` / `Bluetooth` / `Ethernet`), port (COM port, adresa BLE alebo hostiteľ) a sériové pripojenie.
* **Kalibračná správa: Stiahnuť** — stiahne kalibračný certifikát tohto zariadenia s vysledovateľnosťou podľa NIST (PDF) a otvorí ho vo vašom prehliadači PDF. Dostupné po zadaní sériového čísla; certifikát sa uloží do vyrovnávacej pamäte pri prvom pripojení.
* **Názov zariadenia** — kliknutím na ceruzku môžete zmeniť názov; nastavenie pretrváva v rámci projektu.
* **Farba čiary grafu** — vzorník farieb; nastavenie pretrváva v rámci projektu.
* **Integračný čas (ms)**— posuvník + číslo,**1–500 ms**, predvolené nastavenie**32 ms**. Deaktivované, ak je funkcia AE zapnutá.
* **Priemer snímok**— posuvník + číslo,**1–50 snímok**, predvolené nastavenie**20**.
* **AE: ZAP/VYPN**— prepínač automatickej expozície;**predvolene ZAPNUTÉ** pri pripojení. Vypnutím môžete nastaviť čas integrácie ručne.
* **Zastaviť prenos / Spustiť prenos** — pozastavenie alebo obnovenie živého prenosu.
* **Nahrávať / Zastaviť nahrávanie** — nahrávanie `.daq` pre každý senzor (vyžaduje otvorený projekt).
* **Cap** — profil korekcie cap (ďalšia časť).
* **Riadky s informáciami v reálnom čase** — čas integrácie (ms), FPS, vzorky, nahrávanie (červené `REC` alebo `Off`) a stav (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Len DAQ-E: riadky sieť, firmvér a PTP

* **Názov hostiteľa / IP** — aktuálna adresa zariadenia.
* **Firmware** — aktuálna verzia firmvéru spolu s akčnou bunkou: tlačidlo **Aktualizovať na \<version\>** sa zobrazí, ak táto verzia Chloros obsahuje novší obraz firmvéru pre DAQ-E. Aktualizácia sa prenesie cez sieť za približne 30 sekúnd; senzor sa automaticky reštartuje a opätovne pripojí, pričom v prípade prerušenia prenosu zostane aktuálny firmvér nedotknutý. Priebeh sa zobrazuje v reálnom čase (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`) a v bunke sa pri aktuálnom stave zobrazuje hodnota `Up to date`.
* **Synchronizácia PTP** — aktuálny stav PTP (prepne sa na `unknown`). Firmware DAQ-E verzie 1.2.0 a vyššej sa zúčastňuje na štandarde IEEE 1588 PTPv2 ako hodiny výlučne v podriadenom režime; backend hostiteľa Chloros je PTP grandmaster a každá kamera DAQ-E a LATTICE v sieti LAN je jej podriadená v doméne 0, pričom časové značky sa udržujú s presnosťou približne 1 ms.
* **Sieť**— adresa, na ktorej zariadenie beží, a spôsob jej získania (DHCP, link-local alebo statická), ako aj uložené nastavenie, ktoré sa uplatní po reštarte.**Zmeniť…**otvorí formulár:**Automaticky (DHCP)**alebo**Statická IP**s IP adresou, maskou podsiete a voliteľnou bránou.**Použiť a reštartovať** to uloží, reštartuje senzor a opäť ho pripojí. Statická adresa mimo siete tohto počítača je odmietnutá, pretože Chloros by sa po reštarte nemohol pripojiť k senzoru; následne sa zobrazí okienko, v ktorom môžete túto voľbu napriek tomu povoliť pre počítač, ktorý bude presmerovaný do tejto siete. **Streamovať pri zapnutí**určuje, či senzor začne streamovať hneď po zapnutí, ešte predtým, ako sa pripojí akýkoľvek softvér.**Restartovať senzor** ho reštartuje a opätovne pripojí s aktuálnymi nastaveniami. Reštart sa odmietne, pokiaľ senzor zaznamenáva údaje. Novinka vo verzii 1.3.0.

V prípade skupiny Ambient+Object sa v modálnom okne s ozubeným kolieskom zobrazujú iba zdrojové senzory skupiny, názov zariadenia a farba čiary grafu.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->



### Výber krytu

Rozbaľovacie menu **Kryt** oznamuje aplikácii Chloros, ktorý fyzický kryt je nasadený na difúzor senzora, a na každé spektrum aplikuje korekčný profil tohto krytu nameraný vo výrobnom závode. Možnosti výberu závisia od modelu:

| Model | Možnosti krytu |
| --- | --- |
| DAQ-U | Žiadny (holý senzor), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (kosínusový korektor) |
| DAQ-M | Žiadny (holý senzor), Sunshine (kosínusový korektor) |
| DAQ-E | Žiadny (holý senzor), FOV 15°, FOV 45°, FOV 90°, Sunshine (kosínusový korektor) |

**Predvoleným nastavením pre každý model je režim Sunshine (kosínusový korektor)** — spoločnosť MAPIR dodáva každý DAQ s nainštalovaným krytom Sunshine, čo je štandardná konfigurácia pre vonkajšie použitie: 180° hemisférický výhľad s kosínusovou chybou ≤ ±4 % do 60° a ≤ ±4,5 % do 70° (neodporúča sa pri výške Slnka pod ~15°), s konštrukčne daným útlmom (~12×). Váš výber zostane zachovaný v projekte.

{% hint style="warning" %}
**Výber krytu sa musí zhodovať s fyzickým krytom.**Ani senzor, ani softvér nedokážu zistiť, ktorý kryt je nasadený. Tento výber ovplyvňuje jednak korekciu v reálnom čase, jednak záznam zapísaný do každého súboru `.daq` — vzhľadom na ~12× útlm krytu Sunshine neohlásená zmena krytky nesprávne koriguje spektrá približne o tento faktor. (Odstránenie a opätovná montáž tej istej krytky opakuje chybu na približne 1,5 %.) Zvoľte**Žiadny (holý senzor)** len vtedy, ak je krytka fyzicky odstránená; na zariadení DAQ-E sa pri voľbe „None“ stále uplatňuje továrenský geometrický profil pre jeho zapustený sklenený difúzor – nejde o nečinnosť – a holé zariadenie DAQ-E je laboratórnou konfiguráciou, nie podporovanou konfiguráciou pre použitie v teréne.
{% endhint %}

{% hint style="info" %}
Aktualizácia z predchádzajúcej príručky: prepínač „Sunshine Diffuser Installed“ na strane prehliadača z verzie 1.1.0 už neexistuje. Správa krytiek sa teraz riadi profilom krytky pre každý senzor, ktorý sa uplatňuje na strane servera.
{% endhint %}

***

## Oblasť grafu

V hornej lište, ktorá zostáva vždy viditeľná, sa nachádza **prepínač medzi zobrazením zoznamu a mriežky**a posuvník**zoomovania grafu** (veľkosť dlaždíc 200–2000 px). Zobrazenie sa automaticky prepne na mriežku, ak existuje viac ako jedna skupina grafov, a späť na zoznam, ak je ich jedna alebo menej. Režim zobrazenia a veľkosť grafu zostávajú zachované pre každý projekt.**Spektrálny graf** pre každý senzor zobrazuje:

* **Os X** — vlnová dĺžka (nm). Mriežka snímača je 340–1010 nm s rozstupom 5 nm (135 bodov), pre zobrazenie interpolovaná na 1 nm.
* **Os Y**— pre kalibrovaný snímač (zelená ikona**Kalibrovaný**),**Výkon (W/m²/nm)**— radiometricky kalibrované spektrálne ožiarenie vo všetkých troch jednotkách — s automatickou predponou SI (m/µ/n) zvolenou na základe vrcholu. Senzor, ktorého kalibračný balík nie je načítaný, zobrazuje svoje surové hodnoty ako**Intenzita (počty)**, bez škálovania. Ak graf prekrýva oba typy, výkon je na ľavej osi a počty na pravej osi.
* Dúhová spektrálna výplň pod jednou krivkou; viaceré senzory na jednom grafe sa prekrývajú ako farebné čiary s stlmenými výplňami.
* **Nabehnutie kurzora**— vertikálny kurzor s vlnovou dĺžkou a hodnotou na senzor (W/m²/nm pre kalibrovaný senzor, inak celkový počet impulzov);**ťahaním** môžete zväčšovať alebo zmenšovať (pri zväčšení sa zobrazí tlačidlo na zmenšenie).
* Tlačidlo **+** (len v zobrazení mriežky) na pridanie senzora do tohto grafu alebo vytvorenie skupiny (pozri nižšie).
* Názov zariadenia vycentrovaný v hornej časti a indikátor načítavania, kým sa nezobrazí prvý snímok.

**Nasytenie** nie je na samotnom grafe označené: nasýtený senzor zobrazuje červený stavový text `SATURATED` a červený riadok `Saturated: Yes` v tabuľke živých údajov. Na jeho odstránenie skráťte integračný čas alebo opäť aktivujte AE.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->



***

## Tabuľka živých údajov (zobrazenie zoznamu)

Pod grafom v zobrazení zoznamu, aktualizované každých 500 ms:

* **Všetky modely**: Vzorka farby svetla (sRGB z CIE XYZ), Nasytené (Áno/Nie), CIE 1931 X/Y/Z, Chromatickosť x/y, CIE u′/v′, CCT (K), CRI (Ra), dominantná vlnová dĺžka (nm), maximálna vlnová dĺžka (nm), čistota excitácie, Duv, CIE L\*/a\*/b\* a Munsell H/V/C.
* **Iba kalibrované senzory**(akýkoľvek z modelov DAQ-U / DAQ-M / DAQ-E po načítaní balíka továrenskej kalibrácie — rozpoznáte to podľa zeleného označenia**Kalibrované** v riadku senzora): Celkový výkon (W/m²), fotopický lux (lx), skotopický lux (lx), pomer S/P, PPFD plus PPFD Red /Green /Blue (µmol/m²/s) a opické ožiarenia — S-kónus, melanopické, rodopické, M-kónus, L-kónus (všetky v W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Skupiny odrazivosti (Okolie + Objekt)

Dva prepojené senzory je možné skombinovať do živého zobrazenia odrazivosti — bez použitia kamery:

1. V zobrazení mriežky kliknite na **+**na dlaždici grafu a vyberte**Kombinovať okolie + objekt**.
2. Vyberte senzor **Zdroj okolitého osvetlenia**a senzor**Skener objektu**(dva odlišné senzory), potom kliknite na**Vytvoriť**.

Chloros vypočíta R(λ) = objekt(λ) / okolité osvetlenie(λ) pre každú vlnovú dĺžku na základe dvoch živých dátových tokov (0, ak je okolité osvetlenie ≤ 0). Názov skupiny závisí od kalibračnej triedy senzorov:

* Oba senzory sú kalibrované (balík načítaný) → **„Zdanlivá odrazivosť“**.
* Ak je niektorý zo senzorov nekalibrovaný → **„Relatívna odrazivosť“**.

Skupina sa zobrazuje ako zelený riadok `REF` v bočnom paneli a vo vlastnom grafe (duhová výplň, hodnoty pri nabehnutí kurzora na 4 desatinné miesta, priblíženie ťahaním).

Ponuka **+**ponúka aj možnosť**Pridať nový senzor** s tromi umiestneniami: *Zlúčiť nový senzor* (pridať do tohto grafu), *Presunúť existujúci senzor sem* alebo *Zobraziť nový senzor* (vlastný graf).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->



### Tabuľka vegetačných indexov

V zobrazení zoznamu sa pod grafom skupiny odrazivosti nachádza tabuľka vegetačných indexov, vypočítaná na základe aktuálnej odrazivosti v strede pásiem **modrá 450 / zelená 550 / červená 670 / NIR 800 nm** (hodnoty s presnosťou na 4 desatinné miesta, `---`, ak nie je možné vypočítať; pre zobrazenie úplného názvu prejdite kurzorom nad názvom indexu):

* **Vždy zobrazené** (nezávislé od mierky, ľubovoľná kombinácia senzorov): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Iba ak sú oba senzory kalibrované na výkon** (načítané sú obe sady): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## Nahrávanie súborov `.daq`

* Nahrávanie vyžaduje **otvorený projekt** — inak sú deaktivované tlačidlá „Nahrávať všetko“ (bočný panel) aj tlačidlá „Nahrávať“ pre jednotlivé senzory.
* Súbory sa ukladajú do adresára **`<project folder>/light_sensor/`**; názvy súborov obsahujú ID senzora a časovú pečiatku a názov zariadenia sa ukladá spolu so záznamom.
* Keď sa nahrávanie zastaví (tlačidlo „Zastaviť“, „Zastaviť všetko“ alebo odpojenie uprostred nahrávania), dokončený súbor `.daq` sa **automaticky pridá do otvoreného projektu** — objaví sa v zozname súborov projektu bez potreby ručného pridávania a je pripravený slúžiť ako údaje o dopadajúcom žiarení pre [spracovanie odrazivosti](README.md).
* Počas nahrávania sa v živých riadkoch modálneho okna nastavení zobrazuje červený indikátor `REC`.

Pre kvantitatívne hodnoty ožiarenia je potrebné vypočítať priemer z údajov za minimálne 15 sekúnd — ide o charakteristiku prístroja, nie o chybu.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->



***

## Rozloženie viacerých senzorov a uchovávanie projektov

* Kombinujte viacero senzorov na jednom grafe (zdieľané osi), zachovajte samostatné grafy (automatické rozloženie mriežky), presúvajte senzory medzi grafmi, presúvajte a zaraďujte riadky/políčka a skryte jednotlivé senzory pomocou prepínača v podobe oka.
* Chloros si pre každý projekt uchováva: názvy zariadení, farby grafov, veľkosť grafu, režim zobrazenia a nastavenia každého senzora (integračný čas, priemerné hodnoty snímok, stav AE, výber krytu).
* **Pri opätovnom otvorení projektu sa senzory automaticky opätovne pripoja** podľa adresy – COM port pre DAQ-U, zariadenie BLE pre DAQ-M, názov hostiteľa mDNS pre DAQ-E (rozpoznaný aj v prípade zmeny IP adresy zariadenia) — a opätovne uplatní uložený profil krytu každého senzora, priemerné hodnoty snímok, stav AE a manuálny integračný čas.

***

## Párovanie kamery (DLS)

Nie je potrebné nič spárovať. Na rozdiel od pracovných postupov DLS pre drony, ktoré vopred priraďujú svetelný senzor ku kamere, Chloros priraďuje údaje DAQ k snímkam až neskôr: pri importe/spracovaní sa hodnoty `.daq` interpolujú na časovú pečiatku expozície každého záberu. Nahrávajte s akýmkoľvek pripojeným senzorom (`.daq` sa automaticky zaradí do projektu) a spracovanie odrazivosti nájde správne hodnoty podľa času — pozrite si [Svetelné senzory DAQ](README.md), kde sa dozviete, ako sa využívajú údaje o dopadajúcom svetle. V prípade **živých**snímok LATTICE namiesto toho vyberte senzor v roletovom menu**Light Sensor** na karte Cameras; každé zachytenie potom uloží nameranú hodnotu, ktorá sa použila, priradenú výlučne k kamerám daného zachytenia (pozri [Pracovné postupy s odrazivosťou](reflectance.md)).

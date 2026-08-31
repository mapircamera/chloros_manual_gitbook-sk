# Karta DAQ v Chloros

Karta DAQ — označená ako **Svetelné senzory** v bočnom paneli Chloros — slúži ako rozhranie na živé ovládanie [svetelných senzorov DAQ-U, DAQ-M a DAQ-E](README.md): pripojte senzory cez ľubovoľný prenosový protokol, sledujte kalibrované spektrá v reálnom čase, vypočítavajte odrazivosť v reálnom čase z dvojice senzorov a zaznamenávajte súbory `.daq` priamo do vášho projektu.

Karta sa sprístupní hneď po dokončení spustenia backendu Chloros. Grafy na tejto karte sú napájané službou DAQ systému Chloros prostredníctvom živého pripojenia, ktoré sa v prípade prerušenia automaticky obnoví (s odkladom 2–10 s); pokiaľ je služba nedostupná, v riadku Stav senzora sa zobrazuje **Žiadny server**.

Rozloženie tvorí **bočný panel senzorov**(jeden riadok na každý pripojený senzor) a**oblasť grafov** (jedna dlaždica grafu na senzor alebo skupinu).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Pripojenie senzora

Kliknite na **Pripojiť senzor** v hornej časti bočného panela. V hlavnej oblasti sa otvorí dialógové okno pripojenia (alebo ako prekrývacie okno pri pridávaní ďalšieho senzora – v takom prípade sa zobrazí tlačidlo Zrušiť).

| Ovládací prvok | Správanie |
| --- | --- |
| **Typ zariadenia** | `DAQ-U (USB)` (predvolené), `DAQ-M (Bluetooth)` alebo `DAQ-E (Ethernet)`. Zmenou typu sa spustí nové vyhľadávanie pre novo vybraný prenosový protokol. |
| **Port / Zariadenie BLE / Názov hostiteľa / IP** | Zobrazuje zoznam zistených zariadení ako `device - description`; automaticky sa vyberie prvý záznam rozpoznaný ako senzor. Počas vyhľadávania sa zobrazuje `Scanning...` (USB), `Scanning (N)...` s 8-sekundovým odpočítavaním (BLE) alebo `Discovering ethernet sensors (N)...` s 5-sekundovým odpočítavaním (Ethernet). Prázdne výsledky sa zobrazujú ako `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Obnoviť** | Okamžite opätovne skenuje vybraný transport (vypnuté uprostred skenovania BLE/Ethernet). |
| **Pripojiť** | Aktivuje sa po výbere zariadenia; počas nadväzovania pripojenia sa označenie zmení na `Connecting...`. |

Vyhľadávanie prebieha iba **počas zobrazenia dialógového okna pripojenia** a opakuje sa každých 15 sekúnd iba pre vybraný transport – samotné otvorenie karty nespustí vyhľadávanie. V prípade zlyhania sa v dialógovom okne zobrazí: *„Pripojenie zlyhalo. Skúste odpojiť a opäť pripojiť senzor, potom znovu kliknite na Pripojiť.“*

Bočný panel sa automaticky otvorí po pripojení prvého senzora.

{% hint style="info" %}
**DAQ-E sa nezobrazuje?** DAQ-E nemá stavovú LED — skontrolujte indikátor PoE/pripojenia na prepínači alebo porte injektora, do ktorého je zapojený, a po zapnutí počkajte niekoľko sekúnd, kým sa zariadenie naštartuje. Zariadenie Chloros musí byť v tej istej vysielacej doméne (mDNS neprechádza cez smerovače). Na zariadení Windows potvrďte výzvu firewallu Defender pri prvom pripojení zariadenia Chloros k jeho multicastovým socketom (mDNS UDP 5353, dáta DAQ-E UDP 5002, PTP UDP 319/320). Dve jednotky DAQ-E v jednej sieti LAN sa detekujú samostatne, každá pod vlastným názvom hostiteľa `daq-e-<id>.local`.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Typ zariadenia ponúka DAQ-U (USB), DAQ-M (Bluetooth) a DAQ-E (Ethernet)</figcaption></figure>***

## Bočný panel senzorov

Každý pripojený senzor má jeden riadok (plus jeden riadok na skupinu „Ambient+Object“). Riadky je možné presúvať ťahaním a ich poradie zároveň mení poradie dlaždíc v grafe. Kliknutím na riadok nastavíte daný senzor/skupinu ako aktívny graf v zobrazení zoznamu.

| Prvok | Význam |
| --- | --- |
| Farebný ľavý okraj | Farba grafu senzora. |
| Ikona prenosu | `DAQ-U` / `DAQ-M` / `DAQ-E` alebo zelený štítok `REF` pre skupinu odrazivosti „Okolie + objekt“. |
| Názov zariadenia | Predvolene je to sériové číslo senzora (jeho stabilná identita pre kalibráciu, názvy súborov `.daq` a zhodu pri importe); vlastné názvy sa zachovávajú pre každý projekt. |
| Ikona **Kalibrované** (zelená) | Zobrazuje sa, keď je načítaný továrenský kalibračný balík senzora, t. j. spektrá sú v skutočných jednotkách W/m²/nm. |
| Ikona **Dostupná aktualizácia** (jantárová, len DAQ-E) | Bežiaci firmware je starší ako obraz dodávaný s touto verziou Chloros. Počas aktualizácie zobrazuje priebeh v reálnom čase (`Flashing… N%`, `Restarting sensor…`, potom `Updated X → Y` alebo `Failed`). |
| Oko | Prepína viditeľnosť tohto senzora na grafe. |
| Ozubené koleso | Otvorí modálne okno nastavení pre jednotlivé senzory (nižšie). |
| ✕ (červené) | Odpojí senzor alebo odstráni skupinu Ambient+Object. |

Nad riadkami sa nachádzajú dve tlačidlá:

* **Pripojiť senzor** — otvorí dialógové okno na pripojenie (počas činnosti sa premenuje na `Connecting...`).
* **Zaznamenať všetko / Zastaviť všetko**— spustí alebo zastaví nahrávanie `.daq` na**každom**pripojenom senzore. Vyžaduje aspoň jeden senzor**a otvorený projekt** (popis: „Otvorte projekt na nahrávanie“); počas prebiehajúceho nahrávania sa tlačidlo zafarbí na červeno.

V prázdnom stave sa zobrazuje text „Žiadne pripojené senzory“.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Nastavenia jednotlivých senzorov (modálne okno s ozubeným kolieskom)

Otvoríte kliknutím na ikonu ozubeného kolesa v riadku senzora. Obsah v poradí:

* **Informačné riadky** — Typ zariadenia (DAQ-U/M/E), Pripojenie (`Serial (USB)` / `Bluetooth` / `Ethernet`), port (COM port, adresa BLE alebo hostiteľ) a sériové číslo.
* **Kalibračná správa: Stiahnuť** — načíta kalibračný certifikát tohto zariadenia s vysledovateľnosťou podľa NIST (PDF) a otvorí ho vo vašom prehliadači PDF. K dispozícii po zistení sériového čísla; certifikát sa uloží do vyrovnávacej pamäte pri prvom pripojení.
* **Názov zariadenia** — kliknutím na ceruzku môžete zmeniť názov; nastavenie pretrváva v rámci projektu.
* **Farba čiary grafu** — vzorka farby; nastavenie zostáva zachované pre každý projekt.
* **Integračný čas (ms)**— posuvník + číslo,**1–500 ms**, predvolené nastavenie**32 ms**. Deaktivované, keď je funkcia AE zapnutá.
* **Priemer snímok**— posuvník + číslo,**1–50 snímok**, predvolené nastavenie**20**.
* **AE: ZAPNUTÉ/VYPNUTÉ**— prepínač automatickej expozície;**predvolene ZAPNUTÉ** pri pripojení. Vypnite ho, ak chcete nastaviť čas integrácie ručne.
* **Zastaviť prenos / Spustiť prenos** — pozastaví alebo obnoví živý prenos.
* **Nahrávať / Zastaviť nahrávanie** — nahrávanie `.daq` pre každý senzor (vyžaduje otvorený projekt).
* **Cap** — profil korekcie cap (ďalšia časť).
* **Riadky s informáciami v reálnom čase** — integračný čas (ms), FPS, vzorky, nahrávanie (červené `REC` alebo `Off`) a stav (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Len DAQ-E: riadky siete, firmvéru a PTP

* **Názov hostiteľa / IP** — aktuálna adresa zariadenia.
* **Firmware** — aktuálna verzia firmvéru spolu s akčnou bunkou:<version\>

tlačidlo</version\>

**Aktualizovať na \<version\>** sa zobrazí</version\>

,<version\>

keď táto verzia Chloros obsahuje novší obraz firmvéru pre DAQ-E. Aktualizácia sa prenesie cez sieť za približne 30 sekúnd; senzor sa automaticky reštartuje a opätovne pripojí, pričom prerušený prenos ponechá aktuálny firmvér nedotknutý. Priebeh aktualizácie sa zobrazuje v reálnom čase (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`) a v príslušnom riadku sa zobrazuje hodnota `Up to date`, ak je aktualizácia aktuálna.
* **Synchronizácia PTP** — aktuálny stav PTP (prepne sa na `unknown`). Firmware DAQ-E verzie 1.2.0 a vyššej sa zúčastňuje na IEEE 1588 PTPv2 ako hodiny výlučne v podriadenom režime; backend hostiteľa s adresou Chloros je PTP grandmaster a všetky zariadenia DAQ-E a kamery LATTICE v sieti LAN sú jeho podriadenými zariadeniami v doméne 0, pričom časové značky sa udržujú s presnosťou približne 1 ms.

V prípade skupiny Ambient+Object sa v modálnom okne zariadenia zobrazujú iba zdrojové senzory skupiny, názov zariadenia a farba grafu.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### Výber krytu

Rozbaľovacie menu **Cap** oznamuje systému Chloros, ktorý fyzický kryt je nasadený na difúzor senzora, a na každé spektrum aplikuje korekčný profil tohto krytu nameraný vo výrobe. Možnosti závisia od modelu:

| Model | Možnosti krytu |
| --- | --- |
| DAQ-U | Žiadny (holý senzor), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (kosínusový korektor) |
| DAQ-M | Žiadny (holý senzor), Sunshine (kosínusový korektor) |
| DAQ-E | Žiadny (holý senzor), FOV 15°, FOV 45°, FOV 90°, Sunshine (kosínusový korektor) |

**Predvoleným nastavením pre každý model je Sunshine (kosínusový korektor)** — Spoločnosť MAPIR dodáva každý DAQ s nainštalovaným krytom Sunshine, čo je štandardná konfigurácia pre vonkajšie použitie: hemisférický výhľad 180° s kosínusovou chybou ≤ ±4 % do 60° a ≤ ±4,5 % do 70° (neodporúča sa pri výške Slnka pod ~15°), konštrukčne tlmený (~12×). Váš výber zostane zachovaný v projekte.

{% hint style="warning" %}
**Výber krytu sa musí zhodovať s fyzickým krytom.**Ani senzor, ani softvér nedokážu zistiť, ktorý kryt je nasadený. Tento výber ovplyvňuje nielen korekciu v reálnom čase, ale aj údaj zapísaný do každého súboru `.daq` — vzhľadom na približne 12-násobné zoslabenie krytu Sunshine môže neoznámená výmena krytu spôsobiť nesprávnu korekciu spektier približne o tento faktor. (Odstránenie a opätovná montáž toho istého krytu spôsobuje odchýlku približne 1,5 %.) Možnosť**None (holý senzor)** vyberte len vtedy, ak je kryt fyzicky odstránený; na zariadení DAQ-E sa pri voľbe „None“ stále uplatňuje výrobný geometrický profil pre jeho zapustený sklenený difúzor – nejde o nečinnosť – a holé zariadenie DAQ-E predstavuje laboratórnu konfiguráciu, nie podporovanú konfiguráciu v teréne.
{% endhint %}

{% hint style="info" %}
Aktualizácia z predchádzajúceho manuálu: prepínač „Sunshine Diffuser Installed“ na strane prehliadača z verzie 1.1.0 už neexistuje. Správa krytiek sa teraz riadi profilom krytiek pre každý senzor, ktorý sa aplikuje na strane servera.
{% endhint %}

***

## Oblasť grafov

Na hornom lište, ktorá zostáva vždy viditeľná, sa nachádza **prepínač medzi zobrazením zoznamu a mriežky**a posuvník**Zväčšenie grafu** (veľkosť dlaždice 200–2000 px). Zobrazenie sa automaticky prepne do mriežky, ak existuje viac ako jedna skupina grafov, a späť do zoznamu, ak je ich jedna alebo menej. Režim zobrazenia a veľkosť grafu sa uchovávajú pre každý projekt.**Spektrálny graf** pre každý senzor zobrazuje:

* **Os X** — Vlnová dĺžka (nm). Mriežka snímača je 340–1010 nm s rozstupom 5 nm (135 bodov), pre zobrazenie interpolovaná na 1 nm.
* **Os Y** — výkon (W/m²), s automatickou predponou SI (m/µ/n) vybranou na základe vrcholu. Spektrá sú radiometricky kalibrované spektrálne ožiarenie (W/m²/nm) vo všetkých troch prenosoch.
* Dúhová spektrálna výplň pod jednou krivkou; viaceré senzory na jednom grafe sa prekrývajú ako farebné čiary so stlmenou výplňou.
* **Nabehnutie kurzorom**— vertikálny kurzor s vlnovou dĺžkou a hodnotou pre každý senzor;**ťahaním** môžete zväčšovať alebo zmenšovať (počas zväčšenia sa zobrazí tlačidlo na zmenšenie).
* Tlačidlo **+** (len v mriežkovom zobrazení) na pridanie senzora do tohto grafu alebo vytvorenie skupiny (nižšie).
* Názov zariadenia je vycentrovaný v hornej časti a do príchodu prvého snímku sa zobrazuje otočné koliesko.

**Nasytenie** nie je označené priamo na grafe: nasýtený senzor zobrazuje červený stavový text `SATURATED` a červený riadok `Saturated: Yes` v tabuľke živých údajov. Na odstránutie tohto stavu skráťte integračný čas alebo opäť zapnite automatickú expozíciu (AE).

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## Tabuľka živých údajov (zobrazenie zoznamu)

Pod grafom v zobrazení zoznamu, aktualizované každých 500 ms:

* **Všetky modely**: Vzorka farby svetla (sRGB z CIE XYZ), Nasytené (Áno/Nie), CIE 1931 X/Y/Z, Chromatickosť x/y, CIE u′/v′, CCT (K), CRI (Ra), Dominantná vlnová dĺžka (nm), vrcholová vlnová dĺžka (nm), čistota excitácie, Duv, CIE L\*/a\*/b\* a Munsell H/V/C.
* **Iba kalibrované senzory**(akýkoľvek z modelov DAQ-U / DAQ-M / DAQ-E po nainštalovaní balíka továrenskej kalibrácie — rozpoznáte to podľa zeleného označenia**Kalibrované** v riadku senzora): Celkový výkon (W/m²), fotopický lux (lx), skotopický lux (lx), pomer S/P, PPFD a PPFD Red/Green/Blue (µmol/m²/s), a opické ožiarenia — S-čapík, melanopické, rodopické, M-čapík, L-čapík (všetky v W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Skupiny odrazivosti (okolie + objekt)

Dva prepojené senzory je možné zlúčiť do živého zobrazenia odrazivosti — bez použitia kamery:

1. V mriežkovom zobrazení kliknite na **+**na dlaždici grafu a vyberte**Zlúčiť okolie + objekt**.
2. Vyberte senzor **Zdroj okolitého osvetlenia**a senzor**Skener objektu**(dva odlišné senzory), potom kliknite na**Vytvoriť**.

Chloros vypočíta R(λ) = objekt(λ) / okolité(λ) pre každú vlnovú dĺžku z dvoch živých prúdov (0, ak okolité ≤ 0). Názov skupiny sa riadi kalibračnou triedou senzorov:

* Oba senzory sú kalibrované (balík je načítaný) → **„Zdanlivá odrazivosť“**.
* Ak nie je kalibrovaný ani jeden senzor → **„Relatívna odrazivosť“**.

Skupina sa zobrazuje ako zelený riadok `REF` v bočnom paneli a vo vlastnom grafe (duhová výplň, hodnoty po prechode kurzorom na 4 desatinné miesta, priblíženie ťahaním).

Ponuka **+**ponúka aj možnosť**Pridať nový senzor** s tromi umiestneniami: *Zlúčiť nový senzor* (pridať do tohto grafu), *Presunúť existujúci senzor sem* alebo *Zobraziť nový senzor* (vlastný graf).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### Tabuľka vegetačných indexov

V zobrazení zoznamu sa pod grafom skupiny odrazivosti nachádza tabuľka vegetačných indexov, vypočítaná z aktuálnej odrazivosti v strede pásiem **modrá 450 / zelená 550 / červená 670 / NIR 800 nm** (hodnoty na 4 desatinné miesta, `---`, ak nie je možné vypočítať; pre zobrazenie úplného názvu prejdite kurzorom nad názov indexu):

* **Vždy zobrazené** (nezávislé od mierky, akákoľvek kombinácia senzorov): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Iba ak sú oba senzory kalibrované podľa výkonu** (oba balíky načítané): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## Nahrávanie súborov `.daq`

* Na nahrávanie je potrebný **otvorený projekt** — v opačnom prípade sú deaktivované funkcie „Nahrávať všetko“ (bočný panel) aj tlačidlo „Nahrávať“ pri jednotlivých senzoroch.
* Súbory sa ukladajú do adresára **`<project folder>/light_sensor/`**; názvy súborov obsahujú ID senzora a časovú pečiatku a názov zariadenia sa ukladá spolu so záznamom.
* Keď sa nahrávanie zastaví (tlačidlo „Stop“, „Stop All“ alebo odpojenie uprostred nahrávania), dokončený súbor `.daq` sa **automaticky pridá do otvoreného projektu** — objaví sa v zozname súborov projektu bez potreby ručného pridávania a je pripravený slúžiť ako údaje o dopadajúcom žiarení pre [spracovanie odrazivosti](README.md).
* Počas nahrávania sa v živých riadkoch modálneho okna nastavení zobrazuje červený indikátor `REC`.

Pre kvantitatívne hodnoty ožiarenia je potrebné vypočítať priemer z údajov za minimálne 15 sekúnd — ide o charakteristiku prístroja, nie o chybu.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## Rozloženie viacerých senzorov a uchovávanie projektov

* Kombinujte viacero senzorov na jednom grafe (zdieľané osi), zachovajte samostatné grafy (automatické rozloženie mriežky), presúvajte senzory medzi grafmi, ťahaním menite poradie riadkov/polí a skryte jednotlivé senzory pomocou prepínača v podobe oka.
* V rámci každého projektu sa pre projekt Chloros zachovávajú: názvy zariadení, farby grafov, veľkosť grafu, režim zobrazenia a nastavenia každého senzora (integračný čas, priemerné hodnoty rámcov, stav AE, výber obmedzenia).
* **Pri opätovnom otvorení projektu sa senzory automaticky opätovne pripoja** podľa adresy — COM port pre DAQ-U, zariadenie BLE pre DAQ-M, názov hostiteľa mDNS pre DAQ-E (rozpozná sa aj v prípade zmeny IP adresy zariadenia) — a opätovne sa uplatnia uložené nastavenia každého senzora: profil krytu, priemerné hodnoty snímok, stav AE a manuálny integračný čas.***

## Párovanie kamery (DLS)

Nie je potrebné nič párovať. Na rozdiel od pracovných postupov DLS s dronmi, ktoré vopred priraďujú svetelný senzor ku kamere, Chloros spája údaje z DAQ so snímkami až neskôr: v čase importu/spracovania sa hodnoty z `.daq` interpolujú na časovú pečiatku expozície každého záberu. Nahrávajte s akýmkoľvek pripojeným senzorom (`.daq` sa automaticky zaradí do projektu) a spracovanie odrazivosti nájde správne namerané hodnoty podľa času – pozrite si [Svetelné senzory DAQ](README.md), kde sa dozviete, ako sa využívajú údaje o dopadajúcom svetle.</version\>

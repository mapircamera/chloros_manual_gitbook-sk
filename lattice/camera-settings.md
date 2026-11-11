# Nastavenia kamier

Karta **Kamery** je ovládací panel aplikácie Chloros

pre kamery LATTICE: hlavná oblasť so živým obrazom, kde sa každá pripojená kamera zobrazuje ako živá dlaždica, a bočný panel, ktorý sa prepína medzi tromi stránkami — **zoznamom kamier**,**paneli nastavení**(nastavenia pre jednotlivé kamery, pole kamier alebo nastavenia snímania – vždy jedno po druhom) a**kalkulátor indexu**. Táto stránka popisuje všetky ovládacie prvky v zozname kamier, v paneli nastavení pre jednotlivé kamery a v paneli nastavení poľa kamier. Režimy snímania, výber typu exportu a postup „Zachytiť všetko“ sa nachádzajú na sprievodnej stránke [Nastavenia a režimy snímania](capture.md).

Karta „Kamery“ sa zobrazí v bočnom paneli, akonáhle je backend Chloros

pripravený. Všetky nižšie uvedené ovládacie prvky komunikujú s lokálnym backendom cez `127.0.0.1:5000`; zmeny sa okamžite prejavia v živom obraze kamery, pokiaľ nie je uvedené inak.

## Typy kamier používané na tejto stránke

Ovládacie prvky sa zobrazujú alebo skryjú v závislosti od toho, aký typ kamery je vybraný. V tejto príručke sa používajú nasledujúce pojmy:

| Pojem | Význam | Kanály filtra |
| --- | --- | --- |
| **RGB

kamera** | LATTICE M3C s filtrom FRGB (model obsahuje `-FRGB`) | Red

/ Green

/ Blue

|
| **Multispektrálna kamera typu Bayer** | LATTICE M3C s filtrom FRGN, FOCN alebo FNGB | FRGN: Red

/ Green

/ NIR

· FOCN: Orange

/ Cyan

/ NIR

· FNGB: NIR

/ Green

/ Blue

|
| **Mono (M3M)** | LATTICE M3M — jeden úzkopásmový filter, jedno kalibrované pásmo | Jedno pásmo |
| **Člen poľa** | Kamera pripojená ako súčasť synchronizovaného poľa (kombinované alebo samostatné zobrazenie) | Podľa svojho filtra |

Kamery RGB

prechádzajú fotometrickým spracovaním (vyváženie bielej, farebné profily, gama); multispektrálne a mono kamery prechádzajú radiometrickým reťazcom a preskakujú fotometrické nastavenia. Členovia poľa odovzdávajú nastavenia na úrovni streamu (formát pixelov, rozlíšenie, binning, spúšť, snímková frekvencia) do poľa — tieto riadky sa v paneli pre jednotlivé kamery stanú iba na čítanie a presunú sa do panela nastavení poľa.

## Hlavná oblasť prenosu



<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

Ak nie sú pripojené žiadne kamery, v oblasti prenosu sa zobrazí úvodná obrazovka **„Pripojte kameru a začnite“**s dvoma tlačidlami:**Pripojiť kameru**(zelené, otvorí dialógové okno na pripojenie jednej kamery) a**Pripojiť pole** (modré, otvorí dialógové okno na pripojenie poľa). Samotné dialógové okná na pripojenie sú zdokumentované v [Pripájanie kamier](connecting.md); pojmy týkajúce sa poľa (synchronizácia, vrstvy, šírka pásma) v [Súbory viacerých kamier](arrays.md). Keď otvoríte uložený projekt, ktorý obsahuje kamery, na úvodnej obrazovke sa namiesto toho zobrazí ikona načítavania s textom „Znovu otváranie N uložených kamier…“, kým Chloros

obnovuje prúdy z poslednej relácie.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Horná lišta

| Ovládací prvok | Funkcia |
| --- | --- |
| **Prepínanie režimu zobrazenia**| Prepína medzi**mriežkovým zobrazením**(všetky dlaždice ako bunky) a**zoznamovým zobrazením**(riadky v plnej šírke: každá kombinovaná zostava ako jeden zložený riadok, každý člen**samostatnej** zostavy ako vlastný riadok s vlastným prúdom a tabuľkou kalibračných cieľov, potom JEDNA samostatná kamera – tá aktívna – pod tým). Nápovedy: „Prepnúť na zobrazenie mriežky“ / „Prepnúť na zobrazenie zoznamu“. |
| **Zámok mriežky**(visací zámok) | Predvolene**uzamknuté** — dlaždice sú zafixované na mieste. Odomknite, aby ste mohli dlaždice presúvať a zaraďovať do ľubovoľných pozícií (medzery zostanú zachované). Mriežka sa automaticky opäť uzamkne zakaždým, keď sa pripojí nová kamera. Nápovedy: „Odomknúť mriežku (povoliť presúvanie dlaždíc)“ / „Zamknúť mriežku (zafixovať dlaždice na mieste)“. |
| Posuvník **Zväčšenie prúdu** | Veľkosť dlaždice, od 60 px až po celú šírku kontajnera. Bunky si zachovávajú pomer strán 4:3. Pri šírke bunky menšej ako 200 px sa prekrytia s názvom a fps skryjú, aby dlaždica zostala prehľadná. |

### Dlaždice živého prenosu

Každá kamera vykresľuje kompozitnú živú dlaždicu; kamera môže navyše zobrazovať tri dlaždice v odtieňoch šedej s **rozdelením podľa kanálov** (pozri [Rozdelenie kanálov](#display-overlays-drawn-over-the-live-feed)), pričom pole kamier vykresľujú kombinovanú dlaždicu. Aktívna dlaždica je označená výberovým prstencom v farbe kamery (alebo poľa kamier).

Pri nabehnutí kurzora na dlaždicu sa zobrazí tlačidlo na zatvorenie **X**:

* Zatvorenie **zloženej** dlaždice, pričom jej rozdelenia kanálov zostanú aktívne, iba skryje zloženú dlaždicu.
* Zatvorenie **poslednej viditeľnej dlaždice samostatnej kamery** odpojí danú kameru.
* **Rozdelené dlaždice členov kombinovaného poľa nikdy neodpoja** kameru — iba ju skryjú.

Ak je mriežka odomknutá, pretiahnite ľubovoľnú dlaždicu do ľubovoľného slotu; rozloženie sa uloží spolu s projektom.

## Bočný panel — zoznam

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

kamier Prvá stránka bočného panela obsahuje zoznam všetkých pripojených kamier a sústav:

* **Pripojiť kameru**(zelené) /**Pripojiť pole** (modré, počas skenovania sa zobrazuje „Detekujem...“). Obe možnosti sú deaktivované, pokiaľ je otvorené dialógové okno pripojenia.
* **Zachytiť všetko** (červené) — zachytí všetky uvedené kamery s typmi exportu zvolenými v Nastaveniach zachytávania. Vyžaduje otvorený projekt. Úplná dokumentácia je k dispozícii v [Nastaveniach a režimoch snímania](capture.md).
* **Ozubené koliesko nastavení snímania** (vedľa tlačidla Zachytiť všetko) — otvorí [panel nastavení snímania](capture.md#the-capture-settings-pane). Je deaktivované bez projektu alebo počas snímania.

### Riadky kamier

Každý riadok kamery zobrazuje farebne označený okraj (vlastná farba kamery), označenie „CAM“ — s modrým písmenom **M**(hlavná) alebo zelené**S** (podriadená) označenie úlohy pre členov poľa — a zobrazený názov. Predvolený názov je `LATTICE-MODEL (serial)`; premenujte ho v paneli nastavení jednotlivých kamier. Tlačidlá v riadku:

| Tlačidlo | Účinok |
| --- | --- |
| **Oko**| Prepína viditeľnosť. Skryté kamery zmiznú z mriežky a sú**vylúčené z funkcie Zachytiť všetko**. |
| **Ozubené koleso** | Otvorí okno nastavení pre jednotlivé kamery (ďalšia časť). |
| **Pozastaviť / Prehrať**| Zastaví živý náhľad**len na strane zobrazenia** — záznam na serveri pokračuje. Pozastavené kamery nemôžu nahrávať. |
| **X** | Odpojiť. Používateľské rozhranie sa aktualizuje okamžite (v ideálnom prípade); samotné odpojenie na pozadí môže trvať 10–30 s. |

### Riadky poľa

Riadok poľa zobrazuje označenie „ARRAY“ v farbe poľa, názov poľa (premenovateľný v nastaveniach poľa) a tlačidlo **DAQ · zapnuté/vypnuté**—**zapnuté**, ak je nastavený svetelný senzor na úrovni poľa *alebo* akýkoľvek člen poľa má senzor pre jednotlivé kamery; v jeho popise sa uvádza, ktorý senzor čo sníma. Kamery v poli sú uvedené pod ním so zarážkou vo vlastných riadkoch. Tlačidlá v riadku poľa: **oko**(skryje/zobrazí VŠETKY členy naraz),**ozubené koleso**(panel nastavení poľa),**X**(odpojí celé pole).

Stav svetelného senzora (DLS) používaný v riadkoch poľa a v paneli nastavení poľa má štyri stavy:**vypnutý**,**čaká**(zatiaľ žiadne spektrum),**aktívne**(spektrum dorazilo v priebehu posledných 3 s) a**neaktuálne** — žiadne nové spektrum za posledné 3 s, ale posledná hodnota sa *stále používa* (hodnoty DAQ nikdy nevypršia v ceste zachytávania).

Kým nezmeníte poradie, zoznam zobrazuje najprv polia, potom samostatné kamery, pričom kamery (a členovia každého poľa) sú zoradené v **poradí kamier**: M3C (Bayer) pred M3M (mono); medzi M3C, FRGB, FRGN, FOCN, potom FNGB; M3M podľa vlnovej dĺžky filtra, najnižšia ako prvá. Predvolené rozloženie mriežky dodržiava rovnaké poradie. Samostatné kamery a celé skupiny polí môžete v bočnom paneli presúvať navzájom, aby ste zmenili poradie v zozname; členy poľa nie je možné presúvať samostatne.

## Panel nastavení pre jednotlivé kamery

Otvoríte ho kliknutím na **ozubené koliesko** v riadku kamery. Panel sa posunie nad zoznam kamier.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Záhlavie**:**farebná vzorka**kamery (kliknutím otvoríte natívny výber farieb — nastavuje farbu ohraničenia bočného panela a výberového kruhu dlaždíc),**názov**s tlačidlom**Pomenovať**v podobe ceruzky (uloženie prázdneho názvu vráti predvolený názov `MODEL (serial)`) a**×** na zatvorenie.

### Živý histogram

V hornej časti panela sa nachádza živý histogram jasu vypočítaný z náhľadu vo formáte JPEG

pri frekvencii ~8 Hz. Priemer je vážený podľa Bayerovho algoritmu — (R+2G+B)/4 — aby zodpovedal vlastnému meraniu automatického nastavenia expozície (AE) kamery.

* **Oranžová prerušovaná čiara** = cieľ automatického nastavenia expozície (Orange

). **Presuňte ju vodorovne na zmenu cieľa** — pri uvoľnení sa odošle jeden príkaz a ťahaním sa režim cieľa automatického nastavenia expozície prepne na manuálny.
* **Zelená súvislá čiara** = skutočná priemerná luma (to, čo automatické nastavenie expozície aktuálne poskytuje).
* **Tlačidlo RGB** (vpravo hore): prepína prekrývacie histogramy jednotlivých pásiem sfarbené podľa filtra fotoaparátu (napr. pri FRGN: šedá NIR

, zelená, červená). Na monochromatických (M3M) fotoaparátoch je na tlačidle nápis „MONO“ a je deaktivované — monochromatický režim vždy zobrazuje jednopásmový histogram jasu.
* Popisky na osi X zodpovedajú bitovej hĺbke senzora aktuálneho formátu pixelov: 0..255, 0..1023, 0..4095 alebo 0..65535.

### Riadky

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

s informáciami o kamere

| Riadok | Správanie |
| --- | --- |
| **Model** | Iba na čítanie (napr. `LATT-M3C-L87-FRGN`). |
| **Radiometrická kalibrácia** | Green

á **„Aktívna“**ikona s popisom zobrazujúcim úroveň kalibrácie, hash, dátum kalibrácie a zoznam pásiem, načítanými z kalibračného balíka kamery (pozri [Továrenská radiometrická kalibrácia](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Skryté pre kamery typu RGB

** — disponujú fotometrickou kalibráciou vyváženia bielej, nie radianciou pre jednotlivé pásma. |
| **Správa o kalibrácii**| Tlačidlo**Stiahnuť** — otvorí PDF certifikát NIST o kalibrácii kamery podľa sériového čísla vo prehliadači vášho operačného systému. Ak certifikát ešte nie je uložený v cache, Chloros

namiesto toho zobrazí upozornenie. |
| **Sériové číslo** | Iba na čítanie. |
| **Firmware**| Zobrazuje aktuálnu verziu, potom vyhľadá dostupnú verziu pre tento model (uloženú v cache podľa modelu — skupina N kamier skontroluje server len raz). Stavy: „Kontroluje sa…“ → tlačidlo**„Aktualizovať na X“**→ „Aktualizuje sa…“ → „Aktualizované z A na B“ / „Zlyhalo: …“ / „Preskočené: …“ / zelené**„Aktuálne“**. Popis tlačidla aktualizácie: „Obnovenie továrenských nastavení + aktualizácia firmvéru + preprogramovanie UserSet1. ~2–3 minúty; neodpájajte.“ |
| **IP** | Iba na čítanie. |
| **Teplota** | Iba na čítanie, aktualizuje sa každé 3 s. Pri ≥65 °C svieti oranžovo a pri ≥75 °C červeno so symbolom ⚠. |
| **Kalibračný cieľ** zaškrtávacie políčko | Aktivuje detekciu cieľa odrazivosti ArUco s validačnou tabuľkou NDVI

pre každý panel pod živým prenosom (zobrazenie zoznamu) — pozri [Tabuľka kalibračného cieľa](#the-calibration-target-table). Platí len pre danú reláciu — vždy sa otvára v vypnutom stave. |
| Rozbaľovacie menu **Svetelný senzor** | Priradí svetelný senzor DAQ (DAQ-E/M/U, zo zoznamu na karte Svetelné senzory) k tejto kamere na účely korekcie osvetlenia smerom nadol (DLS) a prediktívnej automatickej expozície. „Voľba „Žiadny“ zruší priradenie. Ak nie sú pripojené žiadne senzory, v roletovom menu sa zobrazí „(žiadne senzory nie sú pripojené — otvorte kartu DAQ)“. Priradenie sa uloží spolu s projektom. |

#### Tabuľka kalibračných terčov

Táto tabuľka slúži **len na overenie** („len na overenie — nemení kalibráciu“). V jej hlavičke sú uvedené sériové číslo kamery, filter, číslo terča a veľkosť značky v pixeloch, pričom každý panel má jeden riadok:

* **Panel**, potom jeden stĺpec na každé pásmo s hodnotou panelu — kamera typu Bayer zobrazuje svoje tri pásma podľa názvu a vlnovej dĺžky (napr. FRGN: Red

660 nm / Green

550 nm / NIR

850 nm); **monochromatická (M3M) kamera zobrazuje svoje jediné pásmo** (napr. `F750`) a nemá stĺpce NDVI

.
* **Sat** — podiel pixelov panelu, ktoré sú nasýtené (jantárová farba od 1 %, červená od 5 %).
* **NDVI

Meas**,**NDVI

Theo** (referenčná odrazivosť NDVI

panelu) a **Δ**— iba multispektrálne kamery typu Bayer. Malá značka za „NDVI

Meas“ udáva, z čoho bol nameraný NDVI

vypočítaný:**ρ**(odrazivosť — je k dispozícii hodnota zo svetelného senzora),**L**(svetelná intenzita — žiadna hodnota zo svetelného senzora) alebo**DN**(surové hodnoty kamery). Iba**ρ** NDVI

je porovnateľné s NDVI

Theo: pre L a DN je stĺpec Δ sivý a poznámka to uvádza („Chýba údaj zo svetelného senzora: NDVI

je vypočítané zo žiarivosti, preto Δ neslúži na kontrolu kalibrácie“ / „NDVI

je vypočítané zo surových hodnôt kamery, preto Δ neslúži na kontrolu kalibrácie“). Pri hodnote ρ je Δ zelené pri hodnote pod 0,02, oranžové pri hodnote pod 0,05 a v ostatných prípadoch červené.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Expozícia a zosilnenie

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Všetky číselné vstupy tu používajú otočné ovládače s funkciou „podržať pre zrýchlenie“: ťuknutie = ±1, podržanie &gt;1,5 s = ±10, podržanie &gt;3 s = ±100. Hodnota sa odošle do kamery po uvoľnení.

| Ovládací prvok | Rozsah / možnosti | Predvolené | Platí pre | Čo robí |
| --- | --- | --- | --- | --- |
| **Expozícia (us)**| Min./max. v reálnom čase kamery | Auto | Všetko | Doba expozície v mikrosekundách, s prepínačom**Auto/Manuálne**prepínačom. Auto = nepretržitá automatická expozícia na strane kamery. Nastavením manuálnej expozície zostane**Zosilnenie** v režime Auto, ak v ňom bolo, a je možné využívať celý rozsah zosilnenia kamery, kým nezmeníte zosilnenie alebo ho nenastavíte na Manuál. |
| **Zosilnenie (dB)**| Aktuálne min./max. hodnoty kamery (napr. až 48 dB) | Manuálne (vypnuté) | Všetko | Analógové/digitálne zosilnenie s vlastným prepínačom**Auto/Manual**. |
| **Cieľová jasnosť AE**| 0–255 | 80, režim**Auto**| Všetko (upraviteľné, ak je zapnutá AE alebo automatické zosilnenie) | Jasnosť, na ktorú sa zameriava AE. V režime**Auto**(predvolené nastavenie) si cieľovú hodnotu vyberá riadiaci modul na základe histogramu, pričom expozícia sa udržiava na úrovni 60–75 % maximálnej hodnoty snímača. Zadanie hodnoty alebo pretiahnutie oranžovej čiary na histograme prepne režim na**Ručný**. |
| **Vyhladenie AE** | 0,5–40, krok 0,1 | 8,0 | Všetko | Tlmenie AE. Popis: „Nižšia hodnota = AE reaguje rýchlejšie (môže pulzovať pri vysokej snímkovej frekvencii). Vyššia hodnota = plynulejšie / pomalšie.“ Hodnoty výrazne nižšie ako predvolené môžu spôsobiť pulzovanie AE a destabilizovať streamovanie pri vysokej snímkovej frekvencii; 8,0 je stabilná predvolená hodnota. |
| **Oblasť záujmu AE**| Zaškrtávacie políčko „Zapnúť“ + tlačidlo**Zameranie**| Vypnuté | Všetko | Ak je táto funkcia zapnutá, AE meria iba oblasť označenú zelenou prerušovanou čiarou namiesto celého snímku.**Aim** aktivuje funkciu „kliknutie na umiestnenie“ v živom prenose: kliknutím vycentrujete oblasť na 30 % snímky; kliknutím a ťahaním vytvoríte vlastný obdĺžnik (minimálne 5 % × 5 %). Funkcia „Aim“ sa po jednom umiestnení automaticky deaktivuje. Oblasť sa premietne späť do natívnych súradníc kamery podľa akejkoľvek rotácie/zrkadlenia, ktoré ste nastavili, a uloží sa spolu s projektom. |
| **Rýchlosť ladenia AE** | 0,1–5, krok 0,1 | 1,0 | Len pre členov skupiny | Ako rýchlo cieľ automatického AE sleduje zmeny jasu scény; pri hodnote 1,0× sa prehodnocuje každých 2,5 s. |
| **Ochrana svetiel** | Prísna (1 %) / Normálna (5 %) / Uvoľnená (15 %) | Prísna | Kamery, ktoré poskytujú toto nastavenie | Koľko z obrazu sa môže preexponovať do biela, kým automatická expozícia (AE) obraz stmaví. |

{% hint style="info" %}
**Požiadavky na osvetlenie pre multispektrálne kamery typu Bayer (RGN

/ OCN

/ NGB

):** scéna musí mať dostatok svetla vo všetkých troch kanáloch, inak kalibrácia nebude fungovať správne — jedna expozícia snímača pokrýva všetky tri spektrá. Na meranie osvetlenia použite svetelný senzor DAQ alebo prejdite na režim „all-mono“ (M3M), aby každé pásmo malo vlastnú expozíciu. Ak snímka túto podmienku poruší, Chloros

to zistí a upozorní vás (oznámenie o obmedzení oddeľovania). V reálnom čase sa v dlaždici s rozdelením kanálov zobrazí **„Dáta pásma s nízkou spoľahlivosťou · N %“**, ak program unmix obmedzil toto pásmo na 25 % alebo viac jeho pixelov.
{% endhint %}

{% hint style="info" %}
**Ako automatická expozícia využíva expozíciu a zosilnenie (1.3.0).** Ak je zosilnenie nastavené na Auto, Chloros

udržiava zosilnenie na úrovni 0 dB, kým expozícia nedosiahne najdlhšiu povolenú hodnotu (hornú hranicu AE, spúšťaciu periódu poľa alebo strop expozície DAQ); až potom sa zosilnenie zvýši. V poli má prednosť spúšťacia frekvencia: automatická expozícia vyplní obdobie, ktoré táto frekvencia určuje, a až potom zvyšuje zosilnenie, takže stmavujúca scéna už neznižuje frekvenciu spúšťania v režime Auto (vo verzii 1.2.0 to tak bolo). **Ručná**expozícia na člene poľa, ktorá trvá dlhšie ako spúšťacia perióda, spomalí spúšťanie poľa tak, aby sa tomu prispôsobilo; frekvencia automatického spúšťania sa obnoví, akonáhle sa každý člen, ktorý ju spomalil, vráti k automatickej expozícii. Ak sú nastavenia**Expozícia**a**Cieľová jasnosť AE** obidve na Auto a svetelný senzor DAQ poskytuje hodnotu, expozícia každej kamery sa udrží po ustálení automatickej expozície, pokiaľ namerané osvetlenie zostane v rozmedzí približne ±25 %; väčšia, trvalá zmena osvetlenia, reštart nastavení (napríklad zmena oblasti záujmu) alebo vaša vlastná zmena expozície ju opäť aktivuje.
{% endhint %}

{% hint style="info" %}
**Ako funguje automatická expozícia (1.3.0).**

* Pri manuálnej expozícii zostáva zosilnenie v režime Auto, kým ho nezmeníte.
* Malé preosvetlené oblasti nezatemňujú scénu: odlesky do 1 % záberu sa ignorujú a v paneloch známeho kalibračného terča až do 0,5 % panelu.
* Maximálna expozícia DAQ platí len vtedy, keď je v zornom poli kalibračný terč; ak nie je viditeľný žiadny panel, kamera exponuje podľa scény.
* Expozícia zostáva stabilná pri konštantnom osvetlení a ustáli sa do niekoľkých sekúnd po výraznej zmene. Prekročenie rozsahu už trvalo neobmedzuje expozíciu a zmena oblasti záujmu (ROI) opäť nastaví správnu expozíciu.
* Cieľová hodnota jasu neprekračuje dosah kamerydosah, takže návrat svetla nespôsobí preosvetlenie snímky.
* Kamera, ktorej automatická expozícia prestane reagovať v jasnom slnku, sa udrží na úrovni zodpovedajúcej aktuálnemu osvetleniu, ktoré sníma svetelný senzor alebo, ak nie je pripojený, vlastné snímky kamery, a uvoľní sa, keď sa svetlo zmení. Tmavý alebo zakrytý svetelný senzor to nenaruší.
{% endhint %}

### Formát pixelov a rozlíšenie

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Členovia poľa** zobrazujú riadky „Aktuálne“ (formát + ŠxV) a „Zlučovanie“ len na čítanie s poznámkou „Nastaviť v nastaveniach poľa“ — obnovenie prenosu na jednom člene by narušilo synchronizáciu, preto sa tieto nastavenia spravujú v [paneli nastavení poľa](#array-settings-pane).**Samostatné kamery** majú:

| Ovládací prvok | Možnosti | Čo robí |
| --- | --- | --- |
| **Formát pixelov** | 8-bit / 10-bit / 12-bit / 16-bit | Hĺbka bitov snímača; rodina formátov závisí od snímača. Predvolená hodnota je 12-bit a projekt uloží nastavenie, vrátane 8-bit, a pri opätovnom pripojení sa k nemu prispôsobí. |
| **Rozlíšenie** | Plné / Polovičné / Štvrtinové | Vzhľadom na aktuálne zlučovanie: Plné = 2048/N × 1536/N pre binning N×N. |
| **Zlučovanie** | 1×1 (žiadne) / 2×2 / 4×4 | Hardvérové zlučovanie N×N — väčšie hodnoty znižujú rozlíšenie, ale zvyšujú pomer signálu k šumu (SNR) a snímkovú frekvenciu. Zmena tejto hodnoty reštartuje prenos a vynuluje všetky oblasti záujmu (ROI) na nové plné zorné pole. |
| **Aktuálne** | len na čítanie | Skutočné rozmery Š×V a posun (x, y), ktoré sú aktuálne platné. |

### Živý náhľad

Všetko v tejto sekcii sa týka **len zobrazenia**— mení to, čo vidíte v živom prenose, zatiaľ čo uložené snímky zostávajú lineárne a nezmenené — s jednou výnimkou:**Vignette** je radiometrická a ovplyvňuje aj exporty (ako je uvedené nižšie).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Ovládací prvok | Rozsah / možnosti | Predvolené | Platí pre | Čo robí |
| --- | --- | --- | --- | --- |
| **Rozlíšenie renderovania** | 360p (najrýchlejšie) / 480p / 720p / 1080p / Natívne rozlíšenie senzora (najpomalšie) | 720p | Všetko | Výška, pri ktorej backend spúšťa radiometrický reťazec náhľadov. Nižšia hodnota zvyšuje snímkovú frekvenciu bez zmeny zorného poľa. |
| **Index**| Zaškrtávacie políčko „Zapnúť“ + ozubené koliesko | Vypnuté | Iba multispektrálny Bayer,**nie** členovia kombinovaného poľa | Náhľad indexu živých rastlín. Ikona ozubeného kolesa otvorí zdieľaný [Index Calculator](#index-calculator-pane), ktorý je prednastavený na prirodzené pásma filtrov kamery (napr. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Vlastný výraz spolu s LUT (zapnuté/vypnuté, predvolená úroveň 7-stop, predvolená minimálna hodnota 0,2, predvolená maximálna hodnota 1) sa vypočíta pri každom snímku náhľadu. Členovia kombinovaného poľa skryjú tento riadok — pole vlastní jeden zdieľaný index. |
| **Vyváženie bielej** | Vypnuté / Jednorazovo / Nepretržite + tlačidlo opätovného zachytenia | Nepretržite | Iba RGB

| Vyváženie bielej v reálnom čase. Tlačidlo obnovenia znovu zachytí vyváženie bielej z aktuálneho spektra DLS (vypnuté, ak je režim Vypnuté). |
| **Gama** | Zapnuté / Vypnuté | Zapnuté | Len RGB

| Zobrazenie gama (γ = 2,2 LUT) v živom náhľade. Uložené snímky zostávajú lineárne. |
| **Odšumovanie** | Zaškrtávacie políčko + intenzita 0–100 | Vypnuté / 50 | Všetko (pre každú kameru, aj v rámci polí) | Bilaterálny filter v živom náhľade. Vyššia hodnota = hladší, ale menej ostrý obraz. |
| **Ostrosť** | Zaškrtávacie políčko + intenzita 0–100 | Vypnuté / 30 | Všetko | Maska neostrosti v živom náhľade, aplikovaná ako posledná. Môže zosilniť šum. Iba v náhľade. |
| **Vigneta**| Zaškrtávacie políčko + intenzita 0–100 | Vypnuté / 0 | Všetko | Ručné odstránenie zvyškov vignety (zosvetľuje rohy), nanesené na odhad inteligentnej vignety z poľa.**Radiometrické — ovplyvňuje živý náhľad AJ exporty**, na rozdiel od odšumovania a ostrosti. |
| **Farebný profil** | Raw / Lineárny / Prirodzený / Vylepšený / Vlastná teplota | Prirodzený | Iba RGB

| Pozri nižšie. |
| **Farebná teplota** | 2000–10000 K, krok 100 | 5500 K | Iba RGB

, profil Vlastná teplota | Upevňuje vyváženie bielej na pevnú korelovanú farebnú teplotu (vstup DLS sa ignoruje). Posledná zvolená hodnota v Kelvinoch sa zapamätá aj pri prepínaní profilov. |
| **Svetlé oblasti** | −100 až 0, krok 1 | 0 (vypnuté) | Iba RGB

| Novinka vo verzii 1.3.0. Stmavuje najjasnejšie a najbielejšie oblasti obrázka (0 = bez zmeny, −100 = najsilnejšie). Platí pre živý náhľad **a pre snímky na úrovni displeja (Náhľad)**, ktoré zaznamenávajú toto nastavenie (`Chloros:DisplayHighlights`); exporty žiarivosti a odrazivosti nie sú nikdy ovplyvnené. Nemôže obnoviť údaje zo senzora, ktoré boli orezané — na to znížte expozíciu. Prekrývanie zebra sleduje stmavený obraz. Deaktivované s farebným profilom Raw („Neaplikuje sa s farebným profilom Raw.“). |
| **Sýtosť** | 0–200 (100 = neutrálne) | 100 | iba RGB

| Sýtosť HSV v živom náhľade. |
| **Kontrast** | 0–200 (100 = neutrálny) | 100 | Len RGB

| Lineárny kontrast okolo strednej šedej v živom náhľade. |
| **Zrkadliť po horizontále / Zrkadliť po vertikále** | Zaškrtávacie políčka | Vypnuté | Všetko | Otočiť náhľad horizontálne / vertikálne. |
| **Otočenie**| 0° / 90° / 180° / 270° | 0° | Všetko | Otočiť náhľad. Orientácia sa uplatňuje na konci reťazca náhľadov na pozadí.**Uložené snímky (1.3.0):**produkty Náhľad, Radiancia, Odrazivosť a Index samostatnej kamery sa otočia pomocou Zrkadlenia/Otočenia, zatiaľ čo**súbory Raw a Debayered zostávajú v natívnej orientácii kamery**; každý súbor záberu zaznamenáva toto nastavenie do svojho XMP, takže pri neskoršom spracovaní súboru Raw sa uplatní rovnakým spôsobom.**Súbory v poli zostávajú v natívnej orientácii** (nastavenie sa zaznamenáva, ale neuplatňuje) a kompozitné zobrazenia poľa ho ignorujú. |**Sémantika farebných profilov** (kamery RGB

):

* **Raw** — úplne obíde spracovateľský reťazec.
* **Lineárny** — tmavý signál + vyrovnanie poľa + vyváženie bielej; bez farebnej matice, bez gama.
* **Prírodný** *(predvolené)* — lineárny plus nameraná matica farebnej korekcie a tónová krivka prispôsobená scéne.
* **Vylepšený**— Prírodný plus živosť a lokálny kontrast CLAHE. Dodatočné náklady sa vzťahujú**len na živý náhľad** — uložené snímky vždy dostanú kompletnú úpravu bez ohľadu na profil.
* **Vlastná teplota** — Prírodný s vyvážením bielej nastaveným na zvolenú hodnotu v kelvinoch.

{% hint style="warning" %}
V prípade režimov „Prirodzený“, „Vylepšený“ a „Vlastná teplota“ sa v paneli zobrazuje poznámka k tónovaniu: snímky sú zosvetlené podľa konkrétnej scény, takže uložené *zobrazené* obrázky nie sú porovnateľné medzi jednotlivými snímkami. **Pre merania exportujte radianciu alebo odrazivosť.**
{% endhint %}

### Prekrývacie vrstvy (nakreslené na živom prenose)

Tieto funkcie sú dostupné iba v používateľskom rozhraní — sú nakreslené na videu, nikdy neovplyvňujú prúd ani záznamy.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Prekrývacia vrstva | Ovládacie prvky | Predvolené nastavenie | Funkcia |
| --- | --- | --- | --- |
| **Zebra** | Zaškrtávacie políčko + prah 200–255 | Vypnuté / 250 | Magentové diagonálne pruhy na orezaných pixeloch. |
| **Zameriavací kríž** | Zaškrtávacie políčko | Vypnuté | Značka stredu snímky. |
| **Mriežka** | Vypnuté / 3 × 3 / 9 × 9 | Vypnuté | Kompozičná mriežka. |
| **Histogram** | Zaškrtávacie políčko + šírka 0,10–0,90 snímky | Vypnuté / 0,25 | Pás histogramu priamo v prúde. |
| **Focus Peak** | Zaškrtávacie políčko + prah 20–200 + vzorka farby | Vypnuté / 80 / `#ff5722` | Zvýraznenie hrán podľa Sobelovho algoritmu na zaostrenie. |
| **Rozdelenie kanálov** | Tlačidlo „Zobraziť rozdelenie (Red

/ Green

/ NIR

)“ / Tlačidlo „Skryť rozdelenia“ | Skryté | Pridá tri nezávislé dlaždice v odtieňoch šedej pre každý kanál vedľa kompozitného obrazu (označenie tlačidla zodpovedá filtrovým kanálom fotoaparátu). Každú dlaždicu rozdelenia je možné presúvať a má farbu ohraničenia fotoaparátu. Nedostupné na monochromatických fotoaparátoch. Uloží sa spolu s projektom. |

### Bodový expozimeter

* Zaškrtávacie políčko **Kliknutím na vzorku**: kliknutím na živý obraz odoberiete vzorku jedného pixela (označí ho krížik) alebo kliknutím a ťahaním vyberiete oblasť pre výpočet priemeru pixelov. Tlačidlo**Vymazať**vymaže vzorku a krížik. Táto funkcia sa vzájomne vylučuje s režimom**Aim** v AE-ROI.
* **Rozbaľovacie menu**Zobraziť**:**Raw (bitová hĺbka)**— natívne digitálne hodnoty v bitovej hĺbke snímača (napr. 12-bit → 0..4095) — alebo**Zobrazenie (8-bit)** (predvolené). Ak je aktívny živý index, v položke Zobrazenie sa namiesto toho zobrazí vypočítaná hodnota indexu (napr. NDVI

).
* Na paneli s údajmi sú uvedené súradnice pixelov, veľkosť snímky, formát pixelov, bitová hĺbka a tabuľka kanálov (Chan / Hodnota / %) s označeniami pásiem a vlnovými dĺžkami; páry zelených pixelov podľa Bayerovho systému sú zpriemerované; vzorky z oblasti zobrazujú „N px avg“.

Stav bodového expozimetra platí len pre danú reláciu.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Prediktívna automatická expozícia (riadená DLS)

Táto sekcia sa zobrazí len vtedy, ak je **pripojený aspoň jeden svetelný senzor DAQ** — riešiteľ potrebuje na jej riadenie živé spektrum dopadajúceho žiarenia.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Ovládací prvok | Rozsah | Predvolené nastavenie | Funkcia |
| --- | --- | --- | --- |
| **Zapnúť** | Zaškrtávacie políčko | Zapnuté (samostatné kamery) | Riešiteľ uzavretej formy využíva spektrum DLS spolu so skalármi kalibračného balíka kamery, aby najjasnejšie pásmo dosiahlo hodnotu blízku saturácii, pričom najtmavšie pásmo zostane nad minimálnou hodnotou SNR — jeden záznam expozície na jedno riešenie, bez stabilizačnej slučky. Určené pre časozberné snímky so solárnym napájaním, kde musí byť každý záber správne exponovaný. Backend sa automaticky prepne na reaktívnu automatickú expozíciu (AE), ak sú údaje DLS neaktuálne/chýbajú alebo ak nie je načítaný kalibračný balík. |
| **Vyhladenie (α)** | 0,05–1,0, krok 0,05 | 0,3 | Vyhladenie po sebe idúcich prediktívnych riešení (nižšia hodnota = väčšie vyhladenie). |
| **Odrazivosť scény**| Tlačidlo**Prekalibrovať ρ** | — | Prepočíta faktor odrazivosti scény, ktorý riešiteľ používa. |

{% hint style="info" %}
**Pri použití viacerých fotoaparátov je prediktívna automatická expozícia (AE) štandardne vypnutá** — pri viacerých fotoaparátoch sa o expozíciu starajú inteligentná automatická expozícia Chloros

a automatická expozícia na strane fotoaparátu (s ochranou proti presýteniu), pričom odhad odrazivosti jednej scényodhad odrazivosti nie je bezpečný v zmiešaných scénach. Ak konkrétne chcete radiometrickú expozíciu riadenú DLS, môžete ju tu opäť zapnúť pre jednotlivé kamery.
{% endhint %}

**Strop expozície riadený DAQ a AE viazaná na dopadajúce svetlo.** Nezávisle od vyššie uvedeného zaškrtávacieho políčka, keď je svetelný senzor DAQ priradený k RGB

kamere, Chloros

vypočíta — na základe nameranej absolútnej intenzity dopadajúceho žiarenia — maximálnu expozíciu × zosilnenie, pri ktorej povrch so 100 % odrazivosťou zostane pod úrovňou prekrytia, a uplatní ju ako **strop**automatického nastavenia expozície. Od verzie 1.3.0 sa tento strop uplatňuje len vtedy, keď je kalibračný cieľ v zornom poli danej kamery (zaznamenaný v jej zábere za posledných 10 s); ak sa v zornom poli nenachádza žiadny cieľ, namiesto toho beží bežná automatická expozícia kamery s vlastnou ochranou proti preexponovaniu, takže zvyšok scény nie je tmavší, ako je potrebné. Pokiaľ je limit aktívny, kamera je**viazaná na dopadajúce svetlo**: pracuje v režime otvorenej slučky pri expozícii meranej podľa dopadajúceho svetla so zosilnením na úrovni 0 dB — expozícia sleduje namerané svetlo, nie obsah scény. Keďže horná hranica môže expozíciu iba skrátiť, sama o sebe nemôže spôsobiť preexponovanie. Od verzie 1.3.0 sa horná hranica vypočítava pomocou vlastnej expozičnej konštanty kamery z jej kalibračného balíka (priemerná hodnota pre celú flotilu sa používa len vtedy, ak balík túto hodnotu neobsahuje). Horná hranica sa automaticky deaktivuje – a obnoví sa normálna automatická expozícia scény – vždy, keď nie je v zornom poli žiadny kalibračný cieľ, chýba hodnota z DAQ, je neaktuálna (&gt;30 s), alebo je tmavý, alebo ak ≥15 % snímky preexponuje pri pevne nastavenej expozícii (čo znamená, že senzor a kamera vnímajú odlišné osvetlenie). Neexistuje žiadny prepínač v grafickom rozhraní; ide o štandardné správanie vždy, keď je k RGB

kamere pripojený DAQ.

### Získavanie údajov a spúšťanie

Členovia<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

poľa navyše zobrazujú riadky **Role**(Master v modrej / Slave v zelenej),**Sync Line**a**Peers**, ktoré sú len na čítanie.

| Ovládací prvok | Možnosti | Predvolené | Poznámky |
| --- | --- | --- | --- |
| **Režim spúšťania** | Vypnuté / Zapnuté | Zapnuté | Zakázané pre členy poľa (spúšťanie riadi pole). |
| **Zdroj spúšťania** | Softvér / Linka 0 (M8) / Linka 1 / Linka 2 | Linka 0 | Skrytý, ak je režim spúšťania vypnutý; deaktivovaný pre členov poľa. Linka 0 je opticky izolovaný externý spúšťací vstup M8. |
| **Rýchlosť snímok**| Automaticky / Ručne + hodnota | Automaticky |**Automaticky**: obmedzenie snímkovej frekvencie kamery je vypnuté — expozícia určuje fps a v okienku sa zobrazuje aktuálna snímková frekvencia v reálnom čase.**Ručné**: rýchlosť snímok obmedzíte pomocou posuvníka (od 1 až po maximum obmedzené šírkou pásma), pričom východiskovou hodnotou je aktuálna rýchlosť. Členovia poľa vidia položku „N fps (v reálnom čase)“ len na čítanie s poznámkou „Nastavené v nastaveniach poľa“. |

### Sieť / Prenos

| Riadok | Správanie |
| --- | --- |
| **Veľkosť paketu**| 1500 (štandardná) / 9000 (Jumbo) — predvolené**Jumbo**. |
| **Priepustnosť** | Limit priepustnosti spojenia v MB/s, len na čítanie. Backend túto hodnotu pri každom pripojení/odpojení vyvažuje medzi všetkými pripojenými kamerami. |
| **Spracovanie vyrovnávacej pamäte** | Režim spracovania vyrovnávacej pamäte, len na čítanie. |

### Zachytávanie

Panel končí tlačidlom **„Otvoriť nastavenia zachytávania…“**, ktoré presmeruje na [panel Nastavenia zachytávania](capture.md#the-capture-settings-pane) (deaktivované, kým nie je otvorený projekt — „Vytvorte alebo otvorte projekt na uloženie záznamov“). Ak je kamera skrytá alebo pozastavená, upozornenie vám pripomenie, aby ste ju pred nahrávaním odkryli alebo obnovili.

## Panel nastavení poľa

Otvoríte kliknutím na **ozubené koliesko**v riadku POLE. Nadpis: názov poľa s ceruzkou na premenovanie a**×** na zatvorenie. Časti nižšie označené ako *len kombinované* sa zobrazujú iba pre polia pripojené v kombinovanom režime zobrazenia.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Synchronizácia

Riadky **Master**,**Slaves**a**Sync Line** sú len na čítanie.

### Senzor okolitého osvetlenia

Zobrazuje sa pre kombinované aj samostatné polia:

* Rozbaľovacie menu **Light Sensor**— priradí jeden DAQ k celému poľu. Výber sa okamžite uplatní, prenáša sa do roletového menu**Senzor osvetlenia** každej kamery v poli (stále je možné nastavenie pre každú kameru prepisovať) a začne odosielať spektrá do poľa.
* Riadok **Stav** v reálnom čase: Vypnuté · „Čaká sa na prvé spektrum…“ · „Aktívne — všetky kamery v poli sú korigované na osvetlenie“ · „Žiadne nové spektrum za posledné 3 s — stále sa používa posledná hodnota (žiadny časový limit pre neaktuálne údaje)…“.
* Poznámka v paneli: „Radiometrická korekcia pre celé pole. Nastavenia jednotlivých kamier majú prednosť.“

### Kalibračný cieľ

Vlastná skupina priamo za položkou Senzor okolitého osvetlenia, pre kombinované aj samostatné polia:

* Zaškrtávacie políčko **Povoliť** — „Detekovať cieľ MAPIR

ArUco a overiť LUT pre NDVI

vs. odrazivosť panelu“; ovplyvňuje prekrývanie cieľa a validačné tabuľky kombinovanej dlaždice. V prípade **samostatného** poľa aktivuje vlastnú [tabuľku kalibračného cieľa](#the-calibration-target-table) (jedna na každý riadok kamery v zobrazení zoznamu).

### Zachytenie — jednotné nastavenia senzorov

Zobrazuje sa pre kombinované aj samostatné polia (nadpis skupiny znie „Zachytenie (jednotné pre všetky kamery)“). Tieto nastavenia sa jednotne uplatňujú na každého člena (zmeny pre jednotlivých členov by narušili synchronizáciu). Úpravy sa pripravujú a uplatňujú spoločne pomocou tlačidla **Uplatniť**— s výnimkou prepínača**ROI zapnuté / ROI vypnuté**, ktorý sa uplatňuje okamžite.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Ovládací prvok | Možnosti / rozsah | Čo robí |
| --- | --- | --- |
| **Formát pixelov** | 8-bit / 10-bit / 12-bit / 16-bit | Jednotná bitová hĺbka pre všetky členy; každá kamera preberá formát tejto hĺbky pre svoj vlastný senzor. |
| **Binning** | 1x1 / 2x2 / 4x4 | Hardvérové zlučovanie — zachováva plné zorné pole a zároveň zvyšuje pomer signálu k šumu (SNR) a snímkovú frekvenciu. Jeho zmena vynuluje polia ROI na nové plné zorné pole. |
| Prednastavenie **rozlíšenia** | Plné / Polovičné / Štvrtinové | V závislosti od binningu; vyplní polia ROI orezaním so stredom v strede. |
| **Orezanie ROI (px)**| Prepínač**ROI zapnuté / vypnuté**, tlačidlo**Vybrať**, polia Š / V / X / Y, mapa celého snímača | Orezanie snímača — pozri [Editor ROI](#the-roi-editor). Šírka/výška sa prispôsobujú násobkom 16 (minimálne 64); posuny sa zarovnávajú na násobky 4. Tip „max WxH“ ukazuje hornú hranicu a tlačidlo**Reset** vráti polia do plného zorného poľa. Počas úpravy oranžový orezávací rámček sleduje každú zmenu poľa na mape a na dlaždici senzora. |
| **Frekvencia spúšťania**| Prepínač Auto / Manuálne + fps 0,5–10, krok 0,5 |**Auto**(predvolené): backend odvodí frekvenciu spúšťania z rozlíšenia a šírky pásma — vstup je deaktivovaný a zobrazuje odvodenú hodnotu.**Ručné**: zafixuje vašu hodnotu po kliknutí na Použiť. Zafixovaná frekvencia, ktorú nová veľkosť snímky nedokáže zvládnuť, sa udrží na najvyššej frekvencii, ktorú táto veľkosť umožňuje, a obnoví sa, keď to veľkosť snímky opäť dovolí. |

Poznámka v paneli: „Zmeny formátu/rozlíšenia na krátko reštartujú všetky kamery. Frekvencia spúšťania sa uplatňuje v reálnom čase.“ Tlačidlá **Použiť / Zrušiť** sa nachádzajú v spodnej časti panela.

#### Editor ROI

Novinka vo verzii 1.3.0. Riadok orezania ROI a mapa pod ním fungujú spoločne:

* **ROI zapnuté / ROI vypnuté**—**okamžite**zapne alebo vypne orezanie senzora (kamery sa na krátko reštartujú; dlaždica poľa zobrazuje „Uplatňovanie nastavení…“).**Vypnuté**načíta celý snímač a**zapamätá si vaše orezanie**— polia ho naďalej zobrazujú;**Zapnuté** ho znovu uplatní. Zapnutie oblasti záujmu bez nastaveného orezania uplatní orezanie na polovicu veľkosti so stredom v strede („Nebolo nastavené žiadne orezanie, preto bolo uplatnené orezanie na polovicu veľkosti so stredom v strede. Upravte ho pomocou polí.“).
* **Vybrať**— nakreslí výrez na mape: pretiahnite kurzorom cez oblasť alebo kliknite na dva protiľahlé rohy (kláves Esc zruší výber). Obdĺžnik sa zapíše do polí Š/V/X/Y, prispôsobený podľa vyššie uvedených krokov; nič sa neuplatní, kým nestlačíte**Uplatniť**, takže ho môžete najprv doladiť pomocou šípok v poliach. Jedno stlačenie tlačidla Vybrať vytvorí jeden obdĺžnik.
* **Mapa**zobrazuje výrez na celom snímači, takže ho môžete opäť rozšíriť: „Celý snímač, živý“, kým kamery čítajú celý snímač, alebo „Snímka v plnom formáte z {čas}. Kamery snímajú len výrez“, keď sú orezané. Ak pole doteraz vysielalo len orezané snímky, na mape sa zobrazí „Zatiaľ žiadny pohľad v plnom formáte: toto pole vysielalo len orezané snímky.“ s tlačidlom**Zobraziť celý snímok**, ktoré vypne oblasť záujmu (ROI). Názov výrezu `W×H @ X,Y` sa nachádza mimo rámčeka (nad ním alebo pod ním, ak sa rámček dotýka hornej hrany), na mape aj na dlaždici živého náhľadu.
* Funkcia **Použiť**potvrdí nastavenia polí: orezanie zapne oblasť záujmu (ROI); plný snímok (po stlačení**Reset**) ju vypne, pričom si systém zapamätá predchádzajúce orezanie; oblasť záujmu, ktorá je vypnutá a ktorej polia stále zobrazujú zapamätané orezanie, zostane vypnutá aj po uplatnení ďalšej zmeny (formát, zlučovanie, frekvencia).
* **V zarovnanom poli je okno orezania každého člena umiestnené podľa zarovnania**, takže každá kamera sníma rovnakú časť scény. Pole, ktoré bolo orezané ešte pred zarovnaním, sa najskôr zarovná na celý snímač (orezanie sa zruší, vyrieši sa zarovnanie a orezanie sa obnoví s oknami pre každú kameru).
* Po zmene orezania sa automatická expozícia opäť prispôsobí novému rámovaniu. V rámci orezania môže expozícia člena stále bežať až do konca spúšťacieho obdobia.
* Stav zapnutia/vypnutia oblasti záujmu (ROI) a zapamätané orezanie sa ukladajú spolu s projektom.

### Zarovnanie (ko-registrácia)

Zobrazené pre **kombinované aj samostatné polia** (verzia 1.2.0 to zobrazovala len pre kombinované polia); nadpis skupiny znie „Zarovnanie“.

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Zvyšková** hodnota: „RMS x,xx px“ — zelená pri hodnote pod 1 px, oranžová pri hodnote pod 3 px, červená v ostatných prípadoch alebo ak zlyhala niektorá kamera; „žiadny profil“ pred prvým výpočtom.
* Súhrnný riadok: „✓ Všetky kamery zarovnané (N)“ / „⚠ p/N kamier zarovnaných —  <serial (filter)="">zlyhalo“ / „Orezanie aktívne — Prekalibrujte na zarovnanie (využíva celý senzor)“ / „Čaká sa na ustálenie expozície…“ / „Aplikujú sa nastavenia…“ (kamery sa reštartujú) / „Sústava prestala dodávať snímky — znovu pripojte sústavu“.
* Tabuľka pre jednotlivé kamery: kamera (posledné 4 znaky sériového čísla + filter), chyba reprojekcie v px s počtom zhôd („ref“ pre hlavnú kameru) a skóre normalizovanej krížovej korelácie prekrývania voči minimálnej hranici 0,35.
* **Prekalibrovať zarovnanie** (pred prvým profilom sa zobrazuje nápis „Kalibrovať zarovnanie“) — opätovne spustí ko-registráciu na nových snímkach.
* **Zaškrtávacie políčko „Automaticky exponovať kamery pre zarovnanie“** (predvolene zaškrtnuté) — dočasne zosvetlí tmavé alebo ploché kamery (najprv expozícia, potom zosilnenie), aby mali textúru na porovnanie, a potom obnoví automatickú expozíciu.

Chloros automaticky zarovnáva každé pole po pripojení v oboch režimoch zobrazenia, akonáhle sa ustáli automatická expozícia jednotlivých členov (pokúša sa o to až 5 minút); kombinovaný náhľad sa tiež zarovná pri otvorení. Samostatná matica nemá kompozitnú dlaždicu, preto táto sekcia zisťuje stav zarovnania a sama spúšťa funkciu „Recalibrate“. Prekalibrujte, ak sa zmenilo zaostrenie alebo hĺbka scény. Zarovnanie je **zásadne len pre danú reláciu** — nikdy sa neukladá do profilu, pretože závisí od aktuálnej vzdialenosti scény. Zábery sa stále dajú exportovať s pixelovým zarovnaním (pozri [Vyrovnané exporty](capture.md#per-array-controls)).

### Inteligentná vineta

* Zaškrtávacie políčko **Povoliť korekciu**— aplikuje odhad vinety pre každú kameru na radiometrický reťazec (v reálnom čase**aj** pri exportoch).
* **Kalibrovať z aktuálneho pohľadu**— najprv nasmerujte pole na jednotný cieľ (plochý panel, stenu alebo oblohu); každá kamera sa vyrovná individuálne a stav hlási zisk vyrovnania „n/N kamier · −x,x %“. Tlačidlo**Vymazať** odstráni odhad.
* Stav: „Kalibrované“, „Nekalibrované“ alebo „Čiastočne kalibrované — spustiť znova“, ak sa pri aktuálnom orámovaní stále uplatňujú odhady len niektorých členov (napríklad po tom, čo funkcia „Prekalibrovať“ posunula orezávacie okno člena) — spustite kalibráciu znova.
* Jemné doladenie pre každú kameru pomocou posúvača **Vignette** pre jednotlivé kamery v [Živom náhľade](#live-preview).

### Živý náhľad

* Zaškrtávacie políčko **Histogram**+ posuvník šírky (0,10–0,90 snímky) — pruh histogramu v živom náhľade na dlaždici každého člena naraz (vrátane dlaždíc s rozdelenými kanálmi), [Prekrytie histogramu](#display-overlays-drawn-over-the-live-feed) nastavené pre celé pole. Zobrazuje sa ako zapnuté, ak ho už má každý člen. V**samostatnom** poli je to jediný riadok v skupine.
* *Iba v kombinácii:* **Index**: zaškrtávacie políčko + ozubené koliesko — otvorí zdieľanú [Kalkulačku indexu](#index-calculator-pane) s pásmami nakreslenými zo**všetkých** kamier členov. Riadok s náhľadom výrazu pod ňou zobrazuje aktuálny výraz („Nenastavený žiadny výraz — otvorte kalkulačku a vytvorte ho“), ktorý sa aktualizuje každú sekundu.
* *Iba v kombinovanom režime:* roletové menu **Rozlíšenie renderovania**(rovnaké prednastavenia ako pre jednotlivé kamery, predvolené 720p): výška prúdu živého náhľadu**a** veľkosť uloženého kompozitného exportu. Poznámka v paneli: „Náhľad + veľkosť uloženého kompozitu. Obrazy z jednotlivých kamier sa vždy exportujú v plnom rozlíšení.“

### Zobrazovacie vrstvy *(len v kombinovanom režime)** Zaškrtávacie políčko **Povoliť** (predvolene vypnuté — hlavná kamera sa zobrazuje priamo; zapnuté = vrstvený kompozit).
* Rozbaľovacie menu **Rozbaľovacie ponuky**Popredie**/**Pozadie**: každá členská kamera (podľa názvu) alebo**Index**. Ak je v poli Popredie nastavená možnosť Index, pixely mimo rozsahu LUT Min/Max zobrazujú vrstvu Pozadie.

### Rozdelené zobrazenie *(len kombinované)*

**„Zobraziť členské kamery“**— tlačidlo**Rozdeliť / Skryť členské kamery**, ktoré pridáva živý prenos každej členskej kamery ako samostatné dlaždice mriežky vedľa kompozície. Dlaždice čítajú existujúci rámcový buffer poľa (bez dodatočného pripojenia kamery). Iba v mriežkovom zobrazení; ukladá sa pre každé pole spolu s projektom.

### Funkcie

Panel len na čítanie, aktualizovaný každých 5 s:

* **Názov úrovne**: „Súbežné snímanie“ (zelená) · „Súbežné snímanie (FTD – postupné vysielanie)“ (zelená) · „Postupné snímanie (100 ms posun)“ (oranžová) · „Konfigurácia je príliš veľká“ (červená).
* **Stav snímky**: „x,xx % neúplné“ — zelená pri menej ako 1 %, oranžová pri menej ako 5 %, červená pri 5 % alebo viac.
* ****Linková linka**: „NIC {mbps} Mbps – trvalá {MB/s} MB/s“.

Toto je aktuálny rozpočet šírky pásma poľa. Informácie o základných hodnotách fps a sieťovom modeli — a o tom, čo je potrebné zmeniť, keď sa úroveň zmení na oranžovú alebo červenú — nájdete v [Poliach s viacerými kamerami](arrays.md) a v [Referencii CLI](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Panel Kalkulátor indexu

Tretia stránka bočného panela, spoločná pre nastavenie Indexu pre jednotlivé kamery a pre kombinované pole (vždy len jedna – nadpis znie „Kalkulátor indexu — <camera name="">“ alebo „Kalkulátor indexu — <array name="">“). Prijíma zoznam pásiem (prirodzené pásma filtra kameryprirodzené pásma kamery alebo všetky pásma vo všetkých členoch sústavy), aktuálny výraz a konfiguráciu LUT (zapnuté/vypnuté, úroveň – predvolené 7-stop, min – predvolené 0,2, max – predvolené 1), ako aj živý histogram indexu. Tlačidlo **Apply** potvrdí výraz; zmeny LUT sa v reálnom čase premietajú do náhľadu.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Nastavenia pre jednotlivé kamery vs. nastavenia spravované poľom

Stručný prehľad toho, čo sa nachádza kde, keď je kamera členom poľa:

| Spravované poľom (v paneli kamery len na čítanie) | Stále pre jednotlivé kamery v rámci poľa |
| --- | --- |
| Formát pixelov, rozlíšenie, zlučovanie | Automatická expozícia (expozícia, zosilnenie, cieľová hodnota, vyhladenie, oblasť záujmu) |
| Režim/zdroj spúšte, frekvencia snímok | Odšumovanie, ostrosť, vineta |
| | Orientácia (zrkadlenie/rotácia), prekrývacie prvky na displeji (nastavenia poľa → funkcia Live Preview nastaví histogram na všetkých členoch naraz), bodový expozimeter |
| | Index (polia so samostatným zobrazením), priradenie svetelného senzora |

Ďalšie prierezové správanie:

* **Kombinované vs. samostatné zobrazenie** sa volí pri pripojení poľa: kombinované = jedna zarovnaná kompozitná dlaždica (členy vysielajú iba cez Split View); samostatné = každý člen vykresľuje svoju vlastnú synchronizovanú dlaždicu. Kamera nikdy nezobrazuje súčasne samostatný prenos aj dlaždicu poľa.
* ****Automatické opätovné pripojenie**: otvorenie uloženého projektu obnoví jeho kamery a polia a pred obnovením prenosov opätovne aplikuje všetky uložené nastavenia na backend.
* **Obmedzenie snímania**: skryté alebo pozastavené kamery sú vylúčené z funkcie „Capture All“; pole je úplne blokované len vtedy, ak sú VŠETKY členy skryté/pozastavené. Pozrite si [Nastavenia a režimy snímania](capture.md).

## Ako sa nastavenia uchovávajú

Stav karty kamery sa ukladá **spolu s projektom**, nie v prehliadači:

* Každá reaktívna zmena vytvorí snímku kamier a polí v súbore `cameras.json` projektu (s odskokom 500 ms). To zahŕňa názvy a farby kamier, nastavenia expozície/zosilnenia/AE, formát pixelov/rozlíšenie/binning, frekvenciu spúšťania, nastavenia náhľadu (rozlíšenie renderovania, odstránenie šumu, ostrosť, vinetácia, farebný profil, svetlá, sýtosť/kontrast), orientáciu, prekrývacie vrstvy, rozdelenie kanálov, konfiguráciu indexu, nastavenia prediktívneho AE, AE ROI, názvy polí, režim zobrazenia, nastavenia snímania polí (vrátane orezania oblasti záujmu, či je oblasť záujmu zapnutá a zapamätané orezanie, keď je vypnutá), rozpočet vodičov poľa a blok mriežky (zoom kanála, režim zobrazenia, uzamknutie mriežky, ručné poradie dlaždíc, skryté kamery, uzavreté dlaždice, aktívna kamera).
* Priradenia svetelných senzorov sa ukladajú do súboru projektu `sensors.json`.
* Opätovné otvorenie projektu opätovne pripojí hardvér a znovu uplatní všetky tieto nastavenia.
* **Žiaden otvorený projekt = iba relácia**: bez projektu sa po zatvorení programu Chloros nič neuloží.
* Iba pre aktuálnu reláciu bez ohľadu na projekt: stav pozastavenia, vzorky bodového merania, začiarkavacie políčko Kalibračný cieľ pre každú kameru (vždy otvorené v polohe vypnuté) a profil zarovnania poľa (podľa návrhu prepočítavaný pre každú reláciu).
* Jedna výnimka: výbery exportu v **Nastaveniach snímania** a režim snímania pretrvávajú pre každý projekt v lokálnom úložisku aplikácie, nie v `cameras.json` — pozri [Nastavenia a režimy snímania](capture.md).</array></camera></serial>

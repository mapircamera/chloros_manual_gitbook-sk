# Nastavenia kamier

Karta **Kamery**je ovládací panel v reálnom čase pre kamery LATTICE v Chloros: hlavná oblasť zobrazenia, v ktorej sa každá pripojená kamera zobrazuje ako živá dlaždica, a bočný panel, ktorý prechádza medzi tromi stránkami –**zoznamom kamier**,**panel nastavení**(nastavenia pre jednotlivé kamery, pole kamier alebo snímanie – vždy jedno po druhom) a**kalkulátor indexu**. Táto stránka dokumentuje všetky ovládacie prvky v zozname kamier, v paneli nastavení pre jednotlivé kamery a v paneli nastavení poľa kamier. Režimy snímania, výber typu exportu a postup „Capture All“ (Snímať všetko) nájdete na doplnkovej stránke [Nastavenia a režimy snímania](capture.md).

Karta Kamery sa zobrazí v bočnom paneli, akonáhle bude backend Chloros pripravený. Všetky nižšie uvedené ovládacie prvky komunikujú s lokálnym backendom prostredníctvom `127.0.0.1:5000`; zmeny sa okamžite prejavia na živom obraze z kamery, pokiaľ nie je uvedené inak.

## Typy kamier používané na tejto stránke

Ovládacie prvky sa zobrazujú alebo skryjú v závislosti od toho, aký typ kamery je vybraný. V tejto príručke sa používajú nasledujúce pojmy:

| Pojem | Význam | Kanály filtra |
| --- | --- | --- |
| **Kamera RGB** | LATTICE M3C s filtrom FRGB (model obsahuje `-FRGB`) | Red / Green / Blue |
| **Multispektrálna kamera Bayer** | LATTICE M3C s filtrom FRGN, FOCN alebo FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — jeden úzkopásmový filter, jedno kalibrované pásmo | Jedno pásmo |
| **Člen poľa** | Kamera pripojená ako súčasť synchronizovaného poľa (kombinované alebo samostatné zobrazenie) | Podľa svojho filtra |

Kamery RGB prechádzajú fotometrickým spracovaním (vyváženie bielej, farebné profily, gama); multispektrálne a monochromatické kamery prechádzajú radiometrickým reťazcom a preskakujú fotometrické nastavenia. Členovia poľa odovzdávajú nastavenia na úrovni prúdu (formát pixelov, rozlíšenie, zlučovanie pixelov, spúšťač, frekvencia snímok) do sústavy — tieto riadky sa v paneli pre jednotlivé kamery stanú iba na čítanie a presunú sa do panela nastavení sústavy.

## Hlavná oblasť

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

prenosu Ak nie sú pripojené žiadne kamery, v oblasti prenosu sa zobrazí úvodná obrazovka **„Pripojte kameru a začnite“**s dvoma tlačidlami:**Pripojiť kameru**(zelené, otvorí dialógové okno na pripojenie jednej kamery) a**Pripojiť pole** (modré, otvorí dialógové okno na pripojenie poľa). Samotné dialógové okná na pripojenie sú zdokumentované v časti [Pripojenie kamier](connecting.md); pojmy týkajúce sa poľa (synchronizácia, vrstvy, šírka pásma) sú popísané v [Súbory viacerých kamier](arrays.md). Keď otvoríte uložený projekt, ktorý obsahuje kamery, na úvodnej obrazovke sa namiesto toho zobrazí indikátor načítavania s textom „Znovu otváranie N uložených kamier…“ , zatiaľ čo Chloros obnovuje prúdy z poslednej relácie.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Horná lišta

| Ovládací prvok | Funkcia |
| --- | --- |
| **Prepínanie režimu zobrazenia**| Prepína medzi**mriežkovým zobrazením**(všetky dlaždice ako bunky) a**zobrazením zoznamu** (polia v plnej šírke hore, JEDNA aktívna kamera dole). Nápovedy: „Prepnúť na mriežkové zobrazenie“ / „Prepnúť na zobrazenie zoznamu“. |
| **Zámok mriežky**(visací zámok) | Predvolene**uzamknutá** — dlaždice sú zafixované na mieste. Odomknutím môžete dlaždice presúvať a usporiadať do ľubovoľných pozícií (medzery zostanú zachované). Mriežka sa automaticky opäť uzamkne pri každom pripojení novej kamery. Nápovedy: „Odomknúť mriežku (povoliť presúvanie dlaždíc)“ / „Zamknúť mriežku (zafixovať dlaždice na mieste)“. |
| Posuvník **Zväčšenie prúdu** | Veľkosť dlaždice, od 60 px až po celú šírku kontajnera. Dlaždice si zachovávajú pomer strán 4:3. Pri šírke dlaždice menšej ako 200 px sa prekrytia s názvom a fps skryjú, aby dlaždica zostala prehľadná. |

### Dlaždice živého prenosu

Každá kamera vykresľuje kompozitnú živú dlaždicu; kamera môže navyše zobrazovať tri dlaždice v odtieňoch šedej s **rozdelením podľa kanálov** (pozri [Rozdelenie kanálov](#display-overlays-drawn-over-the-live-feed)), a pole kamier vykresľujú kombinovanú dlaždicu. Aktívna dlaždica je označená výberovým prstencom v farbe kamery (alebo poľa).

Po nabehnutí kurzora na dlaždicu sa zobrazí tlačidlo na zatvorenie **X**:

* Zatvorenie **kompozitnej** dlaždice, pričom jej rozdelené kanály zostanú zobrazené, iba skryje kompozitnú dlaždicu.
* Zatvorenie **poslednej viditeľnej dlaždice samostatnej kamery** odpojí danú kameru.
* **Rozdelené dlaždice, ktoré sú súčasťou kombinovaného poľa, nikdy kameru neodpoja** — iba ju skryjú.

Ak je mriežka odomknutá, môžete ľubovoľnú dlaždicu pretiahnuť do ľubovoľného miesta; rozloženie sa uloží spolu s projektom.

## Bočný panel — zoznam

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

kamier Prvá stránka bočného panela obsahuje zoznam všetkých pripojených kamier a skupín:

* **Pripojiť kameru**(zelené) /**Pripojiť skupinu** (modré, počas skenovania sa zobrazuje „Detekujem...“). Obe možnosti sú deaktivované, pokiaľ je otvorené dialógové okno pripojenia.
* **Zachytiť všetko** (červené) — zachytí všetky uvedené kamery s typmi exportu zvolenými v Nastaveniach zachytávania. Vyžaduje otvorený projekt. Úplná dokumentácia je k dispozícii v [Nastaveniach a režimoch zachytávania](capture.md).
* **Ikona ozubeného kolesa nastavení snímania** (vedľa tlačidla „Zachytiť všetko“) — otvorí [panel nastavení snímania](capture.md#the-capture-settings-pane). Je deaktivovaná bez projektu alebo počas snímania.

### Riadky kamier

Každý riadok kamery zobrazuje farebne označený okraj (vlastná farba kamery), označenie „CAM“ — s modrým písmenom **M**(hlavná) alebo zeleným písmenom**S** (slave) pre členov poľa — a zobrazené meno. Predvolené meno je `LATTICE-MODEL (serial)`; premenujte ho v paneli nastavení pre jednotlivé kamery. Tlačidlá v riadku:

| Tlačidlo | Účinok |
| --- | --- |
| **Oko**| Prepína viditeľnosť. Skryté kamery zmiznú z mriežky a sú**vylúčené z funkcie Zachytiť všetko**. |
| **Ozubené koleso** | Otvorí okno nastavení pre jednotlivé kamery (ďalšia časť). |
| **Pozastaviť / Prehrať**| Pozastaví živý náhľad**len na strane zobrazenia** – záznam na pozadí pokračuje. Pozastavené kamery nemôžu zaznamenávať. |
| **X** | Odpojiť. Používateľské rozhranie sa aktualizuje okamžite (v optimálnom prípade); samotné odpojenie na pozadí môže trvať 10–30 s. |

### Riadky poľa

Riadok poľa zobrazuje označenie „ARRAY“ v farbe poľa, názov poľa (ktorý je možné premenovať v nastaveniach poľa) a tlačidlo **DAQ · zapnúť/vypnúť**—**zapnuté**, ak je nastavený svetelný senzor na úrovni poľa *alebo* akýkoľvek člen má senzor pre konkrétnu kameru; v jeho popise je presne uvedené, ktorý senzor čo sníma. Kamery v poli sú uvedené pod ním so zarážkou vo vlastných riadkoch. Tlačidlá v riadku poľa: **oko**(skryje/zobrazí VŠETKY kamery naraz),**ozubené koleso**(okno nastavení poľa),**X**(odpojí celé pole).

Stav svetelného senzora (DLS) používaný v riadkoch poľa a v paneli nastavení poľa má štyri stavy:**vypnutý**,**čaká**(zatiaľ žiadne spektrum),**aktívny**(spektrum dorazilo v priebehu posledných 3 s) a**neaktuálny** — žiadne nové spektrum za 3 s, ale posledná hodnota sa *stále používa* (hodnoty DAQ nikdy nevypršia na ceste zachytávania).

Samostatné kamery a celé skupiny polí môžete v bočnom paneli presúvať jedna cez druhú, aby ste zmenili poradie v zozname; členy poľa nie je možné presúvať samostatne.

## Panel nastavení pre jednotlivé kamery

Otvorte ho kliknutím na **ozubené koliesko** v riadku kamery. Panel sa posunie nad zoznam kamier.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Záhlavie**:**farebná vzorka**kamery (kliknutím otvoríte natívny výber farieb – nastavuje farbu ohraničenia bočného panela a výberového prstenca dlaždíc),**názov**s tlačidlom**Pomenovať**v podobe ceruzky (uloženie prázdneho názvu vráti predvolený názov `MODEL (serial)`) a**×** na zatvorenie.

### Živý histogram

V hornej časti panela sa nachádza živý histogram jasu vypočítaný z náhľadu JPEG pri frekvencii ~8 Hz. Priemerná hodnota je vážená podľa Bayerovho algoritmu — (R+2G+B)/4 — aby zodpovedala vlastnému meraniu automatického nastavenia expozície (AE) fotoaparátu.

* **Orange prerušovaná čiara**= cieľová hodnota AE.**Presuňte ju vodorovne, aby ste zmenili cieľ** — pri uvoľnení sa odošle jeden príkaz a presúvaním sa režim cieľa automatického nastavenia expozície prepne na manuálny.
* **Green plná čiara** = skutočná priemerná luma (hodnota, ktorú aktuálne poskytuje automatické nastavenie expozície).
* **Tlačidlo RGB** (vpravo hore): prepína prekrývacie histogramy pre jednotlivé pásma, farebne odlíšené podľa filtra fotoaparátu (napr. pri FRGN: sivá NIR, zelená, červená). Pri monochromatických (M3M) kamerách má tlačidlo nápis „MONO“ a je deaktivované — monochromatický režim vždy zobrazuje jednopásmový histogram jasu.
* Popisky na osi X zodpovedajú bitovej hĺbke senzora aktuálneho formátu pixelov: 0..255, 0..1023, 0..4095 alebo 0..65535.

### Riadky

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

s informáciami o kamere | Riadok | Správanie |
| --- | --- |
| **Model** | Iba na čítanie (napr. `LATT-M3C-L87-FRGN`). |
| **Radiometrická kalibrácia**| Green**„Aktívna“**s popisom zobrazujúcim úroveň kalibrácie, hash, dátum kalibrácie a zoznam pásiem, načítanými z kalibračného balíka kamery (pozri [Továrenská radiometrická kalibrácia](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Skryté pre kamery RGB** — disponujú fotometrickou kalibráciou vyváženia bielej, nie žiarivosťou v jednotlivých pásmach. |
| **Správa o kalibrácii**| Tlačidlo**Stiahnuť** — otvorí PDF certifikát NIST o kalibrácii kamery podľa sériového čísla v prehliadači vášho operačného systému. Ak certifikát ešte nie je uložený v cache, Chloros namiesto toho zobrazí upozornenie. |
| **Sériové číslo** | Iba na čítanie. |
| **Firmware**| Zobrazuje aktuálnu verziu a následne vyhľadá dostupnú verziu pre tento model (uložené v cache podľa modelu — skupina N kamier skontroluje server len raz). Stavy: „Kontroluje sa…“ → Tlačidlo**„Aktualizovať na X“**→ „Aktualizuje sa…“ → „Aktualizované z A na B“ / „Zlyhalo: …“ / „Preskočené: …“ / zelené**„Aktualizované“**. Nápoveda k tlačidlu aktualizácie: „Obnovenie továrenských nastavení + aktualizácia firmvéru + preprogramovanie UserSet1. ~2–3 minúty; neodpájajte.“ |
| **IP** | Iba na čítanie. |
| **Teplota** | Iba na čítanie, aktualizuje sa každých 3 s. Pri ≥65 °C svieti oranžovo a pri ≥75 °C červeno so symbolom ⚠. |
| Zaškrtávacie políčko **Kalibračný cieľ** | Aktivuje detekciu odrazového cieľa ArUco pomocou validačnej tabuľky NDVI pre každý panel pod živým prenosom (zobrazenie zoznamu). Platí len pre danú reláciu — vždy sa otvára v deaktivovanom stave. |
| Rozbaľovacie menu **Svetelný senzor** | Priradí svetelný senzor DAQ (DAQ-E/M/U, zo zoznamu na karte Svetelné senzory) k tejto kamere na účely korekcie osvetlenia smerujúceho nadol (DLS) a prediktívnej automatickej expozície. Voľba „Žiadne“ zruší priradenie. Ak nie sú pripojené žiadne senzory, v roletovom menu sa zobrazí „(žiadne senzory nie sú pripojené — otvorte kartu DAQ)“. Priradenie sa uloží spolu s projektom. |

### Expozícia a zosilnenie

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Všetky číselné vstupy tu používajú otočné ovládače s funkciou „podržaním zrýchlenia“: ťuknutie = ±1, podržanie &gt;1,5 s = ±10, podržanie &gt;3 s = ±100. Hodnota sa odošle do kamery po uvoľnení.

| Ovládací prvok | Rozsah / možnosti | Predvolené nastavenie | Platí pre | Funkcia |
| --- | --- | --- | --- | --- |
| **Expozícia (us)**| Minimálna/maximálna hodnota v reálnom čase kamery | Auto | Všetko | Doba expozície v mikrosekundách s prepínačom**Auto/Ručné**. Auto = nepretržitá automatická expozícia na strane kamery. |
| **Zosilnenie (dB)**| Aktuálne min./max. hodnoty fotoaparátu (napr. až 48 dB) | Manuálne (vypnuté) | Všetko | Analógové/digitálne zosilnenie s vlastným prepínačom**Auto/Manuálne**. |
| **Cieľová jasnosť AE**| 0–255 | 80, režim**Auto**| Všetko (upraviteľné, ak je zapnutá AE alebo automatické zosilnenie) | Jasnosť, na ktorú sa zameriava AE. V režime**Auto**(predvolené nastavenie) si cieľovú hodnotu vyberá riadiaci modul na základe histogramu, pričom expozícia sa udržiava na úrovni 60–75 % maximálnej hodnoty snímača. Zadanie hodnoty alebo pretiahnutie oranžovej čiary na histograme prepne režim na**Ručný**. |
| **Vyhladenie AE** | 0,5–40, krok 0,1 | 8,0 | Všetko | Tlmenie AE. Popis: „Nižšia hodnota = AE reaguje rýchlejšie (môže pulzovať pri vysokej snímkovej frekvencii). Vyššia hodnota = plynulejšie / pomalšie.“ Hodnoty výrazne nižšie ako predvolená hodnota môžu spôsobiť pulzovanie AE a destabilizovať streamovanie pri vysokých snímkových frekvenciách; 8,0 je stabilná predvolená hodnota. |
| **Oblasť záujmu AE**| Zaškrtávacie políčko „Zapnúť“ + tlačidlo**Aim**| Vypnuté | Všetko | Ak je táto funkcia zapnutá, AE meria iba oblasť označenú prerušovanou zelenou čiarou namiesto celého snímku.**Aim** aktivuje funkciu „kliknutie na umiestnenie“ v živom prenose: kliknutím vycentrujete oblasť na 30 % snímky; kliknutím a ťahaním vytvoríte vlastný obdĺžnik (minimálne 5 % × 5 %). Funkcia „Aim“ sa po jednom umiestnení automaticky deaktivuje. Oblasť sa premietne späť do natívnych súradníc kamery podľa akéhokoľvek nastavenia rotácie/zrkadlenia a uloží sa spolu s projektom. |
| **Rýchlosť ladenia AE** | 0,1–5, krok 0,1 | 1,0 | Len pre členov skupiny | Rýchlosť, s akou cieľ automatického AE sleduje zmeny jasu scény; pri hodnote 1,0× sa kontrola opakuje každých 2,5 s. |
| **Ochrana svetlých oblastí** | Prísna (1 %) / Normálna (5 %) / Uvoľnená (15 %) | Prísna | Kamery, ktoré poskytujú toto nastavenie | Koľko z obrazu sa môže preexponovať do biela, kým automatická expozícia (AE) obraz stmaví. |

{% hint style="info" %}
**Požiadavky na osvetlenie pre multispektrálne kamery typu Bayer (RGN / OCN / NGB):** scéna musí mať dostatok svetla vo všetkých troch kanáloch, inak kalibrácia nebude fungovať správne — expozícia jedného senzora pokrýva všetky tri spektrá. Na meranie osvetlenia použite svetelný senzor DAQ alebo prejdite do režimu „all-mono“ (M3M), aby každé spektrálne pásmo malo vlastnú expozíciu. Ak snímka túto podmienku poruší, Chloros to zistí a upozorní vás (oznámenie „unmix-clamp“).
{% endhint %}

### Formát pixelov a

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

rozlíšenie**Členovia poľa** zobrazujú riadky „Current“ (formát + Š x V) a „Binning“, ktoré sú len na čítanie, s poznámkou „Nastavené v nastaveniach poľa“ — reštart prenosu u jedného člena by narušil synchronizáciu, preto sa tieto nastavenia spravujú v [paneli nastavení poľa](#array-settings-pane).**Samostatné kamery** majú:

| Ovládací prvok | Možnosti | Čo robí |
| --- | --- | --- |
| **Formát pixelov** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Formát pixelov snímača (bitová hĺbka). |
| **Rozlíšenie** | Plné / Polovičné / Štvrtinové | Vzhľadom na aktuálne zlučovanie: Plné = 2048/N × 1536/N pre zlučovanie N×N. |
| **Zlučovanie** | 1x1 (žiadne) / 2x2 / 4x4 | Hardvérové zlučovanie N×N — väčšie hodnoty znižujú rozlíšenie, ale zvyšujú pomer signálu k šumu (SNR) a snímkovú frekvenciu. Zmena tejto hodnoty reštartuje prenos a resetuje všetky oblasti záujmu (ROI) na nové plné zorné pole. |
| **Aktuálne** | len na čítanie | Skutočné rozmery Š × V a posun (x, y), ktoré sú aktuálne platné. |

### Živý náhľad

Všetko v tejto sekcii sa týka **len zobrazenia**— mení to, čo vidíte v živom prenose, zatiaľ čo uložené snímky zostávajú lineárne a nezmenené — s jednou výnimkou:**Vignette** je radiometrická a ovplyvňuje aj exporty (uvedené nižšie).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Ovládací prvok | Rozsah / možnosti | Predvolené | Platí pre | Čo robí |
| --- | --- | --- | --- | --- |
| **Rozlíšenie renderovania** | 360p (najrýchlejšie) / 480p / 720p / 1080p / Natívne rozlíšenie senzora (najpomalšie) | 720p | Všetko | Výška, pri ktorej backend spúšťa reťazec radiometrického náhľadu. Nižšia hodnota zvyšuje snímkovú frekvenciu bez zmeny zorného poľa. |
| **Index**| Zaškrtávacie políčko „Zapnúť“ + ozubené koliesko | Vypnuté | Iba multispektrálne snímače typu Bayer,**nie** členovia kombinovaných polí | Živý náhľad indexu vegetácie. Ikona ozubeného kolesa otvorí zdieľaný panel [Index Calculator](#index-calculator-pane), ktorý je prednastavený na prirodzené pásma filtrov kamery (napr. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Vlastný výraz spolu s LUT (zapnuté/vypnuté, predvolená úroveň 3, predvolená minimálna hodnota 0,2, predvolená maximálna hodnota 1) sa vypočíta pri každom snímku náhľadu. Členovia kombinovaného poľa tento riadok skryjú — pole vlastní jeden zdieľaný index. |
| **Vyváženie bielej** | Vypnuté / Jednorazovo / Nepretržite + tlačidlo opätovného zachytenia | Nepretržite | Iba RGB | Vyváženie bielej v reálnom čase. Tlačidlo obnovenia opätovne zachytí vyváženie bielej z aktuálneho spektra DLS (vypnuté, ak je režim Vypnuté). |
| **Gamma** | Zapnuté / Vypnuté | Zapnuté | Len RGB | Zobrazenie gammy (γ = 2,2 LUT) v živom náhľade. Uložené snímky zostávajú lineárne. |
| **Odstránenie šumu** | Zaškrtávacie políčko + intenzita 0–100 | Vypnuté / 50 | Všetko (pre každú kameru, aj v rámci polí) | Bilaterálny filter v živom náhľade. Vyššia hodnota = hladší, ale menej ostrý obraz. |
| **Ostrosť** | Zaškrtávacie políčko + intenzita 0–100 | Vypnuté / 30 | Všetko | Maska neostrosti v živom náhľade, aplikovaná ako posledná. Môže zosilniť šum. Iba v náhľade. |
| **Vigneta**| Zaškrtávacie políčko + intenzita 0–100 | Vypnuté / 0 | Všetko | Ručné odstránenie zvyškových vinetov (zosvetľuje rohy), nanesené na odhad inteligentnej vinetácie z matice.**Radiometrické — ovplyvňuje živý náhľad AJ export**, na rozdiel od funkcií Odšumovanie a Ostrosť. |
| **Farebný profil** | Raw / Lineárny / Prirodzený / Vylepšený / Vlastná teplota | Prirodzený | Iba RGB | Pozri nižšie. |
| **Farebná teplota** | 2000–10000 K, krok 100 | 5500 K | Iba RGB, profil Vlastná teplota | Upevní vyváženie bielej na pevnú korelovanú farebnú teplotu (vstup DLS sa ignoruje). Posledná zvolená hodnota v kelvinoch sa zapamätá aj pri prepínaní profilov. |
| **Sýtosť** | 0–200 (100 = neutrálna) | 100 | Iba RGB | Sýtosť HSV v živom náhľade. |
| **Kontrast** | 0–200 (100 = neutrálny) | 100 | Iba RGB | Lineárny kontrast okolo strednej šedej farby v živom náhľade. |
| **Zrkadlenie po horizontále / Zrkadlenie po vertikále** | Zaškrtávacie políčka | Vypnuté | Všetko | Zrkadlenie náhľadu po horizontále / vertikále. |
| **Otočenie**| 0° / 90° / 180° / 270° | 0° | Všetko | Otočenie náhľadu. Orientácia sa uplatňuje na konci reťazca náhľadov na strane servera —**uložené snímky zostávajú v pôvodnej orientácii fotoaparátu**, a kompozitné zobrazenia ju ignorujú. |**Sémantika farebných profilov** (fotoaparáty RGB):

* **Raw** — úplne obíde reťaz spracovania.
* **Lineárne** — tmavý signál + vyrovnanie poľa + vyváženie bielej; bez farebnej matice, bez gama.
* **Prirodzené** *(predvolené)* — lineárne plus nameraná matica farebnej korekcie a tónová krivka prispôsobená scéne.
* **Vylepšený**— Natural plus živosť a lokálny kontrast CLAHE. Dodatočný poplatok sa vzťahuje**len na živý náhľad** — uložené snímky vždy prechádzajú kompletným spracovaním bez ohľadu na profil.
* **Vlastná teplota** — Natural s vyvážením bielej nastaveným na zvolenú hodnotu v Kelvinoch.

{% hint style="warning" %}
Pri nastaveniach Prírodný, Vylepšený a Vlastná teplota sa v paneli zobrazuje poznámka k tónom: snímky sú zosvetlené podľa konkrétnej scény, takže uložené *zobrazené* obrázky nie sú porovnateľné medzi jednotlivými snímkami. **Exportujte žiarivosť alebo odrazivosť na účely merania.**
{% endhint %}

### Prekryvné vrstvy na displeji (nakreslené na živom prenose)

Tieto sú k dispozícii len v používateľskom rozhraní – sú nakreslené na videu, nikdy sa nedotýkajú živého prenosu ani záznamov.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Prekrývací prvok | Ovládacie prvky | Predvolené nastavenie | Čo robí |
| --- | --- | --- | --- |
| **Zebra** | Zaškrtávacie políčko + prah 200–255 | Vypnuté / 250 | Magentové diagonálne pruhy na orezaných pixeloch. |
| **Zameriavací kríž** | Zaškrtávacie políčko | Vypnuté | Značka stredu snímky. |
| **Mriežka** | Vypnuté / 3 × 3 / 9 × 9 | Vypnuté | Kompozičná mriežka. |
| **Histogram** | Zaškrtávacie políčko + šírka 0,10–0,90 snímky | Vypnuté / 0,25 | Pás histogramu v reálnom čase. |
| **Focus Peak** | Zaškrtávacie políčko + prah 20–200 + vzorka farby | Vypnuté / 80 / `#ff5722` | Zvýraznenie hrán podľa Sobelovej metódy na zaostrenie. |
| **Rozdelenie kanálov** | „Zobraziť rozdelenie (Red / Green / NIR)“ / tlačidlo „Skryť rozdelenia“ | Skryté | Pridáva tri nezávislé dlaždice v odtieňoch šedej pre každý kanál vedľa kompozitného obrazu (označenie tlačidla zodpovedá filtrovým kanálom fotoaparátu). Každú rozdelenú dlaždicu je možné presúvať a má farbu rámčeka fotoaparátu. Nedostupné na monochromatických kamerách. Ukladá sa spolu s projektom. |

### Bodový expozimeter

* Zaškrtávacie políčko **Kliknutím odoberiete vzorku**: kliknutím na živý obraz odoberiete vzorku jedného pixela (označí ho mriežka) alebo kliknutím a ťahaním vyberiete oblasť pre výpočet priemernej hodnoty pixelov.**Vymazať**vymaže vzorku a zameriavací kríž. Vzájomne sa vylučuje s režimom**Aim** v AE-ROI.
* Rozbaľovacie menu **Zobraziť**:**Raw (bitová hĺbka)**— natívne digitálne hodnoty v bitovej hĺbke snímača (napr. 12-bit → 0..4095) — alebo**Zobrazenie (8-bit)** (predvolené). Ak je aktívny živý index, v položke Zobrazenie sa namiesto toho zobrazí vypočítaná hodnota indexu (napr. NDVI).
* Panel s údajmi uvádza súradnice pixelov, veľkosť snímky, formát pixelov, bitovú hĺbku a tabuľku kanálov (Chan / Hodnota / %) s označeniami pásiem a vlnovými dĺžkami; páry zelenej farby podľa Bayerovho systému sú zpriemerované; vzorky z oblasti zobrazujú „N px avg“.

Stav bodového expozimetra platí len pre danú reláciu.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Prediktívna automatická expozícia (riadená DLS)

Táto sekcia sa zobrazí len vtedy, ak je **pripojený aspoň jeden svetelný senzor DAQ** — riešiteľ potrebuje na jej riadenie živé spektrum dopadajúceho žiarenia.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Ovládací prvok | Rozsah | Predvolené nastavenie | Funkcia |
| --- | --- | --- | --- |
| **Zapnúť** | Zaškrtávacie políčko | Zapnuté (samostatné kamery) | Riešiteľ uzavretej formy využíva spektrum DLS spolu so skalármi kalibračného balíka kamery na dosiahnutie najjasnejšieho pásma blízko saturácie, pričom najtmavšie pásmo udržiava nad minimálnou hodnotou SNR — jeden zápis expozície na jedno riešenie, bez stabilizačnej slučky. Určené pre časozberné snímky napájané solárnou energiou, kde musí byť každý záber správne exponovaný. Backend automaticky prejde na reaktívnu automatickú expozíciu (AE), ak sú údaje DLS neaktuálne alebo chýbajú, alebo ak nie je načítaný kalibračný balík. |
| **Vyhladenie (α)** | 0,05–1,0, krok 0,05 | 0,3 | Vyhladenie po sebe idúcich prediktívnych riešení (nižšia hodnota = hladšie). |
| **Odrazivosť scény**| Tlačidlo**Prekalibrovať ρ** | — | Prepočíta faktor odrazivosti scény, ktorý používa riešiteľ. |

{% hint style="info" %}
**Pri použití poľa je prediktívna automatická expozícia (AE) štandardne vypnutá** — pri poľách sa expozícia riadi inteligentnou automatickou expozíciou Chloros plus automatickou expozíciou na strane kamery (s ochranou proti presýteniu) a jediný odhad odrazivosti scény v prediktívnej automatickej expozícii nie je bezpečný pri zmiešaných scénach. Ak konkrétne požadujete radiometrickú expozíciu riadenú systémom DLS, môžete ju tu opäť aktivovať pre jednotlivé kamery.
{% endhint %}

**Horná hranica expozície riadená zberníkom údajov (DAQ) a**AE s fixáciou na dopadajúce svetlo.**Nezávisle od zaškrtnutia políčka vyššie, keď je svetelný senzor DAQ priradený ku kamere RGB, Chloros vypočíta — na základe nameranej absolútnej intenzity žiarenia smerujúceho nadol — maximálnu expozíciu × zosilnenie , pri ktorom povrch s 100 % odrazivosťou zostane pod úrovňou prekrytia, a použije ju ako**strop**pre automatickú expozíciu. Pokiaľ je strop aktívny, kamera je**viazaná na dopadajúce svetlo**: pracuje v režime otvorenej slučky pri expozícii meranej na dopadajúce svetlo so zosilnením 0 dB — expozícia sleduje namerané svetlo, nie obsah scény. Keďže horná hranica môže expozíciu iba skrátiť, sama o sebe nemôže spôsobiť preexponovanie. Horná hranica sa automaticky deaktivuje — a obnoví sa normálna automatická expozícia scény — vždy, keď chýba hodnota z DAQ, je neaktuálna (&gt;30 s) alebo tmavá, alebo ak ≥15 % snímky vykazuje preexponovanie pri fixovanej expozícii (čo znamená, že senzor a kamera vnímajú odlišné osvetlenie). Neexistuje žiadny prepínač v grafickom rozhraní; ide o štandardné správanie vždy, keď je kamera RGB prepojená s DAQ.

### Členovia poľa Acquisition &amp; Trigger

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Array navyše zobrazujú riadky **Role**(Master v modrej / Slave v zelenej),**Sync Line**a**Peers**, ktoré sú len na čítanie.

| Ovládanie | Možnosti | Predvolené | Poznámky |
| --- | --- | --- | --- |
| **Režim spúšťača** | Vypnuté / Zapnuté | Zapnuté | Deaktivované pre členov poľa (spúšťanie riadi pole). |
| **Zdroj spúšťania** | Softvér / Linka 0 (M8) / Linka 1 / Linka 2 | Linka 0 | Skrytý, ak je režim spúšťania vypnutý; deaktivovaný pre členy poľa. Linka 0 je opticky izolovaný externý spúšťací vstup M8. |
| **Rýchlosť snímok**| Auto / Ručné + hodnota | Automaticky |**Automaticky**: obmedzenie snímkovej frekvencie kamery je vypnuté — expozícia určuje fps a v okienku sa zobrazuje aktuálna snímková frekvencia v reálnom čase.**Ručné**: počet snímok za sekundu obmedzíte pomocou posúvača (od 1 až po maximum obmedzené šírkou pásma), pričom východiskovou hodnotou je aktuálna snímková frekvencia. Členovia poľa vidia údaj „N fps (v reálnom čase)“ len na čítanie s poznámkou „Nastavené v nastaveniach poľa“. |

### Sieť / Prenos

| Riadok | Správanie |
| --- | --- |
| **Veľkosť paketu**| 1500 (štandardná) / 9000 (Jumbo) — predvolené**Jumbo**. |
| **Priepustnosť** | Limit priepustnosti spojenia len na čítanie v MB/s. Backend túto hodnotu pri každom pripojení/odpojení vyvažuje medzi všetkými pripojenými kamerami. |
| **Spracovanie vyrovnávacej pamäte** | Režim spracovania vyrovnávacej pamäte len na čítanie. |

### Zachytávanie

Panel končí tlačidlom **„Otvoriť nastavenia zachytávania…“**, ktoré vás presmeruje do [panela Nastavenia zachytávania](capture.md#the-capture-settings-pane) (deaktivované, kým nie je otvorený projekt — „Vytvorte alebo otvorte projekt na uloženie záznamov“). Ak je kamera skrytá alebo pozastavená, upozornenie vám pripomenie, aby ste ju pred snímkovaním odkryli alebo obnovili.

## Panel nastavení poľa

Otvorí sa kliknutím na **ozubené koliesko**v riadku POLE. Záhlavie: názov poľa s ceruzkou na premenovanie a**×** na zatvorenie. Časti nižšie označené ako *len kombinované* sa zobrazujú iba pre polia pripojené v kombinovanom režime zobrazenia.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Synchronizácia

Riadky **Master**,**Slaves**a**Sync Line** sú určené len na čítanie.

### Senzor okolitého osvetlenia

Zobrazuje sa pre kombinované aj samostatné polia:

* Zaškrtávacie políčko **Kalibračný cieľ** — „Detekovať cieľ ArUco MAPIR a overiť NDVI voči LUT odrazivosti panelu“; riadi prekrytie cieľa kombinovanej dlaždice a tabuľku overovania.
* Rozbaľovacie menu **Senzor osvetlenia** — priradí jeden DAQ k celému poľu. Výber sa okamžite uloží, prenesie sa do rozbaľovacieho menu Senzor osvetlenia každej kamery v poľu (stále je možné nastavenie pre každú kameru prepisovať) a začne odosielať spektrá do poľa.
* Riadok **Stav** v reálnom čase: Vypnuté · „Čaká sa na prvé spektrum…“ · „Aktívne — všetky kamery v poli majú korigované osvetlenie“ · „Žiadne nové spektrum za posledné 3 s — stále sa používa posledná hodnota (bez časového limitu pre neaktuálne údaje)…“.
* Poznámka v paneli: „Radiometrická korekcia pre celé pole. Nastavenia jednotlivých kamier majú prednosť.“

### Zachytávanie — jednotné nastavenia senzorov *(len kombinované)*

Tieto nastavenia sa jednotne vzťahujú na každého člena (zmeny pre jednotlivých členov by narušili synchronizáciu). Úpravy sa pripravujú a aplikujú spoločne.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Ovládací prvok | Možnosti / rozsah | Čo robí |
| --- | --- | --- |
| **Formát pixelov** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Jednotný formát senzora pre všetky členy. |
| **Binning** | 1x1 / 2x2 / 4x4 | Hardvérový binning — zachováva celé zorné pole a zároveň zvyšuje pomer signálu k šumu (SNR) a snímkovú frekvenciu. Jeho zmena vynuluje polia ROI na nové celé zorné pole. |
| **Predvolené rozlíšenie** | Plné / Polovičné / Štvrtinové | V závislosti od binningu; vyplní polia ROI orezaním so stredom v strede. |
| **Orezanie ROI (px)**| Číselné polia Š / V / X / Y | Orezanie snímača. Šírka/výška sa prispôsobujú násobkom 16 (minimálne 64); posuny sa prispôsobujú násobkom 4. Tip „max WxH“ ukazuje hornú hranicu a tlačidlo**Reset** vráti plné zorné pole. Počas úpravy sa na dlaždici poľa zobrazuje oranžový náhľad orezania v reálnom čase (vrátane schémy celého snímača pri rozširovaní orezania smerom von). |
| **Frekvencia spúšťania**| Prepínač Auto / Manuálne + fps 0,5–10, krok 0,5 |**Auto**(predvolené): backend odvodí frekvenciu spúšťania z rozlíšenia a šírky pásma — vstup je deaktivovaný a zobrazuje odvodenú hodnotu.**Ručné**: pri kliknutí na**Použiť** sa hodnota zafixuje. |

Poznámka v paneli: „Zmeny formátu/rozlíšenia na krátko reštartujú všetky kamery. Frekvencia spúšťania sa uplatňuje v reálnom čase.“ Tlačidlá **Použiť / Zrušiť** sa nachádzajú v spodnej časti panela.

### Zarovnanie (ko-registrácia) *(len v kombinácii)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Políčko **Zvyšková chyba**: „RMS x,xx px“ — zelená farba pri hodnote pod 1 px, oranžová pri hodnote pod 3 px, červená v ostatných prípadoch alebo ak zlyhala akákoľvek kamera; „žiadny profil“ pred prvým vyriešením.
* Súhrnný riadok: „✓ Všetky kamery zarovnané (N)“ / „⚠ p/N kamier zarovnaných —  <serial (filter)="">zlyhalo“ / „Orezanie aktívne — Prekalibrujte na zarovnanie (využíva celý senzor)“ / „Čaká sa na ustálenie expozície…“.
* Tabuľka pre jednotlivé kamery: kamera (posledné 4 znaky sériového čísla + filter), chyba reprojekcie v px s počtom zhôd („ref“ pre hlavnú kameru) a skóre normalizovanej krížovej korelácie prekrývania voči minimálnej hranici 0,35.
* Tlačidlo **Prekalibrovať zarovnanie** (s nápisom „Kalibrovať zarovnanie“ pred prvým profilom) — opätovne spustí ko-registráciu na nových snímkach.
* Zaškrtávacie políčko **„Automaticky exponovať kamery pre zarovnanie“** (predvolene zaškrtnuté) — dočasne zosvetlí tmavé alebo ploché kamery (najprv expozícia, potom zosilnenie), aby mali textúru na porovnanie, a potom obnoví automatickú expozíciu.

Kombinovaný náhľad sa pri otvorení automaticky zarovná; v prípade zmeny zaostrenia alebo hĺbky scény vykonajte rekalibráciu. Zarovnanie je **zásadne platné len pre danú reláciu** — nikdy sa neukladá do profilu, pretože závisí od aktuálnej vzdialenosti scény. Snímky je stále možné exportovať s pixelovou registráciou (pozri [Vyrovnané exporty](capture.md#per-array-controls)).

### Inteligentná vineta

* Zaškrtávacie políčko **Povoliť korekciu**— aplikuje odhad vinety pre každú kameru na radiometrický reťazec (v reálnom čase**aj** pri exportoch).
* ****Kalibrovať z aktuálneho pohľadu**— najskôr nasmerujte pole na jednotný cieľ (plochý panel, stenu alebo oblohu); každá kamera sa vyrovná individuálne a stav hlási nárast rovinnosti „n/N kamier · −x,x %“. Tlačidlo**Vymazať** odstráni odhad.
* Jemné doladenie pre každú kameru pomocou posuvníka **Vignette** pre jednotlivé kamery v [Živom náhľade](#live-preview).

### Živý náhľad *(len kombinovaný)** **Index**: zaškrtnite políčko + ozubené koliesko — otvorí sa zdieľaná [Kalkulačka indexu](#index-calculator-pane) s pásmami nakreslenými zo**všetkých** kamier v skupine. Riadok s náhľadom výrazu pod ňou zobrazuje aktuálny výraz („Žiaden výraz nie je nastavený — otvorte kalkulačku a vytvorte ho“), ktorý sa aktualizuje každú sekundu.
* Rozbaľovacie menu **Rozlíšenie renderovania**(rovnaké prednastavenia ako pre jednotlivé kamery, predvolené 720p): výška streamu živého náhľadu**a** veľkosť uloženého kompozitného exportu. Poznámka v paneli: „Náhľad + veľkosť uloženého kompozitu. Obrázky z jednotlivých kamier sa vždy exportujú v plnom rozlíšení.“

### Zobrazenie vrstiev *(len kombinované)** Zaškrtávacie políčko **Zapnúť** (predvolene vypnuté — hlavná kamera sa zobrazuje priamo; zapnuté = vrstvený kompozit).
* Rozbaľovacie ponuky **Popredie**/**Pozadie**: každá členská kamera (podľa názvu) alebo**Index**. Ak je v položke Popredie zvolený Index, pixely mimo rozsahu LUT Min/Max zobrazujú vrstvu Pozadie.

### Rozdelené zobrazenie *(len v kombinácii)*

**„Zobraziť členské kamery“**— tlačidlo**Rozdeliť / Skryť členské kamery**, ktoré pridá živý prenos každej členskej kamery ako samostatné dlaždice mriežky vedľa kompozície. Dlaždice čítajú existujúci rámcový buffer poľa (bez dodatočného pripojenia kamery). Iba v mriežkovom zobrazení; ukladá sa pre každé pole spolu s projektom.

### Funkcie

Panel len na čítanie, aktualizovaný každých 5 s:

* **Názov úrovne**: „Súbežné snímanie“ (zelená) · „Súbežné snímanie (FTD – posunuté vysielanie)“ (zelená) · „Postupné snímanie (100 ms posun)“ (oranžová) · „Konfigurácia je príliš veľká“ (červená).
* **Stav snímok**: „x,xx % neúplné“ — zelená pri menej ako 1 %, oranžová pri menej ako 5 %, červená pri 5 % alebo viac.
* **Linková linka**: „NIC {mbps} Mbps – trvalá {MB/s} MB/s“.

Toto je aktuálny rozpočet šírky pásma poľa. Informácie o základnom modeli fps a sieťovom modeli — a o tom, čo zmeniť, keď sa úroveň zmení na oranžovú alebo červenú — nájdete v [Polia s viacerými kamerami](arrays.md) a [Referenčnú príručku CLI](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Panel Kalkulátor indexu

Tretia stránka bočného panela, spoločná pre nástroj Index pre jednotlivé kamery a nástroj Index pre kombinované pole (vždy len jeden – nadpis znie „Index Calculator — <camera name="">“ alebo „Index Calculator —<array name="">

“). Prijíma zoznam pásiem (prirodzené pásma filtrov kamery alebo všetky pásma vo všetkých členoch poľa), aktuálny výraz a konfiguráciu LUT (zapnuté/vypnuté, úroveň – predvolené 3, min. – predvolené 0,2, max. – predvolené 1), ako aj živý histogram indexu. Tlačidlo **Použiť** potvrdí výraz; zmeny LUT sa okamžite uplatnia v náhľade.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Nastavenia pre jednotlivé kamery vs. nastavenia spravované poľom

Stručný prehľad toho, čo sa nachádza kde, keď je kamera členom poľa:

| Spravované na úrovni poľa (v paneli kamery len na čítanie) | Stále pre každú kameru zvlášť v rámci poľa |
| --- | --- |
| Formát pixelov, rozlíšenie, zlučovanie pixelov | Automatická expozícia (expozícia, zosilnenie, cieľová hodnota, vyhladenie, oblasť záujmu) |
| Režim/zdroj spúšte, snímková frekvencia | Odšumovanie, ostrosť, vinetácia |
| | Orientácia (zrkadlenie/rotácia), prekrývacie vrstvy, bodový expozimeter |
| | Index (polia so samostatným zobrazením), priradenie svetelného senzora |

Ďalšie prierezové správanie:

* **Kombinované vs. samostatné zobrazenie** sa volí pri pripojení poľa: kombinované = jedna zarovnaná kompozitná dlaždica (členy vysielajú iba cez Split View); samostatné = každý člen vykresľuje vlastnú synchronizovanú dlaždicu. Kamera nikdy nezobrazuje súčasne samostatný prenos aj dlaždicu z matice.
* **Automatické opätovné pripojenie**: otvorenie uloženého projektu obnoví jeho kamery a matice a pred obnovením prenosov opätovne aplikuje všetky uložené nastavenia na backend.
* **Ovládanie snímania**: skryté alebo pozastavené kamery sú vylúčené z funkcie „Capture All“; pole je úplne zablokované len vtedy, ak sú VŠETKY členy skryté/pozastavené. Pozrite si [Nastavenia a režimy snímania](capture.md).

## Ako sa nastavenia uchovávajú

Stav karty „kamera“ sa ukladá **spolu s projektom**, nie v prehliadači:

* Každá reaktívna zmena vytvorí snímku kamier a polí v súbore `cameras.json` projektu (s odskokom 500 ms). To zahŕňa názvy a farby kamier, nastavenia expozície/zosilnenia/AE, formát pixelov/rozlíšenie/binning, frekvenciu spúšťania, nastavenia náhľadu (rozlíšenie renderovania, odstránenie šumu, ostrosť, vinetácia, farebný profil, sýtosť/kontrast), orientáciu, prekrývanie, rozdelenie kanálov, konfiguráciu indexu, nastavenia prediktívnej automatickej expozície (AE), oblasť záujmu (ROI) pre automatickú expozíciu, názvy polí, režim zobrazenia, nastavenia snímania polí (vrátane polohy orezania oblasti záujmu) a blok mriežky (zoom kanála, režim zobrazenia, uzamknutie mriežky, ručné usporiadanie dlaždíc, skryté kamery, uzavreté dlaždice, aktívna kamera).
* Priradenia svetelných senzorov sa ukladajú do súboru projektu `sensors.json`.
* Opätovné otvorenie projektu opäť pripojí hardvér a znovu uplatní všetky tieto nastavenia.
* **Žiaden otvorený projekt = iba relácia**: ak nie je otvorený žiadny projekt, po zatvorení programu Chloros sa nič neuloží.
* Iba relácia bez ohľadu na projekt: stav pozastavenia, vzorky bodového merania, začiarkavacie políčko Kalibračný cieľ pre každú kameru (vždy sa otvorí v polohe vypnuté) a profil zarovnania poľa (podľa návrhu sa prepočítava pre každú reláciu).
* Jedna výnimka: voľby exportu v **Nastaveniach snímania** a režim snímania pretrvávajú pre každý projekt v lokálnom úložisku aplikácie, nie v `cameras.json` — pozri [Nastavenia a režimy snímania](capture.md).</array></camera></serial>

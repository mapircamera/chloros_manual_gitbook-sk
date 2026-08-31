# Systémy s viacerými kamerami

**Systém**LATTICE pozostáva z dvoch alebo viacerých kamier LATTICE prepojených do jednej synchronizovanej jednotky. Jedna kamera je**hlavná**: vysiela hardvérový spúšťací impulz GPIO na zdieľanú synchronizačnú linku (predvolene**Line2**), takže všetky kamery zachytávajú ten istý okamih. Chloros pridáva časovú synchronizáciu PTP, živý náhľad (dlaždice pre každú kameru alebo jeden zarovnaný kompozitný obraz z viacerých pásiem) a synchronizované snímanie – každý cyklus snímania vytvára jednu**skupinu snímok**, v ktorej všetky kamery zdieľajú rovnaký časový údaj a ID snímky (uvádzané ako `fid:N` vo výstupe snímania).

Polia slúžia na to, aby monochromatické (M3M) kamery generovali vegetačné indexy – jedna kamera prispieva jedným pásmom a pole ich zarovnáva do viacpásmového súboru. Pozri [Monochromatické kamery a vegetačné indexy](mono-indices.md).

Existujú tri rovnocenné spôsoby pripojenia poľa a všetky spúšťajú rovnaký postup „smart-prep“:

| Rozhranie | Vstupný bod |
| --- | --- |
| GUI | Karta Kamery → **Pripojiť pole** (modré tlačidlo) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (prvé sériové číslo = hlavné zariadenie) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (prvé sériové číslo = hlavná jednotka) |

Funkcia Smart-prep vykonáva v tomto poradí: testovanie sieťovej dostupnosti (ICMP DF ping + test GVSP), výber synchronizačnej úrovne, automatické zmenšenie veľkosti rámca tak, aby sa zmestil do vedenia, aktiváciu PTP, automatický výber formátu pixelov pre každú kameru, automatické nastavenie expozície na základe uloženého stavu každej kamery a konfiguráciu spúšťača GPIO na linke 2.

{% hint style="info" %}
Aby toto všetko fungovalo, kamery musia byť dostupné na prepojení — pozri [Pripojenie kamier](connecting.md) pre vyhľadávanie, adresovanie a stiahnutie kalibrácie pri prvom pripojení. Pri zostavách s viacerými kamerami sú nastavenia prijímačasú rovnako dôležité ako rýchlosť spojenia; kompletná tabuľka príznakov a riešení sa nachádza v [CLI Referencia § Nastavenie a ladenie hostiteľskej sieťovej karty](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dialógové okno „Array Connect“

Karta „Cameras“ (Kamery) → **Connect Array**(Pripojiť pole) otvorí trojkrokový sprievodca:**Select (Vybrať) → Display Mode (Režim zobrazenia) → Settings (Nastavenia)**.

### Krok 1 — Výber hlavnej a podriadených

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

kamier Dialógové okno ihneď po otvorení prehľadá sieť („Prehľadávanie siete...“) a následne skontroluje zapojenie spúšťača GPIO („Kontrola zapojenia GPIO...“). Na vytvorenie poľa potrebujete aspoň **2 kamery**.

Test zapojenia vopred vyplní výber rolí, ak je to možné, a zobrazí jeden z troch nápisov:

| Nápis | Význam |
| --- | --- |
| „Zistená hlavná kamera GPIO — výbery vopred vyplnené“ (zelená) | Test zistil topológiu spúšťača; políčka pre hlavnú a podriadenú kameru sú už zaškrtnuté. |
| „Hlavná kamera nezistená – skontrolujte GPIO kábel“ (oranžová) | Žiadna kamera nezaznamenala spúšťací impulz; skontrolujte synchronizačné zapojenie. Role môžete stále vybrať ručne. |
| „Chýba synchronizačný kábel: {sériové čísla}“ (oranžová) | Uvedené kamery nemajú pripojený synchronizačný kábel. |

Tabuľka kamier obsahuje stĺpce **Kamera / Sériové číslo / IP / Hlavná (rádio) / Podriadená (zaškrtávacie políčko)**:

* Vyberte presne **jednu hlavnú kameru**a**jednu alebo viac podriadených kamier**. Opätovným kliknutím na rádio aktuálneho mastera sa jeho výber zruší.
* Kameru označenú ako **„Chýba synchronizačný kábel“** nemožno nikdy vybrať ako slave — slave bez spúšťacieho zapojenia by nekonečne čakala na synchronizačnej linke a poskytovala by nefunkčný obraz. Túto kameru namiesto toho pripojte ako samostatnú kameru.
* Kamery, ktoré sú už pripojené samostatne, *nie* sú deaktivované: pripojenie do poľa uvoľní samostatnú reláciu a kameru opäť otvorí v rámci poľa.

**Ďalej: Režim zobrazenia →**sa aktivuje, akonáhle je vybraný hlavný prístroj a aspoň jeden podriadený prístroj.**Opätovné vyhľadávanie** znovu spustí vyhľadávanie a testovanie zapojenia.

{% hint style="warning" %}
**Zrušiť** je deaktivované počas prebiehajúceho skenovania alebo testovania — zrušenie uprostred testovania môže spôsobiť zlyhanie kamery SDK s firmvérom LATTICE. Počkajte, kým sa dokončí indikátor prebiehajúcej operácie.
{% endhint %}

### Krok 2 — Režim

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

zobrazenia | Režim | Čo získate |
| --- | --- |
| **Samostatné kamery** | Jedna živá dlaždica na kameru, všetky sa spúšťajú súčasne, aby snímky zostali synchronizované. Každá kamera si zachováva vlastnú farbu a nastavenia. |
| **Kombinované kamery** *(predvolené)* | Jedna dlaždica zobrazujúca zarovnaný kompozitný obraz NDVI/index z viacerých pásiem. Kamery zdieľajú farbu pole. |

Režim zobrazenia mení iba prezentáciu živého náhľadu — správanie pri snímaní je v oboch prípadoch rovnaké.

### Krok 3 — Nastavenia poľa a predpokladaný výsledok

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

Pri vstupe do tejto scény Chloros požiada backend o **odporúčanie**a automaticky uplatní kombináciu ROI a binningu, ktorá zodpovedá prijímacej slučke vášho NIC (uprednostňuje zlučovanie pred orezávaním ROI, pretože zlučovanie zachováva celé zorné pole). Každá zmena, ktorú vykonáte, spustí analýzu v reálnom čase a aktualizuje panel**Predpokladaný výsledok** na pravej strane.

Ľavý stĺpec — nastavenia:

| Ovládací prvok | Možnosti | Predvolené | Poznámky |
| --- | --- | --- | --- |
| **ROI (zorné pole)** | Plné (2048×1536) / Polovičné (1024×768) / Štvrtinové (512×384) | Plné | Orezanie snímača: Polovičné/štvrtinové orezať na menšiu oblasť pri natívnej veľkosti pixelov. |
| **Binning** | 1× / 2× (súčet 2×2) / 4× (súčet 4×4) | 1× | Hardvérový binning: 2×2 = plné zorné pole pri štvrtine nákladov na prenos; 4×4 = plné zorné pole pri 1/16. Skryté, ak kamery nepodporujú binning. |
| **Obraz na strane vedenia** (čítanie) | — | — | Šírka × výška po binningu, ktorá sa skutočne odosiela po vedení, zaokrúhlená na násobky 16 (minimálne 64). |
| **Rozlíšenie na pinoch**| zaškrtávacie políčko | vypnuté | Chloros zvyčajne automaticky aktivuje zlučovanie pri pripojení, keď predpokladaná rýchlosť klesne pod**1,5 fps**. Zlučovanie zachováva zvolenú veľkosť snímky a akceptuje nižšiu rýchlosť — a premení konfiguráciu s nadmerným zaťažením na tvrdé odmietnutie pripojenia namiesto automatického zníženia rýchlosti. |
| **Rýchlosť spúšťania** | 0,5–60 fps, krok 0,1 | prázdne = auto | Rýchlosť spúšťania hlavného zariadenia. Nechajte prázdne, aby ju Chloros odvodil sám. |
| **Kapacita linky**| 20–2000 MB/s, krok 10 | prázdne = auto | Koľko môže hostiteľ skutočne spracovať, v MB/s —**jediné číslo, od ktorého závisí celé prideľovanie poľa.** Automaticky zistené zo sieťového adaptéra. Znížte túto hodnotu, ak pole hlási poškodené rámce: zistená hodnota nadhodnocuje USB adaptéry a zdieľané prepínače. Zmenou tejto hodnoty sa opäť spustí živá projekcia. |

Pravý stĺpec — **Predpokladaný výsledok**:

* **Úroveň synchronizácie** — „Súbežné snímanie“ (zelená), „Súbežné snímanie (FTD – postupné vysielanie)“ (zelená), „Postupné snímanie (100 ms posun)“ (oranžová) alebo „Konfigurácia je príliš veľká“ (červená).
* **Prognóza fps** — zobrazuje sa ako rozsah („slabé → jasné“), pretože rýchlosť synchronizovaného poľa je obmedzená expozíciou najpomalšej kamery.
* **Linka NIC** — rýchlosť spojenia a trvalá priepustnosť („NIC {mbps} Mbps · trvalá {N} MB/s“).
* **Kontrola simultánneho vysielania** — dokáže prijímací prstenec NIC hostiteľa absorbovať jeden simultánny burst zo všetkých kamier („Simultánne vysielanie: X MB · využiteľný prstenec NIC: Y MB ✓/✗“).
* **Kontrola kapacity vedenia** — agregovaný dopyt v ustálenom stave vs. strop vedenia zabezpečujúci ochranu pred kolíziami („Kapacita vedenia: {dopyt} MB/s požadovaný {n} kamerami · strop {strop} MB/s ✓/✗ prekročený“).
* **„Maximálny počet kamier na tomto vedení: {n} — stanovený minimálnou šírkou pásma na kameru, takže zlučovanie ho nezvýši.“** — zobrazí sa, keď sa blížiš k (alebo prekročíš) hornú hranicu počtu kamier.
* **„Pri týchto nastaveniach DOCHÁDZA K VYNEChÁVANÍ SNÍMKOV.“**— červené varovanie s dôvodom zo strany backendu, plus zoznam prekážok a modré**návrhy na opravu** („Aby sa táto matica zmestila do siete“ / „Aby sa odblokovalo simultánne snímanie“).

Tlačidlo **Použiť a pripojiť** je zablokované, kým neexistuje prognóza, a jeho popisok vám vysvetlí, prečo je odmietnuté:

| Popisok tlačidla | Význam | Čo skutočne pomôže |
| --- | --- | --- |
| „Analyzujem...“ | Analýza stále prebieha. | Počkajte. |
| **„Príliš veľa kamier pre túto sieť“**| Pole preťažuje sieťové pripojenie (kontrola agregácie zlyhala). | Menej kamier, jumbo rámce v celom reťazci alebo rýchlejšia sieťová karta.**Menšia oblasť záujmu (ROI) NEPOMÔŽE** — pozri nižšie. |
| **„Znížte ROI na povolenie“** | Pri týchto nastaveniach by dochádzalo k vypadávaniu rámcov (zlyhala kontrola burst/ring). | Znížte ROI, zvýšte binning alebo opravte prijímací ring sieťovej karty. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Počas pripájania sa môže zobraziť zelený **panel sťahovania kalibrácie** s ukazovateľom priebehu pre každý sériový port: keď sa kamera pripája k zariadeniu po prvýkrát, Chloros stiahne z kamery cez GigE jej továrenský kalibračný balík s veľkosťou ~3,8 MB (približne 70 sekúnd na jednu kameru). U kamier uložených v cache sa tento panel nikdy nezobrazí. Pozrite si [Pripájanie kamier](connecting.md).

## Šírka pásma: koľko kamier sa zmestí

To, koľko kamier môže pole prenášať, závisí od šírky pásma kábla, nie od Chloros, takže plánovacie údaje nájdete v príručke k hardvéru: **[Plánovanie šírky pásma poľa](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Čo s nimi robí Chloros: dialógové okno pripojenia spustí sieťovú sondu, odhadne dosiahnuteľnú snímkovú frekvenciu a vyberie úroveň, ktorá vyhovuje. Ak pole nadmerne zaťaží linku, odmietne sa pripojiť namiesto toho, aby ticho zahadzovalo pakety — pozri panel s predpokladanými výsledkami opísaný vyššie.

## Keď chýbajú rámce

Kamera môže chýbať v zverejnenej skupine z dvoch úplne odlišných dôvodov,
ktoré si vyžadujú opačné riešenia. Chloros ich počíta oddelene, namiesto toho, aby hlásil jedno
„neúplné“ číslo, ktoré nešpecifikuje ani jeden z nich:

| Čo sa stalo | Čo to znamená | Kde hľadať |
| --- | --- | --- |
| **Poškodený**— rámec dorazil, ale bol štrukturálne poškodený | Strata paketov GVSP na sieťovej trase |**Káblový rozpočet**, prijímací krúžok sieťovej karty, jumbo rámce, prepínač |
| **Nikdy nedorazil**— žiadny rámec vôbec neprišiel | Kamera sa nespustila alebo z nej nič nevyšlo |**Synchronizačný kábel M8**, synchronizačná linka, či sú všetky zariadenia v prevádzkovom režime |

Rozdelenie sa prehodnocuje každých 10 sekúnd počas prenosu dát z pole. Ak presiahne 5 %,
zaznamená sa s uvedením oboch čísel a každý poškodený buffer sa nahlási pri prvom
výskytu na danej kamere, potom sa údaje zoskupujú raz za minútu, aby zostala čitateľnosť aj pri dlhých reláciách.

**Poškodené snímky s nulovým počtom „nikdy nedoručených“ znamenajú, že spúšťanie a synchronizácia kábla sú v poriadku**a každý stratený snímok sa nachádza na sieťovej trase. Riešením je znížiť**Wire Budget** a
znovu sa pripojiť.

{% hint style="warning" %}
**Zníženie frekvencie spúšťania nepomáha pri poškodených snímkach.** Tempo paketov kamery
sa nastaví len raz, pri pripojení. Zníženie frekvencie spúšťania mení to, ako často dochádza k
sériovému prenosu, nie to, akou rýchlosťou sa samotný sériový prenos dostáva do siete. Na meranej zostave so 4 kamerami
zníženie frekvencie spúšťania o 5× nič nezmenilo, zatiaľ čo zníženie šírky pásma z 240 na
200 MB/s znížilo podiel poškodených rámcov v tom istom systéme z 10,4 % na nulu.
{% endhint %}

Spustené pole sa nemôže samo preplánovať — odpojte ho a znovu pripojte, aby nástroj na výber času pripojenia
mohol pracovať s novým limitom.

### Sieťové adaptéry USB sú obmedzené na 200 MB/s

USB ethernetový adaptér uvádza svoju *ethernetovú* prenosovú rýchlosť, ale to, čo skutočne
dokáže udržať, je obmedzené zbernicou USB a jej ovládačom. USB 10GbE dongle sa kedysi pripisovala
priepustnosť približne 1000 MB/s — číslo, ktoré nikto nikdy nemeral — a synchronizácia
štyroch kamier s touto fiktívnou rezervou spôsobila poškodenie 6–18 % snímok, zatiaľ čo pole
stále hlásilo správnu cieľovú snímkovú frekvenciu. Adaptéry pripojené cez USB sú teraz obmedzené na
**200 MB/s**. Toto obmedzenie je absolútne, nie percentuálne, pretože limitom je
zbernica: adaptér USB 1 GbE dosahuje rýchlosť približne 80 MB/s a nie je týmto obmedzením ovplyvnený.

Ak je váš hostiteľ skutočne rýchlejší ako tento limit, zvýšte hodnotu **Wire Budget**, aby to odzrkadľovala.

## Časová synchronizácia PTP

*Synchronizácia* snímok vychádza z hardvérového spúšťača; **PTP** (IEEE 1588 PTPv2) poskytuje porovnateľné *časové značky* vo všetkých zariadeniach. Je to štandardne zapnuté pri pripojení poľa:

* **Backend hostiteľa Chloros spúšťa PTP grandmaster**. Kamery LATTICE a svetelné senzory DAQ-E sú k nemu pripojené ako slave v doméne 0, takže časové značky obrázkov a spektrá DAQ sa zosúladia na jeden takt (~1 ms).
* `--no-ptp` (CLI) túto funkciu deaktivuje pre prácu na pracovnej doske — časové značky medzi kamerami potom **nie sú** porovnateľné.
* Stav synchronizácie skontrolujte pomocou CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Samotná karta „Kamery“ nemá indikátor PTP; v tejto časti sa nachádzajú synchronizačné údaje pre jednotlivé kamery: **Rola**(Master/Slave) určená len na čítanie,**Synchronizačná linka** a úroveň schopností poľa. Stav PTP DAQ-E sa zobrazuje v podrobnostiach senzora na karte „Svetelné senzory“.

## Zobrazenie

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

poľa v reálnom čase Hlavná oblasť zobrazenia ponúka dve rozloženia (prepínanie v hornom paneli): **zobrazenie mriežky**(každá dlaždica je bunka; presúvaním môžete zmeniť poradie, ak je zámok mriežky odomknutý) a**zobrazenie zoznamu**(polia v plnej šírke hore, jedna aktívna kamera pod nimi). Posuvník**Feed Zoom** (Priblíženie prúdu dát) upravuje veľkosť dlaždíc; pri šírke bunky menšej ako 200 px sa prekrývacie texty s názvom a fps automaticky skryjú.**Samostatný režim** zobrazuje jednu dlaždicu na jednu kameru. Každá dlaždica obsahuje:

* názov kamery (vľavo hore),
* **údaj o fps** (vpravo hore) — ide o *skutočnú frekvenciu snímok* kamery hlásenú backendom, nie o frekvenciu aktualizácie náhľadu (živý náhľad je obmedzený na 30 fps bez ohľadu na frekvenciu snímok),
* stavová bodka — zelená (prenos) / oranžová (načítavanie) / červená (chyba),
* **točiaca sa ikona zastaraného snímku**, ak počas 2 s neprišiel žiadny nový snímok — bežné počas ~5 s po každom pripojení/odpojení, kým backend vyvažuje šírku pásma medzi kamerami.**Kombinovaný režim**zobrazuje jedinú zloženú dlaždicu: backend vykonáva debayering, škálovanie, zarovnávanie, odšumovanie, konverziu na žiarivosť v jednotlivých pásmach (plus odrazivosť DLS, ak je pripojený svetelný senzor), vyhodnocuje indexový výraz poľa, aplikuje LUT a výsledok streamuje vo formáte MJPEG. Kým sa nezobrazí prvý zarovnaný snímok, dlaždica informuje o svojom stave: „Príprava poľa…“, „Kalibrácia zarovnania…“, „Čakanie na prvý snímok…“ alebo – ak sa vyčerpá časový limit na automatické opakovanie zarovnania (~30 s) – „Vyžaduje sa zarovnanie“ s tlačidlom**Kalibrovať zarovnanie**.

Užitočné informácie o kombinovanom režime:

* Kompozit je zaregistrovaný na snímku **hlavnej**kamery. Zameriavanie AE-ROI a bodové meranie expozície na kompozite sú presné pre hlavnú kameru a približné pre podriadené kamery; použite**Rozdelený pohľad** (nastavenia poľa → „Zobraziť členské kamery“) pre dlaždice s presnosťou na pixel pre každú kameru bez otvárania ďalších pripojení kamier.
* **Zobraziť vrstvy**(nastavenia poľa; predvolene vypnuté) vám umožňuje vybrať vrstvu v popredí a na pozadí — ľubovoľnú členskú kameru alebo**Index**. Ak je v popredí nastavený**Index**, pixely mimo rozsahu LUT Min/Max zobrazujú vrstvu na pozadí.
* **Rozlíšenie renderovania** (predvolene 720p) nastavuje výšku živého prenosu *a* veľkosť exportovaného kompozitu. Obrazy jednotlivých kamier sa vždy exportujú v plnom rozlíšení.
* Zarovnanie sa počíta pre každú reláciu a nikdy sa neukladá – pozrite si časť o zarovnaní v paneli nastavení poľa, kde nájdete reziduálne hodnoty RMS a tlačidlo „Recalibrate“ (Prekalibrovať).

## Zachytávanie: monitorovanie vs. analýza

Povrchy zachytávania poľa sa jasne delia na **úroveň monitorovania**(zaznamenávanie toho, čo vidíte) a**úroveň analýzy** (zaznamenávanie surových údajov, kalibrácia neskôr):

| Pracovný postup | Úroveň | Čo sa ukladá | Grafické rozhranie | CLI |
| --- | --- | --- | --- | --- |
| **Zaznamenávanie**(snímky) | Analýza | Jedna synchronizovaná skupina snímok na jeden prechod; súbory pre každú kameru na každej vybranej úrovni exportu (surové/odbayerované/radiančné/reflexné/náhľad/index) + `.daq` sidecar |**Zaznamenať všetko** + Nastavenia zaznamenávania | `lattice array-capture` |
| **Nahrávanie indexového videa** | Monitorovanie | Zobrazený živý kompozitný indexový obraz — 8-bitový, rozlíšenie náhľadu, vstavaná LUT; vyžaduje otvorený živý prenos | ● Nahrávanie indexového videa (kombinované polia) | `lattice array-record` |
| **Séria surových snímok → vytvorenie videa**| Analýza | Surové snímky zo senzora pri plnej rýchlosti snímania + manifest + `.daq`, následne offline rekonštrukcia na kalibrované video s indexom žiarivosti / odrazivosti / indexu, časovo zosúladené s hodnotami z DAQ | ⦿ Nahrávanie série surových snímok →**Vytvorenie videa** | `lattice array-burst` → `lattice array-build-video` |

Pravidlo: ak majú pixely slúžiť na *merania*, použite zachytenie alebo sériu snímok (na účely analýzy); ak potrebujete len *pozerať sa alebo predviesť*, čo snímač zaznamenal, nahrajte indexové video (na účely monitorovania).

### Nastavenia snímania (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

Ikona ozubeného kolesa vedľa položky **Capture All** otvorí panel Nastavenia snímania (vyžaduje otvorený projekt — snímky sa ukladajú do neho):

* **Režim snímania**:**Single**(jeden prechod) /**Continuous**(nepretržite; obmedzené počtom záznamov, predvolené 1, alebo trvaním, predvolené 10 s) /**Interval** (časozber: N záznamov každých X intervalov, celkom Y, predvolené 1 každých 5 s počas 1 minúty).
* **Typy exportu na kameru**: Raw, Debayered, Radiance, Reflectance, Preview, Index — všetko, čo je použiteľné, je predvolene zapnuté. Radiance/Reflectance sú skryté pre kamery s filtrom RGB;**Reflectance sa zobrazuje len vtedy, ak má kamera svetelný senzor DAQ** (vlastný alebo zdedený z poľa); Index vyžaduje nakonfigurovaný indexový výraz.
* **Vyrovnané**(pre pole, predvolene**zapnuté**): deformuje exporty členov podľa vyrovnávacieho profilu poľa, aby boli exporty pixelovo zarovnané. Formát Raw zostáva vždy nedeformovaný, ale transformáciu nesie v metadátach.
* **Najrýchlejšie snímanie** (prepínač): iba surové dáta + priradené čítanie DAQ + bezplatný kompozit kombinovaného indexu, pričom sa pri snímaní preskočí kalibračná matematika pre dosiahnutie maximálnej rýchlosti — žiarivosť/odrazivosť/index sa neskôr znovu vypočítajú z uloženého súboru `.daq`.
* Výbery zostávajú zachované v projekte. Skryté alebo pozastavené kamery sa preskakujú.

Ekvivalentný súbor CLI (rovnaký koncový bod backendu, rovnaká sémantika):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

Kompresia záznamov pre TIFF je `deflate` (bezstratová, predvolená) alebo `none` — úplné tabuľky príznakov, štruktúra priečinka so záznamami a pravidlá opätovného spracovania sú uvedené v [Referencii CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Párovanie svetelného senzora DAQ

Náhľady s korekciou odrazivosti a osvetlenia vyžadujú údaje o dopadajúcom svetle zo senzora DAQ (pripojeného na karte **Svetelné senzory**):

* **Riadok poľa**na bočnom paneli zobrazuje**políčko **&quot;DAQ · zapnutý/vypnutý“** — *zapnutý*, ak je nastavený svetelný senzor na úrovni poľa **alebo** ak má niektorá z kamier v poli vlastný senzor; v popise sa uvádza, ktorý senzor napája ktorú kameru.
* Priradite nastavenie pre celé pole v nastaveniach poľa → **Snímač okolitého osvetlenia**→ roletové menu**Svetelný snímač**. Výber zostáva zachovaný v rámci projektu, prenáša sa na každú kameru v poli a jednotlivé kamery ho môžu aj naďalej prepisovať svojím vlastným snímačom.
* Stavový riadok pod ním hlási aktuálny stav: **Vypnuté**→ „Čaká sa na prvé spektrum…“ →**„Aktívne — všetky kamery v poli majú korigované osvetlenie“** → alebo, ak za posledné 3 s neprišlo žiadne nové spektrum, upozornenie na neaktuálne údaje — naďalej sa používa posledná hodnota (hodnoty v ceste snímania nikdy nevypršia).

Ak je priradený senzor: sprístupní sa typ exportu „Reflectance“, živé náhľady sú korigované na osvetlenie, prediktívna automatická expozícia môže využívať spektrum a každý záznam odrazivosti zapíše skutočne použitú hodnotu DAQ ako **`.daq` sidecar** vedľa snímky, aby bolo možné zachytenie neskôr opätovne spracovať.

## `array-connect` CLI možnosti

| Príznak | Predvolené | Popis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automaticky vyhľadať všetky kamery LATTICE (vyžaduje ≥2) | **Prvá sériová kamera je MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Synchronizačná linka GPIO. |
| `--target-fps F` | auto | Frekvencia spúšťania hlavnej kamery. |
| `--binning {1,2,4}` | auto | Hardvérové zoskupovanie. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Prepisovanie výberu synchronizačnej úrovne expertom. |
| `--wire-ceiling-mbps MB_PER_S` | automaticky detekované | Rozpočet hostiteľa pre prenos v MB/s — forma poľa **Wire Budget** v CLI. Znížte ho, ak pole hlási poškodené rámce. Ukladá sa spolu s projektom, takže pri neskoršom opätovnom pripojení sa obnoví. |
| `--no-recommend` | vypnuté | Preskočiť krok analýzy siete. |
| `--no-ptp` | vypnuté | Deaktivovať PTP (časové značky medzi kamerami potom nie sú porovnateľné). |

`lattice array-list`, `array-status` a `array-disconnect` spravujú trvalú reláciu. Úplný zoznam podpríkazov, vrátane zarovnania (`align-calibrate` / `align-apply`) a sieťových nástrojov, sa nachádza v [CLI Referencia § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); ekvivalenty SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) sa nachádzajú v [SDK Referencia](../reference/sdk-reference.md). Od Python je rozpočet na vedenie `connect_array(..., wire_ceiling_mbps=120)` a rozdelenie medzi poškodené a nedoručené sa nachádza na [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).

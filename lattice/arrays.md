# Systémy s viacerými kamerami

**Systém**LATTICE pozostáva z dvoch alebo viacerých kamier LATTICE prepojených do jednej synchronizovanej jednotky. Jedna kamera je**hlavná**: vysiela hardvérový spúšťací impulz GPIO na zdieľanú synchronizačnú linku (predvolene**Line2**), takže každý člen sníma v rovnakom okamihu. Chloros

pridáva časovú synchronizáciu PTP, živý náhľad (mozaiky jednotlivých kamier alebo jediný zarovnaný viacpásmový kompozit) a synchronizované snímanie — každý snímací cyklus vytvára jednu **skupinu snímok**, v ktorej všetky kamery zdieľajú rovnaký časový údaj a ID snímky (uvádzané ako `fid:N` vo výstupe snímania).

Polia sú spôsobom, akým monochromatické (M3M) kamery vytvárajú vegetačné indexy – jedna kamera prispieva jedným pásmom a pole ich zarovnáva do viacpásmového súboru. Pozrite si [Mono kamery a vegetačné indexy](mono-indices.md).

Existujú tri rovnocenné spôsoby pripojenia poľa a všetky spúšťajú rovnaký postup „smart-prep“:

| Rozhranie | Vstupný bod |
| --- | --- |
| GUI | Karta Kamery → **Pripojiť pole** (modré tlačidlo) |
| CLI

| `chloros-cli lattice array-connect --serials SN1,SN2,…` (prvé sériové číslo = hlavné) |
| Python

SDK

| `connect_array(serials=[…])` → `ArraySession` (prvé sériové číslo = hlavné zariadenie) |

Smart-prep vykonáva: test sieťovej dostupnosti (ICMP DF ping + GVSP test), výber synchronizačnej úrovne, odhad snímkovej frekvencie pri požadovanej veľkosti snímky, rozhodnutie o komprimovanom 12-bitovom prenose (pozri [Zbalený 12-bitový prenos](#packed-12-bit-transport)), aktiváciu PTP, automatický výber formátu pixelov pre každú kameru, nastavenie počiatočnej hodnoty automatického nastavenia expozície na základe uloženého stavu každej kamery a konfiguráciu spúšťača GPIO na linke 2.

**V predvolenom nastavení sa používa celý senzor (novinka vo verzii 1.3.0).** Connect zachováva veľkosť snímky, ktorú ste požadovali — celý snímač s rozlíšením 2048×1536, bez zlučovania a bez orezávania, pokiaľ si nezvolíte inak — a nechá snímkovú frekvenciu klesnúť na úroveň, ktorú poskytuje hostiteľ a sieť. Nikdy sám od seba nezlučuje ani neorezáva (verzia 1.2.0 automaticky aktivovala zlučovanie pri frekvencii nižšej ako 1,5 fps). Zlučovanie alebo menšia oblasť záujmu (ROI) je na vás: v dialógovom okne Connect si vyberte jednu z možností a v časti „Predpokladaný výsledok“ sa zobrazí snímková frekvencia, ktorú táto voľba poskytuje; `array-connect` vypíše rýchlejšie možnosti spolu s ich predpokladanými frekvenciami.

{% hint style="info" %}
Aby toto všetko fungovalo, kamery musia byť dostupné cez sieťové pripojenie — pozrite si [Pripojenie kamier](connecting.md) pre vyhľadávanie, adresovanie a stiahnutie kalibrácie pri prvom pripojení. Pri zostavách s viacerými kamerami sú nastavenia prijímacieho kruhu hostiteľskej sieťovej karty rovnako dôležité ako rýchlosť spojenia; kompletná tabuľka príznakov a riešení sa nachádza v [Referencia CLI

§ Nastavenie a ladenie hostiteľskej sieťovej karty](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dialógové okno „Pripojenie poľa“

Karta Kamery → **Pripojiť pole**otvorí trojkrokový sprievodca:**Vybrať → Režim zobrazenia → Nastavenia**.

### Krok 1 — Výber hlavnej a podriadených kamier



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->

Hneď po otvorení dialógové okno prehľadá sieť („Prehľadávanie siete...“) a potom skontroluje synchronizačné káble („Kontrola synchronizačných káblov...“), aby zistilo, ktoré kamery sú pripojené na synchronizačnú linku M8. Na vytvorenie poľa potrebujete aspoň **2 kamery**.

Synchronizačné káble M8 sa spájajú v jednom uzle, takže **hlavnou kamerou môže byť akákoľvek pripojená kamera**. Testovací program vopred priradí úlohy — kamera RGB

(`-FRGB`) ako hlavná, ak je pripojená káblom, inak prvá pripojená kamera a ostatné pripojené kamery ako podriadené — a zobrazí jeden z troch nápisov:

| Nápis | Význam |
| --- | --- |
| „Na {n} kamerách bol nájdený synchronizačný kábel. Akákoľvek z nich môže byť hlavnou; predvolená je RGB

kamera (inak prvá).“ (zelená) | Sonda našla kamery na synchronizačnej linke; políčka pre hlavnú a podriadenú kameru sú už zaškrtnuté. |
| „Hlavná kamera nebola zistená – skontrolujte kábel GPIO“ (oranžová) | Žiadna kamera nezaznamenala spúšťací impulz; skontrolujte synchronizačné káblové pripojenie. Úlohy sú stále predvolené (RGB

kamera, inak prvá kamera v zozname, ako hlavná) a môžete ich zmeniť. |
| „Chýba synchronizačný kábel: {sériové čísla}“ (oranžová) | Uvedené kamery nemajú pripojený synchronizačný kábel. |

Tabuľka kamier obsahuje stĺpce **Kamera / Sériové číslo / IP / Hlavná (rádio) / Podriadená (zaškrtávacie políčko)**. Riadky sú zoradené podľa**poradia kamier** — kamery M3C (Bayer) pred kamerami M3M (mono); medzi kamerami M3C, FRGB, FRGN, FOCN a potom FNGB; kamery M3M podľa vlnovej dĺžky filtra, najnižšia ako prvá (F450 pred F808). Zoznam kamier v bočnom paneli a predvolené rozloženie mriežky používajú rovnaké poradie.

* Vyberte presne **jedného hlavného**a**jedného alebo viacerých podriadených**. Opätovným kliknutím na rádio aktuálneho hlavného sa jeho výber zruší. Výber iného hlavného sa vykonáva jedným kliknutím: nový hlavný opustí podriadených a predchádzajúci hlavný sa stane podriadeným (pokiaľ nemá synchronizačný kábel).
* Kameru označenú ako **„Bez synchronizačného kábla“** nemožno nikdy vybrať ako podriadenú – podriadená kamera bez spúšťacieho zapojenia by navždy čakala na synchronizačnom vedení a poskytovala by nefunkčný obraz. Túto kameru namiesto toho pripojte ako samostatnú kameru.
* Kamery, ktoré sú už pripojené samostatne, *nie* sú deaktivované: pripojenie do poľa uvoľní samostatnú reláciu a opäť otvorí kameru v rámci poľa.

**Ďalej: Režim zobrazenia →**sa aktivuje po výbere hlavnej kamery a aspoň jednej podriadenej kamery.**Opätovné vyhľadávanie** znovu spustí vyhľadávanie a testovanie zapojenia.

{% hint style="warning" %}
**Zrušiť** je deaktivované počas prebiehajúceho vyhľadávania alebo testovania zapojenia — zrušenie uprostred testovania môže spôsobiť zlyhanie kamerového SDK

vo firmvéri kamery LATTICE. Počkajte, kým sa dokončí indikátor prebiehajúcej operácie.
{% endhint %}

### Krok 2 — Režim zobrazenia



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Režim | Čo získate |
| --- | --- |
| **Samostatné kamery** | Jedna živá dlaždica na kameru, všetky sa spúšťajú súčasne, aby snímky zostali synchronizované. Každá kamera si zachováva vlastnú farbu a nastavenia. |
| **Kombinované kamery** *(predvolené)* | Jedna dlaždica zobrazujúca zarovnaný viacpásmový kompozitný obraz NDVI

/index. Kamery zdieľajú farbu poľa. |

Režim zobrazenia mení iba prezentáciu živého náhľadu — správanie pri snímaní je v oboch prípadoch rovnaké.

### Krok 3 — Nastavenia poľa a predpokladaný výsledok

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->



Pri vstupe do tejto scény si Chloros

vyžiada od backendu **odporúčanie**a uplatní ho:**plný senzor — bez zlučovania, bez orezávania ROI — vždy, keď to umožňuje akákoľvek snímková frekvencia**, bez ohľadu na rýchlosť, ktorú poskytuje hostiteľ a prenosové médium. Iba v prípade, že žiadna rýchlosť nedokáže poskytnúť plný senzor, odporúčanie zmenší snímku, a v takom prípade uprednostňuje zlučovanie pred orezávaním ROI, pretože zlučovanie zachováva celé zorné pole. Od tohto momentu sa rozhodujete medzi rozlíšením a snímkovou frekvenciou. Každá zmena, ktorú vykonáte, spustí analýzu v reálnom čase a aktualizuje panel**Predpokladaný výsledok** na pravej strane.

Ľavý stĺpec — nastavenia:

| Ovládací prvok | Možnosti | Predvolené | Poznámky |
| --- | --- | --- | --- |
| **ROI (zorné pole)** | Plné (2048×1536) / Polovičné (1024×768) / Štvrtinové (512×384) | Plné | Orezanie snímača: Polovičné/štvrtinové orezanie na menšiu oblasť pri natívnej hustote pixelov. |
| **Binning** | 1× / 2× (súčet 2×2) / 4× (súčet 4×4) | 1× | Hardvérový binning: 2×2 = plné zorné pole pri štvrtine nákladov na prenos; 4×4 = plné zorné pole pri 1/16. Skryté, ak kamery nepodporujú binning. |
| **Obraz na strane vedenia** (čítanie) | — | — | Šírka × výška po binningu, ktorá sa skutočne odosiela po vedení, zaokrúhlená na násobky 16 (minimálne 64). |
| **Frekvencia spúšťania** | 0,5–60 fps, krok 0,1 | prázdne = auto | Frekvencia spúšťania hlavného zariadenia. Nechajte prázdne, aby to Chloros

mohol odvodiť. |
| **Kapacita prenosu**| 20–2000 MB/s, krok 10 | prázdne = auto | Koľko môže hostiteľ skutočne prijať, v MB/s —**jediné číslo, od ktorého závisí celé prideľovanie poľa.** Automaticky zistené zo sieťového adaptéra. Znížte ho, ak pole hlási poškodené rámce: zistená hodnota nadhodnocuje USB adaptéry a zdieľané prepínače. Zmenou sa znovu spustí živá projekcia. Pole sa otvorí s limitom, s ktorým bolo naposledy pripojené pole (uložené s projektom), takže limit, ktorý ste raz upravili, zostane zachovaný. |

Zaškrtávacie políčko **Rozlíšenie pinov** z verzie 1.2.0 bolo odstránené: pri pripájaní sa vždy zachová veľkosť rámca, ktorú zvolíte, a namiesto toho sa zníži frekvencia rámcov.

Pravý stĺpec — **Predpokladaný výsledok**:

* **Úroveň synchronizácie** — „Súbežné snímanie“ (zelená), „Súbežné snímanie (FTD – postupné vysielanie)“ (zelená), „Postupné snímanie (100 ms posun)“ (oranžová) alebo „Konfigurácia je príliš veľká“ (červená). Chloros

vyberá úroveň: simultánne snímanie vždy, keď je možné zdieľať linku rýchlosťou, na ktorú sa dajú prispôsobiť kamery (snímaná frekvencia klesá, keď sa zmenšuje podiel každej kamery), a striedavé snímanie len vtedy, keď to nie je možné — takže pole sa pripojí na akúkoľvek sieť pri snímanej frekvencii, ktorú táto sieť umožňuje.
* **Projekcia fps** — snímková frekvencia, pri ktorej by mal systém bežať. Pri simultánnom snímaní ide o jedno číslo: najprv sa určí rýchlosť a automatická expozícia vyplní časový interval, ktorý táto rýchlosť poskytuje, a potom zvýši zosilnenie, aby ju neoslabila tmavá scéna (pozri [Ako automatická expozícia rozdeľuje expozíciu a zosilnenie](camera-settings.md)). Pri postupnom snímaní sa zobrazuje ako rozsah („tmavé → svetlé“), pretože expozícia každej kamery predlžuje časový posun v tmavej scéne.
* **Linka NIC** — rýchlosť spojenia a trvalá priepustnosť („NIC {mbps} Mbps · trvalá {N} MB/s“).
* **Kontrola simultánneho vysielania** — dokáže prijímací kruh sieťovej karty hostiteľa prijať jeden simultánny burst zo všetkých kamier („Simultánne vysielanie: X MB · využiteľný kruh sieťovej karty: Y MB ✓/✗“).
* **Kontrola kapacity vedenia** — celkový dopyt v ustálenom stave vs. strop kapacity vedenia zabezpečujúci ochranu pred kolíziami („Kapacita vedenia: {dopyt} MB/s požadovaný {n} kamerami · strop {strop} MB/s ✓/✗ prekročenie kapacity“).
* **„Maximálny počet kamier na tomto kanáli: {n} — nastavený minimálnou šírkou pásma na kameru, takže zlučovanie tento počet nezvýši.“** — zobrazí sa, keď sa blížiš k (alebo prekročíš) strop počtu kamier.
* **„Pri týchto nastaveniach DOJDE K VYNECHÁVANIU SÁZOK.“**— červené varovanie s dôvodom zo strany backendu, plus zoznam prekážok a modré**návrhy na opravu** („Aby sa táto matica zmestila do siete“ / „Aby sa odblokovalo simultánne snímanie“).

Tlačidlo **Použiť a pripojiť** je zablokované, kým neexistuje prognóza, a jeho popisok vám vysvetlí, prečo je odmietnuté:

| Názov tlačidla | Význam | Čo skutočne pomôže |
| --- | --- | --- |
| „Analyzujem...“ | Analýza stále prebieha. | Počkajte. |
| **„Príliš veľa kamier pre túto sieť“**| Sieť nedokáže prenášať pole ani po jednej kamere naraz (zlyhala kontrola agregácie): je oveľa viac kamier, než dokáže spracovať, alebo na nej streamujú iné kamery. | Menej kamier, jumbo rámce end-to-end alebo rýchlejšia sieťová karta.**Menšia oblasť záujmu (ROI) NEPOMÔŽE** — pozri nižšie. |
| **„Zmenšite oblasť záujmu (ROI), aby sa aktivovala“** | Pri týchto nastaveniach by dochádzalo k vypadávaniu snímok (kontrola burst/ring zlyhala). | Zmenšite oblasť záujmu (ROI), zvýšte binning alebo opravte prijímací ring sieťovej karty. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



Počas pripájania sa môže zobraziť zelený **panel sťahovania kalibrácie** s ukazovateľom priebehu pre každé sériové číslo: keď sa kamera pripája k zariadeniu po prvýkrát, Chloros

stiahne z kamery cez GigE jej továrenský kalibračný balík s veľkosťou ~3,8 MB (približne 70 sekúnd na kameru). U kamier uložených v cache sa tento panel nikdy nezobrazí; pri viac ako šiestich kamerách sa jeho riadky posúvajú. Pozrite si [Pripájanie kamier](connecting.md).

### Komprimovaný 12-bitový prenos

12-bitové formáty (`BayerRG12`, `Mono12`) sa zvyčajne prenášajú ako 16-bitové slová. **Komprimovaný**prenos odosiela tých istých 12 bitov v 1,5 bajte na pixel a dekomprimuje ich na hostiteľskom zariadení, takže odosielanie každého snímku trvá o 25 % menej času — pri plnom využití senzora to znamená približne o tretinu viac snímok za sekundu.**Uložené údaje sú identické.** Režim „array connect“ to rozhoduje automaticky (`auto`): komprimuje len vtedy, keď to pomáha a hostiteľ má voľný výpočtový výkon na dekomprimáciu. Grafické rozhranie (GUI) nemá na to žiadne prepínanie; z príkazového riadku (CLI

) použite `array-connect --packed {auto,on,off}` (predvolené nastavenie je `auto`). 8-bitové a 16-bitové formáty sa nikdy nekomprimujú.

## Šírka pásma: koľko kamier sa zmestí

To, koľko môže pole prenášať, závisí od šírky pásma siete, nie od Chloros

u, takže plánovacie údaje nájdete v príručke k hardvéru: **[Plánovanie šírky pásma poľa](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Ako s nimi nChloros

: dialógové okno pripojenia spustí sieťovú sondu, odhadne dosiahnuteľnú snímkovú frekvenciu a vyberie úroveň, ktorá vyhovuje. Pole sa pripojí v akejkoľvek sieti: na pomalom spojení sa každá kamera prispôsobí svojmu podielu na linke a snímková frekvencia klesne, namiesto toho, aby bolo pole odmietnuté alebo aby sa zmenšili jeho snímky. Odmietnuté je len pole, ktorého pripojenie nedokáže prenášať ani jednu kameru naraz (**Príliš veľa kamier pre túto sieť**), namiesto toho, aby sa pakety ticho zahadzovali — pozri panel s predpokladanými výsledkami opísaný vyššie.

## Keď chýbajú snímky

Kamera môže chýbať v zverejnenej skupine z dvoch úplne odlišných dôvodov,
ktoré si vyžadujú opačné riešenia. Chloros

ich počíta oddelene, namiesto toho, aby uvádzal jedno
„neúplné“ číslo, ktoré nešpecifikuje ani jeden z nich:

| Čo sa stalo | Čo to znamená | Kde hľadať |
| --- | --- | --- |
| **Poškodený**— rámec dorazil, ale bol štrukturálne poškodený | Strata paketov GVSP na sieťovej trase |**Kapacita linky**, prijímací krúžok sieťovej karty, jumbo rámce, prepínač |
| **Nikdy nedorazil**— žiadny rámec vôbec neprišiel | Kamera sa nespustila alebo z nej nič nevyšlo |**Synchronizačný kábel M8**, synchronizačná linka, či sú všetky zariadenia v prevádzkovom režime |

Rozdelenie sa prehodnocuje každých 10 sekúnd počas prenosu z pole. Ak presiahne 5 %,
zaznamená sa s uvedením oboch čísel a každý poškodený buffer sa nahlási pri prvom
výskytu na danej kamere, potom sa údaje zosumarizujú raz za minútu, aby zostala čitateľnosť aj pri dlhých reláciách.

**Poškodené snímky s nulovým počtom „nikdy nedoručených“ znamenajú, že spúšťanie a synchronizácia kábla sú v poriadku**a každý stratený snímok sa nachádza na sieťovej trase. Riešením je znížiť**Wire Budget** a
znova sa pripojiť.

{% hint style="warning" %}
**Zníženie frekvencie spúšťania nepomáha pri poškodených snímkach.** Tempo paketov kamery
sa nastaví jednorazovo pri pripojení. Zníženie frekvencie spúšťania mení to, ako často dochádza k sérii
snímok, nie to, ako rýchlo sa samotná séria prenáša po kábli. Na testovanej zostave so 4 kamerami
5-násobné zníženie frekvencie spúšťania nič nezmenilo, zatiaľ čo zníženie wire budgetu z 240 na
200 MB/s znížilo podiel poškodených snímok v tej istej zostave z 10,4 % na nulu.
{% endhint %}

Bežiaca zostava sa nemôže sama preplánovať — odpojte ju a znovu pripojte, aby nástroj na výber času pripojenia
mohol pracovať s novým limitom.

### USB sieťové adaptéry sú obmedzené

USB ethernetový adaptér uvádza svoju *ethernetovú* prenosovú rýchlosť, ale to, čo skutočne
dokáže udržať, je obmedzené zbernicou USB a jej ovládačom. USB 10GbE kľúču sa kedysi pripisovala
priepustnosť približne 1000 MB/s – čo je hodnota, ktorú nikto nikdy nemeral – a synchronizácia
štyroch kamier s touto fiktívnou rezervou spôsobila poškodenie 6–18 % snímok, zatiaľ čo pole
stále hlásilo správnu cieľovú snímkovú frekvenciu. Adaptéry pripojené cez USB sú teraz obmedzené na
**200 MB/s** v systéme Windows

a na platforme NVIDIA Jetson a na **400 MB/s** v systéme Linux

x86_64. Toto obmedzenie je absolútne, nie percentuálne, pretože limitom je samotná
zbernica: adaptér USB 1 GbE dosahuje rýchlosť približne 80 MB/s a toto obmedzenie sa ho netýka.

Ak je váš hostiteľ skutočne rýchlejší ako tento limit, zvýšte hodnotu **Wire Budget**, aby to odzrkadľovala.

## Časová synchronizácia PTP

*Synchronizácia* snímok vychádza z hardvérového spúšťača; **PTP** (IEEE 1588 PTPv2) poskytuje porovnateľné *časové značky* na všetkých zariadeniach. Je povolená štandardne pri pripojení poľa:

* **Backend hostiteľa Chloros

spúšťa PTP grandmaster**. Kamery LATTICE a svetelné senzory DAQ-E sa k nemu pripájajú ako slave v doméne 0, takže časové značky snímok a spektrá DAQ sa zosúladia na jeden takt (~1 ms).
* `--no-ptp` (CLI

) to deaktivuje pre prácu na pracovnom stole — časové značky medzi kamerami potom **nie sú** porovnateľné.
* Skontrolujte stav synchronizácie pomocou CLI

:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Samotná karta „Kamery“ nemá indikátor PTP; synchronizačné údaje pre jednotlivé kamery sú tam uvedené v poliach **Úloha**(Master/Slave),**Synchronizačná linka** a úroveň schopností poľa, ktoré sú určené len na čítanie. Stav PTP DAQ-E sa zobrazuje v podrobnostiach senzorov na karte „Svetelné senzory“.

## Zobrazenie

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

poľa v reálnom čase Hlavná oblasť zobrazenia ponúka dve rozloženia (prepínanie v hornom paneli): **mriežkové zobrazenie**(každá dlaždica je bunka; presúvaním môžete zmeniť poradie, ak je zámok mriežky odomknutý) a**zobrazenie zoznamu**(riadky v plnej šírke:**kombinované**pole tvorí jeden riadok – jeho zložená dlaždica –, zatiaľ čo**samostatné**pole priraďuje**každému členovi vlastný riadok**s vlastným živým prenosom a vlastnou tabuľkou kalibračných cieľov; pod poliami je jedna samostatná kamera — tá aktívna). Posuvník**Zoom kanála** mení veľkosť dlaždíc; pri šírke bunky menšej ako 200 px sa prekrývacie vrstvy s názvom a fps automaticky skryjú. Zložkové kamery sa zobrazujú v bočnom paneli a vo výchozom rozložení mriežky v poradí kamier (M3C pred M3M, najskôr FRGB, monokromátické podľa vlnovej dĺžky); odomknutie mriežky a ťahanie však aj naďalej prepisuje poradie v mriežke.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



**Samostatný režim** zobrazuje jednu dlaždicu na jednu kameru. Každá dlaždica obsahuje:

* názov kamery (vľavo hore),
* **údaj o fps** (vpravo hore) — ide o *skutočnú frekvenciu snímok* kamery hlásenú backendom, nie o frekvenciu aktualizácie náhľadu (živý náhľad je obmedzený na 30 fps bez ohľadu na frekvenciu snímok),
* stavovú bodku — zelená (prenos) / oranžová (načítavanie) / červená (chyba),
* **otočný indikátor zastaraného snímku**, ak počas 2 s neprišiel žiadny nový snímok — bežné počas ~5 s po akomkoľvek pripojení/odpojení, kým backend vyvažuje šírku pásma medzi kamerami.**Kombinovaný režim**zobrazuje jedinú zloženú dlaždicu: backend vykonáva debayering, zmenšuje, zarovnáva, odstraňuje šum, prevádza na žiarivosť v jednotlivých pásmach (plus odrazivosť DLS, ak je pripojený svetelný senzor), vyhodnocuje indexový výraz poľa, aplikuje LUT a výsledok streamuje vo formáte MJPEG. Kým sa nezobrazí prvý zarovnaný snímok, dlaždica informuje o svojom stave: „Pripravuje sa pole…“, „Kalibruje sa zarovnanie…“, „Čaká sa na prvý snímok…“ alebo – ak sa vyčerpajú pokusy o automatické zarovnanie dlaždice (asi jedna minúta) – „Vyžaduje sa zarovnanie“ s tlačidlom**Kalibrovať zarovnanie**.

Hodnota fps na dlaždici poľa tiež informuje o stave prenosu:**„Aplikujú sa nastavenia…“**počas reštartovania kamier v dôsledku zmeny nastavení (formát, zlučovanie, ROI),**„Zastavené“**, keď je pole aktívne, ale prestalo dodávať skupiny snímok, a**„Zastavené“**, keď prestala vysielať – v takom prípade pole znovu pripojte. Ak spektrálne oddelenie muselo obmedziť pásmo zobrazené v kompozícii na veľkej časti jej pixelov (25 % alebo viac), dlaždica zobrazí**„Údaje pásma s nízkou spoľahlivosťou · N %“**: indexové hodnoty založené na tomto pásme sú sporné (zvyčajne jedno pásmo prijalo príliš málo svetla).

Užitočné informácie o kombinovanom režime:

* Kompozit je zarovnaný s snímkou **hlavnej**kamery. Zameriavanie AE-ROI a bodové meranie expozície na kompozite sú presné pre hlavnú kameru a približné pre podriadené kamery; použite**Rozdelené zobrazenie** (nastavenia sústavy → „Zobraziť členské kamery“) pre dlaždice s pixelovou presnosťou pre každú kameru bez otvárania ďalších pripojení kamier.
* **Zobrazenie vrstiev**(nastavenia poľa; predvolene vypnuté) vám umožňuje vybrať vrstvu v popredí a na pozadí — ľubovoľnú členskú kameru alebo**Index**. Ak je v popredí nastavený**Index**, pixely mimo rozsahu LUT Min/Max zobrazujú vrstvu na pozadí.
* **Rozlíšenie renderovania** (predvolene 720p) nastavuje výšku živého prenosu *a* veľkosť exportovaného kompozitu. Obrázky jednotlivých kamier sa vždy exportujú v plnom rozlíšení.
* Zarovnanie sa počíta pre každú reláciu a nikdy sa neukladá. Chloros

automaticky zarovná **každú maticu po pripojení, v oboch režimoch zobrazenia**, akonáhle sa ustáli automatická expozícia členov (pokúša sa o to až 5 minút). Sekcia**Vyrovnanie**v paneli nastavení poľa zobrazuje reziduálne hodnoty RMS a tlačidlo**Prekalibrovať vyrovnanie** — aj pre samostatné polia, ktoré nemajú kompozitnú dlaždicu.

## Zachytávanie: monitorovanie vs. analýza

Povrchy snímania polí sa jasne delia na **úroveň monitorovania**(zaznamenávanie toho, čo vidíte) a**úroveň analýzy** (zaznamenávanie surových údajov, kalibrácia neskôr):

| Pracovný postup | Úroveň | Čo sa ukladá | GUI | CLI

|
| --- | --- | --- | --- | --- |
| **Zaznamenávanie**(snímky) | Analýza | Jedna synchronizovaná skupina snímok na jeden prechod; súbory pre každú kameru na každej vybranej úrovni exportu (surové/bez debayeringu/žiarivosť/odrazivosť/náhľad/index) + sidecar `.daq` |**Záznam všetkého** + Nastavenia záznamu | `lattice array-capture` |
| **Záznam indexového videa** | Monitorovanie | Zobrazený živý kompozitný indexový obraz — 8-bitový, rozlíšenie náhľadu, vstavaná LUT. Z CLI

bez otvoreného grafického rozhrania backend vykreslí NDVI

(alebo `--expr`) po zarovnaní poľa; nahrávanie, pri ktorom sa nezapíše žiadny snímok, zlyhá | ● Nahrávanie indexového videa (kombinované polia) | `lattice array-record` |
| **Surová séria → vytvorenie videa**| Analýza | Surové snímky zo senzora pri plnej rýchlosti snímania + manifest + `.daq`, následne offline rekonštrukcia na kalibrované video s radianciou / odrazivosťou / indexom, časovo zosúladené s hodnotami z DAQ | ⦿ Nahrávanie surovej série →**Vytvorenie videa** | `lattice array-burst` → `lattice array-build-video` |

Pravidlo: ak majú pixely slúžiť na *merania*, použite zachytenie alebo sériu snímok (na účely analýzy); ak potrebujete len *pozerať sa alebo predviesť*, čo snímač zaznamenal, nahrajte indexové video (na účely monitorovania).

### Nastavenia zachytávania (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



Ozubené koliesko vedľa položky **Zachytiť všetko** otvorí panel Nastavenia zachytávania (vyžaduje otvorený projekt — záznamy sa ukladajú do neho):

* **Režim snímania**:**Jednorazové**(jeden prechod) /**Neprerušované**(za sebou; obmedzené počtom snímok, predvolené 1, alebo trvaním, predvolené 10 s) /**Interval** (časozber: N záznamov každých X intervalov, celkom Y; predvolené nastavenie je 1 každých 5 s počas 1 minúty).
* **Typy exportu na kameru**: Raw, Debayered, Radiance, Reflectance, Preview, Index — všetko, čo je použiteľné, je predvolene zapnuté. Radiance/Reflectance sú skryté pre kamery s RGB

filtrom; **Reflectance sa zobrazuje len vtedy, ak má kamera svetelný senzor DAQ** (vlastný alebo zdedený z poľa); Index vyžaduje nakonfigurovaný indexový výraz.
* **Aligned**(pre pole, predvolene**zapnuté**): deformuje exporty členov podľa profilu zarovnania poľa, takže exporty sú pixelovo zarovnané. Surové dáta zostávajú vždy nepremenené, ale transformáciu nesú v metadátach. Súbory členov poľa sa ukladajú v natívnej orientácii každej kamery (akékoľvek nastavenie zrkadlenia/rotácie sa zaznamenáva do súboru, ale neuplatňuje sa).
* **Najrýchlejšie snímanie** (prepínač): iba surové dáta + priradená hodnota DAQ + bezplatný kompozit s kombinovaným indexom, pričom sa pri snímaní preskakujú kalibračné výpočty pre dosiahnutie maximálnej rýchlosti — žiarivosť/odrazivosť/index sa neskôr obnovia z uloženého súboru `.daq`.
* Výbery zostávajú zachované v projekte. Skryté alebo pozastavené kamery sa preskočia.
* **Neprerušované** snímanie na poliach prebieha vo vnútri backendu Chloros

(novinka vo verzii 1.3.0): keď každá kamera v zozname patrí do poľa, backend spúšťa snímaciu slučku sám, zapisuje jednu skupinu a zároveň vyberá ďalšiu, bez požiadavky na každé snímanie. Plný disk zastaví beh („Zastavené: disk plný. Uložené N, F zlyhalo“). Pozrite si [Nastavenia a režimy zachytávania](capture.md#capture-modes), kde nájdete poznámku k programu Microsoft Defender týkajúcu sa vysokých rýchlostí zachytávania.

Ekvivalentný príkaz CLI

(rovnaký koncový bod backendu, rovnaká sémantika):

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

Kompresia TIFF

u pre zachytenia je `deflate` (bezstratová, predvolená) alebo `none` — úplné tabuľky príznakov, štruktúra priečinka zachytení a pravidlá opätovného spracovania sú uvedené v [Referencii CLI

](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Párovanie svetelného senzora DAQ

Náhľady s korekciou odrazivosti a osvetlenia vyžadujú údaje o dopadajúcom svetle zo senzora DAQ (pripojeného na karte **Svetelné senzory**):

* V **riadku poľa**na bočnom paneli sa zobrazuje**políčko „DAQ · zapnuté/vypnuté“** — *zapnuté*, ak je nastavený svetelný senzor na úrovni poľa **alebo** ak má niektorá z kamier v poli vlastný senzor; v jeho popise sa uvádza, ktorý senzor zásobuje signálom ktorú kameru.
* Priraďte nastavenie pre celé pole v nastaveniach poľa → **Senzor okolitého osvetlenia**→ roletové menu**Svetelný senzor**. Výber zostáva zachovaný v rámci projektu, prenáša sa na každú kameru v poli a jednotlivé kamery ho môžu aj naďalej prepisovať svojím vlastným senzorom.
* Stavový riadok pod ním informuje o aktuálnom stave: **Vypnuté**→ „Čaká sa na prvé spektrum…“ →**„Aktívne — všetky kamery v poli majú korigované osvetlenie“** → alebo, ak za posledné 3 s neprišlo žiadne nové spektrum, upozornenie na neaktuálne údaje — naďalej sa používa posledná hodnota (hodnoty v ceste zachytávania nikdy nevypršia).

Ak je priradený senzor: sprístupní sa typ exportu „Reflectance“, živé náhľady sú korigované na osvetlenie, prediktívna automatická expozícia môže využívať spektrum a každé zachytenie odrazivosti zapíše skutočne použitú hodnotu DAQ ako **súbor `.daq`** do zložky `daq/`, aby bolo možné zachytenie neskôr opätovne spracovať. Vedľa neho sa v súbore `.daq.cameras.json` uvádzajú názvy kamier, ktorých snímky táto hodnota popisuje; pri opätovnom spracovaní program Chloros

priraďuje túto hodnotu iba k snímkam z týchto kamier.

## Možnosti CLI

u pre súbor `array-connect`

| Príznak | Predvolené | Popis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatické vyhľadanie všetkých kamier LATTICE (vyžaduje ≥2) | **Prvé sériové číslo je MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Synchronizačná linka GPIO. |
| `--target-fps F` | auto | Rýchlosť spúšťania Master. |
| `--binning {1,2,4}` | auto | Hardvérové zlučovanie. Nastavenie „auto“ znamená použitie celého snímača (bez zlučovania), ak to umožňuje akákoľvek snímková frekvencia; pripojenie „never bins“ slúži len na udržanie vysokej frekvencie. |
| `--packed {auto,on,off}` | `auto` | [Komprimovaný 12-bitový prenos](#packed-12-bit-transport): `auto` komprimuje len vtedy, keď je to prospešné a hostiteľ si môže dovoliť dekompresiu. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Odborné prepisovanie výberu synchronizačnej vrstvy. |
| `--wire-ceiling-mbps MB_PER_S` | automaticky detekované | Kapacita sieťového pripojenia hostiteľa v MB/s — forma poľa **Wire Budget** v rozhraní CLI

. Znížte ju, ak pole hlási poškodené rámce. Ukladá sa spolu s projektom, takže pri neskoršom opätovnom pripojení sa obnoví. |
| `--no-recommend` | vypnuté | Preskočiť krok analýzy siete. |
| `--no-ptp` | vypnuté | Deaktivovať PTP (časové značky medzi kamerami potom nie sú porovnateľné). |

`lattice array-list`, `array-status` a `array-disconnect` spravujú trvalú reláciu. Úplný zoznam podpríkazov, vrátane zarovnania (`align-calibrate` / `align-apply`) a sieťových nástrojov, sa nachádza v [Referencii CLI

§ chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); ekvivalenty v SDK

(`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) sa nachádzajú v [Referencii SDK

](../reference/sdk-reference.md). V jazyku Python

je rozpočet na prenos `connect_array(..., wire_ceiling_mbps=120)` a rozdelenie medzi poškodené a nedoručené správy v reálnom čase sa nachádza v [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).

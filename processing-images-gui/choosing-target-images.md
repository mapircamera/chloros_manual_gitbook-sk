# Výber cieľových snímok

Označením snímok, ktoré obsahujú kalibračné ciele, presne určíte programu Chloros, kde ich má hľadať. Ak je v stĺpci „Target“ zaškrtnutá aspoň jedna snímka, program Chloros skenuje **len zaškrtnuté snímky** — označenie cieľov teda slúži nielen na urýchlenie spracovania, ale aj na to, aby sa snímky z prieskumu nezamieňali s cieľmi.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Prečo označovať cieľové snímky?

### Označenie riadi proces skenovania

Keď v stĺpci „Cieľ“ zaškrtnete konkrétne snímky:

* Chloros skenuje iba zaškrtnuté snímky na prítomnosť cieľov
* Detekcia cieľov prebieha oveľa rýchlejšie
* Snímky z prieskumu nemôžu spôsobiť falošnú detekciu cieľov

Ak nie sú zaškrtnuté **žiadne** snímky, program Chloros prejde do režimu skenovania všetkých snímok v projekte:

* Algoritmy detekcie cieľov sa spustia na každej snímke
* Zbytočne sa skontrolujú stovky alebo tisíce snímok
* Spracovanie trvá podstatne dlhšie, najmä pri veľkých súboroch údajov

{% hint style="success" %}
**Zvýšenie rýchlosti**: Označením 2–3 snímok s cieľmi v súbore údajov obsahujúcom 500 snímok môžete skrátiť čas detekcie cieľov z viac ako 30 minút na menej ako 1 minútu.
{% endhint %}

***

## Ako označiť snímky cieľov

### Krok 1: Identifikujte snímky cieľov

Prezrite si importované snímky v prehliadači súborov a zistite, ktoré snímky obsahujú kalibračné ciele.

**Bežné scenáre:*** **Cieľ pred snímaním**: Zachytený pred začatím relácie
* **Cieľ po snímaní**: Zachytený po ukončení relácie
* **Ciele v teréne**: Ciele umiestnené v oblasti snímania
* **Viac cieľov**: 2–3 snímky cieľov na reláciu (odporúčané)

### Krok 2: Skontrolujte stĺpec „Target“ (<img src="../.gitbook/assets/image (33).png" alt="" data-size="original">)

Pre každý obrázok obsahujúci kalibračný cieľ:

1. Vyhľadajte obrázok v tabuľke prehliadača súborov
2. Nájdite stĺpec **Target** (stĺpec úplne vpravo)
3. Zaškrtnite políčko v stĺpci „Target“ pre daný obrázok
4. Opakujte tento postup pre všetky snímky obsahujúce ciele

### Krok 3: Overte svoj výber

Pred spracovaním ešte raz skontrolujte:

* [ ] Všetky snímky s kalibračnými cieľmi sú zaškrtnuté
* [ ] Žiadne snímky bez cieľov nie sú omylom zaškrtnuté
* [ ] Ciele sú na zaškrtnutých snímkach jasne viditeľné

***

## LATTICE: Ciele sú voliteľné pri nahrávaní prostredníctvom DAQ

V prípade multispektrálnych kamier LATTICE je kalibračný cieľ v rámci snímky **jednou z dvoch** možných referencií odrazivosti:

* **Cieľ v rámci snímky**: keď označený snímok s cieľom prejde kontrolou kvality (QA) nástrojChloros, cieľ sa stane**absolútnou referenciou odrazivosti** pre snímky v jeho okolí.
* **Smer svetla smerom nadol v DAQ**: ak nie je prítomný žiadny cieľ (alebo neprejde kontrolou kvality), softvér Chloros namiesto toho vypočíta odrazivosť na základe intenzity žiarenia smerom nadol zo svetelného senzora DAQ (ρ = π·L/E). Ak vaše snímky pokrýva záznam z `.daq` alebo DAQ-M `.csv`, získate kalibrovanú odrazivosť**bez akýchkoľvek snímok cieľa**.

Toto automatické správanie je predvolené. V súbore CLI / SDK to zodpovedá nastaveniu `--reflectance-source auto`; môžete tiež vynútiť nastavenie `target` (prísne — bez nahradenia údajov z DAQ) alebo `daq` (prednosť údajov z DAQ). Pozrite si [Referenciu CLI](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**Geometrie cieľov LATTICE**: okrem klasickej detekcie panelov používaných pre Survey3 spracovanie LATTICE podporuje**cieľe označené ArUco**,**cieľe s pevnou oblasťou záujmu (ROI)**a**pásové cieľe**, konfigurované podľa projektu. Skeny**nameraných** odrazivostí cieľov na jednotku je možné poskytnúť podľa sériového čísla (CLI: `--target-reflectance-dir`, jeden `<serial>.csv` na jednotku cieľa), pričom ako záložné riešenie slúžia nominálne spektrá T3/T4P.

{% hint style="info" %}
**Modul F988**: Odrazivosť modulu F988 sa kalibruje pomocou panelu odrazivosti umiestneného v scéne: pásmo leží mimo kalibrovaného rozsahu svetelného senzora DAQ, preto aplikáciChloros použije váš najnovší záznam z panelu a zachová ho medzi jednotlivými pozorovaniami panelu. Ak sa modul F988 spracováva výlučne prostredníctvom DAQ, nástroj Chloros odmietne odrazivosť založenú na DAQ pre dané pásmo (dôvod vynechania `dls-uncalibrated-band-988`) — podporovaným postupom je práca s panelom.
{% endhint %}

***

## Osvedčené postupy pre snímky cieľa

### Pokyny na snímanie cieľa

**Načasovanie:**

* Snímajte snímky cieľa bezprostredne pred začiatkom a počas celej snímacej relácie
* Za rovnakých svetelných podmienok ako svetelný senzor DAQ
* Pre dosiahnutie najlepších výsledkov je ideálne snímať snímky cieľa čo najčastejšie. V opačnom prípade sa na priebežnú úpravu kalibrácie použijú údaje zo svetelného senzora.

**Poloha kamery:**

* Držte kameru nad cieľom tak, aby bol vystredený a vyplňoval približne 40–60 % stredu snímky.
* Udržujte kameru rovnobežne s povrchom cieľa alebo v nadire voči nemu

**Osvetlenie:**

* Rovnaké okolité osvetlenie ako pri svetelnom senzore DAQ
* Vyhnite sa tieňom na povrchoch cieľov
* Nezakrývajte zdroj svetla svojím telom, vozidlom ani vegetáciou
* Zamračená obloha poskytuje najkonzistentnejšie výsledky

**Stav cieľa:**

* Udržujte panely cieľa čisté a suché
* Všetky panely vášho cieľa (napr. všetky 4 na modeli T4) by mali byť jasne viditeľné a ničím nezakryté
* Ciele by mali byť, ak je to možné, v kolmej polohe alebo v nadire voči zdroju svetla

### Koľko snímok cieľa?

**Minimálne:**1 snímka cieľa na jednu reláciu.**Odporúčané:** 3–5 snímok cieľa na jednu reláciu.**Odporúčaný postup:**

* 3–5 snímok zachytených krátko po spustení záznamu svetelného senzora
* Pre dosiahnutie najlepších výsledkov striedajte polohu kamery medzi jednotlivými zábermi
* Voliteľné: pravidelne v priebehu relácie, ak sa svetelné podmienky neustále menia

***

## Práca s viacerými kamerami

### Konfigurácie s dvoma kamerami

Ak používate súčasne dve kamery typu „MAPIR“ (napr. Survey3W RGN + Survey3N OCN):

1. Zachyťte snímky cieľa **oboma kamerami** súčasne
2. Pre obe kamery použite **rovnaký fyzický cieľ**

3. V prehliadači súborov označte snímky cieľa pre**oba typy kamier**

4. Nástroj Chloros použije na kalibráciu každej kamery príslušné ciele

### Stĺpec „Model kamery“

Stĺpec **„Model kamery“** pomáha identifikovať, ktoré snímky pochádzajú z ktorej kamery:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* atď.

Tento stĺpec použite na overenie, či ste v projekte označili ciele pre každý typ kamery.

***

## Nastavenia detekcie cieľov

### Úprava citlivosti detekcie

Ak funkcia „Chloros“ (Detekcia cieľov) nedetekuje vaše ciele správne, upravte tieto nastavenia v [Nastaveniach projektu](adjusting-project-settings.md):**Minimálna plocha kalibračnej vzorky (px):*** **Predvolené nastavenie**: 25 pixelov
* **Zvýšte hodnotu**, ak dochádza k falošným detekciám pri malých artefaktoch
* **Znížte hodnotu**, ak sa ciele nedetekujú**Minimálne zhlukovanie cieľov (0–100):*** **Predvolené**: 60
* **Zvýšte**, ak sa ciele rozdeľujú na viacero detekcií
* **Znížte**, ak sa ciele s farebnými odlišnosťami nedetekujú úplne

{% hint style="info" %}
**Tip k CLI**: `chloros-cli process` podporuje rovnaké nastavenia (`--min-target-size`, `--target-clustering`) a jeho príznak `--target`/`--targets` označuje celý vstupný priečinok ako určený výlučne pre panel cieľov. Pozrite si [Referenciu CLI](../reference/cli-reference.md).
{% endhint %}

***

## Bežné problémy s cieľovými obrázkami

### Problém: Neboli zistené žiadne ciele

**Možné príčiny:**

* Cieľové obrázky nie sú označené v prehliadači súborov
* Cieľ je v zábere príliš malý (&lt; 30 % obrázka)
* Zlé osvetlenie (tiene, odlesky)
* Príliš prísne nastavenia detekcie cieľov

**Riešenia:**

1. Skontrolujte, či je pri správnych obrázkoch zaškrtnutý stĺpec „Cieľ“
2. Skontrolujte kvalitu obrázkov cieľov v náhľade
3. Ak je kvalita nízka, znovu nasnímajte ciele
4. V prípade potreby upravte nastavenia detekcie cieľov

### Problém: Falošné detekcie cieľov

**Možné príčiny:**

* Biele budovy, vozidlá alebo povrch terénu sú mylne považované za ciele
* Svetlé škvrny vo vegetácii
* Príliš nízka citlivosť detekcie

**Riešenia:**

1. Označte len skutočné snímky cieľov — skenované budú len zaškrtnuté snímky
2. Zväčšite minimálnu plochu kalibračnej vzorky
3. Zväčšite minimálnu hodnotu zhlukovania cieľov
4. Uistite sa, že snímky cieľov zobrazujú len cieľ (minimálne rušivé prvky v pozadí)

***

## Kontrolný zoznam overovania

Pred začatím spracovania overte výber snímok cieľov:

* [ ] Aspoň 1 označená snímka cieľa na jednu reláciu (alebo v prípade LATTICE záznam `.daq`/`.csv` pokrývajúci danú reláciu)
* [ ] Zaškrtnuté sú políčka v stĺpci „Cieľ“ pre všetky snímky cieľa
* [ ] Snímky cieľa boli nasnímané v rovnakom časovom období ako prieskum
* [ ] Ciele sú po kliknutí jasne viditeľné v náhľade
* [ ] Všetky kalibračné panely sú viditeľné na každom snímku cieľa
* [ ] Na cieľoch nie sú žiadne tiene ani prekážky
* [ ] V prípade dvoch kamier: Ciele sú označené pre oba typy kamier

***

## Spracovanie bez cieľov

### LATTICE: S nahrávkou DAQ

Ak svetelný senzor DAQ zaznamenal intenzitu slnečného žiarenia smerujúceho nadol počas vašich snímok LATTICE, nie je potrebný žiadny cieľ:

1. Importujte súbor `.daq` (alebo DAQ-M `.csv`) so snímkami
2. Nechajte stĺpec „Cieľ“ nezaškrtnutý
3. Odrazivosť sa automaticky vypočíta z referenčného žiarenia smerujúceho nadol z DAQ
4. Radiance nikdy nepotrebuje cieľ ani DAQ — vychádza výlučne z továrenskej radiometrickej kalibrácie kamery

### Spracovanie bez akejkoľvek referencie

Spracovanie je možné vykonať aj bez cieľov a bez DAQ:

1. Nechajte všetky začiarkavacie políčka v stĺpci „Target“ nezačiarknuté
2. **Deaktivujte** „Kalibrácia odrazivosti / vyváženie bielej“ v nastaveniach projektu – detekcia cieľov sa potom úplne preskočí
3. Korekcia vinetácie sa aj naďalej uplatní
4. Výstup nebude kalibrovaný na absolútnu odrazivosť (LATTICE multispektrálny režim naďalej exportuje produkty bez debayeringu, náhľadové a radiančné produkty)

{% hint style="warning" %}
**Neodporúča sa pre vedeckú prácu v rámci projektu „Survey3“**: Bez kalibrácie odrazivosti predstavujú hodnoty pixelov Survey3 iba relatívnu jasnosť, nie vedecké merania odrazivosti. Pre presné a opakovateľné výsledky používajte kalibračné ciele (alebo v prípade LATTICE svetelný senzor DAQ).
{% endhint %}

***

## Ďalšie kroky

Po označení cieľových obrázkov:

1. **Skontrolujte nastavenia** – pozrite si [Úprava nastavení projektu](adjusting-project-settings.md)
2. **Spustite spracovanie** – pozrite si [Spustenie spracovania](starting-the-processing.md)
3. **Sledujte priebeh** – pozrite si [Sledovanie spracovania](monitoring-the-processing.md)

Ďalšie informácie o samotných kalibračných terčoch nájdete v časti [Kalibračné terče](../calibration-targets.md).

# Výber cieľových obrázkov

Označenie obrázkov, ktoré obsahujú kalibračné ciele, je kľúčovým krokom, ktorý výrazne urýchľuje spracovateľský reťazec Chloros. Predbežným výberom cieľových obrázkov eliminujete potrebu, aby Chloros prehľadával každý obrázok vo vašom súbore údajov s cieľom nájsť kalibračné ciele.

## Prečo označovať cieľové obrázky?

### Rýchlosť spracovania

Bez označenia cieľových obrázkov musí Chloros:

* Prehľadať každý jeden obrázok vo vašom projekte
* Spustiť algoritmy detekcie cieľov na každom obrázku
* Zbytočne skontrolovať stovky alebo tisíce obrázkov

**Výsledok**: Spracovanie môže trvať podstatne dlhšie, najmä v prípade veľkých dátových súborov.

### S označenými cieľovými obrázkami

Keď zaškrtnete stĺpec Cieľ pre konkrétne obrázky:

* Chloros skenuje iba zaškrtnuté obrázky na prítomnosť cieľov
* Detekcia cieľov prebieha oveľa rýchlejšie
* Celková doba spracovania sa výrazne skráti

{% hint style="success" %}
**Zvýšenie rýchlosti**: Označením 2–3 cieľových obrázkov v súbore s 500 obrázkami môžete skrátiť čas detekcie cieľov z viac ako 30 minút na menej ako 1 minútu.
{% endhint %}

***

## Ako označiť obrázky cieľov

### Krok 1: Identifikujte svoje obrázky cieľov

Prezrite si importované obrázky v prehliadači súborov a identifikujte, ktoré obrázky obsahujú kalibračné ciele.

**Bežné scenáre:*** **Cieľ pred snímaním**: Zachytený pred začatím relácie
* **Cieľ po snímaní**: Zachytený po dokončení relácie
* **Ciele v teréne**: Ciele umiestnené v oblasti snímania
* **Viac cieľov**: 2–3 cieľové obrázky na reláciu (odporúčané)

### Krok 2: Skontrolujte stĺpec Cieľ

Pre každý obrázok obsahujúci kalibračný cieľ:

1. Vyhľadajte obrázok v tabuľke prehliadača súborov
2. Nájdite stĺpec **Cieľ** (stĺpec úplne vpravo)
3. Zaškrtnite políčko v stĺpci Cieľ pre daný obrázok
4. Opakujte pre všetky obrázky obsahujúce ciele

### Krok 3: Overte svoj výber

Pred spracovaním ešte raz skontrolujte:

* [ ] Všetky snímky s kalibračnými cieľmi sú zaškrtnuté
* [ ] Žiadne snímky bez cieľa nie sú omylom zaškrtnuté
* [ ] Ciele sú na zaškrtnutých snímkach jasne viditeľné

***

## Osvedčené postupy pre snímky cieľa

### Pokyny na snímanie cieľa

**Načasovanie:**

* Snímky cieľa snímejte bezprostredne pred a počas celej relácie snímania
* Za rovnakých svetelných podmienok ako váš svetelný senzor DAQ
* Pre dosiahnutie najlepších výsledkov snímejte obrázky cieľov v ideálnom prípade čo najčastejšie. V opačnom prípade sa na úpravu kalibrácie v priebehu času použijú údaje zo svetelného senzora.

**Poloha fotoaparátu:**

* Držte fotoaparát nad cieľom tak, aby bol vystredený a vyplňoval približne 40–60 % stredu obrázka.
* Udržujte fotoaparát paralelne/v nadire voči povrchu cieľa

**Osvetlenie:**

* Rovnaké okolité osvetlenie ako u vášho svetelného senzora DAQ
* Vyhnite sa tieňom na povrchoch cieľov
* Nezakrývajte zdroj svetla svojím telom, vozidlom alebo vegetáciou
* Zamračená obloha poskytuje najkonzistentnejšie výsledky

**Stav cieľa:**

* Udržujte panely cieľa čisté a suché
* Všetky 4 panely by mali byť jasne viditeľné a ničím nezakryté
* Ciele by mali byť pokiaľ možno kolmé/v nadire voči zdroju svetla

### Koľko snímok cieľa?

**Minimálne:**1 snímka cieľa na jednu reláciu.**Odporúčané:** 3–5 snímok cieľa na jednu reláciu.**Osvedčený postup:**

* 3–5 snímok zachytených krátko po začatí záznamu svetelného senzora
* Pre dosiahnutie najlepších výsledkov striedajte polohu kamery medzi jednotlivými zábermi
* Voliteľné: pravidelne v priebehu relácie, ak sa svetelné podmienky neustále menia

***

## Práca s viacerými kamerami

### Nastavenia s dvoma kamerami

Ak používate dve kamery MAPIR súčasne (napr. Survey3W RGN + Survey3N OCN):

1. Zachyťte snímky cieľa **oboma kamerami** súčasne
2. Použite **rovnaký fyzický cieľ** pre obe kamery
3. Označte snímky cieľa pre **oba typy kamier** v prehliadači súborov
4. Chloros použije vhodné ciele pre kalibráciu každej kamery

### Stĺpec Model kamery

Stĺpec **Model kamery** pomáha identifikovať, ktoré snímky pochádzajú z ktorej kamery:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* atď.

Tento stĺpec použite na overenie, či ste v projekte označili ciele pre každý typ kamery.

***

## Nastavenia detekcie cieľov

### Úprava citlivosti detekcie

Ak Chloros nedetekuje vaše ciele správne, upravte tieto nastavenia v [Nastaveniach projektu](adjusting-project-settings.md):**Minimálna plocha kalibračnej vzorky:*** **Predvolené**: 25 pixelov
* **Zvýšte**, ak dochádza k falošným detekciám na malých artefaktoch
* **Znížte**, ak sa ciele nedetekujú**Minimálne zhlukovanie cieľov:*** **Predvolené**: 60
* **Zvýšte**, ak sa ciele rozdeľujú na viacero detekcií
* **Znížte**, ak sa ciele s farebnými odlišnosťami nedetekujú úplne***

## Bežné problémy s obrázkami cieľov

### Problém: Nedetekované ciele

**Možné príčiny:**

* Obrázky cieľov nie sú označené v prehliadači súborov
* Cieľ je v snímke príliš malý (&lt; 30 % obrázka)
* Zlé osvetlenie (tiene, odlesky)
* Príliš prísne nastavenia detekcie cieľov

**Riešenia:**

1. Overte, či je v stĺpci Cieľ zaškrtnutá možnosť pre správne obrázky
2. Skontrolujte kvalitu obrázkov cieľov v náhľade
3. Ak je kvalita zlá, znovu zachyťte ciele
4. V prípade potreby upravte nastavenia detekcie cieľov

### Problém: Falošné detekcie cieľov

**Možné príčiny:**

* Biele budovy, vozidlá alebo povrch zeme sa mýlia s cieľmi
* Svetlé škvrny vo vegetácii
* Príliš nízka citlivosť detekcie

**Riešenia:**

1. Označte len skutočné obrázky cieľov, aby ste obmedzili rozsah detekcie
2. Zväčšite minimálnu kalibračnú vzorkovú plochu
3. Zväčšite minimálnu hodnotu zhlukovania cieľov
4. Uistite sa, že obrázky cieľov zobrazujú len cieľ (minimálne rušivé prvky v pozadí)

***

## Kontrolný zoznam overovania

Pred začatím spracovania overte výber obrázkov cieľov:

* [ ] Označený je aspoň 1 obrázok cieľa na jednu reláciu
* [ ] Zaškrtnuté sú políčka v stĺpci Cieľ pre všetky obrázky cieľov
* [ ] Obrázky cieľov boli zachytené v rovnakom časovom rámci ako prieskum
* [ ] Ciele sú jasne viditeľné v náhľade po kliknutí
* [ ] Všetky 4 kalibračné panely sú viditeľné na každom obrázku cieľa
* [ ] Na cieľoch nie sú žiadne tiene ani prekážky
* [ ] Pre dvojitú kameru: Ciele sú označené pre oba typy kamier

***

## Spracovanie bez cieľov

### Spracovanie bez kalibračných cieľov

Hoci sa to pre vedeckú prácu neodporúča, môžete spracovávať bez cieľov:

1. Nechajte všetky začiarkavacie políčka v stĺpci Cieľ nezačiarknuté
2. **Deaktivujte** „Kalibráciu odrazivosti“ v nastaveniach projektu
3. Korekcia vinety sa bude naďalej uplatňovať
4. Výstup nebude kalibrovaný na absolútnu odrazivosť

{% hint style="warning" %}
**Neodporúča sa**: Bez kalibrácie odrazivosti hodnoty pixelov predstavujú iba relatívnu jasnosť, nie vedecké merania odrazivosti. Pre presné a opakovateľné výsledky použite kalibračné ciele.
{% endhint %}

***

## Ďalšie kroky

Po označení vašich cieľových obrázkov:

1. **Skontrolujte nastavenia** – pozrite si [Úprava nastavení projektu](adjusting-project-settings.md)
2. **Spustite spracovanie** – pozrite si [Spustenie spracovania](starting-the-processing.md)
3. **Sledujte priebeh** – pozrite si [Sledovanie spracovania](monitoring-the-processing.md)

Ďalšie informácie o samotných kalibračných cieľoch nájdete v časti [Kalibračné ciele](../calibration-targets.md).

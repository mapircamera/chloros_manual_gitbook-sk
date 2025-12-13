# Výber cieľových obrázkov

Označenie obrázkov, ktoré obsahujú kalibračné ciele, je kľúčovým krokom, ktorý výrazne urýchľuje spracovanie Chloros. Predbežným výberom cieľových obrázkov eliminujete potrebu, aby Chloros skenoval každý obrázok vo vašej dátovej sade za účelom nájdenia kalibračných cieľov.

## Prečo označovať cieľové obrázky?

### Rýchlosť spracovania

Bez označenia cieľových obrázkov musí Chloros:

* Skenovať každý jeden obrázok vo vašom projekte.
* Spustiť algoritmy detekcie cieľov na každom obrázku.
* Zbytočne skontrolovať stovky alebo tisíce obrázkov.

**Výsledok**: Spracovanie môže trvať podstatne dlhšie, najmä v prípade veľkých dátových súborov.

### S označenými cieľovými obrázkami

Keď zaškrtnete stĺpec Cieľ pre konkrétne obrázky:

* Chloros skenuje iba zaškrtnuté obrázky na ciele
* Detekcia cieľov je oveľa rýchlejšia
* Celková doba spracovania je výrazne skrátená

{% hint style=&quot;success&quot; %}
**Zrýchlenie**: Označením 2–3 cieľových obrázkov v súbore údajov s 500 obrázkami môžete skrátiť čas detekcie cieľov z viac ako 30 minút na menej ako 1 minútu.
{% endhint %}

***

## Ako označiť cieľové obrázky

### Krok 1: Identifikujte svoje cieľové obrázky

Prezrite si importované obrázky v prehliadači súborov a identifikujte, ktoré obrázky obsahujú kalibračné ciele.

**Bežné scenáre:**

* **Cieľ pred zachytením**: Zachytený pred začatím relácie
* **Cieľ po zachytení**: Zachytený po dokončení relácie
* **Ciele v teréne**: Ciele umiestnené v oblasti zachytenia
* **Viac cieľov**: 2–3 cieľové obrázky na reláciu (odporúčané)

### Krok 2: Skontrolujte stĺpec Cieľ

Pre každý obrázok obsahujúci kalibračný cieľ:

1. Vyhľadajte obrázok v tabuľke prehliadača súborov.
2. Vyhľadajte stĺpec **Cieľ** (stĺpec úplne vpravo).
3. Zaškrtnite políčko v stĺpci Cieľ pre daný obrázok.
4. Opakujte pre všetky obrázky obsahujúce ciele.

### Krok 3: Overte svoj výber

Pred spracovaním skontrolujte:

* [ ] Všetky obrázky s kalibračnými cieľmi sú zaškrtnuté.
* [ ] Žiadne obrázky, ktoré nie sú cieľmi, nie sú omylom zaškrtnuté.
* [ ] Ciele sú na zaškrtnutých obrázkoch jasne viditeľné.

***

## Osvedčené postupy pre cieľové obrázky

### Pokyny na zachytávanie cieľov

**Načasovanie:**

* Zachytávajte cieľové obrázky bezprostredne pred a počas zachytávania.
* Za rovnakých svetelných podmienok ako svetelný senzor DAQ.
* V ideálnom prípade zachytávajte cieľové obrázky čo najčastejšie, aby ste dosiahli najlepšie výsledky. V opačnom prípade sa na úpravu kalibrácie v priebehu času použijú údaje zo svetelného senzora.

**Poloha kamery:**

* Držte kameru nad cieľom tak, aby bola vystredená a vyplňovala približne 40–60 % stredu obrázka.
* Udržujte kameru paralelne/nadir s povrchom cieľa

**Osvetlenie:**

* Rovnaké okolité osvetlenie ako váš svetelný senzor DAQ.
* Vyhnite sa tieňom na povrchoch cieľa.
* Nezakrývajte zdroj svetla svojím telom, vozidlom alebo vegetáciou.
* Zamračené podmienky poskytujú najkonzistentnejšie výsledky.

**Stav cieľa:**

* Udržujte panely cieľa čisté a suché.
* Všetky 4 panely by mali byť jasne viditeľné a nezakryté.
* Ciele by mali byť v prípade možnosti kolmé/nadir k zdroju svetla.

### Koľko obrázkov cieľa?

**Minimálne:** 1 cieľový obraz na jednu reláciu. **Odporúčané:** 3–5 cieľových obrazov na jednu reláciu.

**Najlepší postup:**

* 3–5 obrazov zachytených krátko po začatí nahrávania svetelného senzora
* Pre dosiahnutie najlepších výsledkov otáčajte kameru medzi jednotlivými zábermi
* Voliteľné: pravidelne v polovici relácie, ak sa podmienky osvetlenia neustále menia

***

## Práca s viacerými kamerami

### Nastavenie dvoch kamier

Ak používate dve kamery MAPIR súčasne (napr. Survey3W RGN + Survey3N OCN):

1. Zachyťte cieľové obrázky **oboma kamerami** súčasne.
2. Použite **rovnaký fyzický cieľ** pre obe kamery.
3. Označte cieľové obrázky pre **oba typy kamier** v prehliadači súborov.
4. Chloros použije vhodné ciele pre kalibráciu každej kamery.

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

Ak Chloros nedetekuje vaše ciele správne, upravte tieto nastavenia v [Nastaveniach projektu](adjusting-project-settings.md):

**Minimálna plocha vzorky kalibrácie:**

* **Predvolené nastavenie**: 25 pixelov
* **Zvýšte**, ak dochádza k falošným detekciám malých artefaktov.
* **Znížte**, ak sa ciele nedetekujú.

**Minimálne zhlukovanie cieľov:**

* **Predvolené nastavenie**: 60
* **Zvýšte**, ak sa ciele rozdeľujú na viacero detekcií.
* **Znížte**, ak sa ciele s farebnými odchýlkami nedetekujú úplne.

***

## Bežné problémy s obrazom cieľa

### Problém: Nedetekujú sa žiadne ciele.

**Možné príčiny:**

* Obrazy cieľov nie sú označené v prehliadači súborov.
* Cieľ je v ráme príliš malý (&lt; 30 % obrazu).
* Zlé osvetlenie (tiene, odlesky)
* Príliš prísne nastavenia detekcie cieľov

**Riešenia:**

1. Overte, či je v stĺpci Cieľ zaškrtnuté správne obrázky.
2. Skontrolujte kvalitu obrázkov cieľov v náhľade.
3. Ak je kvalita zlá, znovu zachyťte ciele.
4. V prípade potreby upravte nastavenia detekcie cieľov.

### Problém: Falošné detekcie cieľov

**Možné príčiny:**

* Biele budovy, vozidlá alebo povrch zeme zamieňané za ciele
* Jasné škvrny vo vegetácii
* Príliš nízka citlivosť detekcie

**Riešenia:**

1. Označte len skutočné cieľové obrázky, aby ste obmedzili rozsah detekcie.
2. Zväčšite minimálnu kalibračnú vzorkovú oblasť.
3. Zväčšite minimálnu hodnotu zhlukovania cieľov.
4. Uistite sa, že cieľové obrázky zobrazujú len cieľ (minimálne rušenie pozadia).

***

## Kontrolný zoznam overenia

Pred začatím spracovania overte výber obrázkov cieľov:

* [ ] Aspoň 1 obrázok cieľa označený na jednu reláciu.
* [ ] Zaškrtnuté políčka stĺpca Cieľ pre všetky obrázky cieľov.
* [ ] Obrázky cieľov zachytené v rovnakom časovom rámci ako prieskum.
* [ ] Ciele jasne viditeľné v náhľade po kliknutí.
* [ ] Všetky 4 kalibračné panely viditeľné na každom obrázku cieľa.
* [ ] Na cieľoch nie sú žiadne tiene ani prekážky.
* [ ] V prípade dvoch kamier: Ciele sú označené pre oba typy kamier.

***

## Spracovanie bez cieľov

### Spracovanie bez kalibračných cieľov

Hoci sa to neodporúča pre vedeckú prácu, môžete spracovávať bez cieľov:

1. Nechajte všetky začiarkavacie políčka v stĺpci Cieľ nezačiarknuté.
2. **Deaktivujte** „Kalibráciu odrazivosti“ v nastaveniach projektu.
3. Korekcia vinetácie sa bude naďalej uplatňovať.
4. Výstup nebude kalibrovaný na absolútnu odrazivosť.

{% hint style=&quot;warning&quot; %}
**Neodporúča sa**: Bez kalibrácie odrazivosti hodnoty pixelov predstavujú iba relatívnu jasnosť, nie vedecké merania odrazivosti. Pre presné a opakované výsledky použite kalibračné ciele.
{% endhint %}

***

## Ďalšie kroky

Po označení cieľových obrázkov:

1. **Skontrolujte nastavenia** – pozrite si [Úprava nastavení projektu](adjusting-project-settings.md)
2. **Spustite spracovanie** – pozri [Spustenie spracovania](starting-the-processing.md)
3. **Sledujte priebeh** – pozri [Sledovanie spracovania](monitoring-the-processing.md)

Ďalšie informácie o kalibračných cieľoch nájdete v časti [Kalibračné ciele](../calibration-targets.md).

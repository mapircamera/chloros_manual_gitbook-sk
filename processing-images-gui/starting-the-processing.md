# Spustenie spracovania

Po importe obrázkov, označení kalibračných cieľov a konfigurácii nastavení projektu ste pripravení na spustenie spracovania. Táto stránka vás prevedie spustením spracovateľského reťazca Chloros.

## Kontrolný zoznam pred spracovaním

Pred kliknutím na tlačidlo Štart skontrolujte, či je všetko pripravené:

* [ ] **Súbory importované** – Všetky snímky sa zobrazujú v prehliadači súborov
* [ ] **Cieľové snímky označené** – Stĺpec Cieľ je zaškrtnutý pre kalibračné snímky (alebo je importovaný záznam `.daq` pre LATTICE)
* [ ] **Zistené modely kamier** – stĺpec „Model kamery“ zobrazuje správne kamery
* [ ] **Nastavenia nakonfigurované** – nastavenia projektu skontrolované a upravené
* [ ] **Vybrané indexy** – pridané požadované multispektrálne indexy (ak je to potrebné)
* [ ] **Zvolený formát exportu** – výstupný formát vhodný pre váš pracovný postup

{% hint style="info" %}
**Tip**: Pred spracovaním prejdite niekoľko snímok v prehliadači súborov, aby ste overili, či sa načítajú správne.
{% endhint %}

***

## Spustenie spracovania

### Nájdite tlačidlo Štart

Tlačidlo Štart/Prehrať sa nachádza v hornej lište aplikácie Chloros:

* Poloha: V hornej strednej časti okna
* Ikona: **Tlačidlo Prehrať/Štart** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Stav: Tlačidlo je aktívne (svieti), keď je program pripravený na spracovanie

### Kliknutím spustíte spracovanie

1. Kliknite na **tlačidlo Prehrávanie/Štart** v hornej lište
2. Spracovanie sa začne okamžite
3. Počas spracovania sa tlačidlo zmení na tlačidlo **Zastaviť**

4. Ukazovateľ priebehu sa aktualizuje a zobrazuje stav spracovania

{% hint style="success" %}
**Spracovanie začalo**: Po kliknutí program Chloros automaticky vykoná všetky kroky spracovania – detekciu cieľa, debayering, kalibráciu, výpočet indexu a export. Automaticky zistí, či je váš projekt typu Survey3, LATTICE alebo ich kombináciou, a na každú kameru aplikuje správny postup spracovania.
{% endhint %}

***

## Vysvetlenie režimov spracovania

Chloros pracuje v dvoch rôznych režimoch spracovania v závislosti od vašej licencie:

### Bezplatný režim (sekvenčné spracovanie)

**Dostupné pre všetkých používateľov**

**Ako to funguje:**

* Spracováva snímky po jednej, sekvenčne
* Jednovláknová prevádzka
* Nižšia spotreba pamäte

**Indikátor priebehu zobrazuje 2 fázy:**

1.**Detekcia cieľov** – vyhľadávanie kalibračných cieľov
2. **Spracovanie** – aplikácia kalibrácie a export snímok**Doba spracovania:**

* Oveľa pomalšie ako paralelné spracovanie v režime Chloros+
* Vhodné pre malé až stredné súbory údajov (&lt; 200 obrázkov)

### Režim Chloros+ (paralelné spracovanie)

**Vyžaduje licenciu Chloros+**

**Ako to funguje:**

* Spracováva viacero snímok súčasne pomocou [4-vláknového spracovateľského potrubia](../processing-architecture/processing-pipeline.md)
* [Dynamická adaptácia výpočtov](../processing-architecture/dynamic-compute-adaptation.md) automaticky vyberie optimálnu stratégiu pre váš hardvér pri spustení
* Akcelerácia GPU (CUDA) s grafickými kartami NVIDIA (stolné počítače a Jetson)
* **Počet pracovných procesov sa prispôsobuje hardvéru**: stratégie využívajúce GPU spúšťajú**1 až 4 súbežné pracovné procesy** (v závislosti od VRAM – Jetson s nízkou pamäťou spúšťa 1, stolný počítač s GPU s 12 GB a viac spúšťa až 4); Systémy využívajúce iba CPU spúšťajú jedného pracovného procesa na jedno fyzické jadro, mínus jeden**Indikátor priebehu zobrazuje 4 fázy** (odpovedajúce 4 vláknam v potrubí):

1. **Detekcia** (vlákno 1) – Vyhľadávanie kalibračných cieľov
2. **Analýza** (vlákno 2) – Preskúmanie metadát obrázku a výpočet kalibrácie
3. **Kalibrácia** (vlákno 3) – Odstránenie bayesovského efektu, korekcia vinety, kalibrácia, výpočet indexu
4. **Export** (vlákno 4) – Uloženie spracovaných obrázkov a indexov**Interakcia s ukazovateľom priebehu:*** **Naveďte kurzor myši** na ukazovateľ, aby sa zobrazil podrobný rozbaľovací panel so 4 fázami
* **Kliknite** na ukazovateľ priebehu, aby sa rozbaľovací panel zafixoval na mieste
* **Kliknite znovu**, aby sa panel odblokoval a skryl**Doba spracovania:**

* Výrazne rýchlejšie ako v bezplatnom režime
* Akcelerácia GPU ešte viac zvyšuje rýchlosť

{% hint style="info" %}
**Chloros+ Rýchlosť**: Pri veľkých dátových súboroch môže byť paralelné spracovanie 5–10-krát rýchlejšie ako sekvenčný režim. Projekt s 500 obrázkami, ktorý v bezplatnom režime trvá 2 hodiny, sa s Chloros+ môže dokončiť za 15–20 minút.
{% endhint %}

***

## Čo sa deje počas spracovania

### Fáza 1: Detekcia cieľov

**Čo robí Chloros:**

* Prehľadáva snímky, ktoré ste zaškrtli v stĺpci „Cieľ“ (ak nie je zaškrtnutý žiadny, prehľadáva všetky snímky)
* Identifikuje kalibračné panely v každom cieli
* Extrahuje hodnoty odrazivosti z cieľových panelov
* Zaznamenáva časové značky cieľov pre plánovanie kalibrácie

**Trvanie:** 1–30 sekúnd (s označenými cieľmi), 5–30+ minút (neoznačené)

### Fáza 2: Odstránenie bayera (konverzia RAW)

**Čo robí Chloros:**

* Konvertuje údaje vo formáte RAW s Bayerovým vzorom na plnohodnotné 3-kanálové snímky (mono moduly LATTICE zostávajú jednopásmové – debayering sa u nich preskočí s poznámkou v protokole)
* Používa vybraný algoritmus demosaicingu
* Zachováva maximálnu kvalitu a detaily snímky

**Trvanie:** Závisí od počtu snímok a rýchlosti CPU/GPU

### Fáza 3: Kalibrácia

**Čo robí Chloros:*** **Korekcia vinetácie**: Odstraňuje stmavnutie okrajov spôsobené objektívom
* **Kalibrácia odrazivosti**: Normalizuje pomocou cieľových hodnôt odrazivosti a/alebo údajov o dopadajúcom žiarení z DAQ
* Uplatňuje korekcie vo všetkých pásmach/kanáloch
* Pre každý obrázok používa vhodnú kalibračnú referenciu na základe časovej pečiatky

**Trvanie:** Väčšina času spracovania

### Fáza 4: Výpočet indexu

**Čo robí Chloros:**

* Vypočíta nakonfigurované multispektrálne indexy (NDVI, NDRE atď.)
* Uplatňuje matematické operácie s pásmami na kalibrované snímky
* Generuje indexové snímky pre každý vybraný index

**Trvanie:** Niekoľko sekúnd na snímku

### Fáza 5: Export

**Čo robí Chloros:**

* Ukladá spracované snímky vo vybranom formáte
* **LATTICE fan-out**: každý surový snímok LATTICE sa exportuje ako každý povolený produkt v jednom kroku — debayered, náhľad, radiancia (vždy float32), odrazivosť
* Ukladá súbory do výstupnej zložky projektu: `<project>/<camera>/<format>/<Product>_Images/`
* **Zachováva názov zdrojového súboru** — produkt identifikuje zložka, nepridáva sa žiadna príponka**Trvanie:** Záleží od formátu exportu a veľkosti súboru***

## Správanie spracovania

### Automatický spracovateľský reťazec

Po spustení beží celý reťazec automaticky:

* Nie je potrebná žiadna interakcia používateľa
* Všetky nakonfigurované kroky sa vykonávajú postupne
* Aktualizácie priebehu sa zobrazujú v reálnom čase
* Exportované súbory sa ukladajú na disk hneď po dokončení — hotové výstupy môžete otvárať, zatiaľ čo spracovanie pokračuje

### Využitie počítača počas spracovania

**Voľný režim:**

* Relatívne nízke využitie procesora (jednovláknový režim)
* Počítač zostáva odozvový pre ostatné úlohy
* Je bezpečné minimalizovať okno Chloros a pracovať v iných aplikáciách

**Chloros+ Paralelný režim:**

* Vysoké využitie procesora v rámci pracovnej skupiny stratégie
* S akceleráciou GPU: Vysoké využitie grafickej karty
* Počas spracovania môže byť počítač menej odozvový
* Vyhnite sa spúšťaniu iných úloh náročných na procesor

{% hint style="warning" %}
**Tip na zvýšenie výkonu**: Pre dosiahnutie najlepšieho výkonu Chloros+ zatvorte ostatné aplikácie a nechajte Chloros využívať všetky systémové zdroje.
{% endhint %}

### Spracovanie nie je možné pozastaviť (ale jeho zastavenie je čisté)

* Po spustení nie je možné spracovanie pozastaviť a neskôr obnoviť
* Kliknutím na **Zastaviť** sa spustenie čistým spôsobom zastaví už pri prvom kliknutí
* Produkty, ktoré boli exportované ešte pred zastavením, zostanú na disku
* Zastavený beh presne informuje o tom, čo sa podarilo dokončiť (pozrite si riadky `[RUN-SUMMARY]` v protokole)
* Nové spustenie začne spracovateľský reťazec od začiatku

**Tip na plánovanie:** Pri veľmi rozsiahlych projektoch zvážte spracovanie v dávkach alebo použitie CLI pre lepšiu kontrolu.***

## Sledovanie spracovania

Počas spracovania môžete:

* **Sledovať ukazovateľ priebehu** – zobraziť celkové percento dokončenia
* **Zobraziť aktuálnu fázu** – detekcia, analýza, kalibrácia alebo export
* **Skontrolovať kartu protokolu** – zobraziť podrobné správy a varovania týkajúce sa spracovania
* **Prezrieť dokončené snímky** – exportované súbory sa počas spracovania ukladajú na disk

Podrobné informácie o monitorovaní nájdete v časti [Monitorovanie spracovania](monitoring-the-processing.md).

***

## Zastavenie spracovania

Ak potrebujete zastaviť spracovanie:

### Ako zastaviť

1. Nájdite **tlačidlo Zastaviť** (počas spracovania nahrádza tlačidlo Štart)
2. Kliknite naň raz – na lište sa zobrazí **„Zastavuje sa...“**, kým sa dokončí spracovanie aktuálneho snímku
3. Spracovanie sa ukončí v definitívnom zastavenom stave a v protokole sa vypíše presný výpis `[RUN-SUMMARY]` o tom, čo bolo dokončené

### Kedy zastaviť

**Platné dôvody na zastavenie:**

* Zistili ste, že boli použité nesprávne nastavenia
* Zabudli ste označiť cieľové obrázky
* Importovali ste nesprávne obrázky
* Systém beží príliš pomaly alebo nereaguje

**Po zastavení:**

* Produkty exportované pred zastavením zostávajú na disku
* Skontrolujte a odstráňte prípadné problémy, podľa potreby upravte nastavenia
* Spustite spracovanie znova — spracovanie začne od začiatku

***

## Odhady doby spracovania

Skutočná doba spracovania sa výrazne líši v závislosti od:

* Počet snímok
* Rozlíšenie snímok
* Vstupný formát RAW vs. JPG
* Režim spracovania (Free vs. Chloros+)
* Rýchlosť procesora a počet jadier
* Dostupnosť grafickej karty (len Chloros+)
* Počet indexov, ktoré sa majú vypočítať
* Počet povolených exportných produktov (LATTICE)

### Hrubé odhady (Chloros+, snímky s rozlíšením 12 MP, moderný procesor)

| Počet obrázkov | Režim Free | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 snímok   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 snímok  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 snímok  | 1–1,5 hod. | 20–30 min      | 10–15 min      |
| 500 obrázkov  | 2–3 hodiny   | 45–60 min      | 20–30 min      |
| 1000 obrázkov | 4–6 hodín   | 1,5–2 hodiny      | 40–60 min      |

{% hint style="info" %}
**Prvé spustenie**: Počiatočné spracovanie môže trvať dlhšie, keďže Chloros vytvára vyrovnávaciu pamäť a profily. Následné spracovanie podobných súborov údajov bude rýchlejšie.
{% endhint %}

***

## Bežné problémy pri spustení

### Tlačidlo Spustiť je deaktivované (sivé)

**Možné príčiny:**

* Neimportovali sa žiadne obrázky
* Backend nie je úplne spustený
* Predchádzajúce spracovanie stále prebieha
* Projekt nie je úplne načítaný

**Riešenia:**

1. Počkajte, kým sa backend úplne inicializuje (skontrolujte ikonu v hlavnom menu)
2. Overte, či sú obrázky importované v prehliadači súborov
3. Ak tlačidlo zostáva deaktivované, reštartujte Chloros
4. Skontrolujte protokol ladenia, či neobsahuje chybové správy

### Spracovanie sa spustí, ale okamžite zlyhá

**Možné príčiny:**

* V projekte nie sú žiadne platné obrázky
* Poškodené súbory obrázkov
* Nedostatočný voľný priestor na disku
* Nedostatočná pamäť (RAM)

**Riešenia:**

1. Skontrolujte protokol ladenia <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">, či neobsahuje chybové správy
2. Overte, či je k dispozícii dostatok miesta na disku
3. Skúste spracovať menšiu podmnožinu obrázkov
4. Overte, či obrázky nie sú poškodené

### Spustenie sa dokončí, ale nezapíše žiadne obrázky

Spustenie, pri ktorom boli požadované obrazové produkty, ale nebol zapísaný žiadny, sa považuje za **zlyhanie, nie za úspech** — Chloros to jasne hlási:

* Protokol grafického rozhrania (GUI) vypíše hlásenie `[RUN-SUMMARY]`, ktoré naznačuje pravdepodobnú príčinu — neboli importované žiadne snímky, nebol detekovaný žiadny cieľ alebo boli všetky požadované produkty vynechané ako neaplikovateľné (napr. požiadavka na radianciu/odrazivosť z kamier, ktoré podporujú iba RGB)
* Ekvivalent CLI (`chloros-cli process`) vypíše `Processing finished but wrote no image products.` a **ukončí sa s nenulovým kódom**, takže skripty to môžu zistiť
* Úmyselné spustenie iba s metadátami (všetky exportované produkty sú deaktivované, žiadne indexy) sa stále považuje za úspešné

Úplnú sémantiku nájdete v [referencii k CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).

### Upozornenie „Nebol zistený žiadny cieľ“

**Možné príčiny:**

* Zabudli ste označiť cieľové obrázky
* Cieľové obrázky neobsahujú viditeľné ciele
* Nastavenia detekcie cieľov sú príliš prísne

**Riešenia:**

1. Prečítajte si časť [Výber cieľových obrázkov](choosing-target-images.md)
2. Označte príslušné obrázky v stĺpci „Cieľ“
3. Overte, či sú ciele viditeľné na označených obrázkoch
4. V prípade potreby upravte nastavenia detekcie cieľov

***

## Tipy na úspešné spracovanie

### Pred začatím

1. **Najprv otestujte na malej podmnožine** – spracujte 10–20 obrázkov na overenie nastavení
2. **Skontrolujte voľné miesto na disku** – zabezpečte 2–3-násobok veľkosti súboru údajov (viac, ak sú povolené všetky produkty LATTICE)
3. **Zatvorte nepotrebné aplikácie** – uvoľnite systémové zdroje
4. **Overte snímky s cieľmi** – prezrite si označené ciele, aby ste sa uistili o ich kvalite
5. **Uložte projekt** – projekt sa ukladá automaticky, ale je dobré ho uložiť aj ručne

### Počas spracovania

1. **Vyhnite sa režimu spánku systému** – Vypnite režimy úspory energie
2. **Nechajte Chloros v popredí** – Alebo aspoň viditeľný na paneli úloh
3. **Občas sledujte priebeh spracovania** – Skontrolujte, či sa neobjavili varovania alebo chyby
4. **Nenačítavajte iné náročné aplikácie** – Najmä pri paralelnom režime Chloros+

### Chloros+ Akcelerácia GPU

Ak používate akceleráciu grafického procesora NVIDIA:

1. Aktualizujte ovládače NVIDIA na najnovšiu verziu
2. Uistite sa, že grafický procesor má 4 GB a viac VRAM (7 GB a viac pre súbežné odstraňovanie pixelov s podporou textúr)
3. Zatvorte aplikácie náročné na grafický procesor (hry, editovanie videa)
4. Sledujte teplotu grafického procesora (zabezpečte dostatočné chladenie)

***

## Ďalšie kroky

Po spustení spracovania:

1. **Sledujte priebeh** – pozrite si [Sledovanie spracovania](monitoring-the-processing.md)
2. **Počkajte na dokončenie** – spracovanie prebieha automaticky
3. **Skontrolujte výsledky** – pozrite si [Dokončenie spracovania](finishing-the-processing.md)

Informácie o tom, čo robiť počas spracovania, nájdete v časti [Monitorovanie spracovania](monitoring-the-processing.md).

# Spustenie spracovania

Po importe obrázkov, označení kalibračných cieľov a konfigurácii nastavení projektu ste pripravení na spustenie spracovania. Táto stránka vás prevedie spustením spracovateľského reťazca programu Chloros.

## Kontrolný zoznam pred spracovaním

Pred kliknutím na tlačidlo „Štart“ skontrolujte, či je všetko pripravené:

* [ ] **Súbory importované** – Všetky snímky sa zobrazujú v prehliadači súborov
* [ ] **Cieľové snímky označené** – V stĺpci „Cieľ“ sú zaškrtnuté kalibračné snímky (alebo bol pre LATTICE importovaný záznam `.daq`)
* [ ] **Zistené modely kamier** – stĺpec „Model kamery“ zobrazuje správne kamery
* [ ] **Nastavenia nakonfigurované** – nastavenia projektu skontrolované a upravené
* [ ] **Vybrané indexy** – pridané požadované multispektrálne indexy (ak je to potrebné)
* [ ] **Zvolený formát exportu** – výstupný formát vhodný pre váš pracovný postup

{% hint style="info" %}
**Tip**: Pred spracovaním prejdite niekoľko obrázkov v prehliadači súborov, aby ste overili, či sa načítali správne.
{% endhint %}

***

## Spustenie spracovania

### Nájdite tlačidlo Štart

Tlačidlo Štart/Prehrať sa nachádza v hornej lište programu Chloros:

* Poloha: V hornej strednej časti okna
* Ikona: **Tlačidlo Prehrať/Štart** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Stav: Tlačidlo je aktívne (svieti), keď je program pripravený na spracovanie

### Kliknutím spustíte spracovanie

1. Kliknite na **tlačidlo Prehrávanie/Štart** v hornej lište
2. Spracovanie sa začne okamžite
3. Počas spracovania sa tlačidlo zmení na tlačidlo **Zastaviť**

4. Ukazovateľ priebehu sa aktualizuje a zobrazuje stav spracovania

{% hint style="success" %}
**Spracovanie začaté**: Po kliknutí program Chloros automaticky vykoná všetky kroky spracovania – detekciu cieľa, debayering, kalibráciu, výpočet indexu a export. Automaticky zistí, či je váš projekt typu Survey3, LATTICE alebo ich kombináciou, a na každú kameru uplatní správny postup spracovania.
{% endhint %}

***

## Vysvetlenie režimov spracovania Program

Chloros pracuje v dvoch rôznych režimoch spracovania v závislosti od vašej licencie:

### Bezplatný režim (sekvenčné spracovanie)

**Dostupné pre všetkých používateľov**

**Ako to funguje:**

* Spracováva snímky po jednej, postupne
* Jednovláknová prevádzka
* Nižšia spotreba pamäte

**Indikátor priebehu zobrazuje 2 fázy:**

1.**Detekcia cieľov** – Vyhľadávanie kalibračných cieľov
2. **Spracovanie** – Uplatnenie kalibrácie a export snímok**Doba spracovania:**

* Oveľa pomalšie ako paralelný režim Chloros+
* Vhodné pre malé až stredné súbory dát (< 200 obrázkov)

### Režim Chloros+ (paralelné spracovanie)

**Vyžaduje licenciu Chloros+**

**Ako to funguje:**

* Spracováva viacero snímok súčasne pomocou [4-vláknového spracovateľského potrubia](../processing-architecture/processing-pipeline.md)
* [Dynamická adaptácia výpočtov](../processing-architecture/dynamic-compute-adaptation.md) automaticky vyberá optimálnu stratégiu pre váš hardvér pri spustení
* Akcelerácia GPU (CUDA) s grafickými kartami NVIDIA
* **Počet pracovných procesov sa prispôsobuje hardvéru**: stratégie GPU spúšťajú**2–4 súbežné pracovné procesy** (škálované podľa VRAM — 3 na GPU s 2–12 GB, 2 až 4 na GPU s 12 GB a viac); Systémy s výhradne CPU spúšťajú jedného pracovníka na každé fyzické jadro, mínus jeden**Indikátor priebehu zobrazuje 4 fázy** (odpovedajúce 4 vláknam potrubia):

1. **Detekcia** (vlákno 1) – Vyhľadávanie kalibračných cieľov
2. **Analýza** (vlákno 2) – Preskúmanie metadát obrázkov a výpočet kalibrácie
3. **Kalibrácia** (vlákno 3) – Odstránenie bayeresovho efektu, korekcia vinety, kalibrácia, výpočet indexu
4. **Export** (vlákno 4) – Uloženie spracovaných obrázkov a indexov**Interakcia s indikátorom priebehu:*** **Naveďte kurzor myši** na indikátor, aby sa zobrazil podrobný rozbaľovací panel so 4 fázami
* **Kliknutím** na indikátor priebehu panel zafixujete na mieste
* **Ďalším kliknutím** panel odfixujete a skryjete**Doba spracovania:**

* Výrazne rýchlejšie ako v bezplatnom režime
* Akcelerácia GPU ďalej zvyšuje rýchlosť

{% hint style="info" %}
**Rýchlosť Chloros+**: Paralelné spracovanie môže byť pri veľkých dátových súboroch 5–10-krát rýchlejšie ako sekvenčný režim. Projekt s 500 obrázkami, ktorý v bezplatnom režime trvá 2 hodiny, sa s Chloros+ môže dokončiť za 15–20 minút.
{% endhint %}

***

## Čo sa deje počas spracovania

### Fáza 1: Detekcia cieľov

**Čo robí Chloros:**

* Prehľadáva obrázky, ktoré ste zaškrtli v stĺpci „Cieľ“ (ak nie je zaškrtnutý žiadny, prehľadáva všetky obrázky)
* Identifikuje kalibračné panely v každom cieli
* Extrahuje hodnoty odrazivosti z panelov cieľov
* Zapisuje časové značky cieľov pre plánovanie kalibrácie

**Trvanie:** 1–30 sekúnd (s označenými cieľmi), 5–30+ minút (bez označenia)

### Fáza 2: Debayering (konverzia formátu RAW)

**Čo robí Chloros:**

* Konvertuje údaje vo formáte RAW s Bayerovým vzorom na plnohodnotné 3-kanálové snímky (mono moduly LATTICE zostávajú jednopásmové – debayering sa u nich preskočí s poznámkou v protokole)
* Používa vybraný algoritmus demosaicingu
* Zachováva maximálnu kvalitu a detaily snímky

**Trvanie:** Závisí od počtu snímok a rýchlosti CPU/GPU

### Fáza 3: Kalibrácia

**Čo robí Chloros:*** **Korekcia vinetácie**: Odstraňuje stmavnutie okrajov spôsobené objektívom
* **Kalibrácia odrazivosti**: Normalizuje pomocou cieľových hodnôt odrazivosti a/alebo údajov o dopadajúcom žiarení zo zberu údajov (DAQ)
* Uplatňuje korekcie vo všetkých pásmach/kanáloch
* Pre každý snímok používa vhodnú kalibračnú referenciu na základe časovej pečiatky

**Trvanie:** Väčšina času spracovania

### Fáza 4: Výpočet indexov

**Čo robí Chloros:**

* Vypočíta nakonfigurované multispektrálne indexy (NDVI, NDRE atď.)
* Uplatňuje matematické operácie s pásmami na kalibrované snímky
* Generuje indexové snímky pre každý vybraný index

**Trvanie:** Niekoľko sekúnd na snímku

### Fáza 5: Export

**Čo robí Chloros:**

* Ukladá spracované snímky vo vybranom formáte
* **LATTICE fan-out**: každý surový snímok LATTICE sa exportuje ako každý povolený produkt v jednom kroku — debayering, náhľad, radiancia (vždy float32), odrazivosť
* Ukladá súbory do výstupnej štruktúry projektu: `<project>/<camera>/<format>/<Product>_Images/`
* **Zachováva názov zdrojového súboru** — produkt identifikuje zložka, nepridáva sa žiadna prípona**Trvanie:** Záleží od formátu exportu a veľkosti súboru***

## Správanie spracovania

### Automatický spracovateľský reťazec

Po spustení beží celý reťazec automaticky:

* Nie je potrebná žiadna interakcia používateľa
* Všetky nakonfigurované kroky sa vykonávajú postupne
* Aktualizácie priebehu sa zobrazujú v reálnom čase
* Exportované súbory sa ukladajú na disk hneď po dokončení — hotové výstupy môžete otvárať, zatiaľ čo spracovanie pokračuje

### Využitie počítača počas spracovania

**Režim Free:**

* Relatívne nízke využitie procesora (jednovláknový režim)
* Počítač zostáva odozvový pre ostatné úlohy
* Je bezpečné minimalizovať aplikáciu Chloros a pracovať v iných aplikáciách

**Režim Chloros+ Parallel:**

* Vysoké využitie procesora v rámci pracovnej skupiny stratégie
* S akceleráciou GPU: Vysoké využitie grafickej karty
* Počas spracovania môže byť počítač menej pohotový
* Vyhnite sa spúšťaniu iných úloh náročných na procesor

{% hint style="warning" %}
**Tip na zvýšenie výkonu**: Pre dosiahnutie najlepšieho výkonu Chloros+ zatvorte ostatné aplikácie a nechajte Chloros využívať všetky systémové zdroje.
{% endhint %}

### Spracovanie nemožno pozastaviť (ale jeho ukončenie je čisté)

* Po spustení nie je možné spracovanie pozastaviť a neskôr obnoviť
* Kliknutím na **Stop** sa beh čistým spôsobom zastaví už pri prvom kliknutí
* Výstupy, ktoré boli exportované pred zastavením, zostanú na disku
* Zastavený beh presne hlási, čo sa podarilo dokončiť (pozri riadky `[RUN-SUMMARY]` v protokole)
* Nové spustenie začne spracovávanie od začiatku

**Tip na plánovanie:** Pri veľmi veľkých projektoch zvážte spracovanie v dávkach alebo použitie CLI pre lepšiu kontrolu.***

## Sledovanie spracovávania

Počas spracovávania môžete:

* **Sledovať ukazovateľ priebehu** – zobraziť celkové percento dokončenia
* **Zobraziť aktuálnu fázu** – detekcia, analýza, kalibrácia alebo export
* **Skontrolovať kartu protokolu** – zobraziť podrobné správy a varovania týkajúce sa spracovania
* **Zobraziť náhľad dokončených obrázkov** – exportované súbory sa počas spracovania objavujú na disku

Podrobné informácie o monitorovaní nájdete v článku [Monitorovanie spracovania](monitoring-the-processing.md).

***

## Zastavenie spracovania

Ak potrebujete zastaviť spracovanie:

### Ako zastaviť

1. Nájdite **tlačidlo Zastaviť** (počas spracovania nahrádza tlačidlo Štart)
2. Kliknite naň raz — na lište sa zobrazí **„Zastavuje sa...“**, kým sa dokončí spracovávaný obrázok
3. Spracovanie sa ukončí v definitívnom zastavenom stave a v protokole sa vytlačí presný výpis `[RUN-SUMMARY]` o tom, čo bolo dokončené

### Kedy zastaviť

**Platné dôvody na zastavenie:**

* Zistili ste, že boli použité nesprávne nastavenia
* Zabudli ste označiť cieľové obrázky
* Importovali ste nesprávne obrázky
* Systém beží príliš pomaly alebo nereaguje

**Po zastavení:**

* Výstupy exportované pred zastavením zostanú na disku
* Skontrolujte a odstráňte prípadné problémy, podľa potreby upravte nastavenia
* Spustite spracovanie nanovo — spustenie začne od začiatku

***

## Odhady doby spracovania

Skutočná doba spracovania sa výrazne líši v závislosti od:

* Počtu snímok
* Rozlíšenia snímok
* Vstupného formátu (RAW vs. JPG)
* Režimu spracovania (Free vs. Chloros+)
* Rýchlosť procesora a počet jadier
* Dostupnosť grafickej karty (len Chloros+)
* Počet indexov, ktoré sa majú vypočítať
* Počet povolených exportovaných produktov (LATTICE)

### Hrubé odhady (Chloros+, obrázky s rozlíšením 12 MP, moderný procesor)

| Počet obrázkov | Režim Free | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 obrázkov   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 obrázkov  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 obrázkov  | 1–1,5 hod. | 20–30 min.      | 10–15 min.      |
| 500 obrázkov  | 2–3 hod.   | 45–60 min.      | 20–30 min.      |
| 1 000 obrázkov | 4–6 hodín   | 1,5–2 hodiny      | 40–60 minút      |

{% hint style="info" %}
**Prvé spustenie**: Počiatočné spracovanie môže trvať dlhšie, keďže Chloros vytvára vyrovnávaciu pamäť a profily. Následné spracovanie podobných dátových súborov bude rýchlejšie.
{% endhint %}

***

## Bežné problémy pri spustení

### Tlačidlo Štart je deaktivované (sivé)

**Možné príčiny:**

* Neimportovali sa žiadne obrázky
* Backend nie je úplne spustený
* Predchádzajúce spracovanie stále prebieha
* Projekt nie je úplne načítaný

**Riešenia:**

1. Počkajte, kým sa backend úplne inicializuje (skontrolujte ikonu v hlavnom menu)
2. Overte, či sú obrázky importované v prehliadači súborov
3. Ak tlačidlo zostáva neaktívne, reštartujte Chloros
4. Skontrolujte protokol ladenia, či neobsahuje chybové správy

### Spracovanie sa spustí, ale ihneď zlyhá

**Možné príčiny:**

* V projekte nie sú žiadne platné obrázky
* Poškodené súbory obrázkov
* Nedostatočný priestor na disku
* Nedostatočná pamäť (RAM)

**Riešenia:**

1. Skontrolujte protokol ladenia <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">, či neobsahuje chybové správy
2. Overte, či je k dispozícii dostatok miesta na disku
3. Skúste spracovať menšiu podmnožinu obrázkov
4. Overte, či obrázky nie sú poškodené

### Spustenie sa dokončí, ale nezapíše žiadne obrázky

Spustenie, ktoré požadovalo výstupné obrázky, ale nezapísalo žiadne, sa považuje za **zlyhanie, nie za úspech** — Chloros to jasne hlási:

* Protokol grafického rozhrania (GUI) vypíše hlásenia typu `[RUN-SUMMARY]`, ktoré naznačujú pravdepodobnú príčinu — neboli importované žiadne snímky, nebol zistený žiadny cieľ alebo boli všetky požadované produkty vynechané ako nepoužiteľné (napr. požiadavka na radianciu/odrazivosť z kamier, ktoré snímajú iba v RGB)
* Ekvivalent v CLI (`chloros-cli process`) vypíše `Processing finished but wrote no image products.` a **ukončí sa s nenulovým kódom**, takže skripty to môžu detekovať
* Úmyselné spustenie iba s metadátami (všetky exportované produkty sú deaktivované, žiadne indexy) sa stále považuje za úspech

Úplnú sémantiku nájdete v [referencii CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).

### Zložka projektu je pre tento počítač príliš hlboko

**Iba pre Windows, novinka vo verzii 1.3.0.** Systém Windows sa dodáva s vypnutou podporou dlhých ciest, pričom žiadna cesta k súboru nesmie byť dlhšia ako 260 znakov. Produkty Chloros sa nachádzajú približne 130 znakov pod zložkou projektu, takže zložka projektu, ktorá je hlbšie, nemôže prijímať svoje produkty. Namiesto spustenia a nezapísania ničho Chloros teraz pred začatím spracovania vykoná kontrolu:

* Kliknutím na tlačidlo Štart sa spustenie nezačne: indikátor priebehu zobrazí dôvod a v lógovom súbore ladenia sa zobrazí kód `[PROCESS] refused: The project folder is too deep for this computer…`. Správa uvádza dĺžku zložky projektu a najdlhšiu zložku projektu, ktorá sa do nej zmestí.
* Vytvorenie alebo otvorenie projektu v zložke, ktorá je príliš hlboko umiestnená dokonca aj pre vlastné súbory projektu, je tiež odmietnuté. V okne so správou sa uvádza dôvod, najdlhší názov projektového priečinka, ktorý sa zmestí, a postup, ako povoliť dlhé cesty; nič sa nevytvorí ani neotvorí; v lógovom súbore sa to zaznamená ako `[PROJECT] refused: …`. Duplikovanie projektu pod dlhším názvom môže tiež prekročiť limit: kópia sa vytvorí, ale neotvorí, a tá istá správa uvádza dôvod.

**Riešenia:**

1. Uložte projekty do kratšej zložky — zmeňte**Pracovný adresár** v [Nastaveniach projektu](../project-settings/project-settings.md) a projekt vytvorte znovu, alebo
2. Zapnite dlhé cesty v systéme Windows a reštartujte program Chloros: spustite inštalátor programu Chloros s zaškrtnutou možnosťou **Povoliť dlhé cesty v systéme Windows** (táto možnosť je predvolene zaškrtnutá) alebo použite zásady skupiny: *Konfigurácia počítača > Šablóny správcu > Systém > Súborový systém > Povoliť dlhé cesty Win32*.

### Upozornenie „Nebol zistený žiadny cieľ“

**Možné príčiny:**

* Zabudli ste označiť cieľové obrázky
* Cieľové obrázky neobsahujú viditeľné ciele
* Nastavenia detekcie cieľov sú príliš prísne

**Riešenia:**

1. Prečítajte si časť [Výber cieľových obrázkov](choosing-target-images.md)
2. Označte príslušné obrázky v stĺpci Cieľ
3. Overte, či sú ciele viditeľné v označených obrázkoch
4. V prípade potreby upravte nastavenia detekcie cieľov

***

## Tipy na úspešné spracovanie

### Pred spustením

1. **Najprv otestujte na malej vzorke** – spracujte 10–20 obrázkov na overenie nastavení
2. **Skontrolujte voľné miesto na disku** – zabezpečte 2–3-násobok veľkosti dátového súboru (viac, ak sú povolené všetky produkty LATTICE)
3. **Zatvorte nepotrebné aplikácie** – uvoľnite systémové zdroje
4. **Skontrolujte snímky s cieľmi** – prezrite si náhľad označených cieľov, aby ste sa uistili o kvalite
5. **Uložte projekt** – projekt sa ukladá automaticky, ale je dobré ho uložiť aj ručne

### Počas spracovania

1. **Vyhnite sa režimu spánku systému** – Vypnite režimy úspory energie
2. **Nechajte program Chloros v popredí** – Alebo aspoň viditeľný na paneli úloh
3. **Občas sledujte priebeh spracovania** – Skontrolujte, či sa neobjavujú varovania alebo chyby
4. **Nenačítavajte iné náročné aplikácie** – Najmä pri paralelnom režime Chloros+

### Akcelerácia Chlorosu+ pomocou GPU

Ak používate akceleráciu pomocou GPU od spoločnosti NVIDIA:

1. Aktualizujte ovládače NVIDIA na najnovšiu verziu
2. Uistite sa, že GPU má 4 GB a viac VRAM (7 GB a viac pre súbežné odstraňovanie bayeringu s podporou textúr)
3. Zatvorte aplikácie náročné na GPU (hry, editovanie videa)
4. Sledujte teplotu GPU (zabezpečte dostatočné chladenie)

***

## Ďalšie kroky

Po spustení spracovania:

1. **Sledujte priebeh** – pozrite si [Sledovanie spracovania](monitoring-the-processing.md)
2. **Počkajte na dokončenie** – spracovanie prebieha automaticky
3. **Skontrolujte výsledky** – pozrite si [Dokončenie spracovania](finishing-the-processing.md)

Informácie o tom, čo robiť počas spracovania, nájdete v článku [Monitorovanie spracovania](monitoring-the-processing.md).

# Spustenie spracovania

Po importe obrázkov, označení kalibračných cieľov a konfigurácii nastavení projektu ste pripravení na spustenie spracovania. Táto stránka vás prevedie spustením spracovateľského reťazca Chloros.

## Kontrolný zoznam pred spracovaním

Pred kliknutím na tlačidlo Štart skontrolujte, či je všetko pripravené:

* [ ] **Súbory importované** – Všetky snímky sa zobrazujú v prehliadači súborov
* [ ] **Cieľové snímky označené** – Stĺpec Cieľ je zaškrtnutý pre kalibračné snímky
* [ ] **Modely kamier zistené** – Stĺpec Model kamery zobrazuje správne kamery
* [ ] **Nastavenia nakonfigurované** – Nastavenia projektu skontrolované a upravené
* [ ] **Vybrané indexy** – Pridané požadované multispektrálne indexy (ak je to potrebné)
* [ ] **Zvolený formát exportu** – Výstupný formát vhodný pre váš pracovný postup

{% hint style="info" %}
**Tip**: Pred spracovaním prejdite niekoľkými obrázkami v prehliadači súborov, aby ste overili, či sa načítajú správne.
{% endhint %}

***

## Spustenie spracovania

### Nájdite tlačidlo Štart

Tlačidlo Štart/Prehrať sa nachádza v hornom paneli Chloros:

* Poloha: V strede hornej časti okna
* Ikona: **Tlačidlo Prehrať/Štart** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Stav: Tlačidlo je aktívne (svieti), keď je pripravené na spracovanie

### Kliknite na tlačidlo na spustenie

1. Kliknite na **tlačidlo Prehrávanie/Štart** v hornej lište
2. Spracovanie sa začne okamžite
3. Tlačidlo sa počas spracovania deaktivuje (zsivie)
4. Ukazovateľ priebehu sa aktualizuje a zobrazuje stav spracovania

{% hint style="success" %}
**Spracovanie začalo**: Po kliknutí Chloros automaticky vykoná všetky kroky spracovania – detekciu cieľa, debayering, kalibráciu, výpočet indexu a export.
{% endhint %}

***

## Vysvetlenie režimov spracovania

Chloros pracuje v dvoch rôznych režimoch spracovania v závislosti od vašej licencie:

### Bezplatný režim (sekvenčné spracovanie)

**Dostupné pre všetkých používateľov**

**Ako to funguje:**

* Spracováva obrázky jeden po druhom, sekvenčne
* Jednovláknová operácia
* Nižšia spotreba pamäte

**Indikátor priebehu zobrazuje 2 fázy:**

1.**Detekcia cieľa** – vyhľadávanie kalibračných cieľov
2. **Spracovanie** – aplikácia kalibrácie a export obrázkov**Doba spracovania:**

* Oveľa pomalšie ako paralelné režim Chloros+
* Vhodné pre malé až stredné súbory údajov (&lt; 200 obrázkov)

### Režim Chloros+ (paralelné spracovanie)

**Vyžaduje licenciu Chloros+**

**Ako to funguje:**

* Spracováva viacero obrázkov súčasne pomocou [4-vláknového spracovateľského potrubia](../processing-architecture/processing-pipeline.md)
* [Dynamická adaptácia výpočtov](../processing-architecture/dynamic-compute-adaptation.md) automaticky vyberá optimálnu stratégiu pre váš hardvér
* Akcelerácia GPU (CUDA) s grafickými kartami NVIDIA (stolné počítače a Jetson)
* Škálovateľné od Jetson Nano (1 pracovník) po stolný počítač s GPU 12 GB+ (3–4 pracovníci)

**Indikátor priebehu zobrazuje 4 fázy** (odpovedajúce 4 vláknam potrubia):

1. **Detekcia** (Vlákno 1) – Hľadanie kalibračných cieľov
2. **Analýza** (Vlákno 2) – Preskúmanie metadát obrazu a výpočet kalibrácie
3. **Kalibrácia** (Vlákno 3) – Odstránenie bayeringu pomocou GPU, korekcia vinety, výpočet indexu
4. **Export** (vlákno 4) – Uloženie spracovaných obrázkov a indexov**Interakcia s indikátorom priebehu:*** **Naveďte kurzor myši** na indikátor, aby sa zobrazil podrobný rozbaľovací panel so 4 fázami
* **Kliknite** na indikátor priebehu, aby sa rozbaľovací panel zmrazil na mieste
* **Kliknite znova**, aby sa panel odmrazil a skryl**Doba spracovania:**

* Výrazne rýchlejšie ako v bezplatnom režime
* Škálovateľné podľa počtu jadier procesora
* Akcelerácia GPU ďalej zvyšuje rýchlosť

{% hint style="info" %}
**Chloros+ Rýchlosť**: Paralelné spracovanie môže byť pri veľkých dátových súboroch 5-10x rýchlejšie ako sekvenčný režim. Projekt s 500 obrázkami, ktorý v bezplatnom režime trvá 2 hodiny, sa s Chloros+ môže dokončiť za 15-20 minút.
{% endhint %}

***

## Čo sa deje počas spracovania

### Fáza 1: Detekcia cieľa

**Čo robí Chloros:**

* Skenuje označené obrázky cieľa (alebo všetky obrázky, ak nie sú žiadne označené)
* Identifikuje 4 kalibračné panely v každom cieli
* Extrahuje hodnoty odrazivosti z panelov cieľa
* Zaznamenáva časové údaje cieľa pre plánovanie kalibrácie

**Trvanie:** 1–30 sekúnd (s označenými cieľmi), 5–30+ minút (neoznačené)

### Fáza 2: Debayering (konverzia RAW)

**Čo robí Chloros:**

* Konvertuje údaje v formáte RAW s Bayerovým vzorom na plnohodnotné obrázky RGB
* Používa vysoko kvalitný algoritmus demosaicingu
* Zachováva maximálnu kvalitu a detaily obrázku

**Trvanie:** Záleží od počtu obrázkov a rýchlosti procesora

### Fáza 3: Kalibrácia

**Čo robí Chloros:*** **Korekcia vinetácie**: Odstraňuje stmavnutie objektívu na okrajoch
* **Kalibrácia odrazivosti**: Normalizuje pomocou cieľových hodnôt odrazivosti
* Uplatňuje korekcie vo všetkých pásmach/kanáloch
* Používa vhodný kalibračný cieľ pre každý obrázok na základe časovej pečiatky

**Trvanie:** Väčšina času spracovania

### Fáza 4: Výpočet indexu

**Čo robí Chloros:**

* Vypočíta nakonfigurované multispektrálne indexy (NDVI, NDRE atď.)
* Uplatňuje matematické operácie na pásma na kalibrované snímky
* Generuje indexové snímky pre každý vybraný index

**Trvanie:** Niekoľko sekúnd na snímku

### Fáza 5: Export

**Čo robí Chloros:**

* Ukladá kalibrované snímky vo vybranom formáte
* Exportuje indexové snímky s nakonfigurovanými farbami LUT
* Zapisuje súbory do podpriečinkov modelov kamier
* Zachováva pôvodné názvy súborov s príponami

**Trvanie:** Záleží od formátu exportu a veľkosti súboru***

## Správanie spracovania

### Automatický spracovateľský reťazec

Po spustení beží celý reťazec automaticky:

* Nie je potrebná žiadna interakcia používateľa
* Všetky nakonfigurované kroky sa vykonávajú postupne
* Aktualizácie priebehu sa zobrazujú v reálnom čase

### Využitie počítača počas spracovania

**Voľný režim:**

* Relatívne nízke využitie CPU (jednovláknový)
* Počítač zostáva odozvový pre ostatné úlohy
* Bezpečné minimalizovať Chloros a pracovať v iných aplikáciách

**Chloros+ Paralelný režim:**

* Vysoké využitie CPU (viacvláknové, až 16 jadier)
* S akceleráciou GPU: Vysoké využitie GPU
* Počas spracovania môže byť počítač menej citlivý
* Vyhnite sa spúšťaniu iných úloh náročných na CPU

{% hint style="warning" %}
**Tip na zvýšenie výkonu**: Pre dosiahnutie najlepšieho výkonu Chloros+ zatvorte ostatné aplikácie a nechajte Chloros využívať všetky systémové zdroje.
{% endhint %}

### Spracovanie nemožno pozastaviť

**Dôležité obmedzenia:**

* Po spustení nemožno spracovanie pozastaviť
* Spracovanie môžete zrušiť, ale pokrok sa stratí
* Čiastočné výsledky sa neukladajú
* V prípade zrušenia je potrebné začať od začiatku

**Tip na plánovanie:** Pri veľmi veľkých projektoch zvážte spracovanie v dávkach alebo použitie CLI pre lepšiu kontrolu.***

## Monitorovanie spracovania

Počas spracovania môžete:

* **Sledovať ukazovateľ priebehu** – Zobraziť celkové percento dokončenia
* **Zobraziť aktuálnu fázu** – Detekcia, analýza, kalibrácia alebo export
* **Skontrolovať kartu protokolu** – zobraziť podrobné správy a varovania o spracovaní
* **Zobraziť náhľad dokončených obrázkov** – niektoré exportované súbory sa môžu zobraziť počas spracovania

Podrobné informácie o monitorovaní nájdete v časti [Monitorovanie spracovania](monitoring-the-processing.md).

***

## Zrušenie spracovania

Ak potrebujete zastaviť spracovanie:

### Ako zrušiť

1. Nájdite **tlačidlo Zastaviť/Zrušiť** (počas spracovania nahrádza tlačidlo Štart)
2. Kliknite na tlačidlo Zastaviť
3. Spracovanie sa okamžite zastaví
4. Čiastočné výsledky sa zrušia

### Kedy zrušiť

**Platné dôvody na zrušenie:**

* Zistili ste, že boli použité nesprávne nastavenia
* Zabudli ste označiť cieľové obrázky
* Importovali sa nesprávne obrázky
* Systém beží príliš pomaly alebo nereaguje

**Po zrušení:**

* Skontrolujte a opravte všetky problémy
* Upravte nastavenia podľa potreby
* Spustite spracovanie od začiatku
* Pre najlepší výsledok úplne zatvorte Chloros a reštartujte

{% hint style="warning" %}
**Žiadne čiastočné výsledky**: Zrušením sa zruší celý pokrok. Chloros neukladá čiastočne spracované obrázky.
{% endhint %}

***

## Odhady času spracovania

Skutočný čas spracovania sa výrazne líši v závislosti od:

* Počet obrázkov
* Rozlíšenie obrázkov
* Vstupný formát RAW vs. JPG
* Režim spracovania (Free vs Chloros+)
* Rýchlosť procesora a počet jadier
* Dostupnosť grafickej karty (len Chloros+)
* Počet indexov na výpočet
* Zložitosť exportného formátu

### Hrubé odhady (Chloros+, obrázky s rozlíšením 12 MP, moderný procesor)

| Počet obrázkov | Režim Free | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 obrázkov   | 15–20 min | 5–8 min        | 3–5 min        |
| 100 obrázkov  | 30–40 min | 10–15 min      | 5–8 min        |
| 200 obrázkov  | 1–1,5 hod. | 20–30 min      | 10–15 min      |
| 500 obrázkov  | 2–3 hod.   | 45–60 min      | 20–30 min      |
| 1000 obrázkov | 4–6 hod.   | 1,5–2 hod.      | 40–60 min      |

{% hint style="info" %}
**Prvé spustenie**: Počiatočné spracovanie môže trvať dlhšie, keďže Chloros vytvára vyrovnávaciu pamäť a profily. Nasledujúce spracovanie podobných dátových súborov bude rýchlejšie.
{% endhint %}

***

## Bežné problémy pri spustení

### Tlačidlo Spustiť je deaktivované (sivá farba)

**Možné príčiny:**

* Neimportované obrázky
* Backend nie je úplne spustený
* Predchádzajúce spracovanie stále beží
* Projekt nie je úplne načítaný

**Riešenia:**

1. Počkajte, kým sa backend úplne inicializuje (skontrolujte ikonu v hlavnom menu)
2. Overte, či sú obrázky importované v prehliadači súborov
3. Ak tlačidlo zostáva deaktivované, reštartujte Chloros
4. Skontrolujte protokol ladenia, či neobsahuje chybové správy

### Spracovanie sa spustí, ale ihneď zlyhá

**Možné príčiny:**

* V projekte nie sú žiadne platné obrázky
* Poškodené súbory obrázkov
* Nedostatočný priestor na disku
* Nedostatočná pamäť (RAM)

**Riešenia:**

1. Skontrolujte protokol ladenia <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> , či neobsahuje chybové správy
2. Overte dostupný priestor na disku
3. Skúste spracovať menšiu podmnožinu obrázkov
4. Overte, či obrázky nie sú poškodené

### Upozornenie „Nebol zistený žiadny cieľ“

**Možné príčiny:**

* Zabudli ste označiť cieľové obrázky
* Cieľové obrázky neobsahujú viditeľné ciele
* Nastavenia detekcie cieľov sú príliš prísne

**Riešenia:**

1. Prečítajte si [Výber cieľových obrázkov](choosing-target-images.md)
2. Označte príslušné obrázky v stĺpci Cieľ
3. Overte, či sú ciele viditeľné v označených obrázkoch
4. V prípade potreby upravte nastavenia detekcie cieľov

***

## Tipy na úspešné spracovanie

### Pred začatím

1. **Najprv otestujte na malej vzorke** – spracujte 10–20 obrázkov, aby ste overili nastavenia
2. **Skontrolujte voľné miesto na disku** – zabezpečte 2–3-násobok veľkosti dátového súboru
3. **Zatvorte nepotrebné aplikácie** – uvoľnite systémové zdroje
4. **Overte cieľové obrázky** – prehliadnite si označené ciele, aby ste sa uistili o kvalite
5. **Uložte projekt** – Projekt sa ukladá automaticky, ale je dobré ho uložiť aj ručne

### Počas spracovania

1. **Vyhnite sa režimu spánku systému** – Vypnite režimy úspory energie
2. **Nechajte Chloros v popredí** – Alebo aspoň viditeľný na paneli úloh
3. **Občas sledujte priebeh** – Skontrolujte, či sa neobjavili varovania alebo chyby
4. **Nenačítavajte iné náročné aplikácie** – najmä pri paralelnom režime Chloros+

### Chloros+ GPU akcelerácia

Ak používate GPU akceleráciu NVIDIA:

1. Aktualizujte ovládače NVIDIA na najnovšiu verziu
2. Uistite sa, že GPU má 4 GB+ VRAM
3. Zatvorte aplikácie náročné na GPU (hry, editovanie videa)
4. Sledujte teplotu GPU (zabezpečte dostatočné chladenie)

***

## Ďalšie kroky

Po spustení spracovania:

1. **Sledujte priebeh** – pozrite si [Sledovanie spracovania](monitoring-the-processing.md)
2. **Počkajte na dokončenie** – spracovanie prebieha automaticky
3. **Skontrolujte výsledky** – pozrite si [Dokončenie spracovania](finishing-the-processing.md)

Informácie o tom, čo robiť počas spracovania, nájdete v časti [Monitorovanie spracovania](monitoring-the-processing.md).

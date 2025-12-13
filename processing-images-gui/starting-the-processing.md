# Spustenie spracovania

Po importe obrázkov, označení kalibračných cieľov a konfigurácii nastavení projektu ste pripravení na spustenie spracovania. Táto stránka vás prevedie spustením spracovateľského potrubia Chloros.

## Kontrolný zoznam pred spracovaním

Pred kliknutím na tlačidlo Štart skontrolujte, či je všetko pripravené:

* [ ] **Importované súbory** – Všetky obrázky sa zobrazia v prehliadači súborov
* [ ] **Označené cieľové obrázky** – Stĺpec Cieľ je zaškrtnutý pre kalibračné obrázky
* [ ] **Detegované modely kamery** – stĺpec Camera Model (Model kamery) zobrazuje správne kamery
* [ ] **Nastavenia konfigurované** – nastavenia projektu skontrolované a upravené
* [ ] **Indexy vybrané** – pridané požadované multispektrálne indexy (ak je to potrebné)
* [ ] **Exportný formát vybraný** – výstupný formát vhodný pre váš pracovný postup

{% hint style=&quot;info&quot; %}
**Tip**: Pred spracovaním kliknite na niekoľko obrázkov v prehliadači súborov, aby ste overili, či sa načítali správne.
{% endhint %}

***

## Spustenie spracovania

### Nájdite tlačidlo Štart

Tlačidlo Štart/Prehrať sa nachádza v hornej lište Chloros:

* Poloha: V hornej strednej časti okna
* Ikona: **Tlačidlo Prehrať/Štart** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Stav: Tlačidlo je aktivované (svietiace), keď je pripravené na spracovanie

### Kliknutím spustite

1. Kliknite na **tlačidlo Prehrať/Spustiť** v hornej lište
2. Spracovanie sa začne okamžite
3. Tlačidlo sa počas spracovania deaktivuje (zosivie)
4. Ukazovateľ priebehu sa aktualizuje a zobrazuje stav spracovania

{% hint style=&quot;success&quot; %}
**Spracovanie začalo**: Po kliknutí Chloros automaticky spracuje všetky kroky spracovania – detekciu cieľa, debayering, kalibráciu, výpočet indexu a export.
{% endhint %}

***

## Porozumenie režimom spracovania

Chloros funguje v dvoch rôznych režimoch spracovania v závislosti od vašej licencie:

### Bezplatný režim (sekvenčné spracovanie)

**Dostupné pre všetkých používateľov**

**Ako to funguje:**

* Spracováva obrázky jeden po druhom, postupne.
* Jednovláknová prevádzka.
* Nižšia spotreba pamäte.

**Indikátor priebehu zobrazuje 2 fázy:**

1. **Detekcia cieľa** – vyhľadávanie kalibračných cieľov.
2. **Spracovanie** – aplikácia kalibrácie a export obrázkov.

**Doba spracovania:**

* Mnoho pomalšie ako paralelný režim Chloros+
* Vhodné pre malé až stredné dátové súbory (&lt; 200 obrázkov)

### Režim Chloros+ (paralelné spracovanie)

**Vyžaduje licenciu Chloros+**

**Ako to funguje:**

* Spracováva viacero obrázkov súčasne
* Viacvláknová prevádzka (až 16 paralelných pracovníkov)
* Využívanie viacerých jadier CPU
* Voliteľné zrýchlenie GPU (CUDA) s grafickými kartami NVIDIA

**Indikátor priebehu zobrazuje 4 fázy:**

1. **Detekcia** – hľadanie kalibračných cieľov
2. **Analýza** – preskúmanie metadát obrázkov a príprava potrubia
3. **Kalibrácia** – aplikovanie korekcií a kalibrácií
4. **Export** – ukladanie spracovaných obrázkov a indexov

**Interakcia s ukazovateľom priebehu:**

* **Naveďte kurzor myši** na ukazovateľ, aby sa zobrazil podrobný rozbaľovací panel so 4 fázami
* **Kliknutím** na ukazovateľ priebehu zmrazíte rozbaľovací panel na mieste
* **Ďalším kliknutím** panel rozmrazíte a skryjete

**Doba spracovania:**

* Výrazne rýchlejšie ako vo voľnom režime
* Škáluje sa podľa počtu jadier CPU
* Akcelerácia GPU ďalej zvyšuje rýchlosť

{% hint style=&quot;info&quot; %}
**Chloros+ Rýchlosť**: Paralelné spracovanie môže byť 5-10x rýchlejšie ako sekvenčný režim pre veľké dátové súbory. Projekt s 500 obrázkami, ktorý v bezplatnom režime trvá 2 hodiny, sa s Chloros+ môže dokončiť za 15-20 minút.
{% endhint %}

***

## Čo sa deje počas spracovania

### Fáza 1: Detekcia cieľa

**Čo robí Chloros:**

* Skenuje označené cieľové obrázky (alebo všetky obrázky, ak nie sú označené)
* Identifikuje 4 kalibračné panely v každom cieli
* Extrahuje hodnoty odrazivosti z cieľových panelov
* Zapisuje časové značky cieľov pre plánovanie kalibrácie

**Trvanie:** 1–30 sekúnd (s označenými cieľmi), 5–30+ minút (bez označenia)

### Fáza 2: Debayering (konverzia RAW)

**Čo robí Chloros:**

* Konvertuje údaje RAW Bayerovho vzoru na úplné obrazy RGB
* Používa vysoko kvalitný algoritmus demosaicingu
* Zachováva maximálnu kvalitu obrazu a detaily

**Trvanie:** Záleží od počtu obrazov a rýchlosti CPU

### Fáza 3: Kalibrácia

**Čo robí Chloros:**

* **Korekcia vinetácie**: Odstraňuje stmavnutie objektívu na okrajoch
* **Kalibrácia odrazivosti**: Normalizuje pomocou cieľových hodnôt odrazivosti
* Používa korekcie vo všetkých pásmach/kanáloch
* Používa vhodné kalibračné ciele pre každý obraz na základe časovej pečiatky

**Trvanie:** Väčšina času spracovania

### Fáza 4: Výpočet indexu

**Čo robí Chloros:**

* Vypočíta nakonfigurované multispektrálne indexy (NDVI, NDRE atď.)
* Uplatňuje matematiku pásiem na kalibrované obrazy
* Generuje indexové obrazy pre každý vybraný index

**Trvanie:** Niekoľko sekúnd na jeden obraz

### Fáza 5: Export

**Čo robí Chloros:**

* Ukladá kalibrované obrazy vo vybranom formáte
* Exportuje indexové obrazy s nakonfigurovanými farbami LUT
* Zapisuje súbory do podadresárov modelu fotoaparátu
* Zachováva pôvodné názvy súborov s príponami

**Trvanie:** Záleží od formátu exportu a veľkosti súboru

***

## Správanie spracovania

### Automatický spracovateľský proces

Po spustení celý proces prebieha automaticky:

* Nie je potrebná žiadna interakcia používateľa
* Všetky nakonfigurované kroky sa vykonávajú postupne
* Aktualizácie priebehu sa zobrazujú v reálnom čase

### Využitie počítača počas spracovania

**Voľný režim:**

* Relatívne nízke využitie CPU (jednovláknové)
* Počítač zostáva citlivý na ostatné úlohy
* Bezpečné minimalizovať Chloros a pracovať v iných aplikáciách

**Chloros+ Paralelný režim:**

* Vysoké využitie CPU (viacvláknové, až 16 jadier)
* S akceleráciou GPU: Vysoké využitie GPU
* Počítač môže byť počas spracovania menej citlivý
* Vyhnite sa spúšťaniu iných úloh náročných na CPU

{% tip style=&quot;warning&quot; %}
**Tip na zvýšenie výkonu**: Pre dosiahnutie najlepšieho výkonu Chloros+ zatvorte ostatné aplikácie a nechajte Chloros využívať všetky systémové zdroje.
{% endhint %}

### Spracovanie nemožno pozastaviť

**Dôležité obmedzenia:**

* Po spustení nie je možné spracovanie pozastaviť.
* Spracovanie môžete zrušiť, ale pokrok sa stratí.
* Čiastočné výsledky sa neukladajú.
* V prípade zrušenia je nutné začať od začiatku.

**Tip na plánovanie:** Pri veľmi veľkých projektoch zvážte spracovanie v dávkach alebo použitie CLI pre lepšiu kontrolu.

***

## Monitorovanie spracovania

Počas spracovania môžete:

* **Sledovať ukazovateľ priebehu** – zobraziť celkové percento dokončenia
* **Zobraziť aktuálnu fázu** – detekcia, analýza, kalibrácia alebo export
* **Skontrolovať kartu protokolu** – zobraziť podrobné správy a varovania týkajúce sa spracovania
* **Prezrieť si dokončené obrázky** – počas spracovania sa môžu zobraziť niektoré exportované súbory

Podrobné informácie o monitorovaní nájdete v časti [Monitorovanie spracovania](monitoring-the-processing.md).

***

## Zrušenie spracovania

Ak potrebujete zastaviť spracovanie:

### Ako zrušiť

1. Nájdite **tlačidlo Zastaviť/Zrušiť** (počas spracovania nahrádza tlačidlo Štart)
2. Kliknite na tlačidlo Zastaviť
3. Spracovanie sa okamžite zastaví
4. Čiastočné výsledky sa vyradia

### Kedy zrušiť

**Platné dôvody na zrušenie:**

* Zistili ste, že boli použité nesprávne nastavenia
* Zabudli ste označiť cieľové obrázky
* Importovali ste nesprávne obrázky
* Systém beží príliš pomaly alebo nereaguje

**Po zrušení:**

* Skontrolujte a opravte všetky problémy
* Upravte nastavenia podľa potreby
* Spustite spracovanie od začiatku
* Pre najčistejší zážitok úplne zatvorte Chloros a reštartujte

{% hint style=&quot;warning&quot; %}
**Žiadne čiastočné výsledky**: Zrušením sa zrušia všetky pokroky. Chloros neukladá čiastočne spracované obrázky.
{% endhint %}

***

## Odhady doby spracovania

Skutočná doba spracovania sa výrazne líši v závislosti od:

* Počtu obrázkov
* Rozlíšenia obrázkov
* Vstupného formátu RAW vs JPG
* Režimu spracovania (Free vs Chloros+)
* Rýchlosti CPU a počtu jadier
* Dostupnosti GPU (len Chloros+)
* Počtu indexov, ktoré sa majú vypočítať
* Zložitosti exportného formátu

### Hrubé odhady (Chloros+, 12MP obrázky, moderný CPU)

| Počet obrázkov | Režim Free | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 obrázkov   | 15–20 minút | 5–8 minút        | 3–5 minút        |
| 100 obrázkov  | 30–40 minút | 10–15 minút      | 5–8 minút        |
| 200 obrázkov  | 1–1,5 hodiny | 20–30 minút      | 10–15 minút      |
| 500 obrázkov  | 2–3 hodiny   | 45–60 minút      | 20–30 minút      |
| 1000 obrázkov | 4–6 hodín   | 1,5–2 hodiny      | 40–60 minút      |

{% hint style=&quot;info&quot; %}
**Prvé spustenie**: Počiatočné spracovanie môže trvať dlhšie, pretože Chloros vytvára vyrovnávaciu pamäť a profily. Následné spracovanie podobných dátových súborov bude rýchlejšie.
{% endhint %}

***

## Bežné problémy pri spustení

### Tlačidlo Spustiť je deaktivované (sivá farba)

**Možné príčiny:**

* Neimportovali sa žiadne obrázky.
* Backend nie je úplne spustený.
* Predchádzajúce spracovanie stále prebieha.
* Projekt nie je úplne načítaný.

**Riešenia:**

1. Počkajte, kým sa backend úplne inicializuje (skontrolujte ikonu hlavného menu).
2. Overte, či sú obrázky importované v prehliadači súborov.
3. Ak tlačidlo zostáva deaktivované, reštartujte Chloros.
4. Skontrolujte protokol ladenia, či neobsahuje chybové správy.

### Spracovanie sa spustí, ale ihneď zlyhá

**Možné príčiny:**

* V projekte nie sú platné obrázky
* Poškodené obrazové súbory
* Nedostatočný priestor na disku
* Nedostatočná pamäť (RAM)

**Riešenia:**

1. Skontrolujte protokol ladenia <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> , či neobsahuje chybové hlásenia.
2. Skontrolujte, či je k dispozícii dostatok miesta na disku.
3. Skúste spracovať menšiu podmnožinu obrázkov.
4. Skontrolujte, či obrázky nie sú poškodené.

### Upozornenie „Nebol zistený žiadny cieľ“

**Možné príčiny:**

* Zabudli ste označiť cieľové obrázky.
* Cieľové obrázky neobsahujú viditeľné ciele.
* Nastavenia detekcie cieľa sú príliš prísne.

**Riešenia:**

1. Prečítajte si [Výber cieľových obrázkov](choosing-target-images.md)
2. Označte príslušné obrázky v stĺpci Cieľ
3. Overte, či sú ciele viditeľné v označených obrázkoch
4. V prípade potreby upravte nastavenia detekcie cieľov

***

## Tipy na úspešné spracovanie

### Pred začatím

1. **Najskôr otestujte na malej podmnožine** – spracujte 10–20 obrázkov, aby ste overili nastavenia.
2. **Skontrolujte voľné miesto na disku** – zabezpečte 2–3-násobok veľkosti dátového súboru.
3. **Zatvorte nepotrebné aplikácie** – uvoľnite systémové zdroje.
4. **Overte cieľové obrázky** – skontrolujte náhľad označených cieľov, aby ste zabezpečili kvalitu.
5. **Uložte projekt** – projekt sa ukladá automaticky, ale je dobré ho uložiť aj ručne.

### Počas spracovania

1. **Vyhnite sa režimu spánku systému** – deaktivujte režimy úspory energie.
2. **Nechajte Chloros v popredí** – alebo aspoň viditeľný na paneli úloh.
3. **Občas skontrolujte priebeh** – skontrolujte, či sa nezobrazujú varovania alebo chyby.
4. **Nenačítavajte iné náročné aplikácie** – najmä v paralelnom režime Chloros+.

### Chloros+ GPU akcelerácia

Ak používate NVIDIA GPU akceleráciu:

1. Aktualizujte ovládače NVIDIA na najnovšiu verziu.
2. Uistite sa, že GPU má 4 GB+ VRAM.
3. Zatvorte aplikácie náročné na GPU (hry, editovanie videa).
4. Sledujte teplotu GPU (zabezpečte dostatočné chladenie).

***

## Ďalšie kroky

Po začatí spracovania:

1. **Sledujte priebeh** – pozrite si [Sledovanie spracovania](monitoring-the-processing.md).
2. **Počkajte na dokončenie** – spracovanie prebieha automaticky.
3. **Skontrolujte výsledky** – pozri [Dokončenie spracovania](finishing-the-processing.md).

Informácie o tom, čo robiť počas spracovania, nájdete v časti [Monitorovanie spracovania](monitoring-the-processing.md).

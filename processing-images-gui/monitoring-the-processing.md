# Sledovanie spracovania

Po spustení spracovania ponúka Chloros viacero spôsobov, ako sledovať priebeh, kontrolovať prípadné problémy a zistiť, čo sa deje s vašim súborom údajov. Na tejto stránke sa dozviete, ako sledovať priebeh spracovania a interpretovať informácie, ktoré poskytuje Chloros.

## Prehľad ukazovateľa priebehu

Ukazovateľ priebehu v hornom záhlaví zobrazuje stav spracovania v reálnom čase a percentuálnu úroveň dokončenia.

### Ukazovateľ priebehu v bezplatnom režime

Pre používateľov bez licencie Chloros+:

**Zobrazenie priebehu v 2 fázach:**

1.**Detekcia cieľov** – vyhľadávanie kalibračných cieľov v obrázkoch
2. **Spracovanie** – Uplatňovanie korekcií a export**Indikátor priebehu zobrazuje:**

* Celkové percento dokončenia (0–100 %)
* Názov aktuálnej fázy
* Jednoduchú vizualizáciu vo forme horizontálneho pruhu

### Indikátor priebehu Chloros+

Pre používateľov s licenciou Chloros+:

**4-fázové zobrazenie priebehu:**

1.**Detekcia** – vyhľadávanie kalibračných cieľov
2. **Analýza** – preskúmanie obrázkov a príprava spracovateľského reťazca
3. **Kalibrácia** – aplikovanie korekcií vinetácie a odrazivosti
4. **Export** – ukladanie spracovaných súborov**Interaktívne funkcie:*** **Naveďte kurzor** na indikátor priebehu, aby sa zobrazil rozbalený 4-fázový panel
* **Kliknite** na indikátor priebehu, aby sa rozbalený panel zmrazil/pripol
* **Kliknite znova**, aby sa rozmrazil a automaticky skryl pri odsunutí myši
* Každá fáza zobrazuje individuálny priebeh (0–100 %)

***

## Vysvetlenie jednotlivých fáz spracovania

{% hint style="info" %}
**Architektúra potrubia**: Tieto 4 fázy grafického používateľského rozhrania zodpovedajú [4-vláknovému spracovateľskému potrubiu](../processing-architecture/processing-pipeline.md). V systémoch s akceleráciou GPU využíva vlákno 3 (Kalibrácia) výhody [Dynamickej adaptácie výpočtov](../processing-architecture/dynamic-compute-adaptation.md), ktorá optimalizuje spracovanie pre váš konkrétny hardvér.
{% endhint %}

### Fáza 1: Detekcia (detekcia cieľa)

**Čo sa deje:**

* Chloros skenuje obrázky označené zaškrtávacím políčkom Cieľ
* Algoritmy počítačového videnia identifikujú 4 kalibračné panely
* Hodnoty odrazivosti extrahované z každého panelu
* Zaznamenávanie časových značiek cieľov pre správne naplánovanie kalibrácie

**Trvanie:**

* S označenými cieľmi: 10–60 sekúnd
* Bez označených cieľov: 5–30+ minút (skenuje všetky obrázky)

**Indikátor priebehu:**

* Detekcia: 0 % → 100 %
* Počet naskenovaných obrázkov
* Počet nájdených cieľov

**Na čo si dať pozor:**

* Ak sú ciele správne označené, malo by sa to dokončiť rýchlo
* Ak to trvá príliš dlho, ciele môžu byť neoznačené
* Skontrolujte protokol ladenia, či sa v ňom nachádzajú správy „Cieľ nájdený“

### Fáza 2: Analýza

**Čo sa deje:**

* Čítanie metadát EXIF obrázkov (časové značky, nastavenia expozície)
* Určenie stratégie kalibrácie na základe časových údajov cieľov
* Usporiadanie fronty spracovania obrázkov
* Príprava procesorov pre paralelné spracovanie (len Chloros+)

**Trvanie:** 5–30 sekúnd**Indikátor priebehu:**

* Analýza: 0 % → 100 %
* Rýchla fáza, zvyčajne sa dokončí rýchlo

**Na čo si dávať pozor:**

* Mal by prebiehať plynule bez prestávok
* V debug logu sa zobrazia varovania o chýbajúcich metadátach

### Fáza 3: Kalibrácia

**Čo sa deje:*** **Debayering**: Konverzia RAW Bayerovho vzoru na 3 kanály
* **Korekcia vinetácie**: Odstránenie stmavnutia okrajov objektívu
* **Kalibrácia odrazivosti**: Normalizácia pomocou cieľových hodnôt
* **Výpočet indexu**: Výpočet multispektrálnych indexov
* Spracovanie každého obrázku cez celý proces

**Trvanie:** Väčšina celkového času spracovania (60–80 %)**Indikátor priebehu:**

* Kalibrácia: 0 % → 100 %
* Aktuálny spracovávaný obrázok
* Dokončené obrázky / Celkový počet obrázkov

**Správanie pri spracovaní:*** **Voľný režim**: Spracováva jeden obrázok po druhom postupne
* **Režim Chloros+**: Spracováva až 16 obrázkov súčasne
* **Akcelerácia GPU**: Výrazne urýchľuje túto fázu**Na čo si dávať pozor:**

* Plynulý postup v počte obrázkov
* Skontrolujte protokol ladenia (Debug Log) pre správy o dokončení jednotlivých obrázkov
* Upozornenia týkajúce sa kvality obrázkov alebo problémov s kalibráciou

### Fáza 4: Export

**Čo sa deje:**

* Zapisovanie kalibrovaných snímok na disk vo vybranom formáte
* Export multispektrálnych indexových snímok s farbami LUT
* Vytváranie podpriečinkov modelov kamier
* Zachovanie pôvodných názvov súborov s príslušnými príponami

**Trvanie:** 10–20 % celkového času spracovania**Indikátor priebehu:**

* Export: 0 % → 100 %
* Súbory sa zapisujú
* Formát exportu a cieľová destinácia

**Na čo si dávať pozor:**

* Upozornenia na nedostatok miesta na disku
* Chyby pri zápise súborov
* Dokončenie všetkých nakonfigurovaných výstupov

***

## Karta Debug Log (Log ladenia)

Log ladenia poskytuje podrobné informácie o priebehu spracovania a všetkých zistených problémoch.

### Prístup k logu ladenia

1. Kliknite na ikonu **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> v ľavom bočnom paneli
2. Otvorí sa panel protokolu, ktorý zobrazuje správy o spracovaní v reálnom čase
3. Automaticky sa posúva, aby zobrazoval najnovšie správy

### Vysvetlenie správ protokolu

#### Informačné správy (biele/šedé)

Bežné aktualizácie spracovania:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Varovné správy (žlté)

Nekritické problémy, ktoré nezastavia spracovanie:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Akcia:** Preverte varovania po spracovaní, ale nepretrhávajte ho

#### Chybové správy (Red)

Kritické problémy, ktoré môžu spôsobiť zlyhanie spracovania:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Akcia:** Zastavte spracovanie, odstráňte chybu a reštartujte

### Bežné správy protokolu

| Správa                          | Význam                                | Potrebná akcia                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| „Cieľ zistený v [názov súboru]“ | Cieľ kalibrácie bol úspešne nájdený  | Žiadne – normálne                                         |
| „Spracovávanie obrázku X z Y“        | Aktualizácia aktuálneho priebehu                | Žiadne – normálne                                         |
| „Nenašli sa žiadne ciele“               | Neboli zistené žiadne ciele kalibrácie        | Označte cieľové obrázky alebo deaktivujte kalibráciu odrazivosti |
| „Nedostatočný priestor na disku“        | Nedostatok úložného priestoru pre výstup          | Uvoľnite miesto na disku                                    |
| „Preskakovanie poškodeného súboru“        | Obrazový súbor je poškodený                  | Znovu skopírujte súbor z SD karty                             |
| „PPK údaje použité“               | Použité GPS korekcie zo súboru .daq | Žiadne – normálne                                         |

### Kopírovanie údajov protokolu

Kopírovanie protokolu na účely riešenia problémov alebo podpory:

1. Otvorte panel Debug Log (Protokol ladenia)
2. Kliknite na tlačidlo **„Copy Log“** (Kopírovať protokol) (alebo kliknite pravým tlačidlom myši → Vybrat všetko)
3. Vložte do textového súboru alebo e-mailu
4. V prípade potreby pošlite na podporu MAPIR

***

## Monitorovanie systémových zdrojov

### Využitie procesora

**Režim Free:**

* 1 jadro procesora na ~100 %
* Ostatné jadrá sú neaktívne alebo dostupné
* Systém zostáva odozvový

**Režim Chloros+ Parallel:**

* Viac jadier na 80–100 % (až 16 jadier)
* Vysoké celkové využitie procesora
* Systém môže pôsobiť menej odozvovo

**Ako monitorovať:**

* Správca úloh Windows (Ctrl+Shift+Esc)
* Karta Výkon → sekcia CPU
* Hľadajte procesy „Chloros“ alebo „chloros-backend“

### Využitie pamäte (RAM)

**Typické využitie:**

* Malé projekty (&lt; 100 obrázkov): 2–4 GB
* Stredné projekty (100–500 obrázkov): 4–8 GB
* Veľké projekty (500+ obrázkov): 8–16 GB
* Paralelný režim Chloros+ využíva viac RAM

**Ak je málo pamäte:**

* Spracúvajte menšie dávky
* Zatvorte ostatné aplikácie
* Ak pravidelne spracúvate veľké súbory údajov, zvoľte si väčšiu RAM

### Využitie GPU (Chloros+ s CUDA)

Keď je zapnuté GPU zrýchlenie:

* GPU NVIDIA vykazuje vysoké využitie (60–90 %)
* Využitie VRAM sa zvyšuje (vyžaduje 4 GB+ VRAM)
* Fáza kalibrácie je výrazne rýchlejšia

**Na sledovanie:**

* Ikona NVIDIA v systémovej lište
* Správca úloh → Výkon → GPU
* GPU-Z alebo podobný monitorovací nástroj

### Diskové I/O

**Čo môžete očakávať:**

* Vysoká čítacia rýchlosť disku počas fázy analýzy
* Vysoká rýchlosť zápisu na disk počas fázy exportu
* SSD je výrazne rýchlejší ako HDD

**Tip na zvýšenie výkonu:**

* Ak je to možné, použite SSD pre priečinok projektu
* Vyhnite sa sieťovým diskom pre veľké dátové súbory
* Uistite sa, že disk nie je takmer plný (ovplyvňuje rýchlosť zápisu)

***

## Zisťovanie problémov počas spracovania

### Varovné signály

**Zastavenie priebehu (žiadna zmena po dobu 5 a viac minút):**

* Skontrolujte protokol ladenia, či neobsahuje chyby
* Overte dostupný priestor na disku
* Skontrolujte Správcu úloh, či beží proces Chloros

**Časté zobrazovanie chybových správ:**

* Zastavte spracovanie a skontrolujte chyby
* Bežné príčiny: priestor na disku, poškodené súbory, problémy s pamäťou
* Pozrite si časť Riešenie problémov nižšie

**Systém nereaguje:**

* Paralelný režim Chloros+ využíva príliš veľa zdrojov
* Zvážte zníženie počtu súbežných úloh alebo modernizáciu hardvéru
* Voľný režim je menej náročný na zdroje

### Kedy zastaviť spracovanie

Zastavte spracovanie, ak sa zobrazí:

* ❌ Chyby „Disk je plný“ alebo „Nie je možné zapísať súbor“
* ❌ Opakované chyby poškodenia obrazových súborov
* ❌ Systém úplne zamrzol (nereaguje)
* ❌ Zistili ste, že boli nakonfigurované nesprávne nastavenia
* ❌ Importovali ste nesprávne obrazy

**Ako zastaviť:**

1. Kliknite na**tlačidlo Zastaviť/Zrušiť** (nahrádza tlačidlo Štart)
2. Spracovanie sa zastaví, pokrok sa stratí
3. Opravte problémy a začnite od začiatku

***

## Riešenie problémov počas spracovania

### Spracovanie je veľmi pomalé

**Možné príčiny:**

* Neoznačené cieľové obrázky (skenovanie všetkých obrázkov)
* HDD namiesto SSD úložiska
* Nedostatočné systémové zdroje
* Nastavených je veľa indexov
* Prístup k sieťovému disku

**Riešenia:**

1. Ak ste práve začali a ste vo fáze detekcie: Zrušte, označte ciele, reštartujte
2. Do budúcnosti: Použite SSD, znížte počet indexov, modernizujte hardvér
3. Zvážte použitie CLI na hromadné spracovanie veľkých dátových súborov

### Upozornenia na „miesto na disku“

**Riešenia:**

1. Okamžite uvoľnite miesto na disku
2. Presuňte projekt na disk s väčším priestorom
3. Znížte počet indexov na export
4. Použite formát JPG namiesto TIFF (menšie súbory)

### Časté správy o „poškodených súboroch“

**Riešenia:**

1. Znovu skopírujte obrázky z SD karty, aby ste zaistili ich integritu
2. Skontrolujte SD kartu na chyby
3. Odstráňte poškodené súbory z projektu
4. Pokračujte v spracovaní zostávajúcich obrázkov

### Prehrievanie systému / Obmedzenie výkonu

**Riešenia:**

1. Zabezpečte dostatočné vetranie
2. Odstráňte prach z ventilačných otvorov počítača
3. Znížte zaťaženie pri spracovaní (použite režim Free namiesto Chloros+)
4. Spracúvajte v chladnejších častiach dňa

***

## Oznámenie o dokončení spracovania

Keď sa spracovanie dokončí:

* Ukazovateľ priebehu dosiahne 100 %
* V lógovom súbore sa zobrazí správa **„Spracovanie dokončené“**
* Tlačidlo Štart sa opäť aktivuje
* Všetky výstupné súbory sa nachádzajú v podpriečinku modelu fotoaparátu

***

## Ďalšie kroky

Po dokončení spracovania:

1. **Skontrolujte výsledky** – pozrite si [Dokončenie spracovania](finishing-the-processing.md)
2. **Skontrolujte výstupnú zložku** – overte, či boli všetky súbory správne exportované
3. **Prezrite si protokol ladenia** – skontrolujte, či sa v ňom nenachádzajú žiadne varovania alebo chyby
4. **Prezrite si náhľad spracovaných obrázkov** – použite prehliadač obrázkov alebo externý softvér

Informácie o prezeraní a používaní spracovaných výsledkov nájdete v článku [Dokončenie spracovania](finishing-the-processing.md).

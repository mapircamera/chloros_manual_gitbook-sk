# Sledovanie spracovania

Po začatí spracovania ponúka Chloros niekoľko spôsobov, ako sledovať priebeh, kontrolovať problémy a porozumieť tomu, čo sa deje s vašim súborom údajov. Na tejto stránke je vysvetlené, ako sledovať spracovanie a interpretovať informácie, ktoré poskytuje Chloros.

## Prehľad ukazovateľa priebehu

Indikátor priebehu v hornej hlavičke zobrazuje stav spracovania v reálnom čase a percento dokončenia.

### Indikátor priebehu v bezplatnom režime

Pre používateľov bez licencie Chloros+:

**Zobrazenie priebehu v 2 fázach:**

1. **Detekcia cieľa** – vyhľadávanie kalibračných cieľov v obrazoch
2. **Spracovanie** – aplikovanie korekcií a export

**Indikátor priebehu zobrazuje:**

* Celkové percento dokončenia (0–100 %)
* Názov aktuálnej fázy
* Jednoduchá vizualizácia horizontálneho pruhu

### Indikátor priebehu Chloros

Pre používateľov s licenciou Chloros+:

**4-fázové zobrazenie priebehu:**

1. **Detekcia** – hľadanie kalibračných cieľov
2. **Analýza** – skúmanie obrázkov a príprava potrubia
3. **Kalibrácia** – aplikovanie korekcií vinetácie a odrazivosti
4. **Export** – ukladanie spracovaných súborov

**Interaktívne funkcie:**

* **Naveďte kurzor na** ukazovateľ priebehu, aby sa zobrazil rozšírený 4-fázový panel
* **Kliknutím** na ukazovateľ priebehu zmrazíte/pripnete rozšírený panel
* **Ďalším kliknutím** ho odzmrazíte a automaticky skryjete po odstránení kurzora myši
* Každá fáza zobrazuje individuálny priebeh (0–100 %)

***

## Vysvetlenie jednotlivých fáz spracovania

### Fáza 1: Detekcia (detekcia cieľov)

**Čo sa deje:**

* Chloros skenuje obrázky označené zaškrtávacím políčkom Cieľ
* Algoritmy počítačového videnia identifikujú 4 kalibračné panely
* Hodnoty odrazivosti extrahované z každého panelu
* Časové značky cieľov zaznamenané pre správne naplánovanie kalibrácie

**Trvanie:**

* S označenými cieľmi: 10–60 sekúnd
* Bez označených cieľov: 5–30+ minút (skenuje všetky obrázky)

**Indikátor priebehu:**

* Detekcia: 0 % → 100 %
* Počet naskenovaných obrázkov
* Počet nájdených cieľov

**Na čo si dávať pozor:**

* Ak sú ciele správne označené, proces by mal byť rýchlo dokončený.
* Ak to trvá príliš dlho, ciele môžu byť neoznačené.
* Skontrolujte protokol ladenia, či sa v ňom nachádzajú správy „Cieľ nájdený“.

### Fáza 2: Analýza

**Čo sa deje:**

* Čítanie metadát EXIF obrázkov (časové značky, nastavenia expozície)
* Určenie stratégie kalibrácie na základe časových značiek cieľov
* Organizovanie fronty spracovania obrázkov
* Príprava paralelných spracovateľov (len Chloros+)

**Trvanie:** 5–30 sekúnd

**Indikátor priebehu:**

* Analýza: 0 % → 100 %
* Rýchla fáza, zvyčajne sa dokončí rýchlo

**Na čo si dávať pozor:**

* Pokrok by mal prebiehať plynule bez prestávok
* Upozornenia na chýbajúce metadáta sa zobrazia v protokole ladenia

### Fáza 3: Kalibrácia

**Čo sa deje:**

* **Debayering**: Konverzia vzoru RAW Bayer na 3 kanály
* **Korekcia vinetácie**: Odstránenie stmavnutia okrajov objektívu
* **Kalibrácia odrazivosti**: Normalizácia pomocou cieľových hodnôt
* **Výpočet indexu**: Výpočet multispektrálnych indexov
* Spracovanie každého obrázku prostredníctvom celého procesu

**Trvanie:** Väčšina celkového času spracovania (60–80 %)

**Indikátor priebehu:**

* Kalibrácia: 0 % → 100 %
* Spracovávanie aktuálneho obrazu
* Dokončené obrazy / Celkový počet obrazov

**Správanie spracovania:**

* **Voľný režim**: Spracováva jeden obraz za druhým postupne
* **Režim Chloros+**: Spracováva až 16 obrazov súčasne
* **GPU akcelerácia**: Výrazne urýchľuje túto fázu.

**Na čo si dávať pozor:**

* Stabilný pokrok v počte obrázkov.
* Skontrolujte protokol ladenia, či obsahuje správy o dokončení jednotlivých obrázkov.
* Upozornenia týkajúce sa kvality obrázkov alebo problémov s kalibráciou.

### Fáza 4: Exportovanie

**Čo sa deje:**

* Zapisovanie kalibrovaných obrázkov na disk vo vybranom formáte
* Exportovanie multispektrálnych indexových obrázkov s farbami LUT
* Vytváranie podadresárov modelov kamier
* Zachovanie pôvodných názvov súborov s príslušnými príponami

**Trvanie:** 10–20 % celkového času spracovania

**Indikátor pokroku:**

* Exportovanie: 0 % → 100 %
* Zapisovanie súborov
* Formát exportu a cieľová destinácia

**Na čo si dávať pozor:**

* Upozornenia na nedostatok miesta na disku
* Chyby pri zápise súborov
* Dokončenie všetkých nakonfigurovaných výstupov

***

## Karta Debug Log (Logs)

Logs poskytuje podrobné informácie o priebehu spracovania a všetkých problémoch, ktoré sa vyskytli.

### Prístup k Logs

1. Kliknite na ikonu **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> v ľavom bočnom paneli.
2. Otvorí sa panel protokolu, ktorý zobrazuje správy o spracovaní v reálnom čase.
3. Automaticky sa posúva, aby zobrazoval najnovšie správy.

### Porozumenie správam protokolu

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

**Akcia:** Po spracovaní skontrolujte varovania, ale neprerušujte spracovanie.

#### Chybové správy (Red)

Kritické problémy, ktoré môžu spôsobiť zlyhanie spracovania:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Akcia:** Zastavte spracovanie, odstráňte chybu a reštartujte.

### Bežné hlásenia protokolu

| Hlásenie                          | Význam                                | Potrebná akcia                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| „Cieľ zistený v \[názov súboru]“ | Kalibračný cieľ úspešne nájdený  | Žiadne – normálne                                         |
| „Spracovanie obrázku X z Y“        | Aktualizácia aktuálneho priebehu                | Žiadne – normálne                                         |
| „Nenájdené žiadne ciele“               | Nezistené žiadne kalibračné ciele        | Označte cieľové obrázky alebo deaktivujte kalibráciu odrazivosti |
| „Nedostatočný priestor na disku“        | Nedostatočná pamäťová kapacita pre výstup          | Uvoľnite miesto na disku                                    |
| „Preskakuje sa poškodený súbor“        | Obrazový súbor je poškodený                  | Znovu skopírujte súbor z SD karty                             |
| „Použité údaje PPK“               | Použité korekcie GPS zo súboru .daq | Žiadne – normálne                                         |

### Kopírovanie údajov protokolu

Kopírovanie protokolu na účely riešenia problémov alebo podpory:

1. Otvorte panel Debug Log (Protokol ladenia).
2. Kliknite na tlačidlo **„Copy Log“ (Kopírovať protokol)** (alebo kliknite pravým tlačidlom myši → Vybrať všetko).
3. Vložte do textového súboru alebo e-mailu.
4. V prípade potreby pošlite na podporu MAPIR.

***

## Monitorovanie systémových zdrojov

### Využitie CPU

**Voľný režim:**

* 1 jadro CPU na ~100 %
* Ostatné jadrá sú neaktívne alebo dostupné
* Systém zostáva odozvový

**Chloros+ Paralelný režim:**

* Viacero jadier na 80–100 % (až 16 jadier)
* Vysoké celkové využitie CPU
* Systém môže reagovať pomalšie

**Monitorovanie:**

* Windows Správca úloh (Ctrl+Shift+Esc)
* Karta Výkon → sekcia CPU
* Vyhľadajte procesy „Chloros“ alebo „chloros-backend“

### Využitie pamäte (RAM)

**Typické využitie:**

* Malé projekty (&lt; 100 obrázkov): 2–4 GB
* Stredné projekty (100–500 obrázkov): 4–8 GB
* Veľké projekty (500+ obrázkov): 8–16 GB
* Chloros+ paralelný režim využíva viac RAM

**Ak je pamäť nedostatočná:**

* Spracúvajte menšie dávky
* Zatvorte ostatné aplikácie
* Ak pravidelne spracúvate veľké dátové súbory, zvýšte kapacitu pamäte RAM

### Využitie GPU (Chloros+ s CUDA)

Keď je povolené zrýchlenie GPU:

* GPU NVIDIA vykazuje vysoké využitie (60–90 %)
* Zvýšené využitie VRAM (vyžaduje 4 GB+ VRAM)
* Fáza kalibrácie je výrazne rýchlejšia

**Na monitorovanie:**

* Ikona NVIDIA v systémovej lište
* Správca úloh → Výkon → GPU
* GPU-Z alebo podobný monitorovací nástroj

### Diskové I/O

**Čo môžete očakávať:**

* Vysoké čítanie disku počas fázy analýzy
* Vysoké zapisovanie disku počas fázy exportu
* SSD je výrazne rýchlejší ako HDD

**Tip na zvýšenie výkonu:**

* Ak je to možné, používajte SSD pre projektovú zložku
* Vyhnite sa sieťovým diskom pre veľké dátové súbory
* Uistite sa, že disk nie je takmer plný (ovplyvňuje rýchlosť zápisu)

***

## Detekcia problémov počas spracovania

### Varovné signály

**Zastavenie priebehu (bez zmeny po dobu 5 a viac minút):**

* Skontrolujte protokol ladenia, či neobsahuje chyby.
* Overte, či je k dispozícii dostatok miesta na disku.
* Skontrolujte Správcu úloh, či beží Chloros.

**Často sa zobrazujú chybové správy:**

* Zastavte spracovanie a skontrolujte chyby.
* Bežné príčiny: miesto na disku, poškodené súbory, problémy s pamäťou.
* Pozrite si časť Riešenie problémov nižšie.

**Systém nereaguje:**

* Chloros+ paralelné režimy používajú príliš veľa zdrojov.
* Zvážte zníženie počtu súbežných úloh alebo upgrade hardvéru.
* Voľný režim je menej náročný na zdroje.

### Kedy zastaviť spracovanie

Spracovanie zastavte, ak sa zobrazí:

* ❌ Chyba „Disk je plný“ alebo „Nie je možné zapísať súbor“
* ❌ Opakované chyby poškodenia obrazových súborov
* ❌ Systém úplne zamrzol (nereaguje)
* ❌ Zistili ste, že boli nakonfigurované nesprávne nastavenia
* ❌ Importovali sa nesprávne obrazy

**Ako zastaviť:**

1. Kliknite na **tlačidlo Zastaviť/Zrušiť** (nahradzuje tlačidlo Štart)
2. Spracovanie sa zastaví, pokrok sa stratí
3. Opravte problémy a začnite od začiatku

***

## Riešenie problémov počas spracovania

### Spracovanie je veľmi pomalé

**Možné príčiny:**

* Neoznačené cieľové obrázky (skenovanie všetkých obrázkov)
* HDD namiesto SSD úložiska
* Nedostatočné systémové zdroje
* Veľa nakonfigurovaných indexov
* Prístup k sieťovému disku

**Riešenia:**

1. Ak ste práve začali a ste vo fáze detekcie: Zrušte, označte ciele, reštartujte
2. Do budúcnosti: Použite SSD, znížte počet indexov, aktualizujte hardvér
3. Zvážte použitie CLI na hromadné spracovanie veľkých dátových súborov

### Upozornenia „Disk Space“ (Miesto na disku)

**Riešenia:**

1. Okamžite uvoľnite miesto na disku
2. Presuňte projekt na disk s väčším priestorom
3. Znížte počet indexov na export.
4. Použite formát JPG namiesto TIFF (menšie súbory).

### Časté správy „Poškodený súbor“

**Riešenia:**

1. Znovu skopírujte obrázky z SD karty, aby ste zaistili integritu.
2. Skontrolujte SD kartu, či neobsahuje chyby.
3. Odstráňte poškodené súbory z projektu.
4. Pokračujte v spracovaní zostávajúcich obrázkov.

### Prehrievanie/obmedzovanie systému

**Riešenia:**

1. Zabezpečte dostatočné vetranie.
2. Odstráňte prach z vetracích otvorov počítača.
3. Znížte zaťaženie spracovaním (použite režim Free namiesto Chloros+).
4. Spracovávajte v chladnejších častiach dňa.

***

## Oznámenie o dokončení spracovania

Po dokončení spracovania:

* Ukazovateľ priebehu dosiahne 100 %
* V protokole ladenia sa zobrazí správa **„Spracovanie dokončené“**
* Tlačidlo Štart sa opäť aktivuje
* Všetky výstupné súbory sa nachádzajú v podsložke modelu fotoaparátu

***

## Ďalšie kroky

Po dokončení spracovania:

1. **Skontrolujte výsledky** – pozrite si časť [Dokončenie spracovania](finishing-the-processing.md)
2. **Skontrolujte výstupnú zložku** – overte, či boli všetky súbory správne exportované
3. **Skontrolujte protokol ladenia** – skontrolujte, či sa nezobrazujú žiadne varovania alebo chyby
4. **Prezrite si spracované obrázky** – použite prehliadač obrázkov alebo externý softvér

Informácie o kontrole a používaní spracovaných výsledkov nájdete v časti [Dokončenie spracovania](finishing-the-processing.md).

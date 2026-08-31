# Sledovanie spracovania

Po spustení spracovania ponúka Chloros viacero spôsobov, ako sledovať priebeh, kontrolovať prípadné problémy a zistiť, čo sa deje s vašou dátovou sadou. Na tejto stránke sa dozviete, ako sledovať priebeh spracovania a interpretovať informácie, ktoré poskytuje Chloros.

## Prehľad ukazovateľa priebehu

Ukazovateľ priebehu v hornej lište zobrazuje stav spracovania v reálnom čase a percentuálnu úroveň dokončenia. Priebeh sa prenáša naživo z backendu prostredníctvom Server-Sent Events (SSE), takže ukazovateľ odráža to, čo spracovateľský reťazec skutočne robí.

### Ukazovateľ priebehu v bezplatnom režime

Pre používateľov bez licencie Chloros+:

**Zobrazenie priebehu v 2 fázach:**

1.**Detekcia cieľov** – vyhľadávanie kalibračných cieľov v obrázkoch
2. **Spracovanie** – Uplatňovanie korekcií a export**Indikátor priebehu zobrazuje:**

* Celkové percento dokončenia (0–100 %)
* Názov aktuálnej fázy
* Jednoduché vizualizovanie pomocou horizontálneho pruhu

### Indikátor priebehu Chloros+

Pre používateľov s licenciou Chloros+:

**Zobrazenie priebehu v 4 fázach:**

1.**Detekcia** – vyhľadávanie kalibračných cieľov
2. **Analýza** – skúmanie obrázkov a príprava spracovateľského reťazca
3. **Kalibrácia** – Aplikácia korekcií vinety a odrazivosti
4. **Export** – Ukladanie spracovaných súborov**Interaktívne funkcie:*** **Naveďte kurzor** na indikátor priebehu, aby sa zobrazil rozbalený 4-fázový panel
* **Kliknutím** na indikátor priebehu zamrazíte/pripnete rozbalený panel
* **Kliknite znovu** na uvoľnenie a automatické skrytie pri odsunutí kurzora
* Každá fáza zobrazuje individuálny priebeh (0–100 %)

{% hint style="info" %}
**CLI parita**: počas behu `chloros-cli process` hlásia tie isté štyri vlákna stavy „Detekcia“, „Analýza“, „Spracovanie“ a „Export“, pričom `chloros-cli export-status` zobrazuje priebeh exportu vlákna 4 v reálnom čase z iného terminálu. Pozrite si [Referenčnú príručku k programu CLI](../reference/cli-reference.md).
{% endhint %}

***

## Vysvetlenie jednotlivých fáz spracovania

{% hint style="info" %}
**Architektúra potrubia**: Tieto 4 fázy grafického používateľského rozhrania zodpovedajú [4-vláknovému spracovateľskému potrubiu](../processing-architecture/processing-pipeline.md). V systémoch s akceleráciou GPU využíva vlákno 3 (Kalibrácia) výhody [dynamickej adaptácie výpočtov](../processing-architecture/dynamic-compute-adaptation.md), ktorá optimalizuje spracovanie pre váš konkrétny hardvér.
{% endhint %}

### Fáza 1: Detekcia (detekcia cieľov)

**Čo sa deje:**

* Chloros skenuje obrázky, ktoré ste označili zaškrtnutím políčka Cieľ (všetky obrázky, len ak nie je zaškrtnutý žiadny)
* Algoritmy počítačového videnia identifikujú kalibračné panely
* Z každého panelu sa extrahujú hodnoty odrazivosti
* Zaznamenávajú sa časové značky cieľov pre správne naplánovanie kalibrácie

**Trvanie:**

* S označenými cieľmi: 10–60 sekúnd
* Bez označených cieľov: 5–30+ minút (skenuje všetky obrázky)

**Indikátor priebehu:**

* Detekcia: 0 % → 100 %
* Počet naskenovaných obrázkov (počíta sa len počet obrázkov, ktoré sa skutočne skenujú)
* Počet nájdených cieľov

**Na čo si dávať pozor:**

* Ak sú ciele správne označené, proces by mal prebehnúť rýchlo
* Ak to trvá príliš dlho, ciele môžu byť neoznačené
* Skontrolujte protokol ladenia (Debug Log) na prítomnosť správ „Target found“

### Fáza 2: Analýza

**Čo sa deje:**

* Čítanie metadát EXIF zo snímok (časové značky, nastavenia expozície)
* Určenie stratégie kalibrácie na základe časových značiek cieľov a dostupných údajov o dopade žiarenia z DAQ
* Usporiadanie fronty spracovania snímok
* Príprava procesov na paralelné spracovanie (len Chloros+)

**Trvanie:** 5–30 sekúnd**Indikátor priebehu:**

* Analýza: 0 % → 100 %
* Rýchla fáza, zvyčajne sa dokončí rýchlo

**Na čo si dávať pozor:**

* Priebeh by mal byť plynulý bez prestávok
* V ladiacom protokole sa zobrazia varovania o chýbajúcich metadátach

### Fáza 3: Kalibrácia

**Čo sa deje:*** **Debayering**: Konverzia vzoru RAW Bayer na 3 kanály (pri monochromatických moduloch LATTICE sa tento krok preskakuje, s poznámkou)
* **Korekcia vinetácie**: Odstránenie stmavnutia okrajov objektívu
* **Kalibrácia odrazivosti**: Normalizácia pomocou cieľových hodnôt a/alebo DAQ downwellingu
* **Výpočet indexov**: Výpočet multispektrálnych indexov
* Spracovanie každého snímku cez celý spracovateľský reťazec

**Trvanie:** Väčšina celkového času spracovania (60–80 %)**Indikátor priebehu:**

* Kalibrácia: 0 % → 100 %
* Práve sa spracúva aktuálny obraz
* Dokončené obrazy / Celkový počet obrazov

**Správanie pri spracovaní:*** **Voľný režim**: Spracováva jeden obraz po druhom postupne
* **Režim Chloros+**: Spúšťa skupinu pracovných procesov prispôsobenú hardvéru — 1–4 súbežné pracovné procesy na systémoch s GPU (podľa VRAM), jeden pracovný proces na fyzické jadro (mínus jeden) na systémoch iba s CPU. Pozri [Dynamická adaptácia výpočtov](../processing-architecture/dynamic-compute-adaptation.md)
* **Akcelerácia pomocou GPU**: Výrazne urýchľuje túto fázu**Na čo si dávať pozor:**

* Plynulý postup podľa počtu obrázkov
* Skontroluj ladiaci protokol, či sa zobrazujú správy o dokončení jednotlivých obrázkov
* Upozornenia týkajúce sa kvality obrázkov alebo problémov s kalibráciou

### Fáza 4: Export

**Čo sa deje:**

* Zapisovanie spracovaných obrázkov na disk vo vybranom formáte hneď po ich dokončení
* **LATTICE**: každý snímok sa rozdelí do všetkých povolených produktov (debayering / náhľad / radiancia / odrazivosť)
* Export multispektrálnych indexových snímok s farbami LUT
* Vytvorenie výstupného stromu `<project>/<camera>/<format>/<Product>_Images/` — exportované súbory si zachovávajú názov zdrojového súboru; zložka identifikuje produkt

**Trvanie:** 10–20 % celkového času spracovania**Indikátor priebehu:**

* Export: 0 % → 100 %
* Prebieha zapisovanie súborov
* Formát exportu a cieľová destinácia

**Na čo si dávať pozor:**

* Upozornenia na nedostatok miesta na disku
* Chyby pri zápise súborov
* Dokončenie všetkých nakonfigurovaných výstupov

***

## Karta Debug Log

Log ladenia poskytuje podrobné informácie o priebehu spracovania a všetkých zistených problémoch. Do konzoly logu sa tiež zaznamenávajú správy o spustení backendu, takže log poskytuje kompletný prehľad aj v prípade, že ho otvoríte neskôr.

### Prístup k logu ladenia

1. Kliknite na ikonu **Log ladiaceho protokolu**<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

v ľavom bočnom paneli
2. Otvorí sa panel protokolu, ktorý zobrazuje správy o spracovaní v reálnom čase
3. Obsah sa automaticky posúva, aby zobrazoval najnovšie správy

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written) -->

### Vysvetlenie správ v protokole

Riadky protokolu s predponou Chloros majú na začiatku v zátvorkách označenie subsystému – napríklad `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Najdôležitejšie je poznať **súhrn behu**, ktorý sa vytlačí na konci každého behu (vrátane zastavených behov):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Ďalšie riadky s tipmi typu `[RUN-SUMMARY]` nasledujú vždy, keď je potrebné niečo vysvetliť — napríklad beh, ktorý nevytvoril žiadne výsledky, alebo kamera, ktorej požadovaný produkt bol vynechaný ako neaplikovateľný. Riadky `[EXPORT-CHECK]` vysvetľujú vynechania pre jednotlivé kamery (napr. prečo kamera RGB nezískala produkt žiarivosti).

Všeobecné úrovne závažnosti správ (príklady nižšie sú ilustratívne, nie doslovné):

#### Informačné správy (biele/šedé)

Bežné aktualizácie spracovania: spustenie spracovania, detekcia cieľov (s počtom panelov), priebeh kalibrácie jednotlivých snímok, export súborov, dokončenie spracovania.

#### Varovné správy (žltá)

Nekritické problémy, ktoré nezastavia spracovanie — napr. chýbajúce údaje GPS v snímke, veľký časový posun medzi snímkami cieľov alebo nízky kontrast v kalibračnom paneli.

**Akcia:** Po spracovaní skontrolujte varovania, ale spracovanie neprerušujte

#### Chybové správy (Red)

Kritické problémy, ktoré môžu spôsobiť zlyhanie spracovania – napr. zaplnený disk, poškodený obrazový súbor alebo nezistenie žiadnych cieľov pri požiadavke na kalibráciu odrazivosti.

**Postup:** Zastavte spracovanie, odstráňte chybu a spustite spracovanie znova

### Bežné situácie v protokole

| Situácia                             | Význam                                       | Potrebný postup                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Cieľ zistený v súbore \[názov súboru]        | Cieľ kalibrácie bol úspešne nájdený         | Žiadne – normálny stav                                         |
| Ukazovatele priebehu na jednotlivých snímkach              | Aktuálna aktualizácia priebehu                       | Žiadne – normálny stav                                         |
| Nenašli sa žiadne ciele                      | Neboli zistené žiadne kalibračné ciele               | Označte snímky cieľov alebo deaktivujte kalibráciu odrazivosti |
| Nedostatočný priestor na disku               | Nedostatok miesta na ukladanie výstupných dát                 | Uvoľnite miesto na disku                                    |
| Preskakovanie poškodeného súboru               | Obrazový súbor je poškodený                         | Znovu skopírujte súbor z SD karty                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Snímku bez surového rámca nie je možné spracovať | Znovu nasnímajte so surovým rámcom, alebo použite CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Spustenie nevytvorilo žiadne obrazové výstupy — hlásené ako zlyhanie s tipmi | Prečítajte si riadky s tipmi; skontrolujte, čo bolo preskočené a prečo |

### Kopírovanie údajov z protokolu

Ak chcete skopírovať protokol na účely riešenia problémov alebo technickej podpory:

1. Otvorte panel Debug Log
2. Kliknite na tlačidlo **„Copy Log“** (alebo kliknite pravým tlačidlom myši → Vybrat všetko)
3. Vložte do textového súboru alebo do e-mailu
4. V prípade potreby pošlite na podporu MAPIR

***

## Monitorovanie systémových zdrojov

### Využitie procesora

**Voľný režim:**

* 1 jadro procesora na ~100 %
* Ostatné jadrá sú v nečinnosti alebo sú k dispozícii
* Systém naďalej reaguje

**Paralelný režim Chloros+:**

* Viac jadier s vysokým využitím — ich počet závisí od stratégie zvolenej prostredníctvom [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* Systém môže pôsobiť menej odozvivo

**Na sledovanie:**

* Správca úloh (Ctrl+Shift+Esc)
* Karta Výkon → sekcia CPU
* Hľadajte procesy „Chloros“ alebo „chloros-backend“

### Využitie pamäte (RAM)

**Typické využitie:**

* Malé projekty (&lt; 100 obrázkov): 2–4 GB
* Stredné projekty (100–500 obrázkov): 4–8 GB
* Veľké projekty (500 a viac obrázkov): 8–16 GB
* Paralelný režim Chloros+ spotrebuje viac pamäte RAM

**Ak je málo pamäte:**

* Spracúvajte menšie dávky
* Zatvorte ostatné aplikácie
* Ak pravidelne spracúvate veľké súbory údajov, rozšírte pamäť RAM

### Využitie GPU (Chloros+ s CUDA)

Keď je povolené akcelerovanie pomocou GPU:

* GPU NVIDIA vykazuje vysoké využitie (60–90 %)
* Zvyšuje sa využitie VRAM (vyžaduje sa 4 GB a viac VRAM; 7 GB a viac pre súbežné odstraňovanie bayeringu s podporou textúr)
* Fáza kalibrácie je výrazne rýchlejšia

**Na sledovanie:**

* Ikona NVIDIA v systémovej lište
* Správca úloh → Výkon → Grafická karta
* GPU-Z alebo podobný monitorovací nástroj

### Vstupno-výstupné operácie na disku

**Čo môžete očakávať:**

* Vysoká rýchlosť čítania z disku počas fázy analýzy
* Vysoká rýchlosť zápisu na disk počas fázy exportu
* SSD je výrazne rýchlejší ako HDD

**Tip na zvýšenie výkonu:**

* Ak je to možné, použite SSD pre priečinok s projektom
* Pri veľkých dátových súboroch sa vyhnite sieťovým diskom
* Uistite sa, že disk nie je takmer plný (ovplyvňuje to rýchlosť zápisu)

***

## Zisťovanie problémov počas spracovania

### Varovné signály

**Progres sa zastaví (žiadna zmena po dobu 5 a viac minút):**

* Skontrolujte protokol ladenia, či neobsahuje chyby
* Overte, či je k dispozícii dostatok miesta na disku
* Skontrolujte Správcu úloh, či beží proces Chloros

**Často sa zobrazujú chybové hlásenia:**

* Zastavte spracovanie a skontrolujte chyby
* Bežné príčiny: miesto na disku, poškodené súbory, problémy s pamäťou
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
* ❌ Importovali sa nesprávne obrázky

**Ako zastaviť:**

1. Kliknite na**tlačidlo Zastaviť** (nahrádza tlačidlo Štart) — stačí raz
2. Na lište sa zobrazí „Zastavuje sa...“, kým sa dokončí spracovanie prebiehajúceho obrázku, potom sa spustenie ukončí v zastavenom stave
3. Už exportované výstupy zostanú na disku; do protokolu sa zapíše presný kód chyby `[RUN-SUMMARY]`, ktorý označuje, čo sa dokončilo
4. Odstráňte problémy a spustite proces znovu – spustenie začne od začiatku

***

## Riešenie problémov počas spracovania

### Spracovanie prebieha veľmi pomaly

**Možné príčiny:**

* Neoznačené cieľové obrázky (prehľadávanie všetkých obrázkov)
* Ukladanie na HDD namiesto SSD
* Nedostatočné systémové zdroje
* Nastavených je veľa indexov
* Prístup k sieťovému disku

**Riešenia:**

1. Ak sa spracovanie práve spustilo a je vo fáze detekcie: Zastavte ho, označte ciele a reštartujte
2. Do budúcnosti: Použite SSD, znížte počet indexov, modernizujte hardvér
3. Zvážte použitie CLI na hromadné spracovanie veľkých dátových súborov

### Upozornenia na „miesto na disku“

**Riešenia:**

1. Okamžite uvoľnite miesto na disku
2. Presuňte projekt na disk s väčším priestorom
3. Znížte počet indexov na export
4. Deaktivujte exportné produkty LATTICE, ktoré nepotrebujete (Nastavenia projektu → Spracovanie)
5. Použite formát JPG namiesto TIFF (menšie súbory)

### Časté hlásenia o „poškodených súboroch“

**Riešenia:**

1. Znovu skopírujte obrázky z SD karty, aby ste zaistili ich integritu
2. Skontrolujte SD kartu na prítomnosť chýb
3. Odstráňte poškodené súbory z projektu
4. Pokračujte v spracovaní zostávajúcich obrázkov

### Prehrievanie systému / obmedzovanie výkonu

**Riešenia:**

1. Zabezpečte dostatočné vetranie
2. Odstráňte prach z ventilačných otvorov počítača
3. Znížte zaťaženie pri spracovaní (použite režim Free namiesto Chloros+)
4. Spracúvajte v chladnejších časoch dňa

***

## Oznámenie o dokončení spracovania

Po dokončení spracovania:

* Ukazovateľ priebehu dosiahne 100 %
* V ladiacom protokole sa zobrazia riadky `[RUN-SUMMARY]` s konečnými počtami
* Tlačidlo Štart sa opäť aktivuje
* Všetky výstupné súbory sa nachádzajú vo výstupnej štruktúre projektu pre jednotlivé kamery: `<project>/<camera>/<format>/<Product>_Images/`

***

## Ďalšie kroky

Po dokončení spracovania:

1. **Skontrolujte výsledky** – pozrite si [Dokončenie spracovania](finishing-the-processing.md)
2. **Skontrolujte výstupný priečinok** – overte, či boli všetky súbory správne exportované
3. **Skontrolujte protokol ladenia** – skontrolujte, či sa v ňom nenachádzajú žiadne varovania alebo chyby
4. **Prehliadnite si spracované snímky** – Použite prehliadač obrázkov alebo externý softvér

Informácie o kontrole a používaní spracovaných výsledkov nájdete v časti [Dokončenie spracovania](finishing-the-processing.md).

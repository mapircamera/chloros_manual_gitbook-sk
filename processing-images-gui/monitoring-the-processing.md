# Sledovanie spracovania

Po spustení spracovania ponúka Chloros niekoľko spôsobov, ako sledovať priebeh, kontrolovať prípadné problémy a zistiť, čo sa deje s vašou dátovou sadou. Na tejto stránke sa dozviete, ako sledovať priebeh spracovania a interpretovať informácie, ktoré Chloros poskytuje.

## Prehľad ukazovateľa priebehu

Indikátor priebehu v hornej časti obrazovky zobrazuje stav spracovania v reálnom čase a percentuálnu úroveň dokončenia. Priebeh sa prenáša naživo z backendu prostredníctvom Server-Sent Events (SSE), takže indikátor odzrkadľuje to, čo spracovateľský reťazec skutočne robí.

### Indikátor priebehu v bezplatnom režime

Pre používateľov bez licencie Chloros+:

**Dvojfázové zobrazenie priebehu:**

1.**Detekcia cieľov** – Vyhľadávanie kalibračných cieľov v obrázkoch
2. **Spracovanie** – Uplatňovanie korekcií a export**Indikátor priebehu zobrazuje:**

* Celkové percento dokončenia (0–100 %)
* Názov aktuálnej fázy
* Jednoduchá vizualizácia vo forme horizontálneho pruhu

### Indikátor priebehu v Chloros+

Pre používateľov s licenciou Chloros+:

**Zobrazenie priebehu v 4 fázach:**

1.**Detekcia** – vyhľadávanie kalibračných cieľov
2. **Analýza** – preskúmanie obrázkov a príprava spracovateľského reťazca
3. **Kalibrácia** – Aplikácia korekcií vinety a odrazivosti
4. **Export** – Uloženie spracovaných súborov**Interaktívne funkcie:*** **Naveďte kurzor** na ukazovateľ priebehu, aby sa zobrazil rozbalený 4-fázový panel
* **Kliknutím** na ukazovateľ priebehu zamrazíte/pripnete rozbalený panel
* **Kliknite znovu** na odblokovanie a automatické skrytie pri odsunutí kurzora
* Každá fáza zobrazuje individuálny priebeh (0–100 %)

{% hint style="info" %}
**Parita CLI**: počas behu `chloros-cli process` sa v tých istých štyroch vláknach zobrazuje stav „Detecting“, „Analyzing“, „Processing“ a „Exporting“, pričom `chloros-cli export-status` zobrazuje priebeh exportu vo vlákne 4 v reálnom čase z iného terminálu. Pozrite si [Referenčnú príručku k CLI](../reference/cli-reference.md).
{% endhint %}

***

## Vysvetlenie jednotlivých fáz spracovania

{% hint style="info" %}
**Architektúra potrubia**: Tieto 4 fázy v grafickom rozhraní zodpovedajú [4-vláknovému spracovateľskému potrubiu](../processing-architecture/processing-pipeline.md). V systémoch s akceleráciou GPU vlákno 3 (Kalibrácia) využíva [dynamickú adaptáciu výpočtov](../processing-architecture/dynamic-compute-adaptation.md), ktorá optimalizuje spracovanie pre váš konkrétny hardvér.
{% endhint %}

### Fáza 1: Detekcia (detekcia cieľa)

**Čo sa deje:**

* Chloros skenuje obrázky, ktoré ste označili zaškrtnutím políčka „Cieľ“ (všetky obrázky, ak nie je zaškrtnuté žiadne políčko)
* Algoritmy počítačového videnia identifikujú kalibračné panely
* Z každého panelu sa extrahujú hodnoty odrazivosti
* Zaznamenávajú sa časové pečiatky cieľov pre správne naplánovanie kalibrácie

**Trvanie:**

* S označenými cieľmi: 10–60 sekúnd
* Bez označených cieľov: 5–30+ minút (skenuje všetky obrázky)

**Indikátor priebehu:**

* Detekcia: 0 % → 100 %
* Počet naskenovaných snímok (počíta sa len snímky, ktoré sa skutočne skenujú)
* Počet nájdených cieľov

**Na čo si dávať pozor:**

* Ak sú ciele správne označené, proces by mal prebehnúť rýchlo
* Ak to trvá príliš dlho, ciele môžu byť neoznačené
* Skontrolujte protokol ladenia (Debug Log) na správy typu „Target found“

### Fáza 2: Analýza

**Čo sa deje:**

* Čítanie metadát EXIF obrázkov (časové značky, nastavenia expozície)
* Určenie stratégie kalibrácie na základe časových značiek cieľov a dostupných údajov DAQ o dopade slnečného žiarenia
* Usporiadanie fronty na spracovanie obrázkov
* Príprava procesov na paralelné spracovanie (len Chloros+)

**Trvanie:** 5–30 sekúnd**Indikátor priebehu:**

* Analýza: 0 % → 100 %
* Rýchla fáza, zvyčajne sa dokončí rýchlo

**Na čo si dávať pozor:**

* Postup by mal prebiehať plynulo bez prestávok
* V lógovom súbore Debug Log sa zobrazia varovania o chýbajúcich metadátach

### Fáza 3: Kalibrácia

**Čo sa deje:*** **Debayering**: Konverzia vzoru RAW Bayer na 3 kanály (pri monokromatických moduloch LATTICE sa táto fáza preskočí, s poznámkou)
* **Korekcia vinetácie**: Odstránenie stmavnutia okrajov objektívu
* **Kalibrácia odrazivosti**: Normalizácia pomocou cieľových hodnôt a/alebo zníženia citlivosti DAQ
* **Výpočet indexov**: Výpočet multispektrálnych indexov
* Spracovanie každého snímku cez celý spracovateľský reťazec

**Trvanie:** Väčšina celkového času spracovania (60–80 %)**Indikátor priebehu:**

* Kalibrácia: 0 % → 100 %
* Aktuálny obrázok sa spracúva
* Dokončené obrázky / Celkový počet obrázkov

**Spôsob spracovania:*** **Režim Free**: Spracováva jeden snímok po druhom postupne
* **Režim Chloros+**: Spúšťa hardvérovo adaptívny fond pracovných procesov — 1–4 súbežné pracovné procesy na systémoch s GPU (podľa VRAM), jeden pracovný proces na fyzické jadro (mínus jeden) na systémoch iba s CPU. Pozri [Dynamická adaptácia výpočtov](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU akcelerácia**: Výrazne urýchľuje túto fázu**Na čo si dávať pozor:**

* Plynulý postup podľa počtu obrázkov
* Skontrolujte ladiaci protokol, či obsahuje správy o dokončení jednotlivých obrázkov
* Varovania týkajúce sa kvality snímok alebo problémov s kalibráciou

### Fáza 4: Export

**Čo sa deje:**

* Ukladanie spracovaných snímok na disk vo vybranom formáte hneď po ich dokončení
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

## Karta Debug Log (Log ladenia)

Log ladenia poskytuje podrobné informácie o priebehu spracovania a všetkých zistených problémoch. Do konzoly logu sa tiež zaznamenávajú správy o spustení backendu, takže log poskytuje kompletný prehľad aj v prípade, že ho otvoríte neskôr.

### Prístup k protokolu ladenia

1. Kliknite na ikonu **Protokol ladenia** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

v ľavom bočnom paneli
2. Otvorí sa panel protokolu, ktorý zobrazuje správy o spracovaní v reálnom čase
3. Automatické posúvanie zobrazuje najnovšie správy

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written), with the 1.3.0 bug icon (Report a bug) visible at the top of the tab -->

### Vysvetlenie správ v protokole Riadky protokolu

Chloros u začínajú značkami v zátvorkách, ktoré označujú názov subsystému — napríklad `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Najdôležitejšie je poznať **súhrn behu**, ktorý sa zobrazuje na konci každého behu (vrátane zastavených behov):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Dodatočné riadky s tipmi `[RUN-SUMMARY]` nasledujú vždy, keď je potrebné niečo vysvetliť — napríklad beh, ktorý nevytvoril žiadny výstup, alebo kamera, ktorej požadovaný produkt bol vynechaný ako nepoužiteľný. Riadky `[EXPORT-CHECK]` vysvetľujú vynechania pre jednotlivé kamery (napr. prečo kamera RGB nezískala produkt žiarivosti).

Všeobecné úrovne závažnosti správ (príklady nižšie sú ilustratívne, nie doslovné):

#### Informačné správy (biele/šedé)

Bežné aktualizácie spracovania: spustenie spracovania, detekcia cieľov (s počtom panelov), priebeh kalibrácie jednotlivých snímok, export súborov, dokončenie spracovania.

#### Varovné správy (žlté)

Nekritické problémy, ktoré nezastavia spracovanie — napr. chýbajúce údaje GPS v snímke, veľký časový rozdiel medzi snímkami cieľov alebo nízky kontrast v kalibračnom paneli.

**Akcia:** Po spracovaní skontrolujte varovania, ale spracovanie neprerušujte

#### Chybové správy (v Red)

Kritické problémy, ktoré môžu spôsobiť zlyhanie spracovania — napr. zaplnený disk, poškodený obrazový súbor alebo nezistenie žiadnych cieľov pri požiadavke na kalibráciu odrazivosti.

**Postup:** Zastavte spracovanie, odstráňte chybu a spustite spracovanie znova

### Bežné situácie v protokole

| Situácia                             | Význam                                       | Potrebné opatrenie                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Cieľ zistený v súbore \[názov súboru]        | Cieľ kalibrácie bol úspešne nájdený         | Žiadne – normálny stav                                         |
| Čiarky postupu pre jednotlivé obrázky              | Aktualizácia aktuálneho postupu                       | Žiadne – normálny stav                                         |
| Nenašli sa žiadne ciele                      | Neboli zistené žiadne kalibračné ciele               | Označte obrázky cieľov alebo deaktivujte kalibráciu odrazivosti |
| Nedostatočný priestor na disku               | Nedostatok úložného priestoru pre výstup                 | Uvoľnite miesto na disku                                    |
| Preskakovanie poškodeného súboru               | Obrazový súbor je poškodený                         | Znovu skopírujte súbor z SD karty                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Snímku bez surového snímku nie je možné spracovať | Znovu nasnímajte so surovým snímkom, alebo použite CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Spustenie nevytvorilo žiadne obrazové výstupy — hlásené ako zlyhanie s tipmi | Prečítajte si riadky s tipmi; skontrolujte, čo bolo preskočené a prečo |
| `[PROCESS] refused: The project folder is too deep for this computer…` | Dlhé cesty v systéme Windows sú vypnuté a priečinok projektu je príliš hlboko v štruktúre na to, aby spĺňal limit 260 znakov pre cestu; nič nebolo spracované | Použite kratší priečinok projektu (správa uvádza najdlhšiu povolenú dĺžku), alebo zapnite dlhé cesty v systéme Windows a reštartujte program Chloros — pozrite si [Spustenie spracovania](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer) |
| `[unmix-clamp-mask] …` | Nepodarilo sa zapísať masku clamp produktu LATTICE; samotný produkt je v poriadku a hlási `Chloros:UnmixClampMask=unavailable` | Skontrolujte voľné miesto na disku a oprávnenia priečinka, potom spracovanie zopakujte, ak masku potrebujete |

### Kopírovanie údajov z protokolu

Kopírovanie protokolu na účely riešenia problémov alebo technickej podpory:

1. Otvorte panel Debug Log (Protokol ladenia)
2. Kliknite na tlačidlo **Copy**(Kopírovať) (skopíruje protokol do schránky) alebo na**Download** (Stiahnuť), aby ste ho uložili ako textový súbor
3. Vložte do textového súboru alebo do e-mailu
4. V prípade potreby pošlite na podporu MAPIR



### Nahlásenie chyby z karty protokolu

Novinka vo verzii 1.3.0: ikona chyby (**Nahlásiť chybu**) v hornej časti karty Debug Log odošle hlásenie priamo do MAPIR.

1. Kliknite na ikonu chyby v hornej časti karty Debug Log
2. Popíšte problém: čo ste robili, čo ste očakávali a čo sa namiesto toho stalo (povinné)
3. Voliteľne zaškrtnite **Pridať niekoľko obrázkov (až 10)**a vyberte obrázky z otvoreného projektu alebo použite**Prehľadávať...** pre akýkoľvek iný obrazový súbor
4. Kliknite na **Odoslať**

Správa obsahuje váš popis, všetky priložené obrázky, protokoly Chloros a podrobnosti o systéme (verzia Chloros, operačný systém, grafická karta a nastavenia projektu). Na odoslanie správy musíte byť prihlásení do Chloros (karta „User“). Počas odosielania môžete okno zavrieť; po dokončení sa do protokolu zapíše referenčné číslo, napríklad `Bug report sent. Reference: BR-…` – uveďte ho, keď budete kontaktovať MAPIR. Ak sa správa nedá odoslať (napríklad kvôli chýbajúcemu pripojeniu k internetu), Chloros ju uloží do vášho počítača a zobrazí cestu k súboru, aby ste ju mohli poslať e-mailom na adresu support@mapir.camera.

***

## Monitorovanie systémových zdrojov

### Využitie procesora

**Voľný režim:**

* 1 jadro procesora na ~100 %
* Ostatné jadrá sú v nečinnosti alebo sú k dispozícii
* Systém naďalej reaguje

**Paralelný režim Chloros+:**

* Viac jadier s vysokým využitím – ich počet závisí od stratégie zvolenej prostredníctvom [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* Systém môže pôsobiť menej odozvovo

**Ako monitorovať:**

* Správca úloh systému Windows (Ctrl+Shift+Esc)
* Karta Výkon → sekcia CPU
* Hľadajte procesy „Chloros “ alebo „chloros-backend“

### Využitie pamäte (RAM)

**Typické využitie:**

* Malé projekty (< 100 obrázkov): 2–4 GB
* Stredné projekty (100–500 obrázkov): 4–8 GB
* Veľké projekty (500 a viac obrázkov): 8–16 GB
* Paralelný režim Chloros+ využíva viac pamäte RAM

**Ak je málo pamäte:**

* Spracúvajte menšie dávky
* Zatvorte ostatné aplikácie
* Ak pravidelne spracúvate veľké súbory údajov, rozšírte pamäť RAM

### Využitie GPU (Chloros+ s CUDA)

Keď je zapnutá akcelerácia GPU:

* GPU NVIDIA vykazuje vysoké využitie (60–90 %)
* Využitie VRAM sa zvyšuje (vyžaduje 4 GB a viac VRAM; 7 GB a viac pre súbežné odstraňovanie bayeringu s ohľadom na textúry)
* Fáza kalibrácie je výrazne rýchlejšia

**Na sledovanie:**

* Ikona NVIDIA v systémovej lište
* Správca úloh → Výkon → GPU
* GPU-Z alebo podobný monitorovací nástroj

### Diskové I/O

**Čo môžete očakávať:**

* Vysoká rýchlosť čítania z disku počas fázy analýzy
* Vysoká rýchlosť zápisu na disk počas fázy exportu
* SSD je výrazne rýchlejší ako HDD

**Tip na zvýšenie výkonu:**

* Ak je to možné, použite SSD pre projektovú zložku
* Pri veľkých dátových súboroch sa vyhnite sieťovým diskom
* Uistite sa, že disk nie je takmer plný (ovplyvňuje to rýchlosť zápisu)

***

## Zisťovanie problémov počas spracovania

### Varovné signály

**Zastavenie priebehu (žiadna zmena po dobu 5 a viac minút):**

* Skontrolujte protokol ladenia, či neobsahuje chyby
* Overte, či je k dispozícii dostatok miesta na disku
* V Správcovi úloh skontrolujte, či beží program Chloros **Často sa zobrazujú chybové hlásenia:**

* Zastavte spracovanie a skontrolujte chyby
* Bežné príčiny: nedostatok miesta na disku, poškodené súbory, problémy s pamäťou
* Pozrite si časť Riešenie problémov nižšie

**Systém nereaguje:**

* Paralelný režim programu Chloros+ spotrebúva príliš veľa zdrojov
* Zvážte zníženie počtu súbežných úloh alebo modernizáciu hardvéru
* Režim „Free“ je menej náročný na zdroje

### Kedy zastaviť spracovanie

Zastavte spracovanie, ak sa objaví:

* ❌ Chyby typu „Disk je plný“ alebo „Nie je možné zapísať súbor“
* ❌ Opakované chyby poškodenia obrazových súborov
* ❌ Systém úplne zamrzol (nereaguje)
* ❌ Zistili ste, že boli nakonfigurované nesprávne nastavenia
* ❌ Importovali ste nesprávne obrazy

**Ako zastaviť:**

1. Kliknite na**tlačidlo Zastaviť** (nahrádza tlačidlo Štart) — stačí raz
2. Na lište sa zobrazí „Zastavuje sa...“, kým sa dokončí spracovávaný obraz, potom sa spustenie ukončí v zastavenom stave
3. Už exportované produkty zostanú na disku; protokol zobrazí presný záznam `[RUN-SUMMARY]` o tom, čo sa dokončilo
4. Odstráňte problémy a spustite proces znovu — spustenie začne od začiatku

***

## Riešenie problémov počas spracovania

### Spracovanie prebieha veľmi pomaly

**Možné príčiny:**

* Neoznačené cieľové obrázky (skenovanie všetkých obrázkov)
* Ukladanie na HDD namiesto SSD
* Nedostatočné systémové zdroje
* Nastavených je veľa indexov
* Prístup k sieťovému disku

**Riešenia:**

1. Ak ste práve začali a ste vo fáze detekcie: Zastavte, označte ciele, reštartujte
2. Do budúcnosti: Používajte SSD, znížte počet indexov, modernizujte hardvér
3. Zvážte použitie CLI pre hromadné spracovanie veľkých dátových súborov

### Upozornenia na „miesto na disku“

**Riešenia:**

1. Okamžite uvoľnite miesto na disku
2. Presuňte projekt na disk s väčším priestorom
3. Znížte počet indexov na export
4. Deaktivujte exportné produkty LATTICE, ktoré nepotrebujete (Nastavenia projektu → Spracovanie)
5. Použite formát JPG namiesto TIFF u (menšie súbory)

### Časté hlásenia o „poškodených súboroch“

**Riešenia:**

1. Znovu skopírujte obrázky z SD karty, aby ste zaistili ich integritu
2. Skontrolujte SD kartu na chyby
3. Odstráňte poškodené súbory z projektu
4. Pokračujte v spracovaní zostávajúcich obrázkov

### Prehrievanie systému / obmedzovanie výkonu

**Riešenia:**

1. Zabezpečte dostatočné vetranie
2. Odstráňte prach z ventilačných otvorov počítača
3. Znížte zaťaženie pri spracovaní (použite režim „Free“ namiesto „Chloros+“)
4. Spracovávajte v chladnejších častiach dňa

***

## Oznámenie o dokončení spracovania

Po dokončení spracovania:

* Ukazovateľ priebehu dosiahne 100 %
* V lógovom súbore Debug Log sa zobrazia riadky `[RUN-SUMMARY]` s konečnými počtami
* Tlačidlo Štart sa opäť aktivuje
* Všetky výstupné súbory sa nachádzajú vo výstupnej štruktúre projektu pre jednotlivé kamery: `<project>/<camera>/<format>/<Product>_Images/`

***

## Ďalšie kroky

Po dokončení spracovania:

1. **Skontrolujte výsledky** – pozrite si [Dokončenie spracovania](finishing-the-processing.md)
2. **Skontrolujte výstupný priečinok** – overte, či boli všetky súbory správne exportované
3. **Skontrolujte protokol ladenia** – Skontrolujte, či sa v ňom nenachádzajú žiadne varovania alebo chyby
4. **Prezrite si spracované obrázky** – Použite prehliadač obrázkov alebo externý softvér

Informácie o kontrole a používaní spracovaných výsledkov nájdete v časti [Dokončenie spracovania](finishing-the-processing.md).

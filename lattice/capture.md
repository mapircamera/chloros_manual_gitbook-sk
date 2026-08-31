# Nastavenia a režimy snímania

Snímanie na karte „Kamery“ sa ovláda pomocou jedného červeného tlačidla **Capture All**a jedného panela**Capture Settings**, ktorý určuje, aký výsledok toto tlačidlo prinesie: ktoré kamery sa zapoja, aké typy súborov ukladá každá kamera a či sa spúšť aktivuje jednorazovo, nepretržite alebo v určitých intervaloch. Táto stránka popisuje celý postup – konfiguráciu, samotné snímanie, umiestnenie súborov na disku a spôsob ich neskoršieho spracovania na kalibrované výstupy. Ovládacie prvky pre kamery a sústavy sa nachádzajú v časti [Nastavenia kamery](camera-settings.md).

{% hint style="info" %}
**Na snímanie je potrebný otvorený projekt.** Tlačidlá „Capture All“ a ozubené koliesko „Capture Settings“ sú deaktivované, kým nie je otvorený projekt („Vytvorte alebo otvorte projekt na uloženie snímok“). Každá snímka sa ukladá do projektovej zložky v `captures/`.
{% endhint %}

## Panel **Nastavenia zachytávania**Otvorte ho pomocou**ozubeného kolieska vedľa položky Zachytiť všetko**v zozname kamier na bočnom paneli alebo pomocou tlačidla**„Otvoriť nastavenia zachytávania…“** v dolnej časti akéhokoľvek panela nastavení pre jednotlivé kamery. V záhlaví je nápis „Nastavenia snímania“ s tlačidlom ← späť.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Vaše voľby v tomto paneli – zahrnuté kamery, začiarkavacie políčka pre jednotlivé typy a režim snímania – sa ukladajú **pre každý projekt** a obnovia sa pri jeho opätovnom otvorení.

### Režimy snímania

Tri tlačidlá režimov v hornej časti panela:

| Režim | Čo robí | Podnastavenia (predvolené hodnoty) |
| --- | --- | --- |
| **Jednorazové** *(predvolené)* | Jedno snímanie zo všetkých vybraných kamier. | — |
| **Neprerušované**| Snímky za sebou až do splnenia podmienky zastavenia. | Zastavenie podľa**počtu snímok** (predvolené 1) *alebo* **trvania snímania** (predvolené 10 s; jednotky: sekundy / minúty / hodiny / dni). |
| **Interval**(časozber) | Séria snímok podľa časovača. |**Počet snímok / interval**(predvolené nastavenie 1) ·**Každých**N jednotiek (predvolené nastavenie 5 s) ·**Po dobu** N jednotiek (predvolené nastavenie 1 m). |

V režime Nepretržitého snímania alebo Intervalového snímania sa tlačidlo „Zachytiť všetko“ počas prevádzky zmení na tlačidlo **Zastaviť (N)**, pričom sa pritom počíta počet snímok.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Výber kamier a typov exportu

Pomocný text v paneli to zhrňuje: vyberte, ktoré kamery a typy exportu má funkcia „Zachytiť všetko“ generovať — všetko je predvolene zapnuté a voľby sa uložia spolu s týmto projektom.

* Tlačidlá **Vybrať všetko / Nevybrať nič** naraz prepnú začiarkavacie políčka „Zahrnúť“ pri každej kamere.
* **Prepínače typov hromadného exportu**(dva riadky tlačidiel):**Všetko v surovom formáte / Všetko bez debayeringu / Všetko v náhľade / Všetko v Radiance / Všetko v odrazivosti / Všetko v indexe**. Každé z nich má trojstupňové farebné označenie: zelená ✓ = zapnuté pre každú kameru, ktorá to podporuje, oranžová – = zapnuté pre niektoré, sivá = žiadna. Prepínač je deaktivovaný, ak žiadna pripojená kamera tento typ nepodporuje. Všetky sú sivé, keď je zapnutá funkcia „Fastest Capture“.
* **Riadky pre jednotlivé kamery**: zaškrtávacie políčko „Zahrnúť“ a rozbaľovací zoznam (▸/▾) príslušných typov exportu pre danú kameru s jednotlivými zaškrtávacími políčkami. Riadok zobrazuje počet zapnutých položiek, napríklad „4/6“.

### Typy exportu a kamery, ktoré ich podporujú

Existuje šesť typov exportu: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. V riadku každej kamery sa zobrazujú len tie, ktoré sú pre ňu relevantné:

| Typ exportu | Obsah | RGB (FRGB) | Bayer multispektrálny (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayerova mozaika (mono: jedno spektrálne pásmo) priamo zo senzora | ✓ | ✓ | ✓ |
| **Debayered** | Lineárna demosaika (mono: 1-kanálová škála šedej) | ✓ | ✓ | ✓ |
| **Náhľad** | Kompletný reťazec spracovania (vyváženie bielej + gama podľa profilu fotoaparátu; multispektrálne: rozloženie do falošných farieb) | ✓ | ✓ | ✓ |
| **Žiarivosť** | float32 W/m²/sr/nm cez kompletný radiometrický reťazec | — (neponúka sa) | ✓ | ✓ |
| **Odrazivosť** | uint16 ρ (32768 = 1,0) | — (nenútené) | ✓ — zobrazené len vtedy, ak má kamera svetelný senzor DAQ (vlastný alebo zdedený z jej poľa) | rovnaké ako multispektrálne |
| **Index** | Vykreslenie vegetačného indexu (LUT) | — | ✓ — vyžaduje povolený, neprázdny indexový výraz na kamere a nie je k dispozícii pre členov kombinovaného poľa (pole vlastní jeden zdieľaný index) | — (index vyžaduje ≥2 pásma; pozri [Mono kamery a vegetačné indexy](mono-indices.md)) |

Žiarivosť a odrazivosť sa nikdy neponúkajú pre kamery RGB — žiarivosť na jeden Bayer nie je pre širokopásmový fotometrický senzor zmysluplná.

### Najrýchlejšie zachytenie

Prepínač **⚡ Najrýchlejšie zachytenie — iba surové dáta**(oranžový, keď je zapnutý) prepisuje všetky voľby exportu na**iba surové dáta** — plus bezplatný kompozit kombinovaného indexu pre súbory — aby sa snímka uložila čo najrýchlejšie: výpočty žiarivosti, odrazivosti a zobrazenia sa v čase snímania úplne vynechajú.

{% hint style="info" %}
**Súbor `.daq` sa aj tak uloží.** Ak je priradený svetelný senzor, funkcia Najrýchlejšie zachytenie aj naďalej zapisuje namerané hodnoty DAQ smerom nadol vedľa surových snímok — takže produkty žiarivosti, odrazivosti a indexu je možné neskôr vytvoriť opätovným spracovaním (pozri [Opätovné spracovanie zachytených údajov](#re-processing-captures-into-calibrated-products)). Funkcia Fastest Capture tiež neovplyvňuje výber zaškrtávacích políčok: ak ju vypnete, výber sa obnoví.
{% endhint %}

### Ovládacie prvky pre jednotlivé polia

Každé pripojené pole má v paneli vlastnú skupinovú kartu:

* **Začiarkavacie políčko „Include“** (s tromi stavmi pre jednotlivé členy) a názov poľa s režimom zobrazenia: „(combined | separate)“.
* Zaškrtávacie políčko **Aligned**(predvolene**zapnuté**): deformuje exporty členov podľa profilu zarovnania poľa, takže exporty sú pixelovo zarovnané medzi kamerami. Surové dáta zostávajú bez deformácie, ale transformáciu nesú vo svojich metadátach. (Samotný profil sa vypočíta v [paneli nastavení poľa](camera-settings.md#alignment-co-registration-combined-only).)
* Riadky kamier členov sú vnořené vnútri karty.

Karta poľa obsahuje aj dva záznamníky. Predstavte si ich ako **monitorovanie vs. analýza**:

| Záznamník | Úroveň | Čo zaznamenáva |
| --- | --- | --- |
| **● Nahrávať indexové video / ■ Zastaviť nahrávanie** *(len kombinované pole)* | **Monitorovanie** | Kompozitný obraz živého kombinovaného indexu do videa pri 10 fps — 8-bitové, rozlíšenie náhľadu, vstavaná LUT. Vyžaduje otvorený projekt a streamovaný živý náhľad. Zobrazuje snímky a uplynutý čas počas nahrávania. |
| **⦿ Nahrávanie surových sériových snímok / ■ Zastaviť surovú sériu** *(ľubovoľné pole)* | **Analýza**| Surové snímky v formáte Bayer pri rýchlosti živého snímania (bez spracovania) plus manifest na každý snímok a hodnoty `.daq`, uložené do formátu `captures/bursts/`. Po sérii sa zobrazí tlačidlo**Vytvoriť video**: offline spracuje sériu na kalibrované video — kombinovaný index a/alebo žiarivosť / odrazivosť / index pre každú kameru — plus voliteľné súbory TIFF. Vytvorenie kombinovaného indexu sa spustí automaticky po zastavení série.

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

|## Postup

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

„Zachytiť všetko“ Stlačte **Zachytiť všetko** v zozname kamier na bočnom paneli:

1. Každá zahrnutá, viditeľná a nezastavená kamera sníma s vybranými typmi exportu. **Skupiny sa spúšťajú ako jeden synchronizovaný spúšťací signál** (jedna synchronizovaná skupina pre všetkých členov — pozri [Skupiny s viacerými kamerami](arrays.md)); samostatné kamery zaznamenávajú individuálne.
2. Skryté (oko) alebo pozastavené kamery sa preskočia. Pole je úplne zablokované len vtedy, keď sú *všetky* jeho členy skryté alebo pozastavené.
3. Vždy, keď je priradený svetelný senzor, zodpovedajúca hodnota DAQ merania dopadajúceho žiarenia sa uloží ako súbor `.daq` spolu so snímkami — aj v prípade záznamov iba v surovom formáte — takže radiometrické produkty je možné vždy odvodiť neskôr.
4. Tlačidlo zobrazuje priebeh v reálnom čase — „Zaznamenávanie… hotovo/celkom“ — a v režime Nepretržité/Interval sa zmení na **Stop (N)**. Každá položka snímania má časový limit 300 s.
5. Po dokončení preletu sa v výsledkovom hlásení zobrazí **„Uložených N súborov“**alebo**„Uložených N, F zlyhalo“**, plus „(S skryté/pozastavené/preskočené)“, ak boli kamery preskočené.

## Kam sa ukladajú záznamy

Záznamy sa ukladajú v rámci otvoreného projektu do adresára `<project>/captures/`. Každý typ exportu sa ukladá do **vlastnej podzložky**, takže pri viacúrovňovom zázname sa typy nikdy nemiešajú:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` je časová pečiatka záznamu a `<serial>` sériové číslo kamery. Samostatné záznamy sú pomenované ako `capture_<ts>_SN<serial>_<level>`; záznamy z poľa z jedného synchronizovaného spúšťača sú pomenované ako `sync_<ts>_SN<serial>_<level>` a **zdieľajú jeden časový údaj pre všetky kamery v skupine** (prípona úrovne sa vynechá, ak kamera ukladá iba jednu úroveň).
* **Jedna výnimka, ktorú treba vedieť:** úroveň zobrazenia sa ukladá do priečinka s názvom `preview/`, zatiaľ čo súbory si v názve zachovávajú `_display` — priečinok a prípona sa líšia iba pre túto úroveň.
* Neznáme úrovne sa ukladajú do zložky s ich vlastným názvom; ak nie je možné vytvoriť podzložku, súbor sa zapíše do koreňového adresára záznamov, namiesto toho, aby sa stratil.
* Snímky vo formáte TIFF sú štandardne bezstratovo komprimované (DEFLATE) a obsahujú úplné metadáta o kalibrácii a spracovaní **priamo v súbore XMP** — snímky sú samopopisné a okrem súboru `.daq` neobsahujú žiadne ďalšie doplnkové súbory.

Toto je rovnaká štruktúra, akú súbory `chloros-cli lattice capture` / `array-capture` zapisujú do svojho adresára `-o` — zdokumentované v [Referencii CLI § Ako vyzerá priečinok so záznamami](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Opätovné spracovanie záznamov na kalibrované produkty

Zachytené surové snímky spolu s uloženým súborom `.daq` sú všetko, čo spracovateľský reťazec potrebuje — preto je funkcia Fastest Capture bezpečná pre skutočnú prácu.

* **GUI**: pridajte zložku so záznamami do projektu ([Pridávanie súborov do projektu](../processing-images-gui/adding-files-to-a-project.md)) a spracujte ich ako zvyčajne.
* **CLI**: nasmerujte `process` na**koreňový adresár snímok**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` zvyčajne importuje len zložku, ktorú určíte, ale ak táto zložka neobsahuje žiadne obrázky a má podzložky, automaticky prechádza do nich — takže súbory v podzložkách na danej úrovni a v koreňovej zložke `.daq` sa načítajú naraz. Každý záznam sa importuje ako **jeden obrázok**, pričom ostatné úrovne sú pripojené ako režimy zobrazenia, nie ako jeden obrázok na úroveň.

Priame pomenovanie podzložky úrovne (napr. `…/captures/raw/`) tiež funguje, ale koreňové súbory `.daq` zostanú vynechané — skopírujte ich spolu s nimi, keď znovu odvádzate radiometrický produkt z `raw/`, inak sa časová pečiatka nebude mať s čím zosúladiť.

{% hint style="warning" %}
**Spracovanie sa vždy začína od súboru `raw`.**V rámci každého záznamu je zdrojom spracovateľského reťazca surový snímok; `debayered`, `radiance`, `reflectance` a `preview` slúžia ako režimy zobrazenia, ale nikdy sa nevracajú späť do spracovateľského reťazca — opätovné spracovanie odvodeného produktu by znamenalo opätovné uplatnenie výpočtov vinetácie, farieb a jasu, ktoré sú už zapracované do jeho pixelov, preto sa Chloros odmietne namiesto dvojitého spracovania. Rendery `index/` a `composite/` sa vôbec nespracúvajú (sú to výstupy, nie záznamy). Zložka „captures“ uložená**bez** importu surových dát sa zobrazuje normálne, ale `process` ju preskočí a oznámi to; `--input-level {raw,debayered,processed}` je zámerný únikový mechanizmus, ktorý vynúti vstupný bod. Presné správy o preskočení nájdete v [Referencii k CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).
{% endhint %}

Ďalšie dve správania, ktoré stojí za to poznať pri skriptovaní opätovného spracovania:

* Spustenie `chloros-cli process`, ktoré požiadalo o produkty, ale nezapísalo **žiadne obrazové produkty, zlyhá s jasnou chybou a ukončí sa s nenulovým kódom** — nikdy nedosiahnete tiché prázdne spustenie. Úspešné spustenia hlásia počet svojich produktov. (Úmyselné spustenie iba s metadátami sa stále považuje za úspech.)
* Opätovne importované spracované výstupy nikdy nezaberajú slot pre surové dáta zo zachytenia — pôvodné surové dáta vždy zostávajú zdrojom spracovateľského reťazca.

## Ekvivalenty CLI

Všetko na tejto stránke je možné spúšťať bez grafického rozhrania. Režimy snímania v grafickom rozhraní sa priamo zhodujú s `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Jednorazový | `chloros-cli lattice array-capture` |
| Nepretržitý | `array-capture --continuous [--count N] [--duration S]` |
| Intervalové | `array-capture --interval S [--duration S]` |
| Najrýchlejšie zachytávanie | `array-capture --fastest` |
| Zarovnané zaškrtávacie políčko | `--aligned / --no-aligned` |
| Začiarkavacie políčka typu exportu | `--processing LEVEL` alebo `--levels L1,L2,…` (predvolené `all`) |
| Nahrávanie indexového videa | `chloros-cli lattice array-record` |
| Nahrávanie surových sériových snímok / Vytvorenie videa | `chloros-cli lattice array-burst` / `array-build-video` |

Úplné tabuľky príznakov, možnosť inteligentného automatického nastavenia expozície (`--smart`) a model s konštantnou rýchlosťou sa nachádzajú v [CLI Referencia § Režimy snímania, Záznamníky a offline spracovanie](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

# Nastavenia a režimy snímania

Snímanie na karte „Kamery“ sa ovláda pomocou jedného červeného tlačidla **Capture All**a jedného panela**Capture Settings**, ktorý určuje, aký výsledok toto tlačidlo prinesie: ktoré kamery sa zapoja, aké typy exportu každá kamera ukladá a či sa spúšť aktivuje jednorazovo, nepretržite alebo v určitých intervaloch. Táto stránka popisuje celý postup – konfiguráciu, samotné snímanie, umiestnenie súborov na disku a spôsob ich neskoršieho spracovania na kalibrované výstupy. Ovládacie prvky pre kamery a sústavy sa nachádzajú v časti [Nastavenia kamery](camera-settings.md).

{% hint style="info" %}
**Zaznamenávanie vyžaduje otvorený projekt.** Možnosť „Zachytiť všetko“ a ozubené koliesko „Nastavenia snímania“ sú deaktivované, kým nie je otvorený projekt („Vytvorte alebo otvorte projekt na uloženie snímok“). Každá snímka sa ukladá do projektovej zložky v `captures/`.
{% endhint %}

## Panel Nastavenia snímania

Otvorte ho pomocou **ozubeného kolieska vedľa položky Zachytiť všetko**v zozname kamier na bočnom paneli alebo pomocou tlačidla**„Otvoriť nastavenia snímania…“** v dolnej časti akéhokoľvek panela nastavení pre jednotlivé kamery. V záhlaví je nápis „Nastavenia snímania“ s tlačidlom ← späť.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Vaše voľby v tomto okne – vybrané kamery, začiarkavacie políčka pre jednotlivé typy a režim snímania – sa ukladajú **pre každý projekt zvlášť** a obnovia sa pri jeho opätovnom otvorení.

### Režimy snímania

Tri tlačidlá režimov v hornej časti okna:

| Režim | Funkcia | Podnastavenia (predvolené) |
| --- | --- | --- |
| **Jednorazové** *(predvolené)* | Jedno snímanie zo všetkých vybraných kamier. | — |
| **Neprerušované**| Snímanie za sebou, kým nenastane podmienka zastavenia. | Zastavenie podľa**počtu snímok** (predvolené 1) *alebo* **trvania snímania** (predvolené 10 s; jednotky: sekundy / minúty / hodiny / dni). |
| **Interval**(časozber) | Séria snímok podľa časovača. |**Počet snímok / interval**(predvolené 1) ·**Každých**N jednotiek (predvolené 5 s) ·**Po dobu** N jednotiek (predvolené 1 m). |

V režime Nepretržité alebo Interval sa tlačidlo Zachytiť všetko počas behu zmení na tlačidlo **Zastaviť (N)**, pričom sa počítajú snímky, ako sa ukladajú. Zaplnený disk zastaví beh (pozri [Priebeh funkcie „Zachytiť všetko“](#the-capture-all-flow)).**Režim „Neprerušovaný“ na poliach beží v rámci backendu (novinka vo verzii 1.3.0).** Ak všetky kamery v zozname patria do poľa (samostatné kamery sú skryté alebo vylúčené), režim „Continuous“ už neposiela jednu požiadavku na každý záznam: backend Chloros spúšťa cyklus zachytávania sám, zapisuje jednu synchronizovanú skupinu, zatiaľ čo vyberá ďalšiu, a tlačidlo zisťuje jej počet. Ak sú v zozname samostatné kamery, Chloros zachytáva záznamy po jednotlivých priechodoch ako doteraz.

{% hint style="warning" %}
**Rýchle snímanie v systéme Windows: nechajte backend Chloros vylúčený z kontroly programom Microsoft Defender.** Skenovanie každého súboru, ktorý sa zapisuje pri vysokej rýchlosti snímania, v reálnom čase stojí približne 30 % rýchlosti snímania v testoch MAPIR (stovky súborov za sekundu pri malej oblasti záujmu). Inštalačný program pre Windows ponúka možnosť **„Vylúčiť backend Chloros u zo skenovania programom Microsoft Defender (oveľa rýchlejšie snímanie z kamery; odporúčané)“**, ktorá je predvolene zaškrtnutá, a pri odinštalovaní toto vylúčenie zruší. Ak ste túto možnosť odškrtli, spustite inštalačný program znovu s zaškrtnutou možnosťou alebo pridajte vylúčenie procesu Defender pre `chloros-backend.exe` (v priečinku `backend` v inštalačnom priečinku Chloros). Keď Defender počas nepretržitého snímania poľa využíva jedno alebo viac jadier procesora, backend Chloros to raz zaznamená do svojho protokolu spolu s týmito dvoma riešeniami.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Výber kamier a typov exportu

Text nápovedy v tomto paneli to zhrňuje: vyberte, ktoré kamery a typy exportu má funkcia „Zachytiť všetko“ generovať — všetko je štandardne zapnuté a výbery sa uložia spolu s týmto projektom.

* Tlačidlá **Vybrať všetko / Nevybrať nič** naraz prepnú začiarkavacie políčka všetkých kamier.
* **Hromadné prepínanie typov exportu**(dva riadky tlačidiel):**Všetko v surovom formáte / Všetko bez debayeringu / Všetko v náhľade / Všetko v radiancii / Všetko v odrazivosti / Všetko v indexe**. Každý z nich má trojstupňové farebné označenie: zelená ✓ = zapnuté pre každú kameru, ktorá to podporuje, oranžová – = zapnuté pre niektoré, sivá = žiadna. Prepínač je deaktivovaný, ak žiadna pripojená kamera tento typ nepodporuje. Všetky sú sivé, keď je zapnutá funkcia Najrýchlejšie snímanie.
* **Riadky pre jednotlivé kamery**: začiarkavacie políčko na zaradenie a rozbaľovací zoznam (▸/▾) príslušných typov exportu pre danú kameru s individuálnymi začiarkavacími políčkami. V riadku sa zobrazuje počet zapnutých položiek, napríklad „4/6“.

### Typy exportu a kamery, ktoré ich podporujú

Existuje šesť typov exportu: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. V riadku každej kamery sa zobrazujú len tie, ktoré sú pre ňu relevantné:

| Typ exportu | Obsah | RGB (FRGB) | Bayer multispektrálny (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayerova mozaika (mono: jedno spektrálne pásmo) priamo zo senzora | ✓ | ✓ | ✓ |
| **Debayered** | Lineárna demosaika: z surového dátového súboru sa odstráni tmavý posun (DSNU) z kalibračného balíka, potom nasleduje demosaika zohľadňujúca okraje — bez korekcie linearity, vyrovnania poľa alebo oddelenia kanálov. Uložené ako 3-kanálový RGB TIFF (mono: 1-kanálová škála sivej) | ✓ | ✓ | ✓ |
| **Náhľad** | Kompletný reťazec zobrazenia (vyváženie bielej + gama podľa profilu fotoaparátu, plus nastavenie „Highlights“ (svetlá) u RGB fotoaparátu; multispektrálne: rozloženie falošných farieb) | ✓ | ✓ | ✓ |
| **Žiarivosť** | float32 W/m²/sr/nm prostredníctvom úplného radiometrického reťazca | — (neponúka sa) | ✓ | ✓ |
| **Odrazivosť** | uint16 ρ (32768 = 1,0) | — (nie je k dispozícii) | ✓ — zobrazuje sa len vtedy, ak má kamera svetelný senzor DAQ (vlastný alebo zdedený z jej poľa) | rovnako ako multispektrálne |
| **Index** | Vykreslenie vegetačného indexu (LUT) | — | ✓ — vyžaduje povolený, neprázdny indexový výraz na kamere a nie je k dispozícii pre členov kombinovaného poľa (pole vlastní jeden zdieľaný index) | — (index vyžaduje ≥2 pásma; pozri [Mono kamery a vegetačné indexy](mono-indices.md)) |

Žiarivosť a odrazivosť sa nikdy neponúkajú pre kamery typu RGB — žiarivosť na báze Bayerovej matice nemá zmysel pre širokopásmový fotometrický senzor.

### Najrýchlejšie zachytenie

Prepínač **⚡ Najrýchlejšie zachytenie — iba surové dáta**(oranžový, keď je zapnutý) prepisuje všetky voľby exportu na**iba surové dáta** — plus bezplatný kompozit kombinovaného indexu pre polia — takže snímka sa uloží čo najrýchlejšie: výpočty žiarivosti/odrazivosti/zobrazenia sa v čase zachytenia úplne preskočia.

{% hint style="info" %}
**`.daq` sa stále ukladá.** Ak je priradený svetelný senzor, funkcia Najrýchlejšie zachytenie aj naďalej zapisuje hodnotu DAQ smerujúcu nadol (do priečinka `daq/` zachytených údajov) — takže produkty žiarivosti, odrazivosti a indexu je možné neskôr vytvoriť opätovným spracovaním (pozri [Opätovné spracovanie záznamov](#re-processing-captures-into-calibrated-products)). Funkcia Fastest Capture tiež neovplyvňuje vaše výbery v začiarkavacích poliach: ak ju vypnete, výbery sa obnovia.
{% endhint %}

### Ovládacie prvky pre jednotlivé sústavy

Každá pripojená sústava má v paneli vlastnú skupinovú kartu:

* **Začiarkavacie políčko „Zahrnúť“** (s tromi možnosťami pre jednotlivé členy) a názov sústavy s režimom zobrazenia: „(kombinované | samostatné)“.
* Zaškrtávacie políčko **Aligned**(predvolene**zapnuté**): deformuje exporty členov podľa profilu zarovnania poľa, takže exporty sú pixelovo zarovnané medzi kamerami. Surové dáta zostávajú bez deformácie, ale transformáciu nesú vo svojich metadátach. (Samotný profil sa vypočíta v [paneli nastavení poľa](camera-settings.md#alignment-co-registration).) Súbory členov poľa sa ukladajú v natívnej orientácii každej kamery, pričom akékoľvek nastavenia zrkadlenia/rotácie sa zaznamenávajú do súboru, namiesto toho, aby sa aplikovali.
* Riadky členských kamier sú vnořené do karty.

Karta poľa obsahuje aj dva záznamníky. Predstavte si ich ako **monitorovanie vs. analýza**:

| Záznamník | Úroveň | Čo zaznamenáva |
| --- | --- | --- |
| **● Nahráva indexové video / ■ Zastaví nahrávanie** *(len kombinované polia)* | **Monitorovanie** | Živý kompozitný indexový obraz do videa pri 10 fps — 8-bitový, rozlíšenie náhľadu, vstavaná LUT. Vyžaduje otvorený projekt a streamované živé zobrazenie. Počas nahrávania zobrazuje snímky a uplynutý čas. |
| **⦿ Nahráva surové snímky v sérii / ■ Zastaviť surovú sériu** *(ľubovoľná matica)* | **Analýza**| Surové snímky v formáte Bayer pri rýchlosti živého snímania (bez spracovania) plus manifest na každý snímok a hodnoty `.daq`, uložené do súboru `captures/bursts/`. Po sérii sa zobrazí tlačidlo**Vytvoriť video**: offline spracuje sériu na kalibrované video — kombinovaný index a/alebo žiarivosť / odrazivosť / index pre každú kameru — plus voliteľné súbory TIFF. Vytvorenie kombinovaného indexu sa spustí automaticky po zastavení série. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## Postup

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

„Zachytiť všetko“

Stlačte tlačidlo **Zachytiť všetko** v zozname kamier na bočnom paneli:

1. Každá zahrnutá, viditeľná a nezastavená kamera sníma s vybranými typmi exportu. **Súbory kamier sa spúšťajú ako jeden synchronizovaný spúšťač** (jedna synchronizovaná skupina naprieč všetkými členmi — pozri [Súbory viacerých kamier](arrays.md)); samostatné kamery sníma individuálne.
2. Skryté (oko) alebo pozastavené kamery sa preskočia. Pole je úplne zablokované len vtedy, ak sú *všetky* jeho členy skryté alebo pozastavené.
3. Vždy, keď je priradený svetelný senzor, zodpovedajúca hodnota DAQ merania dopadajúceho žiarenia sa uloží ako súbor `.daq` do priečinka `daq/` so záznamami — a to aj v prípade záznamov iba v surovom formáte —, aby bolo možné neskôr vždy odvodiť radiometrické produkty.
4. Tlačidlo zobrazuje priebeh v reálnom čase — „Zaznamenávanie… hotové/celkom“ — a v režime Continuous/Interval sa zmení na **Stop (N)**. Každá položka záznamu má časový limit 300 s.
5. Po dokončení preletu sa v výsledkovom hlásení zobrazí **„Uložených N súborov“**alebo**„Uložených N, F zlyhalo“**, plus „(S skryté/pozastavené/preskočené)“, ak boli kamery preskočené. Ak sa disk zaplní, beh sa zastaví a zobrazí hlásenie**„Zastavené: disk plný. Uložené N, F zlyhalo“** — každý súbor, ktorý sa nepodarilo zapísať, sa počíta ako zlyhanie.

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
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` je časová pečiatka záznamu a `<serial>` je sériové číslo kamery. Samostatné záznamy sú pomenované ako `capture_<ts>_SN<serial>_<level>`; záznamy z poľa z jedného synchronizovaného spúšťača sú pomenované ako `sync_<ts>_SN<serial>_<level>` a **majú spoločný časový údaj pre všetky kamery v skupine** (prípona úrovne sa vynecháva, ak kamera ukladá iba jednu úroveň).
* **Jedna výnimka, ktorú treba vedieť:** úroveň zobrazenia sa ukladá do priečinka s názvom `preview/`, zatiaľ čo súbory si v názve zachovávajú `_display` — priečinok a prípona sa líšia iba pre túto úroveň.
* Neznáme úrovne sa ukladajú do zložky s ich vlastným názvom; ak nie je možné vytvoriť podzložku, súbor sa zapíše do koreňového adresára záznamov, namiesto toho, aby sa stratil.
* TIFF súbory so záznamami sú štandardne bezstratovo komprimované (DEFLATE) a obsahujú svoje úplné metadáta o kalibrácii a spracovaní **priamo v XMP súbore** — snímky sú samopopisné a nemajú žiadne doplnkové súbory okrem súboru `.daq` (a jeho `.daq.cameras.json`).
* **Masky obmedzenia (novinka vo verzii 1.3.0).**Ak muselo spektrálne oddelenie 3×3 obmedziť pásmo na hodnotu 0 aspoň na jednom pixeli, produkty žiarivosti, odrazivosti a indexu dostanú masku na úrovni jednotlivých pixelov, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bit 0 = pásmo v kanáli R produktu, bit 1 = G, bit 2 = B); na ňu odkazuje XMP tag produktu `Chloros:UnmixClampMask`. Masky sa nachádzajú**vedľa** zložiek produktov, nikdy vo vnútri nich, takže zložku produktu, ako je napríklad `reflectance/`, je možné odovzdať fotogrametrickému softvéru tak, ako je. Vlastný import programu Chloros masky preskakuje.
* **Orientácia (1.3.0).** Produkty samostatnej kamery typu Preview, Radiance, Reflectance a Index sa otáčajú podľa nastavenia Mirror/Rotation; súbory Raw a Debayered zostávajú tak, ako ich snímač načítal. Všetky súbory v poli zostávajú v pôvodnom formáte. Každý súbor zaznamenáva toto nastavenie vo svojom XMP, takže pri opätovnom spracovaní sa uplatňuje rovnakým spôsobom.
* **K akej kamere patrí súbor `.daq` (1.3.0).** `.daq` záznamu sa pri spracovaní priraďuje iba k snímkam z fotoaparátov, ktorých názvy sú uvedené v jeho `.daq.cameras.json`. Samostatná kamera bez vlastného svetelného senzora preto už nevyužíva údaje z inej kamery – priraďte k nej senzor alebo ho zaznamenajte na karte Svetelné senzory. Záznam zo senzora osvetlenia (a akýkoľvek kód `.daq` bez tohto súboru, vrátane starších záznamov) stále slúži pre každú kameru.

Ide o rovnakú štruktúru, akú súbory `chloros-cli lattice capture` / `array-capture` zapisujú do svojho adresára `-o` — zdokumentované v [Referencii CLI § Ako vyzerá zložka so záznamami](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Opätovné spracovanie záznamov na kalibrované produkty

Zachytené surové snímky spolu s uloženým súborom `.daq` sú všetko, čo spracovateľský reťazec potrebuje — preto je funkcia Fastest Capture bezpečná pre skutočnú prácu.

* **GUI**: pridajte zložku so záznamami do projektu ([Pridávanie súborov do projektu](../processing-images-gui/adding-files-to-a-project.md)) a spracujte ich ako zvyčajne.
* **CLI**: nastavte `process` na **koreňovú zložku so záznamami**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` zvyčajne importuje len zložku, ktorú určíte, ale ak táto zložka neobsahuje žiadne obrázky a má podzložky, automaticky prechádza do nižších úrovní — takže podzložky na danej úrovni a zložka `daq/` sa načítajú naraz. Každé zachytenie sa importuje ako **jeden obrázok** s ostatnými úrovňami pripojenými ako režimy zobrazenia, nie ako jeden obrázok na úroveň.

Priame pomenovanie podzložky úrovne (napr. `…/captures/raw/`) tiež funguje, ale zložka `daq/` zostane vynechaná – skopírujte ju vedľa, keď-odvádzať radiometrický produkt z `raw/`, inak sa časová pečiatka nebude mať s čím porovnať.

{% hint style="warning" %}
**Spracovanie vždy začína od `raw`.** V rámci každého záznamu je zdrojom spracovateľského reťazca surový snímok; `debayered`, `radiance`, `reflectance`, a `preview` sa objavujú ako zobrazené režimy, ale nikdy sa nevracajú späť cez potrubie — opätovné spracovanie odvodeného produktu by znamenalo opätovné uplatnenie výpočtov vinetácie, farieb a jasu, ktoré sú už zapracované do jeho pixelov, preto Chloros uprednostňuje odmietnutie pred dvojitým spracovaním. Rendery `index/` a `composite/` sa vôbec nespracúvajú (sú to výstupy, nie záznamy). Zložka so záznamami uložená **bez** importu surových dát sa zobrazuje normálne, ale `process` ju preskočí a oznámi to; `--input-level {raw,debayered,processed}` je zámerný únikový mechanizmus, ktorý vynúti vstupný bod. Presné správy o preskočení nájdete v [Referencii CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).
{% endhint %}

Ďalšie dve správania, ktoré stojí za to poznať pri skriptovaní opätovného spracovania:

* Spustenie `chloros-cli process`, ktoré požiadalo o výstupy, ale nezapísalo **žiadne obrazové výstupy, zlyhá s jasnou chybou a ukončí sa s nenulovým kódom** — nikdy nedôjde k tichému prázdnemu spusteniu. Úspešné spustenia hlásia počet výstupov. (Úmyselné spustenie iba s metadátami sa stále považuje za úspech.)
* Opätovne importované spracované výstupy nikdy nezaberajú slot surových dát zachytenia — pôvodné surové dáta vždy zostávajú zdrojom spracovateľského reťazca.

## Ekvivalenty v CLI



Všetko na tejto stránke je možné ovládať bez grafického rozhrania. Režimy zachytávania v grafickom rozhraní (GUI) priamo zodpovedajú `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Jednorazový | `chloros-cli lattice array-capture` |
| Nepretržitý | `array-capture --continuous [--count N] [--duration S]` |
| Interval | `array-capture --interval S [--duration S]` |
| Najrýchlejšie zachytávanie | `array-capture --fastest` |
| Zaškrtávacie políčko „Aligned“ | `--aligned / --no-aligned` |
| Začiarkavacie políčka typu exportu | `--processing LEVEL` alebo `--levels L1,L2,…` (predvolené `all`) |
| Nahrávanie indexového videa | `chloros-cli lattice array-record` |
| Nahrávanie surových sériových snímok / Vytvorenie videa | `chloros-cli lattice array-burst` / `array-build-video` |

Úplné tabuľky príznakov, možnosť inteligentného automatického nastavenia expozície (smart-AE) (`--smart`) a model s konštantnou rýchlosťou sú uvedené v [Referencii CLI, § Režimy snímania, záznamníky a offline spracovanie](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

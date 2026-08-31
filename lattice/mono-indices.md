# Mono kamery a vegetačné indexy

## Jedna kamera = jedno spektrálne pásmo

Kamera **M3M**je monochromatickou verziou modelu Bayer**M3C**: ide o monochromatický senzor IMX265 s jedným úzkopásmovým interferenčným filtrom. Názov pásma je uvedený v reťazci modelu — `M3M-<lens>-F<wavelength>`, napr. `M3M-L87-F685` (zobrazené v Chloros ako `LATT-M3M-L87-F685`). Snímač poskytuje**jedno pásmo v odtieňoch šedej** bez Bayerovej mozaiky: nie je čo demosaikovať, nie je potrebné odstraňovať presluchy medzi kanálmi a nie je potrebné nastavovať vyváženie bielej.

Dôsledky, ktoré stojí za to poznať pred plánovaním monochromatického systému:

* **Žiarivosť a odrazivosť sú plne definované pre každé pásmo.**Ide o radiometrické mapy pre jednotlivé pásma, takže jedna kamera M3M produkuje kalibrovanú žiarivosť typu float32 (W/m²/sr/nm) a odrazivosť typu uint16 (`32768` = ρ 1,0) presne tak, ako to robí pásmo M3C. Mono snímky obsahujú**identitnú** maticu odozvy senzora — nie je potrebné ani sa neuplatňuje žiadne 3×3 rozloženie.
* **Jedna mono kamera nedokáže vypočítať index vegetácie.** NDVI, NDRE a podobné indexy vyžadujú aspoň dva pásma. Na výpočet indexov z mono hardvéru je potrebné kombinovať viacero kamier M3M — pozri nižšie.
* Kamery M3M prenášajú **Mono12** (12-bitové, 2 bajty/pixel v prenose), čo má význam pre [plánovanie šírky pásma pole](arrays.md#bandwidth-the-rules-of-thumb).

## Čo Chloros pri mono vynecháva — a ako vás o tom informuje

Fázy farebného spracovania sa na jednopásmový senzor jednoducho nevzťahujú. Chloros **tieto fázy preskakuje s jednoriadkovou správou** namiesto vyhodnotenia chyby a naďalej ich normálne spúšťa pre akúkoľvek kameru M3C (Bayer) v tej istej relácii:

| Fáza | Správanie v režime Mono (M3M) | Správanie v režime M3C |
| --- | --- | --- |
| Demosaic / debayer | Preskočené — výstupná úroveň `debayered` je jednokanálový obrázok v odtieňoch šedej. | 3-kanálový demosaic. |
| Vyváženie bielej (`lattice white-balance`) | Preskočené s jednoriadkovou správou. | Spúšťa sa normálne. |
| Farebný profil (`lattice color-profile`) | Preskočené s jednoriadkovou správou. | Spúšťa sa normálne. |
| Sýtosť/kontrast (`lattice color`) | Preskočené s jednoriadkovou správou. | Beží normálne. |
| Odstránenie spektrálneho presluchu | Identita (bez matice 3×3). | Použitá matica 3×3 pre každú kameru. |
| Žiarivosť / odrazivosť | **Beží** — pre každý pás, plne kalibrované. | Beží pre každý pás. |

GUI používa rovnaké obmedzenie: pre monochromatickú kameru panel nastavení pre každú kameru skryje riadky určené výlučne pre RGB (vyváženie bielej, gama, farebný profil, sýtosť, kontrast, rozdelenie kanálov) a živý histogram je uzamknutý na jedinú krivku **MONO**. Rozlišovacím znakom v celom stohu je token `M3M` v reťazci modelu, ktorý sa v grafickom rozhraní zobrazuje ako SDK.

## Indexy vyžadujú ≥ 2 pásma: zarovnanie → zoskupenie → indexovanie

Pracovný postup pre mono indexovanie pozostáva vždy z tých istých troch krokov:

1. **Zarovnanie** — nasmerujte niekoľko kamier M3M na rôzne vlnové dĺžky (napr. F650 „Red“ a F850 „NIR“), prepojte ich do [multikamerového poľa](arrays.md) a nechajte Chloros vypočítať deformáciu pre spoločnú registráciu medzi kamerami.
2. **Stack** — zarovnané snímky sa zjednotia do jedného viacpásmového obrazu (každá kamera prispieva jedným pomenovaným pásmom).
3. **Index** — vypočítajte indexový vzorec pre pásma v stacku, prípadne ho renderujte cez LUT.

V grafickom rozhraní (GUI) predstavuje celý tento reťazec režim zobrazenia poľa **Combined Cameras**: živý kompozit je už zarovnaný a kalkulátor indexu poľa (nižšie) definuje vzorec, podľa ktorého sa vykresľuje. Exportované zábery je možné deformovať do rovnakého zarovnania pomocou možnosti snímania**Aligned**.

## Kalkulátor indexu

Kalkulátor indexu vytvára indexový výraz používaný v živom náhľade a pri exportoch indexu pre jednotlivé kamery. Ide o jednu spoločnú plochu, ktorú možno otvoriť z dvoch miest na bočnom paneli karty „Kamery“:

* **Pre jednotlivé kamery**— Živý náhľad → ozubené koliesko**Index** (len kamery Bayer RGN/OCN/NGB; samostatná monochromatická kamera nemá ovládací prvok indexu, pretože jedno pásmo nemôže vytvoriť index).
* **Pre pole**— nastavenia poľa → Živé zobrazenie → ozubené koliesko**Index**. Toto je monochromatická cesta: zoznam pásiem zahŕňa**všetky kamery v poli**, takže monochromatický pár tu prispieva svojimi dvoma pásmami.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Ovládacie prvky, zhora nadol:

* **Čipy pásiem** („Pásma — kliknutím pridáte do výrazu“) — jedno tlačidlo na každé dostupné pásmo, označené názvom farby + vlnovou dĺžkou v nm (duplicitné názvy farieb sú odlíšené, napr. „Farba 850“). Kliknutím vložíte token pásma na miesto kurzora. Pásma z kamier, ktoré nedokážu produkovať žiarivosť na pásmo (RGB/FRGB), sú odfiltrované.
* **Tlačidlá operátorov a funkcií** — `+ - * / ( ) ^ ,` a `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Textové pole pre výraz** — voľne zadávaný vzorec; zástupný symbol zobrazuje klasickú formu NDVI ako `(NIR - Red) / (NIR + Red)`. Náhľad s tokenizáciou nad ním, určený len na čítanie, vykresľuje pásmové čipy, čísla a vlajky ako neznáme tokeny.
* **Banner platnosti**— sivý „Prázdne — nebude použitý žiadny index“; zelený „Platný výraz“; červený s konkrétnou chybou analýzy (neznámy pás, nejednoznačný pás zachytený viacerými kamerami, chýbajúca zátvorka, …); alebo oranžový, ak je výraz platný, ale**konštantný** (napr. `X/X`, alebo v menovateli NDVI zadané `−` namiesto `+`) — konštanta premietne celý snímok do jednej farby.
* Zobrazí sa samostatné oranžové varovanie, ak je použitý výraz v poriadku, ale **živý snímok je jednotvárny** (plochá alebo nasýtená scéna) — zistenie zrútenia histogramu sa vykoná automaticky.
* Vstupy **Apply LUT**(predvolene zapnuté; vypnuté = rozťahovanie stupníc šedej),**Level**2/3/5/7-stop (predvolene 7-stop) a**Min / Max**po bokoch lišty prechodu. Min je predvolene nastavené na**0,2**— zväčšuje farebnú škálu do rozsahu relevantného pre vegetáciu, zatiaľ čo hodnoty pod touto hodnotou prechádzajú ako stupnice šedej; nastavte Min na −1 pre celý rozsah indexu (tlačidlo**Reset** obnoví rozsah −1…+1). Max je predvolene nastavené na 1.
* **Živý histogram** rozdelenia indexu — pruhy v kvadratickom meradle, jantárové čiary 2. a 98. percentilu, biela mediánová čiara a údaje o hodnotách mimo rozsahu („◀ N % &lt; lo“ / „hi &lt; N % ▶“), ktoré sa nad 1 % zmenia na jantárové ako signál na rozšírenie okna Min/Max.
* Tlačidlo **Apply**potvrdí výraz v živom prúde; úpravy LUT sa uplatňujú v reálnom čase bez stlačenia tlačidla Apply. Výrazy sú zámerne**len pre danú reláciu** — medzi reláciami sa neukladajú.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## Cesta CLI

Ten istý reťazec zarovnanie → zásobník → index, skriptovateľný od začiatku do konca:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` priraďuje symboly predvoľby k názvom pásiem v zásobníku. Dve pravidlá vám ušetria neúspešné spustenie:

* **Pri symboloch sa rozlišujú veľké a malé písmená** a musia sa presne zhodovať s názvami kanálov prednastavenia — prednastavenia používajú malé písmená (NDVI sú `red`,`nir`; skontrolujte `--list-presets`). `--channel red=Red_660` funguje; `--channel RED=660` zlyhá s chybou `channel_map missing entries`.
* Strana pásma musí uviesť názov pásma v zarovnanom zozname (`lattice align-info --profile align.json` ich vypisuje). Režim offline tiež akceptuje indexy pásiem počítané od 0, napr. `--channel red=0 --channel nir=1`.

`lattice index` tiež beží úplne offline na základe uloženého zarovnaného viacpásmového súboru TIFF:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Prednastavenia indexov

`lattice index --preset` (a [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) na karte Obrázok, ktorý používa rovnaký engine) obsahuje týchto **22 prednastavení**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Pre vzorce a symboly kanálov jednotlivých predvolieb spustite `chloros-cli lattice index --list-presets` a pre dostupné farebné prechody spustite `--list-gradients`. Vlastné vzorce používajú `--formula EXPR` s rovnakou syntaxou ako Kalkulátor indexov. Upozorňujeme, že tento zoznam predvolieb je špecifický pre indexovací modul LATTICE — roletové menu „Spracovanie“ v nastaveniach projektu pre importované snímky obsahuje iný zoznam (pozri [Vzorce multispektrálnych indexov](../project-settings/multispectral-index-formulas.md)).

Kompletná sada príznakov (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, ovládače deformácie zarovnania pre `--live`, a ďalšie) je zdokumentovaná v [Referencii k CLI § Index / Matematika vegetácie](../reference/cli-reference.md#index--vegetation-maths); ekvivalenty SDK sa nachádzajú v [Referencii SDK](../reference/sdk-reference.md).

## Zachytávanie indexových produktov z mono poľa

Pri pripojenom poli a aplikovanom indexovom výraze funkcia `array-capture` (alebo vo grafickom rozhraní **Zachytiť všetko**) uloží úrovne exportu pre každú kameru *a* indexové vykreslenie — `--index`/`--no-index` to prepína na CLI a zachytáva štandardne všetky príslušné úrovne. Príspevok monokamery do každej skupiny záznamov tvorí jej jedno pásmo na úrovniach surových údajov/odstránenia debayeringu (stupnice šedi)/radiancie/reflektancie, plus zdieľaný kompozit kombinovaného indexu, ak pole beží v kombinovanom režime. Pozri [Sústavy s viacerými kamerami § Zachytávanie](arrays.md#capturing-monitoring-vs-analysis).

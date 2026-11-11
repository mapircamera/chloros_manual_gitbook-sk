# Mono kamery a indexy vegetácie

## Jedna kamera = jedno pásmo

Kamera **M3M**je monochromatickou verziou modelu Bayer**M3C**: ide o monochromatický senzor IMX265 s jedným úzkopásmovým interferenčným filtrom. Názov pásma je uvedený v reťazci modelu — `M3M-<lens>-F<wavelength>`, napr. `M3M-L87-F685` (v programe Chloros sa zobrazuje ako `LATT-M3M-L87-F685`). Snímač poskytuje **jedno pásmo v odtieňoch šedej** bez Bayerovej mozaiky: nie je čo demosaikovať, nie je potrebné odstraňovať presluchy medzi kanálmi a nie je potrebné nastavovať vyváženie bielej.

Dôsledky, ktoré stojí za to poznať pred plánovaním monochromatického systému:

* **Žiarivosť a odrazivosť sú plne definované pre každé pásmo.**Ide o radiometrické mapy pre jednotlivé pásma, takže jedna kamera M3M produkuje kalibrovanú žiarivosť typu float32 (W/m²/sr/nm) a odrazivosť typu uint16 (`32768` = ρ 1,0) presne tak, ako to robí pásmo M3C. Mono snímky obsahujú**identitnú** maticu odozvy senzora — nie je potrebné ani sa neuplatňuje žiadne 3×3 rozloženie.
* **Jedna mono kamera nedokáže vypočítať index vegetácie.** NDVI, NDRE a podobné indexy vyžadujú aspoň dve pásma. Na výpočet indexov z mono hardvéru je potrebné kombinovať viacero kamier M3M — pozri nižšie.
* Kamery M3M prenášajú **Mono12** (12-bitové: 2 bajty/pixel pri prenose, alebo 1,5 bajtu/pixel, ak pole používa [balený 12-bitový prenos](arrays.md#packed-12-bit-transport)), čo má význam pre [plánovanie šírky pásma poľa](arrays.md#bandwidth-how-many-cameras-fit).

## Čo Chloros pri mono preskakuje — a ako vás o tom informuje

Fázy farebného spracovania sa na jednopásmový senzor jednoducho nevzťahujú. Chloros **ich preskočí s jednoriadkovou správou** namiesto vygenerovania chyby a naďalej ich beží normálne pre akúkoľvek kameru M3C (Bayer) v tej istej relácii:

| Fáza | Správanie v režime Mono (M3M) | Správanie v režime M3C |
| --- | --- | --- |
| Demosaic / debayer | Preskočené — výstupná úroveň `debayered` je jednokanálový obrázok v odtieňoch šedej. | 3-kanálový demosaic. |
| Vyváženie bielej (`lattice white-balance`) | Preskočené s jednoriadkovou správou. | Spúšťa sa normálne. |
| Farebný profil (`lattice color-profile`) | Preskočené s jednoriadkovou správou. | Spúšťa sa normálne. |
| Sýtosť/kontrast (`lattice color`) | Preskočené s jednoriadkovou správou. | Beží normálne. |
| Odstránenie spektrálneho presluchu | Identita (bez matice 3×3). | Použitá matica 3×3 pre každú kameru. |
| Žiarivosť / odrazivosť | **Beží** — pre každý pás, plne kalibrované. | Beží pre každý pás. |

GUI používa rovnaké obmedzenie: v prípade monochromatickej kamery panel nastavení pre každú kameru skryje riadky týkajúce sa iba RGB (vyváženie bielej, gama, farebný profil, sýtosť, kontrast, rozdelenie kanálov) a živý histogram je uzamknutý na jedinú stopu **MONO**. Rozlišovacím znakom v celom stohu je token `M3M` v reťazci modelu, ktorý sa v grafickom rozhraní/SDK-u zobrazuje ako `is_mono`.

## Indexy vyžadujú ≥ 2 pásma: zarovnanie → stohovanie → indexovanie

Pracovný postup pre monoindex sa vždy skladá z tých istých troch krokov:

1. **Zarovnanie** — nasmerujte niekoľko kamier M3M na rôzne vlnové dĺžky (napr. F650 „Red “ a F850 „NIR “), prepojte ich do [sústavy viacerých kamier](arrays.md) a nechajte program Chloros vypočítať deformáciu pre spoločnú registráciu medzi kamerami.
2. **Stack** — zarovnané snímky sa zjednotia do jedného multibandového obrazu (každá kamera prispieva jedným pomenovaným pásmom).
3. **Index** — vypočíta indexový vzorec pre pásma zoskupenia, prípadne ho vykreslí prostredníctvom LUT.

V grafickom rozhraní je celý tento reťazec zobrazený v režime **Combined Cameras**: živý kompozitný obraz je už zarovnaný a kalkulátor indexu zoskupenia (nižšie) definuje vzorec, podľa ktorého sa vykresľuje. Zaznamenané exporty je možné deformovať do rovnakého zarovnania pomocou možnosti snímania**Aligned**.

## Kalkulátor indexu

Kalkulátor indexu vytvára indexový výraz používaný v živom náhľade a pri exportoch indexu pre jednotlivé kamery. Ide o jednu zdieľanú plochu, ktorú je možné otvoriť z dvoch miest v bočnom paneli karty „Kamery“:

* **Pre jednotlivé kamery**— Živý náhľad → ozubené koliesko**Index** (len kamery RGN/OCN/NGB typu Bayer; samostatná monochromatická kamera nemá ovládanie indexu, pretože jedno pásmo nemôže vytvoriť index).
* **Pre pole**— nastavenia poľa → Živé náhľady → ozubené koliesko**Index**. Toto je mono cesta: zoznam pásiem zahŕňa**všetky kamery v poli**, takže mono pár tu prispieva svojimi dvoma pásmami.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Jeho ovládacie prvky, zhora nadol:

* **Čipy pásiem** („Pásma — kliknutím pridáte do výrazu“) — jedno tlačidlo na každé dostupné pásmo, označené názvom farby + vlnovou dĺžkou v nm (duplicitné názvy farieb sú odlíšené, napr. „Farba 850“). Kliknutím vložíte token pásma na miesto kurzora. Pásma z kamier, ktoré nedokážu generovať žiarivosť na jednotlivé pásma (RGB/FRGB), sú odfiltrované.
* **Tlačidlá operátorov a funkcií** — `+ - * / ( ) ^ ,` a `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Textové pole pre výraz** — voľne zadávaný vzorec; zástupný symbol zobrazuje klasickú formu NDVI `(NIR - Red) / (NIR + Red)`. Nade ním sa nachádza náhľad v režime len na čítanie, ktorý zobrazuje pásmové čipy, čísla a označuje neznáme symboly vlajkami.
* **Banner platnosti**— sivý „Prázdne — nebude použitý žiadny index“; zelený „Platný výraz“; červený s konkrétnou chybou analýzy (neznámy pás, nejednoznačný pás zachytený viacerými kamerami, chýbajúca zátvorka, …); alebo oranžový, ak je výraz platný, ale**konštanta** (napr. `X/X` alebo menovateľ NDVI zadaný ako `−` namiesto `+`) — konštanta priradí celému snímku jednu farbu.
* Samostatné oranžové varovanie sa zobrazí, ak je použitý výraz v poriadku, ale **živý snímok je jednotvárny** (plochá alebo nasýtená scéna) — zistenie zrútenia histogramu pre vás.
* Vstupy **Použiť LUT**(predvolene zapnuté; vypnuté = rozťahovanie stupňov šedej),**Úroveň**2/3/5/7-stop (predvolene 7-stop) a**Min / Max**po stranách lišty prechodu. Min je predvolene nastavené na**0,2**— zväčšuje farebnú škálu do rozsahu relevantného pre vegetáciu, zatiaľ čo hodnoty pod touto hranicou prechádzajú ako šedá škála; nastavte Min na −1 pre celý rozsah indexu (tlačidlo**Reset** obnoví rozsah −1…+1). Max je predvolene nastavené na 1.
* **Živý histogram** rozdelenia indexu — stĺpce v kvadratickom meradle, jantárové čiary 2. a 98. percentilu, biela mediánová čiara a údaje o chvostoch mimo rozsahu („◀ N% < lo“ / „hi < N% ▶“), ktoré sa nad 1 % zmenia na jantárové ako signál na rozšírenie okna Min/Max.
* Tlačidlo **Apply** potvrdí výraz v živom prúde; úpravy LUT sa uplatňujú v reálnom čase bez stlačenia tlačidla Apply. Ak je projekt otvorený, nastavenia výrazu a LUT sa uložia spolu s projektom a obnovia sa pri jeho opätovnom otvorení; bez projektu trvajú len počas danej relácie.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->



## Cesta v CLI



Ten istý reťazec zarovnanie → zoskupenie → index, skriptovateľný od začiatku do konca:

```bash
chloros-cli lattice align-calibrate --serials SN_RED,SN_NIR --save align.json
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`align-calibrate` používa prepojené pole, ktoré obsahuje jeho kamery, alebo ich otvorí sám pre jedno spustenie. `index --live` vždy otvára kamery sám ([dočasné pole](../reference/cli-reference.md#multi-camera-transient-sessions)) a nemá `--serials`: otvára každú kameru v sieti. Preto uvoľnite každú kameru, ktorú Chloros drží pred tým – GUI, `cam-connect`, `array-connect` (`chloros-cli lattice array-disconnect --all`): otvorenie zadržanej kamery druhýkrát môže zastaviť prúd jej držiteľa alebo spôsobiť pád backendu. Kamera, ktorá nedodáva žiadny snímok, spôsobí zlyhanie behu a nezapíše sa žiadny zásobník ani index.

`--channel` priraďuje symboly predvolby k názvom pásiem v zásobníku. Dve pravidlá vám pomôžu vyhnúť sa zlyhaniu behu:

* **Pri symboloch sa rozlišujú veľké a malé písmená** a musia sa presne zhodovať s názvami kanálov predvolby — predvolby používajú malé písmená (predvolby NDVI sú `red`, `nir`; pozri `--list-presets`). `--channel red=Red_660` funguje; `--channel RED=660` zlyhá s chybou `channel_map missing entries`.
* Strana pásma musí uvádzať názov pásma v zarovnanom stohu (`lattice align-info --profile align.json` ich uvádza). Režim offline akceptuje aj indexy pásiem počítané od 0, napr. `--channel red=0 --channel nir=1`.

`lattice index` beží aj offline na základe uloženého zarovnaného viacpásmového súboru TIFF. Každý formulár beží na backende, takže musí byť spustený a prihlásený na platenej úrovni Chloros+; relatívny `--input` alebo `--output` sa vyrieši voči zložke, z ktorej spúšťate príkaz:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Prednastavenia indexu

`lattice index --preset` (a položka [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) na karte Obrázok, ktorá používa ten istý engine) obsahuje týchto **22 prednastavení**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Pre vzorce a symboly kanálov jednotlivých predvolieb spustite `chloros-cli lattice index --list-presets` a pre dostupné farebné prechody spustite `--list-gradients`. Vlastné vzorce používajú `--formula EXPR` s rovnakou syntaxou ako Kalkulátor indexov. Upozorňujeme, že tento zoznam predvolieb je špecifický pre indexovací modul LATTICE — roletové menu „Spracovanie“ v nastaveniach projektu pre importované snímky obsahuje iný zoznam (pozri [Vzorce multispektrálnych indexov](../project-settings/multispectral-index-formulas.md)).

Kompletná sada príznakov (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, ovládače zarovnania a deformácie pre `--live` a ďalšie) je zdokumentovaná v [Referencii CLI § Index / Vegetation Maths](../reference/cli-reference.md#index--vegetation-maths); ekvivalenty v SDK sa nachádzajú v [Referencii SDK](../reference/sdk-reference.md).

## Zachytávanie indexových produktov z monokromatického poľa

Pri pripojenom poli a aplikovanom indexovom výraze funkcia `array-capture` (alebo GUI **Zachytiť všetko**) uloží úrovne exportu pre každú kameru *a* indexové vykreslenie — `--index`/`--no-index` to prepína v CLI a zachytáva štandardne všetky príslušné úrovne. Príspevok mono kamery do každej skupiny záznamov tvorí jej jedno pásmo na úrovniach surových údajov/odstráneného debayeringu (šedá škála)/radiancie/odrazivosti, plus zdieľaný kompozit kombinovaného indexu, ak pole beží v kombinovanom režime. Pozri [Sústavy s viacerými kamerami § Zaznamenávanie](arrays.md#capturing-monitoring-vs-analysis).

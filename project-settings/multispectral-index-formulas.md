---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Vzorce multispektrálnych indexov

Nižšie uvedené vzorce indexov využívajú kombináciu rozsahov priemernej priepustnosti filtra Survey3:

<table><thead><tr><th align="center">Survey3 Farba filtra</th><th width="196.199951171875" align="center">Survey3 Názov filtra</th><th width="159.800048828125" align="center">Rozsah priepustnosti (FWHM)</th><th align="center">Priemerná priepustnosť</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN – Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB – Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN – Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN – Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN – NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR – NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>Pri použití týchto vzorcov sa názov môže končiť na „\_1“ alebo „\_2“, čo zodpovedá tomu, ktorý filter NIR bol použitý buď NIR1, alebo NIR2.

V prípade kamier LATTICE M3C (trojité pásmové filtre typu Bayer) používa ten istý indexovací modul pásma filtrov M3C:

| Filter M3C | Pásmo 1 (stred/FWHM) | Pásmo 2 (stred/FWHM) | Pásmo 3 (stred/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

Kamery LATTICE M3M sú jednopásmové (jeden úzkopásmový filter na kameru), preto sa pre samostatný obraz z kamery M3M nevypočítavajú viacpásmové indexy. Na výpočet indexov s M3M skombinujte dve alebo viac kamier do zarovnaného viacpásmového súboru a použite indexovací modul LATTICE (`chloros-cli lattice index` alebo nástroj na výpočet indexov v reálnom čase v grafickom rozhraní).

***

## Kde funguje každý názov indexu

Chloros má **tri** indexové plochy a ich prednastavené zoznamy nie sú identické. V tejto časti skontrolujte, či daný názov bude fungovať tam, kde ho plánujete použiť.

| Kde sa nachádzate | Ktorý zoznam platí | Počet |
| --- | --- | --- |
| Nastavenia projektu → Index → Pridať index (GUI) | Povrch 1 | 27 |
| Prehliadač obrázkov [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Povrch 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Povrch 2 | 22 |
| SDK `process_folder(indices=[...])` | Povrch 2 | 22 |
| `chloros-cli lattice index --preset` | Povrch 3 | 22 (iné číslo 22) |
| Kartu „Kamery“ – kalkulátor indexu v reálnom čase | Surface 3 | 22 (iné číslo 22) |

Surface 1 a 2 pracujú s **jedným snímkom naraz z jednej kamery**, pričom používajú symbolové sloty `x`/`y`/`z`(/`a`), ktoré sú viazané na filtračné kanály danej kamery. Surface 3 pracuje s**zarovnaným viacpásmovým súborom snímok** — niekoľko kamier LATTICE zarovnaných do jedného kockového súboru — a kanály označuje malými písmenami.

### 1. Nastavenia projektu v grafickom rozhraní / roletové menu sandboxu prehliadača obrázkov — 27 vzorcov

Roletové menu ich uvádza v tomto poradí (ide o poradie vloženia, nie abecedné):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

V grafickom rozhraní pretiahnete filtračné kanály vašej kamery na pozície pásiem vo vzorci, takže akýkoľvek vzorec je možné použiť s akýmkoľvek priradením pásiem, ktoré vaša kamera podporuje. Vlastné vzorce, ktoré ste uložili, sú pridané pod tento zoznam.

**Päť vzorcov dostupných iba v grafickom rozhraní** — tie, ktoré zoznam CLI/SDK `--indices` neprijíma — sú implementované takto:

| Prednastavenie iba pre grafické rozhranie | Vzorec (ako je implementovaný) | Sloty |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (štyri polia) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Zámerné priradenie pre každý z nich je uvedené v samostatnej časti nižšie na tejto stránke (napríklad GARI očakáva x=Green, y=NIR, z=Blue, a = Red). GARI je jediný vzorec v Chloros, ktorý používa štvrtý slot.

### 2. Rozšírenie názvu CLI / SDK `--indices` — 22 predvolieb

Možnosť `chloros-cli process --indices` (a parameter SDK `indices`) akceptuje tieto názvy predvolieb:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Názvy s neznámym indexom sa bez upozornenia preskočia.** Názov, ktorý nie je v tomto zozname (vrátane piatich vzorcov určených výlučne pre grafické rozhranie `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` a akýkoľvek vlastný vzorec, ktorý ste uložili v grafickom rozhraní) sa vynechá len s upozornením v protokole – spustenie pokračuje bez tohto indexu a samotné spustenie sa aj naďalej vyhodnotí ako úspešné. Upozornenie sa vypíše takto:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Názvy sa porovnávajú bez ohľadu na veľkosť písmen po odstránení medzier, takže `ndvi`, `NDVI` a ` NDVI ` sú rovnaké prednastavenia. Prednastavenie sa tiež preskočí, ak vyžaduje pásmo, ktoré filter vašej kamery neposkytuje.
{% endhint %}

Presné vzorce tak, ako sú implementované (symboly `x`/`y`/`z` sú pozície pásiem; predvolené priradenie je uvedené pre každú predvoľbu):

| Predvoľba | Vzorec (tak, ako je implementovaný) | Predvolený filter | Sloty (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Ako sa z prednastaveného názvu stanú pozície pásiem

Keď zadáte holý názov, napríklad `NDVI`, program Chloros musí rozhodnúť, ktorý kanál ktorého súboru číta každý symbol. Používa na to túto tabuľku, ktorá priraďuje kód filtra k pozícii v poli každého kanála:

| Kód filtra | Kanál → index poľa |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` je akceptovaný ako alias pre Orange, tiež 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

**Predvolený filter** prednastavenia (stĺpec „Predvolený filter“ vyššie) sa použije, ak projekt obsahuje obrázky s týmto filtrom. Ak tomu tak nie je, Chloros prehľadá filtre skutočne prítomné v projekte v poradí `RGN, OCN, NGB, RGB, RE, NIR` a vyberie prvý, ktorý dokáže poskytnúť všetky kanály, ktoré prednastavenie potrebuje. Ak žiadny z nich to nedokáže, prednastavenie sa pre dané spustenie vynechá. Preto `NDVI` požiadaný na súbore údajov obsahujúcom iba OCN stále produkuje zmysluplný výsledok — viaže sa na pozície OCNpozíciami Orange a NIR.

Reťazce modelu LATTICE M3C obsahujú filter s predponou `F` (`LATT-M3C-L41-FRGN`), ale táto predpona sa vynechá pri načítaní kódu filtra z obrazu, takže kamera FRGN spracováva údaje cez riadok `RGN` uvedený vyššie a nevyžaduje si žiadne špeciálne zaobchádzanie.

### 3. Indexovací modul LATTICE (`lattice index --preset`, kalkulátor indexu v reálnom čase) — 22 predvolieb

Modul LATTICE pracuje so zarovnanými viacpásmovými sériami snímok (živé polia alebo exportované viacpásmové súbory TIFF) a používa názvy kanálov v malých písmenách (`red`, `green`, `blue`, `red_edge`, `nir`). Jeho zoznam predvolieb sa líši od dvoch vyššie uvedených:

| Predvoľba | Vzorec | Kanály |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | červená, NIR |
| GNDVI | `(nir - green) / (nir + green)` | zelená, nir |
| BNDVI | `(nir - blue) / (nir + blue)` | modrá, nir |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | červená\_okraj, nir |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | modrá, zelená, nir |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | červená, nir |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | červená, nir |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | červená, NIR |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | modrá, červená, NIR |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | červená, nir |
| CVI | `(nir / green) - (red / green)` | červená, zelená, NIR |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | červená, nir |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | červená, nir |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | červená, zelená, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | červená, zelená, modrá |
| NGRDI | `(green - red) / (green + red)` | červená, zelená |
| VARI | `(green - red) / (green + red - blue)` | červená, zelená, modrá |
| TGI | `green - 0.39*red - 0.61*blue` | červená, zelená, modrá |
| EXG | `2*green - red - blue` | červená, zelená, modrá |
| CIRE | `(nir / red_edge) - 1` | červená\_okraj, nir |
| CIGREEN | `(nir / green) - 1` | zelená, nir |
| NDWI | `(green - nir) / (green + nir)` | zelená, nir |

Spustite príkaz `chloros-cli lattice index --list-presets` na vytlačenie tejto tabuľky z vašej nainštalovanej verzie a príkaz `--list-gradients` na zobrazenie dostupných farebných prechodov. Pri symboloch kanálov sa rozlišujú veľké a malé písmená a musia sa zhodovať s názvami predvolieb písanými malými písmenami (napr. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI

Ako je implementované v grafickom rozhraní a v zozname predvolieb CLI/SDK, CVI je vzorec pre pomer-pomerov:

$$
CVI = {(z / y) \over (x / y)}
$$

s predvoleným mapovaním kanálov RGB x=Red, y=Green, z=Blue. V grafickom rozhraní môžete akýkoľvek kanál vašej kamery pretiahnuť do slotov x/y/z. Upozorňujeme, že prednastavenie `CVI` indexového modulu LATTICE používa iný vzorec, `(NIR / Green) - (Red / Green)` — pre povrch, ktorý používate, si pozrite tabuľky vyššie.

***

## ENDVI – Vylepšený normalizovaný index vegetácie

Tento index okrem NIR a zeleného kanála využíva aj modrý kanál a je obľúbený pri kamerách s filtrom NGB, kde modré pásmo nahrádza červené.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Implementácia je symbolický vzorec `((x+y)-(2*z))/((x+y)+(2*z))` — priraďte kanály NIR a Green vašej kamery do pozícií x/y a Blue do pozície z (pre kameru NGB: x=NIR, y=Green, z=Blue).

***

## EVI – Vylepšený index vegetácie

Tento index bol pôvodne vyvinutý na použitie s údajmi MODIS ako vylepšenie indexu NDVI prostredníctvom optimalizácie signálu vegetácie v oblastiach s vysokým indexom listovej plochy (LAI). Je najužitočnejší v oblastiach s vysokými hodnotami LAI, kde môže dôjsť k nasýteniu indexu NDVI. Využíva oblasť modrej odrazivosti na korekciu signálov pôdneho pozadia a na zníženie vplyvov atmosféry, vrátane rozptylu aerosólov.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Hodnoty EVI by sa mali v prípade pixelov s vegetáciou pohybovať v rozmedzí od 0 do 1. Svetlé prvky, ako sú mraky a biele budovy, spolu s tmavými prvkami, ako je voda, môžu spôsobiť anomálne hodnoty pixelov v snímke EVI. Pred vytvorením snímky EVI by ste mali z odrazového obrazu odstrániť mraky a svetlé prvky a voliteľne nastaviť prahové hodnoty pixelov od 0 do 1.

_Referencia: Huete, A., et al. „Prehľad radiometrických a biofyzikálnych vlastností vegetačných indexov MODIS.“ Remote Sensing of Environment 83 (2002): 195–213._

***

## FCI1 – Index lesného porastu 1

_Iba v grafickom rozhraní — nie je k dispozícii ako prednastavenie CLI/SDK `--indices`._

Tento index rozlišuje lesný porast od iných typov vegetácie pomocou multispektrálnych snímok odrazivosti, ktoré obsahujú pásmo červeného okraja.

$$
FCI1 = Red * RedEdge
$$

Lesnaté oblasti budú mať nižšie hodnoty FCI1 v dôsledku nižšej odrazivosti stromov a prítomnosti tieňov v korunách stromov.

_Referencia: Becker, Sarah J., Craig S.T. Daughtry a Andrew L. Russ. „Robust forest cover indices for multispectral images.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Index lesného porastu 2

_Iba v grafickom rozhraní (GUI) – nie je k dispozícii ako prednastavenie CLI/SDK `--indices`._

Tento index rozlišuje lesný porast od iných typov vegetácie pomocou multispektrálnych snímok odrazivosti, ktoré neobsahujú pásmo červenej hrany.

$$
FCI2 = Red * NIR
$$

Lesnaté oblasti budú mať nižšie hodnoty FCI2 kvôli nižšej odrazivosti stromov a prítomnosti tieňov v korunách stromov.

_Referencia: Becker, Sarah J., Craig S.T. Daughtry a Andrew L. Russ. „Robustné indexy lesného porastu pre multispektrálne snímky.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – Index globálneho monitorovania životného prostredia

_Iba GUI – nie je k dispozícii ako prednastavenie CLI/SDK `--indices`._

Tento nelineárny index vegetácie sa používa na globálne monitorovanie životného prostredia na základe satelitných snímok a snaží sa korigovať vplyvy atmosféry. Je podobný indexu NDVI, je však menej citlivý na vplyvy atmosféry. Ovplyvňuje ho holá pôda, preto sa neodporúča používať v oblastiach s riedkou alebo stredne hustou vegetáciou.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Kde:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referencia: Pinty, B. a M. Verstraete. GEMI: nelineárny index na monitorovanie globálnej vegetácie zo satelitov. Vegetation 101 (1992): 15–20._

***

## GARI – Green – index odolný voči atmosférickým vplyvom

_Iba v grafickom rozhraní (GUI) – nie je k dispozícii ako CLI/SDK `--indices` prednastavenie._

Tento index je citlivejší na široký rozsah koncentrácií chlorofylu a menej citlivý na atmosférické vplyvy ako NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Konštanta gama je váhová funkcia, ktorá závisí od podmienok aerosólov v atmosfére. Program ENVI používa hodnotu 1,7, čo je odporúčaná hodnota podľa Gitelsona, Kaufmana a Merzylaka (1996, s. 296).

_Referencia: Gitelson, A., Y. Kaufman a M. Merzylak. „Využitie kanála Green pri diaľkovom snímaní globálnej vegetácie z EOS-MODIS.“ Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – Green – Index chlorofylu

Tento index sa používa na odhad obsahu chlorofylu v listoch u širokého spektra rastlinných druhov.

$$
GCI = {NIR \over Green} - 1
$$

Využitie širokého spektra vlnových dĺžok v oblasti NIR a zelenej farby umožňuje lepšie odhadovať obsah chlorofylu a zároveň poskytuje vyššiu citlivosť a lepší pomer signálu k šumu.

_Referencia: Gitelson, A., Y. Gritz a M. Merzlyak. „Vzťahy medzi obsahom chlorofylu v listoch a spektrálnou odrazivosťou a algoritmy na nedestruktívne hodnotenie chlorofylu v listoch vyšších rastlín.“ Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green Listový index

Tento index bol pôvodne navrhnutý na použitie s digitálnou kamerou RGB na meranie pokrytia pšenicou, pričom červené, zelené a modré digitálne hodnoty (DN) sa pohybujú v rozmedzí od 0 do 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Hodnoty GLI sa pohybujú v rozmedzí od -1 do +1. Záporné hodnoty predstavujú pôdu a neživé prvky, zatiaľ čo kladné hodnoty predstavujú zelené listy a stonky.

_Referencia: Louhaichi, M., M. Borman a D. Johnson. „Prostorovo lokalizovaná platforma a letecká fotografia na dokumentáciu vplyvov pasenia na pšenicu.“ Geocarto International 16, č. 1 (2001): 65–70._

***

## GNDVI – Green Normalizovaný index vegetácie

Tento index je podobný indexu NDVI s tým rozdielom, že meria zelené spektrum v rozmedzí 540 až 570 nm namiesto červeného spektra. Tento index je citlivejší na koncentráciu chlorofylu ako index NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referencia: Gitelson, A., a M. Merzlyak. „Diaľkové snímanie koncentrácie chlorofylu v listoch vyšších rastlín.“ Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – Green Optimalizovaný index vegetácie upravený o pôdu

Tento index bol pôvodne navrhnutý s využitím farebno-infračervenej fotografie na predpovedanie potrieb dusíka u kukurice. Je podobný indexu OSAVI, avšak namiesto zeleného pásma používa červené.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referencia: Sripada, R., et al. „Určovanie sezónnych potrieb dusíka pre kukuricu pomocou leteckej farebnej infračervenej fotografie.“ Doktorandská práca, North Carolina State University, 2005._

***

## Pomerný vegetačný index GRVI – Green

Tento index je citlivý na rýchlosť fotosyntézy v lesných porastoch, keďže odrazovosť zeleného a červeného spektra je silne ovplyvnená zmenami v listových pigmentoch.

$$
GRVI = {NIR \over Green }
$$

_Referencia: Sripada, R., et al. „Letecká farebná infračervená fotografia na stanovenie počiatočných potrieb dusíka v kukurici v priebehu sezóny.“ Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – Green Vegetčný index upravený o pôdu

Tento index bol pôvodne navrhnutý s využitím farebnej infračervenej fotografie na predpovedanie potrieb dusíka pre kukuricu. Je podobný indexu SAVI, avšak namiesto zeleného pásma používa červené.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referencia: Sripada, R., et al. „Určovanie sezónnych potrieb dusíka pre kukuricu pomocou leteckej farebnej infračervenej fotografie.“ Doktorandská práca, North Carolina State University, 2005._

***

## LAI - Index listovej plochy

Tento index sa používa na odhad pokrytia listov a na prognózovanie rastu a výnosu plodín. Program ENVI vypočítava zelený index LAI pomocou nasledujúceho empirického vzorca od Boegha a kol. (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Kde EVI je:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Vysoké hodnoty LAI sa zvyčajne pohybujú v rozmedzí približne od 0 do 3,5. Ak však scéna obsahuje mraky a iné jasné prvky, ktoré spôsobujú presýtenie pixelov, hodnoty LAI môžu prekročiť 3,5. Ideálne by ste mali pred vytvorením obrázku LAI z vašej scény odstrániť mraky a jasné prvky.

_Referencia: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde a A. Thomsen. „Letecké multispektrálne údaje na kvantifikáciu indexu listovej plochy, koncentrácie dusíka a fotosyntetickej účinnosti v poľnohospodárstve.“ Remote Sensing of Environment 81, č. 2–3 (2002): 179–193._

***

## LCI – Index chlorofylu v listoch

_Iba v grafickom rozhraní (GUI) – nie je k dispozícii ako prednastavenie CLI/SDK `--indices`._

Tento index sa používa na odhad obsahu chlorofylu vo vyšších rastlinách a je citlivý na zmeny odrazivosti spôsobené absorpciou chlorofylu.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referencia: Datt, B. „Diaľkové snímanie obsahu vody v listoch eukalyptu.“ Journal of Plant Physiology 154, č. 1 (1999): 30–36._

***

## MNLI – modifikovaný nelineárny index

Tento index je vylepšením nelineárneho indexu (NLI), ktorý zahŕňa index vegetácie upravený o pôdu (SAVI) s cieľom zohľadniť pôdné pozadie. Program ENVI používa faktor úpravy pozadia korún stromov (_L_) s hodnotou 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referencia: Yang, Z., P. Willis a R. Mueller. „Vplyv snímky AWIFS vylepšenej pomerom pásiem na presnosť klasifikácie plodín.“ Zborník z sympózia Pecora 17 o diaľkovom prieskume (2008), Denver, CO._

***

## MSAVI2 – Modifikovaný index vegetácie upravený o pôdu 2

Tento index je zjednodušenou verziou indexu MSAVI navrhnutého Qi a kol. (1994), ktorý vylepšuje index vegetácie upravený o pôdu (SAVI). Znižuje pôdný šum a zvyšuje dynamický rozsah signálu vegetácie. MSAVI2 je založený na induktívnej metóde, ktorá na zvýraznenie zdravej vegetácie nepoužíva konštantnú hodnotu _L_ (ako v prípade SAVI).

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referencia: Qi, J., A. Chehbouni, A. Huete, Y. Kerr a S. Sorooshian. „Modifikovaný index vegetácie upravený podľa pôdy.“ Remote Sensing of Environment 48 (1994): 119–126._

***

## MSR – Modifikovaný jednoduchý pomer

Tento index je modifikáciou jednoduchého pomeru NIR/Red navrhnutou s cieľom linearizovať jeho vzťah s biofyzikálnymi parametrami a je citlivejší ako NDVI pri vyšších hustotách vegetácie.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Referencia: Chen, J. „Hodnotenie vegetačných indexov a modifikovaného jednoduchého pomeru pre boreálne aplikácie.“ Canadian Journal of Remote Sensing 22 (1996): 229–242._

***

## NDRE – Normalizovaný rozdiel RedEdge

Tento index je podobný indexu NDVI, avšak porovnáva kontrast medzi NIR a RedEdge namiesto Red, ktorý často skôr detekuje stres vegetácie.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalizovaný rozdielový vegetačný index

Tento index je meradlom zdravej, zelenej vegetácie. Kombinácia jeho normalizovaného rozdielového vzorca a využitia oblastí s najvyššou absorpciou a odrazivosťou chlorofylu zaisťuje jeho spoľahlivosť v širokom spektre podmienok. V podmienkach hustého porastu sa však môže nasýtiť, keď hodnota LAI dosiahne vysokú úroveň.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Hodnota tohto indexu sa pohybuje v rozmedzí od -1 do 1. Bežný rozsah pre zelenú vegetáciu je 0,2 až 0,8.

_Referencia: Rouse, J., R. Haas, J. Schell a D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Tretie sympózium ERTS, NASA (1973): 309–317._

***

## NLI – nelineárny index

Tento index vychádza z predpokladu, že vzťah medzi mnohými vegetačnými indexmi a biofyzikálnymi parametrami povrchu je nelineárny. Linearizuje vzťahy s parametrami povrchu, ktoré majú tendenciu byť nelineárne.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referencia: Goel, N. a W. Qin. „Vplyv architektúry korún stromov na vzťahy medzi rôznymi vegetačnými indexmi a LAI a Fpar: počítačová simulácia.“ Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – Optimalizovaný index vegetácie upravený o pôdu

Tento index vychádza z indexu vegetácie upraveného o pôdu (SAVI). Používa štandardnú hodnotu 0,16 pre korekčný faktor pozadia korunového zápoja. Rondeaux (1996) zistil, že táto hodnota poskytuje väčšiu variabilitu pôdy ako SAVI pri nízkom vegetačnom pokryve, pričom vykazuje zvýšenú citlivosť na vegetačný pokryv väčší ako 50 %. Tento index sa najlepšie používa v oblastiach s relatívne riedkou vegetáciou, kde je pôda viditeľná cez koruny stromov.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referencia: Rondeaux, G., M. Steven a F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.“ Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – Renormalizovaný rozdielový vegetačný index

Tento index využíva rozdiel medzi vlnovými dĺžkami v blízkej infračervenej a červenej oblasti spolu s indexom NDVI na zvýraznenie zdravej vegetácie. Nie je citlivý na vplyvy pôdy a geometrie pozorovania Slnka.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referencia: Roujean, J. a F. Breon. „Odhad absorpcie PAR vegetáciou na základe meraní obojsmernej odrazivosti.“ Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – Index vegetácie upravený o pôdu

Tento index je podobný indexu NDVI, potláča však vplyvy pixelov pôdy. Využíva faktor úpravy pozadia korunového zápoja, _L_, ktorý je funkciou hustoty vegetácie a často si vyžaduje predchádzajúcu znalosť množstva vegetácie. Huete (1988) navrhuje optimálnu hodnotu _L_ = 0,5 na zohľadnenie variácií pozadia pôdy prvého rádu. Tento index sa najlepšie používa v oblastiach s relatívne riedkou vegetáciou, kde je pôda viditeľná cez koruny stromov.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referencia: Huete, A. „Index vegetácie upravený o pôdu (SAVI).“ Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – transformovaný diferenciálny vegetačný index

Tento index je užitočný na monitorovanie vegetačného pokrytia v mestskom prostredí. Na rozdiel od indexov NDVI a SAVI nedochádza k jeho nasýteniu.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referencia: Bannari, A., H. Asalhi a P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping“ V zborníku z konferencie Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, zväzok 5 (2002)._

***

## VARI – Index viditeľného spektra odolný voči atmosférickým vplyvom

Tento index vychádza z indexu ARVI a používa sa na odhad podielu vegetácie v scéne s nízkou citlivosťou na atmosférické vplyvy.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referencia: Gitelson, A., et al. „Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction.“ International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Index vegetácie so širokým dynamickým rozsahom

Tento index je podobný indexu NDVI, ale používa váhový koeficient (_a_) na zníženie rozdielu medzi príspevkami signálov v blízkej infračervenej a červenej oblasti k indexu NDVI. Index WDRVI je obzvlášť účinný v scénach so strednouaž vysokú hustotu vegetácie, keď hodnota NDVI presahuje 0,6. Hodnota NDVI má tendenciu sa stabilizovať pri náraste podielu vegetácie a indexu listovej plochy (LAI) stúpajú, zatiaľ čo WDRVI je citlivejší na širší rozsah podielov vegetácie a na zmeny v LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Váhový koeficient (_a_) sa môže pohybovať v rozmedzí od 0,1 do 0,2. Hodnota 0,2 je odporúčaná Henebrym, Viñom a Gitelsonom (2004).

_Referencie_

_Gitelson, A. „Index vegetácie so širokým dynamickým rozsahom pre diaľkové kvantifikovanie biofyzikálnych charakteristík vegetácie.“ Journal of Plant Physiology 161, č. 2 (2004): 165–173._

_Henebry, G., A. Viña a A. Gitelson. „Vegetačný index so širokým dynamickým rozsahom a jeho potenciálny prínos pre analýzu medzier.“ Gap Analysis Bulletin 12: 50–56._

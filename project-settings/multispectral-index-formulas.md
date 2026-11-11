---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Vzorce multispektrálnych indexov

Nižšie uvedené vzorce indexov využívajú kombináciu rozsahov priemerných priepustností filtrov Survey3:

<table><thead><tr><th align="center">Farba filtra Survey3</th><th width="196.199951171875" align="center">Názov filtra Survey3</th><th width="159.800048828125" align="center">Rozsah priepustnosti (FWHM)</th><th align="center">Priemerná priepustnosť</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re-RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Pri použití týchto vzorcov môže názov končiť na „\_1“ alebo „\_2“, čo zodpovedá tomu, ktorý NIR filter bol použitý – buď NIR1, alebo NIR2.

V prípade kamier LATTICE M3C (Bayer s trojitou pásmovou priepustnosťou) používa ten istý indexovací modul filtračné pásma M3C:

| Filter M3C | Pásmo 1 (stred/FWHM) | Pásmo 2 (stred/FWHM) | Pásmo 3 (stred/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orangeová 615 nm / 21 nm | Cyanová 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

Kamery LATTICE M3M sú jednopásmové (jeden úzkopásmový filter na kameru), preto sa pre samostatný obraz M3M nevypočítavajú viacpásmové indexy. Na výpočet indexov s M3M skombinujte dve alebo viac kamier do zarovnaného viacpásmového súboru a použite indexovací modul LATTICE (`chloros-cli lattice index` alebo nástroj Index Calculator v grafickom rozhraní).

***

## Kde funguje každý názov indexu

Systém Chloros má **tri** povrchy indexov a ich prednastavené zoznamy nie sú identické. V tejto časti si overte, či daný názov bude fungovať tam, kde ho plánujete použiť.

| Kde sa nachádzate | Ktorý zoznam platí | Počet |
| --- | --- | --- |
| Nastavenia projektu → Index → Pridať index (GUI) | Povrch 1 | 27 |
| Prehliadač obrázkov [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Povrch 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Povrch 2 | 22 |
| SDK `process_folder(indices=[...])` | Povrch 2 | 22 |
| `chloros-cli lattice index --preset` | Povrch 3 | 22 (iné číslo 22) |
| Kartu „Kamery“ – kalkulátor indexu v reálnom čase | Povrch 3 | 22 (iné číslo 22) |

Surface 1 a 2 pracujú s **jedným obrázkom naraz z jednej kamery**, pričom používajú symbolové sloty `x`/`y`/`z`(/`a`) viazané na filtračné kanály danej kamery. Povrch 3 pracuje s**zarovnaným viacpásmovým stohom** — niekoľkými kamerami LATTICE zaregistrovanými do jedného kockového súboru — a odkazuje na kanály pomocou názvov písaných malými písmenami.

### 1. Nastavenia projektu v grafickom rozhraní / roletové menu sandboxu prehliadača obrázkov — 27 vzorcov

Roletové menu ich uvádza v tomto poradí (ide o poradie vloženia, nie abecedné):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

V grafickom rozhraní pretiahnete filtračné kanály vašej kamery na pozície pásiem vo vzorci, takže akýkoľvek vzorec je možné použiť s akýmkoľvek priradením pásiem, ktoré vaša kamera podporuje. Vlastné vzorce, ktoré ste uložili, sú pridané pod tento zoznam.

**Päť vzorcov určených výlučne pre grafické rozhranie** — tie, ktoré zoznam CLI/SDK `--indices` neprijíma — sú implementované takto:

| Prednastavenie určené výlučne pre grafické rozhranie | Vzorec (ako je implementovaný) | Sloty |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (štyri polia) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Zamýšľané priradenie pre každý z nich je uvedené v samostatnej časti nižšie na tejto stránke (napríklad GARI očakáva x=Green, y=NIR, z=Blue, a=Red). GARI je jediný vzorec v Chloros, ktorý používa štvrtý slot.

### 2. CLI / SDK Rozšírenie názvov `--indices` — 22 predvolieb

Možnosť `chloros-cli process --indices` (a parameter SDK `indices`) akceptuje tieto názvy predvolieb:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Názvy neznámych indexov sa bez upozornenia preskočia.** Názov, ktorý nie je v tomto zozname (vrátane piatich vzorcov určených výlučne pre grafické rozhranie `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` a akýkoľvek vlastný vzorec, ktorý ste uložili v grafickom rozhraní) sa vynechá len s upozornením v protokole — spustenie pokračuje bez tohto indexu a samotné spustenie sa aj tak vyhodnotí ako úspešné. Upozornenie sa vypíše takto:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Názvy sa porovnávajú bez ohľadu na veľkosť písmen po odstránení medzier, takže `ndvi`, `NDVI` a ` NDVI ` sú rovnaké prednastavenia. Prednastavenie sa tiež preskočí, ak vyžaduje pásmo, ktoré filter vašej kamery neposkytuje.
{% endhint %}

Presné vzorce tak, ako sú implementované (symboly `x`/`y`/`z` sú pozície pásiem; predvolené priradenie je uvedené pre každé prednastavenie):

| Prednastavenie | Vzorec (tak, ako je implementovaný) | Predvolený filter | Sloty (x, y, z) |
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
| CVI | `(z/y)*(x/y)` | RGN | Red, Green, NIR |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | NGB | NIR, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Ako sa z názvu predvoľby stanovia pozície pásiem

Keď zadáte holý názov, napríklad `NDVI`, program Chloros musí rozhodnúť, ktorý kanál ktorého súboru číta každý symbol. Používa túto tabuľku, ktorá priraďuje kód filtra k pozícii každého kanála v poli:

| Kód filtra | Kanál → index poľa |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` je akceptovaný ako alias pre Orange, tiež 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

**Predvolený filter** prednastavenia (stĺpec „Predvolený filter“ vyššie) sa použije, ak projekt obsahuje obrázky s týmto filtrom. Ak tomu tak nie je, program Chloros prehľadá filtre skutočne prítomné v projekte v poradí `RGN, OCN, NGB, RGB, RE, NIR` a vyberie prvý, ktorý dokáže poskytnúť všetky kanály potrebné pre prednastavenie. Ak žiadny z nich nie je schopný, prednastavenie sa pre dané spustenie vynechá. Preto `NDVI` požiadaný na súbore údajov obsahujúcom iba OCN stále produkuje zmysluplný výsledok — viaže sa na pozície Orange a NIR v OCN.

Reťazce modelu LATTICE M3C obsahujú filter s predponou `F` (`LATT-M3C-L41-FRGN`), avšak predpona sa vynechá, keď sa kód filtra načítava z obrazu, takže kamera FRGN rozlišuje prostredníctvom riadku `RGN` vyššie a nevyžaduje si žiadne špeciálne zaobchádzanie.

### 3. Indexovací modul LATTICE (`lattice index --preset`, kalkulátor indexu v reálnom čase) — 22 predvolieb

Modul LATTICE pracuje s vyrovnanými viacpásmovými sériami snímok (živé polia alebo exportované viacpásmové súbory TIFF) a používa názvy kanálov písané malými písmenami (`red`, `green`, `blue`, `red_edge`, `nir`). Jeho zoznam predvolieb sa líši od dvoch vyššie uvedených:

| Predvoľba | Vzorec | Kanály |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | červená, NIR |
| GNDVI | `(nir - green) / (nir + green)` | zelený, NIR |
| BNDVI | `(nir - blue) / (nir + blue)` | modrý, NIR |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | červený okraj, NIR |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | modrá, zelená, NIR |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | červená, NIR |
| OSAVI | `(nir - red) / (nir + red + 0.16)` | červená, NIR |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | červená, NIR |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | modrá, červená, NIR |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | červená, NIR |
| CVI | `nir * red / green**2` | červená, zelená, NIR |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | červená, NIR |
| TDVI | `1.5 * (nir - red) / sqrt(nir**2 + red + 0.5)` | červená, NIR |
| LAI | `3.618 * (2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)) - 0.118` | modrá, červená, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | červená, zelená, modrá |
| NGRDI | `(green - red) / (green + red)` | červená, zelená |
| VARI | `(green - red) / (green + red - blue)` | červená, zelená, modrá |
| TGI | `green - 0.39*red - 0.61*blue` | červená, zelená, modrá |
| EXG | `2*green - red - blue` | červená, zelená, modrá |
| CIRE | `(nir / red_edge) - 1` | červená\_okraj, nir |
| CIGREEN | `(nir / green) - 1` | zelená, nir |
| NDWI | `(green - nir) / (green + nir)` | zelená, nir |

Spustite `chloros-cli lattice index --list-presets` na vytlačenie tejto tabuľky z vašej nainštalovanej verzie a `--list-gradients` pre dostupné farebné prechody. Symboly kanálov rozlišujú veľké a malé písmená a musia zodpovedať názvom predvolieb písaným malými písmenami (napr. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI – index vegetácie na základe chlorofylu

Tento index je citlivý na obsah chlorofylu v listoch v širokom rozsahu hustôt korún stromov. Využíva pásma NIR, červené a zelené.

$$
CVI = NIR * {Red \over Green^{2}}
$$

V grafickom rozhraní (GUI) a v zozname predvolieb CLI/SDK je to symbolický vzorec `(z/y)*(x/y)`, kde x = Red, y = Green, z = NIR (pásma kamery typu RGN); prednastavenie indexového modulu LATTICE je `nir * red / green**2`. Každá plocha vypočíta rovnaký index.

_Referencia: Vincini, M., E. Frazzi a P. D&#x27;Alessio. „Širokopásmový index vegetácie založený na chlorofyle listov v mierke korún stromov.“ Precision Agriculture 9 (2008): 303–319._

***

## ENDVI – Vylepšený normalizovaný rozdielový vegetačný index

Tento index okrem NIR a zeleného kanála využíva aj modrý kanál a je obľúbený pri kamerách s NGB filtrom, kde modré spektrálne pásmo nahrádza červené.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Implementácia prebieha pomocou symbolického vzorca `((x+y)-(2*z))/((x+y)+(2*z))` — priraďte kanály NIR a Green vašej kamery do pozícií x/y a Blue do pozície z (pre kameru NGB: x=NIR, y=Green, z=Blue).

***

## EVI – vylepšený vegetačný index

Tento index bol pôvodne vyvinutý na použitie s údajmi MODIS ako vylepšenie indexu NDVI prostredníctvom optimalizácie signálu vegetácie v oblastiach s vysokým indexom listovej plochy (LAI). Je najužitočnejší v regiónoch s vysokým LAI, kde môže dôjsť k nasýteniu NDVI. Využíva oblasť modrej odrazivosti na korekciu signálov pôdneho pozadia a na zníženie vplyvov atmosféry, vrátane rozptylu aerosólov.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Hodnoty EVI by sa mali pohybovať v rozmedzí od 0 do 1 pre pixely vegetácie. Svetlé prvky, ako sú mraky a biele budovy, spolu s tmavými prvkami, ako je voda, môžu viesť k anomálnym hodnotám pixelov v EVI snímke. Pred vytvorením EVI snímky by ste mali z obrazu odrazivosti odstrániť mraky a svetlé prvky a prípadne nastaviť prahové hodnoty pixelov v rozmedzí od 0 do 1.

_Referencia: Huete, A., et al. „Prehľad radiometrického a biofyzikálneho výkonu vegetačných indexov MODIS.“ Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Index lesného porastu 1

_Iba v grafickom rozhraní (GUI) — nie je k dispozícii ako prednastavenie CLI/SDK `--indices`._

Tento index rozlišuje lesný porast od iných typov vegetácie pomocou multispektrálnych snímok odrazivosti, ktoré obsahujú pásmo červeného okraja.

$$
FCI1 = Red * RedEdge
$$

Lesnaté oblasti budú mať nižšie hodnoty FCI1 kvôli nižšej odrazivosti stromov a prítomnosti tieňov v korunách stromov.

_Referencia: Becker, Sarah J., Craig S.T. Daughtry a Andrew L. Russ. „Robustné indexy lesného porastu pre multispektrálne snímky.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Index lesného porastu 2

_Iba v grafickom rozhraní (GUI) — nie je k dispozícii ako prednastavenie CLI/SDK `--indices`._

Tento index rozlišuje lesné koruny od iných typov vegetácie pomocou multispektrálnych snímok odrazivosti, ktoré neobsahujú pásmo červeného okraja.

$$
FCI2 = Red * NIR
$$

Lesnaté oblasti budú mať nižšie hodnoty FCI2 kvôli nižšej odrazivosti stromov a prítomnosti tieňov v korunách stromov.

_Referencia: Becker, Sarah J., Craig S.T. Daughtry a Andrew L. Russ. „Robustné indexy lesného porastu pre multispektrálne snímky.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – Globálny index monitorovania životného prostredia

_Iba GUI – nie je k dispozícii ako prednastavenie CLI/SDK `--indices`._

Tento nelineárny index vegetácie sa používa na globálne monitorovanie životného prostredia na základe satelitných snímok a snaží sa korigovať vplyvy atmosféry. Je podobný indexu NDVI, je však menej citlivý na atmosférické vplyvy. Ovplyvňuje ho holá pôda; preto sa neodporúča používať v oblastiach s riedkou alebo stredne hustou vegetáciou.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Kde:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referencia: Pinty, B. a M. Verstraete. „GEMI: nelineárny index na monitorovanie globálnej vegetácie zo satelitov“. Vegetation 101 (1992): 15–20._

***

## GARI – Green Atmospherically Resistant Index (zelený index odolný voči atmosférickým vplyvom)

_Iba GUI – nie je k dispozícii ako prednastavenie CLI/SDK `--indices`._

Tento index je citlivejší na široký rozsah koncentrácií chlorofylu a menej citlivý na atmosférické vplyvy ako NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gama konštanta je váhová funkcia, ktorá závisí od podmienok aerosólov v atmosfére. ENVI používa hodnotu 1,7, čo je odporúčaná hodnota podľa Gitelsona, Kaufmana a Merzylaka (1996, s. 296).

_Referencia: Gitelson, A., Y. Kaufman a M. Merzylak. „Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.“ Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – Green Chlorophyll Index (zelený chlorofylový index)

Tento index sa používa na odhad obsahu chlorofylu v listoch u širokého spektra rastlinných druhov.

$$
GCI = {NIR \over Green} - 1
$$

Široké spektrum vlnových dĺžok v oblasti NIR a zeleného spektra umožňuje lepšie odhadovať obsah chlorofylu a zároveň poskytuje vyššiu citlivosť a lepší pomer signálu k šumu.

_Referencia: Gitelson, A., Y. Gritz a M. Merzlyak. „Vzťahy medzi obsahom chlorofylu v listoch a spektrálnou odrazivosťou a algoritmy na nedestruktívne hodnotenie chlorofylu v listoch vyšších rastlín.“ Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – index Green Leaf Index

Tento index bol pôvodne navrhnutý na použitie s digitálnou RGB kamerou na meranie pokrytia pšenicou, pričom červené, zelené a modré digitálne hodnoty (DN) sa pohybujú v rozmedzí od 0 do 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Hodnoty GLI sa pohybujú v rozmedzí od -1 do +1. Záporné hodnoty predstavujú pôdu a neživé prvky, zatiaľ čo kladné hodnoty predstavujú zelené listy a stonky.

_Referencia: Louhaichi, M., M. Borman a D. Johnson. „Prostorovo lokalizovaná platforma a letecká fotografia na dokumentáciu vplyvov pasenia na pšenicu.“ Geocarto International 16, č. 1 (2001): 65–70._

***

## GNDVI – Greenný normalizovaný index vegetácie

Tento index je podobný indexu NDVI s tým rozdielom, že meria zelené spektrum v rozsahu 540 až 570 nm namiesto červeného spektra. Tento index je citlivejší na koncentráciu chlorofylu ako NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referencia: Gitelson, A., a M. Merzlyak. „Diaľkové snímanie koncentrácie chlorofylu v listoch vyšších rastlín.“ Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI –Green Optimized Soil Adjusted Vegetation Index (Zelený optimalizovaný index vegetácie upravený podľa pôdy)

Tento index bol pôvodne navrhnutý s využitím farebnej infračervenej fotografie na predpovedanie potrieb dusíka pre kukuricu. Je podobný indexu OSAVI, avšak namiesto červeného pásma používa zelené.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referencia: Sripada, R., et al. „Určovanie sezónnej potreby dusíka pre kukuricu pomocou leteckej farebnej infračervenej fotografie.“ Doktorandská práca, North Carolina State University, 2005._

***

## GRVI – Vegetčný index pomeru zelene (Green Ratio Vegetation Index)

Tento index citlivo reaguje na rýchlosť fotosyntézy v lesných porastoch, keďže odrazivosť zeleného a červeného spektra je silne ovplyvnená zmenami v listových pigmentoch.

$$
GRVI = {NIR \over Green }
$$

_Referencia: Sripada, R., et al. „Letecká farebno-infračervená fotografia na stanovenie počiatočnej potreby dusíka u kukurice počas vegetačného obdobia.“ Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – index vegetácie upravený o zelenú pôdu (Green Soil Adjusted Vegetation Index)

Tento index bol pôvodne navrhnutý s využitím farebnej infračervenej fotografie na predpovedanie potrieb dusíka u kukurice. Je podobný indexu SAVI, avšak namiesto červeného pásma používa zelené.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referencia: Sripada, R., et al. „Určovanie potrieb dusíka u kukurice počas vegetačného obdobia pomocou leteckej farebnej infračervenej fotografie.“ Doktorandská práca, North Carolina State University, 2005._

***

## LAI – index listovej plochy

Tento index sa používa na odhad pokrytia listov a na prognózovanie rastu a výnosu plodín. Program ENVI vypočítava zelený LAI pomocou nasledujúceho empirického vzorca od Boegha a kol. (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Kde EVI je:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Vysoké hodnoty LAI sa zvyčajne pohybujú v rozmedzí približne od 0 do 3,5. Ak však scéna obsahuje mraky a iné jasné prvky, ktoré spôsobujú nasýtenie pixelov, hodnoty LAI môžu prekročiť 3,5. Pred vytvorením snímky LAI by ste mali v ideálnom prípade z vašej scény odstrániť mraky a jasné prvky.

_Referencia: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde a A. Thomsen. „Letecké multispektrálne údaje na kvantifikáciu indexu listovej plochy, koncentrácie dusíka a fotosyntetickej účinnosti v poľnohospodárstve.“ Remote Sensing of Environment 81, č. 2–3 (2002): 179–193._

***

## LCI – index chlorofylu v listoch

_Iba v grafickom rozhraní (GUI) – nie je k dispozícii ako prednastavenie CLI/SDK `--indices`._

Tento index sa používa na odhad obsahu chlorofylu vo vyšších rastlinách a je citlivý na zmeny odrazivosti spôsobené absorpciou chlorofylu.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referencia: Datt, B. „Diaľkové snímanie obsahu vody v listoch eukalyptu.“ Journal of Plant Physiology 154, č. 1 (1999): 30–36._

***

## MNLI – Modifikovaný nelineárny index

Tento index je vylepšením nelineárneho indexu (NLI), ktorý zahŕňa index vegetácie upravený o pôdu (SAVI) s cieľom zohľadniť pôdné pozadie. Program ENVI používa hodnotu faktora úpravy pozadia korunového zápoja (_L_) rovnú 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referencia: Yang, Z., P. Willis a R. Mueller. „Vplyv snímky AWIFS vylepšenej pomerom pásiem na presnosť klasifikácie plodín.“ Zborník z sympózia Pecora 17 o diaľkovom prieskume (2008), Denver, CO._

***

## MSAVI2 – Modifikovaný index vegetácie upravený o pôdu 2

Tento index je zjednodušenou verziou indexu MSAVI, ktorý navrhli Qi a kol. (1994), ktorý vylepšuje index vegetácie upravený o pôdu (SAVI). Znižuje pôdný šum a zvyšuje dynamický rozsah signálu vegetácie. MSAVI2 je založený na induktívnej metóde, ktorá na zvýraznenie zdravej vegetácie nepoužíva konštantnú hodnotu _L_ (ako v prípade SAVI).

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referencia: Qi, J., A. Chehbouni, A. Huete, Y. Kerr a S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.“ Remote Sensing of Environment 48 (1994): 119–126._

***

## MSR – Modifikovaný jednoduchý pomer

Tento index je modifikáciou jednoduchého pomeru NIR/Red, navrhnutou s cieľom linearizovať jeho vzťah s biofyzikálnymi parametrami, a je citlivejší ako NDVI pri vyšších hustotách vegetácie.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Referencia: Chen, J. „Hodnotenie vegetačných indexov a modifikovaného jednoduchého pomeru pre boreálne aplikácie.“ Canadian Journal of Remote Sensing 22 (1996): 229–242._

***

## NDRE – Normalizovaný rozdiel RedEdge

Tento index je podobný indexu NDVI, avšak porovnáva kontrast medzi NIR a RedEdge namiesto Red, čím často skôr detekuje stres vegetácie.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalizovaný rozdielový vegetačný index

Tento index je meradlom zdravej, zelenej vegetácie. Kombinácia jeho normalizovaného rozdielu a využitia oblastí s najvyššou absorpciou a odrazivosťou chlorofylu z neho robí robustný index v širokom spektre podmienok. Môže sa však v podmienkach hustej vegetácie, keď je hodnota LAI vysoká, dôjsť k jeho nasýteniu.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Hodnota tohto indexu sa pohybuje v rozmedzí od -1 do 1. Bežný rozsah pre zelenú vegetáciu je 0,2 až 0,8.

_Referencia: Rouse, J., R. Haas, J. Schell a D. Deering. Monitorovanie vegetačných systémov na Veľkých planinách pomocou ERTS. Tretie sympózium ERTS, NASA (1973): 309–317._

***

## NLI – nelineárny index

Tento index vychádza z predpokladu, že vzťah medzi mnohými vegetačnými indexmi a biofyzikálnymi parametrami povrchu je nelineárny. Linearizuje vzťahy s parametrami povrchu, ktoré majú tendenciu byť nelineárne.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referencia: Goel, N. a W. Qin. „Vplyv architektúry korún stromov na vzťahy medzi rôznymi vegetačnými indexmi a LAI a Fpar: počítačová simulácia.“ Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – optimalizovaný index vegetácie upravený o pôdu

Tento index vychádza z indexu vegetácie upraveného o pôdu (SAVI). Používa štandardnú hodnotu 0,16 pre korekčný faktor korún stromov. Rondeaux (1996) zistil, že táto hodnota poskytuje väčšiu variabilitu pôdy ako SAVI pri nízkom vegetačnom pokryve, pričom vykazuje zvýšenú citlivosť pri vegetačnom pokryve vyššom ako 50 %. Tento index sa najlepšie používa v oblastiach s relatívne riedkou vegetáciou, kde je pôda viditeľná cez koruny stromov.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referencia: Rondeaux, G., M. Steven a F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.“ Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – renormalizovaný diferenciálny vegetačný index

Tento index využíva rozdiel medzi vlnovými dĺžkami v blízkej infračervenej a červenej oblasti spolu s indexom NDVI na zvýraznenie zdravej vegetácie. Je necitlivý na vplyvy pôdy a geometrie pozorovania Slnka.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referencia: Roujean, J. a F. Breon. „Odhad PAR absorbovaného vegetáciou na základe meraní obojsmernej odrazivosti.“ Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – index vegetácie upravený o pôdu

Tento index je podobný indexu NDVI, potláča však vplyvy pixelov pôdy. Využíva korekčný faktor pozadia korunového zápoja, _L_, ktorý je funkciou hustoty vegetácie a často si vyžaduje predchádzajúcu znalosť množstva vegetácie. Huete (1988) navrhuje optimálnu hodnotu _L_ = 0,5 na zohľadnenie variácií pozadia pôdy prvého rádu. Tento index sa najlepšie používa v oblastiach s relatívne riedkou vegetáciou, kde je pôda viditeľná cez koruny stromov.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referencia: Huete, A. „A Soil-Adjusted Vegetation Index (SAVI).“ Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – Transformovaný index rozdielov vegetácie

Tento index je užitočný na monitorovanie vegetačného pokrytia v mestskom prostredí. Na rozdiel od indexov NDVI a SAVI nedochádza k jeho nasýteniu.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referencia: Bannari, A., H. Asalhi a P. Teillet. „Transformovaný index rozdielu vegetácie (TDVI) pre mapovanie vegetačného pokrytia“ V zborníku z konferencie Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, zväzok 5 (2002)._

***

## VARI – index odolný voči atmosférickým vplyvom v viditeľnom spektre

Tento index vychádza z indexu ARVI a používa sa na odhad podielu vegetácie v scéne s nízkou citlivosťou na atmosférické vplyvy.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referencia: Gitelson, A., a kol. „Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction.“ International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – index vegetácie so širokým dynamickým rozsahom

Tento index je podobný indexu NDVI, avšak využíva váhový koeficient (_a_) na zníženie rozdielu medzi príspevkami signálov v blízkej infračervenej a červenej oblasti k indexu NDVI. Index WDRVI je obzvlášť účinný v scénach so strednou až vysokou hustotou vegetácie, keď hodnota NDVI presahuje 0,6. NDVI má tendenciu sa stabilizovať pri zvyšovaní podielu vegetácie a indexu listovej plochy (LAI), zatiaľ čo WDRVI je citlivejší na širší rozsah podielov vegetácie a na zmeny LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Váha koeficientu (_a_) sa môže pohybovať v rozmedzí od 0,1 do 0,2. Hodnota 0,2 je odporúčaná Henebrym, Viñom a Gitelsonom (2004).

_Referencie_

_Gitelson, A. „Index vegetácie so širokým dynamickým rozsahom pre diaľkové kvantifikovanie biofyzikálnych charakteristík vegetácie.“ Journal of Plant Physiology 161, č. 2 (2004): 165–173._

_Henebry, G., A. Viña a A. Gitelson. „Vegetačný index so širokým dynamickým rozsahom a jeho potenciálne využitie pre analýzu medzier.“ Gap Analysis Bulletin 12: 50–56._

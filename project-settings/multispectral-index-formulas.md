---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Vzorce multispektrálnych indexov

Nižšie uvedené vzorce indexov využívajú kombináciu rozsahov priemernej priepustnosti filtra Survey3:

<table><thead><tr><th align="center">Survey3 Farba filtra</th><th width="196.199951171875" align="center">Survey3 Názov filtra</th><th width="159.800048828125" align="center">Rozsah priepustnosti (FWHM)</th><th align="center">Priemerná priepustnosť</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN – Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN – Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN – Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR – NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>Pri použití týchto vzorcov môže názov končiť na „\_1“ alebo „\_2“, čo zodpovedá tomu, ktorý filter NIR, buď NIR1 alebo NIR2, bol použitý.

***

## EVI – Vylepšený index vegetácie

Tento index bol pôvodne vyvinutý na použitie s dátami MODIS ako vylepšenie indexu NDVI optimalizáciou signálu vegetácie v oblastiach s vysokým indexom listovej plochy (LAI). Je najužitočnejší v oblastiach s vysokým indexom LAI, kde môže dôjsť k nasýteniu indexu NDVI. Využíva oblasť modrej odrazivosti na korekciu signálov pôdneho pozadia a na zníženie vplyvov atmosféry, vrátane rozptylu aerosólov.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Hodnoty EVI by sa mali pohybovať v rozmedzí od 0 do 1 pre pixely vegetácie. Svetlé prvky, ako sú mraky a biele budovy, spolu s tmavými prvkami, ako je voda, môžu viesť k anomálnym hodnotám pixelov v obraze EVI. Pred vytvorením obrazu EVI by ste mali z obrazu odrazivosti odstrániť mraky a svetlé prvky a voliteľne nastaviť prahové hodnoty pixelov od 0 do 1.

_Referencia: Huete, A., et al. „Prehľad radiometrického a biofyzikálneho výkonu indexov vegetácie MODIS.“ Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Index lesného porastu 1

Tento index rozlišuje lesný porast od iných typov vegetácie pomocou multispektrálnych snímok odrazivosti, ktoré obsahujú pásmo červenej hrany.

$$
FCI1 = Red * RedEdge
$$

Lesnaté oblasti budú mať nižšie hodnoty FCI1 kvôli nižšej odrazivosti stromov a prítomnosti tieňov v korunách stromov.

_Referencia: Becker, Sarah J., Craig S.T. Daughtry a Andrew L. Russ. „Robustné indexy lesného porastu pre multispektrálne snímky.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Index lesného porastu 2

Tento index rozlišuje lesné koruny od iných typov vegetácie pomocou multispektrálnych snímok odrazivosti, ktoré neobsahujú červené okrajové pásmo.

$$
FCI2 = Red * NIR
$$

Lesnaté oblasti budú mať nižšie hodnoty FCI2 kvôli nižšej odrazivosti stromov a prítomnosti tieňov v korunách stromov.

_Referencia: Becker, Sarah J., Craig S.T. Daughtry a Andrew L. Russ. „Robustné indexy lesného porastu pre multispektrálne snímky.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI – Index globálneho monitorovania životného prostredia

Tento nelineárny index vegetácie sa používa na globálne monitorovanie životného prostredia zo satelitných snímok a snaží sa korigovať atmosférické vplyvy. Je podobný indexu NDVI, ale je menej citlivý na atmosférické vplyvy. Ovplyvňuje ho holá pôda, preto sa neodporúča používať v oblastiach s riedkou alebo stredne hustou vegetáciou.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Kde:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referencia: Pinty, B., a M. Verstraete. GEMI: nelineárny index na monitorovanie globálnej vegetácie zo satelitov. Vegetation 101 (1992): 15-20._

***

## GARI – Green Index odolný voči atmosférickým vplyvom

Tento index je citlivejší na široký rozsah koncentrácií chlorofylu a menej citlivý na atmosférické vplyvy ako NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Konštanta gama je váhová funkcia, ktorá závisí od podmienok aerosólov v atmosfére. ENVI používa hodnotu 1,7, čo je odporúčaná hodnota od Gitelsona, Kaufmana a Merzylaka (1996, strana 296).

_Referencia: Gitelson, A., Y. Kaufman a M. Merzylak. „Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.“ Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI – Green Index chlorofylu

Tento index sa používa na odhad obsahu chlorofylu v listoch u širokého spektra rastlinných druhov.

$$
GCI = {NIR \over Green} - 1
$$

Široké spektrum vlnových dĺžok NIR a zelených vlnových dĺžok poskytuje lepšiu predpoveď obsahu chlorofylu a zároveň umožňuje vyššiu citlivosť a vyšší pomer signálu k šumu.

_Referencia: Gitelson, A., Y. Gritz a M. Merzlyak. „Vzťahy medzi obsahom chlorofylu v listoch a spektrálnou odrazivosťou a algoritmy pre nedestruktívne hodnotenie chlorofylu v listoch vyšších rastlín.“ Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green Index listov

Tento index bol pôvodne navrhnutý na použitie s digitálnou kamerou RGB na meranie pokrytia pšenicou, kde červené, zelené a modré digitálne čísla (DN) sa pohybujú v rozmedzí od 0 do 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Hodnoty GLI sa pohybujú v rozmedzí od -1 do +1. Záporné hodnoty predstavujú pôdu a neživé prvky, zatiaľ čo kladné hodnoty predstavujú zelené listy a stonky.

_Referencia: Louhaichi, M., M. Borman a D. Johnson. „Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.“ Geocarto International 16, č. 1 (2001): 65–70._

***

## GNDVI – Green Normalizovaný index vegetácie

Tento index je podobný indexu NDVI, s tým rozdielom, že meria zelené spektrum od 540 do 570 nm namiesto červeného spektra. Tento index je citlivejší na koncentráciu chlorofylu ako index NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referencia: Gitelson, A., a M. Merzlyak. „Diaľkové snímanie koncentrácie chlorofylu v listoch vyšších rastlín.“ Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Optimalizovaný index vegetácie upravený o pôdu

Tento index bol pôvodne navrhnutý s využitím farebnej infračervenej fotografie na predpovedanie potrieb dusíka pre kukuricu. Je podobný indexu OSAVI, ale nahrádza zelené pásmo červeným.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referencia: Sripada, R., et al. „Určenie sezónnej potreby dusíka pre kukuricu pomocou leteckej farebnej infračervenej fotografie.“ Doktorandská práca, North Carolina State University, 2005._

***

## GRVI – Green Pomerný index vegetácie

Tento index je citlivý na rýchlosť fotosyntézy v lesných porastoch, keďže odrazivosť zeleného a červeného spektra je silne ovplyvnená zmenami v pigmentácii listov.

$$
GRVI = {NIR \over Green }
$$

_Referencia: Sripada, R., et al. „Letecká farebná infračervená fotografia na stanovenie počiatočných potrieb dusíka v kukurici počas vegetačného obdobia.“ Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Vegetation Index upravený o pôdu

Tento index bol pôvodne navrhnutý s využitím farebnej infračervenej fotografie na predpovedanie potrieb dusíka pre kukuricu. Je podobný indexu SAVI, ale nahrádza červené pásmo zeleným.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referencia: Sripada, R., et al. „Určenie sezónnej potreby dusíka pre kukuricu pomocou leteckej farebnej infračervenej fotografie.“ Doktorandská práca, North Carolina State University, 2005._

***

## LAI – Index listovej plochy

Tento index sa používa na odhad pokrytia listov a na prognózu rastu a výnosu plodín. ENVI vypočíta zelený LAI pomocou nasledujúceho empirického vzorca od Boegha et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Kde EVI je:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Vysoké hodnoty LAI sa zvyčajne pohybujú v rozmedzí približne od 0 do 3,5. Ak však scéna obsahuje mraky a iné jasné prvky, ktoré vytvárajú nasýtené pixely, hodnoty LAI môžu prekročiť 3,5. Ideálne by ste mali pred vytvorením snímky LAI z vašej scény odstrániť mraky a jasné prvky.

_Referencia: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde a A. Thomsen. „Letecké multispektrálne údaje na kvantifikáciu indexu listovej plochy, koncentrácie dusíka a fotosyntetickej účinnosti v poľnohospodárstve.“ Remote Sensing of Environment 81, č. 2-3 (2002): 179-193._

***

## LCI – Index chlorofylu v listoch

Tento index sa používa na odhad obsahu chlorofylu vo vyšších rastlinách, ktoré sú citlivé na zmeny odrazivosti spôsobené absorpciou chlorofylu.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referencia: Datt, B. „Diaľkové snímanie obsahu vody v listoch eukalyptu.“ Journal of Plant Physiology 154, č. 1 (1999): 30–36._

***

## MNLI – Modifikovaný nelineárny index

Tento index je vylepšením nelineárneho indexu (NLI), ktorý zahŕňa index vegetácie upravený o pôdu (SAVI) s cieľom zohľadniť pôdne pozadie. ENVI používa hodnotu faktora úpravy pozadia korún stromov (_L_) 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referencia: Yang, Z., P. Willis a R. Mueller. „Vplyv snímky AWIFS vylepšenej pomerom pásiem na presnosť klasifikácie plodín.“ Zborník z sympózia Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – Modifikovaný index vegetácie upravený o pôdu 2

Tento index je zjednodušenou verziou indexu MSAVI navrhnutého Qi et al. (1994), ktorý vylepšuje index vegetácie upravený o pôdu (SAVI). Znižuje šum pôdy a zvyšuje dynamický rozsah signálu vegetácie. MSAVI2 je založený na induktívnej metóde, ktorá nepoužíva konštantnú hodnotu _L_ (ako v prípade SAVI) na zvýraznenie zdravej vegetácie.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referencia: Qi, J., A. Chehbouni, A. Huete, Y. Kerr a S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.“ Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE – Normalizovaný rozdiel RedEdge

Tento index je podobný indexu NDVI, ale porovnáva kontrast medzi NIR a RedEdge namiesto Red, čo často umožňuje skôr zistiť stres vegetácie.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalizovaný index vegetácie

Tento index je meradlom zdravej, zelenej vegetácie. Kombinácia jeho normalizovanej formulácie a využitia oblastí s najvyššou absorpciou a odrazivosťou chlorofylu ho robí robustným v širokom spektre podmienok. Môže sa však nasýtiť v podmienkach hustého porastu, keď sa hodnota LAI stane vysokou.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Hodnota tohto indexu sa pohybuje v rozmedzí od -1 do 1. Bežný rozsah pre zelenú vegetáciu je 0,2 až 0,8.

_Referencia: Rouse, J., R. Haas, J. Schell a D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Third ERTS Symposium, NASA (1973): 309-317._

***

## NLI – nelineárny index

Tento index vychádza z predpokladu, že vzťah medzi mnohými vegetačnými indexmi a povrchovými biofyzikálnymi parametrami je nelineárny. Linearizuje vzťahy s povrchovými parametrami, ktoré majú tendenciu byť nelineárne.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referencia: Goel, N. a W. Qin. „Vplyv architektúry korún stromov na vzťahy medzi rôznymi vegetačnými indexmi a LAI a Fpar: počítačová simulácia.“ Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – Optimalizovaný index vegetácie upravený o pôdu

Tento index je založený na indexe vegetácie upravenom o pôdu (SAVI). Používa štandardnú hodnotu 0,16 pre faktor úpravy pozadia korunového zápoja. Rondeaux (1996) zistil, že táto hodnota poskytuje väčšiu variabilitu pôdy ako SAVI pri nízkom vegetačnom pokryve, pričom vykazuje zvýšenú citlivosť na vegetačný pokryv väčší ako 50 %. Tento index sa najlepšie používa v oblastiach s relatívne riedkou vegetáciou, kde je pôda viditeľná cez koruny stromov.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referencia: Rondeaux, G., M. Steven a F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.“ Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI – Renormalizovaný index vegetácie

Tento index využíva rozdiel medzi vlnovými dĺžkami v blízkej infračervenej a červenej oblasti spolu s indexom NDVI na zvýraznenie zdravej vegetácie. Je necitlivý na vplyvy pôdy a geometrie pozorovania slnka.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referencia: Roujean, J., a F. Breon. „Odhad absorpcie PAR vegetáciou na základe meraní obojsmernej odrazivosti.“ Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – Index vegetácie upravený o pôdu

Tento index je podobný indexu NDVI, ale potláča vplyv pixelov pôdy. Používa faktor úpravy pozadia korún stromov, _L_, ktorý je funkciou hustoty vegetácie a často vyžaduje predchádzajúcu znalosť množstva vegetácie. Huete (1988) navrhuje optimálnu hodnotu _L_=0,5 na zohľadnenie variácií pozadia pôdy prvého rádu. Tento index sa najlepšie používa v oblastiach s relatívne riedkou vegetáciou, kde je pôda viditeľná cez koruny stromov.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referencia: Huete, A. „A Soil-Adjusted Vegetation Index (SAVI).“ Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – transformovaný index rozdielu vegetácie

Tento index je užitočný na monitorovanie vegetačného pokrytia v mestských prostrediach. Na rozdiel od indexov NDVI a SAVI nedochádza k jeho nasýteniu.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referencia: Bannari, A., H. Asalhi a P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping“ In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI – Index viditeľného spektra odolný voči atmosférickým vplyvom

Tento index vychádza z indexu ARVI a používa sa na odhad podielu vegetácie v scéne s nízkou citlivosťou na atmosférické vplyvy.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referencia: Gitelson, A., et al. „Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction.“ International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Index vegetácie so širokým dynamickým rozsahom

Tento index je podobný indexu NDVI, ale používa váhový koeficient (_a_) na zníženie rozdielu medzi príspevkami signálov v blízkej infračervenej a červenej oblasti k indexu NDVI. WDRVI je obzvlášť účinný v scénach so strednou až vysokou hustotou vegetácie, keď NDVI presahuje hodnotu 0,6. NDVI má tendenciu sa stabilizovať, keď sa zvyšuje podiel vegetácie a index listovej plochy (LAI), zatiaľ čo WDRVI je citlivejší na širší rozsah podielov vegetácie a na zmeny v LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Váha koeficientu (_a_) sa môže pohybovať v rozmedzí od 0,1 do 0,2. Hodnota 0,2 je odporúčaná Henebrym, Viñom a Gitelsonom (2004).

_Referencie_

_Gitelson, A. „Index vegetácie s širokým dynamickým rozsahom pre diaľkové kvantifikovanie biofyzikálnych charakteristík vegetácie.“ Journal of Plant Physiology 161, č. 2 (2004): 165–173._

_Henebry, G., A. Viña a A. Gitelson. „Index vegetácie s širokým dynamickým rozsahom a jeho potenciálna využiteľnosť pre analýzu medzier.“ Gap Analysis Bulletin 12: 50–56._

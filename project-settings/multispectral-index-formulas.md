---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Vzorec multispektrálneho indexu

Nižšie uvedené vzorce indexu používajú kombináciu priemerných rozsahov priepustnosti filtrov Survey3:

<table><thead><tr><th align="center">Survey3 Farba filtra</th><th width="196.199951171875" align="center">Survey3 Názov filtra</th><th width="159.800048828125" align="center">Rozsah priepustnosti (FWHM)</th><th align="center">Priemerná priepustnosť</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Pri použití týchto vzorcov môže názov končiť na „\_1“ alebo „\_2“, čo zodpovedá tomu, ktorý filter NIR, buď NIR1 alebo NIR2, bol použitý.

***

## EVI – Vylepšený index vegetácie

Tento index bol pôvodne vyvinutý na použitie s údajmi MODIS ako vylepšenie oproti NDVI optimalizáciou signálu vegetácie v oblastiach s vysokým indexom listovej plochy (LAI). Je najužitočnejší v oblastiach s vysokým LAI, kde môže dôjsť k nasýteniu NDVI. Používa modrú odrazovú oblasť na korekciu signálov pozadia pôdy a na zníženie atmosférických vplyvov, vrátane rozptylu aerosólov.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Hodnoty EVI by sa mali pohybovať v rozmedzí od 0 do 1 pre pixely vegetácie. Svetlé prvky, ako sú mraky a biele budovy, spolu s tmavými prvkami, ako je voda, môžu spôsobiť anomálne hodnoty pixelov v obraze EVI. Pred vytvorením obrazu EVI by ste mali z obrazu odrazu zamaskovať mraky a svetlé prvky a voliteľne nastaviť prahovú hodnotu pixelov od 0 do 1.

_Referencia: Huete, A., et al. „Prehľad radiometrických a biofyzikálnych vlastností indexov vegetácie MODIS.“ Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Index lesného porastu 1

Tento index rozlišuje lesné porasty od iných typov vegetácie pomocou multispektrálnych obrazov odrazivosti, ktoré obsahujú červené okrajové pásmo.

$$
FCI1 = Red * RedEdge
$$

Lesnaté oblasti budú mať nižšie hodnoty FCI1 kvôli nižšej odrazivosti stromov a prítomnosti tieňov v korunách stromov.

_Referencia: Becker, Sarah J., Craig S.T. Daughtry a Andrew L. Russ. „Robustné indexy lesného porastu pre multispektrálne snímky.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Index lesného porastu 2

Tento index rozlišuje koruny stromov od iných typov vegetácie pomocou multispektrálnych obrazov odrazivosti, ktoré neobsahujú červené okrajové pásmo.

$$
FCI2 = Red * NIR
$$

Zalesnené oblasti budú mať nižšie hodnoty FCI2 kvôli nižšej odrazivosti stromov a prítomnosti tieňov v korunách stromov.

_Referencia: Becker, Sarah J., Craig S.T. Daughtry a Andrew L. Russ. „Robustné indexy lesného porastu pre multispektrálne snímky.“ Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI – Index globálneho monitorovania životného prostredia

Tento nelineárny index vegetácie sa používa na globálne monitorovanie životného prostredia zo satelitných snímok a snaží sa korigovať atmosférické vplyvy. Je podobný NDVI, ale je menej citlivý na atmosférické vplyvy. Ovplyvňuje ho holá pôda, preto sa neodporúča používať v oblastiach s riedkou alebo stredne hustou vegetáciou.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Kde:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referencia: Pinty, B., a M. Verstraete. GEMI: nelineárny index na monitorovanie globálnej vegetácie zo satelitov. Vegetation 101 (1992): 15-20._

***

## GARI - Green Index odolný voči atmosférickým vplyvom

Tento index je citlivejší na široký rozsah koncentrácií chlorofylu a menej citlivý na atmosférické vplyvy ako NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Konštanta gama je váha, ktorá závisí od podmienok aerosólu v atmosfére. ENVI používa hodnotu 1,7, čo je odporúčaná hodnota od Gitelsona, Kaufmana a Merzylaka (1996, strana 296).

_Referencia: Gitelson, A., Y. Kaufman a M. Merzylak. „Použitie kanála Green pri diaľkovom snímaní globálnej vegetácie z EOS-MODIS.“ Diaľkové snímanie životného prostredia 58 (1996): 289-298._

***

## GCI – Green Index chlorofylu

Tento index sa používa na odhad obsahu chlorofylu v listoch širokého spektra rastlinných druhov.

$$
GCI = {NIR \over Green} - 1
$$

Široké spektrum NIR a zelených vlnových dĺžok umožňuje lepšie odhadovať obsah chlorofylu a zároveň poskytuje vyššiu citlivosť a vyšší pomer signálu k šumu.

_Referencia: Gitelson, A., Y. Gritz a M. Merzlyak. „Vzťahy medzi obsahom chlorofylu v listoch a spektrálnou odrazivosťou a algoritmy pre nedestruktívne hodnotenie chlorofylu v listoch vyšších rastlín.“ Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI – Green Index listov

Tento index bol pôvodne navrhnutý na použitie s digitálnou kamerou RGB na meranie pokrytia pšenice, kde červené, zelené a modré digitálne čísla (DN) sa pohybujú v rozmedzí od 0 do 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Hodnoty GLI sa pohybujú v rozmedzí od -1 do +1. Záporné hodnoty predstavujú pôdu a neživé prvky, zatiaľ čo kladné hodnoty predstavujú zelené listy a stonky.

_Referencia: Louhaichi, M., M. Borman a D. Johnson. „Priestorovo lokalizovaná platforma a letecká fotografia na dokumentáciu vplyvu pasenia na pšenicu.“ Geocarto International 16, č. 1 (2001): 65-70._

***

## GNDVI - Green Normalizovaný index vegetácie

Tento index je podobný indexu NDVI, s tým rozdielom, že meria zelené spektrum od 540 do 570 nm namiesto červeného spektra. Tento index je citlivejší na koncentráciu chlorofylu ako index NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referencia: Gitelson, A., a M. Merzlyak. „Diaľkové snímanie koncentrácie chlorofylu v listoch vyšších rastlín.“ Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Optimalizovaný index vegetácie upravený podľa pôdy

Tento index bol pôvodne navrhnutý s použitím farebnej infračervenej fotografie na predpovedanie potrieb dusíka pre kukuricu. Je podobný OSAVI, ale nahrádza zelené pásmo červeným.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referencia: Sripada, R., et al. „Určenie sezónnej potreby dusíka pre kukuricu pomocou farebnej infračervenej leteckej fotografie.“ Doktorandská práca, North Carolina State University, 2005._

***

## GRVI – Green Pomerový index vegetácie

Tento index je citlivý na fotosyntetické rýchlosti v korunách lesov, pretože odrazivosť zelenej a červenej farby je silne ovplyvnená zmenami v pigmentoch listov.

$$
GRVI = {NIR \over Green }
$$

_Referencia: Sripada, R., et al. „Letecká farebná infračervená fotografia na určenie potrieb dusíka v kukurici na začiatku sezóny.“ Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Index vegetácie upravený podľa pôdy

Tento index bol pôvodne navrhnutý s použitím farebnej infračervenej fotografie na predpovedanie potrieb dusíka pre kukuricu. Je podobný indexu SAVI, ale nahrádza zelené pásmo červeným.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referencia: Sripada, R., et al. „Určenie sezónnych potrieb dusíka pre kukuricu pomocou farebnej infračervenej leteckej fotografie.“ Doktorandská práca, North Carolina State University, 2005._

***

## LAI – Index listovej plochy

Tento index sa používa na odhad pokrytia listov a na predpovedanie rastu a výnosu plodín. ENVI vypočíta zelený LAI pomocou nasledujúceho empirického vzorca od Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Kde EVI je:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Vysoké hodnoty LAI sa zvyčajne pohybujú v rozmedzí približne od 0 do 3,5. Ak však scéna obsahuje mraky a iné jasné prvky, ktoré vytvárajú nasýtené pixely, hodnoty LAI môžu prekročiť 3,5. Pred vytvorením obrazu LAI by ste mali ideálne zamaskovať mraky a jasné prvky zo scény.

_Referencia: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde a A. Thomsen. „Letecké multispektrálne údaje na kvantifikáciu indexu listovej plochy, koncentrácie dusíka a fotosyntetickej účinnosti v poľnohospodárstve.“ Remote Sensing of Environment 81, č. 2-3 (2002): 179-193._

***

## LCI – Index chlorofylu listov

Tento index sa používa na odhad obsahu chlorofylu vo vyšších rastlinách, ktoré sú citlivé na zmeny odrazivosti spôsobené absorpciou chlorofylu.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referencia: Datt, B. „Diaľkové snímanie obsahu vody v listoch eukalyptu.“ Journal of Plant Physiology 154, č. 1 (1999): 30-36._

***

## MNLI – modifikovaný nelineárny index

Tento index je vylepšením nelineárneho indexu (NLI), ktorý zahŕňa index vegetácie upravený podľa pôdy (SAVI), aby zohľadnil pôdne pozadie. ENVI používa hodnotu faktora úpravy pozadia korún stromov (_L_) 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referencia: Yang, Z., P. Willis a R. Mueller. „Vplyv obrazu AWIFS s vylepšeným pomerom pásiem na presnosť klasifikácie plodín.“ Zborník z sympózia Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – Modifikovaný index vegetácie upravený podľa pôdy 2

Tento index je jednoduchšou verziou indexu MSAVI navrhnutého Qi et al (1994), ktorý vylepšuje index vegetácie upravený podľa pôdy (SAVI). Znižuje šum pôdy a zvyšuje dynamický rozsah signálu vegetácie. MSAVI2 je založený na indukčnej metóde, ktorá nepoužíva konštantnú hodnotu _L_ (ako v prípade SAVI) na zvýraznenie zdravej vegetácie.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referencia: Qi, J., A. Chehbouni, A. Huete, Y. Kerr a S. Sorooshian. „Modifikovaný index vegetácie upravený podľa pôdy.“ Diaľkové snímanie životného prostredia 48 (1994): 119-126._

***

## NDRE – Normalizovaný rozdiel RedEdge

Tento index je podobný NDVI, ale porovnáva kontrast medzi NIR a RedEdge namiesto Red, ktorý často skôr detekuje stres vegetácie.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalizovaný index vegetácie

Tento index je meradlom zdravej, zelenej vegetácie. Kombinácia jeho normalizovanej diferenciálnej formulácie a využitia oblastí s najvyššou absorpciou a odrazivosťou chlorofylu ho robí robustným v širokom spektre podmienok. Môže sa však nasýtiť v podmienkach hustej vegetácie, keď LAI dosiahne vysokú hodnotu.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Hodnota tohto indexu sa pohybuje v rozmedzí od -1 do 1. Bežný rozsah pre zelenú vegetáciu je 0,2 až 0,8.

_Referencia: Rouse, J., R. Haas, J. Schell a D. Deering. Monitorovanie vegetačných systémov v Great Plains pomocou ERTS. Tretie sympózium ERTS, NASA (1973): 309-317._

***

## NLI – nelineárny index

Tento index predpokladá, že vzťah medzi mnohými vegetačnými indexmi a biofyzikálnymi parametrami povrchu je nelineárny. Lineárizuje vzťahy s parametrami povrchu, ktoré majú tendenciu byť nelineárne.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referencia: Goel, N. a W. Qin. „Vplyv architektúry korún stromov na vzťahy medzi rôznymi vegetačnými indexmi a LAI a Fpar: počítačová simulácia.“ Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI – Optimalizovaný index vegetácie upravený podľa pôdy

Tento index je založený na indexe vegetácie upravenom podľa pôdy (SAVI). Používa štandardnú hodnotu 0,16 pre faktor úpravy pozadia korún stromov. Rondeaux (1996) zistil, že táto hodnota poskytuje väčšiu variabilitu pôdy ako SAVI pre nízky vegetačný pokryv, pričom vykazuje zvýšenú citlivosť na vegetačný pokryv väčší ako 50 %. Tento index sa najlepšie používa v oblastiach s relatívne riedkou vegetáciou, kde je pôda viditeľná cez koruny stromov.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referencia: Rondeaux, G., M. Steven a F. Baret. „Optimalizácia indexov vegetácie upravených podľa pôdy.“ Diaľkové snímanie životného prostredia 55 (1996): 95-107._

***

## RDVI – Renormalizovaný index rozdielov vegetácie

Tento index využíva rozdiel medzi vlnovými dĺžkami blízkej infračervenej a červenej oblasti spolu s NDVI na zvýraznenie zdravej vegetácie. Je necitlivý na vplyvy pôdy a geometrie pozorovania slnka.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referencia: Roujean, J., a F. Breon. „Odhadovanie PAR absorbovaného vegetáciou z obojsmerných meraní odrazivosti.“ Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI – Index vegetácie upravený podľa pôdy

Tento index je podobný indexu NDVI, ale potláča vplyv pixelov pôdy. Používa faktor úpravy pozadia korún stromov, _L_, ktorý je funkciou hustoty vegetácie a často vyžaduje predchádzajúcu znalosť množstva vegetácie. Huete (1988) navrhuje optimálnu hodnotu _L_=0,5, aby sa zohľadnili variácie pozadia pôdy prvého rádu. Tento index sa najlepšie používa v oblastiach s relatívne riedkou vegetáciou, kde je pôda viditeľná cez koruny stromov.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referencia: Huete, A. „Index vegetácie upravený podľa pôdy (SAVI).“ Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – transformovaný index rozdielov vegetácie

Tento index je užitočný na monitorovanie vegetačného pokrytia v mestských prostrediach. Nenasýti sa tak ako NDVI a SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referencia: Bannari, A., H. Asalhi a P. Teillet. „Transformovaný index rozdielov vegetácie (TDVI) pre mapovanie vegetačného pokrytia“ V zborníku Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, zväzok 5 (2002)._

***

## VARI – Index odolnosti voči atmosférickým vplyvom

Tento index je založený na ARVI a používa sa na odhad podielu vegetácie v scéne s nízkou citlivosťou na atmosférické vplyvy.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referencia: Gitelson, A., et al. „Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Index vegetácie s širokým dynamickým rozsahom

Tento index je podobný indexu NDVI, ale používa váhový koeficient (_a_) na zníženie rozdielov medzi príspevkami signálov v blízkej infračervenej a červenej oblasti k indexu NDVI. WDRVI je obzvlášť účinný v scénach s miernou až vysokou hustotou vegetácie, keď NDVI presahuje 0,6. NDVI má tendenciu sa stabilizovať, keď sa zvyšuje podiel vegetácie a index listovej plochy (LAI), zatiaľ čo WDRVI je citlivejší na širší rozsah podielov vegetácie a na zmeny v LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Váha koeficientu (_a_) sa môže pohybovať v rozmedzí od 0,1 do 0,2. Hodnota 0,2 je odporúčaná Henebrym, Viñom a Gitelsonom (2004).

_Referencie_

_Gitelson, A. „Index vegetácie s širokým dynamickým rozsahom pre diaľkové kvantifikovanie biofyzikálnych charakteristík vegetácie.“ Journal of Plant Physiology 161, č. 2 (2004): 165-173._

_Henebry, G., A. Viña a A. Gitelson. „Index vegetácie s širokým dynamickým rozsahom a jeho potenciálna využiteľnosť pre analýzu medzier.“ Gap Analysis Bulletin 12: 50-56._

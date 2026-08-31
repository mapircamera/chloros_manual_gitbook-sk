---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Podporované kamery

Chloros spracováva snímky z dvoch radov kamier MAPIR na **všetkých platformách** (Windows, Linux amd64 a Linux arm64/Jetson):

* **Survey3** — kamery Survey3W (širokouhlé) a Survey3N (úzkouhlé). Vstup: `RAW+JPG`.
* **LATTICE**— multispektrálne kamerové moduly M3C a M3M. Vstup: záznamy z kamier `.tif`/`.tiff`. Kamery LATTICE je možné**ovládať v reálnom čase** aj z Chloros — prostredníctvom karty „Kamery“ v grafickom užívateľskom rozhraní (Windows) alebo z `chloros-cli lattice` / Python SDK (Windows a Linux) — vrátane synchronizovaných sústav viacerých kamier. Pozrite si [príručku LATTICE](lattice/).

Spracovateľský reťazec prijíma aj vstupné súbory typu `.dng`.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Výrobca</th><th width="250">Model kamery</th><th width="138">Model filtra</th><th width="187">Typ snímky</th></tr></thead><tbody><tr><td><strong>Výrobca</strong></td><td><strong>Model fotoaparátu</strong></td><td><strong>Model filtra</strong></td><td><strong>Typ snímky</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

Rad LATTICE je modulárny multispektrálny kamerový systém postavený na senzore Sony IMX265 s globálnou uzávierkou (3,1 MP, pixely 3,45 µm). Každá kamera ukladá svoju identitu ako reťazec modelu:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros ju zobrazuje s predponou `LATT-` (napríklad `LATT-M3M-L41-F550`) a reťazec modelu riadi všetko ďalej – profil snímača, rozloženie pásiem a kalibrácia sa riešia automaticky; nie je potrebné nič konfigurovať pre jednotlivé kamery. Číslo objektívu predstavuje **horizontálne zorné pole v stupňoch**: `L41` = úzke 41°, `L87` = široké 87°.

Existujú dve konfigurácie snímača:

| Konfigurácia | Snímač      | Typ filtra                           | Pásma na kameru                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Bayerova farebná | Trojité pásmové                       | 3 spektrálne pásma z jednej expozície                                 |
| **M3M**       | Čiernobiely  | Jeden úzkopásmový interferenčný filter | 1 kalibrované pásmo — kombináciou viacerých kamier M3M sa získavajú vegetačné indexy |

### Možnosti filtrov M3C (Bayer)

| Filter | Pásma (názov @ stredná vlnová dĺžka v nm / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 615/21 · Cyan 490/38 · NIR 808/14 |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

### Katalóg filtrov M3M (mono) — 23 SKU

Číslo F je označenie SKU; namerané pásmo (vyrazené na každom kalibrovanom vývoze) je sken filtra pre danú šaržu:

| SKU    | Stred (nm, namerané) | Hranice FWHM (nm) | Šírka (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nominálne)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nominálne)           | 867–893         | 26         |
| F905   | — (nominálne)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_„Hranice pásma sú merané ako hodnoty plnej šírky pri polovičnom maxime z filtrovaných skenov MAPIR pre každú šaržu — ide o tie isté hodnoty, ktoré Chloros vkladá do každého kalibrovaného exportu.“_ „— (nominálne)“ = zatiaľ nebol vykonaný sken šarže; pre tieto SKU je uvedený stred číslo SKU a šírka je údaj výrobcu.

† „Odrazivosť F988 je kalibrovaná pomocou odrazového panelu v scéne: pásmo leží mimo kalibrovaného rozsahu svetelného senzora DAQ, takže Chloros používa váš najnovší záznam z panelu a uchováva ho medzi jednotlivými meraniami panelu.“ Pozri [Kalibračné ciele](calibration-targets.md).

Informácie o ovládaní kamery v reálnom čase, poliach, nastavení siete a reťazci radiometrického spracovania nájdete v [príručke LATTICE](lattice/).

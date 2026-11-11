---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formáty výstupných obrázkov

Chloros exportuje spracované produkty v štyroch formátoch súborov. Formát vyberte v nastaveniach projektu (GUI), pomocou `--format` (CLI) alebo pomocou `export_format` (SDK). CLI a SDK akceptujú presne nižšie uvedené reťazce.

| Formátovací reťazec | Prípona | Typ pixelu | Rozsah pixelov | Poznámky |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(predvolené)* | `.tif` | číselná hodnota typu uint16 | 0 – 65535 | Odporúčané pre fotogrametriu / GIS. |
| `TIFF (8-bit)` | `.tif` | uint8 – číselná hodnota | 0 – 255 | Pre všetky 8-bitové záznamy, ktoré ponúkajú iba 8-bitové formáty: Chloros nezvyšuje bitovú hĺbku. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % odrazivosť. Niektoré aplikácie nedokážu čítať súbory TIFF s plávajúcou desatinnou čiarkou; súbory sú väčšie. |
| `PNG (8-bit)` | `.png` | uint8 digitálne číslo | 0 – 255 | Bezstratová kompresia, vhodná na prezeranie na webe a vizualizáciu. |
| `JPG (8-bit)` | `.jpg` | uint8 číselná hodnota | 0 – 255 | Kompresia so stratou, najmenšie súbory. |

## Umiestnenie výstupných súborov

Výstupné súbory sa ukladajú do projektového priečinka, zoskupené podľa kamery a následne podľa formátu súboru:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Zložka pre kameru je `LATT-<sensor>-<lens>-F<filter>` pre LATTICE a `<model>_<filter>` (napr. `Survey3N_RGN`) pre Survey3. **Každý exportovaný produkt si zachováva názov zdrojového súboru — produkt identifikuje zložka, nie prípona súboru.** Úplné pravidlá nájdete v časti [Kde sa ukladajú výstupy](reference/cli-reference.md) v referenčnej príručke CLI.

## Produkty LATTICE (úrovne zachytávania a exportu)

Jeden surový snímok LATTICE sa v jednom prechode rozdelí na všetky požadované produkty. Každý typ produktu má svoje vlastné prepínacie tlačidlá (zaškrtávacie políčka v grafickom rozhraní alebo v CLI: `--debayered` / `--preview` / `--radiance` / `--reflectance`, všetky sú štandardne zapnuté):

| Úroveň | Obsah | Typ údajov |
| --- | --- | --- |
| `raw` | Dáta Bayer priamo zo senzora (monochromatické kamery: jedno spektrálne pásmo). Spracovanie vždy začína od surových dát. | Tak, ako boli zachytené |
| `debayered` | Lineárna demosaika — 3-kanálová pre M3C, 1-kanálová v odtieňoch šedej pre M3M: odstránený tmavý posun z kalibračného balíka, následne demosaika zohľadňujúca hrany, bez korekcie linearity, vyrovnania poľa alebo oddelenia zložiek. Export `process` zapíše ten istý súbor, aký sa na tejto úrovni uloží pri snímaní, a to v každom režime spracovania. | Lineárna DN |
| `radiance` | Absolútna spektrálna žiarivosť z úplného radiometrického reťazca, v **W/m²/sr/nm**. Vždy sa zapisuje ako 32-bitový súbor TIFF (`tiff32/Radiance_Images/`), bez ohľadu na zvolený formát exportu. | float32 |
| `reflectance` | Odrazivosť ρ, kde **DN 32768 = ρ 1,0 (100 %)** s rezervou až do ρ 2,0. Pripravené pre Pix4D. | uint16 |
| `preview` | Render pripravený na zobrazenie: RGB = vyváženie bielej + gama; multispektrálne = rozloženie falošných farieb. | 8-bitové zobrazenie |

### Orientácia

Súbory `raw` a `debayered` vždy zostávajú v **pôvodnej** orientácii senzora; nastavenie otočenia/prevrátenia fotoaparátu sa uplatňuje iba na ostatné produkty (náhľad, radiancia, odrazivosť, index). Pri snímaní poľa zostávajú súbory všetkých členov v natívnej orientácii, pokiaľ nepožiadate o zmenu orientácie (`--orient` v CLI, `orient=True` v SDK). Každý súbor zaznamenáva orientáciu, ktorá patrí k jeho pixelom, v XMP — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — a `ChlorosOrientationApplied` udáva, či ju nesú jeho vlastné pixely (`false` v surových a debayeroch súboroch). Prehliadač obrázkov Chlorosu upravuje súbory vo formáte RAW a po odstránení bayera podľa tohto záznamu iba na účely zobrazenia; samotné súbory sa nikdy neprepisujú. Súbor bez tohto záznamu (vytvorený pred verziou 1.3.0 alebo iným nástrojom) sa zobrazuje tak, ako bol uložený.

## Masky obmedzenia

Produkty žiarivosti a odrazivosti multispektrálnej kamery M3C prechádzajú demixovaním v pásme 3×3. Ak demixovanie v niektorom pásme vygeneruje zápornú hodnotu, táto hodnota sa obmedzí na nulu (živé zobrazenia sa obmedzujú presne rovnakým spôsobom). Novinkou vo verzii 1.3.0 je, že produkt, v ktorom bol obmedzený aspoň jeden pixel, dostane sprievodnú **masku obmedzenia**, ktorá presne uvádza, ktoré pixely a pásma:

* **Kde:**v priečinku `clamp_masks/`**vedľa** priečinkov s produktmi, nikdy vo vnútri nich: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif` a `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Čo:** jednopásmový súbor TIFF s názvom `uint8` na vlastnej pixelovej mriežke produktu (vrátane orientácie, oblasti záujmu (ROI), zlučovania pixelov a zarovnania poľa). Bit 0 (`mask & 1`) je nastavený tam, kde bolo pásmo v kanáli R produktu orezané, bit 1 (`& 2`) v kanáli G, bit 2 (`& 4`) v kanáli B. Podiel orezaných pixelov na jednotlivých pásmach je v produkte tiež označený ako `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` a `Chloros:BandBUnmixClampFraction`.
* **Ako to uvádza produkt:** XMP značka `Chloros:UnmixClampMask` obsahuje cestu k maske vzhľadom na zložku produktu (`../clamp_masks/<product folder>/<name>_clampmask.tif`, vždy oddelené znakom `/`), `none`, ak nedošlo k orezaniu žiadneho pixelu (maska sa nezapíše), alebo `unavailable`, ak sa maska nemohla zapísať. Produkt bez značky nikdy neprešiel procesom oddelenia farieb (kamery RGB a monochromatické, úrovne raw a po odstránení bayesovského kódu).

Keďže žiadna zložka produktu neobsahuje masku, môžete zložku produktu, napríklad `Reflectance_Calibrated_Images/`, odovzdať do Pix4D, Metashape alebo iného fotogrametrického softvéru tak, ako je, vrátane podzložiek – odovzdajte zložku produktu, nie formátovú zložku nad ňou. Vlastný import programu Chloros vynecháva masky typu „clamp“. CLI ich počíta samostatne: `Image products written: N (plus M clamp masks)`.

## Indexové produkty

Každý index sa ukladá do vlastnej zložky `<INDEX>_Index_Images/` (napr. `NDVI_Index_Images/`).

* **Celé formáty**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) ukladajú**lineárne roztiahnutie pre každý obrázok**, nie mapu −1…1: DN 0 je najnižšia indexová hodnota daného snímku a plný rozsah je najvyššia. Súbor obsahuje `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` a `Chloros:IndexMax`, teda aj `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Keďže každý snímok má svoje vlastné koncové body, porovnávajte snímky podľa hodnôt, nie podľa DN.
* **`TIFF (32-bit, Percent)`** ukladá samotné indexové hodnoty ako float32 bez preškálovania — ide o najlepšiu voľbu pre kvantitatívnu prácu v rámci celého letu.

Novinkou vo verzii 1.3.0 je, že indexový produkt popisuje **sám seba** namiesto opakovania značiek zdrojových surových dát. Jeho GPS a kamerové značky sa skopírujú z produktu, z ktorého bol vypočítaný (produkt odrazivosti, ak existuje), a potom:

| Značka | Hodnota |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (takže súbor sa nikdy znovu neimportuje ako záznam) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / použitý vzorec |
| `Chloros:IndexLevel` | `reflectance` alebo `radiance` — z čoho bol index vypočítaný (chýba, ak nie je ani jedno) |
| `Chloros:IlluminationInvariant` | `True` pre index vypočítaný z odrazivosti, `False` z jasu alebo hodnôt senzora |
| `Chloros:ChlorosPipelineVersion` | verzia programu Chloros, ktorá index vytvorila |

`Chloros:PixelScale` je z indexových produktov odstránené — použite vyššie uvedené `IndexMin` / `IndexMax`. Úplný postup spätného načítania nájdete v časti „Celé číslo indexového rastra predstavuje kontrastné roztiahnutie na jeden obrázok“ v [Referencii CLI](reference/cli-reference.md).

## Čítanie hodnôt odrazivosti pixelov

Odrazivosť je uložená ako celé číslo a **DN, ktoré zodpovedá ρ = 1,0 (100 % odrazivosť), závisí od zdrojovej kamery**:

| Zdrojová kamera | ρ = 1,0 je DN | Ako zistiť |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (rozsah až do ρ 2,0) | V súbore je vložená značka XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (obmedzené na ρ 1,0) | Chýba značka `Chloros:PixelScale` — táto absencia je signálom. (Produkty Survey3 obsahujú od verzie 1.3.0 v každom režime spracovania aj iné značky spracovania `Chloros:*`, napríklad `Chloros:CalibrationApplied`.) |

**Prečítajte značku XMP `Chloros:PixelScale` a vydelte ňou** namiesto predpokladania konštanty. Značka je definovaná v doméne uint16, takže zostáva `32768` vo všetkých výstupných formátoch, ktoré menia mierku — najprv normalizujte uložený dátový typ späť na uint16 (×257 z 8-bitového, ×65535 z float32).

{% hint style="warning" %}
**8-bitové záznamy.** Výstupy 8-bitového záznamu sú 8-bitové. Jeho odrazivosť je preškálovaná a označená ako `Chloros:PixelScale=32768`, takže mierka zostáva zachovaná; 8-bitové produkty DN a súbory odrazivosti vytvorené pred verziou 1.3.0 sú obmedzené na rozsah 0–255 a neobsahujú žiadnu značku mierky. Ak v súbore odrazivosti LATTICE chýba táto značka, nepredpokladajte žiadnu mierku; namiesto toho vykonajte reexport v 16-bitovom alebo 32-bitovom formáte.
{% endhint %}

Úplné pravidlá (vrátane značiek kompatibilných s MicaSense) nájdete v časti **„Čítanie pixelov odrazivosti“** v [Referencii CLI](reference/cli-reference.md).

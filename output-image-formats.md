---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formáty výstupných obrázkov

Chloros exportuje spracované výsledky v štyroch formátoch súborov. Formát vyberte v nastaveniach projektu (GUI), pomocou `--format` (CLI) alebo pomocou `export_format` (SDK). Funkcie CLI a SDK akceptujú presne nižšie uvedené reťazce.

| Formátovací reťazec | Rozšírenie | Typ pixelu | Rozsah pixelov | Poznámky |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(predvolené)* | `.tif` | uint16 digitálne číslo | 0 – 65535 | Odporúčané pre fotogrametriu / GIS. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % odrazivosť. Niektoré aplikácie nedokážu čítať súbory TIFF s plávajúcou desatinnou čiarkou; súbory sú väčšie. |
| `PNG (8-bit)` | `.png` | uint8 – digitálne číslo | 0 – 255 | Bezstratná kompresia, vhodná na prehliadanie na webe a vizualizáciu. |
| `JPG (8-bit)` | `.jpg` | číselná hodnota typu uint8 | 0 – 255 | Kompresia so stratou, najmenšie súbory. |

## Umiestnenie výstupných súborov

Výstupné súbory sa ukladajú do projektovej zložky, zoskupené podľa kamery a následne podľa formátu súboru:

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

Zložka kamery je `LATT-<sensor>-<lens>-F<filter>` pre LATTICE a `<model>_<filter>` (napr. `Survey3N_RGN`) pre Survey3. **Každý exportovaný produkt si zachováva názov zdrojového súboru – produkt identifikuje zložka, nie príponou súboru.** Úplné pravidlá nájdete v časti [Kam sa ukladajú výstupy](reference/cli-reference.md) v referenčnom materiáli k CLI.

## Produkty LATTICE (úrovne zachytávania a exportu)

Jeden surový snímok LATTICE sa v jednom prechode rozdelí na všetky požadované produkty. Každý typ produktu má vlastné prepínacie tlačidlá (začiarkavacie políčka v grafickom rozhraní alebo CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, všetky sú štandardne zapnuté):

| Úroveň | Obsah | Typ údajov |
| --- | --- | --- |
| `raw` | Dáta Bayer priamo zo senzora (monochromatické kamery: jedno spektrálne pásmo). Spracovanie vždy začína od surových dát. | Tak, ako boli zachytené |
| `debayered` | Lineárna demosaika — 3-kanálová pre M3C, 1-kanálová v odtieňoch šedej pre M3M. | Lineárna DN |
| `radiance` | Absolútna spektrálna žiarivosť z úplného radiometrického reťazca, v **W/m²/sr/nm**. Vždy sa zapisuje ako 32-bitový TIFF (`tiff32/Radiance_Images/`), bez ohľadu na zvolený formát exportu. | float32 |
| `reflectance` | Odrazivosť ρ, kde **DN 32768 = ρ 1,0 (100 %)** s rezervou až do ρ 2,0. Pripravené pre Pix4D. | uint16 |
| `preview` | Render pripravený na zobrazenie: RGB = vyváženie bielej + gama; multispektrálny = rozloženie falošných farieb. | 8-bitové zobrazenie |

## Čítanie hodnôt odrazivosti pixelov

Odrazivosť je uložená ako celé číslo a **DN, ktoré znamená ρ = 1,0 (100 % odrazivosť), závisí od zdrojovej kamery**:

| Zdrojová kamera | ρ = 1,0 je DN | Ako zistiť |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (rozsah až do ρ 2,0) | V súbore je vložená značka XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (obmedzený na ρ 1,0) | Chýbajú značky XMP typu `Chloros:*` — táto absencia je signálom. |

**Prečítajte značku XMP `Chloros:PixelScale` a vydelte ňou** namiesto toho, aby ste predpokladali konštantu. Značka je definovaná v doméne uint16, takže zostáva `32768` vo všetkých výstupných formátoch, ktoré menia mierku — najprv normalizujte uložený dátový typ späť na uint16 (×257 z 8-bitového, ×65535 z float32).

{% hint style="warning" %}
**Jeden prípad nemá žiadne mierky, čo je zámerom.** Keď sa 8-bitový zdrojový záznam (BayerRG8) zapíše ako 8-bitový TIFF, potrubie obmedzí rozsah na 0–255 namiesto preškálovania, takže súbor nemá žiadne mierky — Chloros tam zámerne vynecháva `Chloros:PixelScale`. Ak v súbore odrazivosti LATTICE chýba táto značka, nepredpokladajte žiadnu mierku; namiesto toho súbor znovu exportujte v 16-bitovom alebo 32-bitovom formáte.
{% endhint %}

Úplné pravidlá (vrátane značiek kompatibilných s MicaSense) nájdete v časti **„Čítanie pixelov odrazivosti“** v [Referencii CLI](reference/cli-reference.md).

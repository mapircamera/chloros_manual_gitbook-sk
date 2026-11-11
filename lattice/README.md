# Kamery LATTICE

LATTICE je modulárny multispektrálny kamerový systém spoločnosti MAPIR určený na poľnohospodárske a vedecké snímanie. Každá kamera LATTICE je postavená na senzore Sony IMX265 s globálnou uzávierkou (**3,1 MP, pixely 3,45 µm**) a pripája sa cez Ethernet ako zariadenie**GigE Vision**.

Chloros 1.3.0 ovláda kamery LATTICE v reálnom čase — vyhľadávanie, živý náhľad, snímanie a synchronizované zostavy viacerých kamier — z troch rozhraní:

| Rozhranie    | Kde                                                          | Platformy                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| Grafické rozhranie        | Karta **Kamery** v bočnom paneli Chloros                         | Windows 10/11 x64                                        |
| CLI        | Rodina príkazov `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Hľadáte hardvér?**Kamerové moduly, objektívy, filtre a pásma, rámy a upevnenia, káble, PoE a spúšťacie zapojenie sú zdokumentované v [**užívateľskej príručke LATTICE**](https://mapir.gitbook.io/lattice-camera). Táto kapitola sa zaoberá ovládaním kamier z prostredia Chloros.

Záznamy z LATTICE sú štandardné súbory typu `.tif`/`.tiff` a Chloros ich vždy spracováva na základe surových záznamov. Úplný zoznam príkazov a rozhranie API nájdete v [Referencii CLI](../reference/cli-reference.md) a [Referencii SDK](../reference/sdk-reference.md).

## Dve konfigurácie senzorov

| Konfigurácia | Senzor       | Filter                                | Čo poskytuje jedna kamera                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayerova farebná | trojitý pásmový filter                |**Tri kalibrované pásma z jednej expozície**                 |
| **M3M**| Monochromatická   | jeden úzkopásmový interferenčný filter |**Jedno kalibrované pásmo**; kombináciou viacerých kamier M3M získate indexy |

Keďže kamera M3M je monochromatická a vybavená jediným filtrom, každé pásmo má svoju vlastnú expozíciu. Kamera M3C pokrýva všetky tri svoje pásma jednou expozíciou snímača.

## Reťazce modelov a názvy

Každá kamera ukladá svoju identitu v GenICam `DeviceUserID` ako reťazec modelu:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros ho zobrazuje s predponou `LATT-` (napríklad `LATT-M3M-L87-F450`). Ten istý reťazec `LATT-…` sa zapisuje do EXIF tagu `Model` pri každom exporte a používa sa ako názov výstupnej zložky fotoaparátu v spracovaných projektoch.

| Složka | Hodnoty                                                   | Význam                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Snímač    | `M3C` / `M3M`                                            | Bayerova farebná matica / monochromatická matica                                                                          |
| Objektív | `L41` / `L87`                                            | Číslo udáva **horizontálne zorné pole v stupňoch**: L41 = úzke (41°), L87 = široké (87°)    |
| Filter    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) alebo `F<nm>` (M3M) | Pozri [Filtry a spektrálne pásma](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Reťazec modelu riadi všetko, čo nasleduje: Chloros určuje profil senzora, rozloženie pásiem a továrenskú kalibráciu na základe `DeviceUserID` + `DeviceSerialNumber`. Pre jednotlivé kamery nie je potrebné nič konfigurovať — pozri [Pripojenie kamier](connecting.md).

## Filtre a spektrálne pásma

Stredy pásiem, šírky FWHM a kompletný katalóg 23 produktov M3M sú špecifikáciami produktov, preto sa nachádzajú v príručke k hardvéru: [**Filtre a spektrálne pásma**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Čo je dôležité zo softvérovej stránky: kód filtra v reťazci modelu určuje, ktoré produkty môže Chloros vytvoriť. Kamery s RGB filtrom (`FRGB`) poskytujú iba produkty po odstránení bayesovského kódu a náhľady — žiarivosť a odrazivosť na jednotlivé pásma nemajú pre širokopásmový senzor význam, preto ich Chloros vynecháva a uvádza to. Každý iný filter poskytuje kompletný reťazec žiarivosť → odrazivosť → index.

## Radiometrická kalibrácia v skratke

Každá kamera LATTICE je individuálne kalibrovaná vo výrobe podľa reťazca vysledovateľného podľa NIST a dodáva sa s certifikátom pre každú kameru. Čo táto kalibrácia zahŕňa, ako sa meria a akú presnosť môžete uvádzať, nájdete v príručke k hardvéru: [**Továrenská radiometrická kalibrácia**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Pokiaľ ide o softvér, dôležité je, aby aplikácia Chloros pri pripojení kamery určila správnu kalibráciu a aby použité koeficienty boli zachované pri každom exporte — pozri [Pripojenie kamier](connecting.md).

## V tejto kapitole

* [Pripojenie kamier](connecting.md) — automatické vyhľadávanie, dialógové okno pre pripojenie v grafickom rozhraní, ekvivalenty v CLI a SDK a spôsob určenia továrenskej kalibrácie (balík v kamere vs. cloud) pri pripojení kamery.
* [Nastavenia kamier](camera-settings.md) — všetky ovládacie prvky na karte Kamery: zoznam kamier, panel nastavení pre jednotlivé kamery a panel nastavení poľa (vrátane editora ROI a zarovnania).
* [Nastavenia a režimy snímania](capture.md) — Snímanie všetkého, jednorazové / nepretržité / intervalové snímanie, typy exportu a štruktúra priečinka so snímkami.
* [Súbory viacerých kamier](arrays.md) — sprievodca pripojením súboru kamier, sonda synchronizačného kábla, šírka pásma, PTP a živý náhľad súboru kamier.
* [Mono kamery a vegetačné indexy](mono-indices.md) — kamery M3M a pracovný postup zarovnanie → zoskupenie → index.

Kompletný zoznam príkazov nájdete v [Referencii CLI](../reference/cli-reference.md) a [Referencii SDK](../reference/sdk-reference.md).

# Kamery LATTICE

LATTICE je modulárny multispektrálny kamerový systém spoločnosti MAPIR určený na poľnohospodárske a vedecké snímanie. Každá kamera LATTICE je postavená na senzore Sony IMX265 s globálnou uzávierkou (**3,1 MP, pixely 3,45 µm**) a pripája sa cez Ethernet ako zariadenie**GigE Vision**.

Chloros 1.2.0 ovláda kamery LATTICE v reálnom čase — vyhľadávanie, živý náhľad, snímanie a synchronizované usporiadania viacerých kamier — z troch rozhraní:

| Rozhranie    | Kde                                                          | Platformy                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| Grafické používateľské rozhranie        | Karta **Kamery** v bočnom paneli Chloros                         | Windows 10/11 x64                                        |
| CLI        | Rodina príkazov `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86_64, Linux aarch64 (Jetson) |

> **Hľadáte hardvér?**Kamerové moduly, objektívy, filtre a pásma, rámy a upevnenia, káble, PoE a spúšťacie vedenie sú zdokumentované v [**užívateľskej príručke LATTICE**](https://mapir.gitbook.io/lattice-camera). Táto kapitola sa zaoberá ovládaním kamier z Chloros.

Záznamy LATTICE sú štandardné súbory typu `.tif`/`.tiff` a Chloros ich vždy spracováva na základe surových záznamov. Kompletný zoznam príkazov nájdete v [Referencii k CLI](../reference/cli-reference.md) a [Referencii k SDK](../reference/sdk-reference.md), kde nájdete kompletný zoznam príkazov a rozhranie API.

## Dve konfigurácie senzorov

| Konfigurácia | Senzor       | Filter                                | Čo poskytuje jedna kamera                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayerova farebná | trojitý pásmový filter                |**Tri kalibrované pásma z jednej expozície**                 |
| **M3M**| Monochromatická   | jeden úzkopásmový interferenčný filter |**Jedno kalibrované pásmo**; pre indexové hodnoty kombinujte viacero kamier M3M |

Keďže kamera M3M je za jediným filtrom monochromatická, každé pásmo má svoju vlastnú expozíciu. Kamera M3C pokrýva všetky tri svoje pásma jednou expozíciou snímača.

## Reťazce modelov a pomenovanie

Každá kamera ukladá svoju identitu v GenICam `DeviceUserID` ako reťazec modelu:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros ho zobrazuje s predponou `LATT-` (napríklad `LATT-M3M-L87-F450`). Ten istý reťazec `LATT-…` sa zapisuje do EXIF značky `Model` pri každom exporte a používa sa ako názov výstupnej zložky fotoaparátu v spracovaných projektoch.

| Složka | Hodnoty                                                   | Význam                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Snímač    | `M3C` / `M3M`                                            | Bayerova farebná matica / monochromatická matica                                                                          |
| Objektív | `L41` / `L87`                                            | Číslo udáva **horizontálne zorné pole v stupňoch**: L41 = úzke (41°), L87 = široké (87°)    |
| Filter    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) alebo `F<nm>` (M3M) | Pozri [Filtry a spektrálne pásma](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Reťazec modelu riadi všetko, čo nasleduje: Chloros určuje profil senzora, rozloženie pásiem a továrenskú kalibráciu na základe kombinácie `DeviceUserID` a `DeviceSerialNumber`. Na úrovni jednotlivých kamier nie je potrebné nič konfigurovať — pozri [Pripojenie kamier](connecting.md).

## Filtre a pásma

Stredy pásiem, okraje FWHM a kompletný katalóg 23 položiek M3M sú špecifikáciami produktu, takže sa nachádzajú v príručke k hardvéru: [**Filtre a spektrálne pásma**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Čo je dôležité zo softvérovej stránky: kód filtra v reťazci modelu určuje, ktoré produkty môže Chloros vytvoriť. Kamery s filtrom RGB (`FRGB`) generujú iba produkty po odstránení bayeresovho kódovania a náhľady — žiarivosť a odrazivosť v jednotlivých pásmach nemajú pre širokopásmový senzor význam, preto ich Chloros vynecháva a uvádza to. Každý iný filter poskytuje úplný reťazec žiarivosť → odrazivosť → index.

## Radiometrická kalibrácia v skratke

Každá kamera LATTICE je individuálne kalibrovaná vo výrobnom závode podľa reťazca vysledovateľného podľa NIST a dodáva sa s certifikátom pre každú kameru. Čo táto kalibrácia zahŕňa, ako sa meria a akú presnosť môžete uvádzať, nájdete v príručke k hardvéru: [**Továrenská radiometrická kalibrácia**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Pokiaľ ide o softvér, dôležité je, že Chloros určí správnu kalibráciu pri pripojení kamery a zafixuje použité koeficienty pri každom exporte — pozri [Pripojenie kamier](connecting.md).

## V tejto kapitole

* [Pripojenie kamier](connecting.md) — automatické vyhľadávanie, dialógové okno pripojenia v grafickom rozhraní, ekvivalenty CLI/SDK a spôsob určenia továrenskej kalibrácie (balík v kamere vs. cloud) pri pripojení kamery.

Ďalšie témy týkajúce sa LATTICE — nastavenia kamery a ovládanie v reálnom čase, režimy snímania, sústavy viacerých kamier a spracovanie a indexy mono (M3M) — sú rozpracované v samostatných častiach tejto príručky a kompletný zoznam príkazov nájdete v [CLI Referencii](../reference/cli-reference.md) a [SDK Reference](../reference/sdk-reference.md).

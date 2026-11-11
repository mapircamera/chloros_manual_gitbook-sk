# Pracovné postupy pre odrazivosť

Svetelný senzor DAQ prevádza radiometrické snímky na odrazivosť. Existujú dva odlišné pracovné postupy:

1. **Jeden senzor** — jeden senzor DAQ meria ožiarenie smerom nadol, zatiaľ čo kamera sníma, a program Chloros
vydelí žiarivosť zachytenú kamerou touto referenčnou hodnotou.
2. **Dva senzory** — dva senzory DAQ, z ktorých jeden sleduje oblohu a druhý objekt, vytvárajú v reálnom čase spektrálnu krivku odrazivosti bez použitia kamery.

## Jeden senzor + kamera (referenčné smerom nadol)

DAQ funguje ako senzor svetla smerom nadol (DLS): kamera meria odrazené žiarenie smerom nahor **L**(W/m²/sr/nm), DAQ meria ožiarenie smerom nadol**E** (W/m²/nm) a program Chloros
vypočíta odrazivosť pre každé pásmo podľa vzorca:

> ρ = π · L / E

Hodnota z DAQ je vždy **časovo zosúladená s expozíciou** — preto DAQ a kamery zdieľajú hodiny synchronizované protokolom PTP (pozri [DAQ-E Networking &amp; Time Sync](ethernet-ptp.md)). Na prácu vonku si nasaďte clonu Sunshine Cosine a správne ju deklarujte; deklarácia clony priamo ovplyvňuje hodnotu E (pozri [Profily clôn a kalibrovaný rozsah](caps-and-range.md)). Pri kvantitatívnej práci nezabudnite na charakteristiku prístroja: kvantitatívna intenzita žiarenia sa vypočíta ako priemer nameraných hodnôt za minimálne 15 s.

### Zaznamenávanie v reálnom čase

Pripojte DAQ k kamere na karte „Kamery“: panel nastavení každej kamery obsahuje roletové menu **Svetelný senzor**, v ktorom sú uvedené všetky pripojené zariadenia DAQ (DAQ-U/M/E) z karty „Svetelné senzory“; v prípade synchronizovaného poľa sa výber svetelného senzora pre celé pole prenesie na každý člen (jednotlivé kamery môžu stále prevážiť). Po priradení sa spektrá senzora prenášajú do slotu DLS kamery a exportované hodnoty odrazivosti sa delia zodpovedajúcou hodnotou. V poli používa člen s vlastným senzorom svoje vlastné namerané hodnoty — pre svoje pásma v živom kompozite, svoje exportované údaje a automatickú expozíciu — a ostatní členovia používajú hodnoty celého poľa.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Dve správania, ktoré stojí za to poznať:

* **Žiadne pripojenie DAQ → odrazivosť je odmietnutá, nie simulovaná.** Program Chloros
odmietne produkt odrazivosti a zaznamená dôvod vynechania namiesto toho, aby ticho vrátil menej presný produkt.
* **Použitá hodnota sa zachová.** Pre každý snímok odrazivosti sa skutočne použitá hodnota z DAQ zapíše ako súbor `.daq` do zložky `daq/` daného záznamu, takže záznam je možné neskôr opätovne spracovať ([Nahrávanie a formát .daq](recording.md)). Súbor `.daq.cameras.json` vedľa neho označuje kamery, ktoré popisuje, a pri opätovnom spracovaní sa priradí iba k snímkam týchto kamier.

### Spracovanie zaznamenaných snímok

Na spracovanie po lete nahrajte počas relácie súbor `.daq` a uchovajte ho spolu so snímkami — spracovateľský reťazec automaticky priradí záznam s zodpovedajúcim časovým údajom k snímkam smerujúcim nadol a pri prvom použití načíta chýbajúcu továrenskú kalibráciu z cloudu spoločnosti MAPIR
. Nahrávky z grafického používateľského rozhrania sa po zastavení automaticky pridajú do otvoreného projektu.

Referenčnú hodnotu odrazivosti je možné vybrať v čase spracovania – `--reflectance-source` na `chloros-cli process` alebo nastavenie zdroja odrazivosti v nastaveniach projektu v grafickom rozhraní:

| Hodnota | Správanie |
| --- | --- |
| `auto` (predvolené) | Cieľ kalibrácie v rámci snímky, ktorý prešiel kontrolou kvality (QA), je absolútnou referenciou; náhradným riešením je zosilnenie signálu zo zberu údajov (DAQ) (ρ = π·L/E). Cieľ, ktorý prešiel kontrolou kvality s výhradou (vypadnutý panel alebo nefunkčné pásmo), nie je automaticky uprednostňovaný: reťazce cieľ, cieľ+DAQ a iba DAQ sa vyskúšajú v poradí podľa ich nameraného skóre na vynechaných paneloch. |
| `daq` | DAQ má prednosť |
| `target` | Prísny cieľ v rámci snímky; žiadna náhrada DAQ |

Pozrite si [Kalibračné terče](../calibration-targets.md) pre pracovné postupy s terčmi a [kapitolu LATTICE](../lattice/README.md) a [referenciu CLI
](../reference/cli-reference.md) pre kompletný spracovateľský reťazec. Pri čítaní exportovaných pixelov odrazivosti použite označenú mierku (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3
: 65535) — pozri [Formáty výstupných obrázkov](../output-image-formats.md).

### Pásma mimo kalibrovaného rozsahu DAQ

Radiometricky kalibrovaný rozsah DAQ je ~374–974 nm. Program Chloros
odmietne odrazivosť získanú z DAQ pre akékoľvek pásmo kamery, ktorého menej ako polovica spektrálnej váhy leží v tomto rozsahu, a nahlási dôvod vynechania `dls-uncalibrated-band-<nm>`. Zo súčasných modelov to ovplyvňuje iba model F988: odrazivosť modelu F988 sa kalibruje pomocou panelu odrazivosti umiestneného v scéne: pásmo leží mimo kalibrovaného rozsahu svetelného senzora DAQ, preto Chloros
použije váš najnovší záznam z panelu a zachová ho medzi jednotlivými meraniami panelu. Ak je kamera F988 prevádzkovaná iba v režime DAQ, Chloros
odmietne odrazivosť založenú na DAQ pre dané pásmo s dôvodom preskočenia `dls-uncalibrated-band-988` — podporovaným postupom je práca s panelom.

## Dva senzory (okolité svetlo + objekt)

Dva senzory DAQ – ľubovoľná dvojica, cez akékoľvek rozhrania – poskytujú spektrum odrazivosti v reálnom čase bez kamery: jeden senzor je nasmerovaný na oblohu (**Zdroj okolitého svetla**), druhý na objekt (**Skener objektu**) a Chloros
vypočíta hodnotu pre každú vlnovú dĺžku:

> R(λ) = objekt(λ) / okolité(λ)

(nula, ak okolité ≤ 0).

### V grafickom rozhraní

Keď sú oba senzory pripojené na karte Svetelné senzory, otvorte prekrývacie okno na pridanie senzora (tlačidlo „+“ na dlaždici grafu v mriežkovom zobrazení) a vyberte možnosť **Kombinovať okolité svetlo + objekt**. Z roletových menu**Zdroje okolitého osvetlenia**a**Skenér objektu**vyberte oba senzory a kliknite na**Vytvoriť**. Skupina sa zobrazí ako samostatný graf a ako riadok v bočnom paneli so zelenou značkou**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->


Pod grafom odrazivosti (zobrazenie zoznamu) sa v reálnom čase v **tabuľke indexov vegetácie** vypočítavajú indexy z krivky s použitím stredov pásiem: modrá 450 / zelená 550 / červená 670 / NIR
800 nm. Vždy sa zobrazujú indexy založené na pomere, ktoré rušia absolútnu stupnicu (NDVI
, GNDVI
, ENDVI, WDRVI
, GRVI
, CVI
, GCI
, MSR) sa zobrazujú vždy; indexy, ktoré vyžadujú absolútnu odrazivosť (EVI
, SAVI
, OSAVI
, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI
, NLI, MNLI, FCI, GEMI
), sa zobrazujú len vtedy, ak sú oba senzory modelmi s kalibráciou výkonu.

### Zdanlivá vs. relatívna — pravidlo označovania

Chloros
označuje výstup dvoch senzorov podľa toho, čo môže daná dvojica senzorov skutočne poskytovať:

| Dvojica senzorov | Označenie |
| --- | --- |
| Oba senzory kalibrované — načítaný továrenský balík | **Zdanlivá odrazivosť** |
| Akýkoľvek senzor je nekalibrovaný | **Relatívna odrazivosť** |

Všetky tri modely sú radiometrické: akonáhle je načítaný továrenský kalibračný balík senzora, jeho spektrá sú vyjadrené v absolútnych hodnotách W/m²/nm, takže pomer medzi dvoma kalibrovanými senzormi zodpovedá absolútnej zdanlivej odrazivosti — transportný koeficient o tom nerozhoduje. Senzor, ktorý stále vysiela surové počty impulzov (balík nie je dostupný), zníži výsledok na relatívnu krivku (tvar spektra zostáva platný). Oba senzory by mali mať správne deklarované limity ([Profily limitov a kalibrovaný rozsah](caps-and-range.md)).

### Z jazyka Python


V zjednotenom rozhraní SDK
neexistuje špeciálna funkcia pre dva senzory: otvorte dve relácie s `chloros_sdk.connect_daq_sensor()` a sami porovnajte ich spektrá s `latest()`, pričom použite rovnakú konvenciu označovania. (Nástroj na záznam s dvoma senzormi existuje aj vo vnútornom rozhraní MAPIR
pre priamy prístup k hardvéru, ktorý je pre úplnosť uvedený v [Referencii CLI
](../reference/cli-reference.md) — nie je súčasťou dodávaného CLI
; podporovaným postupom v reálnom čase je vyššie uvedený pracovný postup s grafickým rozhraním.)

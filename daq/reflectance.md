# Pracovné postupy založené na odrazivosti

Svetelný senzor DAQ prevádza radiometrické snímky na hodnoty odrazivosti. Existujú dva odlišné pracovné postupy:

1. **Jeden senzor** — jeden senzor DAQ meria intenzitu slnečného žiarenia smerujúceho nadol, zatiaľ čo kamera sníma obraz, a Chloros delí žiarivosť kamery touto referenčnou hodnotou.
2. **Dva senzory** — dva senzory DAQ, z ktorých jeden sleduje oblohu a druhý objekt, vytvárajú spektrálnu krivku odrazivosti v reálnom čase bez použitia kamery.

## Jeden senzor + kamera (referenčné smerom nadol)

DAQ funguje ako senzor svetla smerom nadol (DLS): kamera meria žiarivosť smerujúcu nahor **L**(W/m²/sr/nm), zariadenie DAQ meria ožiarenie smerujúce nadol**E** (W/m²/nm) a Chloros vypočíta odrazivosť pre každé pásmo podľa vzorca:

> ρ = π · L / E

Hodnota z DAQ je vždy **časovo zosúladená s expozíciou** — preto DAQ a kamery zdieľajú hodiny riadené protokolom PTP (pozri [DAQ-E Networking &amp; Time Sync](ethernet-ptp.md)). Pri práci vonku si nasaďte košínusovú čiapku Sunshine a správne ju deklarujte; deklarácia čiapky priamo ovplyvňuje hodnotu E (pozri [Profily čiapok a kalibrovaný rozsah](caps-and-range.md)). Pri kvantitatívnej práci nezabudnite na charakteristiku prístroja: kvantitatívna intenzita žiarenia vychádza z priemeru meraní trvajúcich minimálne 15 sekúnd.

### Zaznamenávanie v reálnom čase

Pripojte zariadenie DAQ ku kamere na karte „Kamery“: v paneli nastavení každej kamery sa nachádza roletové menu **Svetelný senzor**, ktoré obsahuje zoznam všetkých pripojených zariadení DAQ (DAQ-U/M/E) z karty „Svetelné senzory“; v prípade synchronizovaného poľa sa výber svetelného senzora pre celé pole prenesie na každý prvok (jednotlivé kamery môžu stále prevážiť toto nastavenie). Po priradení sa spektrá senzora prenášajú do slotu DLS kamery a exportované hodnoty odrazivosti sa delia zodpovedajúcou nameranou hodnotou.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

Dve správania, ktoré stojí za to poznať:

* **Žiadne priradené zariadenie DAQ → hodnota odrazivosti je odmietnutá, nie je falšovaná.** Chloros odmietne výsledok odrazivosti a zaznamená dôvod vynechania, namiesto toho, aby bez upozornenia vrátil nižšiu hodnotu.
* **Použitá nameraná hodnota sa zachová.** Pre každý snímok odrazivosti sa skutočne použitá hodnota DAQ zapíše ako doplnkový súbor `.daq` vedľa snímky, takže záznam je možné neskôr opätovne spracovať ([Záznam a formát .daq](recording.md)).

### Spracovanie zaznamenaných snímok

Na účely spracovania po lete zaznamenajte počas relácie súbor `.daq` a uchovajte ho spolu so snímkami — spracovateľský reťazec automaticky vyrieši zhodu časových údajov smerom nadol a pri prvom použití stiahne akékoľvek chýbajúce továrenské kalibrácie z cloudu MAPIR. Záznamy z grafického rozhrania sa po zastavení automaticky pridajú do otvoreného projektu.

Referenčnú hodnotu odrazivosti je možné vybrať v čase spracovania – buď `--reflectance-source` na `chloros-cli process`, alebo nastavenie zdroja odrazivosti v nastaveniach projektu v grafickom rozhraní:

| Hodnota | Správanie |
| --- | --- |
| `auto` (predvolené) | Absolútnou referenciou je kalibračný cieľ v rámci snímky, ktorý prešiel kontrolou kvality (QA); záložným riešením je zosilnenie DAQ smerom nadol (ρ = π·L/E) |
| `daq` | Rozhodujúce je DAQ |
| `target` | Prísny cieľ v rámci snímky; žiadna náhrada DAQ |

Pozri [Kalibračné ciele](../calibration-targets.md) pre pracovné postupy týkajúce sa cieľov a [kapitolu LATTICE](../lattice/README.md) a [Referenciu CLI](../reference/cli-reference.md) pre kompletný spracovateľský reťazec. Pri čítaní exportovaných pixelov odrazivosti použite označené mierky (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) — pozri [Formáty výstupných obrázkov](../output-image-formats.md).

### Pásma mimo kalibrovaného rozsahu DAQ

Radiometricky kalibrovaný rozsah DAQ je ~374–974 nm. Chloros odmieta odrazivosť založenú na DAQ pre akékoľvek pásmo kamery, ktorého menej ako polovica spektrálnej váhy sa nachádza v tomto rozsahu, pričom ako dôvod vynechania uvádza `dls-uncalibrated-band-<nm>`. Zo súčasných modelov to ovplyvňuje iba model F988: odrazivosť kamery F988 je kalibrovaná pomocou panelu odrazivosti umiestneného v scéne: pásmo leží mimo kalibrovaného rozsahu svetelného senzora DAQ, takže Chloros použije váš najnovší záznam z panelu a zachová ho medzi jednotlivými meraniami panelu. Ak je kamera F988 prevádzkovaná iba v režime DAQ, kód Chloros odmietne odrazivosť založenú na DAQ pre toto pásmo s dôvodom preskočenia `dls-uncalibrated-band-988` — podporovaným postupom je práca s panelom.

## Dva senzory (okolité svetlo + objekt)

Dva senzory DAQ — ľubovoľná dvojica, na akýchkoľvek nosičoch — poskytujú spektrum odrazivosti v reálnom čase bez kamery: jeden senzor je nasmerovaný na oblohu (**Zdroj okolitého svetla**), druhý na objekt (**Skener objektu**) a Chloros vypočíta pre každú vlnovú dĺžku:

> R(λ) = objekt(λ) / okolité(λ)

(nula, ak okolité ≤ 0).

### V grafickom rozhraní

Keď sú oba senzory pripojené na karte Svetelné senzory, otvorte prekrývacie okno na pridanie senzora (tlačidlo „+“ na dlaždici grafu v mriežkovom zobrazení) a vyberte možnosť **Kombinovať okolité svetlo + objekt**. Z roletových menu „Ambient Light Source“ a „Object Scanner“ vyberte oba senzory a kliknite na „Create“. Skupina sa zobrazí ako samostatný graf a ako riadok v bočnom paneli so zelenou značkou**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

Pod grafom odrazivosti (zobrazenie zoznamu) sa v reálnom čase zobrazuje **tabuľka indexov vegetácie**, ktorá vypočítava indexy z krivky pomocou stredov vlnových dĺžok modrej 450 / zelenej 550 / červenej 670 / NIR 800 nm. Indexy založené na pomeroch, ktoré rušia absolútnu mierku (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) sa vždy zobrazujú; indexy, ktoré vyžadujú absolútnu odrazivosť (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) sa zobrazujú len vtedy, ak sú oba senzory modelmi s kalibráciou podľa výkonu.

### Zdánlivá vs. relatívna – pravidlo označovania

Chloros označuje výstup dvoch snímačov podľa toho, čo môže daná dvojica snímačov skutočne uvádzať:

| Dvojica snímačov | Označenie |
| --- | --- |
| Oba snímače kalibrované — načítaný výrobný kalibračný balík | **Zdánlivá odrazivosť** |
| Akýkoľvek senzor nekalibrovaný | **Relatívna odrazivosť** |

Všetky tri modely sú radiometrické: po načítaní továrenského kalibračného balíka senzora sú jeho spektrá vyjadrené v absolútnych hodnotách W/m²/nm, takže pomer dvoch kalibrovaných senzorov zodpovedá absolútnej zdanlivej odrazivosti — transportný režim o tom nerozhoduje. Senzor, ktorý naďalej vysiela surové počty impulzov (balík nie je dostupný), zníži výsledok na relatívnu krivku (tvar spektra zostáva platný). Oba senzory by mali mať správne deklarované limity ([Profily limitov a kalibrovaný rozsah](caps-and-range.md)).

### Z Python

V združenom rozhraní SDK neexistuje špeciálna funkcia pre dva senzory: otvorte dve relácie s `chloros_sdk.connect_daq_sensor()` a sami porovnajte ich spektrá `latest()`, pričom použite rovnakú konvenciu označovania. (Nástroj na záznam s dvoma senzormi existuje aj na vnútornom povrchu MAPIR s priamym hardvérovým pripojením, uvedenom v [Referencii CLI](../reference/cli-reference.md) pre úplnosť — nie je súčasťou dodávaného CLI; vyššie uvedený pracovný postup v grafickom rozhraní je podporovanou cestou.)

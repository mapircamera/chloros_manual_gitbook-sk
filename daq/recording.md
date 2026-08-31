# Záznam a formát .daq

Súbor `.daq` je formát záznamu svetelného senzora Chloros: ide o **databázu SQLite** obsahujúcu kalibrované spektrálne snímky z jedného senzora DAQ. Zaznamenajte jeden takýto súbor počas snímacej relácie a proces spracovania odrazivosti môže neskôr každý obraz vydeliť intenzitou slnečného žiarenia smerujúceho nadol, nameranou presne v danom okamihu.

## Čo obsahuje súbor .daq

| Vlastnosť | Hodnota |
| --- | --- |
| Kontajner | Databáza SQLite, jeden súbor na senzor na jeden záznam |
| Názov súboru | Obsahuje **ID senzora**a**časovú pečiatku**, napr. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum na snímku | 135 bodov, 340–1010 nm v krokoch po 5 nm, plus tristimulačné súradnice CIE XYZ |
| Jednotky | Kalibrované spektrálne ožiarenie, **W/m²/nm** (uplatnený balík továrenskej kalibrácie + profil krytu) |
| Vložené metadáta | ID senzora (kľúč na načítanie továrenskej kalibrácie daného zariadenia) a platný profil krytu — pozri [Profily krytov a kalibrovaný rozsah](caps-and-range.md) |

Formát je identický pre DAQ-U, DAQ-M aj DAQ-E, takže pri následnom spracovaní nezáleží na tom, ktorý prenosový modul údaje zaznamenal.

Kalibrované zaznamenávanie vyžaduje balík továrenskej kalibrácie senzora. V prípade zariadení DAQ-U a DAQ-M backend načíta balík z cloudu MAPIR na základe ID senzora (ak sa to nepodarí, záznam sa odmietne); zariadenia DAQ-E sú z tohto vyňaté, pretože si kalibráciu uchovávajú priamo v zariadení.

## Záznam z grafického používateľského rozhrania

Záznam v grafickom rozhraní vyžaduje **otvorený projekt** (inak sú tlačidlá „Záznam“ deaktivované):

* **Záznam všetkého / Zastaviť všetko** — v hornej časti bočného panela „Svetelné senzory“; spustí alebo zastaví záznam `.daq` na všetkých pripojených senzoroch naraz.
* **Nahrávať / Zastaviť nahrávanie** — pre každý senzor zvlášť, v modálnom okne nastavení (ikona ozubeného kolesa). Počas nahrávania sa v riadkoch s aktuálnymi informáciami o senzore zobrazuje červený indikátor „REC“.

Súbory sa ukladajú do `<project>/light_sensor/` a keď sa nahrávanie zastaví — či už pomocou tlačidla Zastaviť, Zastaviť všetko alebo odpojením snímača, ktorý nahráva — hotový súbor `.daq` sa **automaticky pridá do otvoreného projektu**. Objaví sa v zozname súborov projektu bez potreby ručného pridávania a je už pripravený na spracovanie odrazivosti.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Nahrávanie z CLI

CLI nahráva prostredníctvom skupiny senzorov backendu (backend musí bežať – tieto príkazy sú ľahkými klientmi HTTP):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Získaj hodnotu `--sensor-id` z `chloros-cli daq pool-list`. Dve predvolené nastavenia, ktoré stojí za to poznať:

| Možnosť | Predvolené |
| --- | --- |
| `--duration` | `0` — zaznamenávať až do `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` na súborovom systéme **backendu**, nie na súborovom systéme CLI |

Rozdiel vo výstupnom adresári má význam, ak súbor CLI smeruje na backend na inom počítači: súbor sa uloží tam, kde beží backend.

## Nahrávanie z Python

`DAQSensorSession` (vrátené funkciou `chloros_sdk.connect_daq_sensor()`) poskytuje prístup k tej istej združenej nahrávke: `record_start(output_dir=None, device_name=None)` vráti cestu k súboru, `record_stop()` vráti `{path, rows}`. Úplnú reláciu API nájdete v [Referencii k SDK](../reference/sdk-reference.md). Triedy priamo prepojené s hardvérom v SDK (len pri inštalácii na stolných počítačoch) štandardne zapisujú záznamy do `~/Documents/DAQ/`; pre vydané verzie je podporovanou cestou vyššie uvedená združená cesta.

## Použitie súboru .daq počas spracovania

Na výpočet odrazivosti zo snímok potrebuje Chloros hodnotu ožiarenia smerujúceho nadol zodpovedajúcu každej expozícii:

* **Uložte súbor `.daq` spolu so snímkami.**Počas spracovania spracovateľský reťazec automaticky určí**intenzitu žiarenia smerujúceho nadol zodpovedajúcu časovej pečiatke** na základe zaznamenaného súboru `.daq` (akýkoľvek model DAQ) — alebo na základe natívneho súboru DAQ-M `.csv` — nachádzajúceho sa vedľa snímok. Záznamy z grafického používateľského rozhrania túto podmienku spĺňajú automaticky, keďže sa pridávajú do projektu hneď po ich ukončení.
* **Kalibrácia sa načíta na požiadanie.** Ak balík továrenskej kalibrácie pre konkrétnu kameru alebo DAQ ešte nie je lokálne uložený v cache, Chloros ho pri prvom použití automaticky stiahne z cloudu MAPIR (jednorazové pripojenie k internetu; uložené v cache pod `~/.chloros/`).
* **Živé záznamy si zapisujú vlastný doplnkový súbor.** Pre každý snímok odrazivosti zachytený v reálnom čase sa skutočne použitá hodnota z DAQ uloží ako doplnkový súbor `.daq` vedľa snímky, takže záznam je možné neskôr opätovne spracovať bez pôvodného záznamu.

## Získanie údajov o ožiarení späť

Spracovanie projektu tiež exportuje všetky záznamy zo svetelných senzorov, ktoré obsahuje, do
zložky `Light Sensor/` vedľa obrazových produktov. Na to **nie** sú potrebné snímky:
svetelný senzor letiaci samostatne predstavuje kompletný záznam a zložka obsahujúca iba súbory typu `.daq`
je platným vstupom. Správa o behu uvádza, koľko produktov zo svetelných senzorov bolo zapísaných.

| Produkt | Čo to je |
| --- | --- |
| `<name>_calibrated.daq` | Archív, ktorý je možné opätovne spracovať, v rovnakom schéme ako živý záznam, teraz s deklaráciou kalibračného balíka, ktorý ho vytvoril. Jeho opätovný import ho **ne**kalibruje druhýkrát. |
| `<name>_calibrated.csv` | Spektrálna intenzita žiarenia v W/m²/nm na vlastnej vlnovolnej mriežke senzora, jeden riadok na jedno meranie, plus fotometrické stĺpce: celkový výkon, fotopický a skotopický lux, PPFD s rozdelením na modrú, zelenú a červenú zložku a špičková vlnová dĺžka. |

DAQ-U alebo DAQ-M, ktorého kalibračný balík nie je možné načítať – ste offline alebo
daný senzor nemá v súbore žiadnu kalibráciu – sa **preskočí s uvedením dôvodu** a nikdy sa nezapíše
ako „kalibrovaný“ súbor obsahujúci surové počty. Pripojte sa k internetu a spustite znovu. DAQ-E
má vlastnú kalibráciu, takže ju potrebuje len vtedy, keď nie je zariadenie pripojené a
v lokálnej pamäti nie je nič uložené.

### DAQ-A: surové počty a prečo je to správna odpoveď

**DAQ-A** predchádza systému kalibračných balíkov pre jednotlivé sériové čísla a nemá žiadny balík, ktorý by bolo možné
načítať. Nie je to prehliadnutie: DAQ-A sa kalibruje v teréne pomocou
reflexného terčíka a kalibrácia založená na terčíku vyžaduje iba *relatívnu*
odozvu senzora — čo sú presne jeho surové počty. Chloros sa s nimi dnes kalibruje.

Záznam z DAQ-A sa teda exportuje, ale pod iným názvom:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, nie `_calibrated` – ide o iný názov súboru, nie o príznak vnútri súboru,
pretože tento údaj musí pretrvať aj v prípade, že sa súbor odošle e-mailom len ako holý názov. Záhlavie `.csv`
uvádza `raw spectral sensor counts (NOT irradiance)` a upozorňuje, že hodnoty sú
porovnateľné **v rámci** súboru, nie medzi senzormi. Stĺpce, ktoré majú význam
len pre skutočnú intenzitu žiarenia – celkový výkon, lux, PPFD – zostávajú prázdne, namiesto toho, aby sa
vypočítavali z počtov.

Staršie záznamy DAQ-A-SD (schéma v1.01 / v1.02) zaznamenávajú len čas zápisu súboru, nie
časovú pečiatku pre každé meranie. Chloros nebude porovnávať snímky s týmito údajmi — priradenie snímky k
času zápisu by bolo nesprávne, hoci by to na prvý pohľad nevyzeralo tak — ale export ich číta správne a
CSV uvádza, na ktorom hodine sa nachádza.

Úplný popis pracovných postupov s odrazivosťou – jeden senzor s kamerou a dva senzory pre okolie/objekt – nájdete v [Pracovné postupy s odrazivosťou](reflectance.md).

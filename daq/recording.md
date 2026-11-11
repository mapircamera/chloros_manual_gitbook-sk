# Záznam a formát .daq

Súbor `.daq` je formát záznamu svetelného senzora programu Chloros: ide o **databázu SQLite** obsahujúcu kalibrované spektrálne snímky z jedného senzora DAQ. Zaznamenajte jeden takýto súbor počas snímacej relácie a proces spracovania odrazivosti môže neskôr každý obraz vydeliť intenzitou slnečného žiarenia smerujúceho nadol, nameranou presne v danom okamihu.

## Čo obsahuje súbor .daq

| Vlastnosť | Hodnota |
| --- | --- |
| Kontajner | Databáza SQLite, jeden súbor na senzor na záznam |
| Názov súboru | Obsahuje **ID senzora**a**časovú pečiatku**, napr. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum na snímku | 135 bodov, 340–1010 nm v krokoch po 5 nm, plus tristimulačné súradnice CIE XYZ |
| Jednotky | Kalibrované spektrálne ožiarenie, **W/m²/nm** (uplatnený balík továrenskej kalibrácie + profil krytu) |
| Vložené metadáta | ID senzora (kľúč na načítanie továrenskej kalibrácie danej jednotky) a platný profil krytu — pozri [Profily krytov a kalibrovaný rozsah](caps-and-range.md) |

Formát je identický pre modely DAQ-U, DAQ-M, DAQ-E a DAQ-E-S, takže pri následnom spracovaní nezáleží na tom, ktorý zariadenie záznam zaznamenalo. Záznam z DAQ-E-S okrem každého spektra ukladá aj údaje o náklone, otáčaní a sklonu zariadenia.

**Záznamy prežijú výpadok napájania (1.3.0).**Živý záznam sa zapisuje do denníka a pri každom potvrdení sa vykonáva úplná synchronizácia s diskom, takže výpadok napájania alebo ukončenie procesu spôsobí stratu len posledných približne 3 sekúnd, nie celého súboru. Záznam zastavený uprostred zápisu môže vedľa súboru `.daq` zanechať súbor `<name>.daq-journal`:**oba súbory si uchovajte a skopírujte spolu.** Program Chloros pri prvom načítaní súboru vráti denník späť. Zaneprázdnený disk spôsobuje oneskorenie súboru, nikdy však oneskorenie údajov zo senzora.

**DAQ-E, ktoré prestane odosielať údaje, sa opäť pripojí (verzia 1.3.0).** Keď prúd z DAQ-E utíchne (vypnutie a zapnutie napájania prepínača, prerušenie spojenia), backend ho opäť pripojí s pôvodnými nastaveniami a nahrávanie na ňom pokračuje v tom istom súbore `.daq` s medzerou označenou v súbore.

Kalibrované nahrávanie vyžaduje balík továrenskej kalibrácie senzora. V prípade zariadení DAQ-U a DAQ-M backend načíta balík z cloudu spoločnosti MAPIR na základe ID senzora (ak sa to nepodarí, nahrávanie sa odmietne); zariadenia DAQ-E sú z tejto povinnosti vyňaté, pretože si kalibráciu uchovávajú priamo v zariadení.

## Nahrávanie z grafického používateľského rozhrania

Nahrávanie v grafickom rozhraní vyžaduje **otvorený projekt** (inak sú tlačidlá Nahrávať deaktivované):

* **Nahrávať všetko / Zastaviť všetko** — v hornej časti bočného panela Svetelných senzorov; spustí alebo zastaví nahrávanie `.daq` na všetkých pripojených senzoroch naraz.
* **Nahrávať / Zastaviť nahrávanie** — pre každý senzor zvlášť, v modálnom okne nastavení (ikona ozubeného kolesa). Počas nahrávania sa v riadkoch s aktuálnymi informáciami o senzore zobrazuje červený indikátor „REC“.

Súbory sa ukladajú do `<project>/light_sensor/` a keď sa nahrávanie zastaví — či už pomocou tlačidla Zastaviť, Zastaviť všetko alebo odpojením snímača, z ktorého sa nahráva — hotový súbor `.daq` sa **automaticky pridá do otvoreného projektu**. Objaví sa v zozname súborov projektu bez potreby ručného pridávania a je už pripravený na spracovanie odrazivosti.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Nahrávanie z CLI



CLI nahráva prostredníctvom skupiny senzorov backendu (backend musí bežať — tieto príkazy sú jednoduché HTTP klienti):

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

Získajte hodnotu `--sensor-id` z `chloros-cli daq pool-list`. Dve predvolené nastavenia, ktoré stojí za to poznať:

| Možnosť | Predvolené |
| --- | --- |
| `--duration` | `0` — zaznamenávať až do `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` v súborovom systéme **backendu**, nie v súborovom systéme CLI |

Rozdiel vo výstupnom adresári má význam, ak CLI smeruje na backend na inom počítači: súbor sa uloží tam, kde beží backend. Ak je backend na tom istom počítači, relatívny `-o` (napríklad `./out` vyššie) sa vyrieši vo vzťahu k priečinku, v ktorom ste spustili príkaz.

## Nahrávanie z Python

u `DAQSensorSession` (vrátené funkciou `chloros_sdk.connect_daq_sensor()`) sprístupňuje tú istú združenú nahrávku: `record_start(output_dir=None, device_name=None)` vráti cestu k súboru, `record_stop()` vráti `{path, rows}`. Úplné API pre relácie nájdete v [Referencii SDK](../reference/sdk-reference.md). Triedy SDK pre priamy prístup k hardvéru (len pre inštalácie na stolných počítačoch) štandardne zapisujú záznamy do `~/Documents/DAQ/`; pre vydané zostavenia je podporovanou cestou vyššie uvedená združená cesta.

## Použitie súboru .daq počas spracovania

Na výpočet odrazivosti zo snímok potrebuje program Chloros údaje o ožiarení smerujúcom nadol zodpovedajúce každej expozícii:

* **Uložte súbor `.daq` spolu so snímkami.**Počas spracovania spracovateľský reťazec automaticky určí**intenzitu slnečného žiarenia smerujúceho nadol zodpovedajúcu časovej pečiatke** na základe zaznamenaného súboru `.daq` (akýkoľvek model DAQ) — alebo na základe natívneho súboru DAQ-M `.csv` — nachádzajúceho sa vedľa snímok. Záznamy z grafického používateľského rozhrania túto podmienku spĺňajú automaticky, keďže sa pridávajú do projektu hneď po ich ukončení.
* **Kalibrácia sa načíta na požiadanie.** Ak balík továrenskej kalibrácie pre konkrétnu kameru alebo DAQ ešte nie je lokálne uložený v cache, Chloros ho pri prvom použití automaticky stiahne z cloudu MAPIR (jednorazové pripojenie k internetu; uložené v cache pod názvom `~/.chloros/`).
* **Záznamy v reálnom čase zapisujú vlastný doplnkový súbor.** Pre každý snímok odrazivosti zachytený v reálnom čase sa skutočne použitá hodnota z DAQ uloží ako súbor `.daq` do priečinka `daq/` príslušného záznamu, takže záznam je možné neskôr opätovne spracovať bez pôvodného záznamu.
* **Záznamy z rekordérov `chloros_scripts` (verzia 1.3.0).**Súbor `.daq` vytvorený verejnými záznamníkmi [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) je uvedený ako model DAQ, ktorý ho vytvoril, a jeho limit je možné nastaviť v**Nastavenia projektu → Svetelný senzor DAQ** rovnako ako u akéhokoľvek iného záznamu. Tieto záznamníky zapisujú surové hodnoty senzora; Chloros načíta továrenskú kalibráciu senzora cez sériový port a aplikuje ju.
* **Čí je daná hodnota (1.3.0).**Vedľa každého záznamu `.daq` je v položke `.daq.cameras.json` uvedené meno kamery (kamier), ktorých snímky táto hodnota popisuje, a spracovanie ju priraďuje**len k snímkam z týchto kamier**. Samostatná kamera bez vlastného svetelného senzora si preto už nepreberá hodnotu z inej kamery – priraďte k nej senzor alebo zaznamenajte vlastnú hodnotu. Záznam vytvorený priamo pomocou DAQ (GUI Record, `daq pool-record`) a akýkoľvek záznam `.daq` bez tohto súboru (vrátane starších záznamov) stále slúži pre všetky kamery.

## Získanie intenzity osvetlenia späť

Spracovanie projektu tiež exportuje všetky záznamy svetelných senzorov, ktoré obsahuje, do
`Light Sensor/` vedľa obrazových produktov. Na to **nie** sú potrebné snímky:
samostatný záznam zo svetelného senzora je kompletný záznam a zložka obsahujúca iba súbory `.daq`
je platným vstupom. Projekt, ktorý obsahuje iba záznamy zo svetelných senzorov, ich vypisuje a
spracováva rovnako z aplikácie aj z CLI, a zostávajú v zozname aj po ukončení spustenia. Správa o spustení
uvádza, koľko produktov zo svetelných senzorov bolo vytvorených.

| Produkt | Čo to je |
| --- | --- |
| `<name>_calibrated.daq` | Archív, ktorý je možné opätovne spracovať, v rovnakom schéme ako živý záznam, teraz s deklaráciou kalibračného balíka, ktorý ho vytvoril. Jeho opätovný import ho **ne** kalibruje druhýkrát. |
| `<name>_calibrated.csv` | Spektrálna intenzita žiarenia v W/m²/nm na vlastnej vlnovej mriežke senzora, jeden riadok na jedno meranie, plus fotometrické stĺpce: celkový výkon, fotopický a skotopický lux, PPFD s rozdelením na modrú, zelenú a červenú zložku a špičková vlnová dĺžka. |

DAQ-U alebo DAQ-M, ktorého kalibračný balík nie je možné načítať — ste offline alebo
tento senzor nemá v súbore žiadnu kalibráciu — je **preskočený s uvedením dôvodu**, nikdy sa nezapíše
ako „kalibrovaný“ súbor obsahujúci surové počty. Pripojte sa k internetu a spustite znovu. DAQ-E
má vlastnú kalibráciu, takže ju potrebuje len vtedy, keď nie je zariadenie pripojené a
v lokálnej cache nie je nič uložené.

### DAQ-A: surové počty a prečo je to správna odpoveď

**DAQ-A** predchádza systému kalibračných balíkov pre jednotlivé sériové čísla a nemá žiadny balík, ktorý by bolo možné
načítať. Nie je to prehliadnutie: zariadenie DAQ-A sa kalibruje v teréne pomocou
reflexného terčíka a kalibrácia založená na terčíku vyžaduje iba *relatívnu*
odozvu senzora — čo sú presne jeho surové počty. Spoločnosť Chloros s nimi kalibruje dodnes.

Záznam z DAQ-A sa teda exportuje, ale pod iným názvom:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, nie `_calibrated` – ide o iný názov súboru, nie o príznak vnútri súboru,
pretože táto informácia musí prežiť aj v prípade, že sa súbor odošle e-mailom len ako holý názov. Záhlavie `.csv`
uvádza `raw spectral sensor counts (NOT irradiance)` a upozorňuje, že hodnoty sú
porovnateľné **v rámci** súboru, nie medzi senzormi. Stĺpce, ktoré majú význam
len pre skutočné ožiarenie — celkový výkon, lux, PPFD — zostávajú prázdne, namiesto toho, aby boli
vypočítané z počtov.

Staršie záznamy DAQ-A-SD (schéma v1.01 / v1.02) zaznamenávajú len čas zápisu súboru, nie
časovú pečiatku pre každé meranie. Chloros nebude porovnávať snímky s týmito údajmi — priradenie snímky k
času zápisu by bolo nesprávne, hoci by to na prvý pohľad nevyzeralo tak — ale export ich číta správne a
CSV súbor uvádza, na ktorom časovači sa nachádza.

Kompletný popis pracovných postupov s odrazivosťou – jeden senzor s kamerou a dva senzory (okolie/objekt) – nájdete v [Pracovných postupoch s odrazivosťou](reflectance.md).

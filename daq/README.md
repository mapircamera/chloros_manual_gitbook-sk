# Svetelné senzory DAQ

> **Hľadáte informácie o hardvéri?**Samotné senzory – modely, montáž, kryty, porty, napájanie a aplikácia SCANNER – sú popísané v**[užívateľskej príručke DAQ](https://mapir.gitbook.io/daq)**. Táto kapitola sa venuje ich používaniu v programe Chloros. Svetelné senzory**DAQ** od spoločnosti

MAPIR merajú okolité osvetlenie vo forme radiometricky kalibrovaných spektier. V programe Chloros plnia dve úlohy:

* **Samostatný spektrálny prístroj** — grafy spektra v reálnom čase, kolorimetrické údaje a záznamy `.daq`, všetko z [kartu Svetelné senzory](gui.md), cez [CLI](cli-quick-start.md) alebo prostredníctvom Python SDK.
* **Zdroj dopadajúceho žiarenia pre odrazivosť** — počas spracovania program Chloros interpoluje vaše hodnoty `.daq` na časovú pečiatku expozície každého záberu a pomocou nameraného dopadajúceho svetla prevádza žiarivosť kamery na odrazivosť (`--reflectance-source daq`); pre kalibrované pásma nie je potrebný panel v scéne.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Tri modely, jeden formát údajov

| Model | Prenos | Vyhľadávanie |
| --- | --- | --- |
| **DAQ-U** | USB (sériové) | vyhľadávanie cez sériový port |
| **DAQ-M** | Bluetooth Low Energy | vyhľadávanie BLE podľa názvu |
| **DAQ-E** | Ethernet (IPv4, napájanie cez PoE) | mDNS `_daq-e._tcp` (názov hostiteľa `daq-e-<id>.local`) |**DAQ-E-S (novinka vo verzii 1.3.0).** DAQ-E s pevným difúzorom a senzorom náklonu. V zozname Chlorosu je uvedený ako `DAQ-E-S` a funguje presne ako DAQ-E, s tým rozdielom, že nevyžaduje korekciu krytu a každé meranie obsahuje údaje o náklone, otáčaní a sklonu zariadenia. Pozri [DAQ-E-S: pevný difúzor a náklon](caps-and-range.md#daq-e-s-fixed-diffuser-and-tilt).

Všetky tri zariadenia používajú rovnaký komunikačný protokol a poskytujú identické údaje:

* **135-bodové spektrum v rozsahu od 340 do 1010 nm s krokom 5 nm**, plus tristimulačné hodnoty CIE XYZ, v každom snímku.
* **Radiometricky kalibrovaná spektrálna intenzita žiarenia v W/m²/nm** — predtým, ako sa údaje dostanú k vám, sa na ne uplatní súbor továrenských kalibrácií každého zariadenia (spolu s jeho aktívnym profilom korekcie krytu).
* Rovnaký **formát záznamu `.daq`** (súbor SQLite). Ďalšie spracovanie je identické bez ohľadu na to, ktorý prenosový kanál súbor vytvoril.

Prenosové protokoly (sériové USB, BLE, mDNS/zeroconf) sú integrované v backende Chloros — na komunikáciu s ktorýmkoľvek z týchto troch modelov prostredníctvom grafického rozhrania alebo príkazov `pool-*` v CLI nie je potrebné nič inštalovať.

***

## Kalibrovaný rozsah: hlásený 340–1010 nm, kalibrovaný ~374–974 nm

Senzor hlási celý rozsah 340–1010 nm, ale radiometrický zisk sledovateľný podľa NIST sa rozprestiera približne v rozmedzí **374–974 nm**. Chloros odmieta rozdelenie podľa absolútnej odrazivosti pre akékoľvek pásmo kamery, ktorého menej ako polovica spektrálnej váhy leží v rámci tohto kalibrovaného rozsahu; vynechané pásmo je hlásené s dôvodom vynechania `dls-uncalibrated-band-<nm>`.

Zo všetkých dodávaných modelov filtrov LATTICE sa to týka iba modelu **F988**:

Odrazivosť modelu F988 je kalibrovaná pomocou panelu odrazivosti umiestneného v scéne: pásmo leží mimo kalibrovaného rozsahu svetelného senzora DAQ, preto Chloros použije váš najnovší záznam z panelu a zachová ho medzi jednotlivými meraniami panelu.

Ak sa záznam z modelu F988 spracováva iba s dostupnými údajmi z DAQ, Chloros odmietne odrazivosť založenú na DAQ pre dané pásmo s dôvodom vynechania `dls-uncalibrated-band-988` — podporovaným postupom pre F988 je [pracovný postup s panelom odrazivosti](../calibration-targets.md).

***

## ID senzorov

Každý DAQ hlási stabilné ID senzora. Jeho formát sa líši podľa modelu:

| Model | Formát ID | Príklad |
| --- | --- | --- |
| DAQ-U | 5 oktetov s pomlčkami | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 oktetov s pomlčkami | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

ID senzora je:

* vtlačené do každého súboru `.daq`, ktorý zaznamenáva,
* kľúč, ktorý Chloros používa na načítanie balíka továrenských kalibrácií danej jednotky,
* hodnota, ktorú odovzdávate `--sensor-id` v príkazoch CLI, a
* v prípade DAQ-E je to aj jeho mDNS hostname (`daq-e-def330.local`) — hodnota, ktorú `--eth-host` akceptuje.

***

## Továrenská kalibrácia a cloud

Každá jednotka DAQ je individuálne kalibrovaná vo výrobe pomocou radiometrického reťazca sledovateľného podľa NIST a spoločnosť Chloros načíta kalibračný balík každej jednotky na základe jej ID senzora. Kalibračnú správu pre jednotlivé jednotky (PDF) je možné stiahnuť z nastavení senzora na [karte Svetelné senzory](gui.md).

{% hint style="warning" %}
**Modely DAQ-U a DAQ-M vyžadujú na kalibráciu prístup do cloudu.**Ani jeden z týchto modelov neukladá žiadne údaje do vnútornej pamäte: ich továrenské kalibračné súbory sa nachádzajú v cloude spoločnosti MAPIR a sú načítané na základe ID senzora (následne sa uložia do lokálnej vyrovnávacej pamäte). Chloros potrebuje pripojenie k internetu, aby mohol odosielať kalibrované údaje v jednotkách W/m²/nm zo zariadenia DAQ-U alebo DAQ-M.**Výnimkou je DAQ-E** — ten má kalibráciu uloženú priamo v zariadení.
{% endhint %}
<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

***

## Kam sa ukladajú záznamy

| Povrch | Predvolené miesto uloženia `.daq` |
| --- | --- |
| Grafické rozhranie — karta Svetelné senzory | `<project folder>/light_sensor/` (hotové záznamy sa automaticky pridávajú do projektu) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` na počítači, na ktorom beží backend (relatívna cesta `-o` sa vyrieši voči priečinku, v ktorom ste spustili príkaz, ak je backend na tom istom počítači) |

Každý názov súboru `.daq` obsahuje ID senzora a časovú pečiatku.

***

## V tejto kapitole

* [**Karta DAQ v programe Chloros**](gui.md) — kompletný návod na používanie grafického rozhrania: pripojenie jednotlivých modelov, nastavenia pre každý senzor, spektrálne grafy, kolorimetrické údaje v reálnom čase, odrazivosť dvoch senzorov a záznam.
* [**Rýchly štart CLI (pool-\*)**](cli-quick-start.md) — ovládanie senzorov DAQ z `chloros-cli daq pool-*`, podporovaná cesta z príkazového riadku.
* [**Profily obmedzení a kalibrovaný rozsah**](caps-and-range.md) — aké obmedzenia existujú pre jednotlivé modely, ako ich deklarovať a podrobnosti o kalibrovanom spektrálnom rozsahu.
* [**Záznam a formát .daq**](recording.md) — formát SQLite `.daq` a pracovné postupy pri zázname.
* [**Sieťové pripojenie DAQ-E a synchronizácia času**](ethernet-ptp.md) — prenosové režimy DAQ-E a synchronizácia času PTP.
* [**Pracovné postupy pre odrazivosť**](reflectance.md) — využitie údajov o dopadajúcom žiarení z DAQ na výpočet odrazivosti.
* Kompletnú dokumentáciu na úrovni príkazových vlajok nájdete v [Referencii CLI](../reference/cli-reference.md) (sekcia `chloros-cli daq`) a v [Referencii SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), ktoré sú napísané tak, aby ich mohli priamo využívať AI asistenti.

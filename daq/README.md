# Svetelné senzory DAQ

> **Hľadáte informácie o hardvéri?**Samotné senzory – modely, spôsob montáže, kryty, porty, napájanie a aplikácia SCANNER – sú popísané v**[užívateľskej príručke DAQ](https://mapir.gitbook.io/daq)**. Táto kapitola sa zaoberá ich používaním od verzie Chloros.

Svetelné senzory **DAQ** z MAPIR merajú okolité osvetlenie vo forme radiometricky kalibrovaných spektier. V Chloros plnia dve úlohy:

* **Samostatný spektrálny prístroj** — grafy spektra v reálnom čase, kolorimetrické údaje a záznamy z modelu `.daq`, všetko z [kartu Svetelné senzory](gui.md), [CLI](cli-quick-start.md) alebo z Python a SDK.
* **Zdroj ožiarenia smerujúceho nadol pre odrazivosť** — počas spracovania Chloros interpoluje vaše hodnoty z `.daq` na časovú pečiatku expozície každého záberučasovej pečiatky expozície a pomocou nameraného dopadajúceho svetla prevádza žiarivosť kamery na odrazivosť (`--reflectance-source daq`), pre kalibrované pásma nie je potrebný panel v scéne.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Tri modely, jeden formát údajov

| Model | Prenos | Vyhľadávanie |
| --- | --- | --- |
| **DAQ-U** | USB (sériové) | vyhľadávanie cez sériový port |
| **DAQ-M** | Bluetooth Low Energy | vyhľadávanie BLE podľa názvu |
| **DAQ-E** | Ethernet (IPv4, napájanie cez PoE) | mDNS `_daq-e._tcp` (názov hostiteľa `daq-e-<id>.local`) |

Všetky tri zariadenia používajú rovnaký komunikačný protokol a poskytujú identické údaje:

* **135-bodové spektrum v rozsahu od 340 do 1010 nm s krokom 5 nm**, plus tristimulačné hodnoty CIE XYZ, v každom snímku.
* **Radiometricky kalibrovaná spektrálna intenzita žiarenia v W/m²/nm** — predtým, ako sa údaje dostanú k vám, sa na ne aplikuje balík továrenskej kalibrácie každého zariadenia (vrátane jeho aktívneho profilu korekcie krytu).
* Rovnaký **formát záznamu `.daq`** (súbor SQLite). Ďalšie spracovanie je identické bez ohľadu na to, ktorý prenosový kanál súbor vytvoril.

Prenosové vrstvy (sériové USB, BLE, mDNS/zeroconf) sú zahrnuté v rámci backendu Chloros — na komunikáciu s ktorýmkoľvek z týchto troch modelov prostredníctvom grafického rozhrania alebo príkazov CLI v `pool-*` nie je potrebné nič inštalovať.

***

## Kalibrovaný rozsah: hlásený 340–1010 nm, kalibrovaný ~374–974 nm

Senzor hlási celý rozsah 340–1010 nm, ale radiometrický zisk sledovateľný podľa NIST sa rozprestiera približne v rozmedzí **374–974 nm**. Chloros odmieta delenie absolútnej odrazivosti pre akékoľvek pásmo kamery, ktorého menej ako polovica spektrálnej váhy sa nachádza v rámci tohto kalibrovaného rozsahu; vynechané pásmo sa hlási s dôvodom vynechania `dls-uncalibrated-band-<nm>`.

Zo všetkých dodávaných filtrov LATTICE sa to týka iba modelu **F988**:

Odrazivosť F988 je kalibrovaná pomocou panelu odrazivosti v scéne: pásmo leží mimo kalibrovaného rozsahu svetelného senzora DAQ, takže Chloros použije váš najnovší záznam z panelu a zachová ho medzi jednotlivými meraniami panelu.

Ak sa snímanie modelu F988 spracováva len s dostupnými údajmi DAQ, kód Chloros odmietne odrazivosť založenú na DAQ pre dané pásmo s dôvodom preskočenia `dls-uncalibrated-band-988` — podporovanou cestou pre model F988 je [pracovný postup s panelom odrazivosti](../calibration-targets.md) je podporovaná cesta pre F988.

***

## ID senzorov

Každé DAQ hlási stabilné ID senzora. Jeho tvar sa líši podľa modelu:

| Model | Tvar ID | Príklad |
| --- | --- | --- |
| DAQ-U | 5 oktetov s pomlčkami | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-oktetový s pomlčkami | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

ID senzora je:

* vtlačené do každého súboru `.daq`, ktorý zaznamenáva,
* kľúč, ktorý Chloros používa na načítanie balíka továrenskej kalibrácie danej jednotky,
* hodnota, ktorú odovzdávate do `--sensor-id` v príkazoch CLI a `pool-*`, a
* v prípade zariadenia DAQ-E aj jeho mDNS hostname (`daq-e-def330.local`) — hodnotu, ktorú prijíma `--eth-host`.

***

## Továrenská kalibrácia a cloud

Každá jednotka DAQ je individuálne kalibrovaná vo výrobe pomocou radiometrického reťazca sledovateľného podľa NIST a Chloros načíta kalibračný balík každej jednotky podľa jej ID senzora. Kalibračnú správu pre jednotlivé jednotky (PDF) je možné stiahnuť z nastavení senzora na [karte Svetelné senzory](gui.md).

{% hint style="warning" %}
**Modely DAQ-U a DAQ-M vyžadujú na kalibráciu prístup do cloudu.**Ani jeden z týchto modelov neukladá žiadne údaje do vnútornej pamäte: ich továrenské kalibračné súbory sa nachádzajú v cloude MAPIR a sú načítané na základe ID senzora (následne sa uložia do lokálnej vyrovnávacej pamäte). Chloros potrebuje pripojenie k internetu, aby mohol odosielať kalibrované údaje v jednotkách W/m²/nm zo zariadenia DAQ-U alebo DAQ-M.**Výnimkou je DAQ-E** — kalibráciu má uloženú priamo v zariadení.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## Kam sa ukladajú záznamy

| Povrch | Predvolené umiestnenie v `.daq` |
| --- | --- |
| Grafické rozhranie — karta Svetelné senzory | `<project folder>/light_sensor/` (dokončené záznamy sa automaticky pridávajú do projektu) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` na počítači, na ktorom beží backend |

Každý názov súboru `.daq` obsahuje ID senzora a časovú pečiatku.

***

## V tejto kapitole

* [**Karta DAQ v Chloros**](gui.md) — kompletný návod na používanie grafického rozhrania: pripojenie jednotlivých modelov, nastavenia pre každý senzor, spektrálne grafy, kolorimetrické údaje v reálnom čase, odrazivosť dvoch senzorov a zaznamenávanie.
* [**Rýchly štart CLI (pool-\*)**](cli-quick-start.md) — ovládanie senzorov DAQ z programu `chloros-cli daq pool-*`, podporovaná cesta z príkazového riadku.
* [**Profily limitov a kalibrovaný rozsah**](caps-and-range.md) — aké limity existujú pre jednotlivé modely, ako ich deklarovať a podrobnosti o kalibrovanom spektrálnom rozsahu.
* [**Záznam a formát .daq**](recording.md) — formát SQLite `.daq` a pracovné postupy pri zázname.
* [**Sieťové pripojenie DAQ-E a synchronizácia času**](ethernet-ptp.md) — režimy prenosu DAQ-E a synchronizácia času PTP.
* [**Pracovné postupy pre odrazivosť**](reflectance.md) — využitie údajov o dopadajúcom žiarení z DAQ na výpočet odrazivosti.
* Kompletnú dokumentáciu na úrovni príznakov nájdete v [referenčnom materiáli CLI](../reference/cli-reference.md) (časť `chloros-cli daq`) a v [Referenciu k SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), ktoré sú napísané tak, aby ich mohli priamo využívať asistenti umelej inteligencie.

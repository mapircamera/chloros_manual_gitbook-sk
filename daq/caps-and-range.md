# Profily krytov a kalibrovaný rozsah

> Samotné kryty — ktoré kryty sa dodávajú s ktorými senzormi, ako sa montujú a aké majú optické vlastnosti — sú zdokumentované v **[užívateľskej príručke DAQ](https://mapir.gitbook.io/daq)**. Táto stránka sa zaoberá *deklarovaním* namontovaného krytu pre systém Chloros
, čo je nevyhnutné na správnu korekciu.

Všetky továrenské radiometrické kalibrácie svetelných senzorov DAQ opisujú *holý* senzor. Fyzický kryt nasadený na difúzor mení svetlo, ktoré senzor zachytáva, preto Chloros
aplikuje na kalibračný balík továrensky nameraný **profil korekcie krytu**. Nahlásenie správneho krytu je súčasťou získavania kalibrovaných údajov — táto stránka popisuje, aké kryty existujú pre jednotlivé modely, ako ich nahlásiť a aký je skutočný kalibrovaný spektrálny rozsah senzora.

## Dostupnosť krytiek podľa modelu

| Profil krytky (`cap_id`) | Fyzická krytka | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Kryt s korekciou kosínusového efektu slnečného žiarenia (**štandardne na každom modeli**) | Áno | Áno | Áno |
| `fov_15` / `fov_45` / `fov_90` | Kónusy obmedzujúce zorné pole (15° / 45° / 90°) | Áno | — | Áno |
| `fov_30` / `fov_60` | Kónusy obmedzujúce zorné pole (30° / 60°) | Áno | — | — |
| `none` | Bez nasadeného krytu | — | — | Áno |

Poznámky špecifické pre jednotlivé modely:

* **DAQ-M má jeden profil krytu: `sunshine_cosine`.** „Bare-plus-Sunshine-cap“ je jeho produktová definícia a holý DAQ-M nepotrebuje žiadny geometrický profil.
* **Holý DAQ-U je skutočne holý** — nepotrebuje žiadny geometrický profil, preto pre neho neexistuje profil `none`.
* **`none` na DAQ-E NIE JE neaktívny.** Zapustený difúzor DAQ-E pokrytý sklom má vlastnú skutočnú geometrickú korekciu, takže „bez krytu“ je na tomto modeli sám o sebe meraným profilom.
* **Holý DAQ-E nedokáže merať priame slnečné žiarenie pri žiadnej výške** — kryt Sunshine je konfiguráciou pre terénne použitie. Neplánujte prácu vonku s holým DAQ-E.

V nastaveniach jednotlivých senzorov v grafickom rozhraní (ikona ozubeného kolesa na karte Svetelné senzory) ponúka roletové menu **Kryt** na zariadeniach DAQ-U a DAQ-M aj možnosť „Žiadny (nezakrytý senzor)“ — na týchto dvoch modeloch „nezakrytý“ jednoducho znamená, že sa neuplatňuje žiadna korekcia krytu, ako je uvedené v poznámkach vyššie. Túto možnosť vyberte len vtedy, ak je kryt fyzicky odstránený.

## Deklarovanie krytu — a prečo je to dôležité

**Deklarované označenie `cap_id` sa musí zhodovať s krytom, ktorý je fyzicky nasadený na senzore.** Ani senzor, ani softvér nedokážu detekovať nasadený kryt. Deklarácia ovplyvňuje dve veci:

1. **Korekciu v reálnom čase**, ktorá sa uplatňuje na každé spektrum.
2. **Označenie krytu zapísané do každého záznamu `.daq`**, na ktoré sa spolieha následné spracovanie odrazivosti.

Kryt Sunshine má **konštrukčne stanovený útlm približne 12×**, takže záznam s nesprávne deklarovaným krytom spôsobí nesprávne škálovanie spektier približne o tento faktor. Zmeny krytu deklarujte ihneď.

### Nastavenie krytu

GUI: Karta Svetelné senzory → ikona ozubeného kolesa v riadku senzora → roletové menu **Kryt**. Predvolenou hodnotou pre každý model je `sunshine_cosine`, čo je predpoklad (modely DAQ-U, DAQ-M a DAQ-E sa dodávajú bez krytu a záznam uvádza, že kryt bol predpokladaný, kým si nevyberiete iný), a tento výber zostáva zachovaný v rámci projektu.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(backend musí bežať):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI
syntakticky akceptuje celý zoznam `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); každý profil sa pri pripojení overuje voči modelu senzora, takže nedostupné ID obmedzenia (napríklad ID určené len pre E na DAQ-U) vyvolá jasnú chybu namiesto nesprávnej korekcie. Predvolená hodnota backendu, ak sa nič neprevedie, je `sunshine_cosine`.

Python
SDK
: pri pripájaní pomenujte kapu pomocou `connect_daq_sensor(cap_id="sunshine_cosine")` alebo ju počas behu programu vymeňte pomocou `DAQSensorSession.set_cap("fov_45")`; pozrite si [Referenčnú príručku SDK
](../reference/sdk-reference.md#daq-sensor-sessions).

Pokročilé: profily sú súčasťou inštalácie Chloros
pod názvom `daq/cap_profiles/<u|m|e>/<cap_id>.json` a používateľ ich môže prekonfigurovať na `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Okrem limitov senzory, ktoré nikdy neboli prekalibrované, automaticky dostávajú malé vylepšenie tmavého posunu odvodené z flotily – bez akejkoľvek akcie zo strany používateľa.

## DAQ-E-S: pevný difúzor a naklonenie

**DAQ-E-S** je zariadenie DAQ-E s dvoma rozdielmi, ktoré sú pre Chloros
dôležité. Všetko ostatné (vyhľadávanie, pripájanie, PTP, zaznamenávanie, nastavenia zariadenia) funguje presne tak ako pri zariadení DAQ-E.

* **Jeho difúzor je pevne namontovaný, takže nevyžaduje žiadnu korekciu pomocou krytky.** Difúzor bol na zariadení už v čase merania jeho továrenskej kalibrácie, takže táto kalibrácia už udáva skutočné ožiarenie smerom nadol. Chloros
naň neuplatňuje žiadny profil obmedzenia, bez ohľadu na to, čo sa požaduje: obmedzenie pomenované pre tento difúzor sa ignoruje so správou, jeho záznamy uvádzajú, že obmedzenie pochádza z modelu, a varovanie o predpokladanom obmedzení sa v jeho prípade nikdy nespustí. V nastaveniach projektu → Svetelný senzor DAQ sa v jeho riadku nezobrazuje žiadny limit a celoprojektové prepisovanie limitu sa naň nevzťahuje. Neuplatňujte na jeho údaje limit ručne: korekcia slnečného žiarenia by v takom prípade vykazovala približne 11-násobne vyššiu hodnotu.
* **Je vybavený 3-osovým akcelerometrom.** Kosínusová charakteristika difúzora platí len vtedy, keď je v horizontálnej polohe, takže naklonený senzor hlási nesprávnu intenzitu žiarenia bez akýchkoľvek ďalších príznakov. Každé meranie DAQ-E-S preto obsahuje údaje o naklonení, otáčaní a sklonu, a záznam `.daq` z jedného z nich ich ukladá vedľa každého spektra (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), takže môžete rozlíšiť oblak od narazeného držiaka.

Informácie o každom poli polohy a o tom, ako sa hlási chyba v dátovom toku polohy, nájdete v [Referencii CLI
pre DAQ-E-S](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Výkon slnečnej clony (konfigurácia pre vonkajšie použitie)

Čísla, na ktorých môžete stavať postupy:

| Vlastnosť | Hodnota |
| --- | --- |
| Zorné pole | 180° hemisférické |
| Chyba kosínusovej odozvy | ≤ ±4 % pri uhle dopadu do 60°; ≤ ±4,5 % pri uhle dopadu do 70° |
| Limit pri nízkom slnku | Neodporúča sa pri výške slnka pod ~15° |
| Útlm | ~12× (podľa konštrukcie) |
| Opakovateľnosť opätovnej montáže krytu | ≈ 1,5 % |
| Kvantitatívna intenzita žiarenia | Priemerné hodnoty z meraní trvajúcich **≥ 15 s** (charakteristika prístroja, nie je to chyba) |

Pre akúkoľvek hodnotu kvantitatívnej intenzity žiarenia – vrátane referenčných hodnôt odrazivosti – použite priemernú hodnotu z meraní trvajúcich aspoň 15 sekúnd namiesto jedného snímku.

## Kalibrovaný spektrálny rozsah

| Vlastnosť | Hodnota |
| --- | --- |
| Spektrálne vzorkovanie | 340–1010 nm v krokoch po 5 nm (135 bodov) |
| Radiometricky kalibrovaný rozsah | **~374–974 nm** (vynútené softvérom) |

Senzor zaznamenáva celý rozsah 340–1010 nm, avšak radiometrický zisk sledovateľný podľa NIST sa rozprestiera v rozsahu ~374–974 nm. Chloros
**odmieta rozdelenie podľa absolútnej odrazivosti** pre akékoľvek pásmo kamery, ktorého menej ako polovica spektrálnej váhy spadá do tohto rozsahu, a namiesto vytvorenia nekalibrovaného výstupu hlási dôvod preskočenia `dls-uncalibrated-band-<nm>`. Zo všetkých dodávaných modelov kamier sa mimo tohto rozsahu nachádza iba filter F988; ten namiesto toho využíva pracovný postup s panelom odrazivosti — pozri [Pracovné postupy s odrazivosťou](reflectance.md).

Informácie o modeloch snímačov, transportoch a identifikátoroch snímačov nájdete v [prehľade DAQ](README.md). Informácie o tom, ako sa počas spracovania spotrebúva limit, nájdete v [Záznam a formát .daq](recording.md).

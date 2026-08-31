# Profily krytiek a kalibrovaný rozsah

> Samotné krytky – informácie o tom, ktorá krytka sa dodáva s ktorým senzorom, ako sa montujú a aké sú ich optické vlastnosti – sú uvedené v **[užívateľskej príručke k DAQ](https://mapir.gitbook.io/daq)**. Táto stránka sa zaoberá *deklarovaním* namontovaného krytu v Chloros, čo je podmienkou správnej korekcie.

Všetky továrenské radiometrické kalibrácie svetelných senzorov DAQ opisujú *holý* senzor. Fyzický kryt namontovaný na difúzore mení svetlo, ktoré senzor zachytáva, takže Chloros aplikuje **profil korekcie krytu** nameraný vo výrobe na kalibračný balík. Deklarovanie správneho krytu je súčasťou získavania kalibrovaných údajov – táto stránka popisuje, aké kryty existujú pre jednotlivé modely, ako ich deklarovať a aký je skutočný kalibrovaný spektrálny rozsah senzora.

## Dostupnosť krytiek podľa modelu

| Profil krytky (`cap_id`) | Fyzická krytka | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Kryt s korektorom kosínusového efektu slnečného žiarenia (**štandardne na každom modeli**) | Áno | Áno | Áno |
| `fov_15` / `fov_45` / `fov_90` | Kónusy obmedzujúce zorné pole (15° / 45° / 90°) | Áno | — | Áno |
| `fov_30` / `fov_60` | Kužele obmedzujúce zorné pole (30° / 60°) | Áno | — | — |
| `none` | Bez nasadeného krytu | — | — | Áno |

Poznámky k jednotlivým modelom:

* **DAQ-M má jeden profil krytu: `sunshine_cosine`.** „Bare-plus-Sunshine-cap“ je jeho produktová definícia a holý DAQ-M nepotrebuje žiadny geometrický profil.
* **Holý DAQ-U je skutočne holý** — nepotrebuje žiadny geometrický profil, preto preň neexistuje profil `none`.
* **`none` na DAQ-E NIE JE neaktívny.** Zapustený difúzor DAQ-E pokrytý sklom má vlastnú skutočnú geometrickú korekciu, takže „bez krytu“ je na tomto modeli sám o sebe meraným profilom.
* **Holý DAQ-E nedokáže merať priame slnečné žiarenie pri žiadnej výške** — kryt Sunshine je konfiguráciou pre terénne použitie. Neplánujte prácu vonku s holým DAQ-E.

V nastaveniach jednotlivých senzorov v grafickom rozhraní (ikona ozubeného kolesa na karte Svetelné senzory) ponúka roletové menu **Kryt** na zariadeniach DAQ-U a DAQ-M aj možnosť „Žiadny (nezakrytý senzor)“ — na týchto dvoch modeloch „nezakrytý“ jednoducho znamená, že sa neuplatňuje žiadna korekcia krytu, ako je uvedené v poznámkach vyššie. Túto možnosť vyberte len vtedy, ak je kryt fyzicky odstránený.

## Deklarovanie krytu — a prečo je to dôležité

**Deklarované označenie `cap_id` sa musí zhodovať s krytom, ktorý je fyzicky nasadený na senzore.** Ani senzor, ani softvér nedokážu detekovať nasadený kryt. Deklarácia ovplyvňuje dve veci:

1. **Korekciu v reálnom čase**, ktorá sa uplatňuje na každé spektrum.
2. **Označenie krytu zapísané do každého záznamu `.daq`**, na ktoré sa spolieha následné spracovanie odrazivosti.

Kryt Sunshine má **konštrukčne stanovený útlm približne 12×**, takže záznam s nesprávne deklarovaným krytom spôsobuje nesprávne škálovanie spektier približne o tento faktor. Zmeny krytu deklarujte ihneď.

### Nastavenie krytu

GUI: Karta Svetelné senzory → ikona ozubeného kolesa v riadku senzora → roletové menu **Kryt**. Predvoleným nastavením pre každý model je `sunshine_cosine` (všetky senzory DAQ sa dodávajú s nainštalovaným kosínusovým korektorom) a tento výber zostáva zachovaný v rámci projektu.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (backend musí bežať):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

Model CLI syntakticky akceptuje celý zoznam `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); každý profil sa pri pripojení overuje voči modelu senzora, takže nedostupné ID snímača (napríklad ID určené len pre E na zariadení DAQ-U) vyvolá jasnú chybu namiesto nesprávnej korekcie. Predvolené nastavenie backendu, ak sa nič neprevedie, je `sunshine_cosine`.

Python SDK poznámka: `cap_id` **nie** je ovládačom typu SDK — ovládače `connect_daq_sensor()` / `DAQSensorSession` neposkytujú žiadny parameter pre kondenzátor. Kapacitu vyberte pomocou vyššie uvedených príkazov CLI alebo z roletového menu v grafickom rozhraní; pozrite si [Referenciu SDK](../reference/sdk-reference.md).

Pokročilé: profily sú súčasťou inštalácie Chloros na adrese `daq/cap_profiles/<u|m|e>/<cap_id>.json` a je možné ich pre každého používateľa prepisovať na adrese `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Okrem limitov sa senzorom, ktoré nikdy neboli prekalibrované, automaticky priradí malé vylepšenie tmavého posunu odvodené z flotily – bez akéhokoľvek zásahu používateľa.

## Výkon slnečného limitu (konfigurácia pre vonkajšie použitie)

Čísla, na ktorých môžete stavať postupy:

| Vlastnosť | Hodnota |
| --- | --- |
| Zorné pole | 180° hemisférické |
| Chyba kosínusovej odozvy | ≤ ±4 % pri dopade do 60°; ≤ ±4,5 % pri dopade do 70° |
| Limit pri nízkom slnku | Neodporúča sa pod ~15° výškou slnka |
| Útlm | ~12× (podľa konštrukcie) |
| Opakovateľnosť opätovnej montáže krytu | ≈ 1,5 % |
| Kvantitatívna intenzita žiarenia | Priemerná hodnota z meraní trvajúcich **≥ 15 s** (charakteristika prístroja, nie je to chyba) |

Pre akúkoľvek hodnotu kvantitatívnej intenzity žiarenia – vrátane referenčných hodnôt odrazivosti – použite priemernú hodnotu z meraní trvajúcich aspoň 15 sekúnd namiesto jedného snímku.

## Kalibrovaný spektrálny rozsah

| Vlastnosť | Hodnota |
| --- | --- |
| Spektrálne vzorkovanie | 340–1010 nm v krokoch po 5 nm (135 bodov) |
| Radiometricky kalibrovaný rozsah | **~374–974 nm** (vynútené softvérom) |

Senzor zaznamenáva celú mriežku 340–1010 nm, ale radiometrický zisk sledovateľný podľa NIST sa rozprestiera v rozsahu ~374–974 nm. Chloros **odmieta rozdelenie podľa absolútnej odrazivosti** pre akékoľvek spektrálne pásmo kamery, ktorého menej ako polovica spektrálnej váhy spadá do tohto rozsahu, pričom namiesto vytvorenia nekalibrovaného výstupu nahlási dôvod vynechania `dls-uncalibrated-band-<nm>`. Zo všetkých dodávaných modelov kamier sa mimo tohto rozsahu nachádza iba filter F988; ten namiesto toho využíva pracovný postup s panelom odrazivosti – pozri [Pracovné postupy s odrazivosťou](reflectance.md).

Informácie o modeloch snímačov, transportoch a identifikátoroch snímačov nájdete v [prehľade DAQ](README.md). Informácie o tom, ako sa počas spracovania spotrebúva limit, nájdete v [Záznam a formát .daq](recording.md).

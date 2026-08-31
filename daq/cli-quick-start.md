# CLI Rýchly štart (pool-*)

Dodávané zariadenie `chloros-cli` ovláda senzory DAQ prostredníctvom rodiny príkazov **`daq pool-*`** — tenké klienty HTTP, ktoré ovládajú senzor prostredníctvom trvalého poola senzorov backendu Chloros. Backend riadi prenos, takže grafické rozhranie (GUI), skripty CLI a SDK zdieľajú jednu aktívnu referenciu namiesto toho, aby sa navzájom súperili o port. Všetko, čo zákazník potrebuje, je dostupné prostredníctvom `pool-*`: pripojenie, prenos dát, záznam kalibrovaných súborov `.daq` a výmena profilov snímačov.

`pool-*` je tiež **jediným** rozhraním DAQ vo vydaných zostavách. `chloros-cli daq --help` vypisuje podpríkazy `pool-*`, a vyvolanie podpríkazu DAQ pre priamy hardvér v dodanej verzii skončí výslovnou chybou, ktorá uvádza chýbajúci balík a odkazuje späť na `pool-*` — nič sa neprevalí bez upozornenia. (Príkazy pre priamy hardvér bežia iba zo zdrojového kódu MAPIR; ani `pip install chloros-sdk` ich neposkytuje.)

***

## Predpoklady

* **Backend Chloros musí bežať** — príkazy `pool-*` sú klientmi HTTP, nie hardvérovými ovládačmi. V systéme Windows spustite desktopovú aplikáciu Chloros (tá spustí backend). Na zariadení Linux/Jetson bez grafického rozhrania aktivujte službu: `sudo systemctl enable --now chloros-backend.service`.
* **Prihlásenie na Chloros+ (platená úroveň)**: najprv spustite `chloros-cli login`. Vynútenie prebieha na strane servera — bez prihlásenia príkazy zlyhajú s chybou `401 AUTH_REQUIRED`; v bezplatnej verzii (Iron) zlyhajú s chybou `403 PLAN_UPGRADE_REQUIRED`.
* Príkazy sú štandardne určené pre `http://127.0.0.1:5000`; rodina `daq pool-*` rešpektuje premennú prostredia `CHLOROS_BACKEND_URL`, ak váš backend beží inde.

***

## Päťminútová relácia

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — otvorenie senzora v skupine

| Varianta | Význam |
| --- | --- |
| `daq pool-connect` | Inteligentná detekcia: nájsť akékoľvek zariadenie DAQ na tomto počítači. |
| `daq pool-connect --port PORT` | DAQ-U na konkrétnom sériovom porte (napr. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M cez BLE, automatické vyhľadávanie MAC. |
| `daq pool-connect --mac MAC` | DAQ-M na známej adrese BLE MAC (predpokladá `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E so známym názvom hostiteľa alebo IP adresou — **spoľahlivá cesta**. |
| `daq pool-connect --eth` | DAQ-E s automatickým vyhľadávaním (mDNS, s náhradným riešením ARP). Pozrite si upozornenie nižšie. |

Nastavovacie príznaky, všetky voliteľné:

| Príznak | Význam |
| --- | --- |
| `--integration-time MS` / `-t MS` | Ručne nastavený integračný čas v milisekundách. |
| `--frame-avg N` / `-f N` | Priemerný počet snímok na hlásené spektrum. |
| `--no-ae` | Vypnúť automatickú expozíciu (AE je štandardne zapnutá). |
| `--no-stream` | Pripojiť bez spustenia prenosu (neskôr pokračovať pomocou `pool-stream --start`). |
| `--cap-id CAP` | Profil korekcie stropu; predvolené nastavenie backendu je `sunshine_cosine`. Pozri [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**Upozornenie týkajúce sa automatického vyhľadávania `--eth`.** Na hostiteľovi s viacerými sieťovými rozhraniami (viac ako jedno aktívne sieťové rozhranie) môže byť *prvý* príkaz `pool-connect --eth` po spustení systému prázdny, aj keď je senzor v poriadku — vyhľadávanie môže prehliadnuť rozhranie senzora, pokiaľ je cache ARP prázdna. Ak `--eth` nič nenájde, skúste to znova alebo vyhľadávanie úplne preskočte pomocou `--eth-host <ip-or-hostname>`, čo je spoľahlivý postup na zariadeniach s viacerými sieťovými rozhraniami. Názov hostiteľa DAQ-E je `daq-e-<id>.local` (napr. `daq-e-def330.local`); funguje aj jeho číselná IP adresa.
{% endhint %}

## `pool-list` — zobrazenie pripojených zariadení

Zobrazí všetky senzory v skupine backendov, vrátane `sensor_id`, ktoré potrebujú všetky ostatné príkazy:

| Model | Formát `sensor_id` | Príklad |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktetový s pomlčkami | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — čítanie rámcov spektra

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Vráti najnovší rámec alebo najnovšie rámce; `--recent N`; `--json` generuje strojovo čitateľný výstup pre skriptovanie. Rámce predstavujú radiometricky kalibrované spektrálne ožiarenie (W/m²/nm) na 135-bodovej mriežke v rozsahu 340–1010 nm, pričom je už aplikovaný profil krytu senzora. Pre kvantitatívne hodnoty ožiarenia je potrebné získať priemer z rámcov trvajúcich aspoň 15 sekúnd — ide o charakteristiku prístroja, nie o chybu.

## `pool-stream` — pozastavenie alebo obnovenie prenosu

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — zaznamenajte súbor `.daq`

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Príznak | Predvolené | Význam |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Dĺžka nahrávania v sekundách; `0` znamená, že sa bude vykonávať, kým nezadáte `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Výstupný adresár, určený **na počítači, na ktorom beží backend**. |
| `--device-name NAME` | — | Štítok uložený spolu so záznamom. |
| `--stop` | — | Zastavenie prebiehajúceho záznamu. |

{% hint style="info" %}
Nahrávanie prebieha v backende, takže súbor `.daq` sa ukladá do súborového systému **stroja s backendom** — štandardne do adresára `~/Documents/DAQ Live View/`, nie nevyhnutne tam, kde ste spustili program CLI. Názvy súborov obsahujú ID senzora a časovú pečiatku.
{% endhint %}

## `pool-set-cap` — deklarácia nasadeného krytu

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

ID krytu určuje výrobcom nameraný korekčný profil, ktorý sa aplikuje na každé spektrum, a **musí zodpovedať krytu fyzicky nasadenému na senzore** — ani senzor, ani softvér nedokážu kryt samy detekovať a tento výber sa zaznamenáva do každého súboru `.daq`. Všetko je štandardne nastavené na `sunshine_cosine` (každý DAQ sa dodáva s nainštalovaným krytom Sunshine s kosínusovou korekciou, konštrukčne s útlmom ~12× — nedeklarovaná zmena krytu nesprávne koriguje spektrá približne o tento faktor).

| `--cap-id` | K dispozícii na |
| --- | --- |
| `sunshine_cosine` (predvolené) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | iba DAQ-U |
| `none` | iba DAQ-E — pozri poznámku |

Identifikátor krytu, ktorý nie je súčasťou sady senzora, je pri pripojení odmietnutý s jasnou chybou. `none` (DAQ-E) znamená, že kryt je fyzicky odstránený — na zapustený sklenený difúzor zariadenia DAQ-E sa stále uplatňuje profil geometrie z výroby, takže nejde o nefunkčnú konfiguráciu, a zariadenie DAQ-E bez krytu je laboratórnou konfiguráciou, nie podporovanou konfiguráciou pre použitie v teréne. (Holý DAQ-U je skutočne holý a nepotrebuje žiadny korekčný profil; DAQ-M sa používa so svojím krytom Sunshine.)

## `pool-disconnect` — uvoľnenie senzorov

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Prehľad príkazov

| Príkaz | Účel |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Otvoriť senzor v skupine backendov. |
| `daq pool-list` | Zobraziť všetky senzory v skupine spolu s ich `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Posledných N kalibrovaných spektrálnych snímok. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Obnovenie / pozastavenie streamovania. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Spustenie / zastavenie záznamu `.daq` (na strane backendu). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Zmena profilu korekcie obmedzenia počas behu. |
| `daq pool-disconnect --sensor-id ID [--all]` | Uvoľnenie jedného senzora alebo všetkých. |

***

## Riešenie problémov pri prvom pripojení DAQ-E

1. DAQ-E nemá stavovú LED – skontrolujte napájanie pomocou indikátora PoE/link na prepínači alebo porte injektora a po zapnutí počkajte niekoľko sekúnd, kým sa zariadenie naštartuje a pripojí k sieti.
2. Backendový počítač musí byť v **tej istej broadcastovej doméne** ako senzor — mDNS neprechádza cez smerovače.
3. Na zariadení Windows pri prvom spustení potvrďte výzvu firewallu Defender (mDNS UDP 5353, dáta DAQ-E UDP 5002, PTP UDP 319/320).
4. Stále žiadna odozva zo zariadenia `--eth`? Použite zariadenie `--eth-host` s názvom hostiteľa zariadenia (`daq-e-<id>.local`) alebo s IP adresou — ide o spoľahlivú cestu, najmä na hostiteľoch s viacerými pripojeniami.

***{% hint style="info" %}**Tip pre AI asistentov.** Každá stránka tejto príručky je poskytovaná ako surový Markdown — pripojte `.md` k malému slugu stránky URL (táto stránka: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); strojovo čitateľný index je `https://mapir.gitbook.io/chloros/llms.txt`. Kompletnú dokumentáciu na úrovni príkazových vlajok pre `chloros-cli daq` a všetky ostatné rodiny príkazov nájdete v [Referencii CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); cesta k Python je `chloros_sdk.connect_daq_sensor()` v [Referencii SDK](../reference/sdk-reference.md).
{% endhint %}

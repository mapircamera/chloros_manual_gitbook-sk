# Rýchly návod na použitie CLI (pool-*)

Dodávané zariadenie `chloros-cli` ovláda senzory DAQ prostredníctvom rodiny príkazov **`daq pool-*`** — jednoduchých HTTP klientov, ktoré ovládajú senzor cez trvalú skupinu senzorov backendu Chloros. Backend riadi prenos, takže grafické rozhranie (GUI), CLI a skripty SDK zdieľajú jednu aktívnu identifikáciu namiesto toho, aby sa navzájom súperili o port. Všetko, čo zákazník potrebuje, je dostupné prostredníctvom `pool-*`: pripojenie, streamovanie, nahrávanie kalibrovaných súborov `.daq` a výmena profilov kap.

`pool-*` je podporované rozhranie DAQ vo vydaných verziách. `chloros-cli daq --help` vypisuje podpríkazy `pool-*` a `daq discover` / `daq list` spustia rovnaké skenovanie ako `pool-discover`. Ak nie je k dispozícii podpríkaz DAQ pre priamy prístup k hardvéru, program sa ukončí s explicitnou chybou, v ktorej sa uvedie chýbajúci balík a odkáže vás späť na `pool-*` — nič sa neukončí bez upozornenia. (`pip install chloros-sdk` tiež neposkytuje príkazy pre priamy hardvér.)

***

## Predpoklady

* **Backend Chloros musí bežať** — príkazy `pool-*` sú HTTP klienti, nie hardvérové ovládače. V systéme Windows spustite desktopovú aplikáciu Chloros (tá spustí backend). Na bezdisplejovom Linuxe/Jetsone aktivujte službu ako svoj používateľ: `sudo systemctl enable --now chloros-backend@$USER.service` (pozri [Inštalácia v Linuxe](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Prihlásenie do Chloros+ (platená úroveň)**: najskôr spustite `chloros-cli login`. Vynucovanie prebieha na strane servera — bez prihlásenia príkazy zlyhajú s chybou `401 AUTH_REQUIRED`; v bezplatnej verzii (Iron) zlyhajú s chybou `403 PLAN_UPGRADE_REQUIRED`.
* Príkazy sú štandardne zamerané na `http://127.0.0.1:5000`; skupina príkazov `daq pool-*` rešpektuje premennú prostredia `CHLOROS_BACKEND_URL`, ak váš backend beží inde.

***

## Päťminútová relácia

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

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

| Variant | Význam |
| --- | --- |
| `daq pool-connect` | Inteligentná detekcia: vyhľadaj akékoľvek zariadenie DAQ na tomto počítači. |
| `daq pool-connect --port PORT` | DAQ-U na konkrétnom sériovom porte (napr. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M cez BLE, automatické vyhľadávanie MAC. |
| `daq pool-connect --mac MAC` | DAQ-M na známej adrese BLE MAC (predpokladá `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E so známym názvom hostiteľa alebo IP adresou — **spoľahlivá cesta**. |
| `daq pool-connect --eth` | DAQ-E s automatickým vyhľadávaním (mDNS, s náhradným riešením ARP). Pozrite si upozornenie nižšie. |

Nastavovacie príznaky, všetky voliteľné:

| Príznak | Význam |
| --- | --- |
| `--integration-time MS` / `-t MS` | Ručne nastavený integračný čas v milisekundách. |
| `--frame-avg N` / `-f N` | Počet snímok, z ktorých sa vypočíta priemer pre dané spektrum. |
| `--no-ae` | Vypnúť automatickú expozíciu (AE je štandardne zapnutá). |
| `--no-stream` | Pripojiť bez spustenia prenosu (neskôr pokračovať pomocou `pool-stream --start`). |
| `--cap-id CAP` | Profil korekcie Cap; predvolené nastavenie backendu je `sunshine_cosine`. Pozri [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

Pri pripojení zariadenia DAQ-E, ktoré je už v skupine pod inou adresou – pod názvom hostiteľa `daq-e-<id>.local` alebo pod IP adresou – sa vráti existujúci záznam v skupine namiesto toho, aby sa zariadenie otvorilo druhýkrát.

{% hint style="warning" %}
**Upozornenie týkajúce sa automatického vyhľadávania zariadenia `--eth`.** Na hostiteľovi s viacerými sieťovými rozhraniami (viac ako jedno aktívne sieťové rozhranie) môže byť *prvý* `pool-connect --eth` po spustení prázdny, aj keď je senzor v poriadku – vyhľadávanie môže prehliadnuť rozhranie senzora, kým je cache ARP prázdna. Ak `--eth` nič nenájde, skúste to znova alebo vyhľadávanie úplne preskočte pomocou `--eth-host <ip-or-hostname>`, čo je spoľahlivý postup na zariadeniach s viacerými sieťovými rozhraniami. Názov hostiteľa zariadenia DAQ-E je `daq-e-<id>.local` (napr. `daq-e-def330.local`); funguje aj jeho čistá IP adresa.
{% endhint %}

## `pool-discover` — vyhľadávanie bez pripojenia

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Vyhľadáva senzory, ku ktorým by ste sa mohli pripojiť, a zobrazuje adresu, na ktorú sa môžete pripojiť, bez toho, aby ste sa pripájali. `--only` prijíma podmnožinu `usb,ble,eth` oddelenú čiarkami; `--timeout SEC` nastavuje čas vyhľadávania na jeden prenos (predvolené nastavenie 5 s, obmedzené na 1–20 s). Senzory, ktoré sú už otvorené v skupine, nie sú uvedené v zozname (pripojený DAQ-M prestane vysielať signál) — pre ne použite `pool-list`. V dodanej verzii vykonávajú príkazy `chloros-cli daq discover` a `daq list` toto isté skenovanie.

## `pool-list` — zobraziť, čo je pripojené

Zobrazuje všetky senzory v backendovom poole, vrátane `sensor_id`, ktorý potrebujú všetky ostatné príkazy:

| Model | Typ `sensor_id` | Príklad |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktétový s pomlčkami | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

DAQ-E, ktorého PTP bolo vypnuté (nastavenie je uložené v zariadení), je v `pool-list` označené pomocou `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` a príkazu, ktorý ho opäť zapne — pozri [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — čítanie rámcov spektra

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Vráti najnovší rámec alebo najnovšie rámce; `--recent N` generuje strojovo čitateľný výstup pre skriptovanie. Rámce predstavujú radiometricky kalibrované spektrálne ožiarenie (W/m²/nm) na 135-bodovej mriežke v rozsahu 340–1010 nm, pričom je už aplikovaný profil krytu senzora. Pre kvantitatívne hodnoty ožiarenia je potrebné získať priemer z rámcov trvajúcich aspoň 15 sekúnd — ide o charakteristiku prístroja, nie o chybu.

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

| Prieznak | Predvolené | Význam |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Dĺžka nahrávania v sekundách; `0` znamená, že sa bude spúšťať, kým nezadáte `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Výstupný adresár. Pri backende na tomto počítači sa relatívna cesta vyhodnocuje vo vzťahu k priečinku, v ktorom ste spustili príkaz; pri vzdialenom backende sa cesta vyhodnocuje **na počítači, na ktorom beží backend**. |
| `--device-name NAME` | — | Štítok uložený spolu so záznamom. |
| `--stop` | — | Zastaviť prebiehajúci záznam. |
| `--confirm-after SEC` | `20` | Po spustení počkajte maximálne túto dobu na zapísanie prvého riadka a v prípade, že žiadny nedorazí, vydajte hlasné varovanie (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`). `0` túto kontrolu deaktivuje. |
| `--heartbeat SEC` | `30` | S `--duration` vytlačte počet riadkov každých SEC sekúnd a upozornite, ak prestane rásť. `0` túto kontrolu deaktivuje. |

{% hint style="info" %}
Záznam prebieha na backende, takže súbor `.daq` sa uloží do súborového systému **backendového počítača** — štandardne v `~/Documents/DAQ Live View/`, nie nevyhnutne tam, kde ste spustili CLI. Názvy súborov obsahujú ID senzora a časovú pečiatku.
{% endhint %}

## `pool-set-cap` — deklarácia nasadeného krytu

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

ID krytu určuje výrobcom nameraný korekčný profil, ktorý sa aplikuje na každé spektrum, a **musí zodpovedať krytu fyzicky nasadenému na senzore** — ani senzor, ani softvér nedokážu kryt detekovať samostatne a tento výber je zaznamenaný v každom súbore `.daq`. Všetky zariadenia majú predvolené nastavenie `sunshine_cosine` (každé zariadenie DAQ sa dodáva s nainštalovaným krytom Sunshine s kosínusovou korekciou, ktorý má konštrukčne stanovené útlmenie ~12× — neoznámená zmena krytu spôsobí nesprávnu korekciu spektier približne o tento faktor).

| `--cap-id` | K dispozícii na |
| --- | --- |
| `sunshine_cosine` (predvolené) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | iba DAQ-U |
| `none` | iba DAQ-E — pozri poznámku |

Identifikátor krytu, ktorý nie je súčasťou sady senzora, je pri pripojení odmietnutý s jasnou chybou. `none` (DAQ-E) znamená, že kryt je fyzicky odstránený — na zapustený sklenený difúzor zariadenia DAQ-E sa stále uplatňuje výrobný geometrický profil, takže nejde o nefunkčnú konfiguráciu, a holé zariadenie DAQ-E je laboratórnou konfiguráciou, nie podporovanou konfiguráciou pre použitie v teréne. (Holý DAQ-U je skutočne holý a nepotrebuje žiadny korekčný profil; DAQ-M sa používa so svojím krytom Sunshine.)

## `pool-disconnect` — uvoľnenie senzorov

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` uvoľní senzor bez ohľadu na to, kto ho otvoril, takže uvoľní aj senzor, ktorý v poole zanechal havarovaný skript (najskôr sa dokončí nahrávanie na ňom). Senzor, ktorý nie je v poole, nemá čo uvoľniť: príkaz vypíše `<sensor-id>: already released (not in the pool)` a ukončí sa `0`. Uvoľnenie, ktoré backend odmietol alebo nepotvrdil, ukončí sa `1`.

## `pool-enable-ptp` — opätovné zapnutie PTP na zariadení DAQ-E

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

DAQ-E uchováva nastavenie zapnutia/vypnutia PTP vo svojej vlastnej pamäti, takže jednotka, ktorej PTP bolo vypnuté, zostáva vypnutá — a jej časové značky nie sú synchronizované s PTP — až kým sa opäť nezapne. `pool-list` takúto jednotku označí a vypíše pre ňu tento príkaz. Program Chloros nikdy sám o sebe jednotku opäť nezapne, pretože vypnutie môže byť zámerné. Novinka vo verzii 1.3.0.

## Nastavenia zariadenia DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` zobrazí firmvér senzora (a informáciu, či táto verzia Chlorosu obsahuje novšiu verziu), jeho adresu a spôsob jej získania, uložené sieťové nastavenia, ak sa líšia od aktuálnych, streamovanie pri zapnutí a PTP. `pool-set-network` reštartuje senzor, aby sa zmena uplatnila, a opätovne ho pripojí s pôvodnými nastaveniami; `--no-reboot` ich iba uloží. V sieti tohto počítača musí byť statická adresa, inak ju Chloros odmietne; `--force` toto obíde, avšak len pre počítač, ktorý bude preadresovaný alebo smerovaný na túto adresu. Reštarty sú odmietnuté, pokiaľ senzor zaznamenáva údaje. `pool-update-firmware` je ekvivalentom položky **Update to \<version\>** </version\>v grafickom rozhraní <version\>pre príkazový riadok a spôsobom aktualizácie zariadenia DAQ-E v systéme Linux, ktorý nemá grafické rozhranie. Novinka vo verzii 1.3.0.***

## Prehľad príkazov

| Príkaz | Účel |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Otvorí senzor v backendovej skupine. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Vyhľadáva senzory bez pripojenia (nájde BLE MAC zariadenia DAQ-M). |
| `daq pool-list` | Zobraziť všetky senzory v skupine spolu s ich `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Posledných N kalibrovaných spektrálnych rámcov. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Obnovenie / pozastavenie streamovania. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Spustenie / zastavenie záznamu `.daq` (na strane backendu). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Zmena profilu korekcie cap počas behu programu. |
| `daq pool-disconnect --sensor-id ID [--all]` | Uvoľnenie jedného alebo všetkých senzorov. |
| `daq pool-enable-ptp --sensor-id ID` | Znovu zapnúť PTP pre zariadenie DAQ-E, ktorého PTP je vypnuté. |
| `daq pool-info --sensor-id ID [--json]` | Firmvér zariadenia DAQ-E, adresa, uložené sieťové nastavenia, streamovanie pri zapnutí a PTP. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP alebo statická adresa; reštartuje a opätovne pripojí senzor. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Streamovanie pri zapnutí. |
| `daq pool-reboot --sensor-id ID` | Reštartuje zariadenie DAQ-E a opätovne ho pripojí. |
| `daq pool-update-firmware --sensor-id ID` | Nahraje firmvér zariadenia DAQ-E z tejto verzie do staršieho senzora. |

***

## Riešenie problémov pri prvom pripojení DAQ-E

1. DAQ-E nemá stavovú LED — skontrolujte napájanie pomocou indikátora PoE/link na prepínači alebo porte injektora a po zapnutí počkajte niekoľko sekúnd, kým sa zariadenie naštartuje a pripojí k sieti.
2. Backendový počítač musí byť v **tej istej vysielacej doméne** ako senzor — mDNS neprechádza cez smerovače.
3. Firewall musí povoliť prístup pre mDNS (UDP 5353), dáta DAQ-E (UDP 5002) a PTP (UDP 319/320). Inštalačný program pre Windows pridá na to pravidlo a v Linuxe `.deb` ich otvorí v `ufw` / `firewalld`, ktorý bol aktívny v čase inštalácie; firewall tretej strany alebo firewall aktivovaný neskôr vyžaduje, aby boli povolené.
4. Stále nič z `--eth`? Použite `--eth-host` s názvom hostiteľa zariadenia (`daq-e-<id>.local`) alebo IP adresou, čím sa preskočí vyhľadávanie.

***
{% hint style="info" %}
**Tip pre AI asistentov.** Každá stránka tejto príručky je poskytovaná vo formáte surového Markdownu — pripojte `.md` k malým písmenám v URL-slugu stránky (táto stránka: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); strojovo čitateľný index je `https://mapir.gitbook.io/chloros/llms.txt`. Kompletnú dokumentáciu na úrovni príkazových vlajok pre `chloros-cli daq` a všetky ostatné rodiny príkazov nájdete v [Referencii CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); Pythonová cesta je `chloros_sdk.connect_daq_sensor()` v [Referencii SDK](../reference/sdk-reference.md).
{% endhint %}
</version\>

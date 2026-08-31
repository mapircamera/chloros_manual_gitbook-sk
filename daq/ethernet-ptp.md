# Sieťové pripojenie a synchronizácia času zariadenia DAQ-E

> Informácie o fyzickom nastavení siete pre senzor – kabeláž, PoE, pridelenie IP adresy a vlastné sieťové nastavenia zariadenia – nájdete v **[užívateľskej príručke k zariadeniu DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Táto stránka sa venuje zariadeniu Chloros: pripojeniu, synchronizácii času a postupu v prípade, ak vyhľadávanie neprebehne úspešne.

DAQ-E je členom rodiny DAQ s ethernetovým pripojením: napájaný cez PoE, vyhľadávaný prostredníctvom mDNS (služba `_daq-e._tcp`) a adresovateľný pomocou hostname odvodeného od ID senzora — `daq-e-<6 hex>.local`, napr. `daq-e-def330.local`. Táto stránka sa zaoberá tým, ako prenáša dáta v sieti a ako sa podieľa na časovej synchronizácii PTP.

## Režimy prenosu

| Režim | Koncový bod | Odberatelia | Poznámky |
| --- | --- | --- | --- |
| **Multicast** (predvolené) | UDP `239.10.10.10:5002` | Akýkoľvek počet zariadení v tej istej sieti LAN prijíma ten istý tok | Každý datagram je overený pomocou CRC-16/CCITT |
| **Raw** | TCP port `5000` | Presne jeden klient (exkluzívne) | Kompatibilný na úrovni bajtov s DAQ-U |

Chloros štandardne používa multicast, čo umožňuje, aby GUI, CLI a SDK všetky súčasne sledovali jeden senzor.

## Sieťové požiadavky

* **Rovnaká vysielacia doména.** Počítač, na ktorom beží Chloros, musí byť v rovnakom segmente siete L2 ako senzor — vyhľadávanie mDNS neprechádza cez smerovače.
* **Výzva brány firewallu pre Windows: potvrďte ju.** Keď program Chloros po prvýkrát viaže multicastové sokety, program Windows Defender zobrazí jednorazovú výzvu. Povolením sa pokryjú dáta DAQ-E (UDP 5002), mDNS (UDP 5353) a PTP (UDP 319/320). Na zariadení Linux sa toto neoznámi.
* **Napájanie cez PoE, bez stavovej LED.** DAQ-E nemá vlastnú LED — skontrolujte napájanie pomocou indikátora link/PoE na prepínači alebo porte injektora a po zapnutí počkajte niekoľko sekúnd, kým sa zariadenie naštartuje a pripojí k sieti.

## Pripojenie

**GUI:** Karta Svetelné senzory → Pripojiť senzor → Typ zariadenia „DAQ-E (Ethernet)“. Vyhľadávanie prebieha len počas zobrazenia dialógového okna pripojenia (prehľadávanie mDNS a ARP na adrese Windows), opakuje sa každých 15 sekúnd; tlačidlo Obnoviť spustí okamžité opätovné vyhľadávanie. Zistené senzory sa zobrazia v roletovom menu; automaticky sa vyberie prvý zistený senzor.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (beží backend):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Hostitelia s viacerými sieťovými kartami a prvé pripojenie po spustení systému

Na hostiteľoch s viac ako jedným aktívnym sieťovým rozhraním môže byť **prvý** `pool-connect --eth` po spustení prázdny, aj keď je senzor v poriadku — vyhľadávanie môže prehliadnuť rozhranie, na ktorom senzor beží, pokiaľ je cache ARP ešte prázdna. Spoľahlivým riešením je vynechať vyhľadávanie a adresu zadať explicitne:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` prijíma názov hostiteľa mDNS alebo IP adresu, vždy sa zameriava na správny senzor a je odporúčanou formou pre skripty a inštalácie bez grafického rozhrania. V grafickom rozhraní použite tlačidlo Obnoviť v dialógovom okne pripojenia a počkajte na dokončenie cyklu opätovného vyhľadávania.

## Nastavenia zariadenia a firmvér

Samotný senzor uchováva sieťové nastavenia – statická IP adresa vs. DHCP + adresovanie link-local, názov zariadenia, automatické spustenie streamovania pri štarte, heslo OTA. Tieto nastavenia na strane zariadenia nie sú v dodávanom CLI dostupné ako príkazy; spravujú sa prostredníctvom grafického rozhrania Chloros, kde sú zobrazené, alebo s podporou MAPIR.

**Aktualizácie firmvéru sú integrované do grafického rozhrania.**Ak pripojené zariadenie DAQ-E používa staršiu verziu firmvéru, ako je obraz dodávaný s vašou verziou Chloros, v riadku senzora sa zobrazí oranžová ikona**K dispozícii je aktualizácia** a v modálnom okne nastavení s ozubeným kolieskom sa ponúka<version>

tlačidlo</version> „Aktualizovať na<version>

“. Aktualizácia sa prenesie cez sieť za približne 30 sekúnd; senzor sa automaticky reštartuje a opäť sa pripojí, pričom v prípade prerušenia prenosu zostane aktuálny firmvér nedotknutý.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Časová synchronizácia PTP

Firmware zariadenia DAQ-E verzie v1.2.0 a novších podporuje štandard IEEE 1588 PTPv2 ako bežný (výlučne podriadený) časovač. **Backend hostiteľa Chloros je PTP grandmaster** — každý DAQ-E a každá kamera LATTICE v sieti LAN sa k nemu pripájajú ako podriadené zariadenia v doméne 0, čím sa udržiavajú časové pečiatky všetkých zariadení v tolerancii ~1 ms. Vďaka tomuto zdieľanému časovaču je možné zosúladiť časové značky meraní z DAQ s expozíciami kamery (pozri [Nahrávanie a formát .daq](recording.md)).

Skontrolujte synchronizáciu z CLI:

| Príkaz | Zobrazuje |
| --- | --- |
| `chloros-cli time-sync status` | Stav hostiteľa ako hlavného časového generála, priority BMCA, identita hodín |
| `chloros-cli time-sync peers` | Všetky zistené podriadené zariadenia (senzory DAQ-E + kamery LATTICE) |
| `chloros-cli time-sync cameras` | Stav PTP pre každú kameru (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Reštartovanie procesu grandmaster |

V grafickom rozhraní (GUI) sa v modálnom okne nastavení DAQ-E zobrazuje riadok **PTP Sync** v reálnom čase s aktuálnym stavom PTP senzora.

Podrobnosti pre spotrebiteľov vyžadujúcich prísne zosúladenie:

* Každý streamovaný datagram obsahuje pole príznakov; **bit 2 je nastavený v rámcoch, ktorých časová pečiatka je synchronizovaná s PTP**. Potrubia, ktoré vyžadujú prísne zosúladenie kamery a DAQ, by mali byť riadené týmto bitom.
* Pred synchronizovaným snímaním skontrolujte, či sa senzor nachádza v zozname `chloros-cli time-sync peers`. (Interné hardvérové nástroje MAPIR môžu tiež riadiť nahrávanie na základe PTP synchronizácie pomocou príznaku `--wait-ptp`, ktorý čaká až 15 s, kým senzor dosiahne stav SLAVE; táto funkcia nie je súčasťou dodávaného zariadenia CLI.)
* Pokiaľ PTP aktívne funguje v režime slave, senzor odmieta ručné zasielanie taktu („PTP poskytuje takt“). Je to tak navrhnuté – dôverujte PTP.

## Poznámky k Linux

* **PTP potrebuje `libcap2-bin` v čase inštalácie.** Skript `.deb` postinst udeľuje oprávnenie skriptu `cap_net_bind_service=+ep` na skripte `/usr/lib/chloros/chloros-backend`, aby mohol viazať porty PTP 319/320 bez oprávnenia root. Ak chýba súbor `libcap2-bin`, tento krok sa preskočí a PTP sa nepodarí spustiť. Riešenie:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Jetson / Raspberry Pi bez grafického rozhrania:** pri prvej inštalácii sa vygeneruje jednotka systemd `chloros-backend.service`, ale nie je aktivovaná. Pre neustále aktívny PTP (a dostupnosť DAQ) bez grafického rozhrania:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Bez nej beží PTP len počas otvorenia grafického rozhrania Chloros.

## Riešenie problémov: „Nenašli sa žiadne zariadenia DAQ-E“

| Kontrola | Podrobnosti |
| --- | --- |
| Napájanie | Žiadna LED na senzore — skontrolujte indikátory PoE a prepojenia na prepínači/porte injektora; po zapnutí počkajte niekoľko sekúnd |
| Vysielacia doména | Hostiteľ a senzor sú v rovnakom segmente L2; mDNS nesmeruje |
| Firewall Windows | Pri prvom spustení potvrďte výzvu programu Defender (UDP 5002, 5353, 319/320) |
| Hostiteľ s viacerými sieťovými kartami | Pri prvom vyhľadávaní po spustení sa môže stať, že senzor nebude nájdený — pripojte sa pomocou `--eth-host <ip-or-hostname>` |
| Opätovné vyhľadávanie v grafickom rozhraní | Vyhľadávanie prebieha len počas otvorenia dialógového okna pripojenia; použite tlačidlo Obnoviť |</version>

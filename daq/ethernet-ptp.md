# Sieťové pripojenie a synchronizácia času zariadenia DAQ-E

> Informácie o fyzickom nastavení siete pre senzor – kabeláž, PoE, pridelenie IP adresy a vlastné sieťové nastavenia zariadenia – nájdete v **[užívateľskej príručke k zariadeniu DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Táto stránka sa venuje časti týkajúcej sa zariadenia Chloros: pripojeniu, synchronizácii času a postupu v prípade, ak vyhľadávanie neprinesie žiadne výsledky.

DAQ-E je ethernetovým členom rodiny DAQ: napájaný cez PoE, vyhľadávaný prostredníctvom mDNS (služba `_daq-e._tcp`) a adresovateľný pomocou hostname odvodeného od jeho ID senzora — `daq-e-<6 hex>.local`, napr. `daq-e-def330.local`. Táto stránka popisuje, ako prenáša dáta v sieti a ako sa podieľa na časovej synchronizácii PTP.

## Režimy prenosu

| Režim | Koncový bod | Odberatelia | Poznámky |
| --- | --- | --- | --- |
| **Multicast** (predvolené) | UDP `239.10.10.10:5002` | Akýkoľvek počet zariadení v tej istej sieti LAN prijíma ten istý tok | Každý datagram je overený pomocou CRC-16/CCITT |
| **Raw** | TCP port `5000` | Presne jeden klient (exkluzívne) | Kompatibilný na úrovni bajtov s DAQ-U |

Chloros štandardne používa multicast, čo umožňuje, aby GUI, CLI a SDK mohli všetky súčasne sledovať jeden senzor.

## Sieťové požiadavky

* **Rovnaká vysielacia doména.** Počítač, na ktorom beží Chloros, musí byť v rovnakom segmente siete L2 ako senzor — vyhľadávanie mDNS neprechádza cez smerovače.
* **Firewall.** V systéme Windows inštalátor pridá do každého sieťového profilu pravidlo firewallu **Chloros Backend (UDP-In)**pre backend, ktoré pokrýva dáta DAQ-E (UDP 5002), mDNS (UDP 5353) a PTP (UDP 319/320), a pravidlo**„Chloros Backend (TCP-In)“**, obmedzené na lokálnu podsieť, pre pripojenie, ktoré zariadenie DAQ-E nadväzuje späť na počítač počas aktualizácie firmvéru. V systéme Linux u `.deb` otvorí porty UDP v `ufw` alebo `firewalld`, ak je tento firewall aktívny v čase inštalácie. Firewall tretej strany alebo firewall aktivovaný neskôr vyžaduje povolenie tých istých portov.
* **Napájanie cez PoE, bez stavovej LED.** Zariadenie DAQ-E nemá vlastnú LED – napájanie skontrolujte pomocou indikátora link/PoE na prepínači alebo na porte injektora a po zapnutí počkajte niekoľko sekúnd, kým sa zariadenie naštartuje a pripojí k sieti.

## Pripojenie

**GUI:** Karta Svetelné senzory → Pripojiť senzor → Typ zariadenia „DAQ-E (Ethernet)“. Vyhľadávanie prebieha len počas zobrazenia dialógového okna pripojenia (prehľadávanie mDNS a ARP v systéme Windows), pričom sa opakuje každých 15 sekúnd; tlačidlo Obnoviť spustí okamžité opätovné vyhľadávanie. Zistené senzory sa zobrazia v roletovom menu; automaticky sa vyberie prvý zistený senzor.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (beží backend):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

DAQ-E, ktorý je už v skupine, sa rozpoznáva podľa názvu hostiteľa aj podľa IP adresy: opätovné pripojenie pod inou adresou vráti existujúci záznam v skupine namiesto toho, aby sa jednotka otvorila druhýkrát.

### Hostitelia s viacerými sieťovými kartami a prvé pripojenie po spustení

Na hostiteľoch s viac ako jedným aktívnym sieťovým rozhraním môže byť **prvé** pripojenie `pool-connect --eth` po spustení systému neúspešné, aj keď je senzor v poriadku — vyhľadávanie môže prehliadnuť rozhranie, na ktorom senzor beží, pokiaľ je cache ARP ešte prázdna. Spoľahlivým riešením je preskočiť vyhľadávanie a adresu zadať explicitne:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` akceptuje názov hostiteľa mDNS alebo IP adresu, vždy sa zameriava na správny senzor a je odporúčanou formou pre skripty a inštalácie bez grafického rozhrania. V grafickom rozhraní použite tlačidlo Obnoviť v dialógovom okne pripojenia a počkajte na dokončenie cyklu opätovného vyhľadávania.

## Nastavenia zariadenia a firmvér

Samotný senzor uchováva svoje sieťové nastavenia: DHCP (továrenské nastavenie, ktoré prejde na lokálnu adresu 169.254.x.x, ak žiadny server DHCP neodpovedá) alebo statickú adresu. Uchováva tiež informáciu o tom, či začne streamovať pri zapnutí. Tieto nastavenia môžete zmeniť v grafickom rozhraní (v nastaveniach senzora, ikona ozubeného kolesa, **Sieť**:**Zmeniť…**,**Prúd pri zapnutí**,**Reštartovať senzor**; pozri [Karta DAQ](gui.md)), alebo cez CLI a SDK pomocou `pool-set-network`, `pool-set-auto-stream` a `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; pozri [Referenčnú príručku CLI](../reference/cli-reference.md#daq-e-device-settings)). Senzor sa reštartuje, aby sa uplatnila zmena v sieti, a Chloros ho opäť pripojí. V sieti počítača, na ktorom beží Chloros, musí byť statická adresa: Chloros odmietne takú, ku ktorej sa po reštarte nedokáže pripojiť. ID senzora a jeho hostname `daq-e-<id>.local` sa nikdy nemenia. Novinka vo verzii 1.3.0.

**Aktualizácie firmvéru sú integrované do grafického používateľského rozhrania (GUI).** Ak pripojené zariadenie DAQ-E používa staršiu verziu firmvéru, ako je obraz dodávaný s vašou verziou Chloros, v riadku jeho senzora sa zobrazí oranžová ikona **Aktualizácia k dispozícii** a v modálnom okne nastavení s ozubeným kolieskom sa ponúka <version>tlačidlo</version>

„Aktualizovať na <version>“. Aktualizácia sa prenesie cez sieť za približne 30 sekúnd; senzor sa automaticky reštartuje a opäť pripojí, pričom v prípade prerušenia prenosu zostane aktuálny firmvér nedotknutý. Z CLI alebo SDK a v systéme Linux, ktorý nemá grafické rozhranie, spustite príkaz `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Synchronizácia času PTP

Firmware DAQ-E verzie 1.2.0+ sa zúčastňuje na štandarde IEEE 1588 PTPv2 ako bežný (výhradne podriadený) časovač. **Backend hostiteľa Chloros je PTP grandmaster** — každý DAQ-E a každá kamera LATTICE v sieti LAN sa k nemu pripájajú ako podriadené zariadenia v doméne 0, čím sa udržujú časové značky všetkých zariadení v tolerancii ~1 ms. Vďaka tomuto zdieľanému časovaču je možné zosúladiť časové značky meraní z DAQ s expozíciami kamery (pozri [Nahrávanie a formát .daq](recording.md)).

Skontrolujte synchronizáciu z CLI:

| Príkaz | Zobrazí |
| --- | --- |
| `chloros-cli time-sync status` | Stav hostiteľského grandmastera, priority BMCA, identitu hodín a `health`: `OK` alebo `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` s informáciou od `health_detail`, prečo sa jeho pakety nedostávajú do siete |
| `chloros-cli time-sync peers` | Zobrazené všetky podriadené zariadenia (senzory DAQ-E + kamery LATTICE) |
| `chloros-cli time-sync cameras` | Stav PTP pre každú kameru (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Reštartovať proces grandmastera |

V grafickom používateľskom rozhraní (GUI) sa v modálnom okne nastavení DAQ-E zobrazuje riadok **PTP Sync** v reálnom čase s aktuálnym stavom PTP senzora.**Zariadenie DAQ-E s vypnutým PTP zostáva vypnuté.** Zariadenie si uchováva nastavenie zapnutia/vypnutia PTP vo svojej vlastnej pamäti, takže zariadenie, ktorého PTP bolo vypnuté, zostáva vypnuté aj po opätovnom pripojení alebo vypnutí a zapnutí napájania a jeho časové značky nie sú synchronizované s PTP. `chloros-cli daq pool-list` takúto jednotku označí (`!! PTP is switched OFF on this unit …`) a vypíše príkaz, ktorý to napraví:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros nikdy sám nezapne jednotku späť, pretože vypnutý stav môže byť zámerný.

Podrobnosti pre používateľov vyžadujúcich prísne zosúladenie:

* Každý streamovaný datagram obsahuje pole príznakov; **bit 2 je nastavený v rámcoch, ktorých časová pečiatka je synchronizovaná s PTP**. Potrubia, ktoré vyžadujú prísne zosúladenie kamery/DAQ, by mali byť riadené týmto bitom.
* Pred synchronizovaným zachytením overte, či sa senzor objavuje v `chloros-cli time-sync peers` a či `chloros-cli daq pool-list` neoznačuje jeho PTP ako vypnuté. (Vnútorné nástroje MAPIR pre priamu prácu s hardvérom môžu tiež riadiť nahrávanie na základe uzamknutia PTP pomocou príznaku `--wait-ptp`, ktorý čaká až 15 s, kým senzor dosiahne stav SLAVE; tieto nástroje nie sú súčasťou dodávaného CLI.)
* Pokiaľ je PTP aktívne v režime slave, senzor odmieta ručné zasielanie hodín („PTP poskytuje hodiny“). Je to tak zámerne — dôverujte PTP.

## Poznámky k Linuxu

* **PTP porty bez oprávnení root.** PTP viaže porty UDP 319/320. `.deb` zapisuje do `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), takže backend sa na ne pripája a beží pod vaším používateľským účtom. Od verzie 1.3.0 balík nenastavuje na backende žiadnu súborovú schopnosť (`setcap`) a nepotrebuje `libcap2-bin`; nepridávajte túto schopnosť sami, pretože rozbalený backend sa potom nespustí. Ak PTP nemôže viazať port 319, pozrite si [Inštalácia v systéme Linux](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Jetson bez grafického rozhrania / hostiteľský počítač s Linuxom:** balík nainštaluje služby backendu, ale žiadnu z nich neaktivuje. Ak chcete, aby PTP (a DAQ) boli stále aktívne bez toho, aby niekto spúšťal príkaz, aktivujte službu ako svoj používateľ:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Bez toho bude PTP bežať len vtedy, keď niečo spustilo backend — príkaz `chloros-cli` alebo skript SDK. Pozrite si [Inštalácia v systéme Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) pre informácie o používateľovi root `chloros-backend.service` a o tom, ako z neho prejsť.

## Riešenie problémov: „Nenašli sa žiadne zariadenia DAQ-E“

| Kontrola | Podrobnosti |
| --- | --- |
| Napájanie | Na senzore nesvieti žiadna LED — skontrolujte indikátory PoE a pripojenia na prepínači/portu injektora; po zapnutí počkajte niekoľko sekúnd |
| Vysielacia doména | Hostiteľ a senzor sú v rovnakom segmente L2; mDNS nesmeruje |
| Firewall | Windows: pravidlo **Chloros Backend (UDP-In)** v inštalačnom programe povoľuje prísun dát DAQ-E (UDP 5002), mDNS (5353) a PTP (319/320); firewall od iného výrobcu vyžaduje rovnaké nastavenie. Linux: `.deb` otvorí tieto porty v `ufw` / `firewalld` len vtedy, ak bol firewall aktívny v čase inštalácie — v prípade neskoršieho zapnutia ich musíte povoliť ručne |
| Hostiteľ s viacerými sieťovými kartami | Vyhľadávanie prehľadáva mDNS s náhradným riešením ARP; `--eth-host <ip-or-hostname>` preskočí vyhľadávanie a pripojí sa priamo |
| Opätovné vyhľadávanie cez grafické rozhranie | Vyhľadávanie beží len počas otvorenia dialógového okna pripojenia; použite tlačidlo Obnoviť |

# Pripojenie kamier

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Karta Kamery pred pripojením akéhokoľvek zariadenia</p></figcaption></figure>Chloros automaticky vyhľadáva kamery LATTICE na prepojení — z karty „Kamery“ v grafickom rozhraní, z `chloros-cli lattice` alebo z Python SDK. Reťazec modelu kamery riadi všetko, čo nasleduje: Chloros určí profil senzora, rozloženie pásiem a továrenskú kalibráciu na základe údajov z kamery `DeviceUserID` + `DeviceSerialNumber`, takže **nie je potrebné nič konfigurovať pre jednotlivé kamery**.

Pred pripojením sa uistite, že je nastavená hostiteľská sieť — adresovanie link-local, jumbo rámce a v prípade polí nastavenia prijímacej vyrovnávacej pamäte sieťovej karty. Ide o nastavenie na strane hardvéru, ktoré je uvedené v príručke LATTICE: [**Nastavenie siete**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Pripojenie z grafického používateľského rozhrania

Otvorte kartu **Kamery**v bočnom paneli Chloros (karty hardvéru sa zobrazia po dokončení spustenia backendu) alebo použite hlavné menu →**Pripojiť sa ku kamere**. Obe možnosti otvoria dialógové okno**Pripojiť kameru (kamery)**.

### Dialógové okno **Pripojiť kameru (kamery)**Dialógové okno hneď po otvorení prehľadá sieť („Prehľadávanie siete...“) a zobrazí zoznam všetkých nájdených kamier. V každom riadku je uvedený**model**kamery (napr. `LATT-M3M-L41-F550`),**sériové číslo**a**IP adresa**.

* **Kliknutím na riadok ho vyberiete**(zelené zvýraznenie). Môžete vybrať**viacero kamier** a pripojiť ich naraz — Chloros ich pripája postupne.
* Riadky s označením **„Pripojené“** sú už pripojené a nemožno ich znovu vybrať.
* Riadky s označením **„V poli“** patria do aktuálne pripojeného poľa kamier. Ak chcete danú kameru používať samostatne, najskôr odpojte pole.
* **Pripojiť** — pripojí vybrané kamery; ak je vybraných viac ako jedna kamera, na tlačidle sa zobrazí počet, napr. „Pripojiť (3)“.
* **Opätovné vyhľadávanie** — spustí vyhľadávanie znova.
* **Zatvoriť** — zatvorí dialógové okno.
* Ak sa vyhľadávanie ukončí bez výsledkov, v dialógovom okne sa zobrazí **„V sieti neboli nájdené žiadne kamery“** — pozrite si časť [Riešenie problémov](connecting.md#troubleshooting) nižšie.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Dialógové okno „Pripojiť kameru(-y)“ — zobrazené tu bez kamier v sieti</p></figcaption></figure>### Prvé pripojenie: stiahnutie kalibračného balíka

Keď je daná kamera **prvýkrát**pripojená k zariadeniu, Chloros stiahne továrenský kalibračný balík kamery (\~3,8 MB) priamo z kamery cez rozhranie GigE. Počas tohto procesu sa v dialógovom okne zobrazí zelený panel**„Sťahovanie kalibračných údajov z kamery“**s ukazovateľom priebehu pre každé sériové číslo – počítajte s približne**70 sekundami** na jednu kameru. Balík sa uloží do vyrovnávacej pamäte na hostiteľskom počítači, takže pri neskorších pripojeniach tej istej kamery sa sťahovanie úplne preskočí (a panel sa nezobrazí).

### Analyzovať systém

Tlačidlo **Analyzovať systém** v dialógovom okne preverí hostiteľa a sieť (počas spustenia sa zobrazuje nápis „Analyzovanie...“) a vygeneruje diagnostickú správu:

* **Hostiteľ** — jadrá procesora a RAM; názov a pamäť grafického procesora (GPU) alebo „GPU: Žiadny nezistený“.
* **Sieťové rozhrania** — názov každej sieťovej karty, rýchlosť spojenia, MTU (s označením „jumbo“, ak je aktívne), stav pripojenia (aktívne/neaktívne) a informácia, či je pripojená na zbernicu USB.
* **Kamery**— sériové číslo, model, IP adresa a**na ktorej sieťovej karte je každá kamera pripojená**.
* **Výkon** — aktuálny vs. ideálny počet snímok za sekundu (fps) na kameru pre daný formát pixelov, so zeleným riadkom „Potenciál: možné zlepšenie o N×“, ak ideálna hodnota prevyšuje aktuálnu.
* **Upozornenia a očíslované odporúčania** — alebo „Systém vyzerá v poriadku pre aktuálny počet kamier“, ak nie je potrebné nič opravovať.

Spustite ho vždy, keď sa vyhľadávanie alebo streamovanie správa neočakávane — identifikuje väčšinu problémov na strane sieťovej karty (nesprávna hodnota MTU, kamera na nesprávnom rozhraní, obmedzenia USB adaptéra) bez opustenia dialógového okna.

### Pripojenie skupiny

Ak chcete pripojiť dve alebo viac kamier ako **synchronizovanú skupinu**, použite namiesto toho sprievodcu pripojením skupiny (**Pripojiť skupinu kamier**): sprievodca vás prevedie výberom hlavnej a podriadených kamier (predvyplneným pomocou sondy na zapojenie GPIO), výberom režimu zobrazenia (samostatné vs. kombinované dlaždice) a nastaveniami poľa so živou projekciou dosiahnuteľných snímok za sekundu a šírky pásma kábla, než potvrdíte zmeny. Sprievodca a pracovné postupy pre sústavy sú popísané v časti tejto príručky venovanej sústavám viacerých kamier; ekvivalentom pre model CLI je „Postup prvého pripojenia kamery LATTICE“ v [Referenčnom manuáli k modelu CLI](../reference/cli-reference.md).

## Pripojenie z CLI a SDK

Prístup k službám CLI a SDK vyžaduje platenú úroveň Chloros+ a prihlásenie; toto sa vynucuje na strane servera (`401 AUTH_REQUIRED`, ak nie ste prihlásení, `403 PLAN_UPGRADE_REQUIRED` v bezplatnej úrovni).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Úplné podpisy, možnosti a pracovné postupy zachytávania: [CLI Referencia](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK Referencia](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Ako prebieha kalibrácia pri pripojení

Každá kamera LATTICE má svoj továrenský kalibračný balík **uložený priamo v kamere** a model Chloros pri pripojení kamery tiež skontroluje cloud modelu MAPIR:

| Situácia   | Čo používa model Chloros                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**|**Najnovšia kalibrácia zverejnená pre dané sériové číslo** — kópia v cloude má prednosť pred kópiou v kamere. Kamera, ktorá bola prekalibrovaná alebo aktualizovaná prostredníctvom MAPIR, sa preto aktualizuje automaticky; nie je potrebná žiadna akcia zo strany používateľa. |
| **Offline**|**Balík v fotoaparáte** zostáva v pôvodnom stave. Plne offline pracovné postupy naďalej fungujú; jednoducho nepreberajú novšie kalibrácie, kým sa fotoaparát aspoň raz nepripojí online (alebo nedôjde k obnoveniu továrenského nastavenia).                                                  |

V čase snímania sa skutočne použité koeficienty **uložia do XMP metadát každého snímku**. Neskoršia aktualizácia kalibrácie nikdy bez upozornenia nezmení snímky, ktoré ste už nasnímali — pri opätovnom spracovaní starého snímku sa použijú koeficienty uložené v jeho XMP, nie tie, ktoré sú dnes najnovšie.

## Riešenie problémov

* **„V sieti neboli nájdené žiadne kamery“**— overte nastavenie lokálnej siete v [Nastavenie siete](https://mapir.gitbook.io/lattice-camera/setup/network-setup): statická sieťová karta hostiteľa `169.254.x.x/16`, kamery na rovnakom prepojení, neočakáva sa použitie DHCP ani brány. Potom použite funkciu**Analyze System**v dialógovom okne pripojenia, aby ste skontrolovali, na ktorej sieťovej karte je (alebo nie je) viditeľná každá kamera. Po akejkoľvek zmene kabeláže alebo sieťovej karty vykonajte**Rescan**.
* **Zariadenie, ktoré predtým fungovalo, sa odmieta pripojiť** (brány panelu poľa s `FRAMES WILL DROP` / `Reduce ROI to enable`) — aktualizácia ovládača sieťovej karty bez upozornenia vynulovala nastavenia prijímacieho krúžku. Znovu ich nastavte alebo spustite `chloros-cli lattice network --fix` z terminálu s oprávneniami správcu; pozrite si [Nastavenie siete](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Kamera zobrazuje hlásenie „In Array“** — patrí do pripojenej relácie poľa. Odpojte pole, ak chcete kameru používať samostatne.

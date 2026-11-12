---
metaLinks: {}
---

# Začíname

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros je softvérová aplikácia od spoločnosti [MAPIR](https://www.mapir.camera) určená na spracovanie multispektrálnych snímok, živé ovládanie hardvéru MAPIR a zaznamenávanie údajov zo senzorov. Chloros 1.3.0 podporuje celú produktovú rodinu MAPIR:

* **Kamery Survey3** — spracúvajú zábery vo formáte RAW+JPG na kalibrované mapy odrazivosti a vegetačných indexov. Pozrite si [Podporované kamery](supported-cameras.md).
* **Kamery LATTICE** — pripojte multispektrálne kamerové moduly GigE v reálnom čase, jednotlivo alebo ako synchronizované sústavy viacerých kamier: náhľad, snímanie a spracovanie na kalibrované produkty žiarivosti a odrazivosti. Pozrite si [sekciu LATTICE](lattice/README.md).
* **Svetelné senzory DAQ** — spektrálne senzory DAQ-U (USB), DAQ-M (Bluetooth) a DAQ-E a DAQ-E-S (Ethernet): kalibrované spektrá v reálnom čase, `.daq` záznamy a osvetlenie smerujúce nadol na spracovanie odrazivosti. Pozrite si [sekciu DAQ](daq/README.md).

{% hint style="success" %}
**Novinky v Chloros 1.3.0**: Asistenti umelej inteligencie môžu ovládať Chloros prostredníctvom `chloros-mcp` a rovnaké funkcie ponúkajú aj CLI, SDK a `chloros-mcp`. Produkty s indexom LATTICE boli vylepšené, polia so všetkými senzormi sú rýchlejšie a fungujú v akejkoľvek sieti, pričom backend pokračuje v prevádzke aj medzi jednotlivými príkazmi. Pozrite si zoznam noviniek nižšie a [stiahnite si](download.md) inštalačné súbory.
{% endhint %}

{% hint style="info" %}
**Používate Chloros s AI asistentom?** Táto príručka je na to určená. Nasmerujte svojho asistenta na:

* `https://mapir.gitbook.io/chloros/llms.txt` — strojovo čitateľný index všetkých stránok.
* Akúkoľvek stránku vo formáte surového Markdownu — pripojte k jej URL u `.md` (napr. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Referenčná príručka CLI](reference/cli-reference.md) a [Referenčná príručka SDK](reference/sdk-reference.md) — kompletné referenčné stránky s presnými hodnotami napísané pre použitie veľkými jazykovými modelmi (LLM).

Príklad zadania: *„Prečítajte si https://mapir.gitbook.io/chloros/reference/cli-reference.md, a potom napíšte skript, ktorý sa prihlási a spracuje priečinok ~/flights/flight_001 na súbory GeoTIFF s odrazivosťou a NDVI.“*

Alebo nechajte priamo spustiť Chloros: `chloros-mcp`, server MCP nainštalovaný spolu s SDK, funguje s aplikáciami Claude Desktop, Claude Code, Cursor a ďalšími klientmi MCP.

Kompletný návod: [Používanie Chloros s AI asistentmi](ai-assistants.md).
{% endhint %}

***

## Novinky v Chloros 1.3.0

* **AI asistenti môžu ovládať Chloros.** `chloros-mcp`, server MCP nainštalovaný spolu s Python SDK, umožňuje aplikáciám Claude Desktop, Claude Code, Cursor a ďalším klientom MCP spracovávať obrazové údaje a ovládať kamery LATTICE, senzorové polia a svetelné senzory DAQ. Pozrite si [Používanie Chloros s AI asistentmi](ai-assistants.md).
* **CLI, SDK a `chloros-mcp` ponúkajú rovnaké funkcie.** Z bezpečnostných dôvodov zostávajú v CLI len prihlásenie, konfigurácia DAQ-E a priame DAQ servery. Niektoré skripty verzie 1.2.0 si vyžadujú úpravy: pozrite si časť **Zmeny vo verzii 1.3.0** v hornej časti [Referenčnej príručky k CLI](reference/cli-reference.md) a [Referenčnej príručky k SDK](reference/sdk-reference.md).
* **Vylepšené indexové produkty LATTICE.** NDVI a ostatné indexové produkty exportované zo záznamov LATTICE boli vylepšené vo všetkých režimoch spracovania.
* **Možnosti inštalátora pre Windows.**V časti**Systémové nastavenia**ponúka inštalátor možnosti**Povoliť dlhé cesty vo Windows**a**Vylúčiť backend Chloros z kontroly programom Microsoft Defender**, pričom obe sú štandardne zaškrtnuté. Prvá možnosť umožňuje, aby adresáre projektov mali názvy dlhšie ako 260 znakov; druhá zabraňuje spomaleniu snímania kamery v dôsledku skenovania v reálnom čase (približne o 30 % rýchlosti snímania pri vysokých rýchlostiach). Inštalačný program teraz tiež pridáva zložku CLI do `PATH` len raz a odstraňuje duplicitné záznamy, ktoré zanechali predchádzajúce inštalačné programy. Pozrite si [Stiahnuť](download.md).
* **Príliš hlboké projektové zložky sú odmietnuté hneď na začiatku.** Ak sú v systéme Windows vypnuté dlhé cesty (predvolené nastavenie systému Windows), projektová zložka, ktorá je príliš hlboká na limit 260 znakov v ceste, je odmietnutá ešte pred spustením akéhokoľvek procesu, pričom sa zobrazí najdlhšia projektová zložka, ktorá sa zmestí, a pokyny na zapnutie dlhých ciest. Predtým takéto spustenie nevytvorilo žiadne výstupy. Pri zapnutých dlhých cestách si produkty vo veľmi hlbokých zložkách teraz zachovávajú svoje metadáta (GPS a XMP tagy).
* **Masky obmedzenia.** Ak pri oddeľovaní pásiem produktu LATTICE (žiarivosť alebo odrazivosť) vznikne záporná hodnota pásma, táto hodnota sa obmedzí na nulu a maska na úrovni jednotlivých pixelov v zložke `clamp_masks/` vedľa zložiek s produktmi zaznamenáva, ktoré pixely a pásma boli ovplyvnené. Produkt pomenúva svoju masku v značke `Chloros:UnmixClampMask`. Živé náhľady sa teraz orezávajú presne tak, ako pri exporte. Pozrite si [Formáty výstupných obrázkov](output-image-formats.md).
* **Produkty sa opisujú samy.** Indexové produkty obsahujú svoje vlastné spracovateľské tagy (`Chloros:ProcessingLevel=index`, indexový výraz a informáciu, či boli vypočítané na základe radiancie alebo odrazivosti) namiesto tagov zdrojových surových dát. Produkty typu Survey3 obsahujú spracovateľské tagy typu Chloros vo všetkých režimoch spracovania. Surové a debayered súbory LATTICE si zachovávajú natívnu orientáciu senzora a zaznamenávajú ju v XMP; prehliadač obrázkov ich otáča iba na účely zobrazenia.
* **Balíčky pre Linux sa dodávajú v nezbalenom stave.** Backend a CLI sa inštalujú ako priečinky pod `/usr/lib/chloros/`, takže sa už pri každom spustení samy nerozbaľujú. Na x86_64 už backend nestrávi spúšťaním približne 100 sekúnd a súbor `.deb` má približne polovičnú veľkosť oproti predchádzajúcej verzii. Súčasťou balíka je ExifTool, ktorý teraz beží na čistej inštalácii Ubuntu. `.deb` teraz inštaluje aj Python SDK pre systém `python3` (nainštalujte ho pomocou `sudo apt install ./chloros_1.3.0_<arch>.deb`), zvyšuje veľkosť vyrovnávacích pamätí prijímania socketov v jadre, ktoré potrebujú prúdy z kamery, a pridáva backendovú službu, ktorá beží pod vaším používateľským účtom. Balík pre Jetson, `chloros_1.3.0_arm64_jp6.deb`, je zostavený pre JetPack 6 a spracováva dáta na GPU zariadenia Jetson pomocou CUDA verzie PyTorch pre JetPack. V systéme Linux sa každý exportovaný obrázok teraz dostane na disk len raz (najprv sa zapíše a označí v pamäti, potom sa skopíruje na miesto): na zariadení Jetson, kde tempo určuje disk, trvá spustenie LATTICE s 288 obrázkami 14 minút namiesto 24. Pozrite si [Inštalácia v systéme Linux](linux/linux-installation.md).
* **OCN kamery LATTICE merajú presnejšie na slnku.** Radiancia, odrazivosť a indexy prechádzajú rýchlou korekciou po rozložení pásiem, takže sivý cieľ vykazuje na slnku hodnotu blízku neutrálnemu NDVI. RGN kamery zostávajú bez zmien.
* **Hlásenie chýb priamo v aplikácii.** Ikona chyby v hornej časti karty Debug Log odošle spoločnosti MAPIR popis, až 10 snímok, protokoly a podrobnosti o systéme. Pozri [GUI: Navigácia](navigation.md).
* **LATTICE polia.**Polia si štandardne zachovávajú plný rozsah snímača a komprimovaný 12-bitový prenos (`--packed`, predvolené nastavenie `auto`) zvyšuje snímkovú frekvenciu pri plnom rozsahu snímača približne o tretinu; pole funguje v akejkoľvek sieti pri snímkovej frekvencii, ktorú táto sieť umožňuje (pomalé pripojenie znižuje snímkovú frekvenciu namiesto odmietnutia poľa alebo zmenšenia jeho snímok); pole, ktorého kamery prešli reštartom, sa samo obnoví s opätovným zapnutím PTP; automatická expozícia už neznižuje túto frekvenciu pri slabom osvetlení (najprv vyplní obdobie a potom zvýši zosilnenie); Neprerušované snímanie drží krok s rýchlymi poliami; ovládací prvok**Highlights** pre kamery RGB; a SDK získava `ArraySession`, `grab()`, `frame_stream()` a `continuous()`. Pozrite si [sekciu LATTICE](lattice/README.md).
* **Stabilnejšia automatická expozícia.** Expozícia sa pri konštantnom osvetlení udržuje a po zmene sa ustáli v priebehu niekoľkých sekúnd; malé svetlé miesta už scénu nezatmavujú; pri manuálnej expozícii zostáva zosilnenie nastavené na Auto; a kamera, ktorá prestane reagovať na priamom slnku, sa udrží na úrovni zodpovedajúcej jej osvetleniu, ktoré sníma svetelný senzor alebo, ak nie je pripojený, vlastné snímky kamery. Pozrite si [Nastavenia kamery](lattice/camera-settings.md).
* **Svetelné senzory.** **Podpora DAQ-E-S:** jej pevný difúzor nevyžaduje korekciu krytu a každé meranie obsahuje údaje o náklone, otáčaní a sklone zariadenia. Záznam uvádza, či bol kryt zvolený alebo predpokladaný, a spracovanie varuje v prípade predpokladaného krytu: modely DAQ-U, DAQ-M a DAQ-E sa dodávajú bez krytu, preto je potrebné uviesť názov namontovaného krytu. Záznamy v reálnom čase `.daq` prežijú výpadok napájania (uchovajte súbor `.daq-journal` spolu s jeho `.daq`), a zariadenie DAQ-E, ktoré prestane odosielať údaje, sa opäť pripojí s medzerou označenou v jeho zázname. Projekt, ktorý obsahuje iba záznamy zo svetelných senzorov, ich vypisuje a spracováva prostredníctvom aplikácie a CLI, pričom záznam vytvorený pomocou `chloros_scripts` je uvedený pod názvom DAQ, ktorý ho vytvoril, s obmedzením, ktoré si môžete nastaviť. Každé zachytenie ukladá hodnotu DAQ, ktorá sa pri ňom použila. Pozrite si [sekciu DAQ](daq/README.md).
* **Spracovanie.**Rámce LATTICE začínajú spracovanie počas detekcie cieľa. Referencie cieľa a DAQ sa určujú pre každé pásmo. Presunutý projekt opätovne prepojí svoje súbory. Exporty 8-bitových záznamov sú 8-bitové, v novom formáte**TIFF (8-bit)**. Spustenie programu Survey3 bez kalibračného cieľa sa uloží ako jeho náhradný produkt (`Vignette_Corrected_Images` alebo `Sensor_Response_Images`), nie ako odrazivosť. DLS tagy odrazivostného obrazu pre jednotlivé pásma sú vždy tagy jeho vlastného snímku. Funkcia Texture Aware je pre snímky LATTICE odmietnutá; na ne sa vždy uplatňuje štandardný debayer.
* **CLI a SDK.** Relatívne výstupné cesty teraz smerujú do priečinka, z ktorého spúšťate príkaz alebo skript (inštalovaný backend ich predtým riešil vo vlastnom inštalačnom priečinku, čo viedlo k zlyhaniu). Backend beží medzi príkazmi a skriptami a po 30 minútach nečinnosti sa sám zastaví; spravujú to `chloros-cli backend status` a `backend stop`. Príkazy na zachytávanie uvedú názvy všetkých súborov, do ktorých sa nepodarilo zapísať, a ukončia sa s kódom 1; opakované zachytávanie sa zastaví pri zaplnení disku. Úspešné spustenie `process` vygeneruje hlásenie `Image products written: N (plus M clamp masks)`. V systéme Windows funkcia SDK `ChlorosLocal` / `process_folder` vyhľadá nainštalovaný backend (a použije ten, ktorý už beží), a `read_image_audit_tags` funguje z bežného súboru `pip install`.

***

## Novinky vo verzii Chloros 1.2.0

* **Ovládanie kamery v reálnom čase — nová karta Kamery.** Pripojte kamery LATTICE po jednej alebo ako synchronizované sústavy viacerých kamier (časová synchronizácia PTP, snímanie spúšťané hardvérom), s prekrývaním živého náhľadu, histogramami pre jednotlivé pásma, inteligentnou automatickou expozíciou, kalkulátorom indexu v reálnom čase a aktualizáciami firmvéru kamery priamo v aplikácii.
* **Svetelné senzory — nová záložka „Svetelné senzory“.** Pripojte senzory DAQ-U (USB), DAQ-M (Bluetooth) a DAQ-E (Ethernet); zobraziť kalibrované spektrá v reálnom čase (W/m²/nm), zaznamenávať súbory `.daq` do vášho projektu, vyberať profily korekcie kapacity a aktualizovať firmvér DAQ-E cez sieť.
* **Režimy snímania a záznamníky.** Jednorazové / nepretržité / intervalové snímanie plus režim najrýchlejšieho snímania iba v surovom formáte; výber na úrovni projektu, ktoré kamery a typy exportu vytvára funkcia „Capture All“; maticové záznamníky pre indexové video na úrovni monitorovania a surové série na úrovni analýzy s offline zostavovaním videa.
* **Spracovateľský reťazec LATTICE.** Importujte zložky so záznamami LATTICE a každý surový snímok rozložte na produkty bez bayeringu, náhľad, radianciu float32 (W/m²/sr/nm) a odrazivosť s prepínačmi pre každý produkt. Reflektancia môže pochádzať z kalibračného cieľa v rámci snímky alebo z DAQ downwellingu; na exporty sa uplatňuje zarovnanie matice; chýbajúca továrenská kalibrácia sa automaticky stiahne podľa sériového čísla kamery.
* **Projekty si pamätajú hardvér.** Pripojené kamery a svetelné senzory sa ukladajú spolu s projektom (`cameras.json` / `sensors.json`) a pri opätovnom otvorení projektu sa opäť pripoja so svojimi uloženými nastaveniami. Pozrite si [GUI: Projekty](projects.md).
* **Vylepšenia prehliadača obrázkov.** Zobrazenie pixelov/indexu kurzora so správnym škálovaním odrazivosti pre každý súbor, histogramy vrstiev, posuvník pre zlučovanie GSD, režimy mriežky „Per Trigger“ / „Per Camera“, zobrazenia produktov LATTICE a export indexu/LUT do sandboxu na disk.
* **Výrazne rozšírené rozhranie CLI a SDK.** Nové rodiny príkazov `lattice`, `daq pool-*`, `project` a `time-sync`; nové možnosti `process` (`--input-level`, prepínače pre jednotlivé produkty, `--reflectance-source`, príznaky zarovnania poľa); inteligentné pripojovacie identifikátory SDK (`connect_camera` / `connect_array` / `connect_daq_sensor`), ktoré automaticky spúšťajú backend; `open_project()` automatizácia; balík SDK je súčasťou inštalačných programov a je zverejnený na PyPI ako `chloros-sdk`.
* **Prehľadná sémantika chýb.** Spustenie `chloros-cli process`, ktoré požiadalo o produkty, ale žiadny nezapísalo, teraz jasne signalizuje zlyhanie a ukončí sa s nenulovým kódom; úspešné spustenia hlásia, koľko obrazových produktov zapísali.
* **Nové usporiadanie výstupu.** Produkty sa ukladajú do zložiek `<project>/<camera>/<format>/<Product>_Images/` a zachovávajú názov zdrojového súboru — produkt identifikuje zložka, nie prípona súboru. Pozrite si [Formáty výstupných snímok](output-image-formats.md).
* **Viac vstupov, plánov a jazykov.** Podpora vstupov `.dng`; všetkých 38 jazykov rozhrania je plne obsadených; hardvérové obmedzenia na plán s bezplatným (bez prihlásenia) využitím až 4 kamier a 2 svetelných senzorov.
* **Spoľahlivosť.** Funkcia „Zastaviť spracovanie“ sa ukončuje bezchybne s presným súhrnom behu, projekty s viacerými kamerami exportujú údaje z každej kamery a aktualizácie inštalátora vás už neodhlásia.

***

Aplikácia Chloros je k dispozícii v 3 rozhraniach:

## Chloros: Desktopová aplikácia s grafickým rozhraním

Samostatné okno so všetkými funkciami, vrátane kariet „Kamery“ a „Svetelné senzory“ v reálnom čase. _Iba pre Windows._

## [Chloros CLI: Rozhranie príkazového riadku](CLI.md)

Hromadné spracovanie z príkazového riadku plus príkazy v reálnom čase `lattice`, `daq pool-*`, `project` a `time-sync`. Ideálne pre automatizáciu, skriptovanie a prevádzku bez grafického rozhrania. K dispozícii na **Windows, Linux amd64 a Linux arm64 (NVIDIA Jetson)**. _Pre prístup k CLI je potrebná platená úroveň Chloros+._

## [Chloros API: Python SDK](api-python-sdk.md)

Programové rozhranie v jazyku Python pre automatizáciu a vlastné pracovné postupy: spracovanie celého potrubia, relácie s živými kamerami/poliami, relácie so senzormi DAQ a automatizácia uložených projektov. Inštaluje sa spolu s balíkom pre desktop/CLI a je tiež publikované ako `pip install chloros-sdk`. _Pre prístup k API je potrebná platená úroveň Chloros+._

***

## Podporované platformy

| Platforma | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Áno | Áno | Áno |
| **Linux amd64 (x86_64)** | Nie | Áno | Áno |
| **Linux arm64 (NVIDIA Jetson)** | Nie | Áno | Áno |

Pokyny na inštaláciu v systéme Linux nájdete v časti [Linux a Edge Computing](linux/linux-overview.md).

***

## Začíname v troch krokoch

1. **Inštalácia** — stiahnite si a spustite inštalačný program pre vašu platformu. Pozrite si [Stiahnutie](download.md).
2. **Prihláste sa (voliteľné pre grafické rozhranie)** — grafické rozhranie spracováva obrázky zadarmo aj bez účtu. [Prihlásenie do Chloros+](chloros+-login.md) odomkne paralelné spracovanie, akceleráciu GPU, vyššie limity zariadení a prístup k CLI a SDK.
3. **Vytvorte svoj prvý projekt** — otvorte Chloros, vytvorte [nový projekt](projects.md), [pridajte svoje obrázky](processing-images-gui/adding-files-to-a-project.md) a [začnite spracovanie](processing-images-gui/starting-the-processing.md). Ak chcete namiesto toho ovládať hardvér v reálnom čase, otvorte kartu Kamery alebo Svetelné senzory — pozrite si [GUI: Navigácia](navigation.md).

***

## Chloros


Hoci je Chloros pre väčšinu úloh k dispozícii zadarmo, možno zistíte, že potrebujete viac. V takom prípade vám môže priniesť výhody platená licencia pre Chloros+. S licenciou Chloros+ môžete odomknúť nové funkcie, ako napríklad:

* **Viacvláknové spracovanie**: výrazne zrýchlite spracovanie obrazu pri väčších projektoch súbežným spracovaním obrázkov v spracovateľskom reťazci.
* **Akcelerácia GPU (CUDA)**: využite dnešné možnosti väčšej pamäte GPU na ďalšie zrýchlenie spracovateľského reťazca obrazu. Pre dosiahnutie najlepších výsledkov odporúčame 4 GB alebo viac VRAM.
* **Chloros+**[**CLI**](CLI.md) **Prístup**: spúšťajte Chloros+ z príkazového riadku na automatizáciu a integráciu do vášho vlastného softvéru. K dispozícii v každej platenej úrovni; vynucované na strane servera.
* **Chloros+**[**API**](api-python-sdk.md) **Prístup:** spúšťajte Chloros+ z Python u na programové ovládanie, čo umožňuje plynulú integráciu s vašimi výskumnými procesmi, pracovnými postupmi analýzy údajov a vlastnými aplikáciami. K dispozícii v každom platenom balíku; vynucované na strane servera.
* **Vyššie hardvérové limity**: pripojte naraz viac kamier a svetelných senzorov. Bez prihlásenia sa grafické používateľské rozhranie pripojí k maximálne 4 kamerám a 2 svetelným senzorom DAQ; platené úrovne zvyšujú oba limity:

| Úroveň | Kamery | Svetelné senzory DAQ |
| --- | --- | --- |
| Iron (bezplatný, bez prihlásenia) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Použitie na viacerých zariadeniach**: každá licencia Chloros+ umožňuje registráciu 2 a viac zariadení. Na správu registrovaných zariadení použite svoj účet MAPIR Cloud. Podporu pre ďalšie zariadenia získate aktualizáciou licencie Chloros+.
* **Pokročilá metóda debayeringu zohľadňujúca textúru:** vysokokvalitný debayering zohľadňujúci hrany v kombinácii s modelom odšumovania založeným na umelej inteligencii a strojovom učení, ktorý odstraňuje takmer všetok šum spôsobený debayeringom.
* **Vlastné vzorce multispektrálnych indexov:** zadávajte vlastné multispektrálne indexy do rastrových kalkulátorov Chloros, a to ako pre spracovanie, tak aj pre testovacie prostredie na prezeranie obrázkov.
* **Linux a edge computing:** spúšťajte Chloros na platformách Linux x86_64 a ARM64, vrátane NVIDIA Jetson, pre spracovanie v teréne a na okraji siete. Pozrite si [Prehľad Linux u](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Ceny a registrácia do Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->

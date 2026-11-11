# Referenčná príručka k Chloros CLI

**Verzia:**

1.3.0**Vytvorené:**

29. 7. 2026 o 19:19 ·**Revidované:**

6. 11. 2026**Cieľová skupina:** Optimalizované pre použitie LLM; čitateľné pre ľudí.**Rozsah:** Všetky podpríkazy nástroja `chloros-cli` určené pre používateľov, vrátane volieb a príkladov, ktoré je možné skopírovať a vložiť.

Tento dokument je kompletnou referenčnou príručkou pre nástroj príkazového riadku `chloros-cli`, ktorý je súčasťou balíka MAPIR Chloros. Je zámerne vyčerpávajúci, aby LLM (alebo človek) mohol zostaviť akýkoľvek podporovaný pracovný postup z nižšie uvedených zoznamov bez nutnosti prezerať zdrojový kód.

Ak potrebujete len to najdôležitejšie, prejdite na:
- [Päťminútový rýchly štart](#five-minute-quickstart)
- [Pracovný postup prvého pripojenia kamery LATTICE](#lattice-camera-first-connect-workflow)
- [Pracovný postup prvého pripojenia snímača DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Režimy snímania, záznamníky a offline spracovanie](#capture-modes-recorders--offline-reprocess)

> **Ovládate Chlorosa pomocou asistenta umelej inteligencie?** Claude Desktop, Claude Code, Cursor a ďalšie klienty MCP môžu spúšťať spracovanie, kamery a polia LATTICE, ako aj svetelné senzory DAQ prostredníctvom `chloros-mcp`, server MCP, ktorý sa inštaluje spolu s Python SDK (a s balíkom .deb pre Linux, v `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. Nastavenia, možnosti a nástroje nájdete v referenčnej príručke SDK v časti [MCP Server (AI asistenti)](sdk-reference.md#mcp-server-ai-assistants). Asistent s príkazovým riadkom môže tiež priamo spúšťať každý príkaz na tejto stránke.

> **Zmeny vo verzii 1.3.0** (čo si všimne existujúci skript alebo pracovný postup verzie 1.2.0).
>
> **Zmeny, s ktorými sa skript verzie 1.2.0 môže musieť vysporiadať:**
>
> - **Platí `--preset`.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` a `align-stream` boli v verzii 1.2 vynechané.0 a fungovali na základe vlastných nastavení kamery. Teraz nastavuje formát predvoľby, expozíciu, zosilnenie a snímkovú frekvenciu: `--preset high_speed` zaznamenáva 8-bitové snímky pri 5 ms, 6 dB, 60 fps. `lattice cam-connect --preset` tiež zachováva formát predvoľby (verzia 1.2.0 nastavila kameru na 12-bitový režim po uplatnení predvoľby). Pozri [Bežné možnosti](#common-options-shared-by-most-subcommands).
> - **Formát Bayerových pixelov na kamere M3M (mono) predstavuje jej hĺbku.** Prednastavenia uvádzajú formáty Bayer, ktoré mono senzor nedokáže zaznamenať: vo verzii 1.2.0 kamera odmietla tento formát a zachovala ten, ktorý mala, takže `--preset high_speed` tam nebol 8-bitový. Teraz je to formát Mono s rovnakou hĺbkou: `high_speed` a `--pixel-format BayerRG8` poskytujú `Mono8`, `high_quality`, `triggered` a `BayerRG12` dávajú `Mono12`. V zmiešanom poli M3C + M3M každá kamera používa zdieľaný formát vo svojej vlastnej rodine a projekt `network-analysis` / `analyze-array --pixel-format` premietajú každú kameru v danej hĺbke. To isté platí aj naopak: M3C, pre ktorú bol požiadaný formát Mono, preberá formát Bayer danej hĺbky, takže pole, ktorého hlavnou kamerou je M3M, už svoje kamery M3C v režime Mono neotvára. Pozri [Jednopásmové kamery Mono (M3M)](#mono-m3m-single-band-cameras).
> - **Exporty `lattice array-capture` sú štandardne zarovnané, rovnako ako v aplikácii.** Súbory `debayered`, `display`, `radiance`, `reflectance` a indexové súbory sú deformované do rámca hlavnej kamery a orezané na prekrývanie členov (menšie ako senzor o posuny kamier, `XMP-mapir:AlignmentJSON` `"applied": true`), takže záznam z CLI obsahuje rovnaké súbory ako záznam tej istej scény z aplikácie Chloros; `raw` zostáva v pôvodnom formáte. Predtým sa pri zázname bez `--aligned` každý súbor ukladal v pôvodnej veľkosti, bez deformácie. `--no-aligned` to zachováva. To isté platí pre `--smart`, `--continuous` a `--interval`, ako aj na SDK a `chloros-mcp`. Pozri [režimy `array-capture`](#array-capture-modes).
> - **`lattice array-capture --compression` má predvolené nastavenie `none`.** Súbory sa zapisujú v nekomprimovanej podobe (približne 6,3 MB na snímku v plnom rozlíšení) pre najrýchlejšiu trvalú rýchlosť; verzia 1.2.0 zapisovala formát `deflate` (približne 4,1 MB). Pre menšie súbory použite `--compression deflate`. Oba formáty sú bezstratové.
> - **Formáty PNG a JPEG podporujú iba produkty typu `display` / `preview` a indexové produkty.** Zachytenie formátu `--format png` alebo `jpg` s úrovňou, ktorú 8-bitový súbor nedokáže pojmúť, bude odmietnuté ešte pred začatím zachytávania (kód HTTP 400, ukončenie `1`), rovnako ako predvolené nastavenie `--processing all` s `--format png`/`jpg`. Pozri [Úrovne exportu zachytených údajov](#capture-export-levels-the-all-default).
> - **`lattice dls` vyžaduje `--mac`.**Východisková adresa MAC už neexistuje (bola to adresa jednej jednotky); `daq pool-discover --only ble` vyhľadáva adresu senzora.**`lattice dls --camera SERIAL`** je nový.
> - **Ďalšie chyby ukončia `1`.** `process`, ak sa nedali overiť exporty behu alebo došlo k strate obrazu, hoci ostatné produkty sú zapísané; príkaz zachytenia s akýmkoľvek súborom, ktorý sa nepodarilo zapísať (ostatné súbory sa stále uložia); `lattice continuous` a `multi-stream` pri zaplnenom disku; `array-burst` / `array-record`, keď rekordér nahlási chybu; `array-build-video`, keď bol požadovaný produkt preskočený alebo nebol vytvorený podľa požiadavky; `project burst` / `stream`, keď zhotovenie zlyhá v priebehu procesu; `project sensor log`, keď záznam neobsahuje žiadne údaje; `lattice exposure --set`, `gain --set` a `configure --set` s hodnotou, ktorú kamera odmieta. `lattice network-analysis --json` sa ukončí rovnako ako súhrn (`1` pre `needs_force_slip`). Naopak: `daq pool-disconnect` senzora, ktorý už nie je v skupine, ukončí `0`X. Pozri [Kódy ukončenia](#exit-codes).
> - **Pomalý `lattice array-connect` nikdy nezlyhá, ak je pole pripojené.** Čaká až 600 s (180 s vo verzii 1.2.0; `project connect` 90 s), pričom postupne vypisuje kroky backendu, a ak sa pripojenie do tej doby neuskutoční, backend ho zruší — každá kamera sa vráti do stavu, v akom bola nájdená — takže kód `Error:` (ukončenie `1`) znamená, že žiadne pole nie je pripojené. Vo verzii 1.2.0 mohlo pripojenie bezprostredne po zapnutí kamier pretrvať dlhšie ako čakacia doba a pripojiť pole aj po zlyhaní príkazu. Kombinácia klávesov Ctrl+C teraz zruší pripojenie. Pozri [Trvalé pripojenia](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` zachováva celý senzor.** Bez `--width` / `--height` / `--binning` je každá kamera nastavená na plný senzor a snímková frekvencia klesne na úroveň, ktorú poskytuje tento počítač a sieť; výstup prihlásenia uvádza rýchlejšie možnosti (binning alebo menšia oblasť záujmu) spolu s ich predpokladanými rýchlosťami. Verzia 1.2.0 zmenšila alebo zoskupila snímky, aby udržala rýchlosť. Pozri [Možnosti `array-connect`](#array-connect-options).
> - **Pomalá sieť už nebráni prevádzke poľa.** Každá kamera je prispôsobená svojej podielovej šírke pásma a snímková frekvencia sa tomu prispôsobí. Ak kamery nemožno spomaliť natoľko, pole beží v režime sériového vysielania (`slip-emit-and-capture`): každý snímok dorazí, ale kamery nie sú exponované súčasne. `--force-tier sim-capture-sim-emit` trvá na simultánnej expozícii. Pozri [FPS poľa a model sériového snímania](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` určujú, aké nastavenie connect sa uplatní**: bez `--pixel-format` sú formáty nastavené v connect (Mono12 / BayerRG12, predtým BayerRG8) a rozhodnutie o [komprimovanom 12-bitovom prenose](#packed-12-bit-transport---packed). Ten istý príkaz môže preto uvádzať inú snímkovú frekvenciu ako vo verzii 1.2.0.
> - **Relatívne cesty sa vzťahujú na miesto, kde spustíte príkaz.** `-o` a ostatné cesty, ktoré otvára backend (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), `output` kroku receptu, a predvolené `output/` z `lattice index --live` sa vyhľadávajú voči priečinku, z ktorého spúšťate `chloros-cli`. Vo verzii 1.2.0 ich backend riešil vo vlastnej inštalačnej zložke, čo v prípade nainštalovanej verzie viedlo k chybe „Prístup je zamietnutý“. Pozri [Konvencie](#conventions).
> - **`lattice index` beží na backende v každej podobe**, rovnako ako ostatné príkazy `lattice`, preto najprv spustite backend. Offline index (`--input`) vypočítava backend, ktorý načíta súbor TIFF a výsledky zapíše vedľa neho alebo do súborov `-o`, a príkazy `--list-presets` / `--list-gradients` vypíšu tabuľky backendu. V systéme Windows teraz fungujú: predtým Windows CLI odpovedalo na každú formu `lattice index`, vrátane `--live`, s `Error: lattice_sdk not available`. V Linuxe sa offline index spúšťal v rámci CLI bez backendu a bez prihlásenia; teraz potrebuje oboje, rovnako ako každý iný príkaz CLI. Pozri [Index / Vegetation Maths](#index--vegetation-maths).
> - **Nové súbory vedľa produktov.** Každý produkt typu radiancie, odrazivosti a indexu, v ktorom bolo v rámci oddelenia pásiem obmedzené niektoré pixel, dostane masku v priečinku `clamp_masks/` vedľa priečinka produktu a v výstupe sa vytlačí riadok `clamp mask:`. Pozri [Maska obmedzenia pri oddeľovaní kanálov](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Orientácia kamery.** Záznam z jednej kamery uloží súbory `raw` a `debayered` tak, ako ich snímač prečítal, a zaznamená otočenie/rotáciu kamery do súboru XMP; súbor `process` to aplikuje na vytvorené produkty. Súbory členov poľa zostávajú v pôvodnom formáte, pokiaľ neprevediete nový `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Pozrite si [Orientácia fotoaparátu](#ako-vyzerá-zložka-s-zachytenými-snímkami).
> - **Názov úrovne exportu, ktorý nie je „one“, bude odmietnutý.** Zoznam `lattice capture --processing` a všetky zoznamy typu `--levels` obsahujú položky `raw`, `debayered`, `radiance`, `reflectance` alebo `display` (alebo alias, napríklad `preview`; `--processing` a tiež `all`). Akýkoľvek iný názov je odmietnutý ešte pred dotykom s kamerou (HTTP 400 `UNKNOWN_LEVEL`, ukončenie `1`); 1.2.0 uložil pre ňu `raw` a ukončil činnosť `0`.
> - **`daq pool-set-cap` potrebuje `--cap-id`.** Bez neho sa príkaz ukončí `2`; 1.2.0 odstránila korekciu krytu, čím zostal holý senzor. `--cap-id none` vyžaduje holý senzor.
> - **`lattice resolution --binning-mode` už neexistuje.** Zlučovanie je vždy nastavené na „Average“; príznak sa stále prijíma, vypíše varovanie a nevykoná žiadnu akciu. Pozri [Nastavenie senzora](#sensor-tuning).
> - **`process --debayer texture-aware` je odmietnutý pre záznamy LATTICE.** V prípade zložky obsahujúcej snímky LATTICE ukončí `process` po importe program `1`, pričom uvedie ich počet, ešte pred spracovaním čohokoľvek. Zábery z LATTICE vždy prechádzajú štandardným demosaikovaním, takže verzia 1.2.0 túto voľbu prijala a ignorovala; ani aplikácia Chloros ju pre ne neponúka. Pozri [`chloros-cli process`](#chloros-cli-process).
> - **Fotoaparáty sa vrátia do stavu, v akom boli nájdené — pokiaľ nie je nastavené `--keep-settings`.** `lattice array-disconnect`, každý prechodný príkaz (`multi-capture` bez poľa, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) a `lattice continuous` vrátia každú kameru do pôvodného stavu: jej spúšť a zapojenie synchronizačného vedenia, jej PTP a všetky nastavenia kamery, ktoré sa počas relácie zmenili (expozícia a zosilnenie, cieľ AE, limity a tlmenie, vyváženie bielej, úroveň čiernej, formát pixelov, zlučovanie a ROI, otočenie, snímková frekvencia). `--keep-settings` namiesto toho uloží nastavenia každej kamerynastavenia do vlastnej pamäte kamery, takže sa s nimi zapne; `--clear-settings` túto pamäť resetuje na továrenské nastavenia. V oboch prípadoch sú kamery ponechané v režime voľného chodu. Projekt nastaví svoje kamery podľa vlastných uložených nastavení, keď sa k nim pripojí. Pozri [Pamäť kamery](#camera-memory----keep-settings----clear-settings).
> - **Všetky režimy ukladajú projekt rovnakým spôsobom.** Akákoľvek zmena na kamere, poli alebo svetelnom senzore pripojenom v rámci projektu – z aplikácie, CLI, SDK alebo servera MCP – sa uloží do súboru `cameras.json` / `sensors.json`, a otvorenie projektu v akomkoľvek režime nastaví zariadenia podľa neho. Predtým projekt ukladala len aplikácia a jej ďalšie uloženie prepísalo všetko ostatné. Pozrite si [Jeden záznam projektu, každý režim](#one-project-record-every-mode).
> - **Súbory vytvorené z viacerých kamier uvádzajú názvy všetkých z nich.** Viacpásmový stack (`align-apply`, `align-stream`, `index --live --save-multiband`), živý index vytvorený z pásiem viacerých kamier a obrázok `composite/` zo zachytenia poľa obsahujú sériové číslo, model a kalibračný balík každej zdrojovej kamery vo forme zoznamov (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), pričom EXIF `SerialNumber` uvádza sériové čísla oddelené čiarkami. Predtým neobsahovali žiadnu kameru alebo len sériové číslo prvej kamery. Pozri [Súbory vytvorené z viacerých kamier](#files-made-from-several-cameras).
> - **Iba súbor `raw` obsahuje kalibračné značky kamery.** Sada kompatibilná s MicaSense (`BlackLevel`, `RadiometricCalibration`, súbor pre každý kanál `Band{R,G,B}RadiometricCalibration`, súbory `MicaSenseVignetting*`) a dátová časť `CalibrationJSON` popisujú surové hodnoty senzora, takže sú obsiahnuté iba v súbore záznamu `raw`. Všetky ostatné súbory – súbory zachytenia `debayered`, `display`, `radiance`, `reflectance` a indexové súbory, rámce `lattice continuous` a `multi-stream`, a každý produkt typu `process` — obsahuje identifikačné údaje kamery (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) a pri snímaní aj vlastný záznam snímky (`XMP-mapir:FrameJSON`: expozícia, zosilnenie, teplota, formát pixelov, čas snímania). Vo verzii 1.2.0 obsahovali aj súbor surových údajov, ktorý nástroj pri ich spracovaní (Pix4D, Metashape) aplikoval na pixely, ktoré nie sú surovými hodnotami. `process` vyhľadá kalibráciu takéhoto súboru podľa jeho sériového čísla a `CalBundleSha256`.
>
> **Ďalšie zmeny:**
>
> - **Automatická snímková frekvencia poľa sa prispôsobuje tomu, čo pole poskytuje.** Frekvencia, ktorá stráca viac ako 1 % svojich spúšťacích impulzov, sa zníži, pričom sa to posudzuje na základe viacerých dlhších pozorovaní namiesto jedného, a frekvencia, z ktorej bola znížená, sa občas opäť vyskúša. Frekvencia, ktorú nastavíte sami, zostane nezmenená.
> - **Automatická expozícia je stabilnejšia.** Udržuje sa pri konštantnom osvetlení, ustáli sa v priebehu niekoľkých sekúnd po zmene, ignoruje malé preosvetlené oblasti a obnoví funkčnosť fotoaparátu, ktorý prestal reagovať v jasnom slnku. Pozrite si [Ako funguje automatická expozícia](#how-auto-exposure-behaves).
> - **`process` počíta iba vaše snímky a nahrávky.** Pri opätovnom spustení na priečinku, v ktorom sa nachádza súbor `camera_cal/` projektu, sa tieto kalibračné súbory už nepočítajú („Nájdených 96 snímok“ v sade s 90 snímkami) a projekt, ktorý ich už zahrnul, ich vynechá.
> - **`lattice multi-test` vychádza zo stavu zapnutia kamier.** Na dobu spustenia zapne PTP kamier, počká, kým sa ich hodiny synchronizujú a ustália pred prvým spustením – každá kamera je synchronizovaná po dobu 120 s, približne 2 minúty od zapnutia (`--ptp-settle SECONDS` to nastaví), keďže práve synchronizované hodiny majú odchýlku až ~200 us po dobu približne 2 minút — a vráti PTP každej kamery do stavu, v akom ho našiel. Na kameru, ktorá je stále synchronizovaná z predchádzajúceho testu, sa už nečaká. V minulosti sa PTP vypínalo v sieti s jedným prepínačom, kde žiadne zariadenie DAQ-E neodpovedalo, a voľne bežiace hodiny každej kamery sa interpretovali ako hodiny PTP, ktoré sa nezosynchronizujú (`PTP is not locked`, pri každom behu). Pozri [Viac kamier (prechodné relácie)](#multi-camera-transient-sessions).
> - **Záznam `.daq` uvádza, či bol kryt zvolený alebo predpokladaný.** DAQ-U, DAQ-M a DAQ-E sa dodávajú bez krytu a Chloros nedokáže zistiť, či je kryt nasadený, takže senzor pripojený bez pomenovaného krytu dostane predvolene hodnotu `sunshine_cosine`, čo je *predpoklad*. Každý záznam Chlorosu (schéma 1.30) teraz ukladá `cap_id_source` vedľa `cap_id`: `operator`, ak ste krytku pomenovali (vrátane `none`), a `auto_default`, ak išlo o predvolené nastavenie. Pri spracovaní záznamu, pri ktorom sa krytka predpokladala, pričom nastavenie DAQ krytky v projekte zostalo na Auto, vypíše `[daq-dls] WARNING` s názvom súboru: holý DAQ-U bez korekcie slnečného žiarenia vykazuje približne 30-násobne vyššiu hodnotu. **Pomenujte limit, ktorý ste nastavili** pri pripojení, alebo nastavte limit DAQ projektu a varovanie zmizne.
> - **V systéme Windows sa už nezobrazujú okná konzoly pip.** Na počítači s nainštalovaným Pythonom spustili backend a každý z jeho obrazových pracovníkov pri spustení raz kód `pip`, a každý z nich otvoril okno konzoly. Teraz to už nerobia.
> - **Živé záznamy `.daq` prežijú výpadok napájania.**Záznam sa zapisuje s denníkom a úplnou synchronizáciou disku pri každom potvrdení, za senzorom, takže pomalý disk spomaľuje len súbor, nikdy však namerané hodnoty. Záznam zastavený uprostred odosielania (výpadok napájania, ukončenie) môže vedľa seba zanechať súbor `<name>.daq-journal`:**skopírujte oba súbory spolu.**Chloros pri prvom načítaní súboru vráti denník späť, takže sa záznam otvorí so všetkým, čo bolo potvrdené pred zastavením (stratia sa približne posledné 3 s); vo verzii 1.2.0 mohol výpadok napájania v nevhodnom okamihu spôsobiť, že celý súbor bude nečitateľný („obraz disku databázy je poškodený“).
> - **Záznam na vyťaženom disku už nezastaví senzor.** `daq pool-record` zaraďuje odčítania do fronty a zapisuje ich vo vlastnom vlákne; odčítanie, ktoré sa nepodarilo zapísať, sa započítava (`daq pool-list` zobrazuje `N write error(s)` s poslednou chybou) namiesto toho, aby došlo k zamrznutiu senzora (jeden zápis trval 8,4 s popri štyroch kamerách ukladajúcich súbory TIFF).
> - **`lattice multi-capture` bez poľa už nestráca snímky, ktoré uložil, keď vyprší časový limit spúšťača.** Vypršanie časového limitu spúšťača kamery (`SC_ERR_TIMEOUT -1011`, pozorované pri štyroch kamerách na jednom prepínači 1 GbE) ukončí beh v danej skupine snímok: už uložené skupiny sú uvedené v zozname s kódom `Saved:`, potom nasleduje chyba, ukončenie `1`. Vo verzii 1.2.0 to bola chyba HTTP500 bez zoznamu, hoci súbory boli na disku.
> - **Kamery spúšťané softvérom už nevynechávajú každý druhý spúšťací signál.** Kamera v režime softvérového spúšťania teraz zaradí do fronty spúšťací signál, ktorý dorazí počas jej čítania (alebo Chloros počká, kým bude pripravená), tak ako to už robili kamery synchronizované káblom.
> - **`daq pool-discover` označuje DAQ-E-S `DAQ-E-S`** v stĺpci `MODEL` (a `--json` v stĺpci `model`). Pre každý ethernetový senzor uvádzalo `DAQ-E`; ten, ktorý neodpovie na kontrolu identity do približne 2 s, je stále uvedený ako `DAQ-E`.
> - **Vylepšené indexové produkty LATTICE** (NDVI a ďalšie indexy) od verzie `process` vo všetkých režimoch spracovania.
> - **Kamery OCN: vylepšené produkty žiarivosti, odrazivosti a indexov pri slnečnom osvetlení.** Po oddelení pásiem sa uplatňuje korekcia flotily; kamery RGN zostávajú nezmenené a hodnoty OCN sa líšia od verzie 1.2.0. Platí to aj pre zábery OCN zhotovené v skoršej verzii, ak sa spracúvajú na počítači, ktorý obsahuje kalibráciu, s ktorou bol záber zhotovený (program Chloros si uchováva kalibráciu kamery po jej pripojení).
> - **`lattice continuous` zaznamenáva surový snímok.** Spustenie programu `tiff` vytvorilo 3-rovinné snímky, z ktorých `process` nemôže vychádzať. Jeho snímky sú teraz súborom `raw`, ktorý zapisuje zachytenie poľa: jedna mozaiková rovina s vlastnými hodnotami senzora, nekomprimovaná, s kalibračným paketom. Skript, ktorý z týchto súborov číta tri roviny, prečíta len jednu.
>
> - **Každé zachytenie ukladá hodnotu DAQ, ktorú použilo, rovnako ako to robí grafické používateľské rozhranie.** `lattice capture`, `lattice continuous` (jeden súbor `.daq` na jeden rámec), `array-capture` a `multi-capture` s pripojeným poľom (a SDK) zapisujú súbory `.daq` a jeho `.match.json` vždy, keď je DAQ viazaný alebo zaradený do skupiny, vrátane zachytenia iba v surovom formáte; vo verzii 1.2.0 sa surové alebo debayeredové zachytenie ukladá iba s `--force-daq`. `--no-daq` túto funkciu vypína, `--daq-tolerance-ms MS` nastavuje toleranciu zhody pre dané zachytenie a nový `daq pool-bind` priradí kameru alebo pole k DAQ z CLI. Snímka uložená pomocou `.daq` sa v `process` spáruje iba s kamerami, ktoré sú v nej uvedené. Záznam teraz čaká na ďalšie odčítanie z DAQ, aby mohol zlúčiť hodnoty z oboch strán expozície. Pozrite si [Čítanie DAQ, ktoré zachytáva záznam](#the-daq-reading-a-capture-saves) a [Čítanie DAQ ďaleko od expozície](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** pridáva do projektu index pre pásma každej kamery, pričom je zahrnutý aj nový projekt. Vo verzii 1.2.0 bol názov, ktorý projekt ešte neuvádzal, ignorovaný pomocou `Unknown index name(s) ignored`.
> - **`process` vedľa aktívneho poľa:**už nespúšťa nanovo backend, ktorý obsahuje kamery alebo DAQ, a čaká až 15 s na ukončenie bežiaceho behu.**Ctrl+C** zastaví beh v backende, nielen v CLI. Pozri [Poznámky k `process`](#notes).
> - **Prípad, keď je priečinok projektu príliš hlboký na to, aby spĺňal limit 260 znakov v ceste v systéme Windows**, je hneď odmietnutý s použitím najdlhšieho priečinka, ktorý sa zmestí, namiesto spustenia, ktoré nezapíše nič. Inštalátor pre Windows teraz ponúka možnosť „Povoliť dlhé cesty v Windows“ a vylúčenie backendu z Microsoft Defender, pričom obe možnosti sú štandardne zaškrtnuté. Pozrite si [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** spúšťa vlastné nepretržité zachytávanie backendu: riadok s priebehom každé 2 s, súhrn namiesto zoznamu jednotlivých súborov a ukončenie s nenulovým kódom v prípade akéhokoľvek nezapísaného súboru alebo zaplneného disku. Pozrite si [režimy `array-capture`](#array-capture-modes).
> - **Prvý `array-capture` po `array-connect`** čaká až 20 s na prvú skupinu snímok poľa namiesto toho, aby vrátil chybu HTTP 503 „Pole sa ešte zahrieva“.
> - **Expozícia poľa v oblasti záujmu (ROI):** malá oblasť záujmu (ROI) už neobmedzuje expozíciu člena poľa na dobu voľného chodu senzora (4,9 ms pri rozlíšení 768 × 304). Pozrite si [Expozícia na člene poľa](#exposure-on-an-array-member).
> - **Automatická expozícia už neznižuje automatickú snímkovú frekvenciu matice.** Matica synchronizovaná káblom beží s frekvenciou, ktorú umožňuje sieť a čítanie z kamier; automatická expozícia vyplní obdobie, ktoré táto frekvencia poskytuje, a potom zvýši zosilnenie. Ručná expozícia dlhšia ako toto obdobie (`lattice exposure --set`, `configure --set`) spomalí spúšť, aby sa prispôsobila tejto expozícii, namiesto toho, aby bola odmietnutá, a frekvencia sa obnoví, keď sa člen vráti do automatického režimu. Pozri [Expozícia na člene poľa](#exposure-on-an-array-member).
> - **Polia poskytujú menej neúplných skupín snímok** a menej chýb 503 („Žiadna nová synchronizovaná skupina snímok“): spúšťací vstup každej podriadenej kamery je teraz zbavený rušivých signálov.
> - **Komprimovaný 12-bitový prenos** (`--packed`, predvolené nastavenie `auto`) zvyšuje rýchlosť snímok poľa s plným snímačom približne o tretinu. Teraz sa používa `array-connect --packet-size` (verzia 1.2.0 ho akceptovala, ale ignorovala). Pozri [Komprimovaný 12-bitový prenos](#packed-12-bit-transport---packed).
> - **Pole, ktorého kamery sa po vypnutí a zapnutí napájania samy obnovia.** Po vypnutí a zapnutí napájania prepínača sa pole `array-connect`-otvorí kamery, keď odpovedia, a opäť začne streamovať s opätovne zapnutým PTP; vo verzii 1.2.0 sa natrvalo zastavilo a bolo potrebné nové `array-connect`. `array-status` hlási `ptp_enabled: False` (a `ptp_off:`), keď kamera číta PTP ako vypnuté, pričom predtým hlásilo nastavenie, s ktorým bolo pole pripojené. Pozri [Trvalé pripojenia](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` zaznamenáva bez otvoreného grafického rozhrania.** Ak je kombinovaný pohľad grafického rozhrania zatvorený, backend vykresľuje pohľad sám: NDVI poľaNDVI alebo index, ktorý zadáte pomocou nového `--expr`. Záznam, ktorý nezapíše žiadny snímok, teraz zlyhá (`no-frames: …`, ukončenie `1`). Predtým sa nahrávalo len vtedy, keď bolo otvorené kombinované zobrazenie grafického rozhrania, a nahrávanie, ktoré nezaznamenávalo nič, aj tak ukončilo program `0`. Pozri [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Kamera, ktorú už Chloros drží, sa neotvorí druhýkrát.** Príkaz, ktorý by ju chcel otvoriť, namiesto toho vráti odpoveď HTTP 409 `CAMERA_HELD`, pričom uvedie, kto ju drží a aký príkaz ju uvoľňuje. To platí aj pre kameru, ktorú práve používa iný príkaz (napr. `multi-test` alebo `multi-capture` otvárajúce svoje kamery, alebo `lattice capture`): druhý príkaz je odmietnutý, kým sa prvý nedokončí. Jeho opätovné otvorenie by mohlo zastaviť prenos držiteľa alebo spôsobiť zlyhanie backendu. Pozri [Trvalé pripojenia](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Dočasné príkazy pre viacero kamier odovzdajú všetky kamery alebo zlyhajú.** `lattice multi-capture` bez pripojeného poľa, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` a `index --live` ukončia činnosť, pričom `1` uvedie názov kamery, ktorá nedodala žiadny snímok. Prúdy preskočia skupinu snímok, v ktorej chýba kamera, a zastavia sa po troch po sebe idúcich; `multi-test` vytlačí jeden riadok na spúšť, `(no frame)` tam, kde kamera zlyhala; `align-stream` tiež ukončí `1` pri chybe úlohy. Každé spustenie potom vráti každú kameru do stavu, v akom ju našlo: jej spúšť a zapojenie synchronizačnej linky, jej PTP (`PtpEnable`) a všetky nastavenia kamery, ktoré beh zmenil – expozícia a zosilnenie, cieľ AE, limity a tlmenie, vyváženie bielej, úroveň čiernej, formát pixelov, zlučovanie a ROI, otočenie, snímková frekvencia – pokiaľ `--keep-settings` / `--clear-settings` (pozri [Pamäť kamery](#camera-memory----keep-settings----clear-settings)). Predtým tieto príkazy zachovali všetky snímky, ktoré prišli, a ukončili `0` (`align-stream` vygeneroval chybu úlohy ako `Note:`), a nechali každú kameru v režime `TriggerMode` `On`. Tiež nastavili expozíciu každej kamery na základe predchádzajúcej scény pred uložením snímky (najviac niekoľko sekúnd), pričom pripojené pole spustí každú kameru na: 40 multispektrálnych, 60 RGB (nastavuje to `--brightness N`) a ponechá ju počas celého behu; rovnako tak aj jediný príkaz `lattice capture` pre kameru, ktorú Chloros neovláda. Predtým bol prvý snímok každej kamery nasnímaný pri vlastnom limite expozície kamery, približne 100 ms, bez ohľadu na osvetlenie, a bol často orezaný. Pozri [Viac kamier (dočasné relácie)](#multi-camera-transient-sessions).
> - **Dočasné príkazy pre viac kamier zdieľajú sieť rovnakým spôsobom ako `array-connect` a otvárajú len `--serials`.** Každá kamera je prispôsobená svojmu podielu na hostiteľskom spojení, takže štyri kamery bežia na 1 Gbps spojení; predtým si štyri kamery medzi sebou vyžadovali 125 MB/s, všetko cez 1 GbE, a každý snímok strácal pakety. Pole, ktoré sieť nedokáže preniesť, je odmietnuté ešte predtým, ako začne akákoľvek kamera vysielať. `--serials` teraz vyberá kamery, pričom prvá spustí spúšťač; predtým každý príkaz otvoril všetky kamery v sieti bez ohľadu na to, čo uvádzal `--serials`. Porucha spôsobená stratou paketov GVSP sa teraz označuje ako taká, namiesto toho, aby sa pripisovala synchronizačnému káblu M8. Pozri [Viac kamier (dočasné relácie)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` ponecháva kamery vo voľnom režime, tak ako boli nájdené.** `TriggerMode`, `Off` a všetko ostatné tak, ako to zistilo pripojenie — nastavenia spúšťača a synchronizačnej linky (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP a nastavenia kamery, ktoré sa počas relácie zmenili – takže jedinýpríkaz pre kameru na bývalom členskom zariadení získa snímky naraz a hlavné zariadenie už nezostáva na `TriggerSource`, `Software`. `--keep-settings` uloží nastavenia relácie do pamäte každej kamery, `--clear-settings` túto pamäť vynuluje (pozri [Pamäť kamery](#camera-memory----keep-settings----clear-settings)). `lattice continuous` tiež nastaví kameru, ktorú zistí ako zapnutú, do režimu voľného chodu na dobu behu (a po jeho skončení ju vráti späť), a beh bez rámca ukončí `1`.
> - **`lattice configure --get` / `lattice exposure`** vypíše `note:` sskutočnou expozíciou alebo zosilnením počas prevádzky automatického režimu. Zamietnutý príkaz `lattice exposure --set` / `configure --set` udáva rozsah, ktorý kamera akceptuje, a namiesto vypnutia automatickej expozície obnoví automatický režim. Pozri [Nastavenie senzora](#sensor-tuning).
> - **Nové ovládacie prvky kamery:**`lattice highlights` (zníženie jasu bielych povrchov v zobrazení RGB) a `lattice exposure --brightness-auto`.**`lattice multi-detect`** vyhľadá káblové kamery počas jedného cyklu sondy a ako hlavnú kameru predvolí kameru RGB.
> - **Nové príkazy DAQ:** `daq pool-bind`; `pool-enable-ptp`; a pre vlastné nastavenia zariadenia DAQ-E `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` a `pool-update-firmware`. Pozrite si [Nastavenia zariadenia DAQ-E](#daq-e-device-settings).
> - **DAQ-E, ktoré prestane odosielať dáta, je opätovne pripojené backendom.** Keď prúd z DAQ-E v skupine utíchne (vypnutie a zapnutie prepínača, ktorý ho napája, prerušenie spojenia s kamerou), backend ho opätovne pripojí s pôvodnými nastaveniami a počas jeho neprítomnosti opakuje pokusy v postupne sa predlžujúcich intervaloch. Nahrávanie na ňom pokračuje v tom istom súbore `.daq` s medzerou označenou v súbore, a súbor `pool-record --stop` uvádza zoznam všetkých prerušení. Vo verzii 1.2.0 senzor zostal neaktívny až do súborov `pool-disconnect` + `pool-connect`, čo zároveň ukončilo nahrávanie. Pozri [DAQ-E, ktorý prestane odosielať dáta](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** hlási, či pakety PTP grandmastera dorazia do siete (`health`), pričom v prípade, že nedorazia, na stderr sa vypíše `WARNING:`. Pozri [`time-sync`](#chloros-cli-time-sync).
> - **Balík .deb pre Linux:** zvyšuje veľkosť vyrovnávacích pamätí prijímania socketov v jadre, ktoré potrebujú prúdy z kamery (pri predvolenom nastavení jadra dochádza k strate snímok a zníženiu snímkovej frekvencie), pridáva backendovú službu, ktorá beží pod vaším používateľským účtom, a `lattice network` kontroluje tieto vyrovnávacie pamäte. CLI nikdy nezastaví backend, ktorý beží ako služba, a pri aktualizácii sa povolené backendové služby spustia znova. Pozri [Spustenie backendu ako služby](#running-the-backend-as-a-service-headless) a [Linuxové hostiteľa — vyrovnávacie pamäte prijímania socketov](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` hlási, keď sa nepodarilo vyrovnať:**každý dôvod, prečo bol záznam uložený nevyrovnaný (chýbajúci profil vyrovnania, členy s rôznymi rozlíšeniami, pole nesynchronizované s káblom, rámec, na ktorý sa profil nedá zmapovať) sa vypíše ako riadok `Alignment:`.**`chloros-cli daq help`** je `daq --help` (ukončilo sa to ako `2`) a v Linuxe sú v nápovede uvedené príkazy priamo pre hardvér `daq`, , ktoré táto kompilácia obsahuje.
> - **Jetson:** balík je `chloros_1.3.0_arm64_jp6.deb` pre JetPack 6 a jeho spracovanie na GPU beží na Orine s CUDA PyTorchom od JetPacku. Pozri [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Konvencie

- Všetky príkazy začínajú predponou `chloros-cli`. V systéme Windows je binárny súbor `chloros-cli.exe`; v systéme Linux/Jetson je to `chloros-cli`.
- Voliteľné argumenty sú uvedené ako `--flag`. Povinné pozičné argumenty sú uvedené bez zátvoriek.
- Ak je uvedená predvolená hodnota, vynechaním príznaku sa použije táto hodnota.
- CLI je jednoduchý HTTP klient nad backendom Chloros (server na adrese `127.0.0.1:5000`, ktorý používa aj desktopová aplikácia). **Ktoré príkazy ho spúšťajú:**
  - Základné príkazy — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` a `selftest` — využívajú bežiaci backend alebo ho spustia, ak žiadny nebeží, a po dokončení ho opäť zastavia.
  - **Príkazy `lattice`, `project` a `daq` nikdy žiadny backend nespustia.** Potrebujú backend, ktorý už beží: backend aplikácie pre pracovnú plochu systému Windows, backendovú službu pre systém Linux (#running-the-backend-as-a-service-headless) alebo backend, ktorý spustíte sami (pozri [Spustenie backendu](#spustenie-backendu)). Ak takýto backend chýba, vypíšu hlásenie `Error: Chloros backend not reachable at http://127.0.0.1:5000` spolu s príkazom, ktorý ho spúšťa, a ukončia sa s hlásením `1`.
- Chyba `CHLOROS_BACKEND_URL=<url>` smeruje rodiny príkazov **`lattice`**,**`project`**a**`daq pool-*`** na iný backend (`lattice network-analysis` namiesto toho smeruje na `--backend-url`). Hlavné príkazy zámerne fixujú `http://127.0.0.1:<port>` a ignorujú ho (literál IPv4 zabraňuje penalizácii v systéme Windows vo výške približne 2 s na požiadavku pri prechode z `localhost` na `::1`). Pozri [Premenné prostredia](#environment-variables).
- **Relatívne cesty sa riešia vo vzťahu k priečinku, z ktorého spúšťate `chloros-cli`.** Backend je proces, ktorý otvára výstupné a vstupné priečinky, takže pred odoslaním cesty ho CLI prevedie na absolútnu: `-o` / `--output` (vrátane jeho predvoleného nastavenia, `output`), `--input`, `--burst-dir`, `--profile`, `--save` a `--mask`, a krok [receptu](#recipe-dsl) `output` (predvolený `output`)a predvolený priečinok `output` v `lattice index --live`. `-o captures/` preto zapisuje do `<current folder>/captures/` a vytlačené cesty sú absolútne. `~` je rozvinutý. Pri komunikácii so vzdialeným backendom (`CHLOROS_BACKEND_URL` označujúci iný počítač) sa cesty odosielajú tak, ako boli zadané, a riešia sa na danom počítači.
- Pre všetky volania SDK/CLI je vyžadované prihlásenie prostredníctvom účtu Chloros+ volania (spustite `chloros-cli login` raz na každom počítači; uloží sa do vyrovnávacej pamäte v súbore `~/.chloros/`).
- Príklady používajú cesty v systéme Linux; v systéme Windows nahraďte `/home/user/...` hodnotou `C:/Users/.../...`. Sériový port zariadenia DAQ-Uje v štýle `COM3` v systéme Windows a `/dev/ttyUSB0` / `/dev/ttyACM0` v systéme Linux.
- Prehľady zapisujú vzájomne sa vylučujúce príznaky ako `{--a | --b}` a voliteľné príznaky ako `[...]`.

---

## Prehľad na najvyššej úrovni

```
chloros-cli [global options] COMMAND [command options]
```

### Globálne možnosti

| Príznak | Popis |
| --- | --- |
| `--backend-exe PATH` | Prepíše automaticky detekovaný spustiteľný súbor backendu. |
| `--port N` | Port HTTPu backendu (predvolené: `5000`). Používajú ho iba základné príkazy (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); `lattice`, `project` a `daq` používajú `CHLOROS_BACKEND_URL` (predvolený `http://127.0.0.1:5000`). |
| `-v, --verbose` | Zapnúť podrobný výstup. |
| `--restart` | Vynúti reštart backendu (ukončí všetky bežiace inštancie `backend_server.py`). Backend spúšťaný službou systemd (Linux .deb) sa nikdy nezastaví: príkaz namiesto toho vypíše `sudo systemctl restart <unit>` a ukončí sa `1`. |
| `--version` | Vypíše verziu (`Chloros CLI 1.3.0`). |
| `--help` | Zobrazí pomoc najvyššej úrovne. |

### Zoznam príkazov

| Príkaz | Účel |
| --- | --- |
| [`process`](#chloros-cli-process) | Spracovať priečinok so záznamami z aplikácií Survey3 alebo LATTICE od začiatku do konca. |
| [`login`](#chloros-cli-login) | Overiť toto zariadenie pomocou účtu Chloros+. |
| [`logout`](#chloros-cli-logout) | Vymazať uložené prihlasovacie údaje z vyrovnávacej pamäte. |
| [`status`](#chloros-cli-status) | Zobraziť aktuálny stav licencie / overenia. |
| [`export-status`](#chloros-cli-export-status) | Zobraziť priebeh exportu Live Thread-4 počas behu `process`. |
| [`language`](#chloros-cli-language) | Nastaviť alebo zobraziť zoznam jazykov zobrazenia CLI (podporovaných je 38). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Predvolená zložka projektu (zdieľaná s grafickým rozhraním). |
| [`update`](#chloros-cli-update) | Vyhľadanie a inštalácia aktualizácií Chloros (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnostika systému + rýchle testy. |
| [`time-sync`](#chloros-cli-time-sync) | Stav/ovládanie PTP grandmaster. |
| [`lattice`](#chloros-cli-lattice) | Ovládanie a snímanie kamery LATTICE (viac ako 45 podpríkazov). |
| [`daq`](#chloros-cli-daq) | Ovládanie spektrálnych senzorov DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Otvorenie a spustenie uloženého projektu Chloros (kamery + DAQ). |

---

## Inštalácia

`chloros-cli` je súčasťou inštalačného balíka Chloros pre každú podporovanú platformu — nie je k dispozícii samostatné stiahnutie pre CLI. Inštaláciou balíka pre danú platformu sa `chloros-cli` pridá do vášho `PATH` spolu s desktopovou aplikáciou a binárnym súborom backendu, ktorý ovláda.

Najnovšie súbory na stiahnutie: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Inštalátor obsahuje aj praktické spúšťacie skripty (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), ktoré otvárajú pripravené rozhranie CLI.

### Windows (.exe)

1. Stiahnite si inštalačný program pre Windows zo stránky na stiahnutie.
2. Spustite súbor `Chloros Installer 1.3.0.exe` a postupujte podľa pokynov sprievodcu. Predvolená inštalačná cesta je `C:\Program Files\Chloros\` (CLI sa nachádza v `C:\Program Files\Chloros\cli\`, ktorú inštalačný program pridá do premennej PATH).
3. Otvorte nový terminál (`cmd.exe`, PowerShell alebo terminál systému Windows), aby sa načíta aktualizovaný súbor `PATH`.

```powershell
chloros-cli --version
```

Inštalačný program automaticky pridá `chloros-cli.exe` do vášho systémového `PATH` a zahrnie ovládače kamery LATTICE. Backend je `C:\Program Files\Chloros\backend\chloros-backend.exe` (zložka `backend` vo vašej inštalačnej zložke), ktorý sa nenachádza v `PATH`.

**Systémové nastavenia.** Stránka *Vybrať ďalšie úlohy* v inštalátore obsahuje v časti **Systémové nastavenia**dve možnosti, ktoré sú obe**štandardne zaškrtnuté**:

| Možnosť (ako je uvedené v inštalátore) | Čo robí | Vypnutie tichej inštalácie |
| --- | --- | --- |
| **Povoliť dlhé cesty v systéme Windows** (umožňuje, aby zložky projektov boli umiestnené hlbšie ako 260 znakov; odporúčané) | Nastaví systémovú hodnotu `LongPathsEnabled` v systéme Windows na hodnotu 1. Systém Windows sa dodáva s touto hodnotou vypnutou, a v takom prípade žiadna cesta nesmie presiahnuť 260 znakov; produkty Chloros sa nachádzajú približne 130 znakov pod zložkou projektu, takže hlbšia zložka projektu je odmietnutá (pozri [Riešenie problémov](#troubleshooting-pointers)). Zmena nadobúda platnosť pre programy spustené po inštalácii. Pri odinštalovaní sa ponechá zapnuté, pretože iné programy sa na to môžu spoliehať. | `/MERGETASKS="!longpaths"` |
| **Vylúčiť backend Chlorosu z kontroly programom Microsoft Defender**(oveľa rýchlejšie snímanie z kamery; odporúčané) | Pridá vylúčenie**procesu** Defender pre `chloros-backend.exe` v inštalačnom priečinku, takže ochrana v reálnom čase nekontroluje každý súbor, ktorý backend zapíše, bez ohľadu na to, kam ho zapíše. Žiaden iný program nie je vylúčený; vylúčenie sa pri odinštalovaní odstráni. Ak Defender chýba alebo je zablokovaný zásadami, tento krok sa ticho nezrealizuje a inštalácia pokračuje. | `/MERGETASKS="!defenderexclusion"` |**Prečo je voľba Defender dôležitá:** pri vysokej rýchlosti zachytávania stojí skenovanie zachytených súborov v reálnom čase približne 30 % rýchlosti zachytávania (merané pri nepretržitom zachytávaní z 9 kamier: 77,7 snímok/s bez Defenderu, 52,7 skenovaných). `lattice array-capture --continuous` vypíše upozornenie, ak zistí, že Defender pri snímaní využíva jedno alebo viac jadier procesora; spustite inštalátor znovu so zaškrtnutou touto voľbou.

Pri tichej inštalácii (`/SILENT` alebo `/VERYSILENT`) sa uplatňujú obe možnosti, pokiaľ nie je zvolené ich vypnutie, napr. `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Firewall systému Windows.** Inštalačný program vždy pridá dve pravidlá pre prichádzajúci prevádzku pre `chloros-backend.exe` do každého sieťového profilu a odinštalačný program ich odstráni: `Chloros Backend (UDP-In)` pre prúdy z kamier, PTP, mDNS a dátový tok DAQ-E, a `Chloros Backend (TCP-In)`, obmedzené na lokálnu podsieť, pre aktualizáciu firmvéru DAQ-E, pri ktorej sa senzor opäť pripojí k tomuto počítaču, aby získal obraz (pozri [Nastavenia zariadenia DAQ-E](#daq-e-device-settings)). HTTP API backendu zostáva viazané na `127.0.0.1`.

### Linux amd64 (.deb)

Pre pracovné stanice s Ubuntu 22.04 LTS alebo novším / x86_64 založené na Debiane.

> **Ubuntu 20.04 nie je podporovaný.** Zoznam závislostí balíka je odvodený od
> toho, na čo sa backend skutočne odkazuje, a to zahŕňa aj `libc6 (>= 2.35)`;
> verzia focal obsahuje glibc 2.31. `apt` odmietne inštaláciu, namiesto toho, aby ju nechal zlyhať počas
> behu. Zoznam uvádza aj `libgl1`, ktorý načíta súčasťou balíka OpenCV a v minimálnom
> obrazu servera môže chýbať; `apt install` ho obsahuje.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

Inštalátor .deb nainštaluje:
- `chloros-cli` až `/usr/bin/chloros-cli` a `chloros-backend` (spustí backend) až po `/usr/bin/chloros-backend`
- skompilovaný backend a CLI, rozbalené, v `/usr/lib/chloros/backend/` a `/usr/lib/chloros/cli/`
- Ovladače kamier LATTICE
- Modely odšumovača, kalibračné balíky a konfigurácia kanála aktualizácií
- Pythonové SDK pre systém `python3` (`import chloros_sdk`). `apt install` so sebou prináša aj svoje požiadavky, `python3-numpy` a `python3-requests` (odporúčané, nie povinné: vyššie uvedené programy sú samostatné; `dpkg -i` ich preskakuje a výstup inštalácie uvádza, čo chýba). Vo virtualenv: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
– `/etc/sysctl.d/60-chloros-gige.conf`, ktoré zvyšujú veľkosť prijímacích vyrovnávacích pamätí socketov jadra pre prúdy z kamery prúdy kamery na 32 MB a aplikuje ich naraz (hodnota, ktorá je už nastavená na vyššiu hodnotu, zostane nezmenená). Odstránením balíka sa odstráni aj tento súbor. Pozrite si [Linuxové hostiteľa — prijímacie vyrovnávacie pamäte socketov](#linux-hosts--socket-receive-buffers).
- Dve voliteľné služby systemd pre backend, ani jedna z nich nie je povolená (ďalšia časť)

#### Spustenie backendu ako služby (bez grafického rozhrania)

Bez služby beží backend (a s ním aj PTP grandmaster) len dovtedy, kým ho niečo nespustí: `chloros-backend` v termináli, skriptom SDK alebo základným príkazom `chloros-cli` počas trvania jeho behu. Ak chcete mať backend neustále spustený na stroji bez grafického rozhrania, aktivujte službu **ako váš používateľ**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` spustí backend ako tento používateľ. Záznamy, ktoré zapisuje, patria tomuto používateľovi, jeho podrobný protokol sa nachádza v jeho `~/.cache/chloros/logs`, a používa jeho súbor `chloros-cli login` (`~/.chloros`), takže sa najprv raz prihláste ako tento používateľ. PTP nepotrebuje oprávnenia root. Rovnako ako každý nainštalovaný backend, načúva iba na porte `127.0.0.1:5000`.

`chloros-backend.service` spúšťa backend ako **root**. Stále je nainštalovaný tak, aby stroj, na ktorom je povolený, naďalej bežal, ale všetko, čo root backend zapíše, patrí rootovi, takže používateľ nemôže zmazať svoje vlastné záznamy. Ak sa chcete prepnúť na svojho používateľa:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

Podrobný protokol služby root sa nachádza v súbore `/var/log/chloros`, ktorý môžu čítať členovia skupiny `adm` (služba nastavuje premennú `CHLOROS_LOG_DIR`; pozri [Premenné prostredia](#environment-variables)). Inštalácia balíka vypíše príkaz `enable` s vyplneným vaším používateľským menom a tiež prepínač, ak je služba root povolená.

**Povoľte jednu backendovú jednotku.** Zároveň beží len jedna backendová jednotka. Druhá povolená jednotka (`chloros-backend.service` popri `chloros-backend@<user>.service` alebo dve používateľské jednotky `@`) sa odmietne spustiť a zostane v stave `failed` s výstupným stavom 98; systém systemd ju nespustí znovu. Jeho výstup (`systemctl status`) uvádza dôvod: `Another instance is already running. Exiting.`, potom to, čo zadržiava backend (port 5000 alebo súbor uzamknutia a používateľ, ktorému patrí), a potom `systemd will not restart <unit> (exit status 98)`. Po reštarte môže bežať ktorákoľvek z týchto jednotiek. Ak chcete prepnúť, deaktivujte tú, ktorú nepoužívate, ako je uvedené vyššie.**CLI nikdy nezastaví backend služby.** `chloros-cli` ho používa tak, ako je, rovnako ako GUI a SDK: `process` na ňom beží a načíta svoj vlastný projekt namiesto toho posledného. Tam, kde by CLIinak reštartovalo backend (`--restart` alebo `process`, ktorého import nenašiel nič), označí jednotku, vypíše `sudo systemctl restart <unit>` a ukončí `1`, pretože backend zastavený týmto spôsobom by zostal neaktívny. `GET /api/status` pomenuje jednotku ako `service` (`null` v prípade backendu, na ktorom nebeží žiadna služba).**Aktualizácia a odstránenie.** Aktualizácia balíka zastaví všetky služby backendu a následne opäť spustí každú, ktorá je povolená, pričom ako prvé sa spustia jednotky `@<user>` (jednotka spustená ručne, ale nikdy nepovolená, zostane zastavená, rovnako ako po reštarte). Ak je povolených viac služieb, beží len jedna a ostatné zostávajú v stave zlyhania, ako je uvedené vyššie; aktualizácia vypíše kódy `Chloros: N backend units are enabled: …` a `Keep one enabled: sudo systemctl disable --now <unit>`. Odstránenie balíka ich zastaví a deaktivuje. Vymazanie balíka tiež odstráni súbor `/var/log/chloros`.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Rovnaká štruktúra ako v prípade balíka .deb pre amd64, vrátane prijímacích vyrovnávacích pamätí a [služieb backendu](#running-the-backend-as-a-service-headless), pre Jetson Orin / Orin NX / Orin Nano na **JetPack 6** (L4T R36). Jeho PyTorch je CUDA verzia pre Orin z JetPack 6, takže spracovanie na GPU, vrátane debayera s podporou textúr, beží na GPU zariadenia Jetson. Balík neobsahuje samotnú CUDA: využíva knižnice CUDA a cuDNN, ktoré inštaluje JetPack 6 (`sudo apt install nvidia-jetpack` na obraze, kde chýbajú). Ak nie je k dispozícii žiadne GPU s podporou CUDA, spracovanie prebieha na CPU a funkcia zohľadňujúca textúry používa štandardný debayer (pozri [`process`](#chloros-cli-process)).

### Jednorazová autentifikácia na každom zariadení

Každá platforma vyžaduje jednorazové prihlásenie do Chloros+, aby fungovali volania SDK/CLI:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Prihlasovacie údaje sa ukladajú do vyrovnávacej pamäte v súbore `~/.chloros/user_session.json`.

### Spustenie backendu

Príkazy `lattice`, `project` a `daq` komunikujú s backendom, ktorý už beží; nikdy ho nespúšťajú (pozri [Konvencie](#konvencie)). Počas ich používania nechajte jeden bežať:

| Platforma | Ako |
| --- | --- |
| Windows | Otvorte desktopovú aplikáciu Chloros (jej backend obsluhuje aj CLI) alebo spustite príkaz `"C:\Program Files\Chloros\backend\chloros-backend.exe"` v samostatnom termináli (v priečinku `backend` vo vašom inštalačnom priečinku) a nechajte ho otvorený. |
| Linux / Jetson | Spustite `chloros-backend` v samostatnom termináli alebo nechajte jeden backend neustále bežať pomocou [backendovej služby](#running-the-backend-as-a-service-headless). |

Zároveň beží len jeden backend; všetci klienti (GUI, CLI, skripty SDK) ho zdieľajú. Základné príkazy (`process`, `login`, `status`, …) ho tiež používajú a spustia ho, ak žiadny nebeží.

**Backend beží nepretržite medzi jednotlivými príkazmi**, takže nasledujúci príkaz už nemusí čakať na jeho spustenie. Zastaví sa, keď sa odhlásite (`chloros-cli logout`, pokiaľ nie je otvorená aplikácia Chloros alebo prebieha spustenie alebo offline kompilácia), keď spustíte `chloros-cli backend stop`, alebo automaticky po 30 minútach, ak ho nič nevyužíva: žiadny príkaz, skript ani aplikácia, žiadna úloha ani offline kompilácia, žiadna pripojená kamera ani svetelný senzor. `backend stop` odmietne ukončenie, pokiaľ prebieha úloha alebo offline kompilácia, a uvedie jej názov; `--force` ju okamžite zastaví a táto úloha alebo kompilácia sa stratí (séria sa zachová a `array-build-video` ju zostaví znovu). Nastavte `CHLOROS_BACKEND_IDLE_MIN` pred spustením backendu, aby ste zmenili 30-minútový limit (`0`: nikdy); [služba backendu](#running-the-backend-as-a-service-headless) sa nikdy sama nezastaví. `chloros-cli backend status` ukazuje, či niečo beží, čo to používa a či beží nejaká offline kompilácia. `process` otvorí svoj projekt v bežiacom backende, pokiaľ nie je otvorená aplikácia Chloros s projektom, ktorý otvorila: v takom prípade to odmietne a musíte najprv spracovať z aplikácie alebo zatvoriť daný projekt. Projekt, ktorý zostal otvorený po predchádzajúcom spustení `process` alebo skripte SDK, sa jednoducho nahradí.

### Overenie inštalácie

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` ukončí činnosť (`0`) len vtedy, ak prejde všetkých sedem kontrol. Posledná z nich vyžaduje grafický procesor CUDA (grafický procesor NVIDIA, alebo Jetson Orin) a modely odšumovača, takže na počítači bez neho sa zobrazí `Failed: CUDA + Denoiser` a program sa ukončí na `1`; všetko okrem debayera s podporou textúr, ktorý je určený výlučne pre GPU, tam stále funguje.

> **Vyžaduje sa predplatné Chloros+.**CLI vyžaduje aktívny plán Chloros+.**Copper**je základná úroveň Chloros+ — každá platená úroveň Chloros+ má prístup k CLI/SDK; iba bezplatná úroveň**Iron** ho nemá. (Priradenie ID plánu: `0`=Iron/free, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Upgrade na [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Tento limit je vynucovaný na strane backendu, nielen prostredníctvom CLI: žiadosť označená SDK/CLI bez platného plánu je odmietnutá s kódom `403 PLAN_UPGRADE_REQUIRED`, bez ohľadu na to, či pochádza z `chloros-cli`, Python SDK alebo vlastného HTTP klienta. Odhlásený volajúci dostane namiesto toho kód chyby `401 AUTH_REQUIRED`. Prístup funguje offline počas tolerančného obdobia plánu (30 dní pri mesačnom pláne, až do vypršania platnosti pri ročnom pláne) a po jej uplynutí sa zastaví; kód `chloros-cli status` naďalej funguje, aby bol dôvod viditeľný (ide o jedinú trasu SDK/CLI, ktorá je oslobodená od obmedzenia podľa úrovne — `GET /api/license-status`).

---

## Päťminútový rýchly štart

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (krok 2) spustí backend pre svoje vlastné spustenie, ak žiadny nebeží, a zastaví ho po dokončení, takže spustite jeden pre krok 4 a ďalej. Pozrite si [Spustenie backendu](#start-the-backend).

---

## `chloros-cli process`

Spracujte priečinok s obrázkami prostredníctvom kompletného potrubia Chloros (detekcia cieľa → kalibrácia → vineta → odrazivosť → export indexu).

### Prehľad

```
chloros-cli process INPUT [OPTIONS]
```

### Polohové argumenty

| Argument | Popis |
| --- | --- |
| `INPUT` | Cesta k vstupnému priečinku obsahujúcemu súbory `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) alebo `.dng`. |

### Bežné možnosti

| Príznak | Predvolené nastavenie | Popis |
| --- | --- | --- |
| `-o, --output PATH` | nový priečinok s názvom `-n` vo výchozom priečinku projektu | Priečinok projektu. Projekt je pomenovaný podľa neho a zložka, v ktorej sa už nachádza súbor `project.json`, dostane názov `_1` / `_2` namiesto toho, aby bol prepísaný. Predvolenou zložkou projektu je tá, ktorú vybral `set-project-folder` (alebo grafické rozhranie), inak je to `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | automaticky (časová pečiatka) | Názov projektu, používa sa iba **bez** `-o` (s `-o` projekt preberá názov zložky). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` používa neurálny debayer Chloros+ pre snímky Survey3; je pomalší, ale poskytuje vyššiu kvalitu. Je určený iba pre Survey3: Zábery LATTICE vždy používajú štandardný demosaic, takže v prípade zložky obsahujúcej zábery LATTICE program `process` po importe odmietne program `texture-aware` s chybou uvádzajúcou počet týchto súborov a ukončí program `1` ešte pred spracovaním čohokoľvek (aplikácia Chloros pre ne tiež neponúka funkciu Texture Aware). Vyžaduje si grafický procesor s podporou CUDA (grafický procesor NVIDIA v systémoch Windows / Linux x64, grafický procesor Orin na Jetsone): bez neho spustenie používa `standard` a backend zaznamená jeden riadok `[DEBAYER] … needs a CUDA GPU …`, v ktorom uvádza dôvod (zobrazené s `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Korekcia vinety. |
| `--reflectance / --no-reflectance` | `--reflectance` | Kalibrácia odrazivosti (používa panelový cieľ, ak je nájdený, alebo sériovú kalibráciu NIST pre LATTICE). V prípade multispektrálneho LATTICE to slúži zároveň ako prepínač **produktu** odrazivosti — pozri [Prepínače exportu podľa produktu](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | vypnuté | Použiť korekcie PPK GNSS zo súborov sidecar. |
| `--exposure-pin-1 CAMERA_MODEL` | žiadne | Model kamery pripojený k expozíciovému pinu 1, pre časovú synchronizáciu PPK. |
| `--exposure-pin-2 CAMERA_MODEL` | žiadne | Model kamery pripojený k expozíciovému pinu 2, pre časovú synchronizáciu PPK. |
| `--recal-interval SECONDS` | `0` | Minimálny interval rekalibrácie v sekundách (*Minimálny interval rekalibrácie* projektu). |
| `--timezone-offset HOURS` | `0` | Časové posunutie svetelného senzora voči UTC v hodinách (projektové *Časové posunutie svetelného senzora*). |
| `--format FORMAT` | `TIFF (16-bit)` | Jedna z hodnôt `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Program Chloros nikdy nezvyšuje bitovú hĺbku obrázku: pri všetkých 8-bitových obrázkoch sa formát s väčšou hĺbkou zapisuje ako `TIFF (8-bit)` a v priebehu spracovania sa to uvádza raz. V projekte, v ktorom sa kombinujú 8-bitové a 12/16-bitové snímky, sa každý snímok exportuje vo svojej vlastnej hĺbke alebo v nižšej. Radiance je vždy 32-bitové s plávajúcou desatinnou čiarkou. |
| `--indices NAME [NAME ...]` | žiadne | Vegetčné indexy podľa názvu predvolby (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), bez ohľadu na veľkosť písmen. Každý z nich sa pridá do projektu pre príslušné spektrálne pásma danej kamery, pričom sa zahrnie nový projekt. Názov, ktorý nie je predvolbou, alebo ktorý nie je možné vytvoriť z príslušných spektrálnych pásiem kamery (napr. „`NDRE`“ vyžaduje RE a NIR, čo žiadna kamera samostatne neposkytuje), je uvedený vo varovaní a nedostane žiadny indexový obrázok. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Vynútiť vstupný bod spracovateľského reťazca pre súbory LATTICE TIFF (súbory Survey3.raw to neovplyvní). Tiež únikový mechanizmus, ktorý umožňuje spracovanie záznamu **bez surových dát** vôbec — pozri [Ako vyzerá priečinok so záznamami](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | zapnuté | Vygenerovať lineárny produkt po odstránení bayera (`Debayered_Images`). Pozri [Prepínače exportu pre jednotlivé produkty](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | zapnuté | Vysiela náhľad na displeji (`Preview_Images`): RGB = vyváženie bielej (DAQ-svetelný zdroj, ak je k dispozícii, inak šedý svet) + gama; multispektrálne = rozloženie falošných farieb. |
| `--radiance / --no-radiance` | zapnuté | Vysiela radianciu typu float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referencia pre produkt odrazivosti LATTICE: `auto` = cieľ v rámci snímky, ktorý prešiel kontrolou kvality (QA), je absolútnou referenciou (cieľ s vlajkou QA súťaží s DAQ o namerané skóre), záložná hodnota DAQ pre zostupné žiarenie (ρ = π·L/E); `target` = prísne (bez nahradenia DAQ); `daq` = rozhodujúci je DAQ. Pozri [Prepínače exportu podľa produktu](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | žiadne | Adresár **meraných** skenov cieľovej odrazivosti na jednotku (`<serial>.csv`); v prípade chýbajúcich údajov sa použijú nominálne spektrá T3/T4P. |
| `--array-alignment / --no-array-alignment` | zapnuté | LATTICE polia: uplatňujú zarovnanie medzi modulmi, ktoré je zaznamenané v súbore XMP každého záznamu `Chloros:Alignment*`, na každý spracovaný produkt (odstránenie matice / náhľad / žiarivosť / odrazivosť / index). Žiadna operácia pre snímky bez týchto značiek. |
| `--array-alignment-crop / --no-array-alignment-crop` | orezanie | Orezanie zarovnaných výstupov na oblasť spoločného prekrývania poľa tak, aby všetky moduly zdieľali jednu stopu; `--no-…` zachováva plnú plochu snímača (čierna výplň mimo zdroja). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Prevzorkovanie pre deformáciu pri zarovnaní. `nearest` zachováva presné zdrojové hodnoty DN (bez zmiešavania radiometrických hodnôt medzi pixelmi). |

### Možnosti detekcie cieľa

| Príznak | Popis |
| --- | --- |
| `--min-target-size PIXELS` | Minimálna veľkosť kalibračného cieľa v pixeloch pre detektor. |
| `--target-clustering 0-100` | Percento zhlukovania cieľov (predvolené `60`). |
| `--target / --targets` | Hľadať kalibračné ciele iba v podadresári `target` alebo `targets` vstupného súboru (rýchlejšie). |

### Príklady

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Prepínače exportu pre jednotlivé produkty (LATTICE multispektrálny)

Spracovanie LATTICE sa rozvetvuje na **všetky príslušné produkty v jednom cykle**. Štyri prepínače pre jednotlivé typy — `--debayered`, `--preview`, `--radiance`, `--reflectance` — sú všetky**štandardne zapnuté**; na vypnutie jedného z nich použite formulár `--no-<type>`. Hlavné kamery RGB vždy vysielajú iba debayered + náhľad (bez žiarivosti/odrazivosti podľa jednotlivých pásiem), preto sú pre ne prepínače `--radiance`/`--reflectance` bez účinku. Prepínače sa ignorujú pre Survey3 `.raw` (ktorý sleduje štandardnú cestu odrazivosti/cieľa). *(Starý príznak `--radiometric-output {reflectance,radiance,sensor-response}` bol **odstránený** a nahradený týmito prepínačmi; úroveň `sensor-response` už neexistuje.)*

| Produkt | Výstup | Je potrebný DAQ downwelling? |
| --- | --- | --- |
| `--debayered` | Lineárna demosaika (`Debayered_Images`): ten istý súbor, aký sa uloží pri zachytení na úrovni `debayered` — tmavý posun (DSNU) kalibračného balíka z surových údajov, potom demosaik s ohľadom na hrany pomocou OpenCV; bez LUT linearity, vyrovnania poľa ani oddelenia zložiek. | Nie. |
| `--preview` | Náhľad na displeji (`Preview_Images`): RGB = WB + gama; multispektrálne = rozťahovanie falošných farieb. | Č. |
| `--radiance` | float32 W/m²/sr/nm z kompletného radiometrického reťazca (`Radiance_Images`). | Č. |
| `--reflectance` | uint16 odrazivosť ρ (`32768` = 1,0), Pripravené pre Pix4D. | **Áno**, pokiaľ nie je ukotvené cieľom v rámci snímky, ktorý prešiel kontrolou kvality (pozri nižšie). |

`--reflectance-source` vyberá referenciu odrazivosti:**`auto`**(predvolené) nastaví cieľ v rámci snímky, ktorý prešiel kontrolou kvality, ako**absolútnu referenciu**— reťazce empirických čiar ukotvené cieľom sa krížovo porovnávajú na vyňatých paneloch a nameraný víťaz sa aplikuje pás po páse: pás, ktorý DAQ nedokáže kalibrovať, alebo ktorého prispôsobenie sklonu DAQ zlyhá, preberá vlastnú čiaru cieľa (`ReflectanceElBandMethods` určuje, ktorý) — cieľ, ktorý prejde kontrolou kvality s označením (vypadnutý panel, nefunkčné pásmo), súťaží na základe tohto nameraného skóre aj s deľiteľom DAQ a snímka, voči ktorej prehrá, je označená ako `ReflectanceChain=daq` — v prípade, že nie je prítomný žiadny cieľ alebo zlyhá kontrola kvality (QA), sa prejde na delenie DAQ na základe poklesu signálu (ρ = π·L/E);**`target`**je prísne (bez nahradenia DAQ);**`daq`**sa rozhodne pre správanie riadené DAQ. Geometria cieľa (ArUco / pevná oblasť záujmu / pás) pochádza z konfigurácie cieľa projektu; `--target-reflectance-dir DIR` uchováva**merané** skeny (`<serial>.csv`) vyhľadané podľa sériového čísla/QR cieľovej jednotky, s nominálnymi spektrami T3/T4P ako záložným riešením.

Cesta odrazivosti DAQ automaticky vyrieši **časovo zosúladené dopadajúce žiarenie**na základe zaznamenaného súboru**`.daq`**(DAQ-U/M/E)**alebo natívneho súboru DAQ-M `.csv`**nájdeného spolu so snímkami. Ak nie je kalibračný balík pre konkrétnu kameru alebo DAQ uložený v lokálnej cache, pipeline**ho pri prvom použití automaticky stiahne z AWS** (vyžaduje jednorazové pripojenie k internetu; uloží sa do vyrovnávacej pamäte pod `~/.chloros/`).**Čí je to údaj.**Hodnota `.daq`, ktorú zachytáva záznam, je údaj z kamery (kamier), ktorú (ktoré) popisuje — uvedená v doplnkovom súbore `<file>.daq.cameras.json` vedľa nej (`lattice capture`, `multi-capture`, `array-capture` a v grafickom rozhraní; surová séria uvádza hodnoty každého člena vo svojom zozname) — a `process` ich priraďuje**len k snímkam z týchto kamier**. Samostatná kamera bez vlastného záznamu si preto už nepožičiava DAQ inej kamery; priraďte k nej DAQ alebo nahrajte vlastný záznam. Vlastný záznam DAQ (`daq pool-record` alebo záznam spustený v grafickom rozhraní) alebo akýkoľvek `.daq` bez tohto sidecaru (vrátane starších záznamov) stále slúži všetkým kamerám.

#### Čítanie pixelov odrazivosti (Pix4D / Metashape / vaše vlastné skripty)

Odrazivosť je uložená ako celé číslo DN a **DN, ktoré znamená ρ = 1,0, závisí od zdrojovej kamery**:

| Zdroj | ρ = 1,0 je | Ako zistiť |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (rezerva až do ρ 2,0) | V súbore je označený XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (obmedzené na ρ 1,0) | Chýbajú značky XMP typu `Chloros:*` — táto absencia *je* signálom. |

**Prečítajte si `Chloros:PixelScale` a vydelte týmto číslom** namiesto predpokladania konštanty. Značka je definovaná v doméne uint16, takže zostáva rovnaká (`32768`) vo všetkých výstupných formátoch, ktoré menia mierku — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` a `TIFF (32-bit, Percent)` sú všetky samopopisné (najprv normalizujte uložený dátový typ späť na uint16: ×257 z 8-bitového, ×65535 z typu float).

> **Súbory zo starších verzií nemôžu obsahovať žiadne mierky.** 8-bitový TIFF so odrazivosťou sa preškaluje a označí ako `Chloros:PixelScale`, rovnako ako ostatné. Verzie pred 1.3.0 ho namiesto toho *orezali* na rozsah 0..255, takže každá hodnota nad ρ≈0,008 bola znížená na 255, a vynechali aj dvojicu `Chloros:PixelScale`, aj `MicaSense:RadiometricCalibration`. **Ak v súbore odrazivosti LATTICE chýba táto značka, nepredpokladajte mierku — spracujte ho znovu** namiesto toho, aby ste delili pixely, ktoré nikdy neboli deliteľné.

#### Čítanie indexových pixelov (NDVI a ostatné vegetačné indexy)

**Celé číslo indexového rastra predstavuje kontrastné rozťahovanie na úrovni jednotlivých snímok, nie mapu v rozsahu −1…1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` a `JPG (8-bit)` – vrátane 8-bitového súboru TIFF, do ktorého sa zapisuje 8-bitový záznam – ukladajú

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

kde `IndexMin` / `IndexMax` sú **najnižšia a najvyššia konečná hodnota indexu daného snímku**. DN 0 nie je −1 a plný rozsah nie je +1.

Program Chloros označuje oba koncové body, aby zostalo mapovanie invertovateľné:

| Značka | Význam |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — konvencia uvedená nižšie |
| `Chloros:IndexMin` | hodnota indexu uložená ako DN 0 |
| `Chloros:IndexMax` | hodnota indexu uložená ako plný rozsah |

Prečítajte ich a invertujte, pričom najskôr normalizujte uložený DN na rozsah 0…1 podľa maximálnej hodnoty uloženého dátového typu, aby jeden výraz pokryl každý formát celých čísel:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

Na čítanie nie je potrebné `-config pix4d.config` — exiftool číta tieto vlastnosti XMP bez pomoci. Konfigurácia je potrebná len na ich *zapísanie*.

> **`IndexMin == IndexMax`** znamená, že každý pixel daného snímku mal rovnakú indexovú hodnotu. Raster je vyplnený v strede rozsahu a nie je čím deliť — použite koncovú hodnotu.

> **Žiadne značky `Chloros:Index*` znamenajú, že nie je uvedené žiadne mapovanie.** Export LUT je farebný výstup, ktorého pixely sú farebné prechody viazané na vaše prahové okno LUT, a snímka bez pixelu s konečným indexom nemá žiadny rozsah, ktorý by bolo možné uviesť. V žiadnom z týchto prípadov nepredpokladajte priradenie – platí rovnaké pravidlo ako pri chýbajúcom `PixelScale`.**Keďže rozťahovanie sa vzťahuje na jednotlivé snímky, dva snímky z jedného letu nie sú porovnateľné z hľadiska DN**— každý má svoje vlastné koncové body, takže rovnaká hodnota NDVI sa v rôznych snímkach nachádza na rôznych hodnotách DN. Pre kvantitatívnu prácu v rámci letu (zónové štatistiky, pevné prahové hodnoty, časové rady) exportujte**`TIFF (32-bit, Percent)`**: indexové rastery sa tam zapisujú ako float32**samotné hodnoty indexu**, bez potreby preškálovania a bez značiek. Napriek názvu formátu nie sú indexové produkty násobené ×100 — toto škálovanie sa vzťahuje iba na odrazivosť s celočíselným dátovým typom.

#### Maska obmedzenia pri spektrálnom rozložení (na ktorých pixeloch bolo pásmo obmedzené)

Spektrálne rozloženie 3×3 môže spôsobiť, že pásmo bude záporné — ak svetlo takmer neobsahuje dané pásmo alebo ak scéna nespadá do rozsahu, ktorý popisuje matica rozloženia kamery. To nie je skutočná hodnota, takže export ju nastaví na 0 – a živý náhľad zobrazuje rovnakú hodnotu 0. Pásmo môže byť obmedzené na niekoľko pixelov alebo takmer na všetky; maska zaznamenáva obidve možnosti. `Chloros:Band{R,G,B}UnmixClampFraction` udáva, na akej časti snímky k tomu došlo; maska obmedzenia udáva, **ktoré pixely**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— v priečinku `clamp_masks/`**vedľa** priečinka produktu, nikdy v ňom (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; export z `process` do `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — pre každý produkt žiarivosti, odrazivosti a indexu, v ktorom bol aspoň jeden pixel obmedzený: zachytáva a exportuje podobne ako `process` (a TIFF súbory jednotlivých snímok z `array-build-video --save-tiffs`). Jednopásmové `uint8`, na vlastnej pixelovej mriežke produktu (vrátane orientácie, oblasti záujmu (ROI), zlučovania a zarovnania poľa). Každá maska uvádza názov kamery príslušného produktu (`Make` / `Model` / sériové číslo), čas zachytenia a označenia objektívu a uvádza zoznam pásiem, ktoré jej bity reprezentujú (`XMP-Camera:BandName`, najskôr bit 0); jej `ImageDescription` a `Software` ju označujú ako masku s obmedzením a neobsahuje žiadnu kalibráciu.
- **Bity**: bit 0 = pásmo v kanáli R produktu (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Nastavený bit znamená, že hodnota pixelu v danom pásme bola obmedzená na 0 — považujte ho, ako aj akýkoľvek index z neho odvodený, za málo spoľahlivý.
- **`Chloros:UnmixClampMask`** je vyznačené na každom produkte, na ktorom prebehlo oddelenie: cesta k maske relatívna voči vlastnej zložke produktu (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, vždy oddelená znakom `/` – pripojte ju k zložke obrázka, aby sa maska otvorila), alebo `none`, ak nebol obmedzený žiadny pixel. Chýbajúca značka znamená, že na produkte sa nikdy nepreviedol proces unmix (kamery RGB alebo monochromatické, úrovne raw/debayered).
- `lattice capture` / `array-capture` vytlačia cestu k maske pod uloženým súborom (`clamp mask: …`) a v zozname výsledkov snímania je uvedená ako `clamp_mask` daného snímku.
- `lattice index --live --save-multiband` zapíše jednu masku na každú zdrojovú kameru pre viacpásmový stack — pozri [`lattice index` — masky viacpásmového stacku](#lattice-index--multiband-stack-masks). Video nemá žiadnu masku: `array-build-video` namiesto toho hlási frakcie obmedzenia každého snímku (pozri [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Keďže žiadna zložka produktu neobsahuje masku, v akejkoľvek hĺbke je možné zložku produktu (`radiance/`, `reflectance/`, zložka `*_Images/` z exportu `process`) odovzdať programu Pix4D, Metashape alebo inému fotogrametrickému softvéru tak, ako je — vrátane podzložiek — a prijíma iba snímky. Odovzdajte mu zložku s produktom, nie zložku nad ňou (tá obsahuje aj `clamp_masks/` a všetky ostatné typy exportov). Vlastný import programu Chloros masky preskakuje. Čítanie masky: `(mask & 1) != 0` znamená „pásmo R-kanála bolo tu obmedzené“, `& 2` G, `& 4` B.

#### EXIF prenesené do exportu

`process` skopíruje **blok GPS a jeho ExifIFD** zo zdrojového záznamu do každého produktu, takže
export obsahuje `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` a
`CameraSerialNumber` spolu s georeferencovaním.

**`FocalLength` nie je pre fotogrametriu voliteľný.** Pix4D vypočíta vzdialenosť vzorky na zemi (GSD) na základe
ohniskovej vzdialenosti a nadmorskej výšky; ak táto značka chýba, program sa vráti k úplne nesprávnemu meradlu. Pri jednom
lete nad pomarančovým hájom s 49 zábermi chýbajúca značka premenila lokalitu s rozmermi 411 m × 160 m na rekonštruovanú
47,8 km × 13 km – ortofotomapu s rozlíšením 455 MP, ktorá obsahovala prevažne údaje „nodata“, čo sa následne interpretovalo ako problém s dlaždicovaním a
problém s formátom BigTIFF, skôr než niekto skontroloval GSD. Ak vaša ortofotomapa vyjde v nepravdepodobnom
mierke, najprv spustite na exportovanom produkte skript `exiftool -FocalLength`.

Kópia zámerne **nie** je `-all:all`: štrukturálne značky IFD0'štruktúrne tagy narušujú výstup LATTICE pri
kopírovaní a `ExifImageWidth` / `ExifImageHeight` sú vylúčené, pretože popisujú
*zdrojové* zachytenie – export, ktorého rozmery boli kedykoľvek zmenené, by inak mal rozmery
v rozpore s vlastným rastrovým obrazom. XMP sa zapisuje priamo, nie kopíruje, pretože ExifTool
pri kopírovaní bloku XMP vynecháva XMP značky z rovnakého volania (čo by viedlo k strate kalibračných značiek MAPIR).

### Kam sa výstupy ukladajú

Výstupy sa ukladajú **do projektovej zložky, zoskupené podľa fotoaparátu a následne podľa formátu súboru**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Zložka fotoaparátu pre LATTICE je `LATT-<sensor>-<lens>-F<filter>` (zodpovedá EXIF
`Model`) a `<model>_<filter>` pre Survey3 — dve kamery, ktoré zdieľajú senzor a filter, ale líšia sa
objektívom, majú oddelené stromy, pretože sa líšia vinetáciou, zorným poľom a skreslením. Formát
zložky je nasledovný: `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` alebo `tiff32` pre
`TIFF (32-bit, Percent)`.

> **Každý exportovaný produkt si zachováva názov zdrojového súboru.** Export radiancie súboru
> `capture_…_raw.tif` sa stále nazýva `capture_…_raw.tif` — nachádza sa však v
> `tiff32/Radiance_Images/`. **Produkt identifikuje zložka, nie názov súboru**, takže vyhľadávanie pomocou globu
> pre `*radiance*.tif` nič nenájde; namiesto toho použite zhodu podľa adresára.

### Záznamy zo svetelného senzora — kalibrované `.daq` + `.csv`

`process` spracováva aj záznamy `.daq` vo vašom vstupnom priečinku a na to **nepotrebuje**
na to nepotrebuje žiadne snímky: samostatne letiaci DAQ-U / DAQ-M / DAQ-E predstavuje kompletný
záznam a zložka obsahujúca iba súbory `.daq` je platným vstupom.

Záznam z DAQ je možné vykonať **bez** jeho kalibrácie — presne to robia verejne dostupné
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) robia štandardne: zapisujú surové hodnoty senzorov a označujú súbor tak, aby
program Chloros načítal továrenskú kalibráciu daného senzora **cez sériový port** (najskôr z lokálnej vyrovnávacej pamäte,
potom z cloudu MAPIR) a aplikuje ju. `process` zapíše výsledok späť:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` obsahuje jeden riadok na jedno meranie: časovú pečiatku UTC, integračný čas, celkový výkon,
fotopický/skotopický lux, PPFD (a jeho rozdelenie na modrú/zelenú/červenú), špičkovú vlnovú dĺžku, potom
celé spektrum na vlastnej vlnovej mriežke senzora. Súbor `.daq` sa znovu načíta bez toho, aby bol
kalibrovaný druhýkrát.

`<name>` je vlastný názov súboru záznamu. Dva záznamy v rôznych zložkách môžu zdieľať
jednu hodnotu – údaje za jednotlivé snímky (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) uložené v
tej istej milisekunde dvoma kamerami – a každý z nich stále dostane svoje vlastné výstupy: prvý si ponechá
`<name>`, ďalší pridáva zložky, ktoré ho odlišujú, **za** názvom
(`<name>__<capture>_daq_calibrated.daq`), takže dátum a čas záznamu sú stále na začiatku.

V prípade úspechu sa v správe o behu uvádza `Light-sensor products written: N (calibrated .daq + .csv)`.
Text v zátvorkách opisuje, čo sa skutočne zapísalo, takže znie
`(RAW COUNTS — this sensor has no calibration bundle)` pre senzor bez zoskupenia a
`(N calibrated, M raw counts)` pre zložku obsahujúcu oboje. Vlastné nadpisy backendu
`[DAQ-EXPORT]` a `[RUN-SUMMARY]` odvodzujú svoje znenie rovnakým spôsobom — žiadny z
týchto troch nemôže označiť surový export za kalibrovaný.

Záznam DAQ-U / DAQ-M / DAQ-E, ktorého kalibračný balík nie je možné načítať — ste
offline alebo daný senzor nemá v súbore žiadnu kalibráciu — je **vynechaný s uvedením dôvodu** v riadku
`[DAQ-EXPORT]`, nikdy sa nezapíše ako „kalibrovaný“ súbor obsahujúci surové počty.
Pripojte sa k internetu a spustite proces znovu. Dôvod je ten, ktorý čítačka skutočne
zistila pre daný súbor (nečitateľná schéma, chýbajúci balík, chyba zápisu), a v súhrne
behu sú uvedené **odlišné** dôvody – dvadsať preskočených súborov z jednej príčiny sa zobrazí ako jedna
príčina, nie ako dvadsať opakovaní tej istej príčiny. CLI na konci
spustenia uvádza názvy všetkých preskočených záznamov spolu s dôvodom:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Zlyhá aj zložka so záznamami, z ktorej sa **žiadny** neexportuje: pozri
[Zlyhá spustenie, ktoré nezapíše žiadne obrázky](#a-run-that-writes-no-images-fails).

Záznam, ktorého názov končí na `_calibrated` alebo `_raw` (`field_raw.daq`), **nie je
importovaný**: ide o názvy, ktoré `process` priraďuje svojim vlastným výstupom, a výstup prečítaný späť
ako záznam by sa započítal dvakrát. CLI to uvádza a označuje súbor —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— namiesto toho, aby hlásilo prázdnu zložku. Takýto záznam pred spracovaním premenujte.

#### Export záznamov DAQ-A ako surové počty

Rodina **DAQ-A** predchádza systému balíkov podľa sériového čísla a nemá žiadny kalibračný balík,
ktorý by bolo potrebné stiahnuť — namiesto toho sa kalibruje v teréne pomocou reflektančného terča, čo je
dôvod, prečo ho nikdy nepotrebovala. Odmietnutie týchto záznamov by im znemožnilo
vôbec získať ich údaje, preto sa exportujú pod **iným názvom**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Ide o odlišný názov súboru, nie o príznak vnútri súboru, pretože údaj musí prežiť
odoslanie e-mailom ako holý názov. Hlavička `.csv` uvádza
`raw spectral sensor counts (NOT irradiance)` a upozorňuje, že hodnoty sú porovnateľné
**v rámci** súboru — čo je presne to, na čo ich kalibrácia na základe cieľa používa — a
nie medzi senzormi. Fotometrické stĺpce závislé od výkonu (celkový výkon, fotopický a
skotopický lux, PPFD) sú zapísané ako **NULL** namiesto integrácie z počtov, a v súhrne
behu je uvedené `RAW COUNTS`, takže „exportované“ v protokole nemožno interpretovať ako ožiarenie.

Staršie záznamy **v1.01 / v1.02** (zapisuje ich DAQ-A-SD) neobsahujú epochu pre každé meranie,
iba čas zápisu súboru. Nástroj na porovnávanie snímok ↔ dopadajúceho žiarenia ich stále odmieta – porovnávanie
snímky s časom zápisu by bolo neviditeľne nesprávne –, ale exportér ich číta a
CSV súbor vypíše `clock=daq_created_on`, takže produkt uvádza, na ktorom časovom systéme beží.

### Poznámky

- `process` automaticky zistí, či je vaša zložka typu Survey3, „LATTICE“ alebo zmiešaná.
- Priebeh sa prenáša prostredníctvom Server-Sent Events; CLI zobrazuje priebeh v reálnom čase pre každé vlákno (Detekcia, Analýza, Spracovanie, Export).
- V prípade Linuxu/Jetsonu CLI skontroluje swap a môže zobraziť varovanie pred spracovaním veľkých zložiek. Debayer s podporou textúr tiež automaticky uplatňuje obmedzenie frekvencie GPU na zariadeniach Jetson s nízkou spotrebou energie (Nano, Orin Nano).
- V prípade úspechu správa o spustení uvádza, koľko obrazových výstupov bolo vytvorených (`Image products written: N`, resp. `(plus M clamp masks)`, ak dekonvolúcia obmedzila akýkoľvek pixel).
- **`process` bezpečne zdieľa bežiaci backend.** Automaticky reštartuje už bežiaci backend iba vtedy, ak je tento backend neaktívny, má načítaný predchádzajúci projekt a neobsahuje žiadne kamery, polia ani DAQ. Backend, ktorý obsahuje niektorú z týchto položiek (GUI streamujúce pole, skript SDK, záznam DAQ), sa používa tak, ako je, nikdy sa nespúšťa nanovo, takže `process` môže bežať vedľa aktívneho poľa. Ak backend hlási prebiehajúci beh, `process` počká až 15 s na jeho dokončenie (vyčistenie práve ukončeného behu trvá približne jednu sekundu), potom vráti odmietnutie s chybou `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. Globálny príznak `--restart` stále vynúti spustenie nového backendu a ukončí na ňom akúkoľvek reláciu kamery, poľa alebo DAQ. Backend, ktorý beží ako služba Linuxu systemd, sa nikdy nerestartuje: `process` ho používa a namiesto toho vypíše `--restart` (pozri [Spustenie backendu ako služby](#running-the-backend-as-a-service-headless)). Rovnako ani vlastný backend aplikácie Chloros: keď je aplikácia otvorená, používa ho `process` a načíta svoj projekt namiesto projektu aplikácie (backend spustený na jej mieste by otvorenú aplikáciu z neho uzamkol).
- **Spustenie v chode si zachováva svoj projekt.** Počas prebiehajúceho spracovávania — spusteného aplikáciou, `process`, skriptom SDK alebo `chloros-mcp` — backend odmieta vytvárať, otvárať, importovať do, pridávať alebo odstraňovať súbory, presúvať, vymazať alebo zmeniť nastavenia spracovania projektu, pričom vráti HTTP 409 `PROCESSING_IN_PROGRESS` a `A processing run is in progress on project '<name>', so ... now is refused`; ak sa to podarí, aplikácia `process` ukončí prácu a zobrazí sa `1`. Nastavenia zobrazenia (veľkosť náhľadov) sa naďalej ukladajú. Režim spracovania a nastavenia cieľovej odrazivosti projektu sú odmietnuté rovnakým spôsobom (spustenie si zachováva režim, v ktorom bolo spustené). Záznam uložený do projektu uprostred behu (z aplikácie, CLI, SDK alebo `chloros-mcp`) sa uloží ako zvyčajne, ale do zoznamu súborov projektu sa zaradí až po ukončení behu; čo potvrdzuje aj príslušná odpoveď (`project_add`), rovnako ako záznam zo svetelného senzora zastavený v rámci projektu. Predtým mohol druhý klient počas behu prepnúť otvorený projekt a prebiehajúci beh sa exportoval do druhého projektu.
- **Klávesová skratka Ctrl+C zastaví beh v backende, nielen v CLI.** CLI požiada backend o zastavenie spracovania, vypíše `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` a čaká až 120 s na `Processing stopped.`, aby sa `chloros-cli process … && next` nespustil000865 na backende, ktorý je stále zaneprázdnený spustením. Druhé stlačenie klávesovej skratky Ctrl+C ukončí čakanie. `process` následne ukončí `130`, ako to robí každý prerušený príkaz.

#### Spustenie, ktoré nezapíše žiadne snímky, zlyhá

Ak ste požiadali o produkty a beh nezapísal **žiadne** — iba `project.json` a
`calibration_data.json` — `process` to považuje za zlyhanie: vypíše
`Processing finished but wrote no image products.` a **ukončí sa s nenulovým kódom**, takže skript to môže
zistiť. Správa uvádza názov projektovej zložky a bežné príčiny:

- vstupná zložka nebola rozpoznaná ako záznam (skontrolujte rozloženie a `--input-level`), alebo
- každý požadovaný produkt bol preskočený, pretože nebol aplikovateľný pre dané kamery (napr. požiadavka na
  radianciu/odrazivosť z kamier, ktoré podporujú iba RGB).

Spustite program znovu s parametrom `--verbose` a skontrolujte protokol backendu, či sa v ňom nachádzajú riadky `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
ktoré vysvetľujú vynechania pre jednotlivé kamery, ktoré sa inak nedostanú do výstupu CLI.

Úmyselné spustenie iba s metadátami — všetky produkty vypnuté a bez `--indices` — je stále
**úspešné**, pretože prázdny výstup obrázku je v tomto prípade správnym výsledkom.

#### Spustenie, pri ktorom nebolo možné overiť exportované súbory, zlyhá

Po spustení `process` skontroluje tagy exportovaných súborov pomocou ExifTool. Ak sa ExifTool nedal spustiť,
vypíše `Overenie exportu ZLYHALO: ExifTool sa nedal spustiť, takže tagy N z M exportovaných súborov
neboli skontrolované (pozri riadky [EXIFTOOL] v protokole backendu)` and `Výstupy sú zapísané a zostávajú
na mieste; spustite proces znovu, aby ste ich overili.`, does not print `Spracovanie dokončené!` a **ukončí sa s nenulovým kódom**.
Výstupy zostávajú zachované.

#### Ktoré snímky spustenie zachytí

Snímky dokončené pred spustením `process` sú zahrnuté. Snímky uložené počas spustenia nie sú; `process`
upozorňuje, koľko ich bolo vynechaných, a opätovné spustenie ich zahrnie.

#### Spustenie, pri ktorom došlo k strate snímky, zlyhá

Ak počas behu došlo k zlyhaniu snímky, `process` vypíše `N image(s) failed and were not exported` s
najčastejšou príčinou, nevypíše `Processing complete!` a **ukončí sa s nenulovým kódom**. Ostatné výstupy sú
zapísané a zostávajú na svojom mieste, pričom protokol backendu obsahuje každú chybu.

To isté platí pre **spustenie iba so senzorom osvetlenia**: zložka so záznamami `.daq` nemá podľa definície žiadne snímky na export
a beh sa hodnotí na základe kalibrovaných súborov `.daq` / `.csv`, ktoré namiesto toho zapísal.
Ak nezaznamenal **žiadny** záznam — všetky záznamy boli preskočené — vypíše
`Processing finished but wrote no light-sensor products.`, pomenuje každý záznam podľa
dôvodu, prečo bol preskočený, a **ukončí sa s nenulovým výsledkom**.

---

## `chloros-cli login`

Overte toto zariadenie pomocou cloudového účtu Chloros+. Prihlasovacie údaje sú bezpečne uložené v pamäti cache v súbore `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Príklady

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (odstránenie časti hesla alebo duplikovanie jeho častí). Pri chybe 401 CLI automaticky opakuje pokus s `$$`, pričompridaním na koniec, potom s polovicou hesla bez duplicitných znakov; ak je opakovaný pokus úspešný, prihlási vás a vypíše správnu syntax s jednoduchými úvodzovkami, ktorú máte použiť nabudúce.

> **Použitie bez grafického rozhrania/prostredníctvom skriptu: žiadna uložená relácia znamená výzvu na prihlásenie.** Základné príkazy (`process`, `status`, `export-status`, `time-sync`, …) spustený bez uloženého prihlásenia požiada o zadanie `Email:` / `Password:` cez štandardný vstup (stdin), než pokračuje. Ak je vstup stdin uzavretý (cron, jednotka systemd), program okamžite zlyhá s chybou `login cancelled`; ak je vstupom stdin otvorený terminál alebo potrubie, čaká na vstup. Pred naplánovaním úloh bez grafického rozhrania spustite na každom počítači raz program `chloros-cli login EMAIL PASSWORD`.

---

## `chloros-cli logout`

Vymaže reláciu uloženú v cache a pri nasledujúcom volaní vynúti nové prihlásenie.

Odhlásenie počas prebiehajúceho spracovania ho nezastaví: spracovanie sa dokončí v režime, v ktorom začalo, a nasledujúce spracovanie prebehne štandardným spôsobom. Uvádza to `logout` (`The processing run in progress finishes in <mode> mode; the next run uses standard`) a rovnako aj aplikácia pri odhlásení odhlásite.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Zobrazí aktuálnu úroveň licencie (Iron/Copper/Bronze/Silver/Gold), overeného používateľa a počet viazaní na zariadenie.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Zisťuje aktuálny priebeh exportu Thread-4. Je bezpečné volať túto funkciu **počas** behu `process` z iného shellu.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Nastavuje jazyk zobrazenia v CLI (podporovaných je 38, vrátane CJK, RTL a indických písem). Na starších konzolách, ktoré nedokážu zobraziť skript, sa plynule prepne na angličtinu.

```
chloros-cli language [LANG_CODE] [--list]
```

### Príklady

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Príkazy pre priečinok projektu

Tieto príkazy spravujú umiestnenie predvoleného priečinka projektu (zdieľané s grafickým rozhraním).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` vypíše priečinok, do ktorého sa ukladajú nové projekty. Ak konfigurovaný priečinok už neexistuje (bol odstránený, premenovaný alebo sa nachádza na disku, ktorý nie je pripojený), CLI, rovnako ako grafické rozhranie, prejde na predvolený priečinok: predvolený priečinok sa vypíše na stdout, varovanie s názvom chýbajúceho priečinka sa odošle na stderr a kód ukončenia je `0`.

---

## `chloros-cli update`

Iba pre Linux/Jetson. Skontroluje `version_url` voči `/etc/chloros/update.conf` a ak existuje novšia verzia, stiahne zodpovedajúcu verziu `.deb` a ihneď ju nainštaluje pomocou `sudo dpkg -i` (bez výzvy na potvrdenie; `sudo` môže požiadať o zadanie hesla). Na zariadení Jetson sa aktualizácia ponúka len pre systém JetPack 6. V systéme Windows tento príkaz nevykoná žiadnu akciu a hlási `up to date`: aktualizujte pomocou nového inštalátora.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

V systéme Linux/Jetson vykonáva CLI tiež **automatickú kontrolu aktualizácií pri každom spustení** (neblokujúca, nikdy nezdržiava príkaz): prečíta `/etc/chloros/update.conf`, uloží výsledok do vyrovnávacej pamäte na 1 hodinu v `~/.chloros/update_cache.json` a vypíše `Update available: vX.Y.Z / Run: chloros-cli update`, ak existuje novšia verzia. V prípade akejkoľvek chyby a v systéme Windows sa táto kontrola ticho preskočí; príkaz `CHLOROS_DISABLE_UPDATE_CHECK=1` ju vypne. Vďaka 1-hodinovej cache sa verzia vydaná v priebehu tej hodiny môže stále zobrazovať ako aktuálna.

---

## `chloros-cli selftest`

Spustí 7-krokový test funkčnosti: verzia, dostupnosť portov, spustenie backendu, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), prítomnosť modelu odšumovača, pripravenosť CUDA + odšumovač. Používa bežiaci backend alebo ho na účely testu spustí a následne zastaví. Vypíše `N/7 checks passed` a neúspešné kontroly a ukončí sa `0` len vtedy, ak všetkých sedem prejde; kontrola CUDA + odšumovač zlyhá na počítači bez grafického procesora s podporou CUDA (pozri [Overenie inštalácie](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Stav a riadenie PTP grandmaster. Hostiteľský počítač Chloros spúšťa PTP grandmaster; kamery LATTICE a jednotky DAQ-E sú k nemu pripojené ako podriadené zariadenia na účely časových značiek medzi zariadeniami (približne 1 ms medzi zariadeniami). PTP zosúlaďuje **časové značky**: kamery v poli exponujú súčasne, pretože ich spúšťa hardvérový spúšťač M8 (s oneskorením menej ako 100 µs medzi modulmi), nie kvôli PTP. `chloros-cli time-sync` bez podpríkazu spustí `status`.

| Podpríkaz | Popis |
| --- | --- |
| `status` | Zobrazí stav grandmastera, priority BMCA, identitu hodín a informáciu, či jeho pakety dosahujú sieť: `health` je `OK` (`STOPPED`, ak grandmaster nebeží), alebo `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` s riadkom `health_detail`, a potom sa do stderr vypíše riadok `WARNING:` (stdout zostáva v JSON). `sync_sent` / `announce_sent` počítajú len správy, ktoré odišli; `*_send_errors` a `last_send_error` počítajú tie, ktoré neodišli. Po prerušení a obnovení spojenia s kamerou grandmaster sám obnoví svoje sokety (`rebinds`, `last_rebind`). |
| `peers` | Zoznam podriadených zariadení zistených prostredníctvom Delay_Req (kamery + senzory DAQ-E). |
| `cameras` | Stav PTP pre každú kameru (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Reštartovanie procesu grandmaster. |
| `set-priority --priority1 N --priority2 N` | Prepisovanie priorít BMCA. |

Grandmaster vysiela oznámenia na sieťové karty kamier (akýkoľvek adaptér s adresou `169.254.x.x`). Ak nie je k dispozícii žiadny, prejde na adaptér predvolenej trasy iba vtedy, ak je pripojený káblom, nikdy cez Wi-Fi alebo virtuálny adaptér (Tailscale, WSL), keďže žiadna kamera ani DAQ-E sa nenachádza za takým adaptérom. Ak nemá čo oznámiť, beží bez socketov (`health` `NO_SOCKETS`) a pri pripojení ďalšej kamery alebo DAQ sa znovu vytvorí na sieťovej karte, ktorá je v tom čase aktívna; `time-sync restart` to urobí ihneď.

Grandmaster beží vo vlastnom procese, vedľa backendu (`own_process` / `process_pid` v `time-sync status`), s vysokou prioritou vo Windowse (`process_priority`), takže vyťažený backend (spracovávanie, export, niekoľko príkazov naraz) nemôže oneskoriť časy, ktoré zaznamenáva; predtým mohlo jedno takéto oneskorenie posunúť hodiny každej kamery naraz. Tento proces končí spolu s backendom; ak sa nedá spustiť, grandmaster beží vnútri backendu ako predtým (`own_process` false, a protokol backendu to uvádza).

### Príklady

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Ovládanie kamery LATTICE. Každý podpríkaz prechádza cez backend Chloros; backend spravuje skupinu kamier, takže nasledujúce volania z CLI opätovne používajú ten istý otvorený identifikátor.

### Bežné možnosti (spoločné pre väčšinu podpríkazov)

| Príznak | Popis |
| --- | --- |
| `-d, --device N` | Index kamery (predvolené: 0). |
| `-s, --serial SN` | Konkrétne sériové číslo; prepisuje `--device`. |
| `--serials SN1,SN2,…` | Sériové čísla oddelené čiarkami pre prevádzku s viacerými kamerami. [Dočasné príkazy pre viacero kamier](#multi-camera-transient-sessions) otvoria len tieto kamery, pričom prvá spustí spúšť. |
| `--all` | Pracovať so všetkými zistenými kamerami. |
| `--exposure US` | Doba expozície v mikrosekundách. |
| `--gain DB` | Zosilnenie v dB. |
| `--pixel-format FMT` | napr. `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` na M3M. Názov určuje bitovú hĺbku v rámci vlastnej rodiny kamery: na modeli M3M názov typu Bayer určuje formát Mono jeho hĺbky (`BayerRG8` určuje `Mono8`), na modeli M3C názov typu Mono určuje názov typu Bayer (`Mono12` nastavuje `BayerRG12`). |
| `--width N` / `--height N` | Rozmery obrázku. |
| `--preset {default,high_quality,high_speed,triggered}` | Použiť prednastavenie: formát pixelov, expozíciu, zosilnenie a frekvenciu snímok (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` ich uvádza). Na M3M je formát Mono s rovnakou hĺbkou: `high_speed` zachytáva `Mono8`, `high_quality` a `triggered` `Mono12`. Príznak, ktorý je s ním uvedený (`--exposure`, `--gain`, `--pixel-format`) má prednosť pred predvolbou a nastavenie, ktoré predvolba neuvádza, zachová vlastné nastavenie kamery. Všetky voľnéprevádzky s výnimkou `triggered`, ktorý aktivuje kameru na hardvérový signál na linke 2 — ak túto linku nič neovláda, bude čakať donekonečna namiesto snímania. |
| `-o, --output DIR` | Výstupný adresár (predvolené: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Veľkosť paketu GVSP. `auto` spúšťa sondy ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Pracovný postup prvého pripojenia kamery LATTICE

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referencia podpríkazov

#### Zisťovanie a informácie

| Podpríkaz | Účel |
| --- | --- |
| `lattice info` | Zoznam pripojených kamier (výrobca, model, sériové číslo, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analýza hostiteľského systému na účely optimálnej konfigurácie kamery. `--no-discover` preskočí vyhľadávanie kamier (rýchlejšie, analýza len na základe sieťovej karty). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Kontrola/oprava nastavení sieťovej karty (v systéme Linux aj vyrovnávacie pamäte prijímania socketov jadra — pozri [Linuxové hostiteľa](#linux-hosts--socket-receive-buffers)); odhad šírky pásma/FPS. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Odporúčanie pre sieťovú+ odporúčanie poľa (vráti `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` sú predvolene nastavené na plný senzor, `2048` × `1536`. Pre `0` sa ukončuje pre `ok`, `auto_capped_fps` a `auto_shrunk`, a z `1` na `needs_force_slip` a `error`, s alebo bez `--json`. Dosiahne backend na `--backend-url` (predvolené `http://127.0.0.1:5000`), nie na `CHLOROS_BACKEND_URL`. `auto_capped_fps` zachová požadované rozlíšenie, ale obmedzí cieľovú hodnotu fps — prečítajte `recommended.recommended_target_fps` a odovzdajte ho ako cieľ pripojenia; považujte to za úspech, nie ako chybu. `auto_shrunk` sa objaví len vtedy, keď nie je možné doručiť požadovaný snímok pri žiadnej frekvencii snímok; connect nikdy sám od seba nezmenšuje snímky. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Analýza „čo keby“ bez otvorenia kamier (`--width` / `--height` predvolené `2048` × `1536`). Tak toto, ako aj `network-analysis` predpovedajú, čo bude connect streamovať: bez `--pixel-format`, platia formáty pre jednotlivé kamery, ktoré connect používa (Mono12 / BayerRG12), a s `--packed auto` (predvolené) sa uplatňuje rozhodnutie, ktoré by connect urobil na tomto počítači [v balenom 12-bitovom prenose](#packed-12-bit-transport---packed), ktoré by Connect urobil na tomto počítači — takže uvedená hodnota fps je rýchlosť, pri ktorej pole beží. Obe hodnoty zobrazujú rozhodnutie o balení. **`--n-active` je celkový počet kamier v sieti, nie len v tomto poli**— zvýšte ho, ak samostatné kamery vysielajú súčasne, alebo ak sa kapacita linky počíta na základe požiadavky, ktorá ich podhodnocuje (predvolené: `len(--models)`). Vždy vypíše súhrnné riadky `Wire budget:` (požadovaných MB/s vs. strop bez rizika kolízií) a `Max cameras:` a označí `** OVER-SUBSCRIBED**`, ak pole prekročí kapacitu linky — pozri [Model fps a burstového prenosu poľa](#array-fps--burst-model). |
| `lattice gpu` | Zobraziť stav GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Skontrolovať alebo aktualizovať firmvér kamery. Miestny výber `.fwa` je zafixovaný: súbor v `firmware/<MODEL_PREFIX>/` zodpovedajúci `MIN_FIRMWARE_VERSION` zostavy sa nahraje, ak je prítomný (najvyššia verzia slúži iba ako záložná možnosť), takže novší obraz od dodávateľa uložený na disku zostáva neaktívny, kým sa tento výber nezmení — zámerne novšie verzie sa dostávajú do zariadení prostredníctvom podpísaného manifestu AWS, čo je preferované v prípade novších verzií. |
| `lattice presets [--apply NAME]` | Zobraziť zoznam predvolieb kamery alebo ich použiť. `--apply` nastavuje, čo znamenajú názvy predvolieb (ich formát pixelov, expozícia, zosilnenie, snímková frekvencia, spúšť), a vypíše tieto nastavenia tak, ako ich kamera číta po: na M3M formát Mono pre hĺbku predvoľby; snímkovú frekvenciu, pri ktorej beží, ktorá môže byť nižšia, ako vyžaduje predvoľba (`high_speed` vyžaduje 60 fps; full-frame kamera na 1 Gbps spojení beží približne pri 35). Všetky ostatné nastavenia, ktoré program „open“ zapísal, sa vrátia tak, ako boli zistené. Príkaz `--apply default` obnoví fotoaparát na továrenské predvolené nastavenia: továrenské nastavenia s voľne bežiacou automatickou expozíciou na Line2, vypnutým obmedzením priepustnosti spojenia a vypnutou korekciou chýb, pričom sa zachová kalibrovaná vyváženosť bielej u RGB-fotoaparátu (na ostatných je nastavená jednotka). Do pamäte kamery sa nič neuloží, takže pri nasledujúcom zapnutí sa spustí z vlastnej pamäte tak ako predtým. Pri zachytení alebo pripojení `--preset` nie je `default` prednastavený. |
| `lattice status` | Stav kamery v reálnom čase. |

#### Zachytávanie

| Podpríkaz | Účel |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Jeden snímok. **Štandardne ukladá každý typ exportu**(`--processing all`); pozri [Úrovne exportu záznamu](#capture-export-levels-the-all-default). `--processing` prijíma `all`, `raw`, `debayered`, `radiance`, `reflectance` alebo `display` (`preview` je akceptované pre `display`); akýkoľvek iný názov je odmietnutý pred dotykom s kamerou (ukončenie `1`). `--levels` uloží explicitnú podmnožinu (prepíše `--processing`). Na snímanie sa otvorí kamera, ktorú Chloros nedrží, a jej expozícia sa najskôr nastaví podľa scény (najviac na niekoľko sekúnd), tak ako to robia [dočasné príkazy](#multi-camera-transient-sessions); keď sa snímanie skončí, bez ohľadu na to, ako sa skončí, táto kamera sa vráti do stavu, v akom bola nájdená (formát, expozícia, farebný pipeline, každé nastavenie, ktoré snímanie zmenilo — zoznam pod [Multi-Camera (Transient Sessions)](#multi-camera-transient-sessions)). Hodnota DAQ, ktorá načítala použitý záznam, sa uloží ako sidecar `.daq`, vrátane surového záznamu; `--no-daq` / `--daq-tolerance-ms` to upravujú ([DAQ čítajúci záznam ukladá](#the-daq-reading-a-capture-saves)). `--force-daq` je stále akceptovaný. `--jpeg-quality` = kvalita JPEG 1–100 (predvolené nastavenie 95).**Súbory uložené do otvoreného projektu sa k nemu pripoja.** Rovnako ako v aplikácii, záznam, ktorého súbory sa uložia do priečinka projektu otvoreného v programe Chloros, sa pridá do zoznamu súborov daného projektu; súbory uložené inde zostanú nedotknuté. Príkaz vypíše jeden riadok s touto informáciou (`Project: ...`), alebo, ak práve prebieha spracovanie, že sú zadržané a pripoja sa po jeho ukončení. To isté platí aj pre `multi-capture`, `array-capture` (`--smart` tiež) a `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Ukladá na disk až do stlačenia Ctrl+C. Výstupom behu `tiff`sú súborom `raw`, do ktorého sa zapisujú snímky zachytené touto kamerou: mozaika senzora, jedna rovina s vlastnými hodnotami, nekomprimovaná, s kalibračným paketom (2048 x 1536 12-bitový snímok má veľkosť 6,0 MiB plus balík, bez ohľadu na scénu), takže `process` vychádza z nich. `png` a `jpg` obsahujú 8 bitov, takže tieto série sú snímky po demosaikovaní určené na prezeranie, z ktorých `process` nevychádza. Pri viazaní na DAQ (`daq pool-bind`) alebo v zoskupenom režime dostane každý snímok svoje vlastné súbory `.daq` a `.match.json` v priečinku `daq/` danej série, keďže `lattice capture` jeden uloží; `--no-daq` túto funkciu vypne a pri zastavení sa vypíše počet snímok. Každý snímok sa zapíše raz spolu s úplnými metadátami; pri stlačení Ctrl+C sa už zachytené snímky zapíšu pred uvoľnením kamery. Program sa sám zastaví, keď sa disk zaplní, vypíše `Failed:` / `Disk full:` a ukončí sa na `1`; akýkoľvek snímok, ktorý sa nepodarilo zapísať, tiež ukončí program na `1`. Fotoaparát ponechaný v pohotovostnom režime pre hardvérový spúšťač (`TriggerMode` `On`) je počas behu nastavená na voľný chod. Po ukončení behu sa kamera vráti do pôvodného stavu — jej nastavenia spúšte a všetky nastavenia, ktoré beh zmenil (expozícia, zosilnenie, formát, …) — pokiaľ sa nevyvolá kód `--keep-settings`, ktorý uloží nastavenia záznamu do vlastnej pamäte kamery (voľný režim), alebo kód `--clear-settings`, ktorý túto pamäť obnoví na továrenské nastavenia. Beh, ktorý nezaznamená žiadny snímok, je neúspešný: ak sa počas 10 s nezaznamená žiadny snímok, sám sa zastaví, vytlačí `No frames:` s dôvodom a ukončí sa `1`. Snímky, s ktorými zapisovače nestíhali držať krok, sa počítajú ako vynechané a pri zastavení sa vypíše kód `Dropped:` s dôvodom (snímková frekvencia kamery, zapisovačov a fronty); nižšia snímková frekvencia alebo 8-bitový formát pixelov zachová všetky snímky. Príkaz sa každých 2 s hlási backendu; ak sa ukončí bez zastavenia nahrávania (`kill -9`, nedostatok pamäte), backend ho zastaví 30 s po poslednej kontrole a vráti kameru späť (`stop_reason` `client_gone` a riadok v protokole backendu). `multi-stream` a `align-stream` fungujú rovnako. Spustenie, pri ktorom sa kamera otvára dlhšie, ako je čas čakania príkazu (60 s; 180 s pre `multi-stream` / `align-stream`) — kamera sa stále spúšťa — je backendom zrušené: kamera sa vráti do pôvodného stavu a uzavrie sa a žiadne nahrávanie sa neprebieha (504 HTTP `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Živé náhľady MJPEG v prehliadači: pripojí kameru, vypíše URL streamu a otvorí ho vo vašom prehliadači. `--ae-damping` nastaví tlmenie automatickej expozície (0,4–100). |

##### Živé prenosy (MJPEG; bez RTSP)

Chloros poskytuje živé video vo formáte **MJPEG cez HTTP**(`multipart/x-mixed-replace`, hranica `frame`) z backendu.**RTSP nie je podporované.** Otvorte URL v prehliadači alebo ho prečítajte pomocou ľubovoľného MJPEG klienta (`cv2.VideoCapture(url)` z OpenCV, ffmpeg, VLC). Kamera musí byť najskôr pripojená v backende (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` alebo cez grafické rozhranie).

| URL (`http://127.0.0.1:5000` + …) | Prúd |
| --- | --- |
| `/api/camera/stream/<serial>` | Náhľad na displeji, až do cca 30 fps. Pre prvok poľa platí, `?max_h=` (predvolené `720`) a `?jpeg_q=` (predvolené `80`) nastavujú výšku a kvalitu JPEG. Toto je URL, ktorú otvára `lattice viewer`. |
| `/api/camera/<serial>/stream-annotated` | Ten istý náhľad s nakreslenými prekrývacími vrstvami grafického rozhrania: `zebra=1`, `crosshair=1`, `grid=3x3` (alebo `9x9`), `histogram=1`, `peaking=1`, `spot=1`; bez dotazu sa použijú uložené nastavenia prekrývajúcich sa vrstiev projektu pre danú kameru. |
| `/api/camera/<serial>/stream-reflectance` | 8-bitová odrazivosť na pásmo: ρ = pixel ÷ časť`X-Chloros-Pixel-Scale` (255, pokiaľ nie je nastavená hodnota `refl_max`). `fps` (predvolené nastavenie `10`), `jpeg_q` (predvolené `95`), `max_h` (predvolené `0` = natívne), `band=<name>`, `align=1`. Formát JPEG, teda so stratou kvality. Odmietnuté s kódom HTTP 409, ak kamera nemá odrazivosť (RGB kamera, chýba kalibračný balík, chýba hodnota DAQ). |

- **Zosúladenie snímok medzi kamerami.**Každá časť, ktorú pošle**člen poľa**, vo všetkých troch prúdoch obsahuje hlavičky `X-Chloros-Group-Id` (počítač synchronizovanej skupiny poľa), `X-Chloros-Frame-Id` a `X-Chloros-Timestamp-Ns`. Časti z rôznych kamier s rovnakým `X-Chloros-Group-Id` pochádzajú z rovnakého spúšťača. Samostatná kamera nemá žiadnu skupinu a neodosiela žiadne ID skupiny. Prehliadače ignorujú hlavičky jednotlivých častí; klient, ktorý sám analyzuje viacčastý prúd, si ich prečíta.
- **Pozastavené, kým je otvorené zobrazenie poľa.** `/api/camera/stream/<serial>` člena poľa odosiela statický rámec „paused“, kým je otvorený kombinovaný živý prúd poľa (napríklad zobrazenie poľa v grafickom rozhraní), a pokračuje v odosielaní po jeho zatvorení.

#### Nastavenie senzorov

| Podpríkaz | Účel |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Čítanie/zápis ľubovoľného uzla GenICam. `--set ExposureTime=…` (`Gain=…`) bez prepínačov `ExposureAuto` (`GainAuto`), ktoré najprv vypnú automatický režim, ako to robí `lattice exposure --set`; ak kamera odmietne hodnotu, vypíše `Failed to set <node>: …` na stderr a ukončí sa `1`. `--get` a predvolený zoskupený náhľad vypíše hodnotu `note:` pod `ExposureTime` / `Gain` počas behu tohto automatického režimu — pozri [Čítanie expozície a zosilnenia v automatickom režime](#reading-exposure-and-zisk-v-automatickom-režime). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Expozícia a AE. Hodnota expozície `--set` alebo cieľová hodnota `--brightness` sa zachovajú ako vaše (automatické riadenie expozície Chloros to necháva tak); zosilnenie zostáva tak, ako bolo, takže v automatickom režime sa neustále prispôsobuje cieľu, kým nenastavíte `lattice gain`. `--auto` vráti kontrolu nad expozíciou, `--brightness-auto` vráti cieľ späť automatickému nastaveniu Chlorosu. Bez žiadnych príznakov prečíta uzly expozície, s rovnakým nastavením `note:` ako `configure --get`. Ak fotoaparát odmietne hodnotu `--set`, vypíše `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` na stderr — automatický režim, v ktorom bolo vypnuté zapisovanie, sa obnoví namiesto toho, aby zostal vypnutý — nevypíše riadok `Exposure: … us (auto OFF)` a ukončí sa `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Zosilnenie a automatické zosilnenie. Pri hodnote `--set` kamera odmietne vypísať `Failed to set Gain: …` na stderr, nevypíše riadok `Gain: … dB (auto OFF)` a ukončí sa `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | Oblasť záujmu (ROI) snímača a zlučovanie pixelov. Zlučovanie pixelov je vždy **priemerné** — pozri nižšie. |
| `lattice format [--set FMT] [--list]` | Formát pixelov. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Hardvérová/softvérová spúšť. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (bez príznakov = jednorazové vyváženie bielej) | Operácie vyváženia bielej. Iba kamery RGB/Bayer; na monochromatických M3M sa táto operácia nevykonáva (preskočí sa). Funkcia one-shot zachová vyváženie bielej, ktoré vypočíta (`BalanceWhiteAuto` a zosilnenia); všetky ostatné nastavenia, ktoré sa zapisujú do otvoreného súboru, sa vrátia do pôvodného stavu, a to v plnom rozsahu, ak sa funkcia one-shot nedá spustiť. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | Farebný spracovateľský reťazec pre RGB. `natural` (predvolené) je lacná úprava v reálnom čase; `enhanced` pridáva odstránenie farebných okrajov + živosť + lokálny kontrast CLAHE pre plný vzhľad s paritou v strede pri ~2× vyšších nákladoch na spracovanie na snímku, takže nižšia **živá** snímková frekvencia — uložené zábery vždy dostanú plné spracovanie v oboch prípadoch. Iba kamery RGB/Bayer; preskočené na mono M3M. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Sýtosť/kontrast v živom náhľade (kamery s RGB filtrom); `100` je neutrálne, `--reset` nastaví obe hodnoty na 100. Uložené snímky nie sú ovplyvnené. Preskočené na mono M3M. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (žiadne príznaky = čítanie) | Zobrazenie **svetlých oblastí** (kamery s RGB filtrom): stlmí najjasnejšie, najbielejšie pixely — napr. jasný biely povrch, ktorý sa zobrazuje ako plochá biela škvrna. `0` = vypnuté, `-100` = najsilnejšie. Živý náhľad a snímky na úrovni zobrazenia sa tým riadia; jas/odrazivosť sa tým nikdy neriadia a nedokáže obnoviť údaje zo senzora, ktoré boli orezané (na to znížte expozíciu). Preskočené na monochromatickom M3M. |
| `lattice filter [--list]` | Zobrazí zoznam modelov filtrov, ktoré pozná Chloros (`RGN-IMX265`, `OCN`, `NGB`, …). Filter fotoaparátu sa načíta z reťazca modelu a nedá sa nastaviť: `--set NAME` iba vyhľadá model a potom ukončí `1` s `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Zisťuje stav napájania/teplotných uzlov; prepína do režimu nízkej spotreby v pohotovostnom stave. `--sleep` ponechá kameru v režime spánku (limit priepustnosti jej spojenia je zapnutý (`DeviceLinkThroughputLimitMode` `On`) na minimálnej hodnote, režim úspory energie je zapnutý, ak ho kamera má) pri všetkých ostatných nastaveniach sa jej otvorený zápis vráti do pôvodného stavu; spánok, ktorý zlyhá, sa obnoví v plnom rozsahu. Prebudenie vráti spojenie do stavu, v akom ho našiel spánok (limit, režim a oneskorenie paketov `GevSCPD`, ktoré kamera udržiava na hodnote 0, pokiaľ je limit zapnutý; záznam sa uchováva na tomto počítači, takže ho nájde aj prebudenie po reštarte backendu), alebo v prípade kamery, ktorú tento počítač nepreviedol do režimu spánku, ale ktorá vyzerá, že spí (jej limit je nastavený na minimálnu hodnotu), s vypnutým limitom a hodnotou `GevSCPD` ponechanou na 0 pri 80 (továrenské nastavenia); limit nastavený nad minimálnou hodnotou zostáva nezmenený. Sonda a prebudenie vrátia späť to, čo ich otvorenie zapísalo. |

> **Binning je vždy priemerný.** `lattice resolution --binning N` nastavuje priemerný binning a
> neponúka prepínanie režimov. Binning typu Sum vynásobí každú hodnotu DN o N², čo stojí približne dva stupne
> rezervy pre jasné oblasti na každý krok binningu; Chloros tento faktor opäť vydelí pred
> radiometrickým reťazcom, ale pixely, ktoré Sum posunul do stropu snímača, sú orezané a nemožno ich
> obnoviť. Neprináša to nič, čo by neprinášalo nastavenie „Average“ – index je invariantný voči mierke a skaláre
> žiarivosti sú kalibrované v mierke „Average“. Ak je kamera dodaná už nastavená na „Sum“ (
> nastavená iným nástrojom alebo obnovením továrenských nastavení), Chloros to zistí na základe spätného čítania uzla a
> kompenzuje to; spustením `lattice resolution --binning N` sa to vráti späť na „Average“. V verzii 1.2.0
> `--binning-mode` sa stále akceptuje, takže staršie skripty bežia, ale vypíše sa len varovanie:
> zlučovanie zostáva na úrovni „Average“.

> **Zmena binningu, formátu pixelov a spúšťača na kamere nemá žiadny účinok.** Na kamere, kde
> backend zostáva otvorený a streamuje (`cam-connect`, grafické rozhranie), sa `lattice resolution --binning`
> nemá žiadny účinok (vypíše, že binning je stále aktívny, `Binning: 1x1`), a
> `lattice format --set` a `lattice trigger --mode` sú kamerou odmietnuté
> (`SC_ERR_ERROR -1001`). Najprv ho uvoľnite (`lattice cam-disconnect -s <SN>`) alebo ich nastavte
> pri pripojení (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Čítanie expozície a zosilnenia v automatickom režime

Keď je v kamere spustená funkcia `ExposureAuto` (`GainAuto`), jej uzol `ExposureTime` (`Gain`) uchováva **poslednú zapísanú hodnotu**, nie aktuálny stav kamery. V prípade kamery, ktorú backend drží otvorenú (`cam-connect`, `array-connect`, grafické rozhranie), `lattice configure --get`, predvolený pohľad `lattice configure` a `lattice exposure` (bez príznakov) preto vypíšu hodnotu uzla a pod ňou poznámku s najnovším snímkom kamery:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Hodnotu snímky prečítajte z poznámky; hodnota uzla je len posledný zápis. Cez HTTP tá istá odpoveď `POST /api/camera/<serial>/read-nodes` ponecháva `nodes` nezmenené a pridáva `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, najnovšieho snímku) a `notes` (predchádzajúca veta). Obe sú prítomné len vtedy, ak automatický režim nie je `Off` a je známy snímok; kamera otvorená len na čítanie nemá snímku a nedostane ani jeden z nich. `lattice gain` (bez príznakov) vytlačí len uzol — pre poznámku použite `lattice configure --get Gain`.

##### Expozícia na člene poľa

Počas streamovania poľa beží každý člen s nastavením **`AcquisitionFrameRateEnable`**, a to pri frekvencii snímok o niečo vyššej ako je spúšťacia frekvencia poľa: jej perióda je spúšťacia perióda znížená o 15 % (minimálne 3 ms), alebo vlastná maximálna frekvencia kamery, ak je spúšťacia frekvencia už blízko tejto hodnoty. `configure --get` na prvku preto pri tomto nastavení číta hodnoty `AcquisitionFrameRateEnable` ako `True` a `AcquisitionFrameRate`; povolenie sa vypne, keď sa pole zastaví.**Nezapisujte tieto dva uzly na člen poľa** — pole ich udržiava v súlade so svojím spúšťačom a prepisuje ich.

Toto nastavenie je zároveň maximálnou expozíciou člena: približne 85 ms pri spúšťači 10 fps (perióda 100 ms), pri **akomkoľvek ROI alebo binningu**. Pred verziou 1.3.0 malá oblasť záujmu (ROI) obmedzovala expozíciu člena na periódou voľného chodu senzora (4,9 ms pri rozlíšení 768 × 304), bez ohľadu na to, ako pomaly sa pole spúšťalo.**Ručne zadaná hodnota `ExposureTime`, ktorá presahuje tento limit, spomalí spúšť, aby sa mu prispôsobila**, ak sa zapíše pomocou `lattice exposure --set` alebo `configure --set` (alebo cez grafické rozhranie). Pole najprv otvorí limit daného člena pre danú hodnotu a nechá pomalšiu frekvenciu bežať niekoľko spúšťacích periód, takže zápis môže trvať o niekoľko sekúnd dlhšie; spúšť sa potom aktivuje najviac tak rýchlo, ako to dovoľuje expozícia. V poli s automatickou frekvenciou sa predchádzajúca frekvencia obnoví, akonáhle sa každý člen, ktorého manuálna expozícia spúšť spomalila, vráti na automatickú expozíciu. Hodnota, ktorú samotná kamera nemôže prijať, je naďalej odmietnutá, pričom v chybe je uvedený rozsah, ktorý kamera akceptuje, a automatická expozícia člena sa obnoví, ak bola vypnutá iba pre tento zápis. Expozícia podľa receptu `apply` spúšť nezpomalí: je udržiavaná na hornej hranici. Chloros nikdy nezkracuje manuálnu expozíciu, aby vyhovovala rýchlejšiemu spúšťaniu; pevne nastavená frekvencia (`array-connect --target-fps F`), ktorá je rýchlejšia, než to expozícia dovoľuje, ponecháva danému členu strop dostatočne dlhý na to, a tento člen prijíma spúšťanie nie rýchlejšie, než to dovoľuje jeho expozícia.

#### Kalibrácia a senzory

| Podpríkaz | Účel |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibrácia pomocou cieľa odrazivosti. Vyhľadávajú sa až `--attempts` snímok (predvolené `30`); neúplný rámec (strata paketu GVSP) stojí jeden pokus, nie celý beh. Ak sa nedarí s hodnotou `no target detected in N attempts`, alebo pri hodnote `no complete frame in N attempts: <last error>`, ak neprišiel žiadny kompletný rámec. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | Spektrum a amplitúdy pásiem zariadenia DAQ-M, načítané cez Bluetooth (BLE) prostredníctvom backendu. `--mac` je vlastná adresa BLE MAC senzora a je potrebná pri ktorejkoľvek z troch akcií: neexistuje žiadna predvolená hodnota, keďže každá jednotka má svoju vlastnú (`daq pool-discover --only ble` ich vypisuje); bez neho sa príkaz ukončí na hodnote `1`. Ak nie je zadaná žiadna akcia, vypíše sa nápoveda. `--camera SERIAL` integruje zobrazené hodnoty pásiem pomocou vlastnej nameranej spektrálnej odozvy danej kamery namiesto krivky filtračného modelu — RGB kamery, ak je kalibračný balík kamery načítaný (pripojený) alebo uložený v cache tohto počítača. V nadpise je uvedené, čia krivka bola použitá: `Band magnitudes (camera <SERIAL>):`, alebo `Band magnitudes (model <filter>):`, ak kamera žiadnu nemá (nie je to RGB kamera, alebo nie je nainštalovaný ani uložený v cache žiadny balík) a namiesto toho bola použitá krivka typu „fleet“. Cez HTTP prijíma `POST /api/dls/cli-read` hodnotu `camera_serial` a vráti hodnotu `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Uplatniť korekciu vinetácie na existujúce snímky, pričom každý má vlastné ploché pole objektívu (z jeho EXIF Model). `--lens-model` sa uplatňuje na snímky, ktorých Model neuvádza žiadny fotoaparát; snímka, ktorej Model je s tým v rozpore, sa preskočí. |

#### Viac fotoaparátov (dočasné relácie)

Dočasný príkaz otvorí kamery na jedno spustenie a po jeho ukončení ich zatvorí: `multi-capture` bez pripojeného trvalého poľa, `multi-stream` a `multi-test` v tomto prípade, a nižšie uvedené `align-calibrate`, `align-apply`, `align-stream` a `index --live`. Pre všetky z nich platia tieto pravidlá:

- **`--serials SN1,SN2,…` vyberá kamery.** Otvoria sa len tie, a prvá z nich spustí spúšť (v prípade `align-calibrate` a `--reference`, ak sú zadané). Sériové číslo, ktoré nie je v sieti, spôsobí zlyhanie behu ešte pred otvorením akejkoľvek kamery. Bez `--serials` sa otvoria všetky kamery v sieti. Kamera, ktorú už Chloros drží (GUI, `cam-connect`, `array-connect`), odmietne spustenie bez ohľadu na to, či ju `--serials` menuje alebo nie: zdieľa sieť, pre ktorú je spustenie určené. (`index --live` bez `--serials` namiesto toho otvorí kamery povolené v jej profile: pozri [Index / Vegetation Maths](#index--vegetation-maths).)
- **Kamery zdieľajú sieť rovnakým spôsobom, ako ju zdieľa `array-connect`.** Každá kamera je prispôsobená svojmu podielu na prenosovej kapacite hostiteľského prepojenia (ide o medzipaketové oneskorenie, nie o limit priepustnosti kamery, ktorého minimálnu hodnotu 31,25 MB/s už štyri kamery na 1 GbE prekračujú), takže štyri kamery bežia na 1 Gbps spojení. Keď kamery nemôžu všetky naraz vysielať najpomalším tempom, akého dosahujú, spúšťajú sa jedna po druhej, každá s vlastným pripojením, a príkaz vypíše `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Ak sa nezmestí ani jedna kamera naraz, spustenie je odmietnuté skôr, ako začne akákoľvek kamera vysielať: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP 409, ukončenie `1`.
- **Všetky kamery musia odosielať dáta, inak spustenie zlyhá.** Kamera, ktorá neodosiela žiadny snímok, predstavuje chybu: príkaz vypíše `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, pričom `<why>` je chyba načítania danej kamery, potom sa uvedie príčina a program sa ukončí s kódom `1`. Rámec, ktorý dorazil so stratou paketov GVSP (`IncompleteBufferError: GVSP packet loss …`), znamená, že sieť nezvláda kombinovanú šírku pásma kamier (`Every missing frame arrived with GVSP packet loss: …`): skontrolujte rýchlosť spojenia hostiteľa a uplinku prepínača (`lattice network`), použite jumbo rámce, alebo otvorte menej kamier pomocou `--serials`. Kamera, ktorá neposlala žiadne dáta, zvyčajne nedostáva spúšťací signál od hlavnej kamery: skontrolujte jej synchronizačný kábel M8 a potom spustite `lattice multi-detect`. Čo každý príkaz robí so skupinou rámcov, v ktorej chýba kamera, je uvedené v príslušnom riadku.
- **Každá kamera sa vráti do stavu, v akom bola nájdená.** Po ukončení behu sa každá kamera vráti do stavu, v akom bola pri jej otvorení: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, a `LineMode` / `LineSource` na `Line2` a `Line3` (a odstránenie šumu na spúšťacom vstupe poskytuje pole linku podriadeného zariadenia, `LineFilterSelector` / `LineFilterWidth`); jeho `PtpEnable` (a s ním nastavené `PtpSlaveOnly` / `PtpDomainNumber`); `ExposureAuto` / `GainAuto` (a `ExposureTime` / `Gain`, kde bola nastavená manuálna hodnota); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (a každý `BalanceRatio`, kde bolo vyváženie bielej nastavené manuálne); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, binning, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (a `Gamma`, kde bola zapnutá funkcia gama), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; veľkosť paketu (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); frekvencia, `DeviceLinkThroughputLimitMode` (a `DeviceLinkThroughputLimit`, kde bol režim `On`), medzimedzipaketové oneskorenie `GevSCPD` a časový posun začiatku `GevSCFTD`, ktoré každá kamera poskytuje; blokové údaje, ktoré každý prúd aktivuje (`ChunkModeActive`, každý `ChunkEnable` pod svojím `ChunkSelector`); `AcquisitionFrameRateEnable` (a `AcquisitionFrameRate`, kde bolo povolenie zapnuté — ak je vypnuté, rýchlosť je len na čítanie a číta maximálne toľko, koľko umožňuje tempo, takže to nie je nastavenie). Zapíše sa len to, čo sa líši: najprv sa kamera zastaví, potom sa zaznamená frekvencia a rýchlosť snímok pred expozíciou, ktorá vyžaduje svoje snímkové obdobie (kamera obmedzuje hodnotu `ExposureTime` na jedno snímkové obdobie). Pri `--keep-settings` sa späť odpojí zapojenie spúšte a synchronizačnej linky, kamera beží vo voľnom režime a zvyšok sa uloží do vlastnej pamäte kamery podľa nastavení behu; `--clear-settings` tam nastaví továrenské nastavenia.
- **Expozícia sa nastavuje na základe scény pred zachytením prvého snímku.** Séria snímaní zachytáva synchronizované skupiny snímok a vyraďuje ich, pričom na základe týchto snímok nastavuje expozíciu a zosilnenie každej kamery. Cieľom je dosiahnuť priemernú jasnosť na cieli kamery s maximálne 1 % preexponovaných častí snímky; scéna s jasnými vrcholmi nedosahuje tento cieľ. To zvyčajne trvá 2–6 skupín snímok a najviac 5 s.
  - Každá kamera zameriava cieľ, od ktorého ju spustí pripojené pole, podľa svojho filtra: TargetBrightness **40**na multispektrálnej kamere (RGN, OCN, NGB) a**60** na RGB kamere. To je polovica cieľovej hodnoty expozície 80 (tri štvrtiny pri RGB), takže je menej pravdepodobné, že dôjde k orezaniu jasného odrazového panelu. `--brightness N` (0–255) namiesto toho nastavuje cieľ pre každú kameru (`multi-test` nemá `--brightness`: jeho automatická expozícia je vypnutá). Protokol backendu uvádza cieľovú hodnotu každej kamery: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - Expozícia má prednosť, a to až do limitu automatického nastavenia expozície danej kamery. Zosilnenie pokrýva len to, čo zostane.
  - Zistená expozícia a zosilnenie sa zachovávajú počas celého behu, takže každý snímok kamery `multi-stream` sa sníma s rovnakou expozíciou. Snímkovanie, ktoré musí sledovať meniace sa osvetlenie, patrí do prepojeného poľa (`array-connect`), ktorého automatická expozícia sa neustále prispôsobuje.
  - Snímky `--exposure` a `--gain` zostávajú presne také, ako boli zadané. Ak sú zadané obe, žiadny snímok sa nezahodí.
  - Po ukončení behu sa automatická expozícia každej kamery opäť zapne.
  - Protokol backendu zaznamenáva výsledok: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …` alebo `exposure NOT solved after …` s uvedením dôvodu a `(not settled)` pre každú kameru, ktorá nebola zapnutá. V takom prípade sa snímky aj tak zaznamenajú.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` mení 5-sekundový limit; `0` túto funkciu vypne (pozri [Premenné prostredia](#environment-variables)).

**Kým Chloros drží akúkoľvek kameru** (GUI, `cam-connect` alebo pripojené pole), `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` a `index --live` odmietnu bežať, pretože otvárajú každú kameru v sieti a druhé otvorenie zadržanej kamery môže spôsobiť pád backendu. (`index --live` odmietne spustenie, ak je kamera, ktorú by mal otvoriť, zadržaná: ide o kameru `--serials` alebo kamery v jej profile, a akúkoľvek kameru pre profil, ktorý žiadnu nešpecifikuje.) Backend odpovie HTTP 409 `CAMERA_HELD`, pričom uvedie každú zablokovanú kameru a príkazy na jej uvoľnenie (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`) a príkaz sa ukončí (`1`). Ak je pripojené pole, `array-capture --continuous` ho zaznamená namiesto `multi-stream` a `array-capture` (štandardne zarovnaný) uloží zarovnanú sadu na miesto `align-apply`; `multi-capture` a `align-calibrate` používajú pripojené pole, ktoré obsahuje ich kamery, namiesto ich otvárania.

| Podpríkaz | Účel |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Zobrazí zoznam všetkých kamier so synchronizačnými úlohami; s `--serials` iba tie, s `MASTER` iba prvú. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Jeden synchronizovaný snímok z každej kamery. Pri pripojení trvalého poľa **štandardne uloží všetky typy exportu**a hodnotu DAQ, ktorá sa použila ([Hodnota DAQ, ktorú zachytáva snímok, sa uloží](#the-daq-reading-a-capture-saves)); dočasná záložná možnosť bez poľa je**len bez debayeringu** (pre zvyšok najprv spustite `array-connect`). `--brightness` sa vzťahuje iba na náhradný režim: pripojené pole si zachováva vlastnú automatickú expozíciu. Náhradný režim neukladá `.daq` (`--no-daq` / `--daq-tolerance-ms` tam nemajú žiadny účinok) a `multi-capture` neberie do úvahy `--force-daq`. V náhradnom režime kamera, ktorá nedodáva žiadny snímok, zlyhá pri zachytávaní (`Error: multi-capture failed: no frame from …`, ukončenie `1`) a žiadny súbor z tejto skupiny snímok sa neuloží. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Synchronizované snímky zo streamu (dočasné), `--fps` predvolené `3`, až kým sa nestlačí Ctrl+C alebo `--count` skupiny. Skupina snímok, pre ktorú kamera neposlala žiadne údaje, sa nezapíše, a tri po sebe ukončia prúd. Ak bola nejaká skupina preskočená, beh sa ukončí vytlačením `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)`, plus `Error: the stream stopped: …` s uvedením názvu kamery, keď ukončila prúd, a ukončí sa `1`X. Zastaví sa, keď sa disk zaplní, vypíše `Failed:` / `Disk full:` a ukončí sa `1`; akýkoľvek súbor, ktorý sa nepodarilo zapísať, tiež ukončí program `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | Test synchronizácie GPIO (prechodný), `--count` predvolené spúšťače `10`. Vypíše, ako je sieť zdieľaná (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)` a riadok `Serial emit:`, keď sú kamery spúšťané jedna po druhej, pričom časové značky potom obsahujú časový posun), jeden riadok `Sync group: master SN:<serial> -> SN:<serial>, …` na každú synchronizačnú skupinu (`(no slaves: not synced)` pre kameru bez synchronizačného kábla, ktorá sa spúšťa samostatne), potom jeden riadok na jedno spustenie: `[  0] SN:<serial>=<timestamp> \| …`, pričom `SN:<serial>=(no frame)` je pre kameru, ktorá pri danom spustení neposlala žiadne údaje. Potom riadok `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` a riadok `SN:<serial>: mean <±x> us, jitter <y> us` na každú kameru, pričom každá kamera je porovnaná s hlavnou kamerou v rámci jedného taktu: spúšť, pri ktorej sa v časových pečiatkach miešajú zariadenia hodín kamier a PTP (PTP vypnuté alebo stále sa synchronizuje) sa vynechá, pričom ich počíta riadok `Note:`. Akýkoľvek vynechaný snímok spôsobí zlyhanie testu (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, ukončenie `1`); tri neúplné spúšťacie signály za sebou test predčasne ukončia. Test zlyhá (ukončenie `1`) aj vtedy, ak menej ako dva spúšťače majú všetky kamery synchronizované na jeden takt (`… the clocks cannot be compared`), alebo ak sa hodiny nikdy nezhodujú: Hodiny PTP sa od hlavného hodín líšia o viac ako 1 ms (plus akékoľvek posunutie na sériovej úrovni) (`… PTP is not locked`), alebo hodiny zariadení, ktorých posun sa medzi spúšťacími impulzmi zmení o viac ako túto hodnotu. Spustí PTP grandmaster, ak ešte nebeží, zapne PTP na každej kamere (tak ako to robí `array-connect`, bez ohľadu na konfiguráciu siete) a pred prvým spúšťačom počká, kým sa PTP hodiny každej kamery zosynchronizujú s grandmasterom a ustália sa: `PtpStatus` Slave na jednom hlavnom časovači, s odchýlkou do 100 us od neho a stabilný po dobu 3 s, pričom každá kamera je takto synchronizovaná bez prerušenia po dobu 120 s — približne 2 minúty od zapnutia. Hodiny, ktoré sa práve synchronizovali, majú odchýlku až ~200 us približne 2 minúty po tom, čo sa na displeji zobrazí „Slave“, a vlastný indikátor `PtpOffsetFromMaster` kamery to nezobrazuje; po 120 s sa kamery dostanú do odchýlky ~10 us od ustálených hodín. Kamera stále synchronizovaná od predchádzajúceho testu, pri ktorom zostal PTP zapnutý, ako to tento backend zaznamenal, počíta od tohto synchronizovania, takže test vykonaný hneď po predchádzajúcom už nečaká; kamera, ktorú test zapne, alebo kamera s aktívnym PTP, u ktorej tento backend nezaznamenal synchronizáciu (zapnutá inde alebo pred reštartom backendu), čaká svojich 120 s, a v poli oboch kamier, ktoré sa práve synchronizovali, stále čakajú na svoj čas. Kamera, ktorá opustí režim Slave, prekročí 100 us alebo počas čakania sleduje iného grandmastera pri 3 odčítaniach za sebou (1,5 s), začne odpočítavanie znova; jedno alebo dve takéto odčítania sa zaznamenajú (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) a odpočítavanie pokračuje; identifikátor grandmastera, ktorý nie je možné prečítať, sa považuje za neznámy, nie za zmenu. Ak sa odpočet všetkých kamier reštartuje naraz (v priebehu jednej sekundy), ide o pohyb času grandmastera, nie kamier: protokol backendu uvádza `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` ho uvádza v `grandmaster_steps` a čakanie, ktorého čas potom vyprší, začína svoju chybu s `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Limit sa pre to nepredĺži. `--ptp-settle SECONDS` nastavuje čas ustálenia pre beh (0–600; `0` pre čakanie bez ustálenia). Čakanie je obmedzené na čas ustálenia plus 90 s (štandardne 210 s, minimálne 90 s) a príkaz čaká na backend 240 s plus čas ustálenia (štandardne 360 s). Vypíše `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)` a protokol backendu obsahuje každých 5 s riadok so stavom a posunom každej kamery, ako dlho je uzamknutá a koľko času jej ešte zostáva: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Každá kamera sa zapne s vypnutým PTP; od tohto momentu sa hodiny synchronizujú približne za 10 s. Následne sa každá kamera vráti do stavu, v akom ju test našiel – jej PTP, automatická expozícia a zosilnenie zostanú počas behu testu vypnuté a ostatné nastavenia (pokiaľ nie je `--keep-settings`) — takže kamera, ktorá zostala so zapnutým PTP (`lattice configure -s <serial> --set PtpEnable=True` alebo beh s `--keep-settings`), zostáva synchronizovaná a akonáhle tento backend zistí, že bola synchronizovaná počas doby ustálenia, neskorší test na ňu nečaká. Ak sa hodiny počas čakania nezosúladia, test zlyhá ešte pred spustením (`PTP: NOT locked …`, ukončenie `1`) a uvádza, čo treba urobiť: kamera, ktorá sa nikdy nezamkla (`PtpStatus Listening`), znamená, že grandmaster sa k nej nedostáva (`time-sync status`); tá, ktorá neustále vypadávala, je označená podľa toho, ako dlho bola zamknutá (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); hodiny sa stále zosúlaďujú: skontrolujte `time-sync status`, a aby boli synchronizované pred testom, zapnite PTP a nechajte ho zapnuté (`lattice configure -s <serial> --set PtpEnable=True` pre každú kameru, alebo spustite test s `--keep-settings`; test obnoví zistené nastavenia, takže zostane zapnuté), potom ho spustite. |
| `lattice multi-detect [--line LINE] [--json]` | Nájdite kamery na synchronizačnej linke M8 a predvolenú hlavnú kameru: kameru RGB (model končiaci na `-FRGB`), ak je pripojená káblom, inak prvú pripojenú kameru podľa sériového čísla. Káble M8 sa stretávajú v jednej križovatke, takže **akákoľvek pripojená kamera môže byť hlavnou kamerou** — vytlačený údaj `MASTER` je predvolený, nie skutočný stav zapojenia. Každá kamera postupne ovláda linku a ostatné kamery sa kontrolujú na prítomnosť jej impulzu; kamera, ktorá už bola nájdená v zapojenej skupine, sa znovu nekontroluje, takže úplne zapojená skupina prejde jedným cyklom. Ak chýba `--line`, skontroluje sa `Line2`, potom `Line3`. Vytlačí `MASTER: SN:…`, po jednom `SLAVE: SN:…` pre každú ďalšiu zapojenú kameru a `(probed on <line>)`; `--json` pridá celkový výsledok (`master`, `slaves`, `no_cable` — kamery, ku ktorým impulz nedošiel — `line` a pre každú kameru `results`). Po dokončení sa každej kamere obnovia pôvodné nastavenia spúšťača a synchronizačnej linky (tie isté uzly ako vyššie). |

#### Zarovnanie

| Podpríkaz | Účel |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — plus ovládače detektora/porovnávača `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, ovládacie prvky RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombinácia viacerých snímok `[--averaging mean\|median\|inlier_weighted]`, geometrické obmedzenia `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, priestorové obmedzenie `[--roi X0,Y0,X1,Y1] [--mask PATH]` aprepísania na úrovni podriadených jednotiek `[--per-cam-override SN:KEY=VALUE]` (opakovateľné) | Vypočítajte profil zarovnania z živých kamier. Predvolené nastavenia: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (v priečinku, z ktorého sa spúšťa), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate` má vlastné predvolené nastavenia; pozri [jeho tabuľku](#project-align-calibrate-options).) `--prefilter` má predvolené nastavenie `gradient` (mapa hrán; zodpovedá vyrovnávaču GUI/pole — hrany pretrvávajú naprieč spektrálnymi pásmami). `--matcher flann` sa oplatí pri počte znakov nad ~5000; `--averaging median` je odolný voči jednému zle zachytenému snímku, `inlier_weighted` váži podľa počtu zhôd; `--lock-scale` premieta na najbližšiu rotáciu (bez mierky), `--lock-axis` vynuluje jednu zložku posunu; `--mask` sa uplatňuje na každú kameru (pre nastavenia pre jednotlivé kamery použite `--per-cam-override`, napr. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` odmietne uložiť kalibráciu, ktorej RMS reprojekcie prekračuje limit. `--method manual` čerpá svoje body z `--points PATH`: JSON-objekt `{"<serial>": [[x, y], ...], ...}`, ktorý udáva súradnice pixelov rovnakých charakteristík pre každú kameru, vrátane referenčnej kamery (minimálne 2 na kameru); každý z `--frames` sa vypočíta z týchto bodov. Ak kamery tvoria prepojené pole, kalibrácia využíva živé snímky tohto poľa a profil sa zároveň stáva zarovnaním poľa pre jeho zábery. V opačnom prípade sa otvorí [dočasné pole](#multi-camera-transient-sessions): kamera, ktorá neposkytuje žiadny snímok v žiadnom z `--frames`, neprejde kalibráciou (ukončí sa `1` s uvedením jej názvu) a profil sa neuloží. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Zachyťte jeden zarovnaný viacpásmový snímok. `--bit-depth` je predvolene nastavený na zosúladenie s kamerou; `--no-crop` zachováva celý snímok (doplní čiernou farbou); `--interpolation` (predvolené `linear`) a `--border-mode`/`--border-value` (predvolené nastavenie `constant`/0) riadia deformáciu CPU — cesta GPU je v každom prípade bilineárna. Otvorí [dočasné pole](#multi-camera-transient-sessions): kamera, ktorá neposkytuje žiadny snímok, zlyhá pri zachytávaní (ukončí sa `1`, pričom sa jej priradí názov) a nič sa nezapíše. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Streamovo zarovnané viacpásmové snímky (rovnaké ovládače warpu ako `align-apply`), predvolené nastavenie `--fps` a `3`. Otvorí [dočasné pole](#multi-camera-transient-sessions): skupina snímok, pre ktorú kamera nedodala žiadne údaje, sa neuloží a tri po sebe idúce snímky ukončia stream. Ak bola akákoľvek skupina preskočená, beh sa ukončí vypísaním `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` a `Error: …`, a ukončí sa výstupom `1`. Prúd, ktorý sa ukončí z dôvodu akejkoľvek inej chyby, tiež vypíše `Error: …` a ukončí sa výstupom `1`. |
| `lattice align-info --profile PATH [--json]` | Zobraziť podrobnosti profilu. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Zmeniť poradie vrstiev. |

#### Index / Matematika vegetácie

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` priraďuje symboly prednastavenia k pásmam** (NDVI: `red`, `nir`), po jednom `--channel SYMBOL=BAND` pre každý.

- **`--live`: iba pre symbol, na ktorý zoskupenie nedokáže odpovedať.** Rovnako ako v prípade `--input`, symbol prednastavenia, ktorý vynecháte, prevezme jedno pásmo zo zásobníka (pásma zapnutých kamier profilu), ktorého názov sa ním začína. Toto sa skontroluje pred otvorením akejkoľvek kamery: symbol, na ktorý neodpovedá žiadne pásmo alebo viacero pásiem, sa v takom prípade odmietne, pričom sa uvedú ich názvy a ukončí sa `1`. Vlastný `--formula` vyžaduje pre každý symbol `--channel`.
- **Produkty `--live` uvádzajú, komu patria.** `index.tif` a `index_LUT.png` obsahujú údaje o kamere zo zoznamu (značka, model, sériové číslo), čas jej záznamu (expozícia referenčnej kamery, UTC) a objektív, pričom obidva uvádzajú index ako svoje jediné pásmo (`XMP-Camera:BandName` `NDVI`; farebný súbor PNG uvádza index, nie svoje farebné kanály), rovnako ako to robí produkt `--input`. Index zložený z pásiem viacerých kamier namiesto toho uvádza názvy každého z nich a jeho `Model` je `LATTICE` — pozri [Súbory vytvorené z viacerých kamier](#files-made-from-several-cameras).
- **`--input`: iba pre symbol, na ktorý súbor nedokáže odpovedať.** Symbol, ktorý vynecháte, sa priradí k tomu pásmu súboru, ktorého názov týmto symbolom začína: `red` preberá `Red_660` alebo `Red_660_RGN`, `nir` preberá `NIR_850`. Názvy pochádzajú zo samotného súboru: z vlastného zoznamu pásiem zásobníka alebo, v prípade záznamu radiancie/reflektancie Chloros alebo produktu `process`, z jeho značiek `Chloros:Band{R,G,B}Name` / `XMP-Camera:BandName`. Preto `lattice index --input <reflectance capture> --preset NDVI` nepotrebuje `--channel`.
- **Dve pásma zodpovedajúce jednému symbolu sa medzi sebou nerozlišujú** (stoh s 9 kamerami má dve NIR pásma): pomenujte ho. Symbol, ktorému nič neodpovedá, sa odmietne pomocou `channel_map missing entries`, ukončite `1`.**Názvy pásiem v súbore** obsahujú filter kamery: pásma kamery M3C sú `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` na kamere typu RGN; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` na kamere typu OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` na kamere typu RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` na NGB — aje `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` uvádza zoznam pásiem profilu.

Úplná sada príznakov: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (opakovateľné), `--capture-level raw|debayered|radiance|reflectance|unknown` (prepíše úroveň zachytenia zaznamenanú v zdrojovom súbore TIFF; predvolené nastavenie: načítané z metadát TIFF, v prípade chýbajúcich údajov sa použije hodnota `Chloros:ProcessingLevel` zo súboru Chloros, takže pri zachytení odrazivosti sa hlási hodnota `reflectance`), `--output PATH` / `-o` (offline: cesta; s príponou obrázku sú súbory pomenované podľa kmeňa — `-o ndvi_FRGN.png` zapíše `ndvi_FRGN.tif` a `ndvi_FRGN_LUT.png` — a bez nej je to predpona, `-o ndvi` -> `ndvi.tif`, `ndvi_LUT.png`; predvolené vedľa vstupu, ako `<input name>_<preset>`; s `--live` a bez `-o`, priečinok `output/` pod priečinkom, z ktorého spúšťate príkaz), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Pri súbore `--live` sa ovládače deformácie zarovnania vzťahujú aj na: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (cieľ automatického nastavenia expozície kamier, pozri pravidlá pre prechodné stavy), `--serials SN1,SN2,…`. `--live` otvorí [prechodné pole](#multi-camera-transient-sessions): kamera, ktorá nedodá žiadny snímok, spôsobí zlyhanie behu (ukončí sa `1` s uvedením jej názvu), a nezapíše sa žiadny zásobník ani index. Otvorí sa `--serials` (prvá spustí spúšť), inak kamery profilu, ktorých položka `layer_order` je povolená, pričom ako prvá sa použije jej referenčná kamera — referenčná kamera aj s deaktivovanými pásmami, keďže ostatné sú zarovnané podľa na ňu a spúšťa spúšťač. Kamera, ktorej pásma sú deaktivované, sa nikdy nekalibruje, takže RGB kamera deaktivovaná v profile už nespôsobí zlyhanie indexu odrazivosti (`no-absolute-radiance-scale`). Profil, ktorý neoznačuje žiadne kamery, otvorí všetky kamery v sieti, ako doteraz.

Každý formulár beží na backende, takže ten musí byť spustený: spustite ho z grafického rozhrania Chloros alebo pomocou príkazu, ktorý CLI vypíše, keď sa k nemu nemôže pripojiť. V režime offline backend číta `--input` a sám zapisuje TIFF s plávajúcou desatinnou čiarkou a farebný PNG, takže relatívny `--input` alebo `-o` sa vzťahuje na priečinok, z ktorého spúšťate príkaz (pozri [Konvencie](#conventions)). Oba offline výstupy obsahujú záznam o fotoaparáte zo vstupného súboru, rovnako ako export indexu `process`: značka / model / sériové číslo, GPS, údaje o objektíve, čas záznamu (vlastný čas vstupného súboru; ak jeho EXIF neuvádza žiadny, ide o okamih expozície, ktorý zaznamenal, alebo časovú pečiatku v názve súboru, v UTC) a identifikačný kód kalibrácie MAPIR — nikdy kalibráciu raw-DN, ktorá sa na index nevzťahuje — plus `Chloros:ProcessingLevel=index` a výraz indexu; plávajúce číslo TIFF označuje jedno pásmo, predvolbu. `--list-presets` a `--list-gradients` vytlačia prednastavenia a gradientové tabuľky backendu. Vzorec, ktorý backend odmietne, alebo pásmo `--channel`, ktoré TIFF nemá, ukončí `1` s dôvodom backendu.

> **Pri symboloch `--channel` sa rozlišujú veľké a malé písmená.** Časť so symbolom sa musí presne zhodovať s názvami kanálov prednastavenia (prednastavenia používajú malé písmená, napr. NDVI = `red`,`nir` — skontrolujte `--list-presets`), a strana s pásmami sa musí zhodovať s názvom pásma v zarovnanom stohu (alebo byť indexom pásma počítaným od 0 v režime offline). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` funguje; `--channel RED=660` zlyhá s chybou `channel_map missing entries` .

#### `lattice index` — masky pre viacpásmové stohy

Pri `--capture-level radiance` alebo `reflectance` môže 3×3 oddeľovanie každej kamery obmedziť pásmo na 0 (pozri [Maska obmedzenia pri oddeľovaní kanálov](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). Viacpásmový stoh obsahuje pásma z viacerých kamier a každá kamera obmedzuje svoje vlastné, takže `--live --save-multiband` zapíše **jednu masku na každú zdrojovú kameru** vedľa stohu:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Každá maska je jednopásmová `uint8` na pixelovej mriežke **stohu** (po tom istom zarovnaní, deformácii a orezať), s vlastnými bitmi danej kamery: bit 0 = pásmo jej kanála R, bit 1 = G, bit 2 = B. Pás zo zoskupenia prevzatý z kanála *c* kamery je obmedzený tam, kde je nastavený bit *c* danej kamery (R = 0, G = 1, B = 2).
- Vložený pôvod každého pásma uvádza názov masky príslušnej kamery (`unmix_clamp_mask`, vzhľadom na zložku stohu, alebo `none`) a bit, ktorý sa má čítať (`unmix_clamp_mask_bit`).
- Kamera, ktorej dekomposícia neobmedzila nič, nedostane žiadny súbor; mono alebo RGB kamera neprebieha dekomposíciu a tiež nedostane žiadny súbor. CLI vypíše každú masku pod stohom (`clamp mask SN…: …`); odpoveď ich uvádza ako `clamp_masks`.
- Indexový súbor TIFF/PNG z toho istého behu sa vypočíta z toho istého zásobníka na tej istej mriežke, takže ho masky tiež opisujú. `--capture-level raw` nevykonáva žiadne oddelenie farieb a nezapíše žiadnu masku.
- Zoznam a index sa nachádzajú priamo v `--output`, takže masky idú do vlastného súboru `clamp_masks/` v rámci `--output` — nikdy nad zložkou, ktorú ste pomenovali. Tento priečinok obsahuje rôzne typy výstupov (zásobník, index, farebný render): vyberte z neho len potrebné súbory, neposielajte celý priečinok do fotogrametrického softvéru.

#### Trvalé pripojenia (Smart-Prep, tok ekvivalentný grafickému rozhraniu)

Tieto príkazy udržujú kamery otvorené v backendovom poole aj medzi jednotlivými spusteniami cez CLI.

| Podpríkaz | Účel |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Pridá jednu kameru do fondu (jedna kamera, bez poľa). [Bežné možnosti](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` a `--packet-size` (`standard` = 1500, `jumbo` = 9000 alebo bajtov; sondy `auto`) sa aplikujú pri pripojení (prepínače typu `--exposure` alebo `--gain`, ktoré automatický režim vypnú). Vráti sa, akonáhle sa spustí prenos z kamery (čaká sa na to približne 15 s, potom 10 s na prvý kompletný buffer) a vypíše `stream: streaming` alebo `stream: no_frames -- connected, no frames yet: …`, ak prenos nedodal nič — v prípade kamery s jumbo paketmi, s odporúčaním opätovného pripojenia s kódom `--packet-size standard`. Kým sa jedna kamera pripája (prvé pripojenie stiahne jej kalibráciu, trvá to približne 70 s), ďalší príkaz, ktorý potrebuje kameru, na ňu čaká maximálne 5 s, potom zlyhá s chybou HTTP 503 `CAMERA_POOL_BUSY` s uvedením názvu kamery, ktorá sa pripája; spustite ho o chvíľu znova. `lattice status` a zoznam kamier nikdy nečakajú: kamera uprostred pripájania sa zobrazuje ako pripájajúca sa, spolu s jej fázou. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Uvoľnenie. Kamera sa vráti do stavu, v akom bola pri pripájaní (spúšťacie zapojenie, PTP, nastavenia); `--keep-settings` namiesto toho uloží svoje nastavenia do vlastnej pamäte, `--clear-settings` vynuluje túto pamäť na továrenské nastavenia (pozri [Pamäť kamery](#camera-memory----keep-settings----clear-settings)). Kamera, ktorá je členom poľa, uvoľní celé pole. |
| `lattice cam-list` | Zoznam kamier v skupine. |
| **`lattice array-connect`**|**Pripojenie trvalého synchronizovaného poľa (ODPORÚČANÝ vstupný bod).** Spustí kompletný proces inteligentnej prípravy v grafickom rozhraní. Trvá to približne minútu pre 9 kamier; hneď po zapnutí kamier to môže trvať niekoľko minút. Počas čakania vypíše každý krok, ktorého backend dosiahne, ako `    <s> s: <step>` (napríklad `    41 s: open SN:214000535 (4 of 9)`), a protokol backendu obsahuje jeden riadok s časom na každý krok (`[array-connect <id>] <step>: <s> s …`) a posledný riadok s najpomalšími krokmi. Čaká až 600 s. Pripojenie, ktoré sa do tej doby neuskutoční, backend zruší: každá kamera, ktorú otvoril, sa vráti do stavu, v akom bola nájdená, a uzavrie sa, príkaz vypíše `Error:` s krokom, ktorého dosiahol, a ukončí sa `1`, pričom žiadne pole nie je pripojené (HTTP 504 `CLIENT_DEADLINE_PASSED`). Klávesová skratka Ctrl+C zruší pripojenie rovnakým spôsobom (`Interrupted: …`); ak sa pole už pripojilo, riadok to uvedie a uvedie `array-disconnect`, ktorý ho uvoľní. Príkaz ukončený počas čakania (`kill -9`) spôsobí, že pripojenie zostane zrušené, ak backend nedostane od neho žiadnu odpoveď po dobu 30 sekúnd. Pole, ktoré je aktívne, ale ešte nemá synchronizovanú skupinu rámcov, sa stále hlási ako pripojené, pričom jeho tok je označený ako `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Uvoľnenie poľa. Každá kamera zostáva v režime voľného chodu (`TriggerMode` `Off`), pričom všetko ostatné zostáva tak, ako to pripojenie zistilo — jej nastavenia spúšťača a synchronizačnej linky (nastavenia hlavnej`TriggerSource` sa vráti do pôvodného stavu, jej synchronizačná linka sa vráti na vstup), jej PTP a nastavenia kamery, ktoré relácia zmenila — takže príkaz pre jednu kameru na členskom zariadení funguje okamžite. `--keep-settings` uloží nastavenia každej kamerytak, ako ich zanechala relácia, do vlastnej pamäte kamery; `--clear-settings` obnoví túto pamäť na továrenské nastavenia. Pozri [Pamäť kamery](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Zoznam pripojených polí. |
| `lattice array-status [--array-id ID]` | Snímková frekvencia v reálnom čase, PTP, posledná chyba. `ptp_enabled` je rovnaké ako `True` len vtedy, ak každý člen má zapnuté čítanie PTP; `ptp_off:` označuje člena, ktorý ho nemá zapnuté. Riadok `state:` číta hodnotu `restarting (applying settings)` počas reštartu nastavení, hodnotu `recovering (…)` pri opätovnom otvorení člena, ktorý stratil napájanie (pozri poznámku nižšie), `stalled (…)` počas behu cyklu zachytávania, ktorý však nezverejňuje žiadnu skupinu rámcov, a `stopped (…)` po tom, čo cyklus zachytávania skončil (znovu pripojte pole). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Jedno synchronizované zachytenie z aktívneho poľa — Jednorazové / Nepretržité / Intervalové / Najrýchlejšie. **Predvolené nastavenie je `all`**(jeden súbor na príslušný typ exportu na kameru),**zarovnané** tak, ako je zachytenie v aplikácii (`--no-aligned` pre natívne súbory). Preskočené kamery (napr. RGB vylúčené z radiancie/reflexie) sú hlásené s kódom `Skipped: SN:<serial> (<reason>)`; hodnota DAQ, ktorá načítala použitý záznam, sa uloží spolu s ním, vrátane záznamu len v surovom formáte, a je hlásená s kódom `DAQ: <path>` ([Hodnota DAQ, ktorá načítala záznam, sa uloží](#the-daq-reading-a-capture-saves)). Pozri [Režimy snímania, záznamníky a offline spracovanie](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Nahrávanie živého zobrazenia kombinovaného indexu do videa/GIF (monitorovacia kvalita; ak nie je otvorené grafické rozhranie, backend vykresľuje NDVI poľa, alebo `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Séria surových snímok Bayer s vysokou snímkovou frekvenciou (na účely analýzy; spracovanie offline). Podobne ako `array-record` a `array-capture --continuous` sa pri každom dotazovaní hlási backendu: ukončí sa bez zastavenia (`kill -9`), backend ho zastaví 30 s po poslednej kontrole (`stop_reason` `client_gone`; manifest série snímok je finalizovaný a začalo sa vytváranie kombinovaného indexu, rovnako ako pri zastavení) a pole zostáva pripojené. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Opätovné spracovanie uloženého surového súboru snímok na kalibrované video(-á). |

> **Kamera, ktorá stratila napájanie alebo spojenie, sa sama opäť pripojí.** Keď člen prestane odpovedať (vypnutie a zapnutie napájania prepínača reštartuje každú kameru, ktorú napája), pole na ňu počká, opäť ju otvorí, keď začne opäť odpovedať, a vráti jej nastavenia, ktoré určil `array-connect`: PTP, spúšťaciu a synchronizačnú linku, sieťové tempo, formát pixelov a veľkosť snímky, ako aj automatickú expozíciu. Následne opäť vysiela bez nového `array-connect`. Medzitým `array-status` zobrazuje `state: recovering (...)` a `last_error` uvádza, na ktorú kameru čaká; následne vypíše `re-opened:`. Fotoaparát, ktorý sa reštartoval, sa vráti s vypnutým PTP, takže kým sa znovu neotvorí, `ptp_enabled` je `False` a `ptp_off:` ho pomenuje.

> **Zadržaná kamera sa nikdy neotvorí druhýkrát.** Druhé otvorenie zastaví prúd zadržiavateľa a prepíše jeho nastavenia, čo môže spôsobiť zlyhanie backendu. Takže v prípade kamery, ktorú drží pool (GUI alebo `cam-connect`), člena poľa alebo kamery, ktorú drží úloha na pozadí (bežiaci `lattice continuous`, `multi-stream` alebo `align-stream`, či už vlastná alebo iného klienta), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, jednorazový-shot `white-balance` a `presets --apply` odmietnu s HTTP 409 `CAMERA_HELD`, pričom uvedú držiteľa a príkaz na uvoľnenie (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>` alebo v prípade úlohy jej ID a meno osoby, ktorá ju spustila) a ukončenie `1`. To isté platí pre kameru, ktorú práve používa iný príkaz (napr. `multi-test` alebo `multi-capture` otvárajúci jej kamery, ďalší jednorazový príkaz), pre každý príkaz tu: chyba hovorí, že treba počkať, kým sa tento príkaz dokončí. `lattice power` bez `--sleep` prečíta vlastnú referenciu držiteľa, ak nejaká existuje (pool, pole, otvorená kamera úlohy), a je odmietnutý v prípade kamery, ktorú práve používa iný príkaz. Každá odpoveď `CAMERA_HELD` obsahuje kód v položkách `code` aj `error_code`. Pokiaľ ide o príkazy, ktoré otvárajú všetky kamery, pozri [Viac kamier (dočasné relácie)](#multi-camera-transient-sessions).

##### Možnosti `array-connect`

| Príznak | Predvolené | Popis |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatické vyhľadanie všetkých kamier LATTICE (vyžaduje ≥2) | Prvá séria je MASTER. Ak sa vynechá, vyhľadávanie sa obmedzí na modely LATTICE (M3C / M3M) a pripojí sa všetky z nich. |
| `--line {Line0,Line2,Line3}` | `Line2` | Synchronizačná linka GPIO. |
| `--target-fps F` | auto | Rýchlosť spúšťania Mastera. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Prepis výberu úrovne. |
| `--wire-ceiling-mbps MB_PER_S` | automaticky detekované | **Trvalá šírka pásma hostiteľa v MB/s — hodnota, od ktorej závisí alokácia celého poľa.** Znížte ho, ak pole hlási poškodené rámce GVSP: automatická hodnota je odvodená od deklarovanej rýchlosti spojenia sieťovej karty, ktorá nadhodnocuje USB adaptéry, úzke PCIe kanály a vyťažené zdieľané štruktúry. Ukladá sa v bloku zachytenia poľa projektu, takže sa obnoví pri opätovnom otvorení alebo pri opätovnom pripojení cez CLI/SDK. Pozri [Stav poľa](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | žiadne (celý senzor) | Hardvérové zlučovanie (vždy priemer). |
| `--width N` / `--height N` / `--pixel-format FMT` | celý snímač; formát pre každú kameru | Explicitná veľkosť snímky alebo formát pixelov pre každú kameru namiesto výberu celého snímača a formátu pre každú kameru (Mono12 / BayerRG12). |
| `--no-recommend` | vypnuté | Preskočiť krok analýzy siete (každá kamera si potom zachová svoju aktuálnu veľkosť namiesto toho, aby bola nastavená na celý snímač). |
| `--no-ptp` | vypnuté | Zakázať PTP (časové značky medzi kamerami potom **nie sú** porovnateľné). |
| `--packed {auto,on,off}` | `auto` | Streamujte 12-bitové snímky v komprimovanej forme a dekomprimujte ich na tomto počítači — viac snímok za sekundu pri plnom rozlíšení snímača, identické údaje. `auto` komprimuje len vtedy, keď je to užitočné a tento počítač to zvládne; `off` nikdy nekomprimuje. Pozri [Komprimovaný 12-bitový prenos](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | Veľkosť paketu GVSP pre každú kameru v poli. `auto` umožňuje každej kamere testovať sieť; `jumbo` = 9000, `standard` = 1500 alebo veľkosť v bajtoch ju obmedzuje. Pred verziou 1.3.0 `array-connect` túto vlajku prijal, ale ignoroval ju. |

> **Predvolene plný senzor.** Bez `--width` / `--height` / `--binning` /
> `--pixel-format`, `array-connect` nastaví každú kameru na plný senzor — bez
> zlučovania, žiadne orezávanie ROI — a nechá snímkovú frekvenciu klesnúť na úroveň, ktorú tento počítač a
> sieť poskytujú, namiesto zmenšovania snímok s cieľom udržať frekvenciu.
> Riadok s analýzou siete zobrazuje predpokladanú frekvenciu a nasledujúci riadok uvádza
> rýchlejšie možnosti (binning alebo menšie ROI) s ich predpokladanými frekvenciami; prejdite
> `--binning` alebo `--width`/`--height`, aby ste si jednu zvolili. Príkaz Connect nikdy sám o sebe nevykonáva zlučovanie ani orezávanie
> snímok.

##### Komprimovaný 12-bitový prenos (`--packed`)

Pri plnom využití senzora je snímková frekvencia poľa určená tým, ako rýchlo môže každá kamera odosielať
svoje snímky. 12-bitové formáty (`BayerRG12`, `Mono12`) sa zvyčajne prenášajú ako 16-bitové
slová; **komprimovaný** prenos odosiela tých istých 12 bitov v 1,5 bajte na pixel a
ich na tomto počítači rozbalí, takže odosielanie každého snímku trvá o 25 % menej času —
približne **o 33 % viac snímok za sekundu pri plnom využití snímača** (merané: 10,7 → 14,0 fps
na sústave s 9 kamerami). **Uložené údaje sú identické**: rovnaké hodnoty, rovnaké
16-bitové súbory, rovnaké metadáta.

- `auto` (predvolené nastavenie) komprimuje len vtedy, ak to má zmysel **a** tento počítač si môže dovoliť
  dekompresiu. Pri pripojení backend zmeria svoju vlastnú rýchlosť rozbaľovania, počet jadier procesora,
  aktuálne zaťaženie procesora a sieťovú kapacitu, takže stolný počítač s voľnými jadrami komprimuje
  pri plnom rozsahu snímača, zatiaľ čo vyťažený alebo počítač s nízkym výkonom streamuje nerozbalené dáta. Malé oblasti záujmu (ROI) a zlučované snímky prinášajú
  len málo.
- `off` nikdy nekomprimuje — použite ho na vylúčenie komprimácie počas diagnostiky alebo na
  počítači, ktorý musí vyhrať celý svoj procesor pre inú prácu.
- `on` vždy komprimuje 12-bitové formáty.

Výstup connect zobrazuje rozhodnutie a dôvod (`komprimované 12-bitové: zapnuté (auto:
predpokladaných 11,0 -> 14,4 fps, …)`); `/api/camera/array/status` hlási
`packed_transport` a `packed_decision`. Ak chcete nastaviť predvolené nastavenie pre celý počítač
namiesto toho, aby ste vlajku zadávali zakaždým, nastavte `CHLOROS_ARRAY_PACKED12=off` (alebo
`on`) v prostredí backendu — explicitné nastavenie `--packed on|off` má však prednosť.
8-bitové a 16-bitové formáty sa nikdy nekomprimujú.

### Jeden projektový záznam, všetky režimy

Zložka projektu obsahuje nastavenia každého zariadenia: `cameras.json` — pre každú kameru (`cameras[].settings`, pre jednu kameru aj pre každý člen poľa) a nastavenia platné pre celé pole (`arrays[].capture`: formát, ROI, binning, frekvencia spúšťania, rozpočet na prenos) — a `sensors.json`, pre každý svetelný senzor (integračný čas, priemer snímky, limit). Aplikácia, CLI, SDK a server MCP čítajú a zapisujú do rovnakých súborov rovnakým spôsobom:

- **Zariadenie pripojené v rámci projektu sa do neho uloží a zaznamenáva do neho každú zmenu.** „V rámci projektu“ znamená s otvoreným projektom v backende (aplikácie) alebo — z SDK — prostredníctvom súboru `ChlorosProject`, ktorý určuje názov vlastnej zložky. Kamera, pole alebo svetelný senzor, ktoré projekt ešte neobsahuje, sa pridajú do jeho súborov pri pripojení (`lattice cam-connect`, `array-connect`, `daq pool-connect`, aplikácia, SDK, `chloros-mcp`) spolu so svojimi nastaveniami. Zmena z akéhokoľvek režimu (`lattice exposure`, `gain`, `configure --set`, `resolution`, nastavenia poľaa frekvencie spúšťania, kryt svetelného senzora, ovládacie prvky aplikácie) sa o chvíľu uloží do súborov daného projektu. Expozícia, zosilnenie alebo cieľová hodnota automatického nastavenia expozície (AE) nastavené na fotoaparáte sa uložia ako aktuálne hodnoty, takže pri opätovnom otvorení sa tieto hodnoty zachovajú.
- **Otvorenie projektu nastaví zariadenia podľa neho** vo všetkých režimoch: fotoaparát, pole alebo svetelný senzor pripojený bez vlastných nastavení prevezme uložené nastavenia projektu (`lattice cam-connect` / `array-connect` pri otvorenom projekte, `chloros-cli project connect`, `open_project().connect_all()`, opätovné otvorenie aplikácie). Pole pripojené pod novým identifikátorom aktualizuje pole, ktoré projekt uložil s rovnakými kamerami; pole s inými kamerami sa pridá vedľa neho.
- **Vlastné uloženie aplikácie nikdy neprepíše aktuálne nastavenia zariadenia** ich staršou kópiou a zachová pre každý prvok prekrytia uložené volajúcim z CLI alebo SDK.
- **Formát pixelov, ktorý nastavíte, je ten, ktorý si projekt uchováva.** Formát nastavený v nastaveniach poľa, pomocou `--pixel-format` alebo `connect_array(pixel_format=)`, vrátane 8-bitového, sa uloží a pri opätovnom pripojení sa presne zachová. Ak nie je nastavený žiadny formát, každá kamera sa pripojí vo svojom predvolenom 12-bitovom nastavení (`BayerRG12`, `Mono12` pri monochromatickom senzore). Názov formátu označuje jeho bitovú hĺbku; rodina zodpovedá senzoru, takže farebná kamera sa nikdy nenastaví na formát Mono ani monochromatická kamera na formát Bayer, a riadok `[PixelFormat <serial>] ... written` to uvádza, keď bol formát priradený. Exporty 8-bitový záznam sa exportuje ako 8-bitový, vrátane odrazovej hodnoty: je preškálovaný a označený kódom `PixelScale`.
- Zariadenie pripojené bez projektu nezaznamenáva nič.
- **Záznam sa priraďuje k projektu vytvorenému po ňom; otvorenie projektu ponechá záznam v pôvodnom projekte.**Keď sa vytvorí projekt — SDK `create_project()`, `chloros-mcp` alebo**Nový projekt**v aplikácii — alebo duplikovaný (funkcia**Duplikovať**v aplikácii, ktorá otvorí kópiu), pričom sa záznam stále zapisuje do projektu, ktorý bol otvorený predtým, záznam pokračuje a to, čo sa zapisuje, sa uloží do nového projektu, na rovnaké miesto v ňom (nižšie). Otvorenie existujúceho projektu — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, funkcia aplikácie**Otvoriť projekt** — ponechá nahrávanie stále prebiehať v projekte, do ktorého sa nahráva: tam sa dokončí a jej súbory sa pridajú do zoznamu súborov daného projektu (aplikácia zastaví každé nahrávanie pred otvorením projektu).
  - `array-record` a `array-burst`: video alebo zložka sériových snímok so všetkými snímkami sa tam presunie ako celok po zastavení nahrávania (je to `--duration`, Ctrl+C). `Burst dir`, ktorý sa vypíše, je nový, a `--build` a `array-build-video` pracujú na jeho základe. Ak disk odmietne presun, nahrávka zostane tam, kde bola zapísaná; protokol na koncikoniec protokolu uvádza dôvod, rovnako ako aj HTTP-odpoveď pri zastavení (`relocation_error`). Surové snímky zastavenej série a `.daq` sa pridajú do zoznamu súborov projektu, do ktorého sa dostanú; video (`array-record`, video vytvorené zo série záznamov) sa do zoznamu súborov projektu nepridá.
  - `array-capture --continuous`: každé zachytenie po zmene sa uloží do nového projektu; tie, ktoré boli uložené predtým, zostanú tam, kde sú.
  - `daq pool-record` a `project sensor log` (a vlastné nahrávanie aplikácie pomocou svetelného senzora): súbor `.daq` sa po ukončení nahrávania presunie do zložky nového projektuzložku `light_sensor` a zaradí sa do zoznamu súborov daného projektu.

  Záznam do zložky mimo otvoreného projektu (napr. `-o` niekde inde) zostáva tam, kde je, a súbory `lattice continuous` a `multi-stream` si zachovávajú zložku, do ktorej boli uložené.
- **V aplikácii sa záznamy po vytvorení nového projektu uložia do neho.** Vytvorenie alebo duplikovanie projektu ponechá všetky prebiehajúce snímacie úlohy a nahrávanie v chode a každý záznam zhotovený po ňom (Zaznamenať všetko, Spustiť zaznamenávanie) sa uloží do nového projektu a pridá sa do jeho zoznamu súborov. Otvorenie existujúceho projektu v aplikácii najprv zastaví všetky prebiehajúce úlohy.
- **Projekt sa počas nahrávania nepresúva.**Presun otvoreného projektu do iného pracovného adresára (v aplikácii v časti**Nastavenia → Pracovný adresár**; žiadny príkaz CLI nepresúva projekt, ale HTTP klient odosielajúci na `/api/move-project` áno) je odmietnuté s chybou HTTP 409 `PROJECT_RECORDING`, pokiaľ do neho zapisuje záznam poľa alebo séria snímok, nepretržité snímanie poľa, úloha typu `lattice continuous` / `multi-stream` alebo sa do neho zapisuje záznam zo svetelného senzora: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` Aplikácia to odmietne rovnakým spôsobom, ešte predtým, ako požiada o zadaní priečinka, pokiaľ beží jej vlastná funkcia „Zachytiť všetko“, „Začať snímanie“ alebo záznam zo senzora osvetlenia.

Toto je odlišné od toho, v akom stave zostanú kamery po odpojení (pozri [Pamäť kamery](#camera-memory----keep-settings----clear-settings)): projekt si nastaví svoje kamery sám pri otvorení.

### Pamäť kamery — `--keep-settings` / `--clear-settings`

Keď je pripojená skupina kamier, ich kamery preberú nastavenia skupiny a vy ich potom môžete zmeniť. Keď Chloros uvoľní kamery — `array-disconnect`, koniec prechodného príkazu (`multi-capture` bez poľa, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), zastavenie `lattice continuous`, ukončenie `lattice capture` kamery, ktorú neobsahuje — každú z nich vráti do stavu, v akom bola pred pripojením: jej spúšť a zapojenie synchronizačnej linky, nastavenia PTP a kamery (úplný zoznam je v časti [Viac kamier (dočasné relácie)](#multi-camera-transient-sessions)).

Ak chcete používať kamery mimo projektu, dve príznaky v ktoromkoľvek z týchto príkazov zapíšu do **vlastnej pamäte** kamery — to, s čím sa spustí v akomkoľvek softvéri:

- **`--keep-settings`** uloží nastavenia každej kamery tak, ako ich relácia zanechala v jej pamäti (UserSet2, vytvorené ako spúšťacie nastavenie). Kamery naďalej bežia vo voľnom režime (`TriggerMode` `Off`), pričom zapojenie synchronizačnej linky zostáva nezmenené. Kamera bez UserSet2 ukladá do UserSet1; tento počítač si to pamätá, takže Chlorosovo pole sa samoopraví — čo resetuje UserSet1, ktorý sa odchýlil od základnej hodnoty systému Chloros — ho ponechá bez zmeny.
- **`--clear-settings`** resetuje pamäť každej kamery na továrenské nastavenia: základnú hodnotu systému Chloros, ktorú jeho pole s automatickou opravou zakotvilo — továrenské predvolby s `TriggerMode`, `Off`, `TriggerSource`, `Line2`, automatická expozícia, kalibrovaná vyváženosť bielej RGB-kamery zachovaná — v UserSet1, vytvorená ako spúšťací súbor.
- Tieto dve možnosti nie je možné kombinovať. Každá kamera dostane riadok: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` alebo `SN:…: camera memory reset to the factory settings.`; odpoveď `camera_memory` je pre každú kameru rovnaká.

Aplikácia tieto príznaky neponúka.

**Každý spôsob ukončenia vráti kamery do pôvodného stavu.**Rovnaké vrátenie do pôvodného stavu prebieha pri každej ceste ukončenia, nielen pri vyššie uvedených príkazoch: `cam-disconnect` (a kamera**X**v aplikácii);**X** v aplikácii na poli alebo v riadku ktoréhokoľvek člena poľa (pole nemôže bežať bez člena, takže sa uvoľní celé pole); zatvorenie aplikácie (dáva backendu až 20 s na vrátenie každej kamery, pole a spustený príkaz `lattice continuous` späť, než ju zastaví; príkaz, ktorý v tom čase ešte beží — prechodný príkaz otvárajúci svoje kamery alebo čakajúci na ich PTP hodiny, jednorazový príkaz `lattice capture`, `power`, `white-balance` alebo `presets --apply` – sa zastaví v najbližšom bezpečnom bode, najskôr vráti svoje kamery späť, a to až na 8 s, a odpovie HTTP 503 `SHUTTING_DOWN`; stále bežiace pripojenie typu array — `lattice array-connect`, pripojenie aplikácie — sa zastaví pred svojou ďalšou kamerou a vráti kamery, ktoré otvorilo, a to až na 10 s, a odpovie HTTP 503 `SHUTTING_DOWN`; a ten, ktorý dorazí počas vypínania, nič neotvorí: HTTP 503 `SHUTTING_DOWN`); a príkaz CLI, ktorý je ukončený alebo ktorého terminál sa zatvorí (SIGTERM / SIGHUP zastavia jeho úlohu — a vráti kamery späť — predtým, ako sa ukončí (`143` / `129`, podobne ako Ctrl+C). `lattice continuous`, `multi-stream` alebo `align-stream`, ktoré boli priamo ukončené (SIGKILL) a nemôžu nič zastaviť, majú svoju úlohu zastavenú a kamery vrátené do pôvodného stavu backendom 30 s po jej poslednej aktualizácii. Backend, ktorý sa zrúti alebo je násilne ukončený, nemôže nič obnoviť; reštart vráti každú kameru do jej pamäte a ďalšie pripojenie Chlorosu prepojí kameru, ktorú nájde v pohotovostnom režime.**Projekt si nastavuje svoje kamery sám.** Keď sa pole pripojí k otvorenému projektu, každý člen dostane nastavenia, ktoré pre neho projekt uložil — expozíciu, zosilnenie alebo cieľovú hodnotu automatického nastavenia expozície (udržované ako vaše, takže automatické nastavenie expozície ich nemení) a režim vyváženia bielej (korekcia Chloros; vlastné vyváženie bielej kamery zostáva vypnuté) — čokoľvek, čo obsahuje pamäť kamery.

### Smart-AE / Smart-Capture

Polia LATTICE spúšťajú nepretržitú automatickú expozíciu (AE) na pozadí hneď po pripojení, ale novozameraná scéna si vyžaduje chvíľu na konvergenciu. `array-capture --smart` je **pripravené riešenie**: počká, kým sa AE ustáli na všetkých kamerách v poli, a potom spustí snímanie. Použite ho, keď meníte scény uprostred relácie.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Predvolené nastavenie stabilizácie je konzervatívne: časový limit 5 s, okno stability 1,5 s, tolerancia rozpätia expozície ±5 %. Ak potrebujete od automatizácie odlišné správanie, nastavte ho prostredníctvom SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`).

#### Ako funguje automatická expozícia

- **Najprv expozícia, až potom zosilnenie.** Ak je zosilnenie nastavené na Auto, zostáva na hodnote 0 dB, kým expozícia nedosiahne najdlhšiu povolenú hodnotu.
- **Pri manuálnej expozícii zostáva zosilnenie v režime Auto**, kým nezmeníte zosilnenie alebo ho nenastavíte na manuálny režim. Po návrate do režimu Auto je limit zosilnenia taký, aký bol predtým.
- **Malé preosvetlené oblasti nezatemňujú scénu.** Odlesky do 1 % záberu sa ignorujú a v prípade panelov známeho kalibračného terča až do 0,5 % panelu.
- **Horná hranica expozície DAQ platí len vtedy, keď je v zornom poli kalibračný terč.** Zabraňuje preexponovaniu bieleho panelu; ak nie je vidieť žiadny panel, kamera exponuje podľa scény.
- **Konštantné osvetlenie zostáva stabilné a zmenené osvetlenie sa ustáli v priebehu niekoľkých sekúnd.** Pri konštantnom osvetlení zostávajú expozícia a jej limit nezmenené a po výraznej zmene osvetlenia sa obraz ustáli v priebehu niekoľkých sekúnd. Preexponovanie už natrvalo neobmedzuje expozíciu a pri zmene oblasti záujmu (ROI) sa expozícia opäť nastaví.
- **Cieľová hodnota jasu zostáva v dosahu.** Nezvyšuje sa, pokiaľ je kamera na limite expozície a jej zosilnenie nemôže stúpať, takže svetlo, ktoré sa vráti, nespôsobí prežiarenie záberu.
- **Kamera, ktorej automatická expozícia prestane reagovať, zostane nastavená na úroveň zodpovedajúcu aktuálnemu osvetleniu.** Pri jasnom slnku by kamera mohla klesnúť na najkratšiu expozíciu a zostať tam s tmavým záberom, pričom všetky jej nastavenia vykazujú normálne hodnoty. Chloros sleduje záber, udržiava túto kameru na úrovni, ktorú si zachovala pre dané osvetlenie, a uvoľní ju, keď sa osvetlenie zmení. Tmavý alebo zakrytý svetelný senzor to nenaruší.

### Úrovne exportu záberov (predvolené nastavenie `all`)

Od tejto verzie, `lattice capture`, `lattice multi-capture` a `lattice array-capture` **predvolené nastavenie je `--processing all`** — jeden uložený súbor na typ exportu, ktorý platí pre každú kameru, čo zodpovedá správaniu funkcie „Zachytiť všetko“ v grafickom rozhraní. Úrovne sú:

| Úroveň | Výstup | Platí pre |
| --- | --- | --- |
| `raw` | Jednokanálový Bayer (mono kamery: jeden pás) priamo zo senzora. | Všetky kamery. |
| `debayered` | 3-kanálová farba, uložená v poradí kanálov **RGB**(súbor TIFF s fotometrickou interpretáciou RGB; súbory PNG/JPEG sa rovnako otvárajú ako RGB): tmavý posun (DSNU) z kalibračného balíka odpočítaný od surových údajov, potom demosaik s ohľadom na hrany pomocou OpenCV— bez LUT linearity, vyrovnania poľa ani oddeľovania farieb (čiernobiele kamery: 1-kanálová škála sivej). Snímky bez bayerského filtra v pamäti, ktoré vráti Python SDK (`grab()`, `frame_stream()`), sú v poradí**BGR** podľa OpenCV; len uložený súbor je v formáte RGB. | Všetky kamery. |
| `radiance` | float32 W/m²/sr/nm prostredníctvom úplného radiometrického reťazca. | Iba multispektrálne (M3C/M3M) — **vynechané pre kamery s RGB filtrom**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), pripravené pre Pix4D. | Iba multispektrálne a **len ak je priradený DAQ a kamera je kalibrovaná**; inak sa preskočí. |
| `display` | Úplný reťazec náhľadu v grafickom rozhraní (CCM + WB + gama podľa profilu kamery). Uložené v priečinku `preview/`. `lattice capture` akceptuje pre tento reťazec aj názov `preview`; `array-capture` / `multi-capture` prijímajú iba `display`. | Všetky kamery. |

Zadajte jednu úroveň, aby ste uložili len túto jednu (`--processing debayered`). Keď požiadate o `all`, úrovne, ktoré sa nevzťahujú na danú kameru, sa preskočia (a nahlásia sa), nie sú označené ako chyba — nepripojená alebo nekalibrovaná kamera stále dostane `raw` / `debayered` / `preview`.

#### DAQ, ktorý číta záznam, ukladá

Vždy, keď je DAQ priradený ku kamere alebo sústave, alebo je zaradený do skupiny, hodnota smerujúca nadol použitá pri čítaní záznamu sa zapíše do súboru **`.daq`** v podpriečinku `daq/` priečinka s zachytenými údajmi, spolu s údajom `.match.json` (ako ďaleko bola hodnota od expozície) a `.cameras.json` (kto je autorom hodnoty), a je uvedená v riadku `DAQ:`. To platí pre každú úroveň, vrátane surových alebo debayered záznamov, takže akýkoľvek záznam je možné neskôr opätovne spracovať na radianciu, odrazivosť alebo index. Ide o ten istý súbor s rovnakým názvom a doplnkovými súbormi, bez ohľadu na to, či bol záznam získaný prostredníctvom grafického používateľského rozhrania (GUI), príkazového riadku (CLI) alebo vývojárskeho balíka (SDK). Ak neexistuje žiadna hodnota z DAQ, neexistuje ani položka `.daq`, a položka `multi-capture` bez pripojeného poľa (dočasná náhradná možnosť) neuloží nič.

| Priepínač | Účinok |
| --- | --- |
| `--no-daq` | Neukladať hodnotu z merania, ak ju príslušné úrovne nepoužívajú (`raw`, `debayered`). Úroveň `radiance`, `reflectance` alebo `display` ju stále zapisuje, pretože opätovný import potrebuje hodnotu, s ktorou bol produkt vytvorený. |
| `--daq-tolerance-ms MS` | Aká veľká môže byť odchýlka nameranej hodnoty od expozície a stále sa považovať za v tolerancii (predvolená hodnota 500). Mení sa len výsledný verdikt (u `.match.json`, `in_tolerance` a `tolerance_ms`, u odrazivosti `DlsInTolerance`), nikdy sa nemení použitá nameraná hodnota. Pozri [Hodnota DAQ vzdialená od expozície](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | Starší spôsob požiadania o hodnotu `.daq` pri snímaní iba v surovom formáte. Teraz je to predvolené nastavenie a hodnoty `lattice capture` a `array-capture` tento príznak stále akceptujú (`multi-capture` nie). `--no-daq` má prednosť pred ním. |

Ktorý DAQ: ten viazaný na kameru alebo pole (výber svetelného senzora v grafickom rozhraní, uložená väzba projektu alebo `chloros-cli daq pool-bind`), inak najnovší, ktorý má backend k dispozícii. Séria surových dát (`array-burst`) si zachováva vlastný súbor `.daq` pre každé samostatné meranie v priečinku burst, rovnako ako predtým.

**Formáty PNG a JPEG obsahujú 8 bitov na kanál, preto využívajú iba súbory `display` / `preview` a indexové súbory.** Súbory typu `--format png` alebo `jpg` s úrovňou, ktorú nedokážu obsiahnuť, sú odmietnuté ešte pred zachytením akýchkoľvek údajov, pričom sa uvedie úroveň a formát TIFF, ktorý ju podporuje:

- `radiance` (float32) a `reflectance` (uint16, `32768` = 1,0) — vždy, takže `--processing all` (predvolené) s hodnotami `--format png`/`jpg` je tiež odmietnuté: použite `--format tiff` alebo `--levels display`.
- `raw` a `debayered` z kamery vysielajúcej viac ako 8 bitov (12-bitové snímky M3C / Mono12 M3M) — CLI nedokáže zistiť bitovú hĺbku kamery, preto backend tieto snímky odmietne (HTTP 400). Jednotlivýsnímka `lattice capture --processing debayered` zachytáva 8-bitový snímok pre formáty PNG/JPEG a nie je odmietnutá, rovnako ako `multi-capture` bez trvalého poľa (8-bitový debayered).

Každá trasa zachytávania odpovedá rovnakým spôsobom (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst` a HTTP-trasy za nimi.

### Ako vyzerá priečinok so záznamami

Každý typ exportu sa ukladá do **vlastného podpriečinka**v rámci priečinka `-o`, takže pri viacúrovňovom zázname sa typy nikdy nemiešajú. Nákres znázorňuje**predvolený záznam `all` z jednej kamery pomocou `lattice capture`** — niekoľko úrovní na jednu kameru, takže každý súbor má príponu `_<level>`:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Ako sa menia názvy súborov:

- **Pole kamery — sériové číslo, potom filter:** každý súbor so záznamom uvádza názov kamery, ktorá ho nasnímala, ako `SN<serial>-<filter>`, kde `<filter>` je časť modelu kamery označujúca filter: `SN261902142-F450` pre `M3M-L87-F450`, `SN213800234-FRGN` pre model `M3C-L87-FRGN`, `SN214701292-FRGB` pre RGB kameru. Filter spája sériové číslo s `-`, nie s `_`, takže názov má rovnaké polia oddelené znakom `_` ako predtým (predpona, dátum, čas, milisekundy, kamera, úroveň). Kamera, ktorej model sa nedá prečítať (vymazané DeviceUserID), je pomenovaná len ako `SN<serial>`. Záznamy z predchádzajúcich verzií nemajú v názve žiadny filter; program Chloros číta obidva typy.
- **Predpona:** `lattice capture` pomenúva súbory `capture_…`; `array-capture`, `multi-capture` s pripojeným poľom a `project capture` / `burst` / `stream` z poľa ich pomenuje ako `sync_<ts>_SN<serial>-<filter>…`.
- **Prípona úrovne sa používa len pri viacerých úrovniach.**Kamera, ktorá ukladá**viacero**úrovní v jednom zázname (`all` alebo `--levels a,b`), pridáva `_<level>`. Kamera, ktorá ukladá**jednu** úroveň si zachováva jednoduchý názov bez prípony — `--processing reflectance` zapíše `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` — takže skripty napísané pre jednúrovňové zábery naďalej fungujú. Úroveň je naďalej určená zložkou.
- **Masky obmedzenia nasledujú za názvom produktu:** `<product name>_clampmask.tif`. Zachytenie odrazivosti na jednej úrovni zapíše `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; zachytenie `all` zapíše `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **Rozšírenie** nasleduje po `--format`: predvolené `tiff` dáva `.tiff`, `--format tif` dáva `.tif` (masky obmedzenia sú vždy `.tif`).
- **Jedna výnimka:** `lattice capture --processing debayered` (alebo `--levels debayered`) prijíma rýchlu jedinúa zapíše `capture_<ts>_SN<serial>-<filter>.tiff` priamo do `-o` — bez podadresára.
- **Prúdy a série** obsahujú rovnaké pole: `lattice continuous` zapíše `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` a surovú sériu `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` je časová pečiatka záznamu a `<serial>` sériové číslo kamery, takže jedna synchronizovaná skupina zdieľa
časovú pečiatku medzi kamerami. **Všimnite si jednu asymetriu:** úroveň `display` je uložená v priečinku
s názvom `preview/`, zatiaľ čo súbory s viacerými úrovňami zachovávajú `_display` v názve — zložka a prípona sa líšia
len pre túto úroveň. Neznáme úrovne sa ukladajú do zložky s vlastným názvom a ak sa podzložka
nedá vytvoriť, súbor sa zapíše do koreňového adresára výstupu, namiesto toho, aby sa stratil.

**Opätovné spracovanie zložky so záznamami:**nasmerujte `chloros-cli process` na**koreňový adresár zachytených snímok**
(`output/`). `process` zvyčajne importuje len zložku, ktorú určíte, ale ak táto zložka neobsahuje žiadne
obrázky a obsahuje podzložky, automaticky prechádza do nižších úrovní — takže podzložky na danej úrovni aj
podzložka `daq/` sa načítajú naraz. Každá úroveň zachytenia sa importuje ako jeden obrázok, pričom
ostatné úrovne sú k dispozícii ako režimy, a nie ako jeden obrázok na úroveň.

Priamo pomenovať **podzložku úrovne** (napr. `output/raw/`) tiež funguje. Týmto spôsobom zostane `daq/`
vynechaný, takže skopírujte alebo nasmerujte čítanie DAQ vedľa neho, keďodvádzať radiometrický
produkt z `raw/` — inak sa časová pečiatka nebude mať s čím porovnať.

**Spracovanie vždy začína od `raw`.** V rámci každého záznamu je zdrojom spracovateľského reťazca surový snímok;
`debayered`, `radiance`, `reflectance` a `preview` sa objavujú ako režimy zobrazenia, ale nikdy sa nevracajú
späť do spracovateľského reťazca. Opätovné spracovanie odvodeného produktu by znamenalo opätovné uplatnenie vinety, CCM a
výpočtov žiarivosti, ktoré sú už zapracované do jeho pixelov, preto Chloros uprednostňuje odmietnutie pred
dvojitým spracovaním. Dva dôsledky, o ktorých stojí za to vedieť:

- Výstupy `index/` a `composite/` sa **nikdy** nespracované. Ide o výstupy, nie o snímky —
  render s LUT pre NDVI nemá žiadnu zmysluplnú interpretáciu žiarivosti.
- Zložka so snímkami exportovaná **bez** `raw` (napr. `array-capture --processing reflectance`) nemá
  žiadny platný zdroj v spracovateľskom reťazci. Tieto záznamy sa importujú a zobrazujú normálne, ale `process` ich preskočí
  a oznámi to takto:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Ak skutočne potrebujete posunúť odvodený produkt ďalej — reláciu hubu zachytenú s
  zapnutým `demosaic` alebo zo staršej zložky — `--input-level {raw,debayered,processed}` vynúti vstupný
  bod a prekonfiguruje preskočenie. Táto vlajka je zámerným únikovým východom; `auto` (predvolené nastavenie)
  nikdy nespracuje záznam, ktorý neobsahuje surové dáta.

**Kalibračné značky: iba pri `raw`.** Súbor `raw` zo záznamu obsahuje kalibračný paket kamery: užitočné dáta `CalibrationJSON` a značky kompatibilné s MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), ktoré opisujú surové hodnoty senzora. Všetky ostatné súbory na jednotlivých úrovniach (`debayered`, `display`, `radiance`, `reflectance`, render `index/`) a každý rámec `lattice continuous` / `multi-stream` nesie identitu kamery (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) a vlastný záznam snímky, `XMP-mapir:FrameJSON` (expozícia, zosilnenie, teplota, formát pixelov, čas zachytenia, zhoda DAQ), keďže každý produkt `process` nesie túto identitu. `process` vyhľadá kalibráciu takéhoto súboru podľa jeho sériového čísla a `CalBundleSha256` v kalibračnej cache programu Chloros, pričom balík stiahne len raz, ak ho tento počítači chýba.

#### Súbory vytvorené z viacerých kamier

Súbor vytvorený z pásiem viacerých kamier — multibandový stack (`align-apply`, `align-stream`, `index --live --save-multiband`), živý index vytvorený z viacerých kamier (`index.tif`, `index_LUT.png`), obraz z hromadného snímania `composite/` — uvádza názvy všetkých kamier, z ktorých pochádza:

- **Zoznamy podľa jednotlivých kamier, v poradí pásiem** (kompozit: vrstva pozadia, potom popredie; indexová vrstva, kamery, z ktorých pochádzajú jej pásma): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (kalibračný balík, ktorým prešli pásma každej kamery, inak — súbor `align-apply` / `align-stream`, ktorého pásma neprešli žiadnym — balík kamery Chloros, ktorý si uložila do vyrovnávacej pamäte, ten, podľa ktorého sú pomenované jej súbory so záznamom), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — paralelné zoznamy, pričom položka *n* v každom z nich popisuje tú istú kameru; `-`, kde kamera nemá žiadnu hodnotu (žiadny kalibračný balík, neznámy objektív). Každý fotoaparát predstavuje **jednu** položku: ak má jeho balík pre dané pole viacero hodnôt — napríklad certifikáty halogénových žiaroviek v balíku — sú spojené pomocou `;` (`F-1479-1;F-1480-1;F-1481-1`), nikdy čiarkou, takže zoznam obsahuje jednu položku na jednu kameru bez ohľadu na to, ako je vytlačený.
- **EXIF `SerialNumber`** obsahuje rovnaké sériové čísla, oddelené čiarkami (`214000535,214000551`).
- Neobsahuje **žiadnu** identifikáciu konkrétnej kamery (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): tie identifikujú jednu kameru a jej sadu, a `process` by na ich základe určil kalibráciu súboru. Index vypočítaný z pásiem viacerých kamier (`index.tif`, `index_LUT.png`) obsahuje v EXIF údaj `Model` `LATTICE` (súborné zariadenie), nie model `LATT-…` jednej kamery.

Súbor s pásmami **jednej** kamery — buď TIFF súbor (`--per-band`), snímka (`align-apply --per-camera`), zoskupenie alebo živý index jednej kamery — obsahuje identifikačné značky tejto kamery, rovnako ako každý iný súbor so záznamom. Súbory stohu, súbory pre jednotlivé pásma a súbory pre jednotlivé kamery s označením `align-apply`, ako aj každý stoh s označením `align-stream` a jeho súbory pre jednotlivé pásma obsahujú čas zachytenia svojej skupiny (expozícia referenčnej kamery, UTC), a snímka `--per-camera` z monokamery je jedna rovina, podobne ako stoh v `--bit-depth`.**Orientácia kamery (zrkadlenie / otočenie).** Nastavenie orientácie kamery (funkcie Mirror H /
Mirror V / rotácia) v grafickom rozhraní určuje, čo sa pri snímaní uloží:

- **`raw` a `debayered` sa uložia presne tak, ako ich snímač prečítal**, bez ohľadu na nastavenie.
  Otočená alebo prevrátená Bayerova mozaika by predstavovala odlišný vzor farebných filtrov a kalibračné
  mapy (dark offset, flat field) sú na vlastnej mriežke snímača.
- **`preview`/`display`, `radiance`, `reflectance` a render `index/` sú orientované** — otočené
  raz, až po vykonaní všetkých kalibračných výpočtov, takže sa otvárajú tak, ako bol fotoaparát nasmerovaný. Ich
  `clamp_masks/…_clampmask.tif` sú na tej istej mriežke.
- **Každý súbor záberu zaznamenáva nastavenie** v XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, v smere hodinových ručičiek, po
  otočeniach) a `ChlorosOrientationApplied` — `false` na surových a debayeroch (natívnych pixeloch),
  `true` na orientovaných výstupoch. `ChlorosProcessing` uvádza operácie aplikované na
  pixely daného súboru (`none` v prípade natívnych). To zahŕňa každý snímok, ktorý prenáša `lattice continuous` (
  vlastné nastavenie kamery, neuplatnené) a každý snímok z `lattice multi-stream` (identita: jeho
  snímky sú natívne, podobne ako pri snímaní poľa). `lattice align-stream` / `align-apply`
  stohy zaznamenávajú nastavenie hlavnej kamery, neaplikované: ich pásma sú deformované do snímky hlavnej kamery,
  ale nie sú otočené.
- **`process` uplatňuje zaznamenanú orientáciu** pri spracovaní surových údajov: najprv tmavý posun, vyrovnanie
  poľa, demosaik a oddelenie farieb v súradniciach snímača, potom deformácia zarovnania poľa a
  orientácia ako posledná — takže produkty náhľadu, radiancie, odrazivosti a indexu vychádzajú presne tak, ako
  pri živom snímaní, a zaznamenávajú sa do `ChlorosOrientationApplied=true`. Nekalibrované záložné
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) sú orientované rovnakým spôsobom, po
  vyrovnania poľa. Produkt po odstránení bayera zostáva v pôvodnom formáte.
- **Prehliadač obrázkov Chloros zobrazuje surový alebo debayered súbor otočený podľa jeho záznamu** (len na zobrazenie —
  súbor sa nemení), takže vyzerá ako jeho produkty; miniatúra, zobrazenie v plnej veľkosti,
  odčítanie kurzora a sandbox Index/LUT používajú všetky rovnaké otočenie. Súbor bez záznamu a
  produkt (už orientovaný) sa zobrazujú tak, ako sú uložené. Export do sandboxu zaznamenáva orientáciu, ktorú
  majú jeho pixely (`ChlorosOrientationApplied=true`; identifikátor pre vrstvu bez záznamu).
- **Záznamy bez záznamu sa spracúvajú ako natívne** (nezmenené správanie). Pred verziou 1.3.0
  záznam z jednej kamery s nastaveným prevrátením alebo otočením sám otočil svoje surové a debayered súbory;
  program Chloros nedokáže rozoznať takýto užotočený surový súbor od natívneho a nerobí žiadne odhady, preto ho
  spracováva tak, ako bol uložený. Kalibračné mapy sú na vlastnej mriežke senzora, takže pre takýto
  záznam použite záznam zhotovený verziou 1.3.0, ak je dôležitá jeho kalibrácia.
- **Pole štandardne neotáčajú žiadny súbor člena** (deformácia zarovnania a XMP súbor `Alignment*` sú v
  vlastných, neotočených snímkach kamier) a zaznamenávajú identitu. `array-capture --orient` otočí produkty každého
  člena po transformácii zarovnania: podľa orientácie **hlavného snímku** v čase, keď boli
  transformované do jeho snímku (aby členy zostali vzájomne zaregistrované), inak podľa orientácie ich vlastnej kamery. Každý
  súbor zaznamenáva orientáciu rámca, v ktorom sa nachádzajú jeho vlastné pixely — surový údaj, orientáciu vlastnej kamery —
  a surový údaj, ktorý obsahuje transformáciu zarovnania, zaznamenáva tiež `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: hlavného snímku, pre výstup deformovaný
  do rámca hlavného záberu. `process` otočí každý produkt o hodnotu, ktorá patrí k jeho pixelom:
  o zarovnanú hodnotu, ak deformuje surový záber (`--array-alignment`, predvolené nastavenie — produkty zo živého zarovnaného
  zachytenia), vlastný záber kamery, ak sa nevyrovnáva (`--no-array-alignment` – zo živého
  nezarovnaného záznamu). Rovnaká voľba existuje pre `array-burst`, `project capture` / `burst` /
  `stream`, recepty a SDK.

### Preskočené zábery v poliach so zmiešanými filtrami

Ak v jednom poli kombinujete RGB a multispektrálne kamery, `array-capture --processing radiance` (alebo `reflectance`) uloží multispektrálne snímky a **preskočí** RGB-kamery — žiarivosť na jeden Bayerov filter nemá pre širokopásmový senzor význam. CLI výslovne vypíše každý uložený súbor (s úrovňou exportu), každý zapísaný súbor `.daq` a každé vynechanie, takže počet súborov nie je prekvapujúci:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Tokeny dôvodov preskočenia majú formát `<level>-not-applicable-to-rgb-cam`. Kamera, ktorej kalibračný balík nie je načítaný, preskočí `radiance` / `reflectance` s `<level>-needs-calibration (connect the camera so its cal pack loads)`, a ak nie je k dispozícii použiteľná hodnota odrazivosti zo senzora DAQ, preskočí sa s kódom `reflectance-skipped-no-dls`. Odrazivosť sa môže preskočiť aj s kódmi `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, a s `dls-uncalibrated-band-<nm>`, ak sa pásmo nachádza prevažne mimo radiometricky kalibrovaného rozsahu svetelného senzora DAQ (~374–974 nm) — spomedzi dodávaných SKU iba F988, ktorého podporovanou cestou je pracovný postup s panelom odrazivosti.

Použite `--processing debayered` (alebo `display`), ak chcete zahrnúť všetky kamery bez ohľadu na typ filtra, alebo predvolený `all`, aby ste získali všetky príslušné úrovne pre každú kameru naraz.

### Hodnota DAQ vzdialená od expozície

Každá hodnota `.daq`, ktorú zachytí snímka, je hodnota DAQ zodpovedajúca danej expozícii, a hodnota `.match.json` vedľa nej zaznamenáva, aká veľká bola vzdialenosť medzi týmito dvoma hodnotami (`gap_ms`) a či sa táto vzdialenosť nachádza v rámci tolerancie (`in_tolerance`, 500 ms, pokiaľ nie je v nastavení snímania uvedené inak `--daq-tolerance-ms`). Hodnota mimo tohto rozsahu sa aj tak použije, takže nedochádza k strate žiadneho snímku, a je označená: súbor XMP súboru odrazivosti obsahuje `DlsInTolerance=False` a `DlsMatchGapS`. `lattice capture`, `multi-capture` a `array-capture` to tiež vytlačia pod príslušným `.daq`, pričom sa to pri vysvetľovaní javí ako chyba prúdu DAQ'pri vysvetľovaní:

**Ako sa hodnota nameranej hodnoty priraďuje k expozícii.** Zachytenie čaká na prvú nameranú hodnotu z DAQ po expozícii, aby mohlo zladiť namerané hodnoty na oboch stranách tejto hodnoty (`method` `interpolated`). DAQ-E odosiela hodnotu každých 0,4 s za denného svetla a každých 1–2 s v interiéri, takže čakanie predlžuje príkaz o približne jeden a pol takého intervalu, nie expozíciu. Na DAQ, ktorý prestal odosielať, sa nečaká. Hodnota DAQ-E je priemerná hodnota osvetlenia za interval od predchádzajúceho merania, takže je umiestnená uprostred tohto intervalu: `gap_ms` sa meria od expozície po tento stred a `bracket_stamps_ns` udáva vlastné časové značky meraní. `array-capture --continuous` čaká bez spomalenia snímania: jeho surové súbory sa zapisujú ihneď a ich `.daq` sa zapíše hneď po príchode hodnoty. S `--interval` každý prechod čaká ako jedno zachytenie.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Ak sa z daného odčítania nezapísal žiadny súbor odrazivosti, prvý riadok namiesto toho končí `saved anyway; its .match.json says in_tolerance: false`.

Záznam aj tak končí kódom `0`: všetky súbory boli zapísané a uvádzajú, čo obsahujú. DAQ, ktorého tok sa zastavil, naďalej poskytuje svoje posledné meranie, preto ho skontrolujte pomocou kódu `chloros-cli daq pool-list`, skôr než sa spoliehate na odrazivosť tohto záznamu. Backend opätovne pripojí zoskupené zariadenie DAQ-E, ktoré prestalo odosielať údaje (pozri [DAQ-E, ktorý prestane dodávať údaje](#a-daq-e-that-stops-delivering)). Hodnota mimo tolerancie vytlačí len svoj riadok `DAQ:`, pokiaľ nejde o uchovanú hodnotu (pozri nižšie).

**Hodnota uchovaná po vypadnutí príslušného DAQ.** Keď žiadny DAQ nemôže poskytnúť hodnotu pre danú expozíciu (priradený DAQ nie je k dispozícii alebo žiadny DAQ zo združeného poola nemá hodnotu, napr. po `daq pool-disconnect`), zachytenie opätovne použije poslednú hodnotu, ktorú mal Chloros, a jeho `.match.json` opisuje túto hodnotu vo vzťahu k tejto expozícii: `method`, `retained`, `gap_ms` z tejto expozície na vzorky, z ktorých bola hodnota nameraná, `in_tolerance` z tej medzery a `retained_from_epoch_ns`, expozícia, ku ktorej bola hodnota pôvodne priradená. Zachytenie vykonané dlho po vypnutí DAQ preto vytlačí riadky uvedené vyššie a pod je ešte jedna, ktorá sa vytlačí aj vtedy, keď je nameraná hodnota stále v tolerancii:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

Jeho `.daq` stále uvádza názov DAQ, ktorý nameral hodnotu, spolu s kalibráciou a krytom, pod ktorými bola táto hodnota nameraná, takže pri opätovnom importe je zrejmé, ktorý DAQ, kalibrácia a kryt ju vyprodukovali.

### Súbory, ktoré sa nepodarilo zapísať

Kamera, ktorej súbor sa nedá zapísať – preplnený disk, chyba oprávnení – nezabráni ukladaniu súborov ostatných kamier. Každý príkaz na zachytenie (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; a recepty `capture`, `burst` a `stream`) vypíše každý súbor, ktorého uloženie zlyhalo, do stderr, potom **ukončí činnosť `1`**, hoci zvyšok sa uložil:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Zahrnuté sú všetky súbory, ktoré má zachytenie obsahovať, vrátane jedinéhosnímka s odstráneným bayeringom `lattice capture --levels debayered`. Záznam `.daq`, ktorého sidecar s informáciami o pôvode `.match.json` sa nepodarilo zapísať, je uvedený ako `[match]` (samotný súbor `.daq` bol uložený).

Na plnom disku sa opakujúce sa záznamy (`array-capture --continuous` / `--interval`, `project burst` / `stream`, recept `burst` / `stream`) **zastaví sa v tomto kroku** namiesto toho, aby počas zvyšku behu na ten istý disk nezapisoval nič. Akákoľvek iná chyba sa hlási v každom kroku, v ktorom k nej dôjde, a beh pokračuje až do konca, potom sa ukončí `1`. `array-capture --continuous` (nepretržité zachytávanie backendu) neuvádza žiadne súbory, vrátane tých, pri ktorých došlo k chybe: ich počet sa uvádza v riadkoch s priebehom a súhrnom (`N failed`, `N file(s) failed to write.`) a ukončí sa `1` — pozri [Neprerušovaný režim](#continuous-mode---continuous).

Záznamníky — `array-burst` a `array-record` — hlásia svoje vlastné chyby. `array-burst` sa sám zastaví, keď sa disk zaplní, alebo tesne predtým. Oba pokračujú aj napriek akémukoľvek inému zlyhaniu zápisu. Keď niektorý z nich nahlási chybu, tá sa vypíše za počítadlami a príkaz **ukončí `1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` namiesto `Disk full:` znamená, že sériový záznam sa zastavil skôr, ako sa disk disk. Rámce zaznamenané pred zastavením zostanú zachované. `array-burst --build` preskočí kompiláciu, keď sa vyčerpá miesto, pretože videá by sa ukladali na ten istý disk. Najprv uvoľnite miesto, potom spustite `array-build-video --burst-dir …`. Po akejkoľvek inej chybe kompilácia stále beží.

Chyba `array-record`, pri ktorej sa nezaznamenal žiadny snímok, tiež znamená zlyhanie nahrávania (`Recorder error: no-frames: the combined view published no frame to record …`, ukončite `1`) — pozri [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Režimy zachytávania, rekordéry a offline spracovanie

Všetky tieto funkcie pracujú s **trvalým poľom** (najprv spustite `array-connect`). Zrkadlia panel zachytávania v grafickom rozhraní.

### Režimy `array-capture`

`array-capture` je jediný príkaz so štyrmi režimami uzávierky a súborom prepínačov pre export:

| Režim | Príznak | Správanie |
| --- | --- | --- |
| **Jednorazový** *(predvolené)* | (žiadny) | Jedna synchronizovaná skupina snímok, potom ukončenie. |
| **Neprerušované** | `--continuous` | Neprerušované snímanie až do zachytenia `Ctrl+C`, `--count N`, `--duration S` alebo naplnenia disku. Spustí vlastné nepretržité zachytávanie backendu (režim „Continuous“ v grafickom rozhraní) — pozri [Režim Continuous](#continuous-mode---continuous) nižšie. |
| **Interval** | `--interval S` | Jedno prechádzanie každých `S` sekúnd (merané od začiatku každého prechádzania), rovnaké hranice. |
| **Najrýchlejší** | `--fastest` | Iba surové+ priradená hodnota z DAQ + kompozit kombinovaného indexu; preskakuje výpočty žiarivosti/odrazivosti/zobrazenia, aby sa snímka načítala rýchlo. Implikuje `--processing raw --force-daq`. Uložené `.daq` neskôr spracujte na kalibrované produkty. |

Prepínače exportu (možno kombinovať s ľubovoľným režimom; všetky zdieľajú koncový bod GUI/SDK):

| Priepínač | Účinok |
| --- | --- |
| `--array-id ID` | Pole, z ktorého sa má zachytiť (vynechajte, ak je pripojené len jedno). |
| `-o, --output DIR` | Výstupná zložka (predvolene `output`, v zložke, z ktorej spúšťate príkaz). |
| `--format {tiff,tif,png,jpg,jpeg}` | Formát súboru (predvolené nastavenie je `tiff`). Formáty PNG / JPEG podporujú iba `display` a indexové vykresľovanie; pozri [Úrovne exportu zachytenia](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Jedna úroveň exportu (`raw`, `debayered`, `radiance`, `reflectance`, `display`) alebo `all` (predvolené). |
| `--levels L1,L2,…` | Explicitná podmnožina typov exportu (napr. `raw,radiance,reflectance`); **prepíše `--processing`**. |
| `--aligned` / `--no-aligned` | **Predvolené nastavenie: zarovnané, tak ako to robí aplikácia Chloros pri zachytávaní toho istého poľa**(jeho nastavenie zachytávania**Zarovnané** pre každé pole, zapnuté, pokiaľ ho nevypnete). Ne-surový export každého člena (`debayered`, `display`, `radiance`, `reflectance`, vykreslenie indexu) je deformovaný do rámca hlavnej kamery a orezaný na spoločný prekrývajúci sa úsek členov, takže súbory sú pixelovo zarovnané a majú rovnakú veľkosť, orezanie a tagy `Alignment*` ako v aplikácii — menšie ako senzor (alebo oblasti záujmu – ROI) o posuny členov; v rámci oblasti záujmu (ROI) sa transformácia prepočítava na okno. `XMP-mapir:AlignmentJSON` uvádza `"applied": true` a poskytuje `crop_roi`. Surový súbor zostáva bez deformácie v pôvodnej veľkosti, ale transformáciu nesie v metadátach (`"applied": false`). `--no-aligned` ukladá každý súbor v pôvodnej veľkosti, bez deformácie. `--aligned` je predvolené nastavenie. Nezávisle od toho, či je deformácia aplikovaná alebo nie, každý súbor v súbore zaznamenáva transformáciu svojej kamery `Alignment*`, akonáhle má pole profil zarovnania — s výnimkou kamery, ktorú profil nevyrovná (vynechaná alebo vynútene nastavená na identitu v dôsledku neúspešného riešenia), a referenčnej kamery, ak nie je vyrovnaná žiadna iná kamera: tie nemajú žiadne značky `Alignment*`. Ak pole nemá profil, záznam sa uloží nevyrovnaný a CLI vypíše dôvod na riadku `Alignment:`, rovnako ako v každom inom prípade, keď nebolo možné použiť deformáciu (členy s rôznymi rozlíšeniami, pole bez synchronizácie káblov, snímka, na ktorú sa profil nedá zmapovať). Automatické zarovnanie poľa vytvorí profil hneď, ako sa ustáli expozícia; `align-calibrate` vytvorí zarovnanie na požiadanie. |
| `--orient` | Premeňte **výstupy**každého člena — `display`, `radiance`, `reflectance`, render `index/` a `composite/` — podľa orientácie kamery po deformácii zarovnania: produkt deformovaný do rámca hlavného snímku (predvolené nastavenie) podľa**orientácie vzorového snímku**, takže členy zostávajú zarovnané; nezarovnaný (`--no-aligned` alebo záznam, na ktorý nebolo možné aplikovať deformáciu) podľa orientácie vlastnej kamery.**Predvolené nastavenie: vypnuté — každý súbor člena zostáva v pôvodnom formáte**, ako je to vždy v prípade záznamov v podobe poľa. `raw` a `debayered` zostávajú v pôvodnom formáte v oboch prípadoch. Každý súbor zaznamenáva orientáciu, ktorú majú jeho produkty (pozri [Orientácia kamery](#ako-vyzerá-zložka-so-zachytenými-dátami)); súbor XMP `Alignment*` zostáva transformáciou natívneho rámca, aplikovanou pred orientáciou, takže `process` (deformácia, potom orientácia) reprodukuje živý výstup. |
| `--index` / `--no-index` | Uložiť / preskočiť prekrývací obrazec s indexom vegetácie pre jednotlivé kamery, ak je nakonfigurovaný. Predvolené nastavenie: vykresliť ho. |
| `--no-daq` | Preskočiť súbor `.daq`, ak žiadna zvolená úroveň nepoužíva DAQ (zachytávanie iba surových dát). Predvolene sa hodnota uloží, takže snímky je možné offline spracovať na odrazivosť/index. `--force-daq` je stále akceptované (predvolené). |
| `--daq-tolerance-ms MS` | Tolerancia zhody DAQ pre tieto záznamy (predvolene 500 ms). |
| `--smart` | Pred spustením počkať, kým sa AE ustáli na všetkých kamerách (pozri [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Kompresia pixelov vo formáte TIFF. `none` (predvolené od verzie 1.3.0; verzia 1.2.0 zapisovala `deflate`) = nekomprimované — pixely snímača presne tak, ako boli načítané, ~6,3 MB na snímku v plnom rozlíšení a najrýchlejšia trvalá rýchlosť; `deflate` = bezstratová kompresia zlib L1 + horizontálny prediktor, ~4,1 MB na snímku, pomalšie. Obe sú bezstratové a pri importe sa načítajú identicky. |

> **TIFF s jednorazovým zápisom + model s trvalou rýchlosťou.**Každý súbor sa zapíše**raz**: pixely, balík XMP, IFD0 Make/Model/CameraSerialNumber a EXIF tagy `ExposureTime` / `SerialNumber` v jednom prechode, pričom sa nič následne neprepisuje. `--continuous`X zachytáva po sebe idúce synchronizované skupiny — každý záznam preberá skupinu po predchádzajúcej, vrátane tej, ktorá bola zverejnená ešte počas ukladania predchádzajúceho záznamu — takže beží rýchlosťou poľa. DEFLATE drží GIL v Pythone, takže komprimované zápisy sa**ne**paralelizujú medzi vláknami zapisovača pre jednotlivé kamery; nepretržité snímanie v plnom rozlíšení pri rýchlosti senzora vyžaduje nekomprimované zápisy (predvolené nastavenie)**a** disk triedy NVMe (~600 MB/s trvalo pre pole s 9 kamerami pri ~10 fps). Ten istý ovládač je dostupný ako `compression` na `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Režim nepretržitého snímania (`--continuous`)

`array-capture --continuous` (bez `--interval` alebo `--smart`) odovzdáva úlohu **vlastnému nepretržitému záznamu backendu** — relácii, v ktorej beží režim Continuous v grafickom rozhraní. Backend ukladá každú synchronizovanú skupinu samostatne, pričom sú v procese dve snímky a medzi skupinami nie je žiadna požiadavka z CLI, takže drží krok s rýchlym poľom, kde požiadavka na jednu snímku zaostáva (nameraná rýchlosť 73–77 snímok/s na poli s 9 kamerami pri oblasti záujmu (ROI) s rozlíšením 512 pixelov). Každé prepnutie exportu uvedené vyššie sa vzťahuje na každý záznam.

- **Obmedzenia:** `--count N` sa zastaví po N záznamoch (jeden záznam = jedna synchronizovaná skupina: súbory každého člena); `--duration S` po S sekundách. Bez týchto nastavení beží program až do `Ctrl+C`, zaplnenia disku, chyby alebo odpojenia sústavy. `Ctrl+C` zastaví reláciu backendu, potom vypíše súhrn.
- **Priebeh** každých 2 s, s pridaním `, N failed` a `, N groups skipped`, ak sú odlišné od nuly (preskočená skupina je taká, ktorú pole zverejnilo, ale nebola zachytená):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Žiadny zoznam podľa súborov.** Spustenie končí jedným súhrnným riadkom — zachytenia, rýchlosť, súbory a výstupná zložka — nasledovaným kódmi `N file(s) failed to write.` / `N group(s) skipped.`, ak sú odlišné od nuly:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Zastaví sa** pri zaplnenom disku s kódom `Continuous capture stopped: the drive is full.` a pri chybe alebo strate poľa s kódom `Continuous capture stopped: <reason>.` (oba na stderr).
- **Stav ukončenia:** `0`, ak sa zachytávania uložili a všetky súbory boli zapísané, vrátane `Ctrl+C`; `1` v prípade akéhokoľvek súboru, ktorého sa nepodarilo zapísať, pri zaplnenom disku, chybe, odpojení poľa alebo ak sa nezaznamenalo vôbec nič.
- **Microsoft Defender:** keď Defender využíva jedno jadro procesora alebo viac na kontrolu súborov, ktoré zaznamenáva program, spustenie raz vypíše na stderr: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Pozrite si [Nastavenia systému](#windows-exe).
- Naraz je možné vykonať len jeden nepretržitý záznam na jedno pole; druhý je odmietnutý s chybou `a continuous capture is already running for this array`. Prostredníctvom HTTP je relácia `POST /api/camera/array/continuous/start` (telo `/api/camera/array/capture` plus `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` a `POST /api/camera/array/continuous/stop`.

**Interval**(`--interval S`, s alebo bez `--continuous`) a**`--continuous --smart`** zachovávajú cyklus na požiadavku: jedna požiadavka na zachytenie na jeden prechod — v každom naplánovanom okamihu pre Interval, po ustálení AE pre `--smart` — so zoznamom súborov každého prechodu ako v režime Single, riadkom `Done — N capture pass(es).` na konci a počítaním prechodov pomocou `--count N`.

### `array-record` — kombinovaný index videa/GIF (na účely monitorovania)

Zaznamenáva **živý kombinovaný indexový pohľad** na pole do súboru `.avi` (a voliteľne do súboru `.gif`). Ak je otvorený kombinovaný pohľad v grafickom rozhraní, zaznamenáva tento pohľad tak, ako sa zobrazuje. Ak nie je otvorené žiadne okno, backend sám vykreslí zobrazenie: index, ktorý zadáte pomocou `--expr`, alebo štandardne NDVI poľa, akonáhle je pole zarovnané. Predvolené NDVI používa červené a NIR pásma najbližšie k 660 nm a 850 nm, rovnako ako živý pohľad. Pohľad v grafickom rozhraní otvorený počas nahrávania sa zaznamenáva, pokiaľ zostáva otvorený. Každé 2 s kontroluje priebeh a zastaví sa pri `--duration`, `Ctrl+C`, alebo keď sa záznamník sám ukončí.

Záznam, pri ktorom sa nezapíše žiadny snímok, zlyhá: záznamník nahlási `no-frames: …` (pole sa môže stále zarovnávať alebo stabilizovať expozíciu) a príkaz **sa ukončí `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Príznak | Predvolené | Popis |
| --- | --- | --- |
| `--array-id ID` | len pole | Cieľové pole (vynechajte, ak je pripojené len jedno). |
| `-o, --output DIR` | `output` | Výstupný adresár. Relatívna cesta sa nachádza v zložke, z ktorej spúšťate príkaz. |
| `--fps F` | `10` | Rýchlosť záznamu snímok. |
| `--duration S` | až do stlačenia Ctrl+C | Automatické zastavenie po `S` sekundách. |
| `--gif` | vypnuté | Zapísať aj animovaný súbor GIF. |
| `--gif-only` | vypnuté | Uložiť len GIF (bez `.avi`). |
| `--expr EXPR` | NDVI poľa | Indexový výraz, ktorý sa má zaznamenať, ak nie je otvorené žiadne okno GUI, v názvoch pásiem poľa (napr. `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; pozri [názvy pásiem](#index--vegetation-maths)). Krátky názov farby (`NIR`, `Red`) funguje len vtedy, ak ho má aspoň jedno pásmo poľa. Ak je otvorené okno GUI, zaznamená sa namiesto toho zobrazenie, ako sa zobrazuje. |

### `array-burst` — séria snímok v surovom formáte Bayer s vysokou snímkovou frekvenciou (na analytické účely)

Priamo číta vyrovnávaciu pamäť synchronizovanej skupiny zo slučky snímok — **nie je potrebný kalibračný reťazec, exiftool ani živý náhľad** — takže beží pri plnej rýchlosti snímania fotoaparátu. Zapíše surové snímky + manifest pre každý snímok + jeden súbor `.daq` pre každé odlišné čítanie DLS pod `<output>/bursts/<base>/`. Spracujte offline (ďalší príkaz) alebo zadajte `--build`, aby sa to vykonalo ihneď po zastavení.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Príznak | Predvolené | Popis |
| --- | --- | --- |
| `--array-id ID` | iba pole | Cieľové pole. |
| `-o, --output DIR` | `output` | Výstupný adresár (výstup sa ukladá do `<DIR>/bursts/<base>/`). |
| `--duration S` | až do stlačenia Ctrl+C | Automatické zastavenie po `S` sekundách. |
| `--max-frames N` | neobmedzené | Automatickézastavenie po `N` surových snímok. |
| `--orient` | vypnuté | Výstupy, ktoré zostavenie vytvorí z tejto série, sú otočené podľa orientácie kamery — výstupy jednotlivých kamier podľa orientácie ich vlastnej kamery, kombinovaný index podľa orientácie hlavnej kamery — tak, ako to robí `array-capture --orient`. Surové snímky zostávajú v pôvodnom formáte; `manifest.json` tejto série zaznamenáva orientáciu každého člena a zvolenú možnosť. |
| `--build` | vypnuté | Po zastavení okamžite znovu spracujte sériu (rovnako ako `array-build-video`). |
| `--products …` | `combined:index` | S `--build`: ktoré video(-á) sa má(-jú) vytvoriť (pozri nižšie). |
| `--fps F` | `10` | Pri použití `--build`: počet snímok za sekundu (fps) výstupného videa. |
| `--save-tiffs` | vypnuté | S `--build`: ukladať aj kalibrované súbory TIFF pre každý snímok (každý so svojou [maskou obmedzenia unmix](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on), ak bol pixel obmedzený; výsledok kompilácie ich uvádza ako `clamp_masks`). |
| `--gif` | vypnuté | S `--build`: zapisuje aj animované súbory GIF. |

### `array-build-video` — offline spracovanie uloženého súboru snímok

Časovo zosúladí každý surový snímok s najbližšou uloženou hodnotou `.daq` a prejde ho cez **rovnaký reťazec žiarivosti / odrazivosti / indexu ako importný proces**, čím vytvorí jedno alebo viac videí.

`--products` je zoznam položiek `kind:level` oddelených čiarkami, kde `kind` ∈ `per_cam` | `combined` a `level` ∈ `radiance` | `reflectance` | `index`. Samotný `level` (bez `kind:`) má predvolenú hodnotu `per_cam`. Predvolená hodnota je `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Prieznak | Predvolené | Popis |
| --- | --- | --- |
| `--burst-dir DIR` | (povinné) | Cesta k priečinku burst (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Zoznam `kind:level`, ako je uvedené vyššie. |
| `--fps F` | `10` | Počet snímok za sekundu (fps) výstupného videa. |
| `--save-tiffs` | vypnuté | Uložiť spolu s videom (videami) aj kalibrované súbory TIFF pre každý snímok. |
| `--gif` | vypnuté | Zapísať aj animované súbory GIF. |

**Video neobsahuje žiadnu masku obmedzenia** (jeden obrazový tok, 8-bitový). Namiesto toho výsledok kompilácie obsahuje `clamp_fractions`: pre každý video produkt, podiel pixelov každého snímku, ktoré boli pri oddeľovaní obmedzené v pásme, ktoré obsahuje jeho kanál R / G / B — produkt žiarivosti/odrazivosti daného snímku, ktorý by bol označený v `Chloros:Band{R,G,B}UnmixClampFraction` (pre kombinovaný index, pre každú zúčastnenú kameru). Spustenie vypíše najhorší snímok pre každý produkt na riadku `Clamp:`. `--save-tiffs` pridáva masky na úrovni pixelov pre súbory TIFF, v súbore `clamp_masks/<level>/` vedľa zložky `<level>/` pre súbory TIFF.**Predčasne ukončená kompilácia nezanecháva poškodené video.** Každé video sa zapisuje ako `<name>.partial.avi` a konečný názov dostane až po dokončení, takže staršie video s týmto názvom zostane zachované, kým nebude jeho náhrada kompletná. Spustite kompiláciu znovu, aby ste nahradili zostávajúce video `.partial.avi`.

Spustenie vypíše každé video, ktoré zapísalo, potom akýkoľvek produkt `Skipped:` (požadovaná kamera/úroveň – alebo kombinovaný index – ktorý nevytvoril žiadne video, spolu s dôvodom – napr. chýbajúci kalibračný balík, chýbajúce DLS meranie odrazivosti alebo chýbajúca dvojica pásiem červenej a NIR pre kombinovaný index) a akýkoľvek produkt typu `Note:`. RGB kamera, pre ktorú bola požiadaná žiarivosť alebo odrazivosť, dostane namiesto toho svoje zobrazené video, s vyvážením bielej, ak je k dispozícii kalibračný balík kamery; kamera, ktorá vykresľuje bez neho, je uvedená v `Note:`. Vráti **nenulovú hodnotu**, ak bol požadovaný produkt preskočený alebo nebol vytvorený podľa požiadavky (konfigurovaný indexový výraz sa nevyhodnocuje offline); videá, ktoré boli vytvorené, sa aj tak zapíšu a uvedú v zozname.

> **Vyberte správny záznamník.** `array-record` je určený na *monitorovanie* — zaznamenáva renderovaný živý kompozitný obraz: kombinovaný pohľad z grafického rozhrania alebo, ak nie je otvorené žiadne grafické rozhranie, vlastný pohľad backendu NDVI (alebo `--expr`). `array-burst` → `array-build-video` je určený na *analýzu* — ukladá surové údaje zo senzorov pri plnej rýchlosti a následne rekonštruuje kalibrované videá s jasom/odrazivosťou/indexom, pričom nie je potrebný živý náhľad.

### Jednopásmové kamery Mono (M3M)

Rad **M3M**je monochromatickou verziou modelu Bayer**M3C**: jedna úzkopásmová interferenčná filtračná mriežka na kameru (`M3M-<lens>-F<wavelength>`, napr. `M3M-L87-F685`), takže senzor poskytuje**jedno pásmo v odtieňoch šedej** bez Bayerovej mozaiky. Nie je potrebné vykonávať demosaikáciu, odstraňovať presluchy medzi kanálmi ani nastavovať vyváženie bielej – celý farebný reťazec pre zobrazenie v RGB sa jednoducho neuplatňuje.

Čo to znamená v CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**rozpoznajú monochromatickú kameru a**preskočia ju s jednoriadkovou správou** namiesto toho, aby posielali bezvýznamné nastavenia. V tej istej relácii stále fungujú normálne s kamerou typu RGB/Bayer M3C.
- **Formát pixelov Bayer určuje ich hĺbku.** M3M ponúka iba formáty Mono, takže formát Bayer z `--pixel-format` alebo `--preset` nastaví formát Mono s rovnakou hĺbkou: `BayerRG8` a `--preset high_speed` dávajú `Mono8`; `BayerRG12`, `high_quality` a `triggered` dávajú `Mono12`; zbalený názov zostáva zbalený (`BayerRG12p` dáva `Mono12p`). Súbory a názov `lattice format` označujú formát Mono. Pole, ktoré kombinuje kamery M3C a M3M, používa jeden formát rovnakým spôsobom: Bayer na každom M3C, jeho hĺbka Mono na každom M3M. Názov zodpovedá jeho bitovej hĺbke: to isté platí aj naopak, takže M3C sa nikdy nezapíše do formátu Mono (`--pixel-format Mono12` je `BayerRG12` na M3C, bez ohľadu na to, ktorá kamera je hlavnou kamerou poľa), a `lattice format --set` sa mapuje rovnakým spôsobom a vytlačí formát, ktorý nastavil.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** stále fungujú — žiarivosť a odrazivosť sú radiometrické mapy *na jedno pásmo* a sú pre jedno pásmo dokonale definované. Mono snímky obsahujú **identitnú** maticu odozvy senzora (bez 3×3 rozloženia), takže rovina prechádza kalibračnými výpočtami bez zmien.
- **Jedna monochromatická kamera nedokáže vypočítať vegetačný index.**NDVI, NDRE a podobné indexy vyžadujú aspoň dve pásma (napr. Red + NIR). Ak chcete získať index z monochromatického hardvéru, nasmerujte**niekoľko** kamier M3M na rôzne vlnové dĺžky, zosúladte ich do jedného viacpásmového súboru a vypočítajte index *z neho*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` otvára kamery sám (ako [dočasnú reláciu](#multi-camera-transient-sessions)), takže najprv nič nepripájajte: `--serials`, inak sa aktivujú kamery v profile (pozri [Index / Vegetation Maths](#index--vegetation-maths)). Pokiaľ Chloros drží akúkoľvek kameru (GUI, `cam-connect`, `array-connect`) je žiadosť odmietnutá s kódom HTTP 409 `CAMERA_HELD` a program sa ukončí `1`; uvoľnite ich pomocou `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

`--channel` symboly sa musia **presne** zhodovať s názvami kanálov v predvolbe (rozlišuje sa veľkosť písmen; NDVI sú v malých písmenách `red`, `nir` — pozri `--list-presets`), a názov strany pásma označuje pásmo v zarovnanom stohu (režim offline akceptuje aj indexy pásiem počítané od 0, napr. `--channel red=0 --channel nir=1`).

Rozlišovacím znakom v celom stohu je token `M3M` v reťazci modelu (nikdy sa neobjavuje v reťazci `M3C`), ktorý sa v grafickom rozhraní (GUI) a SDK-u zobrazuje ako `is_mono`.

---

## Nastavenie a ladenie sieťovej karty hostiteľa (súbory kamier LATTICE)

Kamery LATTICE prenášajú GVSP cez ethernetový adaptér hostiteľa, takže v prípade súborov s viacerými kamerami má **ovládač**adaptéra a**veľkosť prijímacieho kruhu** sú rovnako dôležité ako rýchlosť prenosu. Nesprávne nastavenia sa prejavujú ako `FRAMES WILL DROP` / `Reduce ROI to enable` v paneli Nastavenia poľa (a v `lattice network-analysis` / SDK `analyze_array_network()`), aj keď sú samotné kamery v poriadku.

### USB 10GbE adaptéry — Realtek RTL8157 („Realtek USB 10GbE Family Controller“)

Nižšie uvedené nastavenia sú vlastnosti ovládača pre systém Windows. V systéme Linux má USB adaptér tiež obmedzenie (400 MB/s na x86_64, 200 MB/s na NVIDIA Jetson) (pozri [Model fps a burstu poľa](#array-fps--burst-model)), a pre každý adaptér sú dôležité vyrovnávacie pamäte prijímania socketov jadra — pozri [Hostitelia Linuxu](#linux-hosts--socket-receive-buffers).

| Položka | Požadovaná hodnota | Prečo je to dôležité |
| --- | --- | --- |
| **Verzia ovládača**|**≥ v10.67 (január 2026)**, INF `rtump64x64sta.inf` | Starší ovládač z roku**2016**(v10.65, `rtump64x64.inf`) nesprávne spracováva vypnutie a chyby s kódom**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**pri vypnutí/reštarte/prechode do režimu spánku. Prechod sa zasekne (časový limit ~5 minút), používateľ vynúti vypnutie a opakované nečisté vypnutia**poškodia úložisko WMI**(PowerShell/nástroje začnú zlyhávať s chybou `Invalid class`) a**zablokujú USB stack** pri nasledujúcom spustení (sieťová karta sa neaktivuje; USB zariadenia prestanú byť rozpoznávané). Predtým, ako sa spoliehate na čisté reštarty, nainštalujte aktualizáciu z realtek.com (alebo od výrobcu hardvérového kľúča). |
| **Prijímacie vyrovnávacie pamäte**— kľúčové slovo `ReceiveBufferLen` |**256**(maximum ovládača) | Príjemný krúžok sieťovej karty. Predvolená hodnota ovládača**32**ponecháva len ~0,26 MB použiteľného priestoru v krúžku — čo je príliš málo na sériu dát z viacerých kamier — takže panel poľa hlási chybu `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` a blokuje pripojenie. Pri hodnote**256**je prstenec veľký (**~13,5 MB namerané na laboratórnom 10GbE hostiteľovi**), čo poskytuje prijímacej potrubnej linke skutočnú rezervu pre viac kamerové GVSP bursty. (O tom, či sa daná konfigurácia skutočne *pripojí*, rozhodujú dve kontroly — kontrola **priepustnosti s ohľadom na vyprázdňovanie**a kontrola**agregátneho prekročenia kapacity** — nie je to surové porovnanie burstu s krúžkom; pozri [Model fps a burstu poľa](#array-fps--burst-model).) |
| **Prijímacie URB**— kľúčové slovo `PendingReceives` |**64** (max.) | Bloky požiadaviek USB v prenose; zvyšujte spolu s prijímacími vyrovnávacími pamäťami na absorpciu burstov. |
| **Jumbo rámec** — kľúčové slovo `*JumboPacket` | **9014** | Potrebné pre pakety GVSP s veľkosťou 9000 bajtov (6× menej paketov na rámec ako pri 1500). |

> ⚠️ **Aktualizácia ovládača sieťovej karty RESETUJE tieto pokročilé vlastnosti na predvolené hodnoty.**Po aktualizácii alebo výmene ovládača adaptéra**znova nastavte** `ReceiveBufferLen=256` a `PendingReceives=64`, inak sa panel poľa opäť zablokuje, hoci sa „na hardvéri nič nezmenilo“. Toto je hlavná príčina toho, že zariadenie, ktoré predtým fungovalo, sa náhle odmietne pripojiť.

Aplikujte z **PowerShellu s oprávneniami správcu** (nahraďte názov svojho adaptéra, napr. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` sa vzťahuje na USB 10GbE adaptéry.** Teraz rozpozná typ adaptéra a nastaví správne kľúčové slovo pre prijímací krúžok: `*ReceiveBuffers`→2048 pre PCIe sieťové karty (Intel I219, atď.) alebo `ReceiveBufferLen`→256 + `PendingReceives`→64 pre **USB** 10GbE radič Realtek (ktorý nezverejňuje `*ReceiveBuffers`). Cieľové hodnoty sú obmedzené na maximálnu hodnotu hlásenú každým ovládačom (`NumericParameterMaxValue`), takže sa nikdy nezapíše hodnota mimo rozsahu. Spustite to z terminálu s **právo na správcu**; rovnako ako pri každom ladení založenom na registri, zmena nadobudne platnosť až po reštarte adaptéra alebo reštarte počítača. Ručné príkazy `Set-NetAdapterAdvancedProperty` uvedené vyššie zostávajú dobrou alternatívou — uplatňujú sa za behu (previažu adaptér) bez potreby reštartu.

### Hostitelia so systémom Linux
— vyrovnávacie pamäte pre príjem cez socket

V systéme Linux, vrátane zariadenia NVIDIA Jetson, majú vyrovnávacie pamäte jadra pre príjem cez socket rovnaký význam ako prijímací krúžok sieťovej karty. Pri predvolenom nastavení jadra (`net.core.rmem_max` / `net.core.rmem_default` = 212 992 bajtov) séria paketov GVSP z kamery pretečie streamovací socket skôr, než ich stihne prečítať Chloros. Pakety sú v jadre zahodené, zatiaľ čo samotná sieťová karta nehlási žiadnu stratu, a pole stráca skupiny snímok a znižuje svoju snímkovú frekvenciu. Pri meraní na poli so 4 kamerami pripojených cez USB 10 GbE adaptér zvýšenie oboch hodnôt na 32 MB zvýšilo snímkovú frekvenciu poľa po pripojení z 4,96 na 7,52 fps a rýchlosť nepretržitého surového záznamu zo 4,7 na 7,6 záznamov/s, pričom v jadre nedošlo k žiadnemu zahodenému paketu.

- **Súbory .deb ich nastavujú.** Inštalácia balíka zapíše hodnoty `/etc/sysctl.d/60-chloros-gige.conf` (obe na `33554432`) a ihneď ich uplatní. Hodnota, ktorá je už nastavená na vyššiu úroveň, zostane nezmenená.
- **`lattice network` ich kontroluje.** Ak je niektorá z hodnôt nižšia, v časti *Zistené problémy* sa zobrazí položka `socket receive buffers too small for GigE Vision (…)` a protokol backendu obsahuje rovnaké varovanie pri prvom pripojení kamery alebo poľa.
- **Ak ich chcete zvýšiť ručne** (do nasledujúceho reštartu; súbor balíka ich aplikuje pri každom spustení):

```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` ich tiež zvyšuje a udržiava, ale len vtedy, ak backend beží ako root. Backend, ktorý beží pod vaším používateľským účtom (spustený s `chloros-backend` alebo [`chloros-backend@<user>`](#spustenie-backendu-ako-služby-bez-grafického-rozhrania)) nemôže meniť nastavenia jadra a namiesto toho vypíše riadok `sudo sysctl`.

**Automatické ladenie adaptérov sa týka iba pripojených adaptérov.** Pri prvom pripojení kamery backend bežiaci ako root naladí káblové adaptéry, ktorých prijímací krúžok je malý. Teraz ladí iba adaptéry s aktívnym spojením, takže odpojený integrovaný port si zachováva svoju hodnotu MTU a prijímacie vyrovnávacie pamäte zostávajú nezmenené. `lattice network` už neuvádza MTU odpojeného adaptéraMTU odpojeného adaptéra ako problém. `lattice network`, `--fix`, `lattice probe` a ladenie adaptérov zohľadňujú iba káblové ethernetové porty, nie CAN zbernicu zariadenia Jetson (`can0`) ani prepojenia v režime USB zariadenia (`usb0`, `usb1`), ktoré ponúka hostiteľskému PC; USB ethernetový adaptér pre kamery sa stále počíta.

### Základy siete (všetky prepojenia LATTICE)

- **Adresovanie:** lokálne pre dané pripojenie `169.254.0.0/16` (GigE Vision LLA). Hostiteľský počítač používa statickú adresu `169.254.x.x/16`; kamery a DAQ-E si adresy prideľujú automaticky v tom istom rozsahu. Nie je potrebný DHCP ani brána.
- **Veľkosť paketu:**uprednostňujte jumbo (9000), ale nechajte to zistiť automatickým testovaním — prostredníctvom GVSP testu obíde 1500-bajtový limit ICMP kamery, takže sa nastaví na jumbo tam, kde to kábel skutočne prenáša.**Každá kamera sa meria raz za reláciu backendu a znovu, keď sa zmení jej spojenie (iný adaptér, iná rýchlosť spojenia alebo prerušenie spojenia, ako napríklad pri výmene prepínača); pri opätovnom pripojení sa výsledok použije znovu. Po zmene za prepínačom, ktorý zostal v prevádzke, reštartujte backend, aby sa meranie vykonalo znovu.**Prvé pripojenie môže pri tejto kontrole občas trvať niekoľko minút a protokol to v takom prípade zaznamená. Použite príkaz „pin“ s kódom `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` len vtedy, ak viete viac ako sonda, a uprednostňujte príkazy na jednotlivé príkazy pred trvalými nastaveniami: príkaz „pin“ obíde sondu, takže ak cesta nedokáže skutočne preniesť 9000,**každé** zachytenie vyprší s chybou `SC_ERR_TIMEOUT -1011` (pozri [Premenné prostredia](#environment-variables)).
- **Veľkosť prstenca RX sa mení v závislosti od hodnoty `ReceiveBufferLen`:**pri predvolenom nastavení `32` je použiteľný prstenec ~0,26 MB (príliš malý na akýkoľvek burst z viacerých kamier); pri maximálnom nastavení `256` je veľký (~13,5 MB namerané na laboratórnom 10GbE hostiteľovi), čo poskytuje skutočnú rezervu. O tom, či sa konfigurácia pripojí, rozhoduje kontrola pripustnosti zohľadňujúca odber**a** nižšie uvedená kontrola agregovaného nadmerného predpísania – nie surové porovnanie burstu s krúžkom.

### Model fps a burstu pole

Ako čítať panel Nastavenia poľa (a `lattice analyze-array` / `analyze_array_network` v SDK):

- **Burst sa sčítava pre každú kameru v jej skutočnom formáte pixelov.**Mono**M3M**kamery streamujú**Mono12 (2 B/px)**, alebo s `--pixel-format` mono formát jej hĺbky (`BayerRG8` je `Mono8`, 1 B/px); kamery**M3C**s filtrom Bayer vysielajú 8- alebo 12-bitový signál (firmvér M3C bez upozornenia vysiela BayerRG12 aj v prípade, že je požadovaný BayerRG8). Takže snímka v plnom rozlíšení zo 4 kamier má veľkosť**~12,6 MB, ak sú všetky 8-bitové, ale ~25 MB s tromi 12-bitovými mono kamerami**. Projekcia určuje formát každej kamery na základe jej modelu (identifikačná cache), takže dátový tok zodpovedá tomu, čo skutočne prenáša kábel — nie je to predpoklad, že všetko je vo formáte BayerRG8.
- **USB ethernetový adaptér je obmedzený na 200 MB/s bez ohľadu na údaje na typovom štítku.** Tabuľka účinnosti, ktorá prevádza rýchlosť spojenia na trvalú hodnotu, je odvodená od PCIe; sieťová karta USB uvádza svoju *ethernetovú* rýchlosť spojenia, je však obmedzená zbernicou USB a jej ovládačom. USB 10GbE adaptér dosahoval rýchlosť ~1063 MB/s „trvalú“ rýchlosť – hodnotu, ktorá nebola nikdy overená – a výsledné spomalenie poškodilo 6–18 % rámcov, pričom stále vykazovalo správnu cieľovú hodnotu fps. Rýchlosť sieťových kariet pripojených cez USB je teraz obmedzená na absolútnu hodnotu: **200 MB/s** v systéme Windows a na NVIDIA Jetson, **400 MB/s** v systéme Linux x86_64 (v systéme Linux je adaptér pripojený k portu USB 2.0 obmedzený na **30 MB/s** a v popise špecifikácií sa uvádza, že je potrebné ho pripojiť k portu USB 3) (obmedzenie spočíva v USB zbernici a ovládači hostiteľa, takže sa nezvyšuje podľa technických parametrov; USB 1 GbE adaptér dosahuje rýchlosť ~80 MB/s a nie je týmto obmedzený). V systéme Linux sa adaptér identifikuje prostredníctvom sysfs (s náhradným označením `ethtool -i` pre `bus-info`). Každá hodnota je nameraná: v systéme Windows tá istá zostava 4 kamier poškodila 6–10 % snímok pri rýchlosti nad 200 MB/s; v systéme Linux x86_64 ten istý adaptér fungoval bezchybne pri každej maximálnej rýchlosti až do 480 MB/s (dosiahnutých 273 MB/s, čo je vlastný limit kamier). Na platforme NVIDIA Jetson ten istý adaptér fungoval bez chýb len pri 200 MB/s a pri vyšších rýchlostiach dochádzalo k prerušovanému strácaniu snímok. `wire_ceiling_source` v zázname o schopnostiach to uvádza slovne a `nic_is_usb` to označuje. Obe nastavenia je možné prepisovať pomocou `--wire-ceiling-mbps`.
- **Priepustnosť zohľadňuje odber, nie porovnanie celého burstu s krúžkom.** Súčasný burst sa musí zmestiť len do *prechodného backlogu* = `max(0, Σ per-cam arrival − host drain) × emit_window`, nie do celého burstu. V štruktúre s rýchlym hostiteľom a pomalými kamerami (**PCIe**10G hostiteľ + 4× 1 GbE kamery: príchod ≈ 320 MB/s, odber ≈ 1063 MB/s) sa hostiteľ vyprázdňuje rýchlejšie, než sa kamery napĺňajú, backlog ≈ 0, takže simulované vysielanie v plnom rozlíšení**je povolené**, hoci 25 MB burst presahuje 13,5 MB ring. Ak pripojíte tie isté štyri kamery k**USB** 10GbE adaptéru v systéme Windows, odosielanie je 200 MB/s, nie 1063 — prichádzajúci tok ho predbieha a strata sa prejaví skôr ako poškodené rámce než ako nižšia snímková frekvencia. Na hostiteľskom počítači s 1 GbE spôsobuje minimálna hodnota DLThr kamier 31,25 MB/s, že prichádzajúci tok prevyšuje odber → správne **blokuje** (pre *túto* triedu blokov znížte ROI alebo použite zlučovanie ≥ 2). Priepustnosť je jednou z **dvoch** brán pripojenia — druhou je nižšie uvedená kontrola agregátneho nadmerného predplatného.
- **Predpokladaná snímková frekvencia (fps) zodpovedá cyklu najpomalšej kamery.**Pri simultánnom vysielaní každá kamera odosiela svoj snímok v rovnakom čase, takže pole pracuje v cykle, ktorý je určený dlhším z dvoch časov: časom na čítanie senzora najpomalšieho člena a časom vysielania (počet bajtov snímku ÷ rýchlosť vysielania kamery:**prístupové spojenie** kamery, 1 GbE ≈ 80 MB/s, alebo jej podiel na maximálnej prenosovej rýchlosti hostiteľa, ak je táto nižšia). Každá expozícia sa prekrýva s čítaním predchádzajúceho snímku, takže expozícia nevstupuje do cyklu: automatická expozícia vyplní obdobie dané rýchlosťou, potom zvýši zosilnenie a tmavá scéna už neznižuje rýchlosť (vo verzii 1.2.0 to tak bolo). Iba manuálna expozícia dlhšia ako toto obdobie spomaľuje spúšťanie (pozri [Expozícia na člene poľa](#exposure-on-an-array-member)). Pridanie kamier nezpomalí pole, pokiaľ sa nezmenší ich podiel na prenosovej šírke. Pri sériovom vysielaní (`slip-emit-and-capture`) sa namiesto toho sčitujú časy vysielania kamier a expozícia každého člena sa zohľadňuje v časovom posune. Merané pri plnom senzore, 12-bitovom režime, na poli s 9 kamerami s hostiteľom PCIe 10 GbE: 10,7 fps, alebo 14,0 fps s [baleným prenosom](#packed-12-bit-transport---packed).
- **Po nepretržitom snímaní projekcia uvádza, akú rýchlosť si udržala.** Snímanie trvajúce 30 s alebo viac pri automatickej rýchlosti zaznamenáva rýchlosť, na ktorej sa ustálila pri danej veľkosti poľa na tomto počítači; krátke pokusy regulátora rýchlosti o dosiahnutie rýchlejšej rýchlosti sa nezohľadňujú. Od tohto momentu riadok `array-connect` a panel Nastavenia poľa uvádzajú túto rýchlosť, ak je nižšia ako rýchlosť modelu, spolu s dôvodom obmedzenia: `clock-bound` (pole bežalo pri rýchlosti, na ktorej sa ustálilo, snímacia slučka čakala na spúšťací takt), `loop-bound` (snímacia slučka nemala dostatok času), `capture-bound` (snímacia slučka zaostala a preskočila skupiny) alebo `measured` (nie je jasné, o ktorú situáciu ide). Hodnota modelu je uvedená v `wire_fps_max`. Pole stále stúpa až na úroveň, ktorú prenáša sieť.
- **Pole funguje v akejkoľvek sieti — záleží len na snímkovej frekvencii.**Každá kamera dostáva rovnaký podiel z**limitu káblovej kapacity bez kolízií**(`sustained × sim_emit_factor`), synchronizovaného s GevSCPD, a všetky naďalej exponujú pri rovnakom spúšťači. Na pomalom kábli je podiel malý a s ním klesá aj snímková frekvencia: 8 kamier s plným rozlíšením a 12-bitovou hĺbkou na 1 GbE bez jumbo rámcov dostane 52 / 8 = 6,5 MB/s každá, čo je približne**1 fps**(približne 1,3 fps po kompresii), zatiaľ čo 10G hostiteľ ich spúšťa rýchlosťou 11–14. Ak kamery nemožno synchronizovať tak pomaly — pre zistenie tohto faktu prečítajte rozsah GevSCPD každej kamery — pole prejde na**sériové**vysielanie (`slip-emit-and-capture`): kamery sa striedajú na linke pri plnej rýchlosti spojenia, spúšťajú sa jedna po druhej, takže každý rámec stále dorazí, ale členy sú**nezobrazované súčasne**. `analyze-array` a výstupný názov príkazu connect označujú tento režim; nastavte `--force-tier sim-capture-sim-emit` na hodnotu, aby ste odmietli prechod na náhradný režim. Veľkosť snímky nemá vplyv na toto zdieľanie, takže**zlučovanie snímok a menšia oblasť záujmu (ROI) zvyšujú snímkovú frekvenciu, ale nemenia to, či sa pole zmestí**. Iba vedenie, ktoré nedokáže prenášať pole ani sériovo, je**preťažené**— napr. oveľa viac kamier, než dokáže jedno vedenie zvládnuť, alebo mnoho iných kamier vysielajúcich na ňom — a v takom prípade je príznakom strata paketov GVSP, nie nižšia rýchlosť: `analyze-array` vynuluje hodnoty dosiahnuteľných snímok za sekundu a vypíše `** OVER-SUBSCRIBED**`, pripojenie pokračuje s hlasným varovaním a HTTP pripojenie s `pin_resolution: true` (prísny režim) **odmietne**. Riešením je znížiť počet kamier, použiť jumbo rámce v celom reťazci alebo rýchlejšiu sieťovú kartu. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` zníži úroveň odmietnutia na hlasné varovanie pre testovacie účely — pozri [Premenné prostredia](#environment-variables).

### Stav poľa — ktorý subsystém stráca rámce

`GET /api/camera/array/<array_id>/capability` pripojeného poľa obsahuje živý
blok `health`, ktorý sa prehodnocuje v priebežnom **10-sekundovom** okne. Rozdeľuje stratu rámcov
na dve príčiny, ktoré si vyžadujú opačné opravy, namiesto toho, aby hlásilo jednu „neúplnú“
mieru, ktorá žiadnu z nich nešpecifikuje:

| Pole | Čo to znamená | Ktorý subsystém |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (na sériový port) | Rámec **dorazil a bol štrukturálne poškodený**— strata paketov GVSP. |**Sieť**: kapacita linky, tempo, prstenec prijímača sieťovej karty, MTU |
| `never_arrived_rate_pct` (na sériové číslo) | Rámec **vôbec neprišiel**— kamera sa nespustila alebo z nej nič nevyšlo. |**Spúšť / synchronizácia**: kábel M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Najhoršia miera neúspešnosti kamery pre každú z nich. | — |
| `per_cam_rate_pct` | Kombinovaná miera neúspešnosti na kameru (obe príčiny spolu). | — |
| `stable_for_seconds` | Ako dlho zostala každá kamera pod hranicou 0,01 %. | — |

Pri hodnote nad 5 % backend zaznamená do protokolu riadok `[array-health <id>] WARN` s označením rozdelenia — pri
prvom porušení, pri zmene stupňa závažnosti, raz za minútu, pokiaľ stav pretrváva, a raz, keď
sa stav vyrieši. Poškodená polovica zapíše do protokolového súboru backendu riadok `[gvsp-corrupt <SN>]` pri
prvom dosiahnutí prahu na kameru a dôvod, potom súhrn každých 60 s; panel protokolu aplikácie namiesto toho zobrazuje riadok
`[array-health <id>]`. Každé vyhodnotenie sa aj naďalej zaznamenáva do backendového súboru protokolu;
počítadlá sa posúvajú pri každom vyrovnávacom pamäti bez ohľadu na to, čo sa vypíše.

Ten istý záznam uvádza číslo, na ktorom závisí celá alokácia:

| Pole | Čo znamená |
| --- | --- |
| `wire_ceiling_mbps` | Aktuálny trvalý rozpočet šírky pásma hostiteľa, MB/s. |
| `wire_ceiling_source` | Odkiaľ pochádza toto číslo, slovami — napr. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (v Linux e x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) alebo `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, ak to nastaví `--wire-ceiling-mbps` (alebo pole **Wire Budget** v grafickom rozhraní). |
| `nic_is_usb` | `true` pre USB ethernetový adaptér — pozri obmedzenie USB vyššie. |

**Výklad:** hodnota `gvsp_corrupt_rate_pct` odlišná od nuly pri hodnote `never_arrived_rate_pct` rovnej 0
znamená, že spúšťanie a synchronizácia kábla sú v poriadku a 100 % strát je na sieťovej
trase — znížte hodnotu `--wire-ceiling-mbps` a znovu sa pripojte. Opačný vzor poukazuje skôr na
synchronizačný kábel alebo spúšťaciu linku.

> **`--target-fps` nie je kľúčovým faktorom pre poškodené rámce.** Tempo GevSCPD sa zapisuje
> raz pri pripojení, takže zníženie frekvencie spúšťania mení pracovný cyklus, nie však
> rýchlosť súbežného vysielania. Zmerané 5-násobné zníženie požiadaviek neprinieslo žiadne zlepšenie;
> zníženie maximálnej rýchlosti z 240 na 200 MB/s znížilo na tom istom zariadení podiel poškodených rámcov z 10,4 %
> na 0,00 %.

> **Bežiace pole sa nemôže samo preplánovať na firmvéri LATTICE (M3C / M3M).** Nemôže
> to opraviť sama; odpojte a znovu pripojte, aby nástroj na výber času pripojenia mohol
> preplánovať s novým limitom.

### Príznak → riešenie

| Príznak (Nastavenia poľa / pripojenie / `analyze_array_network`) | Príčina | Riešenie |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` sa resetuje na 32 (zvyčajne po aktualizácii ovládača) | Nastavte `ReceiveBufferLen`→256, `PendingReceives`→64; znovu otvorte panel (reštartujte backend, ak si uložil do vyrovnávacej pamäte starú veľkosť prstena) |
| Zablokovanie pri reštarte/vypnutí; neskôr chyby WMI `Invalid class`, sieťová karta sa nedá aktivovať, chýbajú USB zariadenia | Starý ovládač Realtek USB 10GbE z roku 2016 → BSOD `0x9F` → nútené vypnutia | Aktualizujte ovládač adaptéra na verziu ≥ v10.67 (2026), potom znovu použite vyššie uvedené nastavenia prijímacieho prstenca |
| Pole sa pripája pri plnom senzore, ale má nízku snímkovú frekvenciu | Režim Auto udržiava plný senzor a necháva frekvenciu klesnúť na úroveň, ktorú prenáša kábel | Zvoľte jednu z rýchlejších možností uvedených v zozname výstupov pripojenia (`--binning 2` / `4` alebo nižšiu hodnotu `--width`/`--height`), povolte jumbo rámce alebo aktualizujte spojenie |
| Pole hlási správnu cieľovú hodnotu fps, ale poskytuje len jej zlomok; `health.gvsp_corrupt_rate_pct` nie je nulové, `never_arrived_rate_pct` 0 | Odhadovaná šírka pásma hostiteľa nadhodnocuje skutočnú priepustnosť (typické pre USB ethernetový adaptér, úzku PCIe linku alebo zdieľanú sieťovú štruktúru) | Znovu sa pripojte s nižšou hodnotou `--wire-ceiling-mbps` a znovu skontrolujte blok stavu. **Nie** `--target-fps` — Pacing GevSCPD je pri pripojení pevne nastavený |
| Linux: pole stráca skupiny rámcov a znižuje svoju rýchlosť prenosu rámcov, zatiaľ čo sieťová karta nehlási žiadne straty; `lattice network` uvádza `socket receive buffers too small` | Prijímacie vyrovnávacie pamäte socketov jadra sú nastavené na predvolenú hodnotu, takže pakety z kamery sú v jadre stratené | Zvýšte hodnotu `net.core.rmem_max` / `rmem_default` na 32 MB — pozri [hostitelia Linux](#linux-hosts--socket-receive-buffers). Balík .deb to vykoná pri inštalácii |
| Kamery chýbajú v zverejnených skupinách; hodnota `health.never_arrived_rate_pct` je odlišná od nuly, `gvsp_corrupt_rate_pct` 0 | Spúšťacia/synchronizačná cesta — kamery sa nespúšťajú, nejde o sieťový problém | Skontrolujte synchronizačný kábel M8 a hodnotu `--line`; overte, či sú všetky zariadenia v režime „armed“ (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` prekročené v `analyze-array` alebo odmietnutie pripojenia v prísnom režime (`pin_resolution`; `array over-subscribes the wire`) | Kábel nedokáže prenášať dáta pri žiadnej frekvencii, ktorú kamery dosahujú, či už súčasne alebo postupne — je to oveľa viac kamier, než koľko kábel dokáže obslúžiť, alebo na ňom streamujú aj iné kamery | Menší počet kamier, jumbo rámce od začiatku do konca alebo rýchlejšia sieťová karta. **ROI/binning nepomôže** (zdieľanie je nezávislé od veľkosti rámca). Chyba `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` sa na testovacom prostredí prekoná (akceptuje stratu paketov) |
| Pole sa pripája v režime `slip-emit-and-capture` (sériovo) v pomalej sieti | Kamery nemožno synchronizovať dostatočne pomaly, aby mohli zdieľať kábel súčasne, preto sa striedajú; snímky nie sú exponované v rovnakom okamihu | Jumbo rámce alebo rýchlejšia sieťová karta obnovujú simultánne snímanie; `--force-tier sim-capture-sim-emit` na tom trvá (a namiesto toho stráca pakety) |

---

## `chloros-cli daq`

Príkazy pre spektrálny senzor. Príkazy `pool-*` sú jednoduché HTTP-klienty, ktoré ovládajú senzor prostredníctvom trvalého poola backendu, preto potrebujú spustený backend (pozri [Spustenie backendu](#spustenie-backendu)). Backend vlastní transport, takže skripty CLI a SDK zdieľajú jednu aktívnu referenciu namiesto toho, aby sa hádali o sériový port. `pool-discover` je jediný spôsob, ako zistiť BLE MAC zariadenia DAQ-M.

> **`chloros-cli daq --help`** (alebo `-h`, alebo `daq help`) vypisuje podpríkazy `pool-*` (v Linux u spolu s príkazmi pre priamy prístup k hardvéru, ktoré sú súčasťou kompilácie), a `chloros-cli daq <subcommand> --help` (alebo `daq help <subcommand>`) uvádza možnosti každého z nich. Týkajú sa pripojenia, stream, záznam kalibrovaných súborov `.daq`, výmenu profilov kapacitných senzorov a priradenie DAQ k kamere, a v prípade DAQ-E aj jeho sieťové nastavenia, PTP, reštart a aktualizácie firmvéru. DAQ je možné ovládať aj z jazyka Python pomocou `chloros_sdk.connect_daq_sensor()`, ktorý používa tú istú združenú cestu.
>
> **Používajte príkazy `pool-*` na každej platforme.** Správajú sa rovnako v systémoch Windows, Linux a Jetson. V systéme Windows sú `daq discover` a `daq list` sú aliasy príkazu `pool-discover` a akéhokoľvek iného príkazu `daq`, ktorý nie je `pool-*` ukončí `1` s `DAQ direct-hardware commands require the full daq package`. V systémoch Linux a Jetson sú príkazy `daq discover`, `daq list` a ostatné príkazy, ktoré nie sú typu `pool-*` a `daq`, priamymipríkazy na úrovni hardvéru: otvárajú senzor vo vlastnom procese CLI, mimo fondu backendu (teda nie na senzore, ktorý drží backend, grafické rozhranie alebo skript), a tam sa prijímajú príkazy `daq discover`, `--only` a `--timeout`, ale nie `--json`. Táto referenčná príručka dokumentuje iba príkazy `pool-*`.

### Postup prvého pripojenia senzora DAQ

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referenčná príručka k `pool-*`

| Podpríkaz | Účel |
| --- | --- |
| `daq pool-connect` (smart-detect) | Otvorí senzor v backendovej skupine. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U na konkrétnom sériovom porte. |
| `daq pool-connect --ble` | DAQ-M cez BLE, automatické vyhľadávanie MAC. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M cez BLE na známej adrese MAC (predpokladá `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E cez Ethernet so známym názvom hostiteľa alebo IP adresou (predpokladá `--eth`). |
| `daq pool-connect --eth` | DAQ-E cez Ethernet, automatické vyhľadanie hostiteľa (mDNS + záložný režim ARP; funguje aj s prázdnou vyrovnávacou pamäťou ARP v systémoch Windows a Linux). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Nastavenie integračného okna / stavu AE. |
| `daq pool-connect --no-stream` | Pripojiť, ale zatiaľ nespustiť prenos (pokračovať s `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil korekcie Cap. Predvolený profil na backende je `sunshine_cosine`; `none` je holý senzor. Senzor typu [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) ho ignoruje (pevný difúzor). **Senzory sa dodávajú bez krytu, preto pomenujte kryt, ktorý ste nasadili:** predvolené nastavenie sa zaznamenáva ako predpokladané (`cap_id_source=auto_default`) a spracovanie na to upozorní, keďže holý DAQ-U pri korekcii slnečného žiarenia vykazuje približne 30-násobne vyššie hodnoty. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Prehľadávajte každý transport na prítomnosť senzorov, ku ktorým by ste sa mohli pripojiť, bez toho, aby ste sa pripájali. **Takto zistíte BLE MAC adresu zariadenia DAQ-M.** `--timeout` sa vzťahuje na každý transport (predvolené 5 s, 1–20). Senzory, ktoré sú už otvorené v poole, sa nezobrazujú – pripojený DAQ-M prestane vysielať reklamy – preto pre ne použite `pool-list`. `usb` vypisuje iba sériové zariadenia USB: vlastné sériové porty počítača (`COM1`, `/dev/ttyS*` v Linux u) nemôžu byť senzormi a sú vynechané. |
| `daq pool-list` | Zobraziť všetky senzory v backendovej skupine. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Priradiť kameru alebo pole k DAQ, podobne ako to robí výber svetelného senzora v grafickom rozhraní: jeho snímky používajú, porovnávajú a ukladajú hodnoty z tohto DAQ. `SENSOR` je identifikátor senzora z `pool-list` alebo adresa, na ktorej je DAQ dostupný. `--clear` (nie `SENSOR`) zruší priradenie a snímanie sa vráti k najnovšiemu DAQ. Priradenie trvá až do reštartovania backendu alebo do odpojenia danej kamery, poľa alebo DAQ: odpojenie (v tomto prípade z SDK alebo v aplikácii) zruší všetky priradenia hardvéru, ktorý sa odpojí, bez ohľadu na to, kto ich nastavil, vrátane členov poľa spolu s poľom. Kamera musí byť pripojená v otvorenom projekte alebo predtým otvorená na tomto počítači a pole musí byť pripojené alebo v otvorenom projekte; akékoľvek iné sériové číslo alebo ID je odmietnuté na základe názvu a nič sa nepripojí. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Uvoľnenie — bez ohľadu na to, kto senzor otvoril, takže uvoľňuje aj senzor, ktorý v poole zanechal havarovaný skript (akékoľvek nahrávanie na ňom sa najskôr dokončí). Senzor, ktorý nie je v poole, nemá čo uvoľniť: vypíše `ID: already released (not in the pool)` a ukončí sa `0`; uvoľnenie, ktoré backend odmietol alebo nepotvrdil, ukončí sa `1`. SDK: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Najnovších N rámcov spektra (predvolene 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Obnovenie / pozastavenie streamovania. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Spustenie / zastavenie nahrávania .daq. `-o` je predvolene nastavené na `~/Documents/DAQ Live View` na počítači backendu (relatívna cesta sa nachádza v priečinku, z ktorého spúšťate príkaz); `--duration` (`-d`, predvolené nastavenie `0` = až po `--stop`). `--confirm-after S` vydá varovanie, ak S sekúnd po spustení neprišla žiadna hodnota (predvolené nastavenie 20; hodnota 0 túto funkciu vypne); s nastavením `--duration` `--heartbeat S` vypíše počet údajov každých S sekúnd a upozorní, keď prestane rásť (predvolené nastavenie 30; hodnota 0 túto funkciu vypne). Záznam na DAQ-E, ktoré prestane vysielať, pokračuje v tom istom súbore, akonáhle sa backend opäť pripojí, a `--stop` vypíše medzery ([DAQ-E, ktoré prestane vysielať](#a-daq-e-that-stops-delivering)). Kombinácia klávesov Ctrl+C predčasne ukončí nahrávanie `--duration` a nahlási to; bez `--duration` sa nahrávanie po stlačení Ctrl+C (ukončenie `130`) pokračuje až do `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Zmena profilu korekcie kondenzátora počas behu programu. Je potrebné nastaviť `--cap-id`; `--cap-id none` odstráni korekciu kondenzátora (holý senzor). DAQ-E-S zachová `as_recorded` bez ohľadu na nastavenie. |
| `daq pool-enable-ptp --sensor-id ID` | Znovu zapnite PTP pre DAQ-E, ktorého PTP je vypnuté. Nastavenie sa uloží do zariadenia, takže zariadenie, ktoré zostalo vypnuté, zostane vypnuté — jeho časové značky nie sú synchronizované cez PTP — až kým sa tento príkaz nespustí. `pool-list` označí takúto jednotku (`!! PTP is switched OFF`) a vypíše pre ňu tento príkaz. Chloros nikdy sám od seba jednotku opäť nezapne, pretože vypnutie môže byť zámerné. |
| `daq pool-info --sensor-id ID [--json]` | Vlastné nastavenia zariadenia DAQ-E: jeho firmvér (a či táto verzia Chloros u obsahuje novšiu verziu), jeho adresa a spôsob jej získania (DHCP / link-local / statická), uložené sieťové nastavenie, kým ho senzor nespustí (jedno uložené s priloženým `--no-reboot`), prúd pri zapnutí a PTP. V prípade senzora, ktorého prúd nedosahuje tento počítač, prečíta senzor pri novom pripojení a oznámi to. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Nastavenie DAQ-E na DHCP (`--auto`) alebo statickú adresu (`--mask`, predvolené nastavenie `255.255.255.0`). Senzor sa reštartuje, aby sa nastavenie uplatnilo, a opäť sa pripojí s pôvodnými nastaveniami, pričom na to čaká až `--timeout` sekúnd (predvolene 45); `--no-reboot` ju iba uloží. Pozri [Nastavenia zariadenia DAQ-E](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Určuje, či zariadenie DAQ-E začne pri zapnutí vysielať dáta bez pripojeného softvéru (uložené na senzore). Program Chloros spustí vysielanie v oboch prípadoch. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Reštartujte zariadenie DAQ-E a opätovne ho pripojte s predchádzajúcimi nastaveniami (čaká maximálne `--timeout`, predvolene 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Nahrajte firmvér zariadenia DAQ-E, ktorý je súčasťou tejto verzie Chloros, do senzora so staršou verziou a potom ho znovu pripojte (trvá to približne minútu; nevypínajte ho; predvolený čas na obnovenie činnosti je 45 s). Senzor, ktorý už má aktuálnu verziu, sa ponechá bez zásahu. |

### DAQ-E, ktorý prestane odosielať údaje

DAQ-E v bazéne, ktorý 20 sekúnd neodosiela žiadne spektrum, je opätovne pripojený backendom. Dosiahne sa to vypnutím a opätovným zapnutím prepínača, ktorý napája senzor: senzor sa spolu s ním reštartuje a nezačne opäť odosielať údaje, kým nedostane pokyn (pokiaľ nie je nastavené `pool-set-auto-stream --on`).

- **Najprv skontroluje, či ide o ten istý senzor.** Na adrese, ktorú mal senzor, bez akejkoľvek zmeny požiada o názov senzora, MAC a sériové číslo. Žiada tiež pod názvom senzora `daq-e-<id>.local`, ale dôveruje tejto odpovedi len vtedy, ak pochádza z vlastného sieťového adaptéra (MAC) senzora. Žiadne iné zariadenie, ktoré odpovie, nie je opätovne pripojené ani sa do neho nezapisuje.
- **Znovu sa pripojí s nastaveniami relácie:** čas integrácie, priemerná hodnota snímok, automatická expozícia a profil obmedzenia zostanú tak, ako boli, nie podľa predvolených nastavení senzora pri zapnutí. Senzor, ktorý mal zapnutý protokol PTP a vrátil sa s vypnutým protokolom, sa opäť zapne. Potom sa znovu spustí prúd dát a prijímač počítača pre tento prúd.
- **Opätovné pokusy sa vykonávajú v postupne sa predlžujúcich intervaloch:** ihneď, potom po 5, 10, 20 a 40 sekundách, následne každých 60 sekúnd, pokiaľ je senzor nedostupný. Pri každom pokuse sa senzor opýta len na to, aký je. Senzor, ktorý odpovie, ale po troch pokusoch o opätovné pripojenie stále nič neposiela, sa nechá na pokoji a nahlási sa (`auto-reconnect GAVE UP`). `pool-stream --start` to skúša znova.
- **`pool-list` ukazuje, v akom stave sa nachádza.** Kým senzor mlčí, jeho riadok `STREAM FAULT` informuje o tom, čo sa deje pri opätovnom pripájaní. Riadok sa vymaže s prvým spektrom a senzor potom vykazuje hodnotu `reconnected automatically N time(s)`.
- **Záznam pokračuje v rovnakom `.daq`.** Súbor označuje každú medzeru dvoma riadkami: `event_type` 5 pri poslednom spektre pred medzerou a 6 pri prvom po nej, pričom každý obsahuje iba časovú pečiatku. `pool-record --duration` vypíše `!! STREAM LOST …`, kým je senzor preč, a `stream resumed after …`, keď sa vráti. `pool-record --stop` vymenuje každú medzeru a varuje, rovnako ako v prípade prípade prázdneho súboru (záznam, ktorý neobsahuje žiadne namerané hodnoty, ukončí sa na `1`).
- **Záznam si zachováva jednu kalibráciu.** Ak sa senzor vráti s odlišnou kalibráciou počas prebiehajúceho záznamu, zvyšok záznamu si zachová kalibráciu, s ktorou začal, a varovanie pri zastavení na to upozorní. `pool-disconnect` + `pool-connect` preberie novú kalibráciu.
- **Vaše príkazy majú prednosť.** Senzor, ktorý ste zastavili pomocou `pool-disconnect` alebo pozastavili pomocou `pool-stream --stop`, sa nikdy znovu nepripojí, ani keď je proces opätovného pripojenia už v chode.
- **`pool-info` na neaktívnom senzore** prečíta senzor pri novom pripojení a pridá `!! stream NOT reaching this computer`. Senzor, ktorý neodpovedá, je ihneď nahlásený. `pool-list` už nezobrazuje stav PTP pre senzor, ktorý prestal odpovedať.
- DAQ-U a DAQ-M nie sú zahrnuté. DAQ-M sa sám opätovne pripojí cez Bluetooth a DAQ-U odpojený od USB vyžaduje opätovné použitie `pool-connect`.

### DAQ-E-S — pevný difúzor a senzor náklonu

**DAQ-E-S** je zariadenie DAQ-E s dvoma rozdielmi, ktoré sú dôležité pre softvér. Všetko ostatné — vyhľadávanie, pripájanie, PTP, nahrávanie, nižšie uvedené nastavenia zariadenia — funguje presne tak ako pri DAQ-E.**Jeho difúzor je pevne zabudovaný, takže nevyžaduje žiadnu korekciu krytu.** Difúzor typu Sunshine je zabudovaný a bol na zariadení už v čase merania jeho továrenskej kalibrácie, takže táto kalibrácia už udáva skutočné ožiarenie smerom nadol. Program Chloros nastaví jeho kryt na `as_recorded` (bez profilu) bez ohľadu na to, čo sa požaduje: limit s názvom odvodeným od neho (`sunshine_cosine`, `fov_*`, `none`) sa ignoruje so zobrazením správy a jeho záznamy ukladajú hodnotu `cap_id_source=model`, takže varovanie o predpokladanom obmedzení sa v jeho prípade nikdy nespustí. Neaplikujte na jeho údaje obmedzenie ani ručne: `sunshine_cosine` na vrchole by vykazoval približne 11-násobne vyššiu hodnotu.

**Je vybavený 3-osovým akcelerometrom.** Kosínusová charakteristika difúzora slnečného žiarenia platí len vtedy, keď je v horizontálnej polohe, takže naklonený senzor hlási nesprávnu intenzitu žiarenia — a odrazivosť túto chybu preberá — bez akýchkoľvek ďalších príznakov. Každé meranie DAQ-E-S preto obsahuje informáciu o svojej polohe:

| Pole | Význam |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Uhly v stupňoch. `None`, ak firmvér označí uhol za nedefinovaný (voľný pád, nefunkčná os). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Zrýchlenie v mili-g. Hodnota `mag_mg` vzdialená od ~1000 znamená, že jednotka bola v pohybe a uhly pre daný vzorok nemajú význam. |
| `cal_applied` | Nastavené: osi a uhly sú v súradnicovom systéme difúzora (uplatňuje sa kalibrácia upevnenia zariadenia). Vypnuté: surový súradnicový systém dosky, ktorý môže vykazovať sklon až ~2,3°, ktorý nie je skutočný. |
| `sample_age_ms` | Vek vzorky z akcelerometra v čase odoslania spektra. |
| `temp_raw` | **Trend** nekalibrovanej teploty čipu — iba rozdiely, nikdy nie nameraná hodnota teplomera. |

`daq pool-latest --json` ich vráti pod kľúčom `imu` každého rámca (`null` na DAQ-U, DAQ-M alebo bežnom DAQ-E). Údaj `.daq` zaznamenaný z DAQ-E-S ukladá rovnaké hodnoty do svojich stĺpcov `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) vedľa každého spektra, takže čitateľ dokáže rozlíšiť oblak od prevráteného statívu. Na všetkých ostatných modeloch sú tieto stĺpce prázdne.

Dátový tok o polohe sa zaznamenáva samostatne: `daq pool-list` vypíše `!! ATTITUDE STREAM FAULT (spectra unaffected)`, keď je zariadenie vypnuté, a koľko porúch sa mu podarilo následne odstrániť. Porucha v tomto dátovom toku nikdy nespôsobí medzeru v spektrálnom zázname. `daq pool-discover` uvádza jednotku ako `DAQ-E-S` (ako `DAQ-E`, ak neodpovie na kontrolu identity do približne 2 s).

### Nastavenia zariadenia DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` bez `--gateway` neukladá žiadnu bránu** (akákoľvek predtým nastavená brána sa zruší), čo je vhodné pre sieť kamier bez smerovača.
- **V sieti tohto počítača musí byť statická adresa.** Program Chloros odmietne takúto adresu, ak jej podsieť neobsahuje adresu tohto počítača v sieti senzora: po reštarte by sa k senzoru nedalo dostať, aby sa to dalo zmeniť späť. Nastavenie `--force` použite len vtedy, ak bude tento počítač presmerovaný do novej podsiete.
- **Reštart sa odmietne, pokiaľ senzor nahráva.** Najprv zastavte nahrávanie (`pool-record --stop`). Nahrávanie spustené tesne pred reštartom sa týmto reštartom ukončí, a chyby `pool-set-network`, `pool-reboot` a `pool-update-firmware` končia riadkom `Warning:`, v ktorom je uvedený názov súboru záznamu (aj keď sa senzor potom nevráti).
- **Po reštarte Chloros opätovne pripojí iba samotný senzor.** Čaká na mieste, kde sa senzor nachádzal – a po riadku `--auto` aj pod jeho názvom `daq-e-<id>.local` – na odpoveď s názvom senzora, názvu hostiteľa a modelu, z sieťového adaptéra (MAC), ktorý mal, a opätovne ho pripojí na adresu, ktorá odpovedala. Odpoveď pod daným názvom je akceptovaná iba s touto MAC adresou, takže z počítača, ktorý sa k senzoru pripája cez smerovač, alebo z WSL2 v jeho predvolenom NAT sieťovom nastavení, sa senzor, ktorý bol prostredníctvom DHCP presunutý na novú adresu, znovu nepripojí: príkaz to oznámi a vypíše príkaz `pool-connect --eth-host <its address> …`, ktorý ho obnoví s pôvodnými nastaveniami citlivosti a expozície.
- **`pool-update-firmware` odmietne senzor, ktorého verziu firmvéru nedokáže prečítať.** Senzor, ktorého návrat po blesku trvá dlhšie ako nastavená doba čakania, nezlyhal: hneď ako odpovie, je opäť pripojený s vyplnenými nastaveniami nového firmvéru a príkaz upozorní, že bol pomalý.
- **Aktualizácia firmvéru vyžaduje prichádzajúce TCP pripojenie.** Senzor sa opäť pripojí k tomuto počítaču, aby získal snímku. Program Chloros počúva na adaptéri, ktorý smeruje k senzoru (na každom adaptéri, len ak nedokáže zistiť, ktorý to je), a obsluhuje iba vlastnú adresu senzora; akékoľvek iné pripojenie sa uzavrie bez obsluhy. V systéme Windows ho pravidlo firewallu inštalátora `Chloros Backend (TCP-In)` púšťa dovnútra (pozri [Windows (.exe)](#windows-exe)); firewall, ktorý ho blokuje, spôsobí zlyhanie aktualizácie s chybou `… device never connected back on TCP <port> …`.
- **V prostredí WSL2 použite zrkadlenú sieť** (`networkingMode=mirrored` v `%UserProfile%\.wslconfig`). V predvolenom režime NAT je adresa tohto počítača v sieti senzora adresou samotného WSL, takže každá statická adresa je odmietnutá (s výnimkou `--force`), živý prenos sa nedostane do WSL a aktualizácia firmvéru sa nemôže dokončiť, pretože senzor sa pripája späť k počítaču, aby prijal obraz.
- **ID senzora a názov hostiteľa sa nemenia.** Nech má akúkoľvek adresu, DAQ-E je stále `daq-e-<id>` a `daq-e-<id>.local`, takže `--eth-host daq-e-<id>.local` ho naďalej nájde.
- **GUI má rovnaké nastavenia**: karta Svetelné senzory, nastavenia senzora (ozubené koliesko), Sieť. GUI si udržiava vlastné pripojenie k senzoru, takže ak zmeníte senzor, ku ktorému má GUI otvorené pripojenie, priamo v GUI: reštart z CLI ukončí reláciu grafického rozhrania s ním.
- Ekvivalenty v SDK sú `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` a `update_firmware()`.

---

## `chloros-cli project`

Otvorenie, pripojenie a ovládanie uloženého projektu Chloros (zložku obsahujúcu súbory `cameras.json` + `sensors.json` + `project.json`). Všetko prebieha cez backend, takže grafické rozhranie (GUI) a príkazový riadok (CLI) vytvárajú identický stav hardvéru.

### Prehľad podpríkazov

| Podpríkaz | Účel |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Vytlačí zoznam zariadení projektu (kamery, pole, senzory), prečíta ich zo súborov a skontroluje zdrojové súbory (pozri [Presunutý projekt](#a-moved-project---relink)). Projekt sa v backende neotvorí: projekt, ktorý je tam otvorený, ako aj jeho zariadenia a záznamy, zostanú tak, ako sú. Ukončí sa `1`, ak sa nedá nájsť žiadny zo zdrojových súborov projektu. |
| `project devices PATH [--reconnect]` | Zobraziť zoznam alebo znova spustiť vyhľadávanie. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Pripojiť každú uloženú kameru / pole / senzor. Každé pole sa pripája rovnako ako v prípade `lattice array-connect`: až 600 s, pričom jednotlivé kroky sa vypíšu do stderr (`[array <id>]    <s> s: <step>`; stdout je správa vo formáte JSON), a pole, ktoré sa v tomto čase nepripojilo, sa zruší — jeho kamery sa vrátia do pôvodného stavu — a nahlásia sa pod kódom `errors`, nikdy nezostanú pripojené. `--packed` nastaví pre každé pole komprimovaný 12-bitový prenos (predvolené nastavenie je `auto`; projekt neukladá žiadne nastavenia pre jednotlivé polia; pozri [Zhustený 12-bitový prenos](#packed-12-bit-transport---packed)). Každý svetelný senzor sa otvorí v DAQ poole backendu na jeho uloženej adrese s uloženým integračným časom, priemerovaním snímok, automatickou expozíciou a obmedzením: snímač bez zvoleného obmedzenia dostane nastavenie `sunshine_cosine`, snímač uložený bez obmedzenia (`none`) zostane bez obmedzenia a snímač DAQ-E-S nikdy nepoužíva obmedzenie. Panel DAQ v grafickom rozhraní nie je súčasťou skupiny: senzor najprv zatvorte tam. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Jednorazový záznam z pomenovanej kamery alebo poľa (`--prefix` – predvolené, `capture`). `--orient` (polia): pole ukladá surové dáta, ktoré zostávajú v natívnom formáte — tým sa zaznamenáva, že produkty, ktoré z nich vytvára `process`, majú byť orientované ako `array-capture --orient`. Predvolené nastavenie: natívne. `project burst` / `stream` a recepty `capture` / `burst` / `stream` (`orient: true`) majú rovnakú voľbu. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Séria N snímok z určenej kamery alebo poľa (`-n/--count` predvolené 5; `-i/--interval` interval medzi snímkami v sekundách, predvolené 0). Séria snímok z poľa odstraňuje duplicitné opakované synchronizované skupiny (kontrolný mechanizmus zastaralosti), takže pole s čiastočným cyklom nemôže vrátiť N kópií jedného snímku; vypíše výsledky za každú iteráciu. Zlyhanie zachytávania v priebehu procesu zastaví sériu a ukončí `1` s uvedením počtu snímok uložených pred zlyhaním. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Zachytávanie na disk v slučke: `--fps F` zachytáva po jednej (predvolene 3), jeden snímok na zachytenie z kamery, jednu synchronizovanú skupinu snímok z poľa (s rovnakým kontrolným mechanizmom zastaralosti ako `project burst`), kým `-n N` nezachytí (predvolené nastavenie `0` = až do stlačenia Ctrl+C, čo vypíše počet uložených snímok a ukončí program `0`, alebo `1`, ak nebola uložená žiadna snímka alebo došlo k chybe súboru; stlačenie klávesovej skratky Ctrl+C pred zachytením `-n N` ukončí program `130`). Zlyhanie zachytávania v priebehu procesu zastaví prúd a ukončí program `1` s informáciou o počte snímok uložených pred zlyhaním. `--poll-interval` je zastarané a ignorované: stále sa však akceptuje, takže staršie skripty fungujú, pričom sa na stderr vypíše upozornenie. |
| `project sensor read PATH NAME [--json]` | Najnovší rámec spektra. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Nahrávanie súboru .daq pre `--seconds` (predvolené 10). Záznam, ktorý neobsahuje žiadne hodnoty, sa nahlási na stderr a program sa ukončí `1`. |
| `project run PATH RECIPE.yaml` | Spustiť recept na zachytávanie YAML/JSON. `--dry-run` vykoná validáciu bez spustenia. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Vypočíta zarovnanie pre pole — pozri [tabuľku príkazových vlajok nižšie](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Vypíše aktuálny profil zarovnania. |
| `project align clear PATH NAME` | Vymaže profil z vyrovnávacej pamäte. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Posunie transformáciu jedného podriadeného (predvolené hodnoty `0`, `0`, `0`, `1.0`: bez zmeny). |
| `project align export PATH NAME --to FILE` | Uložiť profil do formátu JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Načítať uložený profil. |

#### Možnosti `project align calibrate`

| Príznak | Predvolené | Popis |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Metóda zarovnania. **Tieto názvy sa líšia od `lattice align-calibrate`**, ktorý prijíma skrátené formy `orb` / `akaze` / `phase`; tieto dva príkazy nie sú v rámci tohto príznaku zameniteľné. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformácia modelu tak, aby sa prispôsobil. |
| `--frames N` | `1` | Synchronizované snímky snímok na priemeru. |
| `--reference SN` | hlavnej | Sériové číslo referenčnej kamery; všetky ostatné členy sú na ňu deformované. |
| `--max-features N` | `5000` | Limit počtu charakteristík ORB. |
| `--ratio-threshold F` | `0.75` | Test pomeru Lowe'test pomeru. |
| `--ransac-threshold-px F` | `3.0` | Prahová hodnota vnútorných bodov RANSAC. |
| `--min-matches N` | `15` | **Kritérium kvality** — odmietne riešenie, ak je počet zhodných vnútorných bodov nižší ako táto hodnota. |
| `--max-reproj-err-px F` | `4.0` | **Kritérium kvality** — odmietnuť riešenie, ak je chyba reprojekcie RMS vyššia ako táto hodnota. |
| `--checkerboard RxC` | — | Geometria dosky pre `--method checkerboard`, napr. `9x6`. |
| `--name PROFILE` | prázdne | Názov profilu vložený do uloženého súboru JSON. **Nie je to názov poľa** — to je pozičný `NAME`. |

Tieto dve kontroly kvality sú dôvodom, prečo sa kalibrácia môže úspešne vyriešiť, ale napriek tomu
sa odmietne uložiť: profil, ktorý nesplní aspoň jednu z nich, by bez upozornenia nesprávne zaregistroval každé
neskoršie zachytenie, preto je odmietnutý namiesto toho, aby bol uložený.

### Presunutý projekt (`--relink`)

Projekt `project.json` označuje každý zdrojový súbor — každý obrázok a záznam zo svetelného senzora
— podľa jeho úplnej cesty. Ak presuniete priečinok projektu alebo celý priečinok projektov
na iný disk, tieto cesty budú naďalej odkazovať na pôvodné umiestnenie.

Otvorenie projektu opraví to, čo dokáže sám: súbor, ktorý bol v zložke projektu,
sa vyhľadáva v zložke, v ktorej sa projekt nachádza teraz, a každá cesta, ktorá sa tam nájde,
sa preorientuje a uloží (`project.json` sa najskôr zálohuje ako
`project.json.before-relink-<date>-<time>`). K tomu dochádza pri každom otvorení – aplikácie,
`chloros-cli project open`, `chloros-cli process` do existujúceho projektu, SDK a `chloros-mcp`. Cesta sa preorientuje iba na súbor, ktorý sa tam nachádza.

Súbory, ktoré sa nachádzali mimo zložky projektu – napríklad v rámci relácie Hub alebo pri importe pomocou funkcie „Pridať zložku“ –
sa týmto spôsobom nedajú nájsť. Uveďte názov zložky, v ktorej sa teraz nachádzajú:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` sa vzťahuje na zložku, do ktorej boli súbory presunuté, alebo na akúkoľvek zložku nad ňou: zložka každého chýbajúceho
súboru sa hľadá pod ňou podľa koncových názvov zložiek
(`Hub Projects/<session>/raw/<serial>`, potom `<session>/raw/<serial>`, ...), a súbory v každom
súbory z daného priečinka sa načítajú z toho miesta, kde sa ich nájde najviac. `project open` vypíše,
čo bolo preorientované, koľko zdrojových súborov sa našlo a v ktorých priečinkoch sa nachádzali
zvyšné. Niektoré stále chýbajú: varovanie, ukončenie `0` – projekt sa otvorí so zvyškom. Nenašlo sa
nič: chyba s uvedením názvov týchto zložiek a `--relink`, ukončenie `1`; backend rovnako
odmieta otvoriť takýto projekt pre `process` alebo `project devices --reconnect` (HTTP 409 `PROJECT_SOURCES_MISSING`).

### Príklady

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL receptov

`project run RECIPE.yaml` prijíma súbor vo formáte YAML alebo JSON, ktorý popisuje postupnosť akcií:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Podporované akcie: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` preberá kľúče `cameras.json`, `settings` — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (staršie názvy `exposure_time`, `exposure_auto`, `gain`, `gain_auto` sú akceptované ako aliasy). Expozícia, zosilnenie alebo cieľová hodnota nastavená týmto spôsobom zostáva zachovaná: automatické riadenie expozície programu Chloros (vrátane expozície meranej systémom DAQ) ju nemení. Ručná expozícia ponechá zosilnenie tak, ako bolo: v automatickom režime fotoaparát neustále upravuje zosilnenie (v celom rozsahu) smerom k cieľovej jasnosti, kým nenastavíte `gain_db` alebo `auto_gain: "Off"`. Následné zadanie príkazu `auto_exposure: "Continuous"` vráti expozíciu späť do automatického režimu a príkaz `target_brightness_auto: true` vráti zafixovanú cieľovú hodnotu späť do automatického nastavenia programu Chloros. Akcia `burst` prijíma parametre `name` (povinný), `count` (predvolené nastavenie 5), `interval` (sekúnd, predvolené 0), `output`, `format`, a `settings` (rovnaký tvar nastavení pre každú kameru ako `apply`); séria snímok v poli používa rovnaký strážny mechanizmus pre čerstvo synchronizovanú skupinu ako `project burst`.

V krokoch `capture`, `burst` alebo `stream` sa v kroku `output` predvolene nastaví na hodnotu `output`. Relatívna cesta `output` — vrátane predvoleného — sa rieši vo vzťahu k priečinku, z ktorého spúšťate `chloros-cli project run`, nie vo vzťahu k priečinku súboru receptu alebo priečinku projektu (vo verzii 1.2.0 sa nachádzal v inštalačnom priečinku backendu).

Spustite ho:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Každé spustenie sa najprv overí voči projektu.**Každá akcia musí mať ako kľúč presne jedno z vyššie uvedených slovies — `- capture: {name: FrontLeft}`, nie `- action: capture` alebo `- type: capture` — ak má projekt `name`, musí mať aj `cameras.json` / `sensors.json`, kameru alebo pole pre `apply` / `capture` / `burst` / `stream` a svetelný senzor pre `sensor`; `stream` vyžaduje, aby `count` > 0, a `wait` vyžaduje trvanie. Recept s akoukoľvek takouto chybou nevykoná**žiadnu** akciu (`[recipe] error: …`, `[recipe] not run: …`, ukončenie `1`), a to ani s `stop_on_error: false`. Kľúče, ktoré krok nepoužíva, sú `[recipe] warning:`, rovnako ako každé pole alebo svetelný senzor, ktorý recept ovláda a ktorý nie je momentálne pripojený. Recept nikdy nepripája zariadenia: najskôr spustite `chloros-cli project connect PATH`. `--dry-run` vytlačí recept, potom rovnaké chyby a varovania a `[recipe] valid: …` / `NOT valid: …`, nespustí nič a ukončí sa `1`, ak sa v recepte vyskytne chyba.

---

## Premenné prostredia

| Premenná | Účinok |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Prepíše URLu backendu (predvolené `http://127.0.0.1:5000`) — **rešpektujú ju len rodiny príkazov `lattice`, `project` a `daq pool-*`** (`lattice network-analysis` používa namiesto toho `--backend-url`). Základné príkazy (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) pripájajú `http://127.0.0.1:<port>` a ignorujú túto premennú (literál IPv4 obchádza penalizáciu systému Windows `localhost`→`::1` ~2 s na žiadosť), takže sa vždy zameriavajú na lokálny počítač. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Nastavuje sa v prostredí backendu. `1` zmierňuje odmietnutie v dôsledku prekročenia kapacity — pole spojené s `pin_resolution` alebo [dočasný príkaz pre viac kamier](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — na hlasné varovanie, pričom akceptuje stratu paketov GVSP. Iba na testovacie účely — pozri [Model fps a burst pre pole](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Nastavuje sa v prostredí backendu: `on` alebo `off` nastaví predvolené nastavenie [komprimovaný 12-bitový prenos](#packed-12-bit-transport---packed) pre každé pole (predvolené nastavenie `auto`). Explicitné nastavenie `--packed` má stále prednosť. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (alebo `true` / `yes`) vypne [automatickú kontrolu aktualizácií](#chloros-cli-update) v Linux CLI pri spustení. |
| `CHLOROS_CLI_MODE` | Nastavuje samotné CLI; nariaďuje backendu, aby povolil paralelné spracovanie. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` preskočí test náhradného riešenia GVSP (len výsledky ICMP). **Týmto sa vypne podpora jumbo paketov, nejde len o potlačenie záznamov v protokole** — kamera odpovedá na pingy DF len do veľkosti 1500 na každej trase, takže táto kontrola je jediná, ktorá dokáže detekovať jumbo pakety. Ušetrí ~1 s na kameru na jedno pripojenie; stojí ~1,45× maximálnu šírku pásma, ak by sieť *mohla* prenášať jumbo pakety. SDK vás upozorní, keď to nastavíte. |
| `CHLOROS_GVSP_SESSION_REUSE` | Nastavte v prostredí backendu: `0` meria veľkosť paketu pri každom pripojení namiesto jedného merania na kameru na jednu reláciu backendu. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fixuje veľkosť paketu GVSP na N bajtov; úplne vynecháva testovanie. Uprednostňujte nastavenie na príkaz (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) pred trvalým nastavením: fixná veľkosť sa prestane prispôsobovať sieti pred ňou a nastavenie hodnoty 9000 na trase, ktorá nepodporuje jumbo pakety, spôsobí, že **každé** zachytenie vyprší s časovým limitom pri použití `SC_ERR_TIMEOUT -1011`. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Nastavte v prostredí backendu: maximálny počet sekúnd, počas ktorých [dočasný príkaz pre viacero kamier](#multi-camera-transient-sessions) alebo `lattice capture` kamery, ktorú nič neudržiava, strávi nastavením expozície každej kamery zo scény predtým, než si uchová snímku (predvolené `5`). `0` túto funkciu vypne: automatická expozícia každej kamery sa potom spustí od prvého snímku, počnúc jej limitom expozície. |
| `CHLOROS_LOG_DIR` | Nastavuje sa v prostredí backendu: zložka, do ktorej sa ukladá podrobný protokol. Predvolené nastavenie je `%LOCALAPPDATA%\Chloros\logs` v systéme Windows a `~/.cache/chloros/logs` v systéme Linux (`$XDG_CACHE_HOME/chloros/logs`, ak je to nastavené). Koreňová služba balíka .deb nastavuje `/var/log/chloros`, ku ktorému má prístup skupina `adm` — pozri [Spustenie backendu ako služby](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Miesto, kam sa ukladajú dočasné súbory. CLI automaticky používa `/mnt/ssd/tmp`, ak existuje. |

---

## Kódy ukončenia

| Kód | Význam |
| --- | --- |
| `0` | Úspech. |
| `1` | Všeobecná chyba (väčšina chýb podpríkazov). Príkaz na zaznamenávanie tiež ukončí činnosť s kódom `1`, ak sa nepodarilo zapísať akýkoľvek súbor, o ktorého zapísanie bol požiadaný, aj keď ostatné súbory boli uložené, a s kódom `array-burst` / `array-record` ukončí program `1`, keď zaznamenávač nahlási chybu — pozri [Súbory, ktoré sa nepodarilo zapísať](#files-that-fail-to-write). `process` ukončí činnosť aj v prípade, že sa nepodarilo overiť exporty daného behu alebo došlo k strate obrazu, hoci ostatné produkty boli zapísané. |
| `2` | Chyba argumentu (neznámy alebo chýbajúci príznak; rovnako aj `time-sync set-priority` bez ohľadu na prioritu). |
| `130` | Prerušené klávesovou skratkou Ctrl+C: akýkoľvek príkaz, ktorý sa tým predčasne ukončí — `lattice`, `project`, `daq` a `process` (`process` po tom, čo backend zastavil beh, pozri [Poznámky k `process`](#notes)). Príkaz, ktorého bežným ukončením je Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` a `project stream` s `-n 0` — ukončí sa podľa toho, čo vykonal: `0` po zapísaní všetkých údajov, `1` v prípade akéhokoľvek zlyhania. |
| `143` / `129` | Ukončený signálom SIGTERM / SIGHUP (128 + signál) po vykonaní čistenia príkazu: bežiaca úloha bola zastavená a jej kamery vrátené do pôvodného stavu, spracovanie v backende bolo zastavené. `lattice continuous` alebo iný záznam, ktorého bežné zastavenie je Ctrl+C, ukončí tiež `143`, nie `0`. |

`lattice network-analysis` ukončí `1` v prípade verdiktov `needs_force_slip` a `error`, a program `selftest` ukončí program `1`, pokiaľ neprejdú všetky sedem kontrol.

---

## Tipy na riešenie problémov

- **„Vyžaduje sa prihlásenie“** → Spustite na tomto počítači raz program `chloros-cli login EMAIL PASSWORD`.
- **„Zložka projektu je pre tento počítač príliš hlboká“** → Na tomto počítači sú vypnuté dlhé cesty v systéme Windows (predvolené nastavenie systému Windows), takže žiadna cesta nesmie presiahnuť 260 znakov a produkty sa nachádzajú až približne 130 znakov pod zložkou projektu. Nič nebolo spracované. Správa uvádza najdlhšiu zložku projektu, ktorá sa zmestí: použite kratšiu (`-o` alebo `chloros-cli set-project-folder`), alebo zapnite dlhé cesty v Windows a reštartujte Chloros. Inštalátor pre Windows ponúka túto možnosť ako „Povoliť dlhé cesty v Windows“, ktorá je predvolene zaškrtnutá; inak použite Zásady skupiny: *Konfigurácia počítača > Šablóny správcu > Systém > Súborový systém > Povoliť dlhé cesty Win32*.
- **„backend unreachable“** → Spustite desktopovú aplikáciu Chloros (Windows) alebo spustite backend ručne: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` vo Windows (zložka `backend` vo vašej inštalačnej zložke; nenachádza sa v `PATH`), `chloros-backend` v systéme Linux. Alebo skontrolujte `CHLOROS_BACKEND_URL`, ak ide o vzdialený prístup.
- **Príkazy `lattice` zlyhávajú s chybou „LATTICE camera drivers not found“** → Backendu sa nepodarilo načítať runtime pre kamery LATTICE, s ktorým je nainštalovaný. Každý príkaz `lattice` beží na backende; samotné CLI neobsahuje žiadne ovládače kamery. Preinštalujte Chloros: inštalátor pre Windows a balík .deb nainštalujú runtime vedľa backendu.
- **V okne „Array connect“ / „Array Settings“ sa zobrazuje hlásenie „FRAMES WILL DROP“ alebo „Reduce ROI to enable“** → Príjemný krúžok sieťovej karty hostiteľa je príliš malý (zvyčajne sa po aktualizácii ovládača sieťovej karty resetuje na hodnotu 32). Pozrite si [Nastavenie a ladenie sieťovej karty hostiteľa](#host-nic-setup--tuning-lattice-arrays) — nastavte `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Počítač sa zasekne pri reštarte/vypnutí, potom sa nezobrazuje WMI `Invalid class` / sieťová karta sa nedá aktivovať / chýbajú USB disky** → Zastaraný ovládač USB 10GbE adaptéra spôsobuje chybu `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Aktualizujte ovládač adaptéra — pozrite si [Nastavenie a ladenie sieťovej karty hostiteľa](#host-nic-setup--tuning-lattice-arrays).
- **Systém Linux: záznamy patria používateľovi root, preto ich nemôžete odstrániť a protokol backendu sa nenachádza vo vašom `~/.cache`** → Backend beží ako root cez `chloros-backend.service`. Prepnite na `chloros-backend@<user>.service`, kde beží pod vaším používateľským účtom — pozrite si [Spustenie backendu ako služby](#running-the-backend-as-a-service-headless). Do tej doby sa protokol služby root nachádza v `/var/log/chloros` (prístupný pre skupinu `adm`).
- **Linux: jednotka backendu je `failed` s hlásením „Ďalšia inštancia už beží“ (stav ukončenia 98)** → Port už obsadzuje iný backend: naraz beží len jeden. Ak sú povolené dve backendové jednotky, ponechajte jednu: `sudo systemctl disable --now <unit>` pre tú, ktorú nepoužívate — pozrite si [Spustenie backendu ako služby](#running-the-backend-as-a-service-headless). Ak je povolená len táto jednotka, port obsadzuje backend spustený mimo systémového prostredia systemd (pomocou `chloros-backend`, `chloros-cli` alebo skriptu SDK): zastavte tento backend a potom spustite `sudo systemctl start <unit>`.
- **Upozornenie na swap na Jetsone** → Pridajte swap založený na súbore; CLI vypíše presné príkazy `fallocate` / `swapon`.
- **„backend nie je dostupný“ pri príkaze `lattice`, `project` alebo `daq`** → nebeží žiadny backend: tieto príkazy ho nikdy nespustia. Spustite príkaz `chloros-cli backend status` a pozrite si časť [Spustenie backendu](#start-the-backend).
- **„`daq net ...` nie je v Chloros k dispozícii“** (Linux a Jetson) → Niektoré vlastné nastavenia zariadenia DAQ-E sú vyhradené pre službu MAPIR. Nastavenia siete, reštarty a aktualizácie firmvéru sú `pool-set-network`, `pool-reboot` a `pool-update-firmware`; v ostatných prípadoch kontaktujte podporu MAPIR.
- **„Priame hardvérové príkazy DAQ vyžadujú kompletný balík daq“** (Windows) → Použite `pool-*`, ktorý ovláda ten istý senzor prostredníctvom backendu, alebo `chloros_sdk.connect_daq_sensor()` z Pythonu.

---

## Pozrite si tiež

- [Referenčná príručka Python SDK](sdk-reference.md) — programový ekvivalent každého príkazu CLI.
- [Svetelné senzory DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) v príručke k Chlorosu — senzory, ich limity a kalibrovaný rozsah, záznam, sieťové pripojenie DAQ-E.
- [Kamery LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) v príručke k Chlorosu — pripájanie, nastavenia kamery, režimy snímania, polia.
- Online dokumentácia: `https://mapir.gitbook.io/chloros/cli`

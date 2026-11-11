# Referenčná príručka k Chloros Python SDK

**Verzia:**

1.3.0**Vytvorené:**

29. 7. 2026 o 19:19 ·**Aktualizované:**

6. 11. 2026**Balík:** `chloros-sdk` (PyPI)**Cieľová skupina:** Optimalizované pre použitie v LLM; čitateľné pre ľudí.**Rozsah:** Všetky verejné triedy, funkcie a pomocné funkcie sprístupnené balíkom `import chloros_sdk`, vrátane príkladov, ktoré je možné skopírovať a vložiť, pokrývajúcich spracovanie obrazu, ovládanie jednej kamery, synchronizované polia, senzorov DAQ a automatizáciu projektov, ako aj server `chloros-mcp` pre asistentov umelej inteligencie.

Ak potrebujete len to najdôležitejšie, prejdite na:
- [Inštalácia a rýchly štart](#inštalácia)
- [Smart-Connect pre kamery LATTICE](#smart-connect-for-lattice-cameras)
- [Relácie senzorov DAQ](#daq-sensor-sessions)
- [Nástroje pre kamery](#camera-utilities)
- [Prechodné polia, profily zarovnania a indexy](#transient-arrays-alignment-profiles--indices)
- [Zložka projektu a recepty](#project-folder--recipes)
- [Automatizácia projektov](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Živé snímky vo vašom vlastnom pracovnom postupe](#live-frames-in-your-own-pipeline)
- [Server MCP (AI asistenti)](#mcp-server-ai-assistants)

> **Zmeny vo verzii 1.3.0** (čo zaznamená existujúci skript verzie 1.2.0).
>
> **Zmeny, s ktorými sa skript verzie 1.2.0 môže musieť vysporiadať:**
>
> - **Vydanie, ktoré backend nepotvrdí, teraz vyvolá výnimku.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` na poli, ktoré otvorila relácia, `DAQSensorSession.disconnect()` / `force_disconnect()` na senzore, ktorý otvorila, a chyby `ChlorosProject.disconnect_all()` vyvolajú chybu `ChlorosConnectError`, ak backend uvoľnenie odmietne alebo ho nepotvrdí včas, pretože zariadenie môže byť stále zablokované. Vo verzii 1.2.0 sa vracali bez upozornenia. Ukončenie bloku `with` stále nespúšťa žiadnu výnimku.
> - **Vypožičané zariadenie sa nikdy nezruší.** `with attach_array(...)` a explicitné `disconnect()` na ňom nechajú pole bežať (vo verzii 1.2.0 opustenie bloku ho zrušilo, aj keď ho vlastnilo grafické rozhranie); ak ho chcete ukončiť, zavolajte `force_disconnect()`. Podobne aj `connect_daq_sensor()`, ktorý zistil, že senzor je už v poole (vrátane inteligentnej detekcie s nulovým príznakom), ho ponechá otvorený na `disconnect()` a pri opustení bloku `with`.
> - **Formáty PNG a JPEG prijímajú iba produkty typu `display` a indexové produkty.** `capture()` s `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) a na úrovni `radiance` alebo `reflectance`, alebo `processing="all"`, vyvolá `ValueError` pred akoukoľvek požiadavkou; 12-bitový `raw` / `debayered` je backendom odmietnutý. Pozri [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` otvorí daný senzor alebo vyvolá** `ChlorosConnectError` so zoznamom toho, čo našiel; nikdy neprejde na iný senzor. Pozri [Pinned Transport](#pinned-transport).
> - **`frame_stream()` vyvolá výnimku namiesto toho, aby prešiel do tichého režimu:** `FrameFetchError` v prípade odmietnutia by v nasledujúcom takte zopakoval (čítanie odrazivosti bez akéhokoľvek DAQ), po 60 s bez rámca a po 60 s bez nového spúšťača. Pozri [Jeden spúšťač na jedno čítanie](#one-trigger-per-read--group_id).
> - **Časový limit odčítania alebo prerušenie spojenia vyvolá chybu `ChlorosConnectError`** pri každom volaní smart-connect a chybu `analyze_array_network()`, pričom verzia 1.2.0 vyvolala základný časový limit alebo chybu spojenia.
> - **`connect_array()` čaká štandardne 600 s (`timeout`, 180 s vo verzii 1.2.0) a pripojenie, ktoré sa nepodarilo nadviazať, sa zruší.** Volanie odošle svoj kód chyby `timeout` do backendu, ktorý vráti každú kameru do stavu, v akom bola nájdená, a nepripojí nič, ak sa pripojenie neuskutoční včas; volanie vyvolá kód chyby `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`). Vo verzii 1.2.0 sa mohlo stať, že pripojenie bezprostredne po zapnutí kamier pretrvalo dlhšie ako trvanie volania a pripojilo pole aj po tom, čo volanie vyvolalo výnimku. `connect_all()` čaká na každé uložené pole rovnakým spôsobom (120 s vo verzii 1.2.0). Pozri [`connect_array()` Signature](#connect_array-signature).
> - **`connect_array()` zachováva celý senzor.** Bez `width` / `height` / `binning` je každá kamera nastavená na plný snímač a snímková frekvencia klesne na úroveň, ktorú poskytuje hostiteľ a sieť; verzia 1.2.0 zmenšil alebo zlučoval snímky, aby udržal snímkovú frekvenciu. Pozri [Synchronizované pole](#synchronized-array--arraysession-smart-prep).
> - **Formát pixelov Bayer na kamere M3M (mono) predstavuje jej hĺbku.** Kamera M3M ponúka iba formáty Mono, pričom prednastavenia označujú formáty Bayer: vo verzii 1.2.0 kamera odmietla tento formát a zachovala si svoj vlastný, zatiaľ čo `settings.pixel_format` uvádzal formát Bayer. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` a polespoločný formát teraz nastavujú formát Mono s rovnakou hĺbkou na M3M: `"BayerRG8"` a `high_speed` dávajú `Mono8`; `"BayerRG12"`, `high_quality` a `triggered` dávajú `Mono12`; `cam.settings.pixel_format` ho pomenúva. To isté platí aj naopak: M3C sa nikdy neuloží vo formáte Mono, takže `"Mono12"` – z ktoréhokoľvek z týchto, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` alebo uložený formát poľa – na M3C vytvára `BayerRG12`, bez ohľadu na to, ktorá kamera je hlavnou kamerou poľa. Prenesený objekt `CameraSettings` si zachováva svoj názov, takže ten istý objekt naďalej žiada od každej kamery túto hĺbku vo svojej vlastnej skupine. `connect_camera(preset=...)` zachováva formát predvoľby (verzia 1.2.0 nastaví kameru na 12-bitový režim po aplikovaní predvoľby) a kamera otvorená na základe predvoľby už nezapíše to, čo nasnímala, do `PRESETS`. Akýkoľvek iný formát, ktorý kamera odmietne (ktorý chýba v jej zozname `PixelFormat`, napríklad `RGB8` na M3M alebo `BayerRG16` vo firmvéri bez neho), nie je tiež pomenovaný ako samostatný: po pripojení `LatticeCamera` sa `apply_settings()` alebo `set_format()`, `cam.settings.pixel_format` a `packed_transport` uvádzajú formát, v ktorom kamera vysiela, a jeden vytlačený riadok uvádza formát, ktorý odmietla.
> - **`analyze_array_network()` uvádza, aké pripojenie sa uplatňuje:** `pixel_format=None` (predtým `"BayerRG8"`) označuje formáty, ktoré pripojenie nastavuje (Mono12 / BayerRG12) a `packed="auto"` rozhodnutie o komprimovanom prenose, takže uvádzaná hodnota fps sa môže líšiť od 1.2.0&#x27;. Ak odovzdáte `pixel_format`, projektuje to M3M v danej hĺbke. Pozrite si [Network-Analysis Helper](#network-analysis-helper).
> - **`CameraSession.capture()` vráti `CaptureResult`** (stále ide o zoznam slovníkov rámcov) s hodnotami `.skipped`, `.failed`, `.daq_files` a `.daq_matches`. Výsledok `ArraySession.capture()` obsahuje tiež `.alignment` (či bolo použité zarovnanie a prečo nie) a `.oriented`.
> - **Zachytenia polí sú štandardne zarovnané, rovnako ako v aplikácii.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` a `ArrayHandle.capture()` / `capture_smart()` deformujú súbory `debayered`, `display`, `radiance`, `reflectance` a indexové súbory do snímky hlavnej kamery a oreže ich na prekrývanie členov (menšie ako senzor o posuny kamier, `XMP-mapir:AlignmentJSON` `"applied": true`), takže sú to rovnaké súbory ako záznam tej istej scény v aplikácii Chloros; `raw` zostáva v pôvodnej veľkosti. Predtým sa pri snímaní bez `aligned=True` ukladali všetky súbory v pôvodnej veľkosti, bez korekcie skreslenia. `aligned=False` to zachováva. `chloros-mcp``capture_array` a `start_capture_run` používajú rovnaké predvolené nastavenie. Pozri [Metódy `ArraySession`](#arraysession-methods).
> - **`connect_all()` označuje backend, ktorý odmieta SDK**. Spustený backend, ktorý je odhlásený alebo má tarifu nižšiu ako Copper, sa hlási pod kódom `errors` s vlastným dôvodom (HTTP 401 / 403), nie ako „nebeží žiadny backend Chloros“, a každé volanie `connect_all()` opäť skontroluje backend, takže jeho volanie po prihlásení funguje na tom istom projektovom objekte.
> - **Relatívna cesta `output_dir` funguje s lokálnym backendom.** Relatívne cesty sa pred odoslaním premenia na absolútne voči pracovnému adresáru vášho skriptu. Vo verzii 1.2.0 ich backend riešil vo vlastnej inštalačnej zložke, čo viedlo k chybe „Prístup je zamietnutý“.
> - **`numpy` je teraz požiadavkou** SDK (volania live-frame vracajú numpy polia).
> - **Názov úrovne exportu, ktorý nie je „one“, je odmietnutý.** `processing=` a `levels=` prijímajú `raw`, `debayered`, `radiance`, `reflectance` alebo `display` (alebo alias, ako napríklad `preview`; `processing=` a tiež `"all"`). Akýkoľvek iný názov je odmietnutý ešte pred dotykom s kamerou (HTTP 400 `UNKNOWN_LEVEL`: `capture()` vyvolá `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); verzia 1.2.0 pre to uložila alebo vrátila `raw`.
> - **Funkcia Texture Aware je pre snímky LATTICE odmietnutá.** V projekte obsahujúcom snímky LATTICE vyvolá `configure(debayer="Texture Aware (Slow, Highest Quality)")` výnimku `ChlorosConfigurationError`, rovnako ako `process()` po výnimke `configure()`, ktorá o to pred importom. Hodnota Texture Aware, ktorú projekt uložil skôr, je nastavená na Standard, s chybou `UserWarning`. Snímky LATTICE vždy používajú štandardnú demosaiku, takže verzia 1.2.0 toto nastavenie akceptovala a ignorovala; aplikácia Chloros ho pre ne tiež neponúka. Pozri [Podporované hodnoty](#supported-values).
> - **Kamery sa vrátia do stavu, v akom boli nájdené — pokiaľ nie je nastavené `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, na konci každej prechodnej funkcie (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) a zastavenie funkcie `start_camera_continuous()` vrátia každú kameru do pôvodného stavu: jej zapojenie spúšte a synchronizačnej linky, jej PTP a všetky nastavenia kamery, ktoré sa počas relácie zmenili (expozícia a zosilnenie, cieľ automatického nastavenia expozície (AE), limity a tlmenie, vyváženie bielej, úroveň čiernej, formát pixelov, zlučovanie a oblasť záujmu (ROI), otočenie, snímková frekvencia). `keep_settings=True` naopak uloží nastavenia každej kamery do jej vlastnej pamäte, aby sa s nimi spustila; `clear_settings=True` túto pamäť obnoví na továrenské nastavenia. V oboch prípadoch kamery zostávajú v režime voľného chodu. Projekt nastaví svoje kamery podľa vlastných uložených nastavení, keď sa k nim pripojí. Pozri [Pamäť kamery](#camera-memory--keep_settings--clear_settings).
> - **Každý režim ukladá projekt rovnakým spôsobom.** Akákoľvek zmena na kamere, poli alebo svetelnom senzore pripojenom v rámci projektu — či už z SDK, aplikácie, CLI alebo servera MCP — sa uloží do súboru `cameras.json` / `sensors.json`, a otvorenie projektu v akomkoľvek režime nastaví zariadenia podľa neho; `ChlorosProject` pomenuje svoj vlastný priečinok v backende. Predtým projekt ukladala len aplikácia. Pozri [Jeden záznam projektu, každý režim](#one-project-record-every-mode).
> - **Súbory vytvorené z viacerých kamier uvádzajú názvy všetkých kamier.** Viacpásmový stack, živý index vytvorený z pásiem viacerých kamier a obraz z hromadného snímania `composite/` obsahujú sériové číslo, model a kalibračný balík každej zdrojovej kamery vo forme zoznamov (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), pričom EXIF `SerialNumber` uvádza sériové čísla oddelené čiarkami. Predtým neobsahovali žiadne informácie o fotoaparátoch alebo obsahovali len sériové číslo prvého fotoaparátu. Pozri [Súbory vytvorené z viacerých kamier](#files-made-from-several-cameras).
> - **Iba súbor `raw` obsahuje kalibračné údaje kamery.** Sada kompatibilná s MicaSense (`BlackLevel`, `RadiometricCalibration`, súbory prekanálový `Band{R,G,B}RadiometricCalibration`, súbory `MicaSenseVignetting*`) a dátová časť `CalibrationJSON` opisujú surové hodnoty senzora, takže sú priložené iba k súboru záznamu `raw`. Všetky ostatné súbory — súbory záznamov`debayered`, `display`, `radiance`, `reflectance` a indexové súbory, snímky z nepretržitého záznamu a z viacerých kamiera každý produkt `process()` — obsahuje identifikátor kamery (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) a pri snímaní aj vlastný záznam snímky (`XMP-mapir:FrameJSON`: expozícia, zosilnenie, teplota, formát pixelov, čas snímania). Vo verzii 1.2.0 obsahovali aj súbor surových údajov, ktorý nástroj na ich spracovanie (Pix4D, Metashape) aplikoval na pixely, ktoré nie sú surovými hodnotami. Spracovanie zistí kalibráciu takéhoto súboru podľa jeho sériového čísla a `CalBundleSha256`.
>
> **Ďalšie zmeny:**
>
> - **Automatická snímková frekvencia poľa sa prispôsobuje tomu, čo pole poskytuje.** Frekvencia, ktorá stratí viac ako 1 % svojich spúšťacích impulzov, sa zníži, pričom sa posudzuje na základe viacerých dlhších pozorovaní namiesto jedného, a frekvencia, z ktorej bola znížená, sa občas opäť vyskúša. Frekvencia, ktorú nastavíte sami, zostane nezmenená.
> - **Automatická expozícia je stabilnejšia.** Pri konštantnom osvetlení zostáva nezmenená, po zmene sa ustáli v priebehu niekoľkých sekúnd, ignoruje malé preosvetlené oblasti a obnoví funkčnosť fotoaparátu, ktorý prestal reagovať v jasnom slnku. Pozrite si [Ako funguje automatická expozícia](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` počíta iba vaše snímky a nahrávky.** V priečinku obsahujúcom súbory `camera_cal/` projektu sa tieto kalibračné súbory už nepočítajú ako snímky a nahrávky a projekt, ktorý ich už prijal, ich vyradí.
> - **`configure(radiometric_output=...)` sa uplatňuje.** Teraz nastavuje produkty projektu `Export radiance` / `Export reflectance`; predtým mali prednosť vlastné nastavenia produktov každého projektu, takže `radiance`, `sensor-response` a `all` všetky exportovali žiarivosť a odrazivosť.
> - **`test_array_sync()` prechádza zo stavu zapnutia kamier.** Zapne PTP kamier na dobu behu, počká, kým sa ich hodiny synchronizujú a ustália pred prvým spustením (`ptp_lock`) — každá kamera je synchronizovaná po dobu 120 s, približne 2 minúty od zapnutia (`ptp_settle_s=` to nastaví), keďže práve synchronizované hodiny majú odchýlku až ~200 us po dobu približne 2 minút — a vráti PTP každej kamery do stavu, v akom ho našiel. Na kameru, ktorá je stále synchronizovaná z predchádzajúceho testu, sa už nečaká. Predtým nechával PTP vypnutý v sieti s jedným prepínačom, kde žiadny DAQ-E neodpovedal, a čítal voľne bežiace hodiny každej kamery ako PTP hodiny, ktoré sa nesynchronizovali („PTP nie je synchronizované“, pri každom behu).
> - **Záznam `.daq` uvádza, či bol jeho kryt zvolený alebo predpokladaný.** DAQ-U, DAQ-M a DAQ-E sa dodávajú bez krytu a Chloros nedokáže zistiť, či je kryt nasadený, takže senzor pripojený bez pomenovaného krytu dostane predvolene `sunshine_cosine`, čo je *predpoklad*. Každý záznam Chloros (schéma 1.30) teraz ukladá `cap_id_source` vedľa `cap_id`: `operator`, ak ste kryt pomenovali (vrátane `none`), `auto_default`, ak bol predvolený. Pri spracovaní záznamu, pri ktorom bol kondenzátor predpokladaný, pričom nastavenie kondenzátora DAQ v projekte zostalo na Auto, sa vytlačí `[daq-dls] WARNING` s názvom súboru: samotný DAQ-U bez korekcie slnečného žiarenia vykazuje približne 30-násobne vyššiu hodnotu. **Pomenujte kondenzátor, ktorý ste namontovali** pri pripojení (`connect_daq_sensor(cap_id=...)` alebo `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`, alebo z roletového menu kondenzátorov v grafickom rozhraní), alebo nastavte kondenzátor DAQ v projekte a varovanie zmizne.
> - **V systéme Windows sa už nezobrazujú okná konzoly pip.** Na počítači s nainštalovaným Pythonom spúšťali backend a každý z jeho obrazových pracovníkov pri štarte raz program `pip` a každý z nich otvoril okno konzoly. Teraz sa to už nedeje.
> - **Živé záznamy `.daq` prežijú výpadok napájania.**Záznam sa zapisuje s denníkom a úplnou synchronizáciou disku pri každom potvrdení, za senzorom, takže pomalý disk spomaľuje len súbor, nikdy však namerané hodnoty. Záznam, ktorý sa zastavil uprostred potvrdenia (výpadok napájania, ukončenie), môže vedľa seba zanechať súbor `<name>.daq-journal`:**skopírujte oba súbory spolu.**Chloros pri prvom načítaní súboru vráti denník späť, takže sa záznam otvorí so všetkým, čo bolo potvrdené pred zastavením (stratia sa približne posledné 3 s); vo verzii 1.2.0 mohol výpadok napájania v nevhodnom okamihu spôsobiť, že celý súbor bude nečitateľný („obraz disku databázy je poškodený“).
> - **`DAQSensorSession.record_start()` na vyťaženom disku už nezastavuje senzor**: merania sa zaradia do fronty a zapisujú sa vo vlastnom vlákne, a to, ktoré sa nepodarilo zapísať, sa započítava do `status()` v rámci `record_errors` (spolu s `record_last_error`).
> - **Priamy `LatticeCamera`:**snímka bez nastavenia expozície/zisku v bloku teraz hlási**skutočnú** expozíciu a zisk kamery (záložné riešenie čítalo hodnotu z vyrovnávacej pamäte, ktorá mohla zostať nemenná počas celého cyklu automatickej expozície), a `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` uplatňujú rovnaké 5 µs odstránenie šumu na vstupe, aké dostávajú káblové podriadené kamery Chloros (nefiltrované, podriadená kamera spustila snímky bez odoslania spúšťača).
> - **Kamery spúšťané softvérom už nevynechávajú každý druhý spúšťací signál.** Kamera v režime softvérového spúšťania teraz zaradí do fronty spúšťací signál, ktorý dorazí počas jej čítania (alebo Chloros počká, kým bude pripravená), tak ako to už robili kamery synchronizované káblom.
> - **`discover_daq_sensors()` označuje DAQ-E-S ako `DAQ-E-S`.** Jeho `model` bol `DAQ-E` pre každý ethernetový senzor; ten, ktorý neodpovie na kontrolu identity do približne 2 s, je stále uvedený v zozname ako `DAQ-E`.
> - **Novinka: `chloros-mcp`, server MCP pre asistentov umelej inteligencie.** Claude Desktop, Claude Code, Cursor a ďalšie MCP klienty dokážu spracovávať snímky (vytvoriť projekt, importovať snímky, vybrať nastavenia, spracovať, vygenerovať správu o výsledkoch spracovania) a ovládať kamery LATTICE, senzorové polia a svetelné senzory DAQ: pripojiť, zobraziť náhľad, nastaviť expozíciu a farby, zachytiť, zaznamenať. Jeho nástroje pokrývajú funkcie tohto SDK. Inštaluje sa spolu s SDK a balík .deb pre Linux ho umiestni do adresára `/usr/bin`. Pozrite si [MCP Server (AI asistenti)](#mcp-server-ai-assistants).
> - **Novinka: nástroje pre kamery**, príkazy v adresári `chloros-cli lattice`, ktoré SDK volá: farebný profil, sýtosť / kontrast, najsvetlejšie oblasti, vyváženie bielej (jednorazové, automatické, pevné pomery), prednastavenia, modely filtrov, stav a aktualizácia firmvéru, napájanie (test, spánok, prebudenie), kalibrácia cieľa, sieťové adaptéry (kontrola, odhad, oprava), korekcia vinetácie v súboroch, čítanie DLS a test hostiteľa. Pozri [Nástroje pre kameru](#camera-utilities).
> - **Novinka: zvyšok CLI ako volania SDK**, takže CLI, SDK a `chloros-mcp` ponúkajú rovnaké funkcie (CLI si z bezpečnostných dôvodov zachováva iba prihlásenie, konfiguráciu zariadenia DAQ-E a priame servery DAQ): kamery otvorené jedným príkazom (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), súbory profilov zarovnania (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), indexy (`index_catalog()`, `compute_index()`, `capture_live_index()`), hlavný súbor PTP (`time_sync_status()` a súvisiace súbory), zložku projektu (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) a recepty (`run_project_recipe()`, vlastný spúšťač CLI). Pre svetelné senzory: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` a `light_sensor_reflectance()` (odrazivosť a indexy okolitého prostredia a objektov pre dva senzory); pre kameru: `CameraSession.status()` a `capture_repeated()`; pre pole: `ArraySession.capability()` a `set_member_settings()` (expozícia, zosilnenie člena poľa, ...). Pozrite si [Prechodné polia, profily zarovnania a indexy](#transient-arrays-alignment-profiles--indices), [Projektová zložka a recepty](#project-folder--recipes) a [Sessions snímačov DAQ](#daq-sensor-sessions).
> - **Vylepšené indexové produkty LATTICE** (NDVI a ďalšie indexy) z `process()` vo všetkých režimoch spracovania.
> - **Kamery LATTICE OCN: vylepšené produkty žiarivosti, odrazivosti a indexov pri slnečnom svetle.** Po rozložení pásiem sa aplikuje korekcia flotily — v reálnom čase, pri snímkach a v `process()`; kamery RGN zostávajú bez zmien. Pozrite si [Radiometrický výstup](#radiometric-output-lattice-multispectral-pipeline).
> - **Každý záznam ukladá hodnotu DAQ, ktorú použil, rovnako ako grafické používateľské rozhranie.** `CameraSession.capture()` a `ArraySession.capture()` / `continuous()` zapisujú súbor `.daq` a jeho `.match.json` vždy, keď je DAQ viazaný alebo zaradený do skupiny, vrátane záznamu len v surovom formáte; vo verzii 1.2.0 sa zachytávanie v surovom formáte alebo po odstránení Bayerovho filtra ukladalo iba s `force_daq=True`. `daq=False` túto funkciu vypína a `daq_tolerance_ms=` nastavuje toleranciu zhody; oba výsledky uvádzajú, čo bolo uložené v `.daq_files` a `.daq_matches`. Zachytenie teraz čaká na ďalšie odčítanie z DAQ, aby mohlo zlúčiť hodnoty z oboch strán expozície. Pozri [Odčítanie z DAQ, ktoré zachytenie ukladá](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
> - **`ArraySession` obsahuje živé snímky a nepretržité snímanie na strane servera.** `grab()` a `frame_stream()` vracajú snímky typu numpy z jedného spúšťača (rovnaké čítanie ako v prípade projektového API `ArrayHandle.grab()` / `frame_stream()`), a `continuous()` ukladá každú synchronizovanú skupinu v backende a vráti [`ContinuousHandle`](#continuoushandle). Pozrite si [Živé rámce vo vašom vlastnom potrubí](#live-frames-in-your-own-pipeline).
> - **Pole `grab()` je priradené k jednému spúšťaču** (`ArrayHandle.grab()` a nový `ArraySession.grab()`). Vo verzii 1.2.0 12 z 20 štyrochkamier `ArrayHandle.grab()` prekrývalo dva spúšťače. Každý snímok teraz obsahuje `group_id` a pole `frame_stream()` poskytuje každý spúšťač raz. Pozri [Jeden spúšťací signál na jedno čítanie](#one-trigger-per-read--group_id).
> - **Snímky podľa pásma a zarovnané snímky:** `grab()` / `frame_stream()` prijímajú `band=`, `align=`, `bit_depth=` a `refl_max=` na identifikátory kamery a poľa. Pozrite si [Export jedného pásma](#exporting-a-single-band--band).
> - **Nové metódy a argumenty `ArraySession`:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (spolu s `CaptureResult.group_seq` a `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` spája po sebe idúce skupiny a zastaví sa pri plnom disku.
> - **`ArraySession.record()` zaznamenáva bez otvoreného grafického rozhrania.** Ak je kombinovaný pohľad grafického rozhrania zatvorený, backend vykresľuje pohľad sám: poleNDVI alebo index, ktorý odovzdáte ako `expr=`. `RecorderHandle.stop()` vyvolá výnimku `ChlorosConnectError` v prípade neúspešného záznamu, pri ktorom nebol zapísaný žiadny snímok, a zachová výsledok zastavenia v `.result` (zanechanie bloku `with` nevyvolá výnimku: skontrolujte `rec.result`). Pozrite si [`RecorderHandle`](#recorderhandle).
> - **Nové metódy `DAQSensorSession`:** `enable_ptp()`, `force_disconnect()` a pre vlastné nastavenia zariadenia DAQ-E `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` a `update_firmware()`. `record_stop()` vráti `warning` a `gaps`. Reštart, ktorý ukončil nahrávanie, ju v odpovedi pomenuje, alebo, ak reštart zlyhá, v položke `ChlorosConnectError` (`recording_closed`; pozri [Nastavenia zariadenia DAQ-E](#daq-e-device-settings)). Pozri [Metódy `DAQSensorSession`](#daqsensorsession-methods).
> - **Zoskupené zariadenie DAQ-E, ktoré prestane odosielať dáta, je opätovne pripojené backendom** s pôvodnými nastaveniami a záznam na ňom pokračuje v tom istom `.daq` s medzerou označenou v súbore; `record_stop()` vráti medzery ako `gaps`. Vo verzii 1.2.0 senzor zostal neaktívny, kým nebol odpojený a znovu pripojený, čo ukončilo záznam. Pozrite si v referenčnom materiáli k CLI [DAQ-E, ktorý prestane odosielať údaje](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` funguje na základe nainštalovaného SDK.** Inštalačný balík (inštalátora PyPI) teraz obsahuje potrebné načítače projektov; predtým `open_project()` vyvolával chybu `ImportError` s názvom `project_cameras` / `project_sensors` v akomkoľvek SDK nainštalovanom cez pip. `connect_all()` pripája uložené kamery, polia a svetelné senzory prostredníctvom bežiaceho backendu (svetelné senzory prostredníctvom svojho DAQ poola) a uložené zariadenie, ktoré nemá ako vyhľadať, je uvedené pod položkou `errors` s uvedením dôvodu namiesto toho, aby bolo bez vysvetlenia preskočené. Uložené pole, ktoré je už spustené v backende (v grafickom rozhraní, alebo v `chloros-cli project connect`) je pripojené a ponechané v prevádzke, zatiaľ čo predtým to bolo hlásené ako chyba („Kamera  <sn>je už v poli<id>“). Pozri [Metódy `ChlorosProject`](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` a `process_lattice_capture()` vyhľadávajú Chloros v systéme Windows.** Hľadajú miesto, kam inštalátor umiestňuje backend (štandardne `<install folder>\backend\`, `C:\Program Files\Chloros\backend\`, vlastný priečinok podľa záznamu samotného inštalátora) a pripoja sa k už bežiacemu backendu bez toho, aby ho vôbec hľadali. Vo verzii 1.2.0 vyhodili na nainštalovanom počítači chybu „Chloros backend not found“, pokiaľ ste nepreviedli `backend_exe=`.
> - **Backend, ktorý sa zrúti pri spustení, je ihneď nahlásený.** Keď volanie `ChlorosLocal()` alebo smart-connect spustí backend, ktorý sa ukončí skôr, ako odpovie, volanie vyvolá v priebehu niekoľkých sekúnd chybu `ChlorosBackendError` z volaní `ChlorosLocal` a `ChlorosConnectError` z volaní smart--connect, spolu s kódom ukončenia, príkazom, ktorý spustil, a poslednými riadkami, ktoré backend vypísal. Verzia 1.2.0 vyčkala 60-sekundový časový limit na spustenie a neuviedla dôvod. Pozri [Automatické spustenie backendu](#backend-auto-start).
> - **Backend beží aj medzi spúšťaním skriptov.** Po ukončení bloku `with ChlorosLocal(...)` návratom `process_folder()` / `process_lattice_capture()` a ukončením `chloros-mcp` už nezastaví backend, ktorý spustilo SDK; nasledujúci skript ho znovu použije. Zastaví sa pri `logout()` (pokiaľ nie je otvorená aplikácia Chloros alebo prebieha spustenie), pri `shutdown_backend()` alebo sám po 30 minútach, ak ho nič nevyužíva (`CHLOROS_BACKEND_IDLE_MIN` v prostredí backendu to mení; `0`: nikdy).
> - **Backend spustený prostredníctvom SDK je požiadaný o ukončenie, nie je násilne ukončený.** `shutdown_backend()` umožňuje backendu uvoľniť svoje kamery a svetelné senzory a dokončiť akékoľvek nahrávanie; je ukončený len vtedy, ak stále beží o 10 s neskôr. Verzia 1.2.0 ho ukončila okamžite (v systéme Windows išlo o násilné ukončenie), čo mohlo spôsobiť, že svetelný senzor DAQ pripojený k tomuto backendu prestal reagovať. Backend, ktorý SDK zistilo, že beží, sa naďalej ponechá bez zásahu.
> - **V systéme Linux sa pri automatickom spustení nikdy nespustí `backend_server.py` z aktuálneho priečinka.** Ak nebol nainštalovaný žiadny backend, skript spustený v zložke, v ktorej sa nachádzal, ho spustil; teraz sa spúšťa iba nainštalovaný backend (alebo, v prípade `ChlorosLocal`, ten, ktorý `backend_exe=` určí).
> - **`configure(indices=...)`**pridáva každé prednastavenie do projektu pre pásma príslušnej kamery, vrátane nového projektu, a varuje (`UserWarning`) v prípade názvu, ktorý nemôže pridať.**`import_images()`** hlási záznamy `.daq` ako `scan_count` a už nevydáva varovanie týkajúce sa zložky na ukladanie záznamov LATTICE.
> - **`read_image_audit_tags()` funguje po inštalácii cez pip.** Sám číta metadáta obrázku; vo verzii 1.2.0 vracal `{}` bez interných súčastí desktopovej inštalácie v ceste k Pythonu.
> - **`capture()` hneď po `connect_array()`** čaká až 20 sekúnd na prvú skupinu snímok poľa namiesto toho, aby vyhodil chybu HTTP 503 „Pole sa ešte zahrieva“.
> - **`status()` hlási zastavenie.** Nové pole `stalled` má hodnotu True počas behu cyklu zachytávania, ale nezverejňuje nič, a `actual_fps` má potom hodnotu 0. Vo verzii 1.2.0 si `actual_fps` zachovalo svoju poslednú hodnotu počas zablokovania, takže zablokované pole zobrazovalo hodnotu `actual_fps > 0`, zatiaľ čo zachytávanie zlyhalo s chybou 503 „No fresh synced frame group“.
> - **Menej neúplných sád snímok a chýb zachytávania 503:** spúšťací vstup každej podriadenej kamery je teraz zbavený rušivých impulzov.
> - **Komprimovaný 12-bitový prenos** (`connect_array(packed="auto")`) zvyšuje snímkovú frekvenciu poľa s plným snímačom približne o tretinu. Pozri [Komprimovaný 12-bitový prenos](#packed-12-bit-transport-packed).
> - **Expozícia poľa v rámci oblasti záujmu (ROI):** malá oblasť záujmu (ROI) už neobmedzuje expozíciu člena poľa na dobu voľného chodu snímača (4,9 ms pri rozlíšení 768 × 304). Pozrite si [Expozícia členov poľa](#exposure-on-array-members).
> - **Automatická expozícia už neznižuje automatickú snímkovú frekvenciu poľa.** Pole synchronizované káblom beží rýchlosťou, ktorú umožňuje sieť a čítanie kamier; automatická expozícia vyplní obdobie, ktoré táto rýchlosť poskytuje, a potom zvýši zosilnenie. Ručná expozícia dlhšia ako toto obdobie spomalí spúšť, aby sa jej prispôsobila, namiesto toho, aby bola odmietnutá. Pozri [Expozícia na prvkoch poľa](#exposure-on-array-members).
> - **`read_nodes()`**pridáva `frame_values` a `notes` počas behu automatického nastavenia expozície (pozri [`read_nodes()` pri automatickom nastavení expozície](#read_nodes-under-auto-exposure--frame_values-and-notes)). Keď kamera odmietne expozíciu alebo zosilnenie,**`set_settings()`** oznámi rozsah, ktorý akceptuje, a obnoví automatický režim, ktorý bol pre danú hodnotu vypnutý; `set_settings(target_brightness_auto=True)` je nový.
> - **`orient=`** na `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` a `ArrayHandle.capture()` / `capture_smart()` otočí produkty podľa orientácie kamery. Pozrite si [Metódy `ArraySession`](#arraysession-methods).
> - **`DLS()` vyžaduje vlastnú hodnotu `mac_address` senzora.** Už neexistuje predvolená adresa MAC (bola to adresa jednej jednotky); `DLS()` bez nej vyvolá chybu `DLSError`. Pozrite si [veľkosti pásiem DAQ-M](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **Inštalačný program pre Windows** ponúka možnosti „Povoliť dlhé cesty vo Windows“ a „Vylúčiť backend Chloros z kontroly Microsoft Defender“, ktoré sú štandardne zaškrtnuté. Pozri [Krok 1](#step-1--install-the-chloros-platform-package).
> - **Linuxový balík .deb** nainštaluje Pythonové SDK pre systém `python3` (predtým sa na disk umiestňoval len balík wheel), zvyšuje veľkosť vyrovnávacích pamätí prijímania socketov jadra, ktoré potrebujú prúdy z kamery, a pridáva backendovú službu, ktorá beží pod vaším používateľským účtom. Balík pre Jetson je `chloros_1.3.0_arm64_jp6.deb`, určený pre JetPack 6, s CUDA PyTorchom pre JetPack. Pozrite si [Krok 1](#step-1--install-the-chloros-platform-package) a [Krok 2](#linux-deb).

---

## Architektúra za 60 sekúnd

SDK je tenká vrstva v jazyku Python nad backendom Chloros (rovnaký server Flask, aký používajú grafické rozhranie pre stolné počítače aj CLI). Na automatizáciu importujete `chloros_sdk` a voláte metódy vyššej úrovne; v pozadí sa sa každé volanie premení na HTTP požiadavku na lokálny backend na porte 5000 — `http://127.0.0.1:5000/api/...` (úmyselne nie `localhost`, ktorý sa v systéme Windows najskôr premenuje na `::1` a pri backende podporujúcom iba IPv4 trvá spracovanie jednej požiadavky približne 2 s). Backend spravuje hardvérový fond — kamery, senzory DAQ, profily zarovnania, rámcové vyrovnávacie pamäte — takže skripty SDK môžu koexistovať s grafickým rozhraním bez toho, aby sa museli deliť o sériové porty alebo šírku pásma sieťovej karty.

K dispozícii máte tri rozhrania:

1. **`ChlorosLocal` + voľné funkcie** (`process_folder`, `process_lattice_capture`) — Spracovateľský reťazec obrazu. Spustite spracovanie celého priečinka vrátane kalibrácie, odstránenia Bayerovho filtra a exportu indexu pomocou jedného volania v jazyku Python.
2. **Ovládacie prvky Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Otvorenie trvalej relácie backendu pre živý hardvér. Rovnaký postup „smart-prep“ ako v grafickom rozhraní: sieťová sonda, automatický výber vrstvy, PTP, inicializácia AE, konfigurácia spúšťača GPIO.
3. **`ChlorosProject` / `open_project`** — Načítajte uložený projekt (zložka s `cameras.json` + `sensors.json` + `project.json`), naraz pripojte jeho kamery, polia a svetelné senzory a spúšťajte snímky pomocou pomenovaných identifikátorov.

Povrchy 1 a 2 **automaticky spustia lokálny backend**, ak ešte žiadny nepočúva (ten istý balíkový binárny súbor, ktorý spúšťa GUI/CLI) — takže holý skript funguje z nového shellu bez toho, aby ste museli najprv spustiť backend: `ChlorosLocal` (a `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` a `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` a `analyze_array_network()` sa nikdy nespustia: potrebujú bežiaci backend. Odovzdajte `auto_start_backend=False`, ak sa chcete odhlásiť (napr. pri odkazovaní na vzdialený backend, ktorý sa nikdy nespustí). Pozri [Automatické spustenie backendu](#backend-auto-start). Surface 3 sa správa odlišne: `open_project()` neprijíma žiadny parameter `auto_start_backend` a `connect_all()` nikdy nespustí backend — raz skontroluje `http://127.0.0.1:5000` a ak nič neodpovie, prejde na priame (bez backendu) ovládanie zariadenia `lattice_sdk`. To vyžaduje runtime SDKu Arena a OpenCV (`cv2`), ktoré je možné importovať do vášho Pythonu, čo však balík SDK wheel neinštaluje; bez nich sa každá uložená kamera a pole nahlásia ako chyba spolu s dôvodom. Svetelné senzory projektu nemajú možnosť prevádzky bez backendu: ak nebeží žiadny backend, každý uložený senzor sa nahlási ako chyba spolu s dôvodom (pozri [`SensorHandle`](#sensorhandle)). Iba `proj.process()` a `stream(..., overlays=True)` odložene vytvoria objekt `ChlorosLocal()` (ktorý sa spúšťa automaticky).

Všetky tri vyžadujú autorizáciu– vyžadujú autorizáciu: spustite `chloros-cli login` raz na danom počítači alebo sa prihláste cez grafické rozhranie na ploche. Čo vyvolá volanie bez platnej relácie, závisí od rozhrania (pozri [Výnimky](#exceptions)): `ChlorosAuthenticationError` z `ChlorosLocal` (a `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` z volaní smart-connect a ich relácií, `status: "error"` s výnimkou `error_code` z `analyze_array_network()` a záznam pod `errors` v správe `connect_all()`.

Požiadavky:
- Python 3.7+ (ako uvádza balík; vyvinuté/testované na verzii 3.10)
- Lokálne nainštalovaný Chloros Desktop (binárny súbor backendu je súčasťou inštalátora)
- Aktívne prihlásenie do Chloros+ na **ľubovoľnej platenej úrovni Chloros+ (Copper alebo vyššej)**; bezplatná úroveň**Iron**nemá prístup k SDK ani CLI. Toto pravidlo sa vynucuje**na strane servera**:** každá požiadavka označená SDK/CLI musí obsahovať aktívnu reláciu aj platený plán. Plán nižší ako Copper dostane chybu `403` spolu s `error_code: PLAN_UPGRADE_REQUIRED` (`ChlorosLicenseError` z volaní `ChlorosLocal`, `ChlorosConnectError` z volaní smart-connect). Odhlásený volajúci dostane namiesto toho kód `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError` z `ChlorosLocal`, `ChlorosConnectError` z hovorov cez Smart Connect) — tieto dva sú odlišné, pretože opätovné spustenie `chloros-cli login` odstráni chybu `401`, ale nedokáže odstrániť chybu `403`, ktorá si vyžaduje aktualizáciu plánu.
- Offline používanie je podporované počas tolerančného obdobia plánu: úroveň sa číta z vyrovnávacej pamäte overovania serverom (5 minút) alebo z vyrovnávacej pamäte podpísanej licencie viazanej na zariadenie (30 dní pre mesačné plány, do vypršania predplatného pre ročné plány). Po uplynutí tohto obdobia sa plán zmení na bezplatný a prístup k SDK/CLI sa pozastaví, kým sa zariadenie aspoň raz nepripojí k serveru. Chyba `chloros-cli status` (`GET /api/license-status`) zostáva dostupná v bezplatnej úrovni, takže je viditeľný dôvod.
- Windows 10/11 64-bit, **Ubuntu 22.04 LTS alebo novší**, alebo Jetson (JetPack 6). Ubuntu 20.04**nie je** podporovaný: závislosti `.deb` sú odvodené od toho, na čo sa backend viaže, vrátane `libc6 (>= 2.35)`, a Focal dodáva glibc 2.31.

---

## Inštalácia

Python SDK je tenká Pythonová vrstva nad backendom Chloros. Pokiaľ neprevádzkujete backend na inom počítači (pozri [Režim vzdialeného backendu](#remote-backend-mode-pip-only-host-via-tunnel)), potrebujete **lokálne nainštalovaný balík Chloros pre desktop** (inštalátor pre Windows alebo Linux `.deb`), ktorý obsahuje senzory DAQ light. Balík obsahuje binárny súbor backendu, runtime SDKu Arena pre kamery LATTICE a kalibračné balíčky.

Najnovšie súbory na stiahnutie: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Krok 1 — Inštalácia balíka platformy Chloros

#### Windows (.exe)

1. Stiahnite si súbor `Chloros Installer 1.3.0.exe` zo stránky na stiahnutie.
2. Spustite inštalačný program a postupujte podľa pokynov sprievodcu. Predvolená inštalačná cesta je `C:\Program Files\Chloros\`.
3. Spustite Chloros aspoň raz a prihláste sa pomocou svojho účtu Chloros+.

Inštalačný program povolí cez Windows Firewall v každom sieťovom profile prístup pre backend (`chloros-backend.exe`, nič iné) cez bránu Windows Firewall vo všetkých sieťových profiloch: prichádzajúci UDP pre prúdy z kamier, PTP, mDNS a prúd DAQ-E, a prichádzajúci TCP iba z lokálnej podsiete pre pripojenie, ktoré DAQ-E nadväzuje späť na počítač počas aktualizácie firmvéru ([`update_firmware()`](#daq-e-device-settings)). Odinštalátor odstráni obe pravidlá. HTTP API backendu zostáva v každom prípade na adrese `127.0.0.1`.

Skupina **Nastavenia systému** v sprievodcovi obsahuje dve možnosti, ktoré sú štandardne zaškrtnuté:

| Možnosť | Čo robí | Vypnutie tichej inštalácie |
| --- | --- | --- |
| **Povoliť dlhé cesty v systéme Windows** | Umožňuje, aby boli projektové priečinky umiestnené hlbšie, než je limit 260 znakov pre cesty v systéme Windows. Ak sú dlhé cesty vypnuté (predvolené nastavenie systému Windows), projektový priečinok, ktorý je pre dané produkty umiestnený príliš hlboko, bude odmietnutý ešte pred spracovaním čohokoľvek — pozri [Riešenie problémov](#troubleshooting-pointers). | `/MERGETASKS="!longpaths"` |
| **Vylúčiť backend Chloros z kontroly programom Microsoft Defender** | Zastaví skenovanie v reálnomskenovanie každého súboru, ktorý zapíše zachytený obraz z kamery. Pri vysokej rýchlosti zachytávania skenovanie spomaľuje rýchlosť o približne 30 %: pri meraní s ROI 512 na sústave 9 kamier dosahovalo nepretržité zachytávanie surových dát rýchlosť 52,7 snímok/s so skenovaním a 77,7 snímok/s s vylúčeným backendom. | `/MERGETASKS="!defenderexclusion"` |

Neprerušované snímanie hlási zaťaženie programu Defender ako `defender_cores` (pozri [`ContinuousHandle`](#continuoushandle)).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Inštalujte pomocou `apt install ./…`, nie `dpkg -i`: apt nainštaluje aj požiadavky Python SDK, ktoré balík odporúča, ale nevyžaduje (pozri [Krok 2](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Pre Jetson Orin / Orin NX / Orin Nano na JetPack 6 (L4T R36). Jeho PyTorch je CUDA verzia pre JetPack 6, takže spracovanie na GPU prebieha na GPU zariadenia Orin; využíva knižnice CUDA a cuDNN, ktoré inštaluje JetPack 6 (`sudo apt install nvidia-jetpack` na obraze, kde chýbajú).

#### Linux: prijímacie vyrovnávacie pamäte a backendová služba

Oba balíky .deb zvyšujú veľkosť prijímacích vyrovnávacích pamätí socketov jadra, ktoré potrebujú prúdy z kamery LATTICE, na 32 MB (`/etc/sysctl.d/60-chloros-gige.conf`). Pri predvolenom nastavení jadra sú pakety z kamery v jadre zahodené a pole stráca snímky a snímkovú frekvenciu; `chloros-cli lattice network` hlási nízky stav vyrovnávacích pamätí. Pozrite si referenčnú príručku CLI v časti [Linuxové hostiteľa — vyrovnávacie pamäte pre príjem cez socket](cli-reference.md#linux-hosts--socket-receive-buffers).

Na počítači bez grafického rozhrania spustite backend ako službu **pod vaším používateľským účtom**, aby skript našiel spustený backend (`connect_all()` nikdy nespustí backend):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

Backend potom beží ako vy: zachytené údaje, ktoré zapisuje, sú vaše a používa váš `chloros-cli login`. Rootový `chloros-backend.service` je stále nainštalovaný pre stroje, ktoré ho majú povolený, ale každý súbor, ktorý rootový backend zapíše, patrí rootovi. Pozrite si časť [Spustenie backendu ako služby](cli-reference.md#running-the-backend-as-a-service-headless) v referenčnej príručke k CLI.

Backend, ktorý spúšťa táto služba, je zdieľaný a klient ho nikdy nezastaví: skripty SDK a `chloros-cli` ho používajú tak, ako ho nájdu (`ChlorosLocal.shutdown_backend()` zastaví iba backend, ktorý sám spustil `ChlorosLocal`), a `GET /api/status` označuje túto jednotku ako `service`. Reštartujte ho pomocou `sudo systemctl restart chloros-backend@$USER.service`. Aktualizácia balíka zastaví služby backendu počas inštalácie, takže bežiaci skript stratí prístup ku kamerám a senzorom a následne znovu spustí všetky povolené zariadenia; odstránenie balíka ich zastaví a deaktivuje povolené zariadenia. Naraz beží len jeden backend, preto povolte len jednu jednotku: druhá sa odmietne spustiť a zostane v stave `failed` (stav ukončenia 98, nespustený nanovo) a jeho výstup `systemctl status` vysvetľuje prečo. Ak chcete prepnúť medzi jednotkami, deaktivujte tú, ktorú nepoužívate.</id></sn>

### Krok 2 — Inštalácia Python SDK

**Inštalačný program Chloros obsahuje zodpovedajúci balík SDK vo formáte wheel.** Každý inštalačný program pre Windows a balík .deb pre Linux umiestni na disk súbor `chloros_sdk-X.Y.Z-py3-none-any.whl`, ktorý presne zodpovedá verzii grafického rozhrania, príkazového riadku (CLI) a backendu. Nemusíte sledovať PyPI, aby ste zostali v synchronizácii.

#### Windows

Inštalačný program automaticky spustí súbor `pip install` s priloženým balíčkom „wheel“ pomocou vášho systémového Pythonu (uprednostňuje sa spúšťač `py.exe`, v prípade potreby sa použije `python -m pip`). Nie je potrebná žiadna akcia — po úspešnej inštalácii funguje `import chloros_sdk` vo vašom prostredí Python. Ak na počítači nie je nainštalovaný Python, inštalátor tento krok ticho preskočí a GUI + CLI budú naďalej fungovať.

#### Linux (.deb)

Oba balíky .deb (x86_64 a Jetson) nainštalujú SDK pre systém `python3`, takže `import chloros_sdk` funguje po inštalácii bez potreby pip a bez pripojenia k sieti. Balíky (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` a načítače projektov) sa umiestnia do `/usr/lib/python3/dist-packages`, a príkaz SDK `chloros-mcp` ([MCP server](#mcp-server-ai-assistants)) do `/usr/bin`, pričom všetky sú súčasťou balíka: aktualizácia ich nahradí a odstránenie balíka ich odstráni.

Vlastné požiadavky SDK, `python3-numpy` a `python3-requests`, sú **odporúčané, nie povinné** (GUI, CLI a backend sú samostatné): `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` na Jetsone) ich nainštaluje, `dpkg -i` nie. Výstup inštalácie uvádza verziu SDK, ktorú `python3` importuje, všetky chýbajúce požiadavky a o existencii súboru `chloros_sdk` na inom mieste v ceste `python3`, ktorý prekrýva kópiu balíka (napríklad balík nainštalovaný skôr pomocou `pip install --user`).

Vo virtuálnom prostredí (virtualenv), ktoré nevidí systémové balíky, nainštalujte do neho priložený súbor wheel:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

V prípade nasadenia na zariadeniach Jetson s izolovaným prostredím (air-gapped) prebieha tento proces úplne offline — SDK aj súbor wheel sú už na disku.

#### Verejný PyPI

Pre hostiteľa používajúceho iba pip (bez nainštalovaného balíka Chloros pre desktop; SDK v takom prípade riadi backend na inom počítači, pozri [Režim vzdialeného backendu](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

PyPI obsahuje najnovšiu stabilnú verziu. Udržujte SDK v rovnakej verzii ako backend Chloros, ktorý ovláda (pozri [Verzovanie a kompatibilita](#versioning--compatibility)). Pomocný program DAQ-M `DLS` tiež vyžaduje `bleak`: `pip install "chloros-sdk[camera]"` (alebo `pip install bleak`).

#### Overenie

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Vyžaduje sa predplatné Chloros+.** Všetky volania SDK vyžadujú aktívne prihlásenie do Chloros+. Spustite `chloros-cli login user@example.com 'YourPassword'` raz na každom počítači; prihlasovacie údaje sa ukladajú do vyrovnávacej pamäte v súbore `~/.chloros/`.

### Potrebujem balík Desktop Package?

Samotný balík pip **nestačí** pre väčšinu pracovných postupov. Tu je prehľad toho, čo vyžaduje každá rozhranie SDK:

| Rozhranie SDK | Potrebuje balík Desktop Package? | Prečo |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Áno**(na spustenie)**/ Nie** (backend je už dostupný) | Použiť bežiaci backend (z desktopovej aplikácie alebo z iného spusteného skriptu); inak automaticky spustiť nainštalovaný: `/usr/lib/chloros/chloros-backend` (Linux) alebo `<install folder>\backend\chloros-backend.exe` (systém Windows; predvolene `C:\Program Files\Chloros\backend\`, vlastná inštalačná zložka sa načíta zo záznamu inštalátora). Ak chcete názov zmeniť, zadajte `backend_exe=`. |
| `chloros-mcp` ([MCP server](#mcp-server-ai-assistants)) | **Áno**(na spustenie)**/ Nie** (backend je už dostupný) | Vytvorené na základe `ChlorosLocal`, teda rovnaké ako v riadku vyššie. |
| `read_image_audit_tags` | **Nie** | Načíta vlastné metadáta spracovaného obrázku (len štandardná knižnica Pythonu). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Áno**(lokálne)**/ Nie**(vzdialené) | Čisté HTTP klienti cez backend. Lokálny backend → vyžaduje sa balík pre desktop. Vzdialený backend → `backend_url=`**cez tunel** (pozri Režim vzdialeného backendu — dodávané backendy sa viažu iba na loopback). |
| `ChlorosProject` / `open_project` | **Áno** | Spravuje uložené projekty cez backend. |
| Priame triedy LATTICE (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Áno** | Vyžaduje sa runtime prostredie Arena SDK, ktoré je súčasťou balíka pre stolné počítače, a OpenCV (`cv2`) vo vašom Pythone, ktoré neinštaluje ani balík pre stolné počítače, ani SDK (`pip install opencv-python`). V opačnom prípade je `CAMERA_AVAILABLE` rovnaké ako `False` pri importe. `DLS` tiež potrebuje `bleak`. |

### Režim vzdialeného backendu (hostiteľ iba s pip, cez tunel)

> **Nainštalovaný backend nie je dostupný cez LAN.** Naslúcha iba na
> loopbacku (`127.0.0.1` / `::1`), takže `backend_url="http://<lan-ip>:5000"`
> **nemôže dosiahnuť nainštalovaný Chloros**. Ak chcete ovládať backend na inom
> počítači, sami presmerujte jeho port loopback a nasmerujte SDK na
> tunel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Hostitelia bez grafického rozhrania / CI / robotika môžu mať jeden počítač s plnou inštaláciou pracovnej plochy ako „Chloros server“ a `pip install chloros-sdk` všade inde — ale prenos medzi nimi prebieha cez vyššie uvedený tunel zriadený používateľom, nie cez priamu URL v LAN.

> **`ChlorosLocal` na hostiteľskom počítači s výhradne pip.** `ChlorosLocal(backend_url=BACKEND)` sa pripája k backendu, ku ktorému má prístup bez inštalácie na pracovnej ploche: hľadá lokálny binárny súbor backendu len vtedy, keď ho musí spustiť (ak nič neodpovedá a `auto_start_backend=True`). Až do verzie 1.2.0 najprv hľadal binárny súbor a na hostiteľovi, na ktorom je nainštalovaný len pip, vyvolal chybu `ChlorosBackendError` („Backend Chloros nebol nájdený…“) aj v prípade, že bol backend dostupný. Zložky, ktoré zadáte (`process_folder`, `import_images`), sa na tomto počítači skontrolujú a prečítajú backendom, takže pri vzdialenom backende musia mať na oboch zariadeniach rovnaký názov zložky, napríklad na zdieľanom disku.

### Pracovný postup len s DAQ (hostiteľ len s pip)

`pip install chloros-sdk` sám o sebe nemôže ovládať senzor DAQ: senzor DAQ sa vždy ovláda prostredníctvom backendu Chlorosu s `connect_daq_sensor()`:

- **Senzor na tomto počítači:** nainštalujte tu balík Chloros pre stolné počítače. `connect_daq_sensor()` spustí svoj backend, ak žiadny nebeží.
- **Senzor na inom počítači:** tento počítač potrebuje balík pre stolné počítače; k jeho backendu sa dostanete cez tunel v [režime vzdialeného backendu](#remote-backend-mode-pip-only-host-via-tunnel). Backend otvorí senzor na svojom vlastnom počítači, takže DAQ-U je pripojený k tomuto zariadeniu, DAQ-M sa nachádza v dosahu Bluetooth a DAQ-E je v sieti, ku ktorej má prístup.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

V oboch prípadoch backend vynucuje plán „Chloros+“: prihláste sa na počítači, na ktorom beží backend (`chloros-cli login` alebo desktopová aplikácia), s akýmkoľvek plateným tarifom (Copper alebo vyšším). Odhlásený backend a tarif nižší ako Copper vyvolajú chybu `ChlorosConnectError` obsahujúcu dôvod backendu (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Rýchly štart

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Index API najvyššej úrovne

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Spracovanie obrazu — `ChlorosLocal`

Hlavná trieda spracovateľského reťazca. Pri vytvorení spustí backend (ak žiadny neodpovedá a `auto_start_backend=True`), vytvorí a nakonfiguruje projekty, monitoruje priebeh a vráti súhrny po ukončení spustenia.

### Konštruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metódy

| Metóda | Popis |
| --- | --- |
| `create_project(project_name, camera=None)` | Vytvorí nový projekt (voliteľne s šablónou kamery, napr. `"Survey3N_RGN"`). |
| `open_project(project_path, relink=None)` | Otvorí uložený projekt (jeho zložku alebo jeho názov v zložke projektov) a nastaví ho ako otvorený projekt. Cesty presunutého projektu sú presmerované na zložku, v ktorej sa teraz nachádza; `relink` označuje zložku, v ktorej sa teraz nachádzajú ostatné chýbajúce súbory. Vráti odpoveď backendu pomocou `source_files`; vyvolá výnimku `ChlorosProjectFilesMissingError`, ak sa žiadna nenájde. Pozrite si časť **Presunutý projekt** nižšie. |
| `import_images(folder_path, recursive=False)` | Import obrázkov vo formátoch RAW/TIF/JPG/DNG **a `.daq` záznamov zo svetelného senzora**. Vráti `count` (obrázky) a `scan_count` (záznamy). Upozorní len v prípade, ak zložka neobsahuje ani jedno, ani druhé. |
| `export_light_sensor(daq=True, csv=True)` | Zapíše kalibrované súbory `.daq` + `.csv` pre každý záznam zo svetelného senzora v projekte do súboru `<project>/Light Sensor/`. Pozri [Záznamy svetelného senzora](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Nastaviť ovládače spracovania. `indices` sú názvy predvolieb (`NDVI`, `GNDVI`, `OSAVI`, …), z ktorých každý je pridaný do projektu pre príslušné pásma danej kamery; názov, ktorý nie je prednastavením, alebo ktorý pásma nemôžu vytvoriť (`NDRE` vyžaduje RE a NIR, čo žiadna kamera sama o sebe neposkytuje), dostane názov `UserWarning` a žiadny indexový obrázok. `daq_log_path` označuje záznam zo svetelného senzora (súbor typu `.daq` alebo `.csv`, prípadne zložku s takýmito súbormi), s ktorým sa porovnáva každý snímok, a to v programoch LATTICE aj Survey3, namiesto záznamov importovaných spolu so snímkami; ak chýba cesta alebo neobsahuje žiadny záznam, použijú sa importované záznamy. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Spustí spracovateľský reťazec. Vráti `{"status": "complete", "async": False}`, plus kľúč `summary`, ak ho backend poskytuje — pozri [Súhrn a tipy po spustení](#post-run-summary--hints) — `export_check_failure`, ak sa výstupy spustenia nedali overiť, a `image_failure`, ak došlo k strate obrazu. `wait=False` vráti `{"status": "started", "async": True}` naraz. |
| `get_config()` / `get_status()` / `status()` | Skontrolujte stav backendu. |
| `logout()` | Odhlásenie: vymaže uložené prihlasovacie údaje z vyrovnávacej pamäte a uvoľní slot zariadenia tohto počítača (`device_deregistration` vo výsledku). Prebiehajúce spracovanie sa nezastaví: dokončí sa v režime, v ktorom sa spustilo (`run_in_progress: true` a `run_mode` vo výsledku), a nasledujúce spustenie používa štandardné spracovanie. |
| `shutdown_backend()` | Zastaví backend, ak ho spustila táto inštancia; backend, ktorý zistila, že beží, sa ponechá bez zásahu. Zanechanie bloku `with` alebo vykonanie garbage collection na inštancii ho nezastaví: backend pokračuje v behu pre ďalší skript a zastaví sa sám po 30 minútach, ak ho nič nevyužíva. Backend je požiadaný o vypnutie, takže najprv uvoľní svoje kamery a svetelné senzory a dokončí akékoľvek nahrávanie; je ukončený len vtedy, ak stále beží aj o 10 sekúnd neskôr. |
| `discover_cameras()` | Zistiť kamery LATTICE **prostredníctvom backendu tejto inštancie** (`/api/camera/discover`). Vráti zoznam slovníkov (`serial`, `model`, `ip`, …) — v rovnakom formáte, aký vidí GUI/CLI. Prázdny zoznam, ak sa nenašli žiadne kamery alebo je backend nedostupný. |
| `camera_capture(output_dir, format="tiff", **settings)` | Zachytí jeden snímok**prostredníctvom backendu**(automaticky spusteného týmto identifikátorom), aby prešiel rovnakou prípravou ako v GUI alebo CLI (predvolene 12-bitové, opätovné použitie vyrovnávacej pamäte, vložené kalibračné metadáta). Cieľ vyriešte pomocou `serial=` alebo `device_index=`; odovzdajte `exposure`/`gain`/`pixel_format`/`preset` ako `**settings`. Vráti slovník starších metadát (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Vytvára kompozitné náhľadové snímky s prekrývaním z združenej kamery — odľahčený MJPEG klient cez trasu `/api/camera/<serial>/stream-annotated` na strane backendu (zebra / mriežka / nitkový kríž / histogram / peaking / bodové označenie vykreslené na strane servera). `decode=True` generuje polia BGR a vyžaduje OpenCV (`cv2`), ktoré SDK neinštaluje; `False` generuje surové bajty formátu JPEG. Dostupné aj na úrovni projektu ako `ChlorosProject.stream(overlays=True)`. |

**Spustená úloha si zachováva svoj projekt.** Počas prebiehajúceho spracovania — či už tohto skriptu, aplikácie, `chloros-cli process` alebo `chloros-mcp` — backend odmieta vytvoriť, otvoriť, importovať do projektu alebo zmeniť jeho nastavenia spracovania (HTTP 409 `PROCESSING_IN_PROGRESS`): `create_project()` a `import_images()` vyvolajú výnimku `ChlorosProcessingError`, a `configure()` vyvoláva `ChlorosConfigurationError`, a `process()` s iným režimom, ako je ten, v ktorom prebieha spustenie, vyvoláva `ChlorosProcessingError` (spustenie si zachováva režim, v ktorom začalo), pričom každé z nich obsahuje správu backendu. Zachytenie uložené do projektu sa k nemu pripojí po ukončení spustenia (uvádza to odpoveď `project_add`). Počkajte, kým sa beh dokončí (`get_status()`), a potom pokračujte.**Presunutý projekt.** `project.json` projektu označuje každý zdrojový súbor (každý obrázok a záznam zo svetelného senzora) podľa jeho úplnej cesty, takže presunutie zložky projektu – alebo celej zložky projektov na iný disk – spôsobí, že súbory zostanú označené podľa pôvodného umiestnenia. Každé otvorenie projektu opraví, čo sa dá, automaticky: súbor, ktorý bol v zložke projektu, sa vyhľadá v zložke, v ktorej sa projekt nachádza teraz, a každá tam nájdená cesta sa preorientuje a uloží (`project.json` sa najskôr zálohuje ako `project.json.before-relink-<date>-<time>`). Cesta sa preorientuje iba na súbor, ktorý sa tam nachádza. Súbory, ktoré sa nachádzali mimo zložky projektu – relácia Hub, import pomocou funkcie Pridať zložku – sa nájdu iba v zložke `relink=<the folder they are in now>` (alebo v akejkoľvek zložke nad ňou). `open_project()` vráti `source_files`: `total`, `found`, `missing`, `missing_roots` (zložky, v ktorých sa nachádzali chýbajúce súbory), `missing_outside_project` a `relink` (čo bolo presmerované, odkiaľ kam). Projekt, v ktorom chýbajú niektoré súbory, sa otvorí so zvyškom; projekt, v ktorom sa nenašiel žiadny súbor, sa neotvorí, a súbor `open_project()` vyvolá chybu `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). Chyby `project open --relink` a `chloros-mcp` v CLI`open_project` (`relink_folder`) v CLI fungujú rovnako.

Použite ako správcu kontextu pre zaručené vyčistenie:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Záznamy zo svetelného senzora — kalibrované `.daq` + `.csv`

Záznamy z DAQ-U / DAQ-M / DAQ-E je možné zaznamenávať **bez** ich kalibračného balíka. To je to,
čo verejné [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) robia štandardne: zapisujú surové hodnoty snímačov a označujú
súbor tak, aby Chloros najprv načítal továrenskú kalibráciu daného snímača **podľa sériového čísla** — najprv z lokálnej cache
, potom z cloudu MAPIR — a aplikuje ju pri importe.

Chloros zapíše výsledok späť ako dva produkty na jeden záznam, pod
`<project>/Light Sensor/`:

| Produkt | Čo to je |
| --- | --- |
| `<name>_calibrated.daq` | Archív, ktorý je možné opätovne spracovať — rovnaká schéma ako pri živom zázname, teraz s deklaráciou balíka, ktorý ho vytvoril. Jeho opätovný import ho **ne** kalibruje druhýkrát. |
| `<name>_calibrated.csv` | Spektrálna intenzita žiarenia v W/m²/nm na vlastnej vlnovej mriežke senzora, jeden riadok na jedno meranie, plus fotometrické stĺpce (celkový výkon, fotopický/skotopický lux, PPFD a jeho rozdelenie na modrú/zelenú/červenú zložku, vrcholová vlnová dĺžka). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Iba senzory bez balíka (DAQ-A).** Surové spektrálne počty senzora — *nie* ožiarenie. Pozri nižšie. |

`<name>` je názov súboru samotného záznamu. Dva záznamy v rôznych zložkách môžu zdieľať
jedno — hodnoty namerané pri jednom snímaní (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) uložené v
tej istej milisekunde dvoma kamerami — a každá z nich stále dostane svoje vlastné výstupy: prvá si ponechá
`<name>`, ďalší pridáva zložky, ktoré ho odlišujú, **za** názvom
(`<name>__<capture>_daq_calibrated.daq`), takže dátum a čas záznamu sú stále na prvom mieste.

`process()` vykonáva tento export ako jednu zo svojich fáz. **Nevyžaduje** snímky:
sám o sebe lietajúci svetelný senzor predstavuje prvotriedny pracovný postup a takýto projekt má konštrukčne nulový
počet snímok.

**Záznamy DAQ-A sa exportujú ako surové počty.** Rodina DAQ-A predchádza systému zväzkov podľa sériového čísla
a nemá žiadny zväzok, ktorý by bolo potrebné načítať — namiesto toho sa kalibruje v teréne voči
reflexnému terču, preto ho nikdy nepotrebovala. Tieto záznamy sa exportujú
pod predponou `_raw` namiesto `_calibrated`: ide o odlišný názov súboru namiesto príznaku
vnútri súboru, pretože identifikátor musí prežiť odoslanie e-mailom ako holý názov.
Hlavička `.csv` uvádza `raw spectral sensor counts (NOT irradiance)` a upozorňuje, že
hodnoty sú porovnateľné **v rámci** súboru — presne to, na čo slúži kalibrácia pomocou cieľa
— a nie medzi senzormi. Fotometrické stĺpce závislé od výkonu (celkový výkon,
fotopický/skotopický lux, PPFD) sa vracajú ako **NULL** namiesto toho, aby boli integrované z počtov.

DAQ-U / DAQ-M / DAQ-E, ktorých balík sa jednoducho nepodarilo načítať, sa stále **preskočia**,
nezapíšu sa v surovom formáte: v takom prípade balík existuje a „znovu sa pripojiť a spracovať“ je skutočná rada.

Staršie záznamy **v1.01 / v1.02** (zapisuje ich DAQ-A-SD) neobsahujú epochu pre každé odčítanie,
len čas zápisu súboru. Porovnávač obrazu↔svetelného toku ich stále odmieta — porovnanie
s časom zápisu by bolo neviditeľne nesprávne — ale exportér ich číta a
v CSV sa vytlačí `clock=daq_created_on`, takže produkt uvádza, na akých hodinách sa nachádza.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Záznam, ktorého kalibračný balík nie je možné načítať (offline alebo senzor bez
kalibrácie v súbore), je nahlásený pod `skipped` **s uvedením dôvodu**. Nikdy sa
nezapíše ako „kalibrovaný“ súbor obsahujúci surové počty — pripojte sa k internetu a
spustite proces znovu, čím sa export dokončí.

### Spätné volania o priebehu

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Súhrn a tipy po spustení

Po dokončení `process()` načíta `GET /api/processing-summary` a pripojí telo ako `result["summary"]`. Načítanie prebieha podľa najlepších možností a nikdy neblokuje úspešný návrat — ak nie je súhrn k dispozícii, `process()` prejde na jednoduchú formu `{"status": "complete", "async": False}`. Každý záznam v `summary["hints"]` — celé vety s navrhovaným riešením, napr. prečo beh vyprodukoval nulový výstup — sa tiež opätovne vygeneruje ako Pythonový formát `UserWarning`, takže behy s nulovým výstupom sú samodiagnostické, aj keď slovník nikdy nepreveríte:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` je strojovo čitateľná polovica:

| Kľúč | Čo počíta |
| --- | --- |
| `models` | Skupiny kamier v behu. |
| `images_in_groups` | Zdrojové snímky v týchto skupinách. |
| `targets_found` | Zistené ciele odrazivosti. |
| `images_calibrated` | Snímky, pre ktoré beh vytvoril produkt odrazivosti. |
| `images_failed` | Snímky, ktoré zlyhali a neboli exportované. |
| `exported_files` | **Súbory produktov snímok , ktoré beh vytvoril**: každý produkt (odstránenie matice, náhľad, žiarivosť, odrazivosť, index, vykreslenia cieľov) vo všetkých formátoch. |
| `reflectance_files` | Produkty odrazivosti medzi nimi (`Reflectance_Calibrated_Images`), bez ohľadu na to, či boli kalibrované cieľom alebo ožiarením svetelného senzora (DAQ). Spustenie bez cieľa, ktoré napriek tomu zapísalo odrazivosť, to uvádza v upozornení; iba spustenie, ktoré nezapísalo nič, dostane informáciu, že chýbajú výstupy odrazivosti. |
| `clamp_masks_written` | Vedľa nich sú uvedené [masky obmedzenia oddeľovania](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) uvedené vedľa nich — sú doplnkové údaje, nie výstupy, takže nie sú v `exported_files` (CLI ich počíta oddelene rovnakým spôsobom). |
| `captures_left_out` | Surové snímky v priečinku `captures` projektu, ktoré tento beh nezaznamenal. Spustenie čaká pri štarte na to, čo bolo zachytené pred ním, takže táto hodnota je 0, pokiaľ nebolo počas spustenia uložené žiadne zachytenie alebo sa nevyčerpala doba čakania. `summary["captures_left_out_note"]` uvádza, čo s nimi robiť. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Záznamy zo svetelného senzora, zámerne počítané oddelene — pochádzajú z inej fázy a existujú aj pre behy bez akýchkoľvek snímok, takže ich zahrnutie by spôsobilo, že beh len s DAQ by vyzeral, ako keby exportoval snímky. |

Spolu s nimi: `summary["output_dirs"]` (každý adresár, do ktorého sa zapisovalo),
`summary["light_sensor_export"]`, `summary["stopped"]` (platí, ak používateľ prerušil
beh, aby sa čiastočné počty neinterpretovali ako dokončený beh s nedostatočnou produkciou) a
`summary["groups"]` (rozpis podľa skupín).

`exported_files` sa zaznamenáva potrubím **priamo počas zápisu**, nie sa neskôr skenuje z
objektov obrazu projektu. Paralelná stratégia a stratégia GPU vytvárajú svoje vlastné objekty obrazu
(v podprocesoch pracovníkov pre cesty GPU), takže staré skenovanie hlásilo
`0 file(s) written` pre každý takýto beh a následne vysielal upozornenie o nulových exportoch — pri behoch,
kde všetko fungovalo. Ak vytvárate skript na základe tohto čísla, správne fungujúci paralelný beh teraz
hlási nenulový počet.

Preskočenia svetelným senzorom hlásia dôvod, ktorý čítač skutočne zistil pre každý súbor —
nečitateľná schéma, chýbajúci balík, chyba zápisu — **deduplikované**, takže dvadsať súborov
preskočených z jedného dôvodu sa vyhodnotí ako jeden dôvod, a nie ako dvadsať opakovaní tohto dôvodu.

> **`process()` sa nespustí, ak beh nevytvorí žiadne obrazy.** Toto je jedno z dvoch miest, kde sa SDK a
> CLI zámerne líšia: `chloros-cli process` považuje situáciu „boli požiadané produkty, žiadne neboli
> zapísané“ za zlyhanie a ukončí sa s nenulovým kódom, zatiaľ čo SDK sa vráti normálne a nahlási
> stav prostredníctvom `summary` / hints. Ak by sa váš proces mal zastaviť pri prázdnom behu, skontrolujte to
> sami — skontrolujte `summary` (alebo spočítajte súbory v zložke projektu), namiesto toho, aby ste sa spoliehali na
> absenciu výnimky. Bežnými príčinami sú vstupná zložka, ktorá nebola rozpoznaná ako
> záznam, a produkty preskočené ako nepoužiteľné pre prítomné kamery (napr. radiancia z kamier s iba RGB
> rozlíšením).
>
> Chyba `process()` sa nevyskytne ani v prípade, že sa nedali overiť výstupy behu: zobrazí sa varovanie a nastaví sa
> `result["export_check_failure"]`. Výstupy sú zapísané. Ak chcete proces v tomto bode zastaviť, skontrolujte tento kľúč. To isté platí pre beh, pri ktorom
> došlo k strate snímky: systém vydá varovanie a nastaví `result["image_failure"]`, pri ktorom `chloros-cli process`
> ukončí beh s nenulovým kódom.

### Pomocné funkcie

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Každá beží vo svojom vlastnom `with ChlorosLocal(...)`; backend, ktorý musela spustiť, pokračuje v behu, keď sa vráti, pre ďalšie volanie, a zastaví sa po 30 minútach, ak ju nikto nepoužíva. Ani jedna z nich neprijíma `auto_start_backend`, `radiometric_output` ani `daq_log_path`; pre ne použite `ChlorosLocal` a `configure()`. `process_lattice_capture()` neprijíma hodnoty `camera`, `debayer` ani `ppk`: preskočí detekciu panelu a používa štandardný debayer.

### Podporované hodnoty

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrický výstup (multispektrálny spracovateľský reťazec LATTICE)

Úroveň exportu multispektrálneho (M3C/M3M) potrubia LATTICE s kódom `process` — `reflectance` (predvolené), `radiance`, `sensor-response` alebo `all` (každý príslušný režim pre každý obrázok) — nastavuje produkty projektu **„Exportovať radianciu“**a**„Exportovať odrazivosť“**: `radiance` exportuje iba radianciu, `reflectance` iba odrazivosť, `sensor-response` ani jedno (len produkty po odstránení bayera a náhľady), `all` oboje. Produkty po odstránení bayera a náhľady nie je možné na tejto úrovni meniť. `configure()` má na to vyhradené kľúčové slovo:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Pokročilá úniková možnosť — zapisovanie produktov cez `custom_settings` — stále funguje, ale pamätajte, že nahrádza celý blok nastavení (pozri varovanie nižšie). Zapíšte kľúče `"Export <product>"`: vlastné nastavenia projektu `"Export radiance"` / `"Export reflectance"` majú prednosť pred starším kľúčom `"Radiometric output"`, takže tento kľúč sám o sebe nič nemení.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (predvolené nastavenie) delí žiarivosť kamery **s časovou pečiatkou zosúladeným DAQ downwellingom**, automaticky vypočítaným zo zaznamenaného `.daq` (DAQ-U/M/E)**alebo natívny produkt typu DAQ-M `.csv`**, ktorý sa nachádza spolu so snímkami; akýkoľvek balík kalibrácie pre konkrétnu kameru alebo DAQ, ktorý lokálne chýba, sa pri prvom použití**automaticky stiahne z AWS**. Rozhranie CLI to sprístupňuje podľaprepínačov typu produktu na `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**Čí je to údaj.**Údaj, ktorý zachytáva záznam typu `.daq`, je údaj z kamery (kamier), ktorú (ktoré) popisuje — uvedenej v sprievodnom súbore typu `<file>.daq.cameras.json` vedľa neho (jednotlivé, synchronizované a pole záznamov; surová séria uvádza hodnoty každého člena vo svojom manifeste) — a zodpovedá**len snímkam z týchto kamier**. Samostatná kamera bez vlastnej hodnoty preto už nevyužíva DAQ inej kamery; priraďte k nej DAQ alebo nahrajte vlastný záznam. Vlastný záznam DAQ (`chloros-cli daq pool-record` alebo záznam spustený v grafickom rozhraní) alebo akýkoľvek záznam typu `.daq` bez tohto súboru „sidecar“ (vrátane starších záznamov) stále slúži pre všetky kamery.**Kamery OCN na slnku.** Každý kalibračný balík OCN je vytvorený pri úzkopásmovom osvetlení, s jednou LED diódou na pásmo, zatiaľ čo slnečné svetlo tieto pásma zmiešava. Chloros 1.3.0 po oddelení pásiem aplikuje hromadnú korekciu na základe slnečného osvetlenia, čo zlepšuje produkty OCN týkajúce sa radiancie, odrazivosti a indexu v slnečnom svetle. Funguje všade tam, kde prebieha oddelenie pásiem — v `process()`, pri snímaní, v `grab()` / `frame_stream()` a v živom náhľade — pre balíky OCN, ktoré to podporujú; kamery RGN zostávajú nezmenené. Snímky OCN zachytené staršou verziou túto funkciu tiež získajú, ak sa spracúvajú na počítači, ktorý obsahuje kalibráciu, s ktorou boli nasnímané (Chloros si uchováva kalibráciu kamery po jej pripojení); v opačnom prípade sa spracúvajú ako doteraz. Hodnoty žiarivosti, odrazivosti a indexu OCN sa preto líšia od hodnôt vo verzii 1.2.0.

> `custom_settings` **nahrádza** celý blok vypočítaných nastavení (podľa návrhu obchádza ostatné kľúčové slová a validáciu `configure()`). Ak ho použijete, zahrňte všetky kľúče `Project Settings`, ktoré vás zaujímajú, ako je to v príklade vyššie.

#### Čítanie indexových pixelov (index NDVI a ostatné vegetačné indexy)

**Celé číslo indexového rastra predstavuje kontrastné rozťahovanie na úrovni jednotlivých snímok, nie mapu v rozsahu −1…1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` a `JPG (8-bit)` — vrátane 8-bitového formátu TIFF, v ktorom sa ukladajú 8-bitové snímky — ukladajú

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

Na čítanie nie je potrebný `-config pix4d.config` — exiftool číta tieto vlastnosti XMP bez pomoci. Konfigurácia je potrebná len na ich *zapisovanie*.

> **`IndexMin == IndexMax`** znamená, že každý pixel daného snímku mal rovnakú hodnotu indexu. Raster je vyplnený v strednej škále a nie je čím deliť — použite koncovú hodnotu.

> **Žiadne značky `Chloros:Index*` znamenajú, že nie je uvedené žiadne mapovanie.** Export LUT je farebne upravený výstup, ktorého pixely majú farebné prechody priradené k prahovému oknu vašej LUT, a snímka bez pixelu s konečným indexom nemá žiadny rozsah, ktorý by bolo možné uviesť. V žiadnom z týchto prípadov nepredpokladajte existenciu mapovania — platí rovnaké pravidlo ako pri chýbajúcej značke `PixelScale`.**Keďže rozťahovanie sa vzťahuje na jednotlivé snímky, dva snímky z jedného letu nie sú porovnateľné z hľadiska DN**— každý má svoje vlastné koncové body, takže rovnaká hodnota NDVI sa v rôznych snímkach nachádza na rôznych hodnotách DN. Pre kvantitatívnu prácu v rámci letu (zónové štatistiky, pevné prahové hodnoty, časové rady) exportujte**`TIFF (32-bit, Percent)`**: indexové rastery sú tam zapísané ako float32**samotné hodnoty indexu**, bez potreby preškálovania a bez značiek. Napriek názvu formátu nie sú indexové produkty násobené ×100 — toto škálovanie sa vzťahuje iba na odrazivosť s celočíselným dátovým typom.

#### Maska obmedzenia pri oddeľovaní spektrálnych zložiek — na ktorých pixeloch bolo pásmo obmedzené

Spektrálne oddeľovanie 3×3 môže spôsobiť, že pásmo bude záporné — ak svetlo obsahuje toto pásmo len v minimálnej miere alebo ak scéna nespadá do rozsahu, ktorý popisuje matica oddeľovania kamery. Keďže to nie je skutočná hodnota, export ju nastaví na 0 — a živý náhľad zobrazuje rovnakú hodnotu 0. Pásmo môže byť obmedzené na niekoľko pixelov alebo takmer na všetky; maska zaznamenáva obidve možnosti. `Chloros:Band{R,G,B}UnmixClampFraction` udáva, na akej časti snímky k tomu došlo; maska obmedzenia udáva, **ktoré pixely**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— v priečinku `clamp_masks/`**vedľa** priečinka produktu, nikdy nie v ňom (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — pre každý produkt radiancie, odrazivosti a indexu, v ktorom došlo k obmedzeniu aspoň jedného pixelu. Jednopásmový `uint8`, na vlastnej pixelovej mriežke produktu (orientácia, ROI, zlučovanie a deformácia zarovnania poľa sú zahrnuté). Každá maska uvádza názov kamery príslušného produktu (`Make` / `Model` / sériové číslo), čas zachytenia a označenia objektívu a uvádza zoznam pásiem, ktoré jej bity reprezentujú (`XMP-Camera:BandName`, najprv bit 0); jej `ImageDescription` a `Software` ju označujú ako masku s obmedzením a neobsahuje žiadnu kalibráciu.
- **Bity**: bit 0 = pásmo v kanáli R produktu (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Nastavený bit znamená, že hodnota pixelu v danom pásme bola obmedzená na 0 — považujte ho, ako aj akýkoľvek index z neho odvodený, za málo spoľahlivý.
- **`Chloros:UnmixClampMask`** je označený na každom produkte, na ktorom prebehlo oddelenie: cesta k maske relatívna voči vlastnej zložke produktu (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, vždy oddelená znakom `/`– oddelené – pripojte ju k zložke obrázka, aby sa maska otvorila), alebo `none`, ak nebol obmedzený žiadny pixel. Chýbajúca značka znamená, že na produkte sa nikdy neprebehol proces oddelenia farieb (kamery RGB alebo monochromatické, úrovne raw/debayered).
- Výsledky zachytenia to uvádzajú ako **`clamp_mask`** snímky (cesta k maske, alebo `None`, ak nebolo nič obmedzené) — vo výsledkoch `CameraSession.capture()` sú to snímky `ArraySession.capture()`snímky `CaptureResult` a odpovede HTTP. Offline zostavenie uvádza svoje masky ako `clamp_masks`.
- Viacpásmový zásobník živého indexu (`/api/camera/cli-array-index` s `save_multiband`) dostáva jednu masku na zdrojovú kameru, a to vo vlastnom súbore `clamp_masks/` vo výstupnej zložke — pozri referenčnú príručku CLI`lattice index`. Video nemá žiadnu masku: výsledok offline zostavenia namiesto toho uvádza frakciu obmedzenia každého snímku.

Keďže žiadny produktový priečinok neobsahuje masku v žiadnej úrovni, produktový priečinok je možné odovzdať fotogrametrickému softvéru tak, ako je — vrátane podpriečinkov — a ten spracováva iba snímky. Odovzdajte mu produktový priečinok, nie zložku nad ňou. Vlastný import programu Chloros masky preskakuje.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect pre kamery LATTICE

Trvalé relácie na pozadí pre hardvér v reálnom čase. Používajú sa rovnaké koncové body ako v grafickom rozhraní, takže správanie je identické v SDK, CLI aj GUI.

### Jedna kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

Expozícia, zosilnenie a cieľ nastavené tu zostávajú zachované: automatické riadenie expozície systému Chloros (vrátane expozície meranej DAQ) ich nemení. `cam.set_settings(exposure_auto="Continuous")` (alebo `gain_auto=`) vráti expozíciu (zosilnenie) späť, a `cam.set_settings(target_brightness_auto=True)` vráti cieľ späť automatickému nastaveniu cieľa v Chloros.

#### `connect_camera()` Podpis

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **Vráti sa so spusteným prúdom.** Backend čaká, s časovým obmedzením (približne 15 s na spustenie prúdu, potom 10 s na jeho prvý kompletný buffer — nie na prvý náhľadový snímok, ktorý nasleduje približne o 1,5 s neskôr), takže hneď po ňom nasleduje `status()`, čo znamená, že sa zobrazí `streaming`. `connect_stream` relácie je blok odpovede `stream`: `state` `streaming`, `no_frames` (prúd sa spustil, ale nič nedoručil — uvádza to blok `note` a v prípade kamery s jumbo paketmi sa má opätovne pripojiť s veľkosťou paketu 1500 bajtov), `starting` alebo `start_failed`, pričom `waited_s`. `None` pre kameru, ktorá už bola pripojená. Náhľad (`get_camera_preview`, `/api/camera/jpeg/<serial>`) vyžiadaný hneď po čakaní, obmedzenom na 4 s, na prvý náhľadový snímok kamery namiesto odpovede „Zatiaľ nie je k dispozícii žiadny snímok“; kamera, ktorej stream zatiaľ nič nedodal (`no_frames` alebo pripravená na hardvérový spúšťač), stále odpovedá okamžite.
- **Veľkosť paketu si môžete nastaviť sami.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` otvára kameru s paketmi o veľkosti 1500 bajtov, bez testovania (`chloros-cli lattice cam-connect --packet-size standard` odosiela to isté).
- **Kamerový fond je obmedzený na jedno pripojenie naraz**: otvorenie, testovanie veľkosti paketu, prvé stiahnutie kalibrácie (asi 70 s pre kameru, ktorá je na tomto počítači nová), spustenie prenosu. Žiadosť, ktorá medzitým potrebuje kameru (`read_nodes`, `set_settings`, zachytenie, druhé pripojenie) na ňu čaká maximálne 5 s, potom zlyhá s chybou HTTP 503 `CAMERA_POOL_BUSY`, pričom uvedie názov pripájajúcej sa kamery (`ChlorosConnectError`); opýtajte sa o chvíľu skúste znova. Zoznamy nikdy nečakajú: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` a `CameraSession.status()` odpovedajú okamžite. Kamera uprostred pripájania je uvedená pod odpoveďou skupiny `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — a `elapsed_s`), a jeho hodnota `status()` udáva hodnotu `connecting: True` s hodnotou `phase`; ostatné kamery sa čítajú v reálnom čase, ak je skupina voľná, inak sa odpovedá na základe ich posledného odčítania (`live: False`).

#### Metódy `CameraSession`

| Metóda | Popis |
| --- | --- |
| `status(timeout=15.0)` | Aktuálny stav kamery, ako ho číta `chloros-cli lattice status`: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` alebo `stopped`), `device` (model, sériové číslo, verzia firmvéru, teplota, `is_mono`) a jeho aktuálne nastavenia. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Načíta uzly GenICam; vráti `{nodes, errors, enums, device, was_in_pool, in_array_id}`, plus `frame_values` a `notes` počas behu automatického nastavenia expozície (pozri nižšie). |
| `set_settings(**kwargs)` | Zapíše uzly podľa popisného názvu (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). Hodnota `exposure_time` (`gain`) predstavuje manuálnu expozíciu (zisk): pri zápise bez `exposure_auto` (`gain_auto`), najprv sa vypne automatický režim, aby hodnota nebola odmietnutá bežiacim automatickým režimom. Výsledná hodnota `applied` zachováva to, čo kamera zaznamenala; zápis, ktorý fotoaparát odmietne, sa nachádza v `errors` a nie je zachovaný. Chyba uvádza rozsah, ktorý fotoaparát momentálne akceptuje (maximum v `ExposureTime` závisí od snímkovej frekvencie fotoaparátu), a automatický režim, ktorý bol vypnutý iba pre odmietnutú hodnotu, sa opäť zapne; `restored` ho uvádza (napr. `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Zachytenie **jediného** snímku. Vráti `CaptureResult`: zoznam slovníkov metadát snímok (jeden na každú uloženú úroveň), s `.skipped`, `.failed`, `.daq_files` a `.daq_matches`. (Sériové/viacrámcové snímanie bolo odstránené — ak potrebujete sériu, volajte `capture()` v slučke.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Zábery za sebou (`interval_s=0`) alebo v intervaloch z tejto kamery, pre snímky `count` a/alebo `duration_s` (aspoň jeden, aby sa proces ukončil), podobne ako to robí `ArraySession.capture_repeated()` pre pole. Vráti zoznam `CaptureResult`; zastaví sa po snímke, ktorá hlási zaplnený disk. `on_capture(i, result)` beží po každej z nich; ostatné kľúčové slová idú do `capture()`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Uvoľnenie z fondu, kamera sa vráti do stavu, v akom bola pri pripojení (spúšťacie zapojenie, PTP, nastavenia); `keep_settings=True` namiesto toho uloží svoje nastavenia do vlastnej pamäte, `clear_settings=True` túto pamäť vynuluje (pozri [Pamäť kamery](#camera-memory--keep_settings--clear_settings)); vráti `camera_memory`, keď je požadované jedno z nich. Kamera, ktorá je členom poľa, uvoľní celé pole. Žiadna operácia, ak sme sa pripojili k už otvorenej relácii. Vráti sa, akonáhle backend potvrdí uvoľnenie, alebo ak nie je čo uvoľniť (kamera už neexistuje alebo nebeží žiadny backend); uvoľnenie, pri ktorom došlo k časovému limitu alebo ktoré backend odmietol, vyvolá výnimku `ChlorosConnectError`, pretože kamera môže byť stále zadržaná a ďalšie pripojenie by s ňou kolidovalo. Ukončenie bloku `with` nikdy nevyvolá výnimku. |

Atribúty: `serial`, `device` (záznam o zistení kamery) a `backend_url`.

Ovládacie prvky exportu `capture()` (rovnaký model ako pole + grafické rozhranie):

- `processing` / `levels` — `processing="all"` ukladá všetky použiteľné typy exportu; `levels=["raw","radiance"]` ukladá len tie (prepíše `processing`). Ak chcete použiť predvolené nastavenie backendu, vynechajte oba.
- `daq` — hodnota DAQ, ktorá sa použila pri snímaní, sa štandardne uloží ako doplnkový súbor `.daq`, pričom je zahrnuté aj surové snímanie, takže snímku je možné neskôr opätovne spracovať na odrazivosť/index. `daq=False` ho preskočí, ak ho žiadna úroveň nepoužíva. `force_daq=True` je starší zápis predvoleného nastavenia. Bez DAQ, žiadne `.daq`.
- `daq_tolerance_ms` — aká veľká môže byť odchýlka od expozície a stále sa nachádzať v tolerancii (predvolené 500). Mení sa len výsledok.

#### `read_nodes()` pri automatickej expozícii — `frame_values` a `notes`

Počas behu `ExposureAuto` (`GainAuto`) uzol `ExposureTime` (`Gain`) načítava **poslednú zapísanú hodnotu**, nie aktuálny stav kamery. `nodes` stále uchováva túto hodnotu uzla. Keď požiadate o `ExposureTime` alebo `Gain` na pripojenej kamere, ktorej automatický režim nie je `Off`, odpoveď obsahuje:

- `frame_values` — hodnota z najnovšieho snímku kamery, pod názvom uzla (`ExposureTime` v µs, `Gain` v dB);
- `notes` — jedna veta na každý takýto uzol, v ktorej sú uvedené obe hodnoty.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Oba kľúče chýbajú, ak je automatický režim `Off` (uzol je vtedy skutočnou hodnotou kameryskutočná hodnota) a pre kameru, ktorá nie je pripojená k backendu. V prípade poľa poskytuje `ArraySession.status()["member_ae"]` hodnoty snímok všetkých členov naraz.

#### Limit expozície meraný DAQ (HTTP)

Pri pripojení svetelného senzora DAQ program Chloros obmedzuje expozíciu × zosilnenie kamery na základe nameraného denného svetla, aby nedošlo k preexponovaniu bieleho referenčného panelu. Tento limit platí len vtedy, keď je kalibračný cieľ v zornom poli danej kamery (zaznamenaný v jej snímke za posledných 10 s); ak nie je cieľ v zornom poli, namiesto toho sa používa bežná automatická expozícia kamery s vlastnou ochranou proti preexponovaniu, aby zvyšok scény nebol tmavší, než je potrebné. Neexistuje žiadny SDK wrapper; túto funkciu je možné načítať alebo prepnúť pre každú kameru pomocou `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (telo požiadavky POST s kódom `{"enabled": false}` túto funkciu pre danú kameru vypne). Odpoveď obsahuje `enabled`, `cap_us` (hornej hranice v režime živého náhľadu, `None` v neaktívnom režime, vrátane situácie, keď nie je v zornom poli žiadny kalibračný cieľ), `k_exp_g` (konštanta, pomocou ktorej sa strop vypočíta) a:

| Pole | Význam |
| --- | --- |
| `k_exp_g_source` | Ktorá konštanta sa používa: `camera` — vlastná konštanta tejto kamery, vypočítaná z laboratórneho bodu v jej kalibračnom balíku (kamery RGB); `fleet` — konštanta flotily, používaná v prípade, že sada neobsahuje laboratórny bod; `override` — prepis poľa nastavený na backende. |
| `k_exp_g_fleet` | Konštanta flotily, na porovnanie. |

### Synchronizované pole — `ArraySession` (Smart-Prep)

`connect_array` je **odporúčaný východiskový bod** pre konfigurácie s viacerými kamerami. V pozadí spúšťa kompletný pracovný postup Smart-Prep prostredníctvom grafického používateľského rozhrania:

1. **Analýza siete**(`/api/camera/array/recommend`) — nastaví**celý senzor** (bez zlučovania, bez orezávania oblasti záujmu) a umožňuje, aby sa snímková frekvencia znížila na úroveň, ktorú poskytuje tento hostiteľ a sieť, pričom sa zohľadňujú formáty pixelov a spôsob kompresie, ktoré bude Connect používať. Snímky sa nikdy nezmenšujú s cieľom udržať vysokú frekvenciu: nastavte menšiu hodnotu `width` / `height` alebo `binning=2` / `4`, aby ste vymenili rozlíšenie za snímkovú frekvenciu (koncový bod`alternatives` uvádza zoznam týchto možností s ich predpokladanými fps).
2. **Automatický výber úrovne** — `sim-capture-sim-emit` (všetky kamery exponujú pri rovnakom spúšťači, vysielanie je synchronizované tak, aby sa zdieľala linka, pričom snímková frekvencia klesá s klesajúcim podielom) v akejkoľvek sieti, v ktorej je možné kamery synchronizovať tak, aby zdieľali linku; `slip-emit-and-capture` (kamery sa striedajú na vedení, nie sú exponované súčasne) iba vtedy, ak to nie je možné.
3. **PTP povolené**štandardne — časové značky medzi kamerami sa zosúladia na jednom spoločnom časovači s presnosťou**~1 ms**, pričom backendom je [PTP grandmaster](#ptp-grandmaster--time_sync_status). Súčasná expozícia je zabezpečená hardvérovým spúšťačom M8 (**&lt; 100 µs** medzi modulmi), nie prostredníctvom PTP: PTP synchronizuje *časové značky*, nie expozície.
4. **Automatický výber formátu pixelov pre každú kameru** — čiernobiely → `Mono12`, farebný → `BayerRG12`.
5. **Inicializácia AE** — zaznamená aktuálny stav AE každej kamery, aby pripojenie počas prevádzky nezresetovalo expozíciu.
6. **Konfigurácia spúšťača GPIO** — `connect_array` aktivuje každú kameru (`TriggerMode=On`, `TriggerSource=Line2`), takže impulz hlavnej kameryimpulzy riadia podriadené kamery cez kábel M8. Ide o krok platný len pre pole: samostatná kamera otvorená pomocou `LatticeCamera` namiesto toho beží vo voľnom režime. Uvoľnenie poľa (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) vráti všetky kamery späť do režimu voľného chodu (`TriggerMode=Off`), pričom všetko ostatné zostane tak, ako to zistilo pripojenie: nastavenia spúšte a synchronizačnej linky (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; synchronizačný vodič hlavnej kamery späť na vstup), PTP a nastavenia kamery, ktoré sa počas relácie zmenili – pokiaľ sa nezmenili nastavenia `keep_settings=True` / `clear_settings=True` (pozri [Pamäť kamery](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Atribúty: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` a `packed_reason` (posledné štyri sú `None` na rukoväti od `attach_array`), `connect_stream` a `backend_url`. `connect_stream` je blok `stream` odpovede na pripojenie, tvar jedného pripojenia plus `members` (`{serial: {state, why?}}`): `streaming` po príchode prvej synchronizovanej skupiny poľa (backend na ňu čaká, s časovým obmedzením približne 10 s), alebo `no_frames` s posledným zlyhaním načítania každého člena ako jeho `why` a, v prípade jumbo paketov, náhradná hodnota 1500 bajtov v `note`. `None` na rukoväti `attach_array`.

#### Podpis `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` udáva, ako dlho volanie čaká na pripojenie, a je odoslané do
backendu ako jeho termín. Ak sa do tohto času nepripojí, volanie sa zruší: každá kamera, ktorú
otvorilo, sa vráti do stavu, v akom bola nájdená a uzavretá, žiadne pole nie je pripojené a
volanie vyvolá chybu `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, pričom sa uvádza
krok, do ktorého sa pripojenie dostalo). Volanie, ktoré vyvolalo výnimku, teda nikdy nenechá pole
pripojené za sebou. Pole s 9 kamerami sa pripojí za približne jednu minútu; hneď po
zapnutí kamier to môže trvať niekoľko minút. Pole, ktoré je aktívne, ale ešte nemá
synchronizovanú skupinu snímok, je vrátené s chybou `connect_stream["state"]`
`no_frames`. Protokol backendu obsahuje jeden riadok s časom pre každý krok pripojenia
(`[array-connect <id>] <step>: <s> s …`) a posledný riadok uvádzajúci najpomalší.

Hodnoty `force_tier`:
- `"sim-capture-sim-emit"` — skutočná simultánnosť (všetky kamery sa spúšťajú na rovnakom okraji hodín).
- `"sim-capture-ftd-stagger"` — flexibilné časové posunutie (kamery vysielajú v mierne posunutých časoch, takže sa pakety na vedení sériovo usporiadajú).
- `"slip-emit-and-capture"` — sekvenčné snímanie pre každú kameru (bez časovej synchronizácie). Vyberá sa automaticky len vtedy, keď nie je možné spomaliť kamery natoľko, aby mohli zdieľať vedenie súčasne.

`wire_ceiling_mbps` prepisuje **trvalý rozpočet hostiteľa na šírku pásma** v MB/s — jediné
číslo, od ktorého závisí alokácia celého poľa. Nechajte nastavenie `None` na automaticky zistenú
hodnotu. Znížte ju, ak pole hlási poškodené rámce GVSP: automatická hodnota je odvodená
od inzerovanej rýchlosti spojenia sieťovej karty, ktorá nadhodnocuje USB adaptéry, úzke PCIe kanály a
vyťažené zdieľané štruktúry — a toto nadhodnotenie sa prejavuje skôr ako poškodené rámce než ako
viditeľne pomalé spojenie. Hodnota sa ukladá do bloku zachytávania poľa v projekte, takže jej
opätovné otvorenie alebo neskoršie nastavenie `connect_array` ju obnoví rovnako ako akékoľvek iné nastavenie poľa.
Pozrite si [Stav poľa](#array-health--which-subsystem-is-losing-frames).

#### Komprimovaný 12-bitový prenos (`packed=`)

Pri plnom využití senzorov je snímková frekvencia poľa daná rýchlosťou, akou môže každá kamera odosielať
svoje snímky. 12-bitové formáty (`BayerRG12`, `Mono12`) sa zvyčajne prenášajú ako 16-bitové
slová; komprimovaný prenos odosiela tých istých 12 bitov v 1,5 bajte na pixel a
dekomprimuje ich na hostiteľskom zariadení, takže odoslanie každého snímku trvá o 25 % menej času — približne
**o 33 % viac snímok za sekundu pri plnom využití snímača** (merané: 10,7 → 14,0 fps na
sústave s 9 kamerami). **Uložené dáta sú identické**: rovnaké hodnoty, rovnaké 16-bitové
súbory, rovnaké metadáta.

- `packed="auto"` (predvolené nastavenie) komprimuje len vtedy, ak to pomáha **a** hostiteľ si môže
  dovoliť dekompresiu: pri pripojení backend zmeria svoju vlastnú rýchlosť dekompresie,
  počet procesorových jadier, aktuálne zaťaženie a sieťovú kapacitu. Stolný počítač s voľnými jadrami
  komprimuje pri plnom vyťažení senzorov; vyťažený alebo nízkoenergetický hostiteľ (jednočipové počítače,
  vstavané radiče) streamuje v nekomprimovanej podobe. Malé oblasti záujmu (ROI) a zlučované snímky prinášajú len malý prínos.
- `packed="off"` (alebo `False`) nikdy nekomprimuje — na vylúčenie tejto možnosti počas diagnostiky,
  alebo na hostiteľskom počítači, ktorý musí vyhraďovať svoje CPU pre inú prácu.
- `packed="on"` (alebo `True`) vždy komprimuje 12-bitové formáty.
- Akákoľvek iná hodnota vyvolá `ValueError`. 8-bitové a 16-bitové formáty sa nikdy
  nekomprimujú.

`analyze_array_network(..., packed="auto")` sa riadi rovnakým rozhodnutím,
takže jeho fps je rýchlosť, pri ktorej bude pole bežať (odpoveď obsahuje
`packed_transport` a `packed_reason`); jeho predvolená hodnota `pixel_format` je tá, ktorú
uplatňuje connect (Mono12 / BayerRG12), a nie 8-bitová.

Rozhodnutie a jeho dôvod sa nachádzajú v relácii:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` uplatňuje rovnaké prepisovanie na každé
pole v projekte (predvolené nastavenie `None`, čo znamená `"auto"`: projekt neukladá žiadne
nastavenie kompresie pre jednotlivé polia).
Pre predvolené nastavenie pre celý počítač nastavte `CHLOROS_ARRAY_PACKED12=off` (alebo `on`) v
prostredí backendu; explicitné nastavenie `packed="on"` / `"off"` má stále prednosť.
`/api/camera/array/status` hlási `packed_transport` a `packed_decision`.

#### Očakávané rýchlosti snímkovania

Namerané rýchlosti snímkovania poľa (hardvérovo synchronizované, 12-bitové; plný senzor = 2048 × 1536). Použite ich na kontrolu správnosti projektov `status()["actual_fps"]`; `analyze_array_network()`, ktoré odhadujú rýchlosť pre váš vlastný hostiteľ a sieť.

| Nastavenie | Rýchlosť snímok poľa |
| --- | --- |
| Plný senzor, hostiteľská sieťová karta PCIe 10 GbE, komprimovaný prenos (`packed="auto"` na stolnom počítači) | ~14 fps |
| Plný senzor, hostiteľská sieťová karta PCIe 10 GbE, nekomprimovaný prenos (`packed="off"`) | ~10,5 fps |
| Plný senzor, 8 kamier na adaptéri USB 10 GbE | ~3,8 fps |
| Plný senzor, hostiteľské pripojenie 1 GbE | 1,2–1,7 fps |
| Binning 2 × 2 (`binning=2`), hostiteľská sieťová karta PCIe 10 GbE | ~30–33 fps |

- Pri pripojení cez PCIe 10 GbE sú pri plnom využití snímača limitom **kamery**, nie sieť: 4 aj 8 kamier bežia rovnakou rýchlosťou.
- **`target_fps` je spúšťacia frekvencia a pole nikdy nebeží rýchlejšie.** `target_fps=1` prevádzkuje pole rýchlosťou 1 fps; `None` (predvolené nastavenie) ho prevádzkuje najvyššou rýchlosťou, ktorú zvládne hostiteľ a sieť. Akákoľvek hodnota nad touto rýchlosťou sa zníži na túto úroveň (pozri [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` z `4294` v `status()` nie je rýchlosťou spojenia.** Znamená to, že systém Windows nedokázal prečítať rýchlosť adaptéra (ovládač nahlásil najväčšiu 32-bitovú hodnotu), čo je typické pre USB ethernetové adaptéry. Použiteľná rýchlosť USB adaptéra v systéme Windows je v každom prípade obmedzená na 200 MB/s (pozri [Stav poľa](#array-health--which-subsystem-is-losing-frames)); pokiaľ ide o jeho ladenie, pozrite si časť „USB 10GbE adaptéry“ v referenčnom materiáli k CLI v sekcii [Nastavenie a ladenie hostiteľskej sieťovej karty](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Obmedzenie USB platí aj v systéme Linux: 400 MB/s na x86_64, 200 MB/s na NVIDIA Jetson.** Adaptér sa identifikuje prostredníctvom sysfs (v prípade potreby sa použije `ethtool -i`), takže USB 5 GbE alebo 10 GbE adaptér je obmedzený na tento limit, a nie na svoju menovitú rýchlosť 500 / ~1060 MB/s. Hodnota pre x86_64 bola nameraná v systéme Linux: ten istý adaptér, ktorý v systéme Windows poškodzuje snímky pri rýchlosti nad 200 MB/s, bezchybne spracovával pole štyroch kamier pri každom limite až do 480 MB/s. Na platforme NVIDIA Jetson fungoval ten istý adaptér bezchybne len pri 200 MB/s a pri vyšších rýchlostiach dochádzalo k prerušovanému strácaniu rámcov. Ak váš hostiteľský počítač spracováva viac rámcov bez `gvsp_corrupt`, zvýšte túto hodnotu pomocou `wire_ceiling_mbps`. V systéme Linux je adaptér pripojený k portu USB 2.0 obmedzený na 30 MB/s; pripojte ho k portu USB 3.

#### Nadmerné využívanie (minimálna rýchlosť na kameru)

Pole funguje v akejkoľvek sieti; záleží na snímkovej frekvencii. Sim-emit pacing prideľuje každej kamere rovnaký podiel z rozpočtu pre prenos bez kolízií a všetky kamery stále snímať pri rovnakom spúšťači, takže na pomalom pripojení je tento podiel — a snímková frekvencia — malý: 8 plnohodnotnýchrozlíšení 12-bitových kamier na 1 GbE bez jumbo rámcov dostane po 6,5 MB/s, čo je približne **1 fps**(približne 1,3 fps v komprimovanej podobe). Podiel je obmedzený na najpomalšiu rýchlosť, na ktorú môžu byť kamery synchronizované (`per_cam_floor_bps`: pri pripojení sa prečíta rozsah GevSCPD každej kamery a nikdy neklesne pod 1 MB/s). Ak si sieť vyžaduje nižšiu rýchlosť, pripojenie prejde na**sériový**prenos (`slip-emit-and-capture`): kamery sa striedajú pri plnej rýchlosti spojenia, takže každý rámec dorazí, ale členovia**nie sú vystavení súčasne**— úroveň odpovede a blokátory (`SERIAL: ...`) to potvrdzujú; odovzdajte `force_tier="sim-capture-sim-emit"`, aby ste odmietli tento náhradný režim. Veľkosť snímky neovplyvňuje zdieľanie, takže**zlučovanie a ROI zvyšujú frekvenciu snímok, ale nemenia to, či sa pole zmestí**. Len linka, ktorá nedokáže prenášať pole pri žiadnom tempe, či už súčasne alebo striedavo,**je preťažená** — režim zlyhania je strata paketov GVSP, nie nižšia snímková frekvencia. Riešenia: menej kamier, jumbo rámce end-to-end alebo rýchlejšia sieťová karta.

- Odpovede `analyze_array_network()` a `/api/camera/array/connect` obsahujú `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` a `per_cam_floor_bps`. Ak je hodnota `oversubscribed` pravdivá, projekcia **nastaví polia fps na nulu** (`achievable_fps_max` / `fps_bright` / `fps_dark`) namiesto toho, aby hlásila zavádzajúcu hodnotu, ktorá naznačuje pomalú, ale fungujúcu rýchlosť.
- `POST /api/camera/array/connect` akceptuje parameter tela `pin_resolution` (**len pre HTTP — nie je to kľúčový argument SDK**; `connect_array` ho nezverejňuje). Ide o prísny režim: preťažené pripojenie s nastaveným `pin_resolution` je**tvrdo odmietnuté** s chybou, ktorá uvádza všetky možné riešenia. Bez neho sa pripojenie uskutoční s varovaním, že pole stratí pakety. V oboch prípadoch pripojenie zachová požadovanú veľkosť rámca — nikdy ho samo od seba nezlučuje ani neorezáva.
- Iba na testovanie: nastavte v prostredí backendu parameter `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1`, aby sa odmietnutie zmenilo na výrazné varovanie – pripojíte sa aj tak a akceptujete stratu paketov.

#### Stav poľa – ktorý subsystém stráca rámce

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) prenáša aktívny blok `health` na
pripojenom poli, ktorý sa prehodnocuje v priebežnom **10-sekundovom** okne. Rozdeľuje stratu rámcov
na dve príčiny, ktoré si vyžadujú opačné riešenia, namiesto jednej miery „neúplnosti“, ktorá
nešpecifikuje ani jednu z nich:

| Pole | Čo to znamená | Ktorý subsystém |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (na sériové rozhranie) | Snímka **dorazila a bola štrukturálne poškodená**— strata paketov GVSP. |**Sieť**: kapacita linky, tempo, prstenec prijímača NIC, MTU |
| `never_arrived_rate_pct` (podľa sériového čísla) | Rámec **vôbec neprišiel**— kamera sa nespustila alebo z nej nič nevyšlo. |**Spúšťač / synchronizácia**: kábel M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Najhoršia rýchlosť kamery pre každú z nich. | — |
| `per_cam_rate_pct` | Kombinovaná miera neúplnosti na kameru (obe príčiny spolu). | — |
| `stable_for_seconds` | Ako dlho zostala každá kamera pod 0,01 %. | — |

Okrem `health` uvádza ten istý záznam aj číslo, na ktorom visí celá alokácia:

| Pole | Čo to znamená |
| --- | --- |
| `wire_ceiling_mbps` | Platný trvalý rozpočet šírky pásma hostiteľa, MB/s. |
| `wire_ceiling_source` | Odkiaľ táto hodnota pochádza, slovami — napr. `USB-capped 200 MB/s (was theoretical 1062; …)` alebo `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, keď ho nastavil `wire_ceiling_mbps=`. |
| `nic_is_usb` | `true` pre USB ethernetový adaptér. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Význam:** hodnota `gvsp_corrupt_rate_pct` odlišná od nuly a hodnota `never_arrived_rate_pct` rovná 0 znamenajú, že
spúšťanie a synchronizácia kábla sú v poriadku a 100 % strát je na sieťovej trase — znížte
`wire_ceiling_mbps` a znovu pripojte. Opačný vzor poukazuje skôr na synchronizačný kábel alebo
spúšťaciu linku.

> **`target_fps` nie je kľúčovým faktorom pre poškodené rámce.** Tempo GevSCPD sa zapisuje raz pri
> pripojení, takže zníženie frekvencie spúšťača mení pracovný cyklus, nie však
> rýchlosť súbežného vysielania. Zmerané zníženie požiadaviek o 5× neprinieslo žiadne zlepšenie, zatiaľ čo
> zníženie maximálnej rýchlosti z 240 na 200 MB/s znížilo podiel poškodených rámcov na tom istom zariadení z 10,4 % na
> 0,00 %.

> **Bežiace pole sa nemôže samo preplánovať na firmvéri LATTICE (M3C / M3M).** Nemôže
> tento problém vyriešiť sama; odpojte a znovu pripojte, aby nástroj na výber času pripojenia preplánoval na základe
> nového stropu.

**USB ethernetový adaptér je sondou obmedzený na 200 MB/s** bez ohľadu na jeho
označenia: tabuľka účinnosti, ktorá prevádza rýchlosť spojenia na trvalú hodnotu, je
odvodená od PCIe, a USB sieťová karta oznamuje svoju rýchlosť ethernetového spojenia, pričom je obmedzená
USB zbernicou a jej ovládačom. Toto obmedzenie je absolútne, nie frakčné — USB 1 GbE adaptér
dosahuje ~80 MB/s a nie je týmto obmedzením ovplyvnený. Platí to pre Windows aj Linux (vrátane Jetsonu);
pozri [Očakávané rýchlosti prenosu rámcov](#expected-frame-rates), kde bolo toto meranie vykonané.

#### Metódy `ArraySession`

| Metóda | Popis |
| --- | --- |
| `status(timeout=10.0)` | Aktuálny stav bežiaceho poľa: `target_fps` / `target_fps_auto` (frekvencia spúšťania a či ju ovláda regulátor frekvencie), `target_fps_clamped_reason` (prečo je fixná frekvencia udržiavaná na nižšej úrovni, inak `None`), `actual_fps` (meraná; 0, keď sa slučka uchopenia vzdala alebo zablokovala), `recommended_fps`, `streaming` (False, keď sa slučka zachytávania vzdala — skontrolujte toto, nielen `last_error`; zostáva True aj po reštarte nastavení, čo signalizuje `restarting`), `stalled` (True, pokiaľ cyklus snímkovania beží, ale nezverejnil žiadnu skupinu snímok približne počas desiatich snímkových periód, 2–30 s; `last_error` to signalizuje až po 60 s), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` len vtedy, keď bolo pole pripojené cez PTP **a** každý člen naposledy prečítal `PtpEnable` v stave „on“; nie je to to isté ako stav „locked“, ktorý meria `ptp_epoch`), `ptp_configured` (čo požadovalo pripojenie), `ptp_off_members` / `ptp_unread_members` (členovia, ktorí naposledy čítali hodnotu `PtpEnable` ako vypnuté / neodpovedali), `member_recovery` (`None` alebo člen, ktorý stratil napájanie alebo spojenie: `state`, `waiting`, hoci neodpovedá, `reopening`, potom `recovered`, spolu s `members` a `why` — pole ho opäť otvorí, keď opäť odpovie, a vráti mu jeho PTP času pripojenia, spúšťač a zapojenie synchronizačnej linky, nastavenia tempa a kamery, potom opäť vysiela bez nových hodnôt `connect_array`) a `ptp_epoch` (zmerané), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` znamená, že systém Windows nedokázal zistiť rýchlosť adaptéra — pozri [Očakávané rýchlosti snímok](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (či je reťazec žiarivosti na jednotlivé pásma použiteľný a ktorý člen ju narušil) a `band_clamp` — zatiaľ čo živý náhľad vykresľuje pole: podiel osvetlených pixelov, ktoré dekonvolúcia obmedzila na 0, pre každý prvok (`members`: `{serial: [band0, band1, band2]}` v poradí pásiem R, G, B kamery) a pre každý pásmo, ktoré kompozit zobrazuje (`composite`: `{band name: share}`), pričom `warn` udáva podiel, ktorý grafické rozhranie označuje ako „dáta pásma s nízkou spoľahlivosťou“ (merané približne raz za sekundu; `None`, ak nebeží živý náhľad) a `member_ae` — pre každý prvok expozícia (`exposure_us`) a zosilnenie (`gain_db`) z posledného zverejneného snímku, získané priamo zo snímku (`read_nodes` z `ExposureTime` / `Gain` vráti poslednú hodnotu zapísanú počas behu automatickej expozície, nie aktuálnu hodnotu). |
| `set_member_settings(serial, **kwargs)` | Zapíše nastavenia jednej kamery, keďže `CameraSession.set_settings()` ich preberá (expozícia, zosilnenie, cieľová jasnosť, ...); člen zostáva v poli. Vyvolá výnimku `ChlorosConnectError` pre sériový prístroj, ktorý nie je členom. Pozri [Expozícia na členoch poľa](#exposure-on-array-members). |
| `capability(timeout=10.0)` | Čo stanovila funkcia smart-prep (úroveň snímania, veľkosť snímky, `pixel_format`), blok živého prenosu `health` a rozpočet hostiteľa na prenos. Pozri [Stav poľa](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Zmena frekvencie spúšťania počas prevádzky poľa bez opätovného pripojenia. Číslo ju fixuje (regulátor frekvencie ju ponechá bez zmeny); `None` ho vráti do automatického režimu. Vráti `{requested, target_fps, target_fps_auto, clamped}` — `clamped` je True, ak bola frekvencia znížená na úroveň, ktorú hostiteľ a sieť dokážu prenášať. Pozri [Intervalové snímanie pre prieskumy pomocou dronov](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Jeden snímok z každej kamery, **všetky z jedného spúšťača**, ako numpy polia (neukladajú sa žiadne súbory). Vráti `GrabResult` (alebo `dict`): `{serial: array}` alebo `{serial: (array, metadata)}` s `with_metadata=True`. `.group_id` je spúšťač, ktorý snímky zdieľajú; `.errors` priraďuje každú vynechanú kameru k dôvodu (kamera, ktorá nemôže byť obslúžená pri danom spúšťači, je vynechaná a nikdy nie je nahradená inou). `processing` je `raw` / `debayered` / `radiance` / `reflectance` / `display`. To isté čítanie ako v projektovom API [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) — jedna spoločná implementácia — takže každé kľúčové slovo tam (`band`, `align`, `bit_depth`, `refl_max`) tu funguje. Vyžaduje `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iterátor, ktorý vráti jeden výsledok `grab()` na jeden tik (len kľúčové argumenty), s tempom `fps`, ohraničený hodnotou `duration` (sekúnd) a/alebo `count` (`None` = neobmedzený). **Každá vrátená množina je novým spúšťačom:** tik, ktorého najnovším spúšťačom je množina, ktorú už vygeneroval, čaká na ďalší spúšťač namiesto toho, aby sa opakoval, takže tok beží pri nižšej z hodnôt `fps` a vlastnej snímkovej frekvencii poľa. Vyvolá výnimku `FrameFetchError` namiesto toho, aby prestal komunikovať: okamžite, keď každá kamera z nejakého dôvodu odmietne, že by sa nasledujúci tik mal opakovať, po 60 sekundách tikov bez snímky a po 60 sekundách bez nového spúšťača („žiadny nový spúšťač počas 60 sekúnd“). Rovnaká implementácia ako `ArrayHandle.frame_stream()`. Pozri [Živé snímky vo vašom vlastnom potrubí](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Spustí vlastné nepretržité snímanie backendu (režim Continuous v grafickom rozhraní) → [`ContinuousHandle`](#continuoushandle). Ukladá **každú** synchronizovanú skupinu, ktorú pole publikuje, tak ako by ju uložil `capture()` (rovnaké nastavenia exportu), až kým nie je zastavené alebo kým nie je spustený `max_captures` / `duration_s` (`None` = kým sa nezastaví). Vyvolá výnimku `ChlorosConnectError`, ak už pre toto pole beží nejaký proces. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Jedna synchronizovaná skupina zachytávania. Vráti `CaptureResult` (zoznam slovníkov snímok + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Predchádzajúci výsledok`group_seq` ako `after_seq`, aby ste získali nasledujúcu synchronizovanú skupinu po ňom, namiesto čakania na novú. Ovládacie prvky exportu nižšie. |
| `capture(..., smart=True)` | **Inteligentné zachytenie** — čaká, kým sa AE ustáli na všetkých kamerách, a potom spustí zachytenie. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Najrýchlejšie zachytenie: iba surové dáta + priradená hodnota DAQ (+ voľný kombinovaný index). Zodpovedá tlačidlu „Najrýchlejšie zachytenie“ v grafickom rozhraní. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Jednorazové / nepretržité / intervalové zachytenie v jednej ohraničenej slučke. Vráti `list[CaptureResult]`.**Vyžaduje `count` a/alebo `duration_s`**, aby sa ukončilo (SDK nemá Ctrl+C). Takisto sa ukončí predčasne, po prechode, keď prechod nahlási plný disk (záznam `.failed` s `disk_full`); tento prechod je posledným výsledkom. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Spustí nahrávanie živého zobrazenia kombinovaného indexu do videa/GIF → `RecorderHandle`. Video je vo formáte Motion-JPEG `.avi` (nie je potrebný ffmpeg). Ak nie je otvorené žiadne grafické rozhranie, backend ho vykreslí: `expr` (indexový výraz v názvoch pásiem poľa, napr. `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`), alebo štandardne NDVI poľa. Jeden kompozitný záznamník na jedno pole. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Spustite sériové snímanie surových Bayerových dát s vysokou snímkovou frekvenciou → `RecorderHandle`. Spracujte offline pomocou `build_video()`. `orient=True`: výstupy programu `build_video()` sú otočené podľa orientácie kamery (výstupy jednotlivých kamier podľa orientácie danej kamery, kombinovaný index podľa orientácie hlavnej kamery); surové snímky zostávajú v pôvodnej orientácii a manifest snímkovej série zaznamenáva orientáciu každého člena. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Offline spracovanie uloženého surového sériového záznamu na kalibrované video(-á). Blokuje sa až do dokončenia (`wait=True`) a vráti výsledok zostavenia: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — pozri [výsledok zostavenia](#the-build_video-result) nižšie. |
| `build_video_status(job_id, timeout=15.0)` | Kontroluje stav offline úlohy kompilácie: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Uvoľní celé pole — **ak ho otvorila táto relácia** (`connect_array`): potom nasleduje `force_disconnect()` a vyvolá výnimku `ChlorosConnectError`, ak uvoľnenie nie je potvrdené (blok `with` nikdy nevyvolá výnimku pri ukončení). Pri rukoväti z `attach_array` ponechá pole v prevádzke a vypíše jeden riadok do stderr s touto informáciou: pole môže patriť k GUI alebo k inému skriptu. Každá kamera zostane vo voľnom prevádzkovom režime a inak tak, ako ju našlo pripojenie; `keep_settings=True` uloží jej nastavenia do vlastnej pamäte kamery, `clear_settings=True` obnoví túto pamäť na továrenské nastavenia (pozri [Pamäť kamery](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Uvoľní celé pole bez ohľadu na to, kto ho otvoril. Použite ho na pripojenú rukoväť, ak máte na mysli ukončenie poľa (napr. s cieľom opätovného pripojenia s odlišnými nastaveniami). Vráti sa, akonáhle backend potvrdí uvoľnenie, alebo ak nie je čo uvoľniť (pole už neexistuje alebo nebeží žiadny backend); ukončenie, ktoré vypršalo alebo ktoré backend odmietol, vyvolá výnimku `ChlorosConnectError`. S ktoroukoľvek vlajkou vráti to, čo získala pamäť každej kamery (`camera_memory`), inak vráti `None`. |

`capture()` – ovládacie prvky exportu (rovnaký koncový bod, aký používa GUI/CLI):

- `processing` / `levels` — `processing="all"` (alebo `levels=["raw","radiance",…]`) uloží každý príslušný typ exportu pre každú kameru; jedna hodnota `processing` ukladá len túto úroveň.
- `aligned` — **štandardne zapnuté (`None`), keďže zachytávanie toho istého poľa aplikáciou Chloros je**(jeho nastavenie zachytávania**Aligned** pre každé pole). Ne-surový export každého člena je deformovaný do rámca hlavnej kamery podľa [profilu zarovnania](#array-alignment) a orezané na spoločný prekrývajúci sa úsek členov, takže súbory sú pixelovo zarovnané a majú rovnakú veľkosť, orezanie a tagy `Alignment*` ako v aplikácii (v rámci oblasti záujmu (ROI) sa transformácia prispôsobí oknu); `XMP-mapir:AlignmentJSON` uvádza `"applied": true` spolu s jeho `crop_roi`. Surový súbor zostáva bez deformácie v pôvodnej veľkosti, ale transformáciu nesie v metadátach. `aligned=False` ukladá každý súbor v pôvodnej veľkosti, bez deformácie; `aligned=True` je predvolené nastavenie. `continuous()`, `capture(smart=True)`, `capture_repeated()` a transformácie na úrovni projektu `ArrayHandle.capture()` / `capture_smart()` majú rovnaké predvolené nastavenie. Nezávisle od toho, či je deformácia aplikovaná alebo nie, každý súbor člena zaznamenáva transformáciu `Alignment*` svojej kamery, akonáhle má pole profil zarovnania — s výnimkou kamery, ktorú profil nezarovná (bola vynechaná alebo bola v dôsledku neúspešného riešenia vynútene nastavená na identitu), a referenčná kamera, ak nie je zarovnaná žiadna iná kamera: tieto kamery nemajú žiadne značky `Alignment*`. Ak pole nemá profil, záznam sa uloží nezarovnaný a výsledok to uvádza: `result.alignment` je `{"requested": True, "applied": False, "warnings": [...]}`, jedno varovanie za každý dôvod, pre ktorý nebolo možné použiť deformáciu (chýbajúci profil, súbory s rôznymi rozlíšeniami, pole, ktoré nie je možné, snímka, na ktorú sa profil nedá zmapovať). `applied` je `True` len vtedy, ak každý export, ktorý požadoval deformáciu, ju aj dostal. Automatické zarovnanie poľa vytvorí profil, akonáhle sa ustáli expozícia; `ArrayHandle.calibrate_alignment()` / `align-calibrate` v CLI ho vytvoria na požiadanie. `result.oriented` udáva, či `orient=True` otočil produkty.
- `render_index=False` — preskočiť prekrývanie indexu vegetácie pre každú kameru; predvolene sa vykresľuje tam, kde je to nakonfigurované.
- `daq` / `daq_tolerance_ms` — hodnota z DAQ sa štandardne ukladá ako súbor typu „sidecar“ s názvom `.daq`, aj keď to žiadna zvolená úroveň nevyžaduje; v takom prípade to `daq=False` preskočí, a `daq_tolerance_ms` nastavuje toleranciu zhody (predvolene 500). `force_daq=True` je starší zápis predvoleného nastavenia. Pozri [Hodnoty DAQ, ktoré zachytáva záznam](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
- `orient=True` — zapne pre každého člena **výstupy**(`display`, `radiance`, `reflectance`, index na kameru a kompozit) podľa orientácie kamery po deformácii pri zarovnaní: podľa orientácie**hlavnej kamery**v čase, keď boli deformované do jej rámca (predvolené nastavenie `aligned` — členy zostávajú zarovnané), inak podľa orientácie ich vlastnej kamery.**Predvolené nastavenie: každý súbor prvku v natívnom formáte**, ako je to vždy pri snímkach v poli; `raw` a `debayered` zostávajú v natívnom formáte v oboch prípadoch. Každý súbor zaznamenáva orientáciu rámca, v ktorom sa nachádzajú jeho pixely (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — surový súbor, orientácia vlastného fotoaparátu), a XMP súbor `Alignment*` zachováva transformáciu natívneho rámca, aplikovanú pred orientáciou; surový súbor obsahujúci túto transformáciu zaznamenáva aj `ChlorosAligned*` (orientáciu referenčného snímku), teda orientáciu výsledného snímku deformovaného do orientácie referenčného snímku. Spracovanie otočí každý výsledný snímok podľa orientácie, ktorá patrí k jeho pixelom: podľa zarovnanej orientácie, ak deformuje surový súbor (zapnuté zarovnanie poľa), alebo podľa orientácie samotného fotoaparátu, ak nie. `capture_repeated()` a `capture_fastest()` ju prenášajú ďalej; na úrovni projektu ju preberajú aj `ArrayHandle.capture()` / `capture_smart()`.

**Kompresia TIFF (ovládací prvok HTTP):** `ArraySession.capture()` neodosiela kľúč `compression`, takže platí predvolené nastavenie backendu — `POST /api/camera/array/capture` číta parameter tela `compression`, `"none"` štandardne: nekomprimované, pixely senzora presne tak, ako boli načítané (~6,3 MB na snímku v plnom rozlíšení). `"deflate"` (bezstratná kompresia zlib L1 + horizontálny prediktor, ~4,1 MB/snímka) zapisuje menšie súbory pomalšie — obe sú bezstratné a pri importe sa čítajú identicky. SDK pre to neposkytuje žiadny kwarg; únikovou cestou je `chloros-cli lattice array-capture --compression deflate` alebo surové HTTP. DEFLATE blokuje Python GIL, takže komprimované zápisy sa neparalelizujú medzi vláknami zapisovača pre jednotlivé kamery — nepretržité snímanie v plnom rozlíšení pri rýchlosti snímača vyžaduje nekomprimované predvolené nastavenie. Podrobnosti: [Referencia CLI → array-capture](cli-reference.md).**Po sebe idúce skupiny (protokol HTTP):** koncový bod vráti `group_seq`; slučka, ktorá ho pri ďalšej požiadavke pošle späť ako `after_seq`, získa ďalšiu synchronizovanú skupinu — vrátane tej, ktorá bola zverejnená počas ukladania predchádzajúceho záznamu — namiesto čakania na skupinu zverejnenú po požiadavke, pričom `skipped_groups` hlási všetky skupiny, ktoré slučka vynechala. `capture_repeated`, `--continuous` v CLI aj režim Continuous v GUI to všetko robia; intervalové prechody zostávajú aktuálne v každom naplánovanom okamihu.**Prepísanie exportu na úrovni jednotlivých členov (len HTTP):**ten istý koncový bod prijíma aj `exclude_serials` (zoznam – vyradenie členov z uloženej sady; pole sa naďalej spúšťa ako jedna synchronizovaná skupina a vylúčení členovia sa vracajú v `excluded`), `serial_levels` (prepísania na úrovni kamery `{serial: [level tokens]}`) a `serial_index` (prepísania prekrývania indexov na úrovni kamery `{serial: bool}`). Ide o parametre telesa zodpovedajúce grafickému rozhraniu (GUI) a**zatiaľ nie sú kľúčovými argumentmi SDK**; členovia, ktorí chýbajú v mapách, sa vrátia k hodnotám platným pre celé pole `levels` / `render_index`.

##### Kontrola preskočených kamier — `CaptureResult.skipped`

`ArraySession.capture()` vráti `CaptureResult`, čo je podtrieda `list`: prechádzajte ju, indexujte ju, `len()` — všetky existujúce vzory naďalej fungujú. Nový kód môže skontrolovať atribút `.skipped`, aby zistil, ktoré kamery boli vylúčené a prečo. Najbežnejším prípadom sú kamery RGB v poli so zmiešanými filtrami, keď požiadate o `processing="radiance"` alebo `"reflectance"` — žiarivosť na jeden Bayerov pixel nemá pre širokopásmový senzor žiadny význam, takže backend tieto kamery preskočí, aby nevytváral nezmyselné údaje.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Tokeny dôvodov majú formát `<level>-not-applicable-to-rgb-cam` (jeden záznam na každú preskočenú úroveň, pričom každý obsahuje `level`). Kamera, ktorej kalibračný balík nie je načítaný, preskočí `radiance` / `reflectance` s `<level>-needs-calibration (connect the camera so its cal pack loads)`, a ak nie je k dispozícii použiteľná hodnota odrazivosti zo zariadenia DAQ, preskočí sa na `reflectance-skipped-no-dls`. Ďalšie preskočenia súvisiace s odrazivosťou sú `reflectance-skipped-no-fresh-dls` (nie je k dispozícii žiadna nová hodnota dopadajúceho žiarenia), `reflectance-skipped-bound-daq-unavailable (…)` (nebolo možné dosiahnuť pripojený DAQ) a `dls-uncalibrated-band-<nm>` — pásmo leží prevažne mimo radiometricky kalibrovaného rozsahu svetelného senzora DAQ (~374–974 nm), takže absolútne rozdelenie podľa odrazivosti na základe DAQ je odmietnuté a snímka sa výrazne zníži na úroveň odozvy senzora. Zo sériovo vyrábaných modelov to spúšťa iba F988; podporovanou cestou pre túto kameru je pracovný postup s panelom odrazivosti.

##### Súbory, ktoré sa nepodarilo zapísať — `CaptureResult.failed`

Záznam, pri ktorom sa podarilo zapísať len niektoré súbory, stále vráti: zoznam obsahuje všetky súbory, ktoré **boli** zapísané, a `.failed` uvádza zvyšok, jeden slovník na súbor s `serial` (`None` pre kombinovaný kompozit a sidecar poľa `.daq`), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, alebo `match` — pôvod súboru `.daq` v súbore `.match.json`, keď bol súbor `.daq` zapísaný, ale jeho sidecar nebol) a `error`. Na plnom disku obsahuje záznam aj `disk_full: True`; každé neskoršie zachytenie na tento disk by zlyhalo rovnakým spôsobom, čo je dôvod, prečo sa `capture_repeated()` zastaví práve tam. Pri čiastočnom zachytení sa nevyvolá žiadna výnimka a súbor v `.failed` nezostal na disku napoly zapísaný. `.failed` je pri čistom zachytení prázdny.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### DAQ čítajúci záznam ukladá súbory — `.daq_files` a `.daq_matches`

Vždy, keď je DAQ priradený ku kamere alebo sústave, alebo je zaradený do združenia, každý záznam ukladá hodnotu merania smerom nadol, ktorú použil: súbor `.daq` v podsložke `daq/` zložky so záznamom, spolu so súbormi `.match.json` a `.cameras.json` (ktorého hodnota to je). To platí aj pre surové alebo debayered snímky a ide o ten istý súbor bez ohľadu na to, či snímka pochádza z grafického rozhrania (GUI), príkazového riadku (CLI) alebo vývojárskeho balíka (SDK). `daq=False` to vynechá v prípade snímky, ktorej úrovne nepoužívajú DAQ (úroveň typu `radiance`, `reflectance` alebo `display` ju stále zapisuje); `daq_tolerance_ms` posúva hranicu tolerancie pre daný zachytenia a nikdy nie na použité namerané hodnoty. Ktorý DAQ: ten viazaný na kameru alebo pole (výber svetelného senzora v grafickom rozhraní, viazanie projektu alebo `chloros-cli daq pool-bind`), inak najnovší, aký má backend k dispozícii. CLI uvádza ovládacie prvky `--no-daq` a `--daq-tolerance-ms` ([Referencia CLI](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

Záznam uloží hodnotu zo svetelného senzora, ktorú použil, ako `.daq` spolu s hodnotou `<file>.daq.match.json` vedľa nej, ktorá udáva, ako bola táto hodnota viazaná na expozíciu: `method`, `gap_ms` (od expozície k najbližšej hodnote DAQ) a `in_tolerance` (v rámci 500 ms, resp. `daq_tolerance_ms` záznamu). XMP súbor súboru odrazivosti obsahuje rovnaký záver ako `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` a `Chloros:DlsInTolerance`.

Záznam čaká na prvé meranie z DAQ po expozícii, aby mohol zlúčiť merania na oboch stranách tohto merania (`method` `interpolated`). DAQ-E odosiela hodnotu každých 0,4 s pri dennom svetle a každých 1–2 s v interiéri, takže `capture()` sa vráti až približne o jeden a pol takého intervalu neskôr; samotná expozícia nie je oneskorená. Na DAQ, ktorý prestal odosielať údaje, sa nečaká. Hodnota z DAQ-E predstavuje priemernú intenzitu osvetlenia za interval od predchádzajúcej hodnoty, takže je umiestnená uprostred tohto intervalu: `gap_ms` sa meria k tomuto stredu a `.match.json`&#x27;`bracket_stamps_ns` poskytuje vlastné časové značky odčítaní. `ArraySession.continuous()` čaká bez spomalenia snímania. Zadnénečakajú, takže si zachovávajú svoju frekvenciu a sú priradené k najbližšej hodnote pred expozíciou.

Hodnota nikdy nevyprší, takže žiadny snímok sa nestratí kvôli oneskoreniu — namiesto toho je označená:

- **Prúd z DAQ sa zastavil** — použije sa jeho najnovšia hodnota a `gap_ms` udáva, ako je stará (`in_tolerance: false` viac ako 500 ms).
- **Žiadne DAQ na načítanie**— odpojené (`disconnect()`, `chloros-cli daq pool-disconnect`) alebo nie je možné dosiahnuť pripojené DAQ — použije sa posledná hodnota z kamery a priradí sa k**tejto** expozícii: `method`, `retained`, `gap_ms` z tejto expozície, `in_tolerance` sa prepočíta a `retained_from_epoch_ns` označuje expozíciu, s ktorou bol pôvodne spárovaný. Kamera, ktorá nikdy nemala nameranú hodnotu, stále nemá žiadnu odrazivosť a výsledok uvádza dôvod (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, alebo odmietnutie snímky).

Súbory, ktoré snímka uloží do priečinka projektu otvoreného v programe Chloros, sa pridajú do zoznamu súborov daného projektu, rovnako ako snímky v aplikácii (súbory uložené inde zostanú nedotknuté): `CameraSession.capture()`, `ArraySession.capture()` (aj `smart=True`) a `capture_cameras_transient()`. Výsledok to uvádza v súbore `.project_add` (`capture_cameras_transient()`: slovník v súbore `project_add`) -- `held`, `files` a `message` v backende; `held` je `True` počas prebiehajúceho spracovania a súbory sa pripoja po jeho ukončení. `None`, ak sa do projektu nepridal žiadny súbor.

`CameraSession.capture()` a `ArraySession.capture()` ich vracajú ako výsledné súbory `.daq_files` a `.daq_matches`, ako odpovede zachytené protokolom HTTP (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) obsahujú `daq_files` a `daq_matches`, po jednom na každý `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` a `stream_fault` — vlastná chyba prúdu zariadenia DAQ, vyhľadávaná iba v prípade, že nameraná hodnota je mimo tolerancie. Predtým, ako sa spoliehate na odrazivosť zo záznamu, skontrolujte `in_tolerance` v jeho `.match.json` alebo XMP, alebo v [`status()`](#daqsensorsession-methods).

Úrovne `processing`:

| Úroveň | Výstup |
| --- | --- |
| `"raw"` | Jednokanálový Bayer (mono kamery: jedno spektrálne pásmo) priamo zo senzora. |
| `"debayered"` *(predvolené nastavenie SDK)* | 3-kanálová farba: tmavý posun (DSNU) z kalibračného balíka odpočítaný z surových údajov, potom demosaikácia s ohľadom na hrany pomocou OpenCV — bez LUT linearity, vyrovnania poľa ani oddelenia (čiernobiele kamery: 1-kanálová škála sivej). Uložený súbor TIFF je v poradí kanálov **RGB**(fotometrické RGB, takže akýkoľvek prehliadač obrázkov zobrazuje skutočné farby); polia v pamäti, ktoré vracajú funkcie `grab()` / `frame_stream()`, sú v poradí**BGR** podľa OpenCV — pre porovnanie s uloženým súborom obrátenie kanálov (`frame[..., ::-1]`). `chloros-cli process` zapisuje rovnaký výsledok ako `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm prostredníctvom úplného radiometrického reťazca. Iba multispektrálne — kamery RGB sa preskakujú. |
| `"reflectance"` | uint16 0..32768 (pripravené pre Pix4D); vyžaduje spárovanie so živým DAQ pre absolútnu referenciu. Iba multispektrálne. |
| `"display"` | Úplný reťazec zodpovedajúci náhľadu v grafickom rozhraní (CCM + WB + gama podľa profilu kamery). |
| `"all"` | **Jeden súbor na príslušnú úroveň** pre každú kameru (zodpovedá predvolenému nastaveniu v grafickom rozhraní „Capture All“ / CLI). Vrátený súbor `CaptureResult` potom obsahuje jeden slovník snímok na každý súbor `(cam, level)`, pričom úroveň je uvedená v každom slovníku; neplatné úrovne sa objavujú v `.skipped`. DAQ, ktorý čítal použitý záznam, je uložený ako doplnkový súbor `.daq`. |

> **Poznámka — predvolené nastavenie sa líši od CLI.** `ArraySession.capture()` má predvolené nastavenie na `processing="debayered"`; príkaz `chloros-cli lattice array-capture` má predvolené nastavenie na `processing="all"`. Použite príkaz `processing="all"` explicitne z SDK, aby ste napodobnili viacúrovňové ukladanie v CLI a GUI.

##### Názvy súborov zachytenia poľa

Každá úroveň sa ukladá do vlastnej podzložky `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` pre `display`, `index/`, `composite/`). Každý súbor má v názve označenie kamery ako `SN<serial>-<filter>`: sériové číslo, potom časť názvu modelu kamery označujúca filter — `SN214000533-FOCN` pre kameru `M3C-L87-FOCN`, `SN261902142-F450` pre model `M3M-L87-F450` — spojené s `-`, takže názov má rovnaké polia oddelené znakom `_` ako predtým. Kamera, ktorej model sa nedá prečítať, je len `SN<serial>` a zábery z predchádzajúcich verzií nemajú v názve žiadny filter. Názov súboru závisí od toho, o koľko úrovní bol člen požiadaný:

| Požadovaný počet úrovní | Názov súboru | Príklad (`reflectance`, predvolené `format="tiff"`) |
| --- | --- | --- |
| **Jedna úroveň** (`processing="reflectance"` alebo `levels` s jednou položkou) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — bez prípony úrovne, rovnako ako v predchádzajúcich verziách, takže existujúce skripty naďalej fungujú | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Viac úrovní** (`processing="all"` alebo `levels` s viac ako jednou položkou) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

Za názvom produktu nasleduje [maska obmedzenia](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on), `clamp_masks/<product folder>/<product name>_clampmask.tif`: zachytenie odrazivosti na jednejúrovňové zachytenie odrazivosti zapíše `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, zatiaľ čo `processing="all"` zapíše `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. Pravidlo pre príponu sa určuje pre každý prvok na základe požadovaných úrovní, takže vynechaná úroveň (napr. odrazivosť na RGB kamere) ho nemení. Čítajte cesty z vrátených snímok namiesto ich vytvárania. Zachytenie z jednej kamery (`CameraSession.capture()`) vždy pridáva úroveň: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### Formáty PNG a JPEG: spracúvajú sa iba zobrazené a indexované produkty

Formáty PNG alebo JPEG obsahujú 8 bitov na kanál. Žiadosť o formát s úrovňou, ktorú nedokáže obsiahnuť, bude odmietnutá ešte pred zachytením akýchkoľvek údajov, pričom sa zobrazí správa uvádzajúca danú úroveň a formát TIFF, ktorý ju podporuje:

- **`radiance`**(float32) a**`reflectance`**(uint16, `32768` = 1,0) — a teda aj `processing="all"`, ktorý ich zahŕňa. SDK to kontroluje na základe argumentov: `capture()` vyvolá**`ValueError`** bez požiadavky (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` z kamery, ktorá vysiela viac ako 8 bitov** (12-bitový M3C, Mono12 M3M) — backend ich odmietne, keďže pozná bitovú hĺbku každej kamery: HTTP 400 s kódmi `error_code` a `FORMAT_CANNOT_HOLD_LEVEL`, vygenerované ako `ChlorosConnectError` (`CameraSession` / `ArraySession`) alebo `RuntimeError` (`CameraHandle` / `ArrayHandle`). Jednostupňové zachytenie `debayered` z jednej kamery zachytáva 8-bitový snímok pre formáty PNG/JPEG a nie je odmietnuté.

Radiometrické údaje uložte ako `format="tiff"` (`ext=".tiff"`); pre `display` a indexové výstupy použite formáty PNG/JPEG.

#### Ako funguje automatická expozícia

- **Najprv expozícia, až potom zosilnenie.** Ak je zosilnenie nastavené na Auto, zostáva na hodnote 0 dB, kým expozícia nedosiahne najdlhšiu povolenú hodnotu.
- **Pri manuálnej expozícii zostáva zosilnenie v režime Auto**, kým nezmeníte zosilnenie alebo ho nenastavíte na manuálny režim. Po návrate do režimu Auto je limit zosilnenia taký, aký bol predtým.
- **Malé jasné oblasti nezatmavujú scénu.** Odlesky do 1 % záberu sa ignorujú a v paneloch známeho kalibračného terča až do 0,5 % panelu.
- **Horná hranica expozície DAQ platí len vtedy, keď je v zornom poli kalibračný terč.** Zabraňuje to preexponovaniu bieleho panelu; ak nie je viditeľný žiadny panel, kamera exponuje podľa scény.
- **Konštantné osvetlenie zostáva stabilné a zmenené osvetlenie sa ustáli v priebehu niekoľkých sekúnd.** Pri konštantnom osvetlení zostávajú expozícia a jej limit nezmenené a po výraznej zmene osvetlenia sa obraz ustáli v priebehu niekoľkých sekúnd. Preexponovanie už natrvalo neobmedzuje expozíciu a pri zmene oblasti záujmu (ROI) sa expozícia opäť nastaví.
- **Cieľová hodnota jasu zostáva v dosahu.** Nezvyšuje sa, pokiaľ je kamera na limite expozície a jej zosilnenie nemôže stúpať, takže svetlo, ktoré sa vráti, nespôsobí preexponovanie snímky.
- **Kamera, ktorej automatická expozícia prestane reagovať, je udržovaná na úrovni zodpovedajúcej aktuálnemu osvetleniu.** Pri jasnom slnku by kamera mohla klesnúť na najkratšiu expozíciu a zostať tam s tmavým snímkom, pričom všetky jej nastavenia by vykazovali normálne hodnoty. Chloros sleduje snímok, udržiava túto kameru na úrovni, ktorú si zachovala pre dané osvetlenie, a uvoľní ju, keď sa osvetlenie zmení. Tmavý alebo zakrytý svetelný senzor tento proces nenarušuje.

#### Expozícia na členoch poľa

Kamera LATTICE obmedzuje hodnotu `ExposureTime` na jednu periódou svojej snímkovej frekvencie. Počas prenosu dát z poľa spúšťChloros každý člen s nastavenou hodnotou `AcquisitionFrameRateEnable`, pri frekvencii tesne nad spúšťacou frekvenciou, takže horná hranica expozície člena leží tesne pod spúšťacím obdobím, **v akejkoľvek oblasti záujmu (ROI)**:

- Obdobie snímky prvku je obdobie spúšťača znížené o rezervu 15 % tohto obdobia (minimálne 3 ms, maximálne polovica obdobia) a jeho horná hranica expozície je toto obdobie snímky znížené o približne 0,1 ms. Príklad: spúšť 10 fps (100 ms) dáva rýchlosť člena približne 11,8 fps a hornú hranicu expozície približne 85 ms.
- **Ručné nastavenie `ExposureTime`, ktoré je dlhšie ako tento strop, spomalí spúšť, aby sa tomu prispôsobila.** Chloros najprv otvorí strop daného člena na túto hodnotu a nechá pomalšiu frekvenciu bežať niekoľko spúšťacích periód (zápis môže trvať o niekoľko sekúnd dlhšie), potom ju zapíše, a spúšť sa aktivuje najviac tak rýchlo, ako to expozícia dovoľuje. V poli s automatickou frekvenciou sa predchádzajúca frekvencia obnoví hneď, ako sa každý člen, ktorého manuálna expozícia spúšť spomalila, vráti k automatickej expozícii. Hodnota, ktorú samotná kamera nemôže zaznamenať, je naďalej odmietnutá, pričom rozsah, ktorý teraz akceptuje, je uvedený v chybe, a automatická expozícia člena sa obnoví, ak bola vypnutá len na účely tohto zápisu. Pred verziou 1.3.0 malá oblasť záujmu (ROI) obmedzovala každého člena na vlastnú voľnú periódu snímača, bez ohľadu na frekvenciu spúšťania: 4,9 ms pri rozlíšení 768 × 304, takže žiadny člen nemohol byť zosvetlený.
- Chloros nikdy nezkracuje manuálnu expozíciu, aby sa prispôsobila rýchlejšiemu spúšťaniu. Rýchlosť automatického režimu zostáva v rámci toho, čo umožňujú manuálne expozície; ak je nastavená rýchlosť rýchlejšia, než umožňuje expozícia, ponechá sa pre daný člen strop dostatočne dlhý na túto rýchlosť a spúšťa sa len s frekvenciou, ktorú umožňuje jeho expozícia. Znížte expozíciu alebo nastavenú rýchlosť, ak musí každý člen stihnúť každé spustenie.
- **Spätné čítanie:** pri čítaní uzlov prvku sa zobrazí `AcquisitionFrameRateEnable` True a `AcquisitionFrameRate` pri frekvencii prvku; povolenie sa opäť vypne, keď sa pole zastaví.
- **Tieto dva uzly vlastní pole. Nezapisujte hodnoty `AcquisitionFrameRateEnable` ani `AcquisitionFrameRate` na člen poľa** — namiesto toho nastavte rýchlosť poľarýchlosť poľa pomocou `target_fps` / `set_target_fps()`.

Expozíciu člena zapíšte pomocou `arr.set_member_settings(serial, exposure_time=20000)` (kľúčové slová `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` funguje rovnako). Člen zostane v poli. Neotvárajte člena poľa pomocou `connect_camera()`: príkaz connect odmietne kameru, ktorá je už v poli, a príkaz `CameraSession`, ktorý by vrátil, odstráni kameru z jej poľa pri odpojení. Príkaz `ArraySession.status()["member_ae"]` nahlási expozíciu každého členaexpozíciu a zosilnenie z jej najnovšieho snímku.

### Jeden záznam projektu, každý režim

Zložka projektu obsahuje nastavenia každého zariadenia: `cameras.json` — pre každú kameru (`cameras[].settings`, pre jednu kameru a pre každého člena poľa) a pre celé pole (`arrays[].capture`: formát, ROI, binning, frekvencia spúšťania, rozpočet na prenos) — a `sensors.json`, pre každý svetelný senzor (integračný čas, priemer snímky, limit). Aplikácia, CLI, SDK a server MCP všetky čítajú a zapisujú do rovnakých súborov rovnakým spôsobom:

- **Zariadenie pripojené v rámci projektu sa do neho uloží a zaznamenáva do neho každú zmenu.** „V rámci projektu“ znamená s otvoreným projektom v backende (aplikácie) alebo — prostredníctvom súboru `ChlorosProject` (`open_project()`), ktorý v backende určuje vlastnú zložku (`project_path`). Kamera, pole alebo svetelný senzor, ktoré projekt ešte neobsahuje, sa pri pripojení pridajú do jeho súborov (`connect_camera()`, `connect_array()`, pripojenie svetelného senzora, CLI, aplikácia, `chloros-mcp`), spolu s jej nastaveniami. Zmena z akéhokoľvek režimu (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, nastavenia poľa, `set_cap()`, CLI, ovládacích prvkov v aplikácii) sa o chvíľu uloží do súborov daného projektu. Nastavenie expozície, zosilnenia alebo cieľovej hodnoty AE na kamere sa uloží ako zachované, takže pri opätovnom otvorení projektu sa tieto nastavenia znovu použijú.
- **Otvorenie projektu nastaví jeho zariadenia podľa neho**, a to v každom režime: fotoaparát, pole alebo svetelný senzor pripojený bez vlastných nastavení prevezmeuložené nastavenia (`lattice cam-connect` / `array-connect` pri otvorenom projekte, `chloros-cli project connect`, `open_project().connect_all()`, opätovné otvorenie aplikácie). Pole pripojené pod novým ID aktualizuje pole, ktoré projekt uložil s rovnakými kamerami; pole s inými kamerami sa pridá vedľa neho.
- **Vlastné uloženie aplikácie nikdy neprepíše aktuálne nastavenia zariadenia** staršou kópiou týchto nastavení a zachová pre každý prvok prekrytia uložené volajúcim z CLI alebo SDK.
- **Formát pixelov, ktorý nastavíte, je ten, ktorý si projekt uchováva.** Formát nastavený v nastaveniach poľa, pomocou `--pixel-format` alebo `connect_array(pixel_format=)`, vrátane 8-bitového, sa uloží a pri opätovnom pripojení sa presne zachová. Ak nie je nastavený žiadny formát, každá kamera sa pripojí vo svojom predvolenom 12-bitovom nastavení (`BayerRG12`, `Mono12` na monochromatickom senzore). Názov formátu označuje jeho bitovú hĺbku; rodina sa vzťahuje na snímač, takže farebná kamera sa nikdy nenastaví na formát Mono a monochromatická kamera na formát Bayer, a riadok `[PixelFormat <serial>] ... written` to uvádza, keď bol formát priradený. Výstupy 8-bitového záznamu sú 8-bitové, vrátane odrazivosti: preškálované a označené kódom `PixelScale`.
- Zariadenie pripojené bez projektu nezaznamenáva nič.
- **Záznam sa priradí k projektu vytvorenému po ňom; otvorenie ho ponechá v pôvodnom projekte.**Keď sa vytvorí projekt — `create_project()`, `chloros-mcp` alebo pomocou funkcie**Nový projekt**v aplikácii — alebo duplikovaný (funkcia aplikácie**Duplikovať**, ktorá otvorí kópiu), zatiaľ čo záznam stále zapisuje do projektu, ktorý bol otvorený predtým, záznam pokračuje a to, čo zapisuje, sa uloží do nového projektu, na rovnaké miesto v ňom (nižšie). Otvorenie existujúceho projektu — `ChlorosProject.process()` (čo otvorí jej projekt na pozadí), `chloros-cli process`, funkcia aplikácie**Otvoriť projekt** — ponechá nahrávanie stále prebiehať v projekte, do ktorého sa nahráva: tam sa dokončí a jej súbory sa pridajú do zoznamu súborov daného projektu (aplikácia zastaví každé nahrávanie pred otvorením projektu).
  - `ArraySession.record()` a `ArraySession.burst()`: video alebo zložka sériových snímok so všetkými snímkami sa tam presunie ako celok po zastavení nahrávania. Výsledok zastavenia uvádza novú cestu (`out_dir`, `video_path`) a z nej sa vytvorí zostavenie série snímok. Ak disk odmietne presun, záznam zostane tam, kde bol zapísaný, a výsledok to uvádza (`relocation_error`). Surové snímky zastavenej série a súbor `.daq` sa pridajú do zoznamu súborov projektu, do ktorého sa dostanú; video sa do zoznam súborov projektu.
  - `ArraySession.continuous()`: každý záznam po zmene sa uloží do nového projektu; tie, ktoré boli uložené predtým, zostanú tam, kde sú.
  - `DAQSensorSession.record_start()` (a vlastný záznam aplikácie zo svetelného senzora): súbor `.daq` sa po ukončení nahrávania presunie do zložky `light_sensor` nového projektu; `record_stop()` vráti novú cestu a súbor sa pridá do zoznamu súborov daného projektu.

  Záznam do zložky mimo otvoreného projektu zostane tam, kde je, a úlohy typu `start_camera_continuous()` / multi-stream si zachovajú zložku, ktorá im bola pridelená.
- **V aplikácii sa záznamy po vytvorení nového projektu presúvajú do neho.** Vytvorenie alebo duplikovanie projektu zachová všetky prebiehajúce záznamy a nahrávanie, a každý záznam zhotovený po ňom (Záznam všetkého, Spustiť záznam) sa uloží do nového projektu a pridá sa do jeho zoznamu súborov. Otvorenie existujúceho projektu v aplikácii najprv zastaví všetky spustené úlohy.
- **Projekt sa počas nahrávania nepresúva.**Presunutie otvoreného projektu do iného pracovného adresára (v aplikácii v časti**Nastavenia → Pracovný adresár**; žiadne volanie SDK nepresúva projekt, HTTP klient odosielajúci na `/api/move-project` áno) je odmietnuté s chybou HTTP 409 `PROJECT_RECORDING`, pokiaľ do neho zapisuje záznam z matice alebo sériové snímanie, nepretržité snímanie z matice, úloha nepretržitého snímania / viacerých prúdov z kamery alebo záznam zo svetelného senzora: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Toto je odlišné od toho, v akom stave zostanú kamery po odpojení (pozri [Pamäť kamery](#camera-memory--keep_settings--clear_settings)): projekt si pri otvorení sám nastaví svoje kamery.

### Pamäť kamery — `keep_settings` / `clear_settings`

Keď je pole pripojené, jeho kamery preberajú nastavenia poľa a vy ich potom môžete zmeniť. Keď program Chloros uvoľní kamery — `ArraySession.disconnect()` / `force_disconnect()`, na konci prechodnej funkcie, zastavenie `start_camera_continuous()` — vráti každú z nich do stavu, v akom bola pred pripojením: jej spúšť a zapojenie synchronizačnej linky, nastavenia PTP a kamery — `ExposureAuto` / `GainAuto` (a manuálne nastavenia `ExposureTime` / `Gain`), `TargetBrightness` a limity AE a tlmenie, vyváženie bielej, úroveň čiernej, `PixelFormat`, zlučovanie a oblasť záujmu (ROI), farebný reťazec (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, PTP: `PtpSlaveOnly` / `PtpDomainNumber`, veľkosť paketu (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), frekvencia (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), údaje v blokoch (`ChunkModeActive`, každý `ChunkEnable` pod svojím `ChunkSelector`), odstránenie šumu zo spúšťacieho vstupu a pole poskytuje synchronizačnú linku podriadeného zariadenia (`LineFilterSelector` / `LineFilterWidth`) a frekvenciu snímania (`AcquisitionFrameRate` iba v prípade, že bolo jeho povolenie zapnuté: ak je vypnuté, frekvencia číta maximálnu hodnotu, ktorú umožňuje taktovanie, nie nastavenie). Zapisuje sa iba to, čo sa líši: kedy sa kamera zastavila ako prvá, taktovanie a frekvencia snímok pred expozíciou, ktorá vyžaduje ich snímkovú periódou.

Pri používaní kamier mimo projektu dva argumenty zapisujú do **vlastnej pamäte** kamery — to, s čím sa kamera spúšťa, v akomkoľvek softvéri (na `disconnect()` / `force_disconnect()`, vo všetkých prechodných funkciách, `start_camera_continuous()` a `ChlorosProject.disconnect_all()`; MCP: rovnaké argumenty ako v `disconnect_device` a prechodných nástrojoch; CLI: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** uloží nastavenia každej kamery tak, ako ich relácia zanechala v jej pamäti (UserSet2, vytvorená ako spúšťacia sada). Kamery zostávajú naďalej v režime voľného chodu (`TriggerMode` `Off`), zapojenie synchronizačnej linky zostáva tak, ako bolo zistené. Kamera bez UserSet2 ukladá do UserSet1; tento počítač si to pamätá, takže samonapravovanie poľa Chloros — ktoré resetuje UserSet1, ktorý sa odchýlil od základnej hodnoty Chloros — ju ponechá bez zmeny.
- **`clear_settings=True`** obnoví pamäť každej kamery na továrenské nastavenia: referenčnú hodnotu systému Chloros, ktorú jeho funkcia automatického opravovania poľa zakotvuje — továrenské predvolby s `TriggerMode`, `Off`, `TriggerSource`, `Line2`, automatická expozícia, kalibrovaná vyváženosť bielej RGB kamery zachovaná — v UserSet1, vytvorená ako spúšťacia sada.
- Nie obidve (`ValueError`; backend odpovedá 400). Odpoveď obsahuje `camera_memory`, pre každú kameru `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` alebo `{"cleared": bool, "errors": [...]}` — vráti ho `force_disconnect()`; pre úlohu prichádza s `stop()`.

Aplikácia tieto možnosti neponúka.

**Každý spôsob ukončenia vráti kamery späť.**Vrátenie do pôvodného stavu prebieha pri každej ceste ukončenia: `CameraSession.disconnect()` (kamera**X**aplikácie),**X** v poli alebo akýkoľvek prvok poľa (celé pole sa uvoľní), zatvorenie aplikácie (backend má až 20 s na vrátenie každej kamery, poľa a bežiaceho nepretržitého nahrávania; prechodná funkcia alebo jednorazové volanie, ktoré stále beží, sa zastaví na najbližšom bezpečnom bode, najprv vráti svoje kamery, a to až 8 s, a odpovie HTTP 503 `SHUTTING_DOWN`; stále bežiaca funkcia `connect_array()` sa zastaví pred svojou ďalšou kamerou a vráti kamery, ktoré otvorila, a to až 10 s, a vyvolá `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); a ten, ktorý dorazí počas vypínania, nič neotvorí: HTTP 503 `SHUTTING_DOWN`), a príkaz `chloros-cli`, ktorý je ukončený alebo ktorého terminál sa zatvorí (jeho úloha sa zastaví a kamery sa najskôr vrátia späť). Backend, ktorý sa zrúti alebo je násilne ukončený, nemôže nič vrátiť späť; reštart vráti každú kameru do jej pamäte.**Projekt si nastavuje kamery sám.** Keď sa pole pripojí prostredníctvom projektu — otvoreného projektu aplikácie alebo `ChlorosProject`, ktorý sa identifikuje voči backendu (`project_path`) — každý člen získa nastavenia, ktoré pre neho projekt uložil: expozíciu, zosilnenie alebo cieľovú hodnotu automatického nastavenia expozície (uložené ako vaše, takže automatická expozícia ich nemení) a režim vyváženia bielej (korekcia Chloros; vlastné vyváženie bielej fotoaparátu zostáva vypnuté), nech už pamäť fotoaparátu obsahuje čokoľvek.

### Režimy snímania a záznamové zariadenia

Povrch poľa odzrkadľuje panel snímania v grafickom rozhraní: režimy Jednorazový / Sériový / Intervalový / Najrýchlejší uzávierky, plus dve záznamové zariadenia (kompozitné video v reálnom čase a séria snímok v surovom formáte → offline spracovanie).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**je slučka nepretržitého/intervalového snímania v SDK. Keďže neexistuje `Ctrl+C`, ktorý by ju zo skriptu prerušil,**musíte** odovzdať `count` a/alebo `duration_s` (zastaví sa, keď sa dosiahne ktorýkoľvek z nich). `interval_s` sa meria od začiatku každého prechodu (zodpovedá to grafickému rozhraniu). Nepretržité prechody (`interval_s=0`) reťazia hodnoty `group_seq` → `after_seq`, takže ide o po sebe idúce synchronizované skupiny v rýchlosti poľa. Zostávajúce kwargs prechádzajú priamo do `capture()`.
- **`continuous`**je vlastné nepretržité zachytávanie backendu — režim Continuous v grafickom rozhraní.**Použite ho namiesto `capture_repeated()`, ak chcete, aby sa každý spúšťací signál ukladal na disk s vysokou snímkovou frekvenciou:** `capture_repeated()` odosiela zo skriptu jednu požiadavku HTTP na skupinu, zatiaľ čo `continuous()` udržiava dve zachytenia v behu vnútri backendu bez obojsmernej komunikácie medzi skupinami. Merané pri ROI 512 na sústave s 9 kamerami: 73–77 záznamov/s, oproti sústave, ktorá publikuje približne 78. `capture_repeated()` si ponechajte pre intervalové zaznamenávanie (`interval_s`) a pre prelety, ktoré kontrolujete jeden po druhom. V systéme Windows vylúčte backend z programu Microsoft Defender, aby ste dosiahli vysoké rýchlosti (pozri [Krok 1](#step-1--install-the-chloros-platform-package)).
- **`record`** je určený na *monitorovanie*: zaznamenáva vykreslený živý kompozitný kombinovaný index. Ak je otvorený kombinovaný pohľad v grafickom rozhraní, zaznamenáva tento pohľad tak, ako sa zobrazuje. Ak nie je otvorený žiadny, backend vykreslí pohľad sám: `expr`, ak ho zadáte, inak NDVI poľa (červené a NIR pásma najbližšie k 660 nm a 850 nm, tak, ako ich zachytáva živý pohľad), hneď ako je pole zarovnané. Pásma v `expr` pomenujte tak, ako ich pomenuje pole: `<band>_<filter>` pre kameru M3C (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` pre M3M (`Mono_850`); krátky názov farby (`NIR`, `Red`) funguje len vtedy, ak ho má aspoň jeden pás poľa. Pohľad na grafické používateľské rozhranie otvorený počas záznamu sa zaznamenáva, pokiaľ zostáva otvorený. Záznam, ktorý nezapíše žiadny snímok, zlyhá s chybami `error`, `"no-frames: …"` (pole sa môže stále zosúlaďovať alebo stabilizovať expozíciu), a vyvolá sa výnimka `stop()` (pozri [`RecorderHandle`](#recorderhandle)). Jeden kompozitný záznamník na jedno pole (vyvolá výnimku, ak už jeden beží).
- **`burst` → `build_video`** je *analytická úroveň*: `burst` zapisuje surové snímky + manifest pre každý snímok + jeden `.daq` na každé odlišné čítanie DLS pod `<output>/bursts/<base>/` pri plnej rýchlosti snímacej slučky (bez reťazca, bez exiftool, bez živého náhľadu). `build_video` časovo zosúladí každý snímok s najbližším `.daq` a opätovne spustí reťazec importu pre odrazivosť/reflexie/indexu. `products` je zoznam `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (predvolené: kombinovaný index). `burst().stop()` tiež automaticky spustí vytvorenie kombinovanéhoindex, ktorý sa v konečnom výsledku vráti ako `build_job`.

##### Výsledok `build_video`

| Kľúč | Význam |
| --- | --- |
| `success` | `True` iba vtedy, ak bol každý požadovaný produkt vytvorený podľa požiadaviek: aspoň jeden výstup, nič v `skipped`, a nie v `degraded`. |
| `outputs` | Cesty k uloženým videám (a GIF súborom). |
| `errors` | Problémy, ako text. |
| `skipped` | Jeden záznam `{"serial", "level", "reason"}` na každý požadovaný produkt, ktorý **nevyprodukoval** žiadne video — buď kvôli (chýba kalibračný balík pre danú kameru, chýba DLS odčítanie odrazivosti, zlyhanie spracovateľského reťazca) alebo kombinovaného indexu, ktorý obsahuje `"kind": "combined"` a `serial` `None` (žiadna dvojica pásiem červeného/NIR medzi kamerami, žiadna skupina so žiarivosťou pre obe pásma). Akýkoľvek záznam spôsobuje `success` `False`; ostatné videá sa naďalej zapisujú a uvádzajú v zozname. |
| `substitutions` | Kamery boli spracované na inej úrovni, ako bolo požadované, a dôvod — RGB-kamera, pre ktorú bola požadovaná žiarivosť/odrazivosť, dostane svoje zobrazené video s vyvážením bielej, ak je k dispozícii kalibračný balík kamery. Nie je to chyba. |
| `notes` | Informačný text, nie chyby — napr. kamera bez indexového výrazu pre danú kameru nemá indexové video, alebo zobrazované video RGB-kamery nemá vyváženú bielu (chýba kalibračný balík alebo je bez továrenského vyváženia bielej), s uvedením názvu kamery. Nie je to chyba. |
| `degraded` | `True`, keď bol produkt zapísaný, ale nie je to ten požadovaný: offline kombinovaný index je vstavaný normalizovaný rozdiel červenej a NIR, preto sa nakonfigurovaný indexový výraz nevyhodnocuje. Vytvára `success` `False`. |
| `combined` | Súhrn kombinovaného indexu: použité pásma, rozsah LUT, vytvorené snímky. |
| `clamp_masks` | Cesty [masiek obmedzenia oddeľovania](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) presnímky kalibrovaných TIFF súborov (`save_tiffs=True`), v súbore `clamp_masks/<level>/` vedľa zložky `<level>/` týchto TIFF súborov, po jednej pre každý TIFF, v ktorom bol pixel obmedzený. Video žiadnu neobsahuje. |
| `clamp_fractions` | Čo uvádza video namiesto masky: jeden súbor `{"kind", "serial", "level", "frames"}` na každý videozáznam (`kind` `"combined"` pre kamery, ktoré sú členmi kombinovaného indexu), `frames` zoznam `[frame, R, G, B]` — podiel pixelov v danom snímku, ktoré boli pri oddeľovaní obmedzené v pásme, ktoré obsadzuje jeho kanál R / G / B (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Vrátené funkciami `ArraySession.record()` a `ArraySession.burst()`. Použite ho ako správcu kontextu na automatické zastavenie pri ukončení rozsahu alebo ho ovládajte manuálne.

| Člen | Popis |
| --- | --- |
| `job_id` | ID úlohy na pozadí (reťazec). |
| `kind` | `"composite"` (z `record`) alebo `"raw"` (z `burst`). |
| `start_stats` | Slovník vrátený volaním `start`. |
| `result` | `None` počas behu; konečný slovník výsledkov po zastavení. |
| `stats(timeout=10.0)` | Štatistiky úlohy v reálnom čase (počet zapísaných snímok, dosiahnutá snímková frekvencia, uplynulý). |
| `stop(timeout=60.0)` | Zastavenie záznamníka; vráti a uloží do vyrovnávacej pamäte konečný výsledok (`.result` zostane `None`, ak samotná požiadavka na zastavenie zlyhala). V prípade zlyhania záznamu `record()` — t. j. záznamu, pri ktorom nebol zapísaný žiadny snímok (`error` `"no-frames: …"`) — uloží výsledok do vyrovnávacej pamäte na `.result` a potom vyvolá výnimku `ChlorosConnectError`. Idempotentné (druhé volanie vráti výsledok z vyrovnávacej pamäte bez vyvolania výnimky). Opustenie bloku `with` zastaví záznamník bez vyvolania chyby, preto skontrolujte `rec.result.get("error")` po ňom. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Vrátené funkciou `ArraySession.continuous()`. Použite ho ako správcu kontextu — opustenie bloku `with` zastaví zachytávanie, ak ešte beží — alebo ho ovládajte pomocou `status()` / `wait()` / `stop()`. V danom okamihu prebieha len jeden nepretržitý záznam na jedno pole.

| Člen | Popis |
| --- | --- |
| `start_status` | Slovník stavu vrátený volaním start. |
| `result` | `None` počas behu; konečný slovník stavu po ukončení zachytávania (nastavený funkciou `stop()`, alebo funkciou `status()` / `wait()`, ktorá zaznamenala jeho ukončenie). |
| `status(timeout=10.0)` | Slovník aktuálneho stavu (polia nižšie). |
| `wait(timeout=None, poll_s=0.5)` | Blokovať, kým sa zachytávanie samo neukončí — dosiahnutie hodnoty `max_captures` alebo `duration_s`, vyprázdnenie poľa, zaplnený disk, chyba — a vrátiť konečný stav. S `timeout` vráti najnovší stav po uplynutí toľkých sekúnd, aj keď proces stále beží: skontrolujte `running`. |
| `stop(timeout=30.0)` | Zastaví zachytávanie a vráti jeho konečný stav. Idempotentné (druhé volanie vráti výsledok z vyrovnávacej pamäte). |

Polia stavu:

| Pole | Význam |
| --- | --- |
| `running` | True, pokiaľ prebieha zachytávanie. |
| `captures` | Uložené synchronizované skupiny. |
| `files` | Súbory zapísané. |
| `failed_files` | Súbory, ktoré sa nepodarilo zapísať (ako v [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)). |
| `failed_captures` | Pokusy o zachytenie, ktoré zlyhali ako celok, vrátane tých, pri ktorých sa zachytenie opakovalo (HTTP 503, keď pole nemalo včas novú skupinu). |
| `skipped_groups` | Skupiny, ktoré pole zverejnilo, ale neboli uložené (zachytávanie zaostalo). |
| `captures_per_s` | Priemerná rýchlosť ukladania od začiatku. |
| `elapsed_s` / `started_at` | Doba behu v sekundách; čas začiatku (sekundy od Unixovej epochy). |
| `last_group_seq` | Najnovšia synchronizovaná skupina, ktorá bola uložená. |
| `last_error` | Posledná chyba alebo `None`. |
| `stop_reason` | Po ukončení: `count` (dosiahnuté `max_captures`), `duration` (dosiahnuté: `duration_s`), `stopped` (vyvolané: `stop()`), `array_gone` (pole bolo odpojené), `error` (20 zlyhaní za sebou alebo požiadavka, ktorú backend odmietol), `disk_full` (súbor, na ktorý na disku nebolo miesto). `None` počas behu. |
| `max_captures` / `duration_s` | Počiatočné hranice (`0` = žiadne). |
| `defender_cores` | Jadrá procesora, ktoré Microsoft Defender použil za posledných 10 sekúnd; `None`, kým sa nezmerá aspoň jedno, a vždy `None` na hostiteľskom počítači bez programu Defender. |
| `defender_warned` | Má hodnotu True, keď zachytenie zaznamená, že Defender ho spomaľuje (na 1 jadre alebo viac). Vylúčte backend zo skenovania programom Defender — pozrite si [Krok 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Živé snímky vo vašom vlastnom potrubí

Ak chcete synchronizované snímky posielať do vlastného kódu — strojové učenie, vlastný prehliadač, vlastný záznamník — použite `ArraySession.frame_stream()`. Každý tik generuje `GrabResult`: `{serial: numpy array}` pre každú kameru, **všetko z jedného spúšťača**, pričom ID tohto spúšťača je uvedené v `.group_id`. Ide o rovnakú implementáciu ako v prípade `ArrayHandle.frame_stream()` v projektovom API, takže všetko v častiach [Jeden spúšťač na jedno čítanie](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#aligning-the-streams--align) a [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) platí pre obidva.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Každá vygenerovaná množina predstavuje nový spúšťač.** `frames.group_id` sa v rámci toku nikdy neopakuje; ak požiadate o viac `fps`, než je rýchlosť, s akou beží pole, tok čaká na ďalší spúšťač, takže beží pri nižšej z týchto dvoch rýchlostí.
- **Chýbajúce kamery sú označené, nie sú dopĺňané.** Kamera, ktorá nemohla byť obslúžená pri danom spúšťači, chýba v slovníku a je uvedená v `frames.errors`.
- **Končí to náhle.** V prípade odmietnutia sa `FrameFetchError` zopakuje pri nasledujúcom tiknutí, po 60 sekundách bez snímky alebo po 60 sekundách bez nového spúšťača.**Čo to prináša.**`frame_stream()` prenáša celé snímky numpy cez lokálne HTTP pripojenie a dosahuje maximálnu rýchlosť takmer**170 MB/s**. To stačí na spracovanie surových snímok v reálnom čase, alebo na spracovanie zoskupených snímok či poľa ROI, ale nie na spracovanie každého spúšťača v plnom rozlíšení — ak chcete zachovať**každý** spúšťač, uložte na disk pomocou [`continuous()`](#continuoushandle). Merané na sústave štyroch kamier (FOCN, FRGN, FRGB, F450) s komprimovaným prenosom; každý tik predstavoval kompletnú sadu jedného spúšťača:

| | Plný senzor (pole 14,2–14,4 fps) | Zlučovanie 2 × 2 (pole ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 sád/s | 22,2 sád/s |
| `frame_stream(processing="debayered")` | 2,3 sád/s | 9,3 sád/s |
| `frame_stream(processing="display")` | 0,8 sád/s | 3,0 sád/s |
| MJPEG na kameru, 4 prúdy pri rozlíšení 720p | 13,5 fps každý; každý snímok zodpovedá spúšťaču | ~25 fps každý; ~45 % spúšťačov prítomných vo všetkých 4 |

Použite `processing="raw"` pre priepustnosť a demosaic na vašej strane; úroveň `debayered` tiež odstraňuje tmavý posun kalibračného balíka, čo demosaik na úrovni `raw` nerobí.

#### MJPEG prúdy pre každú kameru (bez RTSP)

**RTSP nie je podporované.** Rozhranie pre streamovanie je MJPEG cez HTTP (`multipart/x-mixed-replace`, jeden súbor JPEG na časť, každá časť s `Content-Length`), jedna URL adresa na kameru:

| URL | Čo poskytuje |
| --- | --- |
| `GET /api/camera/stream/<serial>` | 8-bitový náhľad, až ~30 fps. Pre člen poľa určujú veľkosť a kvalitu hodnoty `max_h` (predvolené 720, 64–2160) a `jpeg_q` (predvolené 80) nastavujú veľkosť a kvalitu. |
| `GET /api/camera/<serial>/stream-annotated` | To isté náhľadové zobrazenie s nakreslenými grafickými prvkami rozhrania (zebra, mriežka, nitkový kríž, histogram, peaking, bodový expozimeter). `ChlorosLocal.camera_stream()` to pre vás prečíta. |
| `GET /api/camera/<serial>/stream-reflectance` | 8-bitová odrazivosť na pásmo — pozri [Živý prúd odrazivosti](#live-reflectance-stream-mjpeg). |

- **Spúšťacie hlavičky.** V prípade prvku poľa obsahuje každá časť všetkých troch prúdov `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` a `X-Chloros-Timestamp-Ns`. Časti z rôznych kamier s rovnakým `X-Chloros-Group-Id` tvoria rovnaký spúšťací signál: zhromaždite časti a zoskupte ich podľa tejto hlavičky, aby ste znovu zostavili sadu. Samostatná kamera nemá žiadnu skupinu: jej časti náhľadu neobsahujú žiadnu z týchto hlavičiek (jej súčasti odrazivosti majú prázdnu hlavičku `X-Chloros-Group-Id`).
- **Prúd náhľadu pri zaťažení vynecháva snímky.** Poskytuje najnovšiu snímku a preskakuje všetky, ktoré nestihol odoslať včas, takže pri vysokých rýchlostiach poľa sa nie každý spúšťač dostane do prúdu každej kamery (pozri tabuľku vyššie). Pre každý spúšťací signál použite `frame_stream()` alebo `continuous()`.
- **`/api/camera/stream/<serial>` sa pozastaví, kým je otvorený kombinovaný živý stream poľa** (`/api/camera/array/stream/<array_id>` — otvorí ho zobrazenie poľa v grafickom rozhraní). Stream každého člena potom odošle jeden rámec s náhradným textom „Pozastavené (aktívny stream poľa)“ bez hlavičiek `X-Chloros-*` a nič viac, kým sa kombinovaný prúd nezatvorí; potom sa sám obnoví. `stream-annotated` a `stream-reflectance` sa nezastavia. Zatvorte zobrazenie poľa v grafickom rozhraní, keď skript číta náhľadové prúdy jednotlivých kamier.
- **Pre čísla, nie obrázky, použite `frame_stream()`.**Formát JPEG je stratový a 8-bitový; `frame_stream()` vráti rovnaké výstupy bez kodeku v ceste.

### Intervalové snímanie pre prieskumy pomocou dronov

Prieskumný let vyžaduje jednu synchronizovanú skupinu snímok každých niekoľko metrov letu, nie každý snímok, ktorý pole dokáže vygenerovať. Toto riadia dve nastavenia:**`target_fps`**určuje, ako často kamery snímkujú, a**`capture_repeated(interval_s=…)`** určuje, ako často sa snímka ukladá.**Kamery pokračujú v snímaní aj medzi ukladaniami.** Pri každom intervale sa uloží najnovšia synchronizovaná skupina; snímky medzi nimi sa nezaznamenávajú. Automatická expozícia sa prispôsobuje pri každom snímku, takže v čase uloženia je expozícia už ustálená. Neexistuje režim, v ktorom by sa kamery medzi jednotlivými zábermi zastavili a pre každý záber sa znovu aktivovali: každý záber by v takom prípade musel čakať na spustenie prenosu a ustálenie expozície.**Nastavte snímkovú frekvenciu podľa potrieb intervalu.**Ak ponecháte nastavenie na automatiku, pole beží pri najvyššej frekvencii, ktorú zvládne hostiteľ a sieť. Znížením frekvencie znížite zahrievanie kamery, zaťaženie siete a procesora hostiteľa bez zmeny obsahu záznamu. Zvoľte frekvenciu, ktorá poskytuje**2–4 snímky na interval**:

- Každé uloženie čaká na ďalší snímok po jeho vyžiadaní, takže uložená skupina sa dostane až o jednu snímkovú periódu neskôr, ako bolo naplánované. Vyššia frekvencia udržuje tento časový odstup malý.
- Automatická expozícia sa upravuje raz za snímok, takže veľmi nízka frekvencia spomaľuje rýchlosť, akou expozícia sleduje prechádzajúci mrak.

**Interval vypočítajte na základe letového plánu:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Napríklad stopa 60 m pri 80 % prekrývaní dopredu znamená rozstup 12 m; pri rýchlosti 10 m/s to je `interval_s=1.2`, a fixná hodnota `target_fps=3` zodpovedá 3 až 4 snímkam na interval. Zaznamenávanie je časovo riadené: SDK nemá režim spúšťaný na základe vzdialenosti ani GPS, preto na meracích trasách udržiavajte konštantnú rýchlosť voči zemi.

**Uložte surové dáta priamo na lietadle a kalibrujte neskôr.** Hodnota `interval_s` sa meria od začiatku každého ukladania, takže ak ukladanie trvá dlhšie ako interval, nasledujúce ukladanie sa spustí neskôr a rozstup sa posunie. Ukladanie surových dát spolu s hodnotou svetelného senzora (`processing="raw"`; táto hodnota sa ukladá štandardne) zabezpečuje, že každé ukladanie je krátke a každý snímok je možné neskôr previesť na odrazivosť.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**nastaví rýchlosť od začiatku;**`arr.set_target_fps(fps)`**ju mení počas letu bez opätovného pripojenia a**`arr.set_target_fps(None)`** ju vráti do automatického režimu. Rýchlosť presahujúca kapacitu hostiteľa a siete sa zníži na tento strop — skontrolujte `clamped` vo výsledku — a najnižšia rýchlosť je 0,1 fps.
- **Nepoužívajte `burst()` na redukciu snímok.** Zapisuje každý snímok, ktorý pole vygeneruje; nasleduje po fixnej hodnote `target_fps`, takže je to správny nástroj len vtedy, ak chcete zachovať každý snímok.
- **`target_fps` neopravuje poškodené snímky.** Znížením tejto hodnoty sa zníži počet generovaných snímok, nie rýchlosť odosielania jednotlivých snímok; v prípade snímok poškodených v GVSP namiesto toho znížte hodnotu `wire_ceiling_mbps` (pozri [Stav poľa](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Pripojenie k už pripojenému poľu — `attach_array`

Ak je pole už spustené (otvorilo ho grafické rozhranie alebo predchádzajúca relácia SDK volala `connect_array`), použite namiesto opätovného pripojenia funkciu `attach_array` na získanie referencie naň. V takejto situácii sa vždy zobrazí chyba „Kamera  <sn>je už v poli<id>“, pretože odoslanie požiadavky POST s kódom `/array/connect` pre člena v skupine nie je idempotentné; `attach_array` číta `/api/camera/array/list` a porovnáva buď podľa array_id, alebo podľa sériových čísel.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Vzor: Skripty SDK, ktoré sú súčasťou grafického používateľského rozhrania na pracovnej ploche, by mali najprv vyskúšať `attach_array` a ak v skupine ešte nie je žiadne pole, prejsť na `connect_array`.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Pripojené pole je vypožičané, nie je vo vlastníctve.** Rukoväť z `attach_array` nikdy pole nezruší: ani ukončenie bloku `with`, ani explicitné `disconnect()` ho neuvoľní (explicitné volanie vypíše jeden riadok do stderr s informáciou, že polia zostalo spustené). Platí rovnaké pravidlo ako pre `CameraSession` / `DAQSensorSession`. Spoločné používanie s grafickým rozhraním je teda bezpečné s aj bez `with`:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Ak ho skutočne chcete ukončiť – napríklad aby ste sa znovu pripojili s iným `wire_ceiling_mbps` – zavolajte `arr.force_disconnect()`. Opätovné pripojenie po bežnom `disconnect()` na pripojenom handle zlyhá s chybou „Kamera je<sn> už v poli<id>“, pretože nebolo uvoľnené nič.

### Pomocník pre analýzu siete

Užitočné pred otvorením poľa – odhaduje, či vaše navrhované nastavenia budú vyhovovať:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Nikdy nespustí backend: ak backend neodpovedá, vyvolá sa chyba `ChlorosConnectError`, zatiaľ čo odmietnutá požiadavka sa vráti ako výsledok. Chybný `packed` vyvolá chybu `ValueError`.

`status` je jednou z chýb `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (inak `error` — vrátane zamietnutej požiadavky: nie ste prihlásený, `error_code: AUTH_REQUIRED`, alebo plán pod minimálnou hranicou SDK, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` znamená, že požadované rozlíšenie vyhovuje prstencu RX iba pri obmedzenej spúšťacej frekvencii — zachovajte rozlíšenie a prejdite z `target_fps=result["recommended"]["recommended_target_fps"]` na `connect_array` (pozri [Príklad 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` sa objaví len vtedy, keď požadovaný snímok nemožno doručiť pri žiadnej snímkovej frekvencii; connect nikdy sám nezmenšuje snímky, preto `recommended` použite sami.

**Ako čítať projekciu** (rovnaký model ako v paneli Nastavenia poľa v grafickom rozhraní):

- **Burst (`frame_bytes_total`) sa sčítava pre každú kameru v reálnom formáte pixelov danej kamery.**Mono**M3M**kamery vysielajú Mono12 (2 B/px) alebo s nastavením `pixel_format` formát Mono svojej hĺbky (`"BayerRG8"` je `Mono8`, 1 B/px), takže bez neho má snímka v plnom rozlíšení zo 4 kamier veľkosť**~25 MB** pri troch mono kamerách, nie ~12,6 MB, ako vyplýva z predpokladu, že všetky sú 8-bitové. Backend určí formát každej kamery na základe jej modelu.
- **Admittance (`burst_fits_nic_ring`) zohľadňuje odber**, nie na základe porovnania „celý burst vs. prstenec“: sim-emit sa uplatňuje, keď hostiteľ vyprázdňuje prstenec RX rýchlejšie, ako ho kamery napĺňajú. Hostiteľ 10G + kamery 1 GbE**prijímajú** snímky v plnom rozlíšení aj vtedy, keď burst presahuje kapacitu prstenca; 1 GbE hostiteľ blokuje (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` je konzervatívny strop pre sériové načítanie** — `max(readout+emit, N×emit)` s vysielaním na kameru obmedzeným na 1 GbE Camera Link, nezávisle od expozície. Napr. ~2,8 fps pre pole so 4 kamerami v plnom rozlíšení a 12-bitovej hĺbke (zodpovedá nameranej hodnote ~2,7–3,0 v runtime). Úplný model: [Referencia CLI→ Model fps a burst pre pole](cli-reference.md#array-fps--burst-model).
- **Po nepretržitom snímaní projekcia uvádza, akú rýchlosť si udržala.** Snímanie trvajúce 30 s alebo viac pri automatickej rýchlosti zaznamenáva rýchlosť, na ktorej sa ustálila pri danej veľkosti poľa na tomto počítači; krátke pokusy regulátora rýchlosti o dosiahnutie rýchlejšej rýchlosti sa nezohľadňujú. Od tohto momentu riadok v `array-connect`, analýza v `connect_array()` a panel Nastavenia poľa uvádzajú túto frekvenciu, ak je nižšia ako frekvencia modelu, spolu s informáciou o tom, čo ju obmedzilo v `cycle_limiter`: `"clock"` (pole bežalo pri rýchlosti, na ktorej sa ustálilo, pričom cyklus snímania čakal na spúšťací takt), `"loop"` (cyklus snímania nemal dostatok času), `"capture"` (zachytávanie zaostalo a preskočilo skupiny) alebo `"measured"` (nie je uvedené, o ktorý prípad ide). Hodnota modelu je uvedená v `wire_fps_max`. Pole stále stúpa na hodnotu, ktorú prenáša linka.
- **Prekročenie kapacity (`oversubscribed: true`) znamená, že linka nedokáže prenášať pole pri žiadnej frekvencii, ktorú kamery dosahujú** — polia fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) majú hodnotu 0 a zlučovanie snímok to nedokáže vyriešiť (znižuje počet bajtov na snímku, nie počet bajtov za sekundu pri danom tempe). Samotné pomalé pripojenie toto nespôsobuje: namiesto toho klesá snímková frekvencia a ak nie je možné nastaviť dostatočne pomalé tempo kamier, úroveň sa zmení na `slip-emit-and-capture` (sériová). Riešením je menej kamier, jumbo rámce alebo rýchlejšia sieťová karta; `max_cams_collision_safe` hlási maximálnu kapacitu vášho kábla a kamier. Odpoveď obsahuje aj kódy `aggregate_demand_bps`, `collision_safe_ceiling_bps` a `per_cam_floor_bps`. Pozrite si časť [Prekročenie kapacity](#over-subscription-the-per-cam-floor).

### Zisťovanie a vypisovanie

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Každá z nich prijíma iba kľúčové slová `backend_url=` a `timeout=30.0` a vráti zoznam slovníkov. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) prijíma aj `auto_start_backend=True`; volania `list_*` nikdy nespustia backend.

---

## Smart-AE / Smart-Capture

Polia LATTICE spúšťajú nepretržité automatické zaostrovanie (AE) na pozadí hneď po pripojení, ale u novo nasmerovanej scény trvá chvíľu, kým sa obraz ustáli. **Smart-capture** je praktická funkcia: zisťuje expozíciu každej kamery, čaká, kým sa pole v rámci okna ustáli, a potom spustí snímanie. Je to ekvivalent v grafickom rozhraní: tlačidlo „smart“ capture v desktopovej aplikácii volá ten istý koncový bod backendu.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Pri ovládaní prostredníctvom `ChlorosProject` (nasledujúca časť) máte k dispozícii viac nastavení:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Politika inteligentnej automatickej expozície (smart-AE) je štandardne konzervatívna. Pre náročnú radiometrickú prácu nastavte prísnejšie hodnoty v `exposure_tolerance_pct`; pre rýchlo sa meniace scény, kde vám stačí „dostatočná presnosť“, nastavte voľnejšie hodnoty.

---

## Relácie senzorov DAQ

Trvalá skupina backendov pre spektrálne senzory (DAQ-U cez USB, DAQ-M cez BLE, DAQ-E cez Ethernet). Zrkadlí povrch kamery: inteligentná detekcia, opätovné použitie skupiny, idempotentné pripojenie.

### Inteligentná detekcia (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Priorita: Ethernet → BLE → USB. Odovzdajte akýkoľvek jeden explicitný náznak na fixovanie prenosu.

### Fixovaný prenos

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` určuje jeden senzor a otvorí sa výlučne tento senzor. Porovná sa so senzormi, ktoré sú už otvorené, potom sa od hostiteľa DAQ-E alebo z IP adresy (v prípade DAQ-E, ktorý sieťové skenovanie vidí iba podľa IP adresy, sa požiada o jeho názov, MAC a sériové číslo v režime iba na čítanie, takže ho nájde aj ktorýkoľvek z nich), BLE MAC zariadenia DAQ-M a identifikátor, ktorý zariadenie DAQ-U alebo DAQ-M nahlási po otvorení (akékoľvek iné otvorené zariadenie na kontrolu sa opäť zatvorí). Ak sa nenájde, `connect_daq_sensor` vyvolá výpis `ChlorosConnectError` s informáciami o tom, čo sa našlo; vrátane adresy (`port=` / `mac=` / `eth_host=`), ktorá obsahuje iný senzor, vyvolá sa tiež. Nikdy sa nevráti k akémukoľvek viditeľnému senzoru — na to slúži volanie s príznakom nula.</id></sn></id></sn>

### `connect_daq_sensor()` Podpis

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

Senzor, ktorý sa už nachádza v skupine, sa vráti tak, ako je (vypožičaný): `integration_time`, `frame_avg` a `enable_ae` sa naň neuplatňujú. Najprv ho odpojte, aby ste ho mohli otvoriť s vašimi nastaveniami. Výnimkou je kód `cap_id`: na vypožičanom senzore sa nastavuje pomocou kódu `set_cap()`. Ak sa tento kód vynechá, platí predvolené nastavenie backendu (pozri poznámku pod tabuľkou metód).

### Metódy `DAQSensorSession`

| Metóda | Popis |
| --- | --- |
| `status(timeout=10.0)` | Súhrn záznamu v poole (stav streamovania/nahrávania, rozsah vlnových dĺžok, kalibračný kľúč, integračný čas, frame_avg, stav AE). Pre DAQ-E, ktorého stream prestal vysielať: `reconnect` (ak je aktívne automatické opätovné pripojenie backendu: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None` počas prenosu), `stream_lost_s`, `reconnects` a `last_reconnect`, a `record_gaps` pre prebiehajúci záznam; `rx_stream_fault` označuje stav opätovného pripojenia, kým nedorazí prvé spektrum. |
| `latest(*, n=1, timeout=10.0)` | Vráti až N najnovších rámcov spektra (každý `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Obnovenie / pozastavenie prenosu (identifikátor zostáva otvorený). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Spustí nahrávanie súboru .daq. Vráti cestu k súboru. Nefunguje pre DAQ-U/M bez kalibračného balíka AWS (DAQ-E je výnimkou). |
| `record_stop()` | Zastaví nahrávanie. Vráti `{path, rows, warning, gaps}`; `warning` je rovnaké ako `None` v prípade bezchybného záznamu, inak sa vráti diagnóza backendu (napríklad prázdny súbor alebo medzera v dátovom toku), ktorá sa zároveň vypíše na stderr. `gaps` vypisuje každý úsek, v ktorom prúd nedodal nič: `start_utc`, `end_utc` (`None`: stále chýba pri zastavení), `duration_s`, `reconnected`, `marked_in_file`. Pri nepretržitom zázname je tento riadok prázdny. |
| `set_cap(cap_id, timeout=15.0)` | Nastavte profil korekcie maximálnej hodnoty, ktorý sa bude uplatňovať na hodnoty od nasledujúceho merania: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"` alebo `"none"` (alebo `None`) pre holý snímač. Vráti súhrn snímača, ktorého hodnotu `cap_id` opäť prečíta. DAQ-E-S udržiava hodnotu `as_recorded` bez ohľadu na požiadavku (má zabudovaný difúzor). Vytvorí hodnotu `ChlorosConnectError` pre profil, ktorý sa nedá načítať. CLI: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Iba DAQ-E: opätovné zapnutie PTP pre jednotku, ktorej PTP je vypnuté (`status()["rx_ptp_enabled"]` je rovnaké ako `False`; nastavenie je uložené v zariadení, takže zostane vypnuté, kým sa opäť nezapne). Vráti aktualizovaný stav. Vyvolá výnimku `ChlorosConnectError` pre DAQ-U / DAQ-M alebo ak zariadenie nepotvrdí. Program Chloros nikdy sám nezapne zariadenie späť, pretože vypnutý stav môže byť zámerný. CLIA: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Iba DAQ-E: vlastné nastavenia zariadenia. `firmware` (`running`, `bundled`, `update_available`), `ip` a `ip_mode` (`dhcp` / `linklocal` / `static`: čo beží teraz), `network` (uložené nastavenie: `mode`, plus `ip` / `mask` / `gateway` v statickom režime), `network_change_pending` (platí dovtedy, kým jednotka nezačne pracovať s uloženým nastavením, ktoré obsahuje aj hodnotu `set_network(..., reboot=False)`), `auto_stream`, `ptp`. `firmware["update_available"]` je `None`, keďverzia zariadenia sa nedala prečítať. Pokiaľ prúd zariadenia nedosahuje tento počítač, prečíta sa pri novom pripojení a slovník pridá `stream_silent` (`True`), `stream_fault` a `reconnect`; v prípade, že jednotka neodpovedá, sa vygeneruje hlásenie. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E iba: DHCP (`"auto"`) alebo statická adresa. Zariadenie sa reštartuje, aby sa nastavenie uplatnilo, a opäť sa pripojí; táto relácia pokračuje v práci a `address` sleduje zariadenie. `reboot=False` to iba uloží. Pozrite si [Nastavenia zariadenia DAQ-E](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Iba DAQ-E: prenos pri zapnutí bez pripojeného softvéru (uložené v zariadení). Vráti potvrdené nastavenie. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Iba DAQ-E: reštartujte zariadenie a znovu ho pripojte s pôvodnými nastaveniami. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Len pre DAQ-E: nahrajte firmvér dodávaný s touto verziou Chloros do zariadenia so staršou verziou a potom ho znovu pripojte (trvá to približne minútu; nevypínajte ho). Vráti kódy `updated`, `fw_before`, `fw_after`, `bundled` a `warning`, ak sa zariadenie obnovilo pomaly alebo sa obnovilo s inou verziou; kód `updated` je `False` pre zariadenie, ktoré už má aktuálnu verziu. Vyvolá výnimku `ChlorosConnectError` pre jednotku, ktorej verziu nie je možné prečítať. CLI: `pool-update-firmware`. |
| `disconnect()` | Uvoľnenie z fondu. Žiadnapre pripojené rukoväte, ktoré nie sú vo vlastníctve. Na senzore, ktorý otvorila táto relácia, je to `force_disconnect()` a vyvolá výnimku `ChlorosConnectError`, ak uvoľnenie nie je potvrdené (ukončenie bloku `with` nikdy nevyvolá výnimku). |
| `force_disconnect(timeout=30.0)` | Uvoľnenie senzora bez ohľadu na to, kto ho otvoril — napríklad senzor, ktorý v poole zanechal havarovaný skript a ku ktorému sa nový skript dostane len ako pripojená rukoväť, ktorej `disconnect()` nevykonáva žiadnu akciu. Ukončenie senzora iného klienta ukončí akékoľvek nahrávanie, ktoré práve prebieha. Vráti sa, akonáhle backend potvrdí uvoľnenie, alebo ak nie je čo uvoľniť (senzor nie je v poole alebo nebeží žiadny backend); uvoľnenie, pri ktorom došlo k časovému limitu alebo ktoré backend odmietol, vyvolá výnimku `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Atribúty: `sensor_id`, `model`, `transport`, `address` (nasleduje po DAQ-E, ktorého adresu `set_network()` zmenil), `info` a `backend_url`.

> **Uveďte názov krytky.** Modely DAQ-U, DAQ-M a DAQ-E sa dodávajú bez kondenzátora a zariadenie Chloros nedokáže zistiť, či je nainštalovaný: senzor pripojený bez `cap_id` dostane hodnotu `sunshine_cosine`, čo je predpoklad, ktorý podľa záznamov uvádza, že vykonalo (`cap_id_source=auto_default`). Holý DAQ-U po tejto korekcii vykazuje približne 30-násobne vyššie hodnoty, preto pre holý senzor zadajte `cap_id="none"` (alebo `set_cap("none")`) a profil krytu, ktorý ste inak namontovali.

### DAQ-E-S — s pevným difúzorom a senzorom náklonu

**DAQ-E-S** je zariadenie DAQ-E s dvoma rozdielmi, ktoré sú dôležité pre softvér. Všetko ostatné — vyhľadávanie, pripájanie, PTP, zaznamenávanie, nižšie uvedené nastavenia zariadenia — funguje presne tak ako pri zariadení DAQ-E.**Jeho difúzor je pevne zabudovaný, takže nevyžaduje žiadnu korekciu krytu.** Difúzor typu Sunshine je zabudovaný a bol na zariadení už v čase merania jeho továrenskej kalibrácie, takže táto kalibrácia už udáva skutočné ožiarenie smerom nadol. Chloros nastaví svoj limit na `as_recorded` (bez profilu) bez ohľadu na to, čo sa požaduje: limit s rovnakým názvom (`sunshine_cosine`, `fov_*`, `none`) sa ignoruje so zobrazením správy a jeho záznamy ukladajú hodnotu `cap_id_source=model`, takže varovanie o predpokladanom obmedzení sa v jeho prípade nikdy nespustí. Neaplikujte na jeho údaje limit ručne: `sunshine_cosine` v hornej polohe by vykazoval približne 11-násobne vyššiu hodnotu.

**Je vybavený 3-osovým akcelerometrom.** Kosínusová charakteristika difúzora slnečného žiarenia platí len vtedy, keď je zariadenie vo vodorovnej polohe, takže naklonený senzor hlási nesprávnu intenzitu žiarenia — a odrazivosť túto chybu preberá — bez akýchkoľvek ďalších príznakov. Každé meranie DAQ-E-S preto obsahuje informáciu o svojej polohe:

| Pole | Význam |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Uhly v stupňoch. `None`, ak firmvér označí uhol za nedefinovaný (voľný pád, nefunkčná os). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Zrýchlenie v mili-g. `mag_mg`, ak je hodnota vzdialená od ~1000, znamená to, že sa jednotka pohybovala a uhly pre daný vzorok nemajú význam. |
| `cal_applied` | Nastavené: osi a uhly sú v súradnicovom systéme difúzora (uplatňuje sa kalibrácia upevnenia zariadenia). Vypnuté: surový súradnicový systém dosky, ktorý môže vykazovať sklon až ~2,3°, ktorý nie je skutočný. |
| `sample_age_ms` | Vek vzorky z akcelerometra v čase odoslania spektra. |
| `temp_raw` | **Trend** teploty čipu bez kalibrácie — iba rozdiely, nikdy nie nameraná hodnota teplomera. |

`DAQSensorSession.latest()` ich vráti pod kľúčom `imu` každého rámca (`None` na DAQ-U, DAQ-M alebo bežnom DAQ-E). `.daq` zaznamenané z DAQ-E-S ukladá rovnaké hodnoty do svojich stĺpcov `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) vedľa každého spektra, takže čitateľ môže rozlíšiť oblak od prevráteného statívu. Na všetkých ostatných modeloch sú tieto stĺpce prázdne.

Prúd údajov o polohe hlási údaje vo vlastných poliach v `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), ktoré sú oddelené od spektrálnych, aby sa výpadok akcelerometra nikdy neinterpretoval ako medzera v zázname. `discover_daq_sensors()` a `model` danej relácie uvádzajú `DAQ-E-S`.

### Nastavenia zariadenia DAQ-E

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **V sieti tohto počítača musí byť statická adresa.** `set_network` vyvolá chybu `ChlorosConnectError` v prípade, že podsieť nezahŕňa adresu tohto počítačav sieti senzora: po reštarte sa k senzoru nebude dať dostať, aby sa to dalo vrátiť späť. Kód `force=True` použite len vtedy, ak bude tento počítač presmerovaný do novej podsiete.
- **`set_network`, `reboot` a `update_firmware` reštartujú senzor** a vyvolajú chybu `ChlorosConnectError` počas nahrávania, a ak sa nevráti do `comeback_timeout` sekúnd. ID senzora a názov hostiteľa `daq-e-<id>.local` sa nikdy nemenia.
- **Záznam, ktorý sa začal tesne pred reštartom, sa týmto reštartom uzavrie a pomenuje.** Keď sa senzor vráti, vrátený slovník obsahuje `recording_closed` (cestu k súboru) a hodnotu `warning`, ktorá to uvádza. Ak sa senzor nevráti alebo zlyhá prenos firmvéru, namiesto toho to uvádza hodnota `ChlorosConnectError`: jej správa končí varovaním a sú nastavené hodnoty `e.recording_closed` / `e.warning` sú nastavené (oba `None`, ak nebolo uzavreté žiadne nahrávanie). Po obnovení spojenia so senzorom spustite nahrávanie znova.
- **Pripojí sa iba samotný senzor.** Po reštarte sa zariadenie musí prihlásiť tam, kde bolo -- a po `set_network("auto")` aj pod svojím menom `.local` -- svojím menom, názvom hostiteľa a modelom, z sieťového adaptéra (MAC), ktorý mal; `address` sa stane IP adresou, ktorá odpovedala. Odpoveď pod daným názvom sa počíta iba s touto MAC adresou, takže z počítača, ktorý sa k senzoru pripája cez smerovač, alebo z predvoleného NAT sieťového prostredia WSL2, senzor, ktorý bol presunutý prostredníctvom DHCP, nie je opätovne pripojený (`ChlorosConnectError`; pripojte ho znovu na jeho novú adresu).
- **Aktualizácia firmvéru vyžaduje, aby sa senzor opäť pripojil.** `update_firmware()` osloví zariadenie a zariadenie otvorí späť TCP spojenie s týmto počítačom na stiahnutie obrazu na porte, ktorý vyberie operačný systém. Backend načúva na adaptéri, ktorý dosahuje zariadenie (na každom adaptéri, len ak nedokáže určiť, ktorý to je), a obsluhuje iba toto zariadenie: akékoľvek iné pripojenie je odmietnuté (`[DAQ-FW] … refused a firmware connection from <ip>` vo výstupe backendu), kým čaká. Firewall, ktorý blokuje pripojenie, spôsobí zlyhanie aktualizácie s chybou `device never connected back on TCP <port>`. Inštalátor systému Windows pridá pre to pravidlo (pozri [Krok 1](#windows-exe)); na inom firewalle povoľte prichádzajúci TCP prevádzku do backendu zo podsiete senzora.
- **V prostredí WSL2 použite zrkadlenú sieť** (`networkingMode=mirrored` v `.wslconfig`): v predvolenom režime NAT je každá statická adresa odmietnutá (vlastná adresa WSL nie je v sieti senzora), prúd sa nedostane do WSL a aktualizácia firmvéru sa nemôže dokončiť.
- Grafické rozhranie si udržiava vlastné pripojenie k senzoru: reštart zo skriptu ukončí reláciu grafického rozhrania s ním. Grafické rozhranie má rovnaké nastavenia (karta Svetelné senzory, nastavenia senzora, Sieť).

### Vyhľadávanie — nájdenie adresy na pripojenie

`discover_daq_sensors()` prehľadáva rozhrania USB / BLE / ETH a hľadá senzory, ktoré by ste *mohli* otvoriť. Je to ekvivalent `discover_lattice_cameras()` pre DAQ a jediný spôsob, ako získať **BLE MAC adresu zariadenia DAQ-M** — zariadenie DAQ-E má názov hostiteľa a zariadenie DAQ-U má port COM, ale MAC adresa nie je vyznačená na zariadení ani uvedená v operačnom systéme.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Pole | Popis |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM port / BLE MAC / názov hostiteľa — odovzdať na `connect_daq_sensor` ako `port=` / `mac=` / `eth_host=`. |
| `display` | Čitateľný názov. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, alebo `None`. Ethernetový senzor má označenie `DAQ-E-S` alebo `DAQ-E`, ako sa identifikuje na požiadanie, len na čítanie, počas skenovania (`DAQ-E`, ak neodpovie včas). Sériový port USB sa identifikuje ako `DAQ-U`, ak jeho mostíkový čip patrí medzi tie, ktoré používa DAQ-U (FTDI, CH340, CP210x, Prolific) — čo naznačuje popis portu, nie sonda — a `None` pre akékoľvek iné USB zariadenie, ktoré je stále uvedené v zozname, takže ho môžete odoslať ako `port=`. Sériový port bez identifikátora výrobcu USB a bez takéhoto mostíkového čipu (UART na základnej doske, ako napríklad `/dev/ttyS*` v Linuxe, sériové pripojenie Bluetooth, Intel AMT) nemôže byť DAQ-U a nie je uvedený v zozname. |
| `extra` | Podrobnosti podľa typu prenosu (inzerovaný názov BLE; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). Prázdne hodnoty sú vynechané. |

| Parameter | Predvolená hodnota | Popis |
| --- | --- | --- |
| `transports` | všetky tri | Sekvencia (alebo reťazec vo formáte CSV) obmedzujúca skenovanie. Oplatí sa ju zadať, ak viete, čo chcete — BLE je pomalšia časť. |
| `scan_timeout` | 5 | Dĺžka okna skenovania na jeden transport v sekundách; backend obmedzuje hodnotu na 1–20. |
| `timeout` | 60,0 | Horná hranica HTTP pre celé volanie (rovnako ako inde v SDK). |
| `auto_start_backend` | `True` | Spustí lokálny backend, ak žiadny nebeží. Nikdy sa nespustí pre vzdialený `backend_url`. |

> **Senzory, ktoré sú už otvorené v poole, sa nezobrazujú.** Pripojené periférne zariadenie BLE prestane vysielať reklamy a otvorený COM port nie je možné skenovať, takže zoznam vyhľadávania uvádza len to, čo je *dostupné na pripojenie*. Hneď po pripojení zariadenia sa očakáva prázdny výsledok — pre zariadenia, ktoré už máte pripojené, použite `list_daq_sensors()`. Transporty, ktorých skenovanie nemožno spustiť (nie je nainštalovaný bleak / zeroconf), sa preskočia namiesto vyvolania chyby, takže zariadenie bez Bluetooth stále dostane odpovede pre USB a ETH.

### Zoznam

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Ktorý DAQ napája ktorú kameru — `daq_bindings()`

Odrazivosť snímky a súbor `.daq`, do ktorého sa ukladá, používajú DAQ priradené k danej kamere alebo poli, inak najnovšie DAQ, ktoré má backend k dispozícii. Výber svetelného senzora v grafickom rozhraní a súbor `chloros-cli daq pool-bind` to nastavujú; rovnako ako:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` vráti `{"cameras": {serial: b}, "arrays": {array_id: b}}`: každý `b` obsahuje `binding` (nastavená hodnota), `source` (`project`, z `sensors.json` projektu alebo `runtime`), `known` / `known_as` (či názvy kľúčov kamery alebo poľa sú známe niektorému z Chlorosov a ako: `connected`, `project` — `cameras.json` otvoreného projektu — alebo v prípade kamery `seen`: už bol na tomto počítači otvorený; `known: False` pre kľúč, ktorý nič neoznačuje) a `resolved` (senzor v skupine, na ktorý je teraz priradený: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). Viazanie počas behu má prednosť pred projektompočas danej relácie; súbor projektu sa nemení. Odpojenie kamery, poľa alebo zariadenia DAQ zruší všetky priradenia tohto hardvéru, bez ohľadu na to, kto ich nastavil (členovia poľa sa priraďujú spolu s poľom).

`set_daq_bindings()` viaže iba kameru, ktorá je pripojená, nachádza sa v otvorenom projekte alebo bola predtým otvorená na tomto počítači, a pole, ktoré je pripojené alebo sa nachádza v otvorenom projekte (alebo kľúč, ktorý je už viazaný). Akýkoľvek iný kľúč vyvolá `ChlorosConnectError` s názvom (`400`, `code`, `UNKNOWN_BINDING_TARGET`, s kľúčmi `unknown`), a v tomto volaní sa **nič** nepripája. `None` vymaže akýkoľvek kľúč.

### Okolité + odrazivosť objektu — `light_sensor_reflectance()`

Dva združené DAQ, z ktorých jeden smeruje nahor k oblohe (okolie) a druhý nadol na cieľ (objekt), poskytujú odrazivosť cieľa a indexy vegetácie na základe svojich najnovších meraní: dvojica Okolie + Objekt v grafickom rozhraní a `chloros-cli daq reflectance` bez toho, aby bolo potrebné senzory otvárať druhýkrát. Oba musia byť pripojené a musia vysielať dáta.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Pásma.** Odrazivosť každého pásma jeintegrované ožiarenie objektu v danom pásme voči ožiareniu okolia. `daq_band_presets(filter_code=None)` uvádza zoznamy pásmových sád: po jednej pre každý filter LATTICE M3C (`RGN`, `OCN`, `NGB`, vlastný pásmový model kamery, takže NDVI z DAQ a NDVI z kamery sú rovnaké veličiny) a `GENERIC` širokopásmové. Jeho hodnota `suggested` je prednastavená hodnota zodpovedajúca `filter_code`, inak ide o filter prvej pripojenej kamery (`suggested_from` určuje, ktorý).
- **Vráti** `bands` (pre každé pásmo `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (vstavané), `custom` (každý s `value` alebo `valid: False` a `error`), `invalid_bands`, `reflectance_type`, `preset` a hodnoty týchto dvoch meraní `timestamps` (`ts_ns`). Rozsah, ktorý senzor okolia nedokázal zmerať (mimo svojho rozsahu, na úrovni šumu), je neplatný, s uvedením dôvodu, a každý index, ktorý ho potrebuje, je vynechaný a nikdy sa nepočíta voči nule.
- **Zdanlivé alebo relatívne.** Indexy závislé od mierky (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) vyžadujú absolútny pomer, preto sa vypočítajú len vtedy, ak sú oba senzory kalibrované na výkon (každý má `calibration_bundle_sha`): `"Apparent Reflectance"`. V opačnom prípade je výsledkom `"Relative Reflectance"`, obsahujúci iba indexy nezávislé na mierke.
- **Dve rôzne vlnové mriežky** sa porovnávajú na základe ich prekrývania: body senzora okolia v rozsahu objektu a spektrum objektu interpolované na ne.
- **Dva senzory, nikdy len jeden.** Ak sa jeden senzor označí ako obidva (rovnaké ID, v akomkoľvek pravopise, ktorý pool akceptuje), vyvolá to chybu `ChlorosConnectError`: hodnota senzora pre seba samého je 1,0 v každom pásme, čo nemeria nič. Backend to tiež odmietne (`POST /api/daq/indices` odpovedá na `400`, `code` na `SAME_SENSOR`, keď `ambient_sensor_id` a `object_sensor_id`, ktoré posiela SDK, predstavujú jeden združený senzor, napr. jeho ID a jeho port).

### Spoločné využívanie s rozhraním CLI a inými skriptmi

Skupina senzorov DAQ na backende je zdieľaná všetkými klientmi CLI a SDK na danom počítači. Ak je senzor už v skupine — otvorený funkciou `chloros-cli daq pool-connect` alebo iným skriptom — volanie funkcie `connect_daq_sensor(port="COM3")` z jazyka Python vráti vypožičanú referenciu: jeho `disconnect()` je bezoperácia, takže váš skript neodoberie senzor klientovi, ktorý ho otvoril, pri ukončení rozsahu. To isté platí pre `sensor_id=` (zadané pomocou `:` alebo `-`, obe možnosti sú platné), pre DAQ-E identifikovaného podľa IP adresy, ak bol otvorený podľa hostname (alebo naopak), a pre inteligentnú detekciu s nulovým príznakom, keď backend odpovie senzorom, ktorý už mal k dispozícii — blok `with` uvoľňuje vždy len senzor, ktorý sám otvoril.

GUI Chloros **nie je** súčasťou tohto fondu: jeho panel DAQ prevádzkuje vlastné pripojenie k senzoru, takže `connect_daq_sensor` si nikdy nepožičiava senzor, ktorý má otvorený GUI. Nepoužívajte ten istý senzor súčasne z grafického rozhrania a zo skriptu. DAQ-U alebo DAQ-M, ktoré grafické rozhranie bežne drží, sa jednoducho nepodarí otvoriť, ale DAQ-E prijme druhú reláciu: pripojenie skriptu zmení nastavenia zberu údajov zariadenia v rámci grafického rozhrania a jeho `disconnect()` zastaví prúd zariadenia, čo ukončí živý náhľad grafického rozhrania a akékoľvek nahrávanie, ktoré práve prebieha. Najprv zatvorte senzor v grafickom rozhraní.

### PTP Grandmaster — `time_sync_status()`

Backend spúšťa PTP grandmaster, z ktorého kamery LATTICE a jednotky DAQ-E čerpajú svoje časové značky. Spúšťa sa na požiadanie: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` a `connect_all()` ho spustia, ak ešte nebeží — vrátane pripojenia, ktoré následne zlyhá, napríklad keď sa `connect_all()` pokúsi pripojiť k jednotke DAQ-E, ktorá neexistuje — a rovnako tak aj `test_array_sync()` a každé volanie, ktoré otvára kamery pre jeden príkaz (`capture_cameras_transient()`, `start_multi_camera_stream()`, volania na zarovnanie, `capture_live_index()`).

- **Kde sa to oznamuje.** Na káblových sieťových adaptéroch, ktoré sú pripojené ku kamerám (vrátane adresy link-local `169.254.x.x`), inak na adaptéri predvolenej trasy, ak je tento káblový. Nikdy cez Wi-Fi alebo virtuálny adaptér (Tailscale, WSL): za nimi sa nenachádza žiadna kamera ani DAQ-E. Ak nie je k dispozícii žiadny takýto adaptér, beží bez oznamovania (`health` `NO_SOCKETS`) a priradí sa k adaptéru, ktorý sa aktivuje pri ďalšom pripojení kamery alebo DAQ.
- **Po prerušení spojenia.** Keď adaptér kamery stratí spojenie a znovu ho nadviaže (vytiahnutie kábla, výmena prepínača, opätovné vyhľadanie USB adaptéra), grandmaster sám obnoví svoje sokety bez reštartu, a opätovne skontroluje všetky pripojené zariadenia DAQ-E, pričom opätovne aktivuje tie, ktoré zostali v stave „stuck“. Obnoví sa tiež vtedy, keď všetky kamery a zariadenia DAQ-E, ktoré obsluhoval, zostanú neaktívne po dobu 15 sekúnd.
- **Dostáva sa k káblu?** Prečítajte si `time_sync_status()` (`GET /api/time-sync/status`). `state` iba uvádza, že vlákno hlavného modulu beží; `health` uvádza, či jeho pakety odchádzajú: `OK` (nezistila sa žiadna porucha), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (niektoré adaptéry zlyhávajú) alebo `PEERS_SILENT` (kamery a zariadenia DAQ-E, ktoré obsluhoval, prestali odpovedať), s vetou v `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` počítajú len správy, ktoré odišli; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` a `last_delay_req_age_s` opisujú tie, ktoré neboli odoslané, a `rebinds` / `last_rebind` popisujú opätovné odoslania. `sync_sends_slow` počíta synchronizácie, ktorých odosielanie trvalo dlhšie ako 200 us, a `sync_send_max_us` je najdlhšia; čas odoslania každého Follow_Up sa počíta maximálne do 200 us. Grandmaster beží vo vlastnom procese (`own_process`, `process_pid`) s vysokou prioritou v systéme Windows (`process_priority`), takže vyťažený backend nemôže oneskoriť časové značky a posúvať hodiny každej kamery naraz; ak sa tento proces nedá spustiť, beží v rámci backendu ako doteraz (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Funkcia | Čo robí |
| --- | --- |
| `time_sync_status()` | Stav grandmastera, jeho stav, identifikátor hodín, priority a počítadlá správ (viď vyššie). `chloros-cli time-sync status`. |
| `time_sync_peers()` | PTP podriadené zariadenia, ktoré zachytáva (kamery, DAQ-E): `peers`, každý z `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Každá pripojená kamerová sústava: `PtpStatus`, `PtpOffsetFromMaster` a `PtpMeanPathDelay` (kamera, ktorej firmvér nemá žiadne uzly PTP, má iba sériové číslo). `time-sync cameras`. |
| `restart_time_sync()` | Zastavte a reštartujte grandmaster; po tom sa znovu skontroluje PTP každého zoskupeného DAQ-E (`daq_ptp_rechecked`). Vyvolá výnimku `ChlorosConnectError`, ak sa sa nevráti. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Jeho priority PTP (0–255, nižšia hodnota vyhráva voľbu najlepšieho mastera) pre sieť s iným PTP masterom. Vyžaduje, aby bol spustený. `time-sync set-priority`. |

Každý z nich vezme `backend_url=` a `timeout=` a spustí lokálny backend, ktorý nebeží.

---

## Nástroje pre kamery

Čo robí `chloros-cli lattice` okrem pripojenia a zachytávania, ako bežné funkcie v `chloros_sdk` (modul `chloros_sdk.camera_utils`). Každá z nich volá trasu backendu, ktorú používa príkaz CLI, s pravidlami CLI, takže sa obe správajú rovnako. Prijímajú **sériové** číslo kamery, nie reláciu, a rovnaké kľúčové slová `backend_url=` / `timeout=` ako volania smart-connect; ak backend nebeží, spustí sa, rovnako ako to robí `connect_camera()`. V prípade odmietnutia sa vyvolá výnimka `ChlorosConnectError` s dôvodom zo strany backendu.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Funkcia | Čo robí |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | Farebný profil displeja: `raw`, `linear`, `natural`, `enhanced` alebo `custom_temp` a jeho farebná teplota (obmedzená na 2000–10 000 K; načítaná, ak je profilom `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Sýtosť a kontrast displeja, každý v rozmedzí 0–200 % (100 = bez zmeny); `reset=True` nastaví obe hodnoty späť na 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Zníženie najjasnejších a najbielejších pixelov zobrazenia: -100..0 (zaokrúhlené smerom k nule, potom obmedzené; 0 = vypnuté). |
| `camera_white_balance_once(serial, settle_s=None)` | Jednorazové vyváženie bielej: kamera krátko streamuje, konverguje a vráti hodnotu `red_ratio` / `blue_ratio` (`ratios_read` False, ak ich nemôže nahlásiť). Udrží vyváženie bielej, ktoré vypočítala; všetky ostatné nastavenia, ktoré jej otvoril, sa vrátia tak, ako boli zistené. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (nepretržité) alebo `"off"`; alebo pevné pomery vyváženia `red` / `blue`, ktoré ako prvé vypnú automatický režim. Pomer, ktorý kamera odmietne, je uvedený pod `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | Prednastavenia kamery a ich použitie: kamera zachová nastavenia s názvami prednastavení (odpoveď je `settings`, keďže kamera ich číta následne, napr. snímkovú frekvenciu, pri ktorej beží; `None` pre pole, ktoré prednastavenie neoznačuje), a všetky ostatné nastavenia, ktoré sú v súbore otvorené, sa vrátia do pôvodného stavu. `"default"` obnoví továrenské predvolby kamery (továrenské nastavenia s voľne bežiacou automatickou expozíciou, vypnutým obmedzením priepustnosti spojenia a vypnutou korekciou chýb, pričom sa zachová kalibrovaná vyváženosť bielej kamery RGB), ktoré sa neuložia do jej pamäte: odpoveď obsahuje `factory_defaults: True` a všetky polia z `settings`. V `camera_presets()` je `default` rovnaké ako `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Modely filtrov a podrobnosti o jednom modeli (pásma, podmienkové číslo spektrálnej odozvy). Model filtra kamery sa načíta z reťazca modelu; nenastavuje sa ručne. |
| `camera_firmware_status(serial=None)` | Firmvér každej kameryfirmvér: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` a `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Nahrajte tento firmware verzie Chloros do jednej kamery: obnovenie továrenských nastavení, nahratie, reštart, prepísanie základných nastavení Chloros; kalibrácia kamery zostane zachovaná. Trvá to niekoľko minút; kamera musí zostať zapnutá. `force=True` opätovne nahraje firmvér do aktuálnej kamery. |
| `camera_power_probe(serial)` | Napájacie a teplotné uzly kamery a jej teplota. |
| `sleep_camera(serial)` / `wake_camera(serial)` | Režim nízkej spotreby kamery v pohotovostnom stave (zostáva v spánku, kým nie je prebudená: je zapnutý limit priepustnosti jej spojenia (`DeviceLinkThroughputLimitMode` `On`) na minimálnej hodnote, je zapnutý režim úspory energie, ak ho kamera má), s teplotou pred a po; prebudenie vráti spojenie do stavu, v akom ho našiel režim spánku (limit, režim a oneskorenie paketov `GevSCPD`, ktoré kamera udržiava na hodnote 0, pokiaľ je limit zapnutý; záznam sa uchováva na tomto počítači, takže ho nájde aj prebudenie z neskoršieho backendu), alebo v prípade kamery, ktorú tento počítač nepreviedol do režimu spánku, ale ktorá vyzerá, že spí (jej limit je nastavený na minimálnu hodnotu), s vypnutým limitom a hodnotou `GevSCPD` ponechanou na 0 pri 80 (továrenské nastavenia); limit nastavený nad minimálnou hodnotou sa ponechá tak, ako je. Uchováva sa len stav napájania; všetky ostatné nastavenia, do ktorých sa zapisuje, sa vrátia do pôvodného stavu. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Zachytávajte snímky, kým sa v zornom poli neobjaví kalibračný cieľ MAPIR, a prispôsobte koeficienty pre jednotlivé kanály podľa jeho panelov: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Ethernetové adaptéry tohto počítača pre GigE Vision (MTU, rýchlosť spojenia, jumbo rámce) a akékoľvek `issues`; `estimate=True` pridáva snímkovú frekvenciu a šírku pásma, ktoré by kamery `cameras` dosahovali. |
| `fix_network_settings()` | Použiť nastavenia adaptéra GigE Vision na adaptér kamery tohto počítača. Zmení to nastavenia adaptéra a funguje to len vtedy, ak je program Chloros spustený ako správca; inak sa zobrazí hlásenie s protokolom o zistených nálezoch. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Opravte vinietovanie snímok v priečinku, pričom každá snímka má vlastnú mapu rovinného poľa objektívu: `processed`, `lens_models` (objektív, pre ktorý bol každý súbor korigovaný), `skipped` (s odôvodnením). Bez `lens_model` sa objektív každého súboru odvodí z jeho EXIF Model; súbor `lens_model` sa vzťahuje na súbory, ktorých Model neuvádza žiadny fotoaparát, a súbor, ktorého Model je s ním v rozpore, sa preskočí. Iba súbory. |
| `read_dls(mac, filter=None, camera_serial=None)` | Jedno meranie z DLS (senzor dopadajúceho svetla cez Bluetooth): spektrum, intenzita žiarenia v jednotlivých pásmach a veľkosti pásiem prostredníctvom vlastnej odozvy fotoaparátu, ak je zadané `camera_serial`. `discover_daq_sensors(transports=["ble"])` zistí MAC. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | Hardvér hostiteľa pre kamery a odporúčaná konfigurácia (`report_text`) a CPU / GPU / runtime PyTorch systému. |

- **Niektoré potrebujú kameru len pre seba.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` a `update_camera_firmware()` otvárajú kameru. Pokiaľ ju drží Chloros (pripojená v aplikácii, s `connect_camera()`, ako člen poľa alebo práve používaná iným príkazom, napríklad pri teste poľa alebo pri otváraní kamier na účely snímania), backend odmietne s chybou HTTP 409 `CAMERA_HELD` (v oboch prípadoch, `code` aj `error_code`), pričom uvádza, ako ho uvoľniť. Nastavenia farieb, zvýraznenia, `camera_power_probe()` a čítanie fungujú na pripojenej kamere.
- **Farba, zvýraznenia a vyváženie bielej sú určené pre RGB kamery.** Na jednopásmovej monochromatickej (M3M) kamere tieto nastavenia nič nemenia a vracajú hodnotu `{"skipped": True, "reason": ...}`, rovnako ako CLI. Ide o nastavenia zobrazenia: živý náhľad a snímky na úrovni zobrazenia, nikdy nie o žiarivosť ani odrazivosť.

---

## Prechodné polia, profily zarovnania a indexy

Čo robí `chloros-cli lattice` bez trvalého pripojenia, rovnako ako funkcie v `chloros_sdk` (modul `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` a `index`. Každý z nich odosiela požiadavku, ktorú odosiela príkaz CLI, s predvolenými nastaveniami CLI.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **Samotné otvárajú kamery.** Každý príkaz otvorí svoje kamery, vykoná svoju úlohu a kamery opäť zatvorí, preto potrebuje kamery, ktoré Chloros nedrží (pripojené v aplikácii, s `connect_camera()` alebo ako člen poľa); backend to odmietne s HTTP 409 `CAMERA_HELD` a uvedie, ako jednu uvoľniť. Dočasné otvorenie trvá chvíľu (asi 40 s pre niekoľko kamier). Pre opakované snímanie je je najrýchlejším riešením jednorazové použitie `connect_array()`, ktoré šetrí pri každom type exportu, zatiaľ čo dočasné snímanie ukladá obraz po odstránení bayeresovania. Výnimkou je `calibrate_alignment_profile()`: pri pripojenom poli týchto kamier rieši situáciu z vlastných snímok poľa (`used_live_array`).
- **`serials`** sú kamery, ktoré sa majú otvoriť, pričom prvá spustí spúšť; `None` otvorí všetky kamery v sieti.
- **`settings`** je slovník prepisov nastavení kamery, polia, ktoré nastavujú príznaky v CLI: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` a `preset` (prednastavenie kamery podľa názvu). Zadaná expozícia alebo zosilnenie vypnú automatické riadenie, rovnako ako to robia príznaky.
- **`gpu`**: `True` vynúti warp na GPU, `False` na CPU (`--gpu` / `--no-gpu`); `None` necháva výber na backende.
- **`keep_settings`**/**`clear_settings`** (každá funkcia, ktorá otvára kamery, predvolené nastavenie je `False`): na konci sa každá kamera vráti do stavu, v akom bola nájdená; `keep_settings=True` namiesto toho uloží nastavenia behu do vlastnej pamäte kamery (voľný beh), `clear_settings=True` obnoví túto pamäť na továrenské nastavenia — pozri [Pamäť kamery](#camera-memory--keep_settings--clear_settings). Odpoveď (pre úlohu, v prípade `stop()`) obsahuje `camera_memory`.
- **Cesty sú cesty backendu**: relatívne cesty sa premenia na absolútne voči vášmu pracovnému adresáru, ak je backend na tomto počítači. Funkcie súboru profilu zarovnania (`alignment_profile_info()`, `reorder_alignment_profile()`) čítajú súbor na tomto počítači, rovnako ako to robí CLI.

| Funkcia | Čo robí |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Nepretržite nahráva z jednej kamery, kým nie je zastavená. Vráti hodnotu typu `CliJob`. Snímky z behu typu `tiff` tvoria súbor typu `raw`, do ktorého sa zapisujú snímky z danej kamery (jedna mozaiková rovina pri vlastných hodnotách senzoravlastných hodnôt, nekomprimované); `png` a `jpg` obsahujú 8 bitov, takže tieto behy sú snímky po demosaicovaní určené na prezeranie. Viazané alebo združené DAQ zapisujú jeden súbor `.daq` na každý snímok v behuzložku `daq/` (`daq_files` v jej štatistikách). Každý snímok sa zapíše raz spolu s úplnými metadátami (model, čas snímania, objektív, pásma, kalibrácia na surovom snímku); `stop()` sa vráti až po zapísaní každého zachyteného snímku (až do 25 s) a až potom sa kamera uzavrie. Kamera ponechaná v pohotovostnom režime pre hardvérový spúšťač (`TriggerMode` `On`) je počas behu nastavená na voľný chod (odpoveď na štart `trigger` uvádza, čo bolo zistené). Pri zastavení sa kamera vráti do stavu, v akom bola nájdená — s jej nastaveniami spúšťača a všetkými nastaveniami, ktoré beh zmenil — pokiaľ nie je použitá funkcia `keep_settings=True`, ktorá uloží nastavenia behu do vlastnej pamäte kamery (voľný beh), alebo ak je nastavený kód `clear_settings=True`, ktorý túto pamäť vynuluje. Beh, ktorý nezaznamenáva žiadny snímok, sa po 10 sekundách sám ukončí s kódmi `stop_reason` a `no_frames`, pričom dôvod je uvedený v kóde `last_error`, pričom jeho funkcia `stop()` vráti tento dôvod ako `error`. Snímky, ktoré boli vynechané, pretože zapisovače nestíhali, sa počítajú, pričom `drop_reason` uvádza dôvod. Pre kameru, ktorú drží Chloros, použite `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Synchronizované skupiny snímok v prúde na `fps`, pre skupiny na adrese `count` alebo až do zastavenia. Vráti hodnotu `CliJob` (jeho `start_reply` má `n_cams` a prenos `pacing`); jeho konečné štatistiky zahŕňajú počet zachytených, zapísaných a vynechaných skupín, súbory, ktoré zapisovač nedokázal zapísať, a skupiny, pre ktoré kamera nedodala žiadne údaje. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Jedno synchronizované zachytenie, jeden obraz po odstránení bayera na kameru: `results` (`filepath`, `serial`, `frame_id`) a `pacing`. Objaví sa hlásenie o chýbajúcej kamere, v ktorom sa najskôr uvedie počet uložených súborov. `lattice multi-capture` bez pripojeného poľa. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` synchronizované snímky s pevnou expozíciou: `sync_groups`, `master`, jeden riadok `group_timestamps` na jedno spustenie (kamera, ktorá vynechala jeden záber, chýba v príslušnom riadku) a `sync`: posun každej druhej kamery od hlavnej kamery, spúšť po spúšti (`offsets_us`: na kameru `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), na JEDNOM takte (`clock_domain`: `ptp` alebo `device` — PTP vypnuté, vlastné hodiny prevádzkovej doby každej kamery, takže jej posun je v čase spustenia a synchronizáciu meria len kolísanie). Spúšťač, ktorého časové pečiatky kombinujú hodiny zariadenia a PTP (PTP vypnuté alebo stále sa synchronizuje), sa vynechá a započítava (`groups_mixed_domain`), rovnako ako ten, pri ktorom sú hodiny PTP stále viac ako `agree_within_us` od hlavného (`groups_clocks_disagree`); to potvrdzujú hodnoty `notes`. `clock_offset_mean_us` / `clock_offset_jitter_us` predstavujú najväčšiu priemernú hodnotu a kolísanie na jednu kameru. `success` je nepravdivé, rovnako ako `error`, ak došlo k vynechaniu spúšťača, ak menej ako dva spúšťače synchronizujú všetky kamery na jeden takt, alebo ak sa takty nikdy nezhodujú (`sync.problems`); riadky sa vrátia v oboch prípadoch. Spustí PTP grandmaster, ak ešte nebeží (`time_sync` uvádza, že `grandmaster_running` a `grandmaster_started_by_this_test`), zapne PTP na každej kamere (ako to robí `connect_array()`) a pred prvým spúšťačom počká, kým sa PTP hodiny každej kamery zosynchronizujú s grandmasterom a ustália sa: `PtpStatus` Slave na jednom grandmasterovi, s odchýlkou do 100 us a stabilný po dobu 3 s, pričom každá kamera je takto synchronizovaná, bez prerušenia, po dobu 120 s — približne 2 minúty od zapnutia. Hodiny, ktoré sa práve synchronizovali, majú odchýlku až ~200 us približne 2 minúty po tom, čo sa na nich zobrazí stav „Slave“, a vlastný `PtpOffsetFromMaster` kamery to nezobrazuje; 120 s privedie kamery do odchýlky ~10 us od ustálených hodín. Kamera, ktorá je stále synchronizovaná od predchádzajúceho testu, pri ktorom zostal PTP zapnutý (ako to videl tento backend), počíta od tej synchronizácie, takže test vykonaný hneď po predchádzajúcom už nemusí čakať; kamera, ktorú test zapne, alebo kamera s aktívnym PTP, u ktorej tento backend nezaznamenal synchronizáciu (bola zapnutá inde alebo pred reštartom backendu), čaká svojich 120 s, a v skupine oboch kamier, ktoré sa práve synchronizovali, stále čakajú na svoj čas. Kamera, ktorá opustí režim Slave, prekročí 100 us alebo sleduje iného grandmastera počas 3 odčítaní za sebou (1,5 s) počas čakania, začne odpočítavanie odznova; jedno alebo dve takéto merania sa zaznamenajú (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) a odpočítavanie pokračuje, pričom ID grandmastera, ktoré sa nedá prečítať, sa považuje za neznáme, nie za zmenu. Ak sa odpočet všetkých kamier reštartuje naraz (v priebehu jednej sekundy), ide o pohyb času grandmastera, nie kamier: protokol backenduuvádza `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` to uvádza v `grandmaster_steps` a čakanie, ktoré potom vyprší, začína svoju chybu s `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Limit sa pre to nepredĺži. `ptp_settle_s` (0–600; `0` v prípade, že sa nečaká na ustálenie) nastavuje čas ustálenia pre beh; iná hodnota je odmietnutá (HTTP 400). Čakanie je obmedzené na čas ustálenia plus 90 s (štandardne 210 s, minimálne 90 s) a volanie čaká na backend `timeout` s: štandardne 240 plus čas ustálenia (360 s), rovnako ako to robí `test_array_sync` v rámci `chloros-mcp`. `ptp_lock` má `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (ako dlho bola každá kamera uzamknutá a stabilná), `found_settled` (kamery, ktoré už boli uzamknuté na dobu ustálenia v čase zistenia) a pre každú kameru `status`, `offsets_ns`, `slave_s` (ako dlho je synchronizovaná; None: nie je synchronizovaná) a `remaining_s` (None, ak sa PTP žiadnej kamery nezapne). Protokol backendu obsahuje každých 5 s riadok so stavom každej kamery, informáciou o tom, ako dlho je synchronizovaná a ako dlho jej ešte zostáva. Každá kamera sa zapína s vypnutým PTP; od tohto momentu sa hodiny synchronizujú približne za 10 s. Následne sa každá kamera vráti do stavu, v akom ju test našiel – jej PTP, automatická expozícia a zosilnenie sú počas behu testu nastavené na Vypnuté (`ExposureAuto` / `GainAuto` a manuálne `ExposureTime` / `Gain`), a ostatné, pokiaľ nie je nastavené `keep_settings=True` — teda kamera, pri ktorej zostal PTP zapnutý (`PtpEnable=True` alebo beh s `keep_settings=True`) zostáva uzamknutá a akonáhle tento backend zistí, že je uzamknutá po dobu ustálenia, neskorší test na ňu nečaká. Ak sa hodiny neustália, `success` je pred akýmkoľvek spúšťačom nepravdivé a `error` určuje, čo treba urobiť: kamera, ktorá sa nikdy nezamkla (`not_slave`), znamená, že grandmaster sa k nej nedostáva (`time_sync_status()`); tá, ktorá neustále vypadávala, je označená podľa toho, ako dlho bola zamknutá (`remaining_s`); hodiny sa stále zosúlaďujú (`far`): skontrolujte `time_sync_status()` a aby boli pred testom synchronizované, zapnite PTP a nechajte ho zapnuté (`PtpEnable=True` na každej kamere alebo `keep_settings=True`; test obnoví zistené nastavenia, takže zostane zapnutý), potom ho spustite. `lattice multi-test`. |
| `detect_array_master(line=None)` | Kamery na synchronizačnej linke a predvolený hlavný zariadenie: `master_serial`, `slave_serials`, ktoré testoval `line`. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Aká škála týchto modelov kamier (názvy produktov, `"LATT-M3C-L87-FOCN"`) by sa objavila v sieti tohto počítača: `tier`, `achievable_fps_max`, rozpočet na káble (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` a `fix_suggestions`. Štandardne sa nezobrazujú modely zistených kamier; žiadna kamera nie je otvorená. Informácie o kamerách, ktoré vlastníte, spolu s nastaveniami, ktoré by sa použili pri pripojení, nájdete v [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Vytvorte profil zarovnania na základe živých snímok a uložte ho. `method`: `orb`, `akaze`, `phase`, `checkerboard` (spolu s `checkerboard="9x6"`, vnútorné rohy) alebo `manual` (spolu s `control_points={serial: [[x, y], ...]}`, aj referenčná kamera). `model`: `translation`, `rigid`, `affine` alebo `homography`. Podrobnejšie možnosti predstavujú príznaky v CLI: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` je `{serial: {option: value}}`. `rms_threshold_px` odmieta uložiť horšie riešenie. Vráti `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (pre každú kameru `reprojection_error`, `num_matches`) a `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Jeden zarovnaný viacpásmový obrázok v rámci profilu: `files` (viacpásmový súbor TIFF s jeho `bands` a veľkosťou, plus jeden na pásmo s `per_band` a jeden na kameru s `per_camera`) a `vignette_corrected`. `bands` vyberá a `order` zoradí pásma podľa názvu (zoznam alebo reťazec oddelený čiarkami). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Zosúladené viacpásmové snímky (rovnaké možnosti deformácie ako `capture_aligned_multiband()`), pre `count` alebo až do zastavenia. Vráti `CliJob`; jeho konečné štatistiky zahŕňajú `frames_written` a akékoľvek `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Súbor profilu zhrnutý ako `align-info` ho vytlačí: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (na jednu kameru `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (pre každú kameru `band_names`, `wavelengths_nm`, `enabled`); `profile` je celý súbor. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Zmena súboru profilu na mieste: `order` sú názvy pásiem (každý posúva vrstvu svojej kamery nahor, v tomto poradí), `enable` / `disable` sú sériové čísla. Vráti nový súbor `layer_order`. `lattice align-reorder`. |
| `index_catalog()` | Prednastavenia indexu (`presets`: každý z `name`, `channels`, `formula`) a farba `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Index pre zarovnaný viacpásmový súbor TIFF: `preset` (`NDVI`, `NDRE`, ...) alebo `formula` nad symbolmi, ktoré sa mapujú na pásma (`channel_map`, podľa názvu alebo indexu). Prednastavenýsymbol `channel_map` vynecháva a vyberá jedno pásmo súboru, ktorého názov ním začína (`red` -&gt; `Red_660`, `nir` -&gt; `NIR_850_RGN`), pomenované samotným súborom (jeho zoznamom pásiem alebo `Chloros:Band{R,G,B}Name` v prípade snímky alebo produktu Chloros), takže NDVI na snímke odrazivosti Chloros nepotrebuje `channel_map`; dve pásma zodpovedajúce jednému symbolu sú odmietnuté, nie odhadnuté. `capture_level` udáva, čo pásma obsahujú: `reflectance` alebo `radiance` robia hodnoty kvantitatívnymi; nekalibrovaný súbor (`align-apply`) poskytuje kvalitatívne hodnoty; `"unknown"` to načíta zo súboru, vrátane súboru Chlorosu `Chloros:ProcessingLevel`. Vráti `stats` (tvar, rozsah 1–99 percentilu, rozsah LUT, použitý `capture_level`) a hodnotu `files` zapísanú vedľa vstupu (`<input stem>_<preset>.tif`, `..._LUT.png`) alebo na `output`, cesta: ak majú súbory obrazovú príponu, sú pomenované podľa kmeňa (`output="D:/out/ndvi_FRGN.png"` zapíše `ndvi_FRGN.tif` a `ndvi_FRGN_LUT.png`), bez nej ide o predponu (`D:/out/ndvi` -&gt; `ndvi.tif`, `ndvi_LUT.png`); prečítajte cesty od `files`. Obsahujú záznam o fotoaparáte zo vstupných údajov (značka / model / sériové číslo, GPS, objektív, čas záznamu v UTC, identifikačný kód kalibrácie, ale nikdy jej kalibráciu raw-DN), `Chloros:ProcessingLevel=index` a index ako ich jediný pás (`XMP-Camera:BandName`, TIFF a LUT PNG). Iba súbory; bez kamery. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Zaznamená, kalibruje (`reflectance` alebo `radiance`; `raw` to preskočí), zarovná a vypočíta index v jednom kroku; `save_multiband=True` uchová kalibrovanú sadu, ktorú môže neskôr prevziať `compute_index()`. Vráti `stats` (spolu s `serials`, ktorého pásma použil) a `files`. Otvorí `serials` (prvá spustí spúšť), inak kamery profilu, ktorých záznam `layer_order` je povolený, pričom ako prvá sa otvorí referenčná kamera (otvorená aj pri deaktivovaných pásmach); kamera s deaktivovanými pásmami sa nikdy nekalibruje. `channel_map` je potrebný len pre symbol, na ktorý zásobník nedokáže odpovedať, ako v `compute_index()`: symbol prednastavenia preberá jedno povolené pásmo, ktorého názov sa ním začína, čo sa skontroluje pred otvorením akejkoľvek kamery (ak žiadna alebo viacero neodpovedá, sú odmietnuté a sú pomenované); vlastný `formula` je potrebný pre každý symbol. `index.tif` a LUT obsahujú informácie o kamere zo zásobníka, čase snímania (expozícia referenčnej kamery, UTC) a objektíve, pričom obidva uvádzajú index ako svoje jediné pásmo (`XMP-Camera:BandName`, napr. `NDVI`: farebný súbor PNG uvádza index, nie svoje farebné kanály). `lattice index --live`. |

### `CliJob`

Identifikátor, ktorý vracajú tri prúdy. Úloha beží na backende, ktorý má kamery pod kontrolou, kým nie je zastavená.

| Člen | Popis |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (uložené snímky / zapísané, vynechané, `fps`, `elapsed`, chyby zápisu, `stop_reason`; v prípade nepretržitej úlohy aj `frames_grabbed` a `frames_queued` – každý zachytený snímok je zapísaný, zaradený do fronty, vynechaný alebo sa jeho zápis nepodaril – a `drop_reason`, `writers` a `write_ms`, priemerný čas potrebný na zápis jedného snímku) a `output_dir`. `state` je `running`, `ended` (sám sa zastavil a ešte nebol zastavený), `stopping` (zastavenie zapisuje svoju frontu: aktuálne štatistiky a `stopped_by`) alebo `finished` (jeho konečné štatistiky, `error` a `stopped_by` zastavenia). Záznam o zastavenej úlohe sa uchováva 10 minút (najviac posledných 32), pre každého klienta: iný skript alebo `chloros-mcp` číta ten istý stav a konečné štatistiky. Úloha `chloros-cli` spustená (`lattice continuous`, `multi-stream`, `align-stream`) je zastavená backendom, keď CLI prestane odosielať správy po dobu 30 s — bola ukončená, jej stroj je v režime spánku — s `stop_reason`, `client_gone` a `stopped_by` na strane backendu; čítanie jeho štatistík z tohto miesta ho neudrží v chode. Úloha spustená z SDK beží, kým nie je zastavená. Každé spustenie odošle svoj kód `timeout` ako termín backendu (`client_deadline_s`): kamera, ktorá sa otvorí po ňom (tá sa ešte spúšťa), sa vráti späť a zatvorí a nespustí sa žiadna úloha, pričom volanie vyvolá hodnotu `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). Kamera, ktorú drží úloha, je odmietnutá pri každom ďalšom otvorení (`connect_camera()`, ďalšie spustenie, prechodné zachytenie) s chybou HTTP 409 `CAMERA_HELD`, v ktorej je uvedená úloha a osoba, ktorá ju spustila. |
| `stop(timeout=30.0)` | Zastavte ju a zatvorte jej kamery: `final_stats`, `output_dir` a `error`, ak sa zastavenie neuskutočnilo bezchybne. Druhé volanie vráti rovnakú odpoveď, rovnako ako zastavenie z iného klienta: ak sa pošle, kým prvý ešte zapisuje svoju frontu, čaká naň (až 25 s; po uplynutí tejto doby vyvolá výnimku a konečné štatistiky nasledujú na `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Počkajte, kým sa úloha ukončí sama (dosiahnutie `count`, zaplnený disk, chyba), potom spustite `stop()`, aby ste získali konečné počty. `timeout` vyvolá výnimku `TimeoutError`, kým úloha stále beží; úloha bez `count` sa ukončí až po zastavení. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | Jeho identifikátor a typ (`continuous`, `multi-stream`, `align-stream`), miesto, kam zapisuje, odpoveď počiatočnej trasy a informáciu, či sa spustil `stop()`. |

Použite ho ako správcu kontextu, aby sa zastavil pri opustení rozsahu.

---

## Projektová zložka a recepty

**Projektová zložka** je zložka, v ktorej sa vytvárajú nové projekty, a ktorú spoločne využívajú aplikácia Chloros, CLI a SDK (modul `chloros_sdk.system_utils`; `chloros-cli project-folder`):

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Ak konfigurovaný priečinok už neexistuje, systém sa vráti k predvolenému (`Documents/MAPIR/Chloros Projects`), rovnako ako aplikácia. Relatívna cesta zadaná pre `set_project_folder()` sa berie z vášho pracovného adresára, ak je backend na tomto počítači.

**Recept** je zoznam akcií vo formáte YAML / JSON, ktorý `chloros-cli project run PATH RECIPE` spúšťa na uloženom projekte. `run_project_recipe()` ho spúšťa pomocou vlastného spúšťača CLIvlastným spúšťačom (nainštalovaným spolu s SDK), takže jeden recept sa správa rovnako v oboch prípadoch:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** je súbor `.yaml` / `.yml` / `.json`, alebo samotný recept: slovník s `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`) a voliteľne `overrides` a `stop_on_error` (predvolená hodnota true: spustenie sa zastaví pri prvej akcii, ktorá zlyhá). Akcie sú definované v referencii CLI [Recipe DSL](cli-reference.md#recipe-dsl); relatívny odkaz `output` sa vyrieši voči pracovnému adresáru vášho skriptu.
- **Vrátené hodnoty** `ok` (všetky akcie boli úspešné), `actions` (koľko), `log` (výstup spúšťača; `echo=True` ho tiež vypíše počas behu) a `validation`. Chýbajúci alebo neparsovateľný súbor receptu vyvolá chybu `ChlorosConnectError`.
- **Skontrolované pred spustením.**`validation` je `{"ok", "errors", "warnings", "steps"}`: každá akcia musí mať ako kľúč presne jedno známe sloveso (`{"capture": {"name": ...}}`, nie `{"action": "capture"}` alebo `{"type": "capture"}`), ktorá označuje kameru, pole alebo senzor, ktoré projekt obsahuje, a to správneho typu. Recept s chybou nevykoná**žiadnu** akciu (`ok` false, chyby v `log`), bez ohľadu na to, čo hovorí `stop_on_error`. Nepoužité kľúče, a každé pole alebo svetelný senzor, ktorý recept ovláda a ktorý nie je momentálne pripojený, sú `warnings`. Recept nepripája zariadenia: pripojte prvé zariadenie projektu (`ChlorosProject.connect_all()`, `chloros-cli project connect`, alebo nástroj MCP `connect_project_devices`). `dry_run=True` nespúšťa nič a vráti `{"recipe": ..., "validation": ...}`.
- V procese beží vždy len jeden recept. Recept v YAML vyžaduje PyYAML.

---

## Automatizácia projektu — `ChlorosProject`

Uložený projekt Chloros je priečinok obsahujúci súbory `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` načíta manifest (`relink`: zložka, v ktorej sa teraz nachádzajú chýbajúce zdrojové súbory presunutého projektu, používa sa pri načítaní backendu — `process()` — pozri **Presunutý projekt** pod `ChlorosLocal`), a `connect_all` pripojí online všetky uložené kamery, polia a svetelné senzory s ich uloženými nastaveniami — v rovnakom stave hardvéru, aký by vytvorilo grafické rozhranie. Svetelné senzory sa pripájajú cez [pool senzorov DAQ](#daq-sensor-sessions), preto potrebujú spustený backend (pozri [`SensorHandle`](#sensorhandle)).

### Minimálny príklad

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Alebo ako správca kontextu:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Metódy `ChlorosProject`

| Metóda | Popis |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Zistí a pripojí všetky uložené kamery, polia a svetelné senzory. Vráti správu o pripojení pre každú triedu (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, z ktorých každá obsahuje `connected`, `missing` a `errors`; `arrays` obsahuje aj `alignment`). Ak backend načúva na `127.0.0.1:5000`, uložené kamery a polia sa nájdu a pripoja cez neho. Ak žiadny nie je, tento proces ich nájde a ovláda sám cez `lattice_sdk`, ktorý vyžaduje runtime prostredie SDK pre Arena a OpenCV (`cv2`) — balík SDK neinštaluje ani jedno z nich; ak to nie je možné, každá uložená kamera a pole sa zobrazí v zozname pod `errors` spolu s dôvodom a `missing` zostane prázdny. Backend, ktorý beží, ale odmieta SDK (odhlásený: HTTP 401; plán pod Copper: 403), nie je „žiadny backend“: každá uložená kamera, pole a svetelný senzor sú uvedené pod `errors` spolu s dôvodom backendu, a v tomto procese nie je nič otvorené. Každé volanie opäť skontroluje backend, takže `connect_all()` po prihlásení alebo po spustení Chloros sa pripojí k rovnakému objektu projektu. Uložené pole, ktoré je už spustené v backende s rovnakými kamerami (z grafického rozhrania alebo z `chloros-cli project connect`), sa pripojí tak, ako je, bez opätovného uplatnenia jeho uložených nastavení, a `disconnect_all()` ho nechá bežať. Svetelné senzory vždy prechádzajú cez backend (pozri [`SensorHandle`](#sensorhandle)). Nikdy nespustí backend. |
| `disconnect_all()` | Všetko ukončiť. Najprv sa uvoľní každé zariadenie; potom, ak backend nepotvrdil uvoľnenie kamery, poľa alebo svetelného senzora (časový limit ukončenia vypršal alebo bolo odmietnuté), `ChlorosConnectError` označí každé z nich. Zariadenie, ktoré už neexistuje, alebo ak nebeží žiadny backend, je tiché. Pole alebo svetelný senzor, ktoré mal otvorené iný klient, zostávajú v prevádzke. Ukončenie projektu `with` sa nikdy nespustí. |
| `capture_all(output_dir=".")` | Jeden snímok z každej pripojenej kamery + poľa, a jedno spektrum (`SensorHandle.read()`) z každého pripojeného svetelného senzora, snímané postupne (najprv kamery, potom polia, potom senzory), nie naraz. |
| `stream(camera, overlays=False, fps=10.0)` | Generátor produkujúci snímky BGR `numpy` z určenej kamery (alebo matice). `overlays=False` číta snímky s identifikátorom`grab()` na úrovni `debayered`: cez backend, ak je projekt pripojený cez backend, inak cez priamu `lattice_sdk` slučku snímok (pole generujú `{serial: frame}` slovníky). `overlays=True` smeruje cez `ChlorosLocal.camera_stream()` → MJPEG kanál `/api/camera/<serial>/stream-annotated` backendu, pričom uložený blok kamery `ui.overlay` sa prenáša ako parametre dotazu. Vyžaduje režim backendu, OpenCV (`cv2`) a **samostatnú kameru**: kamera v priamom režime vyvolá chybu `RuntimeError` (backend nemôže získať kameru, ktorú vlastní tento proces) a pole vyvolá chybu `NotImplementedError` (prekrýva kompozitný obraz za každú kameru — streamuje prvok podľa mena). Ekvivalent jednorazového spustenia: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Spustiť zarovnanie na každom aktuálne pripojenom poli. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Spustiť kalibračný/indexovací proces na obrázkoch projektu (zahŕňa `ChlorosLocal.process`; tieto štyri sú **jediné** akceptované kľúčové argumenty — `indices=` atď. vyvolá výnimku `TypeError`; nastaví indexy prostredníctvom `ChlorosLocal.configure()`). Odložene vytvorí `ChlorosLocal()`, ktorý automaticky spustí backend. |

Atribúty:
- `proj.cameras` — `Dict[str, CameraHandle]` s kľúčom podľa názvu A sériového čísla.
- `proj.arrays` — `Dict[str, ArrayHandle]` s kľúčom podľa názvu A array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` s kľúčom podľa názvu A slot_id.
- `proj.config` — slovník `project.json["config"]`.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Úrovne spracovania.** `capture()`, `grab()` a `frame_stream()` všetky používajú ten istý token `processing`
a reťazec je kumulatívny — každá úroveň spúšťa všetko, čo je nad ňou:

| Úroveň | Výstup | Poznámky |
| --- | --- | --- |
| `raw` | 1-kanálový Bayer, natívny pre senzor | Bez demosaicovania. Na tejto úrovni nie sú k dispozícii prekrývacie efekty. |
| `debayered` | 3-kanálová farba (**predvolené** pre `CameraHandle` a všetky `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` a `stream()` majú predvolené nastavenie na `raw`): BGR v pamäti (`grab()` / `frame_stream()`), uložené ako RGB pomocou `capture()` | V režime backendu, ako snímka: tmavý posun (DSNU) z kalibračného balíka z surových údajov, potom demosaik s ohľadom na hrany pomocou OpenCV — bez LUT linearity, vyrovnania poľa ani oddelenia farieb. Jediná úroveň, ktorá funguje bez režimu backendu, kde ide o jednoduchú bilineárnu demosaiku knižnice kamery (bez tmavého posunu). Uložený súbor TIFF je vo fotometrickom RGB formáte, preto je potrebné obrátiť kanály snímky v pamäti (`frame[..., ::-1]`), aby ste ho mohli porovnať so súborom. |
| `radiance` | float32, W/m²/sr/nm, **jedna rovina na pásmo** | Úplný radiometrický reťazec: demosaik + 3×3 oddelenie kanálov (multispektrálne) + DSNU + kalibrácia plochého poľa + stupnica NIST, pričom expozícia × zosilnenie sú vydelené, takže hodnoty sú absolútne. |
| `reflectance` | uint16, 32768 = 1,0 (alebo uint8, 255 = 1,0 — pozri `bit_depth`), **jedna rovina na pásmo** | Žiarivosť vydelená dopadajúcou intenzitou žiarenia (ρ = π·L/E). Vyžaduje sa hodnota z DLS/DAQ — pozri poznámku nižšie. |
| `display` | 8-bitový formát podobný sRGB | Vykreslenie zodpovedajúce grafickému rozhraniu: CCM + vyváženie bielej + gama prostredníctvom aktívneho farebného profilu kamery. |

V priamom (bez backendu) režime je k dispozícii iba `debayered`: `grab()` na inej úrovni ignoruje
`NotImplementedError`, `capture()` ignoruje `processing` a `frame_stream()` vyvoláva
`RuntimeError` na každej úrovni. `reflectance` potrebuje použiteľnú hodnotu smerujúcu nadol — koncový bod rámca automaticky vtiahne
zhrnuté DAQ do slotu DLS kamery, ale ak nie je viazané žiadne DAQ, reťaz odmietne
výstup odrazivosti a úprimne označí zníženie v vrátených metadátach namiesto toho, aby ticho
vrátiť produkt nižšej kvality.

> **Škála odrazivosti DN — neprogramujte ju pevne.** Odrazivosť LATTICE používa `32768` = ρ 1,0 a označuje
> XMP `Chloros:PixelScale=32768`; odrazivosť Survey3 používa `65535` = ρ 1,0 a neobsahuje žiadne
> značky `Chloros:*`. Prečítajte značku a vydelite ňou. Je definovaná v doméne uint16, takže zostáva
> `32768` pre každý formát, ktorý mení mierku (16-bitový TIFF, 8-bitový TIFF, 8-bitový PNG/JPG, 32-bitové percento)
> — najprv normalizujte uložený dátový typ späť na uint16 (×257 z 8-bitového, ×65535 z float). Jedinou
> výnimkou je súbor z verzie staršej ako 1.3.0: 8-bitový TIFF bol vtedy *orezaný*, nie preškálovaný, takže
> ho nepopisuje žiadne mierka a neobsahuje žiadnu dvojicu `PixelScale` ani MicaSense. Chýbajúcu značku v
> súbore odrazivosti LATTICE považujte za „žiadnu platnú mierku“, nie za predvolenú hodnotu.

**Kalibračné tagy: iba v súbore `raw`.** Súbor `raw` zo záznamu obsahuje kalibračný paket kamery: užitočné dáta `CalibrationJSON` a značky kompatibilné s MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), ktoré opisujú surové hodnoty senzora. Každý súbor na ďalšej úrovni (`debayered`, `display`, `radiance`, `reflectance`, render `index/`) a každý snímok z nepretržitého záznamu alebo viacerých kamerových prúdov obsahuje identifikátor kamery (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) a vlastný záznam snímky, `XMP-mapir:FrameJSON` (expozícia, zosilnenie, teplota, formát pixelov, čas snímania, zhoda DAQ), keďže každý produkt typu `process()` nesie túto identitu. Spracovanie vyhľadá kalibráciu takéhoto súboru podľa jeho sériového čísla a `CalBundleSha256` v kalibračnej cache programu Chloros a v prípade, že tento počítač kalibráciu nemá, balík sa stiahne raz.

#### Súbory vytvorené z viacerých kamier

Súbor vytvorený z pásiem viacerých kamier — viacpásmový stack (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), živý index vytvorený z viacerých kamier (`index.tif`, `index_LUT.png`), obraz `composite/` zo záznamu z viacerých kamier — uvádza názvy všetkých kamier, z ktorých bol vytvorený:

- **Vkamery, v poradí podľa pásiem** (kompozit: vrstva pozadia, potom popredie; indexová vrstva, kamery, z ktorých pochádzajú jej pásma): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (kalibračný balík, ktorým prešli pásma každej kamery, inak — zoskupenie `capture_aligned_multiband()` / `start_aligned_stream()`, ktorého pásma neprešli žiadnym — balík kamery, ktorý si Chloros uložil do vyrovnávacej pamäte, ten, podľa ktorého sú pomenované jej súbory so záznamami), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — paralelné zoznamy, pričom položka *n* v každom z nich popisuje tú istú kameru; `-`, kde kamera nemá žiadnu hodnotu (chýba kalibračný balík, neznámy objektív). Každá kamera je **jedna** položka: ak má jej balík viacero hodnôt pre jedno pole — certifikáty halogénových žiaroviek balíka — sú spojené pomocou `;` (`F-1479-1;F-1480-1;F-1481-1`), nikdy čiarkou, takže zoznam obsahuje jednu položku na fotoaparát bez ohľadu na to, ako je vytlačený.
- **EXIF `SerialNumber`** obsahuje rovnaké sériové čísla, oddelené čiarkami (`214000535,214000551`).
- Neobsahuje **žiadnu** identifikáciu jednej kamery (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): tie označujú jednu kameru a jej sadu, a spracovanie by z nich určilo kalibráciu súboru. Index vypočítaný z pásiem viacerých kamier (`index.tif`, `index_LUT.png`) má EXIF `Model` `LATTICE` (súprava), nie model `LATT-…` jedného fotoaparátu.

Súbor pásiem **jednej** kamery — TIFF pre každý pás, snímka `capture_aligned_multiband(per_camera=True)`, zoskupenie alebo živý index jednej kamery — obsahuje identifikačné značky tejto kamery, rovnako ako každý iný súbor so záznamom. Séria snímok `capture_aligned_multiband()`, súbory pre jednotlivé pásma a pre jednotlivé kamery, ako aj každá séria snímok `start_aligned_stream()` a jej súbory pre jednotlivé pásma obsahujú čas záznamu svojej skupiny (expozícia referenčnej kamery, UTC) a snímka `per_camera` z monochromatickej kamery je v jednej rovine, v `bit_depth` rovnako ako súbor snímok.**Orientácia kamery (otočenie / prevrátenie).** Snímky z `grab()` / `frame_stream()` sú vždy v
pôvodnej orientácii kamery na každej úrovni. Uložené zábery sa riadia nastavením orientácie kamery
(možnosti GUI Zrkadliť H / Zrkadliť V / otočenie):

- **Súbory `raw` a `debayered` presne zodpovedajú údajom zo snímača**, bez ohľadu na nastavenie — otočená alebo zrkadlovo otočená Bayerova
  mozaika by predstavovala odlišný vzor farebných filtrov a kalibračné mapy sú
  na vlastnej mriežke snímača.
- **Súbory `display`, `radiance`, `reflectance` a indexové súbory sú orientované** — otočené o jeden stupeň po vykonaní všetkých
  kalibračných výpočtov. Ich masky obmedzenia sú na tej istej mriežke.
- **Každý súbor záznamu zaznamenáva nastavenie** v XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, v smere hodinových ručičiek, po
  otočenia) a `ChlorosOrientationApplied` — `false` na surových a debayeroovaných, `true` na orientovaných
  výstupoch. `ChlorosProcessing` uvádza zoznam operácií aplikovaných na pixely daného súboru (`none` v prípade, ak je
  súbor v natívnom formáte). To zahŕňa každý snímok nepretržitého záznamu (vlastné nastavenie kamery,
  neaplikované) a viackamerového prúdu (identita: jeho snímky sú v natívnej podobe, podobne ako pri
  zachytávaní z matice).
- **Spracovanie projektu aplikuje zaznamenanú orientáciu** na produkty, ktoré vytvára z surových
  (najskôr kalibrácia v súradniciach snímača, potom deformácia zarovnania poľa a nakoniec orientácia),
  takže zodpovedajú výstupom zo živého snímania — vrátane nekalibrovaných výstupov s korekciou odozvy snímača a vinetácie;
  výstup po odstránení bayera zostáva v natívnom formáte.
- **Prehliadač obrázkov Chloros zobrazuje surový alebo debayered súbor otočený podľa jeho záznamu** — iba na zobrazenie,
  súbor sa nemení; súbor bez záznamu sa zobrazuje tak, ako je uložený.
- **Záznamy bez záznamu sa spracúvajú ako natívne.** Pred verziou 1.3.0 záznam z jednej kamery
  sám prevádzal svoje súbory vo formáte RAW a po odstránení bayera; Chloros nedokáže rozlíšiť takýto súbor vo formáte RAW od natívneho a
  neuskutočňuje žiadne odhady.
- **Záznamy z poľa zachovávajú každý súbor v poľu štandardne v natívnej podobe** a zaznamenávajú identitu;
  `ArraySession.capture(orient=True)` spracováva ich výstupy (pozri ovládacie prvky snímania poľa).

#### Výstup podľa pásma a zmiešané polia

Na úrovniach `radiance` a `reflectance` kamera vráti **jednu rovinu na každé pásmo danej
kamery** — tri pre M3C (jej tri nemiešané pásma, poradie kanálov BGR), jednu pre M3M (jej
jediný úzkopásmový filter). Replikované roviny mono kamery sú zlučené, takže **zmiešaná
pole M3C/M3M poskytuje rôzne tvary polí pre každú sériu**. Prečítajte si kľúč `bands` z metadát snímky
, aby ste zistili, čo je každá rovina; nevyvodzujte to z počtu kanálov:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Pred použitím hodnôt vždy skontrolujte `hdr["processing_level"] == "reflectance"`. Zníženie úrovne
(žiadne DAQ čítanie, žiadny kalibračný balík, neznáme rozloženie pásiem) sa hlási v `processing_level` a
`radiometric_refused` namiesto zvýšenia úrovne — pole, ktoré dostanete späť, je stále platný rámec, len
nie je to produkt, o ktorý ste požiadali. Uložený záznam, ktorý bol degradovaný, nesie v súbore XMP rovnaký kód
ako `Chloros:RadiometricRefused` (napr. `no-band-layout`, `no-dls-snapshot`).

#### Jeden spúšťací impulz na jedno čítanie — `group_id`

**Členovia poľa sú obsluhovaní zo synchronizovanej skupiny snímok poľa, nie z vlastného
záznamového vyrovnávacieho pamäťového priestoru každej kamery.** Kamery hardvérovo synchronizovaného poľa exponujú spoločne, ale N čítaní na kameru
sa *neodoberajú* spoločne — čítanie najnovšieho vyrovnávacieho snímku každej kamery prekračuje hranice spúšťača
a jediný vynechaný snímok desynchronizuje súbor bez toho, aby si to niekto všimol.

Poskytovanie zo skupiny je nevyhnutné, ale samo o sebe nestačí: každá požiadavka stále
zaznamenáva *najnovšiu* skupinu vo svojom vlastnom okamihu a cyklus snímkovania môže medzi tým zverejňovať nové snímky.
Pri meraní na živom poli so 4 kamerami **12 z 20 paralelných čítaní prekračovalo dve skupiny** — vždy
presne susediace. Čítanie je teda **fixované**:

1. Každý snímok nesie `group_id`, monotónny počítadlo publikovania sústavy.
2. `grab()` prečíta hlavnú kameru (alebo, ak hlavná kamera nemôže obslúžiť žiadosť, ďalšiu kameru, ktorá to dokáže),
   potom priradí zostávajúce kamery k identifikátoru skupiny, na ktorom sa zastavilo (`?group_id=N` na HTTP
   koncovom bode).
3. Backend uchováva nedávne skupiny **aspoň pol sekundy**, takže súrodenec, ktorý už
   bol predbehnutý neskoršími publikáciami, stále obsluhuje tento spúšťač — dostatočne dlho na to, aby sa
   odrazovosti pre každú kameru prebehnúť aj v prípade plného senzorového poľa s 9 kamerami. Priradenie, ktoré backend už
   neudržiava, je odmietnuté kódmi `409` a `group_aged_out: true` namiesto tichej odpovede
   s iným spúšťačom — `grab()` znovu prečíta nastavenie.

Opätovné meranie s pripínaním: **20 z 20 čítaní vrátilo jeden spúšťač.** `grab()` to urobí za vás;
HTTP-koncový bod potrebujete ovládať priamo len pomocou `group_id=`.

- Kamera, ktorá nemôže byť obslúžená v pripnutej skupine — chýba v nej alebo odmieta žiadosť
  (`band`, ktorý nemá, úroveň, ktorú nedokáže vygenerovať) — je **vynechaná namiesto toho, aby bola doplnená**
  z iného spúšťača a ostatné kamery sa naďalej vracajú.
- Slovník, ktorý vráti `grab()`, je `GrabResult` (obyčajná podtrieda `dict`;
  `from chloros_sdk import GrabResult`): `.errors` priraďuje každé
  vynechané sériové číslo k výnimke, ktorú vyvolalo jeho čítanie, a `.group_id` je zdieľaná skupina. Prázdny
  `.errors` znamená, že túto skupinu tvorili všetky kamery.
- Samostatná kamera nemá žiadnu skupinu, hlási `group_id: None` a **odmieta** `group_id=`
  pin namiesto toho, aby ho ignorovala.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Ten istý počítadlo je zverejnené v **každom MJPEG prúde pre každú kameru** člena poľa — v
náhľade (`/api/camera/stream/<serial>`), v anotovanom náhľade (`/stream-annotated`) a v
prúd odrazivosti (`/stream-reflectance`) — ako `X-Chloros-Group-Id` v každej časti, okrem
`X-Chloros-Frame-Id` a `X-Chloros-Timestamp-Ns`. Prúdy sú kontinuálne, nie
bodové, takže sa zosúlaďujú zhromažďovaním častí a zoskupovaním na základe tejto hlavičky — nie je potrebné
nastavovať žiadny pin. Samostatná kamera nemá žiadnu skupinu. Pozri
[MJPEG prúdy na kameru](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` nikdy nezmlkne.** Tick, ktorý zlyhá z dôvodu, ktorý sa môže vyriešiť sám od seba,
sa preskočí — prúd z kamery preň nevráti nič, prúd z poľa vráti kamery, ktoré
odpovedali (možno žiadne). Tieto dôvody sú: zatiaľ žiadny snímok; vlákno na zachytávanie kamery sa pozastavilo na
sekundu alebo dve (`404 not streaming`, na čo backend odpovedá vždy, keď sa pripojí
alebo odpojí ďalšia kamera, alebo keď zmena nastavenia vyváži fond); reštart backendu alebo `5xx`; vyrovnávanie
sa ešte ustálilo (`409 … retry shortly`); a odmietnutie odrazivosti, pretože DAQ ešte nemá žiadnu hodnotu
(`no-dls-snapshot`, neaktuálna alebo prázdna hodnota). Odmietnutie v nasledujúcom takte by sa len opakovalo —
neznámy `band`, `bit_depth=8` pri radiancii, kalibračný balík bez stupnice radiancie, odmietnutie odrazivosti
s **žiadnym DAQ vôbec** (žiadny priradený k kamere alebo jej matici, žiadny pripojený k
backendu: `dls_source` odmietnutia je `"none"`), akýkoľvek iný `4xx` —
vyvolá `chloros_sdk.FrameFetchError` (`RuntimeError`, nesúci `.status` a `.body`) naraz.
To isté platí pre **60 sekúnd** preskočených tikov bez medziležiaceho rámca, bez ohľadu na to, čo je `fps` — dosť
dlho na to, aby sa prečkalo ustálenie zarovnania a reštart backendu, a dosť krátko na to, aby kamera, ktorá je
skutočne nefunguje, ukončí prenos. Táto chyba vzdania sa je tiež `FrameFetchError`, a to tak pre kameru, ako aj pre
pole, pričom výnimka posledného čítania je `.errors` (a v prípade kamery ako `__cause__`). V prípade
poľa „odmietnuté“ znamená, že každá kamera odmietla; výnimka `.errors` obsahuje dôvod
každej kamery.

**Pole `frame_stream()` generuje každý spúšťač raz** (rovnako ako `ArrayHandle` a `ArraySession`).
Tik, ktorého najnovšia sada je spúšťačom, ktorý už vrátil, čaká na ďalší spúšťač namiesto toho, aby
opäť vrátil tú istú sadu, takže prúd beží pomalšie ako `fps` alebo vlastná frekvencia sady
, a po sebe idúce sady nikdy nezdieľajú `group_id`. Pole, ktoré prestane publikovať, vyvolá
`FrameFetchError` („žiadny nový spúšťač počas 60 s“) namiesto opakovania svojej poslednej sady. Bez toho by
prúd, ktorý požiadal o viac, než pole publikovalo — alebo začal hneď po pripojení — vydával
opakovania: v jednom meranom behu obsahovalo 20 sád 11 rôznych identifikátorov skupín.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Exportovanie jedného pásma — `band`

`grab()` / `frame_stream()` použite `band="<name>"` na export len jedného z pásiem kamery ako
jednokanálové pole `(H, W, 1)` — s rovnakým tvarom, aký produkuje monochromatická kamera, takže výstup jedného pásma sa
spracováva rovnako, či už pochádza z M3M alebo z rozdelenia M3C. Červené pásmo FRGN:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Prijateľné zápisy sú buď úplný názov rozloženia (`Red_660`, `NIR_850`, `Green_550`), alebo jeho skrátená
hlavička (`red`, `nir`, `green`), bez ohľadu na veľkosť písmen. Neznámy názov je odmietnutý a sú zobrazené dostupné
názvy.

> **Výber sa vykonáva podľa názvu, nie podľa čísla kanála — zámerne.** „Kanál 1“ je `Green_550` v
> poradí kanálov BGR poľa, ale *červený* v kóde filtra(`RGN`), a tieto dva
> sa nezhodujú. Výber nesprávneho kanála vedie k získaniu úplne vierohodného rámca nesprávneho pásma, ktorý
> žiadny následný systém nedokáže detekovať. Z tohto dôvodu neexistuje číselný index.

#### Zarovnanie tokov — `align`

`align=True` transformuje snímku do spoločného referenčného rámca svojho poľa. Každá kamera je transformovaná
prostredníctvom **rovnakého** profilu rozlíšenia a výrezu ROI, takže prúdy jednotlivých kamier otvorené nezávisle
sú **vzájomne zaregistrované** — nemusíte ich žiadať spoločne:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Požiadavky a správanie:

- Kamera musí byť v **hardvérovo synchronizovanom** poli (synchronizačný kábel M8), ktoré má profil zarovnania
  — spoločná registrácia predpokladá, že členovia sú exponovaní súčasne, čo softvérové alebo
  voľne bežiace pole nezaručuje. V opačnom prípade bude žiadosť zamietnutá s kódom `409`.
- **Odmietnuté, nikdy sa ticho nekalibruje.** Cesta zachytenia môže uložiť nekalibrované údaje s varovaním,
  pretože súbor si zachováva svoje identifikačné značky a môže byť neskôr znovu zaregistrovaný; živý prenos nemá
  takúto druhú šancu, takže chýbajúci profil je skôr chybou ako náhradným riešením.
- **Žiadosť o zarovnanie ho spustí.** Žiadosť o `align` na poli bez profilu spustí
  kalibráciu a vráti `409` s informáciou, na čo čaká (napr. *„čaká na ustálenie expozície
  na 214000551 (1/3)“*) — pri opakovanom pokuse sa to podarí, zvyčajne v priebehu niekoľkých desiatok sekúnd.
  Toto je dôležité pri bezhlavom režime: automatické zarovnanie pri pripájaní sa vzdá po 30 s a každý pokus
  v tomto časovom okne sa preskočí, kým sa inteligentná automatická expozícia (smart-AE) ešte ustálila, takže pole, ktorého expozícia
  sa konverguje pomaly, skončilo bez profilu pre celú reláciu a bez možnosti získať
  ho bez otvorenia kombinovaného zobrazenia na pracovnej ploche. `alignment_status` tiež hlási
  `ae_align_holdouts`, pričom uvádza názvy kamier, ktoré sa ešte stabilizujú, a ich pokrok.
- Každému prúdu jedného poľa priraďte **rovnaké `max_h`**, inak sa ich výrezy ocitnú na rôznych pixelových
  mriežkach.
- Zarovnanie sa aplikuje *pred* kódovaním bitovej hĺbky, takže deformácia interpoluje hodnoty s plnou presnosťou
  .

#### `bit_depth` — výmena presnosti za šírku pásma

`grab()` a `frame_stream()` akceptujú `bit_depth=16` (predvolené) alebo `bit_depth=8`:

| `bit_depth` | Kódovanie odrazivosti | Náklady |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, rezerva do ρ 2,0 | Plná presnosť. |
| `8` | uint8 zahŕňajúci `0..refl_max` (predvolené ρ 1,0) | Polovičný počet bajtov; kvantizácia ~0,004 ρ pri predvolenom hornej hranici. Hodnoty nad touto hranicou sa orezávajú — pozri nižšie. |

##### 8-bitová horná hranica — `refl_max`

`refl_max` nastavuje hodnotu ρ, ktorú pokrýva 8-bitový rozsah, takže pri `pixel_scale = 255 / refl_max` a ρ je stále
`pixel / pixel_scale`. `grab()` a `frame_stream()` to berú ako kľúčové slovo na oboch kamerových aj
v rukovätiach poľa (`?refl_max=` na HTTP-koncovom bode). Zvýšte túto hodnotu, aby ste získali rezervu pre odlesky,
a to v pomerne hrubších krokoch:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | krok |
| --- | --- | --- |
| `1.0` (predvolené) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **Automatická expozícia nedokáže napraviť 8-bitové preexponovanie a jej použitie na tento účel ešte zhorší kvalitu údajov.**
> ρ = π·L/E po vydelení expozície a zosilnenia, takže ρ je vlastnosťou scény, nie
> nastavenia fotoaparátu — merané na živom poli, zníženie expozície o polovicu posunulo priemernú hodnotu ρ o 1,3 %. Zvýšenie
> expozície s cieľom „prispôsobiť“ rozsah naopak vedie k saturácii pixelov **snímača**, čo
> podhodnocuje L a posúva ρ **do nižších hodnôt** bez vygenerovania chyby (merané: max. ρ 1,203 → 0,985 pri
> 2× expozícii, priemerný pokles −23 %). Rozsah vyberte pomocou `refl_max`; udržanie snímača mimo
> nasýtenia je úlohou automatického nastavenia expozície (AE), čo chráni *platnosť* hodnoty ρ skôr než jej rozsah.

**Prečítajte si `reflectance_clipped_fraction`, nie vlajku.** `reflectance_clipped_at_1_0` sa spustí už pri
jedinom pixeli — v reálnej scéne hlásilo `True`, zatiaľ čo podiel bol `0.000001`, jeden pixel
z milióna. Práve tento zlomok vám napovie, či máte zvýšiť hornú hranicu.

**`bit_depth=8` sa týka iba odrazivosti.** Žiadosť o `radiance` je zamietnutá, nie
vybavená: odrazivosť má pevný rozsah 0..1, ktorý pretrváva aj pri 8-bitovom kódovaní, zatiaľ čo žiarivosť nemá
pevný rozsah, takže jej vtesnanie do 8 bitov vyžaduje percentilové roztiahnutie pre každý snímok, pričom toto mapovanie
sa mení v každom snímku — ide o obrázok, nie o meranie.

#### Živý prúd odrazivosti (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` poskytuje 8-bitovú odrazivosť na pásmo ako
`multipart/x-mixed-replace` — M3C ako 3-kanálový JPEG, M3M ako stupnice šedej. Parametre dotazu:
`fps` (predvolené 10), `jpeg_q` (predvolené 95), `max_h` (0 = natívne), `band` (jedno pomenované pásmo, ako
vyššie), `align` (`1` na spoluregistráciu — rovnaké pravidlá ako v prípade `align` v SDK), `refl_max` (8-bitový
kódovací strop, ako vyššie). Každá časť obsahuje
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` a `X-Chloros-Reflectance-Clipped`.
**Časti z rôznych kamier, ktoré nesú rovnaký `X-Chloros-Group-Id`, predstavujú ten istý spúšťací signál** —
takto sa znova zoskupujú viaceré prúdy z jednej kamery v rámci jedného poľa, čo je zaručené
na strane servera, pretože členovia poľa sú poskytovaní zo synchronizovanej skupiny snímok (pozri
[Jeden spúšťač na jedno čítanie](#one-trigger-per-read--group_id)). Vráti `409`, ak je odrazivosť
v čase pripojenia nedostupná, a ukončí prúd, ak sa stane nedostupnou uprostred behu, takže zastavený
prúd znamená „už žiadna odrazivosť“ skôr než „stále v poriadku, len statický“.

> **Formát JPEG je stratový — tieto pixely predstavujú meranie.** Chyba kompresie sa priamo prejaví v
> hodnotách odrazivosti a je najhoršia na okrajoch, kde sa nachádzajú hranice materiálov. Z tohto dôvodu je
> predvolená kvalita nastavená na 95. Ak potrebujete zachovať čísla v pôvodnej podobe, použite
> `grab(processing="reflectance")` / `frame_stream(...)`, ktoré vracajú rovnaký výsledok bez
> kodeku v ceste (a `bit_depth=8`, ak je obmedzením cena prenosu, nie kodek).

> **EXIF sa prenáša do exportu.** `process()` skopíruje blok GPS zo zdrojového záznamu
> **a jeho ExifIFD** do každého produktu, takže exporty obsahujú `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` a `CameraSerialNumber`, ako aj
> georeferenciu. Na základe kódu `FocalLength` Pix4D vypočíta vzdialenosť medzi bodmi na zemi – bez neho
> sa rekonštrukcia vráti k úplne nesprávnemu meradlu (v jednom nameranom prípade sa lokalita s rozlohou 411 m
> zmenila na lokalitu s rozlohou 47,8 km). Kópia zámerne nie je `-all:all`: štrukturálne tagy IFD0 narušujú
> výstup LATTICE a `ExifImageWidth`/`Height` sú vylúčené, pretože opisujú zdrojové
> snímanie, nie exportovaný raster.

Podvlajky fázy snímania (platia pre radiometrické úrovne — `radiance`, `reflectance`, `display`):

| Príznak | Predvolené | Význam |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + vyrovnanie poľa + 3x3 oddelenie kanálov + radiometrická stupnica NIST. |
| `apply_white_balance` | `True` | WB LUT. Zohľadňuje DLS, ak je k kamere pripojený DAQ. |
| `apply_index` | `False` | Vyhodnotenie vegetačného indexu. |
| `index_expression` | `None` | Prepis vzorca. Prázdneprázdne → automaticky aktivuje index. |
| `annotated` | `False` | Prekrytie dekorácií grafického rozhrania (zebra/mriežka/vrcholy). Nedostupné pre `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Vrátená hodnota je `chloros_sdk.CapturePaths`, typu `CapturePathMap`, nie `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` je `Dict[str, Union[str, List[str]]]`: jednovrstvový
> `processing` priraďuje každému sériovému číslu jednu cestu, zatiaľ čo `processing="all"` mu priraďuje
> **usporiadaný zoznam** všetkých produktov uložených pre danú kameru. `ArrayHandle.capture()` neprijíma
> žiadne `levels=` a ignoruje kľúčové argumenty, ktoré nepozná (`levels`, `daq`,
> `render_index`, `force_daq`), preto zadávajte len tie, ktoré sú tu uvedené (`aligned=False`
> ukladá natívne súbory, rovnako ako `ArraySession.capture()`); pre tieto ovládacie prvky použite
> `ArraySession.capture()`. Aktívny kombinovaný kompozit, ak bol streamovaný, prichádza pod dodatočným
> kľúča `"combined"`, nie pod sériovým číslom. Kód, ktorý predpokladá `str`, sa pokazí v
> zoznamovej forme bez toho, aby proti tomu namietal akýkoľvek typový kontrolór — v poznámke sa uvádzalo, že `Dict[str, str]`
> ešte nejaký čas po vydaní zoznamovej formy, čo je dôvodom existencie tohto aliasu. Normalizujte
> v prípade, že chcete plochú formu:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Čo sa nedalo napísať: `.failed`.** Mapa, ktorú vráti `capture()`, je
> `chloros_sdk.CapturePaths` — obyčajná podtrieda `dict`, takže všetko vyššie uvedené funguje
> bez zmien — ktorá obsahuje aj `.failed`: súbory, ktoré backend nedokázal zapísať,
> rovnaké záznamy ako [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)
> (`serial`, `level`, `error` a `disk_full: True` na plnom disku). Čiastočné
> zachytenie stále vráti všetky cesty, ktoré boli zapísané; na zaplnenom disku je mapa
> prázdna a súbor `.failed` uvádza dôvod. `.skipped` uvádza preskočené záznamy podľa kamery a
> `.group_seq` synchronizovanú skupinu, a `.alignment` / `.oriented` sú
> údaje backendu, rovnako ako v prípade `CaptureResult`. `capture_smart()` ich vráti ako
> `result["frames"].failed` a `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Zarovnanie poľa

`ArrayHandle` sprístupňuje úplnú zarovnávaciu plochu. Profily sú štandardne platné len pre danú reláciu — na ich trvalé uloženie explicitne volajte `export_alignment()`.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Zarovnanie pri pripojení

`connect_all(align=...)` dokáže automaticky zarovnať každé pole pri pripojení:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Ak nie je špecifikované, použije sa `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

Uložený svetelný senzor (DAQ-U, DAQ-M alebo DAQ-E), otvorený pomocou `connect_all()` v [pool senzorov DAQ](#daq-sensor-sessions) — ten istý pool, ktorý používajú `connect_daq_sensor()` a `chloros-cli daq pool-connect`.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` čaká na **ďalšie** spektrum senzora — to, ktoré je už v pamäti vyrovnávacej pri volaní, sa nepočíta — a vráti `(spectrum, is_saturated, integration_time, x, y, z)` alebo `None`, ak žiadne neprišlo do 5 s. Pozastavený tok sa obnoví. Hodnota `spectrum` je vyjadrená v W/m²/nm, ak je senzor kalibrovaný.
- Hodnota `.sensor` je [`DAQSensorSession`](#daq-sensor-sessions), pre všetko ostatné: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` a `.settings` sú uložené položky z `sensors.json`.

**Čo obnovuje `connect_all`.** Každý senzor sa otvorí na svojej uloženej adrese — na svojom COM porte, adrese Bluetooth alebo hostiteľovi DAQ-E — a ak projekt zaznamenal identifikátor senzora, musí ísť o tento senzor: iná jednotka na tejto adrese je odmietnutá, nie je pripojená. Použijú sa jeho uložený integračný čas, priemerovanie snímok a automatická expozícia, ako aj korekcia krytu:

| Uložený profil krytu | Pripojený s |
| --- | --- |
| Profil krytu (napr. `fov_30`) | Tento profil. |
| Žiadny nevybraný | `sunshine_cosine`, predvolené pri pripojení, zaznamenané podľa predpokladu (`cap_id_source=auto_default`), preto systém vydá varovanie. Senzory sa dodávajú bez krytu: vyberte kryt, ktorý je nasadený. |
| `none` — zvolený ako **Žiadny (senzor bez krytu)** v grafickom rozhraní | Žiadna korekcia krytu. |

Senzor DAQ-E-S neprijíma žiadny profil krytky, bez ohľadu na to, čo bolo uložené: jeho difúzor je súčasťou jeho kalibrácie (pozri [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Projekt uložený staršou verziou Chloros zaznamenal holý senzor ako jedinú možnosť, preto sa opäť pripojí k `sunshine_cosine`; v grafickom rozhraní znovu vyberte **Žiadne (holý senzor)** a projekt uložte.

Senzor, ktorý má iný klient už otvorený v skupine (CLI, iný skript), sa pripojí tak, ako je, bez opätovného uplatnenia nastavení, a `disconnect_all()` ho nechá bežať; senzor, ktorý otvoril `connect_all()`, sa uvoľní. Rovnako ako v prípade `connect_daq_sensor()`, grafické rozhranie Chloros **nie je** súčasťou skupiny: pred pripojením projektu zo skriptu zatvorte senzor v grafickom rozhraní.**Požiadavky a správa.** Svetelné senzory vyžadujú spustený backend Chloros s prihlásením na platenej úrovni Chloros+ (Copper alebo vyššej), a `connect_all()` ho nikdy nespustí. Ak žiadny nebeží, každý uložený senzor je uvedený pod `report["sensors"]["errors"]` s touto informáciou. Senzor, ktorý backend nedokáže otvoriť, je tam tiež uvedený spolu s dôvodom backendu; `report["sensors"]["missing"]` zostáva pre svetelné senzory prázdny.

---

## Priamy hardvér (bez backendu)

Ak chcete, aby kamery LATTICE (CI, bezhlavé roboty, vstavané) neboli vôbec závislé od backendu, importujte priamo `lattice_sdk` — ten je opätovne exportovaný prostredníctvom `chloros_sdk`. Upozornenie k `CAMERA_AVAILABLE`: `lattice_sdk` je súčasťou balíka SDK, vyžaduje však prítomnosť runtime prostredia Arena SDK a OpenCV (`cv2`), pričom balík neinštaluje ani jedno z nich. Svetelné senzory prechádzajú cez backend — [`connect_daq_sensor()`](#daq-sensor-sessions) alebo cez `proj.sensors` daného projektu — s jednou výnimkou, pomocným nástrojom DAQ-M pre veľkosti pásiem [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), ktorý tiež vyžaduje `bleak`.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Prednastavenia a spúšťač

Tri zo štyroch prednastavení sú typu **free-run**: kamera exponuje nepretržite a
`capture()` vráti ďalší snímok. Výnimkou je `triggered` — aktivuje
fotoaparát pri hardvérovom signále na linke 2, takže nezachytí nič, kým signál nepríde.

| Prednastavenie | Spúšťač | Použite, keď |
| --- | --- | --- |
| `default` | voľný chod | všeobecné použitie |
| `high_speed` | voľný chod | 8-bitový, obmedzenie na 60 fps, krátka expozícia |
| `high_quality` | voľný chod | 12-bitový, bez obmedzenia fps — bežná voľba pre fotografie |
| `triggered` | **pripravená, linka 2** | kamera je pripojená k synchronizačnému káblu M8 a spúšťa ju niečo iné |

Ak zvolíte `triggered` (alebo si sami nastavíte `trigger_mode="On"`) bez toho, aby
niečo riadilo Linku 2, každý `capture()` vyprší — správne, keďže ste požiadali
kameru, aby čakala. SDK to vysvetľuje, keď k tomu dôjde; pozrite si
[SC_ERR_TIMEOUT počas snímania](#direct-hardware-backend-free).

> **Poznámka — Správy „GVSP probe“ / `SC_ERR_TIMEOUT -1011` pri pripájaní nie sú chybami.**
> Pri pripájaní sa SDK snaží dohodnúť**jumbo rámce** (9000-bajtové GVSP pakety) pre vyššiu priepustnosť. Na priamom spojení sieťovej karty typu point-to-point (napr. adresa `169.254.x.x` v rámci lokálnej siete) sieť zvyčajne nedokáže prenášať jumbo rámce, takže táto kontrola vyprší a do protokolu sa zapíšu riadky ako:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Ide o **predpokladanú záložnú možnosť**: SDK sa automaticky vráti k štandardným 1500-bajtovým paketom a kamera pokračuje v pripájaní normálne (nasledujúce riadky typu `[chunk-enable …]` sú súčasťou bežnej sekvencie pripájania). Zachytávanie stále funguje.
>
> Túto sondu môžete preskočiť, ale **nejde len o potlačenie protokolovania — vypína jumbo rámce.** Kamera odpovedá na pingy s príkazom „Don&#x27;t-Fragment“ len do veľkosti 1500 bajtov bez ohľadu na to, aká dobrá je vaša sieť, takže samotný ping test nikdy nemôže zistiť prítomnosť jumbo rámcov; táto kontrola je jediná, ktorá to dokáže. Ak ju deaktivujete, kamera bude navždy používať štandardné 1500-bajtové pakety, v akejkoľvek sieti:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Oplatí sa to len v sieti, o ktorej *viete*, že nepodporuje jumbo rámce, kde to ušetrí približne jednu sekundu času pripojenia na jednu kameru. Keďže ide o skutočný kompromis a nie len o kozmetickú zmenu, SDK to teraz pri jej použití uvádza:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Nechajte to tak, pokiaľ nemáte dôvod.**Ak zostane povolené, každá kamera sa meria raz za reláciu backendu a znovu, keď sa zmení jej spojenie (iný adaptér, iná rýchlosť spojenia alebo prerušenie spojenia, ako napríklad pri výmene prepínača); pri opätovnom pripojení sa výsledok znovu použije.**Po zmene za prepínačom, ktorý zostal v prevádzke, reštartujte backend, aby sa meranie vykonalo znovu.** Prvé pripojenie môže pri tejto kontrole občas trvať niekoľko minút a v takom prípade to bude uvedené v protokole.
>
> Ak *chcete* priepustnosť jumbo, aktivujte jumbo end-to-end (MTU sieťovej karty 9000 + prepínač, ktorý ich prepúšťa), alebo ju zafixujte pomocou `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, ak viete, že spojenie ju prenáša — hoci uprednostňujte použitie `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` pre jednotlivé príkazy pred trvalým nastavením, pretože pevne nastavená veľkosť preskočí test a zastaví prispôsobovanie sa sieti pred ňou. **Každé** zariadenie na trase musí prenášať jumbo pakety — vrátane akéhokoľvek PoE rozdeľovača alebo injektora, čo je bežný dôvod, prečo inak jumbo-kompatibilná konfigurácia nedokáže tieto pakety prenášať.

> **`SC_ERR_TIMEOUT -1011` počas `capture()` / `grab*()` je iný problém — táto je skutočná chyba.**
> Vyššie uvedená poznámka sa týka iba chyby `-1011` zaznamenanej sondou**connect-time**. Ak sa tá istá chyba vyskytne pri**zachytávaní**, znamená to, že kamera sa pripojila v poriadku, ale neposiela žiadne snímky:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Rozpoznávacím znakom je kamera, ktorej *ovládací* kanál funguje správne — vyhľadávanie prebieha bez problémov, nastavenia a zápisy `[chunk-enable …]` sú úspešné — zatiaľ čo *každý* snímok vyprší.
>
> **Bežnou príčinou je, že kamera je nastavená na hardvérovú spúšť.** Pri chybách `trigger_mode="On"` a `trigger_source="Line2"` kamera nevysiela vôbec nič, kým na synchronizačnom kábli M8 nedôjde k elektrickému nábehu. Ak nemáte kábel, ktorý by túto linku riadil, každé zachytenie čaká donekonečna. Kamera nie je pokazená a sieť funguje správne — robí presne to, čo má.
>
> Kódy `CameraSettings()` a `default` / `high_speed` / `high_quality` nastavujú voľný chod a zachytenie, ktoré vyprší počas aktivácie, sa vysvetlí samo namiesto zobrazenia holého kódu `-1011`. `PRESETS["triggered"]` aktivuje linku 2, podľa návrhu.
>
> Ak chcete vynútiť voľný chod akejkoľvek kamery:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Ak aj s nastavením `trigger_mode="Off"` stále dochádza k časovému limitu, kamera skutočne neposiela údaje – pošlite nám protokol a `ip link show`.

#### Farebné profily (živý náhľad RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` určuje farebný profil displeja pre **živý náhľad** na RGB kamerách (multispektrálne kamery toto nastavenie ignorujú):

| Profil | Význam |
| --- | --- |
| `raw` | Úplné obídenie radiometrického reťazca. |
| `linear` | DSNU + flat + WB, bez CCM, bez gama. |
| `natural` | Lineárne + namerané CCM + sRGB gama, iba s lacnou úpravou (vyhladenie chrominancie + desaturácia svetiel) — realistické predvolené nastavenie. |
| `enhanced` | `natural` plus kompletná úprava typu „hub-parity“ (odstránenie farebných okrajov, živosť, lokálny kontrast CLAHE). Bohatší vzhľad pri približne **dvojnásobných nákladoch na úpravu jedného snímku**, teda nižšia snímková frekvencia v režime LIVE. |
| `custom_temp` | `natural`, ale vyváženie bielej (WB) je zafixované na `custom_cct_k` Kelvin (DLS sa ignoruje; na strane backendu obmedzené na 2000–10000 K). |

Profil je ovládačom rýchlosti/vzhľadu **len pre živý náhľad**: uložené snímky vždy získajú plné a bohaté spracovanie bez ohľadu na vybraný profil, takže výber `natural` s cieľom získať späť čas na snímku neznižuje kvalitu toho, čo sa uloží na disk. Neznámy profil aktivuje `ValueError`; ak je dostupný backend chloros, zmena sa odošle aj do neho, takže sa to odzrkadlí v nasledujúcom náhľadovom snímku (užívatelia Direct SDK bez backendu stále dostanú zmenu nastavení).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Najsvetlejšie miesta (displej RGB) — `set_highlights`

`LatticeCamera.set_highlights(value)` potláča najjasnejšie a najbielejšie pixely **displeja** RGB kamier — oprava pre jasný biely povrch, ktorý sa vykresľuje ako plochá biela škvrna. `value` je `-100`..`0` (obmedzené); `0` je vypnutý a výsledný obraz je potom presne taký, aký by bol bez tohto nastavenia. Silnejšie nastavenia znižujú bielu farbu displeja (na približne 60 % pri `-100`) a rozširujú tónovú krivku, takže povrch, ktorý automatický jas v náhľade posunul do čistej bielej, získa späť svoje detaily.

Ide o nastavenie displeja, ale na rozdiel od sýtosti a kontrastu (ktoré menia len živý náhľad) mení aj to, čo uložíte: živý náhľad a snímky na **úrovni displeja** sa tým riadia, takže aj tieto súbory sú tmavšie (obsahujú hodnotu `Chloros:DisplayHighlights`); exporty radiancie a odrazivosti sa tým nikdy nemenia a nedokáže obnoviť údaje zo senzora, ktoré boli orezané — na to znížte expozíciu. Multispektrálne a monochromatické kamery to ignorujú. Hodnota sa ukladá do `settings.highlights` a odosiela sa prostredníctvom POST na dostupný backend chloros, presne ako `set_color_profile`; ekvivalent v CLI je `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Mono (M3M) kamery a `Calibration`

Mono **M3M** kamera (`M3M-<lens>-F<wavelength>`) je jednopásmová: jedna rovina v odtieňoch šedej, bez Bayerovej mozaiky, bez spektrálnej matice 3×3. `Calibration` ju rozpozná a nastaví príznak `is_mono`. Odrazivosť sa stále uplatňuje ako radiometrická mapa pre jednotlivé pásma (demix je identitná matica), ale viacpásmové výpočty na jednej kamere skôr zvyšujú hodnotu, než aby vracali nezmysel:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Ak chcete vytvoriť vegetačný index z monokromatického hardvéru, skombinujte niekoľko kamier M3M s rôznymi vlnovými dĺžkami do zarovnaného viacpásmového súboru (pozri [Zarovnanie poľa](#array-alignment)) a vypočítajte index pre celý tento súbor namiesto jednej kamery.

#### Magnitúdy pásiem DAQ-M — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` sa pripája k zariadeniu DAQ-M cez Bluetooth pomocou vlastnej adresy BLE MAC. Je to jediná trieda svetelných senzorov, ktorá komunikuje so senzorom priamo, a nie cez backend, takže potrebuje to, čo potrebujú triedy priamych kamier (`CAMERA_AVAILABLE`: runtime SDKu Arena a OpenCV), plus `bleak` pre Bluetooth (`pip install "chloros-sdk[camera]"` alebo `pip install bleak`; bez neho `DLS` vyvolá chybu `DLSError`). Ak je zariadenie DAQ-M pripojené inde (v poole backendu, v projekte, v grafickom rozhraní) prestane vysielať, preto ho tam najprv uvoľnite. **`mac_address` je povinné**: neexistuje žiadna predvolená hodnota, keďže každá jednotka má svoju vlastnú, a ak chýba, `DLS()` vyvolá `DLSError` ešte pred pokusom o akékoľvek pripojenie. Nájdite MAC pomocou `discover_daq_sensors(transports=["ble"])` (pozri [Vyhľadávanie](#discovery--finding-an-address-to-connect-with)) alebo `chloros-cli daq pool-discover --only ble`.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integruje spektrum DAQ-M (najnovšia hodnota, ak je `spectrum` rovné `None`; `DLSError`, ak ešte žiadne nie je) prostredníctvom odoziev troch kanálov kamery a vráti `{"red": …, "green": …, "nir": …}` — kľúče označujú kanály v tomto poradí; pri krivkách RGBovej kamery je tretí (`nir`) jej modrý kanál.

- Bez `spectral_response` používa krivku flotily filtračného modelu (`filter_model`, s ktorou bol `DLS` vytvorený), ktorá bola nameraná na jednom zariadení.
- `spectral_response` namiesto toho používa **vlastné** namerané krivky kamery: slovník `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, uložený v kalibračnom balíku danej kamery (`color_correction.spectral_response_smoothed` — RGB kamery; multispektrálne balíky takúto krivku neobsahujú). Ak nemôže použiť daný slovník, prejde na krivku flotily.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

CLI funkcia `chloros-cli lattice dls --camera SERIAL` vykonáva to isté prostredníctvom backendu, pričom vyhľadáva krivku kamery v jej načítanom alebo uloženom v cache kalibračnom balíku a uvádza, ktorú krivku použila. Cez HTTP je to `POST /api/dls/cli-read` s `mac` (povinné: bez toho je odpoveď HTTP 400, `code` `MAC_REQUIRED`) a `camera_serial` v tele správy; odpoveď `band_magnitudes_source` je `camera <serial>`, ak bola použitá vlastná krivka danej kamery, alebo `model <filter>` pre krivku flotily (kamera bez krivky: nie je RGB alebo nie je načítaný balík ani v cache).

---

## Výnimky

Zachyťte základnú triedu na spracovanie „všetkého, čo sa v Chloros pokazilo“:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` a `ChlorosConfigurationError` sú atribúty `chloros_sdk` (a `chloros_sdk.exceptions`), ale nie v `__all__`, takže `from chloros_sdk import *` ich nezahrňuje: importujte ich podľa názvu. Rodina `ChlorosError` je odvodená od `ChlorosLocal` a funkcií na nej postavených (`process_folder()`, `process_lattice_capture()`); rozhranie smart-connect nadväzuje na `ChlorosConnectError`.

Hierarchia:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Príklady od začiatku do konca

### 1. Spracovanie priečinka s vlastným ukazovateľom priebehu

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Pole LATTICE v reálnom čase → Odrazivosť + referenčné údaje DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Zber údajov riadený projektom

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Prúd snímok z viacerých kamier → NumPy pipeline

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Skript na snímanie bez grafického rozhrania priamo na hardvéri (bez backendu)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Testovanie schopností pred pripojením sústavy so 4 kamerami

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Ekvivalent receptu na snímanie (čistý Python)

`run_project_recipe()` spúšťa recept CLI tak, ako je (pozri [Projektová zložka a recepty](#project-folder--recipes)). Ak je však napísaný v jazyku Python, kroky sú rovnaké:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Server MCP (asistenti umelej inteligencie)

`chloros-mcp` umožňuje AI asistentovi, ktorý komunikuje prostredníctvom [Model Context Protocol](https://modelcontextprotocol.io) (MCP), ako napríklad Claude Desktop, Claude Code alebo Cursor, ovládať Chloros: spracovávať snímky (vytvoriť projekt, importovať zložku so záznamami, vybrať nastavenia, spracovať, vygenerovať správu o výsledkoch spracovania) a ovládať kamery LATTICE, pole kamier a svetelné senzory DAQ (pripojiť, zobraziť náhľad, nastaviť expozíciu a farbu, zachytiť, zaznamenať, aktualizovať firmvér). Jeho nástroje pokrývajú funkcie tohto SDK. Požiadate v bežnom jazyku a asistent vyvolá nástroje servera:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

Server je postavený na tomto SDK: nástroje na spracovanie volajú [`ChlorosLocal`](#image-processing--chloroslocal) a hardvérové nástroje volajú [funkcie smart-connect](#smart-connect-for-lattice-cameras), [DAQ relácie](#daq-sensor-sessions), `ChlorosProject` a [pomocné programy pre kamery](#camera-utilities), ktoré všetky komunikujú s tým istým backendom, aký používa aplikácia Chloros. Ich požiadavky obsahujú `X-Chloros-Client: sdk`, takže backend uplatňuje kontroly prihlásenia a plánu SDK a s tým sú spojené aj pravidlá SDK (napríklad funkcia Texture Aware je pre zábery z kamery LATTICE odmietnutá).

### Inštalácia

Server je súčasťou balíka `chloros-sdk`; nie je potrebné inštalovať nič ďalšie. SDK nainštaluje príkaz `chloros-mcp`, ktorý je rovnaký ako `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** inštalátor Chloros pomocou pip nainštaluje SDK do vášho Pythonu, čím sa `chloros-mcp.exe` umiestni do zložky `Scripts` daného Pythonu.
- **Linux (.deb):** balík nainštaluje SDK pre systémový adresár `python3` a umiestni príkaz `chloros-mcp` do adresára `/usr/bin`, takže je k dispozícii v adresári `PATH`. Vo virtualenv-e pridá `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` súbor `chloros-mcp` do adresára `bin` daného virtualenv-u.

Server používa iba štandardnú knižnicu nad SDK (Python 3.7+). Spúšťa ho klient MCP a komunikuje s ním cez stdin/stdout; nespúšťate ho sami, s výnimkou `--help`.

### Nastavenie klienta

**Claude Desktop:**pridajte server do `claude_desktop_config.json` (v systéme WindowsX do `%APPDATA%\Claude\claude_desktop_config.json`; otvoríte ho cez**Nastavenia &gt; Vývojár &gt; Upraviť konfiguráciu**), potom reštartujte Claude Desktop:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude Code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Kurzor:** rovnaký záznam `mcpServers` ako v Claude Desktop, v `~/.cursor/mcp.json` (v každom projekte; `%USERPROFILE%\.cursor\mcp.json` v systéme Windows) alebo `.cursor/mcp.json` v jednom projekte.

Možnosti sa zadávajú v `args` alebo za príkazom pre Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`). Ak chcete použiť konkrétne Python SDK namiesto `chloros-mcp` v `PATH`, spustite modul s týmto Pythonom (jeho úplná cesta ako `command`, ak nie je prvým `python3` na `PATH`):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Klient spúšťa server bez vášho shellu, takže ak hlási, že sa server nepodarilo spustiť, `chloros-mcp` pravdepodobne nie je na `PATH`, ktorý klient vidí. Namiesto toho zadajte úplnú cestu (`where chloros-mcp` v systéme Windows, `which chloros-mcp` v systéme Linux, kde sa nachádza súbor .deb) alebo spustite modul pomocou Pythonu, ktorý obsahuje SDK.

### Možnosti

| Možnosť | Predvolené nastavenie | Čo robí |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | Backend Chloros, ktorý sa má použiť. |
| `--no-auto-start` | vypnuté | Nikdy nespúšťať backend: použiť už bežiaci backend, napríklad backend aplikácie Chloros, backendovú službu pre Linux alebo tunel k (pozri [Režim vzdialeného backendu](#remote-backend-mode-pip-only-host-via-tunnel); cesty k zložkám musia v tom prípade označovať rovnakú zložku na oboch zariadeniach). |
| `--backend-exe PATH` | tam, kde ho nainštaluje Chloros | Backend, ktorý sa má spustiť, ak žiadny nebeží, pre Chloros nainštalovaný na mieste, kde SDK nehľadá. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` alebo `ERROR`. Protokoly sa zapisujú do stderr, čo väčšina klientov zobrazuje vo svojom protokole MCP; stdout prenáša iba protokolové správy. |

### Nástroje

Argumenty každého nástroja sú argumenty volania SDK, ktoré tento nástroj obalí; klient uvádza každý z nich spolu s jeho úplnou schémou. Cesty sú absolútne.

**Spracovanie**

| Nástroj | Volanie SDK | Čo robí |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Vytvorí projekt a nastaví ho ako otvorený projekt, na ktorom pracujú spracovateľské nástroje. Zadanie jednoduchého názvu vytvorí priečinok vo výchozom priečinku projektu; zadanie absolútnej cesty ho vytvorí priamo na danom mieste. |
| `open_project` | `ChlorosLocal.open_project` | Otvorí uložený projekt a nastaví ho ako otvorený projekt. Cesty presunutého projektu sú presmerované na zložku, v ktorej sa projekt nachádza teraz; `relink_folder` (absolútny) označuje zložku, v ktorej sa teraz nachádzajú ostatné chýbajúce súbory. Vráti `source_files` a `image_count`. Nenašlo sa nič: chybové hlásenie s uvedením ich pôvodnej polohy a názvom `relink_folder`; nič nie je otvorené. |
| `import_images` | `import_images` | Importuje páry Survey3 `.RAW`+`.JPG`, LATTICE `.tif` snímky a DAQ `.daq` záznamy; vráti `count` (obrázky) a `scan_count` (záznamy). |
| `configure_processing` | `configure` | Zmení zadané nastavenia a žiadne iné (hodnoty: [Podporované hodnoty](#supported-values)); `custom_settings` odošle celú konfiguráciu tak, ako je zadaná. |
| `get_project_settings` | `get_config` | Načíta uložené nastavenia spracovania, exportu a indexovania otvoreného projektu. |
| `start_processing` | `process(wait=False)` | Spustí spracovanie otvoreného projektu a ihneď sa vráti. |
| `get_processing_status` | `get_status` | Vygeneruje správu `state` (`idle`, `starting`, `processing` alebo `finished`) a priebeh; po dokončení sa zapíšu výsledky behu: `summary`, `export_check_failure`, ak sa exporty nedali overiť, a `image_failure`, ak došlo k strate snímok; ostatné výstupy sa zapíšu; beh sa nahlási ako neúspešný. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Vytvorí projekt pre priečinok, načíta ho, nakonfiguruje a spustí beh v jednom volaní (`lattice: true` pre snímky LATTICE). Ak je beh v procese, odmietne sa ešte pred vytvorením čohokoľvek a backend v tom čase tiež odmietne zmenu projektu z akéhokoľvek nástroja. |
| `export_light_sensor` | `export_light_sensor` | Zapíše kalibrované súbory `.daq` a `.csv` pre záznamy svetelného senzora projektu bez spracovania obrázkov, pričom načíta továrenskú kalibráciu z MAPIR Cloud, ak nie je uložená v cache (pozri [Záznamy svetelného senzora](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Načíta metadáta auditu Chlorosu zo spracovaného obrazu (zdroj, kalibrácia, použité fázy). |

**Kamery a polia**

| Nástroj | Volanie SDK | Čo robí |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Čo je momentálne pripojené v ktoromkoľvek klientovi, každý s `connected_here`; plus úlohy spustené na serveri. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Kamery a svetelné senzory, ktoré by mohli byť pripojené. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Aktuálny stav jednej kamery, skupiny kamier (vrátane ich schopností a stavu) alebo svetelného senzora. |
| `analyze_array_network` | `analyze_array_network` | Čo tento hostiteľ a sieť dokážu obsluhovať pre navrhované pole. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Pripojte kameru alebo synchronizovanú skupinu (najskôr hlavnú kameru). |
| `connect_project_devices` | `open_project(...).connect_all` | Pripojte každé zariadenie, ktoré si uložený projekt pamätá, spolu s jeho nastaveniami. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Uvoľnite kameru, pole alebo svetelný senzor (pozri Správanie). `keep_settings: true` na poli uloží nastavenia každej kamery do jej vlastnej pamäte, `clear_settings: true` túto pamäť vynuluje (jeho výsledok nesie `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Zariadenia projektu ako celok. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Čítanie alebo zapisovanie nastavení kamery podľa popisného mena alebo názvu uzla GenIC. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Nastavenie rýchlosti poľa alebo návrat k automatickej rýchlosti. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | Jedno zachytenie (`fastest` a `wait_for_exposure` sú `capture_fastest` a snímka s nastavenou expozíciou). Ak chýba `output_dir`, uloží sa do otvoreného projektu`captures` a súbory sa pripoja k projektu (zadržané až do ukončenia spracovania); odpoveď `project_add` to potvrdzuje. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Neprerušované alebo intervalové snímanie ako úloha. |
| `start_array_recording` | `ArraySession.record` / `burst` | Video s kombinovaným indexom alebo surová séria snímok ako úloha. |
| `build_burst_video` | `ArraySession.build_video` | Opätovné spracovanie série snímok na kalibrované video ako úloha. |
| `get_job_status` / `stop_job` | | Sledovanie alebo zastavenie úlohy. |
| `get_camera_preview` | | Živý náhľad z každej kamery, vrátený ako obrázok. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Jeden snímok (alebo jedna synchronizovaná sada) na úrovni spracovania bez uloženia: štatistiky za každú rovinu a súbor PNG pre každú z nich. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | Zarovnanie poľa. |

**Svetelné senzory**

| Nástroj | Volanie SDK | Čo robí |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Pripojenie zariadenia DAQ-U, DAQ-M alebo DAQ-E (`cap_id` nastavuje korekciu krytu). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | Najnovšie namerané hodnoty: časová pečiatka, saturácia, rozsah spektra a vrchol, ako aj spektrum zpriemerované do vlnových dĺžkových intervalov (`full_spectrum` pre každý bod). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Spustenie alebo zastavenie prenosu. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Nahrať súbor `.daq` ako úlohu; `duration_s` ho sám zastaví. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | Korekcia limitu (`none` pre holý senzor). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Ktorý senzor poskytuje údaje o odrazivosti jednotlivých kamier a polí. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Sady pásiem a odrazivosť a indexy okolitého prostredia a objektov dvoch senzorov so streamovaním. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | Vlastné nastavenia zariadenia DAQ-E, uložené v zariadení. |

**Nástroje pre kameru** ([Nástroje pre kameru](#camera-utilities))

| Nástroj | Volanie SDK | Čo robí |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` a ich nastavovacie funkcie | Farba zobrazenia RGB kamery: profil a farebnú teplotu, sýtosť / kontrast, najsvetlejšie miesta. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Jednorazové vyváženie bielej alebo automatické / vypnuté / pevné pomery. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Prednastavenia kamery. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Modely filtrov alebo jeden podľa názvu. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Verzie firmvéru a flash (štandardne úloha). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Stav napájania a teploty; režim spánku alebo prebudenia. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Koeficienty z kalibračného cieľa v zobrazení. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Sieťové adaptéry kamery, odhad a oprava (len pre správcu). |
| `apply_vignette_correction` | `apply_vignette_correction` | Korekcia vinety v priečinku s obrázkami. |
| `read_dls` | `read_dls` | Jedno čítanie DLS. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | Hardvér a prostredie hostiteľa. |

**Prechodné kamery, profily zarovnania a indexy** ([Prechodné polia, profily zarovnania a indexy](#transient-arrays-alignment-profiles--indices)). Každý nástroj, ktorý tu otvára kamery, používa `keep_settings` / `clear_settings` (predvolené nastavenie false): pozri [Pamäť kamery](#camera-memory--keep_settings--clear_settings).

| Nástroj | Volanie SDK | Čo robí |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Nahrávanie z jednej kamery alebo synchronizované streamovanie z viacerých kamier ako úloha (`duration_s` to zastaví automaticky). |
| `capture_cameras_transient` | `capture_cameras_transient` | Jedno synchronizované nahrávanie bez pripojenia poľa. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Kontrola synchronizácie spúšťača a zistenie, ktoré kamery sú v synchronizačnej linke. |
| `analyze_array_models` | `analyze_array_models` | Čo by v tejto sieti dosiahlo pole týchto modelov. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Vyriešiť a uložiť súbor profilu zarovnania (`options` obsahuje podrobnejšie možnosti riešiteľa). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Zarovnaný viacpásmový obrázok alebo ich séria ako úloha. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | Obsah súboru profilu; poradie pásiem a povolené kamery. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Prednastavenia a prechody indexu; index pre viacpásmový súbor TIFF; zachytenie do indexu v jednom kroku. |

**Synchronizácia času, zložka projektu a recepty** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Zložka projektu a recepty](#project-folder--recipes))

| Nástroj | Volanie SDK | Čo robí |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | Stav PTP grandmastera, jeho partnerov a stav PTP každej kamery v poli. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Reštartujte ho; nastavte jeho priority. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | Zložka, v ktorej sa vytvárajú nové projekty. |
| `run_project_recipe` | `run_project_recipe` | Spustite recept uloženého projektu ako úlohu s jej protokolom. Najprv sa vykoná kontrola: recept s chybou sa odmietne a nič sa nespustí; v prípade nepripojených polí / senzorov sa zobrazí varovanie (nepripojí ich: najprv sa spustí `connect_project_devices`). `dry_run` vráti výsledok kontroly a nič nespustí. |

Neexistuje žiadny nástroj na otvorenie uloženého projektu na spracovanie: každá úloha spracovania začína s `create_project` (alebo `process_folder`).

### Požiadavky

- **Chloros nainštalovaný na tom istom počítači.** Spracovanie vykonáva backend Chloros. Server môže namiesto toho použiť backend, ku ktorému sa pripája cez tunel (`--backend-url` s `--no-auto-start`, pozri [Možnosti](#options)).
- **Prihlásenie na danom počítači** v aplikácii Chloros alebo pomocou `chloros-cli login`. Server sa nemôže prihlásiť za vás: nástroj spustený v stave odhlásenia odpovie „Nie ste prihlásený do Chloros“ s dôvodom uvedeným backendom a asistent vás na to upozorní.
- **Akýkoľvek platený balík Chloros+ (Copper alebo vyšší).** Server má prístup cez SDK, pričom bezplatný balík Iron ho nemá: nástroj spustený v rámci bezplatného balíka odpovie „Nedostupné v tomto pláne Chloros“ spolu s dôvodom zo strany backendu.
- Python 3.7 alebo novší s SDK a klient MCP, ktorý spúšťa lokálne (stdio) servery.

### Správanie

- **Backend.** Spustenie servera alebo zobrazenie zoznamu jeho nástrojov nespustí nič; prvé volanie nástroja nadviaže spojenie. Ak beží aplikácia Chloros (alebo backendová služba pre Linux), server používa jej backend a nechá ho bežať. Ak nebeží žiadny backend, server spustí nainštalovaný backend bez okna. Keď sa server ukončí (ukončíte klienta alebo sa server reštartuje), zastaví úlohy snímania a záznamy zo svetelných senzorov, ktoré spustil, a uvoľní zariadenia, ku ktorým sa pripojil; backend naďalej beží, takže nedokončené spracovanie prebieha až do konca a backend sa sám zastaví po 30 minútach, ak ho nič nevyužíva. S `--no-auto-start` server nikdy nespustí backend: volanie nástroja, keď žiadny backend nebeží, vás vyzve na otvorenie aplikácie Chloros a ďalšie volanie po jej otvorení nadviaže spojenie.
- **Jeden otvorený projekt.** Backend má otvorený jeden projekt, ktorý zdieľa s aplikáciou Chloros, keď je aplikácia otvorená, preto nepracujte na inom projekte v aplikácii, kým asistent pracuje.
- **Zmenia sa len odovzdané nastavenia.** `configure_processing` ponechá každé nastavenie, ktoré nebolo zadané, na jeho uloženej hodnote (SDK `configure()` obnoví nastavenia debayera, korekcie vinety, kalibrácie odrazivosti, PPK a formátu exportu na ich predvolené hodnoty, ak nie sú odovzdané; server ich vyplní z projektu). Ak nedokáže prečítať uložené nastavenia projektu, nič nemení a oznámi to, pokiaľ nie sú zadané všetky päť. `indices` nahradí zoznam indexov projektu. Výsledok uvádza to, čo bolo odoslané ako `settings_sent` a pod `warnings` uvádza názvy všetkých indexov, ktoré pásma kamery nedokážu vytvoriť.
- **Spracovanie prebieha podľa princípu „spusti a potom zisťuj stav“.** `start_processing` sa vráti hneď po spustení spracovania, ktoré potom trvá od niekoľkých minút až po hodiny, a asistent na to nadviaže s `get_processing_status`. `state` je `idle` (žiadne spracovanie), `starting` (backend ešte neohlásil spustenie, maximálne 20 s), `processing` alebo `finished`, spolu s [súhrn](#post-run-summary--hints): čo exportoval a upozornenia, ak exportoval menej, než sa očakávalo (žiadne snímky, nenašiel sa kalibračný cieľ). Výstupy sa ukladajú do projektového priečinka, rovnako ako z aplikácie. `start_processing` je odmietnutý, pokiaľ beh prebieha, a backend, ktorý prestane odpovedať, je nahlásený ako chyba, nie ako dokončený beh.
- **Čo vidí asistent.** Varovania SDK sa zobrazujú vo výsledkoch nástroja pod kódom `warnings` a zlyhania (odhlásenie, plán, nesprávna cesta) sa zobrazujú ako chybový text, ktorý asistent dokáže prečítať a odovzdať vám. Zoznam s viac ako 20 položkami sa skráti na prvých 20, pričom jeho úplná dĺžka sa uvádza ako `<key>_total`. Cesty musia byť absolútne.
- **Zariadenia sú zdieľané.** Kamery, polia a svetelné senzory sa nachádzajú v backende, takže asistent môže ovládať zariadenie pripojené cez aplikáciu Chloros a všetky zmeny sa zobrazia v aplikácii. `disconnect_device` uvoľňuje to, čo pripojil server; zariadenie pripojené iným klientom (aplikácia, `chloros-cli`, skript, iný `chloros-mcp`) sa uvoľní iba pomocou `force: true`, čo je `force_disconnect` v rámci SDK, a v prípade odmietnutia sa uvedie meno toho, kto zariadenie pripojil, ak to backend vie; zariadenie, ktoré backend vôbec neeviduje, je hlásené ako nepripojené (`released: false`, `connected: false`, nie je čo uvoľniť). Nahrávka alebo stream, ktorý backend spúšťa, vedie k zobrazeniu `get_job_status` s hodnotami `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (skupiny snímok pre stream z viacerých kamier) — a jednoriadkový výpis `summary` („220 snímok zapísaných, 55 vynechaných, 0 zlyhaní: ...“) počas behu a po dokončení s konečnými číslami; výpis od iného klienta (`other_clients_jobs`) obsahuje to isté (`list_cameras()`, `list_arrays()` a `list_daq_sensors()` obsahujú pre každé zariadenie`connected_by`: `client`, `label`, `name`, `at`). Každé zariadenie `chloros-mcp` sa voči backendu identifikuje ako `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, odoslané ako `X-Chloros-Client-Label`; akýkoľvek skript SDK ho môže nastaviť). Zastavenie záznamu zo svetelného senzora iného klienta si vyžaduje aj `force`.
- **Dlhotrvajúca úloha je úloha.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, a akákoľvek požiadavka na zachytenie, vyriešenie alebo indexovanie s kódom `background` okamžite vráti kód `job_id`; nasleduje kód `get_job_status` a kód `stop_job` to ukončí . Spustenie pokračuje, kým nie je zastavené, pokiaľ mu nezadáte `max_captures`, `duration_s` (alebo v prípade sériového záznamu `max_frames`); `min_free_gb` odmietne spustiť sa, ak má disk málo voľného miesta. Neplatí žiadne obmedzenie, pokiaľ ho nenastavíte. Nahrávanie alebo stream, ktorý backend spúšťa (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) si uchováva identifikátor úlohy backendu, takže ďalší proces `chloros-mcp` — spustený neskôr — ho nájde podľa tohto identifikátora: `get_job_status` naň nadväzuje (a vypisuje také úlohy ako `other_clients_jobs`, od `GET /api/camera/cli-jobs`, každú s `started_by`), a úloha `stop_job` ju zastaví až pomocou úlohy `force: true`. Akákoľvek iná úloha (zaznamenávacia úloha, záznam do poľa alebo burst, zaznamenávanie na pozadí) beží v úlohe `chloros-mcp`, ktorá ju spustila, a vie o nej len tento server: ako to potvrdí iný server, ktorý o ňu požiadal. Zastavená úloha číta z `stopping` počas zapisovania svojej fronty, potom z `finished` s konečnými počtami, počas 10 minút a z akéhokoľvek `chloros-mcp`: ten, ktorý to spustil, vidí vynútené zastavenie iného rovnakým spôsobom. `get_job_status` nikdy nečaká na zastavenie: počas zastavenia (svojho vlastného `stop_job`, zastavenia iného klientaalebo zastavenie, ktoré uvoľňuje kamery úlohy, ktorá sa sama ukončila) odpovie ihneď s `stopping` a aktuálnymi počtami, a `stop_job` úlohy, ktorá sa už zastavuje, to oznámi. Úloha, ktorá sa ukončila, uvádza dôvod v `stop_reason`: vlastný dôvod (`no_frames`, `disk_full`, `member_missing`, alebo v prípade nepretržitého behu poľa `count`, `duration`, `array_gone`), `limit reached` iba vtedy, ak jej počty ukazujú jej `count` (alebo `max_frames` pre sériu, `duration_s` pre záznam) bolo dosiahnuté, `error`, `stopped`,`duration_s`) dosiahnutý, `error`, `stopped` (s hodnotou `stopped_by`), ak ho klient zastavil, inak `finished`.
- **Kam sa ukladajú záznamy.** Do zložky `captures` otvoreného projektu, kde ich aplikácia ukladá, pokiaľ nie je zadané `output_dir`.
- **Obrázky.** `get_camera_preview` a `grab_frames` vracajú obrázky, ktoré si asistent môže prezrieť, popri ich JSON. Obrázky `grab_frames` sú na účely prehliadania kontrastne upravené; ich štatistiky sú skutočné hodnoty.
- **Keď sa server ukončí**, úlohy, ktoré spustil, sa zastavia a záznam zo senzora osvetlenia, ktorý spustil, sa uzavrie. Ak server spustil backend, pred jeho zastavením uvoľní aj zariadenia, ku ktorým sa pripojil; v backende aplikácie zostávajú pripojené.
- **Protokol.** Iba stdio (bez koncového bodu HTTP), iba nástroje (bez zdrojov alebo výziev), textové a obrazové výsledky, verzie protokolu MCP od 2024-11-05 do 2025-11-25.**Čo nie je zahrnuté.** Prihlásenie alebo odhlásenie (účet je váš; server sa nemôže prihlásiť) a zastavenie backendu, ktorý server nespustil. Na všetko ostatné, čo toto SDK robí so spracovaním a hardvérom, existuje nástroj. (Provisioning zariadenia DAQ-E – jeho názov, autentizačný token, heslo OTA, požiadavka na podpis, obnovenie továrenských nastavení – a priame servery DAQ sú výhradne v rámci `chloros-cli`, aj v SDK: nesprávny názov alebo token zablokuje každú inštaláciu Chloros v zariadení.)

---

## Automatické spustenie backendu

Vstupné body smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` a `discover_daq_sensors` — sú jednoduché HTTP klienti, ktorí predpokladajú, že backend načúva na adrese `127.0.0.1:5000` (predvolená URL rozhrania smart-connect). Ak je grafické rozhranie (GUI) alebo príkazový riadok (CLI) už spustené, jeden z nich už beží. Pri spustení z holého skriptu tam nemusí byť — preto tieto funkcie **automaticky spustia priložený binárny súbor backendu** (bez okna, rovnako ako to robí `ChlorosLocal`) pred ich prvým volaním a potom čakajú až 60 s, kým sa spustí. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` a `analyze_array_network()` ho nikdy nespustia.

Pravidlá:

- **Spúšťa sa výlučne lokálna URL.** Spustiteľný je len `backend_url` smerujúci na `localhost` / `127.0.0.1` / `[::1]` je prípustný; akýkoľvek iný hostiteľ sa považuje za stroj niekoho iného a nikdy sa nespustí.
- **Ak sa backend ukončí skôr, než odpovie, čakanie sa ukončí.** Volanie sa okamžite ukončí s výstupným kódom, príkazom, ktorý spustil, a poslednými riadkami, ktoré backend vypísal, namiesto toho, aby čakalo 60 sekúnd. Výnimkou je výstupný kód 98: port alebo súbor uzamknutia už drží iný backend (aplikácia Chloros, služba backendu alebo iný skript, ktorý sa ešte spúšťa), takže volanie naďalej čaká na odpoveď práve od neho. `ChlorosLocal` sa správa rovnako.
- **Backend zostáva spustený na účely opätovného použitia** (rovnako ako v CLI) — pri ukončení skriptu nedochádza k implicitnému vypnutiu. Opätovné spustenie skriptu znovu použije aktívny backend.
- **Túto funkciu môžete vypnúť pomocou `auto_start_backend=False`** pri ktoromkoľvek z týchto volaní (napr. ak ste nasmerovali na vzdialený backend alebo si životný cyklus backendu spravujete sami).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Ak sa nedá nájsť alebo spustiť priložený binárny súbor, nasledujúce volanie HTTP vyvolá akčnú, **platformovo špecifickú** chybu `ChlorosConnectError` namiesto holého záznamu o odmietnutí pripojenia — v systéme Windows vás nasmeruje na desktopovú aplikáciu alebo na príkaz `chloros-cli`; v systéme Linux (bez grafického rozhrania) vás nasmeruje na príkaz `chloros-cli` alebo na `.deb`.

---

## Prostredie a hlavičky

SDK označuje každé HTTP volanie backendu kódom `X-Chloros-Client: sdk`. Backend uplatňuje licenčné pravidlá SDK/CLI (vyžaduje sa prihlásenie **a** platený plán Chloros+), namiesto bezplatnej verzie v grafickom rozhraní. Toto sa nastaví automaticky pri importe — nemusíte robiť nič.

Hlavička sa pridá ku každému volaniu `requests` / `urllib`, ktorého URL obsahuje `localhost` alebo `127.0.0.1`, na akomkoľvek porte – vrátane vašich vlastných lokálnych služieb — a nahrádza hodnotu, ktorú ste nastavili sami. Volania na iné hostiteľa zostávajú nedotknuté.

URLu backendu môžete prepisovať odovzdaním `backend_url=` (`ChlorosLocal` prijíma `backend_url=` alebo `api_url=`; `process_folder()` a `process_lattice_capture()` prijímajú iba `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Nainštalovaný backend načúva iba na loopbacku, takže `backend_url` v LAN sa k nemu nemôže pripojiť; pozri [Režim vzdialeného backendu](#remote-backend-mode-pip-only-host-via-tunnel) pre vzor tunela.)

---

## Verzie a kompatibilita

- Verzia SDK-u je označená ako `chloros_sdk.__version__`.
- SDK viaže správanie na verziu dodávaného backendu. Kombinácia staršieho SDK s novším backendom zvyčajne funguje (koncové body kompatibilné s novšími verziami), ale kombinácia novšieho SDK so starším backendom môže na nových koncových bodoch spôsobiť chyby `404` — aktualizujte desktopovú aplikáciu tak, aby bola kompatibilná.
- Rozhranie smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) a koncový bod analýzy siete vracajú stabilné schémy JSON; nové polia sú doplnkové.

---

## Tipy na riešenie problémov

- **`ChlorosAuthenticationError: Login required`** (z `ChlorosLocal`; volania funkcie Smart Connect vyvolávajú chybu `ChlorosConnectError: Login required. …` z rovnakého dôvodu) → Spustite na tomto počítači raz `chloros-cli login EMAIL PASSWORD` alebo sa prihláste prostredníctvom desktopovej aplikácie Chloros.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (alebo `Failed to create project: …` z `create_project()`) → Na tomto počítači sú vypnuté dlhé cesty v systéme Windows (predvolené nastavenie systému Windows), preto žiadna cesta nesmie presiahnuť 260 znakov a produkty sa nachádzajú približne 130 znakov pod projektovou zložkou. Nič nebolo spracované. Správa uvádza najdlhšiu projektovú zložku, ktorá sa zmestí: použite kratšiu, alebo zapnite dlhé cesty v Windows a reštartujte Chloros. Inštalátor systému Windows ponúka túto možnosť ako „Povoliť dlhé cesty v systéme Windows“, ktorá je predvolene zaškrtnutá; inak použite zásady skupiny: *Konfigurácia počítača &gt; Šablóny správcu &gt; Systém &gt; Súborový systém &gt; Povoliť dlhé cesty Win32*.
- **`ChlorosConnectError: No Chloros backend is running …`** → Funkcia Smart-Connect volá automatické spustenie lokálneho backendu, takže táto chyba sa objaví len vtedy, keď sa nedá nájsť/spustiť priložený binárny súbor (napr. hostiteľ typu pipbez balíka pre pracovnú plochu). Správa závisí od platformy: v systéme Windows otvorte aplikáciu pre pracovnú plochu alebo spustite ľubovoľný príkaz `chloros-cli`; v systéme Linux spustite príkaz `chloros-cli` (neexistuje grafické rozhranie) alebo nainštalujte `.deb`. V prípade vzdialeného backendu zadajte `backend_url=` (a `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** pri importe → `lattice_sdk` sa nepodarilo načítať: runtime Arena SDK alebo OpenCV (`cv2`), nie je možné načítať v tomto Pythone. Povrch riadený backendom (`connect_*`, premietaný cez backend, `ChlorosLocal`) stále funguje.
- **Pole sa pripája pri plnom rozsahu senzora, ale s nízkou snímkovou frekvenciou**→ Tak je to navrhnuté: automatický režim zachováva plný rozsah senzora a nechá frekvenciu klesnúť na úroveň, ktorú prenosové médium zvládne. Prejdite na `binning=2` / `4` alebo menšie `width` / `height` pre vyšší počet snímok za sekundu, alebo povolte jumbo rámce / modernizujte spojenie. Binning a ROI**nepokrývajú** agregované prekročenie kapacity (`oversubscribed: true`, polia fps 0): príliš veľa kamier pre dané pripojenie nemožno vyriešiť pomocou binningu/ROI — znížte počet kamier, aktivujte jumbo rámce alebo prejdite na rýchlejšiu sieťovú kartu (pozri [Prekročenie kapacity](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` hlási, že prijímací krúžok sieťovej karty je veľmi malý (~0,26 MB) / pripojovacie brány s hlásením „FRAMES WILL DROP“** → Prijímací krúžok hostiteľskej sieťovej karty je nastavený na predvolenú hodnotu (často sa po aktualizácii ovládača sieťovej karty resetuje na 32). Na adaptéri Realtek USB 10GbE nastavte `ReceiveBufferLen=256` a `PendingReceives=64` (s zvýšenými oprávneniami) a potom reštartujte backend, aby znovu prečítal kruh. Úplný postup: [Referencia CLI → Nastavenie a ladenie hostiteľskej sieťovej karty](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Hostiteľ sa zasekne pri reštarte/vypnutí, neskôr sa objavujú chyby WMI `Invalid class` / sieťová karta sa nedá aktivovať** → Zastaraný ovládač USB 10GbE spôsobuje chybu `DRIVER_POWER_STATE_FAILURE` (modrá obrazovka smrti – BSOD `0x9F`). Aktualizujte ovládač adaptéra na aktuálnu verziu (≥ 2026) a znovu použite nastavenia prijímacieho kruhu. Pozrite si [Referencia CLI → Nastavenie a ladenie sieťovej karty hostiteľa](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Zachytávanie z poľa zlyhá s chybou HTTP 503 „No fresh synced frame group“** → Pole nezverejnilo žiadnu novú skupinu rámcov počas čakania. Skontrolujte, či hodnota `arr.status()`: `stalled: True` (alebo `streaming: False`) s hodnotou `actual_fps` 0 znamená, že zachytávacia slučka prestala zverejňovať — odpojte a znovu pripojte pole. Ak je hodnota `stalled` False a hodnota `actual_fps` je väčšia ako 0, skúste to znova. Hneď po hodnote `connect_array()` už zachytávanie čaká na prvú skupinu až 20 s.
- **`status()` zobrazuje hodnotu `link_speed_mbps: 4294`** → Systém Windows nedokázal zistiť rýchlosť adaptéra; ide o typický jav pri USB ethernetových adaptéroch a nie je to chyba. Pozrite si [Očakávané rýchlosti rámcov](#expected-frame-rates).
- **`update_firmware()` zlyhá s chybou `device never connected back on TCP <port>`** → Firewall zablokoval spätné pripojenie zariadenia DAQ-E k tomuto počítaču. Inštalačný program systému Windows pridá príslušné pravidlo (pozri [Nastavenia zariadenia DAQ-E](#daq-e-device-settings)); v prípade iného firewallu povolte prichádzajúci TCP prevádzku do backendu zo podsiete senzora.
- **Odrazivosť odmietnutá** → Pre meranie odrazivosti v absolútnej stupnici musí byť k kamere (alebo sústave) priradený aktívny DAQ. Priraďte ho buď prostredníctvom grafického rozhrania, alebo použite `processing="radiance"` (W/m²/sr/nm), ktorý nevyžaduje spárovaný senzor.
- **Zaznamenávanie pomocou `smart=True` trvá dlhšie, ako sa očakávalo** → Konvergencia AE závisí od dynamiky scény. Pri použití `ArrayHandle.capture_smart()` rozšírte `exposure_tolerance_pct`, skráťte `stability_window_s` alebo znížte hodnotu `settle_timeout_s`, čím dosiahnete rýchlejšie a menej ustálené spustenie.

---

## Pozri tiež

- [Referenčná príručka CLI](cli-reference.md) — každý podpríkaz CLI zodpovedá volaniu SDK.
- [Svetelné senzory DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) v príručke k Chloros — senzory, ich kryty a kalibrovaný rozsah, zaznamenávanie, sieťové pripojenie DAQ-E.
- [Kamery LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) v príručke k Chloros — pripájanie, nastavenia kamery, režimy snímania, polia.
- Online dokumentácia: `https://mapir.gitbook.io/chloros/api-python-sdk`

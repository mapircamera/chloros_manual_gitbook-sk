# Dokončenie spracovania

Po dokončení spracovania Chloros je čas skontrolovať výsledky, overiť kvalitu výstupu a pripraviť spracované obrázky na použitie vo vašom pracovnom postupe. Táto stránka vás prevedie záverečnými krokmi a ďalšími akciami.

## Indikácia dokončenia spracovania

Keď sa spracovanie úspešne dokončí, uvidíte niekoľko indikátorov:

* ✅ **Indikátor priebehu**: Dosiahne 100 % dokončenia
* ✅ **Debugovací protokol**: Zobrazí správu „Spracovanie dokončené“
* ✅ **Tlačidlo Štart**: Opäť sa aktivuje (pripravené na ďalšie spracovanie)
* ✅ **Výstupné súbory**: Všetky spracované obrázky uložené do podadresára modelu fotoaparátu

***

## Vyhľadanie spracovaných obrázkov

### Otvorenie výstupného adresára

1. Kliknite na ikonu **Hlavné menu** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> (vľavo hore)
2. Vyberte **„Otvoriť priečinok projektu“**
3. Otvorí sa priečinok projektu v prehliadači súborov
4. Vyhľadajte projekt podľa názvu

***

## Kontrola spracovaných obrázkov

### Rýchly náhľad v prehliadači súborov

**Vstavaný náhľad Windows:**

1. Prejdite do podadresára modelu fotoaparátu
2. Vyberte obrazový súbor
3. Náhľad sa zobrazí v okne náhľadu prehliadača Windows
4. Pomocou šípok prechádzajte medzi obrázkami

### Náhľad v externých prehliadačoch obrázkov

**Odporúčané prehliadače:**

* **QGIS** – bezplatný softvér GIS (najlepší pre multispektrálnu analýzu s georeferenciou)
* **IrfanView** – rýchly, ľahký prehliadač obrázkov (podporuje TIFF)
* **Adobe Photoshop** – profesionálna úprava (podpora TIFF)
* **GIMP** – bezplatná alternatíva k programu Photoshop
* **Windows Photos** – základné prezeranie (nemusí podporovať 16-bitový TIFF)

### Náhľad v prehliadači obrázkov Chloros

Na pokročilú vizualizáciu použite integrovaný prehliadač obrázkov Chloros:

1. Kliknite na miniatúru obrázka v prehliadači súborov.
2. Obrázok sa otvorí v hlavnej oblasti náhľadu.
3. Kliknite na kartu **Prehliadač obrázkov** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> v ľavom bočnom paneli.
4. Použite [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) na interaktívnu analýzu.

Podrobné pokyny nájdete v časti [Prehliadač obrázkov](../image-viewer-gui/opening-an-image-full-screen.md).

***

## Kontrola protokolu ladenia

### Skontrolujte varovania alebo chyby

1. Otvorte kartu **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> .
2. Prejdite správy.
3. Hľadajte žlté varovania alebo červené chyby.
4. Skontrolujte všetky zaznamenané problémy.
5. Kontaktujte podporu MAPIR a požiadajte o pomoc.

### Uloženie protokolu

Ak chcete uchovať záznam o spracovaní alebo ho poslať podpore MAPIR:

1. Kliknite na tlačidlo **„Kopírovať“** alebo **„Stiahnuť“**.
2. Uložte ako textový súbor do priečinka projektu.
3. Priložte k dokumentácii projektu.
4. V prípade problémov pošlite na podporu MAPIR.

***

## Bežné problémy s výstupom a ich riešenia

### Problém: Chýbajúce výstupné súbory

**Možné príčiny:**

* Súbory nespĺňali kritériá spracovania.
* Iba cieľové obrázky (vyňaté z exportu).
* Počas exportu došlo k vyčerpaniu miesta na disku.
* Poškodenie súboru počas spracovania.

**Riešenia:**

1. Skontrolujte protokol ladenia, či neobsahuje správy o preskočení/chybe.
2. Overte, či bolo dostatok miesta na disku.
3. Spočítajte súbory: Malo by sa zhodovať (pôvodný počet – cieľový počet) × (indexy + 1)
4. Znovu importujte a spracujte všetky chýbajúce súbory.

### Problém: Tmavé alebo svetlé okraje (vinetácia je stále viditeľná)

**Možné príčiny:**

* Korekcia vinetácie je vypnutá.
* Fotoaparát/objektív nie je v databáze profilov Chloros.
* Extrémna vinetácia presahuje možnosti korekcie.

**Riešenia:**

1. Overte, či bola v nastaveniach projektu povolená korekcia vinetácie.
2. Skontrolujte, či bol správne detegovaný model fotoaparátu.
3. Ak vinetácia pretrváva, kontaktujte podporu MAPIR.

### Problém: Nesprávne farby alebo hodnoty

**Možné príčiny:**

* Neboli detegované žiadne kalibračné ciele.
* Bol vybraný nesprávny model kalibračného cieľa.
* Kalibrácia odrazivosti je vypnutá.
* Nízka kvalita cieľových obrázkov.

**Riešenia:**

1. Overte, či bola povolená kalibrácia odrazivosti.
2. Skontrolujte správy „Cieľ nájdený“ v protokole ladenia.
3. Skontrolujte kvalitu obrazu cieľa.
4. Opätovne spracujte s označenými správnymi cieľmi.

### Problém: Hodnoty NDVI sa zdajú nesprávne

**Očakávané rozsahy NDVI:**

* **Voda, skaly, pôda**: -0,1 až 0,2
* **Riedka/nezdravá vegetácia**: 0,2 až 0,4
* **Stredná vegetácia**: 0,4 až 0,6
* **Zdravá, hustá vegetácia**: 0,6 až 0,9

**Ak sú hodnoty mimo týchto rozsahov:**

1. Overte, či bola použitá kalibrácia odrazivosti.
2. Overte, či bol zahrnutý protokol svetelného senzora.
3. Skontrolujte, či boli zistené kalibračné ciele.
4. Uistite sa, že bol zistený správny model fotoaparátu.
5. Skontrolujte načasovanie a podmienky snímania cieľového obrazu.

***

## Používanie spracovaných obrázkov

### Pre fotogrametriu / vytvorenie ortomozaiky

**Odporúčaný pracovný postup:**

1. **Importujte kalibrované obrazy odrazivosti** do fotogrametrického softvéru:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Zachovajte metadáta EXIF**: Uistite sa, že údaje GPS zostali zachované pre geotagging.
3. **Kalibrované pracovné postupy**: Použite obrazy odrazivosti pre vedeckú presnosť.
4. **Spracujte indexové mozaiky**: Vytvorte ortomozaiky NDVI z jednotlivých indexových snímok
5. **Exportujte georeferencované GeoTIFF**: Na použitie v aplikáciách GIS

### Pre analýzu GIS

**Odporúčaný pracovný postup:**

1. **Načítajte do QGIS, ArcGIS alebo podobného programu**
2. **Použite 16-bitové obrazy odrazivosti TIFF** pre multibandovú analýzu
3. **Použite indexové obrazy** (NDVI, NDRE) ako pripravené vrstvy vegetácie
4. **Rastrová kalkulačka**: Kombinujte pásma pre vlastnú analýzu
5. **Export**: Vytvorte klasifikačné mapy, detekciu zmien, mapy zdravotného stavu vegetácie

### Pre priamu analýzu / reporting

**Odporúčaný pracovný postup:**

1. **Použite indexové obrázky s farbami LUT** pre vizuálne reporty
2. **Extrahujte štatistiky**: Priemerná hodnota NDVI na pole/parcelu
3. **Časové rady**: Porovnajte indexy v rámci viacerých relácií
4. **Generovanie správ**: Zahrňte mapy, štatistiky a vizualizácie

***

## Archivácia a zálohovanie

### Odporúčaná stratégia zálohovania

**Čo uložiť:**

* ✅ **Pôvodné obrázky RAW/JPG** – archivujte na samostatnom disku/v cloude
* ✅ **Spracované výstupy** – uchovajte kalibrované obrázky a indexy
* ✅ **Súbor projektu** – obsahuje všetky nastavenia pre opätovné spracovanie, ak je to potrebné
* ✅ **Protokol ladenia** – dokumentuje podrobnosti spracovania
* ✅ **Obrázky kalibračného cieľa** – na overenie a opätovné spracovanie

**Odporúčania pre ukladanie:**

* **Okamžitá záloha**: Externý pevný disk
* **Dlhodobý archív**: Ukladanie v cloude (Google Drive, Dropbox atď.)
* **Kritické údaje**: Uložte 2–3 kópie na rôznych miestach

***

## Ďalšie spracovania

### Opätovné použitie nastavení projektu

Ak budete v budúcnosti spracovávať podobné súbory údajov:

1. **Uložte šablónu projektu** (ak ste tak ešte neurobili)
2. **Vytvorte nový projekt** pomocou uloženej šablóny
3. **Importujte nové obrázky**
4. **Spracujte** s identickými nastaveniami pre konzistentnosť

### Hromadné spracovanie viacerých relácií

Pre viacero relácií/datasets:

**Možnosť 1: GUI – viacero projektov**

* Vytvorte samostatný projekt pre každú reláciu
* Použite konzistentné nastavenia šablóny
* Spracujte jeden po druhom

**Možnosť 2: Chloros CLI (len Chloros+)**

* Automatizujte hromadné spracovanie
* Spracujte viacero priečinkov pomocou skriptov
* Pozrite si [CLI dokumentáciu](../CLI.md)

**Možnosť 3: Python SDK (len Chloros+)**

* Programové ovládanie
* Integrácia s analytickými potrubiami
* Pozrite si [API dokumentáciu](../api-python-sdk.md)

***

## Riešenie problémov s následným spracovaním

### Opätovné spracovanie s inými nastaveniami

Ak výsledky nie sú uspokojivé:

1. Zachovajte pôvodné obrázky (nikdy ich nemazajte)
2. Otvorte ten istý projekt v Chloros
3. Upravte nastavenia v paneli Nastavenia projektu
4. Spracujte znovu – výstupy prepíšu predchádzajúce výsledky

### Spracovanie podmnožiny obrázkov

Ak chcete opätovne spracovať len konkrétne obrázky:

1. Vytvorte nový projekt
2. Importujte len obrázky, ktoré potrebujú opätovné spracovanie
3. Použite rovnakú šablónu nastavení
4. Spracujte menšiu dátovú sadu

### Pomoc

Ak narazíte na problémy:

* 📧 **E-mail**: info@mapir.camera (pridajte protokol ladenia)
* 🌐 **Podpora**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Často kladené otázky](../faq.md)
* 📖 **Dokumentácia**: [Chloros Manuál](../)

***

## Zhrnutie: Kompletný pracovný postup

Teraz ste dokončili celý pracovný postup spracovania Chloros:

1. ✅ **Vytvorený projekt** – pozrite si [Projekty](../projects.md)
2. ✅ **Pridané súbory** – pozrite si [Pridávanie súborov](adding-files-to-a-project.md)
3. ✅ **Upravili ste nastavenia** – pozrite si [Úprava nastavení projektu](adjusting-project-settings.md)
4. ✅ **Označili ste ciele** – pozrite si [Výber cieľových obrázkov](choosing-target-images.md)
5. ✅ **Spustené spracovanie** – pozri [Spustenie spracovania](starting-the-processing.md)
6. ✅ **Sledovaný pokrok** – pozri [Sledovanie spracovania](monitoring-the-processing.md)
7. ✅ **Skontrolované výsledky** – táto stránka

**Vaše kalibrované multispektrálne obrázky s korekciou odrazivosti sú pripravené na analýzu!**

***

## Ďalšie zdroje

### Pokročilé funkcie

* [**Prehliadač obrázkov**](../image-viewer-gui/opening-an-image-full-screen.md) – interaktívna vizualizácia a analýza
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – Testovanie vlastného indexu
* [**Vzorec multispektrálneho indexu**](../project-settings/multispectral-index-formulas.md) – Kompletný referenčný index

### Automatizácia a integrácia

* [**CLI Dokumentácia**](../CLI.md) – Hromadné spracovanie z príkazového riadku
* [**Python SDK**](../api-python-sdk.md) – Programová automatizácia
* [**Chloros+ Funkcie**](../#chloros) – Pokročilé možnosti spracovania

### Podpora a vzdelávanie

* [**Často kladené otázky**](../faq.md) – Odpovede na bežné otázky
* [**Kalibračné terče**](../calibration-targets.md) – Porozumenie kalibrácii odrazivosti
* [**Podporované kamery**](../supported-cameras.md) – Kompatibilný hardvér

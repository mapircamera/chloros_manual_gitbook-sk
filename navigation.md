# Grafické rozhranie: Navigácia

Pri prvom spustení programov Chloros a Chloros (prehliadač) sa spustí ich backend. Akonáhle bude pripravený, zobrazí sa ikona hlavného menu v ľavom hornom rohu <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Zľava doprava obsahuje horná lišta:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Hlavné menu

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Z hlavného menu môžete:

* **Nový projekt** — vytvoriť nový projekt
* **Otvoriť projekt** — otvoriť existujúci projekt
* **Otvoriť priečinok projektu** — otvoriť priečinok projektu vo vašom prehliadači súborov
* **Pridať súbory** — pridať jednotlivé obrazové súbory do aktuálneho projektu _(viditeľné po otvorení projektu)_
* **Pridať zložku** — pridať zložku s obrázkami do aktuálneho projektu _(viditeľné po otvorení projektu)_
* **Spustiť spracovanie / Zastaviť spracovanie** — spustiť alebo zastaviť proces spracovania obrázkov _(aktivované po pridaní súborov)_

{% hint style="info" %}
**Iba Windows**: Grafické používateľské rozhranie Chloros Desktop je k dispozícii v Windows. Používatelia Linux by mali vidieť [CLI](CLI.md) a [Python SDK](api-python-sdk.md) pre spracovanie bez grafického rozhrania.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Tlačidlo Prehrať/Spustiť

Ak je táto funkcia povolená, tlačidlo na spustenie spracovania spustí proces spracovania obrazu.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Indikátor priebehu <img src=".gitbook/assets/image (5).png" alt="" data-size="line">V bezplatnom režime Chloros, ktorý spracováva všetky súbory postupne, ukazovateľ priebehu zobrazuje 2 fázy: Detekcia cieľa a Spracovanie.

V platenom režime s licenciou Chloros+, ktorý spracováva všetky súbory súčasne, ukazovateľ priebehu zobrazuje 4 fázy: Detekcia, Analýza, Kalibrácia, Export. Ak umiestnite kurzor myši nad indikátor priebehu Chloros+, rozbalí sa rozšírený panel so 4 indikátormi priebehu, aby ste mohli sledovať priebeh. Kliknutím na horný indikátor priebehu zamrazíte rozbalený panel, ďalším kliknutím ho opäť odblokujete.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Bočné menu

Ľavé bočné menu obsahuje rôzne ikony, s ktorými môžete pracovať:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Nastavenia projektu](project-settings/project-settings.md)

Karta Nastavenia projektu vám umožňuje upraviť globálne nastavenia projektu a nastavenia spracovania projektu. Upravte ich pred začatím spracovania vašich súborov.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Prehliadač súborov

Pridávajte súbory/zložky a odstraňujte súbory z projektu. Duplicitné súbory sa ignorujú. Zaškrtnite políčko v stĺpci cieľ pre akýkoľvek cieľový obrázok a spracovanie bude hľadať ciele len v zaškrtnutých obrázkoch, čo výrazne urýchli čas spracovania. Pomocou prepínača Obrázok/Metadáta môžete prepínať medzi zobrazením mriežky miniatúr vybraných obrázkov a podrobnou tabuľkou metadát.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Prehliadač obrázkov](image-viewer-gui/opening-an-image-full-screen.md)

Po kliknutí na obrázok v hlavnom prehliadači obrázkov sa obrázok otvorí na celej obrazovke na karte Prehliadač obrázkov.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Mapa](image-viewer-gui/map-markers.md)

Prezerajte svoje obrázky na interaktívnej 2D mape na základe ich GPS súradníc. Podporuje poskytovateľov dlaždíc Google Maps a ESRI, pričom automaticky vyberá najlepšiu službu pre vašu polohu. Naveďte kurzor na značky, aby ste videli náhľady miniatúr obrázkov.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Log ladiaceho režimu

V prípade problémov skontrolujte log ladiaceho režimu. Skopírujte/stiahnite log a pošlite ho na [MAPIR podporu](https://www.mapir.camera/community/contact) s požiadavkou o pomoc.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Prihlásenie používateľa](chloros+-login.md)

Bočný panel pre prihlásenie používateľa vám umožňuje prihlásiť sa do vášho účtu Chloros+ a odomknúť pokročilé funkcie. Môžete si tiež zobraziť aktuálnu verziu aplikácie, ako aj nastaviť jazyk zobrazeného textu v grafickom rozhraní Chloros a CLI.

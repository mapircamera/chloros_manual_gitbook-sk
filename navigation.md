# Grafické rozhranie: Navigácia

Pri prvom spustení programu Chloros sa spustí jeho spracovateľský backend. Akonáhle je backend pripravený, v ľavom hornom rohu sa zobrazí ikona hlavného menu <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> a v ľavom bočnom paneli sa odomknú karty „Kamery“ a „Svetelné senzory“ (do tej chvíle sú sivé).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Horná lišta obsahuje zľava doprava:

### Hlavné menu programu „<img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">“

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Z hlavného menu môžete:

* **Nový projekt**— vytvoriť nový projekt. Ak máte uložené šablóny projektov, zobrazí sa roletové menu**Vybrať šablónu**, vďaka čomu nový projekt začne s nastaveniami šablóny.
* **Otvoriť projekt**— otvoriť existujúci projekt. Zoznam obsahuje tlačidlo**Otvoriť priečinok projektu**, ktoré otvorí priečinok s projektmi vo vašom prehliadači súborov.
* **Duplikovať projekt** — skopírujte aktuálne otvorený projekt pod novým názvom (navrhuje sa voľný názov, napr. „MôjProjekt (2)“) a otvorte kópiu. _(viditeľné po otvorení projektu)_
* **Pridať súbory** — pridá jednotlivé obrazové súbory do aktuálneho projektu _(viditeľné po otvorení projektu)_
* **Pridať zložku** — pridá jednu alebo viac zložiek s obrázkami do aktuálneho projektu _(viditeľné po otvorení projektu)_
* **Spustiť spracovanie / Zastaviť spracovanie** — spustí alebo zastaví proces spracovania obrázkov _(aktívne po pridaní súborov)_
* **Pripojiť sa k kamere** — prejde na [kartu Kamery](lattice/) na pripojenie kamery alebo sústavy kamier LATTICE. Funguje aj bez otvoreného projektu.
* **Pripojiť sa k svetelnému senzoru** — prejsť na [kartu Svetelné senzory](daq/) a pripojiť svetelný senzor DAQ. Funguje aj bez otvoreného projektu.

{% hint style="info" %}
**Iba Windows**: Grafické používateľské rozhranie Chloros Desktop je k dispozícii na Windows. Používatelia modelu Linux by si mali prečítať dokumentáciu k modelom [CLI](CLI.md) a [Python SDK](api-python-sdk.md) týkajúcu sa spracovania bez grafického rozhrania.
{% endhint %}

### Tlačidlo „Play/Start“ (<img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

)

Ak je táto funkcia zapnutá, tlačidlo na spustenie spracovania naštartuje proces spracovania obrazu.

### Ukazovateľ priebehu v režime „<img src=".gitbook/assets/image (4).png" alt="" data-size="line">

“ (<img src=".gitbook/assets/image (5).png" alt="" data-size="line">

) V bezplatnom režime Chloros, ktorý spracováva všetky súbory postupne, ukazovateľ priebehu zobrazuje 2 fázy: Detekcia cieľa a Spracovanie.

V platenom režime s licenciou Chloros+, ktorý spracováva všetky súbory súčasne, ukazovateľ priebehu zobrazuje 4 fázy: detekcia, analýza, kalibrácia a export. Ak umiestnite kurzor myši nad indikátor priebehu Chloros+, rozbalí sa rozšírený panel so 4 fázami priebehu, vďaka čomu môžete sledovať priebeh. Kliknutím na hornú časť indikátora priebehu panel zafixujete, ďalším kliknutím ho opäť uvoľníte.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Bočné menu

Ľavé bočné menu obsahuje rôzne ikony na interakciu, v tomto poradí zhora nadol:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Nastavenia projektu](project-settings/project-settings.md)

Karta Nastavenia projektu vám umožňuje upraviť globálne nastavenia projektu a nastavenia spracovania projektu. Tieto nastavenia upravte pred začatím spracovania súborov.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Prehliadač súborov

Pridávajte súbory/zložky a odstraňujte súbory z projektu. Duplicitné súbory sa ignorujú. Zaškrtnite políčko v stĺpci „cieľ“ pri akomkoľvek cieľovom obrázku a spracovanie bude hľadať ciele iba v zaškrtnutých obrázkoch, čo výrazne urýchli čas spracovania. Pomocou prepínača Obrázok/Metadáta môžete prepínať medzi zobrazením mriežky náhľadov vybraných obrázkov a podrobnou tabuľkou metadát.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Prehliadač obrázkov](image-viewer-gui/opening-an-image-full-screen.md)

Po kliknutí na obrázok v hlavnom prehliadači obrázkov sa obrázok otvorí na celej obrazovke na karte „Image Viewer“.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Prehliadač máp](image-viewer-gui/map-markers.md)

Prezerajte si svoje obrázky na interaktívnej 2D mape na základe ich GPS súradníc. Podporuje poskytovateľov dlaždíc Google Maps a ESRI a automaticky vyberá najlepšiu službu pre vašu polohu. Prejdite kurzorom nad značkami a zobrazte náhľady miniatúr obrázkov.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kamery](lattice/)

Pripojte a ovládajte kamery LATTICE v reálnom čase – po jednej alebo ako synchronizované sústavy viacerých kamier. Na tejto karte sa zobrazujú dlaždice s živým náhľadom s prekrývacími vrstvami a histogramami, nastavenia pre jednotlivé kamery a sústavy, ako aj nastavenia snímania, ktoré určujú, ktoré kamery a typy exportu sa použijú pri funkcii „Capture All“. Funkcia bude dostupná, akonáhle bude pripravený backend; kompletný návod nájdete v [sekcii LATTICE](lattice/).

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Svetelné senzory](daq/)

Pripojte svetelné senzory DAQ — DAQ-U (USB), DAQ-M (Bluetooth) a DAQ-E (Ethernet) — a sledujte ich živé kalibrované spektrálne grafy v jednotkách W/m²/nm. Odtiaľto môžete zaznamenávať súbory `.daq` do otvoreného projektu, premenovať senzory, vybrať profily korekcie kapacity a aktualizovať firmvér DAQ-E. K dispozícii, akonáhle bude backend pripravený; kompletný návod nájdete v [sekcii DAQ](daq/).

#### Ladiaci protokol <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line">

V prípade problémov skontrolujte protokol, či neobsahuje ladiace výpisy. Skopírujte alebo stiahnite protokol a pošlite ho na [podporu MAPIR](https://www.mapir.camera/community/contact) s žiadosťou o pomoc.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Prihlásenie používateľa](chloros+-login.md)

Bočný panel pre prihlásenie používateľa vám umožňuje prihlásiť sa do vášho účtu Chloros+ a odomknúť tak pokročilé funkcie. Môžete si tiež zobraziť aktuálnu verziu aplikácie, ako aj nastaviť jazyk zobrazeného textu v grafickom rozhraní Chloros a v CLI.

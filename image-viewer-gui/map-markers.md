# Značky na mape

Na karte Mapa sa vaše snímky zobrazujú na interaktívnej 2D mape na základe ich GPS súradníc. To vám poskytuje geografický prehľad o vašej snímkovacej relácii a pomáha vizualizovať priestorové pokrytie. Je to užitočné aj pri prvom importe snímok, keď chcete rýchlo odstrániť všetky snímky, ktoré nepotrebujete spracovať.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Prístup k záložke Mapa

1. Otvorte alebo vytvorte projekt v Chloros
2. Importujte snímky, ktoré obsahujú GPS metadáta
3. Kliknite na záložku **Mapa** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> v ľavom bočnom paneli
4. Na mape sa zobrazia značky v GPS polohe každého snímku

{% hint style="info" %}
**Vyžaduje sa GPS**: Na mape sa zobrazia len obrázky s vloženými GPS súradnicami v ich EXIF metadátach. Uistite sa, že máte počas snímania zapnuté GPS vo vašom fotoaparáte.
{% endhint %}

***

## Úprava obrázkov z karty Mapa

Karta **Mapa**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> má rovnaké tlačidlá na pridávanie  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  a odstránenia  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  ako karta [**Prehliadač súborov**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Zobrazuje tiež rovnaký zoznam projektových súborov, ale s odlišnými nadpismi stĺpcov:

### Názov súboru

* Pôvodný názov súboru z fotoaparátu
* Zachováva konvenciu pomenovania fotoaparátu (napr. IMG\_0001.RAW)

### Zemepisná šírka

* Zemepisná šírka snímky

### Zemepisná dĺžka

* Zemepisná dĺžka snímky

### Nadmorská výška

* Nadmorská výška snímky

{% hint style="info" %}
Kliknutím na nadpisy stĺpcov tabuľky sa tiež zoradia údaje v riadkoch
{% endhint %}

***

## Značky snímok

Každá snímka s údajmi GPS je na mape znázornená značkou:

### Zobrazenie značiek

* Značky označujú presné GPS súradnice, kde bol každý obrázok nasnímaný
* Značky v skupinách sa môžu zoskupiť pri oddialení
* Priblížte si mapu, aby ste videli polohy jednotlivých obrázkov

{% hint style="success" %}
SUPER-ZOOM: Keď dosiahnete maximálnu úroveň priblíženia od poskytovateľa mapových dlaždíc, dlaždica sa pri ďalšom priblížení zväčší, čo vám umožní vidieť značky, ktoré sú blízko pri sebe.
{% endhint %}

### Náhľad pri prechádzaní kurzorom

* **Prejdite kurzorom myši** nad akoukoľvek značkou, aby ste videli náhľad miniatúry daného obrázku
* To umožňuje rýchlu vizuálnu identifikáciu bez opustenia zobrazenia mapy
* Užitočné na vyhľadávanie konkrétnych obrázkov v rámci rozsiahlej snímkovacej relácie

***

## Poskytovatelia mapových dlaždíc

{% hint style="success" %}
**Automatický výber**: Chloros automaticky vyberie službu dlaždíc, ktorá poskytuje najlepšiu úroveň priblíženia pre vašu aktuálnu polohu na mape. V prípade potreby môžete ručne prepínať medzi poskytovateľmi.
{% endhint %}

Karta Mapa podporuje dvoch poskytovateľov dlaždíc pre snímky pozadia mapy:

### Google Maps

* Štandardné satelitné a mapové snímky od Google
* Najvhodnejšie pre všeobecné celosvetové pokrytie

### ESRI

* Satelitné a letecké snímky z ESRI ArcGIS
* V určitých regiónoch často poskytuje snímky s vyšším rozlíšením

***

## Typy mapových dlaždíc

Môžete si vybrať typ mapovej vrstvy (zľava doprava):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Terén

Zobrazuje výškové profily a mapové dlaždice s detailmi (cesty atď.)

### Mapa

Zobrazuje štandardné (nižšia šírka pásma) mapové dlaždice s detailmi (cesty atď.)

### Satelit

Zobrazuje detailné (vyššia šírka pásma) satelitné mapové dlaždice

### Hybrid

Zobrazuje satelitné mapové dlaždice s pridanými detailmi (cesty atď.)

***

## Navigácia na mape

### Ovládacie prvky priblíženia

* **Priblíženie/oddialenie**: Použite koliesko myši alebo tlačidlá priblíženia
* **Celá obrazovka**: Zobraziť mapu na celej obrazovke

### Ovládacie prvky posúvania

* **Posúvanie**: Kliknite a ťahajte pre pohyb po mape***

## Príklady použitia

### Vizualizácia letovej dráhy

* Zobrazenie oblasti pokrytia snímok z dronov
* Identifikácia medzier v pokrytí snímok
* Overenie vykonania letovej dráhy

### Kontrola pozemného prieskumu

* Zobrazenie priestorového rozloženia pozemných snímok
* Lokalizácia kalibračných cieľových snímok vo vzťahu k oblasti prieskumu
* Plánovanie ďalších miest snímania

### Kontrola kvality

* Rýchlo identifikujte snímky zachytené na neočakávaných miestach
* Overte presnosť GPS v rámci súboru údajov
* Porovnajte polohy snímok s poznámkami z terénu

***

## Riešenie problémov

### Nezobrazujú sa značky

**Možné príčiny:**

* Snímky neobsahujú metadáta GPS
* GPS bolo počas snímania na kamere vypnuté
* EXIF údaje boli odstránené externým softvérom

**Riešenie**: Overte, či je GPS na fotoaparáte zapnuté, a znovu naimportujte pôvodné súbory

### Značky na nesprávnom mieste

**Možné príčiny:**

* GPS fotoaparátu malo slabý signál zo satelitu
* Posun GPS počas snímania

**Riešenie**: Ide zvyčajne o problém v čase snímania; zvážte použitie PPK/RTK GPS pre presné aplikácie

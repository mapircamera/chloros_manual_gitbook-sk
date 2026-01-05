# Značky na mape

Karta Mapa zobrazuje vaše obrázky na interaktívnej 2D mape na základe ich GPS súradníc. Poskytuje geografický prehľad vašej snímkovej relácie a pomáha vám vizualizovať priestorové pokrytie. Je tiež užitočná pri prvom importe obrázkov, aby ste mohli rýchlo odstrániť všetky obrázky, ktoré nepotrebujete spracovať.

## Prístup k karte

1. Otvorte alebo vytvorte projekt v Chloros.
2. Importujte obrázky, ktoré obsahujú GPS metadáta.
3. Kliknite na kartu **Mapa** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> v ľavom bočnom paneli.
4. Na mape sa zobrazia značky v GPS polohe každého obrázka.

{% hint style=&quot;info&quot; %}
**GPS nevyhnutné**: Na mape sa zobrazia len obrázky s vloženými GPS súradnicami v metadátach EXIF. Uistite sa, že máte počas snímania zapnuté GPS vo fotoaparáte.
{% endhint %}

***

## Úprava obrázkov z karty Mapa

Karta **Mapa**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> má rovnaké funkcie pridávania  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  a odstránenie  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  ako karta [**Prehliadač súborov**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Zobrazuje tiež rovnaký zoznam projektových súborov, ale s inými názvami stĺpcov:

### Názov súboru

* Pôvodný názov súboru z fotoaparátu.
* Zachováva konvenciu názvov fotoaparátu (napr. IMG\_0001.RAW).

### Zemepisná šírka

* Zemepisná šírka obrázku.

### Zemepisná dĺžka

* Zemepisná dĺžka obrázku.

### Nadmorská výška

* Nadmorská výška obrázku

{% hint style=&quot;info&quot; %}
Kliknutím na záhlavia stĺpcov tabuľky sa tiež zoradia údaje v riadkoch.
{% endhint %}

***

## Značky obrázkov

Každý obrázok s údajmi GPS je na mape označený značkou:

### Zobrazenie značiek

* Značky označujú presné GPS súradnice miesta, kde bol každý obrázok zachytený.
* Zoskupené značky sa môžu pri oddialení zobrazenia zoskupiť.
* Zväčšením zobrazenia môžete vidieť polohy jednotlivých obrázkov.

{% hint style=&quot;success&quot; %}
SUPER-ZOOM: Keď dosiahnete maximálnu úroveň priblíženia od poskytovateľa mapových dlaždíc, dlaždica sa pri ďalšom priblížení zväčší, čo vám umožní vidieť značky, ktoré sú blízko seba.
{% endhint %}

### Náhľad pri prechádzaní kurzorom

* **Naveďte kurzor myši** na ľubovoľnú značku, aby ste videli náhľad miniatúry daného obrázku.
* To umožňuje rýchlu vizuálnu identifikáciu bez opustenia zobrazenia mapy.
* Užitočné pre vyhľadávanie konkrétnych obrázkov v rámci veľkej snímky.

***

## Poskytovatelia mapových dlaždíc

{% hint style=&quot;success&quot; %}
**Automatický výber**: Chloros automaticky vyberie službu dlaždíc, ktorá poskytuje najlepšiu úroveň priblíženia pre vašu aktuálnu polohu na mape. Ak chcete, môžete ručne prepínať medzi poskytovateľmi.
{% endhint %}

Karta Mapa podporuje dvoch poskytovateľov dlaždíc pre pozadie mapy:

### Google Maps

* Štandardné satelitné a mapové snímky od Google
* Najlepšie pre všeobecné celosvetové pokrytie

### ESRI

* Satelitné a letecké snímky od ESRI ArcGIS
* Často poskytuje snímky s vyšším rozlíšením v určitých regiónoch

***

## Typy dlaždíc mapy

Môžete si vybrať typ vrstvy mapy (zľava doprava):

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

* **Priblíženie/oddialenie**: Použite koliesko myši alebo tlačidlá priblíženia.
* **Celá obrazovka**: Zobrazenie mapy na celej obrazovke.

### Ovládacie prvky posunu

* **Posun**: Kliknutím a ťahaním sa pohybujete po mape.***

## Prípady použitia

### Vizualizácia letovej dráhy

* Zobrazenie pokrytia oblasti snímaním dronom
* Identifikácia medzier v pokrytí obrazom
* Overenie vykonania letovej dráhy

### Kontrola pozemného prieskumu

* Zobrazenie priestorového rozloženia pozemných snímok
* Lokalizácia kalibračných cieľových obrazov vo vzťahu k prieskumnej oblasti
* Plánovanie ďalších miest snímania

### Kontrola kvality

* Rýchlo identifikujte snímky zachytené na neočakávaných miestach.
* Overte presnosť GPS v celom súbore údajov.
* Porovnajte polohy snímok s poznámkami z terénu.

***

## Riešenie problémov

### Nezobrazujú sa značky

**Možné príčiny:**

* Snímky neobsahujú metadáta GPS.
* GPS bolo počas snímania na fotoaparáte vypnuté.
* EXIF údaje boli odstránené externým softvérom.

**Riešenie**: Overte, či je GPS vo fotoaparáte zapnuté, a znovu naimportujte pôvodné súbory.

### Značky na nesprávnom mieste

**Možné príčiny:**

* GPS fotoaparátu malo slabý signál zo satelitu.
* GPS sa počas snímania posunulo.

**Riešenie**: Ide zvyčajne o problém so snímaním. Zvážte použitie PPK/RTK GPS pre presné aplikácie.

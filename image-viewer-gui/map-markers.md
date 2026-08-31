# Značky na mape

Karta „Mapa“ znázorňuje vaše snímky na interaktívnej 2D mape na základe ich GPS súradníc. Poskytuje vám geografický prehľad o snímkovacej relácii a je najrýchlejším spôsobom, hneď po importe, ako odstrániť snímky, ktoré nechcete spracovávať.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Prístup na kartu „Mapa“

1. Otvorte alebo vytvorte projekt v programe Chloros
2. Importujte snímky, ktoré obsahujú metadáta GPS
3. Kliknite na kartu **Mapa** <img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> v ľavom bočnom paneli
4. Na mape sa zobrazí značka v mieste GPS každej snímky

{% hint style="info" %}
**Vyžaduje sa GPS**: na mape sa zobrazujú len obrázky, ktoré majú v EXIF metadátach GPS súradnice. Obrázok bez súradníc zostáva v projekte a spracováva sa normálne — jednoducho nemá žiadnu značku.
{% endhint %}

***

## Úprava obrázkov na karte Mapa

Karta **Mapa**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> obsahuje rovnaké tlačidlá na pridanie <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> a odstránenie <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> súborov ako karta [**Prehliadač súborov**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Zobrazuje rovnaký zoznam projektových súborov s geografickými stĺpcami:

| Stĺpec        | Obsah                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Názov**      | Názov súboru tak, ako bol zhotovený fotoaparátom                             |
| **Zemepisná šírka** | Desatinné stupne, šesť desatinných miest                                |
| **Dĺžka** | Desatinné stupne, šesť desatinných miest                                |
| **Nadmorská výška** | Metre, jedno desatinné miesto — `-`, ak snímka neobsahuje údaje o nadmorskej výške |

{% hint style="info" %}
Kliknutím na akýkoľvek nadpis stĺpca ho zoradíte podľa neho; ďalším kliknutím poradie obrátite.
{% endhint %}

{% hint style="warning" %}
**Nadmorská výška je výška nad hladinou mora, nie výška nad zemou.** Hodnota pochádza z EXIF značky snímky `GPSAltitude`, ktorá sa vzťahuje na priemernú hladinu mora. Nejedná sa o letovú výšku nad terénom a Chloros z nej nevypočíta vzdialenosť vzorky od zemského povrchu – nad poľom vo výške 300 m nad hladinou mora zaznamená dron vo výške 100 m nad zemským povrchom (AGL) približne 400 m. Tento stĺpec použite na identifikáciu výnimočných hodnôt a potvrdenie konzistentnej letovej výšky, nie ako meranie AGL.
{% endhint %}

***

## Značky snímok

Každá snímka s údajmi GPS dostane značku na svojich súradniciach.

### Zobrazenie značiek

* Značky sa nachádzajú na presných súradniciach zaznamenaných pre každý záber
* Značky, ktoré sú blízko pri sebe, sa pri oddialení môžu vizuálne prekrývať – priblížením ich od seba oddelíte
* Vybrané a zvýraznené značky sa zobrazujú nad ostatnými

### Náhľad pri nabehnutí kurzora

* **Nabehnite kurzorom** na akúkoľvek značku, aby sa zobrazil náhľad daného obrázku s názvom súboru
* **Kliknutím**na značku vyberiete obrázok a**pripnete** vyskakovacie okno – zostane otvorené, kým nekliknete inde. Pokiaľ je vyskakovacie okno pripnuté, prechod kurzorom nad inými značkami ho nezatvorí
* Ide o rýchly spôsob, ako nájsť jeden konkrétny snímok vo veľkej relácii bez opustenia mapy

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Karta Mapa zobrazuje všetky obrázky s geoznačkami v projekte</p></figcaption></figure>### Super-zoom

{% hint style="success" %}
**SUPER-ZOOM**: keď dosiahnete maximálne zväčšenie, pre ktoré poskytovateľ dlaždíc má k dispozícii snímky, ďalšie zväčšovanie dlaždice zväčšuje namiesto toho, aby sa zastavilo, takže môžete od seba odlíšiť značky, ktoré sa nachádzajú takmer na sebe.
{% endhint %}

* Super-zoom sa aktivuje len vtedy, keď ste **na** maximálnom priblížení poskytovateľa pre danú lokalitu a dlaždice sa už načítali. Pod touto hranicou funguje priblíženie normálne
* Rozsah je **1× až 32×** nad maximálnym priblížením samotného poskytovateľa
* Indikátor v rohu zobrazuje aktuálny superzoom ako percento a tlačidlo **×** vedľa neho vás jedným kliknutím vráti k bežnému priblíženiu
* Oddialenie sa vždy prenáša priamo na mapu, takže sa nikdy nemôžete zaseknúť v režime superzoomu
* Zväčšovanie a posúvanie v režime super-zoomu prenáša výsledný posun späť na mapu, takže oblasť mimo stredu, do ktorej ste sa presunuli, naďalej žiada o dlaždice namiesto toho, aby zostala prázdna
* Značky sú vykresľované ako vektorové prvky, nie ako rastrované, takže zostávajú ostré na každej úrovni super-zoomu

***

## Poskytovatelia dlaždíc mapy

{% hint style="success" %}
**Automatický výber**: Chloros vyberá službu dlaždíc, ktorá ponúka najlepšiu úroveň priblíženia pre danú polohu vašich obrázkov. Kedykoľvek môžete prejsť na inú službu ručne.
{% endhint %}

| Poskytovateľ        | Poznámky                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Široké celosvetové pokrytie; podporuje všetky štyri typy dlaždíc                                                                                                            |
| **Esri ArcGIS**| V určitých regiónoch často letecké snímky s vyšším rozlíšením. Typ dlaždíc**Terrain** nie je pre Esri k dispozícii a jeho tlačidlo je deaktivované, pokiaľ je zvolený Esri |***

## Typy mapových dlaždíc

Vyberte typ mapovej vrstvy pomocou tlačidiel (zľava doprava):

![](&lt;../.gitbook/assets/image (14).png&gt;)

| Typ                 | Zobrazuje                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Terén**          | Tieňovanie výškových rozdielov s detailmi mapy (cesty, popisy). Iba Google       |
| **Mapa**              | Štandardné dlaždice uličnej mapy — možnosť s najnižšou šírkou pásma              |
| **Satelit**        | Podrobné satelitné snímky, bez popisov — možnosť s najvyššou šírkou pásma |
| **Hybrid** (predvolené) | Satelitné snímky s nakreslenými cestami a popismi                |

Karta Mapa sa otvorí v režime **Hybrid**. Váš výber sa prenesie aj pri zmene poskytovateľa, ak to poskytovateľ podporuje.***

## Navigácia na mape

* **Priblíženie**: koliesko myši alebo tlačidlá priblíženia na mape
* **Posun**: kliknutím a ťahaním
* **Celá obrazovka**: ovládací prvok „Celá obrazovka“ zväčší mapu na celé okno***

## Príklady použitia

### Kontrola trasy letu

* Prehľad o oblasti pokrytia letu dronu na prvý pohľad
* Zistenie medzier, kde chýba záznam
* Potvrdenie, že let prebehol podľa plánu

### Kontrola pozemného prieskumu

* Prehľad o rozložení pozemných snímok
* Lokalizácia kalibračných cieľových rámcov vo vzťahu k prieskumnej oblasti
* Rozhodnutie, kde sú potrebné ďalšie snímky

### Kontrola kvality

* Nájdite snímky zachytené na neočakávaných miestach a odstráňte ich pred spracovaním
* Zoradiť podľa nadmorskej výšky, aby ste odhalili snímku zachytenú v nesprávnej výške alebo takú, kde bola slabá poloha GPS
* Porovnajte polohy snímok s terénnymi poznámkami

***

## Riešenie problémov

### Nezobrazujú sa žiadne značky

**Možné príčiny**

* Snímky neobsahujú metadáta GPS
* Počas snímania bolo na fotoaparáte vypnuté GPS
* EXIF údaje boli pred importom odstránené iným softvérom

**Čo robiť**: skontrolujte, či je v fotoaparáte zapnutá funkcia GPS, a znovu naimportujte pôvodné súbory. Môžete skontrolovať, či konkrétny súbor obsahuje súradnice, tak, že ho vyhľadáte v tabuľke súborov na karte Mapa — snímka bez súradníc tam nemá žiadny riadok.

### Značky sú na nesprávnom mieste

**Možné príčiny**: slabé satelitné signály v čase snímania alebo odchýlka GPS počas relácie.**Čo robiť**: ide o problém súvisiaci s časom snímania, ktorý program Chloros nedokáže dodatočne opraviť. Pre presnú prácu použite pracovný postup s PPK/RTK GPS – pozrite si nastavenie**Použiť PPK korekcie** v [Nastaveniach projektu](../project-settings/project-settings.md).

### Mapa je prázdna alebo sa prestali načítať dlaždice

Poskytovatelia dlaždíc sú online služby. Ak sa dlaždice prestanú načítať, skontrolujte sieťové pripojenie zariadenia a potom skúste zmeniť poskytovateľa. Ak ste boli výrazne zväčšení, stlačte tlačidlo **×** na resetovanie, aby ste sa vrátili na normálnu úroveň zväčšenia, a nechajte mapu znovu požiadať o dlaždice.***

## Súvisiace stránky

* [**Mriežka obrázkov**](image-grid.md) — rovnaká sada obrázkov ako miniatúry
* [**Otvorenie obrázku na celú obrazovku**](opening-an-image-full-screen.md) — podrobné prezretie jedného obrázku
* [**Pridávanie súborov do projektu**](../processing-images-gui/adding-files-to-a-project.md) — tlačidlá na pridávanie/odstraňovanie súborov, ktoré sa nachádzajú aj na tejto karte

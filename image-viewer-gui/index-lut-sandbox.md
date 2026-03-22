# Sandbox pre indexy/LUT

Sandbox pre indexy/LUT je interaktívny pracovný priestor v rámci prehliadača snímok Chloros, ktorý vám umožňuje experimentovať s výpočtami multispektrálnych indexov a farebnými vizualizáciami v reálnom čase. Tento výkonný nástroj vám pomáha testovať rôzne indexy, spresňovať rozsahy hodnôt a vytvárať vizualizácie pripravené na publikovanie bez nutnosti opätovného spracovania celého súboru údajov.

## Čo je Index/LUT Sandbox?

### Účel

Sandbox poskytuje:

* **Výpočet indexu v reálnom čase** – Okamžité použitie akéhokoľvek vegetačného indexu
* **Interaktívne nastavenie LUT** – Jemné doladenie farebných prechodov a rozsahov
* **Optimalizácia pracovného postupu** – Určenie najlepších nastavení pred hromadným spracovaním

### Sandbox vs. spracovanie projektu

**Index/LUT Sandbox (interaktívny):**

* Jeden obrázok naraz
* Okamžitá spätná väzba
* Experimentálny a iteratívny
* Žiadne trvalé zmeny súborov
* Ideálny na preskúmavanie a testovanie

**Spracovanie projektu (hromadné):**

* Celý súbor údajov naraz
* Predkonfigurované nastavenia
* Trvalé výstupné súbory
* Časovo náročné
* Najlepšie, keď sú nastavenia finálne

{% hint style="success" %}
**Najlepší pracovný postup**: Použite Sandbox na experimentovanie a nájdenie optimálnych nastavení indexu a LUT, potom tieto nastavenia aplikujte počas spracovania projektu na celý súbor údajov.
{% endhint %}

***

## Práca s indexom/LUT Sandboxom

### Porozumenie vopred vypočítaným indexom

V Chloros je možné indexy použiť počas spracovania projektu. Na určenie toho, ktoré nastavenia indexu a LUT chcete použiť pri exportoch, je najjednoduchšie použiť prehliadač obrázkov Sandbox.

Sandbox vám umožňuje:

* **Použiť nové indexy a farebné prechody (LUT)** na vizualizáciu údajov
* **Interaktívne upravovať nastavenia vizualizácie*** **Zobraziť** už vypočítané indexové obrázky
* **Skontrolovať** hodnoty pixelov na všetkých úrovniach priblíženia

### Otvorenie sandboxu

Sandbox Index/LUT je dostupný v karte **Prehliadač obrázkov** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Kliknite na obrázok v mriežke obrázkov prehliadača súborov, otvorí sa na karte **Prehliadač obrázkov**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> 2. Kliknite na kartu**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , aby sa otvoril ľavý vyskakovací bočný panel, ak ešte nie je otvorený

### Výber obrázku, na ktorý sa má použiť index/LUT

Práca s indexom v <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Otvorte obrázok** z hlavnej mriežky obrázkov kliknutím naň
2. Otvorí sa karta **Prehliadač obrázkov** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sa otvorí
3. Kliknite na **roztváraciu ponuku Vrstva** (v pravom hornom rohu prehliadača)
4. Z roztváracej ponuky vyberte vrstvu:
   * RAW (Odrazivosť)

### Použitie indexu na obrázok

Keď je obrázok na celej obrazovke a bočný panel **Prehliadač obrázkov** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Zaškrtnite políčko Index v hornej časti bočného panela
2. Z roletového menu vľavo vyberte filter vášho fotoaparátu
3. Z roletového menu vpravo vyberte požadovaný vzorec indexu
4. Pretiahnite farebné kruhy kanálov filtra na miesta vo vzorci indexu nižšie
5. Akonáhle je vzorec platný, obrázok sa aktualizuje a zobrazí hodnoty indexu
6. Pohybujte kurzorom myši, aby ste videli hodnoty v mieste, kde sa kurzor nachádza
7. Zväčšite zobrazenie, aby ste videli jednotlivé pixely a ich príslušné hodnoty

Každý index má špecifický rozsah hodnôt a význam:

#### Príklad NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

Kompletnú dokumentáciu k vzorcom indexov nájdete v [Vzorce multispektrálnych indexov](../project-settings/multispectral-index-formulas.md).

***

## Práca s LUT (vyhľadávacími tabuľkami)

### Čo je LUT?

**Vyhľadávacia tabuľka (LUT)** priraďuje numerické hodnoty indexov k farbám na účely vizualizácie:

* **Vstup**: Hodnota pixelu indexu (napr. NDVI 0,65)
* **Výstup**: Farba (napr. jasne zelená)
* **Účel**: Uľahčiť videnie a interpretáciu vzorov**LUT v odtieňoch šedej vs. farebná LUT:**

* Odtiene šedej: Vedecké a neutrálne, zobrazujú surové dáta
* Farebná LUT: Intuitívne a pôsobivé, zvýrazňuje vzory a rozdiely

{% hint style="success" %}
**Sila vizualizácie**: Použitie farebnej LUT na indexový obrázok v odtieňoch šedej výrazne uľahčuje identifikáciu vzorov, anomálií a oblastí záujmu na prvý pohľad.
{% endhint %}

### Použitie LUT na indexový obrázok

Keď máte indexový obrázok zobrazujúci

1. Kliknite na tlačidlo <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> tlačidlo „+Pridať LUT“
2. Vyberte farebný prechod
3. Nastavte minimálny a maximálny bod orezania
4. Nastavte režim orezania
5. Zaškrtnite políčko Index v bočnom paneli **Prehliadač obrázkov** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na bočnom paneli, aby ste aplikovali LUT

### Výber farebného prechodu

**Výber prechodu:**

1. V paneli LUT vyhľadajte**farebnú lištu prechodu**

2. Naveďte kurzor myši na ňu, aby ste videli dostupné prednastavenia prechodov
3. Vyberte požadovaný prechod
4. Obraz sa **okamžite aktualizuje** s novými farbami, keď je zaškrtnuté políčko Index

{% hint style="success" %}
**Osvedčená prax**: Pre vegetačné indexy, ako je NDVI, je prechod Red-Yellow-Green najintuitívnejší, pretože zodpovedá prirodzeným farebným asociáciám (zelená = zdravá, žltá = stredná, červená = stresovaná).
{% endhint %}

### Úprava farebných tried

**Ovládací prvok Triedy**určuje, koľko diskrétnych farebných krokov sa zobrazí vo vašom gradiente:**Možnosti počtu tried:*** **2–5 tried**: Veľmi široké kategórie, zreteľné zóny
* **6–10 tried**: Vyvážené, vhodné na klasifikáciu
* **11–20 tried**: Plynulé prechody, súvislý vzhľad
* **20+ tried**: Takmer súvislé, maximálna plynulosť**Ako nastaviť:**

1. V paneli LUT vyhľadajte**štvorce farebných vzoriek pod lištou prechodu**

2. Upravte počet tried pridaním pomocou tlačidla +
3. Odstráňte počet tried dvojitým kliknutím na farebnú vzorku
4. Prechod sa **v reálnom čase** aktualizuje na obrázku**Vplyv na vizualizáciu:*** **Menej tried** (3–5): Vytvára odlišné zóny, zjednodušená klasifikácia, ľahšie rozlíšiteľné kategórie
* **Stredný počet tried** (6–10): Vyvážený prístup, vhodný pre väčšinu aplikácií
* **Viac tried** (15–20): Plynulé prechody, detailné variácie, fotografický vzhľad**Kedy použiť:*** **Málo tried (3–5)**: Prezentačné snímky, klasifikačné mapy, jednoduché správy
* **Stredný počet tried (6–10)**: Všeobecná analýza, vyvážené detaily, štandardné správy
* **Veľa tried (15–20)**: Vedecká analýza, podrobná kontrola, výstupy v kvalite vhodné na publikovanie

### Jemné ladenie rozsahov hodnôt

**Ovládacie prvky rozsahu hodnôt**určujú, ktoré hodnoty indexu sa priraďujú k ktorým farbám vo vašom prechode:**Ovládacie prvky rozsahu v paneli LUT:*** **Minimálna hodnota**: Dolná hranica farebnej stupnice
* **Maximálna hodnota**: Horná hranica farebnej stupnice
* **Medzipodielové hodnoty**: Automaticky rozdelené medzi min a max (na základe počtu tried)

#### Úprava minimálnych/maximálnych hodnôt

**Úprava rozsahov hodnôt:**

1. V paneli LUT vyhľadajte vstupné polia**Minimálna hodnota**a**Maximálna hodnota**

2. Kliknite na pole**Minimálna hodnota**

3. Zadajte požadovanú minimálnu hodnotu (napr. `0.2`)
4. Stlačte kláves **Enter** alebo kliknite mimo poľa
5. Opakujte pre pole **Max Value** (napr. `0.9`)
6. Vizualizácia sa **okamžite aktualizuje**{% hint style="info" %}**Automatické škálovanie**: Keď po prvýkrát použijete LUT, Chloros automaticky nastaví min/max na skutočný rozsah údajov v obrázku. Tento rozsah môžete následne zúžiť, aby ste sa zamerali na konkrétne rozsahy hodnôt, ktoré vás zaujímajú.
{% endhint %}

**Príklady úprav rozsahu NDVI:*** **Celý rozsah**: `-1.0` až `1.0` (zobraziť všetky možné hodnoty)
* **Zamerané na vegetáciu**: `0.2` až `0.9` (vylúčiť holú pôdu a vodu)
* **Iba zdravá vegetácia**: `0.5` až `0.9` (zvýrazniť iba vitálne rastliny)
* **Detekcia stresu**: `0.2` až `0.5` (zdôrazniť problémové oblasti)
* **Vlastný rozsah**: Upravte na základe pozorovaných hodnôt pixelov**Prečo upravovať rozsahy?*** **Zvýšiť kontrast** v oblasti záujmu
* **Vylúčiť irelevantné hodnoty** (napr. vodné plochy, holú pôdu)
* **Štandardizovať vizualizáciu** naprieč viacerými snímkami alebo dátumami
* **Zdôrazniť jemné rozdiely** v úzkom rozsahu hodnôt

### Odstránenie hodnôt mimo rozsahu

Keď hodnoty pixelov presahujú váš definovaný minimálny/maximálny rozsah, môžete pomocou **režimov orezávania** ovládať spôsob ich zobrazenia.

#### **Dostupné možnosti režimov orezávania:**

#### 1. Minimálna a maximálna hodnota

* Pixely **pod minimálnou hodnotou**→ zobrazia sa pomocou**prvej farby** v prechode (napr. červenej)
* Pixely **nad maximálnou hodnotou**→ zobrazia sa pomocou**poslednej farby** v prechode (napr. zelenej)
* **Prípad použitia**: Zdôraznenie extrémov, zobrazenie celého rozsahu údajov s nasýtenými farbami na hraniciach
* **Príklad**: Hodnoty NDVI pod 0,2 sa všetky zobrazujú červenou farbou, hodnoty nad 0,9 sa všetky zobrazujú zelenou farbou

#### 2. Priehľadné pozadie

* Pixely **mimo rozsahu**sa stanú**úplne priehľadnými*** Len pixely **v rámci rozsahu** zobrazujú farebný prechod
* **Prípad použitia**: GIS prekrývanie, izolovanie špecifických rozsahov hodnôt, zvýraznenie len oblastí záujmu
* **Príklad**: Zobraziť len hodnoty NDVI 0,4–0,7 vo farbe, všetko ostatné priehľadné

{% hint style="warning" %}
**Obmedzenie priehľadnosti**: Priehľadné pixely sa v prehliadači zobrazia ako farba pozadia. Pri exportovaní počas spracovania sa priehľadnosť zachová vo formáte PNG, ale nie vo formáte JPG.
{% endhint %}

#### 3. Pozadie indexu

* Pixely **mimo rozsahu**sa zobrazujú v**odtieňoch šedej** (zobrazujú surové hodnoty indexu)
* Pixely **v rámci rozsahu**zobrazujú**farebný prechod*** **Prípad použitia**: Jemné zvýraznenie, zachovanie kontextu pri zdôraznení oblastí záujmu
* **Príklad**: Farebné zvýraznenie stresovanej vegetácie (NDVI 0,3–0,5) pri zobrazení zdravých oblastí v šedej farbe

#### 4. Pôvodné pozadie

* Pixely **mimo rozsahu**zobrazujú**pôvodný multispektrálny obraz*** Pixely **v rozsahu**zobrazujú**farebný prechod*** **Prípad použitia**: Najintuitívnejšie – kombinuje prirodzený kontext obrazu s analytickou farebnou vrstvou
* **Príklad**: Pozrite si skutočný vzhľad poľa/plodiny s farebne označenými stresovými oblasťami

### Výber správneho režimu orezávania

| Režim orezávania              | Najvhodnejšie pre                                   | Štýl vizualizácie          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimálna a maximálna hodnota**    | Zobrazenie všetkých údajov, vedecká analýza     | Všetky pixely sú farebné           |
| **Priehľadné pozadie** | GIS prekrytia, izolovanie špecifických rozsahov    | Farba v rozsahu, mimo rozsahu prázdne |
| **Indexové pozadie**       | Jemné zdôraznenie, zachovanie kontextu údajov  | Farba v rozsahu, mimo rozsahu šedá  |
| **Pôvodné pozadie**    | Správy, prezentácie, intuitívna analýza | Farba v rozsahu, mimo rozsahu fotografia |

### Vytváranie vlastných farieb LUT

Pre úplnú kontrolu nad vizualizáciou môžete vytvoriť **vlastné farebné prechody** úpravou jednotlivých farebných bodov.**Vytvorenie vlastného prechodu:**

1. V paneli LUT vyhľadajte**náhľadovú lištu prechodu**

2. Vyhľadajte**štvorce vzoriek farieb** pod prechodom
3. **Kliknite na farebný bod**, aby ste ho vybrali
4. Otvorí sa **výber farieb**

5. Vyberte novú farbu pomocou:
   * **Farebného kruhu**: Vizuálny výber farieb
   * **Posúvačov RGB/HSV**: Presné ovládanie farieb
   * **Zadania hexadecimálneho kódu**: Presná špecifikácia farby (napr. `#FF0000` pre červenú)
6. Kliknite mimo nástroj na výber farieb, **aby sa nová farba uplatnila**

7. Prechod sa**okamžite aktualizuje** na obrázku**Pridávanie alebo odstraňovanie farebných bodov:*** **Pridať bod**: Kliknite na ikonu +, aby ste na koniec pridali nový farebný vzor
* **Odstrániť bod**: Dvojitým kliknutím na farebný štvorček odstránite vzor**Stratégie prispôsobenia:*** **Invertovať prechod**: Zmeňte poradie farieb, aby ste obrátili význam (napr. zelená = nízka, červená = vysoká)
* **Farby značky**: Zlaďte farebnú paletu vašej organizácie pre správy
* **Vhodné pre farebne slepých**: Použite kombinácie oranžovej a modrej alebo fialovej a žltej
* **Optimalizácia tlače**: Vyberte farby, ktoré fungujú pri farebnej aj čiernobielej tlači
* **Viac prahov**: Použite odlišné farby pri konkrétnych prahových hodnotách na klasifikáciu

{% hint style="info" %}
**Ukladanie vlastných prechodov**: Vlastné prechody je možné uložiť a znovu použiť. Kliknite na ikonu uloženia v paneli LUT, aby ste si uložili vlastné farebné schémy pre budúce použitie.
{% endhint %}

***

## Interaktívny pracovný postup

### Aktualizácie v reálnom čase

Všetky úpravy LUT v sandboxe aktualizujú obrázok **okamžite a interaktívne**:

* **Prepnutie vrstvy** → Obrázok sa okamžite zmení
* **Výber prechodu** → Farby sa okamžite aktualizujú
* **Úprava rozsahu hodnôt** → Kontrast sa mení v reálnom čase
* **Zmena tried** → Hladkosť prechodu sa okamžite aktualizuje
* **Úprava orezania** → Zobrazenie pozadia sa okamžite zmení
* **Úprava farieb** → Vlastný prechod sa okamžite uplatní**Nie je potrebné tlačidlo „Uplatniť“** – všetky zmeny sú živé a interaktívne!

{% hint style="success" %}
**Živá spätná väzba**: Okamžitá vizuálna spätná väzba vám umožňuje rýchlo experimentovať s rôznymi nastaveniami, kým nenájdete optimálnu vizualizáciu pre vaše potreby analýzy.
{% endhint %}

### Pracovný postup iteratívneho vylepšovania

**Typický pracovný postup optimalizácie LUT:**

1.**Vyberte indexovú vrstvu** (napr. RAW (Odrazivosť))
2. **Použite index** – Vyberte filter kamery a vzorec indexu, pretiahnite farebné kruhy na príslušné miesto vo vzorci indexu
3. **Použite gradient LUT** – Začnite s prednastavením Red-Yellow-Green
4. **Skontrolujte hodnoty pixelov** – Pohybujte kurzorom po obrazovke a všimnite si rozsahy hodnôt
5. **Nastavte min/max** – Zúžte rozsah, aby ste sa zamerali na vegetáciu (napr. 0,2 až 0,9)
6. **Vyberte orezanie** – Skúste „Pôvodné pozadie“ pre kontext
7. **Vylepšite farby** – V prípade potreby prispôsobte gradient pre konkrétne zdôraznenie
8. **Dokončite nastavenia**– Zaznamenajte nastavenia a skopírujte ich do nastavení projektu pre spracovanie exportu

### Kontrola hodnôt pixelov

Porozumenie skutočných hodnôt pixelov je kľúčové pre nastavenie efektívnych rozsahov LUT:**Ako kontrolovať hodnoty:**

1. Hodnoty pixelov sa zobrazia, keď je na obrázku**zaškrtnuté** políčko Index alebo Index aj LUT.
2. **Pohybujte kurzorom** po rôznych oblastiach obrázka
3. **Sledujte hodnoty pixelov** zobrazené v legende pri prechádzaní kurzorom
4. Zväčšite zobrazenie, aby ste videli jednotlivé pixely zvýraznené plávajúcou hodnotou
5. **Zapíšte si** rozsahy hodnôt pre rôzne prvky:
   * **Zdravá vegetácia**: napr. NDVI 0,55–0,85
   * **Stresovaná vegetácia**: napr. NDVI 0,30–0,50
   * **Holá pôda**: napr. NDVI 0,05–0,25
   * **Voda** (ak je prítomná): napr. NDVI -0,05 až 0,10**Použitie hodnôt pixelov na nastavenie rozsahov LUT:**Po skontrolovaní hodnôt pixelov prispôsobte minimálne a maximálne hodnoty LUT:**Príklad scenára:*** **Pozorovanie**: Hodnoty pôdy = 0,05–0,25, Stresovaná = 0,25–0,50, Zdravá = 0,50–0,85
* **Cieľ**: Vizualizovať len zdravie rastlín (vylúčiť pôdu)
* **Nastavenia LUT**: Min = `0.25`, Max = `0.85`
* **Orezanie**: „Pôvodné pozadie“, aby bola pôda zobrazená v prirodzenej farbe
* **Výsledok**: Farebný prechod sa vzťahuje len na vegetáciu, pôda sa zobrazuje ako na pôvodnom obrázku

{% hint style="info" %}
**Dynamický rozsah**: Rôzne plodiny, ročné obdobia a fázy rastu budú mať rôzne rozsahy hodnôt. Pred nastavením rozsahov LUT vždy skontrolujte hodnoty pixelov vo vašom konkrétnom súbore údajov.
{% endhint %}

***

## Vlastné indexy (Chloros+)

### Vytváranie vzorcov vlastných indexov

{% hint style="info" %}
**Kde vytvoriť**: Vlastné indexy je možné nakonfigurovať v**Nastaveniach projektu** pred spracovaním, ako aj v bočnom paneli sandboxu prehliadača obrázkov.
{% endhint %}

**Vytvorenie vlastného indexu:**

1.**Otvorte Nastavenia projektu** (pred spracovaním) alebo bočný panel sandboxu prehliadača obrázkov
2. Prejdite na **roztváraciu ponuku Vzorec indexu**

3. Vyhľadajte možnosť**„Vlastný“** (musíte byť prihlásení s licenciou Chloros+)
4. **Definujte svoj vzorec** pomocou premenných pásiem:
   * Názvy pásiem: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` atď.
   * Operátory: `+`, `-`, `*`, `/`, `^` (exponent)
   * Funkcie: `sqrt()`, `abs()` atď. (ak sú podporované)
   * Zátvorky: `()` pre poradie operácií
5. **Pomenujte svoj index** (napr. „MyIndex“ alebo „CustomNDVI“)
6. **Uložte konfiguráciu**

**Príklady vlastných vzorcov:**

```

Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style="warning" %}
**Overenie vzorca**: Uistite sa, že váš vzorec používa pásma dostupné vo vašej kamere. Napríklad RedEdge je k dispozícii len na kamerách s filtrom RedEdge.
{% endhint %}

***

## Ďalšie kroky

Teraz, keď už rozumiete Index/LUT Sandbox:

* **Použite pri spracovaní**: Použite zistené nastavenia v [Nastaveniach projektu](../project-settings/project-settings.md)
* **Hromadné spracovanie**: Použite optimalizované indexy na celé súbory údajov
* **Ďalšie informácie**: Prečítajte si [Vzorce multispektrálnych indexov](../project-settings/multispectral-index-formulas.md)

Súvisiaca dokumentácia:

* [**Obrázkové vrstvy**](image-layers.md) – Správa a vizualizácia vrstiev
* [**Otvorenie obrázku na celú obrazovku**](opening-an-image-full-screen.md) – Základy prehliadača obrázkov
* [**Spracovanie obrázkov (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Kompletný pracovný postup spracovania

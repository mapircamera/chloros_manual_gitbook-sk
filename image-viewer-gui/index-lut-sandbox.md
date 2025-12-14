# Index/LUT Sandbox

Index/LUT Sandbox je interaktívny pracovný priestor v rámci prehliadača obrázkov Chloros, ktorý vám umožňuje experimentovať s výpočtami multispektrálnych indexov a vizualizáciami farieb v reálnom čase. Tento výkonný nástroj vám pomáha testovať rôzne indexy, vylepšovať rozsahy hodnôt a vytvárať vizualizácie pripravené na publikovanie bez nutnosti opätovného spracovania celého súboru údajov.

## Čo je Index/LUT Sandbox?

### Účel

Sandbox poskytuje:

* **Výpočet indexu v reálnom čase** – okamžité použitie akéhokoľvek vegetačného indexu
* **Interaktívne nastavenie LUT** – jemné doladenie farebných prechodov a rozsahov
* **Optimalizácia pracovného postupu** – určenie najlepších nastavení pred hromadným spracovaním

### Sandbox vs. spracovanie projektu

**Index/LUT Sandbox (interaktívny):**

* Jeden obrázok naraz
* Okamžitá spätná väzba
* Experimentálny a iteratívny
* Žiadne trvalé zmeny súborov
* Ideálny na preskúmavanie a testovanie

**Spracovanie projektu (hromadné):**

* Celý súbor údajov naraz
* Vopred nakonfigurované nastavenia
* Trvalé výstupné súbory
* Časovo náročné
* Najlepšie, keď sú nastavenia finalizované

{% hint style=&quot;success&quot; %}
**Najlepší pracovný postup**: Použite Sandbox na experimentovanie a nájdenie optimálnych nastavení indexu a LUT, potom tieto nastavenia použite počas spracovania projektu pre celý súbor údajov.
{% endhint %}

***

## Práca s Index/LUT Sandbox

### Porozumenie vopred vypočítaným indexom

V Chloros je možné indexy použiť počas spracovania projektu. Na určenie, ktoré nastavenia indexu a LUT chcete použiť na export, je najjednoduchšie použiť sandbox prehliadača obrázkov.

Sandbox vám umožňuje:

* **Použiť nové indexy a farebné prechody (LUT)** na vizualizáciu údajov.
* **Interaktívne prispôsobiť nastavenia vizualizácie**.
* **Zobraziť** už vypočítané indexové obrázky.
* **Skontrolovať** hodnoty pixelov na všetkých úrovniach zväčšenia.

### Otvorenie sandboxu

Sandbox Index/LUT je prístupný v **prehliadači obrázkov** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Kliknite na obrázok v mriežke prehliadača súborov, otvorí sa v prehliadači obrázkov <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
2. Kliknite na kartu **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> , aby sa otvoril ľavý bočný panel, ak ešte nie je otvorený

### Výber obrázku, na ktorý sa má použiť index/LUT

Ak chcete pracovať s indexom v prehliadači obrázkov <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Otvorte obrázok** z hlavnej mriežky obrázkov kliknutím naň.
2. Otvorí sa karta **Prehliadač obrázkov** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
3. Kliknite na **roztiahnuteľné menu Vrstva** (v pravom hornom rohu prehliadača).
4. Z roztiahnuteľného menu vyberte vrstvu:
   * RAW (odrazivosť)

### Použitie indexu na obrázok

Keď je obrázok zobrazený na celej obrazovke a bočný panel **Prehliadač obrázkov** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Zaškrtnite políčko Index v hornej časti bočného panela.
2. Z ľavého roletového menu vyberte filter vašej kamery.
3. Z pravého roletového menu vyberte požadovaný indexový vzorec.
4. Pretiahnite farebné kruhy kanálov filtra na miesta v indexovom vzorci nižšie.
5. Akonáhle je vzorec platný, obrázok sa aktualizuje a zobrazí hodnoty indexu.
6. Pohybujte kurzorom myši, aby ste videli hodnoty v mieste kurzora.
7. Zväčšite zobrazenie, aby ste videli jednotlivé pixely a ich pridružené hodnoty.

Každý index má špecifický rozsah hodnôt a význam:

#### NDVI Príklad

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

Úplnú dokumentáciu vzorcov indexov nájdete v časti [Vzorce multispektrálnych indexov](../project-settings/multispectral-index-formulas.md).

***

## Práca s LUT (vyhľadávacími tabuľkami)

### Čo je LUT?

**Vyhľadávacia tabuľka (LUT)** priraďuje numerické hodnoty indexu k farbám na účely vizualizácie:

* **Vstup**: Hodnota pixelu indexu (napr. NDVI 0,65)
* **Výstup**: Farba RGB (napr. jasne zelená)
* **Účel**: Uľahčiť videnie a interpretáciu vzorov

**Škála sivej vs. farebná LUT:**

* Škála sivej: Vedecká a neutrálna, zobrazuje surové údaje
* Farebná LUT: Intuitívna a pôsobivá, zdôrazňuje vzory a rozdiely

{% hint style=&quot;success&quot; %}
**Vizuálna sila**: Použitie farebnej LUT na indexový obrázok v odtieňoch sivej výrazne uľahčuje identifikáciu vzorov, anomálií a oblastí záujmu na prvý pohľad.
{% endhint %}

### Použitie LUT na indexový obrázok

Keď máte indexový obrázok zobrazujúci

1. Kliknite na tlačidlo <img src="../.gitbook/assets/image.png" alt="" data-size="line"> „+Pridať LUT“
2. Vyberte farebný prechod
3. Nastavte minimálne/maximálne koncové body orezania
4. Nastavte režim orezania
5. Zaškrtnite políčko Index v bočnom paneli **Prehliadača obrázkov** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na bočnom paneli, aby ste aplikovali LUT.

### Výber farebného prechodu

**Výber prechodu:**

1. V paneli LUT vyhľadajte **farebný gradientový pruh**.
2. Naveďte naň kurzor myši, aby sa zobrazili dostupné prednastavenia gradientu.
3. Vyberte požadovaný gradient.
4. Obraz sa **okamžite aktualizuje** novými farbami, keď je zaškrtnuté políčko Index.

{% tip style=&quot;success&quot; %}
**Osvedčené postupy**: Pre vegetačné indexy, ako je NDVI, je najintuitívnejší prechod Red-Yellow-Green, pretože zodpovedá prirodzeným asociáciám farieb (zelená = zdravá, žltá = stredná, červená = stresovaná)..
{% endhint %}

### Úprava farebných tried

**Ovládací prvok Triedy** určuje, koľko diskrétnych farebných krokov sa zobrazí vo vašom gradiente:

**Možnosti počtu tried:**

* **2–5 tried**: Veľmi široké kategórie, odlišné zóny
* **6–10 tried**: Vyvážené, vhodné na klasifikáciu
* **11–20 tried**: Plynulé prechody, súvislý vzhľad
* **20+ tried**: Takmer súvislé, maximálna plynulosť

**Ako nastaviť:**

1. V paneli LUT vyhľadajte **farebné vzorky pod lištou prechodu**
2. Upravte počet tried pridaním pomocou tlačidla +
3. Odstráňte počet tried dvojitým kliknutím na farebný vzorník.
4. Prechod sa aktualizuje **v reálnom čase** na obrázku.

**Vplyv na vizualizáciu:**

* **Menej tried** (3–5): Vytvára odlišné zóny, zjednodušenú klasifikáciu, ľahšie rozlíšiteľné kategórie.
* **Stredný počet tried** (6–10): Vyvážený prístup, vhodný pre väčšinu aplikácií.
* **Viac tried** (15-20): Plynulé prechody, podrobné variácie, fotografický vzhľad

**Kedy použiť:**

* **Málo tried (3-5)**: Prezentačné snímky, klasifikačné mapy, jednoduché správy
* **Stredný počet tried (6-10)**: Všeobecná analýza, vyvážené detaily, štandardné správy
* **Veľa tried (15–20)**: Vedecká analýza, podrobná kontrola, výstupy v kvalite publikácií

### Jemné ladenie rozsahov hodnôt

**Ovládacie prvky rozsahu hodnôt** určujú, ktoré hodnoty indexu sa priraďujú k ktorým farbám vo vašom prechode:

**Ovládacie prvky rozsahu v paneli LUT:**

* **Minimálna hodnota**: Dolná hranica farebnej škály
* **Maximálna hodnota**: Horná hranica farebnej škály
* **Stredné hodnoty**: Automaticky rozdelené medzi min a max (na základe počtu tried)

#### Úprava minimálnych/maximálnych hodnôt

**Úprava rozsahov hodnôt:**

1. V paneli LUT vyhľadajte vstupné polia **Minimálna hodnota** a **Maximálna hodnota**
2. Kliknite na pole **Minimálna hodnota**
3. Zadajte požadovanú minimálnu hodnotu (napr. `0.2`)
4. Stlačte kláves **Enter** alebo kliknite mimo poľa
5. Opakujte pre pole **Maximálna hodnota** (napr. `0.9`)
6. Vizualizácia sa **okamžite aktualizuje**

{% hint style=&quot;info&quot; %}
**Automatické škálovanie**: Keď prvýkrát použijete LUT, Chloros automaticky nastaví min/max na skutočný rozsah údajov v obrázku. Potom môžete tento rozsah zúžiť, aby ste sa zamerali na konkrétne rozsahy hodnôt, ktoré vás zaujímajú.
{% endhint %}

**Príklad úprav rozsahu NDVI:**

* **Celý rozsah**: `-1.0` až `1.0` (zobraziť všetky možné hodnoty)
* **Zamerané na vegetáciu**: `0.2` až `0.9` (vylúčiť holú pôdu a vodu)
* **Iba zdravá vegetácia**: `0.5` až `0.9` (zvýrazniť iba vitálne rastliny)
* **Detekcia stresu**: `0.2` až `0.5` (zdôraznenie problémových oblastí)
* **Vlastný rozsah**: Úprava na základe pozorovaných hodnôt pixelov

**Prečo upravovať rozsahy?**

* **Zvýšenie kontrastu** v oblasti záujmu
* **Vylúčte irelevantné hodnoty** (napr. vodné plochy, holá pôda)
* **Štandardizujte vizualizáciu** naprieč viacerými obrázkami alebo dátumami
* **Zdôraznite jemné rozdiely** v úzkom rozsahu hodnôt

### Orezávanie hodnôt mimo rozsah

Keď hodnoty pixelov presahujú definovaný minimálny/maximálny rozsah, môžete pomocou **režimov orezávania** kontrolovať, ako sa zobrazujú.

#### **Dostupné možnosti režimov orezávania:**

#### 1. Minimálna a maximálna hodnota

* Pixely **pod minimálnou hodnotou** → zobrazenie pomocou **prvej farby** v gradiente (napr. červená)
* Pixely **nad maximálnou hodnotou** → zobrazenie pomocou **poslednej farby** v gradiente (napr. zelená)
* **Prípad použitia**: Zdôraznenie extrémov, zobrazenie celého rozsahu údajov s nasýtenými farbami na hraniciach
* **Príklad**: Hodnoty NDVI pod 0,2 sa zobrazujú červenou farbou, hodnoty nad 0,9 sa zobrazujú zelenou farbou

#### 2. Priehľadné pozadie

* Pixely **mimo rozsahu** sa stanú **úplne priehľadnými**
* Len pixely **v rozsahu** zobrazujú farebný gradient
* **Prípad použitia**: GIS prekrývanie, izolovanie špecifických rozsahov hodnôt, zvýraznenie len oblastí záujmu
* **Príklad**: Zobraziť len NDVI 0,4-0,7 vo farbe, všetko ostatné priehľadné

{% hint style=&quot;warning&quot; %}
**Obmedzenie priehľadnosti**: Priesvitné pixely sa v prehliadači zobrazia ako farba pozadia. Pri exportovaní počas spracovania sa priehľadnosť zachová vo formáte PNG, ale nie vo formáte JPG.
{% endhint %}

#### 3. Pozadie indexu

* Pixely **mimo rozsahu** sa zobrazujú v **odtieňoch sivej** (zobrazujú surové hodnoty indexu)
* Pixely **v rozsahu** zobrazujú **farebný prechod**
* **Prípad použitia**: Jemné zvýraznenie, zachovanie kontextu a zdôraznenie oblastí záujmu.
* **Príklad**: Farebne zvýraznená namáhaná vegetácia (NDVI 0,3–0,5) a zdravé oblasti zobrazené v sivej farbe.

#### 4. Pôvodné pozadie

* Pixely **mimo rozsah** sa zobrazia ako **pôvodný multispektrálny obraz**.
* Pixely **v rozsahu** zobrazujú **farebný prechod**
* **Prípad použitia**: Najintuitívnejší – kombinuje prirodzený kontext obrazu s analytickým farebným prekrytím
* **Príklad**: Pozrite si skutočný vzhľad poľa/plodiny s farebne označenými stresovými oblasťami

### Výber správneho režimu orezávania

| Režim orezávania              | Najvhodnejšie pre                                   | Štýl vizualizácie          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum a maximum**    | Zobrazenie všetkých údajov, vedecká analýza     | Všetky pixely farebné           |
| **Transparentné pozadie** | GIS prekrytia, izolovanie špecifických rozsahov    | Farba v rozsahu, mimo rozsahu prázdne |
| **Indexové pozadie**       | Jemné zdôraznenie, zachovanie kontextu údajov  | Farba v rozsahu, mimo rozsahu sivá  |
| **Pôvodné pozadie**    | Správy, prezentácie, intuitívna analýza | Farba v rozsahu, mimo rozsahu fotografia |

### Vytvorenie vlastných farieb LUT

Pre úplnú kontrolu nad vizualizáciou môžete vytvoriť **vlastné farebné prechody** úpravou jednotlivých farebných zastavení.

**Vytvorenie vlastného prechodu:**

1. V paneli LUT vyhľadajte **lištu náhľadu prechodu**.
2. Pod prechodom vyhľadajte **štvorce farebných vzoriek**.
3. **Kliknutím na farebné zastavenie** ho vyberte.
4. Otvorí sa **výber farieb**.
5. Vyberte novú farbu pomocou:
   * **Farebného kruhu**: Vizuálny výber farieb.
   * **Posuvníkov RGB/HSV**: Presné ovládanie farieb.
   * **Zadania hexadecimálneho kódu**: Presná špecifikácia farieb (napr. `#FF0000` pre červenú).
6. Kliknutím mimo výberu farieb **aplikujete novú farbu**.
7. Prechod **sa okamžite aktualizuje** na obrázku.

**Pridávanie alebo odstraňovanie farebných zastavení:**

* **Pridanie zastavenia**: Kliknutím na ikonu + pridáte nový vzor na koniec.
* **Odstránenie zastavenia**: Dvojitým kliknutím na farebný štvorček odstránite vzor.

**Stratégie prispôsobenia:**

* **Invertovať prechod**: Obráťte poradie farieb, aby ste obrátili význam (napr. zelená = nízka, červená = vysoká).
* **Farby značky**: Zlaďte farebnú paletu vašej organizácie pre správy.
* **Vhodné pre farboslepých**: Použite kombinácie oranžová-modrá alebo fialová-žltá.
* **Optimalizácia tlače**: Vyberte farby, ktoré fungujú pri farebnej aj čiernobielej tlači.
* **Viac prahov**: Použite odlišné farby pri konkrétnych prahoch hodnôt na klasifikáciu.

{% hint style=&quot;info&quot; %}
**Uloženie vlastných prechodov**: Vlastné prechody je možné uložiť a znovu použiť. Kliknutím na ikonu uloženia v paneli LUT môžete uložiť vlastné farebné schémy pre budúce použitie.
{% endhint %}

***

## Interaktívny pracovný postup

### Aktualizácie v reálnom čase

Všetky úpravy LUT v sandboxe aktualizujú obraz **okamžite a interaktívne**:

* **Prepnúť vrstvu** → Obraz sa okamžite zmení
* **Vybrať prechod** → Farby sa aktualizujú okamžite
* **Upravte rozsah hodnôt** → Kontrast sa mení v reálnom čase
* **Zmeňte triedy** → Plynulosť prechodu sa aktualizuje okamžite
* **Upravte orezanie** → Zobrazenie pozadia sa okamžite zmení
* **Úprava farieb** → Vlastný gradient sa uplatní okamžite

**Nie je potrebné tlačidlo „Uplatniť“** – všetky zmeny sú živé a interaktívne!

{% hint style=&quot;success&quot; %}
**Živá spätná väzba**: Okamžitá vizuálna spätná väzba vám umožňuje rýchlo experimentovať s rôznymi nastaveniami, kým nenájdete optimálnu vizualizáciu pre vaše potreby analýzy.
{% endhint %}

### Iteratívny pracovný postup vylepšovania

**Typický pracovný postup optimalizácie LUT:**

1. **Vyberte indexovú vrstvu** (napr. RAW (odrazivosť))
2. **Použite index** – Vyberte filter fotoaparátu a indexovú formulu, pretiahnite farebné kruhy na vhodné miesto v indexovej formule
3. **Použite gradient LUT** – Začnite s predvoleným nastavením Red-Yellow-Green
4. **Skontrolujte hodnoty pixelov** – presuňte kurzor a zaznamenajte rozsahy hodnôt
5. **Upravte min/max** – zúžte, aby ste sa zamerali na vegetáciu (napr. 0,2 až 0,9)
6. **Vyberte orezanie** – vyskúšajte „Original Background“ (Pôvodné pozadie) pre kontext
7. **Vylepšite farby** – v prípade potreby prispôsobte gradient pre konkrétny dôraz
8. **Finalizujte nastavenia** – Dokumentujte nastavenia a skopírujte ich do nastavení projektu na účely exportu

### Kontrola hodnôt pixelov

Porozumenie skutočným hodnotám pixelov je kľúčové pre nastavenie efektívnych rozsahov LUT:

**Ako kontrolovať hodnoty:**

1. Hodnoty pixelov sa zobrazia, ak je u obrázku zaškrtnuté políčko Index alebo políčka Index a LUT.
2. **Presuňte kurzor** na rôzne oblasti obrázku
3. **Pozorujte hodnoty pixelov** zobrazené v legende, keď sa nad nimi nachádzate
4. Zväčšite zobrazenie, aby ste videli jednotlivé pixely zvýraznené plávajúcou hodnotou
5. **Zapíšte si** rozsahy hodnôt pre rôzne vlastnosti:
   * **Zdravá vegetácia**: napr. NDVI 0,55–0,85
   * **Stresovaná vegetácia**: napr. NDVI 0,30–0,50
   * **Holá pôda**: napr. NDVI 0,05–0,25
   * **Voda** (ak je prítomná): napr. NDVI -0,05 až 0,10

**Použitie hodnôt pixelov na nastavenie rozsahov LUT:**

Po skontrolovaní hodnôt pixelov prispôsobte minimálne/maximálne hodnoty LUT:

**Príklad scenára:**

* **Pozorovanie**: Hodnoty pôdy = 0,05–0,25, stres = 0,25–0,50, zdravý stav = 0,50–0,85
* **Cieľ**: Vizualizovať iba zdravie rastlín (vynechať pôdu)
* **Nastavenia LUT**: Min = `0.25`, Max = `0.85`
* **Orezanie**: „Pôvodné pozadie“ na zobrazenie pôdy v prirodzenej farbe
* **Výsledok**: Farebný prechod sa vzťahuje iba na vegetáciu, pôda sa zobrazuje ako pôvodný obrázok

{% hint style=&quot;info&quot; %}
**Dynamický rozsah**: Rôzne plodiny, ročné obdobia a fázy rastu budú mať rôzne rozsahy hodnôt. Pred nastavením rozsahov LUT vždy skontrolujte hodnoty pixelov vo vašej konkrétnej sade údajov.
{% endhint %}

***

## Vlastné indexy (Chloros+)

### Vytváranie vlastných vzorcov indexov

{% hint style=&quot;info&quot; %}
**Kde vytvoriť**: Vlastné indexy je možné nakonfigurovať v **Nastaveniach projektu** pred spracovaním, ako aj v bočnom paneli sandboxu prehliadača obrázkov.
{% endhint %}

**Vytvorenie vlastného indexu:**

1. **Otvorte Nastavenia projektu** (pred spracovaním) alebo bočný panel sandboxu prehliadača obrázkov
2. Prejdite na **roztiahnuteľné menu Vzorec indexu**
3. Vyhľadajte možnosť **„Vlastný“** (musíte byť prihlásení s licenciou Chloros+)
4. **Definujte svoj vzorec** pomocou premenných pásma:
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

{% hint style=&quot;warning&quot; %}
**Overenie vzorca**: Uistite sa, že váš vzorec používa pásma dostupné vo vašej kamere. Napríklad RedEdge je k dispozícii len v kamerách s filtrom RedEdge.
{% endhint %}

***

## Ďalšie kroky

Teraz, keď už rozumiete Index/LUT Sandbox:

* **Použite na spracovanie**: Použite objavené nastavenia v [Nastaveniach projektu](../project-settings/project-settings.md)
* **Hromadné spracovanie**: Použite optimalizované indexy na celé súbory údajov
* **Ďalšie informácie**: Prečítajte si [Vzorec multispektrálneho indexu](../project-settings/multispectral-index-formulas.md)

Súvisiaca dokumentácia:

* [**Vrstvy obrázkov**](image-layers.md) – Správa vrstiev a vizualizácia
* [**Otvorenie obrázku na celú obrazovku**](opening-an-image-full-screen.md) – Základy prehliadača obrázkov
* [**Spracovanie obrázkov (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Kompletný pracovný postup spracovania

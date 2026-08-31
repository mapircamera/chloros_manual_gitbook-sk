# Index/LUT Sandbox

Index/LUT Sandbox je interaktívny pracovný priestor v bočnom paneli prehliadača obrázkov Chloros. Vyberiete vzorec, priradíte k nemu kanály vašej kamery, vyfarvíte ho pomocou prechodu a nastavíte rozsah hodnôt – a obrázok sa pritom aktualizuje v reálnom čase. Od verzie 1.2.0 môžete tiež **uložiť to, čo ste vytvorili**, buď pre jeden obrázok, alebo pre celý projekt, bez nutnosti opätovného spracovania.

## Na čo slúži Sandbox

| Index/LUT Sandbox (interaktívny)        | Spracovanie projektu (hromadné)       |
| -------------------------------------- | -------------------------------- |
| Jeden obrázok po druhom, okamžitá spätná väzba  | Celá sada údajov v jednom cykle     |
| Experimentálne a iteratívne             | Predkonfigurované nastavenia          |
| Rendruje v reálnom čase; ukladá len na požiadanie  | Vždy zapisuje výsledné súbory      |
| Ideálne na hľadanie správnych nastavení | Najlepšie, keď sú nastavenia finálne |

{% hint style="success" %}
**Bežný pracovný postup**: ladte v Sandboxe, kým vizualizácia nevyzerá tak, ako chcete, a potom buď exportujte priamo zo Sandboxu, alebo skopírujte rovnaké nastavenia indexu a LUT do [Nastavení projektu](../project-settings/project-settings.md), aby ich nasledujúce spracovanie zapracovalo do každého obrázku.
{% endhint %}

***

## Otvorenie Sandboxu

1. Kliknite na obrázok v mriežke — otvorí sa na celej obrazovke v karte **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Kliknite na ikonu **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">, aby sa vysunul ľavý bočný panel, ak ešte nie je otvorený
3. Z roletového menu vrstiev v pravom hornom rohu vyberte viacpásmovú vrstvu — bežnou voľbou je **RAW (Reflectance)**, pretože hodnoty indexu vypočítané na základe kalibrovanej odrazivosti sú medzi obrázkami porovnateľné

Bočný panel zobrazuje zhora nadol:

* názov obrázka a model fotoaparátu
* tlačidlo **Exportovať/Uložiť obrázok(-y)** — zobrazí sa po zaškrtnutí políčka Index alebo LUT
* zaškrtávacie políčka **Index**a**LUT**
* panel konfigurácie indexu
* panel **Hodnoty kurzora** s odčítanými hodnotami, histogramom a ovládacím prvkom GSD

{% hint style="warning" %}
**Nedostupné pre monochromatické kamery.** Na jednopásmovom snímku LATTICE M3M sú obe začiarkavacie políčka deaktivované a zobrazuje sa popisok _„Nedostupné pre monochromatické (M3M) senzory“_ — viacpásmový index nie je pre jedno pásmo definovaný. Na výpočet indexov z kamier M3M skombinujte dve alebo viac snímok do zarovnaného viacpásmového stohu a použite indexovací modul LATTICE.
{% endhint %}

***

## Použitie indexu

1. Zaškrtnite políčko **Index** v hornej časti bočného panela
2. Z ľavého roletového menu vyberte filter vašej kamery (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Z pravého roletového menu vyberte indexový vzorec — k dispozícii je 27 vstavaných vzorcov a akékoľvek vlastné vzorce, ktoré ste si uložili
4. Vzorec sa zobrazí ako matematický výraz nižšie, s prázdnym kruhom v každom slote pásma. **Presuňte farebný kruh kanála na políčko**, aby ste ho priradili
5. Akonáhle sú priradené všetky políčka, ktoré vzorec používa, obrázok sa aktualizuje a zobrazí hodnoty indexu
6. Presuňte kurzor nad obrázok, aby ste si prečítali hodnoty; panel **Hodnoty kurzora** pridá riadok indexu s hodnotou pod kurzorom

Dvojitým kliknutím na priradené miesto ho vymažete. Neúplný vzorec je bežný stav počas ťahania, nie chyba — obrázok sa jednoducho neaktualizuje, kým nie je vzorec kompletný.

Kruhy kanálov sú farebne označené: červená = Red, zelená = Green, modrá = Blue, oranžová = Orange, cyan = Cyan, fialová = NIR, magenta = RE. Tie isté farby sa používajú pre bodky kanálov a krivky histogramu v paneli Hodnoty kurzora.

### Príklad NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

Kompletný prehľad vzorcov – všetky tri zoznamy predvolieb a informácie o tom, ktoré názvy kde fungujú – nájdete v [Vzorce multispektrálnych indexov](../project-settings/multispectral-index-formulas.md).

### S zaškrtnutou možnosťou Index, ale bez LUT

Obrázok je vykreslený v **odtieňoch šedej**, roztiahnutý medzi dvoma prahovými hodnotami. Je to zámerné: indexový obrázok predstavuje skalárne údaje a odtiene šedej sú ich verným zobrazením. Ak chcete farbu, pridajte LUT.***

## Práca s LUT (vyhľadávacími tabuľkami)**Vyhľadávacia tabuľka** priraďuje indexové hodnoty k farbám: vstup NDVI 0,65, výstup konkrétna zelená farba. Nemení údaje – mení spôsob, akým ich interpretujete.

### Pridanie LUT

1. Kliknite na <img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> tlačidlo **„+ Pridať LUT“** pod vzorcom
2. Vyberte farebný prechod
3. Nastavte minimálnu a maximálnu hodnotu orezania
4. Zvoľte režim orezania
5. Zaškrtnite políčko **LUT** v bočnom paneli, aby sa LUT vykreslila

Políčko LUT zostáva neaktívne, kým sa LUT skutočne nenakonfiguruje v indexe.

### Výber farebného prechodu

Naveďte kurzor na **pruh prechodu**, aby sa otvoril zoznam predvolieb — Chloros obsahuje**sedem** predvolieb prechodov:

| # | Prechod                            | Tvar                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Žltá → Green (**predvolené**)  | Rozbiehavý — zodpovedá bežnej predstave o vegetácii, zelená = zdravá |
| 2 | Fialová → Žltá → Green             | Divergentný, s výrazným nízkym koncom                                  |
| 3 | Hnedá → Biela → Blue                | Divergentný okolo svetlého stredného bodu                                   |
| 4 | Čierna → Fialová → Ružová → Svetložltá | Sekvenčné, od tmavého k svetlému                                           |
| 5 | Red → Žltá → Blue                 | Odchýlka okolo svetlého stredného bodu                                   |
| 6 | Fialová → Blue → Green → Žltá      | Postupné, od tmavého k svetlému                                           |
| 7 | Orange → Biela → Fialová             | Rozbiehajúci sa okolo svetlého stredového bodu                                   |

**Rozbiehajúci sa**gradient umiestňuje neutrálnu farbu do stredu okna, čo je prehľadné, ak stredný bod má určitý význam (prahová hodnota, referenčný dátum).**Sekvenčný** prechod prechádza monotónne od tmavého k svetlému, čo je vhodné pre veličinu, ktorá má len stavy „viac“ a „menej“.

Každá predvolba má sedem farebných bodov. Kliknutím na predvolbu sa obrázok okamžite aktualizuje (ak je zaškrtnuté políčko LUT).

### Úprava farebných bodov

Pod lištou prechodu sa nachádza rad farebných vzoriek, po jednej na každý bod:

* **Zmena farby**: kliknutím na vzorku otvoríte výber farieb (farebné koleso, posuvníky RGB/HSV alebo hexadecimálny kód, napr. `#FF0000`)
* **Pridať bod**: kliknite na tlačidlo**+** na konci riadku — pridá sa biely bod
* **Odstrániť bod**:**dvojité kliknutie** na vzorku
* **Uloženie upraveného prechodu**: kliknite na ikonu uloženia vedľa lišty prechodu, čím pridáte upravený prechod do zoznamu predvolieb, aby ste ho mohli znovu vybrať

Prechod, ktorý ste nakonfigurovali pre index, sa ukladá spolu s týmto indexom v nastaveniach projektu, takže zostane zachovaný aj po zatvorení a opätovnom otvorení projektu.

**Menej bodov**vytvára výrazné zóny, ktoré pôsobia ako klasifikácia;**viac bodov** vytvára plynulé, takmer fotografické prechody. Tri až päť bodov je vhodných pre prezentačné snímky a klasifikačné mapy; šesť až desať pre všeobecnú analýzu; pätnásť alebo viac pre podrobnú kontrolu a publikačné grafy.

### Nastavenie rozsahu hodnôt

Ovládací prvok prahu je **posuvník s dvoma úchytmi**v rozsahu od −1 do +1, s upraviteľným textovým poľom na každom konci pre zadanie presných hodnôt a tlačidlom**AUTO**.

* Potiahnite ľubovoľnú rukoväť alebo zadajte číslo do príslušného poľa a stlačte kláves Enter
* Funkcia **AUTO**nastaví rozsah na**

2. a 98. percentil** platných indexových hodnôt obrázka — ide o dobrý východiskový bod, ktorý ignoruje extrémne hodnoty. Chloros adaptívne zaokrúhľuje výsledok na 4 desatinné miesta v prípade veľmi úzkeho rozsahu, na 3 v prípade úzkeho rozsahu a v ostatných prípadoch na 2
* Akékoľvek ručné nastavenie má prednosť pred nastavením AUTO, kým opäť nestlačíte tlačidlo AUTO

Príklad okien NDVI:

| Cieľ                                    | Min  | Max |
| --------------------------------------- | ---- | --- |
| Zobraziť všetko                         | −1,0 | 1,0 |
| Iba vegetácia, bez pôdy a vody         | 0,2  | 0,9 |
| Iba zdravá vegetácia                 | 0,5  | 0,9 |
| Zdôrazniť stres                        | 0,2  | 0,5 |

Zúžením rozsahu sa zvýši kontrast v oblasti, ktorá vás zaujíma, a všetko ostatné sa dostane mimo rozsah — kde **režim orezávania** rozhoduje o tom, čo sa s tým stane.***

## Režimy orezávania

Keď hodnota indexu pixelu presiahne rozsah min/max, režim orezania rozhoduje o tom, ako sa pixel vykreslí.

| Názov roletového menu                  | Uložená hodnota      | Pixely mimo rozsahu sa vykreslia ako                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minimum a maximum** (predvolené) | `clip`            | Najbližšia koncová farba prechodu — hodnoty pod minimom nadobúdajú prvú farbu, hodnoty nad maximom nadobúdajú poslednú |
| **Priehľadné pozadie**      | `transparent`     | Úplne priehľadné (skutočná alfa)                                                                                                  |
| **Indexové pozadie**| `indexColor`      | Šedá stupnica, roztiahnutá cez**celý** indexový rozsah obrázku, takže štruktúra mimo rozsahu je stále viditeľná v šedej farbe                |
| **Pôvodné pozadie**         | `backgroundColor` | Samotný podkladový obrázok, takže farebná vrstva leží na skutočnej scéne                                                |

| Režim                       | Najvhodnejšie pre                               | Vzhľad                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Minimálna a maximálna hodnota**      | Úplné zobrazenie údajov, vedecká analýza | Každý pixel je sfarbený                      |
| **Priehľadné pozadie** | GIS prekrytia, izolácia hodnotového pásma   | Farba vnútri okna, mimo neho nič |
| **Indexové pozadie**       | Zdôraznenie pri zachovaní kontextu údajov    | Farba vnútri, šedá vonku               |
| **Pôvodné pozadie**    | Správy a prezentácie              | Farba vnútri, fotografia vonku         |

{% hint style="info" %}
**Pixely bez údajov sú vždy priehľadné, v každom režime.** Pixel, ktorého index nie je konečný (delenie 0/0) alebo je presne −1,0 alebo +1,0 (saturation sentinels, keď jedno spektrálne pásmo má hodnotu nula, zatiaľ čo druhé nie) sa považuje za pixel bez údajov, a nie za extrémnu hodnotu. Tým sa zabráni tomu, aby preexponované svetlá a úplne tmavé tiene zasahovali do farebnej škály, namiesto toho, aby sa vykresľovali ako najextrémnejšie hodnoty v snímke. To isté pravidlo určuje, ktoré pixely sa používajú pre prahové hodnoty AUTO a indexový histogram, takže všetky tri sa zhodujú.
{% endhint %}

Priehľadnosť sa zachová, ak sa export uloží vo formáte PNG. Vo formáte JPG ju nie je možné vyjadriť.

***

## Čítanie hodnôt počas ladenia

Panel **Hodnoty kurzora** pod konfiguračným panelom slúži ako merací prístroj pre Sandbox:

* Presuňte kurzor nad obrázok a prečítajte si zdrojové hodnoty jednotlivých kanálov, ako aj hodnotu indexu v samostatnom riadku
* Zapnite tlačidlo **INDEX** nad histogramom, aby ste videli rozdelenie indexových hodnôt v snímke, pričom vaše dve prahové hodnoty klipu sú znázornené ako oranžové prerušované čiary a hodnota kurzora ako biela čiara — toto je najrýchlejší spôsob, ako vybrať okno, ktoré skutočne obsahuje vaše údaje
* Zapnite **CURSOR**, aby ste videli značkovacie čiary pri hodnotách pod kurzorom
* Zväčšite zobrazenie nad 60× (menej, ak je nastavená veľkosť bloku GSD), aby sa zvýraznili jednotlivé zobrazené pixely s plávajúcou hodnotou

Praktický postup:

1. Poznačte si hodnoty nad zdravou vegetáciou, stresovanou vegetáciou, holou pôdou a vodou
2. Pozrite sa, kde sa tieto zhluky nachádzajú na histograme indexu
3. Nastavte min/max tak, aby ohraničili zhluk, ktorý vás zaujíma
4. Vyberte režim orezania — _Original Background_ zachová viditeľnosť scény okolo neho

***

## Export z Sandboxu

Všetko uvedené vyššie je živý náhľad, kým to neuložíte. Tlačidlo **Exportovať/Uložiť obrázok(-y)** v hornej časti bočného panela otvorí okno, ktoré sa posunie nad bočný panel (namiesto toho, aby zakrývalo obrázok, takže stále vidíte, o čom rozhodujete).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Možnosti

| Možnosť                          | Účinok                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Použiť na aktuálny obrázok**      | Uloží presne zobrazený obrázok s týmito nastaveniami                                                                                                |
| **Použiť na všetky obrázky v projekte** | Znovu spustí identickú konfiguráciu na každom obrázku v projekte. Obrázky bez pásiem, ktoré tento index potrebuje, sa preskočia a nepovažujú sa za chyby |
| **Gradientová lišta indexu/LUT**      | Pri každom exporte sa tiež uloží samostatný obrázok s legendou, na ktorom je označený rozsah hodnôt                                                                     |
| **Histogram indexu**             | Pri každom exporte sa tiež uloží samostatný obrázok histogramu, ktorý zobrazuje minimálne a maximálne hodnoty údajov a prahové hodnoty orezania                                               |

Ak je hodnota **Veľkosť bloku GSD** na karte snímky väčšia ako 1, v paneli sa to zobrazí ešte pred potvrdením: export uloží to, čo vidíte, vrátane blokového priemerovania. Ak chcete plné rozlíšenie, najprv nastavte ovládací prvok GSD späť na hodnotu 1.

### Kam sa súbory ukladajú

Každé kliknutie na **Export**vytvorí**novú zložku, ktorá sa nikdy viac nepoužije**:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Príklady: `Sandbox_Exports/NDVI_LUT_001/`, potom `Sandbox_Exports/NDVI_LUT_002/` pre ďalšie spustenie. Číslovanie sa odvodzuje na základe skenovania toho, čo sa už nachádza na disku, takže pretrváva aj po reštartoch a ručnom vymazaní zložiek. Nikdy sa nič neprepíše — zmyslom Sandboxu je porovnávať jeden pokus s predchádzajúcim.

Vnútri zložky, pre každý obrázok:

| Súbor                                                   | Obsah                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | Vykreslený obrázok, pixel po pixeli tak, ako ho zobrazil prehliadač |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | Sprievodný súbor s gradientnou lištou, ak je požadovaný                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | Sprievodný súbor s indexovým histogramom, ak je požadovaný                  |

Obe doplnkové súbory sú vždy uložené v **plnom rozlíšení**, aj keď je hlavný obrázok blokovo priemerný: veľkosť bloku zodpovedá rozlíšeniu displeja a oba doplnkové súbory obsahujú skutočné hodnoty indexu na každý pixel. Zároveň obsahujú viac informácií ako verzie zobrazené na obrazovke – obe uvádzajú okno rozťahovania _aj_ skutočné minimálne a maximálne hodnoty údajov, takže uložená legenda je čitateľná aj o niekoľko mesiacov neskôr bez nutnosti otvárať projekt.

### Priebeh a výsledky

Export celého projektu trvá niekoľko minút, takže program informuje o priebehu cez kanál v reálnom čase namiesto toho, aby blokoval systém:

* Ukazovateľ priebehu zobrazuje `current / total` a súbor, ktorý sa práve zapisuje
* Po dokončení panel oznámi, koľko obrázkov bolo exportovaných, koľko bolo preskočených a cestu k výstupnej zložke
* Preskočené obrázky sú uvedené spolu s dôvodom (zobrazí sa maximálne päť, potom nasleduje riadok „+N ďalších“). Bežným dôvodom je vrstva, ktorá nemá kanály potrebné pre tento index
* Ak by **žiadny** obrázok v projekte nemohol použiť tento index, spustenie nahlási zlyhanie namiesto toho, aby vám zanechalo prázdnu zložku

V danom okamihu beží len jeden export v sandboxe. Spustenie druhého exportu, kým je prvý v chode, je odmietnuté jasnou správou, aby sa zabránilo tomu, že by sa dva behy navzájom prekrývali pri spracovaní toho istého projektového súboru.

### Mriežka vyberie spustenie

Každé dokončené spustenie sa zobrazí ako samostatné tlačidlo na paneli nástrojov [mriežky obrázkov](image-grid.md) s označením `<IndexName> <Index|LUT> <NNN>`. Takto môžete porovnávať spustenia: dvakrát exportujte s rôznymi gradientmi alebo prahovými hodnotami a potom prepínajte medzi dvoma tlačidlami v mriežke.

***

## Vlastné vzorce indexov (Chloros+)

{% hint style="info" %}
**Kde ich vytvoriť**: v bočnom paneli Sandboxu alebo v**Nastaveniach projektu** pred spracovaním. Obe možnosti zapisujú do rovnakého zoznamu na úrovni projektu.
{% endhint %}

1. Otvorte kalkulačku vlastných vzorcov z roletového menu vzorcov indexu (vyžaduje prihlásenie s oprávneným predplatným Chloros+)
2. Napíšte vzorec pomocou **symbolov pásmových slotov** `x`, `y`, `z`, `a`, `b`, `c` — nie názvy pásiem
3. Dostupné operátory: `+`, `-`, `*`, `/`, `^` a `()` na zoskupovanie
4. Dostupné funkcie: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Pomenujte ho a uložte – objaví sa v spodnej časti roletového menu vzorcov a jeho sloty priradíte ťahaním kruhov kanálov, presne tak ako pri vstavanej predvolbe

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Vlastné vzorce sú dostupné len v grafickom rozhraní.** Možnosť CLI/SDK `--indices` rozširuje 22 názvov vstavaných predvolieb a bez upozornenia preskočí všetko ostatné, vrátane vašich vlastných vzorcov. Ak chcete vlastný vzorec spracovať hromadne, nakonfigurujte ho v nastaveniach projektu a spustite spracovanie, alebo použite export „Uplatniť na všetky obrázky projektu“ v prostredí Sandbox.
{% endhint %}

***

## Riešenie problémov

### „Táto vrstva nemá kanály, ktoré tento index vyžaduje“

Vzorec číta pozíciu kanála, ktorú aktuálna vrstva nemá — napríklad index s tromi slotmi v súbore s jedným alebo dvoma kanálmi. Prepnite na viacpásmovú vrstvu (reflektancia alebo debayering) alebo vyberte index, ktorý zodpovedá filtru vašej kamery.

### „Nepodarilo sa nadviazať spojenie s backendom na spracovanie obrázkov“

Backend neodpovedá. Skontrolujte kartu Log; ak sa backend reštartuje, Sandbox sa sám obnoví, akonáhle bude opäť k dispozícii.

### Obraz sa nezmenil, keď som pretiahol kruh

Vzorec ešte nie je kompletný. Neúplný vzorec sa spracováva ako bežný stav počas pretiahnutia — nič sa nerenderuje a nič sa nehlási ako chyba. Vyplňte všetky polia, ktoré vzorec používa.

### Celý obrázok má jednu farbu

Vaše okno klipu je pravdepodobne ďaleko mimo rozsahu údajov. Stlačte **AUTO**, aby sa okno prispôsobilo 2. alebo 98. percentilu, alebo zapnite histogram**INDEX**, aby ste videli, kde sa údaje skutočne nachádzajú.

### Exportované farby nezodpovedajú tomu, čo som videl

Mali by – exportná cesta je zámerným zrkadlovým obrazom živého náhľadu, vrátane alfa kanála v režime orezávania, a priemerné hodnoty blokov sa uplatňujú _po_ sfarbení presne tak, ako to robí prehliadač. Ak sa líšia, skontrolujte, či sa veľkosť bloku GSD medzi prezeraním a exportom nezmenila.

***

## Ďalšie kroky

* [**Vrstvy obrázku**](image-layers.md) — na ktorej vrstve spustiť index a čo znamenajú jej hodnoty
* [**Otvorenie snímky na celú obrazovku**](opening-an-image-full-screen.md) — podrobné informácie o zobrazení kurzora, histogramu a ovládaní GSD
* [**Vzorce multispektrálnych indexov**](../project-settings/multispectral-index-formulas.md) — všetky prednastavenia na každom povrchu
* [**Nastavenia projektu**](../project-settings/project-settings.md) — uloženie nájdených nastavení do spracovateľského cyklu

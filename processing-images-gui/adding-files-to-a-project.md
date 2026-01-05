# Pridávanie súborov do projektu

Po vytvorení alebo otvorení projektu v Chloros je ďalším krokom pridanie multispektrálnych snímok, aby ste mohli začať so spracovaním. Kartu Prehliadač súborov<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> umožňuje jednoduché importovanie snímok a správu dátového súboru.

## Prístup k prehliadaču súborov

1. Otvorte alebo vytvorte projekt v Chloros
2. Kliknite na ikonu **Prehliadač súborov** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> v ľavom bočnom paneli.
3. Panel Prehliadač súborov zobrazí zoznam súborov vášho projektu.

{% tip style=&quot;info&quot; %}
**Podporované typy súborov**: Chloros podporuje obrazové súbory RAW+JPG a JPG z fotoaparátov MAPIR Survey3W a Survey3N. Odporúčajú sa iba súbory RAW+JPG.
{% endhint %}

***

## Pridávanie obrázkov do projektu

Existujú dva hlavné spôsoby pridávania obrázkov do projektu:

### Spôsob 1: Pridať súbory

Túto možnosť použite na importovanie jednotlivých obrazových súborov alebo malého výberu súborov.

1. Kliknite na tlačidlo **„Pridať súbory“** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> v hornej časti panela prehliadača súborov.
2. Prejdite do priečinka obsahujúceho vaše obrázky.
3. Vyberte jeden alebo viacero obrazových súborov (podržte kláves **Ctrl**, aby ste vybrali viacero súborov).
4. Kliknite na **„Otvoriť“**, aby ste importovali vybrané súbory.

### Spôsob 2: Pridať priečinok

Túto možnosť použite na importovanie všetkých obrázkov z priečinka naraz.

1. Kliknite na tlačidlo **„Pridať priečinok“** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> v hornej časti panela prehliadača súborov.
2. Prejdite do priečinka obsahujúceho obrázky z vašej snímkovej relácie a vyberte ho.
3. Kliknite na **„Vybrať priečinok“**, aby ste importovali všetky podporované obrázky z tohto priečinka.***

## Vysvetlenie tabuľky prehliadača súborov

Po importe sa obrázky zobrazia v tabuľke s nasledujúcimi stĺpcami:

### Názov súboru

* Pôvodný názov súboru z fotoaparátu
* Zachováva konvenciu názvov fotoaparátu (napr. IMG\_0001.RAW)

### Časová pečiatka

* Dátum a čas zachytenia obrázku
* Extrahované z metadát EXIF obrázku
* Používa sa na synchronizáciu PPK a detekciu kalibračného cieľa

### Model fotoaparátu

* Automaticky detegovaná konfigurácia fotoaparátu a filtra
* Príklady: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Používa sa na aplikovanie správnych profilov spracovania

### Stĺpec cieľa (začiarkavacie políčko)

* Začiarknite toto políčko pre obrázky, ktoré obsahujú kalibračné ciele
* Výrazne urýchľuje detekciu cieľov počas spracovania
* Podrobnosti nájdete v časti [Výber cieľových obrázkov](choosing-target-images.md)

***

## Správa súborov vo vašom projekte

### Odstránenie súborov

Ak chcete odstrániť nepotrebné obrázky z vášho projektu:

1. Vyberte jeden alebo viacero obrázkov v tabuľke prehliadača súborov
2. Kliknite na tlačidlo **„Odstrániť vybrané“** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> .
3. Potvrďte odstránenie (súbory sa neodstránia z disku, iba z projektu).

### Triedenie a filtrovanie

* **Triedenie podľa stĺpca**: Kliknutím na akýkoľvek záhlavie stĺpca môžete obrázky triediť.
* **Triedenie podľa časovej pečiatky**: Užitočné pre organizovanie chronologických sekvencií záberov.
* **Filter modelu fotoaparátu**: Zoskupte obrázky podľa typu fotoaparátu, ak používate viacero fotoaparátov.***

## Náhľad obrázkov

### Zobrazenie celého obrázku

Kliknite na ľubovoľnú miniatúru obrázku v prehliadači súborov, aby sa zobrazil v hlavnej oblasti náhľadu:

1. Obrázok sa zobrazí v strednom paneli náhľadu.
2. Pomocou ovládacích prvkov zväčšenia/zmenšenia môžete prezerať detaily obrázku.
3. Medzi obrázkami môžete prechádzať pomocou klávesov so šípkami.

### Rýchla navigácia

* **Predchádzajúci obrázok**: Kliknite na šípku vľavo alebo stlačte klávesu ←
* **Ďalší obrázok**: Kliknite na šípku vpravo alebo stlačte klávesu →
* **Zväčšenie/zmenšenie**: Použite koliesko myši alebo tlačidlá zväčšenia
* **Posun**: Po zväčšení kliknite a ťahajte po obrázku***

## Spracovanie duplicitných súborov

Chloros automaticky detekuje a ignoruje duplicitné súbory:

* Súbory s identickými názvami sú preskočené.
* Zabraňuje náhodnému dvojitému spracovaniu.
* Pri detekcii duplicitných súborov sa zobrazí varovná správa.

{% hint style=&quot;warning&quot; %}
**Dôležité**: Pred importom nepremenujte ani nemodifikujte pôvodné obrazové súbory. Chloros sa spolieha na pôvodné názvy súborov a metadáta pre správne spracovanie.
{% endhint %}

***

## Zmiešané súbory údajov z kamery

Ak váš projekt obsahuje obrázky z viacerých kamier MAPIR:

1. Chloros automaticky rozpozná každý model kamery.
2. Každý typ kamery sa spracováva pomocou príslušného kalibračného profilu.
3. Prehliadač súborov zobrazuje model kamery v stĺpci Model kamery.
4. Spracovanie uplatňuje správne nastavenia pre každý typ kamery.

**Príklad scenára**: Survey3W RGN + Survey3N OCN konfigurácia s dvoma kamerami.***

## Osvedčené postupy

### Usporiadajte pred importom

* Uložte kalibračné cieľové obrázky do rovnakého priečinka ako snímky z prieskumu.
* Zachovajte pôvodnú štruktúru priečinkov z fotoaparátu/SD karty.
* Nemiešajte dátové súbory z rôznych relácií v jednom projekte.

### Pomenovanie súborov

* Zachovajte pôvodné názvy súborov fotoaparátu (IMG\_0001.RAW atď.).
* Pred importom súbory nepremenúvajte.
* Pôvodné názvy obsahujú dôležité metadáta.

### Kalibračné cieľové obrázky

* Vždy zahrňte 1–2 kalibračné cieľové obrázky na reláciu.
* Zachyťte ciele pred a po relácii zachytávania.
* Umiestnite ciele do rovnakých svetelných podmienok ako oblasť zachytávania.
* Označte cieľové obrázky pomocou začiarkavacieho políčka Cieľ, aby ste urýchlili spracovanie.

***

## Bežné problémy a riešenia

### Obrázky sa nezobrazujú po importe

**Možné príčiny:**

* Nepodporovaný formát súboru (len RAW+JPG a JPG z fotoaparátov MAPIR)
* Obrázky pochádzajú z fotoaparátov, ktoré nie sú MAPIR (pozri [Podporované fotoaparáty](../supported-cameras.md))
* Poškodenie súboru alebo neúplný prenos z SD karty

**Riešenie**: Overte kompatibilitu formátu súboru a modelu fotoaparátu.

### Model fotoaparátu nie je detegovaný

**Možné príčiny:**

* Upravené metadáta EXIF.
* Obrázky upravené v externom softvéri.
* Neúplný prenos súborov.

**Riešenie**: Znovu importujte pôvodné, neupravené súbory z fotoaparátu/SD karty.

### Chýbajúce časové značky

**Možné príčiny:**

* Nesprávne nastavené hodiny fotoaparátu
* EXIF údaje odstránené externým softvérom

**Riešenie**: Overte, či boli nastavenia času fotoaparátu počas snímania správne.***

## Ďalšie kroky

Po importe súborov:

1. **Skontrolujte zoznam súborov** – Uistite sa, že sa všetky obrázky načítali správne.
2. **Skontrolujte modely fotoaparátov** – Overte správne rozpoznanie fotoaparátu.
3. **Označte cieľové obrázky** – pozrite si [Výber cieľových obrázkov](choosing-target-images.md)
4. **Upravte nastavenia** – nakonfigurujte možnosti spracovania v [Nastaveniach projektu](adjusting-project-settings.md)
5. **Spustite spracovanie** – pozrite si časť [Spustenie spracovania](starting-the-processing.md).

Podrobné informácie o konfigurácii projektu nájdete v časti [Úprava nastavení projektu](adjusting-project-settings.md).

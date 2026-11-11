# Grafické rozhranie: Projekty Program

Chloros vám umožňuje vytvárať projekty, ktoré je možné v budúcnosti opäť otvoriť. Projekt je obyčajný priečinok (nachádzajúci sa vo vašom projektovom priečinku), ktorý obsahuje:

* `project.json` — nastavenia projektu, zoznam súborov a nastavenia zobrazenia
* `cameras.json` — kamery a senzorové polia pripojené počas otvorenia projektu spolu s ich nastaveniami
* `sensors.json` — svetelné senzory DAQ pripojené počas otvorenia projektu, vrátane priradení kamery↔senzora
* vaše snímky, záznamy `.daq` a zložky so spracovanými výstupmi

Neexistuje žiadny proprietárny formát projektových súborov — projekt tvoria priečinok a jeho súbory JSON, vďaka čomu je možné projekty ľahko kopírovať, archivovať a ovládať prostredníctvom [CLI](CLI.md) alebo [Python SDK](api-python-sdk.md).

Pokiaľ to nezmeníte v [Nastaveniach projektu](project-settings/project-settings.md), zložka projektu sa nachádza v zložke Dokumenty pod názvom `Documents\MAPIR\Chloros Projects` (`~/Documents/MAPIR/Chloros Projects` v systéme Linux).

{% hint style="warning" %}
**Windows: používajte krátke názvy zložiek projektov, pokiaľ nie sú zapnuté dlhé cesty.**V systéme Windows sú dlhé cesty štandardne vypnuté, a preto žiadna cesta k súboru nesmie byť dlhšia ako 260 znakov. Spracované produkty sa nachádzajú približne 130 znakov pod zložkou projektu, takže od verzie 1.3.0 program Chloros odmietne príliš hlbokú zložku projektu – pri jej vytvorení, otvorení alebo pri spustení spracovania – namiesto toho, aby bežal a nezaznamenal žiadne údaje. Správa uvádza najdlhšiu zložku projektu, ktorá sa zmestí. Použite kratšiu zložku projektu alebo zapnite dlhé cesty v systéme Windows (možnosť inštalátora**Povoliť dlhé cesty v systéme Windows**, ktorá je predvolene zaškrtnutá) a reštartujte program Chloros. Pozrite si časť [Spustenie spracovania](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).
{% endhint %}

## Nový projekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>V hlavnom menu vyberte položku „Nový projekt“ a zadajte jedinečný názov pre svoj projekt.

Ak ste si uložili nejaké šablóny projektov, pod poľom na zadanie názvu sa zobrazí roletové menu **Vybrať šablónu** — výberom jednej z nich spustíte nový projekt s nastaveniami danej šablóny. Šablóny sa ukladajú v [Nastaveniach projektu](project-settings/project-settings.md): do poľa „Názov šablóny projektu“ zadajte názov a kliknite na ikonu uloženia.

## Otvoriť projekt

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>V časti „Otvoriť projekt“ sa zobrazuje zoznam všetkých projektov vo vašej projektovej zložke, pričom v spodnej časti sa nachádza položka <strong>Otvoriť projektovú zložku</strong></p></figcaption></figure>

Vyberte možnosť „Otvoriť projekt“, aby sa zobrazil zoznam existujúcich projektov v zložke projektov. Ak neexistujú žiadne projekty, sekundárne bočné menu sa neotvorí. Na vyššie uvedenej fotografii môžete vidieť niektoré projekty vytvorené prostredníctvom grafického používateľského rozhrania (t1, t2, t3). Projekty s názvami typu DATE\_TIME boli vytvorené prostredníctvom CLI podľa predvoleného schématu pomenovania projektov. Kliknutím na akýkoľvek názov projektu ho otvoríte.

Kliknutím na tlačidlo „Otvoriť zložku projektu“ otvoríte prehliadač súborov vášho počítača v ceste k projektu. Cestu k projektu môžete upraviť v [Nastaveniach projektu](project-settings/project-settings.md).

Ak bol od posledného otvorenia projektu presunutý alebo odstránený akýkoľvek zdrojový obrazový súbor projektu, Chloros namiesto otvorenia prázdnej mriežky zobrazí dialógové okno s presným zoznamom chýbajúcich súborov.

## Duplikovať projekt

Táto funkcia je dostupná po otvorení projektu. Zvoľte „Duplikovať projekt“, aby ste skopírovali aktuálny projekt pod novým názvom — Chloros navrhne najbližší voľný názov (napr. „MôjProjekt (2)“) — a duplikát sa okamžite otvorí.

## Pridať súbory

Po otvorení projektu vyberte z hlavného menu položku „Pridať súbory“, aby ste do aktuálneho projektu pridali jednotlivé obrazové súbory. Táto funkcia zodpovedá funkcii pridávania v prehliadači súborov, ale pre väčšie pohodlie je prístupná priamo z hlavného menu.

## Pridať zložku

Po otvorení projektu vyberte z hlavného menu položku „Pridať zložku“, aby ste do aktuálneho projektu pridali zložky s obrázkami. Môžete vybrať viacero zložiek naraz. Duplicitné súbory sa ignorujú.

## Spustiť / Zastaviť spracovanie

Po pridaní súborov do projektu sa v hlavnom menu sprístupní položka „Spustiť spracovanie“. Ide o rovnakú akciu ako kliknutie na tlačidlo Prehrať/Spustiť v hornom záhlaví. Počas spracovania sa táto položka menu zmení na „Zastaviť spracovanie“, čo vám umožní zastaviť spracovateľský reťazec.

## Pripojiť k kamere / Pripojiť k svetelnému senzoru

V spodnej časti hlavného menu sa nachádzajú dve hardvérové skratky, ktoré sú k dispozícii bez ohľadu na to, či je projekt otvorený:

* **Pripojiť k kamere** — otvorí [kartu Kamery](lattice/), kde môžete pripojiť kameru alebo maticu LATTICE.
* **Pripojiť sa k svetelnému senzoru** — otvorí [kartu Svetelné senzory](daq/), kde môžete pripojiť svetelný senzor DAQ.

Pripojenie hardvéru pri otvorenom projekte sa uloží do projektu (pozri nižšie). Bez otvoreného projektu sú pripojenia platné len počas danej relácie.

{% hint style="info" %}
Položky ponuky Pridať súbory, Pridať priečinok a Spustiť/Zastaviť spracovanie sú viditeľné alebo aktívne len vtedy, keď je otvorený projekt a boli pridané súbory. Poskytujú rýchly prístup k akciám, ktoré sú k dispozícii aj prostredníctvom bočného panela prehliadača súborov a tlačidiel v hlavičke.
{% endhint %}

## Projekty si pamätajú váš hardvér

Novinka vo verzii 1.2.0: projekt si uchováva hardvér, ktorý pripojíte, pokiaľ je otvorený. Kamery a polia (s nastaveniami pre každú kameru, názvami, farbami a rozložením mriežky) sa automaticky zaznamenávajú do súboru `cameras.json` a svetelné senzory (s názvami, farbami a priradeniami ku kamerám) do súboru `sensors.json` — automaticky, počas vašej práce.

Keď projekt **znova otvoríte**, Chloros sa hneď nedotkne žiadneho hardvéru. Každá polovica sa opäť pripojí pri prvom otvorení karty, ku ktorej patrí:

* Otvorením karty **Kamery** sa opäť pripoja uložené kamery a polia a znovu sa uplatnia ich uložené nastavenia.
* Otvorenie karty **Svetelné senzory**opätovne pripojí uložené senzory DAQ a znovu uplatní ich uložené nastavenia, vrátane krytu. Senzor nastavený na**Žiadne (holý senzor)** v ponuke Cap zostane holý. Projekt uložený v skoršej verzii programu Chloros zaznamenal holý senzor ako žiadnu voľbu, takže sa obnoví s korekciou kosínusu slnečného žiarenia; nastavte ho opäť na Žiadne a uložte projekt.

Týmto spôsobom otvorenie projektu len na prehliadanie alebo export obrázkov nikdy nespustí streamovanie z kamier. Ak sa pri otvorení karty nedá nájsť uložené zariadenie, dialógové okno vám oznámi, ktoré zariadenia nie sú k dispozícii, aby ste ich mohli opäť pripojiť alebo odstrániť.

## Záznamy DAQ a súbory .daq v projekte

* Záznamy `.daq` vytvorené počas otvorenia projektu (z karty Svetelné senzory alebo počas snímania) sa **automaticky pridajú do projektu**.
* Importované súbory `.daq` a všetky záznamy projektu sú uvedené v časti **Svetelný senzor DAQ** v [Nastaveniach projektu](project-settings/project-settings.md), pričom každý z nich má svoj profil korekcie odleskov.
* Počas spracovania poskytujú súbory `.daq` projektu osvetlenie smerujúce nadol pre produkty odrazivosti — pozri [Formáty výstupných obrázkov](output-image-formats.md).

## Spustenie uloženého projektu bez grafického rozhrania

Uložený projekt je možné spustiť bez grafického rozhrania:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` pracuje s cestou k zložke projektu — pozri [Referenčnú príručku CLI](reference/cli-reference.md).
* **SSDK**: `chloros_sdk.open_project(path)` vráti identifikátor projektu; `connect_all()` aktivuje všetky uložené kamery, polia a svetelné senzory s ich uloženými nastaveniami (svetelné senzory prostredníctvom bežiaceho backendu Chloros) — pozri [Referenciu SDK](reference/sdk-reference.md).

# Grafické rozhranie: Projekty

Chloros vám umožňuje vytvárať projekty, ktoré je možné v budúcnosti opäť otvoriť. Projekt je obyčajný priečinok (nachádzajúci sa vo vašom priečinku „Project Folder“), ktorý obsahuje:

* `project.json` — nastavenia projektu, zoznam súborov a preferencie zobrazenia
* `cameras.json` — kamery a senzorové polia pripojené počas otvorenia projektu spolu s ich nastaveniami
* `sensors.json` — svetelné senzory DAQ pripojené počas otvorenia projektu, vrátane priradení kamery↔senzora
* vaše snímky, záznamy `.daq` a zložky so spracovanými výstupmi

Neexistuje žiadny proprietárny formát projektových súborov — zložka a jej súbory JSON tvoria projekt, čo tiež uľahčuje kopírovanie, archiváciu a prenášanie projektov z [CLI](CLI.md) alebo [Python SDK](api-python-sdk.md).

## Nový projekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>V hlavnom menu vyberte možnosť „Nový projekt“ a zadajte jedinečný názov pre svoj projekt.

Ak ste si uložili nejaké šablóny projektov, pod poľom na zadanie názvu sa zobrazí roletové menu **Vybrať šablónu** — výberom jednej z nich spustíte nový projekt s nastaveniami danej šablóny. Šablóny sa ukladajú v [Nastaveniach projektu](project-settings/project-settings.md): do poľa „Názov šablóny projektu“ zadajte názov a kliknite na ikonu uloženia.

## Otvoriť projekt

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>Zoznam „Otvoriť projekt“ obsahuje všetky projekty vo vašej projektovej zložke, pričom v spodnej časti sa nachádza <strong>položka „Otvoriť projektovú zložku“</strong></p></figcaption></figure>Vyberte možnosť „Otvoriť projekt“, aby sa zobrazil zoznam existujúcich projektov v zložke projektov. Ak neexistujú žiadne projekty, sekundárne bočné menu sa neotvorí. Na vyššie uvedenej fotografii môžete vidieť niektoré projekty vytvorené prostredníctvom grafického používateľského rozhrania (t1, t2, t3). Projekty s názvami DATE\_TIME boli vytvorené programom CLI pomocou predvoleného schématu pomenovania projektov. Kliknutím na akýkoľvek názov projektu ho otvoríte.

Kliknutím na tlačidlo „Otvoriť priečinok projektu“ otvoríte prehliadač súborov vášho počítača v ceste k projektu. Cestu k projektu môžete upraviť v [Nastaveniach projektu](project-settings/project-settings.md).

Ak bol od posledného otvorenia projektu presunutý alebo odstránený akýkoľvek zdrojový obrazový súbor projektu, namiesto otvorenia prázdnej mriežky sa v Chloros zobrazí dialógové okno s presným zoznamom chýbajúcich súborov.

## Duplikovať projekt

Táto funkcia je dostupná po otvorení projektu. Zvoľte „Duplikovať projekt“, aby ste skopírovali aktuálny projekt pod novým názvom — program Chloros navrhne najbližší voľný názov (napr. „Môj projekt (2)“) — a duplikát sa okamžite otvorí.

## Pridať súbory

Po otvorení projektu vyberte z hlavného menu možnosť „Pridať súbory“, aby ste do aktuálneho projektu pridali jednotlivé obrazové súbory. Táto funkcia zodpovedá funkcii pridávania v prehliadači súborov, ale pre väčšie pohodlie je prístupná priamo z hlavného menu.

## Pridať zložku

Po otvorení projektu vyberte z hlavného menu položku „Pridať zložku“, aby ste do aktuálneho projektu pridali zložky s obrázkami. Môžete vybrať viacero zložiek naraz. Duplicitné súbory sa ignorujú.

## Spustiť / Zastaviť spracovanie

Po pridaní súborov do projektu sa v hlavnom menu sprístupní položka „Spustiť spracovanie“. Ide o rovnakú akciu ako kliknutie na tlačidlo Prehrať/Spustiť v hornom záhlaví. Počas spracovania sa táto položka menu zmení na „Zastaviť spracovanie“, aby ste mohli proces zastaviť.

## Pripojiť k kamere / Pripojiť k svetelnému senzoru

V spodnej časti hlavného menu sa nachádzajú dve hardvérové skratky, ktoré sú k dispozícii bez ohľadu na to, či je projekt otvorený:

* **Pripojiť k kamere** — otvorí [kartu Kamery](lattice/), kde môžete pripojiť kameru alebo maticu LATTICE.
* **Pripojiť sa k svetelnému senzoru** — otvorí [kartu Svetelné senzory](daq/), kde môžete pripojiť svetelný senzor DAQ.

Pripojenie hardvéru pri otvorenom projekte uloží toto pripojenie do projektu (pozri nižšie). Bez otvoreného projektu sú pripojenia platné len počas aktuálnej relácie.

{% hint style="info" %}
Položky ponuky Pridať súbory, Pridať priečinok a Spustiť/Zastaviť spracovanie sú viditeľné alebo aktívne len vtedy, keď je otvorený projekt a boli pridané súbory. Poskytujú rýchly prístup k akciám, ktoré sú k dispozícii aj prostredníctvom bočného panela prehliadača súborov a tlačidiel v hlavičke.
{% endhint %}

## Projekty si pamätajú váš hardvér

Novinka vo verzii 1.2.0: projekt si uchováva hardvér, ktorý pripojíte, pokiaľ je otvorený. Kamery a polia (vrátane nastavení pre každú kameru, mien, farieb a rozloženia mriežky) sa automaticky zaznamenávajú do súboru `cameras.json` a svetelné senzory (vrátane mien, farieb a priradení ku kamerám) do súboru `sensors.json` — automaticky, počas vašej práce.

Keď projekt **znovu otvoríte**, súbor Chloros sa s hardvérom okamžite nespojí. Každá polovica sa opäť pripojí pri prvom otvorení karty, ku ktorej patrí:

* Otvorenie karty **Kamery** opätovne pripojí uložené kamery a polia a znovu uplatní ich uložené nastavenia.
* Otvorenie karty **Svetelné senzory** opätovne pripojí uložené senzory DAQ.

Týmto spôsobom otvorenie projektu len na prehliadanie alebo export obrázkov nikdy nespustí streamovanie z kamier. Ak sa pri otvorení karty nedá nájsť uložené zariadenie, dialógové okno vás informuje, ktoré zariadenia nie sú k dispozícii, aby ste ich mohli opätovne pripojiť alebo odstrániť.

## Záznamy DAQ a súbory .daq v projekte

* Záznamy `.daq` vytvorené počas otvorenia projektu (z karty Svetelné senzory alebo počas snímania) sa **automaticky pridajú do projektu**.
* Importované súbory `.daq` a všetky záznamy projektu sú uvedené v časti **Svetelný senzor DAQ** v [Nastaveniach projektu](project-settings/project-settings.md), pričom každý z nich má svoj profil korekcie osvetlenia.
* Počas spracovania poskytujú súbory `.daq` projektu osvetlenie smerujúce nadol pre produkty odrazivosti — pozri [Formáty výstupných obrázkov](output-image-formats.md).

## Spustenie uloženého projektu bez grafického rozhrania

Uložený projekt je možné spustiť bez grafického používateľského rozhrania:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` pracuje s cestou k priečinku projektu — pozri [Referenciu CLI](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` vráti identifikátor projektu; `connect_all()` aktivuje všetky uložené kamery a senzory s ich uloženými nastaveniami — pozri [Referenčná príručka k SDK](reference/sdk-reference.md).

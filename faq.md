---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Často kladené otázky

<details>

<summary>Môžem v programe Chloros spracovávať snímky z kamier, ktoré nie sú značky MAPIR?</summary>

Nie, Chloros podporuje iba spracovanie snímok z kamier MAPIR — z radov Survey3 a LATTICE. Ďalšie informácie nájdete v zozname [podporovaných modelov kamier](supported-cameras.md). Spracovanie záznamov z iných kamier ponúkame v službe MAPIR Cloud, kompletný zoznam nájdete [tu](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Podporuje program Chloros kamery LATTICE?</summary>

Áno. Chloros 1.3.0 podporuje kamerové moduly LATTICE M3C a M3M v celom rozsahu: **ovládanie v reálnom čase**— vyhľadávanie, pripájanie, náhľad a snímanie prostredníctvom karty „Kamery“ v grafickom používateľskom rozhraní (`chloros-cli lattice`) alebo prostredníctvom Python SDK, vrátane synchronizovaných sústav viacerých kamier s časovou synchronizáciou PTP — a**kompletné radiometrické spracovanie** snímok (surové → debayered → žiarivosť → odrazivosť → index). Pozrite si [Podporované kamery](supported-cameras.md) a [príručku LATTICE](lattice/README.md).

</details>

<details>

<summary>Môžem kalibrovať svoje snímky na odrazivosť bez kalibračného terča?</summary>

**Survey3:** Nie. Bez snímky kalibračného terčíka, ktorá bola nasnímaná v čase snímania snímok bez terčíka, nebudete môcť priradiť hodnoty pixelov snímky k známemu percentu odrazivosti. Ak zároveň nezahrniete záznam zo svetelného senzora MAPIR, spektrum okolitého osvetlenia sa nezmeria a výsledky odrazivosti nebudú presné.

**LATTICE:** Áno. Odrazivosť je možné odvodiť od intenzity žiarenia smerujúceho nadol, meranej svetelným senzorom DAQ namiesto panelu (ρ = π·L/E). Ak sa v zábere nachádza cieľ, ktorý prešiel kontrolou kvality (QA), stáva sa štandardne absolútnou referenciou (`--reflectance-source auto`), pokiaľ neprešiel s vlajkou, kedy sa cieľ a reťazce DAQ porovnávajú na základe nameraného skóre. Jedna výnimka: „Odrazivosť F988 sa kalibruje pomocou panelu odrazivosti v scéne: pásmo leží mimo kalibrovaného rozsahu svetelného senzora DAQ, takže Chloros použije váš najnovší záznam z panelu a zachová ho medzi jednotlivými pozorovaniami panelu.“ Pozrite si [Kalibračné ciele](calibration-targets.md).

</details>

<details>

<summary>Potrebujem svetelný senzor DAQ?</summary>

Nie pre žiarivosť: Exporty žiarivosti z LATTICE pochádzajú z továrenskej radiometrickej kalibrácie každej kamery a nepotrebujú ani senzor DAQ, ani cieľ. Pre **odrazivosť**potrebujete referenciu pre okolité osvetlenie – buď meranie dopadajúceho svetla senzorom DAQ, alebo kalibračný cieľ v zábere. Senzor DAQ vám umožňuje získať kalibrovanú odrazivosť**bez umiestňovania akýchkoľvek panelov do scény**. Nahraté súbory `.daq` sa automaticky priraďujú k vašim snímkam podľa časovej pečiatky. Pozrite si [Kalibračné terče](calibration-targets.md) a [Referenčné informácie o CLI](reference/cli-reference.md).

</details>

<details>

<summary>Môžem používať Chloros s AI asistentom (Claude, ChatGPT atď.)?</summary>

Áno — táto príručka a CLI/SDK sú na to navrhnuté:

* Kompletný index príručky je dostupný na adrese `https://mapir.gitbook.io/chloros/llms.txt`, aby AI asistenti mohli nájsť každú stránku.
* Surový markdown každej stránky je dostupný na jej URL-e s malými písmenami, ku ktorej je pridané `.md` (napríklad `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Referenčná príručka k CLI](reference/cli-reference.md) a [Referenčná príručka k SDK](reference/sdk-reference.md) sú napísané pre použitie veľkými jazykovými modelmi (LLM): presné príznaky, predvolené nastavenia, sémantika ukončenia a príkazy, ktoré je možné skopírovať a vložiť.

Informácie o tom, ako nasmerovať svojho asistenta na Chloros, nájdete v časti [AI asistenti](ai-assistants.md).

</details>

<details>

<summary>Kam sa ukladajú moje spracované výstupné súbory?</summary>

Výstupy sa ukladajú do projektovej zložky, zoskupené podľa kamery a následne podľa formátu súboru:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **zložka-kamery** — `LATT-<sensor>-<lens>-F<filter>` pre LATTICE, `<model>_<filter>` (napr. `Survey3N_RGN`) pre Survey3
* **zložka formátu** — `tiff16`, `tiff8`, `png8`, `jpg8` alebo `tiff32`
* **zložky produktov** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (vždy pod `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — iba multispektrálne snímky LATTICE: masky na úrovni jednotlivých pixelov sa nachádzajú vedľa zložiek s produktmi, pozri nižšie**Exportované súbory si zachovávajú názov zdrojového súboru — produkt identifikuje zložka, nie prípona súboru.**Pri použití CLI sa zložka projektu vytvorí vo vašej predvolenej zložke projektov (`Documents/MAPIR/Chloros Projects`, pokiaľ ste ju nezmenili), pokiaľ nezadáte `-o`; relatívna cesta `-o` sa berie relatívne voči zložke, z ktorej spúšťate príkaz. Všimnite si, že spustenie s parametrom `chloros-cli process`, ktoré požiadalo o produkty, ale žiadne nezapísalo, vypíše `Processing finished but wrote no image products.` a**ukončí sa s nenulovým kódom**, takže skripty to môžu zistiť. Pozrite si [Formáty výstupných obrázkov](output-image-formats.md) a [Referenčnú príručku CLI](reference/cli-reference.md).

</details>

<details>

<summary>Čo sú to priečinky clamp_masks a súbory *_clampmask.tif?</summary>

Novinka vo verzii 1.3.0. Produkty žiarivosti a odrazivosti z multispektrálnej kamery LATTICE M3C prechádzajú separáciou pásiem 3×3 a ak sa v niektorom pásme vytvorí záporná hodnota, program Chloros túto hodnotu ořízne na nulu. Produkt s aspoň jedným orezaným pixelom dostane jednopásmovú masku typu `uint8`, ktorá označuje, ktoré pixely a pásma boli orezané (bit 0 = kanál R produktu, bit 1 = G, bit 2 = B). Masky sa ukladajú do zložky `clamp_masks/` **vedľa** zložiek produktov — napríklad `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — a XMP tag produktu `Chloros:UnmixClampMask` uvádza názov jeho masky (alebo uvádza `none`).

Nie sú to obrázky scény: vlastný import programu Chloros ich preskakuje a keďže sa nikdy nenachádzajú vo vnútri zložky produktu, môžete zložku produktu odovzdať programom Pix4D alebo Metashape tak, ako je. Pozrite si [Masky Clamp](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Prečo Chloros hlási: „Zložka projektu je pre tento počítač príliš hlboká“?</summary>

Toto sa týka iba systému Windows. Systém Windows je dodávaný s vypnutou podporou dlhých ciest, pričom žiadna cesta k súboru nesmie byť dlhšia ako 260 znakov. Produkty Chloros sa nachádzajú približne 130 znakov pod zložkou projektu, takže Chloros pred spracovaním odmietne hlbšiu zložku projektu — namiesto toho, aby sa spustil a nezapísal nič, ako sa to mohlo stať pred verziou 1.3.0. Správa uvádza najdlhšiu zložku projektu, ktorá sa zmestí.

Problém vyriešte jedným z nasledujúcich spôsobov:

* Použite kratšiu zložku projektu (položka **Pracovný adresár** v Nastaveniach projektu alebo príkazy `-o` / `chloros-cli set-project-folder` pre rozhranie CLI).
* Zapnite podporu dlhých ciest v systéme Windows a reštartujte Chloros. Túto funkciu zabezpečuje voľba **Povoliť dlhé cesty v systéme Windows** v inštalátore Chloros (štandardne zaškrtnutá); ak ste ju odškrtli, spustite inštalátor znovu.

Pozrite si [Spustenie spracovania](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Môžem svoje obrázky upraviť pred spracovaním v programe Chloros?</summary>

Nie. Program Chloros predpokladá, že vstupné údaje neboli upravené. Nemenite názvy súborov.

</details>

<details>

<summary>Môžem nastaviť svoje kamery MAPIRSurvey3 na automatickú expozíciu a spracovať snímky v programe Chloros?</summary>

Nie. Datasety snímok z kamery Survey3 musia mať pevnú/uzamknutú expozíciu, takže nesmie byť použitá automatická rýchlosť uzávierky ani automatické nastavenie ISO. Všetky snímky z rovnakého modelu kamery musia mať identickú rýchlosť uzávierky a hodnotu ISO (expozíciu).

Fotoaparáty LATTICE toto obmedzenie nemajú: Chloros riadi ich expozíciu v reálnom čase (Smart AE) a pri každom zábere zaznamenáva skutočne použitú expozíciu a zosilnenie, čo zohľadňuje radiometrický proces.

</details>

<details>

<summary>Môže program Chloros spracovávať alebo analyzovať ortomozaické snímky?</summary>

Nie. Podporované sú iba jednotlivé snímky z kamery MAPIR, nie zložené snímky, ako je ortomozaická mapa.

</details>

<details>

<summary>Ako môžem urýchliť krok detekcie cieľov v programe Chloros?</summary>

V tabuľke prehliadača súborov môžete v pravom stĺpci vopred vybrať cieľové snímky, čím nariadite programu Chloros, aby hľadal kalibračné ciele iba v týchto snímkach, čo výrazne urýchli spracovanie.

</details>

<details>

<summary>Ak chcem nahrať svoje snímky do <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> mal by som ich pred nahratím spracovať v programe Chloros?</summary>

Ak plánujete nahrať snímky na našu online spracovateľskú platformu [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), neupravujte ich pred nahratím. Cloud vykoná všetky tie isté úkony a ešte viac.

</details>

<details>

<summary>Bude MAPIR niekedy podporovať funkciu X? Naozaj by som si prial, aby MAPIR ponúkal funkciu X.</summary>

Vždy nás zaujímajú spätné väzby týkajúce sa našich produktov. Ak zistíte nejaký problém s našimi produktmi alebo máte návrh, ako ich môžeme vylepšiť, prosím, [KONTAKTUJTE NÁS](https://www.mapir.camera/community/contact) a podelte sa s nami o svoje myšlienky. Väčšina nášho výskumu a vývoja sa riadi najdôležitejšími potrebami našich zákazníkov.

</details>

<details>

<summary>Je Chloros dostupný pre Linux?</summary>

Áno! Chloros 1.3.0 podporuje Linux amd64 (x86_64) a arm64 (NVIDIA Jetson JetPack 6) prostredníctvom balíkov `.deb`. CLI a Python SDK sú v systéme Linux plne podporované, vrátane ovládania kamery LATTICE v reálnom čase a senzorov DAQ. Pre Linux nie je k dispozícii grafické používateľské rozhranie (GUI) – všetka interakcia prebieha prostredníctvom [CLI](CLI.md) alebo [Python SDK](api-python-sdk.md). Podrobnosti nájdete v [Prehľade pre Linux](linux/linux-overview.md).

</details>

<details>

<summary>Môžem spustiť Chloros na NVIDIA Jetson?</summary>

Áno! Chloros podporuje **rodinu** NVIDIA Jetson Orin — Orin Nano, Orin NX a AGX Orin — s nainštalovaným JetPack 6. Staršie modely (Jetson Nano, TX2, Xavier NX) nemôžu spúšťať JetPack 6 a súčasný balík ich nepodporuje. Chloros automaticky rozpozná váš model Jetson a optimalizuje stratégiu spracovania. Balík pre Jetson využíva verziu PyTorch pre CUDA z JetPack 6, takže spracovanie prebieha na grafickom procesore (GPU) zariadenia Jetson. Pokyny na nastavenie a nasadenie nájdete v [Príručke k NVIDIA Jetson](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Optimalizuje sa Chloros automaticky pre môj hardvér?</summary>

Áno! Chloros obsahuje funkciu [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md), ktorá automaticky detekuje váš procesor (CPU), grafický procesor (GPU), pamäť RAM a (na zariadeniach Jetson) teplotné senzory. Následne vyberie optimálnu stratégiu spracovania — od `GPU_PARALLEL` na grafických procesoroch s veľkou pamäťou cez `GPU_SINGLE` na menších až po `CPU_PARALLEL` na systémoch bez grafického procesora s podporou CUDA, ktorý môže PyTorch využívať. Nie je potrebná žiadna ručná konfigurácia.

</details>

<details>

<summary>Čo je to 4-vláknový spracovateľský pipeline?</summary>

Chloros používa pre používateľov Chloros+ architektúru s 4-vláknovým spracovacím potrubím: Vlákno 1 (Detekcia) načíta obrázky a detekuje kalibračné ciele, vlákno 2 (Kalibrácia) vypočíta kalibráciu odrazivosti, vlákno 3 (Spracovanie) vykonáva GPU-akcelerované odstraňovanie bayeringu a výpočet indexu a vlákno 4 (Export) zapisuje výstupné súbory. Viac obrázkov môže byť súčasne spracovávaných v rôznych vláknach, čím sa dosahuje maximálna priepustnosť. Podrobnosti nájdete v časti [Spracovateľský kanál](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Ako spustím diagnostiku na mojej inštalácii Chloros?</summary>

Použite príkaz `selftest` na spustenie 7-krokového testu funkčnosti: verzia, dostupnosť portov, spustenie backendu, pripojenie k API (`/api/test`), informácie o systéme (`/api/system-info` — GPU/CUDA/PyTorch), prítomnosť modelu odšumovača a pripravenosť CUDA + odšumovača:

```bash
chloros-cli selftest
```

Toto je obzvlášť užitočné v systémLinux na overenie nastavenia GPU a CUDA. Na zariadení Jetson zvyčajne kód `CUDA: False` znamená, že chýbajú knižnice CUDA a cuDNN pre JetPack 6: nainštalujte ich pomocou príkazu `sudo apt install nvidia-jetpack`.

</details>

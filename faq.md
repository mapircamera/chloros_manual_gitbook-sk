---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Často kladené otázky

<details>

<summary>Môžem pomocou zariadenia Chloros spracovávať snímky z kamier, ktoré nie sú značky MAPIR?</summary>

Nie, program Chloros podporuje iba spracovanie snímok z kamier MAPIR — z radov Survey3 a LATTICE. Ďalšie informácie nájdete v zozname [podporovaných modelov kamier](supported-cameras.md). Spracovanie záznamov z iných kamier ponúkame v službe MAPIR Cloud, kompletný zoznam nájdete [tu](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Podporuje Chloros kamery LATTICE?</summary>

Áno. Chloros 1.2.0 podporuje kamerové moduly LATTICE M3C a M3M v celom rozsahu: **ovládanie v reálnom čase**— vyhľadávanie, pripájanie, náhľad a snímanie z karty „Kamery“ v grafickom používateľskom rozhraní, `chloros-cli lattice` alebo v Python SDK, vrátane synchronizovaných sústav viacerých kamier s časovou synchronizáciou PTP — a**kompletné radiometrické spracovanie** zachytených snímok (surové → debayered → žiarivosť → odrazivosť → index). Pozrite si [Podporované kamery](supported-cameras.md) a [príručku LATTICE](lattice/README.md).

</details>

<details>

<summary>Môžem kalibrovať svoje snímky na odrazivosť bez kalibračného terča?</summary>

**Survey3:** Nie. Bez snímky kalibračného terčíka, ktorá bola nasnímaná v rovnakom čase ako snímky bez terčíka, nebudete môcť priradiť hodnoty pixelov snímky k známemu percentu odrazivosti. Ak nezahrniete ani záznam zo svetelného senzora MAPIR, spektrum okolitého svetla sa nezmeria a výsledky odrazivosti nebudú presné.**LATTICE:** Áno. Odrazivosť je možné odvodiť od intenzity zosvetlenia smerom nadol, meranej svetelným senzorom DAQ namiesto panelu (ρ = π·L/E). Ak je v zábere prítomný cieľ, ktorý prešiel kontrolou kvality (QA), stáva sa štandardne absolútnou referenciou (`--reflectance-source auto`). Jedna výnimka: „Odrazivosť F988 sa kalibruje pomocou odrazivostného panelu v scéne: pásmo leží mimo kalibrovaného rozsahu svetelného senzora DAQ, takže Chloros použije váš najnovší záznam z panelu a zachová ho medzi jednotlivými pozorovaniami panelu.“ Pozrite si [Kalibračné terče](calibration-targets.md).

</details>

<details>

<summary>Potrebujem svetelný senzor DAQ?</summary>

Nie pre žiarivosť: Exportované údaje o žiarivosti z LATTICE vychádzajú z továrenskej radiometrickej kalibrácie každej kamery a nevyžadujú ani senzor DAQ, ani cieľ. Pre **odrazivosť**potrebujete referenciu pre okolité osvetlenie — buď meranie dopadajúceho svetla senzorom DAQ, alebo kalibračný cieľ v zábere. Snímač DAQ vám umožňuje získať kalibrovanú odrazivosť**bez umiestňovania akýchkoľvek panelov do scény**. Nahraté súbory `.daq` sa automaticky priraďujú k vašim snímkam podľa časovej pečiatky. Pozrite si [Kalibračné terče](calibration-targets.md) a [Referenčný materiál CLI](reference/cli-reference.md).

</details>

<details>

<summary>Môžem používať Chloros s AI asistentom (Claude, ChatGPT atď.)?</summary>

Áno — táto príručka a súbory CLI/SDK sú na to prispôsobené:

* Úplný index príručky je dostupný na adrese `https://mapir.gitbook.io/chloros/llms.txt`, aby umelá inteligencia mohla nájsť každú stránku.
* Surový kód Markdown každej stránky je dostupný na jej stránke s názvom v malých písmenách URL s pridaným `.md` (napríklad `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Referenčná príručka CLI](reference/cli-reference.md) a [SDK](reference/sdk-reference.md) sú napísané pre použitie s veľkými jazykovými modelmi (LLM): presné príznaky, predvolené nastavenia, sémantika ukončenia a príkazy, ktoré je možné skopírovať a vložiť.

Pozrite si [AI asistenti](ai-assistants.md), kde sa dozviete, ako nasmerovať svojho asistenta na Chloros.

</details>

<details>

<summary>Kam sa ukladajú moje spracované výstupné súbory?</summary>

Výstupné súbory sa ukladajú do projektovej zložky, zoskupené podľa kamery a následne podľa formátu súboru:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **zložka-kamery** — `LATT-<sensor>-<lens>-F<filter>` pre LATTICE, `<model>_<filter>` (napr. `Survey3N_RGN`) pre Survey3
* **zložka formátu** — `tiff16`, `tiff8`, `png8`, `jpg8` alebo `tiff32`
* **zložky produktov** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (vždy pod `tiff32`), `<INDEX>_Index_Images/`**Exportované súbory si zachovávajú názov zdrojového súboru — produkt identifikuje zložka, nie prípona súboru.**V prípade CLI sa priečinok projektu vytvorí vedľa vstupného priečinka, pokiaľ nezadáte `-o`. Všimnite si, že spustenie `chloros-cli process`, ktoré požiadalo o produkty, ale žiadne nezapísalo, vypíše `Processing finished but wrote no image products.` a**ukončí sa s nenulovým kódom**, takže skripty to môžu zistiť. Pozrite si [Formáty výstupných obrázkov](output-image-formats.md) a [Referenčnú príručku k programu CLI](reference/cli-reference.md).

</details>

<details>

<summary>Môžem svoje obrázky upraviť pred spracovaním v Chloros?</summary>

Nie. Chloros predpokladá, že vstupné údaje neboli upravené. Mená súborov nemenia.

</details>

<details>

<summary>Môžem nastaviť svoje kamery MAPIR a Survey3 na automatickú expozíciu a spracovať snímky v programe Chloros?</summary>

Nie. Sady obrazových dát musia mať pevnú/uzamknutú expozíciu, takže nie je možné použiť automatickú rýchlosť uzávierky ani automatické nastavenie ISO. Všetky snímky z rovnakého modelu kamery musia mať identickú rýchlosť uzávierky a hodnotu ISO (expozíciu).

Kamery LATTICE toto obmedzenie nemajú: Chloros riadi ich expozíciu v reálnom čase (Smart AE) a pri každom snímaní zaznamenáva skutočne použitú expozíciu a zosilnenie, čo zohľadňuje radiometrický spracovateľský reťazec.

</details>

<details>

<summary>Môže Chloros spracovávať alebo analyzovať ortomozaické snímky?</summary>

Nie. Podporované sú iba jednotlivé snímky z kamery MAPIR, nie zložené snímky, ako je ortomozaická mapa.

</details>

<details>

<summary>Ako môžem urýchliť krok detekcie cieľov v programe Chloros?</summary>

V tabuľke prehliadača súborov môžete v pravom stĺpci vopred vybrať cieľové snímky, čím nariadite programu Chloros, aby hľadal kalibračné ciele iba v týchto snímkach, čo výrazne urýchli spracovanie.

</details>

<details>

<summary>Ak chcem nahrať svoje snímky do <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">cloudu MAPIR,</a> mal by som ich pred nahratím spracovať v programe Chloros?</summary>

Ak plánujete nahrať obrázky na našu online spracovaciu platformu [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), neupravujte obrázky pred nahratím. Cloud vykoná všetky tie isté úkony a ešte viac.

</details>

<details>

<summary>Bude MAPIR niekedy podporovať funkciu X? Naozaj by som si prial, aby MAPIR ponúkal funkciu X.</summary>

Vždy nás zaujímajú spätné väzby na naše produkty. Ak zistíte nejaký problém s našimi produktmi alebo máte návrh, ako ich môžeme vylepšiť, prosím, [KONTAKTUJTE NÁS](https://www.mapir.camera/community/contact) a podelte sa s nami o svoje názory. Väčšina nášho výskumu a vývoja sa riadi tým, že načúvame najväčším potrebám našich zákazníkov.

</details>

<details>

<summary>Je Chloros k dispozícii pre Linux?</summary>

Áno! Chloros 1.2.0 podporuje Linux amd64 (x86_64) a arm64 (NVIDIA Jetson JetPack 6) prostredníctvom balíkov `.deb`. CLI a Python SDK sú plne podporované na Linux, vrátane živého ovládania kamery LATTICE a senzorov DAQ. Pre Linux nie je k dispozícii grafické používateľské rozhranie — všetka interakcia prebieha prostredníctvom [CLI](CLI.md) alebo [Python SDK](api-python-sdk.md). Podrobnosti nájdete v [Linux Prehľad](linux/linux-overview.md).

</details>

<details>

<summary>Môžem spustiť Chloros na platforme NVIDIA Jetson?</summary>

Áno! Chloros podporuje platformy NVIDIA Jetson, vrátane modelov Jetson Nano, Orin Nano, Orin NX a AGX Orin so systémom JetPack 6. Chloros automaticky rozpozná váš model Jetson a optimalizuje stratégiu spracovania. Pokyny na nastavenie a nasadenie nájdete v [Príručke k NVIDIA Jetson](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Optimalizuje sa Chloros automaticky pre môj hardvér?</summary>

Áno! Chloros obsahuje funkciu [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md), ktorá automaticky rozpozná váš procesor (CPU), grafickú kartu (GPU), pamäť RAM a (na zariadeniach Jetson) teplotné senzory. Následne vyberie optimálnu stratégiu spracovania – od `GPU_PARALLEL` na systémoch s veľkou pamäťou cez `GPU_SINGLE` na zariadeniach s obmedzenými zdrojmi až po `CPU_PARALLEL` na systémoch bez grafického procesora NVIDIA. Nie je potrebná žiadna manuálna konfigurácia.

</details>

<details>

<summary>Čo je to 4-vláknový spracovateľský kanál?</summary>

Chloros využíva 4-vláknovú potrubovú architektúru pre používateľov Chloros+: Vlákno 1 (Detekcia) načíta obrázky a detekuje kalibračné ciele, vlákno 2 (Kalibrácia) vypočíta kalibráciu odrazivosti, vlákno 3 (Spracovanie) vykonáva GPU-akcelerované odstraňovanie bayeringu a výpočet indexu a vlákno 4 (Export) zapisuje výstupné súbory. Viac obrázkov môže byť súčasne spracovávaných v rôznych vláknach, čím sa dosiahne maximálna priepustnosť. Podrobnosti nájdete v časti [Spracovateľský potrubný systém](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Ako spustím diagnostiku na mojej inštalácii Chloros?</summary>

Použite príkaz `selftest` na spustenie 7-krokového testu funkčnosti: verzia, dostupnosť portov, spustenie backendu, pripojiteľnosť API (`/api/test`), informácie o systéme (`/api/system-info` — GPU/CUDA/PyTorch), prítomnosť modelu odšumovača a pripravenosť CUDA + odšumovača:

```bash
chloros-cli selftest
```

Toto je obzvlášť užitočné na systémoch Linux/Jetson na overenie nastavenia GPU a CUDA.

</details>

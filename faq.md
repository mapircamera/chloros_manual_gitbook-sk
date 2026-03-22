---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Často kladené otázky

<details>

<summary>Môžem spracovávať snímky z kamier, ktoré nie sú značky MAPIR, pomocou Chloros?</summary>

Nie, Chloros podporuje iba spracovanie obrázkov z kamier MAPIR. Ďalšie informácie nájdete v zozname [podporovaných modelov fotoaparátov](supported-cameras.md). Spracovanie snímok z iných fotoaparátov ponúkame v službe MAPIR Cloud, kompletný zoznam nájdete [tu](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Môžem kalibrovať svoje snímky na odrazivosť bez kalibračného terča?</summary>

Nie. Bez snímky kalibračného terčíka, ktorá bola nasnímaná v čase snímania snímok bez terčíka, nebudete môcť priradiť hodnoty pixelov snímky k známemu percentu odrazivosti. Ak nezahrniete ani protokol zo senzora osvetlenia MAPIR, spektrum okolitého osvetlenia nebude merané a výsledky odrazivosti nebudú presné.

</details>

<details>

<summary>Môžem svoje snímky upraviť pred spracovaním v Chloros?</summary>

Nie. Chloros predpokladá, že vstupné údaje neboli upravené. Nezmeňujte názvy súborov.

</details>

<details>

<summary>Môžem nastaviť svoje kamery MAPIR a Survey3 na automatickú expozíciu a spracovať snímky v programe Chloros?</summary>

Nie. Datasety snímok Survey3 musia mať pevnú/uzamknutú expozíciu, takže nie je možné použiť automatickú rýchlosť uzávierky ani automatické ISO. Všetky snímky z rovnakého modelu kamery musia mať identickú rýchlosť uzávierky a ISO (expozíciu).

</details>

<details>

<summary>Môže Chloros spracovávať alebo analyzovať ortomozaické snímky?</summary>

Nie. Podporované sú len jednotlivé snímky z fotoaparátu MAPIR, nie zložené snímky, ako je ortomozaická mapa.

</details>

<details>

<summary>Ako môžem urýchliť krok detekcie cieľov v programe Chloros?</summary>

V tabuľke prehliadača súborov predvolením cieľových snímok v pravom stĺpci nariadite programu Chloros, aby hľadal kalibračné ciele iba v týchto snímkach, čím sa výrazne urýchli spracovanie.

</details>

<details>

<summary>Ak chcem nahrať svoje snímky do <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> mal by som ich pred nahratím spracovať v Chloros?</summary>

Ak plánujete nahrať obrázky na našu online spracovateľskú platformu [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), neupravujte ich pred nahratím. Cloud vykoná všetky rovnaké spracovania a ešte viac.

</details>

<details>

<summary>Bude MAPIR niekedy podporovať funkciu X? Naozaj by som si prial, aby MAPIR ponúkal funkciu X.</summary>

Vždy nás zaujímajú spätné väzby na naše produkty. Ak zistíte nejaký problém s našimi produktmi alebo máte návrh, ako ich môžeme vylepšiť, prosím, [KONTAKTUJTE NÁS](https://www.mapir.camera/community/contact) a podelte sa s nami o svoje myšlienky. Väčšina nášho výskumu a vývoja sa riadi počúvaním najväčších potrieb našich zákazníkov.

</details>

<details>

<summary>Je Chloros k dispozícii pre Linux?</summary>

Áno! Chloros 1.1.0 podporuje Linux amd64 (x86\_64) a arm64 (NVIDIA Jetson JetPack 6) prostredníctvom balíkov `.deb`. CLI a Python SDK sú plne podporované na Linux. Pre Linux neexistuje grafické používateľské rozhranie (GUI) — všetka interakcia prebieha prostredníctvom [CLI](CLI.md) alebo [Python SDK](api-python-sdk.md). Podrobnosti nájdete v [Linux Prehľad](linux/linux-overview.md).

</details>

<details>

<summary>Môžem spustiť Chloros na NVIDIA Jetson?</summary>

Áno! Chloros 1.1.0 podporuje platformy NVIDIA Jetson vrátane Jetson Nano, Orin Nano, Orin NX a AGX Orin s operačným systémom JetPack 6. Chloros automaticky rozpozná váš model Jetson a optimalizuje stratégiu spracovania. Pokyny na nastavenie a nasadenie nájdete v [príručke NVIDIA Jetson](linux/nvidia-jetson-guide.md).

</details>

<details>

<summary>Optimalizuje sa Chloros automaticky pre môj hardvér?</summary>

Áno! Chloros 1.1.0 obsahuje funkciu [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md), ktorá automaticky detekuje váš procesor (CPU), grafickú kartu (GPU), pamäť RAM a (na zariadeniach Jetson) teplotné senzory. Následne vyberie optimálnu stratégiu spracovania – od `GPU_PARALLEL` na systémoch s veľkou pamäťou cez `GPU_SINGLE` na zariadeniach s obmedzenými zdrojmi až po `CPU_PARALLEL` na systémoch bez grafického procesora NVIDIA. Nie je potrebná žiadna manuálna konfigurácia.

</details>

<details>

<summary>Čo je 4-vláknový spracovateľský pipeline?</summary>

Chloros 1.1.0 používa 4-vláknovú potrubnú architektúru pre používateľov Chloros+: Vlákno 1 (Detekcia) načíta obrázky a detekuje kalibračné ciele, vlákno 2 (Kalibrácia) vypočíta kalibráciu odrazivosti, vlákno 3 (Spracovanie) vykonáva GPU-akcelerované odstraňovanie bayeringu a výpočet indexu a vlákno 4 (Export) zapisuje výstupné súbory. Viac obrázkov môže byť súčasne v rôznych vláknach pre maximálnu priepustnosť. Podrobnosti nájdete v [Spracovateľskom potrubí](processing-architecture/processing-pipeline.md).

</details>

<details>

<summary>Ako spustím diagnostiku na mojej inštalácii Chloros?</summary>

Pomocou príkazu `selftest` spustite 7 systémových diagnostík, vrátane kontroly verzie, dostupnosti portov, spustenia backendu, pripojiteľnosti API, informácií o systéme, modelov odšumovača a dostupnosti CUDA:

```bash
chloros-cli selftest
```

Toto je obzvlášť užitočné na systémoch Linux/Jetson na overenie nastavenia GPU a CUDA.

</details>

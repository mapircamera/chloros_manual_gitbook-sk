# Prehľad Linux

Chloros 1.2.0 poskytuje natívnu podporu Linux pre **CLI**a**Python SDK** — spracovanie multispektrálnych snímok bez grafického rozhrania, ako aj ovládanie kamery LATTICE a svetelného senzora DAQ v reálnom čase — na pracovných staniciach, serveroch a zariadeniach NVIDIA Jetson.

{% hint style="info" %}
**Na Linux nie je k dispozícii grafické používateľské rozhranie (GUI).**Grafické používateľské rozhranie Chloros je dostupné iba na Windows. Používatelia Linux komunikujú s Chloros prostredníctvom [CLI](../CLI.md) a [Python SDK](../api-python-sdk.md). `.deb` skutočne pridáva položku**Chloros CLI** do ponuky vašej aplikácie – jednoducho otvorí emulátor terminálu, v ktorom beží `chloros-cli`.
{% endhint %}

***

## Matica podpory platforiem

| Funkcia | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Grafické rozhranie pre stolné počítače** | Áno | N/A | Nie | Nie |
| **CLI** (`chloros-cli`) | Áno | Áno | Áno | Áno |
| **Python SDK** (`chloros-sdk`) | Áno | Áno | Áno | Áno |
| **Spracovávanie obrazu** | Áno | Áno | Áno | Áno |
| **Ovládanie kamery LATTICE (naživo)** | Áno (karta Kamery) | Áno (`chloros-cli lattice`, SDK) | Áno | Áno |
| **Svetelné senzory DAQ (v reálnom čase)** | Áno (karta Svetelné senzory) | Áno (`chloros-cli daq pool-*`, SDK) | Áno | Áno |
| **Časová synchronizácia PTP (hostiteľ je grandmaster)** | Áno | Áno (`chloros-cli time-sync`) | Áno | Áno |
| **Akcelerácia GPU (CUDA)** | Áno | Áno | Áno | Áno (JetPack 6) |
| **Debayer s ohľadom na textúry** | Áno (Chloros+) | Áno (Chloros+) | Áno (Chloros+) | Áno (Chloros+) |
| **Dynamická adaptácia výpočtov** | Áno | Áno | Áno | Áno |
| **Backend ako systémová služba** (`chloros-backend.service`) | Nie | Nie | Áno (na základe voľby) | Áno (na základe voľby) |
| **Aktualizátor na mieste** (`chloros-cli update`) | Nie (spustite inštalátor) | Nie (spustite inštalátor) | Áno | Áno |***

## Podporované architektúry

| Architektúra | Popis | Balík |
| --- | --- | --- |
| **amd64 (x86_64)** | Štandardné procesory pre stolné počítače a servery (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Procesory ARM — rodina NVIDIA Jetson Orin | `chloros_<version>_arm64_jp6.deb` (verzia JetPack 6) |

## Podporované distribúcie Linux

* **Ubuntu 22.04 LTS alebo novší** (amd64)
* **Debian 12 alebo novší** (amd64)
* **NVIDIA JetPack 6** (arm64 — platformy Jetson Orin)***

## Čo získajú používatelia Linux

* **Chloros CLI** — kompletné rozhranie príkazového riadku pre dávkové spracovanie, automatizáciu a skriptovanie
* **Chloros Python SDK** — programovateľné rozhranie Python pre výskumné pipeline a vlastné nástroje (inštalovateľné z PyPI a tiež súčasťou balíka `.deb` ako wheel s kompatibilnou verziou)
* **Ovládanie kamier LATTICE** — vyhľadávanie, pripájanie, konfigurácia a snímanie z kamier LATTICE a synchronizovaných sústav viacerých kamier prostredníctvom `chloros-cli lattice` a SDK; `.deb` obsahuje runtime prostredie Arena SDK, ktoré kamery vyžadujú
* **Ovládanie svetelných senzorov DAQ** — pripojte senzory DAQ-U/M/E, streamujte kalibrované spektrá a zaznamenávajte súbory `.daq` prostredníctvom `chloros-cli daq pool-*` a SDK
* **Časová synchronizácia PTP** — backend Chloros prevádzkuje PTP grandmaster, ku ktorému sú kamery LATTICE a senzory DAQ-E pripojené ako slave; skontrolujte ho pomocou `chloros-cli time-sync`, a udržujte ho v chode bez monitorovania pomocou jednotky systemd `chloros-backend.service` (pozri [Inštalácia Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automatizácia projektov** — spúšťajte uložené projekty bez obsluhy pomocou `chloros-cli project` a `open_project` v rámci SDK
* **Akcelerácia GPU** — spracovanie akcelerované technológiou CUDA na grafických procesoroch NVIDIA (stolné počítače a Jetson)
* **Dynamická adaptácia výpočtov** — automatická detekcia hardvéru a výber stratégie spracovania, s možnosťou ručného zásahu pomocou `CHLOROS_STRATEGY` ako expertného núdzového riešenia
* **Všetky funkcie spracovania** — rovnaký proces ako v prípade Windows: kalibrácia, korekcia vinety, vegetačné indexy a všetky formáty exportu
* **Funkcie Chloros+** — viacvláknové (konvejorové) spracovanie, debayer s ohľadom na textúru a vlastné indexy, s plateným plánom Chloros+

## Čo používatelia Linux nedostanú

* **GUI pre stolné počítače** — žiadne grafické rozhranie; všetka interakcia prebieha cez CLI alebo Python SDK
* **Prehliadač obrázkov** — žiadny interaktívny prehliadač obrázkov, zobrazenie mriežky ani značky na mape
* **Vizuálne riadenie projektov** — projekty sa vytvárajú a riadia prostredníctvom príkazov CLI a volaní SDK (samotný hardvér — kamery, senzory, snímanie — zostáva plne ovládateľný z terminálu)***

## Požiadavky na licenciu

Prístup k CLI a SDK vyžaduje **platenú úroveň Chloros+ — Copper alebo vyššiu**(Copper, Bronze, Silver, Gold). Bezplatná úroveň**Iron** neumožňuje prístup k CLI/SDK. Toto obmedzenie je vynucované na strane backendu, nielen prostredníctvom CLI:

| Situácia | Odpoveď backendu |
| --- | --- |
| Nie ste prihlásený | `401` s `error_code: AUTH_REQUIRED` |
| Prihlásený na bezplatnej úrovni Iron | `403` s `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` funguje v akejkoľvek úrovni — je to jediná trasa, ktorá nie je obmedzená bránou — takže dôvod odmietnutia je vždy viditeľný.

***

## Začíname s Linux

1. **Nainštalujte Chloros** — pozrite si [Inštalácia Linux](linux-installation.md) pre inštaláciu `.deb`
2. **Overte** — `chloros-cli --version` vytlačí `Chloros CLI 1.2.0`; `chloros-cli selftest` spustí 7-krokovú diagnostiku
3. **Nainštalujte Python a SDK** (voliteľné) — `pip install chloros-sdk`
4. **Prihláste sa** — `chloros-cli login your@email.com 'your-password'` (raz na každý počítač a znovu po každej aktualizácii balíka)
5. **Spracujte svoj prvý súbor údajov** — `chloros-cli process ~/datasets/flight001`

V prípade zariadenia NVIDIA Jetson si prečítajte špeciálnu [Príručku pre NVIDIA Jetson](nvidia-jetson-guide.md), kde nájdete informácie o nastavení špecifickom pre danú platformu, tepelnom správaní a nasadení v teréne.

***

## Ďalšie kroky

* [Inštalácia Linux](linux-installation.md) — podrobný postup inštalácie, umiestnenie súborov a riešenie problémov pre architektúry amd64 a arm64
* [Príručka NVIDIA Jetson](nvidia-jetson-guide.md) — nastavenie špecifické pre Jetson, správanie pamäte a tepelné správanie, nasadenie v teréne
* [CLI : Príkazový riadok](../CLI.md) — sprievodca CLI
* [API : Python SDK](../api-python-sdk.md) — príručka k SDK
* [CLI Referenčná príručka](../reference/cli-reference.md) a [SDK Referenčná príručka](../reference/sdk-reference.md) — vyčerpávajúci zoznam príkazov/API pre verziu 1.2.0
* [Dynamická adaptácia výpočtov](../processing-architecture/dynamic-compute-adaptation.md) — ako sa Chloros prispôsobuje vášmu hardvéru

{% hint style="info" %}
**Programové čítanie tejto príručky.** Každá stránka je tiež dostupná ako surový Markdown na vlastnej adrese URL plus `.md` (napríklad `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), a index celého manuálu je uverejnený na [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}

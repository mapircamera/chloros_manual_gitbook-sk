# Prehľad systému Linux

Chloros 1.3.0 poskytuje natívnu podporu pre systém Linux pre **CLI**a**Python SDK** – spracovanie multispektrálnych snímok bez grafického rozhrania, ako aj ovládanie kamery LATTICE a svetelných senzorov DAQ v reálnom čase – na pracovných staniciach, serveroch so systémom Linux a zariadeniach NVIDIA Jetson na okraji siete.

{% hint style="info" %}
**V systéme Linux nie je k dispozícii grafické používateľské rozhranie (GUI).**Grafické používateľské rozhranie Chloros Desktop je dostupné iba pre systém Windows. Používatelia systému Linux komunikujú s Chloros prostredníctvom [CLI](../CLI.md) a [Python SDK](../api-python-sdk.md). `.deb` pridáva do ponuky aplikácií položku**Chloros CLI** — jednoducho otvorí emulátor terminálu, v ktorom beží `chloros-cli`.
{% endhint %}

***

## Tabuľka podpory platforiem

| Funkcia | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **GUI pre stolné počítače** | Áno | N/A | Nie | Nie |
| **CLI** (`chloros-cli`) | Áno | Áno | Áno | Áno |
| **Python SDK** (`chloros-sdk`) | Áno | Áno | Áno | Áno |
| **Spracovávacia línia obrazu** | Áno | Áno | Áno | Áno |
| **Ovládanie kamery LATTICE (naživo)** | Áno (karta Kamery) | Áno (`chloros-cli lattice`, SDK) | Áno | Áno |
| **Svetelné senzory DAQ (v reálnom čase)** | Áno (karta Svetelné senzory) | Áno (`chloros-cli daq pool-*`, SDK) | Áno | Áno |
| **Časová synchronizácia PTP (hostiteľ je grandmaster)** | Áno | Áno (`chloros-cli time-sync`) | Áno | Áno |
| **Akcelerácia GPU (CUDA)** | Áno | Áno | Áno | Áno (JetPack 6) |
| **Debayer s podporou textúr** (vyžaduje GPU s CUDA) | Áno (Chloros+) | Áno (Chloros+) | Áno (Chloros+) | Áno (Chloros+) |
| **Dynamická adaptácia výpočtov** | Áno | Áno | Áno | Áno |
| **Backend ako systémová služba** (`chloros-backend@<user>.service`) | Nie | Nie | Áno (na základe voľby) | Áno (na základe voľby) |
| **Aktualizátor na mieste** (`chloros-cli update`) | Nie (spustite inštalátor) | Nie (spustite inštalátor) | Áno | Áno |

***

## Podporované architektúry

| Architektúra | Popis | Balík |
| --- | --- | --- |
| **amd64 (x86_64)** | Štandardné procesory pre stolné počítače/servery (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Procesory ARM — rodina NVIDIA Jetson Orin (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Podporované distribúcie Linuxu

* **Ubuntu 22.04 LTS alebo novší** (amd64)
* **Debian 12 alebo novší** (amd64)
* **NVIDIA JetPack 6** (arm64 — platformy Jetson Orin)

***

## Čo získajú používatelia Linuxu

* **Chloros CLI** — kompletné rozhranie príkazového riadku pre dávkové spracovanie, automatizáciu a skriptovanie
* **Chloros Python SDK** — programové rozhranie v jazyku Python pre výskumné pracovné postupy a vlastné nástroje, inštalované balíkom `.deb` pre systém `python3` (koleso s zodpovedajúcou verziou pre virtualenvs sa nachádza v `/usr/lib/chloros/sdk/`; PyPI je určené pre stroje bez balíka Chloros)
* **Ovládanie kamier LATTICE** — vyhľadávanie, pripájanie, konfigurácia a snímanie z kamier LATTICE a synchronizovaných sústav viacerých kamier prostredníctvom balíka `chloros-cli lattice` a SDK; balík `.deb` obsahuje runtime prostredie Arena SDK, ktoré kamery vyžadujú
* **Ovládanie svetelných senzorov DAQ** — pripojte senzory DAQ-U/M/E, streamujte kalibrované spektrá a zaznamenávajte súbory `.daq` prostredníctvom `chloros-cli daq pool-*` a SDK
* **Časová synchronizácia PTP** — backend Chloros spúšťa hlavný PTP server, ku ktorému sú kamery LATTICE a senzory DAQ-E pripojené ako podriadené zariadenia; skontrolujte ho pomocou `chloros-cli time-sync` a udržujte ho v chode bez obsluhy pomocou jednotky systemd `chloros-backend@<user>.service` (pozri [Inštalácia v Linuxe](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automatizácia projektov** — spúšťajte uložené projekty bez obsluhy pomocou `chloros-cli project` a modulu `open_project` v SDK
* **Akcelerácia GPU** — spracovanie akcelerované technológiou CUDA na grafických procesoroch NVIDIA (stolové grafické procesory na architektúre amd64 a grafický procesor Jetson s JetPack 6)
* **Dynamická adaptácia výpočtov** — automatická detekcia hardvéru a výber stratégie spracovania, s možnosťou prepisovania pomocou `CHLOROS_STRATEGY` ako expertným únikovým riešením
* **Všetky funkcie spracovania** — rovnaký postup ako v systéme Windows: kalibrácia, korekcia vinety, vegetačné indexy a všetky formáty exportu
* **Funkcie Chloros+** — viacvláknové (pipelinové) spracovanie, debayer s podporou textúr (na grafickom procesore CUDA) a vlastné indexy v rámci plateného plánu Chloros+

## Čo používatelia Linuxu nemajú k dispozícii

* **Grafické rozhranie na pracovnej ploche** — žiadne grafické rozhranie; všetka interakcia prebieha cez CLI alebo Python SDK
* **Prehliadač obrázkov** — žiadny interaktívny prehliadač obrázkov, zobrazenie v mriežke ani značky na mape
* **Vizuálne riadenie projektov** — projekty sa vytvárajú a riadia prostredníctvom príkazov CLI a volaní SDK (samotný hardvér — kamery, senzory, snímanie — zostáva plne ovládateľný z terminálu)

***

## Licenčné požiadavky

Prístup k CLI a SDK vyžaduje **platenú úroveň Chloros+ — Copper alebo vyššiu**(Copper, Bronze, Silver, Gold). Bezplatná úroveň**Iron** nemá prístup k CLI/SDK. Toto obmedzenie je vynucované backendom, nielen CLI:

| Situácia | Odpoveď backendu |
| --- | --- |
| Nie ste prihlásený | `401` s `error_code: AUTH_REQUIRED` |
| Prihlásený v bezplatnej úrovni Iron | `403` s `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` funguje na akejkoľvek úrovni — je to jediná trasa, ktorá nie je obmedzená bránou — takže dôvod odmietnutia je vždy viditeľný.

***

## Začíname na Linuxu

1. **Nainštalujte Chloros** — pozrite si [Inštalácia na Linux](linux-installation.md) pre inštaláciu `.deb`
2. **Overte** — `chloros-cli --version` vypíše `Chloros CLI 1.3.0`; `chloros-cli selftest` spustí 7-krokovú diagnostiku
3. **Skontrolujte Python SDK** — inštalácia `.deb` ho nainštaluje: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (vo virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Prihláste sa** — `chloros-cli login your@email.com 'your-password'` (raz na každom počítači; aktualizácie balíka zachovávajú prihlásenie, odstránenie balíka ho vymaže)
5. **Spracujte svoj prvý súbor údajov** — `chloros-cli process ~/datasets/flight001`

Pre NVIDIA Jetson si prečítajte špeciálnu [Príručku pre NVIDIA Jetson](nvidia-jetson-guide.md), kde nájdete informácie o nastavení špecifickom pre danú platformu, tepelnom správaní a nasadení v teréne.

***

## Ďalšie kroky

* [Inštalácia v systéme Linux](linux-installation.md) — podrobný postup inštalácie, umiestnenie súborov a riešenie problémov pre architektúry amd64 a arm64
* [Príručka pre NVIDIA Jetson](nvidia-jetson-guide.md) — nastavenie špecifické pre Jetson, správanie pamäte a tepelné správanie, nasadenie v teréne
* [CLI: Príkazový riadok](../CLI.md) — sprievodca CLI
* [API: Python SDK](../api-python-sdk.md) — sprievodca SDK
* [Referenčná príručka CLI](../reference/cli-reference.md) a [Referenčná príručka SDK](../reference/sdk-reference.md) — vyčerpávajúci zoznam príkazov a API pre verziu 1.3.0
* [Dynamická adaptácia výpočtového výkonu](../processing-architecture/dynamic-compute-adaptation.md) — ako sa Chloros prispôsobuje vášmu hardvéru

{% hint style="info" %}
**Programové čítanie tejto príručky.** Každá stránka je zároveň dostupná ako surový Markdown na vlastnej URL plus `.md` (napríklad `https://mapir.gitbook.io/chloros/linux/linux-installation.md`) a index celej príručky je uverejnený na [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}

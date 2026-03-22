# Linux – Prehľad

Chloros 1.1.0 prináša natívnu podporu pre **CLI**a**Python SDK**, čo umožňuje bezhlavé spracovanie multispektrálnych obrazov na pracovných staniciach, serveroch a okrajových zariadeniach NVIDIA Jetson.

{% hint style="info" %}
**Na Linux nie je k dispozícii grafické používateľské rozhranie.** Grafické používateľské rozhranie Chloros Desktop je k dispozícii iba na Windows. Linux používatelia komunikujú s Chloros prostredníctvom [CLI](../CLI.md) a [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Matica podpory platforiem

| Funkcia | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **GUI pre stolné počítače** | Áno | N/A | Nie | Nie |
| **CLI** | Áno | Áno | Áno | Áno |
| **Python SDK** | Áno | Áno | Áno | Áno |
| **Akcelerácia GPU (CUDA)** | Áno | Áno | Áno | Áno (JetPack 6) |
| **Debayer s podporou textúr** | Áno (Chloros+) | Áno (Chloros+) | Áno (Chloros+) | Áno (Chloros+) |
| **Dynamická adaptácia výpočtov** | Áno | Áno | Áno | Áno |***

## Podporované architektúry

| Architektúra | Popis | Spôsob inštalácie |
| --- | --- | --- |
| **amd64 (x86_64)** | Štandardné procesory pre stolné počítače/servery (Intel, AMD) | Balík `.deb` |
| **arm64 (aarch64)** | Procesory založené na architektúre ARM, predovšetkým NVIDIA Jetson | Balík `.deb` (JetPack 6) |

## Podporované distribúcie Linux

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — platformy Jetson)***

## Čo získajú používatelia Linux

* **Chloros CLI** — Kompletné rozhranie príkazového riadku pre dávkové spracovanie, automatizáciu a skriptovanie
* **Chloros Python SDK** — Programové rozhranie Python (`pip install chloros-sdk`) na integráciu do výskumných procesov a vlastných nástrojov
* **Akcelerácia GPU** — Spracovanie akcelerované technológiou CUDA na grafických procesoroch NVIDIA (stolné počítače a Jetson)
* **Dynamická adaptácia výpočtov** — Automatická detekcia hardvéru a optimalizácia stratégie spracovania
* **Všetky funkcie spracovania** — Rovnaký multispektrálny spracovateľský reťazec ako Windows (kalibrácia, korekcia vinety, vegetačné indexy, všetky formáty exportu)
* **Funkcie Chloros+** — Viacvláknové spracovanie, debayer s podporou textúr, vlastné indexy (s licenciou Chloros+)

## Čo používatelia Linux nedostanú

* **GUI pre stolné počítače** — Žiadne grafické rozhranie; všetka interakcia prebieha cez CLI alebo Python SDK
* **Prehliadač obrázkov** — Žiadny interaktívny prehliadač obrázkov, zobrazenie v mriežke ani značky na mape
* **Vizuálne riadenie projektov** — Projekty sa spravujú prostredníctvom príkazov CLI a volaní SDK***

## Začíname s Linux

1. **Inštalácia Chloros** — Pozrite si [Inštalácia Linux](linux-installation.md) pre inštaláciu balíka `.deb`
2. **Nainštalujte Python SDK** (voliteľné) — `pip install chloros-sdk`
3. **Aktivujte svoju licenciu** — `chloros-cli login your@email.com 'password'`
4. **Spracujte svoj prvý dataset** — `chloros-cli process ~/datasets/flight001`

Používatelia NVIDIA Jetson nájdu v špecializovanej [príručke NVIDIA Jetson](nvidia-jetson-guide.md) informácie o nastavení a optimalizácii špecifické pre túto platformu.

***

## Ďalšie kroky

* [Inštalácia Linux](linux-installation.md) — Podrobné inštrukcie na inštaláciu pre amd64 a arm64
* [Príručka NVIDIA Jetson](nvidia-jetson-guide.md) — Nastavenie špecifické pre Jetson, riadenie teploty a nasadenie v teréne
* [CLI : Príkazový riadok](../CLI.md) — Kompletná referenčná príručka k CLI
* [API : Python SDK](../api-python-sdk.md) — Kompletná referenčná príručka k SDK
* [Dynamická adaptácia výpočtov](../processing-architecture/dynamic-compute-adaptation.md) — Ako sa Chloros prispôsobuje vášmu hardvéru

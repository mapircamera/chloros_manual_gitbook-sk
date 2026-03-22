# Dynamická adaptácia výpočtov

Chloros 1.1.0 prináša inteligentnú detekciu hardvéru a automatický výber stratégie spracovania. Spracovateľský modul sa prispôsobí vášmu hardvéru – od Jetson Nano až po pracovnú stanicu s viacerými grafickými kartami – bez akejkoľvek manuálnej konfigurácie.

***

## Ako to funguje

Keď sa spustí Chloros, automaticky vytvorí profil vášho systému:

1. **Detekuje operačný systém** — Windows alebo Linux
2. **Identifikuje jadrá CPU a celkovú RAM**

3.**Zistí prítomnosť GPU** — schopnosť NVIDIA CUDA, VRAM, model
4. **Identifikuje model Jetson** (ak je to relevantné) — prostredníctvom `/proc/device-tree/model`
5. **Skontroluje teplotné senzory** (Jetson) — pre spracovanie zohľadňujúce teplotu
6. **Vyberá optimálnu výpočtovú stratégiu** — na základe všetkého zisteného hardvéru
7. **Automaticky konfiguruje počet pracovníkov, typ potrubia a alokáciu pamäte**Výsledok sa ukladá do vyrovnávacej pamäte, aby sa nasledujúce spustenia začali rýchlejšie. Ak dôjde k zmene hardvéru (napr. pridanie GPU), Chloros pri nasledujúcom spustení vykoná nové profilovanie.***

## Výpočtové stratégie

Chloros vyberá jednu z troch výpočtových stratégií na základe vášho hardvéru:

| Stratégia | Požadované GPU | Pracovníci | Potrubie | Najvhodnejšie pre |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Áno (12 GB+ VRAM alebo 16 GB+ zdieľanej) | 3–4 | `fused_gpu` | Desktopové GPU s 12 GB+, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Áno (&lt; 12 GB VRAM) | 1–3 | `tiled_gpu` | Základné grafické karty, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Nie | jadrá – 1 | `cpu_fallback` | Systémy bez grafického procesora NVIDIA |

### Typy potrubia

* **`fused_gpu`** — Plná spracovateľská cesta grafického procesora. Všetky operácie debayeringu, korekcie a indexovania prebiehajú na GPU v jedinom zlúčenom cykle. Najvyššia priepustnosť, ale vyžaduje viac VRAM.
* **`tiled_gpu`** — Cesta GPU s efektívnym využívaním pamäte. Spracováva obrázky v dlaždiciach, aby sa zmestili do obmedzenej pamäte GPU. Nižšia priepustnosť, ale funguje na zariadeniach s obmedzenou pamäťou.
* **`cpu_fallback`** — Spracovanie výlučne na CPU s využitím viacvláknovej paralelnosti. Používa sa, ak nie je k dispozícii žiadne GPU NVIDIA.***

## Správanie špecifické pre platformu

| Platforma | Stratégia | Pracovníci | Potrubie | Poznámky |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sériové) | Režim s efektívnym využívaním pamäte, spracováva jeden obrázok naraz |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (súbežné) | Odporúčané okrajové zariadenie — skutočné paralelné spracovanie GPU |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (súbežné) | Maximálny výkon na okraji siete |
| **Stolný počítač s 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Dobrý výkon stolného počítača s pamäťovo efektívnymi dlaždicami |
| **Stolný počítač s grafickou kartou 12 GB+** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Optimálny výkon na stolnom počítači |
| **Systém len s CPU** | `CPU_PARALLEL` | jadrá – 1 | `cpu_fallback` | Nie je potrebná GPU, používa ThreadPool |

{% hint style="info" %}
**Unifikovaná pamäť Jetson**: Zariadenia Jetson zdieľajú pamäť GPU a CPU. Jetson Orin NX 16 GB hlási ~15,3 GB VRAM, ale ide o tú istú fyzickú RAM, ktorú používajú procesy OS a CPU. Chloros to zohľadňuje pri nastavovaní prahov alokácie pamäte.
{% endhint %}

***

## Dynamické prideľovanie pamäte GPU

Chloros používa [4-vláknový spracovateľský pipeline](processing-pipeline.md):

* **Vlákno 1** (Detekcia) — Načítanie obrázku, analýza EXIF, detekcia cieľa
* **Vlákno 2** (Kalibrácia) — Výpočet kalibrácie odrazivosti
* **Vlákno 3** (Spracovanie) — Debayer GPU, korekcia vinetácie, výpočet indexu
* **Vlákno 4** (Export) — Zapisovanie súborov, vkladanie metadát

Ako predchádzajúce vlákna potrubia dokončia svoju prácu (napr. všetky obrázky boli detegované), ich alokácia pamäte GPU sa uvoľní a **prerozdelí sa na zostávajúce aktívne vlákna**. To znamená, že vlákno 3 (fáza náročná na GPU) získava postupne viac pamäte v miere, ako postupuje potrubie, čím sa zlepšuje priepustnosť pre prácu s najvyššou výpočtovou náročnosťou.

### Fázy alokácie

| Fáza | Aktívne vlákna | Rozdelenie pamäte GPU |
| --- | --- | --- |
| **Začiatok** | 1, 2, 3, 4 | Rozdelené medzi všetky vlákna |
| **Stredná-raná** | 2, 3, 4 | Pamäť vlákna 1 prerozdelená |
| **Stredná-neskorá** | 3, 4 | Pamäť vlákien 1+2 ide do 3+4 |
| **Neskorá** | 3 alebo 4 | Maximálna pamäť pre zostávajúce vlákno |***

## Spracovanie s podporou textúr

Metóda debayeringu s podporou textúr (len Chloros+) využíva výrazne viac pamäte GPU ako štandardná metóda kvôli modelu odšumovania AI/ML:

* Systémy s **&lt; 7 GB VRAM** sú v režime s podporou textúr nútené používať synchronnú spracovateľskú slučku (jeden obrázok po druhom)
* Systémy s **7 GB a viac VRAM** môžu spracovávať Texture Aware súbežne, hoci s nižším počtom pracovných procesov v porovnaní so štandardným režimom***

## Riadenie teploty (Jetson)

Zariadenia Jetson majú teplotné obmedzenia, najmä pri inštaláciách v uzavretých priestoroch alebo vo vzduchu. Chloros monitoruje teploty GPU a CPU a automaticky prispôsobuje spracovanie:

| Teplota | Reakcia |
| --- | --- |
| **&lt; 70 °C** | Normálna prevádzka — plná rýchlosť |
| **70 °C** (Varovanie) | Zníženie veľkosti dávky |
| **80 °C** (Kritické) | Agresívne obmedzenie — zníženie súbežnosti a počtu pracovných procesov |
| **90 °C** (Vypnutie) | Úplné zastavenie spracovania GPU |

Monitorovanie teploty využíva `tegrastats` na platformách Jetson. Na stolných systémoch s dostatočným chladením sa tepelné obmedzovanie spúšťa len zriedka.

***

## Riešenie zaťaženia pamäte

Chloros monitoruje tlak na systémovú pamäť počas spracovania:

* **Prah pamäte**: 85 % využitie spustí konzervatívne správanie
* **Zníženie OOM**: Ak dôjde k udalosti nedostatku pamäte, alokácia sa zníži o 25 % (multiplikátor 0,75x)
* **Záložný postup potrubia**: Pri veľkom zaťažení pamäte sa potrubie automaticky prepne z `fused_gpu` na `tiled_gpu`
* **Odporúčania týkajúce sa výmennej pamäte**: Na Jetsone vás Chloros upozorní, ak je výmenná pamäť nedostatočná pre veľkosť vášho súboru údajov***

## Monitorovanie prispôsobenia výpočtov

### Výstup stavu CLI

Po spustení spracovania zobrazí CLI zistený profil hardvéru:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Diagnostika systému

Spustite `chloros-cli selftest`, aby ste videli úplný profil hardvéru a overili výpočtové schopnosti:

```bash
chloros-cli selftest
```

Týmto sa skontroluje dostupnosť CUDA, pamäť GPU, modely odšumovača a konektivita backendu.

***

## Ďalšie kroky

* [Spracovateľský pipeline](processing-pipeline.md) — Porozumenie architektúre 4-vláknového pipeline
* [Príručka NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Nasadenie a optimalizácia špecifické pre Jetson
* [CLI : Príkazový riadok](../CLI.md) — Úplná referenčná príručka CLI

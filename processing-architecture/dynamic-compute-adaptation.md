# Dynamická adaptácia výpočtov

Chloros 1.3.0 využíva detekciu hardvéru a automatický výber stratégie spracovania. Spracovateľský modul sa prispôsobí vášmu hardvéru – od Jetson Orin Nano až po pracovnú stanicu s viacerými grafickými kartami – bez akejkoľvek manuálnej konfigurácie.

***

## Ako to funguje

Pri spustení Chloros u sa vykoná profilovanie vášho systému:

1. **Zistí operačný systém** — Windows alebo Linux
2. **Identifikuje jadrá procesora a celkovú veľkosť pamäte RAM**

3.**Zistí prítomnosť grafického procesora** — podporu NVIDIA CUDA, VRAM, model
4. **Identifikuje model Jetson** (ak je to relevantné) — prostredníctvom `/proc/device-tree/model`
5. **Skontroluje teplotné senzory** (Jetson) — pre spracovanie zohľadňujúce teplotu
6. **Vyberie stratégiu výpočtov** — na základe všetkého zisteného hardvéru
7. **Automaticky konfiguruje počet pracovných procesov, typ potrubia a alokáciu pamäte**

Zistený profil sa na danú reláciu ukladá do vyrovnávacej pamäte a na disk, vďaka čomu sa neskoršie spustenia začnú rýchlejšie:

| Platforma | Profil v vyrovnávacej pamäti |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (uprednostňuje `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Vymazanie tohto súboru vynúti nové detekovanie — je to užitočné po pridaní grafického procesora (GPU) alebo väčšieho množstva pamäte RAM. Chloros tiež automaticky vykoná nové detekovanie, ak bola vyrovnávacia pamäť zapísaná nekompatibilnou staršou verziou.

***

## Výpočtové stratégie

Chloros vyberá jednu z troch výpočtových stratégií na základe vášho hardvéru:

| Stratégia | Vybratá v prípade | Pracovníci | Vykonávateľ | Potrubie |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| Grafická karta CUDA s**12 GB a viac VRAM**(na Jetson s jednotnou pamäťou, vyžaduje tiež celkovo 12 GB+ zdieľanej RAM) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, minimálne 2 —**obmedzené na 2 na Jetson** | Pracovné procesy (samostatné kontexty CUDA) | `fused_gpu` |
| **`GPU_SINGLE`**| CUDA GPU s**2–12 GB VRAM**| 3 (prekrývanie I/O; prístup k GPU serializovaný semaforom).**1 (sekvenčné) na zariadeniach Jetson s menej ako 12 GB RAM** | Pracovné procesy (samostatné kontexty CUDA); sekvenčné v rámci procesu na zariadeniach Jetson s nízkou kapacitou RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Bez CUDA GPU alebo s menej ako 2 GB VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Praktické príklady vzorca pre počet pracovníkov `GPU_PARALLEL`: 12 GB VRAM → 2 pracovníci, 16 GB → 3 pracovníci, 20 GB a viac → 4 pracovníci. Každému pracovníkovi je pridelených 3,5 GB, čo zodpovedá nameranej potrebe úlohy s podporou textúr, v rámci 90 % VRAM zostávajúcej po vytvorení rezervy 0,5 GB.

{% hint style="info" %}
**Stratégie pre GPU vyžadujú PyTorch, ktorý dokáže využívať CUDA.** Zariadenie, ktorého PyTorch nemá použiteľné CUDA zariadenie, dostane hodnotu `CPU_PARALLEL`, bez ohľadu na typ jeho GPU. Balík Jetson 1.3.0 používa verziu PyTorch pre CUDA z JetPack 6, takže zariadenie Jetson spĺňa podmienky hneď po inštalácii knižníc CUDA a cuDNN z JetPacku. Zariadenie Jetson s menej ako 12 GB RAM spracováva vždy len jeden obrázok naraz, a to priamo počas spracovania (zaznamenané ako `GPU_SINGLE`).
{% endhint %}

Stratégie využívajúce GPU spúšťajú skupinu pracovných procesov, z ktorých každý je samostatným procesom operačného systému s vlastným kontextom CUDA, ktoré Chloros spúšťa opätovným spustením svojho vlastného nainštalovaného backendového programu (nie prostredníctvom `multiprocessing` v Python e, ktorého spúšťanie pracovných procesov vo vnútri balíka aplikácie nefunguje). Stratégia CPU využíva skupinu vlákien (Python ov `ThreadPoolExecutor`). Chloros nepoužíva žiadny distribuovaný framework tretej strany (napríklad Ray).

### Typy spracovacích reťazcov

* **`fused_gpu`** — Cesta spracovania výlučne na GPU. Operácie debayeringu, korekcie a indexovania prebiehajú na GPU v jedinom zlúčenom prechode. Najvyššia priepustnosť, vyžaduje najviac VRAM.
* **`tiled_gpu`** — Cesta na GPU s efektívnym využívaním pamäte. Spracováva obrázky v dlaždiciach, aby sa zmestili do obmedzenej pamäte GPU. Nižšia priepustnosť, ale funguje na zariadeniach s obmedzenou pamäťou.
* **`cpu_fallback`** — Spracovanie výlučne na CPU s využitím viacvláknového paralelizmu. Používa sa, ak nie je k dispozícii žiadne grafické jadro NVIDIA, a ako záložný variant v krajnom prípade, ak zlyhajú obe cesty spracovania na GPU.

Reťaz náhradných riešení v prostredí runtime je vždy `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Ručné prepisovanie stratégie

Nastavte premennú prostredia `CHLOROS_STRATEGY`, aby ste vynútili konkrétnu stratégiu — ide o expertný únikový mechanizmus pre prípady, keď automatická detekcia vyberie niečo nevhodné pre vašu situáciu (napríklad ak chcete ponechať grafický procesor voľný pre inú prácu):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Pri porovnávaní premennej sa nerozlišujú veľké a malé písmená; všetko, čo nie je jedným z týchto troch názvov, sa ignoruje a automatická detekcia pokračuje normálne. Aj pri prepisovaní premennej vám Chloros stále vyberá počet pracovníkov:

| Prepis | Použitý počet pracovníkov |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Prepísanie na `GPU_SINGLE` alebo `GPU_PARALLEL` na počítači bez použiteľného zariadenia CUDA je odmietnuté s riadkom `[STRATEGY]` v protokole, a spustenie používa `CPU_PARALLEL`.

Odporúča sa nastaviť to pre jednotlivé príkazy namiesto trvalého nastavenia, aby sa bežné spustenia mohli naďalej automaticky prispôsobovať.

***

## Správanie špecifické pre platformu

| Platforma | Stratégia | Pracovníci | Potrubie | Poznámky |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekvenčné) | Režim s efektívnym využívaním pamäte, jeden obraz naraz |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekvenčné) | Spoločná RAM s kapacitou menej ako 12 GB vynúti sekvenčné spracovanie |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (súbežné) | Odporúčané okrajové zariadenie — obmedzenie Jetsonu na 2 pracovné procesy |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (súbežné) | Maximálny výkon na okraji siete (tiež obmedzené na 2 pracovné procesy pre Jetson) |
| **Stolný počítač s 8 GB GPU** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 pracovníci sa prekrývajú pri I/O, zatiaľ čo semafor sériovo spravuje prístup k GPU |
| **Stolný počítač s GPU s 12 GB a viac** | `GPU_PARALLEL` | 2–4 | `fused_gpu` (súbežne) | Optimálny výkon stolného počítača: 12 GB → 2 pracovníci, 16 GB → 3, 20 GB a viac → 4 |
| **Systém iba s CPU** | `CPU_PARALLEL` | fyzické jadrá − 1 (min. 2) | `cpu_fallback` | Nie je potrebná GPU, používa fond vlákien |

{% hint style="info" %}
**Unifikovaná pamäť Jetson**: Zariadenia Jetson zdieľajú pamäť GPU a CPU. Zariadenie Jetson Orin NX s 16 GB vykazuje ~15,3 GB VRAM, ale ide o tú istú fyzickú RAM, ktorú používajú operačný systém a procesy CPU. Preto sa zariadenia Jetson s 16 GB a viac kvalifikujú pre `GPU_PARALLEL` rovnako ako stolové GPU s 12 GB a viac, avšak sú obmedzené na 2 pracovné procesy – GPU, pracovné procesy a ich kontexty CUDA pre každý pracovný proces čerpajú zo toho istého zdieľaného fondu.
{% endhint %}

### Rozpočet GPU podľa VRAM (diskrétne grafické karty)

Na hostiteľských počítačoch x86_64 s diskrétnou grafickou kartou NVIDIA zistená VRAM tiež určuje, koľko prostriedkov si môže spracovanie na karte vyžiadať a akú veľkosť môžu dosiahnuť dávky:

| Zistená VRAM | Strop rozpočtu GPU | Multiplikátor veľkosti dávky |
| --- | --- | --- |
| **8 GB a viac** | 90 % | ×2,0 |
| **6–8 GB** | 85 % | ×1,75 |
| **3,5–6 GB** | 80 % | ×1,5 |
| **2–3,5 GB** | 75 % | ×1,25 |
| **Menej ako 2 GB** | 70 % | ×1,0 |

Diskrétne grafické procesory vyhradzujú pre systém len 0,5 GB, pretože nezdieľajú systémovú RAM. Profily Jetson vyhradzujú oveľa viac a majú nižšie obmedzenie — pozri [Príručku NVIDIA Jetson](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dynamické prideľovanie pamäte GPU

Chloros používa [4-vláknový spracovateľský kanál](processing-pipeline.md):

* **Vlákno 1** (Detekcia) — Načítanie obrázku, analýza EXIF, detekcia cieľa
* **Vlákno 2** (Kalibrácia) — Výpočet kalibrácie odrazivosti
* **Vlákno 3** (Spracovanie) — Odstránenie Bayerovho vzoru na GPU, korekcia vinetácie, výpočet indexu
* **Vlákno 4** (Export) — Ukladanie súborov, vkladanie metadát

Vlákna 1, 2 a 4 majú nízku spotrebu GPU; Vlákno 3 je najnáročnejšie. Ako sa dokončujú predchádzajúce vlákna spracovateľského reťazca, ich kapacita GPU sa **prerozdelí medzi zostávajúce aktívne vlákna**, takže vlákno 3 získava v priebehu spracovania postupne viac pamäte.

### Fázy alokácie

| Fáza | Aktívne vlákna | Rozdelenie pamäte GPU |
| --- | --- | --- |
| **Začiatok** | 1, 2, 3, 4 | Rozdelené medzi všetky vlákna, väčšina pre vlákno 3 |
| **Stredná fáza** | 2, 3, 4 | Podiel vlákna 1 je prerozdelený |
| **Stredná až neskorá fáza** | 3, 4 | Podiely vlákien 1 a 2 prechádzajú na vlákna 3 a 4 |
| **Neskorá fáza** | 3 alebo 4 | Posledné aktívne vlákno dostane maximálnu alokáciu |

Tieto čísla sa riadia dvoma pravidlami:

* Vlákno, ktoré je **jediným** aktívnym, dostane maximálnu alokáciu podľa svojho profilu.
* Ak je aktívnych viac ako jedna *náročná* úloha na GPU, základná alokácia každej náročnej úlohy sa medzi ne rozdelí (nikdy však neklesne pod nakonfigurované minimum).

Hodnota skutočne použitá počas behu programu je **nižšia** z alokácie profilu platformy a aktuálneho odporúčania z monitora pamäte GPU, takže vyťažená karta má vždy prednosť pred optimistickým profilom.

***

## Spracovanie s ohľadom na textúry

Deyzer s podporou textúr (**len Chloros+** — `--debayer texture-aware`) spúšťa model odšumovania založený na AI/ML, ktorý potrebuje približne 1,75 GB VRAM v FP16 na jednu kópiu, takže využíva oveľa viac pamäte GPU ako štandardná metóda:

* Systémy s **menej ako 7 GB VRAM**spracúvajú Texture Aware v**synchrónnej slučke, jeden obrázok po druhom** — nezmestí sa tam viacero kópií modelu a skupina pracovníkov by len zvýšila konflikt
* Systémy s **7 GB a viac VRAM** môžu spracovávať Texture Aware súbežne, hoci s nižším počtom pracovných procesov v porovnaní so štandardnou metódou
* Na **Jetsone** je Texture Aware vždy priradený k jednému pracovníkovi
* Ak **nie je k dispozícii použiteľný CUDA GPU**, spustenie používa štandardný debayer a backend zaznamená do protokolu jeden riadok `[DEBAYER] … needs a CUDA GPU …` s vysvetlením dôvodu
* Na zariadeniach Jetson s nízkou spotrebou (Nano, Orin Nano) CLI uplatňuje obmedzenie frekvencie GPU, keď požiadate o režim „Texture Aware“ — pozrite si [Príručku NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Riadenie teploty (Jetson)

Zariadenia Jetson majú teplotné obmedzenia, najmä pri inštaláciách v uzavretých priestoroch alebo vo vzduchu. Chloros monitoruje integrované teplotné senzory zariadenia Jetson a automaticky prispôsobuje veľkosť dávok:

| Teplota | Reakcia |
| --- | --- |
| **< 70 °C** | Normálna prevádzka — plná rýchlosť |
| **70 °C** (Varovanie) | Veľkosť dávky sa postupne znižuje (100 % → 50 % v rozmedzí 70 °C až 80 °C) |
| **80 °C** (Kritické) | Agresívne obmedzenie výkonu (50 % → 0 % v rozmedzí 80 °C až 90 °C) |
| **90 °C** (Vypnutie) | Úplné zastavenie spracovania na GPU |

Na stolných počítačoch s dostatočným chladením sa tepelné obmedzovanie spúšťa len zriedka.

***

## Riešenie tlaku na pamäť

Chloros nepretržite sleduje pamäť GPU počas spracovania a reaguje na troch úrovniach.

**Určenie veľkosti dávky.** Dávka začína na 8 obrázkoch vynásobených koeficientom platformy z tabuliek vyššie. Chloros potom skontroluje voľnú VRAM, rezervuje 20 % z nej pre vlastnú réžiu PyTorchu a predpokladá približne 100 MB pamäte GPU na obrázok s rozlíšením 12 MP — veľkosť dávky je tá menšia z dvoch hodnôt: limit odvodený od pamäte alebo základná hodnota platformy. Nikdy neklesne pod hodnotu 1.

**Preventívne zmenšovanie.**Pri**využití VRAM nad 85 %** sa veľkosť dávok zmenší skôr, ako dôjde k zlyhaniu.**Obmedzenie alokácie na vlákno.** Ako stúpa aktuálne využitie, rozpočet GPU pre každé vlákno sa znižuje: ×0,75 pri využití nad 80 %, ×0,5 pri využití nad 90 %. Monitorovacie pásma sú 70 % (konzervatívne), 85 % (normálny prevádzkový limit) a 95 % (riziko nedostatku pamäte).**OOM backoff a obnovenie.** Ak napriek tomu dôjde k udalosti nedostatku pamäte:

* veľkosť dávky sa **zníži na polovicu** a pri každom ďalšom nedostatku pamäte sa zníži opäť na polovicu — každá nasledujúca úspešná dávka posunie túto penalizáciu o jeden krok späť
* alokácie aktívnych vlákien sa znížia na 70 % ich aktuálnej hodnoty a alokátor prejde na konzervatívnu stratégiu, ktorá sa opäť uvoľní po sérii úspešných alokácií
* pri silnom zaťažení sa potrubie vráti z režimu `fused_gpu` do režimu `tiled_gpu` a ako posledná možnosť do režimu `cpu_fallback`

**Pamäť RAM hostiteľa (Jetson).** Pred spracovaním CLI odhadne maximálnu kapacitu pamäte hostiteľa na základe počtu snímok a režimu debayeringu a varuje, ak je pravdepodobné, že RAM spolu so swapom na disku nebude stačiť, pričom vypíše presné príkazy na pridanie swapového priestoru — pozri [Príručku NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Monitorovanie prispôsobenia výpočtovej vrstvy

### Systémová diagnostika

`chloros-cli selftest` je najrýchlejší spôsob, ako overiť, čo vidí výpočtová vrstva:

```bash
chloros-cli selftest
```

Jeho 7 kontrol zahŕňa verziu, dostupnosť portov, spustenie backendu, `/api/test`, informácie o systéme, prítomnosť modelu odšumovača a pripravenosť CUDA + odšumovača. Kontrola 5 priamo vypíše riadok s informáciami o hardvéri:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Kontrola 7 vypíše `CUDA: <bool>, Denoiser: <bool>` — obe musia byť pravdivé, aby bolo možné vôbec používať Texture Aware.

### Protokoly backendu

Stratégia a počet pracovníkov sa volia v rámci backendu na začiatku každého behu — neexistuje žiadny banner v CLI, ktorý by ich oznamoval. Keď sa niečo správa neočakávane (prepnutie na alternatívnu cestu GPU, OOM, odšumovač, ktorý sa nenačíta), objaví sa to v protokole backendu pre danú reláciu:

| Platforma | Umiestnenie protokolu |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (jeden súbor na jedno spustenie) |
| **Linux, backend spustený cez CLI** | tiež `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Priebeh v reálnom čase

Počas behu CLI zobrazuje priebežný priebeh jednotlivých vlákien (Detekcia, Analýza, Spracovanie, Export) prenášaný prostredníctvom Server-Sent Events — praktický prehľad o tom, či je vlákno 3 úzkym miestom. Pozrite si [Spracovateľský potrubie](processing-pipeline.md).

***

## Ďalšie kroky

* [Spracovateľský potrubie](processing-pipeline.md) — Porozumenie architektúre potrubia so 4 vláknami
* [Príručka NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Nasadenie a optimalizácia špecifické pre Jetson
* [CLI: Príkazový riadok](../CLI.md) — Príručka k CLI

* [Referenčná príručka CLI](../reference/cli-reference.md) — Úplný zoznam príkazov pre verziu 1.3.0

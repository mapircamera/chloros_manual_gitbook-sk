# Dynamická adaptácia výpočtov

Chloros 1.2.0 využíva detekciu hardvéru a automatický výber stratégie spracovania. Spracovateľský modul sa prispôsobí vášmu hardvéru – od zariadenia Jetson Orin Nano až po pracovnú stanicu s viacerými grafickými kartami – bez akejkoľvek manuálnej konfigurácie.

***

## Ako to funguje

Pri spustení Chloros vykoná profilovanie vášho systému:

1. **Detekuje operačný systém** — Windows alebo Linux
2. **Identifikuje jadrá procesora a celkovú veľkosť pamäte RAM**

3.**Zistí prítomnosť grafického procesora** — podporu NVIDIA CUDA, VRAM, model
4. **Identifikuje model Jetson** (ak je to relevantné) — prostredníctvom `/proc/device-tree/model`
5. **Skontroluje teplotné senzory** (Jetson) — pre spracovanie zohľadňujúce teplotu
6. **Vyberie výpočtovú stratégiu** — na základe všetkého zisteného hardvéru
7. **Automaticky konfiguruje počet pracovných procesov, typ potrubia a alokáciu pamäte**

Zistený profil sa na dobu trvania relácie ukladá do vyrovnávacej pamäte a na disk, vďaka čomu sa neskoršie spustenia začínajú rýchlejšie:

| Platforma | Profil v vyrovnávacej pamäti |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (uprednostňuje `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Odstráňte tento súbor, aby ste vynútili nové rozpoznanie — užitočné po pridaní grafickej karty alebo väčšieho množstva pamäte RAM. Chloros tiež automaticky vykoná nové rozpoznanie, ak bola vyrovnávacia pamäť zapísaná nekompatibilnou staršou verziou.

***

## Výpočtové stratégie

Chloros vyberá jednu z troch stratégií výpočtov na základe vášho hardvéru:

| Stratégia | Vybrané v prípade | Pracovníci | Vykonávateľ | Potrubie |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| GPU s podporou CUDA hlási**12 GB a viac VRAM**(v jednotnej pamäti Jetson, vyžaduje tiež celkovo 12 GB a viac zdieľanej RAM) | `min(4, VRAM ÷ 4GB)`, minimálne 2 —**obmedzené na 2 na Jetsone** | `ProcessPoolExecutor` (spawn) | `fused_gpu` |
| **`GPU_SINGLE`**| CUDA GPU s**2–12 GB VRAM**| 3 (prekrývanie I/O; prístup k GPU serializovaný semaforom).**1 (sekvenčné) na zariadeniach Jetson s menej ako 12 GB RAM** | `ProcessPoolExecutor` (spawn); sekvenčné v rámci procesu na zariadeniach Jetson s nízkou kapacitou RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Bez grafického procesora CUDA alebo s menej ako 2 GB VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Praktické príklady vzorca pre počet pracovníkov `GPU_PARALLEL`: 12 GB VRAM → 3 pracovníci, 16 GB a viac → 4 pracovníci, akýkoľvek Jetson → 2 pracovníci.

Paralelizmus je implementovaný pomocou štandardného `concurrent.futures` v Python: stratégie GPU používajú `ProcessPoolExecutor` s metódou spustenia **spawn** (každý pracovník je samostatný proces s vlastným kontextom CUDA — `fork` by skopíroval už inicializovaný stav CUDA a poškodil by potomkov), a stratégia CPU používa `ThreadPoolExecutor`. Chloros nepoužíva žiadny distribuovaný framework tretej strany (napríklad Ray).

### Typy spracovateľských ciest

* **`fused_gpu`** — Cesta spracovania úplne na GPU. Operácie debayeringu, korekcie a indexovania prebiehajú na GPU v jednom zjednotenom prechode. Najvyššia priepustnosť, vyžaduje najviac VRAM.
* **`tiled_gpu`** — Cesta na GPU s efektívnym využitím pamäte. Spracováva obrázky po dlaždiciach, aby sa zmestili do obmedzenej pamäte GPU. Nižšia priepustnosť, ale funguje na zariadeniach s obmedzenou pamäťou.
* **`cpu_fallback`** — Spracovanie výlučne na CPU s využitím viacvláknovej paralelizácie. Používa sa, ak nie je k dispozícii žiadne grafické jadro NVIDIA, a ako záložná možnosť v krajnom prípade, ak zlyhajú obe cesty spracovania na grafickom jadre.

Reťaz náhradných riešení v prostredí behu je vždy `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Ručné prepisovanie stratégie

Nastavte premennú prostredia `CHLOROS_STRATEGY`, aby ste vynútili konkrétnu stratégiu — ide o expertnú poistku pre prípady, keď automatická detekcia vyberie niečo, čo nie je pre vašu situáciu vhodné (napríklad ak chcete ponechať grafický procesor voľný pre inú prácu):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Pri porovnávaní premennej sa nerozlišujú veľké a malé písmená; všetko, čo nie je jedným z týchto troch mien, sa ignoruje a automatická detekcia pokračuje normálne. Aj pri prepisovaní vám Chloros stále vyberie počet pracovníkov:

| Prepis | Použitý počet pracovníkov |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Odporúčame nastaviť to pre jednotlivé príkazy namiesto trvalého nastavenia, aby sa bežné spustenia mohli naďalej automaticky prispôsobovať.

***

## Správanie špecifické pre platformu

| Platforma | Stratégia | Pracovníci | Potrubie | Poznámky |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekvenčné) | Režim s nízkou spotrebou pamäte, jeden obrázok naraz |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekvenčné) | Spoločná RAM s kapacitou menšou ako 12 GB núti k sekvenčnému spracovaniu |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (súbežné) | Odporúčané okrajové zariadenie — obmedzenie Jetson na 2 pracovné procesy |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (súbežné) | Maximálny výkon na okraji siete (tiež obmedzený na 2 pracovníkov pre Jetson) |
| **Stolný počítač s 8 GB GPU** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 pracovné procesy sa prekrývajú pri I/O, zatiaľ čo semafor sériovo spravuje prístup k GPU |
| **Stolný počítač s GPU s 12 GB a viac** | `GPU_PARALLEL` | 3–4 | `fused_gpu` (súbežne) | Optimálny výkon stolného počítača: 12 GB → 3 pracovníci, 16 GB a viac → 4 |
| **Systém len s CPU** | `CPU_PARALLEL` | fyzické jadrá − 1 (min. 2) | `cpu_fallback` | Nie je potrebná GPU, využíva fond vlákien |

{% hint style="info" %}
**Unifikovaná pamäť Jetson**: Zariadenia Jetson zdieľajú pamäť GPU a CPU. Zariadenie Jetson Orin NX s 16 GB vykazuje ~15,3 GB VRAM, ide však o tú istú fyzickú RAM, ktorú používajú operačný systém a procesy CPU. Preto sa zariadenia Jetson s 16 GB a viac kvalifikujú pre `GPU_PARALLEL` rovnako ako stolové grafické karty s 12 GB a viac, avšak sú obmedzené na 2 pracovné procesy – grafická karta, pracovné procesy a ich kontexty CUDA pre každý pracovný proces čerpajú zo toho istého zdieľaného fondu.
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
* **Vlákno 3** (Spracovanie) — Odstránenie matice pixelov na GPU, korekcia vinetácie, výpočet indexu
* **Vlákno 4** (Export) — Ukladanie súborov, vkladanie metadát

Vlákna 1, 2 a 4 majú nízku spotrebu GPU; vlákno 3 je najnáročnejšie. Ako sa dokončujú predchádzajúce vlákna spracovateľského reťazca, ich kapacita GPU sa **prerozdelí medzi zostávajúce aktívne vlákna**, takže vlákno 3 získava postupne viac pamäte v priebehu spracovania.

### Fázy prideľovania

| Fáza | Aktívne vlákna | Rozdelenie pamäte GPU |
| --- | --- | --- |
| **Počiatočná** | 1, 2, 3, 4 | Rozdelené medzi všetky vlákna, väčšina pre vlákno 3 |
| **Stredná-raná** | 2, 3, 4 | Podiel vlákna 1 je prerozdelený |
| **Stredná-neskorá** | 3, 4 | Podiely vlákien 1 a 2 prechádzajú na vlákna 3 a 4 |
| **Neskorá fáza** | 3 alebo 4 | Posledné aktívne vlákno dostane maximálnu alokáciu |

Tieto čísla sa riadia dvoma pravidlami:

* Vlákno, ktoré je **jediným** aktívnym, dostane maximálnu alokáciu podľa svojho profilu.
* Ak je aktívnych viac ako jedna *náročná* úloha na GPU, základná alokácia každej náročnej úlohy sa medzi ne rozdelí (nikdy však klesne pod nakonfigurované minimum).

Hodnota skutočne použitá počas behu je **nižšia** z alokácie profilu platformy a aktuálneho odporúčania z monitora pamäte GPU, takže vyťažená karta má vždy prednosť pred optimistickým profilom.***

## Spracovanie s ohľadom na textúry

Debuter s ohľadom na textúry (**len Chloros+** — `--debayer texture-aware`) spúšťa model odšumovania založený na AI/ML, ktorý potrebuje približne 1,75 GB VRAM v FP16 na jednu kópiu, takže využíva oveľa viac pamäte GPU než štandardná metóda:

* Systémy s **menej ako 7 GB VRAM**spracúvajú Texture Aware v**synchrónnej slučke, jeden obrázok po druhom** — nezmestí sa tam viacero kópií modelu a skupina pracovníkov by len zvýšila konflikt
* Systémy s **7 GB a viac VRAM** môžu spracovávať Texture Aware súbežne, hoci s nižším počtom pracovných procesov v porovnaní so štandardnou metódou
* Na **Jetsone** je Texture Aware vždy priradený k jednému pracovníkovi a na modeloch s nízkou spotrebou energie (Nano, Orin Nano) sa automaticky uplatňuje aj obmedzenie frekvencie GPU — pozri [Príručku NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)***

## Riadenie teploty (Jetson)

Zariadenia Jetson majú teplotné obmedzenia, najmä pri inštaláciách v uzavretých priestoroch alebo vo vzduchu. Chloros monitoruje integrované teplotné senzory zariadenia Jetson a automaticky prispôsobuje veľkosť dávok:

| Teplota | Reakcia |
| --- | --- |
| **&lt; 70 °C** | Normálna prevádzka – plná rýchlosť |
| **70 °C** (Varovanie) | Veľkosť dávky sa postupne znižuje (100 % → 50 % v rozmedzí 70 °C až 80 °C) |
| **80 °C** (Kritické) | Agresívne obmedzenie výkonu (50 % → 0 % v rozmedzí 80 °C až 90 °C) |
| **90 °C** (Vypnutie) | Úplné zastavenie spracovania GPU |

Na stolných počítačoch s dostatočným chladením sa tepelné obmedzenie spúšťa len zriedka.

***

## Spracovanie zaťaženia pamäte

Chloros nepretržite monitoruje pamäť GPU počas spracovania a reaguje na troch úrovniach.

**Veľkosť dávky.** Dávka začína na 8 obrázkoch vynásobených koeficientom platformy z tabuliek uvedených vyššie. Chloros následne skontroluje voľnú VRAM, vyhradí 20 % z nej pre vlastnú réžiu PyTorchu a predpokladá približne 100 MB pamäte GPU na jeden obrázok s rozlíšením 12 MP — veľkosť dávky je daná menšou z hodnôt: limitom odvodeným od pamäte alebo základnou hodnotou platformy. Nikdy neklesne pod hodnotu 1.**Preventívne zníženie.**Pri**využití VRAM nad 85 %** sa veľkosti dávok znížia skôr, než dôjde k zlyhaniu.**Obmedzenie alokácie na vlákno.** Ako stúpa aktuálne využitie, rozpočet GPU pre každé vlákno sa znižuje: ×0,75 pri využití nad 80 %, ×0,5 pri využití nad 90 %. Monitorovacie pásma sú 70 % (konzervatívne), 85 % (normálny prevádzkový limit) a 95 % (riziko OOM).**OOM backoff a obnova.** Ak napriek tomu dôjde k udalosti nedostatku pamäte (OOM):

* veľkosť dávky sa **zníži na polovicu** a pri každom ďalšom nedostatku pamäte sa zníži opäť na polovicu — každá nasledujúca úspešná dávka túto penalizáciu posunie o jeden krok späť
* alokácie aktívnych vlákien sa znížia na 70 % ich aktuálnej hodnoty a alokátor prejde na konzervatívnu stratégiu, ktorá sa opäť uvoľní po sérii úspešných alokácií
* pri silnom zaťažení sa potrubie vráti z režimu `fused_gpu` do režimu `tiled_gpu` a ako posledná možnosť do režimu `cpu_fallback`

**Pamäť RAM hostiteľa (Jetson).** Pred spracovaním odhadne CLI maximálnu kapacitu pamäte hostiteľa na základe počtu snímok a režimu debayeringu a varuje, ak je pravdepodobné, že RAM spolu so swapom podporovaným súbormi nebude postačovať, pričom vypíše presné príkazy na pridanie swapového priestoru — pozri [Príručku NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).***

## Monitorovanie prispôsobenia výpočtov

### Systémová diagnostika

`chloros-cli selftest` je najrýchlejší spôsob, ako overiť, čo vidí výpočtová vrstva:

```bash
chloros-cli selftest
```

Jeho 7 kontrol zahŕňa verziu, dostupnosť portov, spustenie backendu, `/api/test`, informácie o systéme, prítomnosť modelu odšumovača a pripravenosť CUDA + odšumovača. Kontrola 5 priamo vypíše hardvérovú líniu:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Kontrola 7 vypíše `CUDA: <bool>, Denoiser: <bool>` — obe musia byť pravdivé, aby bolo možné vôbec používať Texture Aware.

### Protokoly backendu

Stratégia a počet pracovníkov sa volia v rámci backendu na začiatku každého behu — neexistuje žiadny banner CLI, ktorý by ich oznamoval. Keď sa niečo správa neočakávane (prepad na GPU cestu, OOM, denoiser, ktorý sa nenačíta), objaví sa to v protokole backendu pre danú reláciu:

| Platforma | Umiestnenie protokolu |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (jeden súbor na jedno spustenie) |
| **Linux, CLI – spustený backend** | tiež `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Priebeh v reálnom čase

Počas behu zobrazuje súbor CLI priebeh jednotlivých vlákien v reálnom čase (detekcia, analýza, spracovanie, export) prostredníctvom Server-Sent Events — praktický ukazovateľ toho, či je vlákno 3 úzkym miestom. Pozrite si [Spracovateľský kanál](processing-pipeline.md).

***

## Ďalšie kroky

* [Spracovateľský kanál](processing-pipeline.md) — Porozumenie architektúre kanála so 4 vláknami
* [Príručka NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Nasadenie a optimalizácia špecifická pre Jetson
* [CLI : Príkazový riadok](../CLI.md) — Príručka k CLI
* [Referenčná príručka k CLI](../reference/cli-reference.md) — Úplný zoznam príkazov pre verziu 1.2.0

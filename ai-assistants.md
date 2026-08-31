# Používanie Chloros s AI asistentmi

Táto príručka je určená pre dve cieľové skupiny: ľudí a AI asistentov, s ktorými ľudia čoraz častejšie pracujú. Na každej strane sú uvedené presné hodnoty, predvolené nastavenia a príkazy, ktoré je možné skopírovať a vložiť, aby asistent (Claude, ChatGPT, Copilot, programovací agent atď.) mohol na prvý pokus napísať fungujúcu automatizáciu Chloros.

Verzia Chloros: **

1.2.0**. Platformy CLI/SDK: Windows 10/11 x64 a Linux (x86_64 / Jetson aarch64).

## Čo odovzdať svojmu asistentovi

| Zdroj | URL | Na čo slúži |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Strojovo čitateľný index všetkých stránok v tejto príručke. |
| **CLI Referencia** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Kompletná príkazová sada `chloros-cli`: všetky príkazy, príznaky, predvolené nastavenia, kódy ukončenia a pravidlá pre výstupné priečinky. Napísané pre použitie v LLM. |
| **Referencia SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Kompletný prehľad `chloros_sdk` Python API: triedy, signatúry, výnimky a praktické príklady. Napísané pre študentov magisterského štúdia (LLM). |
| **Akákoľvek stránka vo formáte surového Markdownu** | pripojte `.md` k stránke URL | napr. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` vráti stránku ako surový Markdown — ideálne na vloženie do kontextového okna alebo na načítanie z agenta. |

Odkazy v príručke: [CLI Referencia](reference/cli-reference.md) · [SDK Referencia](reference/sdk-reference.md).

{% hint style="info" %}
Tieto dve referenčné stránky sú samostačné: asistent, ktorý si prečítal jednu z nich, nepotrebuje zvyšok príručky na napísanie správneho skriptu.
{% endhint %}

## Príklady príkazov

Skopírujte, vyplňte `<placeholders>` a vložte do svojho asistenta.

### 1. Spracovanie zložky letov do NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Hromadné sledovanie adresára so záznamami

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Pripojenie pole LATTICE a zaznamenanie

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Zaznamenajte spektrá svetelného senzora DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Skriptovanie DAQ z príkazového riadku vždy prebieha prostredníctvom rodiny `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Ďalšie podpríkazy typu `daq`, ktoré si váš asistent môže vymyslieť, nie sú v dodávaných verziách k dispozícii a ich spustenie skončí chybou.
{% endhint %}

## Prečo skripty napísané umelou inteligenciou fungujú dobre s Chloros

Každé z nich je skutočné, overené správanie Chloros 1.2.0 — odstraňujú klasické chyby automatizácie vytvorenej strojovo:

* **Žiadne zložité nastavovanie.**Inteligentné pomocné funkcie SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) a vstupné body spracovania (`ChlorosLocal`, `process_folder`)**automaticky spúšťajú lokálny backend**. Generovaný skript nevyžaduje otvorené grafické rozhranie ani ručne spustený server — potrebuje iba nainštalovaný balík desktop/CLI.
* **Celý proces prebieha v rámci jedného volania.** `chloros_sdk.process_folder("path", indices=["NDVI"])` vykonáva import → kalibráciu → odrazivosť → export indexu od začiatku do konca. Menšia plocha, menej miest, kde by sa generovaný skript mohol pokaziť.
* **Spustenia bez výstupu sa samodiagnostikujú.** Po `process()` sa k výsledku pripojí súhrn spustenia a každý tip na spracovanie (napr. *prečo* spustenie nevytvorilo žiadny výstup) sa opäť vygeneruje ako Python `UserWarning` — takže aj skript, ktorý nikdy nekontroluje výsledný slovník, zobrazí diagnózu.
* **CLI zlyhá s výraznou chybou.**Spustenie `chloros-cli process`, ktoré požiadalo o výstupy, ale žiadne nezapísalo, vypíše `Processing finished but wrote no image products.` a**ukončí sa s nenulovým kódom**, takže skripty v shelli a CI to zistia jednoduchou kontrolou kódu ukončenia. Úspešné spustenia hlásia kód `Image products written: N`.

Jedna asymetria, ktorú by mal asistent poznať: `process()` v rámci SDK zámerne **nevyvolá** výnimku pri behu bez produktov — namiesto toho to nahlási prostredníctvom súhrnu/tipov. Ak sa musí potrubie Python zastaviť pri prázdnom behu, skontrolujte súhrn (recept 2 to robí).

## Upozornenia

* **Chloros+ vyžaduje prihlásenie.**CLI a SDK vyžadujú**platenú** úroveň Chloros+, čo sa vynucuje na strane servera: požiadavky zlyhajú s chybou `401 AUTH_REQUIRED`, ak nie ste prihlásení, a s chybou `403 PLAN_UPGRADE_REQUIRED` na bezplatnej úrovni. Pred spustením vygenerovaných skriptov spustite na každom počítači príkaz `chloros-cli login`. Pozrite si [Chloros+ Prihlásenie](chloros+-login.md).
* **Príkazy na zachytávanie riadia skutočný hardvér.** Príkazy `lattice` / `daq` / `project` a objekty relácie SDK sa pripájajú k fyzickým kamerám a senzorom, prenášajú z nich dáta a spúšťajú ich. Pred prvým spustením skontrolujte vygenerovaný skript a spustite ho za prítomnosti hardvéru.
* **Náhodne skontrolujte výstupy.** Pred zverejnením výsledkov overte priečinky s produktmi a niekoľko hodnôt pixelov. Najmä TIFF súbory odrazivosti sú škálované podľa zdroja — prečítajte si XMP značku `Chloros:PixelScale` (LATTICE: 32768 = odrazivosť 1,0; Survey3: 65535) namiesto predpokladania deliteľa. Obe referenčné stránky to dokumentujú v časti „Čítanie pixelov odrazivosti“.
* **Drobné úskalia, ktoré môžu spôsobiť chyby v generovanom kóde:**`pool-record` zapisuje do súborového systému**backendového hostiteľa** (predvolené nastavenie `~/Documents/DAQ Live View/`); na počítačoch s viacerými sieťovými rozhraniami uprednostnite `daq pool-connect --eth-host <ip-or-hostname>` pred automatickým vyhľadávaním; a kdekoľvek sa objaví backend URL, použite `http://127.0.0.1:5000` (nikdy `localhost`).

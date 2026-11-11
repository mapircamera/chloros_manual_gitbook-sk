# Používanie Chlorosu s AI asistentmi

Táto príručka je určená pre dve cieľové skupiny: ľudí a umelú inteligenciu, s ktorou ľudia čoraz častejšie pracujú. Na každej strane sú uvedené presné hodnoty, predvolené nastavenia a príkazy, ktoré je možné skopírovať a vložiť, aby asistent (Claude, ChatGPT, Copilot, programovací agent atď.) mohol na prvý pokus napísať fungujúcu automatizáciu v Chloros.

Verzia Chloros: **

1.3.0**. Platformy CLI/SDK: Windows 10/11 x64 a Linux (x86\_64 / Jetson aarch64).

## Čo poskytnúť svojmu asistentovi

| Zdroj | URL | Na čo slúži |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Strojovo čitateľný index všetkých stránok v tejto príručke. |
| **Referencia CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Kompletný prehľad príkazov `chloros-cli`: všetky príkazy, príznaky, predvolené nastavenia, kódy ukončenia a pravidlá pre výstupné priečinky. Napísané pre použitie v LLM. |
| **Referencia SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Kompletné Python API pre `chloros_sdk`: triedy, signatúry, výnimky a praktické príklady. Napísané pre použitie v LLM. |
| **Akákoľvek stránka vo formáte surového Markdownu** | pripojte `.md` k URL stránky | napr. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` vráti stránku vo formáte surového Markdownu — ideálne na vloženie do kontextového okna alebo na načítanie z agenta. |

Odkazy v príručke: [Referencia CLI](reference/cli-reference.md) · [Referencia SDK](reference/sdk-reference.md).

{% hint style="info" %}
Tieto dve referenčné stránky sú samostatočné: asistent, ktorý si prečítal jednu z nich, nepotrebuje zvyšok príručky na napísanie správneho skriptu.
{% endhint %}

## Nechajte svojho asistenta spustiť Chloros priamo: `chloros-mcp`

Všetko uvedené vyššie vyžaduje, aby váš asistent *napísal* skript alebo príkaz. S `chloros-mcp` asistent, ktorý ovláda [Model Context Protocol](https://modelcontextprotocol.io) (MCP), ako napríklad Claude Desktop, Claude Code alebo Cursor, ovláda samotný program Chloros. Stačí požiadať bežným jazykom a asistent vyvolá nástroje servera:

* **Spracovanie snímok:** vytvorenie projektu, import zložky so snímkami, výber nastavení, spracovanie a vygenerovanie správy o výsledkoch spustenia.
* **Ovládanie hardvéru:** pripojenie kamier LATTICE, snímačov a svetelných senzorov DAQ, náhľad, nastavenie expozície a farieb, snímanie, nahrávanie a aktualizácia firmvéru.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Nástroje servera pokrývajú funkcie, ktoré poskytuje [Python SDK](api-python-sdk.md). Inštalujú sa spolu s SDK, takže zvyčajne nie je potrebné inštalovať nič ďalšie: v systéme Windows inštalátor Chloros umiestni súbor `chloros-mcp.exe` do priečinka `Scripts` vášho Pythonu, a v systéme Linux `.deb` umiestni súbor `chloros-mcp` do adresára `/usr/bin`.

**Kód Claude:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**pridajte to do súboru `claude_desktop_config.json` (otvoríte ho cez**Nastavenia &gt; Vývojár &gt; Upraviť konfiguráciu**), potom reštartujte aplikáciu Claude Desktop.**Kurzor** používa rovnaký záznam v súbore `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Ak klient hlási, že sa server nepodarilo spustiť, položka `chloros-mcp` pravdepodobne nie je v zozname `PATH`, ktorý vidí: zadajte namiesto toho úplnú cestu (`where chloros-mcp` v systéme Windows, `which chloros-mcp` v systéme Linux).

Na tom istom počítači musíte mať nainštalovaný Chloros a byť tam prihlásený, buď v aplikácii, alebo prostredníctvom `chloros-cli login`, a to na akejkoľvek platenej úrovni Chloros+ (Copper alebo vyššej). Server sa za vás nemôže prihlásiť.

{% hint style="info" %}
Nastavenie klienta pre každú aplikáciu, možnosti, každý nástroj a správanie spustenia: [MCP Server (AI Assistants)](reference/sdk-reference.md#mcp-server-ai-assistants) v referenčnom materiáli k SDK.
{% endhint %}

## Vzorové príkazy

Skopírujte, vyplňte `<placeholders>` a vložte do svojho asistenta.

### 1. Spracovanie zložky letu na NDVI

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

### 3. Pripojenie matice LATTICE a zaznamenávanie

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Záznam spektier svetelných senzorov DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Skriptovanie DAQ z príkazového riadku vždy prebieha prostredníctvom rodiny `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Ostatné podpríkazy typu `daq`, ktoré si váš asistent môže vymyslieť, nie sú v dodávaných verziách k dispozícii a ich spustenie skončí chybou.
{% endhint %}

## Prečo skripty napísané umelou inteligenciou dobre fungujú s Chloros

Každý z týchto prípadov je skutočné, overené správanie Chloros 1.3.0 — odstraňujú klasické chyby automatizácie napísanej strojovo:

* **Žiadne zložité nastavovanie.**Pomocné funkcie SDK pre inteligentné pripojenie (`connect_camera`, `connect_array`, `connect_daq_sensor`) a vstupné body spracovania (`ChlorosLocal`, `process_folder`)**automaticky spúšťajú lokálny backend**. Generovaný skript nepotrebuje otvorené grafické rozhranie ani ručne spustený server — stačí, ak je nainštalovaný balík pre desktop alebo CLI.
* **Celý proces prebieha v rámci jedného volania.** `chloros_sdk.process_folder("path", indices=["NDVI"])` vykonáva import → kalibráciu → odrazivosť → export indexu od začiatku do konca. Menšia plocha, menej miest, kde by sa generovaný skript mohol pokaziť.
* **Spustenia bez výstupu sa samodiagnostikujú.** Po `process()` sa k výsledku pripojí súhrn spustenia a každý tip týkajúci sa spracovania (napr. *prečo* spustenie nevytvorilo žiadny výstup) sa opätovne vygeneruje ako skript v jazyku Python`UserWarning` — takže aj skript, ktorý nikdy nekontroluje výsledný slovník, zobrazí túto diagnostiku.
* **CLI zlyhá s jasným upozornením.**Spustenie s kódom `chloros-cli process`, ktoré požiadalo o výstupy, ale žiadne nezapísalo, vypíše kód `Processing finished but wrote no image products.` a**ukončí sa s nenulovým kódom**, takže skripty v shelli a CI to zistia jednoduchou kontrolou kódu ukončenia. Úspešné spustenia hlásia `Image products written: N`, plus `(plus M clamp masks)`, ak boli zapísané masky. Príkazy na zachytávanie rovnako uvádzajú názvy všetkých súborov, ktoré sa nepodarilo zapísať, a ukončia sa s kódom `1`.
* **Relatívne cesty znamenajú to, čo skript očakáva.** Pri lokálnom backende sa relatívna výstupná cesta (`-o out/` v CLI, `output_dir="out/"` v SDK) rieši vo vzťahu k priečinku, z ktorého sa príkaz alebo skript spúšťa.

Jedna asymetria, ktorú by mal asistent poznať: `process()` v SDK zámerne **nevznesie** výnimku pri spustení s nulovým výstupom — namiesto toho to nahlási prostredníctvom súhrnu/tipov. Ak sa Pythonový pipeline musí zastaviť pri prázdnom spustení, skontrolujte súhrn (recept 2 to robí).

## Upozornenia

* **Vyžaduje sa prihlásenie do Chloros+.**CLI a SDK vyžadujú**platenú** úroveň Chloros+, čo sa vynucuje na strane servera: požiadavky zlyhajú s chybou `401 AUTH_REQUIRED`, ak nie ste prihlásení, a s chybou `403 PLAN_UPGRADE_REQUIRED` v bezplatnej úrovni. Pred spustením vygenerovaných skriptov spustite príkaz `chloros-cli login` raz na každom počítači. Pozrite si [Prihlásenie do Chloros+](chloros+-login.md).
* **Príkazy na zachytávanie ovládajú skutočný hardvér.** Príkazy `lattice` / `daq` / `project`, objekty relácie SDK a nástroje `chloros-mcp` sa pripájajú k fyzickým kamerám a senzorom, prenášajú z nich dáta a spúšťajú ich. Pred prvým spustením si skontrolujte vygenerovaný skript a spustite ho za prítomnosti hardvéru.
* **Náhodne skontrolujte výstupy.** Pred zverejnením výsledkov overte priečinky s produktmi a niekoľko hodnôt pixelov. Najmä TIFF súbory odrazivosti sú škálované podľa zdroja — prečítajte si XMP tag `Chloros:PixelScale` (súbory LATTICE ho obsahujú: 32768 = odrazivosť 1,0; súbory Survey3 ho neobsahujú a používajú 65535 = 1,0) namiesto predpokladania deliteľa. Obe referenčné stránky to dokumentujú v časti „Čítanie pixelov odrazivosti“.
* **Drobné úskalia, ktoré môžu spôsobiť chyby v generovanom kóde:**`pool-record` zapisuje do súborového systému**backendového hostiteľa** (predvolené nastavenie je `~/Documents/DAQ Live View/`); na počítačoch s viacerými sieťovými rozhraniami uprednostňujte `daq pool-connect --eth-host <ip-or-hostname>` pred automatickým vyhľadávaním; použite `http://127.0.0.1:5000` (nikdy `localhost`) všade tam, kde sa objaví URL backendu; skript, ktorý prechádza výstupným stromom projektu, by mal preskočiť zložky `clamp_masks/` (ich súbory `*_clampmask.tif` sú masky, nie obrázky — pozri [Formáty výstupných obrázkov](output-image-formats.md#clamp-masks)); a v systéme Windows, kde je vypnutá funkcia dlhých ciest, udržiavajte zložky projektu krátke — zložka projektu s názvom dlhším ako približne 130 znakov bude odmietnutá ešte pred začatím spracovania.

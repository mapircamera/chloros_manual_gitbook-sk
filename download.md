---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Stiahnutie

Stiahnite si najnovšiu verziu programu Chloros
, aby ste mohli začať so spracovaním multispektrálnych snímok.

### Systémové požiadavky

#### Windows


| Požiadavka          | Minimálne                                              | Odporúčané                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operačný systém** | Windows
10 (64-bitový)                                  | Windows
11 (64-bitový)                                  |
| **Procesor**        | Intel Core i5 alebo ekvivalent                          | Intel Core i7 alebo lepší                              |
| **Pamäť (RAM)**     | 8 GB                                                  | 16 GB alebo viac                                         |
| **Grafická karta**    | Kompatibilná s DirectX 11                                | Grafická karta NVIDIA s 4 GB a viac VRAM                            |
| **Úložný priestor**          | 6 GB voľného miesta                                       | SSD s 10 GB a viac voľného miesta                            |
| **Displej**          | 1920x1080                                            | 2560x1440 alebo vyššie                                  |
| **Internet**         | Potrebné na \[voliteľnú] aktiváciu licencie Chloros
+ | Potrebné na \[voliteľnú] aktiváciu licencie Chloros
+ |

#### Linux
amd64 (x86\_64)

| Požiadavky       | Minimálne                    | Odporúčané               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribúcia**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Procesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 alebo lepší   |
| **Pamäť (RAM)**  | 8 GB                        | 16 GB alebo viac              |
| **Grafická karta** | Žiadna (spracovanie na CPU)      | Grafická karta NVIDIA s 4 GB+ VRAM |
| **Úložný priestor**       | 2 GB voľného miesta             | SSD s 10 GB+ voľného miesta       |
| **Python**        | Python
3.7+ (pre SDK
)      | Python
3.10+              |

#### Linux
arm64 (NVIDIA Jetson)

| Požiadavky      | Minimálne                      | Odporúčané                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platforma**     | NVIDIA Jetson s JetPack 6 | Jetson Orin NX 16 GB alebo AGX Orin |
| **Pamäť (RAM)** | 8 GB (zdieľaná medzi GPU a CPU)         | 16 GB a viac zdieľanej                    |
| **Úložisko**      | 2 GB voľného miesta               | NVMe SSD s 10 GB+ voľného miesta        |
| **Python**       | Python
3.7+ (pre SDK
)        | Python
3.10+                    |

{% hint style="info" %}
**Akcelerácia GPU**: Používatelia Chloros
+ s grafickými kartami NVIDIA môžu využiť akceleráciu CUDA na výrazne rýchlejšie spracovanie. Funguje to s desktopovými grafickými kartami NVIDIA v systémoch Windows
a Linux
amd64, ako aj na zariadeniach NVIDIA Jetson s JetPack 6. Používatelia Chloros
+ tiež získavajú viacvláknové spracovanie pre maximálnu rýchlosť.
{% endhint %}

***

## Stiahnutie Chloros


### Najnovšia stabilná verzia: Verzia 1.3.0 (11. november 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Stiahnite si Chloros pre Windows (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Stiahnite si Chloros pre Linux amd64 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Stiahnite si Chloros pre Linux arm64 / Jetson (.deb)</a>


#### Inštalátor pre Windows
(GUI + CLI
+ Backend)

* **Typ súboru**: .exe (inštalátor pre Windows
)

**Kroky inštalácie:**

1. Stiahnite si uvedený súbor .exe
2. Dvojitým kliknutím na inštalačný súbor spustite inštaláciu
3. Postupujte podľa pokynov inštalačného sprievodcu
4. Zvoľte inštalačný adresár (predvolený: `C:\Program Files\Chloros\`)
5. Na stránke **Vybrať ďalšie úlohy**nechajte zaškrtnuté obe možnosti**Systémové nastavenia** (odporúčané – pozri nižšie)
6. Dokončite inštaláciu a spustite Chloros
alebo Chloros
CLI
7. Prihláste sa pomocou svojho [účtu MAPIR
Cloud Chloros
+](https://cloud.mapir.camera/pricing) (alebo pokračujte s bezplatnou verziou)

**Možnosti systémových nastavení** (obe sú predvolene zaškrtnuté):

| Možnosť, ako je uvedené v inštalátore | Čo robí | Vypnutie tichej inštalácie |
| --- | --- | --- |
| **Povoliť dlhé cesty v systéme Windows** (umožňuje, aby adresáre projektov mali názvy dlhšie ako 260 znakov; odporúčané) | Nastaví systémové nastavenie `LongPathsEnabled` v Windows
na hodnotu 1. Windows
sa dodáva s touto voľbou vypnutou, pričom žiadna cesta k súboru nesmie byť dlhšia ako 260 znakov; produkty Chloros
sa nachádzajú približne 130 znakov pod zložkou projektu, takže ak sú dlhé cesty vypnuté, hlbšia zložka projektu nebude akceptovaná. Nastavenie nadobúda platnosť pre programy spustené po inštalácii. Zostáva zapnuté aj po odinštalovaní, pretože na ňom môžu závisieť iné programy. | `/MERGETASKS="!longpaths"` |
| **Vylúčiť backend Chloros
u z kontroly programom Microsoft Defender** (oveľa rýchlejšie snímanie z kamery; odporúčané) | Pridá vylúčenie procesu Microsoft Defender pre `chloros-backend.exe` v inštalačnej zložke, takže ochrana v reálnom čase nekontroluje každý súbor, ktorý backend zapíše, bez ohľadu na to, kam ho zapíše. Pri vysokej frekvencii snímania predstavuje skenovanie približne 30 % frekvencie snímania (77,7 oproti 52,7 snímok za sekundu pri nepretržitom snímaní z 9 kamier). Žiaden iný program nie je vylúčený a vylúčenie sa pri odinštalovaní odstráni. Ak program Defender chýba alebo je zablokovaný zásadami, tento krok sa ticho nezrealizuje a inštalácia pokračuje. | `/MERGETASKS="!defenderexclusion"` |

Tichá inštalácia (`/SILENT` alebo `/VERYSILENT`) uplatňuje obe možnosti, pokiaľ sa z toho neodhlásite pomocou zobrazeného nastavenia `/MERGETASKS`.

{% hint style="success" %}
Inštalátor pridá priečinok CLI
(`C:\Program Files\Chloros\cli\`) do vášho systému `PATH` len raz — pri opätovnej inštalácii alebo aktualizácii sa už nepridá duplicitný záznam a odstránia sa duplicitné záznamy pridané predchádzajúcimi inštalátormi. Po inštalácii otvorte nový terminál, aby ste mohli používať `chloros-cli`. Inštalačný program tiež pridá do každého sieťového profilu dve pravidlá brány Windows
Firewall pre backend: **Chloros
Backend (UDP-In)**, pre prúdy z kamier, PTP, mDNS a dátový prúd DAQ-E, a**Chloros
Backend (TCP-In)**, obmedzené na lokálnu podsieť, pre aktualizáciu firmvéru DAQ-E, pri ktorej sa senzor opäť pripojí k tomuto počítaču, aby získal obraz. Obe pravidlá sa pri odinštalovaní odstránia.
{% endhint %}

#### Linux
amd64 (balík .deb — CLI
+ backend)

* **Typ súboru**: .deb (balík pre Debian/Ubuntu)
* **Architektúra**: x86\_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux
arm64 — NVIDIA Jetson (balík .deb — CLI
+ backend)

* **Typ súboru**: .deb (JetPack 6)
* **Architektúra**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Použite názov súboru, ktorý ste skutočne stiahli, a ponechajte `./`, aby `apt` nainštaloval práve tento súbor namiesto hľadania balíka podľa názvu. Inštalujte pomocou `apt install`, nie pomocou `dpkg -i`: apt nainštaluje aj požiadavky Python
SDK
, ktoré balík odporúča, ale nevyžaduje, a `dpkg -i` ich preskočí. Od verzie 1.3.0 sa backend a CLI
inštalujú v rozbalenom stave pod `/usr/lib/chloros/`, takže sa spúšťajú bez toho, aby sa najprv museli rozbaliť.

Balík pre Jetson používa verziu PyTorch pre CUDA z JetPack 6, takže spracovanie, vrátane debayera Texture Aware, prebieha na GPU zariadenia Jetson. Využíva knižnice CUDA a cuDNN, ktoré inštaluje JetPack 6 (`sudo apt install nvidia-jetpack` na obraze, kde chýbajú).

Podrobné pokyny na nastavenie nájdete v [Inštalácii pre Linux
](linux/linux-installation.md) a pokyny špecifické pre Jetson v [Príručke NVIDIA Jetson](linux/nvidia-jetson-guide.md).

#### Python
SDK
(všetky platformy)

Každý inštalátor obsahuje zodpovedajúci balík `chloros_sdk`, takže verzia SDK
vždy zodpovedá nainštalovanému grafickému rozhraniu (GUI), príkazovému riadku (CLI
) alebo backendu. V systéme Windows
ho inštalátor automaticky nainštaluje do systémového Python
u. V systéme Linux
inštalátor `.deb` nainštaluje SDK
pre samotný systém `python3` (do `/usr/lib/python3/dist-packages`), takže `import chloros_sdk` funguje hneď po inštalácii, bez pip a bez pripojenia k sieti. Iba do prostredia virtualenv, ktoré nevidí systémové balíky, je potrebné nainštalovať priložený balík wheel:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Pre hostiteľa, ktorí používajú iba pip (nemajú nainštalovaný balík Chloros
; SDK
v takom prípade riadi backend na inom počítači), je SDK
k dispozícii aj na PyPI:

```bash
pip install chloros-sdk
```

Dokumentáciu nájdete v [API
: Python
SDK
](api-python-sdk.md) a v [Referencii SDK
](reference/sdk-reference.md).

{% hint style="info" %}
**Používatelia Linux
u**: Balík `.deb` nainštaluje CLI
, backend a Python
SDK
. Pre Linux
neexistuje grafické rozhranie (GUI) — všetka interakcia prebieha prostredníctvom CLI
alebo SDK
.
{% endhint %}

***

## Ďalšie zdroje

### Python
SDK


Pre vývojárov a automatizačné pracovné postupy je súčasťou každého inštalátora Chloros
Python
SDK
: inštalátor pre Windows
ho nainštaluje do vášho Python
u a inštalátor `.deb` pre Linux
ho nainštaluje pre systém `python3`. Na počítači bez balíka Chloros
(ktorý riadi backend na inom počítači) ho nainštalujte z PyPI:

```bash
pip install chloros-sdk
```

**Dokumentácia**: [API
: Python
SDK
](api-python-sdk.md)

**Požiadavky**: backend Chloros
, ktorý bude riadiť (inštalátor pre Windows
alebo balík `.deb` pre Linux
ho nainštaluje), a prihlásenie do Chloros
+ v rámci platenej úrovne

***

## Čo je súčasťou balíka

### Inštalátor pre Windows


* ✅ **GUI Chloros** – Plnohodnotné grafické rozhranie
* ✅ **CLI
Chloros
** – Rozhranie príkazového riadku (vyžaduje licenciu Chloros
+)
* ✅ **Backend Chloros** – Spracovateľský engine
* ✅ **Python
SDK
** – nainštaluje sa do vášho Python
u, ak je nainštalovaný (vyžaduje licenciu Chloros
+)
* ✅ **Profily kamier** – vopred nakonfigurované šablóny kamier MAPIR


### Balík .deb pre Linux


* ✅ **Chloros
CLI
** – rozhranie príkazového riadku (vyžaduje licenciu Chloros
+)
* ✅ **Chloros
Backend** – spracovateľský engine
* ✅ **Python
SDK
** – nainštalované pre systém `python3` (vyžaduje licenciu Chloros
+)
* ✅ **Profily kamier** – Predkonfigurované šablóny kamier MAPIR

* ❌ Bez grafického rozhrania — Linux
je bez grafického rozhrania, k dispozícii je iba CLI
/SDK


### Python
SDK
(PyPI, pre zariadenia bez balíka Chloros
)

* ✅ **Chloros
SDK
** – Python
API
(vyžaduje licenciu Chloros
+)

***

## Prechod na Chloros
+

Odomknite pokročilé funkcie predplatným Chloros
+:

* 🚀 **Viacvláknové spracovanie** – Paralelné spracovanie obrázkov
* ⚡ **Akcelerácia GPU (CUDA)** – Využite výkon grafických kariet NVIDIA
* 💻 **Prístup cez CLI** – Automatizácia pomocou nástrojov príkazového riadku
* 🐍 **Python
SDK
** – Programový prístup k API

* 📱 **Viac zariadení** – Používanie na 2–10 a viac zariadeniach (v závislosti od plánu)
* **🐻 Pokročilá metóda debayeringu zohľadňujúca textúru** – vysokokvalitný debayering zohľadňujúci hrany v kombinácii s modelom odšumovania založeným na AI/ML, ktorý odstraňuje takmer všetok šum spôsobený debayeringom.
* 🧮 **Vlastné vzorce** – Vytvárajte vlastné multispektrálne indexy

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Zobraziť plány a ceny Chloros</a></p>***

## Pomoc pri inštalácii

### Riešenie problémov

**Inštalácia zlyhá s chybovou správou:**

* Uistite sa, že máte práva správcu
* Dočasne deaktivujte antivírusový softvér
* Skontrolujte, či spĺňate minimálne systémové požiadavky

**Aplikácia sa nespustí (systém Windows

):**

* Overte, či máte nainštalovaný Windows

10/11 (64-bitový)
* Aktualizujte grafické ovládače
* Skontrolujte Podrobnosti udalostí systému Windows

, kde nájdete podrobnosti o chybe
* Kontaktujte podporu s protokolmi chýb

**CLI

sa nespustí (Linux

):**

* Overte, či je balík `.deb` správne nainštalovaný: `dpkg -l | grep chloros`
* Skontrolujte oprávnenia: `sudo chmod +x /usr/bin/chloros-cli`
* Spustite diagnostiku: `chloros-cli selftest`
* Skontrolujte, či nechýbajú knižnice: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (od verzie 1.3.0 je `/usr/lib/chloros/chloros-backend` malý spúšťací skript; samotný program sa nachádza v `/usr/lib/chloros/backend/`)

**Problémy s aktiváciou licencie:**

* Uistite sa, že máte aktívne pripojenie k internetu
* Overte si prihlasovacie údaje na [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Skontrolujte, či firewall neblokuje Chloros

* Podrobné pokyny nájdete na stránke [Chloros

+ Login](chloros+-login.md)

### Podpora

Potrebujete pomoc s inštaláciou alebo nastavením?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Webová stránka**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentácia**: [Začíname](./)
* ❓ **FAQ**: [Často kladené otázky](faq.md)***

## Aktualizácie softvéru

Chloros

kontroluje dostupnosť aktualizácií, oznamuje, keď je k dispozícii nová verzia, a poskytuje odkaz na túto stránku na stiahnutie — aktualizáciu vykonáte spustením nového podpísaného inštalačného súboru. Vaše nastavenia, projekty a prihlasovacie údaje zostanú zachované aj po aktualizácii. V systéme Linux

a na zariadení Jetson program `chloros-cli update` vyhľadáva novšiu verziu a ponúka stiahnutie a inštaláciu zodpovedajúcej verzie `.deb` (tento príkaz je dostupný iba v systéme Linux

).

***

## Zoznam zmien**Verzia 1.3.0 (najnovšia, 11. november 2026)**— úplný zoznam nájdete v časti**Novinky v Chloros

e 1.3.0** na stránke [Začíname](./).**Verzia 1.2.0**— pozrite si časť**Čo je nové v Chloros

e 1.2.0** na stránke [Začíname](./).

<details>

<summary>Verzia 1.0.5</summary>

**Dátum vydania: 10. februára 2026**

**Nové funkcie*** **Metóda debayeringu s ohľadom na textúru \[Iba Chloros

+] –** Metóda s ohľadom na textúru využíva vysokokvalitný debayering zohľadňujúci hrany v kombinácii s modelom odšumovania založeným na AI/ML, ktorý odstraňuje takmer všetok šum spôsobený debayeringom.
* **Podpora kalibračných terčov T4P*** **Rýchlejšie spracovanie na GPU v programe Chloros

+, lepšie riadenie pamäte**

**Opravy chýb*** Úplne nové používateľské rozhranie (GUI), ktoré by teraz malo fungovať na všetkých počítačoch so systémom Windows

.

</details>

<details>

<summary>Verzia 1.0.4</summary>

**Dátum vydania: 5. január 2026**

**Nové funkcie*** **Prepínanie medzi obrázkom a metadátami**: Do prehliadača súborov bolo pridané prepínanie, ktoré umožňuje zobraziť metadáta vybraného obrázku v tabuľke namiesto mriežky obrázkov
* **Posuvník priblíženia mriežky obrázkov**: Nový posuvník v používateľskom rozhraní na nastavenie veľkosti miniatúr (podporuje aj kombináciu klávesov CTRL + koliesko myši)
* **Tlačidlá na export mriežky obrázkov**: Tlačidlá v hornom riadku na prepínanie miniatúr z formátu JPG na spracované výstupy (Targets, Reflectance, Index, LUT)
* **Karta Mapa**: Nová interaktívna 2D mapa zobrazujúca značky GPS polohy obrázkov
  * Podporuje mapové dlaždice Google Maps a ESRI (automaticky vyberá najlepšiu službu dlaždíc na základe dostupnosti úrovne priblíženia)
  * Náhľad miniatúr pri prechádzaní kurzorom myši nad značkami na mape

**Opravy chýb*** Vylepšená podpora inštalácie programu Chloros na počítačoch s iným jazykom ako angličtina

</details>

<details>

<summary>Verzia 1.0.3</summary>

**Dátum vydania: 20. december 2025**

**Nové funkcie*** Prvé spustenie

**Vylepšenia*** Prvé spustenie

**Opravy chýb*** Prvé spustenie

**Známe problémy*** Prvé spustenie

</details>

***

## Licenčná zmluva**Autorský softvér** – Copyright (c) 2026 MAPIR Inc.

Neoprávnené používanie, distribúcia alebo úprava sú zakázané.

**Bezplatná verzia**: K dispozícii na osobné aj komerčné použitie s obmedzenými funkciami**Chloros+**: Licencia na báze predplatného pre pokročilé funkcie a komerčné nasadenie

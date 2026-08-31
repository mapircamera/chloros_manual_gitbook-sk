---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Stiahnutie

Stiahnite si najnovšiu verziu programu Chloros a začnite so spracovaním multispektrálnych snímok.

### Systémové požiadavky

#### Windows

| Požiadavka          | Minimálna                                              | Odporúčaná                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operačný systém** | Windows 10 (64-bitový)                                  | Windows 11 (64-bitový)                                  |
| **Procesor**        | Intel Core i5 alebo ekvivalent                          | Intel Core i7 alebo lepší                              |
| **Pamäť (RAM)**     | 8 GB                                                  | 16 GB alebo viac                                         |
| **Grafická karta**    | Kompatibilná s DirectX 11                                | Grafický procesor NVIDIA s 4 GB+ VRAM                            |
| **Úložisko**          | 6 GB voľného miesta                                       | SSD s 10 GB+ voľného miesta                            |
| **Rozlíšenie obrazovky**          | 1920x1080                                            | 2560x1440 alebo vyššie                                  |
| **Internet**         | Potrebné na \[voliteľnú] aktiváciu licencie Chloros+ | Potrebné na \[voliteľnú] aktiváciu licencie Chloros+ |

#### Linux amd64 (x86_64)

| Požiadavka       | Minimálna                    | Odporúčaná               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribúcia**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Procesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 alebo lepší   |
| **Pamäť (RAM)**  | 8 GB                        | 16 GB alebo viac              |
| **Grafická karta** | Žiadna (spracovanie na CPU)      | Grafická karta NVIDIA s 4 GB+ VRAM |
| **Úložný priestor**       | 2 GB voľného miesta             | SSD s 10 GB a viac voľného miesta       |
| **Python**        | Python 3.7+ (pre SDK)      | Python 3.10 a viac              |

#### Linux arm64 (NVIDIA Jetson)

| Požiadavka      | Minimálna                      | Odporúčaná                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platforma**     | NVIDIA Jetson s JetPack 6 | Jetson Orin NX 16 GB alebo AGX Orin |
| **Pamäť (RAM)** | 8 GB (zdieľaná medzi GPU a CPU)         | 16 GB+ zdieľaná                    |
| **Úložisko**      | 2 GB voľného miesta               | NVMe SSD s 10 GB+ voľného miesta        |
| **Python**       | Python 3.7+ (pre SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Akcelerácia GPU**: Používatelia Chloros+ s grafickými kartami NVIDIA môžu využiť akceleráciu CUDA na výrazne rýchlejšie spracovanie. Funguje to ako na Windows (stolové grafické karty), tak aj na Linux (stolové grafické karty a NVIDIA Jetson). Používatelia verzie Chloros+ majú k dispozícii aj viacvláknové spracovanie pre maximálnu rýchlosť.
{% endhint %}

***

## Stiahnuť Chloros

### Najnovšia stabilná verzia: Verzia 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Stiahnite si Chloros pre Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Stiahnuť Chloros pre Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Stiahnite si Chloros pre Linux arm64 / Jetson (.deb)</a>

#### Inštalačný program Windows (GUI + CLI + Backend)

* **Typ súboru**: .exe (inštalačný program Windows)**Kroky inštalácie:**

1. Stiahnite si uvedený súbor .exe
2. Dvojitým kliknutím na inštalačný program spustite inštaláciu
3. Postupujte podľa pokynov inštalačného sprievodcu
4. Zvoľte inštalačný adresár (predvolený: `C:\Program Files\MAPIR\Chloros\`)
5. Dokončite inštaláciu a spustite program Chloros alebo Chloros CLI
6. Prihláste sa pomocou svojho [účtu MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (alebo pokračujte s bezplatnou verziou)

{% hint style="success" %}
Inštalačný program automaticky pridá `chloros-cli` do systémovej premennej PATH pre prístup z príkazového riadku.
{% endhint %}

#### Linux amd64 (balík .deb — CLI + backend)

* **Typ súboru**: .deb (balík pre Debian/Ubuntu)
* **Architektúra**: x86_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (balík .deb — CLI + backend)

* **Typ súboru**: .deb (JetPack 6)
* **Architektúra**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Podrobné pokyny na inštaláciu nájdete v [Inštalácia Linux](linux/linux-installation.md) a pokyny špecifické pre Jetson v [Príručke k NVIDIA Jetson](linux/nvidia-jetson-guide.md).

#### Python SDK (všetky platformy)

Každý inštalačný balík obsahuje zodpovedajúci balík `chloros_sdk`, takže verzia SDK vždy zodpovedá nainštalovanému grafickému rozhraniu (GUI)/CLI/backendu. V systéme Windows ho inštalátor automaticky nainštaluje do vášho systému Python; v prípade Linux inštalátor umiestni balík do adresára `/usr/lib/chloros/sdk/` a vypíše inštalačný príkaz:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Pre hostiteľské systémy, ktoré používajú iba pip (bez nainštalovaného balíka Chloros), je SDK k dispozícii aj na PyPI:

```bash
pip install chloros-sdk
```

Pozrite si [API : Python SDK](api-python-sdk.md) a [SDK Referencia](reference/sdk-reference.md) pre dokumentáciu.

{% hint style="info" %}
**Používatelia Linux**: Balík `.deb` inštaluje CLI a backend. Pre Linux neexistuje grafické používateľské rozhranie – všetka interakcia prebieha prostredníctvom CLI alebo SDK.
{% endhint %}

***

## Ďalšie zdroje

### Python SDK

Pre vývojárov a automatizačné pracovné postupy nainštalujte Chloros, Python a SDK:

```bash
pip install chloros-sdk
```

**Dokumentácia**: [API: Python SDK](api-python-sdk.md)**Požiadavky**: Musí byť nainštalovaný Chloros (inštalačný program Windows alebo balík Linux `.deb`), je potrebné prihlásenie s licenciou Chloros+***

## Obsah balíka

### Inštalačný program Windows

* ✅ **Chloros GUI** – Plnohodnotné grafické rozhranie
* ✅ **Chloros CLI** – Rozhranie príkazového riadku (vyžaduje licenciu Chloros+)
* ✅ **Chloros Backend** – Spracovateľský modul
* ✅ **Profily kamier** – vopred nakonfigurované šablóny kamier MAPIR

### Balík Linux .deb

* ✅ **Chloros CLI** – rozhranie príkazového riadku (vyžaduje licenciu Chloros+)
* ✅ **Chloros Backend** – Spracovateľský modul
* ✅ **Profily kamier** – Predkonfigurované šablóny kamier MAPIR
* ❌ Bez grafického rozhrania — Linux je iba bez grafického rozhrania CLI/SDK

### Python SDK (pip, všetky platformy)

* ✅ **Chloros SDK** – Python API (vyžaduje licenciu Chloros+)***

## Prechod na Chloros+

Odblokujte pokročilé funkcie s predplatným Chloros+:

* 🚀 **Viacvláknové spracovanie** – Spracúvajte obrázky paralelne
* ⚡ **Akcelerácia pomocou GPU (CUDA)** – Využite výkon grafických kariet NVIDIA
* 💻 **Prístup k CLI** – Automatizácia pomocou nástrojov príkazového riadku
* 🐍 **Python SDK** – Programový prístup k API
* 📱 **Viac zariadení** – Používanie na 2 až 10 a viac zariadeniach (v závislosti od plánu)
* **🐻 Pokročilá metóda debayeringu zohľadňujúca textúru** – vysokokvalitný debayering zohľadňujúci hrany v kombinácii s modelom odšumovania založeným na AI/ML, ktorý odstraňuje takmer všetok šum spôsobený debayeringom.
* 🧮 **Vlastné vzorce** – Vytvárajte vlastné multispektrálne indexy

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Zobraziť plány a ceny Chloros+</a></p>***

## Pomoc pri inštalácii

### Riešenie problémov

**Inštalácia zlyhá s chybovou správou:**

* Uistite sa, že máte práva správcu
* Dočasne deaktivujte antivírusový softvér
* Skontrolujte, či spĺňate minimálne systémové požiadavky

**Aplikácia sa nespustí (Windows):**

* Overte, či je nainštalovaný Windows 10/11 (64-bitová verzia)
* Aktualizujte grafické ovládače
* Skontrolujte Podrobnosti udalostí v programe Windows, kde nájdete podrobnosti o chybe
* Kontaktujte technickú podporu a poskytnite jej protokoly chýb

**CLI sa nespustí (Linux):**

* Overte, či je balík `.deb` správne nainštalovaný: `dpkg -l | grep chloros`
* Skontrolujte oprávnenia: `sudo chmod +x /usr/bin/chloros-cli`
* Spustite diagnostiku: `chloros-cli selftest`
* Skontrolujte, či nechýbajú knižnice: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problémy s aktiváciou licencie:**

* Uistite sa, že máte aktívne pripojenie k internetu
* Overte si prihlasovacie údaje na [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Skontrolujte, či firewall neblokuje Chloros
* Podrobné pokyny nájdete na stránke [Chloros+ Prihlásenie](chloros+-login.md)

### Podpora

Potrebujete pomoc s inštaláciou alebo nastavením?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Webová stránka**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentácia**: [Začíname](./)
* ❓ **FAQ**: [Často kladené otázky](faq.md)***

## Aktualizácie softvéru

Chloros kontroluje dostupnosť aktualizácií, upozorňuje na novú verziu a odkazuje na túto stránku na stiahnutie — aktualizáciu vykonáte spustením nového podpísaného inštalačného programu. Vaše nastavenia a projekty zostanú po aktualizácii zachované. V Linux a Jetson kontroluje `chloros-cli update`, či je k dispozícii novšia verzia, a ponúka stiahnutie a inštaláciu zodpovedajúceho `.deb` (tento príkaz je k dispozícii iba v Linux).

***

## Zoznam zmien**Verzia 1.2.0 (najnovšia)**— úplný zoznam funkcií nájdete v časti**Novinky vo verzii Chloros 1.2.0** na stránke [Začíname](./).

<details>

<summary>Verzia 1.0.5</summary>

**Dátum vydania: 10. februára 2026**

**Nové funkcie*** **Metóda debayeringu zohľadňujúca textúru \[len pre Chloros+] –** Metóda zohľadňujúca textúru využíva vysokokvalitný debayering zohľadňujúci hrany v kombinácii s modelom odšumovania založeným na umelej inteligencii a strojovom učení, ktorý odstraňuje takmer všetok šum spôsobený debayeringom.
* **Podpora kalibračných terčov T4P*** **Rýchlejšie spracovanie na GPU v Chloros+, lepšie riadenie pamäte**

**Opravy chýb*** Úplne nové používateľské rozhranie (GUI), ktoré by malo teraz fungovať na všetkých počítačoch Windows.

</details>

<details>

<summary>Verzia 1.0.4</summary>

**Dátum vydania: 5. január 2026**

**Nové funkcie*** **Prepínanie medzi obrázkom a metadátami**: Do prehliadača súborov bolo pridané prepínanie, ktoré umožňuje zobraziť metadáta vybraného obrázku v tabuľke namiesto mriežky obrázkov
* **Posuvník priblíženia mriežky obrázkov**: Nový posuvník v používateľskom rozhraní na nastavenie veľkosti miniatúr (podporuje aj kombináciu klávesov CTRL + koliesko myši)
* **Tlačidlá na export mriežky obrázkov**: Tlačidlá v hornom riadku na prepínanie miniatúr z formátu JPG na spracované výstupy (cieľové hodnoty, odrazivosť, index, LUT)
* **Karta Mapa**: Nová interaktívna 2D mapa zobrazujúca značky GPS polohy obrázkov
  * Podporuje Google Maps a mapové dlaždice ESRI (automaticky vyberá najlepšiu službu dlaždíc na základe dostupnosti úrovne priblíženia)
  * Náhľad miniatúry po nabehnutí kurzora myši na značky na mape

**Opravy chýb*** Vylepšená podpora inštalácie Chloros na počítačoch s iným jazykom ako angličtina

</details>

<details>

<summary>Verzia 1.0.3</summary>

**Dátum vydania: 20. december 2025**

**Nové funkcie*** Prvé spustenie

**Vylepšenia*** Prvé spustenie

**Opravy chýb*** Prvé spustenie

**Známe problémy*** Prvé spustenie

</details>***

## Licenčná zmluva**Autorský softvér** – Copyright (c) 2026 MAPIR Inc.

Neoprávnené používanie, distribúcia alebo úprava sú zakázané.

**Bezplatná verzia**: K dispozícii na osobné aj komerčné použitie s obmedzenými funkciami**Chloros+**: Licencia na báze predplatného pre pokročilé funkcie a komerčné nasadenie

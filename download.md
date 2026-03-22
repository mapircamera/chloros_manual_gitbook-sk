---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Stiahnutie

Stiahnite si najnovšiu verziu programu Chloros a začnite so spracovaním multispektrálnych snímok.

### Systémové požiadavky

#### Windows

| Požiadavka          | Minimálne                                              | Odporúčané                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operačný systém** | Windows 10 (64-bitový)                                  | Windows 11 (64-bitový)                                  |
| **Procesor**        | Intel Core i5 alebo ekvivalent                          | Intel Core i7 alebo lepší                              |
| **Pamäť (RAM)**     | 8 GB                                                  | 16 GB alebo viac                                         |
| **Grafická karta**    | Kompatibilná s DirectX 11                                | NVIDIA GPU s 4 GB+ VRAM                            |
| **Úložisko**          | 6 GB voľného miesta                                       | SSD s 10 GB+ voľného miesta                            |
| **Displej**          | 1920x1080                                            | 2560x1440 alebo vyššie                                  |
| **Internet**         | Vyžadované pre \[voliteľnú] aktiváciu licencie Chloros+ | Vyžadované pre \[voliteľnú] aktiváciu licencie Chloros+ |

#### Linux amd64 (x86\_64)

| Požiadavka       | Minimálna                    | Odporúčaná               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribúcia**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Procesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 alebo lepší   |
| **Pamäť (RAM)**  | 8 GB                        | 16 GB alebo viac              |
| **Grafická karta** | Žiadna (spracovanie CPU)      | NVIDIA GPU s 4 GB+ VRAM |
| **Úložisko**       | 2 GB voľného miesta             | SSD s 10 GB+ voľného miesta       |
| **Python**        | Python 3.7+ (pre SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Požiadavka      | Minimálne                      | Odporúčané                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platforma**     | NVIDIA Jetson s JetPack 6 | Jetson Orin NX 16 GB alebo AGX Orin |
| **Pamäť (RAM)** | 8 GB (zdieľaná GPU/CPU)         | 16 GB+ zdieľaná                    |
| **Úložisko**      | 2 GB voľného miesta               | NVMe SSD s 10 GB+ voľného        |
| **Python**       | Python 3.7+ (pre SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Akcelerácia GPU**: Používatelia Chloros+ s grafickými kartami NVIDIA môžu využiť akceleráciu CUDA pre výrazne rýchlejšie spracovanie. Funguje to na Windows (stolové grafické karty) aj na Linux (stolové grafické karty a NVIDIA Jetson). Používatelia Chloros+ získavajú tiež viacvláknové spracovanie pre maximálnu rýchlosť.
{% endhint %}

***

## Stiahnuť Chloros

### Najnovšia stabilná verzia (23. marec 2026): Verzia 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Stiahnite si Chloros pre Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Stiahnite si Chloros pre Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Stiahnite si Chloros pre Linux arm64 / Jetson (.deb)</a>

#### Inštalátor Windows (GUI + CLI + Backend)

* **Typ súboru**: .exe (inštalátor Windows)**Kroky inštalácie:**

1. Stiahnite si vyššie uvedený súbor .exe
2. Dvojitým kliknutím na inštalátor spustite inštaláciu
3. Postupujte podľa pokynov inštalačného sprievodcu
4. Vyberte inštalačný adresár (predvolený: `C:\Program Files\[USER]\Chloros\`)
5. Dokončite inštaláciu a spustite Chloros alebo Chloros CLI
6. Prihláste sa pomocou svojho [účtu MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (alebo pokračujte s bezplatnou verziou)

{% hint style="success" %}
Inštalátor automaticky pridá `chloros-cli` do systémovej premennej PATH pre prístup z príkazového riadku.
{% endhint %}

#### Linux amd64 (balík .deb — CLI + Backend)

* **Typ súboru**: .deb (balík Debian/Ubuntu)
* **Architektúra**: x86\_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (balík .deb — CLI + Backend)

* **Typ súboru**: .deb (JetPack 6)
* **Architektúra**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Podrobné pokyny na nastavenie nájdete v [Inštalácii Linux](linux/linux-installation.md) a pokyny špecifické pre Jetson v [Príručke NVIDIA Jetson](linux/nvidia-jetson-guide.md).

#### Python SDK (všetky platformy)

```bash
pip install chloros-sdk
```

Dokumentáciu nájdete v [API : Python SDK](api-python-sdk.md).

{% hint style="info" %}
**Používatelia Linux**: Balík `.deb` inštaluje CLI a backend. Python a SDK sa inštalujú samostatne prostredníctvom pip. Pre Linux neexistuje grafické rozhranie – všetka interakcia prebieha prostredníctvom CLI alebo SDK.
{% endhint %}

***

## Ďalšie zdroje

### Python SDK

Pre vývojárov a automatizačné pracovné postupy nainštalujte Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentácia**: [API: Python SDK](api-python-sdk.md)**Požiadavky**: Musí byť nainštalovaný Chloros (inštalačný program Windows alebo balík Linux `.deb`), je potrebné prihlásenie s licenciou Chloros+***

## Čo je súčasťou balíka

### Inštalátor Windows

* ✅ **Chloros GUI** – Plnohodnotné grafické rozhranie
* ✅ **Chloros CLI** – rozhranie príkazového riadku (vyžaduje licenciu Chloros+)
* ✅ **Chloros Backend** – spracovateľský engine
* ✅ **Profily kamery** – Predkonfigurované šablóny kamery MAPIR

### Balík Linux .deb

* ✅ **Chloros CLI** – Rozhranie príkazového riadku (vyžaduje licenciu Chloros+)
* ✅ **Chloros Backend** – Spracovateľský modul
* ✅ **Profily kamier** – Predkonfigurované šablóny kamier MAPIR
* ❌ Bez grafického rozhrania — Linux je bez grafického rozhrania, iba CLI/SDK

### Python SDK (pip, všetky platformy)

* ✅ **Chloros SDK** - Python API (vyžaduje licenciu Chloros+)***

## Prejdite na Chloros+

Odomknite pokročilé funkcie s predplatným Chloros+:

* 🚀 **Viacvláknové spracovanie** – Spracúvajte obrázky paralelne
* ⚡ **Akcelerácia GPU (CUDA)** – Využite výkon grafických kariet NVIDIA
* 💻 **Prístup k CLI** – Automatizácia pomocou nástrojov príkazového riadku
* 🐍 **Python SDK** – Programový prístup k API
* 📱 **Viac zariadení** – Používajte na 2 až 10 a viac zariadeniach (v závislosti od plánu)
* **🐻 Pokročilá metóda debayeringu s rozpoznaním textúr** – vysoko kvalitný debayering s rozpoznaním okrajov v kombinácii s modelom odšumovania AI/ML, ktorý odstraňuje takmer všetok šum spôsobený debayeringom.
* 🧮 **Vlastné vzorce** – Vytvárajte vlastné multispektrálne indexy

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Zobraziť plány a ceny Chloros+</a></p>***

## Pomoc pri inštalácii

### Riešenie problémov

**Inštalácia zlyhá s chybovou správou:**

* Uistite sa, že máte práva správcu
* Dočasne deaktivujte antivírusový softvér
* Skontrolujte, či spĺňate minimálne systémové požiadavky

**Aplikácia sa nespustí (Windows):**

* Overte, či je nainštalovaný Windows 10/11 (64-bit)
* Aktualizujte grafické ovládače
* Skontrolujte Windows Prehliadač udalostí, kde nájdete podrobnosti o chybe
* Kontaktujte podporu s protokolmi chýb

**CLI sa nespustí (Linux):**

* Overte, či je balík `.deb` správne nainštalovaný: `dpkg -l | grep chloros`
* Skontrolujte oprávnenia: `sudo chmod +x /usr/bin/chloros-cli`
* Spustite diagnostiku: `chloros-cli selftest`
* Skontrolujte, či nechýbajú knižnice: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problémy s aktiváciou licencie:**

* Uistite sa, že máte aktívne pripojenie k internetu
* Overte prihlasovacie údaje na [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Skontrolujte, či firewall neblokuje Chloros
* Podrobné pokyny nájdete na [Chloros+ Prihlásenie](chloros+-login.md)

### Získanie podpory

Potrebujete pomoc s inštaláciou alebo nastavením?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Webová stránka**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentácia**: [Začíname](./)
* ❓ **FAQ**: [Často kladené otázky](faq.md)***

## Zoznam zmien

<details>

<summary>Verzia 1.1.0 (najnovšia)</summary>

**Dátum vydania: marec 2026**

**Nové funkcie*** **Podpora Linux** — Natívne CLI a SDK pre Linux amd64 (x86\_64) a arm64 (NVIDIA Jetson JetPack 6). Inštalácia prostredníctvom balíkov `.deb`.
* **Podpora NVIDIA Jetson** — Optimalizované spracovanie pre zariadenia Jetson Nano, Orin Nano, Orin NX a AGX Orin.
* **Dynamická adaptácia výpočtov** — Automatická detekcia hardvéru a optimalizácia stratégie spracovania. Chloros sa prispôsobí vášmu hardvéru od Jetson Nano až po pracovnú stanicu s viacerými GPU.
* **4-vláknové spracovateľské potrubie** — Súbežné vlákna detekcie, kalibrácie, spracovania a exportu s dynamickým prideľovaním pamäte GPU.
* **Nové príkazy CLI** — `selftest` (systémová diagnostika) a `update` (správa aktualizácií Linux).
* **Nové príznaky procesu CLI** — `--debayer` (štandardný/s ohľadom na textúru), `--indices` (špecifikovať indexy), `--target` (najskôr vyhľadať cieľovú podzložku pre rýchlejšiu detekciu).
* **Nové položky v ponuke GUI** — Pridať súbory, Pridať priečinok a Spustiť/Zastaviť spracovanie sú teraz dostupné z roletového menu hlavnej ponuky.**Vylepšenia**

* Automatická detekcia backendu naprieč platformami (cesty Windows a Linux)
* Vylepšené SDK `get_status()` so sledovaním priebehu na úrovni vlákien
* Nové výnimky SDK: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Riadenie teploty a adaptívne obmedzovanie výkonu pre NVIDIA Jetson
* Automatické riadenie pamäte s prechodom na dlaždicové spracovanie GPU v prípade nedostatku pamäte (OOM)

</details>

<details>

<summary>Verzia 1.0.5</summary>

**Dátum vydania: 10. február 2026**

**Nové funkcie*** **Metóda debayeringu s ohľadom na textúru \[len Chloros+] -** Metóda s ohľadom na textúru používa vysoko kvalitný debayering s ohľadom na hrany v kombinácii s modelom odšumovania AI/ML, ktorý odstraňuje takmer všetok šum spôsobený debayeringom.
* **Podpora kalibračných cieľov T4P*** **Rýchlejšie spracovanie na GPU v Chloros+, lepšie riadenie pamäte**

**Opravy chýb*** Úplne nové používateľské rozhranie (GUI), malo by teraz fungovať na všetkých počítačoch s Windows.

</details>

<details>

<summary>Verzia 1.0.4</summary>

**Dátum vydania: 5. január 2026**

**Nové funkcie*** **Prepínanie medzi obrázkom a metadátami**: Do prehliadača súborov bolo pridané prepínanie, ktoré umožňuje zobraziť metadáta vybraného obrázku v tabuľke namiesto mriežky obrázkov
* **Posuvník priblíženia mriežky obrázkov**: Nový posuvník v používateľskom rozhraní na nastavenie veľkosti miniatúr (podporuje aj CTRL + koliesko myši)
* **Tlačidlá na export mriežky obrázkov**: Tlačidlá v hornom riadku na prepnutie miniatúr z formátu JPG na spracované exporty (cieľové hodnoty, odrazivosť, index, LUT)
* **Karta Mapa**: Nová interaktívna 2D mapa zobrazujúca značky GPS polohy obrázkov
  * Podporuje mapové dlaždice Google Maps a ESRI (automaticky vyberá najlepšiu službu dlaždíc na základe dostupnosti úrovne priblíženia)
  * Náhľad miniatúr pri prechádzaní kurzorom myši nad značkami na mape

**Opravy chýb*** Vylepšená podpora inštalácie Chloros na počítačoch s iným ako anglickým jazykom

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

**Bezplatná verzia**: K dispozícii na osobné a komerčné použitie s obmedzenými funkciami**Chloros+**: Licencia na báze predplatného pre pokročilé funkcie a komerčné nasadenie

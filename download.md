---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Stiahnuť

Stiahnite si najnovšiu verziu Chloros a začnite s multispektrálnym spracovaním obrazu.

### Systémové požiadavky

| Požiadavka          | Minimálne                                              | Odporúčané                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operačný systém** | Windows 10 (64-bit)                                  | Windows 11 (64-bit)                                  |
| **Procesor**        | Intel Core i5 alebo ekvivalent                          | Intel Core i7 alebo lepší                              |
| **Pamäť (RAM)**     | 8 GB                                                  | 16 GB alebo viac                                         |
| **Grafická karta**    | Kompatibilná s DirectX 11                                | NVIDIA GPU s 4 GB+ VRAM                            |
| **Úložisko**          | 6 GB voľného miesta                                       | SSD s 10 GB+ voľného miesta                            |
| **Displej**          | 1920x1080                                            | 2560x1440 alebo vyšší                                  |
| **Internet**         | Potrebný na \[voliteľnú] aktiváciu licencie Chloros+ | Potrebný na \[voliteľnú] aktiváciu licencie Chloros+ |

{% hint style="info" %}
**GPU akcelerácia**: Používatelia Chloros+ s grafickými kartami NVIDIA môžu využiť CUDA akceleráciu pre výrazne rýchlejšie spracovanie. Používatelia Chloros+ získavajú aj viacvláknové spracovanie pre maximálnu rýchlosť.
{% endhint %}

***

## Stiahnuť Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Stiahnuť Chloros tu</a>

### Najnovšia stabilná verzia

**Chloros Inštalačný program pre Windows*** **Verzia**: 1.0.5
* **Dátum vydania**: 10. február 2026
* **Veľkosť súboru (stiahnutie)**: 1,6 GB
* **Veľkosť súboru (inštalácia)**: 5,7 GB
* **Typ súboru**: .exe (inštalačný program Windows)

#### **Kroky inštalácie:**

1. Stiahnite si súbor `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Dvojitým kliknutím na inštalačný súbor spustite inštaláciu
3. Postupujte podľa pokynov inštalačného sprievodcu
4. Vyberte inštalačný adresár (predvolený: `C:\Program Files\[USER]\Chloros\`)
5. Dokončite inštaláciu a spustite Chloros alebo Chloros CLI
6. Prihláste sa pomocou svojho účtu [MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (alebo pokračujte s bezplatnou verziou).

{% hint style="success" %}
Inštalačný program automaticky pridá `chloros-cli` do systémovej cesty PATH pre prístup z príkazového riadku.
{% endhint %}

***

## Ďalšie zdroje

### Python SDK

Pre vývojárov a automatizačné pracovné postupy nainštalujte Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentácia**: [API: Python SDK](api-python-sdk.md)**Požiadavky**: Chloros Desktop musí byť nainštalovaný, Chloros+ vyžaduje prihlásenie s licenciou.***

## Čo je zahrnuté

Inštalácia Chloros obsahuje:

* ✅ **Chloros** – plnohodnotné grafické rozhranie (GUI)
* ✅ **Chloros CLI** – rozhranie príkazového riadku (vyžaduje licenciu Chloros+)
* ✅ **Chloros SDK** – Python API (vyžaduje licenciu Chloros+)
* ✅ **Profily kamery** – Predkonfigurované šablóny kamery MAPIR***

## Upgrade na Chloros+

Odomknite pokročilé funkcie s predplatným Chloros+:

* 🚀 **Viacvláknové spracovanie** – spracovávajte obrázky paralelne
* ⚡ **Akcelerácia GPU (CUDA)** – využite výkon GPU NVIDIA
* 💻 **CLI Access** – automatizujte pomocou nástrojov príkazového riadku
* 🐍 **Python SDK** – programový prístup API
* 📱 **Viac zariadení** – používajte na 2–10+ zariadeniach (v závislosti od plánu)
* **🐻 Pokročilá metóda debayerovania s ohľadom na textúru** – vysokokvalitné debayerovanie s ohľadom na okraje v kombinácii s modelom odšumovania AI/ML, ktorý odstraňuje takmer všetok šum debayerovania. 
* 🧮 **Vlastné vzorce** – Vytvorte vlastné multispektrálne indexy

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Zobraziť plány a ceny Chloros+</a></p>***

## Pomoc pri inštalácii

### Riešenie problémov

**Inštalácia zlyhá s chybovou správou:**

* Uistite sa, že máte práva správcu
* Dočasne deaktivujte antivírusový softvér
* Skontrolujte, či spĺňate minimálne systémové požiadavky

**Aplikácia sa nespustí:**

* Overte, či je nainštalovaný Windows 10/11 (64-bit)
* Aktualizujte grafické ovládače
* Skontrolujte podrobnosti o chybe v Windows Event Viewer
* Kontaktujte podporu s protokolmi chýb

**Problémy s aktiváciou licencie:**

* Uistite sa, že je aktívne pripojenie k internetu
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

<summary>Verzia 1.0.5</summary>

#### **Dátum vydania**: 10. február 2026**Nové funkcie*** **Metóda Debayer s rozpoznávaním textúr \[Chloros+ iba] -** Texture Aware používa vysokokvalitný debayer s rozpoznávaním okrajov v kombinácii s modelom AI/ML na odstraňovanie šumu, ktorý odstraňuje takmer všetok šum debayeringu.
* **Podpora kalibračných cieľov T4P*** **Rýchlejšie spracovanie GPU Chloros+, lepšie riadenie pamäte**

**Opravy chýb*** Úplne nové používateľské rozhranie (GUI), ktoré by malo teraz fungovať na všetkých počítačoch Windows.

</details>

<details>

<summary>Verzia 1.0.4</summary>

#### **Dátum vydania**: 5. január 2026**Nové funkcie*** **Prepínanie obrázkov/metadát**: Do prehliadača súborov bolo pridané prepínanie, ktoré umožňuje zobraziť metadáta vybraného obrázka v tabuľke namiesto mriežky obrázkov
* **Posúvač zväčšenia mriežky obrázkov**: Nový posúvač v používateľskom rozhraní na úpravu veľkosti miniatúr (podporuje aj CTRL + koliesko myši)
* **Tlačidlá exportu mriežky obrázkov**: Tlačidlá v hornom riadku na prepínanie miniatúr z JPG na spracované exporty (cieľové hodnoty, odrazivosť, index, LUT)
* **Karta Mapa**: Nová interaktívna 2D mapa zobrazujúca značky GPS polohy obrázkov.
  * Podporuje mapové dlaždice Google Maps a ESRI (automaticky vyberá najlepšiu dlaždicovú službu na základe dostupnosti úrovne zväčšenia).
  * Náhľad miniatúr pri prechádzaní myšou nad značkami na mape.

**Oprava chýb*** Vylepšená podpora inštalácie Chloros na počítačoch s iným jazykom ako angličtina.

</details>

<details>

<summary>Verzia 1.0.3</summary>

#### **Dátum vydania**: 20. december 2025**Nové funkcie*** Počiatočné spustenie

**Vylepšenia*** Počiatočné spustenie

**Oprava chýb*** Počiatočné spustenie

**Známe problémy*** Počiatočné spustenie

</details>***

## Licenčná zmluva**Vlastnícky softvér** – Copyright (c) 2026 MAPIR Inc.

Neoprávnené používanie, distribúcia alebo úprava je zakázaná.

**Bezplatná verzia**: K dispozícii na osobné a komerčné použitie s obmedzeniami funkcií.**Chloros+**: Licencia na báze predplatného pre pokročilé funkcie a komerčné nasadenie.

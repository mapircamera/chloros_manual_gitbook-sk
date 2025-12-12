---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Stiahnutie

Stiahnite si najnovšiu verziu Chloros pre Windows a začnite s multispektrálnym spracovaním obrazu.

### Systémové požiadavky

| Požiadavka          | Minimálne                         | Odporúčané                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **Operačný systém** | Windows 10 (64-bit)             | Windows 11 (64-bit)             |
| **Procesor**        | Intel Core i5 alebo ekvivalent     | Intel Core i7 alebo lepší         |
| **Pamäť (RAM)**     | 8 GB                             | 16 GB alebo viac                    |
| **Grafická karta**    | Kompatibilná s DirectX 11           | NVIDIA GPU s 4 GB+ VRAM       |
| **Úložisko**          | 2 GB voľného miesta                  | SSD s 10 GB+ voľného miesta       |
| **Displej**          | 1920x1080                       | 2560x1440 alebo vyšší             |
| **Internet**         | Potrebný na aktiváciu licencie | Potrebný na aktiváciu licencie |

{% hint style=&quot;info&quot; %}
**GPU akcelerácia**: Chloros+ používatelia s GPU NVIDIA (4 GB+ VRAM) môžu využiť CUDA akceleráciu pre výrazne rýchlejšie spracovanie.
{% endhint %}

***

## Stiahnuť Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Stiahnuť Chloros tu</a>

### Najnovšia stabilná verzia

**Chloros Inštalátor pre Windows**

* **Verzia**: 1.0.3
* **Dátum vydania**: december ?, 2025
* **Veľkosť súboru**: 1,6 GB
* **Typ súboru**: .exe (inštalátor Windows)

#### **Kroky inštalácie:**

1. Stiahnite si súbor `CHLOROS INSTALLER - CURRENT VERSION.exe`.
2. Dvojitým kliknutím na inštalačný program spustite inštaláciu.
3. Postupujte podľa pokynov inštalačného sprievodcu.
4. Vyberte inštalačný adresár (predvolený: `C:\Program Files\Chloros\`).
5. Dokončite inštaláciu a spustite Chloros.
6. Prihláste sa pomocou svojho účtu MAPIR Cloud Chloros+ (alebo pokračujte s bezplatnou verziou).

{% hint style=&quot;success&quot; %}
Inštalačný program automaticky pridá `chloros-cli` do systémovej cesty PATH pre prístup z príkazového riadku.
{% endhint %}

***

## Ďalšie zdroje

### Python SDK

Pre vývojárov a automatizačné pracovné postupy nainštalujte Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentácia**: [API: Python SDK](api-python-sdk.md)

**Požiadavky**: Chloros Desktop musí byť nainštalovaný, Chloros+ licencia je povinná.

***

## Čo je zahrnuté

Inštalácia Chloros obsahuje:

* ✅ **Chloros Desktop GUI** – plnohodnotné grafické rozhranie
* ✅ **Chloros (prehliadač)** – webové rozhranie pre systémy s nižšou konfiguráciou
* ✅ **Chloros CLI** – rozhranie príkazového riadku (vyžaduje licenciu Chloros+)
* ✅ **Backend Engine** – potrubie na spracovanie obrazu
* ✅ **Profily kamery** - Predkonfigurované šablóny kamery MAPIR

***

## Upgrade na Chloros+

Odomknite pokročilé funkcie s predplatným Chloros+:

* 🚀 **Viacvláknové spracovanie** – spracovávajte obrázky paralelne
* ⚡ **Akcelerácia GPU (CUDA)** – Využite výkon GPU NVIDIA
* 💻 **CLI Access** – Automatizácia pomocou nástrojov príkazového riadku
* 🐍 **Python SDK** – Programový prístup API
* 📱 **Viac zariadení** – Používajte na 2–10+ zariadeniach (v závislosti od plánu)
* 🧮 **Vlastné vzorce** – Vytvárajte vlastné multispektrálne indexy

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Zobraziť Chloros+ Plány a ceny</a></p>***

## Pomoc s inštaláciou

### Riešenie problémov

**Inštalácia zlyhá s chybovou správou:**

* Uistite sa, že máte práva správcu
* Dočasne deaktivujte antivírusový softvér
* Skontrolujte, či spĺňate minimálne systémové požiadavky

**Aplikácia sa nespustí:**

* Skúste verziu Chloros (prehliadač)
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
* ❓ **FAQ**: [Často kladené otázky](faq.md)

***

## Zoznam zmien

<details>

<summary>Verzia 1.0.3</summary>

### **Dátum vydania**: december ?, 2025

#### Nové funkcie

* Počiatočné spustenie

#### Vylepšenia

* Počiatočné spustenie

#### Opravy chýb

* Počiatočné spustenie

#### Známé problémy

* Počiatočné spustenie

</details>***

## Licenčná zmluva

**Vlastnícky softvér** – Copyright (c) 2025 MAPIR Inc.

Neoprávnené používanie, distribúcia alebo úprava je zakázaná.

**Bezplatná verzia**: K dispozícii na osobné a komerčné použitie s obmedzeniami funkcií.

**Chloros+**: Licencia na báze predplatného pre pokročilé funkcie a komerčné nasadenie.

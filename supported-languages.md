# Podporované jazyky

Chloros ponúka plnú podporu rozhrania v **38 jazykoch po celom svete**, vďaka čomu je dostupný pre používateľov na celom svete. Jazyky môžete okamžite prepínať vo všetkých rozhraniach: Desktop, Browser, CLI a Python SDK.

Chloros podporuje nasledujúce jazyky:

| # | Jazyk | Názov v pôvodnom jazyku | Kód CLI |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Angličtina | English | `en` |
| 2 | 🇪🇸 Španielčina | Español | `es` |
| 3 | 🇵🇹 Portugalčina | Português | `pt` |
| 4 | 🇫🇷 Francúzština | Français | `fr` |
| 5 | 🇩🇪 Nemčina | Deutsch | `de` |
| 6 | 🇮🇹 Taliančina | Italiano | `it` |
| 7 | 🇯🇵 Japončina | 日本語 | `ja` |
| 8 | 🇰🇷 Kórejčina | 한국어 | `ko` |
| 9 | 🇨🇳 Čínština (zjednodušená) | 简体中文 | `zh` |
| 10 | 🇹🇼 Čínština (tradičná) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Ruština | Русский | `ru` |
| 12 | 🇳🇱 Holandčina | Nederlands | `nl` |
| 13 | 🇸🇦 Arabčina | العربية | `ar` |
| 14 | 🇵🇱 Poľština | Polski | `pl` |
| 15 | 🇹🇷 Turečtina | Türkçe | `tr` |
| 16 | 🇮🇳 Hindčina | हिंदी | `hi` |
| 17 | 🇮🇩 Indonézština | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnamčina | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Thajčina | ไทย | `th` |
| 20 | 🇸🇪 Švédčina | Svenska | `sv` |
| 21 | 🇩🇰 Dánčina | Dansk | `da` |
| 22 | 🇳🇴 Nórčina | Norsk | `no` |
| 23 | 🇫🇮 Fínčina | Suomi | `fi` |
| 24 | 🇬🇷 Gréčtina | Ελληνικά | `el` |
| 25 | 🇨🇿 Čeština | Čeština | `cs` |
| 26 | 🇭🇺 Maďarčina | Magyar | `hu` |
| 27 | 🇷🇴 Rumunčina | Română | `ro` |
| 28 | 🇺🇦 Ukrajinčina | Українська | `uk` |
| 29 | 🇧🇷 brazílska portugalčina | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 kantónčina | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malajčina | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slovenský jazyk | Slovenčina | `sk` |
| 33 | 🇧🇬 Bulharčina | Български | `bg` |
| 34 | 🇭🇷 Chorvátčina | Hrvatski | `hr` |
| 35 | 🇱🇹 Litovčina | Lietuvių | `lt` |
| 36 | 🇱🇻 Lotyština | Latviešu | `lv` |
| 37 | 🇪🇪 Estónčina | Eesti | `et` |
| 38 | 🇸🇮 Slovinčina | Slovenščina | `sl` |

## Ako zmeniť jazyk

### V Chloros Desktop/Prehliadač

1. Otvorte nastavenia aplikácie
2. Prejdite do ponuky výberu jazyka
3. Zo zoznamu vyberte preferovaný jazyk
4. Rozhranie sa okamžite aktualizuje

### V Chloros CLI

Použite príkaz `language` na zobrazenie alebo zmenu jazyka rozhrania CLI:

```bash
# View current language
chloros-cli language

# Change to Spanish
chloros-cli language es

# Change to Chinese (Simplified)
chloros-cli language zh

# Change to Brazilian Portuguese
chloros-cli language pt-BR

# List all available languages
chloros-cli language --list
```

Ďalšie podrobnosti nájdete v [dokumentácii CLI](CLI.md).

### V Chloros Python SDK

Pri inicializácii SDK nastavte parameter jazyka, aby ste dostávali správy a výstupy vo vašom preferovanom jazyku.

## Pokrytie

Všetkých 38 jazykov je plne podporovaných v:

* **Chloros Desktop** – Kompletný preklad grafického používateľského rozhrania
* **Chloros Browser** – Webové rozhranie vo všetkých jazykoch
* **Chloros CLI** – Rozhranie príkazového riadku a výstupné správy
* **Chloros Python SDK** – Správy a dokumentácia API

Podpora jazykov zaručuje, že používatelia po celom svete môžu efektívne pracovať vo svojom rodnom jazyku bez bariér.

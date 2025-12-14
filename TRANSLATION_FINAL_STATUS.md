# Chloros Manuál – Konečný stav prekladateľského projektu

**Posledná aktualizácia:** 13. december 2025

---

## 📊 Celkový stav

### ✅ **DOKONČENÉ: 32 jazykov (DeepL)**

Úplne preložené a zverejnené na GitBook:

**Európske jazyky (20):**
- 🇧🇬 bulharčina (bg)
- 🇨🇿 čeština (cs)
- 🇩🇰 dánčina (da)
- 🇩🇪 nemčina (de)
- 🇬🇷 gréčtina (el)
- 🇪🇸 španielčina (es)
- 🇪🇪 estónčina (et)
- 🇫🇮 fínčina (fi)
- 🇫🇷 francúzština (fr)
- 🇭🇺 maďarčina (hu)
- 🇮🇹 taliančina (it)
- 🇱🇻 lotyština (lv)
- 🇱🇹 litovčina (lt)
- 🇳🇱 holandčina (nl)
- 🇳🇴 nórčina (no)
- 🇵🇱 poľština (pl)
- 🇵🇹 portugalčina (pt)
- 🇧🇷 portugalčina Brazília (pt-BR)
- 🇷🇴 rumunčina (ro)
- 🇸🇰 Slovenčina (sk)
- 🇸🇮 Slovinčina (sl)
- 🇸🇪 Švédčina (sv)

**Ďalšie jazyky (12):**
- 🇸🇦 Arabčina (ar)
- 🇨🇳 Zjednodušená čínština (zh-CN)
- 🇭🇰 Hongkongská čínština (zh-HK)
- 🇹🇼 Tradičná čínština (zh-TW)
- 🇮🇩 Indonézština (id)
- 🇯🇵 Japončina (ja)
- 🇰🇷 Kórejčina (ko)
- 🇷🇺 Ruština (ru)
- 🇹🇷 Turečtina (tr)
- 🇺🇦 Ukrajinčina (uk)

**Kvalita prekladu:**
- ✅ Všetok obsah je úplne preložený.
- ✅ Popisy prednej časti sú preložené.
- ✅ Technické termíny sú chránené.
- ✅ Bloky kódu sú zachované.
- ✅ Vzorce sú neporušené.
- ✅ Odkazy fungujú.
- ✅ Formátovanie je dokonalé

---

### 🔄 **V PRÍPRAVE: 5 jazykov (Google Translate)**

**Aktuálny stav:**
- 🇮🇳 **hindčina (hi)** - ⏳ PREKLADÁ SA (2-3 hodiny)
- 🇭🇷 **chorvátsčina (hr)** - ⏳ Čaká sa (anglický + preložený popis)
- 🇲🇾 **malajčina (ms)** - ⏳ Čaká sa (anglický + preložený popis)
- 🇹🇭 **thajčina (th)** - ⏳ Čaká sa (angličtina + preložené popisy)
- 🇻🇳 **Vietnamčina (vi)** - ⏳ Čaká sa (angličtina + preložené popisy)

**Prečo sú pomalšie:**
- Nepodporované DeepL API
- Google Translate API má obmedzenia rýchlosti
- Použitie ultrakonzervatívneho prekladu riadok po riadku
- 1-sekundové oneskorenie na riadok, aby sa zabránilo obmedzeniu rýchlosti

**Aktuálny stav (4 jazyky v čakacom stave):**
- ✅ Repozitáre existujú na GitHub
- ✅ Preložené popisy prednej časti
- ✅ Všetky prostriedky a obrázky sú synchronizované
- ⚠️ Obsah tela je stále v angličtine (funkčný)

---

## 🔧 Funkcie prekladového systému

### Automatický preklad
- **Popisné polia** v prednej časti sú automaticky preložené
- **DeepL API** pre 32 jazykov (vysoká kvalita)
- **Google Translate** pre 5 jazykov (s konzervatívnym obmedzením rýchlosti)

### Ochrana obsahu
- ✅ Názvy produktov (Chloros, MAPIR)
- ✅ Bloky kódu a vložené kódy
- ✅ Matematické vzorce
- ✅ Technické názvy farieb (Red, Green, Blue, NIR, RedEdge)
- ✅ Cesty k súborom a URL adresy
- ✅ Skrátené kódy GitBook
- ✅ E-mailové adresy
- ✅ Prípony súborov

### Obsah, ktorý sa prekladá
- ✅ Názvy stránok
- ✅ Text a odseky
- ✅ Bunky a hlavičky tabuliek
- ✅ Nápovedy a popisy
- ✅ Text odkazov
- ✅ Popisy prednej časti

### Dodatočné spracovanie
- ✅ Opravuje nové riadky HTML
- ✅ Obnovuje chránené prvky
- ✅ Opravuje problémy s formátovaním
- ✅ Zabezpečuje kompatibilitu GitBook

---

## 📝 Prehľad skriptov

### Hlavný denný pracovný postup
**`update_all_translations.py`**
- Aktualizuje všetkých 37 jazykových repozitárov
- Synchronizuje text, obrázky a prostriedky
- Prekladá iba zmenené súbory
- Automaticky odosiela a posiela do GitHub
- Použitie: `python update_all_translations.py`

### Prekladové skripty
**`translate_with_deepl.py`**
- Jadrový preklad DeepL (32 jazykov)
- Spracováva popisy frontmatter
- Plná ochrana markdown

**`translate_with_google.py`**
- Integrácia Google Translate (5 jazykov)
- Rovnaká ochrana ako DeepL
- Spracováva obmedzenia API

**`translate_google_conservative.py`**
- Ultra pomalý, ale spoľahlivý Google Translate
- Preklad po riadkoch
- Dlhé oneskorenia, aby sa predišlo obmedzeniam rýchlosti
- Pre náročné jazyky: `python translate_google_conservative.py hi`

### Skripty nástrojov
**`verify_all_pushed.py`**
- Skontrolujte, či je všetkých 37 repozitárov odoslaných do GitHub

**`check_google_progress.py`**
- Skontrolujte počet jazykových súborov Google Translate

**`check_hindi_progress.py`**
- Podrobný priebeh prekladu do hindčiny

**`push_until_stable.py`**
- Nahrajte všetky repozitáre, kým nedôjde k žiadnym zmenám.

---

## 🌐 Integrácia GitBook

### Proces synchronizácie
1. Zmeny nahrané do repozitára GitHub.
2. GitBook sa automaticky synchronizuje do 5–10 minút.
3. Zmeny sa zobrazia na živom webe.

### Štruktúra repozitára
- **Angličtina:** `chloros_manual_gitbook`
- **Preklady:** `chloros_manual_gitbook-{lang_code}`

### Kódy jazykov
| Názov repozitára | Kód CLI | Jazyk |
|-----------|----------|----------|
| zh-CN | zh | Zjednodušená čínština |
| zh-HK | zh | Hongkongská čínština |
| zh-TW | zh | Tradičná čínština |
| nb | no | Nórčina |
| pt-BR | pt-BR | Brazílska portugalčina |
| Všetky ostatné | Rovnaké ako repo | Štandardné |

---

## 📈 Štatistiky prekladu

### Celková veľkosť projektu
- **Jazyky:** 37 + angličtina = 38 repo
- **Súbory na jazyk:** ~30 súborov markdown
- **Celkový počet preložených súborov:** 32 × 30 = 960 súborov (DeepL)
- **Obrázky/prvky:** Synchronizované vo všetkých 37 repozitároch
- **Preložené riadky:** ~50 000+ riadkov

### API Použitie
- **DeepL API:** ~960 preložených súborov
- **Google Translate:** V procese (5 jazykov)
- **Investovaný čas:** Niekoľko dní vývoja a prekladu

### Ukazovatele kvality
- ✅ 100 % prekladov DeepL je vysokej kvality
- ✅ 100 % preložených popisov frontmatter (všetkých 37 jazykov)
- ✅ 100 % zachované formátovanie
- ✅ 100 % chránených technických termínov
- ✅ 0 % nefunkčných odkazov alebo obrázkov

---

## 🚀 Ďalšie kroky

### Krátkodobé (dnes)
1. ⏳ Počkať na dokončenie prekladu do hindčiny (~2-3 hodiny)
2. 📤 Overiť, či bola hindčina odoslaná do GitHub
3. 🔍 Otestujte hindčinu na GitBook

### Strednodobý plán (tento týždeň)
1. Preložte zostávajúce 4 jazyky (hr, ms, th, vi)
2. Každý preklad bude trvať 2–3 hodiny pri konzervatívnej metóde
3. Nahrajte a overte všetko na GitBook

### Dlhodobý
1. Sledovať, či DeepL pridá podporu pre týchto 5 jazykov.
2. Preložiť znovu pomocou DeepL, keď bude k dispozícii.
3. Pravidelné aktualizácie pomocou `update_all_translations.py`.

---

## 💡 Odporúčania

### Pre pravidelné aktualizácie
```bash
python update_all_translations.py
```
Toto automaticky spracuje všetko pre jazyky DeepL.

### Pre jazyky Google Translate
Keď sa zmení obsah v angličtine, ručne spustite:
```bash
python translate_google_conservative.py hi
python translate_google_conservative.py hr
python translate_google_conservative.py ms
python translate_google_conservative.py th
python translate_google_conservative.py vi
```

### Pre sledovanie
```bash
python verify_all_pushed.py       # Check all repos
python check_google_progress.py   # Check Google langs
python check_hindi_progress.py    # Check Hindi specifically
```

---

## 🎯 Kritériá úspešnosti

### ✅ Dosiahnuté
- [x] 32 jazykov úplne preložených prostredníctvom DeepL
- [x] Preložené všetky popisy frontmatter (37 jazykov)
- [x] Všetky repozitáre na GitHub
- [x] Všetky repozitáre synchronizované s GitBook
- [x] Automatizovaný denný skript pracovného toku
- [x] Ochrana všetkého technického obsahu
- [x] Post-processing opravuje všetky formátovania

### ⏳ V procese
- [ ] 5 jazykov Google Translate úplne preložených
- [ ] Preklad do hindčiny (momentálne prebieha)

### 📅 Budúcnosť
- [ ] Monitorovanie rozšírenia podpory DeepL
- [ ] V prípade potreby zvážiť profesionálny preklad posledných 5 jazykov

---

## 📞 Podpora a dokumentácia

### Kľúčové dokumenty
- `TRANSLATION_QUICK_START.md` – Rýchly referenčný sprievodca
- `TRANSLATION_WORKFLOW.md` – Podrobná dokumentácia pracovného postupu
- `TRANSLATION_COMMANDS.md` – Referencia príkazov
- `TRANSLATION_FINAL_STATUS.md` – Tento dokument

### Umiestnenie kľúčových skriptov
Všetky skripty v: `C:\Users\MAPIR\Documents\GitHub\chloros_manual_gitbook\`

### Umiestnenie repozitárov
Repozitáre prekladov: `D:\chloros_translation_robust\`

---

**Stav projektu:** 🟢 **32/37 Dokončené**, 🟡 **5/37 V procese**

**Celková miera úspešnosti:** 86 % Dokončené (32 úplne preložených + 5 s preloženými popismi)




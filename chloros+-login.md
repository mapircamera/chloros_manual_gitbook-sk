# Prihlásenie do Chloros+

## Prihlásenie cez grafické rozhranie

Bočné menu používateľa <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> vám umožňuje prihlásiť sa do vášho účtu Chloros+ a odomknúť ďalšie funkcie.

**Na každom počítači sa stačí prihlásiť len raz.** GUI, CLI a Python SDK zdieľajú rovnakú uloženú reláciu — prihlásenie cez grafické rozhranie na počítači aktivuje aj CLI a SDK na danom počítači (a naopak cez `chloros-cli login`).

Po prihlásení sa zobrazia podrobnosti vášho účtu:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Úrovne plánov

| Plán | `plan_id` | Typ |
| --- | --- | --- |
| Iron | `0` | Bezplatný |
| Copper | `1` | Platený (Chloros+) |
| Bronze | `2` | Platený (Chloros+) |
| Silver | `3` | Platený (Chloros+) |
| Zlato | `4` | Platené (Chloros+) |

V časti [plány a ceny](https://cloud.mapir.camera/pricing) nájdete informácie o tom, čo obsahuje každá platená úroveň.

### Prístup k CLI/SDK vyžaduje platenú úroveň

Prístup k CLI a Python SDK vyžaduje **akúkoľvek platenú úroveň Chloros+ (Copper alebo vyššiu)**. Toto sa vynucuje**na strane servera** — každá požiadavka CLI/SDK musí obsahovať aktívnu reláciu aj platený plán:

| Stav HTTP | `error_code` | Význam | Riešenie |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Nie ste prihlásený na tomto zariadení | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Prihlásený, ale úroveň plánu je príliš nízka (bezplatná úroveň Iron) | Prejdite na akýkoľvek platený plán Chloros+ |

`chloros-cli status` zostáva dostupný aj v bezplatnej úrovni, takže si vždy môžete pozrieť svoj aktuálny plán a dôvod, prečo bol prístup zamietnutý.

### Obmedzenia pripojeného hardvéru podľa plánu

Každý plán obmedzuje počet kamier LATTICE a svetelných senzorov DAQ, ktoré je možné naraz pripojiť naživo:

| Balík | Kamery LATTICE | Svetelné senzory DAQ |
| --- | --- | --- |
| Iron (bezplatný / neprihlásený) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Prihlásenie cez CLI



Prihláste sa pomocou svojich prihlasovacích údajov pre Chloros+, aby ste povolili spracovanie cez CLI. V systéme Linux (bez grafického rozhrania) je to jediný spôsob, ako aktivovať vašu licenciu.

**Syntax:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Používatelia SDK**: Python SDK tiež poskytuje programovú metódu `logout()` na vymazanie prihlasovacích údajov uložených v cache. Podrobnosti nájdete v [Referencii SDK](reference/sdk-reference.md).
{% endhint %}

**Príklad:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Špeciálne znaky**: Heslá obsahujúce znaky ako `$`, `!` alebo medzery uzavrite do jednoduchých úvodzoviek.
{% endhint %}

**Výstup:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Ukladanie prihlasovacích údajov

Uložené prihlasovacie údaje a konfigurácia sa na **všetkých platformách** ukladajú do priečinka `.chloros` vo vašom domovskom adresári:

| Platforma | Cesta k vyrovnávacej pamäti prihlasovacích údajov |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

Inštaláciou novej verzie na existujúcu verziu sa zachovajú vaše prihlasovacie údaje v systéme Windows (inštalačný program) a v systéme Linux (zložka `.deb`, vrátane zložky `chloros-cli update`). Odinštalovanie programu Chloros — alebo odstránenie balíka pre Linux — vymaže prihlásenie uložené v cache, takže sa po novej inštalácii budete musieť prihlásiť znova.

### Vypršanie platnosti plánu a offline tolerančné obdobie

Dátum vypršania platnosti plánu v grafickom rozhraní udáva, kedy vaša licencia prestane platiť. V prípade opakovaných mesačných predplatných je dátum vypršania platnosti na konci mesiaca; v prípade ročných predplatných je to rok od začiatku predplatného.

Program Chloros overuje vašu licenciu online, ale práca v režime offline je podporovaná počas tolerančného obdobia:

* Úspešné overenia na serveri sa ukladajú do vyrovnávacej pamäte na **5 minút**, takže pri bežnom používaní dochádza k veľmi malému počtu volaní licencie.
* Cache podpísanej licencie viazanej na konkrétny počítač pokrýva dlhšie obdobia bez pripojenia: **30 dní pre mesačné plány**a**až do dátumu vypršania platnosti vášho predplatného (najviac 365 dní) pre ročné plány**.
* Po uplynutí tolerančného obdobia sa plán zmení na bezplatnú úroveň Iron, kým sa zariadenie aspoň raz nepripojí k licenčnému serveru; prístup sa obnoví pri najbližšej úspešnej kontrole.

### Obmedzenie počtu zariadení

Každý plán Chloros+ ponúka iný počet registrovaných zariadení. Každé zariadenie, na ktorom sa prihlásite pomocou účtu Chloros+, sa započítava do počtu vašich registrovaných zariadení. Zariadenie môžete premenovať alebo odstrániť na stránke vášho účtu MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Plán Chloros</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Podporované zariadenia</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

Presný počet zariadení, ktoré môže váš účet obsahovať, je uvedený na stránke vášho účtu MAPIR Cloud. Odhlásenie zo zariadenia spoľahlivo uvoľní jeho miesto a zariadenie, ktoré je už zaregistrované, sa môže kedykoľvek znovu prihlásiť, aj keď účet dosiahol limit počtu zariadení.

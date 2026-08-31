# Chloros+ Prihlásenie

## Prihlásenie cez grafické rozhranie

Bočné menu na stránke <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> vám umožňuje prihlásiť sa do vášho účtu Chloros+ a odomknúť ďalšie funkcie.

**Na každom počítači sa stačí prihlásiť len raz.** Grafické používateľské rozhranie, CLI a Python SDK zdieľajú rovnakú reláciu v cache — prihlásenie cez grafické rozhranie na pracovnej ploche aktivuje aj CLI a SDK na danom zariadení (a naopak cez `chloros-cli login`).

Po prihlásení sa zobrazia podrobnosti vášho účtu:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.2.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->
## Úrovne plánov

| Plán | `plan_id` | Typ |
| --- | --- | --- |
| Iron | `0` | Bezplatný |
| Copper | `1` | Platený (Chloros+) |
| Bronze | `2` | Platený (Chloros+) |
| Striebro | `3` | Platené (Chloros+) |
| Zlatá | `4` | Platená (Chloros+) |

V časti [plány a ceny](https://cloud.mapir.camera/pricing) nájdete informácie o tom, čo obsahuje každá platená úroveň.

### Prístup k CLI / SDK vyžaduje platenú úroveň

Prístup k CLI a Python SDK vyžaduje **akúkoľvek platenú úroveň Chloros+ (Copper alebo vyššiu)**. Toto sa vynucuje**na strane servera** — každá požiadavka typu CLI/SDK musí obsahovať aktívnu reláciu aj platený balík:

| Stav HTTP | `error_code` | Význam | Riešenie |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Nie ste prihlásený na tomto zariadení | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Prihlásený, ale úroveň plánu je príliš nízka (bezplatná úroveň Iron) | Prejdite na akýkoľvek platený plán Chloros+ |

`chloros-cli status` zostáva dostupný v bezplatnej úrovni, takže si vždy môžete pozrieť svoj aktuálny plán a dôvod, prečo bol prístup zamietnutý.

### Obmedzenia pripojeného hardvéru podľa plánu

Každý plán obmedzuje počet kamier LATTICE a svetelných senzorov DAQ, ktoré je možné naraz pripojiť v režime živého prenosu:

| Balík | Kamery LATTICE | Svetelné senzory DAQ |
| --- | --- | --- |
| Iron (bezplatný / neprihlásený) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Prihlásenie do CLI

Prihláste sa pomocou svojich prihlasovacích údajov pre Chloros+, aby ste povolili spracovanie v CLI. V systéme Linux (bez grafického rozhrania) je to jediný spôsob, ako aktivovať vašu licenciu.

**Syntax:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Používatelia SDK**: Python SDK poskytuje aj programovú metódu `logout()` na vymazanie prihlasovacích údajov z vyrovnávacej pamäte. Podrobnosti nájdete v [Referencii k SDK](reference/sdk-reference.md).
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
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.2.0"; capture a successful login with the current output format. -->
### Ukladanie prihlasovacích údajov

Uložené prihlasovacie údaje a konfigurácia sa nachádzajú v priečinku `.chloros` vo vašom domovskom adresári na **všetkých platformách**:

| Platforma | Cesta k vyrovnávacej pamäti prihlasovacích údajov |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

### Vypršanie platnosti plánu a offline tolerančné obdobie

Dátum vypršania platnosti plánu v grafickom rozhraní (GUI) udáva, kedy vaša licencia prestane platiť. V prípade opakovaných mesačných predplatných je dátum vypršania platnosti na konci mesiaca; v prípade ročných predplatných je to rok od začiatku predplatného.

Chloros overuje vašu licenciu online, ale práca v režime offline je podporovaná počas tolerančného obdobia:

* Úspešné overenia na serveri sa ukladajú do vyrovnávacej pamäte na **5 minút**, takže pri bežnom používaní dochádza k veľmi malému počtu volaní licencie.
* Cache podpísanej licencie viazanej na konkrétne zariadenie pokrýva dlhšie obdobia offline: **30 dní pre mesačné plány**a**až do dátumu vypršania platnosti vášho predplatného (najviac 365 dní) pre ročné plány**.
* Po uplynutí tolerančného obdobia sa plán prepne na bezplatnú úroveň Iron, kým sa zariadenie aspoň raz nepripojí k licenčnému serveru; prístup sa obnoví pri najbližšej úspešnej kontrole.

### Limit zariadení

Každý plán Chloros+ ponúka iný počet registrovaných zariadení. Každé zariadenie, na ktorom sa prihlásite pomocou účtu Chloros+, sa započítava do počtu vašich registrovaných zariadení. Zariadenie môžete premenovať alebo odstrániť na stránke vášho účtu MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Tarifa Chloros+</th><th align="center">MEĎ</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Podporované zariadenia</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>Presný počet zariadení, ktoré môže váš účet obsahovať, je uvedený na stránke vášho účtu MAPIR Cloud. Odhlásením zo zariadenia sa jeho miesto spoľahlivo uvoľní a zariadenie, ktoré je už zaregistrované, sa môže kedykoľvek znovu prihlásiť, aj keď účet dosiahol limit počtu zariadení.

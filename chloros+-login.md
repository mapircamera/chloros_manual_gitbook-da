# Chloros+-login

## Login via GUI

Via brugerens sidebjælkemenu <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> kan du logge ind på din Chloros+-konto og få adgang til yderligere funktioner.

**Du behøver kun at logge ind én gang pr. computer.** GUI, CLI og Python-SDK deler den samme cachelagrede session — når du logger ind via desktop-GUI&#x27;en, aktiveres også CLI og SDK på den pågældende maskine (og omvendt via `chloros-cli login`).

Når du er logget ind, vises dine kontooplysninger:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Abonnementsniveauer

| Abonnement | `plan_id` | Type |
| --- | --- | --- |
| Iron | `0` | Gratis |
| Copper | `1` | Betalt (Chloros

+) |
| Bronze | `2` | Betalt (Chloros

+) |
| Silver | `3` | Betalt (Chloros

+) |
| Guld | `4` | Betalt (Chloros

+) |

Se [abonnementer og priser](https://cloud.mapir.camera/pricing) for at se, hvad de enkelte betalte abonnementer indeholder.

### Adgang til CLI

/SDK

kræver et betalt abonnement

Adgang til CLI

og Python

SDK

kræver **et hvilket som helst betalt Chloros

+-abonnement (Copper eller højere)**. Dette håndhæves**på serversiden** — hver CLI

/SDK

-anmodning skal indeholde både en aktiv session og et betalt abonnement:

| HTTP

-status | `error_code` | Betydning | Løsning |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Ikke logget ind på denne maskine | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Logget ind, men abonnementsniveauet er for lavt (gratis Iron-niveau) | Opgrader til et hvilket som helst betalt Chloros

+-abonnement |

`chloros-cli status` er stadig tilgængeligt på det gratis niveau, så du kan altid se dit nuværende abonnement og årsagen til, at adgangen er nægtet.

### Begrænsninger for tilsluttet hardware pr. abonnement

Hvert abonnement har en begrænsning for, hvor mange LATTICE-kameraer og DAQ-lyssensorer der kan være tilsluttet live på samme tid:

| Abonnementsniveau | LATTICE-kameraer | DAQ-lyssensorer |
| --- | --- | --- |
| Iron (gratis / ikke logget ind) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## CLI

-login

Log ind med dine Chloros

+-loginoplysninger for at aktivere CLI

-behandling. På Linux

(uden GUI) er dette den eneste måde at aktivere din licens på.

**Syntaks:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK

-brugere**: Python

-SDK

&#x27;en indeholder også en programmatisk `logout()`-metode til at rydde cachelagrede loginoplysninger. Se [SDK

-referencen](reference/sdk-reference.md) for yderligere oplysninger.
{% endhint %}

**Eksempel:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Specialtegn**: Brug enkelt anførselstegn omkring adgangskoder, der indeholder tegn som `$`, `!` eller mellemrum.
{% endhint %}

**Output:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Lagring af legitimationsoplysninger

Cachelagrede legitimationsoplysninger og konfiguration gemmes i mappen `.chloros` i din brugers hjemmemappe på **alle platforme**:

| Platform | Sti til cache med legitimationsoplysninger |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

Hvis du installerer en nyere version oven på en eksisterende, bevares dine loginoplysninger på Windows

(installationsprogrammet) og på Linux

(mappen `.deb`, inklusive `chloros-cli update`). Hvis du afinstallerer Chloros

— eller fjerner Linux

-pakken — slettes den cachelagrede loginoplysning, så du skal logge ind igen efter en ny installation.

### Abonnementsudløb og offline-frist

Abonnementsudløbet i brugergrænsefladen viser, hvornår din licens udløber. For tilbagevendende månedlige abonnementer udløber licensen ved udgangen af måneden; for årlige abonnementer er det et år efter, at du startede abonnementet.

Chloros

validerer din licens online, men det er muligt at arbejde offline inden for en overgangsperiode:

* Vellykkede servervalideringer gemmes i cachen i **5 minutter**, så normal brug medfører meget få licensforespørgsler.
* En signeret, maskinbundet licenscache dækker længere perioder uden internetforbindelse: **30 dage for månedlige abonnementer**og**indtil dit abonnements udløbsdato (højst 365 dage) for årlige abonnementer**.
* Når fristen udløber, skifter abonnementet til det gratis Iron-niveau, indtil maskinen kan oprette forbindelse til licensserveren én gang; adgangen genoptages ved den næste vellykkede kontrol.

### Enhedsbegrænsning

Hvert Chloros

+-abonnement tilbyder et forskelligt antal registrerede enheder. Hver enhed, du logger ind på med en Chloros

+-konto, tæller med i antallet af registrerede enheder. Du kan omdøbe og fjerne en enhed på din MAPIR

Cloud-kontoside.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros-abonnement</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Understøttede enheder</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

Det nøjagtige antal enheder, der er tilladt på din konto, vises på din MAPIR Cloud-kontoside. Når du logger ud af en enhed, frigøres dens plads, og en enhed, der allerede er registreret, kan altid logge ind igen, selv når kontoen har nået sin enhedsgrænse.

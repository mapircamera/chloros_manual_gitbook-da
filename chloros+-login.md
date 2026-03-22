# Chloros+ Login

## Chloros og Chloros (Browser) Login

Via <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> sidebjælkemenuen giver dig mulighed for at logge ind på din Chloros+-konto og låse op for yderligere funktioner.

Når du er logget ind, vises dine kontooplysninger:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI Login

Log ind med dine Chloros+ loginoplysninger for at aktivere CLI-behandling. På Linux (ingen GUI) er dette den eneste måde at aktivere din licens på.

**Syntaks:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**SDK-brugere**: Python SDK tilbyder også en programmatisk `logout()`-metode til at rydde cachelagrede legitimationsoplysninger. Se [Python SDK dokumentationen](api-python-sdk.md#logout) for detaljer.
{% endhint %}

**Eksempel:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Specialtegn**: Brug enkelt anførselstegn omkring adgangskoder, der indeholder tegn som `$`, `!` eller mellemrum.
{% endhint %}

**Output:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Opbevaring af legitimationsoplysninger

Cachelagrede legitimationsoplysninger gemmes på en platformspecifik placering:

| Platform | Sti til cache for legitimationsoplysninger |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### Planens udløb

Planens udløb i GUI&#x27;en viser, hvornår din licens bliver ugyldig. For tilbagevendende månedlige abonnementer er udløbet ved udgangen af måneden. For årlige abonnementer er det et år efter, du startede abonnementet. Licenskontrollen kræver en månedlig internetforbindelse for at verificere, med en 30-dages frist.

### Enhedsbegrænsning

Hver Chloros+-plan tilbyder et forskelligt antal registrerede enheder. Hver enhed, du logger ind på med en Chloros+-konto, tæller med i dit antal registrerede enheder. Du kan omdøbe og fjerne en enhed på din MAPIR Cloud-kontoside.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+-abonnement</th><th align="center">KOBBER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GULD</th></tr></thead><tbody><tr><td align="right">Understøttede enheder</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

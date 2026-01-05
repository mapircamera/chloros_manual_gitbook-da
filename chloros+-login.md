# Chloros+ Login

## Chloros og Chloros (Browser) Login

Brugermenuen <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> sidebar-menuen giver dig mulighed for at logge ind på din Chloros+-konto og låse op for yderligere funktioner.

Når du er logget ind, vises dine kontooplysninger:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI Login

Log ind med dine Chloros+ loginoplysninger for at aktivere CLI-behandling.

**Syntaks:**

```bash
chloros-cli login <email> <password>
```

{% hint style=&quot;info&quot; %}
**SDK-brugere**: Python SDK tilbyder også en programmatisk `logout()`-metode til at rydde cachelagrede loginoplysninger. Se [Python SDK dokumentationen](api-python-sdk.md#logout) for detaljer.
{% endhint %}

**Eksempel:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Specialtegn**: Brug enkelt anførselstegn omkring adgangskoder, der indeholder tegn som `$`, `!` eller mellemrum.
{% endhint %}

**Output:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Planens udløb

Planens udløb i GUI&#x27;en viser, hvornår din licens bliver ugyldig. For tilbagevendende månedlige abonnementer udløber licensen ved udgangen af måneden. For årlige abonnementer udløber licensen et år efter, at du startede abonnementet. Licenskontrollen kræver en månedlig internetforbindelse for at verificere, med en 30 dages frist.

### Enhedsbegrænsning

Hver Chloros+-plan tilbyder et forskelligt antal registrerede enheder. Hver enhed, du logger ind på med en Chloros+-konto, tæller med i dit antal registrerede enheder. Du kan omdøbe og fjerne en enhed på din MAPIR Cloud-kontoside.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+-abonnement</th><th align="center">KOBBER</th><th align="center">BRONZE</th><th align="center">SØLV</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Understøttede enheder</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

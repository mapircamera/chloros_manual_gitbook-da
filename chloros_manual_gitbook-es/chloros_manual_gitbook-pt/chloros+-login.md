# Chloros+ login

## Chloros og Chloros (Browser) Login

Brugermenuen <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> sidebjælke giver dig mulighed for at logge ind på din Chloros+ konto og låse op for yderligere funktioner.

Når du er logget ind vil dine kontooplysninger blive vist:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>

## CLI login

Log ind med dine Chloros+-legitimationsoplysninger for at aktivere CLI-behandling.

**Syntaks:**

```bash
chloros-cli login <email> <password>
```

**Eksempel:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Specialtegn**: Brug enkelte anførselstegn omkring adgangskoder, der indeholder tegn som `$`, `!` eller mellemrum.
{% sluthint %}

**Produktion:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>

### Planens udløb

Planens udløb i GUI viser, hvornår din licens bliver ugyldig. For tilbagevendende månedlige abonnementer er udløbet i slutningen af ​​måneden. For årlige abonnementer er det et år efter, at du startede abonnementet. Licenstjekket kræver en månedlig internetforbindelse for at verificere, med en henstandsperiode på 30 dage.

### Enhedsgrænse

Hver Chloros+ plan tilbyder et forskelligt antal registrerede enheder. Hver enhed, du logger på med en Chloros+-konto, tæller med i dit antal registrerede enheder. Du kan omdøbe og fjerne en enhed på din MAPIR Cloud-kontoside.

<table><thead><tr><th width="168.5999755859375" align="right">Chloros+ Plan</th><th align="center">KOPER</th><th align="center">BRONZE</th><th align="center">SØLV</th><th align="center">GULD</th></tr></thead><tbody><tr><td align="right">Enheder Understøttet</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

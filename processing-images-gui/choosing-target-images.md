# Valg af målbilleder

Det er afgørende at markere, hvilke billeder der indeholder kalibreringsmål, da dette fremskynder Chloros-behandlingsprocessen betydeligt. Ved at forudvælge målbilleder undgår du, at Chloros skal gennemsøge hvert eneste billede i dit datasæt for kalibreringsmål.

## Hvorfor markere målbilleder?

### Behandlingshastighed

Uden at markere målbilleder skal Chloros:

* Scanne hvert eneste billede i dit projekt
* Køre algoritmer til måldetektion på hvert billede
* Tjekke hundredvis eller tusindvis af billeder unødvendigt

**Resultat**: Behandlingen kan tage betydeligt længere tid, især for store datasæt.

### Med markerede målbilleder

Når du markerer kolonnen &quot;Mål&quot; for bestemte billeder:

* Scanner Chloros kun de markerede billeder for mål
* Målregistreringen afsluttes meget hurtigere
* Den samlede behandlingstid reduceres betydeligt

{% hint style="success" %}
**Hastighedsforbedring**: Markering af 2-3 målbilleder i et datasæt på 500 billeder kan reducere målregistreringstiden fra over 30 minutter til under 1 minut.
{% endhint %}

***

## Sådan markerer du målbilleder

### Trin 1: Identificer dine målbilleder

Gennemse dine importerede billeder i filbrowseren, og identificer, hvilke billeder der indeholder kalibreringsmål.

**Almindelige scenarier:*** **Mål før optagelse**: Optaget før sessionen startes
* **Mål efter optagelse**: Optaget efter afslutning af sessionen
* **Mål i feltet**: Mål placeret inden for optagelsesområdet
* **Flere mål**: 2-3 målbilleder pr. session (anbefales)

### Trin 2: Tjek kolonnen &quot;Mål&quot;

For hvert billede, der indeholder et kalibreringsmål:

1. Find billedet i tabellen i filbrowseren
2. Find kolonnen **Mål** (kolonnen længst til højre)
3. Klik på afkrydsningsfeltet i kolonnen &quot;Mål&quot; for det pågældende billede
4. Gentag for alle billeder, der indeholder mål

### Trin 3: Bekræft dit valg

Før behandling skal du dobbelttjekke:

* [ ] Alle billeder med kalibreringsmål er markeret
* [ ] Ingen billeder uden mål er ved en fejl markeret
* [ ] Målene er tydeligt synlige på de markerede billeder

***

## Bedste praksis for målbilleder

### Retningslinjer for optagelse af mål

**Tidspunkt:**

* Tag målbilleder umiddelbart før og under hele din optagelsessession
* Under de samme lysforhold som din DAQ-lyssensor
* Optag ideelt set målbilleder så ofte som muligt for at opnå de bedste resultater. Ellers vil lyssensordataene blive brugt til at justere kalibreringen over tid.

**Kameraposition:**

* Hold kameraet over målet, så det er centreret og fylder ca. 40-60 % af billedets midte.
* Hold kameraet parallelt/lodret i forhold til målets overflade

**Belysning:**

* Samme omgivende belysning som din DAQ-lyssensor
* Undgå skygger på måloverfladerne
* Blokér ikke din lyskilde med din krop, dit køretøj eller vegetation
* Overskyede forhold giver de mest konsistente resultater

**Målforhold:**

* Hold målpanelerne rene og tørre
* Alle 4 paneler skal være tydeligt synlige og uhindrede
* Målene skal være vinkelrette/nadir i forhold til lyskilden, hvis det er muligt

### Hvor mange målbilleder?

**Minimum:**1 målbillede pr. session.**Anbefalet:** 3-5 målbilleder pr. session.**Anbefalet tidsplan:**

* 3-5 billeder taget kort efter, at lyssensoren er begyndt at optage
* Drej kameraet mellem optagelserne for at opnå de bedste resultater
* Valgfrit: med jævne mellemrum midt i sessionen, hvis lysforholdene ændrer sig konstant

***

## Arbejde med flere kameraer

### Opsætning med to kameraer

Hvis du bruger to MAPIR-kameraer samtidigt (f.eks. Survey3W RGN + Survey3N OCN):

1. Optag målbilleder med **begge kameraer** på samme tid
2. Brug **det samme fysiske mål** til begge kameraer
3. Marker målbilleder for **begge kameratyper** i filbrowseren
4. Chloros vil bruge passende mål til kalibrering af hvert kamera

### Kolonnen Kameramodel

Kolonnen **Kameramodel** hjælper med at identificere, hvilke billeder der stammer fra hvilket kamera:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* osv.

Brug denne kolonne til at kontrollere, at du har markeret mål for hver kameratype i dit projekt.

***

## Indstillinger for måldetektion

### Justering af detektionsfølsomhed

Hvis Chloros ikke detekterer dine mål korrekt, skal du justere disse indstillinger i [Projektindstillinger](adjusting-project-settings.md):**Minimum kalibreringsprøveområde:*** **Standard**: 25 pixels
* **Forøg**, hvis der opstår falske registreringer på små artefakter
* **Reducer**, hvis målene ikke registreres**Minimum målgruppering:*** **Standard**: 60
* **Forøg**, hvis målene opdeles i flere registreringer
* **Sænk**, hvis mål med farvevariationer ikke registreres fuldt ud***

## Almindelige problemer med målbilleder

### Problem: Ingen mål registreret

**Mulige årsager:**

* Målbilleder er ikke markeret i filbrowseren
* Målet er for lille i billedet (&lt; 30 % af billedet)
* Dårlig belysning (skygger, genskin)
* Indstillingerne for målregistrering er for strenge

**Løsninger:**

1. Kontroller, at kolonnen &quot;Mål&quot; er markeret for de korrekte billeder
2. Gennemgå kvaliteten af målbillederne i forhåndsvisningen
3. Genoptag målene, hvis kvaliteten er dårlig
4. Juster indstillingerne for måldetektion, hvis nødvendigt

### Problem: Falske måldetektioner

**Mulige årsager:**

* Hvide bygninger, køretøjer eller jorddække forveksles med mål
* Lyse pletter i vegetationen
* Detektionsfølsomheden er for lav

**Løsninger:**

1. Marker kun faktiske målbilleder for at begrænse detekteringsomfanget
2. Forøg det minimale kalibreringsprøveområde
3. Forøg den minimale målklyngeværdi
4. Sørg for, at målbillederne kun viser målet (minimal baggrundsstøj)

***

## Tjekliste til verifikation

Før du starter behandlingen, skal du kontrollere dit valg af målbilleder:

* [ ] Mindst 1 målbillede markeret pr. session
* [ ] Afkrydsningsfelterne i målkolonnen er markeret for alle målbilleder
* [ ] Målbilleder taget inden for samme tidsramme som undersøgelsen
* [ ] Målene er tydeligt synlige i forhåndsvisningen, når der klikkes på dem
* [ ] Alle 4 kalibreringspaneler er synlige i hvert målbillede
* [ ] Ingen skygger eller forhindringer på målene
* [ ] Ved dobbeltkamera: Mål markeret for begge kameratyper

***

## Behandling uden mål

### Behandling uden kalibreringsmål

Selvom det ikke anbefales til videnskabeligt arbejde, kan du behandle uden mål:

1. Lad alle afkrydsningsfelter i målkolonnen være umarkerede
2. **Deaktiver** &quot;Reflektanskalibrering&quot; i projektindstillingerne
3. Vignetteringskorrektion vil stadig blive anvendt
4. Output vil ikke blive kalibreret for absolut reflektans

{% hint style="warning" %}
**Anbefales ikke**: Uden reflektanskalibrering repræsenterer pixelværdierne kun relativ lysstyrke, ikke videnskabelige reflektansmålinger. Brug kalibreringsmål for nøjagtige, repeterbare resultater.
{% endhint %}

***

## Næste trin

Når du har markeret dine målbilleder:

1. **Gennemgå dine indstillinger** – Se [Justering af projektindstillinger](adjusting-project-settings.md)
2. **Start behandlingen** – Se [Start af behandlingen](starting-the-processing.md)
3. **Overvåg fremskridt** – Se [Overvågning af behandlingen](monitoring-the-processing.md)

For mere information om selve kalibreringsmålene, se [Kalibreringsmål](../calibration-targets.md).

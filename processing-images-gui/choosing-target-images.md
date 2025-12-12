# Valg af målbilleder

Det er vigtigt at markere, hvilke billeder der indeholder kalibreringsmål, da det i væsentlig grad fremskynder Chloros-behandlingsprocessen. Ved at forudvælge målbilleder undgår du, at Chloros skal scanne hvert enkelt billede i dit datasæt for kalibreringsmål.

## Hvorfor markere målbilleder?

### Behandlingshastighed

Uden at markere målbilleder skal Chloros:

* Scanne hvert enkelt billede i dit projekt
* Køre måldetekteringsalgoritmer på hvert billede
* Kontrollere hundredvis eller tusindvis af billeder unødigt

**Resultat**: Behandlingen kan tage betydeligt længere tid, især for store datasæt.

### Med markerede målbilleder

Når du markerer kolonnen Mål for bestemte billeder:

* Chloros scanner kun de markerede billeder for mål
* Målregistreringen afsluttes meget hurtigere
* Den samlede behandlingstid reduceres betydeligt

{% hint style=&quot;success&quot; %}
**Hastighedsforbedring**: Markering af 2-3 målbilleder i et datasæt med 500 billeder kan reducere måldetekteringstiden fra over 30 minutter til under 1 minut.
{% endhint %}

***

## Sådan markeres målbilleder

### Trin 1: Identificer dine målbilleder

Gennemse dine importerede billeder i filbrowseren og identificer, hvilke billeder der indeholder kalibreringsmål.

**Almindelige scenarier:**

* **Mål før optagelse**: Optaget før sessionen startes.
* **Mål efter optagelse**: Optaget efter sessionen er afsluttet.
* **Mål i feltet**: Mål placeret inden for optagelsesområdet.
* **Flere mål**: 2-3 målbilleder pr. session (anbefales).

### Trin 2: Kontroller målkolonnen

For hvert billede, der indeholder et kalibreringsmål:

1. Find billedet i filbrowsertabellen.
2. Find kolonnen **Mål** (kolonnen længst til højre).
3. Klik på afkrydsningsfeltet i kolonnen Mål for det pågældende billede.
4. Gentag for alle billeder, der indeholder mål.

### Trin 3: Bekræft dit valg

Før behandlingen skal du dobbelttjekke:

* [ ] Alle billeder med kalibreringsmål er markeret
* [ ] Ingen billeder uden mål er markeret ved en fejl
* [ ] Målene er tydeligt synlige på de markerede billeder

***

## Bedste praksis for målbilleder

### Retningslinjer for optagelse af mål

**Tidspunkt:**

* Optag målbilleder umiddelbart før og under hele optagelsessessionen
* Under de samme lysforhold som din DAQ-lyssensor
* Optag helst målbilleder så ofte som muligt for at opnå de bedste resultater. Ellers vil lyssensordataene blive brugt til at justere kalibreringen over tid.

**Kameraposition:**

* Hold kameraet over målet, så det er centreret og fylder ca. 40-60 % af billedets midte.
* Hold kameraet parallelt/nadir med måloverfladen

**Belysning:**

* Samme omgivende belysning som din DAQ-lyssensor
* Undgå skygger på måloverfladerne
* Blokér ikke din lyskilde med din krop, dit køretøj eller vegetation
* Overskyede forhold giver de mest konsistente resultater

**Målbetingelser:**

* Hold målpanelerne rene og tørre
* Alle 4 paneler skal være tydeligt synlige og uhindrede
* Målene skal være vinkelrette/nadir i forhold til lyskilden, hvis det er muligt

### Hvor mange målbilleder?

**Minimum:** 1 målbillede pr. session. **Anbefalet:** 3-5 målbilleder pr. session.

**Bedste praksis-tidsplan:**

* 3-5 billeder taget kort efter, at lyssensoren er begyndt at optage
* Drej kameraet mellem optagelserne for at opnå de bedste resultater
* Valgfrit: periodisk midt i sessionen, hvis lysforholdene ændrer sig konstant

***

## Arbejde med flere kameraer

### Opsætning med to kameraer

Hvis du bruger to MAPIR-kameraer samtidigt (f.eks. Survey3W RGN + Survey3N OCN):

1. Tag målbilleder med **begge kameraer** på samme tid.
2. Brug det **samme fysiske mål** for begge kameraer.
3. Marker målbilleder for **begge kameratyper** i filbrowseren.
4. Chloros bruger passende mål til kalibrering af hvert kamera.

### Kolonne med kameramodel

Kolonnen **Kameramodel** hjælper med at identificere, hvilke billeder der stammer fra hvilket kamera:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* osv.

Brug denne kolonne til at kontrollere, at du har markeret mål for hver kameratype i dit projekt.

***

## Indstillinger for måldetektering

### Justering af detekteringsfølsomhed

Hvis Chloros ikke detekterer dine mål korrekt, skal du justere disse indstillinger i [Projektindstillinger](adjusting-project-settings.md):

**Minimumskalibreringsprøveområde:**

* **Standard**: 25 pixels
* **Forøg**, hvis der opstår falske detektioner på små artefakter
* **Reducer**, hvis mål ikke detekteres

**Minimumsklyngedannelse af mål:**

* **Standard**: 60
* **Forøg**, hvis målene opdeles i flere registreringer
* **Formindsk**, hvis mål med farvevariationer ikke registreres fuldt ud

***

## Almindelige problemer med målbilleder

### Problem: Ingen mål registreret

**Mulige årsager:**

* Målbilleder er ikke markeret i filbrowseren
* Målet er for lille i rammen (&lt; 30 % af billedet)
* Dårlig belysning (skygger, blænding)
* Indstillingerne for måldetektion er for strenge

**Løsninger:**

1. Kontroller, at kolonnen Mål er markeret for de korrekte billeder
2. Kontroller kvaliteten af målbilledet i forhåndsvisningen
3. Genoptag målene, hvis kvaliteten er dårlig
4. Juster indstillingerne for måldetektion, hvis det er nødvendigt

### Problem: Falske måldetektioner

**Mulige årsager:**

* Hvide bygninger, køretøjer eller jorddækning forveksles med mål
* Lyse pletter i vegetationen
* Detektionsfølsomheden er for lav

**Løsninger:**

1. Marker kun faktiske målbilleder for at begrænse detektionsomfanget
2. Øg det minimale kalibreringsprøveområde
3. Øg den minimale målklyngeværdi
4. Sørg for, at målbillederne kun viser målet (minimal baggrundsstøj)

***

## Tjekliste til verifikation

Før du starter behandlingen, skal du kontrollere dit valg af målbilleder:

* [ ] Mindst 1 målbillede markeret pr. session
* [ ] Afkrydsningsfelterne i målkolonnen er markeret for alle målbilleder
* [ ] Målbilleder optaget inden for samme tidsramme som undersøgelsen
* [ ] Målene er tydeligt synlige i forhåndsvisningen, når der klikkes på dem
* [ ] Alle 4 kalibreringspaneler er synlige i hvert målbillede
* [ ] Ingen skygger eller forhindringer på målene
* [ ] For dobbeltkamera: Mål markeret for begge kameratyper

***

## Målfri behandling

### Behandling uden kalibreringsmål

Selvom det ikke anbefales til videnskabeligt arbejde, kan du behandle uden mål:

1. Lad alle afkrydsningsfelter i kolonnen Mål være umarkerede
2. **Deaktiver** &quot;Reflektanskalibrering&quot; i Projektindstillinger
3. Vignettekorrektion vil stadig blive anvendt
4. Output vil ikke blive kalibreret for absolut reflektans

{% hint style=&quot;warning&quot; %}
**Ikke anbefalet**: Uden reflektanskalibrering repræsenterer pixelværdier kun relativ lysstyrke, ikke videnskabelige reflektansmålinger. Brug kalibreringsmål for nøjagtige, repeterbare resultater.
{% endhint %}

***

## Næste trin

Når du har markeret dine målbilleder:

1. **Gennemgå dine indstillinger** - Se [Justering af projektindstillinger](adjusting-project-settings.md)
2. **Start behandlingen** - Se [Start af behandlingen](starting-the-processing.md)
3. **Overvåg fremskridt** - Se [Overvågning af behandlingen](monitoring-the-processing.md)

For mere information om kalibreringsmålene selv, se [Kalibreringsmål](../calibration-targets.md).

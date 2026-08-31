# Valg af målbilleder

Når du markerer, hvilke billeder der indeholder kalibreringsmål, fortæller du Chloros præcis, hvor den skal lede efter dem. Når mindst ét billede er markeret i kolonnen »Target«, scanner Chloros **kun de markerede billeder** — så ved at markere målene kan du både fremskynde behandlingen og sikre, at kortlægningsbilleder ikke forveksles med et mål.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Hvorfor markere målbilleder?

### Markering styrer scanningen

Når du markerer kolonnen »Mål« for bestemte billeder:

* Chloros scanner kun de markerede billeder for mål
* Målregistreringen afsluttes meget hurtigere
* Kortlægningsbilleder kan ikke forårsage falske målregistreringer

Hvis der **ikke** er markeret nogen billeder, falder Chloros tilbage til at scanne hvert eneste billede i projektet:

* Målregistreringsalgoritmerne kører på hvert enkelt billede
* Hundredvis eller tusindvis af billeder kontrolleres unødvendigt
* Behandlingen tager betydeligt længere tid, især for store datasæt

{% hint style="success" %}
**Hastighedsforbedring**: Ved at markere 2–3 målbilleder i et datasæt på 500 billeder kan måldetekteringstiden reduceres fra over 30 minutter til under 1 minut.
{% endhint %}

***

## Sådan markerer du målbilleder

### Trin 1: Identificer dine målbilleder

Gennemse dine importerede billeder i filbrowseren, og identificer, hvilke billeder der indeholder kalibreringsmål.

**Almindelige scenarier:*** **Mål før optagelse**: Optaget før sessionen startes
* **Mål efter optagelse**: Optaget efter afslutning af sessionen
* **Mål i feltet**: Mål placeret inden for optagelsesområdet
* **Flere mål**: 2–3 målbilleder pr. session (anbefales)

### Trin 2: Marker afkrydsningsfeltet i kolonnen **Target** <img src="../.gitbook/assets/image (33).png" alt="" data-size="original">

For hvert billede, der indeholder et kalibreringsmål:

1. Find billedet i tabellen i filbrowseren
2. Find kolonnen **Target** (kolonnen længst til højre)
3. Klik på afkrydsningsfeltet i kolonnen **Target** for det pågældende billede
4. Gentag for alle billeder, der indeholder mål

### Trin 3: Bekræft dit valg

Før behandling skal du dobbelttjekke:

* [ ] Alle billeder med kalibreringsmål er markeret
* [ ] Ingen billeder uden mål er ved en fejl markeret
* [ ] Målene er tydeligt synlige på de markerede billeder

***

## LATTICE: Mål er valgfrie, når en DAQ optager

For LATTICE-multispektralkameraer er et kalibreringsmål i billedet **en af to** mulige reflektansreferencer:

* **Mål inden for billedrammen**: Når et markeret målbillede opfylder kvalitetskravene (QA) frChloros, bliver målet den**absolutte reflektansreference** for de billeder, der omgiver det.
* **DAQ-nedstråling**: Når der ikke er noget mål til stede (eller QA mislykkes), beregner Chloros i stedet reflektansen ud fra DAQ-lyssensorens nedadgående irradians (ρ = π·L/E). Hvis en `.daq`- eller DAQ-M `.csv`-optagelse dækker dine optagelser, får du kalibreret reflektans**helt uden målbilleder**.

Denne automatiske adfærd er standardindstillingen. I CLI / SDK svarer det til `--reflectance-source auto`; du kan også tvinge `target` (streng — ingen DAQ-substitution) eller `daq` (DAQ-autoritativ). Se [CLI-referencen](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**LATTICE-målgeometrier**: ud over den klassiske paneldetektion, der bruges til Survey3, understøtter LATTICE-behandlingen**ArUco-mærkede mål**,**mål med fast ROI**og**strimmelmål**, der konfigureres pr. projekt.**Målte** reflektansscanninger pr. enhed kan leveres med serienummer (CLI: `--target-reflectance-dir`, én `<serial>.csv` pr. målenhed), med de nominelle T3/T4P-spektre som reserve.

{% hint style="info" %}
**F988-modul**: F988-reflektansen kalibreres ved hjælp af et reflektanspanel i selve scenen: båndet ligger uden for DAQ-lyssensorens kalibrerede område, så Chloros anvender din seneste paneloptagelse og opbevarer den mellem panelobservationerne. Hvis et F988-modul behandles udelukkende med DAQ, afviser Chloros DAQ-baseret reflektans for det pågældende bånd (springe over-årsag `dls-uncalibrated-band-988`) — panel-workflowet er den understøttede fremgangsmåde.
{% endhint %}

***

## Bedste praksis for målbilleder

### Retningslinjer for optagelse af mål

**Tidspunkt:**

* Tag målbilleder umiddelbart før og under hele din optagelsessession
* Under de samme lysforhold som din DAQ-lyssensor
* Optag helst målbilleder så ofte som muligt for at opnå de bedste resultater. Ellers vil lyssensordataene blive brugt til at justere kalibreringen over tid.

**Kameraposition:**

* Hold kameraet over målet, så det er centreret og fylder ca. 40–60 % af billedets midte.
* Hold kameraet parallelt med/lodret under målets overflade

**Belysning:**

* Samme omgivende belysning som din DAQ-lyssensor
* Undgå skygger på målets overflader
* Blokér ikke lyskilden med din krop, dit køretøj eller vegetation
* Overskyede forhold giver de mest ensartede resultater

**Målets tilstand:**

* Hold målpanelerne rene og tørre
* Alle paneler på dit mål (f.eks. alle 4 på en T4) skal være tydeligt synlige og uhindrede
* Målene skal om muligt være vinkelrette på/nadir i forhold til lyskilden

### Hvor mange målbilleder?

**Minimum:**1 målbillede pr. session.**Anbefalet:** 3–5 målbilleder pr. session.**Anbefalet tidsplan:**

* 3–5 billeder taget kort efter, at lyssensoren er begyndt at optage
* Drej kameraet mellem optagelserne for at opnå de bedste resultater
* Valgfrit: med jævne mellemrum midt i sessionen, hvis lysforholdene ændrer sig konstant

***

## Arbejde med flere kameraer

### Opsætninger med to kameraer

Hvis du bruger to MAPIR-kameraer samtidigt (f.eks. Survey3W RGN + Survey3N OCN):

1. Tag målbilleder med **begge kameraer** på samme tid
2. Brug **det samme fysiske mål** til begge kameraer
3. Markér målbillederne for **begge kameratyper** i filbrowseren
4. Chloros vil anvende de relevante mål til kalibrering af hvert kamera

### Kolonnen &quot;Kameramodel&quot;

Kolonnen **&quot;Kameramodel&quot;** hjælper med at identificere, hvilke billeder der stammer fra hvilket kamera:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* osv.

Brug denne kolonne til at kontrollere, at du har markeret mål for hver kameratype i dit projekt.

***

## Indstillinger for måldetektion

### Justering af detektionsfølsomhed

Hvis Chloros ikke detekterer dine mål korrekt, skal du justere disse indstillinger i [Projektindstillinger](adjusting-project-settings.md):**Minimalt kalibreringsprøveområde (px):*** **Standard**: 25 pixels
* **Forøg**, hvis der opstår falske registreringer på små artefakter
* **Sænk**, hvis målene ikke registreres**Minimum målgruppering (0-100):*** **Standard**: 60
* **Forøg**, hvis målene opdeles i flere registreringer
* **Sænk**, hvis mål med farvevariationer ikke registreres fuldt ud

{% hint style="info" %}
**Tip til CLI**: `chloros-cli process` accepterer de samme indstillingsmuligheder (`--min-target-size`, `--target-clustering`), og dens `--target`/`--targets`-flag markerer en hel inputmappe som »kun målpanel«. Se [CLI-referencen](../reference/cli-reference.md).
{% endhint %}

***

## Almindelige problemer med målbilleder

### Problem: Ingen mål registreret

**Mulige årsager:**

* Målbillederne er ikke markeret i filbrowseren
* Målet er for lille i billedet (&lt; 30 % af billedet)
* Dårlig belysning (skygger, genskin)
* Indstillingerne for måldetektion er for strenge

**Løsninger:**

1. Kontroller, at kolonnen »Mål« er markeret for de korrekte billeder
2. Kontroller kvaliteten af målbillederne i forhåndsvisningen
3. Tag nye billeder af målene, hvis kvaliteten er dårlig
4. Juster indstillingerne for måldetektion, hvis det er nødvendigt

### Problem: Falske måldetektioner

**Mulige årsager:**

* Hvide bygninger, køretøjer eller jorddække forveksles med mål
* Lyse pletter i vegetationen
* Detektionsfølsomheden er for lav

**Løsninger:**

1. Markér kun faktiske målbilleder — kun markerede billeder scannes
2. Forøg det mindste kalibreringsprøveområde
3. Forøg den mindste værdi for målgruppering
4. Sørg for, at målbillederne kun viser målet (minimalt baggrundsstøj)

***

## Tjekliste til verifikation

Inden du påbegynder behandlingen, skal du kontrollere dit valg af målbilleder:

* [ ] Mindst 1 markeret målbillede pr. session (eller, for LATTICE, en `.daq`/`.csv`-optagelse, der dækker sessionen)
* [ ] Afkrydsningsfelterne i målsøjlen er markeret for alle målbilleder
* [ ] Målbillederne er optaget inden for samme tidsramme som undersøgelsen
* [ ] Målene er tydeligt synlige i forhåndsvisningen, når der klikkes på dem
* [ ] Alle kalibreringspaneler er synlige i hvert målbillede
* [ ] Ingen skygger eller forhindringer på målene
* [ ] Ved brug af to kameraer: Målene er markeret for begge kameratyper

***

## Behandling uden mål

### LATTICE: Med en DAQ-optagelse

Hvis en DAQ-lyssensor har registreret nedadgående strålingsintensitet under dine LATTICE-optagelser, er der ikke behov for et mål:

1. Importer filen `.daq` (eller DAQ-M `.csv`) med billedmaterialet
2. Lad kolonnen »Mål« være umarkeret
3. Reflektansen beregnes automatisk ud fra DAQ&#x27;ens nedadgående reference
4. Strålingsintensiteten kræver aldrig et mål eller en DAQ — den stammer udelukkende fra kameraets radiometriske fabrikskalibrering

### Behandling uden nogen reference

Du kan også behandle uden mål og uden en DAQ:

1. Lad alle afkrydsningsfelter i kolonnen »Mål« være umarkerede
2. **Deaktiver** »Reflektanskalibrering / hvidbalance« i projektindstillingerne — målregistrering springes da helt over
3. Vignettkorrektion vil stadig blive anvendt
4. Outputtet vil ikke blive kalibreret for absolut reflektans (LATTICE multispektral eksporterer stadig debayered-, preview- og radiance-produkter)

{% hint style="warning" %}
**Anbefales ikke til Survey3 videnskabeligt arbejde**: Uden reflektanskalibrering repræsenterer Survey3-pixelværdier kun relativ lysstyrke, ikke videnskabelige reflektansmålinger. Brug kalibreringsmål (eller, for LATTICE, en DAQ-lyssensor) for at opnå nøjagtige, repeterbare resultater.
{% endhint %}

***

## Næste trin

Når du har markeret dine målbilleder:

1. **Gennemgå dine indstillinger** – Se [Justering af projektindstillinger](adjusting-project-settings.md)
2. **Start behandlingen** – Se [Start af behandlingen](starting-the-processing.md)
3. **Overvåg fremskridtet** – Se [Overvågning af behandlingen](monitoring-the-processing.md)

For mere information om selve kalibreringsmålene, se [Kalibreringsmål](../calibration-targets.md).

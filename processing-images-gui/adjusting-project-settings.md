# Justering af projektindstillinger

Inden du behandler dine billeder, er det vigtigt at konfigurere dine projektindstillinger, så de passer til dine arbejdsgangskrav. Panelet »Projektindstillinger« <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> giver dig omfattende kontrol over kalibrering, behandlingsindstillinger, multispektrale indekser og eksportformater.

## Åbning af projektindstillinger

1. Åbn dit projekt i Chloros
2. Klik på ikonet **Projektindstillinger** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> i venstre sidepanel
3. Panelet Projektindstillinger viser alle konfigurationsmuligheder

{% hint style="info" %}
**Indstillingerne gemmes automatisk** sammen med dit projekt. Når du genåbner et projekt, gendannes alle indstillinger.
{% endhint %}

***

## Hurtig opsætning til almindelige arbejdsgange

### Standardindstillinger (anbefales til de fleste brugere)

Til typiske MAPIR Survey3 kameraworkflows fungerer standardindstillingerne godt:

* ✅ **Vignetteringskorrektion**: Aktiveret
* ✅ **Refleksionskalibrering**: Aktiveret (kræver billeder af MAPIR-mål)
* ✅ **Debayer-metode**: Standard (Hurtig, Medium kvalitet)
* ✅ **Eksportformat**: TIFF (16-bit)

Du skal blot importere dine billeder og starte behandlingen med disse standardindstillinger.

***

## Oversigt over projektindstillinger

Panelet Projektindstillinger er opdelt i flere kategorier. Nedenfor findes et resumé af hvert afsnit. Se [Projektindstillinger](../project-settings/project-settings.md) for den fulde dokumentation.

### Målregistrering

Styrer, hvordan Chloros identificerer kalibreringsmål i dine billeder.

**Vigtige indstillinger:*** **Minimum kalibreringsprøveområde**: Størrelsestærskel for måldetektion (standard: 25 pixels)
* **Minimum målgruppering**: Lighedstærskel for gruppering af målområder (standard: 60)**Hvornår skal du justere:**

* Forøg prøveområdet, hvis der opstår falske detektioner
* Reducer det, hvis mål ikke detekteres
* Juster grupperingen, hvis mål opdeles i flere detektioner

### Behandling

Vigtigste billedbehandlings- og kalibreringsindstillinger.

**Vigtige indstillinger:*** **Vignettekorrektion**: Kompenserer for mørkningen af linsen ved kanterne ✅ Anbefales
* **Reflektanskalibrering**: Normaliserer værdier ved hjælp af kalibreringsmål ✅ Anbefales
* **Debayer-metode**: Algoritme til konvertering af RAW til 3-kanals multispektral
* **Minimalt rekalibreringsinterval**: Tiden mellem brug af kalibreringsmål (0 = brug alle)**Avancerede indstillinger:*** **Lyssensorens tidszoneforskydning**: Til PPK-tidssynkronisering (standard: 0)
* **Anvend PPK-korrektioner**: Bruger GPS-/eksponeringspin-data fra .daq-filer
* **Eksponeringspin 1/2**: Tildeler kameraer til eksponeringspins til opsætninger med to kameraer

### Debayer-metode

Vi tilbyder i øjeblikket 2 debayering-metoder i Chloros:

#### Standard (Hurtig, Mellem kvalitet)

Standard debayer behandler hurtigt, men viser debayering-farvestøj, hvilket resulterer i mindre nøjagtige og mere støjende billeder.

#### Teksturbevidst (langsom, højeste kvalitet) \[Kun Chloros+]

Teksturbevidst bruger en højkvalitets kantbevidst debayer kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al debayering-støj. Teksturbevidst-modellen kræver GPU-hukommelse (VRAM) for at køre. Vi anbefaler at bruge den, når du har &gt;4 GB VRAM til rådighed for hurtigere behandling.

### Indeks (multispektrale indekser)

Konfigurer, hvilke vegetationsindekser der skal beregnes og eksporteres.

**Sådan tilføjes indekser:**

1. Klik på knappen**&quot;Tilføj indeks&quot;**

2. Vælg et indeks fra rullemenuen (NDVI, NDRE, GNDVI osv.)
3. Konfigurer visualiseringsindstillinger (LUT-farver, værdiintervaller)
4. Tilføj flere indekser efter behov

**Populære indekser:*** **NDVI**: Generel vegetationssundhed (mest almindelig)
* **NDRE**: Tidlig stressdetektering sammen med RedEdge
* **GNDVI**: Følsom over for klorofylkoncentration
* **OSAVI**: Fungerer godt med synlig jord
* **EVI**: Områder med højt bladarealindeks (LAI)**Brugerdefinerede formler (kun Chloros+):**

* Opret brugerdefinerede multispektrale indeksformler
* Brug båndmatematik med alle billedkanaler
* Gem brugerdefinerede formler til genbrug

Se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md) for alle tilgængelige indekser og formler.

### Eksport

Styrer outputfilformat og kvalitet.

**Tilgængelige formater:*** **TIFF (16-bit)**: Anbefales til GIS og videnskabelig analyse (interval 0-65.535)
* **TIFF (32-bit, procent)**: Reflektansværdier med flydende komma (interval 0,0-1,0)
* **PNG (8-bit)**: Tabsfri komprimering til visualisering (interval 0-255)
* **JPG (8-bit)**: Mindste filstørrelse, tabsgivende komprimering (interval 0-255)***

## Gemme og indlæse indstillinger

### Gem projektmal

Opret genanvendelige skabeloner for ensartede arbejdsgange:

1. Konfigurer alle ønskede indstillinger i panelet Projektindstillinger
2. Rul ned til afsnittet **&quot;Gem projektskabelon&quot;** nederst
3. Indtast et beskrivende skabelonnavn (f.eks. &quot;Survey3N\_RGN\_Landbrug&quot;)
4. Klik på gem-ikonet

**Fordele:**

* Anvend identiske indstillinger på tværs af flere projekter
* Del konfigurationer med teammedlemmer
* Oprethold konsistens ved gentagne undersøgelser

### Indlæs skabelon i nyt projekt

Når du opretter et nyt projekt:

1. Vælg **&quot;Nyt projekt&quot;** fra hovedmenuen
2. Vælg indstillingen **&quot;Indlæs fra skabelon&quot;**

3. Vælg din gemte skabelon
4. Alle indstillinger anvendes automatisk

### Arbejdsmappe

Indstillingen **&quot;Gem projektmappe&quot;** angiver, hvor nye projekter oprettes som standard:

* **Standardplacering**: `C:\Users\[Username]\Chloros Projects`
* **Skift placering**: Klik på redigeringsikonet og vælg en ny mappe
* **Hvornår skal du ændre**:
  * Netværksdrev til teamsamarbejde
  * Et andet drev med mere lagerplads
  * Organiseret mappestruktur efter år/kunde

***

## PPK-opsætning (Post-Processed Kinematic)

Hvis du bruger MAPIR DAQ-optagere med GPS til præcis geolokalisering:

### Forudsætninger

* MAPIR DAQ med GPS (GNSS)-modul
* .daq-logfil med eksponeringspin-indtastninger
* Kamera tilsluttet DAQ-eksponeringspins under optagelsessessionen

### Konfigurationstrin

1. Placer .daq-logfilen i din projektmappe
2. I Projektindstillinger skal du aktivere afkrydsningsfeltet **&quot;Anvend PPK-korrektioner&quot;**

3. Indstil**&quot;Lyssensorens tidszoneforskydning&quot;**, hvis nødvendigt (standard: 0 for UTC)
4. Tildel kameraer til eksponeringsstik:
   * **Enkelt kamera**: Tildeles automatisk til stik 1
   * **To kameraer**: Tildel hvert kamera manuelt til det korrekte stik**Tildeling af eksponeringsstik:*** **Eksponeringspin 1**: Vælg kameramodel fra rullemenuen
* **Eksponeringspin 2**: Vælg andet kamera eller &quot;Brug ikke&quot;
* Det samme kamera kan ikke tildeles begge pins

{% hint style="warning" %}
**Vigtigt**: Eksponeringspins skal være korrekt tildelt deres respektive kameraer. Forkert tildeling vil resultere i forkerte geolokaliseringsdata.
{% endhint %}

***

## Avancerede scenarier

### Projekter med flere kameraer

Når der behandles billeder fra flere MAPIR-kameraer i ét projekt:

1. Chloros registrerer automatisk hver kameramodel
2. Hvert kamera får den passende behandlingsprofil
3. PPK: Tildel manuelt hvert kamera til den korrekte eksponeringspin
4. Alle kameraer bruger samme eksportformat og indekser

**Eksempel**: Survey3W RGN + Survey3N OCN dobbeltkamerarig

### Time-lapse- eller flerdato-undersøgelser

Ved gentagne undersøgelser af det samme område over tid:

1. Opret en skabelon med dine standardindstillinger
2. Brug ensartet opsætning af kalibreringsmål ved hver session
3. Behandl hver dato som et separat projekt
4. Brug identiske indstillinger for sammenlignelige resultater
5. Eksporter i samme format til tidsmæssig analyse

### Store datasæt

Ved projekter med mange billeder (500+):

* Overvej at opdele i mindre projekter efter dato eller område
* Brug Chloros+ parallelbehandling for hurtigere resultater
* Overvej CLI eller API til batchautomatisering
* Juster minimumsintervallet for rekalibrering for at reducere måldetekteringstiden

***

## Verificering af dine indstillinger

Før du begynder at behandle, skal du gennemgå disse vigtige indstillinger:

* [ ] Kameramodel korrekt registreret i filbrowseren
* [ ] Vignettekorrektion aktiveret
* [ ] Reflektanskalibrering aktiveret
* [ ] Mindst ét kalibreringsmålbillede importeret
* [ ] Ønskede multispektrale indekser tilføjet
* [ ] Eksportformat, der passer til din arbejdsgang
* [ ] PPK-indstillinger konfigureret (hvis du bruger .daq med eksponeringshændelser)

***

## Næste trin

Når dine indstillinger er konfigureret:

1. **Marker kalibreringsmålbilleder** – Se [Valg af målbilleder](choosing-target-images.md)
2. **Start behandlingen** – Se [Start af behandlingen](starting-the-processing.md)
3. **Overvåg fremskridt** – Se [Overvågning af behandlingen](monitoring-the-processing.md)

For fuldstændige detaljer om alle tilgængelige indstillinger, se referencedokumentationen [Projektindstillinger](../project-settings/project-settings.md).

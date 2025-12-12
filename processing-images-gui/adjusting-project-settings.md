# Justering af projektindstillinger

Før du behandler dine billeder, er det vigtigt at konfigurere dine projektindstillinger, så de passer til dine arbejdsgangskrav. Projektindstillingspanelet <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> giver omfattende kontrol over kalibrering, behandlingsindstillinger, multispektrale indekser og eksportformater.

## Adgang til projektindstillinger

1. Åbn dit projekt i Chloros
2. Klik på ikonet **Projektindstillinger** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> i venstre sidepanel
3. Panelet Projektindstillinger viser alle konfigurationsmuligheder

{% hint style=&quot;info&quot; %}
**Indstillingerne gemmes automatisk** sammen med dit projekt. Når du genåbner et projekt, gendannes alle indstillinger.
{% endhint %}

***

## Hurtig opsætning til almindelige arbejdsgange

### Standardindstillinger (anbefales til de fleste brugere)

For typiske MAPIR Survey3 kamera-arbejdsgange fungerer standardindstillingerne godt:

* ✅ **Vignettekorrektion**: Aktiveret
* ✅ **Reflektanskalibrering**: Aktiveret (kræver billeder af MAPIR mål)
* ✅ **Debayer-metode**: Høj kvalitet (hurtigere)
* ✅ **Eksportformat**: TIFF (16-bit)

Du skal blot importere dine billeder og starte behandlingen med disse standardindstillinger.

***

## Oversigt over projektindstillinger

Panelet Projektindstillinger er inddelt i flere kategorier. Nedenfor findes en oversigt over hver sektion. For fuldstændig dokumentation, se [Projektindstillinger](../project-settings/project-settings.md).

### Målregistrering

Styrer, hvordan Chloros identificerer kalibreringsmål i dine billeder.

**Vigtige indstillinger:**

* **Minimumskalibreringsprøveområde**: Størrelsestærskel for måldetektion (standard: 25 pixels)
* **Minimum målklyngedannelse**: Lighedstærskel for gruppering af målregioner (standard: 60)

**Hvornår skal du justere:**

* Forøg prøveområdet, hvis du får falske detektioner.
* Reducer, hvis målene ikke bliver detekteret.
* Juster klyngedannelsen, hvis målene bliver opdelt i flere detektioner.

### Behandling

Vigtigste billedbehandlings- og kalibreringsindstillinger.

**Vigtige indstillinger:**

* **Vignettekorrektion**: Kompenserer for mørkfarvning af linsen i kanterne ✅ Anbefales
* **Reflektanskalibrering**: Normaliserer værdier ved hjælp af kalibreringsmål ✅ Anbefales
* **Debayer-metode**: Algoritme til konvertering af RAW til 3-kanals multispektral
* **Minimumskalibreringsinterval**: Tid mellem brug af kalibreringsmål (0 = brug alle)

**Avancerede indstillinger:**

* **Lyssensor-tidszoneforskydning**: Til PPK-tidssynkronisering (standard: 0)
* **Anvend PPK-korrektioner**: Bruger GPS/eksponeringspin-data fra .daq-filer
* **Eksponeringspin 1/2**: Tildeler kameraer til eksponeringspins til opsætninger med to kameraer

### Indeks (multispektrale indekser)

Konfigurer, hvilke vegetationsindekser der skal beregnes og eksporteres.

**Sådan tilføjes indekser:**

1. Klik på knappen **&quot;Tilføj indeks&quot;**
2. Vælg et indeks fra rullemenuen (NDVI, NDRE, GNDVI osv.)
3. Konfigurer visualiseringsindstillinger (LUT-farver, værdiintervaller)
4. Tilføj flere indekser efter behov

**Populære indekser:**

* **NDVI**: Generel vegetationssundhed (mest almindelig)
* **NDRE**: Tidlig stressdetektering med RedEdge
* **GNDVI**: Følsom over for klorofylkoncentration
* **OSAVI**: Fungerer godt med synlig jord
* **EVI**: Regioner med højt bladarealindeks (LAI)

**Brugerdefinerede formler (kun Chloros+):**

* Opret brugerdefinerede multispektrale indeksformler
* Brug båndmatematik med alle billedkanaler
* Gem brugerdefinerede formler til genbrug

Se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md) for alle tilgængelige indekser og formler.

### Eksport

Styrer outputfilformat og kvalitet.

**Tilgængelige formater:**

* **TIFF (16-bit)**: Anbefales til GIS og videnskabelig analyse (interval 0-65.535)
* **TIFF (32-bit, procent)**: Flydende reflektansværdier (interval 0,0-1,0)
* **PNG (8-bit)**: Tabsfri komprimering til visualisering (interval 0-255)
* **JPG (8-bit)**: Mindste filer, tabsgivende komprimering (interval 0-255)

***

## Gemme og indlæse indstillinger

### Gem projektmal

Opret genanvendelige skabeloner for ensartede arbejdsgange:

1. Konfigurer alle ønskede indstillinger i panelet Projektindstillinger.
2. Rul ned til afsnittet **&quot;Gem projekt skabelon&quot;** nederst.
3. Indtast et beskrivende navn på skabelonen (f.eks. &quot;Survey3N\_RGN\_Landbrug&quot;).
4. Klik på ikonet for gem.

**Fordele:**

* Anvend identiske indstillinger på tværs af flere projekter.
* Del konfigurationer med teammedlemmer.
* Bevar konsistensen ved gentagne undersøgelser.

### Indlæs skabelon i nyt projekt

Når du opretter et nyt projekt:

1. Vælg **&quot;Nyt projekt&quot;** i hovedmenuen.
2. Vælg **&quot;Indlæs fra skabelon&quot;**.
3. Vælg din gemte skabelon.
4. Alle indstillinger anvendes automatisk.

### Arbejdsmappe

Indstillingen **&quot;Gem projektmappe&quot;** angiver, hvor nye projekter oprettes som standard:

* **Standardplacering**: `C:\Users\[Username]\Chloros Projects`
* **Skift placering**: Klik på redigeringsikonet, og vælg en ny mappe
* **Hvornår skal du skifte**:
  * Netværksdrev til teamsamarbejde
  * Et andet drev med mere lagerplads
  * Organiseret mappestruktur efter år/kunde

***

## PPK (Post-Processed Kinematic) opsætning

Hvis du bruger MAPIR DAQ-optagere med GPS til præcis geolokalisering:

### Forudsætninger

* MAPIR DAQ med GPS (GNSS)-modul
* .daq-logfil med eksponeringspin-indtastninger
* Kamera tilsluttet DAQ-eksponeringsstik under optagelsessessionen

### Konfigurationsskridt

1. Placer .daq-logfilen i din projektmappe.
2. I Projektindstillinger skal du aktivere afkrydsningsfeltet **&quot;Anvend PPK-korrektioner&quot;**.
3. Indstil **&quot;Lyssensorens tidszoneforskydning&quot;**, hvis det er nødvendigt (standard: 0 for UTC).
4. Tildel kameraer til eksponeringsstik:
   * **Enkelt kamera**: Tildeles automatisk til pin 1
   * **To kameraer**: Tildel hvert kamera manuelt til den korrekte pin

**Tildeling af eksponeringspins:**

* **Eksponeringspin 1**: Vælg kameramodel fra rullemenuen
* **Eksponeringspin 2**: Vælg andet kamera eller &quot;Brug ikke&quot;
* Det samme kamera kan ikke tildeles til begge pins

{% hint style=&quot;warning&quot; %}
**Vigtigt**: Eksponeringsstifter skal tildeles korrekt til deres respektive kameraer. Forkert tildeling vil resultere i forkerte geolokaliseringsdata.
{% endhint %}

***

## Avancerede scenarier

### Projekter med flere kameraer

Når du behandler billeder fra flere MAPIR-kameraer i ét projekt:

1. Chloros registrerer automatisk hver kameramodel
2. Hvert kamera får den passende behandlingsprofil
3. PPK: Tildel manuelt hvert kamera den korrekte eksponeringspin
4. Alle kameraer bruger samme eksportformat og indekser

**Eksempel**: Survey3W RGN + Survey3N OCN dobbeltkamera-rig

### Time-lapse- eller multi-date-undersøgelser

Til gentagne undersøgelser af det samme område over tid:

1. Opret en skabelon med dine standardindstillinger
2. Brug ensartet kalibreringsmålopsætning hver session
3. Behandl hver dato som et separat projekt
4. Brug identiske indstillinger for sammenlignelige resultater
5. Eksporter i samme format til tidsmæssig analyse

### Store datasæt

Til projekter med mange billeder (500+):

* Overvej at opdele i mindre projekter efter dato eller område.
* Brug Chloros+ parallelbehandling for hurtigere resultater.
* Overvej CLI eller API til batchautomatisering.
* Juster minimumsintervallet for rekalibrering for at reducere måldetekteringstiden.

***

## Bekræftelse af dine indstillinger

Før du begynder at behandle, skal du gennemgå disse vigtige indstillinger:

* [ ] Kameramodel korrekt registreret i filbrowseren
* [ ] Vignettekorrektion aktiveret
* [ ] Reflektanskalibrering aktiveret
* [ ] Mindst et kalibreringsmålbillede importeret
* [ ] Ønskede multispektrale indekser tilføjet
* [ ] Eksportformat, der passer til din arbejdsgang
* [ ] PPK-indstillinger konfigureret (hvis du bruger .daq med eksponeringshændelser)

***

## Næste trin

Når dine indstillinger er konfigureret:

1. **Marker kalibreringsmålbilleder** - Se [Valg af målbilleder](choosing-target-images.md)
2. **Start behandlingen** - Se [Start af behandlingen](starting-the-processing.md)
3. **Overvåg fremskridt** - Se [Overvågning af behandlingen](monitoring-the-processing.md)

For fuldstændige detaljer om alle tilgængelige indstillinger, se referencedokumentationen [Projektindstillinger](../project-settings/project-settings.md).

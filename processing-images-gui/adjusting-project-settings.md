# Justering af projektindstillinger

Inden du behandler dine billeder, er det vigtigt at konfigurere dine projektindstillinger, så de passer til dine arbejdsgangskrav. Panelet »Projektindstillinger« <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> giver dig omfattende kontrol over kalibrering, behandlingsindstillinger, multispektrale indekser og eksportformater.

## Sådan åbner du projektindstillingerne

1. Åbn dit projekt i Chloros
2. Klik på ikonet **Projektindstillinger** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> i venstre sidepanel
3. Panelet Projektindstillinger viser alle konfigurationsmuligheder

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Panelet **Projektindstillinger** — Visning, måldetektion og behandling</p></figcaption></figure>

{% hint style="info" %}
**Indstillingerne gemmes automatisk** sammen med dit projekt. Når du genåbner et projekt, gendannes alle indstillinger.
{% endhint %}

***

## Hurtig opsætning til almindelige arbejdsgange

### Standardindstillinger (anbefales til de fleste brugere)

Standardindstillingerne fungerer godt til typiske Survey3
- og LATTICE-arbejdsgange:

* ✅ **Vignettekorrektion**: Aktiveret
* ✅ **Refleksionskalibrering / hvidbalance**: Aktiveret (bruger MAPIR
-mål og/eller DAQ-lyssensordata)
* ✅ **Debayer-metode**: Standard (Hurtig, Mellemkvalitet)
* ✅ **Eksportformat**: TIFF
(16-bit) (TIFF
(8-bit) til et projekt, der udelukkende består af 8-bit)
* ✅ **Alle eksportprodukter**: Aktiveret (LATTICE gemmer automatisk fan-out i debayered, preview, radiance og reflektans)

Du skal blot importere dine billeder og starte behandlingen med disse standardindstillinger.

***

## Oversigt over projektindstillinger

Panelet Projektindstillinger er opdelt i nedenstående sektioner. To yderligere sektioner — **DAQ-lyssensor**og**Array-justering** — vises automatisk, når dit projekt indeholder de relevante filer. Se [Projektindstillinger](../project-settings/project-settings.md) for fuldstændig dokumentation.

### Visning

* **Opløsning af billedminiaturer**: Opløsningen af miniaturerne i billedgitteret. Indstillinger:**Standard (512 px)**,**1024 px**,**2048 px**,**Fuld opløsning**. Gælder kun visning – påvirker aldrig behandlingen. Højere værdier ser skarpere ud, når der zoomes ind, men indlæses langsommere.

### Målregistrering

Styrer, hvordan Chloros
identificerer kalibreringsmål i dine billeder.

**Vigtige indstillinger:*** **Minimalt kalibreringsprøveområde (px)**: Størrelsestærskel for målregistrering (standard:**25**, interval 0–10000)
* **Minimum målgruppering (0–100)**: Tærskelværdi for lighed ved gruppering af målområder (standard:**60**)**Hvornår skal du justere:**

* Øg prøveområdet, hvis der opstår falske detektioner
* Sænk det, hvis målene ikke detekteres
* Juster grupperingen, hvis målene opdeles i flere detektioner

{% hint style="info" %}
Disse indstillinger er nedtonede, når **Reflektanskalibrering / hvidbalance** er slået fra — når den er slået fra, kører måldetektion slet ikke.
{% endhint %}

### Behandling

Vigtigste indstillinger for billedbehandling og kalibrering.

**Vigtige indstillinger:*** **Vignetteringskorrektion**: Kompenserer for mørkningen ved kanterne ✅ Anbefales
* **Reflektanskalibrering / hvidbalance**: Kalibrerer billeder ved hjælp af detekterede mål (Survey3
) og/eller data fra DAQ-lyssensoren (LATTICE) ✅ Anbefales
* **Debayer-metode**: Algoritme til konvertering af RAW til 3-kanals multispektral
* **Minimalt rekalibreringsinterval**: Mindste tid i sekunder mellem brug af kalibreringsmål (standard:**0** = brug alle, interval 0–3600)**Ukalibrerede reserveprodukter:**Når et billede ikke kan refleksionskalibreres (intet mål tilgængeligt eller kalibrering deaktiveret), eksporteres det som et af to reserveprodukter —**der findes nøjagtigt ét af parret pr. kørsel**, valgt via Vignette-korrektionsknappen:

* **Eksporter sensorrespons**: skriver `Sensor_Response_Images` — bruges, når vignettkorrektion er**slået fra*** **Eksporter vignettekorrigeret**: skriver `Vignette_Corrected_Images` — bruges, når vignettekorrektion er**aktiveret**Det afkrydsningsfelt, der ikke er aktivt, er nedtonet. Hvis du fjerner markeringen fra det aktive felt, forhindres den pågældende fil i at blive skrevet overhovedet.**LATTICE-eksportprodukter** (vises for hvert projekt; de gælder for LATTICE-optagelser):

* **Eksporter debayered**: det lineære debayered-billede (`Debayered_Images`): kalibreringspakkens mørkeforskydning fjernes, hvorefter billedet demosaiceres uden flat field eller unmix — den samme fil, som en optagelse gemmer på sit debayered-niveau — i sensorens oprindelige retning. Gælder for RGB
- og multispektrale moduler.
* **Eksporter forhåndsvisning**: skærmforhåndsvisningen (`Preview_Images`). RGB
= hvidbalance (DAQ-lysart, når tilgængelig, ellers gråskala) + gamma; multispektral = falskfarvestrækning.
* **Eksport af strålingsintensitet**: float32 spektral strålingsintensitet (`Radiance_Images`, W/m²/sr/nm). Gælder kun multispektrale moduler — gælder ikke for RGB
-masterfiler.
* **Eksporter reflektans**: uint16-reflektans (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0), når en `.daq`-nedstrålingsmåling eller et mål inden for billedrammen dækker billedrammen. Gælder kun multispektrale moduler.

Alle fire er **aktiveret som standard**— et importeret LATTICE-råbillede fordeles til alle aktiverede og relevante produkter i én enkelt behandlingsomgang. Afkrydsningsfeltet**Eksporter reflektans** er nedtonet, når reflektankalibrering er deaktiveret. Indstillinger, som ikke kan vælges på grund af en overordnet indstilling, er altid nedtonet med et værktøjstip, der angiver, hvilken indstilling der skal ændres.**Avancerede indstillinger:*** **Tidszoneforskydning for lyssensor**: Antal timer fra UTC til tidsjustering af lyssensoren (standard: 0, interval −12 til +12)
* **Anvend PPK-korrektioner**: Bruger GPS-/eksponeringspin-data fra `.daq`-filer (standard: slået fra)
* **Eksponeringspin 1/2**: Tildeler kameraer til eksponeringspins ved opsætninger med to kameraer

{% hint style="info" %}
**LATTICE-indgangsniveauet er automatisk.** LATTICE-optagelser medbringer deres behandlingsniveau i XMP-metadata, og behandlingen indgår altid i pipelinen ved den rå ramme — der er intet at konfigurere i GUI&#x27;en. (CLI
-flagget `--input-level` findes som en overstyring for avancerede brugere til optagelser med mistede metadata; se [CLI
-referencen](../reference/cli-reference.md).)
{% endhint %}

### Debayer-metode

Vi tilbyder i øjeblikket 2 debayering-metoder i Chloros
:

#### Standard (hurtig, middel kvalitet)

Standard-debayeren behandler hurtigt, men viser farvestøj fra debayeringen, hvilket resulterer i mindre nøjagtige og mere støjfyldte billeder.

#### Teksturbevidst (langsom, højeste kvalitet) \[Kun Chloros
+]

Teksturbevidst bruger en højkvalitets kantbevidst debayer kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al støj fra debayeringen. Modellen kræver GPU-hukommelse (VRAM) for at køre: med **7 GB eller mere VRAM** kan den behandle flere billeder samtidigt; under 7 GB behandler den ét billede ad gangen (mærkbart langsommere). Se [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**LATTICE-optagelser bruger altid standard-demosaic.** Der findes ingen LATTICE-trænet Texture Aware-model, så denne mulighed tilbydes ikke for LATTICE-billeder — Survey3
-billeder i samme projekt kan dog stadig bruge den.
{% endhint %}

### Indeks (multispektrale indekser)

Konfigurer, hvilke vegetationsindekser der skal beregnes og eksporteres. Rullemenuen i brugergrænsefladen tilbyder **27 foruddefinerede indeksformler**.**Sådan tilføjes indekser:**

1. Klik på knappen**&quot;Tilføj indeks&quot;**

2. Vælg et indeks fra rullemenuen (NDVI
, NDRE
, GNDVI
osv.)
3. Konfigurer visualiseringsindstillinger (LUT-farver, værdiintervaller)
4. Tilføj flere indekser efter behov

**Populære indekser:*** **NDVI**: Generel vegetationssundhed (mest almindelig)
* **NDRE**: Tidlig stressdetektering med RedEdge
* **GNDVI**: Følsom over for klorofylkoncentration
* **OSAVI**: Fungerer godt med synlig jord
* **EVI**: Områder med højt bladarealindeks (LAI
)

**Brugerdefinerede formler:**

* Opret brugerdefinerede multispektrale indeksformler med båndregning på tværs af alle billedkanaler
* Gem brugerdefinerede formler til genbrug
* Brugerdefinerede formler er en funktion i Chloros
+; tilgængeligheden afhænger af dit abonnementsniveau

For alle tilgængelige indekser og formler — herunder hvilke navne der kun findes i GUI’en, og hvilke der også fungerer i CLI
’en/SDK
’en — se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md).

### Eksport

Styrer outputfilformatet.

**Tilgængelige formater**(indstilling:**Kalibreret billedformat**, standard**TIFF
(16-bit)**):

* **TIFF
(16-bit)**: Anbefales til GIS og videnskabelig analyse
* **TIFF
(8-bit)**: 8-bit TIFF
* **TIFF
(32-bit, procent)**: Flydende komma-værdier
* **PNG
(8-bit)**: Tabsfri komprimering til visualisering
* **JPG (8-bit)**: Mindste filstørrelse, tabsgivende komprimering

Listen viser, hvad projektets billeder kan indeholde: Chloros
opskalerer ikke bitdybden. Alle 8-bit-billeder tilbyder kun TIFF
(8-bit), PNG
(8-bit) og JPG (8-bit), og standardindstillingen er TIFF
(8-bit). I et blandet projekt eksporteres hver optagelse med sin egen dybde eller lavere. Radiance forbliver 32-bit float.

Uddata gemmes i projektmappen, grupperet efter kamera og format: `<project>/<camera>/<format>/<Product>_Images/`. Radiance gemmes **altid** som float32 i mappen `tiff32`, uanset denne indstilling. Eksporterede filer beholder kildefilnavnet — mappen identificerer produktet. LATTICE-multispektrale eksportfiler kan også skrive pixel-baserede clamp-masker til en `clamp_masks/`-mappe ved siden af produktmapperne. Se [Afslutning af behandlingen](finishing-the-processing.md) for den fulde outputstruktur.

{% hint style="warning" %}
**Aflæsning af reflektansværdier**: Den DN-værdi, der svarer til ρ = 1,0, afhænger af kildekameraet — LATTICE bruger 32768 (mærket som XMP `Chloros:PixelScale`), mens Survey3
bruger 65535. Læs tagget i stedet for at antage en konstant. Se [Output Image Formats](../output-image-formats.md).
{% endhint %}

### DAQ-lyssensor

Dette afsnit viser alle DAQ-downwelling-filer (`.daq` / `.csv`) i dit projekt, én række pr. fil, med angivelse af sensormodel, filnavn og den **cap** korrektion, der gælder for den pågældende fil.

* **Overskrivning af cap (alle filer)**: en enkelt rullemenu, der gælder for hele projektet.**Auto** (standard) bruger den cap, der er registreret for hver fil — der antages solskin, hvor der ikke er registreret noget, men DAQ-U, DAQ-M og DAQ-E leveres uden cap, og Chloros
kan ikke se, hvad der er tilpasset, så behandlingen giver en advarsel om en antaget cap: vælg den cap, der blev tilpasset. Valg af en cap tilsidesætter alle filer undtagen en DAQ-E-S&#x27;s, hvis diffusor er en del af dens kalibrering: råoptagelser korrigeres med den, og optagelser, der allerede har en cap, omrefereres (den registrerede korrektion fortrydes, og den valgte cap anvendes).
* Der vises advarsler i rækkerne, når en registreret hætte var hubbenes antagede standardindstilling i stedet for at være bekræftet af operatøren, og når den valgte hætte ikke har en profil for den pågældende enhedsmodel (tilpasningen afvises for den pågældende fil).

DAQ-optagelser foretaget under fanen Lyssensorer føjes automatisk til det åbne projekt, og importerede `.daq` / `.csv`-filer vises her, så snart de er tilføjet.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Nederste projektindstillinger — Indeks, eksportformat, afsnittet DAQ-lyssensorer samt kontrolelementerne til projektmal/mappe</p></figcaption></figure>### Array-justering

Dette afsnit vises **kun**, når mindst ét billede i projektet indeholder den modul-til-modul-justeringstransformation, som LATTICE-arrays påfører ved optagelsen (`Chloros:Alignment*` XMP). Det viser, hvor mange billeder der har tags, og hvilket kamera der er referencen, med følgende indstillinger:

* **Anvend array-justering** (standard: aktiveret): tilpasser hvert behandlet produkt (debayered / preview / radiance / reflectance / index) til arrayets fælles referencegeometri. Deaktiveret = eksport i sensorens oprindelige geometri.
* **Beskær til fælles overlapning** (standard: aktiveret): beskær justerede eksporter til det område, som alle moduler deler, så hvert bånd har samme dækningsområde. Deaktiveret bevarer det fulde sensorområde (sort udfyldning uden for kilden).
* **Resampling**:**Bilineær (glat, standard)**,**Nærmeste (bevarer nøjagtige værdier)**— ingen blanding mellem pixels, til streng radiometrisk analyse — eller**Kubisk (skarpest)**.***

## Gemme og indlæse indstillinger

### Gem projektmal

Opret genanvendelige maler til ensartede arbejdsgange:

1. Konfigurer alle ønskede indstillinger i panelet Projektindstillinger
2. Rul ned til afsnittet **&quot;Gem projektmal&quot;** nederst
3. Indtast et beskrivende skabelonnavn (f.eks. &quot;Survey3N

\_RGN\_Agriculture&quot;)
4. Klik på gem-ikonet

**Fordele:**

* Anvend identiske indstillinger på tværs af flere projekter
* Del konfigurationer med teammedlemmer
* Oprethold konsistens ved gentagne undersøgelser

### Indlæs skabelon i nyt projekt

Når du opretter et nyt projekt:

1. Vælg **&quot;Nyt projekt&quot;** i hovedmenuen
2. Vælg en projektskabelon i den valgfri skabelonvælger
3. Alle indstillinger fra skabelonen anvendes automatisk

### Arbejdsmappe

Indstillingen **&quot;Arbejdsmappe&quot;** angiver, hvor nye projekter oprettes som standard:

* **Standardplacering**: `Documents\MAPIR\Chloros Projects` i din Windows

-mappe &quot;Dokumenter&quot; (normalt `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`; den følger mappen &quot;Dokumenter&quot;, hvis OneDrive eller en roaming-profil flytter den)
* **Skift placering**: Klik på redigeringsikonet og vælg en ny mappe
* **Delt med CLI**: `chloros-cli` bruger den samme standardindstilling for projektmappen
* **Hold det kort i Windows**: medmindre lange stier i Windows

er aktiveret, afvises en projektmappe, der er længere end ca. 130 tegn, før behandlingen påbegyndes — se [Start af behandlingen](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)
* **Hvornår skal du ændre det**:
  * Netværksdrev til teamsamarbejde
  * Et andet drev med mere lagerplads
  * Organiseret mappestruktur efter år/kunde

***

## PPK-opsætning (Post-Processed Kinematic)

Hvis du bruger MAPIR

DAQ-optagere med GPS til præcis geolokalisering:

### Forudsætninger

* MAPIR

DAQ med GPS (GNSS)-modul
* .daq-logfil med indtastninger for eksponeringsstik
* Kamera tilsluttet DAQ-eksponeringsstikkene under optagelsessessionen

### Konfigurationstrin

1. Placer .daq-logfilen i din projektmappe
2. I Projektindstillinger skal du markere afkrydsningsfeltet **&quot;Anvend PPK-korrektioner&quot;**

3. Indstil**&quot;Tidszoneforskydning for lyssensor&quot;**, hvis nødvendigt (standard: 0 for UTC)
4. Tildel kameraer til eksponeringsstik:
   * **Enkelt kamera**: Tildeles automatisk til stik 1
   * **To kameraer**: Tildel hvert kamera manuelt til det korrekte stik**Tildeling af eksponeringsstik:*** **Eksponeringsstik 1**: Vælg kameramodel fra rullemenuen
* **Eksponeringsstik 2**: Vælg det andet kamera eller &quot;Brug ikke&quot;
* Det samme kamera kan ikke tildeles til begge stik

{% hint style="warning" %}
**Vigtigt**: Eksponeringsstikkene skal tildeles korrekt til de respektive kameraer. Forkert tildeling vil resultere i forkerte geolokaliseringsdata.
{% endhint %}

***

## Avancerede scenarier

### Projekter med flere kameraer

Når der behandles billeder fra flere MAPIR

-kameraer i ét projekt:

1. Chloros

registrerer automatisk hver kameramodel (både Survey3

og LATTICE)
2. Hvert kamera tildeles passende behandlingsprofiler, og hvert kamera får sin egen udgangsmappestruktur
3. PPK: Tildel manuelt hvert Survey3

-kamera til den korrekte eksponeringspin
4. Alle kameraer bruger samme eksportformat og indekser

**Eksempler**: Survey3W

RGN

+ Survey3N

OCN

-rig med to kameraer, eller et LATTICE-array, der kombinerer et RGB

-masterkamera med smalbåndsmoduler

### Time-lapse-optagelser eller undersøgelser på flere datoer

Ved gentagne undersøgelser af det samme område over tid:

1. Opret en skabelon med dine standardindstillinger
2. Brug en ensartet opsætning af kalibreringsmål ved hver session
3. Behandl hver dato som et separat projekt
4. Brug identiske indstillinger for at opnå sammenlignelige resultater
5. Eksporter i samme format til tidsmæssig analyse

### Store datasæt

Til projekter med mange billeder (500+):

* Overvej at opdele i mindre projekter efter dato eller område
* Brug Chloros

+ parallelbehandling for hurtigere resultater
* Overvej at bruge CLI

eller API

til batch-automatisering
* Juster det minimale rekalibreringsinterval for at reducere måldetekteringstiden

***

## Kontrol af dine indstillinger

Før du begynder at behandle, skal du gennemgå disse vigtige indstillinger:

* [ ] Kameramodellen er korrekt registreret i filbrowseren
* [ ] Vignetteringskorrektion aktiveret
* [ ] Reflektanskalibrering aktiveret
* [ ] For Survey3

: mindst ét kalibreringsmålbillede importeret og kontrolleret; for LATTICE: et mål og/eller en `.daq`-nedadrettet optagelse til stede
* [ ] De ønskede multispektrale indekser er tilføjet
* [ ] Eksportformat, der passer til din arbejdsgang
* [ ] PPK-indstillinger er konfigureret (hvis du bruger .daq med eksponeringshændelser)

***

## Næste trin

Når dine indstillinger er konfigureret:

1. **Marker kalibreringsmålbilleder** – Se [Valg af målbilleder](choosing-target-images.md)
2. **Start behandlingen** – Se [Start af behandlingen](starting-the-processing.md)
3. **Overvåg fremskridt** – Se [Overvågning af behandlingen](monitoring-the-processing.md)

For fuldstændige detaljer om alle tilgængelige indstillinger, se referencedokumentationen [Projektindstillinger](../project-settings/project-settings.md).

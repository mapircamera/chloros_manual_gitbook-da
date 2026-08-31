# Justering af projektindstillinger

Inden du behandler dine billeder, er det vigtigt at konfigurere dine projektindstillinger, så de passer til dine krav til arbejdsgangen. Panelet »Projektindstillinger« (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">) giver dig omfattende kontrol over kalibrering, behandlingsindstillinger, multispektrale indekser og eksportformater.

## Sådan åbner du projektindstillingerne

1. Åbn dit projekt i Chloros
2. Klik på ikonet **Projektindstillinger** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> i venstre sidepanel
3. Panelet Projektindstillinger viser alle konfigurationsmuligheder

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Panelet **Projektindstillinger** — Visning, måldetektion og behandling</p></figcaption></figure>{% hint style="info" %}**Indstillingerne gemmes automatisk** sammen med dit projekt. Når du genåbner et projekt, gendannes alle indstillinger.
{% endhint %}

***

## Hurtig opsætning til almindelige arbejdsgange

### Standardindstillinger (anbefales til de fleste brugere)

Standardindstillingerne fungerer godt til typiske Survey3- og LATTICE-arbejdsgange:

* ✅ **Vignetteringskorrektion**: Aktiveret
* ✅ **Refleksionskalibrering / hvidbalance**: Aktiveret (bruger MAPIR-mål og/eller DAQ-lyssensordata)
* ✅ **Debayer-metode**: Standard (hurtig, middel kvalitet)
* ✅ **Eksportformat**: TIFF (16-bit)
* ✅ **Alle eksportprodukter**: Aktiveret (LATTICE eksporterer automatisk til debayered, preview, radiance og reflectance)

Du skal blot importere dine billeder og gå i gang med behandlingen med disse standardindstillinger.

***

## Oversigt over projektindstillinger

Panelet **Projektindstillinger**er opdelt i nedenstående sektioner. To yderligere sektioner —**DAQ-lyssensor**og**Array-justering** — vises automatisk, når dit projekt indeholder de relevante filer. Se [Projektindstillinger](../project-settings/project-settings.md) for den fulde dokumentation.

### Visning

* **Opløsning af billedminiaturer**: Opløsningen af miniaturerne i billedgitteret. Valgmuligheder:**Standard (512 px)**,**1024 px**,**2048 px**,**Fuld opløsning**. Kun til visning — påvirker aldrig behandlingen. Højere værdier ser skarpere ud, når der zoomes ind, men indlæses langsommere.

### Målregistrering

Styrer, hvordan Chloros identificerer kalibreringsmål i dine billeder.

**Vigtige indstillinger:*** **Minimalt kalibreringsprøveområde (px)**: Størrelsestærskel for måldetektering (standard:**25**, interval 0–10000)
* **Minimal målgruppering (0–100)**: Lighedstærskel for gruppering af målområder (standard:**60**)**Hvornår skal du justere:**

* Forøg prøveområdet, hvis der opstår falske registreringer
* Sænk det, hvis målene ikke registreres
* Juster grupperingen, hvis målene opdeles i flere registreringer

{% hint style="info" %}
Disse indstillinger er nedtonede, når **reflektanskalibrering / hvidbalance** er slået fra — når den er slået fra, kører måldetektionen slet ikke.
{% endhint %}

### Behandling

Vigtigste indstillinger for billedbehandling og kalibrering.

**Vigtige indstillinger:*** **Vignetteringskorrektion**: Kompenserer for mørkningen i billedets kanter ✅ Anbefales
* **Reflektanskalibrering / hvidbalance**: Kalibrerer billeder ved hjælp af detekterede mål (Survey3) og/eller data fra DAQ-lyssensoren (LATTICE) ✅ Anbefales
* **Debayer-metode**: Algoritme til konvertering af RAW til 3-kanals multispektralt
* **Minimalt rekalibreringsinterval**: Mindste tid i sekunder mellem brug af kalibreringsmål (standard:**0** = brug alle, interval 0–3600)**Ukalibrerede reserveprodukter:**Når et billede ikke kan refleksionskalibreres (intet mål tilgængeligt eller kalibrering deaktiveret), eksporteres det som et af to reserveprodukter —**der findes nøjagtigt ét af parret pr. kørsel**, valgt af Vignette-korrektionskontakten:

* **Eksporter sensorrespons**: skriver `Sensor_Response_Images` — bruges, når vignettkorrektion er**slået fra*** **Eksporter vignettekorrigeret**: skriver `Vignette_Corrected_Images` — bruges, når vignettekorrektion er**aktiveret**Det afkrydsningsfelt, der ikke er aktivt, er nedtonet. Hvis du fjerner markeringen fra det aktive felt, forhindres den pågældende fil i at blive skrevet.**LATTICE-eksportprodukter** (vises for hvert projekt; de gælder for LATTICE-optagelser):

* **Eksporter debayered**: det lineært debayered billede (`Debayered_Images`). Gælder for RGB og multispektrale moduler.
* **Eksporter forhåndsvisning**: skærmforhåndsvisningen (`Preview_Images`). RGB = hvidbalance (DAQ-lysforhold, når tilgængeligt, ellers gråskala) + gamma; multispektral = falskfarvestrækning.
* **Eksport af strålingsintensitet**: float32 spektral strålingsintensitet (`Radiance_Images`, W/m²/sr/nm). Gælder kun multispektrale moduler — gælder ikke for RGB-masterfiler.
* ****Eksporter reflektans**: uint16-reflektans (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0), når en `.daq`-nedstrålingsmåling eller et mål inden for billedrammen dækker rammen. Gælder kun multispektrale moduler.

Alle fire er **aktiveret som standard**— en importeret LATTICE-råramme fordeles til alle aktiverede og relevante produkter i et enkelt behandlingsforløb. Afkrydsningsfeltet**Eksporter reflektans** er nedtonet, når reflektankalibrering er deaktiveret. Indstillinger, som ikke kan vælges på grund af en overordnet indstilling, er altid nedtonet med et værktøjstip, der angiver, hvilken indstilling der skal ændres.**Avancerede indstillinger:*** **Tidszoneforskydning for lyssensor**: Timer fra UTC til tidssynkronisering af lyssensoren (standard: 0, interval −12 til +12)
* **Anvend PPK-korrektioner**: Bruger GPS-/eksponeringspin-data fra `.daq`-filer (standard: slået fra)
* **Eksponeringspin 1/2**: Tildeler kameraer til eksponeringspins ved opsætninger med to kameraer

{% hint style="info" %}
**LATTICE-indgangsniveauet er automatisk.** LATTICE-optagelser medbringer deres behandlingsniveau i XMP-metadata, og behandlingen indgår altid i pipelinen ved det rå billede — der er intet at konfigurere i brugergrænsefladen. (Flagget CLI og `--input-level` findes som en overstyring for avancerede brugere til optagelser med mistede metadata; se [CLI-referencen](../reference/cli-reference.md).)
{% endhint %}

### Debayer-metode

Vi tilbyder i øjeblikket 2 debayering-metoder i Chloros:

#### Standard (hurtig, middel kvalitet)

Standard-debayeren behandler hurtigt, men viser farvestøj fra debayeringen, hvilket resulterer i mindre nøjagtige og mere støjende billeder.

#### Teksturbevidst (langsom, højeste kvalitet) \[Kun Chloros+]

Teksturbevidst bruger en kantbevidst debayer af høj kvalitet kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al støj fra debayeringen. Modellen kræver GPU-hukommelse (VRAM) for at køre: med **7 GB eller mere VRAM** kan den behandle flere billeder samtidigt; under 7 GB behandler den ét billede ad gangen (mærkbart langsommere). Se [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**LATTICE-optagelser bruger altid standard-demosaic.** Der findes ingen LATTICE-trænet teksturbevidst model, så indstillingen tilbydes ikke for LATTICE-billeder — Survey3-billeder i samme projekt kan dog stadig bruge den.
{% endhint %}

### Indeks (multispektrale indekser)

Konfigurer, hvilke vegetationsindekser der skal beregnes og eksporteres. Rullemenuen i brugergrænsefladen tilbyder **27 foruddefinerede indeksformler**.**Sådan tilføjes indekser:**

1. Klik på knappen**&quot;Tilføj indeks&quot;**

2. Vælg et indeks fra rullemenuen (NDVI, NDRE, GNDVI osv.)
3. Konfigurer visualiseringsindstillingerne (LUT-farver, værdiintervaller)
4. Tilføj flere indekser efter behov

**Populære indekser:*** **NDVI**: Generel vegetationssundhed (mest almindelig)
* **NDRE**: Tidlig stressdetektering sammen med RedEdge
* **GNDVI**: Følsom over for klorofylkoncentration
* **OSAVI**: Fungerer godt med synlig jord
* **EVI**: Områder med højt bladarealindeks (LAI)**Brugerdefinerede formler:**

* Opret brugerdefinerede multispektrale indeksformler med båndregning på tværs af alle billedkanaler
* Gem brugerdefinerede formler til genbrug
* Brugerdefinerede formler er en Chloros+-funktion; tilgængeligheden afhænger af dit abonnementsniveau

For alle tilgængelige indekser og formler — herunder hvilke navne der kun findes i brugergrænsefladen, og hvilke der også fungerer i CLI/SDK — se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md).

### Eksport

Styrer outputfilformatet.

**Tilgængelige formater**(indstilling:**Kalibreret billedformat**, standard**TIFF (16-bit)**):

* **TIFF (16-bit)**: Anbefales til GIS og videnskabelig analyse
* **TIFF (32-bit, procent)**: Flydende komma-værdier
* **PNG (8-bit)**: Tabsfri komprimering til visualisering
* **JPG (8-bit)**: Mindste filstørrelse, tabsgivende komprimering

Resultatfilerne gemmes i projektmappen, grupperet efter kamera og format: `<project>/<camera>/<format>/<Product>_Images/`. Radiance gemmes **altid** som float32 i mappen `tiff32`, uanset denne indstilling. Eksporterede filer beholder kildefilnavnet — mappen identificerer produktet. Se [Afslutning af behandlingen](finishing-the-processing.md) for den fulde uddatastruktur.

{% hint style="warning" %}
**Aflæsning af reflektansværdier**: den DN, der betyder ρ = 1,0, afhænger af kildekameraet — LATTICE bruger 32768 (mærket som XMP `Chloros:PixelScale`), Survey3 bruger 65535. Læs tagget i stedet for at antage en konstant. Se [Output Image Formats](../output-image-formats.md).
{% endhint %}

### DAQ-lyssensor

Dette afsnit viser alle DAQ-downwelling-filer (`.daq` / `.csv`) i dit projekt, én række pr. fil, med angivelse af sensormodel, filnavn og den **cap**-korrektion, der gælder for den pågældende fil.

* **Cap-tilsidesættelse (alle filer)**: en enkelt rullemenu, der gælder for hele projektet.**Auto** (standard) bruger den registrerede cap for hver fil — der antages solskin, hvor der ikke er registreret noget, da alle MAPIR DAQ&#x27;er leveres med solskinskorrektoren. Valg af en cap tilsidesætter alle filer: råoptagelser korrigeres med den, og optagelser, der allerede har en cap, omrefereres (den registrerede korrektion annulleres, og den valgte cap anvendes).
* Rækker viser en advarsel, når en registreret grænseværdi var hubben antaget standardværdi i stedet for at være bekræftet af operatøren, og når den valgte grænseværdi ikke har en profil for den pågældende enhedsmodel (tilpasningen afvises for den pågældende fil).

DAQ-optagelser foretaget under fanen Lyssensorer føjes automatisk til det åbne projekt, og importerede `.daq` / `.csv`-filer vises her, så snart de tilføjes.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Nederste projektindstillinger — Indeks, eksportformat, afsnittet DAQ-lyssensorer samt kontrolelementerne til projektskabeloner og -mapper</p></figcaption></figure>### Array-justering

Dette afsnit vises **kun**, når mindst ét billede i projektet indeholder den modul-til-modul-justeringstransformation, som LATTICE-arrays påfører ved optagelsen (`Chloros:Alignment*` XMP). Det viser, hvor mange billeder der har tags, og hvilket kamera der er referencen, med følgende kontrolelementer:

* **Anvend array-justering** (standard: aktiveret): forvrænger hvert behandlet produkt (debayered / preview / radiance / reflectance / index) til arrayets fælles referencegeometri. Deaktiveret = eksport i sensorens oprindelige geometri.
* **Beskær til fælles overlapning** (standard: aktiveret): beskær justerede eksporter til det område, som alle moduler deler, så hvert bånd har samme dækningsområde. Deaktiveret bevarer det fulde sensorområde (sort udfyldning uden for kilden).
* **Resampling**:**Bilineær (glat, standard)**,**Nærmeste (bevarer nøjagtige værdier)**— ingen blanding mellem pixels, til streng radiometrisk analyse — eller**Kubisk (skarpest)**.***

## Gemme og indlæse indstillinger

### Gem projektskabelon

Opret genanvendelige skabeloner for ensartede arbejdsgange:

1. Konfigurer alle ønskede indstillinger i panelet »Projektindstillinger«
2. Rul ned til afsnittet **»Gem projektskabelon«** nederst
3. Indtast et beskrivende skabelonnavn (f.eks. &quot;Survey3N\_RGN\_Agriculture&quot;)
4. Klik på gem-ikonet

**Fordele:**

* Anvend identiske indstillinger på tværs af flere projekter
* Del konfigurationer med teammedlemmer
* Sikr ensartethed ved gentagne undersøgelser

### Indlæs skabelon i nyt projekt

Når du opretter et nyt projekt:

1. Vælg **&quot;Nyt projekt&quot;** fra hovedmenuen
2. Vælg en projektskabelon i den valgfri skabelonvælger
3. Alle indstillinger fra skabelonen anvendes automatisk

### Arbejdsmappe

Indstillingen **&quot;Arbejdsmappe&quot;** angiver, hvor nye projekter som standard oprettes:

* **Standardplacering**: `C:\Users\[Username]\Chloros Projects`
* **Skift placering**: Klik på redigeringsikonet, og vælg en ny mappe
* **Delt med CLI**: `chloros-cli` bruger den samme standardindstilling for projektmappe
* **Hvornår skal der skiftes**:
  * Netværksdrev til teamsamarbejde
  * Et andet drev med mere lagerplads
  * Organiseret mappestruktur efter år/kunde

***

## Opsætning af PPK (Post-Processed Kinematic)

Hvis du bruger MAPIR DAQ-optagere med GPS til præcis geolokalisering:

### Forudsætninger

* MAPIR DAQ med GPS (GNSS)-modul
* .daq-logfil med indtastninger af eksponeringsstik
* Kamera tilsluttet DAQ-eksponeringsstikkene under optagelsessessionen

### Konfigurations trin

1. Placer .daq-logfilen i din projektmappe
2. I Projektindstillinger skal du markere afkrydsningsfeltet **&quot;Anvend PPK-korrektioner&quot;**

3. Indstil**&quot;Tidszoneforskydning for lyssensor&quot;**, hvis nødvendigt (standard: 0 for UTC)
4. Tildel kameraer til eksponeringsstik:
   * **Enkelt kamera**: Tildeles automatisk til stik 1
   * **To kameraer**: Tildel hvert kamera manuelt til det korrekte stik**Tildeling af eksponeringsstik:*** **Eksponeringspin 1**: Vælg kameramodel fra rullemenuen
* **Eksponeringspin 2**: Vælg det andet kamera eller &quot;Brug ikke&quot;
* Det samme kamera kan ikke tildeles til begge pins

{% hint style="warning" %}
**Vigtigt**: Eksponerings-pins skal tildeles korrekt til de respektive kameraer. Forkert tildeling vil resultere i forkerte geolokaliseringsdata.
{% endhint %}

***

## Avancerede scenarier

### Projekter med flere kameraer

Når der behandles billeder fra flere MAPIR-kameraer i ét projekt:

1. Chloros registrerer automatisk hver kameramodel (både Survey3 og LATTICE)
2. Hvert kamera tildeles passende behandlingsprofiler, og hvert kamera får sin egen udgangsmappestruktur
3. PPK: Tildel manuelt hvert Survey3-kamera den korrekte eksponeringspin
4. Alle kameraer bruger samme eksportformat og indekser

**Eksempler**: Survey3W RGN + Survey3N OCN-rig med to kameraer, eller et LATTICE-array, der kombinerer en RGB-master med smalbåndsmoduler

### Time-lapse- eller flerdato-undersøgelser

Ved gentagne undersøgelser af det samme område over tid:

1. Opret en skabelon med dine standardindstillinger
2. Brug en ensartet opsætning af kalibreringsmål ved hver session
3. Behandl hver dato som et separat projekt
4. Brug identiske indstillinger for at opnå sammenlignelige resultater
5. Eksporter i samme format til tidsmæssig analyse

### Store datasæt

Til projekter med mange billeder (500+):

* Overvej at opdele i mindre projekter efter dato eller område
* Brug Chloros+ parallelbehandling for hurtigere resultater
* Overvej CLI eller API til batch-automatisering
* Juster det mindste rekalibreringsinterval for at reducere måldetekteringstiden

***

## Kontrol af dine indstillinger

Før du begynder behandlingen, skal du gennemgå disse vigtige indstillinger:

* [ ] Kameramodellen er korrekt genkendt i filbrowseren
* [ ] Vignettkorrektion er aktiveret
* [ ] Reflektanskalibrering er aktiveret
* [ ] For Survey3: mindst ét kalibreringsmålbillede importeret og kontrolleret; for LATTICE: et mål og/eller en `.daq`-optagelse af nedadgående stråling til stede
* [ ] De ønskede multispektrale indekser er tilføjet
* [ ] Eksportformatet passer til din arbejdsgang
* [ ] PPK-indstillingerne er konfigureret (hvis du bruger .daq med eksponeringshændelser)

***

## Næste trin

Når dine indstillinger er konfigureret:

1. **Marker kalibreringsmålbilleder** – Se [Valg af målbilleder](choosing-target-images.md)
2. **Start behandlingen** – Se [Start af behandlingen](starting-the-processing.md)
3. **Overvåg fremskridt** – Se [Overvågning af behandlingen](monitoring-the-processing.md)

For fuldstændige detaljer om alle tilgængelige indstillinger, se referencedokumentationen [Projektindstillinger](../project-settings/project-settings.md).

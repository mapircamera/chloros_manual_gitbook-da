# Projektindstillinger

Sidepanelet <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> i Chloros giver dig mulighed for at konfigurere alle aspekter af billedbehandling, detektion af kalibreringsmål, beregning af multispektrale indekser og eksportindstillinger for dit projekt. Disse indstillinger gemmes sammen med dit projekt og kan gemmes som skabeloner til genbrug på tværs af flere projekter.

## Adgang til projektindstillinger

Sådan får du adgang til projektindstillinger:

1. Åbn et projekt i Chloros
2. Klik på fanen **Projektindstillinger**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> i venstre sidepanel
3. Indstillingspanelet viser alle tilgængelige konfigurationsindstillinger organiseret efter kategori

***

## Målregistrering

Disse indstillinger styrer, hvordan Chloros registrerer og behandler kalibreringsmål i dine billeder.

### Mindste kalibreringsprøveområde (px)

* **Type**: Tal
* **Interval**: 0 til 10.000 pixels
* **Standard**: 25 pixels
* **Beskrivelse**: Indstiller det mindste område (i pixels), der kræves for at et detekteret område kan betragtes som en gyldig kalibreringsmålprøve. Mindre værdier vil detektere mindre mål, men kan øge antallet af falske positiver. Større værdier kræver større, tydeligere målområder for detektering.
* **Hvornår skal du justere**:
  * Forøg, hvis du får falske detekteringer på små billedartefakter
  * Sænk, hvis dine kalibreringsmål ser små ud på dine billeder og ikke bliver detekteret

### Mindste målgruppering (0-100)

* **Type**: Tal
* **Interval**: 0 til 100
* **Standard**: 60
* **Beskrivelse**: Styrer klyngetærsklen for gruppering af områder med lignende farver ved detektering af kalibreringsmål. Højere værdier kræver, at flere lignende farver grupperes sammen, hvilket resulterer i en mere konservativ måldetektering. Lavere værdier tillader større farvevariation inden for en målgruppe.
* **Hvornår skal du justere**:
  * Forøg, hvis kalibreringsmål opdeles i flere registreringer
  * Sænk, hvis kalibreringsmål med farvevariation ikke registreres fuldt ud

***

## Behandling

Disse indstillinger styrer, hvordan Chloros behandler og kalibrerer dine billeder.

### Vignetteringskorrektion

* **Type**: Afkrydsningsfelt
* **Standard**: Aktiveret (markeret)
* **Beskrivelse**: Anvender vignetteringskorrektion for at kompensere for mørkningen ved billedkanterne. Vignettering er et almindeligt optisk fænomen, hvor hjørnerne og kanterne af et billede fremstår mørkere end midten på grund af objektivets egenskaber.
* **Hvornår skal det deaktiveres**: Deaktiver kun, hvis din kamera/objektiv-kombination allerede har anvendt vignetteringskorrektion, eller hvis du ønsker at korrigere vignettering manuelt i efterbehandlingen.

### Reflektanskalibrering / hvidbalance

* **Type**: Afkrydsningsfelt
* **Standard**: Aktiveret (markeret)
* **Beskrivelse**: Aktiverer automatisk reflektanskalibrering ved hjælp af detekterede kalibreringsmål i dine billeder. Dette normaliserer reflektansværdierne på tværs af dit datasæt og sikrer ensartede målinger uanset lysforholdene.
* **Hvornår skal det deaktiveres**: Deaktiver kun, hvis du ønsker at behandle rå, ukalibrerede billeder, eller hvis du bruger en anden kalibreringsworkflow.

### Debayer-metode

* **Type**: Rullemenu
* **Valgmuligheder**:
  * Standard (Hurtig, Mellem kvalitet)
  * Teksturbevidst (langsom, højeste kvalitet) \[Chloros+]
* **Standard**: Standard (hurtig, middel kvalitet)
* **Beskrivelse**: Vælger den demosaicing-algoritme, der bruges til at konvertere rå Bayer-mønstersensordata til fuldfarvebilleder. Metoden &quot;Standard (hurtig, middel kvalitet)&quot; giver en optimal balance mellem behandlingshastighed og billedkvalitet. &quot;Teksturbevidst (langsom, højeste kvalitet)&quot; \[Chloros+] bruger en højkvalitets kantbevidst debayer kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al debayering-støj. Teksturbevidst-modellen kræver GPU-hukommelse (VRAM) for at køre. Vi anbefaler at bruge den, når du har &gt;4 GB VRAM til rådighed for hurtigere behandling.
* **Bemærk**: Der kan tilføjes yderligere debayer-metoder i fremtidige versioner af Chloros.

### Mindste rekalibreringsinterval

* **Type**: Tal
* **Interval**: 0 til 3.600 sekunder
* **Standard**: 0 sekunder
* **Beskrivelse**: Indstiller det mindste tidsinterval (i sekunder) mellem brug af kalibreringsmål. Når indstillet til 0, vil Chloros bruge alle detekterede kalibreringsmål. Når indstillet til en højere værdi, vil Chloros kun bruge kalibreringsmål, der er adskilt af mindst dette antal sekunder, hvilket reducerer behandlingstiden for datasæt med hyppige optagelser af kalibreringsmål.
* **Hvornår skal der justeres**:
  * Indstil til 0 for maksimal kalibreringsnøjagtighed, når lysforholdene varierer
  * Forøg (f.eks. til 60-300 sekunder) for hurtigere behandling, når lyset er konstant, og du har hyppige billeder af kalibreringsmål

### Lyssensorens tidszoneforskydning

* **Type**: Tal
* **Interval**: -12 til +12 timer
* **Standard**: 0 timer
* **Beskrivelse**: Angiver tidszoneforskydningen (i timer fra UTC) for tidsstempler på lyssensordata. Dette bruges ved behandling af PPK-datafiler (Post-Processed Kinematic) for at sikre korrekt tidssynkronisering mellem billedoptagelser og GPS-data.
* **Hvornår skal det justeres**: Indstil dette til din lokale tidszoneforskydning, hvis dine PPK-data bruger lokal tid i stedet for UTC. For eksempel:
  * Pacific Time: -8 eller -7 (afhængigt af sommertid)
  * Eastern Time: -5 eller -4 (afhængigt af sommertid)
  * Central European Time: +1 eller +2 (afhængigt af sommertid)

### Anvend PPK-korrektioner

* **Type**: Afkrydsningsfelt
* **Standard**: Deaktiveret (ikke markeret)
* **Beskrivelse**: Aktiverer brugen af Post-Processed Kinematic (PPK)-korrektioner fra MAPIR DAQ-optagere, der indeholder en GPS (GNSS). Når denne funktion er aktiveret, vil Chloros bruge alle .daq-logfiler, der indeholder eksponeringspin-data i dit projektkatalog, og anvende præcise geolokaliseringkorrektioner på dine billeder.
* **Krav**: Der skal være en .daq-logfil med eksponeringspin-poster i dit projektkatalog
* **Hvornår skal funktionen aktiveres**: Det anbefales altid at aktivere PPK-korrektion, hvis du har eksponeringsfeedback-poster i din .daq-logfil.

### Eksponeringspin 1

* **Type**: Dropdown-valg
* **Synlighed**: Kun synlig, når &quot;Anvend PPK-korrektioner&quot; er aktiveret OG der er eksponeringsdata tilgængelige for Pin 1
* **Indstillinger**:
  * Kameramodelnavne, der er registreret i projektet
  * &quot;Brug ikke&quot; – Ignorer denne eksponeringspin
* **Standard**: Vælges automatisk baseret på projektkonfigurationen
* **Beskrivelse**: Tildeler et specifikt kamera til eksponeringspin 1 til PPK-tidssynkronisering. Eksponeringspinnen registrerer det nøjagtige tidspunkt, hvor kameraets lukker udløses, hvilket er afgørende for nøjagtig PPK-geolokalisering.
* **Adfærd ved automatisk valg**:
  * Enkelt kamera + enkelt pin: Vælger automatisk kameraet
  * Enkelt kamera + to pins: Pin 1 tildeles automatisk til kameraet
  * Flere kameraer: Manuel valg kræves

### Eksponeringspin 2

* **Type**: Rullemenu
* **Synlighed**: Kun synlig, når &quot;Anvend PPK-korrektioner&quot; er aktiveret OG eksponeringsdata er tilgængelige for Pin 2
* **Indstillinger**:
  * Kameramodelnavne, der er registreret i projektet
  * &quot;Brug ikke&quot; – Ignorer denne eksponeringspin
* **Standard**: Vælges automatisk baseret på projektkonfigurationen
* **Beskrivelse**: Tildeler en bestemt kamera til eksponeringspin 2 til PPK-tidssynkronisering, når der bruges en opsætning med to kameraer.
* **Adfærd ved automatisk valg**:
  * Enkelt kamera + enkelt pin: Pin 2 indstilles automatisk til &quot;Brug ikke&quot;
  * Enkelt kamera + to pins: Pin 2 indstilles automatisk til &quot;Brug ikke&quot;
  * Flere kameraer: Manuel valg kræves
* **Bemærk**: Det samme kamera kan ikke tildeles både Pin 1 og Pin 2 samtidigt.***

## Indeks

Disse indstillinger giver dig mulighed for at konfigurere multispektrale indekser til analyse og visualisering.

### Tilføj indeks

* **Type**: Panel til konfiguration af specielle indekser
* **Beskrivelse**: Åbner et interaktivt panel, hvor du kan vælge og konfigurere multispektrale vegetationsindekser (NDVI, NDRE, EVI osv.), der skal beregnes under billedbehandlingen. Du kan tilføje flere indekser, hver med sine egne visualiseringsindstillinger.
* **Tilgængelige indekser**: Systemet indeholder over 30 foruddefinerede multispektrale indekser, herunder:
  * NDVI (Normalized Difference Vegetation Index)
  * NDRE (Normalized Difference RedEdge)
  * EVI (Enhanced Vegetation Index)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * Og mange flere (se [Multispektrale indeksformler](multispectral-index-formulas.md) for den komplette liste)
* **Funktioner**:
  * Vælg blandt foruddefinerede indeksformler
  * Konfigurer farveovergange til visualisering (LUT - Look-Up Tables)
  * Indstil tærskelværdier til analyse
  * Opret brugerdefinerede indeksformler

### Brugerdefinerede formler (Chloros+ Funktion)

* **Type**: Række af brugerdefinerede formeldefinitioner
* **Beskrivelse**: Giver dig mulighed for at oprette og gemme brugerdefinerede multispektrale indeksformler ved hjælp af båndmatematik. Brugerdefinerede formler gemmes sammen med dine projektindstillinger og kan bruges på samme måde som indbyggede indekser.
* **Sådan oprettes**:
  1. I panelet Indekskonfiguration skal du finde indstillingen for brugerdefinerede formler
  2. Definer din formel ved hjælp af båndidentifikatorer (f.eks. NIR, Red, Green, Blue)
  3. Gem formlen med et beskrivende navn
* **Formelsyntaks**: Standardmatematiske operationer understøttes, herunder:
  * Aritmetik: `+`, `-`, `*`, `/`
  * Parenteser til rækkefølge af operationer
  * Båndreferencer: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Eksport

Disse indstillinger styrer formatet og kvaliteten af eksporterede, behandlede billeder.

### Kalibreret billedformat

* **Type**: Rullemenu
* **Indstillinger**:
  * **TIFF (16-bit)** - Ukomprimeret 16-bit TIFF-format
  * **TIFF (32-bit, procent)** - 32-bit flydende komma TIFF med reflektansværdier angivet i procent
  * **PNG (8-bit)** - Komprimeret 8-bit PNG-format
  * **JPG (8-bit)** - Komprimeret 8-bit JPEG-format
* **Standard**: TIFF (16-bit)
* **Beskrivelse**: Vælger filformatet til gemning af behandlede og kalibrerede billeder.
* **Anbefalinger til format**:
  * **TIFF (16-bit)**: Anbefales til videnskabelig analyse og professionelle arbejdsgange. Bevarer maksimal datakvalitet uden komprimeringsartefakter. Bedst til multispektral analyse og viderebehandling i GIS-software.
  * **TIFF (32-bit, procent)**: Bedst til arbejdsgange, der kræver reflektansværdier som procenter (0-100 %). Tilbyder maksimal præcision til radiometriske målinger.
  * **PNG (8-bit)**: Godt til visning på nettet og generel visualisering. Mindre filstørrelser med tabsløs komprimering, men reduceret dynamisk rækkevidde.
  * **JPG (8-bit)**: Mindste filstørrelser, bedst til forhåndsvisning og visning på web. Bruger tabsgivende komprimering, som ikke er egnet til videnskabelig analyse.***

## Gem projektmal

Denne funktion giver dig mulighed for at gemme dine aktuelle projektindstillinger som en genbrugelig mal.

* **Type**: Tekstindtastning + Gem-knap
* **Beskrivelse**: Indtast et beskrivende navn til din indstillingsskabelon, og klik på gem-ikonet. Skabelonen gemmer alle dine aktuelle projektindstillinger (målregistrering, behandlingsindstillinger, indekser og eksportformat) for nem genbrug i fremtidige projekter.
* **Anvendelsestilfælde**:
  * Opret skabeloner til forskellige kamerasystemer (RGB, multispektral, NIR)
  * Gem standardkonfigurationer for specifikke afgrødetyper eller analyseworkflows
  * Del ensartede indstillinger på tværs af et team
* **Sådan bruges det**:
  1. Konfigurer alle dine ønskede projektindstillinger
  2. Indtast et skabelonnavn (f.eks. &quot;RedEdge Survey3 NDVI Standard&quot;)
  3. Klik på gem-ikonet
  4. Skabelonen kan nu indlæses, når der oprettes nye projekter

***

## Gem projektmappe

Denne indstilling angiver, hvor nye projekter gemmes som standard.

* **Type**: Visning af mappesti + Rediger-knap
* **Standard (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Standard (Linux)**: `~/.local/share/chloros/projects`
* **Beskrivelse**: Viser den aktuelle standardmappe, hvor nye Chloros-projekter oprettes. Klik på redigeringsikonet for at vælge en anden mappe.
* **Hvornår skal du ændre**:
  * Indstil til et netværksdrev til teamsamarbejde
  * Skift til et drev med mere lagerplads til store datasæt
  * Organiser projekter efter år, kunde eller projekttype i forskellige mapper
* **Bemærk**: Ændring af denne indstilling påvirker kun NYE projekter. Eksisterende projekter forbliver på deres oprindelige placeringer.***

## Bevarelse af indstillinger

Alle projektindstillinger gemmes automatisk sammen med din projektfil (`.mapir`-projektformat). Når du genåbner et projekt, gendannes alle indstillinger nøjagtigt, som du efterlod dem.

### Indstillingshierarki

Indstillingerne anvendes i følgende rækkefølge:

1. **Systemstandardindstillinger** – Indbyggede standardindstillinger defineret af Chloros
2. **Skabelonindstillinger** – Hvis du indlæser en skabelon, når du opretter et projekt
3. **Gemte projektindstillinger** – Indstillinger gemt sammen med projektfilen
4. **Manuelle justeringer** – Eventuelle ændringer, du foretager under den aktuelle session

### Indstillinger og billedbehandling

De fleste ændringer af indstillinger (især i kategorierne Behandling og Eksport) vil udløse en genbehandling af billederne for at afspejle de nye indstillinger. Nogle indstillinger er dog &quot;kun til eksport&quot; og kræver ikke øjeblikkelig genbehandling:

* Gem projektmal
* Arbejdsmappe
* Kalibreret billedformat (gælder ved eksport)

***

## Bedste praksis

1. **Start med standardindstillingerne**: Standardindstillingerne fungerer godt for de fleste MAPIR-kamerasystemer og typiske arbejdsgange.
2. **Opret skabeloner**: Når du har optimeret indstillingerne til en bestemt arbejdsgang eller et bestemt kamera, skal du gemme dem som en skabelon for at sikre ensartethed på tværs af projekter.
3. **Test før fuld behandling**: Når du eksperimenterer med nye indstillinger, skal du teste dem på en lille delmængde af billeder, før du behandler hele dit datasæt.
4. **Dokumenter dine indstillinger**: Brug beskrivende skabelonnavne, der angiver kamerasystemet, behandlingstypen og den tilsigtede anvendelse (f.eks. &quot;Survey3\_RGB\_NDVI\_Agriculture&quot;).
5. **Valg af eksportformat**: Vælg dit eksportformat ud fra din endelige anvendelse:
   * Videnskabelig analyse → TIFF (16-bit eller 32-bit)
   * GIS-behandling → TIFF (16-bit)
   * Hurtig visualisering → PNG (8-bit)
   * Deling på nettet → JPG (8-bit)

***

For mere information om multispektrale indekser i Chloros, se siden [Formler for multispektrale indekser](multispectral-index-formulas.md).

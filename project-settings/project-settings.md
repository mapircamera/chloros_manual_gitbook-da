# Projektindstillinger

Projektindstillinger <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> i Chloros giver dig mulighed for at konfigurere alle aspekter af billedbehandling, kalibreringsmåldetektering, multispektrale indeksberegninger og eksportindstillinger for dit projekt. Disse indstillinger gemmes sammen med dit projekt og kan gemmes som skabeloner til genbrug på tværs af flere projekter.

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
* **Beskrivelse**: Indstiller det mindste område (i pixels), der kræves for, at et detekteret område kan betragtes som en gyldig kalibreringsmålsprøve. Mindre værdier vil detektere mindre mål, men kan øge antallet af falske positiver. Større værdier kræver større, klarere målområder for detektion.
* **Hvornår skal du justere**:
  * Forøg, hvis du får falske detektioner på små billedartefakter.
  * Reducer, hvis dine kalibreringsmål vises små i dine billeder og ikke detekteres.

### Minimum målklyngedannelse (0-100)

* **Type**: Tal
* **Interval**: 0 til 100
* **Standard**: 60
* **Beskrivelse**: Styrer klyngetærsklen for gruppering af lignende farvede områder, når der detekteres kalibreringsmål. Højere værdier kræver, at mere ens farver grupperes sammen, hvilket resulterer i en mere konservativ måldetektering. Lavere værdier tillader større farvevariation inden for en målgruppe.
* **Hvornår skal det justeres**:
  * Forøg, hvis kalibreringsmål opdeles i flere detektioner.
  * Reducer, hvis kalibreringsmål med farvevariation ikke detekteres fuldt ud.

***

## Behandling

Disse indstillinger styrer, hvordan Chloros behandler og kalibrerer dine billeder.

### Vignettekorrektion

* **Type**: Afkrydsningsfelt
* **Standard**: Aktiveret (afkrydset)
* **Beskrivelse**: Anvender vignettekorrektion for at kompensere for mørkfarvning af linsen i billedets kanter. Vignettering er et almindeligt optisk fænomen, hvor hjørnerne og kanterne af et billede fremstår mørkere end midten på grund af linsens egenskaber.
* **Hvornår skal det deaktiveres**: Deaktiver kun, hvis din kamera/objektiv-kombination allerede har anvendt vignettekorrektion, eller hvis du manuelt ønsker at korrigere vignettering i efterbehandlingen.

### Reflektanskalibrering / hvidbalance

* **Type**: Afkrydsningsfelt
* **Standard**: Aktiveret (markeret)
* **Beskrivelse**: Aktiverer automatisk reflektanskalibrering ved hjælp af detekterede kalibreringsmål i dine billeder. Dette normaliserer reflektansværdierne på tværs af dit datasæt og sikrer konsistente målinger uanset lysforholdene.
* **Hvornår skal du deaktivere**: Deaktiver kun, hvis du vil behandle rå, ukalibrerede billeder, eller hvis du bruger en anden kalibreringsworkflow.

### Debayer-metode

* **Type**: Dropdown-valg
* **Indstillinger**:
  * Høj kvalitet (hurtigere) – Den eneste tilgængelige indstilling i øjeblikket
* **Standard**: Høj kvalitet (hurtigere)
* **Beskrivelse**: Vælger den demosaicing-algoritme, der bruges til at konvertere rå Bayer-mønstersensordata til fuldfarvebilleder. Metoden &quot;Høj kvalitet (hurtigere)&quot; giver en optimal balance mellem behandlingshastighed og billedkvalitet.
* **Bemærk**: Der kan tilføjes yderligere debayer-metoder i fremtidige versioner af Chloros.

### Minimumskalibreringsinterval

* **Type**: Tal
* **Interval**: 0 til 3.600 sekunder
* **Standard**: 0 sekunder
* **Beskrivelse**: Indstiller det minimale tidsinterval (i sekunder) mellem brug af kalibreringsmål. Når det er indstillet til 0, vil Chloros bruge alle registrerede kalibreringsmål. Når det er indstillet til en højere værdi, vil Chloros kun bruge kalibreringsmål, der er adskilt med mindst dette antal sekunder, hvilket reducerer behandlingstiden for datasæt med hyppige kalibreringsmålsoptagelser.
* **Hvornår skal det justeres**:
  * Indstil til 0 for maksimal kalibreringsnøjagtighed, når lysforholdene varierer.
  * Øg (f.eks. til 60-300 sekunder) for hurtigere behandling, når lyset er ensartet, og du har hyppige kalibreringsmålebilleder.

### Lyssensorens tidszoneforskydning

* **Type**: Tal
* **Interval**: -12 til +12 timer
* **Standard**: 0 timer
* **Beskrivelse**: Angiver tidszoneforskydningen (i timer fra UTC) for lyssensordatas tidsstempler. Dette bruges ved behandling af PPK-datafiler (Post-Processed Kinematic) for at sikre korrekt tidssynkronisering mellem billedoptagelser og GPS-data.
* **Hvornår skal justeres**: Indstil dette til din lokale tidszoneforskydning, hvis dine PPK-data bruger lokal tid i stedet for UTC. For eksempel:
  * Pacific Time: -8 eller -7 (afhængigt af sommertid)
  * Eastern Time: -5 eller -4 (afhængigt af sommertid)
  * Central European Time: +1 eller +2 (afhængigt af sommertid)

### Anvend PPK-korrektioner

* **Type**: Afkrydsningsfelt
* **Standard**: Deaktiveret (ikke markeret)
* **Beskrivelse**: Aktiverer brugen af efterbehandlede kinematiske (PPK) korrektioner fra MAPIR DAQ-optagere, der indeholder en GPS (GNSS). Når denne funktion er aktiveret, vil Chloros bruge alle .daq-logfiler, der indeholder eksponeringspin-data i dit projektbibliotek, og anvende præcise geolokaliseringkorrektioner på dine billeder.
* **Krav**: .daq-logfil med eksponeringspin-poster skal være til stede i dit projektbibliotek
* **Hvornår skal det aktiveres**: Det anbefales altid at aktivere PPK-korrektion, hvis du har eksponeringsfeedback-poster i din .daq-logfil.

### Eksponeringspin 1

* **Type**: Dropdown-valg
* **Synlighed**: Kun synlig, når &quot;Anvend PPK-korrektioner&quot; er aktiveret, OG eksponeringsdata er tilgængelige for pin 1
* **Indstillinger**:
  * Kameramodelnavne, der er registreret i projektet
  * &quot;Brug ikke&quot; – Ignorer denne eksponeringspin
* **Standard**: Vælges automatisk baseret på projektkonfigurationen
* **Beskrivelse**: Tildeler et specifikt kamera til eksponeringspin 1 til PPK-tidssynkronisering. Eksponeringspinnen registrerer det nøjagtige tidspunkt, hvor kameraets lukker udløses, hvilket er afgørende for nøjagtig PPK-geolokalisering.
* **Automatisk valg**:
  * Enkelt kamera + enkelt pin: Vælger automatisk kameraet
  * Enkelt kamera + to pins: Pin 1 tildeles automatisk til kameraet
  * Flere kameraer: Manuel valg kræves

### Eksponeringspin 2

* **Type**: Dropdown-valg
* **Synlighed**: Kun synlig, når &quot;Anvend PPK-korrektioner&quot; er aktiveret OG eksponeringsdata er tilgængelige for pin 2
* **Valgmuligheder**:
  * Kameramodelnavne, der er registreret i projektet
  * &quot;Brug ikke&quot; – Ignorer denne eksponeringspin
* **Standard**: Vælges automatisk baseret på projektkonfigurationen
* **Beskrivelse**: Tildeler et specifikt kamera til eksponeringspin 2 til PPK-tidssynkronisering, når der bruges en opsætning med to kameraer.
* **Automatisk valg**:
  * Enkelt kamera + enkelt pin: Pin 2 indstilles automatisk til &quot;Brug ikke&quot;
  * Enkelt kamera + to pins: Pin 2 indstilles automatisk til &quot;Brug ikke&quot;
  * Flere kameraer: Manuel valg kræves
* **Bemærk**: Det samme kamera kan ikke tildeles både Pin 1 og Pin 2 samtidigt.

***

## Indeks

Disse indstillinger giver dig mulighed for at konfigurere multispektrale indekser til analyse og visualisering.

### Tilføj indeks

* **Type**: Specielt indekskonfigurationspanel
* **Beskrivelse**: Åbner et interaktivt panel, hvor du kan vælge og konfigurere multispektrale vegetationsindekser (NDVI, NDRE, EVI osv.) til beregning under billedbehandling. Du kan tilføje flere indekser, hver med sine egne visualiseringsindstillinger.
* **Tilgængelige indekser**: Systemet indeholder mere end 30 foruddefinerede multispektrale indekser, herunder:
  * NDVI (Normaliseret forskel i vegetationsindeks)
  * NDRE (Normaliseret forskel RedEdge)
  * EVI (Forbedret vegetationsindeks)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * Og mange flere (se [Multispektrale indeksformler](multispectral-index-formulas.md) for en komplet liste)
* **Funktioner**:
  * Vælg mellem foruddefinerede indeksformler
  * Konfigurer visualiseringsfarveovergange (LUT - Look-Up Tables)
  * Indstil tærskelværdier for analyse
  * Opret brugerdefinerede indeksformler

### Brugerdefinerede formler (Chloros+ funktion)

* **Type**: Array af brugerdefinerede formeldefinitioner
* **Beskrivelse**: Giver dig mulighed for at oprette og gemme brugerdefinerede multispektrale indeksformler ved hjælp af båndmatematik. Brugerdefinerede formler gemmes sammen med dine projektindstillinger og kan bruges på samme måde som indbyggede indekser.
* **Sådan oprettes**:
  1. I panelet Indeks konfiguration skal du finde indstillingen for brugerdefinerede formler.
  2. Definer din formel ved hjælp af båndidentifikatorer (f.eks. NIR, Red, Green, Blue).
  3. Gem formlen med et beskrivende navn.
* **Formelsyntaks**: Standardmatematiske operationer understøttes, herunder:
  * Aritmetik: `+`, `-`, `*`, `/`
  * Parenteser til rækkefølgen af operationer
  * Båndreferencer: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Eksport

Disse indstillinger styrer formatet og kvaliteten af eksporterede behandlede billeder.

### Kalibreret billedformat

* **Type**: Dropdown-valg
* **Indstillinger**:
  * **TIFF (16-bit)** - Ukomprimeret 16-bit TIFF-format
  * **TIFF (32-bit, procent)** – 32-bit flydende komma TIFF med reflektansværdier som procenter
  * **PNG (8-bit)** - Komprimeret 8-bit PNG-format
  * **JPG (8-bit)** - Komprimeret 8-bit JPEG-format
* **Standard**: TIFF (16-bit)
* **Beskrivelse**: Vælger filformatet til gemning af behandlede og kalibrerede billeder.
* **Format anbefalinger**:
  * **TIFF (16-bit)**: Anbefales til videnskabelig analyse og professionelle arbejdsgange. Bevarer maksimal datakvalitet uden komprimeringsartefakter. Bedst til multispektral analyse og videre behandling i GIS-software.
  * **TIFF (32-bit, procent)**: Bedst til arbejdsgange, der kræver reflektansværdier som procenter (0-100 %). Tilbyder maksimal præcision til radiometriske målinger.
  * **PNG (8-bit)**: God til visning på internettet og generel visualisering. Mindre filstørrelser med tabsløs komprimering, men reduceret dynamisk rækkevidde.
  * **JPG (8-bit)**: Mindste filstørrelser, bedst til forhåndsvisning og kun til visning på internettet. Bruger tabsgivende komprimering, som ikke er egnet til videnskabelig analyse.

***

## Gem projektmal

Denne funktion giver dig mulighed for at gemme dine aktuelle projektindstillinger som en genanvendelig mal.

* **Type**: Tekstindtastning + Gem-knap
* **Beskrivelse**: Indtast et beskrivende navn til din indstillingsmal, og klik på gem-ikonet. Malen gemmer alle dine aktuelle projektindstillinger (måldetektering, behandlingsindstillinger, indekser og eksportformat), så de nemt kan genbruges i fremtidige projekter.
* **Anvendelsestilfælde**:
  * Opret skabeloner til forskellige kamerasystemer (RGB, multispektral, NIR)
  * Gem standardkonfigurationer til specifikke afgrødetyper eller analyseworkflows
  * Del ensartede indstillinger på tværs af et team
* **Sådan bruges det**:
  1. Konfigurer alle dine ønskede projektindstillinger.
  2. Indtast et skabelonnavn (f.eks. &quot;RedEdge Survey3 NDVI Standard&quot;).
  3. Klik på ikonet for gem.
  4. Skabelonen kan nu indlæses, når du opretter nye projekter.

***

## Gem projektmappe

Denne indstilling angiver, hvor nye projekter gemmes som standard.

* **Type**: Visning af sti til mappe + knap Rediger
* **Standard**: `C:\Users\[Username]\Chloros Projects`
* **Beskrivelse**: Viser den aktuelle standardmappe, hvor nye Chloros-projekter oprettes. Klik på ikonet Rediger for at vælge en anden mappe.
* **Hvornår skal du ændre det**:
  * Indstil til et netværksdrev for teamsamarbejde.
  * Skift til et drev med mere lagerplads til store datasæt.
  * Organiser projekter efter år, klient eller projekttype i forskellige mapper.
* **Bemærk**: Ændring af denne indstilling påvirker kun NYE projekter. Eksisterende projekter forbliver på deres oprindelige placeringer.

***

## Indstillingspersistens

Alle projektindstillinger gemmes automatisk sammen med din projektfil (`.mapir`-projektformat). Når du genåbner et projekt, gendannes alle indstillinger nøjagtigt, som du efterlod dem.

### Indstillingshierarki

Indstillingerne anvendes i følgende rækkefølge:

1. **Systemstandarder** – Indbyggede standarder defineret af Chloros
2. **Skabelonindstillinger** – Hvis du indlæser en skabelon, når du opretter et projekt
3. **Gemte projektindstillinger** – Indstillinger gemt sammen med projektfilen
4. **Manuelle justeringer** – Alle ændringer, du foretager under den aktuelle session

### Indstillinger og billedbehandling

De fleste ændringer af indstillinger (især i kategorierne Behandling og Eksport) vil udløse en genbehandling af billederne for at afspejle de nye indstillinger. Nogle indstillinger er dog &quot;kun til eksport&quot; og kræver ikke øjeblikkelig genbehandling:

* Gem projektmal
* Arbejdsmappe
* Kalibreret billedformat (gælder ved eksport)

***

## Bedste praksis

1. **Start med standardindstillinger**: Standardindstillingerne fungerer godt for de fleste MAPIR-kamerasystemer og typiske arbejdsgange.
2. **Opret skabeloner**: Når du har optimeret indstillingerne for en bestemt arbejdsgang eller et bestemt kamera, skal du gemme dem som en skabelon for at sikre konsistens på tværs af projekter.
3. **Test før fuld behandling**: Når du eksperimenterer med nye indstillinger, skal du teste på en lille delmængde af billeder, før du behandler hele datasættet.
4. **Dokumenter dine indstillinger**: Brug beskrivende skabelonnavne, der angiver kamerasystemet, behandlingstypen og den tilsigtede anvendelse (f.eks. &quot;Survey3\_RGB\_NDVI\_Agriculture&quot;).
5. **Valg af eksportformat**: Vælg dit eksportformat ud fra din endelige anvendelse:
   * Videnskabelig analyse → TIFF (16-bit eller 32-bit)
   * GIS-behandling → TIFF (16-bit)
   * Hurtig visualisering → PNG (8-bit)
   * Deling på internettet → JPG (8-bit)

***

For mere information om multispektrale indekser i Chloros, se siden [Multispektrale indeksformler](multispectral-index-formulas.md).

# Projektindstillinger

Sidepanelet »Projektindstillinger« <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

i Chloros giver dig mulighed for at konfigurere alle aspekter af billedbehandling, detektering af kalibreringsmål, beregning af multispektrale indekser samt eksportindstillinger for dit projekt. Disse indstillinger gemmes sammen med dit projekt og kan gemmes som skabeloner til genbrug på tværs af flere projekter.

## Sådan åbner du projektindstillingerne

Sådan åbner du projektindstillingerne:

1. Åbn et projekt i Chloros


2. Klik på fanen **Projektindstillinger** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

i venstre sidepanel
3. Indstillingspanelet viser alle tilgængelige konfigurationsmuligheder sorteret efter kategori

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->



{% hint style="info" %}
**Indstillinger, der afhænger af andre indstillinger, er nedtonet.** Når en overordnet indstilling gør en anden indstilling umulig (for eksempel gør det umuligt at vælge *Eksporter reflektans*, hvis man fjerner markeringen fra *Reflektanskalibrering / hvidbalance*), deaktiveres den afhængige indstilling, og dens værktøjstip angiver navnet på den indstilling, der skal ændres.
{% endhint %}

***

## Skærm

### Opløsning af billedminiaturer

* **Type**: Rullemenu
* **Valgmuligheder**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Standard**: Standard (512 px)
* **Beskrivelse**: Opløsning (den længste kant, i pixels), hvormed miniaturerne i billedgitteret gengives. Højere værdier ser skarpere ud, når der zoomes ind, men indlæses langsommere og bruger mere hukommelse. Fuld opløsning svarer til den originale billedstørrelse.
* **Bemærk**: Kun til visning — dette påvirker aldrig behandlingen eller eksporterede filer.***

## Målregistrering

Disse indstillinger styrer, hvordan Chloros registrerer og behandler kalibreringsmål i dine billeder. Begge er kun aktive, mens **Refleksionskalibrering / hvidbalance** er aktiveret (ellers er de nedtonede, da målregistrering i så fald springes helt over).

### Mindste kalibreringsprøveområde (px)

* **Type**: Tal
* **Interval**: 0 til 10.000 pixels
* **Standard**: 25 pixels
* **Beskrivelse**: Indstiller det mindste areal (i pixels), der kræves for at et detekteret område kan betragtes som en gyldig kalibreringsmålprøve. Mindre værdier vil detektere mindre mål, men kan øge antallet af falske positiver. Større værdier kræver større, tydeligere målområder for detektering.
* **Hvornår skal du justere**:
  * Forøg værdien, hvis du får falske registreringer på små billedartefakter
  * Sænk værdien, hvis dine kalibreringsmål fremstår små på dine billeder og ikke bliver registreret

### Mindste målgruppering (0-100)

* **Type**: Tal
* **Interval**: 0 til 100
* **Standard**: 60
* **Beskrivelse**: Styrer klyngetærsklen for gruppering af områder med lignende farver ved detektering af kalibreringsmål. Højere værdier kræver, at flere lignende farver grupperes sammen, hvilket resulterer i en mere konservativ måldetektering. Lavere værdier tillader større farvevariation inden for en målgruppe.
* **Hvornår skal du justere**:
  * Forøg værdien, hvis kalibreringsmål opdeles i flere registreringer
  * Sænk værdien, hvis kalibreringsmål med farvevariation ikke registreres fuldt ud

***

## Behandling

Disse indstillinger styrer, hvordan Chloros behandler og kalibrerer dine billeder.

### Vignetteringskorrektion

* **Type**: Afkrydsningsfelt
* **Standard**: Aktiveret (afkrydset)
* **Beskrivelse**: Anvender vignetteringskorrektion for at kompensere for objektivets mørkning i billedets kanter. Vignettering er et almindeligt optisk fænomen, hvor hjørnerne og kanterne af et billede fremstår mørkere end midten på grund af objektivets egenskaber.
* **Bivirkning**: Denne indstilling vælger også, hvilket *ukalibreret reserveprodukt* en kørsel skriver (se nedenfor).

### Reflektanskalibrering / hvidbalance

* **Type**: Afkrydsningsfelt
* **Standard**: Aktiveret (markeret)
* **Beskrivelse**: Aktiverer reflektanskalibrering — baseret på detekterede kalibreringsmål inden for billedrammen og/eller nedadgående lysdata fra DAQ-lyssensoren, afhængigt af kameraet og de tilgængelige data. Dette normaliserer reflektansværdierne på tværs af dit datasæt og sikrer ensartede målinger uanset lysforholdene.
* **Når deaktiveret**: Målregistrering springes helt over, og**der kan ikke genereres noget reflektansprodukt af noget kamera** — hverken Survey3-måldrevet eller LATTICE DAQ-drevet. De afhængige indstillinger (*Eksporter reflektans*, *Minimalt rekalibreringsinterval* og tærskelværdierne for målregistrering) er nedtonede.

### Ikke-kalibrerede reserveprodukter: Eksporter sensorrespons / Eksporter vignettekorrigeret

* **Type**: To afkrydsningsfelter
* **Standardindstillinger**: Begge aktiveret (afkrydset)
* **Beskrivelse**: Når et billede ikke kan reflektanskalibreres (der blev ikke fundet et kalibreringsmål, eller reflektanskalibrering er slået fra), gemmes det i stedet som et *ukalibreret reserveprodukt*. **Der findes nøjagtigt ét af de to reserveprodukter pr. kørsel for hver kameramodel**, valgt via *Vignettkorrektion*-kontakten:
  * Vignettekorrektion **til**→ `Vignette_Corrected_Images/` (styret af**Eksporter vignettekorrigeret**)
  * Vignettekorrektion **fra**→ `Sensor_Response_Images/` (styret af**Eksporter sensorrespons**)
* Den reserve, der ikke er aktiv, er nedtonet. Hvis du fjerner markeringen fra den aktive, forhindres den pågældende fil i at blive skrevet overhovedet.

### LATTICE-eksportprodukter

For projekter, der indeholder LATTICE-optagelser, fordeles hvert importeret LATTICE-billede til alle aktiverede **og relevante**produkter i et enkelt behandlingsforløb. Fire afkrydsningsfelter styrer fordelingen (alle er som standard**aktiveret**):

| Indstilling | Output-mappe | Hvad der eksporteres |
| --- | --- | --- |
| **Eksporter debayered** | `Debayered_Images/` | Det lineært debayered billede: kalibreringspakkens mørkeforskydning fjernet, derefter en kantbevidst demosaikering uden linearitetskorrektion, flat field eller unmix — den samme fil, som en optagelse gemmer på sit debayered-niveau. Bevares i sensorens oprindelige orientering (registreret i XMP). Gælder for RGB- og multispektrale kameraer. |
| **Eksporter forhåndsvisning** | `Preview_Images/` | Forhåndsvisningen på skærmen. RGB = hvidbalance (DAQ-lysart, hvis tilgængelig, ellers gråskala) + gamma; multispektral = falskfarveudstrækning. |
| **Eksport af strålingsintensitet** | `Radiance_Images/` | Float32 spektral strålingsintensitet i W/m²/sr/nm. Kun multispektral (M3C/M3M) — gælder ikke for RGB-masterfiler. Skrives altid som 32-bit TIFF uanset indstillingen for *Kalibreret billedformat*. |
| **Eksport af reflektans**| `Reflectance_Calibrated_Images/` | Uint16-reflektans, skaleret således at**32768 = reflektans 1,0** (stemplet som XMP `Chloros:PixelScale`). Gælder kun multispektrale billeder og skrives, når en matchende `.daq`-nedstrålingsregistrering (eller et QA-godkendt mål inden for billedrammen) dækker billedrammen. |

* RGB-masterkameraer udsender debayered + preview; radiance/reflektans springes over for disse, da de ikke er relevante.
* Bitdybden for debayered/preview følger indstillingen *Kalibreret billedformat*; radiance er altid float32.
* Survey3-behandlingen påvirkes ikke af disse fire indstillinger.
* Radiance- og reflektansprodukter fra multispektrale M3C-kameraer kan også skrive en **clamp-maske** pr. pixel i en `clamp_masks/`-mappe ved siden af produktmapperne, hvorved det registreres, hvor båndsepareringen har klippet en negativ værdi ned til nul. Se [Output Image Formats](../output-image-formats.md#clamp-masks).

De samme fire indstillinger findes i headless-versionen som `chloros-cli process --debayered / --preview / --radiance / --reflectance`. De erstattede det gamle CLI-flag `--radiometric-output`, som ikke længere findes.

{% hint style="warning" %}
**Hvis alle relevante produkter slås fra, mislykkes kørslen.** Fra og med version 1.2.0 rapporterer en behandlingskørsel, der blev anmodet om produkter, men som ikke skrev noget billedprodukt, en fejl, og CLI'en afsluttes med en værdi forskellig fra nul i stedet for at rapportere en lydløs succes. Loggen angiver det produkt, der ikke kunne skrives, samt årsagen hertil. En bevidst kørsel, der kun omfatter metadata (intet anmodet om) betragtes stadig som en succes.
{% endhint %}

### Reflektanskilde (projektindstilling, indstilles via CLI'en)

Projektet gemmer også, hvilken **reflektansreference** LATTICE-reflektansproduktet bruger. Der er ingen dedikeret kontrol i indstillingspanelet; værdien gemmes i projektkonfigurationen som `Processing → "Target reflectance source"` og indstilles med `chloros-cli process --reflectance-source {auto,target,daq}` (SDK's `configure()` har ikke noget nøgleord for dette):

* **`auto`** (standard): et QA-godkendt kalibreringsmål inden for billedrammen bliver den absolutte reference, mens der falder tilbage til DAQ-nedstrålingsdelingen (ρ = πL/E), når der ikke er noget mål til stede, eller QA mislykkes. Et mål, der består QA med en fejlmarkering (et panel, der er faldet ud, eller et bånd, der er ude af drift), foretrækkes ikke automatisk: kæderne mål, mål+DAQ og kun DAQ afprøves i rækkefølge efter deres målte score på de udeladte paneler.
* **`target`**: streng måldrevet reflektans — ingen DAQ-erstatning.
* **`daq`**: DAQ-autoritativ reflektans; mål inden for rammen bruges ikke som reference.

Den gemte værdi matches uden skelnen mellem store og små bogstaver, og nogle få stavemåder accepteres som aliaser: `target`, `target_image`, `empirical` og `empirical_line` betyder alle **mål**; `daq`, `dls`, `light_sensor` og `sensor` betyder alle**daq**. Alt andet — herunder en manglende nøgle — oversættes til**auto**.**Målte** målscanninger pr. enhed slås op ved hjælp af målenhedens serienummer/QR, som f.eks. `<serial>.csv`, på tre steder: i det bibliotek, der er angivet med `--target-reflectance-dir` (gemt som `Processing → "Target reflectance dir"`), i projektets egen `target_reflectance/`-mappe, og stien i miljøvariablen `CHLOROS_TARGET_REFLECTANCE_DIR`. Hvis der ikke findes nogen målt scanning for den pågældende enhed, anvendes den nominelle, offentliggjorte kurve for målmodellen i stedet.

### Debayer-metode

* **Type**: Valg fra rullemenu
* **Indstillinger**:
  * Standard (hurtig, middel kvalitet)
  * Teksturbevidst (langsom, højeste kvalitet) \[Chloros+]
* **Standard**: Standard (hurtig, middel kvalitet)
* **Beskrivelse**: Vælger den demosaicing-algoritme, der bruges til at konvertere rå sensordata fra Bayer-mønsteret til fuldfarvebilleder. Metoden »Standard (hurtig, middel kvalitet)« giver en optimal balance mellem behandlingshastighed og billedkvalitet. Metoden »Teksturbevidst (langsom, højeste kvalitet)« \[Chloros+] anvender en højkvalitets kantbevidst debayer kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al debayering-støj. Texture Aware-modellen kræver GPU-hukommelse (VRAM) for at køre. Vi anbefaler at bruge den, når du har >4 GB VRAM til rådighed for hurtigere behandling.
* **Når rækken overhovedet er en rullemenu**: rullemenuen med to valgmuligheder vises kun, når**begge** betingelser er opfyldt — du er logget ind med et gyldigt Chloros+-abonnement, **og** projektet indeholder ingen LATTICE-optagelser. Ellers vises rækken som almindelig tekst med teksten `Standard (Fast, Medium Quality)` uden valgmuligheder.
* **LATTICE-bemærkning**: Der findes ingen LATTICE-trænet Texture Aware-model, og pipelinen tvinger standard-demosaic for LATTICE-rammer uanset den gemte værdi. Hvis du tilføjer en LATTICE-mappe til et projekt, hvor Texture Aware allerede var valgt, omskriver Chloros indstillingen tilbage til Standard i stedet for at efterlade en forældet værdi i `project.json`.

### Mindste interval for rekalibrering

* **Type**: Tal
* **Interval**: 0 til 3.600 sekunder
* **Standard**: 0 sekunder
* **Beskrivelse**: Indstiller det mindste tidsinterval (i sekunder) mellem brug af kalibreringsmål. Når værdien er indstillet til 0, vil Chloros bruge alle dekalibreringsmål, der registreres. Når værdien er indstillet til et højere tal, vil Chloros kun bruge kalibreringsmål, der er adskilt af mindst dette antal sekunder, hvilket reducerer behandlingstiden for datasæt med hyppige registreringer af kalibreringsmål.
* **Hvornår skal indstillingen justeres**:
  * Indstil til 0 for maksimal kalibreringsnøjagtighed, når lysforholdene varierer
  * Forøg værdien (f.eks. til 60–300 sekunder) for hurtigere behandling, når lysforholdene er konstante, og der er hyppige billeder af kalibreringsmål

### Tidszoneforskydning for lyssensor

* **Type**: Tal
* **Interval**: -12 til +12 timer
* **Standard**: 0 timer
* **Beskrivelse**: Angiver tidszoneforskydningen (i timer fra UTC) for tidsstempler på lyssensordata, der bruges ved sammenkobling af lyssensorlogfiler med billedoptagelsestidspunkter. Nyere `.daq`-optagelser har deres egen tidszoneangivelse, så dette er hovedsageligt nødvendigt for ældre logfiler, der er optaget i lokal tid.

### Anvend PPK-korrektioner

* **Type**: Afkrydsningsfelt
* **Standard**: Deaktiveret (afkrydset)
* **Beskrivelse**: Aktiverer brugen af PPK-korrektioner (Post-Processed Kinematic) fra MAPIR DAQ-optagere, der indeholder en GPS (GNSS). Når denne funktion er aktiveret, vil Chloros anvende alle .daq-logfiler, der indeholder eksponeringspin-data i dit projektkatalog, og anvende præcise geolokaliseringkorrektioner på dine billeder.
* **Krav**: Der skal findes en .daq-logfil med eksponeringspin-poster i dit projektmappe
* **Hvornår skal det aktiveres**: Det anbefales altid at aktivere PPK-korrektion, hvis du har eksponeringsfeedback-poster i din .daq-logfil.

### Eksponeringspin 1

* **Type**: Valg fra rullemenu
* **Synlighed**: Kun synlig, når "Anvend PPK-korrektioner" er aktiveret OG der er eksponeringsdata tilgængelige for Pin 1
* **Indstillinger**:
  * Kameramodelnavne, der er registreret i projektet
  * "Brug ikke" – Ignorer denne eksponeringspin
* **Standard**: Vælges automatisk baseret på projektkonfigurationen
* **Beskrivelse**: Tildeler et specifikt kamera til eksponeringspin 1 til PPK-tidssynkronisering. Eksponeringspinnen registrerer det nøjagtige tidspunkt, hvor kameraets lukker udløses, hvilket er afgørende for nøjagtig PPK-geolokalisering.
* **Adfærd ved automatisk valg**:
  * Ét kamera + én pin: Vælger automatisk kameraet
  * Ét kamera + to pins: Pin 1 tildeles automatisk til kameraet
  * Flere kameraer: Manuel valg kræves

### Eksponeringspin 2

* **Type**: Valg fra rullemenu
* **Synlighed**: Kun synlig, når "Anvend PPK-korrektioner" er aktiveret OG der foreligger eksponeringsdata for pin 2
* **Indstillinger**:
  * Kameramodelnavne, der er registreret i projektet
  * "Brug ikke" – Ignorer denne eksponeringspin
* **Standard**: Vælges automatisk baseret på projektkonfigurationen
* **Beskrivelse**: Tildeler en bestemt kamera til eksponeringspin 2 til PPK-tidssynkronisering, når der anvendes en opsætning med to kameraer.
* **Adfærd ved automatisk valg**:
  * Ét kamera + én pin: Pin 2 indstilles automatisk til "Brug ikke"
  * Ét kamera + to pins: Pin 2 indstilles automatisk til "Brug ikke"
  * Flere kameraer: Manuel valg kræves
* **Bemærk**: Det samme kamera kan ikke tildeles både pin 1 og pin 2 samtidigt.***

## DAQ-lyssensor

Dette afsnit vises under Projektindstillinger og viser alle DAQ-downwelling-filer i projektet — `.daq`-optagelser og DAQ-M `.csv`-nedstrålingslogfiler. Optagelser foretaget under fanen Lyssensorer føjes automatisk til det åbne projekt.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->



Hver række viser filen, sensormodellen og den diffusorhætte-korrektion, der faktisk gælder for den pågældende fil. Over rækkerne findes en enkelt projektdækkende kontrol:

### Hætte-overstyring (alle filer)

* **Type**: Valg fra rullemenu
* **Indstillinger**: `Auto` samt de kappe-korrektionsprofiler, der gælder for de sensortyper, der findes i projektet
* **Standard**: Auto
* **Gemmes som**: `Processing → "DAQ cap id"` (standard `auto`)
* **Beskrivelse**: `Auto` bruger den registrerede kappe for hver fil (Sunshine-kappen antages, hvis der ikke er registreret noget, men DAQ-U, DAQ-M og DAQ-E leveres uden kappe, og Chloros kan ikke se, hvad der er monteret, så behandlingen viser en advarsel om en antaget kappe: vælg den kappe, der var monteret). Valg af en specifik kappe tilsidesætter **alle** nedadgående filer i projektet undtagen en DAQ-E-S's, hvis diffusor er en del af dens kalibrering; et projekt bestående udelukkende af DAQ-E-S viser ingen rullemenu: råoptagelser korrigeres med den, og optagelser, der allerede har en kappe, omrefereres (den indspillede korrektion fortrydes, og den valgte anvendes).
* **Vigtigt**: Den valgte kappe skal matche den kappe, der fysisk var monteret under optagelsen. Hverken sensoren eller softwaren kan registrere den fysiske kappe — et uoverensstemmende kappe-ID korrigerer spektrene forkert.

Der er bevidst **én** projektdækkende indstilling i stedet for rullemenuer pr. fil: indstillingen gælder for alle nedadrettede kilder i projektet.***

## Array-justering

Dette afsnit vises **kun**, når mindst ét billede i projektet indeholder den modul-til-modul-justeringstransformation, som LATTICE-arrayer mærker ved optagelsestidspunktet (XMP `Chloros:Alignment*`-tags). Det viser, hvor mange billeder der har justeringstags, hvilket kamera der er referencen (`REF`-badge), samt en tabel med antallet af billeder pr. kamera.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Anvend array-justering

* **Type**: Afkrydsningsfelt
* **Standard**: Aktiveret (afkrydset)
* **Gemmes som**: `Processing → "Array alignment"`
* **Beskrivelse**: Forvrænger hvert behandlet produkt (debayered / preview / radiance / reflectance / index) til arrayets fælles referencegeometri ved hjælp af den transformation, der blev stemplet ved optagelsen. Fra = eksport i den oprindelige geometri pr. sensor.

### Beskær til fælles overlapning

* **Type**: Afkrydsningsfelt (kun aktivt, når *Anvend array-justering* er aktiveret)
* **Standard**: Aktiveret (markeret)
* **Gemmes som**: `Processing → "Array alignment crop"`
* **Beskrivelse**: Beskærer justerede eksporter til det område, som alle kameramoduler deler, så hvert bånd har samme dækningsområde. Fra bevarer det fulde sensorområde (sort udfyldning uden for kilden).

### Resampling

* **Type**: Rullemenu (kun aktiv, når *Anvend array-justering* er aktiveret)
* **Indstillinger**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Standard**: Bilineær
* **Gemmes som**: `Processing → "Array alignment interpolation"`
* **Beskrivelse**: Interpolation, der anvendes af justeringsforvrængningen. *Nærmeste* bevarer de nøjagtige kildeværdier (ingen blanding mellem pixels) til streng radiometrisk analyse; *Bilineær* er bedst til kortlægning og visuel brug.

De samme tre indstillinger findes uden foranstav som `chloros-cli process --array-alignment`, `--array-alignment-crop` og `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Indeks

Disse indstillinger giver dig mulighed for at konfigurere multispektrale indekser til analyse og visualisering.

### Tilføj indeks

* **Type**: Specielt konfigurationspanel til indekser
* **Beskrivelse**: Åbner et interaktivt panel, hvor du kan vælge og konfigurere multispektrale vegetationsindekser (NDVI, NDRE, EVI osv.), der skal beregnes under billedbehandlingen. Du kan tilføje flere indekser, hver med sine egne visualiseringsindstillinger.
* **Tilgængelige indekser**: Rullemenuen i brugergrænsefladen indeholder**27** foruddefinerede multispektrale indeksformler (se [Multispektrale indeksformler](multispectral-index-formulas.md) for den komplette liste, herunder hvilke navne der også accepteres af CLI/SDK `--indices`-indstillingen).
* **Funktioner**:
  * Vælg blandt foruddefinerede indeksformler
  * Træk kameraets filterkanaler over i formelens båndpladser
  * Konfigurer farvegradienter til visualisering (LUT – Look-Up Tables)
  * Indstil tærskelværdier og beskæringsmetoder
  * Opret brugerdefinerede indeksformler
* **Bemærk**: Indekser beregnes ikke for LATTICE M3M-monokameraer med et enkelt bånd — multibåndsindekser er udefinerede på et enkelt bånd. Dette påvirker ikke Survey3- og LATTICE M3C-kameraer.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->



Hvert indeks, du tilføjer, gengiver sin formel som matematik med en farvet cirkel pr. båndplads: rød = Red, grøn = Green, blå = Blue, orange = Orange, cyan = Cyan, lilla = NIR, magenta = RE. Træk en cirkel fra rækken over formlen ned på en plads for at knytte den til den; dobbeltklik på en tilknyttet plads for at fjerne den. Indekset beregnes kun, når hver plads, som formlen bruger, har en kanal.

### Brugerdefinerede formler (Chloros+-funktion)

* **Type**: Array af definitioner af brugerdefinerede formler
* **Tilgængelighed**: Kræver login med et gyldigt Chloros+-abonnement.
* **Beskrivelse**: Giver dig mulighed for at oprette og gemme brugerdefinerede multispektrale indeksformler ved hjælp af båndmatematik. Brugerdefinerede formler gemmes sammen med dine projektindstillinger og kan bruges på samme måde som indbyggede indekser.
* **Sådan oprettes**:
  1. Åbn regnemaskinen til brugerdefinerede formler i konfigurationspanelet for indeks
  2. Skriv formlen ved hjælp af **bånd-slot-symbolerne**, ikke båndnavne
  3. Gem formlen med et beskrivende navn — den vises derefter nederst i formel-rullemenuen, og du trækker dit kameras kanalkredse ind på dens slots på nøjagtig samme måde som en indbygget forudindstilling
* **Formelsyntaks**:
  * Båndslots: `x`, `y`, `z`, `a`, `b`, `c` — seks positioner, som du tildeler til reelle kanaler ved at trække
  * Operatører: `+`, `-`, `*`, `/`, `^` og `()` til gruppering
  * Funktioner: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Hvorfor symboler og ikke bandnavne**: En formel skrevet som `(y-x)/(y+x)` fungerer på ethvert kamera, fordi drag-and-drop-tilknytningen afgør, om `y` er 850 nm NIR for et RGN-filter eller 808 nm NIR for et OCN-filter. De indbyggede forudindstillinger gemmes på samme måde — se [Multispektrale indeksformler](multispectral-index-formulas.md) for den nøjagtige symbolform for alle 27.
* **Hvor de fungerer**: brugerdefinerede formler gemmes sammen med projektindstillingerne og kan bruges i [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) samt i databehandlingen. De accepteres**ikke** af CLI/SDK-navnelisten `--indices`, som kun udvider de 22 indbyggede forudindstillingsnavne.

***

## Eksport

Disse indstillinger styrer formatet og kvaliteten af eksporterede, behandlede billeder.

### Kalibreret billedformat

* **Type**: Valg fra rullemenu
* **Indstillinger**:
  * **TIFF (16-bit)** – Ukomprimeret 16-bit TIFF-format
  * **TIFF (8-bit)** – Ukomprimeret 8-bit TIFF-format
  * **TIFF (32-bit, procent)** – 32-bit TIFF med flydende komma, hvor reflektansværdierne angives i procent
  * **PNG (8-bit)** – Komprimeret 8-bit PNG-format
  * **JPG (8-bit)** – Komprimeret 8-bit JPEG-format
* **Standard**: TIFF (16-bit); et projekt, der udelukkende består af 8-bit-filer, viser TIFF (8-bit) og tilbyder kun 8-bit-formaterne
* **Beskrivelse**: Vælger filformatet til gemning af behandlede og kalibrerede billeder. De eksporterede filer placeres i en undermappe for hvert format inde i hvert kameras mappe (`tiff16`, `tiff32`, `png8`, `jpg8`), med én `<Product>_Images/`-mappe pr. produkt. De eksporterede filer beholder kildens filnavn — det er mappen, ikke et filnavnsuffiks, der identificerer produktet.
* **Anbefalinger til formater**:
  * **TIFF (16-bit)**: Anbefales til videnskabelig analyse og professionelle arbejdsgange. Bevarer maksimal datakvalitet uden komprimeringsartefakter. Bedst egnet til multispektral analyse og viderebehandling i GIS-software.
  * **TIFF (32-bit, procent)**: Bedst egnet til arbejdsgange, der kræver reflektansværdier angivet i procent (0–100 %). Tilbyder maksimal præcision til radiometriske målinger.
  * **PNG (8-bit)**: Velegnet til visning på internettet og generel visualisering. Mindre filstørrelser med tabsfri komprimering, men reduceret dynamisk rækkevidde.
  * **JPG (8-bit)**: Mindste filstørrelser, bedst til forhåndsvisning og udelukkende til visning på internettet. Anvender komprimering med tab, hvilket ikke er egnet til videnskabelig analyse.
* **Bemærk**: LATTICE-radiance eksporteres altid som 32-bit float-TIFF, uanset denne indstilling.

***

## Gem projektmal

Denne funktion giver dig mulighed for at gemme dine aktuelle projektindstillinger som en genbrugelig mal.

* **Type**: Tekstindtastning + Gem-knap
* **Beskrivelse**: Indtast et beskrivende navn til din indstillingsmal, og klik på gem-ikonet. Malen gemmer alle dine aktuelle projektindstillinger (målregistrering, behandlingsindstillinger, indekser, og eksportformat) for nem genbrug i fremtidige projekter. Skabeloner gemmes i mappen `Project Templates/` i din projektgemmemappe og kan også vælges eller eksporteres fra hovedmenuen (*Vælg skabelon* / *Gem skabelon* / *Eksporter skabelon*).
* **Anvendelsestilfælde**:
  * Opret skabeloner til forskellige kamerasystemer (RGB, multispektral, NIR)
  * Gem standardkonfigurationer til bestemte afgrødetyper eller analysearbejdsgange
  * Del ensartede indstillinger på tværs af et team
* **Sådan bruges det**:
  1. Konfigurer alle de ønskede projektindstillinger
  2. Indtast et skabelonnavn (f.eks. »RedEdge Survey3 NDVI Standard«)
  3. Klik på gem-ikonet
  4. Skabelonen kan nu indlæses, når der oprettes nye projekter

***

## Gem projektmappe

Denne indstilling angiver, hvor nye projekter gemmes som standard.

* **Type**: Visning af mappesti + Rediger-knap
* **Standard (Windows)**: `Documents\MAPIR\Chloros Projects` i din mappe Dokumenter — normalt `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`, eller hvor OneDrive eller en roamingprofil har flyttet Dokumenter hen
* **Standard (Linux)**: `~/Documents/MAPIR/Chloros Projects`
* **Beskrivelse**: Viser den aktuelle standardmappe, hvor nye Chloros-projekter oprettes. Klik på redigeringsikonet for at vælge en anden mappe. Den valgte indstilling gemmes som en enkelt tekstlinje i `~/.chloros/working_directory.txt` — på Windows er det `C:\Users\<Username>\.chloros\working_directory.txt`. Hvis den pågældende fil mangler eller angiver en sti, der ikke længere findes, falder Chloros tilbage til ovenstående standardindstilling. CLI'en læser og skriver til den samme fil, så `chloros-cli` og GUI'en er altid enige om, hvor projekterne skal ligge.
* **Projektskabeloner** findes i en `Project Templates/`-undermappe i dette bibliotek.
* **Hvornår skal du ændre det**:
  * Indstil til et netværksdrev til teamsamarbejde
  * Skift til et drev med mere lagerplads til store datasæt
  * Organiser projekter efter år, kunde eller projekttype i forskellige mapper
* **Bemærk**: Ændring af denne indstilling påvirker kun NYE projekter. Eksisterende projekter forbliver på deres oprindelige placeringer.
* **Windows-stilængde**: medmindre lange Windows-stier er aktiveret (installationsprogrammets indstilling **Aktiver lange Windows-stier**), afvises en projektmappe, der er længere end ca. 130 tegn, før behandlingen påbegyndes, da dens resultater ville overskride Windows' grænse på 260 tegn for stier. Se [Start af behandlingen](../processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

***

## Bevarelse af indstillinger

Et Chloros-projekt er en **mappe**. Alle projektindstillinger gemmes i `project.json` inde i mappen; tilkoblet hardware huskes sammen med dem i `cameras.json` og `sensors.json`, så når du genåbner et projekt, genoprettes forbindelsen til dets kameraer og lyssensorer. Når du genåbner et projekt, gendannes alle indstillinger nøjagtigt, som du efterlod dem. Gemte projekter kan også styres uden brugergrænseflade med `chloros-cli project` eller SDK'ets `open_project`.

### Indstillingshierarki

Indstillingerne anvendes i følgende rækkefølge:

1. **Systemstandardindstillinger** – Indbyggede standardindstillinger defineret af Chloros

2. **Skabelonindstillinger** – Hvis du indlæser en skabelon, når du opretter et projekt
3. **Gemte projektindstillinger** – Indstillinger gemt sammen med projektfilen
4. **Manuelle justeringer** – Eventuelle ændringer, du foretager under den aktuelle session

### Indstillinger og billedbehandling

Behandlingsindstillingerne læses, når en behandlingskørsel starter. Ændring af en indstilling ændrer ikke med tilbagevirkende kraft de produkter, der allerede findes på disken — kør behandlingen igen for at anvende de nye indstillinger. Nogle få indstillinger påvirker slet ikke behandlingen:

* Opløsning af billedminiaturer (kun til visning)
* Gem projektskabelon
* Gem projektmappe

***

## Reference til konfigurationsnøgler

Til automatisering (CLI `--config`, SDK `configure` eller direkte læsning af `project.json`) er dette de nøjagtige nøgler under `Project Settings`:

| Nøglevej | Type | Standard |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | tal mellem 0 og 10.000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | tal mellem 0 og 100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | bool | `true` |
| `Processing → Export sensor response` | bool | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Standard |
| `Processing → Minimum recalibration interval` | tal 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | tal -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | cap-profil-id eller `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | liste over indekskonfigurationer | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (8-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

`Array alignment`-nøglerne skrives, første gang afsnittet »Array Alignment« vises, eller når et automatiseringskald indstiller dem. Så længe de mangler, bruger pipelinen de samme værdier som vist ovenfor (`true`, `true`, bilineær), så et projekt.json uden disse opfører sig på nøjagtig samme måde som et med dem.

### Nøgler gemt i `project.json` uden kontrol i indstillingspanelet

Disse findes under det samme `Project Settings`-træ og læses af behandlingen, men du finder ikke en widget for dem i sidepanelet:

| Nøglevej | Type | Standard | Indstillet af |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Tilpasning af, hvordan LATTICE-indgangs-TIFF-filer fortolkes; `auto` udleder fra hver fils `Chloros:ProcessingLevel` XMP-tag samt antallet af kanaler. Ignoreres for Survey3-optagelser `.raw`. Bevidst ikke en GUI-indstilling — »auto« er korrekt i alle normale tilfælde. |
| `Processing → Target reflectance dir` | stistreng | `""` | `chloros-cli process --target-reflectance-dir`, eller projektets mål-API |
| `Processing → Target reflectance config` | ordbog med kameraets serienummer som nøgle | `{}` | Registrering af et mål inden for billedrammen (tilstand `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | stistreng | `""` | SDK `process_folder(daq_log_path=…)`. Peger på en `.daq`-optagelse eller en mappe med sådanne |
| `Target Detection → Minimum calibration target squares` | tal | `4` | Gammel standard; ingen kontrol og intet CLI-flag |
| `UI → Grid thumbnail size` | tal | `160` | Billedgitterets egen zoomskyder til miniaturer |

To visningsindstillinger gemmes **på øverste niveau i `project.json`**, helt uden for `Project Settings`, da de vedrører visningstilstand snarere end behandlingsindstillinger:

| Nøglevej | Type | Standard | Indstillet af |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | heltal 1–256 | `1` | GSD (px)-kontrol på billedfanen — se [Åbning af et billede i fuld skærm](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Bedste praksis

1. **Start med standardindstillingerne**: Standardindstillingerne fungerer godt for de fleste MAPIR-kamerasystemer og typiske arbejdsgange.
2. **Opret skabeloner**: Når du har optimeret indstillingerne til en bestemt arbejdsgang eller et bestemt kamera, skal du gemme dem som en skabelon for at sikre ensartethed på tværs af projekter.
3. **Test, før du behandler hele datasættet**: Når du eksperimenterer med nye indstillinger, skal du teste dem på en lille delmængde af billeder, før du behandler hele dit datasæt.
4. **Dokumenter dine indstillinger**: Brug beskrivende skabelonnavne, der angiver kamerasystemet, behandlingstypen og den tilsigtede anvendelse (f.eks. "Survey3\_RGB\_NDVI\_Agriculture").
5. **Valg af eksportformat**: Vælg dit eksportformat ud fra den endelige anvendelse:
   * Videnskabelig analyse → TIFF (16-bit eller 32-bit)
   * GIS-behandling → TIFF (16-bit)
   * Hurtig visualisering → PNG (8-bit)
   * Deling på nettet → JPG (8-bit)

***

For yderligere oplysninger om multispektrale indekser i Chloros, se siden [Formler for multispektrale indekser](multispectral-index-formulas.md).

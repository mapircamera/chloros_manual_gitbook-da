# Projektindstillinger

Sidepanelet **Projektindstillinger** (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

) i Chloros giver dig mulighed for at konfigurere alle aspekter af billedbehandling, detektion af kalibreringsmål, beregning af multispektrale indekser og eksportindstillinger for dit projekt. Disse indstillinger gemmes sammen med dit projekt og kan gemmes som skabeloner til genbrug på tværs af flere projekter.

## Adgang til projektindstillinger

Sådan får du adgang til projektindstillingerne:

1. Åbn et projekt i Chloros
2. Klik på fanen **Projektindstillinger**<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

i venstre sidepanel
3. Indstillingspanelet viser alle tilgængelige konfigurationsmuligheder sorteret efter kategori

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Indstillinger, der afhænger af andre indstillinger, er nedtonet.** Når en overordnet indstilling gør en anden indstilling umulig (for eksempel gør det umuligt at vælge *Eksporter reflektans*, hvis man fjerner markeringen fra *Reflektanskalibrering / hvidbalance*), deaktiveres den afhængige indstilling, og dens værktøjstip angiver den indstilling, der skal ændres.
{% endhint %}

***

## Skærm

### Opløsning af billedminiaturer

* **Type**: Rullemenu
* **Valgmuligheder**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Standard**: Standard (512 px)
* **Beskrivelse**: Opløsning (den længste kant, i pixels), hvormed miniaturerne i billedgitteret gengives. Højere værdier ser skarpere ud, når der zoomes ind, men indlæses langsommere og bruger mere hukommelse. Fuld opløsning svarer til den oprindelige billedstørrelse.
* **Bemærk**: Kun til visning — dette påvirker aldrig behandlingen eller eksporterede filer.***

## Målgenkendelse

Disse indstillinger styrer, hvordan Chloros registrerer og behandler kalibreringsmål i dine billeder. Begge er kun aktive, mens **Refleksionskalibrering / hvidbalance** er aktiveret (ellers er de nedtonede, da målregistrering i så fald springes helt over).

### Mindste kalibreringsprøveområde (px)

* **Type**: Tal
* **Interval**: 0 til 10.000 pixels
* **Standard**: 25 pixels
* **Beskrivelse**: Indstiller det mindste areal (i pixels), der kræves for at et detekteret område kan betragtes som en gyldig kalibreringsmålprøve. Mindre værdier vil registrere mindre mål, men kan øge antallet af falske positiver. Større værdier kræver større og tydeligere målområder for at kunne registreres.
* **Hvornår skal der justeres**:
  * Forøg værdien, hvis du får falske registreringer på små billedartefakter
  * Sænk værdien, hvis dine kalibreringsmål fremstår små på dine billeder og ikke registreres

### Minimumsklyngedannelse for mål (0-100)

* **Type**: Tal
* **Interval**: 0 til 100
* **Standard**: 60
* **Beskrivelse**: Styrer klyngetærsklen for gruppering af områder med lignende farver ved detektering af kalibreringsmål. Højere værdier kræver, at flere lignende farver grupperes sammen, hvilket resulterer i en mere konservativ måldetektering. Lavere værdier tillader større farvevariation inden for en målgruppe.
* **Hvornår skal du justere**:
  * Forøg, hvis kalibreringsmål opdeles i flere detekteringer
  * Sænk værdien, hvis kalibreringsmål med farvevariation ikke detekteres fuldt ud

***

## Behandling

Disse indstillinger styrer, hvordan Chloros behandler og kalibrerer dine billeder.

### Vignetteringskorrektion

* **Type**: Afkrydsningsfelt
* **Standard**: Aktiveret (markeret)
* **Beskrivelse**: Anvender vignetteringskorrektion for at kompensere for objektivmørkning i billedernes kanter. Vignettering er et almindeligt optisk fænomen, hvor hjørnerne og kanterne af et billede fremstår mørkere end midten på grund af objektivets egenskaber.
* **Bivirkning**: Denne indstilling vælger også, hvilket *ukalibreret reserveprodukt* en kørsel skriver (se nedenfor).

### Reflektanskalibrering / hvidbalance

* **Type**: Afkrydsningsfelt
* **Standard**: Aktiveret (markeret)
* **Beskrivelse**: Aktiverer reflektanskalkering — baseret på detekterede kalibreringsmål i billedet og/eller nedadrettede lyssensordata fra DAQ, afhængigt af kameraet og hvad der er tilgængeligt. Dette normaliserer reflektansværdierne på tværs af dit datasæt og sikrer konsistente målinger uanset lysforholdene.
* **Når deaktiveret**: Målregistrering springes helt over, og**der kan ikke genereres noget reflektansprodukt af noget kamera** — hverken Survey3-målstyret eller LATTICE DAQ-styret. De afhængige indstillinger (*Eksporter reflektans*, *Minimalt rekalibreringsinterval* og tærskelværdierne for måldetektering) er nedtonet.

### Ikke-kalibrerede reserveprodukter: Eksporter sensorrespons / Eksporter vignettekorrigeret

* **Type**: To afkrydsningsfelter
* **Standardindstillinger**: Begge aktiveret (markeret)
* **Beskrivelse**: Når et billede ikke kan reflektanskalibreres (der blev ikke fundet noget kalibreringsmål, eller reflektanskalibrering er slået fra), gemmes det i stedet som et *ukalibreret reserveprodukt*. **Der findes nøjagtigt ét af de to reserveprodukter pr. kørsel for hver kameramodel**, valgt via *Vignettkorrektion*-kontakten:
  * Vignettkorrektion **til**→ `Vignette_Corrected_Images/` (styret af**Eksporter vignettkorrigeret**)
  * Vignettekorrektion **slået fra**→ `Sensor_Response_Images/` (styret af**Eksporter sensorrespons**)
* Det reserveprodukt, der ikke er aktivt, er nedtonet. Hvis du fjerner markeringen fra det aktive, forhindres den pågældende fil i at blive gemt.

### LATTICE-eksportprodukter

For projekter, der indeholder LATTICE-optagelser, fordeles hvert importeret LATTICE-billede til alle aktiverede **og relevante**produkter i én enkelt behandlingsomgang. Fire afkrydsningsfelter styrer denne fordeling (alle er som standard**aktiveret**):

| Indstilling | Output-mappe | Hvad der eksporteres |
| --- | --- | --- |
| **Eksporter debayered** | `Debayered_Images/` | Det lineært debayered billede. Gælder for RGB og multispektrale kameraer. |
| **Eksporter forhåndsvisning** | `Preview_Images/` | Skærmforhåndsvisningen. RGB = hvidbalance (DAQ-lysart, hvis tilgængelig, ellers gråskala) + gamma; multispektral = falskfarvestrækning. |
| **Eksport af strålingsintensitet** | `Radiance_Images/` | Float32 spektral strålingsintensitet i W/m²/sr/nm. Kun multispektral (M3C/M3M) — gælder ikke for RGB-masters. Skrives altid som 32-bit TIFF uanset indstillingen *Kalibreret billedformat*. |
| **Eksportreflektans**| `Reflectance_Calibrated_Images/` | Uint16-reflektans, skaleret således at**32768 = reflektans 1,0** (stemplet som XMP `Chloros:PixelScale`). Kun multispektralt, skrives, når en matchende `.daq`-nedadrettet registrering (eller et QA-godkendt mål i billedet) dækker billedet. |

* RGB-hovedkameraer udsender debayered + preview; radiance/reflektans springes over for disse, da de ikke er relevante.
* Bitdybden for debayered/preview følger indstillingen *Kalibreret billedformat*; radiance er altid float32.
* Survey3-behandlingen påvirkes ikke af disse fire indstillinger.

De samme fire indstillinger findes uden forramning som `chloros-cli process --debayered / --preview / --radiance / --reflectance` og som de tilsvarende parametre i SDK. De erstattede det gamle `--radiometric-output`-flag, som ikke længere findes.

{% hint style="warning" %}
**Hvis alle relevante produkter deaktiveres, mislykkes kørslen.** Fra og med version 1.2.0 rapporterer en behandlingskørsel, der anmodede om produkter, men ikke skrev nogen billedprodukter, en fejl, og CLI afsluttes med en værdi forskellig fra nul i stedet for at rapportere en lydløs succes. Loggen angiver navnet på det produkt, der ikke kunne skrives, samt årsagen hertil. En bevidst kørsel, der udelukkende omfatter metadata (intet anmodet om), betragtes stadig som en succes.
{% endhint %}

### Reflektanskilde (projektindstilling, indstilles via CLI/SDK)

Projektet gemmer også, hvilken **reflektansreference** LATTICE-reflektansproduktet bruger. Der er ingen dedikeret kontrol i indstillingspanelet; værdien gemmes i projektkonfigurationen som `Processing → "Target reflectance source"` og indstilles med `chloros-cli process --reflectance-source {auto,target,daq}` eller parameteren `reflectance_source`-parameter:

* **`auto`** (standard): et QA-godkendt kalibreringsmål inden for billedrammen bliver den absolutte reference, hvor der falder tilbage til DAQ&#x27;s nedadgående delingsværdi (ρ = πL/E), når der ikke er noget mål til stede, eller QA mislykkes.
* **`target`**: streng måldrevet reflektans — ingen DAQ-erstatning.
* **`daq`**: DAQ-autoritativ reflektans; mål inden for billedrammen bruges ikke som reference.

Den gemte værdi matches uden skelnen mellem store og små bogstaver, og nogle få stavemåder accepteres som aliaser: `target`, `target_image`, `empirical` og `empirical_line` betyder alle **mål**; `daq`, `dls`, `light_sensor` og `sensor` betyder alle**daq**. Alt andet — herunder en manglende nøgle — oversættes til**auto**.**Målte** målscanninger pr. enhed slås op ved hjælp af målenhedens serienummer/QR-kode, som f.eks. `<serial>.csv`, tre steder: i det bibliotek, der er angivet med `--target-reflectance-dir` (gemt som `Processing → "Target reflectance dir"`), projektets egen `target_reflectance/`-mappe og stien i miljøvariablen `CHLOROS_TARGET_REFLECTANCE_DIR`. Hvis der ikke findes en målt scanning for den pågældende enhed, anvendes den nominelle, offentliggjorte kurve for målmodellen i stedet.

### Debayer-metode

* **Type**: Valg fra rullemenu
* **Indstillinger**:
  * Standard (hurtig, middel kvalitet)
  * Teksturbevidst (langsom, højeste kvalitet) \[Chloros+]
* **Standard**: Standard (hurtig, middel kvalitet)
* **Beskrivelse**: Vælger den demosaicing-algoritme, der anvendes til at konvertere rå sensordata fra Bayer-mønsteret til fuldfarvebilleder. Metoden »Standard (hurtig, middel kvalitet)« giver en optimal balance mellem behandlingshastighed og billedkvalitet. Metoden »Teksturbevidst (Langsom, højeste kvalitet)” \[Chloros+] anvender en højkvalitets kantbevidst debayer kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al debayering-støj. Texture Aware-modellen kræver GPU-hukommelse (VRAM) for at køre. Vi anbefaler at bruge den, når du har &gt;4 GB VRAM til rådighed for hurtigere behandling.
* **Når rækken overhovedet er en rullemenu**: rullemenuen med to valgmuligheder vises kun, når**begge**betingelser er opfyldt — du er logget ind med et gyldigt Chloros+-abonnement,**og** projektet ikke indeholder nogen LATTICE-optagelser. Ellers vises rækken som almindelig tekst med teksten `Standard (Fast, Medium Quality)` uden noget at vælge.
* **LATTICE-bemærkning**: Der findes ingen LATTICE-trænet Texture Aware-model, og pipelinen tvinger standard-demosaic for LATTICE-rammer uanset den gemte værdi. Hvis du tilføjer en LATTICE-mappe til et projekt, hvor »Texture Aware« allerede var valgt, ændrer Chloros indstillingen tilbage til »Standard« i stedet for at efterlade en forældet værdi i `project.json`.

### Mindste rekalibreringsinterval

* **Type**: Tal
* **Interval**: 0 til 3.600 sekunder
* **Standard**: 0 sekunder
* **Beskrivelse**: Indstiller det mindste tidsinterval (i sekunder) mellem brug af kalibreringsmål. Når indstillet til 0, vil Chloros bruge hvert eneste detekteret kalibreringsmål. Når værdien er indstillet til et højere tal, vil Chloros kun anvende kalibreringsmål, der er adskilt af mindst dette antal sekunder, hvilket reducerer behandlingstiden for datasæt med hyppige registreringer af kalibreringsmål.
* **Hvornår skal indstillingen justeres**:
  * Indstil til 0 for maksimal kalibreringsnøjagtighed, når lysforholdene varierer
  * Øg (f.eks. til 60–300 sekunder) for hurtigere behandling, når lysforholdene er konstante, og der er hyppige billeder af kalibreringsmål

### Tidszoneforskydning for lyssensor

* **Type**: Tal
* **Interval**: -12 til +12 timer
* **Standard**: 0 timer
* **Beskrivelse**: Angiver tidszoneforskydningen (i timer fra UTC) for tidsstempler på lyssensordata, der bruges ved sammenkobling af lyssensorlogfiler med billedoptagelsestidspunkter. Nyere `.daq`-optagelser har deres egen tidszoneoplysning, så dette er hovedsageligt nødvendigt for ældre logfiler, der er optaget i lokal tid.

### Anvend PPK-korrektioner

* **Type**: Afkrydsningsfelt
* **Standard**: Deaktiveret (ikke markeret)
* **Beskrivelse**: Aktiverer brugen af Post-Processed Kinematic (PPK)-korrektioner fra MAPIR DAQ-optagere, der indeholder en GPS (GNSS). Når denne funktion er aktiveret, vil Chloros anvende alle .daq-logfiler, der indeholder eksponeringspin-data i dit projektmappe, og anvende præcise geolokaliseringkorrektioner på dine billeder.
* **Krav**: Der skal findes en .daq-logfil med eksponeringspin-indtastninger i dit projektkatalog
* **Hvornår skal det aktiveres**: Det anbefales altid at aktivere PPK-korrektion, hvis du har eksponeringsfeedback-indtastninger i din .daq-logfil.

### Eksponeringspin 1

* **Type**: Valg fra rullemenu
* **Synlighed**: Kun synlig, når &quot;Anvend PPK-korrektioner&quot; er aktiveret OG der er eksponeringsdata tilgængelige for pin 1
* **Indstillinger**:
  * Kameramodelnavne, der er registreret i projektet
  * &quot;Brug ikke&quot; – Ignorer denne eksponeringspin
* **Standard**: Vælges automatisk baseret på projektkonfigurationen
* **Beskrivelse**: Tildeler en bestemt kamera til eksponeringspin 1 til PPK-tidssynkronisering. Eksponeringsstiften registrerer det nøjagtige tidspunkt, hvor kameraets lukker udløses, hvilket er afgørende for nøjagtig PPK-geolokalisering.
* **Adfærd ved automatisk valg**:
  * Ét kamera + én stift: Vælger automatisk kameraet
  * Ét kamera + to ben: Ben 1 tildeles automatisk til kameraet
  * Flere kameraer: Manuel valg kræves

### Eksponeringsben 2

* **Type**: Valg fra rullemenu
* **Synlighed**: Kun synlig, når &quot;Anvend PPK-korrektioner&quot; er aktiveret OG der foreligger eksponeringsdata for ben 2
* **Indstillinger**:
  * Navne på kameramodeller, der er registreret i projektet
  * &quot;Brug ikke&quot; – Ignorer denne eksponeringspin
* **Standard**: Vælges automatisk baseret på projektkonfigurationen
* **Beskrivelse**: Tildeler et specifikt kamera til eksponeringspin 2 til PPK-tidssynkronisering ved brug af en opsætning med to kameraer.
* **Adfærd ved automatisk valg**:
  * Ét kamera + én pin: Pin 2 indstilles automatisk til &quot;Brug ikke&quot;
  * Enkelt kamera + to stik: Stik 2 indstilles automatisk til &quot;Brug ikke&quot;
  * Flere kameraer: Manuel valg kræves
* **Bemærk**: Det samme kamera kan ikke tildeles både stik 1 og stik 2 samtidigt.***

## DAQ-lyssensor

Dette afsnit vises i Projektindstillinger og viser alle DAQ-downwelling-filer i projektet — `.daq`-optagelser og DAQ-M `.csv`-downwelling-logfiler. Optagelser foretaget under fanen Lyssensorer automatisk tilføjet til det åbne projekt.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->

Hver række viser filen, sensormodellen og den diffusorhætte-korrektion, der faktisk gælder for den pågældende fil. Over rækkerne findes en enkelt projektdækkende kontrol:

### Hætte-overstyring (alle filer)

* **Type**: Rullemenu
* **Valgmuligheder**: `Auto` samt de kappe-korrektionsprofiler, der gælder for de sensortyper, der findes i projektet
* **Standard**: Auto
* **Gemmes som**: `Processing → "DAQ cap id"` (standard `auto`)
* **Beskrivelse**: `Auto` anvender den registrerede cap for hver fil (Sunshine-cap antages, hvor der ikke er registreret noget — alle MAPIR DAQ&#x27;er leveres med Sunshine-korrektoren). Valg af en bestemt kappe tilsidesætter**alle** nedadgående filer i projektet: råoptagelser korrigeres med den, og optagelser, der allerede har en kappe, omrefereres (den registrerede korrektion fortrydes, og den valgte anvendes).
* **Vigtigt**: Den valgte kappe skal matche den kappe, der fysisk var monteret under optagelsen. Hverken sensoren eller softwaren kan registrere den fysiske kappe — et forkert kappe-ID korrigerer spektrene forkert.

Der er bevidst **én** projektdækkende indstilling i stedet for dropdown-menuer pr. fil: indstillingen gælder for alle nedadgående kilder i projektet.***

## Array-justering

Dette afsnit vises **kun**, når mindst ét billede i projektet indeholder den modul-til-modul-justeringstransformation, som LATTICE-arrayer mærker ved optagelsestidspunktet (XMP `Chloros:Alignment*`-tags). Det viser, hvor mange billeder der indeholder justeringstags, hvilket kamera der er referencen (`REF`-mærke), samt en tabel med billedantal pr. kamera.

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

De samme tre indstillinger findes uden foranliggende navn som `chloros-cli process --array-alignment`, `--array-alignment-crop` og `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Indeks

Disse indstillinger giver dig mulighed for at konfigurere multispektrale indekser til analyse og visualisering.

### Tilføj indeks

* **Type**: Specielt konfigurationspanel til indekser
* **Beskrivelse**: Åbner et interaktivt panel, hvor du kan vælge og konfigurere multispektrale vegetationsindekser (NDVI, NDRE, EVI osv.), der skal beregnes under billedbehandlingen. Du kan tilføje flere indekser, hver med sine egne visualiseringsindstillinger.
* **Tilgængelige indekser**: Rullemenuen i brugergrænsefladen indeholder**27** foruddefinerede multispektrale indeksformler (se [Multispektrale indeksformler](multispectral-index-formulas.md) for den komplette liste, herunder hvilke navne der også accepteres af CLI/SDK `--indices`-indstillingen).
* **Funktioner**:
  * Vælg blandt foruddefinerede indeksformler
  * Træk dit kameras filterkanaler over på formelens båndpladser
  * Konfigurer farvegradienter til visualisering (LUT – Look-Up Tables)
  * Indstil tærskelværdier og klipningstilstande
  * Opret brugerdefinerede indeksformler
* **Bemærk**: Indekser beregnes ikke for enkeltbånds LATTICE M3M-monokameraer — multi-båndsindekser er udefinerede på ét bånd. Survey3 og LATTICE M3C påvirkes ikke.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->

Hvert indeks, du tilføjer, gengiver sin formel som matematik med en farvet cirkel pr. båndplads: rød = Red, grøn = Green, blå = Blue, orange = Orange, cyan = Cyan, lilla = NIR, magenta = RE. Træk en cirkel fra rækken over formlen over på en plads for at binde den; dobbeltklik på en bundet plads for at rydde den. Indekset beregnes kun én gang, når alle de pladser, som formlen bruger, har en kanal.

### Brugerdefinerede formler (Chloros+-funktion)

* **Type**: Matrix med definitioner af brugerdefinerede formler
* **Tilgængelighed**: Kræver login med et gyldigt Chloros+-abonnement.
* **Beskrivelse**: Giver dig mulighed for at oprette og gemme brugerdefinerede multispektrale indeksformler ved hjælp af båndmatematik. Brugerdefinerede formler gemmes sammen med dine projektindstillinger og kan bruges på samme måde som indbyggede indekser.
* **Sådan oprettes**:
  1. Åbn regnemaskinen til brugerdefinerede formler i panelet Indekskonfiguration
  2. Skriv formlen ved hjælp af **båndslotsymbolerne**, ikke båndnavne
  3. Gem formlen med et beskrivende navn — den vises derefter nederst i formel-rullemenuen, og du trækker dit kameras kanalkredse ind på dens slots på nøjagtig samme måde som en indbygget forudindstilling
* **Formelsyntaks**:
  * Båndslots: `x`, `y`, `z`, `a`, `b`, `c` — seks positioner, som du tilordner til reelle kanaler ved at trække
  * Operatører: `+`, `-`, `*`, `/`, `^`, og `()` til gruppering
  * Funktioner: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Hvorfor symboler og ikke båndnavne**: en formel skrevet som `(y-x)/(y+x)` fungerer på ethvert kamera, fordi drag-and-drop-tilknytningen afgør, om `y` er 850 nm NIR for et RGN-filter eller 808 nm NIR i et OCN-filter. De indbyggede forudindstillinger gemmes på samme måde — se [Multispektrale indeksformler](multispectral-index-formulas.md) for den nøjagtige symbolform for alle 27.
* **Hvor de fungerer**: brugerdefinerede formler gemmes sammen med projektindstillingerne og kan bruges i [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) samt i billedbehandlingen. De accepteres**ikke** af navnelisten CLI/SDK `--indices`, som kun udvider de 22 indbyggede forudindstillingsnavne.***

## Eksport

Disse indstillinger styrer formatet og kvaliteten af eksporterede, bearbejdede billeder.

### Kalibreret billedformat

* **Type**: Valg i rullemenu
* **Indstillinger**:
  * **TIFF (16-bit)** – Ukomprimeret 16-bit TIFF-format
  * **TIFF (32-bit, procent)** - 32-bit TIFF med flydende komma og reflektansværdier angivet i procent
  * **PNG (8-bit)** – Komprimeret 8-bit PNG-format
  * **JPG (8-bit)** - Komprimeret 8-bit JPEG-format
* **Standard**: TIFF (16-bit)
* **Beskrivelse**: Vælger filformatet til gemning af behandlede og kalibrerede billeder. Eksporterede filer gemmes i en undermappe for hvert format inde i hvert kameras mappe (`tiff16`, `tiff32`, `png8`, `jpg8`), med én `<Product>_Images/`-mappe pr. produkt. De eksporterede filer beholder kildefilnavnet — det er mappen, ikke et filnavnesuffiks, der identificerer produktet.
* **Anbefalinger til format**:
  * **TIFF (16-bit)**: Anbefales til videnskabelig analyse og professionelle arbejdsgange. Bevarer maksimal datakvalitet uden komprimeringsartefakter. Bedst egnet til multispektral analyse og viderebehandling i GIS-software.
  * **TIFF (32-bit, procent)**: Bedst egnet til arbejdsgange, der kræver reflektansværdier angivet som procenter (0–100 %). Tilbyder maksimal præcision til radiometriske målinger.
  * **PNG (8-bit)**: Velegnet til visning på nettet og generel visualisering. Mindre filstørrelser med tabsfri komprimering, men reduceret dynamisk rækkevidde.
  * **JPG (8-bit)**: De mindste filstørrelser, bedst til forhåndsvisning og udelukkende til visning på internettet. Anvender komprimering med tab, hvilket ikke er egnet til videnskabelig analyse.
* **Bemærk**: LATTICE-radiance eksporteres altid som 32-bit float TIFF uanset denne indstilling.***

## Gem projektmal

Denne funktion giver dig mulighed for at gemme dine aktuelle projektindstillinger som en genanvendelig mal.

* **Type**: Tekstindtastning + Gem-knap
* **Beskrivelse**: Indtast et beskrivende navn til din indstillingsskabelon, og klik på gem-ikonet. Skabelonen gemmer alle dine aktuelle projektindstillinger (målregistrering, behandlingsindstillinger, indekser og eksportformat), så du nemt kan genbruge dem i fremtidige projekter. Skabeloner gemmes i mappen `Project Templates/` i din projektgemmemappe og kan også vælges eller eksporteres fra hovedmenuen (*Vælg skabelon* / *Gem skabelon* / *Eksporter skabelon*).
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
* **Standard (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Standard (Linux)**: `~/Chloros Projects`
* **Beskrivelse**: Viser den aktuelle standardmappe, hvor nye Chloros-projekter oprettes. Klik på redigeringsikonet for at vælge en anden mappe. Den nye indstilling gemmes som en enkelt tekstlinje i `~/.chloros/working_directory.txt` — på Windows er det `C:\Users\<Username>\.chloros\working_directory.txt`. Hvis den pågældende fil mangler eller angiver en sti, der ikke længere eksisterer, falder Chloros tilbage til ovenstående standardindstilling. CLI læser og skriver til den samme fil, så `chloros-cli` og brugergrænsefladen altid er enige om, hvor projekterne befinder sig.
* **Projektskabeloner** findes i en `Project Templates/`-undermappe i dette bibliotek.
* **Hvornår skal du ændre indstillingen**:
  * Indstil til et netværksdrev til teamsamarbejde
  * Skift til et drev med mere lagerplads til store datasæt
  * Organiser projekter efter år, kunde eller projekttype i forskellige mapper
* **Bemærk**: Ændring af denne indstilling påvirker kun NYE projekter. Eksisterende projekter forbliver på deres oprindelige placeringer.***

## Bevarelse af indstillinger

Et Chloros-projekt er en **mappe**. Alle projektindstillinger gemmes i `project.json` inde i den; tilkoblet hardware gemmes sammen med dem i `cameras.json` og `sensors.json`, så når du genåbner et projekt, genoprettes forbindelsen til dets kameraer og lyssensorer. Når du genåbner et projekt, gendannes alle indstillinger nøjagtigt, som du efterlod dem. Gemte projekter kan også styres uden skærm ved hjælp af `chloros-cli project` eller SDK’s `open_project`.

### Indstillingshierarki

Indstillingerne anvendes i følgende rækkefølge:

1. **Systemstandardindstillinger** – Indbyggede standardindstillinger defineret af Chloros
2. **Skabelonindstillinger** – Hvis du indlæser en skabelon, når du opretter et projekt
3. **Gemte projektindstillinger** – Indstillinger, der er gemt sammen med projektfilen
4. **Manuelle justeringer** – Eventuelle ændringer, du foretager under den aktuelle session

### Indstillinger og billedbehandling

Behandlingsindstillingerne indlæses, når en behandlingskørsel starter. Ændring af en indstilling ændrer ikke med tilbagevirkende kraft de produkter, der allerede findes på disken — kør behandlingen igen for at anvende de nye indstillinger. Nogle få indstillinger har slet ingen indflydelse på behandlingen:

* Opløsning af billedminiaturer (kun til visning)
* Gem projektmal
* Gem projektmappe

***

## Reference til konfigurationsnøgler

Til automatisering (CLI `--config`, SDK `configure` eller ved at læse `project.json` direkte), er dette de nøjagtige nøgler under `Project Settings`:

| Nøglevej | Type | Standard |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | tal mellem 0 og 10.000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | tal 0-100 | `60` |
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
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Nøglerne `Array alignment` skrives, første gang afsnittet Array Alignment renderes, eller et automatiseringskald indstiller dem. Når de ikke er til stede, bruger pipelinen de samme værdier som vist ovenfor (`true`, `true`, bilineær), så et projekt.json uden dem opfører sig på nøjagtig samme måde som et med dem.

### Nøgler gemt i `project.json` uden kontrol i indstillingspanelet

Disse findes under det samme `Project Settings`-træ og læses af behandlingen, men du vil ikke finde en widget til dem i sidepanelet:

| Nøglevej | Type | Standard | Indstillet af |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Tilpasning af, hvordan LATTICE-indgangs-TIFF-filer fortolkes; `auto` udleder fra hver fils`Chloros:ProcessingLevel` XMP-tag samt antallet af kanaler. Ignoreres for Survey3 `.raw`-optagelser. Bevidst ikke en GUI-indstilling — &quot;auto&quot; er korrekt i alle normale tilfælde. |
| `Processing → Target reflectance dir` | sti-streng | `""` | `chloros-cli process --target-reflectance-dir`, eller projektmålet API |
| `Processing → Target reflectance config` | ordbog sorteret efter kameraserienummer | `{}` | Registrering af et mål i billedet (tilstand `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | stistreng | `""` | SDK `process_folder(daq_log_path=…)`. Peger på en `.daq`-optagelse eller en mappe med sådanne |
| `Target Detection → Minimum calibration target squares` | tal | `4` | Gammel standard; ingen kontrol og intet CLI-flag |
| `UI → Grid thumbnail size` | nummer | `160` | Billedgitterets egen zoomskyder til miniaturer |

To visningsindstillinger gemmes **på øverste niveau i `project.json`**, helt uden for `Project Settings`, da de vedrører visningstilstand snarere end behandlingsindstillinger:

| Nøglevej | Type | Standard | Indstillet af |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | heltal 1–256 | `1` | Billedfanens GSD (px)-kontrol — se [Åbning af et billede i fuld skærm](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Bedste praksis

1. **Start med standardindstillingerne**: Standardindstillingerne fungerer godt for de fleste MAPIR-kamerasystemer og typiske arbejdsgange.
2. **Opret skabeloner**: Når du har optimeret indstillingerne til en bestemt arbejdsgang eller et bestemt kamera, skal du gemme dem som en skabelon for at sikre ensartethed på tværs af projekter.
3. **Test, før du behandler det hele**: Når du eksperimenterer med nye indstillinger, skal du teste dem på et lille udsnit af billeder, før du behandler hele dit datasæt.
4. **Dokumenter dine indstillinger**: Brug beskrivende skabelonnavne, der angiver kamerasystemet, behandlingstypen og den tilsigtede anvendelse (f.eks. &quot;Survey3\_RGB\_NDVI\_Landbrug&quot;).
5. **Valg af eksportformat**: Vælg dit eksportformat ud fra den endelige anvendelse:
   * Videnskabelig analyse → TIFF (16-bit eller 32-bit)
   * GIS-behandling → TIFF (16-bit)
   * Hurtig visualisering → PNG (8-bit)
   * Deling på internettet → JPG (8-bit)

***

For mere information om multispektrale indekser i Chloros, se siden [Formler for multispektrale indekser](multispectral-index-formulas.md).

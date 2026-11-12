# Afslutning af behandlingen

Når Chloros har afsluttet behandlingen, er det tid til at gennemgå dine resultater, kontrollere udskriftskvaliteten og gøre de behandlede billeder klar til brug i din arbejdsgang. Denne side guider dig gennem de sidste trin og de næste handlinger.

## Indikation på afsluttet behandling

Når behandlingen er afsluttet uden problemer, vil du se flere indikatorer:

* ✅ **Statusbjælke**: Når 100 % færdig
* ✅ **Fejlfindingslog**: Viser de sidste `[RUN-SUMMARY]`-linjer med antal (billeder, kameragrupper, mål, kalibrerede billeder, gemte filer)
* ✅ **Startknap**: Bliver aktiveret igen (klar til næste behandlingskørsel)
* ✅ **Outputfiler**: Alle behandlede billeder gemmes i projektets output-struktur (nedenfor)

{% hint style="warning" %}
**En kørsel, der ikke skriver nogen billeder, betragtes som en fejl.** Hvis du har anmodet om billedprodukter, og kørslen ikke har skrevet nogen, rapporterer Chloros det som en fejl — `[RUN-SUMMARY]` i lognavnet antyder den sandsynlige årsag (intet importeret, intet mål detekteret eller alle anmodede produkter sprunget over som uanvendelige). Den tilsvarende CLI-kommando afsluttes med en værdi forskellig fra nul. En bevidst kørsel, der kun omfatter metadata (alle eksportprodukter slået fra, ingen indekser), betragtes stadig som en succes. Se [CLI-referencen](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Sådan finder du dine behandlede billeder

### Åbning af outputmappen

1. Klik på **Hovedmenuen**-ikonet <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">
(øverst til venstre)
2. Vælg **"Åbn projektmappe"**

3. Din filudforsker åbner i projektmappen
4. Find dit projekt ved navn

### Output-strukturen

Produkterne gemmes **i projektmappen, grupperet efter kamera og derefter efter filformat**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Kameramappe**: `LATT-<sensor>-<lens>-F<filter>` for LATTICE (svarer til optagelsens EXIF-data: `Model`), `<model>_<filter>` for Survey3 (f.eks. `Survey3N_RGN`). To kameraer, der deler en sensor og et filter, men har forskellige objektiver, har hver deres mappe — vignettering, synsfelt og forvrængning er forskellige.
* **Formatmappe**: følger din indstilling for eksportformat — `tiff16`, `tiff8`, `png8`, `jpg8`, eller `tiff32` for TIFF (32-bit, procent). Radiance er altid float32 og placeres altid under `tiff32`.
* **Produktmapper**:
  * `Reflectance_Calibrated_Images/` — kalibreret reflektans
  * `Debayered_Images/` — lineært debayered (LATTICE): mørk offset fjernet og demosaiced, uden flat field eller unmix, i sensorens oprindelige orientering
  * `Preview_Images/` — visningsforhåndsvisning (LATTICE)
  * `Radiance_Images/` — spektral strålingsintensitet i float32, W/m²/sr/nm (LATTICE multispektral)
  * `Vignette_Corrected_Images/` **eller** `Sensor_Response_Images/` — den ukalibrerede reserve for billeder uden reflektansreference; der findes nøjagtigt én af de to pr. kørsel, valgt af indstillingen for vignetteringskorrektion
  * `<INDEX>_Index_Images/` — én mappe pr. valgt indeks (f.eks. `NDVI_Index_Images`)
* **`clamp_masks/`** (nyt i 1.3.0, kun LATTICE multispektrale M3C-kameraer): vises i en formatmappe ved siden af produktmapperne, når båndsepareringen beskærer negative værdier til nul i et strålings- eller reflektansprodukt. Den indeholder én mask pr. pixel pr. berørt produkt, f.eks. `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif`. Masker findes aldrig inde i en produktmappe, så en produktmappe kan overføres til fotogrammetri-software, som den er. Se [Clamp-masker](../output-image-formats.md#clamp-masks).

{% hint style="info" %}
**Hvert eksporteret produkt beholder navnet på KILDE-filen.**En radiance-eksport af `capture_..._raw.tif` hedder stadig `capture_..._raw.tif` — den ligger blot i `tiff32/Radiance_Images/`.**Mappen identificerer produktet, ikke filnavnet**, så en søgning efter `*radiance*.tif` giver ingen resultater; søg i stedet efter mappen.
{% endhint %}

<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-M3C multispectral camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images, clamp_masks) and tiff32 (Radiance_Images, clamp_masks) subfolders visible -->


### Hvor mange filer bør der være?

Tæl ikke ud fra en formel — antallet af output afhænger af, hvilke produkter der var aktiveret, og hvilke der gælder for hvert kamera (f.eks. får RGB-kameraer ingen radiance/reflectance). Det endelige antal fremgår af loggen: den sidste linje med `[RUN-SUMMARY]` angiver præcist, hvor mange filer der blev skrevet, og hjælpelinjer forklarer alt, hvad der blev sprunget over.

***

## Gennemgang af behandlede billeder

### Hurtig forhåndsvisning i File Explorer

**Indbygget forhåndsvisning i Windows:**

1. Naviger til en produktmappe (f.eks. `tiff16/Reflectance_Calibrated_Images/`)
2. Vælg en billedfil
3. Forhåndsvisningen vises i forhåndsvisningspanelet i Windows-stifinder
4. Brug piletasterne til at bladre gennem billederne

### Forhåndsvisning i eksterne billedvisere

**Anbefalede billedvisere:**

* **QGIS** – Gratis GIS-software (bedst til georefereret multispektral analyse)
* **IrfanView** – Hurtig, letvægts billedfremviser (understøtter TIFF)
* **Adobe Photoshop** – Professionel redigering (understøtter TIFF)
* **GIMP** – Gratis alternativ til Photoshop
* **Windows-fotos** – Grundlæggende visning (understøtter muligvis ikke 16-bit TIFF)

### Forhåndsvisning i Chloros-billedfremviseren

Brug Chloros' indbyggede billedfremviser til avanceret visualisering:

1. Klik på en billedminiature i filbrowseren
2. Billedet åbnes i det primære forhåndsvisningsområde
3. Klik på fanen **Billedfremviser** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
i venstre sidepanel
4. Brug [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) til interaktiv analyse

Se [Billedfremviser](../image-viewer-gui/opening-an-image-full-screen.md) for detaljerede instruktioner.

***

## Aflæsning af reflektans-pixelværdier (GIS / Pix4D / Scripts)

Reflektansen gemmes som et heltal (DN), og **den DN-værdi, der svarer til ρ = 1,0, afhænger af kildekameraet**:

| Kilde          | ρ = 1,0 er | Sådan afgøres det                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (margin op til ρ 2,0) | XMP-tagget `Chloros:PixelScale=32768` er indsat i filen |
| Survey3          | **65535** (begrænset til ρ 1,0)     | Intet `Chloros:PixelScale`-tag — denne fravær er selve signalet (Survey3-produkter indeholder andre `Chloros:*`-behandlingsmærker) |

**Læs `Chloros:PixelScale`-mærket og divider med det** i stedet for at antage en generel værdi på 65535 — at dividere LATTICE-reflektansen med 65535 halverer automatisk alle værdier. En reflektansfil fra før version 1.3.0, der er skrevet som 8-bit, blev beskåret, ikke omskaleret, og har ingen skalatag: Eksporter den igen i 16-bit eller 32-bit i stedet for at dividere den. Se [Output Image Formats](../output-image-formats.md) for den fulde forklaring.

***

## Metadata, der overføres til eksportfilerne

Hvert produkt bevarer kildeoptagelsens **GPS-blok**og dens**EXIF-sub-IFD**, så en
eksport medfører `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` og
`CameraSerialNumber` samt georeferencen.

{% hint style="warning" %}
**Hvis en ortomosaik vises i en absurd skala, skal du først kontrollere `FocalLength`.**
Pix4D beregner GSD ud fra brændvidde plus højde. Uden tagget
falder det tilbage til en vildt forkert skala — på en målt flyvning med 49 optagelser blev en 411 m × 160 m
appelsinplantage blev rekonstrueret til 47,8 km × 13 km, hvilket resulterede i en 455-megapixel ortofoto bestående hovedsageligt af
tom plads. Langsom indlæsning og en uventet stor filstørrelse er symptomer på dette, ikke separate
problemer.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Ikke *alle* tags kopieres. IFD0’s strukturelle tags udelades bevidst (kopiering
af dem ødelægger LATTICE-outputtet), og `ExifImageWidth` / `ExifImageHeight` er udeladt,
fordi de beskriver den oprindelige optagelse — en eksport, der er blevet ændret i størrelse, ville ellers
angive dimensioner, som dens eget raster modsiger.

Chloros skriver også sine egne `Chloros:*` XMP-tags, der beskriver, hvad der er blevet gjort med hvert produkt
(for eksempel `Chloros:ProcessingLevel`, `Chloros:CalibrationApplied`,
`Chloros:VignetteCorrected`). Fra og med version 1.3.0 indeholder Survey3-produkter dem i alle behandlings, indeksprodukter beskriver selve indekset (`Chloros:ProcessingLevel=index`,
formlen og om den blev beregnet ud fra reflektans eller strålingsintensitet) i stedet for at gentage
kildedataets tags, og produkter i mapper, der er dybere end 260 tegn, bevarer også deres metadata. Se [Output Image Formats](../output-image-formats.md#index-products).

***

## Gennemgang af fejlfindingsloggen

### Kontroller for advarsler eller fejl

1. Åbn fanen **Fejlfindingslog** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

2. Rul gennem meddelelserne
3. Se efter gule advarsler eller røde fejl
4. Læs linjerne `[RUN-SUMMARY]` og eventuelle tip
5. Kontakt MAPIR-support for at få hjælp

### Gemning af loggen

For at gemme en oversigt over behandlingen eller sende den til MAPIR-support:

1. Klik på knappen **"Kopier"**eller**"Download"**

2. Gem som tekstfil i projektmappen
3. Vedlæg den med projektdokumentationen
4. Send den til MAPIR-support, hvis der opstår problemer

For at sende et problem direkte til MAPIR skal du bruge fejlikonet (**Rapporter en fejl**) øverst på fanen Debug Log — se [Rapportering af en fejl fra fanen Log](monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

***

## Almindelige problemer med output og løsninger

### Problem: Manglende outputfiler

**Mulige årsager:**

* Produktet gælder ikke for det pågældende kamera (f.eks. radiance/reflektans for RGB-kameraer — det fremgår af loggen)
* En påkrævet reference manglede (f.eks. reflektans uden mål og uden `.daq` nedadgående stråling)
* Afkrydsningsfeltet for eksport af produktet var deaktiveret i projektindstillingerne
* Der var ikke mere plads på disken under eksporten

**Løsninger:**

1. Tjek `[RUN-SUMMARY]`-tipene og `[EXPORT-CHECK]`-linjerne i fejlfindingsloggen — de forklarer, hvilke kameraer der springes over
2. Kontroller afkrydsningsfelterne for eksportprodukter i [Projektindstillinger](adjusting-project-settings.md)
3. Kontroller, at der var tilstrækkelig diskplads
4. Kør processen igen, når årsagen er løst

### Problem: Mørke eller lyse kanter (vignettering stadig synlig)

**Mulige årsager:**

* Vignetteringskorrektion deaktiveret
* Kamera/objektiv findes ikke i Chloros-profildatabasen
* Ekstrem vignettering, der overstiger korrektionsmulighederne

**Løsninger:**

1. Kontroller, at vignetteringskorrektion var aktiveret i Projektindstillinger
2. Kontroller, at kameramodellen er registreret korrekt
3. Kontakt MAPIR-support, hvis vignetteringen fortsætter

### Problem: Forkerte farver eller værdier

**Mulige årsager:**

* Ingen kalibreringsmål er registreret
* Forkert model for kalibreringsmål er valgt
* Reflektanskalibrering er deaktiveret
* Målbilleder af dårlig kvalitet

**Løsninger:**

1. Kontroller, at reflektanskalibrering var aktiveret
2. Kontroller meddelelserne om "Mål fundet" i fejlfindingsloggen
3. Gennemgå kvaliteten af målbillederne
4. Kør behandlingen igen med de korrekte mål markeret

### Problem: NDVI-værdierne ser forkerte ud

**Forventede NDVI-intervaller:**

* **Vand, klipper, jord**: -0,1 til 0,2
* **Tynd/usund vegetation**: 0,2 til 0,4
* **Moderat vegetation**: 0,4 til 0,6
* **Sund, tæt vegetation**: 0,6 til 0,9**Hvis værdierne ligger uden for disse intervaller:**

1. Kontroller, at reflektanskalibrering er blevet anvendt
2. Kontroller, at lyssensorloggen er inkluderet
3. Kontroller, at kalibreringsmålene er blevet registreret
4. Sørg for, at den korrekte kameramodel er blevet registreret
5. Gennemgå tidspunktet og forholdene for optagelsen af målbillederne
6. Hvis du selv beregner indekser ud fra reflektansfiler, skal du kontrollere, at du har divideret med filens `Chloros:PixelScale` (se ovenfor)

***

## Brug af dine behandlede billeder

### Til fotogrammetri / oprettelse af ortomosaikker

**Anbefalet arbejdsgang:**

1.**Importer kalibrerede reflektansbilleder** til fotogrammetri-software:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Bevar EXIF-metadata**: Sørg for, at GPS-data bevares til geotagging
3. **Kalibrerede arbejdsgange**: Brug reflektansbilleder for videnskabelig nøjagtighed — LATTICE-reflektans indeholder de XMP-kalibreringsmærker, som Pix4D læser
4. **Behandl indeksmosaikker**: Opret NDVI-ortomosaikker ud fra individuelle indeksbilleder
5. **Eksporter georefererede GeoTIFF-filer**: Til brug i GIS-applikationer

### Til GIS-analyse

**Anbefalet arbejdsgang:**

1.**Indlæs i QGIS, ArcGIS eller lignende**

2.**Brug 16-bit TIFF** reflektansbilleder til multibåndsanalyse (divider med filens `Chloros:PixelScale`)
3. **Brug indeksbilleder** (NDVI, NDRE) som brugsklare vegetationslag
4. **Rasterberegner**: Kombiner bånd til brugerdefineret analyse
5. **Eksporter**: Opret klassifikationskort, ændringsdetektering, kort over vegetationens sundhedstilstand

### Til direkte analyse / rapportering

**Anbefalet arbejdsgang:**

1.**Brug indeksbilleder med LUT-farver** til visuelle rapporter
2. **Udtræk statistik**: Gennemsnitlig NDVI pr. mark/parcel
3. **Tidsserier**: Sammenlign indekser på tværs af flere sessioner
4. **Generer rapporter**: Inkluder kort, statistik og visualiseringer***

## Arkivering og sikkerhedskopiering

### Anbefalet sikkerhedskopieringsstrategi

**Hvad der skal gemmes:**

* ✅ **Originale RAW/JPG-billeder eller LATTICE-råoptagelser** – Arkiver på separat drev/i skyen; rådata er kilden til behandlingsforløbet, og alt andet kan gendannes ud fra disse
* ✅ **`.daq` / `.csv` lyssensorfiler** – Nødvendige for senere at kunne genberegne reflektansen
* ✅ **Bearbejdede resultater** – Gem kalibrerede billeder og indekser
* ✅ **Projektmappe** (`project.json` og tilhørende filer) – Indeholder alle indstillinger til genbehandling, hvis det bliver nødvendigt
* ✅ **Fejlfindingslog** – Dokumenterer detaljer om behandlingen
* ✅ **Kalibreringsmålbilleder** – Til verifikation og genbehandling

**Anbefalinger til opbevaring:**

* **Umiddelbar sikkerhedskopi**: Ekstern harddisk
* **Langtidsarkiv**: Cloud-lager (Google Drive, Dropbox osv.)
* **Kritiske data**: Opbevar 2-3 kopier på forskellige steder***

## Næste behandlingskørsler

### Genbrug af projektindstillinger

Hvis du skal behandle lignende datasæt i fremtiden:

1. **Gem projektmal** (hvis det ikke allerede er gjort)
2. **Opret nyt projekt** ved hjælp af den gemte mal
3. **Importer nye billeder**

4.**Behandl**med identiske indstillinger for at sikre konsistens

### Batchbehandling af flere sessioner

Ved flere sessioner/datasæt:**Mulighed 1: GUI – Flere projekter**

* Opret et separat projekt for hver session
* Brug ensartede skabelonindstillinger
* Behandl ét ad gangen

**Mulighed 2: Chloros CLI (kun Chloros+)**

* Automatiser batchbehandlingen
* Behandl flere mapper med scripts
* Se [CLI-dokumentation](../CLI.md) og [CLI-reference](../reference/cli-reference.md)

**Mulighed 3: Python SDK (kun Chloros+)**

* Programmatisk styring
* Integration med analysepipelines
* Se [API-dokumentation](../api-python-sdk.md) og [SDK-reference](../reference/sdk-reference.md)

***

## Fejlfinding ved efterbehandling

### Genbehandling med andre indstillinger

Hvis resultaterne ikke er tilfredsstillende:

1. Behold de originale billeder (slet dem aldrig)
2. Åbn det samme projekt i Chloros
3. Juster indstillingerne i panelet »Projektindstillinger«
4. Kør behandlingen igen — resultaterne gemmes i de samme produktmapper, så filer med samme navn fra den forrige kørsel erstattes

### Behandling af en delmængde af billeder

Sådan behandler du kun bestemte billeder igen:

1. Opret et nyt projekt
2. Importer kun de billeder, der skal behandles igen
3. Brug den samme indstillingsskabelon
4. Behandl det mindre datasæt

### Få hjælp

Hvis du støder på problemer:

* 📧 **E-mail**: info@mapir.camera (vedhæft fejlfindingslog)
* 🌐 **Support**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Ofte stillede spørgsmål](../faq.md)
* 📖 **Dokumentation**: [Chloros-manual](../)

***

## Oversigt: Komplet arbejdsgang

Du har nu gennemført hele Chloros-behandlingsarbejdsgangen:

1. ✅ **Oprettet projekt** – Se [Projekter](../projects.md)
2. ✅ **Tilføjet filer** – Se [Tilføjelse af filer](adding-files-to-a-project.md)
3. ✅ **Justeret indstillinger** – Se [Justering af projektindstillinger](adjusting-project-settings.md)
4. ✅ **Markeret mål** – Se [Valg af målbilleder](choosing-target-images.md)
5. ✅ **Startet behandling** - Se [Start af behandlingen](starting-the-processing.md)
6. ✅ **Overvåget fremskridt** - Se [Overvågning af behandlingen](monitoring-the-processing.md)
7. ✅ **Gennemgåede resultater** – Denne side**Dine kalibrerede, reflektanskorrigerede multispektrale billeder er klar til analyse!**

***

## Yderligere ressourcer

### Avancerede funktioner

* [**Billedfremviser**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktiv visualisering og analyse
* [**Indeks/LUT-sandkasse**](../image-viewer-gui/index-lut-sandbox.md) – Test af brugerdefinerede indekser
* [**Multispektrale indeksformler**](../project-settings/multispectral-index-formulas.md) – Komplet indeksreference

### Automatisering og integration

* [**CLI-dokumentation**](../CLI.md) – Batchbehandling via kommandolinjen
* [**Python SDK**](../api-python-sdk.md) – Programmatisk automatisering
* [**Funktioner i Chloros+**](../#chloros) – Avancerede behandlingsfunktioner

### Support og læring

* [**Ofte stillede spørgsmål**](../faq.md) – Svar på almindelige spørgsmål
* [**Kalibreringsmål**](../calibration-targets.md) – Forståelse af reflektanskalibrering
* [**Understøttede kameraer**](../supported-cameras.md) – Kompatibel hardware

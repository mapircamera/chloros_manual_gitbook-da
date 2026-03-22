# Afslutning af behandlingen

Når Chloros er færdig med behandlingen, er det tid til at gennemgå resultaterne, kontrollere udskriftskvaliteten og gøre de behandlede billeder klar til brug i din arbejdsgang. Denne side guider dig gennem de sidste trin og de næste handlinger.

## Indikation på, at behandlingen er afsluttet

Når behandlingen er afsluttet med succes, vil du se flere indikatorer:

* ✅ **Statusbjælke**: Når 100 % færdig
* ✅ **Fejlfindingslog**: Viser meddelelsen &quot;Behandling afsluttet&quot;
* ✅ **Startknap**: Bliver aktiveret igen (klar til næste behandlingskørsel)
* ✅ **Output-filer**: Alle behandlede billeder gemmes i undermappen for kameramodellen***

## Find dine behandlede billeder

### Åbn output-mappen

1. Klik på **Hovedmenuen** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (øverst til venstre)
2. Vælg **&quot;Åbn projektmappe&quot;**

3. Din filudforsker åbner projektmappen
4. Find dit projekt ved hjælp af navnet

***

## Gennemgang af behandlede billeder

### Hurtig forhåndsvisning i filudforskeren

**Windows indbygget forhåndsvisning:**

1. Naviger til undermappen for kameramodellen
2. Vælg en billedfil
3. Forhåndsvisningen vises i Windows Explorer-forhåndsvisningspanelet
4. Brug piletasterne til at bladre gennem billederne

### Forhåndsvisning i eksterne billedvisere

**Anbefalede visere:*** **QGIS** - Gratis GIS-software (bedst til georefereret multispektral analyse)
* **IrfanView** - Hurtig, letvægts billedviser (understøtter TIFF)
* **Adobe Photoshop** – Professionel redigering (understøtter TIFF)
* **GIMP** – Gratis alternativ til Photoshop
* **Windows Photos** – Grundlæggende visning (understøtter muligvis ikke 16-bit TIFF)

### Forhåndsvisning i Chloros Image Viewer

Brug Chloros&#x27;s indbyggede Image Viewer til avanceret visualisering:

1. Klik på en billedminiature i filbrowseren
2. Billedet åbnes i det primære forhåndsvisningsområde
3. Klik på **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> i venstre sidepanel
4. Brug [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) til interaktiv analyse

Se [Billedfremviser](../image-viewer-gui/opening-an-image-full-screen.md) for detaljerede instruktioner.

***

## Gennemgang af fejlfindingsloggen

### Kontroller for advarsler eller fejl

1. Åbn fanen **Fejllog** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> fanen
2. Rul gennem meddelelserne
3. Se efter gule advarsler eller røde fejl
4. Gennemgå eventuelle problemer, der er noteret
5. Kontakt MAPIR support for at få hjælp

### Gem loggen

For at gemme en oversigt over behandlingen eller for at sende den til MAPIR Support:

1. Klik på knappen **&quot;Kopier&quot;**eller**&quot;Download&quot;**

2. Gem som tekstfil i projektmappen
3. Vedlæg den med projektdokumentationen
4. Send den til MAPIR support, hvis der opstår problemer

***

## Almindelige problemer med output og løsninger

### Problem: Manglende outputfiler

**Mulige årsager:**

* Filerne opfyldte ikke behandlingskriterierne
* Billeder kun til mål (ekskluderet fra eksport)
* Der løb tør for diskplads under eksporten
* Filkorruption under behandlingen

**Løsninger:**

1. Tjek fejlfindingsloggen for spring-over-/fejlmeddelelser
2. Kontroller, at der var tilstrækkelig diskplads
3. Tæl filer: Skal svare til (oprindeligt antal - målantal) × (indekser + 1)
4. Importer og behandl eventuelle manglende filer igen

### Problem: Mørke eller lyse kanter (vignettering stadig synlig)

**Mulige årsager:**

* Vignetteringskorrektion deaktiveret
* Kamera/objektiv findes ikke i Chloros-profildatabasen
* Ekstrem vignettering, der overstiger korrektionskapaciteten

**Løsninger:**

1. Kontroller, at vignetteringskorrektion var aktiveret i projektindstillingerne
2. Kontroller, at kameramodellen er registreret korrekt
3. Kontakt MAPIR-support, hvis vignetteringen fortsætter

### Problem: Forkerte farver eller værdier

**Mulige årsager:**

* Ingen kalibreringsmål registreret
* Forkert kalibreringsmålmodel valgt
* Reflektanskalibrering deaktiveret
* Målbilleder af dårlig kvalitet

**Løsninger:**

1. Kontroller, at reflektanskalibrering var aktiveret
2. Tjek meddelelserne &quot;Mål fundet&quot; i fejlfindingsloggen
3. Gennemgå kvaliteten af målbillederne
4. Genbehandl med de korrekte mål markeret

### Problem: NDVI-værdier synes forkerte

**Forventede NDVI-intervaller:*** **Vand, sten, jord**: -0,1 til 0,2
* **Tynd/usund vegetation**: 0,2 til 0,4
* **Moderat vegetation**: 0,4 til 0,6
* **Sund, tæt vegetation**: 0,6 til 0,9**Hvis værdierne ligger uden for disse intervaller:**

1. Kontroller, at reflektanskalibrering er blevet anvendt
2. Kontroller, at lyssensorloggen er inkluderet
3. Kontroller, at kalibreringsmålene er blevet registreret
4. Sørg for, at den korrekte kameramodel er blevet registreret
5. Gennemgå tidspunktet og forholdene for optagelse af målbilleder

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
3. **Kalibrerede arbejdsgange**: Brug reflektansbilleder for videnskabelig nøjagtighed
4. **Behandl indeksmosaikker**: Opret NDVI-ortomosaikker fra individuelle indeksbilleder
5. **Eksporter georefererede GeoTIFF**: Til brug i GIS-applikationer

### Til GIS-analyse

**Anbefalet arbejdsgang:**

1.**Indlæs i QGIS, ArcGIS eller lignende**

2.**Brug 16-bit TIFF** reflektansbilleder til multibåndsanalyse
3. **Brug indeksbilleder** (NDVI, NDRE) som brugsklare vegetationslag
4. **Rasterberegner**: Kombiner bånd til brugerdefineret analyse
5. **Eksport**: Opret klassifikationskort, ændringsdetektering, kort over vegetationens sundhed

### Til direkte analyse / rapportering

**Anbefalet arbejdsgang:**

1.**Brug indeksbilleder med LUT-farver** til visuelle rapporter
2. **Udtræk statistik**: Gennemsnitlig NDVI pr. felt/plot
3. **Tidsserier**: Sammenlign indekser på tværs af flere sessioner
4. **Generer rapporter**: Inkluder kort, statistikker og visualiseringer***

## Arkivering og sikkerhedskopiering

### Anbefalet sikkerhedskopieringsstrategi

**Hvad skal gemmes:*** ✅ **Originale RAW/JPG-billeder** – Arkiver på separat drev/i skyen
* ✅ **Behandlede resultater** – Gem kalibrerede billeder og indekser
* ✅ **Projektfil** – Indeholder alle indstillinger til genbehandling, hvis det er nødvendigt
* ✅ **Fejlfindingslog** – Dokumenterer detaljer om behandlingen
* ✅ **Kalibreringsmålbilleder** – Til verifikation og genbehandling**Anbefalinger til opbevaring:*** **Umiddelbar backup**: Ekstern harddisk
* **Langtidsarkiv**: Cloud-lagring (Google Drive, Dropbox osv.)
* **Kritiske data**: Opbevar 2-3 kopier på forskellige steder***

## Næste behandlingskørsler

### Genbrug af projektindstillinger

Hvis du skal behandle lignende datasæt i fremtiden:

1. **Gem projektmal** (hvis det ikke allerede er gjort)
2. **Opret nyt projekt** ved hjælp af den gemte skabelon
3. **Importer nye billeder**

4.**Behandl**med identiske indstillinger for at sikre konsistens

### Batchbehandling af flere sessioner

Ved flere sessioner/datasæt:**Mulighed 1: GUI – Flere projekter**

* Opret et separat projekt for hver session
* Brug ensartede skabelonindstillinger
* Behandl én ad gangen

**Mulighed 2: Chloros CLI (kun Chloros+)**

* Automatiser batchbehandling
* Behandl flere mapper med scripts
* Se [CLI-dokumentation](../CLI.md)

**Mulighed 3: Python SDK (kun Chloros+)**

* Programmatisk styring
* Integration med analysepipelines
* Se [API-dokumentation](../api-python-sdk.md)

***

## Fejlfinding ved efterbehandling

### Genbehandling med andre indstillinger

Hvis resultaterne ikke er tilfredsstillende:

1. Behold de originale billeder (slet dem aldrig)
2. Åbn det samme projekt i Chloros
3. Juster indstillingerne i panelet Projektindstillinger
4. Behandl igen – resultaterne overskriver de tidligere resultater

### Behandling af en delmængde af billeder

Sådan behandler du kun bestemte billeder:

1. Opret et nyt projekt
2. Importer kun de billeder, der skal behandles igen
3. Brug den samme indstillingsskabelon
4. Behandl det mindre datasæt

### Få hjælp

Hvis du støder på problemer:

* 📧 **E-mail**: info@mapir.camera (vedhæft fejlfindingslog)
* 🌐 **Support**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Ofte stillede spørgsmål](../faq.md)
* 📖 **Dokumentation**: [Chloros Manual](../)***

## Oversigt: Komplet arbejdsgang

Du har nu gennemført hele Chloros-behandlingsworkflowet:

1. ✅ **Oprettet projekt** - Se [Projekter](../projects.md)
2. ✅ **Tilføjet filer** - Se [Tilføjelse af filer](adding-files-to-a-project.md)
3. ✅ **Justeret indstillinger** - Se [Justering af projektindstillinger](adjusting-project-settings.md)
4. ✅ **Markeret mål** - Se [Valg af målbilleder](choosing-target-images.md)
5. ✅ **Startet behandling** - Se [Start af behandlingen](starting-the-processing.md)
6. ✅ **Overvåget fremskridt** - Se [Overvågning af behandlingen](monitoring-the-processing.md)
7. ✅ **Gennemgåede resultater** - Denne side**Dine kalibrerede, reflektanskorrigerede multispektrale billeder er klar til analyse!**

***

## Yderligere ressourcer

### Avancerede funktioner

* [**Billedfremviser**](../image-viewer-gui/opening-an-image-full-screen.md) - Interaktiv visualisering og analyse
* [**Indeks/LUT-sandkasse**](../image-viewer-gui/index-lut-sandbox.md) – Test af brugerdefinerede indekser
* [**Multispektrale indeksformler**](../project-settings/multispectral-index-formulas.md) – Komplet indeksreference

### Automatisering og integration

* [**CLI-dokumentation**](../CLI.md) – Batchbehandling via kommandolinjen
* [**Python SDK**](../api-python-sdk.md) - Programmatisk automatisering
* [**Chloros+ Funktioner**](../#chloros) - Avancerede behandlingsfunktioner

### Support og læring

* [**FAQ**](../faq.md) - Svar på almindelige spørgsmål
* [**Kalibreringsmål**](../calibration-targets.md) - Forståelse af reflektanskalibrering
* [**Understøttede kameraer**](../supported-cameras.md) - Kompatibel hardware

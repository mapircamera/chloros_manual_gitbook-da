# Afslutning af behandlingen

Når Chloros har afsluttet behandlingen, er det tid til at gennemgå dine resultater, kontrollere outputkvaliteten og forberede dine behandlede billeder til brug i dit workflow. Denne side guider dig gennem de sidste trin og de næste handlinger.

## Indikation for færdig behandling

Når behandlingen er afsluttet, vil du se flere indikatorer:

* ✅ **Statusbjælke**: Når 100 % færdig
* ✅ **Fejlfindingslog**: Viser meddelelsen &quot;Behandling færdig&quot;
* ✅ **Startknap**: Bliver aktiveret igen (klar til næste behandlingskørsel)
* ✅ **Outputfiler**: Alle behandlede billeder gemmes i undermappen for kameramodellen

***

## Find dine behandlede billeder

### Åbn outputmappen

1. Klik på **Hovedmenu** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> (øverst til venstre)
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

**Anbefalede billedvisere:**

* **QGIS** – Gratis GIS-software (bedst til georefereret multispektral analyse)
* **IrfanView** – Hurtig, letvægts billedviser (understøtter TIFF)
* **Adobe Photoshop** - Professionel redigering (understøtter TIFF)
* **GIMP** - Gratis alternativ til Photoshop
* **Windows Photos** - Grundlæggende visning (understøtter muligvis ikke 16-bit TIFF)

### Forhåndsvisning i Chloros billedfremviser

Brug Chloros&#x27;s indbyggede billedfremviser til avanceret visualisering:

1. Klik på en billedminiature i filbrowseren.
2. Billedet åbnes i det primære forhåndsvisningsområde.
3. Klik på **Billedfremviser** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> i venstre sidepanel.
4. Brug [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) til interaktiv analyse.

Se [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md) for detaljerede instruktioner.

***

## Gennemgang af fejlfindingsloggen

### Kontroller for advarsler eller fejl

1. Åbn fanen **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> fanen
2. Rul gennem meddelelserne
3. Se efter gule advarsler eller røde fejl
4. Gennemgå eventuelle problemer
5. Kontakt MAPIR support for at få hjælp

### Gem loggen

For at gemme en oversigt over behandlingen eller sende den til MAPIR Support:

1. Klik på knappen **&quot;Kopier&quot;** eller **&quot;Download&quot;**
2. Gem som tekstfil i projektmappen
3. Vedlæg projektdokumentationen
4. Send til MAPIR support, hvis der opstår problemer

***

## Almindelige outputproblemer og løsninger

### Problem: Manglende outputfiler

**Mulige årsager:**

* Filerne opfyldte ikke behandlingskriterierne
* Kun målbilleder (ekskluderet fra eksport)
* Diskpladsen løb tør under eksporten
* Filkorruption under behandlingen

**Løsninger:**

1. Kontroller fejlfindingsloggen for spring/fejlmeddelelser
2. Kontroller, at der var tilstrækkelig diskplads
3. Tæl filer: Skal matche (oprindeligt antal - målantal) × (indekser + 1)
4. Importer og behandl eventuelle manglende filer igen

### Problem: Mørke eller lyse kanter (vignettering stadig synlig)

**Mulige årsager:**

* Vignetteringskorrektion deaktiveret
* Kamera/objektiv findes ikke i Chloros-profildatabasen
* Ekstrem vignettering, der overstiger korrektionsmulighederne

**Løsninger:**

1. Kontroller, at vignetteringskorrektion er aktiveret i projektindstillingerne.
2. Kontroller, at kameramodellen er registreret korrekt.
3. Kontakt MAPIR-support, hvis vignetteringen fortsætter.

### Problem: Forkerte farver eller værdier

**Mulige årsager:**

* Der er ikke registreret nogen kalibreringsmål.
* Der er valgt en forkert kalibreringsmålmodel.
* Reflektanskalibrering er deaktiveret.
* Målbilleder af dårlig kvalitet.

**Løsninger:**

1. Kontroller, at reflektanskalibrering er aktiveret.
2. Kontroller meddelelser om &quot;Mål fundet&quot; i fejlfindingsloggen.
3. Kontroller målbilledernes kvalitet.
4. Genbehandl med de korrekte mål markeret.

### Problem: NDVI-værdierne synes at være forkerte.

**Forventede NDVI-intervaller:**

* **Vand, sten, jord**: -0,1 til 0,2
* **Sparsom/usund vegetation**: 0,2 til 0,4
* **Moderat vegetation**: 0,4 til 0,6
* **Sund, tæt vegetation**: 0,6 til 0,9

**Hvis værdierne ligger uden for disse intervaller:**

1. Kontroller, at reflektanskalibrering er blevet anvendt.
2. Kontroller, at lyssensorloggen er inkluderet.
3. Kontroller, at kalibreringsmålene er blevet registreret.
4. Sørg for, at det korrekte kameramodel er blevet registreret.
5. Gennemgå tidspunktet for og betingelserne for optagelse af målbilleder.

***

## Brug af dine behandlede billeder

### Til fotogrammetri/oprettelse af ortomosaik

**Anbefalet arbejdsgang:**

1. **Importer kalibrerede reflektansbilleder** til fotogrammetri-software:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Bevar EXIF-metadata**: Sørg for, at GPS-data bevares til geotagging
3. **Kalibrerede arbejdsgange**: Brug reflektansbilleder for videnskabelig nøjagtighed
4. **Behandl indeksmosaikker**: Opret NDVI ortomosaikker fra individuelle indeksbilleder
5. **Eksporter georefererede GeoTIFF**: Til brug i GIS-applikationer

### Til GIS-analyse

**Anbefalet arbejdsgang:**

1. **Indlæs i QGIS, ArcGIS eller lignende**
2. **Brug 16-bit TIFF** reflektansbilleder til multibåndsanalyse
3. **Brug indeksbilleder** (NDVI, NDRE) som brugsklare vegetationslag
4. **Rasterberegner**: Kombiner bånd til brugerdefineret analyse
5. **Eksport**: Opret klassificeringskort, ændringsdetektering, kort over vegetationens sundhedstilstand

### Til direkte analyse/rapportering

**Anbefalet arbejdsgang:**

1. **Brug indeksbilleder med LUT-farver** til visuelle rapporter
2. **Uddrag statistik**: Gennemsnitlig NDVI pr. felt/parcel
3. **Tidsserier**: Sammenlign indekser på tværs af flere sessioner
4. **Generer rapporter**: Inkluder kort, statistikker og visualiseringer

***

## Arkivering og sikkerhedskopiering

### Anbefalet sikkerhedskopieringsstrategi

**Hvad skal gemmes:**

* ✅ **Originale RAW/JPG-billeder** – Arkiver på separat drev/cloud
* ✅ **Behandlede output** – Opbevar kalibrerede billeder og indekser
* ✅ **Projektfil** – Indeholder alle indstillinger til genbehandling, hvis det er nødvendigt
* ✅ **Fejlfindingslog** – Dokumenterer behandlingsdetaljer
* ✅ **Kalibreringsmålbilleder** – Til verifikation og genbehandling

**Anbefalinger til opbevaring:**

* **Øjeblikkelig sikkerhedskopi**: Ekstern harddisk
* **Langtidsarkiv**: Cloud-opbevaring (Google Drive, Dropbox osv.)
* **Kritiske data**: Opbevar 2-3 kopier på forskellige steder

***

## Næste behandlingskørsler

### Genbrug af projektindstillinger

Hvis du behandler lignende datasæt i fremtiden:

1. **Gem projektmal** (hvis ikke allerede gjort)
2. **Opret nyt projekt** ved hjælp af gemt mal
3. **Importer nye billeder**
4. **Behandl** med identiske indstillinger for konsistens

### Batchbehandling af flere sessioner

For flere sessioner/datasæt:

**Mulighed 1: GUI - Flere projekter**

* Opret separate projekter for hver session
* Brug ensartede skabelonindstillinger
* Behandl én ad gangen

**Mulighed 2: Chloros CLI (kun Chloros+)**

* Automatiser batchbehandling
* Behandl flere mapper med scripts
* Se [CLI-dokumentation](../CLI.md)

**Mulighed 3: Python SDK (kun Chloros+)**

* Programmatisk kontrol
* Integration med analysepipelines
* Se [API-dokumentation](../api-python-sdk.md)

***

## Fejlfinding efter behandling

### Genbehandling med andre indstillinger

Hvis resultaterne ikke er tilfredsstillende:

1. Gem de originale billeder (slet dem aldrig)
2. Åbn det samme projekt i Chloros
3. Juster indstillingerne i panelet Projektindstillinger
4. Behandl igen – resultaterne overskriver de tidligere resultater

### Behandling af en delmængde af billeder

Sådan genbehandles kun bestemte billeder:

1. Opret et nyt projekt
2. Importer kun de billeder, der skal genbehandles
3. Brug den samme indstillingsskabelon
4. Behandl et mindre datasæt

### Få hjælp

Hvis du støder på problemer:

* 📧 **E-mail**: info@mapir.camera (inkluder fejlfindingslog)
* 🌐 **Support**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Ofte stillede spørgsmål](../faq.md)
* 📖 **Dokumentation**: [Chloros Manual](../)

***

## Resumé: Komplet arbejdsgang

Du har nu gennemført hele Chloros-behandlingsarbejdsgangen:

1. ✅ **Oprettet projekt** - Se [Projekter](../projects.md)
2. ✅ **Tilføjet filer** - Se [Tilføjelse af filer](adding-files-to-a-project.md)
3. ✅ **Justerede indstillinger** - Se [Justering af projektindstillinger](adjusting-project-settings.md)
4. ✅ **Markerede mål** - Se [Valg af målbilleder](choosing-target-images.md)
5. ✅ **Startet behandling** - Se [Start af behandlingen](starting-the-processing.md)
6. ✅ **Overvåget fremskridt** - Se [Overvågning af behandlingen](monitoring-the-processing.md)
7. ✅ **Gennemgåede resultater** - Denne side

**Dine kalibrerede, reflektanskorrigerede multispektrale billeder er klar til analyse!**

***

## Yderligere ressourcer

### Avancerede funktioner

* [**Billedfremviser**](../image-viewer-gui/opening-an-image-full-screen.md) - Interaktiv visualisering og analyse
* [**Indeks/LUT Sandkasse**](../image-viewer-gui/index-lut-sandbox.md) - Brugerdefineret indekstest
* [**Multispektrale indeksformler**](../project-settings/multispectral-index-formulas.md) - Komplet indeksreference

### Automatisering og integration

* [**CLI-dokumentation**](../CLI.md) – Batchbehandling via kommandolinjen
* [**Python SDK**](../api-python-sdk.md) - Programmatisk automatisering
* [**Chloros+ Funktioner**](../#chloros) - Avancerede behandlingsfunktioner

### Support og læring

* [**FAQ**](../faq.md) - Svar på almindelige spørgsmål
* [**Kalibreringsmål**](../calibration-targets.md) - Forståelse af reflektanskalibrering
* [**Understøttede kameraer**](../supported-cameras.md) - Kompatibel hardware

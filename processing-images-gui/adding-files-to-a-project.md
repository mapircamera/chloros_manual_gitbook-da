# Tilføjelse af filer til et projekt

Når du har oprettet eller åbnet et projekt i Chloros, er det næste trin at tilføje dine multispektrale billeder for at påbegynde behandlingen. Fanen »<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">« i filbrowseren gør det nemt at importere billeder og administrere dit datasæt.

## Adgang til filbrowseren

1. Åbn eller opret et projekt i Chloros
2. Klik på ikonet **Filbrowser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> i venstre sidepanel
3. Filbrowser-panelet viser dit projekts filliste

{% hint style="info" %}
**Understøttede filtyper**:

* **Survey3W / Survey3N**: RAW+JPG-par og JPG-billeder (RAW+JPG anbefales)
* **LATTICE**: `.tif` / `.tiff`-optagelser — optaget via Chloros-kamerastyring eller via en LATTICE-hub
* **Lyssensordata**: `.daq`-optagelser (DAQ-U/M/E) og DAQ-M `.csv`-nedstrålingslogfiler — importeret sammen med billederne for at udføre reflektanskalibrering
{% endhint %}

***

## Tilføjelse af billeder til dit projekt

Der er to primære måder at tilføje billeder til dit projekt på:

### Metode 1: Tilføj filer

Brug denne indstilling til at importere enkelte billedfiler eller et lille udvalg af filer.

1. Klik på knappen **&quot;Tilføj filer&quot;** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> øverst i panelet Filbrowser
2. Naviger til den mappe, der indeholder dine billeder
3. Vælg en eller flere billedfiler (hold **Ctrl** nede for at vælge flere filer)
4. Klik på **&quot;Åbn&quot;** for at importere de valgte filer

### Metode 2: Tilføj mappe

Brug denne indstilling til at importere alle billeder fra en mappe på én gang. Du kan vælge **flere mapper** i én dialogboks.

1. Klik på knappen **&quot;Tilføj mappe&quot;** (<img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line">) øverst i panelet Filbrowser
2. Naviger til og vælg den eller de mapper, der indeholder billederne fra din optagelsessession
3. Klik på **&quot;Vælg mappe&quot;** for at importere alle understøttede billeder

{% hint style="info" %}
**Filer, der ikke kan indlæses, rapporteres.** Hvis en mappe indeholder filer, som Chloros genkender, men ikke kan indlæse, får du en advarsel – billeder forsvinder ikke ubemærket fra gitteret.
{% endhint %}

***

## Import af LATTICE-optagelsesmapper

LATTICE-optagelser gemmes med **én undermappe pr. eksportniveau** — for eksempel `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — med den tilhørende nedstrømsfil `.daq` i rodmappen:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Vælg »Tilføj mappe« i roden til optagelserne** (`output/` ovenfor). Når den valgte mappe ikke selv indeholder billeder, men har undermapper, går Chloros automatisk ned i dem — undermapperne på det pågældende niveau og rodmappen `.daq` hentes alle på én gang.**Sådan importeres optagelser:*** Hver optagelse importeres som et **enkelt billede**, grupperet efter optagelse (ikke én post pr. niveau). De øvrige niveauer i den samme optagelse vises som visningstilstande for det pågældende billede.
* **Behandlingen starter altid fra det rå billede.** De øvrige niveauer kan ses, men kun `raw` sendes nogensinde gennem behandlingskæden — genbehandling af et allerede behandlet produkt ville medføre dobbelt anvendelse af korrektioner, så Chloros afvises. En genimporteret eksport kan aldrig indtage en optagelses rå-plads.
* En optagelsesmappe gemmes **uden** rå-import og vises normalt, men behandlingen springer den over og angiver dette i loggen. (Flagget CLI `--input-level` kan tvinge et indgangspunkt i dette tilfælde — se [CLI-referencen](../reference/cli-reference.md#what-a-captures-folder-looks-like).)**LATTICE-hub-sessioner** importeres på samme måde: Vælg »Add Folder« (Tilføj mappe) til den sessionsmappe, der er kopieret fra hubben (den indeholder `raw/` samt `previews/`), sammen med eventuelle DAQ-M `.csv` downwelling-logfiler. Hvis kameraets eller DAQ&#x27;ens kalibrering endnu ikke er gemt i cachen på din maskine, henter Chloros den automatisk via serienummeret ved import (kræver internetforbindelse én gang).***

## Forståelse af filbrowser-tabellen

Når billederne er importeret, vises de i en tabel med følgende kolonner:

### Filnavn

* Originalt filnavn fra kameraet
* Bevarer kameraets navngivningskonvention (f.eks. IMG\_0001.RAW eller capture\_20260816\_101500\_SN213800234\_raw.tif)

### Tidsstempel

* Dato og klokkeslæt for, hvornår billedet blev taget
* Udledt af billedets EXIF-metadata
* Anvendes til lyssensormatchning, PPK-synkronisering og planlægning af kalibreringsmål

### Kameramodel

* Automatisk registreret kamera- og filterkonfiguration
* Eksempler på Survey3: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* LATTICE-eksempler: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Anvendes til at anvende de korrekte behandlingsprofiler

### Målkolonne (afkrydsningsfelt)

* Markér dette afkrydsningsfelt for billeder, der indeholder kalibreringsmål
* Når mindst ét billede er markeret, **scannes kun de markerede billeder** for mål
* Se [Valg af målbilleder](choosing-target-images.md) for detaljer

### Visning af billedmetadata

Ved at klikke på vippeknappen i øverste højre hjørne over tabellen vises det valgte billedes metadata i billedgitterområdet.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Lyssensorfiler i dit projekt

* `.daq`- og `.csv`-filer vises i filbrowserlisten, men er ikke billeder, man kan klikke på — de leverer nedadgående strålingsintensitet til reflektanskalibrering.
* Hver importeret `.daq`/`.csv` vises under **Projektindstillinger → DAQ-lyssensor**, hvor du kan gennemgå den gældende korrektion for diffusorhætten for hver fil. Se [Justering af projektindstillinger](adjusting-project-settings.md).
* Optagelser, du foretager i fanen **Lyssensorer**, føjes automatisk til det åbne projekt — der er ikke behov for manuel import.***

## Administration af filer i dit projekt

### Fjernelse af filer

Sådan fjerner du uønskede billeder fra dit projekt:

1. Vælg et eller flere billeder i tabellen i filbrowseren
2. Klik på knappen **&quot;Fjern valgte&quot;** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Bekræft fjernelsen (filerne slettes ikke fra disken, men fjernes kun fra projektet)

### Sortering og filtrering

* **Sorter efter kolonne**: Klik på en vilkårlig kolonneoverskrift for at sortere billederne
* **Sortering efter tidsstempel**: Nyttigt til at organisere kronologiske optagelsessekvenser
* **Filter efter kameramodel**: Gruppér billeder efter kameratype, hvis du bruger flere kameraer***

## Billedforhåndsvisning

### Visning af fuldt billede

Klik på en vilkårlig billedminiature i filbrowseren for at vise det i det primære forhåndsvisningsområde:

1. Billedet vises i det centrale forhåndsvisningspanel
2. Brug zoomknapperne til at se billeddetaljer
3. Naviger mellem billederne ved hjælp af piletasterne

### Hurtig navigation

* **Forrige billede**: Klik på venstre pil eller tryk på ←-tasten
* **Næste billede**: Klik på højre pil eller tryk på →-tasten
* **Zoom ind/ud**: Brug musehjulet eller zoomknapperne
* **Panorer**: Klik og træk på billedet, når der er zoomet ind***

## Håndtering af dubletter

Chloros registrerer og ignorerer automatisk dubletter:

* Filer med identiske filnavne springes over
* Forhindrer utilsigtet dobbeltbehandling
* Der vises en advarselsmeddelelse, når der registreres dubletter

{% hint style="warning" %}
**Vigtigt**: Omdøb eller rediger ikke dine originale billedfiler før import. Chloros er afhængig af originale filnavne og metadata for korrekt behandling.
{% endhint %}

***

## Datasæt med blandede kameraer

Hvis dit projekt indeholder billeder fra flere MAPIR-kameraer:

1. Chloros registrerer automatisk hver kameramodel — Survey3, LATTICE eller en blanding
2. Hver kameratype behandles med den relevante kalibreringsprofil
3. Filbrowseren viser kameramodellen i kolonnen »Kameramodel«
4. Hvert kamera får sin egen udgående mappestruktur, når det behandles

**Eksempelscenarier**: Survey3W RGN + Survey3N OCN opsætning med to kameraer, eller et LATTICE-array med et RGB-masterkamera og flere smalbåndsmoduler***

## Bedste praksis

### Organiser før import

* Opbevar kalibreringsmålbillederne i samme mappe som undersøgelsesbillederne
* Opbevar hver optagelsessessions `.daq` / `.csv` lyssensorfiler sammen med den pågældende sessions billeder
* Bevar den oprindelige mappestruktur fra dit kamera/SD-kort/hub
* Bland ikke datasæt fra forskellige optagelsessessioner i ét projekt

### Filnavngivning

* Bevar de originale kamerafilnavne (IMG\_0001.RAW, capture\_..., osv.)
* Omdøb ikke filer før import
* De originale navne indeholder vigtige metadata

### Kalibreringsmålbilleder

* Medtag altid 1-2 kalibreringsmålbilleder pr. session (Survey3; for LATTICE kan en DAQ-optagelse erstatte disse — se [Valg af målbilleder](choosing-target-images.md))
* Tag billeder af målene før og efter optagelsessessionen
* Placer målene under de samme lysforhold som optagelsesområdet
* Marker målbillederne ved at markere afkrydsningsfeltet »Mål«

***

## Almindelige problemer og løsninger

### Billeder vises ikke efter import

**Mulige årsager:**

* Filformatet understøttes ikke (se listen over understøttede filtyper øverst på denne side)
* Billederne stammer fra kameraer, der ikke er af typen MAPIR (se [Understøttede kameraer](../supported-cameras.md))
* Filen er beskadiget, eller overførslen fra SD-kortet er ufuldstændig

**Løsning**: Kontroller, at filformatet og kameramodellen er kompatible, og se advarslen ved filindlæsning for at finde de nøjagtige filer, der mislykkedes

### Kameramodel ikke registreret

**Mulige årsager:**

* Ændrede EXIF-metadata
* Billeder redigeret i ekstern software
* Ufuldstændig filoverførsel

**Løsning**: Importer de originale, uændrede filer fra kameraet/SD-kortet igen

### Manglende tidsstempler

**Mulige årsager:**

* Kameraets ur er ikke indstillet korrekt
* EXIF-data er fjernet af ekstern software

**Løsning**: Kontroller, at kameraets tidsindstillinger var korrekte under optagelsen

### Genåbnet projekt rapporterer manglende filer

Hvis kildefilerne er blevet flyttet eller slettet, siden projektet sidst blev åbnet, fortæller Chloros dig, **hvilke** filer der mangler, i stedet for at åbne et tomt gitter. Gendan filerne på deres oprindelige stier, eller fjern de manglende poster og importer dem igen.***

## Næste trin

Når dine filer er importeret:

1. **Gennemgå fillisten** – Sørg for, at alle billeder er indlæst korrekt
2. **Kontroller kameramodeller** – Kontroller, at kameraerne er genkendt korrekt
3. **Marker målbilleder** – Se [Valg af målbilleder](choosing-target-images.md)
4. **Juster indstillingerne** – Konfigurer behandlingsindstillingerne i [Projektindstillinger](adjusting-project-settings.md)
5. **Start behandlingen** – Se [Start af behandlingen](starting-the-processing.md)

Se [Justering af projektindstillinger](adjusting-project-settings.md) for detaljerede oplysninger om projektkonfiguration.

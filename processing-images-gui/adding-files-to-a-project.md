# Tilføjelse af filer til et projekt

Når du har oprettet eller åbnet et projekt i Chloros, er det næste trin at tilføje dine multispektrale billeder for at påbegynde behandlingen. Filbrowseren<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> gør det nemt at importere billeder og administrere dit datasæt.

## Adgang til filbrowseren

1. Åbn eller opret et projekt i Chloros
2. Klik på ikonet **Filbrowser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> i venstre sidepanel
3. Filbrowseren viser dit projekts filliste

{% hint style="info" %}
**Understøttede filtyper**: Chloros understøtter RAW+JPG- og JPG-billedfiler fra MAPIR Survey3W og Survey3N kameraer. Kun RAW+JPG anbefales.
{% endhint %}

***

## Tilføjelse af billeder til dit projekt

Der er to primære måder at tilføje billeder til dit projekt:

### Metode 1: Tilføj filer

Brug denne indstilling til at importere individuelle billedfiler eller et lille udvalg af filer.

1. Klik på knappen **&quot;Tilføj filer&quot;** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> øverst i filbrowserpanelet
2. Naviger til den mappe, der indeholder dine billeder
3. Vælg en eller flere billedfiler (hold **Ctrl** nede for at vælge flere filer)
4. Klik på **&quot;Åbn&quot;** for at importere de valgte filer

### Metode 2: Tilføj mappe

Brug denne indstilling til at importere alle billeder fra en mappe på én gang.

1. Klik på knappen **&quot;Tilføj mappe&quot;** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> øverst i filbrowserpanelet.
2. Naviger til og vælg den mappe, der indeholder dine billeder fra optagelsessessionen.
3. Klik på **&quot;Vælg mappe&quot;** for at importere alle understøttede billeder fra den pågældende mappe.

***

## Forståelse af filbrowser-tabellen

Når billederne er importeret, vises de i en tabel med følgende kolonner:

### Filnavn

* Originalt filnavn fra kameraet
* Overholder kameraets navngivningskonvention (f.eks. IMG\_0001.RAW)

### Tidsstempel

* Dato og klokkeslæt for billedets optagelse
* Uddrag fra billedets EXIF-metadata
* Anvendes til PPK-synkronisering og kalibreringsmåldetektering

### Kameramodel

* Automatisk detekteret kamera- og filterkonfiguration
* Eksempler: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Bruges til at anvende korrekte behandlingsprofiler

### Målkolonne (afkrydsningsfelt)

* Marker dette afkrydsningsfelt for billeder, der indeholder kalibreringsmål
* Fremskynder måldetektering betydeligt under behandlingen
* Se [Valg af målbilleder](choosing-target-images.md) for detaljer

### Visning af billedmetadata

Ved at klikke på skifteknappen i øverste højre hjørne over tabellen vises det valgte billedes metadata i billedgitterområdet.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Administration af filer i dit projekt

### Fjernelse af filer

Sådan fjerner du uønskede billeder fra dit projekt:

1. Vælg et eller flere billeder i tabellen Filbrowser
2. Klik på knappen **&quot;Fjern valgte&quot;** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> knappen.
3. Bekræft fjernelsen (filerne slettes ikke fra disken, men fjernes kun fra projektet).

### Sortering og filtrering

* **Sorter efter kolonne**: Klik på en kolonneoverskrift for at sortere billederne.
* **Sorter efter tidsstempel**: Nyttigt til at organisere kronologiske optagelsessekvenser.
* **Kamerafiltermodel**: Gruppér billeder efter kameratype, hvis du bruger flere kameraer.

***

## Billedforhåndsvisning

### Visning af hele billedet

Klik på en billedminiature i filbrowseren for at vise det i det primære forhåndsvisningsområde:

1. Billedet vises i det midterste forhåndsvisningspanel
2. Brug zoomkontrolknapperne til at se billeddetaljer
3. Naviger mellem billederne ved hjælp af piletasterne

### Hurtig navigation

* **Forrige billede**: Klik på venstre pil eller tryk på ←-tasten
* **Næste billede**: Klik på højrepilen eller tryk på →-tasten
* **Zoom ind/ud**: Brug musehjulet eller zoomknapperne
* **Panorer**: Klik og træk på billedet, når det er zoomet ind

***

## Håndtering af duplikerede filer

Chloros registrerer og ignorerer automatisk duplikerede filer:

* Filer med identiske filnavne springes over
* Forhindrer utilsigtet dobbeltbehandling
* Der vises en advarselsmeddelelse, når der opdages duplikater

{% hint style="warning" %}
**Vigtigt**: Omdøb eller rediger ikke dine originale billedfiler, før du importerer dem. Chloros er afhængig af originale filnavne og metadata for at kunne behandle dem korrekt.
{% endhint %}

***

## Blandede kameradataset

Hvis dit projekt indeholder billeder fra flere MAPIR-kameraer:

1. Chloros registrerer automatisk hver kameramodel
2. Hver kameratype behandles med den relevante kalibreringsprofil
3. Filbrowseren viser kameramodellen i kolonnen Kameramodel
4. Behandlingen anvender de korrekte indstillinger for hver kameratype

**Eksempel på scenario**: Survey3W RGN + Survey3N OCN opsætning med to kameraer

***

## Bedste praksis

### Organiser før import

* Opbevar kalibreringsmålbilleder i samme mappe som undersøgelsesbilleder
* Bevar den oprindelige mappestruktur fra dit kamera/SD-kort
* Bland ikke datasæt fra forskellige sessioner i ét projekt

### Filnavngivning

* Bevar de oprindelige kamerafilnavne (IMG\_0001.RAW osv.)
* Omdøb ikke filer før import
* De originale navne indeholder vigtige metadata.

### Kalibreringsmålbilleder

* Inkluder altid 1-2 kalibreringsmålbilleder pr. session.
* Tag billeder af målene før og efter optagelsessessionen.
* Placer målene under de samme lysforhold som optagelsesområdet.
* Marker målbillederne ved hjælp af afkrydsningsfeltet Mål for at fremskynde behandlingen.

***

## Almindelige problemer og løsninger

### Billeder vises ikke efter import

**Mulige årsager:**

* Filformatet understøttes ikke (kun RAW+JPG og JPG fra MAPIR-kameraer)
* Billederne er fra ikke-MAPIR-kameraer (se [Understøttede kameraer](../supported-cameras.md))
* Fil er beskadiget eller overførslen fra SD-kortet er ufuldstændig

**Løsning**: Kontroller filformatet og kameraets kompatibilitet

### Kameramodel registreres ikke

**Mulige årsager:**

* Ændrede EXIF-metadata
* Billeder redigeret i ekstern software
* Ufuldstændig filoverførsel

**Løsning**: Importer de originale, uændrede filer fra kameraet/SD-kortet igen

### Manglende tidsstempler

**Mulige årsager:**

* Kameraets ur er ikke indstillet korrekt
* EXIF-data fjernet af ekstern software

**Løsning**: Kontroller, at kameraets tidsindstillinger var korrekte under optagelsen

***

## Næste trin

Når dine filer er importeret:

1. **Gennemgå fillisten** – Sørg for, at alle billeder er indlæst korrekt
2. **Kontroller kameramodeller** – Kontroller, at kameraet er registreret korrekt
3. **Marker målbilleder** – Se [Valg af målbilleder](choosing-target-images.md)
4. **Juster indstillinger** – Konfigurer behandlingsindstillinger i [Projektindstillinger](adjusting-project-settings.md)
5. **Start behandlingen** – Se [Start af behandlingen](starting-the-processing.md)

For detaljerede oplysninger om projektkonfiguration, se [Justering af projektindstillinger](adjusting-project-settings.md).

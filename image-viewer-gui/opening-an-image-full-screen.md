# Åbning af et billede i fuld skærm

Chloros Image Viewer tilbyder en dedikeret fuldskærmsgrænseflade til visning, analyse og redigering af dine multispektrale billeder. Uanset om du ser originale billeder eller bearbejdede resultater, tilbyder Image Viewer kraftfulde værktøjer til inspektion og analyse.

## Adgang til billedviseren

### Fra filbrowseren

Den mest almindelige måde at åbne et billede i billedviseren på:

1. Sørg for, at du er på fanen **Filbrowser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klik på en vilkårlig **billedminiature** i billedgitteret
3. Billedet åbnes i **hovedvisningsområdet** (midten af skærmen)
4. Billedet er nu indlæst og klar til visning i fuld skærm

### Åbning af fanen Image Viewer

Når et billede er indlæst i visningsområdet:

1. Klik på ikonet **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> i venstre sidepanel
2. Fanen Billedfremviser åbnes og viser det valgte billede i fuld skærm
3. Avancerede visnings- og analyseværktøjer bliver tilgængelige i venstre sidepanel

***

## Oversigt over Billedfremviser-grænsefladen

### Hovedvisningsområde

Den største del af skærmen viser dit billede:

* **Fuld opløsning**: Billeder vises i oprindelig opløsning
* **Zoombar**: Brug kontrolelementer eller musehjulet til at zoome
* **Panorerbar**: Klik og træk for at flytte rundt, når der er zoomet
* **Billedformat bevares**: Billeder skaleres proportionalt***

## Visningsindstillinger

### Grundlæggende billednavigation

#### Gennemse billeder

Naviger gennem dit billedsæt ved hjælp af tastaturgenveje eller knapper:

* **Næste billede**: Klik på →-knappen eller tryk på**→** (højrepil)-tasten
* **Forrige billede**: Klik på ←-knappen eller tryk på**←** (venstrepil)-tasten
* **Gå til et bestemt billede**: Gå tilbage til filbrowseren og klik på den ønskede miniature

#### Zoomkontroller

Juster forstørrelsen for at se billeddetaljer:

**Zoom ind:*** Klik på **+** (plus)-knappen
* Tryk på **+**- eller**=**-tasten
* Rul musehjulet **opad**

**Zoom ud:*** Klik på **−** (minus)-knappen
* Tryk på **−** (minus)-tasten
* Rul musehjulet **nedad**

#### Panorering ved zoom

Når der er zoomet ind ud over skærmstørrelsen:

1. Flyt musemarkøren over billedet
2. Klik og **hold venstre museknap nede**

3.**Træk** for at flytte billedet rundt
4. Slip for at stoppe panoreringen

**Alternativ**: Brug piletasterne til at panorere i små trin***

## Inspektion af pixelværdier

### Visning af pixelværdier ved markøren

Når du bevæger musemarkøren over billedet, vises pixelværdierne i realtid:**Placering af værdivisning:*** **Flydende tal og rød linje i indeks-LUT-gradientlegenden til højre*** **Når der zoomes yderligere ind, vises en flydende værdi nær markøren og den fremhævede pixel*** Viser værdier for pixel **under markøren eller den fremhævede*** Opdateres, når du bevæger musen

***

## Billedtyper, du kan se

### JPG

**JPG-billeder fra kamera:**

* Viser JPG-data som forhåndsvisning
* Viser originale, ukorrigerede værdier
* Nyttigt til at kontrollere billedkvaliteten før behandling

### RAW (Original)

### RAW (Reflektans)

**Efter behandling:**

* Vignettering korrigeret
* Reflektans kalibreret
* Multibånd TIFF (Red, Green, NIR osv.)
* Videnskabelige data klar til analyse

### RAW (Indeks)

**NDVI, NDRE, GNDVI osv. (\_NDVI.tif-filer):**

* Enkeltbåndsgråtonebilleder
* Pixelværdier repræsenterer indeksberegningsresultater
* Interval typisk -1 til +1 for normaliserede indekser
* Kan anvende farve-LUT&#x27;er til visualisering

***

## Anvendelse af indeks og LUT

Anvend multispektrale indekser og farve-Look-Up-tabeller:

1. Find **Index/LUT Sandbox**i**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sidebjælke
2. Vælg vegetationsindeks (NDVI, NDRE osv.)
3. Vælg multispektral formel, eller opret din egen brugerdefinerede (kun Chloros+)
4. Anvend farve-LUT-gradient til visualisering
5. Juster værdiintervaller og tærskler

Se [Index/LUT Sandbox](index-lut-sandbox.md) for detaljerede instruktioner.

***

## Tastaturgenveje

### Navigation

* **→** (højrepil): Næste billede
* **←** (venstrepil): Forrige billede
* **Home**: Første billede i listen
* **End**: Sidste billede i listen

### Zoom

* **+**eller**=**: Zoom ind
* **−**: Zoom ud
* **Musens rullehjul**: Zoom ind/ud***

### Verificering af indeksberegninger

Kontroller, at indekserne er beregnet korrekt:

1. Åbn NDVI eller et andet indeksbillede
2. Kontroller vegetationsområder:
   * **NDVI**: Bør vise 0,4-0,9 for sunde planter
   * **NDRE**: Højere værdier for kraftig vækst
   * **GNDVI**: Ligner NDVI, men er klorofylfølsom
3. Kontroller ikke-vegetation:
   * **Jord**: Nær 0 eller let negativ
   * **Vand**: Negative værdier (-0,5 til 0)***

## Fejlfinding ved visningsproblemer

### Billedet kan ikke åbnes

**Mulige årsager:**

* Filen er blevet beskadiget under behandlingen
* Filformatet understøttes ikke
* Utilstrækkelig hukommelse til store billeder

**Løsninger:**

1. Prøv at åbne i en ekstern viewer for at kontrollere filens integritet
2. Kontroller, at filformatet matcher den forventede type
3. Luk andre programmer for at frigøre hukommelse
4. Prøv et mindre/andet billede

### Sort eller hvid billedvisning

**Mulige årsager:**

* Værdiinterval uden for visningskapaciteten
* 32-bit float-billede med usædvanlige værdier
* Fejl i indeksberegningen

**Løsninger:**

1. Kontroller pixelværdierne – hvis de alle er meget lave eller meget høje, skal du justere visningsområdet
2. Prøv at åbne i QGIS eller lignende med automatisk justering af området
3. Kontroller fejlloggen fra behandlingen for fejl

### Pixelværdierne ser forkerte ud

**Mulige årsager:**

* Visning af forkert billede (original vs. behandlet)
* Kalibreringen blev ikke anvendt korrekt
* Lyssensordata blev ikke inkluderet i inputtet
* Procenttilstand blev skiftet forkert

**Løsninger:**

1. Kontroller, at du ser det behandlede output (tjek filnavnets endelse)
2. Kontroller status for procenttilstandsknappen
3. Sammenlign med billeder, du ved er korrekte, fra samme datasæt

***

## Næste trin

Nu hvor du kan se billeder i fuld skærm:

* [**Billedlag**](image-layers.md) – Lær om multibåndsvisualisering
* [**Indeks/LUT-sandkasse**](index-lut-sandbox.md) – Anvend brugerdefinerede indekser og farvekortlægning
* [**Formler til multispektrale indekser**](../project-settings/multispectral-index-formulas.md) – Få indsigt i de tilgængelige indekser

Se følgende for arbejdsgangen ved behandling:

* [**Behandling af billeder (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Komplet vejledning til behandling

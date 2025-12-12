# Åbning af et billede i fuld skærm

Chloros Image Viewer har en dedikeret fuldskærmsgrænseflade til visning, analyse og redigering af dine multispektrale billeder. Uanset om du viser originale billeder eller behandlede resultater, tilbyder Image Viewer kraftfulde værktøjer til inspektion og analyse.

## Adgang til billedviseren

### Fra filbrowseren

Den mest almindelige måde at åbne et billede i billedviseren:

1. Sørg for, at du er i fanen **Filbrowser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klik på en **billedminiature** i billedgitteret
3. Billedet åbnes i **hovedforhåndsvisningsområdet** (midten af skærmen)
4. Billedet er nu indlæst og klar til visning i fuld skærm

### Åbning af fanen Billedfremviser

Når et billede er indlæst i forhåndsvisningsområdet:

1. Klik på ikonet **Billedfremviser** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> i venstre sidepanel
2. Fanen Billedfremviser åbnes og viser det valgte billede i fuld skærm
3. Avancerede visnings- og analyseværktøjer bliver tilgængelige i venstre sidepanel

***

## Oversigt over billedfremviserens grænseflade

### Hovedvisningsområde

Den største del af skærmen viser dit billede:

* **Fuld opløsning**: Billeder vises i oprindelig opløsning
* **Zoombar**: Brug kontrolelementer eller mushjulet til at zoome
* **Panorerbar**: Klik og træk for at flytte rundt, når der er zoomet
* **Billedformat bevaret**: Billeder skaleres proportionalt

***

## Visningsindstillinger

### Grundlæggende billednavigation

#### Gennemse billeder

Naviger gennem dit billedsæt ved hjælp af tastaturgenveje eller knapper:

* **Næste billede**: Klik på →-knappen eller tryk på **→** (højrepil)-tasten
* **Forrige billede**: Klik på ←-knappen eller tryk på **←** (venstrepil)-tasten
* **Gå til et bestemt billede**: Gå tilbage til filbrowseren og klik på den ønskede miniature

#### Zoomknapper

Juster forstørrelsen for at se billeddetaljer:

**Zoom ind:**

* Klik på knappen **+** (plus)
* Tryk på tasten **+** eller **=**
* Rul musehjulet **op**

**Zoom ud:**

* Klik på knappen **−** (minus)
* Tryk på tasten **−** (minus)
* Rul musehjulet **ned**

**Tilpas til skærm:**

* Klik på knappen **↔** (Tilpas)
* Tryk på tasten **0** (nul)
* Dobbeltklik på billedet

#### Panorering ved zoom

Når der er zoomet ud over skærmstørrelsen:

1. Flyt musemarkøren over billedet
2. Klik og **hold venstre museknap nede**
3. **Træk** for at flytte billedet rundt
4. Slip for at stoppe panorering

**Alternativ**: Brug piletasterne til at panorere i små trin

***

## Inspektion af pixelværdi

### Visning af pixelværdier ved markøren

Når du flytter musemarkøren over billedet, vises pixelværdierne i realtid:

**Værdiens placering:**

* **Flydende tal og rød linje i højre side af indeks LUT-gradientlegende**
* **Når du zoomer yderligere ind, vises den flydende værdi nær markøren og den fremhævede pixel**
* Viser værdier for pixel **under markøren eller fremhævet**
* Opdateres, når du bevæger musen

***

## Billedtyper, du kan se

### Originalbilleder (før behandling)

**RAW + JPG-billeder fra kameraet:**

* Viser RAW-data som forhåndsvisning
* Viser originale, ukorrigerede værdier
* Nyttigt til at kontrollere billedkvaliteten før behandling

### Kalibrerede reflektansbilleder

**Efter behandling:**

* Vignette korrigeret
* Reflektans kalibreret
* Multibånd TIFF (Red, Green, NIR osv.)
* Videnskabelige data klar til analyse

### Indeksbilleder

**NDVI, NDRE, GNDVI osv. (\_NDVI.tif-filer):**

* Enkeltbånds gråtonebilleder
* Pixelværdier repræsenterer indeksberegningsresultater
* Interval typisk -1 til +1 for normaliserede indekser
* Kan anvende farve-LUT&#x27;er til visualisering

***

## Indeks- og LUT-anvendelse

Anvend multispektrale indekser og farve-Look-Up-tabeller:

1. Find **Index/LUT Sandbox** i **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sidebjælke
2. Vælg vegetationsindeks (NDVI, NDRE osv.)
3. Vælg multispektral formel, eller opret din egen brugerdefinerede formel (kun Chloros+)
4. Anvend farve-LUT-gradient til visualisering
5. Juster værdiintervaller og tærskler

Se [Index/LUT Sandbox](index-lut-sandbox.md) for detaljerede instruktioner.

***

## Tastaturgenveje

### Navigation

* **→** (højrepil): Næste billede
* **←** (venstrepil): Forrige billede
* **Home**: Første billede på listen
* **End**: Sidste billede på listen

### Zoom

* **+** eller **=**: Zoom ind
* **−**: Zoom ud
* **0** (nul): Tilpas til skærmen
* **Mushjul**: Zoom ind/ud

### Visningskontroller

* **P**: Skift til pixelprocenttilstand
* **L**: Skift til lagpanel
* **Esc**: Luk fuldskærm eller vend tilbage til filbrowseren

### Andet

* **Ctrl+S**: Gem det aktuelle billede
* **F**: Fuldskærmstilstand (hvis tilgængelig)

***

### Bekræftelse af indeksberegninger

Kontroller, at indekserne er beregnet korrekt:

1. Åbn NDVI eller et andet indeksbillede
2. Kontroller vegetationsområderne:
   * **NDVI**: Skal vise 0,4-0,9 for sunde planter
   * **NDRE**: Højere værdier for kraftig vækst
   * **GNDVI**: Ligner NDVI, men er klorofylfølsom
3. Kontroller ikke-vegetation:
   * **Jord**: Tæt på 0 eller let negativ
   * **Vand**: Negative værdier (-0,5 til 0)

***

## Fejlfinding ved visningsproblemer

### Billedet kan ikke åbnes

**Mulige årsager:**

* Filen er beskadiget under behandlingen
* Ikke-understøttet filformat
* Utilstrækkelig hukommelse til store billeder

**Løsninger:**

1. Prøv at åbne i ekstern viewer for at kontrollere filens integritet
2. Kontroller, at filformatet svarer til den forventede type
3. Luk andre programmer for at frigøre hukommelse
4. Prøv med et mindre/andet billede

### Sort eller hvid billedvisning

**Mulige årsager:**

* Værdiområdet ligger uden for skærmens kapacitet
* 32-bit float-billede med usædvanlige værdier
* Indeksberegningsfejl

**Løsninger:**

1. Kontroller pixelværdierne – hvis alle er meget lave eller meget høje, skal du justere visningsområdet.
2. Prøv at åbne i QGIS eller lignende med automatisk justering af området.
3. Kontroller fejlfindingsloggen fra behandlingen for fejl.

### Pixelværdierne ser forkerte ud

**Mulige årsager:**

* Visning af forkert billede (original vs. behandlet)
* Kalibreringen blev ikke anvendt korrekt
* Lyssensordata blev ikke inkluderet i input
* Procenttilstand skiftet forkert

**Løsninger:**

1. Kontroller, at du ser behandlet output (kontroller filnavnesuffikset)
2. Kontroller status for procenttilstandsknappen
3. Sammenlign med kendte gode billeder fra samme datasæt

***

## Næste trin

Nu hvor du kan se billeder i fuld skærm:

* [**Billedlag**](image-layers.md) – Lær om multibåndvisualisering
* [**Indeks/LUT Sandkasse**](index-lut-sandbox.md) – Anvend brugerdefinerede indekser og farvekortlægning
* [**Multispektrale indeksformler**](../project-settings/multispectral-index-formulas.md) – Få indsigt i tilgængelige indekser

For arbejdsgangen for behandling, se:

* [**Behandling af billeder (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Komplet vejledning til behandling

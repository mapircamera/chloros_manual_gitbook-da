# Start af behandlingen

Når du har importeret dine billeder, markeret dine kalibreringsmål og konfigureret dine projektindstillinger, er du klar til at påbegynde behandlingen. Denne side guider dig gennem opstarten af Chloros-behandlingsprocessen.

## Tjekliste før behandling

Før du klikker på Start-knappen, skal du kontrollere, at alt er klar:

* [ ] **Filer importeret** – Alle billeder vises i filbrowseren
* [ ] **Målbilleder markeret** – Målkolonnen er markeret for kalibreringsbilleder
* [ ] **Kameramodeller registreret** – Kolonnen Kameramodel viser de korrekte kameraer
* [ ] **Indstillinger konfigureret** – Projektindstillingerne er gennemgået og justeret
* [ ] **Indekser valgt** – Ønskede multispektrale indekser tilføjet (hvis nødvendigt)
* [ ] **Eksportformat valgt** – Outputformat, der passer til din arbejdsgang

{% hint style="info" %}
**Tip**: Klik dig igennem et par billeder i filbrowseren for at kontrollere, at de er indlæst korrekt, inden du fortsætter med behandlingen.
{% endhint %}

***

## Start af behandlingen

### Find Start-knappen

Start/Afspil-knappen er placeret i den øverste header-bjælke i Chloros:

* Placering: Øverst i midten af vinduet
* Ikon: **Afspil/Start-knap** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Status: Knappen er aktiveret (lysende), når den er klar til behandling

### Klik for at starte

1. Klik på **Afspil/Start-knappen** i den øverste header
2. Behandlingen starter straks
3. Knappen bliver deaktiveret (grå) under behandlingen
4. Statusbjælken opdateres og viser behandlingsstatus

{% hint style="success" %}
**Behandlingen er startet**: Når der klikkes, håndterer Chloros automatisk alle behandlingstrin – måldetektering, debayering, kalibrering, indeksberegning og eksport.
{% endhint %}

***

## Forståelse af behandlingsmetoder

Chloros fungerer i to forskellige behandlingsmodi afhængigt af din licens:

### Gratis tilstand (sekventiel behandling)

**Tilgængelig for alle brugere**

**Sådan fungerer det:**

* Behandler billeder et ad gangen, sekventielt
* Enkelt-trådet drift
* Lavere hukommelsesforbrug

**Statusbjælken viser 2 trin:**

1.**Målregistrering** – Scanning efter kalibreringsmål
2. **Behandling** – Anvendelse af kalibrering og eksport af billeder**Behandlingstid:**

* Meget langsommere end Chloros+ parallel tilstand
* Egnet til små til mellemstore datasæt (&lt; 200 billeder)

### Chloros+-tilstand (parallel behandling)

**Kræver Chloros+-licens**

**Sådan fungerer det:**

* Behandler flere billeder samtidigt ved hjælp af en [4-trådet behandlingspipeline](../processing-architecture/processing-pipeline.md)
* [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) vælger automatisk den optimale strategi for din hardware
* GPU (CUDA)-acceleration med NVIDIA-grafikkort (desktop og Jetson)
* Skalerer fra en Jetson Nano (1 worker) til en desktop med 12 GB+ GPU (3-4 workers)

**Statusbjælken viser 4 trin** (svarende til de 4 pipeline-tråde):

1. **Detektering** (Tråd 1) – Find kalibreringsmål
2. **Analyse** (Tråd 2) – Undersøg billedmetadata og beregn kalibrering
3. **Kalibrering** (Tråd 3) – GPU-debayering, vignetteringskorrektion, indeksberegning
4. **Eksport** (Tråd 4) – Gemmer behandlede billeder og indekser**Interaktion med statusbjælken:*** **Hold musen** over bjælken for at se et detaljeret dropdown-panel med 4 trin
* **Klik** på statusbjælken for at fastfryse dropdown-panelet
* **Klik igen** for at frigøre og skjule panelet**Behandlingstid:**

* Betydeligt hurtigere end gratis tilstand
* Skalerer med antallet af CPU-kerner
* GPU-acceleration forbedrer hastigheden yderligere

{% hint style="info" %}
**Chloros+ Hastighed**: Parallel behandling kan være 5-10 gange hurtigere end sekventiel tilstand for store datasæt. Et projekt med 500 billeder, der tager 2 timer i gratis tilstand, kan afsluttes på 15-20 minutter med Chloros+.
{% endhint %}

***

## Hvad sker der under behandlingen

### Trin 1: Målregistrering

**Hvad Chloros gør:**

* Scanner markerede målbilleder (eller alle billeder, hvis ingen er markeret)
* Identificerer de 4 kalibreringspaneler i hvert mål
* Udtrækker reflektansværdier fra målpanelerne
* Registrerer målets tidsstempler til planlægning af kalibrering

**Varighed:** 1-30 sekunder (med markerede mål), 5-30+ minutter (umarkerede)

### Trin 2: Debayering (RAW-konvertering)

**Hvad Chloros gør:**

* Konverterer RAW-Bayer-mønsterdata til fulde RGB-billeder
* Anvender en demosaicing-algoritme af høj kvalitet
* Bevarer maksimal billedkvalitet og detaljer

**Varighed:** Varierer afhængigt af antal billeder og CPU-hastighed

### Trin 3: Kalibrering

**Hvad Chloros gør:*** **Vignettekorrektion**: Fjerner mørkningen ved kanterne
* **Reflektanskalibrering**: Normaliserer ved hjælp af målreflektansværdier
* Anvender korrektioner på tværs af alle bånd/kanaler
* Bruger passende kalibreringsmål for hvert billede baseret på tidsstempel

**Varighed:** Størstedelen af behandlingstiden

### Trin 4: Indeksberegning

**Hvad Chloros gør:**

* Beregner konfigurerede multispektrale indekser (NDVI, NDRE osv.)
* Anvender båndmatematik på kalibrerede billeder
* Genererer indeksbilleder for hvert valgt indeks

**Varighed:** Et par sekunder pr. billede

### Trin 5: Eksport

**Hvad Chloros gør:**

* Gemmer kalibrerede billeder i valgt format
* Eksporterer indeksbilleder med konfigurerede LUT-farver
* Skriver filer til undermapper for kameramodeller
* Bevarer originale filnavne med suffikser

**Varighed:** Varierer afhængigt af eksportformat og filstørrelse***

## Behandlingsadfærd

### Automatisk behandlingspipeline

Når den er startet, kører hele pipelinen automatisk:

* Ingen brugerinteraktion nødvendig
* Alle konfigurerede trin udføres i rækkefølge
* Statusopdateringer vises i realtid

### Computerforbrug under behandling

**Fri tilstand:**

* Relativt lav CPU-udnyttelse (enkelt-trådet)
* Computeren forbliver responsiv til andre opgaver
* Det er sikkert at minimere Chloros og arbejde i andre programmer

**Chloros+ Parallel tilstand:**

* Høj CPU-udnyttelse (multi-trådet, op til 16 kerner)
* Med GPU-acceleration: Højt GPU-forbrug
* Computeren kan være mindre responsiv under behandlingen
* Undgå at starte andre CPU-intensive opgaver

{% hint style="warning" %}
**Tip til ydeevne**: For at opnå den bedste Chloros+ ydeevne skal du lukke andre programmer og lade Chloros bruge alle systemressourcer.
{% endhint %}

### Behandlingen kan ikke sættes på pause

**Vigtige begrænsninger:**

* Når behandlingen er startet, kan den ikke sættes på pause
* Du kan afbryde behandlingen, men fremskridtet går tabt
* Delresultater gemmes ikke
* Skal genstartes fra begyndelsen, hvis den afbrydes

**Planlægningstip:** Ved meget store projekter bør du overveje at behandle i batches eller bruge CLI for bedre kontrol.***

## Overvågning af din behandling

Mens behandlingen kører, kan du:

* **Se statusbjælken** – Se den samlede færdiggørelsesprocent
* **Se den aktuelle fase** – Registrer, analyser, kalibrer eller eksporter
* **Tjekke fanen Log** – Se detaljerede behandlingsmeddelelser og advarsler
* **Få vist forhåndsvisning af færdige billeder** – Nogle eksportfiler kan vises under behandlingen

For detaljerede oplysninger om overvågning, se [Overvågning af behandlingen](monitoring-the-processing.md).

***

## Afbrydelse af behandlingen

Hvis du har brug for at stoppe behandlingen:

### Sådan afbrydes

1. Find **Stop/Annuller-knappen** (erstatter Start-knappen under behandlingen)
2. Klik på Stop-knappen
3. Behandlingen standses øjeblikkeligt
4. Delvise resultater kasseres

### Hvornår skal man annullere

**Gyldige grunde til at annullere:**

* Man har opdaget, at der er blevet brugt forkerte indstillinger
* Man har glemt at markere målbilleder
* Der er importeret forkerte billeder
* Systemet kører for langsomt eller reagerer ikke

**Efter annullering:**

* Gennemgå og rett eventuelle problemer
* Juster indstillingerne efter behov
* Genstart behandlingen fra begyndelsen
* For den bedste oplevelse skal du lukke Chloros helt og genstarte

{% hint style="warning" %}
**Ingen delresultater**: Annullering sletter al fremskridt. Chloros gemmer ikke delvist behandlede billeder.
{% endhint %}

***

## Estimater for behandlingstid

Den faktiske behandlingstid varierer meget afhængigt af:

* Antal billeder
* Billedopløsning
* RAW vs. JPG-indgangsformat
* Behandlingsmode (Free vs. Chloros+)
* CPU-hastighed og antal kerner
* GPU-tilgængelighed (kun Chloros+)
* Antal indekser, der skal beregnes
* Eksportformatets kompleksitet

### Grove skøn (Chloros+, 12 MP-billeder, moderne CPU)

| Antal billeder | Gratis tilstand | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 billeder   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 billeder  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 billeder  | 1-1,5 time | 20-30 min      | 10-15 min      |
| 500 billeder  | 2-3 timer   | 45-60 min      | 20-30 min      |
| 1000 billeder | 4-6 timer   | 1,5-2 timer      | 40-60 min      |

{% hint style="info" %}
**Første kørsel**: Den indledende behandling kan tage længere tid, da Chloros opbygger caches og profiler. Efterfølgende behandling af lignende datasæt vil være hurtigere.
{% endhint %}

***

## Almindelige problemer ved opstart

### Startknappen er deaktiveret (gråtonet)

**Mulige årsager:**

* Ingen billeder importeret
* Backend er ikke fuldt startet
* Forrige behandling kører stadig
* Projektet er ikke fuldt indlæst

**Løsninger:**

1. Vent på, at backend er fuldt initialiseret (tjek ikonet i hovedmenuen)
2. Kontroller, at billederne er importeret i filbrowseren
3. Genstart Chloros, hvis knappen stadig er deaktiveret
4. Tjek fejlloggen for fejlmeddelelser

### Behandlingen starter og mislykkes straks

**Mulige årsager:**

* Ingen gyldige billeder i projektet
* Beskadigede billedfiler
* Utilstrækkelig diskplads
* Utilstrækkelig hukommelse (RAM)

**Løsninger:**

1. Tjek fejlloggen <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> for fejlmeddelelser
2. Kontroller, at der er ledig diskplads
3. Prøv at behandle en mindre delmængde af billeder
4. Kontroller, at billederne ikke er beskadigede

### Advarsel &quot;Ingen mål fundet&quot;

**Mulige årsager:**

* Glemt at markere målbilleder
* Målbillederne indeholder ikke synlige mål
* Indstillingerne for måldetektering er for strenge

**Løsninger:**

1. Gennemgå [Valg af målbilleder](choosing-target-images.md)
2. Marker de relevante billeder i kolonnen &quot;Mål&quot;
3. Kontroller, at målene er synlige i de markerede billeder
4. Juster indstillingerne for måldetektering, hvis det er nødvendigt

***

## Tips til vellykket behandling

### Før du starter

1. **Test først med et lille udsnit** – Behandl 10-20 billeder for at kontrollere indstillingerne
2. **Kontroller ledig diskplads** – Sørg for, at der er 2-3 gange datasættets størrelse ledig
3. **Luk unødvendige programmer** – Frigør systemressourcer
4. **Kontroller målbilleder** – Se et eksempel på markerede mål for at sikre kvaliteten
5. **Gem projektet** – Projektet gemmes automatisk, men det er god praksis at gemme manuelt

### Under behandlingen

1. **Undgå, at systemet går i dvale** – Deaktiver strømbesparende tilstande
2. **Hold Chloros i forgrunden** – Eller i det mindste synligt i proceslinjen
3. **Overvåg fremskridtet lejlighedsvis** – Kontroller for advarsler eller fejl
4. **Undgå at køre andre ressourcekrævende programmer** – Især i Chloros+ parallel-tilstand

### Chloros+ GPU-acceleration

Hvis du bruger NVIDIA GPU-acceleration:

1. Opdater NVIDIA-drivere til den nyeste version
2. Sørg for, at GPU&#x27;en har 4 GB+ VRAM
3. Luk GPU-intensive programmer (spil, videoredigering)
4. Overvåg GPU-temperaturen (sørg for tilstrækkelig køling)

***

## Næste trin

Når behandlingen er startet:

1. **Overvåg fremskridtet** – Se [Overvågning af behandlingen](monitoring-the-processing.md)
2. **Vent på, at behandlingen er færdig** – Behandlingen kører automatisk
3. **Gennemgå resultaterne** – Se [Afslutning af behandlingen](finishing-the-processing.md)

For information om, hvad du skal gøre under behandlingen, se [Overvågning af behandlingen](monitoring-the-processing.md).

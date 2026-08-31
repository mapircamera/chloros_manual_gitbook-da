# Start af behandlingen

Når du har importeret dine billeder, markeret dine kalibreringsmål og konfigureret dine projektindstillinger, er du klar til at påbegynde behandlingen. Denne side vejleder dig gennem opstarten af Chloros-behandlingsprocessen.

## Tjekliste før behandling

Før du klikker på Start-knappen, skal du kontrollere, at alt er klar:

* [ ] **Filer importeret** – Alle billeder vises i filbrowseren
* [ ] **Målbilleder markeret** – Kolonnen „Mål“ er markeret for kalibreringsbilleder (eller en `.daq`-optagelse importeret til LATTICE)
* [ ] **Kameramodeller registreret** – Kolonnen „Kameramodel“ viser de korrekte kameraer
* [ ] **Indstillinger konfigureret** – Projektindstillingerne er gennemgået og justeret
* [ ] **Indekser valgt** – De ønskede multispektrale indekser er tilføjet (hvis nødvendigt)
* [ ] **Eksportformat valgt** – Outputformat, der passer til din arbejdsgang

{% hint style="info" %}
**Tip**: Klik dig igennem et par billeder i filbrowseren for at kontrollere, at de er indlæst korrekt, inden du fortsætter med behandlingen.
{% endhint %}

***

## Start af behandlingen

### Find Start-knappen

Start/Afspil-knappen er placeret i den øverste statuslinje i Chloros:

* Placering: Øverst i midten af vinduet
* Ikon: **Afspil/Start-knap** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Status: Knappen er aktiveret (lysende), når den er klar til behandling

### Klik for at starte

1. Klik på **Afspil/Start-knappen** i den øverste overskriftslinje
2. Behandlingen starter med det samme
3. Knappen bliver til en **Stop**-knap under behandlingen
4. Statusbjælken opdateres og viser behandlingsstatus

{% hint style="success" %}
**Behandlingen er startet**: Når du har klikket, håndterer Chloros automatisk alle trin i behandlingen – måldetektion, debayering, kalibrering, indeksberegning og eksport. Det registrerer automatisk, om dit projekt er Survey3, LATTICE eller en blanding, og anvender den rigtige pipeline til hvert kamera.
{% endhint %}

***

## Forståelse af behandlingsmodi

Chloros fungerer i to forskellige behandlingsmodi afhængigt af din licens:

### Gratis tilstand (sekventiel behandling)

**Tilgængelig for alle brugere**

**Sådan fungerer det:**

* Behandler billederne ét ad gangen, sekventielt
* Enkelt-trådet drift
* Lavere hukommelsesforbrug

**Statusbjælken viser 2 trin:**

1.**Målregistrering** – Scanner efter kalibreringsmål
2. **Behandling** – Anvender kalibrering og eksporterer billeder**Behandlingstid:**

* Meget langsommere end Chloros+ parallel-tilstand
* Velegnet til små til mellemstore datasæt (&lt; 200 billeder)

### Chloros+-tilstand (Parallel behandling)

**Kræver Chloros+-licens**

**Sådan fungerer det:**

* Behandler flere billeder samtidigt ved hjælp af en [4-trådet behandlingspipeline](../processing-architecture/processing-pipeline.md)
* [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) vælger automatisk den optimale strategi for din hardware ved kørselens start
* GPU (CUDA)-acceleration med NVIDIA-grafikkort (desktop og Jetson)
* **Antallet af arbejdsprocesser tilpasses hardwaren**: GPU-strategier kører**1–4 samtidige arbejdsprocesser** (skaleret efter VRAM — en Jetson med lav hukommelse kører 1, en desktop-GPU med 12 GB+ kører op til 4); systemer, der udelukkende bruger CPU, kører én arbejdsproces pr. fysisk kerne, minus én**Statusbjælken viser 4 faser** (svarende til de 4 pipeline-tråde):

1. **Detektering** (tråd 1) – Find kalibreringsmål
2. **Analyse** (tråd 2) – Undersøgelse af billedmetadata og beregning af kalibrering
3. **Kalibrering** (tråd 3) – Debayering, vignetteringskorrektion, kalibrering, indeksberegning
4. **Eksport** (Tråd 4) – Gemmer behandlede billeder og indekser**Interaktion med fremskridtsbjælken:*** **Hold musen** over bjælken for at se et detaljeret dropdown-panel med 4 faser
* **Klik** på fremskridtsbjælken for at fastfryse dropdown-panelet på plads
* **Klik igen** for at frigøre og skjule panelet**Behandlingstid:**

* Betydeligt hurtigere end gratis-tilstand
* GPU-acceleration forbedrer hastigheden yderligere

{% hint style="info" %}
**Chloros+ Hastighed**: Parallel behandling kan være 5-10 gange hurtigere end sekventiel tilstand for store datasæt. Et projekt med 500 billeder, der tager 2 timer i gratis tilstand, kan afsluttes på 15-20 minutter med Chloros+.
{% endhint %}

***

## Hvad sker der under behandlingen

### Trin 1: Målregistrering

**Hvad Chloros gør:**

* Gennemser de billeder, du har markeret i kolonnen »Mål« (alle billeder, hvis ingen er markeret)
* Identificerer kalibreringspanelerne i hvert mål
* Udtrækker reflektansværdier fra målpanelerne
* Registrerer målets tidsstempler til planlægning af kalibrering

**Varighed:** 1–30 sekunder (med markerede mål), 5–30+ minutter (umarkerede)

### Trin 2: Debayering (RAW-konvertering)

**Hvad Chloros gør:**

* Konverterer RAW-data i Bayer-mønster til fulde 3-kanals billeder (LATTICE-mono-moduler forbliver enkeltbånds — debayering springes over for dem med en bemærkning i loggen)
* Anvender den valgte demosaicing-algoritme
* Bevarer maksimal billedkvalitet og detaljer

**Varighed:** Varierer afhængigt af antal billeder og CPU/GPU-hastighed

### Trin 3: Kalibrering

**Hvad Chloros gør:*** **Vignetteringskorrektion**: Fjerner mørkningen ved kanterne
* **Reflektanskalibrering**: Normaliserer ved hjælp af målreflektansværdier og/eller DAQ-downwelling-data
* Anvender korrektioner på tværs af alle bånd/kanaler
* Bruger den passende kalibreringsreference for hvert billede baseret på tidsstemplet

**Varighed:** Størstedelen af behandlingstiden

### Trin 4: Indeksberegning

**Hvad Chloros gør:**

* Beregner konfigurerede multispektrale indekser (NDVI, NDRE osv.)
* Anvender båndmatematik på kalibrerede billeder
* Genererer indeksbilleder for hvert valgt indeks

**Varighed:** Et par sekunder pr. billede

### Trin 5: Eksport

**Hvad Chloros gør:**

* Gemmer de behandlede billeder i det valgte format
* **LATTICE fan-out**: Hvert rå LATTICE-billede eksporteres som alle aktiverede produkter i én omgang — debayered, preview, radiance (altid float32), reflectance
* Skriver filer til projektets output-struktur: `<project>/<camera>/<format>/<Product>_Images/`
* **Beholder kildefilnavnet** — mappen identificerer produktet, der tilføjes intet suffiks**Varighed:** Varierer afhængigt af eksportformat og filstørrelse***

## Behandlingsadfærd

### Automatisk behandlingspipeline

Når den er startet, kører hele processen automatisk:

* Ingen brugerindgriben nødvendig
* Alle konfigurerede trin udføres i rækkefølge
* Statusopdateringer vises i realtid
* Eksporterede filer gemmes på disken, efterhånden som de bliver færdige — du kan åbne færdige resultater, mens processen fortsætter

### Computerforbrug under behandlingen

**Fri tilstand:**

* Relativt lav CPU-udnyttelse (enkelt-trådet)
* Computeren forbliver responsiv over for andre opgaver
* Det er sikkert at minimere Chloros og arbejde i andre programmer

**Chloros+ Parallel tilstand:**

* Højt CPU-forbrug på tværs af strategiens arbejdspool
* Med GPU-acceleration: Højt GPU-forbrug
* Computeren kan være mindre responsiv under behandlingen
* Undgå at starte andre CPU-krævende opgaver

{% hint style="warning" %}
**Ydelsestip**: For at opnå den bedste Chloros+-ydelse skal du lukke andre programmer og lade Chloros bruge alle systemressourcerne.
{% endhint %}

### Behandlingen kan ikke sættes på pause (men kan afbrydes fuldstændigt)

* Når behandlingen er startet, kan den ikke sættes på pause og genoptages senere
* Ved at klikke på **Stop** afbrydes kørslen fuldstændigt ved første klik
* Produkter, der allerede er eksporteret før afbrydelsen, forbliver på disken
* En afbrudt kørsel rapporterer præcist, hvad der er gennemført (se linjerne `[RUN-SUMMARY]` i loggen)
* Et nyt kørsel starter rørledningen fra begyndelsen

**Planlægningstip:** Ved meget store projekter bør du overveje at behandle i batches eller bruge CLI for bedre kontrol.***

## Overvågning af din behandling

Mens behandlingen kører, kan du:

* **Se statusbjælken** – Se den samlede færdiggørelsesprocent
* **Se den aktuelle fase** – Registrering, analyse, kalibrering eller eksport
* **Tjek fanen &quot;Log&quot;** – Se detaljerede behandlingsmeddelelser og advarsler
* **Få et eksempel på færdige billeder** – Eksportfiler vises på disken under behandlingen

Se [Overvågning af behandlingen](monitoring-the-processing.md) for detaljerede oplysninger om overvågning.

***

## Stop af behandlingen

Hvis du har brug for at stoppe behandlingen:

### Sådan stopper du

1. Find **Stop-knappen** (erstatter Start-knappen under behandlingen)
2. Klik på den én gang — statusbjælken viser **&quot;Stopper...&quot;**, mens det igangværende billede færdigbehandles
3. Kørslen afsluttes i en endelig stoppet tilstand, og loggen udskriver en præcis oversigt over, hvad der blev gennemført

### Hvornår skal man stoppe

**Gyldige grunde til at stoppe:**

* Man har opdaget, at der blev brugt forkerte indstillinger
* Man har glemt at markere målbilleder
* Der er importeret forkerte billeder
* Systemet kører for langsomt eller reagerer ikke

**Efter stop:**

* Produkter, der blev eksporteret før stop, forbliver på disken
* Gennemgå og afhjælp eventuelle problemer, juster indstillingerne efter behov
* Genstart behandlingen — kørslen starter forfra

***

## Anslået behandlingstid

Den faktiske behandlingstid varierer meget afhængigt af:

* Antal billeder
* Billedopløsning
* Indgangsformat (RAW eller JPG)
* Behandlingsmodus (Free eller Chloros+)
* CPU-hastighed og antal kerner
* GPU-tilgængelighed (kun Chloros+)
* Antal indekser, der skal beregnes
* Antal aktiverede eksportprodukter (LATTICE)

### Grove skøn (Chloros+, 12 MP-billeder, moderne CPU)

| Antal billeder | Gratis tilstand | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 billeder   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 billeder  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 billeder  | 1-1,5 timer | 20-30 min      | 10-15 min      |
| 500 billeder  | 2–3 timer   | 45–60 min      | 20–30 min      |
| 1000 billeder | 4–6 timer   | 1,5–2 timer      | 40–60 min      |

{% hint style="info" %}
**Første kørsel**: Den indledende behandling kan tage længere tid, da Chloros opbygger caches og profiler. Efterfølgende behandling af lignende datasæt vil foregå hurtigere.
{% endhint %}

***

## Almindelige problemer ved opstart

### Startknappen er deaktiveret (gråtonet)

**Mulige årsager:**

* Der er ikke importeret billeder
* Backend er ikke fuldt opstartet
* Forrige behandling kører stadig
* Projektet er ikke fuldt indlæst

**Løsninger:**

1. Vent, indtil backend er fuldt initialiseret (tjek ikonet i hovedmenuen)
2. Kontroller, at billederne er importeret i filbrowseren
3. Genstart Chloros, hvis knappen fortsat er deaktiveret
4. Tjek fejlloggen for fejlmeddelelser

### Behandlingen starter, men mislykkes straks

**Mulige årsager:**

* Ingen gyldige billeder i projektet
* Beskadigede billedfiler
* Utilstrækkelig diskplads
* Utilstrækkelig hukommelse (RAM)

**Løsninger:**

1. Kontroller fejlloggen <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> for fejlmeddelelser
2. Kontroller, om der er ledig diskplads
3. Prøv at behandle en mindre delmængde af billederne
4. Kontroller, at billederne ikke er beskadigede

### Kørslen afsluttes, men der skrives ingen billeder

En kørsel, der anmodede om billedprodukter, men ikke skrev nogen, behandles som en **fejl, ikke en succes** — Chloros rapporterer det tydeligt:

* GUI-loggen viser `[RUN-SUMMARY]`-henvisninger, der angiver den sandsynlige årsag — ingen billeder importeret, intet mål fundet, eller at alle anmodede produkter er blevet sprunget over som uanvendelige (f.eks. anmodning om radiance/reflektans fra kameraer, der kun understøtter RGB)
* CLI-ækvivalenten (`chloros-cli process`) udskriver `Processing finished but wrote no image products.` og **afsluttes med en værdi forskellig fra nul**, så scripts kan registrere det
* En bevidst kørsel, der kun omfatter metadata (alle eksportprodukter deaktiveret, ingen indekser), tæller stadig som en succes

Se [CLI-referencen](../reference/cli-reference.md#a-run-that-writes-no-images-fails) for den fulde semantik.

### Advarsel om »Ingen mål fundet«

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

1. **Test først med et lille udsnit** – Behandl 10–20 billeder for at kontrollere indstillingerne
2. **Kontroller ledig diskplads** – Sørg for, at der er 2–3 gange datasættets størrelse ledig (mere, hvis alle LATTICE-produkter er aktiveret)
3. **Luk unødvendige programmer** – Frigør systemressourcer
4. **Kontroller målbillederne** – Se en forhåndsvisning af de markerede mål for at sikre kvaliteten
5. **Gem projektet** – Projektet gemmes automatisk, men det er god praksis at gemme manuelt

### Under behandlingen

1. **Undgå, at systemet går i dvaletilstand** – Deaktiver strømbesparende tilstande
2. **Hold Chloros i forgrunden** – Eller i det mindste synlig i proceslinjen
3. **Overvåg fremskridtet lejlighedsvis** – Tjek for advarsler eller fejl
4. **Undlad at køre andre ressourcekrævende programmer** – Især i Chloros+ parallel-tilstand

### Chloros+ GPU-acceleration

Hvis du bruger NVIDIA GPU-acceleration:

1. Opdater NVIDIA-driverne til den nyeste version
2. Sørg for, at GPU’en har mindst 4 GB VRAM (mindst 7 GB til samtidig Texture Aware-debayering)
3. Luk GPU-krævende programmer (spil, videoredigering)
4. Overvåg GPU-temperaturen (sørg for tilstrækkelig køling)

***

## Næste trin

Når behandlingen er startet:

1. **Overvåg fremskridtet** – Se [Overvågning af behandlingen](monitoring-the-processing.md)
2. **Vent på, at behandlingen er færdig** – Behandlingen kører automatisk
3. **Gennemgå resultaterne** – Se [Afslutning af behandlingen](finishing-the-processing.md)

For information om, hvad du skal gøre under behandlingen, se [Overvågning af behandlingen](monitoring-the-processing.md).

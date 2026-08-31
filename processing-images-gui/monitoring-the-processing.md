# Overvågning af behandlingen

Når behandlingen er gået i gang, tilbyder Chloros flere måder at overvåge fremskridtet på, kontrollere for problemer og få indsigt i, hvad der sker med dit datasæt. Denne side forklarer, hvordan du kan følge med i behandlingen og fortolke de oplysninger, som Chloros leverer.

## Oversigt over statusbjælken

Statusbjælken øverst i overskriften viser behandlingsstatus i realtid samt procentdelen af færdiggørelse. Status opdateres live fra backend via Server-Sent Events (SSE), så bjælken afspejler, hvad behandlingsrørledningen rent faktisk foretager sig.

### Statusbjælke i gratis tilstand

For brugere uden Chloros+-licens:

**2-trins fremskridtsvisning:**

1.**Målregistrering** – Find kalibreringsmål i billeder
2. **Behandling** – Anvendelse af korrektioner og eksport**Statusbjælken viser:**

* Samlet færdiggørelsesprocent (0–100 %)
* Navn på den aktuelle fase
* Enkel vandret bjælkevisualisering

### Chloros+ statusbjælke

For brugere med Chloros+-licens:

**4-trins fremskridtsvisning:**

1.**Detektering** – Søgning efter kalibreringsmål
2. **Analyse** – Gennemgang af billeder og forberedelse af behandlingsforløb
3. **Kalibrering** – Anvendelse af vignetterings- og reflektanskorrektioner
4. **Eksport** – Gemning af behandlede filer**Interaktive funktioner:*** **Hold musen over** statusbjælken for at se det udvidede panel med de 4 trin
* **Klik på** statusbjælken for at fastfryse/fastgøre det udvidede panel
* **Klik igen** for at ophæve fastfrysningen, så panelet skjules automatisk, når musen fjernes
* Hvert trin viser den individuelle fremdrift (0-100 %)

{% hint style="info" %}
**CLI-paritet**: Under en `chloros-cli process`-kørsel rapporterer de samme fire tråde som »Detecting«, »Analyzing«, »Processing«, &quot;Exporting&quot;, og `chloros-cli export-status` viser den aktuelle fremskridt for eksport af tråd 4 fra en anden terminal. Se [CLI-referencen](../reference/cli-reference.md).
{% endhint %}

***

## Forståelse af de enkelte behandlingsfaser

{% hint style="info" %}
**Pipeline-arkitektur**: Disse 4 GUI-faser svarer til [4-tråds-behandlingspipeline](../processing-architecture/processing-pipeline.md). På systemer med GPU-acceleration drager tråd 3 (Kalibrering) fordel af [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md), som optimerer behandlingen til din specifikke hardware.
{% endhint %}

### Trin 1: Registrering (målregistrering)

**Hvad sker der:**

* Chloros scanner de billeder, du har markeret med afkrydsningsfeltet »Mål« (alle billeder, hvis ingen er markeret)
* Computervisionsalgoritmer identificerer kalibreringspanelerne
* Refleksionsværdier udtrækkes fra hvert panel
* Tidsstempler for mål registreres med henblik på korrekt planlægning af kalibreringen

**Varighed:**

* Med markerede mål: 10–60 sekunder
* Uden markerede mål: 5–30+ minutter (scanner alle billeder)

**Statusindikator:**

* Registrering: 0 % → 100 %
* Antal scannede billeder (tæller kun de billeder, der faktisk scannes)
* Antal fundne mål

**Hvad du skal være opmærksom på:**

* Bør afsluttes hurtigt, hvis målene er markeret korrekt
* Hvis det tager for lang tid, er målene muligvis ikke markeret
* Tjek fejlfindingsloggen for meddelelser om &quot;Mål fundet&quot;

### Trin 2: Analyse

**Hvad sker der:**

* Indlæsning af billedets EXIF-metadata (tidsstempler, eksponeringsindstillinger)
* Fastlæggelse af kalibreringsstrategi baseret på målets tidsstempler og tilgængelige DAQ-downwelling-data
* Organisering af billedbehandlingskøen
* Forberedelse af parallelle behandlingsprocesser (kun Chloros+)

**Varighed:** 5–30 sekunder**Statusindikator:**

* Analyserer: 0 % → 100 %
* Hurtig fase, afsluttes normalt hurtigt

**Hvad man skal være opmærksom på:**

* Skal forløbe jævnt uden pauser
* Advarsler om manglende metadata vises i fejlfindingsloggen

### Fase 3: Kalibrering

**Hvad der sker:*** **Debayering**: Konvertering af RAW-Bayer-mønster til 3 kanaler (overspringes for LATTICE-monomoduler, med en bemærkning)
* **Vignetteringskorrektion**: Fjernelse af mørkningen ved linsens kanter
* **Reflektanskalibrering**: Normalisering med målværdier og/eller DAQ-nedstråling
* **Indeksberegning**: Beregning af multispektrale indekser
* Behandling af hvert billede gennem hele processen

**Varighed:** Størstedelen af den samlede behandlingstid (60–80 %)**Statusindikator:**

* Kalibrering: 0 % → 100 %
* Det aktuelle billede behandles
* Færdigbehandlede billeder / Samlet antal billeder

**Behandlingsadfærd:*** **Fri tilstand**: Behandler ét billede ad gangen sekventielt
* **Chloros+-tilstand**: Kører en hardware-adaptiv arbejdspool — 1–4 samtidige arbejdsprocesser på GPU-systemer (afhængigt af VRAM), én arbejdsproces pr. fysisk kerne (minus én) på systemer, der kun har CPU. Se [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU-acceleration**: Fremskynder denne fase betydeligt**Hvad man skal holde øje med:**

* Jævn fremgang gennem billedtællingen
* Tjek fejlfindingsloggen for færdiggørelsesmeddelelser pr. billede
* Advarsler om billedkvalitet eller kalibreringsproblemer

### Trin 4: Eksport

**Hvad sker der:**

* De behandlede billeder skrives til disken i det valgte format, efterhånden som de færdiggøres
* **LATTICE**: Hvert billede fordeles til alle aktiverede produkter (debayered / preview / radiance / reflectance)
* Eksport af multispektrale indeksbilleder med LUT-farver
* Oprettelse af outputtræet `<project>/<camera>/<format>/<Product>_Images/` — eksporterede filer beholder kildefilnavnet; mappen identificerer produktet

**Varighed:** 10–20 % af den samlede behandlingstid**Statusindikator:**

* Eksport: 0 % → 100 %
* Filer bliver skrevet
* Eksportformat og destination

**Hvad man skal være opmærksom på:**

* Advarsler om diskplads
* Fejl ved filskrivning
* Afslutning af alle konfigurerede output

***

## Fanen Debug Log

Debug Log giver detaljerede oplysninger om behandlingsforløbet og eventuelle problemer, der opstår. Startmeddelelser fra backend vises også i logkonsollen, så loggen giver et fuldstændigt overblik, selvom du åbner den senere.

### Sådan får du adgang til fejlfindingsloggen

1. Klik på **Fejlfindingslog**-<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

ikonet i venstre sidepanel
2. Logpanelet åbnes og viser behandlingsmeddelelser i realtid
3. Der rulles automatisk for at vise de seneste meddelelser

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written) -->

### Forståelse af logmeddelelser

Chloros-loglinjer er forsynet med et præfiks i form af tags i parentes, der angiver navnet på undersystemet — for eksempel `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Det vigtigste at kende til er **kørselsoversigten**, der udskrives i slutningen af hvert kørsel (herunder afbrudte kørsler):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Der følger ekstra `[RUN-SUMMARY]`-hjælpelinjer, når der er behov for en forklaring — for eksempel et kørsel, der ikke gav noget resultat, eller et kamera, hvis ønskede produkt blev udeladt, da det ikke var relevant. `[EXPORT-CHECK]`-linjer forklarer udeladelser pr. kamera (f.eks. hvorfor et RGB-kamera ikke fik noget strålingsprodukt).

De generelle alvorlighedsniveauer for meddelelser (eksemplerne nedenfor er illustrative, ikke ordrette):

#### Informationsmeddelelser (hvid/grå)

Normale opdateringer om behandlingen: behandlingen er startet, mål er registreret (med antal paneler), fremskridt i kalibrering pr. billede, filer eksporteret, behandlingen er afsluttet.

#### Advarselsmeddelelser (Gul)

Ikke-kritiske problemer, der ikke stopper behandlingen — f.eks. manglende GPS-data i et billede, et stort tidsstempel-gab mellem målbilleder eller lav kontrast i et kalibreringspanel.

**Handling:** Gennemgå advarsler efter behandlingen, men afbryd ikke

#### Fejlmeddelelser (Red)

Kritiske problemer, der kan medføre, at behandlingen mislykkes — f.eks. fuld disk, en beskadiget billedfil eller ingen mål registreret, mens der blev anmodet om reflektanskalibrering.

**Handling:** Stop behandlingen, afhjælp fejlen, genstart

### Almindelige log-situationer

| Situation                             | Betydning                                       | Nødvendig handling                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Mål registreret i \[filnavn]        | Kalibreringsmål fundet uden problemer         | Ingen – normalt                                         |
| Statuslinjer pr. billede              | Aktuel statusopdatering                       | Ingen – normalt                                         |
| Ingen mål fundet                      | Ingen kalibreringsmål fundet               | Marker målbilleder eller deaktiver reflektanskalibrering |
| Utilstrækkelig diskplads               | Ikke nok lagerplads til output                 | Frigør diskplads                                    |
| Springes over på grund af beskadiget fil               | Billedfilen er beskadiget                         | Kopier filen igen fra SD-kortet                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | En optagelse uden et råbillede kan ikke behandles | Optag igen med råbillede, eller brug CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Kørslen producerede ingen billedprodukter — rapporteret som en fejl med tip | Læs tiplinjerne; kontroller, hvad der blev sprunget over, og hvorfor |

### Kopiering af logdata

Sådan kopieres loggen til fejlfinding eller support:

1. Åbn panelet Debug Log
2. Klik på knappen **&quot;Copy Log&quot;** (eller højreklik → Vælg alt)
3. Indsæt i en tekstfil eller e-mail
4. Send til MAPIR-support, hvis nødvendigt

***

## Overvågning af systemressourcer

### CPU-udnyttelse

**Fri tilstand:**

* 1 CPU-kerne på ca. 100 %
* Andre kerner er inaktive eller tilgængelige
* Systemet forbliver responsivt

**Chloros+ parallel tilstand:**

* Flere kerner med høj udnyttelse — hvor mange afhænger af den strategi, der er valgt af [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* Systemet kan føles mindre responsivt

**Sådan overvåges det:**

* Windows Opgavehåndtering (Ctrl+Shift+Esc)
* Fanen Ydeevne → afsnittet CPU
* Se efter processerne &quot;Chloros&quot; eller &quot;chloros-backend&quot;

### Hukommelsesforbrug (RAM)

**Typisk forbrug:**

* Små projekter (&lt; 100 billeder): 2-4 GB
* Mellemstore projekter (100–500 billeder): 4–8 GB
* Store projekter (500+ billeder): 8–16 GB
* Chloros+ i parallel tilstand bruger mere RAM

**Hvis der er lidt hukommelse:**

* Behandl mindre batcher
* Luk andre programmer
* Opgrader RAM, hvis du regelmæssigt behandler store datasæt

### GPU-forbrug (Chloros+ med CUDA)

Når GPU-acceleration er aktiveret:

* NVIDIA-GPU&#x27;en viser høj udnyttelse (60–90 %)
* VRAM-forbruget stiger (kræver 4 GB+ VRAM; 7 GB+ til samtidig Texture Aware-debayering)
* Kalibreringsfasen er betydeligt hurtigere

**Sådan overvåges det:**

* NVIDIA-ikonet i systembakken
* Opgavehåndtering → Ydeevne → GPU
* GPU-Z eller lignende overvågningsværktøj

### Disk-I/O

**Hvad du kan forvente:**

* Høj disk-læsehastighed under analysefasen
* Høj disk-skrivehastighed under eksportfasen
* SSD er betydeligt hurtigere end HDD

**Ydelsestip:**

* Brug SSD til projektmappen, når det er muligt
* Undgå netværksdrev til store datasæt
* Sørg for, at disken ikke er næsten fyldt (påvirker skrivehastigheden)

***

## Registrering af problemer under behandlingen

### Advarselstegn

**Fremskridtet går i stå (ingen ændring i mere end 5 minutter):**

* Kontroller fejlloggen for fejl
* Kontroller, om der er ledig diskplads
* Tjek Opgavelisten for at sikre, at Chloros kører

**Der vises hyppigt fejlmeddelelser:**

* Stop behandlingen, og gennemgå fejlene
* Almindelige årsager: diskplads, beskadigede filer, hukommelsesproblemer
* Se afsnittet Fejlfinding nedenfor

**Systemet reagerer ikke:**

* Chloros+ i parallel tilstand bruger for mange ressourcer
* Overvej at reducere antallet af samtidige opgaver eller opgradere hardwaren
* Fri tilstand er mindre ressourcekrævende

### Hvornår skal behandlingen afbrydes

Afbryd behandlingen, hvis du ser:

* ❌ Fejlmeddelelser som »Disk fuld« eller »Kan ikke skrive fil«
* ❌ Gentagne fejlmeddelelser om beskadigede billedfiler
* ❌ Systemet er helt frosset fast (reagerer ikke)
* ❌ Du har opdaget, at der er konfigureret forkerte indstillinger
* ❌ Der er importeret forkerte billeder

**Sådan stopper du:**

1. Klik på**Stop-knappen** (erstatter Start-knappen) — én gang er nok
2. Statusbjælken viser &quot;Stopper...&quot;, mens den igangværende billedfil afsluttes, hvorefter kørslen afsluttes i en stoppet tilstand
3. Produkter, der allerede er eksporteret, forbliver på disken; loggen viser en præcis oversigt over, hvad der er afsluttet
4. Løs problemerne og genstart — kørslen starter forfra

***

## Fejlfinding under behandling

### Behandlingen er meget langsom

**Mulige årsager:**

* Umarkerede målbilleder (alle billeder scannes)
* HDD i stedet for SSD-lager
* Utilstrækkelige systemressourcer
* Mange indekser konfigureret
* Adgang via netværksdrev

**Løsninger:**

1. Hvis kørslen netop er startet og befinder sig i detekteringsfasen: Stop, markér målene, genstart
2. Fremover: Brug SSD, reducer antallet af indekser, opgrader hardware
3. Overvej at bruge CLI til batchbehandling af store datasæt

### Advarsler om &quot;diskplads&quot;

**Løsninger:**

1. Frigør diskplads med det samme
2. Flyt projektet til et drev med mere plads
3. Reducer antallet af indekser, der skal eksporteres
4. Deaktiver LATTICE-eksportprodukter, du ikke har brug for (Projektindstillinger → Behandling)
5. Brug JPG-format i stedet for TIFF (mindre filer)

### Hyppige meddelelser om &quot;beskadiget fil&quot;

**Løsninger:**

1. Kopier billederne igen fra SD-kortet for at sikre, at de er intakte
2. Test SD-kortet for fejl
3. Fjern beskadigede filer fra projektet
4. Fortsæt behandlingen af de resterende billeder

### Systemoverophedning / begrænsning af ydeevnen

**Løsninger:**

1. Sørg for tilstrækkelig ventilation
2. Fjern støv fra computerens ventilationsåbninger
3. Reducer behandlingsbelastningen (brug Free-tilstand i stedet for Chloros+)
4. Udfør behandlingen på køligere tidspunkter af døgnet

***

## Meddelelse om afsluttet behandling

Når behandlingen er afsluttet:

* Statusbjælken når 100 %
* Linjerne `[RUN-SUMMARY]` vises i fejlfindingsloggen med de endelige tal
* Startknappen bliver aktiveret igen
* Alle outputfiler findes i projektets outputstruktur pr. kamera: `<project>/<camera>/<format>/<Product>_Images/`

***

## Næste trin

Når behandlingen er afsluttet:

1. **Gennemgå resultaterne** – Se [Afslutning af behandlingen](finishing-the-processing.md)
2. **Kontroller outputmappen** – Kontroller, at alle filer er eksporteret korrekt
3. **Gennemgå fejlfindingsloggen** – Se efter eventuelle advarsler eller fejl
4. **Vis forhåndsvisning af de behandlede billeder** – Brug billedviseren eller ekstern software

For oplysninger om gennemgang og brug af dine behandlede resultater, se [Afslutning af behandlingen](finishing-the-processing.md).

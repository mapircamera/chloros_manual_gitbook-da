# Overvågning af behandlingen

Når behandlingen er startet, giver Chloros flere muligheder for at overvåge fremskridt, kontrollere for problemer og forstå, hvad der sker med dit datasæt. Denne side forklarer, hvordan du sporer din behandling og fortolker de oplysninger, Chloros giver.

## Oversigt over fremskridtsbjælke

Statusbjælken i den øverste header viser behandlingsstatus og færdiggørelsesprocent i realtid.

### Statusbjælke i gratis tilstand

For brugere uden Chloros+-licens:

**2-trins statusvisning:**

1. **Måldetektion** – Find kalibreringsmål i billeder
2. **Behandling** – Anvend korrektioner og eksporter

**Statusbjælken viser:**

* Samlet færdiggørelsesprocent (0-100 %)
* Navn på nuværende trin
* Enkel vandret bjælkevisualisering

### Chloros+ statusbjælke

For brugere med Chloros+ licens:

**4-trins statusvisning:**

1. **Detektering** - Find kalibreringsmål
2. **Analyse** - Undersøg billeder og forbered pipeline
3. **Kalibrering** - Anvend vignette- og reflektanskorrektioner
4. **Eksport** - Gem behandlede filer

**Interaktive funktioner:**

* **Hold musen over** fremdriftsbjælken for at se det udvidede 4-trins panel
* **Klik** på fremdriftsbjælken for at fryse/fastgøre det udvidede panel
* **Klik igen** for at frigøre og automatisk skjule, når musen fjernes
* Hvert trin viser individuel fremdrift (0-100 %)

***

## Forståelse af hvert behandlingsstadium

### Trin 1: Registrering (målregistrering)

**Hvad sker der:**

* Chloros scanner billeder markeret med afkrydsningsfeltet Mål
* Computervisionsalgoritmer identificerer de 4 kalibreringspaneler
* Reflektansværdier udvindes fra hvert panel
* Mål-tidsstempler registreres for korrekt kalibreringsplanlægning

**Varighed:**

* Med markerede mål: 10-60 sekunder
* Uden markerede mål: 5-30+ minutter (scanner alle billeder)

**Fremskridtsindikator:**

* Registrerer: 0 % → 100 %
* Antal scannede billeder
* Antal fundne mål

**Hvad skal man være opmærksom på:**

* Bør være hurtigt færdigt, hvis målene er korrekt markeret
* Hvis det tager for lang tid, er målene muligvis ikke markeret
* Tjek fejlfindingsloggen for meddelelser om &quot;Mål fundet&quot;

### Trin 2: Analyse

**Hvad sker der:**

* Læser billedets EXIF-metadata (tidsstempler, eksponeringsindstillinger)
* Bestemmer kalibreringsstrategi baseret på måltidsstempler
* Organiserer billedbehandlingskøen
* Forbereder parallelle behandlingsarbejdere (kun Chloros+)

**Varighed:** 5-30 sekunder

**Fremskridtsindikator:**

* Analyserer: 0 % → 100 %
* Hurtig fase, afsluttes normalt hurtigt

**Hvad man skal være opmærksom på:**

* Skal forløbe jævnt uden pauser
* Advarsler om manglende metadata vises i fejlfindingsloggen

### Trin 3: Kalibrering

**Hvad sker der:**

* **Debayering**: Konvertering af RAW Bayer-mønster til 3 kanaler
* **Vignettekorrektion**: Fjerner mørkfarvning i kanten af linsen
* **Reflektanskalibrering**: Normaliserer med målværdier
* **Indeksberegning**: Beregner multispektrale indekser
* Behandler hvert billede gennem hele pipelinen

**Varighed:** Størstedelen af den samlede behandlingstid (60-80 %)

**Fremskridtsindikator:**

* Kalibrering: 0 % → 100 %
* Nuværende billede behandles
* Færdige billeder / Samlede billeder

**Behandlingsadfærd:**

* **Fri tilstand**: Behandler ét billede ad gangen sekventielt
* **Chloros+ tilstand**: Behandler op til 16 billeder samtidigt
* **GPU-acceleration**: Fremskynder denne fase betydeligt

**Hvad du skal være opmærksom på:**

* Stabil fremgang gennem billedantal
* Kontroller fejlfindingsloggen for meddelelser om færdiggørelse pr. billede
* Advarsler om billedkvalitet eller kalibreringsproblemer

### Fase 4: Eksport

**Hvad sker der:**

* Skriver kalibrerede billeder til disk i valgt format
* Eksporterer multispektrale indeksbilleder med LUT-farver
* Opretter undermapper til kameramodeller
* Bevarer originale filnavne med passende suffikser

**Varighed:** 10-20 % af den samlede behandlingstid

**Fremskridtsindikator:**

* Eksport: 0 % → 100 %
* Filer, der skrives
* Eksportformat og destination

**Hvad du skal være opmærksom på:**

* Advarsler om diskplads
* Fejl ved filskrivning
* Færdiggørelse af alle konfigurerede output

***

## Fanen Debug Log (Fejlfindingslog)

Fejlfindingsloggen giver detaljerede oplysninger om behandlingsforløbet og eventuelle problemer.

### Adgang til fejlfindingsloggen

1. Klik på ikonet **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> i venstre sidepanel.
2. Logpanelet åbnes og viser behandlingsmeddelelser i realtid.
3. Ruller automatisk for at vise de seneste meddelelser.

### Forståelse af logmeddelelser

#### Informationsmeddelelser (hvid/grå)

Normale behandlingsopdateringer:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Advarselsmeddelelser (gul)

Ikke-kritiske problemer, der ikke stopper behandlingen:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Handling:** Gennemgå advarsler efter behandlingen, men afbryd ikke

#### Fejlmeddelelser (Red)

Kritiske problemer, der kan medføre, at behandlingen mislykkes:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Handling:** Stop behandlingen, løs fejlen, genstart.

### Almindelige logmeddelelser

| Meddelelse                          | Betydning                                | Nødvendig handling                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| &quot;Mål fundet i \[filnavn]&quot; | Kalibreringsmål fundet  | Ingen - normalt                                         |
| &quot;Behandler billede X af Y&quot;        | Aktuel statusopdatering                | Ingen - normalt                                         |
| &quot;Ingen mål fundet&quot;               | Ingen kalibreringsmål fundet        | Marker målbilleder eller deaktiver reflektanskalibrering |
| &quot;Utilstrækkelig diskplads&quot;        | Ikke nok lagerplads til output          | Frigør diskplads                                    |
| &quot;Springer beskadiget fil over&quot;        | Billedfilen er beskadiget                  | Kopier filen igen fra SD-kortet                             |
| &quot;PPK-data anvendt&quot;               | GPS-korrektioner fra .daq-fil anvendt | Ingen - normal                                         |

### Kopiering af logdata

Sådan kopieres log til fejlfinding eller support:

1. Åbn panelet Debug Log (Fejlfindingslog).
2. Klik på knappen **&quot;Copy Log&quot;** (Kopier log) (eller højreklik → Vælg alt).
3. Indsæt i tekstfil eller e-mail.
4. Send til MAPIR support, hvis nødvendigt.

***

## Overvågning af systemressourcer

### CPU-brug

**Fri tilstand:**

* 1 CPU-kerne på \~100 %
* Andre kerner er inaktive eller tilgængelige
* Systemet forbliver responsivt

**Chloros+ Parallel tilstand:**

* Flere kerner på 80-100 % (op til 16 kerner)
* Høj samlet CPU-udnyttelse
* Systemet kan føles mindre responsivt

**Overvågning:**

* Windows Task Manager (Ctrl+Shift+Esc)
* Fanen Ydeevne → afsnittet CPU
* Se efter processerne &quot;Chloros&quot; eller &quot;chloros-backend&quot;

### Hukommelsesforbrug (RAM)

**Typisk brug:**

* Små projekter (&lt; 100 billeder): 2-4 GB
* Mellemstore projekter (100-500 billeder): 4-8 GB
* Store projekter (500+ billeder): 8-16 GB
* Chloros+ parallel mode bruger mere RAM

**Hvis hukommelsen er lav:**

* Behandl mindre batcher
* Luk andre programmer
* Opgrader RAM, hvis du regelmæssigt behandler store datasæt

### GPU-forbrug (Chloros+ med CUDA)

Når GPU-acceleration er aktiveret:

* NVIDIA GPU viser høj udnyttelse (60-90 %)
* VRAM-forbruget stiger (kræver 4 GB+ VRAM)
* Kalibreringsfasen er betydeligt hurtigere

**Overvågning:**

* NVIDIA-ikonet i systembakken
* Task Manager → Performance → GPU
* GPU-Z eller lignende overvågningsværktøj

### Disk I/O

**Hvad kan man forvente:**

* Høj disk-læsehastighed under analysefasen
* Høj disk-skrivehastighed under eksportfasen
* SSD er betydeligt hurtigere end HDD

**Tip til ydeevne:**

* Brug SSD til projektmappen, når det er muligt
* Undgå netværksdrev til store datasæt
* Sørg for, at disken ikke er tæt på kapacitetsgrænsen (påvirker skrivehastigheden)

***

## Registrering af problemer under behandlingen

### Advarselstegn

**Fremskridt går i stå (ingen ændringer i mere end 5 minutter):**

* Kontroller fejlfindingsloggen for fejl
* Kontroller, at der er ledig diskplads
* Kontroller Task Manager for at sikre, at Chloros kører

**Fejlmeddelelser vises ofte:**

* Stop behandlingen og gennemgå fejlene
* Almindelige årsager: diskplads, beskadigede filer, hukommelsesproblemer
* Se afsnittet Fejlfinding nedenfor

**Systemet reagerer ikke:**

* Chloros+ parallel mode bruger for mange ressourcer
* Overvej at reducere samtidige opgaver eller opgradere hardware
* Free mode er mindre ressourcekrævende

### Hvornår skal behandlingen stoppes

Stop behandlingen, hvis du ser:

* ❌ &quot;Disk fuld&quot; eller &quot;Kan ikke skrive fil&quot; fejl
* ❌ Gentagne fejl med beskadigede billedfiler
* ❌ Systemet er helt frosset (reagerer ikke)
* ❌ Indså, at der var konfigureret forkerte indstillinger
* ❌ Forkerte billeder importeret

**Sådan stopper du:**

1. Klik på **Stop/Annuller-knappen** (erstatter Start-knappen)
2. Behandlingen standses, og fremskridtet går tabt
3. Løs problemerne, og start forfra

***

## Fejlfinding under behandlingen

### Behandlingen er meget langsom

**Mulige årsager:**

* Umærkede målbilleder (scanning af alle billeder)
* HDD i stedet for SSD-lagerplads
* Utilstrækkelige systemressourcer
* Mange indekser konfigureret
* Adgang til netværksdrev

**Løsninger:**

1. Hvis du lige er startet og er i detekteringsfasen: Annuller, marker mål, start forfra
2. Fremover: Brug SSD, reducer indekser, opgrader hardware
3. Overvej CLI til batchbehandling af store datasæt

### Advarsler om &quot;diskplads&quot;

**Løsninger:**

1. Frigør diskplads med det samme
2. Flyt projektet til et drev med mere plads
3. Reducer antallet af indekser, der skal eksporteres
4. Brug JPG-format i stedet for TIFF (mindre filer)

### Hyppige &quot;korrupte fil&quot;-meddelelser

**Løsninger:**

1. Kopier billederne igen fra SD-kortet for at sikre integriteten
2. Test SD-kortet for fejl
3. Fjern korrupte filer fra projektet
4. Fortsæt behandlingen af de resterende billeder

### Systemet bliver overophedet/throttling

**Løsninger:**

1. Sørg for tilstrækkelig ventilation
2. Rengør computeren for støv
3. Reducer behandlingsbelastningen (brug Free-tilstand i stedet for Chloros+)
4. Behandl i de køligere timer af døgnet

***

## Meddelelse om færdig behandling

Når behandlingen er færdig:

* Statusbjælken når 100 %
* Meddelelsen **&quot;Behandling afsluttet&quot;** vises i fejlfindingsloggen
* Startknappen bliver aktiveret igen
* Alle outputfiler findes i undermappen for kameramodellen

***

## Næste trin

Når behandlingen er afsluttet:

1. **Gennemgå resultaterne** – Se [Afslutning af behandlingen](finishing-the-processing.md)
2. **Kontroller outputmappen** – Kontroller, at alle filer er eksporteret korrekt
3. **Gennemgå fejlfindingsloggen** – Kontroller for advarsler eller fejl
4. **Vis forhåndsvisning af behandlede billeder** – Brug billedviseren eller ekstern software

For information om gennemgang og brug af dine behandlede resultater, se [Afslutning af behandlingen](finishing-the-processing.md).

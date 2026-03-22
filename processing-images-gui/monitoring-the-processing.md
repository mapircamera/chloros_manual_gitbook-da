# Overvågning af behandlingen

Når behandlingen er gået i gang, tilbyder Chloros flere måder at overvåge fremskridtet på, kontrollere for problemer og få indsigt i, hvad der sker med dit datasæt. Denne side forklarer, hvordan du følger med i behandlingen og fortolker de oplysninger, som Chloros leverer.

## Oversigt over statusbjælken

Statusbjælken i den øverste header viser behandlingsstatus i realtid og procentdelen af færdiggørelse.

### Statusbjælke i gratis tilstand

For brugere uden Chloros+-licens:

**2-trins statusvisning:**

1.**Målregistrering** – Find kalibreringsmål i billeder
2. **Behandling** – Anvendelse af korrektioner og eksport**Statusbjælken viser:**

* Samlet færdiggørelsesprocent (0-100 %)
* Navn på den aktuelle fase
* Enkel vandret bjælkevisualisering

### Chloros+ statusbjælke

For brugere med Chloros+-licens:

**4-trins fremskridtsvisning:**

1.**Detektering** – Find kalibreringsmål
2. **Analyse** – Undersøg billeder og forbered pipeline
3. **Kalibrering** – Anvend vignetterings- og reflektanskorrektioner
4. **Eksport** – Gem behandlede filer**Interaktive funktioner:*** **Hold musen over** statusbjælken for at se det udvidede 4-trins panel
* **Klik på** statusbjælken for at fastfryse/fastgøre det udvidede panel
* **Klik igen** for at ophæve fastfrysningen og automatisk skjule panelet, når musen fjernes
* Hvert trin viser den individuelle fremdrift (0-100 %)

***

## Forståelse af hvert behandlingstrin

{% hint style="info" %}
**Pipeline-arkitektur**: Disse 4 GUI-trin svarer til [4-tråds behandlingspipeline](../processing-architecture/processing-pipeline.md). På systemer med GPU-acceleration drager tråd 3 (Kalibrering) fordel af [Dynamisk beregningsadaptation](../processing-architecture/dynamic-compute-adaptation.md), som optimerer behandlingen til din specifikke hardware.
{% endhint %}

### Trin 1: Registrering (målregistrering)

**Hvad sker der:**

* Chloros scanner billeder markeret med afkrydsningsfeltet Mål
* Computervisionsalgoritmer identificerer de 4 kalibreringspaneler
* Refleksionsværdier udtrækkes fra hvert panel
* Mål-tidsstempler registreres for korrekt planlægning af kalibrering

**Varighed:**

* Med markerede mål: 10-60 sekunder
* Uden markerede mål: 5-30+ minutter (scanner alle billeder)

**Statusindikator:**

* Registrering: 0 % → 100 %
* Antal scannede billeder
* Antal fundne mål

**Hvad man skal være opmærksom på:**

* Bør være hurtigt færdig, hvis målene er markeret korrekt
* Hvis det tager for lang tid, er målene muligvis ikke markeret
* Tjek fejlfindingsloggen for &quot;Mål fundet&quot;-meddelelser

### Trin 2: Analyse

**Hvad der sker:**

* Læser billedets EXIF-metadata (tidsstempler, eksponeringsindstillinger)
* Fastlæggelse af kalibreringsstrategi baseret på målernes tidsstempler
* Organisering af billedbehandlingskøen
* Forberedelse af parallelle behandlingsprocesser (kun Chloros+)

**Varighed:** 5-30 sekunder**Statusindikator:**

* Analyse: 0 % → 100 %
* Hurtig fase, afsluttes normalt hurtigt

**Hvad skal man være opmærksom på:**

* Skal forløbe jævnt uden pauser
* Advarsler om manglende metadata vises i fejlfindingsloggen

### Fase 3: Kalibrering

**Hvad sker der:*** **Debayering**: Konvertering af RAW-Bayer-mønster til 3 kanaler
* **Vignettekorrektion**: Fjernelse af mørkningen ved linsens kanter
* **Reflektanskalibrering**: Normalisering med målværdier
* **Indeksberegning**: Beregning af multispektrale indekser
* Behandling af hvert billede gennem hele processen

**Varighed:** Størstedelen af den samlede behandlingstid (60-80 %)**Statusindikator:**

* Kalibrering: 0 % → 100 %
* Aktuelt billede under behandling
* Færdigbehandlede billeder / Samlet antal billeder

**Behandlingsadfærd:*** **Fri tilstand**: Behandler ét billede ad gangen i rækkefølge
* **Chloros+ tilstand**: Behandler op til 16 billeder samtidigt
* **GPU-acceleration**: Fremskynder denne fase betydeligt**Hvad man skal holde øje med:**

* Jævn fremgang gennem billedtællingen
* Tjek fejlfindingsloggen for meddelelser om færdiggørelse pr. billede
* Advarsler om billedkvalitet eller kalibreringsproblemer

### Fase 4: Eksport

**Hvad sker der:**

* Skriver kalibrerede billeder til disken i det valgte format
* Eksporterer multispektrale indeksbilleder med LUT-farver
* Opretter undermapper for kameramodeller
* Bevarer originale filnavne med passende suffikser

**Varighed:** 10-20 % af den samlede behandlingstid**Fremdriftsindikator:**

* Eksport: 0 % → 100 %
* Filer, der skrives
* Eksportformat og destination

**Hvad skal man holde øje med:**

* Advarsler om diskplads
* Fejl ved filskrivning
* Afslutning af alle konfigurerede output

***

## Fanen Debug Log

Debug Log giver detaljerede oplysninger om behandlingsforløbet og eventuelle problemer, der opstår.

### Adgang til Debug Log

1. Klik på ikonet **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> i venstre sidepanel
2. Logpanelet åbnes og viser behandlingsmeddelelser i realtid
3. Ruller automatisk for at vise de seneste meddelelser

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

**Handling:** Stop behandlingen, løs fejlen, genstart

### Almindelige logmeddelelser

| Meddelelse                          | Betydning                                | Nødvendig handling                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| &quot;Mål fundet i \[filnavn]&quot; | Kalibreringsmål fundet  | Ingen - normalt                                         |
| &quot;Behandler billede X af Y&quot;        | Opdatering af aktuelt fremskridt                | Ingen - normalt                                         |
| &quot;Ingen mål fundet&quot;               | Ingen kalibreringsmål fundet        | Marker målbilleder eller deaktiver reflektanskalibrering |
| &quot;Utilstrækkelig diskplads&quot;        | Ikke nok lagerplads til output          | Frigør diskplads                                    |
| &quot;Springer beskadiget fil over&quot;        | Billedfilen er beskadiget                  | Kopier filen igen fra SD-kortet                             |
| &quot;PPK-data anvendt&quot;               | GPS-korrektioner fra .daq-fil anvendt | Ingen - normalt                                         |

### Kopiering af logdata

Sådan kopieres loggen til fejlfinding eller support:

1. Åbn panelet Debug Log
2. Klik på knappen **&quot;Copy Log&quot;** (eller højreklik → Vælg alt)
3. Indsæt i tekstfil eller e-mail
4. Send til MAPIR-support, hvis nødvendigt

***

## Overvågning af systemressourcer

### CPU-forbrug

**Fri tilstand:**

* 1 CPU-kerne på ~100 %
* Andre kerner inaktive eller tilgængelige
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

**Typisk forbrug:**

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
* Task Manager → Ydeevne → GPU
* GPU-Z eller lignende overvågningsværktøj

### Disk-I/O

**Hvad kan forventes:**

* Høj disk-læseaktivitet under analysefasen
* Høj disk-skriveaktivitet under eksportfasen
* SSD er betydeligt hurtigere end HDD

**Tip til ydeevne:**

* Brug SSD til projektmappen, når det er muligt
* Undgå netværksdrev til store datasæt
* Sørg for, at disken ikke er næsten fyldt (påvirker skrivehastigheden)

***

## Registrering af problemer under behandlingen

### Advarselstegn

**Fremskridt går i stå (ingen ændring i 5+ minutter):**

* Tjek fejlloggen for fejl
* Kontroller, om der er ledig diskplads
* Tjek Task Manager for at sikre, at Chloros kører

**Fejlmeddelelser vises hyppigt:**

* Stop behandlingen og gennemgå fejlene
* Almindelige årsager: diskplads, beskadigede filer, hukommelsesproblemer
* Se afsnittet Fejlfinding nedenfor

**Systemet reagerer ikke:**

* Chloros+ parallel mode bruger for mange ressourcer
* Overvej at reducere antallet af samtidige opgaver eller opgradere hardware
* Free mode er mindre ressourcekrævende

### Hvornår skal behandlingen stoppes

Stop behandlingen, hvis du ser:

* ❌ Fejlmeddelelser som &quot;Disk fuld&quot; eller &quot;Kan ikke skrive fil&quot;
* ❌ Gentagne fejl med beskadigede billedfiler
* ❌ Systemet er helt frosset (reagerer ikke)
* ❌ Du har opdaget, at der er konfigureret forkerte indstillinger
* ❌ Forkerte billeder er importeret

**Sådan stopper du:**

1. Klik på**Stop/Annuller-knappen** (erstatter Start-knappen)
2. Behandlingen standses, fremskridt går tabt
3. Løs problemerne og start forfra

***

## Fejlfinding under behandling

### Behandlingen er meget langsom

**Mulige årsager:**

* Umarkerede målbilleder (scanner alle billeder)
* HDD i stedet for SSD-lager
* Utilstrækkelige systemressourcer
* Mange indekser konfigureret
* Adgang til netværksdrev

**Løsninger:**

1. Hvis du lige er startet og er i detekteringsfasen: Annuller, markér mål, genstart
2. Til fremtiden: Brug SSD, reducer indekser, opgrader hardware
3. Overvej CLI til batchbehandling af store datasæt

### Advarsler om &quot;diskplads&quot;

**Løsninger:**

1. Frigør diskplads med det samme
2. Flyt projektet til et drev med mere plads
3. Reducer antallet af indekser, der skal eksporteres
4. Brug JPG-format i stedet for TIFF (mindre filer)

### Hyppige meddelelser om &quot;beskadigede filer&quot;

**Løsninger:**

1. Kopier billederne igen fra SD-kortet for at sikre integriteten
2. Test SD-kortet for fejl
3. Fjern beskadigede filer fra projektet
4. Fortsæt behandlingen af de resterende billeder

### Systemoverophedning / begrænsning

**Løsninger:**

1. Sørg for tilstrækkelig ventilation
2. Rengør støv fra computerens ventilationsåbninger
3. Reducer behandlingsbelastningen (brug Free-tilstand i stedet for Chloros+)
4. Behandl i de køligere timer af døgnet

***

## Meddelelse om afsluttet behandling

Når behandlingen er færdig:

* Statusbjælken når 100 %
* Meddelelsen **&quot;Behandlingen er afsluttet&quot;** vises i fejlfindingsloggen
* Startknappen bliver aktiveret igen
* Alle outputfiler findes i undermappen for kameramodellen

***

## Næste trin

Når behandlingen er afsluttet:

1. **Gennemgå resultaterne** – Se [Afslutning af behandlingen](finishing-the-processing.md)
2. **Kontroller outputmappen** – Kontroller, at alle filer er eksporteret korrekt
3. **Gennemgå fejlfindingsloggen** – Kontroller for eventuelle advarsler eller fejl
4. **Vis forhåndsvisning af behandlede billeder** – Brug billedfremviseren eller ekstern software

For information om gennemgang og brug af dine behandlede resultater, se [Afslutning af behandlingen](finishing-the-processing.md).

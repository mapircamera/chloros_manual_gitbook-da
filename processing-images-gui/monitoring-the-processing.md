# Overvågning af behandlingen

Når behandlingen er gået i gang, tilbyder Chloros

flere måder at overvåge fremskridtet på, kontrollere for problemer og få indsigt i, hvad der sker med dit datasæt. Denne side forklarer, hvordan du kan følge med i din behandling og fortolke de oplysninger, Chloros

leverer.

## Oversigt over fremskridtsbjælken

Statusbjælken i den øverste header viser behandlingsstatus i realtid samt procentdelen af færdiggørelse. Status opdateres live fra backend via Server-Sent Events (SSE), så bjælken afspejler, hvad behandlingsforløbet rent faktisk foretager sig.

### Statusbjælke i gratis tilstand

For brugere uden Chloros

+-licens:

**2-trins fremskridtsvisning:**

1.**Målregistrering** – Find kalibreringsmål i billeder
2. **Behandling** – Anvendelse af korrektioner og eksport**Fremskridtsbjælken viser:**

* Samlet færdiggørelsesprocent (0-100 %)
* Navnet på den aktuelle fase
* Enkel vandret bjælkevisning

### Chloros

+-statusbjælke

For brugere med Chloros

+-licens:

**4-trins statusvisning:**

1.**Detektering** – Findes kalibreringsmål
2. **Analyse** – Undersøgelse af billeder og forberedelse af pipeline
3. **Kalibrering** – Anvendelse af vignetterings- og reflektanskorrektioner
4. **Eksport** – Gemning af behandlede filer**Interaktive funktioner:*** **Hold musen over** statusbjælken for at se det udvidede panel med de 4 trin
* **Klik på** statusbjælken for at fastfryse/fastgøre det udvidede panel
* **Klik igen** for at ophæve fastfrysningen, så panelet skjules automatisk, når musen fjernes
* Hvert trin viser den individuelle fremdrift (0-100 %)

{% hint style="info" %}
**Paritet med CLI**: Under en `chloros-cli process`-kørsel rapporterer de samme fire tråde som »Detecting«, »Analyzing«, »Processing« og »Exporting«, og `chloros-cli export-status` viser live fremskridt for eksport af tråd 4 fra en anden terminal. Se [CLI

-referencen](../reference/cli-reference.md).
{% endhint %}

***

## Forståelse af de enkelte behandlingsfaser

{% hint style="info" %}
**Pipeline-arkitektur**: Disse 4 GUI-faser svarer til [4-tråds-behandlingspipeline](../processing-architecture/processing-pipeline.md). På systemer med GPU-acceleration drager tråd 3 (Kalibrering) fordel af [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md), som optimerer behandlingen til din specifikke hardware.
{% endhint %}

### Trin 1: Registrering (Målregistrering)

**Hvad sker der:**

* Chloros

scanner de billeder, du har markeret med afkrydsningsfeltet »Mål« (alle billeder, hvis ingen er markeret)
* Computervisionsalgoritmer identificerer kalibreringspanelerne
* Refleksionsværdier udtrækkes fra hvert panel
* Målets tidsstempler registreres for korrekt planlægning af kalibreringen

**Varighed:**

* Med markerede mål: 10–60 sekunder
* Uden markerede mål: 5–30+ minutter (scanner alle billeder)

**Statusindikator:**

* Registrering: 0 % → 100 %
* Antal scannede billeder (tæller kun de billeder, der rent faktisk scannes)
* Antal fundne mål

**Hvad man skal være opmærksom på:**

* Bør afsluttes hurtigt, hvis målene er markeret korrekt
* Hvis det tager for lang tid, er målene muligvis ikke markeret
* Tjek fejlfindingsloggen for meddelelser om »Mål fundet«

### Trin 2: Analyse

**Hvad sker der:**

* Indlæsning af billedets EXIF-metadata (tidsstempler, eksponeringsindstillinger)
* Fastlæggelse af kalibreringsstrategi baseret på målobjekternes tidsstempler og tilgængelige DAQ-downwelling-data
* Organisering af billedbehandlingskøen
* Forberedelse af parallelle behandlingsprocesser (kun Chloros

+)

**Varighed:** 5–30 sekunder**Statusindikator:**

* Analyserer: 0 % → 100 %
* Hurtig fase, afsluttes normalt hurtigt

**Hvad man skal være opmærksom på:**

* Fremskridtet bør foregå jævnt uden pauser
* Advarsler om manglende metadata vises i fejlfindingsloggen

### Trin 3: Kalibrering

**Hvad sker der:*** **Debayering**: Konvertering af RAW-Bayer-mønster til 3 kanaler (overspringes for LATTICE-monomoduler, med en bemærkning)
* **Vignetteringskorrektion**: Fjernelse af mørkningen ved objektivets kanter
* **Reflektanskalibrering**: Normalisering med målværdier og/eller DAQ-downwelling
* **Indeksberegning**: Beregning af multispektrale indekser
* Behandling af hvert billede gennem hele processen

**Varighed:** Størstedelen af den samlede behandlingstid (60–80 %)**Statusindikator:**

* Kalibrerer: 0 % → 100 %
* Det aktuelle billede behandles
* Færdigbehandlede billeder / Samlet antal billeder

**Behandlingsadfærd:*** **Fri tilstand**: Behandler ét billede ad gangen sekventielt
* **Chloros

+-tilstand**: Kører en hardware-adaptiv arbejdspool — 1–4 samtidige arbejdsprocesser på GPU-systemer (afhængigt af VRAM), én arbejdsproces pr. fysisk kerne (minus én) på rene CPU-systemer. Se [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU-acceleration**: Fremskynder denne fase betydeligt**Hvad du skal holde øje med:**

* Jævn fremgang i billedtællingen
* Tjek fejlfindingsloggen for meddelelser om færdigbehandling pr. billede
* Advarsler om billedkvalitet eller kalibreringsproblemer

### Trin 4: Eksport

**Hvad sker der:**

* De behandlede billeder skrives til disken i det valgte format, efterhånden som de færdiggøres
* **LATTICE**: Hvert billede fordeles til alle aktiverede produkter (debayered / preview / radiance / reflectance)
* Eksport af multispektrale indeksbilleder med LUT-farver
* Oprettelse af outputtræet `<project>/<camera>/<format>/<Product>_Images/` — eksporterede filer beholder kildefilnavnet; mappen identificerer produktet

**Varighed:** 10–20 % af den samlede behandlingstid**Statusindikator:**

* Eksport: 0 % → 100 %
* Filer skrives
* Eksportformat og destination

**Hvad man skal holde øje med:**

* Advarsler om diskplads
* Fejl ved filskrivning
* Afslutning af alle konfigurerede output

***

## Fanen Debug Log

Debug Log indeholder detaljerede oplysninger om forløbet af behandlingen og eventuelle problemer, der er opstået. Startmeddelelser fra backend vises også i logkonsollen, så loggen giver et fuldstændigt overblik, selvom du åbner den senere.

### Sådan får du adgang til fejlfindingsloggen

1. Klik på **Fejlfindingslog**-ikonet <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

i venstre sidepanel
2. Logpanelet åbnes og viser behandlingsmeddelelser i realtid
3. Der rulles automatisk for at vise de seneste meddelelser

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written), with the 1.3.0 bug icon (Report a bug) visible at the top of the tab -->

### Forståelse af logmeddelelser

Chloros

-loglinjer indledes med tags i parentes, der angiver navnet på undersystemet — for eksempel `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Det vigtigste at kende er **kørselsoversigten**, der vises i slutningen af hver kørsel (inklusive stoppede kørsler):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Der følger ekstra `[RUN-SUMMARY]`-hintlinjer, når noget skal forklares — for eksempel et kørsel, der ikke producerede noget, eller et kamera, hvis ønskede produkt blev sprunget over, da det ikke var relevant. `[EXPORT-CHECK]`-linjer forklarer, hvilke kameraer der er sprunget over (f.eks. hvorfor et RGB

-kamera ikke fik noget strålingsprodukt).

De generelle alvorlighedsniveauer for meddelelser (eksemplerne nedenfor er illustrative, ikke ordrette):

#### Informationsmeddelelser (hvid/grå)

Normale opdateringer om behandlingen: behandlingen er startet, mål er detekteret (med antal paneler), fremskridt i kalibrering pr. billede, filer eksporteret, behandlingen er afsluttet.

#### Advarselsmeddelelser (gul)

Ikke-kritiske problemer, der ikke stopper behandlingen — f.eks. manglende GPS-data i et billede, et stort tidsstempel-gab mellem målbilleder eller lav kontrast i et kalibreringspanel.

**Handling:** Gennemgå advarslerne efter behandlingen, men afbryd ikke

#### Fejlmeddelelser (rødRed

)

Kritiske problemer, der kan medføre, at behandlingen mislykkes — f.eks. fuld disk, en beskadiget billedfil eller ingen mål registreret, mens der blev anmodet om reflektanskalibrering.

**Handling:** Stop behandlingen, løs fejlen, genstart

### Almindelige log-situationer

| Situation                             | Betydning                                       | Nødvendig handling                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Mål fundet i \[filnavn]        | Kalibreringsmål fundet uden problemer         | Ingen – normalt                                         |
| Statuslinjer pr. billede              | Aktuel statusopdatering                       | Ingen – normalt                                         |
| Ingen mål fundet                      | Ingen kalibreringsmål fundet               | Marker målbilleder eller deaktiver reflektanskalibrering |
| Utilstrækkelig diskplads               | Ikke nok lagerplads til output                 | Frigør diskplads                                    |
| Springes over på grund af beskadiget fil               | Billedfilen er beskadiget                         | Kopier filen igen fra SD-kortet                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | En optagelse uden et råbillede kan ikke behandles | Optag igen med råbillede, eller brug CLI

`--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Kørslen producerede ingen billedprodukter — rapporteret som en fejl med tip | Læs tip-linjerne; kontroller, hvad der blev sprunget over, og hvorfor |
| `[PROCESS] refused: The project folder is too deep for this computer…` | Windows

-lange stier er slået fra, og projektmappen er for dyb i forhold til stibegrænsningen på 260 tegn; intet blev behandlet | Brug en kortere projektmappe (meddelelsen angiver den længste, der passer), eller aktiver lange stier i Windows

, og genstart Chloros

— se [Start af behandlingen](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer) |
| `[unmix-clamp-mask] …` | Det var ikke muligt at skrive en LATTICE-produkts clamp-maske; selve produktet fungerer fint og viser `Chloros:UnmixClampMask=unavailable` | Kontroller diskplads og mappetilladelser, og gentag derefter behandlingen, hvis du har brug for masken |

### Kopiering af logdata

Sådan kopieres loggen til fejlfinding eller support:

1. Åbn panelet *Debug Log*
2. Klik på knappen **Kopier**(kopierer loggen til udklipsholderen) eller**Download** for at gemme den som en tekstfil
3. Indsæt i en tekstfil eller en e-mail
4. Send til MAPIR

-support, hvis nødvendigt

### Indberetning af en fejl fra fanen *Log*

Nyt i 1.3.0: Fejlikonet (**Indberet en fejl**) øverst på fanen *Debug Log* sender en rapport direkte til MAPIR

.

1. Klik på fejlikonet øverst på fanen *Debug Log*
2. Beskriv problemet: hvad du lavede, hvad du forventede, og hvad der i stedet skete (påkrævet)
3. Du kan eventuelt markere **Tilføj et par billeder (op til 10)**og vælge billeder fra det åbne projekt eller bruge**Gennemse...** til andre billedfiler
4. Klik på **Send**

Rapporten indeholder din beskrivelse, eventuelle vedhæftede billeder, Chloros

-logfilerne samt systemoplysninger (Chloros

-version, operativsystem, GPU og projektindstillinger). Du skal være logget ind på Chloros

(fanen „Bruger“) for at kunne sende en rapport. Du kan lukke vinduet, mens rapporten sendes; når den er færdig, skrives en reference som f.eks. `Bug report sent. Reference: BR-…` ind i loggen — angiv den, når du kontakter MAPIR

. Hvis rapporten ikke kan uploades (f.eks. på grund af manglende internetforbindelse), gemmer Chloros

den på din computer og viser stien, så du kan sende den via e-mail til support@mapir.camera.

***

## Overvågning af systemressourcer

### CPU-udnyttelse

**Fri tilstand:**

* 1 CPU-kerne på ca. 100 %
* Andre kerner er inaktive eller tilgængelige
* Systemet forbliver responsivt

**Parallel tilstand i Chloros

+:**

* Flere kerner med høj udnyttelse — hvor mange afhænger af den strategi, der er valgt af [Dynamic Compute Adaptation](../processing-architecture/dynamic-compute-adaptation.md)
* Systemet kan føles mindre responsivt

**Sådan overvåges det:**

* Windows

Opgavelister (Ctrl+Shift+Esc)
* Fanen Ydeevne → afsnittet CPU
* Se efter processer med navnet &quot;Chloros

&quot; eller &quot;chloros-backend&quot;

### Hukommelsesforbrug (RAM)

**Typisk forbrug:**

* Små projekter (&lt; 100 billeder): 2–4 GB
* Mellemstore projekter (100–500 billeder): 4–8 GB
* Store projekter (500+ billeder): 8–16 GB
* Chloros

+ i parallel tilstand bruger mere RAM

**Hvis der er lidt hukommelse:**

* Behandl mindre batcher
* Luk andre programmer
* Opgrader RAM, hvis du regelmæssigt behandler store datasæt

### GPU-forbrug (Chloros

+ med CUDA)

Når GPU-acceleration er aktiveret:

* NVIDIA-GPU&#x27;en viser høj udnyttelse (60–90 %)
* VRAM-forbruget stiger (kræver 4 GB+ VRAM; 7 GB+ ved samtidig Texture Aware-debayering)
* Kalibreringsfasen er betydeligt hurtigere

**Sådan overvåges det:**

* NVIDIA-ikonet i systembakken
* Opgavelister → Ydeevne → GPU
* GPU-Z eller lignende overvågningsværktøj

### Disk-I/O

**Hvad du kan forvente:**

* Høj disk-læsehastighed under analysefasen
* Høj disk-skrivehastighed under eksportfasen
* SSD er betydeligt hurtigere end HDD

**Ydelsestip:**

* Brug en SSD til projektmappen, når det er muligt
* Undgå netværksdrev til store datasæt
* Sørg for, at disken ikke er næsten fyldt (påvirker skrivehastigheden)

***

## Opdagelse af problemer under behandlingen

### Advarselstegn

**Fremskridtet går i stå (ingen ændring i 5+ minutter):**

* Tjek fejlloggen for fejl
* Kontroller, om der er ledig diskplads
* Tjek Opgavelederen for at sikre, at Chloros

kører

**Der vises hyppigt fejlmeddelelser:**

* Stop behandlingen og gennemgå fejlene
* Almindelige årsager: diskplads, beskadigede filer, hukommelsesproblemer
* Se afsnittet om fejlfinding nedenfor

**Systemet reagerer ikke:**

* Chloros

i parallel tilstand bruger for mange ressourcer
* Overvej at reducere antallet af samtidige opgaver eller opgradere hardwaren
* Fri tilstand er mindre ressourcekrævende

### Hvornår skal behandlingen stoppes

Stop behandlingen, hvis du ser:

* ❌ Fejlmeddelelser som &quot;Disk fuld&quot; eller &quot;Kan ikke skrive fil&quot;
* ❌ Gentagne fejlmeddelelser om beskadigede billedfiler
* ❌ Systemet er helt frosset fast (reagerer ikke)
* ❌ Du har opdaget, at der er konfigureret forkerte indstillinger
* ❌ Der er importeret forkerte billeder

**Sådan stopper du:**

1. Klik på**Stop-knappen** (erstatter Start-knappen) — én gang er nok
2. Statusbjælken viser &quot;Stopper...&quot;, mens den igangværende billedbehandling afsluttes, hvorefter kørslen afsluttes i en stoppet tilstand
3. Produkter, der allerede er eksporteret, forbliver på disken; loggen viser en præcis oversigt over, hvad der er afsluttet
4. Løs problemerne og genstart — kørslen starter forfra

***

## Fejlfinding under behandling

### Behandlingen er meget langsom

**Mulige årsager:**

* Umarkerede målbilleder (scanning af alle billeder)
* HDD i stedet for SSD-lagring
* Utilstrækkelige systemressourcer
* Mange konfigurerede indekser
* Adgang til netværksdrev

**Løsninger:**

1. Hvis processen lige er startet og er i &quot;Detecting&quot;-fasen: Stop, markér mål, genstart
2. Fremover: Brug SSD, reducer antallet af indekser, opgrader hardware
3. Overvej at bruge CLI

til batchbehandling af store datasæt

### Advarsler om &quot;diskplads&quot;

**Løsninger:**

1. Frigør diskplads med det samme
2. Flyt projektet til et drev med mere plads
3. Reducer antallet af indekser, der skal eksporteres
4. Deaktiver LATTICE-eksportprodukter, du ikke har brug for (Projektindstillinger → Behandling)
5. Brug JPG-format i stedet for TIFF

(mindre filer)

### Hyppige meddelelser om &quot;beskadigede filer&quot;

**Løsninger:**

1. Kopier billederne igen fra SD-kortet for at sikre integriteten
2. Test SD-kortet for fejl
3. Fjern beskadigede filer fra projektet
4. Fortsæt behandlingen af de resterende billeder

### Systemoverophedning / begrænsning af ydeevnen

**Løsninger:**

1. Sørg for tilstrækkelig ventilation
2. Fjern støv fra computerens ventilationsåbninger
3. Reducer behandlingsbelastningen (brug Free-tilstand i stedet for Chloros

+)
4. Udfør behandlingen på køligere tidspunkter af døgnet

***

## Meddelelse om afsluttet behandling

Når behandlingen er færdig:

* Statusbjælken når 100 %
* Linjerne `[RUN-SUMMARY]` vises i fejlfindingsloggen med de endelige tal
* Startknappen bliver aktiveret igen
* Alle outputfiler findes i projektets outputstruktur pr. kamera: `<project>/<camera>/<format>/<Product>_Images/`

***

## Næste trin

Når behandlingen er afsluttet:

1. **Gennemgå resultaterne** – Se [Afslutning af behandlingen](finishing-the-processing.md)
2. **Kontroller outputmappen** – Kontroller, at alle filer er eksporteret korrekt
3. **Gennemgå fejlfindingsloggen** – Kontroller for eventuelle advarsler eller fejl
4. **Se forhåndsvisning af de behandlede billeder** – Brug billedviseren eller ekstern software

For information om gennemgang og brug af dine behandlede resultater, se [Afslutning af behandlingen](finishing-the-processing.md).

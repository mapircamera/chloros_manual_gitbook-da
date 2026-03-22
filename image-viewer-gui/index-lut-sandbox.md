# Indeks/LUT-sandkasse

Indeks/LUT-sandkassen er et interaktivt arbejdsområde i Chloros Image Viewer, der giver dig mulighed for at eksperimentere med beregning af multispektrale indekser og farvevisualiseringer i realtid. Dette kraftfulde værktøj hjælper dig med at afprøve forskellige indekser, finjustere værdiintervaller og oprette visualiseringer, der er klar til offentliggørelse, uden at skulle genbehandle hele dit datasæt.

## Hvad er Index/LUT Sandbox?

### Formål

Sandboxen tilbyder:

* **Indeksberegning i realtid** – Anvend ethvert vegetationsindeks med det samme
* **Interaktiv LUT-justering** – Finjuster farvegradienter og -intervaller
* **Optimering af arbejdsgangen** – Bestem de bedste indstillinger inden batchbehandling

### Sandkasse vs. projektbehandling

**Indeks/LUT-sandkasse (interaktiv):**

* Ét billede ad gangen
* Øjeblikkelig feedback
* Eksperimentel og iterativ
* Ingen permanente ændringer af filer
* Perfekt til udforskning og test

**Projektbehandling (batch):**

* Hele datasættet på én gang
* Forudkonfigurerede indstillinger
* Permanente outputfiler
* Tidskrævende
* Bedst, når indstillingerne er endelige

{% hint style="success" %}
**Bedste arbejdsgang**: Brug Sandbox til at eksperimentere og finde optimale indeks- og LUT-indstillinger, og anvend derefter disse indstillinger under projektbehandling for hele dit datasæt.
{% endhint %}

***

## Arbejde med indeks/LUT-sandkassen

### Forståelse af forudberegnede indekser

I Chloros kan indekser anvendes under projektbehandlingen. For at afgøre, hvilke indeks- og LUT-indstillinger du vil anvende på eksportfiler, er det nemmest at bruge billedfremviserens sandkasse.

Sandkassen giver dig mulighed for at:

* **Anvende nye indekser og farvegradienter (LUT&#x27;er)** til at visualisere dataene
* **Justere visualiseringsindstillinger** interaktivt
* **Se** allerede beregnede indeksbilleder
* **Undersøge** pixelværdier på alle zoomniveauer

### Åbning af sandkassen

Du får adgang til indeks-/LUT-sandkassen i **billedfremviseren** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sidebjælkefanen:

1. Klik på et billede i filbrowserens billedgitter, så åbnes det i fanen **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> fanen
2. Klik på **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> for at åbne den venstre pop-out-sidebjælke, hvis den ikke allerede er åben

### Valg af et billede, som indeks/LUT skal anvendes på

Sådan arbejder du med et indeks i <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sandkasse:

1. **Åbn et billede** fra hovedbilledgitteret ved at klikke på det
2. Fanen **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> fanen åbnes derefter
3. Klik på **Layer-dropdownmenuen** (øverst til højre i visningen)
4. Vælg laget fra dropdownmenuen:
   * RAW (Reflectance)

### Anvendelse af et indeks på et billede

Når billedet er i fuldskærm, og **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sidebjælken er åben:

1. Marker feltet Indeks øverst i sidebjælken
2. Vælg dit kameras filter fra dropdown-menuen til venstre
3. Vælg den ønskede indeksformel fra dropdown-menuen til højre
4. Træk filterkanalens farvecirkler til placeringerne i indeksformlen nedenfor
5. Når formlen er gyldig, opdateres billedet og viser indeksværdierne
6. Flyt musemarkøren rundt for at se værdierne ved markørens placering
7. Zoom ind for at se de enkelte pixels og deres tilhørende værdier

Hvert indeks har et specifikt værdiinterval og en specifik betydning:

#### NDVI Eksempel

```

Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

For fuldstændig dokumentation af indeksformler, se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md).

***

## Arbejde med LUT&#x27;er (opslagstabeller)

### Hvad er en LUT?

En **opslagstabel (LUT)** knytter numeriske indeksværdier til farver med henblik på visualisering:

* **Indgang**: Indekspixelværdi (f.eks. NDVI 0,65)
* **Output**: RGB-farve (f.eks. lysegrøn)
* **Formål**: Gør mønstre lettere at se og fortolke**Gråtoner vs. farve-LUT:**

* Gråtoner: Videnskabeligt og neutralt, viser rådata
* Farve-LUT: Intuitivt og effektfuldt, fremhæver mønstre og forskelle

{% hint style="success" %}
**Visualiseringskraft**: Anvendelse af en farve-LUT på et gråtone-indeksbillede gør det markant nemmere at identificere mønstre, afvigelser og områder af interesse med et enkelt blik.
{% endhint %}

### Anvendelse af en LUT på et indeksbillede

Når du har et indeksbillede, der viser

1. Klik på <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> &quot;+Tilføj LUT&quot;-knappen
2. Vælg farveovergangen
3. Juster klipningens minimums- og maksimumsendepunkter
4. Juster klipningstilstanden
5. Marker afkrydsningsfeltet Indeks i **Billedfremviser** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> for at anvende LUT&#x27;en

### Valg af farvegradient

**Valg af gradient:**

1. Find den**farvede gradientbjælke** i LUT-panelet
2. Hold musen over den for at se de tilgængelige gradientforudindstillinger
3. Vælg den ønskede gradient
4. Billedet **opdateres straks** med nye farver, når afkrydsningsfeltet Indeks er markeret

{% hint style="success" %}
**Bedste praksis**: For vegetationsindekser som NDVI er farveovergangen Red-Gul-Green mest intuitiv, da den stemmer overens med naturlige farveassociationer (grøn = sund, gul = moderat, rød = stresset).
{% endhint %}

### Justering af farveklasser

**Kontrolelementet Klasser**bestemmer, hvor mange diskrete farvestrinn der vises i din gradient:**Indstillinger for antal klasser:*** **2-5 klasser**: Meget brede kategorier, tydelige zoner
* **6-10 klasser**: Afbalanceret, godt til klassificering
* **11-20 klasser**: Glatte farveovergange, kontinuerligt udseende
* **20+ klasser**: Næsten kontinuerlig, maksimal glathed**Sådan justeres:**

1. Find**farveprøvefelterne under farveovergangsstangen** i LUT-panelet
2. Juster antallet af klasser ved at tilføje med +-knappen
3. Fjern antallet af klasser ved at dobbeltklikke på en farveprøve
4. Gradienten opdateres **i realtid** på billedet**Effekt på visualiseringen:*** **Færre klasser** (3-5): Skaber tydelige zoner, forenklet klassificering, lettere at skelne mellem kategorier
* **Mellemklasse** (6-10): Afbalanceret tilgang, velegnet til de fleste anvendelser
* **Flere klasser** (15-20): Glatte overgange, detaljeret variation, fotografisk udseende**Hvornår skal det bruges:*** **Få klasser (3-5)**: Præsentationsslides, klassifikationskort, enkle rapporter
* **Mellemstore klasser (6-10)**: Generel analyse, afbalancerede detaljer, standardrapporter
* **Mange klasser (15-20)**: Videnskabelig analyse, detaljeret inspektion, output i publikationskvalitet

### Finjustering af værdiintervaller

**Værdiintervalkontrollerne**bestemmer, hvilke indeksværdier der skal knyttes til hvilke farver i din farveovergang:**Intervalkontroller i LUT-panelet:*** **Minimumsværdi**: Den nedre grænse for farveskalaen
* **Maksimumsværdi**: Den øvre grænse for farveskalaen
* **Mellemværdier**: Fordeles automatisk mellem min. og maks. (baseret på antallet af klasser)

#### Justering af min./maks. værdier

**Sådan justeres værdiintervaller:**

1. Find indtastningsfelterne**Min. værdi**og**Maks. værdi** i LUT-panelet
2. Klik på feltet **Min. værdi**

3. Indtast den ønskede minimumsværdi (f.eks. `0.2`)
4. Tryk på **Enter** eller klik uden for feltet
5. Gentag for feltet **Maks. værdi** (f.eks. `0.9`)
6. Visualiseringen **opdateres med det samme**{% hint style="info" %}**Automatisk skalering**: Når du først anvender en LUT, indstiller Chloros automatisk min/maks til det faktiske datainterval i billedet. Du kan derefter indsnævre dette interval for at fokusere på specifikke værdiintervaller af interesse.
{% endhint %}

**Eksempel på justering af NDVI-interval:*** **Fuldt interval**: `-1.0` til `1.0` (vis alle mulige værdier)
* **Fokuseret på vegetation**: `0.2` til `0.9` (ekskluderer bar jord og vand)
* **Kun sund vegetation**: `0.5` til `0.9` (fremhæv kun livskraftige planter)
* **Stressdetektering**: `0.2` til `0.5` (fremhæv problemområder)
* **Brugerdefineret interval**: Juster baseret på dine observerede pixelværdier**Hvorfor justere intervaller?*** **Forøg kontrasten** i dit interesseområde
* **Udeluk irrelevante værdier** (f.eks. vandområder, bar jord)
* **Standardiser visualiseringen** på tværs af flere billeder eller datoer
* **Fremhæv subtile forskelle** inden for et snævert værdiinterval

### Beskæring af værdier uden for området

Når pixelværdier falder uden for dit definerede min./maks.-område, kan du styre, hvordan de vises, ved hjælp af **beskæringsmetoder**.

#### **Tilgængelige indstillinger for beskæringsmetoder:**

#### 1. Minimum og maksimum

* Pixels **under minimum**→ vises ved hjælp af den**første farve** i farveovergangen (f.eks. rød)
* Pixels **over maksimum**→ vises ved hjælp af den**sidste farve** i farveovergangen (f.eks. grøn)
* **Anvendelsestilfælde**: Fremhæv ekstremværdier, vis det fulde dataområde med mættede farver ved grænserne
* **Eksempel**: NDVI-værdier under 0,2 vises alle røde, værdier over 0,9 vises alle grønne

#### 2. Gennemsigtig baggrund

* Pixels **uden for området**bliver**fuldstændigt gennemsigtige*** Kun pixels **inden for området** viser farvegradient
* **Anvendelsestilfælde**: GIS-overlay, isolering af specifikke værdiområder, fremhævning af kun områder af interesse
* **Eksempel**: Vis kun NDVI 0,4-0,7 i farve, alt andet gennemsigtigt

{% hint style="warning" %}
**Begrænsning af gennemsigtighed**: Gennemsigtige pixels vises som baggrundsfarven i visningsprogrammet. Når der eksporteres under behandlingen, bevares gennemsigtigheden i PNG-format, men ikke i JPG.
{% endhint %}

#### 3. Indeksbaggrund

* Pixels **uden for området**vises i**gråtoner** (viser rå indeksværdier)
* Pixels **inden for området**viser**farvegradient*** **Anvendelsestilfælde**: Diskret fremhævning, bevarer konteksten, mens områder af interesse fremhæves
* **Eksempel**: Farvefremhæv stresset vegetation (NDVI 0,3-0,5), mens sunde områder vises i gråt

#### 4. Originalbaggrund

* Pixels **uden for området**viser det**originale multispektrale billede*** Pixels **inden for området**viser**farvegradient*** **Anvendelsestilfælde**: Mest intuitivt – kombinerer naturlig billedkontekst med analytisk farveoverlay
* **Eksempel**: Se det faktiske udseende af marken/afgrøden med farvekodede stressområder overlejret

### Valg af den rigtige klippemodus

| Klippemodus              | Bedst egnet til                                   | Visualiseringsstil          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum og maksimum**    | Fuld datavisning, videnskabelig analyse     | Alle pixels farvelagte           |
| **Gennemsigtig baggrund** | GIS-overlejringer, isolering af specifikke intervaller    | Farve inden for intervallet, tomt udenfor |
| **Indeksbaggrund**       | Diskret fremhævning, bevarer datakonteksten  | Farve på området, gråt udenfor  |
| **Originalbaggrund**    | Rapporter, præsentationer, intuitiv analyse | Farve på området, foto udenfor |

### Oprettelse af brugerdefinerede LUT-farver

For fuld kontrol over din visualisering kan du oprette **brugerdefinerede farveovergange** ved at redigere individuelle farvestop.**Sådan opretter du en brugerdefineret gradient:**

1. Find**gradientforhåndsvisningsbjælken** i LUT-panelet
2. Se efter **farveprøvefelterne** under gradienten
3. **Klik på et farvestop** for at vælge det
4. En **farvevælger** åbnes
5. Vælg en ny farve ved hjælp af:
   * **Farvehjul**: Visuel farvevalg
   * **RGB/HSV-skydeknapper**: Præcis farvekontrol
   * **Indtastning af hex-kode**: Præcis farvespecifikation (f.eks. `#FF0000` for rød)
6. Klik uden for farvevælgeren **for at anvende den nye farve**

7. Gradienten**opdateres straks** på billedet**Tilføjelse eller fjernelse af farvestop:*** **Tilføj et stop**: Klik på +-ikonet for at tilføje en ny farveprøve i slutningen
* **Fjern et stop**: Dobbeltklik på farvefeltet for at fjerne farveprøven**Tilpasningsstrategier:*** **Inverter gradient**: Vend farverækkefølgen for at vende betydningen (f.eks. grøn=lav, rød=høj)
* **Brandfarver**: Tilpas til din organisations farvepalet til rapporter
* **Farveblind-venlig**: Brug orange-blå eller lilla-gule kombinationer
* **Udskriftsoptimering**: Vælg farver, der fungerer både ved farve- og gråtoneudskrivning
* **Multi-tærskel**: Brug forskellige farver ved specifikke værditærskler til klassificering

{% hint style="info" %}
**Gemning af brugerdefinerede gradienter**: Brugerdefinerede gradienter kan gemmes og genbruges. Klik på gem-ikonet i LUT-panelet for at gemme dine brugerdefinerede farveskemaer til fremtidig brug.
{% endhint %}

***

## Interaktivt workflow

### Opdateringer i realtid

Alle LUT-justeringer i sandkassen opdaterer billedet **øjeblikkeligt og interaktivt**:

* **Skift lag** → Billedet ændres med det samme
* **Vælg gradient** → Farverne opdateres øjeblikkeligt
* **Juster værdiinterval** → Kontrasten ændres i realtid
* **Skift klasser** → Gradientens glathed opdateres med det samme
* **Rediger klipning** → Baggrundsvisningen ændres øjeblikkeligt
* **Rediger farver** → Brugerdefineret gradient anvendes straks**Ingen &quot;Anvend&quot;-knap nødvendig** – alle ændringer er live og interaktive!

{% hint style="success" %}
**Live-feedback**: Den øjeblikkelige visuelle feedback giver dig mulighed for hurtigt at eksperimentere med forskellige indstillinger, indtil du finder den optimale visualisering til dine analysebehov.
{% endhint %}

### Iterativ forfiningsworkflow

**Typisk LUT-optimeringsworkflow:**

1.**Vælg indekslag** (f.eks. RAW (Reflektans))
2. **Anvend indeks** – Vælg kamerafilter og indeksformel, træk de farvede cirkler til den rette placering i indeksformlen
3. **Anvend LUT-gradient** – Start med forudindstillingen Red-Yellow-Green
4. **Undersøg pixelværdier** – Flyt markøren rundt, bemærk værdiintervaller
5. **Juster min/maks** – Indsnæv for at fokusere på vegetation (f.eks. 0,2 til 0,9)
6. **Vælg klipning** – Prøv &quot;Original baggrund&quot; for at få kontekst
7. **Finjuster farver** – Tilpas gradienten, hvis det er nødvendigt for specifik fremhævning
8. **Færdiggør indstillinger**– Dokumenter indstillingerne og kopier dem til projektindstillingerne til eksportbehandling

### Inspektion af pixelværdier

Det er afgørende at forstå de faktiske pixelværdier for at kunne indstille effektive LUT-intervaller:**Sådan inspiceres værdier:**

1. Pixelværdier vises, når billedet har enten**Indeks**eller både**Indeks**og**LUT** **markeret**.
2. **Flyt markøren** over forskellige områder af billedet
3. **Observer pixelværdier**, der vises i legenden, når du holder markøren over dem
4. Zoom ind for at se individuelle pixels fremhævet med en flydende værdi
5. **Noter** værdier for forskellige elementer:
   * **Sund vegetation**: f.eks. NDVI 0,55-0,85
   * **Stresset vegetation**: f.eks. NDVI 0,30-0,50
   * **Bare jord**: f.eks. NDVI 0,05-0,25
   * **Vand** (hvis til stede): f.eks. NDVI -0,05 til 0,10**Brug af pixelværdier til at indstille LUT-intervaller:**Efter at have undersøgt pixelværdierne skal du justere din LUT-min/max i overensstemmelse hermed:**Eksempelscenarie:*** **Observation**: Jordværdier = 0,05-0,25, Stresset = 0,25-0,50, Sund = 0,50-0,85
* **Mål**: Visualiser kun plantens sundhed (ekskluder jord)
* **LUT-indstillinger**: Min = `0.25`, Max = `0.85`
* **Klipning**: &quot;Original baggrund&quot; for at se jorden i naturlige farver
* **Resultat**: Farvegradienten gælder kun for vegetation, jorden vises som det originale billede

{% hint style="info" %}
**Dynamisk rækkevidde**: Forskellige afgrøder, årstider og vækststadier vil have forskellige værdiintervaller. Kontroller altid pixelværdierne i dit specifikke datasæt, før du indstiller LUT-intervaller.
{% endhint %}

***

## Brugerdefinerede indekser (Chloros+)

### Oprettelse af brugerdefinerede indeksformler

{% hint style="info" %}
**Hvor skal det oprettes**: Brugerdefinerede indekser kan konfigureres i**Projektindstillinger** før behandling samt i sidepanelet i Image Viewer-sandkassen.
{% endhint %}

**Sådan oprettes et brugerdefineret indeks:**

1.**Åbn Projektindstillinger** (inden behandling) eller sidepanelet i Image Viewer-sandkassen
2. Gå til **dropdown-menuen Indeksformel**

3. Find indstillingen**&quot;Brugerdefineret&quot;** (du skal være logget ind med en Chloros+-licens)
4. **Definer din formel** ved hjælp af båndvariabler:
   * Båndnavne: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` osv.
   * Operatører: `+`, `-`, `*`, `/`, `^` (eksponent)
   * Funktioner: `sqrt()`, `abs()` osv. (hvis understøttet)
   * Parenteser: `()` for rækkefølgen af operationer
5. **Navngiv dit indeks** (f.eks. &quot;MyIndex&quot; eller &quot;CustomNDVI&quot;)
6. **Gem konfigurationen**

**Eksempler på brugerdefinerede formler:**

```

Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style="warning" %}
**Formelvalidering**: Sørg for, at din formel bruger bånd, der er tilgængelige i dit kamera. For eksempel er RedEdge kun tilgængelig på kameraer med et RedEdge-filter.
{% endhint %}

***

## Næste trin

Nu hvor du forstår Index/LUT Sandbox:

* **Anvend til behandling**: Brug de fundne indstillinger i [Projektindstillinger](../project-settings/project-settings.md)
* **Batch-behandling**: Anvend optimerede indekser på fulde datasæt
* **Lær mere**: Læs [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md)

Relateret dokumentation:

* [**Billedlag**](image-layers.md) - Lagstyring og visualisering
* [**Åbning af et billede i fuld skærm**](opening-an-image-full-screen.md) - Grundlæggende om billedviseren
* [**Behandling af billeder (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Fuld behandlingsworkflow

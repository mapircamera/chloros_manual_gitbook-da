# Indeks/LUT Sandkasse

Indeks/LUT Sandkasse er et interaktivt arbejdsområde i Chloros Image Viewer, der giver dig mulighed for at eksperimentere med multispektrale indeksberegninger og farvevisualiseringer i realtid. Dette kraftfulde værktøj hjælper dig med at teste forskellige indekser, finjustere værdiintervaller og oprette publikationsklare visualiseringer uden at skulle genbehandle hele dit datasæt.

## Hvad er Index/LUT Sandbox?

### Formål

Sandbox tilbyder:

* **Indeksberegning i realtid** – Anvend ethvert vegetationsindeks med det samme
* **Interaktiv LUT-justering** – Finjuster farvegradienter og -intervaller
* **Workflowoptimering** – Bestem de bedste indstillinger inden batchbehandling

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
* Bedst, når indstillingerne er færdiggjort

{% hint style=&quot;success&quot; %}
**Bedste arbejdsgang**: Brug Sandbox til at eksperimentere og finde optimale indeks- og LUT-indstillinger, og anvend derefter disse indstillinger under projektbehandling for hele dit datasæt.
{% endhint %}

***

## Arbejde med indeks/LUT Sandbox

### Forståelse af forudberegnede indekser

I Chloros kan indekser anvendes under projektbehandlingen. For at bestemme, hvilke indeks- og LUT-indstillinger du vil anvende på eksporten, er det nemmest at bruge billedvisningssandkassen.

Sandkassen giver dig mulighed for at:

* **Anvende nye indekser og farvegradienter (LUT&#x27;er)** til at visualisere dataene
* **Justere visualiseringsindstillinger** interaktivt
* **Se** allerede beregnede indeksbilleder
* **Inspicere** pixelværdier på alle zoomniveauer

### Åbning af sandkassen

Index/LUT-sandkassen åbnes i **Billedviseren** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sidebjælkefanen:

1. Klik på et billede i filbrowseren, så åbnes det i **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> fanen
2. Klik på **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> for at åbne den venstre pop-out sidebjælke, hvis den ikke allerede er åben

### Valg af et billede, som der skal anvendes et indeks/LUT på

For at arbejde med et indeks i billedviseren <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sandkasse:

1. **Åbn et billede** fra hovedbilledgitteret ved at klikke på det
2. Fanen **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> åbnes derefter
3. Klik på **Layer dropdown** (øverst til højre i viseren)
4. Vælg laget fra dropdown-menuen:
   * RAW (Reflectance)

### Anvendelse af et indeks på et billede

Når billedet er i fuld skærm, og sidebjælken **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> er åben:

1. Marker afkrydsningsfeltet Indeks øverst i sidepanelet
2. Vælg dit kameras filter fra dropdown-menuen til venstre
3. Vælg den ønskede indeksformel fra dropdown-menuen til højre
4. Træk filterkanalens farvecirkler til placeringerne i indeksformlen nedenfor
5. Når formlen er gyldig, opdateres billedet og viser indeksværdierne
6. Flyt musemarkøren rundt for at se værdierne på markørens placering
7. Zoom ind for at se de enkelte pixels og deres tilknyttede værdier.

Hvert indeks har et specifikt værdiområde og en specifik betydning:

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

For en komplet dokumentation af indeksformler, se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md).

***

## Arbejde med LUT&#x27;er (opslagstabeller)

### Hvad er en LUT?

En **opslagstabel (LUT)** kortlægger numeriske indeksværdier til farver til visualisering:

* **Indgang**: Indekspixelværdi (f.eks. NDVI 0,65)
* **Output**: RGB farve (f.eks. lysegrøn)
* **Formål**: Gør mønstre lettere at se og fortolke

**Gråtoner vs. farve-LUT:**

* Gråtoner: Videnskabelig og neutral, viser rådata
* Farve-LUT: Intuitiv og effektfuld, fremhæver mønstre og forskelle

{% hint style=&quot;success&quot; %}
**Visualiseringskraft**: Anvendelse af en farve-LUT på et gråtonet indeksbillede gør det betydeligt nemmere at identificere mønstre, afvigelser og områder af interesse med et enkelt blik.
{% endhint %}

### Anvendelse af en LUT på et indeksbillede

Når du har et indeksbillede, der viser

1. Klik på <img src="../.gitbook/assets/image.png" alt="" data-size="line"> knappen &quot;+Tilføj LUT&quot;
2. Vælg farvegradienten
3. Juster minimums-/maksimumsendepunkterne for beskæring
4. Juster beskæringsfunktionen
5. Marker afkrydsningsfeltet Indeks i **Billedviseren** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> -fanen i sidepanelet for at anvende LUT&#x27;en.

### Valg af farvegradient

**Valg af gradient:**

1. Find den **farvede gradientbjælke** i LUT-panelet.
2. Hold musen over den for at se de tilgængelige gradientforudindstillinger.
3. Vælg den ønskede gradient.
4. Billedet **opdateres straks** med nye farver, når afkrydsningsfeltet Indeks er markeret.

{% hint style=&quot;success&quot; %}
**Bedste praksis**: For vegetationsindekser som NDVI er Red-Yellow-Green-gradienten mest intuitiv, da den stemmer overens med naturlige farveassociationer (grøn = sund, gul = moderat, rød = stresset).
{% endhint %}

### Justering af farveklasser

**Klassekontrollen** bestemmer, hvor mange diskrete farvetrin der vises i din gradient:

**Valgmuligheder for antal klasser:**

* **2-5 klasser**: Meget brede kategorier, tydelige zoner
* **6-10 klasser**: Afbalanceret, godt til klassificering
* **11-20 klasser**: Glatte gradienter, kontinuerligt udseende
* **20+ klasser**: Næsten kontinuerlig, maksimal glathed

**Sådan justeres:**

1. I LUT-panelet skal du finde **farveprøvefelterne under gradientbjælken**
2. Juster antallet af klasser ved at tilføje med +-knappen
3. Fjern antallet af klasser ved at dobbeltklikke på en farveprøve
4. Gradienten opdateres **i realtid** på billedet

**Effekt på visualisering:**

* **Færre klasser** (3-5): Skaber tydelige zoner, forenklet klassificering, lettere at skelne mellem kategorier
* **Mellemstore klasser** (6-10): Afbalanceret tilgang, god til de fleste anvendelser
* **Flere klasser** (15-20): Glidende overgange, detaljerede variationer, fotografisk udseende

**Hvornår skal det bruges:**

* **Få klasser (3-5)**: Præsentationsslides, klassifikationskort, enkle rapporter
* **Mellemstore klasser (6-10)**: Generel analyse, afbalancerede detaljer, standardrapporter
* **Mange klasser (15-20)**: Videnskabelig analyse, detaljeret inspektion, output i publikationskvalitet

### Finjustering af værdiintervaller

**Værdiintervalkontrollerne** bestemmer, hvilke indeksværdier der skal knyttes til hvilke farver i din gradient:

**Intervalkontroller i LUT-panelet:**

* **Minimumsværdi**: Nedre grænse for farveskalaen
* **Maksimumsværdi**: Øvre grænse for farveskalaen
* **Mellemværdier**: Fordeles automatisk mellem min. og maks. (baseret på klassetælling)

#### Justering af min./maks. værdier

**Sådan justeres værdiintervaller:**

1. Find indtastningsfelterne **Min. værdi** og **Maks. værdi** i LUT-panelet
2. Klik på feltet **Min. værdi**
3. Indtast den ønskede minimumsværdi (f.eks. `0.2`)
4. Tryk på **Enter** eller klik uden for feltet
5. Gentag for feltet **Maks. værdi** (f.eks. `0.9`)
6. Visualiseringen **opdateres straks**

{% hint style=&quot;info&quot; %}
**Automatisk skalering**: Når du først anvender en LUT, indstiller Chloros automatisk min/maks til det faktiske dataområde i billedet. Du kan derefter indsnævre dette område for at fokusere på specifikke værdiområder af interesse.
{% endhint %}

**Eksempel på NDVI-intervaljusteringer:**

* **Fuldt interval**: `-1.0` til `1.0` (vis alle mulige værdier)
* **Fokuseret på vegetation**: `0.2` til `0.9` (ekskluder bar jord og vand)
* **Kun sund vegetation**: `0.5` til `0.9` (fremhæv kun frodige planter)
* **Stressdetektering**: `0.2` til `0.5` (fremhæv problemområder)
* **Brugerdefineret interval**: Juster ud fra dine observerede pixelværdier

**Hvorfor justere intervaller?**

* **Forøg kontrasten** i dit interesseområde
* **Ekskluder irrelevante værdier** (f.eks. vandområder, bar jord)
* **Standardiser visualiseringen** på tværs af flere billeder eller datoer
* **Fremhæv subtile forskelle** inden for et snævert værdiinterval

### Klipning af værdier uden for området

Når pixelværdier falder uden for dit definerede min./maks.-område, kan du kontrollere, hvordan de vises, ved hjælp af **klipningstilstande**.

#### **Tilgængelige klipningstilstande:**

#### 1. Minimum og maksimum

* Pixels **under minimum** → vises ved hjælp af den **første farve** i gradienten (f.eks. rød)
* Pixels **over maksimum** → vises ved hjælp af den **sidste farve** i gradienten (f.eks. grøn)
* **Anvendelsestilfælde**: Fremhæv ekstremer, vis hele dataintervallet med mættede farver ved grænserne
* **Eksempel**: NDVI-værdier under 0,2 vises alle røde, værdier over 0,9 vises alle grønne

#### 2. Gennemsigtig baggrund

* Pixels **uden for området** bliver **fuldstændig transparente**
* Kun pixels **inden for området** viser farvegradient
* **Anvendelsestilfælde**: GIS-overlay, isolering af specifikke værdiområder, fremhævning af kun områder af interesse
* **Eksempel**: Vis kun NDVI 0,4-0,7 i farve, alt andet transparent

{% hint style=&quot;warning&quot; %}
**Begrænsning af gennemsigtighed**: Gennemsigtige pixels vises som baggrundsfarven i visningsprogrammet. Når de eksporteres under behandlingen, bevares gennemsigtigheden i PNG-format, men ikke i JPG.
{% endhint %}

#### 3. Indeksbaggrund

* Pixels **uden for området** vises i **gråtoner** (viser rå indeksværdier)
* Pixels **inden for området** viser **farvegradient**
* **Anvendelsestilfælde**: Subtil fremhævning, bevar konteksten, mens områder af interesse fremhæves
* **Eksempel**: Fremhæv stresset vegetation med farve (NDVI 0,3-0,5), mens sunde områder vises i gråt

#### 4. Original baggrund

* Pixels **uden for området** viser det **originale multispektrale billede**
* Pixels **inden for området** viser **farvegradient**
* **Anvendelse**: Mest intuitivt – kombinerer naturlig billedkontekst med analytisk farveoverlay
* **Eksempel**: Se det faktiske udseende af marken/afgrøden med farvekodede stressområder overlejret

### Valg af den rigtige beskæringsfunktion

| Beskæringsfunktion              | Bedst til                                   | Visualiseringsstil          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum og maksimum**    | Fuld datavisning, videnskabelig analyse     | Alle pixels farvelagte           |
| **Gennemsigtig baggrund** | GIS-overlejringer, isolering af specifikke områder    | Farve inden for området, tomt udenfor |
| **Indeksbaggrund**       | Subtil fremhævning, bevarelse af datakontekst  | Farve inden for området, gråt udenfor  |
| **Original baggrund**    | Rapporter, præsentationer, intuitiv analyse | Farve på interval, foto udenfor |

### Oprettelse af brugerdefinerede LUT-farver

For fuld kontrol over din visualisering kan du oprette **brugerdefinerede farveovergange** ved at redigere individuelle farvestop.

**Sådan opretter du en brugerdefineret farveovergang:**

1. Find **farveovergangsvisningsbjælken** i LUT-panelet
2. Se efter **farveprøvefelterne** under farveovergangen
3. **Klik på et farvestop** for at vælge det
4. En **farvevælger** åbnes
5. Vælg en ny farve ved hjælp af:
   * **Farvehjul**: Visuel farvevalg
   * **RGB/HSV-skydeknapper**: Præcis farvekontrol
   * **Indtastning af hex-kode**: Præcis farvespecifikation (f.eks. `#FF0000` for rød)
6. Klik uden for farvevælgeren **for at anvende den nye farve**
7. Gradienten **opdateres straks** på billedet

**Tilføjelse eller fjernelse af farvestop:**

* **Tilføj et stop**: Klik på +-ikonet for at tilføje en ny farveprøve i slutningen
* **Fjern et stop**: Dobbeltklik på farvefeltet for at fjerne farveprøven

**Tilpasningsstrategier:**

* **Inverter gradient**: Vend farverejorden for at vende betydningen (f.eks. grøn=lav, rød=høj)
* **Brandfarver**: Tilpas din organisations farvepalet til rapporter
* **Farveblindvenlig**: Brug orange-blå eller lilla-gul kombinationer
* **Udskriftsoptimering**: Vælg farver, der fungerer i både farve- og gråtonet udskrivning
* **Multi-tærskel**: Brug forskellige farver ved specifikke værditærskler til klassificering

{% hint style=&quot;info&quot; %}
**Gemning af tilpassede gradienter**: Tilpassede gradienter kan gemmes og genbruges. Klik på gem-ikonet i LUT-panelet for at gemme dine tilpassede farveskemaer til fremtidig brug.
{% endhint %}

***

## Interaktivt workflow

### Opdateringer i realtid

Alle LUT-justeringer i sandkassen opdaterer billedet **øjeblikkeligt og interaktivt**:

* **Skift lag** → Billedet ændres med det samme
* **Vælg gradient** → Farver opdateres øjeblikkeligt
* **Juster værdiinterval** → Kontrast ændres i realtid
* **Skift klasser** → Gradientglathed opdateres øjeblikkeligt
* **Rediger klipning** → Baggrundsvisning ændres øjeblikkeligt
* **Rediger farver** → Brugerdefineret gradient anvendes øjeblikkeligt

**Ingen &quot;Anvend&quot;-knap nødvendig** – alle ændringer er live og interaktive!

{% hint style=&quot;success&quot; %}
**Live feedback**: Den øjeblikkelige visuelle feedback giver dig mulighed for hurtigt at eksperimentere med forskellige indstillinger, indtil du finder den optimale visualisering til dine analysebehov.
{% endhint %}

### Iterativ forfiningsworkflow

**Typisk LUT-optimeringsworkflow:**

1. **Vælg indekslag** (f.eks. RAW (Reflektans))
2. **Anvend indeks** – Vælg kamerafilter og indeksformel, træk farvede cirkler til det rette sted i indeksformlen
3. **Anvend LUT-gradient** – Start med Red-Yellow-Green-forudindstilling
4. **Kontroller pixelværdier** – Flyt markøren rundt, og noter værdiintervaller
5. **Juster min/maks** – Indsnævr for at fokusere på vegetation (f.eks. 0,2 til 0,9)
6. **Vælg beskæring** – Prøv &quot;Original baggrund&quot; for kontekst
7. **Finjuster farver** – Tilpas gradienten, hvis det er nødvendigt for at fremhæve noget specifikt
8. **Færdiggør indstillinger** - Dokumenter indstillingerne og kopier dem til projektindstillingerne til eksportbehandling

### Inspektion af pixelværdier

Det er afgørende at forstå de faktiske pixelværdier for at kunne indstille effektive LUT-intervaller:

**Sådan inspiceres værdier:**

1. Pixelværdier vises, når billedet har enten indeks eller både indeks og LUT **markeret**.
2. **Flyt markøren** over forskellige områder af billedet
3. **Observer pixelværdierne**, der vises i legenden, når du holder markøren over dem
4. Zoom ind for at se de enkelte pixels fremhævet med en flydende værdi
5. **Noter** værdiintervaller for forskellige funktioner:
   * **Sund vegetation**: f.eks. NDVI 0,55-0,85
   * **Stresset vegetation**: f.eks. NDVI 0,30-0,50
   * **Bare jord**: f.eks. NDVI 0,05-0,25
   * **Vand** (hvis til stede): f.eks. NDVI -0,05 til 0,10

**Brug af pixelværdier til at indstille LUT-intervaller:**

Efter at have inspiceret pixelværdierne skal du justere din LUT min/max i overensstemmelse hermed:

**Eksempel på scenario:**

* **Observation**: Jordværdier = 0,05-0,25, Stresset = 0,25-0,50, Sund = 0,50-0,85
* **Mål**: Visualiser kun plantehelbred (ekskluder jord)
* **LUT-indstillinger**: Min = `0.25`, Max = `0.85`
* **Klipning**: &quot;Original baggrund&quot; for at se jorden i naturlige farver
* **Resultat**: Farvegradient gælder kun for vegetation, jorden vises som det originale billede

{% hint style=&quot;info&quot; %}
**Dynamisk rækkevidde**: Forskellige afgrøder, årstider og vækststadier vil have forskellige værdiintervaller. Kontroller altid pixelværdierne i dit specifikke datasæt, inden du indstiller LUT-intervaller.
{% endhint %}

***

## Brugerdefinerede indekser (Chloros+)

### Oprettelse af brugerdefinerede indeksformler

{% hint style=&quot;info&quot; %}
**Hvor skal det oprettes**: Brugerdefinerede indekser kan konfigureres i **Projektindstillinger** før behandling samt i sidepanelet i Image Viewer-sandkassen.
{% endhint %}

**Sådan oprettes et brugerdefineret indeks:**

1. **Åbn Projektindstillinger** (inden behandling) eller billedvisningssandkassens sidebjælke.
2. Naviger til **dropdown-menuen Indeksformel**.
3. Find indstillingen **&quot;Brugerdefineret&quot;** (du skal være logget ind med Chloros+-licens).
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

{% hint style=&quot;warning&quot; %}
**Formelvalidering**: Sørg for, at din formel bruger bånd, der er tilgængelige i dit kamera. For eksempel er RedEdge kun tilgængelig på kameraer med et RedEdge-filter.
{% endhint %}

***

## Næste trin

Nu hvor du forstår indeks/LUT-sandkassen:

* **Anvend til behandling**: Brug de fundne indstillinger i [Projektindstillinger](../project-settings/project-settings.md)
* **Batch-behandling**: Anvend optimerede indekser på hele datasæt
* **Få mere at vide**: Læs [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md)

Relateret dokumentation:

* [**Billedlag**](image-layers.md) - Lagstyring og visualisering
* [**Åbning af et billede i fuld skærm**](opening-an-image-full-screen.md) - Grundlæggende om billedvisning
* [**Behandling af billeder (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Fuld behandlingsworkflow

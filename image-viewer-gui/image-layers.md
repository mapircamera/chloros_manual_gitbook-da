# Billedlag

I rullemenuen Billedlag i Chloros Image Viewer kan du hurtigt skifte mellem forskellige versioner af det samme billede – fra de originale optagelser til behandlede reflektansudskrifter og beregnede indeksbilleder.

## Hvad er billedlag?

I Chloros refererer **lag** til de forskellige billedoutput, der er tilgængelige for et enkelt kildebillede. Når du behandler billeder, opretter Chloros flere versioner:

* **Originalbilleder** (JPG- og RAW-filer fra dit kamera)
* **Reflektanskalibrerede** output (hvis reflektanskalibrering var aktiveret)
* **Målbilleder** (hvis billedet indeholder kalibreringsmål)
* **Indeksbilleder** (NDVI, NDRE, GNDVI osv., hvis indekser var konfigureret)

Med **Layer Selector-dropdownmenuen** øverst til højre i billedviseren kan du skifte mellem disse versioner uden at forlade viseren.

***

## Tilgængelige lagtyper

### JPG

* Det originale JPG-forhåndsvisningsbillede fra dit kamera
* Altid tilgængeligt for alle billeder
* Ubehandlet, som det er taget af kameraet
* Hurtigst at indlæse og vise

**Hvornår skal det vises:**

* Hurtig forhåndsvisning af originaloptagelsen
* Kontrol af billedkomposition og indramning
* Verificering af optagelseskvalitet før behandling

### RAW (original)

* De originale RAW-sensordata fra dit kamera
* Debayered uden efterbehandling
* Højere bitdybde end JPG (typisk 12-bit eller 14-bit sensordata)

**Hvornår skal det vises:**

* Inspektion af den originale sensordatakvalitet
* Kontrol af sensorproblemer eller artefakter
* Sammenligning af resultater før/efter behandling

### RAW (mål)

* Vises kun for billeder, der er identificeret som indeholdende kalibreringsmål
* Viser det originale RAW-billede med detekteret mål
* Bruges til at kontrollere, at måldetektering var vellykket

**Hvornår skal det vises:**

* Bekræftelse af, at kalibreringsmålene blev detekteret korrekt
* Kontrol af målbilledkvalitet
* Fejlfinding af kalibreringsproblemer

{% hint style=&quot;info&quot; %}
**Målelag**: Dette lag vises kun i rullemenuen for billeder, der indeholder kalibreringsmål. Almindelige optagelsesbilleder har ikke denne mulighed.
{% endhint %}

### RAW (reflektans)

* Det kalibrerede reflektansudgangsbillede
* Vignettekorrigeret (hvis aktiveret i behandlingen)
* Reflektans kalibreret ved hjælp af måldata (hvis aktiveret)
* Multibånd TIFF med alle kamerakanaler
* Pixelværdier repræsenterer procentuel reflektans (når procenttilstand bruges)
* Klar til manipulation med [Index/LUT Sandbox](index-lut-sandbox.md)

**Hvornår skal det vises:**

* Inspektion af kalibrerede resultater
* Verificering af kalibreringskvalitet
* Kontrol af pixelværdier for videnskabelig nøjagtighed
* Sammenligning med originalen for at se kalibreringseffekter

{% hint style=&quot;success&quot; %}
**Anbefalet**: Brug RAW (Reflektans)-lag, når du kontrollerer pixelværdier til videnskabelige målinger og analyser.
{% endhint %}

### RAW (NDVI Index)... og lignende

* Beregnet vegetationsindeksbillede (NDVI i dette eksempel)
* Indeksnavnet ændres afhængigt af, hvilket indeks der blev konfigureret under behandlingen
* Eksempler: RAW (NDVI-indeks), RAW (NDRE-indeks), RAW (GNDVI-indeks) osv.
* Enkeltbånds gråtonebillede, der viser indeksberegningsresultater
* Der vises et lag for hvert indeks, der er konfigureret i projektindstillingerne

**Mulige indeksnavne:**

* RAW (NDVI-indeks)
* RAW (NDRE-indeks)
* RAW (GNDVI-indeks)
* RAW (OSAVI-indeks)
* RAW (EVI-indeks)
* RAW (SAVI-indeks)
* Og mange flere... (se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md))

**Hvornår skal det vises:**

* Undersøgelse af indeksberegningsresultater
* Kontrol af indeksværdiintervaller
* Identificering af områder af interesse
* Verificering af indeksbilleder før brug i GIS eller analyse

***

## Brug af lagvælgeren

### Åbning af rullemenuen

1. Åbn et billede i fuldskærmstilstand (klik på en vilkårlig miniature i billedviseren)
2. Find **lagrullemenuen** i øverste højre hjørne af viseren
3. Dropdown-menuen viser det aktuelt valgte lag (f.eks. &quot;JPG&quot;)
4. Klik på dropdown-menuen for at se alle tilgængelige lag

### Skift af lag

1. Klik på lag-dropdown-menuen for at åbne listen
2. Alle tilgængelige lag for det aktuelle billede vises
3. Klik på et hvilket som helst lagnavn for at skifte til den pågældende version
4. Billedet opdateres straks for at vise det valgte lag

**Hurtigt skift:**

* Dropdown-menuen husker dit sidste valg.
* Når du navigerer til det næste billede, forsøger Chloros at vise den samme lagtype.
* Hvis det lag ikke findes på det næste billede, vises JPG som standard.

### Lagtilgængelighed

Ikke alle lag er tilgængelige for alle billeder:

**Altid tilgængeligt:**

* ✅ JPG (alle billeder har en JPG-forhåndsvisning)

**Betinget tilgængelige:**

* ⚠️ RAW (Original) - Kun hvis billedet er taget i RAW- eller RAW+JPG-tilstand
* ⚠️ RAW (Mål) - Kun hvis billedet indeholder registrerede kalibreringsmål
* ⚠️ RAW (Reflektans) - Kun efter behandling med reflektanskalibrering aktiveret
* ⚠️ RAW (\[Indeks] Indeks) - Kun efter behandling med konfigurerede indekser

***

## Lagpersistens

### Navigation mellem billeder

Når du navigerer til et andet billede (ved hjælp af piletasterne eller ved at klikke på miniaturer):

**Lagepræference bevares:**

* Hvis du viser &quot;RAW (Reflektans)&quot;, viser det næste billede &quot;RAW (Reflektans)&quot; (hvis tilgængeligt)
* Hvis du viser &quot;RAW (NDVI Indeks)&quot;, viser det næste billede &quot;RAW (NDVI Indeks)&quot; (hvis tilgængeligt)
* Hvis det samme lag ikke findes, vises JPG som standard

**Eksempel på arbejdsgang:**

1. Åbn billede 1, skift til RAW (NDVI Index)
2. Tryk på → for at se billede 2
3. Billede 2 viser automatisk RAW (NDVI Index)-laget
4. Fortsæt med at navigere – alle billeder viser NDVI-laget
5. Meget effektivt til gennemgang af indeksresultater på tværs af mange billeder

***

## Almindelige arbejdsgange

### Arbejdsgang 1: Før/efter-sammenligning

**Mål**: Sammenlign originalbillede med kalibreret billede

1. Åbn det behandlede billede i billedviseren
2. Vælg **RAW (Original)** fra rullemenuen
3. Bemærk vignettering og ukalibrerede værdier
4. Skift til **RAW (Reflektans)** fra rullemenuen
5. Sammenlign – vignettering fjernet, værdier kalibreret

### Arbejdsgang 2: Gennemgang af indeks

**Mål**: Hurtig gennemgang af NDVI-resultater på tværs af datasættet

1. Åbn det første behandlede billede
2. Vælg **RAW (NDVI-indeks)** fra rullemenuen
3. Brug → piletasten til at navigere til næste billede
4. NDVI-laget forbliver automatisk
5. Fortsæt gennem alle billeder og kontroller NDVI-mønstre
6. Skift til **RAW (NDRE Index)** for at sammenligne

### Arbejdsgang 3: Målverifikation

**Mål**: Verificer, at alle målbilleder er blevet detekteret korrekt

1. Naviger til et målbillede
2. Vælg **RAW (Target)** fra rullemenuen
3. Verificer, at kalibreringsmålene er tydeligt synlige og detekteret
4. Naviger til næste målbillede
5. Gentag verifikationen for alle mål

### Arbejdsgang 4: Inspektion af pixelværdi

**Mål**: Kontroller reflektansværdier for videnskabelig nøjagtighed

1. Åbn det behandlede billede
2. Vælg **RAW (Reflektans)**-lag
3. Aktiver **Pixelprocent**-tilstand (knap i værktøjslinjen øverst til højre)
4. Flyt markøren over vegetationsområder
5. Kontroller, at pixelværdierne ligger inden for de forventede intervaller (30-70 % for NIR, 5-15 % for Red)
6. Kontroller, at jord- og vandområder har passende værdier

***

## Forståelse af pixelværdier efter lag

Forskellige lag viser forskellige pixelværdiintervaller:

### JPG-lag

* **Interval**: 0-255 (8-bit)
* **Betydning**: Visningsværdier, gammakorrigeret
* **Anvendelse**: Kun visuel inspektion, ikke til videnskabelig måling

### RAW (original)

* **Interval**: 0-65535 (16-bit)
* **Betydning**: Rå digitale sensortal
* **Anvendelse**: Kontrol af sensorens ydeevne, ikke kalibreret

### RAW (reflektans)

* **Interval**: 0-65.535 (16-bit TIFF) eller 0,0-1,0 (32-bit procent)
* **Betydning**: Kalibreret procentuel reflektans
* **Anvendelse**: Videnskabelige målinger og analyser

**For 16-bit TIFF:** Divider med 65.535 for at få procentuel reflektans **For 32-bit procent:** Værdierne repræsenterer direkte procent (0,5 = 50 % reflektans)

### RAW (indeksbilleder)

* **Område**: Varierer efter indeks (typisk -1,0 til +1,0 for normaliserede indekser)
* **Betydning**: Indeksberegningsresultat
* **Eksempler**:
  * NDVI: -1 til +1 (vegetation typisk 0,4 til 0,9)
  * NDRE: -1 til +1 (stressdetektering)
  * EVI: 0 til 1 (forbedret vegetation)

***

## Tips og bedste praksis

### Effektiv skift mellem lag

* **Kendskab til tastaturgenveje**: Der er ingen tastaturgenveje til lag, men navigationspilene (←/→) fungerer på tværs af alle lag
* **Konsistente arbejdsgange**: Vælg et lag (f.eks. NDVI) og gennemgå hele datasættet, før du skifter til et andet
* **Hurtige sammenligninger**: Skift mellem Original og Reflektans for at kontrollere behandlingskvaliteten

### Overvejelser vedrørende ydeevne

* **JPG indlæses hurtigst**: Brug til hurtig navigation gennem mange billeder.
* **RAW-lag indlæses langsommere**: Højere opløsning og bitdybde.
* **Indekslag**: Samme hastighed som reflektanslag.
* **Første indlæsning er langsomst**: Efterfølgende visninger af samme lag caches og er hurtigere.

### Kvalitetsverifikation

* **Kontroller altid RAW (Original)**: Kontroller kildedatas kvalitet, før du stoler på de behandlede resultater
* **Sammenlign lag**: Brug lagskift til at kontrollere, at behandlingen er foregået korrekt
* **Kontroller indeksintervaller**: Brug Pixel Percent-tilstand med indekslag for at kontrollere, at værdierne er rimelige

***

## Fejlfinding

### Lag ikke tilgængeligt

**Problem**: Det forventede lag vises ikke i rullemenuen

**Mulige årsager:**

* Billedet blev ikke behandlet (kun JPG og RAW (Original) tilgængeligt)
* Reflektanskalibrering blev deaktiveret under behandlingen
* Specifikt indeks blev ikke konfigureret i projektindstillingerne
* Billedet er et mål-only-billede (ingen indekser genereret for mål)

**Løsninger:**

1. Kontroller, at billedet er blevet behandlet (tjek outputmappen for behandlede filer)
2. Tjek projektindstillingerne for at bekræfte, at indekserne er konfigureret
3. Behandl igen med de ønskede indekser aktiveret

### Forkert lag vises

**Problem**: Billedet åbnes i et uventet lag

**Årsag**: Lagpræference fra forrige billede er blevet overført, men det lag findes ikke på det aktuelle billede

**Løsning**: Chloros falder automatisk tilbage til JPG, når det foretrukne lag ikke er tilgængeligt – dette er normal adfærd.

### Kan ikke se kalibreringsmål

**Problem**: RAW-laget (mål) viser ikke måldetektering.

**Mulige årsager:**

* Mål blev ikke detekteret under behandlingen
* Billedet indeholder faktisk ikke mål
* Indstillingerne for måldetektering er for strenge

**Løsninger:**

1. Kontroller fejlfindingsloggen for meddelelser om &quot;Mål fundet&quot;
2. Kontroller, at billedet faktisk indeholder synlige kalibreringsmål
3. Juster indstillingerne for måldetektering i projektindstillingerne
4. Se [Valg af målbilleder](../processing-images-gui/choosing-target-images.md)

***

## Relaterede funktioner

### Billedvisningsværktøjer

Når du viser et lag, kan du bruge:

* **Zoomkontroller**: Forstør for at se detaljer
* **Panorering**: Klik og træk for at flytte rundt på det forstørrede billede
* **Pixelværdiinspektion**: Se værdier ved cursorens placering
* **Navigationspile**: Flyt mellem billeder, mens du bevarer laget
* **Pixelprocenttilstand**: Skift mellem DN- og procentvisning

Se [Åbning af et billede i fuld skærm](opening-an-image-full-screen.md) for komplet dokumentation om billedviseren.

### Indeks/LUT-sandkasse

Til interaktiv indeksafprøvning og visualisering:

* **Indeksberegning i realtid**: Test forskellige indeksformler
* **LUT-farvekortlægning**: Anvend farvegradienter på gråtonede indekser
* **Eksporter visualiseringer**: Gem farvede indeksbilleder

Se [Index/LUT Sandbox](index-lut-sandbox.md) for detaljer.

***

## Næste trin

Nu hvor du forstår billedlag:

* [**Åbning af et billede i fuld skærm**](opening-an-image-full-screen.md) - Komplet guide til billedviseren
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Interaktiv indeksvisualisering
* [**Multispektrale indeksformler**](../project-settings/multispectral-index-formulas.md) – Tilgængelige indeksreferencer
* [**Afslutning af behandlingen**](../processing-images-gui/finishing-the-processing.md) – Forståelse af behandlede resultater

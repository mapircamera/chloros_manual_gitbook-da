# Index/LUT-sandkasse

Index/LUT-sandkassen er det interaktive arbejdsområde i sidepanelet i Chloros Image Viewer. Du vælger en formel, knytter kameraets kanaler til den, farvelægger den med en farveovergang og justerer værdiområdet — og billedet opdateres i realtid, mens du arbejder. Siden version 1.2.0 kan du også **gemme det, du har lavet**, enten for et enkelt billede eller for hele projektet, uden at skulle genbehandle det.

## Hvad Sandboxen bruges til

| Index/LUT Sandbox (interaktiv)        | Projektbehandling (batch)       |
| -------------------------------------- | -------------------------------- |
| Ét billede ad gangen, øjeblikkelig feedback  | Hele datasættet i én gennemkørsel     |
| Eksperimentelt og iterativt             | Forudkonfigurerede indstillinger          |
| Renderer live; gemmer kun, når du beder om det  | Skriver altid produktfiler      |
| Perfekt til at finde de rigtige indstillinger | Bedst, når indstillingerne er endelige |

{% hint style="success" %}
**Den sædvanlige arbejdsgang**: finjuster i Sandbox, indtil visualiseringen ser ud, som du ønsker, og eksporter derefter enten direkte fra Sandbox, eller kopier de samme indeks- og LUT-indstillinger til [Projektindstillinger](../project-settings/project-settings.md), så den næste behandlingskørsel indarbejder dem i hvert billede.
{% endhint %}

***

## Åbning af Sandbox

1. Klik på et billede i gitteret — det åbnes i fuld skærm i fanen **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Klik på ikonet **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> for at skubbe venstre sidepanel frem, hvis det ikke allerede er åbent
3. Vælg et multibåndslag fra lag-rullemenuen øverst til højre — **RAW (Reflektans)** er det sædvanlige valg, da indeksværdier beregnet på baggrund af kalibreret reflektans er sammenlignelige på tværs af billeder

Sidepanelet viser, fra top til bund:

* billedets navn og dets kameramodel
* knappen **Eksporter/Gem billede(r)** — vises, når *Index* eller *LUT* er markeret
* afkrydsningsfelterne **Index**og**LUT**
* konfigurationspanelet for indeks
* panelet **Cursorværdier** med aflæsning, histogram og GSD-kontrol

{% hint style="warning" %}
**Ikke tilgængeligt for monokameraer.** På et LATTICE M3M-billede med et enkelt bånd er begge afkrydsningsfelter deaktiveret, med værktøjstipet _&quot;Ikke tilgængeligt for mono (M3M)-sensorer&quot;_ — et multibåndsindeks er udefineret på et enkelt bånd. For at beregne indekser fra M3M-kameraer skal du kombinere to eller flere til en justeret multibåndsstak og bruge LATTICE-indeksmotoren.
{% endhint %}

***

## Anvendelse af et indeks

1. Markér afkrydsningsfeltet **Indeks** øverst i sidepanelet
2. Vælg dit kameras filter fra rullemenuen til venstre (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Vælg en indeksformel fra rullemenuen til højre — 27 indbyggede formler samt eventuelle brugerdefinerede formler, du har gemt
4. Formlen vises som matematisk udtryk nedenfor med en tom cirkel ved hvert båndfelt. **Træk en farvet kanalcirkel over på et felt** for at knytte den til det
5. Når alle pladser, som formlen bruger, er bundet, opdateres billedet og viser indeksværdier
6. Hold markøren over billedet for at aflæse værdier; panelet **Markørværdier** tilføjer en indeksrække med værdien under markøren

Dobbeltklik på en bundet plads for at rydde den. En ufuldstændig formel er en normal tilstand under trækningen, ikke en fejl — billedet opdateres ganske enkelt ikke, før formlen er færdig.

Kanalcirklerne er farvekodede: rød = Red, grøn = Green, blå = Blue, orange = Orange, cyan = Cyan, lilla = NIR, magenta = RE. De samme farver bruges til kanalprikkerne og histogramkurverne i panelet »Cursor Values«.

### Eksempel på NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

For den komplette formeloversigt — alle tre forudindstillede lister og hvilke navne der fungerer hvor — se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md).

### Med Indeks markeret, men uden LUT

Billedet tegnes i **gråtoner**, strakt mellem de to tærskelværdier. Dette er bevidst: indeksbilledet er skalardata, og gråtoner er den ægte gengivelse af det. Tilføj en LUT, når du ønsker farve.***

## Arbejde med LUT&#x27;er (opslagstabeller)

En **opslagstabel** knytter indeksværdier til farver: indtast NDVI 0,65, og der vises en bestemt grøn farve. Det ændrer ikke dataene — det ændrer blot, hvordan du fortolker dem.

### Tilføjelse af en LUT

1. Klik på knappen **&quot;+ Tilføj LUT&quot;** under formlen <img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line">
2. Vælg en farvegradient
3. Indstil minimums- og maksimumsværdier for beskæring
4. Vælg en beskæringsmodus
5. Markér afkrydsningsfeltet **LUT** i sidepanelet for at gengive den

Afkrydsningsfeltet **LUT** forbliver deaktiveret, indtil en LUT rent faktisk er blevet konfigureret på indekset.

### Valg af farvegradient

Hold markøren over **gradientbjælken**for at åbne listen over forudindstillinger — Chloros leveres med**syv** forudindstillede gradienter:

| # | Gradient                            | Form                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Gul → Green (**standard**)  | Divergerende — svarer til den sædvanlige opfattelse af vegetation, hvor grønt = sundt |
| 2 | Lilla → Gul → Green             | Divergerende, med en tydelig lav ende                                  |
| 3 | Brun → Hvid → Blue                | Divergerende omkring et lyst midtpunkt                                   |
| 4 | Sort → Lilla → Lyserød → Lysegul | Sekventiel, fra mørk til lys                                           |
| 5 | Red → Gul → Blue                 | Afvigende omkring et lyst midtpunkt                                   |
| 6 | Lilla → Blue → Green → Gul      | Sekventielt, fra mørkt til lyst                                           |
| 7 | Orange → Hvid → Lilla             | Divergerende omkring et lyst midtpunkt                                   |

En **divergerende**farveovergang placerer en neutral farve i midten af dit vindue, hvilket fungerer godt, når midtpunktet har en betydning (en tærskel, en basisdato). En**sekventiel** farveovergang forløber monotont fra mørkt til lyst, hvilket fungerer godt for en mængde, der kun har &quot;mere&quot; og &quot;mindre&quot;.

Hver forudindstilling har syv farvestop. Klik på en forudindstilling, og billedet opdateres straks (når LUT-feltet er markeret).

### Redigering af farvestoppene

Under gradientbjælken er der en række farveprøver, én pr. stop:

* **Skift en farve**: Klik på en farveprøve for at åbne farvevælgeren (farvehjul, RGB/HSV-skydere eller en hex-kode såsom `#FF0000`)
* **Tilføj et stop**: Klik på**+**-knappen for enden af rækken — et hvidt stop tilføjes
* **Fjern et stop**:**Dobbeltklik** på farveprøven
* **Gem en redigeret farveovergang**: Klik på gem-ikonet ved siden af farveovergangsstangen for at tilføje din redigerede farveovergang til listen over forudindstillinger, så du kan vælge den igen

Den farveovergang, du har konfigureret på et indeks, gemmes sammen med det pågældende indeks i projektets indstillinger, så den bevares, selvom projektet lukkes og åbnes igen.

**Færre stop**skaber tydelige zoner, der fremstår som en klassificering;**flere stop** skaber glidende, næsten fotografiske overgange. Tre til fem stop egner sig til præsentationsdias og klassifikationskort; seks til ti egner sig til generel analyse; femten eller flere egner sig til detaljeret inspektion og publikationsfigurer.

### Indstilling af værdiintervallet

Tærskelregulatoren er en **skyder med to håndtag**, der går fra −1 til +1, med et redigerbart tekstfelt i hver ende til indtastning af nøjagtige værdier samt en**AUTO**-knap.

* Træk i et af håndtagene, eller indtast et tal i feltet og tryk på Enter
* **AUTO**indstiller området til**

2. og 98. percentil** af billedets gyldige indeksværdier — et godt udgangspunkt, der ignorerer afvigende værdier. Chloros afrunder resultatet adaptivt til 4 decimaler for et meget snævert interval, 3 for et snævert interval og 2 i alle andre tilfælde
* Enhver manuel justering har forrang frem for AUTO, indtil du trykker på AUTO igen

Eksempel på NDVI-vinduer:

| Mål                                    | Min  | Maks |
| --------------------------------------- | ---- | --- |
| Vis alt                                | −1,0 | 1,0 |
| Kun vegetation, ekskluder jord og vand | 0,2  | 0,9 |
| Kun sund vegetation                 | 0,5  | 0,9 |
| Fremhæv stress                        | 0,2  | 0,5 |

Ved at indsnævre vinduet øges kontrasten inden for dit interesseområde, og alt andet skubbes uden for området — hvor **klippemodus** bestemmer, hvad der sker med det.***

## Klippemodus

Når en pixels indeksværdi falder uden for min./maks.-vinduet, bestemmer klippemodus, hvordan den tegnes.

| Dropdown-etiket                  | Lagret værdi      | Pixler uden for området tegnes som                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minimum &amp; maksimum** (standard) | `clip`            | Den nærmeste endefarve i gradienten — værdier under minimumet får den første farve, værdier over maksimumet får den sidste |
| **Gennemsigtig baggrund**      | `transparent`     | Fuldt gennemsigtig (ægte alfa)                                                                                                  |
| **Indeksbaggrund**| `indexColor`      | Gråtoner, strækket over billedets**fulde** indeksområde, så strukturer uden for området stadig er synlige i gråt                |
| **Originalbaggrund**         | `backgroundColor` | Selve det underliggende billede, så farveoverlejringen ligger oven på den virkelige scene                                                |

| Tilstand                       | Bedst egnet til                               | Udseende                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Minimum &amp; maksimum**      | Fuld datavisning, videnskabelig analyse | Hver pixel er farvet                      |
| **Gennemsigtig baggrund** | GIS-overlejringer, isolering af et værdiinterval   | Farve inden for vinduet, intet udenfor |
| **Indeksbaggrund**       | Fremhævning med bevarelse af datakonteksten    | Farve indeni, gråt udenfor               |
| **Originalbaggrund**    | Rapporter og præsentationer              | Farve indeni, fotografi udenfor         |

{% hint style="info" %}
**Pixler uden data er altid gennemsigtige, uanset tilstand.** En pixel, hvis indeks ikke er endeligt (en division med 0/0) eller er nøjagtigt −1,0 eller +1,0 (mætningsmarkører, hvor det ene bånd viser nul, mens det andet ikke gør), behandles som ingen data i stedet for som en ekstremværdi. Dette holder udbrændte højlys og mørke skygger ude af din farveskala i stedet for at gengive dem som den mest ekstreme måleværdi i billedet. Den samme regel definerer, hvilke pixels der indgår i AUTO-tærsklerne og indeks-histogrammet, så alle tre stemmer overens.
{% endhint %}

Gennemsigtighed bevares, når eksporten skrives som PNG. Den kan ikke gengives i JPG.

***

## Aflæsning af værdier, mens du finjusterer

Panelet **Cursorværdier** under konfigurationspanelet fungerer som måleinstrument for Sandbox:

* Flyt markøren over billedet, og aflæs kildeværdierne pr. kanal samt indeksværdien i sin egen række
* Aktiver **INDEX**-knappen over histogrammet for at se fordelingen af indeksværdier i billedet, hvor dine to klip-tærskler er markeret med orange stiplede linjer og markørens værdi med en hvid linje — dette er den hurtigste måde at vælge et vindue, der rent faktisk indeholder dine data
* Aktiver **CURSOR** for at se markørlinjer ved værdierne under markøren
* Zoom ind over 60× (mindre, hvis der er indstillet en GSD-blokstørrelse) for at fremhæve de enkelte viste pixels med en flydende værdi

En praktisk fremgangsmåde:

1. Notér værdierne over sund vegetation, stresset vegetation, bar jord og vand
2. Se, hvor disse klynger befinder sig på indekshistogrammet
3. Indstil min./maks., så den klynge, du er interesseret i, rammes ind
4. Vælg en beskæringsindstilling — _Original Background_ holder scenen synlig omkring den

***

## Eksport fra Sandbox

Alt ovenstående er en live-forhåndsvisning, indtil du gemmer det. Knappen **Eksporter/Gem billede(r)** øverst i sidepanelet åbner et vindue, der glider ind over sidepanelet (i stedet for at dække billedet, så du stadig kan se, hvad du træffer beslutning om).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Indstillinger

| Indstilling                          | Effekt                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Anvend på det aktuelle billede**      | Gemmer præcis det viste billede med disse indstillinger                                                                                                |
| **Anvend på alle projektbilleder** | Kører den samme konfiguration igen på hvert billede i projektet. Billeder uden de bånd, som dette indeks kræver, springes over og behandles ikke som fejl |
| **Indeks/LUT-gradientbjælke**      | Skriver også et separat billed med forklaring pr. eksport, hvor værdiområdet er mærket                                                                     |
| **Indeks-histogram**             | Skriver også et separat histogrambillede pr. eksport, der viser dataenes min./maks. værdier og klip-tærsklerne                                               |

Hvis **GSD-blokstørrelsen** på billedfanen er over 1, angiver ruden dette, før du bekræfter: eksporten gemmer det, du ser, inklusive blokgennemsnit. Indstil GSD-kontrollen tilbage til 1 først, hvis du ønsker fuld opløsning.

### Hvor filerne gemmes

Hvert klik på **Eksporter**opretter en**ny mappe, der aldrig genbruges**:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Eksempler: `Sandbox_Exports/NDVI_LUT_001/`, derefter `Sandbox_Exports/NDVI_LUT_002/` ved næste kørsel. Nummereringen genereres ved at scanne, hvad der allerede findes på disken, så den overlever genstarter og mapper, du sletter manuelt. Intet overskrives nogensinde — hele pointen med Sandbox er at sammenligne et forsøg med det forrige.

Indholdet af mappen for hvert billede:

| Fil                                                   | Indhold                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | Det renderede billede, pixel for pixel, som det blev vist i visningsprogrammet |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | Sidecar-filen med farveovergangsstangen, hvis anmodet                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | Sidecar-filen med indekshistogrammet, hvis anmodet                  |

De to sidecars skrives altid i **fuld opløsning**, selv når hovedbilledet er blokgennemsnitligt: en blokstørrelse svarer til skærmopløsningen, og begge sidecars læser de reelle indeksværdier pr. pixel. De udskriver også mere end versionerne på skærmen — begge angiver strækningsvinduet _og_ de faktiske minimums- og maksimumsværdier for dataene, så en gemt legende stadig kan læses måneder senere, selvom projektet ikke er åbent.

### Forløb og resultater

En eksport af hele projektet tager få minutter, så processen rapporterer tilbage via en live statuskanal i stedet for at blokere:

* En statusbjælke viser `current / total` og den fil, der skrives
* Når den er færdig, viser vinduet, hvor mange billeder der blev eksporteret, hvor mange der blev sprunget over, samt stien til outputmappen
* Oversprungne billeder vises med årsagen (op til fem vises, derefter en linje med &quot;+N flere&quot;). Den sædvanlige årsag er et lag, der ikke har de kanaler, som dette indeks kræver
* Hvis **ingen** billeder i projektet kan bruge indekset, rapporterer kørslen en fejl i stedet for at efterlade en tom mappe

Der kan kun køre én sandkasseeksport ad gangen. Hvis man forsøger at starte en anden, mens en allerede er i gang, afvises dette med en tydelig besked i stedet for at lade to kørsler kæmpe om den samme projektfil.

### Gitteret viser kørslen

Hver afsluttet kørsel vises som sin egen knap i værktøjslinjen [billedgitter](image-grid.md) med betegnelsen `<IndexName> <Index|LUT> <NNN>`. Sådan sammenligner du kørsler: eksporter to gange med forskellige gradienter eller tærskler, og skift derefter mellem de to knapper i gitteret.

***

## Brugerdefinerede indeksformler (Chloros+)

{% hint style="info" %}
**Hvor skal de oprettes**: i sidepanelet i Sandbox eller i**Projektindstillinger** inden behandlingen. Begge skriver til den samme liste på projektniveau.
{% endhint %}

1. Åbn regnemaskinen til brugerdefinerede formler fra rullemenuen med indeksformler (kræver, at du er logget ind med et gyldigt Chloros+-abonnement)
2. Indtast formlen ved hjælp af **bånd-slot-symbolerne** `x`, `y`, `z`, `a`, `b`, `c` — ikke båndnavne
3. Tilgængelige operatorer: `+`, `-`, `*`, `/`, `^` og `()` til gruppering
4. Tilgængelige funktioner: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Navngiv og gem den — den vises nederst i formel-rullemenuen, og du tildeler dens slots ved at trække kanalkredse, præcis som med en indbygget forudindstilling

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Brugerdefinerede formler findes kun i GUI&#x27;en.** Indstillingen CLI/SDK `--indices` udvider de 22 indbyggede forudindstillingsnavne og springer automatisk alt andet over, herunder dine brugerdefinerede formler. For at behandle en brugerdefineret formel i batch skal du konfigurere den i Projektindstillinger og køre behandlingen, eller bruge Sandboxens eksportfunktion »Anvend på alle projektbilleder«.
{% endhint %}

***

## Fejlfinding

### &quot;Dette lag har ikke de kanaler, som dette indeks kræver&quot;

Formlen læser en kanalposition, som det aktuelle lag ikke har — for eksempel et indeks med tre slots på en fil med én eller to kanaler. Skift til et multibåndslag (reflektans eller debayered), eller vælg et indeks, der passer til dit kameras filter.

### »Kunne ikke oprette forbindelse til billedbehandlings-backend&#x27;en«

Backend&#x27;en svarer ikke. Tjek fanen Log; hvis backend&#x27;en genstarter, genopretter Sandbox sig selv, så snart den er tilbage.

### Billedet ændrede sig ikke, da jeg trak en cirkel

Formlen er endnu ikke færdig. En ufuldstændig formel behandles som en normal tilstand midt i et træk — der renderes intet, og der rapporteres ikke om nogen fejl. Udfyld alle felter, som formlen bruger.

### Hele billedet har én farve

Dit klipvindue ligger sandsynligvis langt uden for dataene. Tryk på **AUTO**for at justere det til 2.- eller 98.-percentilen, eller aktiver**INDEX**-histogrammet for at se, hvor dataene rent faktisk ligger.

### De eksporterede farver stemmer ikke overens med det, jeg så

Det burde de — eksportstien er en bevidst spejling af live-forhåndsvisningen, inklusive alfa i klip-tilstand, og blokgennemsnittet anvendes _efter_ farvelægningen, præcis som visningsprogrammet gør det. Hvis der er forskel, skal du kontrollere, at GSD-blokstørrelsen ikke er ændret mellem visning og eksport.

***

## Næste trin

* [**Billedlag**](image-layers.md) — hvilket lag der skal køres et indeks på, og hvad værdierne betyder
* [**Åbning af et billede i fuld skærm**](opening-an-image-full-screen.md) — detaljeret beskrivelse af markørvisning, histogram og GSD-kontrol
* [**Formler til multispektrale indekser**](../project-settings/multispectral-index-formulas.md) — alle forudindstillinger, på alle overflader
* [**Projektindstillinger**](../project-settings/project-settings.md) — fastlægning af de indstillinger, du har fundet, i et behandlingsforløb

---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Formler til multispektrale indekser

Nedenstående indeksformler anvender en kombination af gennemsnitlige transmissionsområder fra Survey3-filtre:

<table><thead><tr><th align="center">Survey3-filterfarve</th><th width="196.199951171875" align="center">Survey3-filternavn</th><th width="159.800048828125" align="center">Transmissionsområde (FWHM)</th><th align="center">Gennemsnitlig transmission</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re-RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Når disse formler anvendes, kan navnet slutte på &quot;\_1&quot; eller &quot;\_2&quot;, hvilket angiver, hvilket NIR-filter – enten NIR1 eller NIR2 – der er blevet brugt.

For LATTICE M3C-kameraer (Bayer-tredobbeltbåndspas) anvender den samme indeksmotor M3C-filterbåndene:

| M3C-filter | Bånd 1 (centrum/FWHM) | Bånd 2 (centrum/FWHM) | Bånd 3 (centrum/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

LATTICE M3M-kameraer er enkeltbåndskameraer (ét smalbåndsfilter pr. kamera), så der beregnes ikke multibåndsindekser for et enkelt M3M-billede. For at beregne indekser med M3M skal du kombinere to eller flere kameraer til en justeret multibåndsstak og bruge LATTICE-indeksmotoren (`chloros-cli lattice index` eller GUI&#x27;ens live-indeksberegner).

***

## Hvor hvert indeksnavn fungerer

I Chloros findes der **tre** indeksflader, og deres forudindstillede lister er ikke identiske. Brug dette afsnit til at kontrollere, om et navn vil fungere der, hvor du planlægger at bruge det.

| Hvor du befinder dig | Hvilken liste gælder | Antal |
| --- | --- | --- |
| Projektindstillinger → Indeks → Tilføj indeks (GUI) | Overflade 1 | 27 |
| Billedfremviser [Indeks/LUT-sandkasse](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Overflade 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Overflade 2 | 22 |
| SDK `process_folder(indices=[...])` | Overflade 2 | 22 |
| `chloros-cli lattice index --preset` | Overflade 3 | 22 (et andet 22) |
| Fanen &quot;Kameraer&quot; – live indeksberegner | Overflade 3 | 22 (et andet 22) |

Surface 1 og 2 arbejder med **ét billede ad gangen fra ét kamera**, ved hjælp af symbolpladserne `x`/`y`/`z`(/`a`), der er knyttet til det pågældende kameras filterkanaler. Overflade 3 arbejder med en**justeret multibåndsstak** — flere LATTICE-kameraer, der er samregistreret i én kube — og henviser til kanaler ved hjælp af navne med små bogstaver.

### 1. GUI-projektindstillinger / dropdown-menuen i billedviserens sandkasse — 27 formler

Dropdown-menuen viser dem i denne rækkefølge (det er indsættelsesrækkefølgen, ikke alfabetisk):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

I GUI&#x27;en trækker du dit kameras filterkanaler over på formelens båndpladser, så enhver formel kan bruges med enhver båndtildeling, som dit kamera understøtter. Brugerdefinerede formler, du har gemt, tilføjes under denne liste.

De **fem formler, der kun findes i GUI&#x27;en** — dem, som CLI-/SDK-listen `--indices` ikke accepterer — er implementeret som:

| Forudindstilling kun i GUI | Formel (som implementeret) | Pladser |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (fire slots) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Den tilsigtede tilknytning for hver enkelt er angivet i et eget afsnit længere nede på denne side (for eksempel forventer GARI, at x=Green, y=NIR, z=Blue, a=Red). GARI er den eneste formel i Chloros, der bruger en fjerde plads.

### 2. CLI / SDK `--indices` navneudvidelse — 22 forudindstillinger

Indstillingen `chloros-cli process --indices` (og SDK-parameteren `indices`) accepterer følgende forudindstillingsnavne:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Ukendte indeksnavne springes over uden advarsel.** Et navn uden for denne liste (herunder de fem formler, der kun findes i GUI’en: `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` samt eventuelle brugerdefinerede formler, du har gemt i GUI&#x27;en) udelades med blot en logmeddelelse — kørslen fortsætter uden det pågældende indeks, og selve kørslen rapporteres stadig som vellykket. Meddelelsen vises som:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Navne matches uden skelnen mellem store og små bogstaver efter fjernelse af mellemrum, så `ndvi`, `NDVI` og ` NDVI ` er den samme forudindstilling. En forudindstilling springes også over, hvis den kræver et bånd, som kameraets filter ikke understøtter.
{% endhint %}

De nøjagtige formler, som de er implementeret (symbolerne `x`/`y`/`z` er båndpladser; standardtilknytningen vises pr. forudindstilling):

| Forudindstilling | Formel (som implementeret) | Standardfilter | Pladser (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)*(x/y)` | RGN | Red, Green, NIR |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | NGB | NIR, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Hvordan et forudindstillet navn bliver til båndpositioner

Når du angiver et simpelt navn som f.eks. `NDVI`, skal Chloros afgøre, hvilken kanal i hvilken fil hvert symbol læser fra. Det bruger denne tabel, som knytter en filterkode til array-positionen for hver kanal:

| Filterkode | Kanal → array-indeks |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` accepteres som et alias for Orange, ligeledes 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

Forudindstillingens **standardfilter** (kolonnen »Standardfilter« ovenfor) bruges, når projektet indeholder billeder med det pågældende filter. Hvis det ikke er tilfældet, gennemgår Chloros de filtre, der rent faktisk findes i projektet, i rækkefølgen `RGN, OCN, NGB, RGB, RE, NIR` og vælger det første, der kan levere alle de kanaler, som forudindstillingen har brug for. Hvis ingen kan det, fravælges forudindstillingen for den pågældende kørsel. Derfor giver `NDVI`, der anmodes om på et datasæt, der udelukkende består af OCN, stadig et fornuftigt resultat — det binder sig til OCNs Orange- og NIR-positioner.

LATTICE M3C-modelstrenge indeholder filteret med præfikset `F` (`LATT-M3C-L41-FRGN`), men præfikset fjernes, når filterkoden aflæses fra billedet, så et FRGN-kamera løser problemet via rækken `RGN` ovenfor og kræver ingen særlig håndtering.

### 3. LATTICE-indeksmotor (`lattice index --preset`, live indeksberegner) — 22 forudindstillinger

LATTICE-motoren arbejder med justerede multibåndsstakke (live-arrays eller eksporterede multibånds-TIFF-filer) og bruger kanalnavne med små bogstaver (`red`, `green`, `blue`, `red_edge`, `nir`). Dens liste over forudindstillinger adskiller sig fra de to ovenstående:

| Forudindstilling | Formel | Kanaler |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | rød, NIR |
| GNDVI | `(nir - green) / (nir + green)` | grøn, NIR |
| BNDVI | `(nir - blue) / (nir + blue)` | blå, NIR |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | rød\_kant, NIR |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | blå, grøn, nærinfrarød |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | rød, nærinfrarød |
| OSAVI | `(nir - red) / (nir + red + 0.16)` | rød, nir |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | rød, nir |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | blå, rød, nir |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | rød, nærinfrarød |
| CVI | `nir * red / green**2` | rød, grøn, nærinfrarød |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | rød, nærinfrarød |
| TDVI | `1.5 * (nir - red) / sqrt(nir**2 + red + 0.5)` | rød, nærinfrarød |
| LAI | `3.618 * (2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)) - 0.118` | blå, rød, nærinfrarød |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | rød, grøn, blå |
| NGRDI | `(green - red) / (green + red)` | rød, grøn |
| VARI | `(green - red) / (green + red - blue)` | rød, grøn, blå |
| TGI | `green - 0.39*red - 0.61*blue` | rød, grøn, blå |
| EXG | `2*green - red - blue` | rød, grøn, blå |
| CIRE | `(nir / red_edge) - 1` | rød\_kant, nir |
| CIGREEN | `(nir / green) - 1` | grøn, nir |
| NDWI | `(green - nir) / (green + nir)` | grøn, nir |

Kør `chloros-cli lattice index --list-presets` for at udskrive denne tabel fra din installerede version, og `--list-gradients` for de tilgængelige farvegradienter. Kanalsymboler er store- og småbogstavsfølsomme og skal stemme overens med forudindstillingens navne med små bogstaver (f.eks. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI – Klorofyl-vegetationsindeks

Dette indeks er følsomt over for bladklorofylindholdet på tværs af et bredt spektrum af kronetæthed. Det anvender NIR-, røde og grønne bånd.

$$
CVI = NIR * {Red \over Green^{2}}
$$

I GUI&#x27;en og på listen over forudindstillinger i CLI/SDK er symbolformlen `(z/y)*(x/y)`, hvor x=Red, y=Green, z=NIR (båndene i et RGN-kamera); LATTICE-indeksmotorens forudindstilling er `nir * red / green**2`. Hver overflade beregner det samme indeks.

_Reference: Vincini, M., E. Frazzi og P. D&#x27;Alessio. »A Broad-Band Leaf Chlorophyll Vegetation Index at the Canopy Scale.« Precision Agriculture 9 (2008): 303-319._

***

## ENDVI – Enhanced Normalized Difference Vegetation Index

Dette indeks bruger den blå kanal ud over NIR og grøn og er populært blandt NGB-filtrerede kameraer, hvor det blå bånd erstatter det røde.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Implementeringen er symbolformlen `((x+y)-(2*z))/((x+y)+(2*z))` — tildel dit kameras NIR- og Green-kanaler til x/y-pladserne og Blue til z (for et NGB-kamera: x=NIR, y=Green, z=Blue).

***

## EVI – Enhanced Vegetation Index

Dette indeks blev oprindeligt udviklet til brug med MODIS-data som en forbedring i forhold til NDVI ved at optimere vegetationssignalet i områder med højt bladarealindeks (LAI). Det er mest nyttigt i regioner med højt LAI, hvor NDVI kan blive mættet. Det bruger det blå reflektansområde til at korrigere for jordbaggrundssignaler og til at reducere atmosfæriske påvirkninger, herunder aerosolspredning.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI-værdier bør ligge mellem 0 og 1 for vegetationspixel. Lyse elementer såsom skyer og hvide bygninger samt mørke elementer såsom vand kan medføre unormale pixelværdier i et EVI-billede. Inden du opretter et EVI-billede, bør du fjerne skyer og lyse elementer fra reflektansbilledet og eventuelt indstille en tærskelværdi for pixelværdierne fra 0 til 1.

_Reference: Huete, A., et al. »Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.« Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Skovdækningsindeks 1

_Kun GUI — ikke tilgængelig som CLI-/SDK-forudindstilling `--indices`._

Dette indeks skelner mellem skovkroner og andre vegetationstyper ved hjælp af multispektrale reflektansbilleder, der inkluderer et rødkantbånd.

$$
FCI1 = Red * RedEdge
$$

Skovområder vil have lavere FCI1-værdier på grund af træernes lavere reflektans og tilstedeværelsen af skygger i kronetaget.

_Reference: Becker, Sarah J., Craig S.T. Daughtry og Andrew L. Russ. &quot;Robust forest cover indices for multispectral images.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 – Forest Cover Index 2

_Kun GUI — ikke tilgængelig som CLI- eller SDK-forudindstilling `--indices`._

Dette indeks skelner mellem skovkroner og andre vegetationstyper ved hjælp af multispektrale reflektansbilleder, der ikke indeholder et rødkantbånd.

$$
FCI2 = Red * NIR
$$

Skovområder vil have lavere FCI2-værdier på grund af træernes lavere reflektans og tilstedeværelsen af skygger i kronetaget.

_Reference: Becker, Sarah J., Craig S.T. Daughtry og Andrew L. Russ. »Robust forest cover indices for multispectral images.« Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI – Global Environmental Monitoring Index

_Kun GUI — ikke tilgængelig som CLI- eller SDK-forudindstilling `--indices`._

Dette ikke-lineære vegetationsindeks anvendes til global miljøovervågning ud fra satellitbilleder og forsøger at korrigere for atmosfæriske effekter. Det ligner NDVI, men er mindre følsomt over for atmosfæriske effekter. Det påvirkes af bar jord; derfor anbefales det ikke til brug i områder med sparsom eller moderat tæt vegetation.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Hvor:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Reference: Pinty, B., og M. Verstraete. »GEMI: a Non-Linear Index to Monitor Global Vegetation From Satellites«. Vegetation 101 (1992): 15-20._

***

## GARI – Green Atmospherically Resistant Index

_Kun GUI — ikke tilgængelig som en CLI/SDK-forudindstilling `--indices`._

Dette indeks er mere følsomt over for et bredt spektrum af klorofylkoncentrationer og mindre følsomt over for atmosfæriske påvirkninger end NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gammakonstanten er en vægtningsfunktion, der afhænger af aerosolforholdene i atmosfæren. ENVI anvender en værdi på 1,7, hvilket er den anbefalede værdi fra Gitelson, Kaufman og Merzylak (1996, side 296).

_Reference: Gitelson, A., Y. Kaufman og M. Merzylak. »Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.« Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI – Green Chlorophyll Index

Dette indeks bruges til at estimere bladklorofylindholdet på tværs af et bredt spektrum af plantearter.

$$
GCI = {NIR \over Green} - 1
$$

Brug af brede NIR- og grønne bølgelængder giver en bedre forudsigelse af klorofylindholdet, samtidig med at det muliggør større følsomhed og et højere signal-støj-forhold.

_Reference: Gitelson, A., Y. Gritz og M. Merzlyak. &quot;Sammenhængen mellem bladets klorofylindhold og spektralreflektans samt algoritmer til ikke-destruktiv vurdering af klorofylindholdet i højere planters blade.&quot; Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI – Green Leaf Index

Dette indeks blev oprindeligt udviklet til brug med et digitalt RGB-kamera til måling af hvededække, hvor de røde, grønne og blå digitale tal (DN&#x27;er) varierer fra 0 til 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI-værdier ligger mellem -1 og +1. Negative værdier repræsenterer jord og ikke-levende elementer, mens positive værdier repræsenterer grønne blade og stængler.

_Reference: Louhaichi, M., M. Borman og D. Johnson. »Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.« Geocarto International 16, nr. 1 (2001): 65-70._

***

## GNDVI – Green Normalized Difference Vegetation Index

Dette indeks ligner NDVI, bortset fra at det måler det grønne spektrum fra 540 til 570 nm i stedet for det røde spektrum. Dette indeks er mere følsomt over for klorofylkoncentrationen end NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Kilde: Gitelson, A., og M. Merzlyak. »Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.« Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI – Green Optimized Soil Adjusted Vegetation Index

Dette indeks blev oprindeligt udviklet med farve-infrarød fotografering til at forudsige kvælstofbehovet for majs. Det ligner OSAVI, men erstatter det røde bånd med det grønne.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Kilde: Sripada, R., et al. »Determining In-Season Nitrogen Requirements for Corn Using Aerial Color-Infrared Photography.« Ph.D.-afhandling, North Carolina State University, 2005._

***

## GRVI – Green Ratio Vegetation Index

Dette indeks er følsomt over for fotosyntesehastigheder i skovkronerne, da refleksionerne af grønt og rødt i høj grad påvirkes af ændringer i bladpigmenterne.

$$
GRVI = {NIR \over Green }
$$

_Reference: Sripada, R., et al. »Luftfotografering med farve-infrarødt lys til bestemmelse af tidlige kvælstofbehov i majs i vækstsæsonen.« Agronomy Journal 98 (2006): 968-977._

***

## GSAVI – Green Soil Adjusted Vegetation Index

Dette indeks blev oprindeligt udviklet med farve-infrarød fotografering til at forudsige kvælstofbehovet for majs. Det ligner SAVI, men erstatter det røde bånd med det grønne.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Kilde: Sripada, R., et al. »Bestemmelse af kvælstofbehovet for majs i vækstsæsonen ved hjælp af luftfotografering med farve-infrarødt.« Ph.d.-afhandling, North Carolina State University, 2005._

***

## LAI – Bladarealindeks

Dette indeks bruges til at estimere bladdækket og til at forudsige afgrødens vækst og udbytte. ENVI beregner det grønne LAI ved hjælp af følgende empiriske formel fra Boegh et al. (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Hvor EVI er:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Høje LAI-værdier ligger typisk mellem ca. 0 og 3,5. Når scenen imidlertid indeholder skyer og andre lyse elementer, der skaber mættede pixels, kan LAI-værdierne overstige 3,5. Ideelt set bør du maskere skyer og lyse elementer fra din scene, før du opretter et LAI-billede.

_Reference: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde og A. Thomsen. »Luftbårne multispektrale data til kvantificering af bladarealindeks, kvælstofkoncentration og fotosyntetisk effektivitet i landbruget.” Remote Sensing of Environment 81, nr. 2-3 (2002): 179-193._

***

## LCI – bladklorofylindeks

_Kun GUI — ikke tilgængelig som CLI/SDK-forudindstilling `--indices`._

Dette indeks bruges til at estimere klorofylindholdet i højere planter og er følsomt over for variationer i reflektans forårsaget af klorofylabsorption.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Reference: Datt, B. &quot;Remote Sensing of Water Content in Eucalyptus Leaves.&quot; Journal of Plant Physiology 154, nr. 1 (1999): 30-36._

***

## MNLI – Modificeret ikke-lineært indeks

Dette indeks er en videreudvikling af det ikke-lineære indeks (NLI), der inkorporerer det jordjusterede vegetationsindeks (SAVI) for at tage højde for jordbaggrunden. ENVI anvender en værdi på 0,5 for justeringsfaktoren for kronedækket (_L_).

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Reference: Yang, Z., P. Willis og R. Mueller. »Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.« Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – Modified Soil Adjusted Vegetation Index 2

Dette indeks er en forenklet version af MSAVI-indekset, som blev foreslået af Qi et al. (1994), som er en forbedring af Soil Adjusted Vegetation Index (SAVI). Det reducerer jordstøj og øger det dynamiske område for vegetationssignalet. MSAVI2 er baseret på en induktiv metode, der ikke anvender en konstant _L_-værdi (som ved SAVI) til at fremhæve sund vegetation.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Reference: Qi, J., A. Chehbouni, A. Huete, Y. Kerr og S. Sorooshian. »A Modified Soil Adjusted Vegetation Index.« Remote Sensing of Environment 48 (1994): 119-126._

***

## MSR – Modified Simple Ratio

Dette indeks er en modifikation af det simple NIR/Red-forhold, der er udviklet til at linearisere dets forhold til biofysiske parametre, og det er mere følsomt end NDVI ved højere vegetationsdensiteter.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Reference: Chen, J. »Evaluation of Vegetation Indices and a Modified Simple Ratio for Boreal Applications.« Canadian Journal of Remote Sensing 22 (1996): 229-242._

***

## NDRE – Normaliseret forskel RedEdge

Dette indeks ligner NDVI, men sammenligner kontrasten mellem NIR og RedEdge i stedet for Red, hvilket ofte registrerer vegetationsstress tidligere.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normaliseret forskel i vegetationsindeks

Dette indeks er et mål for sund, grøn vegetation. Kombinationen af dets normaliserede forskelsformulering og brugen af klorofyls områder med den højeste absorption og reflektans gør det robust under en bred vifte af forhold. Det kan dog mættes under forhold med tæt vegetation, når LAI-værdien bliver høj.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Værdien af dette indeks varierer fra -1 til 1. Det almindelige interval for grøn vegetation er 0,2 til 0,8.

_Kilde: Rouse, J., R. Haas, J. Schell og D. Deering. Overvågning af vegetationssystemer i Great Plains med ERTS. Tredje ERTS-symposium, NASA (1973): 309-317._

***

## NLI – Ikke-lineært indeks

Dette indeks antager, at forholdet mellem mange vegetationsindekser og biofysiske overfladeparametre er ikke-lineært. Det lineariserer forhold til overfladeparametre, der har tendens til at være ikke-lineære.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Reference: Goel, N., og W. Qin. &quot;Indflydelse af kronedækket på sammenhængen mellem forskellige vegetationsindekser og LAI og Fpar: En computersimulering.&quot; Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI – Optimized Soil Adjusted Vegetation Index

Dette indeks er baseret på Soil Adjusted Vegetation Index (SAVI). Det anvender en standardværdi på 0,16 for justeringsfaktoren for kronedækket. Rondeaux (1996) fastslog, at denne værdi giver større jordvariation end SAVI ved lav vegetationsdækning, samtidig med at den udviser øget følsomhed over for vegetationsdækning på over 50 %. Dette indeks anvendes bedst i områder med relativt sparsom vegetation, hvor jorden er synlig gennem kronetaget.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Kilde: Rondeaux, G., M. Steven og F. Baret. »Optimization of Soil-Adjusted Vegetation Indices.« Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI – Renormaliseret differensvegetationsindeks

Dette indeks anvender forskellen mellem bølgelængder i det nærinfrarøde og det røde spektrum sammen med NDVI til at fremhæve sund vegetation. Det er ufølsomt over for påvirkninger fra jordbund og solens indfaldsvinkel.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Reference: Roujean, J., og F. Breon. &quot;Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.&quot; Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI – Jordjusteret vegetationsindeks

Dette indeks ligner NDVI, men det undertrykker virkningerne af jordpixels. Det anvender en justeringsfaktor for kronedækket, _L_, som er en funktion af vegetationstætheden og ofte kræver forudgående viden om vegetationsmængderne. Huete (1988) foreslår en optimal værdi på _L_=0,5 for at tage højde for førsteordens variationer i jordbaggrunden. Dette indeks anvendes bedst i områder med relativt sparsom vegetation, hvor jorden er synlig gennem kronlaget.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Reference: Huete, A. »A Soil-Adjusted Vegetation Index (SAVI).« Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – Transformed Difference Vegetation Index

Dette indeks er nyttigt til overvågning af vegetationsdækket i bymiljøer. Det mætter ikke på samme måde som NDVI og SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Reference: Bannari, A., H. Asalhi og P. Teillet. »Transformed Difference Vegetation Index (TDVI) til kortlægning af vegetationsdække« I Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, bind 5 (2002)._

***

## VARI – Visible Atmospherically Resistant Index

Dette indeks er baseret på ARVI og bruges til at estimere vegetationsandelen i et billede med lav følsomhed over for atmosfæriske effekter.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Reference: Gitelson, A., et al. »Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction«. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Wide Dynamic Range Vegetation Index

Dette indeks ligner NDVI, men det anvender en vægtningskoefficient (_a_) for at reducere forskellen mellem bidragene fra de nærinfrarøde og røde signaler til NDVI. WDRVI er særligt effektivt i scener med moderat til høj vegetationsdensitet, når NDVI overstiger 0,6. NDVI har en tendens til at flade ud, når vegetationsandelen og bladarealindekset (LAI) stiger, mens WDRVI er mere følsomt over for et bredere spektrum af vegetationsandele og ændringer i LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Vægtningskoefficienten (_a_) kan variere fra 0,1 til 0,2. En værdi på 0,2 anbefales af Henebry, Viña og Gitelson (2004).

_Referencer_

_Gitelson, A. »Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.« Journal of Plant Physiology 161, nr. 2 (2004): 165-173._

_Henebry, G., A. Viña og A. Gitelson. »Vegetationsindekset med bredt dynamisk område og dets potentielle anvendelighed til gap-analyse.« Gap Analysis Bulletin 12: 50-56._

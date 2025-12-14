---
description: This page lists some multispectral indices that Chloros uses.
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---
# Multispektrale indeksformler

Nedenstående indeksformler bruger en kombination af Survey3-filterets gennemsnitlige transmissionsområder:

<table><thead><tr><th align="center">Survey3-filterfarve</th><th width="196.199951171875" align="center">Survey3-filternavn</th><th width="159.800048828125" align="center">Transmissionsområde (FWHM)</th><th align="center">Gennemsnitlig transmission</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Når disse formler anvendes, kan navnet slutte på &quot;\_1&quot; eller &quot;\_2&quot;, hvilket svarer til, hvilket NIR-filter, enten NIR1 eller NIR2, der blev anvendt.

***

## EVI - Forbedret vegetationsindeks

Dette indeks blev oprindeligt udviklet til brug med MODIS-data som en forbedring i forhold til NDVI ved at optimere vegetationssignalet i områder med højt bladarealindeks (LAI). Det er mest nyttigt i områder med høj LAI, hvor NDVI kan blive mættet. Det bruger det blå reflektansområde til at korrigere for jordbaggrundssignaler og reducere atmosfæriske påvirkninger, herunder aerosolspredning.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

EVI-værdier bør ligge mellem 0 og 1 for vegetationspixels. Lyse elementer såsom skyer og hvide bygninger samt mørke elementer såsom vand kan resultere i unormale pixelværdier i et EVI-billede. Inden du opretter et EVI-billede, bør du maske skyer og lyse elementer fra reflektansbilledet og eventuelt sætte en tærskel for pixelværdierne fra 0 til 1.

_Reference: Huete, A., et al. &quot;Oversigt over den radiometriske og biofysiske ydeevne af MODIS-vegetationsindekser.&quot; Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Skovdækningsindeks 1

Dette indeks skelner mellem skovkroner og andre typer vegetation ved hjælp af multispektrale reflektansbilleder, der inkluderer et rødt kantbånd.

$$
FCI1 = Red * RedEdge
$$

Skovområder vil have lavere FCI1-værdier på grund af træernes lavere reflektans og tilstedeværelsen af skygger i kronen.

_Reference: Becker, Sarah J., Craig S.T. Daughtry og Andrew L. Russ. &quot;Robuste skovdækningsindekser for multispektrale billeder.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Skovdækningsindeks 2

Dette indeks skelner mellem skovkroner og andre typer vegetation ved hjælp af multispektrale reflektansbilleder, der ikke inkluderer et rødt kantbånd.

$$
FCI2 = Red * NIR
$$

Skovområder vil have lavere FCI2-værdier på grund af træernes lavere reflektans og tilstedeværelsen af skygger i kronetaget.

_Reference: Becker, Sarah J., Craig S.T. Daughtry og Andrew L. Russ. &quot;Robuste skovdækningsindekser for multispektrale billeder.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Globalt miljøovervågningsindeks

Dette ikke-lineære vegetationsindeks bruges til global miljøovervågning fra satellitbilleder og forsøger at korrigere for atmosfæriske effekter. Det ligner NDVI, men er mindre følsomt over for atmosfæriske effekter. Det påvirkes af bar jord og anbefales derfor ikke til brug i områder med sparsom eller moderat tæt vegetation.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Hvor:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Reference: Pinty, B., og M. Verstraete. GEMI: et ikke-lineært indeks til overvågning af global vegetation fra satellitter. Vegetation 101 (1992): 15-20._

***

## GARI - Green Atmosfærisk resistent indeks

Dette indeks er mere følsomt over for en bred vifte af klorofylkoncentrationer og mindre følsomt over for atmosfæriske effekter end NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Gamma-konstanten er en vægtningsfunktion, der afhænger af aerosolforholdene i atmosfæren. ENVI bruger en værdi på 1,7, som er den anbefalede værdi fra Gitelson, Kaufman og Merzylak (1996, side 296).

_Reference: Gitelson, A., Y. Kaufman og M. Merzylak. &quot;Brug af en Green-kanal i fjernmåling af global vegetation fra EOS-MODIS.&quot; Fjernmåling af miljøet 58 (1996): 289-298._

***

## GCI - Green Klorofylindeks

Dette indeks bruges til at estimere bladklorofylindholdet i en lang række plantearter.

$$
GCI = {NIR \over Green} - 1
$$

Bred NIR og grønne bølgelængder giver en bedre forudsigelse af klorofylindholdet, samtidig med at det giver større følsomhed og et højere signal-støj-forhold.

_Reference: Gitelson, A., Y. Gritz og M. Merzlyak. &quot;Forholdet mellem bladklorofylindhold og spektral reflektans og algoritmer til ikke-destruktiv klorofylvurdering i højere planteblade.&quot; Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Green Bladindeks

Dette indeks blev oprindeligt designet til brug med et digitalt RGB-kamera til måling af hvededække, hvor de røde, grønne og blå digitale tal (DN) varierer fra 0 til 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI-værdierne varierer fra -1 til +1. Negative værdier repræsenterer jord og ikke-levende elementer, mens positive værdier repræsenterer grønne blade og stængler.

_Reference: Louhaichi, M., M. Borman og D. Johnson. &quot;Spatially Located Platform and Aerial Photography for Documentation of Grazing Impacts on Wheat.&quot; Geocarto International 16, nr. 1 (2001): 65-70._

***

## GNDVI - Green Normaliseret forskelsvegetationsindeks

Dette indeks ligner NDVI, bortset fra at det måler det grønne spektrum fra 540 til 570 nm i stedet for det røde spektrum. Dette indeks er mere følsomt over for klorofylkoncentration end NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Reference: Gitelson, A., og M. Merzlyak. &quot;Fjernmåling af klorofylkoncentration i højere planters blade.&quot; Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Optimeret jordjusteret vegetationsindeks

Dette indeks blev oprindeligt udviklet med farve-infrarød fotografering til at forudsige kvælstofbehovet for majs. Det ligner OSAVI, men erstatter det grønne bånd med det røde.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Reference: Sripada, R., et al. &quot;Bestemmelse af sæsonens kvælstofbehov for majs ved hjælp af luftfotografering med farve-infrarød fotografering.&quot; Ph.D.-afhandling, North Carolina State University, 2005._

***

## GRVI - Green Ratio Vegetation Index

Dette indeks er følsomt over for fotosyntesehastigheder i skovkroner, da grøn og rød refleksion er stærkt påvirket af ændringer i bladpigmenter.

$$
GRVI = {NIR \over Green }
$$

_Reference: Sripada, R., et al. &quot;Luftfotografering med farve-infrarød til bestemmelse af tidlige kvælstofbehov i majs i vækstsæsonen.&quot; Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Jordjusteret vegetationsindeks

Dette indeks blev oprindeligt udviklet med farve-infrarød fotografering til at forudsige kvælstofbehovet for majs. Det ligner SAVI, men erstatter det grønne bånd med det røde.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Reference: Sripada, R., et al. &quot;Bestemmelse af sæsonens kvælstofbehov for majs ved hjælp af luftfotografering med farve-infrarød fotografering.&quot; Ph.D.-afhandling, North Carolina State University, 2005._

***

## LAI - Bladarealindeks

Dette indeks bruges til at estimere løvdækket og til at forudsige afgrødens vækst og udbytte. ENVI beregner grøn LAI ved hjælp af følgende empiriske formel fra Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Hvor EVI er:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Høje LAI-værdier ligger typisk mellem ca. 0 og 3,5. Men når scenen indeholder skyer og andre lyse elementer, der producerer mættede pixels, kan LAI-værdierne overstige 3,5. Ideelt set bør du maske skyer og lyse elementer fra din scene, inden du opretter et LAI-billede.

_Reference: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde og A. Thomsen. &quot;Airborne Multi-spectral Data for Quantifying Leaf Area Index, Nitrogen Concentration and Photosynthetic Efficiency in Agriculture.&quot; Remote Sensing of Environment 81, nr. 2-3 (2002): 179-193._

***

## LCI - Bladklorofylindeks

Dette indeks bruges til at estimere klorofylindholdet i højere planter, der er følsomme over for variationer i refleksion forårsaget af klorofylabsorption.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Reference: Datt, B. &quot;Fjernmåling af vandindholdet i eukalyptusblade.&quot; Journal of Plant Physiology 154, nr. 1 (1999): 30-36._

***

## MNLI - Modificeret ikke-lineært indeks

Dette indeks er en forbedring af det ikke-lineære indeks (NLI), der inkorporerer det jordjusterede vegetationsindeks (SAVI) for at tage højde for jordbunden. ENVI bruger en værdi på 0,5 for justeringsfaktoren for kronetæppe (_L_).

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Reference: Yang, Z., P. Willis og R. Mueller. &quot;Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.&quot; Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 - Modified Soil Adjusted Vegetation Index 2

Dette indeks er en enklere version af MSAVI-indekset foreslået af Qi et al (1994), som forbedrer Soil Adjusted Vegetation Index (SAVI). Det reducerer jordstøj og øger det dynamiske område for vegetationssignalet. MSAVI2 er baseret på en induktiv metode, der ikke bruger en konstant _L_-værdi (som med SAVI) til at fremhæve sund vegetation.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Reference: Qi, J., A. Chehbouni, A. Huete, Y. Kerr og S. Sorooshian. &quot;A Modified Soil Adjusted Vegetation Index.&quot; Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE- Normaliseret forskel RedEdge

Dette indeks ligner NDVI, men sammenligner kontrasten mellem NIR og RedEdge i stedet for Red, som ofte opdager vegetationsstress hurtigere.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Normaliseret vegetationsindeks

Dette indeks er et mål for sund, grøn vegetation. Kombinationen af dets normaliserede forskelsformulering og brugen af de højeste absorptions- og refleksionsregioner af klorofyl gør det robust under en lang række forhold. Det kan dog blive mættet under tætte vegetationsforhold, når LAI bliver højt.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Værdien af dette indeks varierer fra -1 til 1. Det almindelige interval for grøn vegetation er 0,2 til 0,8.

_Reference: Rouse, J., R. Haas, J. Schell og D. Deering. Overvågning af vegetationssystemer i Great Plains med ERTS. Tredje ERTS-symposium, NASA (1973): 309-317._

***

## NLI - Non-Linear Index (Ikke-lineært indeks)

Dette indeks antager, at forholdet mellem mange vegetationsindekser og biofysiske parametre på overfladen er ikke-lineært. Det lineariserer forholdet til overfladeparametre, der har tendens til at være ikke-lineære.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Reference: Goel, N., og W. Qin. &quot;Indflydelse af kronearkitektur på forholdet mellem forskellige vegetationsindekser og LAI og Fpar: En computersimulation.&quot; Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - Optimeret jordjusteret vegetationsindeks

Dette indeks er baseret på det jordjusterede vegetationsindeks (SAVI). Det bruger en standardværdi på 0,16 for justeringsfaktoren for kronedækket. Rondeaux (1996) fastslog, at denne værdi giver større jordvariation end SAVI for lav vegetation, samtidig med at den viser øget følsomhed over for vegetation på over 50 %. Dette indeks bruges bedst i områder med relativt sparsom vegetation, hvor jorden er synlig gennem kronedækket.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Reference: Rondeaux, G., M. Steven og F. Baret. &quot;Optimization of Soil-Adjusted Vegetation Indices.&quot; Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - Renormaliseret forskelsvegetationsindeks

Dette indeks bruger forskellen mellem nærinfrarøde og røde bølgelængder sammen med NDVI til at fremhæve sund vegetation. Det er ufølsomt over for jordens og solens geometriske indvirkning.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Reference: Roujean, J. og F. Breon. &quot;Estimering af PAR absorberet af vegetation fra tovejsreflektansmålinger.&quot; Fjernmåling af miljøet 51 (1995): 375-384._

***

## SAVI - Jordjusteret vegetationsindeks

Dette indeks ligner NDVI, men det undertrykker effekten af jordpixels. Det bruger en justeringsfaktor for baldakinbaggrund, _L_, som er en funktion af vegetationsdensitet og kræver ofte forudgående viden om vegetationsmængder. Huete (1988) foreslår en optimal værdi på _L_=0,5 for at tage højde for førsteordens variationer i jordbaggrunden. Dette indeks bruges bedst i områder med relativt sparsom vegetation, hvor jorden er synlig gennem kronetaget.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Reference: Huete, A. &quot;A Soil-Adjusted Vegetation Index (SAVI).&quot; Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - Transformed Difference Vegetation Index

Dette indeks er nyttigt til overvågning af vegetationsdækket i bymiljøer. Det mættes ikke som NDVI og SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Reference: Bannari, A., H. Asalhi og P. Teillet. &quot;Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping&quot; I Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, bind 5 (2002)._

***

## VARI - Synligt atmosfærisk resistent indeks

Dette indeks er baseret på ARVI og bruges til at estimere andelen af vegetation i et område med lav følsomhed over for atmosfæriske effekter.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Reference: Gitelson, A., et al. &quot;Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI - Vegetationsindeks med bredt dynamisk område

Dette indeks ligner NDVI, men det bruger en vægtningskoefficient (_a_) til at reducere forskellen mellem bidragene fra de nærinfrarøde og røde signaler til NDVI. WDRVI er særligt effektivt i scener med moderat til høj vegetationsdensitet, når NDVI overstiger 0,6. NDVI har tendens til at udjævne sig, når vegetationsfraktionen og bladarealindekset (LAI) stiger, mens WDRVI er mere følsom over for et bredere spektrum af vegetationsfraktioner og ændringer i LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Vægtningskoefficienten (_a_) kan variere fra 0,1 til 0,2. En værdi på 0,2 anbefales af Henebry, Viña og Gitelson (2004).

_Referencer_

_Gitelson, A. &quot;Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.&quot; Journal of Plant Physiology 161, nr. 2 (2004): 165-173._

_Henebry, G., A. Viña og A. Gitelson. &quot;The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis.&quot; Gap Analysis Bulletin 12: 50-56._

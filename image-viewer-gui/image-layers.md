# Billedlag

**Rullemenuen med lag** øverst til højre i billedviseren skifter mellem alle versioner af det billede, du ser på — fra kildebilledet over hvert bearbejdet produkt til de beregnede indeksbilleder — uden at du behøver at forlade viseren.

## Hvad er billedlag?

Et »lag« i Chloros er en **produktfil**, der er registreret i forhold til et kildebillede. Importen giver dig kildefilerne; behandlingen tilføjer et lag for hvert produkt, som kørslen har genereret. Eksporterede filer beholder kildefilnavnet — det er**mappen**, der identificerer produktet, og lagets navn er Chloros’s betegnelse for den pågældende mappe.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Laglisten

### Altid til stede

| Lag | Hvad det er |
| --- | --- |
| **JPG**(eller**PNG**/**TIFF**) | Den grundfil, der fulgte med optagelsen. Survey3 importerer en `.JPG` ved siden af hver `.RAW`; LATTICE-optagelser medfører en PNG- eller TIFF-visningsforhåndsvisning. Mærket efter det, der faktisk blev importeret |
| **RAW (Original)** | Den rå kildebillede, der er debayered til visning uden anvendte korrektioner. Tilgængelig fra importtidspunktet — kræver ingen behandling |

En LATTICE-optagelse, hvis basisfil **er** dens råramme, har ingen separat basispost: `RAW (Original)` dækker den allerede.

### Survey3-behandlingsprodukter

| Lag | Skrevet til | Findes, når |
| --- | --- | --- |
| **RAW (Mål)** | — | Billedet blev identificeret som indeholdende et kalibreringsmål |
| **RAW (reflektans)** | `Reflectance_Calibrated_Images/` | Reflektanskalibrering blev udført med succes på denne ramme |
| **Vignettekorrigeret**| `Vignette_Corrected_Images/` | Billedet kunne ikke reflektanskalibreres**og** *vignettekorrektion* var aktiveret |
| **Sensorrespons**| `Sensor_Response_Images/` | Billedet kunne ikke refleksionskalibreres**og** *vignetteringskorrektion* var slået fra |
| **Hvidbalance** | `White_Balanced_Images/` | Der blev skrevet et produkt med hvidbalance |

{% hint style="info" %}
**Vignettekorrektion og sensorrespons er alternativer, aldrig begge dele.** Der findes nøjagtigt ét ukalibreret reserveprodukt pr. kørsel for hver kameramodel, og *Vignettekorrektion*-kontakten vælger, hvilket det skal være. Se [Projektindstillinger](../project-settings/project-settings.md).
{% endhint %}

### LATTICE-niveauer

LATTICE inddeler fan-out i disse i et enkelt behandlingsforløb. Hvilke der findes, afhænger af eksportindstillingerne pr. produkt i Projektindstillinger og af, hvad der gælder for kameraet.

| Lag | Skrives til | Gælder for |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB og multispektral |
| **RAW (Forhåndsvisning)** | `Preview_Images/` | Multispektralt (falskfarvestrækning) |
| **Hvidbalanceret** | `Preview_Images/` | RGB-masterkameraer — forhåndsvisningen af RGB er registreret under dette navn, så den passer sammen med laget Survey3 med samme navn |
| **RAW (stråling)** | `Radiance_Images/` | Kun multispektral |
| **RAW (reflektans)** | `Reflectance_Calibrated_Images/` | Kun multispektralt, og kun når en matchende `.daq`-nedstrålingsregistrering eller et QA-godkendt mål inden for billedrammen dækker billedrammen |

RGB-masterkameraer har ingen radiometri pr. bånd, så radiance og reflectance springes over for dem som **ikke relevant** — loggen angiver dette i stedet for at fejle uden besked.

### Indeks-, LUT- og sandkasse-lag

| Lagmønster | Eksempel | Hvor det kommer fra |
| --- | --- | --- |
| **RAW (`<INDEX>`-indeks)** | `RAW (NDVI Index)` | Ét pr. indeks konfigureret i projektindstillingerne, beregnet under behandlingen |
| **`<INDEX>` LUT** | `NDVI LUT` | Den farvekortlagte version af et indeks |
| **Sandkasse (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Én pr. [Indeks/LUT-sandkasse](index-lut-sandbox.md)-eksportkørsel |

Hvis det samme indeksnavn konfigureres mere end én gang med forskellige indstillinger, får det andet og de efterfølgende et nummer i navnet (`RAW (NDVI2 Index)`), så lagene forbliver let genkendelige.

***

## Brug af lagvælgeren

1. Åbn et billede i fuld skærm ved at klikke på en miniature i rutenettet
2. Klik på **lagsrullemenuen** øverst til højre i visningsvinduet
3. Vælg et lag — billedet opdateres med det samme

Rullemenuen viser først **JPG, RAW (Original), RAW (Target), RAW (Reflectance)** i den rækkefølge, og viser alt andet efter dem i den rækkefølge, produkterne blev registreret.

### Lagpræference, når du navigerer

Når du trykker på **←**/**→**, går du til det næste billede, og systemet forsøger at holde dig på det samme lag:

1. **Præcis match først** — hvis det næste billede har et lag med samme navn, får du det. Det er dette, der holder dig på `RAW (NDVI Index)`, mens du gennemgår et helt sæt
2. **Derefter et match efter type** — et indekslag søger efter ethvert indekslag, en LUT efter enhver LUT, reflektans efter reflektans, mål efter mål, original efter original, base efter base
3. **Derefter, kun for eksportlag** — navnet bevares, selvom laglisten endnu ikke er ajourført, fordi filen allerede findes på disken. Det er derfor, du kan gennemgå produkter, mens en kørsel stadig er i gang med at skrive dem
4. **I alle andre tilfælde** — det første tilgængelige lag, som normalt er basisbilledet

`.daq`- og `.csv`-sidecar-filer i projektet springes over ved navigation med piletasterne, så man aldrig ender på en lyssensoroptagelse, når man bladrer gennem billederne.

Zoom og panorering overføres også mellem billederne, hvilket gør det nemt at sammenligne før og efter for den samme feltposition.

***

## Forståelse af pixelværdier pr. lag

Panelet [Cursor Values](opening-an-image-full-screen.md#cursor-values) viser den faktiske værdi pr. kanal under din markør i den enhed, som laget er gemt i. Kolonnerne ændrer sig afhængigt af laget:

| Lag | Vist enhed | Bemærkninger |
| --- | --- | --- |
| Base (JPG / PNG / TIFF-forhåndsvisning) | DN, 0–255 | Visningsværdier, gammakorrigeret i RGB. Udelukkende til visuel inspektion |
| RAW (Original) | DN | Rå digitale sensortal. Histogramaksen angiver dybden: 255 (8-bit), 4095 (12-bit) eller 65535 (16-bit) |
| RAW (Debayered) | DN | Lineær, ingen strækning af visningen |
| RAW (Forhåndsvisning) / Hvidbalance | DN | Visningsprodukt — strækket eller gammakorrigeret. Ikke til måling |
| RAW (Strålingsintensitet) | **W/m²/sr/nm** | Float32 fysisk strålingsintensitet. Ingen DN-kolonne |
| RAW (refleksion) | DN **og %** | Procent beregnet ud fra filens egen skala — se nedenfor |
| Indeks / LUT / sandkasse-eksport | Indeksværdi eller RGB-komponenter | En enkeltkanals indeksfil angiver indeksværdien; en farvekortlagt LUT-fil angiver Red/Green/Blue-komponenter |

### Refleksionsgrad: skalaen er pr. fil

{% hint style="warning" %}
**&quot;Divider med 65.535&quot; er kun korrekt for Survey3.** LATTICE-reflektans gemmes i en anden skala, og at blande de to divisorer er den mest almindelige måde at få reflektansværdier, der er nøjagtigt halvdelen af, hvad de burde være.
{% endhint %}

| Kilde | DN, der svarer til reflektans 1,0 | Identificeret ved |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | XMP-tagget `Chloros:PixelScale=32768`, der er indstemplet i hver eneste LATTICE-reflektans-eksport. Den dobbelte headroom betyder, at ρ over 1,0 kan repræsenteres i stedet for at blive beskåret |
| **Survey3**|**65535** | Intet Chloros XMP-skalamærke — Survey3-kalibrering skriver ρ × dtype-max og beskærer ved 1,0 |

Til GIS og scripting: Læs `Chloros:PixelScale` fra filen og divider med det. Hvis tagget mangler, er filen Survey3-skala (65535). Visningsprogrammet, indeks-/LUT-sandkassen og indekseksporten beregner alle skalaen på samme måde, så det tal, du ser ved markøren, er det tal, som indeksberegningen har brugt.

Formatspecifik lagring oven på denne skala:

* **TIFF (32-bit, procent)** lagrer DN / 65535 som et float-tal
* **PNG (8-bit)**og**JPG (8-bit)** gemmer DN × 255 / 65535
* En **8-bit TIFF-eksport af en 8-bit-kildeoptagelse** beskæres til 0–255 i stedet for at blive omskaleret og har bevidst ingen skaleringsmærke. Panelet viser kun DN for disse filer, uden procentkolonne

### Indeksværdier

| Indeksfamilie | Typisk interval | Aflæsning |
| --- | --- | --- |
| Normaliseret forskel (NDVI, GNDVI, NDRE, ENDVI…) | −1 til +1 | Sund vegetation ligger typisk mellem 0,4 og 0,9; bar jord tæt på 0; vand er negativt |
| Jordjusteret (SAVI, OSAVI, MSAVI2…) | ca. −1 til +1,5 | Værdi svarende til NDVI med jordbaggrunden undertrykt |
| Forhold (GRVI, GCI, MSR, CIRE…) | ubegrænset over | Forholdene stiger ubegrænset, når nævnerbåndet går mod nul |
| EVI / LAI | 0 til ~1, 0 til ~3,5 | Skyer og andre mættede pixels skubber begge værdier ud af området — masker dem først |

Se [Formler for multispektrale indekser](../project-settings/multispectral-index-formulas.md) for den nøjagtige formel bag hver forudindstilling.

***

## Almindelige arbejdsgange

### Før/efter-sammenligning

1. Vælg **RAW (Original)** og bemærk vignetteringen og de ukalibrerede værdier
2. Skift til **RAW (Reflectance)**

3. Sammenlign — vignettering fjernet, værdier kalibreret. Zoom og panorering forbliver faste, så du ser på det samme område

### Gennemgå et indeks på tværs af et helt sæt

1. Åbn det første behandlede billede, og vælg indekslaget
2. Tryk gentagne gange på **→** — indekslaget følger med fra billede til billede
3. Hold øje med histogrammet i sidepanelet undervejs: et billede, hvor fordelingen springer, er værd at se nærmere på

### Bekræft kalibreringsmål

1. Vælg **RAW (Target)** på en målramme
2. Bekræft, at målet er tydeligt synligt og registreret
3. Gå til den næste målramme — mållaget følger med

### Kontroller nøjagtigheden af reflektansværdierne

1. Vælg **RAW (Reflectance)**

2. Læs kolonnen**%** i panelet Cursor Values — den er allerede skaleret korrekt for den pågældende fil
3. Kontroller mod kendte materialer i billedet: sund vegetation har høje værdier i NIR og lave værdier i rødt; et kalibreringsmål bør vise værdier tæt på de offentliggjorte reflektansværdier

***

## Fejlfinding

### Et lag, jeg forventede, er ikke i rullemenuen

**Mulige årsager**

* Billedet er aldrig blevet behandlet — kun basislaget og `RAW (Original)` findes
* Produktets eksportknap er ikke markeret i projektindstillingerne
* Produktet gælder ikke for det pågældende kamera (radiance og reflektans på et RGB-masterkamera; ethvert indeks på et enkeltbånds M3M-monokamera)
* Reflektanskalibreringen havde intet at arbejde med — ingen `.daq`-dækning af nedadgående stråling og intet QA-godkendt mål inden for billedrammen — så billedet faldt tilbage til „Vignette Corrected“ eller „Sensor Response“

**Hvad skal du gøre**

1. Tjek kørselsloggen: Chloros angiver, hvornår et anmodet eksportprodukt ikke kunne leveres, og hvorfor
2. Tjek eksportindstillingerne for hvert produkt i [Projektindstillinger](../project-settings/project-settings.md)
3. Bekræft, at produktmappen findes i projektets output-struktur
4. Kør processen igen med produktet aktiveret

### Laglisten ser forældet ud

Chloros genindlæser projektets produktmapper, mens en kørsel er i gang, og retter manglende lagregistreringer ud fra det, der faktisk findes på disken. Derfor vises et lag, der er færdigexporteret, normalt af sig selv i en afstemning. Hvis du skifter væk fra billedet og tilbage igen, tvinges der en ny opløsning frem.

### Reflektansværdierne ser ud til at være halvt så store, som de burde være

Du deler næsten helt sikkert en LATTICE-fil med 65535. Brug `Chloros:PixelScale` (32768), eller se i kolonnen **%**, hvor denne værdi allerede er anvendt.

### Indekslaget findes, men billedet er tomt

Indekset kræver bånd, som dit lag ikke har — for eksempel et indeks, der læser en tredje kanal, anvendt på en fil med én eller to kanaler. Skift til et multibåndslag (reflektans eller debayered), eller vælg et indeks, der passer til kameraets filter.

***

## Næste trin

* [**Åbning af et billede i fuldskærm**](opening-an-image-full-screen.md) — markørvisning, histogram og GSD-styring
* [**Indeks/LUT-sandkasse**](index-lut-sandbox.md) — interaktiv indeksvisualisering og eksport
* [**Multispektrale indeksformler**](../project-settings/multispectral-index-formulas.md) — indeksreferencen
* [**Afslutning af behandlingen**](../processing-images-gui/finishing-the-processing.md) — den udgangsmappestruktur, som disse lag peger på

# Åbning af et billede i fuld skærm

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Et billede åbnet i fuld skærm, med lagvælgeren øverst til højre</p></figcaption></figure>

Chloros Image Viewer er grænsefladen i fuld skærm til visning, inspektion og måling af dine billeder. Det er her, du kan aflæse **reelle pixelværdier** — DN pr. kanal, reflektans i procent eller strålingsintensitet i W/m²/sr/nm — i stedet for det strakte forhåndsvisningsbillede, som skærmen viser.

## Åbning af billedviseren

### Fra filbrowseren

1. Åbn fanen **Filbrowser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klik på en vilkårlig **miniature** i [billedgitteret](image-grid.md)
3. Billedet åbnes i fuld skærm på fanen **Billedfremviser**

Billedet åbnes på det produkt, som billedgitteret viste. Hvis billedgitteret er indstillet til `RAW (Reflectance)`, er det det lag, du kommer til.

### Åbning af sidepanelet i billedviseren

Klik på ikonet **Billedviser** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> i venstre sidepanel for at åbne analysepanelet. Det indeholder, fra top til bund:

* billednavnet og dets kameramodel
* knappen **Eksporter/gem billede(r)** (kun når et indeks eller en LUT er aktivt)
* afkrydsningsfelterne **Indeks**og**LUT** samt indeks-konfigurationspanelet — se [Indeks/LUT-sandkasse](index-lut-sandbox.md)
* panelet **Markørværdier**: aflæsning pr. kanal, laghistogram og GSD-kontrol***

## Navigation og zoom

### Gennemse billeder

* **Næste billede**: →-knappen eller**→**-tasten (højrepil)
* **Forrige billede**: ←-knappen eller**←**-tasten (venstrepil)
* **Spring til et bestemt billede**: Gå tilbage til gitteret og klik på dets miniaturebillede

Zoom og panorering bevares, når du skifter mellem billeder, så du kan gennemgå en serie, mens du forbliver på samme del af billedet.

### Zoom

Zoom styres med **musens rullehjul** i trin på 15 %, forankret til markøren — det punkt, der er under markøren, forbliver under markøren. Området er afgrænset af billedets og vinduets størrelse: Du kan ikke zoome ud ud over »tilpas til vindue«, og den øvre grænse er fastsat af billedets oprindelige opløsning.

Der er ingen dedikerede zoomtaster i fuldskærmsviseren. (I gitteret ændrer **Ctrl + `+` / `−`** størrelsen på miniaturerne — en anden funktion.)

### Panorering ved zoom

Klik og hold venstre museknap nede over billedet, og træk. Panoreringen er begrænset, så billedet ikke kan trækkes ud af skærmbilledet.

### Inspektion pr. pixel ved høj zoom

Når den effektive forstørrelse overstiger **60×**, tegner Chloros en markeringsramme omkring den enkelte viste pixel under markøren samt en flydende værdi ved siden af den.

Den »effektive« forstørrelse tager højde for GSD-blokstørrelsen: med en blokstørrelse på 8 vises markeringen ved 7,5× zoom i stedet for 60×, fordi én vist pixel allerede svarer til 8 × 8 kildepixels. Zoom ud til under tærsklen, og markeringen forsvinder.

### Tastaturgenveje

| Tast                             | Hvor       | Handling                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Fuld skærm | Næste billede                          |
| **←**                           | Fuld skærm | Forrige billede                      |
| **Ctrl + R**                    | Fuld skærm | Nulstil indeks/LUT-sandkassen         |
| **Ctrl + `+`**/**Ctrl + `=`** | Gitter        | Større miniaturer (4 px pr. tryk)  |
| **Ctrl + `−`**                  | Gitter        | Mindre miniaturer (4 px pr. tryk) |***

## Cursorværdier

Flyt markøren over billedet, og panelet **Cursorværdier** viser værdien for hver kanal under det.

{% hint style="success" %}
**Dette er filens reelle tal.** Lærredet på skærmen er en 8-bit udstrakt forhåndsvisning og kan ikke levere dem, så Chloros udtager prøver fra den faktiske produktfil til aflæsningen. Det er derfor, at en 12-bit rå ramme viser værdier over 255, og hvorfor et float32-radiance-lag angiver fysiske enheder.
{% endhint %}

### Hvad kolonnerne betyder

Panelet tilpasser sig det lag, du ser på:

| Det lag, du ser på              | Viste kolonner    | Bemærkninger                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Refleksion                        | **DN**og**%** | Procentdelen beregnes ud fra filens egen skala — se nedenfor                                      |
| Strålingsintensitet                           | **W/m²/sr/nm**   | Fysiske værdier i flydende kommaformat; ingen DN-kolonne, da en DN her er meningsløs                           |
| Rå / Debayered / forhåndsvisning / JPG    | **DN**           | Hele digitale tal                                                                         |
| Eksport af 32-bit procentvis reflektans | **%** alene       | Den gemte flydende værdi er ikke en DN, så afrunding til et heltal ville give en meningsløs værdi som `0` eller `1` |

Hver række er mærket med kanalnavnet for dit kameras filter — `Red / Green / NIR` for RGN, `Orange / Cyan / NIR` for OCN, `NIR / Green / Blue` for NGB, `Red / Green / Blue` for RGB samt det enkelte båndnavn for RE-, NIR- og mono-M3M-kameraer. Hver etiket er forsynet med en farvet prik, der svarer til de kanalkredse, der anvendes i indeksformelredigeringsprogrammet.

Gemt **indeks- og LUT**-billeder er et særligt tilfælde: de indeholder farvekortkomponenter i stedet for spektralbånd, så deres rækker er mærket `Red / Green / Blue` (eller `Index` for en enkelt-kanals indeksfil) i stedet for med kameraets filternavne.

Når et indeks er aktivt i sandkassen, vises der en ekstra række under kanalerne, der viser **indeksværdien** ved markøren, sammen med indeksets navn og en hvid prik, der svarer til markøren på histogrammet.

### Reflektansprocenten bruger den enkelte fils egen skala

{% hint style="warning" %}
**Antag ikke, at 65535 = 100 %.** Chloros gemmer reflektans på forskellige skalaer afhængigt af, hvilket kamera der har produceret den, og visningsprogrammet fastlægger den korrekte skala for hver fil.
{% endhint %}

| Kilde                  | DN, der svarer til reflektans 1,0 | Sådan identificeres den                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | XMP-tagget `Chloros:PixelScale=32768` skrives ind i hver LATTICE-reflektans-eksport. Den 2× store headroom gør det muligt for filen at indeholde ρ over 1,0 uden klipping |
| **Survey3**|**65535**                      | Intet Chloros XMP-skaleringstag — Survey3-kalibreringen skriver ρ × dtype-max og klipper ved 1,0                                                               |

Visningsprogrammet, indeks-/LUT-sandkassen og indekseksporten løser alle skaleringen gennem den samme implementering, så en værdi, du aflæser ved markøren, er den samme værdi, som indeksberegningen har brugt.

To konsekvenser, der er værd at kende:

* En **32-bit procent**TIFF gemmer DN/65535 som et float-tal, og en**8-bit** PNG/JPG-eksport gemmer DN × 255/65535 — visningsprogrammet konverterer begge værdier tilbage, før der vises en procent.
* Ét tilfælde kan ikke gendannes: en **8-bit TIFF-eksport af en optagelse med 8-bit-kilde** beskæres til 0–255 i stedet for at blive omskaleret og bærer bevidst ingen skaleringsmærke. For disse filer udskriver panelet kun DN uden procentkolonne. Dette er det ærlige svar, ikke en fejl.***

## Lagets histogram

Under markørrækkerne vises et live-histogram for det lag, du ser på, i **256 intervaller**. Som standard tegnes der én samlet kurve, vægtet `(R + 2G + B) / 4` — det samme målerum, som LATTICE-kamerahistogrammerne bruger. Hvis du aktiverer**RGB**, erstattes den med kurver pr. kanal i kanalfarverne, der blandes additivt, så overlapninger forbliver læselige. Monolag tegner altid den enkelte kurve.

Den vandrette akse er i lagets egen enhed:

| Lag       | Aksenhed  | Aksens maksimum                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Reflektans | procent    | 125 % — produktets headroom tillader ρ over 1,0           |
| Strålingsintensitet | W/m²/sr/nm | Rammens egen spidsværdi, afrundet op til to betydende cifre |
| 8-bit data  | DN         | 255                                                        |
| 12-bit data | DN         | 4095                                                       |
| 16-bit data | DN         | 65535                                                      |

Når aksen er i DN og lander på et af disse tre loftværdier, kender Chloros også bitdybden for det, du ser på.

Der er tre knapper over histogrammet:

| Knap     | Standard | Effekt                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CURSOR** | Til      | Tegner markeringslinjer på histogrammet ved de nøjagtige værdier, der vises i rækkerne ovenfor, så du kan se, hvor pixlen under din markør befinder sig i billedets fordeling. I RGB-tilstand er der én markør pr. kanal i sin egen farve; ellers en enkelt hvid markør ved den samlede værdi |
| **INDEX**| Til      | Vises kun, mens et indeks er aktivt. Skifter histogrammet fra kildebåndene til**indeksværdifordelingen**, hvor de to klip-tærskler er tegnet som orange stiplede linjer, og markørens indeksværdi som en hvid linje                                                          |
| **RGB**| Fra     | Skifter fra den samlede kurve til kurver pr. kanal. På en monosensor viser denne knap**MONO** og er deaktiveret — der er kun én kanal at vise                                                                                                                                  |

Histogrammet beregnes ud fra de **blokke, du kan se**, ikke de underliggende kildepixels: ændrer du GSD-blokstørrelsen, beregnes fordelingen på ny, så histogrammet, markøren og det viste billede altid stemmer overens.***

## GSD-blokstørrelse

Nederst på panelet findes **GSD (px)**-kontrollen: et talfelt, en skyder fra**1 til 256**og en**RESET**-knap.

Den gør det _viste_ billede grovere ved at beregne gennemsnittet af en N × N-blok af kildepixels til én vist pixel. `1` er den oprindelige opløsning.

* Det påvirker **fuldskærmsvisningen, gitterminiaturerne, markørvisningen og begge histogrammer** — alt, der viser billedet, er baseret på den samme grundopløsning.
* Det gælder **kun visningen**. Behandling og eksport påvirkes ikke. Den eneste undtagelse er bevidst: en [Index/LUT Sandbox](index-lut-sandbox.md)-eksport gemmer det, du ser, så den overfører den aktuelle blokstørrelse, og eksportpanelet advarer dig, når blokstørrelsen er over 1.
* Værdien gemmes **pr. projekt** som `viewer_display.gsd_bin` i `project.json`, så den bevares, selvom programmet lukkes og genåbnes.
* Cursorvisningen angiver blokken, ikke kildepixlen, når blokstørrelsen er over 1 — den viste værdi er gennemsnittet af blokken under din cursor.

{% hint style="info" %}
**Hvorfor »blokstørrelse« og ikke centimeter pr. pixel?** Et tal i cm/px kræver en højde over jordoverfladen. Et enkelt billedesEXIF-data indeholder GPS-højde over middelvandstanden, ikke over det terræn, den var rettet mod, så Chloros vil ikke udskrive en afstand til jordoverfladen, som den ikke kan beregne korrekt. Blokstørrelsen i kildepixels er den samme reserve, som MAPIR-skyværktøjerne bruger , når jordprøveafstanden er ukendt.
{% endhint %}

***

## Billedtyper, du kan se

Lag-rullemenuen øverst til højre i visningsvinduet viser alle versioner af det aktuelle billede. Hvilke poster der vises, afhænger af kameraet og af, hvad der er blevet behandlet — se [Billedlag](image-layers.md) for den fulde liste og for at se, hvordan rullemenuen fungerer.

### Survey3

* **JPG** — kameraets egen forhåndsvisningsfil
* **RAW (Original)** — kildefilen `.RAW`, debayered til visning, ingen korrektioner
* **RAW (Target)** — et billede, der er identificeret som indeholdende et kalibreringsmål
* **RAW (reflektans)** — det kalibrerede reflektansprodukt (65535 = ρ 1,0)
* **Vignettekorrigeret**/**Sensorreaktion** — det ukalibrerede reserveprodukt
* **Hvidbalanceret** — det hvidbalancerede produkt
* **RAW (`<INDEX>`-indeks)**og**`<INDEX>` LUT** — beregnede indeksbilleder

### LATTICE

LATTICE-optagelser bruger den samme rullemenu med navne på pipeline-niveauerne:

| Lag                 | Indhold                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (Original)**    | Den oprindelige RAW-ramme, som den blev optaget                                     |
| **RAW (Debayered)**   | Det lineære debayered-billede                                           |
| **RAW (Preview)**     | Visningsforhåndsvisningen — strækning med falske farver til multispektrale kameraer |
| **Hvidbalance**    | Visningsforhåndsvisningen for RGB-masterkameraer (hvidbalance + gamma)   |
| **RAW (Radiance)**    | Float32-spektralradiance i W/m²/sr/nm                              |
| **RAW (reflektans)** | uint16-reflektans, 32768 = ρ 1,0                                    |

Stråling og reflektans findes kun i multispektralformat: Et RGB-masterkamera har ingen radiometri pr. bånd, så disse lag genereres ikke for det.

***

## Anvendelse af indeks og LUT

Anvend multispektrale indekser og farve-LUT&#x27;er fra sidepanelet:

1. Åbn **Image Viewer**<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">-sidepanelet
2. Marker **Index**

3. Vælg dit kameras filter og en indeksformel, og træk derefter kanalkredsene ind i formelens felter
4. Tilføj en LUT, og vælg en gradient, tærskelværdier og en beskæringsindstilling
5. Se værdierne ved markøren, og gem resultatet med **Eksporter/Gem billede(r)**Se [Index/LUT Sandbox](index-lut-sandbox.md) for den fulde vejledning.***

## Fejlfinding

### Billedet kan ikke åbnes

**Mulige årsager**: Filen blev flyttet eller slettet efter import; produktet blev aldrig skrevet; der er ikke nok hukommelse til et meget stort billede.**Hvad skal du gøre**:

1. Kontroller, at lagets fil stadig findes i projektets output-struktur
2. Åbn filen i et eksternt visningsprogram for at bekræfte, at den er intakt
3. Luk andre programmer for at frigøre hukommelse

### Billedet er sort, hvidt eller har vilde farver

**Mulige årsager**: skærmstrækningen har intet at arbejde med (en næsten konstant ramme); et float32-lag med usædvanlige værdier; et indeks, der ikke genererede gyldige data.**Hvad skal du gøre**:

1. Læs cursorværdierne — hvis alle kanaler ligger på eller tæt på nul, ligger problemet i dataene, ikke i visningen
2. Tjek histogrammet: en enkelt spids i den ene ende indikerer, at rammen er beskåret eller tom
3. Tjek behandlingsloggen for den kørsel, der genererede laget

### Værdierne ser forkerte ud

**Mulige årsager**: du befinder dig på et andet lag, end du tror; du sammenligner en procentdel med en rå DN; du sammenligner en LATTICE-fil med en Survey3-fil ved hjælp af den samme divider.**Hvad skal du gøre**:

1. Bekræft det valgte lag i rullemenuen — panelets enheder følger laget
2. For reflektans skal du bruge **%**-kolonnen i stedet for selv at dividere DN; hvis du skal dividere, skal du bruge den pågældende fils `Chloros:PixelScale` (32768 for LATTICE; hvis den mangler, betyder det 65535 for Survey3)
3. Indstil GSD-blokstørrelsen tilbage til 1 — over 1 aflæser du et blokgennemsnit, ikke en pixel
4. Kontroller, at reflektanskalibreringen faktisk blev kørt for det pågældende billede; et ukalibreret reserveprodukt (Sensor Response / Vignette Corrected) er ikke reflektans

***

## Næste trin

* [**Billedlag**](image-layers.md) — hvert lags navn, når det findes, og hvad dets værdier betyder
* [**Indeks/LUT-sandkasse**](index-lut-sandbox.md) — opbyg, finjuster og eksporter indeksvisualiseringer
* [**Kortmarkører**](map-markers.md) — det samme billedsæt på et kort
* [**Multispektrale indeksformler**](../project-settings/multispectral-index-formulas.md) — indeksreferencen

Se [Behandling af billeder (GUI)](../processing-images-gui/adding-files-to-a-project.md) for arbejdsgangen ved behandling.

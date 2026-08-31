# Billedgitter

Når du har importeret billeder til et projekt, vil du se dem anbragt i et gitter i hovedområdet. Det er i gitteret, du vælger, **hvilken version af hvert billede du ser på** — knapperne oven over gitteret skifter alle miniaturer på én gang mellem kildefilerne og de enkelte bearbejdede resultater.

## Miniaturstørrelse

Brug zoomskyderen øverst til højre til at justere størrelsen på billedminiaturerne. Skyderen går fra **64 px til 1200 px**.

* **Ctrl + musehjul** ændrer også størrelsen på miniaturerne.
* **Ctrl + `+`**/**Ctrl + `=`**og**Ctrl + `−`** ændrer størrelsen med 4 px pr. tryk. Tastaturgenvejen stopper ved 64 px i den lille ende og, i den store ende, ved den størrelse, der passer nøjagtigt til to miniaturer pr. række i det aktuelle vindue.
* Den størrelse, du vælger, gemmes sammen med projektet (`UI → Grid thumbnail size` i `project.json`, standard `160`), så den gendannes, når projektet åbnes igen.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>Miniaturernes *opløsning* er en separat indstilling fra miniaturernes *størrelse*: se **Visning → Opløsning af billedminiaturer** i [Projektindstillinger](../project-settings/project-settings.md) (standard 512 px på den lange side). Størrelsen angiver, hvor stor flisen tegnes; opløsningen angiver, hvor mange detaljer der hentes for at udfylde den.***

## Værktøjslinjen til gitteret

Rækken af knapper over gitteret har op til tre grupper, fra venstre mod højre:

1. **Per Trigger / Per Kamera** — grupperingsfunktion. Vises kun for projekter, der indeholder LATTICE-optagelser.
2. **Kamerafilterknapper** — én pr. LATTICE-kamera. Vises kun i Per Kamera-tilstand.
3. **Knapper til eksport-/visningstilstand** — hvilket produkt hver miniature viser.

Når vinduet er for smalt til at rumme dem alle, foldes grupperne sammen fra højre mod venstre til dropdown-menuer, der vises ved musepeger: eksport-/visningsknapperne foldes først sammen, derefter kameraknapperne. Den sammenklappede gruppe efterlader en enkelt knap mærket med det aktuelt aktive valg, og når man holder markøren over den, glider hele sætket ned. **&quot;Per Trigger / Per Camera&quot; klappes aldrig sammen.**<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->***

## Knapper til eksportvisning

Disse knapper skifter mellem billedtyperne i gitteret med miniaturer. **En knap vises, så snart det produkt, den navngiver, findes** — hvilket for kildefilerne betyder straks ved import, ikke efter behandling.Chloros

genindlæser projektets produkter, mens en kørsel er i gang, så knapperne vises under behandlingen, efterhånden som hvert produkt begynder at blive gemt på disken.

### Grundknappen

Den venstre eksportknap er mærket efter **det, du rent faktisk har importeret**:

| Hvad du har importeret | Knaptekst |
| --- | --- |
|Survey3

RAW+JPG | `JPG` |
| LATTICE-optagelser med en forhåndsvisning ved siden af RAW-billedet | `PNG` eller `TIFF`, afhængigt af hvilke forhåndsvisninger der er |
| LATTICE-optagelser, hvor basisfilen **er** RAW-billedet | *ingen knap* — `RAW (Original)` viser allerede den pågældende fil |

I et blandet projekt følger etiketten den filtypenavn, som flest billeder bruger.

### Produktknapper

| Knap | Viser | Når den vises |
| --- | --- | --- |
| **Mål** | Billeder med et registreret kalibreringsmål | Efter en kørsel, der har registreret mål |
| **Reflektans** | Kalibrerede reflektansbilleder | Kun iSurvey3

-projekter — LATTICE-projekter bruger i stedet `RAW (Reflectance)`, så gitteret viser aldrig to reflektansknapper |
| **Hvidbalanceret** | Det hvidbalancerede produkt (kameraer fraRGB

) | Efter behandling |
| **Vignettekorrigeret** | Den ukalibrerede vignettekorrigerede standardindstilling | Efter en kørsel, hvor reflektanskalibrering ikke kunne anvendes, og *Vignettekorrektion* var aktiveret |
| **Sensorreaktion** | Den ukalibrerede fallback med sensorreaktion | Det samme, men med *Vignettekorrektion* slået fra |
| **`RAW (<INDEX> Index)`** | Én knap pr. beregnet indeks | Efter en kørsel med konfigurerede indekser |
| **`<INDEX> LUT`** | Én knap pr. farvekortlagt indeks | Efter en kørsel med en konfigureret LUT |
| **`<Index> <Index\|LUT> <NNN>`** | Én knap pr. [Indeks/LUT-sandkasse](index-lut-sandbox.md)-eksportkørsel | I det øjeblik en sandkasseeksport er afsluttet |

### Knapper på LATTICE-niveau

Projekter, der indeholder LATTICE-optagelser, tilføjer disse, mærket med niveauets navn i stedet for et produktnavn:

| Knap | Niveau |
| --- | --- |
| **RAW (Original)** | Den importerede råramme |
| **RAW (Radiance)** | Float32 spektral radiance, W/m²/sr/nm |
| **RAW (Reflectance)** | uint16 reflektans, 32768 = ρ 1,0 |

`RAW (Original)` er tilgængeligt fra importøjeblikket — det kræver ingen behandling. Når en LATTICE-import slet ikke har nogen basis-knap (hver optagelses basisfil er dens råramme), flytter gitteret sig selv til den første tilgængelige niveauknap, så markeringen på værktøjslinjen stemmer overens med det, du ser.

To niveauer afChloros

-eksportfiler får **ingen egen gitterknap**:

* **Debayered** — `RAW (Original)`-visningen gengives allerede debayered, så en ekstra knap på et visuelt identisk billede ville være overflødig. `RAW (Debayered)`-produktet skrives stadig til disken og kan stadig vælges fra dropdown-menuen for fuldskærmslag.
* **Preview** — påRGB

-kameraer registreres previewet som `White Balanced`-laget, som har en knap. På multispektrale kameraer registreres det som `RAW (Preview)` og kan tilgås via rullemenuen for fuldskærmslag.

{% hint style="info" %}
Disse niveauknapper vises kun for projekter, der rent faktisk indeholder LATTICE-rammer. »Survey3

«-projekter registrerer nogle af de samme interne lag navne, og knapperne filtreres fra for disse, så et »Survey3

«-gitter bevarer sit velkendte `JPG / Targets / Reflectance`-sæt.
{% endhint %}

Hvis du klikker på en gitterminiature, åbnes [Billedviseren](opening-an-image-full-screen.md) i fuld skærm på **det samme produkt, som gitteret viser** — hvis gitteret er indstillet til `Targets`, åbner miniaturen det eksporterede målbillede.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->


***

## Gruppering af et LATTICE-projekt: pr. trigger vs. pr. kamera

Array-optagelser producerer flere billeder af det samme øjeblik fra forskellige kameramoduler. Grupperingen bestemmer, hvordan gitteret stabler dem. Begge tilstande viser sammenklappelige overskriftsbjælker i fuld bredde; **hver gruppe starter udfoldet**, ogChloros
husker dem, du lukker. Tilstanden for sammenklapning spores separat for hver tilstand, så hvis du lukker en gruppe i »Per kamera«, lukkes der ikke noget i »Per udløser«.

### Per kamera (standard)

Én gruppe pr. kameramodul. Overskriften viser kameramodellen og serienummeret (`LATT-M3M — <serial>`) samt antallet af billeder. Billederne i en gruppe er sorteret kronologisk efter optagelsestidspunkt.

I denne tilstand får værktøjslinjen også én **kamerafilterknap pr. kamera**, mærket `MODEL (SERIAL)`. Alle kameraer er valgt fra starten; ved at klikke på en knap fravælges det pågældende kamera, og dets gruppe fjernes fra rutenettet. Dette er den hurtige måde at gennemgå et bånd på tværs af en hel flyvning.

### Pr. udløser

Én gruppe pr. optagelseshændelse — det sæt billeder, som alle modulerne har taget ved den samme udløser. Overskriften viser optagelsestidspunktet, antallet af kameraer, der har bidraget, samt et mærke for hver kameramodel i gruppen. Brikkerne inden for en gruppe er sorteret efter kameraets serienummer, så det samme bånd ligger i samme kolonne for hver udløsning.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
Ikke-LATTICE-billeder i et blandet projekt grupperes ikke — de vises som almindelige brikker efter grupperne.

***

## Miniaturebillederne i rasteret følger GSD-blokstørrelsen

Hvis du har indstillet en **GSD (px)**-blokstørrelse i sidepanelet på fanen »Billede«, vises gitterminiaturerne i samme opløsning – ikke kun i fuldskærmvisningen. En blokstørrelse på 8 betyder, at hver viste pixel er gennemsnittet af en 8 × 8-blok af kildepixels, overalt i appen, hvor billedet vises.

Da en flise i forvejen kun er et par hundrede piksler bred, holder grove blokstørrelser op med at udgøre en synlig forskel i gitteret længe før, de gør det i fuldskærmsvisningen: en ramme på 4000 px tegnet ind i en flise på 160 px har allerede ca. 25 kildepiksler pr. vist piksel. Se [Åbning af et billede i fuldskærm](opening-an-image-full-screen.md#gsd-block-size) for selve kontrolelementet.

***

## Relaterede sider

* [**Åbning af et billede i fuld skærm**](opening-an-image-full-screen.md) — fuldskærmviseren, markørværdier og histogram
* [**Billedlag**](image-layers.md) — rullemenuen med lag i fuldskærmsviseren
* [**Indeks/LUT-sandkasse**](index-lut-sandbox.md) — opbygning og eksport af indeksvisualiseringer
* [**Projektindstillinger**](../project-settings/project-settings.md) — eksportknapperne, der bestemmer, hvilke produkter der overhovedet findes

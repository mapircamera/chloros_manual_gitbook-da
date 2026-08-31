# Optagelsesindstillinger og -tilstande

Optagelse under fanen »Kameraer« styres via en rød **Capture All**-knap og et**Capture Settings**-vindue, der bestemmer, hvad knappen udløser: hvilke kameraer der deltager, hvilke eksportformater hvert kamera gemmer, og om lukkeren udløses én gang, kontinuerligt eller med et bestemt interval. Denne side beskriver hele forløbet — konfiguration, selve optagelsen, hvor filerne gemmes på disken, og hvordan man senere kan genbehandle dem til kalibrerede produkter. Selve kamera- og array-kontrollerne findes under [Kameraindstillinger](camera-settings.md).

{% hint style="info" %}
**Optagelser kræver, at der er et åbent projekt.** &quot;Optag alt&quot; og tandhjulet til &quot;Optagelsesindstillinger&quot; er deaktiveret, indtil der er åbnet et projekt (&quot;Opret eller åbn et projekt for at gemme optagelser&quot;). Hver optagelse gemmes i projektmappen i `captures/`.
{% endhint %}

## Panelet &quot;Optagelsesindstillinger&quot;

Åbn det med **tandhjulet ved siden af &quot;Optag alt&quot;**i kameralisten i sidepanelet eller med knappen**&quot;Åbn optagelsesindstillinger…&quot;** nederst i ethvert kameraspecifikt indstillingspanel. Overskriften lyder &quot;Optagelsesindstillinger&quot; med en ←-tilbage-knap.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Dine valg her — inkluderede kameraer, afkrydsningsfelter pr. type og optagelsestilstand — gemmes **pr. projekt** og gendannes, når du åbner det igen.

### Optagelsestilstande

Tre tilstandsknapper øverst i vinduet:

| Tilstand | Funktion | Underinstillinger (standard) |
| --- | --- | --- |
| **Enkelt** *(standard)* | Én optagelse på tværs af alle valgte kameraer. | — |
| **Kontinuerlig**| Optagelser i træk, indtil en stopbetingelse opfyldes. | Stop ved**Antal optagelser** (standard 1) *eller* **Optagelsens varighed** (standard 10 s; enheder: sekunder / minutter / timer / dage). |
| **Interval**(timelapse) | Serier efter en timer. |**Optagelser / interval**(standard 1) ·**Hver**N enheder (standard 5 s) ·**I** N enheder (standard 1 m). |

I kontinuerlig eller interval-tilstand bliver knappen »Optag alt« til en **Stop (N)**-knap under kørsel, hvor optagelserne tælles, efterhånden som de kommer ind.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Valg af kameraer og eksporttyper

Hjælpeteksten i ruden opsummerer det: Vælg, hvilke kameraer og eksporttyper &quot;Capture All&quot; skal generere — alt er aktiveret som standard, og valgene gemmes sammen med dette projekt.

* Knapperne **Vælg alle / Vælg ingen** skifter alle kameraers afkrydsningsfelter på én gang.
* **Vælgere til masseeksporttyper**(to rækker med knapper):**All Raw / All Debayered / All Preview / All Radiance / All Reflectance / All Index**. Hver knap har tre farvestadier: grøn ✓ = aktiveret for alle kameraer, der understøtter den, gul – = aktiveret for nogle, grå = ingen. En knap er deaktiveret, når intet tilsluttet kamera understøtter den pågældende type. Alle knapper bliver grå, når &quot;Hurtigst mulig optagelse&quot; er aktiveret.
* **Rækker pr. kamera**: et afkrydsningsfelt til inkludering samt en udvidelig (▸/▾) liste over det pågældende kameras anvendelige eksporttyper med individuelle afkrydsningsfelter. Rækken viser et antal som &quot;4/6&quot;.

### Eksporttyper og hvilke kameraer der understøtter dem

Der findes seks eksporttyper: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Kun de relevante vises i hvert kameras række:

| Eksporttype | Indhold | RGB (FRGB) | Bayer-multispektral (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayer-mosaik (mono: det enkelte bånd) direkte fra sensoren | ✓ | ✓ | ✓ |
| **Debayered** | Lineær demosaik (mono: 1-kanals gråskala) | ✓ | ✓ | ✓ |
| **Forhåndsvisning** | Fuld visningskæde (hvidbalance + gamma i henhold til kameraets profil; multispektral: falskfarveudstrækning) | ✓ | ✓ | ✓ |
| **Strålingsintensitet** | float32 W/m²/sr/nm via den fulde radiometriske kæde | — (tilbydes ikke) | ✓ | ✓ |
| **Reflektans** | uint16 ρ (32768 = 1,0) | — (tilbydes ikke) | ✓ — vises kun, når kameraet har en DAQ-lyssensor (sin egen eller arvet fra sit array) | samme som multispektral |
| **Indeks** | Vegetationsindeks (LUT)-gengivelse | — | ✓ — kræver et aktiveret, ikke-tomt indeksudtryk på kameraet og tilbydes ikke til medlemmer af kombinerede arrays (arrayet har ét fælles indeks) | — (et indeks kræver ≥2 bånd; se [Monokameraer og vegetationsindekser](mono-indices.md)) |

Strålingsintensitet og reflektans tilbydes aldrig for RGB-kameraer — strålingsintensitet pr. Bayer-pixel er ikke meningsfuld for en bredbåndsfotometrisk sensor.

### Hurtigst mulig optagelse

Vippeknappen **⚡ Hurtigste optagelse — kun RAW**(orange, når den er aktiveret) tilsidesætter alle eksportvalg til**kun RAW** — plus en gratis sammensat billedkomposit med kombineret indeks for arrayer — så billedet gemmes så hurtigt som muligt: beregningerne af radiance, reflektans og visning springes helt over ved optagelsestidspunktet.

{% hint style="info" %}
**En `.daq` gemmes stadig.**Når der er tildelt en lyssensor, skriver**Hurtigst mulig optagelse** stadig DAQ-nedstrålingsmålingen ved siden af de rå billeder — så strålings-, reflektans- og indeksprodukter alle kan opbygges senere ved genbehandling (se [Genbehandling af optagelser](#re-processing-captures-into-calibrated-products)). Fastest Capture påvirker heller ikke dine valg i afkrydsningsfelterne: Slå funktionen fra, så kommer de tilbage.
{% endhint %}

### Kontrolelementer pr. array

Hvert tilsluttet array får sit eget gruppekort i ruden:

* **Afkrydsningsfeltet &quot;Include&quot;** (tre tilstande på tværs af medlemmer) og arrayets navn med dets visningsmodus: &quot;(combined | separate)&quot;.
* **Justeret**-afkrydsningsfelt (standard**til**): tilpasser eksporten af medlemmer til arrayets justeringsprofil, så eksporten er pixelregistreret på tværs af kameraer. Raw-data forbliver uforvrænget, men bærer transformationen i sine metadata. (Selve profilen beregnes i [panelet med arrayindstillinger](camera-settings.md#alignment-co-registration-combined-only).)
* Kamerarækkerne for de enkelte medlemmer er indlejret i kortet.

Arraykortet indeholder også to optagere. Tænk på dem som **overvågning kontra analyse**:

| Optager | Grad | Hvad den optager |
| --- | --- | --- |
| **● Optag indeksvideo / ■ Stop optagelse** *(kun kombinerede arrays)* | **Overvågning** | Den kombinerede live-indekskomposit til video ved 10 fps — 8-bit, forhåndsvisningsopløsning, indbygget LUT. Kræver et åbent projekt og en streamet live-visning. Viser billeder + forløbet tid under optagelse. |
| **⦿ Optag rå burst / ■ Stop rå burst** *(alle arrayer)* | **Analyse**| Rå Bayer-billeder med live-optagelseshastighed (ingen behandling) plus et manifest pr. billede og `.daq`-målinger, gemt som `captures/bursts/`. Efter en serie vises en**Opret video**-knap: den genbehandler serien offline til kalibreret video — kombineret indeks og/eller stråling/refleksion/indeks pr. kamera — plus valgfrie TIFF-filer. Oprettelsen af den kombinerede indeks starter automatisk, når du stopper serien.

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

|## »Capture All«-arbejdsgangen

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

Tryk på **Capture All** i kameralisten i sidepanelet:

1. Alle inkluderede, synlige kameraer, der ikke er sat på pause, optager med de valgte eksporttyper. **Arrayer udløses som én synkroniseret trigger** (en enkelt synkroniseret gruppe på tværs af alle medlemmer — se [Multi-Camera Arrays](arrays.md)); enkeltstående kameraer optager individuelt.
2. Skjulte (øje) eller pausede kameraer springes over. Et array er kun fuldt blokeret, når *alle* dets medlemmer er skjulte eller pausede.
3. Når der er tildelt en lyssensor, gemmes den tilsvarende DAQ-nedstrålingsmåling som en `.daq`-fil sammen med billedmaterialet — selv ved optagelser, der kun består af rådata — så der altid senere kan udledes radiometriske produkter.
4. Knappen viser fremskridt i realtid — »Optager… færdig/i alt« — og bliver til **Stop (N)** i kontinuerlig/interval-tilstand. Hvert optagelseselement har en timeout på 300 s.
5. Når optagelsen er færdig, vises en resultatmeddelelse med **»N filer gemt«**eller**»N gemt, F mislykkedes«** samt »(S skjult/sat på pause/sprunget over)«, når kameraer er blevet sprunget over.

## Hvor optagelserne gemmes

Optagelserne gemmes under det åbne projekt i `<project>/captures/`. Hver eksporttype gemmes i sin **egen undermappe**, så en optagelse på flere niveauer aldrig blander typerne:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` er optagelsens tidsstempel, og `<serial>` er kameraets serienummer. Fritstående optagelser navngives `capture_<ts>_SN<serial>_<level>`; array-optagelser fra én synkroniseret udløser navngives `sync_<ts>_SN<serial>_<level>` og **deler ét tidsstempel på tværs af alle kameraer i gruppen** (niveau-suffikset fjernes, når et kamera kun gemmer et enkelt niveau).
* **En uregelmæssighed, man skal være opmærksom på:** displayniveauet gemmes i en mappe med navnet `preview/`, mens filerne beholder `_display` i navnet — mappe og suffiks adskiller sig kun for dette niveau.
* Ukendte niveauer gemmes i en mappe med deres eget navn; hvis der ikke kan oprettes en undermappe, skrives filen til rodmappen for optagelser i stedet for at gå tabt.
* Optagelses-TIFF-filer komprimeres som standard tabsfrit (DEFLATE) og indeholder alle metadata om kalibrering og behandling **inde i filens XMP** — optagelserne er selvbeskrivende og har ingen sidecar-filer ud over den, der har navnet `.daq`.

Dette er det samme layout, som `chloros-cli lattice capture` / `array-capture` skriver til deres `-o`-mappe — dokumenteret i [CLI-reference § Sådan ser en captures-mappe ud](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Genbehandling af optagelser til kalibrerede produkter

Indfangede råbilleder samt den gemte `.daq` er alt, hvad behandlingsrørledningen har brug for — derfor er Fastest Capture sikkert at bruge til egentligt arbejde.

* **GUI**: Føj mappen med optagelser til et projekt ([Tilføjelse af filer til et projekt](../processing-images-gui/adding-files-to-a-project.md)) og behandl som normalt.
* **CLI**: peg `process` mod**optagelsernes rodmappe**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` importerer normalt kun den mappe, du angiver, men når den pågældende mappe ikke indeholder billeder og har undermapper, går programmet automatisk ned i hierarkiet — så undermapperne på det pågældende niveau og *root*-filerne i `.daq` hentes på én gang. Hver optagelse importeres som et **enkelt billede**, hvor de øvrige niveauer er knyttet som visningsmodi, ikke som ét billede pr. niveau.

Det fungerer også at navngive en undermappe på et niveau direkte (f.eks. `…/captures/raw/`), men efterlader rodfilerne `.daq` – kopier dem sammen med, når du genberegner et radiometrisk produkt fra `raw/`, ellers har tidsstemplet intet at matche mod.

{% hint style="warning" %}
**Behandlingen starter altid fra `raw`.**Inden for hver optagelse er råbilledet kilden til behandlingskæden; `debayered`, `radiance`, `reflectance` og `preview` følger med som visningsformater, men føres aldrig tilbage gennem processen — genbehandling af et afledt produkt ville medføre, at vignettering, farve og strålingsberegninger, der allerede er indbagt i dets pixels, blev anvendt på ny, hvorfor Chloros afvises i stedet for at blive behandlet to gange. Renderingerne `index/` og `composite/` behandles slet ikke (de er output, ikke optagelser). En »captures«-mappe, der er gemt**uden** RAW-import, vises normalt, men `process` springer den over og angiver dette; `--input-level {raw,debayered,processed}` er den bevidste nødudgang, der tvinger et indgangssted frem. Se [CLI-referencen](../reference/cli-reference.md#what-a-captures-folder-looks-like) for de nøjagtige spring-over-meddelelser.
{% endhint %}

To yderligere adfærdsmønstre, der er værd at kende, når man skriver scripts til genbehandling:

* En `chloros-cli process`-kørsel, der anmodede om produkter, men ikke skrev **nogen billedprodukter, mislykkes med en tydelig fejlmeddelelse og afsluttes med en værdi forskellig fra nul** — du vil aldrig få en lydløs tom kørsel. Vellykkede kørsler rapporterer deres produktantal. (En bevidst kørsel, der kun omfatter metadata, tæller stadig som en succes.)
* Genimporterede, forarbejdede eksporter indtager aldrig en optagelses rådata-slot — den oprindelige rådata forbliver altid kilden til pipelinen.

## CLI-ækvivalenter

Alt på denne side kan køres uden grafisk brugergrænseflade. GUI-optagelsestilstandene svarer direkte til `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Enkelt | `chloros-cli lattice array-capture` |
| Kontinuerlig | `array-capture --continuous [--count N] [--duration S]` |
| Interval | `array-capture --interval S [--duration S]` |
| Hurtigste optagelse | `array-capture --fastest` |
| Justeret afkrydsningsfelt | `--aligned / --no-aligned` |
| Afkrydsningsfelter for eksporttype | `--processing LEVEL` eller `--levels L1,L2,…` (standard `all`) |
| Optag indeksvideo | `chloros-cli lattice array-record` |
| Optag rå burst / Lav video | `chloros-cli lattice array-burst` / `array-build-video` |

Fuldstændige flagtabeller, indstillingen smart-AE settled-capture (`--smart`) og modellen med konstant hastighed findes i [CLI Reference § Optagelsestilstande, optagere og offline-genbehandling](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

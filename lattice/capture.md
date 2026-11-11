# Optagelsesindstillinger og -tilstande

Optagelse under fanen »Kameraer« styres via en rød **Capture All**-knap og et**Capture Settings**-vindue, der bestemmer, hvad knappen udløser: hvilke kameraer der deltager, hvilke eksportformater hvert kamera gemmer, og om lukkeren udløses én gang, kontinuerligt eller med et bestemt interval. Denne side beskriver hele forløbet — konfiguration, selve optagelsen, hvor filerne gemmes på disken, og hvordan man senere kan bearbejde dem til kalibrerede produkter. Selve kamera- og array-kontrollerne findes under [Kameraindstillinger](camera-settings.md).

{% hint style="info" %}
**Optagelser kræver, at der er et åbent projekt.** ”Optag alt” og tandhjulet til optagelsesindstillinger er deaktiveret, indtil der er åbnet et projekt (“Opret eller åbn et projekt for at gemme optagelser”). Hver optagelse gemmes i projektmappen i `captures/`.
{% endhint %}

## Panelet "Optagelsesindstillinger"

Åbn det med **tandhjulet ved siden af "Optag alt"**i kameralisten i sidepanelet eller med knappen**"Åbn optagelsesindstillinger…"** nederst i ethvert kameraspecifikt indstillingspanel. Overskriften lyder "Optagelsesindstillinger" med en ←-tilbage-knap.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Dine valg her – inkluderede kameraer, afkrydsningsfelter pr. type og optagelsestilstand – gemmes **pr. projekt** og gendannes, når du åbner det igen.

### Optagelsestilstande

Tre tilstandsknapper øverst i panelet:

| Tilstand | Funktion | Underinstillinger (standard) |
| --- | --- | --- |
| **Enkelt** *(standard)* | Én optagelse på tværs af alle valgte kameraer. | — |
| **Kontinuerlig**| Optagelser i træk, indtil en stopbetingelse opfyldes. | Stop ved**antal optagelser** (standard 1) *eller* **optagelsestid** (standard 10 s; enheder: sekunder / minutter / timer / dage). |
| **Interval**(timelapse) | Optagelser i serier efter en timer. |**Optagelser pr. interval**(standard 1) ·**Hver**N enheder (standard 5 s) ·**I** N enheder (standard 1 m). |

I tilstanden Kontinuerlig eller Interval ændres knappen Optag alt til en **Stop (N)**-knap under kørsel, hvor optagelserne tælles, efterhånden som de kommer ind. Et fuldt drev stopper kørslen (se ["Capture All"-forløbet](#the-capture-all-flow)).**Kontinuerlig på arrays kører inde i backend'en (nyt i 1.3.0).** Når alle kameraer på listen tilhører et array (fritstående kameraer er skjult eller udelukket), sender "Kontinuerlig" ikke længere én anmodning pr. optagelse: Chloros-backendet kører selv optagelsessløjfen, hvor det skriver én synkroniseret gruppe, mens det vælger den næste, og knappen aflæser antallet. Med fritstående kameraer på listen foretager Chloros optagelser pass for pass som før.

{% hint style="warning" %}
**Hurtig optagelse på Windows: Sørg for, at Chloros-backend er undtaget fra Microsoft Defender.** Realtidsscanning af hver fil, som en optagelse med høj hastighed skriver, koster ca. 30 % af optagelseshastigheden i MAPIR’s tests (hundredvis af filer pr. sekund ved et lille ROI). Windows-installationsprogrammet tilbyder **"Undtag Chloros-backend fra Microsoft Defender-scanning (meget hurtigere kameraoptagelse; anbefales)"**, som er markeret som standard, og fjerner undtagelsen ved afinstallation. Hvis du har fjernet markeringen, skal du køre installationsprogrammet igen med markeringen aktiveret eller tilføje en undtagelse for Defender-processen for `chloros-backend.exe` (i mappen `backend` i Chloros-installationsmappen). Når Defender bruger en eller flere CPU-kerner under en arrays kontinuerlige optagelse, noterer Chloros-backend det én gang i sin log med disse to samme løsninger.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Valg af kameraer og eksporttyper

Hjælpeteksten i ruden opsummerer det: Vælg, hvilke kameraer og eksporttyper »Capture All« skal generere — alt er aktiveret som standard, og valgene gemmes sammen med dette projekt.

* **Vælg alle / Vælg ingen**-knapperne skifter alle kameraernes afkrydsningsfelter på én gang.
* **Knapper til massevalg af eksporttyper**(to rækker med knapper):**Alle Raw / Alle Debayered / Alle Preview / Alle Radiance / Alle Reflectance / Alle Index**. Hver har tre farvestadier: grøn ✓ = aktiveret for alle kameraer, der understøtter det, gul – = aktiveret for nogle, grå = ingen. En knap er deaktiveret, når ingen tilsluttede kameraer understøtter den pågældende type. Alle knapper bliver grå, når *Fastest Capture* er aktiveret.
* **Rækker pr. kamera**: et afkrydsningsfelt samt en udvidelig (▸/▾) liste over det pågældende kameras anvendelige eksporttyper med individuelle afkrydsningsfelter. Rækken viser et antal, der angiver, hvor mange der er aktiveret, f.eks. »4/6«.

### Eksporttyper og hvilke kameraer der understøtter dem

Der findes seks eksporttyper: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Kun de relevante vises i rækken for hvert kamera:

| Eksporttype | Indhold | RGB (FRGB) | Bayer-multispektral (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayer-mosaik (mono: det enkelte bånd) direkte fra sensoren | ✓ | ✓ | ✓ |
| **Debayered** | Lineær demosaik: kalibreringspakkens mørkeforskydning (DSNU) fjernet fra RAW-filen, efterfulgt af en kantbevidst demosaik — ingen linearitetskorrektion, flat field eller unmix. Gemt som en 3-kanals RGB-TIFF-fil (mono: 1-kanals gråtoner) | ✓ | ✓ | ✓ |
| **Forhåndsvisning** | Fuld visningskæde (hvidbalance + gamma i henhold til kameraets profil samt RGB-kameraets indstilling for højlys; multispektral: falskfarvestrækning) | ✓ | ✓ | ✓ |
| **Strålingsintensitet** | float32 W/m²/sr/nm via den fulde radiometriske kæde | — (tilbydes ikke) | ✓ | ✓ |
| **Reflektans** | uint16 ρ (32768 = 1,0) | — (tilbydes ikke) | ✓ — vises kun, når kameraet har en DAQ-lyssensor (sin egen eller arvet fra sit array) | det samme som multispektral |
| **Indeks** | Vegetationsindeks (LUT)-gengivelse | — | ✓ — kræver et aktiveret, ikke-tomt indeksudtryk på kameraet og tilbydes ikke til medlemmer af kombinerede arrays (arrayet har ét fælles indeks) | — (et indeks kræver ≥2 bånd; se [Monokameraer og vegetationsindekser](mono-indices.md)) |

Strålingsintensitet og reflektans tilbydes aldrig til RGB-kameraer — strålingsintensitet pr. Bayer-pixel er ikke meningsfuld for en bredbåndsfotometrisk sensor.

### Hurtigst mulig optagelse

Vippeknappen **⚡ Hurtigste optagelse — kun RAW**(orange, når den er aktiveret) tilsidesætter alle eksportvalg til**kun RAW** — plus en gratis sammensat fil med kombineret indeks for arrays — så billedet gemmes så hurtigt som muligt: beregningerne af radiance/reflektans/visning springes helt over ved optagelsen.

{% hint style="info" %}
**En `.daq` gemmes stadig.** Når der er tildelt en lyssensor, skriver »Hurtigst mulig optagelse« stadig DAQ-nedadgående måleværdier (i optagelsernes `daq/`-mappe) — så strålings-, reflektans- og indeksprodukter alle kan opbygges senere ved genbehandling (se [Genbehandling af optagelser](#re-processing-captures-into-calibrated-products)). Fastest Capture påvirker heller ikke dine valg i afkrydsningsfelterne: Slå funktionen fra, og de kommer tilbage.
{% endhint %}

### Kontrolelementer pr. array

Hvert tilsluttet array får sit eget gruppekort i ruden:

* **Afkrydsningsfeltet »Include«** (tre tilstande på tværs af medlemmer) og arrayets navn med dets visningsmodus: »(combined | separate)«.
* **Justeret**-afkrydsningsfelt (standard**aktiveret**): tilpasser eksporten af medlemmer til arrayets justeringsprofil, så eksporten er pixelregistreret på tværs af kameraer. Raw-filer forbliver uforvrængede, men indeholder transformationen i deres metadata. (Selve profilen beregnes i [panelet med arrayindstillinger](camera-settings.md#alignment-co-registration).) Array-medlemsfiler gemmes i hvert kameras oprindelige orientering, hvor eventuelle indstillinger for spejling/rotation registreres i filen i stedet for at blive anvendt.
* Kamerarækkerne er indlejret i kortet.

Array-kortet indeholder også to optagere. Tænk på dem som **overvågning kontra analyse**:

| Optager | Grad | Hvad den optager |
| --- | --- | --- |
| **● Optag indeksvideo / ■ Stop optagelse** *(kun kombinerede arrays)* | **Overvågning** | Den kombinerede live-indeks-komposit til video ved 10 fps — 8-bit, forhåndsvisningsopløsning, LUT indbagt. Kræver et åbent projekt og en streamet livevisning. Viser billeder + forløbet tid under optagelse. |
| **⦿ Optag rå burst / ■ Stop rå burst** *(ethvert array)* | **Analyse**| Rå Bayer-billeder med live-optagelseshastighed (ingen behandling) plus et manifest pr. billede og `.daq`-målinger, gemt som `captures/bursts/`. Efter en serie vises en**Opret video**-knap: den genbehandler serien offline til kalibreret video — kombineret indeks og/eller strålingsintensitet / reflektans / indeks pr. kamera — plus valgfrie TIFF-filer. Oprettelsen af den kombinerede indeks starter automatisk, når du stopper serien. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## »Capture All«-forløbet

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->



Tryk på **Capture All** i kameralisten i sidepanelet:

1. Hvert inkluderet, synligt og ikke-pauset kamera optager med de valgte eksporttyper. **Arrays udløses som én synkroniseret trigger** (en enkelt synkroniseret gruppe på tværs af alle medlemmer — se [Multi-Camera Arrays](arrays.md)); enkeltstående kameraer optager individuelt.
2. Skjulte (øje) eller pausede kameraer springes over. Et array er kun fuldt blokeret, når *alle* dets medlemmer er skjulte eller pausede.
3. Når der tildeles en lyssensor, gemmes den tilhørende DAQ-måling af nedadgående lys som en `.daq`-fil i optagelsernes `daq/`-mappe — selv for optagelser, der kun indeholder rådata — så der altid senere kan udledes radiometriske produkter.
4. Knappen viser fremskridt i realtid — »Optager… færdig/i alt« — og skifter i kontinuerlig/interval-tilstand til **Stop (N)**. Hvert optagelseselement har en timeout på 300 s.
5. Når gennemløbet er færdigt, viser en resultatmeddelelse **"N filer gemt"**eller**"N gemt, F mislykkedes"**samt "(S skjult/sat på pause/sprunget over)", når kameraer blev sprunget over. Hvis drevet bliver fyldt op, stopper kørslen og viser**"Stoppet: drev fuldt. Gemt N, F mislykkedes"** — hver fil, der ikke kunne skrives, tæller som mislykket.

## Hvor optagelserne gemmes

Optagelserne gemmes under det åbne projekt i mappen `<project>/captures/`. Hver eksporttype gemmes i sin **egen undermappe**, så en optagelse på flere niveauer aldrig blander typerne:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` er optagelsens tidsstempel, og `<serial>` er kameraets serienummer. Fritstående optagelser navngives `capture_<ts>_SN<serial>_<level>`; array-optagelser fra én synkroniseret udløser navngives `sync_<ts>_SN<serial>_<level>` og **deler ét tidsstempel på tværs af alle kameraer i gruppen** (niveau-suffikset udelades, når et kamera kun gemmer et enkelt niveau).
* **En uregelmæssighed, man skal være opmærksom på:** visningsniveauet gemmes i en mappe med navnet `preview/`, mens filerne beholder `_display` i navnet — mappe og suffiks adskiller sig kun for dette niveau.
* Ukendte niveauer gemmes i en mappe med deres eget navn; hvis der ikke kan oprettes en undermappe, skrives filen til rodmappen for optagelser i stedet for at gå tabt.
* TIFF-optagelser komprimeres som standard tabsfrit (DEFLATE) og indeholder alle deres kalibrerings- og behandlingsmetadata **inde i filens XMP** — optagelserne er selvbeskrivende og har ingen sidecar-filer ud over `.daq`-læsningen (og dens `.daq.cameras.json`).
* **Clamp-masker (nyt i 1.3.0).**Hvor den 3×3 spektrale adskillelse var nødt til at begrænse et bånd til 0 på mindst én pixel, får produkterne for strålingsintensitet, reflektans og indeks en maskering pr. pixel, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bit 0 = båndet i produktets R-kanal, bit 1 = G, bit 2 = B); produktets `Chloros:UnmixClampMask` XMP-tag peger på den. Maskerne ligger**ved siden af** produktmapperne, aldrig inde i en, så en produktmappe som f.eks. `reflectance/` kan overføres til fotogrammetri-software, som den er. Chloros’ egen import springer maskerne over.
* **Orientering (1.3.0).** Et enkeltstående kameras Preview-, Radiance-, Reflectance- og Index-produkter drejes i henhold til dets Mirror/Rotation-indstilling; Raw- og Debayered-filer forbliver, som sensoren har aflæst dem. Alle filer i arrayet forbliver i deres oprindelige format. Hver fil gemmer indstillingen i sin XMP, så genbehandlingen anvender den på samme måde.
* **Hvilket kamera en `.daq` tilhører (1.3.0).** En optagelses `.daq` matches ved behandlingstidspunktet kun med billeder fra de kameraer, som dens `.daq.cameras.json` angiver. Et selvstændigt kamera uden egen lyssensor låner derfor ikke længere et andet kameras måling — knyt en sensor til det, eller registrer en i fanen Lyssensorer. En lyssensorregistrering (og enhver `.daq` uden den pågældende fil, inklusive ældre optagelser) fungerer stadig for alle kameraer.

Dette er det samme layout, som `chloros-cli lattice capture` / `array-capture` skriver til deres `-o`-mappe — dokumenteret i [CLI-reference § Sådan ser en optagelsesmappe ud](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Genbehandling af optagelser til kalibrerede produkter

Optagede råbilleder samt den gemte `.daq` er alt, hvad behandlingspipeline har brug for — derfor er Fastest Capture sikkert at bruge til egentligt arbejde.

* **GUI**: Føj mappen med optagelser til et projekt ([Tilføjelse af filer til et projekt](../processing-images-gui/adding-files-to-a-project.md)) og behandl som normalt.
* **CLI**: Peg `process` mod **roden af optagelserne**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` importerer normalt kun den mappe, du angiver, men når den pågældende mappe ikke indeholder billeder og har undermapper, går programmet automatisk ned i undermapperne — så undermapperne på det pågældende niveau og mappen `daq/` hentes på én gang. Hver indlæsning importeres som et **enkelt billede**, hvor de øvrige niveauer er knyttet som visningsmodi, ikke som ét billede pr. niveau.

Det fungerer også at navngive en undermappe på et niveau direkte (f.eks. `…/captures/raw/`), men det efterlader mappen `daq/` – kopier den med, når du-afleder et radiometrisk produkt fra `raw/`, ellers har tidsstemplet intet at matche mod.

{% hint style="warning" %}
**Behandlingen starter altid fra `raw`.** Inden for hver optagelse er råbilledet kilden til pipelinen; `debayered`, `radiance`, `reflectance`, og `preview` følger med som visningsmodi, men føres aldrig tilbage gennem pipelinen — genbehandling af et afledt produkt ville genanvende vignettering, farve og strålingsberegninger, der allerede er indbagt i dets pixels, så Chloros-funktionen afviser dette frem for at foretage dobbeltbehandling. Renderingerne `index/` og `composite/` behandles slet ikke (de er output, ikke optagelser). En *captures*-mappe gemmes **uden** råimport og vises normalt, men `process` springer den over og angiver dette; `--input-level {raw,debayered,processed}` er den bevidste nødudgang, der tvinger et indgangssted frem. Se [CLI-referencen](../reference/cli-reference.md#what-a-captures-folder-looks-like) for de nøjagtige spring-over-meddelelser.
{% endhint %}

To yderligere adfærdsmønstre, der er værd at kende, når man skriver scripts til genbehandling:

* En `chloros-cli process`-kørsel, der anmodede om produkter, men ikke skrev **nogen billedprodukter, mislykkes med en tydelig fejlmeddelelse og afsluttes med en værdi forskellig fra nul** — du vil aldrig få en lydløs, tom kørsel. Vellykkede kørsler rapporterer antallet af produkter. (En bevidst kørsel, der kun omfatter metadata, tæller stadig som en succes.)
* Genimporterede, behandlede eksporter indtager aldrig en captures rå slot — den oprindelige rådata forbliver altid kilden til pipelinen.

## CLI-ækvivalenter

Alt på denne side kan køres headless. GUI-capture-tilstandene svarer direkte til `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Enkelt | `chloros-cli lattice array-capture` |
| Kontinuerlig | `array-capture --continuous [--count N] [--duration S]` |
| Interval | `array-capture --interval S [--duration S]` |
| Hurtigst mulig indfangning | `array-capture --fastest` |
| Justeret afkrydsningsfelt | `--aligned / --no-aligned` |
| Afkrydsningsfelter for eksporttype | `--processing LEVEL` eller `--levels L1,L2,…` (standard `all`) |
| Optag indeksvideo | `chloros-cli lattice array-record` |
| Optag rå burst / Opret video | `chloros-cli lattice array-burst` / `array-build-video` |

De fulde flagtabeller, indstillingen for smart-AE-optagelse (`--smart`) og modellen med konstant hastighed findes i [CLI-reference § Optagelsestilstande, optagere og offline-genbehandling](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

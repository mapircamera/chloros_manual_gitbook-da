# Arbejdsgange for reflektans

En DAQ-lyssensor omdanner radiometriske billeder til reflektans. Der findes to forskellige arbejdsgange:

1. **Enkelsensor** — én DAQ måler nedadgående strålingsintensitet, mens et kamera tager billeder, og Chloros
dividerer kameraets strålingsintensitet med denne referenceværdi.
2. **Dobbelt sensor** — to DAQ-sensorer, hvor den ene overvåger himlen og den anden et objekt, genererer en live spektralreflektanskurve uden brug af kamera.

## Enkelt sensor + kamera (nedadgående reference)

DAQ&#x27;en fungerer som den nedadgående lyssensor (DLS): kameraet måler opadgående stråling **L**(W/m²/sr/nm), DAQ-enheden måler nedadgående strålingsintensitet**E** (W/m²/nm), og Chloros
beregner reflektansen pr. bånd som:

> ρ = π · L / E

DAQ-målingen er altid **tidsstemplet i forhold til eksponeringen** — derfor deler DAQ&#x27;en og kameraerne et PTP-synkroniseret ur (se [DAQ-E-netværk og tidssynkronisering](ethernet-ptp.md)). Monter Sunshine-kosinus-hætten til udendørs arbejde, og angiv den korrekt; hætten skalerer E direkte (se [Cap Profiles &amp; Calibrated Range](caps-and-range.md)). Ved kvantitativt arbejde skal du huske instrumentets karakteristik: den kvantitative irradians beregnes ud fra et gennemsnit af målinger over mindst 15 sekunder.

### Live-optagelse

Kobl DAQ&#x27;en til et kamera under fanen *Kameraer*: Hvert kameras indstillingspanel har en rullemenu **Lyssensor**, der viser alle tilsluttede DAQ&#x27;er (DAQ-U/M/E) fra fanen *Lyssensorer*; for et synkroniseret array overføres et valg af lyssensor for hele arrayet til hvert enkelt medlem (de enkelte kameraer kan stadig tilsidesætte dette). Når forbindelsen er oprettet, føres sensorens spektre ind i kameraets DLS-slot, og eksporten af reflektans divideres med den matchede måling. I et array bruger et medlem med sin egen sensor sin egen måleværdi — til sine bånd i den live-sammensatte billed, sine eksportdata og sin automatiske eksponering — mens de øvrige medlemmer bruger arrayets.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


To adfærdsmønstre, der er værd at kende:

* **Ingen DAQ-binding → reflektans afvises, ikke forfalskes.** *Chloros
* afviser reflektansproduktet og registrerer årsagen til springet i stedet for lydløst at returnere et ringere produkt.
* **Den anvendte måleværdi bevares.** For hver reflektansramme skrives den faktisk anvendte DAQ-måleværdi som en `.daq`-fil i optagelsernes `daq/`-mappe, så optagelsen kan genbehandles senere ([Optagelse og .daq-formatet](recording.md)). En `.daq.cameras.json` ved siden af angiver de kameraer, den beskriver, og ved genbehandling matches den kun med disse kameras billeder.

### Behandling af optagede billeder

Til behandling efter flyvningen skal du optage en `.daq` under sessionen og gemme den sammen med billederne — behandlingsruten løser automatisk de tidsstempelmatchede nedadgående strålinger og henter eventuelle manglende fabrikskalibreringer fra MAPIR
s cloud ved første brug. GUI-optagelser føjes automatisk til det åbne projekt, når de stopper.

Reflektansreferencen kan vælges ved behandlingstidspunktet — `--reflectance-source` på `chloros-cli process` eller indstillingen for reflektanskilde i GUI’ens projektindstillinger:

| Værdi | Adfærd |
| --- | --- |
| `auto` (standard) | Et QA-godkendt kalibreringsmål inden for billedrammen er den absolutte reference; DAQ-nedstråling (ρ = π·L/E) er reserveværdien. Et mål, der består QA med en fejlmarkering (et panel, der er faldet ud, eller et bånd, der er ude af drift), foretrækkes ikke automatisk: kæderne »mål«, »mål+DAQ« og »kun DAQ« afprøves i rækkefølge efter deres målte score på de udeladte paneler. |
| `daq` | DAQ-autoritativ |
| `target` | Strengt in-frame-mål; ingen DAQ-erstatning |

Se [Kalibreringsmål](../calibration-targets.md) for mål-workflows og [LATTICE-kapitlet](../lattice/README.md) samt [CLI
-referencen](../reference/cli-reference.md) for den fulde behandlingspipeline. Når du læser eksporterede reflektanspixels, skal du bruge den stemplede skala (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3
: 65535) — se [Output-billedformater](../output-image-formats.md).

### Bånd uden for DAQ&#x27;ens kalibrerede område

DAQ&#x27;ens radiometrisk kalibrerede område er ~374–974 nm. Chloros
afviser DAQ-baseret reflektans for ethvert kameraband, hvor mindre end halvdelen af dets spektrale vægt ligger inden for dette interval, og angiver springårsagen `dls-uncalibrated-band-<nm>`. Blandt de leverede SKU’er påvirker dette kun F988: F988-reflektansen kalibreres ved hjælp af et reflektanspanel i scenen: båndet ligger uden for DAQ-lyssensorens kalibrerede område, så Chloros
anvender din seneste paneloptagelse og fastholder den mellem panelobservationerne. Hvis et F988-kamera kører i ren DAQ-tilstand, afviser Chloros
DAQ-baseret reflektans for det pågældende bånd med springårsagen `dls-uncalibrated-band-988` — panel-workflowet er den understøttede fremgangsmåde.

## Dobbelt sensor (omgivende lys + objekt)

To DAQ-sensorer — ethvert par, uanset transportform — giver et live-reflektansspektrum uden kamera: én sensor vender mod himlen (**Omgivende lyskilde**), én vender mod motivet (**Objektscanner**), og Chloros
beregner pr. bølgelængde:

> R(λ) = objekt(λ) / omgivende(λ)

(nul, hvor omgivende ≤ 0).

### I brugergrænsefladen

Når begge sensorer er tilsluttet under fanen Lyssensorer, skal du åbne overlejringen til tilføjelse af sensorer (knappen »+« på en diagramflise i gittervisningen) og vælge **Kombiner omgivende lys + objekt**. Vælg de to sensorer i rullemenuerne**Omgivende lyskilde**og**Objektscanner**, og klik på**Opret**. Gruppen vises som et selvstændigt diagram og som en række i sidepanelet med et grønt**REF**-mærke.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->


Under reflektansdiagrammet (listevisning) beregner en live **vegetationsindekstabel** indekser ud fra kurven ved hjælp af båndcentre ved blå 450 / grøn 550 / rød 670 / NIR
800 nm. Forholdsbaserede indekser, der udligner den absolutte skala (NDVI
, GNDVI
, ENDVI, WDRVI
, GRVI
, CVI
, GCI
, MSR) vises altid; indekser, der kræver absolut reflektans (EVI
, SAVI
, OSAVI
, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI
, NLI, MNLI, FCI, GEMI
), vises kun, når begge sensorer er strømkalibrerede modeller.

### Tilsyneladende vs. relativ — mærkningsreglen

Chloros
mærker uddataene fra de to sensorer ud fra, hvad sensorparret rent faktisk kan angive:

| Sensorpar | Mærke |
| --- | --- |
| Begge sensorer kalibreret — fabrikskalibreringspakke indlæst | **Tilsyneladende reflektans** |
| En af sensorerne er ikke kalibreret | **Relativ reflektans** |

Alle tre modeller er radiometriske: Når en sensors fabrikskalibreringspakke er indlæst, er dens spektre absolutte W/m²/nm, så et par kalibrerede sensorer giver et forhold til en absolut tilsyneladende reflektans — transporten bestemmer ikke dette. En sensor, der stadig streamer rå tællinger (ingen pakke tilgængelig), nedgraderer resultatet til en relativ kurve (spektralformen er stadig gyldig). Begge sensorer skal have korrekt angivne grænseværdier ([Grænseværdiprofiler og kalibreret område](caps-and-range.md)).

### Fra Python


Der findes ikke et dedikeret dual-sensor-kald i den samlede SDK
-grænseflade: Åbn to sessioner med `chloros_sdk.connect_daq_sensor()` og sammenlign selv deres `latest()`-spektre ved at anvende den samme mærkningskonvention. (Der findes også et værktøj til optagelse med to sensorer på MAPIR
s interne direkte hardware-grænseflade, som er anført i [CLI
-referencen](../reference/cli-reference.md) for fuldstændighedens skyld — det er ikke en del af den leverede CLI
; ovenstående GUI-arbejdsgang er den understøttede live-vej.)

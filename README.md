---
metaLinks: {}
---

# Kom godt i gang

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros er et softwareprogram fra [MAPIR](https://www.mapir.camera) til behandling af multispektrale billeder, live-styring af MAPIR-hardware og registrering af sensordata. Chloros 1.3.0 understøtter hele MAPIR-produktfamilien:

* **Survey3-kameraer** — behandler RAW+JPG-optagelser til kalibrerede reflektans- og vegetationsindekskort. Se [Understøttede kameraer](supported-cameras.md).
* **LATTICE-kameraer** — tilslut GigE-multispektrale kameramoduler i realtid, enkeltvis eller som synkroniserede multikamera-arrays: forhåndsvis, optag og bearbejd til kalibrerede strålings- og reflektansprodukter. Se [LATTICE-afsnittet](lattice/README.md).
* **DAQ-lyssensorer** — DAQ-U (USB), DAQ-M (Bluetooth) samt DAQ-E og DAQ-E-S (Ethernet) spektralsensorer: live-kalibrerede spektre, `.daq`-optagelser samt nedadrettet belysning til reflektansbehandling. Se [DAQ-afsnittet](daq/README.md).

{% hint style="success" %}
**Nyt i Chloros 1.3.0**: AI-assistenter kan styre Chloros via `chloros-mcp`, og CLI, SDK og `chloros-mcp` tilbyder de samme funktioner. LATTICE-indeksprodukterne er forbedret, fuldsensor-arrays er hurtigere og fungerer på ethvert netværk, og backend'en fortsætter med at køre mellem kommandoer. Se listen over nye funktioner nedenfor, og [download](download.md) installationsfilerne.
{% endhint %}

{% hint style="info" %}
**Bruger du Chloros sammen med en AI-assistent?** Denne manual er udviklet til netop det. Bed din assistent om:

* `https://mapir.gitbook.io/chloros/llms.txt` — maskinlæsbart indeks over alle sider.
* Enhver side som rå Markdown — tilføj `.md` til dens URL (f.eks. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI-referencen](reference/cli-reference.md) og [SDK-referencen](reference/sdk-reference.md) — komplette referencesider med præcise værdier, skrevet til brug for store sprogmodeller (LLM).

Eksempel på prompt: *"Læs https://mapir.gitbook.io/chloros/reference/cli-reference.md,, og skriv derefter et script, der logger ind og behandler mappen ~/flights/flight_001 til GeoTIFF-filer med reflektans + NDVI."*

Eller lad den køre Chloros direkte: `chloros-mcp`, en MCP-server installeret med SDK, fungerer sammen med Claude Desktop, Claude Code, Cursor og andre MCP-klienter.

Fuld vejledning: [Brug af Chloros med AI-assistenter](ai-assistants.md).
{% endhint %}

***

## Hvad er nyt i Chloros 1.3.0

* **AI-assistenter kan styre Chloros.** `chloros-mcp`, en MCP-server installeret med Python-SDK'en, gør det muligt for Claude Desktop, Claude Code, Cursor og andre MCP-klienter at behandle billeddata og betjene LATTICE-kameraer, -arrays og DAQ-lyssensorer. Se [Brug af Chloros med AI-assistenter](ai-assistants.md).
* **CLI, SDK og `chloros-mcp` tilbyder de samme funktioner.** Af sikkerhedsmæssige årsager forbliver kun login, DAQ-E-konfiguration og de direkte DAQ-servere hos CLI. Nogle 1.2.0-scripts skal ændres: se **Ændringer i 1.3.0** øverst i [CLI-referencen](reference/cli-reference.md) og [SDK-referencen](reference/sdk-reference.md).
* **Forbedrede LATTICE-indeksprodukter.** NDVI og de øvrige indeksprodukter, der eksporteres fra LATTICE-optagelser, er forbedret i alle behandlingsmodi.
* **Windows-installationsindstillinger.**Under**Systemindstillinger**tilbyder installationsprogrammet**Aktiver lange Windows-stier**og**Undtag Chloros-backend fra Microsoft Defender-scanning**, som begge er markeret som standard. Den første indstilling tillader, at projektmapper kan ligge dybere end 260 tegn; den anden forhindrer, at realtidsscanning bremser kameraoptagelsen (ca. 30 % af optagelseshastigheden ved høje hastigheder). Installationsprogrammet tilføjer nu også CLI-mappen til `PATH` kun én gang og fjerner de duplikerede poster, som tidligere installationsprogrammer har efterladt. Se [Download](download.md).
* **Dybe projektmapper afvises på forhånd.** Når lange stier i Windows er deaktiveret (Windows-standardindstillingen), afvises en projektmappe, der er for dyb til stigrænsen på 260 tegn, inden noget kører, og der vises den længste projektmappe, der passer, samt vejledning i, hvordan man aktiverer lange stier. Tidligere skabte en sådan kørsel ingen produkter. Når lange stier er aktiveret, bevarer produkter i meget dybe mapper nu deres metadata (GPS- og XMP-tags).
* **Klipmasker.** Når båndseparationen af et LATTICE-radiance- eller reflektansprodukt resulterer i en negativ båndværdi, klippes denne værdi ned til nul, og en mask per pixel i en `clamp_masks/`-mappe ved siden af produktmapperne registrerer, hvilke pixels og bånd der blev påvirket. Produktet navngiver sin maske i `Chloros:UnmixClampMask`-tagget. Live-visninger beskærer nu nøjagtigt på samme måde som eksporten. Se [Output Image Formats](output-image-formats.md).
* **Produkterne beskriver sig selv.** Indeksprodukter har deres egne behandlings-tags (`Chloros:ProcessingLevel=index`, indeksudtrykket og angivelse af, om de er beregnet ud fra strålingsintensitet eller reflektans) i stedet for kilderaw-filernes. Survey3-produkter har Chloros-behandlings-tags i alle behandlingsmodi. LATTICE-råfiler og debayered-filer forbliver i sensorens oprindelige orientering og registrerer deres orientering i XMP; billedviseren roterer dem kun til visning.
* **Linux-pakker leveres udpakket.** Backend og CLI installeres som mapper under `/usr/lib/chloros/`, så de ikke længere pakker sig selv ud ved hver opstart. På x86_64 bruger backendet ikke længere ca. 100 sekunder på at starte, og `.deb` er ca. halvt så stort som før. Det medfølgende ExifTool kører nu på en ren Ubuntu-installation. `.deb` installerer nu også Python-SDK'en til systemet `python3` (installer den med `sudo apt install ./chloros_1.3.0_<arch>.deb`), øger kernens socket-modtagelsesbuffere, som kamerastrømme har brug for, og tilføjer en backend-tjeneste, der kører som din bruger. Jetson-pakken, `chloros_1.3.0_arm64_jp6.deb`, er bygget til JetPack 6 og kører på Jetsons GPU med JetPacks CUDA-version af PyTorch. På Linux når hvert eksporteret billede nu disken én gang (skrevet og mærket i hukommelsen, derefter kopieret til sin plads): på en Jetson, hvor disken bestemmer tempoet, tager en LATTICE-kørsel med 288 billeder 14 minutter i stedet for 24. Se [Linux-installation](linux/linux-installation.md).
* **LATTICE OCN-kameraer aflæser mere præcist i sollys.** Radiance, reflektans og indekser får en hurtig korrektion efter båndsepareringen, så et gråt mål aflæses tæt på neutral NDVI i sollys. RGN-kameraer er uændrede.
* **Fejlrapporter i appen.** Et fejlikon øverst på fanen Debug Log sender en beskrivelse, op til 10 billeder, logfilerne og systemoplysninger til MAPIR. Se [GUI: Navigation](navigation.md).
* **LATTICE-arrays.**Arrays bevarer som standard den fulde sensor, og pakket 12-bit-transport (`--packed`, standard `auto`) øger billedhastigheden for fuld sensor med ca. en tredjedel; et array fungerer på ethvert netværk med den billedhastighed, netværket tillader (en langsom forbindelse sænker billedhastigheden i stedet for at afvise arrayet eller reducere dets billeder); et array, hvis kameraer genstarter, kommer af sig selv på fode igen, med PTP genaktiveret; automatisk eksponering sænker ikke længere denne hastighed i svagt lys (den udfylder perioden og øger derefter forstærkningen); Kontinuerlig optagelse kan følge med hurtige arrays; en**Highlights**-kontrol til RGB-kameraer; og SDK’s `ArraySession` får `grab()`, `frame_stream()` og `continuous()`. Se [LATTICE-afsnittet](lattice/README.md).
* **Mere stabil automatisk eksponering.** Eksponeringen holdes konstant ved uændret lys og stabiliserer sig inden for få sekunder efter en ændring; små højlys mørkner ikke længere motivet; ved manuel eksponering forbliver forstærkningen på »Auto«; og et kamera, der holder op med at reagere i stærkt sollys, holdes på det niveau, der passer til lysforholdene, målt af lyssensoren eller – hvis der ikke er tilsluttet en – af kameraets egne billeder. Se [Kameraindstillinger](lattice/camera-settings.md).
* **Lyssensorer.** **DAQ-E-S-understøttelse:** Dens faste diffusor kræver ingen kappe-korrektion, og hver måling indeholder enhedens tilt, roll og pitch. En optagelse angiver, om dækslet er valgt eller antaget, og behandlingen advarer, hvis det er antaget: DAQ-U, DAQ-M og DAQ-E leveres uden dæksel, så angiv navnet på det monterede dæksel. Live `.daq`-optagelser overlever et strømsvigt (gem en `.daq-journal`-fil sammen med dens `.daq`), og en DAQ-E, der holder op med at levere data, genforbindes med det udfald markeret i dens optagelse. Et projekt, der kun indeholder optagelser fra lyssensorer, viser en liste over dem og behandler dem fra appen og CLI'en, og en optagelse lavet med `chloros_scripts` vises som den DAQ, der lavede den, med en begrænsning, du kan indstille. Hver optagelse gemmer den DAQ-aflæsning, den brugte. Se [DAQ-afsnittet](daq/README.md).
* **Behandling.**LATTICE-rammer påbegynder behandlingen under måldetektion. Mål- og DAQ-referencer fastlægges pr. bånd. Et flyttet projekt genforbinder sine filer. Eksportfiler fra en 8-bit-optagelse er 8-bit med et nyt**TIFF (8-bit)**-format. En Survey3-kørsel uden kalibreringsmål gemmes som sit fallback-produkt (`Vignette_Corrected_Images` eller `Sensor_Response_Images`), ikke som reflektans. Et reflektansbilledes DLS-tags pr. bånd er altid dets eget rammebillede. Texture Aware afvises for LATTICE-optagelser, som altid får standard-debayer.
* **CLI & SDK.** Relative outputstier ender nu i den mappe, hvorfra du kører kommandoen eller scriptet (et installeret backend plejede at løse dem i sin egen installationsmappe og fejlede). Backend'et fortsætter med at køre mellem kommandoer og scripts og stopper af sig selv efter 30 minutters inaktivitet; `chloros-cli backend status` og `backend stop` styrer dette. Indlæsningskommandoer angiver navnet på alle filer, de ikke kunne skrive, og afsluttes med kode 1, og gentagne indlæsninger stopper, når drevet er fuldt. En vellykket kørsel af `process` rapporterer `Image products written: N (plus M clamp masks)`. På Windows finder SDK's `ChlorosLocal` / `process_folder` finder den installerede backend (og bruger en, der allerede kører), og `read_image_audit_tags` fungerer fra en almindelig `pip install`.

***

## Hvad er nyt i Chloros 1.2.0

* **Live kamerastyring — nyt faneblad »Kameraer«.** Tilslut LATTICE-kameraer enkeltvis eller som synkroniserede multikamera-opstillinger (PTP-tidssynkronisering, hardwareudløst optagelse) med live-preview-overlays, båndvise histogrammer, intelligent automatisk eksponering, en live-indeksberegner og firmwareopdateringer til kameraet direkte i appen.
* **Lyssensorer — ny fane »Lyssensorer«.** Tilslut DAQ-U (USB), DAQ-M (Bluetooth) og DAQ-E (Ethernet)-sensorer; se live kalibrerede spektre (W/m²/nm), gem `.daq`-filer i dit projekt, vælg cap-korrektionsprofiler og opdater DAQ-E-firmware via netværket.
* **Optagelsestilstande og optagere.** Enkelt-/kontinuerlig-/intervaloptagelse samt en "Fastest Capture"-tilstand, der kun optager rådata; projektbaseret valg af, hvilke kameraer og eksporttyper "Capture All" skal generere; array-optagere til indeksvideo i overvågningskvalitet og rå bursts i analysekvalitet med offline videokompileringer.
* **LATTICE-behandlingspipeline.** Importer LATTICE-optagelsesmapper, og udbred hvert råbillede til debayered-, preview-, float32-radiance- (W/m²/sr/nm) og reflektansprodukter med produkt-specifikke indstillingsmuligheder. Reflektansen kan stamme fra et kalibreringsmål i billedet eller DAQ-nedstråling; array-justering anvendes på eksportfiler; manglende fabrikskalibrering downloades automatisk via kameraets serienummer.
* **Projekter husker hardware.** Tilsluttede kameraer og lyssensorer gemmes sammen med projektet (`cameras.json` / `sensors.json`) og genopretter forbindelsen med de gemte indstillinger, når du genåbner projektet. Se [GUI: Projekter](projects.md).
* **Opgraderinger af billedviseren.** Avisning af cursorpixel/indeks med korrekt skalering af reflektans pr. fil, laghistogrammer, en GSD-binning-skyder, gittertilstande pr. trigger/pr. kamera, LATTICE-produktvisninger samt eksport af indeks/LUT-sandbox til disk.
* **CLI og SDK er blevet væsentligt udvidet.** Nye kommandofamilier: `lattice`, `daq pool-*`, `project` og `time-sync`; nye `process`-indstillinger (`--input-level`, produkt-specifikke slå-til/slå-fra-knapper, `--reflectance-source`, flag til array-justering); SDK-smart-connect-håndtag (`connect_camera` / `connect_array` / `connect_daq_sensor`), der automatisk starter backend; `open_project()`-automatisering; SDK-hjulet følger med installationsprogrammerne og er udgivet på PyPI som `chloros-sdk`.
* **Ærlig fejlsemantik.** En `chloros-cli process`-kørsel, der anmodede om produkter, men ikke skrev nogen, fejler nu tydeligt og afsluttes med en værdi forskellig fra nul; vellykkede kørsler rapporterer, hvor mange billedprodukter de har skrevet.
* **Nyt output-layout.** Produkterne placeres i `<project>/<camera>/<format>/<Product>_Images/`-mapper og beholder kildefilnavnet — det er mappen, ikke et filnavnesuffiks, der identificerer produktet. Se [Output-billedformater](output-image-formats.md).
* **Flere indgange, planer og sprog.** `.dng`-indgangsunderstøttelse; alle 38 grænsefladesprog er fuldt udfyldt; hardwarebegrænsninger pr. plan med gratis (uden login) brug af op til 4 kameraer og 2 lyssensorer.
* **Pålidelighed.** »Stop Processing« afsluttes korrekt med en præcis kørselsoversigt, projekter med flere kameraer eksporterer data fra hvert kamera, og opgraderinger via installationsprogrammet logger dig ikke længere ud.

***

Chloros findes i 3 brugergrænseflader:

## Chloros: Desktop-GUI-applikation

Selvstændigt, separat vindue med alle funktioner, herunder fanerne »Live-kameraer« og »Lyssensorer«. _Kun Windows._

## [Chloros CLI: Kommandolinjegrænseflade](CLI.md)

Batchbehandling via kommandolinjen samt live-kommandoerne `lattice`, `daq pool-*`, `project` og `time-sync`. Perfekt til automatisering, scripting og headless-drift. Tilgængelig på **Windows, Linux amd64 og Linux arm64 (NVIDIA Jetson)**. _Der kræves et betalt Chloros+-abonnement for at få adgang til CLI._

## [Chloros API: Python SDK](api-python-sdk.md)

Programmatisk Python-grænseflade til automatisering og brugerdefinerede arbejdsgange: fuld pipeline-behandling, live kamera-/array-sessioner, DAQ-sensorsessioner og automatisering af gemte projekter. Installeres sammen med desktop-/CLI-pakken og er også udgivet som `pip install chloros-sdk`. _API kræver et betalt Chloros+-abonnement for at få adgang._

***

## Understøttede platforme

| Platform | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Ja | Ja | Ja |
| **Linux amd64 (x86_64)** | Nej | Ja | Ja |
| **Linux arm64 (NVIDIA Jetson)** | Nej | Ja | Ja |

Se afsnittet [Linux & Edge Computing](linux/linux-overview.md) for installationsvejledning til Linux.

***

## Kom i gang i tre trin

1. **Installer** — download og kør installationsprogrammet til din platform. Se [Download](download.md).
2. **Log ind (valgfrit for GUI'en)** — GUI'en behandler billeder gratis uden en konto. Et [Chloros+-login](chloros+-login.md) giver adgang til parallelbehandling, GPU-acceleration, højere enhedsgrænser samt adgang til CLI og SDK.
3. **Opret dit første projekt** — åbn Chloros, opret et [nyt projekt](projects.md), [tilføj dine billeder](processing-images-gui/adding-files-to-a-project.md) og [start behandlingen](processing-images-gui/starting-the-processing.md). Hvis du i stedet vil styre live-hardware, skal du åbne fanen Kameraer eller Lyssensorer — se [GUI: Navigation](navigation.md).

***

## Chloros+

Selvom Chloros er gratis at bruge til de fleste opgaver, vil du måske opleve, at du har brug for mere. Her kan en betalt licens til Chloros+ være en fordel for dig. Med en Chloros+-licens kan du få adgang til nye funktioner såsom:

* **Multitrådet behandling**: fremskynd billedbehandlingen betydeligt for større projekter ved samtidig at behandle billeder gennem pipelinen.
* **GPU (CUDA)-acceleration**: udnyt nutidens større GPU-hukommelsesmuligheder til yderligere at fremskynde billedbehandlingspipelinen. Vi anbefaler 4 GB eller mere VRAM for at opnå de bedste resultater.
* **Chloros+**[**CLI**](CLI.md) **Adgang**: Kør Chloros+ fra kommandolinjen for at automatisere og integrere det i din egen software. Tilgængeligt på alle betalte niveauer; håndhæves på serversiden.
* **Chloros+**[**API**](api-python-sdk.md) **Adgang:** Kør Chloros+ fra Python for programmatisk styring, hvilket muliggør problemfri integration med dine forskningspipelines, dataanalyse-workflows og brugerdefinerede applikationer. Tilgængeligt på alle betalte abonnementer; håndhæves på serversiden.
* **Højere hardwaregrænser**: Tilslut flere kameraer og lyssensorer på én gang. Uden login kan GUI'en tilslutte op til 4 kameraer og 2 DAQ-lyssensorer; betalte abonnementer hæver begge grænser:

| Abonnement | Kameraer | DAQ-lyssensorer |
| --- | --- | --- |
| Iron (gratis, ingen login) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Brug af flere enheder**: Hver Chloros+-licens giver mulighed for at registrere 2 eller flere enheder. Brug din MAPIR Cloud-konto til at administrere registrerede enheder. Få støtte til flere enheder ved at opgradere din Chloros+-licens.
* **Avanceret teksturbevidst debayer-metode:** en højkvalitets kantbevidst debayer kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al debayer-støj.
* **Brugerdefinerede multispektrale indeksformler:** Indtast brugerdefinerede multispektrale indekser i Chloros-rasterberegnerne, både til behandling og i billedvisningssandkassen.
* **Linux & Edge Computing:** Kør Chloros på Linux x86_64- og ARM64-platforme, herunder NVIDIA Jetson, til felt- og edge-behandling. Se [Linux-oversigt](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Priser og tilmelding til Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->

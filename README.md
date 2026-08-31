---
metaLinks: {}
---

# Kom godt i gang

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros er et softwareprogram fra [MAPIR](https://www.mapir.camera) til behandling af multispektrale billeder, live-styring af MAPIR-hardware og registrering af sensordata. Chloros 1.2.0 understøtter hele MAPIR-produktfamilien:

* **Survey3-kameraer** — behandler RAW+JPG-optagelser til kalibrerede reflektans- og vegetationsindekskort. Se [Understøttede kameraer](supported-cameras.md).
* **LATTICE-kameraer** — tilslut GigE-multispektralkameramodulerne live, enkeltvis eller som synkroniserede multikamera-opstillinger: forhåndsvisning, optagelse og behandling til kalibrerede strålings- og reflektansprodukter. Se [afsnittet om LATTICE](lattice/README.md).
* **DAQ-lyssensorer** — DAQ-U (USB), DAQ-M (Bluetooth) og DAQ-E (Ethernet) spektralsensorer: live kalibrerede spektre, `.daq`-optagelser og nedadrettet belysning til reflektansbehandling. Se [afsnittet om DAQ](daq/README.md).

{% hint style="success" %}
**Nyt i Chloros 1.2.0**: live-styring af LATTICE-kameraer og -arrayer, integration af DAQ-lyssensorer, optagelsestilstande og optagere, en komplet radiometrisk behandlingspipeline til LATTICE, projektautomatisering fra CLI/SDK og meget mere. Se listen over nye funktioner nedenfor, og [Download](download.md) for at se ændringsloggen.
{% endhint %}

{% hint style="info" %}
**Bruger du Chloros med en AI-assistent?** Denne manual er skabt til netop det. Henvis din assistent til:

* `https://mapir.gitbook.io/chloros/llms.txt` — maskinlæsbart indeks over alle sider.
* Enhver side som rå Markdown — tilføj `.md` til dens URL (f.eks. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI-referencen](reference/cli-reference.md) og [SDK-reference](reference/sdk-reference.md) — komplette referencesider med nøjagtige værdier, skrevet til brug for store sprogmodeller (LLM).

Eksempel på prompt: *&quot;Læs https://mapir.gitbook.io/chloros/reference/cli-reference.md,, og skriv derefter et script, der logger ind og behandler mappen ~/flights/flight_001 til GeoTIFF-filer i formatet reflectance + NDVI.&quot;*

Fuldstændig vejledning: [Brug af Chloros med AI-assistenter](ai-assistants.md).
{% endhint %}

***

## Nyt i Chloros 1.2.0

* **Live kamerastyring — ny fane &quot;Kameraer&quot;.** Tilslut LATTICE-kameraer enkeltvis eller som synkroniserede multikamera-arrays (PTP-tidssynkronisering, hardwareudløst optagelse) med live-forhåndsvisningsoverlejringer, histogrammer pr. bånd, intelligent automatisk eksponering, en live-indeksberegner og firmwareopdateringer til kameraet direkte i appen.
* **Lyssensorer — ny fane »Lyssensorer«.** Tilslut DAQ-U (USB), DAQ-M (Bluetooth) og DAQ-E (Ethernet)-sensorer; se live kalibrerede spektre (W/m²/nm), gem `.daq`-filer i dit projekt, vælg cap-korrektionsprofiler og opdater DAQ-E-firmware via netværket.
* **Optagelsestilstande og optagere.** Enkelt-/kontinuerlig-/intervaloptagelse samt en &quot;Fastest Capture&quot;-tilstand, der udelukkende bruger rådata; projektbaseret valg af, hvilke kameraer og eksporttyper &quot;Capture All&quot; genererer; array-optagere til indeksvideo i overvågningskvalitet og rå bursts i analysekvalitet med offline videokompileringer.
* **LATTICE-behandlingspipeline.** Importer LATTICE-optagelsesmapper, og udbred hvert råbillede til debayered-, preview-, float32-radiance- (W/m²/sr/nm) og reflektansprodukter med produkt-specifikke indstillingsmuligheder. Reflektansen kan stamme fra et kalibreringsmål i billedet eller nedadgående lys fra DAQ; array-justering anvendes på eksporter; manglende fabrikskalibrering downloades automatisk via kameraets serienummer.
* **Projekter husker hardware.** Tilsluttede kameraer og lyssensorer gemmes sammen med projektet (`cameras.json` / `sensors.json`) og genopretter forbindelsen med de gemte indstillinger, når du genåbner projektet. Se [GUI: Projekter](projects.md).
* **Opgraderinger af billedviseren.** Visning af cursorpixel/indeks med korrekt skalering af reflektans pr. fil, laghistogrammer, en GSD-binning-skyder, gittertilstande pr. trigger/pr. kamera, LATTICE-produktvisninger samt eksport af indeks/LUT-sandbox til disk.
* **CLI &amp; SDK, kraftigt udvidet.** Nye kommandofamilier: `lattice`, `daq pool-*`, `project` og `time-sync`; nye `process`-indstillinger (`--input-level`, produkt-specifikke skifteknapper, `--reflectance-source`, flag til array-justering); SDK smart-connect-håndtag (`connect_camera` / `connect_array` / `connect_daq_sensor`), der automatisk starter backend&#x27;et; `open_project()`-automatisering; SDK-hjulet er inkluderet i installationsprogrammerne og udgivet på PyPI som `chloros-sdk`.
* **Ærlig fejlsemantik.** En `chloros-cli process`-kørsel, der anmodede om produkter, men ikke skrev nogen, fejler nu tydeligt og afsluttes med en værdi forskellig fra nul; vellykkede kørsler rapporterer, hvor mange billedprodukter de har skrevet.
* **Nyt output-layout.** Produkterne placeres i `<project>/<camera>/<format>/<Product>_Images/`-mapper og beholder kildefilnavnet — det er mappen, ikke et filnavnesuffiks, der identificerer produktet. Se [Output Image Formats](output-image-formats.md).
* **Flere indgange, planer og sprog.** `.dng`-indgangsunderstøttelse; alle 38 grænsefladesprog er fuldt udfyldt; hardwarebegrænsninger pr. plan med gratis (uden login) brug af op til 4 kameraer og 2 lyssensorer.
* **Pålidelighed.** »Stop Processing« afsluttes korrekt med en præcis oversigt over kørslen, projekter med flere kameraer eksporterer alle kameraer, og opgraderinger via installationsprogrammet logger dig ikke længere ud.***

Chloros er tilgængelig i 3 brugergrænseflader:

## Chloros: Desktop-GUI-applikation

Selvstændigt, separat vindue med alle funktioner, herunder fanerne »Live-kameraer« og »Lyssensorer«. _Kun Windows._

## [Chloros CLI: Kommandolinjegrænseflade](CLI.md)

Batchbehandling via kommandolinjen samt live-kommandoerne `lattice`, `daq pool-*`, `project` og `time-sync`. Perfekt til automatisering, scripting og headless-drift. Tilgængelig på **Windows, Linux amd64 og Linux arm64 (NVIDIA Jetson)**. _CLI kræver et betalt Chloros+-abonnement for at få adgang._

## [Chloros API: Python SDK](api-python-sdk.md)

Programmerbar Python-grænseflade til automatisering og brugerdefinerede arbejdsgange: fuld pipeline-behandling, live kamera-/array-sessioner, DAQ-sensorsessioner og automatisering af gemte projekter. Installeres med desktop-/CLI-pakken og udgives også som `pip install chloros-sdk`. _API kræver et betalt Chloros+-abonnement for at få adgang._

***

## Understøttede platforme

| Platform | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Ja | Ja | Ja |
| **Linux amd64 (x86_64)** | Nej | Ja | Ja |
| **Linux arm64 (NVIDIA Jetson)** | Nej | Ja | Ja |

For installationsvejledning til Linux, se afsnittet [Linux &amp; Edge Computing](linux/linux-overview.md).

***

## Kom i gang i tre trin

1. **Installer** — download og kør installationsprogrammet til din platform. Se [Download](download.md).
2. **Log ind (valgfrit for GUI&#x27;en)** — GUI&#x27;en behandler billeder gratis uden en konto. Et [Chloros+-login](chloros+-login.md) giver adgang til parallelbehandling, GPU-acceleration, højere enhedsgrænser samt adgang til CLI/SDK.
3. **Opret dit første projekt** — åbn Chloros, opret et [nyt projekt](projects.md), [tilføj dine billeder](processing-images-gui/adding-files-to-a-project.md) og [start behandlingen](processing-images-gui/starting-the-processing.md). Hvis du i stedet vil styre live-hardware, skal du åbne fanen *Kameraer* eller *Lyssensorer* — se [GUI: Navigation](navigation.md).

***

## Chloros+

Selvom Chloros er gratis at bruge til de fleste opgaver, kan det være, at du har brug for mere. Her kan en betalt licens til Chloros+ være en fordel for dig. Med en Chloros+-licens kan du få adgang til nye funktioner såsom:

* **Multitrådet behandling**: fremskynder billedbehandlingen betydeligt for større projekter ved at behandle billeder samtidigt gennem behandlingskæden.
* **GPU (CUDA)-acceleration**: Udnyt de nuværende muligheder for større GPU-hukommelse til at gøre billedbehandlingspipeline endnu hurtigere. Vi anbefaler 4 GB eller mere VRAM for at opnå de bedste resultater.
* **Chloros+**[**CLI**](CLI.md)**Adgang**: Kør Chloros+ fra kommandolinjen for at automatisere og integrere det i din egen software. Tilgængelig på alle betalte abonnementer; håndhæves på serversiden.
* **Chloros+**[**API**](api-python-sdk.md)**Adgang:** Kør Chloros+ fra Python for programmatisk styring, hvilket muliggør problemfri integration med dine forskningspipelines, dataanalyse-workflows og brugerdefinerede applikationer. Tilgængeligt på alle betalte abonnementer; håndhæves på serversiden.
* **Højere hardwaregrænser**: Tilslut flere kameraer og lyssensorer på én gang. Uden login kan GUI’en tilslutte op til 4 kameraer og 2 DAQ-lyssensorer; betalte abonnementer hæver begge grænser:

| Abonnement | Kameraer | DAQ-lyssensorer |
| --- | --- | --- |
| Iron (gratis, ingen login) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Brug af flere enheder**: Hver Chloros+-licens giver mulighed for registrering af 2 eller flere enheder. Brug din MAPIR Cloud-konto til at administrere registrerede enheder. Få understøttelse af flere enheder ved at opgradere din Chloros+-licens.
* **Avanceret teksturbevidst debayer-metode:** en højkvalitets kantbevidst debayer kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al støj fra debayeringen.
* **Brugerdefinerede multispektrale indeksformler:** Indtast brugerdefinerede multispektrale indekser i Chloros-rasterberegnerne, både til behandling og til billedvisningssandkassen.
* **Linux &amp; Edge Computing:** kør Chloros på Linux x86_64- og ARM64-platforme, herunder NVIDIA Jetson, til felt- og edge-behandling. Se [Linux-oversigt](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Priser og tilmelding</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->

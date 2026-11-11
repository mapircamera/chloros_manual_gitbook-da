---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Ofte stillede spørgsmål

<details>

<summary>Kan jeg behandle billeder fra kameraer, der ikke er af mærket MAPIR, med Chloros?</summary>

Nej, Chloros understøtter kun behandling af billeder fra MAPIR-kameraer — Survey3- og LATTICE-serierne. Se listen over [understøttede kameramodeller](supported-cameras.md) for yderligere oplysninger. Vi tilbyder dog behandling af billeder fra andre kameraer på MAPIR Cloud; se den fulde liste [her](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Understøtter Chloros LATTICE-kameraer?</summary>

Ja. Chloros 1.3.0 understøtter LATTICE M3C- og M3M-kameramoduler fra start til slut: **live-styring**— find, tilslut, få et eksempel og optag via fanen »Kameraer« i brugergrænsefladen, `chloros-cli lattice`, eller Python-SDK'et, herunder synkroniserede multikamera-opstillinger med PTP-tidssynkronisering — samt**fuld radiometrisk behandling** af optagelser (rådata → debayering → strålingsintensitet → reflektans → indeks). Se [Understøttede kameraer](supported-cameras.md) og [LATTICE-vejledningen](lattice/README.md).

</details>

<details>

<summary>Kan jeg kalibrere mine billeder for reflektans uden et kalibreringsmål?</summary>

**Survey3:** Nej. Uden et billede af kalibreringsmålet, der er taget på samme tid som de øvrige billeder, vil du ikke kunne relatere billedets pixelværdier til en kendt reflektansprocent. Hvis du heller ikke medtager loggen fra en MAPIR-lyssensor, vil det omgivende lysspektrum ikke blive målt, og reflektansresultaterne vil ikke være nøjagtige.

**LATTICE:** Ja. Reflektansen kan relateres til den nedadgående strålingsintensitet, der måles af en DAQ-lyssensor i stedet for et panel (ρ = π·L/E). Når der *er* et QA-godkendt mål inden for rammen til stede, bliver det som standard den absolutte reference (`--reflectance-source auto`), medmindre det godkendes med et flag, hvor målet og DAQ-kæderne konkurrerer om den målte score. En undtagelse: »F988-reflektansen kalibreres ved hjælp af et reflektanspanel i scenen: båndet ligger uden for DAQ-lyssensorens kalibrerede område, så Chloros anvender din seneste paneloptagelse og fastholder den mellem panelobservationerne.« Se [Kalibreringsmål](calibration-targets.md).

</details>

<details>

<summary>Har jeg brug for en DAQ-lyssensor?</summary>

Ikke til strålingsintensitet: LATTICE-strålingsintensitetseksport stammer fra hvert kameras fabriksradiometriske kalibrering og kræver hverken en DAQ-sensor eller et mål. Til **refleksion**har du brug for en reference for det omgivende lys — enten en nedadrettet måling fra en DAQ-lyssensor eller et kalibreringsmål i billedet. En DAQ-sensor giver dig mulighed for at generere kalibreret reflektans**uden at placere paneler i motivet**. Optagede `.daq`-filer matches automatisk med dine billeder via tidsstempel. Se [Kalibreringsmål](calibration-targets.md) og [CLI-referencen](reference/cli-reference.md).

</details>

<details>

<summary>Kan jeg bruge Chloros sammen med en AI-assistent (Claude, ChatGPT osv.)?</summary>

Ja — denne manual og CLI'en/SDK'en er udviklet til netop det:

* Det fulde indeks over manualen findes på `https://mapir.gitbook.io/chloros/llms.txt`, så AI-assistenter kan finde alle sider.
* Hver sides rå Markdown-kode er tilgængelig på sidens URL i små bogstaver med `.md` tilføjet (for eksempel `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI-referencen](reference/cli-reference.md) og [SDK-referencen](reference/sdk-reference.md) er skrevet med henblik på brug af store sprogmodeller (LLM): præcise flag, standardindstillinger, afslutningssemantik og kommandoer, der kan kopieres og indsættes.

Se [AI-assistenter](ai-assistants.md) for at finde ud af, hvordan du kan pege din assistent mod Chloros.

</details>

<details>

<summary>Hvor havner mine behandlede outputfiler?</summary>

Produkterne gemmes i projektmappen, grupperet efter kamera og derefter efter filformat:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **kamera-mappe** — `LATT-<sensor>-<lens>-F<filter>` for LATTICE, `<model>_<filter>` (f.eks. `Survey3N_RGN`) for Survey3
* **formatmappe** — `tiff16`, `tiff8`, `png8`, `jpg8` eller `tiff32`
* **produktmapper** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (altid under `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — kun LATTICE multispektral: masker pr. pixel ved siden af produktmapperne, se nedenfor**Eksporterede filer beholder kildefilens navn — det er mappen, der identificerer produktet, ikke et filnavnesuffiks.**Med CLI oprettes projektmappen under din standardprojektmappe (`Documents/MAPIR/Chloros Projects`, medmindre du har ændret den), medmindre du angiver `-o`; et relativt `-o` tages i forhold til den mappe, hvorfra du kører kommandoen. Bemærk, at en `chloros-cli process`-kørsel, der anmodede om produkter, men ikke skrev nogen, udskriver `Processing finished but wrote no image products.` og**afsluttes med en værdi forskellig fra nul**, så scripts kan registrere det. Se [Output-billedformater](output-image-formats.md) og [CLI-referencen](reference/cli-reference.md).

</details>

<details>

<summary>Hvad er clamp_masks-mapperne og *_clampmask.tif-filerne?</summary>

Nyt i 1.3.0. Radiance- og reflektansprodukterne fra et multispektralt LATTICE M3C-kamera gennemgår en 3×3-bånds-unmix, og hvor der produceres en negativ værdi i et bånd, klipper Chloroser denne værdi til nul. Et produkt med mindst én beskåret pixel får en enkeltbånds `uint8`-maske, der markerer, hvilke pixels og bånd der blev beskåret (bit 0 = produktets R-kanal, bit 1 = G, bit 2 = B). Masker gemmes i en `clamp_masks/`-mappe **ved siden af** produktmapperne — for eksempel `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — og produktets `Chloros:UnmixClampMask` XMP-tag angiver navnet på masken (eller angiver `none`).

De er ikke billeder af scenen: Chloros’ egen import springer dem over, og da de aldrig ligger inde i en produktmappe, kan du overføre en produktmappe til Pix4D eller Metashape, som den er. Se [Clamp-masker](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Hvorfor siger Chloros: »Projektmappen er for dyb til denne computer«?</summary>

Dette gælder kun Windows. Windows leveres med lange stier deaktiveret, og derfor må ingen filsti være længere end 260 tegn. Chloros-produkter ligger op til ca. 130 tegn under projektmappen, så Chloros afviser en dybere projektmappe før behandlingen — i stedet for at køre og ikke skrive noget, som det kunne ske før version 1.3.0. Meddelelsen angiver den længste projektmappe, der passer.

Løs problemet på en af følgende måder:

* Brug en kortere projektmappe (**Arbejdsmappe** i Projektindstillinger eller `-o` / `chloros-cli set-project-folder` til CLI).
* Aktivér lange stier i Windows, og genstart Chloros. Chloros-installationsprogrammets indstilling **Aktiver lange stier i Windows** (markeret som standard) gør netop dette; kør installationsprogrammet igen, hvis du har fjernet markeringen.

Se [Start af behandlingen](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Kan jeg redigere mine billeder, før de behandles i Chloros?</summary>

Nej. Chloros forudsætter, at inddataene ikke er blevet ændret. Du må ikke ændre filnavnene.

</details>

<details>

<summary>Kan jeg indstille mine MAPIR Survey3-kameraer til automatisk eksponering og behandle billederne i Chloros?</summary>

Nej. Survey3-billeddatasæt skal have en fast/låst eksponering, så der må ikke være automatisk lukkertid eller automatisk ISO. Alle billeder fra samme kameramodel skal have identisk lukkertid og ISO (eksponering).

LATTICE-kameraer har ikke denne begrænsning: Chloros styrer deres eksponering i realtid (Smart AE), og hver optagelse registrerer den faktisk anvendte eksponering og forstærkning, hvilket den radiometriske pipeline tager højde for.

</details>

<details>

<summary>Kan Chlorose behandle eller analysere ortomosaikbilleder?</summary>

Nej. Kun individuelle MAPIR-kamerabilleder understøttes, ikke sammensatte billeder som f.eks. et ortomosaikkort.

</details>

<details>

<summary>Hvordan kan jeg fremskynde målregistreringstrinnet i Chloros?</summary>

I filbrowsertabellen kan du forhåndsvælge målbillederne i højre kolonne, hvilket fortæller Chloros, at det kun skal lede efter kalibreringsmål i disse billeder, hvilket i høj grad fremskynder behandlingen.

</details>

<details>

<summary>Hvis jeg vil uploade mine billeder til <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> skal jeg så behandle dem i Chloros, før jeg uploader dem?</summary>

Hvis du planlægger at uploade til vores onlinebehandlingsplatform [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), skal du ikke redigere billederne før upload. Cloud udfører al den samme behandling og mere til.

</details>

<details>

<summary>Vil MAPIR nogensinde understøtte X-funktionen? Jeg ville virkelig ønske, at MAPIR tilbød X.</summary>

Vi er altid interesserede i at modtage feedback på vores produkter. Hvis du støder på et problem med vores produkter eller har et forslag til, hvordan vi kan forbedre dem, så [KONTAKT OS](https://www.mapir.camera/community/contact) for at dele dine tanker. Størstedelen af vores forskning og udvikling er baseret på at lytte til vores kunders største behov.

</details>

<details>

<summary>Er Chloros tilgængeligt til Linux?</summary>

Ja! Chloros 1.3.0 understøtter Linux amd64 (x86_64) og arm64 (NVIDIA Jetson JetPack 6) via `.deb`-pakker. CLI og Python SDK understøttes fuldt ud på Linux, herunder live-styring af LATTICE-kameraer og DAQ-sensorer. Der findes ingen GUI til Linux — al interaktion foregår via [CLI](CLI.md) eller [Python SDK](api-python-sdk.md). Se [Oversigt over Linux](linux/linux-overview.md) for yderligere oplysninger.

</details>

<details>

<summary>Kan jeg køre Chloros på NVIDIA Jetson?</summary>

Ja! Chloros understøtter NVIDIA Jetson **Orin-familien** — Orin Nano, Orin NX og AGX Orin — der kører JetPack 6. Ældre modeller (Jetson Nano, TX2, Xavier NX) kan ikke køre JetPack 6 og understøttes ikke af den aktuelle pakke. Chloros registrerer automatisk din Jetson-model og optimerer behandlingsstrategien. Jetson-pakken bruger JetPack 6’s CUDA-version af PyTorch, så behandlingen kører på Jetsons GPU. Se [NVIDIA Jetson-vejledningen](linux/nvidia-jetson-guide.md) for instruktioner til opsætning og implementering.

</details>

<details>

<summary>Optimerer Chloros automatisk til min hardware?</summary>

Ja! Chloros indeholder [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md), der automatisk registrerer din CPU, GPU, RAM og (på Jetson) temperatursensorer. Derefter vælger den den optimale behandlingsstrategi — fra `GPU_PARALLEL` på GPU'er med stor hukommelse til `GPU_SINGLE` på mindre GPU'er og til `CPU_PARALLEL` på systemer uden en CUDA-GPU, som PyTorch kan bruge. Der kræves ingen manuel konfiguration.

</details>

<details>

<summary>Hvad er 4-tråds-behandlingspipeline?</summary>

Chloros bruger en 4-trådet pipeline-arkitektur til Chloros+-brugere: Tråd 1 (detektion) indlæser billeder og detekterer kalibreringsmål, tråd 2 (kalibrering) beregner reflektanskalkulering, tråd 3 (behandling) udfører GPU-accelereret debayering og indeksberegning, og tråd 4 (eksport) skriver outputfiler. Flere billeder kan være i forskellige tråde samtidigt for at opnå maksimal gennemstrømning. Se [Behandlingspipeline](processing-architecture/processing-pipeline.md) for detaljer.

</details>

<details>

<summary>Hvordan kører jeg en diagnostik på min Chloros-installation?</summary>

Brug kommandoen `selftest` til at køre en 7-trins smogtest: version, porttilgængelighed, opstart af backend, API-forbindelse (`/api/test`), systemoplysninger (`/api/system-info` — GPU/CUDA/PyTorch), tilstedeværelse af denoiser-model samt CUDA- og denoiser-klarhed:

```bash
chloros-cli selftest
```

Dette er især nyttigt på Linux-systemer til at verificere GPU- og CUDA-opsætningen. På en Jetson betyder `CUDA: False` normalt, at JetPack 6’s CUDA- og cuDNN-biblioteker mangler: installer dem med `sudo apt install nvidia-jetpack`.

</details>

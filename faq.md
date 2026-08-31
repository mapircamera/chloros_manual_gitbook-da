---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Ofte stillede spørgsmål

<details>

<summary>Kan jeg behandle billeder fra kameraer, der ikke er af mærket MAPIR, med Chloros?</summary>

Nej, Chloros understøtter kun behandling af billeder fra MAPIR-kameraer – Survey3- og LATTICE-serierne. Se listen over [understøttede kameramodeller](supported-cameras.md) for yderligere oplysninger. Vi tilbyder dog behandling af andre kameraer på MAPIR Cloud; se den fulde liste [her](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Understøtter Chloros LATTICE-kameraer?</summary>

Ja. Chloros 1.2.0 understøtter LATTICE M3C- og M3M-kameramoduler fra start til slut: **live-styring**— find, tilslut, få et eksempel og optag fra fanen „Kameraer“ i brugergrænsefladen, `chloros-cli lattice` eller Python SDK, herunder synkroniserede multikamera-opstillinger med PTP-tidssynkronisering — samt**fuld radiometrisk behandling** af optagelser (rådata → debayering → strålingsintensitet → reflektans → indeks). Se [Understøttede kameraer](supported-cameras.md) og [LATTICE-vejledningen](lattice/README.md).

</details>

<details>

<summary>Kan jeg kalibrere mine billeder for reflektans uden et kalibreringsmål?</summary>

**Survey3:** Nej. Uden et billede af kalibreringsmålet, der er taget på samme tid som de øvrige billeder, vil du ikke kunne relatere billedets pixelværdier til en kendt reflektansprocent. Hvis du heller ikke medtager loggen fra en MAPIR-lyssensor, vil det omgivende lysspektrum ikke blive målt, og reflektansresultaterne vil ikke være nøjagtige.**LATTICE:** Ja. Reflektansen kan refereres til den nedadgående bestrålingsintensitet, der måles af en DAQ-lyssensor i stedet for et panel (ρ = π·L/E). Når der *er* et QA-godkendt mål i billedet, bliver det som standard den absolutte reference (`--reflectance-source auto`). En undtagelse: »F988-reflektansen kalibreres ved hjælp af et reflektanspanel i scenen: båndet ligger uden for DAQ-lyssensorens kalibrerede område, så Chloros anvender din seneste paneloptagelse og opbevarer den mellem panelobservationerne.« Se [Kalibreringsmål](calibration-targets.md).

</details>

<details>

<summary>Har jeg brug for en DAQ-lyssensor?</summary>

Ikke til strålingsintensitet: LATTICE-strålingsintensitetseksport kommer fra hvert kameras fabriksradiometriske kalibrering og kræver hverken en DAQ-sensor eller et mål. Til **reflektans**har du brug for en reference for det omgivende lys — enten en DAQ-lyssensors nedadgående måling eller et kalibreringsmål i billedet. Med en DAQ-sensor kan du generere kalibreret reflektans**uden at placere paneler i motivet**. Optagede `.daq`-filer matches automatisk med dine billeder via tidsstempel. Se [Kalibreringsmål](calibration-targets.md) og [CLI-referencen](reference/cli-reference.md).

</details>

<details>

<summary>Kan jeg bruge Chloros sammen med en AI-assistent (Claude, ChatGPT osv.)?</summary>

Ja — denne manual og CLI/SDK er udviklet med det formål:

* Det fulde indeks over manualen findes på `https://mapir.gitbook.io/chloros/llms.txt`, så AI-assistenter kan finde alle sider.
* Hver sides rå markdown-kode er tilgængelig på den tilhørende side med små bogstaver, URL, med `.md` tilføjet (for eksempel `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [CLI-referencen](reference/cli-reference.md) og [SDK-reference](reference/sdk-reference.md) er skrevet med henblik på brug i LLM&#x27;er: præcise flag, standardindstillinger, afslutningssemantik og kommandoer, der kan kopieres og indsættes.

Se [AI-assistenter](ai-assistants.md) for at se, hvordan du kan pege din assistent mod Chloros.

</details>

<details>

<summary>Hvor gemmes mine behandlede outputfiler?</summary>

Produkterne gemmes i projektmappen, grupperet efter kamera og derefter efter filformat:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **kamera-mappe** — `LATT-<sensor>-<lens>-F<filter>` for LATTICE, `<model>_<filter>` (f.eks. `Survey3N_RGN`) for Survey3
* **formatmappe** — `tiff16`, `tiff8`, `png8`, `jpg8` eller `tiff32`
* **produktmapper** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (altid under `tiff32`), `<INDEX>_Index_Images/`**Eksporterede filer beholder kildefilens navn — mappen identificerer produktet, ikke et filnavnesuffiks.**Med CLI oprettes projektmappen ved siden af inputmappen, medmindre du angiver `-o`. Bemærk, at en kørsel af `chloros-cli process`, der anmodede om produkter, men ikke skrev nogen, udskriver `Processing finished but wrote no image products.` og**afsluttes med en værdi forskellig fra nul**, så scripts kan registrere det. Se [Output Image Formats](output-image-formats.md) og [CLI Reference](reference/cli-reference.md).

</details>

<details>

<summary>Kan jeg redigere mine billeder, før de behandles i Chloros?</summary>

Nej. Chloros forudsætter, at inddataene ikke er blevet ændret. Du må ikke ændre filnavnene.

</details>

<details>

<summary>Kan jeg indstille mine MAPIR Survey3-kameraer til automatisk eksponering og behandle billederne i Chloros?</summary>

Nej. Survey3-billeddatasæt skal have en fast/låst eksponering, så der må ikke være automatisk lukkertid eller automatisk ISO. Alle billeder fra samme kameramodel skal have identisk lukkertid og ISO (eksponering).

LATTICE-kameraer har ikke denne begrænsning: Chloros styrer eksponeringen i realtid (Smart AE), og hver optagelse registrerer den faktisk anvendte eksponering og forstærkning, hvilket den radiometriske pipeline tager højde for.

</details>

<details>

<summary>Kan Chloros behandle eller analysere ortomosaikbilleder?</summary>

Nej. Der understøttes kun individuelle MAPIR-kamerabilleder, ikke sammensatte billeder som f.eks. et ortomosaikkort.

</details>

<details>

<summary>Hvordan kan jeg fremskynde målregistreringstrinnet i Chloros?</summary>

I filbrowserens tabel kan du forhåndsvælge målbillederne i højre kolonne, hvilket fortæller Chloros, at den kun skal søge efter kalibreringsmål i disse billeder, hvilket fremskynder behandlingen betydeligt.

</details>

<details>

<summary>Hvis jeg vil uploade mine billeder til <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> skal jeg så behandle dem i Chloros inden upload?</summary>

Hvis du planlægger at uploade til vores onlinebehandlingsplatform [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), skal du ikke redigere billederne før upload. Cloud udfører al den samme behandling og mere til.

</details>

<details>

<summary>Vil MAPIR nogensinde understøtte funktion X? Jeg ville virkelig ønske, at MAPIR tilbød X.</summary>

Vi er altid interesserede i at modtage feedback på vores produkter. Hvis du støder på et problem med vores produkter eller har et forslag til, hvordan vi kan forbedre dem, så [KONTAKT OS](https://www.mapir.camera/community/contact) for at dele dine tanker. Størstedelen af vores forskning og udvikling er baseret på at lytte til vores kunders største behov.

</details>

<details>

<summary>Er Chloros tilgængeligt til Linux?</summary>

Ja! Chloros 1.2.0 understøtter Linux amd64 (x86_64) og arm64 (NVIDIA Jetson JetPack 6) via `.deb`-pakker. CLI og Python SDK understøttes fuldt ud på Linux, herunder live-styring af LATTICE-kameraer og DAQ-sensorer. Der findes ingen GUI til Linux — al interaktion foregår via [CLI](CLI.md) eller [Python SDK](api-python-sdk.md). Se [Linux Oversigt](linux/linux-overview.md) for yderligere oplysninger.

</details>

<details>

<summary>Kan jeg køre Chloros på NVIDIA Jetson?</summary>

Ja! Chloros understøtter NVIDIA Jetson-platforme, herunder Jetson Nano, Orin Nano, Orin NX og AGX Orin, der kører JetPack 6. Chloros registrerer automatisk din Jetson-model og optimerer behandlingsstrategien. Se [NVIDIA Jetson-vejledningen](linux/nvidia-jetson-guide.md) for instruktioner til opsætning og implementering.

</details>

<details>

<summary>Optimerer Chloros automatisk til min hardware?</summary>

Ja! Chloros indeholder [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md), der automatisk registrerer din CPU, GPU, RAM og (på Jetson) temperatursensorer. Derefter vælger den den optimale behandlingsstrategi – fra `GPU_PARALLEL` på systemer med stor hukommelse til `GPU_SINGLE` på enheder med begrænsninger og til `CPU_PARALLEL` på systemer uden en NVIDIA GPU. Der kræves ingen manuel konfiguration.

</details>

<details>

<summary>Hvad er 4-tråds-behandlingspipeline?</summary>

Chloros anvender en 4-trådet pipeline-arkitektur til brugere af Chloros+: Tråd 1 (detektion) indlæser billeder og detekterer kalibreringsmål, tråd 2 (kalibrering) beregner reflektanskalibrering, tråd 3 (behandling) udfører GPU-accelereret debayering og indeksberegning, og tråd 4 (eksport) skriver outputfiler. Flere billeder kan behandles i forskellige tråde samtidigt for at opnå maksimal gennemstrømning. Se [Behandlingspipeline](processing-architecture/processing-pipeline.md) for yderligere oplysninger.

</details>

<details>

<summary>Hvordan kører jeg en diagnostik på min Chloros-installation?</summary>

Brug kommandoen `selftest` til at køre en 7-trins smoke-test: version, porttilgængelighed, opstart af backend, API-forbindelse (`/api/test`), systemoplysninger (`/api/system-info` — GPU/CUDA/PyTorch), tilstedeværelse af denoiser-model og CUDA + denoiser-klarhed:

```bash
chloros-cli selftest
```

Dette er især nyttigt på Linux/Jetson-systemer til at verificere GPU- og CUDA-opsætningen.

</details>

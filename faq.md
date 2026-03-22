---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Ofte stillede spørgsmål

<details>

<summary>Kan jeg behandle billeder fra kameraer, der ikke er af mærket MAPIR, med Chloros?</summary>

Nej, Chloros understøtter kun behandling af MAPIR-kamerabilleder. Se listen over [understøttede kameramodeller](supported-cameras.md) for mere information. Vi tilbyder behandling af andre kameraer på MAPIR Cloud, se den fulde liste [her](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Kan jeg kalibrere mine billeder for reflektans uden et kalibreringsmål?</summary>

Nej. Uden et billede af kalibreringsmålet, der er taget i forbindelse med optagelsen af de ikke-målrelaterede billeder, vil du ikke kunne relatere billedets pixelværdier til en kendt reflektansprocent. Hvis du heller ikke inkluderer loggen fra en MAPIR-lyssensor, vil det omgivende lysspektrum ikke blive målt, og reflektansresultaterne vil ikke være nøjagtige.

</details>

<details>

<summary>Kan jeg redigere mine billeder inden behandling i Chloros?</summary>

Nej. Chloros antager, at indgangsdataene ikke er blevet ændret. Du må ikke ændre filnavnene.

</details>

<details>

<summary>Kan jeg indstille mine MAPIR Survey3-kameraer til automatisk eksponering og behandle billederne i Chloros?</summary>

Nej. Survey3-billeddatasæt skal have en fast/låst eksponering, så der må ikke være automatisk lukkertid eller automatisk ISO. Alle billeder fra samme kameramodel skal have identisk lukkertid og ISO (eksponering).

</details>

<details>

<summary>Kan Chloros behandle eller analysere ortomosaikbilleder?</summary>

Nej. Kun individuelle MAPIR-kamerabilleder understøttes, ikke sammensatte billeder som et ortomosaikkort.

</details>

<details>

<summary>Hvordan kan jeg fremskynde måldetekteringstrinnet i Chloros?</summary>

I filbrowser-tabellen vil forvalg af målbillederne i højre kolonne fortælle Chloros, at det kun skal lede efter kalibreringsmål i disse billeder, hvilket i høj grad fremskynder behandlingen.

</details>

<details>

<summary>Hvis jeg vil uploade mine billeder til <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> skal jeg så behandle dem i Chloros inden upload?</summary>

Hvis du planlægger at uploade til vores onlinebehandlingsplatform [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), skal du ikke redigere billederne før upload. Cloud udfører al den samme behandling og mere til.

</details>

<details>

<summary>Vil MAPIR nogensinde understøtte X-funktionen? Jeg ville virkelig ønske, at MAPIR tilbød X.</summary>

Vi er altid interesserede i at modtage feedback på vores produkter. Hvis du finder et problem med vores produkter eller har et forslag til, hvordan vi kan forbedre vores produkter, bedes du [KONTAKTE OS](https://www.mapir.camera/community/contact) for at dele dine tanker. Størstedelen af vores forskning og udvikling styres af, at vi lytter til vores kunders største behov.

</details>

<details>

<summary>Er Chloros tilgængelig til Linux?</summary>

Ja! Chloros 1.1.0 understøtter Linux amd64 (x86_64) og arm64 (NVIDIA Jetson JetPack 6) via `.deb`-pakker. CLI og Python SDK understøttes fuldt ud på Linux. Der er ingen GUI til Linux — al interaktion foregår via [CLI](CLI.md) eller [Python SDK](api-python-sdk.md). Se [Linux Oversigt](linux/linux-overview.md) for detaljer.

</details>

<details>

<summary>Kan jeg køre Chloros på NVIDIA Jetson?</summary>

Ja! Chloros 1.1.0 understøtter NVIDIA Jetson-platforme, herunder Jetson Nano, Orin Nano, Orin NX og AGX Orin, der kører JetPack 6. Chloros registrerer automatisk din Jetson-model og optimerer dens behandlingsstrategi. Se [NVIDIA Jetson-vejledningen](linux/nvidia-jetson-guide.md) for instruktioner om opsætning og implementering.

</details>

<details>

<summary>Optimerer Chloros automatisk til min hardware?</summary>

Ja! Chloros 1.1.0 indeholder [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md), der automatisk registrerer din CPU, GPU, RAM og (på Jetson) termiske sensorer. Derefter vælger den den optimale behandlingsstrategi — fra `GPU_PARALLEL` på systemer med stor hukommelse til `GPU_SINGLE` på enheder med begrænsninger til `CPU_PARALLEL` på systemer uden en NVIDIA GPU. Der er ikke behov for manuel konfiguration.

</details>

<details>

<summary>Hvad er 4-trådsbehandlingspipeline?</summary>

Chloros 1.1.0 bruger en 4-trådet pipeline-arkitektur til Chloros+-brugere: Tråd 1 (Detektion) indlæser billeder og detekterer kalibreringsmål, Tråd 2 (Kalibrering) beregner reflektanskalkulering, Tråd 3 (Behandling) udfører GPU-accelereret debayering og indeksberegning, og Tråd 4 (Eksport) skriver outputfiler. Flere billeder kan være i forskellige tråde samtidigt for at opnå maksimal gennemstrømning. Se [Behandlingspipeline](processing-architecture/processing-pipeline.md) for detaljer.

</details>

<details>

<summary>Hvordan kører jeg diagnostik på min Chloros-installation?</summary>

Brug kommandoen `selftest` til at køre 7 systemdiagnostikker, herunder versionskontrol, porttilgængelighed, opstart af backend, API-forbindelse, systeminfo, denoiser-modeller og CUDA-tilgængelighed:

```bash
chloros-cli selftest
```

Dette er især nyttigt på Linux/Jetson-systemer til at verificere GPU- og CUDA-opsætningen.

</details>

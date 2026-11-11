# Behandlingspipeline

Chloros 1.3.0 anvender en behandlingspipeline med 4 tråde, der fungerer som et trinvis samlebånd. Hver tråd håndterer en bestemt fase i arbejdsgangen, så flere billeder kan være under behandling på forskellige trin på samme tid.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Pipelinearkitektur

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Hvert billede gennemløber alle fire tråde i rækkefølge. Med Chloros&#x27; multitrådede behandling optager flere billeder forskellige tråde samtidigt — mens tråd 3 behandler ét billede, kan tråd 1 detektere det næste, tråd 2 kalibrere et andet, og tråd 4 skrive et færdigbehandlet billede til disken.

Fremskridt rapporteres pr. tråd og streames via Server-Sent Events (backend’et offentliggør dem på `/api/events`). I CLI’s live-fremskridtsvisning er de fire faser mærket **Detektering, Analyse, Behandling, Eksport**.***

## Tråddetaljer

### Tråd 1: Registrering

**Formål**: Indlæse billeder og registrere kalibreringsmål.

* Læser billedfiler fra disken — Survey3-parrene `.raw`+`.jpg`, LATTICE `.tif`/`.tiff`-optagelser samt `.dng`
* Udtrækker EXIF-metadata (GPS, kameramodel, tidsstempler, eksponering)
* Registrerer kalibreringsmål: ArUco-markerede målgeometrier for LATTICE-optagelser og den klassiske paneldetektor for Survey3-kalibreringsmålsbilleder
* Output: billeddata + metadata + resultater af måldetektering

Primært en I/O- og CPU-afhængig tråd.

### Tråd 2: Kalibrering

**Formål**: Beregner kalibreringsparametre ud fra de detekterede mål.

* Beregner reflektanskalibreringskoefficienter ud fra målbilleder
* Beregner parametre til vignetteringskorrektion
* Bestemmer kalibreringskurver pr. bånd
* Output: kalibreringsparametre for hvert billede

En CPU-afhængig beregningstråd. Ved en Survey3-kørsel eller en blandet kørsel venter tråd 3 på denne tråd, når reflektanskalibrering er aktiveret, så dens koefficienter er klar, inden noget billede behandles. Ved en ren LATTICE-kørsel kører tråd 3 under detekteringen: hvert billede starter, når dets målanker er endeligt fastlagt. Et billede tager det tidsmæssigt nærmeste mål på begge sider, så den venter, indtil hvert billede fra dens kamera, der er tidsmæssigt tættere på end dets nuværende nærmeste mål, er blevet scannet. Produkterne er de samme, blot hurtigere.

### Tråd 3: Behandling (GPU)

**Formål**: Anvende korrektioner og beregne vegetationsindekser.**Dette er den mest beregningskrævende tråd.*** **Debayering**: konverterer RAW-Bayer-data til flerkanalsbilleder
  * Standard (hurtig, middel kvalitet) — standard, `--debayer standard`
  * Teksturbevidst (langsom, højeste kvalitet) — kun Chloros+, `--debayer texture-aware`, bruger en AI/ML-støjfjernelsesmodel
  * LATTICE mono (M3M)-optagelser er enkeltbånds: demosaik- og hvidbalance-trin springes over for disse (med en logmeddelelse på én linje), mens eventuelle M3C/Bayer-billeder i samme kørsel stadig gennemgår disse trin
* **Vignetteringskorrektion**: anvender objektivvignetteringskorrektion på hele billedet
* **Reflektanskalkulering**: anvender kalibreringskoefficienter til at konvertere til reflektansværdier
* **Indeksberegning**: beregner vegetationsindekser (NDVI, NDRE, GNDVI, …)
* Output: bearbejdede billeddata, der er klar til eksport

Denne tråd drager størst fordel af GPU-acceleration, og det er den tråd, som [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) finjusterer.

### Tråd 4: Eksport

**Formål**: Skriver de behandlede billeder til disken.

* Skriver outputfiler i det valgte format — `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Indlejrer metadata i outputfilerne (GPS, tidsstempler, behandlingsparametre) — herunder `Chloros:*`-behandlingsmærkerne, som fra og med version 1.3.0 skrives i alle behandlingsmetoder, både for Survey3 og LATTICE
* Organiserer output under projektmappen som `<camera>/<format>/<Product>_Images/` — for eksempel `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Eksporterede filer beholder kildefilens navn; mappen identificerer produktet.**
* For LATTICE-optagelser kan et enkelt kildebillede udmønte sig i flere produkter (Debayered, Preview, Radiance, Reflectance, Index), hver i sin egen produktmappe. Indeksprodukter beregnes ud fra det reflektansprodukt, som kørslen har genereret (hvis der ikke findes et, fra et andet produkt, som kørslen har genereret, f.eks. radiance). Radiance- og reflektansprodukter fra multispektrale M3C-kameraer kan også få en pixelbaseret clamp-maske i `<format>/clamp_masks/` (se [Output Image Formats](../output-image-formats.md#clamp-masks))
* Output: endelige filer på disken

Primært en I/O-begrænset tråd — SSD-lagring forbedrer ydeevnen mærkbart.

***

## Bag kulisserne: Executors

Inden for tråd 3 paralleliseres arbejdet pr. billede som følger:

* **GPU-strategier**(`GPU_SINGLE`, `GPU_PARALLEL`) bruger en pulje af**arbejderprocesser**, hvor hver er en separat OS-proces med sin egen CUDA-kontekst. Chloros starter hver arbejdsproces ved at køre sit eget installerede backend-program igen — ikke via Pythons `multiprocessing`, hvis opstart af arbejdsprocesser ikke fungerer inden for den pakkede applikation
* **`CPU_PARALLEL`** bruger en trådpulje (Pythons `ThreadPoolExecutor`) — NumPy og OpenCV frigiver GIL, så tråde er tilstrækkelige
* Jetson-enheder med 8 GB eller mindre delt RAM springer executoren helt over og behandler in-process, sekventielt
* Texture Aware på en GPU med under 7 GB VRAM kører også sekventielt — denoiser-modellen kan ikke passe ind mere end én gang

Chloros bruger ikke noget tredjeparts distribueret framework (såsom Ray). Se [Dynamisk beregnings tilpasning](dynamic-compute-adaptation.md) for at se, hvordan strategien og antallet af arbejdere vælges.

***

## Sekventiel vs. pipelined behandling

### Fri tilstand (sekventiel)

I den gratis version af Chloros behandles billeder **ét ad gangen**, sekventielt gennem alle fire trin:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI’en viser en forenklet statusbjælke i fri tilstand; de serielle faser vises som **Målregistrering**og derefter**Behandling**.

### Chloros+-tilstand (pipelined)

Med en Chloros+-licens arbejder alle fire tråde **samtidigt** på forskellige billeder:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

GUI’ens statusbjælke viser de fire trin; hold musen over den for at se fremskridtet pr. tråd. I CLI vises de samme fire trin live som **Detecting, Analyzing, Processing, Exporting**.

{% hint style="info" %}
**Én betegnelse, to navne.** CLI&#x27;en kalder fase 3 for _Processing_. Backendens fremskridtsfeed i premium-tilstand — den, som GUI-statusbjælken viser — betegner den samme fase som _Calibrating_. Det er den samme tråd, der udfører det samme arbejde (Tråd 3: debayer, korrektioner, indekser).
{% endhint %}

{% hint style="success" %}
**Pipelined-behandling med Chloros+** kan være 3-5 gange hurtigere end sekventiel behandling, afhængigt af din hardware og datasættets størrelse. Hastighedsforøgelsen er størst på systemer med hurtige GPU&#x27;er og SSD&#x27;er.
{% endhint %}

***

## Tråd 4: Eksportstatus

Eksporttråden har sin egen statusopfølgning, som du kan aflæse separat:

**Kommandolinje (CLI):**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Behandlingen er afsluttet, når tråd 4 når 100 %.

{% hint style="info" %}
**En kørsel, der ikke skriver nogen billeder, betragtes som en fejl.**Ved succes rapporterer `chloros-cli process`, hvor mange billedprodukter der blev skrevet (`Image products written: N`). Hvis der blev anmodet om produkter, og**ingen**blev skrevet — kun `project.json` og `calibration_data.json` — udskriver CLI&#x27;en `Processing finished but wrote no image products.` og**afslutter med en værdi forskellig fra nul**, hvorved projektmappen og de sædvanlige årsager angives (indgangsmappen blev ikke genkendt som en optagelse — kontroller layoutet og `--input-level` — eller alle de anmodede produkter var uanvendelige for disse kameraer). Skripter kan stole på afslutningskoden.
{% endhint %}

***

## Forholdet til dynamisk beregnings tilpasning

[Dynamisk beregnings tilpasning](dynamic-compute-adaptation.md) påvirker primært **Tråd 3 (Behandling)**:

* **`GPU_PARALLEL`**: Tråd 3 kører flere billeder gennem GPU&#x27;en samtidigt ved hjælp af `fused_gpu`-pipeline
* **`GPU_SINGLE`**: Tråd 3 serieliserer GPU-adgangen med en semafor, mens arbejdsprocesser overlapper I/O, ved hjælp af `fused_gpu` eller den hukommelseseffektive `tiled_gpu`-pipeline
* **`CPU_PARALLEL`**: Tråd 3 anvender CPU-baseret behandling med multitrådet parallelisme

Tråd 3’s GPU-hukommelsestildeling vokser også, efterhånden som tråd 1 og 2 afsluttes — se [Dynamisk GPU-hukommelsestildeling](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Næste trin

* [Dynamisk beregnings tilpasning](dynamic-compute-adaptation.md) — Sådan vælger Chloros den optimale strategi for din hardware
* [NVIDIA Jetson-vejledning](../linux/nvidia-jetson-guide.md) — Platformsspecifik pipeline-adfærd på Jetson
* [Overvågning af behandlingen](../processing-images-gui/monitoring-the-processing.md) — Overvågning af fremskridt via GUI
* [CLI-reference](../reference/cli-reference.md) — `process`, `export-status`, afslutningskoder og outputlayout

# Behandlingspipeline

Chloros1.2.0 anvender en behandlingspipeline med 4 tråde, der fungerer som et trinvis samlebånd. Hver tråd håndterer en bestemt fase i arbejdsgangen, så flere billeder kan befinde sig i forskellige faser på samme tid.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Pipelinearkitektur

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Hvert billede gennemløber alle fire tråde i rækkefølge. Med Chloros+ multitrådet behandling optager flere billeder forskellige tråde samtidigt — mens tråd 3 behandler et billede, kan tråd 1 detektere det næste, tråd 2 kalibrere et andet, og tråd 4 skrive et færdigt billede til disken.

Status rapporteres pr. tråd og streames via Server-Sent Events (backend’et offentliggør dem på `/api/events`). I CLIs live-statusvisning er de fire faser mærket **Detektering, Analyse, Behandling, Eksport**.***

## Tråddetaljer

### Tråd 1: Detektering

**Formål**: Indlæse billeder og detektere kalibreringsmål.

* Læser billedfiler fra disken — Survey3 `.raw`+`.jpg`-par, LATTICE `.tif`/`.tiff`-optagelser og `.dng`
* Udtrækker EXIF-metadata (GPS, kameramodel, tidsstempler, eksponering)
* Registrerer kalibreringsmål: ArUco-markerede målgeometrier for LATTICE-optagelser og den klassiske paneldetektor for Survey3-fotos af kalibreringsmål
* Output: billeddata + metadata + resultater af måldetektering

Primært en I/O- og CPU-afhængig tråd.

### Tråd 2: Kalibrering

**Formål**: Beregner kalibreringsparametre ud fra de detekterede mål.

* Beregner reflektanskalibreringskoefficienter ud fra målbilleder
* Beregner vignetteringskorrektionsparametre
* Bestemmer kalibreringskurver pr. bånd
* Output: kalibreringsparametre for hvert billede

En CPU-afhængig beregningstråd. Tråd 3 venter på denne, når reflektanskalibrering er aktiveret, så dens koefficienter er klar, inden noget billede behandles.

### Tråd 3: Behandling (GPU)

**Formål**: Anvende korrektioner og beregne vegetationsindekser.**Dette er den mest beregningsintensive tråd.*** **Debayering**: konverterer RAW-Bayer-data til flerkanalsbilleder
  * Standard (hurtig, middel kvalitet) — standard, `--debayer standard`
  * Teksturbevidst (langsom, højeste kvalitet) — kun Chloros+, `--debayer texture-aware`, bruger en AI/ML-støjfjernelsesmodel
  * LATTICE mono (M3M)-optagelser er enkeltbånds: demosaik- og hvidbalance-trin springes over for dem (med en enkeltlinjet logmeddelelse), mens eventuelle M3C/Bayer-billeder i samme kørsel stadig gennemgår disse trin
* **Vignetteringskorrektion**: anvender korrektion af objektivvignettering på hele billedet
* **Reflektanskalibrering**: anvender kalibreringskoefficienter til at konvertere til reflektansværdier
* **Indeksberegning**: beregner vegetationsindekser (NDVI, NDRE, GNDVI, …)
* Output: behandlede billeddata klar til eksport

Denne tråd drager størst fordel af GPU-acceleration, og det er den tråd, som [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) finjusterer.

### Tråd 4: Eksport

**Formål**: Skriver de behandlede billeder til disken.

* Skriver outputfiler i det valgte format — `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Indlejrer metadata i outputfilerne (GPS, tidsstempler, behandlingsparametre)
* Organiserer output under projektmappen som `<camera>/<format>/<Product>_Images/` — for eksempel `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Eksporterede filer beholder kildefilens navn; mappen identificerer produktet.**
* For LATTICE-optagelser kan et enkelt kildebillede udmunde i flere produkter (Debayered, Preview, Radiance, Reflectance, Index), hver i sin egen produktmappe
* Output: endelige filer på disken

Primært en I/O-begrænset tråd — SSD-lagring forbedrer ydeevnen mærkbart.

***

## Bag kulisserne: Executors

Inden for tråd 3 paralleliseres arbejdet pr. billede med Pythons standard `concurrent.futures`:

* **GPU-strategier**(`GPU_SINGLE`, `GPU_PARALLEL`) bruger en `ProcessPoolExecutor` med**spawn** -startmetoden — hver worker er en separat proces med sin egen CUDA-kontekst (`fork` ville arve forælderens initialiserede CUDA-tilstand og ødelægge børnene)
* **`CPU_PARALLEL`** bruger en `ThreadPoolExecutor` — NumPy og OpenCV frigiver GIL, så tråde er tilstrækkelige
* Jetson-enheder med 8 GB eller mindre delt RAM springer executoren helt over og behandler sekventielt i selve processen
* Texture Aware på en GPU med under 7 GB VRAM kører også sekventielt — støjfjernelsesmodellen kan ikke passe ind mere end én gang

Chlorosbruger ikke noget tredjeparts distribueret framework (såsom Ray). Se [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) for at se, hvordan strategien og antallet af arbejdere vælges.

***

## Sekventiel vs. pipelined behandling

### Free Mode (sekventiel)

I den gratis version af Chloros behandles billeder **ét ad gangen**, sekventielt gennem alle fire trin:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI&#x27;en viser en forenklet statusbjælke i fri tilstand; de serielle faser vises som **Målregistrering**og derefter**Behandling**.

### Chloros+ tilstand (pipelined)

Med en Chloros+ licens arbejder alle fire tråde **samtidigt** på forskellige billeder:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

GUI-statuslinjen viser de fire faser; hold markøren over den for at se fremskridtet for hver tråd. I CLI vises de samme fire faser live som **Detecting, Analyzing, Processing, Exporting**.

{% hint style="info" %}
**Én betegnelse, to navne.** CLI kalder fase 3 for _Processing_. Backendets fremskridtsfeed i premium-tilstand — det, som GUI-fremskridtsbjælken viser — betegner den samme fase som _Calibrating_. Det er den samme tråd, der udfører det samme arbejde (Tråd 3: debayer, korrektioner, indekser).
{% endhint %}

{% hint style="success" %}
**Pipelined-behandling med Chloros+** kan være 3-5 gange hurtigere end sekventiel behandling, afhængigt af din hardware og datasættets størrelse. Hastighedsgevinsten er størst på systemer med hurtige GPU&#x27;er og SSD&#x27;er.
{% endhint %}

***

## Tråd 4: Eksportstatus

Eksporttråden har sin egen statusovervågning, som du kan aflæse separat:

**CLI:**

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
**En kørsel, der ikke skriver nogen billeder, betragtes som en fejl.**Ved succes rapporterer `chloros-cli process`, hvor mange billedprodukter der blev skrevet (`Image products written: N`). Hvis der blev anmodet om produkter, og**ingen**blev skrevet — kun `project.json` og `calibration_data.json` — udskriver CLI `Processing finished but wrote no image products.` og**afsluttes med en værdi forskellig fra nul**, hvorved den angiver projektmappen og de sædvanlige årsager (indlæsningsmappen blev ikke genkendt som en optagelse — tjek layoutet og `--input-level` — eller alle de anmodede produkter var uanvendelige for disse kameraer). Skripter kan stole på afslutningskoden.
{% endhint %}

***

## Forholdet til dynamisk beregnings tilpasning

[Dynamisk beregnings tilpasning](dynamic-compute-adaptation.md) påvirker primært **Tråd 3 (Behandling)**:

* **`GPU_PARALLEL`**: Tråd 3 kører flere billeder gennem GPU’en samtidigt ved hjælp af `fused_gpu`-pipeline
* **`GPU_SINGLE`**: Tråd 3 serieliserer GPU-adgangen med en semafor, mens arbejdsprocesser overlapper I/O, ved hjælp af `fused_gpu` eller den hukommelseseffektive `tiled_gpu`-pipeline
* **`CPU_PARALLEL`**: Tråd 3 bruger CPU-baseret behandling med multitrådet parallelisme

Tråd 3&#x27;s GPU-hukommelsestildeling vokser også, efterhånden som tråd 1 og 2 afsluttes — se [Dynamisk GPU-hukommelsestildeling](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Næste trin

* [Dynamisk beregnings tilpasning](dynamic-compute-adaptation.md) — Sådan vælger Chloros den optimale strategi for din hardware
* [NVIDIA Jetson-vejledning](../linux/nvidia-jetson-guide.md) — Platformsspecifik pipeline-adfærd på Jetson
* [Overvågning af behandlingen](../processing-images-gui/monitoring-the-processing.md) — Overvågning af fremskridt via GUI
* [Reference til CLI](../reference/cli-reference.md) — `process`, `export-status`, afslutningskoder og outputlayout

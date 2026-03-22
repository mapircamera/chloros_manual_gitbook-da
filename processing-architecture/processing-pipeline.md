# Behandlingspipeline

Chloros 1.1.0 anvender en 4-trådet behandlingspipeline, der fungerer som et trinvis samlebånd. Hver tråd håndterer en bestemt fase i behandlingsforløbet, hvilket gør det muligt at behandle flere billeder samtidigt på forskellige trin.

***

## Pipelinearkitektur

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Hvert billede gennemløber alle fire tråde i rækkefølge. Med Chloros+ multitrådet behandling kan flere billeder befinde sig i forskellige tråde samtidigt — mens tråd 3 behandler et billede, kan tråd 1 detektere det næste, tråd 2 kan kalibrere et andet, og tråd 4 kan skrive et tidligere behandlet billede til disken.

***

## Tråddetaljer

### Tråd 1: Registrering

**Formål**: Indlæser billeder og registrerer kalibreringsmål.

* Læser billedfiler fra disken (RAW, JPG)
* Udtrækker EXIF-metadata (GPS, kameramodel, tidsstempler, eksponering)
* Registrerer ArUco-kalibreringsmål i markerede målbilleder
* Output: billeddata + metadata + resultater af målregistrering

Dette er primært en I/O- og CPU-bundet tråd.

### Tråd 2: Kalibrering

**Formål**: Beregne kalibreringsparametre ud fra registrerede mål.

* Beregner reflektanskalibreringskoefficienter fra målbilleder
* Beregner vignetteringskorrektionsparametre
* Bestemmer kalibreringskurver pr. bånd
* Output: kalibreringsparametre for hvert billede

Dette er en CPU-intensiv beregningstråd.

### Tråd 3: Behandling (GPU)

**Formål**: Anvende korrektioner og beregne vegetationsindekser.**Dette er den mest beregningsintensive tråd.*** **Debayering**: Konverterer RAW-Bayer-mønsterdata til flerkanalsbilleder
  * Standard (hurtig, middel kvalitet) — standard
  * Teksturbevidst (langsom, højeste kvalitet) — kun Chloros+, bruger AI/ML-støjfjernelse
* **Vignetteringskorrektion**: Anvender objektivvignetteringskorrektion på hele billedet
* **Reflektanskalibrering**: Anvender kalibreringskoefficienter til at konvertere til reflektansværdier
* **Indeksberegning**: Beregner vegetationsindekser (NDVI, NDRE, GNDVI osv.)
* Output: behandlede billeddata klar til eksport

Denne tråd drager størst fordel af GPU-acceleration. Systemet [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) optimerer primært denne tråds adfærd.

### Tråd 4: Eksport

**Formål**: Skriv behandlede billeder til disk.

* Skriver outputfiler i det valgte format (TIFF 16-bit, TIFF 32-bit %, PNG, JPG)
* Indlejrer EXIF-metadata i outputfiler (GPS, tidsstempler, behandlingsparametre)
* Organiserer output i undermapper efter kameramodel
* Output: endelige filer på disken

Dette er primært en I/O-bundet tråd. SSD-lager forbedrer ydeevnen for Tråd 4 betydeligt.

***

## Sekventiel vs. pipelined behandling

### Gratis tilstand (sekventiel)

I den gratis version af Chloros behandles billeder **ét ad gangen**, sekventielt gennem alle fire trin:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI-statusbjælken viser 2 trin: Målregistrering og Behandling.

### Chloros+-tilstand (pipeline)

Med en Chloros+-licens arbejder alle fire tråde **samtidigt** på forskellige billeder:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

GUI-statusbjælken viser 4 trin: Detektering, analyse, kalibrering, eksport. Hold musen over statusbjælken for at se fremskridt pr. tråd.

{% hint style="success" %}
**Pipelined-behandling med Chloros+** kan være 3-5 gange hurtigere end sekventiel behandling, afhængigt af din hardware og datasættets størrelse. Hastighedsforøgelsen er størst på systemer med hurtige GPU&#x27;er og SSD&#x27;er.
{% endhint %}

***

## Status for eksport i tråd 4

I Chloros 1.1.0 har eksporttråden (tråd 4) sin egen dedikerede statusovervågning. Du kan overvåge eksportstatus separat:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Behandlingen er afsluttet, når tråd 4 når 100 %.

***

## Forholdet til dynamisk beregningsadaptation

Systemet [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) påvirker primært **Tråd 3 (Behandling)**:

* **`GPU_PARALLEL`**-strategi: Tråd 3 kører flere billeder gennem GPU&#x27;en samtidigt ved hjælp af `fused_gpu`-pipeline
* **`GPU_SINGLE`**-strategi: Tråd 3 behandler ét billede ad gangen ved hjælp af den hukommelseseffektive `tiled_gpu`-pipeline
* **`CPU_PARALLEL`**-strategi: Tråd 3 bruger CPU-baseret behandling med multitrådet parallelisme

Tråd 3&#x27;s GPU-hukommelsestildeling ændres også dynamisk, når tråd 1 og 2 er færdige — se [Dynamisk GPU-hukommelsestildeling](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Næste trin

* [Dynamisk beregnings tilpasning](dynamic-compute-adaptation.md) — Hvordan Chloros vælger den optimale strategi for din hardware
* [NVIDIA Jetson-vejledning](../linux/nvidia-jetson-guide.md) — Platformspecifik pipelineadfærd på Jetson
* [Overvågning af behandlingen](../processing-images-gui/monitoring-the-processing.md) — GUI-overvågning af fremskridt

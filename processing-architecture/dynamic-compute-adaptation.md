# Dynamisk tilpasning af beregninger

Chloros 1.1.0 introducerer intelligent hardwaredetektering og automatisk valg af behandlingsstrategi. Behandlingsmotoren tilpasser sig din hardware – fra en Jetson Nano til en arbejdsstation med flere GPU&#x27;er – uden behov for manuel konfiguration.

***

## Sådan fungerer det

Når Chloros starter, profilerer det automatisk dit system:

1. **Registrerer operativsystemet** — Windows eller Linux
2. **Identificerer CPU-kerner og samlet RAM**

3.**Registrerer tilstedeværelsen af GPU&#x27;er** — NVIDIA CUDA-kapacitet, VRAM, model
4. **Identificerer Jetson-model** (hvis relevant) — via `/proc/device-tree/model`
5. **Kontrollerer termiske sensorer** (Jetson) — til temperaturbevidst behandling
6. **Vælger den optimale beregningsstrategi** — baseret på al detekteret hardware
7. **Konfigurerer antal arbejdere, pipelinetype og hukommelsesallokering** automatisk

Resultatet caches, så efterfølgende kørsler starter hurtigere. Hvis hardwaren ændres (f.eks. hvis der tilføjes en GPU), genprofilerer Chloros ved næste opstart.

***

## Beregningsstrategier

Chloros vælger en af tre beregningsstrategier baseret på din hardware:

| Strategi | GPU påkrævet | Arbejdere | Pipeline | Bedst egnet til |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Ja (12 GB+ VRAM eller 16 GB+ delt) | 3-4 | `fused_gpu` | Desktop-GPU&#x27;er med 12 GB+, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Ja (&lt; 12 GB VRAM) | 1-3 | `tiled_gpu` | GPU&#x27;er på entry-level, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Nej | kerner - 1 | `cpu_fallback` | Systemer uden NVIDIA GPU |

### Pipeline-typer

* **`fused_gpu`** — Fuld GPU-behandlingsvej. Alle debayer-, korrektions- og indeksoperationer kører på GPU&#x27;en i et enkelt samlet forløb. Højeste gennemstrømning, men kræver mere VRAM.
* **`tiled_gpu`** — Hukommelseseffektiv GPU-vej. Behandler billeder i fliser, så de passer inden for den begrænsede GPU-hukommelse. Lavere gennemstrømning, men fungerer på enheder med begrænset hukommelse.
* **`cpu_fallback`** — Behandling udelukkende på CPU ved hjælp af multitrådet parallelisme. Anvendes, når der ikke er en NVIDIA GPU til rådighed.***

## Platformspecifik adfærd

| Platform | Strategi | Arbejdere | Pipeline | Bemærkninger |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (serialiseret) | Hukommelseseffektiv tilstand, behandler ét billede ad gangen |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (samtidig) | Anbefalet edge-enhed — ægte parallel GPU-behandling |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (samtidig) | Maksimal edge-ydeevne |
| **Desktop med 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | God desktop-ydeevne med hukommelseseffektive fliser |
| **Desktop med 12 GB+ GPU** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Optimal desktop-ydeevne |
| **System kun med CPU** | `CPU_PARALLEL` | kerner - 1 | `cpu_fallback` | Ingen GPU krævet, bruger ThreadPool |

{% hint style="info" %}
**Jetson unified memory**: Jetson-enheder deler GPU- og CPU-hukommelse. En Jetson Orin NX 16 GB rapporterer ~15,3 GB VRAM, men dette er den samme fysiske RAM, der bruges af operativsystemet og CPU-processerne. Chloros tager højde for dette, når tærskler for hukommelsestildeling indstilles.
{% endhint %}

***

## Dynamisk GPU-hukommelsesallokering

Chloros bruger en [4-trådet behandlingspipeline](processing-pipeline.md):

* **Tråd 1** (Detektion) — Indlæsning af billeder, EXIF-parsing, måldetektion
* **Tråd 2** (Kalibrering) — Beregning af reflektanskalibrering
* **Tråd 3** (Behandling) — GPU-debayer, vignetteringskorrektion, indeksberegning
* **Tråd 4** (Eksport) — Filskrivning, indlejring af metadata

Når tidligere tråde i pipelinen har afsluttet deres arbejde (f.eks. når alle billeder er blevet detekteret), frigives deres GPU-hukommelsestildeling og **omfordeles til de resterende aktive tråde**. Dette betyder, at tråd 3 (den GPU-intensive fase) får gradvist mere hukommelse, efterhånden som pipelinen skrider frem, hvilket forbedrer gennemstrømningen for det mest beregningsintensive arbejde.

### Allokeringsfaser

| Fase | Aktive tråde | GPU-hukommelsesfordeling |
| --- | --- | --- |
| **Tidlig** | 1, 2, 3, 4 | Fordelt på alle tråde |
| **Midt-tidlig** | 2, 3, 4 | Tråd 1&#x27;s hukommelse omfordeles |
| **Midt-sen** | 3, 4 | Tråd 1+2&#x27;s hukommelse går til 3+4 |
| **Sen** | 3 eller 4 | Maksimal hukommelse til den resterende tråd |***

## Texture Aware-behandling

Texture Aware-debayer-metoden (kun Chloros+) bruger betydeligt mere GPU-hukommelse end standardmetoden på grund af AI/ML-støjfjernelsesmodellen:

* Systemer med **&lt; 7 GB VRAM** tvinges ind i en synkron behandlingsloop for Texture Aware-tilstand (ét billede ad gangen)
* Systemer med **7 GB+ VRAM** kan behandle teksturbevidst samtidigt, dog med et reduceret antal arbejdere sammenlignet med standard***

## Termisk styring (Jetson)

Jetson-enheder har termiske begrænsninger, især i lukkede eller luftbårne installationer. Chloros overvåger GPU- og CPU-temperaturer og justerer automatisk behandlingen:

| Temperatur | Reaktion |
| --- | --- |
| **&lt; 70 °C** | Normal drift — fuld hastighed |
| **70 °C** (Advarsel) | Reducer batchstørrelse |
| **80 °C** (Kritisk) | Aggressiv begrænsning — lavere samtidighed og antal arbejdsprocesser |
| **90 °C** (Nedlukning) | Stop GPU-behandlingen helt |

Temperaturovervågning bruger `tegrastats` på Jetson-platforme. På desktop-systemer med tilstrækkelig køling udløses termisk begrænsning sjældent.

***

## Håndtering af hukommelsespres

Chloros overvåger systemets hukommelsespres under behandlingen:

* **Hukommelsestærskel**: 85 % udnyttelse udløser konservativ adfærd
* **OOM-reduktion**: Hvis der opstår en out-of-memory-hændelse, reduceres allokeringen med 25 % (0,75x multiplikator)
* **Pipeline-fallback**: Under alvorligt hukommelsespres falder pipelinen automatisk tilbage fra `fused_gpu` til `tiled_gpu`
* **Swap-anbefalinger**: På Jetson advarer Chloros dig, hvis swap-pladsen er utilstrækkelig til din datasættets størrelse***

## Overvågning af beregnings tilpasning

### CLI-statusoutput

Når behandlingen starter, viser CLI den registrerede hardwareprofil:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Systemdiagnostik

Kør `chloros-cli selftest` for at se en fuld hardwareprofil og verificere beregningskapaciteter:

```bash
chloros-cli selftest
```

Dette kontrollerer CUDA-tilgængelighed, GPU-hukommelse, denoiser-modeller og backend-forbindelse.

***

## Næste trin

* [Behandlingspipeline](processing-pipeline.md) — Forståelse af 4-tråds pipelinearkitekturen
* [NVIDIA Jetson-vejledning](../linux/nvidia-jetson-guide.md) — Jetson-specifik implementering og optimering
* [CLI : Kommandolinje](../CLI.md) — Fuld CLI-reference

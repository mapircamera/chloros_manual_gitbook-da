# Dynamisk tilpasning af beregningskapacitet

Chloros 1.2.0 anvender hardwaredetektering og automatisk valg af behandlingsstrategi. Behandlingsmotoren tilpasser sig din hardware — fra en Jetson Orin Nano til en arbejdsstation med flere GPU’er — uden behov for manuel konfiguration.

***

## Sådan fungerer det

Når Chloros starter, udfører det en profilering af dit system:

1. **Registrerer operativsystemet** — Windows eller Linux
2. **Identificerer CPU-kerner og samlet RAM**

3.**Registrerer tilstedeværelsen af GPU** — NVIDIA CUDA-kompatibilitet, VRAM, model
4. **Identificerer Jetson-modellen** (hvis relevant) — via `/proc/device-tree/model`
5. **Kontrollerer termiske sensorer** (Jetson) — til temperaturbevidst behandling
6. **Vælger beregningsstrategi** — baseret på al detekteret hardware
7. **Konfigurerer antal arbejdsprocesser, pipelinetype og hukommelsestildeling** automatisk

Den detekterede profil caches for sessionen i hukommelsen og på disken, så senere kørsler starter hurtigere:

| Platform | Cached profil |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (prioriterer `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Slet den fil for at tvinge en ny registrering — nyttigt efter tilføjelse af en GPU eller mere RAM. Chloros registrerer også automatisk på ny, når cachen blev skrevet af en inkompatibel ældre version.

***

## Beregningsstrategier

Chloros vælger en af tre beregningsstrategier baseret på din hardware:

| Strategi | Valgt når | Arbejdsprocesser | Eksekutor | Pipeline |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| CUDA-GPU, der rapporterer**12 GB+ VRAM**(på Jetson unified memory, kræver også 12 GB+ samlet delt RAM) | `min(4, VRAM ÷ 4GB)`, minimum 2 —**begrænset til 2 på Jetson** | `ProcessPoolExecutor` (spawn) | `fused_gpu` |
| **`GPU_SINGLE`**| CUDA-GPU med**2–12 GB VRAM**| 3 (I/O-overlapning; GPU-adgang serieliseret via en semafor).**1 (sekventielt) på Jetson-enheder med under 12 GB RAM** | `ProcessPoolExecutor` (spawn); sekventiel in-process på Jetson-enheder med lav RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Ingen CUDA-GPU eller under 2 GB VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Eksempler på anvendelse af `GPU_PARALLEL`-formlen for arbejdsenheder: 12 GB VRAM → 3 arbejdsenheder, 16 GB+ → 4 arbejdsenheder, enhver Jetson → 2 arbejdsenheder.

Parallelisme implementeres med Python&#x27;s standard `concurrent.futures`: GPU-strategier bruger en `ProcessPoolExecutor` med **spawn** start-metoden (hver worker er en separat proces med sin egen CUDA-kontekst — `fork` ville kopiere en allerede initialiseret CUDA-tilstand og ødelægge underprocesserne), og CPU-strategien bruger en `ThreadPoolExecutor`. Chloros anvender ikke noget tredjeparts-distribueret framework (såsom Ray).

### Pipeline-typer

* **`fused_gpu`** — Fuld GPU-behandlingsvej. Debayer-, korrektions- og indeksoperationer kører på GPU&#x27;en i et enkelt fusioneret gennemløb. Højeste gennemstrømning, kræver mest VRAM.
* **`tiled_gpu`** — Hukommelseseffektiv GPU-vej. Behandler billeder i fliser, så de passer inden for den begrænsede GPU-hukommelse. Lavere gennemstrømning, men fungerer på enheder med begrænset hukommelse.
* **`cpu_fallback`** — Behandling udelukkende på CPU ved hjælp af multitrådet parallelisme. Anvendes, når der ikke er en NVIDIA-GPU til rådighed, og som sidste udvej, når begge GPU-forløb mislykkes.

Kæden af fallback-løsninger ved kørsel er altid `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Manuel overstyring af strategi

Indstil miljøvariablen `CHLOROS_STRATEGY` for at tvinge en bestemt strategi — en ekspertudvej, når den automatiske detektion vælger noget, der ikke passer til din situation (for eksempel for at holde GPU&#x27;en fri til andet arbejde):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Variablen matches uden skelnen mellem store og små bogstaver; alt, der ikke er et af de tre navne, ignoreres, og den automatiske registrering fortsætter som normalt. Ved en overstyring vælger Chloros stadig antallet af arbejdere for dig:

| Overstyring | Anvendt antal arbejdere |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Det anbefales at indstille det pr. kommando frem for permanent, så normale kørsler fortsat tilpasser sig automatisk.

***

## Platformspecifik adfærd

| Platform | Strategi | Arbejdere | Pipeline | Bemærkninger |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekventiel) | Hukommelseseffektiv tilstand, ét billede ad gangen |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekventiel) | Færre end 12 GB delt RAM tvinger til sekventiel behandling |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (samtidig) | Anbefalet edge-enhed — Jetson-begrænset til 2 arbejdsprocesser |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (samtidig) | Maksimal edge-ydeevne (også begrænset af Jetson til 2 arbejdsprocesser) |
| **Desktop med 8 GB GPU** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 arbejdere overlapper I/O, mens en semafor serieliserer GPU-adgangen |
| **Desktop med 12 GB+ GPU** | `GPU_PARALLEL` | 3–4 | `fused_gpu` (samtidigt) | Optimal desktop-ydeevne: 12 GB → 3 arbejdsprocesser, 16 GB+ → 4 |
| **System kun med CPU** | `CPU_PARALLEL` | fysiske kerner − 1 (min. 2) | `cpu_fallback` | Ingen GPU krævet, bruger en trådpulje |

{% hint style="info" %}
**Jetson unified memory**: Jetson-enheder deler GPU- og CPU-hukommelse. En Jetson Orin NX 16 GB rapporterer ~15,3 GB VRAM, men det er den samme fysiske RAM, der bruges af operativsystemet og CPU-processerne. Derfor kvalificerer Jetson-enheder med 16 GB+ sig til `GPU_PARALLEL` på samme måde som en desktop-GPU med 12 GB+, men er alligevel begrænset til 2 arbejdsprocesser — GPU’en, arbejdsprocesserne og deres CUDA-kontekster pr. arbejdsproces trækker alle på den samme delte pulje.
{% endhint %}

### GPU-budget efter VRAM (diskrete GPU&#x27;er)

På x86_64-værter med en diskret NVIDIA-GPU bestemmer den registrerede VRAM også, hvor meget af kortets processorkraft der kan benyttes, og hvor store batches der kan blive:

| Registreret VRAM | GPU-budgetloft | Multiplikator for batchstørrelse |
| --- | --- | --- |
| **8 GB+** | 90 % | ×2,0 |
| **6–8 GB** | 85 % | ×1,75 |
| **3,5–6 GB** | 80 % | ×1,5 |
| **2–3,5 GB** | 75 % | ×1,25 |
| **Under 2 GB** | 70 % | ×1,0 |

Diskrete GPU&#x27;er reserverer kun 0,5 GB til systemet, da de ikke deler system-RAM. Jetson-profiler reserverer langt mere og har en lavere øvre grænse — se [NVIDIA Jetson-vejledningen](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dynamisk GPU-hukommelsestildeling

Chloros bruger en [4-trådet behandlingspipeline](processing-pipeline.md):

* **Tråd 1** (Detektion) — Indlæsning af billeder, EXIF-parsing, måldetektion
* **Tråd 2** (Kalibrering) — Beregning af reflektanskalibrering
* **Tråd 3** (Behandling) — GPU-debayer, vignetteringskorrektion, indeksberegning
* **Tråd 4** (Eksport) — Filskrivning, indlejring af metadata

Tråd 1, 2 og 4 belaster GPU’en kun lidt; tråd 3 er den, der belaster den mest. Når de tidligere tråde i processen er færdige, **omfordeles deres GPU-budget til de resterende aktive tråde**, så tråd 3 får gradvist mere hukommelse, efterhånden som kørslen skrider frem.

### Allokeringsfaser

| Fase | Aktive tråde | GPU-hukommelsesfordeling |
| --- | --- | --- |
| **Tidlig** | 1, 2, 3, 4 | Fordelt på alle tråde, hvoraf det meste går til tråd 3 |
| **Midt-tidligt** | 2, 3, 4 | Tråd 1’s andel omfordeles |
| **Midt-sent** | 3, 4 | Tråd 1 og 2’s andele går til 3 og 4 |
| **Sen** | 3 eller 4 | Den sidste aktive tråd får sin maksimale tildeling |

Tallene følger to regler:

* En tråd, der er den **eneste** aktive, tildeles den maksimale tildeling ifølge sin profil.
* Når mere end én *krævende* GPU-opgave er aktiv, fordeles hver krævende opgaves basisallokering mellem dem (aldrig under det konfigurerede minimum).

Den værdi, der faktisk anvendes under kørsel, er den **laveste** af platformprofilens tildeling og den aktuelle anbefaling fra GPU-hukommelsesmonitoren, så et travlt grafikkort vinder altid over en optimistisk profil.***

## Teksturbevidst behandling

Den teksturbevidste debayer (**Chloros+ kun** — `--debayer texture-aware`) kører en AI/ML-støjfjernelsesmodel, der kræver ca. 1,75 GB VRAM i FP16 pr. kopi, så den bruger langt mere GPU-hukommelse end standardmetoden:

* Systemer med **under 7 GB VRAM**behandler teksturbevidst i en**synkron løkke, ét billede ad gangen** — der er ikke plads til flere modelkopier, og en arbejdspool ville blot øge konkurrencen
* Systemer med **7 GB eller mere VRAM** kan behandle Texture Aware samtidigt, dog med et reduceret antal arbejdere sammenlignet med Standard
* På **Jetson** er Texture Aware altid fastgjort til en enkelt worker, og på modeller med lavt strømforbrug (Nano, Orin Nano) anvender den også automatisk en GPU-frekvensbegrænsning — se [NVIDIA Jetson-vejledningen](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)***

## Termisk styring (Jetson)

Jetson-enheder har termiske begrænsninger, især i lukkede eller luftbårne installationer. Chloros overvåger Jetsons indbyggede temperatursensorer og skalerer batchstørrelserne automatisk:

| Temperatur | Reaktion |
| --- | --- |
| **&lt; 70 °C** | Normal drift — fuld hastighed |
| **70 °C** (Advarsel) | Batchstørrelsen reduceres gradvist (100 % → 50 % mellem 70 °C og 80 °C) |
| **80 °C** (Kritisk) | Aggressiv begrænsning (50 % → 0 % mellem 80 °C og 90 °C) |
| **90 °C** (Nedlukning) | Stopper GPU-behandlingen fuldstændigt |

På stationære systemer med tilstrækkelig køling udløses termisk begrænsning sjældent.

***

## Håndtering af hukommelsespres

Chloros overvåger GPU-hukommelsen kontinuerligt under behandlingen og reagerer på tre niveauer.

**Batchstørrelse.** En batch starter ved 8 billeder gange platformmultiplikatoren fra tabellerne ovenfor. Chloros kontrollerer derefter ledig VRAM, reserverer 20 % af den til PyTorchs eget overhead og antager ca. 100 MB GPU-hukommelse pr. 12 MP-billede — batchen er den mindste af de to: den hukommelsesbaserede grænse eller platformens basisværdi. Den falder aldrig under 1.**Præemptiv reduktion.**Ved**over 85 % VRAM-udnyttelse** reduceres batchstørrelserne, før noget går i stå.**Begrænsning af tildelingen pr. tråd.** Når den aktuelle udnyttelse stiger, nedskaleres hver tråds GPU-budget: ×0,75 ved udnyttelse over 80 %, ×0,5 ved udnyttelse over 90 %. Overvågningsgrænserne er 70 % (konservativ), 85 % (normal driftsgrænse) og 95 % (risiko for OOM).**OOM-tilbagetrækning og genopretning.** Hvis der alligevel opstår en out-of-memory-hændelse:

* halveres batchstørrelsen, og den halveres igen ved hver efterfølgende OOM — hver efterfølgende vellykket batch rykker denne straf et trin tilbage
* reduceres tildelingerne til aktive tråde til 70 % af deres aktuelle værdi, og allokatoren skifter til sin konservative strategi, som lempes igen efter en række vellykkede tildelinger
* under kraftigt pres skifter pipelinen tilbage fra `fused_gpu` til `tiled_gpu` og til `cpu_fallback` som en sidste udvej

**Værts-RAM (Jetson).** Før behandlingen estimerer CLI den maksimale værtshukommelse ud fra antallet af billeder og debayer-tilstand og advarer, hvis RAM plus filbaseret swap sandsynligvis er utilstrækkeligt, idet den udskriver de nøjagtige kommandoer til at tilføje swap — se [NVIDIA Jetson-vejledningen](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).***

## Overvågning af beregnings tilpasning

### Systemdiagnostik

`chloros-cli selftest` er den hurtigste måde at bekræfte, hvad beregningslaget ser:

```bash
chloros-cli selftest
```

Dens 7 kontroller dækker version, porttilgængelighed, opstart af backend, `/api/test`, systemoplysninger, tilstedeværelse af denoiser-model samt CUDA- og denoiser-klarhed. Kontrol 5 viser hardwarelinjen direkte:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Kontrol 7 viser `CUDA: <bool>, Denoiser: <bool>` — begge skal være sande, for at Texture Aware overhovedet kan bruges.

### Backend-logfiler

Strategien og antallet af arbejdsprocesser vælges internt i backenden ved starten af hvert kørsel — der er ingen CLI-banner, der angiver disse. Når noget opfører sig uventet (en GPU-sti, der falder tilbage, en OOM, en støjfjerner, der ikke vil indlæses), vises det i backend-loggen for den pågældende session:

| Platform | Logplacering |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (én fil pr. opstart) |
| **Linux, CLI-startet backend** | også `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Live-status

Under en kørsel viser CLI live-status pr. tråd (Detektering, Analyse, Behandling, Eksport), der streames via Server-Sent Events — den praktiske indikator på, om tråd 3 udgør flaskehalsen. Se [Behandlingspipeline](processing-pipeline.md).

***

## Næste trin

* [Behandlingspipeline](processing-pipeline.md) — Forståelse af 4-tråds-pipelinearkitekturen
* [NVIDIA Jetson-vejledning](../linux/nvidia-jetson-guide.md) — Jetson-specifik implementering og optimering
* [CLI : Kommandolinje](../CLI.md) — Vejledningen til CLI
* [CLI-reference](../reference/cli-reference.md) — Omfattende kommandoliste for version 1.2.0

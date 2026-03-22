# NVIDIA Jetson-vejledning

Chloros på NVIDIA Jetson muliggør multispektral billedbehandling i udkanten – i felten, på droner og i fjerntliggende installationer. Chloros registrerer automatisk din Jetson-model og optimerer behandlingsstrategien til din hardware.

***

## Understøttede Jetson-modeller

| Model                | RAM            | Behandlingsstrategi                                   | Anbefalet anvendelse                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB delt | `GPU_PARALLEL` (4 arbejdere)                            | Maksimal ydeevne, store datasæt                      |
| **Jetson Orin NX**   | 8–16 GB delt  | `GPU_PARALLEL` (3 arbejdere, 16 GB) / `GPU_SINGLE` (8 GB) | Primær anbefaling til luftbåren og feltbaseret implementering |
| **Jetson Orin Nano** | 8 GB delt     | `GPU_SINGLE` (1 arbejdsenhed)                               | Edge-computing på entry-level                                 |
| **Jetson Nano**      | 4-8 GB delt   | `GPU_SINGLE` (1 arbejdsenhed)                               | Indgangsniveau, begrænset hukommelse                          |

{% hint style="info" %}
**Ældre Jetson-modeller** (TX2, TX1, Xavier NX) understøttes muligvis ikke. Ydelsen vil variere afhængigt af tilgængelig GPU-hukommelse og CUDA-kapacitet.
{% endhint %}

***

## Krav

* **JetPack 6.x** (seneste version anbefales)
* **NVIDIA CUDA** (inkluderet i JetPack)
* **Chloros+-licens** (kræves for adgang til CLI/SDK)

## Installation

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

For generelle detaljer om installation af Linux, se [Linux Installation](linux-installation.md).

***

## Dynamisk beregnings tilpasning på Jetson

Chloros registrerer automatisk din Jetson-model og vælger den optimale behandlingsstrategi. **Der kræves ingen manuel justering.**

### Sådan fungerer det

Ved opstart profilerer Chloros dit system:

1. **Registrerer Jetson-modellen** via `/proc/device-tree/model`
2. **Læser tilgængelig GPU/delt hukommelse**

3.**Vælger en behandlingsstrategi** (`GPU_PARALLEL`, `GPU_SINGLE` eller `CPU_PARALLEL`)
4. **Indstiller antal arbejdere, pipelinetype og hukommelsesallokering** automatisk

### Adfærd pr. model

| Jetson-model                | Strategi       | Arbejdere | Pipeline                       | Samkøring |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8 GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (hukommelseseffektiv) | Serieliseret  |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Serieliseret  |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Serieliseret  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (fuld GPU-sti)    | Samtidig  |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Samtidig  |

{% hint style="success" %}
**Jetson Orin NX 16 GB** er det perfekte valg til edge-implementering — den anvender `GPU_PARALLEL`-strategien med 3 samtidige arbejdsprocesser, hvilket leverer ægte parallel GPU-behandling i en kompakt formfaktor.
{% endhint %}

Den væsentligste forskel mellem platformene er **hukommelsen**. En Jetson Nano med 8 GB delt hukommelse skal behandle billeder ét ad gangen ved hjælp af en hukommelseseffektiv tiled-tilgang, mens en Orin NX med 16 GB kan køre 3 billeder gennem GPU&#x27;en samtidigt ved hjælp af den fusionerede pipeline med højere gennemstrømning.

Se [Dynamisk beregningsadaptation](../processing-architecture/dynamic-compute-adaptation.md) for den komplette reference til beregningsadaptation.

***

## Termisk styring

Jetson-enheder har begrænset termisk spillerum, især i lukkede eller luftbårne installationer. Chloros inkluderer automatisk termisk overvågning og begrænsning:

| Temperatur         | Handling                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normal drift — fuld behandlingshastighed          |
| **70 °C** (Advarsel)  | Reducer batchstørrelsen automatisk                   |
| **80 °C** (Kritisk) | Aggressiv begrænsning — lavere samtidighed         |
| **90 °C** (Nedlukning) | Stop GPU-behandlingen helt — afkøling påkrævet |

{% hint style="warning" %}
**Sørg for tilstrækkelig ventilation og køling** for vedvarende behandling, især i lukkede feltkabinetter eller luftbårne systemer. Termisk begrænsning vil reducere behandlingskapaciteten for at beskytte hardwaren.
{% endhint %}

***

## Hukommelsesstyring

Jetson-enheder bruger **samlet hukommelse** — GPU&#x27;en og CPU&#x27;en deler den samme fysiske RAM. Dette betyder, at den rapporterede VRAM (f.eks. 15,3 GB på Orin NX 16 GB) ikke er dedikeret GPU-hukommelse; den deles med operativsystemet og andre processer.

### Anbefalinger vedrørende swap

Ved store datasæt eller Texture Aware-debayer-behandling kan Chloros anbefale at oprette swap-plads:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Hukommelsesestimater pr. billede:**

* Standard debayer: ~10 MB pr. billede
* Texture Aware debayer: ~15 MB pr. billede

Chloros beregner automatisk den nødvendige hukommelse baseret på størrelsen af dit datasæt og advarer dig, hvis swap anbefales.

### OOM (Out of Memory) Fallback

Hvis der opdages en situation med utilstrækkelig hukommelse under behandlingen:

1. Chloros reducerer automatisk antallet af GPU-arbejdere
2. Skifter fra `fused_gpu` til `tiled_gpu`-pipeline (mere hukommelseseffektiv)
3. Fortsætter behandlingen med reduceret gennemstrømning i stedet for at gå ned

***

## Implementering i felten

### Overvejelser vedrørende strømforbrug

| Jetson-model     | Typisk strømforbrug | Bemærkninger                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5-10 W              | USB-C eller barrel-stik    |
| Jetson Orin Nano | 7-15 W              | DC-barrel-stik          |
| Jetson Orin NX   | 10-25 W             | DC-barrel-stik          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD eller barrel-stik |

Planlæg dit strømforbrug til vedvarende databehandling — det højeste strømforbrug opstår under GPU-intensiv tråd 3 (databehandling).

### Anbefalinger til lagerplads

* **NVMe SSD** anbefales kraftigt til arm64-implementeringer
* SD-kort er for langsomme til databehandling — brug dem kun som boot-medie
* Planlæg med 2-3 gange størrelsen af dine rå billeddata til den behandlede output

### Headless drift via SSH

Chloros CLI er ideel til headless Jetson-installationer:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Automatiseret behandling med systemd

Opret en systemd-tjeneste til automatiseret behandling:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

Kombiner med en systemd-timer til planlagt behandling:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Eksempler på arbejdsgange

### Grundlæggende Jetson-behandling

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK på Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Batchbehandling af flere flyvninger

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Anbefalede Jetson-systemer til brug i felten

Til anvendelse i felten og i luften bør du overveje disse Jetson Orin NX 16 GB-carrierboard-muligheder:

* **Luftbåren/drone**: Systemer med vibrationsklassificering (MIL-STD), letvægt (under 300 g), passiv køling
* **Robust felt**: IP67/IP69K vandtætte kabinetter med PoE GigE-kameratilslutning
* **Minimal/budget**: Udviklerkits med ekstra kabinetter

Kontakt [MAPIR Support](https://www.mapir.camera/community/contact) for specifikke hardwareanbefalinger til dit anvendelsesscenarie.

***

## Næste trin

* [Linux Installation](linux-installation.md) — Generelle detaljer om Linux-installation
* [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) — Fuld reference til beregningsstrategi
* [Processing Pipeline](../processing-architecture/processing-pipeline.md) — Forståelse af 4-tråds-pipeline
* [CLI : Command Line](../CLI.md) — Fuld CLI-reference
* [API : Python SDK](../api-python-sdk.md) — Fuldstændig reference til SDK

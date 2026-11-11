# NVIDIA Jetson-vejledning

Chloros
på NVIDIA Jetson muliggør multispektral billedbehandling i edge-miljøet — i felten, på UAV’er og i fjerntliggende installationer. Chloros
1.3.0 genkender din Jetson-model ved opstart og optimerer behandlingsstrategien i forhold til den fundne hardware. **Der kræves ingen manuel justering.**
{% hint style="info" %}
**Jetson-pakken 1.3.0 udfører behandlingen på Jetsons GPU.** `chloros_1.3.0_arm64_jp6.deb` er udviklet til JetPack 6 og bruger JetPack 6’s CUDA-version af PyTorch, så behandlingen – inklusive den teksturbevidste debayer – kører på Orins GPU. Pakken indeholder ikke CUDA i sig selv: den bruger de CUDA- og cuDNN-biblioteker, som JetPack 6 installerer (`sudo apt install nvidia-jetpack` på et image, der mangler dem). Hvor der ikke er en CUDA-GPU til rådighed, kører behandlingen på CPU&#x27;en, og Texture Aware bruger Standard-debayeren.
{% endhint %}

***

## Understøttede Jetson-modeller

| Model                | RAM            | Behandlingsstrategi                                     | Anbefalet anvendelse                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB delt | `GPU_PARALLEL` (2 arbejdsprocesser)                              | Maksimal ydeevne, store datasæt                      |
| **Jetson Orin NX**   | 8–16 GB delt  | `GPU_PARALLEL` (2 arbejdsprocesser, 16 GB) / `GPU_SINGLE` (8 GB)   | Primær anbefaling til implementering i felten              |
| **Jetson Orin Nano** | 8 GB delt     | `GPU_SINGLE` (1 arbejdsenhed, sekventiel)                     | Edge-computing på entry-level                                 |

{% hint style="info" %}
Linux
-pakken til arm64 kræver **JetPack 6**, som er tilgængelig på Jetson Orin-familien. Ældre modeller (Nano, TX2, Xavier NX) kan ikke køre JetPack 6 og understøttes ikke af den aktuelle pakke.
{% endhint %}

***

## Krav

* **JetPack 6.x** (seneste version anbefales)
* **NVIDIA CUDA og cuDNN** fra JetPack 6 (`sudo apt install nvidia-jetpack`, hvis dit image ikke indeholder dem)
* **Betalt Chloros
+-abonnement** — Copper-niveau eller højere (kræves for al adgang til CLI
/SDK
; håndhæves på serversiden)

## Installation

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

`.deb` installerer Python
-SDK
&#x27;en til systemet `python3`, der passer til denne build. Kun et virtualenv skal have det medfølgende wheel installeret i sig: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (fuldstændig offline). For generelle detaljer om Linux
-installation, filplaceringer og fejlfinding, se [Linux
-installation](linux-installation.md).

{% hint style="info" %}
**Ingen udpakning ved opstart længere.** Fra og med version 1.3.0 installeres backend og CLI
&#x27;en udpakket under `/usr/lib/chloros/`, så de ikke længere udpakker sig selv til et midlertidigt bibliotek hver gang, de startes. Midlertidige filer placeres stadig automatisk i `/mnt/ssd/tmp`, hvis denne mappe findes; ellers kan du indstille `TMPDIR` til en sti på din NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dynamisk beregnings tilpasning på Jetson

### Sådan fungerer det

Ved opstart opretter Chloros
en profil for dit system:

1. **Registrerer Jetson-modellen** via `/proc/device-tree/model`
2. **Læser den tilgængelige delte GPU-/CPU-hukommelse** (Jetson bruger samlet hukommelse)
3. **Spørger PyTorch, om det kan bruge CUDA** — med JetPack 6&#x27;s CUDA og cuDNN installeret kan det det
4. **Vælger automatisk en behandlingsstrategi** (`GPU_PARALLEL`, `GPU_SINGLE` eller `CPU_PARALLEL`) og antallet af arbejdsprocesser

Valget afhænger af **den samlede mængde delt RAM**, ikke af modelnavnet:

* **Under 12 GB samlet RAM**(alle 8 GB Jetson-enheder):**1 worker — bevidst sekventiel behandling**. Hukommelsen er for begrænset til samtidige workers, så billederne behandles ét ad gangen. På Jetson-enheder med**8 GB eller mindre** springer tråd 3 arbejdspuljen helt over og udfører sit arbejde pr. billede i selve processen. Backend-loggen benævner denne strategi `GPU_SINGLE`.
* **12 GB eller mere**(Orin NX 16 GB, AGX Orin): Den samlede hukommelse opfylder betingelserne for `GPU_PARALLEL`, men antallet af arbejdsprocesser er**begrænset til 2 på Jetson** — GPU’en, worker-processernes RAM og deres CUDA-kontekster pr. worker trækker alle på den samme delte pool, så flere workers medfører en risiko for fejl på grund af manglende hukommelse.
* **Ingen brugbar CUDA** (JetPacks CUDA-biblioteker mangler): `CPU_PARALLEL` ved 12 GB eller mere, ét billede ad gangen nedenfor.

Du kan tilsidesætte det automatiske valg med miljøvariablen `CHLOROS_STRATEGY` — se [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Adfærd pr. model

| Jetson-model                | Strategi       | Arbejdere | Udførelse                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Sekventiel in-process-loop (`tiled_gpu` ved hukommelsespres) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Sekventiel loop i processen                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Samtidige arbejdsprocesser, `fused_gpu`-sti  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Samtidige arbejdsprocesser, `fused_gpu`-sti  |

Den væsentligste forskel mellem platformene er **hukommelsen**. En 8 GB Jetson skal behandle billeder ét ad gangen ved hjælp af en hukommelseseffektiv tiled-tilgang, når belastningen er høj, mens en Orin med 16 GB eller mere kan køre to billeder gennem GPU&#x27;en samtidigt ved hjælp af den fusionerede pipeline med højere gennemstrømning.

### GPU-budget pr. model

Hver Jetson-model har også en hardwareprofil, der begrænser, hvor meget af den delte pool behandlingen må benytte, og skalerer batchstørrelserne:

| Model | Øvre grænse for GPU-budget | Multiplikator for batchstørrelse | Reserveret til system/skærm |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70 % | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75 % | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80 % | ×1,5 | 4,0 GB |

Den registrerede RAM-kapacitet justerer profilen: En Jetson, der rapporterer **16 GB eller mere**, får sin batchmultiplikator hævet med ×1,2. Basisbatchstørrelsen før multiplikatorer er 8 billeder.

Se [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) for den komplette reference til beregnings tilpasning.

***

## GPU-frekvensbegrænsning for Texture Aware på Nano og Orin Nano

Texture Aware-debayeren kører GPU-neuralt netværksinferens, hvilket kan udløse **overstrømsadvarsler**på Jetson-modeller med lavt strømforbrug (10–15 W-klassen) ved fuld GPU-klokkehastighed. Når du sender `--debayer texture-aware` på en**Jetson Nano eller Orin Nano**, kontrollerer CLI
&#x27;en GPU&#x27;ens maksimale frekvens og begrænser den til **510 MHz** (510000000), hvis den aktuelt er højere:

* Hvis CLI
&#x27;en kan skrive til GPU-frekvensens sysfs-node, **anvendes begrænsningen automatisk**, og der vises en bekræftelse.
* Hvis ikke (kræver root-rettigheder), viser CLI
&#x27;en den nøjagtige `sudo`-kommando til at anvende begrænsningen manuelt, venter et øjeblik, så du kan læse den, og fortsætter derefter — behandlingen kører stadig, men der kan vises advarsler om overstrøm.

Sådan anvender du begrænsningen selv inden behandlingen:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modeller med højere effekt (Orin NX 25W, AGX Orin 60W) kører ved fuld GPU-hastighed; der anvendes ingen begrænsning. Standard-debayeren udløser aldrig begrænsningen på nogen model.

{% hint style="info" %}
**Texture Aware på Jetson kører altid ét billede ad gangen.** Hver worker ville have brug for sin egen CUDA-kontekst (~1 GB) plus sin egen kopi af denoiser-modellen, hvilket den samlede hukommelse ikke kan klare — derfor er Texture Aware-stien på Jetson fastgjort til en enkelt worker med serialiseret GPU-adgang. Forvent, at Texture Aware vil være markant langsommere end Standard på enhver Jetson. Texture Aware gælder for Survey3
-billeder: LATTICE-optagelser får altid Standard-debayer. Uden en brugbar CUDA-GPU anvender kørslen Standard, og backend-loggen viser en linje med koden `[DEBAYER] … needs a CUDA GPU …`, der forklarer årsagen (vist sammen med `chloros-cli --verbose`).
{% endhint %}

***

## Termisk styring

Jetson-enheder har begrænset termisk spillerum, især i lukkede eller luftbårne installationer. Chloros
overvåger SoC-temperaturen og begrænser automatisk batchstørrelserne:

| Temperatur         | Handling                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normal drift — fuld behandlingshastighed          |
| **70 °C** (Advarsel) | Batchstørrelsen reduceres gradvist (100 % → 50 % mellem 70 °C og 80 °C) |
| **80 °C** (Kritisk) | Aggressiv begrænsning (50 % → 0 % mellem 80 °C og 90 °C) |
| **90 °C** (Nedlukning) | Stopper GPU-behandlingen fuldstændigt — afkøling påkrævet |

{% hint style="warning" %}
**Sørg for tilstrækkelig ventilation og varmeafledning** ved vedvarende databehandling, især i lukkede feltkabinetter eller luftbårne systemer. Termisk begrænsning vil reducere databehandlingskapaciteten for at beskytte hardwaren.
{% endhint %}

***

## Hukommelsesstyring

Jetson-enheder bruger **unified memory** — GPU&#x27;en og CPU&#x27;en deler den samme fysiske RAM. Den angivne VRAM (f.eks. ~15,3 GB på en Orin NX 16 GB) er ikke dedikeret GPU-hukommelse; det er den samme RAM, som operativsystemet og alle andre processer bruger.

### Advarsel om swap og anbefalinger

Inden databehandlingen på Jetson påbegyndes, tæller CLI
&#x27;en antallet af RAW-billeder i din inputmappe (`.tif`, `.tiff`, `.raw`, `.dng` — JPG-forhåndsvisninger tælles ikke med), estimerer det maksimale hukommelsesbehov for kørslen og **advarer inden start**, hvis RAM + swap sandsynligvis ikke er tilstrækkeligt. Advarslen har overskriften `LOW MEMORY WARNING - Jetson Detected`, viser antallet af billeder, RAM, den aktuelle swap og det estimerede maksimale behov, hvorefter den giver de nøjagtige kommandoer `fallocate` / `chmod` / `mkswap` / `swapon`, der er tilpasset størrelsen på dit projekt (aldrig mindre end 8 GB). Den holder en pause på et par sekunder, så meddelelsen ikke forsvinder i scrollback, hvorefter behandlingen fortsætter.**Hukommelsesestimater anvendt af advarslen:**

| Debayer-tilstand | Basis | Pr. billede |
| --- | --- | --- |
| Standard | ~1,5 GB | ~10 MB |
| Teksturbevidst | ~2,5 GB (model + Python-kørselstid) | ~15 MB |

Advarslen udløses, når det estimerede maksimale forbrug overstiger RAM + swap minus en sikkerhedsmargen på 1 GB, og den tæller kun **filbaseret** swap — en opsætning, der udelukkende bruger zram, vil stadig blive markeret.

Sådan tilføjes swap manuelt (eksempel: 8 GB):

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->


### Håndtering af OOM (Out of Memory)

Under behandlingen overvåger Chloros
GPU-hukommelsen og nedskalerer gradvist i stedet for at gå ned:

1. Når GPU-hukommelsesudnyttelsen overstiger **85 %**, reduceres batchstørrelserne forebyggende
2. Hvis der alligevel opstår en out-of-memory-hændelse, **halveres** batchstørrelsen og halveres igen ved hver efterfølgende OOM; hver efterfølgende vellykket batch rykker denne begrænsning et trin tilbage
3. Under vedvarende belastning skifter pipelinen fra `fused_gpu` til den hukommelseseffektive `tiled_gpu`-sti og til CPU-behandling som en sidste udvej

Da GPU’en deler Jetsons RAM, sætter RAM plus swap stadig en grænse for et kørsel — ovenstående swap-advarsel er den kontrol, man skal være opmærksom på.

***

## Implementering i felten

### Overvejelser vedrørende strømforbrug

| Jetson-model     | Typisk strømforbrug | Bemærkninger                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | DC-cylinderstik          |
| Jetson Orin NX   | 10–25 W             | DC-cylinderstik          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD eller cylinderstik |

Planlæg dit strømforbrug til vedvarende databehandling — det højeste strømforbrug opstår under den GPU-intensive tråd 3 (databehandling).

### Anbefalinger til lagerplads

* **NVMe SSD** anbefales på det kraftigste til arm64-installationer
* SD-kort er for langsomme til databehandling — brug dem kun som opstartsmedie
* Regn med 2-3 gange størrelsen af dine rå billeddata til den bearbejdede output

### Headless drift via SSH


Chloros
&#x27; CLI
er ideel til headless Jetson-installationer:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Always-On-backend til LATTICE / DAQ-E-tidssynkronisering

Hvis din Jetson styrer LATTICE-kameraer eller DAQ-E-lyssensorer uden skærm, skal du aktivere backend-systemd-tjenesten **som din bruger**, så PTP-grandmasteren kører kontinuerligt, og SDK
-scripts kan finde en kørende backend (enheden er installeret, men ikke aktiveret som standard):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

Backend kører derefter som dig: de optagelser, den skriver, er dine, og den bruger din `chloros-cli login`. Se [Linux
-installation](linux-installation.md#always-on-ptp-for-headless-hosts) for detaljer, herunder root-`chloros-backend.service` og hvordan pakken gør PTP-porte 319/320 bindbare uden root.

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

`chloros-cli process` afsluttes med en værdi forskellig fra nul, når et kørsel, der anmodede om produkter, ikke skriver nogen billeder, så systemds fejlstatus er meningsfuld til overvågning.

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
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI
```

### Python
-SDK
på Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
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
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## Anbefalede Jetson-systemer til brug i felten

Til anvendelse i felten og i luften bør du overveje disse Jetson Orin NX 16 GB-carrierboard-løsninger:

* **Luftbåren/drone**: Systemer med vibrationsklassificering (MIL-STD), letvægtsdesign (under 300 g) og passiv køling
* **Robust feltbrug**: IP67/IP69K-vandtætte kabinetter med PoE GigE-kameraforbindelse
* **Minimal/budget**: Udviklersæt med ekstra kabinetter

Kontakt [MAPIR
-support](https://www.mapir.camera/community/contact) for specifikke hardwareanbefalinger til dit anvendelsesscenarie.

***

## Næste trin

* [Linux
-installation](linux-installation.md) — Generelle oplysninger om Linux
-installation
* [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) — Fuldstændig reference til beregningsstrategier
* [Behandlingspipeline](../processing-architecture/processing-pipeline.md) — Forståelse af 4-tråds-pipeline
* [CLI: Kommandolinje](../CLI.md) — Vejledning til CLI

* [API
: Python
SDK
](../api-python-sdk.md) — Vejledning til SDK

* [CLI
-reference](../reference/cli-reference.md) og [SDK
-reference](../reference/sdk-reference.md) — Omfattende oversigter over kommandoer og API
til version 1.3.0

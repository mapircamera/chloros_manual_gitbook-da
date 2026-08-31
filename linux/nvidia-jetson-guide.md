# NVIDIA Jetson-vejledning

Chloros på NVIDIA Jetson muliggør multispektral billedbehandling i periferien — i felten, på UAV’er og i fjerntliggende installationer. Chloros 1.2.0 registrerer din Jetson-model ved opstart og optimerer behandlingsstrategien i forhold til den fundne hardware. **Der kræves ingen manuel indstilling.**

***

## Understøttede Jetson-modeller

| Model                | RAM            | Behandlingsstrategi                                     | Anbefalet anvendelse                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB delt | `GPU_PARALLEL` (2 arbejdsprocesser)                              | Maksimal ydeevne, store datasæt                      |
| **Jetson Orin NX**   | 8–16 GB delt  | `GPU_PARALLEL` (2 arbejdsprocesser, 16 GB) / `GPU_SINGLE` (8 GB)   | Førstevalget til anvendelse i luften og i felten |
| **Jetson Orin Nano** | 8 GB delt     | `GPU_SINGLE` (1 arbejdsenhed, sekventiel)                     | Edge-computing på indgangsniveau                                 |

{% hint style="info" %}
Linux arm64-pakken kræver **JetPack 6**, som er tilgængeligt på Jetson Orin-familien. Ældre modeller (Nano, TX2, Xavier NX) kan ikke køre JetPack 6 og understøttes ikke af den aktuelle pakke.
{% endhint %}

***

## Krav

* **JetPack 6.x** (seneste version anbefales)
* **NVIDIA CUDA** (inkluderet i JetPack)
* **Betalt Chloros+-abonnement** — Copper-niveau eller højere (kræves for al adgang til CLI/SDK; håndhæves på serversiden)

## Installation

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

For generelle oplysninger om installation af Linux, filplaceringer og fejlfinding, se [Linux Installation](linux-installation.md).

{% hint style="info" %}
**Placer udpakningsmappen på et hurtigt lagringsmedie.** De kompilerede binærfiler udpakker sig selv til en midlertidig mappe ved hver opstart — hvilket er utroligt langsomt fra et SD-kort. Chloros bruger automatisk `/mnt/ssd/tmp`, hvis den findes; ellers skal du indstille `TMPDIR` til en sti på din NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dynamisk beregnings tilpasning på Jetson

### Sådan fungerer det

Ved opstart profilerer Chloros dit system:

1. **Registrerer Jetson-modellen** via `/proc/device-tree/model`
2. **Læser tilgængeligt delt GPU-/CPU-hukommelse** (Jetson bruger samlet hukommelse)
3. **Vælger en behandlingsstrategi** (`GPU_PARALLEL`, `GPU_SINGLE` eller `CPU_PARALLEL`)
4. **Indstiller antal arbejdsprocesser, pipelinetype og hukommelsestildeling** automatisk

Beslutningen afhænger af **den samlede mængde delt RAM**, ikke af modelnavnet:

* **Under 12 GB samlet RAM**(alle 8 GB Jetson-modeller): `GPU_SINGLE` med**1 worker — bevidst sekventiel behandling**. Hukommelsen er for begrænset til samtidige arbejdsprocesser, så billederne behandles ét ad gangen. På Jetson-enheder med**8 GB eller mindre** springer tråd 3 arbejdspuljen helt over og udfører sit arbejde pr. billede i selve processen.
* **12 GB eller mere**(Orin NX 16 GB, AGX Orin): den samlede hukommelse opfylder kravene til `GPU_PARALLEL`, men antallet af arbejdsprocesser er**begrænset til 2 på Jetson** — GPU’en, arbejdsprocessernes RAM og deres CUDA-kontekster pr. arbejdsproces trækker alle på den samme delte pulje, så flere arbejdsprocesser risikerer fejl på grund af manglende hukommelse.

Du kan tilsidesætte det automatiske valg med miljøvariablen `CHLOROS_STRATEGY` — se [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Adfærd pr. model

| Jetson-model                | Strategi       | Arbejdsprocesser | Udførelse                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Sekventiel in-process-loop (`tiled_gpu` ved hukommelsespres) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Sekventiel loop i processen                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Samtidige arbejdsprocesser, `fused_gpu`-sti  |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 2       | Samtidige arbejdsprocesser, `fused_gpu`-sti  |

Den væsentligste forskel mellem platformene er **hukommelsen**. En Jetson med 8 GB skal behandle billeder ét ad gangen ved hjælp af en hukommelseseffektiv tiled-tilgang, når belastningen er høj, mens en Orin med 16 GB eller mere kan køre to billeder gennem GPU&#x27;en samtidigt ved hjælp af den fusionerede pipeline med højere gennemstrømning.

### GPU-budget pr. model

Hver Jetson-model har også en hardwareprofil, der fastsætter, hvor stor en del af den delte pool behandlingen må benytte, og som skalerer batchstørrelserne:

| Model | Loft for GPU-budget | Multiplikator for batchstørrelse | Reserveret til system/skærm |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70 % | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75 % | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80 % | ×1,5 | 4,0 GB |

Den registrerede RAM-kapacitet justerer profilen: En Jetson, der rapporterer **16 GB eller mere**, får sin batchmultiplikator hævet med ×1,2. Den grundlæggende batchstørrelse før multiplikatorer er 8 billeder.

Se [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) for den komplette reference til beregnings tilpasning.

***

## GPU-frekvensbegrænsning for Texture Aware på Nano og Orin Nano

Texture Aware-debayeren kører GPU-neuralt netværksinferens, hvilket kan udløse **overstrømsadvarsler**på Jetson-modeller med lavt strømforbrug (10–15 W-klassen) ved fuld GPU-klokkehastighed. Inden Texture Aware-behandlingen på en**Jetson Nano eller Orin Nano**starter, kontrollerer Chloros GPU’ens maksimale frekvens og begrænser den til**510 MHz** (510000000), hvis den aktuelt er højere:

* Hvis CLI kan skrive til GPU-frekvensens sysfs-node, **anvendes begrænsningen automatisk**, og der vises en bekræftelse.
* Hvis ikke (kræver root-adgang), viser CLI den nøjagtige `sudo`-kommando til at anvende begrænsningen manuelt, venter et øjeblik, så du kan læse den, og fortsætter derefter — behandlingen kører stadig, men der kan vises advarsler om overstrøm.

Sådan indstiller du begrænsningen selv før behandlingen:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modeller med højere effekt (Orin NX 25W, AGX Orin 60W) kører ved fuld GPU-hastighed; der anvendes ingen begrænsning. Standard-debayer udløser aldrig begrænsningen på nogen model.

{% hint style="info" %}
**Texture Aware på Jetson kører altid ét billede ad gangen.** Hver worker ville have brug for sin egen CUDA-kontekst (~1 GB) plus sin egen kopi af denoiser-modellen, hvilket den samlede hukommelse ikke kan klare — derfor er Texture Aware-stien på Jetson fastgjort til en enkelt worker med serialiseret GPU-adgang. Forvent, at Texture Aware vil være markant langsommere end Standard på enhver Jetson.
{% endhint %}

***

## Termisk styring

Jetson-enheder har begrænset termisk spillerum, især i lukkede eller luftbårne installationer. Chloros overvåger SoC-temperaturen og begrænser automatisk batchstørrelserne:

| Temperatur         | Handling                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normal drift — fuld behandlingshastighed          |
| **70 °C** (Advarsel)  | Batchstørrelsen reduceres gradvist (100 % → 50 % mellem 70 °C og 80 °C) |
| **80 °C** (Kritisk) | Aggressiv begrænsning (50 % → 0 % mellem 80 °C og 90 °C) |
| **90 °C** (Nedlukning) | Stopper GPU-behandlingen fuldstændigt — afkøling påkrævet |

{% hint style="warning" %}
**Sørg for tilstrækkelig ventilation og varmeafledning** ved vedvarende databehandling, især i lukkede feltkabinetter eller luftbårne systemer. Termisk begrænsning vil reducere databehandlingskapaciteten for at beskytte hardwaren.
{% endhint %}

***

## Hukommelsesstyring

Jetson-enheder bruger **unified memory** — GPU&#x27;en og CPU&#x27;en deler den samme fysiske RAM. Den angivne VRAM (f.eks. ~15,3 GB på en Orin NX 16 GB) er ikke dedikeret GPU-hukommelse; det er den samme RAM, som operativsystemet og alle andre processer bruger.

### Advarsel om swap og anbefalinger

Inden behandlingen på Jetson tæller CLI antallet af RAW-billeder i din inputmappe (`.tif`, `.tiff`, `.raw`, `.dng` — JPG-forhåndsvisninger tælles ikke med), estimerer det maksimale hukommelsesbehov for kørslen og **advarer inden start**, hvis RAM + swap sandsynligvis ikke er tilstrækkeligt. Advarslen har overskriften `LOW MEMORY WARNING - Jetson Detected`, viser antallet af billeder, RAM, den aktuelle swap og det estimerede maksimale forbrug, hvorefter den angiver de nøjagtige `fallocate` / `chmod` / `mkswap` / `swapon`-kommandoer, der er tilpasset dit projekt (aldrig mindre end 8 GB). Den holder pause i et par sekunder, så meddelelsen ikke forsvinder i scrollback, hvorefter behandlingen fortsætter.**Hukommelsesestimater anvendt af advarslen:**

| Debayer-tilstand | Basis | Pr. billede |
| --- | --- | --- |
| Standard | ~1,5 GB | ~10 MB |
| Teksturbevidst | ~2,5 GB (model + Python-kørselstid) | ~15 MB |

Advarslen udløses, når det estimerede maksimale forbrug overstiger RAM + swap minus en sikkerhedsmargen på 1 GB, og den tæller kun **filbaseret** swap — en opsætning, der udelukkende bruger zram, vil stadig blive markeret.

Sådan tilføjes swap manuelt (eksempel: 8 GB):



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

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
```### Håndtering af OOM (Out of Memory)

Under behandlingen overvåger Chloros GPU-hukommelsen og nedskalerer gradvist i stedet for at gå ned:

1. Når GPU-hukommelsesudnyttelsen overstiger **85 %**, reduceres batchstørrelserne forebyggende
2. Hvis der stadig opstår en out-of-memory-hændelse, **halveres** batchstørrelsen og halveres igen ved hver efterfølgende OOM; hver efterfølgende vellykket batch rykker denne straf et trin tilbage
3. Under vedvarende belastning skifter pipelinen fra `fused_gpu` til den hukommelseseffektive `tiled_gpu`-vej og til CPU-behandling som en sidste udvej

***

## Implementering i praksis

### Overvejelser vedrørende strømforbrug

| Jetson-model     | Typisk strømforbrug | Bemærkninger                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | DC-cylinderstik          |
| Jetson Orin NX   | 10–25 W             | DC-cylinderstik          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD eller stik |

Planlæg dit strømbudget til vedvarende databehandling — det maksimale strømforbrug opstår under den GPU-intensive tråd 3 (databehandling).

### Anbefalinger vedrørende lagring

* **NVMe SSD** anbefales på det kraftigste til arm64-installationer
* SD-kort er for langsomme til databehandling — brug dem kun som opstartsmedie
* Regn med 2–3 gange størrelsen af dine rå billeddata til den behandlede output

### Headless-drift via SSH

Chloros CLI er ideel til headless Jetson-installationer:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Always-On-backend til LATTICE / DAQ-E-tidssynkronisering

Hvis din Jetson styrer LATTICE-kameraer eller DAQ-E-lyssensorer uden skærm, skal du aktivere backend-systemd-tjenesten, så PTP-grandmasteren kører kontinuerligt (enheden er installeret, men ikke aktiveret som standard):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Se [Linux Installation](linux-installation.md#always-on-ptp-for-headless-hosts) for detaljer, herunder hvordan pakken gør PTP-portene 319/320 bindbare uden root-rettigheder.

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

`chloros-cli process` afsluttes med en værdi forskellig fra nul, når et kørsel, der anmodede om produkter, ikke skriver nogen billeder, så systemd&#x27;s fejlstatus er meningsfuld til overvågning.

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
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Anbefalede Jetson-systemer til brug i felten

Til anvendelse i felten og i luften bør du overveje disse Jetson Orin NX 16 GB carrier board-muligheder:

* **Luftbåren/drone**: Systemer med vibrationsklassificering (MIL-STD), letvægtsdesign (under 300 g) og passiv køling
* **Robust feltbrug**: IP67/IP69K vandtætte kabinetter med PoE GigE-kameraforbindelse
* **Minimal/budgetvenlig**: Udviklerkits med tilbehørskabinetter

Kontakt [MAPIR Support](https://www.mapir.camera/community/contact) for specifikke hardwareanbefalinger til netop dit anvendelsesscenarie.

***

## Næste trin

* [Linux-installation](linux-installation.md) — Generelle detaljer om Linux-installation
* [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) — Fuldstændig reference til beregningsstrategier
* [Behandlingspipeline](../processing-architecture/processing-pipeline.md) — Forståelse af 4-tråds-pipeline
* [CLI : Kommandolinje](../CLI.md) — Vejledningen til CLI
* [API : Python SDK](../api-python-sdk.md) — Vejledningen til SDK
* [CLI-reference](../reference/cli-reference.md) og [SDK-reference](../reference/sdk-reference.md) — Udtømmende liste over kommandoer/API for version 1.2.0

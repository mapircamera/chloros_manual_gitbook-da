# Linux-installation

Chloros distribueres til Linux som `.deb`-pakker, der installerer CLI og backend-serveren. Python SDK er en separat pip-pakke (også medfølgende i `.deb` som et wheel, der matcher versionen).

Pakkefilenavnene indeholder versionen og arkitekturen: `chloros_1.2.0_amd64.deb` til x86_64 og `chloros_1.2.0_arm64_jp6.deb` til JetPack 6 Jetson-builds. Erstat med den fil, du rent faktisk har downloadet, i kommandoerne nedenfor.

***

## Linux amd64 (x86_64)

### Systemkrav

| Krav | Minimum | Anbefalet |
| --- | --- | --- |
| **Distribution** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Processor** | x86_64 (Intel/AMD) | Intel Core i7 eller bedre |
| **Hukommelse (RAM)** | 8 GB | 16 GB eller mere |
| **Grafikkort** | Intet (CPU-behandling) | NVIDIA GPU med 4 GB+ VRAM (12 GB+ låser `GPU_PARALLEL` op, 7 GB+ holder Texture Aware ude af single-image-stien) |
| **Lagringsplads** | 2 GB ledig plads | SSD med 10 GB+ ledig plads |
| **Python** | Python 3.7+ (til SDK) | Python 3.10+ |

> **Ubuntu 20.04 og Debian 11 understøttes ikke.** Afhængighedslisten for `.deb` er
> afledt af, hvad Chloros-backend&#x27;et rent faktisk linker til, og det inkluderer
> `libc6 (>= 2.34)`. Både Focal og Bullseye leveres med glibc 2.31, så `apt` afviser
> installationen direkte i stedet for at lade den mislykkes senere under kørsel.

### Installation

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` løser ikke afhængighederne. Hvis der rapporteres om manglende pakker, fuldfører `sudo apt-get install -f` (eller `sudo apt --fix-broken install`) installationen — dette er den normale fremgangsmåde, ikke en fejl.
{% endhint %}

Kontroller installationen:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Systemkrav

| Krav | Minimum | Anbefalet |
| --- | --- | --- |
| **Platform** | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **JetPack** | JetPack 6.x | Seneste JetPack 6 |
| **Hukommelse (RAM)** | 8 GB (delt mellem GPU og CPU) | 16 GB+ delt (12 GB+ er tærsklen for parallelle GPU-arbejdere) |
| **Lagringsplads** | 2 GB ledig plads | NVMe SSD med 10 GB+ ledig plads |
| **Python** | Python 3.7+ (til SDK) | Python 3.10+ |

### Installation

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Samme layout som amd64 `.deb`, med en CUDA-build, der er optimeret til Jetson Orin / Orin NX / Orin Nano. Se [NVIDIA Jetson-vejledningen](nvidia-jetson-guide.md) for oplysninger om Jetsons hukommelse, termiske egenskaber og adfærd ved feltinstallation.

***

## Python SDK-installation (alle Linux)

SDK er en ren Python HTTP-klient til backend, så den samme pakke fungerer på både amd64 og arm64. To kilder:**Fra PyPI** — den offentliggjorte stabile udgave:

```bash
pip install chloros-sdk
```

**Fra den medfølgende wheel-fil** — garanteret kompatibel med den CLI/backend, du netop har installeret (brug denne, hvis din `.deb` er nyere end PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**PEP 668-distributioner** (Ubuntu 23.10+, Debian 12+) tillader ikke systemomfattende pip-installationer. Brug `pip install --user …`, et virtuelt miljø eller `sudo pip install --break-system-packages …`. Pakkeinstallationsprogrammet installerer aldrig automatisk SDK i dit systems Python — det valg er op til dig.
{% endhint %}

Valgfrie ekstrafunktioner:

| Ekstra | Kommando | Tilføjer |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` til streaming af fremskridt i realtid |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` til BLE (DAQ-M)-transport |

Bekræft SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` installerer Chloros, CLI og backend. Python, SDK kommunikerer med denne backend via et lokalt HTTP API (`http://127.0.0.1:5000`) og starter den automatisk, når det er nødvendigt. Brug altid den bogstavelige IPv4-adresse i stedet for `localhost` — `localhost` kan omdirigeres til `::1` og tager cirka to sekunder pr. anmodning.
{% endhint %}

***

## Første opsætning

### 1. Log ind

Adgang til CLI og SDK kræver et betalt Chloros+-abonnement (**Copper** eller højere), hvilket håndhæves på serversiden: en bruger, der er logget ud, får `401 AUTH_REQUIRED`, og en bruger på gratisabonnementet (Iron) får `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Adgangsoplysningerne gemmes i cachen i `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Du skal logge ind igen efter hver installation eller opgradering.** Pakkens `prerm`-script rydder bevidst `~/.chloros/user_session.json` og den cachelagrede licens for alle brugere på maskinen, så en ny build altid revaliderer licensen i stedet for at stole på en forældet cache.
{% endhint %}

### 2. Kontroller din licensstatus

```bash
chloros-cli status
```

`chloros-cli status` fungerer på alle niveauer (inklusive gratis), så du altid kan se, hvorfor adgangen er eller ikke er tilgængelig.

### 3. Kør systemdiagnostik

```bash
chloros-cli selftest
```

Syv kontroller kører i rækkefølge, og kommandoen afsluttes med en værdi forskellig fra nul, hvis en af dem mislykkes:

| # | Kontrol | Hvad den beviser |
| --- | --- | --- |
| 1 | **Version** | CLI angiver sin version (`v1.2.0`). |
| 2 | **Port tilgængelig** | Port 5000 er ledig, *eller* der er allerede svaret fra en velfungerende Chloros-backend (hvilket tæller som bestået). |
| 3 | **Opstart af backend** | Backend-binærfilen startes. |
| 4 | **API-test (`/api/test`)** | Backend svarer med `status: ok`. |
| 5 | **Systemoplysninger** | Udskriver `GPU: <name>, CUDA: <bool>, PyTorch: <version>` fra `/api/system-info`. |
| 6 | **Denoiser-modeller** | Finder `*.pth.enc`-modeller (på Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + støjfjerner**| Texture Aware kan faktisk bruges — kræver CUDA**og** mindst én modelfil. |

Kørslen afsluttes med `N/7 checks passed`, hvor eventuelle fejl vises med navn.

### 4. Behandl dit første datasæt

```bash
chloros-cli process ~/datasets/flight001
```

***

## Filer og mapper

### Pr. bruger

Chloros gemmer sine legitimationsoplysninger og CLI-konfigurationen i en enkelt platformsuafhængig mappe, **`~/.chloros/`** (på Windows, `%USERPROFILE%\.chloros\`). To Linux-specifikke cacher følger i stedet XDG-konventionerne — disse respekterer `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`, når de er angivet.

| Sti | Formål |
| --- | --- |
| `~/.chloros/user_session.json` | Login-session-cache skrevet af `chloros-cli login` (ryddes ved hver pakkeinstallation/opgradering) |
| `~/.chloros/working_directory.txt` | Overskrivning af standardprojektmappe (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | CLI sprogindstilling (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Sprogindstilling, der deles med Windows-brugergrænsefladen — en `language` her har forrang frem for `cli_language.json` |
| `~/.chloros/update_cache.json` | En times cache til Linux/Jetson-opdateringskontrol ved opstart |
| `~/.chloros/backend.log` | Backend-log, når backend blev startet af CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Cachelagrede LATTICE-kalibreringspakker pr. kamera, indekseret efter serienummer og bundt-hash |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Valgfri brugeroverskrivninger for DAQ-kapacitetskorrektionsprofiler |
| `~/.config/chloros/system_config.json` | Cachelagret hardwareprofil fra Dynamic Compute Adaptation — slet den for at tvinge en ny hardwaredetektion |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Backend-serverlogfiler, én fil pr. opstart |
| `~/Chloros Projects/` | Standardprojektmappe, når der ikke er angivet nogen overskrivning |

### Systemdækkende

| Sti | Formål |
| --- | --- |
| `/usr/bin/chloros-cli` | Wrapper-script — indstiller `LD_LIBRARY_PATH` for de medfølgende native biblioteker og kører derefter den egentlige binærfil |
| `/usr/bin/chloros-backend` | Wrapper-script — det samme, plus `CHLOROS_PRODUCTION=1`, så backendens godkendelsesgate aldrig kan deaktivere sig selv i stilhed |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | De kompilerede binære filer |
| `/usr/lib/chloros/arena_runtime/` | Arena SDK-runtime, der kræves af LATTICE-kameraer |
| `/usr/lib/chloros/models/*.pth.enc` | Krypterede støjfjernelsesmodeller, der bruges af Texture Aware-debayeren |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK-hjul, der passer nøjagtigt til denne build |
| `/usr/lib/chloros/exiftool` | Medfølgende exiftool (symlinket til `/usr/local/bin/exiftool` kun hvis der ikke findes et system-exiftool) |
| `/etc/chloros/update.conf` | Konfiguration af opdateringskanal, der læses af `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Indstiller `net.ipv4.ip_unprivileged_port_start = 319`, så backend&#x27;en kan binde PTP-portene uden root-rettigheder |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Peger den dynamiske loader mod `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Giver den loggede bruger adgang til DAQ-U USB-serielbroen (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Tilvalg af altid-aktiv backend-tjeneste (installeret, **ikke aktiveret**) |
| `/usr/share/applications/chloros-cli.desktop` | &quot;Chloros CLI&quot; -post i programmenuen, der åbner en terminal |

## Placering af backend-eksekverbar fil

CLI og SDK registrerer automatisk backend&#x27;en:

| Komponent | Sti |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Overskriv backend-stien med flagget `--backend-exe` CLI eller konstruktørparameteren `backend_exe` SDK, og porten med `--port` (standard `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` peger på **`lattice`**,**`project`**og**`daq pool-*`**-kommandofamilierne på et fjerntliggende backend. Kernekommandoerne (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ignorerer det bevidst og retter sig altid mod `http://127.0.0.1:<port>`.
{% endhint %}

***

## LATTICE-kameraer og DAQ-lyssensorer på Linux

Alle live-hardware-kommandofamilierne fungerer på Linux (amd64 og Jetson):

* **`chloros-cli lattice`** — finder, opretter forbindelse til, konfigurerer og indsamler data fra LATTICE-kameraer og synkroniserede arrays. `.deb` indeholder det Arena SDK-runtime, som de kræver, og registrerer det hos den dynamiske loader.
* **`chloros-cli daq pool-*`** — tilslut DAQ-U/M/E-lyssensorer via backend-puljen, stream kalibrerede spektre og optag `.daq`-filer. Den kompilerede CLI leveres kun med `pool-*`-familien: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — kør et gemt projekt (dets kameraer, sensorer og behandlingsindstillinger) uden brugergrænseflade.
* **`chloros-cli time-sync`** — inspicér den PTP-grandmaster, som Chloros-backend&#x27;et kører for LATTICE-kameraer og DAQ-E-sensorer.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` er en forudsætning for `pool-latest`, `pool-stream`, `pool-record` og `pool-set-cap`; `pool-list` viser de ID&#x27;er, der aktuelt findes i puljen.

{% hint style="info" %}
**Foretræk `--eth-host` til den første DAQ-E-forbindelse på en maskine med flere netværksgrænseflader.** Automatisk opdagelse gennemsøger mDNS og kan overse sensorens grænseflade på grund af en tom ARP-cache, så den første `pool-connect --eth` efter opstart kan mislykkes, selvom sensoren fungerer perfekt. Ved at angive sensorens IP-adresse eller værtsnavn springes opdagelsen helt over.
{% endhint %}

**DAQ-U-serielle tilladelser** håndteres af den installerede udev-regel (`uaccess` + gruppen `dialout`). Hvis en sensor, der allerede var tilsluttet, forbliver utilgængelig, skal du genindlæse reglerne eller tilslutte den igen:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Se [CLI-referencen](../CLI.md) for det fulde kommandosæt.

### Always-On PTP til headless-værter

Ved den første installation genereres systemd-enheden `chloros-backend.service`, men den er **ikke aktiveret**. På en Jetson uden skærm eller en server, der skal holde PTP-tidssynkroniseringen kørende kontinuerligt for DAQ-E-sensorer og LATTICE-kameraer, skal du aktivere den:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Uden denne kører PTP kun, mens Chloros-backend&#x27;et kører — det vil sige under en aktiv CLI/SDK-session.

Enheden binder backend&#x27;en til `127.0.0.1:5000` (miljøindstillingerne `CHLOROS_HOST` / `CHLOROS_PORT` inde i enheden; overskrives med `sudo systemctl edit chloros-backend.service`) og genstarter den ved fejl efter 5 sekunder.

**Hvordan PTP får sine porte.** PTP bruger UDP 319/320, som begge ligger under den normale grænse på 1024 for privilegerede porte. Pakken `postinst` skriver `/etc/sysctl.d/60-chloros-ptp.conf` med `net.ipv4.ip_unprivileged_port_start = 319`, hvilket lader backend-programmet binde dem, mens det kører som din bruger. Det anvender også `setcap cap_net_bind_service,cap_net_raw=+ep` på backend-binærfilen som en ekstra sikkerhedsforanstaltning — det er derfor, at `libcap2-bin` er angivet som en afhængighed for pakken.***

## Eksempler på Bash-scripts

{% hint style="info" %}
**Skriptvenlige afslutningskoder.**`chloros-cli process` afslutter med `0` ved succes og**en værdi forskellig fra nul ved fejl — herunder et kørsel, der anmodede om billedprodukter, men ikke skrev nogen** (det udskriver `Processing finished but wrote no image products.` og angiver projektmappen samt de sædvanlige årsager). Vellykkede kørsler rapporterer, hvor mange billedprodukter der blev skrevet (`Image products written: N`). Afslutningskoder: `0` succes, `1` fejl, `2` argumentfejl, `130` afbrudt.
{% endhint %}

### Behandling af flere datasæt

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Behandling med brugerdefinerede indstillinger

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Der er præcis fire gyldige `--format`-værdier, og de indeholder mellemrum — sæt altid anførselstegn omkring dem:

| `--format`-værdi | Outputmappe |
| --- | --- |
| `TIFF (16-bit)` *(standard)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` accepterer `standard` (standard) eller `texture-aware` (Chloros+).

### Automatiseret behandling med Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Eksempel på Python og SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Fejlfinding

### CLI findes ikke efter installation

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Adgang nægtet

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### &quot;setcap mislykkedes&quot; under installationen

`.deb` anvender `cap_net_bind_service` på `/usr/lib/chloros/chloros-backend`, så det kan binde PTP-portene 319/320 uden root-rettigheder. Hvis `libcap2-bin` manglede under installationen, springes opkaldet over. Installer det, og geninstaller pakken:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP starter ikke / kan ikke binde port 319

Bekræft, at grænsen for porte uden privilegier er sænket, og anvend den igen for den aktuelle opstart, hvis det ikke var tilfældet:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Kontroller derefter grandmasteren:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### &quot;LATTICE-kameradrivere ikke fundet&quot;

Arena SDK-runtime kan ikke løses. Kontroller, at den loader-konfiguration, som pakken skriver, er til stede og opdateret:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### Backend kunne ikke startes

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Backend-logfilerne for den mislykkede opstart findes i `~/.cache/chloros/logs/`.

### CUDA blev ikke registreret

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` rapporterer det samme i én linje: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Manglende delte biblioteker

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Langsom opstart på SD-kort-systemer

De kompilerede binærfiler udpakker sig selv til et midlertidigt bibliotek ved hver opstart. Hvis `/mnt/ssd/tmp` findes, bruger Chloros det automatisk; ellers skal du indstille `TMPDIR` til et hurtigt filsystem:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Opdatering af Chloros på Linux

Kommandoen `update` gælder kun for Linux/Jetson. Den kontrollerer den version, der er offentliggjort i den opdateringskanal, der er konfigureret på `/etc/chloros/update.conf`, og tilbyder at downloade og installere den tilsvarende `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

På Linux/Jetson udfører CLI desuden en ikke-blokerende opdateringskontrol ved hver opstart (resultatet gemmes i cachen i en time i `~/.chloros/update_cache.json`) og viser `Update available: vX.Y.Z`, når der findes en nyere version. Dine indstillinger og projekter bevares efter en opdatering; du skal dog logge ind igen bagefter.

## Afinstallation

```bash
sudo apt remove chloros
```

Afinstallationen stopper `chloros-backend.service`, gendanner standardgrænsen for ikke-privilegerede porte (1024), fjerner symlinket til det medfølgende exiftool og Arena-loader-konfigurationen samt rydder cachelagrede legitimationsoplysninger. Dine projekter og `~/.chloros/`-datafiler forbliver uændrede.

***

## Næste trin

* [NVIDIA Jetson-vejledning](nvidia-jetson-guide.md) — Jetson-specifik optimering og implementering
* [CLI : Kommandolinje](../CLI.md) — vejledningen til CLI
* [API : Python SDK](../api-python-sdk.md) — vejledningen til SDK
* [CLI-reference](../reference/cli-reference.md) og [SDK-reference](../reference/sdk-reference.md) — udtømmende kommando-/API-oversigter for 1.2.0
* [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) — hvordan Chloros tilpasser sig din hardware

# Installation af Linux

Chloros leveres til Linux som `.deb`-pakker, der installerer CLI og backend. Python SDK installeres separat via pip.

***

## Linux amd64 (x86_64)

### Systemkrav

| Krav | Minimum | Anbefalet |
| --- | --- | --- |
| **Distribution** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Processor** | x86_64 (Intel/AMD) | Intel Core i7 eller bedre |
| **Hukommelse (RAM)** | 8 GB | 16 GB eller mere |
| **Grafikkort** | Intet (CPU-behandling) | NVIDIA GPU med 4 GB+ VRAM |
| **Lagringsplads** | 2 GB ledig plads | SSD med 10 GB+ ledig plads |
| **Python** | Python 3.7+ (til SDK) | Python 3.10+ |

### Installation

Download `.deb`-pakken og installer:

```bash
sudo dpkg -i chloros-amd64.deb
```

Kontroller installationen:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Systemkrav

| Krav | Minimum | Anbefalet |
| --- | --- | --- |
| **Platform** | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **JetPack** | JetPack 6.x | Seneste JetPack 6 |
| **Hukommelse (RAM)** | 8 GB (delt GPU/CPU) | 16 GB+ delt |
| **Lagringsplads** | 2 GB ledig plads | NVMe SSD med 10 GB+ ledig |
| **Python** | Python 3.7+ (til SDK) | Python 3.10+ |

### Installation

Download JetPack 6 `.deb`-pakken og installer:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Kontroller installationen:

```bash
chloros-cli --version
```

For detaljeret Jetson-opsætning, herunder termisk styring og implementering i felten, se [NVIDIA Jetson-vejledningen](nvidia-jetson-guide.md).

***

## Python SDK Installation (Alle Linux)

Python SDK installeres separat via pip og fungerer på både amd64 og arm64:

```bash
pip install chloros-sdk
```

For at inkludere valgfri understøttelse af progress streaming:

```bash
pip install chloros-sdk[progress]
```

Bekræft SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb`-pakken installerer Chloros, CLI og backend. Python SDK er en separat pip-pakke, der kommunikerer med backend via en lokal HTTP API.
{% endhint %}

***

## Konfigurationsmapper

Chloros på Linux følger [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Formål | Linux Sti | Windows Tilsvarende |
| --- | --- | --- |
| **Konfiguration** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Data / Projekter** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Cache / Legitimationsoplysninger** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Placeringer for backend-eksekverbare filer

`.deb`-pakken installerer backend&#x27;en på en standardplacering. CLI og SDK registrerer automatisk backend-stien:

| Installationsmetode | Backend-sti |
| --- | --- |
| `.deb`-pakke | `/usr/lib/chloros/chloros-backend` |
| Manuel / brugerdefineret | `/opt/mapir/chloros/backend/chloros-backend` |

Du kan tilsidesætte backend-stien med flaget `--backend-exe` CLI eller konstruktørparameteren `backend_exe` SDK.

***

## Første opsætning

### 1. Aktiver din licens

Der kræves en Chloros+-licens for at få adgang til CLI og SDK:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Kontroller din licensstatus

```bash
chloros-cli status
```

### 3. Behandl dit første datasæt

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Kør systemdiagnostik

Kontroller, at dit system er konfigureret korrekt:

```bash
chloros-cli selftest
```

Dette kører 7 diagnostiske kontroller, herunder version, opstart af backend, API-forbindelse og CUDA/GPU-tilgængelighed.

***

## Eksempler på Bash-scripts

### Behandl flere datasæt

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Behandl med brugerdefinerede indstillinger

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Automatiseret behandling med Cron

Føj til din crontab (`crontab -e`) for automatisk at behandle nye datasæt:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Eksempel

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

### CLI ikke fundet efter installation

Hvis `chloros-cli` ikke findes efter installation af `.deb`-pakken:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Tilladelse nægtet

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
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

### CUDA ikke fundet

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Manglende delte biblioteker

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Opdatering af Chloros på Linux

Brug den indbyggede opdateringskommando til at søge efter og installere opdateringer:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Næste trin

* [NVIDIA Jetson-vejledning](nvidia-jetson-guide.md) — Jetson-specifik optimering og implementering
* [CLI : Kommandolinje](../CLI.md) — Fuld CLI-kommandoreference
* [API : Python SDK](../api-python-sdk.md) — Fuld SDK-reference
* [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) — Hvordan Chloros tilpasser sig din hardware

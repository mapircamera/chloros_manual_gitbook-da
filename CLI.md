# CLI : Kommandolinje

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** giver kraftfuld kommandolinjeadgang til Chloros-billedbehandlingsmotoren, hvilket muliggør automatisering, scripting og headless-drift til dine billedbehandlingsworkflows.

### Nøglefunktioner

* 🚀 **Automatisering** - Script-batchbehandling af flere datasæt
* 🔗 **Integration** - Integrer i eksisterende arbejdsgange og pipelines
* 💻 **Headless-drift** - Kør uden GUI
* 🌍 **Flersproget** - Understøttelse af 38 sprog
* ⚡ **Parallel behandling** - [Dynamisk beregnings tilpasning](processing-architecture/dynamic-compute-adaptation.md) optimerer automatisk til din hardware

### Krav

| Krav          | Detaljer                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operativsystem** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Licens**          | Chloros+ ([kræver betalt abonnement](https://cloud.mapir.camera/pricing)) |
| **Hukommelse**           | Minimum 8 GB RAM (16 GB anbefales)                                  |
| **Internet**         | Kræves til aktivering af licens                                     |
| **Diskplads**       | Varierer afhængigt af projektstørrelse                                              |

{% hint style="warning" %}
**Licenskrav**: CLI kræver et betalt Chloros+-abonnement. Standardabonnementer (gratis) har ikke adgang til CLI. Besøg [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) for at opgradere.
{% endhint %}

## Hurtig start

### Installation

#### Windows

CLI er automatisk inkluderet i Chloros-installationsprogrammet:

1. Download og kør **Chloros Installer.exe**

2. Gennemfør installationsguiden
3. CLI installeret til: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Installationsprogrammet tilføjer automatisk `chloros-cli` til din systems PATH. Genstart din terminal efter installationen.
{% endhint %}

#### Linux

Installer `.deb`-pakken til din arkitektur:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

For detaljeret opsætning af Linux, se [Linux Installation](linux/linux-installation.md).

### Første opsætning

Før du bruger CLI, skal du aktivere din Chloros+-licens:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Grundlæggende brug

Behandl en mappe med standardindstillinger:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Kommandoreference

### Generel syntaks

```
chloros-cli [global-options] <command> [command-options]
```

***

## Kommandoer

### `process` - Behandle billeder

Behandler billeder i en mappe med kalibrering.

**Syntaks:**

```bash
chloros-cli process <input-folder> [options]
```

**Eksempler:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Indstillinger for kommandoen Process

| Indstilling                | Type    | Standard        | Beskrivelse                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Sti    | _Påkrævet_     | Mappe indeholdende RAW/JPG-multispektrale billeder                                         |
| `-o, --output`        | Sti    | Samme som input  | Outputmappe til behandlede billeder                                                     |
| `-n, --project-name`  | Streng  | Autogenereret | Brugerdefineret projektnavn                                                                    |
| `--vignette`          | Flag    | Aktiveret        | Aktiver vignettkorrektion                                                             |
| `--no-vignette`       | Flag    | -              | Deaktiver vignettkorrektion                                                            |
| `--reflectance`       | Flag    | Aktiveret        | Aktiver reflektanskalibrering                                                         |
| `--no-reflectance`    | Flag    | -              | Deaktiver reflektanskalibrering                                                        |
| `--ppk`               | Flag    | Deaktiveret       | Anvend PPK-korrektioner fra .daq-lyssensordata                                      |
| `--format`            | Valg  | TIFF (16-bit)  | Outputformat: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Heltal | Auto           | Mindste målstørrelse i pixels til detektering af kalibreringspanel                          |
| `--target-clustering` | Heltal | Auto           | Tærskel for målgruppering (0-100)                                                    |
| `--debayer`           | Valg  | `standard`     | Debayer-metode: `standard` eller `texture-aware` (kun Chloros+)                          |
| `--target`, `--targets` | Flag  | Deaktiveret       | Søg kun efter kalibreringsmål i en undermappe med navnet &quot;target&quot; eller &quot;targets&quot; (fremskynder behandlingen) |
| `--indices`           | Liste    | Ingen           | Vegetationsindekser, der skal beregnes (f.eks. `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Streng  | Ingen           | Lås eksponering for kameramodel (Pin 1)                                                 |
| `--exposure-pin-2`    | Streng  | Ingen           | Lås eksponering for kameramodel (Pin 2)                                                 |
| `--recal-interval`    | Heltal | Auto           | Rekalibreringsinterval i sekunder                                                      |
| `--timezone-offset`   | Heltal | 0              | Tidszoneforskel i timer                                                               |

***

### `login` - Godkend konto

Log ind med dine Chloros+-loginoplysninger for at aktivere CLI-behandling.

**Syntaks:**

```bash
chloros-cli login <email> <password>
```

**Eksempel:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Specialtegn**: Brug enkelt anførselstegn omkring adgangskoder, der indeholder tegn som `$`, `!` eller mellemrum.
{% endhint %}

**Output:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Slet legitimationsoplysninger

Slet gemte legitimationsoplysninger og log ud af din konto.

**Syntaks:**

```bash
chloros-cli logout
```

**Eksempel:**

```bash
chloros-cli logout
```

**Output:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**SDK-brugere**: Python SDK tilbyder også en programmatisk `logout()`-metode til at slette legitimationsoplysninger i Python-scripts. Se [Python SDK dokumentationen](api-python-sdk.md#logout) for detaljer.
{% endhint %}

***

### `status` - Kontroller licensstatus

Vis aktuel licens- og godkendelsesstatus.

**Syntaks:**

```bash
chloros-cli status
```

**Eksempel:**

```bash
chloros-cli status
```

**Output:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - Kontroller eksportforløb

Overvåg eksportforløbet for tråd 4 under eller efter behandlingen.

**Syntaks:**

```bash
chloros-cli export-status
```

**Eksempel:**

```bash
chloros-cli export-status
```

**Anvendelsestilfælde:** Kald denne kommando, mens behandlingen kører, for at kontrollere eksportforløbet.***

### `language` - Administrer grænsefladesprog

Vis eller ændr grænsefladesproget for CLI.

**Syntaks:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Eksempler:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Understøttede sprog (38 i alt)

| Kode    | Sprog              | Navn på originalsproget      |
| ------- | --------------------- | ---------------- |
| `en`    | Engelsk               | English          |
| `es`    | Spansk               | Español          |
| `pt`    | Portugisisk            | Português        |
| `fr`    | Fransk                | Français         |
| `de`    | Tysk                | Deutsch          |
| `it`    | Italiensk              | Italiano         |
| `ja`    | Japansk              | 日本語              |
| `ko`    | Koreansk                | 한국어              |
| `zh`    | Kinesisk (forenklet)  | 简体中文             |
| `zh-TW` | Kinesisk (traditionelt) | 繁體中文             |
| `ru`    | Russisk               | Русский          |
| `nl`    | Hollandsk                | Nederlands       |
| `ar`    | Arabisk                | العربية          |
| `pl`    | Polsk                | Polski           |
| `tr`    | Tyrkisk               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonesisk            | Bahasa Indonesia |
| `vi`    | Vietnamesisk            | Tiếng Việt       |
| `th`    | Thai                  | ไทย              |
| `sv`    | Svensk               | Svenska          |
| `da`    | Dansk                | Dansk            |
| `no`    | Norsk             | Norsk            |
| `fi`    | Finsk               | Suomi            |
| `el`    | Græsk                 | Ελληνικά         |
| `cs`    | Tjekkisk                | Čeština          |
| `hu`    | Ungarsk             | Magyar           |
| `ro`    | Rumænsk              | Română           |
| `uk`    | Ukrainsk             | Українська       |
| `pt-BR` | Brasiliansk portugisisk  | Português Brasileiro |
| `zh-HK` | Kantonesisk             | 粵語             |
| `ms`    | Malaysisk                 | Bahasa Melayu    |
| `sk`    | Slovakisk                | Slovenčina       |
| `bg`    | Bulgarsk             | Български        |
| `hr`    | Kroatisk              | Hrvatski         |
| `lt`    | Litauisk            | Lietuvių         |
| `lv`    | Lettisk               | Latviešu         |
| `et`    | Estisk              | Eesti            |
| `sl`    | Slovensk             | Slovenščina      |

{% hint style="success" %}
**Automatisk lagring**: Din sprogindstilling gemmes i `~/.chloros/cli_language.json` og bevares på tværs af alle sessioner.
{% endhint %}

***

### `set-project-folder` - Indstil standardprojektmappe

Ændr placeringen af standardprojektmappen (deles med GUI på Windows).

**Syntaks:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Eksempler:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - Vis projektmappe

Vis den aktuelle placering af standardprojektmappen.

**Syntaks:**

```bash
chloros-cli get-project-folder
```

**Eksempel:**

```bash
chloros-cli get-project-folder
```

**Output:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - Nulstil til standard

Nulstil projektmappen til standardplaceringen.

**Syntaks:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - Kør systemdiagnostik

Kør 7 diagnostiske kontroller for at verificere din systemkonfiguration.

**Syntaks:**

```bash
chloros-cli selftest
```

**Udførte diagnostiske tests:**

1. Versionskontrol
2. Porttilgængelighed (5000)
3. Opstart af backend
4. API-forbindelsestest
5. Systemoplysninger og GPU-detektion
6. Verifikation af støjfjernelsesmodeller
7. Kontrol af CUDA-tilgængelighed

{% hint style="info" %}
**Nyttigt til fejlfinding**: Kør `selftest` efter installationen for at kontrollere, at dit system er konfigureret korrekt, især på Linux/Jetson, hvor GPU- og CUDA-opsætningen muligvis skal verificeres.
{% endhint %}

***

### `update` - Søg efter opdateringer (kun Linux)

Søg efter og installer CLI-opdateringer på Linux-systemer.

**Syntaks:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Indstilling    | Beskrivelse                        |
| --------- | ---------------------------------- |
| `--check` | Søg kun efter opdateringer, installer ikke |

{% hint style="info" %}
Denne kommando er kun tilgængelig på Linux. På Windows leveres opdateringer via installationsprogrammet.
{% endhint %}

***

## Globale indstillinger

Disse indstillinger gælder for alle kommandoer:

| Indstilling            | Type    | Standard       | Beskrivelse                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Sti    | Registreres automatisk | Sti til backend-eksekverbar fil                       |
| `--port`          | Heltal | 5000          | Backend API portnummer                          |
| `--restart`       | Flag    | -             | Tving genstart af backend (afslutter eksisterende processer) |
| `--version`       | Flag    | -             | Vis versionsoplysninger og afslut                |
| `--help`          | Flag    | -             | Vis hjælpeoplysninger og afslut                   |

{% hint style="info" %}
**Automatisk registrering af backend**: Stien `--backend-exe` registreres automatisk pr. platform:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manuelt)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Eksempel med globale indstillinger:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Vejledning til behandlingsindstillinger

### Parallel behandling og dynamisk beregnings tilpasning

Chloros 1.1.0 inkluderer [Dynamisk beregnings tilpasning](processing-architecture/dynamic-compute-adaptation.md) — behandlingsmotoren **registrerer automatisk din hardware** og vælger den optimale strategi:

| Platform | Strategi | Arbejdere | Pipeline | Bemærkninger |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Hukommelseseffektiv, serialiseret |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Samtidig GPU-behandling |
| **Desktop med 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | God desktop-ydeevne |
| **Desktop med 12 GB+ GPU** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Optimal desktop-ydeevne |
| **System kun med CPU** | `CPU_PARALLEL` | kerner - 1 | `cpu_fallback` | Ingen GPU påkrævet |

{% hint style="success" %}
**Ingen manuel konfiguration nødvendig!** Chloros registrerer automatisk din CPU, GPU, RAM og (på Jetson) termiske sensorer og konfigurerer derefter automatisk den optimale behandlingspipeline.
{% endhint %}

### Debayer-metoder

| Metode | CLI-flag | Kvalitet | Hastighed | Licens |
| --- | --- | --- | --- | --- |
| **Standard (Hurtig, Medium kvalitet)** | `--debayer standard` | God | Hurtig | Gratis / Chloros+ |
| **Teksturbevidst (langsom, højeste kvalitet)** | `--debayer texture-aware` | Højeste | Langsom | Kun Chloros+ |

Standardmetoden til debayering er **Standard**. Metoden**Teksturbevidst** bruger en AI/ML-støjfjernelsesmodel for at opnå den højeste kvalitet, men kræver en Chloros+-licens og en NVIDIA GPU.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Vignettekorrektion

**Hvad det gør:** Korrigerer lysfald ved billedkanterne (mørkere hjørner, som er almindelige i kamerabilleder).

* **Aktiveret som standard** – De fleste brugere bør holde denne funktion aktiveret
* Brug `--no-vignette` for at deaktivere

{% hint style="success" %}
**Anbefaling**: Aktivér altid vignettekorrektion for at sikre ensartet lysstyrke i hele billedet.
{% endhint %}

### Reflektanskalibrering

Konverterer rå sensorværdier til standardiserede reflektansprocenter ved hjælp af kalibreringspaneler.

* **Aktiveret som standard** – Afgørende for vegetationsanalyse
* Kræver kalibreringsmålpaneler i billederne
* Brug `--no-reflectance` til at deaktivere

{% hint style="info" %}
**Krav**: Sørg for, at kalibreringspanelerne er korrekt eksponeret og synlige i dine billeder for nøjagtig reflektanskonvertering.
{% endhint %}

### PPK-korrektioner

**Hvad det gør:** Anvender post-processerede kinematiske korrektioner ved hjælp af DAQ-A-SD-logdata for forbedret GPS-nøjagtighed.

* **Deaktiveret som standard**
* Brug `--ppk` for at aktivere
* Kræver .daq-filer i projektmappen fra MAPIR DAQ-A-SD-lyssensor.

### Outputformater

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Bitdybde</th><th width="116.5999755859375">Filstørrelse</th><th>Bedst egnet til</th></tr></thead><tbody><tr><td><strong>TIFF (16-bit)</strong> ⭐</td><td>16-bit heltal</td><td>Stor</td><td>GIS-analyse, fotogrammetri (anbefales)</td></tr><tr><td><strong>TIFF (32-bit, procent)</strong></td><td>32-bit flydende</td><td>Meget stor</td><td>Videnskabelig analyse, forskning</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>8-bit heltal</td><td>Mellem</td><td>Visuel inspektion, deling på nettet</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>8-bit heltal</td><td>Lille</td><td>Hurtig forhåndsvisning, komprimeret output</td></tr></tbody></table>***

## Automatisering og scripting

### PowerShell-batchbehandling (Windows)

Behandl flere datasætmapper automatisk på Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Windows Batch-script (Windows)

Enkel loop til batchbehandling på Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Bash-batchbehandling (Linux)

Behandl flere datasætmapper på Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Python-automatiseringsscript (platformuafhængigt)

Avanceret automatisering med fejlhåndtering (fungerer på Windows og Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Behandlingsworkflow

### Standardworkflow

1. **Indgang**: Mappe indeholdende RAW/JPG-billedpar
2. **Opdagelse**: CLI scanner automatisk efter understøttede billedfiler
3. **Behandling**: Parallel tilstand skaleres til dine CPU-kerner (Chloros+)
4. **Output**: Opretter undermapper for kameramodeller med behandlede billeder

### Eksempel på outputstruktur

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Estimater for behandlingstid

Typiske behandlingstider for 100 billeder (12 MP hver):

| Platform | Tilstand | Estimeret tid | Bemærkninger |
| --- | --- | --- | --- |
| **Desktop 12 GB+ GPU** | `GPU_PARALLEL` | 5-10 min | Hurtigste mulighed |
| **Desktop 8 GB GPU** | `GPU_SINGLE` | 10-15 min | God ydeevne |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 15-25 min | Edge-computing |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 30-60 min | Begrænset hukommelse |
| **Kun CPU** | `CPU_PARALLEL` | 20-40 min | Ingen GPU påkrævet |

{% hint style="info" %}
**Tip til ydeevne**: Behandlingstiden varierer afhængigt af antal billeder, opløsning, debayer-metode og hardware. Texture Aware-debayer tager betydeligt længere tid end Standard. Se [Dynamisk beregnings tilpasning](processing-architecture/dynamic-compute-adaptation.md) for detaljer.
{% endhint %}

***

## Fejlfinding

### CLI ikke fundet

**Windows-fejl:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Løsninger:**

1. Kontroller installationsplaceringen:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Brug den fulde sti, hvis den ikke er i PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Tilføj manuelt til PATH:
   * Åbn Systemegenskaber → Miljøvariabler
   * Rediger PATH-variablen
   * Tilføj: `C:\Program Files\Chloros\resources\cli`
   * Genstart terminalen

**Linux Fejl:**

```
chloros-cli: command not found
```

**Linux Løsninger:**

1. Kontroller installationen:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Genindlæs din shell:

```bash
source ~/.bashrc
```

3. Kontroller tilladelser:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Backend kunne ikke startes**Fejl:**

```

Backend failed to start within 30 seconds
```

**Løsninger:**

1. Kontroller, om backend allerede kører (luk den først)
2. Kontroller, at firewallen ikke blokerer (Windows), eller kontroller porttilgængeligheden (Linux: `lsof -i :5000`)
3. Prøv en anden port:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Tving backend til at genstarte:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. På Linux skal du kontrollere, om backend-eksekverbarfilen findes:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Problemer med licens/godkendelse**Fejl:**

```

Chloros+ license required for CLI access
```

**Løsninger:**

1. Kontroller, at du har et aktivt Chloros+-abonnement
2. Log ind med dine loginoplysninger:

```bash
chloros-cli login user@example.com 'password'
```

3. Kontroller licensstatus:

```bash
chloros-cli status
```

4. Kontakt support: info@mapir.camera

***

### Ingen billeder fundet**Fejl:**

```

No images found in the specified folder
```

**Løsninger:**

1. Kontroller, at mappen indeholder understøttede formater (.RAW, .TIF, .JPG)
2. Kontroller, at mappestien er korrekt (brug anførselstegn for stier med mellemrum)
3. Sørg for, at du har læseadgang til mappen
4. Kontroller, at filtypenavnene er korrekte

***

### Behandlingen går i stå eller hænger**Løsninger:**

1. Kontroller ledig diskplads (sørg for, at der er nok til output)
2. Luk andre programmer for at frigøre hukommelse
3. Reducer antallet af billeder (behandl i batches)

***

### Porten er allerede i brug**Fejl:**

```

Port 5000 is already in use
```

**Løsninger:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## Ofte stillede spørgsmål

### Spørgsmål: Har jeg brug for en licens til CLI?

**Svar:**Ja! CLI kræver en betalt**Chloros+-licens**.

* ❌ Standard (gratis) abonnement: CLI deaktiveret
* ✅ Chloros+ (betalt) abonnementer: CLI fuldt aktiveret

Abonner på: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Spørgsmål: Kan jeg bruge CLI på en server uden GUI?**Svar:** Ja! CLI kører fuldstændigt headless. Dette er den primære anvendelse på Linux.**Windows-server:**
* Windows Server 2016 eller nyere
* Visual C++ Redistributable installeret

**Linux Server:**
* Ubuntu 20.04+ / Debian 11+ (amd64) eller JetPack 6 (arm64)
* Installation via `.deb`-pakken

**Begge platforme:**
* Minimum 8 GB RAM (16 GB anbefales)
* Engangsaktivering af licens: `chloros-cli login user@example.com 'password'`

***

### Spørgsmål: Hvor gemmes de behandlede billeder?**Svar:**Som standard gemmes de behandlede billeder i**samme mappe som input** i undermapper for kameramodeller (f.eks. `Survey3N_RGN/`).

Brug `-o`-indstillingen til at angive en anden outputmappe:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### Spørgsmål: Kan jeg behandle flere mapper på én gang?**A:** Ikke direkte med én kommando, men du kan bruge scripting til at behandle mapper sekventielt. Se afsnittet [Automatisering og scripting](CLI.md#automation--scripting).***

### Spørgsmål: Hvordan gemmer jeg CLI-output i en logfil?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### Spørgsmål: Hvad sker der, hvis jeg trykker på Ctrl+C under behandlingen?**Svar:** CLI vil:

1. Afslutte behandlingen på en ordentlig måde
2. Lukke backend-systemet ned
3. Afslutte med kode 130

Delvist behandlede billeder kan forblive i output-mappen.

***

### Spørgsmål: Kan jeg automatisere CLI-behandlingen?**Svar:** Absolut! CLI er designet til automatisering. Se [Automatisering og scripting](CLI.md#automation--scripting) for PowerShell (Windows), Batch (Windows), Bash (Linux) og Python (platformuafhængige) eksempler.***

### Spørgsmål: Hvordan tjekker jeg CLI-versionen?**Svar:**

```bash
chloros-cli --version
```

**Output:**

```

Chloros CLI 1.1.0
```

***

## Få hjælp

### Hjælp til kommandolinjen

Se hjælpeoplysninger direkte i CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Supportkanaler

* **E-mail**: info@mapir.camera
* **Websted**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Priser**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Komplette eksempler

### Eksempel 1: Grundlæggende behandling

Behandling med standardindstillinger (vignette, reflektans):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Eksempel 2: Videnskabeligt output i høj kvalitet

32-bit float TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Eksempel 3: Hurtig forhåndsvisning

8-bit PNG uden kalibrering til hurtig gennemgang:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Eksempel 4: PPK-korrigeret behandling

Anvend PPK-korrektioner med reflektans:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Eksempel 5: Brugerdefineret outputplacering

Behandl til en anden placering med specifikt format:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Eksempel 6: Godkendelsesworkflow

Komplet godkendelsesflow (det samme på alle platforme):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Eksempel 7: Brug af flere sprog

Skift sprog i brugergrænsefladen (det samme på alle platforme):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```

# API : Python SDK

**Chloros Python SDK** giver programmatisk adgang til Chloros-billedbehandlingsmotoren, hvilket muliggør automatisering, tilpassede arbejdsgange og problemfri integration med dine Python-applikationer og forskningspipelines.

### Nøglefunktioner

* 🐍 **Native Python** - Ren, Pythonic API til billedbehandling
* 🔧 **Fuld API-adgang** - Fuld kontrol over Chloros-behandlingen
* 🚀 **Automatisering** - Opbyg brugerdefinerede arbejdsgange til batchbehandling
* 🔗 **Integration** - Integrer Chloros i eksisterende Python-applikationer
* 📊 **Klar til forskning** - Perfekt til videnskabelige analysepipelines
* ⚡ **Parallel behandling** - Skalerer efter dine CPU-kerner (Chloros+)

### Krav

| Krav          | Detaljer                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros installeret** | Windows: Desktop-installationsprogram; Linux: `.deb`-pakke                  |
| **Licens**          | Chloros+ ([betalt abonnement kræves](https://cloud.mapir.camera/pricing)) |
| **Operativsystem** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 eller nyere                                                |
| **Hukommelse**           | Minimum 8 GB RAM (16 GB anbefales)                                  |
| **Internet**         | Kræves til aktivering af licens                                     |

{% hint style="warning" %}
**Licenskrav**: Python SDK kræver et betalt Chloros+-abonnement for at få adgang til API. Standardabonnementer (gratis) har ikke adgang til API/SDK. Besøg [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) for at opgradere.
{% endhint %}

## Hurtigstart

### Installation

Installer via pip:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**Første opsætning**: Før du bruger SDK, skal du aktivere din Chloros+-licens ved at åbne Chloros, Chloros (Browser) eller Chloros CLI og logge ind med dine loginoplysninger. Dette behøver kun at gøres én gang. På Linux (ingen GUI) skal du bruge: `chloros-cli login user@example.com 'password'`
{% endhint %}

### Grundlæggende brug

Behandl en mappe med blot et par linjer:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**Platformsuafhængige stier**: Kodeeksemplerne på denne side bruger stier i stil med Windows (f.eks. `C:\\DroneImages\\Flight001`). På Linux skal du i stedet bruge Linux-stier (f.eks. `/home/user/drone_images/flight001` eller `~/drone_images/flight001`). SDK fungerer på samme måde på begge platforme.
{% endhint %}

### Fuld kontrol

Til avancerede arbejdsgange:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Installationsvejledning

### Forudsætninger

Før du installerer SDK, skal du sikre dig, at du har:

1. **Chloros installeret** — Windows: Desktop-installationsprogram ([download](download.md)); Linux: `.deb`-pakke ([Linux Installation](linux/linux-installation.md))
2. **Python 3.7+** installeret ([python.org](https://www.python.org))
3. **Aktiv Chloros+-licens** ([opgradering](https://cloud.mapir.camera/pricing))

### Installation via pip

**Standardinstallation:**

```bash
pip install chloros-sdk
```

**Med understøttelse af fremskridtsovervågning:**

```bash
pip install chloros-sdk[progress]
```

**Udviklingsinstallation:**

```bash
pip install chloros-sdk[dev]
```

### Bekræft installation

Test, at SDK er installeret korrekt:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Første opsætning

### Licensaktivering

SDK bruger den samme licens som Chloros, Chloros (Browser) og Chloros CLI. Aktivér én gang via GUI&#x27;en eller CLI:**Windows:**Åbn**Chloros eller Chloros (Browser)** og log ind på fanen Bruger <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> eller brug CLI.**Linux:** Brug CLI (ingen GUI tilgængelig):

```bash
chloros-cli login user@example.com 'your_password'
```

Licensen gemmes lokalt og bevares ved genstart.

{% hint style="success" %}
**Engangsopsætning**: Efter login via GUI&#x27;en eller CLI bruger SDK automatisk den cachelagrede licens. Der kræves ingen yderligere godkendelse!
{% endhint %}

{% hint style="info" %}
**Logout**: SDK-brugere kan programmatisk rydde cachelagrede legitimationsoplysninger ved hjælp af `logout()`-metoden. Se [logout()-metoden](#logout) i API-referencen.
{% endhint %}

### Test forbindelse

Kontroller, at SDK kan oprette forbindelse til Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## API-reference

### ChlorosLocal-klasse

Hovedklasse til lokal Chloros-billedbehandling.

#### Konstruktor

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parametre:**

| Parameter                 | Type | Standard                   | Beskrivelse                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL for lokal Chloros-backend          |
| `auto_start_backend`      | bool | `True`                    | Start backend automatisk, hvis nødvendigt |
| `backend_exe`             | str  | `None` (auto-detect)      | Sti til backend-eksekverbar fil            |
| `timeout`                 | int  | `30`                      | Anmodningstimeout i sekunder            |
| `backend_startup_timeout` | int  | `60`                      | Timeout for opstart af backend (sekunder) |

**Eksempler:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**Automatisk platformdetektering**: SDK prøver automatisk den korrekte backend-sti til din platform:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manuelt)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### Metoder

#### `create_project(project_name, camera=None)`

Opret et nyt Chloros-projekt.

**Parametre:**

| Parameter      | Type | Påkrævet | Beskrivelse                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Ja      | Navn på projektet                                     |
| `camera`       | str  | Nej       | Kameraskabelon (f.eks. &quot;Survey3N\_RGN&quot;, &quot;Survey3W\_OCN&quot;) |

**Returnerer:** `dict` - Svar på oprettelse af projekt**Eksempel:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Importer billeder fra en mappe.

**Parametre:**

| Parameter     | Type     | Påkrævet | Beskrivelse                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Ja      | Sti til mappe med billeder         |
| `recursive`   | bool     | Nej       | Søg i undermapper (standard: False) |

**Returnerer:** `dict` - Importerede resultater med antal filer**Eksempel:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Konfigurer behandlingsindstillinger.

**Parametre:**

| Parameter                 | Type | Standard                 | Beskrivelse                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;Standard (Hurtig, Medium kvalitet)&quot; | Debayer-metode            |
| `vignette_correction`     | bool | `True`                  | Aktiver vignetteringskorrektion      |
| `reflectance_calibration` | bool | `True`                  | Aktiver reflektanskalibrering  |
| `indices`                 | liste | `None`                  | Vegetationsindekser, der skal beregnes |
| `export_format`           | str  | &quot;TIFF (16-bit)&quot;         | Outputformat                   |
| `ppk`                     | bool | `False`                 | Aktiver PPK-korrektioner          |
| `custom_settings`         | dict | `None`                  | Avancerede brugerdefinerede indstillinger        |

**Eksportformater:**

* `"TIFF (16-bit)"` - Anbefales til GIS/fotogrammetri
* `"TIFF (32-bit, Percent)"` - Videnskabelig analyse
* `"PNG (8-bit)"` - Visuel inspektion
* `"JPG (8-bit)"` - Komprimeret output

**Tilgængelige indekser:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 og flere.**Eksempel:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Behandl projektbillederne.

**Parametre:**

| Parameter           | Type     | Standard      | Beskrivelse                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Behandlingsmodus: &quot;parallel&quot; eller &quot;seriel&quot;   |
| `wait`              | bool     | `True`       | Vent på afslutning                       |
| `progress_callback` | callable | `None`       | Tilbagemeldingsfunktion for fremskridt (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Afstemningsinterval for fremskridt (sekunder)   |

**Returnerer:** `dict` - Behandlingsresultater

{% hint style="warning" %}
**Parallel tilstand**: Kræver Chloros+-licens. Skalerer automatisk til dine CPU-kerner (op til 16 arbejdsprocesser).
{% endhint %}

**Eksempel:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Hent den aktuelle projektkonfiguration.

**Returnerer:** `dict` - Aktuel projektkonfiguration**Eksempel:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Henter statusoplysninger om backend, herunder behandlingsstatus pr. tråd.

**Returnerer:** `dict` - Backend-status med følgende struktur:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**Eksempel:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

Lukker backend (hvis startet af SDK).

**Eksempel:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

Rydder cachelagrede legitimationsoplysninger fra det lokale system.

**Beskrivelse:**

Logger ud programmatisk ved at fjerne cachelagrede legitimationsoplysninger. Dette er nyttigt til:
* At skifte mellem forskellige Chloros+-konti
* At rydde legitimationsoplysninger i automatiserede miljøer
* Sikkerhedsformål (f.eks. fjernelse af legitimationsoplysninger før afinstallation)

**Returnerer:** `dict` - Resultat af logout-operation**Eksempel:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**Genautentificering påkrævet**: Efter at have kaldt `logout()` skal du logge ind igen via Chloros, Chloros (Browser) eller Chloros CLI, før du bruger SDK.
{% endhint %}

***

### Hjælpefunktioner

#### `process_folder(folder_path, **options)`

Enkel hjælpefunktion til at behandle en mappe.

**Parametre:**

| Parameter                 | Type     | Standard         | Beskrivelse                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Påkrævet        | Sti til mappe med billeder     |
| `project_name`            | str      | Genereres automatisk  | Projektnavn                   |
| `camera`                  | str      | `None`          | Kameraskabelon                |
| `indices`                 | liste     | `["NDVI"]`      | Indekser, der skal beregnes           |
| `vignette_correction`     | bool     | `True`          | Aktiver vignetteringskorrektion     |
| `reflectance_calibration` | bool     | `True`          | Aktiver reflektanskalibrering |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | Outputformat                  |
| `mode`                    | str      | `"parallel"`    | Behandlingsmodus                |
| `progress_callback`       | callable | `None`          | Status-callback              |

**Returnerer:** `dict` - Behandlingsresultater**Eksempel:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Understøttelse af konteksthåndtering

SDK understøtter konteksthåndtering til automatisk oprydning:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Komplette eksempler

{% hint style="info" %}
**Linux-brugere**: Alle eksempler nedenfor bruger Windows-stier. Erstat `C:\\...`-stier med dine Linux-stier (f.eks. `/home/user/...` eller `~/...`). Alle SDK-funktioner er identiske på tværs af platforme.
{% endhint %}

### Eksempel 1: Grundlæggende behandling

Behandl en mappe med standardindstillinger:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Eksempel 2: Tilpasset arbejdsgang

Fuld kontrol over behandlingspipeline:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Eksempel 3: Batchbehandling af flere mapper

Behandl flere flyvedatasæt:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Eksempel 4: Integration af forskningspipeline

Integrer Chloros med dataanalyse:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Eksempel 5: Tilpasset statusovervågning

Avanceret statusovervågning med logning:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Eksempel 6: Fejlhåndtering

Robust fejlhåndtering til produktionsbrug:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Eksempel 7: Kontoadministration og logout

Administrer legitimationsoplysninger programmatisk:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### Eksempel 8: Kommandolinjeværktøj

Opret et brugerdefineret CLI-værktøj med SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Anvendelse:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## Håndtering af undtagelser

SDK indeholder specifikke undtagelsesklasser for forskellige fejltyper:

### Undtagelseshierarki

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Eksempler på undtagelser

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Avancerede emner

### Brugerdefineret backend-konfiguration

Brug en brugerdefineret backend-placering eller -konfiguration:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Ikke-blokerende behandling

Start behandlingen og fortsæt med andre opgaver:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Hukommelsesstyring

Ved store datasæt skal du behandle i batches:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Fejlfinding

### Backend starter ikke

**Problem:** SDK kan ikke starte backend**Løsninger:**

1. Kontroller, at Chloros er installeret:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Kontroller firewall (Windows) eller porttilgængelighed (Linux: `lsof -i :5000`)
3. Prøv manuel backend-sti:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### Licens ikke fundet**Problem:** SDK advarer om manglende licens**Løsninger:**

1. Åbn Chloros, Chloros (Browser) eller Chloros CLI og log ind.
2. Kontroller, at licensen er gemt i cachen:

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. Hvis du oplever problemer med legitimationsoplysningerne, skal du rydde de cachelagrede legitimationsoplysninger og logge ind igen:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Kontakt support: info@mapir.camera

***

### Importfejl**Problem:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Løsninger:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Behandlings-timeout**Problem:** Behandlingen går i timeout**Løsninger:**

1. Forøg timeout:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Behandl mindre batcher
3. Kontroller ledig diskplads
4. Overvåg systemressourcer

***

### Port allerede i brug**Problem:** Backend-port 5000 optaget**Løsninger:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Eller find og luk den konfliktende proces:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
```

***

## Tips til ydeevne

### Optimér behandlingshastigheden

1. **Brug parallel tilstand** (kræver Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Reducer outputopløsningen** (hvis det er acceptabelt)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Deaktiver unødvendige indekser**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Behandl på SSD** (ikke HDD)***

### Hukommelsesoptimering

For store datasæt:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Baggrundsbehandling

Frigør Python til andre opgaver:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Integrationseksempler

### Django-integration

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## Ofte stillede spørgsmål

### Spørgsmål: Kræver SDK en internetforbindelse?

**Svar:** Kun til den første aktivering af licensen. Efter at have logget ind via Chloros, Chloros (Browser) eller Chloros CLI gemmes licensen lokalt og fungerer offline i 30 dage.***

### Spørgsmål: Kan jeg bruge SDK på en server uden GUI?**Svar:** Ja! SDK fungerer headless på både Windows- og Linux-servere.**Linux (anbefales til headless):**
* Installer via `.deb`-pakken
* Aktiver licens: `chloros-cli login user@example.com 'password'`

**Windows-server:**
* Windows Server 2016 eller nyere
* Chloros installeret (engangs)
* Licens aktiveret via CLI eller på en hvilken som helst maskine

***

### Spørgsmål: Hvad er forskellen mellem Desktop, CLI og SDK?

| Funktion         | Desktop GUI | CLI Kommandolinje | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Grænseflade**   | Peg-og-klik | Kommando          | Python API  |
| **Bedst egnet til**    | Visuelt arbejde | Scripting        | Integration |
| **Automatisering**  | Begrænset     | God             | Fremragende   |
| **Fleksibilitet** | Grundlæggende       | God             | Maksimal     |
| **Licens**     | Chloros+    | Chloros+         | Chloros+    |***

### Spørgsmål: Kan jeg distribuere apps, der er bygget med SDK?**Svar:** SDK-kode kan integreres i dine applikationer, men:

* Slutbrugere skal have Chloros installeret
* Slutbrugere skal have aktive Chloros+-licenser
* Kommerciel distribution kræver OEM-licens

Kontakt info@mapir.camera for spørgsmål vedrørende OEM.

***

### Spørgsmål: Hvordan opdaterer jeg SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### Spørgsmål: Hvor gemmes de behandlede billeder?

Som standard i projektstien:

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### Spørgsmål: Kan jeg behandle billeder fra Python-scripts, der kører efter en tidsplan?**Svar:** Ja! Brug din OS-planlægger med Python-scripts:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** Planlæg via Opgavestyring til at køre dagligt.**Linux:** Planlæg via cron:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### Spørgsmål: Understøtter SDK async/await?**Svar:** Den aktuelle version er synkron. For asynkron adfærd skal du bruge `wait=False` eller køre i en separat tråd:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### Spørgsmål: Hvordan skifter jeg mellem forskellige Chloros+-konti?**Svar:** Brug metoden `logout()` til at rydde cachelagrede legitimationsoplysninger, og log derefter ind igen med den nye konto:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

Efter udlogning skal du godkende med den nye konto via GUI, browser eller CLI, før du bruger SDK igen.

***

## Få hjælp

### Dokumentation

* **API-reference**: Denne side

### Supportkanaler

* **E-mail**: info@mapir.camera
* **Websted**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Priser**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Eksempelkode

Alle eksempler, der er angivet her, er testet og klar til brug. Kopier og tilpas dem til dit brugsscenarie.

***

## Licens**Proprietær software** - Copyright (c) 2025 MAPIR Inc.

SDK kræver et aktivt Chloros+-abonnement. Uautoriseret brug, distribution eller ændring er forbudt.

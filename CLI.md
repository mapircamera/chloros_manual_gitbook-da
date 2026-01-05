# CLI : Kommandolinje

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** giver kraftfuld kommandolinjeadgang til Chloros billedbehandlingsmotor, hvilket muliggør automatisering, scripting og headless-drift til dine billedbehandlingsworkflows.

### Nøglefunktioner

* 🚀 **Automatisering** - Skriptbatchbehandling af flere datasæt
* 🔗 **Integration** - Integrer i eksisterende arbejdsgange og pipelines
* 💻 **Headless-drift** - Kør uden GUI
* 🌍 **Flere sprog** - Understøttelse af 38 sprog
* ⚡ **Parallel behandling** – Skaleres dynamisk til din CPU (op til 16 parallelle arbejdere)

### Krav

| Krav          | Detaljer                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operativsystem** | Windows 10/11 (64-bit)                                              |
| **Licens**          | Chloros+ ([betalt abonnement kræves](https://cloud.mapir.camera/pricing)) |
| **Hukommelse**           | Minimum 8 GB RAM (16 GB anbefales)                                  |
| **Internet**         | Kræves til aktivering af licens                                     |
| **Diskplads**       | Varierer afhængigt af projektets størrelse                                              |

{% hint style=&quot;warning&quot; %}
**Licenskrav**: CLI kræver et betalt Chloros+ abonnement. Standardabonnementer (gratis) har ikke adgang til CLI. Besøg [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) for at opgradere.
{% endhint %}

## Hurtig start

### Installation

CLI er automatisk inkluderet i Chloros-installationsprogrammet:

1. Download og kør **Chloros Installer.exe**

2. Gennemfør installationsguiden
3. CLI installeret til: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Installationsprogrammet tilføjer automatisk `chloros-cli` til din systems PATH. Genstart din terminal efter installationen.
{% endhint %}

### Første opsætning

Inden du bruger CLI, skal du aktivere din Chloros+-licens:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Grundlæggende brug

Behandl en mappe med standardindstillinger:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Kommandoreference

### Generel syntaks

```
chloros-cli [global-options] <command> [command-options]
```

***

## Kommandoer

### `process` - Behandl billeder

Behandl billeder i en mappe med kalibrering.

**Syntaks:**

```bash
chloros-cli process <input-folder> [options]
```

**Eksempel:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Behandlingskommandoindstillinger

| Indstilling                | Type    | Standard        | Beskrivelse                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Sti    | _Påkrævet_     | Mappe, der indeholder RAW/JPG multispektrale billeder                                         |
| `-o, --output`        | Sti    | Samme som input  | Outputmappe til behandlede billeder                                                     |
| `-n, --project-name`  | Streng  | Autogenereret | Brugerdefineret projektnavn                                                                    |
| `--vignette`          | Flag    | Aktiveret        | Aktiver vignettekorrektion                                                             |
| `--no-vignette`       | Flag    | -              | Deaktiver vignettekorrektion                                                            |
| `--reflectance`       | Flag    | Aktiveret        | Aktiver reflektanskalibrering                                                         |
| `--no-reflectance`    | Flag    | -              | Deaktiver reflektanskalibrering                                                        |
| `--ppk`               | Flag    | Deaktiveret       | Anvend PPK-korrektioner fra .daq-lyssensordata                                      |
| `--format`            | Valg  | TIFF (16-bit)  | Outputformat: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Heltal | Auto           | Mindste målstørrelse i pixels til kalibreringspaneldetektion                          |
| `--target-clustering` | Heltal | Auto           | Tærskel for målklyngedannelse (0-100)                                                    |
| `--exposure-pin-1`    | Streng  | Ingen           | Lås eksponering for kameramodel (Pin 1)                                                 |
| `--exposure-pin-2`    | Streng  | Ingen           | Lås eksponering for kameramodel (Pin 2)                                                 |
| `--recal-interval`    | Heltal | Auto           | Rekalibreringsinterval i sekunder                                                      |
| `--timezone-offset`   | Heltal | 0              | Tidszoneforskydning i timer                                                               |

***

### `login` - Godkend konto

Log ind med dine Chloros+-legitimationsoplysninger for at aktivere CLI-behandling.

**Syntaks:**

```bash
chloros-cli login <email> <password>
```

**Eksempel:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Specialtegn**: Brug enkelt anførselstegn omkring adgangskoder, der indeholder tegn som `$`, `!` eller mellemrum.
{% endhint %}

**Output:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Ryd legitimationsoplysninger

Ryd gemte legitimationsoplysninger og log ud af din konto.

**Syntaks:**

```bash
chloros-cli logout
```

**Eksempel:**

```powershell
chloros-cli logout
```

**Output:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style=&quot;info&quot; %}
**SDK Brugere**: Python SDK leverer også en programmatisk `logout()` metode til at rydde legitimationsoplysninger inden for Python scripts. Se [Python SDK dokumentationen](api-python-sdk.md#logout) for detaljer.
{% endhint %}

***

### `status` - Kontroller licensstatus

Vis den aktuelle licens- og godkendelsesstatus.

**Syntaks:**

```bash
chloros-cli status
```

**Eksempel:**

```powershell
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

### `export-status` - Kontroller eksportstatus

Overvåg eksportstatus for tråd 4 under eller efter behandlingen.

**Syntaks:**

```bash
chloros-cli export-status
```

**Eksempel:**

```powershell
chloros-cli export-status
```

**Anvendelsestilfælde:** Kald denne kommando, mens behandlingen kører, for at kontrollere eksportens fremskridt.***

### `language` - Administrer grænsefladesprog

Vis eller ændr CLI-grænsefladesproget.

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

```powershell
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

| Kode    | Sprog              | Indfødt navn      |
| ------- | --------------------- | ---------------- |
| `en`    | Engelsk               | English          |
| `es`    | Spansk               | Español          |
| `pt`    | Portugisisk            | Português        |
| `fr`    | Fransk                | Français         |
| `de`    | Tysk                | Deutsch          |
| `it`    | Italiensk               | Italiano         |
| `ja`    | Japansk              | 日本語              |
| `ko`    | Koreansk                | 한국어              |
| `zh`    | Kinesisk (forenklet)  | 简体中文             |
| `zh-TW` | Kinesisk (traditionelt) | 繁體中文             |
| `ru`    | Russisk               | Русский          |
| `nl`    | Hollandsk                 | Nederlands       |
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
| `cs`    | Tjekkisk                 | Čeština          |
| `hu`    | Ungarsk             | Magyar           |
| `ro`    | Rumænsk              | Română           |
| `uk`    | Ukrainsk             | Українська       |
| `pt-BR` | Brasiliansk portugisisk  | Português Brasileiro |
| `zh-HK` | Kantonesisk             | 粵語             |
| `ms`    | Malay                 | Bahasa Melayu    |
| `sk`    | Slovak                | Slovenčina       |
| `bg`    | Bulgarian             | Български        |
| `hr`    | Kroatisk              | Hrvatski         |
| `lt`    | Litauisk            | Lietuvių         |
| `lv`    | Lettisk               | Latviešu         |
| `et`    | Estisk              | Eesti            |
| `sl`    | Slovensk             | Slovenščina      |

{% hint style=&quot;success&quot; %}
**Automatisk vedvarende**: Din sprogpræference gemmes i `~/.chloros/cli_language.json` og vedbliver på tværs af alle sessioner.
{% endhint %}

***

### `set-project-folder` - Indstil standardprojektmappe

Skift placeringen af standardprojektmappen (delt med GUI).

**Syntaks:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Eksempel:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - Vis projektmappe

Vis den aktuelle standardplacering for projektmappen.

**Syntaks:**

```bash
chloros-cli get-project-folder
```

**Eksempel:**

```powershell
chloros-cli get-project-folder
```

**Output:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - Nulstil til standard

Nulstil projektmappen til standardplaceringen.

**Syntaks:**

```bash
chloros-cli reset-project-folder
```

***

## Globale indstillinger

Disse indstillinger gælder for alle kommandoer:

| Indstilling          | Type    | Standard       | Beskrivelse                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Sti    | Automatisk detekteret | Sti til backend-eksekverbar fil                       |
| `--port`        | Heltal | 5000          | Backend API portnummer                          |
| `--restart`     | Flag    | -             | Tving genstart af backend (afslutter eksisterende processer) |
| `--version`     | Flag    | -             | Vis versionsoplysninger og afslut                |
| `--help`        | Flag    | -             | Vis hjælpeoplysninger og afslut                   |

**Eksempel med globale indstillinger:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Vejledning til behandlingsindstillinger

### Parallel behandling

Chloros+ CLI **skalerer automatisk**parallel behandling, så den passer til din computers kapacitet:**Sådan fungerer det:**

* Registrerer dine CPU-kerner og RAM
* Tildeler arbejdere: **2× CPU-kerner** (bruger hyperthreading)
* **Maksimum: 16 parallelle arbejdere** (for stabilitet)**Systemniveauer:**

| Systemtype   | CPU        | RAM      | Arbejdere  | Ydeevne     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **High-End**  | 16+ kerner  | 32+ GB   | Op til 16 | Maksimal hastighed   |
| **Mellemklasse** | 8-15 kerner | 16-31 GB | 8-16     | Fremragende hastighed |
| **Lavklasse**   | 4-7 kerner  | 8-15 GB  | 4-8      | God hastighed      |

{% hint style=&quot;success&quot; %}
**Automatisk optimering**: CLI registrerer automatisk dit systems specifikationer og konfigurerer optimal parallelbehandling. Ingen manuel konfiguration nødvendig!
{% endhint %}

### Debayer-metoder

CLI bruger **Høj kvalitet (hurtigere)** som standard og anbefalet debayer-algoritme:

| Metode                      | Kvalitet | Hastighed | Beskrivelse                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Høj kvalitet (hurtigere)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Kantbevidst algoritme (standard, anbefalet) |

### Vignettekorrektion

**Hvad gør det:** Korrigerer lysfald ved billedkanterne (mørkere hjørner, som er almindelige i kamerabilleder).

* **Aktiveret som standard** - De fleste brugere bør holde denne funktion aktiveret.
* Brug `--no-vignette` for at deaktivere.

{% hint style=&quot;success&quot; %}
**Anbefaling**: Aktiver altid vignettekorrektion for at sikre ensartet lysstyrke i hele billedet.
{% endhint %}

### Reflektanskalibrering

Konverterer rå sensorværdier til standardiserede reflektansprocenter ved hjælp af kalibreringspaneler.

* **Aktiveret som standard** – Vigtigt for vegetationsanalyse
* Kræver kalibreringsmålpaneler i billeder
* Brug `--no-reflectance` til at deaktivere

{% hint style=&quot;info&quot; %}
**Krav**: Sørg for, at kalibreringspanelerne er korrekt eksponeret og synlige i dine billeder for at sikre nøjagtig reflektanskonvertering.
{% endhint %}

### PPK-korrektioner

**Funktion:** Anvender efterbehandlede kinematiske korrektioner ved hjælp af DAQ-A-SD-logdata for at forbedre GPS-nøjagtigheden.

* **Deaktiveret som standard**
* Brug `--ppk` for at aktivere
* Kræver .daq-filer i projektmappen fra MAPIR DAQ-A-SD lyssensor.

### Outputformater

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Bitdybde</th><th width="116.5999755859375">Filstørrelse</th><th>Bedst egnet til</th></tr></thead><tbody><tr><td><strong>TIFF (16-bit)</strong> ⭐</td><td>16-bit heltal</td><td>Stor</td><td>GIS-analyse, fotogrammetri (anbefales)</td></tr><tr><td><strong>TIFF (32-bit, procent)</strong></td><td>32-bit flydende</td><td>Meget stor</td><td>Videnskabelig analyse, forskning</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>8-bit heltal</td><td>Mellem</td><td>Visuel inspektion, deling på internettet</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>8-bit heltal</td><td>Lille</td><td>Hurtig forhåndsvisning, komprimeret output</td></tr></tbody></table>***

## Automatisering og scripting

### PowerShell-batchbehandling

Behandl flere datasætmapper automatisk:

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

### Windows-batchscript

Enkel loop til batchbehandling:

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

### Python Automatiseringsscript

Avanceret automatisering med fejlhåndtering:

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

1. **Indtastning**: Mappe indeholdende RAW/JPG-billedpar
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

### Anslået behandlingstid

Typisk behandlingstid for 100 billeder (12 MP hver):

| Tilstand              | Tid      | Hardware                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Parallel tilstand** | 5-10 min  | i7/Ryzen 7, 16 GB RAM, SSD (op til 16 arbejdere) |
| **Parallel tilstand** | 10-15 min | i5/Ryzen 5, 8 GB RAM, HDD (op til 8 arbejdere)   |

{% hint style=&quot;info&quot; %}
**Tip til ydeevne**: Behandlingstiden varierer afhængigt af antallet af billeder, opløsningen og computerspecifikationerne.
{% endhint %}

***

## Fejlfinding

### CLI ikke fundet

**Fejl:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Løsninger:**

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

***

### Backend kunne ikke startes**Fejl:**

```

Backend failed to start within 30 seconds
```

**Løsninger:**

1. Kontroller, om backend allerede kører (luk den først)
2. Kontroller, at Windows Firewall ikke blokerer
3. Prøv en anden port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Tving backend til at genstarte:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Problemer med licens/godkendelse**Fejl:**

```

Chloros+ license required for CLI access
```

**Løsninger:**

1. Kontroller, at du har et aktivt Chloros+-abonnement.
2. Log ind med dine loginoplysninger:

```powershell
chloros-cli login user@example.com 'password'
```

3. Kontroller licensstatus:

```powershell
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
3. Sørg for, at du har læsetilladelse til mappen.
4. Kontroller, at filtypenavnene er korrekte.

***

### Behandlingen går i stå eller hænger**Løsninger:**

1. Kontroller den ledige diskplads (sørg for, at der er nok til output).
2. Luk andre programmer for at frigøre hukommelse.
3. Reducer antallet af billeder (behandl i batches).

***

### Porten er allerede i brug**Fejl:**

```

Port 5000 is already in use
```

**Løsning:**

Angiv en anden port:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## Ofte stillede spørgsmål

### Spørgsmål: Har jeg brug for en licens til CLI?

**Svar:**Ja! CLI kræver en betalt**Chloros+ licens**.

* ❌ Standard (gratis) plan: CLI deaktiveret
* ✅ Chloros+ (betalte) abonnementer: CLI fuldt aktiveret

Abonner på: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Spørgsmål: Kan jeg bruge CLI på en server uden GUI?**Svar:** Ja! CLI kører fuldstændig headless. Krav:

* Windows Server 2016 eller nyere
* Visual C++ Redistributable installeret
* Tilstrækkelig RAM (minimum 8 GB, 16 GB anbefales)
* Engangsaktivering af GUI-licens på enhver maskine

***

### Spørgsmål: Hvor gemmes de behandlede billeder?**Svar:**Som standard gemmes de behandlede billeder i**samme mappe som input** i undermapper til kameramodeller (f.eks. `Survey3N_RGN/`).

Brug `-o`-indstillingen til at angive en anden outputmappe:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### Spørgsmål: Kan jeg behandle flere mapper på én gang?**A:** Ikke direkte i én kommando, men du kan bruge scripting til at behandle mapper sekventielt. Se afsnittet [Automatisering og scripting](CLI.md#automation--scripting).***

### Q: Hvordan gemmer jeg CLI-output i en logfil?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### Spørgsmål: Hvad sker der, hvis jeg trykker på Ctrl+C under behandlingen?**Svar:** CLI vil:

1. Stoppe behandlingen på en ordentlig måde
2. Lukke backend
3. Afslutte med kode 130

Delvist behandlede billeder kan forblive i outputmappen.

***

### Spørgsmål: Kan jeg automatisere CLI-behandlingen?**Svar:** Absolut! CLI er designet til automatisering. Se [Automatisering og scripting](CLI.md#automation--scripting) for eksempler på PowerShell, Batch og Python.***

### Spørgsmål: Hvordan tjekker jeg CLI-versionen?**Svar:**

```powershell
chloros-cli --version
```

**Output:**

```

Chloros CLI 1.0.2
```

***

## Få hjælp

### Hjælp til kommandolinjen

Se hjælpeoplysninger direkte i CLI:

```powershell
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

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Eksempel 2: Videnskabelig output i høj kvalitet

32-bit float TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Eksempel 3: Hurtig forhåndsvisning

8-bit PNG uden kalibrering til hurtig gennemgang:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Eksempel 4: PPK-korrigeret behandling

Anvend PPK-korrektioner med reflektans:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Eksempel 5: Brugerdefineret outputplacering

Behandl til et andet drev med et specifikt format:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Eksempel 6: Godkendelsesworkflow

Gennemfør godkendelsesflow:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Eksempel 7: Brug af flere sprog

Skift grænsefladesprog:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```

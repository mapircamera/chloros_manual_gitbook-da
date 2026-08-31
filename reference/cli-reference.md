# Chloros CLI Reference

**Version:**

1.2.0**Genereret:**

29. juli 2026 kl. 19:19 ·**Revideret:**

30. august 2026**Målgruppe:** Optimeret til brug af store sprogmodeller (LLM); læsbar for mennesker.**Omfang:** Alle brugerrettede underkommandoer til `chloros-cli`, med indstillinger og eksempler, der kan kopieres og indsættes.

Dette dokument er den komplette reference til kommandolinjeværktøjet `chloros-cli`, der følger med MAPIR Chloros. Det er bevidst udtømmende, så en LLM (eller menneske) kan sammensætte enhver understøttet arbejdsgang ud fra nedenstående oversigter uden at skulle gennemgå kildekoden.

Hvis du kun har brug for det vigtigste, kan du springe til:
- [Hurtigstart på fem minutter](#five-minute-quickstart)
- [LATTICE-kamera – arbejdsgang ved første tilslutning](#lattice-camera-first-connect-workflow)
- [DAQ-sensor – arbejdsgang ved første tilslutning](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Optagelsestilstande, optagere og offline-genbehandling](#capture-modes-recorders--offline-reprocess)

---

## Konventioner

- Alle kommandoer har præfikset `chloros-cli`. På Windows er binærfilen `chloros-cli.exe`; på Linux /Jetson er den `chloros-cli`.
- Valgfrie argumenter vises som `--flag`. Obligatoriske positionsargumenter vises uden parenteser.
- Hvor der er angivet en standardværdi, anvendes denne værdi, hvis flaget udelades.
- CLI er en &#x27;thin&#x27; HTTP-klient via Chloros-backend (Flask-server på `127.0.0.1:5000`). Backendet startes automatisk af de fleste kommandoer. `CHLOROS_BACKEND_URL=<url>` dirigerer kommandofamilierne **`lattice`**,**`project`**og**`daq pool-*`** mod et fjerntliggende backend — kernekommandoerne (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) fastlåser bevidst `http://127.0.0.1:<port>` og ignorerer den (IPv4-literalet undgår Windows&#x27; `localhost`→`::1` ~2 s-per-anmodning-straf). Se [Miljøvariabler](#environment-variables).
- Der kræves en Chloros+-konto for alle SDK / CLI-kald (kør `chloros-cli login` én gang pr. maskine; gemmes i cachen i `~/.chloros/`).
- Eksemplerne bruger Linux-stier; på Windows skal `/home/user/...` erstattes med `C:/Users/.../...`.

---

## Oversigt på øverste niveau

```
chloros-cli [global options] COMMAND [command options]
```

### Globale indstillinger

| Flag | Beskrivelse |
| --- | --- |
| `--backend-exe PATH` | Tilpas den automatisk detekterede backend-eksekverbare fil. |
| `--port N` | Backend-HTTP-port (standard: `5000`). |
| `-v, --verbose` | Aktiver detaljeret output. |
| `--restart` | Tving en genstart af backend (afslutter alle kørende `backend_server.py`). |
| `--version` | Vis version (`Chloros CLI 1.2.0`). |
| `--help` | Vis hjælp på øverste niveau. |

### Kommandoindeks

| Kommando | Formål |
| --- | --- |
| [`process`](#chloros-cli-process) | Behandler en mappe med Survey3- eller LATTICE-optagelser fra start til slut. |
| [`login`](#chloros-cli-login) | Godkend denne maskine med en Chloros+-konto. |
| [`logout`](#chloros-cli-logout) | Ryd cachelagrede legitimationsoplysninger. |
| [`status`](#chloros-cli-status) | Vis aktuel licens-/godkendelsesstatus. |
| [`export-status`](#chloros-cli-export-status) | Live Thread-4 eksportforløb under en `process`-kørsel. |
| [`language`](#chloros-cli-language) | Indstil eller vis en liste over CLI-visningssprog (38 understøttede sprog). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Standardprojektmappe (delt med GUI&#x27;en). |
| [`update`](#chloros-cli-update) | Søg efter og installer opdateringer til CLI (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Systemdiagnostik + smoke-tests. |
| [`time-sync`](#chloros-cli-time-sync) | PTP-grandmaster-status / -styring. |
| [`lattice`](#chloros-cli-lattice) | LATTICE-kamerastyring og -optagelse (over 45 underkommandoer). |
| [`daq`](#chloros-cli-daq) | DAQ-spektralsensorstyring (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Åbn og kør et gemt Chloros-projekt (kameraer + DAQ&#x27;er). |

---

## Installation

`chloros-cli` leveres som en del af Chloros-installationsprogrammet til alle understøttede platforme — der findes ingen separat download af CLI. Når du installerer platformspakken, tilføjes `chloros-cli` til din `PATH` sammen med desktop-appen og den bagvedliggende binærfil, den kører.

Seneste downloads: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Installationsprogrammet indeholder også praktiske launcher-scripts (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), der åbner en brugsklar CLI-shell; disse er beskrevet i [CLI Brugervejledning](../CLI.md) og gentages ikke her.

### Windows (.exe)

1. Download installationsprogrammet Windows fra downloadsiden.
2. Kør `Chloros-Setup-x.y.z.exe`, og følg guiden. Standardinstallationsstien er `C:\Program Files\Chloros\` (CLIen placeres i `C:\Program Files\Chloros\cli\`, som installationsprogrammet føjer til PATH).
3. Åbn en ny terminal (`cmd.exe`, PowerShell eller en Windows-terminal), så den opdaterede `PATH` bliver fundet.

```powershell
chloros-cli --version
```

Installationsprogrammet tilføjer automatisk `chloros-cli.exe` til dit systems `PATH` og medfølger den Arena-SDK-runtime, der er nødvendig for LATTICE-kameraer.

### Linux amd64 (.deb)

Til Ubuntu 22.04 LTS eller nyere / Debian-baserede x86_64-arbejdsstationer.

> **Ubuntu 20.04 understøttes ikke.** Pakkens liste over afhængigheder er afledt af
> det, som backend&#x27;et rent faktisk linker til, og det inkluderer `libc6 (>= 2.34)`;
> Focal leveres med glibc 2.31. `apt` afviser installationen i stedet for at lade den mislykkes ved
> kørsel.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

.deb-filen installerer:
- `chloros-cli` til `/usr/bin/chloros-cli`
- Den kompilerede backend til `/usr/lib/chloros/chloros-backend`
- Arena-SDK-kørselstid (til LATTICE-kameraer)
- Støjfjernelsesmodeller, kalibreringspakker og konfiguration af opdateringskanal

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Samme opbygning som amd64-.deb-filen, med en CUDA-build, der er tilpasset til Jetson Orin / Orin NX / Orin Nano.

### Autentificer én gang pr. maskine

Hver platform kræver en engangsChloros+ login, før opkald til SDK / CLI fungerer:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Adgangsoplysningerne gemmes i `~/.chloros/user_session.json`.

### Bekræft installationen

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Der kræves et Chloros+-abonnement.**CLI kræver et aktivt Chloros+ abonnement.**Copper**er startChloros+ — alle betalte Chloros+ niveauer har adgang til CLI / SDK; kun det gratis**Iron**-niveau har ikke adgang. (Plan-id-oversigt: `0`=Iron/gratis, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Opgrader på [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Denne begrænsning håndhæves af backend-systemet, ikke kun af CLI: en SDK / CLI -mærket anmodning uden et betalt abonnement afvises med fejlkoden `403 PLAN_UPGRADE_REQUIRED`, uanset om den kommer fra `chloros-cli`, Python SDK eller en manueltHTTP-klient. En afmeldt bruger får i stedet fejlkoden `401 AUTH_REQUIRED`. Adgangen fungerer offline i abonnementets afdragsfri periode (30 dage pr. måned, indtil udløb for årsabonnementer) og ophører, når denne udløber; `chloros-cli status` fortsætter med at fungere, så årsagen er synlig (det er den rute SDK / CLI rute, der er undtaget fra tier-begrænsningen — `GET /api/license-status`).

---

## Fem minutters hurtigstart

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Behandl en mappe med billeder gennem hele Chloros-pipeline (målregistrering → kalibrering → vignette → reflektans → indekseksport).

### Oversigt

```
chloros-cli process INPUT [OPTIONS]
```

### Positionsargumenter

| Argument | Beskrivelse |
| --- | --- |
| `INPUT` | Sti til inputmappe, der indeholder `.raw + .jpg`-filer (Survey3), `.tif/.tiff`-filer (LATTICE) eller `.dng`-filer. |

### Almindelige indstillinger

| Flag | Standard | Beskrivelse |
| --- | --- | --- |
| `-o, --output PATH` | en ny mappe med tidsstempel under din standardprojektsti (`~/Chloros Projects`, medmindre andet er konfigureret) | Projektmappe, der skal oprettes eller genbruges. Hvis mappen allerede indeholder en `project.json`, oprettes der en `_1`/`_2`-søstermappe i stedet for at overskrive den. |
| `-n, --project-name NAME` | auto (tidsstempel) | Projektnavn. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` bruger en Chloros+ neural debayer; langsommere, men højere kvalitet. |
| `--vignette / --no-vignette` | `--vignette` | Vignetteringskorrektion. |
| `--reflectance / --no-reflectance` | `--reflectance` | Reflektanskalibrering (bruger panelmål, hvis det findes, NIST-kalibrering pr. serienummer for LATTICE). For LATTICE-multispektral fungerer dette også som omskifteren for reflektans **produkt** — se [Omskiftere for eksport pr. produkt](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | fra | Anvend PPK GNSS-korrektioner fra sidecar-filer. |
| `--exposure-pin-1 MODEL` | fra | Fastgør en &quot;pin-1&quot; på et Survey3-rig med to kameraer . |
| `--exposure-pin-2 MODEL` | fra | Fastlæg &quot;pin-2&quot;-modellen. |
| `--recal-interval SECONDS` | 0 | Tving genkørsel af kalibreringsberegninger hvert N. sekund af optagelsestiden. |
| `--timezone-offset HOURS` | lokal | Tilpas tidszoneforskydningen, der er indlejret i output-metadataene. |
| `--format FORMAT` | `TIFF (16-bit)` | En af følgende: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | ingen | Vegetationsindekser (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Tving indgangsstedet i pipelinen for LATTICE TIFF-filer (Survey3 .raw påvirkes ikke). Desuden er der en undtagelse, der gør det muligt at behandle en optagelse **uden raw** overhovedet — se [Sådan ser en captures-mappe ud](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | on | Udsend det lineært debayered produkt (`Debayered_Images`). Se [Eksportindstillinger pr. produkt](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | til | Udsender visningsforhåndsvisning (`Preview_Images`): RGB = hvidbalance (DAQ-lysforhold, når tilgængeligt, ellers gråverden) + gamma; multispec = falsk farvestrækning. |
| `--radiance / --no-radiance` | til | Udsender float32-strålingsintensitet (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Reference for LATTICE-reflektansproduktet: `auto` = QA-godkendt i-ramme-målet er den absolutte reference, DAQ-nedstråling (ρ = π·L/E) som reserve; `target` = streng (ingen DAQ-substitution); `daq` = DAQ-autoritativ. Se [Eksportindstillinger pr. produkt](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | ingen | Mappe med **målreflektansscanninger pr. enhed** (`<serial>.csv`); falder tilbage til de nominelle T3/T4P-spektre, hvis der mangler data. |
| `--array-alignment / --no-array-alignment` | on | LATTICE-arrays: Anvend den modul-til-modul-justering, der er angivet i hver optagelses `Chloros:Alignment*` XMP-fil, på alle behandlede produkter (debayered / preview / radiance / reflectance / index). Ingen handling for billeder uden disse tags. |
| `--array-alignment-crop / --no-array-alignment-crop` | beskæring | Beskær de justerede eksporter til arrayets fælles overlapningsområde, så alle moduler deler ét fodaftryk; `--no-…` bevarer det fulde sensorområde (sort udfyldning uden for kilden). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Resampling for justeringsforvrængningen. `nearest` bevarer de nøjagtige kilde-DN&#x27;er (ingen blanding af radiometriske værdier). |

### Indstillinger for måldetektion

| Flag | Beskrivelse |
| --- | --- |
| `--min-target-size PIXELS` | Mindste panel-målstørrelse (px) for detektoren. |
| `--target-clustering 0-100` | Følsomhed ved klyngedannelse. |
| `--target / --targets` | Behandl inputmappen som kun mål-kun-panel (spring undersøgelsesdetektering over). |

### Eksempler

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Eksportindstillinger pr. produkt (LATTICE multispektral)

LATTICE-behandlingen udvides til **alle relevante produkter i én gennemgang**. De fire indstillinger pr. type — `--debayered`, `--preview`, `--radiance`, `--reflectance` — er alle**slået til som standard**; brug formularen `--no-<type>` til at deaktivere en af dem. RGB master-kameraer udsender kun debayered + preview (ingen strålingsintensitet/refleksion pr. bånd), så `--radiance`/`--reflectance` har ingen effekt for dem. Toggle-indstillingerne ignoreres for Survey3 `.raw` (som følger standardreflektans-/målstien). *(Det gamle `--radiometric-output {reflectance,radiance,sensor-response}`-flag blev **fjernet** og erstattet af disse indstillingsknapper; der findes ikke længere et `sensor-response`-niveau.)*

| Produkt | Output | Kræves DAQ-downwelling? |
| --- | --- | --- |
| `--debayered` | Lineær demosaik (`Debayered_Images`). | Nej. |
| `--preview` | Vis forhåndsvisning (`Preview_Images`): RGB = WB + gamma; multispec = falskfarvestrækning. | Nej. |
| `--radiance` | float32 W/m²/sr/nm fra den fulde radiometriske kæde (`Radiance_Images`). | Nr. |
| `--reflectance` | uint16 reflektans ρ (`32768` = 1,0), Pix4D-klar. | **Ja**, medmindre et QA-godkendt mål inden for rammen forankrer den (se nedenfor). |

`--reflectance-source` vælger reflektansreferencen:**`auto`**(standard) gør et QA-godkendt mål inden for rammen til den**absolutte reference**— de målforankrede empiriske linjekæder krydsscoreres på udeladte paneler, og den målte vinder anvendes — med tilbagefald til DAQ-nedstrålingsskellet (ρ = π·L/E), når der ikke er noget mål til stede, eller QA mislykkes;**`target`**er streng (ingen DAQ-substitution);**`daq`**fravælger til fordel for den DAQ-autoritative adfærd. Målgeometri (ArUco / fast ROI /strimmel) stammer fra projektets målkonfiguration; `--target-reflectance-dir DIR` indeholder**målte** scanninger pr. enhed (`<serial>.csv`), der slås op via målenhedens serienummer/QR, med de nominelle T3/T4P-spektre som reserve.

DAQ-reflektansstien udleder automatisk den **tidsstempelmatchede nedadgående stråling**fra en registreret**`.daq`**(DAQ-U/M/E)**eller en DAQ-M-specifik `.csv`**, der findes sammen med billedmaterialet. Hvis der ikke er en lokal cache med et kalibreringssæt pr. kamera eller DAQ,**henterdet fra AWS** ved første brug (kræver internet én gang; gemmes i cachen under `~/.chloros/`).

#### Indlæsning af reflektanspixels (Pix4D / Metashape / dine egne scripts)

Reflektansen gemmes som et heltal (DN), og **den DN-værdi, der svarer til ρ = 1,0, afhænger af kildekameraet**:

| Kilde | ρ = 1,0 er | Sådan identificeres |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margin op til ρ 2,0) | XMP `Chloros:PixelScale=32768` er stemplet på filen. |
| Survey3 | `65535` (afskåret ved ρ 1,0) | Ingen `Chloros:*` XMP-tags — denne fravær *er* signalet. |

**Læs `Chloros:PixelScale` og divider med det** i stedet for at antage en konstant. Tagget er defineret i uint16-domænet, så det forbliver `32768` på tværs af outputformater, der omskalerer — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` og `TIFF (32-bit, Percent)` er alle selvbeskrivende (normaliser først den gemte datatype tilbage til uint16: ×257 fra 8-bit, ×65535 fra float).

> **Et tilfælde har ingen skala, hvilket er tilsigtet.** Når en 8-bit-kildeoptagelse (BayerRG8) skrives som 8-bit-TIFF, *klipper* pipelinen værdierne ned til 0..255 i stedet for at omskalere, så enhver værdi over ρ≈0,008 udjævnes til 255, og filen beskrives uden skalering. Chloros udelader bevidst både `Chloros:PixelScale` og `MicaSense:RadiometricCalibration`-tuplen her og logger årsagen hertil. **Hvis tagget mangler i en LATTICE-reflektansfil, skal du ikke antage, at der er en skala — eksporter på ny i 16-bit eller 32-bit** i stedet for at dividere pixels, der aldrig var delbare.

#### EXIF overføres til eksporten

`process` kopierer kildebilledets **GPS-blok og dens ExifIFD** til hvert produkt, så en
eksport indeholder `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` og
`CameraSerialNumber` sammen med georeferencen.

**`FocalLength` er ikke valgfrit for fotogrammetri.** Pix4D beregner ground sample distance ud fra
brændvidde plus højde; hvis tagget mangler, falder programmet tilbage til en helt forkert skala. På en
flyvning over en appelsinplantage med 49 optagelser forvandlede det manglende tag et område på 411 m × 160 m til et rekonstrueret
47,8 km × 13 km — et 455 MP ortofoto bestående hovedsageligt af nodata, hvilket derefter blev tolket som et fliseproblem og
et BigTIFF-problem, før nogen tjekkede GSD&#x27;en. Hvis dit ortofoto ender med en usandsynlig
skala, skal du først køre `exiftool -FocalLength` over det eksporterede produkt.

Kopien er bevidst **ikke** `-all:all`: IFD0’s strukturtags ødelægger LATTICE-outputtet, når
de kopieres, og `ExifImageWidth` / `ExifImageHeight` er udelukket, fordi de beskriver
*kilde*-optagelsen — en eksport, der nogensinde er blevet ændret i størrelse, ville ellers indeholde dimensioner,
der strider mod dens eget raster. XMP skrives direkte i stedet for at blive kopieret, fordi ExifTool
kasserer XMP-tags fra samme opkald, når XMP-blokken kopieres (hvilket ville fjerne MAPIR
kalibrerings-tags).

### Hvor outputfilerne placeres

Produkterne skrives **i projektmappen, grupperet efter kamera og derefter efter filformat**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Kameramappen er `LATT-<sensor>-<lens>-F<filter>` for LATTICE (svarende til optagelsens EXIF
`Model`) og `<model>_<filter>` for Survey3 — to kameraer, der deler sensor og filter, men adskiller sig
med hensyn til objektiv, har hver deres separate træstruktur, fordi vignettering, synsfelt og forvrængning varierer. Formatmappen
folder følger `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` eller `tiff32` for
`TIFF (32-bit, Percent)`.

> **Hvert eksporteret produkt beholder navnet på filnavn.** En radiance-eksport af
> `capture_…_raw.tif` hedder stadig `capture_…_raw.tif` — den ligger blot i
> `tiff32/Radiance_Images/`. **Mappen identificerer produktet, ikke filnavnet**, så en glob-søgning
> efter `*radiance*.tif` finder intet; søg i stedet i mappen.

### Optagelser fra lyssensoren — kalibreret `.daq` + `.csv`

`process` håndterer også `.daq`-optagelserne i din inputmappe, og den **ikke**
brug for billeder for at gøre det: en DAQ-U / DAQ-M / DAQ-E, der flyves alene, er en komplet
optagelse, og en mappe, der kun indeholder `.daq`-filer, er en gyldig indgang.

En DAQ kan optages **uden** kalibrering — det er netop det, den offentlige
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) optagere
(`record_daq.py`) gør som standard: de skriver rå sensortællinger og mærker filen, så
Chloros henter den pågældende sensors fabrikskalibrering **via seriel port** (først fra den lokale cache,
derefter fra MAPIR Cloud) og anvender den. `process` skriver resultatet tilbage:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` indeholder én række pr. måling: UTC-tidsstempel, integrationstid, total effekt,
fotopisk/skotopisk lux, PPFD (og dens opdeling i blå/grøn/rød), spidsbølgelængde og derefter det
fulde spektrum på sensorens eget bølgelængdegitter. `.daq` importeres igen uden at blive
kalibreret en anden gang.

Ved succes rapporterer kørslen `Light-sensor products written: N (calibrated .daq + .csv)`.
Det, der står i parentes, beskriver, hvad der faktisk blev skrevet, så det lyder
`(RAW COUNTS — this sensor has no calibration bundle)` for en sensor uden bundt og
`(N calibrated, M raw counts)` for en mappe, der indeholder begge dele. Backendens egne
overskrifter `[DAQ-EXPORT]` og `[RUN-SUMMARY]` er formuleret på samme måde — ingen af
de tre kan betegne en rå eksport som kalibreret.

En DAQ-U / DAQ-M / DAQ-E-optagelse, hvis kalibreringspakke ikke kan hentes — du er
offline, eller den pågældende sensor har ingen kalibrering gemt — **springes over med en begrundelse** på en
`[DAQ-EXPORT]`-linje og skrives aldrig ud som en »kalibreret« fil, der indeholder rå tællinger.
Opret forbindelse til internettet, og kør programmet igen. Årsagen er den, som læseren faktisk
har fastslået for den pågældende fil (ulæseligt skema, intet bundt, en skrivefejl), og kørselsoversigten
angiver **separate** årsager — tyve filer, der er sprunget over på grund af én årsag, vises som én
årsag, ikke tyve gentagelser af den.

#### DAQ-A-optagelser eksporteres som rå tællinger

**DAQ-A**-familien er ældre end systemet med bundter pr. serienummer og har intet kalibreringsbundt
at hente — den kalibreres i stedet i felten mod et reflektansmål, hvilket er
grunden til, at den aldrig har haft brug for en. Da afvisningen af disse optagelser gjorde, at de slet ikke kunne få deres
tal ud, eksporteres de derfor under et **andet navn**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Et andet filnavn i stedet for en markør inde i filen, fordi oplysningerne skal kunne overleve
at blive videresendt via e-mail som et rent navn. Overskriften `.csv` angiver
`raw spectral sensor counts (NOT irradiance)` og advarer om, at værdierne er sammenlignelige
**inden for** filen — hvilket er præcis det, målbaseret kalibrering bruger dem til — og
ikke på tværs af sensorer. De effekt-afhængige fotometriske kolonner (total effekt, fotopisk og
skotopisk lux, PPFD) skrives som **NULL** i stedet for at blive integreret ud fra tællinger, og kørselsoversigten
angiver `RAW COUNTS`, så &quot;eksporteret&quot; i en log kan ikke fortolkes som bestrålingsstyrke.

Ældre **v1.01 / v1.02**-optagelser (en DAQ-A-SD skriver disse) indeholder ingen epoke pr. aflæsning,
kun filens skrivetid. Billed↔nedstrålings-matcheren afviser dem stadig — at matche et
billede mod et skrivetidspunkt ville være forkert uden at man kan se det — men eksportøren læser dem, og
CSVen udskriver `clock=daq_created_on`, så produktet angiver, hvilket ur det er på.

### Bemærkninger

- `process` registrerer automatisk, om din mappe er af typen »Survey3«, »LATTICE« eller en blanding.
- Fremskridtet streames via Server-Sent Events; CLI viser live fremskridt pr. tråd (Detektering, Analyse, Behandling, Eksport).
- For Linux /Jetson kontrollerer CLI swap-pladsen og kan vise en advarsel, før store mapper behandles. Den teksturbevidste debayer anvender også automatisk en GPU-frekvensbegrænsning på Jetson-enheder med lavt strømforbrug (Nano, Orin Nano).
- Ved succes rapporterer kørslen, hvor mange billedprodukter den har skrevet (`Image products written: N`).

#### En kørsel, der ikke skriver nogen billeder, mislykkes

Hvis du anmodede om produkter, og kørslen skrev **ingen** — kun `project.json` og
`calibration_data.json` — betragter `process` dette som en fejl: den udskriver
`Processing finished but wrote no image products.` og **afsluttes med en værdi forskellig fra nul**, så et script kan
registrere det. Meddelelsen angiver projektmappen og de sædvanlige årsager:

- indgangsmappen blev ikke genkendt som en optagelse (tjek layoutet og `--input-level`), eller
- alle anmodede produkter blev sprunget over, da de ikke var relevante for disse kameraer (f.eks. anmodning om
  radiance/reflektans fra kameraer, der kun bruger RGB).

Kør programmet igen med `--verbose` og tjek backend-loggen for linjer med `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
som forklarer de kamera-specifikke spring, der ellers ikke vises i CLI&#x27;s output.

En bevidst kørsel, der udelukkende omfatter metadata — alle produkter slået fra og ingen `--indices` — er stadig en
**succes**, fordi en tom billedudgang er det korrekte resultat i dette tilfælde.

Det samme gælder en **kørsel udelukkende med lyssensor**: en mappe med `.daq`-optagelser indeholder per definition ingen billeder, der kan eksporteres,
og kørslen bedømmes ud fra de kalibrerede `.daq` / `.csv`, som den i stedet skrev.

---

## `chloros-cli login`

Godkend denne maskine med en Chloros+ cloud-konto. Adgangsoplysningerne gemmes sikkert i cachen i `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Eksempler

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (ved at fjerne dele af adgangskoden eller duplikere dele af den). Ved en 401-fejl forsøger CLI automatisk igen med `$$` tilføjet, derefter med en halveret og dedupliceret del af adgangskoden; hvis et nyt forsøg lykkes, logger den dig ind og viser den korrekte syntaks med enkeltanførselstegn, som skal bruges næste gang.

> **Brug uden grafisk brugergrænseflade/via script: ingen cachelagret session betyder en interaktiv prompt, ikke en hurtig fejlmelding.** Enhver kommando, der starter en backend (`process`, `status`, `export-status`, `time-sync`, …) uden en cachelagret licens/session, ender i en interaktiv `Email:` / `Password:`-prompt på stdin, før den fortsætter. Et uovervåget job uden en cachelagret session vil derfor hænge og vente på indtastning — kør `chloros-cli login EMAIL PASSWORD` én gang pr. maskine, før du planlægger headless-arbejde.

---

## `chloros-cli logout`

Rydder den cachelagrede session og tvinger et nyt login ved det næste opkald.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Viser det aktuelle licensniveau (Iron/Copper/Bronze/Silver/Gold), den godkendte bruger og antallet af enhedsbindinger.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Henter status for den aktuelle Thread-4-eksport. Kan sikkert kaldes **under** en `process`-kørsel fra en anden shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Indstil visningssproget for CLI (38 sprog understøttes, herunder CJK, RTL og indiske sprog). Skifter problemfrit til engelsk på ældre konsoller, der ikke kan gengive skriften.

```
chloros-cli language [LANG_CODE] [--list]
```

### Eksempler

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Kommandoer til projektmappe

Disse styrer standardplaceringen af projektmappen (delt med GUI&#x27;en).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Kun Jetson. Kontrollerer `version_url` fra `/etc/chloros/update.conf` og tilbyder at downloade og installere den matchende `.deb`.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

På Linux /Jetson udfører CLI også en **automatisk opdateringskontrol ved hver opstart** (ikke-blokerende, forsinker aldrig kommandoen): den læser `/etc/chloros/update.conf`, gemmer resultatet i cachen i 1 time i `~/.chloros/update_cache.json`, og udskriver `Update available: vX.Y.Z / Run: chloros-cli update`, når der findes en nyere version. Springes lydløst over ved eventuelle fejl og ved Windows.

---

## `chloros-cli selftest`

Kører en 7-trins smoke-test: version, porttilgængelighed, opstart af backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), tilstedeværelse af støjfjernelsesmodel, CUDA+støjfjernelsesberedskab.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP-grandmaster-status og -styring. Værten på Chloros kører PTP-; LATTICE-kameraer og DAQ-E-enheder fungerer som slaver til denne for at sikre tidsstempler på tværs af enheder.

| Underkommando | Beskrivelse |
| --- | --- |
| `status` | Vis grandmaster-tilstand, BMCA-prioriteter, uridentitet. |
| `peers` | Vis liste over slaver, der ses via Delay_Req (kameraer + DAQ-E-sensorer). |
| `cameras` | PTP-tilstand pr. kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Genstart grandmaster-processen. |
| `set-priority --priority1 N --priority2 N` | Tilsidesæt BMCA-prioriteter. |

### Eksempler

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE-kamerastyring. Hver underkommando dirigeres gennem backend&#x27;en Chloros; backend&#x27;en ejer kamerapoolen, så efterfølgende CLI-kald genbruger det samme åbne håndtag.

### Almindelige indstillinger (fælles for de fleste underkommandoer)

| Flag | Beskrivelse |
| --- | --- |
| `-d, --device N` | Kameraindeks (standard: 0). |
| `-s, --serial SN` | Specifik serienummer; tilsidesætter `--device`. |
| `--serials SN1,SN2,…` | Komma-adskilte serienumre til drift med flere kameraer. |
| `--all` | Udfør på alle fundne kameraer. |
| `--exposure US` | Eksponeringstidtid i mikrosekunder. |
| `--gain DB` | Forstærkning i dB. |
| `--pixel-format FMT` | f.eks. `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Billeddimensioner. |
| `--preset {default,high_quality,high_speed,triggered}` | Anvend en forudindstilling. Alle kører frit, undtagen `triggered`, som aktiverer kameraet for en hardware-kant på linje 2 — hvis der ikke er noget, der driver den linje, vil det vente i al evighed i stedet for at optage. |
| `-o, --output DIR` | Output-mappe (standard: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP-pakkestørrelse. `auto` kører ICMP+GVSP-prober; `jumbo` = 9000; `standard` = 1500. |

### LATTICE-kamera: Arbejdsgang ved første tilslutning

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Reference til underkommandoer

#### Opdagelse og information

| Underkommando | Formål |
| --- | --- |
| `lattice info` | Vis liste over tilsluttede kameraer (producent, model, serienummer, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analyser værtsystemet for optimal kamerakonfiguration. `--no-discover` springer kamerasøgning over (hurtigere, analyse kun af netværkskort). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Kontroller/rettelse af netværkskortindstillinger; estimering af båndbredde/FPS. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Netværkskapacitet i backend med stabilt skema + anbefaling af array (returnerer `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` bevarer den anmodede opløsning, men sætter et loft over mål-fps — læs `recommended.recommended_target_fps` og send det som forbindelsesmål; behandl det som en succes, ikke som en fejl. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Hvad-hvis-analyse uden at åbne kameraer. **`--n-active` er det samlede antal kameraer på netværket, ikke kun denne arrays**— hæv det, når enkeltstående kameraer streamer samtidigt, eller når netværkskapaciteten beregnes ud fra et behov, der undervurderer antallet af kameraer (standard: `len(--models)`). Viser altid de samlede `Wire budget:`-værdier (krævet MB/s vs. kollisionssikkert loft) og `Max cameras:`-linjerne, og markerer `** OVER-SUBSCRIBED**`, når arrayet overbelaster kablet — se [Array fps &amp; burst-model](#array-fps--burst-model). |
| `lattice gpu` | Vis GPU-status. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Kontroller eller opdater kameraets firmware. Det lokale `.fwa`-valg er fastlåst: filen i `firmware/<MODEL_PREFIX>/`, der matcher buildets `MIN_FIRMWARE_VERSION`, flashes, når den er til stede (kun den højeste version som fallback), så et nyere leverandørbillede, der er klargjort på disken, er inaktivt, indtil den pågældende fastlåste indstilling ændres — bevidst når nyere udgivelser enhederne via det signerede AWS-manifest, hvilket foretrækkes, når det er nyere. |
| `lattice presets [--apply NAME]` | Vis eller anvend kameraindstillinger. |
| `lattice status` | Live kamerastatus. |

#### Optagelse

| Underkommando | Formål |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Enkeltbillede. **Gemmer som standard alle eksporttyper** (`--processing all`); se [Optagelseseksportniveauer](#capture-export-levels-the-all-default). `--levels` gemmer en eksplicit delmængde (tilsidesætter `--processing`); `--force-daq` skriver den tildelte DAQ-aflæsning som en `.daq` sidecar selv ved en udelukkende råoptagelse. `--jpeg-quality` = JPEG kvalitet 1–100 (standard 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Stream til disk indtil Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Browserbaseret live MJPEG-forhåndsvisning. `--ae-damping` indstiller dæmpning af automatisk eksponering (0,4–100). |

#### Sensorjustering

| Underkommando | Formål |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Læs/skriv til enhver GenICam-node. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Eksponering og AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Forstærkning og automatisk forstærkning. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Sensor-ROI og binning. |
| `lattice format [--set FMT] [--list]` | Pixelformat. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Hardware-/softwaretrigger. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (ingen flag = one-shot hvidbalance) | Hvidbalanceoperationer. RGB /Kun Bayer-kameraer; en no-op (springs over) på mono M3M. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB-farvebehandlingsrørledning. `natural` (standard) er den billige live-efterbehandling; `enhanced` tilføjer defringe + vibrance + CLAHE lokal kontrast for det fulde hub-parity-look til ca. 2× pr.-frame-finishomkostningerne, hvilket medfører en lavere **live**-billedhastighed — gemte optagelser får altid den fulde finish uanset hvad. RGB /Kun Bayer-kameraer; springes over på mono M3M. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Vis mætning/kontrast (kameraer med RGB-filter). Springes over på mono M3M. |
| `lattice filter [--set NAME] [--list]` | Indstil kameraets filtermodel (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Mål strøm-/termiske knudepunkter; skift til lavt strømforbrug i tomgang. |

#### Kalibrering og sensorer

| Underkommando | Formål |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibrer ud fra et reflektansmål. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Indbyggede kommandoer til nedadrettet lyssensor. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Anvend vignetteringskorrektion på eksisterende billeder. |

#### Multikamera (transiente sessioner)

| Underkommando | Formål |
| --- | --- |
| `lattice multi-info` | Vis en liste over alle kameraer med synkroniseringsroller. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Ét synkroniseret billede fra hvert kamera. Gemmer **alle eksporttyper som standard**, når der er tilsluttet et permanent array; den midlertidige fallback uden array er**kun debayered** (kør først `array-connect` for resten). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Stream synkroniserede billeder (midlertidigt). |
| `lattice multi-test [--count N]` | GPIO-synkroniseringstest test. |
| `lattice multi-detect [--line LINE] [--json]` | Automatisk detektering af GPIO-master/slave-ledningsføring. |

#### Justering

| Underkommando | Formål |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — plus detektor-/matcher-knapper `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC-knapper `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombination af flere-rammekombination `[--averaging mean\|median\|inlier_weighted]`, geometriske begrænsninger `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, rumlige begrænsninger `[--roi X0,Y0,X1,Y1] [--mask PATH]`, og overskrivninger pr. slave `[--per-cam-override SN:KEY=VALUE]` (gentagelig) | Beregn justeringsprofil fra live-kameraer. `--prefilter` er som standard indstillet til `gradient` (kantkort; matcher GUI/array — kanter bevares på tværs af spektralbånd). `--matcher flann` giver det bedste resultat ved over ~5000 træk; `--averaging median` er robust over for én dårlig optagelse, `inlier_weighted` vægter efter antal match; `--lock-scale` projicerer til den nærmeste rotation (ingen skalering), `--lock-axis` sætter en translationskomponent til nul; `--mask` gælder for alle kameraer (brug `--per-cam-override` til indstillinger pr. kamera, f.eks. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` nægter at gemme en kalibrering, hvis RMS-værdi for reprojicering overskrider grænsen. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Optag et justeret multibåndsbillede. `--bit-depth` indstiller som standard til at matche kameraet; `--no-crop` bevarer hele billedet (udfyldes med sort); `--interpolation` (standard `linear`) og `--border-mode`/`--border-value` (standard `constant`/0) styrer CPU-warp — GPU-stien er uanset hvad bilineær. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Stream-justerede multibåndsrammer (samme warp-knapper som `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Vis profiloplysninger. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Skift lagorden. |

#### Indeks / Vegetationsberegninger

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Fuldt flagsæt: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (kan gentages), `--capture-level raw|debayered|radiance|reflectance|unknown` (tilsidesætter det optagelsesniveau, der er registreret i kildeTIFF; standard: læses fra TIFF-metadata), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Med `--live` gælder også justeringsknapperne: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **`--channel`-symboler er store- og småbogstavsfølsomme.** Symbolsiden skal stemme nøjagtigt overens med forudindstillingens kanalnavne (forudindstillinger bruger små bogstaver, f.eks. NDVI = `red`,`nir` — tjek `--list-presets`), og båndsiden skal stemme overens med et båndnavn i den tilpassede stak (eller være et 0-baseret båndindeks i offline-tilstand). `--channel red=Red_660 --channel nir=NIR_850` fungerer; `--channel RED=660` mislykkes med en `channel_map missing entries`-fejl.

#### Vedvarende forbindelser (Smart-Prep, GUI-ækvivalent flow)

Disse kommandoer holder kameraer åbne i backend-puljen på tværs af CLI-kald.

| Underkommando | Formål |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Tilføj et kamera til puljen (enkeltkamera, intet array). |
| `lattice cam-disconnect [--serial SN] [--all]` | Frigiv. |
| `lattice cam-list` | Vis kameraer i puljen. |
| **`lattice array-connect`**|**Tilslut et permanent synkroniseret array (DET anbefalede indgangspunkt).** Kører det fulde GUI-smart-prep-forløb. |
| `lattice array-disconnect [--array-id ID] [--all]` | Frigiv et array. |
| `lattice array-list` | Vis en liste over tilsluttede arrays. |
| `lattice array-status [--array-id ID]` | Live fps, PTP, sidste fejl. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Én synkroniseret optagelse fra live-arrayet — Enkelt / Kontinuerlig / Interval / Hurtigst. **Standardindstillingen er `all`** (én fil pr. relevant eksporttype pr. kamera). Oversprungne kameraer (f.eks. RGB udelukket fra strålingsintensitet/refleksion) rapporteres med `Skipped: SN:<serial> (<reason>)`; den DAQ-måling, der anvendes til reflektans, gemmes sideløbende og rapporteres med `DAQ: <path>`. Se [Optagelsestilstande, optagere og offline-genbehandling](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Optag den kombinerede indeksvisning i realtid som video/GIF (overvågningskvalitet; den kombinerede strøm skal være åben). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Rå Bayer-seriebilleder med høj billedfrekvens (til analyse; skal genbehandles offline). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Genbehandl en gemt rå seriebilledserie til kalibrerede videoer. |

##### `array-connect`-indstillinger

| Flag | Standard | Beskrivelse |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatisk opdagelse af alle LATTICE-kameraer (kræver ≥2) | Det første serienummer er MASTER. Hvis dette udelades, filtreres opdagelsen til LATTICE (`TRI032*`)-modeller, og alle disse forbindes. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synkroniseringslinje. |
| `--target-fps F` | auto | Master-triggerens affyringshastighed. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Tilsidesæt niveauvælgeren. |
| `--wire-ceiling-mbps MB_PER_S` | automatisk detekteret | **Værtens vedvarende båndbredde i MB/s — det tal, som hele array-allokeringen afhænger af.** Sænk det, når arrayet rapporterer GVSP-korrupte rammer: den automatiske værdi er afledt af NIC’ens angivne linkhastighed, som overvurderer USB-adaptere, smalle PCIe-baner og travle delte netværksstrukturer. Gemmes i projektets array-capture-blok, så en genåbning / CLI / SDK genopretter den ved genforbindelse. Se [Array-tilstand](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Hardware-binning. |
| `--no-recommend` | off | Spring netværksanalysetrinnet over. |
| `--no-ptp` | off | Deaktiver PTP (tidsstempler på tværs af kameraer er da **ikke** sammenlignelige). |

### Smart-AE / Smart-Capture

LATTICE-arrays kører kontinuerlig AE i baggrunden, så snart de er tilsluttet, men det tager et øjeblik for en nyindstillet scene at konvergere. `array-capture --smart` er den **praktiske løsning**: den venter på, at AE stabiliserer sig på alle kameraer i arrayet, og udløser derefter optagelsen. Brug den, når du skifter scene midt i en session.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Stabiliseringspolitikken er som standard konservativ: 5 s timeout, 1.5 s stabilitetsvindue, ±5 % tolerance for eksponeringsspredning. Juster via SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`), hvis du har brug for en anden adfærd fra automatiseringen.

### Eksportniveauer for optagelse (standardindstillingen `all`)

Fra og med denne udgivelse er `lattice capture`, `lattice multi-capture` og `lattice array-capture` **indstillet til `--processing all`** — én gemt fil pr. eksporttype, der gælder for hvert kamera, svarende til GUI’ens »Capture All«-adfærd. Niveauerne er:

| Niveau | Output | Gælder for |
| --- | --- | --- |
| `raw` | Enkeltkanals Bayer (monokameraer: det enkelte bånd) direkte fra sensoren. | Alle kameraer. |
| `debayered` | 3-kanals BGR-demosaik (monokameraer: 1-kanals gråskala). | Alle kameraer. |
| `radiance` | float32 W/m²/sr/nm via den fulde radiometriske kæde. | Kun multispektrale (M3C/M3M) — **overspringes for kameraer med RGB-filter**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Pix4D-klar. | Kun multispektral, og **kun når en DAQ er tilknyttet + kameraet er kalibreret**; ellers springes over. |
| `preview` / `display` | Fuld GUI-forhåndsvisningskæde (CCM + hvidbalance + gamma i henhold til kameraets profil). `lattice capture` navngiver dette `preview`; `array-capture`/`multi-capture` bruger `display`. | Alle kameraer. |

Angiv et enkelt niveau for kun at gemme netop det (`--processing debayered`). Når du anmoder om `all`, springes niveauer, der ikke gælder for et givet kam, over (og rapporteres), men der opstår ingen fejl — et ikke-tilsluttet eller ikke-kalibreret kamera modtager stadig `raw` / `debayered` / `preview`.

For enhver reflektansramme skrives den faktisk anvendte DAQ-nedstrålingsmåling til en **`.daq`**-sidecar ved siden af billedet (så optagelsen kan genbehandles senere) og rapporteres på en `DAQ:`-linje.

### Sådan ser en optagelsesmappe ud

Hver eksporttype placeres i sin **egen undermappe** under `-o`, så en optagelse på flere niveauer aldrig blander typerne:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` er optagelsens tidsstempel, og `<serial>` er kameraets serienummer, så en synkroniseret gruppe deler et
fælles tidsstempel på tværs af kameraerne. **Bemærk den ene asymmetriske:** `display`-niveauet gemmes i en mappe
med navnet `preview/`, mens selve filerne beholder `_display` i navnet — mappe og suffiks adskiller sig
kun for dette niveau. Ukendte niveauer gemmes i en mappe med deres eget navn, og hvis undermappen
ikke kan oprettes, skrives filen til output-roden i stedet for at gå tabt.

**Genbehandling af en captures-mappe:**peg `chloros-cli process` mod**captures-rodmappen**
(`output/`). `process` importerer normalt kun den mappe, du angiver, men når den pågældende mappe ikke indeholder
billeder og har undermapper, går den automatisk ned i hierarkiet — så rodmappens undermapper og
roden `.daq` hentes alle på én gang. Hvert niveau i en indlæsning importeres som et enkelt billede, hvor
de øvrige niveauer er tilgængelige som tilstande, i stedet for ét billede pr. niveau.

At navngive en **niveau undermappe** direkte (f.eks. `output/raw/`) fungerer også. Hvis du gør dette, bliver rodmappen
`.daq` udeladt, så kopier eller henvis DAQ-aflæsningen til den, når du genberegner et radiometrisk
produkt fra `raw/` — ellers har tidsstemplet intet at matche mod.

**Behandlingen starter altid fra `raw`.** Inden for hver optagelse er den rå ramme kilden til pipelinen;
`debayered`, `radiance`, `reflectance` og `preview` følger med som visningsformater, men føres aldrig
tilbage gennem pipelinen.-behandling af et afledt produkt vil genanvende vignettering, CCM og
radiance-beregninger, der allerede er indbagt i dets pixels, så Chloros fravælger dette frem for
dobbeltbehandling. To konsekvenser, der er værd at kende:

- Renderingerne `index/` og `composite/` bliver **aldrig** behandlet. De er output, ikke optagelser —
  en NDVI LUT-rendering har ingen meningsfuld radiance-fortolkning.
- En captures-mappe, der eksporteres **uden** `raw` (f.eks. `array-capture --processing reflectance`), har
  ingen gyldig pipeline-kilde. Disse optagelser importeres og vises normalt, men `process` springer
  dem over og angiver dette:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Hvis du virkelig har brug for at sende et afledt produkt igennem — en hub-session, der er indfanget med
  `demosaic` aktiveret, eller en ældre mappe — tvinger `--input-level {raw,debayered,processed}` indgangen
  og tilsidesætter springet. Dette flag er den bevidste nødudgang; `auto` (standardindstillingen)
  behandler aldrig en optagelse, der ikke har rådata.

### Springede optagelser i blandede filteropstillinger

Når du blander RGB og multispektrale kameraer i én opstilling, gemmer `array-capture --processing radiance` (eller `reflectance`) de multispec-rammerne og **springer** de RGB-kameraer over — strålingsintensiteten pr. Bayer-pixel er ikke meningsfuld for en bredbåndssensor. CLI udskriver hver gemt fil (med eksportniveau), hver `.daq`, der skrives, og hvert spring eksplicit, så antallet af filer ikke kommer som en overraskelse:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Token for springårsagen følger mønsteret `<level>-not-applicable-to-rgb-cam`. Reflektans kan også springes over med `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` og med `dls-uncalibrated-band-<nm>`, når båndet hovedsageligt ligger uden for DAQ-lyssensorensradiometrisk kalibrerede område (~374–974 nm) — blandt de leverbare SKU’er er det kun F988, hvis understøttede forløb er arbejdsgangen med reflektanspanelet.

Brug `--processing debayered` (eller `display`) for at inkludere alle kameraer uanset filtertype, eller standardindstillingen `all` for at få alle relevante niveauer pr. kamera på én gang.

---

## Optagelsestilstande, optagere og offline-genbehandling

Disse fungerer alle på et **persistent array** (kør først `array-connect`). De afspejler GUI-optagelsespanelet.

### `array-capture`-tilstande

`array-capture` er en enkelt kommando med fire lukkerindstillinger samt et sæt eksportindstillinger:

| Indstilling | Flag | Adfærd |
| --- | --- | --- |
| **Enkelt** *(standard)* | (ingen) | Én synkroniseret optagelsesgruppe, derefter afsluttes. |
| **Kontinuerlig** | `--continuous` | På hinanden følgende gennemløb indtil `Ctrl+C`, `--count N` eller `--duration S`. |
| **Interval** | `--interval S` | Ét gennemløb hvert `S` sekunder (målt fra starten af hvert gennemløb), samme grænser. |
| **Hurtigst** | `--fastest` | Kun rådata + den tildelte DAQ-aflæsning + den sammensatte indekskombination; springer strålings-/refleksions-/visningsberegningerne over, så billedet vises hurtigt. Indebærer `--processing raw --force-daq`. Genbehandl de gemte `.daq` til kalibrerede produkter senere. |

Eksportindstillinger (kan kombineres med enhver tilstand; alle deler GUI/SDK-endpoint):

| Flag | Effekt |
| --- | --- |
| `--processing LEVEL` | Enkelt eksportniveau eller `all` (standard). |
| `--levels L1,L2,…` | Eksplicit delmængde af eksporttyper (f.eks. `raw,radiance,reflectance`); **tilside sætter `--processing`**. |
| `--aligned` / `--no-aligned` | Warp hvert medlems ikke-rå eksport til arrayets [justeringsprofil](#alignment) (samregistreret). Rådata forbliver ujusteret, men bærer transformationen i metadata. Faldt tilbage til ujusteret (med en advarsel), hvis arrayet ikke har nogen profil. |
| `--index` / `--no-index` | Gem / spring over overlejringen af vegetationsindeks pr. kamera, hvor en sådan er konfigureret. Standard: gengiv den. |
| `--force-daq` | Gem den tildelte DAQ/DLS-måling som en `.daq`-sidecar, selv når intet valgt niveau har brug for den (f.eks. en udtrækning, der kun indeholder rådata), så billederne kan genbehandles til reflektans/indeks offline. |
| `--smart` | Vent på, at AE stabiliserer sig på alle kameraer, før der udløses (se [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF-pixelkomprimering. `deflate` (standard) = tabsfri zlib L1 + vandret prediktor, ~4,1 MB pr. ramme i fuld opløsning; `none` = ukomprimeret, ~5× hurtigere skrivning ved ~6,3 MB pr. billede — bruges til maksimal vedvarende hastighed, når disken tillader det. Begge er tabsfri og læses identisk ved import. |

> **Enkelt-skrive-TIFFen + modellen med vedvarende hastighed.**Optagelser skrives i**ét**TIFF-fil-gennemløb, der indeholder pixels + XMP + IFD0 Mærke/Model (målt på Mono12 i fuld opløsning: 36 ms komprimeret / 6,5 ms ukomprimeret, mod ~148 ms for den gamle metode med først skrivning og derefter omskrivning med ExifTool); det eneste tilbageværende ExifTool-arbejde (finpudsning af EXIF-sub-IFD) kører i en asynkron baggrundsproces, og et billede er færdigt og klar til import, selvom denne proces aldrig kører. Bemærk, at DEFLATE-komprimering holder GIL&#x27;en Python, så komprimerede skrivninger**ikke**udføres parallelt på tværs af skrivetrådene pr. kamera — vedvarende optagelse i fuld opløsning med 8 kameraer ved sensorhastighed (~10,4 fps) kræver `--compression none`**og** en disk i NVMe-klassen (~500 MB/s vedvarende skrivning). Den samme indstilling er tilgængelig som `compression` på `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — video/GIF med kombineret indeks (overvågningskvalitet)

Optager alt, hvad **live-visningen med kombineret indeks** viser til en `.avi` (og eventuelt en `.gif`). Da den tapper den live sammensatte strøm, skal den kombinerede strøm være åben (f.eks. vises arrayet i forhåndsvisning i GUI&#x27;en), for at billederne kan indlæses. Den tjekker fremskridtet hvert 2. sekund og stopper på `--duration`, `Ctrl+C` eller når optageren selvslutter.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Flag | Standard | Beskrivelse |
| --- | --- | --- |
| `--array-id ID` | kun array | Målarray (udelad, hvis der kun er tilsluttet ét). |
| `-o, --output DIR` | `output` | Output-mappe (backend-lokal). |
| `--fps F` | `10` | Optagelsesbilledhastighed. |
| `--duration S` | indtil Ctrl+C | Automatisk-stop efter `S` sekunder. |
| `--gif` | fra | Skriv også en animeret GIF. |
| `--gif-only` | fra | Skriv kun en GIF (ingen `.avi`). |

### `array-burst` — rå-Bayer-serieoptagelse med høj billedfrekvens (analysekvalitet)

Læser direkte fra grab-loopens synkroniserede gruppebuffer — **ingen kalibreringskæde, intet exiftool, ingen live-visning nødvendig** — så den kører med kameraets fulde optagelseshastighed. Skriver rå billeder + et manifest pr. billede + én `.daq` pr. unik DLS-aflæsning under `<output>/bursts/<base>/`. Genbehandl offline (næste kommando), eller send `--build` for at gøre det straks ved stop.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Flag | Standard | Beskrivelse |
| --- | --- | --- |
| `--array-id ID` | kun array | Målarray. |
| `-o, --output DIR` | `output` | Output (burst lander i `<DIR>/bursts/<base>/`). |
| `--duration S` | indtil Ctrl+C | Automatisk stop efter `S` sekunder. |
| `--max-frames N` | ubegrænset | Automatisk stop efter `N` råbilleder. |
| `--build` | slået fra | Efter stop skal burst-serien straks genbehandles (samme som `array-build-video`). |
| `--products …` | `combined:index` | Med `--build`: hvilke videoer der skal oprettes (se nedenfor). |
| `--fps F` | `10` | Med `--build`: fps for outputvideo. |
| `--save-tiffs` | fra | Med `--build`: gemmer også kalibrerede TIFF-filer pr. billede. |
| `--gif` | fra | Med `--build`: skriver også animerede GIF&#x27;er. |

### `array-build-video` — offline-genbehandling af en gemt serie

Tidsmatcher hvert råbillede med den nærmeste gemte `.daq`-måling og sender det gennem den **samme strålings-/ refleksionsgrad / indekskæde som importpipeline**, hvilket resulterer i en eller flere videoer.

`--products` er en kommasepareret liste over `kind:level`-elementer, hvor `kind` ∈ `per_cam` | `combined` og `level` ∈ `radiance` | `reflectance` | `index`. En `level` uden yderligere angivelser (ingen `kind:`) indstilles som standard til `per_cam`. Standardindstillingen er `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Flag | Standard | Beskrivelse |
| --- | --- | --- |
| `--burst-dir DIR` | (påkrævet) | Sti til burst-mappen (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level`-listen, som ovenfor. |
| `--fps F` | `10` | Antal billeder pr. sekund (fps) for den udgående video. |
| `--save-tiffs` | slået fra | Gem også kalibrerede TIFF-filer pr. billede sammen med videoen/videoerne. |
| `--gif` | slået fra | Skriv også animerede GIF(er). |

> **Vælg den rigtige optager.** `array-record` er *overvågningskvalitet* — den optager det levende sammensatte billede, som det vises, og kræver, at strømmen er åben. `array-burst` → `array-build-video` er *analysekvalitet* — den gemmer rå sensordata ved fuld hastighed og rekonstruerer efterfølgende kalibrerede strålings-/refleksions-/indeksvideoer, uden at der kræves livevisning.

### Mono (M3M) Enkeltbåndskameraer

**M3M**-serien er den monokrome søster til Bayer**M3C**: ét smalbåndsinterferensfilter pr. kamera (`M3M-<lens>-F<wavelength>`, f.eks. `M3M-L87-F685`), så sensoren leverer et**enkelt gråtonebånd** uden Bayer-mosaik. Der er intet at demosaicere, ingen krydstale mellem kanalerne at adskille og ingen hvidbalance at indstille — hele farvebehandlingskæden fra RGB-displayet finder simpelthen ikke anvendelse.

Hvad det betyder på CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**registrerer et monokromt kamera og**springer over med en en-linjers besked** i stedet for at sende meningsløse indstillinger. De kører stadig normalt med et RGB /Bayer M3C-kamera i samme session.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** fungerer stadig — radiance og reflektans er *båndspecifikke* radiometriske kort og er perfekt veldefinerede for ét bånd. Monobilleder har en **identitets**-sensorresponsmatrix (ingen 3×3-unmix), så planet passerer uændret gennem kalibreringsberegningerne.
- **Et enkelt monokamera kan ikke generere et vegetationsindeks.**NDVI / NDRE /osv. kræver mindst to bånd (f.eks. Red + NIR). For at opnå et indeks fra monohardware skal man rette**flere** M3M-kameraer mod forskellige bølgelængder, samle dem i én multibåndsstak og beregne indekset *ud fra den*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel`-symbolerne skal **nøjagtigt** matche forudindstillingens kanalnavne* (der skelnes mellem store og små bogstaver; NDVI er med små bogstaver `red`,`nir` — se `--list-presets`), og båndsiden navngiver et bånd i den justerede stak (offline-tilstand accepterer også 0-baserede båndindekser, f.eks. `--channel red=0 --channel nir=1`).

Skiltegn i hele stakken er tokenet `M3M` i modelstrengen (det forekommer aldrig i en `M3C`-streng), der vises i GUI’en/SDKen som `is_mono`.

---

## Opsætning og finjustering af værts-NIC (LATTICE-arrays)

LATTICE-kameraer streamer GVSP via værtens Ethernet-adapter, så for arrays med flere kameraer er adapterens **driver**og**modtagelsesringstørrelse** lige så vigtige som forbindelseshastigheden. Forkerte indstillinger vises som en `FRAMES WILL DROP` / `Reduce ROI to enable`-fejl i panelet »Array-indstillinger« (og som `lattice network-analysis` / SDKens `analyze_array_network()`), selv når kameraerne i sig selv fungerer korrekt.

### USB 10GbE-adaptere — Realtek RTL8157 (&quot;Realtek USB 10GbE Family Controller&quot;)

| Element | Påkrævet værdi | Hvorfor det er vigtigt |
| --- | --- | --- |
| **Driverversion**|**≥ v10.67 (jan. 2026)**, INF `rtump64x64sta.inf` | Den forældede**2016**-driver (v10.65, `rtump64x64.inf`) håndterer nedlukning og fejlkontroller forkert med**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**ved nedlukning/genstart/dvale. Overgangen går i stå (~5 minutters timeout), brugeren tvinger computeren til at slukke, og gentagne uregelmæssige nedlukninger**ødelægger WMI-repositoriet**(PowerShell/værktøjer begynder at fejle med `Invalid class`) og**blokerer USB-stakken** ved næste opstart (NIC kan ikke aktiveres; USB-drev holder op med at blive registreret). Opdater fra realtek.com (eller dongle-leverandøren), før du stoler på korrekte genstarter. |
| **Modtagelsesbuffere**— nøgleord `ReceiveBufferLen` |**256**(maks. i driveren) | NIC&#x27;ens RX-ring. Driverens standardværdi på**32**efterlader kun ca. 0,26 MB brugbar ring — alt for lidt til en burst fra flere kameraer — så array-panelet rapporterer `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` og blokerer forbindelsen. Ved**256**er ringen stor (**~13,5 MB målt på laboratoriets 10GbE-vært**), hvilket giver RX-pipelineen reel kapacitetsreserve til GVSP-burst med flere kameraer. (Om en given konfiguration rent faktisk *opretter forbindelse*, afgøres af to kontroller — den **drain-aware**adgangs-kontrol og**aggregat-overskrivningskontrollen** – ikke en rå sammenligning mellem burst og ring; se [Array fps &amp; burst-model](#array-fps--burst-model).) |
| **Modtagende URB’er**— nøgleord `PendingReceives` |**64** (maks.) | USB-anmodningsblokke undervejs; øges sideløbende med modtagebuffere for at absorbere bursts. |
| **Jumbo-ramme** — nøgleord `*JumboPacket` | **9014** | Nødvendigt for 9000-byte GVSP-pakker (6 gange færre pakker pr. ramme end ved 1500). |

> ⚠️ **En opdatering af netværkskortdriveren NULSTILLER disse avancerede indstillinger til standardværdierne.**Efter opdatering eller udskiftning af adapterdriveren skal du**genanvende** `ReceiveBufferLen=256` og `PendingReceives=64`, ellers vil array-panelet igen blokere adgangen, selvom &quot;intet er ændret i hardwaren&quot;. Dette er den hyppigste årsag til, at et udstyr, der tidligere fungerede, pludselig nægter at oprette forbindelse.

Anvend fra en **administratorrettet** PowerShell (erstat med dit netværkskortnavn, f.eks. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` dækker USB 10GbE-adaptere.** Den registrerer nu adaptertypen og indstiller det korrekte nøgleord for modtagerringen: `*ReceiveBuffers`→2048 for PCIe-netværkskort (Intel I219 osv.) eller `ReceiveBufferLen`→256 + `PendingReceives`→64 for Realteks **USB** 10GbE-controller (som ikke eksponerer `*ReceiveBuffers`). Målværdierne begrænses til den maksimale værdi, som den enkelte driver rapporterer (`NumericParameterMaxValue`), så der aldrig skrives en værdi uden for-værdi. Kør det fra en **privilegeret** terminal; som ved enhver registreringsdatabase-baseret justering træder ændringen i kraft efter en genstart af adapteren eller en genstart af systemet. De manuelle `Set-NetAdapterAdvancedProperty`-kommandoer ovenfor er stadig et godt alternativ — de træder i kraft med det samme (genbinder adapteren) uden en genstart.

### Netværksgrundlag (alle LATTICE-forbindelser)

- **Adressering:** link-local `169.254.0.0/16` (GigE Vision LLA). Værten tager en statisk `169.254.x.x/16`; kameraer og DAQ-E tildeler sig selv adresser i samme interval. Der kræves ingen DHCP/gateway.
- **Pakke størrelse:**foretræk jumbo (9000), men lad auto-proben finde den — den måler på ny ved hver forbindelse og ser allerede forbi kameraets 1500-byte ICMP-loft via en GVSP-probe, så den ender på jumbo, uanset hvor kablet rent faktisk understøtter det. Fastlås kun med `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, når du ved bedre end sonden, og foretræk per-kommando frem for permanent: en fastlåsning springer sonden over, så hvis stienfaktisk ikke bære 9000,**hver** indfangning går i timeout med `SC_ERR_TIMEOUT -1011` (se [Miljøvariabler](#environment-variables)).
- **RX-ringen skaleres med `ReceiveBufferLen`:**ved standardværdien `32` er den anvendelige ring ~0,26 MB (for lille til enhver multi-cam-burst); ved den maksimale værdi `256` er den stor (~13,5 MB målt på laboratoriets 10GbE-vært), hvilket giver reel spillerum. Om en konfiguration kan oprette forbindelse, afgøres derefter af den drænbevidste adgangs-kontrol**og** den samlede over-subscription-kontrol nedenfor — ikke en rå sammenligning mellem burst og ring.

### Array fps &amp; burst-model

Sådan læser du panelet Array-indstillinger (og `lattice analyze-array` / `analyze_array_network` i SDK):

- **Burst summeres pr. kamera i hvert kameras reelle pixelformat.**Mono**M3M**-kameraer streamer**Mono12 (2 B/px)**;**M3C**-Bayer-kameraer streamer 8- eller 12-bit (TRI032S udsender automatisk BayerRG12, selv når der anmodes om BayerRG8). Så et billede i fuld opløsning fra 4 kameraer er**~12,6 MB, hvis alle er 8-bit, men ~25 MB med tre 12-bit mono-kameraer**. Projektionen afkoder hvert kameras format ud fra dets model (identitetscache), så datastrømmen svarer til det, der faktisk sendes – ikke en antagelse om en standard BayerRG8.
- **En USB-Ethernet-adapter er begrænset til 200 MB/s uanset, hvad der står på typeskiltet.** Effektivitetstabellen, der omregner en linkhastighed til et vedvarende tal, er afledt af PCIe; et USB-netværkskort angiver sin *Ethernet*-linkhastighed, men er begrænset af USB-bussen og dens driver. En USB 10GbE-dongle nåede tidligere op på ~1063 MB/s »vedvarende« — et tal, der aldrig blev testet — og den resulterende hastighedsregulering ødelagde 6–18 % af rammerne, mens den stadig rapporterede en sund mål-fps. USB-tilsluttede netværkskort er nu begrænset til **200 MB/s** som et absolut tal (grænsen er bussen, så den skalerer ikke i forhold til den angivne hastighed; en USB 1 GbE-adapter leverer ca. 80 MB/s og er ikke påvirket). `wire_ceiling_source` i kapacitetsregistret angiver dette med ord, og `nic_is_usb` markerer det. Tilsidesæt på en af to måder med `--wire-ceiling-mbps`.
- **Admittance tager højde for drain, ikke hele burst kontra ring.** En samtidig burst skal kun passe til den *midlertidige backlog* = `max(0, Σ per-cam arrival − host drain) × emit_window`, ikke hele bursten. I et fast-host / slow-cam-netværk (en **PCIe**10G-host + 4× 1 GbE-kameraer: ankomst ≈ 320 MB/s, dræning ≈ 1063 MB/s) dræner værten hurtigere, end kameraerne fylder op, backlog ≈ 0, så**admitterer**fuld-res sim-emit, selvom burst på 25 MB overstiger ringen på 13,5 MB. Sæt de samme fire kameraer bag en**USB**10GbE-adapter, og udledningen er 200 MB/s, ikke 1063 — indgangen overstiger den, og tabet viser sig som korrupte rammer snarere end som en lavere billedhastighed. På en 1 GbE-vært får kameraernes 31,25 MB/s DLThr-grænse gør, at indgående trafik overstiger udgående trafik → den**blokerer** korrekt (for *denne* klasse af blokeringer skal ROI reduceres, eller der skal anvendes binning ≥ 2). Adgang er en af **to** forbindelsesportene — den anden er den samlede overabonnements-kontrol nedenfor.
- **Den forventede fps er et konservativt loft for seriel hentning.**Værtshentesløkken henter i øjeblikket hvert kameras buffer**serielt**(~ét udsendelsesvindue pr. kamera hver), så cyklussen er afgrænset af `max(readout+emit, N × emit)`, hvor udsendelsen pr. kamera er begrænset til kameraets**adgangsforbindelse**(1 GbE ≈ 80 MB/s), ikke værts-uplinket. For et 4-kameraers 12-bit-array i fuld opløsning er det**~2,8 fps**, hvilket svarer til den målte ~2,7–3,0 fps, er bevidst**eksponeringsuafhængigt**, så i svagt oplyste scener kan den faktiske værdi falde en smule under loftet, efterhånden som eksponeringstiden forlænges. Den serielle hentning er den egentlige fps-begrænser; en parallelisering heraf ville hæve loftet mod hastigheden for enkeltudsendelse.
- **Samlet overtegning er en hård forbindelsesblokker.**Båndbreddetildelingen pr. kamera har en nedre grænse på**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), så når minimumsgrænsen er nået, kan den samlede efterspørgsel (`per_cam × N`) overstige den**kollisionssikre netværksgrænse**(`sustained × sim_emit_factor`). Praktiske lofter for fuld opløsning på 1 GbE:**6 kameraer ved 1500 MTU, 9 med jumbo**. Dette loft afhænger udelukkende af netværksforbindelsen og minimumsgrænsen – det er**uafhængigt af rammestørrelse**, så**binning og mindre ROI hjælper IKKE** (de sænker antallet af bytes pr. *ramme*, ikke de GevSCPD-styrede bytes pr. *sekund*); de eneste løsninger er færre kameraer, jumbo-rammer fra ende til ende eller et hurtigere netværkskort. Symptomet vil være tab af GVSP-pakker, ikke en jævn reduktion af fps, så `analyze-array` nulstiller de opnåelige fps-tal og udskriver `**OVER-SUBSCRIBED**`, og `array-connect` med en fastlåst opløsning **nægter at oprette forbindelse** (walk-down sorterer ellers rammerne ned, hvilket heller ikke fjerner denne type blokering). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` nedgraderer afvisningen til en kraftig advarsel til bench-arbejde — se [Miljøvariabler](#environment-variables).

### Array-tilstand — hvilket undersystem mister rammer

Et tilsluttet arrays `GET /api/camera/array/<array_id>/capability` indeholder en live
`health`-blok, der revurderes i et rullende **10-sekunders** vindue. Den opdeler rammetab
i de to årsager, der kræver modsatte løsninger, i stedet for at rapportere én &quot;ufuldstændig&quot;
rate, der ikke angiver nogen af dem:

| Felt | Hvad det betyder | Hvilket undersystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (pr. serienummer) | Framen **ankom og var strukturelt defekt**— GVSP-pakketab. |**Netværk**: ledningskapacitet, pacing, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (pr. serienummer) | Rammen **kom slet ikke**— kameraet udløste ikke, eller der kom intet ud af det. |**Udløser / synkronisering**: M8-kabel, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Det dårligste kameras hastighed for hver. | — |
| `per_cam_rate_pct` | Samlet ufuldstændighedsrate pr. kamera (begge årsager tilsammen). | — |
| `stable_for_seconds` | Hvor længe hvert kamera har ligget under 0,01 %. | — |

Over 5 % logger backend’en en `[array-health <id>] WARN`-linje, der angiver opdelingen — ved den
første overskridelse, ved et skift i alvorlighedsgraden, en gang i minuttet, så længe det varer, og én gang, når
det forsvinder. Den korrupte halvdel udskriver `[gvsp-corrupt <SN>]` ved det første hit pr. kamera og
årsag, derefter en sammenfatning hvert 60. sekund. Hver evaluering havner stadig i backend-logfilen;
tællerne går videre ved hver buffer, uanset hvad der udskrives.

Den samme post angiver det tal, som hele tildelingen afhænger af:

| Felt | Betydning |
| --- | --- |
| `wire_ceiling_mbps` | Værtens gældende vedvarende båndbredde, MB/s. |
| `wire_ceiling_source` | Hvor tallet stammer fra, i ord — f.eks. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` eller `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, når `--wire-ceiling-mbps` (eller GUI&#x27;enss **Wire Budget**-felt) indstiller det. |
| `nic_is_usb` | `true` for en USB-Ethernet-adapter — se ovenstående grænse på 200 MB/s. |

**Fortolkning:** `gvsp_corrupt_rate_pct` forskellig fra nul med `never_arrived_rate_pct` på 0
betyder, at udløsning og kabelsynkronisering er perfekt, og at 100 % af tabet ligger på netværksstien — sænk `--wire-ceiling-mbps` og tilslut igen. Det omvendte mønster peger i stedet på
synkroniseringskablet eller triggerledningen.

> **`--target-fps` er ikke afgørende for korrupte rammer.** GevSCPD-pacing skrives
> én gang ved opkobling, så en sænkning af triggerfrekvensen ændrer duty cycle og ikke
> burst-frekvensen for samtidig udsendelse. En målt 5×-efterspørgselsreduktion gav ingen forbedring;
> at sænke ledningsloftet fra 240 til 200 MB/s reducerede andelen af4 %
> korrupt til 0,00 %.

> **Automatisk nedskalering under datastrømmen er ikke tilgængelig på TRI032S-firmwaren.** Et kørende array
> kan ikke selv løse dette; afbryd forbindelsen og tilslut den igen, så forbindelsestidsvælgeren kan
> planlægge på ny med den nye grænse.

### Symptom → løsning

| Symptom (Arrayindstillinger / tilslutning / `analyze_array_network`) | Årsag | Løsning |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` nulstilles til 32 (typisk efter en driveropdatering) | Indstil `ReceiveBufferLen`→256, `PendingReceives`→64; åbn panelet igen (genstart backend, hvis den har cachelagret den gamle ringstørrelse) |
| Genstart/nedlukning går i stå; senere `Invalid class` WMI-fejl, netkortet kan ikke aktiveres, USB-drev mangler | Gammel Realtek USB 10GbE-driver fra 2016 → BSOD `0x9F` → tvungen-slukninger | Opdater adapterdriveren til ≥ v10.67 (2026), og anvend derefter ovenstående indstillinger for modtagerringen igen |
| Tilslutning lykkes, men returnerer en opløsning under den native | Smart-prep har automatisk formindsket rammen, så den passer til kablet | Opgrader forbindelsen / accepter formindskelsen / `--force-tier slip-emit-and-capture` |
| Arrayet rapporterer en korrekt mål-fps, men leverer kun en brøkdel heraf; `health.gvsp_corrupt_rate_pct` ikke-nul, `never_arrived_rate_pct` 0 | Værten har estimeret et for stort ledningsbudget i forhold til det, der faktisk kan opretholdes (typisk på en USB-Ethernet-adapter, en smal PCIe-bane eller et delt netværk) | Opret forbindelse igen med en lavere `--wire-ceiling-mbps` og kontroller tilstandsblokken igen. **Ikke** `--target-fps` — GevSCPD-pacing er fastsat ved opkobling |
| Kameraer mangler i offentliggjorte grupper; `health.never_arrived_rate_pct` er ikke nul, `gvsp_corrupt_rate_pct` er 0 | Trigger-/synkroniseringssti — kameraerne udløser ikke, det er ikke et netværksproblem | Kontroller M8-synkroniseringskablet og `--line`; bekræft, at alle enheder er aktiveret (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` overskredet i `analyze-array`, eller forbindelsesafvisning med fastlåst opløsning (`array over-subscribes the wire`) | Samlet behov pr. kamera (8 MB/s minimum × N kameraer) overskrider det kollisions-sikre ledningsloft — 6 kameraer i fuld opløsning på 1 GbE @1500 MTU, 9 med jumbo | Færre kameraer, jumbo-rammer fra ende til ende eller et hurtigere netværkskort. **ROI/binning vil IKKE hjælpe** (loftet er uafhængigt af rammestørrelsen). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` tilsidesætter på testbænken (accepterer pakketabet) |

---

## `chloros-cli daq`

Kommandoer til spektralsensoren. To klasser:
- **`pool-*`**— tynde HTTP-klienter, der styrer sensoren via backendens permanente pool.**Dette er den understøttede vej, og den eneste, der findes i den leverede CLI.** Backend&#x27;en ejer transporten, så GUI&#x27;en samt scriptsene CLI og SDK alle deler ét aktivt håndtag i stedet for at kæmpe om den serielle port.
- **Alt andet**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — direkte hardwareadgang, beskrevet nedenfor for fuldstændighedens skyld. Disse kræver `daq`-Pythonspakken, som**ikke er inkluderet i nogen leveret artefakt**: den kompilerede CLI udelukker den (`scripts/Build-CLI.ps1` indstiller `--nofollow-import-to=daq`, og transportfilerne `pyserial` / `bleak` / `zeroconf` med den), og PyPI-pakken SDK indeholder den heller ikke. De kører kun fra en kilde-checkout, så betragt dem som en intern udviklingsvej for MAPIR snarere end noget, man skal benytte sig af.
- **`discover` / `list`** ligger midt imellem de to: de er direkte hardwarekommandoer fra en kildekode-checkout, men i en udgivet build falder de tilbage til `pool-discover`, og backenden udfører scanningen. Så scanningen fungerer overalt — hvilket er vigtigt, fordi det er den eneste måde at finde en DAQ-M&#x27;s BLE MAC på.

> **`chloros-cli daq --help`** (og `-h` / `help`) viser `pool-*`-underkommandoerne — hjælpen dirigeres bevidst til pool-klienten, så den afspejler de kommandoer, der rent faktisk kører. Hvis du påkalder en underkommando til direkte hardware på en udleveret build, afsluttes den med en eksplicit fejlmeddelelse, der angiver det manglende pakke og henviser dig tilbage til `pool-*`; intet mislykkes uden fejlmeddelelse. (`discover` / `list` er undtagelsen — de omdirigerer til `pool-discover` og fungerer uden problemer.)
>
> **Alt, hvad en kunde har brug for, er tilgængeligt via `pool-*`** — opret forbindelse, stream, optag kalibrerede `.daq`-filer og skift kap-profiler. DAQ&#x27;en kan også styres fra Python med `chloros_sdk.connect_daq_sensor()`, som bruger den samme samlede sti.

### Arbejdsgang ved første tilslutning af DAQ-sensor

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*`-reference

| Underkommando | Formål |
| --- | --- |
| `daq pool-connect` (smart-detect) | Åbn en sensor i backend-puljen. |
| `daq pool-connect --port PORT` | DAQ-U på en bestemt seriel port. |
| `daq pool-connect --ble` | DAQ-M via BLE, MAC-adresse scannes automatisk. |
| `daq pool-connect --mac MAC` | DAQ-M via BLE på en kendt MAC-adresse (forudsætter `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E via Ethernet på en kendt vært. |
| `daq pool-connect --eth` | DAQ-E via Ethernet, vært automatisk opdaget (mDNS + ARP-fallback; fungerer fra en tom ARP-cache på Windows og Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Juster integrationsvindue / AE-tilstand. |
| `daq pool-connect --no-stream` | Opret forbindelse, men start ikke streaming endnu (fortsæt med `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Cap-korrektionsprofil. Standardindstillingen i backend er `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Scan alle transporttyper for sensorer, du kan oprette forbindelse til, uden at oprette forbindelse. **Sådan finder du en DAQ-M&#x27;s BLE MAC.** `daq discover` / `daq list` dirigeres automatisk hertil i de udleverede builds. Sensorer, der allerede er åbne i puljen, vises ikke — en tilsluttet DAQ-M holder op med at sende signaler — så brug `pool-list` til dem. |
| `daq pool-list` | Vis alle sensorer i backend-puljen. |
| `daq pool-disconnect --sensor-id ID [--all]` | Frigiv. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Seneste N spektrumrammer. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Genoptag / sæt streaming på pause. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Start/stop en .daq-optagelse. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Skift-korrektionsprofilen under kørsel. |

### Direkte hardware-underkommandoer (kun ved hentning af kildekode — ikke i de leverede builds)

> Anført for fuldstændighedens skyld. Disse kræver pakken `daq` Python samt `pyserial` / `bleak` / `zeroconf`, hvoraf ingen leveres i den kompilerede CLI eller på PyPI SDK — de kører kun fra en MAPIR kildekode-checkout. **Hvis du bruger en udgivet Chloros-build, skal du i stedet bruge ovenstående `pool-*`-kommandoer**; de dækker valg af forbindelse, stream, optagelse og cap.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Åbn, opret forbindelse til og kør et gemt Chloros-projekt (en mappe med `cameras.json` + `sensors.json` + `project.json`). Alt går gennem backend, så GUI&#x27;en og CLI viser den samme hardwarestatus.

### Reference til underkommandoer

| Underkommando | Formål |
| --- | --- |
| `project open PATH` | Udskriv projektets enhedsmanifest (kameraer, arrays, sensorer). |
| `project devices PATH [--reconnect]` | Vis eller gentag opdagelse. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Opret forbindelse til alle gemte kameraer / arrays / sensorer. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Enkeltoptagelse fra et navngivet kamera eller array. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Serieoptagelse med N billeder fra et navngivet kamera eller array (`-n/--count` standard 5; `-i/--interval` sekunder mellem billeder, standard 0). Array-serier fjerner dubletter af gentagne synkroniserede grupper (forældelsesvagt), så et array med delvis cyklus ikke kan returnere N kopier af ét billede; viser resultater pr. iteration. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Stream til disk via et backend-job. `--poll-interval` = sekunder mellem `/stats`-afspørgninger (standard 2,0). |
| `project sensor read PATH NAME [--json]` | Seneste spektrumramme. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Optag .daq. |
| `project run PATH RECIPE.yaml` | Udfør en YAML/JSON-optagelsesopskrift. `--dry-run` validerer uden at køre. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Beregn justering for et array — se [flagtabellen nedenfor](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Udskriv den aktuelle aligneringsprofil. |
| `project align clear PATH NAME` | Slet den cachelagrede profil. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Juster en slaves transformation. |
| `project align export PATH NAME --to FILE` | Gem profilen i JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Indlæs en gemt profil. |

#### `project align calibrate`-indstillinger

| Flag | Standard | Beskrivelse |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Justeringsmetode. **Disse stavemåder adskiller sig fra `lattice align-calibrate`**, som bruger de korte former `orb` / `akaze` / `phase`; de to kommandoer er ikke indbyrdes udskiftelige med denne flag. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformér modellen, så den passer. |
| `--frames N` | `1` | Synkroniserede billedøjebliksbilleder til gennemsnit. |
| `--reference SN` | masteren | Referencelignende kamera; alle andre medlemmer forvrænges til det. |
| `--max-features N` | `5000` | Øvre grænse for antal ORB-træk. |
| `--ratio-threshold F` | `0.75` | Lowe&#x27;s ratio test. |
| `--ransac-threshold-px F` | `3.0` | RANSAC-tærskel for inliers. |
| `--min-matches N` | `15` | **Kvalitetsgrænse** — afvis løsningen, hvis antallet af inlier-match er lavere end dette. |
| `--max-reproj-err-px F` | `4.0` | **Kvalitetsgrænse** — afvis løsningen, hvis RMS-reprojektionsfejlen er større end denne. |
| `--checkerboard RxC` | — | Brætgeometri for `--method checkerboard`, f.eks. `9x6`. |
| `--name PROFILE` | tom | Profilnavn indlejret i den gemte JSON. **Ikke array-navnet** — det er det positionelle `NAME`. |

De to kvalitetskontrolpunkter er årsagen til, at en kalibrering kan lykkes med at løse problemet og alligevel
nægte at gemme: en profil, der fejler i en af dem, vil uden varsel registrere alle
efterfølgende optagelser forkert, så den afvises i stedet for at blive gemt.

### Eksempler

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Opskrifts-DSL

`project run RECIPE.yaml` accepterer en YAML- eller JSON-fil, der beskriver en række handlinger:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Understøttede handlinger: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Handlingen `burst`-handlingen tager `name` (påkrævet), `count` (standard 5), `interval` (sekunder, standard 0), `output`, `format` og `settings` (samme indstillingsstruktur pr. kamera som `apply`); array-bursts bruger den samme nyligtsynkroniseret gruppe-watchdog som `project burst`.

Kør det:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Miljøvariabler

| Variabel | Effekt |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Tilpas backend-URLen (standard `http://127.0.0.1:5000`) — **respekteres kun af kommandofamilierne `lattice`, `project` og `daq pool-*`.** Kernekommandoerne (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) fastlåser `http://127.0.0.1:<port>` og ignorerer denne variabel (IPv4-literalet omgår Windows `localhost`→`::1` ~2 s-per-anmodning-straf), så de altid retter sig mod den lokale maskine. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` nedgraderer arrayet ved-afslag på forbindelse ved abonnement (samlet efterspørgsel pr. kamera &gt; kollisionssikkert ledningsloft med `pin_resolution`) til en højlydt advarsel og fortsæt, hvorved GVSP-pakketab accepteres. Kun til brug i testmiljø — se [Array fps &amp; burst-model](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Indstilles af selve CLI; fortæller backend&#x27;en at aktivere parallelbehandling. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` springer GVSP-fallback-proben over (kun ICMP-resultater). **Dette slår jumbo fra, det dæmper ikke blot loggen** — kameraet svarer kun på DF-pings op til 1500 på hver vej, så denne probe er det eneste, der kan detektere jumbo. Sparer ~1 s pr. kamera pr. forbindelse; koster ~1,45× ledningsloftet, hvis netværket *kunne* have håndteret jumbo. SDKen advarer, når du indstiller den. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fastlægger GVSP-pakkestørrelsen til N bytes; springer sonderingen helt over. Foretræk indstilling pr. kommando (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) frem for at indstille den permanent: en fastlåst størrelse holder op med at tilpasse sig det foranliggende netværk, og hvis man fastsætter 9000 på en rute, der ikke kan håndtere jumbo, vil **alle** indfangninger gå i timeout med fejlkode `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Tilpas Nuitka onefile-udtrækning til et andet bibliotek. CLI bruger automatisk `/mnt/ssd/tmp`, hvis den findes. |

---

## Afslutningskoder

| Kode | Betydning |
| --- | --- |
| `0` | Succes. |
| `1` | Generel fejl (de fleste underkommandofejl). |
| `2` | Argumentfejl. |
| `130` | Afbrudt af Ctrl+C. |

---

## Tips til fejlfinding

- **&quot;Login kræves&quot;** → Kør `chloros-cli login EMAIL PASSWORD` én gang på denne maskine.
- **&quot;backend kan ikke nås&quot;** → Start desktop-appen Chloros, eller kør backend-binærfilen direkte (`chloros-backend`), eller tjek `CHLOROS_BACKEND_URL`, hvis det er fjernbetjent.
- **`lattice`-kommandoer mislykkes med &quot;LATTICE-kameradrivere ikke fundet&quot;** → Arena-SDK-runtime er ikkeikke installeret; CLI leveres med `win32api` medfølgende på Windows, men C-runtime er en del af GUI-installationsprogrammet.
- **Array connect / Array Settings viser &quot;FRAMES WILL DROP&quot; eller &quot;Reduce ROI to enable&quot;** → Værts-NIC&#x27;ens modtagelsesring er for lille (nulstilles normalt til 32 efter en opdatering af NIC-driveren). Se [Opsætning og finjustering af værts-NIC](#host-nic-setup--tuning-lattice-arrays) — indstil `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Maskinen går i baglås ved genstart/nedlukning, hvorefter WMI `Invalid class` / NIC kan ikke aktiveres / USB-drev mangler** → Forældet USB 10GbE-adapterdriver forårsager `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Opdater adapterdriveren — se [Opsætning og finjustering af værts-NIC](#host-nic-setup--tuning-lattice-arrays).
- **Jetson-swap-advarsel** → Tilføj fil-baseret swap; kommandoen `CLI` udskriver de nøjagtige `fallocate` / `swapon`-kommandoer.
- **DAQ-direkte kommandoer mangler** → Forventet: den medfølgende `chloros-cli`-pakke udelukker bevidst `daq`-pakken bevidst udelukker `pool-*` (PyPI-SDK indeholder den heller ikke). Brug `pool-*`, som styrer den samme sensor via backend, eller `chloros_sdk.connect_daq_sensor()` fra Python.

---

## Se også

- [Python SDK Reference](sdk-reference.md) — programmatisk ækvivalent til alle CLI-kommandoer.
- [DAQ-sensorvejledning](../daq/README.md) — sensorspecifik tilslutning + kalibrering.
- Online-dokumentation: `https://mapir.gitbook.io/chloros/cli`

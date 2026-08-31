# Chloros Python SDK Reference

**Version:**

1.2.0**Genereret:**

29. juli 2026 kl. 19:19 ·**Revideret:**

30. august 2026**Pakke:** `chloros-sdk` (PyPI)**Målgruppe:** Optimeret til brug i store sprogmodeller (LLM); læsbar for mennesker.**Omfang:** Alle offentlige klasser, funktioner og hjælpefunktioner, der stilles til rådighed af `import chloros_sdk`, med eksempler, der kan kopieres og indsættes, og som dækker billedbehandling, styring af enkeltkameraer, synkroniserede arrays, DAQ-sensorer og projektautomatisering.

Hvis du kun har brug for de vigtigste punkter, så spring til:
- [Installation og hurtigstart](#installation)
- [Smart-Connect til LATTICE-arrays](#smart-connect-for-lattice-cameras)
- [DAQ-sensorsessioner](#daq-sensor-sessions)
- [Projektautomatisering](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Arkitektur på 60 sekunder

SDKen er et thin-Python-lag oven på Chloros-backend (den samme Flask-server, som desktop-GUI&#x27;en og CLI bruger). Til automatisering importerer du `chloros_sdk` og kalder metoder på højt niveau; bag kulisserne bliver hvert kald til en HTTP-anmodning til det lokale backend på port 5000 — `http://127.0.0.1:5000/api/...` (bevidst ikke `localhost`, som først omdirigeres til `::1` på Windows og tager ca. 2 sekunder pr. anmodning mod et backend, der kun understøtter IPv4). Backend&#x27;et styrer hardwarepuljen — kameraer, DAQ-sensorer, justeringsprofiler, billedbuffere — så SDK-scripts kan køre sideløbende med GUI’en uden at kæmpe om serielle porte eller netværkskortets båndbredde.

Der er tre grænseflader, du vil bruge:

1. **`ChlorosLocal` + frie funktioner** (`process_folder`, `process_lattice_capture`) — Billedbehandlingspipeline. Kør en hel mappe gennem kalibrering / debayer / indekseksport fra ét Python-kald.
2. **Smart-connect-håndtag** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Åbn en vedvarende backend-session til live-hardware. Samme &quot;smart-prep&quot;-forløb som i GUI’en: netværkssonde, automatisk valg af lag, PTP, AE-seeding, GPIO-triggerkonfiguration.
3. **`ChlorosProject` / `open_project`** — Indlæs et gemt projekt (mappe med `cameras.json` + `sensors.json` + `project.json`), tilslut alt på én gang, og udfør optagelser med navngivne håndtag.

Overflader 1 og 2 **starter automatisk en lokal backend**, hvis der ikke allerede er en, der lytter (den samme medfølgende binærfil, som GUI&#x27;en/CLIen starter) — så et simpelt script fungerer fra en ny shell, uden at du først skal starte en backend. Overfør `auto_start_backend=False` for at fravælge dette (f.eks. når der peges på en fjernbackend, som aldrig startes). Se [Automatisk start af backend](#backend-auto-start). Surface 3 opfører sig anderledes: `open_project()` tager ingen `auto_start_backend`-parameter, og `connect_all()` starter aldrig en backend — den tester `http://127.0.0.1:5000` én gang, og hvis der ikke kommer noget svar, falder den lydløst tilbage til direkte (backend-fri) `lattice_sdk`-enhedsstyring. Kun `proj.process()` og `stream(..., overlays=True)` opretter en `ChlorosLocal()` (som starter automatisk).

Alle tre er autorisationsbeskyttede: Kør `chloros-cli login` én gang på maskinen, eller log ind via skrivebords-GUI’en. Opkald til SDK uden en gyldig session udløser `ChlorosAuthenticationError`.

Krav:
- Python 3.7+ (som angivet i pakken; udviklet/testet på 3.10)
- Chloros Desktop installeret lokalt (backend-binærfilen følger med i installationsprogrammet)
- Aktivt Chloros+-login. SDK / CLI kræver mindst **Copper**-niveau (Copper / Bronze / Silver / Gold); det gratis**Iron**-niveau har ingen adgang til SDK / CLI . Dette håndhæves**på serversiden**: hver anmodning med flagget SDK / CLI skal indeholde både en aktiv session og et betalt abonnement, ellers returnerer backend `403` sammen med `error_code: PLAN_UPGRADE_REQUIRED` (vist som `ChlorosLicenseError` af `ChlorosLocal` og som `ChlorosConnectError` af `connect_*`-hjælperne). En afmeldt bruger får i stedet `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) i stedet — de to er forskellige, fordi en genkørsel af `chloros-cli login` løser det første problem, men ikke det andet.
- Offlinebrug understøttes inden for planens afdragsfri periode: niveauet læses fra servervalideringscachen (5 minutter) eller den signerede, maskinbundne licenscache (30 dage for månedsabonnementer, indtil abonnementets udløb for årsabonnementer). Når denne frist udløber, skifter abonnementet til gratis, og adgangen til SDK / CLI stopper, indtil maskinen kan oprette forbindelse til serveren én gang. `chloros-cli status` (`GET /api/license-status`) forbliver tilgængelig på det gratis niveau, så årsagen er tydelig — det er den eneste rute SDK / CLI, der er undtaget fra niveaubegrænsningen.
- Windows 10/11 64-bit, **Ubuntu 22.04 LTS eller nyere**, eller Jetson (JetPack 6). Ubuntu 20.04**ikke** understøttet: `.deb`&#x27;s afhængigheder stammer fra det, som backend&#x27;en linker til, herunder `libc6 (>= 2.34)`, og Focal leveres med glibc 2.31.

---

## Installation

Python SDK er et tyndt Python-lag oven på Chloros-backend. Til alt andet end nogle få DAQ-workflows skal du have **Chloros-desktop-pakken installeret lokalt** (Windows installationsprogram eller Linux `.deb`) — det er den, der leverer backend-binærfilen, Arena-SDK-runtime til LATTICE-kameraer samt kalibreringspakkerne.

Seneste downloads: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Trin 1 — Installer platformspakken Chloros

#### Windows (.exe)

1. Download `Chloros-Setup-x.y.z.exe` fra downloadsiden.
2. Kør installationsprogrammet, og følg vejledningen. Standardinstallationsstien er `C:\Program Files\MAPIR\Chloros\`.
3. Start Chloros mindst én gang, og log ind med din Chloros+-konto.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Trin 2 — Installer Python SDK

**Chloros-installationsprogrammet leveres med et tilhørende SDK-wheel.** Hvert Windows-installationsprogram og Linux .deb placerer en `chloros_sdk-X.Y.Z-py3-none-any.whl` på disken, der nøjagtigt matcher GUI-/CLI-/backend-versionen. Du behøver ikke at holde øje med PyPI for at holde dig synkroniseret.

#### Windows

Installationsprogrammet kører automatisk `pip install` mod den medfølgende wheel ved hjælp af din systemPython (`py.exe`-launcher foretrækkes, falder tilbage til `python -m pip`). Ingen handling kræves — `import chloros_sdk` fungerer i dit Python-miljø efter en vellykket installation. Hvis der ikke er nogen Python på maskinen, springer installationsprogrammet dette trin over uden at give besked, og GUI&#x27;en samt CLI fortsætter med at fungere.

#### Linux (.deb)

.deb-filen placerer wheel-filen i `/usr/lib/chloros/sdk/`. `postinst` viser den nøjagtige kommando — PEP 668-distributioner afviser som standard globale pip-skrivninger, så vi foretager ikke automatisk installation:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Ved air-gapped Jetson-implementeringer foregår dette fuldstændigt offline — wheel-filen findes allerede på disken.

#### Offentlig PyPI

For værter, der kun bruger pip (ingen Chloros-desktop-pakke installeret; workflow med fjernbackend eller udelukkende DAQ):

```bash
pip install chloros-sdk
```

PyPI opdateres ved installationer af release-versioner, så det offentliggjorte wheel matcher den seneste stabile udgivelse. Dev-builds (f.eks. `1.1.4.dev1`) leveres kun via det medfølgende installer-wheel.

#### Bekræft

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+ abonnement kræves.** Alle SDK-kald kræver et aktivt Chloros+ login. Kør `chloros-cli login user@example.com 'YourPassword'` én gang pr. maskine; loginoplysningerne gemmes i `~/.chloros/`.

### Har jeg brug for Desktop-pakken?

Pip-pakken alene er **ikke** tilstrækkelig til de fleste arbejdsgange. Her er, hvad hver SDK-enhed har brug for:

| SDK-overflade | Kræver Desktop-pakken? | Hvorfor |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Ja** | Starter backend-binærfilen automatisk på `/usr/lib/chloros/chloros-backend` (Linux) eller `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Ja**(lokalt)**/ Nej**(fjernt) | Rene HTTP-klienter via backend. Lokalt backend → desktop-pakke påkrævet. Fjernt backend → `backend_url=`**gennem en tunnel** (se Fjern-backend-tilstand — medfølgende backends binder kun til loopback). |
| `ChlorosProject` / `open_project` | **Ja** | Kører gemte projekter via backenden. |
| Direkte LATTICE-klasser (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Ja** | Kræver Arena-SDK-native runtime, som følger med i desktop-pakken. `CAMERA_AVAILABLE` er ellers `False` ved import. |
| Direkte DAQ-klasser (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Nej** | Ren Python via pyserial/bleak/zeroconf. Et miljø, der udelukkende bruger pip, kan styre DAQ&#x27;er fra ende til ende. |

### Fjern-backend-tilstand (pip-only-vært, via tunnel)

> **Den medfølgende backend kan ikke nås via LAN.** Produktions
> -builds binder kun til loopback (begge loopback-familier) og afviser kategorisk den
> eneste ikke-loopback-tilstand (`CHLOROS_CLOUD_MODE`), så
> `backend_url="http://<lan-ip>:5000"` **kan ikke fungere mod en installeret
> Chloros** — dette mønster har kun nogensinde fungeret mod en source/dev
> backend. For at styre en backend på en anden maskine skal du selv videresende dens loopback-
> port og pege SDK mod tunnelen:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Headless-/CI-/robotværter kan beholde én maskine med den fulde desktop-installation som »Chloros-serveren« og `pip install chloros-sdk` alle andre steder — men transporten mellem dem foregår via den ovenfor beskrevne, brugeroprettede tunnel, ikke via en direkte LAN-URLation.

> **Kendt begrænsning — `ChlorosLocal` understøtter ikke udelukkende pip.** `ChlorosLocal(backend_url=BACKEND)` finder i øjeblikket en lokal backend-binærfil i sin konstruktør *inden* den søger på URL og genererer fejlen `ChlorosBackendError` („Chloros-endend not found…&quot;), når der ikke er installeret nogen desktop-pakke — selv med en tilgængelig fjernbackend. Kun smart-connect-grænsefladen ovenfor (`connect_camera` / `connect_array` / `connect_daq_sensor`, plus `analyze_array_network` og hjælpeprogrammerne `list_*` / `discover_*`) fungerer fra en host, der kun kører pip.

### DAQ-only-workflow (host, der kun kører pip)

Hvis du kun har brug for DAQ-sensorer og ikke bruger LATTICE-kameraer eller billedbehandling, er pip-pakken selvstændig:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Intet backend, ingen .deb, ingen Chloros+ login kræves til direkte hardware-DAQ-arbejde.

---

## Hurtigstart

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Topniveau-API-indeks

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Billedbehandling — `ChlorosLocal`

Den overordnede pipeline-klasse. Starter backend ved første brug, opretter og konfigurerer projekter, overvåger fremskridt og returnerer oversigter efter kørsel.

### Konstruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metoder

| Metode | Beskrivelse |
| --- | --- |
| `create_project(project_name, camera=None)` | Opretter et nyt projekt (valgfrit med en kameraskabelon som f.eks. `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Importerer RAW/TIF/JPG/DNG-billeder **og `.daq` lyssensoroptagelser**. Returnerer `count` (billeder) og `scan_count` (optagelser). Viser kun en advarsel, hvis mappen ikke indeholder nogen af delene. |
| `export_light_sensor(daq=True, csv=True)` | Skriv kalibrerede `.daq` + `.csv` for hver lyssensoroptagelse i projektet, til `<project>/Light Sensor/`. Se [Lyssensoroptagelser](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Indstil behandlingsknapper. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Kør pipelinen. Returnerer `{"status": "complete", "async": False}` samt en `summary`-nøgle, når backend leverer en — se [Resumé og tip efter kørsel](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Kontroller backend-tilstanden. |
| `logout()` | Ryd cachelagrede legitimationsoplysninger. |
| `shutdown_backend()` | Afslut backend (hvis SDK -started). |
| `discover_cameras()` | Find LATTICE-kameraer **via denne instans&#x27; backend** (`/api/camera/discover`). Returnerer en liste over ordbøger (`serial`, `model`, `ip`, …) — samme struktur som den, GUI&#x27;en/CLIet ser. Tom liste, hvis der ikke findes nogen, eller hvis backend&#x27;en ikke kan nås. |
| `camera_capture(output_dir, format="tiff", **settings)` | Optag et enkelt billede**via backend&#x27;et**(startes automatisk af dette håndtag), så det får samme forberedelse som GUI&#x27;en/ CLI (12-bit som standard, genbrug af pool, indlejrede kalibreringsmetadata). Løs målet med `serial=` eller `device_index=`; videregiv `exposure`/`gain`/`pixel_format`/`preset` som `**settings`. Returnerer den gamle metadatadikt (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Generer overlay-sammensatte forhåndsvisningsbilleder fra en samlet — tynd MJPEG-klient via backendens `/api/camera/<serial>/stream-annotated`-rute (zebra / gitter / sigtekorn / histogram / peaking / spot tegnet på serversiden). `decode=True` genererer BGR-arrays; `False` genererer rå JPEG-bytes. Kan også tilgås pr. projekt som `ChlorosProject.stream(overlays=True)`. |

Brug som kontekstmanager for garanteret oprydning:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Lys-sensoroptagelser — kalibreret `.daq` + `.csv`

En DAQ-U / DAQ-M / DAQ-E kan optages **uden** dens kalibreringspakke. Det er
netop, hvad de offentlige [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
optagere (`record_daq.py`) gør som standard: de skriver rå sensortællinger og stempler
filen, så Chloros henter den pågældende sensors fabrikskalibrering **via serienummer** — først fra den lokale cache
, derefter fra MAPIR Cloud — og anvender den ved import.

Chloros skriver resultatet tilbage som to produkter pr. optagelse under
`<project>/Light Sensor/`:

| Produkt | Hvad det er |
| --- | --- |
| `<name>_calibrated.daq` | Det genbehandlingsbare arkiv — samme skema som en live-optagelse, men nu med angivelse af den pakke, der har genereret det. Genimport af det **ikke** kalibrerer det en anden gang. |
| `<name>_calibrated.csv` | Spektral irradians i W/m²/nm på sensorens eget bølgelængdegitter, én række pr. måling, plus fotometriske kolonner (total effekt, fotopisk/skotopisk lux, PPFD og dens opdeling i blå/grøn/rød, spidsbølgelængde). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Kun sensorer uden bundle (DAQ-A).** Rå spektrale sensortællinger — *ikke* irradians. Se nedenfor. |

`process()` udfører denne eksport som et af sine trin. Det kræver **ikke** billedmateriale:
en lyssensor, der flyves alene, udgør en førsteklasses arbejdsgang, og et sådant projekt har per definition ingen
billeder.

**DAQ-A-optagelser eksporteres som rå tællinger.** DAQ-A-familien er ældre end det-serielle
bundlesystem og har ingen bundle at hente — den kalibreres i stedet i felten mod et
reflektansmål, hvilket er grunden til, at den aldrig har haft brug for en. Disse optagelser eksporteres
under en `_raw`-stamme i stedet for `_calibrated`: et andet filnavn i stedet for et flag
inde i filen, fordi angivelsen skal kunne overleve at blive videresendt via e-mail som et simpelt navn.
`.csv`-overskriften angiver `raw spectral sensor counts (NOT irradiance)` og advarer om, at
værdierne er sammenlignelige **inden for** filen — præcis det, som målbaseret kalibrering bruger
dem til — og ikke på tværs af sensorer. De effekt-afhængige fotometriske kolonner (total effekt,
fotopisk/skotopisk lux, PPFD) returneres som **NULL** i stedet for at blive integreret ud fra tællinger.

En DAQ-U / DAQ-M / DAQ-E, hvis bundle simpelthen ikke kunne hentes, **springes stadig over**,
og der skrives ikke rådata: der findes bundlet, og er »genopret forbindelse og genbehandl« et reelt råd.

Ældre **v1.01 / v1.02**-optagelser (en DAQ-A-SD skriver disse) har ingen epoke pr. aflæsning,
kun filens skrivetid. Matcheren mellem billede og nedadgående lysstrøm afviser dem stadig — at matche en
ramme mod et skrivetidspunkt ville være usynligt forkert — men eksportøren læser dem, og
CSV udskriver `clock=daq_created_on`, så produktet angiver, hvilket ur det følger.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

En optagelse, hvis kalibreringspakke ikke kan hentes (offline eller en sensor uden
kalibrering i filen), rapporteres under `skipped` **med årsagen**. Den skrives aldrig
udskrevet som en »kalibreret« fil, der indeholder rå tællinger — opret forbindelse til internettet og
kør programmet igen, så eksporten gennemføres.

### Statusopkald

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Oversigt og tip efter kørsel

Når kørslen er afsluttet, henter `process()` `GET /api/processing-summary` og vedhæfter brødteksten som `result["summary"]`. Hentningen sker efter bedste evne og blokerer aldrig et vellykket returværdi — hvis oversigten ikke er tilgængelig, falder `process()` tilbage til den almindelige `{"status": "complete", "async": False}`-form. Hver post i `summary["hints"]` — fulde sætninger med den foreslåede afhjælpning, f.eks. hvorfor en kørsel gav nul output — udsendes også som en Python `UserWarning`, så kørsler med nul output er selvdiagnostiserende, selvom man aldrig inspicerer ordbogen:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` er den maskinlæsbare del:

| Nøgle | Hvad den tæller |
| --- | --- |
| `models` | Kameragrupper i kørslen. |
| `images_in_groups` | Kildebilleder på tværs af disse grupper. |
| `targets_found` | Registrerede reflektansmål. |
| `images_calibrated` | Billeder, som kørslen kalibrerede. |
| `exported_files` | **Billedproduktfiler, som kørslen har genereret.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Optagelser fra lyssensorer-sensoroptagelser, der bevidst tælles separat — de stammer fra en anden fase og findes for kørsler uden billeder overhovedet, så hvis man medregnede dem, ville en kørsel, der kun indeholder DAQ-data, se ud som om den havde eksporteret billeder. |

Sammen med dem: `summary["output_dirs"]` (alle mapper, der er skrevet til),
`summary["light_sensor_export"]`, `summary["stopped"]` (gælder, når brugeren afbrød
kørslen, så delvise tællinger ikke tolkes som en afsluttet kørsel, der producerede for lidt), og
`summary["groups"]` (opdelingen pr. gruppe).

`exported_files` registreres af pipelinen **mens den skriver**, ikke scannet fra
projektets billedobjekter bagefter. De parallelle og GPU-strategier opbygger deres egne billedobjekter
(i arbejdsprocesser for GPU-stierne), så den gamle scanning rapporterede
`0 file(s) written` for hver sådan kørsel og udsendte derefter »zero-exports«-hintet — på kørsler,
hvor alt havde fungeret. Hvis du skriver et script baseret på dette tal, rapporterer en velfungerende parallel kørsel nu
et tal, der ikke er nul.

Rapporter om oversprungne lyssensorer angiver den årsag, som læseren faktisk fastslog for hver fil — et
ulæseligt skema, et manglende bundt, en skrivefejl — **dedupliceret**, så tyve filer, der
blev sprunget over af én årsag, vises som én årsag i stedet for tyve gentagelser af den.

> **`process()` udløses ikke, når en kørsel ikke producerer nogen billeder.** Dette er det eneste sted, hvor SDK og
> CLI bevidst adskiller sig: `chloros-cli process` behandler &quot;produkter blev anmodet om, ingen blev
> skrevet&quot; som en fejl og afsluttes med et tal større end nul, mens SDK afsluttes normalt og rapporterer
> tilstanden via `summary` / hints. Hvis din pipeline skal stoppe ved et tomt kørsel, skal du kontrollere det
> selv — undersøg `summary` (eller tæl filerne i projektmappen) i stedet for at stole på
> fraværet af en undtagelse. De sædvanlige årsager er en inputmappe, der ikke blev genkendt som en
> optagelse, og produkter, der blev sprunget over, da de ikke var relevante for de tilstedeværende kameraer (f.eks. stråling fra RGB -only
> kameraer).

### Hjælpefunktioner

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Understøttede værdier

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrisk output (LATTICE multispektral pipeline)

LATTICE multispektral (M3C/M3M) eksportniveau — `reflectance` (standard), `radiance`, `sensor-response` eller `all` (alle relevante tilstande pr. billede) — svarer til projektets **&quot;Radiometrisk output&quot;**. `configure()` har et dedikeret nøgleord til dette:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Den avancerede udvej — at skrive projektets `"Radiometric output"`-nøgle gennem `custom_settings` — fungerer stadig, men husk, at det erstatter hele indstillingsblokken (se advarslen nedenfor):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (standardindstillingen) dividerer kameraets strålingsintensitet med den **tidsstempelmatchede DAQ-nedstråling**, der automatisk udledes fra en optaget `.daq` (DAQ-U/M/E)**eller en DAQ-M-indbygget `.csv`**, der findes sammen med billedmaterialet; eventuelle kamera- eller DAQ-kalibreringspakker, der mangler lokalt,**hentes automatisk fra AWS** ved første brug. CLI viser dette som produktvælgere pr. type på `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **erstatter** hele blokken med beregnede indstillinger (den omgår bevidst `configure()`&#x27;s øvrige nøgleord og validering). Når du bruger den, skal du medtage alle `Project Settings`-nøgler, , du er interesseret i, som i eksemplet ovenfor.

---

## Smart-Connect til LATTICE-kameraer

Vedvarende backend-sessioner til live-hardware. De samme endpoints, som GUI&#x27;en bruger, så funktionen er identisk på SDK / CLI / GUI.

### Enkelt kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### `connect_camera()`-signatur

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession`-metoder

| Metode | Beskrivelse |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Læser GenICam-noder; returnerer `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Skriv noder efter venligt navn (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Optager et **enkelt** billede. Returnerer en liste med ét element bestående af ordbøger med billedmetadata. (Optagelse af serier/flere billeder er fjernet — kald `capture()` i en løkke, hvis du har brug for en serie.) |
| `disconnect()` | Frigør fra puljen. Ingen handling, hvis vi er tilknyttet en allerede åben session. |

`capture()` eksportkontroller (samme model som arrayet + GUI):

- `processing` / `levels` — `processing="all"` gemmer alle relevante eksporttyper; `levels=["raw","radiance"]` gemmer kun disse (overskriver `processing`). Udelad begge for at bruge backend-standardindstillingen.
- `force_daq=True` — gemmer den tildelte DAQ/DLS-aflæsning som en `.daq`-sidecar, selv ved en raw-only-optagelse, så billedet senere kan genbehandles til reflektans/indeks. Ingen handling, hvis der ikke er knyttet nogen DAQ.

### Synkroniseret array — `ArraySession` (Smart-Prep)

`connect_array` er **det anbefalede udgangspunkt** for opsætninger med flere kameraer. Det kører den fulde GUI-baserede smart-prep-proces i baggrunden:

1. **Netværksanalyse** (`/api/camera/array/recommend`) — finder den største billedstørrelse, der passer til sim-emit-niveauet uden at tabe billeder.
2. **Automatisk valg af tier** — `sim-capture-sim-emit`, hvis kablen kan klare det; ellers `sim-capture-ftd-stagger` eller `slip-emit-and-capture`.
3. **Automatisk formindskning**— formindsker ramme størrelsen / øger binning uden advarsel, når ledningen ikke kan opretholde den ønskede opløsning.**Dette sikkerhedsnet dækker ikke samlet overtegnelse**: for mange kameraer til kablet kan ikke løses ved at reducere billedstørrelsen — se [Overtegnelse](#over-subscription-the-per-cam-floor).
4. **PTP aktiveret**som standard — tidsstempler på tværs af kameraer lander på et fælles ur med en nøjagtighed på**~1 ms**. Samtidig eksponering kommer fra M8-hardwareudløseren (**&lt; 100 µs** mellem modulerne), ikke fra PTP: PTP synkroniserer *tidsstempler*, ikke eksponeringer.
5. **Automatisk valg af pixelformat pr. kamera** — RGB-kameraer → `BayerRG8`, multispektral → `BayerRG12`.
6. **AE-seeding** — tager et øjebliksbillede af hvert kameras aktuelle AE-tilstand, så forbindelsen ikke nulstiller eksponeringen undervejs.
7. **GPIO-triggerkonfiguration** — `connect_array` aktiverer alle kameraer (`TriggerMode=On`, `TriggerSource=Line2`), så masterens puls styrer slaverne via M8-kablet. Dette er et: et enkelt kamera, der åbnes med `LatticeCamera`, kører i stedet frit.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()`-signatur

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`force_tier`-værdier:
- `"sim-capture-sim-emit"` — ægte simultan (alle kameraer udløses på samme klokkant).
- `"sim-capture-ftd-stagger"` — fleksibel tidsmæssig forskydning (kamme udsender på let forskudte tidspunkter, så pakkerne serieliseres på ledningen).
- `"slip-emit-and-capture"` — sekventiel indfangning pr. kam (ingen tidsmæssig synkronisering; eneste mulighed, når ingen rammestørrelse passer til simultan).

`wire_ceiling_mbps` tilsidesætter **værtsens vedvarende ledningsbudget** i MB/s — det eneste
tal, som hele array-allokeringen afhænger af. Lad det stå på `None` for at bruge den automatisk-detekterede
værdi. Sænk den, når arrayet rapporterer GVSP-korrupte rammer: den automatiske værdi er afledt
af NIC’ens annoncerede linkhastighed, som overvurderer USB-adaptere, smalle PCIe-baner og
travle delte fabric-netværk — og denne overvurdering viser sig som korrupte rammer snarere end som en
synligt langsom forbindelse. Værdien gemmes i projektets array-optagelsesblok, så en
genåbning eller en senere `connect_array` gendanner den ligesom enhver anden array-indstilling.
Se [Array-tilstand](#array-health--which-subsystem-is-losing-frames).

#### Over-Subscription (minimumsgrænsen pr. kamera)

Sim-emit-pacing tildeler hvert kamera en andel af det kollisionssikre ledningsbudget, med en minimumsgrænse på **8 MB/s pr. kamera**(`per_cam_floor_bps`). Når `N × floor` overskrider det kollisionssikre loft,**overabonnerer arrayet på ledningen**— fejltilstanden er tab af GVSP-pakker, ikke en lavere billedhastighed — og der findes ingen afhjælpning via billedstørrelse:**binning og ROI reducerer antallet af bytes pr. billede, ikke de tildelte bytes pr. sekund**som den samlede kontrol sammenligner. Praktiske lofter for fuld opløsning på en 1 GbE-vært:**6 kameraer ved 1500 MTU, 9 med jumbo-rammer** (`max_cams_collision_safe` i analysesvaret angiver grænsen for din forbindelse). Løsninger: færre kameraer, jumbo-rammer fra ende til ende eller et hurtigere netværkskort.

- Svarene `analyze_array_network()` og `/api/camera/array/connect` indeholder `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` og `per_cam_floor_bps`. Når `oversubscribed` er sand, nulstiller projektionen **nulstiller fps-felterne** (`achievable_fps_max` / `fps_bright` / `fps_dark`) i stedet for at rapportere en vildledende hastighed, der er langsom, men fungerer.
- `POST /api/camera/array/connect` accepterer en `pin_resolution`-body-parameter (**kun HTTP — ikke en SDK-kwarg**; `connect_array` eksponerer den ikke). Fastlåsning fjerner sikkerhedsnetværket ved binning-walk-down, så en overtegnet forbindelse med `pin_resolution` indstillet**afvises-afvist** med en fejlmeddelelse, der angiver alle mulige løsninger. Uden fastlåsning fortsætter forbindelsen med nedskaleringen-down, men advarer om, at nedskalering ikke kan rydde aggregatet.
- Undtagelse til testmiljø: Indstil `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` i backendens miljø for at nedgradere afvisningen til en tydelig advarsel — du opretter forbindelse alligevel og accepterer pakketabet.

#### Array-tilstand — hvilket undersystem mister rammer

`GET /api/camera/array/<array_id>/capability` bærer en aktiv `health`-blok på et
tilsluttet array, revurderet i et rullende **10-sekunders** vindue. Den opdeler rammetab
i de to årsager, der kræver modsatrettede løsninger, i stedet for én &quot;ufuldstændig&quot; rate, der
ikke angiver nogen af dem:

| Felt | Hvad det betyder | Hvilket undersystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (pr. serienummer) | Framen **ankom og var strukturelt defekt**— GVSP-pakketab. |**Netværk**: ledningskapacitet, pacing, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (pr. serienummer) | Rammen **kom slet ikke**— kameraet udløste ikke, eller der kom intet ud af det. |**Udløser / synkronisering**: M8-kabel, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Det kamera med den dårligste hastighed for hver. | — |
| `per_cam_rate_pct` | Samlet ufuldstændighedsprocent pr. kamera (begge årsager tilsammen). | — |
| `stable_for_seconds` | Hvor længe hvert kamera har ligget under 0,01 %. | — |

Sammen med `health` angiver den samme post det tal, som hele tildelingen afhænger af:

| Felt | Betydning |
| --- | --- |
| `wire_ceiling_mbps` | Værtens gældende vedvarende båndbredde, MB/s. |
| `wire_ceiling_source` | Hvor tallet stammer fra, beskrevet med ord — f.eks. `USB-capped 200 MB/s (was theoretical 1062; …)` eller `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, når `wire_ceiling_mbps=` indstillede det. |
| `nic_is_usb` | `true` til en USB-Ethernet-adapter. |

Der findes ingen SDK-wrapper til dette endpoint — læs det direkte:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Aflesning:** `gvsp_corrupt_rate_pct` forskellig fra nul med `never_arrived_rate_pct` på 0 betyder, at
at trigger og kabelsynkronisering er perfekte, og at 100 % af tabet ligger på netværksstien — sænk
`wire_ceiling_mbps` og opret forbindelse igen. Det omvendte mønster peger i stedet på synkroniseringskablet eller
triggerledningen.

> **`target_fps` er ikke afgørende for korrupte rammer.** GevSCPD-pacing skrives én gang ved
> opkobling, så en sænkning af triggerfrekvensen ændrer duty cycle og ikke
> burst-frekvensen for samtidig udsendelse. En målt 5×-efterspørgselsbegrænsning gav ingen forbedring, mens
> en sænkning af ledningens loft fra 240 til 200 MB/s reducerede fejlprocenten for det samme udstyr fra 10,4 % til
> 0,00 %.

> **Automatisk nedskalering midt i strømmen er ikke tilgængelig på TRI032S-firmwaren.** Et kørende array kan ikke
> løse dette problem selv; afbryd forbindelsen og tilslut den igen, så forbindelsestidsvælgeren planlægger på ny i forhold til
> den nye grænse.

En **USB-Ethernet-adapter er begrænset til 200 MB/s** af sonden uanset dens
mærkeskilt: effektivitetsskemaet, der omregner en linkhastighed til et vedvarende tal, er
afledt af PCIe, og et USB-netværkskort angiver sin Ethernet-forbindelseshastighed, mens det er begrænset af
USB-bussen og dens driver. Begrænsningen er absolut, ikke en brøkdel — et USB 1 GbE-kort
opnår ~80 MB/s og påvirkes ikke.

#### `ArraySession`-metoder

| Metode | Beskrivelse |
| --- | --- |
| `status(timeout=10.0)` | Live `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Én synkroniseret indfangningsgruppe. Returnerer en `CaptureResult` (liste over frame-dictionaries + `.skipped`). Eksportkontroller nedenfor. |
| `capture(..., smart=True)` | **Smart optagelse** — venter på, at AE er stabiliseret på alle kameraer, og udløser derefter. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Hurtigst mulig optagelse: kun rådata + den tildelte DAQ-måling (+ det frie kombinerede indeks). Afspejler GUI-knappen »Hurtigst mulig optagelse«. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Enkelt / Kontinuerlig / Interval i én afgrænset løkke. Returnerer `list[CaptureResult]`.**Kræver `count` og/eller `duration_s`** for at afslutte (SDKen har ingen Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Start optagelse af den kombinerede live-indeksvisning til video/GIF → `RecorderHandle`. Én sammensat optager pr. array. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Start en høj-fps rå-Bayer-burst → `RecorderHandle`. Genbehandle offline med `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Genbehandle en gemt rå-burst offline til kalibreret video(er). Blokerer, indtil det er færdigt (`wait=True`) og returnerer `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Afventer svar fra et offline-build-job: `{running, result, error, burst_dir}`. |
| `disconnect()` | Frigiv hele arrayet. |

`capture()` eksportkontrol (samme endpoint som GUI&#x27;en/CLIen bruger):

- `processing` / `levels` — `processing="all"` (eller `levels=["raw","radiance",…]`) gemmer hver relevant eksporttype pr. kamera; en enkelt `processing`-værdi gemmer kun det pågældende niveau.
- `aligned=True` — forvrænger hvert medlems ikke-rå eksport til arrayets [justeringsprofil](#array-alignment) (co-registreret); rådata forbliver uforvrænget, men bærer transformationen i metadata. Faldt tilbage til ujusteret (med en advarsel, der vises i resultatets `alignment`), hvis arrayet ikke har nogen profil.
- `render_index=False` — springer vegetationsindeksoverlejringen pr. kamera-indeks-overlay; standardindstillingen gengiver det, hvor det er konfigureret.
- `force_daq=True` — gem den tildelte DAQ/DLS-aflæsning som en `.daq`-sidecar, selv når intet valgt niveau har brug for den.

**TIFF-komprimering (HTTP -only-knap):**`ArraySession.capture()` sender ingen `compression`-nøgle, så backendens standardindstilling gælder — `POST /api/camera/array/capture` læser en `compression`-body-parameter, som standard `"deflate"` (tabsfri zlib L1 + vandret prediktor, ~4,1 MB pr. billede i fuld opløsning). `"none"` skriver ukomprimeret (~6,3 MB/billede) med en**~5 gange hurtigere skrivning** — begge er tabsfrie og læses identisk ved import. SDK&#x27;en har ingen kwarg til dette; løsningen er `chloros-cli lattice array-capture --compression none` eller rå HTTP. DEFLATE holder også GIL&#x27;en Python, så komprimerede skrivninger kan ikke paralleliseres på tværs af skrivetrådene pr. kamera — vedvarende optagelse i fuld opløsning med 8 kameraer ved sensorhastighed kræver `compression: "none"`. Detaljer: [CLI Reference → array-capture](cli-reference.md).**Overridelser af eksport pr. medlem (kun HTTP):**det samme endpoint accepterer også `exclude_serials` (liste — fjern medlemmer fra det gemte sæt; arrayet udløses stadig som én synkroniseret gruppe, og de ekskluderede medlemmer returneres i `excluded`), `serial_levels` (`{serial: [level tokens]}`-overstyringer pr. kamera-niveau) og `serial_index` (`{serial: bool}`-overstyringer af indeks-overlay pr. kamera). Dette er GUI-paritet-kropsparametre og**endnu ikke *SDK-kwargs**; medlemmer, der mangler i kortene, falder tilbage til de array-dækkende `levels` / `render_index`.

##### Gennemgang af oversprungne cams — `CaptureResult.skipped`

`ArraySession.capture()` returnerer en `CaptureResult`, som er en underklasse af `list`: iterer den, indekserer den, `len()` den — alle eksisterende mønstre fungerer fortsat. Ny kode kan undersøge `.skipped`-attributten for at se, hvilke kameraer der blev udelukket, og hvorfor. Det mest almindelige tilfælde er RGB kameraer i en blandet-filterarray, når du anmoder om `processing="radiance"` eller `"reflectance"` — strålingsintensitet pr. Bayer er meningsløs for en bredbåndssensor, så backend&#x27;en springer disse kameraer over i stedet for at generere meningsløse resultater.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Årsagstokener følger mønsteret `<level>-not-applicable-to-rgb-cam` (én post pr. oversprunget niveau, hvor hver bærer `level`). De reflektansspecifikke overspring er `reflectance-skipped-no-fresh-dls` (ingen ny nedadrettet måling tilgængelig), `reflectance-skipped-bound-daq-unavailable (…)` (den tilknyttede DAQ kunne ikke nås) og `dls-uncalibrated-band-<nm>` — båndet ligger for det meste uden for DAQ-lyssensorens radiometrisk kalibrerede område (~374–974 nm), så den absolutte DAQ-baserede reflektansopdeling afvises, og billedet nedgraderes tydeligt til sensorrespons. Blandt de leverede SKU’er er det kun F988, der udløser dette; det pågældende kameras understøttede arbejdsgang er arbejdsgangen med reflektanspanelet.

`processing`-niveauer:

| Niveau | Output |
| --- | --- |
| `"raw"` | Enkeltkanals Bayer (monokameraer: enkeltbåndet) direkte fra sensoren. |
| `"debayered"` *(standard for SDK)* | 3-kanals BGR via bilineær demosaik (monokromkameraer: 1-kanals gråtoner). |
| `"radiance"` | float32 W/m²/sr/nm via den fulde radiometriske kæde. Kun multispektral — RGB-kameraer springes over. |
| `"reflectance"` | uint16 0..32768 (Pix4D-klar); kræver en live DAQ-parring for absolut reference. Kun multispektral. |
| `"display"` | Fuld kæde, der matcher GUI-forhåndsvisningen (CCM + WB + gamma i henhold til kameraets profil). |
| `"all"` | **Én fil pr. relevant niveau** for hvert kamera (svarende til GUI-indstillingen »Capture All&quot; / CLI-standardindstillingen). Den returnerede `CaptureResult` indeholder derefter én billedramme-dict pr. `(cam, level)`, med niveauet i hver dict; niveauer, der ikke er relevante, vises i `.skipped`. Den DAQ-måling, der bruges til enhver reflektansramme, gemmes som en `.daq`-sidecar. |

> **Bemærk — standardindstillingen adskiller sig fra CLI.** `ArraySession.capture()` er som standard indstillet til `processing="debayered"`; kommandoen `chloros-cli lattice array-capture` er som standard indstillet til `processing="all"`. Angiv `processing="all"` eksplicit fra SDK for at afspejle CLI /GUI-gemning på flere niveauer.

### Optagelsestilstande og optagere

Arrayets overflade afspejler GUI-optagelsespanelet: Optagelsestilstande (Enkelt / Kontinuerlig / Interval / Hurtigst mulig lukkertid) samt to optagere (live-kompositvideo og rå burst → offline-genbehandling).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**er SDKs kontinuerlige/interval-loop. Da der ikke findes en `Ctrl+C` til at afbryde den fra et script,**skal** du overføre `count` og/eller `duration_s` (den stopper , når en af dem nås). `interval_s` måles fra starten af hvert gennemløb (svarende til GUI&#x27;en). De resterende kwargs sendes direkte videre til `capture()`.
- **`record`** er *overvågningskvalitet*: den indfanger den kombinerede indeks-komposit i realtid, som den vises, så den kombinerede strøm skal være åben, for at billederne kan lande. Én kompositoptager pr. array (udløser en fejl, hvis der allerede kører en).
- **`burst` → `build_video`** er *analyse-klasse*: `burst` skriver rå billeder + et manifest pr. billede + én `.daq` pr. unik DLS-aflæsning under `<output>/bursts/<base>/` med grab-loopens fulde hastighed (ingen kæde, ingen exiftool, ingen live-visning). `build_video` tidsmatcher hvert billede til det nærmeste `.daq` og kører importpipelinens radiance/refleksionsgrad/indeks-kæde. `products` er en liste over `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (standard: det kombinerede indeks). `burst().stop()` starter også automatisk en best-effort-opbygning af det kombinerede-indeksopbygning, der returneres som `build_job` i stopresultatet.

#### `RecorderHandle`

Returneres af `ArraySession.record()` og `ArraySession.burst()`. Brug den som en kontekstmanager til automatisk at stoppe ved afslutning af omfanget, eller styr den manuelt.

| Medlem | Beskrivelse |
| --- | --- |
| `job_id` | Backend-job-id (str). |
| `kind` | `"composite"` (fra `record`) eller `"raw"` (fra `burst`). |
| `start_stats` | Den ordbog, der returneres af `start`-kaldet. |
| `result` | `None` under kørsel; den endelige stop-resultat-dikt, når kørslen er stoppet. |
| `stats(timeout=10.0)` | Live jobstatistik (skrevne frames, realiseret fps, forløbet tid). |
| `stop(timeout=60.0)` | Stopper optageren; returnerer og cachelagrer det endelige resultat. Idempotent (et andet kald returnerer det cachelagrede resultat). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Tilslutning til et allerede tilsluttet array — `attach_array`

Hvis arrayet allerede er oprettet (GUI’en har åbnet det, eller en tidligere SDK-session har kaldt `connect_array`), skal du bruge `attach_array` til at hente et håndtag til det i stedet for at oprette forbindelse igen. `connect_array` giver altid fejlen &quot;Kameraet  er<sn> allerede i arrayet <id>&quot; i den situation, fordi POST-anmodningen til `/array/connect` for et medlem i puljen ikke er idempotent; `attach_array` læser `/api/camera/array/list` og matcher enten efter array_id eller serienumre.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Mønster: SDK-scripts, der kører sammen med desktop-GUI’en, bør først prøve `attach_array` og falde tilbage til `connect_array`, hvis der endnu ikke er noget array i puljen.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Vigtigt — afslutning af context-manager afbryder forbindelsen.**`ArraySession.disconnect()` sender altid en POST-anmodning til `/array/disconnect`; der er ingen tilknyttet-not-owned-guard, som der er for `CameraSession` / `DAQSensorSession`. Hvis du deler med GUI&#x27;en og ikke ønsker at nedlægge arrayet ved scope-afslutning,**skal du ikke bruge `with`-blokken** — opbevar håndtaget i en almindelig variabel og spring det eksplicitte `disconnect()` over:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Hjælp til netværksanalyse

Nyttigt inden du åbner arrayet — forudsiger, om dine foreslåede indstillinger vil passe:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` er en af følgende: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (ellers `error`). `auto_capped_fps` betyder, at den anmodede opløsning kun passer til RX-ringen ved en begrænset triggerhastighed — behold opløsningen og send `target_fps=result["recommended"]["recommended_target_fps"]` videre til `connect_array` (se [Eksempel 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Sådan aflæses projektionen** (samme model som GUI-panelet »Array Settings«):

- **Burst (`frame_bytes_total`) summeres pr. kamera i hvert kameras reelle pixelformat.**Mono**M3M**-kameraer streamer Mono12 (2 B/px) uanset hvilken `pixel_format`-værdi du angiver, så et billede i fuld opløsning fra 4 kameraer er**~25 MB** med tre mono-kameraer, ikke de ~12,6 MB, som en antagelse om, at alle er 8-bit, giver. Backend&#x27;en afgør hvert kameras format ud fra dets model.
- **Admittance (`burst_fits_nic_ring`) er drain-bevidst**, ikke hel-burst-vs-ring: sim-emit passer, når værten tømmer RX-ringen hurtigere, end kameraerne fylder den. En 10G-vært + 1 GbE-kameraer**tillader** fuld opløsning, selv når burstet overstiger ringen; en 1 GbE-vært blokerer (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` er et konservativt loft for seriel hentning** — `max(readout+emit, N×emit)` med udsendelse pr. kamera begrænset til 1 GbE-kameralinket, uafhængigt af eksponering. E.f.eks. ~2,8 fps for et 4-kamera-array i fuld opløsning med 12 bit (svarer til runtime-målingen på ~2,7–3,0). Fuld model: [CLI Reference → Array fps &amp; burst-model](cli-reference.md#array-fps--burst-model).
- **Over-subscription (`oversubscribed: true`) betyder, at N × minimumsgrænsen pr. kamera overskrider den kollisionssikre maksimumsgrænse** — fps-felterne (`achievable_fps_max` / `fps_bright` / `fps_dark`) viser 0, og automatisk komprimering/binning kan ikke løse problemet (disse metoder reducerer antallet af bytes pr. frame, ikke antallet af bytes pr. sekund). Løsningerne er færre kameraer, jumbo-frames eller et hurtigere netværkskort; `max_cams_collision_safe` angiver loftet (6 kameraer i fuld opløsning på 1 GbE ved 1500 MTU, 9 med jumbo). Svaret indeholder også `aggregate_demand_bps`, `collision_safe_ceiling_bps` og `per_cam_floor_bps` (8 MB/s). Se [Over-Subscription](#over-subscription-the-per-cam-floor).

### Opdagelse og visning

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

LATTICE-arrayer kører kontinuerlig AE i baggrunden, så snart de er tilsluttet, men det tager et øjeblik for en nyindstillet scene at konvergere. **Smart-capture** er den praktiske løsning: den aflæser eksponeringen for hvert kamera, venter, indtil arrayet er stabilt på tværs af et vindue, og udløser derefter optagelsen. Det svarer til GUI&#x27;en: desktop-appens &quot;smart&quot;-optagelsesknap kalder det samme backend-endpoint.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Når du styrer via `ChlorosProject` (næste afsnit), får du flere indstillingsmuligheder:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Smart-AE-politikken er som standard konservativ. Stram `exposure_tolerance_pct` for præcistradiometrisk arbejde; lemp den for hurtigt skiftende scener, hvor du blot ønsker &quot;tæt nok&quot;.

---

## DAQ-sensorsessioner

Vedvarende backend-pool til spektralsensorer (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet). Afspejler kameraets overflade: smart-detect, genbrug af pool, idempotent tilkobling.

### Smart-Detect (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prioritet: Ethernet → BLE → USB. Angiv et hvilket som helst eksplicit tip for at fastlåse transporten.

### Fastlagt transport

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### `DAQSensorSession`-metoder

| Metode | Beskrivelse |
| --- | --- |
| `status(timeout=10.0)` | Oversigt over pool-post (streaming-/optagelsestilstand, bølgelængdeområde, kalibrerings-sha, integrationstid, frame_avg, AE-tilstand). |
| `latest(n=1, timeout=10.0)` | Returnerer op til N seneste spektrumrammer. |
| `stream_start()` / `stream_stop()` | Genoptag / sæt streaming på pause (håndtaget forbliver åbent). |
| `record_start(output_dir=None, device_name=None)` | Start optagelse af en .daq-fil. Returnerer filstien. Afvises for DAQ-U/M uden et AWS-kalibreringsbundt (DAQ-E er undtaget). |
| `record_stop()` | Stop optagelse. Returnerer `{path, rows}`. |
| `disconnect()` | Frigør fra puljen. Ingen effekt for tilknyttede, men ikke-ejede håndtag. |

> **Cap-korrektionsprofiler (`cap_id`) er ikke en SDK-knap.** `connect_daq_sensor()` / `DAQSensorSession` eksponerer ingen `cap_id`-parameter eller `set_cap`-metode. Vælg en flådekapacitetskorrektionsprofil via CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) eller backendens `/api/daq`-HTTP-ruter (`/api/daq/connect` og `/api/daq/<id>/cap-id` accepterer `cap_id`).

### Opdagelse — at finde en adresse at oprette forbindelse til

`discover_daq_sensors()` scanner USB / BLE / ETH efter sensorer, du *kunne* åbne. Det er DAQ-modstykket til `discover_lattice_cameras()` og den eneste måde at få fat i en **DAQ-M&#x27;s BLE-MAC** — en DAQ-E har et værtsnavn og en DAQ-U en COM-port, men en MAC er hverken trykt på enheden eller angivet af operativsystemet.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Felt | Beskrivelse |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-port / BLE MAC / værtsnavn — videregives til `connect_daq_sensor` som `port=` / `mac=` / `eth_host=`. |
| `display` | Menneskeligt læsbar etiket. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`, eller `None` for en port, som scanningen ikke kan identificere (USB-serielle adaptere kan ikke skelnes uden en probe, så ukendte elementer vises frem i stedet for at blive skjult). |
| `extra` | Detaljer pr. transport (BLE-annonceret navn, USB-producent, DAQ-E ip/fw/…). Tomme værdier udelades. |

| Parameter | Standard | Beskrivelse |
| --- | --- | --- |
| `transports` | alle tre | Sekvens (eller CSV-streng), der begrænser scanningen. Det er en god idé at angive denne, når du ved, hvad du vil have — BLE er den langsomste del. |
| `scan_timeout` | 5 | Scanningsvindue pr. transport i sekunder; backend begrænser til 1–20. |
| `timeout` | 60,0 | HTTP-loft for hele opkaldet (som andre steder i SDK). |
| `auto_start_backend` | `True` | Start en lokal backend, hvis der ikke kører nogen. Startes aldrig for en fjernbetjent `backend_url`. |

> **Sensorer, der allerede er åbne i puljen, vises ikke.** En tilsluttet BLE-periferienhed holder op med at annoncere, og en åben COM-port kan ikke afprøves, så opdagelsen viser, hvad der er *tilgængeligt for tilslutning*. Et tomt resultat lige efter, at du har tilsluttet noget, er forventet — brug `list_daq_sensors()` til det, du allerede har. Transportprotokoller, hvor scanningen ikke kan køre (ingen bleak / zeroconf installeret), springes over i stedet for at udløse en fejl, så en maskine uden Bluetooth stadig får sine USB- og ETH-svar.

### Liste

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Samkørsel med GUI / CLI

Hvis GUI&#x27;en allerede har en sensor åben, returnerer et kald til `connect_daq_sensor(port="COM3")` fra Python et håndtag mærket `already_connected=True`. Sessionens `disconnect()` er derefter en no-op, så dit SDK-script ikke river sensoren væk under GUI&#x27;en, når scope afslutning.

### Direkte hardwareklasser (ingen backend)

`daq_sdk` reeksporteres af `chloros_sdk`, så du også kan styre sensorerne fra ende til ende i processen uden backend:

> **Tilgængelighed:**`daq_sdk` leveres med Chloros-desktopinstallationen,**ikke** med PyPI-pakken — `pip install chloros-sdk` giver dig `lattice_sdk`, men efterlader `chloros_sdk.DAQ_AVAILABLE == False`. Tjek dette flag, før du bruger disse klasser; på en host-harddisk, der kun kører pip, skal du i stedet køre sensoren via [`connect_daq_sensor()`](#daq-sensor-sessions), som ikke kræver lokale transportbiblioteker.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Foretræk smart-connect-stien (`connect_daq_sensor`), når du ønsker delt ejerskab med GUI&#x27;en; brug de direkte klasser til headless-scripts, der har eksklusivt ejerskab af sensoren.

---

## Projektautomatisering — `ChlorosProject`

Et gemt Chloros-projekt er en mappe, der indeholder `cameras.json` + `sensors.json` + `project.json`. `open_project` indlæser manifestet, og `connect_all` bringer alle gemte enheder online med deres gemte indstillinger — samme hardwaretilstand, som GUI&#x27;en ville skabe.

### Minimalt eksempel

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Eller som en kontekstmanager:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject`-metoder

| Metode | Beskrivelse |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Find og opret forbindelse til alle gemte enheder. Returnerer en forbindelsesrapport pr. klasse. Bruger en kørende backend, når en sådan lytter på `127.0.0.1:5000`; ellers falder den lydløst tilbage til direkte (backend-fri) `lattice_sdk`-enhedskontrol — den opretter aldrig et backend. |
| `disconnect_all()` | Afbryder alt. |
| `capture_all(output_dir=".")` | Ét billede fra hvert kamera + array + spektrum fra hver sensor. |
| `stream(camera, overlays=False, fps=10.0)` | Generator, der genererer BGR-billeder fra et navngivet kamera (eller array). `overlays=False` er en direkte `lattice_sdk`-indlæsningsloop (arrays genererer `{serial: frame}`-dicts). `overlays=True` dirigeres gennem `ChlorosLocal.camera_stream()` → backendens `/api/camera/<serial>/stream-annotated` MJPEG-feed, hvor kameraetsgemte `ui.overlay`-blok videregives som forespørgselsparametre. Kræver backend-tilstand og et **selvstændigt kamera**: et kamera i direkte tilstand udløser `RuntimeError` (backend&#x27;en kan ikke hente et kamera, som denne proces ejer), og et array udløser `NotImplementedError` (overlejrer sammensat billede pr. kamera — streamer et medlem efter navn). Tilsvarende engangsudførelse: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Kør justering på alle aktuelt tilsluttede arrays. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Kør kalibrerings-/ indekspipeline på projektets billeder (indkapsler `ChlorosLocal.process`; disse fire er de **eneste** accepterede kwargs — `indices=` osv. udløser `TypeError`; indekser indstilles via `ChlorosLocal.configure()`). Konstruerer en `ChlorosLocal()`, som automatisk starter en backend. |

Attributter:
- `proj.cameras` — `Dict[str, CameraHandle]` indekseret efter navn OG serienummer.
- `proj.arrays` — `Dict[str, ArrayHandle]` indekseret efter navn OG array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` indekseret efter navn OG slot_id.
- `proj.config` — `project.json["config"]` ordbog.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Behandlingsniveauer.** `capture()`, `grab()`, og `frame_stream()` bruger alle det samme `processing`
token, og kæden er kumulativ — hvert niveau kører alt, der ligger over det:

| Niveau | Output | Bemærkninger |
| --- | --- | --- |
| `raw` | 1-kanals Bayer, sensor-native | Ingen demosaik. Overlejringer er ikke tilgængelige på dette niveau. |
| `debayered` | 3-kanals BGR (**standard**) | Bilineær demosaik. Det eneste niveau, der fungerer uden backend-tilstand. |
| `radiance` | float32, W/m²/sr/nm | Fuld radiometrisk kæde: demosaikering + 3×3-separering (multispektral) + DSNU + flat-field + NIST-skala, hvor eksponering × forstærkning er divideret ud, så værdierne er absolutte. |
| `reflectance` | uint16, 32768 = 1,0 | Strålingsintensitet divideret med nedadgående bestrålingsintensitet (ρ = π·L/E). Kræver en DLS/DAQ-aflæsning — se bemærkningen nedenfor. |
| `display` | 8-bit sRGB-lignende | GUI-ækvivalent gengivelse: CCM + hvidbalance + gamma via kameraets aktive farveprofil. |

Alt andet end `debayered` kræver backend-tilstand; et kamera i direkte tilstand genererer
`NotImplementedError`. `reflectance` kræver en brugbar downwelling-måling — frame-endepunktet trækker
den samlede DAQ automatisk ind i kameraets DLS-slot, men uden en tilknyttet DAQ afviser kæden
reflektansudgangen og markerer ærligt nedgraderingen i de returnerede metadata i stedet for lydløst
at returnere et ringere produkt.

> **Reflektans DN-skala — hardkod den ikke.** LATTICE-reflektans bruger `32768` = ρ 1,0 og angiver
> XMP `Chloros:PixelScale=32768`; Survey3 reflektans bruger `65535` = ρ 1,0 og indeholder ingen
> `Chloros:*`-tags. Læs mærket og divider med det. Det er defineret i uint16-domænet, så det forbliver
> `32768` for hvert format, der skaleres om (16-bit TIFF, 8-bit PNG /JPG, 32-bit procent) — normaliser
> først den gemte datatype tilbage til uint16 (×257 fra 8-bit, ×65535 fra float). Den eneste undtagelse:
> en 8-bit-kildeoptagelse skrevet som 8-bit TIFF bliver *klippet*, ikke omskaleret, så der er ingen skala, der beskriver
> den — Chloros udelader `PixelScale` og MicaSense-tuplen helt i dette tilfælde. Behandl et manglende
> tag i en LATTICE-reflektansfil som »ingen gyldig skala«, ikke som en standardindstilling.

> **EXIF overføres til eksporten.** `process()` kopierer GPS-blokken fra kildeoptagelsen
> **og dens ExifIFD** til hvert produkt, så eksportfilerne indeholder `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` og `CameraSerialNumber` samt
> georeferencen. `FocalLength` er det, Pix4D bruger til at beregne ground sample distance ud fra — uden det
> falder rekonstruktionen tilbage til en helt forkert skala (i et målt tilfælde blev et område på 411 m
> til et på 47,8 km). Kopien er bevidst ikke `-all:all`: IFD0’s strukturelle tags ødelægger
> LATTICE-outputtet, og `ExifImageWidth`/`Height` er udelukket, fordi de beskriver kildeoptagelsen
> optagelse snarere end det eksporterede raster.

Underflag for optagelsesfasen (gælder for de radiometriske niveauer — `radiance`, `reflectance`, `display`):

| Flag | Standard | Betydning |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + 3x3 unmix + NIST-radiometrisk skala. |
| `apply_white_balance` | `True` | WB LUT. DLS-bevidst, når en DAQ er knyttet til kameraet. |
| `apply_index` | `False` | Evaluering af vegetationsindeks. |
| `index_expression` | `None` | Overskrivningsformel. Ikke-tom → aktiverer automatisk indeks. |
| `annotated` | `False` | Overlejring af GUI-dekorationer (zebra/gitter/peaking). Ikke tilgængeligt for `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Returtypen er `CapturePathMap`, ikke `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` er `Dict[str, Union[str, List[str]]]`: en enkelt-niveau
> `processing` tildeler hvert serienummer én sti, mens en flerniveauliste (`"all"` eller en
> eksplicit `levels`-liste) giver den den **ordnede liste** over alle produkter, der er gemt for det pågældende
> kamera. En live-sammensat komposit, hvis en sådan blev streamet, ankommer under den ekstra
> `"combined"`-nøgle i stedet for under en serienummer. Kode, der forudsætter `str`, bryder sammen i
> listeformen uden at nogen typekontrol protesterer — annotationen angav `Dict[str, str]`
> i et stykke tid efter, at listeformen blev udgivet, hvilket er grunden til, at aliaset eksisterer. Normaliser
> når du ønsker den flade form:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Array-justering

`ArrayHandle` eksponerer hele justeringsfladen. Profiler er som standard kun gældende for den aktuelle session — kald `export_alignment()` eksplicit for at gemme dem.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Justering ved opkobling

`connect_all(align=...)` kan automatisk justere alle arrays ved opkobling:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Bruger `project.json["config"]["auto_align_on_connect"]` som standard, hvis ikke andet er angivet.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Direkte hardware (uden backend)

Når du ønsker nul afhængighed af backend (CI, headless-robotter, indlejret), skal du importere `lattice_sdk` og `daq_sdk` direkte — begge reeksporteres af `chloros_sdk`. Bemærk vedrørende `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` findes i PyPI-pakken (men kræver, at Arena-SDK-runtime er installeret), mens `daq_sdk` kun leveres med desktop-installationen.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Forudindstillinger og udløseren

Tre af de fire forudindstillinger kører i **free-run**-tilstand: kameraet eksponerer kontinuerligt, og et
`capture()` returnerer det næste billede. `triggered` er undtagelsen — den aktiverer
kameraet til at reagere på en hardware-flanke på linje 2, så det ikke tager noget billede, før der kommer en.

| Forudindstilling | Udløser | Anvendes når |
| --- | --- | --- |
| `default` | fri kørsel | generel brug |
| `high_speed` | fri kørsel | 8-bit, 60 fps-begrænsning, kort eksponering |
| `high_quality` | free-run | 12-bit, ingen fps-begrænsning — det sædvanlige valg til stillbilleder |
| `triggered` | **klar, Linje 2** | kameraet er tilsluttet via et M8-synkroniseringskabel, og noget andet udløser det |

Hvis du vælger `triggered` (eller selv indstiller `trigger_mode="On"`) uden at der er noget,
der styrer Linje 2, vil alle `capture()` gå i timeout — hvilket er korrekt, da du bad
kameraet om at vente. SDK forklarer dette, når det sker; se
[SC_ERR_TIMEOUT under optagelse](#direct-hardware-backend-free).

> **Bemærk — &quot;GVSP probe&quot; / `SC_ERR_TIMEOUT -1011`-meddelelser ved opkobling er ikke fejl.**&gt; Ved opkobling forsøger SDK at forhandle sig frem til**jumbo-rammer** (9000-byte GVSP-pakker) for at opnå højere gennemstrømning. På en direkte punkt-til-punkt-NIC-forbindelse (f.eks. en link-lokal `169.254.x.x`-adresse) kan netværket normalt ikke håndtere jumbo-rammer, så denne sondering udløber, og der logges linjer som f.eks.:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Dette er den **indbyggede fallback-løsning**: SDK skifter automatisk tilbage til standardpakker på 1500 byte, og kameraet fortsætter med at oprette forbindelse som normalt (de efterfølgende `[chunk-enable …]`-linjer er en del af den normale opkoblingssekvens). Optagelse fungerer stadig.
>
> Du kan springe denne test over, men **den er ikke blot en log-dæmper — den deaktiverer jumbo-rammer.** Kameraet svarer kun på Don&#x27;t-Fragment-pings op til 1500 byte, uanset hvor godt dit netværk er, så ping-testen alene kan aldrig finde jumbo-rammer; denne probe er det eneste, der kan. Deaktiver den, og kameraet kører standardpakker på 1500 byte for altid, på ethvert netværk:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Det kan kun betale sig på et netværk, hvor du *ved*, at det ikke kan håndtere jumbo-pakker, hvor det sparer cirka et sekund i opkoblingstid pr. kamera. Da det er en reel kompromis snarere end en kosmetisk ændring, angiver &quot;SDK&quot; nu dette, når du bruger den:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Lad den være, medmindre du har en grund til at ændre den.** Hvis den forbliver aktiveret, måler hver forbindelse det netværk, du rent faktisk har: Tilslut til en jumbo--kompatibel switch, og den næste forbindelse opfanger jumbo automatisk, uden at du skal konfigurere noget eller genstarte.
>
> Hvis du *ønsker* jumbo-gennemstrømning, skal du aktivere jumbo fra ende til ende (NIC MTU 9000 + en switch, der videregiver dem), eller fastsæt det med `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, når du ved, at forbindelsen understøtter det — men foretræk en kommandobestemt indstilling med `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` frem for en permanent indstilling, da en fast størrelse springer sonderingen over og forhindrer tilpasning til det forudgående netværk. **Hver** enhed i stien skal kunne videregive jumbo-pakker — herunder enhver PoE-splitter eller -injektor, hvilket er den almindelige årsag til, at en ellers jumbo-kompatibel opsætning ikke kan håndtere dem.

> **`SC_ERR_TIMEOUT -1011` under `capture()` / `grab*()` er et andet problem — den er en reel fejl.**&gt; Ovenstående bemærkning vedrører kun `-1011`, der er logget af**connect-time-proben**. Den samme fejl, der opstår ved en**capture**, betyder, at kameraet er tilsluttet korrekt, men ikke sender nogen billeder:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Det afslørende tegn er et kamera, hvis *kontrol*-kanal fungerer korrekt — opdagelse fungerer, indstillinger og `[chunk-enable …]`-skrivninger lykkes alle — mens *hver eneste* frame går i timeout.
>
> **Den sædvanlige årsag er, at kameraet er aktiveret til en hardware-udløser.** Med `trigger_mode="On"` og `trigger_source="Line2"` sender kameraet slet intet, før der kommer en elektrisk flanke på M8-synkroniseringskablet. Hvis der ikke er noget kabel, der driver denne linje, venter hver eneste billedoptagelse i evigheder. Kameraet er ikke i stykker, og netværket fungerer fint — det gør præcis, hvad det er blevet bedt om.
>
> `CameraSettings()` og `default` / `high_speed` / `high_quality` indstiller fri kørsel, og en optagelse, der går i timeout, mens den er aktiveret, forklarer sig selv i stedet for blot at udskrive `-1011`. `PRESETS["triggered"]` aktiverer Line2, som det er beregnet.
>
> Sådan tvinges et hvilket som helst kamera til friløb:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Hvis der stadig opstår timeout med `trigger_mode="Off"`, leverer kameraet virkelig ikke data — send os logfilen og `ip link show`.

#### Farveprofiler (Live-forhåndsvisning af RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` vælger skærmens farveprofil til **live-forhåndsvisningen** på RGB-kameraer (multispec-kameraer ignorerer indstillingen):

| Profil | Betydning |
| --- | --- |
| `raw` | Omgå den radiometriske kæde fuldstændigt. |
| `linear` | DSNU + flat + WB, ingen CCM, ingen gamma. |
| `natural` | Lineær + målt CCM + sRGB-gamma, kun med den enkle efterbehandling (chroma-udjævning + desaturering af højlys) — den realistiske standardindstilling. |
| `enhanced` | `natural` plus den fulde hub-parity-efterbehandling (defringe, vibrance, CLAHE lokal kontrast). Et mere fyldigt udseende til ca. **dobbelt så høj finishomkostning pr. billede**, hvilket giver en lavere LIVE-billedhastighed. |
| `custom_temp` | `natural`, men hvidbalancen fastsat til `custom_cct_k` Kelvin (DLS ignoreres; fastlåst til 2000–10000 K på backend-side). |

Profilen er en ****kun til live-forhåndsvisning** hastigheds-/udseende-knap: gemte optagelser får altid den fulde, rige finish uanset den valgte profil, så at vælge `natural` for at vinde billedtid tilbage sænker ikke kvaliteten af det, der ender på disken. En ukendt profil øger `ValueError`; når et chloros-backend er tilgængeligt, sendes ændringen også til det via POST, så det næste forhåndsvisningsbillede afspejler den (direct-SDK-brugere uden et backend får stadig ændringen i indstillingerne).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Mono (M3M)-kameraer og `Calibration`

Et mono **M3M**-kamera (`M3M-<lens>-F<wavelength>`) er enkeltbåndet: ét gråtoneplan, ingen Bayer-mosaik, ingen 3×3 spektral-crosstalk-matrix. `Calibration` genkender det og eksponerer et `is_mono`-flag. Reflektans gælder stadig som et radiometrisk kort pr. bånd (unmix er identitets ), men multibåndsberegninger på et enkelt kamera giver meningsfulde resultater i stedet for nonsens:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

For at oprette et vegetationsindeks fra monokrom hardware skal du kombinere flere M3M-kameraer ved forskellige bølgelængder til en justeret multibåndsstak (se [Arrayjustering](#array-alignment)) og beregne indekset på tværs af denne stak i stedet for på et enkelt kamera.

DAQ-direkte-tilstand:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **`apply_sensor_settings` accepterede nøgler**— nøjagtigt `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; udfaset til fordel for `cap_id`), `filter_model` (DAQ-M)og `cap_id` (alle DAQ-typer; `None`/`""`/`"none"` = ren sensor, ingen kap-korrektion). Ukendte nøgler**ignoreres uden fejlmeddelelse** — f.eks. gør `{"integration_time": 64}` intet (det skal være `integration_time_ms`). Returnerer `{"applied": [...], "errors": {...}}` og genererer aldrig en fejl.

`chloros_sdk` reeksporterer kun den kerneoverflade, der er brugt ovenfor. Den fulde offentlige `daq_sdk`-API (22 navne) tilføjer følgende — importer dem direkte fra `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Undtagelser

Fang basisklassen for at håndtere &quot;alt, hvad der gik galt Chloros&quot;:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` og `ChlorosConfigurationError` eksporteres på øverste niveau sammen med resten; de kan også importeres fra `chloros_sdk.exceptions` som vist.

Hierarki:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Eksempler fra start til slut

### 1. Behandl en mappe med en brugerdefineret statusbjælke

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Live LATTICE-array → Refleksion + DAQ-reference

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Projektdrevet indsamlingskampagne

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Frame-stream fra flere kameraer → NumPy-pipeline

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Headless direkte hardware (ingen backend) optagelsesskript

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Kapacitetsundersøgelse før tilslutning af et 4-kamera-array

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Optagelsesopskrift svarende til (ren Python)

CLIs opskrifts-DSL har en direkte Python-ækvivalent:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Automatisk start af backend

Smart-Connect-indgangspunkterne — `connect_camera`, `connect_array`, `connect_daq_sensor` og `discover_lattice_cameras` — er tynde HTTP-klienter, der antager, at et backend lytter på `127.0.0.1:5000` (smart-connect-grænsefladens standard-URL). Når GUI’en eller CLI allerede kører, er der et. Fra et simpelt script er der muligvis ikke noget — så disse funktioner **starter automatisk-starter den medfølgende backend-binærfil** (uden vindue, på samme måde som `ChlorosLocal` gør) før deres første opkald og venter derefter op til `backend_startup_timeout` på, at den kommer op.

Regler:

- **Der oprettes kun en lokal URL.** En `backend_url`, der peger på `localhost` / `127.0.0.1` / `[::1]` er tilladt; enhver anden vært antages at være en andens maskine og oprettes aldrig.
- **Backend&#x27;en forbliver kørende med henblik på genbrug** (på samme måde som ved CLI) — der sker ingen implicit nedlukning, når dit script afsluttes. Når scriptet køres igen, genbruges den aktive backend.
- **Fravælg med `auto_start_backend=False`** ved et hvilket som helst af disse opkald (f.eks. når du har peget på en fjernbackend, eller når du selv styrer backendens livscyklus).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Hvis den medfølgende binærfil ikke kan findes eller startes, udløser det efterfølgende HTTP-kald en handlingsbar, **platformspecifik** `ChlorosConnectError` i stedet for en simpel fejlmelding om afvist forbindelse — på Windows henviser den dig til desktop-appen eller en `chloros-cli`-kommando; på Linux (ingen GUI) henviser den til en `chloros-cli`-kommando eller `.deb`.

---

## Miljø og headere

SDK markerer hvert backend-HTTP-opkald med `X-Chloros-Client: sdk`. Backend&#x27;et anvender licensreglerne fra SDK / CLI (login **og** et betalt abonnement på Chloros+ kræves) i stedet for GUI&#x27;ens gratis-niveau. Dette indstilles automatisk ved import — duikke behøver at gøre noget.

`http://localhost` og `http://127.0.0.1` registreres som den lokale backend. Opkald til andre værter (f.eks. din egen analysetjeneste) forbliver uændrede.

Tilsidesæt backend-URLen ved at angive `backend_url=` (eller `api_url=` på `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(En ikke-loopback-`backend_url` når kun en source/dev-backend — medfølgende backends binder kun til loopback; se Fjern-backend-tilstand for tunnelmønsteret.)

---

## Versionering og kompatibilitet

- SDK-versionen eksponeres som `chloros_sdk.__version__`.
- SDK fastlægger adfærden til den medfølgende backend-version. Det fungerer normalt at blande en ældre SDK med en nyere backend (fremadkompatible endpoints), men at blande en nyere SDK med en ældre backend kan medføre `404`-fejl på nye endpoints — opgrader desktop-appen, så den passer.
- Den smarte-connect-grænseflade (`connect_camera` / `connect_array` / `connect_daq_sensor`) og endepunktet til netværksanalyse returnerer stabile JSON-skemaer; nye felter tilføjes.

---

## Fejltil fejlfinding

- **`ChlorosAuthenticationError: Login required`** → Kør `chloros-cli login EMAIL PASSWORD` én gang på denne maskine, eller log ind via Chloros-desktopappen.
- **`ChlorosConnectError: No Chloros backend is running …`** → Smart-connect-opkaldet starter automatisk en lokal backend, så denne fejl vises kun, når den medfølgende binærfil ikke kan findes/startes (f.eks. en host, der kun kører pip uden desktop-pakke). Meddelelsen er platformspecifik: på Windows skal du åbne desktop-appen eller køre en vilkårlig `chloros-cli`-kommando; på Linux skal du køre en `chloros-cli`-kommando (der findes ingen GUI) eller installer `.deb`. For en fjernbackend skal du angive `backend_url=` (og `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** ved import → `lattice_sdk` kunne ikke indlæses (typisk er Arena-SDK-runtime-DLL&#x27;erne ikke installeret). Overfladen uden kamera fungerer stadig.
- **Array connect returnerer sub-native opløsning**→ Backendenssmart-prep-funktion reducerer automatisk billedstørrelsen, så den passer til forbindelsen. Brug `analyze_array_network()` for at se hvorfor, og opgrader derefter forbindelsen, accepter reduktionen eller angiv `force_tier="slip-emit-and-capture"` for sekventiel optagelse. Sikkerhedsforanstaltningen ved reduktionen**gælder ikke** dække samlet overabonnement (`oversubscribed: true`, fps-felter 0): for mange kameraer til kablen kan ikke løses ved binning/ROI — reducer antallet af kameraer, aktiver jumbo-rammer eller skift til et hurtigere netværkskort (se [Over-Subscription](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` rapporterer, at NIC&#x27;ens RX-ring er meget lille (~0,26 MB) / forbindelsesportene viser &quot;FRAMES WILL DROP&quot;** → Værts-NIC&#x27;ens modtagelsesring er på standardindstillingen (ofte nulstillet til 32 efter en opdatering af NIC-driveren). På en Realtek USB 10GbE-adapter skal du indstille `ReceiveBufferLen=256` og `PendingReceives=64` (forhøjet), og genstart derefter backend, så den genlæser ringen. Fuld procedure: [CLI Reference → Opsætning og finjustering af værts-NIC](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Værten fryser ved genstart/nedlukning, efterfølgende WMI `Invalid class`-fejl / NIC kan ikke aktiveres** → Forældet USB 10GbE-driver forårsager `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Opdater adapterdriveren til en aktuel version (≥ 2026) og anvend modtagerringindstillingerne på ny. Se [CLI Reference → Host NIC Setup &amp; Tuning](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Reflektans afvist** → Der skal være knyttet en aktiv DAQ til kameraet (eller arrayet) for at opnå reflektans i absolut skala. Knyt enten via GUI’en, eller brug `processing="radiance"` (W/m²/sr/nm), som ikke kræver en parret sensor.
- **`smart=True`-optagelse tager længere tid end forventet** → AE-konvergens afhænger af scenens dynamik; stram `exposure_tolerance_pct` eller forkort `stability_window_s`, hvis du ønsker en hurtigere (mindre stabil) trigger.

---

## Se også

- [CLI Reference](cli-reference.md) — hver CLI-underkommando afspejler et SDK-kald.
- [DAQ-sensorvejledning](../daq/README.md) — sensorspecifik ledningsføring, kalibrering og registreringsregler.
- Online-dokumentation: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>

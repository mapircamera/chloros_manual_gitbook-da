# API: Python SDK

{% hint style="info" %}
**Leder du efter den komplette API?** Denne side er en praktisk vejledning. Alle offentlige klasser, metoder, nøjagtige signaturer og eksempler, der kan kopieres og indsættes, findes i [SDK-referencen](reference/sdk-reference.md), som er optimeret til AI-assistenter.**Arbejder du med en AI-assistent?** Indsæt denne URL i chatten, så den har den fulde, aktuelle Chloros 1.3.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Hver side i denne manual er tilgængelig som rå markdown under dens slug med små bogstaver + `.md`, og hele manualen er indekseret på `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` på PyPI) styrer alt, hvad desktop-appen kan udføre fra Python: batch-billedbehandling, live-styring af LATTICE-kameraer og -arrayer, DAQ-lyssensor-sessioner samt automatisering af gemte projekter. Det er et tyndt lag oven på det samme lokale backend, som GUI'en og CLI'en bruger (HTTP på `127.0.0.1:5000`), så funktionaliteten er identisk på tværs af alle tre grænseflader.

## Installation

SDK’et følger med Chloros-desktop-pakken, som også indeholder backend’et til billedbehandling og hardware-runtimes, og versionen stemmer altid overens med denne.

**Trin 1 — Installer Chloros.** Windows: Kør desktop-installationsprogrammet (standardsti `C:\Program Files\Chloros\`) fra [Download](download.md)-siden. Linux: Installer pakken `.deb` med `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Linux-installation](linux/linux-installation.md)).**Trin 2 — Kontroller SDK** (Python 3.7+). Der er normalt intet, der skal installeres:

* **Windows:** installationsprogrammet installerer det medfølgende SDK-wheel i dit system-Python (hvis der ikke er installeret noget Python, springer det dette trin over).
* **Linux:** `.deb` installerer SDK'en til systemets `python3` i `/usr/lib/python3/dist-packages`. Dets krav, `python3-numpy` og `python3-requests`, anbefales af pakken: `apt install` medbringer dem, mens `dpkg -i` ikke gør det.
* **Et virtualenv** kan ikke se systemets pakker; installer den medfølgende wheel-fil i det: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` på Linux eller `chloros_sdk-*.whl` i mappen `sdk` i Windows-installationsmappen (`C:\Program Files\Chloros\sdk\` som standard).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

På en maskine uden en Chloros-pakke, hvor SDK'en driver et backend på en anden maskine (se [Hvad en pip-only-installation giver](#what-a-pip-only-install-gets)), skal du installere den fra PyPI: `pip install chloros-sdk`. PyPI opdateres ved hver udgivelse, så den altid svarer til den seneste stabile udgave.

**Trin 3 — Log ind én gang pr. maskine:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Adgangskoder gemmes i `~/.chloros/` (begge platforme). På Windows kan du tilsvarende logge ind via fanen Bruger <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> i desktop-appen. SDK'et kræver et betalt Chloros+-abonnement — se [Licenskrav](#license-requirement) nedenfor.

| Krav | Detaljer |
| --- | --- |
| **Chloros installeret** | Windows: desktop-installationsprogram; Linux: `.deb`-pakke (indeholder backend-binærfilen og SDK'en) |
| **Python** | 3.7 eller nyere (udviklet/testet på 3.10) |
| **Operativsystem** | Windows 10/11 64-bit, Ubuntu 22.04 LTS eller nyere, eller NVIDIA Jetson (JetPack 6) |
| **Licens** | Aktivt Chloros+-login, ethvert betalt abonnement (Copper eller højere) |

## Sejren på 60 sekunder

Et enkelt kald opretter et projekt, importerer en mappe, konfigurerer behandlingen og kører pipelinen — og starter backend-programmet automatisk, hvis det ikke allerede kører:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(På Linux skal du bruge Linux-stier: `/home/user/drone_images/flight001`. SDK'et fungerer på samme måde på begge platforme.)

Skal du behandle en LATTICE-optagelsesmappe? Brug den LATTICE-venlige wrapper — den anvender de rigtige standardindstillinger (ingen detektion af panelmål, standard debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — fuld kontrol over pipelinen

Til alt, der går ud over en enkelt linje, skal du bruge `ChlorosLocal`. Det bruger et backend, der allerede kører (desktop-appens), eller starter det installerede ved første brug (`auto_start_backend=True`; på Windows fra mappen `backend` i Chloros-installationsmappen, som standard `C:\Program Files\Chloros\backend\`), opretter og konfigurerer projekter, overvåger fremskridt og returnerer en oversigt efter kørsel.

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

{% hint style="info" %}
Behold standardindstillingen `http://127.0.0.1:5000` i stedet for at erstatte den med `localhost` — på Windows omdirigeres `localhost` først til `::1`, hvilket tager ca. 2 sekunder pr. anmodning mod backend'en, der kun understøtter IPv4.
{% endhint %}

Brug den som en kontekstmanager. Når du forlader blokken, afsluttes denne instans’ brug af backend’en, ikke selve backend’en: den fortsætter med at køre for det næste script og stopper af sig selv efter 30 minutters inaktivitet. `shutdown_backend()` stopper den instans, der er startet.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` accepterer følgende nøgleord: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` og `custom_settings`. De vigtigste værdier:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

De LATTICE-specifikke knapper (`input_level`, `radiometric_output`, `array_alignment*`-familien) er dokumenteret med deres fulde værditabeller i [SDK-referencen](reference/sdk-reference.md#supported-values).

### Overvågning af fremskridt

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Læsning af oversigten efter kørsel — og opfangning af tomme kørsler

Når kørslen er afsluttet, vedhæfter `process()` backendens behandlingsoversigt som `result["summary"]`. Hver post i `summary["hints"]` er en fuld sætning, der forklarer alt, hvad der er værd at bemærke — for eksempel hvorfor en kørsel ikke gav noget output — og hvert tip udsendes også igen som en Python-`UserWarning`, så tomme kørsler er selvdiagnostiserende, selvom du aldrig undersøger dict'en:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` udløses ikke, når en kørsel ikke genererer billeder.** Dette er det eneste sted, hvor SDK'en og CLI'en bevidst adskiller sig: `chloros-cli process` behandler »produkter blev anmodet om, ingen blev skrevet« som en fejl og afslutter med en værdi forskellig fra nul, mens SDK'et returnerer normalt og rapporterer tilstanden via `summary` / hints. Hvis din pipeline skal stoppe ved en tom kørsel, skal du selv kontrollere det — undersøg `summary` (eller tæl filerne i projektmappen) i stedet for at stole på en undtagelse.
{% endhint %}

## Smart Connect — live-hardware

Tre hjælpeprogrammer åbner vedvarende sessioner i backendens hardwarepulje — den samme pulje, som CLI, andre scripts og GUI'ens kamerafaner bruger, så SDK-scripts kan eksistere side om side med dem uden at kæmpe om serielle porte eller netværksbåndbredde. Alle tre starter automatisk en lokal backend, hvis der ikke kører nogen. GUI'ens DAQ-panel er ikke en del af puljen: Luk en lyssensor i GUI'en, før du åbner den fra et script.

### Enkelt LATTICE-kamera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Relative stier henviser til det sted, hvor dit script kører (ændret i 1.3.0).** Med et lokalt backend er en relativ `output_dir` (inklusive standarden `"output"`) — og de andre stier, som backenden åbner (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — løses i forhold til dit scripts arbejdsmappe, så `cam.capture("output/")` skriver til `<current folder>/output/`. I version 1.2.0 fortolkede et installeret backend dem i sin egen installationsmappe og gav fejlen »Adgang nægtet«. Ved brug af et fjernbackend sendes stierne præcis som de er indtastet.
{% endhint %}

### Synkroniseret array — `connect_array`

`connect_array` er det anbefalede startpunkt for rigge med flere kameraer. Det kører den samme smart-prep-proces som GUI'en: netværksanalyse, automatisk valg af synkroniseringsniveau, PTP-tidssynkronisering, valg af pixelformat pr. kamera, AE-seeding og aktivering af GPIO-trigger. Den **første serienummer er masteren** (den udløser hardware-triggerimpulsen); de øvrige er slaver.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Tilføj `smart=True` til enhver array-optagelse for at afvente, at den automatiske eksponering er stabil på tværs af alle kameraer, før der udløses.

Nyt i 1.3.0: `ArraySession` leverer også synkroniserede billeder direkte, uden at der skrives filer:

* `arr.grab()` returnerer ét billede pr. kamera, alle fra én udløsning, som numpy-arrays (`{serial: array}`); `.group_id` er den delte udløser, og `.errors` angiver eventuelle kameraer, der er udeladt.
* `arr.frame_stream(fps=5.0)` leverer ét sådant sæt pr. tick til live-behandling i din egen pipeline.
* `arr.continuous("output/")` kører backendens egen kontinuerlige optagelse (GUI'ens kontinuerlige tilstand) og returnerer en `ContinuousHandle` sammen med `status()`, `wait()` og `stop()` — brug det i stedet for en optagelsessløjfe, når du har brug for, at hver eneste trigger gemmes med høj frekvens.

For indlæsningstilstande (Enkelt / Kontinuerlig / Interval / Hurtigst), optagere, burst-til-video og array-justering, se [SDK-referencen](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### DAQ-lyssensor — `connect_daq_sensor`

Uden argumenter registrerer `connect_daq_sensor()` automatisk transportprotokollen (prioritet: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Hver ramme indeholder 135-punkts `spectrum` (W/m²/nm, når kalibreret), et `is_saturated`-flag samt CIE `x`, `y`, `z`. For at fastlægge en bestemt sensor eller transport skal du angive et eksplicit tip:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Bemærk, at cap-korrektionsprofiler (`cap_id`) **ikke** er en SDK-indstilling — vælg dem i stedet via `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Gemte projekter — `open_project`

Et gemt Chloros-projekt bevarer sin tilsluttede hardware (`cameras.json` + `sensors.json` sammen med `project.json`), og `chloros_sdk.open_project(path)` kan genoprette forbindelsen til sine kameraer, arrays og lyssensorer på én gang og styre optagelser efter enhedsnavn. Lyssensorerne genopretter forbindelsen via et kørende Chloros-backend med deres gemte indstillinger og optagelseskapacitet. Se [Projektautomatisering](reference/sdk-reference.md#project-automation--chlorosproject) i referencen.

## Hvad en installation, der kun omfatter pip, giver

Kontroller tilgængelighedsflagene på modulniveau, før du bruger hardwareoverflader:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

På en vært med **kun** `pip install chloros-sdk` og ingen Chloros-desktop-pakke:

* `ChlorosLocal`, `process_folder` og `process_lattice_capture` fungerer **ikke** — de kræver den backend-binærfil, der følger med desktop-installationsprogrammet.
* Smart-connect-hjælpeprogrammerne (`connect_camera`, `connect_array`, `connect_daq_sensor`) er rene HTTP-klienter, så de fungerer sammen med en backend på en anden maskine — men de medfølgende backends binder kun til loopback, så du skal selv videresende porten (f.eks. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) og sende `backend_url="http://127.0.0.1:5000"` sammen med `auto_start_backend=False`. Se [Remote-Backend Mode](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* LATTICE-klasserne med direkte hardware-tilkobling (`LatticeCamera`, `CameraPool`, …) kan importeres, men kræver Arena SDK-runtime fra desktop-pakken — uden denne er `CAMERA_AVAILABLE` lig med `False`.
* DAQ-sensorer styres udelukkende via et (tunnelet) backend — `connect_daq_sensor()` eller et projekts `proj.sensors`; backenden åbner sensoren på sin egen maskine.

## AI-assistenter — `chloros-mcp`

SDK'et indeholder `chloros-mcp`, en MCP-server, der gør det muligt for Claude Desktop, Claude Code, Cursor og andre MCP-klienter at styre Chloros: behandle billeder og køre LATTICE-kameraer, -arrays og DAQ-lyssensorer. Dens værktøjer dækker det, som denne SDK gør. Den installeres sammen med SDK'en, så opsætningen for Claude Code består af én linje:

```bash
claude mcp add chloros -- chloros-mcp
```

Andre klienter: [Brug af Chloros med AI-assistenter](ai-assistants.md). Indstillinger og alle værktøjer: [MCP-server (AI-assistenter)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Licenskrav

Adgang til SDK kræver et aktivt Chloros+-login på et hvilket som helst betalt abonnement — **Copper eller højere**(Copper / Bronze / Silver / Gold); det gratis Iron-abonnement har ingen adgang til SDK/CLI. Håndhævelsen sker**på serversiden**: hver SDK-anmodning skal indeholde både en aktiv session og et betalt abonnement, ellers returnerer backend'et `403` / `PLAN_UPGRADE_REQUIRED` (genereret som `ChlorosLicenseError` af `ChlorosLocal` og som `ChlorosConnectError` af `connect_*`-hjælperne). En afmeldt bruger får i stedet fejlkoder `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) i stedet — en genkørsel af `chloros-cli login` løser det første tilfælde, men ikke det andet.

Offline-brug fungerer inden for planens afdragsfri periode: niveauet læses fra servervalideringscachen (5 minutter) eller den signerede, maskinbundne licenscache (30 dage for månedlige planer; indtil abonnementets udløb for årlige planer). Når fristen udløber, skifter abonnementet til gratisadgang, og SDK-adgangen stopper, indtil maskinen har oprettet forbindelse til serveren én gang. `chloros-cli status` forbliver tilgængeligt på det gratis niveau, så årsagen altid er synlig. Se [Chloros+ Login](chloros+-login.md).

## Undtagelser

Fang basisklassen for at håndtere "alt, hvad der gik galt med Chloros":

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

Alle pipeline-undtagelser (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) stammer fra `ChlorosError`. En ting man skal være opmærksom på: `ChlorosConnectError` — udløses kun af `connect_camera` / `connect_array` / `connect_daq_sensor` — stammer fra den almindelige `Exception`, **ikke** fra `ChlorosError`, så `except ChlorosError` vil ikke fange den. Det fulde hierarki findes i [SDK-referencen](reference/sdk-reference.md#exceptions).

## Se også

* [SDK-referencen](reference/sdk-reference.md) — den komplette API-overflade, optimeret til AI-assistenter.
* [CLI-reference](reference/cli-reference.md) — hver CLI-underkommando afspejler et SDK-kald.
* [Brug af Chloros med AI-assistenter](ai-assistants.md) — opsætnings- og promptvejledninger til `chloros-mcp`.
* [Download](download.md) — installationsprogrammer til Windows og Linux.

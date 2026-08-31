# API : Python SDK

{% hint style="info" %}
**Leder du efter den komplette API?** Denne side er en praktisk vejledning. Alle offentlige klasser, metoder, nøjagtige signaturer og eksempler, der kan kopieres og indsættes, findes i [SDK-referencen](reference/sdk-reference.md), som er optimeret til AI-assistenter.**Arbejder du med en AI-assistent?** Indsæt denne URL i chatten, så den har den fulde, aktuelle Chloros 1.2.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Hver side i denne vejledning er tilgængelig som rå markdown under dens slug med små bogstaver + `.md`, og hele vejledningen er indekseret på `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` på PyPI) styrer alt, hvad desktop-appen kan udføre fra Python: batch-billedbehandling, live-styring af LATTICE-kameraer og -arrayer, DAQ-lyssensorsessioner samt automatisering af gemte projekter. Det er et tyndt lag oven på det samme lokale backend, som GUI’en og CLI bruger (HTTP på `127.0.0.1:5000`), så funktionen er identisk på alle tre grænseflader.

## Installation

Installationen foregår i to trin: først Chloros-desktop-pakken (den indeholder backend til billedbehandling og hardware-runtimes), derefter Python-pakken.

**Trin 1 — Installer Chloros.** Windows: Kør desktop-installationsprogrammet (standardsti `C:\Program Files\MAPIR\Chloros\`) fra siden [Download](download.md). Linux: Installer `.deb`-pakken ([Linux-installation](linux/linux-installation.md)).**Trin 2 — Installer SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Du behøver muligvis ikke engang pip: hver installer leveres med et tilhørende SDK-wheel. Windows-installeren installerer det automatisk i dit system Python; Linux `.deb` placerer det i `/usr/lib/chloros/sdk/` og viser den nøjagtige `pip install --user`-kommando. PyPI opdateres ved hver udgivelse, så `pip install chloros-sdk` svarer til den seneste stabile udgave.

**Trin 3 — Log ind én gang pr. maskine:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Loginoplysningerne gemmes i `~/.chloros/` (på begge platforme). På Windows kan du på samme måde logge ind via fanen Bruger<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">er i desktop-appen. SDK kræver et betalt Chloros+-abonnement — se [Licenskrav](#license-requirement) nedenfor.

| Krav | Detaljer |
| --- | --- |
| **Chloros installeret** | Windows: desktop-installationsprogram; Linux: `.deb`-pakke (leverer backend-binærfilen) |
| **Python** | 3.7 eller nyere (udviklet/testet på 3.10) |
| **Operativsystem** | Windows 10/11 64-bit, Ubuntu 22.04 LTS eller nyere, eller NVIDIA Jetson (JetPack 6) |
| **Licens** | Aktivt Chloros+-login, ethvert betalt abonnement (Copper eller højere) |

## 60 sekunders sejr

Et enkelt opkald opretter et projekt, importerer en mappe, konfigurerer behandlingen og kører pipelinen — og starter automatisk backend&#x27;en, hvis den ikke allerede kører:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(På Linux skal du bruge Linux-stier: `/home/user/drone_images/flight001`. SDK fungerer på samme måde på begge platforme.)

Behandler du en LATTICE-optagelsesmappe? Brug den LATTICE-venlige wrapper — den anvender de rigtige standardindstillinger (ingen detektion af panelmål, standard debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — fuld kontrol over behandlingsforløbet

Til alt, der går ud over en enkelt linje, skal du bruge `ChlorosLocal`. Den starter backend ved første brug (`auto_start_backend=True`), opretter og konfigurerer projekter, overvåger fremskridt og returnerer en oversigt efter kørsel.

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
Behold standardindstillingen `http://127.0.0.1:5000` i stedet for at erstatte den med `localhost` — på Windows `localhost` omdirigeres først til `::1` og tager ca. 2 sekunder pr. anmodning mod backend&#x27;en, der kun understøtter IPv4.
{% endhint %}

Brug den som en kontekstmanager for at sikre oprydning:

```python
import chloros_sdk

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

`configure()` accepterer følgende nøgleord: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` og `custom_settings`. De vigtigste værdier:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
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

### Læsning af oversigten efter kørsel — og opdagelse af tomme kørsler

Når kørslen er afsluttet, vedhæfter `process()` backendens behandlingsoversigt som `result["summary"]`. Hver post i `summary["hints"]` er en fuld sætning, der forklarer alt, hvad der er værd at bemærke — for eksempel hvorfor en kørsel ikke gav noget output — og hvert tip genudsendes også som en Python `UserWarning`, så tomme kørsler er selvdiagnostiserende, selvom du aldrig tjekker ordbogen:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` udløses ikke, når en kørsel ikke genererer nogen billeder.** Dette er det eneste sted, hvor SDK og CLI bevidst adskiller sig: `chloros-cli process` behandler &quot;der blev anmodet om produkter, men ingen blev skrevet&quot; som en fejl og afslutter med en værdi forskellig fra nul, mens SDK afslutter normalt og rapporterer tilstanden via `summary` / hints. Hvis din pipeline skal stoppe ved en tom kørsel, skal du selv kontrollere dette — undersøg `summary` (eller tæl filerne i projektmappen) i stedet for at stole på en undtagelse.
{% endhint %}

## Smart Connect — live-hardware

Tre hjælpeprogrammer åbner vedvarende sessioner i backend-hardwarepuljen — den samme pulje, som GUI’en bruger, så SDK-scripts kan køre sideløbende med desktop-appen uden at konkurrere om serielle porte eller netværksbåndbredde. Alle tre starter automatisk en lokal backend, hvis der ikke kører nogen.

### Enkelt LATTICE-kamera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Synkroniseret array — `connect_array`

`connect_array` er det anbefalede udgangspunkt for opsætninger med flere kameraer. Det kører den samme smart-prep-proces som GUI&#x27;en: netværksanalyse, automatisk valg af synkroniseringslag, PTP-tidssynkronisering, valg af pixelformat pr. kamera, AE-seeding og aktivering af GPIO-trigger. Den **første serielle enhed er masteren** (den udløser hardware-triggerimpulsen); de øvrige er slaver.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Tilføj `smart=True` til enhver array-optagelse for at afvente, at den automatiske eksponering er stabil på tværs af alle kameraer, før der udløses. For optagelsestilstande (Enkelt / Kontinuerlig / Interval / Hurtigst), optagere, burst-til-video og array-justering, se [SDK-referencen](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### DAQ-lyssensor — `connect_daq_sensor`

Uden argumenter registrerer `connect_daq_sensor()` automatisk transporttypen (prioritet: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Hver ramme indeholder 135-punkts `spectrum` (W/m²/nm ved kalibrering), et `is_saturated`-flag og CIE `x`, `y`, `z`. For at udpege en bestemt sensor eller et bestemt transportlag — det pålidelige valg på værter med flere netværksgrænseflader, hvor Ethernet-autodetektering kan overse en velfungerende DAQ-E ved første forsøg — skal der angives et eksplicit tip:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Bemærk, at profiler til kapacitetskorrektion (`cap_id`) **ikke** er en SDK-indstilling — vælg dem i stedet via `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap` i stedet.

### Gemte projekter — `open_project`

Et gemt Chloros-projekt bevarer sin tilsluttede hardware (`cameras.json` + `sensors.json` sammen med `project.json`), og `chloros_sdk.open_project(path)` kan genoprette forbindelse til det hele på én gang og udføre datafangst efter enhedsnavn. Se [Projektautomatisering](reference/sdk-reference.md#project-automation--chlorosproject) i referencen.

## Hvad en installation, der kun omfatter pip, giver

Kontroller tilgængelighedsflagene på modulniveau, før du bruger hardwareoverflader:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

På en vært med **kun** `pip install chloros-sdk` og ingen Chloros-desktop-pakke:

* `ChlorosLocal`, `process_folder` og `process_lattice_capture` fungerer **ikke** — de kræver den backend-binærfil, der følger med desktop-installationsprogrammet.
* Smart-connect-hjælpeprogrammerne (`connect_camera`, `connect_array`, `connect_daq_sensor`) er rene HTTP-klienter, så de fungerer sammen med en backend på en anden maskine — men de medfølgende backends binder kun til loopback, så du skal selv videresende porten (f.eks. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) og videregive `backend_url="http://127.0.0.1:5000"` sammen med `auto_start_backend=False`. Se [Remote-Backend Mode](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* De direkte hardwarebaserede LATTICE-klasser (`LatticeCamera`, `CameraPool`, …) kan importeres, men kræver Arena SDK-runtime fra desktop-pakken — uden denne er `CAMERA_AVAILABLE` lig med `False`.
* `daq_sdk` (de direkte DAQ-klasser) følger med desktop-installationen, ikke med PyPI-pakken, så `DAQ_AVAILABLE` er `False` på en host, der kun kører pip — styr i stedet DAQ-sensorerne via `connect_daq_sensor()` mod et (tunneleret) backend.

## Licenskrav

Adgang til SDK kræver et aktivt Chloros+-login på et hvilket som helst betalt niveau — **Copper eller højere**(Copper / Bronze / Silver / Gold); det gratis Iron-abonnementsniveau har ingen adgang til SDK/CLI. Håndhævelsen sker**på serversiden**: hver SDK-anmodning skal indeholde både en aktiv session og et betalt abonnement, ellers returnerer backend&#x27;en `403` / `PLAN_UPGRADE_REQUIRED` (genereret som `ChlorosLicenseError` af `ChlorosLocal` og som `ChlorosConnectError` af `connect_*`-hjælperne). En afmeldt bruger får i stedet `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) i stedet — genkørsel af `chloros-cli login` løser det første tilfælde, men ikke det andet.

Offline-brug fungerer inden for planens nådeperiode: abonnementsniveauet læses fra servervalideringscachen (5 minutter) eller den signerede, maskinbundne licenscache (30 dage for månedlige planer; indtil abonnementets udløb for årlige planer). Når fristen udløber, skifter abonnementet til gratisversionen, og adgangen via SDK stopper, indtil maskinen har oprettet forbindelse til serveren én gang. `chloros-cli status` forbliver tilgængeligt på gratisniveauet, så årsagen altid er synlig. Se [Chloros+ Login](chloros+-login.md).

## Undtagelser

Fang basisklassen for at håndtere &quot;alt, hvad der gik galt med Chloros&quot;:

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

Alle pipeline-undtagelser (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) stammer fra `ChlorosError`. En ting man skal være opmærksom på: `ChlorosConnectError` — udløses kun af `connect_camera` / `connect_array` / `connect_daq_sensor` — stammer fra den almindelige `Exception`, **ikke** fra `ChlorosError`, så `except ChlorosError` vil ikke opfange den. Det fulde hierarki findes i [SDK-referencen](reference/sdk-reference.md#exceptions).

## Se også

* [SDK-reference](reference/sdk-reference.md) — den komplette API-overflade, optimeret til AI-assistenter.
* [CLI-reference](reference/cli-reference.md) — hver CLI-underkommando afspejler et SDK-kald.
* [Download](download.md) — installationsprogrammer til Windows og Linux.

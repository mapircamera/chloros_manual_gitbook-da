# CLI : Kommandolinje

> **Komplet reference:**[CLI Reference](reference/cli-reference.md) dokumenterer**alle flag for alle underkommandoer** og er optimeret til AI-assistenter — indsæt URL i din assistent og bed om en fungerende kommando: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tip til AI-værktøjer:** Enhver side i denne manual er tilgængelig som rå Markdown ved at tilføje `.md` til dens URL (f.eks. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), og `https://mapir.gitbook.io/chloros/llms.txt` indekserer hele manualen til brug for LLM.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## Hvad er „CLI
“?

`chloros-cli` er kommandolinjegrænsefladen til den samme behandlingsmotor, som desktop-appen „Chloros
“ bruger. Det er en tyndHTTP
-klient, der kører oven påChloros
-backend (en lokal server på `127.0.0.1:5000`) — de fleste kommandoer starter backend automatisk, så et enkelt `chloros-cli process …`-kald er alt, hvad et script behøver.

Den kører på **Windows
10/11 (x64)**og**Linux
(x86_64 og NVIDIA Jetson arm64 på JetPack 6)** i enhver terminal uden behov for en grafisk brugergrænseflade. Kontroller din installation med:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Kommandofamilierne i kort oversigt:

* **Behandling og konto** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 sprog — se [Understøttede sprog](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (kunLinux
/Jetson)
* **Live-hardware** — `lattice` (LATTICE-kamerastyring, 45+ underkommandoer), `daq pool-*` (DAQ-lyssensorer), `time-sync` (PTP)
* **Automatisering** — `project` (kør et gemtChloros
-projekt uden brugergrænseflade, inklusive YAML-optagelsesopskrifter)

Globale indstillinger, der er værd at kende: `--port N` (backend-port, standard `5000`), `-v/--verbose`, `--restart` (tvinger genstart af backend), `--backend-exe PATH`. Se [CLI
-referencen](reference/cli-reference.md) for den fulde liste.

***

## Installation

CLI
en **leveres sammen medChloros
-installationsprogrammet** på alle platforme — der findes ingen separatCLI
-download. Hent installationsprogrammet fra [Download](download.md)-siden.

###Windows


Installationsprogrammet placererCLI
på:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

og tilføjer denne mappe til dit system `PATH` — **åbn en ny terminal**efter installationen, så den opdaterede `PATH` bliver hentet. Installationsprogrammet placerer også startskripter (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) i installationsrodmappen samt en**Chloros
CLI
** genvej i Start-menuen, som hver især åbner en terminal med `chloros-cli` klar til brug.

###Linux


Installer `.deb` til din arkitektur:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Dette installerer `chloros-cli` til `/usr/bin/chloros-cli` (allerede på `PATH`) og backend til `/usr/lib/chloros/chloros-backend`, sammen med Arena-SDK
-runtime, der er nødvendig for LATTICE-kameraer. Se [Linux
Installation](linux/linux-installation.md) for yderligere oplysninger.

### Bekræft

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Login og licensering

CLI
(ogPython
SDK
) kræver et **betaltChloros
+ abonnement**— alle betalte abonnementer har det; det gratis abonnement har det ikke. Begrænsningen håndhæves**på serversiden** af backend-systemet, ikke afCLI
-binærfilen: et opkald fra en udlogget bruger afvises med `401 AUTH_REQUIRED`, og et opkald fra en indlogget bruger på det gratis abonnement med fejlkoden `403 PLAN_UPGRADE_REQUIRED`, uanset om det kommer fra `chloros-cli`,SDK
eller en håndkodetHTTP
-klient. Opgrader på [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Log ind **én gang pr. maskine**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Adgangskoder med specialtegn**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` bliver forvrænget af shellen (CLI
en registrerer dette ved en 401-fejl og forsøger automatisk igen, men enkelt anførselstegn undgår problemet helt).
{% endhint %}

Sessionen gemmes i cachen som `~/.chloros/user_session.json` og fungerer fortsat offline i abonnementsperiodens afdragsfri periode (30 dage for månedsabonnementer, indtil udløb for årsabonnementer). `chloros-cli status` fungerer selv uden et betalt abonnement, så årsagen til et afslag altid er synlig.

{% hint style="danger" %}
**Planlægger du headless-opgaver? Log ind først.**En kommando til opstart af backend (`process`, `status`, `export-status`, …) der køres uden**en cachelagret session**, fejler ikke hurtigt — den går over i en interaktiv `Email:` / `Password:`-prompt på stdin. Et uovervåget cron-job eller CI-trin vil derfor**hænge og vente på indtastning**. Kør `chloros-cli login EMAIL 'PASSWORD'` én gang på maskinen, før du planlægger noget.
{% endhint %}

***

## Din første behandlingskørsel

Peg `process` mod en mappe med optagelser — den registrerer automatiskSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` eller en blanding:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Statusopdateringer vises live pr. pipeline-tråd (Detektering, Analyse, Behandling, Eksport), og en vellykket kørsel afsluttes med en rapport over, hvor mange billedprodukter der er skrevet (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Hvor outputfilene gemmes

`process` skriver til en **projektmappe**, ikke til din inputmappe:

* Uden `-o`: projektet oprettes under din standardprojektmappe (delt med GUI&#x27;en; administrer den med `get-project-folder` / `set-project-folder`, fallback `~/Chloros Projects`), navngivet med `-n/--project-name` eller et tidsstempel (`YYYYMMDD_HHMMSS`), når det udelades.
* Med `-o PATH`: den mappe **er** projektmappen. Hvis den allerede indeholder en `project.json`, oprettes der en søstermappe med suffikset `_1`/`_2`… i stedet for at overskrive den eksisterende.

Inden for projektet grupperes produkterne **efter kamera og derefter efter filformat**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Kameramappen er `LATT-<sensor>-<lens>-F<filter>` for LATTICE (svarende til optagelsens EXIF-data `Model`) og `<model>_<filter>` (f.eks. `Survey3N_RGN`) forSurvey3
. Formatmappen følger `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` eller `tiff32` for `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Hvert eksporteret produkt beholder navnet på KILDEfilen.**En radiance-eksport af `capture_..._raw.tif` hedder stadig `capture_..._raw.tif` — den findes blot i `tiff32/Radiance_Images/`.**Det er mappen, der identificerer produktet, ikke filnavnet**, så brug glob-udtryk for mappen, ikke for et `*radiance*`-suffiks.
{% endhint %}

### De indstillinger, du rent faktisk vil bruge

| Flag | Standard | Hvad det gør |
| --- | --- | --- |
| `-o, --output PATH` | standardprojektmappe | Placering af projektmappe (se ovenfor). |
| `-n, --project-name NAME` | tidsstempel | Projektnavn. |
| `--format FMT` | `TIFF (16-bit)` | En af følgende: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | ingen | Vegetationsindekser, der skal eksporteres (se [Vegetationsindekser](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neural debayer, langsommere, højeste kvalitet (Chloros
+, NVIDIA GPU). |
| `--vignette / --no-vignette` | aktiveret | Vignettekorrektion. |
| `--reflectance / --no-reflectance` | aktiveret | Reflektanskalibrering; for LATTICE er dette også kontakten til reflektansproduktet. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Tving indgangsstedet i pipelinen for LATTICE TIFF-filer. |

For alt andet — finjustering af måldetektering, PPK, eksponeringspinde, flag til array-justering — se afsnittet [`process` iCLI
-referencen](reference/cli-reference.md).

***

## Valg af, hvad der skal eksporteres (LATTICE-produkter)

LATTICE-behandlingen fordeles på **alle relevante produkter i én gennemløb**. Fire produkt-specifikke indstillinger er alle**aktiveret som standard**; brug formularen `--no-` til at deaktivere en:

| Vælger | Produkt |
| --- | --- |
| `--debayered` | Lineær demosaik → `Debayered_Images/` |
| `--preview` | Vis forhåndsvisning (hvidbalance + gamma; falskfarveudvidelse til multispektral) → `Preview_Images/` |
| `--radiance` | float32-radiance, W/m²/sr/nm → `Radiance_Images/` (altid `tiff32/`) |
| `--reflectance` | uint16 reflektans, Pix4D-klar → `Reflectance_Calibrated_Images/` |

RGB
Masterkameraer udsender altid kun debayered + preview — stråling/refleksion pr. bånd er ikke meningsfuldt for en bredbåndssensor, så disse indstillinger har ingen effekt for dem.Survey3
`.raw` ignorerer indstillingerne og følger standardvejen for refleksion/mål.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (standard `auto`) vælger reflektansreferencen: `auto` opretter et QA-godkendt [kalibreringsmål](calibration-targets.md) i billedet som den absolutte reference og falder tilbage til DAQ-lyssensorens nedadgående delingsværdi (ρ = π·L/E), når der ikke er noget mål til stede; `target` er streng (ingen DAQ-erstatning); `daq` er DAQ-autoritativ. Målscanninger målt pr. enhed kan leveres med `--target-reflectance-dir`.

{% hint style="info" %}
**Aflæsning af reflektanspixels:**DN-værdien, der betyder ρ = 1,0, er**pr. kilde** — LATTICE-filer indsætter `Chloros:PixelScale=32768` i XMP;Survey3
-filer bruger 65535 (og indeholder ingen `Chloros:*`-tags). Læs tagget og divider med det i stedet for at antage en konstant. Detaljer og det ene bevidste undtagelsestilfælde uden skala findes i [CLI
-referencen](reference/cli-reference.md).
{% endhint %}

**Behandlingen starter altid fra `raw`.** Afledte produkter (eksport af debayering, strålingsintensitet og reflektans) føres aldrig tilbage gennem pipelinen — at importere dem igen og behandle dem ville medføre, at kalibreringsberegningerne blev anvendt to gange, såChloros
springer dem over og angiver dette. `--input-level` er den bevidste bagdør, når man virkelig har brug for at tvinge et indgangspunkt.

***

## Når en kørsel mislykkes

Fra og med version 1.2.0 fejler `process` tydeligt i stedet for at »lykkes« uden at vise noget:

* En kørsel, der **anmodede om produkter, men ikke skrev nogen**— kun `project.json` og `calibration_data.json` — udskriver `Processing finished but wrote no image products.` og**afsluttes med en værdi forskellig fra nul**, så scripts kan registrere det. De sædvanlige årsager: indlæsningsmappen blev ikke genkendt som en optagelse (tjek layoutet og `--input-level`), eller alle de anmodede produkter var uanvendelige for de pågældende kameraer (f.eks. anmodning om radiance/reflektans fra kameraer, der kun harRGB
).
* En **bevidst kørsel, der kun genererer metadata** (alle produkter slået fra, ingen `--indices`) betragtes stadig som en succes — en tom billedudgang er det korrekte resultat i dette tilfælde.
* Kør koden igen med `--verbose`, og tjek backend-loggen for linjer med `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, som forklarer, hvilke kameraer der er blevet sprunget over.

Afslutningskoder: `0` succes · `1` generisk fejl · `2` argumentfejl · `130` afbrudt af Ctrl+C.

***

## Vegetationsindekser

Kør `--indices` med et eller flere forudindstillede navne; hvert indeks havner i sin egen `<INDEX>_Index_Images/`-mappe:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

De 22 forudindstillede navne, som `process --indices` accepterer:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Der findes tre indekslister — bland dem ikke sammen.**Rullemenuen »Projektindstillinger« i GUI’en indeholder 27 formler (tilføjer `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — disse fem findes kun i GUI&#x27;en og gælder**ikke** for `--indices`). Live/offline-kommandoen `lattice index --preset` bruger sin egen separate liste med 22 forudindstillinger. Formler og båndberegninger er dokumenteret i [Formler til multispektrale indekser](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## DAQ-lyssensorer: En hurtig introduktion

`daq pool-*`-familien styrerMAPIR
DAQ-spektralsensorer (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet) gennem backendens permanente pool — GUI&#x27;en,CLI
ogSDK
deler alle ét live-handle. **`pool-*` er den understøttede DAQ-sti i den medfølgendeCLI
**; andre `daq`-underkommandoer, som du muligvis ser henvist til, er enMAPIR
-intern overflade, der kun er til kildeformål, og som afsluttes med en eksplicit fejl, der henviser dig til `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` uden `--duration` kører indtil `pool-record --stop`; standardudskrivningsmappen er `~/Documents/DAQ Live View/` **på backend-maskinen**. Cap-korrektionsprofilen vælges ved opkobling (`--cap-id`, backend-standard `sunshine_cosine`) og kan skiftes under kørsel med `pool-set-cap` — cap-profiler og sensorens kalibrerede område er beskrevet i DAQ-kapitlerne i denne manual.

{% hint style="warning" %}
**DAQ-E på en vært med flere netværkskort:** Den første `pool-connect --eth`-autodetektering efter opstart kan mislykkes, selv med en velfungerende sensor. `--eth-host <ip-or-hostname>` er den pålidelige løsning — brug den, når autodetekteringen ikke giver noget resultat.
{% endhint %}

***

## LATTICE-kameraer, PTP og projektautomatisering

`lattice`-familien (over 45 underkommandoer) dækker LATTICE-kamerabearbejdning fra start til slut: genkendelse, enkeltoptagelser, vedvarende synkroniserede arrays med GUI’ens »smart-prep connect«-flow, live-forhåndsvisning i browseren, justering, indeksberegninger og værts-NIC-diagnostik. Et eksempel:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Sideløbende hermed: `chloros-cli time-sync` rapporterer om den PTP-grandmaster, somChloros
-værten kører (LATTICE-kameraer og DAQ-E-sensorer fungerer som slaver til denne for tidsstempler på tværs af enheder), og `chloros-cli project` åbner et gemtChloros
-projekt og styrer dets kameraer, arrays og sensorer uden brugergrænseflade — herunder scriptbaserede YAML-optagelsesopskrifter.

Disse tre familier (`lattice`, `project`, `daq pool-*`) er også de eneste, der understøtter `CHLOROS_BACKEND_URL` til styring af et **fjerntliggende** backend; kerne-kommandoerne er altid rettet mod den lokale maskine.

Fuldstændige gennemgange findes i LATTICE-kapitlerne i denne manual; alle flag er beskrevet i [CLI
-referencen](reference/cli-reference.md).

***

## Fejlfinding: Top 5

| Symptom | Løsning |
| --- | --- |
| `Login required` eller en planlagt opgave hænger fast ved en `Email:`-prompt | Kør `chloros-cli login EMAIL 'PASSWORD'` én gang på denne maskine — kommandoer uden en cachelagret session kører interaktivt i stedet for at fejle hurtigt. |
| `backend unreachable` | Start desktop-appenChloros
, eller kør backend-binærfilen direkte (`chloros-backend`). Hvis du peger `lattice`/`project`/`daq pool-*` mod en fjernbackend, skal du kontrollere `CHLOROS_BACKEND_URL`. |
| Array-forbindelse blokeret: `FRAMES WILL DROP` / `Reduce ROI to enable` | Værts-NIC’ens modtagelsesring er nulstillet til standardindstillingerne — den hyppigste årsag til, at et tidligere velfungerende system nægter at oprette forbindelse, typisk efter en opdatering af NIC-driveren. Kør `chloros-cli lattice network --fix` fra en **terminal med administratorrettigheder** (eller indstil `ReceiveBufferLen=256`, `PendingReceives=64`); se afsnittet *Opsætning og finjustering af værts-NIC* i referencen. |
| `daq`-underkommando afbrydes: »kræver den fulde DAQ-pakke…« | Forventet på leverede builds — den kompileredeCLI
leverer kun `daq pool-*`-familien, som dækker tilslutning, streaming, optagelse og valg af datakapacitet. Brug `pool-*` (eller `chloros_sdk.connect_daq_sensor()` fraPython
). |
| Jetson viser en advarsel om swap, før der åbnes store mapper | Tilføj filbaseret swap —CLI
viser de nøjagtige `fallocate`/`swapon`-kommandoer, der skal køres. |

***

## Sådan får du hjælp

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Alle flag, alle underkommandoer:** [CLI
Reference](reference/cli-reference.md)
* **TilsvarendePython
:** [Python
SDK
](api-python-sdk.md) og [SDK
Reference](reference/sdk-reference.md)
* **Support:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)

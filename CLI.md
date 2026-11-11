# CLI: Kommandolinje

> **Komplet reference:**[CLI-reference](reference/cli-reference.md) dokumenterer**alle flag for alle underkommandoer** og er optimeret til AI-assistenter — indsæt URL'en i din assistent og bed om en fungerende kommando: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tip til AI-værktøjer:** Enhver side i denne manual er tilgængelig som rå Markdown ved at tilføje `.md` til dens URL (f.eks. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), og `https://mapir.gitbook.io/chloros/llms.txt` indekserer hele manualen til brug for store sprogmodeller (LLM).

> **Vil du have din assistent til at køre Chloros selv?** `chloros-mcp`, en MCP-server installeret med Python-SDK'en, fungerer sammen med Claude Desktop, Claude Code, Cursor og andre MCP-klienter: se [Brug af Chloros med AI-assistenter](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Hvad CLI'en er

`chloros-cli` er kommandolinjegrænsefladen til den samme behandlingsmotor, som Chloros-desktopappen bruger. Det er en letvægts-HTTP-klient over Chloros-backendet (en lokal server på `127.0.0.1:5000`) — de fleste kommandoer starter backendet automatisk, så et enkelt `chloros-cli process …`-kald er alt, hvad et script har brug for. Backend'et forbliver derefter kørende til den næste kommando: den stopper ved `chloros-cli logout` (medmindre appen er åben, eller der er en kørsel eller en build i gang), ved `chloros-cli backend stop` eller af sig selv efter 30 minutter uden klient, kørsel eller tilsluttet enhed. `chloros-cli backend status` viser det.

Det kører på **Windows 10/11 (x64)**og**Linux (x86_64 og NVIDIA Jetson arm64 på JetPack 6)**, i enhver terminal, uden behov for en GUI. Kontroller din installation med:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

Kommandofamilierne i kort oversigt:

* **Behandling og konto** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 sprog — se [Understøttede sprog](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (kun Linux/Jetson)
* **Live-hardware** — `lattice` (LATTICE-kamerastyring, over 45 underkommandoer), `daq pool-*` (DAQ-lyssensorer), `time-sync` (PTP)
* **Automatisering** — `project` (kør et gemt Chloros-projekt uden brugergrænseflade, inklusive YAML-optagelsesopskrifter)

Globale indstillinger, der er værd at kende: `--port N` (backend-port, standard `5000`), `-v/--verbose`, `--restart` (tvinger genstart af backend), `--backend-exe PATH`. Se [CLI-referencen](reference/cli-reference.md) for den fulde liste.

***

## Installation

CLI'en **følger med i Chloros-installationsprogrammet** på alle platforme — der findes ikke en separat CLI-download. Hent installationsprogrammet fra [Download](download.md)-siden.

### Windows


Installationsprogrammet placerer CLI'en i:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

og tilføjer denne mappe til dit system `PATH` én gang (en opgradering fjerner også dubletter efter tidligere installationsprogrammer) — **åbn en ny terminal**efter installationen, så den opdaterede `PATH` bliver hentet. Installationsprogrammet placerer også startskripter (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) i installationsroden samt en**Chloros CLI** genvej i Start-menuen, som hver især åbner en terminal med `chloros-cli` klar til brug.

### Linux


Installer `.deb` til din arkitektur med `apt` (ikke `dpkg -i`, som springer de anbefalede krav fra Python SDK over):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Dette installerer `chloros-cli` til `/usr/bin/chloros-cli` (allerede på `PATH`), backend- og CLI-programmerne udpakket i `/usr/lib/chloros/backend/` og `/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` er et startprogram, der starter backend-programmet), Arena SDK-runtime, som backend-programmet har brug for til LATTICE-kameraer, samt Python-SDK til systemet `python3`. Se [Linux-installation](linux/linux-installation.md) for yderligere oplysninger.

### Bekræft

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Login og licensering

Adgang til CLI (og Python SDK) kræver et **betalt Chloros+-abonnement**— alle betalte niveauer har det; det gratis niveau har det ikke. Begrænsningen håndhæves**på serversiden** af backend'et, ikke af CLI-binærfilen: et opkald fra en bruger, der er logget ud, afvises med `401 AUTH_REQUIRED`, og et opkald fra en indlogget bruger på det gratis abonnement med fejlkoden `403 PLAN_UPGRADE_REQUIRED`, uanset om det kommer fra `chloros-cli`, SDK'en eller en hjemmelavet HTTP-klient. Opgrader på [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Log ind **én gang pr. maskine**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Adgangskoder med specialtegn**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` bliver forvrænget af shellen (CLI'en registrerer dette ved en 401-fejl og forsøger automatisk igen, men enkelt anførselstegn undgår problemet helt).
{% endhint %}

Sessionen gemmes i cachen som `~/.chloros/user_session.json` og fungerer fortsat offline i abonnementets afdragsfri periode (30 dage for månedsabonnementer, indtil udløb for årsabonnementer). `chloros-cli status` fungerer selv uden et betalt abonnement, så årsagen til et afslag altid er synlig.

{% hint style="danger" %}
**Planlægger du headless-opgaver? Log ind først.**En kommando til opstart af backend (`process`, `status`, `export-status`, …) der køres uden**en cachelagret session**, fejler ikke hurtigt — den går over i en interaktiv `Email:` / `Password:`-prompt på stdin. Et uovervåget cron-job eller CI-trin vil derfor**hænge og vente på indtastning**. Kør `chloros-cli login EMAIL 'PASSWORD'` én gang på maskinen, før du planlægger noget.
{% endhint %}

***

## Din første behandlingskørsel

Peg `process` mod en mappe med optagelser — den registrerer automatisk Survey3 (`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` eller en blanding:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Fremskridtsstrømme vises live pr. pipeline-tråd (detektering, analyse, behandling, eksport), og en vellykket kørsel afsluttes med en rapport over, hvor mange billedprodukter der blev skrevet: `Image products written: N`, efterfulgt af `(plus M clamp masks)`, når kørslen også har skrevet [clamp-masker](output-image-formats.md#clamp-masks) (disse tæller ikke som produkter).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Hvis et Chloros-backend allerede kører (f.eks. desktop-appens), bruger `process` det og åbner sit projekt i det. Det afviser, mens appen er åben på et projekt, som appen har åbnet: kør fra appen, eller luk det pågældende projekt. Hvis en tidligere kørsel stadig er ved at afsluttes, venter den op til 15 sekunder på den, før den afviser. `--restart` tvinger en ny backend. På Linux genstartes en backend, der kører som en systemd-tjeneste, aldrig: `process` bruger den, som den er, og hvor den ellers ville genstarte den (`--restart` eller en import, der ikke fandt noget), udskriver den `sudo systemctl restart <unit>` og afslutter `1`.

### Hvor uddataene havner

`process` skriver til en **projektmappe**, ikke til din indgangsmappe:

* Uden `-o`: Projektet oprettes under din standardprojektmappe (delt med GUI’en; administrer den med `get-project-folder` / `set-project-folder`; hvis der ikke er angivet nogen, `Documents/MAPIR/Chloros Projects` i din brugermappe), navngivet efter `-n/--project-name` eller et tidsstempel (`YYYYMMDD_HHMMSS`), hvis det udelades.
* Med `-o PATH`: den pågældende mappe **er** projektmappen. Hvis den allerede indeholder en `project.json`, oprettes der i stedet en `_1`/`_2`… med dette suffiks oprettes i stedet for at overskrive den eksisterende. En relativ `PATH` angives i forhold til den mappe, hvorfra du kører `chloros-cli`.

{% hint style="info" %}
**Relative stier (ændret i 1.3.0).** Hver sti, som backend'et åbner — `-o`/`--output` ved enhver kommando (herunder standardkommandoen `output` for indfangningskommandoer), `--input`, `--burst-dir`, `--profile`, `--save` og `--mask` — løses i forhold til den mappe, hvorfra du kører `chloros-cli`, så `-o captures/` skriver til `<current folder>/captures/`. I version 1.2.0 løste backend'en dem i sin egen installationsmappe, hvilket mislykkedes med fejlmeddelelsen »Adgang nægtet«. Ved en fjernbackend (`CHLOROS_BACKEND_URL`) sendes stierne, som de er indtastet, og løses på den pågældende maskine.
{% endhint %}

{% hint style="warning" %}
**Windows: Hold projektmappen kort, medmindre lange stier er aktiveret.** Når lange stier er deaktiveret i Windows (Windows-standardindstillingen), må ingen filsti være længere end 260 tegn, og produkterne ligger op til ca. 130 tegn under projektmappen. `process` kontrollerer dette, før det starter: en projektmappe, der er for dyb, afvises med **"Projektmappen er for dyb til denne computer"**, den længste projektmappe, der passer, og hvordan man aktiverer lange stier — intet behandles. Brug en kortere `-o` (eller `set-project-folder`), eller aktiver lange stier i Windows (installationsprogrammets indstilling **Aktiver lange stier i Windows**, der er markeret som standard) og genstart Chloros.
{% endhint %}

Inden for projektet er produkterne grupperet **efter kamera og derefter efter filformat**:

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

Kameramappen er `LATT-<sensor>-<lens>-F<filter>` for LATTICE (svarende til optagelsens EXIF-data `Model`) og `<model>_<filter>` (f.eks. `Survey3N_RGN`) for Survey3. Formatmappen følger `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` eller `tiff32` for `TIFF (32-bit, Percent)`.

For et multispektralt M3C-kamera kan der også forekomme en `clamp_masks/`-mappe i en formatmappe ved siden af produktmapperne (for eksempel `tiff16/clamp_masks/Reflectance_Calibrated_Images/` og `tiff32/clamp_masks/Radiance_Images/`). Den indeholder maskerne pr. pixel for strålings- og reflektansprodukter, hvor båndsepareringen har fastsat en negativ værdi til nul — se [Fastlåste masker](output-image-formats.md#clamp-masks). Ingen produktmappe indeholder nogensinde en maske.

{% hint style="info" %}
**Hvert eksporteret produkt beholder navnet på KILDE-filen.**En strålings-eksport af `capture_..._raw.tif` hedder stadig `capture_..._raw.tif` — den findes blot i `tiff32/Radiance_Images/`.**Mappen identificerer produktet, ikke filnavnet**, så brug glob-udtryk for mappen, ikke for et `*radiance*`-suffiks.
{% endhint %}

### De indstillinger, du rent faktisk vil bruge

| Flag | Standard | Hvad det gør |
| --- | --- | --- |
| `-o, --output PATH` | standardprojektmappe | Placering af projektmappen (se ovenfor). |
| `-n, --project-name NAME` | tidsstempel | Projektnavn. |
| `--format FMT` | `TIFF (16-bit)` | En af følgende: `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros opskalerer ikke: på alle 8-bit-billeder skrives et format med større dybde som `TIFF (8-bit)`, og det angives én gang. |
| `--indices NAME [NAME ...]` | ingen | Vegetationsindekser, der skal eksporteres (se [Vegetationsindekser](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neuralt debayer, langsommere, højeste kvalitet (Chloros+, kræver en CUDA-GPU: uden en sådan bruger kørslen `standard`; afvist for LATTICE-optagelser, som altid får `standard`). |
| `--vignette / --no-vignette` | aktiveret | Vignettekorrektion. |
| `--reflectance / --no-reflectance` | aktiveret | Reflektanskalibrering; for LATTICE er dette også kontakten til reflektansproduktet. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Tving indgangsstedet i pipelinen for LATTICE-TIFF-filer. |

For alt andet — finjustering af måldetektion, PPK, eksponeringspins, flag til array-justering — se afsnittet [`process` i CLI-referencen](reference/cli-reference.md).

***

## Valg af, hvad der skal eksporteres (LATTICE-produkter)

LATTICE-behandlingen fordeles på **alle relevante produkter i én gennemløb**. Fire produkt-specifikke indstillinger er alle**aktiveret som standard**; brug formularen `--no-` til at deaktivere en:

| Indstilling | Produkt |
| --- | --- |
| `--debayered` | Lineær demosaik → `Debayered_Images/`: kalibreringspakkens mørkeforskydning fjernes, efterfulgt af en kantbevidst demosaik uden linearitetskorrektion, flat field eller unmix — den samme fil, som en optagelse gemmer på sit `debayered`-niveau. Bevares i sensorens oprindelige orientering (registreret i XMP). |
| `--preview` | Vis forhåndsvisning (hvidbalance + gamma; falskfarveudvidelse til multispektral) → `Preview_Images/` |
| `--radiance` | float32 strålingsintensitet, W/m²/sr/nm → `Radiance_Images/` (altid `tiff32/`) |
| `--reflectance` | uint16 reflektans, Pix4D-klar → `Reflectance_Calibrated_Images/` |

RGB-masterkameraer udsender altid kun debayered + preview — stråling/refleksion pr. bånd er ikke relevant for en bredbåndssensor, så disse indstillinger har ingen effekt for dem. Survey3 `.raw` ignorerer indstillingerne og følger standardvejen for refleksion/mål.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (standard `auto`) vælger reflektansreferencen: `auto` gør et QA-godkendt [kalibreringsmål](calibration-targets.md) inden for billedrammen til den absolutte reference og falder tilbage til DAQ-lyssensorens nedadgående lysdelingsværdi (ρ = π·L/E), når der ikke er noget mål til stede (et mål, der består kvalitetskontrol med et flag, konkurrerer med DAQ om den målte score); `target` er streng (ingen DAQ-erstatning); `daq` er DAQ-autoritativ. Målscanninger målt pr. enhed kan leveres med `--target-reflectance-dir`.

{% hint style="info" %}
**Aflæsning af reflektanspixels:**DN-værdien, der betyder ρ = 1,0, er**pr. kilde** — LATTICE-filer indsætter `Chloros:PixelScale=32768` i XMP; Survey3-filer bruger 65535 og har ikke noget `Chloros:PixelScale`-tag (de har dog andre `Chloros:*`-behandlings-tags). Læs tagget og divider med det i stedet for at antage en konstant. Detaljer og det ene bevidste undtagelsestilfælde uden skala findes i [CLI-referencen](reference/cli-reference.md).
{% endhint %}

**Behandlingen starter altid fra `raw`.** Afledte produkter (eksport af debayering/radiance/reflektans) føres aldrig tilbage gennem pipelinen — at importere dem igen og behandle dem ville medføre dobbelt anvendelse af kalibreringsberegningerne, så Chloros springer dem over og angiver dette. `--input-level` er den bevidste nødudgang, når du virkelig har brug for at tvinge et indgangssted.

***

## Når en kørsel mislykkes

Fra og med version 1.2.0 fejler `process` tydeligt i stedet for at »lykkes« uden at vise noget:

* En kørsel, der **anmodede om produkter, men ikke skrev nogen**— kun `project.json` og `calibration_data.json` — udskriver `Processing finished but wrote no image products.` og**afsluttes med en værdi forskellig fra nul**, så scripts kan registrere det. De sædvanlige årsager: indlæsningsmappen blev ikke genkendt som en optagelse (tjek layoutet og `--input-level`), eller alle de anmodede produkter var uanvendelige for de pågældende kameraer (f.eks. at anmode om radiance/reflektans fra kameraer, der kun tager RGB-billeder).
* En **bevidst kørsel, der kun genererer metadata** (alle produkter slået fra, ingen `--indices`) betragtes stadig som en succes — en tom billedudgang er det korrekte resultat i dette tilfælde.
* Kør kørslen igen med `--verbose`, og tjek backend-loggen for linjer med `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, som forklarer, hvilke kameraer der er blevet sprunget over.
* På Windows, hvor lange stier er deaktiveret, afvises en projektmappe, der er for dyb i forhold til stibegrænsningen på 260 tegn, **inden** behandlingen starter, og den længste projektmappe, der passer, anvendes i stedet (se [Hvor outputfilerne gemmes](#where-the-outputs-land)).
* Optagelseskommandoer (`lattice capture`, `lattice array-capture` og de øvrige optagelsesfamilier) angiver navnet på hver fil, de ikke kunne skrive, og afslutter med `1`; gentagne optagelser stopper, når drevet er fuldt.

Afslutningskoder: `0` succes · `1` generisk fejl · `2` argumentfejl · `130` afbrudt af Ctrl+C.

***

## Vegetationsindekser

Kør `--indices` med et eller flere forudindstillede navne; hvert indeks placeres i sin egen `<INDEX>_Index_Images/`-mappe:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

De 22 forudindstillede navne, som `process --indices` accepterer:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Et navn, som projektet endnu ikke indeholder, tilføjes til det. Et navn, der ikke er en forudindstilling, eller som kameraernes bånd ikke kan danne, springes over med en advarsel, der angiver navnet. Hvis CLI'en ikke kan læse projektets aktuelle indeksliste, lader den listen være uændret og viser en advarsel i stedet for at erstatte den.

{% hint style="warning" %}
**Der findes tre indekslister — bland dem ikke sammen.**GUI'ens rullemenu »Projektindstillinger« indeholder 27 formler (tilføjer `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — disse fem findes kun i GUI’en og gælder**ikke** for `--indices`). Live/offline-kommandoen `lattice index --preset` bruger sin egen separate liste med 22 forudindstillinger. Formler og båndberegninger er dokumenteret i [Formler til multispektrale indekser](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## DAQ-lyssensorer: En hurtig introduktion

`daq pool-*`-familien styrer MAPIR-DAQ-spektralsensorer (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet) gennem backendens permanente pool — GUI, CLI og SDK deler alle ét live-handle. **`pool-*` er den understøttede DAQ-sti i den medfølgende CLI**; andre `daq`-underkommandoer, som du muligvis ser henvist til, er en MAPIR-intern overflade, der kun er til kildebrug, og afsluttes med en eksplicit fejlmeddelelse, der henviser dig til `pool-*`.

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

`pool-record` uden `--duration` kører indtil `pool-record --stop`; standardudskriftsmappen er `~/Documents/DAQ Live View/` **på backend-maskinen**. Cap-korrektionsprofilen vælges ved opkobling (`--cap-id`, backend-standard `sunshine_cosine`) og kan skiftes under kørsel med `pool-set-cap` — cap-profiler og sensorens kalibrerede område er beskrevet i DAQ-kapitlerne i denne manual.

{% hint style="warning" %}
**DAQ-E på en vært med flere netværkskort:** Den første `pool-connect --eth`-autodetektering efter opstart kan mislykkes, selv med en velfungerende sensor. `--eth-host <ip-or-hostname>` er den pålidelige variant — brug den, når autodetekteringen ikke giver noget resultat.
{% endhint %}

***

## LATTICE-kameraer, PTP og projektautomatisering

`lattice`-familien (over 45 underkommandoer) dækker LATTICE-kamerahåndtering fra start til slut: opdagelse, enkeltoptagelser, vedvarende synkroniserede arrays med GUI’ens »smart-prep connect«-flow, live-forhåndsvisning i browseren, justering, indeksberegninger og værts-NIC-diagnostik. Et eksempel:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Sideløbende hermed: `chloros-cli time-sync` rapporterer om den PTP-grandmaster, som Chloros-værten kører (LATTICE-kameraer og DAQ-E-sensorer fungerer som slaver til denne for at sikre tidsstempler på tværs af enheder), og `chloros-cli project` åbner et gemt Chloros-projekt og styrer dets kameraer, arrays og sensorer uden brug af en skærm — herunder scriptbaserede YAML-optagelsesopskrifter.

Disse tre familier (`lattice`, `project`, `daq pool-*`) er også de eneste, der understøtter `CHLOROS_BACKEND_URL` til styring af et **fjerntliggende** backend; kernekommandoerne er altid rettet mod den lokale maskine.

Fuldstændige gennemgange findes i LATTICE-kapitlerne i denne manual; alle flag er beskrevet i [CLI-referencen](reference/cli-reference.md).

***

## Fejlfinding: Top 5

| Symptom | Løsning |
| --- | --- |
| `Login required`, eller et planlagt job hænger fast ved en `Email:`-prompt | Kør `chloros-cli login EMAIL 'PASSWORD'` én gang på denne maskine — kommandoer uden en cachelagret session kører interaktivt i stedet for at fejle hurtigt. |
| `backend unreachable` | Start Chloros-desktopappen, eller start backend selv: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` på Windows (mappen `backend` i din installationsmappe; den findes ikke på `PATH`), `chloros-backend` på Linux. `chloros-cli backend status` viser, om der kører en; `lattice`/`project`/`daq pool-*` starter aldrig en. Hvis du peger dem mod en fjernbackend, skal du kontrollere `CHLOROS_BACKEND_URL`. |
| Array-forbindelse blokeret: `FRAMES WILL DROP` / `Reduce ROI to enable` | Værts-NIC's modtagelsesring er nulstillet til standardindstillingerne — den hyppigste årsag til, at et system, der tidligere fungerede, nægter at oprette forbindelse, typisk efter en opdatering af NIC-driveren. Kør `chloros-cli lattice network --fix` fra en **terminal med administratorrettigheder** (eller indstil `ReceiveBufferLen=256`, `PendingReceives=64`); se afsnittet *Opsætning og finjustering af værts-NIC* i referencen. |
| `daq`-underkommandoen afbrydes: "kræver det fulde DAQ-pakke…" | Brug kommandoerne `daq pool-*`, som styrer den samme sensor via backend: tilslutning, streaming, optagelse, valg af cap, og for en DAQ-E dens netværksindstillinger, genstart og firmwareopdateringer (eller `chloros_sdk.connect_daq_sensor()` fra Python). |
| Jetson viser en swap-advarsel før store mapper | Tilføj filbaseret swap — CLI'en viser de nøjagtige `fallocate`/`swapon`-kommandoer, der skal køres. |

***

## Få hjælp

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Alle flag, alle underkommandoer:** [CLI-reference](reference/cli-reference.md)
* **Python-ækvivalent:** [Python SDK](api-python-sdk.md) og [SDK Reference](reference/sdk-reference.md)
* **Support:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)

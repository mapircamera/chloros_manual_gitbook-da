# Installation på Linux

Chloros distribueres til Linux som `.deb`-pakker, der installerer CLI, backend-serveren og Python-SDK’et (for systemet `python3`, plus et wheel, der passer til versionen, til virtualenvs).

Pakkefilenavnene angiver version og arkitektur: `chloros_1.3.0_amd64.deb` til x86_64, og `chloros_1.3.0_arm64_jp6.deb` for arm64 (NVIDIA Jetson, JetPack 6). Erstat med den fil, du rent faktisk har downloadet, i kommandoerne nedenfor.

{% hint style="info" %}
**Nyt i 1.3.0: Programmerne leveres udpakket.** Backend og CLI installeres som mapper, `/usr/lib/chloros/backend/` og `/usr/lib/chloros/cli/`, i stedet for som enkeltstående selvudpakkende filer. De udpakker sig ikke længere til en midlertidig mappe ved hver opstart, så de starter meget hurtigere (på x86_64 brugte backendet tidligere ca. 100 sekunder på at udpakke ved hver opstart) og har ikke længere brug for flere GB ledig plads i `/tmp`. AMD64-versionen `.deb` er også ca. halvt så stor som før.
{% endhint %}

***

## Linux amd64 (x86_64)

### Systemkrav

| Krav | Minimum | Anbefalet |
| --- | --- | --- |
| **Distribution** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Processor** | x86_64 (Intel/AMD) | Intel Core i7 eller bedre |
| **Hukommelse (RAM)** | 8 GB | 16 GB eller mere |
| **Grafikkort** | Intet (CPU-behandling) | NVIDIA GPU med 4 GB+ VRAM (12 GB+ låser `GPU_PARALLEL` op, 7 GB+ holder Texture Aware ude af single-image-stien) |
| **Lagringsplads** | 2 GB ledig plads | SSD med 10 GB+ ledig plads |
| **Python** | Python 3.7+ (til SDK'en) | Python 3.10+ |

> **Ubuntu 20.04 og Debian 11 understøttes ikke.** Afhængighedslisten for `.deb` er
> afledt af det, som Chloros-backend'et rent faktisk linker til, og det inkluderer en
> `libc6`, der er nyere end den glibc 2.31, som Focal og Bullseye leveres med, så `apt` afviser
> installationen direkte i stedet for at lade den mislykkes senere under kørsel.

### Installation

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Brug `apt install ./…`, ikke `dpkg -i`.** `apt` løser pakkens afhængigheder og installerer også det, den anbefaler: kravene til Python-SDK'en, `python3-numpy` og `python3-requests`. `dpkg -i` installerer ingen af dem. CLI'en og backend'en er selvstændige og kører uden dem, men `import chloros_sdk` har brug for dem; installationsudskriften angiver eventuelle manglende komponenter. Behold `./`, så `apt` installerer filen i stedet for at søge efter en pakke med det navn. Pakken deklarerer også `libgl1`, som den medfølgende OpenCV indlæser, og som en minimal serverimage muligvis mangler.
{% endhint %}

Kontroller installationen:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Systemkrav

| Krav | Minimum | Anbefalet |
| --- | --- | --- |
| **Platform** | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **JetPack** | JetPack 6.x | Seneste JetPack 6 |
| **Hukommelse (RAM)** | 8 GB (delt mellem GPU og CPU) | 16 GB+ delt (12 GB+ er tærsklen for parallelbehandling) |
| **Lagringsplads** | 2 GB ledig plads | NVMe SSD med 10 GB+ ledig plads |
| **Python** | Python 3.7+ (til SDK'en) | Python 3.10+ |

### Installation

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Samme layout som amd64 `.deb` (Python SDK, modtagebuffere og backend-tjenester inkluderet), til Jetson Orin / Orin NX / Orin Nano på **JetPack 6** (L4T R36). Dets PyTorch er JetPack 6's CUDA-build til Orin, så GPU-behandling, inklusive Texture Aware-debayer, kører på Jetsons GPU. Pakken indeholder ikke CUDA i sig selv: den bruger de CUDA- og cuDNN-biblioteker, som JetPack 6 installerer (`sudo apt install nvidia-jetpack` på et image, der mangler dem). Hvor der ikke kan bruges en CUDA-GPU, kører behandlingen på CPU'en, og `--debayer texture-aware` bruger Standard-debayeren (backend'et logger en `[DEBAYER] … needs a CUDA GPU …`-linje, der forklarer hvorfor). Se [NVIDIA Jetson-vejledningen](nvidia-jetson-guide.md) for oplysninger om Jetsons hukommelse, termiske forhold og adfærd ved feltinstallation.

***

## Installation af Python-SDK (alle Linux-versioner)**`.deb` installerer SDK'en for dig.** Begge pakker (amd64 og Jetson) installerer det til systemet `python3`, så `import chloros_sdk` fungerer umiddelbart efter installationen, uden brug af pip og uden netværk. Dets pakker (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` og projektindlæserne) placeres i `/usr/lib/python3/dist-packages`, som tilhører pakken: en opgradering erstatter dem, og fjernelse af pakken fjerner dem. Versionen stemmer altid overens med den CLI og det backend, du har installeret.

SDK’s egne krav, `python3-numpy` og `python3-requests`, er **anbefalet, ikke påkrævet** af pakken: `sudo apt install ./chloros_<version>_<arch>.deb` installerer dem, mens `dpkg -i` ikke gør det. Installationsudskriften angiver den SDK-version, som `python3` importerer, eventuelle manglende krav samt eventuelle `chloros_sdk`-filer andre steder på `python3`'s sti, der overskriver pakkens kopi (f.eks. en, der tidligere er installeret med `pip install --user`).**I et virtualenv**, som ikke har adgang til systemets pakker, skal du installere det medfølgende wheel i det (helt offline):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**Fra PyPI** — kun til en maskine uden en Chloros-pakke installeret, hvis SDK-miljø driver et backend på en anden maskine (se [Remote-Backend Mode](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Valgfrie ekstrafunktioner til en pip-installation (i et virtualenv eller fra PyPI):

| Ekstra | Kommando | Tilføjer |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` til streaming af fremskridt i realtid |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` til BLE (DAQ-M)-transport |

Bekræft SDK'en:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Python-SDK'et kommunikerer med det backend, som `.deb` installerer via et lokalt HTTP-API (`http://127.0.0.1:5000`). `ChlorosLocal` og forbindelseshjælperne starter den automatisk, når det er nødvendigt; et gemt projekts `connect_all()` gør det aldrig, så på en headless-maskine skal du køre backend'en som en [tjeneste](#always-on-ptp-for-headless-hosts). Brug altid den bogstavelige IPv4-adresse i stedet for `localhost` — `localhost` kan omdirigeres til `::1` og koste cirka to sekunder pr. anmodning.
{% endhint %}

***

## Første opsætning

### 1. Log ind

Adgang til CLI og SDK kræver et betalt Chloros+-abonnement (**Copper** eller højere), hvilket håndhæves på serversiden: en bruger, der er logget ud, får `401 AUTH_REQUIRED`, og en bruger på gratisabonnementet (Iron) får `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Loginoplysningerne gemmes i `~/.chloros/user_session.json`.

{% hint style="info" %}
**Opgraderinger bevarer dit login; fjernelse sletter det.** Hvis du installerer en nyere version af `.deb` oven på en eksisterende installation (herunder via `chloros-cli update`), bevares `~/.chloros/user_session.json` og den cachelagrede licens, så headless CLI-/SDK-opgaver fortsat fungerer. Hvis du fjerner pakken (`sudo apt remove chloros`), slettes den gemte login- og licenscache for alle brugere på maskinen, så en ny installation kræver `chloros-cli login` igen.
{% endhint %}

### 2. Kontroller din licensstatus

```bash
chloros-cli status
```

`chloros-cli status` fungerer på alle niveauer (inklusive gratis), så du altid kan se, hvorfor adgangen er eller ikke er tilgængelig.

### 3. Kør systemdiagnostik

```bash
chloros-cli selftest
```

Syv kontroller kører i rækkefølge, og kommandoen afsluttes med en værdi forskellig fra nul, hvis en af dem mislykkes:

| # | Kontrol | Hvad den beviser |
| --- | --- | --- |
| 1 | **Version** | CLI'en rapporterer sin version (`v1.3.0`). |
| 2 | **Port tilgængelig** | Port 5000 er ledig, *eller* der er allerede svaret fra en velfungerende Chloros-backend (hvilket tæller som bestået). |
| 3 | **Opstart af backend** | Backend-binærfilen startes. |
| 4 | **Test af API (`/api/test`)** | Backend svarer med `status: ok`. |
| 5 | **Systemoplysninger** | Udskriver `GPU: <name>, CUDA: <bool>, PyTorch: <version>` fra `/api/system-info`. |
| 6 | **Denoiser-modeller** | Finder `*.pth.enc`-modeller (på Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + støjfjerner**| Texture Aware kan faktisk bruges — kræver CUDA**og** mindst én modelfil. |

Kørslen afsluttes med `N/7 checks passed`, hvor eventuelle fejl vises med navn.

### 4. Behandl dit første datasæt

```bash
chloros-cli process ~/datasets/flight001
```

***

## Filer og mapper

### Brugerbaseret

Chloros gemmer sine legitimationsoplysninger og CLI-konfiguration i en enkelt platformsuafhængig mappe, **`~/.chloros/`** (på Windows, `%USERPROFILE%\.chloros\`). To Linux-specifikke cacher følger i stedet XDG-konventionerne — disse bruger `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`, når de er angivet.

| Sti | Formål |
| --- | --- |
| `~/.chloros/user_session.json` | Cache for loginsession skrevet af `chloros-cli login` (bevares ved pakkeopgraderinger; ryddes, når pakken fjernes) |
| `~/.chloros/working_directory.txt` | Overskrivning af standardprojektmappe (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Sprogindstilling for CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Sprogindstilling, der deles med Windows-brugergrænsefladen — en `language` her har forrang frem for `cli_language.json` |
| `~/.chloros/update_cache.json` | En times cache til Linux/Jetson-opdateringskontrol ved opstart |
| `~/.chloros/backend.log` | Backend-log, når backend blev startet via CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Cachelagrede LATTICE-kalibreringspakker pr. kamera, indekseret efter serienummer og bundle-hash |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Valgfri brugeroverridelser for DAQ-kapacitetskorrektionsprofiler |
| `~/.config/chloros/system_config.json` | Cachelagret hardwareprofil fra Dynamic Compute Adaptation — slet den for at tvinge en ny hardwaredetektering |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Backend-serverlogfiler, én fil pr. opstart |
| `~/Documents/MAPIR/Chloros Projects/` | Standardprojektmappe, når der ikke er angivet nogen overstyring |

### Systemomfattende

| Sti | Formål |
| --- | --- |
| `/usr/bin/chloros-cli` | Wrapper-script — indstiller `LD_LIBRARY_PATH` for de medfølgende native biblioteker (og `TMPDIR` til `/mnt/ssd/tmp`, når den pågældende mappe findes), hvorefter den kører den egentlige binærfil |
| `/usr/bin/chloros-backend` | Wrapper-script — det samme for backend'en, som det markerer som en installeret produktionsversion |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | Den kompilerede CLI og backend, installeret udpakket (nyt i 1.3.0); programmerne er `cli/chloros-cli` og `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | To-linjers launcher-scripts, der starter de ovenstående egentlige programmer — opbevares på disse stier, fordi SDK'et, CLI'et og ældre scripts søger efter dem her |
| `/usr/lib/chloros/arena_runtime/` | Arena SDK-runtime, der kræves af LATTICE-kameraer |
| `/usr/lib/chloros/models/*.pth.enc` | Krypterede støjfjernelsesmodeller, der bruges af Texture Aware-debayer |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (og `lattice_sdk`, `lattice_cam`, `arena_api`, projektindlæsningsprogrammerne) | Python-SDK'en, installeret til systemet `python3` og ejet af pakken |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python-SDK-wheel, der matcher netop denne build, til en virtualenv |
| `/usr/lib/chloros/exiftool` | Medfølgende ExifTool-starter (der kun er symlinket til `/usr/local/bin/exiftool`, hvis der ikke findes et system-exiftool). Fra og med version 1.3.0 leveres ExifTool-scriptet med sit eget Perl-bibliotek i `/usr/lib/chloros/exiftool-dist/`, så det kører uden systempakken `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Konfiguration af opdateringskanal, der læses af `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Indstiller `net.ipv4.ip_unprivileged_port_start = 319`, så backend'en kan binde PTP-portene uden root |
| `/etc/sysctl.d/60-chloros-gige.conf` | Øger kernens socket-modtagelsesbuffere (`net.core.rmem_max`, `net.core.rmem_default`) til 32 MB for LATTICE-kamerastrømme og anvender dem ved installationen (en værdi, der allerede er indstillet højere, forbliver uændret). Ved kernens standardindstilling droppes kamerapakker i kernen, og et array mister billeder og billedhastighed; `chloros-cli lattice network` rapporterer, når bufferne er lave. Se [Linux-værter — socket-modtagelsesbuffere](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Peger den dynamiske loader mod `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Giver den loggede bruger adgang til DAQ-U USB-serielbroen (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Tilvalg af en altid-tændt backend-tjeneste, der kører backenden som den bruger, du angiver (installeret, **ikke aktiveret**) — den, der skal aktiveres, se [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Den samme tjeneste, der kører backend som **root**(installeret,**ikke aktiveret**), opretholdes, så en maskine, der har aktiveret den, fortsætter med at køre |
| `/var/log/chloros/` | Root-tjenestens detaljerede log, som kan læses af gruppen `adm` (slettes, når pakken fjernes) |
| `/usr/share/applications/chloros-cli.desktop` | "Chloros CLI"-post i applikationsmenuen, der åbner en terminal |

**Firewall (nyt i 1.3.0).** Hvis `ufw` eller `firewalld` er aktivt ved installationen, åbner pakken UDP-porte `5002` (DAQ-E-data), `5353` (mDNS-opdagelse), `319` og `320` (PTP), og fjerner disse regler, når pakken fjernes. Hvis der er installeret en firewall, men den ikke er aktiv, tilføjer den intet og viser de porte, der skal tillades, hvis du senere vælger at aktivere den. Den åbner ingen TCP-porte: en DAQ-E-firmwareopdatering, hvor sensoren opretter forbindelse til denne computer via TCP for at hente billedet, mislykkes med `… device never connected back on TCP <port> …`, mens en aktiv firewall blokerer denne forbindelse.

## Placering af backend-eksekverbar fil

CLI'en og SDK'en registrerer automatisk backend'en:

| Komponent | Sti |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (et startprogram til `/usr/lib/chloros/backend/chloros-backend`) |

Overskriv stien til backend med CLI-flagget `--backend-exe` eller SDK-konstruktorparameteren `backend_exe`, og porten med `--port` (standard `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` peger på **`lattice`**,**`project`**og**`daq pool-*`**-kommandofamilierne på en fjernbackend. Kernekommandoerne (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ignorerer den bevidst og retter sig altid mod `http://127.0.0.1:<port>`.
{% endhint %}

***

## LATTICE-kameraer og DAQ-lyssensorer på Linux

Alle kommandofamilierne under »live-hardware« fungerer på Linux (amd64 og Jetson):

* **`chloros-cli lattice`** — finder, opretter forbindelse til, konfigurerer og indsamler data fra LATTICE-kameraer og synkroniserede arrays. `.deb` samler den Arena-SDK-runtime, de kræver, og registrerer den hos den dynamiske loader.
* **`chloros-cli daq pool-*`** — tilslut DAQ-U/M/E-lyssensorer via backend-puljen, stream kalibrerede spektre, optag `.daq`-filer og udskift cap-profiler, og for en DAQ-E kan du ændre dens netværksindstillinger, genstarte den og opdatere dens firmware. `chloros-cli daq discover` scanner efter sensorer, du kan tilslutte, og `chloros-cli daq --help` viser en liste over `pool-*`-underkommandoer.
* **`chloros-cli project`** — kør et gemt projekt (dets kameraer, sensorer og behandlingsindstillinger) uden brugergrænseflade.
* **`chloros-cli time-sync`** — inspicer den PTP-grandmaster, som Chloros-backendet kører for LATTICE-kameraer og DAQ-E-sensorer.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` er en forudsætning for `pool-latest`, `pool-stream`, `pool-record` og `pool-set-cap`; `pool-list` viser de ID'er, der aktuelt findes i puljen.

{% hint style="info" %}
**`--eth-host` springer opdagelsen over.** `pool-connect --eth` finder en DAQ-E via mDNS med en ARP-fallback, der fungerer fra en tom ARP-cache; ved at videregive sensorens IP-adresse eller værtsnavn med `--eth-host` oprettes der en direkte forbindelse til den.
{% endhint %}

**DAQ-U-serielle tilladelser** håndteres af den installerede udev-regel (`uaccess` + gruppen `dialout`). Hvis en sensor, der allerede var tilsluttet, forbliver utilgængelig, skal du genindlæse reglerne eller tilslutte den igen:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Se [CLI-referencen](../CLI.md) for det fulde kommandosæt.

### Always-On PTP til headless-værter

Pakken installerer to systemd-tjenester til backend, som ingen af dem er aktiveret. På en headless Jetson eller server, der skal holde PTP-tidssynkronisering kørende kontinuerligt for DAQ-E-sensorer og LATTICE-kameraer, eller hvor SDK-scripts forbinder gemte projekter (`connect_all()` starter aldrig en backend), skal du aktivere tjenesten **som din bruger**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` kører backend'en som den pågældende bruger: de data, den skriver, tilhører denne bruger, dens detaljerede log findes i brugerens `~/.cache/chloros/logs`, og den bruger brugerens `chloros-cli login` (`~/.chloros`), så log først ind én gang som den pågældende bruger. Når pakken installeres, vises denne kommando med dit brugernavn udfyldt.

Uden en tjeneste kører backend'en (og dermed PTP-grandmasteren) kun, så længe noget har startet den — en `chloros-cli`-kommando eller et SDK-script.

Enheden binder backend'en til `127.0.0.1:5000` (miljøindstillingerne `CHLOROS_HOST` / `CHLOROS_PORT` inde i enheden; tilsidesæt med `sudo systemctl edit chloros-backend@.service`) og genstarter den ved fejl efter 5 sekunder.

**Root-tjenesten.**`chloros-backend.service` kører backend'en som**root**. Den er stadig installeret, så en maskine, der har aktiveret den, fortsætter med at køre, men alt, hvad en root-backend skriver, tilhører root, så en bruger kan ikke slette sine egne optagelser; dens log findes i `/var/log/chloros` og kan læses af gruppen `adm`. For at skifte til din bruger:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**En tjenestes backend deles og stoppes aldrig af en klient.** `chloros-cli` og SDK-scripts bruger den, som den er: `process` kører på den og indlæser sit eget projekt i stedet for det sidste. Hvor CLI'en ellers ville genstarte backend'en (`--restart` eller en `process`, hvis import ikke fandt noget), navngiver den enheden, udskriver `sudo systemctl restart <unit>` og afslutter `1`, fordi et backend, der stoppes på den måde, ville forblive nede. Genstart det selv med `sudo systemctl restart chloros-backend@$USER.service`. En tjenestes backend stopper aldrig af sig selv, når den er inaktiv, og `chloros-cli backend stop` afviser den og angiver kommandoen `systemctl stop`.**Sådan får PTP sine porte.** PTP bruger UDP 319/320, som begge ligger under den normale grænse på 1024 for privilegerede porte. Pakken `postinst` skriver `/etc/sysctl.d/60-chloros-ptp.conf` med `net.ipv4.ip_unprivileged_port_start = 319`, hvilket lader backend'en binde dem, mens den kører som din bruger. Fra og med version 1.3.0 indstiller pakken ingen filrettigheder (`setcap`) på backend'en, og `libcap2-bin` er ikke længere en afhængighed: på den udpakkede backend ville en filrettighed forhindre den i at starte, så tilføj ikke selv en.

***

## Eksempler på Bash-scripts

{% hint style="info" %}
**Skriptvenlige afslutningskoder.**`chloros-cli process` afslutter med `0` ved succes og**en værdi forskellig fra nul ved fejl — herunder et kørsel, der anmodede om billedprodukter, men ikke skrev nogen** (det udskriver `Processing finished but wrote no image products.` og angiver projektmappen samt de sædvanlige årsager). Vellykkede kørsler rapporterer, hvor mange billedprodukter der blev skrevet (`Image products written: N`, plus `(plus M clamp masks)`, når der blev skrevet masker). Afslutningskoder: `0` succes, `1` fejl, `2` argumentfejl, `130` afbrudt.
{% endhint %}

### Behandle flere datasæt

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Behandle med brugerdefinerede indstillinger

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

De gyldige `--format`-værdier er nøjagtigt fire, og de indeholder mellemrum — sæt altid anførselstegn omkring dem:

| `--format`-værdi | Output-mappe |
| --- | --- |
| `TIFF (16-bit)` *(standard)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` accepterer `standard` (standard) eller `texture-aware` (Chloros+).

### Automatiseret behandling med Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Eksempel på Python-SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Fejlfinding

### CLI-værktøjet findes ikke efter installation

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Adgang nægtet

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Backend starter ikke efter tilføjelse af filrettigheder

Kør ikke `setcap` på backend. Fra og med version 1.3.0 indstiller pakken ingen filfunktioner, og PTP's porte 319/320 stammer fra indstillingen `sysctl`, der er beskrevet under [Sådan får PTP sine porte](#always-on-ptp-for-headless-hosts). En filkapacitet sætter den udpakkede backend i en sikker udførelsestilstand, hvor den ikke kan finde sine egne biblioteker, og derfor starter den ikke. Hvis du har tilføjet en, skal du fjerne den:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP starter ikke / kan ikke binde port 319

Bekræft, at grænsen for uprivilegerede porte er blevet sænket, og anvend den igen for den aktuelle opstart, hvis det ikke er tilfældet:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Kontroller derefter grandmasteren:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### `lattice`-kommandoer mislykkes med fejlmeddelelsen "LATTICE-kameradrivere ikke fundet"

Backend'et kunne ikke indlæse den LATTICE-kameraruntime, som det er installeret med. Alle `lattice`-kommandoer kører på backend'et; CLI'en selv indeholder ingen kameradrivere. Geninstaller Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): `.deb` installerer runtime-miljøet ved siden af backend'en, i `/usr/lib/chloros/arena_runtime`, og registrerer det hos den dynamiske loader. Sådan kontrolleres registreringen:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
ls /usr/lib/chloros/arena_runtime | head
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

Backend-logfilerne for den mislykkede opstart findes i `~/.cache/chloros/logs/`. Hvis port 5000 tilhører en backend-tjeneste (`systemctl status chloros-backend@$USER.service`), skal du genstarte tjenesten i stedet for at afbryde processen.

### CUDA blev ikke fundet

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` rapporterer det samme i én linje: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Manglende delte biblioteker

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Langsom opstart på systemer med SD-kort

Fra og med version 1.3.0 installeres backend og CLI'en i udpakket form, så de ikke længere udpakker sig selv til et midlertidigt bibliotek ved hver opstart. Midlertidige filer placeres automatisk i `/mnt/ssd/tmp`, hvis denne mappe findes; ellers kan du angive et hurtigt filsystem:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Opdatering af Chloros på Linux

Kommandoen `update` findes kun på Linux/Jetson. Den tjekker den version, der er offentliggjort i den opdateringskanal, der er konfigureret under `/etc/chloros/update.conf`, og tilbyder at downloade og installere den tilsvarende `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

På Linux/Jetson udfører CLI'en også en ikke-blokerende opdateringskontrol ved hver opstart (resultatet caches i en time i `~/.chloros/update_cache.json`) og viser `Update available: vX.Y.Z`, når der findes en nyere version. Dine indstillinger, projekter og login bevares efter en opdatering.

Opgradering af pakken stopper alle backend-tjenester, mens den installeres, så et kørende script mister sine kameraer og sensorer og derefter genstarter hver enkelt, der er aktiveret (et script, der er startet manuelt, men aldrig aktiveret, forbliver stoppet, ligesom efter en genstart).

## Afinstallation

```bash
sudo apt remove chloros
```

Afinstallationen stopper og deaktiverer alle backend-tjenester, gendanner standardgrænsen for ikke-privilegerede porte (1024), fjerner indstillingsfilen for modtagebufferen (de aktuelle værdier forbliver indtil genstart), symlinket til det medfølgende exiftool, Arena-loader-konfigurationen, de firewall-regler, som pakken har tilføjet, samt den Python-SDK, den har installeret, og rydder cachelagrede legitimationsoplysninger. Dine projekter og `~/.chloros/`-datafiler forbliver uændrede. `sudo apt purge chloros` sletter også `/var/log/chloros`.

***

## Næste trin

* [NVIDIA Jetson-vejledning](nvidia-jetson-guide.md) — Jetson-specifik optimering og implementering
* [CLI: Kommandolinje](../CLI.md) — CLI-vejledningen
* [API: Python SDK](../api-python-sdk.md) — SDK-vejledningen
* [CLI-reference](../reference/cli-reference.md) og [SDK-reference](../reference/sdk-reference.md) — udtømmende oversigter over kommandoer og API'er til version 1.3.0
* [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) — hvordan Chloros tilpasser sig din hardware

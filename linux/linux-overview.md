# Linux – Oversigt

Chloros 1.2.0 tilbyder indbygget Linux-understøttelse til **CLI**og**Python SDK** — headless multispektral billedbehandling samt live-styring af LATTICE-kameraer og DAQ-lyssensorer — på Linux-arbejdsstationer, servere og NVIDIA Jetson-edge-enheder.

{% hint style="info" %}
**Ingen desktop-GUI på Linux.**Chloros-desktop-GUI&#x27;en er kun tilgængelig på Windows. Linux-brugere interagerer med Chloros via [CLI](../CLI.md) og [Python SDK](../api-python-sdk.md). `.deb` tilføjer en**Chloros CLI** til din programmenu — den åbner blot en terminalemulator, der kører `chloros-cli`.
{% endhint %}

***

## Platformsunderstøttelsesoversigt

| Funktion | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Desktop-GUI** | Ja | Ikke relevant | Nej | Nej |
| **CLI** (`chloros-cli`) | Ja | Ja | Ja | Ja |
| **Python SDK** (`chloros-sdk`) | Ja | Ja | Ja | Ja |
| **Billedbehandlingspipeline** | Ja | Ja | Ja | Ja |
| **LATTICE-kamerastyring (live)** | Ja (fanen &quot;Kameraer&quot;) | Ja (`chloros-cli lattice`, SDK) | Ja | Ja |
| **DAQ-lyssensorer (live)** | Ja (fanen Lyssensorer) | Ja (`chloros-cli daq pool-*`, SDK) | Ja | Ja |
| **PTP-tidssynkronisering (værten er grandmaster)** | Ja | Ja (`chloros-cli time-sync`) | Ja | Ja |
| **GPU-acceleration (CUDA)** | Ja | Ja | Ja | Ja (JetPack 6) |
| **Teksturbevidst debayer** | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) |
| **Dynamisk beregnings tilpasning** | Ja | Ja | Ja | Ja |
| **Backend som systemtjeneste** (`chloros-backend.service`) | Nej | Nej | Ja (opt-in) | Ja (opt-in) |
| **In-place-opdatering** (`chloros-cli update`) | Nej (kør installationsprogrammet) | Nej (kør installationsprogrammet) | Ja | Ja |***

## Understøttede arkitekturer

| Arkitektur | Beskrivelse | Pakke |
| --- | --- | --- |
| **amd64 (x86_64)** | Standard desktop-/serverprocessorer (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM-processorer — NVIDIA Jetson Orin-familien | `chloros_<version>_arm64_jp6.deb` (JetPack 6-build) |

## Understøttede Linux-distributioner

* **Ubuntu 22.04 LTS eller nyere** (amd64)
* **Debian 12 eller nyere** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin-platforme)***

## Hvad Linux-brugere får

* **Chloros CLI** — den fulde kommandolinjegrænseflade til batchbehandling, automatisering og scripting
* **Chloros Python SDK** — programmatisk Python-grænseflade til forskningspipelines og brugerdefinerede værktøjer (kan installeres fra PyPI og er også inkluderet i `.deb` som et versionstilpasset wheel)
* **LATTICE-kamerastyring** — find, tilslut, konfigurer og optag fra LATTICE-kameraer og synkroniserede multikamera-opstillinger via `chloros-cli lattice` og SDK; `.deb` indeholder Arena SDK-runtime, som kameraerne kræver
* **DAQ-lyssensorstyring** — tilslut DAQ-U/M/E-sensorer, stream kalibrerede spektre og optag `.daq`-filer via `chloros-cli daq pool-*` og SDK
* **PTP-tidssynkronisering** — Chloros-backendet kører den PTP-grandmaster, som LATTICE-kameraer og DAQ-E-sensorer er slaver til; kontroller den med `chloros-cli time-sync`, og hold den kørende headless med systemd-enheden `chloros-backend.service` (se [Linux-installation](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Projektautomatisering** — kør gemte projekter i headless-tilstand med `chloros-cli project` og SDK&#x27;s `open_project`
* **GPU-acceleration** — CUDA-accelereret behandling på NVIDIA-GPU&#x27;er (desktop og Jetson)
* **Dynamisk beregningsadaptation** — automatisk hardwaredetektion og valg af behandlingsstrategi, med `CHLOROS_STRATEGY`-overstyringen som en nødudgang for eksperter
* **Alle behandlingsfunktioner** — samme pipeline som Windows: kalibrering, vignetteringskorrektion, vegetationsindekser og alle eksportformater
* **Chloros+-funktioner** — multitrådet (pipeline-baseret) behandling, Texture Aware debayer og brugerdefinerede indekser, med et betalt Chloros+-abonnement

## Hvad brugere af Linux ikke får

* **Desktop-GUI** — ingen grafisk brugergrænseflade; al interaktion foregår via CLI eller Python SDK
* **Billedfremviser** — ingen interaktiv billedfremviser, gittervisning eller kortmarkører
* **Visuel projektstyring** — projekter oprettes og styres via CLI-kommandoer og SDK-kald (selve hardwaren — kameraer, sensorer, optagelse — kan fortsat styres fuldt ud fra terminalen)***

## Licenskrav

Adgang til CLI og SDK kræver et **betalt Chloros+-abonnementsniveau — Copper eller højere**(Copper, Bronze, Silver, Gold). Det gratis**Iron**-abonnementsniveau giver ikke adgang til CLI/SDK. Begrænsningen håndhæves af backend-systemet, ikke kun af CLI:

| Situation | Svar fra backend |
| --- | --- |
| Ikke logget ind | `401` med `error_code: AUTH_REQUIRED` |
| Logget ind på det gratis Iron-niveau | `403` med `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` fungerer på alle niveauer — det er den eneste rute, der er undtaget fra gate — så årsagen til et afslag er altid synlig.

***

## Kom godt i gang med Linux

1. **Installer Chloros** — se [Installation af Linux](linux-installation.md) for installationen af `.deb`
2. **Kontroller** — `chloros-cli --version` udskriver `Chloros CLI 1.2.0`; `chloros-cli selftest` kører den 7-trins diagnosticering
3. **Installer Python og SDK** (valgfrit) — `pip install chloros-sdk`
4. **Log ind** — `chloros-cli login your@email.com 'your-password'` (én gang pr. maskine og igen efter hver pakkeopgradering)
5. **Behandl dit første datasæt** — `chloros-cli process ~/datasets/flight001`

For NVIDIA Jetson henvises til den dedikerede [NVIDIA Jetson-vejledning](nvidia-jetson-guide.md) for platformspecifik opsætning, termisk adfærd og implementering i praksis.

***

## Næste trin

* [Linux Installation](linux-installation.md) — detaljeret installation, filplaceringer og fejlfinding for amd64 og arm64
* [NVIDIA Jetson-vejledning](nvidia-jetson-guide.md) — Jetson-specifik opsætning, hukommelse og termisk adfærd samt implementering i praksis
* [CLI : Kommandolinje](../CLI.md) — vejledningen til CLI
* [API : Python SDK](../api-python-sdk.md) — vejledningen til SDK
* [CLI-reference](../reference/cli-reference.md) og [SDK-reference](../reference/sdk-reference.md) — udtømmende oversigter over kommandoer/API til version 1.2.0
* [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) — hvordan Chloros tilpasser sig din hardware

{% hint style="info" %}
**Læsning af denne manual programmatisk.** Hver side leveres også som rå Markdown på sin egen URL plus `.md` (for eksempel `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), og der er offentliggjort et indeks over hele manualen på [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}

# Linux Oversigt

Chloros 1.1.0 introducerer indbygget Linux-understøttelse af **CLI**og**Python SDK**, hvilket muliggør headless multispektral billedbehandling på Linux-arbejdsstationer, servere og NVIDIA Jetson-edge-enheder.

{% hint style="info" %}
**Ingen GUI på Linux.** Chloros Desktop GUI er kun tilgængelig på Windows. Linux-brugere interagerer med Chloros via [CLI](../CLI.md) og [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Platformsunderstøttelsesmatrix

| Funktion | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Desktop-GUI** | Ja | Ikke relevant | Nej | Nej |
| **CLI** | Ja | Ja | Ja | Ja |
| **Python SDK** | Ja | Ja | Ja | Ja |
| **GPU-acceleration (CUDA)** | Ja | Ja | Ja | Ja (JetPack 6) |
| **Teksturbevidst debayer** | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) | Ja (Chloros+) |
| **Dynamisk beregningsadaptation** | Ja | Ja | Ja | Ja |***

## Understøttede arkitekturer

| Arkitektur | Beskrivelse | Installationsmetode |
| --- | --- | --- |
| **amd64 (x86_64)** | Standard desktop-/serverprocessorer (Intel, AMD) | `.deb`-pakke |
| **arm64 (aarch64)** | ARM-baserede processorer, primært NVIDIA Jetson | `.deb`-pakke (JetPack 6) |

## Understøttede Linux-distributioner

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson-platforme)***

## Hvad Linux-brugere får

* **Chloros CLI** — Fuld kommandolinjegrænseflade til batchbehandling, automatisering og scripting
* **Chloros Python SDK** — Programmatisk Python-grænseflade (`pip install chloros-sdk`) til integration i forskningspipelines og brugerdefinerede værktøjer
* **GPU-acceleration** — CUDA-accelereret behandling på NVIDIA GPU&#x27;er (desktop og Jetson)
* **Dynamisk beregningsadaptation** — Automatisk hardwaredetektion og optimering af behandlingsstrategi
* **Alle behandlingsfunktioner** — Samme multispektrale behandlingspipeline som Windows (kalibrering, vignetteringskorrektion, vegetationsindekser, alle eksportformater)
* **Chloros+-funktioner** — Multitrådet behandling, teksturbevidst debayer, brugerdefinerede indekser (med Chloros+-licens)

## Hvad Linux-brugere ikke får

* **Desktop-GUI** — Ingen grafisk brugergrænseflade; al interaktion foregår via CLI eller Python SDK
* **Billedfremviser** — Ingen interaktiv billedfremviser, gittervisning eller kortmarkører
* **Visuel projektstyring** — Projekter styres via CLI-kommandoer og SDK-kald***

## Kom godt i gang med Linux

1. **Installer Chloros** — Se [Linux Installation](linux-installation.md) for installation af `.deb`-pakken
2. **Installer Python SDK** (valgfrit) — `pip install chloros-sdk`
3. **Aktivér din licens** — `chloros-cli login your@email.com 'password'`
4. **Behandl dit første datasæt** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetson-brugere henvises til den dedikerede [NVIDIA Jetson-vejledning](nvidia-jetson-guide.md) for platformspecifik opsætning og optimering.

***

## Næste trin

* [Linux Installation](linux-installation.md) — Detaljerede installationsvejledninger til amd64 og arm64
* [NVIDIA Jetson-vejledning](nvidia-jetson-guide.md) — Jetson-specifik opsætning, termisk styring og implementering i felten
* [CLI : Kommandolinje](../CLI.md) — Fuld CLI-reference
* [API : Python SDK](../api-python-sdk.md) — Fuld SDK-reference
* [Dynamisk beregnings tilpasning](../processing-architecture/dynamic-compute-adaptation.md) — Sådan tilpasser Chloros sig til din hardware

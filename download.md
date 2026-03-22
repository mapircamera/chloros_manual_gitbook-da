---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Download

Download den nyeste version af Chloros for at komme i gang med multispektral billedbehandling.

### Systemkrav

#### Windows

| Krav          | Minimum                                              | Anbefalet                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operativsystem** | Windows 10 (64-bit)                                  | Windows 11 (64-bit)                                  |
| **Processor**        | Intel Core i5 eller tilsvarende                          | Intel Core i7 eller bedre                              |
| **Hukommelse (RAM)**     | 8 GB                                                  | 16 GB eller mere                                         |
| **Grafikkort**    | DirectX 11-kompatibelt                                | NVIDIA GPU med 4 GB+ VRAM                            |
| **Lagringsplads**          | 6 GB ledig plads                                       | SSD med 10 GB+ ledig plads                            |
| **Skærm**          | 1920x1080                                            | 2560x1440 eller højere                                  |
| **Internet**         | Kræves til \[valgfri] Chloros+ licensaktivering | Kræves til \[valgfri] Chloros+ licensaktivering |

#### Linux amd64 (x86\_64)

| Krav       | Minimum                    | Anbefalet               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribution**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Processor**     | x86\_64 (Intel/AMD)        | Intel Core i7 eller bedre   |
| **Hukommelse (RAM)**  | 8 GB                        | 16 GB eller mere              |
| **Grafikkort** | Ingen (CPU-behandling)      | NVIDIA GPU med 4 GB+ VRAM |
| **Lagringsplads**       | 2 GB ledig plads             | SSD med 10 GB+ ledig       |
| **Python**        | Python 3.7+ (til SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Krav      | Minimum                      | Anbefalet                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platform**     | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **Hukommelse (RAM)** | 8 GB (delt GPU/CPU)         | 16 GB+ delt                    |
| **Lagringsplads**      | 2 GB ledig plads               | NVMe SSD med 10 GB+ ledig        |
| **Python**       | Python 3.7+ (til SDK)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-acceleration**: Chloros+-brugere med NVIDIA-GPU&#x27;er kan bruge CUDA-acceleration til betydeligt hurtigere behandling. Dette fungerer både på Windows (desktop-GPU&#x27;er) og Linux (desktop-GPU&#x27;er og NVIDIA Jetson). Chloros+-brugere får også multitrådet behandling for maksimal hastighed.
{% endhint %}

***

## Download Chloros

### Seneste stabile udgivelse (23. marts 2026): Version 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Download Chloros til Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Download Chloros til Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Download Chloros til Linux arm64 / Jetson (.deb)</a>

#### Windows-installationsprogram (GUI + CLI + Backend)

* **Filtype**: .exe (Windows-installationsprogram)**Installationsvejledning:**

1. Download ovenstående .exe-fil
2. Dobbeltklik på installationsprogrammet for at starte installationen
3. Følg vejledningen i installationsguiden
4. Vælg installationsmappe (standard: `C:\Program Files\[USER]\Chloros\`)
5. Afslut installationen og start Chloros eller Chloros CLI
6. Log ind med din [MAPIR Cloud Chloros+-konto](https://cloud.mapir.camera/pricing) (eller fortsæt med gratisversionen)

{% hint style="success" %}
Installationsprogrammet tilføjer automatisk `chloros-cli` til dit systems PATH for kommandolinjeadgang.
{% endhint %}

#### Linux amd64 (.deb-pakke — CLI + Backend)

* **Filtype**: .deb (Debian/Ubuntu-pakke)
* **Arkitektur**: x86\_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (.deb-pakke — CLI + Backend)

* **Filtype**: .deb (JetPack 6)
* **Arkitektur**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Se [Linux Installation](linux/linux-installation.md) for detaljerede installationsvejledninger og [NVIDIA Jetson Guide](linux/nvidia-jetson-guide.md) for Jetson-specifik vejledning.

#### Python SDK (Alle platforme)

```bash
pip install chloros-sdk
```

Se [API : Python SDK](api-python-sdk.md) for dokumentation.

{% hint style="info" %}
**Linux-brugere**: `.deb`-pakken installerer CLI og backend. Python SDK installeres separat via pip. Der er ingen GUI til Linux — al interaktion foregår via CLI eller SDK.
{% endhint %}

***

## Yderligere ressourcer

### Python SDK

For udviklere og automatiseringsworkflows skal du installere Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentation**: [API: Python SDK](api-python-sdk.md)**Krav**: Chloros skal være installeret (Windows-installationsprogram eller Linux `.deb`-pakke), Chloros+ licenslogin kræves***

## Hvad er inkluderet

### Windows-installationsprogram

* ✅ **Chloros GUI** - Grafisk brugergrænseflade med alle funktioner
* ✅ **Chloros CLI** - Kommandolinjegrænseflade (kræver Chloros+ licens)
* ✅ **Chloros Backend** - Behandlingsmotor
* ✅ **Kameraprofiler** - Forudkonfigurerede MAPIR kameraskabeloner

### Linux .deb-pakke

* ✅ **Chloros CLI** - Kommandolinjegrænseflade (kræver Chloros+-licens)
* ✅ **Chloros Backend** - Behandlingsmotor
* ✅ **Kameraprofiler** - Forudkonfigurerede MAPIR-kameraskabeloner
* ❌ Ingen GUI — Linux er kun headless CLI/SDK

### Python SDK (pip, alle platforme)

* ✅ **Chloros SDK** - Python API (kræver Chloros+-licens)***

## Opgrader til Chloros+

Få adgang til avancerede funktioner med et Chloros+-abonnement:

* 🚀 **Multitrådet behandling** - Behandl billeder parallelt
* ⚡ **GPU (CUDA)-acceleration** - Udnyt NVIDIA GPU-kraft
* 💻 **CLI-adgang** – Automatiser med kommandolinjeværktøjer
* 🐍 **Python SDK** – Programmatisk API-adgang
* 📱 **Flere enheder** - Brug på 2-10+ enheder (afhængigt af abonnement)
* **🐻 Avanceret teksturbevidst debayer-metode** - en højkvalitets kantbevidst debayer kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al debayer-støj.
* 🧮 **Brugerdefinerede formler** - Opret brugerdefinerede multispektrale indekser

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Se Chloros+-abonnementer og priser</a></p>***

## Hjælp til installation

### Fejlfinding

**Installationen mislykkes med følgende fejlmeddelelse:**

* Sørg for, at du har administratorrettigheder
* Deaktiver midlertidigt antivirussoftware
* Kontroller, at du opfylder de minimale systemkrav

**Programmet starter ikke (Windows):**

* Kontroller, at Windows 10/11 (64-bit) er installeret
* Opdater grafikkortdrivere
* Tjek Windows Hændelsesvisning for fejldetaljer
* Kontakt support med fejllogfiler

**CLI starter ikke (Linux):**

* Kontroller, at `.deb`-pakken er installeret korrekt: `dpkg -l | grep chloros`
* Kontroller tilladelser: `sudo chmod +x /usr/bin/chloros-cli`
* Kør diagnostik: `chloros-cli selftest`
* Kontroller for manglende biblioteker: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemer med licensaktivering:**

* Sørg for, at internetforbindelsen er aktiv
* Kontroller legitimationsoplysningerne på [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Kontroller, at firewallen ikke blokerer Chloros
* Se [Chloros+ Login](chloros+-login.md) for detaljerede instruktioner

### Få support

Brug for hjælp til installation eller opsætning?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Hjemmeside**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentation**: [Kom godt i gang](./)
* ❓ **FAQ**: [Ofte stillede spørgsmål](faq.md)***

## Ændringslog

<details>

<summary>Version 1.1.0 (Seneste)</summary>

**Udgivelsesdato: marts 2026**

**Nye funktioner*** **Linux-understøttelse** — Indbygget CLI og SDK til Linux amd64 (x86\_64) og arm64 (NVIDIA Jetson JetPack 6). Installer via `.deb`-pakker.
* **NVIDIA Jetson-understøttelse** — Optimeret behandling til Jetson Nano-, Orin Nano-, Orin NX- og AGX Orin-edge-enheder.
* **Dynamisk beregnings tilpasning** — Automatisk hardwaredetektering og optimering af behandlingsstrategi. Chloros tilpasser sig din hardware fra en Jetson Nano til en multi-GPU-arbejdsstation.
* **4-trådet behandlingspipeline** — Samtidige tråde til detektion, kalibrering, behandling og eksport med dynamisk GPU-hukommelsesallokering.
* **Nye CLI-kommandoer** — `selftest` (systemdiagnostik) og `update` (Linux-opdateringsstyring).
* **Nye CLI-procesflag** — `--debayer` (standard/teksturbevidst), `--indices` (angiv indekser), `--target` (søg først i målundermappen for hurtigere detektion).
* **Nye GUI-menupunkter** — Tilføj filer, Tilføj mappe og Start/Stop behandling er nu tilgængelige fra hovedmenuen.**Forbedringer**

* Automatisk detektion af platformsuafhængig backend (Windows og Linux-stier)
* Forbedret SDK `get_status()` med sporingsfunktion for fremskridt pr. tråd
* Nye SDK-undtagelser: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Termisk styring og adaptiv begrænsning til NVIDIA Jetson
* Automatisk hukommelsesstyring med OOM-fallback til flisebelagt GPU-behandling

</details>

<details>

<summary>Version 1.0.5</summary>

**Udgivelsesdato: 10. februar 2026**

**Nye funktioner*** **Teksturbevidst debayer-metode \[Kun Chloros+] -** Teksturbevidst bruger en højkvalitets kantbevidst debayer kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al debayering-støj.
* **Understøttelse af T4P-kalibreringsmål*** **Hurtigere Chloros+ GPU-behandling, bedre hukommelsesstyring**

**Fejlrettelser*** Helt nyt frontend (GUI), bør nu fungere på alle Windows-computere.

</details>

<details>

<summary>Version 1.0.4</summary>

**Udgivelsesdato: 5. januar 2026**

**Nye funktioner*** **Skift mellem billede og metadata**: Der er tilføjet en knap i filbrowseren, så man kan se det valgte billedes metadata i en tabel i stedet for i billedgitteret
* **Zoom-skyder til billedgitter**: Ny skyder i brugergrænsefladen til at justere størrelsen på miniaturerne (understøtter også CTRL + musehjul)
* **Eksportknapper til billedgitter**: Knapper i den øverste række til at skifte miniaturer fra JPG til behandlede eksporter (Mål, Refleksion, Indeks, LUT)
* **Kortfane**: Nyt interaktivt 2D-kort, der viser GPS-positionsmarkører for billeder
  * Understøtter Google Maps og ESRI-kortfliser (vælger automatisk den bedste flisetjeneste baseret på tilgængelighed af zoomniveau)
  * Forhåndsvisning af miniaturebilleder ved musemarkering på kortmarkører

**Fejlrettelser*** Forbedret understøttelse af installation af Chloros på computere, der ikke er på engelsk

</details>

<details>

<summary>Version 1.0.3</summary>

**Udgivelsesdato: 20. december 2025**

**Nye funktioner*** Første udgivelse

**Forbedringer*** Første udgivelse

**Fejlrettelser*** Første udgivelse

**Kendte problemer*** Første udgivelse

</details>***

## Licensaftale**Proprietær software** - Copyright (c) 2026 MAPIR Inc.

Uautoriseret brug, distribution eller ændring er forbudt.

**Gratis version**: Tilgængelig til personlig og kommerciel brug med begrænsninger i funktionerne**Chloros+**: Abonnementsbaseret licens til avancerede funktioner og kommerciel implementering

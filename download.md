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
| **Grafikkort**    | Kompatibelt med DirectX 11                                | NVIDIA GPU med 4 GB+ VRAM                            |
| **Lagringsplads**          | 6 GB ledig plads                                       | SSD med 10 GB eller mere ledig plads                            |
| **Skærm**          | 1920x1080                                            | 2560x1440 eller højere                                  |
| **Internet**         | Kræves til \[valgfri] Chloros+ licensaktivering | Kræves til \[valgfri] Chloros+ licensaktivering |

#### Linux amd64 (x86\_64)

| Krav       | Minimum                    | Anbefalet               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribution**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Processor**     | x86\_64 (Intel/AMD)        | Intel Core i7 eller bedre   |
| **Hukommelse (RAM)**  | 8 GB                        | 16 GB eller mere              |
| **Grafikkort** | Intet (CPU-behandling)      | NVIDIA GPU med 4 GB+ VRAM |
| **Lagringsplads**       | 2 GB ledig plads             | SSD med 10 GB+ ledig plads       |
| **Python**        | Python 3.7+ (til SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Krav      | Minimum                      | Anbefalet                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platform**     | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **Hukommelse (RAM)** | 8 GB (delt mellem GPU og CPU)         | 16 GB+ delt                    |
| **Lagringsplads**      | 2 GB ledig plads               | NVMe SSD med 10 GB+ ledig plads        |
| **Python**       | Python 3.7+ (til SDK)        | Python 3,10+                    |

{% hint style="info" %}
**GPU-acceleration**: Brugere af Chloros+ med NVIDIA-GPU&#x27;er kan benytte CUDA-acceleration til væsentligt hurtigere databehandling. Dette fungerer både på Windows (desktop-GPU&#x27;er) og Linux (desktop-GPU&#x27;er og NVIDIA Jetson). Chloros+-brugere får desuden fordel af multitrådet behandling for maksimal hastighed.
{% endhint %}

***

## Download Chloros

### Seneste stabile udgave: Version 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Download Chloros til Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Download Chloros til Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Download Chloros til Linux arm64 / Jetson (.deb)</a>

#### Windows-installationsprogram (GUI + CLI + Backend)

* **Filtype**: .exe (Windows-installationsprogram)**Installationsvejledning:**

1. Download ovenstående .exe-fil
2. Dobbeltklik på installationsprogrammet for at starte installationen
3. Følg vejledningen i installationsguiden
4. Vælg installationsmappe (standard: `C:\Program Files\MAPIR\Chloros\`)
5. Afslut installationen, og start Chloros eller Chloros CLI
6. Log ind med din [MAPIR Cloud Chloros+-konto](https://cloud.mapir.camera/pricing) (eller fortsæt med gratisversionen)

{% hint style="success" %}
Installationsprogrammet tilføjer automatisk `chloros-cli` til dit systems PATH for adgang via kommandolinjen.
{% endhint %}

#### Linux amd64 (.deb-pakke — CLI + backend)

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

Se [Linux-installation](linux/linux-installation.md) for detaljerede installationsvejledninger og [NVIDIA Jetson-vejledning](linux/nvidia-jetson-guide.md) for Jetson-specifik vejledning.

#### Python SDK (alle platforme)

Hvert installationsprogram indeholder et tilhørende `chloros_sdk`-hjul, så SDK-versionen altid passer til den installerede GUI/CLI/backend. På Windows installerer installationsprogrammet det automatisk i dit system Python; på Linux placerer `.deb` hjulet i `/usr/lib/chloros/sdk/` og viser installationskommandoen:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

For værter, der kun bruger pip (ingen Chloros-pakke installeret), findes SDK også på PyPI:

```bash
pip install chloros-sdk
```

Se [API : Python SDK](api-python-sdk.md) og [SDK-referencen](reference/sdk-reference.md) for dokumentation.

{% hint style="info" %}
**Linux-brugere**: Pakken `.deb` installerer CLI og backend. Der er ingen grafisk brugergrænseflade til Linux — al interaktion foregår via CLI eller SDK.
{% endhint %}

***

## Yderligere ressourcer

### Python SDK

Til udviklere og automatiseringsworkflows skal du installere Chloros, Python og SDK:

```bash
pip install chloros-sdk
```

**Dokumentation**: [API: Python SDK](api-python-sdk.md)**Krav**: Chloros skal være installeret (Windows-installationsprogrammet eller Linux `.deb`-pakken), Chloros+ licenslogin kræves***

## Hvad er inkluderet

### Windows-installationsprogram

* ✅ **Chloros GUI** – Grafisk brugergrænseflade med alle funktioner
* ✅ **Chloros CLI** - Kommandolinjegrænseflade (kræver Chloros+-licens)
* ✅ **Chloros Backend** – Behandlingsmotor
* ✅ **Kameraprofiler** - Forudkonfigurerede MAPIR-kameraskabeloner

### Linux .deb-pakke

* ✅ **Chloros CLI** – Kommandolinjegrænseflade (kræver Chloros+-licens)
* ✅ **Chloros Backend** – Behandlingsmotor
* ✅ **Kameraprofiler** – Forudkonfigurerede MAPIR-kameraskabeloner
* ❌ Ingen GUI — Linux er udelukkende en headless CLI/SDK

### Python SDK (pip, alle platforme)

* ✅ **Chloros SDK** – Python API (kræver Chloros+-licens)***

## Opgrader til Chloros+

Få adgang til avancerede funktioner med et Chloros+-abonnement:

* 🚀 **Multitrådet behandling** – Behandl billeder parallelt
* ⚡ **GPU (CUDA)-acceleration** – Udnyt NVIDIA GPU-kraften
* 💻 **CLI-adgang** - Automatiser med kommandolinjeværktøjer
* 🐍 **Python SDK** - Programmatisk API-adgang
* 📱 **Flere enheder** – Kan bruges på 2–10+ enheder (afhængigt af abonnement)
* **🐻 Avanceret teksturbevidst debayering-metode** – en højkvalitets, kantbevidst debayering kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al støj fra debayeringen.
* 🧮 **Brugerdefinerede formler** – Opret brugerdefinerede multispektrale indekser

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Se Chloros+-abonnementer og priser</a></p>***

## Hjælp til installation

### Fejlfinding

**Installationen mislykkes med følgende fejlmeddelelse:**

* Sørg for, at du har administratorrettigheder
* Deaktiver midlertidigt antivirusprogrammet
* Kontroller, at du opfylder de minimale systemkrav

**Programmet starter ikke (Windows):**

* Kontroller, at Windows 10/11 (64-bit) er installeret
* Opdater grafikkortdrivere
* Tjek Windows Hændelsesviseren for detaljer om fejlen
* Kontakt support med fejllogfilerne

**CLI vil ikke starte (Linux):**

* Kontroller, at `.deb`-pakken er installeret korrekt: `dpkg -l | grep chloros`
* Kontroller brugerrettigheder: `sudo chmod +x /usr/bin/chloros-cli`
* Kør diagnostik: `chloros-cli selftest`
* Kontroller, om der mangler biblioteker: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemer med licensaktivering:**

* Sørg for, at internetforbindelsen er aktiv
* Bekræft loginoplysningerne på [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Kontroller, at firewallen ikke blokerer Chloros
* Se [Chloros+ Login](chloros+-login.md) for detaljerede instruktioner

### Få support

Brug for hjælp til installation eller opsætning?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Hjemmeside**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentation**: [Kom godt i gang](./)
* ❓ **Ofte stillede spørgsmål**: [Ofte stillede spørgsmål](faq.md)***

## Softwareopdateringer

Chloros søger efter opdateringer, meddeler, når en ny version er tilgængelig, og linker til denne downloadside — du opdaterer ved at køre det nye signerede installationsprogram. Dine indstillinger og projekter bevares ved opdateringer. På Linux og Jetson tjekker `chloros-cli update` for en nyere version og tilbyder at downloade og installere den tilsvarende `.deb` (denne kommando findes kun i Linux).

***

## Ændringslog**Version 1.2.0 (seneste)**— se**Hvad er nyt i Chloros 1.2.0** på siden [Kom godt i gang](./) for den fulde liste over funktioner.

<details>

<summary>Version 1.0.5</summary>

**Udgivelsesdato: 10. februar 2026**

**Nye funktioner*** **Teksturbevidst debayer-metode \[Kun Chloros+] -** Teksturbevidst anvender en højkvalitets kantbevidst debayer kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al debayering-støj.
* **Understøttelse af T4P-kalibreringsmål*** **Hurtigere Chloros+ GPU-behandling, bedre hukommelsesstyring**

**Fejlrettelser*** Helt nyt brugergrænseflade (GUI), som nu bør fungere på alle Windows-computere.

</details>

<details>

<summary>Version 1.0.4</summary>

**Udgivelsesdato: 5. januar 2026**

**Nye funktioner*** **Skift mellem billede og metadata**: Der er tilføjet en skifteknap i filbrowseren, så man kan se det valgte billedes metadata i en tabel i stedet for i billedgitteret
* **Zoom-skyder til billedgitter**: Ny skyder i brugergrænsefladen til at justere størrelsen på miniaturerne (understøtter også CTRL + musehjul)
* **Eksportknapper til billedgitter**: Knapper i den øverste række til at skifte miniaturer fra JPG til bearbejdede eksportformater (Mål, Reflektans, Indeks, LUT)
* **Fanen Kort**: Nyt interaktivt 2D-kort, der viser GPS-positionsmarkører for billederne
  * Understøtter Google Maps og ESRI-kortfliser (vælger automatisk den bedste flisetjeneste baseret på tilgængelighed af zoomniveauer)
  * Forhåndsvisning af miniaturer ved at holde musen over kortmarkører

**Fejlrettelser*** Forbedret understøttelse af installation af Chloros på computere med andre sprog end engelsk

</details>

<details>

<summary>Version 1.0.3</summary>

**Udgivelsesdato: 20. december 2025**

**Nye funktioner*** Første udgivelse

**Forbedringer*** Første udgivelse

**Fejlrettelser*** Første udgivelse

**Kendte problemer*** Første udgivelse

</details>***

## Licensaftale**Proprietær software** – Copyright (c) 2026 MAPIR Inc.

Uautoriseret brug, distribution eller ændring er forbudt.

**Gratis version**: Tilgængelig til privat og kommerciel brug med begrænsninger i funktionerne**Chloros+**: Abonnementsbaseret licens til avancerede funktioner og kommerciel anvendelse

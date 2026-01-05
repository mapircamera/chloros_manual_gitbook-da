---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Download

Download den nyeste version af Chloros for at komme i gang med multispektral billedbehandling.

### Systemkrav

| Krav          | Minimum                         | Anbefalet                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **Operativsystem** | Windows 10 (64-bit)             | Windows 11 (64-bit)             |
| **Processor**        | Intel Core i5 eller tilsvarende     | Intel Core i7 eller bedre         |
| **Hukommelse (RAM)**     | 8 GB                             | 16 GB eller mere                    |
| **Grafikkort**    | DirectX 11-kompatibelt           | NVIDIA GPU med 4 GB+ VRAM       |
| **Lagring**          | 6 GB ledig plads                  | SSD med 10 GB+ ledig plads       |
| **Skærm**          | 1920x1080                       | 2560x1440 eller højere             |
| **Internet**         | Kræves til aktivering af licens | Kræves til aktivering af licens |

{% hint style=&quot;info&quot; %}
**GPU-acceleration**: Chloros+-brugere med NVIDIA GPU&#x27;er (4 GB+ VRAM) kan bruge CUDA-acceleration til betydeligt hurtigere behandling. Chloros+-brugere får også multithreaded behandling for maksimal hastighed.
{% endhint %}

***

## Download Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Download Chloros her</a>

### Seneste stabile udgivelse

**Chloros Installer til Windows*** **Version**: 1.0.4
* **Udgivelsesdato**: 5. januar 2026
* **Filstørrelse (download)**: 1,8 GB
* **Filstørrelse (installeret)**: 5,7 GB
* **Filtype**: .exe (Windows-installationsprogram)

#### **Installationsvejledning:**

1. Download filen `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Dobbeltklik på installationsprogrammet for at starte installationen
3. Følg vejledningen i installationsguiden
4. Vælg installationsmappe (standard: `C:\Program Files\[USER]\Chloros\`)
5. Afslut installationen, og start Chloros, Chloros (browser) eller Chloros CLI
6. Log ind med din [MAPIR Cloud Chloros+ konto](https://cloud.mapir.camera/pricing) (eller fortsæt med gratisversionen)

{% hint style=&quot;success&quot; %}
Installationsprogrammet tilføjer automatisk `chloros-cli` til din systems PATH for kommandolinjeadgang.
{% endhint %}

***

## Yderligere ressourcer

### Python SDK

For udviklere og automatiseringsworkflows skal du installere Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentation**: [API: Python SDK](api-python-sdk.md)**Krav**: Chloros Desktop skal være installeret, Chloros+ licenslogin kræves.***

## Hvad er inkluderet

Chloros-installationen inkluderer:

* ✅ **Chloros** - Grafisk interface med alle funktioner
* ✅ **Chloros (Browser)** - Webbaseret interface til systemer med lavere specifikationer
* ✅ **Chloros CLI** - Kommandolinjegrænseflade (kræver Chloros+ licens)
* ✅ **Chloros SDK** - Python API (kræver Chloros+ licens)
* ✅ **Kameraprofiler** - Forudkonfigurerede MAPIR kameraskabeloner***

## Opgrader til Chloros

Få adgang til avancerede funktioner med et Chloros+ abonnement:

* 🚀 **Multitrådet behandling** - Behandl billeder parallelt
* ⚡ **GPU (CUDA) acceleration** - Udnyt NVIDIA GPU-kraft
* 💻 **CLI-adgang** – Automatiser med kommandolinjeværktøjer
* 🐍 **Python SDK** – Programmatisk API-adgang
* 📱 **Flere enheder** – Brug på 2-10+ enheder (afhængigt af abonnement)
* 🧮 **Brugerdefinerede formler** – Opret brugerdefinerede multispektrale indekser

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Se Chloros+ Planer og priser</a></p>***

## Hjælp til installation

### Fejlfinding

**Installationen mislykkes med fejlmeddelelsen:**

* Sørg for, at du har administratorrettigheder
* Deaktiver midlertidigt antivirussoftware
* Kontroller, at du opfylder minimumssystemkravene

**Applikationen starter ikke:**

* Prøv Chloros (browser) versionen
* Kontroller, at Windows 10/11 (64-bit) er installeret
* Opdater grafikkortdrivere
* Kontroller Windows Event Viewer for detaljer om fejlen
* Kontakt support med fejllogfiler

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

<summary>Version 1.0.4</summary>

#### **Udgivelsesdato**: 5. januar 2026**Nye funktioner*** **Skift mellem billede og metadata**: Der er tilføjet en skiftefunktion i filbrowseren, så du kan se metadata for det valgte billede i en tabel i stedet for i billedgitteret.
* **Zoom-skyder til billedgitter**: Ny UI-skyder til justering af miniaturebilleder (understøtter også CTRL + musehjul)
* **Knapper til eksport af billedgitter**: Knapper i den øverste række til at skifte mellem JPG-miniaturer og behandlede eksporter (mål, reflektans, indeks, LUT)
* **Kortfane**: Nyt interaktivt 2D-kort, der viser billedets GPS-positionsmarkører
  * Understøtter Google Maps og ESRI-kortfliser (vælger automatisk den bedste flisetjeneste baseret på tilgængelighed af zoomniveau)
  * Musemarkør-miniaturevisning på kortmarkører

**Fejlrettelser*** Forbedret understøttelse af installation af Chloros på computere, der ikke er på engelsk

</details>

<details>

<summary>Version 1.0.3</summary>

#### **Udgivelsesdato**: 20. december 2025**Nye funktioner*** Første lancering

**Forbedringer*** Første lancering

**Fejlrettelser*** Første lancering

**Kendte problemer*** Første lancering

</details>***

## Licensaftale**Proprietær software** - Copyright (c) 2025 MAPIR Inc.

Uautoriseret brug, distribution eller ændring er forbudt.

**Gratis version**: Tilgængelig til personlig og kommerciel brug med begrænsede funktioner.**Chloros+**: Abonnementsbaseret licens til avancerede funktioner og kommerciel brug.

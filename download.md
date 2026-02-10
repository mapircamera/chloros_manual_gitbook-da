---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Download

Download den nyeste version af Chloros for at komme i gang med multispektral billedbehandling.

### Systemkrav

| Krav          | Minimum                                              | Anbefalet                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operativsystem** | Windows 10 (64-bit)                                  | Windows 11 (64-bit)                                  |
| **Processor**        | Intel Core i5 eller tilsvarende                          | Intel Core i7 eller bedre                              |
| **Hukommelse (RAM)**     | 8 GB                                                  | 16 GB eller mere                                         |
| **Grafikkort**    | DirectX 11-kompatibelt                                | NVIDIA GPU med 4 GB+ VRAM                            |
| **Lagring**          | 6 GB ledig plads                                       | SSD med 10 GB+ ledig plads                            |
| **Skærm**          | 1920x1080                                            | 2560x1440 eller højere                                  |
| **Internet**         | Kræves til \[valgfri] Chloros+ licensaktivering | Kræves til \[valgfri] Chloros+ licensaktivering |

{% hint style="info" %}
**GPU-acceleration**: Chloros+-brugere med NVIDIA GPU&#x27;er kan bruge CUDA-acceleration til betydeligt hurtigere behandling. Chloros+-brugere får også multithreaded-behandling for maksimal hastighed.
{% endhint %}

***

## Download Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Download Chloros her</a>

### Seneste stabile udgivelse

**Chloros-installationsprogram til Windows*** **Version**: 1.0.5
* **Udgivelsesdato**: 10. februar 2026
* **Filstørrelse (download)**: 1,6 GB
* **Filstørrelse (installeret)**: 5,7 GB
* **Filtype**: .exe (Windows-installationsprogram)

#### **Installationsvejledning:**

1. Download filen `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Dobbeltklik på installationsprogrammet for at starte installationen
3. Følg vejledningen i installationsguiden
4. Vælg installationsmappe (standard: `C:\Program Files\[USER]\Chloros\`)
5. Afslut installationen og start Chloros eller Chloros CLI
6. Log ind med din [MAPIR Cloud Chloros+ konto](https://cloud.mapir.camera/pricing) (eller fortsæt med gratis versionen)

{% hint style="success" %}
Installationsprogrammet tilføjer automatisk `chloros-cli` til dit systems PATH for kommandolinjeadgang.
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

* ✅ **Chloros** - Grafisk brugergrænseflade (GUI) med alle funktioner
* ✅ **Chloros CLI** - Kommandolinjegrænseflade (kræver Chloros+ licens)
* ✅ **Chloros SDK** - Python API (kræver Chloros+ licens)
* ✅ **Kameraprofiler** - Forudkonfigurerede MAPIR kameraskabeloner***

## Opgrader til Chloros+

Få adgang til avancerede funktioner med et Chloros+ abonnement:

* 🚀 **Multitrådet behandling** - Behandl billeder parallelt
* ⚡ **GPU (CUDA) acceleration** - Udnyt NVIDIA GPU-kraft
* 💻 **CLI-adgang** - Automatiser med kommandolinjeværktøjer
* 🐍 **Python SDK** - Programmatisk API-adgang
* 📱 **Flere enheder** - Brug på 2-10+ enheder (afhængigt af abonnement)
* **🐻 Avanceret teksturbevidst debayer-metode** – en højkvalitets kantbevidst debayer kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al debayering-støj. 
* 🧮 **Brugerdefinerede formler** – Opret brugerdefinerede multispektrale indekser

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Se Chloros+ Planer og priser</a></p>***

## Hjælp til installation

### Fejlfinding

**Installationen mislykkes med fejlmeddelelsen:**

* Sørg for, at du har administratorrettigheder
* Deaktiver midlertidigt antivirussoftware
* Kontroller, at du opfylder minimumssystemkravene

**Applikationen starter ikke:**

* Kontroller, at Windows 10/11 (64-bit) er installeret
* Opdater grafikkortdrivere
* Kontroller Windows Event Viewer for detaljer om fejlen
* Kontakt support med fejllogfiler

**Problemer med licensaktivering:**

* Sørg for, at internetforbindelsen er aktiv
* Kontroller legitimationsoplysningerne på [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Kontroller, at firewall ikke blokerer Chloros
* Se [Chloros+ Login](chloros+-login.md) for detaljerede instruktioner

### Få support

Brug for hjælp til installation eller opsætning?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Hjemmeside**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentation**: [Kom godt i gang](./)
* ❓ **FAQ**: [Ofte stillede spørgsmål](faq.md)***

## Ændringslog

<details>

<summary>Version 1.0.5</summary>

#### **Udgivelsesdato**: 10. februar 2026**Nye funktioner*** **Teksturbevidst debayer-metode \[Chloros+ Kun] -** Teksturbevidst bruger en højkvalitets kantbevidst debayer kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al debayering-støj.
* **Understøttelse af T4P-kalibreringsmål*** **Hurtigere Chloros+ GPU-behandling, bedre hukommelsesstyring**

**Fejlrettelser*** Helt nyt frontend (GUI), bør nu fungere på alle Windows-computere.

</details>

<details>

<summary>Version 1.0.4</summary>

#### **Udgivelsesdato**: 5. januar 2026**Nye funktioner*** **Skift mellem billede/metadata**: Tilføjet skift i filbrowseren for at se det valgte billedes metadata i en tabel i stedet for billedgitteret
* **Zoom-skyder til billedgitter**: Ny UI-skyder til justering af miniaturestørrelse (understøtter også CTRL + musehjul)
* **Knapper til eksport af billedgitter**: Knapper i den øverste række til at skifte miniaturer fra JPG til behandlede eksporter (mål, reflektans, indeks, LUT)
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

## Licensaftale**Proprietær software** - Copyright (c) 2026 MAPIR Inc.

Uautoriseret brug, distribution eller ændring er forbudt.

**Gratis version**: Tilgængelig til personlig og kommerciel brug med begrænsede funktioner.**Chloros+**: Abonnementsbaseret licens til avancerede funktioner og kommerciel brug.

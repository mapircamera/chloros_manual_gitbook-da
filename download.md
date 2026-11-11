---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Download

Download den nyeste version af Chloros
for at komme i gang med multispektral billedbehandling.

### Systemkrav

#### Windows


| Krav          | Minimum                                              | Anbefalet                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operativsystem** | Windows
10 (64-bit)                                  | Windows
11 (64-bit)                                  |
| **Processor**        | Intel Core i5 eller tilsvarende                          | Intel Core i7 eller bedre                              |
| **Hukommelse (RAM)**     | 8 GB                                                  | 16 GB eller mere                                         |
| **Grafikkort**    | DirectX 11-kompatibelt                                | NVIDIA GPU med 4 GB+ VRAM                            |
| **Lagringsplads**          | 6 GB ledig plads                                       | SSD med 10 GB+ ledig plads                            |
| **Skærm**          | 1920x1080                                            | 2560x1440 eller højere                                  |
| **Internet**         | Kræves til \[valgfri] aktivering af Chloros
+-licens | Kræves til \[valgfri] aktivering af Chloros
+-licens |

#### Linux
amd64 (x86\_64)

| Krav       | Minimum                    | Anbefalet               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribution**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Processor**     | x86\_64 (Intel/AMD)        | Intel Core i7 eller bedre   |
| **Hukommelse (RAM)**  | 8 GB                        | 16 GB eller mere              |
| **Grafikkort** | Intet (CPU-behandling)      | NVIDIA GPU med 4 GB+ VRAM |
| **Lagringsplads**       | 2 GB ledig plads             | SSD med 10 GB+ ledig plads       |
| **Python**        | Python
3.7+ (til SDK
)      | Python
3.10+              |

#### Linux
arm64 (NVIDIA Jetson)

| Krav      | Minimum                      | Anbefalet                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platform**     | NVIDIA Jetson med JetPack 6 | Jetson Orin NX 16 GB eller AGX Orin |
| **Hukommelse (RAM)** | 8 GB (delt mellem GPU og CPU)         | 16 GB+ delt                    |
| **Lagringsplads**      | 2 GB ledig plads               | NVMe SSD med 10 GB+ ledig plads        |
| **Python**       | Python
3.7+ (til SDK
)        | Python
3.10+                    |

{% hint style="info" %}
**GPU-acceleration**: Chloros
+-brugere med NVIDIA-GPU&#x27;er kan benytte CUDA-acceleration til betydeligt hurtigere databehandling. Dette fungerer med NVIDIA-desktop-GPU&#x27;er på Windows
og Linux
amd64 samt på NVIDIA Jetson med JetPack 6. Chloros
+-brugere får desuden fordel af multitrådet databehandling for maksimal hastighed.
{% endhint %}

***

## Download Chloros


### Seneste stabile udgave: Version 1.3.0 (11. november 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Download Chloros til Windows (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Download Chloros til Linux amd64 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Download Chloros til Linux arm64 / Jetson (.deb)</a>


#### Windows
-installationsprogram (GUI + CLI
+ Backend)

* **Filtype**: .exe (Windows
-installationsprogram)

**Installationsvejledning:**

1. Download ovenstående .exe-fil
2. Dobbeltklik på installationsprogrammet for at starte installationen
3. Følg vejledningen i installationsguiden
4. Vælg installationsmappe (standard: `C:\Program Files\Chloros\`)
5. På siden **Vælg yderligere opgaver**skal du lade begge**Systemindstillinger**-valgmuligheder være markeret (anbefales — se nedenfor)
6. Afslut installationen, og start Chloros
eller Chloros
CLI
7. Log ind med din [MAPIR
Cloud Chloros
+-konto](https://cloud.mapir.camera/pricing) (eller fortsæt med gratisversionen)

**Indstillinger for systemindstillinger** (begge markeret som standard):

| Indstilling, som den er formuleret i installationsprogrammet | Hvad den gør | Fravælg lydløs installation |
| --- | --- | --- |
| **Aktiver lange stier i Windows** (giver mulighed for projektmapper med navne på mere end 260 tegn; anbefales) | Indstiller Windows
&#x27; systemomfattende indstilling `LongPathsEnabled` til 1. Windows
leveres med denne indstilling slået fra, og i så fald må ingen filsti være længere end 260 tegn; Chloros
-produkter ligger op til ca. 130 tegn under projektmappen, så hvis lange stier er slået fra, afvises en dybere projektmappe. Træder i kraft for programmer, der startes efter installationen. Indstillingen forbliver aktiveret efter en afinstallation, da andre programmer muligvis er afhængige af den. | `/MERGETASKS="!longpaths"` |
| **Undtag Chloros
-backend fra Microsoft Defender-scanning** (meget hurtigere kameraoptagelse; anbefales) | Tilføjer en undtagelse for Microsoft Defender-processen for `chloros-backend.exe` i installationsmappen, så realtidsbeskyttelsen ikke scanner hver eneste fil, som backend&#x27;en skriver, uanset hvor den skriver den. Ved høje optagelseshastigheder udgør scanningen ca. 30 % af optagelseshastigheden (77,7 mod 52,7 optagelser pr. sekund ved kontinuerlig optagelse med 9 kameraer). Ingen andre programmer undtages, og undtagelsen fjernes ved afinstallation. Hvis Defender ikke er til stede eller er låst af en politik, mislykkes dette trin uden fejlmeddelelse, og installationen fortsætter. | `/MERGETASKS="!defenderexclusion"` |

En lydløs installation (`/SILENT` eller `/VERYSILENT`) anvender begge indstillinger, medmindre du fravælger dem med den viste værdi `/MERGETASKS`.

{% hint style="success" %}
Installationsprogrammet tilføjer CLI
-mappen (`C:\Program Files\Chloros\cli\`) til dit system `PATH` én gang — geninstallation eller opgradering tilføjer ikke længere en duplikatpost og fjerner duplikater, som tidligere installationsprogrammer har tilføjet. Åbn en ny terminal efter installationen for at bruge `chloros-cli`. Installationsprogrammet tilføjer også to Windows
-firewallregler til backend&#x27;en på hver netværksprofil: **Chloros
Backend (UDP-In)**, til kamerastrømme, PTP, mDNS og DAQ-E-datastrømmen, samt**Chloros
Backend (TCP-In)**, begrænset til det lokale undernet, til en DAQ-E-firmwareopdatering, hvor sensoren opretter forbindelse tilbage til denne computer for at indsamle billedet. Begge fjernes ved afinstallation.
{% endhint %}

#### Linux
amd64 (.deb-pakke — CLI
+ backend)

* **Filtype**: .deb (Debian/Ubuntu-pakke)
* **Arkitektur**: x86\_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux
arm64 — NVIDIA Jetson (.deb-pakke — CLI
+ backend)

* **Filtype**: .deb (JetPack 6)
* **Arkitektur**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Brug det filnavn, du rent faktisk har downloadet, og behold `./`, så `apt` installerer den pågældende fil i stedet for at søge efter en pakke ved navn. Installer med `apt install`, ikke med `dpkg -i`: apt installerer også Python
-SDK
&#x27;s krav, som pakken anbefaler snarere end kræver, og `dpkg -i` springer dem over. Siden version 1.3.0 installeres backend og CLI
&#x27;en udpakket under `/usr/lib/chloros/`, så de starter uden først at skulle pakke sig selv ud.

Jetson-pakken bruger JetPack 6&#x27;s CUDA-build af PyTorch, så processing – inklusive den medfølgende Texture Aware-debayer – kører på Jetsons GPU. Den bruger de CUDA- og cuDNN-biblioteker, som JetPack 6 installerer (`sudo apt install nvidia-jetpack` på et image, der mangler dem).

Se [Linux
-installation](linux/linux-installation.md) for detaljerede installationsvejledninger og [NVIDIA Jetson-vejledning](linux/nvidia-jetson-guide.md) for Jetson-specifik vejledning.

#### Python
SDK
(alle platforme)

Hvert installationsprogram indeholder et tilhørende `chloros_sdk`-wheel, så SDK
-versionen altid matcher den installerede GUI/CLI
/backend. På Windows
installerer installationsprogrammet det automatisk i dit system-Python
. På Linux
installerer `.deb` SDK
&#x27;en til selve systemet `python3` (i `/usr/lib/python3/dist-packages`), så `import chloros_sdk` fungerer straks efter installationen, uden pip og uden netværk. Kun et virtualenv, som ikke har adgang til systemets pakker, kræver, at den medfølgende wheel installeres i det:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

For værter, der kun bruger pip (ingen Chloros
-pakke installeret; SDK
&#x27;en styrer i så fald en backend på en anden maskine), findes SDK
&#x27;en også på PyPI:

```bash
pip install chloros-sdk
```

Se [API
: Python
SDK
](api-python-sdk.md) og [SDK
-referencen](reference/sdk-reference.md) for dokumentation.

{% hint style="info" %}
**Linux
-brugere**: Pakken `.deb` installerer CLI
, backend og Python
-SDK
. Der findes ingen GUI til Linux
— al interaktion foregår via CLI
eller SDK
.
{% endhint %}

***

## Yderligere ressourcer

### Python
SDK


Til udviklere og automatiseringsworkflows følger Chloros
Python
SDK
med alle installationsprogrammer: Windows
-installationsprogrammet installerer det i din Python
, og Linux
-installationsprogrammet `.deb` installerer det til systemet `python3`. På en maskine uden Chloros
-pakke (en, der driver et backend på en anden maskine), skal du installere det fra PyPI:

```bash
pip install chloros-sdk
```

**Dokumentation**: [API
: Python
SDK
](api-python-sdk.md)

**Krav**: et Chloros
-backend, der skal styres (Windows
-installationsprogrammet eller Linux
-pakken `.deb` installerer et), samt et Chloros
+-login på et betalt abonnement

***

## Hvad er inkluderet

### Windows
-installationsprogram

* ✅ **Chloros
GUI** – Grafisk brugergrænseflade med alle funktioner
* ✅ **Chloros
CLI
** – Kommandolinjegrænseflade (kræver Chloros
+-licens)
* ✅ **Chloros
-backend** – Behandlingsmotor
* ✅ **Python
SDK
** – installeres i din Python
, hvis en sådan findes (kræver Chloros
+-licens)
* ✅ **Kameraprofiler** – forudkonfigurerede MAPIR
-kameraskabeloner

### Linux
.deb-pakke

* ✅ **Chloros
CLI
** – Kommandolinjegrænseflade (kræver Chloros
+-licens)
* ✅ **Chloros
Backend** – Behandlingsmotor
* ✅ **Python
SDK
** – installeres til systemet `python3` (kræver Chloros
+-licens)
* ✅ **Kameraprofiler** – Forudkonfigurerede MAPIR
-kameraskabeloner
* ❌ Ingen GUI — Linux
er headless, kun CLI
/SDK


### Python
-SDK
(PyPI, til maskiner uden en Chloros
-pakke)

* ✅ **Chloros
-SDK
** – Python
-API
(kræver Chloros
+-licens)

***

## Opgrader til Chloros
+

Få adgang til avancerede funktioner med et Chloros
+-abonnement:

* 🚀 **Multitrådet behandling** – Behandl billeder parallelt
* ⚡ **GPU (CUDA)-acceleration** - Udnyt NVIDIA GPU-kraften
* 💻 **Adgang via CLI** - Automatiser med kommandolinjeværktøjer
* 🐍 **Python
SDK
** - Programmatisk API
-adgang
* 📱 **Flere enheder** - Brug på 2-10+ enheder (afhængigt af abonnement)
* **🐻 Avanceret teksturbevidst debayer-metode** – en højkvalitets, kantbevidst debayer kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al støj fra debayeringen.
* 🧮 **Brugerdefinerede formler** – Opret brugerdefinerede multispektrale indekser

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Se Chloros+-abonnementer og priser</a></p>***

## Hjælp til installation

### Fejlfinding

**Installationen mislykkes med følgende fejlmeddelelse:**

* Sørg for, at du har administratorrettigheder
* Deaktiver midlertidigt antivirusprogrammet
* Kontroller, at du opfylder de minimale systemkrav

**Programmet starter ikke (Windows

):**

* Kontroller, at Windows

10/11 (64-bit) er installeret
* Opdater grafikkortdrivere
* Tjek Windows

-hændelsesviseren for fejldetaljer
* Kontakt support med fejllogfiler

**CLI

starter ikke (Linux

):**

* Kontroller, at `.deb`-pakken er installeret korrekt: `dpkg -l | grep chloros`
* Kontroller brugerrettigheder: `sudo chmod +x /usr/bin/chloros-cli`
* Kør diagnostik: `chloros-cli selftest`
* Kontroller, om der mangler biblioteker: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (siden version 1.3.0 er `/usr/lib/chloros/chloros-backend` et lille startskript; selve programmet findes i `/usr/lib/chloros/backend/`)

**Problemer med licensaktivering:**

* Sørg for, at internetforbindelsen er aktiv
* Bekræft dine loginoplysninger på [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Kontroller, at firewallen ikke blokerer Chloros

* Se [Chloros

+ Login](chloros+-login.md) for detaljerede instruktioner

### Få support

Brug for hjælp til installation eller opsætning?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Hjemmeside**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentation**: [Kom godt i gang](./)
* ❓ **Ofte stillede spørgsmål**: [Ofte stillede spørgsmål](faq.md)***

## Softwareopdateringer

Chloros

tjekker for opdateringer, meddeler, når en ny version er tilgængelig, og linker til denne downloadside — du opdaterer ved at køre det nye signerede installationsprogram. Dine indstillinger, projekter og login bevares efter opdateringer. På Linux

og Jetson tjekker `chloros-cli update` for en nyere version og tilbyder at downloade og installere den tilsvarende `.deb` (denne kommando findes kun på Linux

).

***

## Ændringslog**Version 1.3.0 (Seneste, 11. november 2026)**— se**Hvad er nyt i Chloros

1.3.0** på siden [Kom godt i gang](./) for den fulde liste.**Version 1.2.0**— se**Hvad er nyt i Chloros

1.2.0** på siden [Kom godt i gang](./).

<details>

<summary>Version 1.0.5</summary>

**Udgivelsesdato: 10. februar 2026**

**Nye funktioner*** **Teksturbevidst debayer-metode \[Kun Chloros

+] -** Teksturbevidst bruger en højkvalitets kantbevidst debayer kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al debayer-støj.
* **Understøttelse af T4P-kalibreringsmål*** **Hurtigere Chloros

+ GPU-behandling, bedre hukommelsesstyring**

**Fejlrettelser*** Helt nyt brugergrænseflade (GUI), som nu bør fungere på alle Windows

-computere.

</details>

<details>

<summary>Version 1.0.4</summary>

**Udgivelsesdato: 5. januar 2026**

**Nye funktioner*** **Skift mellem billede og metadata**: Der er tilføjet en skifteknap i filbrowseren, så man kan se det valgte billedes metadata i en tabel i stedet for i billedgitteret
* **Zoom-skyder til billedgitter**: Ny skyder i brugergrænsefladen til at justere størrelsen på miniaturerne (understøtter også CTRL + musehjul)
* **Eksportknapper til billedgitter**: Knapper i den øverste række til at skifte miniaturer fra JPG til bearbejdede eksportformater (Targets, Reflectance, Index, LUT)
* **Fanen Kort**: Nyt interaktivt 2D-kort, der viser GPS-positionsmarkører for billederne
  * Understøtter Google Maps og ESRI-kortfliser (vælger automatisk den bedste flisetjeneste baseret på tilgængelighed ved det aktuelle zoomniveau)
  * Forhåndsvisning af miniaturer ved musemarkering på kortmarkører

**Fejlrettelser*** Forbedret understøttelse af installation af Chloros på computere med andre sprog end engelsk

</details>

<details>

<summary>Version 1.0.3</summary>

**Udgivelsesdato: 20. december 2025**

**Nye funktioner*** Første udgivelse

**Forbedringer*** Første udgivelse

**Fejlrettelser*** Første udgivelse

**Kendte problemer*** Første udgivelse

</details>

***

## Licensaftale**Proprietær software** – Copyright (c) 2026 MAPIR Inc.

Uautoriseret brug, distribution eller ændring er forbudt.

**Gratis version**: Tilgængelig til privat og kommerciel brug med begrænsninger i funktionerne**Chloros+**: Abonnementsbaseret licens til avancerede funktioner og kommerciel anvendelse

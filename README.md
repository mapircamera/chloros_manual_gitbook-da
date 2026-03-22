---
metaLinks: {}
---

# Kom godt i gang

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros er et softwareprogram fra [MAPIR](https://www.mapir.camera) til behandling af billeder og andre sensordata.

***{% hint style="success" %}**Nyt i Chloros 1.1.0**: Indbygget Linux-understøttelse (amd64 og arm64), NVIDIA Jetson edge computing, Dynamic Compute Adaptation, 4-trådet behandlingspipeline, nye CLI-kommandoer og -indstillinger. Se [Download](download.md) for den fulde changelog.
{% endhint %}

Chloros er tilgængelig i 3 applikationstilstande:

## Chloros: Desktop GUI-applikation

Selvstændigt separat vindue med alle funktioner. _Kun Windows._

## [Chloros CLI: Kommandolinjegrænseflade](CLI.md)

Batchbehandling via kommandolinjen. Perfekt til automatisering, scripting og headless-drift. Tilgængelig på **Windows, Linux amd64 og Linux arm64 (NVIDIA Jetson)**. _CLI kræver en Chloros+-licens for at få adgang._

## [Chloros API: Python SDK](api-python-sdk.md)

Programmatisk Python-grænseflade til automatisering og brugerdefinerede arbejdsgange. Perfekt til forskningspipelines, integration med eksisterende Python-applikationer og opbygning af brugerdefinerede værktøjer. Tilgængelig på **alle platforme** via `pip install chloros-sdk`. _API kræver en Chloros+-licens for at få adgang._***

## Understøttede platforme

| Platform | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Ja | Ja | Ja |
| **Linux amd64 (x86_64)** | Nej | Ja | Ja |
| **Linux arm64 (NVIDIA Jetson)** | Nej | Ja | Ja |

For installationsvejledning til Linux, se afsnittet [Linux &amp; Edge Computing](linux/linux-overview.md).

***

## Chloros+

Selvom Chloros er gratis at bruge til de fleste opgaver, vil du måske opleve, at du har brug for mere. Her kan en betalt licens til Chloros+ være en fordel for dig. Med en Chloros+-licens kan du låse op for nye funktioner såsom:

* **Multitrådet behandling**: fremskynd billedbehandlingen betydeligt for større projekter ved at behandle billeder samtidigt gennem pipelinen.
* **GPU (CUDA)-acceleration**: Udnyt nutidens større GPU-hukommelsesmuligheder til at fremskynde billedbehandlingspipeline yderligere. Vi anbefaler 4 GB eller mere VRAM for at opnå de bedste resultater.
* **Chloros+**[**CLI**](CLI.md)**Adgang**: kør Chloros+ fra kommandolinjen for at automatisere og integrere i din egen software.
* **Chloros+**[**API**](api-python-sdk.md)**Adgang:** kør Chloros+ fra Python for programmatisk kontrol, hvilket muliggør problemfri integration med dine forskningspipelines, dataanalyse-workflows og brugerdefinerede applikationer.
* **Brug på flere enheder**: hver Chloros+-licens giver mulighed for registrering af 2+ enheder. Brug din MAPIR Cloud-konto til at administrere registrerede enheder. Tilføj support til flere enheder ved at opgradere din Chloros+-licens.
* **Avanceret teksturbevidst debayer-metode:** en højkvalitets kantbevidst debayer kombineret med en AI/ML-støjfjernelsesmodel, der fjerner næsten al debayering-støj. 
* **Brugerdefinerede multispektrale indeksformler:** indtast brugerdefinerede multispektrale indekser i Chloros-rasterberegnerne, både til behandling og til billedvisningssandkassen.
* **Linux &amp; Edge Computing:** kør Chloros på Linux x86\_64- og ARM64-platforme, herunder NVIDIA Jetson, til felt- og edge-behandling. Se [Linux Oversigt](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Priser og tilmelding</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>

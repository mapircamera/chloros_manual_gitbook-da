# DAQ-lyssensorer

> **Leder du efter hardware?**Selve sensorerne — modeller, montering, dæksler, porte, strømforsyning og SCANNER-appen — er beskrevet i**[DAQ-brugervejledningen](https://mapir.gitbook.io/daq)**. Dette kapitel beskriver brugen af dem fra Chloros.

MAPIR’s **DAQ**-lyssensorer måler omgivende lys som radiometrisk kalibrerede spektre. I Chloros har de to funktioner:

* **Et selvstændigt spektralinstrument** — live-spektrumdiagrammer, kolorimetriske data og `.daq`-optagelser, alt sammen fra [fanen Lyssensorer](gui.md), [CLI](cli-quick-start.md) eller Python SDK.
* **En nedadrettet strålingskilde til reflektans** — under behandlingen interpolerer Chloros dine `.daq`-målinger til hver optagelseseksponeringstidsstempel og bruger det målte nedadgående lys til at konvertere kameraets radiance til reflektans (`--reflectance-source daq`); der kræves ingen panel i motivet for kalibrerede bånd.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Tre modeller, ét dataformat

| Model | Transport | Opdagelse |
| --- | --- | --- |
| **DAQ-U** | USB (seriel) | scanning af seriel port |
| **DAQ-M** | Bluetooth Low Energy | BLE-scanning efter navn |
| **DAQ-E** | Ethernet (IPv4, PoE-strømforsynet) | mDNS `_daq-e._tcp` (værtsnavn `daq-e-<id>.local`) |

Alle tre bruger den samme kommunikationsprotokol og leverer identiske data:

* Et **135-punkts spektrum fra 340 til 1010 nm i trin på 5 nm** samt CIE XYZ-tristimulusværdier i hver ramme.
* **Radiometrisk kalibreret spektral irradians i W/m²/nm** — hver enheds fabrikskalibreringspakke (samt dens aktive cap-korrektionsprofil) anvendes, inden dataene når frem til dig.
* Det samme **`.daq`-optagelsesformat** (en SQLite-fil). Den efterfølgende databehandling er identisk, uanset hvilken transportmetode der har genereret filen.

Transportstakkerne (USB-seriel, BLE, mDNS/zeroconf) er samlet i Chloros-backend&#x27;et — der skal ikke installeres noget for at kommunikere med nogen af de tre modeller fra GUI&#x27;en eller via CLI&#x27;s `pool-*`-kommandoer.

***

## Kalibreret område: 340–1010 nm rapporteret, ~374–974 nm kalibreret

Sensoren rapporterer det fulde 340–1010 nm-gitter, men den NIST-sporbare radiometriske forstærkning dækker ca. **374–974 nm**. Chloros afviser den absolutte reflektansdeling for ethvert kamerabånd, hvor mindre end halvdelen af dets spektrale vægt ligger inden for det kalibrerede interval; det oversprungne bånd rapporteres med overspringelsesårsagen `dls-uncalibrated-band-<nm>`.

Blandt de leverede LATTICE-filter-SKU&#x27;er er kun **F988** berørt:

F988-reflektansen kalibreres ved hjælp af et reflektanspanel i scenen: Båndet ligger uden for DAQ-lyssensorens kalibrerede område, så Chloros anvender din seneste paneloptagelse og opbevarer den mellem panelobservationerne.

Hvis en F988-registrering behandles, hvor der kun er DAQ-data tilgængelige, afviser Chloros DAQ-baseret reflektans for det pågældende bånd med springårsagen `dls-uncalibrated-band-988` — [reflektanspanel-arbejdsgangen](../calibration-targets.md) er den understøttede fremgangsmåde for F988.

***

## Sensor-ID&#x27;er

Hver DAQ rapporterer et stabilt sensor-ID. Dets format varierer alt efter model:

| Model | ID-format | Eksempel |
| --- | --- | --- |
| DAQ-U | 5-oktett med bindestreger | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-oktett med bindestreger | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Sensor-ID&#x27;et er:

* indstemplet i hver `.daq`-fil, den registrerer,
* den nøgle, Chloros bruger til at hente enhedens fabrikskalibreringspakke,
* den værdi, du sender til `--sensor-id` i CLI- og `pool-*`-kommandoerne, og
* for DAQ-E også dens mDNS-værtsnavn (`daq-e-def330.local`) — den værdi, som `--eth-host` accepterer.

***

## Fabrikskalibrering og skyen

Hver DAQ-enhed er individuelt fabrikskalibreret med en NIST-sporbart radiometrisk kæde, og Chloros indlæser hver enheds kalibreringspakke, der er knyttet til dens sensor-ID. Kalibreringsrapporten for den enkelte enhed (PDF) kan downloades fra sensorens indstillinger under [fanen Lyssensorer](gui.md).

{% hint style="warning" %}
**DAQ-U og DAQ-M kræver adgang til skyen for at kunne kalibreres.**Ingen af modellerne gemmer noget lokalt: deres fabrikskalibreringspakker findes i MAPIR&#x27;s sky og hentes via sensor-ID (hvorefter de gemmes i den lokale cache). Chloros kræver en internetforbindelse for at kunne levere kalibrerede W/m²/nm-data fra en DAQ-U eller DAQ-M.**DAQ-E er undtagelsen** — den har sin kalibrering gemt på selve enheden.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## Hvor optagelserne gemmes

| Overflade | Standarddestination for `.daq` |
| --- | --- |
| GUI — fanen Lyssensorer | `<project folder>/light_sensor/` (færdige optagelser føjes automatisk til projektet) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` på den maskine, der kører backend |

Hvert `.daq`-filnavn indeholder sensor-ID&#x27;et og et tidsstempel.

***

## I dette kapitel

* [**Fanen DAQ i Chloros**](gui.md) — den fulde gennemgang af brugergrænsefladen: tilslutning af hver model, indstillinger pr. sensor, spektrumdiagrammer, live kolorimetriske data, reflektans fra to sensorer og optagelse.
* [**CLI Hurtigstart (pool-\*)**](cli-quick-start.md) — styring af DAQ-sensorer fra `chloros-cli daq pool-*`, den understøttede kommandolinjevej.
* [**Cap-profiler og kalibreret spektralområde**](caps-and-range.md) — hvilke cap-profiler der findes pr. model, hvordan man definerer dem, samt det kalibrerede spektralområde i detaljer.
* [**Optagelse og .daq-formatet**](recording.md) — `.daq` SQLite-formatet og arbejdsgange for optagelse.
* [**DAQ-E-netværk og tidssynkronisering**](ethernet-ptp.md) — DAQ-E-transporttilstande og PTP-tidssynkronisering.
* [**Arbejdsgange for reflektans**](reflectance.md) — brug af DAQ-downwelling-data til at beregne reflektans.
* For fuldstændig dokumentation på flag-niveau, se [CLI-referencen](../reference/cli-reference.md) (afsnit `chloros-cli daq`) og [SDK-referencen](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), som begge er skrevet, så de kan bruges direkte af AI-assistenter.

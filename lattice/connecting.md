# Tilslutning af kameraer

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Fanen &quot;Kameraer&quot;, før der er tilsluttet noget</p></figcaption></figure>Chloros finder automatisk LATTICE-kameraer på forbindelsen — fra fanen &quot;Kameraer&quot; i brugergrænsefladen, fra `chloros-cli lattice` eller fra Python SDK. Kameraets modelstreng styrer alt det, der følger efter: Chloros udleder sensorprofilen, båndlayoutet og fabrikskalibreringen fra kameraets `DeviceUserID` + `DeviceSerialNumber`, så **der er intet, der skal konfigureres for hvert enkelt kamera**.

Før tilslutning skal du sikre dig, at værtsnetværket er konfigureret — link-lokal adressering, jumbo-rammer og, for arrays, indstillingerne for NIC-modtagelsesbufferen. Dette er opsætning på hardwaresiden og beskrives i LATTICE-manualen: [**Netværksopsætning**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Tilslutning fra GUI&#x27;en

Åbn fanen **Kameraer**i sidepanelet i Chloros (hardwarefanerne vises, når backend-systemet er færdig med at starte), eller brug hovedmenuen →**Tilslut til kamera**. Begge muligheder åbner dialogboksen**Tilslut kamera(er)**.

### Dialogboksen **Tilslut kamera(er)**Dialogboksen scanner netværket, så snart den åbnes (&quot;Scanner netværk...&quot;), og viser en liste over alle de kameraer, den finder. Hver række viser kameraets**model**(f.eks. `LATT-M3M-L41-F550`),**serienummer**og**IP-adresse**.

* **Klik på en række for at vælge den**(grøn markering). Du kan vælge**flere kameraer** og oprette forbindelse til dem på én gang — Chloros opretter forbindelse til dem i rækkefølge.
* Rækker med et **&quot;Tilsluttet&quot;**-mærke er allerede tilsluttet og kan ikke vælges igen.
* Rækker med et **&quot;I array&quot;**-mærke tilhører et aktuelt tilsluttet kamera-array. Afbryd først forbindelsen til arrayet for at bruge det pågældende kamera som enkeltstående.
* **Tilslut** — tilslutter det eller de valgte kameraer; knappen viser et antal, f.eks. &quot;Tilslut (3)&quot;, når der er valgt mere end ét.
* **Scann igen** — kører søgningen igen.
* **Luk** — lukker dialogboksen.
* Hvis scanningen afsluttes uden resultater, viser dialogboksen **&quot;Ingen kameraer fundet på netværket&quot;** — se [Fejlfinding](connecting.md#troubleshooting) nedenfor.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Dialogboksen Tilslut kamera(er) — vist her uden kameraer på netværket</p></figcaption></figure>### Første tilslutning: download af kalibreringspakke

Den **første gang**et givet kamera tilsluttes en maskine, henter Chloros kameraets fabrikskalibreringspakke (\~3,8 MB) fra selve kameraet via GigE. Mens dette kører, viser dialogboksen et grønt panel med teksten**&quot;Downloader kalibreringsdata fra kameraet&quot;**med en fremskridtsbjælke for hvert serienummer — det tager ca.**70 sekunder** pr. kamera. Pakken gemmes i cachen på værten, så senere tilslutninger af det samme kamera springer downloadet helt over (og panelet vises aldrig).

### Analyser system

Dialogboksens **Analyser system**-knap undersøger værten og netværket (teksten &quot;Analyserer...&quot; vises, mens det kører) og genererer en diagnostikrapport:

* **Vært** — CPU-kerner og RAM; GPU-navn og -hukommelse, eller »GPU: Ingen fundet«.
* **Netværksgrænseflader** — hvert netværkskortets navn, forbindelseshastighed, MTU (med en »jumbo«-markering, hvor den er aktiv), op/ned-status, og om det sidder på en USB-bus.
* **Kameraer**— serienummer, model, IP og**hvilket netværkskort hvert kamera er tilsluttet**.
* **Ydeevne** — aktuelle vs. ideelle fps pr. kamera for det pågældende pixelformat, med en grøn linje med teksten &quot;Potentiale: N× forbedring mulig&quot;, når det ideelle overstiger det aktuelle.
* **Advarsler og nummererede anbefalinger** — eller &quot;Systemet ser godt ud i forhold til det aktuelle antal kameraer,&quot; når der ikke er noget, der skal rettes.

Kør værktøjet, når opdagelse eller streaming opfører sig uventet — det identificerer de fleste problemer på netværkskort-siden (forkert MTU, kamera på den forkerte grænseflade, begrænsninger ved USB-adapteren) uden at forlade dialogboksen.

### Tilslutning af et array

For at tilslutte to eller flere kameraer som et **synkroniseret array**skal du i stedet bruge guiden til array-tilslutning (**Tilslut kameraarray**): Den guider dig gennem valg af master/slave (forudfyldt af en GPIO-forbindelsessonde), valg af visningstilstand (separate vs. kombinerede felter) og en scene med array-indstillinger med en live-visning af opnåelige fps og ledningsbåndbredde, før du bekræfter. Guiden og arbejdsgangene for kameraopstillinger er beskrevet i afsnittet om kameraopstillinger med flere kameraer i denne manual; det tilsvarende for CLI er »LATTICE Camera First-Connect Workflow« i [CLI-referencen](../reference/cli-reference.md).

## Tilslutning fra CLI og SDK

Adgang til CLI og SDK kræver et betalt Chloros+-abonnement og at man er logget ind; dette håndhæves på serversiden (`401 AUTH_REQUIRED`, når man ikke er logget ind, `403 PLAN_UPGRADE_REQUIRED` på det gratis abonnement).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Fuldstændige signaturer, indstillinger og indsamlingsworkflows: [CLI Reference](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK-reference](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Sådan håndteres kalibrering ved opkobling

Hvert LATTICE-kamera har sin fabrikskalibreringspakke **indbygget i kameraet**, og Chloros tjekker også MAPIR&#x27;s cloud, når kameraet opretter forbindelse:

| Situation   | Hvad Chloros bruger                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**| Den**nyeste kalibrering, der er offentliggjort for det pågældende serienummer** — kopien i skyen har forrang for kopien på kameraet. Et kamera, der er blevet rekalibreret eller opdateret af MAPIR, opdaterer sig derfor automatisk; ingen brugerhandling er nødvendig. |
| **Offline**| Den**pakke, der findes på kameraet**, som den er. Fuldt offline-arbejdsgange fungerer fortsat; de henter blot ikke nyere kalibreringer, før kameraet har været online én gang (eller er blevet reflashet til fabriksindstillingerne).                                                  |

På optagelsestidspunktet **fastfryses** de koefficienter, der faktisk anvendes, i hvert billedes XMP-metadata. En senere kalibreringsopdatering ændrer aldrig i det skjulte billeder, du allerede har taget — ved genbehandling af en gammel optagelse anvendes de koefficienter, der er gemt i dens XMP, ikke de nyeste, der findes i dag.

## Fejlfinding

* **&quot;Der blev ikke fundet nogen kameraer på netværket&quot;**— kontroller den link-lokale opsætning i [Netværksopsætning](https://mapir.gitbook.io/lattice-camera/setup/network-setup): statisk værts-NIC `169.254.x.x/16`, kameraer på samme link, ingen DHCP/gateway forventes. Brug derefter**Analyze System**i forbindelsesdialogen til at kontrollere, hvilket netværkskort hvert kamera er (eller ikke er) synligt på.**Rescan** efter enhver ændring af kabelføring eller netværkskort.
* **Et system, der tidligere fungerede, nægter at oprette forbindelse** (array-panelets porte med `FRAMES WILL DROP` / `Reduce ROI to enable`) — en opdatering af netværkskortdriveren har uden varsel nulstillet indstillingerne for modtagelsesringen. Anvend dem igen, eller kør `chloros-cli lattice network --fix` fra en terminal med administratorrettigheder; se [Netværksopsætning](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Et kamera viser &quot;In Array&quot;** — det hører til en tilsluttet array-session. Afbryd arrayet for at bruge kameraet som en selvstændig enhed.

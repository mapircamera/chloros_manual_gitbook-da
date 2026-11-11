# Tilslutning af kameraer

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Fanen &quot;Kameraer&quot;, før der er tilsluttet noget</p></figcaption></figure>

Chloros
finder automatisk LATTICE-kameraer på forbindelsen — fra fanen &quot;Kameraer&quot; i brugergrænsefladen, fra `chloros-cli lattice` eller fra Python
-SDK
&#x27;en. Kameraets modelstreng styrer alt det efterfølgende: Chloros
udleder sensorprofilen, båndlayoutet og fabrikskalibreringen fra kameraets `DeviceUserID` + `DeviceSerialNumber`, så **der er intet, der skal konfigureres for hvert enkelt kamera**.

Før tilslutning skal du sikre dig, at værtsnetværket er konfigureret — link-local-adressering, jumbo-rammer og, for array-konfigurationer, indstillingerne for NIC-modtagebufferen. Dette er opsætning på hardwaresiden og findes i LATTICE-manualen: [**Netværksopsætning**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Tilslutning fra GUI&#x27;en

Åbn fanen **Kameraer** i Chloros
-sidepanelet (hardwarefanerne vises, når backend-systemet er færdig med at starte), eller brug hovedmenuen → **Tilslut til kamera**. Begge åbner dialogboksen**Tilslut kamera(er)**.

### Dialogboksen **Tilslut kamera(er)**Dialogboksen scanner netværket, så snart den åbnes (&quot;Scanner netværk...&quot;), og viser en liste over alle kameraer, den finder. Hver række viser kameraets**model**(f.eks. `LATT-M3M-L41-F550`),**serienummer**og**IP-adresse**.

* **Klik på en række for at vælge den**(grøn markering). Du kan vælge**flere kameraer** og tilslutte dem på én gang — Chloros
tilslutter dem i rækkefølge.
* Rækker med mærket **&quot;Tilsluttet&quot;** er allerede tilsluttet og kan ikke vælges igen.
* Rækker med mærket **&quot;I array&quot;** tilhører et kamera-array, der i øjeblikket er tilsluttet. Afbryd først gruppen for at bruge det pågældende kamera enkeltvis.
* **Tilslut** — tilslutter det eller de valgte kameraer; knappen viser et antal, f.eks. &quot;Tilslut (3)&quot;, når der er valgt mere end ét.
* **Scann igen** — kører søgningen igen.
* **Luk** — lukker dialogboksen.
* Hvis søgningen afsluttes uden resultater, viser dialogboksen **&quot;Ingen kameraer fundet på netværket&quot;** — se [Fejlfinding](connecting.md#troubleshooting) nedenfor.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Dialogboksen Tilslut kamera(er) — vist her uden kameraer på netværket</p></figcaption></figure>

### Første tilslutning: download af kalibreringspakke

Den **første gang** et givet kamera tilsluttes en maskine, henter Chloros
kameraets fabrikskalibreringspakke (\~3,8 MB) fra selve kameraet via GigE. Mens dette kører, viser dialogboksen et grønt panel med teksten **&quot;Downloader kalibreringsdata fra kameraet&quot;**med en fremskridtsbjælke for hver serienummer — regn med ca.**70 sekunder** pr. kamera. Ved mere end seks kameraer ruller fremskridtslinjerne inde i panelet. Pakken gemmes i cachen på værten, så senere tilslutninger af det samme kamera springer downloadet helt over (og panelet vises aldrig).

### Analyser system

Dialogboksens **Analyze System**-knap undersøger værten og netværket (teksten lyder &quot;Analyzing...&quot; mens den kører) og genererer en diagnosticeringsrapport:

* **Host** — CPU-kerner og RAM; GPU-navn og hukommelse, eller &quot;GPU: None detected&quot;.
* **Netværksgrænseflader** — hvert netværkskortets navn, forbindelseshastighed, MTU (med en &quot;jumbo&quot;-markering, hvor det er aktivt), op/ned-tilstand og om det sidder på en USB-bus.
* **Kameraer**— serienummer, model, IP og**hvilket netværkskort hvert kamera er tilsluttet**.
* **Ydeevne** — aktuelle vs. ideelle fps pr. kamera for det pågældende pixelformat, med en grøn linje med teksten &quot;Potentiale: N× forbedring mulig&quot;, når det ideelle overstiger det aktuelle.
* **Advarsler og nummererede anbefalinger** — eller »Systemet ser godt ud i forhold til det aktuelle antal kameraer«, når der ikke er noget, der skal rettes.

Kør værktøjet, når opdagelse eller streaming opfører sig uventet — det identificerer de fleste problemer på netværkskortssiden (forkert MTU, kamera på den forkerte grænseflade, begrænsninger ved USB-adapteren) uden at forlade dialogboksen.

### Tilslutning af et array

For at tilslutte to eller flere kameraer som et **synkroniseret array**skal du i stedet bruge guiden til tilslutning af array (**Connect Camera Array**): Den guider dig gennem valg af master/slave (forudfyldt af en synkroniseringskabel-test — ethvert kablet kamera kan være master, og RGB
-kameraet `-FRGB` er forudvalgt, når det er tilsluttet via kabel), valg af visningstilstand (separate vs. kombinerede felter) samt en scene med array-indstillinger med en live-visning af opnåelige fps og kabelbåndbredde, inden du bekræfter. Guiden og array-arbejdsgangene er beskrevet i afsnittet om multikamera-arrays i denne manual; CLI
-ækvivalenten er »LATTICE Camera First-Connect Workflow« i [CLI
-referencen](../reference/cli-reference.md).

## Tilslutning fra CLI
og SDK
Adgang til

CLI
og SDK
kræver et betalt Chloros
+-abonnement og at man er logget ind; dette håndhæves på serversiden (`401 AUTH_REQUIRED`, når man ikke er logget ind, `403 PLAN_UPGRADE_REQUIRED` på gratisabonnementet).

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

Fuldstændige signaturer, indstillinger og indsamlingsworkflows: [CLI
-reference](../reference/cli-reference.md) § `chloros-cli lattice`, [SDK
-reference](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Sådan håndteres kalibrering ved opkobling

Hvert LATTICE-kamera har sin fabrikskalibreringspakke **gemt på selve kameraet**, og Chloros
tjekker også MAPIR
’ cloud, når kameraet opretter forbindelse:

| Situation   | Hvad Chloros
bruger                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**| Den**nyeste kalibrering, der er offentliggjort for det pågældende serienummer** — kopien i skyen har forrang frem for kopien på kameraet. Et kamera, der er blevet rekalibreret eller opdateret af MAPIR
, opdaterer sig derfor automatisk; der kræves ingen handling fra brugeren. |
| **Offline**| Den**pakke, der findes på kameraet**, som den er. Fuldt offline-arbejdsgange fungerer fortsat; de henter blot ikke nyere kalibreringer, før kameraet har været online én gang (eller er blevet reflashet til fabriksindstillingerne).                                                  |

På optagelsestidspunktet **fastfryses** de koefficienter, der faktisk anvendes, i hvert billedes XMP-metadata. En senere kalibreringsopdatering ændrer aldrig i det skjulte billeder, du allerede har taget — ved genbehandling af en gammel optagelse anvendes de koefficienter, der er gemt i dens XMP, ikke de nyeste, der findes i dag.

## Fejlfinding

* **&quot;Ingen kameraer fundet på netværket&quot;**— kontroller den link-lokale opsætning i [Netværksopsætning](https://mapir.gitbook.io/lattice-camera/setup/network-setup): værts-NIC&#x27;en skal have en statisk adresse `169.254.x.x/16`, kameraerne skal være på samme link, og der forventes ingen DHCP/gateway. Brug derefter**Analyser system**i forbindelsesdialogen til at kontrollere, hvilket netværkskort hvert kamera er (eller ikke er) synligt på.**Scann igen** efter enhver ændring af kabelføring eller netværkskort.
* **Et system, der tidligere fungerede, nægter at oprette forbindelse** (array-panelets porte med `FRAMES WILL DROP` / `Reduce ROI to enable`) — en opdatering af netværkskortdriveren har uden varsel nulstillet indstillingerne for modtagerringen. Anvend dem igen, eller kør `chloros-cli lattice network --fix` fra en terminal med administratorrettigheder; se [Netværksopsætning](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Et kamera viser &quot;In Array&quot;** — det hører til en tilsluttet array-session. Afbryd arrayet for at bruge kameraet som enkeltstående enhed.

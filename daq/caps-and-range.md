# Kappe-profiler og kalibreret rækkevidde

> Selve hætterne — hvilke hætter der følger med hvilke sensorer, hvordan de monteres og deres optiske egenskaber — er beskrevet i **[DAQ-brugervejledningen](https://mapir.gitbook.io/daq)**. Denne side handler om at *angive* den monterede hætte til Chloros, hvilket er det, der sikrer, at korrektionen bliver korrekt.

Hver DAQ-lyssensors radiometriske fabrikskalibrering beskriver den *rene* sensor. Den fysiske kappe, der er monteret over diffusoren, ændrer det lys, sensoren opfanger, så Chloros anvender en fabriksmålt **kappe-korrektionsprofil** oven på kalibreringspakken. At angive den korrekte hætte er en del af at opnå kalibrerede data — denne side beskriver, hvilke hætter der findes pr. model, hvordan man angiver dem, og hvad sensorens kalibrerede spektralområde faktisk er.

## Hætternes tilgængelighed efter model

| Hætteprofil (`cap_id`) | Fysisk hætte | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Sunshine cosinus-korrektionskappe (**standard på alle modeller**) | Ja | Ja | Ja |
| `fov_15` / `fov_45` / `fov_90` | FOV-begrænsende kegler (15° / 45° / 90°) | Ja | — | Ja |
| `fov_30` / `fov_60` | FOV-begrænsende kegler (30° / 60°) | Ja | — | — |
| `none` | Ingen hætte monteret | — | — | Ja |

Modelspecifikke bemærkninger:

* **DAQ-M har én hætteprofil: `sunshine_cosine`.** »Bare-plus-Sunshine-cap« er produktdefinitionen, og en »bare« DAQ-M behøver ingen geometriprofil.
* **En »bare« DAQ-U er en ægte »bare«** — den behøver slet ingen geometriprofil, hvilket er grunden til, at der ikke findes en `none`-profil til den.
* **`none` på en DAQ-E er IKKE en no-op.** DAQ-E’s forsænkede, glasoverdækkede diffusor har sin egen reelle geometrikorrektion, så »ingen kappe« er i sig selv en målt profil på denne model.
* En **DAQ-E uden afskærmning kan ikke måle direkte sollys uanset højden** — Sunshine-afskærmningen er standardkonfigurationen. Planlæg ikke udendørs arbejde med en DAQ-E uden afskærmning.

I GUI’ens indstillinger pr. sensor (tandhjulsikonet under fanen Lyssensorer) tilbyder rullemenuen **Cap** også ”None (bare sensor)” på DAQ-U og DAQ-M — på disse to modeller betyder ”bare” ganske enkelt, at der ikke anvendes nogen kappe-korrektion, jf. bemærkningerne ovenfor. Vælg denne indstilling kun, når hætten er fysisk fjernet.

## Angivelse af hætten — og hvorfor det er vigtigt

**Den angivne `cap_id` skal svare til den hætte, der fysisk sidder på sensoren.** Hverken sensoren eller softwaren kan registrere den monterede kappe. Angivelsen har betydning for to ting:

1. Den **live-korrektion**, der anvendes på hvert spektrum.
2. Det **kappe-stempel, der skrives ind i hver `.daq`-optagelse**, som den efterfølgende reflektansbehandling baserer sig på.

Sunshine-hætten dæmper **ca. 12 gange ifølge designet**, så optagelse med angivelse af den forkerte hætte medfører en forkert skalering af spektrene med omtrent denne faktor. Angiv straks ændringer i hætten.

### Indstilling af hætten

GUI: Fanen Lyssensorer → tandhjulsikonet på sensorlinjen → rullemenuen **Hætte**. Standardindstillingen for alle modeller er `sunshine_cosine`, hvilket er en antagelse (DAQ-U, DAQ-M og DAQ-E leveres uden cap, og en optagelse angiver, at cap blev antaget, indtil du vælger en), og valget bevares i projektet.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(backend skal køre):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI accepterer syntaktisk den fulde `cap_id`-liste (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); hver profil valideres i forhold til sensorens model ved opkobling, så et utilgængeligt cap-id (f.eks. et E-only-id på en DAQ-U) giver en tydelig fejlmeddelelse i stedet for at korrigere forkert. Backend-standardværdien, når der ikke angives noget, er `sunshine_cosine`.

Python SDK: Navngiv cap'en ved opkobling med `connect_daq_sensor(cap_id="sunshine_cosine")`, eller skift den under kørsel med `DAQSensorSession.set_cap("fov_45")`; se [SDK-referencen](../reference/sdk-reference.md#daq-sensor-sessions).

Avanceret: Profiler leveres som en del af Chloros-installationen under `daq/cap_profiles/<u|m|e>/<cap_id>.json` og kan tilsidesættes for den enkelte bruger under `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Uafhængigt af begrænsningerne modtager sensorer, der aldrig er blevet rekalibreret, automatisk en lille finjustering af mørkeforskydningen baseret på flåden — uden brugerindgriben.

## DAQ-E-S: fast diffusor og hældning

**DAQ-E-S** er en DAQ-E med to forskelle, der har betydning for Chloros. Alt andet (opdagelse, tilslutning, PTP, optagelse, enhedsindstillinger) fungerer nøjagtigt som for en DAQ-E.

* **Dens diffusor er fastmonteret, så den kræver ingen cap-korrektion.** Diffusoren sad på enheden, da dens fabrikskalibrering blev målt, så kalibreringen allerede angiver den reelle nedadgående strålingsintensitet. Chloros anvender ingen cap-profil på den, uanset hvad der anmodes om: en cap opkaldt efter den ignoreres med en meddelelse, dens optagelser angiver, at cap'en stammer fra modellen, og advarslen om antaget cap udløses aldrig for den. I Projektindstillinger → DAQ-lyssensor viser dens række ingen cap, og den projektdækkende cap-tilsidesættelse gælder ikke for den. Anvend ikke en cap på dens data manuelt: Sunshine-korrektionen ovenpå ville vise en værdi, der er ca. 11 gange for høj.
* **Den er udstyret med et 3-akset accelerometer.** Diffusorens cosinusrespons er kun gyldig, mens den står vandret, så en skråstillet sensor rapporterer en forkert strålingsintensitet uden andre symptomer. Hver DAQ-E-S-måling indeholder derfor dens hældning, rullning og stigning, og en `.daq`, der er registreret fra en sådan, gemmer disse oplysninger ved siden af hvert spektrum (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), så man kan skelne mellem en sky og et skævt monteret stativ.

For hvert holdningsfelt og hvordan en fejl i holdningsstrømmen rapporteres, se [DAQ-E-S i CLI-referencen](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Sunshine Cap-ydeevne (udendørskonfigurationen)

Tal, som du kan basere procedurer på:

| Egenskab | Værdi |
| --- | --- |
| Synsfelt | 180° halvkugleformet |
| Cosinusresponsfejl | ≤ ±4 % op til 60° indfaldsvinkel; ≤ ±4,5 % op til 70° |
| Grænse for lav solhøjde | Anbefales ikke under ~15° solhøjde |
| Dæmpning | ~12× (ifølge design) |
| Repeterbarhed ved genmontering af hætten | ≈ 1,5 % |
| Kvantitativ irradians | Gennemsnit af **≥ 15 s** målinger (instrumentkarakteristik, ikke en fejl) |

For ethvert tal for kvantitativ irradians — herunder reflektansreferencer — skal der anvendes et gennemsnit af mindst 15 sekunders målinger i stedet for et enkelt billede.

## Kalibreret spektralområde

| Egenskab | Værdi |
| --- | --- |
| Spektral sampling | 340–1010 nm i trin på 5 nm (135 punkter) |
| Radiometrisk kalibreret område | **~374–974 nm** (håndhæves i softwaren) |

Sensoren rapporterer det fulde 340–1010 nm-gitter, men den NIST-sporbare radiometriske forstærkning dækker ~374–974 nm. Chloros **afviser opdelingen efter absolut reflektans** for ethvert kamerabånd, hvor mindre end halvdelen af dets spektrale vægt ligger inden for dette interval, og rapporterer springårsagen `dls-uncalibrated-band-<nm>` i stedet for at levere et ukalibreret produkt. Blandt de kamera-SKU’er, der er på markedet, falder kun F988-filteret uden for dette interval; det bruger i stedet arbejdsgangen med reflektanspaneler — se [Arbejdsgange med reflektans](reflectance.md).

For sensormodeller, transportformer og sensor-ID’er, se [DAQ-oversigten](README.md). For information om, hvordan cap-stemplet forbruges under behandlingen, se [Optagelse og .daq-formatet](recording.md).

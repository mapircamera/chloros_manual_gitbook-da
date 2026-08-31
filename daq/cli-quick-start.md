# CLI Hurtigstart (pool-*)

De medfølgende `chloros-cli`-drev styrer DAQ-sensorer via **`daq pool-*`** kommandofamilien — tynde HTTP-klienter, der betjener sensoren via Chloros-backendens permanente sensorpool. Backend&#x27;en styrer transporten, så GUI&#x27;en, CLI og SDK-scripts deler alle ét aktivt håndtag i stedet for at kæmpe om porten. Alt, hvad en kunde har brug for, er tilgængeligt via `pool-*`: oprette forbindelse, streame, optage kalibrerede `.daq`-filer og skifte cap-profiler.

`pool-*` er også den **eneste** DAQ-overflade i udgivne versioner. `chloros-cli daq --help` viser listen over `pool-*`-underkommandoer, og hvis man udfører en direkte hardware-DAQ-underkommando på en udgivet version, afsluttes programmet med en eksplicit fejlmeddelelse, der angiver det manglende pakke og henviser tilbage til `pool-*` — intet mislykkes lydløst. (Kommandoerne til direkte hardware kører kun fra en MAPIR-kildekode-checkout; `pip install chloros-sdk` stiller dem heller ikke til rådighed.)

***

## Forudsætninger

* **Chloros-backend&#x27;en skal køre** — `pool-*`-kommandoerne er HTTP-klienter, ikke hardwaredrivere. På Windows skal du starte Chloros-desktopappen (den starter backend&#x27;en). På headless Linux/Jetson skal du aktivere tjenesten: `sudo systemctl enable --now chloros-backend.service`.
* **Et Chloros+ (betalt niveau) login**: Kør først `chloros-cli login`. Håndhævelsen sker på serversiden — uden login mislykkes kommandoer med `401 AUTH_REQUIRED`; på det gratis (Iron) niveau mislykkes de med `403 PLAN_UPGRADE_REQUIRED`.
* Kommandoerne er som standard rettet mod `http://127.0.0.1:5000`; `daq pool-*`-familien respekterer miljøvariablen `CHLOROS_BACKEND_URL`, hvis din backend kører et andet sted.

***

## En fem minutters session

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — åbn en sensor i puljen

| Variant | Betydning |
| --- | --- |
| `daq pool-connect` | Smart-detect: find enhver DAQ på denne maskine. |
| `daq pool-connect --port PORT` | DAQ-U på en bestemt seriel port (f.eks. `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M via BLE, MAC-adresse scannes automatisk. |
| `daq pool-connect --mac MAC` | DAQ-M på en kendt BLE-MAC (indebærer `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E med et kendt værtsnavn eller en kendt IP-adresse — **den pålidelige vej**. |
| `daq pool-connect --eth` | DAQ-E med automatisk opdagelse (mDNS, med ARP som reserve). Se forbeholdet nedenfor. |

Indstillingsflag, alle valgfrie:

| Flag | Betydning |
| --- | --- |
| `--integration-time MS` / `-t MS` | Manuel integrationstid i millisekunder. |
| `--frame-avg N` / `-f N` | Gennemsnitligt antal rammer pr. rapporteret spektrum. |
| `--no-ae` | Deaktiver automatisk eksponering (AE er aktiveret som standard). |
| `--no-stream` | Opret forbindelse uden at starte streamingen (fortsæt senere med `pool-stream --start`). |
| `--cap-id CAP` | Cap-korrektionsprofil; standard for backend er `sunshine_cosine`. Se [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**`--eth`-advarsel vedrørende automatisk opdagelse.** På en host med flere netværksgrænseflader (mere end én aktiv netværksgrænseflade) kan den *første* `pool-connect --eth` efter opstart muligvis ikke finde noget, selvom sensoren er i orden — søgningen kan overse sensorens grænseflade, mens ARP-cachen er tom. Hvis `--eth` ikke finder noget, skal du prøve igen eller helt springe opdagelsen over med `--eth-host <ip-or-hostname>`, hvilket er den pålidelige fremgangsmåde på maskiner med flere netværksgrænseflader. DAQ-E&#x27;s værtsnavn er `daq-e-<id>.local` (f.eks. `daq-e-def330.local`); dens rene IP-adresse fungerer også.
{% endhint %}

## `pool-list` — se, hvad der er tilsluttet

Viser alle sensorer i backend-puljen, herunder den `sensor_id`, som alle andre kommandoer har brug for:

| Model | `sensor_id`-format | Eksempel |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktett med bindestreger | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — læser spektrumrammer

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Returnerer den seneste ramme eller de seneste `--recent N`-rammer; `--json` genererer maskinlæsbar output til scripting. Rammerne er radiometrisk kalibrerede spektrale irradiansværdier (W/m²/nm) på et gitter med 135 punkter i området 340–1010 nm, hvor sensorens cap-profil allerede er anvendt. For at opnå kvantitative strålingsværdier skal der beregnes et gennemsnit af mindst 15 sekunders billeder — dette er en egenskab ved instrumentet, ikke en fejl.

## `pool-stream` — sæt streaming på pause eller genoptag den

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — optag en `.daq`-fil

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Flag | Standard | Betydning |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Optagelsens længde i sekunder; `0` betyder, at programmet kører, indtil du udfører `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Output-mappe, fastlagt **på den maskine, der kører backend**. |
| `--device-name NAME` | — | Etiket gemt sammen med optagelsen. |
| `--stop` | — | Stop en igangværende optagelse. |

{% hint style="info" %}
Optagelsen foregår i backend&#x27;en, så filen `.daq` ender på **backend-maskinens** filsystem — som standard i `~/Documents/DAQ Live View/` der, ikke nødvendigvis der, hvor du kørte CLI. Filnavne indeholder sensor-ID&#x27;et og et tidsstempel.
{% endhint %}

## `pool-set-cap` — angiv den monterede hætte

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

Hætten-ID&#x27;et vælger den fabriksmålte korrektionsprofil, der anvendes på hvert spektrum, og den **skal matche den hætten, der fysisk er monteret på sensoren** — hverken sensoren eller softwaren kan selv registrere hætten, og valget er indstemplet i hver `.daq`-fil. Standardindstillingen overalt er `sunshine_cosine` (hver DAQ leveres med Sunshine-kosinuskorrektionshætten monteret, ~12× dæmpning ifølge designet — et udeklareret skift af hætten korrigerer spektrene forkert med omtrent denne faktor).

| `--cap-id` | Tilgængelig på |
| --- | --- |
| `sunshine_cosine` (standard) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Kun DAQ-U |
| `none` | Kun DAQ-E — se bemærkning |

Et dæksel-ID, der ligger uden for sensorens indstillingsområde, afvises ved tilslutning med en tydelig fejlmeddelelse. `none` (DAQ-E) betyder, at hætten er fysisk fjernet — den anvender stadig en fabriksgeometriprofil til DAQ-E&#x27;s forsænkede glasdiffusor, så det er ikke en no-op, og en bar DAQ-E er en bordkonfiguration, ikke en understøttet feltkonfiguration. (En DAQ-U uden kappe er fuldstændig uden kappe og behøver slet ingen korrektionsprofil; DAQ-M bruges sammen med sin Sunshine-kappe.)

## `pool-disconnect` — frigiv sensorer

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Kommandooversigt

| Kommando | Formål |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Åbn en sensor i backend-puljen. |
| `daq pool-list` | Vis alle sensorer i puljen med deres `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | De seneste N kalibrerede spektrumrammer. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Genoptag/sæt streaming på pause. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Start/stop en `.daq`-optagelse (backend-side). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Skift cap-korrektionsprofilen under kørsel. |
| `daq pool-disconnect --sensor-id ID [--all]` | Frigiv én sensor eller dem alle. |

***

## Fejlfinding ved første DAQ-E-tilslutning

1. DAQ-E har ingen status-LED — kontroller strømforsyningen via PoE/link-indikatoren på switchen eller injektorporten, og vent et par sekunder efter opstart, indtil den er startet op og har tilsluttet sig netværket.
2. Backend-maskinen skal være på **samme broadcast-domæne** som sensoren — mDNS passerer ikke routere.
3. På Windows skal du acceptere Defender-firewall-prompten ved første opstart (mDNS UDP 5353, DAQ-E-data UDP 5002, PTP UDP 319/320).
4. Stadig intet fra `--eth`? Brug `--eth-host` med enhedens værtsnavn (`daq-e-<id>.local`) eller IP-adresse — den pålidelige metode, især på værter med flere netværksadresser.

***{% hint style="info" %}**Tip til AI-assistenter.** Hver side i denne manual leveres som rå Markdown — tilføj `.md` til en sides slug i små bogstaver (URL) (denne side: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); så bliver det maskinlæsbare indeks `https://mapir.gitbook.io/chloros/llms.txt`. For fuldstændig dokumentation på flag-niveau af `chloros-cli daq` og alle andre kommandofamilier, hent [CLI-referencen](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); stien til Python er `chloros_sdk.connect_daq_sensor()` i [SDK-referencen](../reference/sdk-reference.md).
{% endhint %}

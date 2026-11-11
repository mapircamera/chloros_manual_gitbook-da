# Hurtigstart til CLI (pool-*)

De medfølgende `chloros-cli`-drev styrer DAQ-sensorerne via **`daq pool-*`**-kommandofamilien — enkle HTTP-klienter, der styrer sensoren via Chloros-backendets permanente sensorpool. Backend'en styrer transporten, så GUI'en, CLI'en og SDK-scripts deler alle ét aktivt håndtag i stedet for at kæmpe om porten. Alt, hvad en kunde har brug for, er tilgængeligt via `pool-*`: oprette forbindelse, streame, optage kalibrerede `.daq`-filer og skifte cap-profiler.

`pool-*` er den understøttede DAQ-grænseflade i udgivne builds. `chloros-cli daq --help` viser listen over `pool-*`-underkommandoer, og `daq discover` / `daq list` udfører den samme scanning som `pool-discover`. Hvor en DAQ-underkommando til direkte hardware ikke er tilgængelig, afsluttes programmet med en eksplicit fejlmeddelelse, der angiver det manglende pakke og henviser dig tilbage til `pool-*` — intet mislykkes i stilhed. (`pip install chloros-sdk` indeholder heller ikke kommandoer til direkte hardware.)

***

## Forudsætninger

* **Chloros-backend'et skal køre** — `pool-*`-kommandoerne er HTTP-klienter, ikke hardwaredrivere. På Windows skal du starte Chloros-desktopappen (den starter backend'et). På headless Linux/Jetson skal du aktivere tjenesten som din bruger: `sudo systemctl enable --now chloros-backend@$USER.service` (se [Linux-installation](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Chloros+ (betalt abonnement) login**: Kør først `chloros-cli login`. Håndhævelsen sker på serversiden — uden login mislykkes kommandoer med `401 AUTH_REQUIRED`; på det gratis (Iron) niveau mislykkes de med `403 PLAN_UPGRADE_REQUIRED`.
* Kommandoerne er som standard rettet mod `http://127.0.0.1:5000`; `daq pool-*`-familien respekterer miljøvariablen `CHLOROS_BACKEND_URL`, hvis din backend kører et andet sted.

***

## En fem minutters session

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

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

Indstillingsflag, alle valgfri:

| Flag | Betydning |
| --- | --- |
| `--integration-time MS` / `-t MS` | Manuel integrationstid i millisekunder. |
| `--frame-avg N` / `-f N` | Gennemsnitligt antal rammer pr. rapporteret spektrum. |
| `--no-ae` | Deaktiver automatisk eksponering (AE er aktiveret som standard). |
| `--no-stream` | Opret forbindelse uden at starte streamingen (genoptag senere med `pool-stream --start`). |
| `--cap-id CAP` | Cap-korrektionsprofil; standardindstillingen for backend er `sunshine_cosine`. Se [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

Tilslutning af en DAQ-E, der allerede findes i puljen under en anden af dens adresser — dens `daq-e-<id>.local`-værtsnavn eller dens IP — returnerer den eksisterende puljeindgang i stedet for at åbne enheden en anden gang.

{% hint style="warning" %}
**`--eth` advarsel vedrørende automatisk opdagelse.** På en vært med flere netværksgrænseflader (mere end én aktiv netværksgrænseflade) kan den *første* `pool-connect --eth` efter opstart muligvis ikke finde noget, selvom sensoren fungerer korrekt — søgningen kan overse sensorens grænseflade, mens ARP-cachen er tom. Hvis `--eth` ikke finder noget, skal du prøve igen eller helt springe opdagelsen over med `--eth-host <ip-or-hostname>`, hvilket er den pålidelige fremgangsmåde på maskiner med flere netværksgrænseflader. DAQ-E’s værtsnavn er `daq-e-<id>.local` (f.eks. `daq-e-def330.local`); dens rene IP-adresse fungerer også.
{% endhint %}

## `pool-discover` — scanning uden at oprette forbindelse

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Scanner efter sensorer, du kan oprette forbindelse til, og viser den adresse, du skal bruge til at oprette forbindelse, uden at oprette forbindelse. `--only` tager en kommasepareret delmængde af `usb,ble,eth`; `--timeout SEC` indstiller scannetiden pr. transport (standard 5 s, begrænset til 1–20 s). Sensorer, der allerede er åbne i puljen, vises ikke (en tilsluttet DAQ-M holder op med at annoncere) — brug `pool-list` til disse. I en leveret build kører `chloros-cli daq discover` og `daq list` denne samme scanning.

## `pool-list` — se, hvad der er tilsluttet

Viser alle sensorer i backend-poolen, herunder den `sensor_id`, som alle andre kommandoer har brug for:

| Model | `sensor_id`-form | Eksempel |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktet med bindestreg | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

En DAQ-E, hvis PTP er blevet slået fra (indstillingen gemmes på enheden), markeres i `pool-list` med `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` og kommandoen, der slår den til igen — se [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — læs spektrumrammer

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Returnerer den seneste ramme eller de seneste `--recent N`-rammer; `--json` genererer maskinlæsbar output til scripting. Rammerne er radiometrisk kalibrerede spektrale irradiansværdier (W/m²/nm) på et 135-punkts gitter i området 340–1010 nm, hvor sensorens cap-profil allerede er anvendt. For at opnå kvantitative strålingsværdier skal der beregnes et gennemsnit af mindst 15 sekunders billeder — dette er en egenskab ved instrumentet, ikke en fejl.

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
| `--duration SEC` / `-d SEC` | `0` | Optagelsens længde i sekunder; `0` betyder, at optagelsen fortsætter, indtil du udfører `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Output-mappe. Med backend på denne computer opløses en relativ sti i forhold til den mappe, hvor kommandoen blev kørt; med en fjernbackend opløses stien **på den maskine, der kører backend**. |
| `--device-name NAME` | — | Etiket, der gemmes sammen med optagelsen. |
| `--stop` | — | Stopper en igangværende optagelse. |
| `--confirm-after SEC` | `20` | Efter opstart skal der ventes op til denne tid, indtil den første række er skrevet, og der skal gives en tydelig advarsel (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`), hvis der ikke kommer nogen. `0` deaktiverer kontrollen. |
| `--heartbeat SEC` | `30` | Med `--duration` udskrives antallet af rækker hvert SEC. sekund, og der gives en advarsel, hvis det holder op med at stige. `0` deaktiverer den. |

{% hint style="info" %}
Optagelsen foregår i backend, så filen `.daq` havner på **backend-maskinens** filsystem — som standard i `~/Documents/DAQ Live View/` der, ikke nødvendigvis der, hvor du kørte CLI'en. Filnavnene indeholder sensor-ID'et og et tidsstempel.
{% endhint %}

## `pool-set-cap` — angiv den monterede hætte

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

Hætten-ID'et vælger den fabriksmålte korrektionsprofil, der anvendes på hvert spektrum, og den **skal matche den hætten, der fysisk er monteret på sensoren** — hverken sensoren eller softwaren kan selv registrere hætten, og valget er indstemplet i hver `.daq`-fil. Standardindstillingen overalt er `sunshine_cosine` (alle DAQ-enheder leveres med Sunshine-kosinuskorrektionshætten monteret, ~12× dæmpning ifølge designet — et udeklareret skift af hætten medfører en forkert korrektion af spektrene med omtrent denne faktor).

| `--cap-id` | Tilgængelig på |
| --- | --- |
| `sunshine_cosine` (standard) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Kun DAQ-U |
| `none` | Kun DAQ-E — se bemærkning |

Et dæksel-ID, der ligger uden for sensorens indstillingsområde, afvises ved tilslutning med en tydelig fejlmeddelelse. `none` (DAQ-E) betyder, at hætten er fysisk fjernet — den anvender stadig en fabriksgeometriprofil til DAQ-E's forsænkede glasdiffusor, så det er ikke en no-op, og en bar DAQ-E er en bordkonfiguration, ikke en understøttet feltkonfiguration. (En DAQ-U uden kappe er helt uden kappe og behøver slet ingen korrektionsprofil; DAQ-M bruges sammen med sin Sunshine-kappe.)

## `pool-disconnect` — frigivelse af sensorer

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` frigiver en sensor, uanset hvem der har åbnet den, så den frigiver også en sensor, som et nedbrudt script har efterladt i puljen (en optagelse på den skal først være afsluttet). En sensor, der ikke er i puljen, har intet at frigive: kommandoen udskriver `<sensor-id>: already released (not in the pool)` og afsluttes med `0`. En frigivelse, som backend'et afviste eller ikke bekræftede, afsluttes med `1`.

## `pool-enable-ptp` — slå en DAQ-E's PTP til igen

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

En DAQ-E gemmer sin PTP-tilstand (tændt/slukket) i sin egen hukommelse, så en enhed, hvis PTP er blevet slukket, forbliver slukket — og dens tidsstempler er ikke PTP-synkroniserede — indtil den tændes igen. `pool-list` markerer en sådan enhed og udskriver denne kommando for den. Chloros tænder aldrig en enhed igen af sig selv, da den kan være slukket bevidst. Nyt i 1.3.0.

## DAQ-E-enhedsindstillinger

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` viser sensorens firmware (og om denne Chloros-udgave indeholder en nyere version), dens adresse og hvordan den fik den, den gemte netværksindstilling, hvis den adskiller sig fra den aktuelle, streaming ved opstart og PTP. `pool-set-network` genstarter sensoren for at anvende ændringen og genopretter forbindelsen med de indstillinger, den havde; `--no-reboot` gemmer den blot. Der skal være en statisk adresse på denne computers netværk, ellers afviser Chloros den; `--force` tilsidesætter dette, men kun for en computer, der vil blive omadresseret eller dirigeret til den. Genstart afvises, mens sensoren registrerer data. `pool-update-firmware` svarer til GUI'ens **Update to \<version\>** for kommandolinjen og er metoden til at opdatere en DAQ-E på Linux, som ikke har en GUI. Nyt i 1.3.0.***

## Kommandooversigt

| Kommando | Formål |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Åbn en sensor i backend-puljen. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Scanner efter sensorer uden at oprette forbindelse (finder en DAQ-M's BLE MAC). |
| `daq pool-list` | Vis alle sensorer i puljen med deres `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | De seneste N kalibrerede spektrumrammer. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Genoptag/sæt streaming på pause. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Start/stop en `.daq`-optagelse (backend-side). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Skift cap-korrektionsprofilen under kørsel. |
| `daq pool-disconnect --sensor-id ID [--all]` | Frigiv én sensor eller dem alle. |
| `daq pool-enable-ptp --sensor-id ID` | Slå PTP til igen for en DAQ-E, hvis PTP er slået fra. |
| `daq pool-info --sensor-id ID [--json]` | En DAQ-E's firmware, adresse, gemte netværksindstillinger, streaming ved opstart og PTP. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP eller en statisk adresse; genstarter og genopretter forbindelsen til sensoren. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Streaming ved opstart. |
| `daq pool-reboot --sensor-id ID` | Genstarter en DAQ-E og opretter forbindelse igen. |
| `daq pool-update-firmware --sensor-id ID` | Flasher denne udgaves DAQ-E-firmware til en ældre sensor. |

***

## Fejlfinding ved første DAQ-E-forbindelse

1. DAQ-E har ingen status-LED — kontroller strømforsyningen via PoE/link-indikatoren på switchen eller injektorporten, og vent et par sekunder efter opstart, så den kan starte op og oprette forbindelse til netværket.
2. Backend-computeren skal være på **samme broadcast-domæne** som sensoren — mDNS passerer ikke routere.
3. En firewall skal tillade mDNS (UDP 5353), DAQ-E-data (UDP 5002) og PTP (UDP 319/320). Windows-installationsprogrammet tilføjer en regel til dette, og Linux-versionen af `.deb` åbner dem i en `ufw` / `firewalld`, der var aktiv på installationstidspunktet; en tredjeparts firewall eller en, der aktiveres senere, skal tillade disse.
4. Stadig intet fra `--eth`? Brug `--eth-host` med enhedens værtsnavn (`daq-e-<id>.local`) eller IP-adresse, hvilket springer opdagelsen over.

***
{% hint style="info" %}
**Tip til AI-assistenter.** Hver side i denne manual leveres som rå Markdown — tilføj `.md` til en sides URL-slug i små bogstaver (denne side: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); det maskinlæsbare indeks er `https://mapir.gitbook.io/chloros/llms.txt`. For fuldstændig dokumentation på flag-niveau af `chloros-cli daq` og alle andre kommandofamilier, hent [CLI-referencen](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); Python-stien er `chloros_sdk.connect_daq_sensor()` i [SDK-referencen](../reference/sdk-reference.md).
{% endhint %}
</version\>

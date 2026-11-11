# DAQ-E-netværk og tidssynkronisering

> Den fysiske netværksopsætning af sensoren — kabelføring, PoE, IP-tildeling og enhedens egne netværksindstillinger — findes i **[DAQ-brugervejledningen](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Denne side dækker Chloros-siden: tilslutning, tidssynkronisering og hvad man skal gøre, hvis opdagelsen ikke giver noget resultat.

DAQ-E er Ethernet-medlemmet af DAQ-familien: den får strøm via PoE, opdages via mDNS (tjeneste `_daq-e._tcp`) og kan adresseres med et værtsnavn, der er afledt af dens sensor-ID — `daq-e-<6 hex>.local`, f.eks. `daq-e-def330.local`. Denne side beskriver, hvordan den overfører data på netværket, og hvordan den deltager i PTP-tidssynkronisering.

## Transporttilstande

| Tilstand | Endepunkt | Brugere | Bemærkninger |
| --- | --- | --- | --- |
| **Multicast** (standard) | UDP `239.10.10.10:5002` | Et vilkårligt antal enheder på samme LAN modtager den samme strøm | Hvert datagram er CRC-16/CCITT-valideret |
| **Raw** | TCP-port `5000` | Præcis én klient (eksklusivt) | Drop-in-byte-kompatibel med DAQ-U |

Chloros bruger multicast som standard, hvilket gør det muligt for GUI, CLI og SDK alle at overvåge én sensor på samme tid.

## Netværkskrav

* **Samme broadcast-domæne.** Den maskine, der kører Chloros, skal befinde sig på det samme L2-netværkssegment som sensoren — mDNS-opdagelse passerer ikke routere.
* **Firewall.** På Windows tilføjer installationsprogrammet firewall-reglen **Chloros Backend (UDP-In)**til backend i alle netværksprofiler, hvilket dækker DAQ-E-data (UDP 5002), mDNS (UDP 5353) og PTP (UDP 319/320), samt en**Chloros Backend (TCP-In)**-regel, der er begrænset til det lokale undernet, til den forbindelse, som en DAQ-E opretter tilbage til computeren under en firmwareopdatering. På Linux åbner `.deb` UDP-portene i `ufw` eller `firewalld`, når denne firewall er aktiv ved installationen. En tredjeparts-firewall eller en, der aktiveres senere, kræver, at de samme porte er tilladt.
* **PoE-strøm, ingen status-LED.** DAQ-E har ingen egen LED — kontroller strømmen via link/PoE-indikatoren på switchen eller injektorporten, og vent et par sekunder efter opstart, indtil enheden er startet op og har tilsluttet sig netværket.

## Tilslutning

**GUI:** Fanen Lyssensorer → Tilslut sensor → Enhedstype "DAQ-E (Ethernet)". Søgningen kører kun, mens tilslutningsdialogen er på skærmen (mDNS-gennemgang plus en ARP-søgning på Windows), og gentages hvert 15. sekund; knappen Opdater gen scanner straks. Fundne sensorer vises i rullemenuen; den første registrerede sensor vælges automatisk.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend kører):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

En DAQ-E, der allerede findes i puljen, genkendes både på sit værtsnavn og sin IP-adresse: Hvis den tilsluttes igen under den anden adresse, returneres den eksisterende puljepost i stedet for at enheden oprettes en ny gang.

### Værter med flere netværkskort og den første tilslutning efter opstart

På værter med mere end ét aktivt netværksinterface kan den **første** `pool-connect --eth` efter opstart være tom, selvom sensoren fungerer korrekt — søgningen kan overse det interface, sensoren befinder sig på, mens ARP-cachen stadig er tom. Den pålidelige løsning er at springe opdagelsen over og angive adressen eksplicit:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` accepterer mDNS-værtsnavnet eller IP-adressen, retter sig altid mod den rigtige sensor og er den anbefalede form til scripts og headless-installationer. I GUI'en skal du bruge knappen »Refresh« i forbindelsesdialogen og give tid til en ny scanning.

## Enhedsindstillinger og firmware

Selve sensoren gemmer sine netværksindstillinger: DHCP (fabriksindstillingen, der falder tilbage til en 169.254.x.x link-local-adresse, når ingen DHCP-server svarer) eller en statisk adresse. Den gemmer også, om den starter streaming ved opstart. Du kan ændre disse indstillinger i GUI'en (sensorens tandhjulsindstillinger, **Netværk**:**Skift…**,**Stream ved opstart**,**Genstart sensor**; se [Fanen DAQ](gui.md)), eller fra CLI og SDK med `pool-set-network`, `pool-set-auto-stream` og `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; se [CLI-referencen](../reference/cli-reference.md#daq-e-device-settings)). Sensoren genstarter for at gennemføre en netværksændring, og Chloros opretter forbindelse til den igen. Der skal være en statisk adresse på netværket for den computer, der kører Chloros: Chloros afviser en adresse, som den ikke kunne nå efter genstarten. Sensorens ID og dens `daq-e-<id>.local`-værtsnavn ændres aldrig. Nyt i 1.3.0.

**Firmwareopdateringer er integreret i GUI'en.** Når en tilsluttet DAQ-E kører en firmware, der er ældre end den, der følger med din Chloros-version, vises der en gul **Update Available**-pille i sensorens række, og i indstillingsvinduet med tandhjulet vises en "Update to <version>"-knap. Opdateringen overføres via netværket på ca. 30 sekunder; sensoren genstarter og opretter automatisk forbindelse igen, og en afbrudt overførsel efterlader den aktuelle firmware intakt. Fra CLI eller SDK og på Linux, hvor der ikke er nogen GUI, skal du køre `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP-tidssynkronisering

DAQ-E-firmware v1.2.0+ deltager i IEEE 1588 PTPv2 som et almindeligt (kun slave) ur. **Chloros-hostens backend er PTP-grandmasteren** — alle DAQ-E-enheder og alle LATTICE-kameraer på LAN'et fungerer som slaver til den i domæne 0, hvilket holder alle enheders tidsstempler inden for en tolerance på ca. 1 ms. Det er denne fælles ur, der gør det muligt at matche DAQ-målingernes tidsstempler med kameraets eksponeringer (se [Optagelse og .daq-formatet](recording.md)).

Kontroller synkroniseringen fra CLI'en:

| Kommando | Viser |
| --- | --- |
| `chloros-cli time-sync status` | Værts-grandmaster-tilstand, BMCA-prioriteter, uridentitet og `health`: `OK` eller `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` med en `health_detail`, der angiver, hvorfor dens pakker ikke når frem til netværket |
| `chloros-cli time-sync peers` | Alle registrerede slaver (DAQ-E-sensorer + LATTICE-kameraer) |
| `chloros-cli time-sync cameras` | PTP-tilstand pr. kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Genstart grandmaster-processen |

I brugergrænsefladen viser DAQ-E-indstillingsvinduet en live **PTP Sync**-række med sensorens aktuelle PTP-tilstand.**En DAQ-E med PTP slået fra forbliver slået fra.** Enheden gemmer sin PTP-indstilling (til/fra) i sin egen hukommelse, så en enhed, hvis PTP var slået fra, forbliver slået fra efter en genforbindelse eller en genstart, og dens tidsstempler er ikke PTP-synkroniserede. `chloros-cli daq pool-list` markerer en sådan enhed (`!! PTP is switched OFF on this unit …`) og udskriver den kommando, der løser problemet:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros slår aldrig en enhed til igen af sig selv, da den kan være slukket bevidst.

Detaljer for brugere, der kræver streng synkronisering:

* Hvert streamet datagram indeholder et flagfelt; **bit 2 er sat på rammer, hvis tidsstempel er PTP-synkroniseret**. Pipelines, der kræver streng synkronisering mellem kamera og DAQ, bør basere deres filtrering på dette bit.
* Før en synkroniseret optagelse skal du kontrollere, at sensoren vises i `chloros-cli time-sync peers`, og at `chloros-cli daq pool-list` ikke markerer dens PTP som slukket. (MAPIRs interne direkte hardwareværktøj kan også styre optagelsen efter PTP-låsning med et `--wait-ptp`-flag, der venter op til 15 sekunder på, at sensoren når SLAVE-tilstand; dette værktøj er ikke en del af den medfølgende CLI.)
* Mens PTP aktivt fungerer som slave, afviser sensoren manuelle klokkesignaler („PTP leverer klokkesignal”). Dette er med vilje — stol på PTP.

## Bemærkninger til Linux

* **PTP-porte uden root-rettigheder.** PTP binder til UDP 319/320. `.deb` skriver `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), så backend'en binder dem, så de kører som din bruger. Fra og med version 1.3.0 indstiller pakken ingen filkapacitet (`setcap`) på backend'en og har ikke brug for `libcap2-bin`; tilføj ikke selv en kapacitet, da den udpakkede backend i så fald ikke starter. Hvis PTP ikke kan binde port 319, se [Linux-installation](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Headless Jetson / Linux-vært:** Pakken installerer backend-tjenesterne, men aktiverer ingen af dem. For at sikre, at PTP (og DAQ-tilgængelighed) altid er tændt, uden at nogen kører en kommando, skal du aktivere tjenesten som din bruger:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Uden dette kører PTP kun, mens noget har startet backend-tjenesten — en `chloros-cli`-kommando eller et SDK-script. Se [Linux-installation](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) for root-brugeren `chloros-backend.service` og hvordan man skifter væk fra denne.

## Fejlfinding: "Ingen DAQ-E-enheder fundet"

| Kontrol | Detaljer |
| --- | --- |
| Strøm | Ingen LED på sensoren — kontroller PoE- og link-indikatorerne på switchen/injektorporten; vent et par sekunder efter opstart |
| Broadcast-domæne | Vært og sensor er på samme L2-segment; mDNS routes ikke |
| Firewall | Windows: Installationsprogrammets regel **Chloros Backend (UDP-In)** tillader DAQ-E-stream (UDP 5002), mDNS (5353) og PTP (319/320) at komme ind; en tredjeparts-firewall skal have de samme indstillinger. Linux: `.deb` åbner disse porte i `ufw` / `firewalld` kun, hvis firewallen var aktiv under installationen — tillad dem selv, hvis den aktiveres senere |
| Vært med flere netværkskort | Opdagelsen gennemsøger mDNS med ARP som reserve; `--eth-host <ip-or-hostname>` springer opdagelsen over og opretter forbindelse direkte |
| Gen-scanning via GUI | Opdagelsen kører kun, mens forbindelsesdialogen er åben; brug Opdater |</version>

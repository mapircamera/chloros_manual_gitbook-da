# DAQ-E-netværk og tidssynkronisering

> Den fysiske netværksopsætning af sensoren — kabelføring, PoE, IP-tildeling og enhedens egne netværksindstillinger — findes i **[DAQ-brugervejledningen](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Denne side dækker Chloros-delen: tilslutning, tidssynkronisering og hvad man skal gøre, hvis opdagelsen ikke giver noget resultat.

DAQ-E er Ethernet-medlemmet af DAQ-familien: den får strøm via PoE, opdages via mDNS (tjenesten `_daq-e._tcp`) og kan adresseres med et værtsnavn, der er afledt af dens sensor-ID — `daq-e-<6 hex>.local`, f.eks. `daq-e-def330.local`. Denne side beskriver, hvordan den overfører data på netværket, og hvordan den deltager i PTP-tidssynkronisering.

## Transporttilstande

| Tilstand | Endepunkt | Brugere | Bemærkninger |
| --- | --- | --- | --- |
| **Multicast** (standard) | UDP `239.10.10.10:5002` | Et vilkårligt antal enheder på samme LAN modtager den samme strøm | Hvert datagram er CRC-16/CCITT-valideret |
| **Raw** | TCP-port `5000` | Præcis én klient (eksklusiv) | Direkte kompatibel med DAQ-U på byte-niveau |

Chloros bruger multicast som standard, hvilket gør det muligt for GUI&#x27;en, CLI og SDK alle at overvåge én sensor på samme tid.

## Netværkskrav

* **Samme broadcast-domæne.** Den maskine, der kører Chloros, skal være på det samme L2-netværkssegment som sensoren — mDNS-opdagelse passerer ikke routere.
* **Windows-firewall-advarsel: Accepter den.** Første gang Chloros binder multicast-sockets, spørger Windows Defender én gang. Hvis du tillader det, dækker det DAQ-E-data (UDP 5002), mDNS (UDP 5353) og PTP (UDP 319/320). På Linux sker dette uden besked.
* **PoE-strøm, ingen status-LED.** DAQ-E har ingen egen LED — kontroller strømmen via link/PoE-indikatoren på switchen eller injektorporten, og vent et par sekunder efter opstart, indtil den er startet op og har tilsluttet sig netværket.

## Tilslutning

**GUI:** Fanen Lyssensorer → Tilslut sensor → Enhedstype &quot;DAQ-E (Ethernet)&quot;. Søgningen kører kun, mens tilslutningsdialogen er på skærmen (mDNS-søgning plus en ARP-søgning på Windows), og gentages hvert 15. sekund; knappen »Opdater« udfører en ny søgning med det samme. Fundne sensorer vises i rullemenuen; den første registrerede sensor vælges automatisk.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend kører):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Værter med flere netværkskort og den første tilslutning efter opstart

På værter med mere end ét aktivt netværksinterface kan den **første** `pool-connect --eth` efter opstart være tom, selvom sensoren fungerer korrekt — opdagelsessøgningen kan overse det interface, sensoren befinder sig på, mens ARP-cachen stadig er kold. Den pålidelige løsning er at springe opdagelsen over og angive adressen eksplicit:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` accepterer mDNS-værtsnavnet eller IP-adressen, målretter altid mod den rigtige sensor og er den anbefalede form til scripts og headless-installationer. I GUI’en skal du bruge knappen »Refresh« i forbindelsesdialogen og lade en genscanningscyklus køre.

## Enhedsindstillinger og firmware

Selve sensoren indeholder netværksindstillinger — statisk IP kontra DHCP + link-local-adressering, enhedsnavn, automatisk streaming ved opstart, OTA-adgangskode. Disse enhedsindstillinger er ikke tilgængelige som kommandoer i den medfølgende CLI; de administreres via Chloros-brugergrænsefladen, hvor de vises, eller via MAPIR-support.

**Firmwareopdateringer er integreret i GUI&#x27;en.**Når en tilsluttet DAQ-E kører en firmware, der er ældre end den, der følger med din Chloros-build, vises der en gul**Update Available**-pille i sensorrækken, og i modalvinduet med tandhjulsindstillingerne vises en &quot;Update to<version>

&quot;-knap. Opdateringen overføres via netværket på ca. 30 sekunder; sensoren genstarter og opretter automatisk forbindelse igen, og en afbrudt overførsel efterlader den aktuelle firmware intakt.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## PTP-tidssynkronisering

DAQ-E-firmware v1.2.0+ deltager i IEEE 1588 PTPv2 som en almindelig (kun slave) ur. **Chloros-hostens backend er PTP-grandmasteren** — alle DAQ-E-enheder og alle LATTICE-kameraer på LAN&#x27;et fungerer som slaver i domæne 0 og holder alle enheders tidsstempler inden for en tolerance på ca. 1 ms. Det er denne fælles ur, der gør det muligt at matche DAQ-målingernes tidsstempler med kameraets eksponeringer (se [Optagelse og .daq-formatet](recording.md)).

Kontroller synkroniseringen fra CLI:

| Kommando | Viser |
| --- | --- |
| `chloros-cli time-sync status` | Værts-grandmaster-tilstand, BMCA-prioriteter, uridentitet |
| `chloros-cli time-sync peers` | Alle registrerede slaver (DAQ-E-sensorer + LATTICE-kameraer) |
| `chloros-cli time-sync cameras` | PTP-tilstand pr. kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Genstart grandmaster-processen |

I GUI&#x27;en viser modalvinduet med DAQ-E-indstillinger en live **PTP Sync**-række med sensorens aktuelle PTP-tilstand.

Detaljer for brugere, der kræver streng synkronisering:

* Hvert streamet datagram indeholder et flagfelt; **bit 2 er sat på rammer, hvis tidsstempel er PTP-synkroniseret**. Pipelines, der kræver streng synkronisering mellem kamera og DAQ, bør basere sin dataindlæsning på dette bit.
* Før en synkroniseret optagelse skal du kontrollere, at sensoren vises i `chloros-cli time-sync peers`. (MAPIR’s interne direkte hardwareværktøj kan også styre optagelsen ved PTP-låsning med et `--wait-ptp`-flag, der venter op til 15 sekunder på, at sensoren når SLAVE-tilstand; dette værktøj er ikke en del af den leverede CLI.)
* Mens PTP aktivt fungerer som slave, afviser sensoren manuelle klokkesignaler (&quot;PTP leverer klokkesignal&quot;). Dette er med vilje — stol på PTP.

## Bemærkninger til Linux

* **PTP kræver `libcap2-bin` ved installationen.** `.deb` postinst tildeler `cap_net_bind_service=+ep` på `/usr/lib/chloros/chloros-backend`, så den kan binde PTP-portene 319/320 uden root-rettigheder. Hvis `libcap2-bin` mangler, springes dette trin over, og PTP vil ikke kunne starte. Løsning:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Headless Jetson / Raspberry Pi:** Ved første installation genereres systemd-enheden `chloros-backend.service`, men den er ikke aktiveret. For PTP, der altid er tændt (og DAQ-tilgængelighed) uden GUI:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Uden denne kører PTP kun, mens Chloros-GUI’en er åben.

## Fejlfinding: »Ingen DAQ-E-enheder fundet«

| Kontrol | Detaljer |
| --- | --- |
| Strøm | Ingen LED på sensoren — kontroller PoE- og link-indikatorerne på switchen/injektorporten; vent et par sekunder efter opstart |
| Broadcast-domæne | Vært og sensor på samme L2-segment; mDNS router ikke |
| Windows-firewall | Accepter Defender-prompten ved første kørsel (UDP 5002, 5353, 319/320) |
| Vært med flere netværkskort | Første opdagelse efter opstart kan muligvis overse sensoren — opret forbindelse med `--eth-host <ip-or-hostname>` |
| Gen-scanning via GUI | Opdagelse kører kun, mens forbindelsesdialogen er åben; brug Opdater |</version>

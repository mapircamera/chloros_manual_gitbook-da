# Fanen »DAQ« i Chloros

Fanen DAQ — mærket **Lyssensorer** i sidepanelet i Chloros — er den interaktive kontrolflade til [DAQ-U-, DAQ-M- og DAQ-E-lyssensorer](README.md): Tilslut sensorer via enhver transportprotokol, se kalibrerede spektre i realtid, beregn refleksionsgrad i realtid fra et sensorpar, og gem `.daq`-filer direkte i dit projekt.

Fanen bliver tilgængelig, så snart Chloros-backend er færdig med at starte. Fanebladets diagrammer forsynes af Chloros’s DAQ-tjeneste via en live-forbindelse, der genoprettes automatisk (2–10 sekunders ventetid) ved afbrydelse; mens tjenesten er utilgængelig, vises der **Ingen server** i sensorens statusrække.

Layoutet består af en **sensor-sidebjælke**(én række pr. tilsluttet sensor) samt et**diagramområde** (én diagramflise pr. sensor eller gruppe).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Tilslutning af en sensor

Klik på **Tilslut sensor** øverst i sidepanelet. Tilslutningsdialogen åbnes i hovedområdet (eller som et overlay, når der tilføjes en anden sensor – i så fald vises en Annuller-knap).

| Kontrol | Adfærd |
| --- | --- |
| **Enhedstype** | `DAQ-U (USB)` (standard), `DAQ-M (Bluetooth)` eller `DAQ-E (Ethernet)`. Ved at skifte genstartes scanningen for den nyvalgte transport. |
| **Port / BLE-enhed / Værtsnavn / IP** | Viser fundne enheder som `device - description`; den første post, der genkendes som en sensor, vælges automatisk. Under scanningen vises `Scanning...` (USB), `Scanning (N)...` med en nedtælling på 8 sekunder (BLE) eller `Discovering ethernet sensors (N)...` med en nedtælling på 5 sekunder (Ethernet). Tomme resultater vises som `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Opdater** | Scanner den valgte transportform igen med det samme (deaktiveret midt i en BLE-/Ethernet-scanning). |
| **Tilslut** | Aktiveres, når en enhed er valgt; omdøbes til `Connecting...`, mens forbindelsen oprettes. |

Søgningen kører kun **mens forbindelsesdialogen er på skærmen** og gentages hvert 15. sekund for den valgte transport — blot at åbne fanen udløser ikke en scanning. Ved en fejl viser dialogen: *&quot;Forbindelsen mislykkedes. Prøv at tage sensoren ud og sætte den i igen, og klik derefter på Tilslut igen.&quot;*

Sidepanelet åbnes automatisk, når din første sensor opretter forbindelse.

{% hint style="info" %}
**Vises DAQ-E ikke?** DAQ-E har ingen status-LED — tjek PoE/link-indikatoren på den switch eller injektorport, den er tilsluttet, og vent et par sekunder efter opstart, så den kan starte op. Chloros-enheden skal være på samme broadcast-domæne (mDNS passerer ikke routere). På Windows skal du acceptere Defender-firewall-prompten, første gang Chloros binder sine multicast-sockets (mDNS UDP 5353, DAQ-E-data UDP 5002, PTP UDP 319/320). To DAQ-E-enheder på ét LAN opdages hver for sig, hver under sit eget `daq-e-<id>.local`-værtsnavn.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Enhedstype tilbyder DAQ-U (USB), DAQ-M (Bluetooth) og DAQ-E (Ethernet)</figcaption></figure>***

## Sensor-sidepanelet

Hver tilsluttet sensor får en række (plus én række pr. Ambient+Object-gruppe). Rækkerne kan omarrangeres ved at trække dem, og deres rækkefølge ændrer også rækkefølgen af diagramfelterne. Klik på en række for at gøre den pågældende sensor/gruppe til det aktive diagram i listevisningen.

| Element | Betydning |
| --- | --- |
| Farvet venstre kant | Sensorens graf-farve. |
| Transport-badge | `DAQ-U` / `DAQ-M` / `DAQ-E` eller et grønt `REF`-mærke for en Ambient+Object-reflektansgruppe. |
| Enhedsnavn | Standardindstillingen er sensorens serienummer (dens faste identitet til kalibrering, `.daq`-filnavne og importmatchning); brugerdefinerede navne bevares pr. projekt. |
| **Kalibreret**-pille (grøn) | Vises, når sensorens fabrikskalibreringspakke er indlæst, dvs. at spektrene er angivet i W/m²/nm. |
| **Opdatering tilgængelig**-pille (ravfarvet, kun DAQ-E) | Den kørende firmware er ældre end den, der følger med denne Chloros-build. Under en opdatering vises fremskridtet i realtid (`Flashing… N%`, `Restarting sensor…`, derefter `Updated X → Y` eller `Failed`). |
| Øje | Skifter mellem, om denne sensor skal vises på diagrammet. |
| Tandhjul | Åbner modalvinduet med indstillinger for den enkelte sensor (nedenfor). |
| ✕ (rød) | Afbryder forbindelsen til sensoren eller fjerner en Ambient+Object-gruppe. |

Over rækkerne findes to knapper:

* **Tilslut sensor** — åbner dialogboksen for tilslutning (omdøbes til `Connecting...`, mens den er i gang).
* **Optag alt / Stop alt**— starter eller stopper en `.daq`-optagelse på**alle**tilsluttede sensorer. Kræver mindst én sensor**og et åbent projekt** (værktøjstip: »Åbn et projekt for at optage«); knappen bliver rød, mens en optagelse kører.

I tom tilstand vises teksten »Ingen tilsluttede sensorer«.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Indstillinger pr. sensor (tandhjulsvinduet)

Åbnes med tandhjulsikonet i en sensorlinje. Indhold i rækkefølge:

* **Info-linjer** — Enhedstype (DAQ-U/M/E), Forbindelse (`Serial (USB)` / `Bluetooth` / `Ethernet`), port (COM-port, BLE-adresse eller vært) og serienummer.
* **Kalibreringsrapport: Download** — henter denne enheds NIST-sporbare kalibreringscertifikat (PDF) og åbner det i din PDF-fremviser. Tilgængelig, når serienummeret er kendt; certifikatet gemmes i cachen ved første forbindelse.
* **Enhedsnavn** — klik på blyanten for at omdøbe; gemmes pr. projekt.
* **Graflinjefarve** — farveprøve; gemmes pr. projekt.
* **Integrationstid (ms)**— skydeknap + tal,**1–500 ms**, standard**32 ms**. Deaktiveret, mens AE er slået til.
* **Billedgennemsnit**— skydeknap + tal,**1–50 billeder**, standard**20**.
* **AE: TIL/FRA**— kontakt til automatisk eksponering;**standardindstilling TIL** ved tilslutning. Slå den fra for at indstille integrationstiden manuelt.
* **Stop streaming / Start streaming** — sæt live-streamingen på pause eller genoptag den.
* **Optag / Stop optagelse** — `.daq`-optagelse pr. sensor (kræver et åbent projekt).
* **Cap** — cap-korrektionsprofilen (næste afsnit).
* **Live-informationsrækker** — Integrationstid (ms), FPS, Prøver, Optagelse (rød `REC` eller `Off`) og Status (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Kun DAQ-E: rækkerne netværk, firmware og PTP

* **Værtsnavn / IP** — enhedens aktuelle adresse.
* **Firmware**— den aktuelle firmwareversion samt en handlingscelle: en**Opdater til \<version\>

**-knap vises, når denne Chloros-build indeholder et nyere DAQ-E-firmwarebillede. Opdateringen overføres via netværket på ca. 30 sekunder; sensoren genstarter og opretter automatisk forbindelse igen, og en afbrudt overførsel efterlader den aktuelle firmware intakt. Fremskridtet vises i realtid (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), og feltet viser `Up to date`, når den er opdateret.
* **PTP-synkronisering** — den aktuelle PTP-tilstand (falder tilbage til `unknown`). DAQ-E-firmware v1.2.0+ deltager i IEEE 1588 PTPv2 som et rent slaveur; backend&#x27;en på Chloros-værten er PTP-grandmasteren, og alle DAQ-E- og LATTICE-kameraer på LAN&#x27;et er slaver til den i domæne 0, hvilket holder tidsstemplerne inden for ca. 1 ms.

For en »Ambient+Object«-gruppe viser gear-modalen kun gruppens kildesensorer, enhedsnavn og graflinjens farve.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### Valg af kappe

Rullemenuen **Cap** fortæller Chloros, hvilket fysisk dæksel der er monteret over sensorens diffusor, og anvender det pågældende dæksels fabriksmålte korrektionsprofil på hvert spektrum. Valgmulighederne afhænger af modellen:

| Model | Dækselvalg |
| --- | --- |
| DAQ-U | Ingen (bar sensor), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (cosinus-korrektor) |
| DAQ-M | Ingen (bar sensor), Sunshine (kosinuskorrektor) |
| DAQ-E | Ingen (bar sensor), synsfelt 15°, synsfelt 45°, synsfelt 90°, Sunshine (kosinuskorrektor) |

**Standardindstillingen for alle modeller er Sunshine (cosinus-korrektor)** — MAPIR leverer alle DAQ-enheder med Sunshine-hætten monteret, og dette er standardkonfigurationen til udendørs brug: et 180° halvkugleformet udsyn med cosinusfejl på ≤ ±4 % op til 60° og ≤ ±4,5 % op til 70° (anbefales ikke ved solhøjde under ca. 15°), med indbygget dæmpning (~12×). Dit valg gemmes i projektet.

{% hint style="warning" %}
**Valget af kappe skal stemme overens med den fysiske kappe.**Hverken sensoren eller softwaren kan registrere, hvilken kappe der er monteret. Valget styrer både den løbende korrektion og den mærkning, der skrives ind i hver `.daq`-fil — med Sunshine-kappens ~12× dæmpning vil et udeklareret kappe-skift medføre en forkert korrektion af spektrene med omtrent denne faktor. (Fjernelse og genmontering af den samme kappe gentager sig til ca. 1,5 %.) Vælg kun**Ingen (bar sensor)**, når hætten er fysisk fjernet; på en DAQ-E anvender »Ingen« stadig en fabriksgeometriprofil for dens forsænkede glasdiffusor — det er ikke en no-op — og en bar DAQ-E er en bordkonfiguration, ikke en understøttet feltkonfiguration.
{% endhint %}

{% hint style="info" %}
Opgradering fra en tidligere manual: Vippeknappen »Sunshine Diffuser Installed« i browseren fra version 1.1.0 er fjernet. Håndtering af hætten foregår nu via denne hætteprofil pr. sensor, der anvendes på serversiden.
{% endhint %}

***

## Diagramområdet

En fast øverste bjælke indeholder en **knap til at skifte mellem liste- og gittervisning**og en**diagramzoom**-skyder (flisestørrelse 200–2000 px). Visningen skifter automatisk til gitter, når der findes mere end én diagramgruppe, og tilbage til liste, når der er én eller færre. Visningstilstand og diagramstørrelse gemmes pr. projekt.**Spektrumdiagrammet** for hver sensor viser:

* **X-aksen** — Bølgelængde (nm). Sensorens gitter er 340–1010 nm med 5 nm mellemrum (135 punkter), interpoleret til 1 nm til visning.
* **Y-aksen** — Effekt (W/m²), med et automatisk SI-præfiks (m/µ/n) valgt ud fra toppen. Spektrene er radiometrisk kalibreret spektral irradians (W/m²/nm) på alle tre transportformer.
* En regnbuefarvet spektral udfyldning under en enkelt kurve; flere sensorer på ét diagram vises som farvede linjer med nedtonet udfyldning.
* **Hold markøren over**— en lodret markør med bølgelængde og værdi pr. sensor;**træk** for at zoome (en zoom-ud-knap vises, mens der er zoomet ind).
* En **+**-knap (kun i gittervisning) til at tilføje en sensor til dette diagram eller oprette en gruppe (nedenfor).
* Enhedsnavnet centreret øverst og en spinner, indtil den første ramme ankommer.

**Mætning** er ikke markeret på selve diagrammet: en mættet sensor viser rød `SATURATED`-statustekst og en rød `Saturated: Yes`-række i tabellen med live-data. Sænk integrationstiden eller aktiver AE igen for at rydde den.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## Live-datatabel (listevisning)

Under diagrammet i listevisning, opdateres hver 500 ms:

* **Alle modeller**: Lysfarveprøve (sRGB fra CIE XYZ), Mættet (Ja/Nej), CIE 1931 X/Y/Z, Kromaticitet x/y, CIE u′/v′, CCT (K), CRI (Ra), dominant bølgelængde (nm), spidsbølgelængde (nm), excitationsrenhed, Duv, CIE L\*/a\*/b\* og Munsell H/V/C.
* **Kun kalibrerede sensorer**(enhver af DAQ-U / DAQ-M / DAQ-E, når fabrikskalibreringspakken er indlæst — det grønne**Kalibreret**-mærke på sensorrækken er kendetegnet): Total effekt (W/m²), fotopisk lux (lx), skotopisk lux (lx), S/P-forhold, PPFD samt PPFD Red/Green/Blue (µmol/m²/s), samt de opiske strålingsintensiteter — S-kegle, melanopisk, rhodopisk, M-kegle, L-kegle (alle W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Reflektansgrupper (Omgivelser + Objekt)

To tilsluttede sensorer kan kombineres til en live-visning af reflektansen — uden brug af kamera:

1. I gittervisningen skal du klikke på **+**på en diagramflise og vælge**Kombiner omgivelser + objekt**.
2. Vælg en **Omgivende lyskilde**-sensor og en**Objektscanner**-sensor (to forskellige sensorer), og klik derefter på**Opret**.

Chloros beregner R(λ) = objekt(λ) / omgivende(λ) pr. bølgelængde ud fra de to live-strømme (0, hvor omgivende ≤ 0). Gruppens betegnelse følger sensorernes kalibreringsklasse:

* Begge sensorer er kalibreret (pakke indlæst) → **&quot;Tilsyneladende reflektans&quot;**.
* En af sensorerne er ikke kalibreret → **&quot;Relativ reflektans&quot;**.

Gruppen vises som en grøn `REF`-række i sidepanelet og i sit eget diagram (regnbuefarvet baggrund, værdier vises ved musepeger med 4 decimaler, træk for at zoome).

**+**-menuen tilbyder også**Tilføj ny sensor** med tre placeringsmuligheder: *Kombiner ny sensor* (tilføj til dette diagram), *Flyt eksisterende sensor hertil* eller *Vis ny sensor* (i sit eget diagram).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### Tabel over vegetationsindeks

I listevisningen findes der en tabel over vegetationsindeks under et reflektansgruppes diagram, beregnet ud fra den aktuelle reflektans ved båndcentrene **blå 450 / grøn 550 / rød 670 / NIR 800 nm** (værdier med 4 decimaler, `---`, når de ikke kan beregnes; hold markøren over et indeksnavn for at se dets fulde navn):

* **Vises altid** (skalainvariant, uanset sensorkombination): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Kun når begge sensorer er strømkalibreret** (begge bundter indlæst): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## Optagelse af `.daq`-filer

* Optagelse kræver et **åbent projekt** — ellers er både »Optag alt« (sidepanelet) og optageknappen for den enkelte sensor deaktiveret.
* Filerne gemmes i **`<project folder>/light_sensor/`**; filnavnene indeholder sensor-ID&#x27;et og et tidsstempel, og enhedsnavnet gemmes sammen med optagelsen.
* Når en optagelse stopper (Stop, Stop All eller en afbrydelse midt i optagelsen), **føjes den færdige `.daq` automatisk til det åbne projekt** — den vises i projektets filliste uden manuel tilføjelse, klar til at fungere som nedadgående data til [reflektansbehandling](README.md).
* En rød `REC`-indikator vises i indstillingsmodalens live-rækker under optagelsen.

For kvantitative strålingsstyrketal skal der beregnes et gennemsnit af mindst 15 sekunders data — dette er en egenskab ved instrumentet, ikke en fejl.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## Layout med flere sensorer og projektpersistens

* Kombiner flere sensorer på ét diagram (fælles akser), opret separate diagrammer (automatisk gitterlayout), flyt sensorer mellem diagrammer, træk og omarranger rækker/felter, og skjul individuelle sensorer med øje-knappen.
* For hvert projekt bevares følgende i Chloros: enhedsnavne, graffarver, diagramstørrelse, visningstilstand og hver sensors indstillinger (integrationstid, rammeudjævning, AE-tilstand, cap-valg).
* **Når et projekt genåbnes, oprettes der automatisk forbindelse til dets sensorer** via adressen — COM-port for DAQ-U, BLE-enhed for DAQ-M, mDNS-værtsnavn for DAQ-E (løses selvom enhedens IP-adresse er ændret) — og genanvender hver sensors gemte cap-profil, billedgennemsnit, AE-tilstand og manuel integrationstid.***

## Kameraparring (DLS)

Der er intet, der skal parres. I modsætning til DLS-arbejdsgange med droner, hvor en lyssensor knyttes til et kamera på forhånd, matcher Chloros DAQ-data med billedmateriale i den efterfølgende proces: ved import/behandling interpoleres `.daq`-målingerne til hvert billedes eksponeringstidsstempel. Optag med en hvilken som helst tilsluttet sensor (`.daq` tilføjes automatisk til projektet), og reflektansbehandlingen finder de rigtige målinger efter tid — se [DAQ-lyssensorer](README.md) for at se, hvordan dataene fra nedadgående lysstråling anvendes.</version\>

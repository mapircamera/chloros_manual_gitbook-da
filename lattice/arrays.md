# Multikamera-opstillinger

En LATTICE **opstilling**består af to eller flere LATTICE-kameraer, der er forbundet som én synkroniseret enhed. Ét kamera fungerer som**master**: Det udsender en hardware-GPIO-triggerimpuls på en fælles synkroniseringslinje (som standard**Line2**), så alle kameraer i opstillingen tager billeder på samme tidspunkt. Chloros tilføjer PTP-tidssynkronisering, en live-forhåndsvisning (kamera-specifikke fliser eller et enkelt justeret multibånd-kompositbillede) samt synkroniseret optagelse — hver optagelsesrunde producerer én**billedgruppe**, hvor alle kameraer deler det samme tidsstempel og billed-ID (rapporteret som `fid:N` i optagelsesoutputtet).

Arrays er den måde, hvorpå monokameraer (M3M) genererer vegetationsindekser — ét kamera bidrager med ét bånd, og arrayet justerer dem til en multibåndsstak. Se [Monokameraer og vegetationsindekser](mono-indices.md).

Der er tre ækvivalente måder at tilslutte en array på, og alle kører den samme »smart-prep«-proces:

| Overflade | Indgangspunkt |
| --- | --- |
| GUI | Fanen „Kameraer“ → **Tilslut array** (blå knap) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (første serienummer = master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (første serienummer = master) |

Smart-prep udfører i rækkefølge: en test af netværkskapacitet (ICMP DF-ping + GVSP-test), valg af synkroniseringslag, automatisk tilpasning af rammestørrelsen til ledningen, aktivering af PTP, automatisk valg af pixelformat pr. kamera, automatisk indstilling af eksponering ud fra hvert kameras gemte tilstand samt konfiguration af GPIO-udløser på Line2.

{% hint style="info" %}
Kameraerne skal være tilgængelige på forbindelsen, før noget af dette fungerer — se [Tilslutning af kameraer](connecting.md) for opdagelse, adressering og download af kalibrering ved første tilslutning. For opsætninger med flere kameraer er værts-NIC’ens indstillinger for modtagelsesringen lige så vigtige som forbindelseshastigheden; den komplette tabel over symptomer og løsninger findes i [CLI Reference § Konfiguration og finjustering af værts-NIC](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dialogboksen »Array Connect«

Fanen »Kameraer« → **Connect Array**åbner en tretrinsguide:**Vælg → Visningstilstand → Indstillinger**.

### Trin 1 — Vælg master og slaver

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

Dialogboksen scanner netværket, så snart den åbnes (&quot;Scanner netværk...&quot;), og tester derefter GPIO-triggerledningerne (&quot;Tester GPIO-ledninger...&quot;). Du skal bruge mindst **2 kameraer** for at oprette et array.

Ledningsundersøgelsen udfylder rollevalget på forhånd, når det er muligt, og viser en af følgende tre meddelelser:

| Meddelelse | Betydning |
| --- | --- |
| &quot;GPIO-master fundet — valg udfyldt automatisk&quot; (grøn) | Testen fandt trigger-topologien; afkrydsningsfelterne for master-radio og slave er allerede udfyldt. |
| &quot;Ingen master fundet – tjek GPIO-kablet&quot; (orange) | Intet kamera har registreret en triggerimpuls; tjek synkroniseringskablerne. Du kan stadig vælge roller manuelt. |
| &quot;Intet synkroniseringskabel: {serienumre}&quot; (orange) | De angivne kameraer har ikke noget synkroniseringskabel tilsluttet. |

Kameratabellen har kolonnerne **Kamera / Seriel / IP / Master (radio) / Slave (afkrydsningsfelt)**:

* Vælg nøjagtigt **én master**og**én eller flere slaver**. Hvis du klikker på den aktuelle masters radio igen, ryddes markeringen.
* Et kamera markeret med **&quot;Intet synkroniseringskabel&quot;** kan aldrig vælges som slave — en slave uden triggerforbindelse vil vente på synkroniseringslinjen i al evighed og levere et dødt signal. Tilslut i stedet det pågældende kamera som et selvstændigt kamera.
* Kameraer, der allerede er tilsluttet som enkeltstående, deaktiveres *ikke*: Array-tilslutning frigiver den enkeltstående session og genåbner kameraet inden for arrayet.

**Næste: Visningstilstand →**aktiveres, når en master og mindst én slave er valgt.**Gensøg** kører genfindings- og ledningssøgningen igen.

{% hint style="warning" %}
**Annuller** er deaktiveret, mens en scanning eller ledningskontrol er i gang — annullering midt i ledningskontrollen kan få kameraet SDK til at gå ned på LATTICE-kameraets firmware. Vent, indtil indikatoren er færdig.
{% endhint %}

### Trin 2 — Visningstilstand

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Tilstand | Hvad du får |
| --- | --- |
| **Separate kameraer** | Én live-flise pr. kamera, der alle udløses samtidigt, så billederne forbliver synkroniserede. Hvert kamera beholder sin egen farve og sine egne indstillinger. |
| **Kombinerede kameraer** *(standard)* | En enkelt flise, der gengiver den justerede multibånds NDVI/indeks-sammensætning. Kameraerne deler arrayets farve. |

Visningstilstanden ændrer kun præsentationen af live-forhåndsvisningen — optagelsesadfærden er den samme i begge tilfælde.

### Trin 3 — Array-indstillinger og det forventede resultat

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

Når du går ind i denne scene, beder Chloros backend-systemet om en **anbefaling**og anvender automatisk en kombination af ROI og binning, der passer til dit NIC’s modtagerring (systemet foretrækker binning frem for ROI-beskæring, da binning bevarer det fulde synsfelt). Hver ændring, du foretager, kører analysen live igen og opdaterer panelet**Forventet resultat** til højre.

Venstre kolonne — indstillinger:

| Kontrol | Valgmuligheder | Standard | Bemærkninger |
| --- | --- | --- | --- |
| **ROI (synsfelt)** | Fuld (2048×1536) / Halv (1024×768) / Kvart (512×384) | Fuld | Sensorbeskæring: Halv/kvart beskæring til et mindre område ved oprindelig pixelafstand. |
| **Binning** | 1× / 2× (sum 2×2) / 4× (sum 4×4) | 1× | Hardware-binning: 2×2 = fuldt synsfelt til en fjerdedel af ledningsomkostningerne; 4×4 = fuldt synsfelt til 1/16. Skjult, hvis kameraerne ikke understøtter binning. |
| **Billeddata på ledningen** (udlæsning) | — | — | Den faktiske bredde × højde efter binning, der sendes via ledningen, afrundet til multipler af 16 (minimum 64). |
| **Pin-opløsning**| afkrydsningsfelt | fra | Chloros øger normalt binning automatisk ved tilslutning, når den forventede hastighed falder til under**1,5 fps**. Fastlåsning bevarer den valgte billedstørrelse og accepterer den lavere hastighed — og omdanner en overbelastet konfiguration til en hård forbindelsesafvisning i stedet for en automatisk nedjustering. |
| **Triggerhastighed** | 0,5–60 fps, trin 0,1 | tom = auto | Masterens triggerfrekvens. Lad feltet være tomt for at lade Chloros udlede den. |
| **Wire Budget**| 20–2000 MB/s, trin 10 | tom = auto | Hvor meget værten faktisk kan absorbere, i MB/s —**det eneste tal, som hele array-allokeringen afhænger af.** Registreres automatisk fra netværksadapteren. Sænk den, hvis arrayet rapporterer om korrupte rammer: den registrerede værdi overvurderer USB-adaptere og delte switche. Ændring af denne værdi genkører prognosen i realtid. |

Højre kolonne — **Prognoseret resultat**:

* **Synkroniseringsniveau** — &quot;Samtidig optagelse&quot; (grøn), &quot;Samtidig optagelse (FTD-forskudt udsendelse)&quot; (grøn), &quot;Forskudt optagelse (100 ms forsinkelse)&quot; (gul) eller &quot;Konfigurationen er for stor&quot; (rød).
* **fps-prognose** — vises som et interval (&quot;svag → stærk&quot;), da hastigheden for et synkroniseret array er begrænset af det langsomste kameras eksponeringstid.
* **NIC-linje** — forbindelseshastighed og vedvarende båndbredde (&quot;NIC {mbps} Mbps · vedvarende {N} MB/s&quot;).
* **Sim-emit-burst-kontrol** — kan værtens NIC-ring absorbere én samtidig burst fra alle kameraer (&quot;Sim-emit-burst: X MB · NIC-ring anvendelig: Y MB ✓/✗&quot;).
* **Kontrol af ledningsbudget** — samlet efterspørgsel i stabil tilstand kontra det kollisionssikre loft for ledningen („Ledningsbudget: {efterspørgsel} MB/s efterspurgt af {n} kameraer · loft {loft} MB/s ✓/✗ overtegnet“).
* **&quot;Maks. antal kameraer på denne ledning: {n} — fastsat af båndbreddegrænsen pr. kamera, så binning ikke øger den.&quot;** — vises, når du nærmer dig (eller overskrider) loftet for antallet af kameraer.
* **&quot;BILLEDER VIL GÅ TABT ved disse indstillinger.&quot;**— rød advarsel med backendens begrundelse samt en liste over hindringer og blå**forslag til løsninger** (&quot;For at få plads til denne konfiguration på netværket&quot; / &quot;For at muliggøre samtidig optagelse&quot;).**Anvend &amp; Tilslut** er spærret, indtil der foreligger en prognose, og knapteksten fortæller dig, hvorfor den afviser:

| Knaptekst | Betydning | Hvad der rent faktisk hjælper |
| --- | --- | --- |
| &quot;Analyserer...&quot; | Analysen er stadig i gang. | Vent. |
| **&quot;For mange kameraer til dette netværk&quot;**| Arrayet overbelaster netværksforbindelsen (aggregatkontrol mislykkedes). | Færre kameraer, jumbo-rammer fra ende til ende eller et hurtigere netværkskort.**Et mindre ROI hjælper IKKE** — se nedenfor. |
| **&quot;Reducer ROI for at aktivere&quot;** | Rammer vil gå tabt ved disse indstillinger (burst-/ringkontrol mislykkedes). | Reducer ROI, øg binning eller reparer netkortets modtagelsesring. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Under tilslutningen kan der vises et grønt **kalibreringsdownload-panel** med en fremskridtsbjælke pr. serienummer: første gang et kamera tilsluttes en maskine, henter Chloros sin fabrikskalibreringspakke på ca. 3,8 MB fra kameraet via GigE (ca. 70 sekunder pr. kamera). Kameraer, der er gemt i cachen, viser aldrig dette panel. Se [Tilslutning af kameraer](connecting.md).

## Båndbredde: hvor mange kameraer der kan tilsluttes

Hvor mange kameraer et array kan håndtere, afhænger af kabeltypen, ikke af Chloros, så planlægningsværdierne findes i hardwarehåndbogen: **[Planlægning af array-båndbredde](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Hvad Chloros gør med dem: Tilslutningsdialogen kører en netværkssondering, beregner den opnåelige billedfrekvens og vælger et niveau, der passer. Hvis arrayet overbelaster kablet, nægter det at oprette forbindelse i stedet for at droppe pakker uden varsel — se panelet med det forventede resultat beskrevet ovenfor.

## Når rammer forsvinder

Et kamera kan mangle i en offentliggjort gruppe af to helt forskellige årsager,
og de kræver modsatrettede løsninger. Chloros tæller dem separat i stedet for at rapportere et
”ufuldstændigt” tal, der ikke angiver nogen af dem:

| Hvad skete der | Hvad betyder det | Hvor skal man lede |
| --- | --- | --- |
| **Korrupte**— billedrammen ankom, men var strukturelt defekt | GVSP-pakketab på netværksstien |**Kabelkapaciteten**, NIC-modtagelsesringen, jumbo-rammer, switchen |
| **Ankom aldrig**— der kom slet ingen ramme | Kameraet udløste ikke, eller der kom intet ud af det |**M8-synkroniseringskablet**, synkroniseringslinjen, om alle enheder er aktiveret |

Opdelingen revurderes hvert 10. sekund, mens arrayet streamer. Over 5 % bliver det
logget med begge tal angivet, og hver beskadiget buffer rapporteres første gang, det
sker pr. kamera, og derefter sammenfattet en gang i minuttet, så en lang session forbliver læsbar.

**Beskadigede rammer med nul „aldrig ankommet“ betyder, at udløsning og kabelsynkronisering er perfekte**og hvert tabt billede ligger på netværksstien. Løsningen er at sænke**Wire Budget** og
genoprette forbindelsen.

{% hint style="warning" %}
**At sænke udløserfrekvensen hjælper ikke på beskadigede billeder.** Kameraets pakkefrekvens
indstilles én gang ved opkobling. At sænke udløserfrekvensen ændrer, hvor ofte en burst
forekommer, ikke hvor hurtigt selve burstet sendes ud på kablet. På et målt system med 4 kameraer ændrede en
5× reduktion af udløserfrekvensen intet, mens en sænkning af wire budget fra 240 til
200 MB/s reducerede andelen af beskadigede frames på det samme setup fra 10,4 % til nul.
{% endhint %}

Et kørende array kan ikke omplanlægge sig selv — afbryd forbindelsen og opret den igen, så
forbindelsestidsvælgeren kan arbejde ud fra det nye budget.

### USB-netværksadaptere er begrænset til 200 MB/s

En USB-Ethernet-adapter angiver sin *Ethernet*-forbindelseshastighed, men hvad den rent faktisk
kan opretholde, er begrænset af USB-bussen og dens driver. En USB 10GbE-dongle blev tidligere angivet
med en gennemstrømning på ca. 1000 MB/s — et tal, som ingen nogensinde havde målt — og at styre
fire kameraer ud fra dette fiktive overskud forvrængede 6–18 % af billederne, mens arrayet
stadig rapporterede en sund målbilledhastighed. USB-tilsluttede adaptere er nu begrænset til
**200 MB/s**. Begrænsningen er et absolut tal snarere end en procentdel, fordi grænsen er
bussen: en USB 1 GbE-adapter leverer ca. 80 MB/s og påvirkes ikke.

Hvis din vært er reelt hurtigere end grænsen, skal du hæve **Wire Budget** for at angive dette.

## PTP-tidssynkronisering

*Synkronisering* af billeder sker via hardwaretriggeren; **PTP** (IEEE 1588 PTPv2) leverer sammenlignelige *tidsstempler* på tværs af alle enheder. Det er aktiveret som standard, når arrayet tilsluttes:

* **Chloros-værtsbackendet kører PTP-grandmasteren**. LATTICE-kameraer og DAQ-E-lyssensorer fungerer som slaver til denne i domæne 0, så billedtidsstempler og DAQ-spektre lander på ét ur (~1 ms).
* `--no-ptp` (CLI) deaktiverer det til laboratoriearbejde — tidsstempler på tværs af kameraer er da **ikke** sammenlignelige.
* Kontroller synkroniseringstilstanden med CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Fanen &quot;Kameraer&quot; har i sig selv ingen PTP-indikator; synkroniseringsoplysningerne pr. kamera, der vises der, er de skrivebeskyttede **Rolle**(Master/Slave),**Synkroniseringslinje** og arrayets kapacitetsniveau. DAQ-E PTP-tilstand vises i sensordetaljerne under fanen &quot;Lyssensorer&quot;.

## Live-visningen

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

af arrayet Hovedvisningsområdet tilbyder to layout (skift mellem dem i den øverste bjælke): **gittervisning**(hver flise er en celle; træk for at omarrangere, når gitterlåsen er låst op) og**listevisning**(arrayer i fuld bredde øverst, ét aktivt kamera nedenunder). Skyderen**Feed Zoom** ændrer størrelsen på felterne; ved en cellebredde under 200 px skjules navn/fps-overlejringerne automatisk.**Separat tilstand** viser ét felt pr. kamera. Hvert felt viser følgende overlejringer:

* kameranavnet (øverst til venstre),
* en **fps-angivelse** (øverst til højre) — dette er kameraets *faktiske optagelsesfrekvens*, som rapporteres af backend-systemet, ikke forhåndsvisningsfrekvensen (live-forhåndsvisning er begrænset til 30 fps uanset optagelsesfrekvens),
* en statusprik — grøn (streaming) / gul (indlæser) / rød (fejl),
* en **spinner for forældet billede**, når der ikke er modtaget et nyt billede i 2 sekunder — normalt i ca. 5 sekunder efter enhver til-/afbrydelse, mens backend-systemet omfordeler båndbredden mellem kameraerne.**Kombineret tilstand**viser en enkelt sammensat flise: backend&#x27;en udfører debayering, skalering, justering, støjfjernelse, konvertering til strålingsintensitet pr. bånd (plus DLS-reflektans, når en lyssensor er tilknyttet), evaluerer arrayets indeksudtryk, anvender LUT&#x27;en og streamer resultatet som MJPEG. Indtil det første justerede billede vises, angiver flisen sin tilstand: »Forbereder array…«, »Kalibrerer justering…«, »Venter på første billede…« eller – hvis budgettet for automatiske justeringsforsøg (~30 s) er opbrugt – »Justering påkrævet« med en**Kalibrer justering**-knap.

Nyttige fakta om kombineret tilstand:

* Kompositbilledet er registreret i forhold til **master**-kameraets billede. AE-ROI-fokusering og spotmåling på kompositbilledet er nøjagtige for masteren og omtrentlige for slaverne; brug**Split View** (arrayindstillinger → &quot;Vis medlemskameraer&quot;) for at få pixelpræcise fliser pr. kamera uden at åbne ekstra kameraforbindelser.
* **Display Layers**(array-indstillinger; standard: slået fra) giver dig mulighed for at vælge et forgrunds- og baggrundslag — ethvert medlemskamera eller**Index**. Når forgrund = Index, viser pixels uden for LUT Min/Max baggrundslaget.
* **Renderopløsning** (standard 720p) indstiller live-streamens højde *og* den gemte eksportstørrelse for det sammensatte billede. Billeder pr. kamera eksporteres altid i fuld opløsning.
* Justeringen beregnes pr. session og gemmes aldrig — se afsnittet om justering i panelet med arrayindstillinger for RMS-residualer og knappen ”Rekalibrer”.

## Optagelse: overvågning vs. analyse

Arrayets optagelsesflader er tydeligt opdelt i **overvågningskvalitet**(optag det, du ser) og**analysekvalitet** (optag rådata, kalibrer senere):

| Arbejdsgang | Kvalitet | Hvad der gemmes | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Optagelse**(stillbilleder) | Analyse | Én synkroniseret billedgruppe pr. gennemløb; filer pr. kamera på hvert valgt eksportniveau (rå/debayered/stråling/refleksion/forhåndsvisning/indeks) + `.daq` sidecar | Knappen**Optag alt** + Optagelsesindstillinger | `lattice array-capture` |
| **Optag indeksvideo** | Overvågning | Den kombinerede live-indekskomposit, som den vises — 8-bit, forhåndsvisningsopløsning, indbygget LUT; kræver, at livestreamen er åben | ● Optag indeksvideo (kombinerede arrays) | `lattice array-record` |
| **Rå burst → opbyg video**| Analyse | Rå sensorrammer ved fuld optagelseshastighed + manifest + `.daq`, derefter offline-rekonstruktion til kalibreret radiance-/reflektans-/indeksvideo, tidsmatchet til DAQ-målinger | ⦿ Optag rå burst →**Opret video** | `lattice array-burst` → `lattice array-build-video` |

Tommelfingerregel: hvis pixels skal levere *målinger*, skal du bruge optagelse eller burst (analyse-kvalitet); hvis du blot skal *se eller demonstrere*, hvad arrayet har registreret, skal du optage indeksvideoen (overvågnings-kvalitet).

### Optagelsesindstillinger (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

Tandet hjulet ved siden af **Capture All** åbner vinduet med optagelsesindstillinger (kræver et åbent projekt — optagelser gemmes i dette):

* **Optagelsestilstand**:**Single**(én gennemgang) /**Continuous**(i træk; begrænset af et optagelsestal, standard 1, eller en varighed, standard 10 s) /**Interval** (timelapse: N optagelser hvert X. interval for i alt Y, standard 1 hver 5. s i 1 minut).
* **Eksporttyper pr. kamera**: Raw, Debayered, Radiance, Reflectance, Preview, Index — alt, hvad der er relevant, er aktiveret som standard. Radiance/Reflectance er skjult for RGB-filterkameraer;**Reflectance vises kun, når kameraet har en DAQ-lyssensor** (sin egen eller arvet fra arrayet); Index kræver et konfigureret indeksudtryk.
* **Justeret**(pr. array, standard**aktiveret**): tilpasser medlemseksport til arrayets justeringsprofil, så eksporten er pixelregistreret. Raw forbliver altid uændret, men indeholder transformationen i metadataene.
* **Hurtigste optagelse** (vippeknap): kun rådata + den tildelte DAQ-aflæsning + den gratis sammensatte indekskomposit, hvor kalibreringsberegningerne springes over ved optagelsen for at opnå maksimal hastighed — genopbyg stråling/refleksion/indeks senere ud fra den gemte `.daq`.
* Valg bevares med projektet. Skjulte eller pausede kameraer springes over.

Den tilsvarende CLI (samme backend-endepunkt, samme semantik):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

TIFF-komprimering til optagelser er `deflate` (tabsfri, standard) eller `none` — fulde flagtabeller, layoutet for optagelsesmappen og reglerne for genbehandling findes i [CLI-referencen](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Parring af en DAQ-lyssensor

For at få forhåndsvisninger, der er korrigeret for refleksion og belysning, kræves der data om nedadrettet lys fra en DAQ-sensor (tilsluttet under fanen **Lyssensorer**):

* I **array-rækken**i sidepanelet vises en**&quot;DAQ · til/fra&quot;-knap** — *til* når der er indstillet en lyssensor på array-niveau **eller** når et hvilket som helst medlemskamera har sin egen; værktøjstipet angiver præcist, hvilken sensor der forsyner hvilket kamera.
* Tildel på array-niveau i array-indstillinger → **Omgivende lyssensor**→ rullemenuen**Lyssensor**. Valget bevares i projektet, overføres til alle medlemskameraer, og de enkelte kameraer kan stadig tilsidesætte det med deres egen sensor.
* Statuslinjen nedenfor viser den aktuelle tilstand: **Fra**→ &quot;Venter på første spektrum…&quot; →**&quot;Aktiv — alle kameraer i arrayet er belysningskorrigerede&quot;** → eller, hvis der ikke er modtaget et nyt spektrum inden for de sidste 3 sekunder, en meddelelse om forældet data — den sidste måling bruges fortsat (målinger udløber aldrig på optagelsesstien).

Når der er tildelt en sensor: eksporttypen »Reflectance« bliver tilgængelig, live-forhåndsvisninger er belysningskorrigerede, den prædiktive autoeksponering kan bruge spektret, og hver reflektansoptagelse skriver den DAQ-måling, der faktisk blev brugt, som en **`.daq`-sidecar** ved siden af billedet, så optagelsen kan genbehandles senere.

## `array-connect` CLI-indstillinger

| Flag | Standard | Beskrivelse |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatisk opdagelse af alle LATTICE-kameraer (kræver ≥2) | **Det første serienummer er MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synkroniseringslinje. |
| `--target-fps F` | auto | Master-udløserens udløsningshastighed. |
| `--binning {1,2,4}` | auto | Hardware-binning. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Ekspert-tilsidesættelse af valg af synkroniseringsniveau. |
| `--wire-ceiling-mbps MB_PER_S` | automatisk detekteret | Værtsledningsbudget i MB/s — den form, som feltet **Wire Budget** har i CLI. Sænk det, hvis arrayet rapporterer om korrupte rammer. Gemmes sammen med projektet, så en senere genopkobling gendanner det. |
| `--no-recommend` | fra | Spring netværksanalysetrinnet over. |
| `--no-ptp` | fra | Deaktiver PTP (tidsstempler på tværs af kameraer kan da ikke sammenlignes). |

`lattice array-list`, `array-status` og `array-disconnect` styrer den vedvarende session. Den fulde reference til underkommandoer, herunder justering (`align-calibrate` / `align-apply`) og netværksværktøjerne, findes i [CLI Reference § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); SDK-ækvivalenterne (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) findes i [SDK-referencen](../reference/sdk-reference.md). Fra Python er ledningsbudgettet `connect_array(..., wire_ceiling_mbps=120)`, og opdelingen mellem korrupt og aldrig ankommet findes i [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).

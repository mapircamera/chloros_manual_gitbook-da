# Multikamera-opstillinger

En LATTICE **opstilling**består af to eller flere LATTICE-kameraer, der er forbundet som én synkroniseret enhed. Ét kamera fungerer som**master**: Det udsender en hardware-GPIO-triggerimpuls på en fælles synkroniseringslinje (standard**Line2**), så alle enheder eksponerer i samme øjeblik. Chloros

tilføjer PTP-tidssynkronisering, en live-forhåndsvisning (kamera-specifikke fliser eller et enkelt justeret multibånds-kompositbillede) samt synkroniseret optagelse — hver optagelsesrunde producerer én **billedgruppe**, hvor alle kameraer deler det samme tidsstempel og billed-ID (angivet som `fid:N` i optagelsesoutputtet).

Arrays er den måde, hvorpå monokameraer (M3M) genererer vegetationsindekser — ét kamera bidrager med ét bånd, og arrayet justerer dem til en multibåndsstak. Se [Monokameraer og vegetationsindekser](mono-indices.md).

Der er tre ækvivalente måder at tilslutte en array på, og alle kører den samme »smart-prep«-proces:

| Overflade | Indgangspunkt |
| --- | --- |
| GUI | Fanen »Kameraer« → **Tilslut array** (blå knap) |
| CLI

| `chloros-cli lattice array-connect --serials SN1,SN2,…` (første serienummer = master) |
| Python

SDK

| `connect_array(serials=[…])` → `ArraySession` (første serienummer = master) |

Smart-prep udfører: en test af netværkskapaciteten (ICMP DF-ping + GVSP-test), valg af synkroniseringsniveau, en fremskrivning af billedfrekvensen ved den anmodede billedstørrelse, beslutningen om pakket 12-bit-transport (se [Pakket 12-bit transport](#packed-12-bit-transport)), aktivering af PTP, automatisk valg af pixelformat pr. kamera, indstilling af automatisk eksponering ud fra hvert kameras gemte tilstand samt konfiguration af GPIO-trigger på Line2.

**Fuld sensor som standard (nyt i 1.3.0).** Connect bevarer den billedstørrelse, du har anmodet om — den fulde 2048×1536-sensor, ingen binning og ingen beskæring, medmindre du vælger andet — og lader billedfrekvensen falde til det, som værten og netværket leverer. Den foretager aldrig binning eller beskæring af sig selv (i version 1.2.0 blev binning automatisk aktiveret ved under 1,5 fps). Det er op til dig, om der skal foretages binning eller bruges et mindre ROI: Vælg en af delene i Connect-dialogboksen, og det forventede resultat viser den billedhastighed, det giver; `array-connect` udskriver de hurtigere muligheder med deres forventede hastigheder.

{% hint style="info" %}
Kameraerne skal være tilgængelige på forbindelsen, før noget af dette fungerer — se [Tilslutning af kameraer](connecting.md) for opdagelse, adressering og download af kalibrering ved første tilslutning. For rigge med flere kameraer er værts-NIC&#x27;ens indstillinger for modtagelsesring lige så vigtige som forbindelseshastigheden; den komplette tabel over symptomer og løsninger findes i [CLI

-reference § Opsætning og finjustering af værts-NIC](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dialogboksen »Array Connect«

Fanen »Kameraer« → **Connect Array**åbner en tretrinsguide:**Vælg → Visningstilstand → Indstillinger**.

### Trin 1 — Vælg master og slaver

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



Dialogboksen scanner netværket, så snart den åbnes (&quot;Scanner netværk...&quot;), og tester derefter synkroniseringskablerne (&quot;Tester synkroniseringskabler...&quot;) for at finde ud af, hvilke kameraer der er tilsluttet M8-synkroniseringslinjen. Du skal bruge mindst **2 kameraer** for at oprette et array.

M8-synkroniseringskablerne mødes i et knudepunkt, så **ethvert tilsluttet kamera kan fungere som master**. Testen udfylder rollerne på forhånd — RGB

-kameraet (`-FRGB`) som master, når det er tilsluttet, ellers det første tilsluttede kamera, og de øvrige tilsluttede kameraer som slaver — og viser et af tre bannere:

| Banner | Betydning |
| --- | --- |
| »Synkroniseringskabel fundet på {n} kameraer. Ethvert af dem kan være master; RGB

-kameraet (ellers det første) er forudvalgt.« (grøn) | Proben har fundet kameraerne på synkroniseringslinjen; afkrydsningsfelterne for master-radio og slave er allerede udfyldt. |
| &quot;Ingen master fundet – tjek GPIO-kablet&quot; (orange) | Intet kamera har registreret en triggerimpuls; tjek synkroniseringskablerne. Rollerne er stadig forudfyldt (RGB

-kameraet, ellers det første kamera på listen, som master), og du kan ændre dem. |
| &quot;Intet synkroniseringskabel: {serienumre}&quot; (orange) | De anførte kameraer har ikke noget synkroniseringskabel tilsluttet. |

Kameratabellen har kolonnerne **Kamera / Serienummer / IP / Master (radio) / Slave (afkrydsningsfelt)**. Rækkerne er opført i**kamerarækkefølge** — M3C (Bayer)-kameraer før M3M (mono); blandt M3C, FRGB, FRGN, FOCN og derefter FNGB; M3M efter filterbølgelængde, den laveste først (F450 før F808). Kameralisten i sidepanelet og gitterets standardlayout bruger den samme rækkefølge.

* Vælg præcis **én master**og**én eller flere slaver**. Hvis du klikker på den aktuelle masters radioknap igen, fjernes markeringen. Det kræver ét klik at vælge en anden master: den nye master forlader slaverne, og den tidligere master bliver en slave (medmindre den ikke har noget synkroniseringskabel).
* Et kamera markeret med **&quot;Intet synkroniseringskabel&quot;** kan aldrig vælges som slave — en slave uden udløserledninger vil vente på synkroniseringslinjen i al evighed og levere et dødt signal. Tilslut i stedet det pågældende kamera som et selvstændigt kamera.
* Kameraer, der allerede er tilsluttet som enkeltstående, deaktiveres *ikke*: array-tilslutning afbryder den enkeltstående session og genåbner kameraet inden for arrayet.

**Næste: Visningstilstand →**aktiveres, når der er valgt en master og mindst én slave.**Genscan** kører genfindings- og ledningssøgningen igen.

{% hint style="warning" %}
**Annuller** er deaktiveret, mens en scanning eller ledningskontrol er i gang — annullering midt i ledningskontrollen kan få kamera-SDK

&#x27;en til at gå ned på LATTICE-kameraets firmware. Vent, indtil spinneren er færdig.
{% endhint %}

### Trin 2 — Visningstilstand

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->



| Tilstand | Hvad du får |
| --- | --- |
| **Separate kameraer** | Én live-flise pr. kamera, som alle udløses samtidigt, så billederne forbliver synkroniserede. Hvert kamera beholder sin egen farve og sine egne indstillinger. |
| **Kombinerede kameraer** *(standard)* | En enkelt flise, der gengiver den justerede multibånds-NDVI

-/indeks-sammensætning. Kameraerne deler arrayets farve. |

Visningstilstanden ændrer kun præsentationen af live-forhåndsvisningen — optagelsesadfærden er den samme i begge tilfælde.

### Trin 3 — Array-indstillinger og det forventede resultat

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->



Når du åbner denne scene, beder Chloros

backend-systemet om en **anbefaling**og anvender den:**fuld sensor — ingen binning, ingen ROI-beskæring — når som helst billedfrekvensen kan klare det**, uanset hvilken hastighed værten og kabelforbindelsen leverer. Først når ingen hastighed kan levere den fulde sensor, formindsker anbefalingen billedet, og i så fald foretrækker den binning frem for ROI-beskæring, fordi binning bevarer det fulde synsfelt. Herfra bytter du opløsning mod billedhastighed. Hver ændring, du foretager, kører analysen live igen og opdaterer panelet**Forventet resultat** til højre.

Venstre kolonne — indstillinger:

| Kontrol | Valgmuligheder | Standard | Bemærkninger |
| --- | --- | --- | --- |
| **ROI (synsfelt)** | Fuld (2048×1536) / Halv (1024×768) / Kvart (512×384) | Fuld | Sensorbeskæring: Halv/kvart beskæring til et mindre område med oprindelig pixelafstand. |
| **Binning** | 1× / 2× (sum 2×2) / 4× (sum 4×4) | 1× | Hardware-binning: 2×2 = fuldt synsfelt til en fjerdedel af ledningsomkostningerne; 4×4 = fuldt synsfelt til 1/16. Skjult, hvis kameraerne ikke understøtter binning. |
| **Billeddata på ledningen** (udlæsning) | — | — | Den faktiske bredde × højde efter binning, der sendes via ledningen, afrundet til multipler af 16 (minimum 64). |
| **Triggerfrekvens** | 0,5–60 fps, trin 0,1 | tom = auto | Masterens triggerfrekvens. Lad feltet stå tomt, så Chloros

kan udlede den. |
| **Kabelkapacitet**| 20–2000 MB/s, trin 10 | tom = auto | Hvor meget værten faktisk kan håndtere, i MB/s —**det eneste tal, som hele array-allokeringen afhænger af.** Registreres automatisk fra netværksadapteren. Sænk det, hvis arrayet rapporterer om beskadigede rammer: den registrerede værdi overvurderer USB-adaptere og delte switche. Hvis du ændrer det, kører prognosen live igen. Feltet åbnes med det budget, som det sidste array var forbundet med (gemt sammen med projektet), så et budget, du har korrigeret én gang, bevares. |

Afkrydsningsfeltet **Pin-opløsning** fra version 1.2.0 er fjernet: Tilslutning bevarer altid den valgte rammestørrelse og sænker i stedet billedfrekvensen.

Højre kolonne — **Projiceret resultat**:

* **Synkroniseringsniveau** — &quot;Samtidig optagelse&quot; (grøn), &quot;Samtidig optagelse (FTD-forskudt udsendelse)&quot; (grøn), &quot;Forskudt optagelse (100 ms afvigelse)&quot; (gul) eller &quot;Konfigurationen er for stor&quot; (rød). Chloros

vælger niveauet: samtidig optagelse, når ledningen kan deles med en hastighed, som kameraerne kan tilpasses til (billedfrekvensen falder, efterhånden som hvert kameras andel mindskes), og forskudt optagelse kun, når det ikke er muligt — så et array kan oprette forbindelse på ethvert netværk med den billedfrekvens, som netværket tillader.
* **fps-prognose** — den billedhastighed, som arrayet forventes at køre med. Ved samtidig optagelse er det ét tal: hastigheden kommer først, og autoeksponeringen udfylder den periode, den giver, og hæver derefter forstærkningen, så en mørk scene ikke sænker den (se [Hvordan autoeksponering fordeler eksponering og forstærkning](camera-settings.md)). Ved forskudt optagelse vises det som et interval (&quot;mørkt → lyst&quot;), fordi hvert kameras eksponering forlænger forskydningen i en mørk scene.
* **NIC-linje** — forbindelseshastighed og vedvarende gennemstrømning (&quot;NIC {mbps} Mbps · vedvarende {N} MB/s&quot;).
* **Sim-emit-burst-kontrol** — kan værtsens NIC-modtagerring absorbere én samtidig burst fra alle kameraer (&quot;Sim-emit-burst: X MB · NIC-ring tilgængelig: Y MB ✓/✗&quot;).
* **Kontrol af ledningsbudget** — samlet efterspørgsel i stabil tilstand kontra det kollisionssikre loft for ledningen (&quot;Ledningsbudget: {efterspørgsel} MB/s efterspurgt af {n} kameraer · loft {loft} MB/s ✓/✗ overtegnet&quot;).
* **&quot;Maks. antal kameraer på denne ledning: {n} — fastsat af båndbreddegrænsen pr. kamera, så binning ikke øger den.&quot;** — vises, når du er tæt på (eller over) loftet for antallet af kameraer.
* **&quot;BILLEDER VIL GÅ TABT ved disse indstillinger.&quot;**— rød advarsel med backendens begrundelse samt en liste over hindringer og blå**løsningsforslag** (&quot;For at få plads til denne konfiguration på netværket&quot; / &quot;For at muliggøre samtidig optagelse&quot;).**Anvend &amp; Opret forbindelse** er spærret, indtil der foreligger en prognose, og knapteksten angiver, hvorfor den afvises:

| Knaptekst | Betydning | Hvad der rent faktisk hjælper |
| --- | --- | --- |
| &quot;Analyserer...&quot; | Analysen er stadig i gang. | Vent. |
| **&quot;For mange kameraer til dette netværk&quot;**| Forbindelsen kan ikke håndtere arrayet, selv ikke ét kamera ad gangen (samlet kontrol mislykkedes): langt flere kameraer, end den kan håndtere, eller andre kameraer, der streamer på den. | Færre kameraer, jumbo-rammer fra ende til ende eller et hurtigere netværkskort.**Et mindre ROI vil IKKE hjælpe** — se nedenfor. |
| **&quot;Reducer ROI for at aktivere&quot;** | Rammer vil gå tabt ved disse indstillinger (burst/ring-kontrol mislykkedes). | Reducer ROI, øg binning eller reparer NIC-modtagelsesringen. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



Under tilslutningen kan der vises et grønt **kalibreringsdownload-panel** med en fremskridtsbjælke pr. serienummer: første gang et kamera tilsluttes en maskine, henter Chloros

dets fabrikskalibreringspakke på ca. 3,8 MB fra kameraet via GigE (ca. 70 sekunder pr. kamera). Kameraer, der er gemt i cachen, viser aldrig dette panel; ved mere end seks kameraer ruller rækkerne. Se [Tilslutning af kameraer](connecting.md).

### Pakket 12-bit-transport

12-bit-formater (`BayerRG12`, `Mono12`) overføres normalt som 16-bit-ord. **Pakket**transport sender de samme 12 bit i 1,5 byte pr. pixel og udpakker dem på værten, så hver ramme tager 25 % kortere tid at sende — ved fuld sensor betyder det cirka en tredjedel flere rammer pr. sekund.**De gemte data er identiske.** Array Connect bestemmer det automatisk (`auto`): den pakker kun, når det er en fordel, og værten har CPU-kapacitet til rådighed til udpakningen. GUI’en har ingen indstilling til dette; fra CLI

’en skal du bruge `array-connect --packed {auto,on,off}` (standard `auto`). 8-bit- og 16-bit-formater pakkes aldrig.

## Båndbredde: hvor mange kameraer der kan være

Hvad et array kan håndtere, afhænger af netværksforbindelsen, ikke af Chloros

, så planlægnings tallene findes i hardwarehåndbogen: **[Planlægning af array-båndbredde](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Hvad Chloros

gør med dem: forbindelsesdialogen kører en netværkssondering, beregner den opnåelige billedhastighed og vælger et niveau, der passer. Et array kan oprette forbindelse på ethvert netværk: på en langsom forbindelse tilpasses hvert kamera til sin andel af båndbredden, og billedhastigheden falder, i stedet for at arrayet afvises eller dets billeder komprimeres. Kun et array, hvor forbindelsen ikke kan håndtere så meget som ét kamera ad gangen, afvises (**For mange kameraer til dette netværk**) i stedet for at droppe pakker uden varsel — se panelet med det forventede resultat beskrevet ovenfor.

## Når billeder forsvinder

Et kamera kan mangle i en offentliggjort gruppe af to helt forskellige årsager,
og de kræver modsatrettede løsninger. Chloros

tæller dem separat i stedet for at rapportere ét
»ufuldstændigt« tal, der ikke angiver nogen af dem:

| Hvad skete der | Hvad betyder det | Hvor skal man lede |
| --- | --- | --- |
| **Korrupt**— billedrammen ankom, men var strukturelt defekt | GVSP-pakketab på netværksstien |**Kabelkapaciteten**, NIC-modtagelsesringen, jumbo-rammer, switchen |
| **Ankom aldrig**— der kom slet ingen ramme | Kameraet udløste ikke, eller der kom intet ud af det |**M8-synkroniseringskablet**, synkroniseringslinjen, om alle medlemmer er aktiveret |

Fordelingen revurderes hvert 10. sekund, mens arrayet streamer. Over 5 % logges det
med begge tal angivet, og hver beskadiget buffer rapporteres første gang, det
sker pr. kamera, og samles derefter en gang i minuttet, så en lang session forbliver læsbar.

**Korrupte billeder med nul »aldrig ankommet« betyder, at udløsning og kabelsynkronisering fungerer perfekt**og at alle mistede billeder skyldes netværksstien. Løsningen er at sænke**Wire Budget** og
genoprette forbindelsen.

{% hint style="warning" %}
**At sænke udløserfrekvensen hjælper ikke på beskadigede billeder.** Kameraets pakke-
timing indstilles én gang ved opkobling. At sænke udløserfrekvensen ændrer, hvor ofte en burst
forekommer, ikke hvor hurtigt selve burstet sendes ud på kablet. På et målt setup med 4 kameraer ændrede en
5× reduktion af triggerfrekvensen intet, mens en sænkning af wire budget fra 240 til
200 MB/s reducerede andelen af beskadigede frames på samme opsætning fra 10,4 % til nul.
{% endhint %}

Et kørende array kan ikke omplanlægge sig selv — afbryd forbindelsen og opret den igen, så
forbindelsestidsvælgeren kan arbejde ud fra det nye budget.

### USB-netværksadaptere er begrænset

En USB-Ethernet-adapter angiver sin *Ethernet*-forbindelseshastighed, men hvad den rent faktisk
kan opretholde, er begrænset af USB-bussen og dens driver. En USB 10GbE-dongle blev tidligere tilskrevet
en gennemstrømning på ca. 1000 MB/s — et tal, som ingen nogensinde havde målt — og at køre
fire kameraer op mod dette fiktive overskud forårsagede fejl i 6–18 % af billederne, mens arrayet
stadig rapporterede en sund målbilledhastighed. USB-tilsluttede adaptere er nu begrænset til
**200 MB/s** på Windows

og NVIDIA Jetson samt **400 MB/s** på Linux

x86_64. Begrænsningen er et absolut tal snarere end en procentdel, fordi grænsen er
bussen: en USB 1 GbE-adapter opnår ca. 80 MB/s og påvirkes ikke.

Hvis din vært er reelt hurtigere end grænsen, skal du hæve **Wire Budget** for at angive dette.

## PTP-tidssynkronisering

*Synkronisering* af billeder sker via hardwaretriggeren; **PTP** (IEEE 1588 PTPv2) leverer sammenlignelige *tidsstempler* på tværs af alle enheder. Det er aktiveret som standard ved tilslutning til arrayet:

* **Chloros

-værtsbackendet kører PTP-grandmasteren**. LATTICE-kameraer og DAQ-E-lyssensorer fungerer som slaver til den i domæne 0, så billedtidsstempler og DAQ-spektre lander på ét ur (~1 ms).
* `--no-ptp` (CLI

) deaktiverer det til laboratoriearbejde — tidsstempler på tværs af kameraer er da **ikke** sammenlignelige.
* Kontroller synkroniseringstilstanden via CLI

:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Fanen »Kameraer« har i sig selv ingen PTP-indikator; synkroniseringsoplysningerne pr. kamera, der vises der, er de skrivebeskyttede **Rolle**(Master/Slave),**Synkroniseringslinje** og arrayets kapacitetsniveau. DAQ-E PTP-tilstand vises i sensordetaljerne på fanen »Lyssensorer«.

## Live-array-visningen

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

Hovedfeedområdet tilbyder to layouts (skift mellem dem i den øverste bjælke): **gittervisning**(hver flise er en celle; træk for at omarrangere, når gitterlåsen er låst op) og**listevisning**(rækker i fuld bredde: et**kombineret**array udgør én række — dets sammensatte flise — mens et**separat**array giver**hvert medlem sin egen række**med sit eget livefeed og sin egen kalibreringsmåltabel; under arrayene vises et enkeltstående kamera — det aktive). Skyderen**Feed Zoom** ændrer flisernes størrelse; ved en cellebredde under 200 px skjules navn/fps-overlays automatisk. Medlemskameraerne vises i sidepanelet og i gitterets standardlayout i kamerarækkefølge (M3C før M3M, FRGB først, monokromkameraer efter bølgelængde); hvis man låser gitteret op og trækker, tilsidesættes gitterrækkefølgen stadig.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



**Separat tilstand** viser én flise pr. kamera. Hver flise viser:

* kameranavnet (øverst til venstre),
* en **fps-visning** (øverst til højre) — dette er kameraets *faktiske optagelsesfrekvens*, som rapporteres af backend-systemet, ikke forhåndsvisningsfrekvensen (live-forhåndsvisning er begrænset til 30 fps uanset optagelsesfrekvens),
* en statusprik — grøn (streaming) / gul (indlæser) / rød (fejl),
* en **spinner for forældet billede**, når der ikke er ankommet et nyt billede i 2 sekunder — normalt i ca. 5 sekunder efter enhver tilslutning/afbrydelse, mens backend&#x27;et omfordeler båndbredden mellem kameraerne.**Kombineret tilstand**viser en enkelt sammensat flise: backend&#x27;en udfører debayering, skalering, justering, støjfjernelse, konverterer til strålingsintensitet pr. bånd (plus DLS-reflektans, når en lyssensor er tilknyttet), evaluerer arrayets indeksudtryk, anvender LUT&#x27;en og streamer resultatet som MJPEG. Indtil det første justerede billede vises, angiver feltet sin status: »Forbereder array…«, &quot;Kalibrerer justering…&quot;, &quot;Venter på første billede…&quot;, eller — hvis flisens automatiske justeringsforsøg (ca. et minut) løber ud — &quot;Justering påkrævet&quot; med en**Kalibrer justering**-knap.

Array-flisens fps-visning angiver også streamens tilstand:**»Anvender indstillinger…«**, mens en ændring af indstillingerne (format, binning, ROI) genstarter kameraerne,**»Stoppet«**, når arrayet er aktivt, men er holdt op med at levere billedgrupper, og**»Stoppet&quot;**, når den er holdt op med at streame — i så fald skal arrayet tilsluttes igen. Når den spektrale adskillelse har været nødt til at begrænse et bånd, der vises i sammensætningen, på en stor andel af dets pixels (25 % eller mere), viser flisen**&quot;Bånddata med lav pålidelighed · N %&quot;**: indeksværdier baseret på dette bånd er tvivlsomme (normalt har et bånd modtaget for lidt lys).

Nyttige oplysninger om kombineret tilstand:

* Kompositbilledet er registreret i forhold til **master**-kameraets billede. AE-ROI-fokusering og spotmåling på kompositbilledet er nøjagtige for masteren og omtrentlige for slaverne; brug**Split View** (arrayindstillinger → &quot;Vis medlemskameraer&quot;) for at få pixelpræcise fliser pr. kamera uden at åbne ekstra kameraforbindelser.
* **Display Layers**(array-indstillinger; som standard slået fra) giver dig mulighed for at vælge et forgrunds- og baggrundslag — ethvert medlemskamera eller**Index**. Når forgrund = Index, viser pixels uden for LUT Min/Max baggrundslaget.
* **Renderopløsning** (standard 720p) indstiller live-streamens højde *og* størrelsen på den gemte sammensatte eksportfil. Billeder pr. kamera eksporteres altid i fuld opløsning.
* Justering beregnes pr. session og gemmes aldrig. Chloros

justerer **hvert array automatisk efter tilslutning, i begge visningstilstande**, så snart medlemmernes automatiske eksponering har stabiliseret sig (programmet forsøger i op til 5 minutter). Afsnittet**Justering**i panelet med arrayindstillinger viser RMS-afvigelserne og knappen**Justering** — også for separate arrays, der ikke har en sammensat flise.

## Optagelse: overvågning vs. analyse

Array-optagelsesfladerne kan tydeligt opdeles i **overvågningskvalitet**(optag det, du ser) og**analysekvalitet** (optag rådata, kalibrer senere):

| Arbejdsgang | Kvalitet | Hvad der gemmes | GUI | CLI

|
| --- | --- | --- | --- | --- |
| **Optagelse**(stillbilleder) | Analyse | Én synkroniseret billedgruppe pr. gennemløb; filer pr. kamera på hvert valgt eksportniveau (rå/debayered/radiance/reflektans/forhåndsvisning/indeks) + `.daq` sidecar |****Capture All**-knap + Optagelsesindstillinger | `lattice array-capture` |
| **Optag indeksvideo** | Overvågning | Den kombinerede live-indekskomposit, som den vises — 8-bit, forhåndsvisningsopløsning, indbygget LUT. Fra CLI

&#x27;en uden åben GUI-visning renderer backend-systemet arrayets NDVI

(eller `--expr`), så snart arrayet er justeret; en optagelse, der ikke skriver nogen ramme, mislykkes | ● Optag indeksvideo (kombinerede arrays) | `lattice array-record` |
| **Rå burst → opbyg video**| Analyse | Rå sensor-frames ved fuld indlæsningshastighed + manifest + `.daq`, derefter offline-rekonstruktion til kalibreret radiance-/reflektans-/indeksvideo, tidsmatchet til DAQ-målinger | ⦿ Optag rå burst →**Opbyg video** | `lattice array-burst` → `lattice array-build-video` |

Tommelfingerregel: hvis pixels skal levere *målinger*, skal du bruge optagelse eller burst (analyse-kvalitet); hvis du blot skal *se eller demonstrere*, hvad arrayet har registreret, skal du optage indeksvideoen (overvågnings-kvalitet).

### Indstillinger for optagelse (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



Tandhjulet ved siden af **Capture All** åbner vinduet med indstillinger for optagelse (kræver et åbent projekt — optagelser gemmes i dette):

* **Optagelsestilstand**:**Single**(én gennemgang) /**Continuous**(kontinuerligt; afgrænset af et antal optagelser, standard 1, eller en varighed, standard 10 s) /**Interval** (timelapse: N optagelser hvert X. interval i alt Y, standardindstilling 1 hver 5. s i 1 minut).
* **Eksporttyper pr. kamera**: Raw, Debayered, Radiance, Reflectance, Preview, Index — alt, hvad der er relevant, er aktiveret som standard. Radiance/Reflectance er skjult for kameraer med RGB

-filter; **Reflectance vises kun, når kameraet har en DAQ-lyssensor** (sin egen eller arvet fra arrayet); Index kræver et konfigureret indeksudtryk.
* **Justeret**(pr. array, standard**aktiveret**): tilpasser eksportfilerne til arrayets justeringsprofil, så eksportfilerne er pixelregistrerede. Raw forbliver altid uforvrænget, men bærer transformationen i metadataene. Array-medlemsfiler gemmes i hvert kameras oprindelige orientering (eventuelle indstillinger for spejling/rotation registreres i filen, men anvendes ikke).
* **Hurtigst mulig optagelse** (slå til/fra): kun raw + den tildelte DAQ-måling + den gratis sammensatte indekskomposit, hvor kalibreringsberegningerne springes over ved optagelsen for at opnå maksimal hastighed — genopbyg radiance/reflektans/indeks senere ud fra den gemte `.daq`.
* Valg bevares med projektet. Skjulte eller pausede kameraer springes over.
* **Kontinuerlig** på arrays kører i Chloros

-backend (nyt i 1.3.0): når hvert kamera på listen tilhører et array, kører backend selv optagelsesløkken og skriver én gruppe, mens den vælger den næste, uden anmodning pr. optagelse. Et fuldt drev stopper kørslen (&quot;Stoppet: drev fuldt. Gemt N, F mislykkedes&quot;). Se [Optagelsesindstillinger og -tilstande](capture.md#capture-modes) for bemærkningen fra Microsoft Defender om høje optagelseshastigheder.

Den tilsvarende CLI

-kommando (samme backend-endpoint, samme semantik):

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

TIFF

-komprimering til optagelser er `deflate` (tabsfri, standard) eller `none` — komplette flagtabeller, strukturen i optagelsesmappen og reglerne for genbehandling findes i [CLI

-reference](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Parring af en DAQ-lyssensor

For at få forhåndsvisninger, der er korrigeret for refleksion og belysning, kræves der data om nedadrettet lys fra en DAQ-sensor (tilsluttet under fanen **Lyssensorer**):

* **Array-rækken**i sidepanelet viser en**&quot;DAQ · til/fra&quot;-knap** — *til* når der er indstillet en lyssensor på array-niveau **eller** når et hvilket som helst medlemskamera har sin egen; dens værktøjstip angiver præcist, hvilken sensor der forsyner hvilket kamera.
* Tildel for hele arrayet i arrayindstillingerne → **Omgivende lyssensor**→ rullemenuen**Lyssensor**. Valget bevares i projektet, overføres til hvert enkelt kamera i arrayet, og de enkelte kameraer kan stadig tilsidesætte det med deres egen sensor.
* Statuslinjen nedenfor viser den aktuelle tilstand: **Fra**→ &quot;Venter på første spektrum…&quot; →**&quot;Aktiv — alle kameraer i arrayet er belysningskorrigerede&quot;** → eller, hvis der ikke er modtaget et nyt spektrum inden for de sidste 3 sekunder, en meddelelse om forældet data — den sidste måling bruges fortsat (målinger udløber aldrig på optagelsesstien).

Når der er tildelt en sensor: eksporttypen »Reflectance« bliver tilgængelig, live-forhåndsvisninger er belysningskorrigerede, den prædiktive autoeksponering kan bruge spektret, og hver reflektansoptagelse skriver den DAQ-måling, der faktisk blev brugt, som en **`.daq`-fil** i optagelsernes `daq/`-mappe, så optagelsen kan genbehandles senere. Ved siden af den angiver en `.daq.cameras.json`-fil navnene på de kameraer, hvis billeder den pågældende måling beskriver; ved genbehandling matcher Chloros

kun målingen med billeder fra disse kameraer.

## `array-connect`-CLI

-indstillinger

| Flag | Standard | Beskrivelse |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatisk opdagelse af alle LATTICE-kameraer (kræver ≥2) | **Det første serienummer er MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synkroniseringslinje. |
| `--target-fps F` | auto | Master-triggerens affyringshastighed. |
| `--binning {1,2,4}` | auto | Hardware-binning. Auto er den fulde sensor (ingen binning), når en billedhastighed understøtter det; connect binner aldrig, blot for at holde hastigheden oppe. |
| `--packed {auto,on,off}` | `auto` | [Pakket 12-bit transport](#packed-12-bit-transport): `auto` pakker kun, når det er nyttigt, og værten har kapacitet til at udpakke. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Ekspertoverstyring af valg af synkroniseringslag. |
| `--wire-ceiling-mbps MB_PER_S` | auto-detected | Værtsbåndbredde i MB/s — CLI

-formen af feltet **Wire Budget**. Sænk den, hvis arrayet rapporterer om beskadigede rammer. Gemmes sammen med projektet, så en senere genopkobling gendanner den. |
| `--no-recommend` | fra | Spring netværksanalysetrinnet over. |
| `--no-ptp` | fra | Deaktiver PTP (tidsstempler på tværs af kameraer kan da ikke sammenlignes). |

`lattice array-list`, `array-status` og `array-disconnect` styrer den vedvarende session. Den fulde oversigt over underkommandoer, herunder justering (`align-calibrate` / `align-apply`) og netværksværktøjerne, findes i [CLI

-reference § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); de tilsvarende SDK

-kommandoer (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) findes i [SDK

-referencen](../reference/sdk-reference.md). Fra Python

er ledningsbudgettet `connect_array(..., wire_ceiling_mbps=120)`, og opdelingen mellem live-korrupt og aldrig ankommet findes i [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).

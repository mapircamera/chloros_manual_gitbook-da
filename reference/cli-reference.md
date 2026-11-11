# Chloros-CLI-reference

**Version:**

1.3.0**Oprettet:**

29. juli 2026 kl. 19:19 ·**Revideret:**

6. november 2026**Målgruppe:** Optimeret til brug af LLM; læsbar for mennesker.**Omfang:** Alle brugerrettede underkommandoer til `chloros-cli`, med indstillinger og eksempler, der kan kopieres og indsættes.

Dette dokument er den komplette reference til kommandolinjeværktøjet `chloros-cli`, der følger med MAPIR Chloros. Det er bevidst udtømmende, så en LLM (eller et menneske) kan sammensætte enhver understøttet arbejdsgang ud fra nedenstående oversigter uden at skulle gennemgå kildekoden.

Hvis du kun har brug for det vigtigste, kan du springe til:
- [Hurtigstart på fem minutter](#five-minute-quickstart)
- [LATTICE-kamera – arbejdsgang ved første tilslutning](#lattice-camera-first-connect-workflow)
- [DAQ-sensor – arbejdsgang ved første tilslutning](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Optagelsestilstande, optagere og offline-genbehandling](#capture-modes-recorders--offline-reprocess)

> **Styrer du Chloros fra en AI-assistent?** Claude Desktop, Claude Code, Cursor og andre MCP-klienter kan køre Processing, LATTICE-kameraer og -arrays samt DAQ-lyssensorer via `chloros-mcp`, MCP-serveren, der installeres sammen med Python-SDK&#x27;et (og med Linux-.deb-filen, i `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. Opsætning, indstillinger og værktøjer findes i SDK-referencens [MCP-server (AI-assistenter)](sdk-reference.md#mcp-server-ai-assistants). En assistent med en shell kan også køre alle kommandoer på denne side direkte.

> **Ændret i 1.3.0** (hvad et eksisterende 1.2.0-script eller en eksisterende 1.2.0-workflow bemærker).
>
> **Ændringer, som et 1.2.0-script muligvis skal håndtere:**
>
> - **`--preset` gælder.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` og `align-stream` blev fjernet i version 1.2.0 og kørte på kameraets egne indstillinger. Den indstiller nu forudindstillingens format, eksponering, forstærkning og billedfrekvens: `--preset high_speed` optager 8-bit ved 5 ms, 6 dB, 60 fps. `lattice cam-connect --preset` bevarer også forudindstillingens format (i version 1.2.0 blev kameraet indstillet til 12-bit efter anvendelse af forudindstillingen). Se [Almindelige indstillinger](#common-options-shared-by-most-subcommands).
> - **Et Bayer-pixelformat på et M3M-kamera (mono) er dets dybde.** Forudindstillingerne navngiver Bayer-formater, som en monosensor ikke kan optage: i 1.2.0 afviste kameraet formatet og beholdt det, det havde, så `--preset high_speed` var ikke 8-bit der. Det er nu Mono-formatet med samme dybde: `high_speed` og `--pixel-format BayerRG8` giver `Mono8`, `high_quality`, `triggered` og `BayerRG12` giver `Mono12`. I en blandet M3C + M3M-konfiguration anvender hvert kamera det fælles format inden for sin egen familie, og `network-analysis` / `analyze-array --pixel-format` projicerer hvert kamera på den pågældende dybde. Det samme gælder omvendt: et M3C, der anmodes om et Mono-format, anvender Bayer-formatet for den pågældende dybde, så et array, hvis master er et M3M, genåbner ikke længere sine M3C-kameraer i Mono. Se [Mono (M3M) enkeltbåndskameraer](#mono-m3m-single-band-cameras).
> - **`lattice array-capture`-eksportfiler er som standard justeret, ligesom appens er.** Hvert medlems `debayered`, `display`, `radiance`, `reflectance` og indeksfiler tilpasses til masterkameraets ramme og beskæres til medlemmernes overlapning (mindre end sensoren med kameraernes forskydninger, `XMP-mapir:AlignmentJSON` `"applied": true`), så en CLI-optagelse består af de samme filer som Chloros-appens optagelse af den samme scene; `raw` forbliver i sin oprindelige størrelse. Tidligere gemte en optagelse uden `--aligned` alle filer i deres oprindelige størrelse, uden at blive tilpasset. `--no-aligned` bevarer denne funktion. Den samme standard gælder for `--smart`, `--continuous` og `--interval` samt til SDK og `chloros-mcp`. Se [`array-capture`-tilstande](#array-capture-modes).
> - **`lattice array-capture --compression` er som standard indstillet til `none`.** Filerne skrives ukomprimeret (ca. 6,3 MB pr. billede i fuld opløsning) for at opnå den hurtigste vedvarende hastighed; version 1.2.0 skrev `deflate` (ca. 4,1 MB). Angiv `--compression deflate` for at få mindre filer. Begge er tabsløse.
> - **PNG og JPEG accepterer kun `display` / `preview` og indeksprodukter.** En `--format png`- eller `jpg`-optagelse med et niveau, som en 8-bit-fil ikke kan rumme, afvises, før der optages noget (HTTP-kode 400, afslutning af `1`), og det samme gælder standardindstillingen `--processing all` med `--format png`/`jpg`. Se [Indfangningseksportniveauer](#capture-export-levels-the-all-default).
> - **`lattice dls` kræver `--mac`.**Der er ikke længere nogen standard-MAC (det var en enheds adresse); `daq pool-discover --only ble` finder en sensors.**`lattice dls --camera SERIAL`** er nyt.
> - **Flere fejl afslutter `1`.** `process`, når en kørsels eksport ikke kunne verificeres, eller den mistede et billede, selvom de andre produkter er skrevet; en indlæsningskommando med en hvilken som helst fil, den ikke kunne skrive (de øvrige filer gemmes stadig); `lattice continuous` og `multi-stream` ved et fuldt drev; `array-burst` / `array-record`, når optageren rapporterer en fejl; `array-build-video`, når et anmodet produkt blev sprunget over eller ikke blev oprettet som anmodet; `project burst` / `stream`, når en optagelse mislykkes undervejs; `project sensor log`, når optagelsen ikke indeholder nogen aflæsning; `lattice exposure --set`, `gain --set` og `configure --set` med en værdi, som kameraet afviser. `lattice network-analysis --json` afsluttes på samme måde som oversigten (`1` for `needs_force_slip`). Omvendt: `daq pool-disconnect` fra en sensor, der ikke længere er i puljen, afslutter `0`X. Se [Afslutningskoder](#exit-codes).
> - **En langsom `lattice array-connect` fejler aldrig, når arrayet er tilsluttet.** Den venter op til 600 s (180 s i 1.2.0; `project connect` 90 s) og udskriver backendens trin undervejs, og en forbindelse, der ikke er oprettet inden da, annulleres af backenden — hvert kamera sættes tilbage, som det blev fundet — så en `Error:` (afslutning `1`) betyder, at der ikke er tilsluttet noget array. I 1.2.0 kunne en tilslutning umiddelbart efter, at kameraerne var tændt, vare længere end ventetiden og tilslutte arrayet, efter at kommandoen var mislykket. Ctrl+C afbryder nu tilslutningen. Se [Vedvarende forbindelser](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` bevarer den fulde sensor.** Uden `--width` / `--height` / `--binning` indstilles hvert kamera til fuld sensor, og billedfrekvensen falder til det, som denne computer og dette netværk kan levere; forbindelsesudskriften viser hurtigere muligheder (binning eller et mindre ROI) med deres forventede hastigheder. 1.2.0 komprimerede eller binede billederne for at holde hastigheden oppe. Se [`array-connect`-indstillinger](#array-connect-options).
> - **Et langsomt netværk afviser ikke længere et array.** Hvert kamera tilpasses sin andel af båndbredden, og billedfrekvensen falder i takt hermed. Hvor kameraerne ikke kan køre så langsomt, kører arrayet i seriel udsendelse (`slip-emit-and-capture`): hver ramme ankommer, men kameraerne eksponeres ikke samtidigt. `--force-tier sim-capture-sim-emit` insisterer på samtidig eksponering. Se [Array fps &amp; burst-model](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` angiver, hvilken connect der gælder**: uden `--pixel-format` fastlægger connect formaterne (Mono12 / BayerRG12, tidligere BayerRG8) samt valget af [pakket 12-bit](#packed-12-bit-transport---packed). Den samme kommando kan derfor angive en anden billedfrekvens end i 1.2.0.
> - **Relative stier henviser til det sted, hvor du kører kommandoen.** `-o` og de andre stier, som backend&#x27;et åbner (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), et opskriftstrins `output`, og standardstien `output/` for `lattice index --live` opløses i forhold til den mappe, hvorfra du kører `chloros-cli`. I version 1.2.0 løste backend-modulet dem i sin egen installationsmappe, hvilket på en installeret build resulterede i fejlen »Adgang nægtet«. Se [Konventioner](#conventions).
> - **`lattice index` kører på backend&#x27;en i alle former**, ligesom de andre `lattice`-kommandoer, så start først backend&#x27;en. Offline-indekset (`--input`) beregnes af backend&#x27;en, som læser TIFF-filen og skriver resultaterne ved siden af den eller i `-o`, og `--list-presets` / `--list-gradients` udskriver backendens tabeller. På Windows fungerer disse nu: Tidligere svarede Windows-CLI på alle former for `lattice index`, herunder `--live`, med `Error: lattice_sdk not available`. På Linux kørte offline-indekset tidligere i CLI&#x27;en uden backend og uden login; nu kræver det begge dele, ligesom alle andre CLI-kommandoer. Se [Indeks / Vegetationsmatematik](#index--vegetation-maths).
> - **Nye filer ved siden af produkterne.** Hvert radiance-, reflektans- og indeksprodukt, hvor båndsepareringen har begrænset en pixel, får en maske i en `clamp_masks/`-mappe ved siden af produktets mappe, og udskrifterne indeholder en `clamp mask:`-linje. Se [Unmix-clamp-maske](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Kameraretning.** En optagelse med et enkelt kamera gemmer `raw` og `debayered`, som sensoren har aflæst dem, og registrerer kameraets spejling/rotation i XMP; `process` anvender denne information på de produkter, det fremstiller. Array-medlemsfiler forbliver i deres oprindelige format, medmindre du overfører den nye `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Se [Kameraretning](#hvordan-en-optagelsesmappe-ser-ud).
> - **Et eksportniveaunavn, der ikke er et, afvises.** `lattice capture --processing` og alle `--levels`-lister accepterer `raw`, `debayered`, `radiance`, `reflectance` eller `display` (eller et alias såsom `preview`; `--processing` samt `all`). Ethvert andet navn afvises, før kameraet berøres (HTTP 400 `UNKNOWN_LEVEL`, afslut `1`); 1.2.0 gemte `raw` til det og afsluttede `0`.
> - **`daq pool-set-cap` kræver `--cap-id`.** Uden det afsluttes kommandoen `2`; 1.2.0 fjernede kap-korrektionen, hvilket efterlod en bar sensor. `--cap-id none` beder om den rene sensor.
> - **`lattice resolution --binning-mode` er væk.** Binning er altid Average; flaget accepteres stadig, udskriver en advarsel og gør intet. Se [Sensorjustering](#sensor-tuning).
> - **`process --debayer texture-aware` afvises for LATTICE-optagelser.** For en mappe, der indeholder LATTICE-optagelser, afslutter `process` `1` efter importen og angiver, hvor mange der er, inden der behandles noget. LATTICE-optagelser får altid standard-demosaic, så 1.2.0 accepterede indstillingen og ignorerede den; Chloros-appen tilbyder den heller ikke til dem. Se [`chloros-cli process`](#chloros-cli-process).
> - **Kameraerne vender tilbage til den tilstand, de blev fundet i — medmindre `--keep-settings`.** `lattice array-disconnect`, hver midlertidig kommando (`multi-capture` uden array, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) og `lattice continuous` sætter hvert kamera tilbage, som det var: dets udløser- og synkroniseringsledningsføring, dets PTP samt alle kameraindstillinger, der blev ændret under sessionen (eksponering og forstærkning, AE-mål, grænser og dæmpning, hvidbalance, sortniveau, pixelformat, binning og ROI, spejling, billedhastighed). `--keep-settings` gemmer i stedet hvert kamerasindstillinger i kameraets egen hukommelse, så det starter op med dem; `--clear-settings` nulstiller denne hukommelse til fabriksindstillingerne. Uanset hvad kører kameraerne frit. Et projekt indstiller sine kameraer ud fra sine egne gemte indstillinger, når det tilslutter dem. Se [Kamerahukommelse](#camera-memory----keep-settings----clear-settings).
> - **Alle tilstande gemmer et projekt på samme måde.** Enhver ændring af et kamera, en matrix eller en lyssensor, der er tilsluttet under et projekt — fra appen, CLI, SDK eller MCP-serveren — gemmes i projektets `cameras.json` / `sensors.json`, og når projektet åbnes i en hvilken som helst tilstand, indstilles enhederne ud fra det. Tidligere gemte kun appen projektet, og den næste gemning overskrev alt andet. Se [Én projektpost, alle tilstande](#one-project-record-every-mode).
> - **Filer, der er oprettet fra flere kameraer, navngiver hvert enkelt.** En multibåndsstak (`align-apply`, `align-stream`, `index --live --save-multiband`), et live-indeks lavet af flere kameras bånd og et array-optagelsesbillede med navnet `composite/` indeholder hver især hvert kildekameras serienummer, model og kalibreringspakke som lister (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), og EXIF-listerne `SerialNumber` indeholder serienumrene adskilt med komma. Tidligere indeholdt de slet ingen kameraoplysninger eller kun det første kameras serienummer. Se [Filer oprettet fra flere kameraer](#files-made-from-several-cameras).
> - **Kun en `raw`-fil indeholder kameraets kalibreringsmærker.** Det MicaSense-kompatible sæt (`BlackLevel`, `RadiometricCalibration`, det kanalbaserede `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`-sættene) samt `CalibrationJSON`-payloadet beskriver sensorens råværdier, så de findes udelukkende i `raw`-optagelsesfilen. Alle andre filer — en optagelses `debayered`-, `display`-, `radiance`-, `reflectance` og indeksfiler, `lattice continuous` og `multi-stream`-rammer, og hvert `process`-produkt — indeholder kameraets identitet (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) og, ved en optagelse, rammens egen registrering (`XMP-mapir:FrameJSON`: eksponering, forstærkning, temperatur, pixelformat, optagelsestidspunkt). I version 1.2.0 indeholdt de også rådatasættet, som et værktøj, der anvender det (Pix4D, Metashape) anvendte på pixels, der ikke er råværdier. `process` finder en sådan fils kalibrering ud fra dens serienummer og `CalBundleSha256`.
>
> **Andre ændringer:**
>
> - **En arrays automatiske billedhastighed tilpasses det, som arrayet leverer.** En hastighed, der mister mere end 1 % af sine udløsninger, sænkes – vurderet ud fra flere længerevarende observationer frem for én enkelt – og en hastighed, som den er blevet sænket fra, forsøges af og til igen. En hastighed, du selv indstiller, forbliver uændret.
> - **Den automatiske eksponering er mere stabil.** Den holder sig konstant ved konstant lys, stabiliserer sig inden for få sekunder efter en ændring, ignorerer små højlys og genopretter et kamera, der holder op med at reagere i stærkt sollys. Se [Sådan fungerer automatisk eksponering](#how-auto-exposure-behaves).
> - **`process` tæller kun dine billeder og optagelser.** Kører man programmet igen på en mappe, der indeholder et projekts `camera_cal/`, tæller den ikke længere disse kalibreringsfiler med („Fundet 96 billeder” på et sæt med 90 billeder), og et projekt, der allerede har indlæst dem, fjerner dem.
> - **`lattice multi-test` starter fra kameraernes tændte tilstand.** Det aktiverer kameraernes PTP til kørslen, venter på, at deres ure låser sig fast og stabiliserer sig inden den første udløsning — hvert kamera låses fast i 120 s, ca. 2 minutter fra tænding (`--ptp-settle SECONDS` indstiller det), da et ur, der netop er låst, afviger med op til ~200 us i ca. 2 minutter — og sætter hvert kameras PTP tilbage til den tilstand, den fandt det i. Der ventes ikke igen på et kamera, der stadig er låst fra en tidligere test. Tidligere blev PTP slået fra på et netværk med en enkelt switch, hvor ingen DAQ-E svarede, og hvert kameras fritløbende ur blev aflæst som et PTP-ur, der ikke ville synkronisere (`PTP is not locked`, ved hver kørsel). Se [Multi-kamera (midlertidige sessioner)](#multi-camera-transient-sessions).
> - **En `.daq`-optagelse angiver, om dens hætte blev valgt eller antaget.** DAQ-U, DAQ-M og DAQ-E leveres uden dæksel, og Chloros kan ikke se, om der er monteret et, så en sensor, der tilsluttes uden et navngivet dæksel, får som standard `sunshine_cosine`, en *antagelse*. Hver Chloros-optagelse (skema 1.30) gemmer nu `cap_id_source` ved siden af `cap_id`: `operator`, hvis du har navngivet en hætte (herunder `none`), og `auto_default`, hvis det var standardindstillingen. Ved behandling af en optagelse, hvor hætten blev antaget, og hvor projektets DAQ-hætte er indstillet til Auto, udskriver en `[daq-dls] WARNING`, der navngiver filen: en ren DAQ-U under solskinnskorrektionen viser en værdi, der er ca. 30 gange for høj. **Navngiv den cap, du har monteret**, når du tilslutter, eller indstil projektets DAQ-cap, så forsvinder advarslen.
> - **Ingen flere pip-konsolvinduer på Windows.** På en pc med Python installeret kørte backend og hver af dens billedbehandlere `pip` én gang ved opstart, og hver åbnede et konsolvindue. Det gør de ikke længere.
> - **Live `.daq`-optagelser overlever et strømsvigt.**En optagelse skrives med en journal og en fuld disksynkronisering ved hver commit, bag sensoren, så en langsom disk forsinker filen, men aldrig målingerne. En optagelse, der stoppes midt i en commit (strømafbrydelse, afbrydelse), kan efterlade `<name>.daq-journal` ved siden af:**kopier de to sammen.** Chloros ruller journalen tilbage, første gang den læser filen, så optagelsen åbnes med alt, der blev commitet før stop (ca. de sidste 3 sekunder går tabt); i 1.2.0 kunne et strømsvigt på det forkerte tidspunkt gøre hele filen ulæselig („database disk image is malformed“).
> - **En optagelse på en travl disk blokerer ikke længere sensoren.** `daq pool-record` sætter aflæsninger i kø og skriver dem på en separat tråd; en aflæsning, der ikke kunne skrives, tælles (`daq pool-list` viser `N write error(s)` med den sidste fejl) i stedet for at fryse sensoren (én skrivning tog 8,4 sekunder ved siden af fire kameraer, der gemte TIFF-filer).
> - **`lattice multi-capture` uden et array mister ikke længere de billeder, den har gemt, når en trigger går i timeout.** En kameras trigger, der går i timeout (`SC_ERR_TIMEOUT -1011`, observeret med fire kameraer på én 1 GbE-switch), afslutter kørslen ved den pågældende billedgruppe: de allerede gemte grupper vises med `Saved:`, derefter fejlen, exit `1`. I version 1.2.0 var det en HTTP-500-fejl uden nogen liste, selvom filerne fandtes på disken.
> - **Softwareudløste kameraer dropper ikke længere hver anden udløsning.** Et kamera i softwareudløsningstilstand sætter nu en udløsning, der ankommer under dets udlæsning, i kø (eller Chloros venter på, at den er klar), ligesom kabelsynkroniserede kameraer allerede gjorde.
> - **`daq pool-discover` navngiver en DAQ-E-S `DAQ-E-S`** i sin `MODEL`-kolonne (og `--json`’s `model`). Der stod `DAQ-E` for hver Ethernet-sensor; en, der ikke svarer på identitetskontrollen inden for ca. 2 sekunder, er stadig opført som `DAQ-E`.
> - **Forbedrede LATTICE-indeksprodukter** (NDVI og andre indekser) fra `process` i alle behandlingsmetoder.
> - **OCN-kameraer: forbedrede radiance-, reflektans- og indeksprodukter i sollys.** Der anvendes en flådekorrektion efter båndsepareringen; RGN-kameraer er uændrede, og OCN-værdierne adskiller sig fra version 1.2.0. Dette gælder også for en OCN-optagelse, der er lavet med en tidligere version, når den behandles på en computer, der indeholder den kalibrering, som optagelsen blev taget med (Chloros bevarer et kameras kalibrering, når kameraet først er blevet tilsluttet).
> - **`lattice continuous` registrerer råbilledet.** En `tiff`-kørsel skrev demosaicerede 3-plan-rammer, som `process` ikke kan tage udgangspunkt i. Dens rammer er nu den `raw`-fil, som en array-optagelse skriver: ét mosaikplan med sensorens egne værdier, ukomprimeret, med kalibreringspakken. Et script, der læser tre planer fra disse filer, læser kun ét.
>
> - **Hver optagelse gemmer den DAQ-aflæsning, den har brugt, ligesom GUI’en gør.** `lattice capture`, `lattice continuous` (én `.daq` pr. ramme), `array-capture` og `multi-capture` med et tilsluttet array (og SDK&#x27;en) skriver `.daq` og dets `.match.json`, hver gang en DAQ bindes eller samles i en pulje, herunder også en optagelse, der udelukkende består af rådata; i 1.2.0 gemmes en rå eller debayered optagelse kun sammen med `--force-daq`. `--no-daq` deaktiverer det, `--daq-tolerance-ms MS` indstiller matchtolerancen for den pågældende optagelse, og den nye `daq pool-bind` binder et kamera eller et array til en DAQ fra CLI&#x27;en. Med `.daq` matches en optagelse, der gemmes, i `process` kun med de kameraer, den navngiver. En optagelse venter nu på DAQ’ens næste aflæsning, så den kan blande aflæsningerne på begge sider af eksponeringen. Se [DAQ-aflæsningen, som en optagelse gemmer](#the-daq-reading-a-capture-saves) og [En DAQ-aflæsning langt fra eksponeringen](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** tilføjer indekset til projektet for hvert kameras bånd, herunder et nyt projekt. I 1.2.0 blev et navn, som projektet endnu ikke indeholdt, ignoreret med `Unknown index name(s) ignored`.
> - **`process` ved siden af et live-array:**genstarter ikke længere et backend, der indeholder kameraer eller DAQ&#x27;er, og venter op til 15 sekunder på en kørsel, der er ved at afsluttes.**Ctrl+C** stopper kørslen i backenden, ikke kun i CLI&#x27;en. Se [`process`-noter](#notes).
> - **En projektmappe, der er for dyb i forhold til Windows’ begrænsning på 260 tegn i stien**, afvises med det samme med den længste mappe, der passer, i stedet for et kørsel, der ikke skriver noget. Windows-installationsprogrammet tilbyder nu »Aktiver lange Windows-stier« og en Microsoft Defender-undtagelse for backend&#x27;en, som begge er markeret som standard. Se [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** kører backendens egen kontinuerlige indsamling: en statuslinje hvert 2. sekund, et resumé i stedet for en filvis oversigt og en afslutning med en værdi forskellig fra nul, hvis der er uindlæste filer eller et fuldt drev. Se [`array-capture`-tilstande](#array-capture-modes).
> - **Den første `array-capture` efter `array-connect`** venter op til 20 sekunder på arrayets første billedgruppe i stedet for at returnere en HTTP-fejl 503 &quot;Array still warming up&quot;.
> - **Array-eksponering under et ROI:** Et lille ROI begrænser ikke længere et array-medlems eksponering til sensorens frie løbetid (4,9 ms ved 768 × 304). Se [Eksponering på et array-medlem](#exposure-on-an-array-member).
> - **Automatisk eksponering sænker ikke længere et arrays automatiske billedhastighed.** Et kabelsynkroniseret array kører med den hastighed, som netværket og kameraernes udlæsning tillader; automatisk eksponering udfylder den periode, som denne hastighed giver, og hæver derefter forstærkningen. En manuel eksponering, der er længere end perioden (`lattice exposure --set`, `configure --set`), sænker udløsningen for at tilpasse sig den i stedet for at blive afvist, og frekvensen vender tilbage, når elementet skifter tilbage til automatisk. Se [Eksponering på et array-medlem](#exposure-on-an-array-member).
> - **Arrays leverer færre ufuldstændige billedgrupper** og færre 503-fejlmeddelelser („Ingen ny synkroniseret billedgruppe“): hver slave-kameraets udløserindgang er nu fri for glitches.
> - **Pakket 12-bit-transport** (`--packed`, standard `auto`) øger billedhastigheden for fuld-sensor-arrayer med ca. en tredjedel. `array-connect --packet-size` anvendes nu (1.2.0 accepterede det og ignorerede det). Se [Pakket 12-bit transport](#packed-12-bit-transport---packed).
> - **Et array, hvor kameraerne genopretter sig selv efter en genstart.** Efter en genstart af switchen genåbner `array-connect`&#x27;s array-åbner kameraerne, når de svarer, og streamer igen med PTP aktiveret; i 1.2.0 stoppede det helt og krævede en ny `array-connect`. `array-status` rapporterer `ptp_enabled: False` (og `ptp_off:`), mens et kamera læser PTP fra, hvor det tidligere rapporterede den indstilling, som arrayet var forbundet med. Se [Vedvarende forbindelser](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` optager uden åben GUI.** Når GUI&#x27;ens kombinerede visning er lukket, gengiver backend&#x27;et selve visningen: arrayetsNDVI eller det indeks, du angiver med den nye `--expr`. En optagelse, der ikke skriver nogen ramme, mislykkes nu (`no-frames: …`, afslut `1`). Tidligere optog den kun, mens GUI&#x27;ens kombinerede visning var åben, og en optagelse, der ikke skrev noget, afsluttede stadig `0`. Se [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Et kamera, som Chloros allerede besidder, åbnes ikke en anden gang.** En kommando, der ellers ville åbne det, returnerer i stedet HTTP-status 409 `CAMERA_HELD` og angiver, hvad der besidder det, samt den kommando, der frigiver det. Dette gælder også et kamera, som en anden kommando bruger i øjeblikket (f.eks. en `multi-test` eller `multi-capture`, der åbner sine kameraer, eller en `lattice capture`): den anden kommando afvises, indtil den første er færdig. At åbne det igen kan stoppe indehaverens stream eller få backendet til at gå ned. Se [Persistent Connections](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Midlertidige kommandoer til flere kameraer leverer alle kameraer eller mislykkes.** `lattice multi-capture` uden tilsluttet array, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` og `index --live` afslutter `1` ved at angive et kamera, der ikke leverede noget billede. Strømmene springer en billedgruppe over, hvor et kamera mangler, og stopper efter tre i træk; `multi-test` udskriver en række pr. udløser, `(no frame)` hvor et kamera gik glip af det; `align-stream` afslutter også `1` ved en jobfejl. Hver kørsel sætter derefter hvert kamera tilbage, som det fandt det: dets trigger- og synkroniseringsledningsforbindelser, dets PTP (`PtpEnable`) og alle kameraindstillinger, som kørslen har ændret — eksponering og forstærkning, AE-mål, grænser og dæmpning, hvidbalance, sortniveau, pixelformat, binning og ROI, spejling, billedhastighed — medmindre `--keep-settings` / `--clear-settings` (se [Kamerahukommelse](#camera-memory----keep-settings----clear-settings)). Tidligere bevarede disse kommandoer alle de billeder, der kom ind, og afsluttede `0` (`align-stream` udskrev en jobfejl som `Note:`), og efterlod hvert kamera i `TriggerMode` `On`. De indstillede også hvert kameras eksponering ud fra scenen, før de gemte et billede (højst et par sekunder), på det mål, hvor et tilsluttet array starter hvert kamera ved: 40 multispektrale, 60 RGB (`--brightness N` indstiller det) og beholder den indstilling under hele optagelsen; det samme gør et enkelt `lattice capture` for et kamera, som Chloros ikke styrer. Tidligere blev hvert kameras første billede taget ved kameraets egen eksponeringsgrænse, ca. 100 ms, uanset lysforholdene, og blev ofte klippet. Se [Multi-kamera (midlertidige sessioner)](#multi-camera-transient-sessions).
> - **Midlertidige multi-kamera-kommandoer deler netværket på samme måde som `array-connect` gør, og åbner kun `--serials`.** Hvert kamera tilpasses sin andel af værtsforbindelsen, så fire kameraer kører på en 1 Gbps-forbindelse; tidligere anmodede de fire om 125 MB/s til deling mellem sig, alt sammen via 1 GbE, og hvert billede mistede pakker. En konstellation, som netværket ikke kan håndtere, afvises, før nogen kameraer streamer. `--serials` vælger nu kameraerne, hvor det første udløser triggeren; tidligere åbnede hver kommando alle kameraer på netværket, uanset hvad `--serials` sagde. En fejl, der skyldes GVSP-pakketab, angiver nu dette i stedet for at pege på M8-synkroniseringskablet. Se [Multi-kamera (midlertidige sessioner)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` lader kameraerne køre frit, som de blev fundet.** `TriggerMode` `Off` og alt andet, som Connect fandt det — indstillingerne for trigger og synkroniseringslinje (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP samt de kameraindstillinger, som sessionen ændrede — så en enkelt-kamera-kommando på et tidligere medlem henter alle billeder på én gang, og masteren forbliver ikke længere på `TriggerSource` `Software`. `--keep-settings` gemmer sessionens indstillinger i hvert kameras hukommelse, mens `--clear-settings` nulstiller denne hukommelse (se [Kamerahukommelse](#camera-memory----keep-settings----clear-settings)). `lattice continuous` indstiller desuden et kamera, som den finder aktiveret, til frit løb under løbet (og sætter det tilbage bagefter), og et løb uden ramme afsluttes med `1`.
> - **`lattice configure --get` / `lattice exposure`** udskriver en `note:` med kameraetsfaktiske eksponering eller forstærkning, mens dets automatiske tilstand kører. Et afvist `lattice exposure --set` / `configure --set` angiver det interval, kameraet accepterer, og sætter den automatiske tilstand tilbage i stedet for at lade den automatiske eksponering være slået fra. Se [Sensorjustering](#sensor-tuning).
> - **Nye kamerakontroller:**`lattice highlights` (træk lyse hvide overflader ned i RGB-visningen) og `lattice exposure --brightness-auto`.**`lattice multi-detect`** finder de kabelførte kameraer i én sondering og indstiller masteren som standard til RGB-kameraet.
> - **Nye DAQ-kommandoer:** `daq pool-bind`; `pool-enable-ptp`; og til en DAQ-E&#x27;s egne indstillinger: `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` og `pool-update-firmware`. Se [DAQ-E-enhedsindstillinger](#daq-e-device-settings).
> - **En DAQ-E, der holder op med at levere data, genforbindes af backend&#x27;en.** Når strømmen fra en DAQ-E i puljen går i stå (f.eks. ved en genstart af den switch, der forsyner den med strøm, et afbrudt kameralink), genopretter backend-systemet forbindelsen med de tidligere indstillinger og forsøger igen med stigende intervaller, mens enheden er væk. En optagelse på enheden fortsætter i den samme `.daq` med afbrydelsen markeret i filen, og `pool-record --stop` viser hvert afbrud. I version 1.2.0 forblev sensoren tavs indtil `pool-disconnect` + `pool-connect`, hvilket også afsluttede optagelsen. Se [En DAQ-E, der holder op med at levere](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** rapporterer, om PTP-grandmasterens pakker når frem til netværket (`health`), og der vises en `WARNING:` på stderr, når de ikke gør det. Se [`time-sync`](#chloros-cli-time-sync).
> - **Linux .deb:** Den øger kernens socket-modtagelsesbuffere, som kamerastrømme har brug for (ved kernens standardindstilling går der billeder tabt, og billedfrekvensen falder), tilføjer en backend-tjeneste, der kører som din bruger, og `lattice network` kontrollerer bufferne. CLI&#x27;en stopper aldrig en backend, som en tjeneste kører, og en opgradering starter de aktiverede backend-tjenester igen. Se [Kørsel af backenden som en tjeneste](#running-the-backend-as-a-service-headless) og [Linux-værter — socket-modtagelsesbuffere](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` angiver, når det ikke var muligt at justere:**hver årsag til, at en optagelse blev gemt uden justering (ingen justeringsprofil, medlemmer med forskellige opløsninger, et array, der ikke er kabelsynkroniseret, en ramme, som profilen ikke kan kortlægges til) vises som en `Alignment:`-linje.**`chloros-cli daq help`** er `daq --help` (det afsluttede `2`), og på Linux angiver hjælpen de direkte hardwarekommandoer , som denne build indeholder.
> - **Jetson:** pakken er `chloros_1.3.0_arm64_jp6.deb` til JetPack 6, og dens GPU-behandling kører på Orin med JetPacks CUDA PyTorch. Se [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Konventioner

- Alle kommandoer har præfikset `chloros-cli`. På Windows er binærfilen `chloros-cli.exe`; på Linux/Jetson er det `chloros-cli`.
- Valgfrie argumenter vises som `--flag`. Obligatoriske positionsargumenter vises uden parenteser.
- Hvor der er angivet en standardværdi, anvendes denne værdi, hvis flaget udelades.
- CLI’en er en letvægts-HTTP-klient via Chloros-backend’et (serveren på `127.0.0.1:5000`, som desktop-appen også bruger). **Hvilke kommandoer starter den:**
  - De centrale kommandoer — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` og `selftest` — bruger et kørende backend eller starter et, når der ikke kører noget, og stopper det igen, når de er færdige.
  - **Kommandoerne `lattice`, `project` og `daq` starter aldrig en.** De kræver en backend, der allerede kører: Windows-desktopappens, Linux-[backend-tjenesten](#running-the-backend-as-a-service-headless) eller en, du selv starter (se [Start backenden](#start-the-backend)). Uden en sådan viser de fejlmeddelelsen `Error: Chloros backend not reachable at http://127.0.0.1:5000` sammen med den kommando, der starter den, og afslutter derefter med `1`.
- `CHLOROS_BACKEND_URL=<url>` henviser til **`lattice`**,**`project`**og**`daq pool-*`** kommandofamilierne til en anden backend (`lattice network-analysis` peger i stedet på `--backend-url`). Kernkommandoerne fastlåser bevidst `http://127.0.0.1:<port>` og ignorerer den (IPv4-literalet undgår Windows’ `localhost`→`::1`-straf på ca. 2 sekunder pr. anmodning). Se [Miljøvariabler](#environment-variables).
- **Relative stier opløses i forhold til den mappe, du kører `chloros-cli` fra.** Backend&#x27;et er den proces, der åbner output- og input-mapperne, så inden en sti sendes, gør CLI&#x27;en den absolut: `-o` / `--output` (inklusive standarden, `output`), `--input`, `--burst-dir`, `--profile`, `--save` og `--mask`, samt [opskrift](#recipe-dsl)-trinnets `output` (standard `output`), samt standardmappen `output` for `lattice index --live`. `-o captures/` skriver derfor til `<current folder>/captures/`, og de udskrevne stier er absolutte. `~` udvides. Ved brug af en fjernbackend (`CHLOROS_BACKEND_URL`, der angiver en anden maskine) sendes stierne som indtastet og løses på den pågældende maskine.
- Der kræves en Chloros+-konto for alle SDK/CLI -opkald (kør `chloros-cli login` én gang pr. maskine; caches i `~/.chloros/`).
- Eksemplerne bruger Linux-stier; på Windows skal du erstatte `/home/user/...` med `C:/Users/.../...`. En DAQ-U&#x27;serielle port er af typen `COM3` på Windows og `/dev/ttyUSB0` / `/dev/ttyACM0` på Linux.
- Oversigterne angiver gensidigt eksklusive flag som `{--a | --b}` og valgfrie flag som `[...]`.

---

## Synopsis på øverste niveau

```
chloros-cli [global options] COMMAND [command options]
```

### Globale indstillinger

| Flag | Beskrivelse |
| --- | --- |
| `--backend-exe PATH` | Tilpas den automatisk detekterede backend-eksekverbare fil. |
| `--port N` | Backend-HTTP-port (standard: `5000`). Anvendes kun af kernekommandoerne (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); `lattice`, `project` og `daq` bruger `CHLOROS_BACKEND_URL` (standard `http://127.0.0.1:5000`). |
| `-v, --verbose` | Aktiver detaljeret output. |
| `--restart` | Tvinger genstart af backend (afslutter eventuelle kørende `backend_server.py`). En backend, der køres af en systemd-tjeneste (Linux .deb) stoppes aldrig: kommandoen udskriver i stedet `sudo systemctl restart <unit>` og afsluttes `1`. |
| `--version` | Vis version (`Chloros CLI 1.3.0`). |
| `--help` | Vis hjælp på øverste niveau. |

### Kommandoindeks

| Kommando | Formål |
| --- | --- |
| [`process`](#chloros-cli-process) | Behandler en mappe med Survey3- eller LATTICE-optagelser fra start til slut. |
| [`login`](#chloros-cli-login) | Godkend denne maskine med en Chloros+-konto. |
| [`logout`](#chloros-cli-logout) | Ryd cachelagrede legitimationsoplysninger. |
| [`status`](#chloros-cli-status) | Vis den aktuelle licens-/godkendelsesstatus. |
| [`export-status`](#chloros-cli-export-status) | Live-status for Thread-4-eksport under en `process`-kørsel. |
| [`language`](#chloros-cli-language) | Indstil eller vis CLI-sprog (38 understøttede sprog). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Standardprojektmappe (delt med GUI&#x27;en). |
| [`update`](#chloros-cli-update) | Søg efter og installer Chloros-opdateringer (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Systemdiagnostik + smoke-tests. |
| [`time-sync`](#chloros-cli-time-sync) | PTP-grandmaster-status / -styring. |
| [`lattice`](#chloros-cli-lattice) | LATTICE-kamerastyring og -optagelse (over 45 underkommandoer). |
| [`daq`](#chloros-cli-daq) | DAQ-spektralsensorstyring (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Åbn og kør et gemt Chloros-projekt (kameraer + DAQ&#x27;er). |

---

## Installation

`chloros-cli` leveres som en del af Chloros-desktopinstallationsprogrammet på alle understøttede platforme — der findes ingen separat CLI-download. Når du installerer platformspakken, tilføjes `chloros-cli` til din `PATH` sammen med desktop-appen og den bagvedliggende binærfil, den kører.

Seneste downloads: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Installationsprogrammet indeholder også praktiske startskripter (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), der åbner en brugsklar CLI-shell.

### Windows (.exe)

1. Download Windows-installationsprogrammet fra downloadsiden.
2. Kør `Chloros Installer 1.3.0.exe`, og følg vejledningen. Standardinstallationsstien er `C:\Program Files\Chloros\` (CLI&#x27;en placeres i `C:\Program Files\Chloros\cli\`, som installationsprogrammet tilføjer til PATH).
3. Åbn en ny terminal (`cmd.exe`, PowerShell eller Windows-terminalen), så den opdaterede `PATH` bliver fundet.

```powershell
chloros-cli --version
```

Installationsprogrammet tilføjer automatisk `chloros-cli.exe` til dit systems `PATH` og inkluderer LATTICE-kameradriverne. Backend&#x27;et er `C:\Program Files\Chloros\backend\chloros-backend.exe` (mappen `backend` i din installationsmappe), som ikke findes på `PATH`.

**Systemindstillinger.** Installationsprogrammets side *Vælg yderligere opgaver* har to indstillinger under **Systemindstillinger**, som begge er**markeret som standard**:

| Indstilling (som angivet i installationsprogrammet) | Hvad den gør | Fravælg lydløs installation |
| --- | --- | --- |
| **Aktiver lange stier i Windows** (giver mulighed for, at projektmapper kan ligge dybere end 260 tegn; anbefales) | Indstiller Windows&#x27; systemomfattende indstilling `LongPathsEnabled` til 1. Windows leveres med denne indstilling slået fra, og så må ingen sti overstige 260 tegn; Chloros-produkter ligger op til ca. 130 tegn under projektmappen, så en dybere projektmappe afvises (se [Fejlfinding](#fejlfindings-tip)). Træder i kraft for programmer, der startes efter installationen. Forbliver aktiveret ved afinstallation, da andre programmer muligvis er afhængige af indstillingen. | `/MERGETASKS="!longpaths"` |
| **Ekskluder Chloros-backend fra Microsoft Defender-scanning**(meget hurtigere kameraoptagelse; anbefales) | Tilføjer en Defender**proces**-ekskludering for `chloros-backend.exe` i installationsmappen, så realtidsbeskyttelsen ikke scanner hver eneste fil, som backend&#x27;en skriver, uanset hvor den skriver den. Ingen andre programmer udelades; udelukkelsen fjernes ved afinstallation. Hvis Defender ikke er til stede eller er låst af en politik, mislykkes trinnet uden fejlmeddelelse, og installationen fortsætter. | `/MERGETASKS="!defenderexclusion"` |**Hvorfor Defender-indstillingen er vigtig:** Ved høje optagelseshastigheder udgør realtidsscanning af optagelsesfilerne ca. 30 % af optagelseshastigheden (målt ved kontinuerlig optagelse med 9 kameraer: 77,7 optagelser/sek. udelukket, 52,7 scannet). `lattice array-capture --continuous` viser en besked, når det måler, at Defender bruger en CPU-kerne eller mere under optagelsen; kør installationsprogrammet igen med indstillingen markeret.

En lydløs installation (`/SILENT` eller `/VERYSILENT`) anvender begge, medmindre det fravælges, f.eks. `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Windows-firewall.** Installationsprogrammet tilføjer altid to indgående regler for `chloros-backend.exe` på hver netværksprofil, og afinstallationsprogrammet fjerner dem: `Chloros Backend (UDP-In)`, til kamerastrømme, PTP, mDNS og DAQ-E-datastrømmen samt `Chloros Backend (TCP-In)`, der er begrænset til det lokale undernet, til en DAQ-E-firmwareopdatering, hvor sensoren opretter forbindelse tilbage til denne computer for at hente billedet (se [DAQ-E-enhedsindstillinger](#daq-e-device-settings)). Backendens HTTP-API forbliver bundet til `127.0.0.1`.

### Linux amd64 (.deb)

Til Ubuntu 22.04 LTS eller nyere / Debian-baserede x86_64-arbejdsstationer.

> **Ubuntu 20.04 understøttes ikke.** Pakkens liste over afhængigheder er afledt af
> det, som backend&#x27;et rent faktisk linker til, og det inkluderer `libc6 (>= 2.35)`;
> focal leveres med glibc 2.31. `apt` afviser installationen i stedet for at lade den mislykkes under
> kørsel. Listen nævner også `libgl1`, som det medfølgende OpenCV indlæser, og som en minimal
> serverimage muligvis mangler; `apt install` leverer det.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

.deb-filen installerer:
- `chloros-cli` til `/usr/bin/chloros-cli` og `chloros-backend` (starter backend&#x27;en) til `/usr/bin/chloros-backend`
- Den kompilerede backend og CLI, udpakket, i `/usr/lib/chloros/backend/` og `/usr/lib/chloros/cli/`
- LATTICE-kameradriverne
- Støjfjernelsesmodeller, kalibreringspakker og konfiguration af opdateringskanalen
- Python-SDK&#x27;en til systemet `python3` (`import chloros_sdk`). `apt install` medfører også sine egne krav, `python3-numpy` og `python3-requests` (anbefalet, ikke påkrævet: ovenstående programmer er selvstændige; `dpkg -i` springer dem over, og installationsudskriften angiver, hvad der mangler). I et virtualenv: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
- `/etc/sysctl.d/60-chloros-gige.conf`, som øger kernens socket-modtagelsesbuffere for kamerastrømme strømme til 32 MB og anvender dem med det samme (en værdi, der allerede er indstillet højere, forbliver uændret). Fjernelse af pakken fjerner filen. Se [Linux-værter — socket-modtagelsesbuffere](#linux-hosts--socket-receive-buffers).
- To valgfri systemd-tjenester til backend&#x27;en, ingen af dem er aktiveret (næste afsnit)

#### Kørsel af backend&#x27;en som en tjeneste (headless)

Uden en tjeneste kører backend&#x27;en (og dermed PTP-grandmasteren) kun, så længe noget har startet den: `chloros-backend` i en terminal, et SDK-script eller en core-kommando `chloros-cli` i hele dens kørselstid. For at få en backend, der altid er tændt på en headless-maskine, skal du aktivere tjenesten **som din bruger**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` kører backend&#x27;en som den pågældende bruger. De data, den skriver, tilhører denne bruger, og dens detaljerede log findes i brugerens `~/.cache/chloros/logs`, og den bruger den pågældende brugers `chloros-cli login` (`~/.chloros`), så log først ind én gang som den pågældende bruger. PTP kræver ikke root-adgang. Ligesom alle installerede backends lytter den kun på porten `127.0.0.1:5000`.

`chloros-backend.service` kører backend&#x27;en som **root**. Det er stadig installeret, så en maskine, der har aktiveret det, fortsætter med at køre, men alt, hvad en root-backend skriver, tilhører root, så en bruger kan ikke slette sine egne optagelser. For at skifte til din bruger:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

Root-tjenestens detaljerede log findes i `/var/log/chloros` og kan læses af medlemmer af gruppen `adm` (tjenesten indstiller `CHLOROS_LOG_DIR`; se [Miljøvariabler](#environment-variables)). Når pakken installeres, vises kommandoen `enable` med dit brugernavn udfyldt samt angivelsen af, om root-tjenesten er aktiveret.

**Aktivér én backend-enhed.** Der kører kun én backend ad gangen. En anden aktiveret enhed (`chloros-backend.service` ved siden af `chloros-backend@<user>.service` eller to brugeres `@`-enheder) nægter at starte og forbliver som `failed` med afslutningsstatus 98; systemd genstarter den ikke. Dens `systemctl status` angiver årsagen: `Another instance is already running. Exiting.`, derefter hvad der holder fast i backend&#x27;en (port 5000 eller låsefilen og den bruger, den tilhører), derefter `systemd will not restart <unit> (exit status 98)`. Efter en genstart kan begge enheder være den, der kører. For at skifte skal du deaktivere den, du ikke bruger, som beskrevet ovenfor.**CLI&#x27;en stopper aldrig en tjenestes backend.** `chloros-cli` bruger den, som den er, ligesom GUI&#x27;en og SDK&#x27;en gør: `process` kører på den og indlæser sit eget projekt i stedet for det sidste. Hvor CLI&#x27;en ellers ville genstarte backend&#x27;en (`--restart` eller en `process`, hvis import ikke fandt noget), navngiver den enheden, udskriver `sudo systemctl restart <unit>` og afslutter `1`, fordi et backend, der stoppes på den måde, ville forblive nede. `GET /api/status` navngiver enheden som `service` (`null` for et backend, hvor ingen tjeneste kører).**Opgradering og fjernelse.** Opgradering af pakken stopper alle backend-tjenester og starter derefter hver enkelt, der er aktiveret, igen, først enhederne `@<user>` (en, der er startet manuelt, men aldrig aktiveret, forbliver stoppet, ligesom efter en genstart). Hvis der er mere end én aktiveret, kører kun én, og de øvrige forbliver i fejltilstand som ovenfor; opgraderingen viser `Chloros: N backend units are enabled: …` og `Keep one enabled: sudo systemctl disable --now <unit>`. Fjernelse stopper og deaktiverer dem. Hvis den renses, slettes også `/var/log/chloros`.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Samme opbygning som amd64 .deb-filen, inklusive modtagebuffere og [backend-tjenester](#running-the-backend-as-a-service-headless), til Jetson Orin / Orin NX / Orin Nano på **JetPack 6** (L4T R36). Dens PyTorch er JetPack 6’s CUDA-build til Orin, så GPU-behandlingen – inklusive den teksturbevidste debayer – kører på Jetsons GPU. Pakken indeholder ikke selve CUDA: den bruger de CUDA- og cuDNN-biblioteker, som JetPack 6 installerer (`sudo apt install nvidia-jetpack` på et image, der mangler dem). Hvor der ikke kan bruges en CUDA-GPU, kører behandlingen på CPU&#x27;en, og den teksturbevidste funktion bruger standard-debayeren (se [`process`](#chloros-cli-process)).

### Autentificer én gang pr. maskine

Hver platform kræver en engangs-login til Chloros+, før SDK- og CLI-kald fungerer:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Loginoplysningerne gemmes i `~/.chloros/user_session.json`.

### Start backend&#x27;en

Kommandoerne `lattice`, `project` og `daq` kommunikerer med en backend, der allerede kører; de starter aldrig en (se [Konventioner](#konventioner)). Sørg for, at der kører et backend, mens du bruger dem:

| Platform | Sådan |
| --- | --- |
| Windows | Åbn Chloros-desktopappen (dens backend betjener også CLI), eller kør `"C:\Program Files\Chloros\backend\chloros-backend.exe"` i en separat terminal (mappen `backend` i din installationsmappe) og lad den være åben. |
| Linux/Jetson | Kør `chloros-backend` i en separat terminal, eller lad en køre hele tiden med [backend-tjenesten](#running-the-backend-as-a-service-headless). |

Der kører kun én backend ad gangen; alle klienter (GUI&#x27;en, CLI&#x27;en, SDK-scripts) deler den. De centrale kommandoer (`process`, `login`, `status`, …) bruger den også og starter en, når der ikke kører nogen.

**Et backend fortsætter med at køre mellem kommandoer**, så den næste ikke skal betale for opstarten igen. Den stopper, når du logger ud (`chloros-cli logout`, medmindre Chloros-appen er åben, eller der er en kørsel eller en offline-build i gang), når du kører `chloros-cli backend stop`, eller af sig selv efter 30 minutter, hvis intet bruger den: ingen kommando, script eller app, ingen kørsel eller offline-build, intet tilsluttet kamera eller lyssensor. `backend stop` afviser, mens en kørsel eller en offline-build er i gang, og angiver navnet på den; `--force` stopper den nu, og den pågældende kørsel eller build går tabt (en burst bevares, og `array-build-video` bygger den igen). Indstil `CHLOROS_BACKEND_IDLE_MIN`, før backend&#x27;en starter, for at ændre de 30 minutter (`0`: aldrig); [backend-tjenesten](#running-the-backend-as-a-service-headless) stopper aldrig af sig selv. `chloros-cli backend status` viser, om der kører en, hvad der bruger den, og eventuelle offline-builds, der kører. `process` åbner sit projekt i den kørende backend, medmindre Chloros-appen er åben med et projekt, den har åbnet: i så fald afviser den det, og du skal enten fortsætte fra appen eller lukke det pågældende projekt først. Et projekt, som en tidligere `process` eller et SDK-script har efterladt åbent, erstattes blot.

### Bekræft installationen

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` afslutter kun `0`, når alle syv kontroller er bestået. Den sidste kræver en CUDA-GPU (en NVIDIA-GPU, eller en Jetson Orin) samt støjfjernelsesmodellerne, så på en computer uden en sådan vises `Failed: CUDA + Denoiser`, og programmet afsluttes med `1`; alt undtagen den GPU-afhængige teksturbevidste debayer fungerer stadig der.

> **Chloros+-abonnement kræves.**CLI kræver et aktivt Chloros+-abonnement.**Copper**er det indledende Chloros+-abonnementsniveau — alle betalte Chloros+-abonnementsniveauer har adgang til CLI/SDK; kun det gratis**Iron**-abonnementsniveau har ikke adgang. (Abonnements-id-oversigt: `0`=Iron/gratis, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Opgrader ved [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Denne begrænsning håndhæves af backend-systemet, ikke kun af CLI: En anmodning med SDK/CLI-flag uden et betalt abonnement afvises med fejlkoden `403 PLAN_UPGRADE_REQUIRED`, uanset om den kommer fra `chloros-cli`, Python-SDK&#x27;et eller en håndkodet HTTP-klient. En afbryder, der er logget ud, får i stedet fejlkoden `401 AUTH_REQUIRED`. Adgangen fungerer offline i abonnementets afdragsfri periode (30 dage for månedsabonnementer, indtil udløb for årsabonnementer) og ophører, når denne udløber; `chloros-cli status` fortsætter med at fungere, så årsagen er synlig (det er den eneste SDK-/CLI-rute, der er undtaget fra niveaubegrænsningen — `GET /api/license-status`).

---

## Hurtigstart på fem minutter

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (trin 2) starter en backend til sin egen kørsel, når der ikke kører nogen, og stopper den, når den er færdig, så start en til trin 4 og frem. Se [Start backend&#x27;en](#start-the-backend).

---

## `chloros-cli process`

Behandl en mappe med billeder gennem hele Chloros-pipeline (målregistrering → kalibrering → vignette → reflektans → indekseksport).

### Oversigt

```
chloros-cli process INPUT [OPTIONS]
```

### Positionsargumenter

| Argument | Beskrivelse |
| --- | --- |
| `INPUT` | Sti til indgangsmappe, der indeholder `.raw + .jpg`- (Survey3), `.tif/.tiff`- (LATTICE) eller `.dng`-filer. |

### Almindelige indstillinger

| Flag | Standard | Beskrivelse |
| --- | --- | --- |
| `-o, --output PATH` | en ny mappe med navnet `-n` i standardprojektmappen | Projektmappen. Projektet opkaldes efter den, og en mappe, der allerede indeholder en `project.json`, får en `_1` / `_2` i stedet for at blive overskrevet. Standardprojektmappen er den, som `set-project-folder` (eller GUI&#x27;en) har valgt, ellers `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | auto (tidsstempel) | Projektnavn, bruges kun **uden** `-o` (med `-o` får projektet mappens navn). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` bruger en Chloros+ neural debayer til Survey3-billeder; langsommere, men med højere kvalitet. Det er kun til Survey3: LATTICE-optagelser bruger altid standard-demosaic, så for en mappe, der indeholder LATTICE-optagelser, afviser `process` `texture-aware` efter importen med en fejlmeddelelse, der angiver, hvor mange der er, og afslutter `1`, før der behandles noget som helst (Chloros-appen tilbyder heller ikke Texture Aware til disse). Der kræves en CUDA-GPU (en NVIDIA-GPU på Windows/Linux x64, Orins GPU på en Jetson): uden en sådan bruger kørslen `standard`, og backend-loggen viser en linje med `[DEBAYER] … needs a CUDA GPU …`, der forklarer hvorfor (vist med `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Vignettekorrektion. |
| `--reflectance / --no-reflectance` | `--reflectance` | Reflektanskalibrering (bruger panelmål, hvis det findes, NIST-kalibrering pr. serienummer for LATTICE). For LATTICE-multispektral fungerer dette også som reflektans-**produktet** — se [Eksportindstillinger pr. produkt](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | fra | Anvend PPK GNSS-korrektioner fra sidecar-filer. |
| `--exposure-pin-1 CAMERA_MODEL` | ingen | Den kameramodel, der er forbundet til eksponeringspin 1, til PPK-tidssynkronisering. |
| `--exposure-pin-2 CAMERA_MODEL` | ingen | Den kameramodel, der er tilsluttet eksponeringspin 2, til PPK-tidssynkronisering. |
| `--recal-interval SECONDS` | `0` | Mindste rekalibreringsinterval, i sekunder (projektets *Mindste rekalibreringsinterval*). |
| `--timezone-offset HOURS` | `0` | Lyssensorens tidszoneforskydning i forhold til UTC, i timer (projektets *Lyssensorens tidszoneforskydning*). |
| `--format FORMAT` | `TIFF (16-bit)` | En af følgende: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros øger aldrig et billedes bitdybde: på billeder, der udelukkende består af 8-bit, skrives et format med større dybde som `TIFF (8-bit)`, og dette angives én gang i koden. I et projekt, der blander 8-bit- og 12/16-bit-optagelser, eksporteres hver optagelse med sin egen dybde eller lavere. Radiance er altid 32-bit float. |
| `--indices NAME [NAME ...]` | ingen | Vegetationsindekser efter forudindstillet navn (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), uden skelnen mellem store og små bogstaver. Hvert indeks tilføjes til projektet for dets kameras bånd, og der inkluderes et nyt projekt. Et navn, der ikke er en forudindstilling, eller som kameraets bånd ikke kan danne (`NDRE` kræver RE og NIR, hvilket intet enkelt kamera leverer), nævnes i en advarsel og får intet indeksbillede. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Tving indgangsstedet i pipelinen for LATTICE TIFF-filer (Survey3.raw påvirkes ikke). Desuden er der en nødudgang, der gør det muligt at behandle en optagelse **uden raw** overhovedet — se [Sådan ser en captures-mappe ud](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | aktiveret | Udsender det lineært debayered produkt (`Debayered_Images`). Se [Eksportknapper pr. produkt](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | aktiveret | Udsender skærmvisningsforhåndsvisningen (`Preview_Images`): RGB = hvidbalance (DAQ-lysart, hvis tilgængelig, ellers gråskala) + gamma; multispektral = falskfarvestrækning. |
| `--radiance / --no-radiance` | til | Udsender float32-radiance (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Reference for LATTICE-reflektansproduktet: `auto` = QA-godkendt mål inden for rammen er den absolutte reference (et mål med et QA-flag konkurrerer med DAQ om den målte score), DAQ-downwelling (ρ = π·L/E) som reserve; `target` = streng (ingen DAQ-erstatning); `daq` = DAQ-autoritativ. Se [Eksporttogler pr. produkt](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | ingen | Mappe med **målte** målreflektansscanninger pr. enhed (`<serial>.csv`); falder tilbage til de nominelle T3/T4P-spektre, hvis der mangler data. |
| `--array-alignment / --no-array-alignment` | aktiveret | LATTICE-arrays: anvend den modul-til-modul-justering, der er angivet i hver optagelses `Chloros:Alignment*` XMP, på alle behandlede produkter (debayered / preview / radiance / reflectance / index). Ingen handling for billeder uden disse tags. |
| `--array-alignment-crop / --no-array-alignment-crop` | beskæring | Beskær de justerede eksporter til arrayets fælles overlapningsområde, så alle moduler deler ét fodaftryk; `--no-…` bevarer det fulde sensorområde (sort udfyldning uden for kilden). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Resampling til justeringsforvrængningen. `nearest` bevarer de nøjagtige kilde-DN&#x27;er (ingen blanding af radiometriske værdier mellem pixels). |

### Indstillinger for måldetektion

| Flag | Beskrivelse |
| --- | --- |
| `--min-target-size PIXELS` | Mindste størrelse i pixels for kalibreringsmål for detektoren. |
| `--target-clustering 0-100` | Procentdel for målgruppering (standard `60`). |
| `--target / --targets` | Søg kun efter kalibreringsmål i en `target`- eller `targets`-undermappe i inputfilen (hurtigere). |

### Eksempler

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Eksportindstillinger pr. produkt (LATTICE multispektral)

LATTICE-behandlingen udvider sig til **alle relevante produkter i én gennemgang**. De fire indstillinger pr. type — `--debayered`, `--preview`, `--radiance`, `--reflectance` — er alle**aktiveret som standard**; brug formularen `--no-<type>` til at deaktivere en. RGB-masterkameraer udsender kun debayered + preview (ingen båndvis stråling/refleksion), så `--radiance`/`--reflectance` har ingen effekt for dem. Toggle-indstillingerne ignoreres for Survey3-`.raw` (som følger standardreflektans-/målstien). *(Det gamle `--radiometric-output {reflectance,radiance,sensor-response}`-flag blev **fjernet** og erstattet af disse indstillinger; der findes ikke længere et `sensor-response`-niveau.)*

| Produkt | Output | Kræves DAQ-downwelling? |
| --- | --- | --- |
| `--debayered` | Lineær demosaik (`Debayered_Images`): den samme fil, som en optagelse gemmer på sit `debayered`-niveau — kalibreringspakkens mørkeforskydning (DSNU) fra rådataene, derefter OpenCV&#x27;s kantbevidste demosaik; ingen linearitets-LUT, flat field eller unmix. | Nej. |
| `--preview` | Vis forhåndsvisning (`Preview_Images`): RGB = hvidbalance + gamma; multispektral = falskfarvestrækning. | Nej. |
| `--radiance` | float32 W/m²/sr/nm fra den fulde radiometriske kæde (`Radiance_Images`). | Nr. |
| `--reflectance` | uint16 reflektans ρ (`32768` = 1,0), Pix4D-klar. | **Ja**, medmindre et QA-godkendt mål inden for rammen forankrer den (se nedenfor). |

`--reflectance-source` vælger reflektansreferencen:**`auto`**(standard) gør et QA-godkendt mål inden for billedrammen til den**absolutte reference**— de målforankrede empiriske linjekæder krydsscores på udvalgte paneler, og den målte vinder anvendes, bånd for bånd: et bånd, som DAQ ikke kan kalibrere, eller hvis DAQ-hældningstilpasning mislykkes, tager målets egen linje (`ReflectanceElBandMethods` angiver hvilket) — et mål, der består QA med en markering (et panel udeladt, et bånd ude af drift), konkurrerer også med DAQ-skellet på den målte score, og en ramme, som det taber til, mærkes `ReflectanceChain=daq` — der falder tilbage til DAQ-downwelling-grænsen (ρ = π·L/E), når der ikke er noget mål til stede, eller QA mislykkes;**`target`**er streng (ingen DAQ-erstatning);**`daq`**fravælger til den DAQ-autoritative adfærd. Målgeometri (ArUco / fast ROI / strimmel) stammer fra projektets målkonfiguration; `--target-reflectance-dir DIR` opbevarer**målte** scanninger pr. enhed (`<serial>.csv`), der slås op ved hjælp af målenhedens serienummer/QR, med de nominelle T3/T4P-spektre som reserve.

DAQ-reflektansstien udleder automatisk den **tidsstempelmatchede nedadgående stråling**fra en registreret**`.daq`**(DAQ-U/M/E)**eller en DAQ-M-indbygget `.csv`**, der findes sammen med billedmaterialet. Hvis en kalibreringspakke pr. kamera eller DAQ ikke er gemt lokalt, henter pipelinen**det automatisk fra AWS** ved første brug (kræver internet én gang; gemmes i cachen under `~/.chloros/`).**Hvem tilhører målingen?**En `.daq`, som en optagelse gemmer, er målingen fra det eller de kameraer, den beskriver — navngivet i en `<file>.daq.cameras.json`-sidecar ved siden af den (`lattice capture`, `multi-capture`, `array-capture` og GUI’en; en rå serie angiver hvert medlems måleværdi i sin manifestfil) — og `process` matcher den**kun med billeder fra disse kameraer**. Et selvstændigt kamera uden egne måleværdier låner derfor ikke længere et andet kameras DAQ; bind en DAQ til det, eller optag en. En DAQ’s egen optagelse (`daq pool-record` eller en optagelse startet i GUI’en) eller enhver `.daq` uden den pågældende sidecar (herunder ældre optagelser) betjener stadig alle kameraer.

#### Aflæsning af reflektanspixels (Pix4D / Metashape / dine egne scripts)

Reflektansen gemmes som et heltal DN, og **den DN-værdi, der svarer til ρ = 1,0, afhænger af kildekameraet**:

| Kilde | ρ = 1,0 er | Sådan afgøres det |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margin op til ρ 2,0) | XMP-tagget `Chloros:PixelScale=32768` er indsat i filen. |
| Survey3 | `65535` (begrænset til ρ 1,0) | Ingen `Chloros:*` XMP-tags — denne fravær *er* selve signalet. |

**Læs `Chloros:PixelScale` og divider med det** i stedet for at antage en konstant. Tagget er defineret i uint16-domænet, så det forbliver `32768` på tværs af outputformater, der omskalerer — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` og `TIFF (32-bit, Percent)` er alle selvbeskrivende (normaliser først den gemte datatype tilbage til uint16: ×257 fra 8-bit, ×65535 fra float).

> **Filer fra ældre versioner kan ikke indeholde nogen skalering.** En 8-bit reflektans-TIFF-fil skaleres om og mærkes `Chloros:PixelScale` ligesom de andre. Versioner før 1.3.0 *beskærede* den i stedet til 0..255, så alle værdier over ρ≈0,008 blev udjævnet til 255, og de udelod både `Chloros:PixelScale` og tuplen `MicaSense:RadiometricCalibration`. **Hvis tagget mangler på en LATTICE-reflektansfil, skal du ikke antage en skala — genbehandl den** i stedet for at dividere pixels, der aldrig var delbare.

#### Læsning af indekspixler (NDVI og de øvrige vegetationsindekser)

**Et heltalindeksraster er en kontrastudvidelse pr. billede, ikke et −1…1-kort.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` og `JPG (8-bit)` indeksprodukter — herunder den 8-bit TIFF, som en 8-bit-optagelse gemmes som — gemmer

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

hvor `IndexMin` / `IndexMax` er **den pågældende rammes egne laveste og højeste endelige indeksværdier**. DN 0 er ikke −1, og fuld skala er ikke +1.

I Chloros-stempler begge endepunkter, så afbildningen forbliver inverterbar:

| Tag | Betydning |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — konventionen nedenfor |
| `Chloros:IndexMin` | den indeksværdi, der er gemt som DN 0 |
| `Chloros:IndexMax` | den indeksværdi, der er gemt som fuld skala |

Læs dem og vend dem om, idet du først normaliserer den gemte DN til 0…1 ved hjælp af den gemte datatypes maksimumsværdi, så ét udtryk dækker alle heltalformater:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

Indlæsning kræver ikke `-config pix4d.config` — exiftool læser disse XMP-egenskaber uden hjælp. Konfigurationen er kun nødvendig for at *skrive* dem.

> **`IndexMin == IndexMax`** betyder, at hver eneste pixel i det pågældende billede havde den samme indeksværdi. Rasteret er udfyldt midt på skalaen, og der er intet at dividere med — tag endepunktet.

> **Ingen `Chloros:Index*`-tags betyder, at der ikke er angivet nogen kortlægning.** En LUT-eksport er en farvelagt rendering, hvis pixels er gradientfarver, der er knyttet til dit LUT-tærskelvindue, og et billede uden pixels med endelig indeks har intet interval at angive. Antag ikke, at der er en kortlægning i nogen af tilfældene — den samme regel gælder som ved manglende `PixelScale`.**Da strækningen er pr. billede, kan to billeder fra samme flyvning ikke sammenlignes som DN**— hvert billede har sine egne endepunkter, så den samme NDVI-værdi ender på forskellige DN-værdier i forskellige billeder. Til kvantitativt arbejde på tværs af en flyvning (zonestatistikker, faste tærskelværdier, tidsserier) skal du eksportere**`TIFF (32-bit, Percent)`**: indeksrastere skrives der som float32**indeksværdier i sig selv**, uden behov for omskalering eller tags. På trods af formatets navn er indeksprodukter ikke ×100 — den skalering gælder kun for reflektans med heltalstypen.

#### Unmix-begrænsningsmaske (hvilke pixels et bånd blev begrænset til)

Den 3×3 spektrale unmix kan gøre et bånd negativt — hvor lyset knap nok indeholder det pågældende bånd, eller hvor scenen falder uden for det, som kameraets unmix-matrix beskriver. Det er ikke en reel værdi, så eksporten fastsætter den til 0 — og live-visningen viser den samme 0. Et bånd kan blive begrænset på nogle få pixels eller på næsten alle; masken registrerer begge dele. `Chloros:Band{R,G,B}UnmixClampFraction` angiver, hvor stor en del af et billede dette skete for; clamp-masken angiver **hvilke pixels**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— i en `clamp_masks/`-mappe**ved siden af** produktets mappe, aldrig inde i den (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; en eksport fra `process`: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — for hver strålings-, reflektans- og indeksprodukt, hvor mindst én pixel er begrænset: både optagelser og `process`-eksportfiler (samt TIFF-filerne pr. billede fra `array-build-video --save-tiffs`). Enkeltbånds `uint8`, på produktets eget pixelgitter (inklusive orientering, ROI, binning og array-justering). Hver maske angiver sit produkts kamera (`Make` / `Model` / serienummer), optagelsestidspunkt og objektiv-tags samt en liste over de bånd, som dens bits repræsenterer (`XMP-Camera:BandName`, bit 0 først); dens `ImageDescription` og `Software` markerer den som en clamp-maske, og den indeholder ingen kalibrering.
- **Bits**: bit 0 = båndet i produktets R-kanal (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). En sat bit betyder, at den pågældende pixels værdi i det pågældende bånd blev begrænset til 0 — betragt den og ethvert indeks, der er opbygget ud fra den, som lavt pålideligt.
- **`Chloros:UnmixClampMask`** er stemplet på hvert produkt, der har kørt unmix: maskenes sti i forhold til produktets egen mappe (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, altid adskilt med `/` — tilføj den til billedets mappe for at åbne masken), eller `none`, når ingen pixel er begrænset. Ingen mærkning betyder, at produktet aldrig har gennemgået unmix-processen (kameraer med RGB- eller monokromoptagelse, rå/debayered niveauer).
- `lattice capture` / `array-capture` udskriver maskenes sti under den gemte fil (`clamp mask: …`), og i listen over optagelsesresultater angives den som billedets `clamp_mask`.
- `lattice index --live --save-multiband` skriver én maske pr. kildekamera til multibåndsstakken — se [`lattice index` — multibåndsstakmasker](#lattice-index--multiband-stack-masks). En video har ingen maske: `array-build-video` rapporterer i stedet hver rammes clamp-fraktioner (se [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Da ingen produktmappe indeholder en maske, uanset dybde, kan en produktmappe (`radiance/`, `reflectance/`, en `process`-eksports `*_Images/`-mappe) overføres til Pix4D, Metashape eller anden fotogrammetri-software, som den er — inklusive undermapper — og den accepterer kun billeder. Overfør produktmappen, ikke mappen ovenover (den indeholder også `clamp_masks/` og alle andre eksporttyper). Chloros’ egen import springer maskerne over. Fortolkning af en maske: `(mask & 1) != 0` betyder »R-kanalbåndet blev begrænset her«, `& 2` G, `& 4` B.

#### EXIF overføres til eksporten

`process` kopierer kildeoptagelsens **GPS-blok og dens ExifIFD** til hvert produkt, så en
eksport indeholder `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` og
`CameraSerialNumber` sammen med georeferencen.

**`FocalLength` er ikke valgfrit til fotogrammetri.** Pix4D beregner ground sample distance ud fra
brændvidde plus højde; hvis tagget mangler, falder programmet tilbage til en helt forkert skala. På en
flyvning over en appelsinplantage med 49 optagelser forvandlede det manglende tag et område på 411 m × 160 m til et rekonstrueret
47,8 km × 13 km — et 455 MP ortofoto bestående hovedsageligt af nodata, hvilket derefter blev tolket som et fliseproblem og
et BigTIFF-problem, før nogen tjekkede GSD&#x27;en. Hvis dit ortofoto ender med en usandsynlig
skala, skal du først køre `exiftool -FocalLength` på det eksporterede produkt.

Kopien er bevidst **ikke** `-all:all`: IFD0&#x27;s strukturtags ødelægger LATTICE-outputtet, når
de kopieres, og `ExifImageWidth` / `ExifImageHeight` er udelukket, fordi de beskriver
*kilde*-optagelsen — en eksport, der nogensinde er blevet ændret i størrelse, ville ellers have dimensioner
, der er i modstrid med dets eget raster. XMP skrives direkte i stedet for at blive kopieret, fordi ExifTool
kasserer XMP-tags fra samme opkald, når XMP-blokken kopieres (hvilket ville fjerne MAPIR-
kalibrerings-tags).

### Hvor outputfilerne gemmes

Produkterne skrives **i projektmappen, grupperet efter kamera og derefter efter filformat**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Kameramappen er `LATT-<sensor>-<lens>-F<filter>` for LATTICE (svarende til optagelsens EXIF
`Model`) og `<model>_<filter>` for Survey3 — to kameraer, der deler sensor og filter, men adskiller sig
med hensyn til objektiv, har hver deres mappe, da vignettering, synsfelt og forvrængning varierer. Formatmappen
mappe følger `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, eller `tiff32` for
`TIFF (32-bit, Percent)`.

> **Hvert eksporteret produkt beholder navnet på KILDE-filen.** En radiance-eksport af
> `capture_…_raw.tif` hedder stadig `capture_…_raw.tif` — den ligger blot i
> `tiff32/Radiance_Images/`. **Mappen identificerer produktet, ikke filnavnet**, så en glob-søgning
> efter `*radiance*.tif` finder intet; søg i stedet efter mappen.

### Lyssensoroptagelser — kalibreret `.daq` + `.csv`

`process` håndterer også `.daq`-optagelserne i din indgangsmappe, og den **behøver ikke**
bruger nogen billeder til at gøre det: en DAQ-U / DAQ-M / DAQ-E, der er fløjet alene, udgør en komplet
optagelse, og en mappe, der kun indeholder `.daq`-filer, er en gyldig indgang.

En DAQ kan optages **uden** kalibrering — det er netop, hvad de offentlige
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) optagere
(`record_daq.py`) gør som standard: de skriver rå sensortællinger og stempler filen, så
Chloros henter den pågældende sensors fabrikskalibrering **via seriel** (først fra den lokale cache,
derefter fra MAPIR-skyen) og anvender den. `process` skriver resultatet tilbage:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` indeholder én række pr. måling: UTC-tidsstempel, integrationstid, total effekt,
fotopisk/skotopisk lux, PPFD (og dens opdeling i blå/grøn/rød), spidsbølgelængde, derefter
fulde spektrum på sensorens eget bølgelængdegitter. `.daq` importeres igen uden at blive
kalibreret en anden gang.

`<name>` er optagelsens eget filnavn. To optagelser i forskellige mapper kan dele
én — måleværdier pr. optagelse (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`), der er gemt i
samme millisekund af to kameraer — og hver får stadig sine egne produkter: den første beholder
`<name>`, den næste tilføjer de mapper, der adskiller den, **efter** navnet
(`<name>__<capture>_daq_calibrated.daq`), så optagelsens dato og klokkeslæt stadig står først.

Ved succes rapporterer kørslen `Light-sensor products written: N (calibrated .daq + .csv)`.
Det, der står i parentes, beskriver, hvad der faktisk blev skrevet, så det lyder
`(RAW COUNTS — this sensor has no calibration bundle)` for en sensor uden bundt og
`(N calibrated, M raw counts)` for en mappe, der indeholder begge dele. Backendens egne
overskrifter `[DAQ-EXPORT]` og `[RUN-SUMMARY]` er formuleret på samme måde — ingen af
de tre kan betegne en rå eksport som kalibreret.

En DAQ-U / DAQ-M / DAQ-E-optagelse, hvis kalibreringspakke ikke kan hentes — du er
offline, eller den pågældende sensor har ingen kalibrering gemt — **sprunget over med en begrundelse** på en
`[DAQ-EXPORT]`-linje og skrives aldrig ud som en »kalibreret« fil, der indeholder rå tællinger.
Opret forbindelse til internettet og kør kørslen igen. Årsagen er den, som læseren faktisk
fastslog for den pågældende fil (ulæseligt skema, intet bundt, en skrivefejl), og kørselsoversigten
angiver **separate** årsager — tyve filer, der er sprunget over på grund af én årsag, vises som én
årsag, ikke tyve gentagelser af den. CLI&#x27;en navngiver hver oversprungen optagelse i slutningen af
kørslen sammen med årsagen:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

En mappe med optagelser, hvor **ingen** af dem eksporteres, mislykkes: se
[En kørsel, der ikke skriver billeder, mislykkes](#a-run-that-writes-no-images-fails).

En optagelse, hvis navn ender på `_calibrated` eller `_raw` (`field_raw.daq`), er **ikke
importeret**: det er de navne, som `process` giver sine egne produkter, og et produkt, der læses tilbage
som en optagelse, vil blive talt med to gange. CLI&#x27;en angiver dette og navngiver filen —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— i stedet for at rapportere en tom mappe. Omdøb en sådan optagelse, før den behandles.

#### DAQ-A-optagelser eksporteres som rå tællinger

**DAQ-A** -familien er ældre end systemet med bundter pr. serienummer og har intet kalibreringsbundt
at hente — den kalibreres i stedet i felten mod et reflektansmål, hvilket er
grunden til, at den aldrig har haft brug for et. Hvis man afviste disse optagelser, havde man ingen mulighed for overhovedet at få
tallene ud, så de eksporteres under et **andet navn**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Et andet filnavn i stedet for en markering inde i filen, fordi oplysningerne skal kunne overleve
at blive videresendt via e-mail som et rent navn. `.csv`-overskriften angiver
`raw spectral sensor counts (NOT irradiance)` og advarer om, at værdierne er sammenlignelige
**inden for** filen — hvilket er præcis det, målbaseret kalibrering bruger dem til — og
ikke på tværs af sensorer. De effekt-afhængige fotometriske kolonner (total effekt, fotopisk og
skotopisk lux, PPFD) skrives som **NULL** i stedet for at blive integreret ud fra tællinger, og kørselsoversigten
angiver `RAW COUNTS`, så &quot;eksporteret&quot; i en logfil kan ikke læses som irradians.

Ældre **v1.01 / v1.02**-optagelser (en DAQ-A-SD skriver disse) indeholder ingen epoke pr. aflæsning,
kun filens skrivetid. Billed↔nedstrålings-matcheren afviser dem stadig — at matche et
billede mod et skrivetidspunkt ville være usynligt forkert — men eksportøren læser dem, og
CSV-filen udskriver `clock=daq_created_on`, så produktet angiver, hvilket ur det er på.

### Bemærkninger

- `process` registrerer automatisk, om din mappe er af typen Survey3, LATTICE eller blandet.
- Statusopdateringer sendes via Server-Sent Events; CLI&#x27;en viser live status pr. tråd (Detektering, Analyse, Behandling, Eksport).
- På Linux/Jetson kontrollerer CLI-en swap-pladsen og kan vise en advarsel, før store mapper behandles. Den teksturbevidste debayer anvender også automatisk en GPU-frekvensbegrænsning på Jetson-enheder med lavt strømforbrug (Nano, Orin Nano).
- Ved succes rapporterer kørslen, hvor mange billedprodukter den har skrevet (`Image products written: N`, med `(plus M clamp masks)`, når unmix-funktionen har begrænset en pixel).
- **`process` deler et kørende backend på en sikker måde.** Det genstarter automatisk et allerede kørende backend, men kun hvis dette backend er inaktivt med et tidligere projekt indlæst og ikke har nogen kameraer, arrays eller DAQ&#x27;er. Et backend, der indeholder noget af dette (GUI’en, der streamer et array, et SDK-script, en DAQ-optagelse), bruges som det er og genstartes aldrig, så `process` kan køre sideløbende med et live-array. Hvis backendet rapporterer, at der er et kørende forløb, venter `process` op til 15 sekunder på, at den afsluttes (en kørsel, der netop er afsluttet, tager ca. et sekund at rydde), hvorefter den afviser med `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. Det globale flag `--restart` tvinger stadig en ny backend og afslutter enhver kamera-, array- eller DAQ-session på den. En backend, der kører som en Linux-systemd-tjeneste, genstartes aldrig: `process` bruger den, og `--restart` udskriver i stedet `sudo systemctl restart <unit>` (se [Kørsel af backend som en tjeneste](#running-the-backend-as-a-service-headless)). Det samme gælder Chloros-appens eget backend: når appen er åben, bruger `process` det og indlæser sit projekt i stedet for appens (et backend, der startes i stedet, ville spærre den åbne app ude).
- **En igangværende kørsel beholder sit projekt.** Mens en behandlingskørsel er i gang — startet af appen, `process`, et SDK-script eller `chloros-mcp` — nægter backendet at oprette, åbne, importere til, tilføje eller fjerne filer fra, flytte, slette eller ændre et projekts behandlingsindstillinger, med HTTP-kode 409 `PROCESSING_IN_PROGRESS` og `A processing run is in progress on project '<name>', so ... now is refused`; et `process`, der modtager denne fejl, afslutter `1`. Visningsindstillinger (miniaturestørrelse) gemmes stadig. Behandlingsmodus og projektets indstillinger for målrefleksion afvises på samme måde (en kørsel beholder den modus, den startede i). En optagelse, der gemmes i projektet midt i et kørsel (fra appen, CLI, SDK eller `chloros-mcp`), gemmes som normalt, men føjes først til projektets filliste, når kørslen slutter; hvilket fremgår af svaret (`project_add`), og det samme gælder en lyssensoroptagelse, der stoppes i projektet. Tidligere kunne en anden klient skifte til det åbne projekt midt i et kørsel, og det igangværende kørsel blev eksporteret til det andet projekt.
- **Ctrl+C stopper kørslen i backend, ikke kun i CLI&#x27;en.** CLI&#x27;en beder backend&#x27;en om at stoppe behandlingen, viser `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` og venter op til 120 sekunder på `Processing stopped.`, så `chloros-cli process … && next` ikke starter `next` på et backend, der stadig er optaget af kørslen. Et andet tryk på Ctrl+C afslutter ventetiden. `process` afslutter derefter `130`, som enhver afbrudt kommando gør.

#### En kørsel, der ikke skriver nogen billeder, mislykkes

Hvis du anmodede om produkter, og kørslen skrev **ingen** — kun `project.json` og
`calibration_data.json` — behandler `process` dette som en fejl: det udskriver
`Processing finished but wrote no image products.` og **afsluttes med en værdi forskellig fra nul**, så et script kan
opdage det. Meddelelsen angiver projektmappen og de sædvanlige årsager:

- indlæsningsmappen blev ikke genkendt som en optagelse (tjek layoutet og `--input-level`), eller
- alle anmodede produkter blev sprunget over, da de ikke var relevante for disse kameraer (f.eks. anmodning om
  radiance/reflektans fra kameraer, der kun har RGB).

Kør programmet igen med `--verbose`, og tjek backend-loggen for linjer med `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
som forklarer de kamera-specifikke spring, der ellers ikke vises i CLI-udskriften.

En bevidst kørsel, der udelukkende omfatter metadata — alle produktindstillinger slået fra og ingen `--indices` — er stadig en
**succes**, fordi en tom billedudskrift er det korrekte resultat i dette tilfælde.

#### En kørsel, hvor eksportfilerne ikke kunne verificeres, mislykkes

Efter en kørsel kontrollerer `process` de eksporterede filers tags med ExifTool. Hvis ExifTool ikke kunne køres,
udskriver den `Eksportverifikation MISLYKKET: ExifTool kunne ikke køres, så tagsene for N ud af M eksporterede filer
ikke blev kontrolleret (se [EXIFTOOL]-linjerne i backend-loggen)` and `Produkterne er skrevet og forbliver
på deres plads; kør kørslen igen for at verificere dem.`, does not print `Behandlingen er afsluttet!`, og **afsluttes med en værdi forskellig fra nul**.
Produkterne bevares.

#### Hvilke optagelser en kørsel omfatter

Optagelser, der er færdiggjort, før `process` køres, medtages. Optagelser, der er gemt under kørslen, medtages ikke; `process`
advarer om, hvor mange der blev udeladt, og hvis programmet køres igen, medtages de.

#### En kørsel, hvor et billede gik tabt, mislykkes

Hvis et billede mislykkedes under et kørsel, udskriver `process` `N image(s) failed and were not exported` med den
mest almindelige årsag, udskriver ikke `Processing complete!` og **afsluttes med en værdi forskellig fra nul**. De øvrige resultater
skrives til fil og forbliver på deres plads, og backend-loggen indeholder hver enkelt fejl.

Det samme gælder en **kørsel, der kun omfatter lyssensoren**: en mappe med `.daq`-optagelser indeholder per definition ingen billeder, der kan eksporteres,
og kørslen bedømmes i stedet ud fra de kalibrerede `.daq` / `.csv`, den skrev.
Hvis den ikke skrev **noget** — alle optagelser blev sprunget over — udskriver den
`Processing finished but wrote no light-sensor products.`, navngiver hver optagelse med
årsagen til, at den blev sprunget over, og **afslutter med en værdi forskellig fra nul**.

---

## `chloros-cli login`

Godkend denne maskine med en Chloros+-cloudkonto. Adgangsoplysningerne gemmes sikkert i `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Eksempler

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (ved at fjerne eller duplikere dele af adgangskoden). Ved en 401-fejl forsøger CLI&#x27;en automatisk igen med `$$`, hvor-tilføjet, derefter med en dedupliceret halvdel af adgangskoden; hvis et nyt forsøg lykkes, logger den dig ind og viser den korrekte syntaks med enkelt anførselstegn, som skal bruges næste gang.

> **Brug uden grafisk brugergrænseflade/via script: ingen cachelagret session betyder en login-prompt.** En kernekommando (`process`, `status`, `export-status`, `time-sync`, …) udført uden en cachelagret session beder om `Email:` / `Password:` på stdin, før den fortsætter. Hvis stdin er lukket (cron, en systemd-enhed), mislykkes det straks med fejlkoden `login cancelled`; hvis stdin er en åben terminal eller et rør, venter det på indtastning. Kør `chloros-cli login EMAIL PASSWORD` én gang pr. maskine, før du planlægger headless-opgaver.

---

## `chloros-cli logout`

Rydder den cachelagrede session og tvinger et nyt login ved det næste opkald.

At logge ud, mens en behandlingskørsel er i gang, stopper den ikke: kørslen afsluttes i den behandlingsmodus, den startede i, og den næste kørsel bruger standardbehandling. `logout` siger det (`The processing run in progress finishes in <mode> mode; the next run uses standard`), og det samme gør appen, når du logger der.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Viser det aktuelle licensniveau (Iron/Copper/Bronze/Silver/Gold), den godkendte bruger og antallet af enhedsbindinger.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Hent status for den aktuelle Thread-4-eksport. Kan sikkert kaldes **under** en `process`-kørsel fra en anden shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Indstil CLI&#x27;ens visningssprog (38 understøttede sprog, herunder CJK, RTL og indiske sprog). Skifter automatisk til engelsk på ældre konsoller, der ikke kan gengive scriptet.

```
chloros-cli language [LANG_CODE] [--list]
```

### Eksempler

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Kommandoer til projektmapper

Disse styrer standardplaceringen for projektmappen (delt med GUI&#x27;en).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` viser den mappe, som nye projekter placeres i. Hvis den konfigurerede mappe ikke længere findes (den er blevet slettet, omdøbt eller ligger på et drev, der ikke er tilsluttet), falder CLI&#x27;en tilbage til standardmappen, ligesom GUI&#x27;en gør: standardmappen vises på stdout, en advarsel med navnet på den manglende mappe sendes til stderr, og afslutningskoden er `0`.

---

## `chloros-cli update`

Kun Linux/Jetson. Kontrollerer `version_url` fra `/etc/chloros/update.conf`, og hvis der findes en nyere version, hentes den relevante `.deb` og installeres straks med `sudo dpkg -i` (ingen bekræftelsesmeddelelse; `sudo` kan bede om din adgangskode). På en Jetson tilbydes opdateringen kun til JetPack 6-systemer. På Windows gør kommandoen intet og rapporterer `up to date`: opdater med et nyt installationsprogram.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

På Linux/Jetson udfører CLI&#x27;en også en **automatisk opdateringskontrol ved hver opstart** (ikke-blokerende, forsinker aldrig kommandoen): den læser `/etc/chloros/update.conf`, gemmer resultatet i cachen i 1 time i `~/.chloros/update_cache.json` og viser `Update available: vX.Y.Z / Run: chloros-cli update`, når der findes en nyere version. Springes lydløst over ved eventuelle fejl og på Windows; `CHLOROS_DISABLE_UPDATE_CHECK=1` deaktiverer funktionen. På grund af den 1-timers cache kan en udgivelse, der offentliggøres inden for den time, stadig vises som opdateret.

---

## `chloros-cli selftest`

Kører en 7-trins smogtest: version, porttilgængelighed, opstart af backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), tilstedeværelse af denoiser-model, klarhed for CUDA+støjfjerner. Den bruger en kørende backend eller starter en til testen og stopper den bagefter. Den udskriver `N/7 checks passed` og de mislykkede kontroller og afslutter kun `0`, når alle syv er bestået; CUDA+støjfjernerkontrollen mislykkes på en computer uden en CUDA-GPU (se [Bekræft installationen](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP-grandmaster-status og -styring. Chloros-værten kører PTP-grandmasteren; LATTICE-kameraer og DAQ-E-enheder fungerer som slaver til den for at sikre tidsstempler på tværs af enheder (ca. 1 ms på tværs af enheder). PTP synkroniserer **tidsstempler**: kameraerne i et array eksponerer samtidigt, fordi M8-hardwareudløseren aktiverer dem (under 100 µs mellem modulerne), ikke på grund af PTP. `chloros-cli time-sync` uden underkommando kører `status`.

| Underkommando | Beskrivelse |
| --- | --- |
| `status` | Vis grandmaster-tilstand, BMCA-prioriteter, uridentitet og om dens pakker når ud til netværket: `health` er `OK` (`STOPPED`, når grandmasteren ikke kører), eller `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` med en `health_detail`, og derefter udskrives en linje med `WARNING:` til stderr (stdout forbliver JSON). `sync_sent` / `announce_sent` tæller kun de meddelelser, der blev sendt; `*_send_errors` og `last_send_error` tæller dem, der ikke blev sendt. Når en kamerforbindelse afbrydes og genoprettes, genopbygger grandmasteren selv sine sockets (`rebinds`, `last_rebind`). |
| `peers` | Liste over slaver, der ses via Delay_Req (kameraer + DAQ-E-sensorer). |
| `cameras` | PTP-tilstand pr. kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Genstart grandmaster-processen. |
| `set-priority --priority1 N --priority2 N` | Tilsidefør BMCA-prioriteter. |

Grandmasteren sender meddelelser via kameraernes netværkskort (enhver adapter med en `169.254.x.x`-adresse inkluderet). Hvis ingen er aktive, falder den tilbage til adapteren for standardruten, men kun hvis denne er kablet – aldrig Wi-Fi eller en virtuel adapter (Tailscale, WSL), da intet kamera eller DAQ-E, der sidder bag et sådant. Når der ikke er noget at annoncere, kører den uden sockets (`health` `NO_SOCKETS`), og den næste kamera- eller DAQ-forbindelse genopbygger den på det netværkskort, der er aktivt på det tidspunkt; `time-sync restart` gør det med det samme.

Grandmaster kører i sin egen proces ved siden af backend (`own_process` / `process_pid` i `time-sync status`), med høj prioritet i Windows (`process_priority`), så et travlt backend (en behandlingskørsel, en eksport, flere kommandoer på én gang) ikke kan forsinke de tidspunkter, den stempler; før kunne en sådan forsinkelse flytte alle kameraers ur på én gang. Den proces slutter med backend’en; hvis den ikke kan startes, kører grandmasteren inde i backend’en som før (`own_process` false, og det fremgår af backend-loggen).

### Eksempler

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE-kamerastyring. Hver underkommando dirigeres gennem Chloros-backend; backend ejer kamerapoolen, så efterfølgende CLI-kald genbruger det samme åbne håndtag.

### Almindelige indstillinger (fælles for de fleste underkommandoer)

| Flag | Beskrivelse |
| --- | --- |
| `-d, --device N` | Kameraindeks (standard: 0). |
| `-s, --serial SN` | Specifik serienummer; tilsidesætter `--device`. |
| `--serials SN1,SN2,…` | Serienumre adskilt med komma til drift med flere kameraer. De [midlertidige kommandoer til flere kameraer](#multi-camera-transient-sessions) åbner kun disse kameraer, hvor det første udløser triggeren. |
| `--all` | Udføres på alle fundne kameraer. |
| `--exposure US` | Eksponeringstid i mikrosekunder. |
| `--gain DB` | Forstærkning i dB. |
| `--pixel-format FMT` | f.eks. `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` på en M3M. Navnet angiver bitdybden inden for kameraets egen familie: på en M3M fastlægger et Bayer-navn mono-formatet for dens dybde (`BayerRG8` fastlægger `Mono8`), på en M3C fastlægger et mono-navn det Bayer-navn (`Mono12` indstiller `BayerRG12`). |
| `--width N` / `--height N` | Billeddimensioner. |
| `--preset {default,high_quality,high_speed,triggered}` | Anvend en forudindstilling: dens pixelformat, eksponering, forstærkning og billedhastighed (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` viser dem). På en M3M er formatet det samme Mono-format med samme dybde: `high_speed` indfanger `Mono8`, `high_quality` og `triggered` `Mono12`. Et flag, der følger med (`--exposure`, `--gain`, `--pixel-format`) har forrang for forudindstillingen, og en indstilling, som forudindstillingen ikke nævner, bevarer kameraets egen. Alle frieundtagen `triggered`, som aktiverer kameraet ved en hardwareflank på linje 2 — hvis der ikke er noget, der driver denne linje, vil det vente i al evighed i stedet for at optage. |
| `-o, --output DIR` | Output-mappe (standard: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP-pakkestørrelse. `auto` kører ICMP+GVSP-prober; `jumbo` = 9000; `standard` = 1500. |

### LATTICE-kameraets arbejdsgang ved første forbindelse

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Reference til underkommandoer

#### Opdagelse og information

| Underkommando | Formål |
| --- | --- |
| `lattice info` | Vis en liste over tilsluttede kameraer (producent, model, serienummer, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analyser værtsystemet for optimal kamerakonfiguration. `--no-discover` springer kamerasøgningen over (hurtigere, analyse baseret udelukkende på netværkskortet). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Kontroller/ret NIC-indstillinger (på Linux også kernens socket-modtagelsesbuffere — se [Linux-værter](#linux-hosts--socket-receive-buffers)); estimer båndbredde/FPS. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Stable-schema-backend-netværk-kapacitet + anbefalinger til array (returnerer `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` er som standard den fulde sensor, `2048` × `1536`. Den afsluttes med `0` for `ok`, `auto_capped_fps` og `auto_shrunk`, og `1` for `needs_force_slip` og `error`, med eller uden `--json`. Den når backend ved `--backend-url` (standard `http://127.0.0.1:5000`), ikke `CHLOROS_BACKEND_URL`. `auto_capped_fps` bevarer den anmodede opløsning, men sætter et loft over mål-fps — læs `recommended.recommended_target_fps` og send det som forbindelsesmål; betragt det som en succes, ikke en fejl. `auto_shrunk` vises kun, når den anmodede ramme ikke kan leveres ved nogen billedhastighed; connect formindsker aldrig rammer på egen hånd. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Hvad-hvis-analyse uden at åbne kameraer (`--width` / `--height` standard `2048` × `1536`). Både dette og `network-analysis` viser, hvad connect vil streame: uden `--pixel-format`, gælder de formater pr. kamera, som connect anvender (Mono12 / BayerRG12), og med `--packed auto` (standard) den [pakket 12-bit](#packed-12-bit-transport---packed) — så den angivne fps er den hastighed, som arrayet kører med. Begge udskriver pakningsvalget. **`--n-active` er det samlede antal kameraer på kablet, ikke kun dette arrays**— forhøj det, når enkeltstående kameraer streamer samtidigt, eller når ledningskapaciteten beregnes ud fra et behov, der undervurderer antallet af kameraer (standard: `len(--models)`). Viser altid de samlede linjer `Wire budget:` (krævet MB/s vs. kollisionssikkert loft) og `Max cameras:` og markerer `** OVER-SUBSCRIBED**`, når arrayet overbelaster kablet — se [Array fps &amp; burst-model](#array-fps--burst-model). |
| `lattice gpu` | Vis GPU-status. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Kontroller eller opdater kameraets firmware. Det lokale `.fwa`-valg er fastlåst: filen i `firmware/<MODEL_PREFIX>/`, der matcher buildets `MIN_FIRMWARE_VERSION`, flashes, når den er til stede (kun den højeste version som reserve), så et nyere leverandørbillede, der er klargjort på disken, er inaktivt, indtil den pågældende fastlåsning ophæves — bevidst sendes nyere udgivelser til enhederne via det signerede AWS-manifest, hvilket foretrækkes, når der er nyere versioner. |
| `lattice presets [--apply NAME]` | Vis eller anvend kamerapræindstillinger. `--apply` angiver, hvad præindstillingerne hedder (deres pixelformat, eksponering, forstærkning, billedhastighed, udløser), og udskriver disse indstillinger, som kameraet læser dem efter: på en M3M, forudindstillingens dybde i Mono-format; den billedhastighed, den kører med, som kan være lavere end forudindstillingen angiver (`high_speed` angiver 60 fps; et full-frame-kamera på en 1 Gbps-forbindelse kører ca. 35). Alle andre indstillinger, som dets open-kommando skrev, genindstilles til de fundne værdier. `--apply default` nulstiller kameraet til fabriksindstillingerne: fabriksindstillingerne med fritløbende automatisk eksponering på Line2, grænsen for forbindelsens båndbredde og fejlkorrektion slået fra, samt et RGB-kameras kalibrerede hvidbalance bevaret (enhed på alle andre). Intet gemmes i kameraets hukommelse, så ved næste opstart starter det fra sin egen hukommelse som før. Som en del af en optagelse eller forbindelse er `--preset` og `default` ikke forudindstillinger. |
| `lattice status` | Live-kamerastatus. |

#### Optagelse

| Underkommando | Formål |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Enkeltbillede. **Gemmer som standard alle eksporttyper**(`--processing all`); se [Optagelseseksportniveauer](#capture-export-levels-the-all-default). `--processing` accepterer `all`, `raw`, `debayered`, `radiance`, `reflectance` eller `display` (`preview` accepteres for `display`); ethvert andet navn afvises, før kameraet berøres (afslut `1`). `--levels` gemmer en eksplicit delmængde (overskriver `--processing`). Et kamera, som Chloros ikke besidder, åbnes til optagelsen, og dets eksponering indstilles først ud fra scenen (højst et par sekunder), ligesom [midlertidige kommandoer](#multi-camera-transient-sessions) gør; når optagelsen afsluttes, uanset hvordan den afsluttes, sættes det pågældende kamera tilbage til den tilstand, det var i (format, eksponering, farvepipeline, alle indstillinger, som optagelsen ændrede — listen under [Multi-Camera (Transient Sessions)](#multi-camera-transient-sessions)). Den DAQ, der læser den anvendte optagelse, gemmes som en `.daq`-sidecar, der indeholder et rådata-uddrag; `--no-daq` / `--daq-tolerance-ms` justerer den ([Den DAQ, der læser en optagelse, gemmer](#the-daq-reading-a-capture-saves)). `--force-daq` accepteres stadig. `--jpeg-quality` = JPEG-kvalitet 1–100 (standard 95).**Filer, der gemmes i det åbne projekt, føjes til det.** Ligesom i appen føjer en optagelse, hvis filer havner i mappen til det projekt, der er åbent i Chloros, dem til det pågældende projekts filliste; filer, der gemmes andetsteds, forbliver uberørte. Kommandoen udskriver en linje, der angiver dette (`Project: ...`), eller, mens en behandlingskørsel er i gang, at de holdes tilbage og tilføjes, når kørslen slutter. Det samme gælder `multi-capture`, `array-capture` (`--smart` ligeledes) og `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Streamer til disk indtil Ctrl+C. En `tiff`-kørsel&#x27;s billeder udgør filen `raw`, som en array-capture skriver fra det pågældende kamera: sensorens mosaik, ét plan med sine egne værdier, ukomprimeret, med kalibreringspakken (et 2048 x 1536 12-bit-billede er 6,0 MiB plus pakken, uanset motivet), så `process` tager udgangspunkt i dem. `png` og `jpg` indeholder 8 bit, så disse serier er demosaicerede billeder til visning, hvilket `process` ikke tager udgangspunkt i. Med en DAQ-bundet (`daq pool-bind`) eller samlet, får hvert billede sit eget `.daq` og `.match.json` i seriens `daq/`-mappe, da `lattice capture` gemmer én; `--no-daq` deaktiverer dette, og ved stop vises antallet. Hvert billede skrives én gang med alle metadata; ved Ctrl+C skrives de billeder, der allerede er hentet, før kameraet frigives. Stopper af sig selv, når drevet er fyldt, udskriver `Failed:` / `Disk full:` og afslutter `1`; ethvert billede, der ikke kunne skrives, afslutter også `1`. Et kamera, der er sat til at være klar til en hardwareudløser (`TriggerMode` `On`) indstilles til fritløb for den pågældende optagelse. Når optagelsen slutter, sættes kameraet tilbage til den tilstand, det blev fundet i — dets udløserindstillinger og alle indstillinger, som optagelsen ændrede (eksponering, forstærkning, format, …) — medmindre der er tale om `--keep-settings`, som gemmer optagelsens indstillinger i kameraets egen hukommelse (frit kørende), eller `--clear-settings`, som nulstiller denne hukommelse til fabriksindstillingerne. En kørsel, der ikke optager nogen ramme, betragtes som en fejl: hvis der ikke er nogen ramme i 10 sekunder, stopper den af sig selv, udskriver `No frames:` med årsagen og afslutter med `1`. Billeder, som skriverne ikke kunne følge med, tælles som tabte, og afbrydelsen udskriver `Dropped:` med årsagen (kameraets billedhastighed, skrivernes og køen); en lavere billedhastighed eller et 8-bit pixelformat bevarer dem alle. Kommandoen tjekker ind hos backend&#x27;en hvert 2. sekund; hvis den går ned uden at stoppe optagelsen (`kill -9`, ikke nok hukommelse), stopper backend&#x27;en den 30 sekunder efter den sidste indtjekning og sætter kameraet tilbage (`stop_reason` `client_gone`, samt en linje i backend-loggen). `multi-stream` og `align-stream` gør det samme. En start, hvor kameraet tager længere tid at åbne, end kommandoen venter (60 sekunder; 180 sekunder for `multi-stream` / `align-stream`) — et kamera, der stadig starter op — annulleres af backendet: kameraet sættes tilbage og lukkes, og ingen optagelse fortsætter (504 HTTP-fejl `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Browserbaseret live MJPEG-forhåndsvisning: opretter forbindelse til kameraet, viser stream-URL&#x27;en og åbner den i din browser. `--ae-damping` indstiller dæmpning af automatisk eksponering (0,4–100). |

##### Livestreams (MJPEG; ingen RTSP)

Chloros leverer livevideo som **MJPEG via HTTP**(`multipart/x-mixed-replace`, grænseværdi `frame`) fra backend.**RTSP understøttes ikke.** Åbn en URL-adresse i en browser, eller læs den med en hvilken som helst MJPEG-klient (OpenCV&#x27;s `cv2.VideoCapture(url)`, ffmpeg, VLC). Kameraet skal først tilsluttes i backend&#x27;en (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` eller via GUI&#x27;en).

| URL (`http://127.0.0.1:5000` + …) | Stream |
| --- | --- |
| `/api/camera/stream/<serial>` | Visningsforhåndsvisning, op til ca. 30 fps. For et array-element, indstiller `?max_h=` (standard `720`) og `?jpeg_q=` (standard `80`) højden og JPEG-kvaliteten. Dette er den URL, som `lattice viewer` åbner. |
| `/api/camera/<serial>/stream-annotated` | Den samme forhåndsvisning med GUI&#x27;ens overlejringer tegnet ind: `zebra=1`, `crosshair=1`, `grid=3x3` (eller `9x9`), `histogram=1`, `peaking=1`, `spot=1`; uden forespørgsel bruger projektet de gemte overlay-indstillinger for det pågældende kamera. |
| `/api/camera/<serial>/stream-reflectance` | 8-bit reflektans pr. bånd: ρ = pixel ÷ den del`X-Chloros-Pixel-Scale`-header (255, medmindre `refl_max` er sat). `fps` (standard `10`), `jpeg_q` (standard `95`), `max_h` (standard `0` = native), `band=<name>`, `align=1`. JPEG, altså med tab. Afvist med HTTP-kode 409, når kameraet ikke har nogen reflektans (et RGB-kamera, ingen kalibreringspakke, ingen DAQ-aflæsning). |

- **Sammenkobling af billeder på tværs af kameraer.**Hver del, som et**array-medlem** sender, på alle tre strømme, bærer `X-Chloros-Group-Id` (arrayets synkroniserede gruppetæller), `X-Chloros-Frame-Id` og `X-Chloros-Timestamp-Ns`-headere. Dele fra forskellige kameraer med samme `X-Chloros-Group-Id` stammer fra den samme trigger. Et enkeltstående kamera har ingen gruppe og sender ingen gruppe-id. Browsere ignorerer overskrifter pr. del; en klient, der selv analyserer den flerdelte strøm, læser dem.
- **Sættes på pause, mens array-visningen er åben.** `/api/camera/stream/<serial>` fra et array-medlem sender en statisk »pauset«-ramme, mens arrayets samlede livestream er åben (f.eks. GUI&#x27;ens array-visning), og genoptager, når den lukkes.

#### Sensorjustering

| Underkommando | Formål |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Læs/skriv til enhver GenICam-node. `--set ExposureTime=…` (`Gain=…`) uden dets `ExposureAuto` (`GainAuto`)-indstillinger, der først slår automatisk tilstand fra, som `lattice exposure --set` gør; en værdi, som kameraet afviser, udskriver `Failed to set <node>: …` på stderr og afslutter `1`. `--get` og den standardgruppevisning udskriver en `note:` under `ExposureTime` / `Gain`, mens den automatiske tilstand kører — se [Aflæsning af eksponering og forstærkning i automatisk tilstand](#reading-exposure-and-gain-under-auto-mode). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Eksponering &amp; AE. En `--set`-eksponering eller et `--brightness`-mål fastholdes som dit (Chloros’ automatiske eksponeringsstyring lader den være i fred); forstærkningen forbliver, som den var, så i auto-tilstand justeres den løbende mod målet, indtil du indstiller `lattice gain`. `--auto` giver eksponeringen tilbage, `--brightness-auto` giver målet tilbage til Chloros’ automatiske mål. Uden flag læser den eksponeringsnoder med samme `note:` som `configure --get`. En `--set`-værdi, som kameraet afviser, udskriver `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` på stderr — i den automatiske tilstand, hvor skrivning var slået fra, genoprettes denne i stedet for at forblive slået fra — udskriver ingen `Exposure: … us (auto OFF)`-linje og afslutter `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Forstærkning og automatisk forstærkning. En `--set`-værdi, som kameraet afviser, udskriver `Failed to set Gain: …` på stderr, ingen `Gain: … dB (auto OFF)`-linje, og afslutter `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | Sensor-ROI og binning. Binning er altid **Gennemsnit** — se nedenfor. |
| `lattice format [--set FMT] [--list]` | Pixelformat. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Hardware-/softwareudløser. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (ingen flag = one-shot hvidbalance) | Hvidbalanceoperationer. Kun RGB-/Bayer-kameraer; en no-op (springes over) på mono M3M. One-shot bevarer den hvidbalance, den beregner (`BalanceWhiteAuto` og forstærkningerne); alle andre indstillinger, som dens open-write skriver, returneres som fundet, og det hele, når one-shot ikke kan køre. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | RGB-farvebehandlingskæde. `natural` (standard) er den billige live-efterbehandling; `enhanced` tilføjer defringe + vibrance + CLAHE lokal kontrast for det fulde hub-parity-look til ca. 2× behandlingsomkostningerne pr. billede, hvilket giver en lavere **live** billedhastighed — gemte optagelser får altid den fulde behandling uanset hvad. Kun RGB-/Bayer-kameraer; overspringes på mono M3M. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Mætning/kontrast i live-forhåndsvisning (kameraer med RGB-filter); `100` er neutralt, `--reset` indstiller begge til 100. Gemte optagelser påvirkes ikke. Springes over på mono M3M. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (ingen flag = læs) | Vis **højlys** (kameraer med RGB-filter): træk de lyseste, hvideste pixels ned — f.eks. en lys hvid overflade, der gengives som en flad hvid klat. `0` = slået fra, `-100` = stærkest. Live-forhåndsvisningen og billederne på skærmniveau følger denne indstilling; stråling/refleksion gør det aldrig, og den kan ikke gendanne sensordata, der er klippet (sænk eksponeringen for det). Springes over på mono M3M. |
| `lattice filter [--list]` | Viser en liste over de filtermodeller, som Chloros kender (`RGN-IMX265`, `OCN`, `NGB`, …). Et kameras filter aflæses fra dets modelstreng og kan ikke indstilles: `--set NAME` slår kun en model op og afslutter derefter `1` med `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Undersøger strøm-/termiske noder; skifter til strømbesparende hviletilstand. `--sleep` lader kameraet være i dvaletilstand (dets grænse for linkgennemstrømning er aktiveret (`DeviceLinkThroughputLimitMode` `On`) på det laveste niveau, og strømbesparelsestilstanden er aktiveret, hvis kameraet har en sådan) hvor alle andre indstillinger genindstilles til deres oprindelige tilstand; en dvaletilstand, der mislykkes, genindstilles fuldt ud. Vågnen sætter forbindelsen tilbage, som den var ved indgangen til dvaletilstanden (grænsen, dens tilstand og pakkeforsinkelsen `GevSCPD`, som kameraet holder på 0, mens grænsen er aktiveret; optegnelsen opbevares på denne computer, så en genopvågning efter genstart af backend også finder den), eller, for et kamera, som denne computer ikke har sat i dvaletilstand, men som ser ud til at være i dvaletilstand (dets grænseværdi er sat til bundniveau), med grænseværdien slået fra og en `GevSCPD`, der er efterladt på 0 ved 80 (fabriksindstillingerne); en grænse, der er indstillet over minimumsgrænsen, forbliver uændret. Sonden og genopvågningen gendanner det, som deres åbning skrev. |

> **Binning er altid gennemsnit.** `lattice resolution --binning N` indstiller gennemsnitsbinning og
> tilbyder ikke en tilstandsomskifter. Sum-binning multiplicerer hver DN med N², hvilket koster cirka to stop i
> højlysheadroom pr. binning-trin; Chloros dividerer denne faktor ud igen før den
> radiometriske kæde, men pixels, som Sum har drevet op til sensorens loft, bliver klippet og kan ikke
> gendannes. Det giver ingen fordele, som »Gennemsnit« ikke giver — et indeks er skalainvariant, og strålingsintensitets-
> skalærerne er kalibreret efter »Gennemsnit«-skalaen. Hvis et kamera allerede er indstillet til »Sum« (efterladt
> der af et andet værktøj eller ved en fabriksindstilling), registrerer Chloros det via en node-readback og
> kompenserer; kører man `lattice resolution --binning N`, sættes det tilbage til Average. 1.2.0
> `--binning-mode`-flag accepteres stadig, så ældre scripts kører, men der vises kun en advarsel:
> binning forbliver på »Average«.

> **Ændring af binning, pixelformat og trigger på et kamera har ingen effekt.** På et kamera, hvor
> backend forbliver åben og streamer (`cam-connect`, GUI’en), har `lattice resolution --binning`
> træder ikke i kraft (der vises en meddelelse om, at binning stadig er aktiv, `Binning: 1x1`), og
> `lattice format --set` og `lattice trigger --mode` afvises af kameraet
> (`SC_ERR_ERROR -1001`). Frigør den først (`lattice cam-disconnect -s <SN>`), eller indstil
> dem ved opkobling (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Aflæsning af eksponering og forstærkning i automatisk tilstand

Mens et kameras `ExposureAuto` (`GainAuto`) kører, gemmer dets `ExposureTime` (`Gain`) den **sidste værdi, der blev skrevet**, og ikke hvad kameraet laver lige nu. For et kamera, som backend’en holder åbent (`cam-connect`, `array-connect`, GUI’en), viser `lattice configure --get`, standardvisningen `lattice configure` og `lattice exposure` (ingen flag) derfor nodeværdien og, under den, en note med kameraets nyeste billede:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Læs billedværdien fra noten; nodeværdien er kun den seneste skrivning. Via HTTP bevarer det samme `POST /api/camera/<serial>/read-nodes`-svar `nodes` uændret og tilføjer `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, den nyeste rammes) og `notes` (sætningen ovenfor). Begge er kun til stede, så længe autotilstanden ikke er `Off`, og der kendes en ramme; et kamera, der kun er åbnet til læsning, har ingen ramme og får ingen af delene. `lattice gain` (ingen flag) udskriver kun noden — brug `lattice configure --get Gain` til noten.

##### Eksponering på et array-medlem

Mens et array streamer, kører hvert medlem med **`AcquisitionFrameRateEnable`**med en billedhastighed, der ligger lidt over arrayets triggerhastighed: dens periode er triggerperioden minus 15 % (mindst 3 ms) eller kameraets egen maksimale hastighed, når triggeren allerede er tæt på denne. `configure --get` på et medlem læser derfor `AcquisitionFrameRateEnable` som `True` og `AcquisitionFrameRate` ved denne indstilling; aktiveringen slås fra igen, når arrayet stopper.**Skriv ikke disse to noder på et array-medlem** — arrayet holder dem synkroniseret med sin trigger og overskriver dem.

Denne indstilling er også medlemmets eksponeringsloft: ca. 85 ms ved en trigger på 10 fps (100 ms periode), uanset **ROI eller binning**. Før version 1.3.0 begrænsede en lille ROI i stedet et medlems eksponering til sensorens friløbsperiode (4,9 ms ved 768 × 304), uanset hvor langsomt arrayet udløste.**En manuel `ExposureTime`, der er længere end loftet, sænker udløsningshastigheden, så den passer til den**, når den skrives med `lattice exposure --set` eller `configure --set` (eller fra GUI&#x27;en). Arrayet åbner først det pågældende medlems loft for værdien og lader den langsommere hastighed køre i et par udløsningsperioder, så skrivningen kan tage et par sekunder længere; udløseren udløses derefter ikke hurtigere, end eksponeringen tillader. På et array med automatisk hastighed vender den tidligere hastighed tilbage, så snart hvert medlem, hvis manuelle eksponering bremsede den, er tilbage på automatisk eksponering. En værdi, som kameraet selv ikke kan optage, afvises stadig, med det acceptable interval angivet i fejlmeddelelsen, og medlemmets automatiske eksponering genindstilles, hvis den kun var slået fra for den pågældende skrivning. En opskrift med eksponeringen `apply` bremser ikke udløseren: den holdes på loftet. Chloros forkorter aldrig en manuel eksponering for at passe til en hurtigere udløser; en fastsat hastighed (`array-connect --target-fps F`), der er hurtigere end en eksponering tillader, giver det pågældende medlem et loft, der er langt nok til den, og det pågældende medlem tager ikke udløsere hurtigere, end dets eksponering tillader.

#### Kalibrering og sensorer

| Underkommando | Formål |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibrer ud fra et reflektansmål. Der søges efter det i op til `--attempts` rammer (standard `30`); en ufuldstændig ramme (GVSP-pakketab) koster ét forsøg, ikke hele kørslen. Det mislykkes ved `no target detected in N attempts`, eller ved `no complete frame in N attempts: <last error>`, når ingen ramme ankom i sin helhed. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | En DAQ-M&#x27;s spektrum og båndstørrelser, læst via Bluetooth (BLE) gennem backend&#x27;en. `--mac` er sensorens egen BLE-MAC-adresse og er påkrævet ved alle tre handlinger: der er ingen standardværdi, da hver enhed har sin egen (`daq pool-discover --only ble` viser dem); uden den afsluttes kommandoen med `1`. Uden nogen handling vises hjælpeteksten. `--camera SERIAL` integrerer de viste båndstørrelser gennem kameraets egen målte spektrale respons i stedet for filtermodellens fladkurve — RGB-kameraer, hvor kameraets kalibreringspakke er indlæst (tilsluttet) eller gemt i cachen på denne computer. Overskriften angiver, hvis kurve der blev brugt: `Band magnitudes (camera <SERIAL>):`, eller `Band magnitudes (model <filter>):`, når kameraet ikke har nogen (ikke et RGB-kamera, eller ingen pakke indlæst eller gemt i cachen), og standardkurven blev i stedet anvendt. Via HTTP tager `POST /api/dls/cli-read` imod `camera_serial` og returnerer `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Anvend vignetteringskorrektion på eksisterende billeder, hvert med sit eget objektivs flade felt (fra dets EXIF-model). `--lens-model` anvendes på billeder, hvis model ikke angiver noget kamera; et billede, hvis model er i modstrid hermed, springes over. |

#### Flere kameraer (midlertidige sessioner)

En midlertidig kommando åbner kameraerne for én kørsel og lukker dem, når den er afsluttet: `multi-capture` uden tilknyttet permanent array, `multi-stream` og `multi-test` her, og `align-calibrate`, `align-apply`, `align-stream` og `index --live` nedenfor. Disse regler gælder for dem alle:

- **`--serials SN1,SN2,…` vælger kameraerne.** Kun disse åbnes, og det første udløser triggeren (for `align-calibrate`, `--reference`, når de er angivet). En serienummer, der ikke er på netværket, får kørslen til at mislykkes, før noget kamera åbnes. Uden `--serials` åbnes alle kameraer på netværket. Et kamera, som Chloros allerede har åbent (GUI’en, `cam-connect`, `array-connect`), afviser kørslen, uanset om `--serials` angiver det eller ej: det deler det netværk, som kørslen er indstillet til. (`index --live` uden `--serials` åbner i stedet de aktiverede kameraer i sin profil: se [Indeks / Vegetationsmatematik](#index--vegetation-maths).)
- **Kameraerne deler netværket på samme måde som `array-connect` deler det.** Hvert kamera tilpasses sin andel af det, som værtsforbindelsen bærer (en forsinkelse mellem pakker, ikke kameraets gennemstrømningsgrænse, hvis nedre grænse på 31,25 MB/s fire kameraer på 1 GbE allerede overskrider), så fire kameraer kører på en 1 Gbps-forbindelse. Når kameraerne ikke alle kan sende på én gang med den langsomste hastighed, de kan nå, udløses de ét efter ét, hvert med forbindelsen for sig selv, og kommandoen udskriver `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Når selv ét kamera ad gangen ikke passer ind, afvises kørslen, før noget kamera streamer: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP-kode 409, afslutning `1`.
- **Alle kameraer skal levere, ellers mislykkes kørslen.** Et kamera, der ikke leverer nogen ramme, er en fejl: kommandoen viser `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, hvor `<why>` er det pågældende kameras hentefejl, efterfulgt af årsagsangivelsen, og afslutter med `1`. En ramme, der ankom med GVSP-pakketab (`IncompleteBufferError: GVSP packet loss …`), betyder, at netværket ikke kan håndtere kameraernes samlede båndbredde (`Every missing frame arrived with GVSP packet loss: …`): Kontroller linkhastigheden på værten og på switchens uplink (`lattice network`), brug jumbo-rammer, eller åbn færre kameraer med `--serials`. Et kamera, der ikke leverede noget, modtager normalt ikke masterens trigger: Kontroller dets M8-synkroniseringskabel, og kør derefter `lattice multi-detect`. Hvad hver kommando gør, når der mangler et kamera i en rammegruppe, fremgår af den pågældende række.
- **Hvert kamera sættes tilbage, som det blev fundet.** Når kørslen afsluttes, får hvert kamera det tilbage, det havde, da kørslen åbnede det: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, samt `LineMode` / `LineSource` på `Line2` og `Line3` (og deglitching af trigger-indgangen fra en matrix giver en slaves linje, `LineFilterSelector` / `LineFilterWidth`); dets `PtpEnable` (og `PtpSlaveOnly` / `PtpDomainNumber` indstillet sammen med det); `ExposureAuto` / `GainAuto` (og `ExposureTime` / `Gain`, hvor den var indstillet til en manuel værdi); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (og hver `BalanceRatio`, hvor hvidbalancen var manuel); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, binning, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (og `Gamma`, hvor gamma var aktiveret), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; pakkestørrelsen (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); takten, `DeviceLinkThroughputLimitMode` (og `DeviceLinkThroughputLimit`, hvor tilstanden var `On`), forsinkelsen mellempakkeforsinkelse `GevSCPD` og startforskydningen `GevSCFTD`, som en matrix giver hvert kamera; de datablokke, som hver strøm aktiverer (`ChunkModeActive`, hver `ChunkEnable` under sin `ChunkSelector`); `AcquisitionFrameRateEnable` (og `AcquisitionFrameRate`, hvor aktiveringen var slået til — når den er slået fra, er hastigheden skrivebeskyttet og læser det maksimale, som takten tillader, så det er ikke en indstilling). Kun det, der adskiller sig, skrives; kameraet stoppede først, pacing og billedhastighed før en eksponering, der kræver sin billedperiode (et kamera begrænser `ExposureTime` til én billedperiode). Med `--keep-settings` fjernes trigger- og synkroniseringsledningsforbindelsen, kameraet kører frit, og resten gemmes i kameraets egen hukommelse, som kørselssættet har indstillet det; `--clear-settings` indstiller fabriksindstillingerne der.
- **Eksponeringen indstilles ud fra scenen, før det første billede gemmes.** Optagelsen tager synkroniserede billedgrupper og kasserer dem, mens den indstiller hvert kameras eksponering og forstærkning ud fra disse billeder. Målet er en gennemsnitlig lysstyrke på kameraets mål, hvor ikke mere end 1 % af billedet er overskydende; en scene med lyse højlys stopper før målet nås. Dette tager normalt 2–6 billedgrupper og højst 5 s.
  - Hvert kamera retter sig mod det mål, som en tilsluttet matrix starter det med, via sit filter: TargetBrightness **40**på et multispektralt kamera (RGN, OCN, NGB) og**60** på et RGB-kamera. Det er halvdelen af det eksponeringsmål, som 80 giver (tre fjerdedele på RGB), så der er mindre risiko for, at et lyst reflektanspanel bliver klippet. `--brightness N` (0-255) indstiller i stedet hvert kameras mål (`multi-test` har ingen `--brightness`: dets automatiske eksponering er slået fra). Backendets log navngiver hvert kameras mål: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - Eksponeringen kommer først, op til kameraets egen grænse for automatisk eksponering. Forstærkningen dækker kun det, der er tilbage.
  - Den fundne eksponering og forstærkning bevares gennem hele optagelsen, så hvert billede i en `multi-stream` tages med samme eksponering. En optagelse, der skal følge skiftende lysforhold, hører hjemme i en tilknyttet matrix (`array-connect`), hvor den automatiske eksponering løbende justeres.
  - `--exposure` og `--gain` bevares nøjagtigt som angivet. Når begge er angivet, kasseres ingen billeder.
  - Når kørslen afsluttes, aktiveres hvert kameras automatiske eksponering igen.
  - Backendens log rapporterer resultatet: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …` eller `exposure NOT solved after …` med årsagen og `(not settled)` for hvert kamera, hvor dette ikke var tilfældet. I dette tilfælde tages billederne stadig.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` ændrer grænsen på 5 sekunder; `0` deaktiverer denne funktion (se [Miljøvariabler](#environment-variables)).

**Mens Chloros holder et hvilket som helst kamera** (GUI’en, `cam-connect` eller et tilsluttet array), `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream` og `index --live` nægter at køre, fordi de åbner alle kameraer på netværket, og en anden åbning af et reserveret kamera kan få backend&#x27;en til at gå ned. (`index --live` afviser, når et kamera, det vil åbne, er reserveret: dets `--serials` eller dets profils kameraer, samt ethvert kamera for en profil, der ikke angiver nogen.) Backend&#x27;et returnerer HTTP-status 409 `CAMERA_HELD`, hvor hvert reserveret kamera og frigivelseskommandoerne (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`), hvorefter kommandoen afsluttes med `1`. Når et array er tilsluttet, registrerer `array-capture --continuous` det i stedet for `multi-stream`, og `array-capture` (justeret som standard) gemmer et justeret sæt i stedet for `align-apply`; `multi-capture` og `align-calibrate` bruger et tilsluttet array, der indeholder deres kameraer, i stedet for at åbne dem.

| Underkommando | Formål |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Viser alle kameraer med synkroniseringsroller; med `--serials` vises kun disse, det første som `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Ét synkroniseret billede fra hvert kamera. Gemmer **alle eksporttyper som standard**, når der er tilsluttet et permanent array, og den DAQ-aflæsning, der blev brugt ([Den DAQ-aflæsning, som en optagelse gemmer](#the-daq-reading-a-capture-saves)); den midlertidige fallback uden array er**kun debayered** (kør først `array-connect` for resten). `--brightness` gælder kun for fallback: et tilsluttet array beholder sin egen autoeksponering. Fallback gemmer ikke `.daq` (`--no-daq` / `--daq-tolerance-ms` har ingen effekt der), og `multi-capture` tager ikke `--force-daq`. I fallback-tilstanden vil et kamera, der ikke leverer nogen ramme, mislykkes i optagelsen (`Error: multi-capture failed: no frame from …`, afbrydelse `1`), og der gemmes ingen fil fra den pågældende rammegruppe. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Stream synkroniserede billeder (midlertidigt), `--fps` standard `3`, indtil Ctrl+C eller `--count`-grupper. En billedgruppe, hvor et kamera ikke har leveret noget, skrives ikke, og tre i træk afslutter streamen. Hvis en gruppe blev sprunget over, afsluttes kørslen ved at udskrive `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)` samt `Error: the stream stopped: …` med angivelse af kameraet, da streamen blev afsluttet, og afslutter `1`X. Stopper automatisk, når drevet er fyldt, udskriver `Failed:` / `Disk full:` og afslutter med `1`; enhver fil, den ikke kunne skrive, afslutter også `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | GPIO-synkroniseringstest (forbigående), `--count` standard `10`-udløsere. Viser, hvordan netværket deles (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)`, og linjen `Serial emit:`, når kameraerne udløses efter hinanden, hvilket tidsstemplerne derefter afspejler), en `Sync group: master SN:<serial> -> SN:<serial>, …`-linje pr. synkroniseringsgruppe (`(no slaves: not synced)` for et kamera uden synkroniseringskabel, som udløses på egen hånd), derefter én række pr. udløsning: `[  0] SN:<serial>=<timestamp> \| …`, med `SN:<serial>=(no frame)` for et kamera, der ikke leverede noget ved den pågældende udløsning. Derefter en linje med `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` og en linje med `SN:<serial>: mean <±x> us, jitter <y> us` pr. kamera, hvor hvert kamera er synkroniseret med masteren inden for et ur: en udløser, hvis tidsstempler blander kameraernes enhedsure og PTP (PTP slået fra eller stadig i gang med at låse) udelades, mens en `Note:` tæller dem. Ethvert udeladt billede medfører, at testen mislykkes (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, afslut `1`); tre ufuldstændige triggere i træk afslutter den før tid. Den mislykkes også (afslut `1`), når færre end to triggere har alle kameraer på samme ur (`… the clocks cannot be compared`), eller når urene aldrig stemmer overens: PTP-ure, der afviger mere end 1 ms (plus eventuel forskydning på det serielle niveau) fra masteren (`… PTP is not locked`), eller enhedsure, hvis forskydning ændrer sig med mere end dette mellem udløsningerne. Den starter PTP-grandmasteren, hvis den ikke kører, aktiverer PTP på alle kameraer (som `array-connect` gør, uanset hvordan netværket ser ud), og inden den første trigger venter den på, at alle kameras PTP-ure låser sig fast på grandmasteren og stabiliserer sig: `PtpStatus` Slave på én grandmaster, inden for 100 us af denne og stabil i 3 s, og hvert kamera låst på denne måde, uden afbrydelse, i 120 s — ca. 2 minutter fra opstart. Et ur, der netop har låst sig fast, afviger med op til ~200 us i ca. 2 minutter, efter at det viser Slave, og kameraets eget `PtpOffsetFromMaster` viser det ikke; 120 s bringer kameraerne inden for ~10 us af et stabilt ur. Et kamera er stadig låst siden en tidligere test, hvor PTP blev efterladt tændt, som denne backend så det, tæller fra den låsning, så en test lige efter en anden venter ikke igen; et kamera, som testen tænder, eller et, der findes med PTP aktiveret, og som denne backend ikke har set låse sig fast (tændt et andet sted eller før en genstart af backenden), venter sine 120 s, og i en række af begge kameraer, der netop er låst fast, venter de stadig på deres. Et kamera, der forlader Slave-tilstanden, overskrider 100 us eller følger en anden grandmaster i 3 aflæsninger i træk (1,5 s) under ventetiden, starter sin tælling forfra; en eller to sådanne aflæsninger logges (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`), og tællingen fortsætter, og et grandmaster-id, der ikke kan aflæses, betragtes som ukendt, ikke som en ændring. Når alle kameraers tælling genstarter på én gang (inden for et sekund), skyldes det, at grandmasterens tid bevæger sig, ikke kameraernes: backendens log viser `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` viser det i `grandmaster_steps`, og en ventetid, der derefter udløber, starter sin fejlmeddelelse med `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Grænsen forlænges ikke for dette. `--ptp-settle SECONDS` indstiller afventningstiden for kørslen (0-600; `0` for ingen afventning). Ventetiden er begrænset til stabiliseringstiden plus 90 s (210 s som standard, mindst 90 s), og kommandoen venter på backend i 240 s plus stabiliseringstiden (360 s som standard). Den udskriver `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`, og backendens log indeholder en linje hvert 5. sekund med hvert kameras tilstand og offset, hvor længe det har været låst, og hvor lang tid der er tilbage: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Hvert kamera tændes med PTP slået fra; hvorefter urene låser sig fast på ca. 10 sekunder. Bagefter sættes hvert kamera tilbage til den tilstand, testen fandt det i — dets PTP, den automatiske eksponering og forstærkning, som testen holder slået fra under kørslen, og resten (medmindre `--keep-settings`) — så et kamera, der er tilbage med PTP slået til (`lattice configure -s <serial> --set PtpEnable=True`, eller et testforløb med `--keep-settings`), forbliver synkroniseret, og når denne backend har konstateret, at det har været synkroniseret i stabiliseringstiden, venter en senere test ikke på det. Hvis urene ikke stabiliserer sig under ventetiden, fejler det før udløsning (`PTP: NOT locked …`, afslut `1`) og angiver, hvad der skal gøres: et kamera, der aldrig blev låst fast (`PtpStatus Listening`), betyder, at grandmasteren ikke når frem til det (`time-sync status`); et kamera, der hele tiden faldt ud, navngives efter, hvor længe det havde været låst fast (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); ure, der stadig konvergerer: tjek `time-sync status`, og for at få dem låst fast inden testen skal du aktivere PTP og lade det være slået til (`lattice configure -s <serial> --set PtpEnable=True` for hvert kamera, eller kør testen med `--keep-settings`; testen genindstiller det, den har fundet, så den forbliver slået til), og kør den derefter. |
| `lattice multi-detect [--line LINE] [--json]` | Find kameraerne på M8-synkroniseringslinjen og en standardmaster: RGB-kameraet (model, der slutter på `-FRGB`), hvis det er tilsluttet via kabel, ellers det første tilsluttede kamera i rækkefølge efter serienummer. M8-kablerne mødes i et samlingspunkt, så **ethvert tilsluttet kamera kan være master** — den påtrykte betegnelse `MASTER` er en standardindstilling, ikke en kablet kendsgerning. Hvert kamera styrer linjen efter tur, og de øvrige kontrolleres for dets puls; et kamera, der allerede er fundet i den kablede gruppe, undersøges ikke igen, så en fuldt kablet opstilling tager én runde. Uden `--line` undersøger den `Line2`, derefter `Line3`. Udskriver `MASTER: SN:…`, ét `SLAVE: SN:…` for hvert andet kablet kamera og `(probed on <line>)`; `--json` tilføjer det fulde resultat (`master`, `slaves`, `no_cable` — kameraer, som pulsen ikke nåede — `line` samt `results` pr. kamera). Når processen er afsluttet, genindstilles hvert kamera til de oprindelige indstillinger for trigger og synkroniseringslinje (de samme noder som ovenfor). |

#### Justering

| Underkommando | Formål |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — plus detektor-/matcher-knapper `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC-knapper `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombination af flere billeder `[--averaging mean\|median\|inlier_weighted]`, geometriske begrænsninger `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, rumlig begrænsning `[--roi X0,Y0,X1,Y1] [--mask PATH]` ogslave-overstyringer `[--per-cam-override SN:KEY=VALUE]` (kan gentages) | Beregn justeringsprofil fra live-kameraer. Standardindstillinger: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (i den mappe, du kører det fra), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate` har sine egne standardindstillinger; se [dets tabel](#project-align-calibrate-options).) `--prefilter` har som standardindstilling `gradient` (kantkort; svarer til GUI/array-aligneren — kanterne bevares på tværs af spektralbåndene). `--matcher flann` giver det bedste resultat ved over ~5000 træk; `--averaging median` er robust over for én dårlig optagelse, `inlier_weighted` vægter efter antal match; `--lock-scale` projicerer til den nærmeste rotation (ingen skalering), `--lock-axis` sætter en translationskomponent til nul; `--mask` gælder for alle kameraer (brug `--per-cam-override` til indstillinger pr. kamera, f.eks. `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` nægter at gemme en kalibrering, hvis RMS for reprojiceringen overskrider grænseværdien. `--method manual` henter sine punkter fra `--points PATH`: et JSON-objekt `{"<serial>": [[x, y], ...], ...}`, der angiver hvert kameras pixelkoordinater for de samme træk, inklusive referencekameraet (mindst 2 pr. kamera); hver enkelt af `--frames`-kalibreringerne løses ud fra disse punkter. Når et tilsluttet array indeholder kameraerne, bruger kalibreringen arrayets live-billeder, og profilen bliver også arrayets justering for dets optagelser. Ellers åbnes et [midlertidigt array](#multi-camera-transient-sessions): et kamera, der ikke leverer noget billede i nogen af `--frames`, fejler kalibreringen (afslut `1` og navngiv den), og der gemmes ingen profil. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Optag et justeret multibåndsbillede. `--bit-depth` indstilles som standard til at matche kameraet; `--no-crop` bevarer hele billedet (udfyldes med sort); `--interpolation` (standard `linear`) og `--border-mode`/`--border-value` (standard `constant`/0) styrer CPU-warp — GPU-stien er uanset hvad bilineær. Åbner en [transient array](#multi-camera-transient-sessions): et kamera, der ikke leverer nogen ramme, fejler optagelsen (afslutter `1` og navngiver den), og der skrives intet. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Stream-justerede multibånds-frames (samme warp-knapper som `align-apply`), `--fps` standard `3`. Åbner en [transient array](#multi-camera-transient-sessions): en billedgruppe, som et kamera ikke har leveret noget til, gemmes ikke, og tre i træk afslutter streamen. Hvis en gruppe blev sprunget over, afsluttes kørslen ved at udskrive `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` og `Error: …`, og afsluttes med `1`. En strøm, der afsluttes på grund af en hvilken som helst anden fejl, udskriver også `Error: …` og afsluttes med `1`. |
| `lattice align-info --profile PATH [--json]` | Vis profiloplysninger. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Skift lagorden. |

#### Indeks / Vegetationsmatematik

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` knytter en forudindstillings symboler til bånd** (NDVI: `red`, `nir`), én `--channel SYMBOL=BAND` for hver.

- **`--live`: kun for et symbol, som stakken ikke kan besvare.** Ligesom med `--input` tager et forudindstillingens symbol, som du udelader, det ene bånd i stakken (de bånd fra profilens aktiverede kameraer), hvis navn begynder med det. Dette kontrolleres, før et kamera åbnes: et symbol, som intet bånd eller flere bånd svarer til, afvises derefter, idet de navngives, og `1` afsluttes. En brugerdefineret `--formula` kræver en `--channel` for hvert symbol.
- **`--live`-produkter angiver, hvem de tilhører.** `index.tif` og `index_LUT.png` indeholder stakens kamera (mærke, model, serienummer), dets optagelsestidspunkt (referencekameraets eksponering, UTC) og objektiv, og begge angiver indekset som deres eneste bånd (`XMP-Camera:BandName` `NDVI`; farve-PNG-filen angiver indekset, ikke dens farvekanaler), ligesom et `--input`-produkt gør. Et indeks fra flere kamerabånd navngiver i stedet hvert enkelt af dem, og dets `Model` er `LATTICE` — se [Filer lavet fra flere kameraer](#files-made-from-several-cameras).
- **`--input`: kun til et symbol, som filen ikke kan svare på.** Et symbol, du udelader, tager det ene bånd i filen, hvis navn begynder med det: `red` tager `Red_660` eller `Red_660_RGN`, `nir` tager `NIR_850`. Navnene stammer fra selve filen: stakens egen båndliste eller, på en Chloros-radiance-/reflektansoptagelse eller et `process`-produkt, dens `Chloros:Band{R,G,B}Name`-/`XMP-Camera:BandName`-tags. Så `lattice index --input <reflectance capture> --preset NDVI` behøver ingen `--channel`.
- **To bånd, der svarer til ét symbol, gisnes ikke imellem** (en stak med 9 kameraer har to NIR-bånd): navngiv det ene. Et symbol, som intet svarer til, afvises med `channel_map missing entries`, afslut med `1`.**Båndnavne i en stak** angiver kameraets filter: et M3C-kameras bånd er `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` på en RGN; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` på en OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` på en RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` på en NGB — og et M3M-kamera&#x27;eneste bånd er `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` angiver båndene i en profil.

Fuldt flagsæt: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (kan gentages), `--capture-level raw|debayered|radiance|reflectance|unknown` (tilsidesætter det optagelsesniveau, der er registreret i kilde-TIFF-filen; standard: læses fra TIFF-metadata, med Chloros-filens `Chloros:ProcessingLevel` som standard, så en reflektansoptagelse rapporterer `reflectance`), `--output PATH` / `-o` (offline: en sti; med en billedfiltypenavn navngives filerne ud fra stammen — `-o ndvi_FRGN.png` skriver `ndvi_FRGN.tif` og `ndvi_FRGN_LUT.png` — og uden en filtypenavn er det et præfiks, `-o ndvi` -&gt; `ndvi.tif`, `ndvi_LUT.png`; standard ved siden af inputtet, som `<input name>_<preset>`; med `--live` og uden `-o`, en `output/`-mappe under den mappe, hvorfra du kører kommandoen), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Med `--live` gælder justeringsknapperne også: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (kameraernes mål for automatisk eksponering, se reglerne for forbigående sessioner), `--serials SN1,SN2,…`. `--live` åbner en [array for forbigående sessioner](#multi-camera-transient-sessions): et kamera, der ikke leverer noget billede, får kørslen til at mislykkes (afslut `1` og navngiv det), og der skrives hverken stak eller indeks. Det åbner `--serials` (det første udløser triggeren), ellers profilens kameraer, hvis `layer_order`-post er aktiveret, med dets reference først — referencen selv med dets bånd deaktiveret, da de andre er justeret i forhold til den, og den udløser triggeren. Et kamera, hvis bånd er deaktiveret, kalibreres aldrig, så et RGB-kamera, der er deaktiveret i profilen, fejler ikke længere et reflektansindeks (`no-absolute-radiance-scale`). En profil, der ikke navngiver nogen kameraer, åbner alle kameraer på netværket, som før.

Hver formular kører på backend&#x27;en, så den skal være i gang: Start den fra Chloros-GUI’en eller med den kommando, som CLI’en viser, når den ikke kan nå den. Offline læser backend’en `--input` og skriver selv float-TIFF-filen og den farvelagte PNG-filen, så en relativ `--input` eller `-o` henviser til den mappe, du kører kommandoen fra (se [Konventioner](#conventions)). Begge offline-produkter indeholder inputfilens kameradata, ligesom en `process`-indekseksport gør: Mærke / Model / serienummer, GPS, objektivmærkerne, optagelsestidspunktet (inputfilens eget; når dens EXIF ikke angiver noget, det eksponeringstidspunkt, den registrerede, eller filnavnets tidsstempel, i UTC) samt MAPIR-kalibreringsidentiteten — aldrig raw-DN-kalibreringen, som ikke gælder for et indeks — plus `Chloros:ProcessingLevel=index` og indeksudtrykket; float-TIFF-navnet angiver et bånd, forudindstillingen. `--list-presets` og `--list-gradients` udskriver backendens forudindstillings- og gradienttabeller. En formel, som backend&#x27;en afviser, eller et `--channel`-bånd, som TIFF&#x27;en ikke har, afslutter `1` med backend&#x27;ens begrundelse.

> **`--channel`-symboler er store- og småbogstavsfølsomme.** Symbolsiden skal stemme nøjagtigt overens med forudindstillingens kanalnavne (forudindstillinger bruger små bogstaver, f.eks. NDVI = `red`,`nir` — se `--list-presets`), og båndsiden skal matche et båndnavn i den tilpassede stak (eller være et 0-baseret båndindeks i offline-tilstand). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` fungerer; `--channel RED=660` mislykkes med en `channel_map missing entries` fejl.

#### `lattice index` — multibåndsstakmasker

Ved `--capture-level radiance` eller `reflectance` kan hvert kameras 3×3-unmix begrænse et bånd til 0 (se [Unmix-begrænsningsmaske](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). En multibåndsstak indeholder bånd fra flere kameraer, og hvert kamera begrænser sit eget, så `--live --save-multiband` skriver **én maske pr. kildekamera** ved siden af stakken:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Hver maske er en enkeltbånds-`uint8` på **stakkens** pixelgitter (gennem den samme justeringsforvrængning og beskæring) med det pågældende kameras egne bits: bit 0 = dets R-kanals bånd, bit 1 = G, bit 2 = B. Et stakbånd hentet fra kanal *c* på et kamera begrænses der, hvor det pågældende kameras bit *c* (R = 0, G = 1, B = 2) er sat.
- Hvert bånds indlejrede herkomst angiver dets kameras maske (`unmix_clamp_mask`, i forhold til stakens mappe, eller `none`) og den bit, der skal læses (`unmix_clamp_mask_bit`).
- Et kamera, hvis unmix-funktion ikke har fastsat noget, får ingen fil; et mono- eller RGB-kamera kører ingen unmix og får heller ingen. CLI&#x27;en udskriver hver maske under stakken (`clamp mask SN…: …`); svaret viser dem som `clamp_masks`.
- Indeks-TIFF-/PNG-filen fra samme kørsel beregnes ud fra den samme stak på det samme gitter, så maskerne beskriver også denne. `--capture-level raw` udfører ingen unmix og skriver ingen maske.
- Stakken og indekset ligger direkte i `--output`, så maskerne havner i `--output`’s egen `clamp_masks/` — aldrig over den mappe, du har navngivet. Den mappe indeholder forskellige produkttyper (stak, indeks, farvelagt rendering): Vælg filerne herfra; overfør ikke hele mappen til fotogrammetri-softwaren.

#### Vedvarende forbindelser (Smart-Prep, GUI-ækvivalent flow)

Disse kommandoer holder kameraer åbne i backend-puljen på tværs af CLI-kald.

| Underkommando | Formål |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Tilføj et kamera til puljen (enkeltkamera, ikke array). De [fælles indstillinger](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` og `--packet-size` (`standard` = 1500, `jumbo` = 9000 eller bytes; `auto`-prober) anvendes ved tilslutning (en `--exposure` eller `--gain` slår automatisk tilstand fra). Den returnerer, når kameraets stream er startet (venter ca. 15 s på det, derefter 10 s på den første fulde buffer) og udskriver `stream: streaming` eller `stream: no_frames -- connected, no frames yet: …`, når streamen ikke leverede noget — for et kamera, der bruger jumbo-pakker, med anbefalingen om at oprette forbindelse igen med `--packet-size standard`. Mens et kamera opretter forbindelse (ved den første forbindelse downloades dets kalibrering, ca. 70 sek.), venter en anden kommando, der har brug for et kamera, højst 5 sekunder på det, hvorefter den fejler med HTTP-kode 503 `CAMERA_POOL_BUSY`, hvor kameraet, der opretter forbindelse, nævnes; kør den igen om et øjeblik. `lattice status` og kameralisten venter aldrig: et kamera, der er midt i en forbindelsesoprettelse, vises som værende under oprettelse med sin fase. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Frigivelse. Kameraet sættes tilbage til den tilstand, det var i ved opkoblingen (udløserforbindelse, PTP, indstillinger); `--keep-settings` gemmer i stedet sine indstillinger i sin egen hukommelse, `--clear-settings` nulstiller denne hukommelse til fabriksindstillingerne (se [Kamerahukommelse](#camera-memory----keep-settings----clear-settings)). Et kamera, der er medlem af en gruppe, frigiver hele gruppen. |
| `lattice cam-list` | Vis kameraer i puljen. |
| **`lattice array-connect`**|**Opret forbindelse til en vedvarende synkroniseret array (DET anbefalede startpunkt).** Kører hele GUI-smart-prep-forløbet. Det tager cirka et minut for 9 kameraer; lige efter at kameraerne er tændt, kan det tage flere. Mens den venter, udskriver den hvert trin, som backend&#x27;en når, som `    <s> s: <step>` (for eksempel `    41 s: open SN:214000535 (4 of 9)`), og backendens log indeholder én tidslinje pr. trin (`[array-connect <id>] <step>: <s> s …`) samt en sidste linje med de langsomste trin. Den venter op til 600 s. En forbindelse, der ikke er oprettet inden da, afbrydes af backendet: hvert kamera, den har åbnet, sættes tilbage, som det blev fundet, og lukkes, kommandoen udskriver `Error:` med det trin, den var nået til, og afsluttes med `1`, og der er ingen array forbundet (HTTP 504 `CLIENT_DEADLINE_PASSED`). Ctrl+C afbryder forbindelsen på samme måde (`Interrupted: …`); hvis arrayet allerede var forbundet, angiver linjen dette og navngiver den kommando (`array-disconnect`), der frigiver det. En kommando, der afbrydes, mens den venter (`kill -9`) medfører, at forbindelsen afbrydes, når backend ikke har hørt fra den i 30 sekunder. Et array, der er oppe, men endnu ikke har en synkroniseret rammegruppe, rapporteres stadig som tilsluttet, med sin strøm som `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Frigiv et array. Hvert kamera efterlades i fritløb (`TriggerMode` `Off`), mens alt andet forbliver, som forbindelsen fandt det — dets trigger- og synkroniseringslinjeindstillinger (masterenss `TriggerSource` tilbage til det, den var, dens synkroniseringslinje tilbage til en indgang), dens PTP og de kameraindstillinger, som sessionen ændrede — så en kommando til et enkelt kamera på et medlem virker med det samme. `--keep-settings` gemmer hvert kamerasindstillinger, som de var ved afslutningen af sessionen, i kameraets egen hukommelse; `--clear-settings` nulstiller denne hukommelse til fabriksindstillingerne. Se [Kamerahukommelse](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Vis tilsluttede arrays. |
| `lattice array-status [--array-id ID]` | Live fps, PTP, sidste fejl. `ptp_enabled` er kun `True`, når alle medlemmer viser PTP som aktiveret; `ptp_off:` angiver et medlem, der ikke gør det. `state:`-linjen læser `restarting (applying settings)` under en genstart af indstillingerne, `recovering (…)` mens et medlem, der har mistet strømmen, genåbnes (se bemærkningen nedenfor), `stalled (…)` mens grab-loopen kører, men ikke offentliggør nogen rammegruppe, og `stopped (…)` når grab-loopen har givet op (genforbind arrayet). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Én synkroniseret optagelse fra live-arrayet — Enkelt / Kontinuerlig / Interval / Hurtigst. **Standardindstilling er `all`**(én fil pr. relevant eksporttype pr. kamera),**justeret** i overensstemmelse med appens optagelse (`--no-aligned` for native filer). Oversprungne kameraer (f.eks. RGB, der er udelukket fra strålingsintensitet/refleksion) rapporteres med `Skipped: SN:<serial> (<reason>)`; den DAQ, der aflæser den anvendte optagelse, gemmes sammen med den, en optagelse, der kun indeholder rådata, medtages, og rapporteres med `DAQ: <path>` ([Den DAQ, der aflæser en optagelse, gemmer](#den-daq-der-læser-en-optagelse-gemmer)). Se [Optagelsestilstande, optagere og offline-genbehandling](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Optag den kombinerede live-indeksvisning som video/GIF (overvågningskvalitet; uden åben GUI-visning gengiver backend-systemet arrayets NDVI, eller `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Rå-Bayer-burst med høj billedfrekvens (analyse-kvalitet; genbehandles offline). Ligesom `array-record` og `array-capture --continuous` tjekker den ind hos backend ved hver afstemning: afbrudt uden at stoppe (`kill -9`), backend&#x27;en stopper den 30 sekunder efter den sidste check-in (`stop_reason` `client_gone`; burstens manifest færdiggøres, og opbygningen af det kombinerede indeks påbegyndes, som ved et stop), og arrayet forbliver forbundet. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Genbehandle en gemt rå burst til kalibrerede videoer. |

> **Et kamera, der mister strømmen eller forbindelsen, kommer af sig selv tilbage.** Når et medlem holder op med at svare (en switch genstarter alle de kameraer, den forsyner, ved en strømcyklus), venter arrayet på det, genåbner forbindelsen, når det svarer igen, og giver det tilbage, hvad `array-connect` har indstillet: PTP, trigger- og synkroniseringslinjen, netværkspacing, pixelformat og billedstørrelse samt automatisk eksponering. Det streamer derefter igen uden nye `array-connect`-indstillinger. I mellemtiden viser `array-status` `state: recovering (...)`, og `last_error` angiver, hvilket kamera der ventes på; bagefter udskrives `re-opened:`. Et kamera, der er genstartet, vender tilbage med PTP slået fra, så indtil det genåbnes, er `ptp_enabled` lig med `False`, og `ptp_off:` navngiver det.

> **Et kamera, der holdes, åbnes aldrig en anden gang.** En anden åbning stopper holderens strøm og overskriver dens indstillinger, hvilket kan få backend til at gå ned. Så på et kamera, som puljen holder (GUI&#x27;en eller `cam-connect`), et array-medlem eller et kamera, som en baggrundsopgave holder (et kørende `lattice continuous`, `multi-stream` eller `align-stream`, enten sit eget eller en anden klients), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, et-shot `white-balance` og `presets --apply` afviser med HTTP-kode 409 `CAMERA_HELD`, idet de angiver indehaveren og frigivelseskommandoen (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>` eller for en opgave dens id og hvem der startede den) og afslutter `1`. Det samme gælder et kamera, som en anden kommando bruger lige nu (en `multi-test` eller `multi-capture`, der åbner sine kameraer, endnu en engangsbetjening), for hver kommando her: fejlen beder om at vente på, at den pågældende kommando er færdig. `lattice power` uden `--sleep` læser gennem indehaverens eget håndtag, hvor der er et (puljen, en matrix, et jobs åbne kamera), og afvises, hvis et kamera bruges af en anden kommando. Hvert `CAMERA_HELD`-svar indeholder koden i både `code` og `error_code`. For de kommandoer, der åbner alle kameraer, se [Multi-Camera (Transient Sessions)](#multi-camera-transient-sessions).

##### `array-connect`-indstillinger

| Flag | Standard | Beskrivelse |
| --- | --- | --- |
| `--serials SN1,SN2,…` | automatisk opdagelse af alle LATTICE-kameraer (kræver ≥2) | Det første serienummer er MASTER. Hvis dette udelades, filtreres opdagelsen til LATTICE (M3C / M3M)-modeller, og alle disse tilsluttes. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO-synkroniseringslinje. |
| `--target-fps F` | auto | Master-triggerens affyringshastighed. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Tilpas niveauvælgeren. |
| `--wire-ceiling-mbps MB_PER_S` | auto-detected | **Værtsenhedens vedvarende båndbredde, i MB/s — det tal, som hele array-allokeringen afhænger af.** Sænk det, når arrayet rapporterer GVSP-korrupte rammer: den automatiske værdi er afledt af NIC&#x27;ens annoncerede linkhastighed, som overvurderer USB-adaptere, smalle PCIe-baner og travle delte netværk. Gemmes i projektets array-capture-blok, så en genåbning eller en genforbindelse via CLI/SDK gendanner den. Se [Array-tilstand](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | ingen (fuld sensor) | Hardware-binning (altid gennemsnit). |
| `--width N` / `--height N` / `--pixel-format FMT` | fuld sensor; format pr. kamera | En eksplicit billedstørrelse eller et eksplicit pixelformat for hvert kamera i stedet for valget mellem fuld sensor og format pr. kamera (Mono12 / BayerRG12). |
| `--no-recommend` | fra | Spring netværksanalysetrinnet over (hvert kamera beholder derefter sin nuværende størrelse i stedet for at blive indstillet til fuld sensor). |
| `--no-ptp` | fra | Deaktiver PTP (tidsstempler på tværs af kameraer er derefter **ikke** sammenlignelige). |
| `--packed {auto,on,off}` | `auto` | Stream 12-bit-rammer pakket og udpak dem på denne computer — flere rammer pr. sekund ved fuld sensor, identiske data. `auto` pakker kun, når det er nyttigt, og denne computer har kapacitet til det; `off` pakker aldrig. Se [Pakket 12-bit-transport](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | GVSP-pakkestørrelse for hvert kamera i arrayet. `auto` lader hvert kamera undersøge netværket; `jumbo` = 9000, `standard` = 1500 eller en størrelse i byte fastlåser det. Før version 1.3.0 accepterede `array-connect` dette flag og ignorerede det. |

> **Fuld sensor som standard.** Uden `--width` / `--height` / `--binning` /
> `--pixel-format`, indstiller `array-connect` alle kameraer til fuld sensor — ingen
> binning, ingen ROI-beskæring — og lader billedfrekvensen falde til det, som denne computer og
> netværket kan levere, i stedet for at formindske billederne for at opretholde frekvensen. Den
> netværksanalyselinje viser den forventede frekvens, og den næste linje viser
> hurtigere muligheder (binning eller en mindre ROI) med deres forventede frekvenser; spring
> `--binning` eller `--width`/`--height` for at vælge en. Connect foretager aldrig binning eller beskæring
> af sig selv.

##### Pakket 12-bit-transport (`--packed`)

Ved fuld sensorindstilling bestemmes et arrays billedhastighed af, hvor hurtigt hvert kamera kan sende
sine billeder. 12-bit-formater (`BayerRG12`, `Mono12`) overføres normalt som 16-bit
ord; **pakket** transport sender de samme 12 bit i 1,5 byte pr. pixel og
udpakker dem på denne computer, så det tager 25 % kortere tid at sende hvert billede —
ca. **33 % flere billeder pr. sekund ved fuld sensor** (målt: 10,7 → 14,0 fps
på et array med 9 kameraer). **De gemte data er identiske**: samme værdier, samme
16-bit-filer, samme metadata.

- `auto` (standard) pakker kun, når det er en fordel, **og** denne computer har kapacitet til
  at udpakke. Ved opkobling måler backend sin egen udpakningshastighed, antal CPU-kerner,
  aktuel CPU-belastning og netværkskapacitet, så en stationær computer med ledige kerner komprimerer
  ved fuld sensor, mens en travl eller strømbesparende computer streamer ukomprimeret. Små ROI’er og binede billeder giver
  lidt.
- `off` komprimerer aldrig — brug den til at udelukke komprimering under fejlfinding eller på en
  computer, der skal bruge al sin CPU-kapacitet til andet arbejde.
- `on` pakker altid 12-bit-formater.

Connect-udgangen viser beslutningen og begrundelsen (`pakket 12-bit: på (auto:
projiceret 11,0 -&gt; 14,4 fps, …)`); `/api/camera/array/status` rapporterer
`packed_transport` og `packed_decision`. For at indstille en standard for hele maskinen
i stedet for at angive flaget hver gang, skal du indstille `CHLOROS_ARRAY_PACKED12=off` (eller
`on`) i backendens miljø — et eksplicit `--packed on|off` har stadig forrang.
8-bit- og 16-bit-formater pakkes aldrig.

### Én projektpost, alle tilstande

En projektmappe indeholder indstillingerne for alle enheder: `cameras.json` — hvert kameras (`cameras[].settings`, for et enkelt kamera og for hvert array-medlem) samt hvert arrays array-dækkende indstillinger (`arrays[].capture`: format, ROI, binning, triggerfrekvens, wire-budget) — og `sensors.json`, hver lyssensors (integrationstid, billedgennemsnit, cap). Appen, CLI&#x27;en, SDK&#x27;en og MCP-serveren læser og skriver alle de samme filer på samme måde:

- **En enhed, der er tilsluttet under et projekt, gemmes i dette og registrerer alle ændringer deri.** »Under et projekt« betyder med et projekt åbent i backend (appens) eller — fra SDK’et — via en `ChlorosProject`, som angiver sin egen mappe. Et kamera, et array eller en lyssensor, som projektet endnu ikke indeholder, tilføjes til dets filer, når det tilsluttes (`lattice cam-connect`, `array-connect`, `daq pool-connect`, appen, SDK&#x27;et, `chloros-mcp`) sammen med sine indstillinger. En ændring fra en hvilken som helst tilstand (`lattice exposure`, `gain`, `configure --set`, `resolution`, et arraysindstillinger og udløserfrekvens, en lyssensors grænseværdi, appens kontrolelementer) gemmes i projektets filer et øjeblik senere. En eksponering, forstærkning eller et AE-mål, der er indstillet på et kamera, gemmes som fastlåst, så genåbning bevarer indstillingen.
- **Når man åbner et projekt, indstilles enhedernes parametre ud fra projektet**, uanset tilstand: et kamera, en matrix eller en lyssensor, der er tilsluttet uden egne indstillinger, får projektets gemte indstillinger (`lattice cam-connect` / `array-connect` med et åbent projekt, `chloros-cli project connect`, `open_project().connect_all()`, genåbning af appen). Et array, der tilsluttes under et nyt id, opdaterer det array, som projektet har gemt med de samme kameraer; et array med andre kameraer tilføjes ved siden af det.
- **Appens egen gemmefunktion overskriver aldrig en enheds aktuelle indstillinger** med en ældre kopi af dem og bevarer de overskrivninger pr. medlem, som en CLI- eller SDK-kaldende part har gemt.
- **Det pixelformat, du indstiller, er det, som projektet bevarer.** Et format, der er indstillet i Array-indstillinger, via `--pixel-format` eller via `connect_array(pixel_format=)`, inklusive 8-bit, gemmes og genforbindes nøjagtigt. Hvis der ikke er angivet noget, opretter hvert kamera forbindelse med sin 12-bit-standardindstilling (`BayerRG12`, `Mono12` på en monosensor). Et formatnavn angiver dets bitdybde; familien er sensorens, så et farvekamera indstilles aldrig til et monofarveformat eller et monokamera til et Bayer-format, og linjen `[PixelFormat <serial>] ... written` angiver dette, når et format er blevet tilknyttet. Eksportfiler fra 8-bit-optagelse eksporteres som 8-bit, inklusive reflektans: omskaleret og mærket med `PixelScale`.
- En enhed, der er tilsluttet uden at være tilknyttet et projekt, optager intet.
- **En optagelse følger et projekt, der er oprettet efter den; når projektet lukkes, forbliver optagelsen i det pågældende projekt.**Når der oprettes et projekt — et SDK-projekt `create_project()`, `chloros-mcp`’s `create_project` eller appens**Nyt projekt**— eller duplikeres (appens**Dupliker**, som åbner kopien), mens en optagelse stadig skriver til det projekt, der var åbent før, fortsætter optagelsen, og det, den skriver, ender i det nye projekt, på samme sted i det (nedenfor). Åbning af et eksisterende projekt — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, appens**Åbn projekt** — efterlader en optagelse, der stadig kører, i det projekt, den optager til: den afsluttes der, og dens filer føjes til det pågældende projekts filliste (appen stopper alle optagelser, før den åbner et projekt).
  - `array-record` og `array-burst`: videoen eller burst-mappen med alle billeder flyttes derhen i sin helhed, når optagelsen stopper (dets `--duration`, Ctrl+C). Den `Burst dir`, den udskriver, er den nye, og `--build` og `array-build-video` arbejder ud fra den. Hvis drevet afviser en flytning, forbliver optagelsen, hvor den blev skrevet; loggen vedlog ved slutningen angiver årsagen, ligesom HTTP-svaret ved stop (`relocation_error`). De rå billeder fra en stoppet burst og `.daq` føjes til fillisten i det projekt, den ender i; en video (en `array-record`, en samlet video fra en burst) føjes ikke til et projekts filliste.
  - `array-capture --continuous`: Alle optagelser efter ændringen gemmes i det nye projekt; dem, der blev gemt før, forbliver, hvor de er.
  - `daq pool-record` og `project sensor log` (samt appens egen optagelse fra lyssensoren): `.daq` flyttes til det nye projekts`light_sensor`-mappe, når optagelsen stopper, og føjes til det pågældende projekts filliste.

  En optagelse i en mappe uden for det åbne projekt (en `-o` et andet sted) forbliver, hvor den er, og `lattice continuous` og `multi-stream` beholder den mappe, de blev tildelt.
- **I appen placeres optagelserne efter oprettelsen af et nyt projekt i dette.** Oprettelse eller duplikering af et projekt fortsætter alle igangværende optagelser, og hver optagelse, der foretages efterfølgende (Capture All, Start Capturing), gemmes i det nye projekt og føjes til dets filliste. Åbning af et eksisterende projekt i appen stopper først alle disse.
- **Et projekt flyttes ikke, mens det optager.**Flytning af det åbne projekt til et andet arbejdsbibliotek (appens**Indstillinger → Arbejdsbibliotek**; ingen CLI-kommando flytter et projekt, men en HTTP-klient, der sender til `/api/move-project`, kan gøre det) afvises med HTTP-statuskode 409 `PROJECT_RECORDING`, mens en array-optagelse eller burst, en kontinuerlig array-optagelse, et `lattice continuous` / `multi-stream`-job eller en lyssensoroptagelse skriver til det: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` Appen afviser på samme måde, inden den beder om en mappe, mens dens egen »Capture All«, »Start Capturing« eller lyssensoroptagelse kører.

Dette er uafhængigt af, hvordan kameraerne er indstillet ved afbrydelse af forbindelsen (se [Kamerahukommelse](#camera-memory----keep-settings----clear-settings)): et projekt indstiller selv sine kameraer, når det åbnes.

### Kamerahukommelse — `--keep-settings` / `--clear-settings`

Når et array er tilsluttet, overtager dets kameraer arrayets indstillinger, og du kan derefter ændre dem. Når Chloros frigiver kameraerne — `array-disconnect`, afslutningen på en midlertidig kommando (`multi-capture` uden array, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), afslutningen af `lattice continuous`, afslutningen af en `lattice capture` for et kamera, som det ikke har — det sætter hver enkelt tilbage, som den var før tilslutningen: dets udløser- og synkroniseringsledningsføring, PTP- og kameraindstillinger (den fulde liste findes under [Multi-kamera (midlertidige sessioner)](#multi-camera-transient-sessions)).

Hvis kameraerne bruges uden for et projekt, skriver to flag på en hvilken som helst af disse kommandoer til kameraets **eget hukommelse** — det, det starter op med, uanset hvilken software der bruges:

- **`--keep-settings`** gemmer hvert kameras indstillinger, som de var ved afslutningen af sessionen, i dets hukommelse (UserSet2, der udgør opstartsindstillingerne). Kameraerne kører stadig frit (`TriggerMode` `Off`), og synkroniseringsledningsføringen forbliver som den er. Et kamera uden UserSet2 gemmer i UserSet1; denne computer husker det, så Chloros&#x27; array-selvhelbredelse — hvilket nulstiller en UserSet1, der er afveget fra Chloros’s basislinje — lader den være i fred.
- **`--clear-settings`** nulstiller hvert kameras hukommelse til fabriksindstillingerne: Chloros’s basislinje, den som arrayets selvhelbredelse indarbejder — fabriksindstillinger med `TriggerMode`, `Off`, `TriggerSource` og `Line2`, automatisk eksponering, et RGB-kameras kalibrerede hvidbalance bevares — i UserSet1, der udgør opstartsættet.
- De to kan ikke kombineres. Hvert kamera får en linje: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` eller `SN:…: camera memory reset to the factory settings.`; svaret `camera_memory` er det samme for hvert kamera.

Appen tilbyder ikke disse flag.

**Hver udgang sætter kameraerne tilbage.**Den samme tilbageførsel som ved opstart kører på hver udgangsvej, ikke kun ovenstående kommandoer: `cam-disconnect` (og appens kamera**X**); appens**X** på en matrix eller på en hvilken som helst række i et matrixmedlem (en matrix kan ikke køre videre uden et medlem, så hele matrixen frigives); lukning af appen (den giver backend op til 20 sekunder til at sætte hvert kamera, array og kørende `lattice continuous` tilbage, før den stopper det; en kommando, der stadig kører på det tidspunkt — en midlertidig kommando, der åbner sine kameraer eller venter på deres PTP-ure, en engangskommando `lattice capture`, `power`, `white-balance` eller `presets --apply` — stopper ved det næste sikre punkt, sætter først sine kameraer tilbage i op til 8 sekunder, og svarer med HTTP-kode 503 `SHUTTING_DOWN`; en array-forbindelse, der stadig kører — `lattice array-connect`, appens forbindelse — stopper før sit næste kamera og lukker de kameraer, den har åbnet, i op til 10 sekunder, og returnerer HTTP-koden 503 `SHUTTING_DOWN`; og en, der ankommer under nedlukningen, åbner intet: HTTP 503 `SHUTTING_DOWN`); og en CLI-kommando, der afbrydes, eller hvis terminal lukkes (SIGTERM / SIGHUP stopper dens opgave — og lukker kameraerne igen — inden den afsluttes `143` / `129`, ligesom Ctrl+C). En `lattice continuous`, `multi-stream` eller `align-stream`, der afbrydes fuldstændigt (SIGKILL), og som ikke kan stoppe noget, får sin opgave stoppet og sine kameraer sat tilbage af backend-systemet 30 sekunder efter dets sidste check-in. Et backend, der går ned eller afbrydes tvungent, kan ikke nulstille noget; en genstart returnerer hvert kamera til dets hukommelse, og den næste Chloros-forbindelse omkonfigurerer et kamera, den finder aktiveret.**Et projekt indstiller selv sine kameraer.** Når et array opretter forbindelse, mens et projekt er åbent, får hvert medlem de indstillinger, som projektet har gemt for det — en eksponering, forstærkning eller et AE-mål, der holdes fast for det (holdes som dit, så automatisk eksponering lader det være i fred) og dets hvidbalancetilstand (korrektion fra Chloros; kameraets egen hvidbalance forbliver slået fra) — uanset hvad kameraets hukommelse indeholder.

### Smart-AE / Smart-Capture

LATTICE-arrayer kører kontinuerlig AE i baggrunden, så snart de er tilsluttet, men det tager et øjeblik for en nyindstillet scene at konvergere. `array-capture --smart` er den **praktiske løsning**: den venter på, at AE stabiliserer sig på alle kameraer i arrayet, og udløser derefter optagelsen. Brug den, når du skifter scene midt i en session.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Standardindstillingen for stabiliseringspolitikken er konservativ: 5 sekunders timeout, 1,5 sekunders stabilitetsvindue, ±5 % tolerance for eksponeringsspredning. Juster via SDK’et (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`), hvis du har brug for en anden adfærd fra automatiseringen.

#### Sådan fungerer automatisk eksponering

- **Eksponering først, forstærkning sidst.** Når forstærkningen er sat til Auto, forbliver den på 0 dB, indtil eksponeringen har nået den længste tilladte værdi.
- **En manuel eksponering lader forstærkningen forblive på Auto**, indtil du ændrer forstærkningen eller indstiller den til manuel. Når den er tilbage på Auto, er forstærkningens grænse den samme som før.
- **Små højlys gør ikke scenen mørkere.** En spejlrefleks på op til 1 % af billedet ignoreres, og i panelerne på et kendt kalibreringsmål ignoreres op til 0,5 % af et panel.
- **DAQ-eksponeringsloftet gælder kun, mens et kalibreringsmål er i synsfeltet.** Det forhindrer, at et hvidt panel klipper; når der ikke ses noget panel, eksponerer kameraet for scenen.
- **Konstant lys forbliver stabilt, og ændret lys stabiliserer sig på få sekunder.** Ved konstant lys forbliver eksponeringen og dens grænse uændret, og efter en stor ændring i lyset stabiliserer billedet sig inden for få sekunder. En udklipning begrænser ikke længere eksponeringen permanent, og en ændring af ROI finder eksponeringen igen.
- **Lysstyrkemålet forbliver inden for rækkevidde.** Det stiger ikke, mens kameraet er ved sin eksponeringsgrænse, og forstærkningen kan ikke stige, så et lys, der vender tilbage, forårsager ikke en overeksponering af billedet.
- **Et kamera, hvis automatiske eksponering holder op med at reagere, holdes på det niveau, der passer til det lys, det befinder sig i.** I stærk sol kan et kamera falde til sin korteste eksponeringstid og forblive der med et mørkt billede, selvom alle indstillingerne ser normale ud. Chloros overvåger billedet, holder kameraet på det niveau, det havde for det pågældende lys, og slipper det, når lyset ændrer sig. En mørk eller tildækket lyssensor forstyrrer ikke dette.

### Optagelses- og eksportniveauer (standardindstillingen for `all`)

Fra og med denne udgivelse, `lattice capture`, `lattice multi-capture` og `lattice array-capture` **indstillet som standard til `--processing all`** — én gemt fil pr. eksporttype, der gælder for hvert kamera, svarende til GUI&#x27;ens »Capture All«-funktion. Niveauerne er:

| Niveau | Output | Gælder for |
| --- | --- | --- |
| `raw` | Enkeltkanals Bayer (monokameraer: det enkelte bånd) direkte fra sensoren. | Alle kameraer. |
| `debayered` | 3-kanals farve, gemt i **RGB**-kanalrækkefølge (en TIFF-fil med RGB-fotometrisk fortolkning; PNG/JPEG åbnes ligeledes som RGB): kalibreringspakkens mørkeforskydning (DSNU) fjernes fra rådataene, hvorefter OpenCV&#x27;s kant-bevidste demosaic — ingen linearitets-LUT, flat field eller unmix (monokameraer: 1-kanals gråtoner). De debayered-rammer i hukommelsen, som Python-SDK&#x27;et returnerer (`grab()`, `frame_stream()`), er**BGR**, OpenCV&#x27;s rækkefølge; kun den gemte fil er i RGB-format. | Alle kameraer. |
| `radiance` | float32 W/m²/sr/nm via den fulde radiometriske kæde. | Kun multispektrale (M3C/M3M) — **overspringes for RGB-filterkameraer**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Pix4D-klar. | Kun multispektral, og **kun når der er tilknyttet en DAQ, og kameraet er kalibreret**; ellers springes det over. |
| `display` | Fuld GUI-forhåndsvisningskæde (CCM + WB + gamma i henhold til kameraets profil). Gemt i en `preview/`-mappe. `lattice capture` accepterer også navnet `preview` for den; `array-capture` / `multi-capture` accepterer kun `display`. | Alle kameraer. |

Angiv et enkelt niveau for kun at gemme netop det (`--processing debayered`). Når du anmoder om `all`, springes niveauer, der ikke gælder for en given kam, over (og rapporteres), men der opstår ingen fejl — et ikke-tilsluttet eller ikke-kalibreret kamera modtager stadig `raw` / `debayered` / `preview`.

#### Den DAQ-måling, der gemmes ved en optagelse

Hver gang en DAQ er knyttet til kameraet eller arrayet, eller er samlet i en pulje, skrives den nedadrettede måling, der blev brugt ved en optagelse, til en **`.daq`**-fil i undermappen `daq/` i captures-mappen, med dens `.match.json` (hvor langt målingen var fra eksponeringen) og dens `.cameras.json` (hvem målingen tilhører), og rapporteres på en `DAQ:`-linje. Dette gælder for alle niveauer, inklusive råoptagelser og debayered-optagelser, så enhver optagelse kan senere genbehandles til strålingsintensitet, reflektans eller et indeks. Det er den samme fil med samme navn og sidecars, uanset om optagelsen stammer fra GUI&#x27;en, CLI&#x27;en eller SDK&#x27;en. Hvis der slet ikke er nogen DAQ-måling, er der ingen `.daq`, og en `multi-capture` uden tilknyttet array (den midlertidige fallback) gemmer intet.

| Flag | Effekt |
| --- | --- |
| `--no-daq` | Gem ikke målingen for en optagelse, hvis niveauer ikke bruger den (`raw`, `debayered`). Et `radiance`-, `reflectance`- eller `display`-niveau skriver den stadig, fordi genimport kræver den måleværdi, som produktet blev fremstillet med. |
| `--daq-tolerance-ms MS` | Hvor langt fra eksponeringen må målingen ligge og stadig regnes for at være inden for tolerancen (standard 500). Kun vurderingen ændres (`.match.json`&#x27;s `in_tolerance` og `tolerance_ms`, reflektansens `DlsInTolerance`), aldrig den anvendte måleværdi. Se [En DAQ-måling langt fra eksponeringen](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | Den ældre måde at anmode om `.daq` ved en optagelse, der kun indeholder rådata. Det er nu standardindstillingen, og `lattice capture` og `array-capture` accepterer stadig flaget (`multi-capture` gør ikke). `--no-daq` har forrang for det. |

Hvilken DAQ: den, der er bundet til kameraet eller arrayet (GUI’ens valg af lyssensor, et projekts gemte binding eller `chloros-cli daq pool-bind`), ellers den nyeste, som backend’et har. En rå burst (`array-burst`) bevarer sin egen `.daq` pr. individuel aflæsning i burst-mappen som før.

**PNG og JPEG har 8 bit pr. kanal, så de bruger kun `display` / `preview` og indeksprodukterne.** `--format png` eller `jpg` med et niveau, de ikke kan rumme, afvises, før der indsamles noget, idet niveauet og TIFF-formatet, der rummer det, angives:

- `radiance` (float32) og `reflectance` (uint16, `32768` = 1,0) — altid, så `--processing all` (standard) med `--format png`/`jpg` afvises også: brug `--format tiff` eller `--levels display`.
- `raw` og `debayered` fra et kamera, der streamer mere end 8 bit (de 12-bit M3C / Mono12 M3M-rammer) — CLI&#x27;en kan ikke se kameraets bitdybde, så backend&#x27;en afviser disse (HTTP 400). Den enkelte-ramme `lattice capture --processing debayered` henter en 8-bit-ramme til PNG/JPEG og afvises ikke, og det samme gælder en `multi-capture` uden en vedvarende matrix (8-bit debayered).

Alle indfangningsruter svarer på samme måde (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst` samt de tilhørende HTTP-ruter.

### Sådan ser en optagelsesmappe ud

Hver eksporttype placeres i sin **egen undermappe**under `-o`, så en optagelse på flere niveauer aldrig blander typerne. Skitsen viser den**standardoptagelse (`all`) fra ét kamera ved hjælp af `lattice capture`** — flere niveauer pr. kamera, så hver fil har et `_<level>`-suffiks:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Sådan varierer filnavnene:

- **Kamerafelt — serienummer, derefter filter:** hver optagelsesfil navngives efter det kamera, der tog den, som `SN<serial>-<filter>`, hvor `<filter>` er filterdelen af kameraets model: `SN261902142-F450` for et `M3M-L87-F450`, `SN213800234-FRGN` for et `M3C-L87-FRGN`, `SN214701292-FRGB` for et RGB-kamera. Filteret sammenkæder serienummeret med `-`, ikke med `_`, så et navn har de samme `_`-adskilte felter som før (præfiks, dato, klokkeslæt, millisekunder, kamera, niveau). Et kamera, hvis model ikke kan læses (et slettet DeviceUserID), navngives blot `SN<serial>`. Optagelser fra tidligere udgivelser har intet filter i navnet; Chloros læser begge typer.
- **Præfiks:** `lattice capture` navngiver filer `capture_…`; `array-capture`, `multi-capture` med en tilknyttet række, og `project capture` / `burst` / `stream` af en række navngiver dem `sync_<ts>_SN<serial>-<filter>…`.
- **Niveausuffiks kun ved flere niveauer.**Et kamera, der gemmer**flere**niveauer i én optagelse (`all` eller `--levels a,b`), tilføjer `_<level>`. Et kamera, der gemmer**ét** niveau beholder det almindelige navn uden suffiks — `--processing reflectance` skriver `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` — så scripts, der er skrevet til optagelser med et enkelt niveau, fortsat fungerer. Niveauet angives stadig af mappen.
- **Clamp-masker følger produktets navn:** `<product name>_clampmask.tif`. En reflektansoptagelse på et enkelt niveau skriver `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; en `all`-optagelse skriver `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **Filer** følger efter `--format`: standardværdien `tiff` giver `.tiff`, `--format tif` giver `.tif` (clamp-masker er altid `.tif`).
- **En undtagelse:** `lattice capture --processing debayered` (eller `--levels debayered`) tager en hurtig enkelt-rammebane og skriver `capture_<ts>_SN<serial>-<filter>.tiff` direkte ind i `-o` — ingen undermappe.
- **Strømme og bursts** har det samme felt: `lattice continuous` skriver `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` og en rå burst `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` er tidsstemplet for optagelsen, og `<serial>` er kameraets serienummer, så en synkroniseret gruppe deler et
fælles tidsstempel på tværs af kameraerne. **Bemærk den ene asymmetri:** `display`-niveauet gemmes i en mappe
med navnet `preview/`, mens filerne på flere niveauer beholder `_display` i navnet — mappe og suffiks adskiller sig
kun for dette niveau. Ukendte niveauer falder tilbage til en mappe med deres eget navn, og hvis undermappen
ikke kan oprettes, skrives filen til output-roden i stedet for at gå tabt.

**Genbehandling af en captures-mappe:**peg `chloros-cli process` mod**captures-rodmappen**
(`output/`). `process` importerer normalt kun den mappe, du angiver, men når den pågældende mappe ikke indeholder
billeder og har undermapper, går den automatisk ned i hierarkiet — så undermapperne på det pågældende niveau og
`daq/`-undermappen hentes alle på én gang. Hvert niveau i en optagelse importeres som et enkelt billede, hvor
de øvrige niveauer er tilgængelige som tilstande, i stedet for ét billede pr. niveau.

Det fungerer også at navngive en **niveau-undermappe** direkte (f.eks. `output/raw/`). Hvis du gør det, bliver `daq/`
efterladt, så kopier eller peg DAQ-aflæsningen ind sammen med, når duudleder et radiometrisk
produkt fra `raw/` — ellers har tidsstemplet intet at sammenholde med.

**Behandlingen starter altid fra `raw`.** Inden for hver optagelse er råbilledet kilden til behandlingsruten;
`debayered`, `radiance`, `reflectance` og `preview` følger med som visningsmodi, men føres aldrig
tilbage gennem pipelinen. Genbehandling af et afledt produkt ville medføre, at vignettering, CCM og
radiance-beregninger, der allerede er indbagt i dets pixels, ville blive anvendt på ny, så Chloros-funktionen fravælger dette frem for
dobbeltbehandling. To konsekvenser, der er værd at kende:

- Renderingerne `index/` og `composite/` bliver **aldrig** behandlet. De er output, ikke optagelser —
  en NDVI-LUT-rendering har ingen meningsfuld strålingsfortolkning.
- En mappe med optagelser, der eksporteres **uden** `raw` (f.eks. `array-capture --processing reflectance`), har
  ingen gyldig pipeline-kilde. Disse optagelser importeres og vises normalt, men `process` springer
  dem over og angiver dette:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Hvis du virkelig har brug for at sende et afledt produkt igennem — en hub-session, der er optaget med
  `demosaic` aktiveret, eller en ældre mappe — tvinger `--input-level {raw,debayered,processed}` indgangsstedet
  og tilsidesætter springet. Dette flag er den bevidste nødudgang; `auto` (standardindstillingen)
  behandler aldrig en optagelse, der ikke indeholder rådata.

**Kalibreringsmærker: kun på `raw`.** `raw`-filen i en optagelse indeholder kameraets kalibreringspakke: `CalibrationJSON`-payload og de MicaSense-kompatible tags (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), som beskriver sensorens råværdier. Alle andre niveauer i filen (`debayered`, `display`, `radiance`, `reflectance`, `index/`-renderingen) samt hver `lattice continuous` / `multi-stream`-ramme indeholder kameraets identitet (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) samt billedets egen post, `XMP-mapir:FrameJSON` (eksponering, forstærkning, temperatur, pixelformat, optagelsestidspunkt, DAQ-match), da hvert `process`-produkt bærer denne identitet. `process` finder en sådan fils kalibrering ved hjælp af dens serienummer og `CalBundleSha256` i Chloros&#x27; kalibreringscache, hvor pakken downloades én gang, hvis denne computeren ikke har den.

#### Filer lavet af flere kameraer

En fil lavet af bånd fra mere end ét kamera — en multibåndsstak (`align-apply`, `align-stream`, `index --live --save-multiband`), et live-indeks lavet af flere kameraer (`index.tif`, `index_LUT.png`), et array-optagelsesbillede (`composite/`) — angiver navnet på hvert kamera, der indgik i det:

- **Lister pr. kamera, i båndrækkefølge** (kompositbilledet: baggrundslag, derefter forgrund; et indekslag med de kameraer, hvis bånd det stammer fra): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (den kalibreringspakke, som hvert kameras bånd er blevet behandlet med, ellers — en `align-apply` / `align-stream`-stak, hvis bånd ikke gennemgår nogen — den pakke, som kameraet Chloros har gemt i cachen, den, som dets optagelsesfiler er opkaldt efter), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — parallelle lister, hvor element *n* i hver beskriver det samme kamera; `-`, hvor et kamera ikke har nogen værdi (ingen kalibreringspakke, et ukendt objektiv). Hvert kamera udgør **ét** element: hvis dets pakke har flere værdier for et felt — f.eks. en pakkes halogenlampecertifikater — sammenkædes de med `;` (`F-1479-1;F-1480-1;F-1481-1`), aldrig med et komma, så en liste viser én post pr. kamera, uanset hvordan den udskrives.
- **EXIF `SerialNumber`** indeholder de samme serienumre, adskilt med komma (`214000535,214000551`).
- Den indeholder **ingen** identitet for et enkelt kamera (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): disse angiver et enkelt kamera og dets pakke, og `process` vil udlede filens kalibrering ud fra disse. Et indeks beregnet ud fra flere kameras bånd (`index.tif`, `index_LUT.png`) har EXIF-dataene `Model` `LATTICE` (riggen), ikke et enkelt kameras `LATT-…`-model.

En fil med bånd fra **ét** kamera — en TIFF-fil (`--per-band`), et `align-apply --per-camera`-billede, en stak eller et live-indeks fra et enkelt kamera — indeholder det pågældende kameras egne identitetsmærker, ligesom alle andre optagelsesfiler gør. `align-apply`’s stak, båndvise og kameravise filer samt hver `align-stream`-stak og dens båndvise filer indeholder deres gruppes optagelsestidspunkt (referencetkameraets eksponering, UTC), og et monokameras `--per-camera`-ramme er et enkelt plan, ligesom stakken i `--bit-depth`.**Kameraretning (spejling / rotation).** Et kameras retningsindstilling (GUI&#x27;ens Mirror H /
Mirror V / rotation) bestemmer, hvad der gemmes ved en optagelse:

- **`raw` og `debayered` gemmes nøjagtigt, som sensoren læser dem**, uanset indstillingen. En
  Bayer-mosaik, der er drejet eller spejlet, vil udgøre et andet farvefiltermønster, og kalibreringskortene
  (mørk offset, fladt felt) ligger på sensorens eget gitter.
- **`preview`/`display`, `radiance`, `reflectance` og `index/`-renderingen er orienteret** — drejet
  én gang efter alle kalibreringsberegningerne, så de åbnes på den måde, kameraet var indrammet. Deres
  `clamp_masks/…_clampmask.tif`-filer ligger på det samme gitter.
- **Hver optagelsesfil registrerer indstillingen** i XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, med uret, efter
  spejlingerne) samt `ChlorosOrientationApplied` — `false` på rådata og debayered (native pixels),
  `true` på de orienterede produkter. `ChlorosProcessing` angiver de operationer, der er anvendt på den pågældende fils
  pixels (`none`, når det er native). Dette omfatter hvert billede, som `lattice continuous` streamer (
  kameraets egen indstilling, ikke anvendt) og hvert billede i `lattice multi-stream` (identiteten: dens
  medlemsbilleder er native, ligesom ved en array-optagelse). `lattice align-stream` / `align-apply`
  -stakke registrerer hovedkameraets indstilling, uanvendt: deres bånd forvrænges ind i hovedkameraets
  billede, men drejes ikke.
- **`process` anvender den registrerede orientering**, når den behandler et råbillede: mørk offset, flad
  feltkorrektion, demosaikering og adskillelse i sensor-koordinater først, derefter forvrængningen fra array-justeringen og til sidst
  orienteringen til sidst — så forhåndsvisning, radiance, reflektans og indeksprodukterne bliver nøjagtigt som
  dem fra liveoptagelsen, og registrerer `ChlorosOrientationApplied=true`. De ukalibrerede reserveprodukter
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) er orienteret på samme måde efter
  flat-feltet. Det debayered produkt forbliver i sin oprindelige form.
- **Chloros-billedviseren viser en rå eller debayered fil, der er drejet i forhold til dens optagelse** (kun til visning —
  filen ændres ikke), så den ligner dens produkter; miniaturebilledet, visningen i fuld størrelse,
  markørvisningen og indeks-/LUT-sandkassen bruger alle den samme drejning. En fil uden registrering og
  et produkt (der allerede er orienteret) vises som gemt. En sandkasseeksport registrerer den orientering, som dens
  pixel har (`ChlorosOrientationApplied=true`; identiteten for et lag uden en registrering).
- **Optagelser uden registreringen behandles som native** (uændret adfærd). Før version 1.3.0 vendte en
  optagelse med et enkelt kamera, hvor der var indstillet spejling eller rotation, selv sine rå- og debayered-filer;
  Chloros kan ikke skelne mellem en sådan allerede-drejet råfil fra en native fil og gætter ikke, så den
  behandler den som den er gemt. Kalibreringskortene ligger på sensorens eget gitter, så til en sådan
  optagelse skal man bruge en, der er taget med 1.3.0, når kalibreringen er vigtig.
- **Arrays roterer som standard ingen medlemsfiler** (justeringsforvrængningen og `Alignment*` XMP-filen findes i
  kameraernes egne, urotede billeder) og registrerer identiteten. `array-capture --orient` roterer hvert
  medlems produkter efter justeringsforvrængningen: efter **masterens** orientering, da de blev
  forvrænget ind i dens ramme (så medlemmerne forbliver samregistrerede), ellers efter deres eget kameras. Hver
  fil registrerer orienteringen af den ramme, dens egne pixels befinder sig i — en råfil, der afspejler dets eget kameras —
  og en råfil, der indeholder justeringstransformationen, registrerer også `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: masterens, for et produkt, der er forvrænget
  ind i masterens ramme. `process` drejer hvert produkt med den vinkel, der hører til dets pixels:
  den justerede, når den forvrænger RAW-filen (`--array-alignment`, standardindstillingen — produkterne fra den live-justerede
  optagelsens produkter), kameraets eget, når det ikke gør det (`--no-array-alignment` — den live
  ujusterede optagelses). Den samme indstilling findes for `array-burst`, `project capture` / `burst` /
  `stream`, opskrifter og SDK&#x27;en.

### Springede optagelser i blandede filteropstillinger

Når du blander RGB- og multispektrale kameraer i et array, gemmer `array-capture --processing radiance` (eller `reflectance`) de multispektrale billeder og **springer** RGB-kameraerne — strålingsintensiteten pr. Bayer-pixel er ikke relevant for en bredbåndssensor. CLI&#x27;en viser hver gemt fil (med dens eksportniveau), hver `.daq`, der er skrevet, og hver udeladelse eksplicit, så antallet af filer ikke kommer som en overraskelse:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Token for springårsager følger mønsteret `<level>-not-applicable-to-rgb-cam`. Et kamera, hvis kalibreringspakke ikke er indlæst, springer `radiance` / `reflectance` over med `<level>-needs-calibration (connect the camera so its cal pack loads)`, og hvis der ikke er nogen brugbar DAQ-reflektansmåling, springes der over med `reflectance-skipped-no-dls`. Reflektansen kan også springes over med `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, og med `dls-uncalibrated-band-<nm>`, når båndet hovedsageligt ligger uden for DAQ-lyssensorens radiometrisk kalibrerede område (~374–974 nm) — blandt de leverbare SKU&#x27;er er det kun F988, hvis understøttede arbejdsgang er reflektanspanelet.

Brug `--processing debayered` (eller `display`) for at medtage alle kameraer uanset filtertype, eller standardindstillingen `all` for at få alle relevante niveauer pr. kamera på én gang.

### En DAQ-måling langt fra eksponeringen

Hver `.daq`, som en optagelse gemmer, er den DAQ-måling, der svarer til den pågældende eksponering, og `.match.json` ved siden af registrerer, hvor langt de to lå fra hinanden (`gap_ms`) og om det ligger inden for tolerancen (`in_tolerance`, 500 ms, medmindre optagelsen er indstillet til `--daq-tolerance-ms`). En måling uden for tolerancen anvendes stadig, så der går ingen billeder tabt, og den mærkes: Reflektansfilens XMP indeholder `DlsInTolerance=False` og `DlsMatchGapS`. `lattice capture`, `multi-capture` og `array-capture` udskriver den også under den relevante `.daq`, med DAQ&#x27;strømfejl, når man forklarer det:

**Hvordan en måling matches med en eksponering.** En optagelse venter på DAQ&#x27;ens første måling efter eksponeringen, så den kan blande målingerne på begge sider af den (`method` `interpolated`). En DAQ-E sender en måling hvert 0,4 s i dagslys og hvert 1–2 s indendørs, så ventetiden udgør cirka halvandet af disse intervaller for kommandoen, ikke for eksponeringen. Der ventes ikke på en DAQ, der er holdt op med at sende. En DAQ-E-måling er lyset gennemsnitligt over intervallet siden den forrige måling, så den placeres midt i dette interval: `gap_ms` måles fra eksponeringen til midten, og `bracket_stamps_ns` angiver målingernes egne tidsstempler. `array-capture --continuous` venter uden at bremse optagelsen: dens råfiler skrives med det samme, og deres `.daq`, så snart målingen ankommer. Med `--interval` venter hvert gennemløb som en enkelt optagelse.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Når der ikke blev skrevet nogen reflektansfil ud fra den pågældende måling, slutter den første linje i stedet med `saved anyway; its .match.json says in_tolerance: false`.

Indlæsningen afsluttes stadig med `0`: alle filer er blevet skrevet, og det angives, hvad de indeholder. En DAQ, hvis datastrøm er gået i stå, fortsætter med at levere sin sidste aflæsning, så kontroller den med `chloros-cli daq pool-list`, før du stoler på den pågældende indlæsnings reflektans. Backend-systemet genopretter forbindelsen til en samlet DAQ-E, der er holdt op med at levere data (se [En DAQ-E, der holder op med at levere](#a-daq-e-that-stops-delivering)). En måling uden for tolerancen udskriver kun sin `DAQ:`-linje, medmindre det er en bevaret måling (nedenfor).

**En måling, der er bevaret, efter at dens DAQ er forsvundet.** Når ingen DAQ kan levere en måling for en eksponering (den bundne DAQ er utilgængelig, eller ingen poolet DAQ har en, f.eks. efter `daq pool-disconnect`), genbruger indsamlingen den sidste måling, som Chloros havde, og dens `.match.json` beskriver denne måling i forhold til denne eksponering: `method`, `retained`, `gap_ms` fra denne eksponering til de prøver, som målingen stammer fra, `in_tolerance` fra det mellemrum, og `retained_from_epoch_ns`, den eksponering, som målingen først blev matchet med. En registrering, der er foretaget længe efter, at DAQ&#x27;en blev fjernet, udskriver derfor linjerne ovenfor og dem endnu en, som udskrives, selvom målingen stadig ligger inden for tolerancen:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

Dens `.daq` angiver stadig den DAQ, der tog målingen, sammen med den kalibrering og det dæksel, som målingen blev taget under, så en genimport ved, hvilken DAQ, kalibrering og hvilket dæksel der producerede den.

### Filer, der ikke kan skrives

Et kamera, hvis fil ikke kan skrives — et fuldt drev, en tilladelsesfejl — forhindrer ikke de andre kameras filer i at blive gemt. Hver optagelseskommando (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; samt en opskrifts `capture`, `burst` og `stream`) udskriver hver fil, der mislykkedes, til stderr, hvorefter **programmet afsluttes med `1`**, selvom resten blev gemt:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Alle filer, som en optagelse omfatter, er dækket, inklusive den eneste-frame debayered `lattice capture --levels debayered` medregnet. En `.daq`-læsning, hvis `.match.json`-herkomst-sidecar ikke kunne skrives, vises som `[match]` (selve `.daq` blev gemt).

På et fuldt drev vises en gentagen optagelse (`array-capture --continuous` / `--interval`, `project burst` / `stream`, en opskrift `burst` / `stream`) **stopper ved den pågældende gennemgang** i stedet for ikke at skrive noget til det samme drev i resten af kørslen. Enhver anden fejl rapporteres ved hver gennemgang, hvor den opstår, og kørslen fortsætter til slutningen, hvorefter den afsluttes med `1`. `array-capture --continuous` (backendens kontinuerlige indlæsning) viser ingen filer, heller ikke de fejlramte: den tæller dem med i sine status- og oversigtslinjer (`N failed`, `N file(s) failed to write.`) og afslutter `1` — se [Kontinuerlig tilstand](#continuous-mode---continuous).

Optagerne — `array-burst` og `array-record` — rapporterer deres egne fejl. `array-burst` stopper af sig selv, når drevet er fyldt, eller lige før det sker. Begge fortsætter uanset eventuelle andre mislykkede skrivninger. Når en af dem rapporterer en fejl, vises den efter tællerne, og kommandoen **afslutter `1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` i stedet for `Disk full:` betyder, at burst-optagelsen stoppede, før drevet blev fyldt drevet. De rammer, der blev optaget før stop, bevares. `array-burst --build` springer kompileringen over, når der ikke er mere plads, fordi videoerne ville blive gemt på det samme drev. Sørg først for ledig plads, og kør derefter `array-build-video --burst-dir …`. Efter enhver anden fejl kører kompileringen stadig.

En `array-record`, der slet ikke skrev nogen ramme, er også en mislykket optagelse (`Recorder error: no-frames: the combined view published no frame to record …`, afslut `1`) — se [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Optagelsestilstande, optagere og offline-genbehandling

Disse fungerer alle på et **persistent array** (kør først `array-connect`). De afspejler GUI-optagelsespanelet.

### `array-capture`-tilstande

`array-capture` er en enkelt kommando med fire lukker-tilstande samt et sæt eksportindstillinger:

| Tilstand | Flag | Adfærd |
| --- | --- | --- |
| **Enkelt** *(standard)* | (ingen) | Én synkroniseret optagelsesgruppe, derefter afsluttes. |
| **Kontinuerlig** | `--continuous` | Optagelser i træk, indtil `Ctrl+C`, `--count N`-optagelser, `--duration S` eller et fuldt drev. Kører backendens egen kontinuerlige indsamling (GUI&#x27;ens kontinuerlige tilstand) — se [Kontinuerlig tilstand](#continuous-mode---continuous) nedenfor. |
| **Interval** | `--interval S` | Ét gennemløb hvert `S` sekund (målt fra starten af hvert gennemløb), samme grænser. |
| **Hurtigst** | `--fastest` | Rå-kun + den tildelte DAQ-aflæsning + den sammensatte indekskombination; springer strålings-/refleksions-/visningsberegningerne over, så billedet vises hurtigt. Indebærer `--processing raw --force-daq`. Genbehandl de gemte `.daq` til kalibrerede produkter senere. |

Eksportindstillinger (kan kombineres med enhver tilstand; alle deler GUI/SDK-endpoint):

| Flag | Effekt |
| --- | --- |
| `--array-id ID` | Den matrix, der skal indlæses fra (udelades, når der kun er én tilsluttet). |
| `-o, --output DIR` | Outputmappe (standard `output`, under den mappe, hvorfra kommandoen køres). |
| `--format {tiff,tif,png,jpg,jpeg}` | Filformat (standard `tiff`). PNG/JPEG accepterer kun `display` og indeksrenderingen; se [Optagelseseksportniveauer](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Enkelt eksportniveau (`raw`, `debayered`, `radiance`, `reflectance`, `display`) eller `all` (standard). |
| `--levels L1,L2,…` | Eksplicit delmængde af eksporttyper (f.eks. `raw,radiance,reflectance`); **tilside sætter `--processing`**. |
| `--aligned` / `--no-aligned` | **Standard: justeret, ligesom Chloros-appens indlæsning af det samme array er**(dens**Justeret**-indstillingen pr. array, som er aktiveret, medmindre du slår den fra). Hvert medlems ikke-rå eksport (`debayered`, `display`, `radiance`, `reflectance`, indeksrenderingen) bliver forvrænget ind i masterkameraets billede og beskåret til medlemmernes fælles overlapning, så filerne er pixelregistrerede og kommer ud med den samme størrelse, beskæring og de samme `Alignment*`-tags som appens – mindre end sensoren (eller ROI’en) med medlemmernes forskydninger; under en ROI omregnes transformationen til vinduet. `XMP-mapir:AlignmentJSON` angiver `"applied": true` og giver `crop_roi`. Raw forbliver uforvrænget i den oprindelige størrelse, men bærer transformationen i metadata (`"applied": false`). `--no-aligned` gemmer hver fil i den oprindelige størrelse, uforvrænget. `--aligned` er standardindstillingen. Uanset om forvrængningen anvendes eller ej, registrerer hver enkelt fil sit kameras `Alignment*`-transformation, så snart arrayet har en justeringsprofil — undtagen et kamera, som profilen ikke kan justere (udeladt eller tvunget til identitet på grund af en mislykket løsning), og referencekameraet, når intet andet kamera er justeret: disse bærer ingen `Alignment*`-tags. Uden en profil på arrayet gemmes optagelsen ujusteret, og CLI&#x27;en udskriver årsagen på en `Alignment:`-linje, ligesom den gør i alle andre tilfælde, hvor forvrængningen ikke kunne anvendes (elementer med forskellige opløsninger, et array, der ikke er kabelsynkroniseret, en ramme, som profilen ikke kan kortlægges til). Arrayets automatiske justering opretter profilen, så snart eksponeringen er stabil; `align-calibrate` opretter en efter behov. |
| `--orient` | Vend hvert medlems **produkter**— `display`, `radiance`, `reflectance`, `index/`-renderingen og `composite/` — efter kameraets orientering efter justeringsforvrængningen: et produkt, der er forvrænget ind i masterrammen (standard) efter**masterens**orientering, så medlemmerne forbliver samregistrerede; et ikke-justeret produkt (`--no-aligned`, eller en optagelse, hvor justeringen ikke kunne anvendes) efter sit eget kameras.**Standard: fra — hver medlemsfil forbliver i sin oprindelige format**, som det altid har været tilfældet med array-optagelser. `raw` og `debayered` forbliver i deres oprindelige ramme uanset hvad. Hver fil registrerer den orientering, som dens produkter har (se [Kameraorientering](#hvordan-en-optagelsesmappe-ser-ud)); `Alignment*`-XMP-filen forbliver den oprindelige rammetransformation, der anvendes før orienteringen, så `process` (forvrængning, derefter orientering) gengiver det originale produkt. |
| `--index` / `--no-index` | Gem / spring over det kamera-specifikke vegetation-indeks-overlay, hvor et sådant er konfigureret. Standard: gengiv det. |
| `--no-daq` | Spring sidecar-filen `.daq` over, når intet valgt niveau bruger DAQ (et udtræk, der kun indeholder rådata). Som standard gemmes målingen, så billederne kan genbehandles til reflektans/indeks offline. `--force-daq` accepteres stadig (standard). |
| `--daq-tolerance-ms MS` | DAQ-matchtolerancen for disse optagelser (standard 500 ms). |
| `--smart` | Vent på, at AE stabiliserer sig på alle kameraer, før der udløses (se [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF-pixelkomprimering. `none` (standard siden 1.3.0; 1.2.0 skrev `deflate`) = ukomprimeret — sensorens pixels præcis som de læses, ~6,3 MB pr. billede i fuld opløsning og den hurtigste vedvarende hastighed; `deflate` = tabsfri zlib L1 + vandret prediktor, ~4,1 MB pr. billede, langsommere. Begge er tabsfrie og læses identisk ved import. |

> **TIFF med enkelt-skrivning + modellen med vedvarende hastighed.**Hver fil skrives**én gang**: pixels, XMP-pakken, IFD0 Make/Model/CameraSerialNumber og EXIF-tagsene `ExposureTime` / `SerialNumber` i ét gennemløb, uden at noget overskriver det bagefter. `--continuous` indfanger på hinanden følgende synkroniserede grupper — hver indfangning tager gruppen efter den forrige, herunder en, der blev offentliggjort, mens den pågældende indfangning stadig blev gemt — så den kører med arrayets hastighed. DEFLATE holder Python-GIL, så komprimerede skrivninger**ikke**paralleliseres på tværs af skrivetrådene pr. kamera; vedvarende optagelse i fuld opløsning ved sensorhastighed kræver ukomprimerede skrivninger (standardindstillingen)**og** en NVMe-klasse-disk (~600 MB/s vedvarende for et array med 9 kameraer ved ~10 fps). Den samme indstilling vises som `compression` på `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Kontinuerlig tilstand (`--continuous`)

`array-capture --continuous` (uden `--interval` eller `--smart`) overdrager opgaven til **backendens egen kontinuerlige optagelse** — den session, hvor GUI&#x27;ens kontinuerlige tilstand kører. Backend&#x27;en gemmer selv hver synkroniseret gruppe med to optagelser i gang og ingen anmodning fra CLI&#x27;en mellem grupperne, så den kan følge med et hurtigt array, hvor en anmodning pr. optagelse halter bagefter (målt til 73–77 optagelser/sek. på et array med 9 kameraer ved et ROI på 512 pixel). Hver eksportindstilling ovenfor gælder for hver optagelse.

- **Begrænsninger:** `--count N` stopper efter N optagelser (én optagelse = én synkroniseret gruppe: alle medlemmers filer); `--duration S` efter S sekunder. Uden nogen af delene kører programmet, indtil `Ctrl+C`, drevet er fuldt, der opstår en fejl, eller arrayet lukkes ned. `Ctrl+C` stopper backend-sessionen og hvorefter oversigten udskrives.
- **Status** hvert 2. sekund, med `, N failed` og `, N groups skipped` tilføjet, når værdien er forskellig fra nul (en oversprungen gruppe er en, som arrayet har offentliggjort, men som ingen indsamling har taget):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Ingen oversigt pr. fil.** Kørslen afsluttes med én oversigtslinje — indfangninger, hastighed, filer og outputmappen — efterfulgt af `N file(s) failed to write.` / `N group(s) skipped.`, når værdien er forskellig fra nul:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Stopper** ved et fuldt drev med `Continuous capture stopped: the drive is full.` og ved en fejl eller hvis arrayet forsvinder med `Continuous capture stopped: <reason>.` (begge på stderr).
- **Afslutningsstatus:** `0`, når der er gemt optagelser, og alle filer er skrevet, inklusive `Ctrl+C`; `1`, hvis der er en fil, der ikke kunne skrives, et fuldt drev, en fejl, hvis arrayet forsvinder, eller hvis der slet ikke er nogen optagelse.
- **Microsoft Defender:** når Defender bruger en CPU-kerne eller mere på at scanne de filer, som optagelsen skriver, udskriver kørslen én gang på stderr: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Se installationsprogrammets [Systemindstillinger](#windows-exe).
- Én kontinuerlig indsamling pr. array ad gangen; en anden afvises med `a continuous capture is already running for this array`. Via HTTP er sessionen `POST /api/camera/array/continuous/start` (`/api/camera/array/capture`-brødteksten plus `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` og `POST /api/camera/array/continuous/stop`.

**Interval**(`--interval S`, med eller uden `--continuous`) og**`--continuous --smart`** bevarer løkken pr. anmodning: én indsamlingsanmodning pr. gennemløb — på hvert planlagt tidspunkt for Interval, efter at AE er afsluttet for `--smart` — hvor filerne for hvert gennemløb er angivet som i Single-tilstand, med en `Done — N capture pass(es).`-linje i slutningen, og `--count N` tæller gennemløbene.

### `array-record` — kombineret indeks for video/GIF (overvågningskvalitet)

Optager arrayets **live-visning med kombineret indeks** til en `.avi` (og eventuelt en `.gif`). Når GUI&#x27;ens kombinerede visning er åben, optager den den visning, som den vises. Hvis der ikke er nogen åben, gengiver backend&#x27;et selv visningen: det indeks, du overfører med `--expr`, eller som standard arrayets NDVI, når arrayet er justeret. Standard-NDVI&#x27;en bruger de røde og NIR-bånd, der ligger tættest på 660 nm og 850 nm, ligesom live-visningen gør. En GUI-visning, der åbnes under optagelsen, optages, så længe den forbliver åben. Den aflæser fremskridtet hvert 2. sekund og stopper ved `--duration`, `Ctrl+C`, eller når optageren afslutter sig selv.

En optagelse, der ikke skriver nogen ramme, mislykkes: optageren rapporterer `no-frames: …` (arrayet kan stadig være i færd med at justere eller stabilisere eksponeringen), og kommandoen **afsluttes med `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Flag | Standard | Beskrivelse |
| --- | --- | --- |
| `--array-id ID` | kun array | Målarray (udelad, hvis der kun er tilsluttet ét). |
| `-o, --output DIR` | `output` | Outputmappe. En relativ sti er under den mappe, hvorfra du kører kommandoen fra. |
| `--fps F` | `10` | Optagelsesfrekvens. |
| `--duration S` | indtil Ctrl+C | Automatisk stop efter `S` sekunder. |
| `--gif` | off | Skriv også en animeret GIF. |
| `--gif-only` | fra | Skriv kun en GIF (ingen `.avi`). |
| `--expr EXPR` | arrayets NDVI | Indeksudtryk, der skal registreres, når der ikke er nogen GUI-visning åben, i arrayets båndnavne (f.eks. `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; se [båndnavne](#index--vegetation-maths)). Et kort farvenavn (`NIR`, `Red`) fungerer kun, når ét bånd i arrayet har det. En åben GUI-visning registreres i stedet som vist. |

### `array-burst` — rå-Bayer-serieoptagelse med høj billedhastighed (analysekvalitet)

Læser direkte fra grab-loopets synkroniserede gruppebuffer — **ingen kalibreringskæde, intet exiftool, ingen live-visning påkrævet** — så den kører med kameraets fulde optagelseshastighed. Skriver rå billeder + et manifest pr. billede + én `.daq` pr. unik DLS-aflæsning under `<output>/bursts/<base>/`. Genbehandl offline (næste kommando), eller angiv `--build` for at gøre det straks ved stop.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Flag | Standard | Beskrivelse |
| --- | --- | --- |
| `--array-id ID` | kun array | Målarray. |
| `-o, --output DIR` | `output` | Output-mappe (burst-data placeres i `<DIR>/bursts/<base>/`). |
| `--duration S` | indtil Ctrl+C | Automatisk stop efter `S` sekunder. |
| `--max-frames N` | ubegrænset | Automatiskstop efter `N` råbilleder. |
| `--orient` | fra | De produkter, som en build genererer fra denne burst, drejes efter kameraets orientering — produkter pr. kamera efter deres eget kameras, det samlede indeks af masterens — som `array-capture --orient` gør. De rå billeder forbliver i deres oprindelige format; seriens `manifest.json` registrerer hvert medlems orientering og indstillingen. |
| `--build` | fra | Efter stop skal serien straks genbehandles (samme som `array-build-video`). |
| `--products …` | `combined:index` | Med `--build`: hvilke videoer der skal oprettes (se nedenfor). |
| `--fps F` | `10` | Med `--build`: billedhastighed (fps) for outputvideoen. |
| `--save-tiffs` | fra | Med `--build`: gem også kalibrerede TIFF-filer pr. frame (hver med sin [unmix-clamp-maske](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on), når en pixel er blevet begrænset; byggeresultatet viser dem som `clamp_masks`). |
| `--gif` | fra | Med `--build`: skriv også animerede GIF-filer. |

### `array-build-video` — offline-genbehandling af en gemt serie

Tidsmatcher hvert råbillede med den nærmeste gemte `.daq`-aflæsning og sender det gennem **den samme strålings-/refleksions-/indekskæde som importpipeline**, hvorved der renderes en eller flere videoer.

`--products` er en kommasepareret liste over `kind:level`-elementer, hvor `kind` ∈ `per_cam` | `combined` og `level` ∈ `radiance` | `reflectance` | `index`. En `level` uden yderligere angivelser (ingen `kind:`) indstilles som standard til `per_cam`. Standardværdien er `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Flag | Standard | Beskrivelse |
| --- | --- | --- |
| `--burst-dir DIR` | (påkrævet) | Sti til burst-mappen (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level`-liste, som ovenfor. |
| `--fps F` | `10` | Antal billeder pr. sekund (fps) for outputvideoen. |
| `--save-tiffs` | fra | Gem også kalibrerede TIFF-filer for hvert enkelt billede sammen med videoen/videoerne. |
| `--gif` | off | Skriv også animerede GIF&#x27;er. |

**En video indeholder ingen clamp-maske** (én billedstrøm, 8-bit). I stedet indeholder byggeresultatet `clamp_fractions`: pr. videoprodukt, hver enkelt frames andel af de pixels, som unmix har begrænset inden for det bånd, som dens R/G/B-kanal indeholder — det `Chloros:Band{R,G,B}UnmixClampFraction`, som den pågældende frames produkt af stråling og refleksion ville blive stemplet med (for det samlede indeks, for hvert enkelt kamera). Kørslen udskriver det dårligste billede pr. produkt på en `Clamp:`-linje. `--save-tiffs` tilføjer maskerne pr. pixel til TIFF-filerne, i `clamp_masks/<level>/` ved siden af TIFF-filernes `<level>/`-mappe.**En afbrudt kompilering efterlader ingen ødelagte videoer.** Hver video skrives som `<name>.partial.avi` og får først sit endelige navn, når den er færdig, så en ældre video med det navn forbliver, indtil dens erstatning er komplet. Kør buildet igen for at erstatte en `.partial.avi`, der er blevet efterladt.

Kørslen udskriver hver eneste video, den har skrevet, derefter eventuelle `Skipped:`-produkter (et anmodet kamera/niveau — eller det kombinerede indeks — der ikke producerede nogen video, med årsagen — f.eks. intet kalibreringsbundt, ingen DLS-aflæsning for reflektans eller intet rød/NIR-båndpar for det kombinerede indeks) samt eventuelle `Note:`. Et RGB-kamera, der er anmodet om radiance eller reflektans, får i stedet sin visningsvideo, hvidbalanceret når kameraets kalibreringspakke er tilgængelig; et kamera, der gengives uden dette, angives i en `Note:`. Den **returnerer en værdi forskellig fra nul**, når et anmodet produkt blev sprunget over eller ikke blev genereret som anmodet (et konfigureret indeksudtryk evalueres ikke offline); de videoer, der blev genereret, skrives stadig og vises på listen.

> **Vælg den rigtige optager.** `array-record` er *overvågningsklasse* — den optager den gengivne live-komposit: GUI&#x27;ens kombinerede visning, eller hvis der ikke er nogen GUI åben, backendens egen NDVI-visning (eller `--expr`). `array-burst` → `array-build-video` er *analyseklasse* — den gemmer rå sensordata med fuld hastighed og rekonstruerer efterfølgende kalibrerede strålings-/refleksions-/indeksvideoer, uden at der kræves en livevisning.

### Mono (M3M) enkeltbåndskameraer

**M3M**-serien er den monokrome søster til Bayer**M3C**: ét smalbåndsinterferensfilter pr. kamera (`M3M-<lens>-F<wavelength>`, f.eks. `M3M-L87-F685`), så sensoren leverer et**enkelt gråtonebånd** uden Bayer-mosaik. Der er intet at demosaicere, ingen krydstale mellem kanalerne at adskille og ingen hvidbalance at indstille — hele farveprocessen til RGB-visning finder simpelthen ikke anvendelse.

Hvad det betyder på CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**registrerer et monokamera og**springer over med en enkeltlinjemeddelelse** i stedet for at sende meningsløse indstillinger. De kører stadig normalt med et RGB-/Bayer M3C-kamera i samme session.
- **Et Bayer-pixelformat angiver dets dybde.** En M3M tilbyder kun monofarveformater, så et Bayer-format fra `--pixel-format` eller `--preset` indstiller monofarveformatet med samme dybde: `BayerRG8` og `--preset high_speed` giver `Mono8`; `BayerRG12`, `high_quality` og `triggered` giver `Mono12`; et pakket navn forbliver pakket (`BayerRG12p` giver `Mono12p`). Filerne og navnet `lattice format` angiver Mono-formatet. En matrix, der blander M3C- og M3M-kameraer, anvender ét format på samme måde: Bayer på hvert M3C, dets Mono-dybde på hvert M3M. Et navn er dets bitdybde: det samme gælder omvendt, så et M3C-kamera placeres aldrig i et Mono-format (`--pixel-format Mono12` er `BayerRG12` på et M3C-kamera, uanset hvilket kamera der er arrayets master), og `lattice format --set` kortlægges på samme måde og udskriver det format, det er indstillet til.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** fungerer stadig — radiance og reflektans er *båndspecifikke* radiometriske kort og er perfekt veldefinerede for et enkelt bånd. Monobilleder indeholder en **identitets**-sensorresponsmatrix (ingen 3×3-separering), så planet passerer uændret gennem kalibreringsberegningerne.
- **Et enkelt monokamera kan ikke generere et vegetationsindeks.**NDVI, NDRE osv. kræver mindst to bånd (f.eks. Red + NIR). For at få et indeks fra monohardware skal man rette**flere** M3M-kameraer mod forskellige bølgelængder, samle dem i én multibåndsstak og beregne indekset *ud fra den*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` åbner selv kameraerne (en [midlertidig session](#multi-camera-transient-sessions)), så tilslut først intet: `--serials`, ellers aktiveres profilens kameraer (se [Indeks / Vegetationsberegninger](#index--vegetation-maths)). Så længe Chloros holder et kamera (GUI&#x27;en, `cam-connect`, `array-connect`) afvises det med HTTP-kode 409 `CAMERA_HELD` og afsluttes `1`; frigiv dem med `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

`--channel`-symbolerne skal **nøjagtigt** matche forudindstillingens kanalnavne (skelner mellem store og små bogstaver; NDVI&#x27;er er med små bogstaver `red`, `nir` — se `--list-presets`), og »band side« skal angive et bånd i den justerede stak (offline-tilstand accepterer også 0-baserede båndindekser, f.eks. `--channel red=0 --channel nir=1`).

Det, der adskiller de forskellige stakke, er tokenet `M3M` i modelstrengen (det forekommer aldrig i en `M3C`-streng), som vises i GUI’en/SDK’et som `is_mono`.

---

## Opsætning og finjustering af værts-NIC (LATTICE-arrays)

LATTICE-kameraer streamer GVSP via værtens Ethernet-adapter, så for arrays med flere kameraer har adapterens **driver**og**modtagelsesringstørrelse** lige så vigtige som forbindelseshastigheden. Forkerte indstillinger vises som en `FRAMES WILL DROP` / `Reduce ROI to enable`-fejl i panelet Array-indstillinger (og i `lattice network-analysis` / SDK&#x27;s `analyze_array_network()`), selv når kameraerne i sig selv fungerer korrekt.

### USB 10GbE-adaptere — Realtek RTL8157 (&quot;Realtek USB 10GbE Family Controller&quot;)

Indstillingerne nedenfor er egenskaber for Windows-driveren. På Linux har en USB-adapter også en begrænsning (400 MB/s på x86_64, 200 MB/s på NVIDIA Jetson) (se [Array fps &amp; burst-model](#array-fps--burst-model)), og kernens socket-modtagelsesbuffere har betydning for alle adaptere — se [Linux-værter](#linux-hosts--socket-receive-buffers).

| Element | Påkrævet værdi | Hvorfor det er vigtigt |
| --- | --- | --- |
| **Driverversion**|**≥ v10.67 (jan. 2026)**, INF `rtump64x64sta.inf` | Den forældede**2016**-driver (v10.65, `rtump64x64.inf`) håndterer nedlukning og fejlchecks forkert med**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**ved nedlukning/genstart/dvaletilstand. Overgangen går i stå (~5 minutters timeout), brugeren tvinger computeren til at slukke, og gentagne uregelmæssige nedlukninger**beskadiger WMI-repositoriet**(PowerShell/værktøjer begynder at fejle med `Invalid class`) og**blokerer USB-stakken** ved næste opstart (NIC kan ikke aktiveres; USB-drev holder op med at blive registreret). Opdater fra realtek.com (eller dongle-leverandøren), før du stoler på korrekte genstarter. |
| **Modtagelsesbuffere**— nøgleord `ReceiveBufferLen` |**256**(driverens maksimum) | NIC&#x27;ens RX-ring. Driverens standardværdi på**32**efterlader kun ca. 0,26 MB brugbar ring — alt for lidt til en multikamera-burst — så array-panelet rapporterer `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` og blokerer forbindelsen. Ved**256**er ringen stor (**~13,5 MB målt på laboratoriets 10GbE-vært**), hvilket giver RX-pipelineen reel kapacitetsreserve til GVSP-burst med flere kameraer. (Om en given konfiguration rent faktisk *opretter forbindelse*, afgøres af to kontroller — den **drain-aware**admittanskontrol og kontrollen af**samlet overtegnelse** — ikke en rå sammenligning mellem burst og ring; se [Array fps &amp; burst-model](#array-fps--burst-model).) |
| **Modtagende URB&#x27;er**— nøgleord `PendingReceives` |**64** (maks.) | USB-anmodningsblokke undervejs; øges sammen med modtagebuffere til absorption af bursts. |
| **Jumbo-ramme** — nøgleord `*JumboPacket` | **9014** | Nødvendigt for 9000-byte GVSP-pakker (6 gange færre pakker pr. ramme end 1500). |

> ⚠️ **En opdatering af NIC-driveren NULSTILLER disse avancerede egenskaber til standardindstillingerne.**Efter opdatering eller udskiftning af adapterdriveren skal du**genanvende** `ReceiveBufferLen=256` og `PendingReceives=64`, ellers vil array-panelet igen blokere adgangen, selvom »intet er ændret i hardwaren«. Dette er den hyppigste årsag til, at et system, der tidligere fungerede, pludselig nægter at oprette forbindelse.

Anvend fra en **administrator-** PowerShell (erstatt med dit adapternavn, f.eks. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` dækker USB 10GbE-adaptere.** Den registrerer nu adaptertypen og indstiller det korrekte nøgleord for modtagelsesringen: `*ReceiveBuffers`→2048 for PCIe-netværkskort (Intel I219, osv.) eller `ReceiveBufferLen`→256 + `PendingReceives`→64 for Realtek **USB** 10GbE-controlleren (som ikke eksponerer `*ReceiveBuffers`). Værdierne begrænses til den maksimale værdi, som den enkelte driver rapporterer (`NumericParameterMaxValue`), så der aldrig skrives en værdi uden for området. Kør det fra en **administrator-terminal**; ligesom enhver registreringsdatabase-baseret justering træder ændringen i kraft efter en genstart af adapteren eller en genstart af systemet. De manuelle `Set-NetAdapterAdvancedProperty`-kommandoer ovenfor er stadig et godt alternativ — de træder i kraft med det samme (genbinder adapteren) uden genstart.

### Linux
-værter — socket-modtagelsesbuffere

På Linux
, herunder NVIDIA Jetson, er kernens socket-modtagelsesbuffere lige så vigtige som netværkskortets modtagelsesring. Ved kerneindstillingen som standard (`net.core.rmem_max` / `net.core.rmem_default` = 212.992 bytes) får en kameras burst af GVSP-pakker stream-socketten til at løbe over, før Chloros
når at læse den. Pakkerne tabes i kernen, mens netværkskortet selv ikke rapporterer noget tab, og et array mister rammegrupper og sænker sin billedhastighed. Målt på et array med 4 kameraer bag en USB 10 GbE-adapter førte en forhøjelse af begge til 32 MB til, at arrayet steg fra 4,96 til 7,52 fps efter opkobling, og en kontinuerlig råoptagelse steg fra 4,7 til 7,6 optagelser/s, uden at der blev tabt pakker i kernen.

- **.deb-filen indstiller dem.** Når pakken installeres, skrives `/etc/sysctl.d/60-chloros-gige.conf` (begge ved `33554432`) og anvendes med det samme. En værdi, der allerede er indstillet højere, forbliver uændret.
- **`lattice network` kontrollerer dem.** Når en af værdierne er lavere, vises `socket receive buffers too small for GigE Vision (…)` under *Fundne problemer*, og backendens log indeholder den samme advarsel ved den første tilslutning af et kamera eller en array.
- **Sådan hæves de manuelt** (indtil næste genstart; pakkens fil anvender dem ved hver opstart):

```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` hæver og opretholder dem også, men kun når backend&#x27;en kører som root. En backend, der kører som din bruger (en, der er startet med `chloros-backend` eller [`chloros-backend@<user>`](#running-the-backend-as-a-service-headless)) kan ikke ændre kerneindstillingerne og udskriver i stedet linjen `sudo sysctl`.

**Den automatiske adapterjustering påvirker kun tilsluttede adaptere.** Når kameraet tilsluttes for første gang, justerer en backend, der kører som root, de kabelforbundne adaptere, hvis modtagelsesring er lille. Den justerer nu kun adaptere med en forbindelse, så en frakoblet indbygget port beholder sin MTU, og den lader modtagebufferne være uændrede. `lattice network` viser ikke længere en frakoblet adaptersMTU som et problem. `lattice network`, `--fix`, `lattice probe` og adapterjusteringen tager kun højde for kablede Ethernet-porte, ikke en Jetsons CAN-bus (`can0`) eller de USB-forbindelser i enhedstilstand (`usb0`, `usb1`), som den tilbyder en værts-pc; en USB-Ethernet-adapter til kameraer tæller stadig med.

### Grundlæggende om netværk (alle LATTICE-forbindelser)

- **Adressering:** link-lokal `169.254.0.0/16` (GigE Vision LLA). Værten tager en statisk adresse `169.254.x.x/16`; kameraer + DAQ-E tildeler sig selv adresser inden for samme interval. Der kræves ingen DHCP/gateway.
- **Pakstørrelse:**foretræk jumbo (9000), men lad auto-proben finde den — den ser forbi kameraets 1500-byte ICMP-begrænsning via en GVSP-probe, så den ender på jumbo, uanset hvor kablet rent faktisk understøtter det.**Hvert kamera måles én gang pr. backend-session og igen, når dets forbindelse ændres (et andet netværkskort, en anden forbindelseshastighed eller en forbindelse, der er gået ned, som det sker ved udskiftning af en switch); ved genopkobling genbruges resultatet. Efter en ændring bag en switch, der forblev oppe, skal backenden genstartes for at måle igen.**En første forbindelse kan lejlighedsvis tage et par minutter ved denne kontrol, og loggen angiver dette, når det sker. Fastlås kun med `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, når du ved bedre end sonden, og foretræk per-kommando frem for permanent: en fastlåsning springer sonden over, så hvis stien faktisk ikke kan bære 9000,**hver** indfangning går i timeout med `SC_ERR_TIMEOUT -1011` (se [Miljøvariabler](#environment-variables)).
- **RX-ringen skaleres med `ReceiveBufferLen`:**ved standardværdien `32` er den anvendelige ring ~0,26 MB (for lille til enhver multi-cam-burst); ved den maksimale værdi `256` er den stor (~13,5 MB målt på laboratoriets 10GbE-vært), hvilket giver reel spillerum. Om en konfiguration opretter forbindelse, afgøres derefter af den drænbevidste adgangs-kontrol**og** den samlede overskrivningskontrol nedenfor — ikke en rå sammenligning mellem burst og ring.

### Array-fps og burst-model

Sådan aflæses panelet »Array-indstillinger« (og `lattice analyze-array` / SDK
’s `analyze_array_network`):

- **Burst summeres pr. kamera i hvert kameras reelle pixelformat.**Mono**M3M**-kameraer streamer**Mono12 (2 B/px)**, eller med `--pixel-format` det Mono-format, der svarer til dets dybde (`BayerRG8` er `Mono8`, 1 B/px);**M3C**-Bayer-kameraer streamer 8- eller 12-bit (M3C-firmwaren udsender automatisk BayerRG12, selv når der anmodes om BayerRG8). Så et billede i fuld opløsning fra 4 kameraer er**~12,6 MB, hvis alle er 8-bit, men ~25 MB med tre 12-bit mono-kameraer**. Projektionen afkoder hvert kameras format ud fra dets model (identitetscache), så burst-dataene svarer til det, kablet rent faktisk overfører — ikke en standardantagelse om BayerRG8.
- **En USB-Ethernet-adapter er begrænset til 200 MB/s uanset, hvad der står på typeskiltet.** Effektivitetstabellen, der omregner en linkhastighed til et vedvarende tal, er afledt af PCIe; et USB-netværkskort angiver sin *Ethernet*-linkhastighed, men er begrænset af USB-bussen og dens driver. En USB 10GbE-dongle opnåede tidligere ~1063 MB/s ”vedvarende” — et tal, der aldrig blev testet — og den deraf følgende hastighedsbegrænsning ødelagde 6–18 % af rammerne, mens den stadig rapporterede en sund mål-fps. USB-tilsluttede netværkskort er nu begrænset til et absolut tal: **200 MB/s** på Windows
og på NVIDIA Jetson, **400 MB/s** på Linux
x86_64 (på Linux
er et kort i en USB 2.0-port begrænset til **30 MB/s**, og specifikationerne angiver, at det skal tilsluttes en USB 3-port) (begrænsningen ligger i værtsmaskinens USB-bus og driver, så den følger ikke specifikationerne; et USB 1 GbE-kort leverer ca. 80 MB/s og påvirkes ikke). På Linux
identificeres adapteren via sysfs (med `ethtool -i` som standard, hvis det ikke lykkes, og `bus-info` som alternativ). Hvert tal er målt: på Windows
ødelagde det samme 4-kamera-array 6–10 % af billederne ved hastigheder over 200 MB/s; på Linux
x86_64 kørte den samme adapter fejlfrit ved alle hastigheder op til 480 MB/s (273 MB/s leveret, kameraernes egen grænse). På NVIDIA Jetson fungerede den samme adapter fejlfrit kun ved 200 MB/s og mistede billeder sporadisk ved højere hastigheder. `wire_ceiling_source` i kapacitetsoversigten angiver dette med ord, og `nic_is_usb` markerer det. Overtag indstillingen på en af måderne med `--wire-ceiling-mbps`.
- **Admittance tager højde for strømforbruget, ikke hele burst-vs-ring.** En samtidig burst behøver kun at passe til den *midlertidige backlog* = `max(0, Σ per-cam arrival − host drain) × emit_window`, ikke hele burstet. På en fast-host / langsom-cam-fabric (en **PCIe**10G-host + 4× 1 GbE-cams: ankomst ≈ 320 MB/s, dræning ≈ 1063 MB/s) tømmer værten hurtigere, end kameraerne fylder op, backlog ≈ 0, så**fuld-res sim-emit** **godkendes**, selvom burst på 25 MB overstiger ringen på 13,5 MB. Sæt de samme fire kameraer bag en**USB** 10GbE-adapter på Windows
, og udtømningen er 200 MB/s, ikke 1063 — ankomsten overstiger den, og tabet viser sig som korrupte rammer snarere end som en lavere billedhastighed. På en 1 GbE-vært får kameraernes DLThr-grænse på 31,25 MB/s ankomsten til at overstige drænet → den **blokerer** korrekt (for *denne* klasse af blokeringer skal ROI reduceres eller binning ≥ 2 anvendes). Admittance er en af **to** forbindelsesportene — den anden er den samlede kontrol af overabonnement nedenfor.
- **Den forventede fps er det langsomste kameras cyklus.**Ved samtidig udsendelse sender hvert kamera sin ramme på samme tid, så arrayet kører i cykler, der svarer til den længste af dets langsomste medlems sensorudlæsning og udsendelsestid (rammebytes ÷ dets udsendelseshastighed: kameraets**adgangsforbindelse**, 1 GbE ≈ 80 MB/s, eller dets andel af værtens båndbreddegrænse, når denne er mindre). Hver eksponering overlapper det forrige billedes udlæsning, så eksponeringen indgår ikke i cyklusen: automatisk eksponering udfylder den periode, som hastigheden giver, hæver derefter forstærkningen, og en mørk scene sænker ikke længere hastigheden (i 1.2.0 gjorde den det). Kun en manuel eksponering, der er længere end perioden, sænker udløsningen (se [Eksponering på et array-medlem](#exposure-on-an-array-member)). Tilføjelse af kameraer bremser ikke arrayet, før deres wire-andele formindskes. Ved seriel udsendelse (`slip-emit-and-capture`) lægges kameraernes udsendelsestider i stedet sammen, og hvert medlems eksponering indgår i fordelingen. Målt ved fuld sensor, 12-bit, på et array med 9 kameraer med en PCIe 10 GbE-vært: 10,7 fps eller 14,0 fps med [pakket transport](#packed-12-bit-transport---packed).
- **Efter en kontinuerlig optagelse angiver fremskrivningen, hvad den har registreret.** En optagelse på 30 sekunder eller mere ved en automatisk hastighed registrerer den hastighed, den stabiliserede sig på ved den pågældende array-størrelse på denne computer; hastighedsregulatorens korte forsøg med en hurtigere hastighed udelades. Fra da af angiver linjen for `array-connect` og panelet »Arrayindstillinger« denne hastighed, når den ligger under modellens, sammen med årsagen til begrænsningen: `clock-bound` (arrayet kørte med den hastighed, det havde fundet frem til, hvor grab-sløjfen ventede på sin trigger-klokke), `loop-bound` (grab-sløjfen havde ikke mere tid tilbage), `capture-bound` (indlæsningen kom bagud og sprang grupper over) eller `measured` (intet angiver, hvilken). Modellens værdi findes i `wire_fps_max`. Arrayet stiger stadig til det, som ledningen kan bære.
- **En array fungerer på ethvert netværk — det er billedfrekvensen, der afgør det.**Hvert kamera får en jævn andel af den**kollisionssikre ledningsgrænse**(`sustained × sim_emit_factor`), synkroniseret med GevSCPD, og alle eksponerer stadig ved den samme udløser. På et langsomt netværk er andelen lille, og billedfrekvensen falder i takt hermed: 8 kameraer i fuld opløsning med 12 bit på 1 GbE uden jumbo-rammer får 52 / 8 = 6,5 MB/s hver, hvilket svarer til ca.**1 fps**(ca. 1,3 fps pakket), hvor en 10G-host kører dem med 11–14. Hvis kameraerne ikke kan synkroniseres så langsomt — find ud af det ved at aflæse hvert kameras GevSCPD-rækkevidde — skifter arrayet til**seriel**udsendelse (`slip-emit-and-capture`): kameraerne skiftes til at sende på kablet ved fuld linkhastighed, udløst det ene efter det andet, så hver ramme stadig ankommer, men medlemmerne**eksponeres ikke samtidigt**. `analyze-array` og connect-udgangen angiver tilstanden; sæt `--force-tier sim-capture-sim-emit` til at afvise fallback. Ramme størrelse spiller ingen rolle i denne deling, så**binning og et mindre ROI øger billedfrekvensen, men ændrer ikke på, om et array passer ind**. Kun en ledning, der ikke kan overføre arrayet, selv ikke serielt, er**overbelastet**— f.eks. langt flere kameraer, end en ledning kan håndtere, eller mange andre kameraer, der streamer på den — og her er symptomet GVSP-pakketab, ikke en lavere hastighed: `analyze-array` nulstiller de opnåelige fps-tal og udskriver `** OVER-SUBSCRIBED**`, forbindelsen oprettes med en højlydt advarsel, og en HTTP
-forbindelse med `pin_resolution: true` (streng tilstand) **afvises** i stedet. Løsninger er færre kameraer, jumbo-rammer fra ende til ende eller et hurtigere netværkskort. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` nedgraderer afvisningen til en kraftig advarsel ved testarbejde — se [Miljøvariabler](#environment-variables).

### Array-tilstand — hvilket undersystem mister rammer

Et tilsluttet arrays `GET /api/camera/array/<array_id>/capability` indeholder en live
`health`-blok, der revurderes i et rullende **10-sekunders** vindue. Den opdeler rammetab
i de to årsager, der kræver modsatrettede løsninger, i stedet for at rapportere én »ufuldstændig«
rate, der ikke specificerer nogen af dem:

| Felt | Hvad det betyder | Hvilket undersystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (pr. seriel port) | Rammen **ankom og var strukturelt defekt**— GVSP-pakketab. |**Netværk**: ledningskapacitet, pacing, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (pr. serienummer) | Rammen **kom slet ikke**— kameraet udløste ikke, eller der kom intet ud af det. |**Udløser / synkronisering**: M8-kabel, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Det dårligste kameras andel for hver. | — |
| `per_cam_rate_pct` | Samlet andel af ufuldstændige data pr. kamera (begge årsager tilsammen). | — |
| `stable_for_seconds` | Hvor længe hvert kamera har ligget under 0,01 %. | — |

Over 5 % logger backend&#x27;en en `[array-health <id>] WARN`-linje, der angiver opdelingen — ved den
første overskridelse, ved et skift i alvorlighedsgraden, en gang i minuttet, mens det varer ved, og én gang, når
det ophører. Den korrupte halvdel skriver `[gvsp-corrupt <SN>]` til backend-logfilen ved det
første hit pr. kamera og årsag, derefter en sammenfatning hvert 60. sekund; appens logpanel viser i stedet linjen
`[array-health <id>]`-linjen i stedet. Hver evaluering ender stadig i backend-logfilen;
tællerne bevæger sig ved hver buffer, uanset hvad der udskrives.

Den samme post angiver det tal, som hele allokeringen er baseret på:

| Felt | Hvad det betyder |
| --- | --- |
| `wire_ceiling_mbps` | Værtens gældende vedvarende båndbredde, MB/s. |
| `wire_ceiling_source` | Hvor tallet stammer fra, i ord — f.eks. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (på Linux
x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) eller `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, når `--wire-ceiling-mbps` (eller GUI&#x27;ens **Wire Budget**-felt) indstiller det. |
| `nic_is_usb` | `true` for en USB-Ethernet-adapter — se USB-begrænsningen ovenfor. |

**Fortolkning:** `gvsp_corrupt_rate_pct` er forskellig fra nul, mens `never_arrived_rate_pct` er 0
betyder, at udløsning og kabelsynkronisering er perfekt, og at 100 % af tabet ligger på netværksstien
— sænk `--wire-ceiling-mbps` og tilslut igen. Det omvendte mønster peger i stedet på
synkroniseringskablet eller udløserledningen.

> **`--target-fps` er ikke afgørende for korrupte rammer.** GevSCPD-pacing skrives
> én gang ved opkobling, så en sænkning af triggerfrekvensen ændrer duty cycle og ikke
> burst-frekvensen for samtidig udsendelse. En målt reduktion af efterspørgslen med 5× gav ingen forbedring;
> at sænke ledningens loft fra 240 til 200 MB/s bragte det samme anlæg fra 10,4 %
> korrupt til 0,00 %.

> **Et kørende array kan ikke omplanlægge sig selv på LATTICE (M3C / M3M) firmware.** Den
> kan ikke selv løse dette; afbryd forbindelsen og tilslut den igen, så forbindelsestidsvælgeren kan
> omplanlægge med den nye grænse.

### Symptom → løsning

| Symptom (Array-indstillinger / tilslutning / `analyze_array_network`) | Årsag | Løsning |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` nulstilles til 32 (typisk efter en driveropdatering) | Indstil `ReceiveBufferLen`→256, `PendingReceives`→64; genåbn panelet (genstart backend, hvis den har cachelagret den gamle ringstørrelse) |
| Genstart/nedlukning hænger; senere `Invalid class` WMI-fejl, NIC kan ikke aktiveres, USB-drev mangler | Gammel Realtek USB 10GbE-driver fra 2016 → BSOD `0x9F` → tvungne nedlukninger | Opdater adapterdriveren til ≥ v10.67 (2026), og anvend derefter ovenstående indstillinger for modtagelsesringen igen |
| Arrayet opretter forbindelse ved fuld sensor, men med en lav billedhastighed | Auto-indstillingen bevarer den fulde sensor og lader hastigheden falde til det, kablet kan klare | Vælg en af de hurtigere indstillinger på listen over forbindelsesudgange (`--binning 2` / `4` eller en lavere `--width`/`--height`), aktiver jumbo-rammer, eller opgrader forbindelsen |
| Arrayet rapporterer en sund mål-fps, men leverer kun en brøkdel heraf; `health.gvsp_corrupt_rate_pct` er ikke nul, `never_arrived_rate_pct` 0 | Værtsenhedens estimerede båndbredde overstiger den faktiske kapacitet (typisk på en USB-Ethernet-adapter, en smal PCIe-bane eller et delt netværk) | Opret forbindelse igen med en lavere `--wire-ceiling-mbps`-værdi, og kontroller tilstandsblokken igen. **Ikke** `--target-fps` — GevSCPD-pacing er fastsat ved opkobling |
| Linux
: et array mister rammegrupper og sænker sin rammehastighed, mens NIC&#x27;en ikke rapporterer om tab; `lattice network` viser `socket receive buffers too small` | Kernelens socket-modtagelsesbuffere er indstillet til standardværdien, så kamerapakker tabes i kernen | Forøg `net.core.rmem_max` / `rmem_default` til 32 MB — se [Linux
-værter](#linux-hosts--socket-receive-buffers). .deb-filen gør dette ved installationen |
| Kameraer mangler i de offentliggjorte grupper; `health.never_arrived_rate_pct` er ikke nul, `gvsp_corrupt_rate_pct` 0 | Udløser-/synkroniseringssti — kameraerne udløses ikke, det er ikke et netværksproblem | Kontroller M8-synkroniseringskablet og `--line`; bekræft, at alle enheder er aktiveret (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` overskredet i `analyze-array`, eller forbindelsesafvisning i streng tilstand (`pin_resolution`; `array over-subscribes the wire`) | Kablet kan ikke håndtere arrayet ved nogen af de hastigheder, kameraerne når, hverken samtidigt eller skiftevis — langt flere kameraer, end kablet kan dele, eller andre kameraer, der streamer på det | Færre kameraer, jumbo-rammer fra ende til ende eller et hurtigere netværkskort. **ROI/binning vil IKKE hjælpe** (delingen er uafhængig af rammestørrelsen). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` tilsidesætter på testbænken (accepterer pakketabet) |
| Arrayet opretter forbindelse i `slip-emit-and-capture` (serielt) på et langsomt netværk | Kameraerne kan ikke køre langsomt nok til at dele kablet samtidigt, så de skiftes; rammerne eksponeres ikke på samme tidspunkt | Jumbo-rammer eller et hurtigere netværkskort genopretter samtidig optagelse; `--force-tier sim-capture-sim-emit` insisterer på det (og mister i stedet pakker) |

---

## `chloros-cli daq`

Kommandoer til spektralsensorer. `pool-*`-kommandoerne er tynde HTTP
-klienter, der styrer sensoren via backendens persistente pool, så de har brug for en kørende backend (se [Start backenden](#start-the-backend)). Backend&#x27;en ejer transporten, så CLI
- og SDK
-scripts deler ét aktivt håndtag i stedet for at kæmpe om den serielle port. `pool-discover` er den eneste måde at finde en DAQ-M&#x27;s BLE MAC på.

> **`chloros-cli daq --help`** (eller `-h` eller `daq help`) viser listen over `pool-*`-underkommandoer (på Linux
sammen med de direkte hardwarekommandoer, som build-processen inkluderer), og `chloros-cli daq <subcommand> --help` (eller `daq help <subcommand>`) angiver de enkelte underkommandoers indstillinger. De dækker opkobling, stream, optagelse af kalibrerede `.daq`-filer, udskiftning af cap-profiler og tilknytning af en DAQ til et kamera samt, for en DAQ-E, dens netværksindstillinger, PTP, genstart og firmwareopdateringer. DAQ’en kan også styres fra Python
med `chloros_sdk.connect_daq_sensor()`, som bruger den samme samlede sti.
>
> **Brug `pool-*`-kommandoerne på alle platforme.** De fungerer på samme måde på Windows
, Linux
og Jetson. På Windows
er er `daq discover` og `daq list` aliaser for `pool-discover`, og enhver anden `daq`-kommando, der ikke er `pool-*` afslutter `1` med `DAQ direct-hardware commands require the full daq package`. På Linux
og Jetson er `daq discover`, `daq list` og de øvrige kommandoer, der ikke er af typen `pool-*` eller `daq`, DAQ-værktøjets direkte-hardwarekommandoer: de åbner sensoren i CLI
’s egen proces, uden for backendens pool (altså ikke på en sensor, som backenden, GUI’en eller et script har), og der accepteres `daq discover`, `--only` og `--timeout`, men ikke `--json`. Denne reference dokumenterer kun `pool-*`-kommandoerne.

### Arbejdsgang ved første tilslutning af DAQ-sensor

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*`-reference

| Underkommando | Formål |
| --- | --- |
| `daq pool-connect` (smart-detect) | Åbn en sensor i backend-puljen. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U på en bestemt seriel port. |
| `daq pool-connect --ble` | DAQ-M via BLE, MAC-adresse scannes automatisk. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M via BLE på en kendt MAC (forudsætter `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E via Ethernet med et kendt værtsnavn eller en kendt IP-adresse (forudsætter `--eth`). |
| `daq pool-connect --eth` | DAQ-E via Ethernet, værten fundet automatisk (mDNS + ARP-fallback; fungerer fra en tom ARP-cache på Windows
og Linux
). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Juster integrationsvindue / AE-tilstand. |
| `daq pool-connect --no-stream` | Opret forbindelse, men start ikke streaming endnu (fortsæt med `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Cap-korrektionsprofil. Standard i backend er `sunshine_cosine`; `none` er en sensor uden dæksel. En [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) ignorerer den (fast diffusor). **Sensorerne leveres uden kappe, så navngiv den kappe, du har monteret:** standarden registreres som antaget (`cap_id_source=auto_default`), og behandlingen advarer om det, da en ubeskyttet DAQ-U under solskinnskorrektionen viser en værdi, der er ca. 30 gange for høj. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Scan hver transport for sensorer, du kan oprette forbindelse til, uden at oprette forbindelse. **Sådan finder du en DAQ-Ms BLE-MAC.** `--timeout` gælder pr. transport (standard 5 s, 1-20). Sensorer, der allerede er åbne i puljen, vises ikke — en tilsluttet DAQ-M holder op med at annoncere — så brug `pool-list` til dem. `usb` viser kun serielle USB-enheder: en computers egne serielle porte (`COM1`, Linux
s `/dev/ttyS*`) kan ikke være en sensor og udelades. |
| `daq pool-list` | Vis alle sensorer i backend-puljen. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Bind et kamera eller et array til en DAQ, ligesom GUI’ens valg af lyssensor gør: dens optagelser bruger, matcher og gemmer den pågældende DAQ&#x27;s måleværdier. `SENSOR` er et sensor-id fra `pool-list` eller den adresse, hvor DAQ&#x27;en kan nås. `--clear` (ikke `SENSOR`) ophæver bindingen, og registreringerne falder tilbage til den nyeste DAQ. Bindingen varer, indtil backend&#x27;en genstarter, eller indtil det pågældende kamera, array eller DAQ afbrydes: en afbrydelse (her fra SDK
&#x27;en eller i appen) ophæver alle bindinger til den hardware, der afbrydes, uanset hvem der har oprettet dem, og medlemmerne af et array følger med arrayet. Kameraet skal være tilsluttet, i det åbne projekt eller tidligere have været åbnet på denne computer, og arrayet skal være tilsluttet eller findes i det åbne projekt; ethvert andet serienummer eller id afvises på grund af navnet, og der oprettes ingen binding. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Frigivelse — uanset hvem der åbnede sensoren, så frigives også en sensor, som et nedbrudt script har efterladt i puljen (eventuelle optagelser på den afsluttes først). En sensor, der ikke er i puljen, har intet at frigive: den udskriver `ID: already released (not in the pool)` og afslutter med `0`; en frigivelse, som backend&#x27;et afviste eller ikke bekræftede, afsluttes med `1`. SDK
: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | De seneste N spektrumrammer (standard 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Genoptag / sæt streaming på pause. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Start / stop en .daq-optagelse. `-o` er som standard indstillet til `~/Documents/DAQ Live View` på backend-computeren (en relativ sti findes under den mappe, hvorfra du kører kommandoen); `--duration` (`-d`, standard `0` = indtil `--stop`). `--confirm-after S` advarer, når der ikke er modtaget nogen måleværdi S sekunder efter start (standard 20; 0 slår funktionen fra); med `--duration` `--heartbeat S` udskriver antallet af målinger hvert S. sekund og advarer, når det holder op med at stige (standard 30; 0 slår det fra). En optagelse på en DAQ-E, der går i stå, fortsætter i samme fil, så snart backend&#x27;en har genoprettet forbindelsen, og `--stop` viser hullerne ([En DAQ-E, der holder op med at levere](#a-daq-e-that-stops-delivering)). Ctrl+C afbryder en `--duration`-optagelse før tid og rapporterer det; uden `--duration` fortsætter optagelsen med at køre efter Ctrl+C (afslut `130`) indtil `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Skift kapacitetskorrektionsprofilen under kørsel. `--cap-id` er påkrævet; `--cap-id none` fjerner kapacitetskorrektionen (en sensor uden kappe). En DAQ-E-S bevarer indstillingen i `as_recorded`, uanset hvad der er indstillet. |
| `daq pool-enable-ptp --sensor-id ID` | Slå PTP til igen for en DAQ-E, hvis PTP er slået fra. Indstillingen gemmes på enheden, så en enhed, der er slukket, forbliver slukket — dens tidsstempler er ikke PTP-synkroniserede — indtil denne kommando køres. `pool-list` markerer en sådan enhed (`!! PTP is switched OFF`) og udskriver denne kommando for den. Chloros
slår aldrig en enhed til igen af sig selv, da den kan være slukket bevidst. |
| `daq pool-info --sensor-id ID [--json]` | En DAQ-E’s egne indstillinger: dens firmware (og om denne Chloros
-udgivelse indeholder en nyere version), dens adresse og hvordan den fik den (DHCP / link-local / statisk), den gemte netværksindstilling, indtil sensoren kører den (en gemt med `--no-reboot` inkluderet), stream ved opstart og PTP. På en sensor, hvis stream ikke når frem til denne computer, læser den sensoren ved en ny forbindelse og angiver dette. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Indstil en DAQ-E til DHCP (`--auto`) eller en statisk adresse (`--mask` standard `255.255.255.0`). Sensoren genstarter for at anvende indstillingen og opretter forbindelse igen med de indstillinger, den havde, og venter op til `--timeout` sekunder (standard 45) på dette; `--no-reboot` gemmer den blot. Se [DAQ-E-enhedsindstillinger](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Om en DAQ-E starter streaming ved opstart uden tilkoblet software (gemt på sensoren). Chloros
starter streamingen uanset hvad. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Genstart en DAQ-E og tilslut den igen med de tidligere indstillinger (venter op til `--timeout`, standard 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Flash DAQ-E-firmwaren, der følger med denne Chloros
-udgivelse, til en sensor, der kører en ældre version, og tilslut den derefter igen (ca. et minut; sluk ikke for den; `--timeout` standard 45 s, før den er klar igen). En sensor, der allerede er opdateret, lades være. |

### En DAQ-E, der holder op med at levere data

En DAQ-E i puljen, der ikke leverer noget spektrum i 20 sekunder, tilsluttes igen af backend-systemet. Dette sker ved at genstarte den switch, der forsyner sensoren med strøm: sensoren genstarter sammen med den og streamer ikke igen, før den får besked på det (medmindre `pool-set-auto-stream --on` er angivet).

- **Den kontrollerer først, at det er den samme sensor.** Den forespørger på den adresse, sensoren havde – uden at ændre noget – om sensorens navn, MAC-adresse og serienummer. Den forespørger også under sensorens `daq-e-<id>.local`-navn, men stoler kun på det svar, når det kommer fra sensorens eget netværkskort (MAC). Intet andet, der svarer, genforbindes eller skrives til.
- **Den genopretter forbindelsen med sessionens indstillinger:** integrationstid, billedgennemsnit, automatisk eksponering og cap-profil, som de var, ikke sensorens standardindstillinger ved opstart. En sensor, der havde PTP slået til og kom tilbage med det slået fra, slås til igen. Derefter genstarter den streamen og computerens modtager til den.
- **Den forsøger igen med stigende intervaller:** straks, derefter efter 5, 10, 20 og 40 sekunder, og derefter hvert 60. sekund, så længe sensoren er væk. Hvert forsøg spørger kun sensoren, hvem den er. En sensor, der svarer, men stadig ikke sender noget efter tre genforbindelsesforsøg, lades i fred og rapporteres (`auto-reconnect GAVE UP`). `pool-stream --start` prøver igen.
- **`pool-list` viser, hvor den står.** Mens sensoren er tavs, angiver dens `STREAM FAULT`-linje, hvad genopkoblingen foretager sig. Linjen ryddes med det første spektrum, og sensoren viser derefter `reconnected automatically N time(s)`.
- **En optagelse fortsætter i samme `.daq`.** Filen markerer hvert hul med to rækker: `event_type` 5 ved det sidste spektrum før hullet og 6 ved det første efter det, hver med kun et tidsstempel. `pool-record --duration` udskriver `!! STREAM LOST …`, mens sensoren er væk, og `stream resumed after …`, når den er tilbage. `pool-record --stop` viser hvert hul og advarer, ligesom den gør ved en tom fil (en optagelse, der ikke indeholder nogen måleværdi, afsluttes med `1`).
- **En optagelse beholder én kalibrering.** Hvis sensoren vender tilbage med en anden kalibrering, mens en optagelse kører, beholder resten af optagelsen den kalibrering, den startede med, og stopadvarslen angiver dette. `pool-disconnect` + `pool-connect` overtager den nye.
- **Dine kommandoer har forrang.** En sensor, som du har brugt `pool-disconnect` på, eller sat på pause med `pool-stream --stop`, genforbindes aldrig, selv ikke når en genforbindelse allerede er i gang.
- **`pool-info` på en tavs sensor** aflæser sensoren ved en ny forbindelse og tilføjer `!! stream NOT reaching this computer`. En sensor, der ikke svarer, rapporteres straks. `pool-list` viser ikke længere en PTP-tilstand for en sensor, der er holdt op med at svare.
- DAQ-U og DAQ-M er ikke omfattet. En DAQ-M genopretter forbindelsen via Bluetooth af sig selv, og en DAQ-U, der er frakoblet USB, kræver `pool-connect` igen.

### DAQ-E-S — fast diffusor og hældningssensor

**DAQ-E-S** er en DAQ-E med to forskelle, der har betydning for softwaren. Alt andet — opdagelse, tilslutning, PTP, optagelse og nedenstående enhedsindstillinger — fungerer nøjagtigt som for en DAQ-E.**Dens diffusor er fastmonteret, så der kræves ingen kap-korrektion.** Diffusoren i Sunshine-stil er indbygget og var monteret på enheden, da fabrikskalibreringen blev målt, så kalibreringen allerede angiver den reelle nedadgående strålingsintensitet. Chloros
indstiller sin kap til `as_recorded` (ingen profil), uanset hvad der anmodes om: en cap opkaldt efter den (`sunshine_cosine`, `fov_*`, `none`) ignoreres med en meddelelse, og dens registreringer gemmes som `cap_id_source=model`, så advarslen om antaget loft udløses aldrig for den. Anvend heller ikke et loft på dens data manuelt: `sunshine_cosine` øverst ville vise en værdi, der er ca. 11 gange for høj.

**Den er udstyret med et 3-akset accelerometer.** Sollysdiffusorens cosinusrespons er kun gyldig, når den er vandret, så en skrå sensor rapporterer en forkert strålingsintensitet — og reflektansen arver fejlen — uden andre symptomer. Hver DAQ-E-S-måling indeholder derfor sin orientering:

| Felt | Betydning |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Vinkler i grader. `None`, når firmwaren markerer vinklen som udefineret (frit fald, en død akse). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Acceleration i milli-g. `mag_mg` langt fra ~1000 betyder, at enheden var i bevægelse, og at vinklerne ikke er meningsfulde for den pågældende prøve. |
| `cal_applied` | Indstillet: Akserne og vinklerne er i diffusorens koordinatsystem (enhedens monteringskalibrering er anvendt). Ryddet: Rå kortkoordinatsystem, som kan vise op til ~2,3° hældning, der ikke er reel. |
| `sample_age_ms` | Alder på accelerometerprøven, da spektret blev sendt. |
| `temp_raw` | Ukalibreret **tendens** for chip-temperatur — kun forskelle, aldrig en termometeraflæsning. |

`daq pool-latest --json` returnerer dem under hver rammes `imu`-nøgle (`null` på en DAQ-U, DAQ-M eller almindelig DAQ-E). En `.daq`, der er registreret fra en DAQ-E-S, gemmer de samme værdier i sine `imu_*`-kolonner (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) ved siden af hvert spektrum, så læseren kan skelne mellem en sky og et væltet stativ. På alle andre modeller er disse kolonner tomme.

Holdningsstrømmen rapporteres separat: `daq pool-list` udskriver `!! ATTITUDE STREAM FAULT (spectra unaffected)`, mens den er nede, og hvor mange fejl den genoprettede sig fra bagefter. En fejl der medfører aldrig et hul i spektraloptagelsen. `daq pool-discover` angiver enheden som `DAQ-E-S` (som `DAQ-E`, hvis den ikke svarer på identitetskontrollen inden for ca. 2 sekunder).

### DAQ-E-enhedsindstillinger

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` uden `--gateway` gemmer ingen gateway** (enhver gateway, der er indstillet tidligere, slettes), hvilket er korrekt for et kameranetværk uden router.
- **Der skal være en statisk adresse på denne computers netværk.** Chloros
afviser en, hvis undernet ikke omfatter denne computers adresse på sensorens netværk: efter genstarten kan intet nå frem til sensoren for at ændre det tilbage. Brug kun `--force`, når denne computer får tildelt en ny adresse eller bliver routet til det nye undernet.
- **En genstart afvises, mens sensoren optager.** Stop først optagelsen (`pool-record --stop`). En optagelse, der blev startet umiddelbart før en genstart, afsluttes af denne, og `pool-set-network`, `pool-reboot` og `pool-update-firmware` afsluttes med en `Warning:`-linje, der angiver optagelsens filnavn (også selvom sensoren derefter ikke vender tilbage).
- **Efter en genstart genopretter Chloros
kun forbindelsen til selve sensoren.** Den venter der, hvor sensoren var – og efter `--auto` også under dens `daq-e-<id>.local`-navn – på et svar med sensorens navn, værtsnavn og model, fra det netværkskort (MAC), den havde, og genopretter forbindelsen til den adresse, der svarede. Et svar under navnet accepteres kun med den pågældende MAC-adresse, så fra en computer, der når sensoren via en router, eller fra WSL2 i dets standard-NAT-netværk, genoprettes forbindelsen ikke til en sensor, som DHCP har flyttet til en ny adresse: kommandoen angiver dette og udskriver kommandoen `pool-connect --eth-host <its address> …`, der genopretter forbindelsen med de indstillinger for lysbegrænsning og eksponering, den havde.
- **`pool-update-firmware` afviser en sensor, hvis firmwareversion den ikke kan aflæse.** En sensor, der tager længere tid end ventetiden om at vende tilbage efter blitzen, er ikke gået i stykker: så snart den svarer, genoprettes forbindelsen med den nye firmwares indstillinger udfyldt, og kommandoen advarer om, at den var langsom.
- **En firmwareopdatering kræver en indgående TCP-forbindelse.** Sensoren opretter forbindelse til denne computer igen for at hente billedet. Chloros
lytter på det netværkskort, der dirigerer trafikken til sensoren (på alle netværkskort, hvis den ikke kan afgøre, hvilket der er tale om), og betjener kun sensorens egen adresse; alle andre forbindelser lukkes uden at blive betjent. I Windows
lader installationsprogrammets `Chloros Backend (TCP-In)`-firewallregel den komme igennem (se [Windows
(.exe)](#windows-exe)); en firewall, der blokerer den, forhindrer opdateringen med fejlkoden `… device never connected back on TCP <port> …`.
- **Under WSL2 skal du bruge spejlet netværk** (`networkingMode=mirrored` i `%UserProfile%\.wslconfig`). I standard-NAT-tilstand er denne computers adresse på sensorens netværk WSL&#x27;s egen, så alle statiske adresser afvises (med undtagelse af `--force`), livestreamen når ikke frem til WSL, og en firmwareopdatering kan ikke gennemføres, fordi sensoren opretter forbindelse tilbage til computeren for at modtage billedet.
- **Sensor-ID’et og værtsnavnet ændres ikke.** Uanset adressen er en DAQ-E stadig `daq-e-<id>` og `daq-e-<id>.local`, så `--eth-host daq-e-<id>.local` kan fortsat finde den.
- **GUI’en har de samme indstillinger**: fanen »Lyssensorer«, sensorens indstillinger (tandhjulet), »Netværk«. GUI’en opretholder sin egen forbindelse til en sensor, så hvis du ændrer en sensor, som GUI’en har åben, fra GUI’en: en genstart fra CLI
&#x27;en afslutter GUI&#x27;ens session med den.
- De tilsvarende SDK
-koder er `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` og `update_firmware()`.

---

## `chloros-cli project`

Åbn, opret forbindelse til og styr et gemt Chloros
-projekt (en mappe med `cameras.json` + `sensors.json` + `project.json`). Alt går gennem backend, så GUI og CLI
&#x27;en viser identisk hardwarestatus.

### Oversigt over underkommandoer

| Underkommando | Formål |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Udskriver projektets enhedsmanifest (kameraer, arrays, sensorer), læser fra dets filer og kontrollerer dets kildefiler (se [Et flyttet projekt](#a-moved-project---relink)). Den åbner ikke projektet i backend: projektet, der er åbent der, samt dets enheder og optagelser, forbliver uændrede. Afslutter `1`, hvis ingen af projektets kildefiler kan findes. |
| `project devices PATH [--reconnect]` | Vis eller kør genopdagelse igen. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Opret forbindelse til alle gemte kameraer / arrays / sensorer. Hvert array opretter forbindelse som `lattice array-connect` gør: op til 600 s, hvor trinene udskrives til stderr (`[array <id>]    <s> s: <step>`; stdout er JSON
-rapporten), og et array, der ikke er tilsluttet inden for den tid, fortrydes — dets kameraer sættes tilbage til den tilstand, de blev fundet i — og rapporteres under `errors`; det forbliver aldrig tilsluttet. `--packed` indstiller hvert arrays pakkede 12-bit-transport (standard `auto`; et projekt gemmer ingen indstillinger pr. array; se [Pakket 12-bit transport](#packed-12-bit-transport---packed)). Hver lyssensor åbnes i backendens DAQ-pulje på sin gemte adresse med sin gemte integrationstid, rammeudjævning, automatisk eksponering og cap: en sensor uden valgt cap får `sunshine_cosine`, en der er gemt uden cap (`none`) forbliver uden cap, og en DAQ-E-S tager aldrig en cap. GUI&#x27;ens DAQ-panel er ikke en del af puljen: luk først en sensor der. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Enkeltoptagelse fra et navngivet kamera eller en array (`--prefix` som standard, `capture`). `--orient` (arrays): arrayet gemmer rådata, som forbliver i oprindelig format — dette angiver, at de produkter, som `process` fremstiller ud fra det, skal orienteres som `array-capture --orient`. Standard: oprindelig. `project burst` / `stream` og en opskrifts `capture` / `burst` / `stream` (`orient: true`) har den samme indstilling. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | N-frame-burst fra et navngivet kamera eller en array (`-n/--count` standard 5; `-i/--interval` sekunder mellem frames, standard 0). Array-serier fjerner dubletter af gentagne synkroniserede grupper (forældelsesvagt), så et array, der kun kører delvist, ikke kan returnere N kopier af ét billede; udskriver resultater pr. iteration. En optagelse, der mislykkes undervejs, stopper burst-optagelsen og afslutter `1` med angivelse af, hvor mange der blev gemt før den. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Optagelse til disk i en løkke: `--fps F` optager et sekund (standard 3), ét billede pr. indfangning fra et kamera, én synkroniseret billedgruppe fra et array (med samme forældelsesvagt som `project burst`), indtil `-n N` indfanger (standard `0` = indtil Ctrl+C, hvilket viser, hvor mange der blev gemt, og afslutter `0`, eller `1`, hvis ingen blev gemt, eller en fil mislykkedes; et Ctrl+C, før `-n N` indfanger, afslutter `130`). En optagelse, der mislykkes undervejs, stopper strømmen og afslutter `1`, idet den angiver, hvor mange billeder der blev gemt før det. `--poll-interval` er forældet og ignoreres: den accepteres stadig, så ældre scripts kører, med en bemærkning på stderr. |
| `project sensor read PATH NAME [--json]` | Seneste spektrumframe. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Optag en .daq-fil til `--seconds` (standard 10). En optagelse uden måleværdier rapporteres på stderr, og  afsluttes `1`. |
| `project run PATH RECIPE.yaml` | Udfør en YAML-/JSON
-optagelsesopskrift. `--dry-run` validerer uden at køre. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Beregn justering for et array — se [flagtabellen nedenfor](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Udskriv den aktuelle justeringsprofil. |
| `project align clear PATH NAME` | Slet den cachelagrede profil. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Juster en slaves transformation (standardindstillingerne `0`, `0`, `0`, `1.0`: ingen ændring). |
| `project align export PATH NAME --to FILE` | Gem profilen som JSON
. |
| `project align import PATH NAME --from FILE [--no-validate]` | Indlæs en gemt profil. |

#### `project align calibrate` Indstillinger

| Flag | Standard | Beskrivelse |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Justeringsmetode. **Disse stavemåder adskiller sig fra `lattice align-calibrate`**, som accepterer de korte former `orb` / `akaze` / `phase`; de to kommandoer kan ikke bruges om hverandre med dette flag. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformér modellen, så den passer. |
| `--frames N` | `1` | Synkroniserede billedøjebliksbilleder til gennemsnittet. |
| `--reference SN` | masteren | Referencemeraets serienummer; alle andre medlemmer forvrænges til det. |
| `--max-features N` | `5000` | Øvre grænse for antal ORB-træk. |
| `--ratio-threshold F` | `0.75` | Lowe&#x27;s forholdstest. |
| `--ransac-threshold-px F` | `3.0` | RANSAC-tærskel for inliers. |
| `--min-matches N` | `15` | **Kvalitetsgrænse** — afvis løsningen, hvis antallet af inlier-match er lavere end dette. |
| `--max-reproj-err-px F` | `4.0` | **Kvalitetsgrænse** — afvis løsningen, hvis den overskrider denne RMS-reprojektionsfejl. |
| `--checkerboard RxC` | — | Brætgeometri for `--method checkerboard`, f.eks. `9x6`. |
| `--name PROFILE` | tom | Profilnavn indlejret i den gemte JSON
. **Ikke array-navnet** — det er det positionelle `NAME`. |

De to kvalitetskontrolpunkter er årsagen til, at en kalibrering kan lykkes med at løse problemet og alligevel
afvise at gemme: en profil, der fejler i et af dem, ville uden varsel registrere alle
efterfølgende optagelser forkert, så den afvises i stedet for at blive gemt.

### Et flyttet projekt (`--relink`)

Et projekts `project.json` navngiver hver kildefil — hvert billede og hver lyssensor-
optagelse — ved hjælp af den fulde sti. Flytter man projektmappen eller hele projektmappen
til et andet drev, henviser disse stier stadig til den gamle placering.

Når man åbner et projekt, retter det selv det, det kan: En fil, der lå i projektets
mappe, søges i den mappe, projektet nu befinder sig i, og alle stier, der findes der,
omdirigeres og gemmes (`project.json` sikkerhedskopieres først som
`project.json.before-relink-<date>-<time>`). Dette sker ved hver åbning – appens,
`chloros-cli project open`, `chloros-cli process` i et eksisterende projekt, SDK
’ets
og `chloros-mcp`’s. En sti omdirigeres kun til en fil, der findes der.

Filer, der lå uden for projektmappen – en hub-session, en import via »Tilføj mappe« –
kan ikke findes på den måde. Angiv navnet på den mappe, de nu befinder sig i:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` tager den mappe, filerne blev placeret i, eller en hvilken som helst mappe over den: hver manglende
fils mappe søges under den ved hjælp af dens efterfølgende mappenavne
(`Hub Projects/<session>/raw/<serial>`, derefter `<session>/raw/<serial>`, ...), og hver
mappe hentes fra det sted, hvor flest af filerne findes. `project open` viser
hvad der blev omdirigeret, hvor mange kildefiler der blev fundet, og de mapper, hvor resten befandt sig
. Nogle mangler stadig: en advarsel, afslut `0` – projektet åbnes med resten. Ingen
fundet: en fejlmeddelelse, der nævner disse mapper og `--relink`, afslut `1`; backend’et
nægter at åbne et sådant projekt for `process` eller `project devices --reconnect` (HTTP
409 `PROJECT_SOURCES_MISSING`).

### Eksempler

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Opskrifts-DSL

`project run RECIPE.yaml` accepterer en YAML- eller JSON-fil, der beskriver en række handlinger:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Understøttede handlinger: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` henter nøglerne `cameras.json`, `settings` — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (de ældre navne `exposure_time`, `exposure_auto`, `gain`, `gain_auto` accepteres som aliaser). En eksponering, forstærkning eller et mål, der er indstillet på denne måde, betragtes som dit eget: Chloros’ automatiske eksponeringsstyring (herunder den DAQ-målte eksponering) lader det være i fred. En manuel eksponering lader forstærkningen være, som den var: i auto-tilstand fortsætter kameraet med at justere forstærkningen (over hele sit område) mod mållysstyrken, indtil du indstiller `gain_db` eller `auto_gain: "Off"`. En senere `auto_exposure: "Continuous"` overdrager eksponeringen tilbage til automatikken, og `target_brightness_auto: true` overdrager en fastsat målværdi tilbage til Chloros&#x27; automatiske mål. Handlingen `burst` kræver `name` (påkrævet), `count` (standard 5), `interval` (sekunder, standard 0), `output`, `format`, og `settings` (samme indstillingsform pr. kamera som `apply`); array-burst-optagelser bruger den samme nyligt synkroniserede gruppe-watchdog som `project burst`.

A `capture`, `burst` eller `stream`-trinnet, er standardindstillingen for `output` `output`. En relativ `output` — inklusive standardværdien — løses i forhold til den mappe, du kører `chloros-cli project run` fra, ikke opskriftfilens mappe eller projektmappen (i 1.2.0 havnede den i backendens installationsmappe).

Kør den:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Hver kørsel kontrolleres først i forhold til projektet.**Hver handling skal have nøjagtigt et af ovenstående verber som nøgle — `- capture: {name: FrontLeft}`, ikke `- action: capture` eller `- type: capture` — med et `name` har projektets `cameras.json` / `sensors.json` et kamera eller en matrix til `apply` / `capture` / `burst` / `stream` og en lyssensor til `sensor`; en `stream` kræver, at `count` &gt; 0, og en `wait` kræver en varighed. En opskrift med en sådan fejl udfører**ingen** handling (`[recipe] error: …`, `[recipe] not run: …`, afslutning `1`), selv med `stop_on_error: false`. Nøgler, som et trin ikke bruger, er en `[recipe] warning:`, ligesom alle arrays eller lyssensorer, som opskriften styrer, og som ikke er tilsluttet i øjeblikket. En opskrift tilslutter aldrig enheder: kør først `chloros-cli project connect PATH`. `--dry-run` udskriver opskriften, derefter de samme fejl og advarsler, og `[recipe] valid: …` / `NOT valid: …`, kører intet og afslutter `1`, når opskriften indeholder en fejl.

---

## Miljøvariabler

| Variabel | Effekt |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Tilpas backend-URL&#x27;en (standard `http://127.0.0.1:5000`) — **respekteres kun af kommandofamilierne `lattice`, `project` og `daq pool-*`** (`lattice network-analysis` bruger i stedet `--backend-url`). Kernekommandoerne (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) fastlåser `http://127.0.0.1:<port>` og ignorerer denne variabel (IPv4-literalet omgår Windows&#x27; `localhost`→`::1` ~2 s pr. anmodning-straf), så de altid retter sig mod den lokale maskine. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Indstilles i backendens miljø. `1` nedgraderer afvisningen ved overtegning — en array-forbindelse med `pin_resolution` eller en [midlertidig multikamera-kommando](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — til en kraftig advarsel, hvor tab af GVSP-pakker accepteres. Kun til testformål — se [Array fps &amp; burst-model](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Indstilles i backendens miljø: `on` eller `off` indstiller standarden til [pakket 12-bit transport](#packed-12-bit-transport---packed) for alle arrays (standard `auto`). En eksplicit angivelse af `--packed` har stadig forrang. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (eller `true` / `yes`) deaktiverer Linux-CLI&#x27;ens [automatiske opdateringskontrol](#chloros-cli-update) ved opstart. |
| `CHLOROS_CLI_MODE` | Indstilles af selve CLI&#x27;en; fortæller backend&#x27;en at aktivere parallelbehandling. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` springer GVSP-fallback-proben over (kun ICMP-resultater). **Dette deaktiverer jumbo, det dæmper ikke blot loggen** — kameraet svarer kun på DF-pings op til 1500 på hver vej, så denne probe er det eneste, der kan registrere jumbo. Sparer ~1 s pr. kamera pr. forbindelse; koster ~1,45× ledningsloftet, hvis netværket *kunne* have håndteret jumbo. SDK&#x27;et advarer, når du indstiller det. |
| `CHLOROS_GVSP_SESSION_REUSE` | Indstilles i backendens miljø: `0` måler pakkestørrelsen ved hver forbindelse i stedet for én gang pr. kamera pr. backend-session. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fastlås GVSP-pakkestørrelsen til N byte; springer sonderingen helt over. Foretræk indstilling pr. kommando (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) frem for at indstille den permanent: en fastlåst størrelse tilpasser sig ikke længere det forliggende netværk, og hvis man fastlåser den til 9000 på en rute, der ikke kan håndtere jumbo-pakker, vil **hver eneste** optagelse gå i timeout med `SC_ERR_TIMEOUT -1011`. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Indstilles i backendens miljø: det maksimale antal sekunder for en [midlertidig multikamera-kommando](#multi-camera-transient-sessions) eller en `lattice capture` for et kamera, som intet holder fast, bruger på at indstille hvert kameras eksponering fra scenen, før det gemmer et billede (standard `5`). `0` deaktiverer denne funktion: hvert kameras egen autoeksponering kører derefter fra det første billede, startende ved dets eksponeringsgrænse. |
| `CHLOROS_LOG_DIR` | Indstilles i backendens miljø: den mappe, hvor den detaljerede logfil gemmes. Standard `%LOCALAPPDATA%\Chloros\logs` på Windows, `~/.cache/chloros/logs` på Linux (`$XDG_CACHE_HOME/chloros/logs`, når dette er angivet). .deb-filens rodtjeneste indstiller `/var/log/chloros`, som kan læses af gruppen `adm` — se [Kørsel af backend som en tjeneste](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Hvor midlertidige filer gemmes. CLI&#x27;en bruger automatisk `/mnt/ssd/tmp`, hvis den findes. |

---

## Afslutningskoder

| Kode | Betydning |
| --- | --- |
| `0` | Succes. |
| `1` | Generel fejl (de fleste underkommandofejl). En optagelseskommando afsluttes også med `1`, hvis en fil, den skulle skrive, ikke kunne skrives, selvom de øvrige blev gemt, og med `array-burst` / `array-record` afslutter `1`, når optageren rapporterer en fejl — se [Filer, der ikke kan skrives](#files-that-fail-to-write). `process` afslutter også `1`, når eksportfilerne fra en kørsel ikke kunne verificeres, eller når et billede gik tabt, selvom de øvrige produkter blev skrevet. |
| `2` | Argumentfejl (et ukendt eller manglende flag; gælder også `time-sync set-priority` uden nogen af disse prioriteter). |
| `130` | Afbrudt af Ctrl+C: enhver kommando, den afbryder — `lattice`, `project`, `daq` og `process` (`process` efter at backend&#x27;et har stoppet kørslen, se [`process`-bemærkninger](#notes)). En kommando, der normalt afbrydes med Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` og `project stream` med `-n 0` — afsluttes i stedet af, hvad den udførte: `0`, når alt var skrevet, `1`, når noget mislykkedes. |
| `143` / `129` | Afbrudt af SIGTERM / SIGHUP (128 + signalet), efter at kommandoens oprydning var kørt: et kørende job blev stoppet, og dets kameraer blev sat tilbage, en behandlingskørsel blev stoppet i backend. En `lattice continuous` eller anden optagelse, hvis normale stop er Ctrl+C, afslutter også `143`, ikke `0`. |

`lattice network-analysis` afslutter `1` for afgørelserne `needs_force_slip` og `error`, og `selftest` afslutter `1`, medmindre alle syv kontroller bestås.

---

## Tips til fejlfinding

- **&quot;Login kræves&quot;** → Kør `chloros-cli login EMAIL PASSWORD` én gang på denne maskine.
- **&quot;Projektmappen er for dyb til denne computer&quot;** → Lange stier i Windows er deaktiveret på denne maskine (Windows-standardindstillingen), så ingen sti må overstige 260 tegn, og produkterne ligger op til ca. 130 tegn under projektmappen. Der blev ikke behandlet noget. Meddelelsen angiver den længste projektmappe, der passer: Brug en kortere (`-o` eller `chloros-cli set-project-folder`), eller aktiver lange stier i Windows og genstart Chloros. Windows-installationsprogrammet tilbyder dette som »Aktiver lange Windows-stier«, der er markeret som standard; ellers skal du bruge gruppepolitik: *Computerkonfiguration &gt; Administrative skabeloner &gt; System &gt; Filsystem &gt; Aktiver lange Win32-stier*.
- **&quot;backend unreachable&quot;** → Start Chloros-skrivebordsappen (Windows), eller start backend selv: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` på Windows (mappen `backend` i din installationsmappe; den findes ikke i `PATH`), `chloros-backend` på Linux. Eller tjek `CHLOROS_BACKEND_URL`, hvis det er fjernbetjent.
- **`lattice`-kommandoer mislykkes med fejlmeddelelsen »LATTICE-kameradrivere ikke fundet«** → Backend&#x27;et kunne ikke indlæse den LATTICE-kameraruntime, som det er installeret med. Alle `lattice`-kommandoer kører på backend; selve CLI&#x27;en indeholder ingen kameradrivere. Geninstaller Chloros: Windows-installationsprogrammet og .deb-filen installerer runtime-miljøet ved siden af backend&#x27;en.
- **Array connect / Array Settings viser &quot;FRAMES WILL DROP&quot; eller &quot;Reduce ROI to enable&quot;** → Værts-NIC&#x27;ens modtagelsesring er for lille (nulstilles ofte til 32 efter en opdatering af NIC-driveren). Se [Opsætning og finjustering af værts-NIC](#host-nic-setup--tuning-lattice-arrays) — indstil `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Maskinen fryser ved genstart/nedlukning, hvorefter WMI `Invalid class` / NIC kan ikke aktiveres / USB-drev mangler** → Forældet USB 10GbE-adapterdriver forårsager `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Opdater adapterdriveren — se [Opsætning og finjustering af værts-NIC](#host-nic-setup--tuning-lattice-arrays).
- **Linux: optagelser tilhører root, så du kan ikke slette dem, og backendens log findes ikke i din `~/.cache`** → Backenden kører som root via `chloros-backend.service`. Skift til `chloros-backend@<user>.service`, som kører det som dig — se [Kørsel af backend som en tjeneste](#running-the-backend-as-a-service-headless). Indtil da findes root-tjenestens logfil i `/var/log/chloros` (kan læses af gruppen `adm`).
- **Linux: En backend-enhed er `failed` med meddelelsen »En anden instans kører allerede« (afslutningsstatus 98)** → En anden backend har allerede porten: Der kan kun køre én ad gangen. Hvis to backend-enheder er aktiveret, skal du beholde den ene: `sudo systemctl disable --now <unit>` til den, du ikke bruger — se [Kørsel af backend som en tjeneste](#running-the-backend-as-a-service-headless). Hvis kun denne enhed er aktiveret, er det en backend, der er startet uden for systemd (ved hjælp af `chloros-backend`, `chloros-cli` eller et SDK-script), der besætter den: Stop den pågældende backend, og kør derefter `sudo systemctl start <unit>`.
- **Jetson-swap-advarsel** → Tilføj filbaseret swap; CLI&#x27;en viser de nøjagtige `fallocate` / `swapon`-kommandoer.
- **&quot;backend not reachable&quot; fra en `lattice`-, `project`- eller `daq`-kommando** → der kører ingen backend: disse kommandoer starter aldrig en. Kør `chloros-cli backend status`, og se [Start backend](#start-the-backend).
- **&quot;`daq net ...` er ikke tilgængelig i Chloros&quot;** (Linux og Jetson) → Nogle af DAQ-E&#x27;s egne indstillinger er reserveret til MAPIR-tjenesten. Netværksindstillinger, genstarter og firmwareopdateringer er `pool-set-network`, `pool-reboot` og `pool-update-firmware`; kontakt MAPIR-support for resten.
- **&quot;DAQ-direkte hardwarekommandoer kræver den fulde daq-pakke&quot;** (Windows) → Brug `pool-*`, som styrer den samme sensor via backend, eller `chloros_sdk.connect_daq_sensor()` fra Python.

---

## Se også

- [Python SDK-reference](sdk-reference.md) — programmatisk ækvivalent til alle CLI-kommandoer.
- [DAQ-lyssensorer](https://mapir.gitbook.io/chloros/daq-light-sensors) i Chloros-manualen — sensorerne, deres kapacitet og kalibrerede rækkevidde, optagelse, DAQ-E-netværk.
- [LATTICE-kameraer](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) i Chloros-manualen — tilslutning, kameraindstillinger, optagelsestilstande, arrays.
- Online-dokumentation: `https://mapir.gitbook.io/chloros/cli`

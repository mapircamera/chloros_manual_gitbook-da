# Chloros Python SDK-reference

**Version:**

1.3.0**Genereret:**

29. juli 2026 kl. 19:19 ·**Revideret:**

6. november 2026**Pakke:** `chloros-sdk` (PyPI)**Målgruppe:** Optimeret til brug af store sprogmodeller (LLM); læsbar for mennesker.**Omfang:** Alle offentlige klasser, funktioner og hjælpefunktioner, der stilles til rådighed af `import chloros_sdk`, med eksempler, der kan kopieres og indsættes, og som dækker billedbehandling, styring af enkeltkameraer, synkroniserede arrays, DAQ-sensorer og projektautomatisering samt `chloros-mcp`-serveren til AI-assistenter.

Hvis du kun har brug for de vigtigste punkter, kan du springe til:
- [Installation og hurtigstart](#installation)
- [Smart-Connect til LATTICE-arrays](#smart-connect-for-lattice-cameras)
- [DAQ-sensorsessioner](#daq-sensor-sessions)
- [Kameraværktøjer](#camera-utilities)
- [Transient-arrays, justeringsprofiler og indekser](#transient-arrays-alignment-profiles--indices)
- [Projektmappe og opskrifter](#project-folder--recipes)
- [Projektautomatisering](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Live-billeder i din egen pipeline](#live-frames-in-your-own-pipeline)
- [MCP-server (AI-assistenter)](#mcp-server-ai-assistants)

> **Ændringer i 1.3.0** (hvad et eksisterende 1.2.0-script bemærker).
>
> **Ændringer, som et 1.2.0-script muligvis skal håndtere:**
>
> - **En udgivelse, som backend'et ikke bekræfter, udløser nu en fejl.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` på en matrix, som sessionen har åbnet, `DAQSensorSession.disconnect()` / `force_disconnect()` på en sensor, den har åbnet, og `ChlorosProject.disconnect_all()` udløser `ChlorosConnectError`, når backend'en afviser frigivelsen eller ikke bekræfter den i tide, fordi enheden muligvis stadig er optaget. I 1.2.0 vendte de stille tilbage. En `with`-blokkes afslutning udløser stadig aldrig en fejl.
> - **En lånt enhed nedlukkes aldrig.** `with attach_array(...)` og et eksplicit `disconnect()` på den lader arrayet køre videre (i 1.2.0 nedlukkede afslutningen af blokken den, selv når GUI'en ejede den); kald `force_disconnect()`, når du ønsker at afslutte den. Tilsvarende efterlader en `connect_daq_sensor()`, der fandt sensoren allerede i puljen – inklusive smart-detect med nul-flag – den åben på `disconnect()` og ved afslutning af dens `with`-blok.
> - **PNG og JPEG tager kun `display` og indeksprodukter.** `capture()` med `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) og et `radiance`- eller `reflectance`-niveau, eller `processing="all"`, udløser `ValueError` før enhver anmodning; en 12-bit `raw` / `debayered` afvises af backend. Se [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` åbner den pågældende sensor eller udløser** `ChlorosConnectError` med en liste over, hvad den har fundet; den falder aldrig tilbage til en anden sensor. Se [Fastgjort transport](#pinned-transport).
> - **`frame_stream()` udløser en fejl i stedet for at gå i dvale:** `FrameFetchError` vil ved en afvisning gentage (en reflektansaflæsning uden nogen form for DAQ), efter 60 sekunder uden ramme og efter 60 sekunder uden ny trigger. Se [Én trigger pr. aflæsning](#one-trigger-per-read--group_id).
> - **En timeout ved aflæsning eller en afbrudt forbindelse udløser `ChlorosConnectError`** fra hvert smart-connect-kald og `analyze_array_network()`, hvor 1.2.0 udløste den underliggende timeout eller forbindelsesfejl.
> - **`connect_array()` venter som standard i 600 s (`timeout`, 180 s i 1.2.0), og en forbindelse, som den giver op på, fortrydes.** Opkaldet sender sin `timeout` til backend'en, som sætter hvert kamera tilbage, som det blev fundet, og ikke opretter nogen forbindelse, når forbindelsen ikke er oprettet i tide; opkaldet udløser `ChlorosConnectError` (HTTP-fejl 504 `CLIENT_DEADLINE_PASSED`). I version 1.2.0 kunne en forbindelse, der blev oprettet umiddelbart efter, at kameraerne var tændt, vare længere end opkaldet og oprette forbindelse til arrayet, efter at fejlen var udløst. `connect_all()` venter på hvert gemt array på samme måde (120 s i 1.2.0). Se [`connect_array()`-signatur](#connect_array-signature).
> - **`connect_array()` bevarer den fulde sensor.** Uden `width` / `height` / `binning` er hvert kamera indstillet til fuld sensor, og billedfrekvensen falder til det, som værten og netværket leverer; 1.2.0 har reduceret eller samlet billederne for at opretholde hastigheden. Se [Synkroniseret array](#synchronized-array--arraysession-smart-prep).
> - **Et Bayer-pixelformat på et M3M-kamera (mono) er dets dybde.** Et M3M tilbyder kun monoformater, og forudindstillingerne benævner dem som Bayer-formater: i 1.2.0 afviste kameraet formatet og beholdt sit eget, mens `settings.pixel_format` navngav det som et Bayer-format. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, et `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` og et array's fælles format indstiller nu Mono-formatet med samme dybde på en M3M: `"BayerRG8"` og `high_speed` giver `Mono8`; `"BayerRG12"`, `high_quality` og `triggered` giver `Mono12`; `cam.settings.pixel_format` navngiver det. Det samme gælder omvendt: en M3C placeres aldrig i et Mono-format, så `"Mono12"` – uanset om det stammer fra en af disse, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` eller et arrays gemte format – giver `BayerRG12` på en M3C, uanset hvilket kamera der er arrayets master. Et `CameraSettings`, som du videregiver, beholder sit navn, så det samme objekt stadig anmoder hvert kamera om den pågældende dybde i sin egen familie. `connect_camera(preset=...)` bevarer forudindstillingens format (1.2.0 indstiller kameraet til 12-bit efter anvendelse af forudindstillingen), og et kamera, der åbnes på en forudindstilling, skriver ikke længere det, det har optaget, til `PRESETS`. Ethvert andet format, som et kamera afviser (et format, der mangler på dets `PixelFormat`-liste, såsom `RGB8` på en M3M eller `BayerRG16` på firmware uden det), navngives heller ikke som sit eget: efter en `LatticeCamera`-forbindelse angiver `apply_settings()` eller `set_format()`, `cam.settings.pixel_format` og `packed_transport` angiver det format, kameraet streamer i, og en udskrevet linje angiver det format, det afviste.
> - **`analyze_array_network()` angiver, hvilken forbindelse der gælder:** `pixel_format=None` (tidligere `"BayerRG8"`) angiver de formater, som forbindelsen indstiller (Mono12 / BayerRG12), og `packed="auto"` angiver beslutningen om pakket transport, så den angivne fps kan afvige fra 1.2.0'. En `pixel_format`, som du sender, projicerer en M3M på den pågældende dybde. Se [Network-Analysis Helper](#network-analysis-helper).
> - **`CameraSession.capture()` returnerer en `CaptureResult`** (stadig en liste over frame-dictionaries) med `.skipped`, `.failed`, `.daq_files` og `.daq_matches`. Et `ArraySession.capture()`-resultat indeholder også `.alignment` (om justeringsforvrængningen blev anvendt, og hvorfor ikke) og `.oriented`.
> - **Array-indfangninger justeres som standard, ligesom appens gør.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` og `ArrayHandle.capture()` / `capture_smart()` forvrænger hvert medlems `debayered`, `display`, `radiance`, `reflectance` og indeksfilerne ind i masterkameraets billede og beskærer dem til medlemmernes overlapning (mindre end sensoren med kameraernes forskydninger, `XMP-mapir:AlignmentJSON` `"applied": true`), så de er de samme filer som Chloros-appens optagelse af den samme scene; `raw` forbliver i sin oprindelige størrelse. Tidligere gemte en optagelse uden `aligned=True` alle filer i den oprindelige størrelse, uden korrektion. `aligned=False` bevarer dette. `chloros-mcp`'s `capture_array` og `start_capture_run` anvender den samme standardindstilling. Se [`ArraySession`-metoder](#arraysession-methods).
> - **`connect_all()` angiver et backend, der afviser SDK'et.** Et kørende backend, der er logget ud eller har et abonnement under Copper, rapporteres under `errors` med sin egen årsag (HTTP 401 / 403), ikke som »ingen Chloros-backend kører«, og hvert `connect_all()`-kald tjekker backenden igen, så det fungerer at kalde den efter at have logget ind på det samme projekt-objekt.
> - **Et relativt `output_dir` fungerer mod et lokalt backend.** Relative stier omdannes til absolutte i forhold til dit scripts arbejdsmappe, før de sendes. I version 1.2.0 løste backenden dem i sin egen installationsmappe, hvilket resulterede i fejlen "Adgang nægtet".
> - **`numpy` er nu et krav** i SDK'et (live-frame-kald returnerer numpy-arrays).
> - **Et eksportniveaunavn, der ikke er et, afvises.** `processing=` og `levels=` accepterer `raw`, `debayered`, `radiance`, `reflectance` eller `display` (eller et alias såsom `preview`; `processing=` samt `"all"`). Ethvert andet navn afvises, før kameraet berøres (HTTP 400 `UNKNOWN_LEVEL`: `capture()` udløser `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); 1.2.0 gemte eller returnerede `raw` for det.
> - **Texture Aware afvises for LATTICE-optagelser.** I et projekt, der indeholder LATTICE-optagelser, udløser `configure(debayer="Texture Aware (Slow, Highest Quality)")` en `ChlorosConfigurationError`, og det samme gør `process()` efter en `configure()`, der anmodede om det før importen. En Texture Aware-værdi, som projektet tidligere har gemt, indstilles til Standard med en `UserWarning`. LATTICE-optagelser får altid standard-demosaic, så 1.2.0 accepterede indstillingen og ignorerede den; Chloros-appen tilbyder den heller ikke til dem. Se [Understøttede værdier](#supported-values).
> - **Kameraer vender tilbage til den tilstand, de blev fundet i — medmindre der er tale om en `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, slutningen af hver midlertidig funktion (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) samt afslutningen af `start_camera_continuous()` sætter hvert kamera tilbage i den tilstand, det var i: dets udløser- og synkroniseringsledningsføring, dets PTP samt alle kameraindstillinger, som sessionen har ændret (eksponering og forstærkning, AE-målet, grænser og dæmpning, hvidbalance, sortniveau, pixelformat, binning og ROI, spejling, billedhastighed). `keep_settings=True` gemmer i stedet hvert kameras indstillinger i kameraets egen hukommelse, så det starter op med disse; `clear_settings=True` nulstiller denne hukommelse til fabriksindstillingerne. Uanset hvad kører kameraerne videre i fritilstand. Et projekt indstiller sine kameraer ud fra sine egne gemte indstillinger, når det opretter forbindelse til dem. Se [Kamerahukommelse](#camera-memory--keep_settings--clear_settings).
> - **Alle tilstande gemmer et projekt på samme måde.** Enhver ændring af et kamera, en matrix eller en lyssensor, der er tilsluttet under et projekt — fra SDK, appen, CLI eller MCP-serveren — gemmes i projektets `cameras.json` / `sensors.json`, og når projektet åbnes i en hvilken som helst tilstand, indstilles enhederne ud fra det; en `ChlorosProject` navngiver sin egen mappe til backend. Tidligere var det kun appen, der gemte projektet. Se [Én projektpost, alle tilstande](#one-project-record-every-mode).
> - **Filer oprettet fra flere kameraer angiver hvert enkelt kamera.** En multibåndsstak, et live-indeks oprettet af flere kamerabånd og et `composite/`-billede fra en array-optagelse indeholder hvert kildekameras serienummer, model og kalibreringspakke som lister (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), og EXIF-listerne `SerialNumber` angiver serienumrene adskilt med komma. Tidligere indeholdt de ingen kamerainformation eller kun det første kameras serienummer. Se [Filer oprettet fra flere kameraer](#files-made-from-several-cameras).
> - **Kun en `raw`-fil indeholder kameraets kalibreringsmærker.** Det MicaSense-kompatible sæt (`BlackLevel`, `RadiometricCalibration`, de-kanals `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`-sættene) samt `CalibrationJSON`-payloaden beskriver sensorens råværdier, så de er kun knyttet til `raw`-optagelsesfilen. Alle andre filer — en optagelsesfil`debayered`-, `display`-, `radiance`-, `reflectance`- og indeksfiler, kontinuerlige optagelser og multi-kamera-stream-rammer samt alle `process()`-produkter — indeholder kameraets identitet (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) og, ved en optagelse, billedrammens egne indstillinger (`XMP-mapir:FrameJSON`: eksponering, forstærkning, temperatur, pixelformat, optagelsestid). I version 1.2.0 indeholdt de også rådatasættet, som et værktøj (Pix4D, Metashape), der anvender det, anvendte på pixels, der ikke er råværdier. Processing finder en sådan fils kalibrering ud fra dens serienummer og `CalBundleSha256`.
>
> **Andre ændringer:**
>
> - **En arrays automatiske billedfrekvens tilpasses det, som arrayet leverer.** En frekvens, der mister mere end 1 % af sine udløsere, sænkes – vurderet ud fra flere lange observationer frem for én enkelt – og en frekvens, som den er blevet sænket fra, forsøges igen fra tid til anden. En frekvens, du selv indstiller, forbliver uændret.
> - **Autoeksponeringen er mere stabil.** Den holder sig konstant ved ensartet lys, stabiliserer sig inden for få sekunder efter en ændring, ignorerer små højlys og får et kamera, der holder op med at reagere i stærkt sollys, til at fungere igen. Se [Sådan fungerer autoeksponering](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` tæller kun dine billeder og optagelser.** En mappe, der indeholder et projekts `camera_cal/`, får ikke længere disse kalibreringsfiler talt med som billeder og optagelser, og et projekt, der allerede har indlæst dem, fjerner dem.
> - **`configure(radiometric_output=...)` gælder.** Den indstiller nu projektets `Export radiance` / `Export reflectance`-produkter; tidligere havde hvert projekts egne produktindstillinger forrang, så `radiance`, `sensor-response` og `all` alle eksporterede strålingsintensitet og reflektans.
> - **`test_array_sync()` starter fra kameraernes tændt-tilstand.** Det tænder kameraernes PTP til kørslen, venter på, at deres ure låser sig fast og stabiliserer sig, før den første udløsning (`ptp_lock`) — hvert kamera låser sig fast i 120 s, ca. 2 minutter fra opstart (`ptp_settle_s=` indstiller det), da et ur, der netop er låst, afviger med op til ~200 us i ca. 2 minutter — og sætter hvert kameras PTP tilbage, som den fandt det. Der ventes ikke igen på et kamera, der stadig er synkroniseret fra en tidligere test. Tidligere lod den PTP være slået fra på et netværk med en enkelt switch, hvor ingen DAQ-E svarede, og læste hvert kameras fritløbende ur som et PTP-ur, der ikke ville synkronisere (»PTP er ikke låst«, ved hvert kørsel).
> - **En `.daq`-optagelse angiver, om dens hætte blev valgt eller antaget.** DAQ-U, DAQ-M og DAQ-E leveres uden hætte, og Chloros kan ikke se, om der er monteret en, så en sensor, der tilsluttes uden en navngivet hætte, får som standard `sunshine_cosine`, en *antagelse*. Hver Chloros-optagelse (skema 1.30) gemmer nu `cap_id_source` ved siden af `cap_id`: `operator`, når du har navngivet en kappe (inklusive `none`), `auto_default`, når den var standard. Ved behandling af en optagelse, hvor kondensatoren blev antaget, og hvor projektets DAQ-kondensator er indstillet til Auto, udskrives en `[daq-dls] WARNING`, der navngiver filen: en ren DAQ-U under solskinnskorrektionen viser en værdi, der er ca. 30 gange for høj. **Navngiv den kapacitor, du har monteret**, når du tilslutter (`connect_daq_sensor(cap_id=...)` eller `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap` eller GUI'ens rullemenu for kondensatorer), eller indstil projektets DAQ-kondensator, så forsvinder advarslen.
> - **Ingen flere pip-konsolvinduer på Windows.** På en pc med Python installeret kørte backend og hver af dens billedarbejdere `pip` én gang ved opstart, og hver åbnede et konsolvindue. Det gør de ikke længere.
> - **Live `.daq`-optagelser overlever et strømsvigt.**En optagelse skrives med en journal og en fuld disksynkronisering ved hver commit bag sensoren, så en langsom disk forsinker filen, men aldrig målingerne. En optagelse, der blev stoppet midt i en commit (strømafbrydelse, afbrydelse), kan efterlade `<name>.daq-journal` ved siden af:**kopier de to sammen.**Chloros ruller journalen tilbage, første gang den læser filen, så optagelsen åbnes med alt, der blev commitet før afbrydelsen (ca. de sidste 3 sekunder går tabt); i 1.2.0 kunne et strømsvigt på det forkerte tidspunkt gøre hele filen ulæselig ("database disk image is malformed").
> - **`DAQSensorSession.record_start()` på en travl disk blokerer ikke længere sensoren**: målingerne sættes i kø og skrives i en separat tråd, og en måling, der ikke kunne skrives, tælles med i `status()`'s `record_errors` (sammen med `record_last_error`).
> - **Direkte `LatticeCamera`:**et billede uden chunk-eksponering/forstærkning rapporterer nu kameraets**reelle** eksponering og forstærkning (fallback-funktionen læste en cachelagret værdi, der kunne forblive fastfrosset i hele et autoeksponeringsforløb), og `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` anvender den samme 5 µs input-deglitch, som Chloros’ kabelslaver får (ufiltreret: en slave udløste billeder uden at der blev sendt en trigger).
> - **Softwareudløste kameraer dropper ikke længere hver anden trigger.** Et kamera i softwareudløsertilstand sætter nu en trigger, der ankommer under dets udlæsning (eller Chloros venter på, at det er klar), i kø, ligesom kabelsynkroniserede kameraer allerede gjorde.
> - **`discover_daq_sensors()` navngiver en DAQ-E-S som `DAQ-E-S`.** Dets `model` var `DAQ-E` for hver Ethernet-sensor; en, der ikke svarer på identitetskontrollen inden for ca. 2 sekunder, vises stadig på listen som `DAQ-E`.
> - **Nyt: `chloros-mcp`, en MCP-server til AI-assistenter.** Claude Desktop, Claude Code, Cursor og andre MCP-klienter kan behandle billedmateriale (oprette et projekt, importere optagelser, vælge indstillinger, behandle, rapportere, hvad kørslen har eksporteret) og betjene LATTICE-kameraer, -arrays og DAQ-lyssensorer: oprette forbindelse, få forhåndsvisning, indstille eksponering og farve, tage billeder, optage. Dens værktøjer dækker det, som denne SDK gør. Den installeres sammen med SDK'en, og Linux-.deb-filen placerer den i `/usr/bin`. Se [MCP-server (AI-assistenter)](#mcp-server-ai-assistants).
> - **Nyt: kamerahjælpeprogrammer**, `chloros-cli lattice`-kommandoer som SDK-kald: farveprofil, mætning/kontrast, højlys, hvidbalance (enkeltbillede, auto, faste forhold), forudindstillinger, filtermodeller, firmwarestatus og -opdatering, strøm (sonde, dvaletilstand, vækning), målkalibrering, netværkskort (kontrol, estimering, reparation), vignetteringskorrektion på filer, en DLS-aflæsning og værtssonden. Se [Kameraværktøjer](#camera-utilities).
> - **Nyt: resten af CLI’en som SDK-kald**, så CLI’en, SDK’en og `chloros-mcp` tilbyder de samme funktioner (CLI’en beholder af sikkerhedsmæssige årsager kun login, konfiguration af en DAQ-E og de direkte DAQ-servere): kameraer, der åbnes med én kommando (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), justeringsprofilfiler (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), indekser (`index_catalog()`, `compute_index()`, `capture_live_index()`), PTP-grandmasteren (`time_sync_status()` og lignende), projektmappen (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) og opskrifter (`run_project_recipe()`, CLI's egen runner). Til lyssensorer: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` og `light_sensor_reflectance()` (to sensorers omgivelses- og objektrefleksion samt indekser); til et kamera: `CameraSession.status()` og `capture_repeated()`; for et array: `ArraySession.capability()` og `set_member_settings()` (et medlems eksponering, forstærkning, ...). Se [Transiente arrays, justeringsprofiler og indekser](#transient-arrays-alignment-profiles--indices), [Projektmappe og opskrifter](#project-folder--recipes) og [DAQ-sensorsessioner](#daq-sensor-sessions).
> - **Forbedrede LATTICE-indeksprodukter** (NDVI og andre indekser) fra `process()` i alle behandlingsmetoder.
> - **LATTICE OCN-kameraer: forbedrede radiance-, reflektans- og indeksprodukter i sollys.** Der anvendes en flådekorrektion efter båndsepareringen — live, i optagelser og i `process()`; RGN-kameraer er uændrede. Se [Radiometrisk output](#radiometric-output-lattice-multispectral-pipeline).
> - **Hver optagelse gemmer den anvendte DAQ-aflæsning, ligesom GUI'en gør.** `CameraSession.capture()` og `ArraySession.capture()` / `continuous()` skriver `.daq` og dets `.match.json`, hver gang en DAQ bindes eller samles i en pulje, herunder en optagelse, der kun indeholder rådata; i 1.2.0 gemmes en rå eller debayered optagelse kun med `force_daq=True`. `daq=False` deaktiverer det, og `daq_tolerance_ms=` indstiller matchtolerancen; begge resultater angiver, hvad der blev gemt i `.daq_files` og `.daq_matches`. En optagelse venter nu på DAQ'ens næste aflæsning, så den kan blande aflæsningerne på begge sider af eksponeringen. Se [Den DAQ-aflæsning, som en optagelse gemmer](#the-daq-reading-en-optagelse-gemmer--daq_filer-og-daq_match).
> - **`ArraySession` har live-rammer og en kontinuerlig optagelse på serversiden.** `grab()` og `frame_stream()` returnerer numpy-rammer fra én trigger (den samme aflæsning som projekt-API'ets `ArrayHandle.grab()` / `frame_stream()`), og `continuous()` gemmer hver synkroniseret gruppe i backend og returnerer en [`ContinuousHandle`](#continuoushandle). Se [Live-rammer i din egen pipeline](#live-frames-in-your-own-pipeline).
> - **En array `grab()` er knyttet til én trigger** (`ArrayHandle.grab()` og den nye `ArraySession.grab()`). I 1.2.0 læser 12 ud af 20 fire-kamera-`ArrayHandle.grab()`-aflæsninger strakte sig over to triggere. Hvert billede bærer nu `group_id`, og en matrix `frame_stream()` giver hver trigger én gang. Se [Én trigger pr. læsning](#one-trigger-per-read--group_id).
> - **Båndspecifikke og justerede billeder:** `grab()` / `frame_stream()` tager `band=`, `align=`, `bit_depth=` og `refl_max=` på kamera- og array-håndtag. Se [Eksport af et enkelt bånd](#exporting-a-single-band--band).
> - **Nye `ArraySession`-metoder og -argumenter:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (sammen med `CaptureResult.group_seq` og `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` sammenkæder på hinanden følgende grupper og stopper, når drevet er fuldt.
> - **`ArraySession.record()` registrerer uden åben GUI.** Når GUI'ens kombinerede visning er lukket, gengiver backend'et selve visningen: arrayetsNDVI eller det indeks, du angiver som `expr=`. `RecorderHandle.stop()` udløser `ChlorosConnectError` for en optagelse, der mislykkedes – en, hvor der ikke blev skrevet nogen ramme – og bevarer stopresultatet på `.result` (hvis der efterlades en `with`-blok, udløses der ikke en undtagelse: tjek `rec.result`). Se [`RecorderHandle`](#recorderhandle).
> - **Nye `DAQSensorSession`-metoder:** `enable_ptp()`, `force_disconnect()` og for en DAQ-E's egne indstillinger `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` og `update_firmware()`. `record_stop()` returnerer `warning` og `gaps`. En genstart, der lukkede en optagelse, angiver navnet på denne i sit svar eller, hvis genstarten mislykkes, i `ChlorosConnectError` (`recording_closed`; se [DAQ-E-enhedsindstillinger](#daq-e-device-settings)). Se [`DAQSensorSession`-metoder](#daqsensorsession-methods).
> - **En samlet DAQ-E, der holder op med at levere data, genforbindes af backend** med de indstillinger, den havde, og en optagelse på den fortsætter i den samme `.daq` med hullet markeret i filen; `record_stop()` returnerer hullerne som `gaps`. I version 1.2.0 forblev sensoren tavs, indtil den blev afbrudt og tilsluttet igen, hvilket afsluttede optagelsen. Se CLI-referencens [En DAQ-E, der holder op med at levere](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` fungerer fra en installeret SDK.** Hjulet (installationsprogrammetog PyPI'er) indeholder nu de projektloadere, den har brug for; før udløste `open_project()` `ImportError`, der navngav `project_cameras` / `project_sensors` i ethvert pip-installeret SDK. `connect_all()` forbinder de gemte kameraer, arrays og lyssensorer via et kørende backend (lyssensorer via sin DAQ-pool), og en gemt enhed, som den ikke har mulighed for at søge efter, vises under `errors` med en begrundelse i stedet for at blive sprunget over uden en forklaring. Et gemt array, der allerede er oppe i backend'en (GUI'ens, eller `chloros-cli project connect`'s) tilsluttes og forbliver kørende, hvor det tidligere blev rapporteret som en fejl (`Camera <sn> is already in array <id>`). Se [`ChlorosProject`-metoder](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` og `process_lattice_capture()` finder Chloros på Windows.** De søger der, hvor installationsprogrammet placerer backend'en (som standard `<install folder>\backend\` og `C:\Program Files\Chloros\backend\`, en brugerdefineret mappe fra installationsprogrammets egen registrering) og opretter forbindelse til en backend, der allerede kører, uden overhovedet at søge efter en. I version 1.2.0 genererede de fejlen »Chloros-backend ikke fundet« på en installeret maskine, medmindre man angav `backend_exe=`.
> - **En backend, der går ned ved opstart, rapporteres straks.** Når `ChlorosLocal()` eller et smart-connect-kald starter en backend, der afbrydes, før den svarer, udløser opkaldet inden for få sekunder fejlkoden `ChlorosBackendError` fra `ChlorosLocal` og `ChlorosConnectError` fra smart--connect-kald, med afslutningskoden, den kommando, den kørte, og de sidste linjer, som backend'en udskrev. 1.2.0 ventede på, at opstartstimeouten på 60 sekunder udløb, og angav ikke årsagen. Se [Backend Auto-Start](#backend-auto-start).
> - **Backend kører videre mellem scripts.** Når man forlader en `with ChlorosLocal(...)`-blok, `process_folder()` / `process_lattice_capture()`, der returneres, og `chloros-mcp`, der afsluttes, stopper ikke længere et backend, som SDK'et har startet; det næste script genbruger den. Den stopper ved `logout()` (medmindre Chloros-appen er åben eller en kørsel er i gang), ved `shutdown_backend()` eller af sig selv efter 30 minutter, hvis intet bruger den (`CHLOROS_BACKEND_IDLE_MIN` i backendens miljø ændrer dette; `0`: aldrig).
> - **En backend, som SDK'et har startet, bliver bedt om at lukke ned, den bliver ikke afbrudt.** `shutdown_backend()` lader backend'en frigive sine kameraer og lyssensorer og afslutte eventuelle optagelser; den afsluttes kun, hvis den stadig kører 10 sekunder senere. 1.2.0 afsluttede den med det samme (på Windows en tvungen afslutning), hvilket kunne medføre, at en DAQ-lyssensor, der var tilsluttet den pågældende backend, ikke reagerede. En backend, som SDK'et fandt i gang, lades stadig være i fred.
> - **På Linux kører autostart aldrig et `backend_server.py` fra den aktuelle mappe.** Uden en installeret backend kørte et script, der blev startet i en mappe, der indeholdt en, den; nu startes kun den installerede backend (eller, for `ChlorosLocal`, den, som `backend_exe=` angiver).
> - **`configure(indices=...)`**tilføjer hver forudindstilling til projektet for kameraets bånd, inklusive et nyt projekt, og advarer (`UserWarning`), hvis der er et navn, den ikke kan tilføje.**`import_images()`** rapporterer `.daq`-optagelser som `scan_count` og advarer ikke længere om en LATTICE-optagelsesmappe.
> - **`read_image_audit_tags()` fungerer ved installation via pip.** Det læser selv billedets metadata; i version 1.2.0 returnerede det `{}` uden de interne komponenter fra desktop-installationen på Python-stien.
> - **`capture()` umiddelbart efter `connect_array()`** venter op til 20 sekunder på arrayets første billedgruppe i stedet for at fejle med HTTP-kode 503 »Array still warming up«.
> - **`status()` rapporterer en afbrydelse.** Det nye felt `stalled` er True, mens indlæsningsløkken kører, men offentliggør intet, og `actual_fps` er derefter 0. I 1.2.0 beholdt `actual_fps` sin sidste værdi under en afbrydelse, så et afbrudt array viste `actual_fps > 0`, mens optagelser mislykkedes med 503 "Ingen ny synkroniseret billedgruppe".
> - **Færre ufuldstændige rammesæt og 503-fejl ved optagelse:** hver slavekameras triggerindgang er nu renset for glitches.
> - **Pakket 12-bit-transport** (`connect_array(packed="auto")`) øger billedhastigheden for hele sensorarrayet med ca. en tredjedel. Se [Pakket 12-bit-transport](#packed-12-bit-transport-packed).
> - **Array-eksponering under et ROI:** Et lille ROI begrænser ikke længere et medlems eksponering til sensorens friløbsperiode (4,9 ms ved 768 × 304). Se [Eksponering på array-medlemmer](#exposure-on-array-members).
> - **Automatisk eksponering sænker ikke længere et arrays automatiske billedhastighed.** Et kabelsynkroniseret array kører med den hastighed, som netværket og kameraernes udlæsning tillader; automatisk eksponering udfylder den periode, som denne hastighed giver, og hæver derefter forstærkningen. En manuel eksponering, der er længere end denne periode, sænker udløsningen, så den passer til den, i stedet for at blive afvist. Se [Eksponering på array-medlemmer](#exposure-on-array-members).
> - **`read_nodes()`**tilføjer `frame_values` og `notes`, mens automatisk eksponering kører (se [`read_nodes()` under automatisk eksponering](#read_nodes-under-auto-exposure--frame_values-and-notes)). Når kameraet afviser en eksponering eller forstærkning, angiver**`set_settings()`** det interval, det accepterer, og genaktiverer den automatiske tilstand, som det havde slået fra for den pågældende værdi; `set_settings(target_brightness_auto=True)` er nyt.
> - **`orient=`** på `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` og `ArrayHandle.capture()` / `capture_smart()` roterer produkterne i henhold til kameraets retning. Se [`ArraySession`-metoder](#arraysession-methods).
> - **`DLS()` kræver sensorens egen `mac_address`.** Der er ikke længere nogen standard-MAC (det var en enheds adresse); `DLS()` uden en sådan udløser `DLSError`. Se [DAQ-M-båndstørrelser](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **Windows-installationsprogrammet** tilbyder "Aktiver lange stier i Windows" og "Undtag Chloros-backend fra Microsoft Defender-scanning", som begge er markeret som standard. Se [Trin 1](#step-1--install-the-chloros-platform-package).
> - **Linux-.deb-filen** installerer Python-SDK'en til systemet `python3` (tidligere blev kun wheel-filen placeret på disken), øger kernens socket-modtagelsesbuffere, som kamerastrømme har brug for, og tilføjer en backend-tjeneste, der kører som din bruger. Jetson-pakken er `chloros_1.3.0_arm64_jp6.deb`, til JetPack 6, med JetPacks CUDA PyTorch. Se [Trin 1](#step-1--install-the-chloros-platform-package) og [Trin 2](#linux-deb).

---

## Arkitektur på 60 sekunder

SDK’et er et tyndt Python-lag oven på Chloros-backend’et (den samme Flask-server, som desktop-GUI’en og CLI’en bruger). Til automatisering importerer du `chloros_sdk` og kalder metoder på højt niveau; bag kulisserne bliver hvert kald til en HTTP-anmodning til det lokale backend på port 5000 — `http://127.0.0.1:5000/api/...` (bevidst ikke `localhost`, som først omdirigeres til `::1` på Windows og tager ca. 2 sekunder pr. anmodning mod et backend, der kun understøtter IPv4). Backend'et ejer hardwarepuljen — kameraer, DAQ-sensorer, justeringsprofiler, billedbuffere — så SDK-scripts kan eksistere side om side med GUI'en uden at kæmpe om serielle porte eller netværkskortets båndbredde.

Der er tre grænseflader, du vil bruge:

1. **`ChlorosLocal` + frie funktioner** (`process_folder`, `process_lattice_capture`) — Billedbehandlingspipeline. Kør en hel mappe gennem kalibrering / debayer / indekseksport fra ét Python-kald.
2. **Smart-connect-håndtag** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Åbn en vedvarende backend-session til live-hardware. Samme "smart-prep"-forløb som i GUI'en: netværkssøgning, automatisk valg af lag, PTP, AE-seeding, GPIO-triggerkonfiguration.
3. **`ChlorosProject` / `open_project`** — Indlæs et gemt projekt (mappe med `cameras.json` + `sensors.json` + `project.json`), tilslut dets kameraer, arrays og lyssensorer på én gang, og styr optagelser med navngivne håndtag.

Overfladerne 1 og 2 **starter automatisk en lokal backend**, hvis der ikke allerede er en, der lytter (den samme medfølgende binærfil, som GUI'en og CLI'en starter) — så et simpelt script fungerer fra en ny shell, uden at du først skal starte en backend: `ChlorosLocal` (og `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` og `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` og `analyze_array_network()` starter aldrig af sig selv: de kræver et kørende backend. Angiv `auto_start_backend=False` for at fravælge (f.eks. når der peges på et fjernbackend, som aldrig startes). Se [Automatisk start af backend](#backend-auto-start). Surface 3 opfører sig anderledes: `open_project()` tager ingen `auto_start_backend`-parameter, og `connect_all()` starter aldrig en backend — den tester `http://127.0.0.1:5000` én gang, og hvis der ikke kommer noget svar, falder den tilbage til direkte (backend-fri) `lattice_sdk`-enhedsstyring. Det kræver Arena SDK-runtime og OpenCV (`cv2`), der kan importeres i din Python, hvilket SDK-hjulet ikke installerer; uden disse rapporteres hvert gemt kamera og hver gemt matrix som en fejl med angivelse af årsagen. Et projekts lyssensorer har ingen backend-fri vej: uden et kørende backend rapporteres hver gemt sensor som en fejl med angivelse af årsagen (se [`SensorHandle`](#sensorhandle)). Kun `proj.process()` og `stream(..., overlays=True)` opretter en `ChlorosLocal()` (som starter automatisk) ved behov.

Alle tre er auth-afhængige: Kør `chloros-cli login` én gang på maskinen, eller log ind via skrivebords-GUI'en. Hvad et kald uden en gyldig session udløser, afhænger af grænsefladen (se [Undtagelser](#exceptions)): `ChlorosAuthenticationError` fra `ChlorosLocal` (og `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` fra smart-connect-kald og deres sessioner, `status: "error"` med en `error_code` fra `analyze_array_network()` samt en post under `errors` i `connect_all()`-rapporten.

Krav:
- Python 3.7+ (som angivet af pakken; udviklet/testet på 3.10)
- Chloros Desktop installeret lokalt (backend-binærfilen følger med i installationsprogrammet)
- Aktivt Chloros+-login på **et hvilket som helst betalt Chloros+-abonnement (Copper eller højere)**; det gratis**Iron**-abonnement har ingen SDK-/CLI-adgang. Dette håndhæves**på serversiden**-siden**: hver anmodning med SDK-/CLI-flag skal indeholde både en aktiv session og et betalt abonnement. Et abonnement under Copper får fejlkoden `403` sammen med `error_code: PLAN_UPGRADE_REQUIRED` (`ChlorosLicenseError` fra `ChlorosLocal`, `ChlorosConnectError` fra smart-connect-opkald). En afmeldt opkaldende får i stedet `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError` fra `ChlorosLocal`, `ChlorosConnectError` fra smart-connect-opkaldene) — de to er forskellige, fordi en genkørsel af `chloros-cli login` løser problemet med `401`, men kan ikke løse problemet med `403`, som kræver en opgradering af abonnementet.
- Offlinebrug understøttes inden for abonnementets afdragsfri periode: abonnementsniveauet læses fra servervalideringscachen (5 minutter) eller den signerede, maskinbundne licenscache (30 dage for månedsabonnementer, indtil abonnementets udløb for årsabonnementer). Når denne frist udløber, skifter abonnementet til gratisversionen, og adgangen til SDK og CLI stopper, indtil maskinen kan oprette forbindelse til serveren én gang. `chloros-cli status` (`GET /api/license-status`) forbliver tilgængelig på gratisniveauet, så årsagen er synlig.
- Windows 10/11 64-bit, **Ubuntu 22.04 LTS eller nyere**eller Jetson (JetPack 6). Ubuntu 20.04 understøttes**ikke**: `.deb`’s afhængigheder stammer fra det, som backend’en linker til, herunder `libc6 (>= 2.35)`, og Focal leveres med glibc 2.31.

---

## Installation

Python-SDK'et er et tyndt Python-lag oven på Chloros-backend'et. Medmindre du kører et backend på en anden maskine (se [Remote-Backend Mode](#remote-backend-mode-pip-only-host-via-tunnel)), skal du have **Chloros-desktop-pakken installeret lokalt** (Windows-installationsprogram eller Linux `.deb`), og den indeholder DAQ-lyssensorer. Pakken indeholder backend-binærfilen, Arena SDK-runtime til LATTICE-kameraer samt kalibreringspakkerne.

Seneste downloads: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Trin 1 — Installer Chloros-platformpakken

#### Windows (.exe)

1. Download `Chloros Installer 1.3.0.exe` fra downloadsiden.
2. Kør installationsprogrammet, og følg vejledningen. Standardinstallationsstien er `C:\Program Files\Chloros\`.
3. Start Chloros mindst én gang, og log ind med din Chloros+-konto.

Installationsprogrammet tillader, at backend-programmet (`chloros-backend.exe`, intet andet) gennem Windows-firewallen på alle netværksprofiler: UDP-indgående trafik til kamerastrømme, PTP, mDNS og DAQ-E-strømmen samt TCP-indgående trafik udelukkende fra det lokale undernet til den forbindelse, som en DAQ-E opretter tilbage til computeren under en firmwareopdatering ([`update_firmware()`](#daq-e-device-settings)). Afinstallationsprogrammet fjerner begge regler. Backendets HTTP-API forbliver på `127.0.0.1` uanset hvad.

Guidens gruppe **Systemindstillinger** har to indstillinger, som begge er markeret som standard:

| Indstilling | Hvad den gør | Fravælg lydløs installation |
| --- | --- | --- |
| **Aktiver lange stier i Windows** | Tillader, at projektmapper ligger dybere end Windows’ grænse på 260 tegn for stier. Når lange stier er deaktiveret (Windows-standardindstillingen), afvises en projektmappe, der ligger for dybt for dens produkter, før noget behandles — se [Fejlfinding](#fejlfindings-tip). | `/MERGETASKS="!longpaths"` |
| **Ekskluder Chloros-backend fra Microsoft Defender-scanning** | Stopper Defenders real-tidsscanning af hver eneste fil, som en kameraoptagelse skriver. Ved høje optagelseshastigheder udgør scanningen ca. 30 % af hastigheden: målt ved ROI 512 på et array med 9 kameraer kørte kontinuerlig RAW-optagelse 52,7 optagelser/s med scanning og 77,7 optagelser/s med backend udelukket. | `/MERGETASKS="!defenderexclusion"` |

En kontinuerlig optagelse rapporterer Defenders belastning som `defender_cores` (se [`ContinuousHandle`](#continuoushandle)).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Installer med `apt install ./…`, ikke `dpkg -i`: apt installerer også kravene til Python-SDK'et, som pakken anbefaler snarere end kræver (se [Trin 2](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Til Jetson Orin / Orin NX / Orin Nano på JetPack 6 (L4T R36). Dets PyTorch er JetPack 6's CUDA-build, så GPU-behandlingen kører på Orins GPU; det bruger de CUDA- og cuDNN-biblioteker, som JetPack 6 installerer (`sudo apt install nvidia-jetpack` på et image, der mangler dem).

#### Linux: modtagelsesbuffere og backend-tjenesten

Begge .deb-pakker øger kernens socket-modtagelsesbuffere, som LATTICE-kamerastrømme har brug for, til 32 MB (`/etc/sysctl.d/60-chloros-gige.conf`). Ved kernens standardindstilling bliver kamerapakker droppet i kernen, og et array mister billeder og billedhastighed; `chloros-cli lattice network` rapporterer, når bufferne er lave. Se CLI-referencens afsnit [Linux-værter — socket-modtagelsesbuffere](cli-reference.md#linux-hosts--socket-receive-buffers).

På en headless-maskine skal du køre backend'en som en tjeneste **under din bruger**, så et script kan finde en, der kører (`connect_all()` starter aldrig en backend):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

Backend'en kører derefter som dig: de indfangninger, den skriver, er dine, og den bruger din `chloros-cli login`. Root-`chloros-backend.service` er stadig installeret på maskiner, hvor den er aktiveret, men alle filer, som en root-backend skriver, tilhører root. Se afsnittet [Kørsel af backend som en tjeneste](cli-reference.md#running-the-backend-as-a-service-headless) i CLI-referencen.

En backend, som tjenesten kører, er delt og stoppes aldrig af en klient: SDK-scripts og `chloros-cli` bruger den, som den er (`ChlorosLocal.shutdown_backend()` stopper kun en backend, som `ChlorosLocal` selv har startet), og `GET /api/status` navngiver enheden som `service`. Genstart den med `sudo systemctl restart chloros-backend@$USER.service`. Opgradering af pakken stopper backend-tjenesterne, mens den installeres, så et kørende script mister sine kameraer og sensorer og starter derefter alle de aktiverede igen; fjernelse af pakken stopper dem og deaktiverer de aktiverede. Der kører kun én backend ad gangen, så aktiver én enhed: en anden nægter at starte og forbliver `failed` (afslutningsstatus 98, genstartes ikke), og dens `systemctl status` forklarer hvorfor. For at skifte enhed skal du deaktivere den, du ikke bruger.

### Trin 2 — Installer Python-SDK’en

**Chloros-installationsprogrammet leveres med et tilhørende SDK-wheel.** Hvert Windows-installationsprogram og hver Linux-.deb-fil placerer en fil med navnet `chloros_sdk-X.Y.Z-py3-none-any.whl` på disken, som passer nøjagtigt til GUI-/CLI-/backend-versionen. Du behøver ikke at holde øje med PyPI for at holde dig opdateret.

#### Windows

Installationsprogrammet kører automatisk `pip install` mod det medfølgende wheel ved hjælp af dit systems Python (`py.exe`-launcher foretrækkes, falder tilbage til `python -m pip`). Der kræves ingen handling — `import chloros_sdk` fungerer i dit Python-miljø efter en vellykket installation. Hvis der ikke er nogen Python på maskinen, springer installationsprogrammet dette trin over uden at give besked, og GUI + CLI fortsætter med at fungere.

#### Linux (.deb)

Begge .deb-pakker (x86_64 og Jetson) installerer SDK'en til systemet `python3`, så `import chloros_sdk` fungerer efter installationen uden brug af pip og uden netværk. Pakkerne (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` og projektindlæsningsprogrammerne) placeres i `/usr/lib/python3/dist-packages`, og SDK'ets `chloros-mcp`-kommando ([MCP-serveren](#mcp-server-ai-assistants)) i `/usr/bin`, som alle tilhører pakken: en opgradering erstatter dem, og fjernelse af pakken fjerner dem.

SDK'ets egne krav, `python3-numpy` og `python3-requests`, er **anbefalet, ikke påkrævet** (GUI'en, CLI'en og backend'en er selvstændige): `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` på en Jetson) installerer dem, mens `dpkg -i` ikke gør det. Installationsudskriften angiver den SDK-version, som `python3` importerer, eventuelle manglende krav, samt en `chloros_sdk` et andet sted på `python3`'s sti, der overskriver pakkens kopi (f.eks. en, der tidligere er installeret med `pip install --user`).

I et virtualenv, som ikke har adgang til systemets pakker, skal du installere det medfølgende wheel i det:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Ved air-gapped Jetson-implementeringer foregår dette fuldstændigt offline — SDK'et og wheel'et findes allerede på disken.

#### Offentlig PyPI

For værter, der kun bruger pip (ingen Chloros-desktop-pakke installeret; SDK'en styrer i så fald et backend på en anden maskine, se [Remote-Backend Mode](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

PyPI indeholder den seneste stabile udgave. Sørg for, at SDK'en har samme version som den Chloros-backend, den styrer (se [Versionering og kompatibilitet](#versioning--compatibility)). DAQ-M-hjælpeprogrammet `DLS` kræver også `bleak`: `pip install "chloros-sdk[camera]"` (eller `pip install bleak`).

#### Bekræft

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+-abonnement kræves.** Alle SDK-kald kræver et aktivt Chloros+-login. Kør `chloros-cli login user@example.com 'YourPassword'` én gang pr. maskine; loginoplysningerne gemmes i `~/.chloros/`.

### Har jeg brug for Desktop-pakken?

Pip-pakken alene er **ikke** tilstrækkelig til de fleste arbejdsgange. Her er, hvad hver SDK-overflade kræver:

| SDK-overflade | Kræver Desktop-pakke? | Hvorfor |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Ja**(for at starte en)**/ Nej** (hvis der allerede er adgang til en backend) | Brug en kørende backend (desktop-appens eller en anden startet script); ellers skal den installerede startes automatisk: `/usr/lib/chloros/chloros-backend` (Linux) eller `<install folder>\backend\chloros-backend.exe` (Windows; `C:\Program Files\Chloros\backend\` som standard, en brugerdefineret installationsmappe læses fra installationsprogrammets optegnelse). Angiv `backend_exe=` for selv at navngive den. |
| `chloros-mcp` ([MCP-server](#mcp-server-ai-assistants)) | **Ja**(for at starte en)**/ Nej** (et backend er allerede tilgængeligt) | Baseret på `ChlorosLocal`, så det samme som rækken ovenfor. |
| `read_image_audit_tags` | **Nej** | Læser det behandlede billedes egne metadata (kun Python-standardbibliotek). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Ja**(lokalt)**/ Nej**(fjernt) | Rene HTTP-klienter via backend. Lokalt backend → desktop-pakke påkrævet. Fjernt backend → `backend_url=`**gennem en tunnel** (se Fjernt-Backend-tilstand — medfølgende backends binder kun til loopback). |
| `ChlorosProject` / `open_project` | **Ja** | Kører gemte projekter via backend. |
| Direkte LATTICE-klasser (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Ja** | Kræver Arena SDK-runtime, som følger med i desktop-pakken, samt OpenCV (`cv2`) i din Python, hvilket hverken desktop-pakken eller SDK installerer (`pip install opencv-python`). `CAMERA_AVAILABLE` er ellers `False` ved import. `DLS` kræver også `bleak`. |

### Fjern-backend-tilstand (kun pip-vært, via tunnel)

> **Den installerede backend kan ikke nås via LAN.** Den lytter kun på
> loopback (`127.0.0.1` / `::1`), så `backend_url="http://<lan-ip>:5000"`
> **kan ikke nå en installeret Chloros**. For at køre et backend på en anden
> maskine skal du selv videresende dens loopback-port og pege SDK'et mod
> tunnelen:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Headless-/CI-/robotværter kan have én maskine med den fulde desktop-installation som »Chloros-server« og `pip install chloros-sdk` alle andre steder — men transporten mellem dem er den ovenfor beskrevne, brugeroprettede tunnel, ikke en direkte LAN-URL.

> **`ChlorosLocal` på en host, der kun kører pip.** `ChlorosLocal(backend_url=BACKEND)` opretter forbindelse til en backend, den kan nå uden en desktop-installation: den søger kun efter en lokal backend-binærfil, når den skal STARTE en (intet svarer, og `auto_start_backend=True`). Frem til version 1.2.0 søgte den først efter binærfilen og udløste `ChlorosBackendError` („Chloros-backend ikke fundet…“) på en host, der kun kører pip, selvom der var en backend, der kunne nås. De mapper, du angiver (`process_folder`, `import_images`) kontrolleres på denne maskine og læses af backend'en, så ved en fjernbackend skal de henvise til den samme mappe på begge, f.eks. et delt drev.

### DAQ-only-arbejdsgang (pip-only-host)

`pip install chloros-sdk` kan ikke alene styre en DAQ-sensor: en DAQ-sensor styres altid via et Chloros-backend med `connect_daq_sensor()`:

- **Sensor på denne maskine:** Installer Chloros-desktop-pakken her. `connect_daq_sensor()` starter sit backend, hvis der ikke kører et.
- **Sensor på en anden maskine:** den pågældende maskine skal have desktop-pakken; få adgang til dens backend via tunnelen i [Remote-Backend Mode](#remote-backend-mode-pip-only-host-via-tunnel). Backenden åbner sensoren på sin egen maskine, så en DAQ-U er tilsluttet den pågældende maskine, en DAQ-M befinder sig inden for dens Bluetooth-rækkevidde, og en DAQ-E er på et netværk, den kan nå.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Uanset hvad håndhæver backend'en Chloros+-planen: log ind på den maskine, der kører backend'en (`chloros-cli login` eller desktop-appen) med et hvilket som helst betalt abonnement (Copper eller højere). En udlogget backend og et abonnement under Copper udløser begge `ChlorosConnectError` med backendens årsag (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Hurtigstart

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## API-indeks på øverste niveau

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Billedbehandling — `ChlorosLocal`

Den overordnede pipeline-klasse. Starter backend'en, når den oprettes (hvis ingen svarer, og `auto_start_backend=True`), opretter/konfigurerer projekter, overvåger fremskridt og returnerer oversigter efter kørsel.

### Konstruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Metoder

| Metode | Beskrivelse |
| --- | --- |
| `create_project(project_name, camera=None)` | Opretter et nyt projekt (valgfrit med en kameraskabelon som `"Survey3N_RGN"`). |
| `open_project(project_path, relink=None)` | Åbner et gemt projekt (dets mappe eller dets navn i projektmappen) og gør det til det åbne projekt. Stierne til et flyttet projekt omdirigeres til den mappe, det nu befinder sig i; `relink` angiver navnet på den mappe, hvor de øvrige manglende filer nu befinder sig. Returnerer backendens svar med `source_files`; udløser `ChlorosProjectFilesMissingError`, hvis der ikke kan findes nogen. Se **Et flyttet projekt** nedenfor. |
| `import_images(folder_path, recursive=False)` | Importerer RAW/TIF/JPG/DNG-billeder **og `.daq` lyssensoroptagelser**. Returnerer `count` (billeder) og `scan_count` (optagelser). Giver kun en advarsel, hvis mappen ikke indeholder nogen af delene. |
| `export_light_sensor(daq=True, csv=True)` | Skriv kalibrerede `.daq` + `.csv` for hver lyssensoroptagelse i projektet, til `<project>/Light Sensor/`. Se [Lyssensoroptagelser](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Indstil behandlingsknapper. `indices` er forudindstillede navne (`NDVI`, `GNDVI`, `OSAVI`, …), som hver især tilføjes til projektet for det pågældende kameras bånd; et navn, der ikke er en forudindstilling, eller som båndene ikke kan danne (`NDRE` kræver RE og NIR, hvilket intet enkelt kamera leverer), får navnet `UserWarning` og intet indeksbillede. `daq_log_path` betegner en optagelse fra en lyssensor (en `.daq`- eller `.csv`-fil eller en mappe med sådanne filer), som hvert billede sammenlignes med – både i LATTICE og Survey3 – i stedet for de optagelser, der er importeret sammen med billederne; hvis en sti mangler eller ikke indeholder nogen optagelse, anvendes de importerede optagelser. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Kør pipelinen. Returnerer `{"status": "complete", "async": False}`, samt en `summary`-nøgle, når backend'et leverer en — se [Oversigt og tip efter kørsel](#post-run-summary--hints) — `export_check_failure`, når kørselens eksport ikke kunne verificeres, og `image_failure`, når der mangler et billede. `wait=False` returnerer `{"status": "started", "async": True}` på én gang. |
| `get_config()` / `get_status()` / `status()` | Undersøg backend-tilstand. |
| `logout()` | Log ud: rydder de cachelagrede legitimationsoplysninger og frigiver denne maskines enhedsplads (`device_deregistration` i resultatet). En igangværende behandlingskørsel stoppes ikke: den afsluttes i den tilstand, den startede i (`run_in_progress: true` og `run_mode` i resultatet), og den næste kørsel bruger standardbehandling. |
| `shutdown_backend()` | Stopper backend'en, hvis denne instans startede den; en backend, der allerede kører, lades i fred. At efterlade en `with`-blok eller at instansen bliver genbrugsindsamlet, stopper den ikke: backend'en fortsætter med at køre for det næste script og stopper af sig selv efter 30 minutter, hvis intet bruger den. Backend'en bliver bedt om at lukke ned, så den frigiver sine kameraer og lyssensorer og afslutter eventuelle optagelser først; den afsluttes kun, hvis den stadig kører 10 sekunder senere. |
| `discover_cameras()` | Find LATTICE-kameraer **via denne instans' backend** (`/api/camera/discover`). Returnerer en liste over ordbøger (`serial`, `model`, `ip`, …) — samme form som GUI'en og CLI'en ser. Tom liste, hvis der ikke findes nogen, eller hvis backend'en ikke kan nås. |
| `camera_capture(output_dir, format="tiff", **settings)` | Optager et enkelt billede**via backend'en**(startes automatisk af dette håndtag), så det får samme forberedelse som GUI'en/CLI'en (12-bit som standard, genbrug af pool, indlejrede kalibreringsmetadata). Bestem målet med `serial=` eller `device_index=`; overfør `exposure`/`gain`/`pixel_format`/`preset` som `**settings`. Returnerer den gamle metadatadikt (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Genererer overlay-sammensatte forhåndsvisningsrammer fra en samlet kamerapool — en letvægts MJPEG-klient via backendens `/api/camera/<serial>/stream-annotated`-rute (zebra / gitter / sigtekorn / histogram / peaking / spot tegnet på serversiden). `decode=True` genererer BGR-arrays og kræver OpenCV (`cv2`), som SDK'et ikke installerer; `False` genererer rå JPEG-bytes. Kan også tilgås pr. projekt som `ChlorosProject.stream(overlays=True)`. |

**En kørsel, der er i gang, beholder sit projekt.** Mens en Processing-kørsel er i gang — dette scripts, appens, `chloros-cli process`'s eller `chloros-mcp`'s — nægter backend at oprette, åbne, importere til eller ændre et projekts Processing-indstillinger (HTTP 409 `PROCESSING_IN_PROGRESS`): `create_project()` og `import_images()` udløser `ChlorosProcessingError`, og `configure()` udløser `ChlorosConfigurationError`, og `process()` med en anden tilstand end den, der kører, udløser `ChlorosProcessingError` (en kørsel beholder den tilstand, den startede i), hver med backendens besked. En optagelse, der er gemt i projektet, føjes derefter til det, når kørslen slutter (dets svar, `project_add`, angiver dette). Vent på, at kørslen afsluttes (`get_status()`), og fortsæt derefter.**Et flyttet projekt.** Et projekts `project.json` navngiver hver kildefil (hvert billede og hver lyssensoroptagelse) med dens fulde sti, så hvis projektmappen – eller hele projektmappen – flyttes til et andet drev, forbliver filnavnene henvist til den gamle placering. Hver gang projektet åbnes, retter det selv det, det kan: Der søges efter en fil, der lå i projektmappen, i den mappe, hvor projektet nu befinder sig, og hver sti, der findes der, omdirigeres og gemmes (`project.json` sikkerhedskopieres først som `project.json.before-relink-<date>-<time>`). En sti omdirigeres kun til en fil, der faktisk findes der. Filer, der lå uden for projektmappen – en hub-session, en »Tilføj mappe«-import – findes kun i `relink=<the folder they are in now>` (eller en hvilken som helst mappe over den). `open_project()` returnerer `source_files`: `total`, `found`, `missing`, `missing_roots` (de mapper, hvor de manglende filer befandt sig), `missing_outside_project` og `relink` (hvad der blev omdirigeret, fra hvor til hvor). Et projekt, hvor nogle filer mangler, åbnes sammen med resten; et projekt, hvor der ikke findes nogen, åbnes ikke, og `open_project()` udløser `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). CLI's `project open --relink` og `chloros-mcp`'`open_project` (`relink_folder`) gør det samme.

Brug som kontekstmanager for garanteret oprydning:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Lyssensoroptagelser — kalibreret `.daq` + `.csv`

En DAQ-U / DAQ-M / DAQ-E kan optages **uden** dens kalibreringspakke. Det er
det, de offentlige [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
optagere (`record_daq.py`) gør som standard: de skriver rå sensortællinger og stempler
filen, så Chloros henter den pågældende sensors fabrikskalibrering **via serienummeret** — først fra den lokale cache
først, derefter MAPIR-skyen — og anvender den ved import.

Chloros skriver resultatet tilbage som to produkter pr. optagelse under
`<project>/Light Sensor/`:

| Produkt | Hvad det er |
| --- | --- |
| `<name>_calibrated.daq` | Det genbehandlingsbare arkiv — samme skema som en live-optagelse, men nu med angivelse af den pakke, der har genereret det. Genimport af det kalibrerer det **ikke** en anden gang. |
| `<name>_calibrated.csv` | Spektral irradians i W/m²/nm på sensorens eget bølgelængdegitter, én række pr. måling, plus fotometriske kolonner (total effekt, fotopisk/skotopisk lux, PPFD og dens opdeling i blå/grøn/rød, spidsbølgelængde). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Kun sensorer uden bundt (DAQ-A).** Rå spektrale sensortællinger — *ikke* bestrålingsstyrke. Se nedenfor. |

`<name>` er optagelsens eget filnavn. To optagelser i forskellige mapper kan dele
én — måleværdier pr. optagelse (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`), der er gemt i
samme millisekund af to kameraer — og hver får stadig sine egne produkter: den første beholder
`<name>`, den næste tilføjer de mapper, der adskiller den, **efter** navnet
(`<name>__<capture>_daq_calibrated.daq`), så optagelsens dato og klokkeslæt stadig står først.

`process()` udfører denne eksport som et af sine trin. Det kræver **ikke** billeder:
en lyssensor, der flyves alene, er en førsteklasses arbejdsgang, og et sådant projekt har ingen
billeder pr. definition.

**DAQ-A-optagelser eksporteres som rå tællinger.** DAQ-A-familien er ældre end det serienummerbaserede
bundlesystem og har ingen bundle at hente — den kalibreres i stedet i felten mod et
reflektansmål, hvilket er grunden til, at den aldrig har haft brug for en. Disse optagelser eksporteres
under en `_raw`-stamme i stedet for `_calibrated`: et andet filnavn i stedet for et flag
inde i filen, fordi angivelsen skal kunne overleve at blive videresendt via e-mail som et rent navn.
`.csv`-overskriften angiver `raw spectral sensor counts (NOT irradiance)` og advarer om, at
værdierne er sammenlignelige **inden for** filen — præcis det, som målbaseret kalibrering bruger
dem til — og ikke på tværs af sensorer. De effekt-afhængige fotometriske kolonner (total effekt,
fotopisk/skotopisk lux, PPFD) returneres som **NULL** i stedet for at blive integreret ud fra tællinger.

En DAQ-U / DAQ-M / DAQ-E, hvis bundle simpelthen ikke kunne hentes, **springes stadig over**,
og der skrives ikke rådata: her findes bundlet, og »genopret forbindelse og genbehandl« er et reelt råd.

Ældre **v1.01 / v1.02**-optagelser (en DAQ-A-SD skriver disse) indeholder ingen epoke pr. aflæsning,
kun filens skrivetid. Matcheren mellem billede og nedadgående lysstrøm afviser dem stadig — at matche en
ramme mod et skrivetidspunkt ville være usynligt forkert — men eksportøren læser dem, og
filen CSV udskriver `clock=daq_created_on`, så produktet angiver, hvilket ur det er baseret på.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

En optagelse, hvis kalibreringspakke ikke kan hentes (offline eller en sensor uden
kalibrering i filen), rapporteres under `skipped` **med årsagen**. Den skrives aldrig
ud som en »kalibreret« fil med rå tællinger — opret forbindelse til internettet og
kør den igen, så eksporten fuldføres.

### Statusopkald

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Oversigt og tip efter kørsel

Når kørslen er afsluttet, henter `process()` `GET /api/processing-summary` og vedhæfter brødteksten som `result["summary"]`. Hentningen sker efter bedste evne og blokerer aldrig et vellykket svar — hvis oversigten ikke er tilgængelig, falder `process()` tilbage til den almindelige `{"status": "complete", "async": False}`-form. Hver post i `summary["hints"]` — hele sætninger med den foreslåede afhjælpning, f.eks. hvorfor en kørsel gav nul output — genudsendes også som en Python-`UserWarning`, så kørsler med nul output er selvdiagnostiserende, selvom du aldrig inspicerer ordbogen:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` er den maskinlæsbare del:

| Nøgle | Hvad den tæller |
| --- | --- |
| `models` | Kameragrupper i kørslen. |
| `images_in_groups` | Kildebilleder på tværs af disse grupper. |
| `targets_found` | Registrerede reflektansmål. |
| `images_calibrated` | Billeder, som kørslen genererede et reflektansprodukt for. |
| `images_failed` | Billeder, der mislykkedes og ikke blev eksporteret. |
| `exported_files` | **Billedproduktfiler , som kørslen genererede**: alle produkter (debayered, preview, radiance, reflektans, indeks, målrenderinger) i alle formater. |
| `reflectance_files` | Reflektansprodukterne blandt disse (`Reflectance_Calibrated_Images`), uanset om de er kalibreret af et mål eller lyssensorens (DAQ) irradians. En kørsel uden mål, der alligevel har skrevet reflektans, angiver dette i et tip; kun en kørsel, der ikke har skrevet nogen, får at vide, at dens reflektans-output mangler. |
| `clamp_masks_written` | De [unmix-clamp-masker](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on), der er angivet ved siden af dem — ledsagere, ikke produkter, så de er ikke med i `exported_files` (CLI'en tæller dem separat på samme måde). |
| `captures_left_out` | Råbilleder i projektets `captures`-mappe, som denne kørsel ikke har taget. En kørsel venter ved opstart på det, der blev optaget før den, så dette er 0, medmindre en optagelse blev gemt under kørslen, eller ventetiden udløb. `summary["captures_left_out_note"]` angiver, hvad der skal gøres med dem. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Optagelser fra lyssensoren, der bevidst tælles separat — de stammer fra en anden fase og findes for kørsler uden billeder overhovedet, så hvis man inkluderede dem, ville en kørsel, der kun indeholdt DAQ, se ud som om den havde eksporteret billeder. |

Sammen med dem: `summary["output_dirs"]` (alle mapper, der er skrevet til),
`summary["light_sensor_export"]`, `summary["stopped"]` (sandt, når brugeren afbrød
kørslen, så delvise tællinger ikke tolkes som et afsluttet kørsel med for lav produktion), og
`summary["groups"]` (opdelingen pr. gruppe).

`exported_files` registreres af pipelinen **mens den skriver**, ikke ved efterfølgende scanning af
projektets billedobjekter. De parallelle og GPU-strategier opbygger deres egne billedobjekter
(i arbejdsprocesser for GPU-stierne), så den gamle scanning rapporterede
`0 file(s) written` for hvert sådant kørsel og udsendte derefter »zero-exports«-hintet — på kørsler,
hvor alt havde fungeret. Hvis du skriver et script baseret på dette tal, rapporterer en velfungerende parallel kørsel nu
et tal, der ikke er nul.

Spring af lyssensoren rapporterer den årsag, som læseren faktisk fastslog for hver fil — et
ulæseligt skema, et manglende bundt, en skrivefejl — **dedupliceret**, så tyve filer,
der blev sprunget over af én årsag, vises som én årsag i stedet for tyve gentagelser af den.

> **`process()` udløses ikke, når en kørsel ikke producerer nogen billeder.** Dette er et af de to steder, hvor SDK og
> CLI bevidst adskiller sig: `chloros-cli process` behandler »produkter blev anmodet om, ingen blev
> skrevet« som en fejl og afsluttes med en værdi forskellig fra nul, mens SDK returnerer normalt og rapporterer
> tilstanden via `summary` / hints. Hvis din pipeline skal stoppe ved et tomt kørsel, skal du kontrollere det
> selv — undersøg `summary` (eller tæl filerne i projektmappen) i stedet for at stole på
> fraværet af en undtagelse. De sædvanlige årsager er en indgangsmappe, der ikke blev genkendt som en
> optagelse, og produkter, der blev sprunget over, da de ikke var relevante for de tilstedeværende kameraer (f.eks. radiance fra kameraer, der kun tager RGB
> billeder).
>
> `process()` udløses heller ikke, når en kørsels eksportfiler ikke kunne verificeres: der vises en advarsel, og
> `result["export_check_failure"]` indstilles. Produkterne skrives til. For at stoppe ved dette skal du kontrollere den nøgle. En kørsel, der
> har mistet et billede, fungerer på samme måde: der vises en advarsel, og `result["image_failure"]` sættes, hvilket får `chloros-cli process`
> til at afslutte med en værdi forskellig fra nul.

### Hjælpefunktioner

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Hver kører inden for sin egen `with ChlorosLocal(...)`; et backend, den var nødt til at starte, fortsætter med at køre, når den returnerer, til det næste kald, og stopper selv efter 30 minutter, hvis ingen bruger den. Ingen af dem tager `auto_start_backend`, `radiometric_output` eller `daq_log_path`; brug `ChlorosLocal` og `configure()` til disse. `process_lattice_capture()` accepterer ikke `camera`, `debayer` eller `ppk`: den springer panelregistreringen over og bruger standard-debayer.

### Understøttede værdier

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrisk output (LATTICE multispektral pipeline)

LATTICE-multispektral (M3C/M3M)-eksportniveauet for `process`-pipeline — `reflectance` (standard), `radiance`, `sensor-response` eller `all` (alle relevante tilstande pr. billede) — indstiller projektets **"Eksporter radiance"**og**"Eksporter reflektans"**: `radiance` eksporterer kun radiance, `reflectance` kun reflectance, `sensor-response` ingen af delene (kun de debayered og preview-produkter), `all` begge dele. Produkterne med debayering og forhåndsvisning kan ikke ændres på dette niveau. `configure()` har et dedikeret nøgleord til dette:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Den avancerede udvej — at skrive produkterne via `custom_settings` — fungerer stadig, men husk, at den erstatter hele indstillingsblokken (se advarslen nedenfor). Skriv nøglerne `"Export <product>"`: et projekts egne `"Export radiance"` / `"Export reflectance"`-indstillinger har forrang for den ældre `"Radiometric output"`-nøgle, så den nøgle alene ændrer intet.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (standardindstillingen) dividerer kameraets strålingsintensitet med den **tidsstempelmatchede DAQ-nedstråling**, der automatisk udledes fra en registreret `.daq` (DAQ-U/M/E)**eller en DAQ-M-specifik `.csv`**, der findes sammen med billedmaterialet; eventuelle kamera- eller DAQ-kalibreringspakker, der mangler lokalt,**hentes automatisk fra AWS** ved første brug. CLI'en eksponerer dette i henhold til-type produktvælgere på `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**Hvem tilhører målingen?**En `.daq`, som en optagelse gemmer, er målingen fra det eller de kameraer, den beskriver — angivet i en `<file>.daq.cameras.json`-sidecar ved siden af den (enkelt-, synkroniserede og array-optagelser; en rå burst angiver hvert medlems måleværdi i sit manifest) — og matches**kun med billeder fra disse kameraer**. Et fristående kamera uden egen måleværdi låner derfor ikke længere et andet kameras DAQ; bind en DAQ til det, eller optag en. En DAQ’s egen optagelse (`chloros-cli daq pool-record` eller en optagelse startet i GUI’en) eller enhver `.daq` uden den pågældende sidecar (herunder ældre optagelser) fungerer stadig for alle kameraer.**OCN-kameraer i sollys.** Hver OCN-kalibreringspakke er opbygget under smalbåndslys, én LED pr. bånd, mens sollys blander båndene. Chloros 1.3.0 anvender en hurtig solbaseret korrektion efter båndadskillelsen, hvilket forbedrer OCN-produkterne for stråling, reflektans og indeks i sollys. Den kører overalt, hvor adskillelsen finder sted — `process()`, optagelser, `grab()` / `frame_stream()` og live-visningen — for OCN-pakker, der understøtter det; RGN-kameraer forbliver uændrede. OCN-billeder, der er optaget med en tidligere version, får også denne funktion, når de behandles på en computer, der indeholder den kalibrering, de blev taget med (Chloros bevarer et kameras kalibrering, når kameraet først er blevet tilsluttet); ellers behandles de som før. OCN-værdier for stråling, reflektans og indeks adskiller sig derfor fra dem i version 1.2.0.

> `custom_settings` **erstatter** hele blokken med beregnede indstillinger (den omgår bevidst `configure()`’s øvrige nøgleord og validering). Når du bruger den, skal du medtage alle `Project Settings`-nøgler, der er relevante for dig, som i eksemplet ovenfor.

#### Læsning af indekspixels (NDVI og de andre vegetationsindekser)

**Et heltalindeksraster er en kontrastudvidelse pr. billede, ikke et −1…1-kort.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` og `JPG (8-bit)` indeksprodukter — herunder den 8-bit TIFF, som en 8-bit-optagelse gemmes som — gemmer

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

hvor `IndexMin` / `IndexMax` er **den pågældende rammes egne laveste og højeste endelige indeksværdier**. DN 0 er ikke −1, og fuld skala er ikke +1.

I Chloros-standarden mærkes begge endepunkter, så afbildningen forbliver inverterbar:

| Tag | Betydning |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — konventionen nedenfor |
| `Chloros:IndexMin` | den indeksværdi, der er gemt som DN 0 |
| `Chloros:IndexMax` | indeksværdien gemt som fuld skala |

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

Indlæsning kræver ingen `-config pix4d.config` — exiftool læser disse XMP-egenskaber uden hjælp. Konfigurationen er kun nødvendig for at *skrive* dem.

> **`IndexMin == IndexMax`** betyder, at hver pixel i det pågældende billede havde den samme indeksværdi. Rasteret er udfyldt midt på skalaen, og der er intet at dividere med — tag endepunktet.

> **Ingen `Chloros:Index*`-tags betyder, at der ikke er angivet nogen kortlægning.** En LUT-eksport er en farvelagt gengivelse, hvis pixels består af farvegradienter, der er knyttet til dit LUT-tærskelvindue, og et billede uden pixels med endelig indeksværdi har intet interval at angive. Antag ikke, at der foreligger en kortlægning i nogen af tilfældene — den samme regel gælder som ved manglende `PixelScale`.**Da strækningen er pr. billede, kan to billeder fra samme flyvning ikke sammenlignes som DN**— hvert billede har sine egne endepunkter, så den samme NDVI-værdi ender på forskellige DN-værdier i forskellige billeder. Til kvantitativt arbejde på tværs af en flyvning (zonestatistikker, faste tærskelværdier, tidsserier) skal du eksportere**`TIFF (32-bit, Percent)`**: indeksrastere skrives der som float32**indeksværdier i sig selv**, uden behov for omskalering eller tags. På trods af formatets navn er indeksprodukter ikke ×100 — den skalering gælder kun for reflektans med heltalstypen.

#### Unmix-begrænsningsmaske — hvilke pixels et bånd blev begrænset på

Den 3×3 spektrale unmix kan gøre et bånd negativt — hvor lyset knap nok indeholder det pågældende bånd, eller hvor scenen falder uden for det, som kameraets unmix-matrix beskriver. Det er ikke en reel værdi, så eksporten fastsætter den til 0 — og livevisningen viser den samme 0. Et bånd kan være begrænset til nogle få pixels eller til næsten alle; masken registrerer begge dele. `Chloros:Band{R,G,B}UnmixClampFraction` angiver, hvor stor en del af et billede det skete for; clamp-masken angiver **hvilke pixels**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— i en `clamp_masks/`-mappe**ved siden af** produktets mappe, aldrig inde i den (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — for hvert strålings-, reflektans- og indeksprodukt, hvor mindst én pixel er blevet fastlåst. Enkeltbånds `uint8`, på produktets eget pixelgitter (orientering, ROI, binning og array-justeringsforvrængning inkluderet). Hver maske angiver sit produkts kamera (`Make` / `Model` / serienummer), optagelsestidspunkt og objektivmærker samt en liste over de bånd, som dens bits repræsenterer (`XMP-Camera:BandName`, bit 0 først); dens `ImageDescription` og `Software` markerer den som en clamp-maske, og den indeholder ingen kalibrering.
- **Bits**: bit 0 = båndet i produktets R-kanal (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). En sat bit betyder, at den pågældende pixels værdi i det pågældende bånd blev begrænset til 0 — betragt den, og ethvert indeks der er opbygget ud fra den, som lavt pålideligt.
- **`Chloros:UnmixClampMask`** er stemplet på hvert produkt, der har kørt unmix: maskenes sti i forhold til produktets egen mappe (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, altid `/`-adskilt — sammenkæd den med billedets mappe for at åbne masken), eller `none`, når ingen pixel er fastlåst. Intet mærke betyder, at produktet aldrig har kørt unmix-processen (RGB- eller monokameraer, rå/debayered niveauer).
- Optagelsesresultater indeholder det som billedrammens **`clamp_mask`** (maskens sti, eller `None`, når intet er begrænset) — i `CameraSession.capture()`-resultater er `ArraySession.capture()`'`CaptureResult`-rammer og HTTP-optagelsessvarene. Offline-buildet angiver sine masker som `clamp_masks`.
- Et live-indeks’ multibåndsstak (`/api/camera/cli-array-index` med `save_multiband`) får én maske pr. kildekamera i output-mappens egen `clamp_masks/` — se CLI-referencen’`lattice index`. En video har ingen maske: resultatet fra offline-opbygningen angiver i stedet hver enkelt frames clamp-fraktion.

Da ingen produktmappe indeholder en maske, uanset dybde, kan en produktmappe overføres til fotogrammetri-software, som den er — inklusive undermapper — og den tager kun billeder ind. Overfør produktmappen, ikke mappen ovenover den. Chloros’ egen import springer maskerne over.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect til LATTICE-kameraer

Vedvarende backend-sessioner til live-hardware. De samme endpoints, som GUI’en bruger, så adfærden er identisk på tværs af SDK, CLI og GUI.

### Enkelt kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

Den eksponering, forstærkning og det mål, der indstilles her, bevares som dine: Chloros’ automatiske eksponeringsstyring (inklusive den DAQ-målte eksponering) rører dem ikke. `cam.set_settings(exposure_auto="Continuous")` (eller `gain_auto=`) returnerer eksponeringen (forstærkningen), og `cam.set_settings(target_brightness_auto=True)` overdrager målet tilbage til Chloros' automatiske mål.

#### `connect_camera()` Signatur

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **Den vender tilbage, når streamen er startet.** Backend'et venter, afgrænset (ca. 15 sekunder på, at streamen starter, derefter 10 sekunder på den første komplette buffer — ikke på det første preview-billede, som følger ca. 1,5 sekunder senere), så der kommer en `status()` lige efter, at der står `streaming`. Sessionens `connect_stream` er svaret `stream`-blok: `state` `streaming`, `no_frames` (strømmen startede og leverede intet — det fremgår af `note`, og for et kamera med jumbo-pakker skal der genoprettes forbindelse med en pakkestørrelse på 1500 byte), `starting` eller `start_failed`, hvor `waited_s`. `None` for et kamera, der allerede var tilsluttet. En forhåndsvisning (`get_camera_preview`, `/api/camera/jpeg/<serial>`), der anmodes om umiddelbart efter, afventer (i op til 4 sekunder) kameraets første forhåndsvisningsbillede i stedet for at svare "Ingen ramme tilgængelig endnu"; et kamera, hvis stream ikke har leveret noget (`no_frames`, eller aktiveret for en hardware-trigger), svarer stadig med det samme.
- **Pakstørrelsen kan du selv fastlægge.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` åbner kameraet med pakker på 1500 byte, uden at der foretages en test (`chloros-cli lattice cam-connect --packet-size standard` sender det samme).
- **Der må kun være én forbindelse ad gangen i kamerapuljen**: åbningen, sonderingen af pakkestørrelsen, en første kalibreringsdownload (ca. 70 s for et kamera, der er nyt på denne computer) og streamens start. En anmodning, der har brug for et kamera i mellemtiden (`read_nodes`, `set_settings`, en optagelse, en anden forbindelse) venter højst 5 sekunder på det og mislykkes derefter med HTTP-kode 503 (`CAMERA_POOL_BUSY`), hvor det kamera, der opretter forbindelse, angives (`ChlorosConnectError`); spørg igen om et øjeblik. Listerne venter aldrig: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` og `CameraSession.status()` svarer med det samme. Et kamera, der er midt i en forbindelse, vises under pool-svaret `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — og `elapsed_s`), og dets `status()` angiver `connecting: True` med dets `phase`; de øvrige kameraer aflæses live, når puljen er ledig, ellers svares der ud fra deres seneste aflæsning (`live: False`).

#### `CameraSession`-metoder

| Metode | Beskrivelse |
| --- | --- |
| `status(timeout=15.0)` | Kameraets live-tilstand, som `chloros-cli lattice status` aflæser den: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` eller `stopped`), `device` (model, serienummer, firmware, temperatur, `is_mono`) og dens aktuelle indstillinger. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Læser GenICam-noder; returnerer `{nodes, errors, enums, device, was_in_pool, in_array_id}`, samt `frame_values` og `notes`, mens automatisk eksponering kører (se nedenfor). |
| `set_settings(**kwargs)` | Skriv noder efter venligt navn (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). En `exposure_time` (`gain`) er en manuel eksponering (forstærkning): angivet uden `exposure_auto` (`gain_auto`), skal autotilstanden først slås fra, så værdien ikke afvises af en kørende autotilstand. Resultatet, `applied`, indeholder det, kameraet har optaget; en skrivning, som kameraet afviser, findes i `errors` og gemmes ikke. Fejlen angiver det interval, som kameraet nu accepterer (et maksimum i `ExposureTime` følger kameraets billedhastighed), og en automatisk tilstand, der kun blev slået fra for den afviste værdi, genaktiveres; `restored` viser det (f.eks. `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Optag en **enkelt** billede. Returnerer en `CaptureResult`: en liste over billedmetadata-ordbøger (én pr. gemt niveau) med `.skipped`, `.failed`, `.daq_files` og `.daq_matches`. (Seriebilledeoptagelse/optagelse af flere billeder er fjernet — kald `capture()` i en løkke, hvis du har brug for en serie.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Back-to-back (`interval_s=0`) eller intervaloptagelser fra dette kamera, for `count`-optagelser og/eller `duration_s` (mindst én, så den afsluttes), ligesom `ArraySession.capture_repeated()` gør for en matrix. Returnerer listen over `CaptureResult`'er; den stopper efter en optagelse, der rapporterer et fuldt drev. `on_capture(i, result)` kører efter hver enkelt; andre nøgleord går til `capture()`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Frigivelse fra puljen; kameraet sættes tilbage, som det var ved tilslutning (udløserledninger, PTP, indstillinger); `keep_settings=True` gemmer i stedet sine indstillinger i sin egen hukommelse, `clear_settings=True` nulstiller denne hukommelse (se [Kamerahukommelse](#camera-memory--keep_settings--clear_settings)); returnerer `camera_memory`, når der anmodes om en af delene. Et kamera, der er medlem af en array, frigiver hele arrayet. Ingen handling, hvis vi tilsluttede os en allerede åben session. Returneres, så snart backend'en bekræfter frigivelsen, eller når der ikke er noget at frigive (kameraet er allerede væk, eller der kører ingen backend); en frigivelse, der er løbet ud, eller som backend'en har afvist, udløser `ChlorosConnectError`, fordi kameraet muligvis stadig er optaget, og den næste forbindelse ville komme i konflikt med det. Afslutningen af `with`-blokken udløser aldrig en fejl. |

Attributter: `serial`, `device` (kameraets opdagelsespost) og `backend_url`.

`capture()`-eksportkontroller (samme model som arrayet + GUI):

- `processing` / `levels` — `processing="all"` gemmer alle relevante eksporttyper; `levels=["raw","radiance"]` gemmer kun disse (overskriver `processing`). Udelad begge for at bruge backend-standardindstillingen.
- `daq` — den DAQ, der har aflæst den anvendte optagelse, gemmes som standard som en `.daq`-sidecar, inklusive en råoptagelse, så billedet senere kan genbehandles til reflektans/indeks. `daq=False` springer den over, når intet niveau bruger den. `force_daq=True` er den ældre stavemåde for standardindstillingen. Ingen DAQ, ingen `.daq`.
- `daq_tolerance_ms` — hvor langt fra eksponeringen må målingen ligge og stadig være inden for tolerancen (standard 500). Kun vurderingen ændres.

#### `read_nodes()` under automatisk eksponering — `frame_values` og `notes`

Mens `ExposureAuto` (`GainAuto`) kører, læser `ExposureTime` (`Gain`) den **sidste værdi, der blev skrevet**, og ikke hvad kameraet rent faktisk gør. `nodes` beholder stadig denne nodeværdi. Når du anmoder om `ExposureTime` eller `Gain` på et tilsluttet kamera, hvis automatiske tilstand ikke er `Off`, tilføjes følgende i svaret:

- `frame_values` — den værdi, som kameraets seneste billede indeholdt, under nodens navn (`ExposureTime` i µs, `Gain` i dB);
- `notes` — én sætning pr. sådan node, der angiver begge værdier.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Begge nøgler mangler, når automatisk tilstand er `Off` (noden er da kameraetsreelle værdi) og for et kamera, der ikke er tilsluttet backend. For en matrix angiver `ArraySession.status()["member_ae"]` alle elementers billedværdier på én gang.

#### DAQ-målt eksponeringsloft (HTTP)

Når en DAQ-lyssensor er tilsluttet, begrænser Chloros et kameras eksponering × forstærkning ud fra det målte dagslys, så et hvidt referencepanel ikke bliver overskåret. Loftet gælder kun, mens et kalibreringsmål er i kameraets synsfelt (set i dets billede inden for de sidste 10 sekunder); når der ikke er noget mål i synsfeltet, kører kameraets almindelige automatiske eksponering i stedet med sin egen højlysbeskyttelse, så resten af scenen ikke holdes mørkere, end den behøver at være. Der findes ingen SDK-wrapper; indlæs eller skift indstillingen pr. kamera med `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (POST-body `{"enabled": false}` deaktiverer funktionen for det pågældende kamera). Svaret indeholder `enabled`, `cap_us` (det aktuelle loft, `None` når funktionen er inaktiv, herunder når der ikke er noget kalibreringsmål i billedet), `k_exp_g` (den konstant, som loftet beregnes ud fra) og:

| Felt | Betydning |
| --- | --- |
| `k_exp_g_source` | Hvilken konstant der anvendes: `camera` — dette kameras egen, beregnet ud fra laboratoriepunktet i dets kalibreringspakke (RGB-kameraer); `fleet` — flådekonstanten, der anvendes, når pakken ikke har noget lab-punkt; `override` — en feltoverskrivning indstillet på backend. |
| `k_exp_g_fleet` | Flådekonstanten, til sammenligning. |

### Synkroniseret array — `ArraySession` (Smart-Prep)

`connect_array` er **det anbefalede udgangspunkt** for opsætninger med flere kameraer. Det kører hele Smart-Prep-forløbet via grafisk brugergrænseflade (GUI) i baggrunden:

1. **Netværksanalyse**(`/api/camera/array/recommend`) — indstiller**hele sensoren** (ingen binning, ingen ROI-beskæring) og lader billedfrekvensen falde til det, som denne vært og dette netværk leverer, projiceret med de pixelformater og den pakning, som Connect vil bruge. Billederne krympes aldrig for at holde frekvensen oppe: send en mindre `width` / `height` eller `binning=2` / `4` for at bytte opløsning mod billedhastighed (endepunktets `alternatives` viser disse muligheder med deres forventede fps).
2. **Automatisk valg af niveau** — `sim-capture-sim-emit` (alle kameraer eksponerer ved samme udløser, udsendelsestempoet tilpasses for at dele ledningen, billedfrekvensen falder i takt med at delingen mindskes) på ethvert netværk, hvor kameraerne kan tilpasses til at dele; `slip-emit-and-capture` (kameraerne skiftes til at bruge ledningen og eksponeres ikke samtidigt) kun når dette ikke er muligt.
3. **PTP aktiveret**som standard — tidsstempler på tværs af kameraer lander på et fælles ur med en nøjagtighed på**~1 ms**, backendens [PTP-grandmaster](#ptp-grandmaster--time_sync_status). Samtidig eksponering sker via M8-hardwareudløseren (**< 100 µs** mellem modulerne), ikke via PTP: PTP synkroniserer *tidsstempler*, ikke eksponeringer.
4. **Automatisk valg af pixelformat pr. kamera** — mono → `Mono12`, farve → `BayerRG12`.
5. **AE-seeding** — tager øjebliksbilleder af hvert kameras aktuelle AE-tilstand, så forbindelsen ikke nulstiller eksponeringen undervejs.
6. **GPIO-triggerkonfiguration** — `connect_array` aktiverer alle kameraer (`TriggerMode=On`, `TriggerSource=Line2`), så masterenspuls styrer slaverne via M8-kablet. Dette trin gælder kun for arrayet: et enkelt kamera, der åbnes med `LatticeCamera`, kører i stedet frit. Når arrayet frigives (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) sætter alle kameraer tilbage til frit kørende tilstand (`TriggerMode=Off`), mens alt andet forbliver, som det var ved tilslutningen: indstillingerne for udløser og synkroniseringslinje (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; masterens synkroniseringslinje tilbage til en indgang), PTP og de kameraindstillinger, som sessionen ændrede — medmindre `keep_settings=True` / `clear_settings=True` (se [Kamerahukommelse](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Attributter: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` og `packed_reason` (de sidste fire er `None` på et håndtag fra `attach_array`), `connect_stream` og `backend_url`. `connect_stream` er forbindelsessvarets `stream`-blok, den enkelte forbindelses form plus `members` (`{serial: {state, why?}}`): `streaming`, når arrayets første synkroniserede gruppe er ankommet (backend venter på den, begrænset til ca. 10 s), eller `no_frames` med hvert medlems seneste hentningsfejl som sin `why` og, for jumbo-pakker, 1500-byte-fallback i `note`. `None` på et `attach_array`-håndtag.

#### `connect_array()`-signatur

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` angiver, hvor længe opkaldet venter på forbindelsen, og det sendes til
backend som dens deadline. En forbindelse, der ikke er oprettet inden da, annulleres: hvert kamera, der
blev åbnet, sættes tilbage til den tilstand, det var i, da det blev fundet, og lukkes; ingen array er forbundet, og
opkaldet genererer `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, hvilket angiver det
trin, som forbindelsen var nået til). Et opkald, der udløser en fejl, efterlader altså aldrig et array
forbundet bag sig. Et array med 9 kameraer opretter forbindelse på cirka et minut; lige efter
at kameraerne er tændt, kan det tage flere minutter. Et array, der er oppe, men endnu ikke har en
synkroniseret billedgruppe, returneres med `connect_stream["state"]`
`no_frames`. Backendens log indeholder én tidslinje pr. forbindelsestrin
(`[array-connect <id>] <step>: <s> s …`) og en sidste linje, der angiver det langsomste.

`force_tier`-værdier:
- `"sim-capture-sim-emit"` — ægte simultan (alle kameraer udløser på samme klokkant).
- `"sim-capture-ftd-stagger"` — fleksibel tidsmæssig forskydning (kameraerne udløser på let forskudte tidspunkter, så pakkerne serieliseres på ledningen).
- `"slip-emit-and-capture"` — sekventiel optagelse pr. kamera (ingen tidsmæssig synkronisering). Vælges automatisk kun, når kameraerne ikke kan køre langsomt nok til at dele ledningen samtidigt.

`wire_ceiling_mbps` tilsidesætter **værtsens vedvarende ledningsbudget** i MB/s — det eneste
tal, som hele array-allokeringen afhænger af. Lad det stå på `None` for at bruge den automatisk registrerede
værdi. Sænk det, når arrayet rapporterer GVSP-korrupte rammer: den automatiske værdi er afledt
af NIC’ens annoncerede linkhastighed, som overvurderer USB-adaptere, smalle PCIe-baner og
belastede delte netværk — og denne overvurdering viser sig som korrupte rammer snarere end som et
synligt langsomt link. Værdien gemmes i projektets array-optagelsesblok, så en
genåbning eller en senere `connect_array` gendanner den ligesom enhver anden array-indstilling.
Se [Array-tilstand](#array-health--which-subsystem-is-losing-frames).

#### Pakket 12-bit-transport (`packed=`)

Ved fuld sensorindstilling bestemmes et arrays billedhastighed af, hvor hurtigt hvert kamera kan sende
sine billeder. 12-bit-formater (`BayerRG12`, `Mono12`) overføres normalt som 16-bit
ord; pakket transport sender de samme 12 bit i 1,5 byte pr. pixel og
udpakker dem på værten, så det tager 25 % kortere tid at sende hvert billede — ca.
**33 % flere billeder pr. sekund ved fuld sensorudnyttelse** (målt: 10,7 → 14,0 fps på et
9-kamera-array). **De gemte data er identiske**: samme værdier, samme 16-bit
filer, samme metadata.

- `packed="auto"` (standard) komprimerer kun, når det er en fordel, **og** værten har
  kapacitet til at udpakke: ved opkobling måler backend sin egen udpakningshastighed,
  antal CPU'er, aktuel belastning og netværkskapacitet. En stationær computer med ledige kerner
  komprimerer ved fuld sensorbelastning; en travl eller strømbesparende vært (enkortcomputere,
  indlejrede controllere) streamer ukomprimeret. Små ROI’er og binne-rammer vinder kun lidt.
- `packed="off"` (eller `False`) komprimerer aldrig — for at udelukke det under fejlsøgning,
  eller på en vært, der skal holde sin CPU fri til andet arbejde.
- `packed="on"` (eller `True`) pakker altid 12-bit-formater.
- Enhver anden værdi udløser `ValueError`. 8-bit- og 16-bit-formater pakkes aldrig
  sammen.

`analyze_array_network(..., packed="auto")`-projekter med samme beslutning,
så dens fps er den hastighed, som arrayet kører med (svaret indeholder
`packed_transport` og `packed_reason`); dens `pixel_format` er som standard indstillet til det, som
connect anvender (Mono12 / BayerRG12) i stedet for 8-bit.

Beslutningen og begrundelsen herfor findes i sessionen:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` anvender den samme tilsidesættelse på alle
arrayer i et projekt (standard `None`, hvilket betyder `"auto"`: et projekt gemmer ingen
pakket indstilling pr. array).
For en standardindstilling, der gælder for hele maskinen, skal du indstille `CHLOROS_ARRAY_PACKED12=off` (eller `on`) i
backendets miljø; en eksplicit indstilling af `packed="on"` / `"off"` har stadig forrang.
`/api/camera/array/status` rapporterer `packed_transport` og `packed_decision`.

#### Forventede billedhastigheder

Målte billedhastigheder for arrayet (hardwaresynkroniseret, 12-bit; fuld sensor = 2048 × 1536). Brug dem til at kontrollere, om `status()["actual_fps"]`; `analyze_array_network()` estimerer billedfrekvensen korrekt for din egen vært og dit netværk.

| Opsætning | Array-billedfrekvens |
| --- | --- |
| Fuld sensor, PCIe 10 GbE-værtsnetværkskort, pakket transport (`packed="auto"` på en stationær computer) | ~14 fps |
| Fuld sensor, PCIe 10 GbE-værtsnetværkskort, ukomprimeret (`packed="off"`) | ~10,5 fps |
| Fuld sensor, 8 kameraer på en USB 10 GbE-adapter | ~3,8 fps |
| Fuld sensor, 1 GbE-værtsforbindelse | 1,2–1,7 fps |
| 2 × 2 binning (`binning=2`), 10 GbE-værts-NIC | ~30–33 fps |

- På en PCIe 10 GbE-forbindelse er det **kameraerne**, der sætter grænsen ved fuld sensor, ikke netværket: 4 og 8 kameraer kører med samme hastighed.
- **`target_fps` er udløserhastigheden, og arrayet kører aldrig hurtigere.** `target_fps=1` kører arrayet med 1 fps; `None` (standardindstillingen) kører det med den højeste hastighed, som værten og netværket kan håndtere. En pin, der ligger over denne hastighed, sænkes til den (se [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` af `4294` i `status()` er ikke en linkhastighed.** Det betyder, at Windows ikke kunne aflæse adapterens hastighed (driveren rapporterede den største 32-bit-værdi), hvilket er typisk for USB-Ethernet-adaptere. En USB-adapters anvendelige hastighed på Windows er uanset hvad begrænset til 200 MB/s (se [Array Health](#array-health--which-subsystem-is-losing-frames)); for at finjustere en sådan, se afsnittet "USB 10GbE-adaptere" i CLI-referencen under [Opsætning og finjustering af værts-NIC](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Der gælder også en USB-begrænsning under Linux: 400 MB/s på x86_64, 200 MB/s på NVIDIA Jetson.** Adapteren identificeres via sysfs (med `ethtool -i` som standard), så en USB 5 GbE- eller 10 GbE-adapter begrænses til denne grænse i stedet for den nominelle hastighed på 500 / ~1060 MB/s. Tallet for x86_64 blev målt på Linux: Den samme adapter, der ødelægger rammer over 200 MB/s på Windows, kørte et 4-kamera-array fejlfrit ved alle lofter op til 480 MB/s. På NVIDIA Jetson fungerede den samme adapter fejlfrit kun ved 200 MB/s og mistede rammer sporadisk ved højere hastigheder. Hvis din vært kan håndtere flere rammer uden `gvsp_corrupt`, skal du øge den med `wire_ceiling_mbps`. På Linux er der afsat 30 MB/s til et kort i en USB 2.0-port; sæt det i en USB 3-port.

#### Over-Subscription (minimumsgrænsen pr. kamera)

Et array fungerer på ethvert netværk; det er billedfrekvensen, der afgør det. Sim-emit-pacing tildeler hvert kamera en jævn andel af det kollisionssikre netværksbåndbreddebudget, og alle udløser stadig på samme trigger, så på et langsomt netværk er andelen — og billedfrekvensen — lille: 8 fulde-opløsning 12-bit-kameraer på 1 GbE uden jumbo-rammer får 6,5 MB/s hver, ca. **1 fps**(ca. 1,3 fps pakket). Andelen er fastsat til den laveste hastighed, som kameraerne kan afstemmes til (`per_cam_floor_bps`: læses fra hvert kameras GevSCPD-interval ved opkobling og er aldrig under 1 MB/s). Hvis netværket kræver en lavere hastighed, falder forbindelsen tilbage til**seriel**udsendelse (`slip-emit-and-capture`): kameraerne skiftes til at sende med fuld linkhastighed, så alle rammer ankommer, men medlemmerne**eksponeres ikke samtidigt**— responsens niveau og blokeringer (`SERIAL: ...`) angiver dette; overfør `force_tier="sim-capture-sim-emit"` for at afvise fallback. Ramme størrelse indgår ikke i fordelingen, så**binning og ROI øger billedfrekvensen, men ændrer ikke, om et array passer**. Kun en forbindelse, der ikke kan overføre arrayet ved nogen hastighed, hverken samtidigt eller skiftevis,**overbelastes** — fejltilstanden er GVSP-pakketab, ikke en lavere billedhastighed. Løsninger: færre kameraer, jumbo-rammer fra ende til ende eller et hurtigere netværkskort.

- Svarene `analyze_array_network()` og `/api/camera/array/connect` indeholder `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` og `per_cam_floor_bps`. Når `oversubscribed` er sand, **nulstiller projektionen fps-felterne** (`achievable_fps_max` / `fps_bright` / `fps_dark`) i stedet for at rapportere en vildledende hastighed, der er langsom, men fungerer.
- `POST /api/camera/array/connect` accepterer en `pin_resolution`-body-parameter (**kun HTTP — ikke en SDK-kwarg**; `connect_array` eksponerer den ikke). Det er streng tilstand: en overtegnet forbindelse med `pin_resolution` indstillet**afvises kategorisk** med en fejlmeddelelse, der angiver alle mulige løsninger. Uden denne indstilling fortsætter forbindelsen med en advarsel om, at arrayet vil miste pakker. Uanset hvad bevarer forbindelsen den anmodede rammestørrelse — den samler eller beskærer aldrig af sig selv.
- Kun til testformål: Indstil `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` i backend-miljøet for at nedgradere afvisningen til en tydelig advarsel — du opretter forbindelse alligevel og accepterer pakketabet.

#### Array-tilstand — hvilket undersystem mister rammer

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) bærer en aktiv `health`-blok på et
tilsluttet array, der revurderes i et rullende **10-sekunders** vindue. Den opdeler rammetab
i de to årsager, der kræver modsatrettede løsninger, i stedet for én »ufuldstændig«-rate, der
ikke angiver nogen af dem:

| Felt | Hvad det betyder | Hvilket undersystem |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (pr. serienummer) | Billedet **ankom og var strukturelt defekt**— GVSP-pakketab. |**Netværk**: ledningskapacitet, pacing, NIC RX-ring, MTU |
| `never_arrived_rate_pct` (pr. serienummer) | Rammen **kom slet ikke**— kameraet udløste ikke, eller der kom intet ud af det. |**Udløser / synkronisering**: M8-kabel, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Det dårligste kameraets hastighed for hver. | — |
| `per_cam_rate_pct` | Samlet ufuldstændighedsrate pr. kamera (begge årsager tilsammen). | — |
| `stable_for_seconds` | Hvor længe hvert kamera har ligget under 0,01 %. | — |

Sammen med `health` angiver den samme post det tal, som hele tildelingen afhænger af:

| Felt | Betydning |
| --- | --- |
| `wire_ceiling_mbps` | Værtens gældende vedvarende båndbredde, MB/s. |
| `wire_ceiling_source` | Hvor tallet stammer fra, i ord — f.eks. `USB-capped 200 MB/s (was theoretical 1062; …)` eller `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, når `wire_ceiling_mbps=` indstiller det. |
| `nic_is_usb` | `true` for en USB-Ethernet-adapter. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Fortolkning:** en værdi forskellig fra nul for `gvsp_corrupt_rate_pct`, mens `never_arrived_rate_pct` er 0, betyder, at
triggering og kabelsynkronisering er perfekte, og at 100 % af tabet ligger på netværksstien — sænk
`wire_ceiling_mbps` og tilslut igen. Det omvendte mønster peger i stedet på synkroniseringskablet eller
triggerledningen.

> **`target_fps` er ikke afgørende for korrupte rammer.** GevSCPD-pacing skrives én gang ved
> opkobling, så en sænkning af triggerfrekvensen ændrer duty cycle og ikke
> burst-frekvensen for samtidig udsendelse. En målt 5× reduktion af efterspørgslen gav ingen forbedring, mens
> sænkning af ledningsloftet fra 240 til 200 MB/s reducerede den samme rigs andel af korrupte rammer fra 10,4 % til
> 0,00 %.

> **Et kørende array kan ikke omplanlægge sig selv på LATTICE (M3C / M3M)-firmware.** Den kan ikke
> løse dette selv; afbryd og tilslut igen, så forbindelsestidsvælgeren genplanlægger i forhold til
> den nye grænse.

En **USB-Ethernet-adapter er begrænset til 200 MB/s** af sonden uanset dens
typenummer: den effektivitetstabel, der omregner en forbindelseshastighed til et vedvarende tal, er
afledt af PCIe, og et USB-netværkskort angiver sin Ethernet-forbindelseshastighed, mens det er begrænset af
USB-bussen og dens driver. Begrænsningen er absolut, ikke en brøkdel — en USB 1 GbE-adapter
opnår ~80 MB/s og påvirkes ikke. Det gælder for Windows og Linux (inklusive Jetson);
se [Forventede billedhastigheder](#expected-frame-rates) for, hvor det blev målt.

#### `ArraySession`-metoder

| Metode | Beskrivelse |
| --- | --- |
| `status(timeout=10.0)` | Live-status for det kørende array: `target_fps` / `target_fps_auto` (triggerfrekvensen og om frekvensregulatoren styrer den), `target_fps_clamped_reason` (hvorfor en fastsat frekvens holdes lavere, ellers `None`), `actual_fps` (målt; 0, når grab-loopen har givet op eller er gået i stå), `recommended_fps`, `streaming` (False, når grab-loopen har givet op — tjek dette, ikke kun `last_error`; forbliver sandt gennem en genstart af indstillingerne, hvilket `restarting` markerer), `stalled` (sandt, mens grab-loopen kører, men ikke har offentliggjort nogen billedgruppe i ca. ti billedperioder, 2–30 s; `last_error` angiver dette først efter 60 s), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` kun når arrayet var forbundet med PTP **og** hvert medlem sidst læste `PtpEnable` som aktiv; ikke det samme som låst, hvilket `ptp_epoch` måler), `ptp_configured` (hvad forbindelsen anmodede om), `ptp_off_members` / `ptp_unread_members` (medlemmer, der sidst læste `PtpEnable` som slukket / ikke svarede), `member_recovery` (`None`, eller et medlem, der har mistet strømmen eller sin forbindelse: `state`, `waiting`, mens det ikke svarer, `reopening`, derefter `recovered`, sammen med `members` og `why` — arrayet genåbner det, når det svarer igen, og giver det sin PTP-værdi fra forbindelsestidspunktet tilbage, trigger- og synkroniseringsledningsføring, tempo- og kameraindstillinger, hvorefter den streamer igen uden nye `connect_array`) og `ptp_epoch` (målt), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` betyder, at Windows ikke kunne aflæse adapterens hastighed — se [Forventede billedhastigheder](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (om strålingskæden pr. bånd kan bruges, og hvilket element der brød den), og `band_clamp` — mens en live-visning gengiver matrixen: den andel af oplyste pixels, som unblandingen har begrænset til 0, pr. element (`members`: `{serial: [band0, band1, band2]}` i kameraets R-, G-, B-båndsrækkefølge) og pr. bånd viser sammensætningen (`composite`: `{band name: share}`), hvor `warn` angiver den andel, som GUI'en markerer som »Bånddata med lav pålidelighed« (målt ca. en gang i sekundet; `None`, når der ikke kører live-visning), og `member_ae` — pr. medlem, eksponeringen (`exposure_us`) og forstærkningen (`gain_db`), som den seneste offentliggjorte ramme indeholdt, fra selve rammen (`read_nodes` af `ExposureTime` / `Gain` returnerer den sidste værdi, der blev skrevet, mens autoeksponeringen kørte, ikke hvad den gør). |
| `set_member_settings(serial, **kwargs)` | Skriv indstillingerne for et medlemskamera, som `CameraSession.set_settings()` henter dem (eksponering, forstærkning, mållysstyrke, ...); medlemmet forbliver i arrayet. Udløser `ChlorosConnectError` for en serienummer, der ikke er et medlem. Se [Eksponering på array-medlemmer](#exposure-on-array-members). |
| `capability(timeout=10.0)` | Det, som smart-prep har fastlagt (optagelsesniveau, billedstørrelse, `pixel_format`), den aktive `health`-blok og værtens båndbreddebudget. Se [Array-tilstand](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Ændrer triggerfrekvensen, mens arrayet kører, uden at genoprette forbindelsen. Et tal fastlåser den (frekvensregulatoren lader den være); `None` sætter den tilbage til auto. Returnerer `{requested, target_fps, target_fps_auto, clamped}` — `clamped` er True, når hastigheden er sænket til det, værten og netværket kan håndtere. Se [Intervaloptagelse til droneopmålinger](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Ét billede fra hvert kamera, **alle fra én udløser**, som numpy-arrays (ingen filer skrives). Returnerer en `GrabResult` (en `dict`): `{serial: array}` eller `{serial: (array, metadata)}` med `with_metadata=True`. `.group_id` er den trigger, som billederne deler; `.errors` knytter hvert udeladt kamera til årsagen (et kamera, der ikke kan betjenes ved den pågældende trigger, udelades og udfyldes aldrig fra et andet). `processing` er `raw` / `debayered` / `radiance` / `reflectance` / `display`. Den samme læsning som projekt-API'et [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) — én fælles implementering — så alle nøgleordene der (`band`, `align`, `bit_depth`, `refl_max`) fungerer her. Kræver `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iterator, der returnerer ét `grab()`-resultat pr. tick (kun nøgleordsargumenter), med en hastighed på `fps`, begrænset af `duration` (sekunder) og/eller `count` (`None` = ubegrænset). **Hvert returneret sæt er en ny trigger:** et tick, hvis nyeste trigger er den mængde, det allerede har genereret, venter på den næste trigger i stedet for at gentage den, så strømmen kører med den laveste af `fps` og arrayets egen billedhastighed. Udløser `FrameFetchError` i stedet for at gå i dvale: med det samme, når alle kameraer af en eller anden grund nægter at gentage det næste tick, efter 60 sekunder med ticks uden billede og efter 60 sekunder uden ny trigger („ingen ny trigger i 60 sekunder“). Samme implementering som `ArrayHandle.frame_stream()`. Se [Live-billeder i din egen pipeline](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Start backendens egen kontinuerlige optagelse (GUI'ens kontinuerlige tilstand) → [`ContinuousHandle`](#continuoushandle). Gemmer **alle** synkroniserede grupper, som arrayet offentliggør, som `capture()` ville gemme dem (samme eksportkontrol), indtil den stoppes eller indtil `max_captures` / `duration_s` (`None` = indtil den stoppes). Udløser `ChlorosConnectError`, hvis der allerede kører en for dette array. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | En synkroniseret indfangningsgruppe. Returnerer en `CaptureResult` (liste over frame-ordbøger + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Overfør et tidligere resultat'`group_seq` som `after_seq` for at hente den næste synkroniserede gruppe efter den i stedet for at vente på en ny. Eksportkontroller nedenfor. |
| `capture(..., smart=True)` | **Smart optagelse** — venter på, at AE stabiliserer sig på alle kameraer, og udløser derefter. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Hurtigst mulig optagelse: kun rådata + den tildelte DAQ-aflæsning (+ det frie kombinerede indeks). Spejler GUI-knappen »Hurtigst mulig optagelse«. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Enkelt / Kontinuerlig / Interval i én afgrænset løkke. Returnerer `list[CaptureResult]`.**Kræver `count` og/eller `duration_s`**, så den afsluttes (SDK'en har ingen Ctrl+C). Afsluttes også tidligt, efter gennemløbet, når et gennemløb rapporterer et fuldt drev (en `.failed`-post med `disk_full`); dette gennemløb er det sidste resultat. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Start optagelse af den kombinerede live-indeksvisning til video/GIF → `RecorderHandle`. Videoen er en Motion-JPEG-fil `.avi` (kræver ikke ffmpeg). Hvis der ikke er nogen GUI-visning åben, gengiver backend'en den: `expr` (et indeksudtryk i arrayets båndnavne, f.eks. `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`) eller som standard arrayets NDVI. Én sammensat optager pr. array. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Start en rå-Bayer-serieoptagelse med høj billedfrekvens → `RecorderHandle`. Genbehandl offline med `build_video()`. `orient=True`: De produkter, som `build_video()` fremstiller, drejes efter kameraets orientering (produkter pr. kamera efter deres eget kameras orientering, det kombinerede indeks efter masterkameraets); de rå billeder forbliver i deres oprindelige format, og burst-manifestet registrerer hvert medlems orientering. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Genbehandler en gemt rå burst-serie offline til kalibrerede videoer. Blokerer, indtil opgaven er færdig (`wait=True`), og returnerer byggeresultatet: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — se [byggeresultatet](#the-build_video-result) nedenfor. |
| `build_video_status(job_id, timeout=15.0)` | Afventer et offline-kompileringsjob: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Frigiver hele arrayet — **hvis denne session åbnede det** (`connect_array`): det er derefter `force_disconnect()`, og udløser `ChlorosConnectError`, når frigivelsen ikke bekræftes (udgangen fra `with`-blokken udløser aldrig en fejl). Ved et håndtag fra `attach_array` lader den arrayet køre videre og udskriver en linje til stderr, der angiver dette: arrayet kan tilhøre GUI'en eller et andet script. Hvert kamera efterlades i frit kørende tilstand og ellers i den tilstand, som forbindelsen fandt det i; `keep_settings=True` gemmer dets indstillinger i kameraets egen hukommelse, mens `clear_settings=True` nulstiller denne hukommelse til fabriksindstillingerne (se [Kamerahukommelse](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Frigiver hele arrayet, uanset hvem der har åbnet det. Brug det på et tilknyttet håndtag, når du ønsker at afslutte arrayet (f.eks. for at genoprette forbindelsen med andre indstillinger). Returnerer, når backend'en bekræfter frigivelsen, eller når der ikke er noget at frigive (arrayet er allerede væk, eller der kører ingen backend); en nedlukning, der er løbet ud, eller som backend'en har afvist, udløser `ChlorosConnectError`. Med begge flag returneres det, som hvert kameras hukommelse har fået (`camera_memory`), ellers `None`. |

`capture()` eksportkontroller (samme endpoint som GUI/CLI bruger):

- `processing` / `levels` — `processing="all"` (eller `levels=["raw","radiance",…]`) gemmer alle relevante eksporttyper pr. kamera; en enkelt `processing`-værdi gemmer kun netop det niveau.
- `aligned` — **aktiveret som standard (`None`), da Chloros-appens optagelse af det samme array er**(dens**justerede** optagelsesindstilling pr. array). Hvert medlems ikke-rå eksport tilpasses til masterkameraets ramme ved hjælp af arrayets [justeringsprofil](#array-alignment) og beskåret til medlemmernes fælles overlapning, så filerne er pixelregistrerede og kommer ud med den samme størrelse, beskæring og de samme `Alignment*`-tags som appens (inden for et ROI omregnes transformationen til vinduet); `XMP-mapir:AlignmentJSON` angiver `"applied": true` med dets `crop_roi`. Raw-filer forbliver uforvrængede i den oprindelige størrelse, men bærer transformationen i metadataene. `aligned=False` gemmer hver fil i den oprindelige størrelse, uforvrænget; `aligned=True` er standardindstillingen beskrevet i detaljer. `continuous()`, `capture(smart=True)`, `capture_repeated()` og projektniveauet `ArrayHandle.capture()` / `capture_smart()` anvender den samme standardindstilling. Uanset om forvrængningen anvendes eller ej, registrerer hver medlemsfil sit kameras `Alignment*`-transformation, så snart arrayet har en justeringsprofil — undtagen et kamera, som profilen ikke justerer (udeladt eller tvunget til identitet på grund af en mislykket løsning), samt referencekameraet, når intet andet kamera er justeret: disse har ingen `Alignment*`-tags. Uden en profil på arrayet gemmes optagelsen ujusteret, og resultatet angiver dette: `result.alignment` er `{"requested": True, "applied": False, "warnings": [...]}`, én advarsel pr. årsag til, at warp ikke kunne anvendes (ingen profil, medlemmer med forskellige opløsninger, et array, der ikke kan-synkroniseret, et billede, som profilen ikke kan kortlægges til). `applied` er kun `True`, når hver eneste eksport, der anmodede om forvrængningen, fik den. Arrayets automatiske justering opretter profilen, så snart eksponeringen er stabil; `ArrayHandle.calibrate_alignment()` / CLI's `align-calibrate` opretter en på forespørgsel. `result.oriented` angiver, om `orient=True` har roteret produkterne.
- `render_index=False` — spring over overlejringen af vegetationsindekset pr. kamera; standardindstillingen gengiver det, hvor det er konfigureret.
- `daq` / `daq_tolerance_ms` — DAQ-målingen gemmes som en `.daq`-sidecar som standard, selv når intet valgt niveau har brug for den; `daq=False` springer den så over, og `daq_tolerance_ms` indstiller matchtolerancen (standard 500). `force_daq=True` er den ældre stavemåde for standardindstillingen. Se [DAQ-målingen, som en optagelse gemmer](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
- `orient=True` — aktiver hvert medlems **produkter**(`display`, `radiance`, `reflectance`, indekset pr. kamera og det sammensatte billede) med kameraets orientering efter justeringsforvrængningen: med**masterens**orientering, da de blev justeret ind i dens ramme (standardindstillingen `aligned` — medlemmerne forbliver samregistrerede), ellers med deres eget kameras orientering.**Standard: hver medlemsfil i oprindelig retning**, som det altid er tilfældet med array-optagelser; `raw` og `debayered` forbliver i oprindelig retning uanset hvad. Hver fil registrerer retningen af den ramme, dens pixels befinder sig i (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — en RAW-fil, sit eget kameras), og XMP-filen `Alignment*` bevarer den oprindelige billedrammetransformation, der er anvendt før orienteringen; en RAW-fil, der indeholder denne transformation, registrerer også `ChlorosAligned*` (masterens), dvs. orienteringen af et produkt, der er forvrænget ind i masterens ramme. Behandlingen drejer hvert produkt med den vinkel, der hører til dets pixels: den justerede, når den forvrænger RAW-filen (Array-justering aktiveret), kameraets egen, når den ikke gør det. `capture_repeated()` og `capture_fastest()` lader den passere igennem; `ArrayHandle.capture()` / `capture_smart()` på projektniveau overtager den også.

**TIFF-komprimering (indstilling kun til HTTP):** `ArraySession.capture()` sender ingen `compression`-nøgle, så backendens standardindstilling gælder — `POST /api/camera/array/capture` læser en `compression`-body-parameter, `"none"` som standard: ukomprimeret, sensorens pixels præcis som de læses (~6,3 MB pr. billede i fuld opløsning). `"deflate"` (tabsfri zlib L1 + vandret prædiktor, ~4,1 MB/ramme) skriver mindre filer langsommere — begge er tabsfrie og læses identisk ved import. SDK'et stiller ingen kwarg til rådighed for dette; udvejen er `chloros-cli lattice array-capture --compression deflate` eller rå HTTP. DEFLATE låser Pythons GIL, så komprimerede skrivninger kan ikke paralleliseres på tværs af skrivetrådene pr. kamera — vedvarende optagelse i fuld opløsning ved sensorhastighed kræver standardindstillingen uden komprimering. Detaljer: [CLI-reference → array-capture](cli-reference.md).**På hinanden følgende grupper (HTTP):** Endpointet returnerer `group_seq`; en løkke, der sender det tilbage som `after_seq` ved sin næste anmodning, henter den næste synkroniserede gruppe — inklusive en, der blev offentliggjort, mens den forrige optagelse blev gemt — i stedet for at vente på en gruppe, der er offentliggjort efter anmodningen, og `skipped_groups` rapporterer alle de grupper, som løkken har overset. `capture_repeated`, CLI’s `--continuous` og GUI’ens kontinuerlige tilstand gør alle dette; intervalpassager forbliver opdaterede på hvert planlagt tidspunkt.**Overridelser af eksport pr. medlem (kun HTTP):**det samme endpoint accepterer også `exclude_serials` (liste — fjern medlemmer fra det gemte sæt; arrayet udløses stadig som én synkroniseret gruppe, og ekskluderede medlemmer returneres i `excluded`), `serial_levels` (`{serial: [level tokens]}`-tilsidesættelser på kameraniveau) og `serial_index` (`{serial: bool}`-tilsidesættelser af indeks-overlay pr. kamera). Dette er GUI-paritet-kropsparametre og**endnu ikke SDK-kwargs**; medlemmer, der mangler i kortene, falder tilbage til de array-dækkende `levels` / `render_index`.

##### Gennemgang af oversprungne kameraer — `CaptureResult.skipped`

`ArraySession.capture()` returnerer en `CaptureResult`, som er en `list`-underklasse: gennemløb den, indeksér den, `len()` — alle eksisterende mønstre fungerer fortsat. Ny kode kan undersøge `.skipped`-attributten for at se, hvilke kameraer der blev udelukket, og hvorfor. Det mest almindelige tilfælde er RGB-kameraer i et array med blandede filtre, når du anmoder om `processing="radiance"` eller `"reflectance"` — strålingsintensitet pr. Bayer-pixel er meningsløs for en bredbåndssensor, så backend-systemet springer disse kameraer over i stedet for at generere meningsløse data.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Årsagstokener følger mønsteret `<level>-not-applicable-to-rgb-cam` (én post pr. oversprunget niveau, hvor hver indeholder `level`). Et kamera, hvis kalibreringspakke ikke er indlæst, springer `radiance` / `reflectance` over med `<level>-needs-calibration (connect the camera so its cal pack loads)`, og hvis der ikke foreligger en brugbar DAQ-reflektansmåling, springes der over med `reflectance-skipped-no-dls`. De øvrige reflektansspecifikke spring er `reflectance-skipped-no-fresh-dls` (ingen ny nedadrettet måling tilgængelig), `reflectance-skipped-bound-daq-unavailable (…)` (den tilknyttede DAQ kunne ikke nås) og `dls-uncalibrated-band-<nm>` — båndet ligger for det meste uden for DAQ-lyssensorens radiometrisk kalibrerede område (~374–974 nm), så den absolutte DAQ-baserede reflektansopdeling afvises, og rammen nedgraderes tydeligt til sensorrespons. Blandt de udleverede SKU’er er det kun F988, der udløser dette; det pågældende kameras understøttede forløb er arbejdsgangen med reflektanspanelet.

##### Filer, der ikke kunne skrives — `CaptureResult.failed`

En optagelse, der kun kunne skrive nogle af sine filer, returnerer stadig: listen indeholder alle filer, der **blev** skrevet, og `.failed` viser resten, én ordbog pr. fil med `serial` (`None` for den kombinerede komposit og en arrays `.daq` sidecar), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, eller `match` — `.daq`'s `.match.json`-herkomst, når `.daq` blev skrevet, men dens sidecar ikke blev det) og `error`. På et fuldt drev indeholder posten også `disk_full: True`; enhver senere indlæsning til det pågældende drev ville mislykkes på samme måde, hvilket er grunden til, at `capture_repeated()` stopper der. Der udløses ingen undtagelse ved en delvis optagelse, og en fil i `.failed` blev ikke efterladt halvvejs skrevet på disken. `.failed` er tom ved en ren optagelse.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### Den DAQ, der læser en optagelse, gemmer — `.daq_files` og `.daq_matches`

Når en DAQ er bundet til kameraet eller arrayet eller er samlet i en pulje, gemmer hver optagelse den nedadgående måling, den anvendte: en `.daq` i optagelsesmappens `daq/`-undermappe, sammen med dens `.match.json` og dens `.cameras.json` (hvis måling det er). Dette gælder også for en rå eller debayered optagelse, og det er den samme fil, uanset om optagelsen stammer fra GUI’en, CLI’en eller SDK’en. `daq=False` springer det over for en optagelse, hvis niveauer ikke bruger DAQ’en (et `radiance`-, `reflectance`- eller `display`-niveau skriver det stadig); `daq_tolerance_ms` flytter tolerancelinjen for den pågældende optagelse og aldrig den anvendte måling. Hvilken DAQ: den, der er knyttet til kameraet eller arrayet (GUI’ens valg af lyssensor, et projekts binding eller `chloros-cli daq pool-bind`), ellers den nyeste, som backend’et har. CLI'en angiver kontrolelementerne `--no-daq` og `--daq-tolerance-ms` ([CLI-reference](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

En optagelse gemmer den lyssensorværdi, den anvendte, som en `.daq` sammen med en `<file>.daq.match.json` ved siden af, der angiver, hvordan værdien var knyttet til eksponeringen: `method`, `gap_ms` (fra eksponeringen til den nærmeste DAQ-måling) og `in_tolerance` (inden for 500 ms, eller optagelsens `daq_tolerance_ms`). Reflektansfilens XMP-data indeholder den samme vurdering som `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` og `Chloros:DlsInTolerance`.

En optagelse venter på DAQ'ens første måling efter eksponeringen, så den kan blande målingerne på hver side af den (`method` `interpolated`). En DAQ-E sender en måling hvert 0,4 s i dagslys og hvert 1–2 s indendørs, så `capture()` returneres op til ca. halvandet af disse intervaller senere; selve eksponeringen forsinkes ikke. Der ventes ikke på en DAQ, der er holdt op med at sende. En DAQ-E-måling er lyset gennemsnitligt over intervallet siden den forrige måling, så den placeres midt i dette interval: `gap_ms` måles til denne midte, og `.match.json`'`bracket_stamps_ns` giver målingernes egne tidsstempler. `ArraySession.continuous()` venter uden at sænke optagelseshastigheden. `capture_repeated()`'s-til-bag-gennemløb venter ikke, så de bevarer deres hastighed og matches med den nærmeste måling før eksponeringen.

En måling udløber aldrig, så ingen ramme går tabt på grund af en forsinket måling — den mærkes i stedet:

- **DAQ'ens strøm er gået i stå** — den nyeste måling anvendes, og `gap_ms` angiver, hvor gammel den er (`in_tolerance: false` for over 500 ms siden).
- **Ingen DAQ at aflæse**— afbrudt (`disconnect()`, `chloros-cli daq pool-disconnect`), eller den tilknyttede DAQ kan ikke nås — kameraets sidste måleværdi genbruges og beskrives i forhold til**denne** eksponering: `method`, `retained`, `gap_ms` fra denne eksponering, `in_tolerance` genberegnes, og `retained_from_epoch_ns` angiver den eksponering, den først blev matchet med. Et kamera, der aldrig har haft en måling, får stadig ingen reflektans, og resultatet angiver årsagen (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, eller optagelsens afvisning).

Filer, som en optagelse gemmer i mappen til det projekt, der er åbent i Chloros, føjes til projektets filliste, ligesom en optagelse i appen gør (filer, der er gemt andetsteds, forbliver uberørte): `CameraSession.capture()`, `ArraySession.capture()` (også `smart=True`) og `capture_cameras_transient()`. Resultatet fremgår af `.project_add` (`capture_cameras_transient()`: ordbogens `project_add`) -- `held`, `files` og backendens `message`; `held` er `True`, mens en behandlingskørsel er i gang, og filerne tilføjes, når den er afsluttet. `None`, når ingen filer er blevet tilføjet til projektet.

`CameraSession.capture()` og `ArraySession.capture()` returnerer dem som resultatet `.daq_files` og `.daq_matches`, da HTTP-indfangningssvarene (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) indeholder `daq_files` og `daq_matches`, én pr. `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` og `stream_fault` — DAQ'ens egen strømfejl, der kun kontrolleres for måleværdier uden for tolerancen. Inden du stoler på reflektansen fra en registrering, skal du kontrollere `in_tolerance` i dens `.match.json` eller XMP, eller DAQ'ens [`status()`](#daqsensorsession-methods).

`processing`-niveauer:

| Niveau | Output |
| --- | --- |
| `"raw"` | Enkeltkanals Bayer (monokameraer: det enkelte bånd) direkte fra sensoren. |
| `"debayered"` *(standard i SDK)* | 3-kanals farve: kalibreringspakkens mørke offset (DSNU) trukket fra rådataene, derefter OpenCV's kantbevidste demosaik — ingen linearitets-LUT, flat field eller unmix (monokameraer: 1-kanals gråskala). Den gemte TIFF-fil er i **RGB**-kanalrækkefølge (fotometrisk RGB, så enhver billedfremviser viser ægte farver); de arrays, som `grab()` / `frame_stream()` returnerer i hukommelsen, er i OpenCV's**BGR** — vend kanalerne om (`frame[..., ::-1]`) for at sammenligne med en gemt fil. `chloros-cli process` skriver det samme resultat som `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm via den fulde radiometriske kæde. Kun multispektral — RGB-kameraer springes over. |
| `"reflectance"` | uint16 0..32768 (Pix4D-klar); kræver en live DAQ-parring for absolut reference. Kun multispektral. |
| `"display"` | Fuld kæde, der matcher GUI-forhåndsvisningen (CCM + WB + gamma i henhold til kameraets profil). |
| `"all"` | **Én fil pr. relevant niveau** for hvert kamera (svarende til GUI-indstillingen »Capture All« / CLI-standard). Den returnerede `CaptureResult` indeholder derefter én frame-dict pr. `(cam, level)`, med niveauet angivet i hver dict; ikke-gældende niveauer vises i `.skipped`. Den DAQ, der læser den anvendte optagelse, gemmes som en `.daq`-sidecar. |

> **Bemærk — standardindstillingen adskiller sig fra CLI'en.** `ArraySession.capture()` er som standard indstillet til `processing="debayered"`; kommandoen `chloros-cli lattice array-capture` er som standard indstillet til `processing="all"`. Overfør `processing="all"` eksplicit fra SDK for at afspejle CLI/GUI-gemningen på flere niveauer.

##### Filnavne for array-optagelser

Hvert niveau gemmes i sin egen undermappe under `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` for `display`, `index/`, `composite/`). Hver fil navngives efter sit kamera som `SN<serial>-<filter>`: først serienummeret, derefter filterdelen af kameraets model — `SN214000533-FOCN` for et `M3C-L87-FOCN`, `SN261902142-F450` for en `M3M-L87-F450` — sammen med `-`, så et navn har de samme `_`-adskilte felter som før. Et kamera, hvis model ikke kan aflæses, er blot `SN<serial>`, og optagelser fra tidligere udgivelser har intet filter i navnet. Filnavnet afhænger af, hvor mange niveauer et medlem blev bedt om:

| Medlem anmodet om | Filnavn | Eksempel (`reflectance`, standard `format="tiff"`) |
| --- | --- | --- |
| **Ét niveau** (`processing="reflectance"` eller et `levels` med ét element) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — intet niveausuffiks, som i tidligere udgivelser, så eksisterende scripts fortsat fungerer | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Flere niveauer** (`processing="all"` eller `levels` med mere end én post) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

[Clamp-masken](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) følger efter produktnavnet, `clamp_masks/<product folder>/<product name>_clampmask.tif`: en enkeltreflektansoptagelse på ét niveau skriver `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, mens `processing="all"` skriver `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. Reglen for suffikset fastlægges for hvert medlem ud fra de niveauer, der blev anmodet om, så et udeladt niveau (f.eks. reflektans på et RGB-kamera) ændrer det ikke. Læs stierne fra de returnerede rammer i stedet for at opbygge dem. En optagelse med et enkelt kamera (`CameraSession.capture()`) tilføjer altid niveauet: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### PNG/JPEG: Vælg kun visnings- og indeksprodukter

Et PNG- eller JPEG-billede rummer 8 bit pr. kanal. Anmodninger om et billede med et niveau, som formatet ikke kan rumme, afvises, før der overhovedet indlæses noget, med en meddelelse, der angiver niveauet og TIFF som det format, der kan rumme det:

- **`radiance`**(float32) og**`reflectance`**(uint16, `32768` = 1,0) — og dermed også `processing="all"`, som omfatter dem. SDK'et kontrollerer dette på argumenterne: `capture()` udløser**`ValueError`** uden en anmodning (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` fra et kamera, der streamer mere end 8 bit** (12-bit M3C, Mono12 M3M) — backend'et afviser disse, da det kender hvert kameras bitdybde: HTTP-statuskode 400 med `error_code` `FORMAT_CANNOT_HOLD_LEVEL`, genereret som `ChlorosConnectError` (`CameraSession` / `ArraySession`) eller `RuntimeError` (`CameraHandle` / `ArrayHandle`). `debayered`-optagelsen på et enkelt niveau fra et kamera henter et 8-bit-billede til PNG/JPEG og afvises ikke.

Gem radiometriske data som `format="tiff"` (`ext=".tiff"`); brug PNG/JPEG til `display` og indeksrenderinger.

#### Sådan fungerer automatisk eksponering

- **Eksponering først, forstærkning sidst.** Når forstærkningen er indstillet til Auto, forbliver den på 0 dB, indtil eksponeringen har nået den længste tilladte værdi.
- **En manuel eksponering lader forstærkningen forblive på »Auto«**, indtil du ændrer forstærkningen eller indstiller den til »manuelt«. Når den er tilbage på »Auto«, er forstærkningens grænse den samme som før.
- **Små højlys gør ikke scenen mørkere.** Et spejlblankt område på op til 1 % af billedet ignoreres, og i panelerne på et kendt kalibreringsmål ignoreres op til 0,5 % af et panel.
- **DAQ-eksponeringsloftet gælder kun, mens et kalibreringsmål er i billedet.** Det forhindrer, at et hvidt panel bliver overskygget; når der ikke ses noget panel, eksponerer kameraet for scenen.
- **Konstant lys forbliver stabilt, og ændret lys stabiliserer sig på få sekunder.** Ved konstant lys forbliver eksponeringen og dens grænse uændret, og efter en stor ændring i lyset stabiliserer billedet sig inden for få sekunder. En udklipning begrænser ikke længere eksponeringen permanent, og en ændring af ROI finder eksponeringen igen.
- **Lysstyrkemålet forbliver inden for rækkevidde.** Det stiger ikke, mens kameraet er ved sin eksponeringsgrænse, og dets forstærkning kan ikke stige, så et lys, der vender tilbage, ikke blænder billedet.
- **Et kamera, hvis automatiske eksponering holder op med at reagere, holdes på det niveau, der passer til det lys, det befinder sig i.** I stærk sol kan et kamera falde til sin korteste eksponeringstid og forblive der med et mørkt billede, selvom alle indstillingerne viser normale værdier. Chloros overvåger billedet, holder kameraet på det niveau, det havde for det pågældende lys, og slipper det, når lyset ændrer sig. En mørk eller tildækket lyssensor forstyrrer ikke dette.

#### Eksponering på array-medlemmer

Et LATTICE-kamera begrænser `ExposureTime` til én periode af dets billedfrekvens. Mens et array streamer, kører Chloros hvert medlem med `AcquisitionFrameRateEnable` aktiveret, med en hastighed, der ligger lige over udløserhastigheden, så et medlems eksponeringsloft ligger lidt under udløserperioden, **ved enhver ROI**:

- Et elements billedperiode er triggerperioden minus en slæk på 15 % af perioden (mindst 3 ms, højst halvdelen af perioden), og dets eksponeringsloft er den pågældende billedperiode minus ca. 0,1 ms. Eksempel: En udløser på 10 fps (100 ms) giver en medlemshastighed på ca. 11,8 fps og et eksponeringsloft på ca. 85 ms.
- **En manuel `ExposureTime`, der er længere end denne grænse, sænker udløsningshastigheden for at tilpasse sig den.** Chloros åbner først medlemmets grænse for den pågældende værdi og lader den lavere hastighed køre i et par udløsningsperioder (skrivningen kan tage et par sekunder længere), hvorefter den skrives, og udløseren affyres ikke hurtigere, end eksponeringen tillader. På et array med automatisk hastighed vender den tidligere hastighed tilbage, når hvert medlem, hvis manuelle eksponering bremsede den, er tilbage på automatisk eksponering. En værdi, som kameraet selv ikke kan registrere, afvises stadig, idet det nu accepterede interval vises i fejlmeddelelsen, og medlemmets automatiske eksponering genindstilles, hvis den kun var slået fra i forbindelse med den pågældende skrivning. Før version 1.3.0 begrænsede et lille ROI hvert medlem til sensorens egen frie løbeperiode, uanset triggerfrekvensen: 4,9 ms ved 768 × 304, så intet medlem kunne gøres lysere.
- Chloros forkorter aldrig en manuel eksponering for at tilpasse sig en hurtigere udløserhastighed. En automatisk hastighed holder sig inden for det, de manuelle eksponeringer tillader; en fastsat hastighed, der er hurtigere end en eksponering tillader, giver det pågældende medlem et loft, der er langt nok til den, og det accepterer ikke udløsere, der er hurtigere, end eksponeringen tillader. Sænk eksponeringen eller den faste hastighed, når hvert medlem skal nå alle udløsere.
- **Tilbagelæsning:** Når man læser et medlems noder, vises `AcquisitionFrameRateEnable` True og `AcquisitionFrameRate` med medlemmets hastighed; aktiveringen slås fra igen, når arrayet stopper.
- **Arrayet ejer disse to noder. Skriv ikke `AcquisitionFrameRateEnable` eller `AcquisitionFrameRate` på et array-medlem** — indstil i stedet arrayetshastighed med `target_fps` / `set_target_fps()` i stedet.

Skriv et medlems eksponering med `arr.set_member_settings(serial, exposure_time=20000)` (nøgleordene i `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` gør det samme). Medlemmet forbliver i arrayet. Åbn ikke et array-medlem med `connect_camera()`: connect afviser et kamera, der allerede er i en matrix, og den `CameraSession`, den ville returnere, fjerner kameraet fra sin matrix, når den afbryder forbindelsen. `ArraySession.status()["member_ae"]` rapporterer hvert medlemseksponering og forstærkning fra dets nyeste billede.

### Én projektpost, alle tilstande

En projektmappe indeholder alle enheders indstillinger: `cameras.json` — hvert kameras (`cameras[].settings`, for et enkelt kamera og for hvert array-medlem) og hvert arrays array-dækkende indstillinger (`arrays[].capture`: format, ROI, binning, triggerfrekvens, ledningsbudget) — samt `sensors.json`, der indeholder indstillingerne for hver lyssensor (integrationstid, billedgennemsnit, cap). Appen, CLI’en, SDK’en og MCP-serveren læser og skriver alle de samme filer på samme måde:

- **En enhed, der er tilsluttet under et projekt, gemmes i dette og registrerer alle ændringer i det.** »Under et projekt« betyder, at der er et projekt åbent i backend (appens), eller — via en `ChlorosProject` (`open_project()`), som angiver sin egen mappe til backend (`project_path`). Et kamera, en array eller en lyssensor, som projektet endnu ikke indeholder, føjes til dets filer, når den tilsluttes (`connect_camera()`, `connect_array()`, lyssensorens tilslutning, CLI'en, appen, `chloros-mcp`) sammen med sine indstillinger. En ændring fra en hvilken som helst tilstand (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, array-indstillingerne, `set_cap()`, CLI'en, appens kontrolelementer) gemmes i projektets filer et øjeblik senere. En eksponering, forstærkning eller et AE-mål, der er indstillet på et kamera, gemmes som en indstilling, så genåbning bevarer den igen.
- **Når man åbner et projekt, indstilles enhedernes indstillinger ud fra projektet**, uanset tilstand: et kamera, et array eller en lyssensor, der er tilsluttet uden egne indstillinger, får projektetsgemte indstillinger (`lattice cam-connect` / `array-connect` med et åbent projekt, `chloros-cli project connect`, `open_project().connect_all()`, appens genåbning). Et array, der tilsluttes under et nyt id, opdaterer det array, som projektet har gemt med de samme kameraer; et array med andre kameraer tilføjes ved siden af det.
- **Appens egen gemmefunktion overskriver aldrig en enheds aktuelle indstillinger** med sin ældre kopi af dem og bevarer de overskrivninger pr. medlem, som en CLI- eller SDK-kaldende part har gemt.
- **Det pixelformat, du indstiller, er det, som projektet bevarer.** Et format, der er indstillet i Array-indstillinger, via `--pixel-format` eller via `connect_array(pixel_format=)`, inklusive 8-bit, gemmes og genforbindes nøjagtigt. Hvis der ikke er angivet noget, opretter hvert kamera forbindelse med sin 12-bit standardindstilling (`BayerRG12`, `Mono12` på en monosensor). Et formatnavn angiver dets bitdybde; familien er sensorens, så et farvekamera indstilles aldrig til et monofarveformat eller et monokamera til et Bayer-format, og linjen `[PixelFormat <serial>] ... written` angiver dette, når et format er blevet tilknyttet. Eksportfiler fra en 8-bit-optagelse er 8-bit, inklusive reflektans: omskaleret og mærket med `PixelScale`.
- En enhed, der er tilsluttet uden et projekt, registrerer intet.
- **En optagelse følger et projekt, der er oprettet efter den; en åbning efterlader den med sit projekt.**Når der oprettes et projekt — `create_project()`, `chloros-mcp`’s `create_project` eller appens**Nyt projekt**— eller duplikeret (appens**Dupliker**, som åbner kopien), mens en optagelse stadig skriver til det projekt, der var åbent før, fortsætter optagelsen, og det, den skriver, ender i det nye projekt, på samme sted i det (nedenfor). Åbning af et eksisterende projekt — `ChlorosProject.process()` (hvilket åbner projektet i baggrunden), `chloros-cli process`, appens**Åbn projekt** — efterlader en optagelse, der stadig kører, sammen med det projekt, den optager til: den afsluttes der, og dens filer føjes til det pågældende projekts filliste (appen stopper alle optagelser, før den åbner et projekt).
  - `ArraySession.record()` og `ArraySession.burst()`: videoen eller burst-mappen med alle billeder flyttes derhen i sin helhed, når optagelsen stopper. Stopresultatet angiver den nye sti (`out_dir`, `video_path`), og en samling af burst-billederne oprettes ud fra den. Hvis drevet afviser flytningen, forbliver optagelsen, hvor den blev skrevet, og resultatet angiver dette (`relocation_error`). De rå billeder fra en stoppet burst og `.daq` føjes til fillisten i det projekt, den ender i; en video tilføjes ikke til et projektets filliste.
  - `ArraySession.continuous()`: alle optagelser efter ændringen gemmes i det nye projekt; dem, der er gemt før, forbliver, hvor de er.
  - `DAQSensorSession.record_start()` (og appens egen lyssensoroptagelse): `.daq` flyttes til det nye projekts `light_sensor`-mappe, når optagelsen stopper; `record_stop()` returnerer den nye sti, og filen føjes til det pågældende projekts filliste.

  En optagelse i en mappe uden for det åbne projekt forbliver, hvor den er, og `start_camera_continuous()` / multi-stream-opgaver beholder den mappe, de blev tildelt.
- **I appen placeres optagelserne efter oprettelsen af et nyt projekt i dette.** Oprettelse eller duplikering af et projekt fortsætter alle igangværende optagelser, og hver optagelse, der foretages efterfølgende (Capture All, Start Capturing), gemmes i det nye projekt og føjes til dets filliste. Åbning af et eksisterende projekt i appen stopper først dem alle.
- **Et projekt flyttes ikke, mens det optager.**Flytning af det åbne projekt til et andet arbejdsbibliotek (appens**Indstillinger → Arbejdsbibliotek**; ingen SDK-kald flytter et projekt, en HTTP-klient, der sender til `/api/move-project`, gør det) afvises med HTTP 409 `PROJECT_RECORDING`, mens en array-optagelse eller burst, en kontinuerlig array-optagelse, et kontinuerligt kamera-/multistream-job eller en lyssensoroptagelse skriver til det: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Dette er uafhængigt af, hvordan kameraerne er indstillet ved afbrydelse af forbindelsen (se [Kamerahukommelse](#camera-memory--keep_settings--clear_settings)): et projekt indstiller selv sine kameraer, når det åbnes.

### Kamerahukommelse — `keep_settings` / `clear_settings`

Når et array tilsluttes, overtager dets kameraer arrayets indstillinger, og du kan derefter ændre dem. Når Chloros frigiver kameraerne — `ArraySession.disconnect()` / `force_disconnect()`, afslutningen af en midlertidig funktion, afbrydelsen af `start_camera_continuous()` — sætter det hvert enkelt kamera tilbage til den tilstand, det var i før tilslutningen: dets trigger- og synkroniseringsledningsforbindelser, PTP- og kameraindstillinger — `ExposureAuto` / `GainAuto` (samt en manuel `ExposureTime` / `Gain`), `TargetBrightness` samt AE-grænserne og dæmpningen, hvidbalancen, sortniveauet, `PixelFormat`, binning og ROI, farvepipeline (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, PTP's `PtpSlaveOnly` / `PtpDomainNumber`, pakkestørrelsen (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), stimuleringsfrekvensen (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), chunk-dataene (`ChunkModeActive`, hver `ChunkEnable` under sin `ChunkSelector`), deglitching af trigger-indgangen, hvor en matrix giver en slaves synkroniseringslinje (`LineFilterSelector` / `LineFilterWidth`) og indsamlingsrammefrekvensen (`AcquisitionFrameRate` kun, hvor aktiveringen var slået til: når den er slået fra, angiver frekvensen det maksimale, som takten tillader, ikke en indstilling). Kun det, der adskiller sig, skrives; kameraet stoppede først, takten og billedfrekvensen før en eksponering, der kræver deres billedperiode.

Når kameraerne bruges uden for et projekt, skriver to argumenter kameraets **egen hukommelse** — det, det starter op med, i enhver software (på `disconnect()` / `force_disconnect()`, alle midlertidige funktioner, `start_camera_continuous()` og `ChlorosProject.disconnect_all()`; MCP: de samme argumenter som for `disconnect_device` og de midlertidige værktøjer; CLI: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** gemmer hvert kameras indstillinger, som de var ved afslutningen af sessionen, i dets hukommelse (UserSet2, der udgør opstartsindstillingerne). Kameraerne kører stadig frit (`TriggerMode` `Off`), og synkroniseringsledningsføringen er som fundet. Et kamera uden UserSet2 gemmer i UserSet1; denne computer husker det, så Chloros' array-selvhelbredelse — som nulstiller et UserSet1, der er afveget fra Chloros' baseline — lader det være i fred.
- **`clear_settings=True`** nulstiller hvert kameras hukommelse til fabriksindstillingerne: Chloros’ referenceværdi, den som arrayets selvhelbredelse indarbejder — fabriksindstillinger med `TriggerMode`, `Off`, `TriggerSource`, `Line2`, automatisk eksponering, et RGB-kameras kalibrerede hvidbalance bevaret — i UserSet1, der udgør opstartsættet.
- Ikke begge (`ValueError`; backend svarer 400). Svaret indeholder `camera_memory`, pr. kamera `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` eller `{"cleared": bool, "errors": [...]}` — `force_disconnect()` returnerer det; for en opgave følger det med `stop()`.

Appen tilbyder ikke disse.

**Hver udgang sætter kameraerne tilbage.**Tilbageførslen i den tilstand, de blev fundet, kører på hver udgangsvej: `CameraSession.disconnect()` (appens kamera**X**), appens**X** på en matrix eller et hvilket som helst matrixelement (hele matrixen frigives), lukning af appen (backend får op til 20 s til at genindsætte alle kameraer, matrixer og kørende kontinuerlig optagelse; en midlertidig funktion eller et engangsopkald, der stadig kører, stopper ved det næste sikre punkt, genindsætter først sine kameraer, i op til 8 sekunder, og returnerer HTTP-kode 503 `SHUTTING_DOWN`; en `connect_array()`, der stadig kører, stopper før sit næste kamera og lukker de kameraer, den har åbnet, i op til 10 sekunder, og genererer `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); og en, der ankommer under nedlukningen, åbner intet: HTTP 503 `SHUTTING_DOWN`), samt en `chloros-cli`-kommando, der afbrydes, eller hvis terminal lukkes (dens opgave stoppes, og dens kameraer lukkes først). Et backend, der går ned eller afbrydes tvungent, kan ikke lukke noget; en genstart returnerer hvert kamera til dets hukommelse.**Et projekt indstiller selv sine kameraer.** Når et array opretter forbindelse via et projekt — appens åbne projekt eller et `ChlorosProject`, som navngiver sig selv over for backend'en (`project_path`) — får hvert medlem de indstillinger, som projektet har gemt for det: en eksponering, forstærkning eller et AE-mål, der holdes fast på det (holdt som dit eget, så automatisk eksponering lader det være i fred) og dets hvidbalance-tilstand (Chloros' korrektion; kameraets egen hvidbalance forbliver slået fra), uanset hvad kameraets hukommelse indeholder.

### Optagelsestilstande og optagere

Arrayets overflade afspejler GUI-optagelsespanelet: Enkelt / Kontinuerlig / Interval / Hurtigste lukkertilstande samt to optagere (live-kompositvideo og rå burst → offline-genbehandling).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**er SDK’s kontinuerlige/interval-loop. Da der ikke findes et `Ctrl+C` til at afbryde det fra et script,**skal** du overføre `count` og/eller `duration_s` (den stopper, når en af dem nås). `interval_s` måles fra starten af hvert gennemløb (svarende til GUI'en). Kontinuerlige gennemløb (`interval_s=0`) kæder `group_seq` → `after_seq` sammen, så de udgør på hinanden følgende synkroniserede grupper i arrayetshastighed. De resterende kwargs sendes direkte videre til `capture()`.
- **`continuous`**er backendens egen kontinuerlige optagelse — GUI’ens kontinuerlige tilstand.**Brug den i stedet for `capture_repeated()`, når du ønsker, at hver udløser skal gemmes på disken med en høj billedhastighed:** `capture_repeated()` foretager én HTTP-anmodning pr. gruppe fra dit script, mens `continuous()` holder to optagelser i gang inde i backend'en uden rundtur mellem grupperne. Målt ved ROI 512 på et array med 9 kameraer: 73–77 optagelser/s, sammenlignet med et array, der udgiver ca. 78. Behold `capture_repeated()` til intervaloptagelse (`interval_s`) og til gennemgange, som du inspicerer én for én. På Windows skal du udelukke backend'en fra Microsoft Defender for at opnå høje hastigheder (se [Trin 1](#step-1--install-the-chloros-platform-package)).
- **`record`** er *overvågningsklasse*: den registrerer den gengivne live-sammensatte indekskomposit. Når GUI'ens kombinerede visning er åben, registrerer den den visning, som den vises. Hvis ingen er åben, renderer backend'en selv visningen: `expr`, hvis du sender en, ellers arrayets NDVI (de røde og NIR-bånd, der ligger tættest på 660 nm og 850 nm, som livevisningen registrerer dem), når arrayet er justeret. Navngiv båndene i `expr`, som arrayet navngiver dem: `<band>_<filter>` for et M3C-kamera (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` for et M3M (`Mono_850`); et kort farvenavn (`NIR`, `Red`) fungerer kun, når ét bånd i arrayet har det. En GUI-visning, der åbnes under optagelsen, optages, så længe den forbliver åben. En optagelse, der ikke skriver nogen ramme, mislykkes med `error`, `"no-frames: …"` (arrayet kan stadig være i færd med at justere eller stabilisere sin eksponering), og `stop()` udløser en fejl (se [`RecorderHandle`](#recorderhandle)). Én sammensat optager pr. array (udløser en fejl, hvis der allerede kører en).
- **`burst` → `build_video`** er *analysekvalitet*: `burst` skriver rå billeder + et manifest pr. billede + én `.daq` pr. unik DLS-aflæsning under `<output>/bursts/<base>/` ved grab-loopens fulde hastighed (ingen kæde, ingen exiftool, ingen live-visning). `build_video` tidssammenkobler hvert billede med det nærmeste `.daq` og kører importpipelinens radiance/reflektans/indeks-kæde. `products` er en liste over `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (standard: det kombinerede indeks). `burst().stop()` starter også automatisk en best-effort-opbygning af det kombineredeindeksopbygning, der returneres som `build_job` i stopresultatet.

##### `build_video`-resultatet

| Nøgle | Betydning |
| --- | --- |
| `success` | `True` kun når alle anmodede produkter blev opbygget som anmodet: mindst én output, intet i `skipped`, og ikke i `degraded`. |
| `outputs` | Stier til de videoer (og GIF'er), der er skrevet. |
| `errors` | Problemer, som tekst. |
| `skipped` | Én `{"serial", "level", "reason"}` pr. anmodet produkt, der **ikke** producerede nogen video — et kameras (intet kalibreringssæt til det pågældende kamera, ingen DLS-aflæsning for reflektans, fejl i behandlingskæden) eller det kombinerede indeks, som indeholder `"kind": "combined"` og `serial` `None` (intet rødt/NIR-båndpar blandt kameraerne, ingen gruppe med strålingsintensitet for begge bånd). Enhver post genererer `success` `False`; de øvrige videoer skrives stadig og vises på listen. |
| `substitutions` | Kameraer gengivet på et andet niveau end det anmodede, og hvorfor — et RGB-kamera, der anmodes om radiance/reflektans, får sin visningsvideo, hvidbalanceret når kameraets kalibreringspakke er tilgængelig. Ikke en fejl. |
| `notes` | Informativ tekst, ikke fejl — f.eks. har et kamera uden et indeksudtryk pr. kamera ingen indeksvideo, eller et RGB-kameras visningsvideo er ikke hvidbalanceret (ingen kalibreringspakke eller en uden fabriksindstillet hvidbalance), hvor kameraet navngives. Ikke en fejl. |
| `degraded` | `True`, når et produkt blev skrevet, men ikke er det anmodede: det kombinerede offline-indeks er den indbyggede normaliserede forskel mellem rødt og NIR, så et konfigureret indeksudtryk evalueres ikke. Giver `success` `False`. |
| `combined` | Oversigt over det kombinerede indeks: de anvendte bånd, LUT-området, opbyggede billeder. |
| `clamp_masks` | Stier til [unmix-clamp-masker](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) for de-frame kalibrerede TIFF-filer (`save_tiffs=True`), i `clamp_masks/<level>/` ved siden af TIFF-filernes `<level>/`-mappe, én pr. TIFF, hvor en pixel er blevet clamped. En video indeholder ingen. |
| `clamp_fractions` | Hvad en video angiver i stedet for en maske: én `{"kind", "serial", "level", "frames"}` pr. videoprodukt (`kind` `"combined"` for de kameraer, der indgår i det samlede indeks), `frames` en liste over `[frame, R, G, B]` — den pågældende rammes andel af de pixels, som unmix har begrænset til det bånd, som dens R/G/B-kanal rummer (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Returneres af `ArraySession.record()` og `ArraySession.burst()`. Brug den som en kontekstmanager til automatisk at stoppe ved afslutning af scope, eller styr den manuelt.

| Medlem | Beskrivelse |
| --- | --- |
| `job_id` | Backend-job-id (str). |
| `kind` | `"composite"` (fra `record`) eller `"raw"` (fra `burst`). |
| `start_stats` | Den ordbog, der returneres af `start`-kaldet. |
| `result` | `None` under kørsel; den endelige stop-resultat-ordbog, når optagelsen er stoppet. |
| `stats(timeout=10.0)` | Live jobstatistik (skrevne billeder, realiseret fps, forløbet). |
| `stop(timeout=60.0)` | Stopper optageren; returnerer og gemmer det endelige resultat (`.result` forbliver `None`, hvis selve stopanmodningen mislykkedes). For en `record()`-optagelse, der mislykkedes — en, der ikke skrev nogen rammer (`error` `"no-frames: …"`) — caches resultatet på `.result`, hvorefter der udløses en undtagelse på `ChlorosConnectError`. Idempotent (et andet kald returnerer det cachelagrede resultat uden at udløse en undtagelse). Hvis man forlader en `with`-blok, stopper optageren uden at udløse en fejl, så tjek `rec.result.get("error")` efterfølgende. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Returneres af `ArraySession.continuous()`. Brug den som en kontekstmanager — hvis du forlader `with`-blokken, stopper optagelsen, hvis den stadig kører — eller styr den med `status()` / `wait()` / `stop()`. Der kører én kontinuerlig indsamling pr. array ad gangen.

| Medlem | Beskrivelse |
| --- | --- |
| `start_status` | Statusordbogen, der returneres af startkaldet. |
| `result` | `None` under kørsel; den endelige statusdiktionær, når indsamlingen er afsluttet (indstillet af `stop()`, eller af et `status()` / `wait()`, der registrerede afslutningen). |
| `status(timeout=10.0)` | Live-statusordbog (felter nedenfor). |
| `wait(timeout=None, poll_s=0.5)` | Blokér, indtil indlæsningen afsluttes af sig selv — `max_captures` eller `duration_s` er nået, arrayet er væk, et fuldt drev, en fejl — og returner den endelige status. Med `timeout` returneres den seneste status efter det angivne antal sekunder, selvom den stadig kører: se `running`. |
| `stop(timeout=30.0)` | Stop optagelsen og returner dens endelige status. Idempotent (et andet kald returnerer det cachelagrede resultat). |

Statusfelter:

| Felt | Betydning |
| --- | --- |
| `running` | Sand, mens indlæsningen kører. |
| `captures` | Synkroniserede grupper gemt. |
| `files` | Filer skrevet. |
| `failed_files` | Filer, der ikke kunne skrives (som i [`CaptureResult.failed`](#filer-der-ikke-kunne-skrives--indlæsningsresultat-mislykket)). |
| `failed_captures` | Indlæsningsforsøg, der mislykkedes samlet set, herunder dem, hvor indlæsningen blev gentaget (en HTTP-503-fejl, mens arrayet ikke havde nogen ny gruppe i tide). |
| `skipped_groups` | Grupper, som arrayet offentliggjorde, men som ikke blev gemt (indsamlingen kom bagud). |
| `captures_per_s` | Gennemsnitlig gemningshastighed siden starten. |
| `elapsed_s` / `started_at` | Antal sekunder, der er gået indtil nu; starttidspunkt (Unix-epoke-sekunder). |
| `last_group_seq` | Den nyeste synkroniserede gruppe, der er gemt. |
| `last_error` | Den seneste fejl, eller `None`. |
| `stop_reason` | Når afsluttet: `count` (`max_captures` nået), `duration` (`duration_s` nået), `stopped` (`stop()` kaldt), `array_gone` (arrayet blev afbrudt), `error` (20 fejl i træk eller en anmodning, som backend afviste), `disk_full` (en fil, som drevet ikke havde plads til). `None` under kørsel. |
| `max_captures` / `duration_s` | De grænser, den blev startet med (`0` = ingen). |
| `defender_cores` | CPU-kerner, som Microsoft Defender har brugt i løbet af de sidste 10 sekunder; `None` indtil der måles en, og altid `None` på en vært uden Defender. |
| `defender_warned` | Sæt til True, når optagelsen har registreret, at Defender bremser den (ved 1 kerne eller mere). Undtag backend fra Defender-scanning — se [Trin 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Live-rammer i din egen pipeline

For at indføre synkroniserede billeder i din egen kode — maskinlæring, en brugerdefineret viewer, din egen optager — skal du bruge `ArraySession.frame_stream()`. Hvert tick genererer en `GrabResult`: `{serial: numpy array}` for hvert kamera, **alt sammen fra én trigger**, med den pågældende triggers id på `.group_id`. Det er den samme implementering som projekt-API'ets `ArrayHandle.frame_stream()`, så alt under [Én trigger pr. læsning](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#aligning-the-streams--align) og [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) gælder for begge.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Hvert udleveret sæt udgør en ny trigger.** `frames.group_id` gentages aldrig inden for en strøm; når du anmoder om flere `fps`, end arrayet kører med, venter strømmen på den næste trigger, så den kører med den langsomste af de to.
- **Manglende kameraer navngives, de udfyldes ikke.** Et kamera, der ikke kunne betjenes ved den pågældende trigger, er fraværende i ordbogen og opføres i `frames.errors`.
- **Det slutter med et brag.** `FrameFetchError` gentages ved en afvisning ved den næste tick, efter 60 sekunder uden frame eller efter 60 sekunder uden ny trigger.**Hvad det leverer.**`frame_stream()` overfører hele numpy-rammer via den lokale HTTP-forbindelse og når en maksimal hastighed på næsten**170 MB/s**. Det er nok til live-behandling af rå billeder eller af et binned- eller ROI-array, men ikke til hver eneste trigger i fuld opløsning — for at beholde**hver eneste** trigger skal man gemme på disken med [`continuous()`](#continuoushandle). Målt på et array med fire kameraer (FOCN, FRGN, FRGB, F450) med pakket transport; hvert tick var et komplet sæt med én trigger:

| | Fuld sensor (array 14,2–14,4 fps) | 2 × 2 binning (array ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 sæt/s | 22,2 sæt/s |
| `frame_stream(processing="debayered")` | 2,3 sæt/s | 9,3 sæt/s |
| `frame_stream(processing="display")` | 0,8 sæt/s | 3,0 sæt/s |
| MJPEG pr. kamera, 4 strømme ved 720p | 13,5 fps hver; hvert billede matchet med en trigger | ~25 fps hver; ~45 % af triggere til stede på alle 4 |

Brug `processing="raw"` til gennemstrømning og demosaikering på din side; `debayered`-niveauet fjerner også kalibreringspakkens mørkeforskydning, hvilket en demosaik af `raw` ikke gør.

#### MJPEG-strømme pr. kamera (ingen RTSP)

**RTSP understøttes ikke.** Streaminggrænsefladen er MJPEG via HTTP (`multipart/x-mixed-replace`, én JPEG pr. del, hver del med en `Content-Length`), én URL pr. kamera:

| URL | Hvad den serverer |
| --- | --- |
| `GET /api/camera/stream/<serial>` | 8-bit forhåndsvisning, op til ~30 fps. For et array-medlem indstiller `max_h` (standard 720, 64–2160) og `jpeg_q` (standard 80) angiver størrelse og kvalitet. |
| `GET /api/camera/<serial>/stream-annotated` | Den samme forhåndsvisning med GUI-overlejringer tegnet på (zebra, gitter, sigtekorn, histogram, peaking, spotmåler). `ChlorosLocal.camera_stream()` læser den for dig. |
| `GET /api/camera/<serial>/stream-reflectance` | 8-bit reflektans pr. bånd — se [Live reflektansstrøm](#live-reflectance-stream-mjpeg). |

- **Trigger-headere.** For et array-medlem indeholder hver del af alle tre strømme `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` og `X-Chloros-Timestamp-Ns`. Dele fra forskellige kameraer med samme `X-Chloros-Group-Id` udgør den samme trigger: indsaml delene og gruppér dem efter denne header for at samle et sæt igen. Et enkeltstående kamera har ingen gruppe: dets forhåndsvisningsdele indeholder ingen af disse headers (dets reflektansdele har en tom `X-Chloros-Group-Id`).
- **Forhåndsvisningsstrømmen dropper billeder under belastning.** Den sender det nyeste billede og springer dem over, som den ikke kunne sende i tide, så ved høje arrayhastigheder når ikke alle triggere frem til hvert kameras strøm (se tabellen ovenfor). Brug `frame_stream()` eller `continuous()` for hver trigger.
- **`/api/camera/stream/<serial>` sættes på pause, mens arrayets samlede livestream er åben** (`/api/camera/array/stream/<array_id>` — GUI'ens array-visning åbner den). Hvert medlems stream sender derefter én "Paused (array stream active)"-pladsholderramme uden `X-Chloros-*`-headere, og intet andet, indtil den samlede stream lukkes; derefter genoptages den af sig selv. `stream-annotated` og `stream-reflectance` sætter ikke på pause. Luk GUI'ens array-visning, når et script læser forhåndsvisningsstreams for hvert kamera.
- **Brug `frame_stream()` til tal, ikke billeder.**JPEG er tabsfyldt og 8-bit; `frame_stream()` returnerer de samme resultater uden nogen codec i stien.

### Intervaloptagelse til droneopmålinger

En opmålingsflyvning kræver én synkroniseret billedgruppe for hver få meter, der tilbagelægges, ikke hvert eneste billede, som arrayet kan producere. To indstillinger styrer dette:**`target_fps`**angiver, hvor ofte kameraerne tager et billede, og**`capture_repeated(interval_s=…)`** angiver, hvor ofte et billede gemmes.**Kameraerne fortsætter med at streame mellem gemningerne.** Hvert interval gemmer den nyeste synkroniserede gruppe; billederne imellem gemmes ikke. Den automatiske eksponering justeres ved hvert billede, så eksponeringen allerede er stabiliseret, når gemningen finder sted. Der findes ingen tilstand, hvor kameraerne stopper mellem optagelserne og vækkes for hver enkelt: Hver optagelse ville i så fald skulle vente på, at streamingen starter, og at eksponeringen stabiliseres.**Fastlås billedfrekvensen til det, intervallet kræver.**Hvis den er sat til auto, kører et array med den højeste hastighed, som værten og netværket kan håndtere. At fastlåse den til en lavere hastighed reducerer varmeudviklingen fra kameraet, netværksbelastningen og værts-CPU’en uden at ændre på, hvad der gemmes. Vælg en hastighed, der giver**2–4 billeder pr. interval**:

- Hver gemning venter på det næste billede, efter at den er anmodet om, så en gemt gruppe ender med at blive gemt op til en billedperiode efter det planlagte tidspunkt. En højere hastighed holder dette mellemrum lille.
- Autoeksponeringen justeres én gang pr. billede, så en meget lav hastighed gør, at eksponeringen følger en forbigående sky langsommere.

**Beregn intervallet ud fra flyveplanen:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

For eksempel giver et 60 m-dækningsområde med 80 % fremadrettet overlapning en afstand på 12 m; ved 10 m/s er det `interval_s=1.2`, og en fastsat `target_fps=3` giver mellem 3 og 4 billeder pr. interval. Optagelsen er tidsbaseret: SDK'et har ingen afstand- eller GPS-udløst tilstand, så hold en jævn hastighed i forhold til jorden på opmålingslinjerne.

**Gem rådata på flyet og kalibrer senere.** `interval_s` måles fra starten af hver gemning, så en gemning, der tager længere tid end intervallet, får den næste til at starte for sent, og afstanden afviger. Ved at gemme rådata sammen med lyssensorens måling (`processing="raw"`; målingen gemmes som standard) holdes hver gemning kort, og hvert billede kan bagefter konverteres til reflektans.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**fastlåser hastigheden fra starten;**`arr.set_target_fps(fps)`**ændrer den midt under flyvningen uden at genoprette forbindelsen, og**`arr.set_target_fps(None)`** sætter den tilbage til auto. En hastighed, der overstiger det, som værten og netværket kan håndtere, sænkes til dette loft — se `clamped` i resultatet — og den laveste hastighed er 0,1 fps.
- **Brug ikke `burst()` til at udtynde billeder.** Den skriver hvert eneste billede, som arrayet producerer; den følger dog en fastsat `target_fps`, så den er kun det rigtige værktøj, når alle billeder ønskes.
- **`target_fps` retter ikke beskadigede rammer.** At sænke værdien reducerer antallet af producerede rammer, ikke hvor hurtigt hver enkelt sendes; for GVSP-beskadigede rammer skal du i stedet sænke `wire_ceiling_mbps` (se [Array Health](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Tilslutning til et allerede tilsluttet array — `attach_array`

Hvis arrayet allerede er oppe (GUI’en har åbnet det, eller en tidligere SDK-session har kaldt `connect_array`), skal du bruge `attach_array` til at hente et håndtag til det i stedet for at oprette forbindelse igen. `connect_array` giver altid fejlen `Camera <sn> is already in array <id>`, fordi en POST-anmodning med `/array/connect` for et medlem i puljen ikke er idempotent; `attach_array` læser `/api/camera/array/list` og matcher enten efter array_id eller serienumre.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Mønster: SDK-scripts, der kører sammen med desktop-GUI’en, bør først prøve `attach_array` og falde tilbage til `connect_array`, hvis der endnu ikke er noget array i puljen.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Et tilknyttet array er lånt, ikke ejet.** Et håndtag fra `attach_array` nedlægger aldrig arrayet: hverken afslutningen af `with`-blokken eller et eksplicit `disconnect()` frigiver det (det eksplicitte kald udskriver en linje til stderr, der angiver, at arrayet fortsat kører). Samme regel som for `CameraSession` / `DAQSensorSession`. Det er altså sikkert at dele ressourcer med GUI'en, uanset om `with` bruges eller ej:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Når du rent faktisk vil afslutte det — for eksempel for at genoprette forbindelse med et andet `wire_ceiling_mbps` — skal du kalde `arr.force_disconnect()`. Genoprettelse af forbindelse efter et almindeligt `disconnect()` på et tilknyttet håndtag mislykkes med `Camera <sn> is already in array <id>`, fordi intet blev frigivet.

### Hjælp til netværksanalyse

Nyttigt inden du åbner arrayet — forudsiger, om dine foreslåede indstillinger vil passe:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Der startes aldrig et backend: et, der ikke svarer, udløser `ChlorosConnectError`, mens en afvist anmodning returneres som resultat. En ugyldig `packed` udløser `ValueError`.

`status` er en af følgende: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (ellers `error` — herunder en afvist anmodning: ikke logget ind, `error_code: AUTH_REQUIRED`, eller en plan under SDK-grænsen, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` betyder, at den anmodede opløsning kun passer til RX-ringen ved en begrænset triggerhastighed — behold opløsningen og gå videre fra `target_fps=result["recommended"]["recommended_target_fps"]` til `connect_array` (se [Eksempel 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` vises kun, når den anmodede ramme ikke kan leveres ved nogen billedhastighed; connect reducerer aldrig rammerne af sig selv, så anvend `recommended` selv.

**Sådan aflæses projektionen** (samme model som panelet »Array-indstillinger« i GUI’en):

- **Burst (`frame_bytes_total`) summeres pr. kamera i hvert kameras reelle pixelformat.**Mono**M3M**-kameraer streamer Mono12 (2 B/px) eller, med en `pixel_format`, dybdekameraets Mono-format (`"BayerRG8"` er `Mono8`, 1 B/px), så uden dette er et 4-kamera-billede i fuld opløsning**~25 MB** med tre mono-kameraer, ikke de ~12,6 MB, som en antagelse om, at alt er 8-bit, giver. Backend-systemet bestemmer hvert kameras format ud fra dets model.
- **Admittance (`burst_fits_nic_ring`) tager højde for dræning**, ikke baseret på hel-burst-vs-ring: sim-emit passer, når værten tømmer RX-ringen hurtigere, end kameraerne fylder den. En 10G-vært + 1 GbE-kameraer**tillader** fuld opløsning, selv når burstet overstiger ringen; en 1 GbE-host blokerer (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` er et konservativt loft for seriel hentning** — `max(readout+emit, N×emit)` med udsendelse pr. kamera begrænset til 1 GbE-kameralinket, uafhængigt af eksponering. F.eks. ~2,8 fps for et 4-kamera-array i fuld opløsning med 12 bit (svarer til runtime-målingen på ~2,7–3,0). Fuld model: [CLI-reference → Array fps & burst-model](cli-reference.md#array-fps--burst-model).
- **Efter en kontinuerlig optagelse angiver fremskrivningen, hvad den holdt.** En optagelse på 30 s eller mere ved en automatisk hastighed registrerer den hastighed, den stabiliserede sig på ved den pågældende array-størrelse på denne computer; hastighedsregulatorens korte forsøg med en hurtigere hastighed udelades. Fra da af angiver `array-connect`'linje, `connect_array()`'s analyse og panelet Arrayindstillinger angiver denne hastighed, når den ligger under modellens, sammen med årsagen til begrænsningen i `cycle_limiter`: `"clock"` (arrayet kørte med den hastighed, det havde fundet frem til, mens grab-loopen ventede på sin trigger-clock), `"loop"` (grab-loopen havde ikke mere tid tilbage), `"capture"` (optagelsen kom bagud og sprang grupper over) eller `"measured"` (der står ikke, hvilken af delene der er tale om). Modellens tal findes i `wire_fps_max`. Arrayet stiger stadig til det, som ledningen kan bære.
- **Over-subscription (`oversubscribed: true`) betyder, at ledningen ikke kan bære arrayet ved nogen af de hastigheder, kameraerne når** — fps-felterne (`achievable_fps_max` / `fps_bright` / `fps_dark`) viser 0, og binning kan ikke løse problemet (det sænker antallet af bytes pr. billede, ikke antallet af bytes pr. sekund). Et langsomt kabel alene forårsager ikke dette: billedfrekvensen falder i stedet, og hvis kameraerne ikke kan køre langsomt nok, bliver niveauet `slip-emit-and-capture` (seriel). Løsningerne er færre kameraer, jumbo-rammer eller et hurtigere netværkskort; `max_cams_collision_safe` angiver den maksimale kapacitet for din forbindelse og dine kameraer. Svaret indeholder også `aggregate_demand_bps`, `collision_safe_ceiling_bps` og `per_cam_floor_bps`. Se [Over-Subscription](#over-subscription-the-per-cam-floor).

### Opdagelse og liste

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Hver af dem tager kun nøgleord `backend_url=` og `timeout=30.0` og returnerer en liste over ordbøger. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) tager også `auto_start_backend=True`; `list_*`-kald starter aldrig et backend.

---

## Smart-AE / Smart-Capture

LATTICE-arrays kører kontinuerlig AE i baggrunden, så snart de er tilsluttet, men det tager et øjeblik for en nyindstillet scene at konvergere. **Smart-capture** er den praktiske løsning: den aflæser eksponeringen for hvert kamera, venter, indtil arrayet er stabilt på tværs af et vindue, og udløser derefter optagelsen. Det svarer til GUI’en: desktop-appens »smart«-optagelsesknap kalder det samme backend-endpoint.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Når du styrer via `ChlorosProject` (næste afsnit), får du flere indstillingsmuligheder:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Smart-AE-politikken er som standard konservativ. Stram `exposure_tolerance_pct` til krævende radiometrisk arbejde; lemp den til hurtigt skiftende scener, hvor du blot ønsker "tæt nok".

---

## DAQ-sensorsessioner

Vedvarende backend-pool til spektrale sensorer (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet). Afspejler kameraets overflade: smart-detect, genbrug af pool, idempotent tilkobling.

### Smart-Detect (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prioritet: Ethernet → BLE → USB. Angiv et hvilket som helst eksplicit hint for at fastlåse transporten.

### Fastlåst transport

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` angiver én sensor, og kun denne sensor åbnes nogensinde. Den sammenlignes med de sensorer, der allerede er åbne, hvorefter en DAQ-E’s værtsnavn eller IP (en DAQ-E, som netværksscanningen kun ser via IP, forespørges – skrivebeskyttet – om sit navn, MAC og serienummer, så enhver af disse finder den også), en DAQ-M’s BLE-MAC samt det id, som en DAQ-U eller DAQ-M rapporterer, når den er åbnet (enhver anden, der åbnes for at kontrollere, lukkes igen). Hvis den ikke findes, udløser `connect_daq_sensor` en `ChlorosConnectError`, der angiver, hvad der blev fundet; inklusive en adresse (`port=` / `mac=` / `eth_host=`), der indeholder en anden sensor, udløser den også en fejl. Den falder aldrig tilbage til den sensor, der er synlig — det er det, zero-flag-kaldet er til.

### `connect_daq_sensor()`-signatur

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

En sensor, der allerede er i puljen, returneres som den er (lånt): `integration_time`, `frame_avg` og `enable_ae` anvendes ikke på den. Frakobl den først for at åbne den med dine indstillinger. `cap_id` er undtagelsen: på en lånt sensor indstilles den med `set_cap()`. Udelades dette, gælder backendens standardindstilling (se bemærkningen under metodetabellen).

### `DAQSensorSession`-metoder

| Metode | Beskrivelse |
| --- | --- |
| `status(timeout=10.0)` | Oversigt over pool-indtastninger (streaming-/optagelsestilstand, bølgelængdeområde, kalibrerings-sha, integrationstid, frame_avg, AE-tilstand). For en DAQ-E, hvis stream er gået i stå: `reconnect` (hvor backendens automatiske genopkobling er aktiv: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None`, mens den streamer), `stream_lost_s`, `reconnects` og `last_reconnect` samt `record_gaps` for den igangværende optagelse; `rx_stream_fault` angiver genforbindelsestilstanden, indtil det første spektrum ankommer. |
| `latest(*, n=1, timeout=10.0)` | Returnerer op til N seneste spektrumrammer (hver `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Genoptag / sæt streaming på pause (håndtaget forbliver åbent). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Start optagelse af en .daq-fil. Returnerer filstien. Afvises for DAQ-U/M uden et AWS-kalibreringspakke (DAQ-E er undtaget). |
| `record_stop()` | Stop optagelse. Returnerer `{path, rows, warning, gaps}`; `warning` er `None` for en fejlfri optagelse, ellers backendets diagnose (f.eks. en tom fil eller et hul i strømmen), som også udskrives til stderr. `gaps` viser hvert afsnit, hvor strømmen ikke leverede noget: `start_utc`, `end_utc` (`None`: stadig tabt ved stop), `duration_s`, `reconnected`, `marked_in_file`. Den er tom ved kontinuerlig registrering. |
| `set_cap(cap_id, timeout=15.0)` | Indstil den cap-korrektionsprofil, der skal anvendes på målingerne fra den næste og frem: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"` eller `"none"` (eller `None`) ved en sensor uden dæksel. Returnerer sensorensoversigt, som `cap_id` læser tilbage. En DAQ-E-S holder `as_recorded` uanset hvad der forespørges (dens diffusor er indbygget). Hæver `ChlorosConnectError` for en profil, der ikke kan indlæses. CLI-kommando: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Kun DAQ-E: Slå PTP til igen for en enhed, hvor PTP er slået fra (`status()["rx_ptp_enabled"]` er `False`; indstillingen gemmes på enheden, så den forbliver slukket, indtil den tændes igen). Returnerer den opdaterede status. Udløser `ChlorosConnectError` for en DAQ-U / DAQ-M, eller når enheden ikke bekræfter. Chloros slår aldrig en enhed til igen af sig selv, da den kan være slukket bevidst. CLI-kommando: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Kun DAQ-E: enhedens egne indstillinger. `firmware` (`running`, `bundled`, `update_available`), `ip` og `ip_mode` (`dhcp` / `linklocal` / `static`: hvad den kører nu), `network` (den gemte indstilling: `mode`, plus `ip` / `mask` / `gateway`, når den er i statisk tilstand), `network_change_pending` (gælder, indtil enheden kører den gemte indstilling, en indstilling, der er gemt med `set_network(..., reboot=False)` inkluderet), `auto_stream`, `ptp`. `firmware["update_available"]` er `None`, når enhedensversion ikke kunne læses. Mens enhedens datastrøm ikke når frem til denne computer, læses den ved en ny forbindelse, og ordbogen tilføjer `stream_silent` (`True`), `stream_fault` og `reconnect`; en enhed, der ikke svarer, udløser en fejl. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E kun: DHCP (`"auto"`) eller en statisk adresse. Genstarter enheden for at anvende den og opretter forbindelse igen; denne session fortsætter med at fungere, og `address` følger enheden. `reboot=False` gemmer den kun. Se [DAQ-E-enhedsindstillinger](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Kun DAQ-E: streamer ved opstart uden tilsluttet software (gemt på enheden). Returnerer den bekræftede indstilling. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Kun DAQ-E: Genstart enheden og tilslut den igen med de indstillinger, den havde. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Kun DAQ-E: Flash den firmware, der følger med denne Chloros-udgivelse, til en enhed, der kører en ældre version, og tilslut den derefter igen (ca. et minut; sluk ikke for strømmen). Returnerer `updated`, `fw_before`, `fw_after`, `bundled` og en `warning`, hvis enheden var lang tid om at vende tilbage eller vendte tilbage med en anden version; `updated` er `False` for en enhed, der allerede kører den nyeste version. Udløser `ChlorosConnectError` for en enhed, hvis version ikke kan læses. CLI: `pool-update-firmware`. |
| `disconnect()` | Frigivelse fra puljen. Ingen-op for tilknyttede håndtag, der ikke ejes. På en sensor, som denne session har åbnet, er det `force_disconnect()`, og der genereres en `ChlorosConnectError`, når frigivelsen ikke bekræftes (udgangen fra `with`-blokken udløser aldrig en fejl). |
| `force_disconnect(timeout=30.0)` | Frigiv sensoren uanset hvem der har åbnet den — for eksempel en sensor, som et nedbrudt script har efterladt i puljen, og som et nyt script kun når frem til som et tilknyttet håndtag, hvis `disconnect()` ikke udfører nogen handling. Afslutning af en anden klients sensor afslutter enhver optagelse, den er i gang med. Returnerer, når backend bekræfter frigivelsen, eller når der ikke er noget at frigive (sensoren er ikke i puljen, eller der kører ingen backend); en frigivelse, der er løbet ud, eller som backenden har afvist, udløser `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Attributter: `sensor_id`, `model`, `transport`, `address` (følger efter en DAQ-E, som `set_network()` har omadresseret), `info` og `backend_url`.

> **Navngiv hætten.** DAQ-U, DAQ-M og DAQ-E leveres uden kondensator, og Chloros kan ikke se, om der er monteret en: en sensor, der tilsluttes uden `cap_id`, får `sunshine_cosine`, en antagelse, som dens registreringer angiver, at de har foretaget (`cap_id_source=auto_default`). En ubeskyttet DAQ-U viser under denne korrektion ca. 30 gange for høje værdier, så indtast `cap_id="none"` for en ubeskyttet sensor (eller `set_cap("none")`), og profilen for den kappe, du ellers har monteret.

### DAQ-E-S — fast diffusor og hældningssensor

**DAQ-E-S** er en DAQ-E med to forskelle, der har betydning for softwaren. Alt andet — opdagelse, tilslutning, PTP, registrering, enhedsindstillingerne nedenfor — fungerer nøjagtigt som for en DAQ-E.**Dens diffusor er fastmonteret, så der kræves ingen hættekorrektion.** Diffusoren i Sunshine-stil er indbygget og sad på enheden, da fabrikskalibreringen blev målt, så kalibreringen allerede angiver den reelle nedadgående strålingsintensitet. Chloros indstiller sin cap til `as_recorded` (ingen profil), uanset hvad der anmodes om: en cap opkaldt efter den (`sunshine_cosine`, `fov_*`, `none`) ignoreres med en meddelelse, og dens registreringer gemmer `cap_id_source=model`, så advarslen om antaget cap udløses aldrig for den. Du må heller ikke manuelt anvende en begrænsning på dens data: `sunshine_cosine` øverst vil vise en værdi, der er ca. 11 gange for høj.

**Den er udstyret med et 3-akset accelerometer.** Sollysdiffusorens cosinusrespons er kun gyldig, når den er vandret, så en skrå sensor rapporterer en forkert strålingsintensitet — og reflektansen arver fejlen — uden andre symptomer. Hver DAQ-E-S-måling indeholder derfor sin orientering:

| Felt | Betydning |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Vinkler i grader. `None`, når firmwaren markerer vinklen som udefineret (frit fald, en død akse). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Acceleration i milli-g. `mag_mg` langt fra ~1000 betyder, at enheden var i bevægelse, og at vinklerne ikke er meningsfulde for den pågældende prøve. |
| `cal_applied` | Indstillet: akserne og vinklerne er i diffusorens koordinatsystem (enhedens monteringskalibrering er anvendt). Ryddet: rå kortkoordinatsystem, som kan vise op til ~2,3° hældning, der ikke er reel. |
| `sample_age_ms` | Alder på accelerometerprøven, da spektret blev sendt. |
| `temp_raw` | Ukalibreret **tendens** for chip-temperatur — kun forskelle, aldrig en termometeraflæsning. |

`DAQSensorSession.latest()` returnerer dem under hver rammes `imu`-nøgle (`None` på en DAQ-U, DAQ-M eller almindelig DAQ-E). En `.daq`, der er registreret fra en DAQ-E-S, gemmer de samme værdier i sine `imu_*`-kolonner (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) ved siden af hvert spektrum, så læseren kan skelne mellem en sky og et væltet stativ. På alle andre modeller er disse kolonner tomme.

Holdningsstrømmen rapporterer om sine egne felter i `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), adskilt fra spektrumfelterne, så et udfald af accelerometeret aldrig tolkes som et hul i optagelsen. `discover_daq_sensors()` og sessionens `model` angiver `DAQ-E-S`.

### DAQ-E-enhedsindstillinger

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **Der skal være en statisk adresse på denne computers netværk.** `set_network` udløser `ChlorosConnectError` for en enhed, hvis undernet ikke omfatter denne computersadresse på sensorens netværk: efter genstarten kunne intet nå frem til sensoren for at ændre den tilbage. Godkend kun `force=True`, når denne computer får tildelt en ny adresse eller omdirigeres til det nye undernet.
- **`set_network`, `reboot` og `update_firmware` genstarter sensoren** og udløser `ChlorosConnectError`, mens den registrerer, og når den ikke vender tilbage inden for `comeback_timeout` sekunder. Sensor-ID'et og værtsnavnet `daq-e-<id>.local` ændres aldrig.
- **En optagelse, der blev startet lige før genstarten, afsluttes af denne og navngives.** Når sensoren vender tilbage, indeholder det returnerede dict `recording_closed` (filens sti) og et `warning`, der angiver dette. Når den ikke vender tilbage, eller hvis firmwareoverførslen mislykkes, angiver `ChlorosConnectError` dette i stedet: dens besked slutter med advarslen, og `e.recording_closed` / `e.warning` er sat (begge `None`, når ingen optagelse blev lukket). Start optagelsen igen, når sensoren er tilbage.
- **Kun selve sensoren tilsluttes igen.** Efter genstarten skal enheden svare, hvor den var -- og efter `set_network("auto")` også under sit `.local`-navn -- med sit navn, værtsnavn og model, fra den netværksadapter (MAC), den havde; `address` bliver den IP-adresse, der svarede. Et svar under navnet tæller kun med den pågældende MAC-adresse, så fra en computer, der når sensoren via en router, eller fra WSL2's standard NAT-netværk, bliver en sensor, som DHCP har flyttet, ikke genforbundet (`ChlorosConnectError`; tilslut den igen på den nye adresse).
- **En firmwareopdatering kræver, at sensoren opretter forbindelse igen.** `update_firmware()` inviterer enheden, og enheden åbner en TCP-forbindelse tilbage til denne computer for at hente billedfilen på en port, som operativsystemet vælger. Backend'et lytter på det netværkskort, der har forbindelse til enheden (på alle netværkskort, hvis det ikke kan afgøre, hvilket der er tale om), og betjener kun den pågældende enhed: enhver anden forbindelse afvises (`[DAQ-FW] … refused a firmware connection from <ip>` i backendens output), mens den fortsætter med at vente. En firewall, der blokerer forbindelsen, forhindrer opdateringen med fejlkoden `device never connected back on TCP <port>`. Windows-installationsprogrammet tilføjer reglen herfor (se [Trin 1](#windows-exe)); på en anden firewall skal indgående TCP-trafik til backend'en fra sensorens undernet tillades.
- **Under WSL2 skal du bruge spejlet netværk** (`networkingMode=mirrored` i `.wslconfig`): i standard-NAT-tilstand afvises alle statiske adresser (WSL's egen adresse er ikke på sensorens netværk), strømmen når ikke frem til WSL, og en firmwareopdatering kan ikke gennemføres.
- GUI’en opretholder sin egen forbindelse til en sensor: en genstart fra et script afslutter GUI’ens session med sensoren. GUI’en har de samme indstillinger (fanen Lyssensorer, sensorens indstillinger, Netværk).

### Opdagelse — at finde en adresse at oprette forbindelse til

`discover_daq_sensors()` scanner USB / BLE / ETH for sensorer, som du *kunne* åbne. Det er DAQ-modstykket til `discover_lattice_cameras()` og den eneste måde at få fat i en **DAQ-M's BLE-MAC** — en DAQ-E har et værtsnavn og en DAQ-U en COM-port, men en MAC-adresse er hverken trykt på enheden eller angivet af operativsystemet.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Felt | Beskrivelse |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-port / BLE MAC / værtsnavn — videregives til `connect_daq_sensor` som `port=` / `mac=` / `eth_host=`. |
| `display` | Menneskeligt læsbar etiket. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, eller `None`. En Ethernet-sensor er `DAQ-E-S` eller `DAQ-E`, da den identificerer sig selv, når den bliver spurgt, skrivebeskyttet, under scanningen (`DAQ-E`, når den ikke svarer i tide). En USB-serielport viser `DAQ-U`, når dens bridge-chip er en af dem, som DAQ-U bruger (FTDI, CH340, CP210x, Prolific) — et tip fra portens beskrivelse, ikke en probe — og `None` for enhver anden USB-enhed, som stadig er opført, så du kan videregive den som `port=`. En seriel port uden USB-leverandør-id og uden en sådan brochip (en UART på bundkortet, såsom Linuxs `/dev/ttyS*`, en seriel Bluetooth-forbindelse, Intel AMT) kan ikke være en DAQ-U og er ikke opført. |
| `extra` | Detaljer pr. transport (BLE-annonceret navn; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). Tomme værdier udelades. |

| Parameter | Standard | Beskrivelse |
| --- | --- | --- |
| `transports` | alle tre | Sekvens (eller CSV-streng), der begrænser scanningen. Det er en god idé at angive denne, når du ved, hvad du vil have — BLE er den langsomme del. |
| `scan_timeout` | 5 | Scanningsvindue pr. transport i sekunder; backend begrænser til 1–20. |
| `timeout` | 60,0 | HTTP-loft for hele opkaldet (som andre steder i SDK'en). |
| `auto_start_backend` | `True` | Starter en lokal backend, hvis der ikke kører nogen. Startes aldrig for en fjern `backend_url`. |

> **Sensorer, der allerede er åbne i puljen, vises ikke.** En tilsluttet BLE-periferienhed holder op med at annoncere, og en åben COM-port kan ikke afprøves, så opdagelseslisten viser, hvad der er *tilgængeligt for tilslutning*. Der forventes et tomt resultat lige efter, at du har tilsluttet noget — brug `list_daq_sensors()` til det, du allerede har. Transportprotokoller, hvor scanningen ikke kan køre (ingen bleak/zeroconf installeret), springes over i stedet for at udløse en fejl, så en maskine uden Bluetooth stadig får sine USB- og ETH-svar.

### Liste

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Hvilken DAQ forsyner hvilket kamera — `daq_bindings()`

En optagelses reflektans og den `.daq`, den gemmer, bruger den DAQ, der er knyttet til det pågældende kamera eller den pågældende matrix, ellers den nyeste DAQ, som backend'en har. Valget af lyssensor i GUI'en og `chloros-cli daq pool-bind` indstiller den; det samme gør:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` returnerer `{"cameras": {serial: b}, "arrays": {array_id: b}}`: hver `b` indeholder `binding` (den indstillede værdi), `source` (`project`, fra projektets `sensors.json` eller `runtime`), `known` / `known_as` (om kamera- eller array-nøglenavnet er et, som en Chloros kender, og hvordan: `connected`, `project` — det åbne projekts `cameras.json` — eller, for et kamera, `seen`: tidligere åbnet på denne computer; `known: False` for en nøgle, der ikke navngiver noget) og `resolved` (den pool-sensor, den nu er knyttet til: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). En runtime-binding har forrang for projektetsfor den pågældende session; projektfilen ændres ikke. Afbrydelse af forbindelsen til et kamera, et array eller en DAQ ophæver alle bindinger til den pågældende hardware, uanset hvem der har oprettet dem (medlemmerne af et array bindes sammen med arrayet).

`set_daq_bindings()` binder kun et kamera, der er tilsluttet, findes i det åbne projekt eller tidligere er blevet åbnet på denne computer, samt et array, der er tilsluttet eller findes i det åbne projekt (eller en nøgle, der allerede er bundet). Enhver anden nøgle udløser `ChlorosConnectError` med navnet (`400`, `code`, `UNKNOWN_BINDING_TARGET`, sammen med nøglerne `unknown`), og **intet** i dette kald bindes. `None` rydder alle nøgler.

### Omgivende lys + objektreflektans — `light_sensor_reflectance()`

To sammenkoblede DAQ'er, hvor den ene kigger op mod himlen (omgivelser) og den anden ned mod målet (objekt), giver målets reflektans og vegetationsindekser ud fra deres seneste målinger: GUI'ens par »Omgivelser + Objekt« og `chloros-cli daq reflectance` uden at åbne sensorerne en anden gang. Begge skal være tilsluttet og streame data.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Bånd.** Hvert bånds reflektans er objektets bånd-integrerede irradians over omgivelsernes. `daq_band_presets(filter_code=None)` viser båndsættene: ét pr. LATTICE M3C-filter (`RGN`, `OCN`, `NGB`, kameraets egen båndmodel, så en DAQ-NDVI og en kamera-NDVI er den samme størrelse) samt `GENERIC` bredbånd. Dets `suggested` er den forudindstilling, der matcher `filter_code`, ellers det første tilsluttede kameras filter (`suggested_from` angiver hvilket).
- **Returnerer** `bands` (pr. bånd `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (de indbyggedeindbyggede), `custom` (hver med `value` eller `valid: False` og `error`), `invalid_bands`, `reflectance_type`, `preset` og de to måleværdier `timestamps` (`ts_ns`). Et bånd, som omgivelsessensoren ikke kunne måle (uden for dens måleområde eller ved dens støjgrænse), er ugyldigt med angivelse af årsagen, og alle indekser, der kræver dette, udelades og beregnes aldrig i forhold til nul.
- **Tilsyneladende eller relativ.** De skalafhængige indekser (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) kræver et absolut forhold, så de vises kun, når begge sensorer er strømkalibrerede (hver har en `calibration_bundle_sha`): `"Apparent Reflectance"`. Ellers er resultatet `"Relative Reflectance"`, hvor kun de skalainvariante indekser indgår.
- **To forskellige bølgelængdegitter** sammenlignes med hensyn til deres overlapning: omgivelsessensorens punkter inden for objektets rækkevidde, objektets spektrum interpoleret til disse.
- **To sensorer, aldrig én.** At navngive én sensor som begge (samme id, uanset hvilken stavemåde puljen accepterer) giver `ChlorosConnectError`: en sensors aflæsning af sig selv er 1,0 på hvert bånd, hvilket ikke måler noget. Backend'et afviser det også (`POST /api/daq/indices` svarer på `400`, `code` `SAME_SENSOR`, når `ambient_sensor_id` og `object_sensor_id`, som SDK'en sender, er én samlet sensor, f.eks. dens id og dens port).

### Fælles brug med CLI'en og andre scripts

Backendets DAQ-sensorpool deles af alle CLI- og SDK-klienter på maskinen. Hvis en sensor allerede er i poolen — åbnet af `chloros-cli daq pool-connect` eller af et andet script — returnerer et kald til `connect_daq_sensor(port="COM3")` fra Python et lånt håndtag: dens `disconnect()` er en no-op, så dit script ikke river sensoren væk fra den klient, der åbnede den, når scope afsluttes. Det samme gælder for `sensor_id=` (indtastet med `:` eller `-` – begge dele fungerer), for en DAQ-E, der blev navngivet ved sin IP, da den blev åbnet via værtsnavn (eller omvendt), og for smart-detect med nul-flag, når backend svarer med en sensor, den allerede havde — en `with`-blok frigiver kun en sensor, den selv har åbnet.

Chloros-GUI’en er **ikke** en del af denne pulje: dens DAQ-panel kører sin egen forbindelse til sensoren, så `connect_daq_sensor` låner aldrig en sensor, som GUI’en har åben. Brug ikke den samme sensor fra GUI’en og fra et script på samme tid. En DAQ-U eller DAQ-M, som GUI’en har åben, kan normalt ikke åbnes, men en DAQ-E accepterer en anden session: scriptets forbindelse ændrer enhedens indsamlingsindstillinger under GUI'en, og dens `disconnect()` stopper enhedens stream, hvilket afslutter GUI'ens livevisning og enhver optagelse, den foretager. Luk først sensoren i GUI'en.

### PTP Grandmaster — `time_sync_status()`

Backend’et kører den PTP-grandmaster, som LATTICE-kameraer og DAQ-E-enheder henter deres tidsstempler fra. Den starter efter behov: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` og `connect_all()` starter den, hvis den ikke kører — herunder en forbindelse, der derefter mislykkes, såsom når `connect_all()` forsøger at oprette forbindelse til en DAQ-E, der ikke er til stede — og det samme gælder `test_array_sync()` og hvert opkald, der åbner kameraer for en kommando (`capture_cameras_transient()`, `start_multi_camera_stream()`, justeringskald, `capture_live_index()`).

- **Hvor det meddeles.** På de kablede netværkskort, der vender mod kameraerne (herunder en `169.254.x.x` link-local-adresse), ellers på kortet for standardruten, hvis dette er kablet. Aldrig på Wi-Fi eller et virtuelt kort (Tailscale, WSL): der er ingen kameraer eller DAQ-E'er bagved disse. Hvis der ikke er en sådan adapter aktiv, kører den med annoncering på ingenting (`health` `NO_SOCKETS`) og binder sig til den adapter, der aktiveres, når det næste kamera eller DAQ opretter forbindelse.
- **Efter et forbindelsestab.** Når kameraadapteren mister sin forbindelse og genopretter den (et kabel trækkes ud, en switch udskiftes, en USB-adapter genregistreres), genopbygger grandmasteren selv sine sockets uden genstart, og kontrollerer alle tilsluttede DAQ-E'er igen, hvorved den genaktiverer den, der blev hængende i mellemrummet. Den genopbygger også, når alle kameraer og DAQ-E'er, den betjente, har været inaktive i 15 sekunder.
- **Når den ud til kablet?** Læs `time_sync_status()` (`GET /api/time-sync/status`). `state` angiver kun, at grandmaster-tråden kører; `health` angiver, om dens pakker sendes ud: `OK` (ingen fejl konstateret), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (nogle adaptere svigter) eller `PEERS_SILENT` (de kameraer og DAQ-enheder, den betjente, svarede ikke længere), med en sætning i `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` tæller kun de meddelelser, der er sendt; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` og `last_delay_req_age_s` beskriver dem, der ikke gjorde det, og `rebinds` / `last_rebind` genopbygningerne. `sync_sends_slow` tæller de Syncs, hvis afsendelse varede længere end 200 us, og `sync_send_max_us` er den længste; hver Follow_Ups oprindelsestid tæller højst 200 us af dens afsendelse. Grandmasteren kører i sin egen proces (`own_process`, `process_pid`) med høj prioritet på Windows (`process_priority`), så et travlt backend ikke kan forsinke de tidsstempler, det sætter, og flytte alle kameras ure på én gang; hvis denne proces ikke kan startes, kører den inde i backenden som før (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Funktion | Hvad den gør |
| --- | --- |
| `time_sync_status()` | Grandmasterens tilstand, sundhed, uridentitet, prioriteter og meddelelsestællere (ovenfor). `chloros-cli time-sync status`. |
| `time_sync_peers()` | De PTP-slaver, den hører (kameraer, DAQ-E'er): `peers`, hver `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Hvert tilsluttet array-kameras `PtpStatus`, `PtpOffsetFromMaster` og `PtpMeanPathDelay` (et kamera, hvis firmware ikke har nogen PTP-noder, har kun sit serienummer). `time-sync cameras`. |
| `restart_time_sync()` | Stop og genstart grandmasteren; hver samlet DAQ-E's PTP kontrolleres igen efter (`daq_ptp_rechecked`). Udløser `ChlorosConnectError`, når den ikke kommer tilbage. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Dens PTP-prioriteter (0-255, lavere vinder valget af best-master) for et netværk med en anden PTP-master. Kræver, at den kører. `time-sync set-priority`. |

Hver tager `backend_url=` og `timeout=` og starter et lokalt backend, der ikke kører.

---

## Kameraværktøjer

Hvad `chloros-cli lattice` gør ud over at oprette forbindelse og indsamle data, som almindelige funktioner i `chloros_sdk` (modul `chloros_sdk.camera_utils`). Hver af dem kalder den backend-rute, som CLI-kommandoen bruger, med CLI's regler, så de to opfører sig ens. De tager et kameras **serienummer**, ikke en session, og de samme `backend_url=` / `timeout=`-nøgleord som smart-connect-kald; et backend, der ikke kører, startes, ligesom `connect_camera()` gør. Et afslag udløser `ChlorosConnectError` med backendets begrundelse.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Funktion | Hvad den gør |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | Skærmens farveprofil: `raw`, `linear`, `natural`, `enhanced` eller `custom_temp` samt dens farvetemperatur (begrænset til 2000–10000 K; aflæses, når profilen er `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Skærmens mætning og kontrast, hver 0–200 % (100 = uændret); `reset=True` sætter begge tilbage til 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Sænk de lyseste, hvideste pixels i skærmvisningen: -100..0 (afkortet mod nul, derefter begrænset; 0 = slukket). |
| `camera_white_balance_once(serial, settle_s=None)` | Engangs-hvidbalance: kameraet streamer kortvarigt, konvergerer og returnerer `red_ratio` / `blue_ratio` (`ratios_read` False, når den ikke kan rapportere dem). Den bevarer den hvidbalance, den har beregnet; alle andre indstillinger, som dens open skrev, returneres som fundet. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (kontinuerlig) eller `"off"`; eller faste `red` / `blue`-balanceforhold, som først slår automatisk tilstand fra. Et forhold, som kameraet afviser, er angivet under `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | Kameraets forudindstillinger, og når man anvender en: kameraet beholder de indstillinger, som forudindstillingen angiver (svaret er `settings`, da kameraet læser dem bagefter, f.eks. den billedhastighed, det kører med; `None` for et felt, som forudindstillingen ikke navngiver), og alle andre indstillinger, som dets åbne fil indeholdt, genindstilles til de oprindelige værdier. `"default"` nulstiller kameraet til fabriksindstillingerne (dets fabriksindstillinger med fritløbende automatisk eksponering, grænsen for linkgennemstrømning og fejlkorrektion slået fra, et RGB-kameras kalibrerede hvidbalance bevares), som ikke gemmes i dets hukommelse: svaret indeholder `factory_defaults: True` og alle felter i `settings`. I `camera_presets()` er `default` lig med `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Filtermodellerne og detaljerne for én model (bånd, spektralrespons-tilstandsnummer). Et kameras filtermodel læses fra dets modelstreng; den indstilles ikke manuelt. |
| `camera_firmware_status(serial=None)` | Hvert kamerasfirmware: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` og `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Flash denne Chloros-builds firmware til ét kamera: fabriksindstilling, flash, genstart, Chloros-basisindstillingerne overskrives; kameraets kalibrering bevares. Det tager flere minutter; kameraet skal forblive tændt. `force=True` flasher et eksisterende kamera igen. |
| `camera_power_probe(serial)` | Kameraets strøm- og termiske noder samt dets temperatur. |
| `sleep_camera(serial)` / `wake_camera(serial)` | Kameraets strømbesparende dvaletilstand (det forbliver i dvaletilstand, indtil det vækkes: dens grænse for linkgennemstrømning er aktiveret (`DeviceLinkThroughputLimitMode` `On`) på det laveste niveau, strømbesparelsestilstanden er aktiveret, hvor kameraet har en sådan), med temperaturen før og efter; opvågningen sætter forbindelsen tilbage til den tilstand, den var i, da den gik i dvale (grænsen, dens tilstand og pakkeforsinkelsen `GevSCPD`, som kameraet holder på 0, mens grænsen er aktiveret; optegnelsen gemmes på denne computer, så en opvågning fra et senere backend også kan finde den), eller, for et kamera, som denne computer ikke har sat i dvaletilstand, men som ser ud til at være i dvaletilstand (grænsen er aktiveret ved minimumsværdien), med grænsen deaktiveret og en `GevSCPD` sat til 0 ved 80 (fabriksindstillingerne); en grænse, der er indstillet til over minimumsværdien, forbliver uændret. Kun strømtilstanden gemmes; alle andre indstillinger, som deres åbne skrivning vedrører, genindstilles til de oprindelige værdier. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Optag billeder, indtil et MAPIR-kalibreringsmål er i billedet, og tilpas koefficienterne pr. kanal ud fra dets paneler: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Denne computers Ethernet-adaptere til GigE Vision (MTU, linkhastighed, jumbo-rammer) og eventuelle `issues`; `estimate=True` tilføjer den billedhastighed og båndbredde, som `cameras`-kameraer ville få. |
| `fix_network_settings()` | Anvend GigE Vision-adapterindstillingerne på denne computerskameraadapter. Det ændrer adapterens indstillinger og fungerer kun, når Chloros kører som administrator; ellers vises en log med de fundne oplysninger. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Udfør vignetteringskorrektion af billederne i en mappe, hvor hvert billede har sit eget objektivs flat-field-kort: `processed`, `lens_models` (det objektiv, hver fil er korrigeret for), `skipped` (med begrundelser). Uden `lens_model` stammer hvert fils objektiv fra dets EXIF-model; en `lens_model` gælder for filer, hvis model ikke angiver noget kamera, og en fil, hvis model er i modstrid hermed, springes over. Kun filer. |
| `read_dls(mac, filter=None, camera_serial=None)` | En aflæsning fra en DLS (Bluetooth-sensor til nedadrettet lys): spektrum, strålingsintensitet pr. bånd og båndstørrelser gennem kameraets egen respons, når `camera_serial` angives. `discover_daq_sensors(transports=["ble"])` finder MAC-adressen. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | Værtens hardware til kameraer og den anbefalede konfiguration (`report_text`) samt systemets CPU / GPU / PyTorch-runtime. |

- **Nogle har brug for kameraet for sig selv.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` og `update_camera_firmware()` åbner kameraet. Mens Chloros holder det (forbundet i appen, med `connect_camera()`, som medlem af en matrix eller i brug af en anden kommando lige nu, f.eks. en array-test eller en optagelse, der åbner sine kameraer), afviser backend’et med HTTP-kode 409 `CAMERA_HELD` (i både `code` og `error_code`), hvor der angives, hvordan det frigives. Farveindstillingerne, højlys, `camera_power_probe()` og aflæsningerne fungerer på et tilsluttet kamera.
- **Farve, højlys og hvidbalance gælder for RGB-kameraer.** På et enkeltbånds-mono (M3M)-kamera ændrer indstillingsfunktionerne intet og returnerer `{"skipped": True, "reason": ...}`, ligesom CLI. Det er skærmindstillinger: live-visning og optagelser på skærmniveau, aldrig strålingsintensitet eller reflektans.

---

## Transiente arrays, justeringsprofiler og indekser

Hvad `chloros-cli lattice` gør uden en vedvarende forbindelse, som funktioner i `chloros_sdk` (modul `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` og `index`. Hver af dem sender den anmodning, som CLI-kommandoen sender, med CLI's standardindstillinger.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **De åbner selv kameraerne.** Hver kommando åbner sine kameraer, udfører sin opgave og lukker dem igen, så den har brug for kameraer, som Chloros ikke har åbne (tilsluttet i appen, med `connect_camera()` eller som et array-element); backend'et afviser med HTTP-kode 409 `CAMERA_HELD` og angiver, hvordan man frigiver et. En midlertidig åbning tager et stykke tid (ca. 40 sekunder for et par kameraer). Ved gentagne optagelser er `connect_array()` én gang den hurtigste metode, og den gemmer alle eksporttyper, hvor en midlertidig optagelse gemmer et debayered billede. `calibrate_alignment_profile()` er undtagelsen: på et tilsluttet array af disse kameraer udleder den fra arrayets egne billeder (`used_live_array`).
- **`serials`** er de kameraer, der skal åbnes, hvor det første udløser triggeren; `None` åbner alle kameraer på netværket.
- **`settings`** er en ordbog med overstyringer af kameraindstillinger, de felter, som CLI-flagene indstiller: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` og `preset` (en kameraindstilling med dette navn). En angivet eksponering eller forstærkning deaktiverer denne automatiske styring, ligesom flagene gør.
- **`gpu`**: `True` tvinger GPU-warp, `False` tvinger CPU (`--gpu` / `--no-gpu`); `None` lader backend'en vælge.
- **`keep_settings`**/**`clear_settings`** (alle funktioner, der åbner kameraer, standard `False`): til sidst sættes hvert kamera tilbage til den tilstand, det blev fundet i; `keep_settings=True` gemmer i stedet kørselens indstillinger i kameraets egen hukommelse (frit kørende), `clear_settings=True` nulstiller denne hukommelse til fabriksindstillingerne — se [Kamerahukommelse](#camera-memory--keep_settings--clear_settings). Svaret (for en opgave, `stop()`'s) indeholder `camera_memory`.
- **Stier er backendens**: relative stier gøres absolutte i forhold til dit arbejdsbibliotek, når backend'en kører på denne maskine. Funktionerne til alignment-profilfiler (`alignment_profile_info()`, `reorder_alignment_profile()`) læser filen på denne computer, ligesom CLI'en gør.

| Funktion | Hvad den gør |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Optager fra ét kamera kontinuerligt, indtil det stoppes. Returnerer en `CliJob`. Rammerne fra en `tiff`-kørsel udgør den `raw`-fil, som en array-optagelse skriver fra det pågældende kamera (et mosaikplan med sensorensegne værdier, ukomprimeret); `png` og `jpg` indeholder 8 bits, så disse kørsler er demosaicerede billeder til visning. En bundet eller samlet DAQ skriver én `.daq` pr. billede i kørslen`daq/`-mappe (`daq_files` i dens statistik). Hvert billede skrives én gang med sine fulde metadata (model, optagelsestidspunkt, objektiv, bånd, kalibrering på et råbillede); `stop()` returneres, når hvert enkelt indsamlet billede er skrevet (op til 25 s), og først derefter lukkes kameraet. Et kamera, der er sat til hardware-udløser (`TriggerMode` `On`) indstilles til fritløb for optagelsen (startbesvarelsens `trigger` angiver, hvad der blev fundet). Ved stop sættes kameraet tilbage, som det blev fundet — dets udløserindstillinger og alle indstillinger, som kørslen ændrede — medmindre `keep_settings=True`, som gemmer kørslens indstillinger i kameraets egen hukommelse (frit kørende), eller `clear_settings=True`, som nulstiller denne hukommelse. En kørsel, der ikke optager nogen billeder, afslutter sig selv efter 10 sekunder med `stop_reason` `no_frames` og årsagen i `last_error`, og dens `stop()` returnerer denne årsag som `error`. Billedrammer, der er gået tabt, fordi skriverne ikke kunne følge med, tælles, og `drop_reason` angiver årsagen. For et kamera, som Chloros-holder, skal du bruge `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Stream synkroniserede billedgrupper ved `fps`, for `count` grupper eller indtil det stoppes. Returnerer en `CliJob` (dens `start_reply` har `n_cams` og ledningen `pacing`); dens endelige statistik tæller de hentede, skrevne og droppede grupper, de filer, som skriveren ikke kunne skrive, og de grupper, hvor et kamera ikke leverede noget. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Én synkroniseret optagelse, et debayered billede pr. kamera: `results` (`filepath`, `serial`, `frame_id`) og `pacing`. En gruppe, hvor der mangler et kamera, vises, med angivelse af, hvor mange filer der blev gemt først. `lattice multi-capture` uden tilsluttet array. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` synkroniserede optagelser med fast eksponering: `sync_groups`, `master`, én række `group_timestamps` pr. udløsning (et kamera, der gik glip af et billede, mangler i sin række) og `sync`: hvert andet kameras forskydning i forhold til masteren, udløser for udløser (`offsets_us`: pr. kamera `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), på ÉN klokke (`clock_domain`: `ptp` eller `device` — PTP slået fra, hvert kameras eget driftstidsur, så dets forskydning er fra opstart, og kun jitteret måler synkroniseringen). En trigger, hvis tidsstempler blander enhedsklokken og PTP (PTP slået fra eller stadig i låsning), udelades og tælles med (`groups_mixed_domain`), ligesom en trigger, hvor en PTP-klokke stadig er mere end `agree_within_us` fra masteren (`groups_clocks_disagree`); dette fremgår af `notes`. `clock_offset_mean_us` / `clock_offset_jitter_us` er det største gennemsnit og den største jitter pr. kamera. `success` er falsk, sammen med `error`, når en trigger blev overset, når færre end to triggere har alle kameraer på samme ur, eller når urene aldrig stemmer overens (`sync.problems`); rækkerne returneres uanset hvad. Den starter PTP-grandmasteren, hvis den ikke kører (`time_sync` angiver `grandmaster_running` og `grandmaster_started_by_this_test`), aktiverer hvert kameras PTP (som `connect_array()` gør), og inden den første trigger venter den på, at hvert kameras PTP-ur låser sig fast på grandmasteren og stabiliserer sig: `PtpStatus` Slave på én grandmaster, inden for 100 us af denne og stabil i 3 s, og hvert kamera låst på denne måde, uden afbrydelse, i 120 s — ca. 2 minutter fra opstart. Et ur, der netop er låst fast, afviger med op til ~200 us i ca. 2 minutter, efter at det har læst »Slave«, og kameraets eget `PtpOffsetFromMaster` viser det ikke; 120 s bringer kameraerne inden for ~10 us af et stabilt ur. Et kamera, der stadig er låst siden en tidligere test, hvor PTP blev efterladt tændt, som denne backend så det, tæller fra den låsning, så en test, der udføres umiddelbart efter en anden, behøver ikke at vente igen; et kamera, som testen tænder, eller et, der findes med PTP aktiveret, og som denne backend ikke har set låse sig fast (tændt et andet sted eller før en genstart af backenden), venter sine 120 s, og i en række med begge kameraer, der netop er låst fast, venter de stadig på deres tid. Et kamera, der forlader Slave-tilstanden, overskrider 100 us eller følger en anden grandmaster i 3 aflæsninger i træk (1,5 s) under ventetiden, starter sin tælling forfra; en eller to sådanne aflæsninger logges (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`), og tællingen fortsætter, og et grandmaster-id, der ikke kan aflæses, betragtes som ukendt, ikke som en ændring. Når alle kameraers tælling genstarter på én gang (inden for et sekund), skyldes det grandmasterens tidsforløb, ikke kameraernes: backend'log viser `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` opføres i `grandmaster_steps`, og en ventetid, der derefter udløber, starter sin fejlmeddelelse med `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Grænsen forlænges ikke for dette. `ptp_settle_s` (0-600; `0` uden afventning af stabilisering) indstiller stabiliseringstiden for kørslen; en anden værdi afvises (HTTP-kode 400). Ventetiden er begrænset til afregningstiden plus 90 s (210 s som standard, mindst 90 s), og opkaldet venter på backend-systemet i `timeout` s: som standard 240 plus afregningstiden (360 s), ligesom `chloros-mcp`’s `test_array_sync` gør. `ptp_lock` har `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (hvor længe hvert kamera har været låst fast og stået stille), `found_settled` (de kameraer, der allerede var låst fast i stabiliseringsperioden, da de blev fundet) og, pr. kamera, `status`, `offsets_ns`, `slave_s` (hvor længe det har været låst; Ingen: ikke låst) og `remaining_s` (Ingen, når intet kameras PTP ville tænde). Backendens log indeholder en linje hvert 5. sekund med hvert kameras tilstand, hvor længe det har været låst, og hvor lang tid der er tilbage. Hvert kamera tændes med PTP slået fra; herefter synkroniseres urene på ca. 10 sekunder. Bagefter sættes hvert kamera tilbage til den tilstand, testen fandt det i — dets PTP, den automatiske eksponering og forstærkning, som testen holder slået fra under kørslen (`ExposureAuto` / `GainAuto` og en manuel `ExposureTime` / `Gain`), og resten, medmindre `keep_settings=True` — så et kamera, der er efterladt med PTP aktiveret (`PtpEnable=True`, eller et testforløb med `keep_settings=True`) forbliver låst, og når denne backend har set, at det har været låst i afregningstiden, venter en senere test ikke på det. Hvis urene ikke afregner, er `success` falsk før enhver udløser, og `error` angiver, hvad der skal gøres: et kamera, der aldrig er blevet låst (`not_slave`), betyder, at grandmasteren ikke når frem til det (`time_sync_status()`); et kamera, der hele tiden faldt ud, navngives efter, hvor længe det har været låst (`remaining_s`); ure, der stadig konvergerer (`far`): tjek `time_sync_status()`, og for at få dem låst fast før testen skal du aktivere PTP og lade det være slået til (`PtpEnable=True` på hvert kamera, eller `keep_settings=True`; testen gendanner de fundne indstillinger, så den forbliver tændt), og kør den derefter. `lattice multi-test`. |
| `detect_array_master(line=None)` | Kameraerne på synkroniseringslinjen og en standardmaster: `master_serial`, `slave_serials`, som `line` har scannet. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Hvilken række af disse kameramodeller (produktnavne, `"LATT-M3C-L87-FOCN"`) der ville dukke op på denne computers netværk: `tier`, `achievable_fps_max`, kabelforsyningen (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` og `fix_suggestions`. Som standard vises de fundne kameramodeller; der er ikke åbnet noget kamera. For kameraer, du har, med de indstillinger, en forbindelse ville bruge, se [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Løs en justeringsprofil ud fra live-billeder og gem den. `method`: `orb`, `akaze`, `phase`, `checkerboard` (sammen med `checkerboard="9x6"`, de indre hjørner) eller `manual` (sammen med `control_points={serial: [[x, y], ...]}`, også referencekameraets). `model`: `translation`, `rigid`, `affine` eller `homography`. De mere detaljerede indstillinger er CLI-flagene: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` er `{serial: {option: value}}`. `rms_threshold_px` nægter at gemme en dårligere løsning. Returnerer `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (pr. kamera `reprojection_error`, `num_matches`) og `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Et justeret multibåndsbillede gennem profilen: `files` (multibånds-TIFF-filen med `bands` og dens størrelse, plus ét pr. bånd med `per_band` og ét pr. kamera med `per_camera`) og `vignette_corrected`. `bands` udvælger og `order` sorterer båndene efter navn (en liste eller en kommasepareret streng). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Strømjusterede multibåndsbilleder (de samme warp-indstillinger som `capture_aligned_multiband()`), for `count` eller indtil det stoppes. Returnerer en `CliJob`; dens endelige statistik tæller `frames_written` og eventuelle `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | En profilfil, der sammenfattes som `align-info`, udskriver følgende: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (pr. kamera `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (pr. kamera `band_names`, `wavelengths_nm`, `enabled`); `profile` er hele filen. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Ændrer profilfilen på stedet: `order` er båndnavne (hvert navn flytter sit kameras lag opad i den rækkefølge), `enable` / `disable` er serienumre. Returnerer den nye `layer_order`. `lattice align-reorder`. |
| `index_catalog()` | Indeksforudindstillingerne (`presets`: henholdsvis `name`, `channels`, `formula`) og farve `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Et indeks over en justeret multibånds-TIFF: en `preset` (`NDVI`, `NDRE`, ...) eller et `formula` over de symboler, som `channel_map` kortlægger til bånd (`{"NIR": "NIR_850", "Red": 0}`, efter navn eller indeks). Et forudindstillets symbol `channel_map` udelader det ene bånd i filen, hvis navn starter med det (`red` -> `Red_660`, `nir` → `NIR_850_RGN`), navngivet af selve filen (dens båndliste eller en Chloros-optagelses/produkts `Chloros:Band{R,G,B}Name`), så NDVI-værdien på en Chloros-reflektansoptagelse behøver ingen `channel_map`; to bånd, der svarer til ét symbol, afvises, de gættes ikke. `capture_level` angiver, hvad båndene indeholder: `reflectance` eller `radiance` gør værdierne kvantitative; en ukalibreret stak (`align-apply`'s) giver kvalitative værdier; `"unknown"` læser det fra filen, herunder en Chloros-fils `Chloros:ProcessingLevel`. Returnerer `stats` (form, 1–99-percentilintervallet, LUT-intervallet, den anvendte `capture_level`) samt den `files`, der er angivet ved siden af inputtet (`<input stem>_<preset>.tif`, `..._LUT.png`) eller ved `output`, en sti: med en billedfiltypenavn navngives filerne ud fra deres rod (`output="D:/out/ndvi_FRGN.png"` skriver `ndvi_FRGN.tif` og `ndvi_FRGN_LUT.png`), uden en er det et præfiks (`D:/out/ndvi` → `ndvi.tif`, `ndvi_LUT.png`); læs stierne fra `files`. De indeholder kameraoplysningerne fra inputtet (mærke / model / serienummer, GPS, objektiv, optagelsestidspunkt i UTC, kalibreringsidentiteten, men aldrig dens raw-DN-kalibrering), `Chloros:ProcessingLevel=index` samt indekset som deres eneste bånd (`XMP-Camera:BandName`, både TIFF- og LUT-PNG-filer). Kun filer; intet kamera. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Optag, kalibrer (`reflectance` eller `radiance`; `raw` springer det over), juster og beregn et indeks i ét trin; `save_multiband=True` gemmer den kalibrerede stak, som `compute_index()` kan hente senere. Returnerer `stats` (sammen med `serials`, hvis bånd den har brugt) og `files`. Den åbner `serials` (den første udløser triggeren), ellers profilens kameraer, hvis `layer_order`-post er aktiveret, først referencen (åbnes selv med dets bånd deaktiveret); et kamera, hvis bånd er deaktiveret, kalibreres aldrig. `channel_map` er kun nødvendigt for et symbol, som stakken ikke kan besvare, som i `compute_index()`: et forudindstillet symbols navn tager det ene aktiverede bånd, hvis navn starter med det, kontrolleret før noget kamera åbnes (hvis ingen eller flere svarer, afvises det, idet de navngives); et brugerdefineret `formula` kræver hvert symbol. `index.tif` og LUT'en indeholder stakkens kamera, optagelsestidspunkt (referencekameraets eksponering, UTC) og objektiv, og begge angiver indekset som deres eneste bånd (`XMP-Camera:BandName`, f.eks. `NDVI`: farve-PNG-filen angiver indekset, ikke dens farvekanaler). `lattice index --live`. |

### `CliJob`

Det håndtag, som de tre strømme returnerer. Jobbet kører på backend-siden, som beholder ejerskabet over sine kameraer, indtil det stoppes.

| Medlem | Beskrivelse |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (rammer gemt / skrevet, droppet, `fps`, `elapsed`, skrivefejl, `stop_reason`; et kontinuerligt job også `frames_grabbed` og `frames_queued` – hver indsamlet ramme skrives, sættes i kø, kasseres eller mislykkes – og `drop_reason`, `writers` og `write_ms`, den gennemsnitlige tid, det tog at skrive et billede) og `output_dir`. `state` er `running`, `ended` (den stoppede af sig selv og er endnu ikke blevet stoppet), `stopping` (en stop skriver sin kø ud: live-statistikker, samt `stopped_by`) eller `finished` (dets endelige statistik, stop-jobbet `error` og `stopped_by`). En registrering af et stoppet job opbevares i 10 minutter (højst de sidste 32) for hver klient: et andet script eller `chloros-mcp` læser den samme status og de endelige statistikker. Et job, der blev startet med `chloros-cli` (`lattice continuous`, `multi-stream`, `align-stream`) bliver stoppet af backend'en, når CLI'en holder op med at tjekke ind i 30 sekunder — afbrudt, da maskinen er i dvaletilstand — med `stop_reason`, `client_gone` og `stopped_by` i backend'en; at læse dens statistik herfra holder den ikke i live. Et job, der startes fra SDK, kører, indtil det stoppes. Hver start sender sin `timeout` som backendens deadline (`client_deadline_s`): et kamera, der åbnes efter det (et, der stadig starter op), sættes tilbage og lukkes, og der startes ingen opgave, og opkaldet genererer `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). Et kamera, som en opgave besidder, afvises ved enhver anden åbning (`connect_camera()`, en ny start, en midlertidig optagelse) med HTTP-kode 409 `CAMERA_HELD`, hvor jobbet og den, der startede det, angives. |
| `stop(timeout=30.0)` | Stop det og luk dets kameraer: `final_stats`, `output_dir` og `error`, når afbrydelsen ikke forløb problemfrit. Et andet opkald returnerer det samme svar, og det samme gør en stop-anmodning fra en anden klient: en, der sendes, mens den første stadig skriver sin kø ud, venter på den (op til 25 sekunder; derefter udløser den en fejl, og de endelige statistikker følger på `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Vent, indtil den afsluttes af sig selv (når `count` er nået, et fuldt drev eller en fejl), og udløs derefter `stop()` for at indsamle de endelige tal. `timeout` udløser `TimeoutError`, mens den stadig kører; et job uden `count` slutter først, når det stoppes. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | Dets id og type (`continuous`, `multi-stream`, `align-stream`), hvor den skriver, svar fra startruten og om `stop()` er kørt. |

Brug den som en kontekstmanager for at stoppe den ved udgang af omfanget.

---

## Projektmappe og opskrifter

**Projektmappen** er den mappe, hvor nye projekter oprettes, og som deles af Chloros-appen, CLI'en og SDK'en (modul `chloros_sdk.system_utils`; `chloros-cli project-folder`):

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

En konfigureret mappe, der ikke længere findes, falder tilbage til standardmappen (`Documents/MAPIR/Chloros Projects`), ligesom appen gør. En relativ sti, der angives til `set_project_folder()`, hentes fra dit arbejdsbibliotek, når backend-systemet kører på denne maskine.

**En opskrift** er den YAML-/JSON-handlingsliste, som `chloros-cli project run PATH RECIPE` kører mod et gemt projekt. `run_project_recipe()` kører den med CLI'egen runner (installeret med SDK'en), så en opskrift opfører sig ens uanset om den køres fra:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** er en `.yaml` / `.yml` / `.json`-fil eller selve opskriften: en ordbog med `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`) samt eventuelt `overrides` og `stop_on_error` (standardværdi true: kørslen stopper ved den første handling, der mislykkes). Handlingerne er [Recipe DSL](cli-reference.md#recipe-dsl) fra CLI-referencen; en relativ `output` opløses i forhold til dit scripts arbejdsmappe.
- **Returnerer** `ok` (alle handlinger lykkedes), `actions` (hvor mange), `log` (kørerens output; `echo=True` udskriver det også under kørslen) og `validation`. En opskriftsfil, der mangler eller ikke kan fortolkes, udløser `ChlorosConnectError`.
- **Kontrolleres før kørsel.**`validation` er `{"ok", "errors", "warnings", "steps"}`: hver handling skal have nøjagtigt ét kendt verbum som nøgle (`{"capture": {"name": ...}}`, ikke `{"action": "capture"}` eller `{"type": "capture"}`), der navngiver et kamera, et array eller en sensor, som projektet har, af den rigtige type. En opskrift med en fejl udfører**ingen** handling (`ok` false, fejlene i `log`), uanset hvad `stop_on_error` siger. Ubrugte nøgler, og alle arrays eller lyssensorer, som opskriften styrer, men som ikke er tilsluttet i øjeblikket, er `warnings`. En opskrift tilslutter ikke enheder: Tilslut projektets første (`ChlorosProject.connect_all()`, `chloros-cli project connect`, eller MCP-værktøjet `connect_project_devices`). `dry_run=True` kører intet og returnerer `{"recipe": ..., "validation": ...}`.
- Der kører kun én opskrift ad gangen i en proces. En YAML-opskrift kræver PyYAML.

---

## Projektautomatisering — `ChlorosProject`

Et gemt Chloros-projekt er en mappe, der indeholder `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` indlæser manifestet (`relink`: den mappe, hvor de manglende kildefiler fra et flyttet projekt nu befinder sig, og som bruges, når backend'en indlæser det — `process()` — se **Et flyttet projekt** under `ChlorosLocal`), og `connect_all` sætter alle gemte kameraer, arrays og lyssensorer online med deres gemte indstillinger — samme hardwaretilstand, som GUI'en ville skabe. Lyssensorer opretter forbindelse via backendens [DAQ-sensorpool](#daq-sensor-sessions), så de har brug for et kørende backend (se [`SensorHandle`](#sensorhandle)).

### Minimalt eksempel

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Eller som en kontekstmanager:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject`-metoder

| Metode | Beskrivelse |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Find og opret forbindelse til alle gemte kameraer, arrays og lyssensorer. Returnerer en forbindelsesrapport pr. klasse (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, hver med `connected`, `missing` og `errors`; `arrays` indeholder også `alignment`). Med et backend, der lytter på `127.0.0.1:5000`, findes de gemte kameraer og arrays og tilsluttes via dette. Hvis der ikke er noget, finder og styrer denne proces dem selv via `lattice_sdk`, som kræver Arena SDK-runtime og OpenCV (`cv2`) — SDK-hjulet installerer ingen af delene; når det ikke kan, vises hvert gemt kamera og array under `errors` med årsagen, og `missing` forbliver tomt. Et backend, der kører, men afviser SDK'et (logget ud: HTTP 401; en plan under Copper: 403), er ikke »intet backend«: hvert gemt kamera, array og lyssensor vises under `errors` med backendets årsag, og der er intet åbent i denne proces. Hvert opkald tjekker backend'en igen, så `connect_all()` efter indlogning eller efter opstart af Chloros opretter forbindelse til det samme projektobjekt. Et gemt array, der allerede er oprettet i backend'en med de samme kameraer (GUI'ens eller `chloros-cli project connect`'s), tilknyttes som det er, uden at de gemte indstillinger anvendes på ny, og `disconnect_all()` lader det køre videre. Lyssensorer går altid gennem backend'en (se [`SensorHandle`](#sensorhandle)). Den opretter aldrig en backend. |
| `disconnect_all()` | Afvikl alt. Først frigives alle enheder; derefter, hvis backend'en ikke har bekræftet frigivelsen af et kamera, en array eller en lyssensor (afviklingen gik i timeout eller blev afvist), navngiver `ChlorosConnectError` dem hver især. En enhed, der allerede er væk, eller hvis der ikke kører noget backend, er tavs. Et array eller en lyssensor, som en anden klient havde åben, forbliver kørende. Projektets `with`-afslutning udløses aldrig. |
| `capture_all(output_dir=".")` | Én ramme fra hvert tilsluttet kamera + array, og ét spektrum (`SensorHandle.read()`) fra hver tilsluttet lyssensor, taget efter tur (kameraer, derefter arrays, derefter sensorer), ikke på ét tidspunkt. |
| `stream(camera, overlays=False, fps=10.0)` | Generator, der genererer BGR `numpy`-rammer fra et navngivet kamera (eller array). `overlays=False` læser rammer med håndtaget'`grab()` på `debayered`-niveau: gennem backend'en, når projektet er forbundet via en sådan, ellers en direkte `lattice_sdk`-indlæsningsloop (arrays genererer `{serial: frame}`-dictionaries). `overlays=True` dirigerer gennem `ChlorosLocal.camera_stream()` → backendens `/api/camera/<serial>/stream-annotated` MJPEG-feed, hvor kameraets gemte `ui.overlay`-blok videregives som forespørgselsparametre. Kræver backend-tilstand, OpenCV (`cv2`) og et **selvstændigt kamera**: et kamera i direkte tilstand udløser `RuntimeError` (backend'en kan ikke hente et kamera, som denne proces ejer), og en matrix udløser `NotImplementedError` (overlejrer sammensat billede pr. kamera — streamer et element efter navn). Tilsvarende engangsudførelse: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Kør justering på alle aktuelt tilsluttede arrays. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Kør kalibrerings-/indekspipeline på projektets billeder (indkapsler `ChlorosLocal.process`; disse fire er de **eneste** accepterede kwargs — `indices=` osv. udløser `TypeError`; indekser indstilles via `ChlorosLocal.configure()`). Konstruerer en `ChlorosLocal()` på en lazy måde, som automatisk starter et backend. |

Attributter:
- `proj.cameras` — `Dict[str, CameraHandle]` indekseret efter navn OG serienummer.
- `proj.arrays` — `Dict[str, ArrayHandle]` indekseret efter navn OG array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` indekseret efter navn OG slot_id.
- `proj.config` — `project.json["config"]` ordbog.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Behandlingsniveauer.** `capture()`, `grab()` og `frame_stream()` bruger alle det samme `processing`
token, og kæden er kumulativ — hvert niveau kører alt, hvad der ligger over det:

| Niveau | Output | Bemærkninger |
| --- | --- | --- |
| `raw` | 1-kanals Bayer, sensor-native | Ingen demosaik. Overlejringer er ikke tilgængelige på dette niveau. |
| `debayered` | 3-kanals farve (**standard** for `CameraHandle` og alle `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` og `stream()` er som standard indstillet til `raw`): BGR i hukommelsen (`grab()` / `frame_stream()`), gemt som RGB ved hjælp af `capture()` | I backend-tilstand, som en optagelse: kalibreringspakkens mørkeforskydning (DSNU) fra rådataene, derefter OpenCV’s kantbevidste demosaik — ingen linearitets-LUT, fladfelt eller unmix. Det eneste niveau, der fungerer uden backend-tilstand, er kamerabibliotekets almindelige bilineære demosaik (ingen mørkeforskydning). Den gemte TIFF-fil er fotometrisk RGB, så vend kanalerne i en ramme i hukommelsen (`frame[..., ::-1]`) for at sammenligne den med en fil. |
| `radiance` | float32, W/m²/sr/nm, **ét plan pr. bånd** | Fuld radiometrisk kæde: demosaikering + 3×3-unmix (multispektral) + DSNU + flat-field + NIST-skala, hvor eksponering × forstærkning er divideret ud, så værdierne er absolutte. |
| `reflectance` | uint16, 32768 = 1,0 (eller uint8, 255 = 1,0 — se `bit_depth`), **ét plan pr. bånd** | Strålingsintensitet divideret med nedadgående irradians (ρ = π·L/E). Kræver en DLS/DAQ-aflæsning — se bemærkningen nedenfor. |
| `display` | 8-bit sRGB-lignende | GUI-ækvivalent gengivelse: CCM + hvidbalance + gamma via kameraets aktive farveprofil. |

I direkte (backend-fri) tilstand er kun `debayered` tilgængelig: `grab()` på et andet niveau forårsager, at
`NotImplementedError`, `capture()` ignorerer `processing`, og `frame_stream()` hæver
`RuntimeError` på hvert niveau. `reflectance` har brug for en brugbar nedadgående aflæsning — rammeendepunktet trækker
den samlede DAQ automatisk ind i kameraets DLS-slot, men uden en tilknyttet DAQ afviser kæden
reflektansudgangen og markerer nedgraderingen ærligt i de returnerede metadata i stedet for lydløst
at returnere et ringere produkt.

> **Reflektans DN-skala — hardkod den ikke.** LATTICE-reflektans bruger `32768` = ρ 1,0 og angiver
> XMP `Chloros:PixelScale=32768`; Survey3-reflektans bruger `65535` = ρ 1,0 og indeholder ingen
> `Chloros:*`-tags. Læs tagget og divider med det. Det er defineret i uint16-domænet, så det forbliver
> `32768` for alle formater, der skaleres om (16-bit TIFF, 8-bit TIFF, 8-bit PNG/JPG, 32-bit procent)
> — normaliser først den gemte datatype tilbage til uint16 (×257 fra 8-bit, ×65535 fra float). Den eneste
> undtagelse er en fil fra en build før 1.3.0: en 8-bit TIFF-fil blev dengang *beskåret*, ikke omskaleret, så
> der ikke er nogen skala, der beskriver den, og den indeholder ingen `PixelScale`- eller MicaSense-tuple. Behandl et manglende tag på en
> LATTICE-reflektansfil som »ingen gyldig skala«, ikke som en standard.

**Kalibrerings-tags: kun på `raw`.** `raw`-filen fra en optagelse indeholder kameraets kalibreringspakke: `CalibrationJSON`-payload og de MicaSense-kompatible tags (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), som beskriver sensorens råværdier. Hver anden fil på hvert niveau (`debayered`, `display`, `radiance`, `reflectance`, `index/`-renderingen) samt hvert billede fra kontinuerlig optagelse/multikamerastrøm indeholder kameraets identitet (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) samt billedrammens egne optagelsesdata, `XMP-mapir:FrameJSON` (eksponering, forstærkning, temperatur, pixelformat, optagelsestid, DAQ-match), da hvert `process()`-produkt bærer denne identitet. Behandlingen finder en sådan fils kalibrering ved hjælp af dens serienummer og `CalBundleSha256` i Chloros' kalibreringscache og downloader pakken én gang, hvis denne computer ikke har den.

#### Filer lavet af flere kameraer

En fil lavet af bånd fra mere end ét kamera — en multibåndsstak (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), et live-indeks lavet af flere kameraer (`index.tif`, `index_LUT.png`), et array-optagelsesbillede `composite/` — navngiver hvert kamera, der indgik i det:

- **Per-kamera-lister, i båndrækkefølge** (sammensætningen: baggrundslag, derefter forgrund; et indekslag, de kameraer, som båndene stammer fra): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (den kalibreringspakke, som hvert kameras bånd er blevet behandlet med, ellers — en `capture_aligned_multiband()` / `start_aligned_stream()`-stak, hvis bånd ikke er blevet behandlet med nogen — den pakke, som kameraet Chloros har gemt i cachen, den, som dets optagelsesfiler er opkaldt efter), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — parallelle lister, hvor element *n* i hver beskriver det samme kamera; `-`, hvor et kamera ikke har nogen værdi (ingen kalibreringspakke, et ukendt objektiv). Hvert kamera udgør **ét** element: hvor dets pakke har flere værdier for et felt — en pakkes halogenlampecertifikater — de er forbundet med `;` (`F-1479-1;F-1480-1;F-1481-1`), aldrig med komma, så en liste viser ét element pr. kamera, uanset hvordan den udskrives.
- **EXIF `SerialNumber`** indeholder de samme serienumre, adskilt med komma (`214000535,214000551`).
- Den indeholder **ingen** identitet for et enkelt kamera (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): disse angiver et kamera og dets pakke, og behandlingen vil udlede filens kalibrering ud fra dem. Et indeks beregnet ud fra flere kameras bånd (`index.tif`, `index_LUT.png`) har EXIF-værdierne `Model` og `LATTICE` (riggen), ikke ét kameras `LATT-…`-model.

En fil med bånd fra **ét** kamera — en TIFF-fil pr. bånd, en `capture_aligned_multiband(per_camera=True)`-ramme, en stak eller et live-indeks fra et enkelt kamera — indeholder det pågældende kameras egne identitetsmærker, ligesom alle andre optagelsesfiler gør. `capture_aligned_multiband()`’s stak, bånd- og kameraspecifikke filer samt hver `start_aligned_stream()`-stak og dens båndfiler indeholder deres gruppes optagelsestid (referencekameraets eksponering, UTC), og et monokameras `per_camera`-billede er et enkelt plan, på `bit_depth` ligesom stakken.**Kameraretning (spejling/rotation).** Rammer fra `grab()` / `frame_stream()` er altid i
kameraets oprindelige retning på alle niveauer. Gemte optagelser følger kameraets retningsindstilling
(GUI’ens Mirror H / Mirror V / rotation):

- **`raw`- og `debayered`-filer er nøjagtigt det, sensoren har aflæst**, uanset indstillingen — en Bayer-
  mosaik, der er drejet eller spejlet, ville udgøre et andet farvefiltermønster, og kalibreringskortene er
  på sensorens eget gitter.
- **`display`, `radiance`, `reflectance` og indeksfilerne er orienteret** — drejet én gang efter al
  kalibreringsberegningerne. Deres clamp-masker ligger på det samme gitter.
- **Hver optagelsesfil registrerer indstillingen** i XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, med uret, efter
  spejlvendingerne) og `ChlorosOrientationApplied` — `false` på rå og debayered, `true` på de orienterede
  produkter. `ChlorosProcessing` angiver de operationer, der er anvendt på den pågældende fils pixels (`none`, når
  den er i oprindelig form). Dette omfatter hvert enkelt billede i en kontinuerlig optagelse (kameraets egen indstilling,
  ikke anvendt) og i en strøm fra flere kameraer (identiteten: dens medlemsbilleder er native, ligesom ved en
  array-optagelse).
- **Behandling af et projekt anvender den registrerede orientering** på de produkter, det fremstiller ud fra en rå
  (først kalibrering i sensor-koordinater, derefter array-justeringsforvrængning, til sidst orienteringen),
  så de matcher produkterne fra live-optagelsen — inklusive de ukalibrerede sensorrespons-/vignetteringskorrigerede
  reserveprodukter; dets debayered-produkt forbliver i oprindelig form.
- **Chloros-billedviseren viser en rå eller debayered fil, der er roteret i henhold til dens optagelse** — kun til visning,
  filen ændres ikke; en fil uden optagelsen vises som den er gemt.
- **Optagelser uden optagelsen behandles som native.** Før version 1.3.0 roterede en optagelse med et enkelt kamera
  selv sine RAW- og debayered-filer; Chloros kan ikke skelne en sådan RAW-fil fra en uændret fil og
  gætter ikke.
- **Array-optagelser bevarer som standard alle medlemsfiler i deres oprindelige form** og registrerer identiteten;
  `ArraySession.capture(orient=True)` konverterer deres produkter (se kontrolelementerne for array-optagelser).

#### Udgang pr. bånd og blandede arrays

På niveauerne `radiance` og `reflectance` returnerer et kamera **ét plan pr. bånd for det pågældende
kamera** — tre for et M3C (dets tre ublandede bånd, BGR-kanalrækkefølge), ét for et M3M (dets
enkelt smalbåndsfilter). Monokameraets replikerede planer sammenlægges, så et **blandet
M3C/M3M-array giver forskellige arrayformer pr. serie**. Læs `bands`-nøglen fra rammens
metadata for at finde ud af, hvad hvert plan er; udled det ikke ud fra antallet af kanaler:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Kontroller altid `hdr["processing_level"] == "reflectance"`, før du bruger værdierne. En nedgradering
(ingen DAQ-aflæsning, ingen kalibreringspakke, ukendt båndlayout) rapporteres i `processing_level` og
`radiometric_refused` i stedet for at blive opgraderet — den array, du får tilbage, er stadig en gyldig ramme, bare
ikke det produkt, du bad om. En gemt optagelse, der er blevet nedgraderet, har den samme kode i sin XMP
som `Chloros:RadiometricRefused` (f.eks. `no-band-layout`, `no-dls-snapshot`).

#### Én trigger pr. aflæsning — `group_id`

**Array-medlemmer leveres fra arrayets synkroniserede billedgruppe, ikke fra hvert enkelt kameras egen
optagelsesbuffer.** Kameraerne i et hardwaresynkroniseret array eksponerer samtidigt, men N aflæsninger pr. kamera
*samples* ikke samtidigt — aflæsning af hvert kameras seneste bufferede billede strækker sig over triggergrænserne, og et enkelt tabt billede desynkroniserer sættet, uden at nogen bemærker det.

Levering fra gruppen er nødvendig, men ikke tilstrækkelig i sig selv: hver anmodning tager stadig
et øjebliksbillede af *den seneste* gruppe på sit eget tidspunkt, og grab-løkken kan offentliggøre imellem.
Målt på et live-array med 4 kameraer **strakte 12 ud af 20 parallelle aflæsninger sig over to grupper** — altid
nøjagtigt tilstødende. Så aflæsningen er **fastlåst**:

1. Hvert billede bærer `group_id`, arrayets monotone udgivelsestæller.
2. `grab()` læser masteren (eller, hvis masteren ikke kan betjenes, det næste kamera, der kan),
   hvorefter de resterende kameraer fastgøres til den gruppe-id, den landede på (`?group_id=N` på HTTP-
   endepunktet).
3. Backend’et opbevarer de seneste grupper i **mindst et halvt sekund**, så en søstergruppe, der allerede
   er blevet overhalet af senere udsendelser, stadig betjenes af den pågældende trigger — længe nok til, at en
   refleksionseksport kan nå frem og tilbage, selv på et array med 9 kameraer med fuld sensor. En tilknytning, som backend'et ikke
   længere opretholder, afvises med `409` og `group_aged_out: true` i stedet for at blive besvaret uden fejlmeddelelse
   med en anden trigger — `grab()` genlæser sættet.

Genmålt med pinning: **20 ud af 20 aflæsninger returnerede én trigger.** `grab()` gør dette for dig;
du har kun brug for `group_id=`, når du driver HTTP-endpointet direkte.

- Et kamera, der ikke kan betjenes i den fastgjorte gruppe — mangler i den eller afviser anmodningen
  (et `band`, det ikke har, et niveau, det ikke kan producere) — **udelades i stedet for at blive udfyldt**
  fra en anden trigger, og de øvrige kameraer returneres stadig.
- Den ordbog, som `grab()` returnerer, er en `GrabResult` (en almindelig `dict`-underklasse;
  `from chloros_sdk import GrabResult`): `.errors` knytter hvert
  udeladt serienummer til den undtagelse, som dets læsning udløste, og `.group_id` er den delte gruppe. Et tomt
  `.errors` betyder, at alle kameraer udgør denne gruppe.
- Et enkeltstående kamera har ingen gruppe, rapporterer `group_id: None` og **afviser** en `group_id=`
  pin i stedet for at ignorere den.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Den samme tæller offentliggøres på **hver enkelt MJPEG-strøm pr. kamera** for et medlem af et array —
forhåndsvisningen (`/api/camera/stream/<serial>`), den annoterede forhåndsvisning (`/stream-annotated`) og
reflektansstrømmen (`/stream-reflectance`) — som `X-Chloros-Group-Id` på hver del, ud over
`X-Chloros-Frame-Id` og `X-Chloros-Timestamp-Ns`. Strømme er kontinuerlige snarere end
øjebliksbilleder, så de matches ved at indsamle dele og gruppere efter den pågældende header — der er ingen
pin, der skal indstilles. Et enkeltstående kamera har ingen gruppe. Se
[MJPEG-strømme pr. kamera](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` bliver aldrig tavs.** Et tick, der mislykkes af en årsag, der kan løse sig selv,
springes over — en kamerastrøm giver intet for det, en array-strøm returnerer de kameraer, der
svarede (muligvis ingen). Årsagerne er: ingen frame endnu; kameraets hentetråd er sat på pause i et
sekund eller to (`404 not streaming`, som backend’en svarer på, når et andet kamera opretter
eller afbryder forbindelse, eller når en indstillingsændring omfordeler puljen); en genstart af backend eller `5xx`; justering
er stadig ved at falde på plads (`409 … retry shortly`); og en afvisning af reflektans, fordi DAQ'en endnu ikke har nogen måling
(`no-dls-snapshot`, en forældet eller tom måling). En afvisning ved næste tick ville blot gentage sig — en
ukendt `band`, `bit_depth=8` ved strålingsintensitet, en kalibreringspakke uden strålingsintensitetsskala, en reflektansafvisning
med **slet ingen DAQ** (ingen knyttet til kameraet eller dets array, ingen forbundet til
backend: afvisningens `dls_source` er `"none"`), enhver anden `4xx` —
udløser `chloros_sdk.FrameFetchError` (en `RuntimeError`, der bærer `.status` og `.body`) på
én gang. Det samme gør **60 sekunder** med springede ticks uden nogen ramme imellem, uanset hvad `fps` er — lang
nok til at klare justeringen og en genstart af backend, kort nok til at et kamera, der
virkelig er væk, afbryder streamen. Den pågældende opgivelsesfejl er også en `FrameFetchError`, både for et kamera og for
et array, med den sidste aflæsnings egen undtagelse på `.errors` (og, for et kamera, som `__cause__`). For
et array betyder »afvist«, at hvert kamera er afvist; undtagelsen `.errors` indeholder hvert kameras
årsag.

**En matrix `frame_stream()` afgiver hver trigger én gang** (både `ArrayHandle` og `ArraySession`). Et
tick, hvis nyeste sæt er den trigger, det allerede har returneret, venter på den næste trigger i stedet for
at returnere det samme sæt igen, så strømmen kører med den laveste hastighed af `fps` og arrayets egen rammehastighed, og på hinanden følgende sæt deler aldrig en `group_id`. Et array, der holder op med at udgive, genererer
`FrameFetchError` („ingen ny trigger i 60 s“) i stedet for at gentage sit sidste sæt. Uden dette ville en
strøm, der anmodede om mere, end arrayet udgav — eller startede lige efter opkobling — udlevere
gentagelser: i et målt forløb indeholdt 20 sæt 11 forskellige gruppe-id'er.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Eksport af et enkelt bånd — `band`

`grab()` / `frame_stream()` brug `band="<name>"` til at eksportere blot ét af et kameras bånd som et
enkeltkanals `(H, W, 1)`-array — den samme form, som et monokamera producerer, så udgang fra et enkelt bånd
håndteres på samme måde, uanset om det stammer fra et M3M eller fra opdeling af et M3C. Et FRGN’s røde bånd:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Accepterede stavemåder er det fulde layoutnavn (`Red_660`, `NIR_850`, `Green_550`) eller dets korte
hoved (`red`, `nir`, `green`), hvor der ikke skelnes mellem store og små bogstaver. Et ukendt navn afvises med en liste over de tilgængelige
navne.

> **Valget sker efter navn, ikke efter kanalnummer — bevidst.** »Kanal 1« er `Green_550` i
> arrayets BGR-kanalrækkefølge, men *rød* i filterkodensbogstavrækkefølge (`RGN`), og de to
> stemmer ikke overens. Vælger man den forkerte, får man en fuldstændig plausibel ramme fra det forkerte bånd, som
> intet nedstrøms kan registrere. Der er af den grund intet numerisk indeks.

#### Justering af strømmene — `align`

`align=True` tilpasser billedet til arrayets fælles referenceramme. Hvert kamera tilpasses
gennem den **samme** profilopløsning og beskærings-ROI, så strømme pr. kamera, der åbnes uafhængigt af hinanden,
kommer ud **gensidigt registreret** — du behøver ikke at anmode om dem sammen:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Krav og adfærd:

- Kameraet skal være i et **hardwaresynkroniseret** array (M8-synkroniseringskabel), der har en justeringsprofil
  — samregistrering forudsætter, at medlemmerne eksponeres samtidigt, hvilket et software- eller
  fri-kørende array ikke kan levere. Afvist med `409` i modsat fald.
- **Afvist, aldrig stiltiende ujusteret.** Optagelsesstien kan gemme ujusterede data med en advarsel,
  da en fil bevarer sine proveniens-tags og kan registreres på ny senere; en livestream har ingen
  sådan anden chance, så en manglende profil er en fejl snarere end en fallback.
- **Anmodning om justering starter processen.** Anmodning om `align` på et array uden profil igangsætter
  kalibreringen og returnerer `409` med det, den venter på (f.eks. *"venter på, at eksponeringen
  stabilisere sig på 214000551 (1/3)"*) — prøv igen, og det lykkes, typisk inden for nogle få snesevis af sekunder.
  Dette er vigtigt ved headless-drift: den automatiske justering ved opkobling giver op efter 30 sekunder, og hvert forsøg
  inden for dette tidsvindue springes over, mens smart-AE stadig er ved at stabilisere sig, så et array, hvis eksponering
  konvergerer langsomt, plejede at ende uden profil for hele sessionen og uden mulighed for at få
  en uden at åbne den kombinerede visning på skrivebordet. `alignment_status` rapporterer også
  `ae_align_holdouts`, hvor de angiver, hvilke kameraer der stadig er ved at stabilisere sig, samt deres fremskridt.
- Giv hver strøm i en matrix den **samme `max_h`**, ellers ender deres beskæringer på forskellige pixelgitter.
  
- Justeringen anvendes *før* bitdybdekodningen, så forvrængningen interpolerer værdier med fuld præcision.

#### `bit_depth` — bytte præcision mod båndbredde

`grab()` og `frame_stream()` accepterer `bit_depth=16` (standard) eller `bit_depth=8`:

| `bit_depth` | Reflektanskodning | Omkostninger |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, margen til ρ 2,0 | Fuld præcision. |
| `8` | uint8, der dækker `0..refl_max` (standard ρ 1,0) | Halvdelen af bytes; ~0,004 ρ-kvantisering ved standardloftet. Værdier over loftet beskæres — se nedenfor. |

##### 8-bit-loftet — `refl_max`

`refl_max` angiver den ρ, som 8-bit-intervallet dækker, så `pixel_scale = 255 / refl_max` og ρ er stadig
`pixel / pixel_scale`. `grab()` og `frame_stream()` behandler det som et nøgleord på både kamera- og
array-håndtag (`?refl_max=` på HTTP-endpointet). Forøg den for at skabe mere spillerum til spejlende
højlys, i forholdsmæssigt grovere trin:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | trin |
| --- | --- | --- |
| `1.0` (standard) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **Automatisk eksponering kan ikke rette en 8-bit-klipning, og at forsøge at bruge den gør dataene værre.**
> ρ = π·L/E, hvor eksponering og forstærkning er divideret ud, så ρ er en egenskab ved motivet, ikke ved
> kameraindstillingen — målt på et live-array flyttede en halvering af eksponeringen middelværdien af ρ med 1,3 %. At øge
> eksponeringen for at »tilpasse« rækkevidden får i stedet pixels til at gå i **sensor**-mætning, hvilket
> undervurderer L og skævvrider ρ **nedad** uden at der vises en fejl (målt: maks. ρ 1,203 → 0,985 ved
> 2× eksponering, gennemsnit −23 %). Vælg området med `refl_max`; at holde sensoren ude af
> mætning er netop det, AE allerede er beregnet til, og det beskytter ρ’s *gyldighed* snarere end dets område.

**Læs `reflectance_clipped_fraction`, ikke flaget.** `reflectance_clipped_at_1_0` udløses af en
enkelt pixel — i en reel scene rapporterede den `True`, mens brøkdelen var `0.000001`, én pixel
ud af en million. Brøken er det, der fortæller dig, om du skal hæve loftet.

**`bit_depth=8` er udelukkende refleksionsgrad.** Anmodninger om det til `radiance` afvises i stedet for at
blive imødekommet: reflektans har et fast interval på 0..1, der kan håndteres ved 8-bit-kodning, mens radiens ikke har noget
fast interval, så for at få det til at passe ind i 8 bit kræves der en percentiludstrækning pr. billede, hvis kortlægning
ændrer sig for hvert billede — et billede, ikke en måling.

#### Live-reflektansstrøm (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` leverer 8-bit reflektans pr. bånd som
`multipart/x-mixed-replace` — en M3C som 3-kanals JPEG, en M3M som gråtoner. Forespørgselsparametre:
`fps` (standard 10), `jpeg_q` (standard 95), `max_h` (0 = native), `band` (ét navngivet bånd, som
ovenfor), `align` (`1` til co-register — samme regler som SDK's `align`), `refl_max` (det 8-bit
kodningsloft, som ovenfor). Hver del indeholder
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` og `X-Chloros-Reflectance-Clipped`.
**Dele fra forskellige kameraer, der bærer det samme `X-Chloros-Group-Id`, udgør den samme trigger** —
sådan samles flere strømme pr. kamera fra et array igen, og dette garanteres
på serversiden, da array-medlemmerne leveres fra den synkroniserede billedgruppe (se
[Én trigger pr. aflæsning](#one-trigger-per-read--group_id)). Den returnerer `409`, når reflektansen er
utilgængelig ved opkobling, og afslutter strømmen, hvis den bliver utilgængelig under afspilning, så en fastlåst
strøm betyder »ingen reflektans længere« snarere end »stadig i orden, bare statisk«.

> **JPEG er tabsgivende — disse pixels er en måling.** Komprimeringsfejl påvirker direkte
> reflektansværdierne og er værst ved kanterne, hvor materialegrænserne er. Kvaliteten
> er derfor som standard sat til 95. Hvis du har brug for, at tallene forbliver intakte, skal du bruge
> `grab(processing="reflectance")` / `frame_stream(...)`, som returnerer det samme produkt uden
> codec i stien (og `bit_depth=8`, hvis ledningsomkostningerne, ikke codec'en, er begrænsningen).

> **EXIF overføres til eksporten.** `process()` kopierer GPS-blokken fra kildeoptagelsen
> **og dens ExifIFD** til hvert produkt, så eksportfilerne indeholder `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` og `CameraSerialNumber` samt
> georeferencen. `FocalLength` er det, Pix4D bruger til at beregne ground sample distance ud fra — uden det
> falder rekonstruktionen tilbage til en vildt forkert skala (i et målt tilfælde blev et område på 411 m
> forvandlet til et på 47,8 km). Kopien er bevidst ikke `-all:all`: IFD0’s strukturelle tags bryder
> LATTICE-outputtet, og `ExifImageWidth`/`Height` er udelukket, fordi de beskriver kildeoptagelsen
> snarere end det eksporterede raster.

Underflag for optagelsesfasen (gælder for de radiometriske niveauer — `radiance`, `reflectance`, `display`):

| Flag | Standard | Betydning |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + 3x3 unmix + NIST-radiometrisk skala. |
| `apply_white_balance` | `True` | WB LUT. DLS-kompatibel, når en DAQ er knyttet til kameraet. |
| `apply_index` | `False` | Evaluering af vegetationsindeks. |
| `index_expression` | `None` | Tilsidesæt formel. Ikke-tom → aktiverer indekset automatisk. |
| `annotated` | `False` | Overlejring af GUI-dekorationer (zebra/gitter/peaking). Ikke tilgængelig for `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Returværdien er en `chloros_sdk.CapturePaths`, af typen `CapturePathMap`, ikke `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` er `Dict[str, Union[str, List[str]]]`: en enkelt-niveau
> `processing` giver hver serienummer én sti, mens `processing="all"` giver den
> **ordnede liste** over alle produkter, der er gemt for det pågældende kamera. `ArrayHandle.capture()` tager
> ingen `levels=`, og den ignorerer nøgleordsargumenter, den ikke kender (`levels`, `daq`,
> `render_index`, `force_daq`), så angiv kun dem, der er vist her (`aligned=False`
> gemmer native filer, ligesom ved `ArraySession.capture()`); brug
> `ArraySession.capture()` til disse kontrolelementer. En live kombineret komposit, hvis en sådan blev streamet, ankommer under den ekstra
> `"combined"`-nøgle i stedet for under en serienummer. Kode, der antager `str`, går i stykker i
> listeformen uden at nogen typekontrol protesterer — annotationen sagde `Dict[str, str]`
> i et stykke tid efter, at listeformen blev udgivet, hvilket er grunden til, at aliaset findes. Normaliser
> når du ønsker den flade form:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Hvad der ikke kunne skrives: `.failed`.** Det kort, som `capture()` returnerer, er en
> `chloros_sdk.CapturePaths` — en almindelig `dict`-underklasse, så alt ovenfor fungerer
> uændret — som også indeholder `.failed`: de filer, som backend'et ikke kunne skrive,
> de samme poster som [`CaptureResult.failed`](#filer-der-ikke-kunne-skrives--captureresultfailed)
> (`serial`, `level`, `error` og `disk_full: True` på et fuldt drev). En delvis
> indsamling returnerer stadig alle stier, der blev skrevet; på et fuldt drev er oversigten
> tom, og `.failed` forklarer hvorfor. `.skipped` viser overspringelser pr. kamera, og
> `.group_seq` viser den synkroniserede gruppe, og `.alignment` / `.oriented` er
> backend-elementerne, ligesom på `CaptureResult`. `capture_smart()` returnerer dem som
> `result["frames"].failed` og `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Array-justering

`ArrayHandle` eksponerer hele justeringsfladen. Profiler er som standard kun gældende for den aktuelle session — kald `export_alignment()` eksplicit for at gemme dem.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Justering ved opkobling

`connect_all(align=...)` kan automatisk justere alle arrays ved opkobling:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Faldt tilbage til `project.json["config"]["auto_align_on_connect"]`, når ikke angivet.

### `SensorHandle`

En gemt lyssensor (DAQ-U, DAQ-M eller DAQ-E), der åbnes af `connect_all()` i backendens [DAQ-sensorpool](#daq-sensor-sessions) — den samme pool, som `connect_daq_sensor()` og `chloros-cli daq pool-connect` bruger.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` venter på sensorens **næste** spektrum — et, der allerede er bufferet, når du kalder, tæller ikke — og returnerer `(spectrum, is_saturated, integration_time, x, y, z)` eller `None`, hvis der ikke ankommer noget inden for 5 sekunder. En sat på pause strøm genoptages. `spectrum` angives i W/m²/nm, når sensoren er kalibreret.
- `.sensor` er sensorens [`DAQSensorSession`](#daq-sensor-sessions), for alt andet: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` og `.settings` er den gemte post fra `sensors.json`.

**Hvad `connect_all` gendanner.** Hver sensor åbnes på sin gemte adresse — dens COM-port, Bluetooth-adresse eller DAQ-E-vært — og hvis projektet har registreret sensorens id, skal det være netop den sensor: en anden enhed på den adresse afvises og anses for ikke at være tilsluttet. Dens gemte integrationstid, billedgennemsnit og automatisk eksponering anvendes, ligesom dens kap-korrektion:

| Gemt cap | Forbundet med |
| --- | --- |
| En cap-profil (f.eks. `fov_30`) | Den pågældende profil. |
| Ingen valgt | `sunshine_cosine`, standardindstillingen for tilslutning, registreret som antaget (`cap_id_source=auto_default`), så behandlingen giver en advarsel. Sensorerne leveres uden kappe: vælg den kappe, der er monteret. |
| `none` — valgt som **Ingen (sensor uden kappe)** i brugergrænsefladen | Ingen kappekorrektion. |

En DAQ-E-S bruger ingen hætteprofil, uanset hvad der er gemt: dens diffusor er en del af kalibreringen (se [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Et projekt, der er gemt af en tidligere Chloros, registrerede en sensor uden kappe som det eneste valg, så den opretter forbindelse igen med `sunshine_cosine`; vælg **Ingen (sensor uden kappe)** i GUI'en endnu en gang, og gem projektet.

En sensor, som en anden klient allerede har åben i puljen (CLI'en, et andet script), tilsluttes som den er, uden at indstillingerne anvendes på ny, og `disconnect_all()` lader den køre; en sensor, som `connect_all()` har åbnet, frigives. Ligesom med `connect_daq_sensor()` er Chloros-GUI'en **ikke** en del af puljen: luk en sensor i GUI’en, før du tilslutter projektet fra et script.**Krav og rapporten.** Lyssensorer kræver, at der kører et Chloros-backend, der er logget ind på et betalt Chloros+-abonnement (Copper eller højere), og `connect_all()` starter aldrig et sådant. Hvis der ikke kører et, vises alle gemte sensorer under `report["sensors"]["errors"]` med en angivelse heraf. En sensor, som backend'en ikke kan åbne, vises også der med backend'ens begrundelse; `report["sensors"]["missing"]` forbliver tom for lyssensorer.

---

## Direkte hardware (uden backend)

Når du ønsker nul afhængighed af backend for LATTICE-kameraer (CI, headless-robotter, indlejret), skal du importere `lattice_sdk` direkte — den reeksporteres af `chloros_sdk`. Bemærk vedrørende `CAMERA_AVAILABLE`: `lattice_sdk` findes i SDK-hjulet, men kræver, at Arena SDK-runtime og OpenCV (`cv2`) er til stede, og hjulet installerer ingen af delene. Lyssensorer går gennem backend'en — [`connect_daq_sensor()`](#daq-sensor-sessions) eller et projekts `proj.sensors` — med én undtagelse, nemlig DAQ-M-båndmagnitud-hjælperen [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), som også kræver `bleak`.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Forudindstillinger og triggeren

Tre af de fire forudindstillinger kører i **free-run**: kameraet eksponerer kontinuerligt, og en
`capture()` returnerer det næste billede. `triggered` er undtagelsen — den aktiverer
kameraet ved en hardwareflanke på linje 2, så den optager intet, før der kommer en.

| Forudindstilling | Udløser | Brug den, når |
| --- | --- | --- |
| `default` | free-run | generel brug |
| `high_speed` | free-run | 8-bit, 60 fps-begrænsning, kort eksponering |
| `high_quality` | free-run | 12-bit, ingen fps-begrænsning — det sædvanlige valg til stillbilleder |
| `triggered` | **klar, linje 2** | kameraet er tilsluttet via et M8-synkroniseringskabel, og noget andet udløser det |

Hvis du vælger `triggered` (eller selv indstiller `trigger_mode="On"`) uden at
linje 2 er aktiveret, vil alle `capture()` gå i timeout — hvilket er korrekt, da du bad
kameraet om at vente. SDK'et forklarer dette, når det sker; se
[SC_ERR_TIMEOUT under optagelse](#direct-hardware-backend-free).

> **Bemærk — "GVSP-probe" / `SC_ERR_TIMEOUT -1011`-meddelelser ved opkobling er ikke fejl.**
> Ved opkobling forsøger SDK'et at forhandle sig frem til**jumbo-rammer** (9000-byte GVSP-pakker) for at opnå højere gennemstrømning. På en direkte punkt-til-punkt-NIC-forbindelse (f.eks. en link-lokal `169.254.x.x`-adresse) kan netværket normalt ikke håndtere jumbo-rammer, så denne probe går i timeout og logger linjer som f.eks.:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Dette er den **indbyggede fallback-løsning**: SDK'en skifter automatisk tilbage til standardpakker på 1500 byte, og kameraet fortsætter med at oprette forbindelse som normalt (de efterfølgende `[chunk-enable …]`-linjer er en del af den normale forbindelsessekvens). Optagelse fungerer stadig.
>
> Du kan springe denne probe over, men **den er ikke blot en log-dæmper — den slår jumbo-rammer fra.** Kameraet svarer kun på »Don't-Fragment«-pings op til 1500 byte, uanset hvor godt dit netværk er, så ping-testen alene kan aldrig finde jumbo-rammer; denne probe er det eneste, der kan. Deaktiverer du den, kører kameraet standardpakker på 1500 byte for evigt, på ethvert netværk:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Det kan kun betale sig på et netværk, hvor du *ved*, at det ikke kan håndtere jumbo-rammer, hvor det sparer cirka et sekund i forbindelsestid pr. kamera. Da det er en reel afvejning snarere end en kosmetisk ændring, angiver SDK’et nu dette, når du bruger det:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Lad det være, medmindre du har en grund til at ændre det.**Hvis den forbliver aktiveret, måles hvert kamera én gang pr. backend-session og igen, når dets forbindelse ændres (en anden adapter, en anden forbindelseshastighed eller en forbindelse, der er gået ned, som det sker ved udskiftning af en switch); ved genopkobling genbruges resultatet.**Efter en ændring bag en switch, der forblev oppe, skal du genstarte backenden for at måle igen.** En første forbindelse kan lejlighedsvis tage et par minutter ved denne kontrol, og det fremgår af loggen, når det sker.
>
> Hvis du *ønsker* jumbo-throughput, skal du aktivere jumbo fra ende til ende (NIC MTU 9000 + en switch, der lader dem passere), eller fastsæt den med `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, når du ved, at forbindelsen understøtter det — men foretræk en `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` pr. kommando frem for at indstille den permanent, da en fastlåst størrelse springer sonden over og holder op med at tilpasse sig det forudgående netværk. **Hver eneste** enhed i stien skal kunne videregive jumbo-pakker — herunder enhver PoE-splitter eller -injektor, hvilket er den almindelige årsag til, at en ellers jumbo-kompatibel opsætning ikke kan håndtere dem.

> **`SC_ERR_TIMEOUT -1011` under `capture()` / `grab*()` er et andet problem — det er en reel fejl.**
> Ovenstående bemærkning vedrører kun `-1011`, der er logget af**connect-time-proben**. Den samme fejl, der opstår ved en**capture**, betyder, at kameraet er tilsluttet korrekt, men ikke sender billeder:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Det afslørende tegn er et kamera, hvis *kontrol*-kanal fungerer korrekt — opdagelse fungerer, indstillinger og `[chunk-enable …]`-skrivninger lykkes alle — mens *hvert* billede går i timeout.
>
> **Den sædvanlige årsag er, at kameraet er indstillet til en hardwareudløser.** Med `trigger_mode="On"` og `trigger_source="Line2"` sender kameraet slet intet, før der kommer en elektrisk flanke på M8-synkroniseringskablet. Hvis der ikke er noget kabel, der driver den linje, venter hver eneste hentning i al evighed. Kameraet er ikke i stykker, og netværket fungerer fint — det gør præcis, hvad det er blevet bedt om.
>
> `CameraSettings()` og `default` / `high_speed` / `high_quality` indstiller til fritløb, og en optagelse, der går i timeout, mens den er aktiveret, forklarer sig selv i stedet for blot at vise en `-1011`. `PRESETS["triggered"]` aktiverer Line2, som det er beregnet.
>
> Sådan tvinges et hvilket som helst kamera til friløb:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Hvis der stadig opstår timeout med `trigger_mode="Off"`, leverer kameraet reelt ikke data — send os loggen og `ip link show`.

#### Farveprofiler (RGB-livevisning) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` vælger skærmens farveprofil til **livevisningen** på RGB-kameraer (multispek-kameraer ignorerer indstillingen):

| Profil | Betydning |
| --- | --- |
| `raw` | Omgå den radiometriske kæde fuldstændigt. |
| `linear` | DSNU + flat + WB, ingen CCM, ingen gamma. |
| `natural` | Lineær + målt CCM + sRGB-gamma, kun med den enkle efterbehandling (chroma-udjævning + desaturering af højlys) — den realistiske standardindstilling. |
| `enhanced` | `natural` plus den fulde »hub-parity«-efterbehandling (defringe, vibrance, CLAHE lokal kontrast). Et mere fyldigt udseende til omtrent **det dobbelte af efterbehandlingsomkostningerne pr. billede**, hvilket giver en lavere LIVE-billedhastighed. |
| `custom_temp` | `natural`, men hvidbalancen fastsat til `custom_cct_k` Kelvin (DLS ignoreres; fastlåst til 2000–10000 K på backend-siden). |

Profilen er en **kun-live-preview**-knap til hastighed/udseende: gemte optagelser får altid den fulde, rige finish uanset den valgte profil, så at vælge `natural` for at vinde billedtid tilbage sænker ikke kvaliteten af det, der ender på disken. En ukendt profil aktiverer `ValueError`; når en chloros-backend er tilgængelig, sendes ændringen også til den via POST, så den næste forhåndsvisningsramme afspejler den (direkte SDK-brugere uden en backend får stadig ændringen i indstillingerne).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Højdepunkter (RGB-skærm) — `set_highlights`

`LatticeCamera.set_highlights(value)` trækker de lyseste, hvideste pixels fra **skærmen** på RGB-kameraer — løsningen på, at en lys hvid overflade gengives som en flad hvid klat. `value` er `-100`..`0` (begrænset); `0` er slået fra, og gengivelsen er da nøjagtig den samme som uden indstillingen. Stærkere indstillinger sænker skærmens hvidniveau (til ca. 60 % ved `-100`) og udvider tonekurven, så en overflade, som forhåndsvisningens automatiske lysstyrke har skubbet til rent hvidt, får sine detaljer tilbage.

Det er en skærmkontrol, men i modsætning til mætning og kontrast (som kun ændrer live-forhåndsvisningen) ændrer den også det, du gemmer: live-forhåndsvisningen og **skærmniveau**-optagelser følger den, så disse filer bliver også mørkere (de har `Chloros:DisplayHighlights`); eksportfiler med strålings- og reflektansdata gør det aldrig, og den kan ikke gendanne sensordata, der er klippet — sænk eksponeringen for det. Multispec- og monokameraer ignorerer den. Værdien gemmes på `settings.highlights` og sendes via POST til et tilgængeligt chloros-backend, præcis som `set_color_profile`; CLI-ækvivalenten er `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Monokameraer (M3M) og `Calibration`

Et monokamera **M3M** (`M3M-<lens>-F<wavelength>`) er enkeltbåndet: ét gråtoneplan, ingen Bayer-mosaik, ingen 3×3 spektral-crosstalk-matrix. `Calibration` genkender det og viser et `is_mono`-flag. Reflektans gælder stadig som et radiometrisk kort pr. bånd (unmix er identitetsmatricen), men multibåndsberegninger på et enkelt kamera giver meningsfulde resultater i stedet for nonsens:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

For at opbygge et vegetationsindeks fra monokrom hardware skal man kombinere flere M3M-kameraer ved forskellige bølgelængder til en justeret multibåndsstak (se [Arrayjustering](#array-alignment)) og beregne indekset på tværs af denne stak i stedet for på et enkelt kamera.

#### DAQ-M-båndmagnituder — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` opretter forbindelse til en DAQ-M via Bluetooth ved hjælp af sin egen BLE-MAC. Det er den eneste klasse af lyssensorer, der kommunikerer direkte med sensoren i stedet for via backend, så den har brug for det samme som de direkte kameraklasser (`CAMERA_AVAILABLE`: Arena SDK-runtime og OpenCV) plus `bleak` til Bluetooth (`pip install "chloros-sdk[camera]"` eller `pip install bleak`; uden den udløser `DLS` en fejl (`DLSError`)). En DAQ-M, der er tilsluttet et andet sted (backendens pool, et projekt, GUI'en) holder op med at annoncere, så frigiv den der først. **`mac_address` er påkrævet**: der er ingen standard, da hver enhed har sin egen, og `DLS()` uden en sådan udløser `DLSError`, før der forsøges nogen forbindelse. Find MAC-adressen med `discover_daq_sensors(transports=["ble"])` (se [Discovery](#discovery--finding-an-address-to-connect-with)) eller `chloros-cli daq pool-discover --only ble`.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integrerer et DAQ-M-spektrum (den seneste måling, når `spectrum` er `None`; `DLSError`, hvis der endnu ikke er nogen) gennem et kameras tre kanalresponser og returnerer `{"red": …, "green": …, "nir": …}` — nøglerne navngiver kanalerne i den rækkefølge; med et RGB-kameras kurver er den tredje (`nir`) dets blå kanal.

- Uden `spectral_response` anvender den filtermodelens standardkurve (den, som `filter_model` og `DLS` blev bygget med), som blev målt på én enhed.
- `spectral_response` anvender i stedet kameraets **egne** målte kurver: en dict `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, som er gemt i kameraets kalibreringspakke (`color_correction.spectral_response_smoothed` — RGB-kameraer; multispektrale pakker indeholder ikke en sådan kurve). Hvis en dict ikke kan bruges, falder den tilbage til flådekurven.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

CLI's `chloros-cli lattice dls --camera SERIAL` gør det samme via backend'en ved at finde kameraets kurve i dets indlæste eller cachelagrede kalibreringspakke og angiver, hvilken kurve der blev brugt. Via HTTP er det `POST /api/dls/cli-read` med `mac` (påkrævet: uden dette er svaret HTTP-fejl 400, `code` `MAC_REQUIRED`) og `camera_serial` i brødteksten; svarets `band_magnitudes_source` er `camera <serial>`, når kameraets egen kurve blev brugt, eller `model <filter>` for flådekurven (et kamera uden kurve: ikke RGB eller ingen pakke indlæst eller gemt i cachen).

---

## Undtagelser

Fang basisklassen for at håndtere "alt, hvad der gik galt i Chloros":

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` og `ChlorosConfigurationError` er attributter til `chloros_sdk` (og af `chloros_sdk.exceptions`), men ikke i `__all__`, så `from chloros_sdk import *` indlæser dem ikke: importer dem ved navn. `ChlorosError`-familien er afledt af `ChlorosLocal` og de funktioner, der er bygget på den (`process_folder()`, `process_lattice_capture()`); Smart-Connect-overfladen udvider `ChlorosConnectError`.

Hierarki:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## End-to-End-eksempler

### 1. Behandl en mappe med en brugerdefineret statusbjælke

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Live LATTICE-array → Refleksion + DAQ-reference

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Projektdrevet indsamlingskampagne

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Frame-stream fra flere kameraer → NumPy-pipeline

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Headless direkte hardware-optagelsesskript (uden backend)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Kapacitetsundersøgelse før tilslutning af et 4-kamera-array

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Tilsvarende optagelsesopskrift (ren Python)

`run_project_recipe()` kører en CLI-opskrift, som den er (se [Projektmappe & opskrifter](#project-folder--recipes)). De samme trin, skrevet i Python, er:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## MCP-server (AI-assistenter)

`chloros-mcp` gør det muligt for en AI-assistent, der kommunikerer via [Model Context Protocol](https://modelcontextprotocol.io) (MCP), såsom Claude Desktop, Claude Code eller Cursor, at styre Chloros: behandle billeder (oprette et projekt, importere en mappe med optagelser, vælge indstillinger, behandle, rapportere, hvad kørslen har eksporteret) og betjene LATTICE-kameraer, -arrays og DAQ-lyssensorer (oprette forbindelse, forhåndsvise, indstille eksponering og farve, tage billeder, optage, opdatere firmware). Dens værktøjer dækker det, som denne SDK gør. Du stiller spørgsmål i almindeligt sprog, og assistenten kalder serverens værktøjer:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

Serveren er bygget på denne SDK: behandlingsværktøjerne kalder [`ChlorosLocal`](#image-processing--chloroslocal), og hardwareværktøjerne [smart-connect-kald](#smart-connect-for-lattice-cameras), [DAQ-sessioner](#daq-sensor-sessions), `ChlorosProject` og [kamera-hjælpeprogrammerne](#camera-utilities), som alle kommunikerer med det samme backend, som Chloros-appen bruger. Dens anmodninger bærer `X-Chloros-Client: sdk`, så backend'et anvender SDK'ets login- og abonnementskontrol, og SDK'ets regler følger med (f.eks. afvises Texture Aware for LATTICE-optagelser).

### Installation

Serveren leveres i `chloros-sdk`-pakken; der er intet andet, der skal installeres. SDK’et installerer en `chloros-mcp`-kommando, som er den samme som `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** Chloros-installationsprogrammet pip-installerer SDK'en i din Python, hvilket placerer `chloros-mcp.exe` i den pågældende Pythons `Scripts`-mappe.
- **Linux (.deb):** Pakken installerer SDK'en til systemet `python3` og placerer kommandoen `chloros-mcp` i `/usr/bin`, så den findes i alle brugeres `PATH`. I et virtualenv tilføjer `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` `chloros-mcp` til virtualenv'ets `bin`.

Serveren bruger kun standardbiblioteket oven på SDK'en (Python 3.7+). En MCP-klient starter den og kommunikerer med den via stdin/stdout; du kører den ikke selv, undtagen med `--help`.

### Klientopsætning

**Claude Desktop:**tilføj serveren til `claude_desktop_config.json` (på Windows er det `%APPDATA%\Claude\claude_desktop_config.json`;**Indstillinger > Udvikler > Rediger konfiguration** åbner den), og genstart derefter Claude Desktop:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude-kode:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Cursor:** den samme `mcpServers`-post som i Claude Desktop, i `~/.cursor/mcp.json` (alle projekter; `%USERPROFILE%\.cursor\mcp.json` på Windows) eller `.cursor/mcp.json` i ét projekt.

Indstillingerne placeres i `args` eller efter kommandoen til Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`). For at bruge en bestemt Python-SDK i stedet for den, der er angivet i `chloros-mcp` på `PATH`, skal du køre modulet med den pågældende Python-version (dens fulde sti som `command`, hvis den ikke er den første `python3` på `PATH`):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

En klient starter serveren uden din shell, så hvis den rapporterer, at serveren ikke kunne startes, er `chloros-mcp` sandsynligvis ikke på den `PATH`, som klienten ser. Angiv i stedet den fulde sti (`where chloros-mcp` på Windows, `which chloros-mcp` på Linux, hvor .deb-filerne er `/usr/bin/chloros-mcp`), eller kør modulet med den Python-version, der indeholder SDK'en.

### Indstillinger

| Indstilling | Standard | Hvad den gør |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | Den Chloros-backend, der skal bruges. |
| `--no-auto-start` | off | Start aldrig et backend: brug et, der allerede kører, f.eks. Chloros-appens, Linux-backend-tjenesten eller en tunnel til en anden maskines (se [Fjern-backend-tilstand](#remote-backend-mode-pip-only-host-via-tunnel); mappestierne skal i så fald henvise til den samme mappe på begge maskiner). |
| `--backend-exe PATH` | hvor Chloros installerer den | Den backend, der skal startes, når ingen kører, for en Chloros-installation, hvor SDK'et ikke søger. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` eller `ERROR`. Logfiler sendes til stderr, som de fleste klienter viser i deres MCP-log; stdout indeholder kun protokolmeddelelser. |

### Værktøjer

Hvert værktøjs argumenter er de samme som for det SDK-kald, det omslutter; klienten angiver hvert enkelt med dets fulde skema. Stier er absolutte.

**Behandling**

| Værktøj | SDK-kald | Hvad det gør |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Opretter et projekt og gør det til det åbne projekt, som behandlingsværktøjerne arbejder på. Et simpelt navn opretter mappen i standardprojektmappen; en absolut sti opretter den der. |
| `open_project` | `ChlorosLocal.open_project` | Åbner et gemt projekt og gør det til det åbne projekt. Stierne til et flyttet projekt omdirigeres til den mappe, det nu befinder sig i; `relink_folder` (absolut) angiver navnet på den mappe, hvor de øvrige manglende filer nu befinder sig. Returnerer `source_files` og `image_count`. Intet fundet: en fejlmeddelelse, der angiver, hvor de var, og navngiver `relink_folder`; intet er åbnet. |
| `import_images` | `import_images` | Importerer Survey3-parrene `.RAW`+`.JPG`, LATTICE `.tif`-optagelser og DAQ `.daq`-optagelser; returnerer `count` (billeder) og `scan_count` (optagelser). |
| `configure_processing` | `configure` | Ændrer de overførte indstillinger og ingen andre (værdier: [Understøttede værdier](#supported-values)); `custom_settings` sender en hel konfiguration som angivet. |
| `get_project_settings` | `get_config` | Læser det åbne projekts gemte indstillinger for behandling, eksport og indeksering. |
| `start_processing` | `process(wait=False)` | Begynder at behandle det åbne projekt og vender straks tilbage. |
| `get_processing_status` | `get_status` | Rapporterer `state` (`idle`, `starting`, `processing` eller `finished`) og fremskridt; når kørslen er færdig, angives `summary`, `export_check_failure`, hvis eksportfilerne ikke kunne verificeres, og `image_failure`, hvis billeder gik tabt: de øvrige produkter skrives; rapporter kørslen som mislykket. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Opretter et projekt for en mappe, importerer det, konfigurerer og starter kørslen i ét trin (`lattice: true` til LATTICE-optagelser). Mens en kørsel er i gang, afvises den, før den opretter noget, og backend afviser også en projektændring fra ethvert værktøj på det tidspunkt. |
| `export_light_sensor` | `export_light_sensor` | Skriver kalibrerede `.daq`- og `.csv`-filer til projektets lyssensor- optagelser uden at behandle billederne og henter fabrikskalibreringen fra MAPIR-skyen, når den ikke er gemt i cachen (se [Lyssensoroptagelser](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Læser et behandlet billedes Chloros-auditmetadata (kilde, kalibrering, anvendte trin). |

**Kameraer og arrays**

| Værktøj | SDK-kald | Hvad det gør |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Hvad der er tilsluttet lige nu, på enhver klient, hver med `connected_here`; plus serverens kørende opgaver. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Kameraer og lyssensorer, der kan tilsluttes. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Et kameras, en arrays (med dens kapacitet og tilstand) eller en lyssensors aktuelle tilstand. |
| `analyze_array_network` | `analyze_array_network` | Hvad denne vært og dette netværk kan håndtere for en foreslået array. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Tilslut et kamera eller et synkroniseret array (master først). |
| `connect_project_devices` | `open_project(...).connect_all` | Tilslut alle enheder, som et gemt projekt husker, med deres indstillinger. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Frigør et kamera, et array eller en lyssensor (se Adfærd). `keep_settings: true` på et array gemmer hver enkelt kameras indstillinger i sin egen hukommelse, `clear_settings: true` nulstiller denne hukommelse (resultatet medfører `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Projektets enheder samlet. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Læs eller skriv kameraindstillinger ved hjælp af et venligt navn eller et GenICam-nodenavn. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Fastlæg en arrays hastighed, eller sæt den tilbage til den automatiske hastighed. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | En optagelse (`fastest` og `wait_for_exposure` er `capture_fastest` og optagelsen med fast eksponering). Uden `output_dir` gemmes den i det åbne projekts`captures`-mappe, og filerne føjes til projektet (holdes tilbage, indtil en behandlingskørsel er afsluttet); svaret `project_add` bekræfter dette. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Kontinuerlige eller intervaloptagelser, som en opgave. |
| `start_array_recording` | `ArraySession.record` / `burst` | En video med kombineret indeks eller en rå burst som en opgave. |
| `build_burst_video` | `ArraySession.build_video` | Genbehandle en serie til kalibreret video som en opgave. |
| `get_job_status` / `stop_job` | | Følg eller stop en opgave. |
| `get_camera_preview` | | Hvert kameras live-forhåndsvisning, returneret som et billede. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Ét billede (eller ét synkroniseret sæt) på et behandlingsniveau, uden gemning: statistik pr. plan og et PNG-billede af hvert. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | En arrays justering. |

**Lyssensorer**

| Værktøj | SDK-kald | Funktion |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Tilslut en DAQ-U, DAQ-M eller DAQ-E (`cap_id` indstiller dens kapkorrektion). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | De seneste måleværdier: tidsstempel, mætning, spektrumområde og topværdi samt spektrumet gennemsnitligt fordelt på bølgelængdeintervaller (`full_spectrum` for hvert punkt). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Start eller stop strømmen. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Optag en `.daq`-fil som en opgave; `duration_s` stopper den automatisk. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | Korrektion af dækslet (`none` for en bar sensor). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Hvilken sensor leverer reflektansen til hvert kamera og hver matrix. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Båndsættene samt to streaming-sensorers reflektans og indekser for omgivelser + objekt. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | En DAQ-E's egne indstillinger, gemt i enheden. |

**Kameraværktøjer** ([Kameraværktøjer](#camera-utilities))

| Værktøj | SDK-kald | Hvad det gør |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` og deres indstillingsfunktioner | Et RGB-kameras skærmfarve: profil og farvetemperatur, mætning / kontrast, højlys. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Engangs-hvidbalance eller auto / fra / faste forhold. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Kameraindstillinger. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Filtermodellerne eller én efter navn. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Firmwareversioner og en flash (som standard en opgave). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Strøm- og termisk tilstand; dvaletilstand eller vågen. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Koefficienter fra et kalibreringsmål i visningen. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Kameraets netværkskort, et skøn og løsningen (Kun administrator). |
| `apply_vignette_correction` | `apply_vignette_correction` | Vignettekorrigering af en mappe med billeder. |
| `read_dls` | `read_dls` | En DLS-aflæsning. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | Værtens hardware og kørselstid. |

**Transiente kameraer, justeringsprofiler og indekser** ([Transiente arrays, justeringsprofiler og indekser](#transient-arrays-alignment-profiles--indices)). Hvert værktøj her, der åbner kameraer, tager `keep_settings` / `clear_settings` (standardværdien er false): se [Kamerahukommelse](#camera-memory--keep_settings--clear_settings).

| Værktøj | SDK-kald | Hvad det gør |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Optager ét kamera eller streamer flere synkront som en opgave (`duration_s` stopper det automatisk). |
| `capture_cameras_transient` | `capture_cameras_transient` | En synkroniseret optagelse uden at tilslutte et array. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Kontrol af udløsersynkronisering, og hvilke kameraer der er på synkroniseringslinjen. |
| `analyze_array_models` | `analyze_array_models` | Hvad et array af disse modeller ville opnå på dette netværk. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Løs og gem en justeringsprofilfil (`options` indeholder de mere detaljerede løsningsindstillinger). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Et justeret multibåndsbillede eller en strøm af sådanne billeder som en opgave. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | Indholdet af en profilfil; dens båndrækkefølge og aktiverede kameraer. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Indeksforudindstillinger og gradienter; et indeks over en multibånds-TIFF; optagelse til indeks i ét trin. |

**Tidssynkronisering, projektmappe og opskrifter** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Projektmappe og opskrifter](#project-folder--recipes))

| Værktøj | SDK-kald | Hvad det gør |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | PTP-grandmasterens tilstand, dens peers og hvert array-kameras PTP-tilstand. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Genstart den; indstil dens prioriteter. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | Den mappe, hvor nye projekter oprettes. |
| `run_project_recipe` | `run_project_recipe` | Kør en gemt projektrecept som en opgave med dens log. Kontrolleres først: En opskrift med en fejl afvises, og der køres intet; der gives en advarsel om ikke-tilsluttede arrays/sensorer (de tilsluttes ikke: `connect_project_devices` først). `dry_run` afslutter kontrollen og kører intet. |

Der findes ikke noget værktøj til at åbne et gemt projekt til behandling: hvert behandlingsjob starter med `create_project` (eller `process_folder`).

### Krav

- **Chloros skal være installeret på samme computer.** Chloros-backendet udfører behandlingen. Serveren kan i stedet bruge et backend, der nås via en tunnel (`--backend-url` sammen med `--no-auto-start`, se [Indstillinger](#options)).
- **Logget ind på den pågældende computer**, i Chloros-appen eller med `chloros-cli login`. Serveren kan ikke logge ind for dig: et værktøj, der kaldes, mens du er logget ud, svarer »Ikke logget ind på Chloros« med backendens begrundelse, og assistenten informerer dig herom.
- **Et hvilket som helst betalt Chloros+-abonnement (Copper eller højere).** Serveren kræver SDK-adgang, og det gratis Iron-abonnement har ingen: et værktøj, der kaldes på det gratis abonnement, svarer "Ikke tilgængeligt på dette Chloros-abonnement" med backendens begrundelse.
- Python 3.7 eller nyere med SDK og en MCP-klient, der starter lokale (stdio) servere.

### Fungerer sådan

- **Backend'en.** At starte serveren eller vise en liste over dens værktøjer starter intet; det første værktøjsopkald opretter forbindelse. Hvis Chloros-appen (eller Linux-backend-tjenesten) kører, bruger serveren dens backend og lader den køre videre. Hvis der ikke kører noget backend, starter serveren det installerede backend uden vindue. Når serveren afsluttes (du lukker klienten, eller den genstarter serveren), stopper den de indsamlingsopgaver og lyssensoroptagelser, den har startet, og frigiver de enheder, den har oprettet forbindelse til; backenden fortsætter med at køre, så en uafsluttet behandlingskørsel fortsætter til sin afslutning, og den lukker sig selv ned efter 30 minutter, hvis der ikke er noget, der bruger den. Med `--no-auto-start` starter serveren aldrig en backend: et værktøj, der kaldes, uden at der kører en, beder dig om at åbne Chloros-appen, og det næste kald, efter du har gjort det, opretter forbindelse.
- **Ét åbent projekt.** Backend'en har ét åbent projekt, der deles med Chloros-appen, når appen er åben, så arbejd ikke på et andet projekt i appen, mens assistenten er i gang.
- **Kun de indstillinger, der overføres, ændres.** `configure_processing` lader alle indstillinger, der ikke er angivet, forblive på deres gemte værdi (SDK's `configure()` nulstiller debayer, vignettekorrektion, reflektanskalibrering, PPK og eksportformat til deres standardværdier, når de ikke overføres; serveren udfylder dem fra projektet). Hvis den ikke kan læse projektets gemte indstillinger, ændrer den intet og meddeler dette, medmindre alle fem er angivet. `indices` erstatter projektets indeksliste. Resultatet viser, hvad der blev sendt som `settings_sent`, og navngiver under `warnings` alle indekser, som kameraets bånd ikke kan danne.
- **Behandlingen foregår efter »start-then-poll«-princippet.** `start_processing` returnerer, så snart kørslen starter, hvilket derefter tager fra minutter til timer, og assistenten følger op med `get_processing_status`. `state` er `idle` (ingen behandling), `starting` (backend har endnu ikke rapporteret kørslen, i op til 20 sekunder), `processing` eller `finished`, med kørslens [oversigt](#post-run-summary--hints): hvad den eksporterede, samt bemærkninger, hvis den eksporterede mindre end forventet (ingen billeder, intet kalibreringsmål fundet). Produkterne gemmes i projektmappen, ligesom fra appen. `start_processing` afvises, mens en kørsel er i gang, og et backend, der holder op med at svare, rapporteres som en fejl, ikke som en afsluttet kørsel.
- **Hvad assistenten ser.** SDK’s advarsler vises i værktøjets resultat under `warnings`, og fejl (afmeldt, plan, en forkert sti) vises som fejltekst, som den kan læse og videregive til dig. En liste med mere end 20 poster beskæres til de første 20, mens den fulde længde vises som `<key>_total`. Stier skal være absolutte.
- **Enheder deles.** Kameraer, arrays og lyssensorer findes i backend’en, så assistenten kan styre en enhed, der er forbundet via Chloros-appen, og ændringer vises i appen. `disconnect_device` viser, hvad serveren er forbundet med; en enhed, som en anden klient er forbundet til (appen, `chloros-cli`, et script, en anden `chloros-mcp`) frigives kun med `force: true`, som er SDK'ets `force_disconnect`, og afvisningen angiver, hvem der har tilsluttet den, når backend'et kender det; en enhed, som backend'et slet ikke har registreret, rapporteres som ikke tilsluttet (`released: false`, `connected: false`, intet at frigive). En optagelse eller stream, som backend'en kører, medfører, at dens `get_job_status`-visning ledsages af `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (billedgrupper for en stream med flere kameraer) — samt en enkelt linje `summary` ("220 frames skrevet, 55 droppet, 0 mislykkedes: ..."), mens den kører, og når den er færdig, dens endelige tal; en anden klients oversigt over den (`other_clients_jobs`) indeholder det samme (`list_cameras()`, `list_arrays()` og `list_daq_sensors()` indeholder hver sin enhedss `connected_by`: `client`, `label`, `name`, `at`). Hver `chloros-mcp` navngiver sig selv over for backend'en som `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, sendt som `X-Chloros-Client-Label`; ethvert SDK-script kan indstille det). Det kræver også `force` at stoppe en anden klients lyssensoroptagelse.
- **Langvarigt arbejde er en opgave.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, og enhver anmodning om indfangning, løsning eller indeksering af `background` returnerer straks en `job_id`; `get_job_status` følger efter, og `stop_job` afslutter den. En kørsel fortsætter, indtil den stoppes, medmindre du giver den `max_captures`, `duration_s` (eller, for en burst, `max_frames`); `min_free_gb` nægter at starte, når drevet har for lidt ledig plads. Der gælder ingen begrænsning, medmindre du indstiller en. En optagelse eller stream, som backend'en kører (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) beholder backendens job-id, så en anden `chloros-mcp` — en, der startes senere — finder den ved hjælp af dette id: `get_job_status` følger efter den (og viser sådanne job som `other_clients_jobs`, fra `GET /api/camera/cli-jobs`, hver med `started_by`), og `stop_job` stopper det først med `force: true`. Ethvert andet job (en optagelse, en array-optagelse eller burst, en baggrundsoptagelse) kører i den `chloros-mcp`, der startede den, og kun den server ved det: hvis en anden server, der har anmodet om det, angiver det. En stoppet opgave læser `stopping`, mens den skriver sin kø ud, derefter `finished` med de endelige tal, i 10 minutter og fra enhver `chloros-mcp`: den, der startede det, ser en andens tvungne stop på samme måde. `get_job_status` venter aldrig på et stop: under et (sit eget `stop_job`, en anden klients, eller det stop, der frigiver kameraerne fra et job, der er afsluttet af sig selv) svarer den straks med `stopping` og de aktuelle tællinger, og `stop_job` fra et job, der allerede er ved at stoppe, angiver dette. Et job, der er afsluttet, angiver årsagen i `stop_reason`: sin egen årsag (`no_frames`, `disk_full`, `member_missing`, eller for en kontinuerlig kørsel af en array `count`, `duration`, `array_gone`), `limit reached` kun når dets tællinger viser dets `count` (eller en bursts `max_frames`, en optagelses`duration_s`) er nået, `error`, `stopped` (med `stopped_by`), når en klient har stoppet den, ellers `finished`.
- **Hvor optagelserne gemmes.** Til det åbne projekts `captures`-mappe, hvor appen gemmer dem, medmindre `output_dir` er angivet.
- **Billeder.** `get_camera_preview` og `grab_frames` returnerer billeder, som assistenten kan se, ud over deres JSON-data. `grab_frames`-billederne er kontrastforstærkede til visning; statistikken er de reelle værdier.
- **Når serveren afsluttes**, stoppes de opgaver, den har startet, og en lyssensoroptagelse, den har startet, lukkes. Hvis serveren har startet backend'en, frigiver den også de enheder, den har tilsluttet, inden backend'en stoppes; i appens backend forbliver de tilsluttet.
- **Protokol.** Kun stdio (ingen HTTP-endpoint), kun værktøjer (ingen ressourcer eller prompter), tekst- og billedresultater, MCP-protokolversioner fra 2024-11-05 til 2025-11-25.**Hvad der ikke er med.** At logge ind eller ud (kontoen er din; serveren kan ikke logge ind) og at stoppe et backend, som serveren ikke har startet. Alt andet, som denne SDK gør med databehandling og hardware, har et værktøj. (En DAQ-E's konfiguration – dens navn, autentificeringstoken, OTA-adgangskode, signaturkrav, fabriksindstilling – og de direkte DAQ-servere hører udelukkende til `chloros-cli`, også i SDK’et: et forkert navn eller token spærrer enhver Chloros-installation ude af enheden.)

---

## Automatisk opstart af backend

Smart-connect-indgangspunkterne — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` og `discover_daq_sensors` — er tynde HTTP-klienter, der forudsætter, at en backend lytter på `127.0.0.1:5000` (smart-connect-grænsefladens standard-URL). Når GUI'en eller CLI'en allerede kører, er der en i gang. Fra et simpelt script er der muligvis ingen — så disse funktioner **starter automatisk den medfølgende backend-binærfil** (uden vindue, på samme måde som `ChlorosLocal` gør) før deres første opkald og venter derefter op til 60 s på, at den kommer op. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` og `analyze_array_network()` starter aldrig en.

Regler:

- **Der oprettes kun en lokal URL.** En `backend_url`, der peger på `localhost` / `127.0.0.1` / `[::1]` er gyldig; enhver anden vært antages at være en andens maskine og oprettes aldrig.
- **En backend, der afsluttes, før den svarer, afbryder ventetiden.** Opkaldet returnerer straks med afslutningskoden, den kommando, den kørte, og de sidste linjer, som backenden udskrev, i stedet for at vente de 60 sekunder ud. Afslutningskode 98 er undtagelsen: et andet backend har allerede porten eller låsefilen (Chloros-appens, backend-tjenestens eller et andet scripts, der stadig er ved at starte), så opkaldet fortsætter med at vente på, at netop det svarer. `ChlorosLocal` gør det samme.
- **Backend'et forbliver kørende med henblik på genbrug** (ligesom i CLI'en) — der sker ingen implicit nedlukning, når dit script afsluttes. Genkørsel af scriptet genbruger den aktive backend.
- **Fravælg med `auto_start_backend=False`** ved ethvert af disse opkald (f.eks. når du har peget på en fjernbackend, eller hvis du selv styrer backendens livscyklus).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Hvis den medfølgende binærfil ikke kan findes eller startes, udløser det efterfølgende HTTP-kald en handlingsbar, **platformspecifik** `ChlorosConnectError` i stedet for en simpel fejlmeddelelse om afvist forbindelse — på Windows henviser den til desktop-appen eller en `chloros-cli`-kommando; på Linux (ingen GUI) henviser den til en `chloros-cli`-kommando eller `.deb`.

---

## Miljø og headere

SDK'et markerer hvert backend-HTTP-opkald med `X-Chloros-Client: sdk`. Backend'et anvender SDK/CLI-licensregler (login **og** et betalt Chloros+-abonnement kræves) i stedet for GUI'ens gratis-niveau. Dette indstilles automatisk ved import – du behøver ikke at gøre noget.

Hovedet tilføjes til hvert `requests` / `urllib`-opkald, hvis URL indeholder `localhost` eller `127.0.0.1`, på enhver port — inklusive dine egne lokale tjenester — og erstatter en værdi, du selv har angivet. Opkald til andre værter forbliver uændrede.

Tilsidesæt backend-URL'en ved at angive `backend_url=` (`ChlorosLocal` accepterer kun `backend_url=` eller `api_url=`; `process_folder()` og `process_lattice_capture()` accepterer kun `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(En installeret backend lytter kun på loopback, så en LAN-`backend_url` kan ikke nå den; se [Remote-Backend Mode](#remote-backend-mode-pip-only-host-via-tunnel) for tunnelmønsteret.)

---

## Versionering og kompatibilitet

- SDK-versionen vises som `chloros_sdk.__version__`.
- SDK'et er bundet til den medfølgende backend-version. Det fungerer normalt at kombinere et ældre SDK med en nyere backend (fremadkompatible endpoints), men at kombinere et nyere SDK med en ældre backend kan medføre `404`-fejl på nye endpoints — opgrader desktop-appen, så den passer.
- Smart-connect-grænsefladen (`connect_camera` / `connect_array` / `connect_daq_sensor`) og netværksanalyse-endpointet returnerer stabile JSON-skemaer; nye felter tilføjes.

---

## Tips til fejlfinding

- **`ChlorosAuthenticationError: Login required`** (fra `ChlorosLocal`; Smart-Connect-kald genererer `ChlorosConnectError: Login required. …` af samme årsag) → Kør `chloros-cli login EMAIL PASSWORD` én gang på denne maskine, eller log ind via Chloros-desktopappen.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (eller `Failed to create project: …` fra `create_project()`) → Lange stier i Windows er deaktiveret på denne maskine (Windows-standardindstillingen), så ingen sti må overstige 260 tegn, og produkterne placeres op til ca. 130 tegn under projektmappen. Der blev ikke behandlet noget. Meddelelsen angiver den længste projektmappe, der passer: Brug en kortere, eller aktiver lange stier i Windows og genstart Chloros. Windows-installationsprogrammet tilbyder dette som »Aktiver lange Windows-stier«, der er markeret som standard; ellers skal du bruge gruppepolitik: *Computerkonfiguration > Administrative skabeloner > System > Filsystem > Aktiver lange Win32-stier*.
- **`ChlorosConnectError: No Chloros backend is running …`** → Smart-connect kalder automatisk en lokal backend, så denne fejl vises kun, når den medfølgende binærfil ikke kan findes/startes (f.eks. en pip-only-host uden desktop-pakke). Meddelelsen er platformafhængig: på Windows skal du åbne desktop-appen eller køre en vilkårlig `chloros-cli`-kommando; på Linux: Kør en `chloros-cli`-kommando (der findes ingen GUI) eller installer `.deb`. For en fjernbackend skal du angive `backend_url=` (og `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** ved import → `lattice_sdk` kunne ikke indlæses: Arena SDK-runtime eller OpenCV (`cv2`), kan ikke importeres i denne Python-version. Den backend-drevne overflade (`connect_*`, projiceres gennem en backend, `ChlorosLocal`) fungerer stadig.
- **Array forbinder ved fuld sensor, men en lav billedhastighed**→ Som designet: auto bevarer den fulde sensor og lader hastigheden falde til det, kablet kan klare. Brug `binning=2` / `4` eller en mindre `width` / `height` for flere billeder pr. sekund, eller aktiver jumbo-rammer / opgrader forbindelsen. Binning og ROI dækker**ikke** samlet over-subscription (`oversubscribed: true`, fps-felter 0): for mange kameraer til kabelforbindelsen kan ikke løses ved hjælp af binning/ROI — reducer antallet af kameraer, aktiver jumbo-rammer eller skift til et hurtigere netværkskort (se [Over-Subscription](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` rapporterer, at NIC'ens RX-ring er meget lille (~0,26 MB) / forbindelsesportene viser "FRAMES WILL DROP"** → Værts-NIC'ens modtagelsesring er på standardindstillingen (ofte nulstillet til 32 efter en opdatering af NIC-driveren). På et Realtek USB 10GbE-kort skal du indstille `ReceiveBufferLen=256` og `PendingReceives=64` (forhøjet) og derefter genstarte backend, så den genlæser ringen. Fuld procedure: [CLI-reference → Opsætning og finjustering af værts-NIC](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Værten fryser ved genstart/nedlukning, senere WMI-fejl med koden `Invalid class` / NIC'en kan ikke aktiveres** → Forældet USB 10GbE-driver forårsager `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Opdater adapterdriveren til en aktuel version (≥ 2026) og anvend modtagerringindstillingerne igen. Se [CLI-reference → Opsætning og finjustering af værts-NIC](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Array-indlæsning mislykkes med HTTP-fejl 503 "Ingen ny synkroniseret rammegruppe"** → Arrayet har ikke offentliggjort nogen ny rammegruppe inden for ventetiden. Kontroller `arr.status()`: `stalled: True` (eller `streaming: False`) sammen med `actual_fps`. Hvis 0 betyder, at indlæsningsløkken er stoppet med at offentliggøre — afbryd forbindelsen til arrayet og opret forbindelse igen. Hvis `stalled` er False, og `actual_fps` er større end 0, skal du prøve igen. Lige efter `connect_array()` venter en indfangning allerede op til 20 sekunder på den første gruppe.
- **`status()` viser `link_speed_mbps: 4294`** → Windows kunne ikke aflæse adapterens hastighed; dette er typisk for USB-Ethernet-adaptere og er ikke en fejl. Se [Forventede rammehastigheder](#expected-frame-rates).
- **`update_firmware()` mislykkes med `device never connected back on TCP <port>`** → En firewall blokerede DAQ-E's forbindelse tilbage til denne computer. Windows-installationsprogrammet tilføjer reglen (se [DAQ-E-enhedsindstillinger](#daq-e-device-settings)); med en anden firewall skal du tillade indgående TCP-trafik til backend fra sensorens undernet.
- **Reflektans afvist** → En aktiv DAQ skal være knyttet til kameraet (eller arrayet) for at opnå reflektans i absolut skala. Knyt enten via GUI'en, eller brug `processing="radiance"` (W/m²/sr/nm), som ikke kræver en parret sensor.
- **`smart=True`-optagelse tager længere tid end forventet** → AE-konvergens afhænger af scenens dynamik. Med `ArrayHandle.capture_smart()` skal du udvide `exposure_tolerance_pct`, forkort `stability_window_s` eller sænk `settle_timeout_s` for en hurtigere, mindre stabil udløsning.

---

## Se også

- [CLI-reference](cli-reference.md) — hver CLI-underkommando svarer til et SDK-kald.
- [DAQ-lyssensorer](https://mapir.gitbook.io/chloros/daq-light-sensors) i Chloros-manualen — sensorerne, deres dæksler og kalibrerede rækkevidde, optagelse, DAQ-E-netværk.
- [LATTICE-kameraer](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) i Chloros-manualen — tilslutning, kameraindstillinger, optagelsestilstande, arrays.
- Online-dokumentation: `https://mapir.gitbook.io/chloros/api-python-sdk`

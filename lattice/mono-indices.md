# Monokromkameraer og vegetationsindekser

## Ét kamera = ét bånd

Et **M3M**-kamera er den monokrome udgave af Bayer**M3C**: en monokrom IMX265-sensor bag et enkelt smalbåndsinterferensfilter. Modelstrengen angiver båndet — `M3M-<lens>-F<wavelength>`, f.eks. `M3M-L87-F685` (vises i Chloros som `LATT-M3M-L87-F685`). Sensoren leverer et**enkelt gråtonebånd** uden Bayer-mosaik: der er intet at demosaikere, ingen krydstale mellem kanalerne at adskille og ingen hvidbalance at indstille.

Konsekvenser, der er værd at kende, før du planlægger et monosystem:

* **Strålingsintensitet og reflektans er fuldt defineret pr. bånd.**Det er radiometriske kort pr. bånd, så et M3M-kamera producerer kalibreret float32-strålingsintensitet (W/m²/sr/nm) og uint16-reflektans (`32768` = ρ 1,0) præcis som et M3C-bånd gør. Mono-billeder indeholder en**identitets**-sensorresponsmatrix — der er ikke behov for eller anvendelse af 3×3-unmix.
* **Et enkelt monokamera kan ikke generere et vegetationsindeks.** NDVI, NDRE og lignende kræver mindst to bånd. For at beregne indekser fra monohardware kombinerer man flere M3M-kameraer — se nedenfor.
* M3M-kameraer streamer **Mono12** (12-bit, 2 bytes/pixel over kablet), hvilket er vigtigt for [budgettering af array-båndbredde](arrays.md#bandwidth-the-rules-of-thumb).

## Hvad Chloros springer over for mono — og hvordan det fortæller dig det

Farvepipeline-trin gælder ganske enkelt ikke for en enkeltbåndssensor. Chloros **springer dem over med en enkelt linjes besked** i stedet for at give en fejlmeddelelse, og kører dem stadig normalt for ethvert M3C (Bayer)-kamera i samme session:

| Trin | Mono (M3M)-adfærd | M3C-adfærd |
| --- | --- | --- |
| Demosaic / debayer | Springes over — eksportniveauet for `debayered` er et 1-kanals gråtonebillede. | 3-kanals demosaic. |
| Hvidbalance (`lattice white-balance`) | Springes over med en enkelt linje tekst. | Kører normalt. |
| Farveprofil (`lattice color-profile`) | Springes over med en enkelt linje tekst. | Kører normalt. |
| Mætning/kontrast (`lattice color`) | Springes over med en en-linjers meddelelse. | Kører normalt. |
| Spektral crosstalk-separering | Identitet (ingen 3×3-matrix). | 3×3-matrix pr. kamera anvendt. |
| Strålingsintensitet / reflektans | **Kører** — pr. bånd, fuldt kalibreret. | Kører pr. bånd. |

GUI&#x27;en anvender den samme gating: for et monokamera skjuler indstillingspanelet pr. kamera de rækker, der kun gælder for RGB (hvidbalance, gamma, farveprofil, mætning, kontrast, kanalopdeling), og live-histogrammet er låst til en enkelt **MONO**-kurve. Diskriminatoren i hele stakken er `M3M`-tokenet i modelstrengen, som vises i GUI’en som SDK.

## Indekser kræver ≥ 2 bånd: justering → stakning → indeksering

Arbejdsgangen for mono-indeksering består altid af de samme tre trin:

1. **Justering** — ret flere M3M-kameraer mod forskellige bølgelængder (f.eks. en F650 &quot;Red&quot; og en F850 &quot;NIR&quot;), forbind dem som et [multikamera-array](arrays.md), og lad Chloros beregne co-registreringsforvrængningen mellem kameraerne.
2. **Stak** — de justerede billeder bliver til ét multibåndsbillede (hvert kamera bidrager med ét navngivet bånd).
3. **Indeks** — udregn en indeksformel over stakkens bånd, eventuelt ved at gengive den gennem en LUT.

I brugergrænsefladen udgør hele denne kæde visningsfunktionen **Kombinerede kameraer**: det sammensatte livebillede er allerede justeret, og arrayets indeksberegner (nedenfor) definerer den formel, det gengiver. Optagede eksporter kan forvrænges til samme justering med optagelsesindstillingen**Justeret**.

## Indeksberegneren

Indeksberegneren opretter det indeksudtryk, der bruges af livevisningen og indekseksportene pr. kamera. Det er én fælles overflade, der åbnes fra to steder i sidepanelet på fanen *Kameraer*:

* **Per kamera**— Live Preview →**Index**-tandhjul (kun RGN/OCN/NGB Bayer-kameraer; et enkelt monokamera har ingen indekskontrol, da et enkelt bånd ikke kan danne et indeks).
* **Per-array**— array-indstillinger → Live Preview →**Index**-tandhjul. Dette er mono-stien: båndlisten omfatter**alle kameraer i arrayet**, så et monopar bidrager her med sine to bånd.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Dens kontrolelementer, fra top til bund:

* **Båndchips** (&quot;Bånd — klik for at tilføje til udtrykket&quot;) — én knap pr. tilgængeligt bånd, mærket med farvenavn + bølgelængde i nm (duplikerede farvenavne skelnes, f.eks. &quot;Farve 850&quot;). Ved at klikke indsættes båndtokenet ved markøren. Bånd fra kameraer, der ikke kan producere stråling pr. bånd (RGB/FRGB), filtreres fra.
* **Operator- og funktionsknapper** — `+ - * / ( ) ^ ,` samt `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Tekstfelt til udtryk** — frit indtastet formel; pladsholderen viser den klassiske NDVI-form `(NIR - Red) / (NIR + Red)`. En skrivebeskyttet, tokeniseret forhåndsvisning ovenover gengiver båndchips, tal og flag som ukendte tokens.
* **Gyldighedsbanner**— gråt »Tom — der anvendes ingen indeksering«; grønt »Gyldigt udtryk«; rødt med den specifikke parsefejl (ukendt bånd, tvetydigt bånd registreret af flere kameraer, manglende parentes, …); eller gult, når udtrykket er gyldigt, men**konstant** (f.eks. `X/X`, eller en NDVI-nævner indtastet med `−` i stedet for `+`) — en konstant kortlægger hele billedet til én farve.
* Der vises en separat gul advarsel, hvis det anvendte udtryk er korrekt, men **live-billedet er ensartet** (flad eller mættet scene) — histogramkollapset opdages automatisk for dig.
* **Anvend LUT**(standard: til; fra = gråskalaudvidelse),**Niveau**2/3/5/7-stop (standard 7-stop) og**Min / Max**-indstillingerne, der flankerer gradientbjælken. Min er som standard indstillet til**0,2**— det zoomer farveforløbet ind på det vegetationsrelevante område, mens værdier under dette vises som gråtoner; indstil Min til −1 for det fulde indeksområde (knappen**Nulstil** gendanner −1…+1). Max er som standard indstillet til 1.
* **Live-histogram** over indeksfordelingen — kvadratrodsskalerede søjler, gule p2/p98-percentillinjer, en hvid medianlinje og visninger af værdier uden for området („◀ N% &lt; lo“ / „hi &lt; N% ▶&quot;), der skifter til ravfarve over 1 % som tegn på, at Min/Max-vinduet skal udvides.
* **Anvend**anvender udtrykket på livestrømmen; LUT-justeringer træder i kraft live uden at trykke på Anvend. Udtryk er bevidst**kun gældende for den aktuelle session** — de gemmes ikke mellem sessioner.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## CLI-stien

Den samme kæde af justering → stak → indeks, der kan scriptes fra start til slut:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` knytter en forudindstillings symboler til stakkens båndnavne. To regler sparer dig for en mislykket kørsel:

* **Symboler er store- og småbogstavsfølsomme** og skal stemme nøjagtigt overens med forudindstillingens kanalnavne — forudindstillinger bruger små bogstaver (NDVI er `red`,`nir`; tjek `--list-presets`). `--channel red=Red_660` fungerer; `--channel RED=660` mislykkes med en `channel_map missing entries`-fejl.
* Båndsiden skal angive et bånd i den justerede stak (`lattice align-info --profile align.json` viser en liste over dem). Offline-tilstand accepterer også 0-baserede båndindekser, f.eks. `--channel red=0 --channel nir=1`.

`lattice index` kører også fuldstændigt offline mod et gemt, justeret multibånd-TIFF:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Indeksforudindstillinger

`lattice index --preset` (og fanen &quot;Billede&quot;s [Indeks/LUT-sandkasse](../image-viewer-gui/index-lut-sandbox.md), som bruger den samme motor) leveres med disse **22 forudindstillinger**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Kør `chloros-cli lattice index --list-presets` for hver forudindstillings formel og kanalsymboler, og `--list-gradients` for de tilgængelige farvegradienter. Brugerdefinerede formler bruger `--formula EXPR` med samme syntaks som Indeksberegneren. Bemærk, at denne liste over forudindstillinger er specifik for LATTICE-indeksmotoren — rullemenuen for databehandling under »Project Settings« for importerede billeder indeholder en anden liste (se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md)).

Det komplette sæt af flag (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, justeringsknapper til `--live`, og mere) er dokumenteret i [CLI-referencen § Indeks / Vegetationsmatematik](../reference/cli-reference.md#index--vegetation-maths); SDK-ækvivalenter findes i [SDK-referencen](../reference/sdk-reference.md).

## Indlæsning af indeksprodukter fra en mono-array

Når en array er tilsluttet og et indeksudtryk er anvendt, gemmer `array-capture` (eller GUI-funktionen **Capture All**) eksportniveauerne pr. kamera *og* indeksrenderingen — `--index`/`--no-index` aktiverer/deaktiverer dette på CLI, og indfangningen inkluderer som standard alle relevante niveauer. Et monokameras bidrag til hver indfangningsgruppe er dets ene bånd på rå-/debayered- (gråtoner)/strålings-/refleksionsniveauer samt den delte sammensatte indeks-komposit, når arrayet kører i kombineret tilstand. Se [Multikamera-arrayer § Optagelse](arrays.md#capturing-monitoring-vs-analysis).

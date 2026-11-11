# Monokameraer og vegetationsindekser

## Ét kamera = ét bånd

Et **M3M**-kamera er den monokrome udgave af Bayer**M3C**: en monokrom IMX265-sensor bag et enkelt smalbåndsinterferensfilter. Modelstrengen angiver båndet — `M3M-<lens>-F<wavelength>`, f.eks. `M3M-L87-F685` (vises i Chloros som `LATT-M3M-L87-F685`). Sensoren leverer et **enkelt gråtonebånd** uden Bayer-mosaik: der er intet at demosaikere, ingen krydstale mellem kanalerne at adskille og ingen hvidbalance at indstille.

Konsekvenser, der er værd at kende, før du planlægger et monosystem:

* **Stråling og reflektans er fuldt defineret pr. bånd.**Det er radiometriske kort pr. bånd, så et M3M-kamera producerer kalibreret float32-stråling (W/m²/sr/nm) og uint16-refleksion (`32768` = ρ 1,0) præcis som et M3C-bånd gør. Mono-billeder indeholder en**identitets**-sensorresponsmatrix — der er ikke behov for eller anvendelse af 3×3-unmix.
* **Et enkelt monokamera kan ikke generere et vegetationsindeks.** NDVI, NDRE og lignende kræver mindst to bånd. For at beregne indekser fra monohardware kombinerer man flere M3M-kameraer — se nedenfor.
* M3M-kameraer streamer **Mono12** (12-bit: 2 byte/pixel i datastrømmen, eller 1,5 byte/pixel, når et array bruger [pakket 12-bit-transport](arrays.md#packed-12-bit-transport)), hvilket har betydning for [båndbreddeplanlægning for arrayet](arrays.md#bandwidth-how-many-cameras-fit).

## Hvad Chloros

springer over ved mono — og hvordan det informerer dig om det

Faser i farvepipeline gælder simpelthen ikke for en enkeltbåndssensor. Chloros **springer dem over med en enkelt linjes besked** i stedet for at give en fejlmeddelelse, og kører dem stadig normalt for ethvert M3C (Bayer)-kamera i samme session:

| Trin | Mono (M3M)-adfærd | M3C-adfærd |
| --- | --- | --- |
| Demosaic / debayer | Springes over — eksportniveauet `debayered` er et 1-kanals gråtonebillede. | 3-kanals demosaic. |
| Hvidbalance (`lattice white-balance`) | Springes over med en enkelt linje tekst. | Kører normalt. |
| Farveprofil (`lattice color-profile`) | Springes over med en enkelt linje tekst. | Kører normalt. |
| Mætning/kontrast (`lattice color`) | Springes over med en enkelt linje besked. | Kører normalt. |
| Spektral crosstalk-separation | Identitet (ingen 3×3-matrix). | 3×3-matrix pr. kamera anvendt. |
| Radiance / reflektans | **Kører** — pr. bånd, fuldt kalibreret. | Kører pr. bånd. |

GUI'en anvender den samme gating: for et monokamera skjuler indstillingspanelet pr. kamera de rækker, der kun vedrører RGB (hvidbalance, gamma, farveprofil, mætning, kontrast, kanalopdeling), og live-histogrammet er låst til en enkelt **MONO**-kurve. Diskriminatoren i hele stakken er tokenet `M3M` i modelstrengen, der vises i GUI’en/SDK’et som `is_mono`.

## Indekser kræver ≥ 2 bånd: juster → stak → indeks

Arbejdsgangen for mono-indekser består altid af de samme tre trin:

1. **Justering** — ret flere M3M-kameraer mod forskellige bølgelængder (f.eks. et F650 »Red « og et F850 »NIR «), forbind dem som et [multikamera-array](arrays.md), og lad Chloros beregne co-registreringsforvrængningen mellem kameraerne.
2. **Stak** — de justerede billeder bliver til ét multibåndsbillede (hvert kamera bidrager med et navngivet bånd).
3. **Indeks** — beregne en indeksformel over stakens bånd og eventuelt gengive den via en LUT.

I brugergrænsefladen udgør hele denne kæde visningsfunktionen **Kombinerede kameraer**: den sammensatte live-visning er allerede justeret, og arrayets indeksberegner (nedenfor) definerer den formel, den gengiver. Optagede eksporter kan forvrænges til den samme justering med optagelsesindstillingen**Aligned**.

## Indeksberegneren

Indeksberegneren opretter det indeksudtryk, der bruges af livevisningen og indekseksportene pr. kamera. Det er én fælles overflade, der åbnes fra to steder i sidepanelet under fanen *Cameras*:

* **Per kamera**— Live-forhåndsvisning →**Indeks**-tandhjul (kun RGN-/OCN-/NGB-Bayer-kameraer; et enkelt monokamera har ingen indekskontrol, da et enkelt bånd ikke kan danne et indeks).
* **Per-array**— array-indstillinger → Live Preview →**Index**-tandhjul. Dette er mono-stien: båndlisten omfatter**alle medlemskameraer**, så et monopar bidrager her med sine to bånd.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Dens kontrolelementer, fra top til bund:

* **Båndchips** ("Bånd — klik for at tilføje til udtryk") — én knap pr. tilgængeligt bånd, mærket med farvenavn + bølgelængde i nm (duplikerede farvenavne adskilles, f.eks. "Farve 850"). Et klik indsætter båndtokenet ved markøren. Bånd fra kameraer, der ikke kan producere strålingsintensitet pr. bånd (RGB/FRGB), filtreres fra.
* **Operator- og funktionsknapper** — `+ - * / ( ) ^ ,` plus `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Tekstfelt til udtryk** — frit indtastet formel; pladsholderen viser den klassiske NDVI-form `(NIR - Red) / (NIR + Red)`. Et skrivebeskyttet, tokeniseret eksempel ovenover gengiver båndchips, tal og markerer ukendte token.
* **Gyldighedsbanner**— gråt »Tom — der anvendes ingen indeks«; grønt »Gyldigt udtryk«; rødt med den specifikke parsefejl (ukendt bånd, tvetydigt bånd fra flere kameraer, manglende parentes, …); eller gult, når udtrykket er gyldigt, men**konstanten** (f.eks. `X/X` eller en NDVI-nævner indtastet som `−` i stedet for `+`) — en konstant kortlægger hele billedet til én farve.
* Der vises en separat gul advarsel, hvis det anvendte udtryk er korrekt, men **live-rammen er ensartet** (flad eller mættet scene) — histogramkollapset registreres automatisk for dig.
* **Anvend LUT**(standard: aktiveret; deaktiveret = gråskalaudvidelse),**Niveau**2/3/5/7-stop (standard 7-stop) og**Min / Max**-indstillingerne, der flankerer gradientbjælken. Min er som standard indstillet til**0,2**— det zoomer farveforløbet ind på det vegetationsrelevante område, mens værdier under dette vises som gråtoner; indstil Min til −1 for det fulde indeksområde (knappen**Nulstil** gendanner −1…+1). Max er som standard indstillet til 1.
* **Live-histogram** over indeksfordelingen — kvadratrodsskalerede søjler, ravfarvede p2/p98-percentillinjer, en hvid medianlinje og visninger af værdier uden for området ("◀ N% < lo" / "hi < N% ▶"), der skifter til ravfarve over 1 % som tegn på, at Min/Max-vinduet skal udvides.
* **Anvend** overfører udtrykket til live-strømmen; LUT-justeringer anvendes live uden at trykke på Anvend. Når et projekt er åbent, gemmes udtrykket og LUT-indstillingerne sammen med projektet og gendannes, når du åbner det igen; uden et projekt varer de kun i den aktuelle session.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->



## CLI-stien

Den samme kæde af align → stack → index, der kan scriptes fra start til slut:

```bash
chloros-cli lattice align-calibrate --serials SN_RED,SN_NIR --save align.json
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`align-calibrate` bruger et tilkoblet array, der indeholder dets kameraer, eller åbner dem selv til den enkelte kørsel. `index --live` åbner altid selv kameraerne (en [midlertidig matrix](../reference/cli-reference.md#multi-camera-transient-sessions)) og har ingen `--serials`: den åbner alle kameraer på netværket. Så frigiv alle kameraer, som Chloros holder, før det — GUI'en, `cam-connect`, `array-connect` (`chloros-cli lattice array-disconnect --all`): hvis et kamera, der allerede er holdt, åbnes en anden gang, kan det stoppe dets indehavers stream eller få backend til at gå ned. Et kamera, der ikke leverer nogen frame, får kørslen til at mislykkes, og der skrives hverken en stack eller et indeks.

`--channel` knytter en forudindstillings symboler til stackens båndnavne. To regler kan redde dig fra en mislykket kørsel:

* **Symboler er store- og småbogstavsfølsomme** og skal stemme nøjagtigt overens med forudindstillingens kanalnavne — forudindstillinger bruger små bogstaver (NDVI’erne er `red`,`nir`; se `--list-presets`). `--channel red=Red_660` fungerer; `--channel RED=660` mislykkes med en `channel_map missing entries`-fejl.
* Båndsiden skal angive navnet på et bånd i den justerede stak (`lattice align-info --profile align.json` viser en liste over dem). Offline-tilstand accepterer også båndindekser, der starter fra 0, f.eks. `--channel red=0 --channel nir=1`.

`lattice index` kører også offline mod en gemt, justeret multibånds-TIFF-fil. Hver formular kører på backend, så den skal være i gang, og du skal være logget ind på et betalt Chloros+-abonnement; en relativ `--input` eller `--output` henviser til den mappe, du kører kommandoen fra:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Indeksforudindstillinger

`lattice index --preset` (og fanen "Billede"s [Indeks/LUT-sandkasse](../image-viewer-gui/index-lut-sandbox.md), som bruger den samme motor) leveres med disse **22 forudindstillinger**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Kør `chloros-cli lattice index --list-presets` for hver forudindstillings formel og kanalsymboler, og `--list-gradients` for de tilgængelige farvegradienter. Brugerdefinerede formler bruger `--formula EXPR` med samme syntaks som indeksberegneren. Bemærk, at denne liste over forudindstillinger er specifik for LATTICE-indeksmotoren — rullemenuen »Behandling« under »Projektindstillinger« for importerede billeder indeholder en anden liste (se [Multispektrale indeksformler](../project-settings/multispectral-index-formulas.md)).

Det komplette sæt af flag (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, justeringsknapperne til `--live` og mere) er dokumenteret i [CLI-reference § Indeks / Vegetationsmatematik](../reference/cli-reference.md#index--vegetation-maths); SDK-ækvivalenterne findes i [SDK-referencen](../reference/sdk-reference.md).

## Indfangning af indeksprodukter fra et mono-array

Når et array er tilsluttet og et indeksudtryk er anvendt, gemmer `array-capture` (eller GUI-funktionen **Capture All**) gemmer eksportniveauerne pr. kamera *og* indeksrenderingen — `--index`/`--no-index` skifter mellem dem i CLI, og standardindstillingen er at inkludere alle relevante niveauer. Et monokameras bidrag til hver optagelsesgruppe er dets ene bånd på rå-/debayered (gråtoner)/radiance-/reflektansniveauer samt den delte sammensatte indekskomposit, når arrayet kører i kombineret tilstand. Se [Multikamera-arrayer § Optagelse](arrays.md#capturing-monitoring-vs-analysis).

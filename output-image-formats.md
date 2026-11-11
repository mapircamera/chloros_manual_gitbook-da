---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Outputbilledformater

Chloros eksporterer behandlede produkter i fire filformater. Vælg formatet i Projektindstillinger (GUI), med `--format` (CLI) eller med `export_format` (SDK). CLI og SDK accepterer nøjagtigt nedenstående strenge.

| Formatstreng | Filtypenavn | Pixeltype | Pixelinterval | Bemærkninger |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(standard)* | `.tif` | uint16 digitalt tal | 0 – 65535 | Anbefales til fotogrammetri / GIS. |
| `TIFF (8-bit)` | `.tif` | uint8 digitalt tal | 0 – 255 | Til optagelser, der udelukkende er i 8-bit, og som kun tilbyder 8-bit-formater: Chloros opskalerer ikke bitdybden. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % reflektans. Nogle programmer kan ikke læse TIFF-filer med flydende komma; filerne er større. |
| `PNG (8-bit)` | `.png` | uint8 digitalt tal | 0 – 255 | Tabsfri komprimering, velegnet til visning på internettet og visualisering. |
| `JPG (8-bit)` | `.jpg` | uint8 digitalt tal | 0 – 255 | Komprimering med tab, mindste filstørrelse. |

## Hvor outputfilerne gemmes

Produkterne gemmes i projektmappen, grupperet efter kamera og derefter efter filformat:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Kameramappen er `LATT-<sensor>-<lens>-F<filter>` for LATTICE og `<model>_<filter>` (f.eks. `Survey3N_RGN`) for Survey3 **Hvert eksporteret produkt beholder kildefilens navn — det er mappen, der identificerer produktet, ikke et filnavnesuffiks.** Se [Hvor outputfilerne gemmes](reference/cli-reference.md) i CLI-referencen for de fulde regler.

## LATTICE-produkter (optagelses- og eksportniveauer)

En rå LATTICE-ramme uddeles til alle de ønskede produkter i én enkelt gennemløb. Hver produkttype har sin egen aktiveringsknap (GUI-afkrydsningsfelter eller CLI-kommandoerne `--debayered` / `--preview` / `--radiance` / `--reflectance`, alle er som standard slået til):

| Niveau | Indhold | Datatype |
| --- | --- | --- |
| `raw` | Bayer-data direkte fra sensoren (monokameraer: det enkelte bånd). Behandlingen starter altid fra rådata. | Som optaget |
| `debayered` | Lineær demosaik — 3-kanals for M3C, 1-kanals gråtoner for M3M: kalibreringspakkens mørkeforskydning fjernes, efterfulgt af en kantbevidst demosaik uden linearitetskorrektion, flat field eller unmix. En `process`-eksport skriver den samme fil, som en optagelse gemmer på dette niveau, i alle behandlingsmodi. | Lineær DN |
| `radiance` | Absolut spektral strålingsintensitet fra den fulde radiometriske kæde, i **W/m²/sr/nm**. Skrives altid som 32-bit TIFF (`tiff32/Radiance_Images/`), uanset det valgte eksportformat. | float32 |
| `reflectance` | Reflektans ρ, hvor **DN 32768 = ρ 1,0 (100 %)** med spillerum op til ρ 2,0. Pix4D-klar. | uint16 |
| `preview` | Visningsklar rendering: RGB-farver = hvidbalance + gamma; multispektral = falskfarvestrækning. | 8-bit skærm |

### Orientering

`raw`- og `debayered`-filer bevarer altid sensorens **oprindelige** orientering; et kameras indstilling for spejling/rotation anvendes kun på de øvrige produkter (forhåndsvisning, strålingsintensitet, reflektans, indeks). Array-optagelser bevarer hvert medlems filer i deres oprindelige retning, medmindre du anmoder om retningsændring (`--orient` i CLI, `orient=True` i SDK). Hver fil registrerer den orientering, der hører til dens pixels, i XMP — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — og `ChlorosOrientationApplied` angiver, om dens egne pixels bærer orienteringen (`false` på rå og debayered filer). Chloros-billedviseren konverterer RAW- og debayered-filer i henhold til denne post udelukkende med henblik på visning; selve filerne overskrives aldrig. En fil uden en sådan post (skrevet før version 1.3.0 eller med et andet værktøj) vises, som den er gemt.

## Clamp-masker

Radiance- og reflektansprodukterne fra et multispektralt M3C-kamera gennemgår en 3×3-bånds-unmix. Hvor unmixen giver en negativ værdi i et bånd, beskæres denne værdi til nul (live-visninger beskæres på nøjagtig samme måde). Nyt i 1.3.0: Et produkt, hvor mindst én pixel er blevet beskåret, får en tilhørende **clamp-maske**, der angiver præcis, hvilke pixels og bånd der er tale om:

* **Hvor:**i en `clamp_masks/`-mappe**ved siden af** produktmapperne, aldrig inde i en: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif` og `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Hvad:** en enkeltbånds `uint8`-TIFF-fil på produktets eget pixelgitter (inklusive orientering, ROI, binning og array-justering). Bit 0 (`mask & 1`) er sat, hvor båndet i produktets R-kanal blev beskåret, bit 1 (`& 2`) G-kanalen, bit 2 (`& 4`) B-kanalen. Andelen af klippede pixels pr. bånd er også angivet på produktet som `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` og `Chloros:BandBUnmixClampFraction`.
* **Sådan angives det i produktet:** XMP-tagget `Chloros:UnmixClampMask` indeholder maskenes sti i forhold til produktets mappe (`../clamp_masks/<product folder>/<name>_clampmask.tif`, altid adskilt med `/`), `none`, når ingen pixel blev beskåret (der skrives ingen maske), eller `unavailable`, når en maske ikke kunne skrives. Et produkt uden tag har aldrig gennemgået unmix-processen (kameraer i RGB- og monokrom-format samt rå- og debayered-niveauer).

Da ingen produktmappe indeholder en maske, kan du overføre en produktmappe som f.eks. `Reflectance_Calibrated_Images/` til Pix4D, Metashape eller anden fotogrammetri-software, som den er, inklusive undermapper — overfør produktmappen, ikke formatmappen ovenover den. Chloros&#x27; egen import springer clamp-masker over. CLI-kommandoen tæller dem separat: `Image products written: N (plus M clamp masks)`.

## Indeksprodukter

Hvert indeks havner i sin egen `<INDEX>_Index_Images/`-mappe (f.eks. `NDVI_Index_Images/`).

* **Heltalsformater**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) gemmer en**lineær strækning pr. billede**, ikke en −1…1-afbildning: DN 0 er den pågældende rammes laveste indeksværdi, og fuld skala er den højeste. Filen indeholder `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` og `Chloros:IndexMax`, altså også `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Da hvert billede har sine egne endepunkter, skal billederne sammenlignes som værdier, ikke som DN.
* **`TIFF (32-bit, Percent)`** gemmer selve indeksværdierne som float32 uden omskalering — det bedste valg til kvantitativt arbejde på tværs af en flyvning.

Nyt i 1.3.0: Et indeksprodukt beskriver **sig selv** i stedet for at gentage kildedataets tags. Dets GPS- og kameratags kopieres fra det produkt, det er beregnet ud fra (reflektansproduktet, hvis der er et), og derefter:

| Tag | Værdi |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (så filen aldrig importeres igen som en optagelse) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / den anvendte formel |
| `Chloros:IndexLevel` | `reflectance` eller `radiance` — hvad indekset blev beregnet ud fra (mangler, hvis ingen af delene er til stede) |
| `Chloros:IlluminationInvariant` | `True` for et indeks beregnet ud fra reflektans, `False` ud fra strålingsintensitet eller sensorværdier |
| `Chloros:ChlorosPipelineVersion` | den Chloros-version, der skrev indekset |

`Chloros:PixelScale` er fjernet fra indeksprodukter — brug `IndexMin` / `IndexMax` ovenfor. Den fulde vejledning til genindlæsning findes under &quot;Et heltal-indeksraster er en kontrastudvidelse pr. billede&quot; i [CLI-referencen](reference/cli-reference.md).

## Aflæsning af reflektans-pixelværdier

Reflektansen gemmes som et heltal, og **det DN, der svarer til ρ = 1,0 (100 % reflektans), afhænger af kildekameraet**:

| Kildekamera | ρ = 1,0 er DN | Sådan afgøres det |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margin op til ρ 2,0) | XMP-tagget `Chloros:PixelScale=32768` er indstemplet i filen. |
| Survey3 | `65535` (beskåret ved ρ 1,0) | Intet `Chloros:PixelScale`-mærke — denne fravær er tegn på det. (Survey3-produkter indeholder andre `Chloros:*`-behandlingsmærker, såsom `Chloros:CalibrationApplied`, i alle behandlingsmodi fra og med version 1.3.0.) |

**Læs `Chloros:PixelScale` XMP-tagget og divider med det** i stedet for at antage en konstant. Tagget er defineret i uint16-domænet, så det forbliver `32768` på tværs af outputformater, der skaleres om — normaliser først den gemte datatype tilbage til uint16 (×257 fra 8-bit, ×65535 fra float32).

{% hint style="warning" %}
**8-bit-optagelser.** Eksportdata fra en 8-bit-optagelse er 8-bit. Dens reflektans skaleres om og mærkes `Chloros:PixelScale=32768`, så skalaen bevares; 8-bit DN-produkter og reflektansfiler, der er skrevet før version 1.3.0, beskæres til 0–255 og har ingen skalamærkning. Hvis mærkningen mangler på en LATTICE-reflektansfil, skal du ikke antage, at der er en skala; eksporter i stedet igen i 16-bit eller 32-bit.
{% endhint %}

Se **&quot;Læsning af reflektanspixels&quot;** i [CLI-referencen](reference/cli-reference.md) for de fuldstændige regler (herunder de MicaSense-kompatible mærker).

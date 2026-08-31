---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Outputbilledformater

Chloros eksporterer behandlede produkter i fire filformater. Vælg formatet i projektindstillingerne (GUI) med `--format` (CLI) eller med `export_format` (SDK). CLI og SDK accepterer nøjagtigt nedenstående strenge.

| Formatstreng | Udvidelse | Pixeltype | Pixelinterval | Bemærkninger |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(standard)* | `.tif` | uint16 digitalt tal | 0 – 65535 | Anbefales til fotogrammetri / GIS. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100 % reflektans. Nogle programmer kan ikke læse TIFF-filer med flydende komma; filerne er større. |
| `PNG (8-bit)` | `.png` | uint8 digitalt tal | 0 – 255 | Tabsfri komprimering, velegnet til visning på internettet og visualisering. |
| `JPG (8-bit)` | `.jpg` | uint8 digitalt tal | 0 – 255 | Komprimering med tab, mindste filstørrelse. |

## Hvor udfilserne gemmes

Filerne gemmes i projektmappen, grupperet efter kamera og derefter efter filformat:

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

Kameramappen er `LATT-<sensor>-<lens>-F<filter>` for LATTICE og `<model>_<filter>` (f.eks. `Survey3N_RGN`) for Survey3. **Hvert eksporteret produkt beholder kildefilens navn — det er mappen, der identificerer produktet, ikke et filnavnsuffiks.** Se [Hvor outputfilerne gemmes](reference/cli-reference.md) i CLI-referencen for de fulde regler.

## LATTICE-produkter (optagelses- og eksportniveauer)

Én LATTICE-råramme forgrener sig til alle anmodede produkter i én enkelt gennemkørsel. Hver produkttype har sin egen indstilling (GUI-afkrydsningsfelter eller CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, alle er som standard slået til):

| Niveau | Indhold | Datatype |
| --- | --- | --- |
| `raw` | Bayer-data direkte fra sensoren (monokameraer: det enkelte bånd). Behandlingen starter altid fra rådata. | Som optaget |
| `debayered` | Lineær demosaik — 3-kanals for M3C, 1-kanals gråtoner for M3M. | Lineær DN |
| `radiance` | Absolut spektral strålingsintensitet fra den fulde radiometriske kæde, i **W/m²/sr/nm**. Skrives altid som 32-bit TIFF (`tiff32/Radiance_Images/`), uanset det valgte eksportformat. | float32 |
| `reflectance` | Refleksionsgrad ρ, hvor **DN 32768 = ρ 1,0 (100 %)** med spillerum op til ρ 2,0. Pix4D-klar. | uint16 |
| `preview` | Visningsklar rendering: RGB = hvidbalance + gamma; multispektral = falskfarvestrækning. | 8-bit visning |

## Aflæsning af reflektanspixelværdier

Reflektansen gemmes som et heltal, og **det DN-tal, der svarer til ρ = 1,0 (100 % reflektans), afhænger af kildekameraet**:

| Kildekamera | ρ = 1,0 er DN | Sådan afgøres det |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margin op til ρ 2,0) | XMP-tagget `Chloros:PixelScale=32768` er indstemplet i filen. |
| Survey3 | `65535` (begrænset ved ρ 1,0) | Ingen `Chloros:*` XMP-tags — denne fravær er signalet. |

**Læs `Chloros:PixelScale` XMP-tagget og divider med det** i stedet for at antage en konstant. Tagget er defineret i uint16-domænet, så det forbliver `32768` på tværs af outputformater, der omskalerer — normaliser først den gemte datatype tilbage til uint16 (×257 fra 8-bit, ×65535 fra float32).

{% hint style="warning" %}
**Ét tilfælde har ingen skalering, hvilket er tilsigtet.** Når en 8-bit-kildeoptagelse (BayerRG8) skrives som 8-bit TIFF, klipper pipelinen til 0–255 i stedet for at omskalere, så der er ingen skalering, der beskriver filen — Chloros udelader bevidst `Chloros:PixelScale` her. Hvis tagget mangler i en LATTICE-reflektansfil, skal du ikke antage, at der er en skala; eksporter i stedet igen i 16-bit eller 32-bit.
{% endhint %}

For de fuldstændige regler (herunder de MicaSense-kompatible tags) se **&quot;Læsning af reflektanspixels&quot;** i [CLI-referencen](reference/cli-reference.md).

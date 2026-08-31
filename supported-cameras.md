---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Understøttede kameraer

Chloros behandler billeddata fra to MAPIR-kamerafamilier på **alle platforme** (Windows, Linux amd64 og Linux arm64/Jetson):

* **Survey3** — Survey3W (bred) og Survey3N (smal) kameraer. Indgang: `RAW+JPG`.
* **LATTICE**— M3C og M3M multispektrale kameramoduler. Indgang: `.tif`/`.tiff`-optagelser. LATTICE-kameraer kan også**styres live** fra Chloros — via fanen »Kameraer« i GUI’en (Windows) eller `chloros-cli lattice` / Python SDK (Windows og Linux) — herunder synkroniserede multikamera-opstillinger. Se [LATTICE-vejledningen](lattice/).

Behandlingspipeline accepterer også `.dng`-indlæsningsfiler.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Producent</th><th width="250">Kameramodel</th><th width="138">Filtermodel</th><th width="187">Billedtype</th></tr></thead><tbody><tr><td><strong>Producent</strong></td><td><strong>Kameramodel</strong></td><td><strong>Filtermodel</strong></td><td><strong>Billedtype</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

LATTICE-serien er et modulært multispektralt kamerasystem baseret på Sony IMX265 global-shutter-sensoren (3,1 MP, 3,45 µm pixels). Hvert kamera gemmer sin identitet som en modelstreng:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros viser den med præfikset `LATT-` (for eksempel `LATT-M3M-L41-F550`), og modelstrengen styrer alt nedstrøms — sensorprofil, båndlayout og kalibrering fastlægges automatisk; der er intet, der skal konfigureres for hvert enkelt kamera. Objektivnummeret angiver **det vandrette synsfelt i grader**: `L41` = smalt 41°, `L87` = bredt 87°.

Der findes to sensorkonfigurationer:

| Konfiguration | Sensor      | Filtertype                           | Bånd pr. kamera                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Bayer-farve | Trebåndspas                       | 3 spektralbånd fra en enkelt eksponering                                 |
| **M3M**       | Monokrom  | Enkelt smalbåndsinterferensfilter | 1 kalibreret bånd — kombiner flere M3M-kameraer til vegetationsindekser |

### M3C (Bayer)-filterindstillinger

| Filter | Bånd (navn @ centrum nm / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 615/21 · Cyan 490/38 · NIR 808/14 |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

### M3M (mono) filterkatalog — 23 varenumre

F-tallet er varenummeret; det målte bånd (stemplet på hver kalibreret eksport) er filterscanningen pr. parti:

| Varenummer    | Centrum (nm, målt) | FWHM kanter (nm) | Bredde (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nominel)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nominel)           | 867–893         | 26         |
| F905   | — (nominel)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_&quot;Båndkanterne er målt som værdier for fuld bredde ved halv maksimalintensitet fra MAPIR&#x27;s filterscanninger pr. parti — de samme værdier, som Chloros indstempler i hver kalibreret eksport.&quot;_ &quot;— (nominel)&quot; = endnu ingen lot-scanning; for disse SKU&#x27;er er det angivne centrum SKU-nummeret, og bredden er producentens tal.

† &quot;F988-reflektansen kalibreres ved hjælp af et reflektanspanel i scenen: båndet ligger uden for DAQ-lyssensorens kalibrerede område, så Chloros anvender din seneste paneloptagelse og opbevarer den mellem panelobservationerne.&quot; Se [Kalibreringsmål](calibration-targets.md).

For live kamerastyring, arrayer, netværksopsætning og den radiometriske behandlingskæde, se [LATTICE-vejledningen](lattice/).

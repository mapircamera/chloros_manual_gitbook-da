# LATTICE-kameraer

LATTICE er MAPIR’ modulopbyggede multispektrale kamerasystem til billedoptagelse inden for landbrug og videnskab. Hvert LATTICE-kamera er baseret på Sony IMX265 global-shutter-sensoren (**3,1 MP, 3,45 µm pixels**) og tilsluttes via Ethernet som en**GigE Vision**-enhed.

Chloros 1.2.0 styrer LATTICE-kameraer i realtid — opdagelse, live-forhåndsvisning, optagelse og synkroniserede multikamera-opstillinger — fra tre grænseflader:

| Grænseflade    | Hvor                                                          | Platforme                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | Fanen **Kameraer** i sidepanelet i Chloros                         | Windows 10/11 x64                                        |
| CLI        | `chloros-cli lattice`-kommandofamilie                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Leder du efter hardware?**Kameramoduler, objektiver, filtre og bånd, rammer og monteringsbeslag, kabler, PoE og triggerledninger er beskrevet i [**LATTICE-brugervejledningen**](https://mapir.gitbook.io/lattice-camera). Dette kapitel beskriver styring af kameraerne fra Chloros.

LATTICE-optagelser er standardfiler af typen `.tif`/`.tiff`, og Chloros behandler dem altid ud fra den rå optagelse. Se [CLI-referencen](../reference/cli-reference.md) og [SDK-referencen](../reference/sdk-reference.md) for den komplette kommando og API-overfladen.

## To sensor-konfigurationer

| Konfiguration | Sensor       | Filter                                | Hvad et kamera leverer                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayer-farve | tredobbelt båndpasfilter                |**Tre kalibrerede bånd fra en enkelt eksponering**                 |
| **M3M**| Monokrom   | enkelt smalbåndsinterferensfilter |**Ét kalibreret bånd**; kombiner flere M3M-kameraer for indekser |

Da et M3M-kamera er monokromt bag et enkelt filter, får hvert bånd sin egen eksponering. Et M3C-kamera dækker alle sine tre bånd med én sensoreksponering.

## Modelstrenge og navngivning

Hvert kamera gemmer sin identitet i GenICam `DeviceUserID` som en modelstreng:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros viser den med præfikset `LATT-` (for eksempel `LATT-M3M-L87-F450`). Den samme `LATT-…`-streng skrives ind i EXIF-tagget `Model` ved hver eksport og bruges som navnet på kameraets outputmappe i behandlede projekter.

| Komponent | Værdier                                                   | Betydning                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Sensor    | `M3C` / `M3M`                                            | Bayer-farve / monokrom                                                                          |
| Objektiv      | `L41` / `L87`                                            | Tallet angiver **det vandrette synsfelt i grader**: L41 = smalt (41°), L87 = bredt (87°)    |
| Filter    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) eller `F<nm>` (M3M) | Se [Filtre og spektralbånd](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Modelstrengen styrer alt det, der følger efter: Chloros bestemmer sensorprofilen, båndopstillingen og fabrikskalibreringen ud fra `DeviceUserID` + `DeviceSerialNumber`. Der er intet, der skal konfigureres pr. kamera — se [Tilslutning af kameraer](connecting.md).

## Filtre og bånd

Båndcentre, FWHM-kanter og det fulde M3M-katalog med 23 SKU&#x27;er er produktspecifikationer, så de findes i hardwaremanualen: [**Filtre og spektralbånd**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Hvad der er vigtigt på softwaresiden: filterkoden i modelstrengen bestemmer, hvilke produkter Chloros kan generere. RGB-filterkameraer (`FRGB`) udsender udelukkende debayered- og preview-produkter — strålingsintensitet og reflektans pr. bånd er ikke meningsfuldt for en bredbåndssensor, så Chloros springer dem over og angiver dette. Alle andre filtre leverer den fulde kæde af strålingsintensitet → reflektans → indeks.

## Radiometrisk kalibrering i korte træk

Hvert LATTICE-kamera kalibreres individuelt på fabrikken i forhold til en NIST-sporbare kæde og leveres med et certifikat for hvert enkelt kamera. Hvad dette omfatter, hvordan det måles, og hvilken nøjagtighed du kan angive, fremgår af hardwaremanualen: [**Fabriksradiometrisk kalibrering**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

På softwaresiden er det vigtigt, at Chloros fastlægger den korrekte kalibrering, når et kamera tilsluttes, og låser de anvendte koefficienter fast i hver eksport — se [Tilslutning af kameraer](connecting.md).

## I dette kapitel

* [Tilslutning af kameraer](connecting.md) — automatisk registrering, GUI-tilslutningsdialogen, CLI/SDK-ækvivalenter samt hvordan fabrikskalibrering håndteres (pakke på kameraet vs. cloud), når et kamera tilsluttes.

Yderligere LATTICE-emner — kameraindstillinger og live-styring, optagelsestilstande, multikamera-opstillinger samt mono (M3M)-behandling og indekser — behandles i deres egne afsnit i denne manual, og den komplette kommandoflade findes i [CLI-referencen](../reference/cli-reference.md) og [SDK-reference](../reference/sdk-reference.md).

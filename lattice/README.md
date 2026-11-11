# LATTICE-kameraer

LATTICE er MAPIRs modulære multispektrale kamerasystem til landbrugs- og videnskabelig billedbehandling. Alle LATTICE-kameraer er baseret på Sony IMX265 global-shutter-sensoren (**3,1 MP, 3,45 µm pixels**) og tilsluttes via Ethernet som en**GigE Vision**-enhed.

Chloros 1.3.0 styrer LATTICE-kameraer i realtid — opdagelse, live-forhåndsvisning, optagelse og synkroniserede multikamera-opstillinger — fra tre grænseflader:

| Grænseflade    | Hvor                                                          | Platforme                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | Fanen **Kameraer** i Chloros-sidepanelet                         | Windows 10/11 x64                                        |
| CLI        | `chloros-cli lattice`-kommandofamilien                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Leder du efter hardware?**Kameramoduler, objektiver, filtre og bånd, rammer og monteringsbeslag, kabler, PoE og triggerledninger er beskrevet i [**LATTICE-brugervejledningen**](https://mapir.gitbook.io/lattice-camera). Dette kapitel omhandler styring af kameraerne fra Chloros.

LATTICE-optagelser er standardfiler af typen `.tif`/`.tiff`, og Chloros behandler dem altid ud fra den rå optagelse. Se [CLI-referencen](../reference/cli-reference.md) og [SDK-referencen](../reference/sdk-reference.md) for den komplette kommando- og API-grænseflade.

## To sensorkonfigurationer

| Konfiguration | Sensor       | Filter                                | Hvad et kamera leverer                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayer-farve | tredobbelt båndpasfilter                |**Tre kalibrerede bånd fra en enkelt eksponering**                 |
| **M3M**| Monokrom   | enkelt smalbåndsinterferensfilter |**Ét kalibreret bånd**; kombiner flere M3M-kameraer for indekser |

Da et M3M-kamera er monokromt bag et enkelt filter, får hvert bånd sin egen eksponering. Et M3C-kamera dækker alle tre bånd med én sensoreksponering.

## Modelstrenge og navngivning

Hvert kamera gemmer sin identitet i GenICam `DeviceUserID` som en modelstreng:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros viser den med præfikset `LATT-` (for eksempel `LATT-M3M-L87-F450`). Den samme streng, `LATT-…`, skrives ind i EXIF-tagget `Model` ved hver eksport og bruges som navnet på kameraets outputmappe i behandlede projekter.

| Komponent | Værdier                                                   | Betydning                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Sensor    | `M3C` / `M3M`                                            | Bayer-farve / monokrom                                                                          |
| Objektiv      | `L41` / `L87`                                            | Tallet angiver **det vandrette synsfelt i grader**: L41 = smalt (41°), L87 = bredt (87°)    |
| Filter    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) eller `F<nm>` (M3M) | Se [Filtre og spektralbånd](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

Modelstrengen styrer alt det efterfølgende: Chloros udleder sensorprofilen, båndopstillingen og fabrikskalibreringen ud fra `DeviceUserID` + `DeviceSerialNumber`. Der er intet, der skal konfigureres pr. kamera — se [Tilslutning af kameraer](connecting.md).

## Filtre og bånd

Båndcentre, FWHM-kanter og det fulde M3M-katalog med 23 SKU'er er produktspecifikationer, så de findes i hardwaremanualen: [**Filtre og spektralbånd**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Hvad der er vigtigt på softwaresiden: filterkoden i modelstrengen bestemmer, hvilke produkter Chloros kan generere. RGB-filterkameraer (`FRGB`) leverer udelukkende debayered- og preview-produkter — strålingsintensitet og reflektans pr. bånd er ikke meningsfulde for en bredbåndssensor, så Chloros springer dem over og angiver dette. Alle andre filtre leverer den fulde kæde af strålingsintensitet → reflektans → indeks.

## Radiometrisk kalibrering i korte træk

Hvert LATTICE-kamera kalibreres individuelt på fabrikken i forhold til en NIST-sporbart kæde og leveres med et certifikat for hvert enkelt kamera. Hvad dette omfatter, hvordan det måles, og hvilken nøjagtighed du kan angive, fremgår af hardwaremanualen: [**Fabriksradiometrisk kalibrering**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

På softwaresiden er det afgørende, at Chloros finder den korrekte kalibrering, når et kamera tilsluttes, og fastlåser de anvendte koefficienter i hver eksport — se [Tilslutning af kameraer](connecting.md).

## I dette kapitel

* [Tilslutning af kameraer](connecting.md) — automatisk genkendelse, GUI-dialogboksen til tilslutning, CLI-/SDK-ækvivalenter og hvordan fabrikskalibreringen fastlægges (pakke på kameraet vs. cloud), når et kamera tilsluttes.
* [Kameraindstillinger](camera-settings.md) — alle kontrolelementer under fanen "Kameraer": kameralisten, indstillingspanelet for hvert enkelt kamera og panelet med array-indstillinger (herunder ROI-editoren og justering).
* [Optagelsesindstillinger og -tilstande](capture.md) — Optag alt, enkelt-/kontinuerlig-/intervaloptagelse, eksporttyper og mappenes struktur for optagelser.
* [Multikamera-arrayer](arrays.md) — guiden til tilslutning af arrayer, synkroniseringskabel-sonde, båndbredde, PTP og live-visning af arrayet.
* [Monokameraer og vegetationsindekser](mono-indices.md) — M3M-kameraer og arbejdsgangen justering → stabling → indeksering.

Den komplette kommandoflade findes i [CLI-referencen](../reference/cli-reference.md) og [SDK-referencen](../reference/sdk-reference.md).

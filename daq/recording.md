# Optagelse og .daq-formatet

En `.daq`-fil er Chloros’s optagelsesformat for lyssensorer: en **SQLite-database** med kalibrerede spektralbilleder fra én DAQ-sensor. Optag en sådan fil under en optagelsessession, og reflektans-pipeline kan senere dividere hvert billede med den nedadgående strålingsintensitet, der blev målt på netop det tidspunkt.

## Hvad en .daq-fil indeholder

| Egenskab | Værdi |
| --- | --- |
| Container | SQLite-database, én fil pr. sensor pr. optagelse |
| Filnavn | Indeholder **sensor-ID**og et**tidsstempel**, f.eks. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum pr. billede | 135 punkter, 340–1010 nm i trin på 5 nm, plus CIE XYZ-tristimulus |
| Enheder | Kalibreret spektral irradians, **W/m²/nm** (fabriks-kalibreringspakke + cap-profil anvendt) |
| Indlejrede metadata | Sensor-ID (nøglen til at hente den pågældende enheds fabrikskalibrering) og den gældende cap-profil — se [Cap-profiler og kalibreret område](caps-and-range.md) |

Formatet er identisk på tværs af DAQ-U, DAQ-M og DAQ-E, så den efterfølgende behandling er ligeglad med, hvilken transport der har registreret det.

Kalibreret registrering kræver sensorens fabrikskalibreringspakke. For DAQ-U og DAQ-M henter backend-systemet pakken fra MAPIR&#x27;s cloud ved hjælp af sensor-ID&#x27;et (optagelse afvises, hvis dette ikke er muligt); DAQ-E-enheder er undtaget, da de opbevarer deres kalibrering på selve enheden.

## Optagelse fra GUI&#x27;en

Optagelse i GUI&#x27;en kræver et **åbent projekt** (ellers er optagelsesknapperne deaktiveret):

* **Optag alt / Stop alt** — øverst i sidepanelet for lyssensorer; starter eller stopper en `.daq`-optagelse på alle tilsluttede sensorer på én gang.
* **Optag / Stop optagelse** — pr. sensor, i indstillingsvinduet med tandhjulsikonet. En rød &quot;REC&quot;-indikator vises i sensorens live-informationsrækker under optagelse.

Filerne gemmes i `<project>/light_sensor/`, og når en optagelse stopper — uanset om det sker via Stop, Stop alt eller ved at afbryde forbindelsen til en optagelsessensor — føjes den færdige `.daq` **automatisk til det åbne projekt**. Den vises i projektets filliste uden behov for manuel tilføjelse og er allerede klar til reflektansbehandling.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Optagelse fra CLI

CLI optager via backendens sensorpool (backenden skal køre — disse kommandoer er tynde HTTP-klienter):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Hent værdien `--sensor-id` fra `chloros-cli daq pool-list`. To standardindstillinger, der er værd at kende:

| Indstilling | Standard |
| --- | --- |
| `--duration` | `0` — optag indtil `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` på **backendens** filsystem, ikke CLI&#x27;s |

Forskellen mellem output-mapperne er vigtig, når CLI peger på en backend på en anden maskine: filen placeres der, hvor backenden kører.

## Optagelse fra Python

`DAQSensorSession` (returneret af `chloros_sdk.connect_daq_sensor()`) viser den samme samlede optagelse: `record_start(output_dir=None, device_name=None)` returnerer filstien, mens `record_stop()` returnerer `{path, rows}`. Se [SDK-referencen](../reference/sdk-reference.md) for den fulde session API. SDK’s direkte hardwareklasser (kun desktop-installationer) skriver optagelser til `~/Documents/DAQ/` som standard; for udgivne versioner er den samlede sti ovenfor den understøttede rute.

## Brug af en .daq-fil ved databehandlingen

For at generere reflektans ud fra billeddata har Chloros brug for nedadgående strålingsintensitet, der er tilpasset hver eksponering:

* **Opbevar `.daq` sammen med billeddataene.**Ved databehandlingen udleder pipelinen automatisk den**tidsstempelmatchede nedadgående stråling** fra en optaget `.daq` (enhver DAQ-model) — eller fra en DAQ-M-native `.csv` — der findes sammen med billederne. GUI-optagelser opfylder dette automatisk, da de føjes til projektet i det øjeblik, de stopper.
* **Kalibrering hentes efter behov.** Hvis en fabrikskalibreringspakke pr. kamera eller pr. DAQ ikke allerede er gemt lokalt i cachen, henter Chloros den automatisk fra MAPIR’s cloud ved første brug (internetforbindelse kræves én gang; gemmes i cachen under `~/.chloros/`).
* **Live-optagelser skriver deres egen sidecar.** For ethvert reflektansbillede, der er optaget live, gemmes den faktisk anvendte DAQ-måling som en `.daq`-sidecar ved siden af billedet, så optagelsen kan genbehandles senere uden den oprindelige optagelse.

## Hvordan man henter strålingsintensiteten ud igen

Når et projekt behandles, eksporteres alle lyssensoroptagelser, det indeholder, til en
`Light Sensor/`-mappe ved siden af billedprodukterne. Dette kræver **ikke** billeddata: en
lyssensor, der er fløjet alene, udgør en komplet optagelse, og en mappe, der kun indeholder `.daq`
-filer, er en gyldig indgang. Kørslen rapporterer, hvor mange lyssensorprodukter den har skrevet.

| Produkt | Hvad er det? |
| --- | --- |
| `<name>_calibrated.daq` | Et arkiv, der kan genbehandles, i samme skema som en live-optagelse, men som nu angiver den kalibreringspakke, der har genereret det. Genimport af det kalibrerer det **ikke** en anden gang. |
| `<name>_calibrated.csv` | Spektral irradians i W/m²/nm på sensorens eget bølgelængdegitter, én række pr. måling, plus fotometriske kolonner: total effekt, fotopisk og skotopisk lux, PPFD med opdeling i blå/grøn/rød samt spidsbølgelængde. |

En DAQ-U eller DAQ-M, hvis kalibreringspakke ikke kan hentes — du er offline, eller
den pågældende sensor har ingen kalibrering gemt — **springes over med en begrundelse** og skrives aldrig ud
som en »kalibreret« fil med rå tællinger. Opret forbindelse til internettet og kør programmet igen. En DAQ-E
har sin egen kalibrering, så den har kun brug for dette, når enheden ikke er tilsluttet, og
der ikke er noget gemt i den lokale cache.

### DAQ-A: rå tællinger, og hvorfor det er det rigtige svar

**DAQ-A** er fra før systemet med kalibreringspakker pr. serienummer og har ingen pakke at
hente. Det er ikke en forglemmelse: en DAQ-A kalibreres i felten mod et
reflektansmål, og målbaseret kalibrering kræver kun sensorens *relative*
respons — hvilket er præcis, hvad dens rå tællinger er. Chloros kalibrerer med dem i dag.

Så en DAQ-A-optagelse eksporteres, men under et andet navn:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, ikke `_calibrated` — et andet filnavn i stedet for en markering inde i filen,
fordi angivelsen skal kunne bevares, når filen videresendes via e-mail som et simpelt navn. `.csv`-
overskriften angiver `raw spectral sensor counts (NOT irradiance)` og advarer om, at værdierne er
sammenlignelige **inden for** filen og ikke på tværs af sensorer. De kolonner, der kun har
betydning for den reelle bestrålingsstyrke — total effekt, lux, PPFD — efterlades tomme i stedet for at
blive beregnet ud fra tællingerne.

Ældre DAQ-A-SD-optagelser (skema v1.01 / v1.02) registrerer kun filens skrivetid, ikke et
tidsstempel pr. måling. Chloros vil ikke matche billederne med disse — at knytte et billede til et
skrivetidspunkt ville være forkert, uden at det nogensinde ser forkert ud — men eksporten læser dem fint, og
CSV angiver, hvilket ur det er baseret på.

For den fulde historie om reflektans — enkelt-sensor med et kamera og dobbelt-sensor (omgivelser/objekt) — se [Reflektans-arbejdsgange](reflectance.md).

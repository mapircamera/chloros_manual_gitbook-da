# Optagelse og .daq-formatet

En `.daq`-fil er Chloros’ optagelsesformat for lyssensorer: en **SQLite-database** med kalibrerede spektralrammer fra én DAQ-sensor. Optag en sådan fil under en optagelsessession, og reflektans-pipeline kan senere dividere hvert billede med den nedadgående strålingsintensitet, der blev målt på netop det tidspunkt.

## Hvad en .daq-fil indeholder

| Egenskab | Værdi |
| --- | --- |
| Container | SQLite-database, én fil pr. sensor pr. optagelse |
| Filnavn | Indeholder **sensor-ID**og et**tidsstempel**, f.eks. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum pr. billede | 135 punkter, 340–1010 nm i trin på 5 nm, plus CIE XYZ-tristimulus |
| Enheder | Kalibreret spektral irradians, **W/m²/nm** (fabriks-kalibreringspakke + cap-profil anvendt) |
| Indlejrede metadata | Sensor-ID (nøglen til at hente den pågældende enheds fabrikskalibrering) og den gældende cap-profil — se [Cap-profiler og kalibreret område](caps-and-range.md) |

Formatet er identisk på tværs af DAQ-U, DAQ-M, DAQ-E og DAQ-E-S, så efterfølgende databehandling er uafhængig af, hvilken transportenhed der har registreret dataene. En DAQ-E-S-optagelse gemmer også enhedens hældning, rulling og stigning ved siden af hvert spektrum.

**Optagelser overlever et strømsvigt (1.3.0).**En live-optagelse skrives med en journal og en fuld disksynkronisering ved hver commit, så et strømsvigt eller en afbrudt proces koster kun de sidste ca. 3 sekunder, ikke selve filen. En optagelse, der stoppes midt i skrivningen, kan efterlade en `<name>.daq-journal`-fil ved siden af `.daq`:**behold og kopier de to sammen.** Chloros ruller journalen tilbage, første gang den læser filen. En optaget disk forsinker filen, men aldrig sensorens målinger.

**En DAQ-E, der holder op med at levere data, genforbindes (1.3.0).** Når en DAQ-E's datastrøm stopper (f.eks. ved genstart af switchen eller et afbrudt link), genopretter backend-systemet forbindelsen med de samme indstillinger, og en optagelse på enheden fortsætter i den samme `.daq`-fil med det manglende interval markeret i filen.

Kalibreret optagelse kræver sensorens fabrikskalibreringspakke. For DAQ-U og DAQ-M henter backend-systemet pakken fra MAPIRs cloud ved hjælp af sensor-ID’et (optagelsen afvises, hvis dette ikke er muligt); DAQ-E-enheder er undtaget, da de gemmer deres kalibrering på selve enheden.

## Optagelse fra GUI’en

Optagelse i GUI'en kræver et **åbent projekt** (ellers er optagelsesknapperne deaktiveret):

* **Optag alt / Stop alt** — øverst i sidepanelet for lyssensorer; starter eller stopper en `.daq`-optagelse på alle tilsluttede sensorer på én gang.
* **Optag / Stop optagelse** — pr. sensor, i modalvinduet med tandhjulsindstillinger. En rød "REC"-indikator vises i sensorens live-inforækker under optagelse.

Filerne gemmes i `<project>/light_sensor/`, og når en optagelse stopper — uanset om det sker via Stop, Stop alt eller ved at afbryde forbindelsen til en optagelsessensor — føjes den færdige `.daq` **automatisk til det åbne projekt**. Den vises i projektets filliste uden behov for manuel tilføjelse og er allerede klar til reflektansbehandling.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Optagelse fra CLI



CLI'en optager via backendens sensorpool (backenden skal køre — disse kommandoer er tynde HTTP-klienter):

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
| `--output` / `-o` | `~/Documents/DAQ Live View/` på **backendens** filsystem, ikke på CLI'ens |

Forskellen mellem output-mapperne er vigtig, når CLI'en peger på et backend på en anden maskine: filen havner der, hvor backenden kører. Når backend'en befinder sig på samme computer, opløses en relativ `-o` (f.eks. `./out` ovenfor) i forhold til den mappe, hvor kommandoen blev kørt.

## Optagelse fra Python

`DAQSensorSession` (returneret af `chloros_sdk.connect_daq_sensor()`) viser den samme samlede optagelse: `record_start(output_dir=None, device_name=None)` returnerer filstien, `record_stop()` returnerer `{path, rows}`. Se [SDK-referencen](../reference/sdk-reference.md) for den fulde session-API. SDK’s direkte hardwareklasser (kun desktop-installationer) skriver optagelser til `~/Documents/DAQ/` som standard; for udgivne builds er den samlede sti ovenfor den understøttede metode.

## Brug af en .daq-fil ved databehandlingen

For at beregne reflektans ud fra billeddata har Chloros brug for nedadgående strålingsintensitet, der er tilpasset hver eksponering:

* **Opbevar `.daq` sammen med billeddataene.**Ved databehandlingen udleder pipelinen automatisk den**tidsstempelmatchede nedadgående stråling** fra en optaget `.daq` (enhver DAQ-model) — eller fra en DAQ-M-specifik `.csv` — der findes sammen med billederne. GUI-optagelser opfylder dette automatisk, da de tilføjes til projektet i det øjeblik, de stopper.
* **Kalibreringen hentes efter behov.** Hvis en fabrikskalibreringspakke pr. kamera eller pr. DAQ ikke allerede er gemt lokalt i cachen, henter Chloros den automatisk fra MAPIRs cloud ved første brug (internetforbindelse kræves én gang; gemmes i cachen under `~/.chloros/`).
* **Live-optagelser skriver deres egen sidecar.** For ethvert reflektansbillede, der er optaget live, gemmes den faktisk anvendte DAQ-måling som en `.daq`-fil i optagelsernes `daq/`-mappe, så optagelsen kan genbehandles senere uden den oprindelige optagelse.
* **Optagelser fra `chloros_scripts`-optagere (1.3.0).**En `.daq`-fil, der er skrevet af de offentlige [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)-optagere, angives som den DAQ-model, der har genereret den, og dens grænseværdi kan indstilles i**Projektindstillinger → DAQ-lyssensor** ligesom for enhver anden optagelse. Disse optagere skriver rå sensortællinger; Chloros henter sensorens fabrikskalibrering via seriel port og anvender den.
* **Hvem der står bag målingen (1.3.0).**Ved siden af hver optagelse `.daq` angiver en `.daq.cameras.json` det eller de kameraer, hvis billeder målingen beskriver, og behandlingen matcher den**kun med billeder fra disse kameraer**. Et selvstændigt kamera uden egen lyssensor låner derfor ikke længere et andet kameras måling — knyt en sensor til det, eller optag en. En optagelse foretaget med selve DAQ'en (GUI Record, `daq pool-record`) og enhver `.daq` uden den pågældende fil (herunder ældre optagelser) fungerer stadig for alle kameraer.

## Hvordan man får strålingsintensiteten ud igen

Når man behandler et projekt, eksporteres også alle lyssensoroptagelser, det indeholder, til en
`Light Sensor/`-mappe ved siden af billedprodukterne. Dette kræver **ikke** billeddata: en
lyssensoroptagelse i sig selv er en komplet optagelse, og en mappe, der kun indeholder `.daq`-
filer, er en gyldig indgang. Et projekt, der kun indeholder lyssensoroptagelser, viser en liste over dem og
behandles både fra appen og CLI, og de forbliver på listen efter kørslen. Kørslen
rapporterer, hvor mange lyssensorprodukter den har skrevet.

| Produkt | Hvad er det? |
| --- | --- |
| `<name>_calibrated.daq` | Et arkiv, der kan genbehandles, i samme skema som en live-optagelse, men som nu angiver det kalibreringsbundt, der har genereret det. Genimport af det kalibrerer det **ikke** en anden gang. |
| `<name>_calibrated.csv` | Spektral irradians i W/m²/nm på sensorens eget bølgelængdegitter, én række pr. måling, plus fotometriske kolonner: total effekt, fotopisk og skotopisk lux, PPFD med opdeling i blå/grøn/rød samt spidsbølgelængde. |

En DAQ-U eller DAQ-M, hvis kalibreringspakke ikke kan hentes — du er offline, eller
den pågældende sensor har ingen kalibrering gemt — **springes over med en begrundelse** og skrives aldrig ud
som en »kalibreret« fil med rå tællinger. Opret forbindelse til internettet og kør programmet igen. En DAQ-E
har sin egen kalibrering, så den har kun brug for dette, når enheden ikke er tilsluttet, og
der ikke er noget gemt lokalt.

### DAQ-A: rå tællinger, og hvorfor det er det rigtige svar

**DAQ-A** er fra før systemet med kalibreringspakker pr. serienummer og har ingen pakke at
hente. Det er ikke en forglemmelse: En DAQ-A kalibreres i felten mod et
reflektansmål, og målbaseret kalibrering kræver kun sensorens *relative*
respons — hvilket er præcis, hvad dens rå tællinger er. Chloros kalibrerer med dem i dag.

Så en DAQ-A-optagelse eksporteres, men under et andet navn:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, ikke `_calibrated` — et andet filnavn i stedet for en markør inde i filen,
fordi angivelsen skal kunne bevares, når filen videresendes via e-mail som et simpelt navn. `.csv`-
overskriften angiver `raw spectral sensor counts (NOT irradiance)` og advarer om, at værdierne er
sammenlignelige **inden for** filen og ikke på tværs af sensorer. De kolonner, der kun har
betydning for den reelle bestrålingsstyrke — total effekt, lux, PPFD — efterlades tomme i stedet for at
blive beregnet ud fra tællinger.

Ældre DAQ-A-SD-optagelser (skema v1.01 / v1.02) registrerer kun filens skrivetid, ikke et
tidsstempel pr. måling. Chloros vil ikke sammenholde billeder med disse — at knytte et billede til et
skrivetidspunkt ville være forkert, selvom det ikke umiddelbart ser forkert ud — men eksporten læser dem fint, og
CSV-filen angiver, hvilket ur det er baseret på.

For den fulde historie om reflektans — enkelt-sensor med et kamera og dobbelt-sensor (omgivelser/objekt) — se [Reflektans-arbejdsgange](reflectance.md).

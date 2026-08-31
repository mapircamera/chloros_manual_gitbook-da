# Brug af Chloros med AI-assistenter

Denne vejledning er skrevet til to målgrupper: mennesker og de AI-assistenter, som mennesker i stigende grad arbejder gennem. Hver side indeholder præcise værdier, standardindstillinger og kommandoer, der kan kopieres og indsættes, så en assistent (Claude, ChatGPT, Copilot, en kodningsagent osv.) kan skrive en fungerende Chloros-automatisering ved første forsøg.

Chloros-version: **

1.2.0**. CLI/SDK-platforme: Windows 10/11 x64 og Linux (x86_64 / Jetson aarch64).

## Hvad skal du give din assistent

| Ressource | URL | Hvad den bruges til |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Maskinlæsbart indeks over alle sider i denne manual. |
| **CLI Reference** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Den komplette `chloros-cli`-kommandoflade: alle kommandoer, flag, standardindstillinger, afslutningskoder og regler for output-mapper. Skrevet til brug i LLM. |
| **SDK-reference** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Den komplette `chloros_sdk` Python API: klasser, signaturer, undtagelser og eksempler. Skrevet til brug for LLM. |
| **Enhver side som rå Markdown** | tilføj `.md` til siden URL | f.eks. `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` returnerer siden som rå Markdown — ideelt til at indsætte i et kontekstvindue eller hente fra en agent. |

Links i manualen: [CLI Reference](reference/cli-reference.md) · [SDK Reference](reference/sdk-reference.md).

{% hint style="info" %}
De to referencesider er selvstændige: En assistent, der har læst den ene af dem, behøver ikke resten af manualen for at skrive et korrekt script.
{% endhint %}

## Færdige opskrifter

Kopier, udfyld `<placeholders>` og indsæt det i din assistent.

### 1. Behandl en flyvningsmappe i NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Overvåg et katalog med optagelser i batch

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Tilslut et LATTICE-array og foretag en optagelse

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Registrer DAQ-lyssensorspektre

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
DAQ-scripting fra kommandolinjen foregår altid via `daq pool-*`-familien (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Andre `daq`-underkommandoer, som din assistent måtte opfinde, er ikke tilgængelige i de udgivne versioner og afbrydes med en fejlmeddelelse.
{% endhint %}

## Hvorfor AI-skrevne scripts fungerer godt med Chloros

Hver af disse er en reel, verificeret adfærd i Chloros 1.2.0 — de fjerner de klassiske fejltilstande ved maskinskrevet automatisering:

* **Ingen besværlig opsætning.**SDK’s smart-connect-hjælpere (`connect_camera`, `connect_array`, `connect_daq_sensor`) og behandlingsindgangspunkterne (`ChlorosLocal`, `process_folder`)**starter automatisk det lokale backend**. Et genereret script kræver ikke, at GUI&#x27;en er åben, eller at en server startes manuelt — det kræver kun, at desktop/CLI-pakken er installeret.
* **Hele pipelinen udføres i ét trin.** `chloros_sdk.process_folder("path", indices=["NDVI"])` kører import → kalibrering → reflektans → indekseksport fra start til slut. Mindre overfladeareal, færre steder, hvor et genereret script kan gå galt.
* **Kørsler uden output foretager selvdiagnose.** Efter `process()` vedhæftes kørslens oversigt til resultatet, og alle behandlingshenvisninger (f.eks. *hvorfor* en kørsel ikke gav noget output) genudsendes også som en Python `UserWarning` — så selv et script, der aldrig inspicerer resultatdiktet, viser diagnosen.
* **CLI fejler højlydt.**En `chloros-cli process`-kørsel, der anmodede om resultater, men ikke skrev nogen, udskriver `Processing finished but wrote no image products.` og**afsluttes med en værdi forskellig fra nul**, så shell-scripts og CI opdager det ved en simpel kontrol af afslutningskoden. Vellykkede kørsler rapporterer `Image products written: N`.

En asymmetri, som en assistent bør kende: SDK&#x27;s `process()` udløser bevidst **ikke** en fejl ved et kørsel uden produkter — den rapporterer i stedet via oversigten/tipene. Hvis en Python-pipeline skal stoppe ved en tom kørsel, skal du kontrollere oversigten (opskrift 2 gør det).

## Forbehold

* **Chloros+ login kræves.**CLI og SDK kræver et**betalt** Chloros+-abonnement, hvilket håndhæves på serversiden: anmodninger mislykkes med `401 AUTH_REQUIRED`, hvis man ikke er logget ind, og med `403 PLAN_UPGRADE_REQUIRED` på det gratis abonnement. Kør `chloros-cli login` én gang pr. maskine, før du kører de genererede scripts. Se [Chloros+ Login](chloros+-login.md).
* **Capture-kommandoer styrer fysisk hardware.** Kommandoerne `lattice` / `daq` / `project` og sessionsobjekterne SDK opretter forbindelse til, streamer fra og udløser fysiske kameraer og sensorer. Gennemgå et genereret script, før det køres første gang, og kør det, mens du overvåger hardwaren.
* **Udfør stikprøvekontrol af resultaterne.** Kontroller produktmapperne og nogle få pixelværdier, inden resultaterne offentliggøres. Især skal TIFF-filer med reflektans skaleres pr. kilde — læs `Chloros:PixelScale` XMP-tagget (LATTICE: 32768 = 1,0 reflektans; Survey3: 65535) i stedet for at antage en divisor. Begge referencesider dokumenterer dette under &quot;Læsning af reflektanspixels&quot;.
* **Små faldgruber, der kan forstyrre den genererede kode:**`pool-record` skriver til**backend-hostens** filsystem (standard `~/Documents/DAQ Live View/`); på maskiner med flere netværksgrænseflader bør du foretrække `daq pool-connect --eth-host <ip-or-hostname>` frem for automatisk opdagelse; og brug `http://127.0.0.1:5000` (aldrig `localhost`) overalt, hvor en backend URL forekommer.

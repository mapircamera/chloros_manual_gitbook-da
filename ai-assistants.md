# Brug af Chloros sammen med AI-assistenter

Denne vejledning er skrevet til to målgrupper: mennesker og de AI-assistenter, som mennesker i stigende grad arbejder gennem. Hver side indeholder præcise værdier, standardindstillinger og kommandoer, der kan kopieres og indsættes, så en assistent (Claude, ChatGPT, Copilot, en kodningsagent osv.) kan skrive fungerende Chloros-automatisering ved første forsøg.

Chloros-version: **

1.3.0**. CLI-/SDK-platforme: Windows 10/11 x64 og Linux (x86\_64 / Jetson aarch64).

## Hvad du skal give din assistent

| Ressource | URL | Hvad den bruges til |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Maskinlæsbart indeks over alle sider i denne manual. |
| **CLI-reference** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Den komplette `chloros-cli`-kommandoflade: alle kommandoer, flag, standardindstillinger, afslutningskoder og regler for output-mapper. Skrevet til brug for LLM. |
| **SDK-reference** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Den komplette `chloros_sdk` Python-API: klasser, signaturer, undtagelser og eksempler. Skrevet til brug i LLM. |
| **Enhver side som rå Markdown** | tilføj `.md` til sidens URL | f.eks. returnerer `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` siden som rå Markdown — ideelt til at indsætte i et kontekstvindue eller hente fra en agent. |

Links i manualen: [CLI-reference](reference/cli-reference.md) · [SDK-reference](reference/sdk-reference.md).

{% hint style="info" %}
De to referencesider er selvstændige: en assistent, der har læst den ene af dem, behøver ikke resten af manualen for at skrive et korrekt script.
{% endhint %}

## Lad din assistent køre Chloros direkte: `chloros-mcp`

I alle ovenstående eksempler skal din assistent *skrive* et script eller en kommando. Med `chloros-mcp` styrer en assistent, der taler [Model Context Protocol](https://modelcontextprotocol.io) (MCP), såsom Claude Desktop, Claude Code eller Cursor, selve Chloros. Du stiller et spørgsmål i almindeligt sprog, og den kalder serverens værktøjer:

* **Behandl billeder:** opret et projekt, importer en mappe med optagelser, vælg indstillinger, behandl og rapporter, hvad kørslen eksporterede.
* **Kør hardware:** tilslut LATTICE-kameraer, arrays og DAQ-lyssensorer, vis forhåndsvisning, indstil eksponering og farve, tag billeder, optag og opdater firmware.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Serverens værktøjer dækker det, som [Python SDK](api-python-sdk.md) gør. Det installeres sammen med SDK&#x27;et, så der er normalt ikke andet at installere: på Windows placerer Chloros-installationsprogrammet `chloros-mcp.exe` i din Pythons `Scripts`-mappe, og i Linux-versionen placerer `.deb` filen `chloros-mcp` i `/usr/bin`.

**Claude-kode:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**tilføj dette til `claude_desktop_config.json` (**Indstillinger &gt; Udvikler &gt; Rediger konfiguration**åbner det), og genstart derefter Claude Desktop.**Cursor** bruger den samme indtastning i `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Hvis klienten melder, at serveren ikke kunne startes, er `chloros-mcp` sandsynligvis ikke angivet i den `PATH`, den ser: angiv i stedet den fulde sti (`where chloros-mcp` på Windows, `which chloros-mcp` på Linux).

Du skal have Chloros installeret på den samme computer og være logget ind der, i appen eller via `chloros-cli login`, på et hvilket som helst betalt Chloros+-abonnement (Copper eller højere). Serveren kan ikke logge ind for dig.

{% hint style="info" %}
Klientopsætning for hver app, indstillinger, hvert værktøj og hvordan en kørsel fungerer: [MCP-server (AI-assistenter)](reference/sdk-reference.md#mcp-server-ai-assistants) i SDK-referencen.
{% endhint %}

## Prompt-opskrifter

Kopier, udfyld `<placeholders>`, og indsæt det i din assistent.

### 1. Behandl en flyvemappe til NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Overvåg et optagelseskatalog i batch

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Tilslut et LATTICE-array og tag billeder

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Registrer spektre fra DAQ-lyssensorer

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
DAQ-scripting fra kommandolinjen foregår altid via `daq pool-*`-familien (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Andre `daq`-underkommandoer, som din assistent måtte opfinde, er ikke tilgængelige i de udgivne versioner og afsluttes med en fejlmeddelelse.
{% endhint %}

## Hvorfor AI-skrevne scripts fungerer godt med Chloros

Hver af disse er en reel, verificeret adfærd i Chloros 1.3.0 — de fjerner de klassiske fejlmodi ved maskinskrevet automatisering:

* **Ingen besværlig opsætning.**SDK’s smart-connect-hjælpeprogrammer (`connect_camera`, `connect_array`, `connect_daq_sensor`) og behandlingsindgangspunkterne (`ChlorosLocal`, `process_folder`)**starter automatisk det lokale backend**. Et genereret script behøver hverken, at GUI’en er åben, eller at en server startes manuelt — det kræver blot, at desktop-/CLI-pakken er installeret.
* **Hele pipelinen udføres med ét enkelt kald.** `chloros_sdk.process_folder("path", indices=["NDVI"])` kører import → kalibrering → reflektans → indekseksport fra start til slut. Mindre overfladeareal, færre steder, hvor et genereret script kan gå galt.
* **Kørsler uden output foretager selvdiagnose.** Efter `process()` vedhæftes kørslens oversigt til resultatet, og hvert behandlingshenvisning (f.eks. *hvorfor* en kørsel ikke gav noget output) genudsendes også som en Python-`UserWarning` — så selv et script, der aldrig undersøger resultat-dict&#x27;en, viser diagnosen.
* **CLI&#x27;en fejler tydeligt.**En `chloros-cli process`-kørsel, der anmodede om resultater, men ikke skrev nogen, udskriver `Processing finished but wrote no image products.` og**afsluttes med en værdi forskellig fra nul**, så shell-scripts og CI kan opdage det ved en simpel kontrol af afslutningskoden. Vellykkede kørsler rapporterer `Image products written: N` samt `(plus M clamp masks)`, når der blev skrevet masker. Capture-kommandoer angiver ligeledes navnet på alle filer, de ikke kunne skrive, og afslutter med `1`.
* **Relative stier betyder det, som et script forventer.** Med et lokalt backend opløses en relativ outputsti (`-o out/` i CLI, `output_dir="out/"` i SDK) i forhold til den mappe, hvor kommandoen eller scriptet kører fra.

En asymmetri, som en assistent bør kende: SDK’s `process()` udløser bevidst **ikke** en fejl ved en kørsel uden resultat — den rapporterer i stedet via oversigten/tipene. Hvis en Python-pipeline skal stoppe ved en tom kørsel, skal du tjekke oversigten (opskrift 2 gør det).

## Forbehold

* **Chloros+-login kræves.**CLI&#x27;en og SDK&#x27;en kræver et**betalt** Chloros+-abonnement, hvilket håndhæves på serversiden: anmodninger mislykkes med fejlkoden `401 AUTH_REQUIRED`, hvis man ikke er logget ind, og med `403 PLAN_UPGRADE_REQUIRED` på det gratis abonnement. Kør `chloros-cli login` én gang pr. maskine, før du kører de genererede scripts. Se [Chloros+-login](chloros+-login.md).
* **Capture-kommandoer styrer fysisk hardware.** `lattice` / `daq` / `project`-kommandoer, SDK-sessionobjekterne og `chloros-mcp`&#x27;s værktøjer opretter forbindelse til, streamer fra og udløser fysiske kameraer og sensorer. Gennemgå et genereret script, før det køres for første gang, og kør det, mens du overvåger hardwaren.
* **Udfør stikprøvekontrol af output.** Kontroller produktmapperne og nogle få pixelværdier, inden resultaterne offentliggøres. Især skaleres reflektans-TIFF-filer pr. kilde — læs `Chloros:PixelScale` XMP-tagget (LATTICE-filer indeholder det: 32768 = 1,0 reflektans; Survey3-filer har det ikke og bruger 65535 = 1,0) i stedet for at antage en divider. Begge referencesider dokumenterer dette under &quot;Læsning af reflektanspixels&quot;.
* **Små faldgruber, der kan forstyrre den genererede kode:**`pool-record` skriver til**backend-værts** filsystem (standard `~/Documents/DAQ Live View/`); på maskiner med flere netværksgrænseflader bør man foretrække `daq pool-connect --eth-host <ip-or-hostname>` frem for automatisk opdagelse; brug `http://127.0.0.1:5000` (aldrig `localhost`) overalt, hvor en backend-URL forekommer; et script, der gennemløber et projekts output-træ, bør springe `clamp_masks/`-mapper over (deres `*_clampmask.tif`-filer er masker, ikke billeder — se [Output Image Formats](output-image-formats.md#clamp-masks)); og på Windows, hvor lange stier er deaktiveret, skal projektmapperne holdes korte — en projektmappe, der er dybere end ca. 130 tegn, afvises, før behandlingen starter.

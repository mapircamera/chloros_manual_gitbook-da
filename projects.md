# GUI: Projekter I

Chloros kan du oprette projekter, som kan åbnes igen på et senere tidspunkt. Et projekt er en almindelig mappe (inden for din projektmappe), der indeholder:

* `project.json` — projektindstillinger, filliste og visningsindstillinger
* `cameras.json` — kameraer og arrays, der var tilsluttet, mens projektet var åbent, med deres indstillinger
* `sensors.json` — DAQ-lyssensorer, der var tilsluttet, mens projektet var åbent, samt kamera↔sensor-tilknytninger
* dine optagelser, `.daq`-optagelser og mapper med bearbejdet output

Der er ikke noget proprietært projektfilformat — mappen og dens JSON-filer udgør projektet, hvilket også gør det nemt at kopiere, arkivere og styre projekter fra [CLI](CLI.md) eller [Python SDK](api-python-sdk.md).

Medmindre du ændrer det i [Projektindstillinger](project-settings/project-settings.md), er projektmappen `Documents\MAPIR\Chloros Projects` i din Dokumenter-mappe (`~/Documents/MAPIR/Chloros Projects` på Linux).

{% hint style="warning" %}
**Windows: Hold projektmapperne korte, medmindre lange stier er aktiveret.**Windows leveres med lange stier deaktiveret, og i dette tilfælde må ingen filsti være længere end 260 tegn. Behandlede produkter placeres op til ca. 130 tegn under projektmappen, så fra og med version 1.3.0 afviser Chloros en projektmappe, der er for dyb — når du opretter eller åbner den, eller når du starter behandlingen — i stedet for at køre og ikke skrive noget. Meddelelsen angiver den længste projektmappe, der passer. Brug en kortere projektmappe, eller aktiver lange stier i Windows (installationsprogrammets indstilling**Aktiver lange stier i Windows**, der er markeret som standard) og genstart Chloros. Se [Start af behandlingen](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).
{% endhint %}

## Nyt projekt

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Vælg &quot;Nyt projekt&quot; i hovedmenuen, og indtast et unikt navn til dit projekt.

Hvis du har gemt projektmaler, vises en rullemenu med **Vælg mal** under navnefeltet — hvis du vælger en, starter det nye projekt med indstillingerne fra den pågældende mal. Skabeloner gemmes fra [Projektindstillinger](project-settings/project-settings.md): Indtast et navn i feltet &quot;Gem projekt skabelonnavn&quot; og klik på gem-ikonet.

## Åbn projekt

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>&quot;Åbn projekt&quot; viser en liste over alle projekter i din projektmappe, med <strong>Åbn projektmappe</strong> nederst</p></figcaption></figure>

Vælg »Åbn projekt« for at se en liste over eksisterende projekter i projektmappen. Hvis der ikke findes nogen projekter, åbnes den sekundære sidemenu ikke. Du kan se nogle GUI-oprettede projekter (t1, t2, t3) opført på ovenstående billede. Projekterne DATE\_TIME blev oprettet af CLI&#x27;en ved hjælp af standardnavngivningsskemaet for projekter. Hvis du klikker på et projektnavn, åbnes projektet.

Hvis du klikker på knappen &quot;Åbn projektmappe&quot;, åbnes din computers filudforsker i projektstien. Du kan justere projektstien i [Projektindstillinger](project-settings/project-settings.md).

Hvis nogle af projektets kildebilledfiler er blevet flyttet eller slettet, siden det sidst blev åbnet, viser Chloros en dialogboks, der angiver præcis, hvilke filer der mangler, i stedet for at åbne et tomt gitter.

## Dupliker projekt

Funktionen er tilgængelig, når et projekt er åbent. Vælg &quot;Dupliker projekt&quot; for at kopiere det aktuelle projekt under et nyt navn — Chloros foreslår det næste ledige navn (f.eks. &quot;MitProjekt (2)&quot;) — og duplikatet åbnes straks.

## Tilføj filer

Når et projekt er åbnet, skal du vælge »Tilføj filer« fra hovedmenuen for at tilføje individuelle billedfiler til det aktuelle projekt. Dette svarer til filbrowsers tilføjelsesfunktion, men er for nemheds skyld tilgængeligt direkte fra hovedmenuen.

## Tilføj mappe

Når et projekt er åbnet, skal du vælge &quot;Tilføj mappe&quot; fra hovedmenuen for at tilføje mapper med billeder til det aktuelle projekt. Du kan vælge flere mapper på én gang. Duplikerede filer ignoreres.

## Start/stop behandling

Når filer er tilføjet til et projekt, bliver &quot;Start behandling&quot; tilgængeligt i hovedmenuen. Dette svarer til at klikke på knappen Afspil/Start i den øverste overskrift. Under behandlingen skifter menupunktet til &quot;Stop behandling&quot;, så du kan standse processen.

## Tilslut til kamera / Tilslut til lyssensor

Nederst i hovedmenuen findes to hardware-genveje, der er tilgængelige uanset om der er et projekt åbent eller ej:

* **Tilslut til kamera** — åbner [fanen Kameraer](lattice/) for at tilslutte et LATTICE-kamera eller -array.
* **Tilslut til lyssensor** — åbner fanen [Lyssensorer](daq/) for at tilslutte en DAQ-lyssensor.

Hvis du tilslutter hardware, mens et projekt er åbent, gemmes det i projektet (se nedenfor). Uden et projekt er tilslutningerne kun gældende for den aktuelle session.

{% hint style="info" %}
Menupunkterne Tilføj filer, Tilføj mappe og Start/Stop behandling er kun synlige eller aktiverede, når et projekt er åbent, og der er tilføjet filer. De giver hurtig adgang til handlinger, der også er tilgængelige via sidepanelet Filbrowser og knapperne i overskriften.
{% endhint %}

## Projekter husker din hardware

Nyt i 1.2.0: Et projekt gemmer den hardware, du tilslutter, så længe det er åbent. Kameraer og arrays (med deres indstillinger pr. kamera, navne, farver og gitterlayout) gemmes som et øjebliksbillede i `cameras.json`, og lyssensorer (med navne, farver og kameratilknytninger) i `sensors.json` — automatisk, mens du arbejder.

Når du **genåbner** et projekt, tager Chloros ikke straks kontakt til nogen hardware. Hver del genopretter forbindelsen, første gang du åbner den fane, den hører til:

* Når du åbner fanen **Kameraer**, genoprettes forbindelsen til de gemte kameraer og arrays, og deres gemte indstillinger anvendes igen.
* Når du åbner fanen **Lyssensorer**, genoprettes forbindelsen til de gemte DAQ-sensorer, og deres gemte indstillinger, inklusive cap, anvendes igen. En sensor, der er indstillet til**Ingen (bar sensor)** i dens Cap-menu, forbliver bar. Et projekt, der er gemt af en tidligere version af Chloros, registrerede en bar sensor som det eneste valg, så den vender tilbage med solskinnets cosinus-korrektion; indstil den til Ingen endnu en gang, og gem projektet.

På denne måde vil åbning af et projekt, der kun er til gennemsyn eller eksport af billeder, aldrig aktivere kameraernes streaming. Hvis en gemt enhed ikke kan findes, når dens fane åbnes, viser en dialogboks, hvilke enheder der ikke er tilgængelige, så du kan genoprette forbindelsen eller fjerne dem.

## DAQ-optagelser og .daq-filer i et projekt

* `.daq`-optagelser, der foretages, mens projektet er åbent (fra fanen Lyssensorer eller under optagelser), **føjes automatisk til projektet**.
* Importerede `.daq`-filer og alle projektoptagelser vises i afsnittet **DAQ-lyssensor** under [Projektindstillinger](project-settings/project-settings.md), hver med sin egen profil til korrektion af lysstyrke.
* Under behandlingen leverer projektets `.daq`-filer nedadrettet belysning til reflektansprodukter — se [Outputbilledformater](output-image-formats.md).

## Kørsel af et gemt projekt uden grafisk brugergrænseflade

Et gemt projekt kan køres uden den grafiske brugergrænseflade:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` arbejder med en projektmappe-sti — se [CLI-reference](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` returnerer et projekthåndtag; `connect_all()` sætter alle gemte kameraer, arrays og lyssensorer i drift med deres gemte indstillinger (lyssensorer via et kørende Chloros-backend) — se [SDK-referencen](reference/sdk-reference.md).

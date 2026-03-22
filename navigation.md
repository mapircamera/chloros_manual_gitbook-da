# GUI: Navigation

Når du starter Chloros og Chloros (Browser) for første gang, starter programmet sit backend. Når det er klar, vises ikonet for hovedmenuen øverst til venstre <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Fra venstre mod højre indeholder den øverste overskrift:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Hovedmenu

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Fra hovedmenuen kan du:

* **Nyt projekt** — oprette et nyt projekt
* **Åbn projekt** — åbne et eksisterende projekt
* **Åbn projektmappe** — åbne projektmappen i din filudforsker
* **Tilføj filer** — tilføje individuelle billedfiler til det aktuelle projekt _(synligt, når et projekt er åbnet)_
* **Tilføj mappe** — tilføj en mappe med billeder til det aktuelle projekt _(synlig, når et projekt er åbnet)_
* **Start behandling / Stop behandling** — start eller stop billedbehandlingsprocessen _(aktiveret, når filer er tilføjet)_

{% hint style="info" %}
**Kun Windows**: Chloros Desktop GUI er tilgængelig på Windows. Linux-brugere bør se [CLI](CLI.md) og [Python SDK](api-python-sdk.md) for headless-behandling.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Afspil/Start-knap

Når den er aktiveret, starter knappen til start af behandling billedbehandlingspipeline.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Statusbjælke <img src=".gitbook/assets/image (5).png" alt="" data-size="line">I den gratis Chloros-tilstand, som behandler alle filer sekventielt, viser statusbjælken 2 faser: Målregistrering og Behandling.

I den betalte Chloros+-licenstilstande, som behandler alle filer samtidigt, viser statusbjælken 4 faser: Registrering, Analyse, Kalibrering, Eksport. Hvis du holder musemarkøren over Chloros+-statusbjælken, vises et udvidet panel med de 4 trin, så du kan følge med. Hvis du klikker på den øverste statusbjælke, fryses dropdown-panelet, og hvis du klikker igen, frigives det.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Sidemenu

Menuen i venstre sidepanel indeholder forskellige ikoner, du kan interagere med:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Projektindstillinger](project-settings/project-settings.md)

Fanen Projektindstillinger giver dig mulighed for at justere globale projektindstillinger og indstillinger for projektbehandling. Juster disse, inden du begynder at behandle dine filer.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Filbrowser

Tilføj filer/mapper og fjern filer fra projektet. Duplikerede filer ignoreres. Marker afkrydsningsfeltet i målkolonnen for ethvert målbillede, og behandlingen vil kun se på markerede billeder som mål, hvilket i høj grad fremskynder din behandlingstid. Brug knappen &quot;Billede/Metadata&quot; til at skifte mellem visning af det valgte billedes miniaturegitter og en detaljeret metadatatabel.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Billedfremviser](image-viewer-gui/opening-an-image-full-screen.md)

Når der klikkes på et billede i hovedbilledfremviseren, åbnes det i fuld skærm i fanen Billedfremviser.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Kort](image-viewer-gui/map-markers.md)

Se dine billeder på et interaktivt 2D-kort baseret på deres GPS-koordinater. Understøtter Google Maps og ESRI-kortudbydere og vælger automatisk den bedste tjeneste til din placering. Hold musen over markører for at se forhåndsvisninger af billedminiaturer.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Fejlfindingslog

Gennemgå loggen for fejlfindingsudskrifter, når der opstår problemer. Kopier/download loggen og send den til [MAPIR Support](https://www.mapir.camera/community/contact) for at få hjælp.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Brugerlogin](chloros+-login.md)

Sidepanelet til brugerlogin giver dig mulighed for at logge ind på din Chloros+-konto for at låse op for avancerede funktioner. Du kan også se den aktuelle applikationsversion samt justere sproget for den viste tekst i Chloros GUI og CLI.

# GUI: Navigation

Når du starter Chloros for første gang, starter programmet sit behandlingsbackend. Så snart backenden er klar, vises hovedmenuikonet øverst til venstre <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">, og fanerne »Kameraer« og »Lyssensorer« bliver tilgængelige i venstre sidepanel (de er nedtonede indtil da).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Fra venstre mod højre indeholder den øverste overskrift:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">-hovedmenuen

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Fra hovedmenuen kan du:

* **Nyt projekt**— oprette et nyt projekt. Hvis du har gemte projektskabeloner, vises en rullemenu med**Vælg skabelon**, så det nye projekt starter med indstillingerne fra en skabelon.
* **Åbn projekt**— åbne et eksisterende projekt. Listen indeholder en**Åbn projektmappe**-knap, der åbner projektmappen i din filudforsker.
* **Dupliker projekt** — kopier det aktuelt åbne projekt under et nyt navn (et frit navn som &quot;MitProjekt (2)&quot; foreslås) og åbn kopien. _(synlig, når et projekt er åbnet)_
* **Tilføj filer** — tilføj enkelte billedfiler til det aktuelle projekt _(synlig, når et projekt er åbnet)_
* **Tilføj mappe** — tilføj en eller flere mapper med billeder til det aktuelle projekt _(synlig, når et projekt er åbnet)_
* **Start behandling / Stop behandling** — start eller stop billedbehandlingsprocessen _(aktiveres, når filer er tilføjet)_
* **Tilslut til kamera** — spring til [fanen Kameraer](lattice/) for at tilslutte et LATTICE-kamera eller en array. Fungerer uden et åbent projekt.
* **Tilslut til lyssensor** — gå til fanen [Lyssensorer](daq/) for at tilslutte en DAQ-lyssensor. Fungerer uden et åbent projekt.

{% hint style="info" %}
**Kun Windows**: Chloros Desktop GUI er tilgængelig på Windows. Linux-brugere bør læse dokumentationen til [CLI](CLI.md) og [Python SDK](api-python-sdk.md) for headless-behandling.
{% endhint %}

###<img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Afspil/Start-knap

Når den er aktiveret, starter knappen til start af behandling billedbehandlingspipeline.

###<img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Statusbjælke<img src=".gitbook/assets/image (5).png" alt="" data-size="line">

I den gratis Chloros-tilstand, som behandler alle filer sekventielt, viser statusbjælken 2 faser: Målregistrering og Behandling.

I den betalte Chloros+-licensierede tilstand, hvor alle filer behandles samtidigt, viser fremskridtsbjælken 4 faser: Registrering, analyse, kalibrering og eksport. Hvis du holder musemarkøren over Chloros+-statusbjælken, vises et udvidet panel med de 4 fremskridtstrin, så du kan følge med. Hvis du klikker på den øverste statusbjælke, fryses dropdown-panelet, og hvis du klikker igen, frigives det.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Sidemenu

Menuen i venstre sidepanel indeholder forskellige ikoner, som du kan interagere med, i denne rækkefølge fra top til bund:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Projektindstillinger](project-settings/project-settings.md)

Under fanen Projektindstillinger kan du justere de globale projektindstillinger og indstillingerne for projektbehandlingen. Juster disse, inden du begynder at behandle dine filer.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Filbrowser

Tilføj filer/mapper og fjern filer fra projektet. Duplikater ignoreres. Markér afkrydsningsfeltet i målkolonnen for et hvilket som helst målbillede, så vil behandlingen kun søge efter mål blandt de markerede billeder, hvilket i høj grad fremskynder din behandlingstid. Brug skifteknappen »Billede/Metadata« til at skifte mellem visning af det valgte billedes miniaturegitter og en detaljeret metadatatabel.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Billedfremviser](image-viewer-gui/opening-an-image-full-screen.md)

Når der klikkes på et billede i hovedbilledviseren, åbnes det i fuld skærm på fanen »Billedviser«.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Kortviser](image-viewer-gui/map-markers.md)

Se dine billeder på et interaktivt 2D-kort baseret på deres GPS-koordinater. Understøtter Google Maps og ESRI-kortudbydere og vælger automatisk den bedste tjeneste til din placering. Hold markøren over markører for at se forhåndsvisninger af billedminiaturer.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kameraer](lattice/)

Tilslut og styr LATTICE-kameraer live — et ad gangen eller som synkroniserede multikamera-konfigurationer. Fanen viser live-forhåndsvisningsfelter med overlejringer og histogrammer, indstillinger pr. kamera og pr. opstilling samt optagelsesindstillinger, der bestemmer, hvilke kameraer og eksporttyper »Capture All« genererer. Tilgængeligt, når backend-systemet er klar; se [LATTICE-afsnittet](lattice/) for en fuldstændig gennemgang.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Lyssensorer](daq/)

Tilslut DAQ-lyssensorer — DAQ-U (USB), DAQ-M (Bluetooth) og DAQ-E (Ethernet) — og se deres live, kalibrerede spektrumdiagrammer i W/m²/nm. Herfra kan du gemme `.daq`-filer i det åbne projekt, omdøbe sensorer, vælge kapacitanskorrektionsprofiler og opdatere DAQ-E-firmwaren. Tilgængeligt, når backend er klar; se [DAQ-afsnittet](daq/) for den fulde vejledning.

#### Fejlfindingslog for <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line">

Gennemgå loggen for fejlfindingsudskrifter, når der opstår problemer. Kopier/download loggen og send den til [MAPIR Support](https://www.mapir.camera/community/contact) for at få hjælp.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Brugerlogin](chloros+-login.md)

Via sidepanelet til brugerlogin kan du logge ind på din Chloros+-konto for at få adgang til avancerede funktioner. Du kan også se den aktuelle version af applikationen samt ændre sproget for den viste tekst i Chloros-brugergrænsefladen og CLI.

# Kortmarkører

Fanen »Kort« viser dine billeder på et interaktivt 2D-kort ud fra deres GPS-koordinater. Den giver dig et geografisk overblik over en optagelsessession og er den hurtigste måde – lige efter importen – at fjerne billeder, du ikke ønsker at behandle.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Sådan åbner du fanen &quot;Kort&quot;

1. Åbn eller opret et projekt i Chloros
2. Importer billeder, der indeholder GPS-metadata
3. Klik på fanen **Kort** <img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> i venstre sidepanel
4. Kortet viser en markør ved hvert billedes GPS-placering

{% hint style="info" %}
**GPS påkrævet**: Kun billeder med GPS-koordinater i deres EXIF-metadata vises på kortet. Et billede uden koordinater er stadig en del af projektet og behandles som normalt — det har blot ingen markør.
{% endhint %}

***

## Justering af billeder fra fanen »Kort«

Fanen **Kort**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> har de samme knapper til tilføjelse <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> og fjernelse <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> af filer som fanen [**Filbrowser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Den viser den samme liste over projektfiler med geografiske kolonner:

| Kolonne        | Indhold                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Navn**      | Filnavnet, som det blev gemt fra kameraet                             |
| **Breddegrad**  | Decimale grader, seks decimaler                                |
| **Længdegrad** | Decimale grader, seks decimaler                                |
| **Højde**  | Meter, én decimal — `-`, når billedet ikke indeholder nogen højdeoplysning |

{% hint style="info" %}
Klik på en kolonneoverskrift for at sortere efter den; klik igen for at vende rækkefølgen.
{% endhint %}

{% hint style="warning" %}
**Højde angiver højden over havets overflade, ikke højden over jordoverfladen.** Værdien stammer fra billedets EXIF-tag `GPSAltitude`, som refererer til middelhavsniveauet. Det er ikke flyvehøjden over terrænet, og Chloros vil ikke udlede en jordprøveafstand ud fra den — over en mark 300 m over havets overflade registrerer en drone i 100 m AGL ca. 400 m her. Brug kolonnen til at finde afvigelser og bekræfte en ensartet flyvehøjde, ikke som en AGL-måling.
{% endhint %}

***

## Billedmarkører

Hvert billede med GPS-data får en markør ved dets koordinater.

### Visning af markører

* Markører placeres på de nøjagtige koordinater, der er registreret for hver optagelse
* Markører, der ligger tæt på hinanden, kan visuelt overlappe hinanden, når der zoomes ud — zoom ind for at adskille dem
* Valgte og fremhævede markører vises over de øvrige

### Forhåndsvisning ved musepeger

* **Hold musen over** en markør for at få vist et miniaturebillede af det pågældende billede med filnavnet
* **Klik**på en markør for at vælge billedet og**fastgøre** pop op-vinduet — det forbliver åbent, indtil du klikker et andet sted. Mens et pop op-vindue er fastgjort, fjernes det ikke, når du holder musen over andre markører
* Dette er den hurtige måde at finde et bestemt billede i en stor session uden at forlade kortet

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Fanen »Kort« viser alle geotaggede billeder i projektet</p></figcaption></figure>### Superzoom

{% hint style="success" %}
**SUPERZOOM**: Når du når den maksimale zoom, som fliseleverandøren har billeder til, forstørres fliserne yderligere i stedet for at stoppe, så du kan skelne mellem markører, der ligger næsten oven på hinanden.
{% endhint %}

* Superzoom aktiveres kun, når du er **ved** udbyderens maksimale zoomniveau for den pågældende placering, og fliserne er færdigindlæst. Under dette niveau fungerer zoomfunktionen normalt
* Området er **1× til 32×** ud over udbyderens eget maksimum
* En indikator i hjørnet viser den aktuelle superzoom som en procentdel, og en **×**-knap ved siden af den bringer dig tilbage til normal zoom med et enkelt klik
* Når du zoomer ud, går det altid videre til selve kortet, så du aldrig kan sidde fast i superzoom
* Zooming og panorering i superzoom overfører den resulterende forskydning tilbage til kortet, så det område uden for midten, du har flyttet til, fortsat henter kortfliser i stedet for at blive sort
* Markører tegnes som vektorelementer i stedet for at blive rasteriseret, så de forbliver skarpe på alle superzoom-niveauer

***

## Kortfliseudbydere

{% hint style="success" %}
**Automatisk valg**: Chloros vælger den flisetjeneste, der tilbyder det bedste zoomniveau for det sted, hvor dine billeder befinder sig. Du kan skifte manuelt når som helst.
{% endhint %}

| Udbyder        | Bemærkninger                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Bred global dækning; understøtter alle fire flisetyper                                                                                                            |
| **Esri ArcGIS**| Ofte luftfotos med højere opløsning i bestemte regioner. Flisetypen**Terræn** tilbydes ikke for Esri, og knappen er deaktiveret, når Esri er valgt |***

## Kortflisetyper

Vælg kortlagstypen med knapperne (fra venstre mod højre):

![](&lt;../.gitbook/assets/image (14).png&gt;)

| Type                 | Viser                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Terræn**          | Højdefarvelægning med kortdetaljer (veje, etiketter). Kun Google       |
| **Kort**              | Standardkortfliser — den løsning, der kræver mindst båndbredde              |
| **Satellit**        | Detaljerede satellitbilleder, ingen etiketter — den løsning, der kræver mest båndbredde |
| **Hybrid** (standard) | Satellitbilleder med veje og etiketter tegnet ovenpå                |

Fanen »Kort« åbnes i indstillingen **Hybrid**. Dit valg overføres til udbyderomskifteren, hvor udbyderen understøtter dette.***

## Kortnavigation

* **Zoom**: musens rullehjul eller zoomknapperne på kortet
* **Panorer**: Klik og træk
* **Fuld skærm**: Fuldskærmsknappen udvider kortet til hele vinduet***

## Anvendelsestilfælde

### Gennemgang af flyvevej

* Se dækningsområdet for en droneflyvning på et øjeblik
* Find huller, hvor der er udeladt en passage
* Bekræft, at flyvningen fulgte det planlagte mønster

### Gennemgang af jordmåling

* Se, hvordan de jordbaserede optagelser er fordelt
* Find kalibreringsmålrammer i forhold til måleområdet
* Beslut, hvor der er behov for yderligere optagelser

### Kvalitetskontrol

* Find billeder, der er taget et uventet sted, og fjern dem inden behandling
* Sorter efter højde for at finde et billede, der er taget i forkert højde, eller hvor GPS-positionen var dårlig
* Sammenlign billedernes placeringer med feltnotater

***

## Fejlfinding

### Der vises ingen markører

**Mulige årsager**

* Billederne indeholder ingen GPS-metadata
* GPS var deaktiveret på kameraet under optagelsen
* EXIF-data blev fjernet af anden software før import

**Hvad skal du gøre**: Kontroller, at GPS er aktiveret på kameraet, og importer de originale filer igen. Du kan kontrollere, om en bestemt fil har koordinater, ved at lede efter den i filtabellen under fanen Kort — et billede uden koordinater har ingen række der.

### Markører er placeret forkert

**Mulige årsager**: en dårlig satellitpositionering på optagelsestidspunktet eller GPS-afvigelse under sessionen.**Hvad skal du gøre**: dette er et problem, der opstod på optagelsestidspunktet, og som Chloros ikke kan rette efterfølgende. Til præcisionsarbejde skal du bruge en PPK/RTK-GPS-arbejdsgang — se indstillingen**Anvend PPK-korrektioner** i [Projektindstillinger](../project-settings/project-settings.md).

### Kortet er tomt, eller fliserne holder op med at indlæses

Kortfliseudbyderne er onlinetjenester. Hvis kortfliserne holder op med at komme ind, skal du kontrollere maskinens netværksforbindelse og derefter prøve at skifte udbyder. Hvis du havde zoomet meget ind, skal du trykke på **×**-nulstillingsknappen for at vende tilbage til et normalt zoomniveau og lade kortet anmode om kortfliserne igen.***

## Relaterede sider

* [**Billedgitter**](image-grid.md) — det samme billedsæt som miniaturer
* [**Åbning af et billede i fuld skærm**](opening-an-image-full-screen.md) — detaljeret gennemgang af et enkelt billede
* [**Tilføjelse af filer til et projekt**](../processing-images-gui/adding-files-to-a-project.md) — knapperne til tilføjelse/fjernelse af filer, som denne fane deler

# Kortmarkører

Fanen »Kort« viser dine billeder på et interaktivt 2D-kort baseret på deres GPS-koordinater. Dette giver et geografisk overblik over din optagelsessession og hjælper dig med at visualisere den geografiske dækning. Det er også nyttigt, når du først importerer dine billeder, så du hurtigt kan fjerne de billeder, du ikke har brug for at behandle.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Adgang til fanen Kort

1. Åbn eller opret et projekt i Chloros
2. Importer billeder, der indeholder GPS-metadata
3. Klik på fanen **Kort** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> i venstre sidepanel
4. Kortet viser markører ved hvert billedes GPS-placering

{% hint style="info" %}
**GPS påkrævet**: Kun billeder med indlejrede GPS-koordinater i deres EXIF-metadata vises på kortet. Sørg for, at GPS er aktiveret på dit kamera under optagelsen.
{% endhint %}

***

## Justering af billeder fra fanen Kort

Fanen **Kort**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> har den samme tilføj  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  og fjern  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  filer som fanen [**Filbrowser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Den viser også den samme liste over projektfiler, men med andre kolonneoverskrifter:

### Filnavn

* Det originale filnavn fra kameraet
* Bevarer kameraets navngivningskonvention (f.eks. IMG\_0001.RAW)

### Breddegrad

* Billedets breddegrad

### Længdegrad

* Billedets længdegrad

### Højde

* Billedets højde

{% hint style="info" %}
Ved at klikke på tabelkolonneoverskrifterne sorteres rækkeoplysningerne også
{% endhint %}

***

## Billedmarkører

Hvert billede med GPS-data er repræsenteret af en markør på kortet:

### Visning af markører

* Markører angiver de nøjagtige GPS-koordinater, hvor hvert billede blev taget
* Markører, der ligger tæt sammen, kan grupperes, når der zoomes ud
* Zoom ind for at se de enkelte billedplaceringer

{% hint style="success" %}
SUPER-ZOOM: Når du når det maksimale zoomniveau fra kortudbyderen, forstørres kortflisen ved yderligere zoom, så du kan se markører, der ligger tæt sammen.
{% endhint %}

### Forhåndsvisning ved musemarkering

* **Hold musen** over en markør for at se en miniatureforhåndsvisning af det pågældende billede
* Dette giver mulighed for hurtig visuel identifikation uden at forlade kortvisningen
* Nyttigt til at finde specifikke billeder i en stor optagelsessession

***

## Kortfliseleverandører

{% hint style="success" %}
**Automatisk valg**: Chloros vælger automatisk den flisetjeneste, der giver det bedste zoomniveau for din aktuelle kortplacering. Du kan skifte manuelt mellem udbydere, hvis du ønsker det.
{% endhint %}

Fanen Kort understøtter to fliseudbydere til baggrundskortbillederne:

### Google Maps

* Standard satellit- og kortbilleder fra Google
* Bedst til generel dækning på verdensplan

### ESRI

* Satellit- og luftfotos fra ESRI ArcGIS
* Tilbyder ofte billeder med højere opløsning i visse regioner

***

## Kortflisetyper

Du kan vælge kortlagstype (fra venstre mod højre):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Terræn

Viser højdeprofiler og kortfliser med detaljer (veje osv.)

### Kort

Viser standardkortfliser (lavere båndbredde) med detaljer (veje osv.)

### Satellit

Viser detaljerede satellitkortfliser (højere båndbredde)

### Hybrid

Viser satellitkortfliser med tilføjede detaljer (veje osv.)

***

## Kortnavigation

### Zoomkontroller

* **Zoom ind/ud**: Brug musens rullehjul eller zoomknapperne
* **Fuld skærm**: Vis kortet i fuld skærm

### Panoreringskontroller

* **Panorer**: Klik og træk for at bevæge dig rundt på kortet***

## Anvendelsestilfælde

### Visualisering af flyvevej

* Se dækningsområdet for droneoptagelser
* Identificer huller i billeddækningen
* Bekræft udførelsen af flyvevejen

### Gennemgang af jordbaseret kortlægning

* Se den geografiske fordeling af jordbaserede optagelser
* Find kalibreringsmålbilleder i forhold til kortlægningsområdet
* Planlæg yderligere optagelsessteder

### Kvalitetskontrol

* Identificer hurtigt billeder, der er optaget på uventede steder
* Bekræft GPS-nøjagtigheden på tværs af datasættet
* Krydshenvis billedplaceringer med feltnoter

***

## Fejlfinding

### Der vises ingen markører

**Mulige årsager:**

* Billederne indeholder ikke GPS-metadata
* GPS var deaktiveret på kameraet under optagelsen
* EXIF-data blev fjernet af ekstern software

**Løsning**: Kontroller, at GPS er aktiveret på dit kamera, og importer de originale filer igen

### Markører på forkert placering

**Mulige årsager:**

* Kameraets GPS havde dårlig satellitposition
* GPS-afvigelse under optagelsen

**Løsning**: Dette er typisk et problem ved optagelsestidspunktet; overvej at bruge PPK/RTK GPS til præcisionsanvendelser

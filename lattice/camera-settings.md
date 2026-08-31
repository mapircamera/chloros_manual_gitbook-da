# Kameraindstillinger

Fanen **Kameraer**er Chloros’ live-kontrolpanel til LATTICE-kameraer: et hovedområde, der viser hvert tilsluttet kamera som en live-flise, samt en sidebjælke, der skifter mellem tre sider —**kameralisten**, et**indstillingspanel**(indstillinger pr. kamera, for kameraopstillinger eller optagelse — én ad gangen) samt**Indeksberegneren**. Denne side beskriver alle kontrolelementer i kameralisten, indstillingspanelet pr. kamera og array-indstillingspanelet. Optagelsestilstande, valg af eksporttype og »Capture All«-forløbet findes på den tilhørende side [Optagelsesindstillinger og -tilstande](capture.md).

Fanen „Kameraer“ vises i sidepanelet, når Chloros-backend er klar. Alle nedenstående kontrolelementer kommunikerer med det lokale backend via `127.0.0.1:5000`; ændringer gælder straks for live-kameraet, medmindre andet er angivet.

## Kameratyper, der anvendes på denne side

Kontrolelementerne vises eller skjules afhængigt af, hvilken kameratype der er valgt. I denne vejledning anvendes følgende termer gennemgående:

| Term | Betydning | Filterkanaler |
| --- | --- | --- |
| **RGB-kamera** | LATTICE M3C med FRGB-filter (modellen indeholder `-FRGB`) | Red / Green / Blue |
| **Bayer multispektral** | LATTICE M3C med FRGN, FOCN eller FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — ét smalbåndsfilter, ét kalibreret bånd | Enkeltbånd |
| **Array-medlem** | Et kamera tilsluttet som del af et synkroniseret array (kombineret eller separat skærm) | Pr. filter |

RGB-kameraer gennemgår fotometrisk behandling (hvidbalance, farveprofiler, gamma); multispektrale og monokameraer gennemgår den radiometriske kæde og springer de fotometriske indstillinger over. Array-medlemmer overfører indstillinger på stream-niveau (pixelformat, opløsning, binning, trigger, billedhastighed) til arrayet — disse rækker bliver skrivebeskyttede i panelet pr. kamera og flyttes i stedet til panelet med array-indstillinger.

## Hovedområdet

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

for livebilleder Når der ikke er tilsluttet nogen kameraer, viser livebilledområdet en velkomstmeddelelse med teksten **&quot;Tilslut et kamera for at komme i gang&quot;**med to knapper:**Tilslut kamera**(grøn, åbner dialogboksen til tilslutning af et enkelt kamera) og**Tilslut array** (blå, åbner dialogboksen til tilslutning af et array). Selve tilslutningsdialogboksene er beskrevet i [Tilslutning af kameraer](connecting.md); array-begreber (synkronisering, lag, båndbredde) i [Kameraopstillinger med flere kameraer](arrays.md). Når du åbner et gemt projekt, der indeholder kameraer, viser startskærmbilledet i stedet en spinner med teksten &quot;Genåbner N gemte kameraer…&quot; , mens Chloros gendanner streamene fra den sidste session.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Øverste bjælke

| Kontrol | Funktion |
| --- | --- |
| **Skift mellem visningstilstande**| Skifter mellem**gittervisning**(alle felter som celler) og**listevisning** (arrayer i fuld bredde øverst, ÉT aktivt kamera nedenfor). Værktøjstip: &quot;Skift til gittervisning&quot; / &quot;Skift til listevisning&quot;. |
| **Gitterlås**(hængelås) | Standard**låst** — felter fastlåst på plads. Lås op for at trække og omarrangere felter til en vilkårlig plads (mellemrum bevares). Gitteret låses automatisk igen, hver gang et nyt kamera opretter forbindelse. Værktøjstip: &quot;Lås gitter op (aktivér trækning af felter)&quot; / &quot;Lås gitter (fastlås felter på plads)&quot;. |
| **Feed-zoom**-skyder | Felstørrelse, fra 60 px op til den fulde bredde af beholderen. Felterne bevarer et 4:3-format. Ved en cellebredde under 200 px skjules navn og fps-overlays for at holde feltet rent. |

### Feed-brikker

Hvert kamera gengiver en sammensat live-brik; et kamera kan desuden vise tre gråtonede **kanalopdelte** brikker (se [Kanalopdelinger](#display-overlays-drawn-over-the-live-feed)), og arrays gengiver en samlet brik. Den aktive flise har en markeringsring i kameraets (eller arrayets) farve.

Når man holder musen over en flise, vises en **X**-lukkeknap:

* Hvis man lukker en **sammensat** flise, mens dens kanalopdelinger forbliver synlige, skjules blot den sammensatte flise.
* Hvis du lukker den **sidste synlige flise fra et enkeltstående kamera**, afbrydes forbindelsen til det pågældende kamera.
* **Opdelte fliser fra et kombineret array afbryder aldrig** kameraet — de skjules blot.

Når gitteret er låst op, kan du trække en hvilken som helst flise til en hvilken som helst plads; layoutet gemmes sammen med projektet.

## Sidebjælke — kameraliste

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

Den første side i sidebjælken viser alle tilsluttede kameraer og arrays:

* **Tilslut kamera**(grøn) /**Tilslut array** (blå, viser &quot;Registrerer...&quot; under scanning). Begge er deaktiveret, mens en tilslutningsdialog er åben.
* **Optag alt** (rød) — optager alle kameraer på listen med de eksporttyper, der er valgt i Optagelsesindstillinger. Kræver et åbent projekt. Fuldt dokumenteret i [Optagelsesindstillinger og -tilstande](capture.md).
* **Tandhjul for optagelsesindstillinger** (ved siden af Optag alt) — åbner [panelet Optagelsesindstillinger](capture.md#the-capture-settings-pane). Deaktiveret uden et projekt eller under optagelse.

### Kamerarækker

Hver kamerarække viser en farvekodet ramme (kameraets brugerdefinerede farve), en &quot;CAM&quot;-mærkat — med et blåt **M**(master) eller grønt**S** (slave) for array-medlemmer — samt visningsnavnet. Standardnavnet er `LATTICE-MODEL (serial)`; omdøb det fra indstillingspanelet for det enkelte kamera. Række-knapper:

| Knap | Effekt |
| --- | --- |
| **Øje**| Skift synlighed. Skjulte kameraer forlader gitteret og**udelades fra Capture All**. |
| **Tandhjul** | Åbn indstillingspanelet for det enkelte kamera (næste afsnit). |
| **Pause / Afspil**| Fryser live-forhåndsvisningen**kun på skærmen** — optagelsen i backend kører videre. Kameraer i pause kan ikke optage. |
| **X** | Afbryd forbindelsen. Brugergrænsefladen opdateres med det samme (i bedste fald); selve afbrydelsen i backend kan tage 10–30 sekunder at gennemføre. |

### Array-rækker

En array-række viser et &quot;ARRAY&quot;-mærke i arrayets farve, arrayets navn (kan omdøbes i arrayindstillingerne) og en **DAQ · tænd/sluk**-knap —**tændt**, når lyssensoren på array-niveau er indstillet *eller* et hvilket som helst medlem har en sensor pr. kamera; dens værktøjstip viser præcist, hvilken sensor der sender signaler til hvad. Medlemskameraer vises indrykket nedenunder med deres egne rækker. Knapper i array-rækker: **øje**(skjuler/viser ALLE medlemmer samlet),**tandhjul**(panel med arrayindstillinger),**X**(afbryder forbindelsen til hele arrayet).

Lyssensorens (DLS) status, der bruges i array-rækker og i panelet med array-indstillinger, har fire tilstande:**slukket**,**venter**(intet spektrum endnu),**aktiv**(et spektrum er ankommet inden for de sidste 3 sekunder), og**forældet** — intet nyt spektrum i 3 sekunder, men den sidste måling *bruges stadig* (DAQ-målinger udløber aldrig på indlæsningsstien).

Du kan trække enkeltstående kameraer og hele array-grupper forbi hinanden i sidepanelet for at omarrangere listen; array-medlemmer kan ikke trækkes uafhængigt af hinanden.

## Indstillingsruden pr. kamera

Åbnes med **tandhjulet** på en kamerarække. Ruden glider hen over kameralisten.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Overskrift**: kameraets**farveprøve**(klik for at åbne en indbygget farvevælger — indstiller farven på sidebjælkens kant og flisernes markeringsring),**navnet**med en blyantknap til**Omdøb**(hvis du gemmer et tomt navn, vender det tilbage til standardnavnet `MODEL (serial)`) og**×** for at lukke.

### Live-histogram

Øverst i ruden vises et live-luma-histogram, der beregnes ud fra JPEG-forhåndsvisningen med en frekvens på ca. 8 Hz. Gennemsnittet er Bayer-vægtet — (R+2G+B)/4 — for at matche kameraets egen AE-måling.

* **Orange stiplet linje**= AE-målet.**Træk den vandret for at justere målet** — der sendes en kommando, når du slipper, og ved at trække skifter AE-målmodusen til Manuel.
* **Green fuld linje** = den faktiske gennemsnitlige luma (det, som AE leverer i øjeblikket).
* **RGB-knap** (øverst til højre): skifter mellem båndvise overlejrede histogrammer farvet efter kameraets filter (f.eks. på FRGN: grå NIR, grøn, rød). På monokameraer (M3M) står der »MONO« på knappen, og den er deaktiveret — mono viser altid luma-histogrammet for et enkelt bånd.
* Mærkaterne på X-aksen følger sensorens bitdybde for det aktuelle pixelformat: 0..255, 0..1023, 0..4095 eller 0..65535.

### Rækker

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

med kameraoplysninger | Række | Adfærd |
| --- | --- |
| **Model** | Skrivebeskyttet (f.eks. `LATT-M3C-L87-FRGN`). |
| **Radiometrisk kalibrering**| Green**&quot;Aktiv&quot;**-mærke med en billedtekst, der viser kalibreringsniveau, hash, kalibreringsdato og båndliste, indlæst fra kameraets kalibreringspakke (se [Fabriksradiometrisk kalibrering](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Skjult for RGB-kameraer** — disse har en fotometrisk hvidbalancekalibrering, ikke strålingsintensitet pr. bånd. |
| **Kalibreringsrapport**|**Download**-knap — åbner kameraets NIST-kalibreringscertifikat (pr. serienummer) som PDF i din operativsystems visningsprogram. Hvis certifikatet endnu ikke er gemt i cachen, viser Chloros i stedet et tip. |
| **Serienummer** | Skrivebeskyttet. |
| **Firmware**| Viser den aktuelle version og finder derefter den tilgængelige version til denne model (gemt i cachen pr. model — en gruppe på N kameraer tjekker serveren én gang). Status: »Tjekker…« →**»Opdater til X«**-knap → »Oplader…« → »Opdateret fra A til B« / »Mislykkedes: …« / &quot;Springet over: …&quot; / grøn**&quot;Opdateret&quot;**. Værktøjstip til opdateringsknappen: &quot;Fabriksindstilling + flash + omprogrammering af UserSet1. ~2–3 minutter; afbryd ikke forbindelsen.&quot; |
| **IP** | Skrivebeskyttet. |
| **Temperatur** | Skrivebeskyttet, opdateres hvert 3. sekund. Bliver orange ved ≥65 °C og rød med et ⚠ ved ≥75 °C. |
| **Kalibreringsmål** afkrydsningsfelt | Aktiverer ArUco-reflektansmåldetektion med en NDVI-valideringstabel pr. panel under live-feedet (listevisning). Kun til session — åbnes altid i slukket tilstand. |
| **Lyssensor**-rullemenu | Binder en DAQ-lyssensor (DAQ-E/M/U, fra listen under fanen »Lyssensorer«) til dette kamera til belysningskorrektion af nedadgående lys (DLS) belysningskorrektion og prædiktiv automatisk eksponering. &quot;Ingen&quot; fjerner tilknytningen. Hvis der ikke er tilsluttet nogen sensorer, viser rullemenuen &quot;(ingen sensorer tilsluttet — åbn fanen DAQ)&quot;. Tilknytningen gemmes sammen med projektet. |

### Eksponering og forstærkning

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Alle numeriske indtastninger her bruger drejeknapper, der skal holdes nede for at øge værdien: tryk = ±1, hold &gt;1,5 s = ±10, hold &gt;3 s = ±100. Værdien sendes til kameraet, når du slipper.

| Kontrol | Område / valgmuligheder | Standard | Gælder for | Hvad den gør |
| --- | --- | --- | --- | --- |
| **Eksponering (us)**| Kameraets live min./maks. | Auto | Alle | Eksponeringstid i mikrosekunder, med en**Auto/Manuel**-knap. Auto = kontinuerlig automatisk eksponering på kamerasiden. |
| **Forstærkning (dB)**| Kameraets aktuelle min./maks. (f.eks. op til 48 dB) | Manuel (slået fra) | Alle | Analog/digital forstærkning med sin egen**Auto/Manuel**-knap. |
| **AE-mållysstyrke**| 0–255 | 80, tilstand**Auto**| Alle (kan redigeres, når AE eller automatisk forstærkning er aktiveret) | Den lysstyrke, som AE sigter mod. I**Auto**(standardindstillingen) vælger en histogram-baseret backend-controller selv målet og holder eksponeringen på 60–75 % af sensorens maksimum. Hvis du indtaster en værdi eller trækker i histogrammets orange linje, skifter den til**Manuelt**. |
| **AE-udjævning** | 0,5–40, trin 0,1 | 8,0 | Alle | AE-dæmpning. Værktøjstip: »Lavere = AE reagerer hurtigere (kan pulsere ved høje fps). Højere = jævnere / langsommere.&quot; Værdier langt under standardindstillingen kan få AE til at pulsere og destabilisere streamingen ved høje billedhastigheder; 8,0 er den stabile standardindstilling. |
| **AE-interesseområde**| Afkrydsningsfeltet &quot;Aktiver&quot; +**Sigt**-knappen | Fra | Alle | Når denne funktion er aktiveret, måler AE kun det grønne, stiplede område i stedet for hele billedet.**Sigt** aktiverer »klik-for-at-placere« på live-feedet: et klik centrerer et område ved 30 % af billedet; klik-og-træk tegner et brugerdefineret rektangel (minimum 5 % × 5 %). Sigt deaktiveres automatisk efter én placering. Området kortlægges tilbage til kameraets oprindelige koordinater under enhver rotation/spejling, du har indstillet, og gemmes sammen med projektet. |
| **AE-justeringshastighed** | 0,1–5, trin 0,1 | 1,0 | Kun for array-medlemmer | Hvor hurtigt det automatiske AE-mål sporer ændringer i scenens lysstyrke; 1,0× kontrollerer igen hvert 2,5 s. |
| **Highlight-beskyttelse** | Streng (1 %) / Normal (5 %) / Lempet (15 %) | Streng | Kameraer, der understøtter indstillingen | Hvor stor en del af billedet der må klippes til hvidt, før AE mørkner billedet. |

{% hint style="info" %}
**Lyskrav til Bayer-multispektralkameraer (RGN / OCN / NGB):** Motivet skal have tilstrækkeligt lys i alle tre kanaler, ellers fungerer kalibreringen ikke korrekt — en enkelt sensoreksponering dækker alle tre spektre. Brug en DAQ-lyssensor til at måle lyset, eller gå over til fuldstændig monokrom (M3M), så hvert bånd får sin egen eksponering. Hvis en optagelse ikke overholder dette, registrerer Chloros det og advarer dig (unmix-clamp-meddelelsen).
{% endhint %}

### Pixelformat og

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

opløsning**Array-medlemmer** viser skrivebeskyttede rækker for »Current« (format + BxH) og »Binning« med bemærkningen »Indstilles i array-indstillinger« — en genstart af streamen på et medlem ville bryde synkroniseringen, så disse administreres i [panelet med array-indstillinger](#array-settings-pane).**Fritstående kameraer** har:

| Kontrol | Valgmuligheder | Hvad det gør |
| --- | --- | --- |
| **Pixelformat** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Sensorens pixelformat (bitdybde). |
| **Opløsning** | Fuld / Halv / Kvart | I forhold til den aktuelle binning: Fuld = 2048/N × 1536/N ved N×N-binning. |
| **Binning** | 1x1 (ingen) / 2x2 / 4x4 | Hardware-N×N-binning — større værdier giver lavere opløsning, men øger SNR og billedhastigheden. Ændring af denne indstilling genstarter streamen og nulstiller eventuelle ROI’er til det nye fulde synsfelt. |
| **Aktuel** | skrivebeskyttet | Den faktiske WxH og (x, y)-forskydning, der er aktiv. |

### Live-forhåndsvisning

Alt i dette afsnit vedrører **kun visningen**— det ændrer det, du ser i live-feedet, mens gemte optagelser forbliver lineære og uændrede — med én undtagelse:**Vignette** er radiometrisk og påvirker også eksporter (beskrevet nedenfor).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Kontrol | Interval / valgmuligheder | Standard | Gælder for | Hvad den gør |
| --- | --- | --- | --- | --- |
| **Renderopløsning** | 360p (hurtigst) / 480p / 720p / 1080p / Sensorens oprindelige opløsning (langsomst) | 720p | Alle | Den opløsning, som backend-systemet kører den radiometriske forhåndsvisningskæde i. En lavere opløsning giver højere billedfrekvens uden at ændre synsfeltet. |
| **Indeks**| Afkrydsningsfelt + tandhjul | Fra | Kun Bayer-multispektral,**ikke** medlemmer af kombinerede arrays | Live-forhåndsvisning af vegetationsindeks. Tandhjulet åbner den delte [Indeksberegner](#index-calculator-pane), der er forudindlæst med kameraets filter-naturlige bånd (f.eks. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Det brugerdefinerede udtryk plus LUT (til/fra, standardniveau 3, standardmin. 0,2, standardmaks. 1) beregnes for hvert forhåndsvisningsbillede. Medlemmer af kombinerede arrays skjuler denne række — arrayet har ét fælles indeks. |
| **Hvidbalance** | Fra / En gang / Kontinuerlig + en genoptagelsesknap | Kontinuerlig | Kun RGB | Live-hvidbalance. Opdateringsknappen genoptager hvidbalancen fra det aktuelle DLS-spektrum (deaktiveret, når tilstanden er Fra). |
| **Gamma** | Til / Fra | Til | Kun RGB | Vis gamma (γ = 2,2 LUT) i live-forhåndsvisningen. Gemte optagelser forbliver lineære. |
| **Støjfjernelse** | Afkrydsningsfelt + styrke 0–100 | Fra / 50 | Alle (pr. kamera, også inden for arrays) | Bilateralt filter i live-forhåndsvisningen. Højere værdi = glattere, men blødere detaljer. |
| **Skarphed** | Afkrydsningsfelt + styrke 0–100 | Fra / 30 | Alle | Unsharp mask i live-forhåndsvisningen, anvendes som det sidste. Kan forstærke støj. Kun i forhåndsvisningen. |
| **Vignette**| Afkrydsningsfelt + styrke 0–100 | Fra / 0 | Alle | Manuel fjernelse af resterende vignettering (lysner hjørnerne), lagt oven på arrayets Smart Vignette-estimering.**Radiometrisk — påvirker live-visningen OG eksporten**, i modsætning til Støjfjernelse/Skarphed. |
| **Farveprofil** | Raw / Lineær / Naturlig / Forbedret / Brugerdefineret temperatur | Naturlig | Kun RGB | Se nedenfor. |
| **Farvetemperatur** | 2000–10000 K, trin 100 | 5500 K | Kun RGB, brugerdefineret temperaturprofil | Fastlåser hvidbalancen til en fast korreleret farvetemperatur (DLS-indgang ignoreres). Den senest valgte Kelvin-værdi huskes på tværs af profilskift. |
| **Mætning** | 0–200 (100 = neutral) | 100 | Kun RGB | HSV-mætning i live-forhåndsvisningen. |
| **Kontrast** | 0–200 (100 = neutral) | 100 | Kun RGB | Lineær kontrast omkring mellemgrå i live-forhåndsvisningen. |
| **Spejl H / Spejl V** | Afkrydsningsfelter | Fra | Alle | Spejl forhåndsvisningen vandret / lodret. |
| **Rotation**| 0° / 90° / 180° / 270° | 0° | Alle | Roter forhåndsvisningen. Retningen anvendes i slutningen af backend-forhåndsvisningskæden —**gemte optagelser forbliver i kameraets oprindelige retning**, og sammensatte visninger i arrayet ignorerer den. |**Farveprofil-semantik** (RGB-kameraer):

* **Raw** — omgår behandlingskæden fuldstændigt.
* **Lineær** — mørkt signal + fladfelt + hvidbalance; ingen farvematrix, ingen gamma.
* **Naturlig** *(standard)* — lineær plus den målte farvekorrektionsmatrix og en scenetilpasset tonekurve.
* **Forbedret**— Naturlig plus vibrance og CLAHE-lokal kontrast. Tillægget gælder**kun for live-forhåndsvisning** — gemte billeder får altid den fulde efterbehandling uanset profil.
* **Brugerdefineret temperatur** — Natural med hvidbalance fastsat til den valgte Kelvin-værdi.

{% hint style="warning" %}
For Naturlig, Forbedret og Brugerdefineret temperatur viser panelet en tonebemærkning: billederne lysnes op i forhold til deres egen scene, så gemte *skærmbilleder* kan ikke sammenlignes billede for billede. **Eksporter strålingsintensitet eller refleksion til målinger.**
{% endhint %}

### Skærmoverlejringer (tegnet oven på live-feedet)

Disse findes kun i frontend — de lægges oven på videoen og påvirker aldrig streamen eller optagelserne.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Overlejring | Kontroller | Standard | Hvad den gør |
| --- | --- | --- | --- |
| **Zebra** | Afkrydsningsfelt + tærskel 200–255 | Fra / 250 | Magenta diagonale striber på beskårne pixels. |
| **Sigtemærke** | Afkrydsningsfelt | Fra | Markering i midten af billedrammen. |
| **Gitter** | Fra / 3 × 3 / 9 × 9 | Fra | Kompositionsgitter. |
| **Histogram** | Afkrydsningsfelt + bredde 0,10–0,90 af billedet | Fra / 0,25 | En histogramstribe på feedet. |
| **Fokuspeak** | Afkrydsningsfelt + tærskel 20–200 + farveprøve | Fra / 80 / `#ff5722` | Sobel-kantmarkering til fokusering. |
| **Kanalopdelinger** | &quot;Vis opdelinger (Red / Green / NIR)&quot; / &quot;Skjul opdelinger&quot;-knap | Skjult | Tilføjer tre uafhængige gråtoner pr. kanal ved siden af det sammensatte billede (knapteksten følger kameraets filterkanaler). Hver opdelt flise kan trækkes og har samme kantfarve som kameraet. Ikke tilgængelig på monokameraer. Gemmes sammen med projektet. |

### Spotmåler

* **Klik for at tage en prøve**-afkrydsningsfelt: Klik på live-feedet for at tage en prøve af en enkelt pixel (et krydsmarkerer den), eller klik og træk et område for at få et pixelgennemsnit.**Ryd**sletter prøven og krydset. Kan ikke bruges samtidig med AE-ROI**Sigt**-tilstanden.
* **Vis**-rullemenu:**Raw (bitdybde)**— oprindelige digitale tal i sensorens bitdybde (f.eks. 12-bit → 0..4095) — eller**Display (8-bit)** (standard). Når et live-indeks er aktivt, viser Display i stedet den beregnede indeksværdi (f.eks. NDVI).
* Aflæsningspanelet viser pixelkoordinater, billedstørrelse, pixelformat, bitdybde og en kanaltabel (Chan / Værdi / %) med båndbetegnelser og bølgelængder; Bayer-grønpar er gennemsnitsberegnet; regionsprøver viser „N px avg“.

Spotmålerens tilstand gælder kun for den aktuelle session.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Prædiktiv automatisk eksponering (DLS-styret)

Dette afsnit vises kun, når **mindst én DAQ-lyssensor er tilsluttet** — beregningsmodulet har brug for et live nedadgående spektrum for at kunne styre det.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Kontrol | Område | Standard | Funktion |
| --- | --- | --- | --- |
| **Aktiver** | Afkrydsningsfelt | Til (fritstående kameraer) | En lukket-form-løser bruger DLS-spektret samt kameraetskalibreringspakkes skalærer til at placere det lyseste bånd tæt på mætning, samtidig med at det mørkeste bånd holdes over SNR-grænsen — en enkelt eksponeringsskrivning pr. løsning, ingen stabiliseringsloop. Designet til solcelledrevne timelapse-optagelser, hvor hvert billede skal være korrekt eksponeret. Backend&#x27;et skifter automatisk tilbage til reaktiv AE, når DLS-aflæsningen er forældet/mangler, eller kalibreringspakken ikke er indlæst. |
| **Udglatning (α)** | 0,05–1,0, trin 0,05 | 0,3 | Udglatning af på hinanden følgende forudsigelige løsninger (lavere værdi = mere udglattet). |
| **Scenens reflektans**|**Rekalibrer ρ**-knap | — | Genberegner den scenereflektansfaktor, som løsningsprogrammet bruger. |

{% hint style="info" %}
**Array-forbindelse deaktiverer som standard den prædiktive AE** — for arrays håndterer Chloros’s smarte AE samt kameraets automatiske eksponering eksponeringen (med mætningsbeskyttelse), og den prædiktive AE’s enkeltestimat for scenereflektansen er ikke pålidelig på tværs af blandede scener. Du kan genaktivere den pr. kamera her, hvis du specifikt ønsker DLS-styret radiometrisk eksponering.
{% endhint %}

**DAQ-styret eksponeringsloft og indfaldsfastlåst AE.**Uafhængigt af afkrydsningsfeltet ovenfor beregner Chloros — når en DAQ-lyssensor er tildelt et RGB-kamera — ud fra den målte absolutte nedadgående strålingsintensitet — den maksimale eksponering×forstærkning, hvor en overflade med 100 % refleksion forbliver under clipping, og anvender den som et**loft**for den automatiske eksponering. Mens loftet er aktivt, er kameraet**indfaldsstyret**: det kører i åben sløjfe ved den indfaldsmålte eksponering med forstærkning på 0 dB — eksponeringen følger det målte lys, ikke motivets indhold. Da loftet kun kan forkorte eksponeringen, kan det ikke i sig selv forårsage udklipning. Loftet deaktiveres automatisk — og normal motiv-AE genoptages — når DAQ-aflæsningen mangler, er forældet (&gt;30 s), eller mørk, eller hvis ≥15 % af billedet klipper ved den fastlåste eksponering (hvilket betyder, at sensoren og kameraet registrerer forskellig belysning). Der er ingen GUI-kontakt; dette er standardadfærd, når et RGB-kamera har en DAQ-binding.

### Medlemmer af Acquisition &amp; Trigger

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Array viser desuden de skrivebeskyttede rækker **Role**(Master i blåt / Slave i grønt),**Synkroniseringslinje**og**Peers**-rækker.

| Kontrol | Valgmuligheder | Standard | Bemærkninger |
| --- | --- | --- | --- |
| **Trigger-tilstand** | Fra / Til | Til | Deaktiveret for array-medlemmer (arrayet styrer udløsningen). |
| **Triggerkilde** | Software / Linje 0 (M8) / Linje 1 / Linje 2 | Linje 0 | Skjult, når trigger-tilstand er slået fra; deaktiveret for array-medlemmer. Linje 0 er den opto-isolerede eksterne triggerindgang på M8. |
| **Billedhastighed**| Auto / Manuel + værdi | Auto |**Auto**: kameraets begrænsning af billedhastigheden er slået fra — eksponeringen bestemmer fps, og boksen viser den aktuelle live-hastighed.**Manuel**: Du begrænser fps med en skyder (fra 1 op til det båndbreddebegrænsede maksimum), der tager udgangspunkt i den aktuelle faktiske hastighed. Array-medlemmer ser en skrivebeskyttet angivelse på »N fps (live)« med teksten »Indstillet i array-indstillinger«. |

### Netværk / Transport

| Række | Adfærd |
| --- | --- |
| **Pakke størrelse**| 1500 (Standard) / 9000 (Jumbo) — standard**Jumbo**. |
| **Gennemstrømning** | Skrivebeskyttet grænse for linkgennemstrømning i MB/s. Backend-systemet omfordeler denne værdi mellem alle tilsluttede kameraer ved hver tilslutning/afbrydelse. |
| **Bufferhåndtering** | Skrivebeskyttet tilstand for bufferhåndtering. |

### Optagelse

Panelet afsluttes med en **&quot;Åbn optagelsesindstillinger…&quot;**-knap, der fører til [panelet Optagelsesindstillinger](capture.md#the-capture-settings-pane) (deaktiveret, indtil et projekt er åbent — »Opret eller åbn et projekt for at gemme optagelser«). Hvis kameraet er skjult eller sat på pause, minder et tip dig om at vise det igen/genoptage det, før du foretager en optagelse.

## Panelet Array-indstillinger

Åbnes med **tandhjulet**på en ARRAY-række. Overskrift: array-navn med en blyant til omdøbning og**×** til at lukke. Afsnit nedenfor markeret med *kun kombineret* vises kun for arrayer, der er forbundet i kombineret visningstilstand.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Synkronisering

Skrivebeskyttede rækker for **Master**,**Slaves**og**Synkroniseringslinje**.

### Omgivende lyssensor

Vises for både kombinerede og separate arrayer:

* **Kalibreringsmål**-afkrydsningsfelt — &quot;Registrer MAPIR ArUco-mål og valider NDVI i forhold til panelets reflektans-LUT&quot;; styrer den kombinerede flises måloverlay og valideringstabel.
* Rullemenuen **Lyssensor** — knytter én DAQ til hele arrayet. Valget træder i kraft med det samme, overføres til hver enkelt kameras egen rullemenu for lyssensor (du kan stadig tilsidesætte indstillingen for hvert enkelt kamera) og begynder at videresende spektre til arrayet.
* Live **Status**-linje: Fra · &quot;Venter på første spektrum…&quot; · &quot;Aktiv — alle kameraer i arrayet er belysningskorrigeret&quot; · &quot;Intet nyt spektrum i de sidste 3 s — bruger stadig den sidste aflæsning (ingen timeout for forældede data)…&quot;.
* Bemærkning i ruden: &quot;Radiometrisk korrektion for hele arrayet. Indstillinger pr. kamera tilsidesætter dette.&quot;

### Optagelse — ensartede sensorindstillinger *(kun kombineret)*

Disse indstillinger gælder ensartet for alle medlemmer (ændringer pr. medlem ville ødelægge synkroniseringen). Redigeringer forberedes og anvendes samlet.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Kontrol | Valgmuligheder / interval | Hvad den gør |
| --- | --- | --- |
| **Pixelformat** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Ensartet sensorformat for alle medlemmer. |
| **Binning** | 1x1 / 2x2 / 4x4 | Hardware-binning — bevarer det fulde synsfelt, samtidig med at SNR og billedhastighed forbedres. Ændring heraf nulstiller ROI-felterne til det nye fulde synsfelt. |
| **Opløsning** (forudindstilling) | Fuld / Halv / Kvart | Relativt til binning; udfylder ROI-felterne med et centreret udsnit. |
| **ROI-udsnit (px)**| Numeriske felter for B / H / X / Y | Sensorudsnit. Bredde/højde justeres til multipla af 16 (minimum 64); forskydninger justeres til multipla af 4. Et »max BxH«-tip viser den øvre grænse, og**Nulstil** vender tilbage til det fulde synsfelt. Under redigering tegnes der en orange live-forhåndsvisning af beskæringen på array-flisen (inklusive et skema over hele sensoren, når beskæringen udvides udad). |
| **Udløsningshastighed**| Skift mellem Auto / Manuel + fps 0,5–10, trin 0,5 |**Auto**(standard): backendet udleder udløsningshastigheden ud fra opløsning og båndbredde — indtastningsfeltet er deaktiveret og viser den udledte værdi.**Manuel**: fastlåser din værdi ved tryk på Anvend. |

Bemærk i vinduet: »Ændringer af format/opløsning genstarter alle kameraer kortvarigt. Udløsningsfrekvensen gælder med det samme.« Knapperne **Anvend / Annuller** findes nederst i vinduet.

### Justering (co-registrering) *(kun kombineret)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Restfejl**-felt: „RMS x,xx px“ — grøn under 1 px, gul under 3 px, rød i alle andre tilfælde eller hvis et kamera fejlede; „ingen profil“ før den første løsning.
* Oversigtslinje: &quot;✓ Alle kameraer justeret (N)&quot; / &quot;⚠ p/N kameraer justeret —  <serial (filter)="">fejl&quot; / &quot;Beskæring aktiv — Kalibrer igen for at justere (bruger hele sensoren)&quot; / &quot;Venter på, at eksponeringen stabiliserer sig…&quot;.
* Tabel pr. kamera: kamera (de sidste 4 cifre i serienummeret + filter), reprojiceringsfejl i px med antal match („ref“ for masterkameraet) og den normaliserede krydskorrelationsscore for overlapningen i forhold til minimumsgrænsen på 0,35.
* **Rekalibrer justering**-knap (viser &quot;Kalibrer justering&quot; før den første profil) — kører samregistrering igen på nye billeder.
* **&quot;Autoeksponer kameraer til justering&quot;**-afkrydsningsfelt (markeret som standard) — lyser midlertidigt mørke eller flade kameraer op (først eksponering, derefter forstærkning), så de har tekstur, der kan matches, hvorefter AE genoprettes.

Den kombinerede forhåndsvisning justeres automatisk ved åbning; rekalibrer, hvis fokus eller scenedybde ændres. Justeringen er **kun gældende for den aktuelle session** — den gemmes aldrig i en profil, da den afhænger af den aktuelle scenedybde. Optagelser kan stadig eksporteres med pixelregistrering (se [Justerede eksporter](capture.md#per-array-controls)).

### Smart vignette

* **Aktiver korrektion**-afkrydsningsfelt — anvender vignette-estimatet pr. kamera på den radiometriske kæde (live**og** eksport).
* **Kalibrer fra nuværende visning**— ret først kameraopstillingen mod et ensartet mål (fladskærm, væg eller himmel); hvert kamera udjævnes individuelt, og statusrapporterer en udjævningsgevinst på &quot;n/N kameraer · −x,x %&quot;.**Ryd** fjerner estimatet.
* Finjuster for hvert kamera med skyderen **Vignette** pr. kamera i [Live Preview](#live-preview).

### Live Preview *(kun kombineret)** **Indeks**: aktiver afkrydsningsfeltet + tandhjulet — åbner den delte [Indeksberegner](#index-calculator-pane) med bånd tegnet fra**alle** medlemskameraer. En forhåndsvisningslinje for udtryk nedenfor viser det aktuelle udtryk (&quot;Intet udtryk angivet — åbn beregneren for at oprette et&quot;), der opdateres hvert sekund.
* **Renderopløsning**-rullemenu (samme forudindstillinger som pr. kamera, standard 720p): højden på live-view-streamen**og** den gemte størrelse på den sammensatte eksport. Bemærk i panelet: &quot;Forhåndsvisning + størrelse på gemt sammensat billede. Billeder pr. kamera eksporteres altid i fuld opløsning.&quot;

### Visningslag *(kun kombineret)** **Aktiver**-afkrydsningsfelt (standard: fra — hovedkameraet vises direkte; til = lagdelt sammensætning).
* **Forgrund**/**Baggrund**-rullemenuer: hvert medlemskamera (efter navn) eller**Indeks**. Når Forgrund er Indeks, viser pixels uden for LUT Min/Max baggrundslaget.

### Delt visning *(kun kombineret)*

**&quot;Vis medlemskameraer&quot;**— en**Opdel / Skjul medlemskameraer**-knap, der tilføjer hvert medlems eget livefeed som separate gitterfelter ved siden af sammensætningen. Felterne læser arrayets eksisterende billedbuffer (ingen ekstra kameratilslutning). Kun gittervisning; gemmes pr. array sammen med projektet.

### Funktioner

Et skrivebeskyttet panel, der opdateres hvert 5. sekund:

* **Niveaumærke**: &quot;Samtidig optagelse&quot; (grøn) · &quot;Samtidig optagelse (FTD-forskudt udsendelse)&quot; (grøn) · &quot;Forskudt optagelse (100 ms afvigelse)&quot; (gul) · &quot;Konfigurationen er for stor&quot; (rød).
* **Rammehelbred**: &quot;x,xx % ufuldstændig&quot; — grøn under 1 %, gul under 5 %, rød ved 5 % eller mere.
* **Forbindelseslinje**: &quot;NIC {mbps} Mbps – vedvarende {MB/s} MB/s&quot;.

Dette er arrayets aktuelle båndbreddebudget. For den underliggende fps og netværksmodel — og hvad der skal ændres, når niveauet skifter til gult eller rødt — se [Multi-kamera-arrayer](arrays.md) og [CLI-referencen](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Vinduet &quot;Indeksberegner&quot;

Den tredje side i sidepanelet, som deles af indeks-værktøjet pr. kamera og indeks-værktøjet for det samlede array (ét ad gangen – overskriften lyder »Indeksberegner – <camera name="">« eller »Indeksberegner –<array name="">

«). Den modtager båndlisten (kameraets filter-naturlige bånd eller alle bånd på tværs af arrayets medlemmer), det aktuelle udtryk og LUT-konfigurationen (til/fra, niveau — standard 3, min — standard 0,2, maks — standard 1) samt et live-indeks-histogram. **Anvend** bekræfter udtrykket; LUT-ændringer anvendes live på forhåndsvisningen.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Indstillinger pr. kamera vs. array-styrede indstillinger

Hurtigoversigt over, hvad der findes hvor, når et kamera er medlem af et array:

| Array-styret (skrivebeskyttet i kamerapanelet) | Stadig pr. kamera inden for et array |
| --- | --- |
| Pixelformat, opløsning, binning | Autoeksponering (eksponering, forstærkning, mål, udjævning, ROI) |
| Udløsertilstand/kilde, billedhastighed | Støjfjernelse, skarphed, vignettering |
| | Orientering (spejling/rotation), skærmoverlejringer, spotmåler |
| | Indeks (arrayer med separat visning), binding til lyssensor |

Anden tværgående adfærd:

* **Kombineret vs. separat visning** vælges ved tilslutning af arrayet: kombineret = én justeret sammensat flise (elementer vises kun via Split View); separat = hvert element gengiver sin egen synkroniserede flise. Et kamera viser aldrig både et selvstændigt feed og en array-flise.
* **Automatisk genforbindelse**: Når man åbner et gemt projekt, gendannes kameraerne og arrayerne, og alle gemte indstillinger anvendes igen på backend, før streamene genoptages.
* **Optagelsesbegrænsning**: skjulte eller pausede kameraer udelades fra »Capture All«; et array blokeres fuldstændigt kun, når ALLE medlemmer er skjulte/pausede. Se [Optagelsesindstillinger og -tilstande](capture.md).

## Sådan gemmes indstillingerne

Kamerafanens tilstand gemmes **sammen med projektet**, ikke i browseren:

* Hver reaktiv ændring tager et øjebliksbillede af kameraerne og arrayerne i projektets `cameras.json` (debounced 500 ms). Dette omfatter kameranavne og farver, eksponering/forstærkning/AE-indstillinger, pixelformat/opløsning/binning, udløserhastighed, forhåndsvisningsindstillinger (renderopløsning, støjreduktion, skarphed, vignettering, farveprofil, mætning/kontrast), orientering, overlejringer, kanalopdelinger, indeks-konfiguration, indstillinger for prædiktiv AE, AE-ROI, array-navne, visningstilstand, indstillinger for array-optagelse (herunder ROI-beskæringsposition) samt gitterblokken (feed-zoom, visningstilstand, gitterlås, manuel fliserækkefølge, skjulte kameraer, lukkede fliser, aktivt kamera).
* Lyssensorbindinger gemmes i projektets `sensors.json`.
* Når projektet genåbnes, genoprettes forbindelsen til hardwaren, og alle indstillingerne anvendes på ny.
* **Intet projekt åbent = kun session**: uden et projekt bevares intet, når Chloros lukkes.
* Kun session uanset projekt: pausetilstand, spotmålerprøver, afkrydsningsfeltet »Kalibreringsmål pr. kamera« (er altid slået fra ved opstart) og array-justeringsprofilen (genberegnes pr. session som standard).
* En undtagelse: Valg ved eksport af **optagelsesindstillinger** og optagelsestilstanden bevares pr. projekt i den lokale app-lagerplads i stedet for i `cameras.json` — se [Optagelsesindstillinger og -tilstande](capture.md).</array></camera></serial>

# Kameraindstillinger

Fanen **Kameraer** er Chloros’ live-kontrolpanel til LATTICE-kameraer: et hovedområde, der viser alle tilsluttede kameraer som live-felter, samt en sidebjælke, der skifter mellem tre sider — **kameralisten**, et**indstillingspanel**(indstillinger pr. kamera, for kameraopstillinger eller optagelsesindstillinger — én ad gangen) samt**Indeksberegneren**. Denne side beskriver alle kontrolelementer i kameralisten, indstillingspanelet pr. kamera og indstillingspanelet for kameraopstillinger. Optagelsestilstande, valg af eksporttype og »Capture All«-forløbet findes på den tilhørende side [Optagelsesindstillinger og -tilstande](capture.md).

Fanen »Kameraer« vises i sidepanelet, så snart Chloros-backend er klar. Alle nedenstående kontrolelementer kommunikerer med det lokale backend via `127.0.0.1:5000`; ændringer gælder straks for live-kameraet, medmindre andet er angivet.

## Kameratyper, der anvendes på denne side

Kontrolelementerne vises eller skjules afhængigt af, hvilken type kamera der er valgt. I denne vejledning anvendes følgende termer gennemgående:

| Term | Betydning | Filterkanaler |
| --- | --- | --- |
| **RGB-kamera** | LATTICE M3C med FRGB-filter (modellen indeholder `-FRGB`) | Red / Green / Blue |
| **Bayer-multispektralt** | LATTICE M3C med FRGN, FOCN eller FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — ét smalbåndsfilter, ét kalibreret bånd | Enkeltbånd |
| **Array-medlem** | Et kamera tilsluttet som del af et synkroniseret array (kombineret eller separat visning) | I henhold til dets filter |

 RGB-kameraer gennemgår fotometrisk behandling (hvidbalance, farveprofiler, gamma); multispektrale og mono-kameraer gennemgår den radiometriske kæde og springer de fotometriske indstillinger over. Array-medlemmer overfører indstillinger på stream-niveau (pixelformat, opløsning, binning, trigger, billedhastighed) til arrayet — disse rækker bliver skrivebeskyttede i panelet pr. kamera og flyttes i stedet til panelet med arrayindstillinger.

## Hovedfeedområdet

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->



Når der ikke er tilsluttet kameraer, viser feedområdet en velkomstskærm med teksten **"Tilslut et kamera for at komme i gang"**med to knapper:**Tilslut kamera**(grøn, åbner dialogboksen til tilslutning af et enkelt kamera) og**Tilslut array** (blå, åbner dialogboksen til tilslutning af et array). Selve tilslutningsdialogboksene er beskrevet i [Tilslutning af kameraer](connecting.md); array-begreber (synkronisering, lag, båndbredde) i [Kameraopstillinger med flere kameraer](arrays.md). Når du åbner et gemt projekt, der indeholder kameraer, viser startskærmbilledet i stedet en spinner med teksten "Genåbner N gemte kameraer…", mens Chloros gendanner streamene fra den sidste session.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Øverste bjælke

| Kontrol | Funktion |
| --- | --- |
| **Skift mellem visningstilstande**| Skifter mellem**gittervisning**(alle felter som celler) og**listevisning**(rækker i fuld bredde: hvert kombineret array som én sammensat række, hvert medlem af et**separat** array som sin egen række med sit eget feed og sin egen kalibreringsmåltabel, og derefter ÉT enkeltstående kamera — det aktive — nedenfor). Værktøjstip: »Skift til gittervisning« / »Skift til listevisning«. |
| **Gitterlås**(hængelås) | Standard**låst** — felter fastlåst på plads. Lås op for at trække og omarrangere felter til en vilkårlig plads (mellemrum bevares). Gitteret låses automatisk igen, hver gang et nyt kamera opretter forbindelse. Værktøjstip: »Lås gitter op (aktiver trækning af felter)« / »Lås gitter (fastlås felter på plads)«. |
| **Feed-zoom**-skyder | Felstørrelse, fra 60 px op til den fulde bredde af rammen. Cellerne bevarer et 4:3-format. Når cellebredden er under 200 px, skjules navn og fps-overlays for at holde flisen ren. |

### Feed-fliser

Hvert kamera gengiver en sammensat live-flise; et kamera kan desuden vise tre gråtonede **kanalopdelte** fliser (se [Kanalopdelinger](#display-overlays-drawn-over-the-live-feed)), og arrayer gengiver en samlet flise. Den aktive flise har en markeringsring i kameraets (eller arrayets) farve.

Når man holder musen over en flise, vises en **X**-lukkeknap:

* Hvis du lukker en **sammensat** flise, mens dens kanalopdelinger stadig er synlige, skjules blot den sammensatte flise.
* Hvis du lukker den **sidste synlige flise fra et enkeltstående kamera**, afbrydes forbindelsen til det pågældende kamera.
* **Opdelte fliser fra et kombineret array afbryder aldrig** kameraet — de skjules blot.

Når gitteret er låst op, kan du trække en vilkårlig flise til en vilkårlig plads; layoutet gemmes sammen med projektet.

## Sidepanel — kameraliste

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

Den første side i sidepanelet viser en liste over alle tilsluttede kameraer og arrayer:

* **Tilslut kamera**(grøn) /**Tilslut array** (blå, viser "Registrerer..." under scanning). Begge er deaktiveret, mens en tilslutningsdialog er åben.
* **Optag alt** (rød) — optager alle kameraer på listen med de eksporttyper, der er valgt i Optagelsesindstillinger. Kræver et åbent projekt. Fuldt dokumenteret i [Optagelsesindstillinger og -tilstande](capture.md).
* **Tandhjul for optagelsesindstillinger** (ved siden af Optag alt) — åbner [panelet Optagelsesindstillinger](capture.md#the-capture-settings-pane). Deaktiveret uden et projekt eller under optagelse.

### Kamerarækker

Hver kamerarække viser en farvekodet ramme (kameraets brugerdefinerede farve), en "CAM"-mærkat — med et blåt **M**(master) eller et grønt**S** (slave) som rollebogstav for array-medlemmer — samt visningsnavnet. Standardnavnet er `LATTICE-MODEL (serial)`; omdøb det fra indstillingspanelet for det enkelte kamera. Række-knapper:

| Knap | Effekt |
| --- | --- |
| **Øje**| Skifter mellem synlighed. Skjulte kameraer forlader gitteret og**udelades fra Capture All**. |
| **Tandhjul** | Åbn indstillingspanelet for det enkelte kamera (næste afsnit). |
| **Pause / Afspil**| Fryser live-forhåndsvisningen**kun på skærmen** — backend-optagelsen fortsætter. Kameraer i pause kan ikke optage. |
| **X** | Afbryd forbindelsen. Brugergrænsefladen opdateres straks (i bedste fald); selve afbrydelsen i backend kan tage 10–30 sekunder at gennemføre. |

### Array-rækker

En array-række viser et "ARRAY"-mærke i arrayets farve, arrayets navn (kan omdøbes i array-indstillingerne) og en **DAQ · til/fra**-knap —**til**, når lyssensoren på array-niveau er aktiveret *eller* et hvilket som helst medlem har en sensor pr. kamera; dens værktøjstip viser præcist, hvilken sensor der leverer signaler til hvad. Medlemskameraerne vises indrykket nedenunder med deres egne rækker. Array-række-knapper: **øje**(skjuler/viser ALLE medlemmer samlet),**tandhjul**(array-indstillingspanel),**X**(afbryder forbindelsen til hele arrayet).

Lyssensorens (DLS) status, der bruges i array-rækker og i array-indstillingspanelet, har fire tilstande:**slukket**,**venter**(intet spektrum endnu),**aktiv**(et spektrum er ankommet inden for de sidste 3 sekunder) og**forældet** — intet nyt spektrum i 3 sekunder, men den sidste aflæsning *bruges stadig* (DAQ-målinger udløber aldrig på indlæsningsstien).

Indtil du ændrer rækkefølgen, viser listen først arrayer, derefter enkeltstående kameraer, hvor kameraerne (og hvert arrays medlemmer) er i **kamerarækkefølge**: M3C (Bayer) før M3M (mono); blandt M3C, FRGB, FRGN, FOCN og derefter FNGB; M3M efter filterbølgelængde, den laveste først. Gitterets standardlayout følger den samme rækkefølge. Du kan trække enkeltstående kameraer og hele arraygrupper forbi hinanden i sidepanelet for at omarrangere listen; array-medlemmer kan ikke trækkes uafhængigt af hinanden.

## Indstillingsrude pr. kamera

Åbnes med **tandhjulet** på en kamerarække. Ruden glider hen over kameralisten.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Overskrift**: kameraets**farveprøve**(klik for at åbne en indbygget farvevælger — indstiller farven på sidebjælkens kant og flisernes markeringsring),**navnet**med en blyantknap til**Omdøb**(hvis du gemmer et tomt navn, vender det tilbage til standardnavnet `MODEL (serial)`) og**×** for at lukke.

### Live-histogram

Øverst i ruden vises et live-luma-histogram, der beregnes ud fra JPEG-forhåndsvisningen med en frekvens på ca. 8 Hz. Gennemsnittet er Bayer-vægtet — (R+2G+B)/4 — for at matche kameraets egen AE-lysmåling.

* **Den Orange, stiplede linje**= AE-målet.**Træk den vandret for at ændre målet** — der sendes en kommando, når du slipper, og ved at trække skifter AE-mål-tilstanden til Manuel.
* **Den Green, fuldt optrukne linje** = den faktiske gennemsnitlige luma (det, som AE leverer i øjeblikket).
* **RGB-knap** (øverst til højre): skifter mellem båndvise overlejrede histogrammer farvet efter kameraets filter (f.eks. på FRGN: gråt NIR, grønt, rødt). På monokameraer (M3M) står der »MONO« på knappen, og den er deaktiveret — mono viser altid det enkeltbåndede luma-histogram.
* Mærkaterne på X-aksen følger sensorens bitdybde for det aktuelle pixelformat: 0..255, 0..1023, 0..4095 eller 0..65535.

### Rækker

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

med kameraoplysninger

| Række | Adfærd |
| --- | --- |
| **Model** | Skrivebeskyttet (f.eks. `LATT-M3C-L87-FRGN`). |
| **Radiometrisk kalibrering** | Green **"Aktiv"**-mærke med en billedtekst, der viser kalibreringsniveau, hash, kalibreringsdato og båndliste, indlæst fra kameraets kalibreringspakke (se [Fabriksradiometrisk kalibrering](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Skjult for RGB-kameraer** — disse har en fotometrisk hvidbalancekalibrering, ikke strålingsintensitet pr. bånd. |
| **Kalibreringsrapport**|**Download**-knap — åbner kameraets NIST-kalibreringscertifikat (pr. serienummer) som PDF i din operativsystems visningsprogram. Hvis certifikatet endnu ikke er gemt i cachen, viser Chloros i stedet et tip. |
| **Serienummer** | Skrivebeskyttet. |
| **Firmware**| Viser den aktuelle version og finder derefter den tilgængelige version til denne model (gemt i cachen pr. model — en gruppe på N kameraer tjekker serveren én gang). Status: „Tjekker…“ →**„Opdater til X“**-knap → „Oplader…“ → „Opdateret fra A til B“ / „Mislykkedes: …“ / "Springet over: …" / grøn**"Opdateret"**. Værktøjstip til opdateringsknappen: "Fabriksindstilling + flash + omprogrammering af UserSet1. Ca. 2–3 minutter; afbryd ikke forbindelsen." |
| **IP** | Skrivebeskyttet. |
| **Temperatur** | Skrivebeskyttet, opdateres hvert 3. sekund. Bliver orange ved ≥65 °C og rød med et ⚠ ved ≥75 °C. |
| **Kalibreringsmål** afkrydsningsfelt | Aktiverer ArUco-reflektansmåldetektion med en NDVI-valideringstabel pr. panel under live-feedet (listevisning) — se [Tabellen med kalibreringsmål](#the-calibration-target-table). Gælder kun for den aktuelle session — åbnes altid i slukket tilstand. |
| **Lyssensor**-rullemenu | Tilknytter en DAQ-lyssensor (DAQ-E/M/U, fra listen under fanen »Lyssensorer«) til dette kamera til korrektion af nedadgående lys (DLS) og forudsigelig automatisk eksponering. "”Ingen” fjerner tilknytningen. Hvis der ikke er tilsluttet nogen sensorer, viser rullemenuen ”(ingen sensorer tilsluttet — åbn fanen DAQ)”. Tilknytningen gemmes sammen med projektet. |

#### Tabellen »Kalibreringsmål«

Tabellen er **kun til valideringsformål** ("kun validering — ændrer ikke kalibreringen"). Overskriften angiver kameraets serienummer, filter, målnummer og markørstørrelse i pixels, og den har én række pr. panel:

* **Panel**, derefter én kolonne pr. bånd med panelets værdi — et Bayer-kamera viser sine tre bånd efter navn og bølgelængde (f.eks. FRGN: Red 660 nm / Green 550 nm / NIR 850 nm); et **mono (M3M)-kamera viser sit eneste bånd** (f.eks. `F750`) og har ingen NDVI-kolonner.
* **Sat** — andelen af panelets pixels, der er mættede (gult fra 1 %, rødt fra 5 %).
* **NDVI Meas**,**NDVI Theo** (panelets reference-reflektans-NDVI) og **Δ**— kun Bayer-multispektralkameraer. Et lille tegn efter »NDVI Meas« angiver, hvad den målte NDVI er beregnet ud fra:**ρ**(reflektans — der foreligger en lyssensoraflæsning),**L**(radiance — ingen lyssensorværdi) eller**DN**(rå kameraværdier). Kun en**ρ**-NDVI kan sammenlignes med NDVI Theo: for L og DN er Δ-kolonnen nedtonet, og en note angiver dette (»Ingen lyssensorværdi: NDVI er baseret på strålingsintensitet, så Δ er ikke en kalibreringskontrol« / »NDVI er baseret på rå kameraværdier, så Δ er ikke en kalibreringskontrol«). Med ρ er Δ grøn under 0,02, gul under 0,05 og rød i alle andre tilfælde.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Eksponering og forstærkning

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Alle numeriske indtastninger her bruger drejeknapper, der accelererer, når man holder dem nede: tryk = ±1, hold >1,5 s = ±10, hold >3 s = ±100. Værdien sendes til kameraet, når du slipper.

| Kontrol | Interval / valgmuligheder | Standard | Gælder for | Hvad den gør |
| --- | --- | --- | --- | --- |
| **Eksponering (us)**| Kameraets live min./maks. | Auto | Alle | Eksponeringstid i mikrosekunder, med en**Auto/Manuel**vælger. Auto = kontinuerlig automatisk eksponering på kamerasiden. Hvis du indstiller en manuel eksponering, forbliver**Gain** på Auto, hvis den var det, og du kan frit bruge kameraets fulde forstærkningsområde, indtil du redigerer forstærkningen eller indstiller den til Manuel. |
| **Forstærkning (dB)**| Kameraets aktuelle min./maks. (f.eks. op til 48 dB) | Manuel (slået fra) | Alle | Analog/digital forstærkning med sin egen**Auto/Manuel**-knap. |
| **AE-mållysstyrke**| 0–255 | 80, tilstand**Auto**| Alle (kan redigeres, når AE eller automatisk forstærkning er aktiveret) | Den lysstyrke, som AE sigter mod. I**Auto**(standardindstillingen) vælger en histogrambaseret bagvedliggende controller selv målet og holder eksponeringen på 60–75 % af sensorens maksimum. Hvis du indtaster en værdi eller trækker i histogrammets orange linje, skifter den til**Manuel**. |
| **AE-udjævning** | 0,5–40, trin 0,1 | 8,0 | Alle | AE-dæmpning. Værktøjstip: "Lavere = AE reagerer hurtigere (kan pulsere ved høje fps). Højere = jævnere / langsommere." Værdier langt under standardindstillingen kan få AE til at pulsere og destabilisere streamingen ved høje billedhastigheder; 8,0 er den stabile standardindstilling. |
| **AE-interesseområde**| Afkrydsningsfelt for aktivering +**Sigt**-knap | Fra | Alle | Når denne funktion er aktiveret, måler AE kun det grønne, stiplede område i stedet for hele billedet.**Sigt** aktiverer »klik-for-at-placere« på livefeedet: et klik centrerer et område ved 30 % af billedet; klik-og-træk tegner et brugerdefineret rektangel (minimum 5 % × 5 %). *Aim* deaktiveres automatisk efter én placering. Området kortlægges tilbage til kameraets oprindelige koordinater i henhold til enhver rotation/spejling, du har indstillet, og gemmes sammen med projektet. |
| **AE Tune Speed** | 0,1–5, trin 0,1 | 1,0 | Kun for array-medlemmer | Hvor hurtigt det automatiske AE-mål sporer ændringer i scenens lysstyrke; 1,0× kontrollerer igen hvert 2,5. sekund. |
| **Højlysbeskyttelse** | Streng (1 %) / Normal (5 %) / Lempet (15 %) | Streng | Kameraer, der eksponerer indstillingen | Hvor stor en del af billedet der må klippes til hvidt, før AE mørkner billedet. |

{% hint style="info" %}
**Lyskrav til Bayer-multispektralkameraer (RGN / OCN / NGB):** Motivet skal have tilstrækkeligt lys i alle tre kanaler, ellers fungerer kalibreringen ikke korrekt — en enkelt sensoreksponering dækker alle tre spektre. Brug en DAQ-lyssensor til at måle lyset, eller skift til fuld mono (M3M), så hvert bånd får sin egen eksponering. Hvis en optagelse overtræder dette, registrerer Chloros det og advarer dig (unmix-clamp-meddelelsen). I realtid viser en kanalopdelt flise **»Banddata med lav pålidelighed · N %«**, når unmix har begrænset det pågældende bånd på 25 % eller flere af dets pixels.
{% endhint %}

{% hint style="info" %}
**Sådan fordeler autoeksponeringen eksponering og forstærkning (1.3.0).** Når forstærkningen er sat til "Auto", holder Chloros forstærkningen på 0 dB, indtil eksponeringen har nået den længste tilladte værdi (AE-øvre grænse, et arrays triggerperiode eller DAQ-eksponeringsloftet); først derefter stiger forstærkningen. I et array kommer triggerfrekvensen først: autoeksponeringen udfylder den periode, som frekvensen angiver, hvorefter forstærkningen hæves, så en mørkere scene ikke længere sænker en automatisk triggerhastighed (i 1.2.0 gjorde den det). En **manuel**eksponering på et array-medlem, der varer længere end triggerperioden, sænker arrayets triggerhastighed for at tilpasse sig den; en automatisk frekvens genoprettes, når hvert element, der bremsede den, er tilbage på automatisk eksponering. Når både**Eksponering**og**AE-mållysstyrke** er sat til Auto, og en DAQ-lyssensor leverer en måling, holdes hvert kameras eksponering fast, når den automatiske eksponering har stabiliseret sig, så længe det målte lys forbliver inden for ca. ±25 %; en større, vedvarende lysændring, en genstart af indstillingerne (f.eks. en ændring af ROI) eller din egen ændring af eksponeringen aktiverer den igen.
{% endhint %}

{% hint style="info" %}
**Sådan fungerer automatisk eksponering (1.3.0).**

* Ved manuel eksponering forbliver forstærkningen på Auto, indtil du ændrer forstærkningen.
* Små højlys gør ikke scenen mørkere: en spejling på op til 1 % af billedet ignoreres, og i panelerne på et kendt kalibreringsmål ignoreres op til 0,5 % af et panel.
* DAQ-eksponeringsloftet gælder kun, mens et kalibreringsmål er i billedet; hvis der ikke ses noget panel, eksponerer kameraet for scenen.
* Eksponeringen forbliver stabil ved konstant lys og stabiliserer sig inden for få sekunder efter en stor ændring. En overskydning begrænser ikke længere eksponeringen permanent, og en ændring af ROI finder eksponeringen igen.
* Lysstyrkemålet overskrider ikke kameraetsrækkevidde, så et tilbagevendende lys ikke forårsager blænding i billedet.
* Et kamera, hvis automatiske eksponering holder op med at reagere i stærkt sollys, holdes på det niveau, der passer til det lys, det befinder sig i – målt af lyssensoren eller, hvis der ikke er tilsluttet en, af kameraets egne billeder – og slippes, når lyset ændrer sig. En mørk eller tildækket lyssensor forstyrrer ikke dette.
{% endhint %}

### Pixelformat og opløsning

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Array-medlemmer** viser skrivebeskyttede rækker med "Aktuel" (format + BxH) og "Binning" med bemærkningen "Indstilles i array-indstillinger" — en genstart af streamen på et medlem ville bryde synkroniseringen, så disse administreres i [array-indstillingspanelet](#array-settings-pane).**Fritstående kameraer** har:

| Kontrol | Valgmuligheder | Hvad det gør |
| --- | --- | --- |
| **Pixelformat** | 8-bit / 10-bit / 12-bit / 16-bit | Sensorens bitdybde; formatfamilien er sensorens. Standardindstillingen er 12-bit, og et projekt gemmer den indstilling, du vælger, inklusive 8-bit, og genopretter forbindelsen med denne indstilling. |
| **Opløsning** | Fuld / Halv / Kvart | I forhold til den aktuelle binning: Fuld = 2048/N × 1536/N for N×N-binning. |
| **Binning** | 1x1 (ingen) / 2x2 / 4x4 | Hardware N×N-binning — større værdier sænker opløsningen, men øger SNR og billedfrekvensen. Ændring af indstillingen genstarter streamingen og nulstiller eventuelle ROI'er til det nye fulde synsfelt. |
| **Aktuel** | skrivebeskyttet | Den faktiske WxH og (x, y)-forskydning, der er aktiv. |

### Live-forhåndsvisning

Alt i dette afsnit gælder **kun på skærmen**— det ændrer det, du ser i live-feedet, mens gemte optagelser forbliver lineære og uændrede — med én undtagelse:**Vignette** er radiometrisk og påvirker også eksportfiler (beskrevet nedenfor).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Kontrol | Interval / valgmuligheder | Standard | Gælder for | Hvad den gør |
| --- | --- | --- | --- | --- |
| **Renderingsopløsning** | 360p (hurtigst) / 480p / 720p / 1080p / Sensorens oprindelige opløsning (langsomst) | 720p | Alle | Den højde, hvorpå backend kører den radiometriske forhåndsvisningskæde. En lavere indstilling giver højere billedfrekvens uden at ændre synsfeltet. |
| **Indeks**| Afkrydsningsfelt „Aktiver“ + tandhjul | Fra | Kun Bayer-multispektral,**ikke** medlemmer af kombinerede arrays | Live forhåndsvisning af vegetationsindeks. Tandhjulet åbner det delte [Indeksberegner](#index-calculator-pane), der er forudindlæst med kameraets filter-naturlige bånd (f.eks. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Det brugerdefinerede udtryk plus LUT (til/fra, standardniveau 7-stop, standardmin. 0,2, standardmaks. 1) beregnes for hvert forhåndsvisningsbillede. Medlemmer af den kombinerede matrix skjuler denne række — matrixen har ét fælles indeks. |
| **Hvidbalance** | Fra / En gang / Kontinuerlig + en genoptagelsesknap | Kontinuerlig | Kun RGB | Live hvidbalance. Opdateringsknappen genoptager hvidbalancen fra det aktuelle DLS-spektrum (deaktiveret, når tilstanden er Fra). |
| **Gamma** | Til / Fra | Til | Kun RGB | Vis gamma (γ = 2,2 LUT) i live-forhåndsvisningen. Gemte optagelser forbliver lineære. |
| **Støjfjernelse** | Afkrydsningsfelt + styrke 0–100 | Fra / 50 | Alle (pr. kamera, også inden for arrayer) | Bilateralt filter i live-forhåndsvisningen. Højere værdi = glattere, men blødere detaljer. |
| **Skarphed** | Afkrydsningsfelt + styrke 0–100 | Fra / 30 | Alle | Unsharp mask i live-forhåndsvisningen, anvendes som det sidste. Kan forstærke støj. Kun i forhåndsvisningen. |
| **Vignettering**| Afkrydsningsfelt + styrke 0–100 | Fra / 0 | Alle | Manuel fjernelse af resterende vignettering (lysner hjørnerne), lagt oven på arrayets Smart Vignette-estimering.**Radiometrisk — påvirker livevisningen OG eksporten**, i modsætning til Støjfjernelse/Skarphed. |
| **Farveprofil** | Raw / Lineær / Naturlig / Forbedret / Brugerdefineret temperatur | Naturlig | Kun RGB | Se nedenfor. |
| **Farvetemperatur** | 2000–10000 K, trin 100 | 5500 K | Kun RGB, brugerdefineret temperaturprofil | Fastlåser hvidbalancen til en fast korreleret farvetemperatur (DLS-indgang ignoreres). Den senest valgte Kelvin-værdi huskes på tværs af profilskift. |
| **Højlys** | −100 til 0, trin 1 | 0 (slået fra) | Kun RGB | Nyt i 1.3.0. Mørkner de lyseste, hvideste områder af billedet (0 = uændret, −100 = stærkest). Gælder for live-forhåndsvisning **og for optagelser på skærmniveau (Preview)**, som gemmer indstillingen (`Chloros:DisplayHighlights`); eksport af strålingsintensitet og reflektans påvirkes aldrig. Det kan ikke gendanne sensordata, der er klippet — sænk eksponeringen for at undgå dette. Zebra-overlayet følger det mørkere billede. Deaktiveret med Raw-farveprofilen (»Anvendes ikke med Raw-farveprofilen.«). |
| **Mætning** | 0–200 (100 = neutral) | 100 | Kun RGB | HSV-mætning i live-forhåndsvisningen. |
| **Kontrast** | 0–200 (100 = neutral) | 100 | Kun RGB | Lineær kontrast omkring mellemgrå i live-forhåndsvisningen. |
| **Spejl H / Spejl V** | Afkrydsningsfelter | Fra | Alle | Spejl forhåndsvisningen vandret / lodret. |
| **Rotation**| 0° / 90° / 180° / 270° | 0° | Alle | Roter forhåndsvisningen. Retningen anvendes i slutningen af backend-forhåndsvisningskæden.**Gemte optagelser (1.3.0):**Forhåndsvisning, Radiance, Reflectance og Index-produkter fra et enkeltstående kamera vendes ved hjælp af Spejling/Rotation, mens**Raw og Debayered forbliver i kameraets oprindelige orientering**; hver optagelsesfil gemmer indstillingen i sin XMP, så bearbejdning af en raw-fil senere anvender den på samme måde.**Filer i et array forbliver i deres oprindelige orientering** (indstillingen gemmes, men anvendes ikke), og sammensatte visninger af arrayet ignorerer den. |

**Farveprofil-semantik** (RGB-kameraer):

* **Raw** — omgår behandlingskæden fuldstændigt.
* **Lineær** — mørkt signal + flat-field + hvidbalance; ingen farvematrix, ingen gamma.
* **Naturlig** *(standard)* — lineær plus den målte farvekorrektionsmatrix og en scenetilpasset tonekurve.
* **Forbedret**— Naturlig plus vibrance og CLAHE-lokal kontrast. Den ekstra omkostning gælder**kun live-forhåndsvisning** — gemte optagelser får altid den fulde efterbehandling uanset profil.
* **Brugerdefineret temperatur** — Naturlig med hvidbalance fastsat til den valgte Kelvin-værdi.

{% hint style="warning" %}
For *Naturlig*, *Forbedret* og *Brugerdefineret temperatur* viser panelet en tonebemærkning: billederne lysnes op i forhold til deres egen scene, så gemte *skærmbilleder* ikke kan sammenlignes fra billede til billede. **Eksporter radiance eller reflektans til målinger.**
{% endhint %}

### Visningsoverlejringer (tegnet oven på live-feedet)

Disse findes kun i frontend — de lægges oven på videoen og påvirker aldrig streamen eller optagelserne.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Overlejring | Kontroller | Standard | Hvad den gør |
| --- | --- | --- | --- |
| **Zebra** | Afkrydsningsfelt + tærskel 200–255 | Fra / 250 | Magenta diagonale striber på beskårne pixels. |
| **Sigtemærke** | Afkrydsningsfelt | Fra | Markering af billedrammens midte. |
| **Gitter** | Fra / 3 × 3 / 9 × 9 | Fra | Kompositionsgitter. |
| **Histogram** | Afkrydsningsfelt + bredde 0,10–0,90 af billedet | Fra / 0,25 | En histogramstribe på feedet. |
| **Fokuspeak** | Afkrydsningsfelt + tærskel 20–200 + farveprøve | Fra / 80 / `#ff5722` | Sobel-kantfremhævning til fokusering. |
| **Kanalopdelinger** | "Vis opdelinger (Red / Green / NIR)" / "Skjul opdelinger"-knap | Skjult | Tilføjer tre uafhængige gråtonekort pr. kanal ved siden af det sammensatte billede (knapteksten følger kameraets filterkanaler). Hvert opdelt kort kan trækkes og har samme kantfarve som kameraet. Ikke tilgængeligt på monokameraer. Gemmes sammen med projektet. |

### Spotmåler

* **Klik for at måle**-afkrydsningsfelt: Klik på livebilledet for at måle en enkelt pixel (et sigtekorn markerer den), eller klik og træk et område for at få et pixelgennemsnit.**Ryd**sletter målingen og sigtekornet. Kan ikke bruges sammen med AE-ROI**Aim**-tilstanden.
* **Vis** -rullemenu:**Raw (bitdybde)**— oprindelige digitale tal i sensorens bitdybde (f.eks. 12-bit → 0..4095) — eller**Display (8-bit)** (standard). Når et live-indeks er aktivt, viser Display i stedet den beregnede indeksværdi (f.eks. NDVI).
* Aflæsningspanelet viser pixelkoordinater, billedstørrelse, pixelformat, bitdybde og en kanaltabel (Chan / Værdi / %) med båndbetegnelser og bølgelængder; Bayer-grønpar er gennemsnitsberegnet; regionprøver viser »N px avg«.

Spotmålerens tilstand gælder kun for den aktuelle session.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Forudsigelig automatisk eksponering (DLS-styret)

Dette afsnit vises kun, når **mindst én DAQ-lyssensor er tilsluttet** — beregningsmodulet har brug for et live nedadgående spektrum for at kunne styre det.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Kontrol | Område | Standard | Funktion |
| --- | --- | --- | --- |
| **Aktiver** | Afkrydsningsfelt | Til (fritstående kameraer) | En lukket løsningsalgoritme bruger DLS-spektret samt skalærerne fra kameraets kalibreringspakke til at placere det lyseste bånd tæt på mætning, samtidig med at det mørkeste bånd holdes over SNR-grænsen — én eksponering pr. løsning, ingen stabiliseringsloop. Designet til solcelledrevne timelapse-optagelser, hvor hvert billede skal være korrekt eksponeret. Backend’et skifter automatisk tilbage til reaktiv AE, når DLS-målingen er forældet/mangler, eller kalibreringspakken ikke er indlæst. |
| **Udglatning (α)** | 0,05–1,0, trin 0,05 | 0,3 | Udglatning af på hinanden følgende forudsigelige løsninger (lavere = glattere). |
| **Scenens reflektans**|**Rekalibrer ρ**-knap | — | Genberegner den scenereflektansfaktor, som solveren bruger. |

{% hint style="info" %}
**Array-tilslutning deaktiverer som standard den prædiktive AE** — for arrays håndterer Chloros' smarte AE samt kameraets automatiske eksponering eksponeringen (med mætningsbeskyttelse), og den prædiktive AE's estimat af en enkelt scenes-refleksionsskøn er ikke pålideligt på tværs af blandede scener. Du kan genaktivere den pr. kamera her, hvis du specifikt ønsker DLS-drevet radiometrisk eksponering.
{% endhint %}

**DAQ-drevet eksponeringsloft og indfaldsfastlåst AE.** Uafhængigt af afkrydsningsfeltet ovenfor beregner Chloros — når en DAQ-lyssensor er tildelt et RGB-kamera — ud fra den målte absolutte nedadgående irradians den maksimale eksponering×forstærkning, hvor en overflade med 100 % reflektans forbliver under clipping, og anvender den som et **loft**for den automatiske eksponering. Siden version 1.3.0 gælder loftet kun, mens et kalibreringsmål er i kameraets synsfelt (set i billedet inden for de sidste 10 sekunder); når der ikke er noget mål i synsfeltet, kører kameraets almindelige automatiske eksponering i stedet med sin egen højlysbeskyttelse, så resten af scenen ikke holdes mørkere, end den behøver at være. Mens loftet er aktivt, er kameraet**fastlåst på indfaldende lys**: det kører i åben sløjfe ved den indfaldsmålte eksponering med forstærkning på 0 dB — eksponeringen følger det målte lys, ikke motivets indhold. Da loftet kun kan forkorte eksponeringstiden, kan det ikke i sig selv forårsage udklipning. Siden version 1.3.0 beregnes loftet ud fra kameraets egen eksponeringskonstant fra dets kalibreringspakke (en flåde-gennemsnitsværdi kun, når pakken ikke indeholder en sådan). Loftet deaktiveres automatisk — og normal scene-AE genoptages — når der ikke er noget kalibreringsmål i billedet, DAQ-aflæsningen mangler, er forældet (>30 s), mørk, eller hvis ≥15 % af billedet klipper ved den fastsatte eksponering (hvilket betyder, at sensoren og kameraet registrerer forskellig belysning). Der er ingen GUI-kontakt; dette er standardadfærd, når et RGB-kamera har en DAQ-binding.

### Indsamling og udløser

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Array-medlemmer viser desuden skrivebeskyttede rækker for **Rolle**(Master i blåt / Slave i grønt),**Synkroniseringslinje**og**Peers**.

| Kontrol | Valgmuligheder | Standard | Bemærkninger |
| --- | --- | --- | --- |
| **Trigger-tilstand** | Fra / Til | Til | Deaktiveret for array-medlemmer (arrayet styrer udløsningen). |
| **Triggerkilde** | Software / Linje 0 (M8) / Linje 1 / Linje 2 | Linje 0 | Skjult, når trigger-tilstand er slået fra; deaktiveret for array-medlemmer. Linje 0 er den optoisolerede eksterne triggerindgang på M8. |
| **Billedhastighed**| Auto / Manuel + værdi | Auto |**Auto**: kameraets begrænsning af billedfrekvensen er slået fra — eksponeringen bestemmer fps, og feltet viser den aktuelle live-frekvens.**Manuel**: Du begrænser fps med en skyder (fra 1 op til det båndbreddebegrænsede maksimum), der tager udgangspunkt i den aktuelle faktiske hastighed. Array-medlemmer ser en skrivebeskyttet angivelse »N fps (live)« med teksten »Indstillet i array-indstillinger«. |

### Netværk / Transport

| Række | Adfærd |
| --- | --- |
| **Pakke størrelse**| 1500 (Standard) / 9000 (Jumbo) — standard**Jumbo**. |
| **Gennemstrømning** | Skrivebeskyttet grænse for linkgennemstrømning i MB/s. Backend omfordeler dette mellem alle tilsluttede kameraer ved hver tilslutning/afbrydelse. |
| **Bufferhåndtering** | Skrivebeskyttet bufferhåndteringsmodus. |

### Optagelse

Ruden afsluttes med en **"Åbn optagelsesindstillinger…"**-knap, der fører til [ruden Optagelsesindstillinger](capture.md#the-capture-settings-pane) (deaktiveret, indtil et projekt er åbent — "Opret eller åbn et projekt for at gemme optagelser"). Hvis kameraet er skjult eller sat på pause, minder et tip dig om at vise det igen/genoptage det, før du optager.

## Vinduet Array-indstillinger

Åbnes med **tandhjulet**på en ARRAY-række. Overskrift: array-navn med en blyant til omdøbning og**×** til at lukke. Afsnit nedenfor markeret med *kun kombineret* vises kun for arrays, der er forbundet i kombineret visningstilstand.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Synkronisering

Skrivebeskyttede rækker for **Master**,**Slaves**og**Synkroniseringslinje**.

### Omgivende lyssensor

Vises for både kombinerede og separate arrayer:

* **Lyssensor**-rullemenu — knytter én DAQ til hele arrayet. Valget træder i kraft med det samme, overføres til hvert enkelt kameras egen rullemenu for lyssensor (du kan stadig tilsidesætte indstillingen for hvert kamera) og begynder at videresende spektre til arrayet.
* Live **Status**-linje: Fra · "Venter på første spektrum…" · "Aktiv — alle kameraer i arrayet er belysningskorrigeret" · "Intet nyt spektrum i de sidste 3 sekunder — bruger stadig den sidste aflæsning (ingen timeout for forældede data)…".
* Bemærkning i ruden: "Radiometrisk korrektion for hele arrayet. Indstillinger pr. kamera tilsidesætter dette."

### Kalibreringsmål

Sin egen gruppe umiddelbart efter Omgivende lyssensor, både for kombinerede og separate arrayer:

* **Aktiver**-afkrydsningsfelt — "Registrer MAPIR ArUco-mål og valider NDVI mod panelrefleksions-LUT"; styrer den kombinerede flises måloverlay og valideringstabeller. På et **separat** array aktiverer den hvert medlems egen [kalibreringsmåltabel](#the-calibration-target-table) (én pr. kamerarække i listevisningen).

### Optagelse — ensartede sensorindstillinger

Vises for både kombinerede og separate arrays (gruppeoverskriften lyder "Optagelse (ensartet på tværs af kameraer)"). Disse indstillinger gælder ensartet for alle medlemmer (ændringer pr. medlem ville ødelægge synkroniseringen). Redigeringer lagres midlertidigt og anvendes samlet med **Anvend**— undtagen vippeknappen**ROI til / ROI fra**, som anvendes med det samme.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Kontrol | Valgmuligheder / interval | Hvad den gør |
| --- | --- | --- |
| **Pixelformat** | 8-bit / 10-bit / 12-bit / 16-bit | Ensartet bitdybde for alle medlemmer; hvert kamera anvender formatet med den pågældende dybde til sin egen sensor. |
| **Binning** | 1x1 / 2x2 / 4x4 | Hardware-binning — bevarer det fulde synsfelt, samtidig med at SNR og billedhastighed forbedres. Ændring heraf nulstiller ROI-felterne til det nye fulde synsfelt. |
| **Opløsning** (forudindstilling) | Fuld / Halv / Kvart | Relativt i forhold til binning; udfylder ROI-felterne med et centreret udsnit. |
| **ROI-beskæring (px)**|**ROI til / ROI fra**-knap,**Vælg**-knap, felterne B / H / X / Y, kort over hele sensoren | Sensorbeskæring — se [ROI-editoren](#the-roi-editor). Bredde/højde justeres til multipla af 16 (minimum 64); forskydninger tilpasses til multipler af 4. Et »max WxH«-tip viser det maksimale, og**Nulstil** returnerer felterne til det fulde synsfelt. Under redigering følger den orange beskæringsramme alle feltændringer på kortet og på array-flisen. |
| **Triggerfrekvens**| Vælg mellem Auto / Manuel + fps 0,5–10, trin 0,5 |**Auto**(standard): backendet udleder udløserfrekvensen ud fra opløsning og båndbredde — indtastningsfeltet er deaktiveret og viser den udledte værdi.**Manuel**: fastlåser din værdi ved klik på Anvend. En fastlåst frekvens, som en ny billedstørrelse ikke kan håndtere, holdes på den hurtigste frekvens, som størrelsen tillader, og genoptages, når billedstørrelsen igen tillader det. |

Bemærk i ruden: »Ændringer af format/opløsning genstarter alle kameraer kortvarigt. Triggerfrekvensen gælder live.« Knapperne **Anvend / Annuller** findes nederst i ruden.

#### ROI-editoren

Nyt i 1.3.0. ROI-beskæringsrækken og kortet nedenunder fungerer sammen:

* **ROI til / ROI fra**— slår sensorbeskæringen til eller fra**øjeblikkeligt**(kameraerne genstarter kortvarigt; array-brikken viser "Anvender indstillinger…").**Fra**læser hele sensoren og**husk din beskæring**— felterne viser den fortsat;**Til** anvender den igen. Hvis du aktiverer ROI uden at have indstillet en beskæring, anvendes en centreret beskæring i halv størrelse ("Der blev ikke indstillet nogen beskæring, så der blev anvendt en centreret beskæring i halv størrelse. Juster den med felterne.").
* **Vælg**— tegner beskæringen på kortet: træk hen over den, eller klik på to modstående hjørner (Esc annullerer). Rektanglet placeres i felterne B/H/X/Y, justeret efter ovenstående trin; intet anvendes, før du trykker på**Anvend**, så du kan finjustere det med feltpilene først. Ét rektangel pr. tryk på Vælg.
* **Kortet**viser beskæringen på hele sensoren, så du kan udvide den igen: »Fuld sensor, live«, mens kameraerne læser hele sensoren, eller »Full-frame-øjebliksbillede fra {tidspunkt}. Kameraerne læser kun beskæringen«, mens de er beskåret. Hvis arrayet kun nogensinde har streamet beskåret, viser kortet »Ingen full-frame-visning endnu: dette array har kun streamet beskåret." med en**Vis fuld ramme**-knap, som slår ROI fra. Udsnittets `W×H @ X,Y`-mærke sidder uden for boksen (over den eller under, når boksen rører toppen), på kortet og på live-flisen.
* **Anvend**bekræfter indstillingerne: et beskæringsbillede aktiverer ROI’en; fuldformatet (efter**Nulstil**) deaktiverer den, mens det tidligere beskæringsbillede stadig huskes; en ROI, der er deaktiveret, og hvis felter stadig viser det huskede beskæringsbillede, forbliver deaktiveret, når du anvender en anden ændring (format, binning, hastighed).
* **På et justeret array placeres hvert medlems beskæringsvindue i henhold til justeringen**, så hvert kamera indrammer det samme område af scenen. Et array, der er beskåret, før det overhovedet blev justeret, justeres først på den fulde sensor (beskæringen fjernes, justeringen udføres, og beskæringen gendannes med vinduer for hvert kamera).
* Efter en beskæringsændring tilpasses den automatiske eksponering til den nye indramning. Under en beskæring kan et medlems eksponering stadig køre op til udløsningsperioden.
* ROI’ens tilstand (til/fra) og den gemte beskæring gemmes sammen med projektet.

### Justering (co-registrering)

Vises for **både kombinerede og separate arrays** (1.2.0 viste det kun for kombinerede arrays); gruppeoverskriften lyder »Justering«.

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Rest**-felt: "RMS x,xx px" — grøn ved under 1 px, gul ved under 3 px, rød i alle andre tilfælde eller hvis et kamera fejlede; "ingen profil" før den første løsning.
* Oversigtslinje: "✓ Alle kameraer justeret (N)" / "⚠ p/N kameraer justeret —  <serial (filter)="">fejl" / "Beskæring aktiv — Kalibrer igen for at justere (bruger hele sensoren)" / »Venter på, at eksponeringen stabiliserer sig…« / »Anvender indstillinger…« (kameraerne genstarter) / »Array leverer ikke længere billeder — tilslut arrayet igen«.
* Tabel pr. kamera: kamera (de sidste 4 cifre i serienummeret + filter), reprojiceringsfejl i px med antal match ("ref" for masteren) og den normaliserede krydskorrelationsscore for overlapningen i forhold til tærskelværdien på 0,35.
* **Rekalibrer justering** knap (viser "Kalibrer justering" før den første profil) — kører samregistrering igen på nye billeder.
* **Afkrydsningsfeltet "Auto-eksponer kameraer til justering"** (markeret som standard) — lyser midlertidigt mørke eller flade kameraer op (først eksponering, derefter forstærkning), så de har tekstur, der kan matches, og gendanner derefter AE.

Chloros justerer automatisk alle arrayer efter tilslutning, i begge visningstilstande, så snart medlemmernes automatiske eksponering har fundet et stabilt niveau (programmet fortsætter med at forsøge i op til 5 minutter); den kombinerede forhåndsvisning justeres også ved åbning. Et separat array har ingen sammensat flise, så dette afsnit aflæser justeringsstatus og kører Recalibrate selv. Kalibrer om, hvis fokus eller scenedybde ændres. Justering er **kun for den aktuelle session** — den gemmes aldrig i en profil, da den afhænger af den aktuelle scenedistance. Optagelser kan stadig eksporteres med pixelregistrering (se [Justerede eksporter](capture.md#per-array-controls)).

### Smart vignettering

* Afkrydsningsfeltet **Aktiver korrektion**— anvender vignetteringsestimatet pr. kamera på den radiometriske kæde (live**og** eksporter).
* **Kalibrer fra nuværende visning**— ret først arrayet mod et ensartet mål (fladskærm, væg eller himmel); hvert kamera udjævnes individuelt, og statusrapporterer "n/N kameraer · −x,x %" udjævningsgevinst.**Ryd** fjerner estimatet.
* Status: "Kalibreret", "Ikke kalibreret" eller "Delvist kalibreret — kør igen", når kun nogle medlemmers estimater stadig gælder ved deres nuværende billedramme (for eksempel efter at en *Rekalibrer* har flyttet et medlems beskæringsvindue) — kør *Kalibrer* igen.
* Finjuster pr. kamera med skyderen **Vignette** pr. kamera i [Live Preview](#live-preview).

### Live Preview

* **Histogram**-afkrydsningsfelt + breddeskyder (0,10–0,90 af billedet) — histogramstriben på feedet vises på alle medlemmers fliser på én gang (inklusive fliser med opdelte kanaler), det kamera-specifikke [Histogramoverlay](#display-overlays-drawn-over-the-live-feed), der er indstillet for hele arrayet. Det vises som aktiveret, når alle medlemmer allerede har det. På et**separat** array er dette den eneste række i gruppen.
* *Kun kombineret:* **Indeks**: aktiver afkrydsningsfelt + tandhjul — åbner den delte [Indeksberegner](#index-calculator-pane) med bånd tegnet fra**alle** medlemskameraer. En linje med udtryksforhåndsvisning nedenunder viser det aktuelle udtryk ("Intet udtryk angivet — åbn beregneren for at oprette et"), der opdateres hvert sekund.
* *Kun kombineret:* Rullemenuen **Renderopløsning**(samme forudindstillinger som pr. kamera, standard 720p): højden på live-view-streamen**og** den gemte størrelse på den sammensatte eksport. Bemærk i panelet: "Forhåndsvisning + størrelse på gemt sammensat billede. Billeder pr. kamera eksporteres altid i fuld opløsning."

### Visningslag *(kun kombineret)** **Aktivér**-afkrydsningsfelt (standard: fra — hovedkameraet vises direkte; til = lagdelt sammensætning).
* **Rullemenuerne**Forgrund**/**Baggrund**: hvert medlemskamera (efter navn) eller**Indeks**. Når Forgrund er Indeks, viser pixels uden for LUT Min/Max baggrundslaget.

### Delt visning *(kun kombineret)*

**"Vis medlemskameraer"**— en**Del / Skjul medlemskameraer**-knap, der tilføjer hvert medlems eget livefeed som separate gitterfelter ved siden af sammensætningen. Felterne læser arrayets eksisterende billedbuffer (ingen ekstra kameratilslutning). Kun gittervisning; gemmes pr. array sammen med projektet.

### Funktioner

Et skrivebeskyttet panel, der opdateres hvert 5. sekund:

* **Niveaumærke**: "Samtidig optagelse" (grøn) · "Samtidig optagelse (FTD-forskudt udsendelse)" (grøn) · "Forskudt optagelse (100 ms afdrift)" (gul) · "Konfigurationen er for stor" (rød).
* **Rammestatus**: "x,xx % ufuldstændig" — grøn under 1 %, gul under 5 %, rød ved 5 % eller mere.
* **Linklinje**: "NIC {mbps} Mbps – vedvarende {MB/s} MB/s".

Dette er arrayets aktuelle båndbreddebudget. For de underliggende fps og netværksmodeller — samt hvad der skal ændres, når niveauet skifter til gult eller rødt — se [Multi-Camera Arrays](arrays.md) og [CLI-referencen](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Fanen »Index Calculator«

Den tredje side i sidepanelet, som deles af »Index«-indstillingen pr. kamera og »Index«-indstillingen for det samlede array (én ad gangen — overskriften lyder »Index Calculator — <camera name="">« eller »Index Calculator — <array name="">«). Her vises båndlisten (kameraets naturlige filterbåndnaturlige bånd eller alle bånd på tværs af arrayets medlemmer), det aktuelle udtryk og LUT-konfigurationen (til/fra, niveau — standard 7-stop, min — standard 0,2, maks — standard 1) samt et live-indekshistogram. **Anvend** bekræfter udtrykket; LUT-ændringer anvendes live i forhåndsvisningen.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Indstillinger pr. kamera vs. array-styrede indstillinger

Hurtig oversigt over, hvad der findes hvor, når et kamera er medlem af et array:

| Array-styret (skrivebeskyttet i kamerapanelet) | Stadig pr. kamera inden for et array |
| --- | --- |
| Pixelformat, opløsning, binning | Autoeksponering (eksponering, forstærkning, mål, udjævning, ROI) |
| Udløsertilstand/kilde, billedhastighed | Støjfjernelse, skarphed, vignettering |
| | Orientering (spejling/rotation), displayoverlejringer (array-indstillinger → Live Preview indstiller histogrammet på alle medlemmer på én gang), spotmåler |
| | Indeks (arrays med separat visning), lyssensorbinding |

Anden tværgående adfærd:

* **Kombineret vs. separat visning** vælges ved array-tilslutning: kombineret = én justeret sammensat flise (medlemmer sender kun via Split View); separat = hvert medlem gengiver sin egen synkroniserede flise. Et kamera viser aldrig både et selvstændigt feed og en array-flise.
* **Automatisk genforbindelse**: Når et gemt projekt åbnes, gendannes dets kameraer og arrays, og alle gemte indstillinger anvendes igen på backend, før streamene genoptages.
* **Capture-gating**: Skjulte eller pausede kameraer udelades fra Capture All; et array blokeres fuldstændigt kun, når ALLE medlemmer er skjulte/pausede. Se [Optagelsesindstillinger og -tilstande](capture.md).

## Hvordan indstillingerne gemmes

Kamerafanens tilstand gemmes **sammen med projektet**, ikke i browseren:

* Hver reaktiv ændring tager et øjebliksbillede af kameraerne og arrayene i projektets `cameras.json` (debounced 500 ms). Dette omfatter kameranavne og -farver, indstillinger for eksponering/forstærkning/AE, pixelformat/opløsning/binning, udløserhastighed, forhåndsvisningsindstillinger (renderopløsning, støjfjernelse, skarphed, vignettering, farveprofil, højlys, mætning/kontrast), orientering, overlejringer, kanalopdelinger, indekskonfiguration, indstillinger for prædiktiv AE, AE-ROI, array-navne, visningstilstand, indstillinger for array-optagelse (herunder ROI-beskæring, om ROI er aktiveret, og den gemte beskæring, når den er deaktiveret), arrayets Wire Budget og gitterblokken (feed-zoom, visningstilstand, gitterlås, manuel fliserækkefølge, skjulte kameraer, lukkede fliser, aktivt kamera).
* Lyssensorbindinger gemmes i projektets `sensors.json`.
* Når projektet genåbnes, genoprettes forbindelsen til hardwaren, og alle indstillingerne anvendes på ny.
* **Intet projekt åbent = kun session**: uden et projekt bevares intet, når Chloros lukkes.
* Gælder kun for sessionen uanset projekt: pausetilstand, spotmålerprøver, afkrydsningsfeltet »Kalibreringsmål pr. kamera« (åbnes altid som slået fra) og array-justeringsprofilen (genberegnes pr. session som standard).
* Én undtagelse: Valg ved eksport af **optagelsesindstillinger** og optagelsestilstanden bevares pr. projekt i appens lokale lager i stedet for `cameras.json` — se [Optagelsesindstillinger og -tilstande](capture.md).</array></camera></serial>

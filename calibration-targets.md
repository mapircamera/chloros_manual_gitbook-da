---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibreringsmål

MAPIR tilbyder forskellige kalibreringsmål til en række forskellige anvendelser. Den kompakte T4-R50, som ses nedenfor, indeholder 4 paneler, der er målt for lysrefleksion fra 250 til 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4-diffuse referencemål har følgende refleksionskurver, [data kan downloades her](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflektans :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflektans :: 400-1000 nm</p></figcaption></figure>Når man ser på reflektansgrafen, kan man se, at værdierne er bølgelængde (x-aksen) mod reflektansprocent (y-aksen). Når vi tager et billede af kalibreringsmålet, opretter vi derefter en sammenhæng mellem pixelværdi og reflektansprocent inden for det spektrum, som hvert af kameraets sensorbaner er følsomme over for.

Det betyder, at du med hvert billede, du tager med vores kameraer, kan bruge et foto af vores reflektansmål, såsom [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) eller [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) til at kalibrere billederne for reflektans. Når billedet er kalibreret, svarer hver pixel i billedet til procentvis reflektans.

Hvis du udskriver de kalibrerede billeder i Chloros som typiske JPG- eller TIFF-filer, beregnes reflektansprocenten ved at dividere pixelværdien med billedformatets bitdybde. For JPG divideres med 255, og for TIFF divideres med 65.535. Du kan også vælge PERCENT-formatet i Chloros, og så vil hver pixel variere fra en procentværdi på 0,0 til 1,0 (0 % til 100 % reflektans). Husk blot, at nogle billedapplikationer ikke kan acceptere procentbilleder (flydende komma), og at de er store i størrelse med hensyn til lagerplads.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>

---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibreringsmål

MAPIR tilbyder forskellige kalibreringsmål til en lang række anvendelsesformål. Den kompakte T4-R50, som ses nedenfor, indeholder 4 paneler, der er målt for lysrefleksion i området 250–2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>T4-diffusreferencemålene har følgende reflektanskurver, [data kan downloades her](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Refleksion :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4-refleksion :: 400–1.000 nm</p></figcaption></figure>T4P-diffuse referencemål har følgende reflektanskurver, [data kan downloades her](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P Refleksion :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P-reflektans :: 400-1000 nm</p></figcaption></figure>Når man ser på reflektansgrafen, kan man se, at værdierne er bølgelængde (x-aksen) mod reflektansprocent (y-aksen). Når vi tager et billede af kalibreringsmålet, opretter vi derefter en sammenhæng mellem pixelværdi og reflektansprocent inden for det spektrum, som hvert af kameraets sensorbaner er følsomme over for.

Dette betyder, at med hvert billede, du tager med vores kameraer, kan du bruge et foto af vores reflektansmål, såsom [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) eller [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), til at kalibrere billederne for reflektans. Når kalibreringen er udført, svarer hver pixel i billedet til en procentdel af reflektansen.

Hvis du eksporterer de kalibrerede billeder i Chloros som et typisk JPG- eller TIFF-billede, beregnes reflektansprocenten ved at dividere pixelværdien med billedformatets bitdybde. For JPG divideres der altså med 255, og for TIFF divideres der med 65.535. Du kan også vælge PERCENT-formatet i Chloros, og så vil hver pixel ligge mellem 0,0 og 1,0 (0 % til 100 % reflektans). Bare husk, at nogle billedprogrammer ikke kan håndtere billeder i procent (flydende komma), og at de fylder meget på harddisken.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>

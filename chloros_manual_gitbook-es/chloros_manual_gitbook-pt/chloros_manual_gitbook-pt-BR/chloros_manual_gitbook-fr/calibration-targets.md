---
description: Lab-målte paneler, der bruges til at kalibrere opsamlede data i efterbehandling
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibreringsmål

MAPIR tilbyder forskellige kalibreringsmål til at dække en række applikationer. Den kompakte T4-R50, som ses nedenfor, indeholder 4 paneler, der er blevet målt for lysreflektans fra 250 - 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

T4 diffuse referencemål har følgende reflektanskurver, [data download her](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Refleks :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Refleks :: 400-1000nm</p></figcaption></figure>

Når man ser på reflektansgrafen, kan man se, at værdierne er bølgelængde (x-akse) versus reflektansprocent (y-akse). Når vi tager et billede af kalibreringsmålet, skaber vi et forhold mellem pixelværdi og reflektansprocent inden for det spektrum, som hvert af kameraets sensorbånd er følsomme over for.

Det betyder, at med hvert billede, du tager med vores kameraer, kan du bruge et foto af vores reflektionsmål, såsom [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) eller [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) til at kalibrere billederne til refleksion. Når de er kalibreret, er hver pixel i billedet lig med procent reflektans.

Hvis du udsender de kalibrerede billeder i Chloros som den typiske JPG eller TIFF, beregnes reflektansprocenten ved at dividere pixelværdien med billedformatets bitdybde. Så for JPG divideres med 255, og for TIFF divideres med 65.535. Du kan også vælge output i PERCENT-format i Chloros, og så vil hver pixel variere fra en procentværdi på 0,0 til 1,0 (0% til 100% reflektans). Bare husk på, at nogle billedapplikationer ikke kan acceptere procent (floating point) billeder, og de er store i størrelse lagringsmæssigt.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption>___TAG <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>

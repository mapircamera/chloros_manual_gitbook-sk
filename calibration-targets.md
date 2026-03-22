---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibračné terče

Spoločnosť MAPIR ponúka rôzne kalibračné terče pre širokú škálu aplikácií. Kompaktný model T4-R50, ktorý vidíte nižšie, obsahuje 4 panely, ktorých odrazivosť svetla bola zmeraná v rozsahu 250 – 2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Difúzne referenčné terče T4 majú nasledujúce krivky odrazivosti, [stiahnutie údajov tu](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Odrazivosť T4 :: 250–2 500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Odrazivosť T4 :: 400–1 000 nm</p></figcaption></figure>Difúzne referenčné terče T4P majú nasledujúce krivky odrazivosti, [dáta na stiahnutie tu](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR T4P odrazivosť :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR T4P Odrazivosť :: 400–1000 nm</p></figcaption></figure>Pri pohľade na graf odrazivosti vidíte, že hodnoty predstavujú vlnovú dĺžku (os x) v porovnaní s percentom odrazivosti (os y). Keď zachytíme obraz kalibračného terča, vytvoríme vzťah medzi hodnotou pixelu a percentom odrazivosti v rámci spektra, na ktoré je citlivé každé z pásiem senzora fotoaparátu.

To znamená, že pri každom snímku, ktorý zachytíte našimi kamerami, môžete použiť fotografiu našich cieľov odrazivosti, ako je [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) alebo [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), na kalibráciu snímok z hľadiska odrazivosti. Po kalibrácii sa každý pixel na snímke rovná percentu odrazivosti.

Ak kalibrované snímky exportujete v Chloros ako bežný JPG alebo TIFF, potom sa percentuálna odrazivosť vypočíta vydelením hodnoty pixelu bitovou hĺbkou formátu snímky. Pre JPG teda vydelíte 255 a pre TIFF vydelíte 65 535. Môžete si tiež zvoliť výstup vo formáte PERCENT v Chloros, pričom každý pixel bude mať hodnotu v rozmedzí od 0,0 do 1,0 (od 0 % do 100 % odrazivosti). Majte však na pamäti, že niektoré aplikácie na spracovanie obrázkov nepodporujú obrázky v percentách (s plávajúcou desatinnou čiarkou) a z hľadiska úložného priestoru sú tieto obrázky veľké.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>

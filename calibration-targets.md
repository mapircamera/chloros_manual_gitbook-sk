---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Kalibračné terče

MAPIR ponúka rôzne kalibračné terče pre širokú škálu aplikácií. Kompaktný model T4-R50 uvedený nižšie obsahuje 4 panely, ktoré boli merané na odraz svetla v rozsahu 250 – 2 500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Difúzne referenčné ciele T4 majú nasledujúce krivky odrazivosti, [stiahnutie údajov tu](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Odrazivosť :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Odrazivosť :: 400–1000 nm</p></figcaption></figure>Na grafe odrazivosti vidíte, že hodnoty sú vlnová dĺžka (os x) oproti percentuálnej odrazivosti (os y). Keď zachytíme obraz kalibračného cieľa, vytvoríme vzťah medzi hodnotou pixelu a percentuálnou odrazivosťou v rámci spektra, na ktoré sú citlivé jednotlivé senzorové pásma fotoaparátu.

To znamená, že pri každom obrázku, ktorý zachytíte našimi kamerami, môžete použiť fotografiu našich cieľov odrazivosti, ako napríklad [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) alebo [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), na kalibráciu obrázkov pre odrazivosť. Po kalibrácii sa každý pixel v obraze rovná percentuálnej odrazivosti.

Ak kalibrované snímky v Chloros exportujete ako bežný formát JPG alebo TIFF, percentuálna hodnota odrazivosti sa vypočíta vydelením hodnoty pixelu bitovou hĺbkou formátu snímky. Pre formát JPG vydelíte hodnotu 255 a pre formát TIFF vydelíte hodnotu 65 535. Môžete tiež zvoliť výstup vo formáte PERCENT v Chloros, a potom každý pixel bude mať hodnotu v rozmedzí od 0,0 do 1,0 (0 % až 100 % odrazu). Majte však na pamäti, že niektoré aplikácie na spracovanie obrázkov neakceptujú obrázky v percentách (s plávajúcou desatinnou čiarkou) a z hľadiska veľkosti úložiska sú veľké.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>

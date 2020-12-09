---------------------------------------------------
Provisional Surface Temperature Processing (PSTP)
---------------------------------------------------
Data Created: 28 September, 2019

This R script utilizes specific raster layers of the USGS
provisional surface temperature product in order to quantify
and apply masking effects to Landsat surface temperatures layers
to make them analysis ready.
For the provisional temperature data the Landsat bands we used were the CDIST (distance of pixel from cloud) and ST (surface temperature in K)

 Required Packages
===================
* rgeos R package
* Rgdal R package
* raster R package
* ggplot2 R package

 Parameters
-------------
Describe any steps needed for the script to run. It will help to 
specify which line in the code will need to be changed by the user
based on their needs. The R script completes the following processes: 

Part I
1. Stack all CDIST and ST bands from multiple Landsat tifs from single folder together (i.e. Landsat Folder).
2. Open shapefile and create vector study area extent. 
2. Crop all stacked CDIST and ST bands to study area extent and save cropped data to new output folder (Cropped Files).

Part II
3. Make lists of cropped CDIST and ST files.
4. A mask is made from each CDIST band based on a defined value (i.e. distance from cloud) and applied to its corresponding ST band.
(Think of the distance from cloud number used in the cloud masking as the buffer distance around each cloud pixel you would also like to crop out (i.e. [CDIST<2] cuts out all pixels with values 0(cloud) to 2 (within 2 units of measure from cloud pixel)).
5. Write out newly masked ST rasters as GeoTifs to a new folder.

Part III
6. Pre-select a single Landsat image to base the water mask off of. An ideal image would be one that contains a high contrast between water temperature and land temperature values (typically in cooler months).
Water mask is based off of the water temperature (i.e. [Water_mask < 2800] makes mask of all ST values < 2800 (degree K*10)) so choosing an image where the water is much colder than een the coolest land ensures that only areas of water are in the mask.
7. Apply this water mask to all cropped and cloud-masked ST bands.
8. Write out newly masked ST files


---------
 Contact
---------
Josi Robertson
josi.robertson@gmail.com

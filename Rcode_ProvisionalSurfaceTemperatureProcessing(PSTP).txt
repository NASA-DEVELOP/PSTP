#### install and import needed packages into R
install.packages("raster")
install.packages("rgdal")
install.packages("rgeos")
install.packages(("ggplot2"))
library(raster)
library(rgdal)
library(rgeos)
library(ggplot2)

### turn off factors and adjust settings
options(stringsAsFactors = FALSE)
par("mar") #check figure margin settings (default = 5.1  4.1  4.1  2.1)
par(mar=c(1,1,1,1)) #re-size figure margins to usable size


####################   1)  Stack multiple landsat tifs from single folder togather, crop all at once, and save to new output folder:
############# For the provisional temperature data the Landsat bands we used were the CDIST (distance of pixel from cloud) and ST (surface temperature in K)
##### below makes a list of all CDIST and ST files within a folder (name of folder with be the one all of our tifs are in) 
all_files = list.files("Landsat Folder File Pathway", pattern = "ST.tif$", full.names = TRUE) #Grabs all ST and CDIST tiffs togather
all_files

# stack (then brick) data so can edit all files as group
landsat_stack_all = stack(all_files)
landsat_brick_all = brick(landsat_stack_all)
# view brick attributes
landsat_brick_all


##### Open Vector (i.e. study area shapefile) layer & plot. The study area shapefile will provide the extent for cropping our stacked tifs
crop_extent = readOGR("study_area.shp")

## Plot study area shapefile to see if it looks right
plot(crop_extent,
	main = "Shapefile imported into R -cropped extent",
	axes = TRUE,
	border = "blue")

## Crop the stack of landsat rasters using the vector extent from the study area shapefile
all_files_crop = crop(landsat_brick_all, crop_extent)
#check cropped files
names(all_files_crop)

##### upack stack and save newly cropped files as GeoTiffs to new folder (Cropped Files)
writeRaster(all_files_crop, file.path("Cropped Files Folder Pathway", names(landsat_brick_all)), bylayer=TRUE, overwrite = TRUE, format='GTiff')



################### 2) Cloud Mask all cropped files

# make a list that pulls only the CDIST rasters
filenamesCDIST = list.files(path="Cropped Files Folder Pathway", pattern = glob2rx("*CDIST*.tif$"), full.names = TRUE)

# make a list that pulls only the ST rasters
filenamesST = list.files(path="Cropped Files Folder Pathway", pattern = glob2rx("*_ST.*tif$"), full.names = TRUE)

# check the lengths of both lists (they should match)
print(length(filenamesCDIST)) 
print(length(filenamesST))

# set length of list as variable
numfiles = length(filenamesCDIST)

# create a sequence of the number of files will use (i.e. number of CDIST/ST rasters)
fileseq = seq(numfiles)
# check that sequence numbers match the length
print(fileseq)

##### For each CDIST-> make cloud mask and then apply it to the corresponding ST raster

for (i in fileseq){

	print(i)
	print(filenamesCDIST[i])
	print(filenamesST[i])

	CDIST = raster(filenamesCDIST[i])
	CDIST[CDIST < 1] = NA    # mask only 0's (i.e. distance from cloud=0), will increase range to account for a 'buffer' area around cloud

	ST = raster(filenamesST[i])
	masked <- mask(x = ST, mask = CDIST)


# writes out newly masked ST rasters as GeoTif to a DIFFERENT folder 
writeRaster(masked, file.path("Cloud Masked Files Folder Pathway", names(masked)), bylayer=TRUE, overwrite = TRUE, format='GTiff')

}


################### 3) Water Mask all cropped and Cloud masked files

##### Choose the ST.tif file that will be used as a water mask for all other ST files
Water_mask = raster("File Pathway")

# stack all cloud masked ST files 
filenames = list.files(path="Cloud Masked Files Folder Pathway", pattern = glob2rx("*_ST.tif$"), full.names = TRUE)
print(length(filenames))
Water_stack = stack(filenames)

# apply water mask to stack and output to same folder, old cloud masked files will be overwritten by cloud & water masked files
Water_mask[Water_mask < 2800] = NA
Wmasked <- mask(x = Water_stack, mask = Water_mask)
writeRaster(Wmasked, file.path("Cloud Masked Files Folder Pathway", names(Wmasked)), bylayer=TRUE, overwrite = TRUE, format='GTiff')



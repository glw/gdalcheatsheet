gdal/ogr cheatsheet
===

GDAL Cheat Sheet

---
+ Compress imagery

as seen on http://blog.cleverelephant.ca/2015/02/geotiff-compression-for-dummies.html

        gdal_translate \
        -co COMPRESS=JPEG \
        -co PHOTOMETRIC=YCBCR \
        -co TILED=YES \
        5255C.tif 5255C_JPEG_YCBCR.tif

Create overviews

        gdaladdo \
        --config COMPRESS_OVERVIEW JPEG \
        --config PHOTOMETRIC_OVERVIEW YCBCR \
        --config INTERLEAVE_OVERVIEW PIXEL \
        -r average \
        5255C_JPEG_YCBCR.tif \
        2 4 8 16
        
Compress a bunch of tifs...

        for FILE in *.tif
        do
        BASE=`basename $FILE .tif`
        NEWFILE=${BASE}_c.tif
        gdal_translate -b 1 -b 2 -b 3 -co COMPRESS=JPEG -co TILED=YES -co PHOTOMETRIC=YCBCR $FILE $NEWFILE
        done
        

+ GDALWARP: *see below for using gdalwarp to merge tifs
---
    
        gdalwarp -s_srs EPSG:4326 -t_srs EPSG:27700 home_wgs84.bmp home_OSGB36.tif
    
   
Crops image based on shapefile select polygon using -cwhere
    
        gdalwarp -cutline shpfile.shp -cwhere "fieldname = 'fieldvalue'" -crop_to_cutline inimage.tif outimage.tif
         
                * options:
                -cblend <pixels> to feather eadges of imagery for a better seamless image
                -multi to multi-threaded processing
                
   
Mask raster to cutline, use NO_GEOTRANSFORM for un-georeferenced images
    
        gdalwarp -to SRC_METHOD=NO_GEOTRANSFORM -to DST_METHOD=NO_GEOTRANSFORM -cutline cut_line.csv in.tif out.tif

                * Format of cut_line.csv
                id,WKT
                1,"POLYGON((9756 9321, 9756 360, 816 360, 816 9321, 9756 9321))"
    
                * options:
                -dstalpha to create an alpha band, masking nodata pixels
    
    
Subset with -te 
    
        gdalwarp -te -7.35 48.48 3.79 59.51 merged_DEM.tif subset_DEM.tif
        
    
Mosaic with gdal

        gdalwarp --config GDAL_CACHEMAX 3000 -wm 3000 *.tif final_mosaic.tif


* Note that it is usually a good idea to "optimise" the resulting image with gdal_translate.
        
        
+ gdal_translate
---

Compress tif
        

        gdal_translate -of GTiff -co COMPRESS=DEFLATE -co TILED=NO image1.tif image1_compressed.tif
        
        * optional argument to resize image -outsize 50% 50%
        

Convert Multi-band GeoTiff file to JPEG:


        gdal_translate -of JPEG 884084-utm.tif 884084-utm.jpg 


If GDAL complains about strange tags used in a tif file (http://www.gdal.org/frmt_gtiff.html)

        
        gdal_translate -co < PROFILE=BASELINE > or <PROFILE=GeoTIFF> input.tif output.tif
        

+ gdal_contour:
---
    
       gdal_contour -a elev dem.tif contour.shp -i 10.0


+ gdal_merge
---
    
Merge DEMs

        gdal_merge srtm_35_01.tif srtm_35_02.tif srtm_35_03.tif srtm_36_01.tif srtm_36_02.tif srtm_37_02.tif -o merged_DEM.tif


Merge Rasters:

        gdal_merge -o Theale_merged.tif Theale1_cal.bmp Theale2_cal.bmp Theale3_cal.bmp Theale4_cal.bmp
        
 
Copy all tifs to new location

        for %I in (image1.tif image2.tif image3.tif image4.tif) \
        do \
        copy %I test\folder\
        
+ gdal2tiles - Create web map tiles

        gdal2tiles.py --zoom=11-15 --title=maptitle in.tif output_folder_name
        
+ gdal polygonize

        gdal_polygonize Project_clip1.tif -f "ESRI Shapefile" vector.shp crit2 Value
        
        
+ gdal calc
  source: http://www.gdal.org/gdal_calc.html
  
        Average 2 bands together
        gdal_calc.py -A input.tif -B input2.tif --outfile=result.tif --calc="(A+B)/2"
    




OGR
===

+ SUBSET SHAPEFILE:
---
    
       ogr2ogr -spat -1.5 51 -0.5 52 -f "ESRI Shapefile" watersubset.shp waterways.shp


+ MEND SHAPEFILE:
---
    
       ogr2ogr -skipfailures -f "ESRI Shapefile" mended.shp broken.shp


+ KML to SHP:
---
    
       ogr2ogr -f "ESRI Shapefile" thames.shp thames.kml


+ CONVERT SHAPE TO KML and change proj:
---
    
       ogr2ogr -f "KML" -s_srs "EPSG:27700" -t_srs  "EPSG:4326" rail.kml rail_OSGB36.shp


+ Convert shp to kml w/ descriptions:
---
    
       ogr2ogr -f "KML" sample.kml sample.shp -dsco NameField=Field1 -dsco DescriptionField=field2


+ Convert shp to kml - using "where" as query featuures:
---
    
       ogr2ogr -f "KML" -where "NBRHOOD='Telegraph Hill'" realtor_neighborhoods.kml realtor_neighborhoods.shp


+ Convert shp to kml - using "select" to add specific attributes:
---
    
       ogr2ogr –SELECT “field1 field2 field3” -t_srs EPSG:4326 -f "KML" outPutFileName.kml inPutFileName.shp


+ REPROJECT SHAPE FILE:
---
    
       ogr2ogr -s_srs "EPSG:4326" -t_srs "EPSG:27700" buildings_OS.shp buildings.shp


+ Get information about a shapefile: (List Fields and type)
---
    
      ogrinfo -al ssurgo_geo.shp **This Lists ALL geometry which can be a pain

      ogrinfo -al -geom=NO ssurgo_geo.shp **Will not list millions of pages of geometry


+ Extract all polygons from a SSURGO shapefile where the mapunit symbol is 'ScA':
---
    
      ogr2ogr -where "musym = 'ScA' " ssurgo_ScA.shp ssurgo_utm.shp 
    
+ GPX files
---
source: http://www.gdal.org/ogr/drv_gpx.html
    
      ogr2ogr --config GPX_SHORT_NAMES YES out input.gpx track_points
    
* GPX_SHORT_NAMES YES = converts long column names to shorter names to prevent non-unique names
* out = is the ouput location and folder
* track_points = the feature type you are converting/extracting from the file. Other options are: waypoints, route_points, routes, tracks. If nothing is specified then all are extracted. An empty shp file is created for those with no features.
    

+ iterate over many files linux/unix (source: http://gothos.info/tag/gdal-ogr/)
---

    #!/bin/bash
    # from Sherman (2008) Desktop GIS Mapping the Planet With Open Source Tools pp 243-44

    for shp in *.shp \
    do \
    echo “Processing $shp” \
    ogr2ogr -f “ESRI Shapefile” -t_srs EPSG:4326 geo/$shp $shp \
    done \

+ Ogr with SQL
---
source: http://www.sarasafavi.com/intro-to-ogr-part-i-exploring-data.html

    ogrinfo city_of_austin_parks.shp -sql "SELECT COUNT(*) FROM city_of_austin_parks"
    
    * add '-so' for summary only 

SQL and look at only one feature
    
    ogrinfo -q city_of_austin_parks.shp -sql "SELECT * FROM city_of_austin_parks" -fid 1
    
    * '-q' = quiet

Same using all SQL

    ogrinfo -q city_of_austin_parks.shp -sql "SELECT * FROM city_of_austin_parks WHERE fid IN (1,3)"

Using OGR with GNU Parallel 
Source:http://blog.faraday.io/how-to-crunch-lots-of-geodata-in-parallel/
    
    mkdir wgs84  
    ls *.shp | parallel ogr2ogr -t_srs 'EPSG:4326' wgs84/{} {} 
    
    

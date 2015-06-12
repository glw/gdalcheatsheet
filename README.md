gdal/ogr cheatsheet
===

GDAL Cheat Sheet
(not sure of original sources)


+ GDALWARP:
---
    
    gdalwarp -s_srs EPSG:4326 -t_srs EPSG:27700 home_wgs84.bmp home_OSGB36.tif
    

GDAL_warp subset raster with SHP file
---
    gdal_warp -cutline shpfile.shp -cwhere "Id = 0" inimage.tif outimage.tif


Note that it is usually a good idea to "optimise" the resulting image with gdal_translate.

+ Convert Multi-band GeoTiff file to JPEG:
---
    
    gdal_translate -of JPEG 884084-utm.tif 884084-utm.jpg 


+ Contour:
---
    
    gdal_contour -a elev dem.tif contour.shp -i 10.0


+ SUBSET SHAPEFILE:
---
    
    ogr2ogr -spat -1.5 51 -0.5 52 -f "ESRI Shapefile" watersubset.shp waterways.shp


+ MEND SHAPEFILE:
---
    
    ogr2ogr -skipfailures -f "ESRI Shapefile" mended.shp broken.shp


+ MERGE DEMs (DEM tiffs):
---
    
    gdal_merge srtm_35_01.tif srtm_35_02.tif srtm_35_03.tif srtm_36_01.tif srtm_36_02.tif srtm_37_02.tif -o merged_DEM.tif


+ SUBSET DEMs (DEM tiffs):
---
    
    gdalwarp -te -7.35 48.48 3.79 59.51 merged_DEM.tif subset_DEM.tif


+ MERGE RASTERS:
---
    
    gdal_merge -o Theale_merged.tif Theale1_cal.bmp Theale2_cal.bmp Theale3_cal.bmp Theale4_cal.bmp


OGR
===

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


+ Get information about a shapefile:
---
    
    ogrinfo -al ssurgo_geo.shp


+ Extract all polygons from a SSURGO shapefile where the mapunit symbol is 'ScA':
---
    
    ogr2ogr -where "musym = 'ScA' " ssurgo_ScA.shp ssurgo_utm.shp 
    

source: http://www.gdal.org/ogr/drv_gpx.html
    
    ogr2ogr --config GPX_SHORT_NAMES YES out input.gpx track_points

    GPX_SHORT_NAMES YES = converts long column names to shorter names to prevent non-unique names
    out = is the ouput location and folder
    track_points = the feature type you are converting/extracting from the file. Other options are: waypoints,     route_points, routes, tracks. If nothing is specified then all are extracted. An empty shp file is created for those with no features.
    
+ gdal polygonize
---
    gdal_polygonize Project_clip1.tif -f "ESRI Shapefile" vector.shp crit2 Value
    

+ iterate over many files linux/unix (source: http://gothos.info/tag/gdal-ogr/)
---

    #!/bin/bash
    # from Sherman (2008) Desktop GIS Mapping the Planet With Open Source Tools pp 243-44

    for shp in *.shp
    do
    echo “Processing $shp”
    ogr2ogr -f “ESRI Shapefile” -t_srs EPSG:4326 geo/$shp $shp
    done

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


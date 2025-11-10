# An Introduction to Spatial Data in R {#spatial}

## Objectives

- Understand the structures of spatial data in R, including both vector (points,
lines, polygons) and raster (gridded) data
- Be familiar with the core R packages for spatial data (`sf`, `terra`)
- Plot spatial data using `ggplot2`
- Work with both geographic and projected coordinates in R

## Additional reading

Edzer Pebsma, and Roger Bivand. Spatial Data Science With Applications in R. Chapter 7.1: Package sf. Available: https://r-spatial.org/book/07-Introsf.html#sec-sfintro

Claudia Engel. Using Spatial Data with R. https://cengel.github.io/R-spatial/

## Characteristics of spatial data

For this lesson, I assume you have some working knowledge or experience working with
spatial information. You might have collected waypoints while working in the field,
plugged coordinates into Google Maps, or worked extensively with spatial data in
a GIS. This lesson will not cover spatial *analysis* -- that is a whole course
in itself. Instead, the goal of this lesson is for you to be able to read, manipulate,
and plot data that has a spatial component, focusing on points.

Data with a spatial component is simply data that has coordinates associated with
it. These data can be points (e.g., an organism's location), lines (e.g., an
animal's movement trajectory), polygons (e.g., the boundaries of a study site).
These three types of data are called **vector data** (also called feeatures). 
Here, *vector* means something different than it usually does in R; rather than 
a series of elements of the same type, it means that the data is defined by coordinate 
pairs: the pairs of coordinates that define points or vertices. Spatial data can 
also come in a **raster**, or gridded, format, in which case a region is divided 
into rectangles of the same size, each of which is associated with a value. We will 
mainly focus on vector data in this lesson.

## The `sf` package

The `sf` package is the primary R package for manipulating spatial data. "sf" stands
for "simple features," which is the structure for storing and accessing vector
data through the package. The `sf` package superceded the `sp` package, so if you
find functions on StackOverflow or other sources that use functions starting with 
`sp`, know that these are currently not recommended for use.

At its core, you can think of an `sf` object as an extension of a data frame
or tibble. It contains a column (similar to a list-column) with information about
the features (or *geometries*). Here's an example of a shapefile with points representing
centroids (center points) of Georgia counties:


``` r
library(tidyverse)
library(sf)
```




``` r
class(ga_counties_point)
```

```
## [1] "sf"         "data.frame"
```

This is an `sf` object and also a data frame. Looking at the first few rows of data:


``` r
head(ga_counties_point)
```

```
## Simple feature collection with 6 features and 3 fields
## Geometry type: POINT
## Dimension:     XY
## Bounding box:  xmin: -84.49417 ymin: 31.03786 xmax: -81.44214 ymax: 34.88169
## Geodetic CRS:  WGS 84
##     NAME10                    Reg_Comm Sq_Miles                   geometry
## 1   Lanier            Southern Georgia  199.803 POINT (-83.06265 31.03786)
## 2    Bryan Coastal Regional Commission  455.108 POINT (-81.44214 32.01314)
## 3  Appling   Heart of Georgia Altamaha  512.558 POINT (-82.28893 31.74921)
## 4    Rabun           Georgia Mountains  376.854 POINT (-83.40222 34.88169)
## 5 Bleckley   Heart of Georgia Altamaha  219.121 POINT (-83.32789 32.43442)
## 6  Fayette Atlanta Regional Commission  199.286  POINT (-84.49417 33.4139)
```

You can see that there are columns giving us the name area, and associated regional
commission for each county, but also a `geometry` column that contains a series 
of features called `POINT`, each of which has an associated latitude and longitude. 
Similarly, a different object that contains *polygons* representing county boundaries 
looks like this:


``` r
head(ga_counties_poly)
```

```
## Simple feature collection with 6 features and 3 fields
## Geometry type: MULTIPOLYGON
## Dimension:     XY
## Bounding box:  xmin: -84.62722 ymin: 30.84471 xmax: -81.13833 ymax: 35.00067
## Geodetic CRS:  WGS 84
##     NAME10                    Reg_Comm Sq_Miles                       geometry
## 1   Lanier            Southern Georgia  199.803 MULTIPOLYGON (((-83.0428 30...
## 2    Bryan Coastal Regional Commission  455.108 MULTIPOLYGON (((-81.40496 3...
## 3  Appling   Heart of Georgia Altamaha  512.558 MULTIPOLYGON (((-82.46585 3...
## 4    Rabun           Georgia Mountains  376.854 MULTIPOLYGON (((-83.61831 3...
## 5 Bleckley   Heart of Georgia Altamaha  219.121 MULTIPOLYGON (((-83.30089 3...
## 6  Fayette Atlanta Regional Commission  199.286 MULTIPOLYGON (((-84.55744 3...
```

Here, each feature contains many coordinates, each of which is a vertex of the polygon.
In addition, printing the object to the screen tells us about some characteristics
of the data, including its projection (see below), the types of geometries it
contains (point, polygon, etc.), and its bounding box (the minimum and maximum
X and Y coordinates).

The `geometry` column acts differently from other columns in tibbles or data frames
because it is "locked"; you cannot un-select it:


``` r
# This does not work
select(ga_counties_point, -geometry) %>% head()
```

```
## Simple feature collection with 6 features and 3 fields
## Geometry type: POINT
## Dimension:     XY
## Bounding box:  xmin: -84.49417 ymin: 31.03786 xmax: -81.44214 ymax: 34.88169
## Geodetic CRS:  WGS 84
##     NAME10                    Reg_Comm Sq_Miles                   geometry
## 1   Lanier            Southern Georgia  199.803 POINT (-83.06265 31.03786)
## 2    Bryan Coastal Regional Commission  455.108 POINT (-81.44214 32.01314)
## 3  Appling   Heart of Georgia Altamaha  512.558 POINT (-82.28893 31.74921)
## 4    Rabun           Georgia Mountains  376.854 POINT (-83.40222 34.88169)
## 5 Bleckley   Heart of Georgia Altamaha  219.121 POINT (-83.32789 32.43442)
## 6  Fayette Atlanta Regional Commission  199.286  POINT (-84.49417 33.4139)
```

However, otherwise `sf` objects can be used with filtering, selecting, and joining
operations just like you would with any other data frame:


``` r
# Some examples
filter(ga_counties_point, NAME10 %in% c("Clarke", "Oconee")) 
```

```
## Simple feature collection with 2 features and 3 fields
## Geometry type: POINT
## Dimension:     XY
## Bounding box:  xmin: -83.43698 ymin: 33.83493 xmax: -83.36731 ymax: 33.95117
## Geodetic CRS:  WGS 84
##   NAME10          Reg_Comm Sq_Miles                   geometry
## 1 Clarke Northeast Georgia  121.027 POINT (-83.36731 33.95117)
## 2 Oconee Northeast Georgia  186.355 POINT (-83.43698 33.83493)
```

``` r
arrange(ga_counties_point, NAME10) %>% head()
```

```
## Simple feature collection with 6 features and 3 fields
## Geometry type: POINT
## Dimension:     XY
## Bounding box:  xmin: -84.44473 ymin: 31.29713 xmax: -82.28893 ymax: 34.35411
## Geodetic CRS:  WGS 84
##     NAME10                  Reg_Comm Sq_Miles                   geometry
## 1  Appling Heart of Georgia Altamaha  512.558 POINT (-82.28893 31.74921)
## 2 Atkinson          Southern Georgia  344.592 POINT (-82.87996 31.29713)
## 3    Bacon          Southern Georgia  285.951 POINT (-82.45265 31.55367)
## 4    Baker         Southwest Georgia  349.074 POINT (-84.44473 31.32617)
## 5  Baldwin            Middle Georgia  267.405 POINT (-83.24982 33.06895)
## 6    Banks         Georgia Mountains  233.861 POINT (-83.49734 34.35411)
```

Because the `geometry` column is locked, it can act unexpectedly during grouping
operations:


``` r
ga_counties_point %>%
  group_by(Reg_Comm) %>%
  summarize(n = n()) %>%
  head()
```

```
## Simple feature collection with 6 features and 2 fields
## Geometry type: MULTIPOINT
## Dimension:     XY
## Bounding box:  xmin: -84.76798 ymin: 30.92246 xmax: -81.0923 ymax: 34.91666
## Geodetic CRS:  WGS 84
## # A tibble: 6 × 3
##   Reg_Comm                        n                                     geometry
##   <chr>                       <int>                             <MULTIPOINT [°]>
## 1 Atlanta Regional Commission    10 ((-84.15418 33.45298), (-84.49417 33.4139),…
## 2 Central Savannah River Area    13 ((-83.00106 33.2702), (-82.87877 33.56608),…
## 3 Coastal Regional Commission    10 ((-81.49362 31.21324), (-81.63629 30.92246)…
## 4 Georgia Mountains              13 ((-82.96425 34.35087), (-83.22921 34.37541)…
## 5 Heart of Georgia Altamaha      17 ((-82.28893 31.74921), (-82.63703 31.80558)…
## 6 Middle Georgia                 11 ((-83.17122 32.80236), (-83.42707 32.66713)…
```

Here, by grouping by regional commission name, we have compressed all the county
centroids into a single MULTIPOINT geometry.

To remove the `geometry` column and convert the `sf` object back to a non-spatial
object, use `st_drop_geometry()`:


``` r
ga_counties_df <- st_drop_geometry(ga_counties_point)
head(ga_counties_df)
```

```
##     NAME10                    Reg_Comm Sq_Miles
## 1   Lanier            Southern Georgia  199.803
## 2    Bryan Coastal Regional Commission  455.108
## 3  Appling   Heart of Georgia Altamaha  512.558
## 4    Rabun           Georgia Mountains  376.854
## 5 Bleckley   Heart of Georgia Altamaha  219.121
## 6  Fayette Atlanta Regional Commission  199.286
```

``` r
class(ga_counties_df)
```

```
## [1] "data.frame"
```

## Map projections and coordinate reference systems

The globe is round but we make plots and measurements in two-dimensional space. 
Doing so inevitably creates some distortion in distance, area, and/or shape. For 
example, consider common some global map projections: 

<img src="spatial_files/figure-html/unnamed-chunk-10-1.png" width="672" />

Each of these has different features, whether creating straight lines for 
navigation (Mercator), distances between points (Azimuthal), or areas of polygons
(Mollweide). Except for the geographic coordinates, whose units are degrees of
latitude and longitude, all of these projections measure X and Y in meters. Another
common projection system is the Universal Transverse Mercator (UTM), where you
will usually see coordinates labeled "northing" and "easting". UTMs are useful
because the system divides the Earth into zones, which results is less distortion
within each zone. However, this property means that UTMs are less useful for 
continental or global-scale maps, since distortion is enhanced outside the zone.
When using UTMs, you need to know not only the northing and easting, but also
which zone was used to create the coordinates.

[This report from the U.S. Geological Survey](https://doi.org/10.3133/70047422) 
provides an overview of some common projections. 

Projections are identified using their coordinate reference system (CRS), which is a
set of information defining how to locate coordinates on a map relative to their
actual location on the three-dimensional globe. We can look at these properties of
an `sf` object using the `st_crs()` function.


``` r
st_crs(ga_counties_point)
```

```
## Coordinate Reference System:
##   User input: epsg:4326 
##   wkt:
## GEOGCRS["WGS 84",
##     ENSEMBLE["World Geodetic System 1984 ensemble",
##         MEMBER["World Geodetic System 1984 (Transit)"],
##         MEMBER["World Geodetic System 1984 (G730)"],
##         MEMBER["World Geodetic System 1984 (G873)"],
##         MEMBER["World Geodetic System 1984 (G1150)"],
##         MEMBER["World Geodetic System 1984 (G1674)"],
##         MEMBER["World Geodetic System 1984 (G1762)"],
##         MEMBER["World Geodetic System 1984 (G2139)"],
##         MEMBER["World Geodetic System 1984 (G2296)"],
##         ELLIPSOID["WGS 84",6378137,298.257223563,
##             LENGTHUNIT["metre",1]],
##         ENSEMBLEACCURACY[2.0]],
##     PRIMEM["Greenwich",0,
##         ANGLEUNIT["degree",0.0174532925199433]],
##     CS[ellipsoidal,2],
##         AXIS["geodetic latitude (Lat)",north,
##             ORDER[1],
##             ANGLEUNIT["degree",0.0174532925199433]],
##         AXIS["geodetic longitude (Lon)",east,
##             ORDER[2],
##             ANGLEUNIT["degree",0.0174532925199433]],
##     USAGE[
##         SCOPE["Horizontal component of 3D system."],
##         AREA["World."],
##         BBOX[-90,-180,90,180]],
##     ID["EPSG",4326]]
```

This contains a lot of information, and you don't need to parse all of it. The last 
bit of information (`ID["EPSG",4326]]`) tells you that the code for this CRS
is EPSG:4326. The first bit of information tells us that we are using geographic
coordinates ("GEOGCRS") under the WGS 84 system, which is the most common form of 
geographic coordinates (lat/long).  If available, it can be helpful to pull out a 
subset of this information using the proj4string:


``` r
st_crs(ga_counties_point)$proj4string
```

```
## [1] "+proj=longlat +datum=WGS84 +no_defs"
```

Using `sf`, we can easily reproject data from one coordinate system to another 
using the `st_transform()` function. If you know what coordinate system you want 
to use, you can usually Google its name and "crs" to find the code. For example,
the code for the Web Mercator projection is EPSG:3857:


``` r
ga_counties_point_merc <- st_transform(ga_counties_point, crs = "EPSG:3857")
st_crs(ga_counties_point_merc)$proj4string
```

```
## [1] "+proj=merc +a=6378137 +b=6378137 +lat_ts=0 +lon_0=0 +x_0=0 +y_0=0 +k=1 +units=m +nadgrids=@null +wktext +no_defs"
```

Now, you see we are using a new coordinate system. If we look at the data itself,
you can see that the coordinates have changed from decimal degrees to meters:


``` r
head(ga_counties_point_merc)
```

```
## Simple feature collection with 6 features and 3 fields
## Geometry type: POINT
## Dimension:     XY
## Bounding box:  xmin: -9405848 ymin: 3637666 xmax: -9066098 ymax: 4147815
## Projected CRS: WGS 84 / Pseudo-Mercator
##     NAME10                    Reg_Comm Sq_Miles                 geometry
## 1   Lanier            Southern Georgia  199.803 POINT (-9246492 3637666)
## 2    Bryan Coastal Regional Commission  455.108 POINT (-9066098 3765036)
## 3  Appling   Heart of Georgia Altamaha  512.558 POINT (-9160362 3730435)
## 4    Rabun           Georgia Mountains  376.854 POINT (-9284292 4147815)
## 5 Bleckley   Heart of Georgia Altamaha  219.121 POINT (-9276018 3820472)
## 6  Fayette Atlanta Regional Commission  199.286 POINT (-9405848 3950373)
```

Converting between projectionsis important for spatial analysis (e.g., overlaying 
two datasets in different projections) and plotting (see examples above and methods 
below).

[XKCD has also covered map projections.](https://xkcd.com/977/)

## Raster data and the `terra` package

Raster data are much more compact than vector data. Instead of storing each rectangle
as a series of vertices with associated information, rasters can store information
about the grid (its starting point, cell size, dimensions, and a projection) and 
couple that with a matrix that contains data. This avoids needing to store the 
actual location of each individual cell; in doing so, you can have a raster that 
contains thousands or millions of cells but takes up much less space than an 
equivalent feature set. A raster can have one or multiple "bands", which are 
the data layers associated with each grid cell.

The modern package for dealing with raster data is `terra`. Like `sf`, `terra` 
superceded an older package (`raster`), so be careful with StackOverflow pages 
from before 2023.

Here is an example of a raster object that contains information on mean temperature
across the globe from the [WorldClim database](https://www.worldclim.org/data/worldclim21.html):


``` r
library(terra)
```




``` r
mean_temp
```

```
## class       : SpatRaster 
## dimensions  : 1080, 2160, 1  (nrow, ncol, nlyr)
## resolution  : 0.1666667, 0.1666667  (x, y)
## extent      : -180, 180, -90, 90  (xmin, xmax, ymin, ymax)
## coord. ref. : lon/lat WGS 84 (EPSG:4326) 
## source      : wc2.1_10m_tavg_01.tif 
## name        : wc2.1_10m_tavg_01 
## min value   :          -45.8840 
## max value   :           34.0095
```

Just this summary gives us a lot of information: the raster is a 1,080 by 2,160
grid across the globe (its extent is the full range of latitude and lontitude).
It is in the WGS84 geographic coordinate system and has a range of values between
-46 and 34 degrees. It doesn't tell us information about the units of the values,
how the data was collected, etc.; we would need to use the metadata to learn this.

A simple call to `plot()` will display the data:


``` r
plot(mean_temp)
```

<img src="spatial_files/figure-html/unnamed-chunk-18-1.png" width="672" />

Here's what this looks like across Georgia:


``` r
ga_ext <- ext(ga_counties_poly) #Get the bounding box (extent) of the GA data
plot(mean_temp, ext = ga_ext) #Plot mean temperature in this box
plot(ga_counties_poly$geometry, add = T) #Add the counties to this map
```

<img src="spatial_files/figure-html/unnamed-chunk-19-1.png" width="672" />

And across Clarke county:


``` r
clarke_poly <- ga_counties_poly %>% filter(NAME10 == "Clarke")
clarke_ext <-  ext(clarke_poly) #Get the bounding box (extent) of Clarke cty
plot(mean_temp, ext = clarke_ext) #Plot mean temperature in this box
plot(clarke_poly$geometry, add = T) #Add the counties to this map
```

<img src="spatial_files/figure-html/unnamed-chunk-20-1.png" width="672" />

This is where you start to see the grid size - Clarke county is so small that
it only contains two grid cells of this raster (at 0.17 degree resolution,
roughly 15 km). If we wanted to do an analysis or make a plot at this scale,
we would probably want to use higher-resolution raster data.

Just like with feature data, it is possible to reproject a raster, using the 
`project()` function in `terra`:


``` r
mean_temp_merc <- project(mean_temp, "EPSG:3857")
mean_temp_merc
```

```
## class       : SpatRaster 
## dimensions  : 2407, 199, 1  (nrow, ncol, nlyr)
## resolution  : 200527.4, 201540.1  (x, y)
## extent      : -20037508, 19867438, -242578415, 242528681  (xmin, xmax, ymin, ymax)
## coord. ref. : WGS 84 / Pseudo-Mercator (EPSG:3857) 
## source(s)   : memory
## name        : wc2.1_10m_tavg_01 
## min value   :         -43.92729 
## max value   :          33.00763
```

However, reprojecting rasters is a little more complicated than reprojecting
features/vectors, because we need to make assumptions about how to re-draw the grid
and how to assign new values to the new grid cells. This is why the minimum and 
maximum values of `mean_temp_merc` differ from those in `mean_temp`; the function
made some assumptions about how to combine values when cells in the new grid contained
multiple cells from the old grid.

## Reading and writing: spatial data outside of R

### Reading and creating vectors/features

Vector data outside of R is most often stored as a shapefile with the extension 
`.shp`. `sf` can load spatial data with other file extensions (e.g., `.gpx`, 
`kml`, `.gpkg`). Load the data with the function `st_read()`, which will then
provide information about the object you just read:


``` r
# Read in polygon data of GA counties
ga_counties_raw <- st_read("data/raw/Georgia_Counties/Georgia_Counties.shp")
```

```
## Reading layer `Georgia_Counties' from data source 
##   `/Users/cst80488/Library/CloudStorage/OneDrive-UniversityofGeorgia/Teaching/DataMgmt/DataMgmt_book/data/raw/Georgia_Counties/Georgia_Counties.shp' 
##   using driver `ESRI Shapefile'
## Simple feature collection with 159 features and 22 fields
## Geometry type: MULTIPOLYGON
## Dimension:     XY
## Bounding box:  xmin: -373971.6 ymin: 129391.4 xmax: 1094729 ymax: 1831135
## Projected CRS: NAD83 / Georgia East (ftUS)
```

You can also create an `sf` object from data with coordinates as columns using
`st_as_sf()`:


``` r
# Our coordinate information about the `penguins` islands:
island_info <- data.frame(island_name = c("Torgersen", "Biscoe", "Dream"),
                          area_sqkm = c(0.126, 478.38, 1.678),
                          long = c(-64.0833, -65.9164, -64.2333),
                          lat = c(-64.7667, -65.7474, -64.7333))
island_info_sf <- st_as_sf(island_info, coords = c("long","lat"),
                           crs = "epsg:4326")
island_info_sf
```

```
## Simple feature collection with 3 features and 2 fields
## Geometry type: POINT
## Dimension:     XY
## Bounding box:  xmin: -65.9164 ymin: -65.7474 xmax: -64.0833 ymax: -64.7333
## Geodetic CRS:  WGS 84
##   island_name area_sqkm                  geometry
## 1   Torgersen     0.126 POINT (-64.0833 -64.7667)
## 2      Biscoe   478.380 POINT (-65.9164 -65.7474)
## 3       Dream     1.678 POINT (-64.2333 -64.7333)
```

I specified the X and Y coordinates as a vector to the `coords` argument and
told the function what CRS to use, and I have an `sf` object with a `geometry`
column that replaced the columns `long` and `lat`. When reading a shapefile into
R, it is not necessary to specify the coordinate reference sysetm because that 
information is embedded in the file.

### Reading and creating rasters

Raster data is often stored as an TIFF image (`.tif`), and larger rasters often
come in more compressed formats (e.g., NetCDF, `.nc`). Reading in a TIFF using
`terra` is as simple as including the file path within the `rast()` function:


``` r
mean_temp <- rast("data/raw/wc2.1_10m_tavg/wc2.1_10m_tavg_01.tif")
```

If a raster contains multiple layers, you can specify which to read using the 
`layers` argument.

It is also possible to create a raster from scratch by specifying the dimensions,
extent, resoultion, and values. For example, if I wanted to create a 0.1-degree 
resoultion raster that covers the extent of the `penguins` data, I might do this:


``` r
# Create an empty raster
penguins_gridded <- rast(xmin = min(island_info$long)-1, xmax = max(island_info$long)+1,
                         ymin = min(island_info$lat)-1, ymax = max(island_info$lat)+1,
                         resolution = 0.1,
                         crs = crs(island_info_sf))
# Assign random values to the raster (in this case we don't have data to fill)
values(penguins_gridded) <- rnorm(ncell(penguins_gridded))

# Look at the raster
plot(penguins_gridded)
plot(island_info_sf$geometry, add = T, col = "red", pch = 19)
```

<img src="spatial_files/figure-html/unnamed-chunk-25-1.png" width="672" />

### Writing spatial data to files

To write vector graphics, use `sf::st_write()`. The function will use the file 
extension you supply to infer what type of file to write.


``` r
st_write(island_info_sf, "data/clean/penguin_island_points.shp") # Creates a shapefile
st_write(island_info_sf, "data/clean/penguin_island_points.kml") # Creates a KML
```

Note that if you write to a shapefile, R will also create files with the extensions
`.prj.`, `.shx`, and `.dbf`. These are part of ESRI's shapefile format and provide
additional data (for example, the projection or associated names/data). 

To write a raster, use `terra::writeRaster()` or `terra::writeCDF()`:


``` r
writeRaster(penguins_gridded, "data/clean/penguin_random_raster.tif")
```

Both `st_write()` and `writeRaster()` have safeguards in place to ensure you don't
accidentally overwrite your files. `st_write()` gives you the option to append
information (i.e., add it to the existing file, `append = TRUE`) or overwrite it
(`append = FALSE`). In `writeRaster()`, use `overwrite = TRUE` to overwrite an
exsiting file. 

## Plotting spatial data using `ggplot2`

`ggplot2` interfaces cleanly with the `sf` package by providing a geometry type
for `sf` objects. For example, to plot Georgia county centroids, we can use:


``` r
ggplot(ga_counties_point) +
  geom_sf()
```

<img src="spatial_files/figure-html/unnamed-chunk-28-1.png" width="672" />

Although this map could also be achieved using just latitude and longitude, the 
plot now automatically labels axes with degrees, and uses a 1:1 axis ratio instead
of fitting the plot to the window size. This function also works with polygons:


``` r
ggplot(ga_counties_poly) +
  geom_sf()
```

<img src="spatial_files/figure-html/unnamed-chunk-29-1.png" width="672" />

In most other ways, this acts like a regular ggplot. For example, we can color
our features by a column:


``` r
ggplot(ga_counties_poly) +
  geom_sf(aes(fill = Reg_Comm))
```

<img src="spatial_files/figure-html/unnamed-chunk-30-1.png" width="672" />

Rasters are less easily supported using `ggplot2`; it is easiest to first convert
them to a long-form data frame.


``` r
# Crop January temperature raster to the extent to Georgia
mean_temp_ga <- crop(mean_temp, ga_counties_poly)
# Convert to a data frame
mean_temp_ga_df <- as.data.frame(mean_temp_ga, xy = T)
head(mean_temp_ga_df)
```

```
##           x        y wc2.1_10m_tavg_01
## 1 -85.58333 34.91667           2.96000
## 2 -85.41667 34.91667           3.10475
## 3 -85.25000 34.91667           3.58500
## 4 -85.08333 34.91667           3.43375
## 5 -84.91667 34.91667           3.49200
## 6 -84.75000 34.91667           3.47875
```

Now, we have the X and Y coordinates of each raster cell along with its value. 
For convenience, I will rename this column before plotting the raster.


``` r
mean_temp_ga_df <- rename(mean_temp_ga_df, jan_temp = wc2.1_10m_tavg_01)
ggplot() +
  geom_tile(data = mean_temp_ga_df, aes(x = x, y = y, fill = jan_temp))
```

<img src="spatial_files/figure-html/unnamed-chunk-32-1.png" width="672" />

We can also combine the data types to overlay the data:


``` r
ggplot() +
  geom_tile(data = mean_temp_ga_df, aes(x = x, y = y, fill = jan_temp)) +
  geom_sf(data = ga_counties_poly, fill = NA, color = "black") +
  scale_fill_viridis_c("Mean temperature\nin January (°C)", option = "magma") +
  theme_void()
```

<img src="spatial_files/figure-html/unnamed-chunk-33-1.png" width="672" />

## Other topics

We have barely touched the surface of what you can do with spatial data in R. This
lesson ends here, but if you are a GIS whiz...know that you can do almost everything
you already know how to do in R (though it may not always be as efficient). Here
are some topics we didn't cover but might interest you:

* creating line and polygon vectors/features from coordinates
* different object types in the `sf` world (`sfc`, etc.) that behave differently from data frames
* spatial manipulation (intersections, extractions, cropping, etc.)
* spatial analysis (autocorrelation, trends) - often relying on additional packages
* creating interactive maps with the `leaflet` package




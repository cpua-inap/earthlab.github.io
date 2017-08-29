---
layout: single
title: "GIS in R: intro to vector format spatial data - points, lines and polygons"
excerpt: "This lesson introduces what vector data are and how to open vector data stored in
shapefile format in R. "
authors: ['Leah Wasser']
modified: '2017-08-19'
category: [courses]
class-lesson: ['class-intro-spatial-r']
permalink: /courses/earth-analytics/week-4/intro-vector-data-r/
nav-title: 'Vector data in R'
module-title: 'Spatial data in R and remote sensing uncertainty'
module-description: 'This tutorial covers the basic principles of LiDAR remote sensing and
the three commonly used data products: the digital elevation model, digital surface model and the canopy height model. Finally it walks through opening lidar derived raster data in R / RStudio'
module-nav-title: 'Spatial Data in R'
module-type: 'class'
course: "earth-analytics"
week: 4
sidebar:
  nav:
author_profile: false
comments: true
order: 1
topics:
  spatial-data-and-gis: ['vector-data', 'coordinate-reference-systems']
  reproducible-science-and-programming:
---


{% include toc title="In This Lesson" icon="file-text" %}

<div class='notice--success' markdown="1">

## <i class="fa fa-graduation-cap" aria-hidden="true"></i> Learning Objectives

After completing this tutorial, you will be able to:

* Describe the characteristics of 3 key vector data structures: points, lines and polygons.
* Open a shapefile in R using `readOGR()`.
* View the metadata of a vector spatial layer in R including CRS
* Access the tabular (`data.frame`) attributes of a vector spatial layer in `R`.

## <i class="fa fa-check-square-o fa-2" aria-hidden="true"></i> What you need

You will need a computer with internet access to complete this lesson and the data for week 5 of the course.

[<i class="fa fa-download" aria-hidden="true"></i> Download Week 5 Data (~500 MB)](https://ndownloader.figshare.com/files/7525363){:data-proofer-ignore='' .btn }

</div>

## About Vector Data
Vector data are composed of discrete geometric locations (x,y values) known as
**vertices** that define the "shape" of the spatial object. The organization
of the vertices, determines the type of vector that we are working
with: point, line or polygon.

<figure>
    <a href="{{ site.url }}/images/courses/earth-analytics/week-5/pnt_line_poly.png">
    <img src="{{ site.url }}/images/courses/earth-analytics/week-5/pnt_line_poly.png" alt="points lines and polygons graphic."></a>
    <figcaption> There are 3 types of vector objects: points, lines or
    polygons. Each object type has a different structure.
    Image Source: Colin Williams (NEON)
    </figcaption>
</figure>

* **Points:** Each individual point is defined by a single x, y coordinate.
There can be many points in a vector point file. Examples of point data include:
sampling locations, the location of individual trees or the location of plots.
* **Lines:** Lines are composed of many (at least 2) vertices, or points, that
are connected. For instance, a road or a stream may be represented by a line. This
line is composed of a series of segments, each "bend" in the road or stream
represents a vertex that has defined `x, y` location.
* **Polygons:** A polygon consists of 3 or more vertices that are connected and
"closed". Thus the outlines of plot boundaries, lakes, oceans, and states or
countries are often represented by polygons. Occasionally, a polygon can have a
hole in the middle of it (like a doughnut), this is something to be aware of but
not an issue we will deal with in this tutorial.

<i class="fa fa-star"></i> **Data Tip:** Sometimes, boundary layers such as
states and countries, are stored as lines rather than polygons. However, these
boundaries, when represented as a line, will not create a closed object with a
defined "area" that can be "filled".
{: .notice--success}

## Shapefiles: Points, Lines, and Polygons
Geospatial data in vector format are often stored in a `shapefile` format.
Because the structure of points, lines, and polygons are different, each
individual shapefile can only contain one vector type (all points, all lines
or all polygons). You will not find a mixture of point, line and polygon
objects in a single shapefile.

Objects stored in a shapefile often have a set of associated `attributes` that
describe the data. For example, a line shapefile that contains the locations of
streams, might contain the associated stream name, stream "order" and other
information about each stream line object.

* More about shapefiles can found on
<a href="https://en.wikipedia.org/wiki/Shapefile" target="_blank">Wikipedia</a>.

## Import Shapefiles

We will use the `rgdal` package to work with vector data in `R`. Notice that the
`sp` package automatically loads when `rgdal` is loaded. We will also load the
`raster` package so we can explore raster and vector spatial metadata using similar commands.


```r
# work with spatial data; sp package will load with rgdal.
library(rgdal)
library(rgeos)
## Error in library(rgeos): there is no package called 'rgeos'
# for metadata/attributes- vectors or rasters
library(raster)

# set working directory to earth-analytics dir
# setwd("pathToDirHere")
```

The shapefiles that we will import are:

* A polygon shapefile representing our field site boundary,
* A line shapefile representing roads, and
* A point shapefile representing the location of field siteslocated at the
<a href="http://www.neonscience.org/science-design/field-sites/harvard-forest" target="_blank"> San Joachin field site</a>.

The first shapefile that we will open contains the point locations where trees
have been measured at the study site. The data are stored in shapefile format.
To import shapefiles we use the `R` function `readOGR()`.

`readOGR()` requires two components:

1. The directory where our shapefile lives: `data/week_03/D17-California/SJER/vector_data/`
2. The name of the shapefile (without the extension): `SJER_plot_centroids`

You can call each element separately

`readOGR("path","fileName")`

Or you can simply include the entire path to the shp file in the path argument.
Both ways to open a shapefile are demonstrated below:


```r
# Import a polygon shapefile: readOGR("path","fileName")

sjer_plot_locations <- readOGR(dsn="data/week_03/california/SJER/vector_data",
                               "SJER_plot_centroids")
## Error in ogrInfo(dsn = dsn, layer = layer, encoding = encoding, use_iconv = use_iconv, : Cannot open data source

# note the code below works too
#sjer_plot_locations <- readOGR(dsn="data/week_03/california/SJER/vector_data/SJER_plot_centroids.shp")
```

<i class="fa fa-star"></i> **Data Tip:** The acronym, OGR, refers to the
OpenGIS Simple Features Reference Implementation.
<a href="https://trac.osgeo.org/gdal/wiki/FAQGeneral" target="_blank">
Learn more about OGR.</a>
{: .notice--success}

## Shapefile Metadata & Attributes

When we import the `SJER_plot_centroids` shapefile layer into `R` the `readOGR()` function automatically stores
information about the data. We are particularly interested in the geospatial
**metadata**, describing the format, `CRS`, `extent`, and other components of
the vector data, and the **attributes** which describe properties associated
with each individual vector object.

 <i class="fa fa-star"></i> **Data Tip:** The
 [Shapefile Metadata & Attributes in R](http://www.neondataskills.org/R/shapefile-attributes-in-R/)
tutorial provides more information on both metadata and attributes
and using attributes to subset and plot data.
{: .notice--success}

## Spatial Metadata
Key metadata for all shapefiles include:

1. **Object Type:** the class of the imported object.
2. **Coordinate Reference System (CRS):** the projection of the data.
3. **Extent:** the spatial extent (geographic area that the shapefile covers) of
the shapefile. Note that the spatial extent for a shapefile represents the
extent for ALL spatial objects in the shapefile.

We can view shapefile metadata using the `class`, `crs` and `extent` methods:


```r
# view just the class for the shapefile
class(sjer_plot_locations)
## Error in eval(expr, envir, enclos): object 'sjer_plot_locations' not found

# view just the crs for the shapefile
crs(sjer_plot_locations)
## Error in crs(sjer_plot_locations): object 'sjer_plot_locations' not found

# view just the extent for the shapefile
extent(sjer_plot_locations)
## Error in extent(sjer_plot_locations): object 'sjer_plot_locations' not found

# view all metadata at same time
sjer_plot_locations
## Error in eval(expr, envir, enclos): object 'sjer_plot_locations' not found
```

Our `sjer_plot_locations` object is a polygon of class `SpatialPointsDataFrame`,
in the CRS **UTM zone 18N**. The CRS is critical to interpreting the object
`extent` values as it specifies units.

<figure>
    <a href="{{ site.url }}/images/courses/earth-analytics/week-5/spatial_extent.png">
    <img src="{{ site.url }}/images/courses/earth-analytics/week-5/spatial_extent.png" alt="the spatial extent represents the spatial area that a particular dataset covers."></a>
    <figcaption>The spatial extent of a shapefile or R spatial object represents
    the geographic "edge" or location that is the furthest north, south east and
    west. Thus is represents the overall geographic coverage of the spatial object.
    Image Source: National Ecological Observatory Network (NEON)
    </figcaption>
</figure>

## Spatial Data Attributes
Each object in a shapefile has one or more attributes associated with it.
Shapefile attributes are similar to fields or columns in a spreadsheet. Each row
in the spreadsheet has a set of columns associated with it that describe the row
element. In the case of a shapefile, each row represents a spatial object - for
example, a road, represented as a line in a line shapefile, will have one "row"
of attributes associated with it. These attributes can include different types
of information that describe objects stored within a shapefile. Thus, our road,
may have a name, length, number of lanes, speed limit, type of road and other
attributes stored with it.

<figure>
    <a href="{{ site.url }}/images/courses/earth-analytics/week-5/attribute_table.png">
    <img src="{{ site.url }}/images/courses/earth-analytics/week-5/attribute_table.png" alt="A shapefile has an associated attribute table. Each spatial feature in an R spatial object has the same set of
    associated attributes that describe or characterize the feature.
    Attribute data are stored in a separate *.dbf file. "></a>
    <figcaption>Each spatial feature in an R spatial object has the same set of
    associated attributes that describe or characterize the feature.
    Attribute data are stored in a separate *.dbf file. Attribute data can be
    compared to a spreadsheet. Each row in a spreadsheet represents one feature
    in the spatial object.
    Image Source: National Ecological Observatory Network (NEON)
    </figcaption>
</figure>


We view the attributes of a `SpatialPointsDataFrame` using `objectName@data`
(e.g., `sjer_plot_locations@data`).


```r
# alternate way to view attributes
sjer_plot_locations@data
## Error in eval(expr, envir, enclos): object 'sjer_plot_locations' not found
```

In this case, our polygon object only has one attribute: `id`.

## Metadata & Attribute Summary
We can view a metadata & attribute summary of each shapefile by entering
the name of the `R` object in the console. Note that the metadata output
includes the **class**, the number of **features**, the **extent**, and the
**coordinate reference system** (`crs`) of the `R` object. The last two lines of
`summary` show a preview of the `R` object **attributes**.


```r
# view a summary of metadata & attributes associated with the spatial object
summary(sjer_plot_locations)
## Error in summary(sjer_plot_locations): object 'sjer_plot_locations' not found
```


# Plot a Shapefile
Next, let's visualize the data in our `R` `spatialpointsdataframe` object using
`plot()`.


```r
# create a plot of the shapefile
# 'pch' sets the symbol
# 'col' sets point symbol color
plot(sjer_plot_locations, col="blue",
     pch=8,
     main="SJER Plot Locations\nMadera County, CA")
## Error in plot(sjer_plot_locations, col = "blue", pch = 8, main = "SJER Plot Locations\nMadera County, CA"): object 'sjer_plot_locations' not found
```

<div class="notice--warning" markdown="1">

## <i class="fa fa-pencil-square-o" aria-hidden="true"></i> Test your knowledge: Import Line & Polygon Shapefiles

Using the steps above, import the `data/week_03/california/madera-county-roads/tl_2013_06039_roads`
and `data/week_03/california/SJER/vector_data/SJER_crop.shp` shapefiles into
`R`. Call the roads object `sjer_roads` and the crop layer
`sjer_crop_extent`.

Answer the following questions:

1. What type of `R` spatial object is created when you import each layer?
2. What is the `CRS` and `extent` for each object?
3. Do the files contain, points, lines or polygons?
4. How many spatial objects are in each file?
</div>


```
## Error in ogrInfo(dsn = dsn, layer = layer, encoding = encoding, use_iconv = use_iconv, : Cannot open data source
## Error in ogrInfo(dsn = dsn, layer = layer, encoding = encoding, use_iconv = use_iconv, : Cannot open data source
## Error in eval(expr, envir, enclos): object 'sjer_roads' not found
## Error in eval(expr, envir, enclos): object 'sjer_plot_locations' not found
## Error in crs(sjer_roads): object 'sjer_roads' not found
## Error in extent(sjer_roads): object 'sjer_roads' not found
## Error in crs(sjer_plot_locations): object 'sjer_plot_locations' not found
## Error in extent(sjer_plot_locations): object 'sjer_plot_locations' not found
## Error in eval(expr, envir, enclos): object 'sjer_roads' not found
## Error in eval(expr, envir, enclos): object 'sjer_roads' not found
## Error in eval(expr, envir, enclos): object 'sjer_roads' not found
## Error in eval(expr, envir, enclos): object 'sjer_roads' not found
## Error in eval(expr, envir, enclos): object 'sjer_roads' not found
```

## Plot Multiple Shapefiles
The `plot()` function can be used to plot spatial objects. Use the following
arguments to add a title to your plot and to layer several spatial objects
on top of each other in your plot.

* `add = TRUE`: overlay a shapefile or raster on top the existing plot. This argument mimics layers in a typical GIS application like QGIS.
* `main=""`: add a title to the plot. To add a line break to your title, use `\n` where the line break should occur.


```r
# Plot multiple shapefiles
plot(sjer_crop_extent, col = "lightgreen",
     main="NEON Harvard Forest\nField Site")
## Error in plot(sjer_crop_extent, col = "lightgreen", main = "NEON Harvard Forest\nField Site"): object 'sjer_crop_extent' not found
plot(sjer_roads, add = TRUE)
## Error in plot(sjer_roads, add = TRUE): object 'sjer_roads' not found

# Use the pch element to adjust the symbology of the points
plot(sjer_plot_locations,
  add  = TRUE,
  pch = 19,
  col = "purple")
## Error in plot(sjer_plot_locations, add = TRUE, pch = 19, col = "purple"): object 'sjer_plot_locations' not found
```


<div class="notice--warning" markdown="1">

## <i class="fa fa-pencil-square-o" aria-hidden="true"></i> Optional challenge: Import & plot roads shapefile

Import the madera-county-roads layer. Plot the roads.

Next, try to plot the roads on top of the
SJER crop extent. What happens?

* Check the CRS of both layers. What do you notice?

</div>

<div class="notice--info" markdown="1">

## Additional resources: Plot Parameter Options
For more on parameter options in the base `R` `plot()` function, check out these
resources:

* <a href="http://www.statmethods.net/advgraphs/parameters.html" target="_blank">Parameter methods in `R`.</a>
* <a href="https://codeyarns.files.wordpress.com/2011/07/20110729-vim-named-colors.png?w=700" target="_blank">Color names in `R`</a>

</div>
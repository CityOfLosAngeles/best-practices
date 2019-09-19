# Working with Geospatial Data: Advanced

Place matters. After covering the [intermediate tutorial](./spatial-analysis-intermediate.md), you're ready to cover some advancaed spatial analysis topics. 

Below are short demos of __________ . 
* [Types of Geometric Shapes](#types-of-geometric-shapes)
* use WKT to write to database, serialization


## Getting Started 

```
# Import Python packages
import pandas as pd
import geopandas as gpd
from shapely.geometry import Point
from geoalchemy2 import WKTElement
```

## Types of Geometric Shapes
There are six possible geometric shapes that are represented in geospatial data. [More description here.](http://postgis.net/workshops/postgis-intro/geometries.html#representing-real-world-objects)
* Point
* MultiPoint: collection of points
* LineString
* MultiLineString: collection of linestrings, which are disconnected from each other
* Polygon
* MultiPolygon: collection of polygons, which can be disconnected or overlapping from each other

The ArcGIS equivalent of these are just points, lines, and polygons.

## Geometry in Geopandas vs 

*** (Notes for this section): Write about in-memory (geopandas) vs WKB/WKT? When is shapely used vs when is geoalchemy used? Is there difference between SRID and EPSG? ***

If you're loading a GeoDataFrame (gdf), having the `geometry` column is necessary to do spatial operations in your Python session. The `geometry` column is composed of Shapely objects, such as Point or MultiPoint, LineString or MultiLineString, and Polygon or MultiPolygon.


Databases often store geospatial information as well-known text (WKT) or its binary equivalent, well-known binary (WKB). These are well-specified interchange formats for the importing and exporting of geospatial data. Often, querying a database (PostGIS, SpatiaLite, etc) or writing data to the database requires converting the `geometry` column to/from WKT/WKB.

The spatial referencing system identifier (SRID) is the <b>geographic coordinate system</b> of the latitude and longitude coordinates. As you are writing the coordinates into WKT/WKB, don't forget to set the SRID. WGS84 is a common geographic coordinate system to use, which provides latitude and longitude in decimal degrees. The SRID for WGS84 is 4326. [Refresher on geographic coordinated system vs projected coordinated system.](./spatial-analysis-basics.md)

``` 
# Set the SRID
srid = 4326
df = df.dropna(subset=['lat', 'lon'])
df["geometry"] = df.apply(
    lambda x: WKTElement(Point(x.lon, x.lat).wkt, srid=srid), axis = 1
)

```

## WKTElement, etc etc. 

*** (Notes for this section): shapely vs geoalchemy, their use cases for working with geometry. Maybe this section gets folded into the one above ***

* Writing to database with WKT?
* Use shapely if in gpd

<br>

[« Previous](./spatial-analysis-intermediate.md)
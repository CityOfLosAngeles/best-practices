# Advanced: Working with Geospatial Data

Place matters. After covering the [intermediate tutorial](./spatial-analysis-intermediate.md), you're ready to cover some advancaed spatial analysis topics. 

Below are short demos of __________ . 
* [Types of Geometric Shapes](#Types-of-Geometric-Shapes)
* 
* use WKT to write to database, serialization


## Getting Started 

```
# Import Python packages
import pandas as pd
import geopandas as gpd
from shapely.geometry import Point
from sqlalchemy import create_engine
from geoalchemy2 import Geometry, WKTElement
```

## Types of Geometric Shapes
There are six possible geometric shapes that are represented in geospatial data. [More description here.](http://postgis.net/workshops/postgis-intro/geometries.html#representing-real-world-objects)
* Point
* MultiPoint: collection of points
* Linestring
* MultiLineString: collection of linestrings
* Polygon
* MultiPolygon: collection of polygons

The ArcGIS equivalent of these are just points, lines, and polygons.

## Geometry in Geopandas vs 
If you're loading a gdf, then having the `geometry` column is necessary to do spatial operations in your Python session. The `geometry` column is composed of Shapely objects, such as Point or MultiPoint, Linestring or MultiLineString, and Polygon or MultiPolygon.

But, in writing a gdf to a database, such as PostGIS or ____, you must use WKT? WKT stands for well-known text, and is _____. Text is better for writing to database? 

The spatial referencing system identifier (SRID) is the <b>geographic coordinate system</b> of the latitude and longitude coordinates. 

``` 
# Set the SRID
srid = 4326
df = df.dropna(subset=['lat', 'lon'])
df["geometry"] = df.apply(
    lambda x: WKTElement(Point(x.lon, x.lat).wkt, srid=srid), axis = 1
)

```

## WKTElement, etc etc. 
* Writing to database with WKT?
* Use shapely if in gpd

<br>

[« Previous](./spatial-analysis-intermediate.md)
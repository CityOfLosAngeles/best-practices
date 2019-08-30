# Intro: Working with Geospatial Data

Place matters. That's why data analysis often includes a geospatial or geographic component. Data analysts are called upon to merge tabular and geospatial data, count the number of points within given boundaries, and create a map illustrating the results.   

Below are short demos of common techniques to help get you started with exploring and visualizing your geospatial data. 
* [Importing/Exporting Data in Python](#Importing/Exporting-Data-in-Python)
* [Merging Tabular and Geospatial Data](#Merging-Tabular-and-Geospatial-Data)
* [Attaching geographic characteristics to all points/lines that fall within a boundary (spatial join/dissolve)](#Attaching-geographic-characteristics-to-all-points/lines-that-fall-within-a-boundary-(spatial-join/dissolve))
* [Aggregating and calculating summary statistics](#Aggregating-and-calculating-summary-statistics)
* [Buffers](#Buffers)


## Importing/Exporting Data in Python
```
# Import Python packages
import pandas as pd
import geopandas as gpd
```

### <b> Local files </b>
We import a tabular dataframe `my_csv.csv` and a geodataframe `my_geojson.geojson` or `my_shapefile.shp`. 
```
df = pd.read_csv(../folder/my_csv.csv)

# GeoJSON
gdf = gpd.read_file(../folder/my_geojson.geojson)
gdf.to_file(driver = 'GeoJSON', filename = '../folder/my_geojson.geojson' )


# Shapefile (collection of files: .shx, .shp, .prj, .dbf, etc)
# The collection files must be put into a folder before importing
gdf = gpd.read_file(../folder/my_shapefile/)
gdf.to_file(driver = 'ESRI Shapefile', filename = '../folder/my_shapefile.shp' )
```

### <b> S3 </b>
Our team often uses Amazon S3 as a bucket storage. To access data in S3, you'll have to have AWS access credentials stored at `~/.aws/credentials` per the [documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html). 

To read in our dataframe (df) and geodataframe (gdf) from S3: 

```
df = pd.read_csv('s3://bucket-name/my_csv.csv')
gdf = gpd.read_file('s3://bucket-name/my_geojson.geojson')

gdf.to_file(driver = 'GeoJSON', filename = 's3://bucket-name/my_geojson.geojson')
```

## Merging Tabular and Geospatial Data
We might have two files: Council District boundaries (geospatial) and population values (tabular). Through visual inspection, we know that `CD` and `District` are columns that help us make this match.

Our dataframe looks like this:
| CD | Council_Member | Population |
| ---| ---- | --- |
| 1 | Leslie Knope | 1,500 |
| 2 | Jeremy Jamm | 2,000 
| 3 | Douglass Howser | 2,250

Our geodataframe looks like this:
| District | Geometry 
| ---| ---- | 
| 1 |  polygon 
| 2 |  polygon 
| 3 |  polygon 

We could merge these two dfs using the District and CD columns. If our left df is the gdf, then our merged df will also be a gdf.
```
merge = pd.merge(gdf, df, left_on = 'District', right_on = 'CD')
merge
```

| District | Geometry | CD | Council_Member | Population
| ---| ---- | --- | --- | --- | --- 
| 1 | polygon | 1 | Leslie Knope | 1,500
| 2 | polygon | 2 | Jeremy Jamm | 2,000
| 3 | polygon | 3 | Douglass Howser | 2,250

## Attaching geographic characteristics to all points/lines that fall within a boundary (spatial join/dissolve)
Sometimes with a point shapefile (list of lat/lon points), we want to count how many points fall within the boundary. Unlike the previous example, these points aren't attached with Council District information, so we need to generate that ourselves.

The ArcGIS equivalent of this is a <b> spatial join </b> between the point and polygon shapefiles, then <b> dissolving </b> to calculate summary statistics.

```
locations = gpd.read_file('../folder/paunch_burger_locations.geojson)
gdf = gpd.read_file('../folder/council_boundaries.geojson)

# Make sure both our gdfs are projected to the same coordinate reference system (EPSG:4326 = WGS84)
locations = locations.to_crs({'init':'epsg:4326'})
gdf = gdf.to_crs({'init':'epsg:4326'})
```

`locations` lists the Paunch Burgers locations and their annual sales. 
| Store | City | Sales_millions | Geometry | 
| ---| ---- | --- | --- |
| 1 | Pawnee  | $5 | (x1,y1) 
| 2 | Pawnee | $2.5 | (x2, y2)
| 3 | Pawnee  | $2.5 | (x3, y3) 
| 4 | Eagleton  | $2 | (x4, y4)  
| 5 | Pawnee  | $4 | (x5, y5)  
| 6 | Pawnee  | $6 | (x6, y6)  
| 7 | Indianapolis  | $7 | (x7, y7)

`gdf` is the Council District boundaries. 
| District  | Geometry
| ---| ---- | 
| 1 | polygon |
| 2 | polygon
| 3 | polygon

A spatial join finds the Council District the location falls within and attaches that information. 

```
join = gpd.sjoin(locations, gdf, how = 'inner', op = 'intersects)

# how = 'inner' means that we only want to keep observations that matched, i.e locations that were within the council district boundaries.
# op = 'intersects' means that we are joining based on whether or not the location intersects with the council district.
``` 

The `join` gdf looks like this. We lost Stores 4 (Eagleton) and 7 (Indianapolis) because they were outside of Pawnee City Council boundaries.
| Store | City | Sales_millions | Geometry_x | District | Geometry_y 
| ---| ---- | --- | --- | --- | ---|
| 1 | Pawnee  | $5 | (x1,y1) | 1 | polygon
| 2 | Pawnee | $2.5 | (x2, y2) | 2 | polygon
| 3 | Pawnee  | $2.5 | (x3, y3) | 3 | polygon
| 5 | Pawnee  | $4 | (x5, y5) | 1 | polygon
| 6 | Pawnee  | $6 | (x6, y6) | 2 | polygon


## Aggregating and calculating summary statistics
We want to count the number of Paunch Burger locations and their total sales within each District.

```
summary = join.pivot_table(index = ['District', 'Geometry_y], 
    values = ['Store', 'Sales_millions'], 
    aggfunc = {'Store': 'count', 'Sales_millions': 'sum}).reset_index()

OR

summary = join.groupby(['District', 'Geometry_y']).agg({'Store': 'count', 
    'Sales_millions': 'sum}).reset_index()

summary.rename(column = {'Geometry_y': 'Geometry'}, inplace = True)
summary
```
| District | Store | Sales_millions|  Geometry  
| ---| ---- | --- | --- | --- |
| 1 | 2 | $9 | polygon
| 2 | 2 | $8.5 | polygon 
| 3 | 1 | $2.5 | polygon 

By keeping the `Geometry` column, we're able to export this as a GeoJSON or shapefile to map in ArcGIS or QGIS.

```
summary.to_file(driver = 'GeoJSON', 
    filename = '../folder/pawnee_sales_by_district.geojson')

summary.to_file(driver = 'ESRI Shapefile', 
    filename = '../folder/pawnee_sales_by_district.shp')
```

## Buffers 
Buffers are areas of a certain distance around a given point, line, or polygon. A 5 mile buffer around a point would be a circle of 5 mile radius centered at the point.

We start with two point shapefiles: `locations` (Paunch Burger locations) and `homes` (home addresses for my 2 friends). The goal is to find out how many Paunch Burgers are located within a 2 miles of my friends.

`locations`: Paunch Burger locations 
| Store | City | Sales_millions | Geometry | 
| ---| ---- | --- | --- |
| 1 | Pawnee  | $5 | (x1,y1) 
| 2 | Pawnee | $2.5 | (x2, y2)
| 3 | Pawnee  | $2.5 | (x3, y3) 
| 4 | Eagleton  | $2 | (x4, y4)  
| 5 | Pawnee  | $4 | (x5, y5)  
| 6 | Pawnee  | $6 | (x6, y6)  
| 7 | Indianapolis  | $7 | (x7, y7)


`homes`: friends' addresses 
| Name |  Geometry  
| ---| ---- | --- | 
| Leslie Knope | (x1, y1) 
| Ann Perkins | (x2, y2)  

First, prepare our point gdf and change it to the right projection. Pawnee is in Indiana, so we'll use EPSG:2965. In southern California, use EPSG:2229.

```
# Use NAD83/Indiana East projection (units are in feet)
homes = homes.to_crs({'init':'epsg:2229'})
locations = locations.to_crs({'init':'epsg:2229'})
```

Next, draw a 2 mile buffer around `homes`.
```
# Make a copy of the homes gdf
homes_buffer = homes.copy()

# Overwrite the existing geometry and change it from point to polygon
miles_to_feet = 5280
two_miles = 2 * miles_to_feet
homes_buffer['geometry] = homes.geometry.buffer(two_miles)
```

### <b> Selecting points within a buffer </b>

Do a spatial join between `locations` and `homes_buffer`. Repeat the process of spatial join and aggregation in Python as illustrated in the previous section (spatial join and dissolve in ArcGIS).

```
sjoin = gpd.sjoin(locations, homes_buffer, how = 'inner', op = 'intersects)
sjoin
```

`sjoin` looks like this. 
* Geometry_x is the point geometry from our left df `locations`.
* Geometry_y is the polygon geometry from our right df `homes_buffer`.

| Store | Geometry_x | Name | Geometry_y | 
| ---| ---- | --- | --- |
| 1 | (x1,y1)   | Leslie Knope | polygon
| 3 | (x3, y3)  | Ann Perkins |  polygon
| 5 | (x5, y5)    | Leslie Knope | polygon
| 6 | (x6, y6)   | Leslie Knope |  polygon

Count the number of Paunch Burger locations for each friend.

```
count = sjoin.pivot_table(index = 'Name', 
    values = 'Store', aggfunc = 'count).reset_index()

OR 

count = sjoin.groupby('Name').agg({'Store':'count}).reset_index()
```

The final `count`:
| Name | Store |
| ---| ---- | 
| Leslie Knope | 3
| Ann Perkins | 1

# Intermediate: Working with Geospatial Data

Place matters. After covering the [intro tutorial](./spatial-analysis-intro.md), you're ready to take your spatial analysis to the next level. 

Below are short demos of other common manipulations of geospatial data. 
* [Create geometry column from latitude and longitude coordinates](#Create-geometry-column-from-latitude-and-longitude-coordinates)
* [Use a loop to do spatial joins and aggregations over different boundaries](#Use-a-loop-to-do-spatial-joins-and-aggregations-over-different-boundaries)
* [More complicated explanation of geometry (WKTElement and WKBElement)](#More-complicated-geometry)
* [Multiple geometry columns](#Multiple-geometry-columns)
* use WKT to write to database, serialization
* use shapely to use in gpd, geometry column uses shapely objects


## Getting Started 
```
# Import Python packages
import pandas as pd
import geopandas as gpd
from shapely.geometry import Point

df = pd.read_csv('../folder/pawnee_businesses.csv')
```

| Business | X | Y | Sales_millions
| ---| ---- | --- | ---| 
| Paunch Burger | x1 | y1 | 5
| Sweetums | x2 | y2 | 30
| Jurassic Fork | x3 | y3 | 2
| Gryzzl | x4 | y4 | 40


## Create geometry column from latitude and longitude coordinates 
Sometimes, latitude and longitude coordinates are given in a tabular form. The file is read in as a dataframe (df), but it needs to be converted into a geodataframe (gdf). The `geometry` column contains a Shapely object (point, line, or polygon), and is what makes it a <b>geo</b>dataframe. A gdf can be exported as GeoJSON or shapefile.

In ArcGIS/QGIS, this is equivalent to adding XY data, selecting the columns that correspond to latitude and longitude, and exporting the layer as a shapefile.

First, drop all the points that are potentially problematic (NAs or zeroes).

```
# Drop NAs
df = df.dropna(subset=['X', 'Y'])

# Keep non-zero values for X, Y
df = df[(df.X != 0) & (df.Y != 0)]
```

Then, create the `geometry` column.  We use a lambda function and apply it to all rows in our df. For every row, take the XY coordinates and make it Point(X,Y). Make sure you set the projection (coordinate reference system)!

```
# Create geometry column
df['geometry'] = df.apply(
    lambda row: Point(row.X, row.Y), axis=1)

# Rename columns
df.rename(columns = {'X': 'longitude', 'Y':'latitude'}, inplace=True)

# Set projection. Pawnee is in Indiana, so we'll use EPSG:2965. In southern California, use EPSG:2229.
df = df.to_crs({'init':'epsg:2965'})

df
```

| Business | longitude | latitude | Sales_millions | geometry
| ---| ---- | --- | ---| ---| 
| Paunch Burger | x1 | y1 | 5 | Point(x1, y1)
| Sweetums | x2 | y2 | 30 | Point(x2, y2)
| Jurassic Fork | x3 | y3 | 2 | Point(x3, y3) 
| Gryzzl | x4 | y4 | 40 | Point(x4, y4)



## Use a loop to do spatial joins and aggregations over different boundaries
Let's say we want to do a spatial join between `df` to 2 different boundaries. Different government departments often use different boundaries for their operations (i.e. city planning districts, water districts, transportation districts, etc). Looping over dictionary items would be an efficient way to do this.

We want to count how the number of stores and total sales within each Council District and Planning District. 

`df`: list of Pawnee stores 

| Business | longitude | latitude | Sales_millions | geometry
| ---| ---- | --- | ---| ---| 
| Paunch Burger | x1 | y1 | 5 | Point(x1, y1)
| Sweetums | x2 | y2 | 30 | Point(x2, y2)
| Jurassic Fork | x3 | y3 | 2 | Point(x3, y3) 
| Gryzzl | x4 | y4 | 40 | Point(x4, y4)

`council_district` and `planning_district` are polygon shapefiles; `df` is a point shapefile. For simplicity, `council_district` and `planning_district` both use column `ID` as the unique identifier.


```
# Save the dataframes into dictionaries
boundaries = {'council': council_district, 'planning': planning_district}

# Create empty dictionaries to hold our results
results = {}


# Loop over different boundaries (council, planning)
for key, value in boundaries.items():
    # Define new variables using f string
    join_df = f"{key}_join"
    agg_df = f"{key}_summary"
    # Spatial join, but don't save it into the results dictionary
    join_df = gpd.sjoin(df, value, how = 'inner', op = 'intersects')
    # Aggregate and save results into results dictionary
    results[agg_df] = join_df.groupby('ID').agg(
        {'Business': 'count', 'Sales_millions': 'sum})
```

Our results dictionary contains 2 dataframes: `council_summary` and `planning_summary`. We can see the contents of the results dictionary using this:
```
for key, value in results.items():
    display(key)
    display(value.head())


# To access the "dataframe", write this:
results["council_summary"].head()
results["planning_summary].head()
```

`council_summary` would look like this, with the total count of Business and sum of Sales_millions within the council district:
| ID | Business | Sales_millions  
| ---| ---- | --- |
| 1 | 2 | 45 
| 2 | 1 | 2  
| 3 | 1 | 30 


## Multiple geometry columns
Sometimes we want to iterate over different options, and we want to see the results side-by-side. Here, we draw multiple buffers around `df`, specifically, 100 ft and 200 ft buffers.

```
# Make sure our projection has US feet as its units
df.to_crs({'init': 'epsg:2965'})

# Add other columns for the different buffers
df['geometry100] = df.geometry.buffer(100)
df['geometry200'] = df.geometry.buffer(200)

df
```
| Business | Sales_millions | geometry | geometry100 | geometry200
| ---| ---- | --- | ---| ---| --- | --- |
| Paunch Burger | 5 | Point(x1, y1) | polygon | polygon
| Sweetums | 30 | Point(x2, y2) | polygon | polygon
| Jurassic Fork  | 2 | Point(x3, y3) | polygon | polygon
| Gryzzl | 40 | Point(x4, y4) | polygon | polygon


To create a new gdf with just 100 ft buffers, select the relevant geometry column, `geometry100`, and set it as the geometry of the gdf.

```
df100 = df[['Business', 'Sales_millions', 
    'geometry100']].set_geometry('geometry100')
```
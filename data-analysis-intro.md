# Data Analysis: Intro

Python tutorials for the basics of data cleaning and wrangling abound. [Chris Albon's guide](https://chrisalbon.com/#python) is particularly helpful. Rather than reinventing the wheel, this tutorial instead highlights specific methods and operations that might make your life easier as a data analyst. 

* [Merging tabular and geospatial data](#Merging-tabular-and-geospatial-data)
* [Grouping](#Grouping)
* [Aggregating](#Aggregating)
* [Exporting aggregated output](#Exporting-aggregated-output)

## Getting Started

```
import numpy as np
import pandas as pd
import geopandas as gpd
```

Refer to the [Data Management best practices](./data-management.md) to get started importing various file types.


## Merging tabular and geospatial data
Merging data from multiple sources creates one large dataframe (df) to perform data analysis. Let's say there are 3 sources of data that need to be merged:

Dataframe #1: `council_population` (tabular)

| CD | Council_Member | Population |
| ---| ---- | --- |
| 1 | Leslie Knope | 1,500 |
| 2 | Jeremy Jamm | 2,000 
| 3 | Douglass Howser | 2,250


Dataframe #2: `paunch_locations` (geospatial)

| Store | City | Sales_millions | CD | Geometry |
| ---| ---- | --- | --- | --- |
| 1 | Pawnee  | $5 | 1|  (x1,y1) 
| 2 | Pawnee | $2.5 | 2 | (x2, y2)
| 3 | Pawnee  | $2.5 | 3 | (x3, y3) 
| 4 | Eagleton  | $2 | | (x4, y4)  
| 5 | Pawnee  | $4 | 1 | (x5, y5)  
| 6 | Pawnee  | $6 | 2 | (x6, y6)  
| 7 | Indianapolis  | $7 | | (x7, y7)


If `paunch_locations` did not come with the council district information, use a spatial join to attach the council district within which the store falls. More on spatial joins [here](./spatial-analysis-intro.md).


Dataframe #3: `council_boundaries` (geospatial)

| District | Geometry 
| ---| ---- | 
| 1 |  polygon 
| 2 |  polygon 
| 3 |  polygon 


First, merge `paunch_locations` with `council_population` using the `CD` column they have in common. 

```
merge1 = pd.merge(paunch_locations, council_population, on = 'CD', 
    how = 'inner', validate = 'm:1')

# m:1 many-to-1 merge means that CD appears multiple times in 
paunch_locations, but only once in council_population.
```

Next, merge `merge1` and `council_boundaries`. Use CD and District as the column to match on.

```
merge2 = pd.merge(merge1, council_boundaries, left_on = 'CD', right_on = 'District', 
    how = 'left', validate = 'm:1')
```

`merge2` is a geodataframe (gdf) because the <i><b> base, </i></b> `paunch_locations`, is a gdf. `merge2` looks like this:

| Store | City | Sales_millions | CD | Geometry_x | Council_Member | Population | Geometry_y
| ---| ---- | --- | --- | --- | ---| ---| ---|
| 1 | Pawnee  | $5 | 1|  (x1,y1) | Leslie Knope | 1,500 | polygon
| 2 | Pawnee | $2.5 | 2 | (x2, y2) | Jeremy Jamm | 2,000 | polygon
| 3 | Pawnee  | $2.5 | 3 | (x3, y3) | Douglass Howser | 2,250 | polygon
| 5 | Pawnee  | $4 | 1 | (x5, y5) | Leslie Knope | 1,500 | polygon
| 6 | Pawnee  | $6 | 2 | (x6, y6)  | Jeremy Jamm | 2,000 | polygon


## Grouping
Sometimes it's necessary to create a new column to group together certain values of a column. Here are two ways to accomplish this:

Method #1: Write a function using if-else statement and apply it using a lambda function.

```
# The function is called elected_year, and it operates on every row.
def elected_year(row):
    # For each row, if Council_Member says 'Leslie Knope', then return 2012 as the value.
    if row.Council_Member == 'Leslie Knope':
        return 2012
    elif row.Council_Member == 'Jeremy Jamm':
        return 2008
    elif row.Council_Member == 'Douglass Howser':
        return 2006

# Use a lambda function to apply the elected_year function to all rows in the df. 
# Don't forget axis = 1 (apply function to all rows)!
council_population['Elected'] = council_population.apply(lambda row: 
    elected_year(row), axis = 1)

council_population
```

| CD | Council_Member | Population | Elected
| ---| ---- | --- | --- |
| 1 | Leslie Knope | 1,500 | 2012
| 2 | Jeremy Jamm | 2,000 | 2008
| 3 | Douglass Howser | 2,250 | 2006


Method #2: Loop over every value, fill in the new column value, then attach that new column.

```
# Create a list to store the new column
sales_group = []


for row in paunch_locations['Sales_millions']:
    # If sales are more than $3M, but less than $5M, tag as moderate.
    if (row >= 3) & (row <= 5) :
        sales_group.append('moderate')
    # If sales are more than $5M, tag as high.
    elif row >=5:
        sales_group.append('high')
    # Anything else, aka, if sales are less than $3M, tag as low. 
    else:
        sales_group.append('low')

paunch_locations['sales_group'] = sales_group

paunch_locations
```

| Store | City | Sales_millions | CD | Geometry | sales_group 
| ---| ---- | --- | --- | --- | --- |
| 1 | Pawnee  | $5 | 1|  (x1,y1) | moderate
| 2 | Pawnee | $2.5 | 2 | (x2, y2) | low
| 3 | Pawnee  | $2.5 | 3 | (x3, y3) | low
| 4 | Eagleton  | $2 | | (x4, y4) | low
| 5 | Pawnee  | $4 | 1 | (x5, y5) | moderate
| 6 | Pawnee  | $6 | 2 | (x6, y6) | high  
| 7 | Indianapolis  | $7 | | (x7, y7) | high


## Aggregating
One of the most common form of summary statistics is aggregating by groups. In Excel, it's called a pivot table. In ArcGIS, it's doing a dissolve and calculating summary statistics. There are two ways to do it in Python: `groupby` and `agg` or `pivot_table`.

To answer the question of how many Paunch Burger locations there are per Council District and the sales generated per resident, 

```
# Method #1: groupby and agg
pivot = merge2.groupby(['CD', 'Geometry_y']).agg({'Sales_millions': 'sum',
     'Store': 'count', 'Population': 'mean'}).reset_index()

# Method #2: pivot table
pivot = merge2.pivot_table(index= ['CD', 'Geometry_y'], 
    values = ['Sales_millions', 'Store', 'Population'], 
    aggfunc= {'Sales_millions': 'sum', 'Store': 'count', 'Population': 'mean'}).reset_index()

    # to only find one type of summary statistic, use aggfunc = 'sum'

# reset_index() will compress the headers of the table, forcing them to appear in 1 row rather than 2 separate rows 
```

`pivot` looks like this:

| CD | Geometry_y | Sales_millions | Store | Council_Member | Population 
| ---| ---- | --- | --- | --- | ---| ---| 
| 1 | polygon  | $9 | 2 | Leslie Knope | 1,500 
| 2 | polygon | $8.5 | 2 | Jeremy Jamm | 2,000
| 3 | polygon  | $2.5 | 1 | Douglass Howser | 2,250 


## Exporting aggregated output
Python can do most of the heavy lifting for data cleaning, transformations, and general wrangling. But, for charts or tables, it might be preferable to finish in Excel or ArcGIS/QGIS so that visualizations conform to the corporate style guide. 

Dataframes can be exported into Excel and written into multiple sheets.

```
import xlsxwriter

## init a writer 
writer = pd.ExcelWriter('../outputs/filename.xlsx', engine='xlsxwriter')

council_population.to_excel(writer, sheet_name = 'council_pop')
paunch_locations.to_excel(writer, sheet_name = 'paunch_locations')
merge2.to_excel(writer, sheet_name = 'merged_data')
pivot.to_excel(writer, sheet_name = 'pivot')

# Close the Pandas Excel writer and output the Excel file.
writer.save()
```

Geodataframes can be exported as a shapefile or GeoJSON to visualize in ArcGIS/QGIS.
```
gdf.to_file(driver = 'ESRI Shapefile', filename = '../folder/my_shapefile.shp' )

gdf.to_file(driver = 'GeoJSON', filename = '../folder/my_geojson.geojson')
```
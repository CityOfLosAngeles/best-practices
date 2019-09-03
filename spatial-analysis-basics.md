# Basics: Working with Geospatial Data

Place matters. That's why data analysis often includes a geospatial or geographic component. 

Below are short demos of the basics to get started working with geospatial data. 
* [Importing and Exporting Data in Python](#Importing-and-Exporting-Data-in-Python)
* [Setting and Projecting Coordinate Reference System](#Setting-and-Projecting-Coordinate-Reference-System)

## Getting Started

```
# Import Python packages
import pandas as pd
import geopandas as gpd
```

## Importing and Exporting Data in Python
### <b> Local files </b>
We import a tabular dataframe `my_csv.csv` and a geodataframe `my_geojson.geojson` or `my_shapefile.shp`. 
```
df = pd.read_csv('../folder/my_csv.csv')

# GeoJSON
gdf = gpd.read_file('../folder/my_geojson.geojson')
gdf.to_file(driver = 'GeoJSON', filename = '../folder/my_geojson.geojson' )


# Shapefile (collection of files: .shx, .shp, .prj, .dbf, etc)
# The collection files must be put into a folder before importing
gdf = gpd.read_file('../folder/my_shapefile/')
gdf.to_file(driver = 'ESRI Shapefile', filename = '../folder/my_shapefile.shp' )
```

### <b> S3 </b>
Data can also be stored in an Amazon S3 as a bucket storage. To access data in S3, you'll have to have AWS access credentials stored at `~/.aws/credentials` per the [documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html). 

To read in our dataframe (df) and geodataframe (gdf) from S3: 

```
df = pd.read_csv('s3://bucket-name/my_csv.csv')
gdf = gpd.read_file('s3://bucket-name/my_geojson.geojson')
gdf = gpd.read_file('zip+s3://bucket-name/my-shapefile.zip')


gdf.to_file(driver = 'GeoJSON', filename = 's3://bucket-name/my_geojson.geojson')
```

## Setting and Projecting Coordinate Reference System
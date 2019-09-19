# Data Analysis: Intermediate

After polishing off the [intro tutorial](./data-analysis-intro.md), you're ready to devour some more techniques to simplify your life as a data analyst. 

* <a href = "data-analysis-intermediate.md#Looping-over-columns-with-a-dictionary">Looping over columns with a dictionary</a>
* [Looping over columns with a dictionary](data-analysis-intermediate##Looping-over-columns-with-a-dictionary)
* [Looping over dataframes with a dictionary](data-analysis-intermediate##Looping-over-dataframes-with-a-dictionary)


## Getting Started

```
import numpy as np
import pandas as pd
import geopandas as gpd
```

## Looping over columns with a dictionary
If there are operations or data transformations that need to be performed on multiple columns, the best way to do that is with a loop.

```
columns = ['colA', 'colB', 'colC']

for c in columns:
    # Fill in missing values for all columns with zeros
    df[c] = df[c].fillna(0)
    # Multiply all columns by 0.5
    df[c] = df[c] * 0.5
```

## Looping over dataframes with a dictionary
It's easier and more efficient to use a loop to do the same operations over the different dataframes (df). Here, we want to find the number of Pawnee businesses and Tom Haverford businesses are located in each Council District. 

This type of question is perfect for a loop. Each df is spatially joined `council_district`, followed by some aggregation. 

`business`: list of Pawnee stores 

| Business | longitude | latitude | Sales_millions | geometry
| ---| ---- | --- | ---| ---| 
| Paunch Burger | x1 | y1 | 5 | Point(x1, y1)
| Sweetums | x2 | y2 | 30 | Point(x2, y2)
| Jurassic Fork | x3 | y3 | 2 | Point(x3, y3) 
| Gryzzl | x4 | y4 | 40 | Point(x4, y4)


`tom`: list of Tom Haverford businesses

| Business | longitude | latitude | Sales_millions | geometry
| ---| ---- | --- | ---| ---| 
| Tom's Bistro | x1 | y1 |30 | Point(x1, y1)
| Entertainment 720 | x2 | y2 | 1 | Point(x2, y2)
| Rent-A-Swag | x3 | y3 | 4 | Point(x3, y3) 


```
# Save our existing dfs into a dictionary. The business df is named 'pawnee"; the tom df is named 'tom'. 
dfs = {'pawnee': business, 'tom': tom}

# Create an empty dictionary called summary_dfs to hold the results
summary_dfs = {}

# Loop over key-value pair. 
## Keys: pawnee, tom (names given to dataframes)
## Values: business, tom (dataframes)

for key, value in dfs.items():
    # Use f string to define a variable join_df (result of our spatial join)
    ## join_{key} would be join_pawnee or join_tom in the loop
    join_df = "join_{key}"
    # Spatial join
    join_df = gpd.sjoin(value, council_district, how = 'inner', op = 'intersects')
    # Calculate summary stats with groupby, agg, then save it into summary_dfs, naming it 'pawnee' or 'tom'.
    summary_dfs[key] = join.groupby('ID').agg(
        {'Business': 'count', 'Sales_millions': 'sum'})
```

Now, our `summary_dfs` dictionary contains 2 items, which are the 2 dataframes with everything aggregated. 

```
# To view the contents of this dictionary
for key, value in summary_dfs.items():
    display(key)
    display(value)

# To access the df
summary_dfs["pawnee"]
summary_dfs["tom"]
```

`join_tom`: result of spatial join between tom and council_district

| Business | longitude | latitude | Sales_millions | geometry | ID
| ---| ---- | --- | ---| ---| --- |
| Tom's Bistro | x1 | y1 | 30 | Point(x1, y1) | 1
| Entertainment 720 | x2 | y2 | 1 | Point(x2, y2) | 3
| Rent-A-Swag | x3 | y3 | 4 | Point(x3, y3) | 3


`summary_dfs["tom"]`: result of the counting number of Tom's businesses by CD
| ID | Business | Sales_millions  
| ---| ---- | --- |
| 1 | 1 | 30 
| 3 | 2 | 5 

 

<br>

[« Previous](./data-analysis-intro.md)
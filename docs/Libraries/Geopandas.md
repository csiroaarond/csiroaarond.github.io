Installation
```
pip install geopandas
```

Import
```
import geopandas as gpd
```

Read in spatial data into GeoDataFrame (Pandas Dataframe that can perform spatial operations on geospatial data, these operations are performed onto the geometry attribute of the geodataframe)
```
import os
data_pkg_path = 'data'
filename = 'packagename.gpkg'
path = os.path.join(data_pkg_path, filename)
gdf = gpd.read_file(path, layer='nameoflayer')
gdf.info()
gdf.geometry
```

Layer example: roads [What are Layers?](../Terminologies/Layers.md)

Filter Data
```
gdf_filtered = gdf[gdf['columnname'].str.match('^startofstringinfield') == True]
```

Projections [What are projections?](../Terminologies/Projections.md)

View Coordinate Reference System (CRS)
```
gdf_filtered.crs
```

To change Projected CRS (e.g. EPSG:4326 to EPSG:32643)
```
gdf_reprojected = gdf_filtered.to_crs('EPSG:32643')
```

This is performed to compute line lengths, if the task involves calculate something like roads lengths.

If gdf_filtered only contains rows with national highways.
```
gdf_filtered = gdf[gdf['ref'].str.match('^NH') == True]
```

```
gdf_reprojected['length'] = gdf_reprojected['geometry'].length
```

Length of the national highways an be performed by applying the sum formula on the length attribute.

```
total_length = roads_reprojected['length'].sum()
print('Total length of national highways in the state is {} KM'.format(total_length/1000))
```

Combining Datasets

In this example, combine major roads layer with districts.

Combine the datasets using spatial joins. sjoin(), takes two arguments, op (the spatial predicate to decide which objects join: intersects, within, contains) and how (the type of join: left, right, inner).
```
districts_gdf = gpd.read_file(path, layer='districts')
districts_reprojected = districts_gdf.to_crs('EPSG:32643')
joined = gpd.sjoin(gdf_reprojected, districts_reprojected, how='left', op='intersects')
```

With combined major roads and districts, the resulting geodataframe now has the combined data with districts to be able to find the sum of all major roads within each district.

```
results = joined.groupby('DISTRICT')['length'].sum()/1000
print(results)
```

Output the file via...
```
output_filename = 'national_highways_by_districts.csv'
output_dir = 'output'
output_path = os.path.join(output_dir, output_filename)
results.to_csv(output_path)
print('Successfully written output file at {}'.format(output_path))
```



### Creating Spatial Data

[GeoNames Dataset](https://www.geonames.org/export/)
 
Read in data with column name assignment
```
column_names = [
    'geonameid', 'name', 'asciiname', 'alternatenames', 
    'latitude', 'longitude', 'feature class', 'feature code',
    'country code', 'cc2', 'admin1 code', 'admin2 code',
    'admin3 code', 'admin4 code', 'population', 'elevation',
    'dem', 'timezone', 'modification date'
]

df = pd.read_csv(path, sep='\t', names=column_names)
```

Get subset of data containing only  mountains by filtering 'feature class' equal to 'T'
```
mountains = df[df['feature class']=='T']
print(mountains.head()[['name', 'latitude', 'longitude', 'dem','feature class']])
```

Result:
```
                 name  latitude  longitude   dem feature class
15       Vulcan Point  52.10222  177.53889 -9999             T
16     Tropical Ridge  51.99167  177.50833   267             T
17  Thirty-Seven Hill  52.84528  173.15278   193             T
20       Square Point  52.86120  173.33679    30             T
21       Square Bluff  51.65000  178.70000 -9999             T
```

Create the geometry column from the x and y coordinates (longitude and latitude). Then convert the DataFrame into a GeoDataFrame, specifying the CRS (projection) and the new geometry column.

```
geometry = gpd.points_from_xy(mountains.longitude, mountains.latitude)
gdf = gpd.GeoDataFrame(mountains, crs='EPSG:4326', geometry=geometry)
```

Export geodataframe into gpkg file.

```
output_dir = 'output'
output_filename = 'mountains.gpkg'
output_path = os.path.join(output_dir, output_filename)

gdf.to_file(driver='GPKG', filename=output_path, encoding='utf-8')
print('Successfully written output file at {}'.format(output_path))
```

This gpkg file can be viewed as a layer in qgis.
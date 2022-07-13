[RasterIO](https://rasterio.readthedocs.io/en/latest/) is a modern library to work with geospatial data in a gridded format. It excels at providing an easy way to read/write raster data and access individual bands and pixels as `numpy` arrays.

RasterIO is built on top of the popular [GDAL (Geospatial Data Abstraction Library)](https://gdal.org/). GDAL is written in C++ so the Python API provided by GDAL is not very intuitive for Python users. RaserIO aims to make it easy for Python users to use the underlying GDAL library in an intuitive way.

![[srtm.png]]

Import raster data
```
import rasterio
import os
data_pkg_path = 'data'
srtm_dir = 'srtm'
filename = 'N28E087.hgt'
path = os.path.join(data_pkg_path, srtm_dir, filename)
```

Read raster data (can read any format supported by gdal library)
```
dataset = rasterio.open(path)
```

Check information on raster via meta attribute
```
metadata = dataset.meta
```
 Metadata:
```
{
	'driver': 'SRTMHGT',
	 'dtype': 'int16',
	 'nodata': -32768.0,
	 'width': 3601,
	 'height': 3601,
	 'count': 1,
	 'crs': CRS.from_epsg(4326),
	 'transform': Affine(0.0002777777777777778, 0.0, 86.99986111111112,
			0.0, -0.0002777777777777778, 29.000138888888888)
}
```

Read the pixel values by selecting a band of the raster data. From the metadata.count value, there is only 1 band available in the data. So just read band 1, normally index band from 1, but there is only 1.
```
band1 = dataset.read(1)
```

```
[[5217 5211 5208 ... 5097 5098 5089]
 [5206 5201 5200 ... 5080 5075 5069]
 [5199 5194 5191 ... 5063 5055 5048]
 ...
 [5347 5345 5343 ... 5747 5750 5757]
 [5338 5338 5336 ... 5737 5740 5747]
 [5332 5331 5332 ... 5734 5736 5744]]
```

When finished with dataset, close.
```
dataset.close()
```

Merge datasets (merge 4 tiles together, and "mosaic them together"). 
Find all the individual files in the directory using the `os.listdir()` function.
```
srtm_path = os.path.join(data_pkg_path, 'srtm')
all_files = os.listdir(srtm_path)
```

```
['N28E086.hgt', 'N28E087.hgt', 'N27E087.hgt', 'N27E086.hgt']
```

Open each file and add to a list.
```
dataset_list = []
for file in all_files:
    path = os.path.join(srtm_path, file)
    dataset_list.append(rasterio.open(path))
```

Use the merge method to merge every tile in the list into a single geoTiff. The data and the transform is saved to separate variables
```
from rasterio import merge
merged_result = merge.merge(dataset_list)

merged_data = merged_result[0]
merged_transform = merged_result[1]
```

Verify the array shape is sum of all rasters.
```
merged_data.shape
```

Write the raster data. Metadata needs to be specified, some can be copied directly from the files (crs, dtype, nodata), but some are taken from the merged data such as height and width.
```
output_filename = 'merged.tif'
output_dir = 'output'
output_path = os.path.join(output_dir, output_filename)

new_dataset = rasterio.open(output_path, 'w', 
                            driver='GTiff',
                            height=merged_data.shape[1],
                            width=merged_data.shape[2],
                            count=1,
                            nodata=-32768.0,
                            dtype=merged_data.dtype,
                            crs='+proj=latlong',
                            transform=merged_transform)
new_dataset.write(merged_data)
new_dataset.close()
```

![[Pasted image 20220713102507.png]]



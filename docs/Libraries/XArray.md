[Xarray](http://xarray.pydata.org/) is an evolution of rasterio and is inspired by libraries like pandas to work with raster datasets. It is particularly suited for working with multi-dimensional time-series raster datasets. It also integrates tightly with [dask](https://dask.org/) that allows one to scale raster data processing using parallel computing.

[rioxarray](https://corteva.github.io/rioxarray/stable/index.html) is an extension of xarray that makes it easy to work with geospatial rasters.

Read data
```
import os
data_pkg_path = 'data'
srtm_dir = 'srtm'
filename = 'N28E087.hgt'
path = os.path.join(data_pkg_path, srtm_dir, filename)

import rioxarray as rxr
rds = rxr.open_rasterio(path)
rds.values
```

```
array([[[5217., 5211., 5208., ..., 5097., 5098., 5089.],
        [5206., 5201., 5200., ..., 5080., 5075., 5069.],
        [5199., 5194., 5191., ..., 5063., 5055., 5048.],
        ...,
        [5347., 5345., 5343., ..., 5747., 5750., 5757.],
        [5338., 5338., 5336., ..., 5737., 5740., 5747.],
        [5332., 5331., 5332., ..., 5734., 5736., 5744.]]], dtype=float32)
```

```
rds.coords
```

```
Coordinates:
  * band         (band) int64 1
  * x            (x) float64 87.0 87.0 87.0 87.0 87.0 ... 88.0 88.0 88.0 88.0
  * y            (y) float64 29.0 29.0 29.0 29.0 29.0 ... 28.0 28.0 28.0 28.0
    spatial_ref  int64 0
```

Slice main dataset, and get band1 using sel()
```
band1 = rds.sel(band=1)
```

Raster data is stored in rio.
```
print('CRS:', rds.rio.crs)
print('Resolution:', rds.rio.resolution())
print('Bounds:', rds.rio.bounds())
print('Width:', rds.rio.width)
print('Height:', rds.rio.height)
```

Merge rasters (same as using rasterIO but using xarray and rio)
```
import rioxarray as rxr
from rioxarray.merge import merge_arrays
```

```
srtm_path = os.path.join(data_pkg_path, 'srtm')
all_files = os.listdir(srtm_path)
output_filename = 'merged.tif'
output_dir = 'output'
output_path = os.path.join(output_dir, output_filename)

datasets = []
for file in all_files:
    path = os.path.join(srtm_path, file)
    datasets.append(rxr.open_rasterio(path))
	
merged = merge_arrays(datasets)
merged.rio.to_raster(output_path)
```


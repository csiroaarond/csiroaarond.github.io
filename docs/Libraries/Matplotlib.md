### Visualising Rasters with Matplotlib

```
%matplotlib inline
import matplotlib.pyplot as plt
```

 Here we create a row of 4 plots and render each of the source SRTM rasters. We can use the `cmap` option to specify a color ramp. Here we are using the built-in _Greys_ ramp. Appending **_r** gives us the inverted ramp with blacks representing lower elevation values.
```
fig, axes = plt.subplots(1, 4)
fig.set_size_inches(15,3)
plt.tight_layout()
for index, dataset in enumerate(datasets):
    ax = axes[index]
    dataset.plot(ax=ax, cmap='Greys_r')
    ax.axis('off')
    filename = all_files[index]
    ax.set_title(filename)
```
![[Pasted image 20220713150816.png]]

Plotting the merged dataset...
```
fig, ax = plt.subplots()
fig.set_size_inches(12, 10)
merged.plot(ax=ax, cmap='Greys_r')
ax.set_title('merged')
plt.show()
```

![[Pasted image 20220713150804.png]]

### Creating Maps with GeoPandas

Import datasets into geopandas geodataframe.
```
import geopandas as gpd
import os
data_pkg_path = 'data'
filename = 'karnataka.gpkg'
path = os.path.join(data_pkg_path, filename)
districts = gpd.read_file(path, layer='karnataka_districts')
roads = gpd.read_file(path, layer='karnataka_major_roads')
national_highways = roads[roads['ref'].str.match('^NH') == True]
```

Load matplotlib
```
%matplotlib inline
import matplotlib.pyplot as plt
```

Setup the plot figure and axes.
-   Figure: This is the main container of the plot. A figure can contain multiple plots inside it
-   Axes: Axes refers to an individual plot or graph. A figure contains 1 or more axes.

```
fig, axes = plt.subplots(1, 3)
fig.set_size_inches(15,7)
```

subplots returns the figure and a tuple containing all the axes within the figure. These 3 axes can be unpacked into separate variables.
```
ax0, ax1, ax2 = axes
```

Render the districts layer from the loaded datasets into axis 0. This can be referenced directly using the ax parameter in the plot function.
```
districts.plot(ax=ax0, linewidth=1, facecolor='none', edgecolor='#252525')
fig
```

![[Pasted image 20220713154509.png]]

Render the roads layer on axis 1.
```
roads.plot(ax=ax1, linewidth=0.4, color='#2b8cbe')
fig
```

![[Pasted image 20220713154600.png]]

Render the national highways layer on axis 2.

```
national_highways.plot(ax=ax2, linewidth=1, color='#de2d26')
fig
```

![[Pasted image 20220713154737.png]]

Save map as png.
```
output_filename = 'map_layout.png'
output_dir = 'output'
output_path = os.path.join(output_dir, output_filename)

if not os.path.exists(output_dir):
    os.mkdir(output_dir)

fig.savefig(output_path, dpi=300)
```

Create map with multiple layers. Add all plots to same axis.

```
fig, ax = plt.subplots()
fig.set_size_inches(10,15)

plt.axis('off')

districts.plot(ax=ax, linewidth=1, facecolor='none', edgecolor='#252525')
roads.plot(ax=ax, linewidth=0.4, color='#2b8cbe')
national_highways.plot(ax=ax, linewidth=1, color='#de2d26')


output_filename = 'multiple_layers.png'
output_path = os.path.join(output_dir, output_filename)
plt.savefig(output_path, dpi=300)
```

![[Pasted image 20220713161411.png]]

Labelling features and annotating the map

We can also add labels to the maps, but that requires a bit of pre-processing. Let's say we want to add a label for each of the distrit polygons. First, we need to decide the anchor position of the label. We can use `representative_point()` to get a point inside each polygon that best represents the geometry. It is similar to a centroid, but is guranteed to be inside the polygon. Below code creates a new field in the GeoDataFrame called `label_position` with the coordinates of the anchor point.

```
districts['label_position'] = districts['geometry'].apply(lambda x: x.representative_point().coords[:])
districts['label_position'] = [coords[0] for coords in districts['label_position']]
```

Use the `annotate()` function and iterate over each polygon to add labels with the name of the district from the _DISTRICT_ column and place it at the coordinates from the _label_position_ column.

```
fig, ax = plt.subplots(figsize=(10, 15))
plt.axis('off')

districts.plot(ax=ax, linewidth=1, facecolor='none', edgecolor='#252525')

for idx, row in districts.iterrows():
    plt.annotate(text=row['DISTRICT'], xy=row['label_position'], horizontalalignment='center')
```

![[Pasted image 20220713161618.png]]
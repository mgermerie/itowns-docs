
# The [`View`](add link)

The entry point for using iTowns is a [`View`](add link), which supports data visualisation. Among other things, the
[`View`] implements a [`Scene`](add link) and a [`Camera`](add link) from three.js. A [`View`](add link) also
instantiates a [`MainLoop`](add link), one of whose roles is to create a [`Renderer`](add link) and frequently call its
`render` method from the [`View`](add link)'s [`Scene`](add link) and [`Camera`](add link), each time a frame is to be 
rendered.



# Data structure

To support data that are to be visualized, we create [`Layers`](add link) that we add to a [`View`](add link). 
Different types of [`Layers`](add link) exist, depending on which type of data they support. We therefore have :
- [`GeometryLayer`](add link) to display vector data.
- [`RasterLayer`](add link) to display original raster data or rasterized vector data.

In the end of the data displaying process, which will be described later, a [`GeometryLayer`](add link) will display 
data as a set of [`Object3D`](add link). A [`RasterLayer`](add link) on another hand will display data as a set of
[`Texture`](add link).


Among these two main [`Layers`](add link) types, sub-types exist :
- [`GeometryLayer`](add link)
	- [`TiledGeometryLayer`](add link) : the [`Object3D`](add link) this is a [`GeometryLayer`](add link) 
	- [`FeatureGeometryLayer`](add link)
	- [`3DTileLayer`](add link)
	- [`PointCloudLayer`](add link)
	- [`OrientedImageLayer`](add link)
- [`RasterLayer`](add link) :
	- [`ColorLayer`](add link)
	- [`ElevationLayer`](add link)


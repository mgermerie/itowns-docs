_*NOTE : link to diagram
https://lucid.app/lucidspark/e705a67f-c803-4148-8c39-5d2798e323ae/edit?viewport_loc=-886%2C-168%2C3840%2C2160%2C0_0&invitationId=inv_fc06d978-ce8e-4623-adac-d2160c5d404di *_


# Sources

ITowns comes with a [`Source`]() class as well as many  classes
inherited from it. A [`Source`]() holds methods to retrieve data in a specific
format using a specific protocol.

- [`WFSSource`](), [`WMSSource`](), [`TMSSource`]() and [`WMTSSource`]() allow
	downloading data from respectively WFS, WMS, TMS and WMTS caches. The format
	of the data retrieved from these caches is specified with the `format`
	property of the [`Source`]() instances.
- [`VectorTilesSource`]() allows downloading vector tiles from [mapbox]().
- [`FileSource`]() allows retrieving data from a single given file, with its
	format specified with the `format` property of the [`Source`]() instance.
- _TODO : check the standard for Oriented Images format_
- [`PotreeSource`]() is for downloading data that is in the [potree file
	format](https://github.com/PropellerAero/potree-propeller-private/blob/master/docs/file_format.md).
- [`EntwinePointTileSource`]() allows retrieving data from an [EPT (Entwine
	Point Tile)]() cache.
- [`C3DTilesSource`]() allows retrieving 3d tiles data, and
	[`C3DTilesIonSource`]() allows retrieving them from a [Cesium Ion]() server.

## Fetching and parsing

Each source implements a `fetcher` and a `parser` methods, depending on the
format of the data it aims to retrieve.

The `fetcher` allows to fetch raw data from its origin - which can be a server
using TMS protocol, or a file stored locally for example. It should be noted
that the `fetcher` used for images reads them as three.js [`Texture`]().

The `parser` transforms fetched data into a pivot format. A bunch of `parsers`
are already implemented in iTowns, the output format of which varies :

- [`GeoJsonParser`](), [`KMLParser`](), [`GPXParser`](), [`ShapefileParser`](),
	and [`VectorTileParser`]() parse data - from the formats mentioned in the
	methods names - into a [`FeatureCollection`]().
- [`LASParser`](), [`PotreeBinParser`](), [`PotreeCinParser`]() and
	[`PntsParser`]() parse points from data into a three.js
	[`BufferGeometry`]().
- [`GDFParser`](), [`GTXParser`]() and [`ISGParser`]() are build for specific
	geoid grid formats - GDF, GTX and ISG. They parse data into a
	[`GeoidGrid`]().
- _*TODO : add [`B3dmParser`]()*_

The `parser` used for images is a simple identity function, which thus returns the
[`Texture`]() created by the `fetcher`.

## Cache management in the source

_*TODO : complete this part*_

Each source also implements a `loadData` method which, when called, runs the
[`Source`]()'s `fetcher`, then its `parser`. It stores the `parser` result in
the [`Source`]()'s cache.


# Layers

To support data that are to be visualized, we create [`Layers`]() that we add to
a [`View`](). Different types of [`Layers`]() exist, depending on which type of
data they support. We therefore have :
- [`GeometryLayer`]() to display vector data.  [`RasterLayer`]() to display
- original raster data or rasterized vector data.

In the end of the data displaying process implemented by [`Layers`]() - which is
not described here, a [`GeometryLayer`]() will display data as a set of
[`Object3D`](). A [`RasterLayer`]() on another hand will display data
as a set of [`Texture`]().

Among these two main [`Layers`]() types, sub-types exist.

## Sub-types of [`GeometryLayer`]()

### - [`TiledGeometryLayer`]()

The [`Object3D`]() displayed with a [`TiledGeometryLayer`]() represent a tiled
terrain. Each tile from this terrain is a [`TileMesh`](), which inherits
[three.js `Mesh`]().

Data, in the form of other types of [`Layers`](), can be attached to a
[`TiledGeometryLayer`](). It means that the data are subdivided into data nodes,
with one node for each tile of the terrain. In other words the spatial
subdivision of the terrain is applied to the data.

A [`TiledGeometryLayer`]() can have multiple attached [`Layers`](), but a
[`Layer`]() can be attached to only one [`TiledGeometryLayer`]().

One of the assets of the [`TiledGeometryLayer`]() is that a [`TileMesh`]() that
composes it is a kind of hub gathering multiple data through multiple
[`Layers`](). The tile's visibility, transformation matrix and other attributes
can then be applied to all data attached to it.

### - [`FeatureGeometryLayer`]()

A [`FeatureGeometryLayer`]() is a preconfigured [`GeometryLayer`]() made to be
attached to a [`TiledGeometryLayer`]().

### - [`C3DTilesLayer`]()

A [`3DTilesLayer`]() can be used to support the display of [3d tiles]() data.

### - [`PointCloudLayer`]()

### - [`OrientedImageLayer`]()

## Sub-types of [`RasterLayer`]()

### - [`ColorLayer`]()

### - [`ElevationLayer`]()

## Data processing within layers

_*TODO : ADD INFO ABOUT THE UPDATE (AND CONSEQUENTLY CONVERT) METHOD IN EACH
LAYER.*_



# The [`View`]()

A [`View`]() is a support on which data is visualized in iTowns, making it the
entry point for using iTowns. It implements a [`Scene`]() and a [`Camera`]()
from three.js.

A [`View`]() also instantiates a [`MainLoop`](), one of whose roles is to
instantiate a three.js [`Renderer`](). A [`MainLoop`]() also implements two
methods : [`step`]() and [`scheduleViewUpdate`]().

When called, the [`step`]() goes through a selection of [`Layers`]() that need
updating and calls their `update` method. This calling is done recursively
among the [`Layers`]() and their attached [`Layers`]() : we first call the
`update` of [`Layers`]() which are not attached to any other [`Layer`](). We
then call it for each [`Layers`]() attached to these "level 0 [`Layers`]()", and
so on for the subsequently attached [`Layers`](). After updating [`Layers`](),
the [`step`]() method asks the [`Renderer`]() to render the [`View`]()'s
`scene`.

The [`scheduleViewUpdate`]() method of the [`MainLoop`]() is used to ask the
browser to execute the [`step`]() method at its next available frame. This is
done using [`requestAnimationFrame`](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame).

A [`View`]() implements a `notifyChange` method which tells the [`MainLoop`]()
to run [`scheduleViewUpdate`](). Its first parameter allows to specify which
elements need to be updated in the [`step`]() method :
- it can be a [`Layer`](), in which case only this [`Layer`]() will be updated
- it can be a [`Camera`](), in which case all [`Layers`]() will be updated.

If no parameter is specified, all [`Layers`]() are updated.




_*NOTE : Schedules a step on next frame, each step calls MainLoopEvents callback, update
of layers, if needs redraw, call render.*_


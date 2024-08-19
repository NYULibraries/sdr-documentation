# GeoServer Layer Management

## Environments

NYU has the following GeoServer instances:

### Staging

- [Staging Public GeoServer](http://maps-public-stage.library.nyu.edu/geoserver/web/)
- [Staging Restricted GeoServer](http://maps-restricted-stage.library.nyu.edu/geoserver/web/)

### Production

- [Production Public GeoServer](https://maps-public.geo.nyu.edu/geoserver/web/)
- [Production Restricted GeoServer](https://maps-restricted.geo.nyu.edu/geoserver/web/)

## Convert Shapefile into GeoServer Layer

The following sections show you how to make a shapefile available as a GeoServer layer via a PostGIS table.

For the following steps you will need the following tools installed:

- `ogr2ogr` - provided by [gdal]()
- `shp2pgsql` - provided by [PostGIS]()
- `psql` - provided by [PostgreSQL]()

On macOS you can install these using [Homebrew](https://brew.sh):

```bash
$ brew install gdal
$ brew install postgis
$ brew install postgresql
```

### Re-project Vector Data to EPSG:4326 (WGS84)

First we need to re-project the vector data to EPSG:4326 (WGS84).

To view the Shapefile's current CRS, use the following command and look for the `SOURCECRS` entry.

```bash
$ ogrinfo -so -al original.shp 
```

To re-project the Shapefile to EPSG:4326 use the following command:

```bash
$ ogr2ogr -t_srs EPSG:4326 -lco ENCODING=UTF-8 reprojected.shp original.shp
```

### Convert Shapefile to SQL Statements

Next we need to convert our Shapefile into a file of SQL statements that will create a table and populate it with
records:

```bash
$ shp2pgsql -I -s 4326 reprojected.shp table_name > table_name.sql
```

### Import Vector Data into PostGIS

Finally, we run the SQL file against the PostGIS database:

```bash
$ psql --host=HOSTNAME_HERE --username=USERNAME_HERE --dbname=geospatial_data --file=table_name.sql
```

### Publish a PostGIS Table as a GeoServer Layer

After logging into your desired GeoServer instance, do the following:

1. Under **Data** click on **Layers**.
2. Click on **Add a new layer**.
3. In **Add layer from** select `sdr:vector_postgis`.
4. In the list of available layers, filter for the layer you want to add.
5. Click **Publish** next to the layer you want to publish.
6. Scroll down to **Bounding Boxes**:
    1. For **Native Bounding Box** click **Compute from data**.
    2. For **Lat/Lon Bounding Box** click **Compute from native bounds**.
7. Click **Save**.
8. In the sidebar click on **Layer Review**.
9. Search for your layer in the resulting list.
10. Click on **OpenLayers** in that result's row.

## Unpublish a GeoServer Layer

After logging into your desired GeoServer instance, do the following:

1. Under **Data** click on **Layers**.
2. Use the search box to find the Layer you want to remove
3. Check the box in the first column of the row.
4. Click **Remove selected layers**.
5. Click **OK** in the confirmation dialog.

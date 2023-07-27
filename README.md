# Overture Maps

https://github.com/OvertureMaps/data

Getting started:

```
duckdb
LOAD spatial;
LOAD httpfs;
SET s3_region='us-west-2';
```

23-07-26

Inspecting Overture Maps database 

AOI (5.1, 52.1, 5.2, 52.2):

<img width="428" alt="Screenshot 2023-07-26 at 21 45 13" src="https://github.com/bertt/overture/assets/538812/6bc09b8e-9358-4ac1-aa9d-be38eaee7f44">

## Buildings

785524851 buildings

height in est_height?

```
COPY (
SELECT ST_GeomFromWkb(geometry) AS geometry, JSON(names) AS names, JSON(height) as height
from  read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=buildings/type=*/*', filename=true, hive_partitioning=1)
where bbox.minX > 5.1 and bbox.maxX < 5.2 and bbox.minY>52.1 and bbox.maxY<52.2 
) TO 'buildings.geojson'
WITH (FORMAT GDAL, DRIVER 'GeoJSON');
```

result: [buildings.geojson](buildings.geojson)

## Places

2049 features / 59175720 

```
COPY (
SELECT ST_GeomFromWkb(geometry) AS geometry, JSON(names) AS names
from  read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=places/type=*/*', filename=true, hive_partitioning=1)
where bbox.minX > 5.1 and bbox.maxX < 5.2 and bbox.minY>52.1 and bbox.maxY<52.2 
) TO 'places.geojson'
WITH (FORMAT GDAL, DRIVER 'GeoJSON');
```

result: [places.geojson](places.geojson)

## Transportation

624933749 features

## Admins

99403 features


```
COPY (
SELECT ST_GeomFromWkb(geometry) AS geometry, JSON(names) AS names
from  read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=admins/type=*/*', filename=true, hive_partitioning=1)
) TO 'admins.geojson'
WITH (FORMAT GDAL, DRIVER 'GeoJSON');
```

## Downloading data

With aws:

```
$ aws s3 cp --recursive --region us-west-2 --no-sign-request s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=transportation/
```

warning 70+GB data!

after that convert to GeoJSON:

```
$ duckdb
D load spatial;
D SELECT count(ST_GeomFromWkb(geometry)) from read_parquet('./type=segment/*');
D COPY (
SELECT ST_GeomFromWkb(geometry) AS geometry from  read_parquet('./type=segment/*')
) TO 'transport_segment.geojson'
WITH (FORMAT GDAL, DRIVER 'GeoJSON');
D COPY (
SELECT ST_GeomFromWkb(geometry) AS geometry from  read_parquet('./type=connector/*')
) TO 'transport_connector.geojson'
WITH (FORMAT GDAL, DRIVER 'GeoJSON');
```

Describe:

```
D select * from read_parquet('./type=segment/*') limit 10;
┌──────────────────────┬──────────────────────┬─────────┬───┬──────────────────────┬──────────────────────┬─────────┐
│          id          │      updatetime      │ version │ . │         bbox         │       geometry       │  type   │
│       varchar        │      timestamp       │  int32  │   │ struct(minx double.  │         blob         │ varchar │
├──────────────────────┼──────────────────────┼─────────┼───┼──────────────────────┼──────────────────────┼─────────┤
│ segment.87269954df.  │ 2023-07-15 01:06:5.  │       0 │ . │ {'minx': -107.5931.  │ \x01\x02\x00\x00\x.  │ segment │
│ segment.8627a0ac7f.  │ 2023-07-15 01:06:5.  │       0 │ . │ {'minx': -104.3583.  │ \x01\x02\x00\x00\x.  │ segment │
│ segment.862aa81a7f.  │ 2023-07-15 01:06:5.  │       0 │ . │ {'minx': -76.59282.  │ \x01\x02\x00\x00\x.  │ segment │
│ segment.8d1e24b9b9.  │ 2023-07-15 01:06:5.  │       0 │ . │ {'minx': 16.815531.  │ \x01\x02\x00\x00\x.  │ segment │
│ segment.871f14618f.  │ 2023-07-15 01:06:5.  │       0 │ . │ {'minx': 7.9296541.  │ \x01\x02\x00\x00\x.  │ segment │
│ segment.885883932b.  │ 2023-07-15 01:06:5.  │       0 │ . │ {'minx': 3.7815252.  │ \x01\x02\x00\x00\x.  │ segment │
│ segment.882a1639c1.  │ 2023-07-15 01:06:5.  │       0 │ . │ {'minx': -74.59577.  │ \x01\x02\x00\x00\x.  │ segment │
│ segment.873c1e8d0f.  │ 2023-07-15 01:06:5.  │       0 │ . │ {'minx': 86.196555.  │ \x01\x02\x00\x00\x.  │ segment │
│ segment.892e6e2e0a.  │ 2023-07-15 01:06:5.  │       0 │ . │ {'minx': 134.25871.  │ \x01\x02\x00\x00\x.  │ segment │
│ segment.8b380b14e2.  │ 2023-07-15 01:06:5.  │       0 │ . │ {'minx': 3.6438104.  │ \x01\x02\x00\x00\x.  │ segment │
├──────────────────────┴──────────────────────┴─────────┴───┴──────────────────────┴──────────────────────┴─────────┤
│ 10 rows                                                                                      11 columns (6 shown) │
└───────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
D select * from read_parquet('./type=connector/*') limit 10;
┌──────────────────────┬──────────────────────┬─────────┬───┬──────────────────────┬──────────────────────┬───────────┐
│          id          │      updatetime      │ version │ . │         bbox         │       geometry       │   type    │
│       varchar        │      timestamp       │  int32  │   │ struct(minx double.  │         blob         │  varchar  │
├──────────────────────┼──────────────────────┼─────────┼───┼──────────────────────┼──────────────────────┼───────────┤
│ connector.8f3da556.  │ 2023-07-24 00:27:1.  │       0 │ . │ {'minx': 75.935448.  │ \x01\x01\x00\x00\x.  │ connector │
│ connector.8f2ba010.  │ 2023-07-24 00:27:1.  │       0 │ . │ {'minx': -72.21874.  │ \x01\x01\x00\x00\x.  │ connector │
│ connector.8f1ea006.  │ 2023-07-24 00:27:1.  │       0 │ . │ {'minx': 12.223394.  │ \x01\x01\x00\x00\x.  │ connector │
│ connector.8f64ab44.  │ 2023-07-24 00:27:1.  │       0 │ . │ {'minx': 95.749541.  │ \x01\x01\x00\x00\x.  │ connector │
│ connector.8f489a41.  │ 2023-07-24 00:27:1.  │       0 │ . │ {'minx': -96.57796.  │ \x01\x01\x00\x00\x.  │ connector │
│ connector.8f195ab2.  │ 2023-07-24 00:27:1.  │       0 │ . │ {'minx': -3.392677.  │ \x01\x01\x00\x00\x.  │ connector │
│ connector.8f2b834d.  │ 2023-07-24 00:27:1.  │       0 │ . │ {'minx': -77.17431.  │ \x01\x01\x00\x00\x.  │ connector │
│ connector.8f1fad75.  │ 2023-07-24 00:27:1.  │       0 │ . │ {'minx': 10.795418.  │ \x01\x01\x00\x00\x.  │ connector │
│ connector.8f31586e.  │ 2023-07-24 00:27:1.  │       0 │ . │ {'minx': 127.48397.  │ \x01\x01\x00\x00\x.  │ connector │
│ connector.8f1e24aa.  │ 2023-07-24 00:27:1.  │       0 │ . │ {'minx': 16.912270.  │ \x01\x01\x00\x00\x.  │ connector │
├──────────────────────┴──────────────────────┴─────────┴───┴──────────────────────┴──────────────────────┴───────────┤
│ 10 rows                                                                                        11 columns (6 shown) │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```


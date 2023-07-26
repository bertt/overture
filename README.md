# Overture Maps

https://github.com/OvertureMaps/data

23-07-26

Inspecting Overture Maps database 

AOI (5.1, 52.1, 5.2, 52.2):

<img width="428" alt="Screenshot 2023-07-26 at 21 45 13" src="https://github.com/bertt/overture/assets/538812/6bc09b8e-9358-4ac1-aa9d-be38eaee7f44">

## Buildings

785524851 buildings

```
COPY (
SELECT ST_GeomFromWkb(geometry) AS geometry, JSON(names) AS names
from  read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=buildings/type=*/*', filename=true, hive_partitioning=1)
where bbox.minX > 5.1 and bbox.maxX < 5.2 and bbox.minY>52.1 and bbox.maxY<52.2 
) TO 'buildings.geojson'
WITH (FORMAT GDAL, DRIVER 'GeoJSON');
```

## Places

2049 features / 59175720 


[places.geojson](places.geojson)

```
COPY (
SELECT ST_GeomFromWkb(geometry) AS geometry, JSON(names) AS names
from  read_parquet('s3://overturemaps-us-west-2/release/2023-07-26-alpha.0/theme=places/type=*/*', filename=true, hive_partitioning=1)
where bbox.minX > 5.1 and bbox.maxX < 5.2 and bbox.minY>52.1 and bbox.maxY<52.2 
) TO 'places.geojson'
WITH (FORMAT GDAL, DRIVER 'GeoJSON');
```

## Transportation

624933749 features

## Admins

99403 features





# Borders of the monitoring territory

External boundary coverage dataset — Lower Volga and Podonia: Rostov,
Volgograd, Astrakhan regions, west and center of Saratov region,
Republic of Kalmykia. 16 Tops, EPSG:4326plaza 435 thousand km2.

The same polygon in three formats - take any:

    fire_monitoring_aoi.geojson          RFC 7946, UTF-8 encoding
    fire_monitoring_aoi.gpkg             GeoPackage, aoi layer
    fire_monitoring_aoi_shapefile.zip    shp + shx + dbf + prj + cpg

Separate files shp/shx/dbf/prj/cpg lie nearby - in the archive they are collected
together, so as not to lose any of them during the transfer. .cpg file is required:
Without it, QGIS and ArcGIS read dbf in system encoding and spoil the Cyrillic alphabet
attributely.

## What's inside?

Three objects:

| id | what |
|---|---|
| `aoi` | border of monitoring territory |
| `utm_32637` | strip of UTM 37N, west of 42°E. |
| `utm_32638` | UTM 38N, east of 42°E. |

UTM zones are given for convenience: all coverage fits into 37N and 38N
It is easier to choose the desired projection immediately than
project later.

Attributes: `name`, `role`, `subjects`, `seasons` (2019–2025)),
`months` (04–10), `utm_zones`, `area_km2`.

## There's nothing inside

Boundaries of closed blocks of the private test. Public/private division built
A computer-readable file with its geometry would be a direct clue,
What areas to mark manually. In the play, they're called words —
This is enough to understand the split device.

## How do you check?

    python -c "import json; d=json.load(open('fire_monitoring_aoi.geojson', encoding='utf-8')); \
        print(len(d['features'), 'object,', len(d)['features'][0]['geometry']['coordinates'][0]), "tops"

Expected: 3 objects, 17 vertices (the first coincides with the last - the ring is closed)).

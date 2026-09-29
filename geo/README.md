# Geo

`kroma_geo` provides measurements, CRS transformations, raster/vector conversion, geometry verification, clipping and AOI context. Dependencies pyproj, shapely and rasterio declared in `pyproject.toml`.

BS pixel area: 20×20 m = 0.04 ha. TRAIN exports count the area before conversion to WGS84; analysis uses the original contours after pruning of the AOI in equal-size EPSG:6933. Common pixel boundaries of densify prior to reprojection so that neighboring severity zones do not create numerical intersections. Verification of intersections uses STRtree.

Historical contextual modules allow Polygon/MultiPolygon AOI; current service analysis contract accepts bbox+ dates. Real OSM/WorldCover/DEM sources and limitations are described in the [source catalogue](../docs/data/sources.md). TEST geo-reference is prohibited by the project contract.

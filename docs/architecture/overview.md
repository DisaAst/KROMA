# KROMA Architecture

Planned flow:

```text
External satellite/weather/geospatial sources
  → ingestion
  → normalization
  → ML/geo processing
  → PostgreSQL/PostGIS
  → backend API
  → frontend map
```

MODIS, VIIRS and NASA FIRMS thermal detections are supposed to be cleaned and combined into fire events. ML will evaluate confidence, Thermal Memory and priority; geo – geometry, rasters and hare analysis by Sentinel-2 and other sources. External weather and infrastructure data will be inputs to threat assessment. Redis and background tasks can be added when regular data downloads appear.

Project boundaries:

- Research code and production code are separated: notebooks and hypothesis testing are in `research/`, verified implementations are in `ml/src/` or `geo/src/`.
- Data and source code are separated: local datasets are in `data/` and do not commit.
- Geo and ML are separated: coordinate transformations, geometries, rasters and PostGIS preparation refer to `geo/`; conclusions, estimates and models refer to `ml/`.
- Backend and Notebooks Separate: API Provides Results but Does Not Execute Research Code.

At the current stage implemented `GET /health`, the launch page, local configuration of services and a separate normalization CSV NASA FIRMS VIIRS. Regular download, storage and analytics stream not yet connected.

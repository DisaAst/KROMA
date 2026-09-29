# Prepared analysis kits

## Purpose

Backend only reads the prepared results through immutable `manifest.json`. The kit does not access the FIRMS or satellite archive at the time of a user request. One version should give the same geometry, numbers and `result_id` after restart.

Built-in `backend/app/prepared_data` is a small `synthetic_demo` for CI. To demonstrate the quality of the model, you need a separate allowed set with `origin: model_output`; it is forbidden to change the origin of the fixture.

## Catalogue structure

```text
prepared-dataset/
├── manifest.json
├── hotspots.geojson
└── burn_zones.geojson
```

`KROMA_PREPARED_DATA_PATH` can point to this directory, directly to the manifest or to a parent directory with multiple child sets. Backend only downloads versions that have manifest, checksum and content verified.

## Manifest v1

Mandatory field groups:

| Fields | Meaning |
| --- | --- |
| `dataset_id`, `dataset_version`, `schema_version` | Stable identity and immutable version |
| `origin` | Only `model_output`, `reference` or `synthetic_demo` |
| `processing_version` | Balance/algorithm version or launch identifier |
| `available_from`, `available_to`, `extent` | Available period and bbox WGS84 |
| `af_coverage`, `bs_coverage`, `bs_valid_coverage` | Separate coverage of hotspots, mapping and valid BS pixels |
| `source_crs`, `area_crs` | CRS artifact and selected equal-square/metric CRS calculation |
| `mask` | Classes 0/1/2/3, nodata, resolution and validation rule |
| `scenes` | Real Scene IDs, Satellite, UTC Time, Role, Known Clouds |
| `artifacts` | Relative path and SHA-256 of each file |
| `license`, `attribution`, `distribution` | Right to use and limit transfers |
| `example` | Content bbox and dates for UI, Swagger and smoke test |

The path of the artifact cannot leave the manifest catalog. Absolute developer-paths and secrets are not allowed.

## GeoJSON heatpoints

`hotspots.geojson` - `FeatureCollection` points WGS84. Each feature must have a unique `id`, `properties.id`, UTC `acquired_at` and the honest name `source`. Initials like FRP are acceptable, but FRP is not interpreted as area.

Points are filtered by `[from 00:00 UTC, to + 1 день 00:00 UTC)` and turned on at the bbox boundary. Repeated ID is rejected when downloading.

## GeoJson Zones

`burn_zones.geojson` contains only class zones, with no separate complete contour that could double the area. Geometry – Polygon or MultiPolygon WGS84, including holes.

Mandatory properties:

- `id`, `burn_event_id`, `assessment_id`;
- `class_id`: 1, 2 or 3;
- `severity`: `low`, `moderate`, `high`;
- UTC `after_acquired_at`;
- `is_complete`; partial assessment is not selected as full;
- `scene_ids`.

Within one assessment, classes cannot intersect in a positive area. For one `burn_event_id`, the server selects the last full assessment, the date of which was included in the request. The geometry then intersects with the AOI, and only after that is the area counted in `area_crs`.

## Verification

```bash
python -m app.cli.validate_dataset /path/to/prepared-dataset
```

The team checks manifest schema, presence and SHA-256, JSON/GeoJSON, uniqueness of ID, times with timezone, coordinates, permissible classes, validity of geometries and positive class overlaps. The return of `0` means that the set can be published in the directory; `1` - that the backend should not give out a result from it.

After checking:

```bash
curl -fsS http://localhost:8000/api/v1/analysis/readiness
curl -fsS http://localhost:8000/api/v1/analysis/datasets
```

## Raster mask preparation

`kroma_geo.vector.vectorize_class_mask` provides verifiable basic vectorization of small categorical masks 0-3 with affine transform, holes and multiple components. For production rasters, a rasterio/GDAL pipeline is allowed, but its output must undergo the same manifest validator and independent verification of the area with a pixel mask.

Nodata/cloud is not class 0. If there is no valid domain, the API returns `no_valid_data` and `null` to areas rather than false zeros.

## Transfer and update

- You cannot store a single copy of a limited set inside a temporary container. Compose host directory is mounted read-only via `KROMA_PREPARED_DATA_HOST_PATH`.
- Any alteration to the artifact, recalculate SHA-256 and release a new `dataset_version`; do not change the old version in place.
- Do not publish scenes, weights, keys and derivatives unless the license permits it. Organizers are provided with an authorized archive or secure link together with checksum.
- After the restart, repeat catalog → analysis → exports and compare `result_id` and areas.

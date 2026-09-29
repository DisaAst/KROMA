# Territory analysis contract v1

Status: **contract v1 implemented** Pydantic, TypeScript, REST and Contract Tests. Built-in example has origin `synthetic_demo`; connection of real `model_output` remains a prerequisite for final acceptance WEB-002/012. Changes to the contract shall be reflected simultaneously in all layers and in this document.

## 1. Minimum model of interaction

For a limited set, use the synchronous REST API. One query returns hotspots, hare zones, summary and metadata. Exports call for the same server function with the same parameters.

No background jobs, arbitrary user upload or start-up training required. If you exceed documented limits, return an understandable error, and not a cropped result under the guise of a complete one.

## 2. Request parameters

| Field | Format | Rule |
| --- | --- | --- |
| `dataset_id` | string | ID of the prepared set from the catalog |
| `dataset_version` | string | Mandatory immutable version; do not substitute a new version silently |
| `bbox` | `minLon,minLat,maxLon,maxLat` | WGS84, longitude/latitude order, positive width and height |
| `from` | `YYYY-MM-DD` | First Included UTC Calendar Day |
| `to` | `YYYY-MM-DD` | Last included calendar day of UTC |

The server normalizes dates in `[from 00:00 UTC, to + 1 день 00:00 UTC)`. `from <= to`. Minimum version does not support bbox via antimeridian. finite coordinates, without NaN/Infinity, within acceptable limits. The accuracy of the user area is not rounded to three characters, as in an existing viewport client.

For the point to use `acquired_at`. Points on the boundary of bbox include, repeated records with the same ID exclude.

For burns in v1 use **Date of the `after_acquired_at` post-fire image**, mapping. She should be in the period. In the UI and the certificate explicitly write: "Gari are selected by the date of post-fire shooting; this is not the date of the start of the fire". This is the product semantics chosen, not the statement that the criteria require it. If the team chooses a different approach in WEB-001, replace the rule in all layers and tests before integration.

If one hare is rated by several suitable scenes, use the last full score of one `burn_event_id` in the selected period. Do not add up several pictures of the same area. Do not automatically combine partial estimates as complete: manifest must describe their coverage and build rule, otherwise such a set is not yet supported. Missing dates should not be replaced by current time.

## 3. Prepared set

Manifest contains:

- `dataset_id`, `dataset_version`, name, description and fixed spatial coverage.
- Available period and scene list: real ID, satellite, shooting time, footprint, cloudiness/validity, if known.
- `origin`: `model_output`, `reference` or `synthetic_demo`. Separately store the name of the external source of observations; do not pass FIRMS as your own model.
- Algorithm/weight version or launch identifier, method of obtaining results.
- Initial CRS, affine transform/mask resolution, nodata, `0/1/2/3` classes and validity mask, if applicable.
- Ways to Local Artifacts, Checksums and the Load/mount Rule.
- License, Attribution and Distribution Restrictions.
- `burn_event_id → assessment_id → scene_ids` communication; do not invent such a connection for arbitrary unrelated chips.

Demonstration of model results using `model_output`. Reference masks and synthetic fixtures are suitable for inspection, but are marked and not issued as a result of the model.

## 4. Area and geometry

- Mask BS: `0` - non-smoking, `1` - weak, `2` - medium, `3` - strong degree of damage; nodata / cloud is not equal to class 0.
- Vectorize classes with preservation of holes and several components. Support Polygon and MultiPolygon.
- Cross geometry with the query area; count the area only of the resulting parts, not entire intersecting hares.
- There should be no positive intersection between classes. Repetitive results of different scenes should not double the area.
- For a fixed set, select and document a suitable metric/equal-area CRS; bbox translate to it prior to trimming and calculation. Do not count hectare by degrees or screen pixels Web Mercator.
- Area = area of correct cropped geometry in m2 / 10 000. Check its consistency with the original mask and its georeference. The rule for partially crossed pixels is the geometric intersection of their vectored boundaries.
- WGS84 longitude/latitude geometry. Do not add an outdated `crs` field to standard GeoJSON; store the calculation CRS description in JSON metadata/help.
- Do not round to whole hectares within the calculation. UI can be rounded up for reading; API and export retain at least 6 decimal places if decimal rounding is used.
- Total area - the sum of the areas of non-overlapping zones of three classes. Do not add the full contour of burns to this amount again.

## 5. Catalogue and endpoints

| Method | Pathway | The result |
| --- | --- | --- |
| GET | `/api/v1/analysis/datasets` | Available versions, coverage, dates, source, limitations and hints for example |
| GET | `/api/v1/analysis` | One agreed result on mandatory parameters |
| GET | `/api/v1/analysis/export/contours` | GeoJSON zones according to the same parameters |
| GET | `/api/v1/analysis/export/report?format=csv` | CSV reference for the same parameters |
| GET | `/api/v1/analysis/export/report?format=json` | JSON reference and metadata for the same parameters |

Export path receives all parameters from section 2; `format` - additional parameter only reference. In the export request, also transmit `expected_result_id` so that the server detects a discrepancy with the result shown and does not download other data silently.

Old endpoints persist until apparent migration. Do not change their semantics hidden for the sake of a new screen.

## 6. Response to analysis

Minimum fields:

```text
schema_version
result_id
request: dataset_id, dataset_version, bbox, from, to
provenance: origin, processing_version, scene_ids, temporal_rule
coverage:
  query_area_ha, covered_area_ha, valid_area_ha
  status: full | partial | none
  method
summary:
  status: ok | empty | no_coverage | no_valid_data
  hotspot_count, burn_scar_count, zone_count
  total_burned_area_ha
  severity: [{class_id, label, area_ha, share} × 3]
hotspots: GeoJSON FeatureCollection
burn_zones: GeoJSON FeatureCollection
warnings: [{code, message}]
calculation: area_method, area_crs, clipping_rule, deduplication_rule
generated_at
```

`result_id` is determined by normalized request, data checksums, schema version and processing. `generated_at` is not involved in the identifier. Cache should not be the only query repository: it is repeated on a full set of parameters, and after restarting the result is restored from an immutable set.

For a validly surveyed area without detection: `empty`, zero areas/counters, three classes with zero area, empty collections, `share=null` at zero total area.

For no coverage/valid pixels: appropriate status, inaccessible areas and `null` fractions, not zero burnout statement. Thermopoints and BS can have different coatings: communicate the limitations of each source, do not hide the available result of one behind the inaccessibility of the other. WEB-001 clarifies individual source coverage fields when formalizing the scheme.

For partial coverage, consider only the available valid part, display a warning and the size of the examined area. Don't extrapolate to the whole bbox.

If the connection of observations with smokes is not established, do not give the number of clusters of thermal points for `burn_scar_count`.

## 7. GeoJSON attributes

Each feature of the exported layer is a zone of one class, not at the same time a complete contour and its daughter zones:

| Attribute | Appointment |
| --- | --- |
| `id` / `feature.id` | Unique stable zone ID as a result |
| `burn_event_id` | Gary/event identifier for grouping |
| `assessment_id` | Identifier of the selected assessment |
| `class_id` | 1, 2 or 3 |
| `severity` | `low`, `moderate`, `high` |
| `area_ha` | Area of Exported, Already Cut Geometry |
| `after_acquired_at` | Date/time of the image determining the time selection |
| `dataset_id`, `dataset_version` | Source |
| `result_id` | Link to map and reference |

If one zone splits into parts, use MultiPolygon or separate unique ID parts; fix the selected option. `burn_scar_count` is considered for unique non-zero intersection, not by the number of parts.

Thermometers must have at least an ID, time, source and coordinates. Extra confidence/FRP output only if available; do not interpret FRP as an area.

## 8. CSV/JSON reference

CSV: UTF-8, documented separator, decimal point, standard field shielding. One `total` line and three `severity` lines; columns include string type, class_id, name, area_ha, share, counts and request/source/result_id metadata. The general string is clearly distinguished from classes, so that the consumer does not resume the result.

Mandatory metadata: bbox, from/to and UTC, typing and processing version, time selection rule, CRS/calculation method, coverage status and warnings. JSON retains structured metadata. For the CSV, determine a safe representation of a list of scenes/warnings and explain the blank values corresponding to `null`.

The file name is secure and reproducible, such as `kroma-<dataset>-<from>-<to>-<result-id-prefix>.geojson`. Exports give a suitable content type and content-disposition. Do not use the user string directly as a path on the disk.

## 9. Mistakes and limitations

- `422`: Incorrect coordinates, dates, format or documented query limit.
- `404`: Unknown set/version.
- `409`: `expected_result_id` does not match the current calculation.
- `503`: the set is known, but its artifacts are inaccessible/failed integrity testing.
- Valid query without intersections is `200` with `no_coverage`, not an unexplained server error.
- Internal errors do not reveal local paths/secrets. Front receives code and understandable message.

Limits are configured: `KROMA_ANALYSIS_MAX_DAYS=366`, `KROMA_ANALYSIS_MAX_AREA_HA=5000000`, `KROMA_ANALYSIS_MAX_FEATURES=50000`. If exceeded, the server returns 422 without silent trimming. Internal benchmark: one sample analysis no longer than 10 seconds after service start; this is the team's engineering goal, not the numerical threshold of the criteria.

## 10. Invariants

1. Map, Help, GeoJSON and CSV/JSON have one `result_id` and query settings.
2. The sum of the area of classes coincides with the total area; the sum of the attributes of zones – with the corresponding class.
3. The geometries of different classes do not overlap in area, are inside the AOI and are valid.
4. Repeated query and restart on the same version do not change geometry, numbers and `result_id`.
5. Empty observation, lack of coverage and absence of valid pixels are distinguishable.
6. Calculation is independent of map scale, theme, camera position or rounded UI signatures.

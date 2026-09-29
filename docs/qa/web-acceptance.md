# Verification protocol WEB-001-WEB-012

Date of inspection: September 19, 2026. Verified by: Codex, Local Development Environment. This protocol records the inspections actually performed, not the promise of a competitive evaluation.

## Fixed environment

- Git base revision to operating changes: `e3aa1dffce8f`.
- OS: Darwin 25.6.0; Python 3.12; Node.js 22 from the project environment.
- Dataset: `kroma-ci-demo@1.0.0`.
- Origin: `synthetic_demo` is not a model output or real fire observation.
- Processing version: `fixture-vectorizer-1.0.0`.
- Content fingerprint: `3868d08baaf8b97edd9fd2b09a8639907319b92819ccf82e537947017d7661d6`.
- Manifest and SHA-256 tested by `python -m app.cli.validate_dataset`.

## Control request

```text
dataset_id=kroma-ci-demo
dataset_version=1.0.0
bbox=99.03,58.03,99.13,58.13
from=2024-08-10
to=2024-08-20
```

Expected and actual `result_id`: `ar_e4be272a0b16e33c1d61b11c66b0f82c`.

| Meaning | Fact |
| --- | ---: |
| Hotspots | 2 |
| Unique burns | 1 |
| Zones | 3 |
| Weak degree | 986.262700 ha; 0.402741 |
| Medium degree | 1052.019394 ha; 0.429593 |
| Strong degree | 410.592194 ha; 0.167666 |
| Totally | 2448.874288 ha |
| AOI/covered/valid | 6571.466198 ha |

The sum of classes and the sum of `area_ha` exported zones coincide with the total with machine accuracy of the received 6-digit API rounding.

## Automatic checks

Completed successfully:

```text
ruff check .                         PASS
pytest                              44 passed
npm run typecheck                   PASS
npm run lint                        PASS
npm test -- --run                   22 passed
npm run build                       PASS
docker compose --env-file .env.example config --quiet  PASS
```

Complete `docker compose build` in this environment was not counted: Docker daemon was available, but the download of the base `python:3.12-slim` did not show any progress and was interrupted. Production build frontend and Compose configuration verified separately; clean Docker launch remains part of witness for WEB-011.

Covered: UTC and right period boundary, unknown version, NaN/inverted bbox, date limit, latest complete assessment, holes/MultiPolygon, touch and overlap, partial pruning, empty/no coverage/null, checksum mismatch, `result_id` stability, 409 export, MIME/Content-Disposition, CSV and GeoJSON/JSON round-trip.

## Time

On a built-in small set, in a local process:

- Download/validation repository + first analysis: 8.080 ms;
- 100 repeats: average 1.805 ms, maximum 2.079 ms.

It is a measurement of engineering fixture, not a benchmark of real model output or the speed of competitive inferencing.

## Manual browser check

Proven on production build through local backend:

- directory loaded, example filled in bbox/dates;
- The “Calculate” button recorded the applied query in the URL;
- Map, Help and Zones Show One `result_id`;
- AOIs, two hotspots, three distinct classes, legend and layer management;
- The table contains total and all three degrees; origin `synthetic_demo` and constraint are visible;
- After changing the draft west from 99.03 to 99.04, the old map / help remained tied to the previous `result_id`, a warning appeared;
- GeoJSON/CSV/JSON buttons are available; their server responses have passed round-trip tests separately;
- all three export buttons re-call from production frontend: backend received GeoJSON, CSV and JSON requests with the same `expected_result_id` and responded to `200`;
- Popup hotspots and zones are opened on the map and closed with the Escape key; the `moderate` technical signature found during re-checking was replaced by “Mediocre degree of damage” and checked in the reassembled frontend;
- The interface is visually checked in dark and light themes, on the usual desktop viewport and 390×844;
- Key fields and actions are available as label/role, dialog/tooltip is closed by Escape on the map;
- Swagger UI `/docs` opened OAS 3.1 and showed the catalog, readiness, analysis, contours and report with diagrams and examples.

The external substrate is considered independent: if it is mistaken, the UI shows a warning, the server result, reference and export are not dependent on tiles.

## Matrix R1-R3

| Criteria | Engineering Confirmation | Final confirmation |
| --- | --- | --- |
| R1 | Compose/README, mandatory bbox+dates REST, Swagger, deterministic response, time measurement | Requires a clean start by another person and a replay with a real prepared set |
| R2 | Single map, Polygon/MultiPolygon/hole, cropping, class/id/area, GeoJSON round-trip | Requires an independent reconciliation on the allowed model output and open the file in the selected GIS |
| R3 | Total + three classes, null/zero, CSV/JSON, zone equality and reference | Requires the control area of a real mask by an independent method |

## Unclosed terms WEB-002/011/012

The final competitive acceptance is intentionally not marked as completed because the workspace does not transfer the allowed geo-referenced result of the model with real scene IDs, license and independently expected areas. You cannot rename the built-in fixture to `model_output`.

We need protection:

1. Connect immutable manifest to origin `model_output` and transfer the set in an acceptable way.
2. Checksums, processing/weights version, scene IDs, coverage/nodata and independent expected areas.
3. Run Compose in a clean directory by another person; check restart and constant read-only mount.
4. Repeat browser script, cURL/Swagger and files on one real `result_id`; open GeoJSON in an independent GIS.
5. Add actual commit/dataset versions, times and verdict. Only then can WEB-002 and WEB-012 be granted DONE status according to the plan criteria.

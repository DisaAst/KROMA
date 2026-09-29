# Key metrics

## Public leaderboard

| v001 | v002 | v003 | v004 | v005 | GOLD v006 |
| ---: | ---: | ---: | ---: | ---: | ---: |
| 0.5526 | 0.6457 | 0.7298 | 0.7406 | 0.8127 | **0.8176** |

Values confirmed by the team. BS OOF v006: all-valid 0.43552, strict 0.44677. Local TRAIN F1/IoU in-sample is not used as a generalization estimate.

## Testing production inference

Main baseline `bf088ec`; integration 2026-09-19. `python inference.py --data-dir data/test --output tmp/integration-submission.csv --workers 2 --threads 2 --verify-against artifacts/submissions/submission_v006.csv`.

- 447 lines: 180 AF + 267 BS class rows (89 chips × 3)).
- Chip errors: 0. Complete CSV-identical `submission_v006.csv`.
- AF fire pixels: 3977. BS burn pixels: 3,900,362.
- Runtime 452.88 s; neural batch 19.81 s; AF + BS postprocessing 432.49 s; assembly/validation 0.52 s.
- RLE roundtrip, template order/keys, no NaN, no duplicates, no overlap.
- Direct service→RLE: BS_te_000001 / 000020 / 000089 coincided with v006.

[Machine protocol](qa/inference-verification.json). The measurement is local, at parallel load; it is not SLA.

## Service runtime

TRAIN `AF_tr_000001`/`BS_tr_000001`, HTTP on local uvicorn. Cold query includes downloading models/data. Cache stores original model/vectorize timings, `total` reflects current call.

| Operation | Measured |
| --- | ---: |
| AF cold HTTP | 4455.6 ms |
| AF cached HTTP | 5.2 ms |
| BS cold HTTP | 16034.0 ms |
| BS cached HTTP | 69.5 ms |
| BS vectorize | 147.6 ms |
| AF GeoJSON / Shapefile | 5.1 / 5.0 ms |
| BS GeoJSON / Shapefile | 23.0 / 11.8 ms |
| Prepared model analysis | 1702.3 ms |
| Prepared reference analysis | 765.2 ms |

[Full protocol](qa/api-verification.json) retains the original metadata until the display version AF is reduced to stable `af-v001`; weights and predictions have not changed.

## Dataset / area

Official Train: 644 scenes, 420 AF / 224 BS. Builder reads GeoTIFF instead of manually fixing counts. Compact demo: 3 BS + 3 AF; model 2916 zones / 81 point, reference 1757 zones.

20×20 m = 400 m2 = **0.04 ha**. `BS_tr_000001`: prediction 462.12 ha = low 461.52 + moderate 0.60 + high 0. GT 454.8 ha. Prepared bbox-analysis uses EPSG:6933 and may differ slightly from the area of nominal UTM pixels; both methods are explicitly indicated.

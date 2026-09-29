# Final product integration — 2026-09-19

## Main and work boundaries

Baseline main/origin/main: `bf088ec8f9cf6e7bf3029ad55463805f47853e30`. Initial status/branch/log/fetch completed; working tree was clean. Repeated fetch before completion confirmed the same remote audit. Reset/clean/rebase/force not applied. Models have not been retrained; `weighted_comboloss_sampler.pt` is omitted at the explicit subsequent instruction of the user.

## What's happened?

Existing KROMA design and pages, MapLibre map, FastAPI routes, scenario/live, prepared-analysis contract, geo area/clipping, TRAIN metadata index and previews, BS subprocess isolation, competition CLI and final weights. V001–v006 submission templates and research history were already present.

## Audit and corrections

| Subsystem | Reference state | Outcome |
| --- | --- | --- |
| frontend | PARTIAL: Predict upload is always unavailable, inspector prediction/errors placeholders | Full TRAIN inference/results/exports; Inspector modes, Research filled |
| backend | Prefabricated services are not connected to the UI; only liveness | Unified adapter, readiness/version/device/missing files, cache, runtime header, raster endpoints |
| ML adapter | STALE: Selected v005 with v006 weights | Precise transfer of existing red-edge inference; full byte-identical v006 |
| geo | BROKEN on real complex neighboring contours | GeometryCollection intersection, densification of common borders, STRtree and CRS cache |
| data/index | PARTIAL: builder only tar metadata | CLI for unpacked TRAIN, GeoTIFF CRS/transform, counts/area from masks |
| map | PARTIAL: bright squares, no validation geometry | Clusters AF/BS, subtle footprints, selection, GT/PRED/ERROR on the selected stage |
| prepared analysis | MOCK by default | CI fixture saved, real model/reference derived 3 BS+3 AF sets added |
| exports | PARTIAL: Shapefile backend already, UI not connected | GeoJSON/Shapefile/CSV/JSON and NPZ verified; GT export works without weights |
| Docker | BROKEN: did not install ml, did not contain bundle/index | Full dependencies, CPU Torch, libgomp, data/weight mounts, proxy frontend |
| research | KEEP: A Detailed History with an Outdated Integration Outcome | History saved; only handoff status updated and updated public scores |
| README/docs | STALE: ML declared empty, Research stub, old task backlog | New quickstart, model card/handoff/metrics/demo; plans in docs/archive/task |
| config | WORKING/PARTIAL | Extended env, manifest and previous stable registry; verified by Compose config |
| tests | WORKING, but the main product gaps are not covered | geometry/area/Shapefile/missing mount; smoke and full inference |

## Status of pages

| Page | Status | The real border |
| --- | --- | --- |
| Overview | It works | SCENARIO/live + TRAIN clusters; selected by TRAIN validation geometry |
| Events | It works | Scenario table tagged with SCENARIO; links to real TRAIN BS events added |
| Analytics | It works | Overview remains SCENARIO; real summaries through Analysis |
| Analysis | It works | Prepared scenes ∩ bbox ∩ UTC dates; downloader not implemented |
| Data | It works | 420 AF/224 BS, search/filters, previews with raw TRAIN |
| Predict | It works | Full TRAIN package, AF/BS v006, metrics, probability, export |
| About | It works | Project, two stages of AF+BS, existing 5BIT data |
| Research | It works | Score Progression, Pipeline, Experiments, Limitations |
| Swagger | It works | `/docs` and OpenAPI; routes are called HTTP/curl |

The browser has passed pages, selected TRAIN, BS prediction/errors, AF I4/I5/difference/GT and local TP/FP/FN, inspector before/after, incident detail, Analysis on a real scenario prepared model, Shapefile button, operator menu and Swagger UI via frontend proxy. Production confirms map centering on selected TRAIN scene and switching GT/PRED/ERROR; added color decoding severity and errors. Confirmed compact viewport 900×800 without horizontal overflow Predict. Backend checks invalid query, missing data/artifacts, checksum, no coverage, empty result through existing and new tests. A complete search of each state of each button on each viewport is not automated.

## Verifications and reproducibility

- Python: **134 passed, 3 skipped**. Skips refer to legacy AF research checkpoint and U-Net validation maps that are not in local delivery; current production AF/BS verified separately.
- Frontend: **22 passed**, TypeScript, ESLint, production build – PASS after clean Next-cache generation.
- Ruff and `git diff --check` - PASS.
- `pip check` — PASS; clean Docker dependency install — PASS.
- Geo: 20×20 m = 0.04 ha; 4 pixels = 0.16 ha, 900 pixels = 36 ha; CRS, holes, adjacent classes, geometry collections and Shapefile roundtrip.
- Full inference: **447 lines, 180 AF + 89 BS, 0 failed chips**, SHA256 `4ad1876c70836b6a003629f4d1dda836f17d8070a59d2a5c88c29b50c024865a` coincides with submission_v006. Checked keys/order, RLE roundtrip, duplicates, NaN and class overlap. This TEST does not have a BS with cloud_frac ≥ 0.5; all available cloud scenes are included in the full run.
- Service smoke `BS_te_000001`, `000020`, `000089` coincided with reference; final repeat of `BS_te_000001` after metadata changes also exact.
- Missing weights: `/health` 200/ok, readiness false, inference unavailable; GT export 200.
- All three prepared sets pass the dataset validator. Valid coverage built on masks; BS date-only normalizes to 00:00 UTC, arbitrary shooting time is not made up.
- Docker backend and frontend builds - PASS; container AF_tr_000001 = 44 pixels, BS_tr_000001 = 462.12 ha. Compose local/prod config — PASS.
- HTTP/curl: health/models/docs/openapi, datasets, inference, overlays, raw masks/probability, validation geometry, scenario/live/map routes, real prepared analysis and exports - PASS.

[Inference evidence](inference-verification.json), [API timing evidence](api-verification.json), [Local runtime versions](runtime-versions.txt), [metric](../key_metrics.md).

## Cleanup

KEEP: source, tests, license/critical notices, final weights, submission CSV/metadata, research report and experiment history. Synthetic CI fixture saved with origin synthetic_demo.

MOVE: former `docs/tasks` → `docs/archive/tasks` plans; archive links fixed.

DELETE: Empty `docs/tasks/task-1`; idle JPEG predict flow, placeholders prediction/errors and fictitious stage animation Analysis removed from UI. Old upload endpoint remained compliant diagnostic endpoint with clear unavailable.

IGNORE: venv, IDE/cache, raw datasets, previews/runtime/tmp, OOF `.npy/.npz`, local candidates, logs, unpacked local checkpoint `best_class1.pt/`. Doubles of .gitignore rules cut. Tracked did not have OS/IDE/log/temp/notebook garbage. Repository was not cleaned by destructive Git commands.

## Verification of the report

README, UI, manifest and research/report.md agreed: GOLD v006 / 0.8176; previous v005 / 0.8127. Old research statements are preserved as historical milestones, the final line "v006 is not integrated" corrected. Complete pipeline and inputs did not change mathematically; training was not performed.

## Known limitations

1. Spatial query - bbox + dates; polygon API/downloader not added. Prepared outputs are used only in their coverage.
2. Train in-sample; prepared real demo contains 6 scenes. This is not an independent quality assessment.
3. There is no proven link between each AF chip and the BS event; the full chain of incident lifecycle remains SCENARIO and does not pass for real surveillance.
4. First Web BS New Scene Query Launches Isolated Neural Subprocess; Competition Runs One Batch. Persistent worker not implemented. Cache versioned, nonpersistent, 12 entries; when replacing bundle restart.
5. LIVE requires FIRMS key; external map tiles require networking. RAW FIRMS does not enter the AF model.
6. No Authorization/Roles and Integrated PostGIS. Cross-platform bit-exact guarantee not declared.
7. Full playback of training history requires research OOF/raw inputs outside of Git. Inference only requires a shipped bundle and an official input package.
8. Deployment/push not executed; existing main workflow deploys new published commits.

## Teams and demo

Launch commands - at the beginning [README](../../README.md). Demo: Overview → Events → Data → BS_tr_000001 → PRE/POST → GT/pred/errors → Predict → area/severity → GeoJSON/Shapefile/NPZ → real Analysis → Swagger. Detailed scenario: [demo_script](../demo_script.md).

## Modified files

The full composition of the integration commit (A - added, M - changed, R - moved, D - removed)):

```text
M	.dockerignore
M	.env.example
M	.github/workflows/ci.yml
M	.gitignore
M	README.md
M	backend/Dockerfile
M	backend/README.md
M	backend/app/api/health.py
M	backend/app/api/v1/datasets.py
M	backend/app/main.py
A	backend/app/prepared_data/kroma-official-train-model/burn_zones.geojson
A	backend/app/prepared_data/kroma-official-train-model/hotspots.geojson
A	backend/app/prepared_data/kroma-official-train-model/manifest.json
A	backend/app/prepared_data/kroma-official-train-reference/burn_zones.geojson
A	backend/app/prepared_data/kroma-official-train-reference/hotspots.geojson
A	backend/app/prepared_data/kroma-official-train-reference/manifest.json
M	backend/app/services/analysis.py
M	backend/app/services/ml_service.py
M	backend/app/services/prediction.py
M	backend/app/services/train_dataset.py
M	backend/pyproject.toml
M	data/fire-aoi/monitoring_chips.geojson
M	data/fire-aoi/train_chips_index.json
M	docker-compose.prod.yml
M	docker-compose.yml
R098	docs/tasks/README.md	docs/archive/tasks/README.md
R100	docs/tasks/WEB-001-analysis-contract.md	docs/archive/tasks/WEB-001-analysis-contract.md
R100	docs/tasks/WEB-002-prepared-datasets.md	docs/archive/tasks/WEB-002-prepared-datasets.md
R100	docs/tasks/WEB-003-geometries-and-areas.md	docs/archive/tasks/WEB-003-geometries-and-areas.md
R100	docs/tasks/WEB-004-analysis-api.md	docs/archive/tasks/WEB-004-analysis-api.md
R100	docs/tasks/WEB-005-analysis-query-ui.md	docs/archive/tasks/WEB-005-analysis-query-ui.md
R100	docs/tasks/WEB-006-analysis-map.md	docs/archive/tasks/WEB-006-analysis-map.md
R100	docs/tasks/WEB-007-analysis-report.md	docs/archive/tasks/WEB-007-analysis-report.md
R100	docs/tasks/WEB-008-exports.md	docs/archive/tasks/WEB-008-exports.md
R100	docs/tasks/WEB-009-data-provenance.md	docs/archive/tasks/WEB-009-data-provenance.md
R100	docs/tasks/WEB-010-verification.md	docs/archive/tasks/WEB-010-verification.md
R100	docs/tasks/WEB-011-reproducible-delivery.md	docs/archive/tasks/WEB-011-reproducible-delivery.md
R100	docs/tasks/WEB-012-final-acceptance.md	docs/archive/tasks/WEB-012-final-acceptance.md
R100	docs/tasks/analysis-contract.md	docs/archive/tasks/analysis-contract.md
A	docs/demo_script.md
A	docs/key_metrics.md
A	docs/ml_handoff.md
A	docs/model_card.md
A	docs/qa/api-verification.json
A	docs/qa/final-integration.md
A	docs/qa/inference-verification.json
A	docs/qa/runtime-versions.txt
D	docs/tasks/task-1
M	frontend/Dockerfile
M	frontend/README.md
M	frontend/eslint.config.mjs
M	frontend/next.config.ts
M	frontend/public/data/monitoring_chips.geojson
M	frontend/src/app/about/page.tsx
M	frontend/src/app/research/page.tsx
M	frontend/src/features/analysis/AnalysisProgress.tsx
M	frontend/src/features/analysis/AnalysisReport.tsx
M	frontend/src/features/analytics/AnalyticsView.tsx
M	frontend/src/features/events/EventsView.tsx
M	frontend/src/features/explorer/ChipInspector.tsx
M	frontend/src/features/explorer/ExplorerView.tsx
M	frontend/src/features/map/KromaMap.tsx
M	frontend/src/features/map/MapDataSync.tsx
M	frontend/src/features/map/MapInteractions.tsx
M	frontend/src/features/map/MapLegend.tsx
M	frontend/src/features/map/layers/chips.ts
M	frontend/src/features/map/map.module.css
M	frontend/src/features/predict/PredictView.tsx
A	frontend/src/features/predict/PredictionResult.tsx
M	frontend/src/lib/api/endpoints.ts
M	frontend/src/lib/api/types.ts
M	frontend/src/state/workspace.ts
M	geo/README.md
M	geo/pyproject.toml
M	geo/src/kroma_geo/raster_vector.py
M	geo/src/kroma_geo/vector.py
M	ml/README.md
A	ml/artifacts/README.md
M	ml/artifacts/manifest.json
M	ml/pyproject.toml
M	ml/src/kroma_ml/artifacts.py
M	ml/src/kroma_ml/bs_neural.py
A	ml/src/kroma_ml/bs_rededge.py
M	ml/src/kroma_ml/competition.py
M	ml/src/kroma_ml/service.py
M	pyproject.toml
M	research/README.md
M	research/experiments/bs_v006_ctx_diag.py
M	research/experiments/bs_v006_rededge_diag.py
M	research/report.md
M	scripts/build_train_index.py
M	scripts/build_train_prepared_dataset.py
M	scripts/mount_artifacts.py
M	tests/backend/test_analysis_api.py
M	tests/backend/test_health.py
A	tests/backend/test_product_integration.py
M	tests/geo/test_raster_vector.py
M	tests/geo/test_vector.py
```

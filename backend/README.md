# Backend

FastAPI, REST `/api/v1`, Swagger `/docs`, liveness `/health`, model status `/models`.

From the root: `pip install -e geo -e ml -e backend`, then `OMP_NUM_THREADS=2 uvicorn app.main:app --app-dir backend --reload`. Complete quickstart and env - in [README](../README.md).

Sources shared: SCENARIO replay, RAW FIRMS LIVE, official TRAIN rasters/index, immutable prepared model/reference datasets. ML is called through a single adapter; Torch/LightGBM not imported backend directly. No → readiness false scales, process stays alive. No FIRMS key → LIVE offline.

GeoJSON/Shapefile/CSV/JSON and TRAIN NPZ are available via documented endpoints. Geometry WGS84, areas calculated to UI simplification. Contract prepared datasets: [docs/data/prepared-datasets.md](../docs/data/prepared-datasets.md). Checks: [acceptance](../docs/qa/final-integration.md).

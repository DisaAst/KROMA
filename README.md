# KROMA

KROMA combines **Active Fire (AF)** detection from VIIRS with **Burn Severity (BS)** estimation from before-and-after Sentinel-2 imagery. It provides an interactive map, an official TRAIN inspector, GOLD v006 inference, spatial and temporal analysis, area estimates, and GeoJSON, Shapefile ZIP, CSV and JSON exports.

Developed by the **5 bit** team. This is the personal public copy maintained by [DisaAst](https://github.com/DisaAst). Documentation is in English; literal dataset paths and recorded research artifacts retain their original spelling where required for reproducibility. The application interface is unchanged.

## Quick start

Use Python 3.12 and Node.js 22. On macOS, LightGBM requires `libomp` (`brew install libomp`). Production weights are included in `ml/artifacts` and verified with SHA256. Run commands from the repository root, with the backend and frontend in separate terminals.

```bash
git clone https://github.com/DisaAst/KROMA.git && cd KROMA
python3.12 -m venv .venv && source .venv/bin/activate
python -m pip install -e geo -e ml -e backend pytest ruff
python scripts/mount_artifacts.py
npm ci --prefix frontend
```

Start the backend:

```bash
OMP_NUM_THREADS=2 .venv/bin/uvicorn app.main:app --host 127.0.0.1 --port 8000
```

Start the frontend in a second terminal:

```bash
npm run dev --prefix frontend
```

Open http://localhost:3000 or the Swagger API documentation at http://localhost:8000/docs. The frontend proxies `/api`, `/docs`, `/health` and `/models` to the backend. Set `KROMA_BACKEND_URL` at frontend startup or build time to use a different backend port.

The web application and built-in prepared analysis work without the official TEST dataset. Competition inference requires a local copy of TEST:

```bash
python inference.py --data-dir data/test --output tmp/submission.csv --workers 2 --threads 2
```

For TRAIN inference and original PRE/POST imagery, place the official training data in `data/Мониторинг DATA/train` or set `KROMA_TRAIN_ROOT`, then rebuild the index:

```bash
python scripts/build_train_index.py --train-root "$KROMA_TRAIN_ROOT"
```

Without the variable, run `python scripts/build_train_index.py`. The index and compact derived datasets are already tracked in Git; raw rasters are excluded. Local Python does not automatically load `.env`. You can pass `--env-file .env` to uvicorn after replacing container paths with local paths.

## Available features

- **Overview:** TRAIN AF/BS clusters, scene footprints, and ground truth, predictions and errors at closer zoom levels.
- **Events:** an explicitly marked SCENARIO incident inspector. Real TRAIN scenes are available through Data.
- **Analytics:** SCENARIO time series for demonstration; real prepared results are available in area analysis.
- **Analysis:** a bounding box and inclusive UTC date range select intersecting prepared scenes, hotspots, contours, severity classes and a report.
- **Data:** 420 AF and 224 BS official TRAIN scenes, filters, metadata, I4/I5/difference layers, PRE/POST comparison, dNBR, land cover, ground truth and model output.
- **Predict:** upload a complete chip as ZIP, or select a TRAIN scene for predictions, errors, local metrics and exports.
- **Project, Research and Team:** project information, the processing pipeline, leaderboard results and limitations.

`POST /api/v1/inference/upload` accepts one ZIP of up to 64 MB containing the TIFF files for a single 256 × 256 chip. All filenames must share the same `ID`; subdirectories are allowed.

| Task | Required files |
|---|---|
| AF | `ID_VIIRS_I1-I5.tif` with 8 channels and `ID_AUX.tif` with 5 channels |
| BS | `ID_Sentinel-2_pre.tif` and `ID_Sentinel-2_post.tif` with 10 channels each; `ID_Sentinel-1_pre.tif` and `ID_Sentinel-1_post.tif` with 2 channels each; `ID_AUX.tif` with 3 channels |

An ordinary JPEG is not a valid model input. Uploaded chips do not require labels or coordinates, but quality metrics and vector exports are unavailable for these uploads. TRAIN inference requires source rasters mounted on the server.

## Architecture

```text
Next.js / React / MapLibre
          ↓ REST
FastAPI → dataset/index → previews / TRAIN geometry
        → ML adapter predict_af / predict_bs → versioned cache
        → prepared scenes + bbox/dates → summary / vector exports
          ↓
kroma_ml: AF LightGBM | isolated BS Torch → physics / refiners v006
kroma_geo: CRS, raster geometry, pixel area, clipping, topology
```

The backend calls models through `kroma_ml.service`; it does not import Torch or LightGBM directly. The Torch BS stage runs in a subprocess for macOS MPS and LightGBM compatibility. Competition inference uses one neural subprocess for the whole BS dataset. A new scene requested through the web application starts a separate subprocess; repeated results and geometries use a 12-entry LRU cache keyed by `scene_id + model_version`. Restart the service after replacing the weight bundle.

| Directory | Purpose |
|---|---|
| `frontend/` | Web interface and map |
| `backend/app/` | REST API, services, sources and prepared datasets |
| `ml/src/kroma_ml/` | Production adapter, competition inference, models and features |
| `ml/artifacts/manifest.json` | CURRENT GOLD inputs, outputs, thresholds and file hashes |
| `geo/` | Area calculations, CRS, vectorization and geometry validation |
| `data/fire-aoi/` | TRAIN index and permitted area of interest |
| `research/` | Experiment history and final report |
| `scripts/` | Weight verification and packaging, index and dataset builders |
| `tests/` | Backend, geospatial, ML and integration checks |
| `artifacts/submissions/` | Historical submissions v001–v006 and metadata |

## Data and provenance

The organizers supply official TRAIN and TEST data through [the dataset download](https://disk.yandex.ru/d/-rpmevTflbXZQg). Their terms govern use; this project does not assign a separate license to the source data.

```text
data/Мониторинг DATA/train/
  af/meta.csv, viirs/, aux/, masks/
  bs/meta.csv, sentinel2_pre/, sentinel2_post/, sentinel1_pre/, sentinel1_post/, aux/, masks/
data/test/
  meta.csv, sample_submission.csv, af/, bs/
```

The TRAIN index builder reads metadata, GeoTIFF masks, CRS, affine transforms, dimensions, bounding boxes, event/date/satellite fields, cloud and valid-pixel information, and severity counts. It does not geolocate TEST identifiers or reconstruct their locations or dates. Legacy previews can use `data/yandex/fire-train-renamed.tar`; production inference requires unpacked TRAIN data.

The repository includes these compact datasets under `backend/app/prepared_data/`:

| Dataset | Source | Contents |
|---|---|---|
| `kroma-ci-demo@1.0.0` | `synthetic_demo` / SCENARIO | Small deterministic CI fixture |
| `kroma-official-train-model@v006` | `model_output`, in-sample | 3 BS and 3 AF scenes, 2,916 severity zones and 81 AF points |
| `kroma-official-train-reference@v006` | `reference` | 1,757 official TRAIN ground-truth zones for the same BS scenes |

Prepared analysis selects existing outputs. It does not download satellite imagery for arbitrary areas. Requests outside coverage return `no_coverage`. The catalog validates SHA256 hashes, classes, geometry, dates and non-overlapping zones.

Rebuild the compact demonstration without training:

```bash
OMP_NUM_THREADS=2 python scripts/build_train_prepared_dataset.py --bs 3 --af 3
python -m app.cli.validate_dataset backend/app/prepared_data
```

A different scene selection requires a separate immutable dataset version. The weights and TRAIN data must match the documented handoff.

## Models and metrics

**CURRENT GOLD = v006; previous stable = v005.** AF is unchanged across submissions v001–v006. The AF model uses 39 LightGBM features and a threshold of 0.765. The BS pipeline combines the v003 neural ensemble, the v004 physics/refiner stage, and the v006 component classifier and red-edge refiner2. BS thresholds are 0.3 / 0.8 / 0.2.

| Submission | Public score |
| --- | ---: |
| v001 | 0.5526 |
| v002 | 0.6457 |
| v003 | 0.7298 |
| v004 | 0.7406 |
| v005 | 0.8127 |
| **v006** | **0.8176** |

Public leaderboard scores were confirmed by the team and differ from local TRAIN F1 and BS OOF metrics. BS v006 OOF scores are **0.43552** for all valid pixels and **0.44677** under the strict policy. TRAIN predictions are **in-sample**, because final weights were trained on all TRAIN data. See the [model card](docs/model_card.md), [key metrics](docs/key_metrics.md) and [research report](research/report.md).

## Weights and configuration

`KROMA_ML_ARTIFACTS_PATH` sets the production artifact root, which defaults to `ml/artifacts`. There is no implicit fallback to research files. `python scripts/mount_artifacts.py` validates all 7 current artifacts before copying; `--source <bundle> --dest <directory>` copies a validated bundle and its manifest. It does not download or train models.

Paths and checksums are documented in [ml/artifacts/README.md](ml/artifacts/README.md). Git includes both neural checkpoints, the AF bundle, physics JSON, the v004 refiner and the v006 refiners. The v005 files remain for historical reference. Health endpoints remain available when weights are missing; inference reports that it is unavailable.

## Competition inference and reproducibility

```bash
python inference.py --data-dir data/test --output tmp/submission.csv \
  --workers 2 --threads 2 --profile tmp/runtime.json \
  --verify-against artifacts/submissions/submission_v006.csv
```

The default is v006. The pipeline does not start REST services, vectorization, analytics or a database. It validates sample_submission keys and order, the 447 rows for this TEST dataset, RLE encoding, the absence of NaNs and duplicate keys, and non-overlapping BS classes. A chip error stops submission generation instead of silently substituting an empty mask.

On the integration machine, the full 180 AF + 89 BS run produced **CSV-identical v006** output in 452.88 seconds with 2 workers × 2 threads; the BS neural stage took 19.81 seconds. Other checks were running concurrently, and runtime depends on hardware. Bitwise equality across Torch versions and devices is not guaranteed. Training configurations use seed 42; production inference does not train models. Run `--verify-against` on your platform.

## Training and research history

Training is not required to run the delivered models. The integration pass did not retrain them. Historical commands and splits are recorded in [ml/README.md](ml/README.md), [research/report.md](research/report.md), `research/experiments/af_final_model.py`, `bs_track.py`, `bs_v004_build.py`, `bs_v005_build.py` and `bs_v006_infer.py`. The last research script **trains full refiners**; use the root `inference.py` for inference with existing weights. OOF caches and intermediate arrays are not required at runtime and are excluded from Git.

## REST API and Export

| Endpoint | The result |
| --- | --- |
| `GET /health`, `GET /models` | Liveness, AF/BS readiness, version, missing artifacts, device policy |
| `GET /api/v1/models`, `/api/v1/ml/status` | Manifest provenance and model status |
| `GET /api/v1/datasets/train?kind=bs` | This TRAIN index and counts |
| `GET /api/v1/datasets/train/{id}` | Metadata and availability of inspector |
| `POST /api/v1/inference/af`, `/inference/bs` | JSON `{"chip_id":"..."}` → mask metadata, metrics, runtime, geometry |
| `GET /api/v1/datasets/train/{id}/overlay/{layer}` | PNG: pred / gt / error / probability |
| `GET /api/v1/datasets/train/{id}/rasters` | NPZ: original mask/probability/GT/valid, version |
| `GET /api/v1/datasets/train/{id}/geometry?layer=pred` | WGS84 validation features for the map |
| `GET /api/v1/datasets/train/{id}/export?format=geojson` | GeoJSON; `format=shp` → ZIP Shapefile |
| `GET /api/v1/analysis/datasets`, `/analysis/readiness` | Catalogue and Integrity Check of Prepared Sets |
| `GET /api/v1/analysis` | `dataset_id`, `dataset_version`, `bbox`, `from`, `to` |
| `GET /api/v1/analysis/export/contours` | GeoJSON current result_id |
| `GET /api/v1/analysis/export/shapefile` | ZIP with SHP/SHX/DBF/PRJ/CPG |
| `GET /api/v1/analysis/export/report?format=csv` | CSV; `format=json` → complete JSON |
| `GET /api/v1/live/status`, `/live/hotspots`, `/live/incidents` | RAW FIRMS and heuristic clusters |

All exports of analysis require `expected_result_id` equal to the displayed result; mismatch → 409. HTTP responses have `X-Runtime-Ms`, inference returns model/vectorize/total. Shapefile limits DBF field names to ten characters: `model_version` → `model_ver`, `scene_id` saved, CRS WGS84. GeoJSON contains `contour_id`, scene/event, severity, area_ha, model_version, source.

Area of BS-pixel **20 x 20 m = 0.04 ha**. TRAIN inference counts the area in the original raster projection. AOI analysis trims unsmoothed vectors and calculates EPSG:6933; slight difference from the nominal UTM pixel area is expected and the method is specified in `calculation`. UI does not define the reporting area.

Example:

```bash
curl -fsS http://localhost:8000/health
curl -fsS -H 'Content-Type: application/json' -d '{"chip_id":"BS_tr_000001"}' http://localhost:8000/api/v1/inference/bs
curl -fSLo tmp/contours.zip 'http://localhost:8000/api/v1/datasets/train/BS_tr_000001/export?format=shp'
```

## Docker and deployment

```bash
cp .env.example .env
docker compose up --build
```

Compose mounts TRAIN data and weights read-only. Set `KROMA_TRAIN_HOST_PATH` and `KROMA_ML_ARTIFACTS_HOST_PATH` in `.env` to change their host locations. The backend image installs geo, ml, backend and CPU Torch; GPU deployment requires a compatible Torch runtime. Start the optional database with `docker compose --profile persistence up db`.

For production, run `docker compose -f docker-compose.prod.yml up -d --build`; Caddy serves the API and frontend. The original deployment workflow uses release, health-check and rollback steps. Its deployment job is restricted to `BitsOfDanya/KROMA`, so pushes to this personal copy do not deploy to the original server. On first deployment, `deploy/prepare-train.sh` downloads official TRAIN data (about 2.14 GB) into `/opt/kroma/shared/train`, which persists between releases and is mounted in the backend. Allow at least 3 GiB of free space in `/opt/kroma/shared`.

## Checks

```bash
.venv/bin/ruff check .
.venv/bin/python -m pytest -q
npm run typecheck --prefix frontend
npm run lint --prefix frontend
npm test --prefix frontend
npm run build --prefix frontend
```

The recorded integration checks covered API behavior, areas, CRS, geometry intersections, Shapefile round trips, missing artifacts and hashes, TRAIN previews and overlays, the complete submission, and the frontend. See [acceptance results](docs/qa/final-integration.md) and the [demo script](docs/demo_script.md).

## Limitations

- Analysis supports a bounding box and dates; arbitrary polygons and automatic Sentinel downloads are not implemented.
- Real prepared outputs cover 6 selected TRAIN scenes. This demonstrates the processing path and is not an independent quality evaluation.
- Events and overview analytics are explicitly labeled SCENARIO. No verified link between an individual AF chip and a particular BS fire event is claimed.
- LIVE requires `NASA_FIRMS_API_KEY` and reports offline status without it. Raw FIRMS data is not equivalent to AF model input; LIVE does not compute BS or forecasts.
- The map uses external tiles that require a network. TEST is never displayed on the map.
- Caches and process settings are not persistent. PostGIS is not connected to current services; authentication and roles are not implemented.
- Web BS inference starts a separate neural subprocess for each uncached TRAIN scene; competition inference uses batching. A persistent worker has not been introduced without a separate guarantee of identical results.

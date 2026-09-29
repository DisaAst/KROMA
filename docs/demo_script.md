# Stable demo KROMA

## Preparation

1. Run README quickstart; `/health`: status ok, AF/BS ready; `/api/v1/analysis/readiness`: ready.
2. Connect TRAIN, check for `/api/v1/datasets/train`: AF 420, BS 224. If there are no sources, show only the honestly signed prepared analysis, without the promise of PRE/POST/inference.
3. Open `/predict`, execute `BS_tr_000001` and `AF_tr_000001` in advance to warm up the process and cache. The Internet is needed for the substrate of the map.

## Showdown

1. **Overview** — Existing map, AOI, TRAIN clusters; explain the difference between SCENARIO and RAW FIRMS.
2. **Events** — Open the script event and evidence/timeline. Explicitly named SCENARIO; don't associate it with competitive TRAIN.
3. **Data** — OFFICIAL TRAIN, 420 AF/224 BS, select `BS_tr_000001`.
4. **Inspector** — PRE ↔ POST slider, GT severity, AUX/physics with dNBR and landcover. Click "Model", show in-sample disclaimer, then Errors.
5. **Scene on map** — Selected stage, GT / PRED / ERROR. Error colors: FP red, FN blue, severity mismatch yellow, AF TP green.
6. **Predict** — Select BS chip, run it. For `BS_tr_000001`: 462.12 ha, low 461.52, moderate 0.60, high 0. The mask is not a forecast of the future fire.
7. **Export** — GeoJSON, Shapefile ZIP (inside .prj/.cpg), NPZ masks/probability.
8. **Analysis** — Select `kroma-official-train-model@v006`, use the example of bbox/dates. Show dataset provenance, severity distribution and JSON/CSV/Shapefile.
9. **Swagger** — `/docs`: Execute `/models`, `/datasets/train`, `/analysis`; show consistent result_id for exports.
10. **Research** — v001...v006 leaderboard and limitations. Public score not compared to local in-sample F1.

## If something's not available

No → health scales live, model readiness false; don't promise prediction. No TRAIN → prepared model/reference available. No FIRMS key → LIVE offline; use SCENARIO and TRAIN marked. No network of → tiles statistics/export/API working, card substrate may not be available.

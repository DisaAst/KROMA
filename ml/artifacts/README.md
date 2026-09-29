# Production weights

CURRENT GOLD: **v006**, previous stable: v005. `manifest.json` - authoritative registry: SHA256, dimensions, inputs, outputs, thresholds, provenance and runtime. All 7 current files are included in this Git Delivery (~83 MB); an additional copy of v005 is saved for history.

| Key key | The Way from KROMA_ML_ARTIFACTS_PATH |
| --- | --- |
| af_model | af/af_fulltrain_model.joblib |
| bs_neural_ndvi | bs/neural/bs_full_ndvi_ohem40.pt |
| bs_neural_dual | bs/neural/bs_full_dual.pt |
| bs_thresholds | bs/physics_thresholds.json |
| bs_v004_refiner | bs/v004_refiner.txt |
| bs_component | bs/v006/component_full_v006_rededge.txt |
| bs_refiner2 | bs/v006/refiner2_full_v006_rededge.txt |

Verification: `python scripts/mount_artifacts.py`. Copying the complete set: `python scripts/mount_artifacts.py --source ml/artifacts --dest /path/to/bundle`. Then `KROMA_ML_ARTIFACTS_PATH=/path/to/bundle`. This root is the only source: there is no hidden fallback on research.

If the supply archive has excluded binary files, get from the team **full directory of ml/artifacts of the same commit**, Check each SHA256 with the command above. Do not rename arbitrary `.pt` to the expected name. Do not start training for the sake of preparing production weights. Check for missing files: `/models`; restart backend when you replace weights.

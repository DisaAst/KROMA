# Model card: KROMA GOLD v006

## Purpose

AF - binary detection of active combustion on VIIRS; BS - background/low/moderate/high on Sentinel-2 pre/post. The scope of application is limited to the official format and distribution of competitive data.

## Contract

`kroma_ml.service.predict_af(chip_id, root=None)` and `predict_bs(chip_id, root=None, logits=None)` return NumPy arrays and metadata. Explicit root means inference without GT; default TRAIN root adds GT for local validation. Backend uses only this adapter. Competition transmits batched logits, skipping service geometries.

AF: VIIRS I1–I5, angles/valid and AUX; 39 selected LightGBM features, threshold 0.765. Mask uint8, probability float32, valid, fire_pixels, runtime, model_version; TRAIN adds gt_mask/error_map.

BS: Sentinel-2 pre/post with red-edge, Sentinel-1, AUX. NDVI Attention U-Net + DualHead U-Net → v004 physics/refiner → v006 component/refiner2. The thresholds 0.3/0.8/0.2 are unchanged. Output mask uint8 0/1/2/3, burn_mask, burn_probability, severity_probability, valid, area, timings. Probability is a pipeline estimate, not a proven calibrated probability.

## Metrics and limitations

Public score v006 0.8176; BS OOF all-valid 0.43552, strict 0.44677. These are different metrics. Final models are trained on all TRAIN; local F1/IoUs are labeled in-sample. TEST is not georeferenced. Clouds, weak severity, mixed pixels and transfer to unknown regions limit reliability.

AF invalid pixels are excluded from mask; BS valid policy saved byte-exact from GOLD. GT nodata=255 errors are excluded from the local matrix and error overlay. Clouds are not interpreted as a guaranteed area.

## Delivery

Manifest: `ml/artifacts/manifest.json`. Seed Research Training Configurations 42. Integration did not retrain models. Torch neural network isolated from LightGBM; device automatically CUDA → MPS → CPU. Cross-platform bit identity is not promised; use `--verify-against` for your environment.

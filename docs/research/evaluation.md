# Pilot sampling for quality assessment

`research/experiments/build_evaluation_cohort.py` creates a reproducible set of candidates [NASA published the example of VIIRS S-NPP](https://firms.modaps.eosdis.nasa.gov/content/academy/data_ingest/firms_data_ingest.html). File dated July 12, 2023 is checked against SHA-256 before processing. Command from the root of the repository after installation of `backend`:

```bash
python research/experiments/build_evaluation_cohort.py --download
```

Results are excluded from Git `data/raw/`, `data/interim/` and `data/samples/`. [Fixed manifest](../../research/experiments/viirs-pilot-manifest.json) contains the reference, SHA-256 input, number of normalized records, selection rule and SHA-256 final JSONL. Received from source file 74 599 unique observations and 200 Candidates from the five available strata. Two successive launches gave the same SHA-256 final JSONL.

The pilot selects up to 40 observations in each `source_confidence` x `daynight` pair from a deterministic ID hash. The `train`/`validation`/`test` partition is assigned to whole geographic cells of 5° × 5°. It's a set **independent markup candidates**, It's not ready-made fire tags. `reference_label` remains `null`.

To mark up, you need to save a link to the confirming certificate, the date of inspection, the type of object (`fire`, `non_fire`, `uncertain`) and the inspector. Satellite confidence category cannot serve as a reference label. In ambiguous cases, a second inspector is needed. After markup, you can count accuracy, recall and errors separately by region, time of day and confidence categories. A full benchmark will require multiple seasons, independent regions and negative examples outside of FIRMS; this pilot does not measure the completeness of all fire detections.

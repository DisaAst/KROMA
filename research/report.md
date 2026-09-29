# KROMA: active-fire research baseline

Status as of 18.09.2026. Target geography - the whole of Russia; Krasnoyarsk AOI only one pilot. This report separates working code from testable hypotheses. Sources, licenses and download statuses are summarized in [catalogue](../docs/data/sources.md).

## PRODUCT: from anomaly to solution

`thermal anomaly → satellite observation → hotspot → suspected incident → confirmed/monitored incident → priority update → closed incident → burn scar → severity → damage summary`. The satellite observes a thermal anomaly in a pixel, not a fact, boundary or area of fire. NASA's `source_confidence` describes the pixel detection algorithm; `P(real wildfire)` requires other tags and calibrations. FRP - Radiation Intensity, Not Area.

| Block | Now in service | Research/data layer | Another ML stream |
| --- | --- | --- | --- |
| Detection | Backend LIVE gets FIRMS and normalizes VIIRS NRT | Reproducible historical samples, sources, AOI, filtering | — |
| Monitoring | LIVE is already grouping observations with its algorithm; UI reveals incidents | Independent DBSCAN baseline, time series, Thermal Memory | — |
| Decision support | UI/backends have heuristic indicators | Context, marked confidence, individual threat and priority, checking false alarms | — |
| Post-fire | The service has demo layers | Integration contract only and incident communication check | Romin Sentinel-2 segmentation, burn severity, area by class |

The life cycle requires constant `incident_id`, manual confirmation, closure/reopening and change logging. The current research ID is the hash of a complete set of observations and **Changes with new detection**; It cannot be implemented in LIVE without a separate identity logic.

## Market and Hypothesis Testing

| Analogue | Confirmed capabilities | What it means for KROMA |
| --- | --- | --- |
| [NASA FIRMS](https://firms.modaps.eosdis.nasa.gov/) | Free global MODIS/VIIRS thermal detections, maps/API/notifications; SP and NRT differ in latency and quality. | Map of hot spots and their delivery — **commodity**. Detections include industrial and natural heat sources. |
| [ISDM-Rosleskhoz / Avialesoprotection](https://aviales.ru/default.aspx?textpage=176) | The current monitoring of the entire Russian Federation combines satellites, weather, lightning, cartography, regional reporting and dispatching. | Nationwide Weather Monitoring and Integration. Open incident-level unloading and tag access conditions not confirmed. |
| [SCANEX "Fire Map"](https://new.scanex.ru/company/news/analiz-pozharnoy-obstanovki-na-territorii-rossii-po-sputnikovym-dannym-za-period-s-18-po-27-iyulya-2/) | Covers the Russian Federation, shows hotspots, clusters as large fires, statistics on subjects and pictures. | Simple grouping and regional statistics, too **commodity**; Verified triage and transparency required. |
| [OroraTech Wildfire Solution](https://ororatech.com/all-products/wildfire-solution) | Commercial end-to-end product states clusters, fire confidence, weather/vegetation/terrain, spread simulation, burnt area and severity; [reference](https://help.ororatech.com/must-read) False positives and Incident Overview. | You cannot position confidence, clusters, forecasts or severity as unique in themselves. Algorithms/Quality Comparisons Are Unavailable Without Data. |
| [NOAA Hazard Mapping System](https://www.ospo.noaa.gov/products/land/hms.html) | For North America, analysts check automatic fire detections and false alarms, apply smoke. | Human-in-the-loop and control of false positives is a practical standard of the process, but the geography of the product is not Russian. |

**Working hypothesis of difference** — A transparent Russian incident queue that explains the combination of recurring heat source, seasonality, land type, dynamics and facilities under threat. It's **hypothesis**, There is no manual evaluation of false-positive reduction, speed and utility for operators. Repetitive heat **not always false** (In particular, smoldering and repeated forest fires); Thermal Memory should give a sign and an explanation, not automatically hide the alert. [NASA](https://firms.modaps.eosdis.nasa.gov/content/descriptions/FIRMS_MODIS_Firehotspots.html) Directly indicates torches and volcanoes among anomalies; [NOAA](https://www.ospo.noaa.gov/products/land/hms.html) lists industry, solar glare, smoke plumes and artifacts. For the Russian Federation, we separately check the torches of the oil and gas industry, metallurgy, agricultural fells, settlements and repeated hearths.

## SCALE and data architecture

1. **Russia, easy level:** FIRMS detections, boundary layer, source metadata, incident aggregates; `AOI / region / date_from / date_to / source / mode` queries, spatial tiles and API intervals of 1-5 days. It's not a promise to download the whole country in one request.
2. **Region:** Land cover tiles, extracts OSM, ERA5-Land, DEM and vegetation according to the selected AOI. Simplified [geoBoundaries](https://www.geoboundaries.org/api/current/gbOpen/RUS/ADM1/) obtained and verified in five cities; its 2017/83 units are not suitable for accurate current statistics without an audit. For large vector layers, there is an optional Shapely: the full launch of four pilots was reduced from about 28 to 3 seconds. Federal District Requires Separate Verified Subject Conformity Table.
3. **Incident:** small buffer, Sentinel-2, local infrastructure, detailed relief and burn-severity result of Roma. Sentinel-2/WorldCover global storage is not required.

`geo/src/kroma_geo/context.py` accepts bbox, Polygon/MultiPolygon GeoJSON and the index of named regions. `scripts/ingestion/historical_firms.py` uses **existing** `fetch_csv` and VIIRS backend normalizer, caches raw CSV by source/tile/date, streams JSONL and SHA manifest, deduplits only the current time unit and adds a small adapter for SP/MODIS. Therefore, the bootloader’s memory does not grow with the entire historical period; the size of the temporary `seen` depends on 1-5 days of one source. For MODIS, the `brightness_modis_kelvin` field is separate because the MODIS channel is not VIIRS I4. These research records are compatible with `kroma_ml.Observation.from_record`, but **not** Limited `Literal` backend API 1.0; integration contract needs to be negotiated. The secret of the API does not fall into manifest. Directly running a historical download requires `NASA_FIRMS_API_KEY`; the key is not currently available, so no SP/MODIS downloads have been made.

`ml/src/kroma_ml/incidents.py` implements DBSCAN by Haversine and time neighbors with a spatial index; `eps_km`, `time_window_hours`, `min_samples` parameters are common to any AOI. Output: temporal boundaries, spherical center, number of detections/satellites, FRP statistics/trend, radius of coverage and region with a single destination. radius — **spotting**, not perimeter or area of fire. A bridge between different spans can mistakenly combine nearby fires; density depends on the sensor and season.

`ml/src/kroma_ml/thermal_memory.py` stores only occupied cells of an equal-square spherical grid and counts history **strictly until the time of the incident**: Number of Detections, Days/Months, FRP, Night Ratio, Sensor Diversity, Latest Appearance and Same Month. Grid - simple H3 baseline independent; the physical shape of cells changes with latitude. Novelty – research regional-seasonal rank, uncalibrated probability. Without history, the profile is not published.

`geo/src/kroma_geo/context.py` can calculate distances to points / lines / polygons from the local GeoJSON, including getting into a protected area. `geo/src/kroma_geo/raster.py` can read the WorldCover class by point and composition by local polygon, as well as calculate NDVI / NBR; rasterio - optional dependence, operations are checked on a synthetic raster. Historical section of OSM on 12.07.2023 was obtained for a small window of Kuzbass (835 objects, SHA in Ukraine) [manifest](experiments/osm-kuzbass-2023.manifest.json)): of the two pilot incidents, one inside the coverage and has a road distance of 0.034 km, the other is clearly marked `outside_coverage`, its distances are left by `null`. Internal status – `partial_coverage`: query boundaries, omitted relations and OSM incompleteness do not allow to interpret the absence of an object as evidence of absence on the ground. WorldCover COG, ERA5-Land and DEM are not yet connected, full impact context is missing. `ml/src/kroma_ml/features.py` generates a table of input features without a target variable and separately stores the native confidence and brightness VIIRS/MODIS; pilot JSONLs contain `confidence_inputs` with `null` instead of missing data. `priority_score` and `threat_score` are implemented as explainable functions from normalized inputs; without confirmed priority confidence and contextual data, pilots are not assigned real priority scores.

## Pilots and Real Measurements

Reproducible [configur](experiments/pilots.json) defines four intersections **AOI-window ∩ subject of the Russian Federation** by the resulting geoBoundaries layer; [manifest](experiments/pilots.manifest.json) Contains input and boundary hashes, parameters, number of observations/clusters and runtime. Borders still need revision for current statistics. Entrance - published by NASA S-NPP NRT **only 12.07.2023**; The full file is global, after filtering:

| AOI-window ∩ subject | Type to check | Detections | DBSCAN incidents | Noise | Time |
| --- | --- | ---: | ---: | ---: | ---: |
| Yakutia `RU-SA`, 125–160°E, 56–74°N | woodland | 3442 | 74 | 13 | ≈1.26 s |
| Kuzbas `RU-KEM`, 85–89°E, 52–57°N | industrial | 10 | 2 | 2 | 0.004 seconds |
| Orenburger `RU-ORE`, 50–63°E, 50–55°N | agricultural | 10 | 3 | 2 | 0.004 seconds |
| Krasnoyarsk region `RU-KYA`, 85–110°E, 52–73°N | forest | 9 | 2 | 4 | 0.003s |

Time in the table – local single-process start with `tracemalloc` **After reading, filtering and assigning regions**. The full call with Shapely took about 3 seconds. Python peak allocation for pilot clustering of Yakutia ~1.07 MB, **processor**. Config contains the history window 2020-2023, but there is no data in the local input until 12.07.2023: `historical_observations=0`, Thermal Memory = `null` in all four pilots. In the industrial, agricultural and Krasnoyarsk windows, the samples are too small to infer the type of anomalies.

Separate scale check of DBSCAN itself on all 74 599 Global Detections of the Same Day: **8321 research cluster**, 5591 single/noise detection, **23.09** and **15.13 MiB** Python peak allocation with `tracemalloc` on this machine. The input has already been loaded into memory; it is not a total RSS and not a test of several years in Russia. Cluster counts have no meaningful validation without ground truth.

Sensitivity on the Yakutian window inside `RU-SA` (3442 observations): `eps=2/5/10 км`, 24 h window, `min_samples=2` gives **115/74/66** Clusters; with `eps=5`, window 6 hours — **82**, `min_samples=3` — **69**. This shows a significant dependence on the threshold. Fragmentation and Merge Error **unmeasured**: There is no marking of specific fires/neighborhoods.

Repeat after installing editable `backend`, `geo`, `ml` packages and receiving NASA sample:

```bash
python research/experiments/build_evaluation_cohort.py --download
mkdir -p data/external/boundaries
curl -L https://media.githubusercontent.com/media/wmgeolab/geoBoundaries/9469f09/releaseData/gbOpen/RUS/ADM1/geoBoundaries-RUS-ADM1_simplified.geojson -o data/external/boundaries/geoBoundaries-RUS-ADM1_simplified.geojson
python scripts/prepare/pilot.py --input data/interim/sample_viirs_snpp_071223.jsonl --regions data/external/boundaries/geoBoundaries-RUS-ADM1_simplified.geojson --config research/experiments/pilots.json --output-dir data/processed/pilots --manifest research/experiments/pilots.manifest.json
```

For a true 1–3-year-old fixed pilot, use `NASA_FIRMS_API_KEY` and `scripts/ingestion/historical_firms.py --help`, collect event dates and history separately, then transfer `--history` to the pilot. Prefer SP for completed past seasons by checking the availability of each date. NRT passes through the same core pipeline, but does not replace SP in fixed evaluation.

## LABELS, CONFIDENCE, THREAT, PRIORITY, IMPACT

- **Markings:** Collect stratified manual subset by region/earth type/season/sensor, enable persistent industrial sites as *candidates* hard negative, match open reports/polygons and GWIS burned area as weak confirmation. Fix the source, date, expert confidence and `verified / weak / unresolved` status. You can not declare burned-area absence negative label: small fires may disappear.
- **Confidence:** future `P(real wildfire | observations + context)` at incident-level. Features: FRP/brightness/native confidence **sensory**, day/night, pixel size, neighbors, dynamics, novelty, land cover, proximity to torch/industry, weather, region and season. Start with logistic regression after proven tags appear; then compare booting. Spatial-temporal split by events and regions prevents leakage. Precision, recall, PR-AUC and false-positive reduction at recall ≥90/95%. **Now the model is not trained, these metrics are missing.**
- **Threat:** Potential developmental hazard: wind, fuel/dryness, slope and incident growth. The code takes normalized inputs and gives a layout; input sources are not connected, the formula is not validated.
- **Priority:** Urgency of operator attention: confidence 25%, threat 30%, exposure 25%, accessibility 10%, freshness 10%; returns 0-100 and contribution of each component. `confidence` entry must be a separately validated probability; no real score is currently issued. Evaluation later – expert consensus in rankings and sanity cases, not invented accuracy.
- **Impact context:** nearest settlement, road, power line, industrial facility, airport, protected areas, land cover, wind / precipitation, relief, NDVI. The presence of an OSM object does not mean full coverage. The distribution corridor for now **research-only**: wind, slope and fuel scenario ensemble, uncertainty envelope P50/P80/P95, check on known incidents prior to display to the user; this is not a physical forecast and is not implemented.
- **Post-fire contract:** Expect `incident_id`, geometry/mask reference, `total_burned_area_ha`, `severity_classes`, `area_by_class`, `model_version`, pre/post image dates and quality metadata. There is no competing burn-severity inference in this layer.

## CLAIMS / RISKS / NEXT

| Claim | Status | Ground |
| --- | --- | --- |
| FIRMS VIIRS NRT sample ingestion | READY for a fixed published example | 74 599 normalized observations, SHA manifest; not live SLA |
| AOI filter and research clustering | PARTIAL | 4 regions; grouping errors not identified |
| Historical FIRMS, MODIS/SP | PARTIAL | code, cache, adapter and unit tests; real requests are blocked by missing key |
| Appointment of the subject of the Russian Federation | PARTIAL | GeoJSON API and synthetic test; real actual layer is not fixed |
| Thermal Memory / false-positive reduction | RESEARCH / UNSUPPORTED to improve quality | Algorithm is there; 0 historical observations and no marks |
| Confidence / threat / priority | RESEARCH | contract and explainable formulas; no validated probability or context |
| Impact context / corridor | PARTIAL / RESEARCH | one historical OSM extract in Kuzbass and one incident within its coverage; nationwide context and forecast no |
| Post-fire severity | beyond | Integration contract; quality of Romino ML was not evaluated here |

Main risks: availability of MAP_KEY and historical SP; old / ODbL administrative layer; VIIRS / MODIS confidence difference; rare satellite coverage and gaps under the clouds; false suppression of real repeated fires; lack of incident labels; low OSM completeness in remote locations; regional shifts and instability of DBSCAN thresholds. The following five steps: (1) get MAP_KEY and collect 1-3 years of SP for four windows; (2) agree on the current layer of subjects/counties and its license; (3) manually mark the cases of merge/split and persistent sources; (4) connect AOI-tiles land cover, OSM and ERA5-Land and make matrix confidence; (5) validate operator ranking before integration with the service.

## COMPETITION TRACK: Active Fire (AF) pixel segmentation

A separate track, the Official Organizer Dataset (`data/Мониторинг DATA/train/af/`), does not overlap with the product section above and does not touch Burn Severity (`train/bs/`)). Task: pixel-wise binary segmentation of active combustion (VIIRS I1–I5 + auxiliary rasters → `1` = natural combustion, `0` = everything else), metric – `F1_af`.

### DATA

- `data/Мониторинг DATA/train/af/meta.csv` - 420 chips, the only `kind=af`. Organizers' test **laid down** (No `test/`, `baseline/`, `sample_submission.csv`, `metric.py` – only train). Baseline organizers had nothing to implement; step P6 (section 6) was omitted for an objective reason, documented as a known restriction.
- Rasters: `viirs/{chip}_VIIRS_I1-I5.tif` - 8 channels float32 `I1,I2,I3,I4,I5,solar_zenith,sensor_zenith,valid` (GDAL band descriptions in tag, not by file name - "I1-I5" in name misleading). `aux/{chip}_AUX.tif` - 5 channels float32 `landcover,dem,t2m,rh2m,wind_speed`. `masks/{chip}_MASK.tif` - uint8, `1`=fire, `nodata=255`. All 256×256, gsd=375m, EPSG 32637/32638 (UTM 37N/38N)).
- `meta.csv` is a general scheme with BS; for AF empty: `fire_event_id`, `region`, `date_pre/post`, `s1_date_*`, `cloud_frac`, `landcover_top`, `burn_area_ha`, `sev*_px`. Completed: `chip_id, kind, epsg, x_min..y_max, width, height, gsd, acq_datetime, satellite, valid_frac, n_fire_px`.

**OBSERVATION** `fire_event_id` is empty for all 420 AF lines (this field is only for BS in the general scheme)).
**DECISION** The grouping for split cannot follow `fire_event_id`; the `(x_min,y_min,x_max,y_max)` spatial tile is used as a surrogate group key (`kroma_ml.af_split.group_key`, with a follback on `fire_event_id` if it appears in a future version of the data)).

**OBSERVATION** 420 chips = 57 unique tiles re-removed at different times (up to 38 chips per tile)).
**DECISION** a random split on `chip_id` would give a direct leak of the location (the same background /landcover / DEM in train and val) - mandatory `GroupShuffleSplit` on tiles (`kroma_ml.af_split.train_val_split`, seed = 42, check non-crossing groups through exception at overlap). 366 train / 54 val chips, the share of positive-chips is retained (70.5% / 70.4%)).

**OBSERVATION** 296/420 (70.5%) chips have at least 1 fire pixel, 124 – all negative (`n_fire_px=0`)).
**DECISION** Negative chips are needed in the train as a source of "clean" background (without objects) - included in the general pool for sampling background pixels, not discarded.

**OBSERVATION** summarily 9725 fire-pixel 27 525 120 the pixels of the dataset `fire_frac = 0.000353` (0.035%). `n_fire_px` median for positive chips = 21, maximum 243.
**DECISION** Extreme imbalance confirmed quantitatively by → without sampling/weighing task degenerates into "predict always 0" with accuracy of 99.96% and F1=0; learning only on a balanced-sampled set (all fire + limited background sample on chip), `class_weight="balanced"` in LogisticRegression/LightGBM.

**OBSERVATION** No duplicates of `chip_id`, no NaN in `width/height/gsd/x_min`, `chip_id` parses 1:1 on files in all three subdirectories.
**DECISION** meta.csv does not require cleaning before use (not modified)).

### EDA (pixel sample: all fire pixels + up to 500 background/chip, 155 034 lines, `research/experiments/af_eda.py` → `research/experiments/af_pixel_sample.pkl`))

**OBSERVATION** `I4`: fire mean=341.4K (std 8.6) vs bg mean=301.5K (std 11.8), `I5`: fire 301.4K vs bg 289.5K, `I4-I5`: fire mean=40.1 vs bg mean=12.0. The separation is strong, but the ranges overlap markedly (fire min I4 = 319, bg max I4 = 349)).
**DECISION** Crude `I4`/`I5`/`I4-I5` is not enough for an exact threshold - you need a contextual (locally-relative) signal, not only an absolute one.

**OBSERVATION** `I4 - local_median(I4, window)` divides classes by an order of magnitude stronger than raw channels: for `window=7` fire mean=28.3 (std 13.5) vs bg mean=-0.21 (std 2.9); similarly for `(I4-I5) - local_median(I4-I5)`. The separation grows with the window3→11), but comes to the plateau to 7–11.
**DECISION** Contextual thermal features (`I4_minus_localmed_*`, `D_minus_localmed_*`) are the main features for both physics-baseline and ML; `window=7` is chosen as a working compromise (comparable division with 11, it is cheaper to consider).

**OBSERVATION** `I1`/`I2` (visible/NIR) in fire is lower and with a much smaller spread (std 0.03–0.05) than in bg (std 0.12–0.14, max to 1.18–1.25) – the background has a heavy tail of bright pixels (glint/cloud/snow). `I3` hardly distinguishes between classes (fire mean 0.179 vs bg 0.166)).
**DECISION** `I3` did not give an independent signal for fire/non-fire in this sample — left in the feature stack as a cheap context channel, but not as a separate physical gate in the baseline; it is not confirmed that it specifically catches solar-glint negatives (Section 3 hypothesis was not confirmed on this data)).

**OBSERVATION** `aux_rh2m` (humidity) fire mean = 32.5% vs bg mean = 38.5%; `aux_wind_speed` fire mean = 5.64 vs bg 4.83 m/s – both in the physically expected side (dry and windy near the fire)).
**DECISION** humidity/wind included in the `weather` feature group, tested in ablation (see below) - contribution to F1 in ML is almost zero at already strong thermal-signatures, left as cheap additional features, not as a separate physical gate.

**OBSERVATION** `aux_landcover` (numerical code) differs in distribution (fire median=30 vs bg median=40), but without a class directory, the interpretation of the code is impossible - the organizers did not put the legend of the landcover classes.
**DECISION** landcover is used as a raw numerical feature for ML (useful as a categorical split factor for LightGBM without interpretation), but error-analysis by landscape classes (section 18) is not carried out - no code map, fixed as a restriction.

**OBSERVATION** `I4` saturation (≥367K, typical VIIRS saturation) has never occurred - max fire I4 = 358.2K.
**DECISION** no saturation gate is needed, physical baseline excluded (simplification)).

**OBSERVATION** Moderate shift between satellites: `I4` fire mean - SNPP 341.5K, NOAA-20 340.8K, NOAA-21 347.0K (the latter in a sample of only 281 fire pixels from 17 chips - high variance estimate)).
**DECISION** sensor id as a separate feature is not added - distortion of the same order as noise from a small sample of NOAA-21; when more NOAA-21 chips appear, it is worth rechecking.

### VALIDATION

`kroma_ml.af_split.train_val_split` - `GroupShuffleSplit(test_size=0.2, seed=42)` by tile group key, with a strict check of the absence of group intersection (raise when overlapping)). The `tests/ml/test_af_pipeline.py::test_split_has_no_group_leakage` test checks this on every run. 366 train / 54 val chips.

### Baseline Organisers

Not supplied to `data/Мониторинг DATA/` (only train data and AOI polygon, without test/baseline/metric/sample_submission)). Section 6 is skipped, recorded as a data limitation, not as an uncompleted step.

### PHYSICS-AWARE BASELINE (`kroma_ml.af_physics`, `research/experiments/af_physics_baseline.py`)

Rule: `I4 > i4_min AND (I4-I5) > diff_min AND (I4 - local_median(I4, 7)) > anom_min`. Thresholds selected by grid search by train (2940 combinations), confirmed by val.

| Metrica | Train (tuning)) | Val (held-out) |
| --- | ---: | ---: |
| precision | 0.9336 | 0.9434 |
| recall | 0.8996 | 0.8949 |
| **F1_af** | **0.9163** | **0.9185** |

Best settings: `i4_min=330K, diff_min=24K, anom_min=0` (window 7×7)). Val ≈ train - no retraining of thresholds.

**OBSERVATION** `anom_min` converged to the grid boundary (0 for the first grid, -10 for the refined one) – the context gate does not improve F1 beyond the already specified `I4_min`/`diff_min`.
**DECISION** The physical hard threshold on `I4`+`I4-I5` already implicitly requires a local thermal anomaly (the background rarely gives `I4>330 & I4-I5>24` without real combustion) - the third gate is redundant for this baseline, left in the API with default 0 (does not interfere, but does not help)).

1.47 ms/chip (54 chips for 0.079c) – almost free.

### FEATURE PIPELINE (`kroma_ml.af_features`)

39 features per pixel: raw I1–I5, `I4-I5`, angles, 5 aux channels, + for I4 and I4–I5 – local median/std/anomaly on windows {3,5,7,11}, + I3 local median/anomaly on window 7. Implemented through `scipy.ndimage.median_filter/uniform_filter`single point of entry `chip_feature_stack`s `lru_cache`-wrapped `cached_feature_stack` Repeated experiments on the same chips. NaN/inf from invalid pixels and filter edge effects are sanitized by zero within `chip_feature_stack` (`test_feature_stack_has_no_nan_or_inf` test)).

### ML BASELINE (Tabular, `kroma_ml.af_model`, `research/experiments/af_ablation.py` experiment))

Sampling: all chip fire pixels + random 400 background/chip (+ hard negatives as you add)). LogisticRegression (`class_weight="balanced"`, StandardScaler) and LightGBM (`class_weight="balanced"`, 300 trees)). The threshold is always selected by sweep 0.05-0.95 val (`kroma_ml.af_threshold.sweep`)).

### ABLATION TABLE (val, F1_af, threshold selected for val))

| # | Experiment | Precision | Recall | F1 | train, with | inference, ms/chip |
| - | --- | ---: | ---: | ---: | ---: | ---: |
| 1 | organizer baseline | — | — | — | — | laid down |
| 2 | physics baseline (I4, I4-I5, anomaly) | 0.9434 | 0.8949 | **0.9185** | — | 1.5 |
| 3 | ML raw channels (I1-I5), LogReg | 0.7271 | 0.9970 | 0.8409 | 109.8 | 302 |
| 4 | + thermal contextual, LogReg | 0.6895 | 0.9985 | 0.8157 | 5.2 | 23 |
| 5 | + landcover, LogReg | 0.6878 | 0.9992 | 0.8148 | 4.4 | 23 |
| 6 | + weather (dem/t2m/rh2m/wind), LogReg | 0.6963 | 1.0000 | 0.8210 | 4.2 | 21 |
| 7 | + angles (solar/sensor zenith), LogReg (=full features) | 0.6901 | 1.0000 | 0.8167 | 4.4 | 23 |
| 8 | full features, LightGBM | 0.7883 | 0.9992 | 0.8813 | 5.1 | 88 |
| 9 | + hard-negative mining (out-of-fold, 3-fold GroupKFold) | 0.9423 | 0.9758 | **0.9588** | 6.2 | 96 |
| 10 | + threshold re-tuning (finer grid 0.3–0.99) | 0.9470 | 0.9728 | **0.9597** | — | — |
| 11 | + post-processing (min connected component ≥2) | 0.9534 | 0.8050 | 0.8730 | — | — |

**Best configuration: #10, LightGBM + full features + out-of-fold hard-negative mining, F1_af = 0.9597 on held-out val.** Increment over physics baseline: +0.041 F1.

### U-Net and ensemble: the same spatial validation

The saved experiments use the same `GroupShuffleSplit(seed=42)`: 366 train / 54 val chips as the LightGBM. Checked ID validation, exact match of masks and playback of the probability map of the best checkpoint on the CPU (maximum difference of the first chip is less than `1e-6`)). PyTorch 2.14.0 on Apple Silicon; MPS is available outside of a limited process and is used to supplement D training. All models are compact, `base=16`, ~0.48-0.51 million parameters. Older launches have only the best checkpoint metrics and maps, not the era-stopping; their eras cannot be retrospectively restored. For the corrected D, the 8 epoch journal is in the run, the best weight is selected by val loss.

| MODEL | FEATURES | LOSS | PRECISION | RECALL | F1 | THRESHOLD | TRAIN RUNTIME |
| --- | --- | --- | ---: | ---: | ---: | ---: | ---: |
| Physics baseline | I4, I4-I5, local anomaly | rule | 0.9434 | 0.8949 | 0.9185 | 330 K / 24 K | — |
| LightGBM, the original best | 39 + OOF hard negatives | balanced logloss | 0.9470 | 0.9728 | 0.9597 | 0.92 | preserved-model |
| LightGBM, finer threshold | same | same | 0.9490 | 0.9713 | 0.9600 | 0.935 | non-relearning |
| U-Net A | I1–I5 | BCE + Dice | 0.7692 | 0.7029 | 0.7346 | 0.50 | 253.4 s |
| U-Net A | I1–I5 | Focal | 0.0005 | 0.9426 | 0.0011 | 0.05 | 252.4 seconds |
| U-Net A | I1–I5 | Focal Tversky | 0 | 0 | 0 | 0.05 | 246.1 |
| U-Net B | I1–I5, I4-I5 | BCE + Dice | 0.8572 | 0.8670 | 0.8621 | 0.45 | 212.4 |
| U-Net C | B + 2 local anomalies | BCE + Dice | 0.8400 | 0.8768 | 0.8580 | 0.80 | 267.1 |
| U-Net D corrected | C + 8 aux/valid | BCE + Dice | 0.8277 | 0.8753 | 0.8508 | 0.992 | 179.7 |
| U-Net B + hard negatives | B | BCE + Dice | 0.8666 | 0.8987 | 0.8824 | 0.05 | 85.5 s further training |
| ResUNet B, saved best | B | BCE + Dice | 0.8980 | 0.9380 | 0.9176 | 0.635 | 160.7 seconds |
| `0.9·LightGBM + 0.1·ResUNet` Ensemble | both probability maps | — | 0.9506 | 0.9735 | **0.9619** | 0.89 | schooling |

For A-C and auxiliary old launches, the original coarse threshold sweep is given. Separate Focal/Focal Tversky launches B-D also lie in [results](experiments/af_unet_results.json); They lost to BCE + Dice. D is retrained because the previous normalization erroneously z-score'il the landcover code as a continuous physical quantity. Continuous channels are now getting statistics **train only**, `valid` remains 0/1, landcover is limited to 0-1 without mean/std. Mask 255 is excluded from loss and metrics; in the current 420 files, 255 has not met, protection is checked by a synthetic test. Corrected D above the previous 0.8298, but below B. Threshold 0.992 found additional sweep in upper range 0.95–0.999.

Old best U-Net on self-threshold sweep gets F1 0.9176. The ensemble on the same valid pixels gave +0.0019 F1 to the newly configured LightGBM and +0.0022 to the original F1 0.9597. Error intersections: FP of both 52, FP only LightGBM 17, FP only U-Net 89; FN both 12, FN only LightGBM 26, FN only U-Net 70. Weight and threshold are selected on the same 54 val chips, so this local gain is tested lower in other spatial groups.

### 3-fold group CV

[Experiment](experiments/af_group_cv.py) kept the original 366/54 split unchanged and separately built 3 `GroupKFold` on the same spatial tiles: each 280 train and 140 val chips. In each outer train, LightGBM re-received OOF hard negatives via internal 3-fold GroupKFold; ResUNet B Receives Train-only Normalization and Up to 8 Epoch. Inside each val is selected threshold; `alpha=0.9` ensemble **fixed** single validation. [Full manifest](experiments/af_group_cv_results.json) Contains all precision, recall, F1 and thresholds.

| Model | Fold 1 F1 | Fold 2 F1 | Fold 3 F1 | Mean F1 | Std F1 |
| --- | ---: | ---: | ---: | ---: | ---: |
| LightGBM + OOF hard negatives | 0.9687 | 0.9656 | 0.9579 | **0.9641** | 0.0056 |
| ResUNet B | 0.8985 | 0.9173 | 0.9051 | 0.9070 | 0.0095 |
| Ensemble, alpha 0.9 | 0.9685 | 0.9656 | 0.9576 | 0.9639 | 0.0056 |

The ensemble lost to LightGBM by 0.0002 average F1 and did not win a single fold; the selection of alpha separately on each fold chose 1.0, 0.9, 1.0 respectively, but this is an additional choice for the same val tags and is not a separate assessment of quality. **Solution: the current best AF predictor is LightGBM; ensemble and U-Net in general inference do not connect.** The 3-fold CV is a test of model stability, not an independent test: architecture and alpha were previously selected using the original val, and the thresholds of each fold are also configured to its val. LightGBM thresholds on folds 0.77/0.85/0.765 indicate that the calibration for the external test must be checked separately.

**OBSERVATION** raw/several added ML features (#3–7, LogReg) are worse than physical baseline (0.81–0.84 vs. 0.92) with random negative sampling – recall almost 1.0, but precision 0.69–0.73, that is, the model fills the background with false positives.
**DECISION** randomly sampled background is not representative of the real distribution of negatives (does not contain “difficult” similar-to-fire pixels) – learning only random negatives systemically underestimates precision. Just adding features (landcover/weather/angles) does not solve this problem.

**OBSERVATION** LogReg→LightGBM on the same features and the same random-negative sample already gives +0.06 F1 (0.817→0.881but the precision is still low0.79).
**DECISION** The nonlinear model makes better use of contextual features, but the problem of the representativeness of negatives is primary – it is not solved only by choosing a model.

**OBSERVATION** hard-negative mining (out-of-fold FP, top-20 By confidence on chip, 3-fold GroupKFold without leaking to val) gave the main increase in the experiment: F1 0.881→0.959 (+0.078Almost all due to precision ()0.79→0.94For a small loss, recall ()0.999→0.976).
**DECISION** This is the most effective of the tested levers - recorded as a mandatory step of the final piplin, not an optional revision.

**OBSERVATION** Post-processing by the size of connected components **spoiler** metric: already `min_size=2` drops F1 on 0.959 before 0.873 (recall 0.976→0.805), further only worse (`min_size=4` → F1 0.60).
**DECISION** The TK hypothesis is confirmed - many real fire detections in this dataset are single-connected (1 pixel)). Morphological filtration by component size **not applied** final-pipeline.

### SATELLITE GENERALIZATION

Val-sample by satellite: SNPP (37 chips) F1=0.9496, NOAA-20 (16 chips) F1=0.9753, NOAA-21 - only 1 chip in val (F1 is unreliable on such a sample, not interpreted)). Substantial systematic subsidence on the sensor was not detected on the available data; sensor-id as a feature was not added (insufficient grounds)).

### LANDCOVER GENERALIZATION

Not executed - the organizers did not provide decoding of numerical codes `aux_landcover`, the grouping in forest/cropland/... without the legend would have been a divination. Fixed as an open point.

### DATA LEAKAGE

Split on spatial tiles excludes getting the same location in the train and val. Hard-negative mining uses only out-of-fold predictions on the train (3-fold GroupKFold) – val does not participate in negatives selection or model weight training. But the choice of U-Net, alpha ensemble and threshold configurations uses the same val as F1; it is a source of optimistic bias, even without intersection of spatial groups. FIRMS/external lookup, coordinates and dates were not used as model features (only as debug/EDA grouping)).

### Speed

Physics baseline: ~1.5 ms/chip in early measurement. New Measurement on 10 val chips of the current host without cache: Preparation of 39 LightGBM features 276.1 ms/chip, `predict_proba` 196.5 ms/chip (sum 472.6 ms/chip)). For ResUNet B: Preparation of 6 channels 2.1 ms/chip on the CPU, model 53.4 ms/chip on the CPU or 4.8 ms/chip on the MPS (after warming up, 10 chips, synchronization MPS)). Arithmetic mixing of finished cards ~0.02 ms/chip without taking into account transmission between devices. Older 90–100 ms/chip refer to different run/cache conditions and are not confirmed by new cold measurement. There may be no MPS on the organizer infrastructure, so the U-Net interface has a fallback CPU.

### What's Not Done (in this iteration))

- There is no independent test of organisers; the setup of models, alpha and thresholds is performed on one validation split, so improving the ensemble requires checking for other groups.
- Landcover error-analysis by class (no legend codes from the organizers)).
- External historical AF data were not involved, the official dataset was enough for F1>0.95, the task is clearly not in priority for TZ.

### FILES (AF track)

- `ml/src/kroma_ml/af_data.py`, `af_features.py`, `af_physics.py`, `af_model.py`, `af_metrics.py`, `af_threshold.py`, `af_postprocess.py`, `af_split.py`, `af_infer.py`, `af_torch_data.py`, `af_losses.py`, `af_unet.py` - AF modules; `AFUNetPredictor.predict_chip(...)` gives a binary mask 256×256 without changing the total `inference.py`.
- `research/experiments/af_eda.py`, `af_physics_baseline.py`, `af_ablation.py`, `af_final_model.py`, `af_unet_experiments.py`, `af_ensemble.py`, `af_group_cv.py` – reproducible experiments; checkpoints/probability maps stored locally.
- `tests/ml/test_af_pipeline.py` - 22 AF tests passed, `skipif` for no official dateset.
- `ml/pyproject.toml` – Added dependencies `pandas`, `scikit-learn`, `scipy`, `tifffile`, `joblib`, `lightgbm`, `kroma-geo`.

### NEXT (5 tasks))

1. Get independent test organizers and measure F1 LightGBM without re-selecting the model on these labels.
2. Check the transfer of the threshold to invisible tiles through nested calibration inside the train: CV thresholds 0.765-0.85 are markedly different from the 0.92 original artifact.
3. Get the `aux_landcover` legend, perform FP/FN analysis by surface types and then prepare the integration of the already selected `AFPredictor` into the common interface without changing Burn Severity.

## OFFICIAL TEST

`data/test/` contains the official test: 180 AF chips (`data/test/af/`), 89 BS chips (`data/test/bs/`), `meta.csv`, `sample_submission.csv` (447 lines = 180×1 + 89×3, `chip_id,class_id,rle` format confirmed by an exact comparison with `research/experiments/af_official_test.py audit`, results - `artifacts/af_test_audit.json`). Ground truth is missing in the test - F1/IoU on the test is not counted anywhere in the Pipeline.

### AF VALIDITY BUG AND DIRECTIVE

**OBSERVATION.** Old `valid_mask` required finite for all VIIRS channels. The 127 train chips and 69 test chips lack I1–I3 with the available I4/I5 (thermal), as the model is virtually independent of I1–I3 directly (features are built from I4/I5/I3, but I1/I2 is not included in the 39-channel feature stack ML models)). The old rule turned such chips completely invalid, losing real thermal detections.

**DECISION.** `valid_mask` (`ml/src/kroma_ml/af_data.py`) now requires only: VIIRS `valid==1`, finite I4 and I5, `mask != 255`. Pixels with invalid I1–I3 remain valid for the AF problem; missing reflective channels are reset at the feature construction stage (`np.nan_to_num`, already in `af_features.py`)).

**RE-VALIDATION (3-fold spatial GroupKFold, same split, corrected validity, tag `thermal_valid`)):**

| | precision | recall | F1 | threshold |
|---|---|---|---|---|
| fold 1 | 0.9415 | 0.9720 | 0.9565 | 0.84 |
| fold 2 | 0.9436 | 0.9790 | 0.9610 | 0.80 |
| fold 3 | 0.9500 | 0.9668 | 0.9583 | 0.745 |
| **mean ± std** | | | **0.9586 ± 0.0023** | |

For comparison: old (invalidity-distorted) CV - mean F1 = 0.9641 ± 0.0056. Adjusted figure **below** (more valid, but more I1-I3 quality pixels included in the score and is fair. Old numbers in the sections above are marked superseded by this block, without deleting history.

### AF OOF CALIBRATION

Threshold recorded on pooled out-of-fold train predictions (train-only, test not involved)): **threshold = 0.765, OOF precision = 0.9423, OOF recall = 0.9736, OOF F1 = 0.9577** (`research/experiments/af_oof_calibration.json`, `af_official_test.py calibrate`).

### AF FINAL MODEL (full train)

LightGBM, 39 Hard-negative mining (leakage-safe, OOF on train), trained in all 420 train chips (`research/experiments/af_fulltrain_model.joblib`, `af_fulltrain_manifest.json`). 180 703 learning pixels, 9 725 positive, 276 chips have given hard negatives. Runtime full-learning: 274c.

### AF TEST INFERENCE

180 test chips, threshold=0.765, **determinism confirmed** (3 runs give identical sha256 masks). 134 chips with predicted fire, 46 empty, 3 977 positive pixels total (0.034% of all valid pixels - in the same order as train-positives, no apparent positive-rate shift was detected). Consent with physics-baseline: 3 041 pixel intersection, 936 only LightGBM, 299 Physics alone is a moderate divergence expected for a contextually featured ML model. `artifacts/af_test_manifest.json`, `artifacts/af_test_chips.csv`.

## BS Baseline (playback of Roma's notebook))

**Source:** `cosmohack_dataset.ipynb`. The split file (`bs_temporal_split_train_2019_2023_val_2024.csv`) and checkpoints (`best_temporal_comboloss_sampler_dnbr_landcoverOnehot.pt`, `weighted_comboloss_sampler.pt`) referenced by the notebook are not on the disk - the notebook itself does not reproduce "as is". Moved the working part (AttentionUNet, ConvBlock/UpConvBlock/AttentionBlock, CombinedLoss=CE+Dice, SCL-based valid mask, IGNORE_INDEX=255) to `ml/src/kroma_ml/bs_*.py`, replaced by grouped `train_val_split` according to `fire_event_id` (BS train fire_event_id is 100% full, 224/224 unique - leakage is impossible to build)).

Active channel configuration (config B, as in Roma): pre+post B4/B8A/B11/B12 + RdNBR = 9 channels. `BS_CONFIGS` in `bs_features.py` also defines A (post-only), C (+dNDVI/dNBR1/dNBR2), D (+Sentinel-1 ΔV/ΔVH), E (+dem/slope/landcover) for future ablations (specification 18, not run in this iteration)).

**Result (val, grouped split, 15 eras, base=32, CE+Dice, SGD+cosine)):**

| metric | value |
|---|---|
| IoU class 0 (background) | 0.848 |
| IoU class 1 (low) | 0.196 |
| IoU class 2 (moderate) | 0.462 |
| IoU class 3 (high) | 0.579 |
| **IoU_burn** | **0.414** |
| **mIoU_severity (1-3)** | **0.412** |
| BS_score (0.35×IoU_burn+0.30×mIoU_severity) | 0.269 |

**OBSERVATION.** Class 1 (low severity) IoU is significantly lower 2/3 Confirms Roma's background problem↔Low-severity confusion. p.14). Not removed in this iteration (baseline reproduction only)).

`research/experiments/bs_baseline.py`, `bs_baseline_model.pt`, `bs_baseline_results.json`.

### BS TEST INFERENCE

89 test chips, mean 542 ms/chip (CPU)). All 89 chips received at least one burned pixel (diagnostics, not fitting - the model was not separately calibrated for test). `artifacts/bs_submission.csv`, `artifacts/bs_test_manifest.json`.

## SUBMISSIONS

**submission_v001.csv** (`artifacts/submissions/`): AF final LightGBM (threshold 0.765) + BS baseline AttentionUNet (config B). 447 lines, validator PASSED (row count, column names, exact pair set and order, no duplicates, no NaN, RLE roundtrip on real test dimensions, AF class_id=1 only, BS 1/2/3 classes without intersection, no leaks 255). Manifesto: `artifacts/submissions/submission_v001.json` (includes sha256, local CV/OOF metrics AF and BS, `public_score`/`public_f1_af`/`public_iou_burn`/`public_miou_severity` = null prior to actual loading on the platform)).

## LEADERBOARD

Not filled – the user has not yet downloaded a submission to the platform. `public_*` fields in the submission_v001 manifest will be updated manually after the result.

## SPEED

| stage | time |
|---|---|
| AF test inference | ~380 ms/chip (feature build dominant, 52 c/180 chips = features stage)) |
| BS test inference | ~542 ms/chip (CPU, AttentionUNet base=32, 512×512×9)) |
| AF full retrain | 274 with (420 chips, hard-negative mining included)) |
| BS train (15 eras)) | 819 c (179 train chips, CPU/MPS)) |

## RISKS

- BS baseline is trained on one grouped split (179/45), not on K-fold - the mIoU_severity confidence interval is unknown, retraining for a specific split is possible.
- BS model predicts burn on 100% test chips – it is not verified whether this is realistic or systematic bias (SCL/valid mask differences test vs train are not audited as carefully as for AF)).
- AF corrected-validity CV (0.9586) uses the same val to select the threshold as before - the optimistic bias is small, but not zero.
- U-Net/ResUNet and ensemble for AF remain worse than LightGBM even after valid fix (not retrained in this iteration, expect the same result)).

## LEADERBOARD V001 = 0.5526

Locally expected score before submission ≈ 0.35×0.958(CV F1_af) + 0.269(BS-part) ≈ 0.604. Public gave **0.5526**, gap 0.052. Below is a forensic audit of the source of the gap.

### P0. FORENSIC AUDIT V001 — PASS

`submission_v001.csv` is reassembled software from stored `af_submission.csv` + `bs_submission.csv` and **coincided** with an already uploaded file (same sha)256 `e275d087...`). 447 lines 180 AF + 267 BS (89×3The pair-set is exactly the same as `sample_submission.csv`, duplicates/NaN/leaks 255 No, BS classes don't overlap (all checks - `build_submission.py validate()`).

### P0/3. RLE - reversed from zero

Independent reference-realization (without using production code) on synthetic masks: top-left/top-right/bottom-left/bottom-right pixel, horizontal run, vertical run, two disparate run, empty, full — **All 9 cases coincided in a bit** on `kroma_ml.submission.encode_rle` (row-major/C-order flatten, 1-based start, `start length` pairs, encode→Decode round-trip exact). There is no official reference RLE example in the case documentation (in `sample_submission.csv` all lines are empty), in the evaluation criteria only the text requirement "RLE is decoded without errors, indices do not go beyond the chip, masks do not intersect" - which is already checked by the validator. **RLE recognized as not a source of rupture.**

### P0/4. BS CLASS MAPPING — PASS

`pred(argmax)==1/2/3 → class_id=1/2/3` path, `pred==0` is not exported, 255 is never submitted - confirmed by code (`bs_test_infer.py`, `multiclass_to_binary_rles`) and validator (`build_submission.py`: `if 255 in mask: raise`)). Masks of classes 1/2/3 inside the chip do not intersect in construction (`argmax` gives one class per pixel)).

### P2/11/12. The Biggest Cause: BS Burn Over-Prediction

**OBSERVATION 1 (100%-burn on test)).** On the test, all 89 BS chips received at least one burnt pixel, the average predicted burn-share **40.1%** chip. Official Train Statistics (`sev1_px+sev2_px+sev3_px` from `meta.csv`) gives the average burn share of the total **10.1%** (median 7.5%). Nana **validation** (Same model, same split) predicted burn-share is also overstated — **24.2%** against the true **11.5%** — That is, the problem already exists in-distribution, before any test-specific shift.

**OBSERVATION 2 (confusion matrix, val, pooled over all valid pixels)).**

| true \ pred | 0 | 1 | 2 | 3 |
|---|---|---|---|---|
| 0 (background) | 85.1% | **11.2%** | 2.9% | 0.8% |
| 1 (low) | 8.4% | 77.5% | 13.7% | 0.5% |
| 2 (moderate) | 0.4% | 9.2% | 84.3% | 6.1% |
| 3 (high) | 0.0% | 0.1% | 16.3% | 83.5% |

Class class 0 (a) more than the rest of the6.1M pixels vs 281K/335K/183K, therefore even 11.2% leakage 0→1 maker 803K extra "low severity" pixels - more than the entire true population of the class 1. That's it **class 0↔1 confusion**, The main failure mode predicted in the task.

**ROOT CAUSE.** `calculate_class_weights` (inherited from Roma's notebook, `kroma_ml.bs_torch_data.class_weights`) - a naive class frequency inversion (`counts.sum() / (4 × counts)`) rationed to the mean. This weighting greatly overestimates the weight of rare classes (1/2/3) in CrossEntropy, which pushes the model to systematically over-prediction burn classes - precisely the "dumb class weight" that the task warns about.

**OBSERVATION 3 (SCL/ignore audit, para.12).** Separately found discrepancy between our SCL-derived valid_mask (Rumina logic: invalid if ANY of {pre, post} SCL ∈ {nodata, saturated, cloud shadow, cloud medium/high, cirrus}) and official `meta.valid_frac`: correlation on a sample of 60 train chips in total **0.33**, There are chips where officially valid_frac = 0.94, and our mask gives 0% (`BS_tr_000024`: pre-shot completely in the cloud, post - clean; our AND-cloud-between-pre/post logic kills the entire chip, although officially it is 94% valid)). On average, our logic marks the invalid train **28%** pixels (mean valid_frac=0.717), 12/224 chips (5.4%) valid_frac≈0 - probably excessively strict.

**SCL-STRATEGY EXPERIMENT (paragraph 13, grouped split, same seedd)).** Retrained with "loose" SCL (invalid only nodata+saturated, no cloud/shadow/cirrus): result **better** — bs_score = 0.248 (own mask) / 0.267 (assessed under the old strict mask, for fair comparison) vs. baseline 0.269. **Conclusion: SCL aggression is not the main cause of rupture** (The discrepancy with `meta.valid_frac` is real, documented as an open risk, but does not eliminate burn over-prediction - do not drag this change further (`research/experiments/bs_baseline_results_loose_scl.json`, model `bs_baseline_model_loose_scl.pt`)).

### P8/22. TWO-STAGE CALIBRATION – Confirmed Improvement

Fixed logit bias for classes 1/2/3 (without retraining) coordinate-descent on val, optimizing directly `BSScore = 0.35×IoU_burn + 0.30×mIoU_severity` (not mIoU with background, not F1)):

| | baseline | calibrated (bias 0,-1.6,-1.8,-1.6) |
|---|---|---|
| IoU class 0 | 0.848 | 0.933 |
| IoU class 1 | 0.196 | 0.297 |
| IoU class 2 | 0.462 | 0.510 |
| IoU class 3 | 0.579 | 0.619 |
| **IoU_burn** | **0.414** | **0.571** |
| **mIoU_severity** | **0.412** | **0.475** |
| **bs_score** | **0.269** | **0.342** |

On the test, the same calibration reduced the total predicted burn pixels from 9.35M to 4.76M (burn-share with 40% to ~20Interest rates are much closer to train statistics 10%). `research/experiments/bs_calibrate.py`, `bs_calibration.json`, `bs_calibrated_results.json`.

## SUBMISSIONS (update))

**submission_v002.csv** (`artifacts/submissions/`): AF unchanged (same `af_submission.csv` as in v001), BS = same Roman architecture + logit-bias calibration. Validator PASS, 447 lines, sha256 `da25702c...`. `submission_v002.json` Manifesto.

## DIAGNOSTIC (AF-only / BS-only), item 5/26)

Prepared, **unloaded**: `artifacts/submissions/diagnostic/diagnostic_af_only.csv` (AF=v001, BS=all-empty) and `diagnostic_bs_only.csv` (AF=all-empty, BS=v001)). Empty RLEs passed their own roundtrip. `artifacts/submissions/diagnostic/manifest.json` Manifesto. The user himself decides whether to spend submission attempts on them - if the platform gives a non-zero score on all-empty parts, this will further confirm / refute the hypothesis about the contribution of AF vs BS.

## AF SUBGROUP VALIDATION (paragraph 6, corrected validity OOF))

| subgroup | n chips | precision | recall | F1 (@0.765) |
|---|---|---|---|---|
| missing I1–I3 (`i1_finite_frac<0.05`) | 127 | 0.905 | 0.968 | **0.9355** |
| I1–I3 present (`i1_finite_frac>0.95`) | 147 | 0.945 | 0.983 | **0.9636** |

The gap ≈0.028 F1 is real, but moderate (both subgroups >0.93) and already reflected in the honest 3-fold CV (0.9586) - 127/420−30% train chips in this mode, correlated with 69/180−38% on the test. **Solution: Thermal-only expert does not teach in this iteration** — The gap is not large enough to justify a separate model/routing (see paragraph 28 of the task: time is given to BS)). AF threshold=0.765 retained unchanged.

## EXPERIMENTS TABLE (this iteration))

| id | changed | IoU_burn | mIoU_severity | bs_score | conclusion |
|---|---|---|---|---|---|
| bs_baseline (Roman, strict SCL) | reference, v001 | 0.414 | 0.412 | 0.269 | baseline, fixed, not overwritten (`bs_roman_baseline_*`)) |
| bs_loose_scl | invalid only nodata+saturated | 0.378 (own) / 0.398 (strict-mask) | 0.385 / 0.427 | 0.248 / 0.267 | no better - rejected |
| bs_calibrated | + logit bias (0,-1.6,-1.8,-1.6) over baseline | **0.571** | **0.475** | **0.342** | adopted by → v002 |

## BEST BS (current))

IoU_burn=0.571, IoU_1/2/3=0.297/0.510/0.619, mIoU_severity=0.475, bs_score=0.342 (val, single grouped split, 45 chips - no K-fold CI, cm. RISKS).

## EXPECTED NEXT (3 most promising steps) - SUPERSED, see. Leaderboard V002 below

1. ~BS GroupKFold + hierarchical burn/severity head~~ - executed, see below.
2. ~~Load v002~~ – executed: public 0.6457 (+0.0931 vs v001), confirms calibration as the main source of gain.
3. ~~BS input ablation~~ - in the process, see below.

## LEADERBOARD V002 = 0.6457 (+0.0931 vs v001)

Written in `submission_v002.json` (`public_score_recorded_at`)). The only difference from v001 is the logit-bias BS calibration (AF byte-identical)). Growth +0.0931 on the leaderboard with local growth bs_score +0.073 (0.269→0.342 on single split) - same direction and comparable value, **confirm**, That calibration was the main source of rupture v001and what's local↔The public correlation for BS in this range is reliable.47).

### P1/P6. BS GROUP K-FOLD (fire_event_id, 3 fold, 10 eras, config B, CombinedLoss))

| | IoU_burn | mIoU_severity | BSScore |
|---|---|---|---|
| raw (no calibration) | 0.375 ± 0.026 | 0.336 ± 0.013 | 0.232 ± 0.006 |
| per-fold calibrated | **0.527 ± 0.010** | **0.393 ± 0.017** | **0.302 ± 0.008** |
| OOF pooled + global bias | 0.516 | 0.392 | 0.298 |

This is a more honest estimate than single-split (0.342) - fewer train chips on fold (149 vs 179) and half as many eras (10 vs 15) partly explain the difference; **0.298–0.302: BSScore’s operational waiting range on new domain/fold**, not 0.342. `research/experiments/bs_group_cv_baseline_results.json`.

Noticed: fold 2 era 7 gave `loss=nan` (transient, SGD momentum spike), the training continued and the fold metrics were not affected - not critically, but `torch.nn.utils.clip_grad_norm_` was added in subsequent experiments (`bs_feature_ablation.py`) as a prophylaxis.

### P5/P7. HIERARCHICAL BURN/SEVERITY LOSS - A negative outcome

Implemented as a single 4-class softmax head with `BurnSeverityLoss` (burn-logit via `logsumexp` severity channels + severity CE only on burn pixels) - does not require a change in architecture. The same GroupKFold protocol (config B, 10 eras)):

| | IoU_burn | mIoU_severity | BSScore |
|---|---|---|---|
| hierarchical, raw | 0.308 ± 0.014 | 0.314 ± 0.022 | 0.202 ± 0.010 |
| hierarchical, calibrated | 0.395 ± 0.044 | 0.343 ± 0.034 | **0.241 ± 0.025** |

Worse than baseline (0.302±0.008) on all metrics and with greater variance. **Recorded as a rejected experiment** — not to use further in this iteration (could help with more eras/literal separate heads, but priority is given to areas with already confirmed wins). `research/experiments/bs_group_cv_hierarchical_results.json`.

Found and fixed associated bug in `BurnSeverityLoss`: severity target for IGNORE_INDEX pixels (255-1=254) went beyond the class range and fell into `F.cross_entropy` before applying the mask - corrected by a safe-target pattern (similar to `DiceLoss`)). Added regression tests (`test_hierarchical_loss_handles_ignore_index_and_improves_toward_target`, `test_hierarchical_loss_all_ignored_is_finite`)).

### P2. CLASS 0↔1 ERROR BREAKDOWN (baseline calibrated model, single-split val)

**Landcover** (Percentage of background pixels wrongly predicted as class 1):

| code (WorldCover)) | type (by Roma grouping)) | FP rate 0→1 |
|---|---|---|
| 40 | cropland | **3.56%** |
| 30 | grass/moss | 1.75% |
| 10 | forest/shrub | 1.18% |
| 90 | wetland | 0.28% |
| 80/60/50/20 | elsewhere | 0.18–0.70% |

**RdNBR bin** (background pixels: a sharp jump in FP-rate exactly in the `RdNBR ∈ [0.00, 0.10]` zone — **4.26%** vs. <0.15% outside this zone. This is a classic decision-boundary error: weak but nonzero seasonal spectral changes (not fire) in particular on arable land (harvest, ploughing) give the same sign and small value of RdNBR as a weak burn.

**OHEM-diagnosis (keep_ratio=0.4, same val)).** Hard-40% of pixels: **54.9%** code 40 (cropland)), **32.7%** — 30 (grass/moss), i.e. 87.6% of hard pixels is cropland/grassland, not cloud/invalid. True-class distribution within hard-pool: class 1 share 7.6% (vs 3.5% overall, ×2.1), class 2 - 6.9% (vs 4.2%, ×1.65)). **Conclusion: OHEM at Roma`keep_ratio=0.4`) concentrating correctly on background↔weak confusion in cropland/grassland, not over noise** — 9/10 of the task warned about this risk, the risk is not confirmed. Further OHEM-grid (0.25/0.40/0.60) did not run in this iteration (calibration and feature-ablation had higher expected ROI at the time invested)).

**Withdrawal by direction:** The main remaining reserve is to make the models available landcover code as explicit input (already in config E) and/or limit false burn-triggering precisely in the zone of low positive RdNBR on cropland/grassland - natural next step. FEATURES below.

### P8-P10/P12. FEATURE ABLATION AND TTA (single grouped split 179/45, 15 eras, CombinedLoss, calibrated))

| config | channel | IoU_burn | mIoU_severity | BSScore | conclusion |
|---|---|---|---|---|---|
| B (baseline) | S2 pre/post 8 + RdNBR | 0.571 | 0.475 | 0.342 | reference |
| C | + dNDVI, dNBR1, dNBR2 | 0.561 | 0.492 | 0.344 | +0.002, below the +0.01 threshold |
| D | C + Sentinel-1 ΔVV/ΔVH | 0.541 | 0.463 | 0.328 | worse not to use |
| E | D + DEM/slope/landcover (scalar) | 0.536 | 0.448 | 0.322 | worse not to use |

The spread between single-split launches is comparable to the difference between configs, so no add-on is considered confirmed. Landcover added as scalar code (not one-hot/embedding) - this is a limitation of the experiment, the hypothesis "landcover helps cropland FP" by the E test is not conclusively refuted. Full log C/D/E: `research/experiments/bs_feature_ablation_CDE_15ep.log`.

TTA (baseline model, each variation with its own bias calibration on val): none 0.3424 → 2 flips 0.3448 → dihedral-8 0.3476 (+0.005 at 8x infernance cost)). Below the threshold +0.01, submission is not created because of this. `research/experiments/bs_tta_results.json`.

Check the number of epochs (config B, 25 vs 15 eras, the same split, calibrated): 0.3478 vs 0.3424 (+0.005, bias 0/-1.6/-1.4/-1.0) - within the noise of one split, below the threshold +0.01. Result: `research/experiments/bs_feature_ablation_B_25ep_results.json`.

### Single split, calibrated BSScore)

The calibration gave a major leap (0.269→0.342, public +0.0931). After it, all the tested levers give ≤ +0.005: dNBR indexes +0.002, TTA-8 +0.005, 25 eras +0.005; Sentinel-1, terrain/landcover, hierarchical loss - worse than baseline. **Submission v003 not created**: No change has passed the +0.01 threshold, and the combination (full learning on 224 chips + 25 eras + TTA) cannot be verified locally because there is no held-out data to calibrate it bias.

## ITERATION 3: DATA-CENTRIC BS PIPELINE (`ml/src/kroma_ml/bs_crop.py`, `research/experiments/bs_train.py`)

### Current state audit

Leaderboard: v001 0.5526, v002 0.6457 (BS logit calibration only, AF unchanged)). Roma's new notebook (`cosmohack_dataset (1).ipynb`) contains a 100-ephere OHEM training log (keep_ratio 0.4, no class split weights, temporal val=2024, raw without calibration): BSScore noises in the range of 0.20–0.35, peak 0.351 on the era 60, finale 0.323; predicted burn share 7–8% vs true 9.2% (no over-prediction)). There is no checkpoint and split file on the disk, so it is impossible to evaluate this model on a general evaluator; in the log, you can only look at it as an independent confirmation that OHEM does not flash burn without class weights.

### SCL audit (the proportion of pixels in each class lost by ignoring))

| ignore-rule | class 0 | class 1 | class 2 | class 3 |
|---|---|---|---|---|
| Strict (Old Roma Logic), Train | 28.7% | **19.9%** | 13.8% | 10.3% |
| strict, val | 32.6% | **36.9%** | 27.3% | 10.4% |
| true-invalid only (SCL 0,1) | 0.95% | 0.0% | 0.0% | 0.0% |

The old evaluator (strict mask) therefore threw 37% of class 1 on val: all past BSScore digits (0.342, etc.) are counted on the light subset. Now each experiment is evaluated by two evaluators: `strict_mask` (comparable to the past) and `all_valid_mask` (all pixels except SCL 0/1; closer to a possible official count). v002 model: 0.342 (strict) / 0.270 (all-valid, 0.282 after recalibration).

### Found defect of signs

The former `rdnbr` was counted with B11 (SWIR1), and the standard S2 NBR = (B8A−B12)/(B8A+B12)). Added `dnbr12`/`rdnbr12` (stabilized denominator, clip, NaN/inf test)).

### What's done

- crop sampler 256×256 without resize; mix random/class1/burn = 50/30/20; target pixel falls into a random position inside the crop; validation is always on full 512×512 scenes;
- the same geometric transform (rot90/flip) for all modalities and target; one common brightness coefficient U(0.95.1.05), p=0.3, on pre and post, indices are recalculated after; categorical channels are not interpolated, assert on `unique(GT) ⊂ {0,1,2,3,255}`;
- OHEM (`OHEMLoss`), soft-capped weights, true dual-head (`DualHeadAttentionUNet` + `DualHeadLoss`, ordinal member));
- landcover as 5 one-hot groups (forest/shrub, grass/moss, cropland, wet/water, other), NDVI/dNDVI, SCL quality channels (cloud/shadow/dark/water/snow/clear));
- 8 new regression tests (`tests/ml/test_bs_crop.py`), a total of 52.

Note: brightness-augmentation changes only the raw channels of reflection; normalized indices (dNBR, NDVI) are invariant to the general scale, so "indices recalculation" here is formally true, but practically does not create a signal.

### Screening results (single grouped split 179/45, seed 42, config B if not specified, 30 crop eras×2 ≈ 15 full-scene eras by compute))

| id | what | strict raw→cal | all-valid raw→cal | bias (all-valid) |
|---|---|---|---|---|
| ref_full_ce | full scene, CE, no weights | 0.289→0.316 | 0.199→0.237 | (0,1.0,-0.4,-1.0) |
| crop_ce | crop 256 mix 50/30/20, CE | 0.345→0.347 | 0.274→0.297 | (0,-0.2,-1.0,-1.6) |
| full_ohem25 | full scene, OHEM 0.25 | 0.294→0.335 | 0.242→0.275 | (0,1.0,0.4,0.4) |
| crop_ohem25 | crop + OHEM 0.25 | 0.356→0.357 | 0.301→0.301 | (0,0,0,0) |
| crop_ohem10 | OHEM 0.10 | 0.316→0.316 | 0.276→0.276 | |
| crop_ohem40 | OHEM 0.40 | 0.361→0.362 | 0.313→0.315 | |
| crop_ohem25_bright | + brightness p=0.3 | 0.350→0.359 | 0.305→0.312 | |
| crop_ohem25_true | ignore only SCL 0/1 | 0.352→0.353 | 0.319→0.321 | |
| crop_ohem25_B12 | NBR on B8A/B12 | 0.364→0.365 | 0.320→0.320 | |
| crop_ohem25_lc5 | B12 + 5 Landcover Groups | 0.366→0.368 | 0.325→0.328 | |
| V002 model (for comparison)) | old pipeline + bias(0,-1.6,-1.8,-1.6) | 0.342 | 0.270 | |

Conclusions:1) crop sampler - main lever (+)0.03 strict, +0.06 all-valid relative to full-scene with the same loss/data (2) OHEM 0.25–0.40 add ≈+0.01, OHEM 0.10 It hurts (too narrow hard-pool). (3) at crop+OHEM calibration bias ≈ 0 The model no longer overstates the burn initially, the dependence on post-hoc bias (which made v the first one)002) has disappeared; (4) true-invalid ignore slightly worse on strict, but better on all-valid - that is, it wins on the pixels that the old evaluator hid; (5) B12 And landcover gives small, but unidirectional increases≈+0.003…+0.008), each at the noise level of one seed. All differences <0.02 and not confirmed on GroupKFold.

### Stacked recipe and confirmation on GroupKFold

Stack: crop 256 (50/30/20) + ignore only SCL 0/1 + shared brightness p=0.3 + OHEM 0.40 + B12-NBR + 5 landcover groups (single split 179/45, all-valid evaluator, calibrated): `stack_lc5` 0.346 (strict 0.370), `+NDVI` 0.350, `+quality-каналы` 0.345, `dual-head` 0.348, `dual+ordinal 0.1` 0.348. All within ±0.005 of each other - plateau ≈0.345-0.35 all-valid; quality channels, NDVI, ordinal member did not give a confirmed increase. True dual-head (total encoder, individual burn/severity heads) gives a better IoU class 1 (0.367) and does not require bias (≈0), but the total BSScore is equal to OHEM.

**3-fold GroupKFold (fire_event_id, the same folds as baseline-CV), without any calibration (raw), strict evaluator:**

| recipe | fold1 | fold2 | fold3 | mean |
|---|---|---|---|---|
| old baseline (calibrated, 10 full-scene eras)) | 0.308 | 0.306 | 0.293 | 0.302 ± 0.008 |
| NDVI + OHEM 0.4 (raw) | 0.360 | 0.360 | 0.349 | 0.356 |
| dual-head (raw) | 0.371 | 0.353 | 0.353 | 0.359 |

Pooled OOF, global bias, BSScore (strict/all-valid): NDVI+OHEM 0.361 / 0.343; dual 0.359 / 0.339; **blend 50/50 0.365 / 0.346**; old baseline pooled OOF 0.298 (strict)). Increment ≈ +0.06 on each of the three folds; calibration bias in the new recipe ≈0...0.4 (v002 was −1.6...−1.8)). Disclaimer: New recipe learned 30 crop-age ≈ 15 The full-scene era of compute vs 10 The old CV. from past measurements (15→25 epoch ≈ +0.005that explains ≤ 0.01 from 0.06. Blend gives +0.004-0.005 to the best single model. IoU pooled OOF (blend, all-valid) classes: burn 0.542, class1 0.343, class2 0.533, class3 0.683, mIoU_severity 0.520.

### V003 (not loaded))

`artifacts/submissions/submission_v003.csv` (+`.json`): AF unchanged (byte rows AF v002), BS = full-train (all 224 chips) ensemble of two models (logit-average) + OOF global bias (0, 0.4, 0.2, 0.2)). Validator PASS, 447 lines, sha256 `6a61626d...`. Inference determined (double run of each chip), 120 ms/chip on MPS, 24 s/89 chips. Test: Average predicted burn share 18.7% (v002 ≈ 20%, v001 40%; train 10%) - only logged, model choice not affected. v001/v002 unchanged (sha256 matched)).

Expectations: BSScore = 0.35·IoU_burn + 0.30·mIoU_severity is exactly the BS part of public Score, so the local increase is transferred to Score directly. Estimates of growth relative to the v002 model differ according to the protocol: on one split +0.028 (strict) / +0.06...0.07 (all-valid), on GroupKFold +0.06 (but the old CV had 10 eras instead of ≈15)). The reasonable waiting range is +0.03...+0.06 to Score, i.e. about 0.68-0.70; this is an expectation, not a guarantee (in v002 local +0.073 gave public +0.093)).

Hard-negative crop pool, border sampler/aux head, chip-level head, cloud-aware S1 + optical dropout, boosting refiner, external pretraining, AF thermal expert.

## ITERATION 4: HARD-NEGATIVE / BOUNDARY SAMPLING over V003)

**v003 = public 0.72** (v002 0.6457, v001 0.5526) – gold baseline; v003 artifacts (csv, manifest, full-train checkpoints, OOF logits, calibration) have not changed (sha256 csv `6a61626d…` matches)).

Hypothesis: v003 Won by crop sampling + OHEM, so the next natural step is to add hard negatives to the sampler 0→1, hard positives class 1boundary 0↔1).

**Pools** (179 train-chips single split; hard-examples were mined only on train-chips from the OOF logits v003-model `cv_ndvi`; reservation: OOF logits of the j chip are obtained by a model trained including on validation chips, that is, second-order leakage, validation marks are not included in the selection):

| pool | pixel | composition |
|---|---|---|
| hard negatives (GT=0, confident burn, p_burn>0.5)) | 1 870 864 | cropland 75.5%, grass/moss 22.2%, forest 1.1%, wet/water 1.1%, other 0.1% |
| hard positives (GT=1, sure 0 or 2, conf>0.6)) | 666 652 | — |
| 0↔1 boundary (dilation 2px on GT) | 1 797 741 | — |

Iteration 2: False burn focused on cropland/grass (97.7% hard negatives)).

**Controlled fine-tune** (model 1 v003: B12+landcover5+NDVI, OHEM 0.4, lr 3e-4, 8 eras, same single split, config/loss/augmentation). Control `ft_ref` - the same fine-tune with a reference sampler to separate the sampler effect from further training:

| run | sampler | strict cal | all-valid cal | IoU1 (all) | 0→1 FP px (all) | 0→1 rate | 1→0 FN px | 1→2 px |
|---|---|---|---|---|---|---|---|---|
| v003 model 1 (without fine-tune)) | 50/30/20 | 0.3692 | 0.3495 | 0.358 | 259 904 | 2.44% | 153 278 | 32 138 |
| ft_ref (control)) | 50/30/20 | 0.3735 | 0.3511 | 0.356 | 249 682 | 2.34% | 165 000 | 21 912 |
| ft_A | 40 rnd/25 c1/15 burn/20 hardneg (+hardpos in class1 quota)) | 0.3726 | 0.3519 | 0.367 | 271 340 | 2.55% | 146 591 | 26 381 |
| ft_B | 35/25/15/15 hardneg/10 boundary01 | 0.3696 | 0.3490 | 0.357 | 286 655 | 2.69% | 141 532 | 36 795 |

**The result is negative.** Regarding control: ft_A −0.001 / +0.001, ft_B −0.004 / −0.002 (threshold +0.008 not passed anywhere close)). IoU1 at ft_A grew up0.011 All-valid, but due to more class predictions 1 (recall 0.581→0.612, precision 0.480→0.478) and not because of fewer falsehoods: 0→1 FP **grew** (250k→271k at A, →287k at B), and more calibration bias was required. The Success Criterion (IoU)1↑ upon 0→1 FP↓) not completed. Possible explanation: hard negatives on cropland – pixels near mild burns and with ambiguous marking; oversampling shifts the solution boundary towards class 1, rather than teaching you to separate cropland-season and burn. Border sampler 0↔1 add 1→2 confusion28–37k 16–22k).

Additional training in itself (+8 eras on a small lr) gives only +0.002 ... +0.004 - training v003 is already saturated in duration.

**Didn't start (consciously)):** 3-fold confirmation and blend with v003 – don’t make sense without passing the fast-screen threshold; Focal Tversky (α=0.2, β=0.8) — β=0.8 False positives are more powerful than false ones, and our defect is false positives 0→1Therefore, it is expected to worsen the target metric; Romine standalone BSScore ≈0.323 below our OOF.

**Next Next post: Neural Candidate – Not Real.** The winner of the neural network track remains v003-pipeline; candidate artifacts were not created (`artifacts/submissions/candidates/` is not created), v004/v005 was not engaged in.

## V003 PUBLIC = 0.72

v003 (`artifacts/submissions/submission_v003.csv`, sha256 `6a61626d…039c`) - current gold baseline, frozen. AF unchanged from v002. BS: ensemble of two full-train Attention U-Net (B12 NBR + 5 landcover + NDVI bands with OHEM 0.4); B12 + landcover with true dual head, crop 256Ignore only SCL 0/1Mixed Logistics + OOF bias0, 0.4, 0.2, 0.2). GroupKFold pooled OOF: 0.3458 all-valid / 0.3650 strict. Checkpoints, OOF logits (`/tmp/bs_oof/cv_{ndvi,dual}_f*.npz`) and v003 calibration were used for reading only.

## CLASS-1 / PHYSICS-GUIDED IMPROVEMENT TRACK

Code: `kroma_ml.bs_physics`, `kroma_ml.bs_cv` (canonical evaluator + prediction contract), `kroma_ml.bs_sampling`, `kroma_ml.bs_track_losses`, `kroma_ml.bs_fast_models`; scripts `bs_class1_audit.py`, `bs_physics_baseline.py`, `bs_eval_contract.py`, `bs_fusion.py`, `bs_track.py`. Tests: `tests/ml/test_bs_class1_track.py`.

**Canonical evaluator.** `python research/experiments/bs_eval_contract.py <npz...>`: NPZ with `ids` + one of `logits (N,4,512,512)` / `probs` / `labels (N,512,512)`, optionally `class_order = (background, low, moderate, high)`. Counts per-fold (Folds are determined by chip_id through GroupKFold by fire_event_id), mean/std/worst, pooled, per-class IoU/P/R/F1 on strict and all-valid masks. On OOF-logits of the queue, cv_ndvi fold 1 reproduces its own queue numbers accurately (0.3597 / 0.3360) - to compare other people's results (including "0.81") only through it.

**Class-1 audit (224 scenes, GroupKFold 3)).** Class 1 eat 100Percent of scenes, 4.0% of pixels; 41 187 component 3 px, 73percentage ≤ 9 px, but they only carry 3.2% pixel class 1; 43% pixel class 1 relate to another class;32Percent background, 18% — class 2). Normal pre/post alignment (median shift 0.05 px), mask vs dNBR without shift in 94% of scenes.

**Main conclusion.** Inside the burner circuit, severity is almost determined by physics: dNBR (B)8A/B12) overlaps the distributions 1↔2 = 0.14, 1↔3 = 0.01. If you take the contour from the markup, and severity - from the organizer's thresholds on landcover, BSScore = **0.615**, IoU1 = 0.917 (with one global threshold 0.596)). The only badly separated couple 0↔1 (overlap) 0.22): the outline of the markings is defined by the external perimeter (on the installations - arcs-buffers), and the same field spectrum behind the perimeter is marked with the background. OOF v003component 95% FP class 1 They come from the background 1→0 = 39%, 1→2 = 3%.

**SCL audit.** Strict SCL ejects 23.1% of class 1 pixels (mostly cirrus clouds of SCL 10 and medium probability clouds of SCL 8); in the ejected pixels, dNBR is the same as in the remaining ones (median 0.121 vs 0.117), that is, these are real labels, not cloud noise. 29 scenes lose >90% class 1. True-invalid (SCL 0/1) and "Impossible Pixels Only" modes match.

**Physics baseline (thresholds are selected only on train-folds, the same evaluator)):**

| variant | BS all-valid | BS strict | IoU1 | burn IoU |
|---|---|---|---|---|
| USGS dNBR 0.10/0.27/0.66 without selection | 0.183 | 0.226 | 0.168 | 0.371 |
| dNBR, 3 thresholds (full-train 0.105/0.205/0.39)) | 0.240 | 0.300 | 0.194 | 0.372 |
| RdNBR B12 / legacy B11 | 0.200 / 0.148 | 0.238 / 0.191 | 0.116 / 0.076 | 0.408 / 0.293 |
| RdNBR-gate 0.5 + dNBR-severity | 0.258 | 0.308 | 0.160 | 0.413 |
| **Landcover organizational thresholds (Figure 6))** | 0.247 | 0.301 | 0.230 | 0.358 |

Transition B11→B12 Physically confirmed (+)0.05). ML (v003) is higher than best physics at +0.09 all-valid – almost entirely due to the circuit (burn IoU 0.54 vs 0.41)).

**Post-hoc fusion "ML circuit + single dNBR threshold"** on OOF cv_ndvi/cv_dual neutral (−0.002...0.000): the neural network has already learned global bands. The increase appears only with **landcover-conditional** Organizer thresholds (see below)).

**Long queue of ~15 candidates (sampler/hard/loss/prior/context/SegFormer/MobileUNet) stopped on new priority**, control run of c0 was interrupted on the 15/30 era, there are no results of neural network ablations. The code (`bs_track.py`, `bs_track_queue.py`, `bs_infer_bench.py`) is ready and covered with tests.

## FAST V004 CLASS1/PHYSICS TRACK

**Organizer physics** (Case formulation, pp. 8, figure 6; machine-readable in `research/experiments/bs_v004_physics_thresholds.json`). NBR = (B8A − B12)/(B8A + B12), dNBR = NBR_pre − NBR_post, RdNBR = dNBR/(√|NBR_pre| + 0.01), clip ±6. Thresholds 0/1, 1/2, 2/3: forest/shrub 0.10/0.27/0.66 (WorldCover 10, 20); grasses 0.062/0.204/0.386 (30); arable land 0.07/0.177/0.38 (40); swamp/plain 0.075/0.331/0.677 (90, 95)). For rare 50/60/80 codes, floodplain thresholds are empirically better suited - used as fallback. Coincidence with GT severity inside the GT circuit: 0.938 (global matched threshold - 0.900)). The GEE-notebook of the organizers (`MK1_Krasnoyarsk_fires_GEE.ipynb`) uses only the global scale EFFIS 0.10/0.27/0.44/0.66 and B8 instead of B8A - for the reference case it is not a source of thresholds.

**Refiner.** LightGBM multiclass (400 trees, 63 sheets) on OOF probabilities v003 (training for fold k is only on OOF pixels of two other folds). 37 features: p0.p3, p_burn, p1−p0, top-1/top-2 margin, prediction v003, dNBR, RdNBR, distances to organizing thresholds (d_low/d_mod/d_high), |d_low|), Organizer class, landcover group, NDVI pre/post/dNDVI, B4/B8A/B11/B12 pre/post, SCL pre/post, windows 5/15/31 p_burn/p1/dNBR/share of physical burns, iconic distance to predicted boundary. Selection on stage: 5k evenly + up to 2k errors v vs003 (hard pos/neg) + 1.5k class 1 + 1.5k GT boundaries 0↔1. Main gain features: d_low 21%, organizer class 20%, d_mod 7%, dNBR 6%, p1 5%. Applicable **only** in the zone of p_burn ∈ [0.3, 0.7] or ≤ 2 px from the predicted circuit; confident class 2/3 (p ≥ 0.6) is not moved. Class bias (0, 0.4, 1.0, 0.8) is selected on pooled OOF.

**Hybrid C.** Burn/no-burn contour – from (refined) ML; severity = argmax(log p_sev + 1.0 · onehot))). Physics Can't Draw Harm from Zero (checked by Test)).

| experiment | IoU_burn | IoU1 | IoU2 | IoU3 | mIoU | BSScore all-valid (mean / worst fold) | BSScore strict | 0→1 FP | 1→0 FN | cropland 0→1 FP | runtime |
|---|---|---|---|---|---|---|---|---|---|---|---|
| v003 (OOF, frozen) | 0.542 | 0.343 | 0.533 | 0.683 | 0.620 | 0.3455 / 0.3406 | 0.3648 | 0.0354 | 0.339 | 0.0477 | — |
| organizer physics, full scene | 0.358 | 0.230 | 0.397 | 0.591 | 0.504 | 0.2468 / 0.2427 | 0.3008 | 0.1329 | 0.033 | 0.1113 | 5 ms/chip CPU |
| oracle: GT contour + organizer severity | 1.000 | 0.917 | 0.849 | 0.882 | 0.912 | 0.6148 / 0.6117 | 0.6145 | 0 | 0 | 0 | — |
| hybrid A: v003 contour + organizer severity | 0.542 | 0.357 | 0.540 | 0.699 | 0.630 | 0.3493 / 0.3450 | 0.3685 | 0.0322 | 0.339 | 0.0422 | +0.3 s |
| hybrid B τ=0.7 | 0.542 | 0.359 | 0.562 | 0.719 | 0.640 | 0.3536 / 0.3485 | 0.3732 | 0.0337 | 0.339 | 0.0447 | +0.3 s |
| hybrid C w=1.0 | 0.542 | 0.360 | 0.562 | 0.721 | 0.641 | 0.3538 / 0.3487 | 0.3735 | 0.0336 | 0.339 | 0.0444 | +0.3 s |
| refiner, wide gate | 0.527 | 0.362 | 0.546 | 0.700 | 0.630 | 0.3448 / 0.3418 | 0.3589 | 0.0491 | 0.203 | 0.0543 | ≈1.9 s |
| refiner, p_burn∈[0.2,0.8] ∪ ≤2 px | 0.557 | 0.390 | 0.543 | 0.690 | 0.636 | 0.3568 / 0.3528 | 0.3720 | 0.0433 | 0.185 | 0.0551 | ≈1.9 s |
| refiner, p_burn∈[0.3,0.7] ∪ ≤2 px | 0.561 | 0.388 | 0.543 | 0.691 | 0.636 | 0.3584 / 0.3546 | 0.3752 | 0.0388 | 0.228 | 0.0510 | ≈1.9 s |
| **v004 = refiner [0.3,0.7] ∪ ≤2 px + hybrid C** | **0.561** | **0.399** | **0.566** | **0.724** | **0.653** | **0.3652 / 0.3608** | **0.3822** | 0.0378 | 0.228 | 0.0492 | ≈2.0 s |

Gain v004 over v003 on OOF: +0.020 all-valid (by folds +0.0205 / +0.0186 / +0.0202), +0.017 strict; IoU1 +16% reg., IoU2/IoU3 are also growing, IoU0 is not changing (0.922)). The choice of ~9 gate/hybrid variants on the same OOF gives a little optimism (the spread between neighboring versions ≈0.002), it is much smaller than the increase.

**What exactly has improved?:** (1) class passes 1 (1→0 FN 0.339 → 0.228) - refiner on d_low and the context returns a weak fumes inside the contour; (2) severity within the contour - landcover organizing thresholds (+)0.008 on their own). **What hasn't improved:** False Harm on arable land - cropland 0→1 FP 0.0477 → 0.0492. . . in the arable section, 0 < dNBR < 0.177"share of FP gari" 0.098 → 0.109. Hard Negatives This Refiner Is Not Solved.

**Submission v004** — `artifacts/submissions/submission_v004.csv` (sha256 `743138c2…17c8`), `submission_v004.json` manifesto. AF line byte-identical v003; BS = v003 full-train ensemble (on the test played byte for all 89 scenes) + refiner, trained on OOF of all 224 scenes, + hybrid C. Validator PASS, 447 lines (AF 180, BS 267)). Inferencing is deterministic (repeated full run, including retraining refiner, gave the same sha). 4.8% pixels changed on the test; class volume shift (sev1 +7%, sev2 + 10%, sev3 −6%) of the same sign as OOF for class 1/2. Reproduction: `bs_v004_fasttrack.py` → `bs_v004_combo.py` → `bs_v004_logits.py` (torch/MPS) → `bs_v004_infer.py --refiner --band 0.3 --radius 2 --bias 0,0.4,1.0,0.8 --hybrid hybrid_C_w1.0` → `bs_v004_build.py`. Torch-stage and LightGBM-stage are separated by processes: when loading torch before lightgbm on macOS, libomp (segfault) conflicts).

**External data (only inventory, nothing downloaded)).** HLS Burn Scars (USA, HLS) 30 Uh, 6 channels, ~800 chip 5122, Binary Mask, License Listed as MIT – Binary Pre-training; CaBuAr (California, S2 L2A pre/post, CAL FIRE binary masks, CC BY-NC 4.0) – binary pre-education, non-commercial only; CEMS Wildfire (S2, demarcation masks and grading Copernicus EMS, 500+ images) is the only candidate with severity, but the taxonomy is different (damaged/destroyed)); EO4WildFires (S1+S2+meteo, 31 730 Events, EFFIS annotations) – binary / dimensional signal; SSL4EO-S12 (251k locations S1/S2, CC BY 4.0, there are weights for 13 channels S2) - self-learning pretraining encoder. Direct supervised transfer severity is not possible: in the case, severity is relative to landcover and is given by organizing thresholds.

**NEXT (maximum expected effect)).** The perimeter is the only thing left between 0.365 and oracle 0.615. (1) Short fine-tune of one component v003 with d_low/d_mod/d_high channels and hard negatives of arable land (GT = 0, high p_burn, 0 < dNBR < 0.177) - refiner these FPs did not remove. (2) Perimeter context: crops 384/512 (another process had already started ctx256/384/512) or global attention. (3) Second level Refiner with signs of perimeter shape (contour bulge, distance to the large core)).

## LEADERBOARD: v003 = 0.7298, v004 = 0.7406 (gold baseline)

OOF gains v004+0.020 (all-valid) gave on public +0.011. v001–v004, their manifestos, OOF logits, full-train checkpoints and refiner artifacts unchanged (sha256 verified)).

## V005 CONTOUR TRACK: main-fire components

Code: `kroma_ml/bs_v005.py`, `research/experiments/bs_v005_oof_v004.py` (OOF cards v004, reproduce 0.3652/0.3822 accurately), `bs_v005_contour_diag.py`, `bs_v005_components.py`, `bs_v005_refiner2.py`, `bs_v005_combo.py`, `bs_v005_infer.py`, `bs_v005b_infer.py`, `bs_v005_build.py`.

**Where is the score (OOF, all-valid)).** GT circuit + severity v004 → 0.621; contour v004 + GT-severity → 0.380 (+0.015); pixel best-of v003/v004 → 0.412. The main contour error is false fumes: 3.46 million FP-pixels (59% of the area of GT-gary) against 0.65 million FN.

**Core → class1 (hypothesis tested, rejected in simple form)).** Share of pixels at a distance of ≤ 2 px from the core:

| core | GT class 1 | FP-grill v004 | FP arable |
|---|---|---|---|
| GT class 2/3 (oracle)) | 0.19 (≤ 5 px: 0.37) | 0.02 (> 20 px: 0.88) | 0.01 |
| predicted (p_burn ≥ 0.8 or severity 2/3)) | 0.66 | 0.82 | 0.84 |
| Physical (organizer class ≥ 2)) | 0.20 | 0.52 | 0.57 |

The connection with every confident kernel does not separate the false kernel: the false regions are themselves confident kernels (whole fields)). Hysteresis (T_high 0.7–0.9, T_low 0.3–0.5, radius 0/2), distance-conditioned threshold and global threshold 0.40–0.65 – all within ±0.001 from v004.

**What worked: a component against the “major fire” of the chip.** Coupled components of predicted burns (p_burn ≥ 0.5) → 34 features (area, shape, p/dNBR/organizer physics, landcover, ring around, rank by area / severity 3 in the chip), **distance to the main component of the chip**, fraction of chips, number of components). LightGBM real vs false (overlap ≥ 0.5 with GT), learning only on components of other folds. AUC 0.80/0.83/0.81. The main feature is `dist_main_sev3` (23% gain): the markings circumnavigate the main fire, and the fields similar to the fumes away from it are background.

**Refiner v2** (pixel, binary burn: probabilities v004, organizational physics, context 11/41 px, distance to the main component of the chip, chip statistics, signs of its component at p_burn 0.5 and 0.3 levels; error sampling v004 (FP, FN, class 1, boundary). Probabilities of the component classifier are not consciously submitted to it (avoid the third level of stacking)). Main features: `dist_main_sev3_px` 16%, `d_low` 10%, chip burning share 6%.

| experiment | BS all-valid | BS strict | IoU_burn | IoU1 | IoU2 | IoU3 | 0→burn FP | 1→0 FN | cropland FP | grassland FP | worst fold | contour runtime |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| v004 | 0.3652 | 0.3822 | 0.561 | 0.399 | 0.566 | 0.724 | 0.066 | 0.228 | 0.095 | 0.050 | 0.3608 | — |
| hysteresis (best, h0.9 l0.5)) | 0.3661 | 0.3829 | 0.564 | 0.401 | — | — | 0.065 | 0.237 | 0.094 | 0.048 | 0.3610 | <0.1 s |
| distance-conditioned (best)) | 0.3648 | 0.3817 | 0.561 | 0.400 | — | — | 0.067 | 0.221 | 0.097 | 0.050 | 0.3611 | <0.1 s |
| component classifier, drop p < 0.3 (v005a) | 0.4129 | 0.4312 | 0.645 | 0.462 | 0.651 | 0.762 | 0.033 | 0.310 | 0.042 | 0.034 | 0.4101 | 0.2 s |
| refiner v2 alone (t = 0.35) | 0.4077 | 0.4180 | 0.631 | 0.468 | 0.645 | 0.753 | 0.042 | 0.180 | 0.046 | 0.051 | 0.4024 | 1.3 s |
| **components + refiner v2 (v005b)** | **0.4298** | **0.4401** | **0.671** | **0.504** | **0.669** | **0.775** | **0.032** | **0.218** | **0.036** | **0.039** | **0.4267** | 1.5 s |

v005b by fold: 0.4267 / 0.4325 / 0.4303 v004 0.3667 / 0.3682 / 0.3608 (+0.060 ... +0.070)). Rule: Remove components with a probability of < 0.3; refiner v2 adds gar at p ≥ 0.8 outside retracted components and removes pixels from p < 0.2 inside the remaining ones; severity of added pixels - v004 + organizer prior. Thresholds are selected from ~50 variants for the same OOF; neighboring variants give 0.425–0.430, so the expected optimism of the choice is 0.003.

**Transfer to the test.** v004 on the test is reproduced byte before post-processing. Share of changes: removed 22.0% v004 (OOF 30.3%), added 5.7% (OOF 8.2%) - same direction, ~0.72 from OOF scale (full-train models on the test more confidently)). Considering that +0.020 OOF in v004 gave +0.011 public, the expected direction for v005b is public growth; we do not promise the value (scale reference - about half of the OOF increase)).

**Not done in this iteration.** CatBoost Not Installed (Installation Requires Download) – LightGBM Remains. Dedicated binary segmentation network, context 384/512 (another process is still learning, OOF is not) and steaker – did not start: GPU and memory are busy learning another process. External data: inventory only (HLS Burn Scars, CaBuAr, CEMS Wildfire, EO4WildFires, FLOGA, SSL4EO-S12); download requires separate confirmation.

**Candidates.** `artifacts/submissions/candidates/submission_v005a_component.csv` (OOF 0.4129) and `submission_v005b_component_refiner2.csv` (OOF 0.4298, recommended)). AF in both bytes coincides with v004/v003.

## ITERATION 5: Context (Crowd Size) - Controlled Experiment

Hypothesis: 0↔1 Confusion – from a lack of spatial context, not from a lack of hard examples (hard-negative sampling closed in iteration) 4). Only the crop size changes; Everything else is like v003-models 1 (B12 + landcover5 + NDVI, OHEM 0.4, ignore SCL 0/1, brightness p=0.3, 30 epoch × 2 crop/chip, same single split 179/45Same evaluator, mix 50/30/20). 384 and 512 trained with batch 4 × accumulation 2 (effective batch) 8like 256; Because of memory, BatchNorm can see by 4 sample 8It's a well-known difference. 256 repeated with the current code and reproduced the previous result to the last digit (determinism confirmed)).

| context | BSScore strict (cal) | BSScore all-pixel (cal) | IoU_burn (all) | IoU1 / IoU2 / IoU3 (all) | mIoU_sev | 0→1 FP px (all) | cropland 0→1 (px, rate) | grass 0→1 (px, rate) | 1→0 FN px |
|---|---|---|---|---|---|---|---|---|---|
| 256 (control = v003 recipe) | 0.3692 | 0.3495 | 0.556 | 0.358 / 0.525 / 0.666 | 0.516 | 259 904 | 188 543 (3.89%) | 56 318 (1.54%) | 153 278 |
| 384 | 0.3707 | 0.3423 | 0.552 | 0.343 / 0.508 / 0.640 | 0.497 | 210 452 | 142 725 (2.95%) | 54 097 (1.48%) | 176 372 |
| 512 (full scene) | 0.3709 | 0.3470 | 0.560 | 0.352 / 0.515 / 0.642 | 0.503 | 190 259 | 115 614 (2.39%) | 57 831 (1.59%) | 182 536 |

Runtime: Measurements distorted - 384 and 512 trained in parallel with each other and third-party process (train 871 / 2969 / 3708 s)). Inferencing is always on the full 512 scene, so the cost is the same in construction (measured 104 / 179 / 97 ms / chip on MPS - scatter from competition for GPU, not from the model)). Raw (uncalibrated) all-pixel speeds: 0.346 / 0.328 / 0.324; calibration bias in 384/512 is much larger (0.4–0.8 vs. 0.2–0.4): a model with more context is more conservative in burn.

**Result: Context - Winner No.** Gate (BSScore ≥ +0.008or IoU1 prominently ↑ upon 0→1 FP ↓) not passed: all-pixel -0.007 (384) and -0.0025 (512), strict +0.002 (within noise), IoU1 Both are under control0.343 and 0.352 against 0.358).

But the mechanism is partially confirmed: a larger context **drastically reduces false positives on cropland** (−39% pixels, 3.89%→2.39percentage 512; general 0→1 FP −27%), as suggested by the hypothesis. It's growing at the same time 1→0 (+19% FN: The model just gets more careful and after optimal calibration, the pure BSScore does not change. That is, the context shifts the point on the curve precision/recall class 1, but does not push the curve itself. 3-fold confirmation and blend did not start (gate condition not met, item 7 of the assignment)).

Romina model: there is no checkpoint in the repository (`cosmohack_dataset (1).ipynb` contains only a learning log) - on the canonical evaluator was not evaluated.

Not checked (consciously, item 7 of the task - do not run zoo): logit-blend already trained ctx256 and ctx512 (their errors are complementary to FP/FN, blend does not require retraining), but it would have to be confirmed by 3-fold retraining (~1 h+)).

**NEURAL TRACK: frozen on v003 (public 0.72)). No candidate.**

## LEADERBOARD: v004 = 0.7406 → v005 = 0.81

v005 = v004 → Connection Component Classifier (rejection of areas outside the main chip fire) → pixel contour refiner v2 → organizer severity physics. OOF all-valid 0.3652 → 0.4298 (+0.065) moved to public as +0.069. Confirmed: component-level contour reasoning is a major breakthrough. v005 is frozen, as is v001–v004.

## POST-V005: CONTEXT + RED-EDGE (closed by timebox))

**Context (ctx512, single split, 45 val chips – diagnostics only)).** 58% of FP pixels v005 ctx512 considers background, but ctx512 loses 186k class 1 pixels that v005 finds. At the component level, ctx512's rejection rate separates false and real components of v005: AUC 0.84 (arable land), 0.93 (grass)). Oracle "remove the false component if ctx512 agrees" = +0.030; naive rules on the rejection ratio - from 0.013 to 0.095. Fold-safe 3-fold ctx512 was launched and stopped by timebox in the 1st era (`data/processed/bs_v006/ctx512cv_f*.log` logs); leakage-safe context.

**Red-edge B5/B6/B7.** Against a background-like hare, ΔB5 carries 3-4 times more information than dNBR (|AUC−0.5| inside dNBR bins 0.175 vs 0.047, but against residual FP v005 - weakly (0.04-0.06)). Controlled ablation (component classifier + refiner v2, fold-safe retrained, rule v005 0.3/0.8/0.2)):

| config | BS all-valid | folds | strict | IoU_burn | IoU1 | IoU2 | IoU3 | 0→1 FP | 1→0 FN | cropland FP | grass FP |
|---|---|---|---|---|---|---|---|---|---|---|---|
| A: v005 (retrained by the same code)) | 0.4292 | .4266/.4315/.4297 | 0.4395 | 0.670 | 0.505 | 0.667 | 0.773 | 0.0211 | 0.218 | 0.0358 | 0.0394 |
| C: + red-edge | 0.4355 | .4332/.4349/.4383 | 0.4468 | 0.681 | 0.519 | 0.672 | 0.777 | 0.0198 | 0.211 | 0.0354 | 0.0362 |

+0.006 (on all folds +0.003...+0.009) - below the threshold +0.01, **v006 not created**. Red-edge is the first candidate for the next iteration, context is the second (after fold-safe OOF)).

## SERVICE ADAPTER (v005, frozen)

`ml/src/kroma_ml/service.py`: `predict_af(chip_id, root=None)`, `predict_bs(chip_id, root=None)`, CLI `python -m kroma_ml.service {af|bs} CHIP_ID [--root DIR] [--out file.npz]`. The neural network stage BS is run by a separate process (`kroma_ml.bs_neural`): torch/MPS and LightGBM in one process on macOS give segfault. Verified: on AF_te_000001/000050 and BS_te_000001/000040 output coincides byte with submission_v005. Time: AF ≈ 0.4 s/chip, BS ≈ 4-7 s/chip on CPU/MPS (of which U-Net ≈ 1.7 s)). For train chips, the adapter returns the GT and error map (0 correct, 1 FP, 2 FN, 3 incorrect severity); train chips are included in full-train training, so the quality on them is optimistic.

## POST-V005 FAST CONTOUR (timebox 40 min) → v006 (conservative))

**Oracles on OOF v005 (all-valid 0.4298)).** Ideal solutions in the band ±1/±2/±3 px from the circuit: +0.035/+0.052/+0.067. Perfect recovery of holes and fumes near the accepted (≤ 2/5/10 px): +0.016/+0.023/+0.029. Ideal removal of remaining false components: +0.040. FN v005: 54% further than 10 px of any accepted burn (whole missed areas), 8.8% in holes; FP: 33% at a depth of 1-2 px from the circuit.

**Boundary micro-refiner** (LightGBM only in the band ±3 px, fold-safe, features refiner v2 + red-edge + circuit distance + local fraction of fumes 3/5/7 + probability component: best option (additions only, t = 0.6) 0.4346 vs. 0.4355 y C; replacement solutions in the band - from 0.003 to 0.009. The high oracle of the strip is not predicted by available signs - the accuracy of the markup perimeter. Rejected; class-1 recovery (a frequent case of additions at the circuit) - also without increment.

| experiment | BS all-valid | fold1 / fold2 / fold3 | strict | IoU_burn | IoU1 | IoU2 | IoU3 | 0→1 FP | 1→0 FN | cropland FP | grass FP | runtime |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| v005 | 0.4298 | .4267 / .4325 / .4303 | 0.4401 | 0.671 | 0.504 | 0.669 | 0.775 | 0.0211 | 0.218 | 0.036 | 0.039 | ≈4.1 s |
| + red-edge (C) = **v006** | **0.4355** | .4332 / .4349 / .4383 | **0.4468** | **0.681** | **0.519** | 0.672 | 0.777 | 0.0198 | 0.211 | 0.035 | 0.036 | +0.7 s |
| + boundary (add-only, t 0.6) | 0.4346 | .4329 / .4331 / .4379 | 0.4466 | 0.680 | 0.519 | 0.669 | 0.778 | 0.0200 | 0.208 | 0.037 | 0.037 | +~0.5 s |
| + boundary (replace, t 0.4) | 0.4321 | .4284 / .4359 / .4320 | 0.4432 | 0.676 | 0.509 | 0.666 | 0.776 | 0.0173 | 0.255 | 0.034 | 0.033 | +~0.5 s |

**v006** = v005 + red-edge features (ΔB5/ΔB6/ΔB7, ΔNDRE5/6/7, NDRE7 post, local averages, ring contrast components) in the component classifier and refiner v2, decision rule v005 (0.3/0.8/0.2) unchanged. OOF +0.0057 (all folds +0.002...+0.008) – between 0.005 and 0.010, so it is a conservative candidate with little expected gain. Test: v005 reproduced byte in the same run; shift relative to v005 – 3.0% deleted, 3.9% added (OOF: 4.5% / 4.1%); severity volumes almost unchanged. Two full runs give the same result (sha 4108db72...). `artifacts/submissions/submission_v006.csv`, validator PASS, 447 lines, RLE roundtrip, no intersection of classes, no NaN, AF byte = v005.

**Context ctx512 (not included)).** Reduces false stains on arable land by about 39% (0→1 tillage 3.89% → 2.39% against ctx256), but loses a weak fumes (1→0 FN 153k → 183k); standalone-soon hasn't grown. Fold-safe 3-fold OOF took ~2-2.5 hours and was stopped by timebox, so the context in the final candidate did not enter the box. Next step: fold-safe ctx512 OOF → signs of rejection in the component classifier (oracle on 45 chips +0.030, AUC rejecting false components 0.84 arable land / 0.93 grass).

At the time of the research run service reproduced v005. Final product integration connected v006 without retraining; full competition CSV played accurately (180 AF + 89 BS, 447 lines)). See `docs/ml_handoff.md`.


## Final product integration · 2026-09-19

CURRENT GOLD **v006**, public score **0.8176**; previous stable v005 **0.8127** (team-value). The historical records above preserve the state of the relevant experiment, including the early conclusion "v006 not created".

Production adapter uses red-edge artifacts and constant rules 0.3/0.8/0.2. Retraining in the integration-pass was not carried out. Full submission: 447 lines, exact match v006, no errors; 452.88 with 2 workers × 2 threads, neural batch 19.81 with. The indicator depends on the machine/parallel load.

Service TRAIN overlays and metrics are marked in-sample; prepared derived 3 BS+3 AF sets are included for reproducible demonstration. TEST geometry/dates not restored. See `docs/key_metrics.md`, `docs/model_card.md` and `docs/qa/final-integration.md`.

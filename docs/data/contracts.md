# Observations and events contracts

The `1.0` version captures the exchange format. Pydantic models are in `backend/app/schemas/observations.py`; ingestion outputs one `HotspotObservation` per JSONL string. These models do not mean that the event database and API are already implemented.

| Essence | Basic fields | Rule |
| --- | --- | --- |
| `Coordinates` | `latitude`, `longitude` | WGS84, degrees; longitude -180...180, latitude -90...90 |
| `HotspotObservation` | `observation_id`, `source`, `satellite`, `acquired_at`, `location`, `brightness_ti4_kelvin`, `brightness_ti5_kelvin`, `frp_mw`, `scan_km`, `track_km`, `source_confidence`, `daynight`, `source_version` | One source detection; time UTC; `observation_id` is determined from source, satellite, time and coordinates |
| `FireEvent` | `event_id`, `first_observed_at`, `last_observed_at`, `centroid`, `observation_ids`, `kroma_confidence`, `priority_score` | Future clustering result; `observation_ids` link event to initial detections |

`source_confidence` retains the category VIIRS `l`, `n` or `h`. This is a characteristic of NASA's original algorithm, not the probability of a fire from KROMA. `kroma_confidence` and `priority_score` are not yet calculated and may be `null`. `frp_mw` stands for radiation power in MW, not the area of the fire.

When you reload the same detection, your ID is saved. Changing the algorithm for generating an ID or fields requires a new version of the scheme and migration. Raw CSVs and normalized sets are in `data/`, while code is in `backend/` and `scripts/`.

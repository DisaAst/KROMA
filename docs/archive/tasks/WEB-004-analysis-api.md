# WEB-004. Unified REST API for territory analysis

Status: DONE (19.09.2026)). Priority: P0. Area: backend. Criteria: R1, R2, R3.

Dependencies: WEB-001, WEB-002, WEB-003. Unlocks the full integration of WEB-005-008.

## Purpose

For one space-time request to obtain an agreed result for the map, reference and unloading. REST should work without a browser and without manual actions on the server.

## What to do

1. Implement the `/api/v1/analysis/datasets` and `/api/v1/analysis` directory [contract](analysis-contract.md).
2. Delete general `AnalysisService`: normalization of the → query selection of data/estimates → filtering of → points intersection of zones → area/→ summary provenance/coverage/result_id.
3. Do not collect new help from existing randomly generated weekly stats. The only source of results is the processed set of zones.
4. Distinguish between the availability of hotspots and BS; correctly return a partial result with a warning.
5. Validate bbox, dates, dataset version and limits. Do not return all data with an unknown / ignored filter.
6. Make deterministic result ID and sort the results. Any cache is keyed to the full normalized request and processing version.
7. Add diagnostic logs: request/result ID, set version, duration, counts, error type. Do not log secrets and bulky source artifacts.
8. Describe queries/responses and errors in OpenAPI with suitable examples.
9. Save existing endpoints and modes; the new service uses a separate prepared repository.
10. Measure cold/repeated request on demo kit. Set limits/timeouts; optimize reading and cache if necessary before entering background jobs.

## Points of change

- New router in `backend/app/api/v1/` and connection to `router.py`.
- Service in `backend/app/services/`, diagrams in `backend/app/schemas/`, adapter from WEB-002.
- Geo-functions WEB-003, limit/data path settings in `backend/app/core/config.py`.
- Tests in `tests/backend/`.

## Acceptance criteria

- [ ] One GET with bbox and dates returns all the data a new screen needs.
- [ ] Changing the bbox/period does change the result when the sample changes.
- [ ] Dates have documented semantics; right time boundary processed correctly.
- [ ] For a single query, the map and summary refer to the same version/estimation.
- [ ] Failed input, unknown set, no files, empty result and lack of coverage are distinguishable.
- [ ] Restarting the backend does not change the result on an immutable set.
- [ ] Typical analysis fits into an internal reference point of 10 seconds on a fixed environment or documented reasonable deviation and correction plan. The requirement of the organizers is a finite time, not just 10 seconds.
- [ ] No external API key is required to analyze the local prepared set.

## Checks

Integration requests for all contract examples. Check for the same pre/post restart requests, two different sets, partial coverage of only one source, maximum allowed query, parallel queries and no data mixing between them. Repeated query does not have to have the same `generated_at`, but the numbers, geometries and `result_id` must match.

## Outside the task

File export is implemented in WEB-008 through this service. Do not add accounts, saved user jobs and run models on arbitrary data.

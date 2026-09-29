# WEB-010. Automatic and manual web script checks

Status: DONE for fixture; repeat to real/model_output. Priority: P1, develop tests with P0 tasks. Area: QA, frontend, backend, geo. Criteria: R1, R2, R3.

Dependencies: WEB-001-009; test scaffolds and fixtures can be created earlier. Final Closing After Integration.

## Purpose

Prove not only the opening of pages, but also the consistency of spatial query, geometries, areas, display and files. Existing API/format tests don't test the entire new scenario.

## Check kits

1. Small synthetic, hand-tested geometry/area: needed for precise unit/integration tests.
2. Prepared result of real scenes: necessary for reproducible acceptance and demonstration.
3. Error fixtures: Class intersection, unknown CRS, missing file, incorrect classes/dates.

Expected areas can not be obtained by the same test method and considered independent evidence: use analytical figures / pixel masks and a separate geo-calculation.

## Matrix of mandatory inspections

| The Domain | Script | Expected behaviour |
| --- | --- | --- |
| Request | One day, right boundary of UTC | Correct inclusion/exclusion of observations |
| Request | Incorrect date, NaN, inverted/zero bbox | 422, understandable UI error |
| Selection | Changing bbox or dates | All related ideas are changing, not just points |
| Geometry | Partial intersection and touch | Only the area inside the AOI; touching does not add area |
| Geometry | Holes, MultiPolygon, three classes | Correct rendering and area, no intersection of classes |
| Data data | Several evaluations of one gari | Selected rule applies, no doubling |
| Data data | No detections with valid coating | Zero, Status Empty, Correct Exports |
| Data data | No coverage/valid BS pixels | Explicit status, inaccessible null areas, not false zeros |
| Data data | There's AF, no BS; there's BS, no AF | Available part of the result is maintained with limitations |
| Versions | Same request after restart | Same numbers, geometry and result_id |
| Versions | Old expected_result_id | 409, no download of inappropriate file |
| Exports | GeoJSON/CSV/JSON | Parses, has attributes, matches the result |
| UI | Draft changed after calculation | Report/Export refers to the last request applied |
| UI | Responses to requests came in reverse order | The old answer doesn't wipe out the new one |
| UI | Direct link, reload, back/forward | Agreed request/result is restored |
| Availability | Keyboard, focus, fields, buttons | The main scenario is available without drawing a mouse |
| Reliability | Tails inaccessible | Reference and export work, the map explains the problem |
| Regression | Review, events, timeline, topics | The old functionality continues to work |

## What to do

- Geo Unit Tests in `tests/geo/`: Vectorization/Pruning, CRS, Area and Validity.
- Backend integration/contract tests in `tests/backend/`: query, errors, versions, export, cache/restart semantics.
- Frontend unit/component tests: form state, null/zero, formatting, request application, errors.
- End-to-end browser scenario: Selecting an example of → request → layers/→ help download → check files. Add a separate E2E runner/configuration if necessary; do not claim that such a runner already exists.
- Check both themes at 390/768/1280/1440 px, tooltip/popover at the edge, browser zoom and tab.
- Add new code checks to CI. Real heavy/limited data is not necessarily a network dependency for every CI run.
- Lock the manual protocol for a real set with versions, input parameters, expected numbers and total.

## Acceptance criteria

- [ ] Matrix covered with automation or explicitly described manual check; no passes without explanation.
- [ ] Map, API, CSV/JSON and GeoJSON were reconciled by one result_id.
- [ ] For geometry, there are independent expected values, not just snapshot current implementations.
- [ ] Lint/typecheck/unit/build, backend pytest/ruff and new tests pass.
- [ ] Real time measurement of typical analysis and export is recorded.
- [ ] Mistakes and limitations are not masquerading as successful results.

## Outside the task

Proof of ML quality on a private sample, load tests for industrial scale and performance scores of competitive inferencing. The speed of a web request is not equal to the speed of `submission.csv`.

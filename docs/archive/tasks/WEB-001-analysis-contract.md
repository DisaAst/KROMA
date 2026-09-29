# WEB-001. The contract of spatio-temporal analysis

Status: DONE (19.09.2026)). Priority: P0. Area: backend, frontend, data. Criteria: R1, R2, R3.

Dependency: none. Unblocking: WEB-002-009.

## Purpose and problem

Dates, region and bbox are used by different parts of the system. To fix one query and result model, according to which the map, reference, export and integration of the results of scene processing are implemented.

## What to do

1. Check and finalize [analysis-contract.md](analysis-contract.md). Consider it an initial proposal, not an existing implementation.
2. Describe Pydantic query schemas, kit directory, analysis result, provenance, coverage, warnings and errors. Agree with TypeScript-types; use the available method to prevent their divergence: generation or contract tests.
3. Record UTC, end date inclusion, temporal selection of hotspots and hares. Accepted v1-version for hares - date of post-fire shooting with an explicit explanation to the user.
4. Describe the rules for re-evaluation of one burn, Polygon/MultiPolygon and partial AOI intersection.
5. Record classes 1/2/3, units of measurement, `null` and zeros, accuracy and rounding rules.
6. Separate AF/thermotor coating and BS: the absence of one source should not nullify the other. Specify which areas are available for coverage calculation and which are unknown.
7. To fix the versioning of the set and algorithm, the composition of deterministic `result_id` and export behavior in case of discrepancy of the version.
8. Determine the configurable limits of bbox/period/volume output. Measured values to specify in WEB-004 and write in the documentation.
9. Add contract examples: normal response, partial coverage, blank observation, no coverage, no valid pixels, validation error.

## Artifacts and points of change

- Finalized `docs/tasks/analysis-contract.md`; when ported to `docs/data/`, leave the reference and do not create two divergent specifications.
- New circuits, such as `backend/app/schemas/analysis.py` and related types of `frontend/src/lib/api/`.
- JSON Fixtures of Contract Responses in Tests. Synthetic fixtures explicitly mark `synthetic_demo`.
- Do not break existing `FeatureCollection` and old APIs: Extension enter explicitly.

## Acceptance criteria

- [ ] For each field, the type, obligation, units, `null` admissibility and meaning are indicated.
- [ ] You can clearly understand why a particular point/gall enters or does not enter a given period.
- [ ] The query captures the version of the data; one version does not change in place.
- [ ] Values in the certificate are derived from the same areas that will be exported.
- [ ] Described the difference “not detected”, “not surveyed”, “closed by clouds”.
- [ ] All contract examples are validated by backend diagrams; frontend is able to read them.
- [ ] The new scheme does not pass off existing heuristic confidence/priority as model quality metrics.

## Checks

Check `from > to`, incorrect date, zero/inverted bbox, NaN/Infinity, coordinates outside WGS84, one calendar date, moment exactly on the right time limit, known and unknown class_id. Verify Equivalent Request Normalization and ID Stability.

## Outside the task

Real geocalculations, screen design, queue of background tasks, arbitrary polygon. Do not promise speed before measurement on data.

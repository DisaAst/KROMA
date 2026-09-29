# Archive: Plan to finalize the KROMA web under evaluation criteria

Audit Date: September 18, 2026. Status of the plan: agreed direction, implementation of tasks is not yet completed.

## Purpose

Bringing the existing web to a finished scenario:

**Select the territory and period of → get hotspots and contours of → hares study areas on three degrees of → damage download the contours and help → reproduce the result through the REST API.**

Save existing review, events, timeline, themes and top navigation. Do not rewrite the application and do not replace work with real results of processing with cosmetic changes.

## Basis and boundaries

The sources of the claims are in [description](../../description/):

- “Assessment Criteria – Natural Fire Monitoring Cosmo Hackathon 2026”, section 4, pages 5–6.
- “Case setting – Cosmo Hackathon Natural Fire Monitor 2026”, pages 14–15; limitations on the use of test data – page 14.

The web service section gives up to **12 points**, It's not all 100 points of competition:

| Code | Demand | Balls | Tasks |
| --- | --- | --- | --- |
| R1 | Instructional launch, space-time request, final response time, mandatory REST API | 0 or 4 | WEB-001, 002, 004, 005, 010, 011, 012 |
| R2 | Map of hotspots for the period and contours of hares; unloading GeoJSON or Shapefile; correct geometries, coordinate system, ID, class and area | 0–4 | WEB-002, 003, 004, 006, 008, 010, 012 |
| R3 | Territory and period reference: total area and three degrees of damage; machine-readable export; correct and reproducible areas | 0–4 | WEB-001, 002, 003, 004, 007, 008, 010, 012 |

This is a match matrix, not a promise of points: the final assessment is put out by the organizers.

A limited set of scenes is allowed. Therefore, the minimum version is used **bbox and GeoJson**; arbitrary polygon and Shapefile are not required. Prepared results of processing real scenes are permissible; synthetic demonstration figures do not confirm the quality of calculations from the source data.

Model training, AF/BS quality, `submission.csv` generation and competitive inferencing speed are outside this plan. The Web needs a contract for the results of these modules, but the implementation of WEB-tasks does not close the ML sections of the evaluation.

## Reference state

- There are Next.js/MapLibre, FastAPI, Docker Compose, CI and autodeploy.
- There are maps, hotspots, hare contours, event cards, analytics and before/after comparisons.
- There is no general recorded request for territory and dates for map, reference and export.
- The main API uses `DemoFireRepository`; weekly figures are generated regardless of the contours.
- `/analytics/summary` accepts dates and region, but not bbox. `/map/burn-scars` accepts bbox, but not dates and does not trim geometry by area.
- Damage zones in demonstration data nested into each other. The area of the zone attribute is not equal to the area of its full polygon.
- The Web does not provide the user with the unloading of selected contours and references.
- Global LIVE/Scenario Mode does not identify the source on all pages in the same way.
- A before/after comparison shows common substrates not tied to the dates of a particular pair of scenes.
- At the time of the audit LIVE on the server returned `configured=false`, `offline`. It is an observation at the date of the audit, not a permanent property of the system.

`docs/tasks/task-1` already had an empty user file. It's left unchanged; the task registry is this file.

## Roster of tasks

Engineering implementation completed on September 19, 2026. Statuses below share code readiness and final proof on real data: built-in `synthetic_demo` does not close the external dependency WEB-002/012.

| ID | The challenge | The Domain | Priority | Dependencies |
| --- | --- | --- | --- | --- |
| [WEB-001](WEB-001-analysis-contract.md) | Fix the contract, period and result semantics | API + data + frontend | P0 | No |
| [WEB-002](WEB-002-prepared-datasets.md) | Prepared scenes and reproducible source of results | Data + Integration | P0 | WEB-001; the results of processing real scenes |
| [WEB-003](WEB-003-geometries-and-areas.md) | Correct geometries, pruning and calculation of areas | Geo + backend | P0 | WEB-001; test data WEB-002 |
| [WEB-004](WEB-004-analysis-api.md) | Unified REST API for territory analysis | Backend | P0 | WEB-001, 002, 003 |
| [WEB-005](WEB-005-analysis-query-ui.md) | Selection of set, territory and dates in UI | Frontend | P0 | WEB-001; integration with WEB-004 |
| [WEB-006](WEB-006-analysis-map.md) | Map of the results of a single request | Frontend | P0 | WEB-004, 005 |
| [WEB-007](WEB-007-analysis-report.md) | Certificate of three degrees of defeat | Frontend | P0 | WEB-004, 005 |
| [WEB-008](WEB-008-exports.md) | Exports of GeoJSON, CSV and JSON | Backend + frontend | P0 | WEB-003, 004, 007 |
| [WEB-009](WEB-009-data-provenance.md) | Honest modes, data origins and snapshots | Frontend + API | P1 | WEB-001, 002, 004 |
| [WEB-010](WEB-010-verification.md) | Verification of geometries, contracts and cross-cutting scenarios | QA + development | P1 | Written together with WEB-002-009; closed after them |
| [WEB-011](WEB-011-reproducible-delivery.md) | Replayable launch and demonstration kit | Infra + documentation | P1 | WEB-002, 004, 008, 010 |
| [WEB-012](WEB-012-final-acceptance.md) | Reception on R1-R3 and rehearsal for protection | QA + team | P1 | WEB-001–011 |

### De facto status

| ID | Status | Confirmation/residual |
| --- | --- | --- |
| WEB-001 | DONE | Pydantic + TypeScript + OpenAPI + Contract Tests |
| WEB-002 | PARTIAL: external data | Manifest/validator/adapter/fixture ready; `model_output` allowed |
| WEB-003 | DONE | Vectorization, valid geometry, holes/MultiPolygon, clip, EPSG:6933 area, overlap tests |
| WEB-004 | DONE | Catalog, readiness, single REST, source coverage, stable result ID, limits/logical errors |
| WEB-005 | DONE | Dataset/bbox/dates, keyboard inputs, map bounds, draft/applied, URL/reload/history semantics |
| WEB-006 | DONE | AOI, hotspots, 3rd class, legend, popups, visibility, fit, tile-error isolation |
| WEB-007 | DONE | Total + three degrees, shares, null/zero, coverage/provenance/calculation |
| WEB-008 | DONE | GeoJSON + CSV + JSON, expected result 409, headers, UI downloads |
| WEB-009 | DONE | Origin: UI/API/export; unsupported LIVE sections and imagery are honestly labelled |
| WEB-010 | DONE for fixture | 44 pytest + 22 frontend tests + lint/typecheck/build + browser protocol |
| WEB-011 | PARTIAL: clean-room witness | Compose/config/docs/mount/CLI ready; need to be run by another person in a clean directory |
| WEB-012 | BLOCKED by WEB-002 | Synthetic rehearsal recorded; real-scene acceptance remains |

Actual requests, numbers, times and restrictions are recorded in [web-acceptance.md](../../qa/web-acceptance.md).

## Implementation arrangements

1. **Contract and data:** WEB-001; then WEB-002 and geology WEB-003. The interface WEB-005 can be developed using clearly marked contract fixtures.
2. **Working request:** WEB-004 + WEB-005. The result is obtained through the API and from the interface.
3. **Full user result:** WEB-006 + WEB-007 + WEB-008. Map, Help and Files refer to a single request.
4. **Credibility and delivery:** WEB-009, completion of WEB-010, WEB-011, WEB-012.

Tests should be added to each task, rather than postponed until the fourth stage. Fixture UI does not mean complete integration. If there are no real results yet, the WEB-002 task remains blocked from the data, and the status of the demonstration is clearly indicated.

## General rules of implementation

- [analysis-contract.md](analysis-contract.md) — Implemented and covered contract of new functionality; its Pydantic and TypeScript representations, OpenAPI and tests must change synchronously.
- Web and REST use the same server logic. Do not count the total area independently in React, Exporter and API.
- The draft form is separated from the completed request. Exports refer to the last result shown, not to unsaved fields.
- The result has a set version, processing version and content identifier. Changing the data should not change the old result.
- All common interface elements are included in `frontend/src/components/ui`. Specific logic of analysis - in a separate feature, not in UI-primitives.
- New functionality to place inside `/analytics`, for example, tab "Analysis of the territory". The existing summary can be left with a separate tab with an honest marking of the source.
- Save light and dark themes, top navigation, map operation and existing routes.
- Do not use the coordinates/dates of impersonal test chips or attempt to restore them. Do not replace model responses with ready-made external test products.
- Do not publish secrets or limited source data. To fix an acceptable method of transferring a demonstration set to the organizers.

## General Definition of Done

- Acceptance criteria for a specific task are met; there are no unsigned stubs instead of the result.
- There are relevant tests and updated documentation.
- Modified front-end modules pass lint, typecheck, unit tests and production build; backend/geo – corresponding to pytest and ruff.
- UI tested in both themes, with keyboard, desktop and narrow screen.
- The states of loading, error, empty result and incomplete coverage are not masked by zeros.
- Old functionality is not broken; new sources are not mixed with demodata.
- R1-R3 points are not considered closed only on the presence of buttons: the script WEB-012 is checked on the prepared real results.

## What's postponed

After the mandatory volume, you can add: arbitrary polygon and import GeoJSON, Shapefile, PDF help, saved requests, personal accounts and roles, notifications, extended LIVE, distribution forecast and a large collection of scenes. This does not depend on the minimum closure of R1-R3.

For a limited set, PostGIS, task queue and permanent storage of analysis results are not required. Do not enter them just for the sake of architectural complexity: first measure the volume and time of response.

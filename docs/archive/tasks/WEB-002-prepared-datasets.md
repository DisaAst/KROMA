# WEB-002. Prepared scenes and source of results

Status: PARTIAL – infrastructure ready, real/model_output allowed. Priority: P0. Area: Data integration, backend. Criteria: R1, R2, R3.

Dependencies: WEB-001; Available results of processing real geo-referenced scenes. Unlock: WEB-003, 004, 009, 011.

## Purpose and problem

Replace synthetic results in the new analysis with a reproducible prepared set. Full satellite ingestion is not required. The main source now is `backend/app/repositories/provider.py`, which returns `DemoFireRepository`; it cannot be imperceptibly passed off as a real calculation.

## Entrance from data command/ML

- Geo-related observations of active combustion with time and source.
- BS results with classes 0–3, nodata/validity mask, source CRS and transform.
- Real Scene IDs, Before/After Shooting Time, Spatial Footprint and Known Share of Cloud.
- Identifier/processing version, evaluation link to the garland and a pair of scenes.
- Information on the license and permissible distribution of artifacts.

You can not restore the geo-reference of impersonal test chips. Use permitted geo-referenced data, such as an accessible learning part or independent historical scenes. Reference masks are acceptable as signed test materials, but not as model predictions.

## What to do

1. Prepare at least one fully traceable real-world example, preferably 2-3 scenarios: gar with several classes, partial intersection of the area, no detections / lack of coverage. This is the recommended demo volume, not the requirement of the organizers to the number of scenes.
2. Create versioned manifest at WEB-001: paths, checksum, coverage, dates, origin, processing versions, scene IDs, classes, CRS, resolution/transform, validity and licenses.
3. Make a set check/prepare command without manual Python editing. Check the availability of files, checksums, georeference, classes and metadata.
4. Add read-only repository/adapter of prepared results; do not delete demo mode and LIVE.
5. Download a set from a configurable path/volume. Do not get tied to the absolute paths of the developer machine or the current date.
6. Provide stable IDs, no duplicates and deterministic issuance.
7. Publish a catalog of available UI/API kits, including coverage limitations.
8. Separate small synthetic fixtures for CI and a demonstration set of model results. Secrets and Heavy/Limited Sources Do Not Commit.

## The result

- Manifest and instructions for obtaining/connecting artifacts.
- Adapter/repository, set validator, small test fixtures.
- Famous example of bbox + dates + expected results for WEB-010/012.
- Explicit list of available and missing inputs. If there are no outputs of the model, the task remains unfinished in terms of the real demonstration.

## Acceptance criteria

- [ ] After restarting, the same set of observations, zones and metadata are obtained.
- [ ] Each result can be determined by the source and version of the processing.
- [ ] The set is independent of the availability of FIRMS at the time of demonstration.
- [ ] Error/missing artifacts are detected before plausible numbers are given.
- [ ] Dates, scene IDs and coverage are taken from data, not generated for card processing.
- [ ] The results of real scenes are not mixed with `DemoFireRepository`.
- [ ] All licenses/restrictions and the method of transfer to the organizers are described.

## Checks

Repeated preparation gives the same checksums. Deleted file, incorrect hash, invalid class, missing CRS, ID duplicates and conflicting dates lead to diagnosis. A limited set can be connected in a clean environment according to the instructions.

## Outside the task

Model training, downloading arbitrary new scenes at the user's request, country-wide processing infrastructure and mandatory PostGIS integration.

# WEB-008. Unloading of contours and machine-readable reference

Status: DONE (19.09.2026)). Priority: P0. Area: backend + frontend. Criteria: R2, R3; supports R1.

Dependencies: WEB-003, WEB-004, WEB-007.

## Purpose

Give the user suitable for GIS and further processing files exactly the result that he sees. Having an internal GeoJSON endpoint without conveniently exporting the selected request is not enough for a finished scenario.

## What to do at the backend

1. Implement endpoints of exports from [contract](analysis-contract.md), Using a common AnalysisService without a separate filtering/calculation algorithm.
2. Get the full parameters of the applied query and `expected_result_id`. If there is a mismatch, return the 409 with the instructions to recalculate, do not download other data.
3. GeoJSON: FeatureCollection zones with Polygon/MultiPolygon, WGS84 longitude/latitude, unique ID, class, intersection area and origin.
4. Do not include the full contour and its zones as equal areas of one export layer. Zones can be grouped by `burn_event_id`.
5. CSV/JSON: total and three classes, counts, parameters, source/version, coverage/warnings, CRS/method and result ID.
6. Provide correct Content-Type, Content-Disposition and secure file name. If necessary, open Content Disposition via configured CORS.
7. Valid empty result unload as empty FeatureCollection and status certificate. In the absence of coverage, do not replace `null` with zero.
8. Eliminate CSV injections in text values, if they can get user / external input; correctly shield commas, quotation marks, transfers and Cyrillic.

## What to do on the frontend

- Two main buttons: "Download contours · GeoJSON" and "Download help · CSV". A JSON Help Can Be an Additional Action.
- Transmit the parameters of the shown result, and not the draft form / current camera.
- Show the preparation/error of the file, allow you to repeat the action.
- Do not save JSON errors as a successful `.geojson`/`.csv` file.
- Correctly free object URLs if Blob is used; do not embed large payloads in the URL.

## Acceptance criteria

- [ ] GeoJSON opens with an independent parser and in a GIS/viewer with the correct location.
- [ ] All zones have a unique ID, class_id 1/2/3, severity and area_ha.
- [ ] Geometry is trimmed by AOI; recounting is consistent with attributes in the established tolerance.
- [ ] The sum of zones / classes is consistent with the certificate and map.
- [ ] CSV reads standard parser, JSON matches circuit, Cyrillic is saved.
- [ ] After changing the form draft, the old shown result is downloaded until the new calculation is applied.
- [ ] You can download the same files directly via REST without UI.
- [ ] Reuploading a fixed result reproduces meaningful data; allowable variable timestamps are described.

## Checks

Integration tests of all MIME/headers, content, filenames, empty results and 409. Error check 422/404/503, expired/deleted set, repeated clicks. Independent geo-calculation, CSV round-trip and reconciliation of three forms of representation of the result.

## Outside the task

Shapefile and PDF: GeoJSON and CSV/JSON are sufficient for minimum closure. Do not make a separate reporting system and store user files without the need.

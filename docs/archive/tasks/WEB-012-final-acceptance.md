# WEB-012. Final acceptance by criteria and demonstration

Status: BLOCKED – expected real/model_output at WEB-002. Priority: P1, the final task. Area: QA and team. Criteria: R1, R2, R3.

Dependencies: WEB-001-011. Do not close only on successful assembly or screenshots of the UI.

## Purpose

Confirm a completed web scenario and prepare a demonstration in which an expert can trace the path from territory/dates to verifiable files and numbers.

## Preconditions

- Demonstrate, dataset/version, processing version, manifest checksum and environment.
- Prepared bbox/dates for meaningful result, partial intersection and empty/uncovered case.
- There are expected numbers and the origin of the data; the results of the model, benchmark and demo are distinguishable.
- On the demo machine, a set and start instruction are available; the basic APIs are independent of external keys.

## Scenario of the demonstration

1. Run the service according to the instructions or show a pre-launched instance and an exact playback command.
2. Open "Analysis of the territory", name the source and boundaries of the prepared set.
3. Select the area and dates, perform the calculation without manual intervention on the server.
4. Show AOI, hotspots and zones of three classes on the map, legend and zone card.
5. Show reference: total area, three classes, shares, source, period and restrictions.
6. Download GeoJSON and CSV; open files, show ID/class_id/area_ha and metadata.
7. Repeat the request via cURL/OpenAPI without a browser interface, verify result_id and numbers.
8. Reduce the area so that it crosses the fumes, count and show the change of areas / contours.
9. Show a valid empty area and an uncoated area; explain the difference between “0” and “no data”.
10. Repeat the original request and confirm reproducibility. Separately perform checks after restart.

Make the demonstration short and coherent; agree on the specific duration with the actual protection regulations, do not invent it.

## Matrix of evidence

| Criteria | What we show | What we keep in the record |
| --- | --- | --- |
| R1: performance | Launch, bbox+dates, finite time response, REST without UI | Command, commit, versions, request, time, HTTP result |
| R2: map and contours | Period Points, Burns, Classes and Download GeoJSON | File, CRS, ID/class/area, validation/trimming |
| R3: reference | Summary and three classes, CSV/JSON, matching geometries | Files, area method, check amounts, dataset/result ID |

## Final checklist

- [ ] R1: instruction and REST work; request does not require manual data/code correction.
- [ ] R2: objects correspond to AOI/period; zones are valid, do not duplicate and have mandatory attributes.
- [ ] R3: Areas are calculated from results, not generated; three classes are present; computer-readable help is available.
- [ ] UI, API and files are the same in content data and result_id.
- [ ] Demonstrated the difference between absence of detection, lack of coverage and invalid pixels.
- [ ] Origin and the reality of images / results do not cause false expectations.
- [ ] The new version passed CI and manual checks of both themes/main desktop and mobile scripts.
- [ ] The restart does not change the fixed set data.
- [ ] There is a list of remaining limitations with an honest description, not hidden defects.
- [ ] We understand that closing this task confirms the readiness of the web section, but does not guarantee jury evaluation and does not close ML/submission.

## Exit artifact

An engineering verification protocol is created in `docs/qa/web-acceptance.md`: date verified, environment, commit, data versions, queries, times, expected/actual values, automatic test results and a list of restrictions. After obtaining an authorized `model_output` and an independent clean-room witness, it must be supplemented with final evidence. Screenshots are an addition, not the only proof.

In case of defect, make a specific correction with a link to the WEB-task. Do not mark the entire web complete if there is only synthetic_demo or space and export diverge.

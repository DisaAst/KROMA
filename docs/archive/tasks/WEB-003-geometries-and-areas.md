# WEB-003. Zone geometry and reproducible area calculation

Status: DONE (19.09.2026)). Priority: P0. Area: geo, backend. Criteria: R2, R3.

Dependencies: WEB-001; fixtures / metadata WEB-002. Real data needed for final acceptance.

## Purpose and confirmed problem

Class zones must be non-overlapping, and the `area_ha` attribute must correspond to geometry. Now `backend/app/demo/dataset.py` creates nested filled landfills. At the time of the audit, `BS-2026-064` had the following values::

| Class class | Attribute, ha | Recalculation of the geometry of the existing geoutility, ha |
| --- | --- | --- |
| low | 122 | about 814 |
| moderate | 391 | about 660 |
| high | 301 | about 301 |

New exports should not inherit such a structure. The full contour and zones cannot be stacked together.

## What to do

1. Implement in `geo/src/kroma_geo/` isolated functions of vectorization/validation, reprojection, intersection with AOI, measurement and verification of overlaps.
2. For model results, get zones from mask classes, save holes, individual islands and nodata. Do not create zones by artificially scaling the general circuit.
3. Support Polygon/MultiPolygon in new domain/output schemes. Reconcile the closed rings and the orientation of the output GeoJSON.
4. Apply the rule for selecting one estimate per bag prior to aggregation. Coverages of different hares / scores should not silently double the area: correct the origin / identification or reject a contradictory set with diagnosis.
5. Cut off areas by area of request. In the bbox at the edge of the burn, only intersection is considered; touching with a point/line does not add area and new fumes.
6. Select the correct CRS calculation for the prepared set, save it in metadata. Do not measure hectare in degrees or Web Mercator.
7. Count the area after pruning. Maintain sufficient accuracy, round only at the output boundary.
8. Calculate the total area, three classes, shares and number of unique hares from one set of zones.
9. Fix the policy of invalid geometry: a valid correction is checked for change in area; a significant error is not corrected silently. Write down a warning or reject a set.
10. If old demo zones are available for download/new help, correct their topology with a separate explicit transformation and leave the origin of `synthetic_demo`. Do not confuse this with vectorization the results of the model.

## The result

Net testable geo-functions, schema adaptation, calculation method documentation and set of reference geometric tests. Older rendering layers check after MultiPolygon appears.

## Acceptance criteria

- [ ] Geometry valid, located within the selected area, have correct rings/holes.
- [ ] Classes do not overlap in positive area; repeated grades do not increase the result.
- [ ] `area_ha` refers specifically to the given geometry, not to the original integer.
- [ ] The sum of the area of the zones by class coincides with the certificate; the sum of three classes - with the total area.
- [ ] Area matched to original mask and georeference; resolution/nodata not ignored.
- [ ] Method, CRS, pruning rules, deduplication and error documented.

## Checks

- Metric mask 20 × 20 m: one valid burnt pixel - 0.04 ha, half geometric intersection - 0.02 ha.
- Polygon with hole; two unrelated parts; bbox completely inside/out/on the border; MultiPolygon.
- Re-evaluation of the same burn, same zone twice, intersecting classes, self-intersection.
- Verification after converting back to WGS84 and re-reading export GeoJSON.
- For sums within one calculation, use a strict machine error. For independent recalculation of geometry, the `max(0,01 га, 0,000001 × площадь)` tolerance reference point; clarify by selected projections/accuracy and justify, not mask systematic errors.

## Outside the task

Improving the accuracy of the segmentation model, generalizing the contours for a beautiful display with loss of accuracy, automatically correcting any geometry without checking.

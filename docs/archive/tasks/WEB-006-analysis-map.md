# WEB-006. Map of analysis results

Status: DONE (19.09.2026)). Priority: P0. Area: frontend/MapLibre. Criterion: R2.

Dependencies: WEB-004, WEB-005; geometry WEB-003.

## Purpose

On one map, show the selected area, hotspots and non-overlapping zones of three degrees of damage from one result. Do not mix them with arbitrary LIVE/demo layers.

## What to do

1. Connect the analysis result as separate GeoJSON sources/layers; do not take data from multiple old endpoints with different filters.
2. Show the boundary of the fixed AOI. If necessary, separately designate the rough area so that it cannot be confused with the calculated.
3. Display hotspots, contours of hares and zones of classes 1/2/3. Support Polygon/MultiPolygon and artifact-free holes.
4. Add point/gare visibility switches, a legend with Russian names "Weak", "Mediocre", "Severe degree of defeat". Color is not the only way to know a class.
5. By clicking on the zone, show ID, class, area exactly inside the AOI, date of post-fire shooting, source / version. By point – ID, time, source and only available baselines.
6. Add "Show the whole area". Do not reconfigure the camera with every background refetch or topic change.
7. Link the subheading of the map to the applied dates/set, not to the draft form.
8. Show a partial coating warning; if coverage geometry is available add its contour/mask. Do not paint an unexplored area as the absence of fires.
9. Handle substrate failures separately from data errors. Help and export remain available if external tiles are not loaded; provide a readable neutral background.
10. Reuse existing base layers and topics; isolate analytical data from operational layers if their existing lifecycle/timeline is not appropriate.

## Acceptance criteria

- [ ] All objects shown belong to the current `result_id`; there is no data from the old request after switching.
- [ ] All classes are distinguishable in two themes, the legend coincides with the map and reference.
- [ ] Polygon with hole and MultiPolygon are displayed correctly.
- [ ] Pop-up cards are not cut off by the edge of the panel / screen and closed with the Escape key.
- [ ] The amount / area in the card is not recalculated on the screen: the server value is used.
- [ ] For zero result, the map saves the AOI and explains the absence of objects.
- [ ] Switching visibility / scale does not change the results of reference and export.
- [ ] An external substrate error does not become an analysis error.

## Checks

Control set with three classes, empty result, point on the border, narrow polygon, hole, several islands, partial coverage, change of request and theme, reload style, inaccessible tiles and narrow screen.

## Outside the task

New predictive layers, real-time satellite download, manual model editing and mandatory split-screen images.

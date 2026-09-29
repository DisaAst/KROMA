# WEB-009. Data sources, modes and reliable “before/after”

Status: DONE (19.09.2026)). Priority: P1, mandatory before protection. Area: frontend + API. Criteria: Reliability of R1–R3 demonstration.

Dependencies: WEB-001, WEB-002, WEB-004.

## Purpose and problem

The user must understand whether he or she is seeing real-world current observations, prepared model results, or a synthetic scenario. Now the LIVE mode changes the map, but the event table and analytics continue to refer to the demo repository. The “before/after” pair uses shared satellite substrates instead of scenes with dates from the card.

## What to do

1. Inventory of sources on `/`, `/events`, `/analytics`, in search, lists of regions, inspector and comparison of images.
2. On each screen, clearly indicate origin: “Actual NASA FIRMS observations”, “Prepared processing results”, “Demonstration scenario” or “Reference data”.
3. In the analysis of the territory, the source is controlled by the selected dataset/version. Do not allow a global LIVE switch to visually promise to change the help source if it is not implemented. The minimum option is to hide/limit this switch on a new screen and sign the source.
4. For existing pages, either connect the source corresponding to the selected mode, or locally show that the partition is only available as a script. Don’t duplicate whole unsupported LIVE analytics for the sake of a formal uniform menu.
5. If FIRMS is not configured or outdated, show offline/stale, time of last successful update and understandable explanation. Do not replace real observations with synthetic ones unnoticed.
6. In the before/after comparison, connect real images of the corresponding pair of scenes, if artifacts are present. Use their bounds/geo-reference, times, cloudiness and attribution.
7. If there are no pairs of images, do not show the common substrate under the guise of snapshots of given dates: hide the comparison or call it “Substrate and classification result”, remove unconfirmed dates / satellites.
8. Label synthetic forecasts, threats and other demonstration assessments. Do not pass off heuristic reliability as an experimental model metric.
9. Save source information in export metadata and reference, not just decorative badge.

## Points of change

`components/shell/ModeSwitch.tsx`, `StatusIndicator.tsx`, `features/events/EventsTable.tsx`, `features/analytics/AnalyticsView.tsx`, `BurnScarCompare.tsx`, `CompareMap.tsx`, `config/map.ts`, new feature analysis and provenance schemes.

## Acceptance criteria

- [ ] On each screen, the source is visible and corresponds to the actual API being called.
- [ ] You cannot accept demo/reference for actual observations or model output.
- [ ] Disabled LIVE does not interfere with the analysis of the prepared set and is not replaced by a demo without warning.
- [ ] Before/after images confirmed by scene IDs/dates or comparison honestly renamed/disabled.
- [ ] The same origin is displayed on the screen, in APIs and files.
- [ ] Not lost attribution of used substrates and images.

## Checks

Switching modes on all pages; LIVE without a key, stale and successful LIVE with a test source; prepared set without raster previews; reference/synthetic origins; click on a direct link; reload; change the topic.

## Outside the task

Mandatory configuration of external FIRMS key, development of satellite archive, mandatory comparison of images in the absence of input artifacts. The real “before/after” is a useful demonstration, but not a separate requirement R1–R3.

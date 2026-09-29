# WEB-007. Information on the selected territory and period

Status: DONE (19.09.2026)). Priority: P0. Area: frontend. Criterion: R3.

Dependencies: WEB-004, WEB-005. Used in conjunction with WEB-006/008.

## Purpose and problem

Show a reproducible reference for a specific query, rather than general demonstration statistics. There is now a total/severely damaged area in the summary, but there is no complete territorial report for the three classes.

## Contents of the certificate

- Applied area: coordinates / name of the set, available coverage.
- Period, UTC and the rule for selecting hares by date of post-fire shooting.
- Total burned area, number of unique hares, thermal points.
- Three separate lines: weak/medium/severe lesion - area in ha and proportion of total burned-out area.
- Compact distribution diagram with text values; table remains the main available form.
- Set, processing version, scene IDs/sources, `result_id` and formation time.
- Restrictions: Incomplete coverage, invalid/cloud pixels, lack of any source.
- Disclosed block "How calculated": area method, CRS, AOI trimming and elimination of duplicates.
- Export buttons WEB-008.

## What to do

1. Create a help component inside feature analysis. Reuse `MetricCard`, `Card`, `StateMessage` and common elements.
2. Receive values from the result WEB-004. Do not add up rounded signatures or use old weekly stats.
3. Show all three classes, even when one has zero. Clarify understandable Russian names without mixing the severity of fire and the severity of vegetation damage.
4. Separately show the area of the selected / surveyed / invalid territory, when it is known. Do not mix it with a burnt out area.
5. Distinguish zero, `null`, lack of coverage and load. For total area 0 do not show NaN/Infinity or artificial 100%.
6. Agree rounding, units and explanation of possible differences in the sums of the displayed rounded values. CSV/JSON maintains original accuracy.
7. When changing the draft, save the signature of the applied request: "Parameters changed - perform the calculation".
8. In an empty/partial result, explain what exactly was examined and what conclusions cannot be drawn.

## Acceptance criteria

- [ ] There is a result and all three classes with areas and shares.
- [ ] The numbers refer to the same `result_id` as the map and downloaded files.
- [ ] A class with 0 ha differs from an unavailable value.
- [ ] The user sees on which territory, dates and source the reference is made.
- [ ] Incomplete coverage is not given for the total absence of fires.
- [ ] Texts and table are readable in both themes, accessible from keyboard/screen reader.
- [ ] The output contains exactly observable/calculated values, without unconfirmed damage estimates or model accuracy.

## Checks

All classes are present; one class is absent; area 0; very small area; large values; partial/unknown validity; only thermal points without BS; BS without hotspots; shape change after successful calculation; number comparison with API and export.

## Outside the task

PDF, reforestation recommendations, financial assessment of damage, seasonal analytics across the country and independent calculation in the browser.

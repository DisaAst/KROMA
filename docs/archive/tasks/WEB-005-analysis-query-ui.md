# WEB-005. Selection of set, territory and period

Status: DONE (19.09.2026)). Priority: P0. Area: frontend. Criterion: R1.

Dependencies: WEB-001; signed fixtures are allowed for development, final integration is WEB-004.

## Purpose

Add a comprehensible input to space-time analysis inside `/analytics`, keeping the existing summary. The user must understand what will be calculated without knowing the API.

## User scripting

1. Open the tab "Analysis of the territory".
2. Select a prepared set; see its source, available dates and coverage.
3. Set the territory with a rectangle on the map, coordinates or button "Take the visible area".
4. Select starting and ending dates.
5. Click "Calculate" and get the result of one recorded request.

## What to do

- Create feature analysis, such as `frontend/src/features/analysis/`: form/query state, query client and screen container. Do not place the entire form in the total `components/ui`.
- Reuse/extend `Field`, `Card`, `PageHeader`, `StateMessage`, buttons and MetricCard. Common new elements - in `components/ui`.
- Separate `draftQuery` and `appliedQuery/result`: entry of new dates and movement of the map should not change the old result or parameters of its export.
- Give the "Use Example" button from the directory so that the user can immediately open the guaranteed supported area / period.
- Implement allocation/change/reset bbox with visual boundary. The allocation tool does not conflict with the movement/measurement of the card.
- Provide numerical coordinate input as an affordable alternative to drawing. Sign the order "West, South, East, North", WGS84.
- Validate dates and bbox on the client, the server remains the source of truth. Warn about going beyond the coverage, do not issue a partial area for a full.
- Show calendar dates and explanation of UTC; for hares, the selection rule for post-fire imagery.
- Save the applied query in the URL with the set version so that the link reproduces the analysis. The draft should not discreetly rewrite it.
- With a new request, cancel an unnecessary fetch or ignore an outdated response. You can leave the old result visible, but sign that the new calculation is still being performed.
- Conditions: source, edit, download, success, empty, no coverage/valid data, partial coverage, error + repetition.

## Acceptance criteria

- [ ] The user can execute the request without manually changing the URL/API and without knowing the coordinates.
- [ ] The rectangle can be set with the mouse and through fields from the keyboard.
- [ ] Incorrect parameters are not sent without explanation; errors are tied to fields and read by the screen reader.
- [ ] The start button reflects the boot status and prevents accidental repeated clicks.
- [ ] When the camera moves, the result and export do not change until a new launch.
- [ ] URL saves the applied query; reload and browser back/forward do not show non-conforming data.
- [ ] The lack of available sets has an understandable message, not an empty select.
- [ ] The screen works in light/dark theme and at widths of 390, 768, 1280, 1440 px without loss of basic actions.

## Checks

Example request, period change only, bbox change only, draw cancellation, area reset, incorrect date, reachout, quick reverse order request switching, keyboard navigation and page update.

## Outside the task

Free landfill, file loading, accounts and stored requests on the server. The results map and reference are implemented WEB-006/007.

# WEB-011. Reproduced launch and delivery of a web service

PARTIAL - Delivery is ready, clean-room witness required. Priority: P1, mandatory before protection. Area: Infrastructure and documentation. Criterion: R1; support for R2/R3 and reproducibility.

Dependencies: WEB-002, WEB-004, WEB-008, WEB-010.

## Purpose

Give the organizers the opportunity to run the service by README without code editing, connect a fixed set and repeat the request. Working team server and autodeploy do not replace the reproducibility of delivery.

## What to do

1. Update README: environment requirements, Docker/Compose, environment variables, ports, artifact preparation, launch, stop, logs and diagnostics.
2. Provide one main start command after documented data preparation. Prefer existing Compose; separate prepared/demonstration profile is acceptable if it simplifies playback.
3. In `.env.example`, describe the dialing path and the necessary parameters without secrets. Demo of prepared scenes should not require FIRMS/API keys.
4. Capture dependencies and versions of images/sets to the extent necessary to re-run. Do not download the “last” untested artifact.
5. Check manifest/checksums when preparing or starting. Separate process vivacity and analysis source readiness: `/health` alone does not prove data availability.
6. Safeguard/reconnect artifacts when restarting and changing release; do not store a single copy inside a temporary container.
7. Add an accurate cURL example of the catalog, analysis and each export with real dataset/version/bbox/dates from the prepared set.
8. README explains no coverage/offline statuses, data sources, area method, time semantics and known limitations.
9. Prepare a secure way to transfer limited data to the organizers. Do not include keys, passwords and forbidden scenes in the public repository / image.
10. Run in a clean directory/environment without local developer-paths and hidden mounts.
11. Check new functionality after deploitation on an existing server without affecting third-party services. Deploy only in a separate transition to implementation / publication, not within the framework of creating this plan.

## Failure checks

No data, wrong path, checksum mismatch, busy port, backend unavailable, external tiles not loaded. Each case corresponds to a diagnosable state/instruction; the launch does not hang indefinitely without a message.

## Acceptance criteria

- [ ] A person who did not develop the code, launched a service on README without manual editing of the source.
- [ ] The prepared analysis works without an external FIRMS key and without downloading new scenes during the request.
- [ ] Requests from README are indeed executed, export files are valid.
- [ ] Restart and re-install retain availability of specified kit version and results.
- [ ] External substrate restrictions are specified; its failure does not block API/reference/export.
- [ ] Versions of the environment, set, verified commit and smoke test results are recorded.
- [ ] Documentation does not promise full LIVE/ML/Permanent Storage unless implemented.

## Outside the task

Unnecessarily changing SSH accesses/secrets, relocating third-party server services, industrial cluster, mandatory migration to PostGIS. Web reproducibility does not replace training/competitive inferencing instructions that are made separately.

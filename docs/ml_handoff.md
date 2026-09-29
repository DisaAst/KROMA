# ML → Product handoff

## Current version

Initial main integration: `bf088ec8f9cf6e7bf3029ad55463805f47853e30`. It already had v006 artifacts, but service/manifest chose v005. Product integration connected existing red-edge maps/component statistics/ring contrast from the study, keeping float16 roundtrip, feature order and rules 0.3/0.8/0.2.

No retraining was carried out. `weighted_comboloss_sampler.pt`, at the subsequent instruction of the user, was not investigated or connected.

## Acceptance

Complete competition run: 180 AF, 89 BS, 447 lines, no failed chips. CSV-identical `artifacts/submissions/submission_v006.csv`. Three direct service BS runs (`BS_te_000001`, `000020`, `000089`) gave the same RLEs. TEST location/date not used. Timings and Control Team in `docs/key_metrics.md`.

## The next ML handoff

Pass a set of weights + manifest, SHA256, stable version, input/output contracts, threshold/feature order and reference submission. Backend calls adapter and should not import Torch/LightGBM. Do not replace v006 file silently: the new version requires invalidation cache and a new version of prepared datasets, UI/README/model card updates. After changing the set, restart the service.

Process limitation saved: MPS Torch separate from LightGBM. Competition neural stage is performed by batch. Web cache tied to chip and version; persistent worker not yet applicable.

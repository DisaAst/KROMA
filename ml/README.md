# ML

Production contract: `kroma_ml.service.predict_af` / `predict_bs`. CURRENT GOLD v006. Competition CLI: `python inference.py --data-dir data/test --output tmp/submission.csv`. Backend and competition use a single adapter. Full Launch, Weights, APIs and Limitations [README](../README.md); modelling — [model card](../docs/model_card.md); transference — [handoff](../docs/ml_handoff.md).

## Research and training

In product integration, training was not performed. Historical training programs and results are stored in `research/experiments` and [research/report.md](../research/report.md).

AF full-train bundle is reproduced by research `python research/experiments/af_final_model.py` in the presence of TRAIN and the hard negatives caches described in the report. BS training contract is available via `python research/experiments/bs_track.py --help`; ensemble configurations and stored normalization/channel_names are listed in checkpoint metadata. Full refiners built in series v004 → v005 → red-edge v006; `research/experiments/bs_v006_infer.py` contains training and requires OOF/cache inputs. This is a research chain, not a production installation team.

7 files from `artifacts/manifest.json` are enough to run the finished model. `data/processed` raw/OOF files don’t need inference. Archival DBSCAN/thermal-memory/priority modules saved as research history; Live backend doesn't pass them off as trained `P(real wildfire)`.

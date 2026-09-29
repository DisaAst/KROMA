# Research

Here are stored hypotheses and experiments. Create notebooks, reviews and reports only when you need them. Fix inputs and playback methods; do not add datasets or secrets to Git. Transfer the verified code to `ml/src/kroma_ml/` or `geo/src/kroma_geo/`.

The first reproducible pilot assessment is described in [document](../docs/research/evaluation.md). His recipe is in `research/experiments/build_evaluation_cohort.py`; the tags for candidates have not yet been collected.

The study of the problem, analogues and four AOIs is in [report](report.md). Configuration and manifest pilots lie in `experiments/`; great results remain in `data/processed/` outside Git.

The AF competition track is also documented in [this report](report.md): `experiments/af_unet_experiments.py` runs a separate configuration and smoke test, `experiments/af_ensemble.py` checks stored probability maps, `experiments/af_group_cv.py` compares models on three spatial folds. Intermediate `.pt` and `.npz` are excluded from Git; final production bundle is included in `ml/artifacts` and verified by manifest.

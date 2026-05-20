# Notebook Guide

Run notebooks in this order when working on Kaggle:

| Order | Notebook | Status | Use |
|---:|---|---|---|
| 1 | `eda.ipynb` | Reference | Understand data layout, missingness, `GR` behavior, and submission format. |
| 2 | `modeling.ipynb` | Stable baseline | Reproduce deterministic baselines and the rolling-feature residual tree. |
| 3 | `advanced-modeling.ipynb` | Current best | Reproduce the typewell-alignment approach that reached public score `15.049`. |
| 4 | `beam-pf-modeling.ipynb` | Main candidate | Test Beam/PF trajectory reconstruction, formation imputation, ensemble blending, and post-processing. |
| 5 | `dwt-modeling.ipynb` | Side candidate | Test multi-scale `GR` log-shape features inspired by DWT-style solutions. |

Current best public score: `15.049` from Advanced Modeling V2.

Advanced Modeling V3 scored `15.306`, so it is treated as a failed refinement rather than the active best path.

Use `beam-pf-modeling.ipynb` as the next serious leaderboard experiment. It is heavier than the other notebooks and is intended for Kaggle GPU runtime.

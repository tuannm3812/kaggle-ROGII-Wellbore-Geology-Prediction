# Notebook Guide

Run notebooks in this order when working on Kaggle:

| Order | Notebook | Status | Use |
|---:|---|---|---|
| 1 | `eda.ipynb` | Reference | Understand data layout, missingness, `GR` behavior, and submission format. |
| 2 | `modeling.ipynb` | Stable baseline | Reproduce deterministic baselines and the rolling-feature residual tree. |
| 3 | `advanced-modeling.ipynb` | Current best | Reproduce the typewell-alignment approach that reached public score `15.049`. |
| 4 | `dwt-modeling.ipynb` | Candidate | Test multi-scale `GR` log-shape features inspired by DWT-style solutions. |

Current best public score: `15.049` from Advanced Modeling V2.

Advanced Modeling V3 scored `15.306`, so it is treated as a failed refinement rather than the active best path.

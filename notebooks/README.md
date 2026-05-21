# Notebook Guide

Run notebooks in this order when working on Kaggle:

| Order | Notebook | Status | Use |
|---:|---|---|---|
| 1 | `01-eda.ipynb` | Reference | Understand data layout, missingness, `GR` behavior, and submission format. |
| 2 | `02-baseline-modeling.ipynb` | Stable baseline | Reproduce deterministic baselines and the rolling-feature residual tree. |
| 3 | `03-typewell-alignment-modeling.ipynb` | Previous best | Reproduce the typewell-alignment approach that reached public score `15.049`. |
| 4 | `04-beam-pf-modeling.ipynb` | Current best | Reproduce Beam/PF trajectory reconstruction, formation imputation, ensemble blending, and post-processing. |

Current best public score: `9.941` from Beam + Particle Filter V1.

Advanced Modeling V3 scored `15.306`, so it is treated as a failed refinement rather than the active best path.

DWT modeling is not part of the active notebook set. Beam/PF produced a much stronger public score and should be the main path for future improvements.

# ROGII - Wellbore Geology Prediction

![ROGII - Wellbore Geology Prediction banner](https://www.googleapis.com/download/storage/v1/b/kaggle-forum-message-attachments/o/inbox%2F4080021%2F3f56527c733365a94d929bdc0600c7ef%2Fig_023b4ba06ac0441e0169fa9248ca54819aacb93888a02601a8.png?generation=1778029361497538&alt=media)

Kaggle notebooks for the [ROGII - Wellbore Geology Prediction](https://www.kaggle.com/competitions/rogii-wellbore-geology-prediction/overview) competition.

The goal is to predict `TVT` (True Vertical Thickness) across the hidden interval of each horizontal wellbore. The notebooks are intended to run directly in Kaggle with the competition dataset attached; no local dependency setup is required.

## 1. Notebook Order

| Order | Notebook | Status | Purpose |
|---:|---|---|---|
| 1 | `notebooks/01-eda.ipynb` | Reference | Explore file layout, schema, missingness, well-level distributions, `GR`/`TVT` behavior, and submission format. |
| 2 | `notebooks/02-baseline-modeling.ipynb` | Stable baseline | Compare deterministic baselines and a feature-tree residual model under masked-tail validation. |
| 3 | `notebooks/03-typewell-alignment-modeling.ipynb` | Previous best | Reproduce the typewell-alignment approach that reached public score `15.049`. |
| 4 | `notebooks/04-beam-pf-modeling.ipynb` | Current best | Reproduce Beam/PF trajectory reconstruction and ensemble blending that reached public score `9.941`. |

See `notebooks/README.md` for the short run guide.

## 2. Data Layout

The competition uses one pair of CSV files per well:

```text
train/{WELLNAME}__horizontal_well.csv
train/{WELLNAME}__typewell.csv
test/{WELLNAME}__horizontal_well.csv
test/{WELLNAME}__typewell.csv
sample_submission.csv
```

Submission rows use:

```text
id,tvt
{WELLNAME}_{row_index},<predicted_tvt>
```

The notebooks resolve both common Kaggle mount paths:

```text
/kaggle/input/competitions/rogii-wellbore-geology-prediction
/kaggle/input/rogii-wellbore-geology-prediction
```

## 3. Current Findings

- Train set: `773` horizontal wells and `773` typewells.
- Public test sample: `3` wells and `14,151` requested predictions.
- Hidden `TVT_input` interval is long: roughly `73-74%` of each public-sample well.
- Test does not include train-only geology-top columns such as `ANCC`, `ASTNU`, `ASTNL`, `EGFDU`, `EGFDL`, and `BUDA`; using them directly would leak.
- Sampled train `GR` missingness is about `32%`.
- Global `GR` to `TVT` correlation is weak, so useful signal is more likely local log-shape alignment than simple row-wise regression.

## 4. Score Progress

Masked-tail validation currently favors carry-forward:

| Model | Mean RMSE | Median RMSE |
|---|---:|---:|
| `carry_forward` | 7.62 | 6.14 |
| `blend_0.25` | 8.43 | 6.87 |
| `damped_trend_035` | 8.99 | 7.26 |
| `linear_trend` | 14.05 | 10.02 |

The feature-tree residual model improved held-out-well validation slightly:

| Model | Validation RMSE |
|---|---:|
| `carry_forward` | 10.281 |
| `feature_tree` | 10.084 |

Latest Kaggle public scores:

| Version | Public score | Readout |
|---|---:|---|
| `V3` | `15.883` | Previous carry-forward/simple baseline level. |
| `V6` | `15.491` | Feature-tree submission improved the score by `0.392`. |
| `V7` | `15.491` | Latest rerun matched the current best; no additional gain. |
| Advanced Modeling `V2` | `15.049` | Typewell-alignment features improved the best score by another `0.442`. |
| Advanced Modeling `V3` | `15.306` | Denser offsets and residual shrinkage dropped versus V2; do not treat as best. |
| Beam + Particle Filter `V1` | `9.941` | Full trajectory candidates plus ensemble blending produced the largest gain. |

The feature baseline moved the public score from `15.883` to `15.491`, typewell-alignment features improved it to `15.049`, and Beam/PF reduced it further to `9.941`. The key lesson is that full `TVT` trajectory reconstruction is much stronger than row-wise residual modeling alone.

## 5. Current Direction

Beam/PF is the current best submitted path. It moves beyond residual features by building full `TVT` trajectory candidates with:

- Numba beam search over typewell `GR` paths;
- particle filters for hidden-interval trajectory reconstruction;
- multi-scale normalized cross-correlation between horizontal and typewell `GR`;
- spatial formation-plane and dense `ANCC` imputation;
- LightGBM/CatBoost ensemble blending and smoothing.

DWT modeling is not kept as an active repo notebook. The Beam/PF result already captures the more important idea: compare and reconstruct log trajectories directly, then blend the best candidate signals.

Next improvements should be controlled Beam/PF ablations:

- compare Beam/PF with and without spatial formation imputation;
- tune the post-processing grid around `alpha`, `tau`, and particle-filter blend weight;
- inspect per-public-well prediction curves for over-smoothing or trajectory jumps;
- simplify the Beam/PF notebook once the strongest components are confirmed.

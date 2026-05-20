# ROGII - Wellbore Geology Prediction

![ROGII - Wellbore Geology Prediction banner](https://www.googleapis.com/download/storage/v1/b/kaggle-forum-message-attachments/o/inbox%2F4080021%2F3f56527c733365a94d929bdc0600c7ef%2Fig_023b4ba06ac0441e0169fa9248ca54819aacb93888a02601a8.png?generation=1778029361497538&alt=media)

Kaggle notebooks for the [ROGII - Wellbore Geology Prediction](https://www.kaggle.com/competitions/rogii-wellbore-geology-prediction/overview) competition.

The goal is to predict `TVT` (True Vertical Thickness) across the hidden interval of each horizontal wellbore. The notebooks are intended to run directly in Kaggle with the competition dataset attached; no local dependency setup is required.

## 1. Notebook Order

| Order | Notebook | Status | Purpose |
|---:|---|---|---|
| 1 | `notebooks/eda.ipynb` | Reference | Explore file layout, schema, missingness, well-level distributions, `GR`/`TVT` behavior, and submission format. |
| 2 | `notebooks/modeling.ipynb` | Stable baseline | Compare deterministic baselines and a feature-tree residual model under masked-tail validation. |
| 3 | `notebooks/advanced-modeling.ipynb` | Current best | Reproduce the typewell-alignment approach that reached public score `15.049`. |
| 4 | `notebooks/beam-pf-modeling.ipynb` | Main candidate | Test Beam/PF trajectory reconstruction and ensemble blending. |
| 5 | `notebooks/dwt-modeling.ipynb` | Side candidate | Test DWT-inspired multi-scale `GR` log-shape matching against typewell offsets. |

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

The feature baseline moved the public score from `15.883` to `15.491`, then typewell-alignment features improved it further to `15.049`. This confirms the main EDA hypothesis: useful signal is more likely in local `GR` log-shape alignment than in simple row-wise regression or additional tuning of the same rolling-feature baseline.

## 5. Current Direction

The typewell-alignment experiment is the current best submitted path:

- candidate-`TVT` search around the carry-forward estimate;
- typewell `GR` interpolation at nearby `TVT` offsets;
- best local `GR` match, offset, difference, slope, and context-spread features;
- per-well normalized `GR`, `MD`, `X`, `Y`, and `Z` features.

The next main candidate is `notebooks/beam-pf-modeling.ipynb`. It moves beyond residual features by building full `TVT` trajectory candidates with beam search, particle filters, multi-scale normalized cross-correlation, spatial formation imputation, model ensembling, and post-processing.

`notebooks/dwt-modeling.ipynb` remains a lighter side candidate for multi-scale `GR` shape features, but Beam/PF is the better path for a large leaderboard jump.

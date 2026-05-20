# ROGII - Wellbore Geology Prediction

Kaggle notebooks for the [ROGII - Wellbore Geology Prediction](https://www.kaggle.com/competitions/rogii-wellbore-geology-prediction/overview) competition.

The goal is to predict `TVT` (True Vertical Thickness) across the hidden interval of each horizontal wellbore. The notebooks are intended to run directly in Kaggle with the competition dataset attached; no local dependency setup is required.

## 1. Notebooks

| Notebook | Purpose |
|---|---|
| `notebooks/eda.ipynb` | Explore file layout, schema, missingness, well-level distributions, `GR`/`TVT` behavior, and submission format. |
| `notebooks/modeling.ipynb` | Compare deterministic baselines and a feature-tree residual model under masked-tail validation, then write `submission.csv`. |

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

## 4. Current Baseline

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

Latest Kaggle public score:

| Metric | Value |
|---|---:|
| Latest score | `15.491` |
| Best score | `15.491` |
| Version | `V6` |

The feature baseline improved the public score from `15.883` to `15.491`, a `0.392` RMSE gain. The gap between local validation and public score still suggests validation is optimistic, but the direction is now useful: inference-safe rolling features can add signal beyond carry-forward.

## 5. Next Experiment

The feature tree has beaten carry-forward once, so the next experiment should add typewell-alignment features rather than only tuning the same rolling-feature model:

- local correlation between horizontal `GR` windows and typewell `GR` windows;
- candidate-TVT search around the carry-forward estimate;
- per-well normalized `GR`, `MD`, `X`, `Y`, and `Z` features.

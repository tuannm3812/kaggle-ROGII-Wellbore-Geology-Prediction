# ROGII - Wellbore Geology Prediction

## 1. Project Overview

This repository contains Kaggle-ready notebooks for the [ROGII - Wellbore Geology Prediction](https://www.kaggle.com/competitions/rogii-wellbore-geology-prediction/overview) competition.

The objective is to predict `TVT` (True Vertical Thickness) across the hidden interval of horizontal wellbores. The work is intentionally notebook-first and Kaggle-first: the notebooks are designed to run in the Kaggle competition environment with no local dependency setup.

Current workstream:

- inspect the file-per-well data structure;
- validate `sample_submission.csv` row indexing;
- understand `TVT_input` missingness and hidden evaluation windows;
- compare horizontal `GR` traces with typewell reference logs;
- build reliable inference-safe baselines;
- generate valid Kaggle `submission.csv` files.

## 2. Repository Contents

| Path | Purpose |
|---|---|
| `notebooks/kaggle-rogii-wellbore-geology-prediction-eda.ipynb` | EDA notebook with file discovery, schema checks, missingness summaries, well-level plots, dynamic insight callouts, and a simple carry-forward submission. |
| `notebooks/kaggle-rogii-wellbore-geology-prediction-eda-metadata.json` | Kaggle metadata for the EDA notebook. |
| `notebooks/kaggle-rogii-wellbore-geology-prediction-baseline-models.ipynb` | Baseline modeling notebook with masked-tail validation and deterministic per-well prediction strategies. |
| `notebooks/kaggle-rogii-wellbore-geology-prediction-baseline-models-metadata.json` | Kaggle metadata for the baseline modeling notebook. |
| `.gitignore` | Ignore rules for local artifacts, notebook checkpoints, generated submissions, and local data. |

## 3. Kaggle Runtime

Attach the competition data source in Kaggle. The notebooks first look for:

```text
/kaggle/input/competitions/rogii-wellbore-geology-prediction
```

They also fall back to:

```text
/kaggle/input/rogii-wellbore-geology-prediction
```

If Kaggle changes the mount layout, the notebooks recursively search `/kaggle/input` for `sample_submission.csv`.

Final submissions should run with internet disabled. The notebooks use standard Kaggle packages only:

- `numpy`
- `pandas`
- `matplotlib`
- `seaborn`

## 4. Notebook Workflow

### 4.1 EDA Notebook

`notebooks/kaggle-rogii-wellbore-geology-prediction-eda.ipynb` is a report-style exploration:

1. Resolve data paths, discover files, and parse the submission index.
2. Build lightweight horizontal-well and typewell metadata.
3. Review schema differences, missingness, and evaluation windows.
4. Plot representative wells with a consistent viridis palette.
5. Sample training rows for target and feature relationships.
6. Summarize EDA insights and modeling hypotheses.
7. Write a simple carry-forward baseline submission.

### 4.2 Baseline Models Notebook

`notebooks/kaggle-rogii-wellbore-geology-prediction-baseline-models.ipynb` compares deterministic, inference-safe baselines:

- carry-forward `TVT_input`;
- per-well linear trend extrapolation;
- damped trend extrapolation;
- validation-selected blends of carry-forward and trend.

The notebook writes:

```text
/kaggle/working/submission.csv
```

## 5. Competition Data Shape

The competition uses a file-per-well layout rather than a single `train.csv` / `test.csv` pair.

Per-well files:

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

The notebooks map each requested `row_index` back to the corresponding test horizontal-well CSV.

## 6. Current EDA Findings

From the saved public-sample run:

- train inventory: `773` horizontal wells and `773` typewells;
- public test inventory: `3` horizontal wells and `3` typewells;
- submission rows: `14,151` predictions across `3` public-sample wells;
- median hidden `TVT_input` interval: roughly `73-74%` of each well;
- train-only geology-top columns exist (`ANCC`, `ASTNU`, `ASTNL`, `EGFDU`, `EGFDL`, `BUDA`) and should be treated as leakage-risk direct features;
- sampled train `GR` missingness is about `32%`;
- global `GR` to `TVT` correlation is weak, so useful signal is likely local log-shape alignment rather than simple row-wise regression.

The public test sample is intentionally small and train-like. Treat public-sample behavior as a smoke test, not as a final estimate of hidden leaderboard performance.

## 7. Baseline Results

Masked-tail validation currently favors the simplest baseline:

| Model | Mean RMSE | Median RMSE | Readout |
|---|---:|---:|---|
| `carry_forward` | 7.62 | 6.14 | Best current simple baseline. |
| `blend_0.25` | 8.43 | 6.87 | Small trend weight hurts on average. |
| `damped_trend_035` | 8.99 | 7.26 | Damping helps, but not enough. |
| `blend_0.50` | 9.96 | 7.60 | Trend contribution is too large. |
| `blend_0.75` | 11.87 | 8.86 | Clear over-extrapolation. |
| `linear_trend` | 14.05 | 10.02 | Recent slope often does not persist. |

Latest Kaggle submission:

| Metric | Value |
|---|---:|
| Latest public score | `15.883` |
| Best public score | `15.883` |
| Submission version | `V3` |
| Daily submissions used | `2 / 5` |

Interpretation: the carry-forward baseline is valid and stable, but the public score is materially worse than masked-tail validation. That gap suggests our validation is still too easy or not representative enough. The next improvement should focus on validation design and inference-safe features before moving to complex models.

## 8. Recommended Next Steps

Priority order:

1. Strengthen validation.
   - evaluate more tail fractions;
   - group results by well type, hidden-window length, and `GR` missingness;
   - keep public-test wells out of modeling decisions except for smoke testing.
2. Improve inference-safe features.
   - add `TVT_input` slope, curvature, and local stability features;
   - add `GR` missingness flags and interpolation variants;
   - add rolling `GR` statistics;
   - normalize `GR`, `MD`, `X`, `Y`, and `Z` per well.
3. Add typewell alignment features.
   - compare horizontal `GR` windows with typewell `GR` windows indexed by candidate `TVT`;
   - create local correlation or distance features;
   - use typewell geology labels only as reference context, not leaked targets.
4. Train classical baselines.
   - start with tree models on masked-tail rows;
   - compare against carry-forward under the same validation protocol;
   - submit only when validation improves consistently across wells.

Decision rule: move to advanced sequence models or fine-tuned models only after an inference-safe classical baseline beats carry-forward consistently across multiple masked-tail validation settings.

## 9. Kaggle CLI Notes

The JSON files are Kaggle kernel metadata files. They are not model inputs; they tell the Kaggle CLI which notebook to upload, which competition dataset to attach, and whether internet, GPU, or TPU are enabled.

Before using the Kaggle CLI, replace:

```text
YOUR_KAGGLE_USERNAME
```

with your Kaggle username.

To use the Kaggle CLI from the flat `notebooks/` folder, copy the relevant metadata file to `kernel-metadata.json` before pushing:

```bash
cp notebooks/kaggle-rogii-wellbore-geology-prediction-baseline-models-metadata.json notebooks/kernel-metadata.json
kaggle kernels push -p notebooks
```

# ROGII - Wellbore Geology Prediction

## 01 - Project Overview

This repository contains Kaggle-only notebooks for the [ROGII - Wellbore Geology Prediction](https://www.kaggle.com/competitions/rogii-wellbore-geology-prediction/overview) competition.

The task is to predict `TVT` (True Vertical Thickness) across the hidden evaluation interval of horizontal wellbores. The project is intentionally lightweight: run the notebooks inside Kaggle with the competition dataset attached, without local dependency setup.

Current focus:

- understand the per-well CSV structure;
- validate the submission row index format;
- explore `TVT_input` missingness and hidden evaluation windows;
- compare horizontal `GR` traces with typewell reference logs;
- build and compare simple baseline models;
- generate valid Kaggle `submission.csv` files.

## 02 - Repository Contents

| Path | Purpose |
|---|---|
| `notebooks/eda/notebook.ipynb` | EDA-first Kaggle notebook with dynamic insight callouts, viridis-styled plots, validation checks, and a simple submission baseline. |
| `notebooks/eda/kernel-metadata.json` | Kaggle kernel metadata for the EDA notebook. |
| `notebooks/baseline-models/notebook.ipynb` | Baseline modeling notebook with masked-tail validation and several lightweight per-well prediction strategies. |
| `notebooks/baseline-models/kernel-metadata.json` | Kaggle kernel metadata for the baseline models notebook. |
| `.gitignore` | Ignore rules for local artifacts, notebook checkpoints, and generated submissions. |

## 03 - Kaggle Runtime Setup

Attach the competition data source in Kaggle. The notebooks first look for:

```text
/kaggle/input/competitions/rogii-wellbore-geology-prediction
```

They also fall back to:

```text
/kaggle/input/rogii-wellbore-geology-prediction
```

If Kaggle changes the mount layout, the notebooks recursively search `/kaggle/input` for `sample_submission.csv`.

Final competition submissions should run with internet disabled. The notebooks use standard Kaggle packages only:

- `numpy`
- `pandas`
- `matplotlib`
- `seaborn`

## 04 - Notebook Workflow

### 04.1 - EDA Notebook

`notebooks/eda/notebook.ipynb` is organized as a report-style workflow:

1. Resolve data paths, discover files, and parse the submission index.
2. Build lightweight horizontal-well and typewell metadata.
3. Review schema differences, missingness, and evaluation windows.
4. Plot representative wells using a consistent viridis palette.
5. Sample training rows for target and feature relationships.
6. Summarize EDA insights and modeling hypotheses.
7. Write a simple carry-forward baseline submission.

### 04.2 - Baseline Models Notebook

`notebooks/baseline-models/notebook.ipynb` compares deterministic, inference-safe baselines under masked-tail validation:

- carry-forward `TVT_input`;
- per-well linear trend extrapolation;
- damped trend extrapolation;
- validation-selected blend of carry-forward and trend.

The notebook writes:

```text
/kaggle/working/submission.csv
```

## 05 - Competition Data Shape

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

## 06 - EDA Insights

The EDA notebook includes dynamic markdown insight blocks that are regenerated from the data each time it runs on Kaggle. These callouts summarize:

- train/test well inventory;
- public sample submission coverage;
- hidden `TVT_input` fraction by split;
- train-only versus inference-time columns;
- target and `GR` missingness patterns;
- carry-forward validation performance;
- generated submission quality checks.

The saved public-sample run showed 773 training wells, 3 public test wells, 14,151 requested prediction rows, and a median hidden `TVT_input` interval of roughly three quarters of each well. Treat those numbers as public-sample context, not final hidden-test assumptions.

## 07 - Recommended Next Steps

Use masked-tail validation on training wells to mimic the competition setup. Compare every model against the carry-forward baseline under the same validation protocol.

Promising feature directions:

- slope and curvature features from the known `TVT_input` prefix;
- rolling-window `GR` features and missingness flags;
- per-well normalization for `GR`, `MD`, and spatial coordinates;
- typewell alignment features using local similarity between horizontal `GR` and typewell `GR` indexed by `TVT`.

Promising model directions:

- per-well linear or spline extrapolation;
- tree models trained on tail-masked rows;
- sequence models or alignment-based models once validation is stable.

## 08 - Kaggle CLI Note

The JSON files are Kaggle kernel metadata files. They are not model inputs; they tell the Kaggle CLI which notebook to upload, which competition dataset to attach, and whether internet/GPU/TPU are enabled.

Before using the Kaggle CLI, replace:

```text
YOUR_KAGGLE_USERNAME
```

with your Kaggle username.

Then push a notebook folder:

```bash
kaggle kernels push -p notebooks/eda
kaggle kernels push -p notebooks/baseline-models
```

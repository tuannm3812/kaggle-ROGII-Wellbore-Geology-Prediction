# ROGII - Wellbore Geology Prediction

![ROGII Wellbore Geology Prediction](https://www.googleapis.com/download/storage/v1/b/kaggle-forum-message-attachments/o/inbox%2F4080021%2F3f56527c733365a94d929bdc0600c7ef%2Fig_023b4ba06ac0441e0169fa9248ca54819aacb93888a02601a8.png?generation=1778029361497538&alt=media)

## 01 - Project Overview

This repository contains a Kaggle-only exploratory analysis and starter baseline notebook for the [ROGII - Wellbore Geology Prediction](https://www.kaggle.com/competitions/rogii-wellbore-geology-prediction/overview) competition.

The task is to predict `TVT` (True Vertical Thickness) across the hidden evaluation interval of horizontal wellbores. The notebook is designed to run inside Kaggle with the competition dataset attached; local dependency setup is intentionally not required.

### 01.1 - Current Focus

- Understand the per-well CSV structure.
- Validate the submission row index format.
- Explore `TVT_input` missingness and hidden evaluation windows.
- Compare horizontal `GR` traces with typewell reference logs.
- Generate a valid carry-forward baseline submission.

## 02 - Repository Contents

| Path | Purpose |
|---|---|
| `rogii_wellbore_geology_prediction.ipynb` | EDA-first Kaggle notebook with dynamic insight callouts, viridis-styled plots, validation checks, and baseline submission generation. |
| `kernel-metadata.json` | Kaggle kernel metadata with the competition source configured and internet disabled. |
| `.gitignore` | Ignore rules for local artifacts, notebook checkpoints, and generated submissions. |

## 03 - Kaggle Runtime Setup

### 03.1 - Data Mount

Attach the competition data source in Kaggle. The notebook first looks for:

```text
/kaggle/input/competitions/rogii-wellbore-geology-prediction
```

It also falls back to:

```text
/kaggle/input/rogii-wellbore-geology-prediction
```

If Kaggle changes the mount layout, the notebook performs a recursive search under `/kaggle/input` for `sample_submission.csv`.

### 03.2 - Internet Setting

Final competition submissions should run with internet disabled. The notebook uses standard Kaggle packages only:

- `numpy`
- `pandas`
- `matplotlib`
- `seaborn`

## 04 - Notebook Workflow

### 04.1 - EDA Sections

The notebook is organized as a report-style workflow:

1. Resolve data paths and discover files.
2. Parse `sample_submission.csv` into `well` and `row_idx`.
3. Build lightweight horizontal-well and typewell metadata.
4. Review schema differences and missingness.
5. Plot representative wells using a consistent viridis palette.
6. Sample training rows for target and feature relationships.
7. Summarize EDA insights and modeling hypotheses.

### 04.2 - Baseline Section

The starter baseline fills the hidden interval by carrying forward the last known `TVT_input` value for each well, then writes:

```text
/kaggle/working/submission.csv
```

This baseline is intentionally simple. Its main purpose is to verify the full Kaggle submission path and provide a reference point for future modeling.

## 05 - Competition Data Shape

The competition uses a file-per-well layout rather than a single `train.csv` / `test.csv` pair.

### 05.1 - Per-Well Files

```text
train/{WELLNAME}__horizontal_well.csv
train/{WELLNAME}__typewell.csv
test/{WELLNAME}__horizontal_well.csv
test/{WELLNAME}__typewell.csv
sample_submission.csv
```

### 05.2 - Submission Format

Submission rows use:

```text
id,tvt
{WELLNAME}_{row_index},<predicted_tvt>
```

The notebook maps each requested `row_index` back to the corresponding test horizontal-well CSV.

## 06 - EDA Insights Captured In The Notebook

The notebook now includes dynamic markdown insight blocks that are regenerated from the data each time it runs on Kaggle. These callouts summarize:

- train/test well inventory;
- public sample submission coverage;
- hidden `TVT_input` fraction by split;
- train-only versus inference-time columns;
- target and `GR` missingness patterns;
- carry-forward validation performance;
- generated submission quality checks.

The saved public-sample run showed 773 training wells, 3 public test wells, 14,151 requested prediction rows, and a median hidden `TVT_input` interval of roughly three quarters of each well. Treat those numbers as public-sample context, not final hidden-test assumptions.

## 07 - Recommended Next Steps

### 07.1 - Validation

Use masked-tail validation on training wells to mimic the competition setup. Compare every model against the carry-forward baseline under the same validation protocol.

### 07.2 - Feature Engineering

Promising directions:

- slope and curvature features from the known `TVT_input` prefix;
- rolling-window `GR` features and missingness flags;
- per-well normalization for `GR`, `MD`, and spatial coordinates;
- typewell alignment features using local similarity between horizontal `GR` and typewell `GR` indexed by `TVT`.

### 07.3 - Modeling

Start with interpretable baselines before moving to heavier models:

- per-well linear or spline extrapolation;
- tree models trained on tail-masked rows;
- sequence models or alignment-based models once validation is stable.

## 08 - Kaggle CLI Note

Before using the Kaggle CLI with `kernel-metadata.json`, replace:

```text
YOUR_KAGGLE_USERNAME
```

with your Kaggle username.

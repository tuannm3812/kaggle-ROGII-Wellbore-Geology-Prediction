# ROGII - Wellbore Geology Prediction

This repository contains a Kaggle-only starter notebook for the [ROGII - Wellbore Geology Prediction](https://www.kaggle.com/competitions/rogii-wellbore-geology-prediction/overview) competition.

The competition is a code/notebook competition. You do not need to set up local dependencies for this repo; run the notebook on Kaggle with the competition data attached.

## Files

- `rogii_wellbore_geology_prediction.ipynb` — Kaggle notebook with first-pass EDA, file discovery, schema/missingness summaries, well-level plots, a carry-forward `TVT_input` baseline, and `submission.csv` generation.
- `kernel-metadata.json` — Kaggle kernel metadata for uploading/running the notebook with internet disabled. Replace `YOUR_KAGGLE_USERNAME` with your Kaggle username before using the Kaggle CLI.
- `.gitignore` — Basic ignore patterns for notebook artifacts.

## Usage

1. Open the notebook on Kaggle.
2. Attach the competition data source. Kaggle should mount it at:
   `/kaggle/input/rogii-wellbore-geology-prediction`
3. Run the EDA sections first to inspect the data, then run the baseline section when you want a starter submission.
4. Submit the generated file:
   `/kaggle/working/submission.csv`

## Competition Data Shape

The dataset uses per-well CSV files instead of a single `train.csv`/`test.csv` pair:

- `train/{WELLNAME}__horizontal_well.csv`
- `train/{WELLNAME}__typewell.csv`
- `test/{WELLNAME}__horizontal_well.csv`
- `test/{WELLNAME}__typewell.csv`
- `sample_submission.csv`

Submission rows use `id,tvt`, where `id = {WELLNAME}_{row_index}`. The notebook reads each test horizontal CSV, fills/predicts the hidden `TVT_input` zone with a simple carry-forward baseline, and maps predictions back to the requested row indices.

## Notes

- This repo intentionally does not include dependency setup because Kaggle provides the runtime.
- The notebook uses standard Kaggle packages: `pandas` and `numpy`.
- Internet must be disabled for final Kaggle submissions.

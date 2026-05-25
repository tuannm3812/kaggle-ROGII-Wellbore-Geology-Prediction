# ROGII - Wellbore Geology Prediction

![ROGII - Wellbore Geology Prediction banner](https://www.googleapis.com/download/storage/v1/b/kaggle-forum-message-attachments/o/inbox%2F4080021%2F3f56527c733365a94d929bdc0600c7ef%2Fig_023b4ba06ac0441e0169fa9248ca54819aacb93888a02601a8.png?generation=1778029361497538&alt=media)

Kaggle notebooks for the [ROGII - Wellbore Geology Prediction](https://www.kaggle.com/competitions/rogii-wellbore-geology-prediction/overview) competition.

The goal is to predict `TVT` (True Vertical Thickness) across the hidden interval of each horizontal wellbore. The notebooks are intended to run directly in Kaggle with the competition dataset attached; no local dependency setup is required.

Current selected submission: Beam + Particle Filter `V1`, public score `9.941`.
Later artifact-workflow reruns are useful for reproducibility checks, but they have not beaten the selected public score.

## 1. Project Snapshot

This repository presents a notebook-first geoscience modeling workflow:

- parse well-level horizontal and typewell logs;
- study hidden `TVT_input` intervals and leakage boundaries;
- build masked-tail validation that resembles competition inference;
- compare carry-forward, feature-tree, typewell-alignment, and Beam/PF models;
- package Beam/PF trained models as Kaggle artifacts for submission-mode replay.

The strongest public result remains Beam + Particle Filter `V1` with public RMSE `9.941`. Later reruns improved reproducibility but scored lower on the visible public set, so V1 remains the selected submission.

## 2. Technical Skills

| Area | Evidence In This Project |
|---|---|
| Geoscience ML | `TVT` trajectory reconstruction from horizontal well logs, paired typewells, and gamma ray signals. |
| Validation design | Held-out-well masked-tail validation to avoid row-level leakage. |
| Feature engineering | Rolling `GR` features, `TVT_input` slopes, spatial coordinates, typewell alignment, formation-plane estimates, and trajectory candidates. |
| Sequence reconstruction | Beam search and particle-filter style trajectory candidates for long hidden intervals. |
| Modeling | `HistGradientBoostingRegressor`, LightGBM, CatBoost, ridge stacking, and post-processing. |
| Kaggle operations | Notebook path discovery, offline-safe runs, `submission.csv` generation, and private model artifact replay. |
| Documentation | Reviewer-facing result summaries, score interpretation, run instructions, and coding standards. |

## 3. Repository Structure

```text
.
├── README.md
├── docs/
│   ├── 1_instructions.md
│   ├── 2_eda_insights.md
│   ├── 3_baseline_models.md
│   └── coding_standards.md
└── notebooks/
    ├── 1_rogii_eda.ipynb
    ├── 2_rogii_baseline.ipynb
    ├── 3_rogii_typewell_alignment.ipynb
    └── 4_rogii_beam_pf.ipynb
```

The repository intentionally does not store raw competition data, trained model zips, Kaggle working directories, or local checkpoints. Large reusable artifacts should stay on Kaggle as private model inputs.

## 4. Run Instructions

Run the notebooks on Kaggle in order:

| Order | Notebook | Status | Purpose |
|---:|---|---|---|
| 1 | `notebooks/1_rogii_eda.ipynb` | Reference | Explore file layout, schema, missingness, well-level distributions, `GR`/`TVT` behavior, and submission format. |
| 2 | `notebooks/2_rogii_baseline.ipynb` | Stable baseline | Compare deterministic baselines and a feature-tree residual model under masked-tail validation. |
| 3 | `notebooks/3_rogii_typewell_alignment.ipynb` | Previous best | Reproduce the typewell-alignment approach that reached public score `15.049`. |
| 4 | `notebooks/4_rogii_beam_pf.ipynb` | Selected best | Reproduce Beam/PF trajectory reconstruction and ensemble blending. V1 remains selected with public score `9.941`. |

For Beam/PF:

1. Run with `CFG.MODE = "train"` to train models, write `submission.csv`, and create `rogii_beam_pf_artifacts.zip`.
2. Upload the artifact zip as a private Kaggle model input.
3. Run again with `CFG.MODE = "submission"` to load artifacts and write `submission.csv` without retraining.
4. Compare train-mode and submission-mode predictions before treating the artifact as reusable.

See `docs/1_instructions.md` for the detailed run guide and modeling approach notes.

Detailed project notes live in:

- `docs/1_instructions.md` for workflow instructions and modeling approaches;
- `docs/2_eda_insights.md` for detailed EDA findings and chart guidance;
- `docs/3_baseline_models.md` for baseline and model result details.

## 5. Data Layout

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

## 6. Current Lessons

- Train set: `773` horizontal wells and `773` typewells.
- Public test sample: `3` wells and `14,151` requested predictions.
- Hidden `TVT_input` interval is long: roughly `73-74%` of each public-sample well.
- Test does not include train-only geology-top columns such as `ANCC`, `ASTNU`, `ASTNL`, `EGFDU`, `EGFDL`, and `BUDA`; using them directly would leak.
- Sampled train `GR` missingness is about `32%`.
- Global `GR` to `TVT` correlation is weak, so useful signal is more likely local log-shape alignment than simple row-wise regression.
- Public score is volatile because the visible public sample has only `3` wells.
- Local masked-tail validation is useful for direction, but it has not ranked Beam/PF reruns exactly the same way as the public leaderboard.
- Beam/PF is the strongest modeling family, but newer reruns can still underperform the selected V1 public score.

## 7. Notebook Output Insights

| Notebook | Latest Output | Insight |
|---|---:|---|
| EDA | `773` train wells, `3` public test wells, `14,151` submission rows | Treat the public leaderboard as a small-sample smoke test, not a stable model ranking. |
| EDA | Median hidden suffix: train `74.0%`, public test `72.7%` | The task is long-range trajectory continuation, not short interpolation. |
| EDA | Sampled `GR` missingness `32.0%`; sampled `GR`/`TVT` correlation `-0.052` | Missingness handling and local log matching matter more than global correlation. |
| Baseline | `10.281 -> 10.084` validation RMSE | Row-wise residual features help, but only modestly. |
| Typewell alignment | `10.281 -> 9.799` validation RMSE | Typewell-aware `GR` alignment adds a real new signal family. |
| Beam/PF latest train run | Stack `10.440`, post-process `10.410` local RMSE | The trajectory ensemble is still the strongest modeling family locally, but the latest public score did not improve. |
| Beam/PF artifact replay | V3 used V2 artifacts; V5 uses best-iteration artifact preservation | Artifact mode is now the right workflow for reproducible submission reruns. |

## 8. Score Progress

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
| `V7` | `15.491` | Rerun matched V6; no additional gain. |
| Advanced Modeling `V2` | `15.049` | Typewell-alignment features improved the best score by another `0.442`. |
| Advanced Modeling `V3` | `15.306` | Denser offsets and residual shrinkage dropped versus V2; do not treat as best. |
| Beam + Particle Filter `V1` | `9.941` | Selected best; full trajectory candidates plus ensemble blending produced the largest gain. |
| Beam + Particle Filter `V3` | `10.197` | Submission-mode replay from V2 artifacts; reproducible but worse than V1. |
| Beam + Particle Filter `V5` | `10.212` | Artifact workflow with LightGBM best-iteration preservation; still worse than V1. |

The feature baseline moved the public score from `15.883` to `15.491`, typewell-alignment features improved it to `15.049`, and Beam/PF reduced it further to `9.941`. The later Beam/PF reruns scored `10.197` and `10.212`, so the selected public submission should remain V1. The key lesson is still that full `TVT` trajectory reconstruction is much stronger than row-wise residual modeling alone.

## 9. Next Work

The most useful improvements now are diagnostic and controlled:

- compare selected V1 predictions against V3/V5 predictions by public well;
- inspect why local validation improved from V1 to later runs while public score dropped;
- create per-well prediction curve plots for public and validation wells;
- compare Beam/PF with and without spatial formation imputation;
- test whether all three LightGBM variants are worth the runtime, since CatBoost often dominates the stack;
- tune the post-processing grid around `alpha`, `tau`, and particle-filter blend weight;
- keep the artifact workflow, but select submissions by public score until a stronger validation split is available.

The best immediate notebook improvement is a small diagnostics section in `4_rogii_beam_pf.ipynb` that writes per-well prediction summaries and optional V1/V3/V5 comparison plots when submission files are available.

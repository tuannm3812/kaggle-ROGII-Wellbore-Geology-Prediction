# ROGII - Wellbore Geology Prediction

![ROGII - Wellbore Geology Prediction banner](https://www.googleapis.com/download/storage/v1/b/kaggle-forum-message-attachments/o/inbox%2F4080021%2F3f56527c733365a94d929bdc0600c7ef%2Fig_023b4ba06ac0441e0169fa9248ca54819aacb93888a02601a8.png?generation=1778029361497538&alt=media)

**Notebook-first solution work** for the [ROGII - Wellbore Geology Prediction](https://www.kaggle.com/competitions/rogii-wellbore-geology-prediction/overview) Kaggle competition.

The competition asks participants to predict **`TVT` (True Vertical Thickness)** across the hidden interval of each horizontal wellbore. The modeling challenge is not a simple row-wise regression problem: each well has a **long hidden suffix**, a visible **`TVT_input` prefix**, a horizontal **`GR` log**, spatial coordinates, and a paired **typewell reference**.

The strongest submitted approach in this repository is **Beam + Particle Filter V1**, with public RMSE `9.941`. Later artifact-workflow reruns improved reproducibility but scored lower on the public leaderboard, so V1 remains the selected submission.

Current selected status:

- `V1` remains selected for leaderboard.
- `V3` and `V5` are diagnostic branches.

Public score checkpoints:

- V1: `9.941` — baseline public-best Beam/PF production path with full trajectory-stack.
- V3: `10.197` — reproducible artifact and versioned diagnostic replay path.
- V5: `10.212` — submission replay using V2 artifact bundle and best-iteration preservation.

## 1. Project Overview

This project builds a sequence-reconstruction workflow for wellbore geology:

1. Explore well-level file structure, hidden interval length, missingness, and leakage boundaries.
2. Establish carry-forward and feature-tree baselines under masked-tail validation.
3. Add typewell-aware `GR` alignment features to improve long-tail inference.
4. Build Beam/PF trajectory candidates and blend them with LightGBM/CatBoost models.
5. Package trained Beam/PF artifacts for Kaggle submission-mode replay.

The key modeling lesson is that full **`TVT` trajectory reconstruction** is much stronger than tuning row-wise residual features alone.

## 2. Technical Skills

| Area | Evidence In This Project |
|---|---|
| Geoscience ML | `TVT` trajectory reconstruction from horizontal wells, typewells, gamma ray logs, and spatial coordinates. |
| Validation design | Held-out-well masked-tail validation that mimics the hidden suffix in test wells. |
| Feature engineering | Rolling `GR` features, `TVT_input` slopes, typewell offset search, formation-plane estimates, and trajectory candidate features. |
| Sequence modeling | Beam-search and particle-filter style candidates for long hidden intervals. |
| Model stacking | LightGBM, CatBoost, ridge blending, post-processing shrinkage, and smoothing. |
| Kaggle workflow | Path auto-detection, offline-safe notebooks, `submission.csv` generation, and private artifact replay. |
| Technical communication | Reviewer-facing EDA notes, baseline readouts, score interpretation, and coding standards. |

## 3. Repository Structure

```text
.
├── README.md
├── docs/
│   ├── 0_readme.md
│   ├── 1_instructions.md
│   ├── 2_eda_insights.md
│   ├── 3_baseline_models.md
│   ├── 4_next_steps.md
│   ├── 7_submission_score_registry.md
│   ├── 6_kaggle_autosubmit_runbook.md
│   └── 5_coding_standards.md
└── notebooks/
    ├── 1_rogii_eda.ipynb
    ├── 2_rogii_baseline.ipynb
    ├── 3_rogii_typewell_alignment.ipynb
    └── 4_rogii_beam_pf.ipynb
```

Detailed notes:

- `docs/1_instructions.md`: competition framing, run order, approaches, and next analysis.
- `docs/2_eda_insights.md`: EDA findings, sample charts, and deeper-analysis targets.
- `docs/0_readme.md`: documentation index and consistency rules.
- `docs/1_instructions.md`: competition framing, run order, approaches, and next analysis.
- `docs/2_eda_insights.md`: EDA findings, sample charts, and deeper-analysis targets.
- `docs/3_baseline_models.md`: baseline, typewell-alignment, and Beam/PF result readouts.
- `docs/4_next_steps.md`: current priority execution plan and completion criteria.
- `docs/7_submission_score_registry.md`: versioned submission score history and promotion policy.
- `docs/6_kaggle_autosubmit_runbook.md`: Kaggle GPU submission workflow.
- `docs/5_coding_standards.md`: notebook and documentation standards for this project.

## 4. Notebook Flow

Run the notebooks on Kaggle in order.

| Order | Notebook | Role | Main Output |
|---:|---|---|---|
| 1 | `notebooks/1_rogii_eda.ipynb` | EDA | Data inventory, missingness, hidden suffix analysis, leakage checks. |
| 2 | `notebooks/2_rogii_baseline.ipynb` | Baseline | Carry-forward, trend, blend, and feature-tree residual model. |
| 3 | `notebooks/3_rogii_typewell_alignment.ipynb` | Alignment model | Typewell-aware `GR` alignment features and residual model. |
| 4 | `notebooks/4_rogii_beam_pf.ipynb` | Selected modeling path | Beam/PF candidates, ensemble blending, `submission.csv`, artifact zip. |

Beam/PF supports two runtime modes:

| Mode | Use Case | Output |
|---|---|---|
| `CFG.MODE = "train"` | Train models and produce artifacts. | `submission.csv` and `rogii_beam_pf_artifacts.zip`. |
| `CFG.MODE = "submission"` | Load attached artifacts and submit without retraining. | `submission.csv`. |

Quick sanity run pattern:

```python
# In the notebook, switch CFG.MODE and run from section 3 onward.
CFG.MODE = "train"      # quick local path
CFG.MODE = "submission"  # replay path (requires attached artifacts)
```

## 5. Data Shape

The competition uses one horizontal-well CSV and one typewell CSV per well:

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

Important EDA readouts:

| Finding | Value | Implication |
|---|---:|---|
| Training inventory | `773` horizontal wells and `773` typewells | Use grouped validation by well. |
| Public test sample | `3` wells and `14,151` rows | Public score is high signal but small sample. |
| Median hidden suffix | `74.0%` train, `72.7%` public test | Model long trajectory continuation, not short interpolation. |
| Sampled `GR` missingness | `32.0%` | Missingness handling is core model logic. |
| Sampled `GR`/`TVT` correlation | `-0.052` | Local log-shape alignment matters more than global correlation. |

## 6. Results

Validation and public results show a clear progression from simple baselines to trajectory reconstruction.

| Stage | Local Readout | Public Score | Interpretation |
|---|---:|---:|---|
| Carry-forward/simple baseline | `10.281` validation RMSE | `15.883` | Strong fallback, weak geological model. |
| Feature-tree residual model | `10.084` validation RMSE | `15.491` | Small but consistent improvement. |
| Typewell alignment | `9.799` validation RMSE | `15.049` | Typewell `GR` matching adds useful signal. |
| Beam/PF V1 | Historical selected run | `9.941` | Best public submission. |
| Beam/PF V3 | Artifact replay from V2 | `10.197` | Reproducible but worse than V1. |
| Beam/PF V5 | Best-iteration artifact workflow | `10.212` | Better artifact discipline, still worse than V1. |

Latest Beam/PF local run:

| Component | Local RMSE |
|---|---:|
| `lgb0` | 10.786 |
| `lgb1` | 10.691 |
| `lgb2` | 10.747 |
| `catboost` | 10.549 |
| ridge stack | 10.440 |
| post-process | 10.410 |

The local Beam/PF validation improved in later reruns, but the public score dropped versus V1. That mismatch is the current main diagnostic target.

## 7. Current Lessons

- The task is a long hidden-suffix reconstruction problem, not a standard tabular regression task.
- Train-only geology-top columns are useful context but cannot be used directly in test-time features.
- Carry-forward is a strong safety baseline, but it does not capture enough trajectory shape.
- Typewell alignment helps most when the hidden interval is long.
- Beam/PF is the strongest modeling family in this project.
- Public ranking can move sharply because the visible public test contains only three wells.
- Artifact replay is necessary for reproducibility, but it does not guarantee a better public score.

## 8. Next Work

The highest-value notebook improvement is diagnostic rather than another broad model rewrite:

1. Add a **Beam/PF diagnostics** section that compares V1, V3, and V5 predictions by public well when those submission files are attached.
2. Plot **per-well `TVT` curves** to find where newer runs diverge from the selected V1 submission.
3. Add a small **ablation table** for CatBoost-only, LightGBM-only, ridge stack, and post-processing.
4. Test whether all three **LightGBM variants** are worth the runtime, since CatBoost often receives the largest stack weight.
5. Tune **post-processing** only after identifying which public well caused the score drop.

The immediate priority is explaining the V1 versus V5 public-score gap before adding more model complexity.

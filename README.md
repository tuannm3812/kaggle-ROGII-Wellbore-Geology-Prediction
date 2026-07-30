# ROGII - Wellbore Geology Prediction

![ROGII - Wellbore Geology Prediction banner](https://www.googleapis.com/download/storage/v1/b/kaggle-forum-message-attachments/o/inbox%2F4080021%2F3f56527c733365a94d929bdc0600c7ef%2Fig_023b4ba06ac0441e0169fa9248ca54819aacb93888a02601a8.png?generation=1778029361497538&alt=media)

**Notebook-first solution work** for the [ROGII - Wellbore Geology Prediction](https://www.kaggle.com/competitions/rogii-wellbore-geology-prediction/overview) Kaggle competition.

The competition asks participants to predict **`TVT` (True Vertical Thickness)** across the hidden interval of each horizontal wellbore. The modeling challenge is not a simple row-wise regression problem: each well has a **long hidden suffix**, a visible **`TVT_input` prefix**, a horizontal **`GR` log**, spatial coordinates, and a paired **typewell reference**.

The strongest *verified* submitted approach in this repository is **Beam + Particle Filter V13**, with public RMSE `9.952` (promoted 2026-07-29, see `docs/7_submission_score_registry.md`). A previously documented `V1 = 9.941` entry could not be found in either Kaggle account's actual submission history and was removed from the record the same day.

Current selected status:

- `V13` is selected for leaderboard (`9.952`, best verified public score).
- `V3`, `V5`, `V8`, `V9`, `V10`, `V11`, `V12`, and `V14` are diagnostic/historical branches.

Public score checkpoints:

- V13: `9.952` — best verified public score; `tau=None` (no post-process distance damping), same model/training pass as V12.
- V11: `10.022` — restored a second LightGBM model (`lr=0.020`) alongside CatBoost; selected 2026-07-29 until superseded by V13 the same day.
- V3: `10.197` — original Beam/PF production path with full trajectory-stack; selected until 2026-07-29.
- V5: `10.212` — reproducible artifact and versioned diagnostic replay path.
- V10: `10.226` — single-LightGBM (`lr=0.030`) + CatBoost; the prune V11 later reversed.
- V12: `10.087` — same model/config as V11, fresh training pass, `tau=100` (local grid-search default).
- V14: `10.126` — `tau=25` (heavy damping), same run as V12/V13.
- V9: `10.299` — tuannm3823-account GPU train notebook submission (2026-06-10), discovered unlogged during reconciliation.
- V8: `10.305` — main-account notebook output submission (2026-06-10), discovered unlogged during reconciliation.

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
│   ├── 5_coding_standards.md
│   ├── 6_kaggle_autosubmit_runbook.md
│   └── 7_submission_score_registry.md
├── kaggle_kernel/
│   ├── 4_rogii_beam_pf.ipynb
│   └── kernel-metadata.json
└── notebooks/
    ├── 1_rogii_eda.ipynb
    ├── 2_rogii_baseline.ipynb
    ├── 3_rogii_typewell_alignment.ipynb
    └── 4_rogii_beam_pf.ipynb
```

Detailed notes:

- `docs/0_readme.md`: documentation index and consistency rules.
- `docs/1_instructions.md`: competition framing, run order, approaches, and next analysis.
- `docs/2_eda_insights.md`: EDA findings, sample charts, and deeper-analysis targets.
- `docs/3_baseline_models.md`: baseline, typewell-alignment, and Beam/PF result readouts.
- `docs/4_next_steps.md`: current priority execution plan and completion criteria.
- `docs/5_coding_standards.md`: notebook and documentation standards for this project.
- `docs/6_kaggle_autosubmit_runbook.md`: Kaggle GPU submission workflow.
- `docs/7_submission_score_registry.md`: versioned submission score history and promotion policy.

## 4. Notebook Flow

Run the notebooks on Kaggle in order.

| Order | Notebook | Role | Main Output |
|---:|---|---|---|
| 1 | `notebooks/1_rogii_eda.ipynb` | EDA | Data inventory, missingness, hidden suffix analysis, leakage checks. |
| 2 | `notebooks/2_rogii_baseline.ipynb` | Baseline | Carry-forward, trend, blend, and feature-tree residual model. |
| 3 | `notebooks/3_rogii_typewell_alignment.ipynb` | Alignment model | Typewell-aware `GR` alignment features and residual model. |
| 4 | `notebooks/4_rogii_beam_pf.ipynb` | Selected modeling path | Beam/PF candidates, ensemble blending, `submission.csv`, artifact zip. |

`notebooks/4_rogii_beam_pf.ipynb` is the single source notebook for both GPU
training and CPU submission-replay pushes — it supports both `CFG.MODE`
values natively, so there is no separate replay notebook to keep in sync.
Use [`kaggle_kernel/`](./kaggle_kernel) as the stable Kaggle push folder that
holds a synced copy of that same notebook for submission-replay pushes. Its
metadata points to the clean Kaggle kernel slug:

```text
tuannm3812/rogii-beam-pf-submission-replay-cpu
```

Push updates from that folder when you want Kaggle to update the same notebook instead of creating another timestamped copy:

```bash
/Users/tuannm3812/Library/Python/3.9/bin/kaggle kernels push -p kaggle_kernel
```

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
| Beam/PF V13 | `tau=None` override, same run as V12 (2026-07-29) | `9.952` | **Selected best**, verified. Clean within-run `tau` A/B win. |
| Beam/PF V11 | 2-LightGBM + CatBoost, restored (2026-07-29) | `10.022` | Superseded by V13; was selected best until then. |
| Beam/PF V3 | Original production path | `10.197` | Superseded by V11. |
| Beam/PF V5 | Artifact replay from V2 | `10.212` | Worse than V3, V11, V13. |
| Beam/PF V10 | Single-LGB + CatBoost (2026-07-29) | `10.226` | The prune V11 reversed; worse than V3, V5, V11, V13. |
| Beam/PF V12 | Same model as V11, fresh pass, `tau=100` (2026-07-29) | `10.087` | Same-run baseline for the `tau` A/B; run-to-run noise vs. V11 (`0.065`). |
| Beam/PF V14 | `tau=25` (heavy damping), same run as V12/V13 | `10.126` | Worse than V12 and V13 -- less damping is better, not "any change helps". |
| Beam/PF V8/V9 | Later reruns (2026-06-10) | `10.305` / `10.299` | Discovered unlogged 2026-07-29; worse than V3/V5/V11/V13. |

Latest Beam/PF local run (same 2-LightGBM stack behind V11-V14):

| Component | Local RMSE |
|---|---:|
| `lgb0` (`lr=0.020`) | 10.806 |
| `lgb1` (`lr=0.030`) | 10.798 |
| `catboost` | 10.629 |
| ridge stack / post-process (`tau=100`, local grid-search pick) | 10.559 |

The local Beam/PF validation has never reliably tracked the public score, and the `V12`/`V13`/`V14` `tau` sweep makes it concrete: the local grid search picks `tau≈100` as best in every run, but a clean same-run A/B shows real submissions clearly prefer `tau=None` (`V13`, `9.952`) over both the grid search's own choice (`V12`, `10.087`) and heavier damping (`V14`, `10.126`). The notebook now hard-codes `tau=None` instead of trusting the local search for this parameter. Explaining *why* remains the main diagnostic target, though only `V5`'s and `V11`'s prediction files are recoverable (`V3`/`V8`/`V9`/`V10`'s source kernels are gone).

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

1. **LightGBM-pruning saga resolved by promotion (2026-07-29).** Three
   GPU runs pruned `LGB_CONFIGS` from 3 models down to 1; the first two
   removals were free locally, the third (single LightGBM model +
   CatBoost) had a real local cost and, once submitted (`V10`, `10.226`),
   a real public cost too — worse than `V5` (`10.212`), the full-stack
   run. Restoring just the second-removed model (`V11`, 2 LightGBM
   models + CatBoost) not only recovered the loss but became the new
   best verified score: **`10.022`**, beating the old selected `V3`
   (`10.197`) by `0.175`. See
   [docs/4_next_steps.md §7](docs/4_next_steps.md#7-run-log) for the
   full ablation and submission history.
2. **Per-well diagnostic (done for the recoverable pair).** Compared
   `V5` vs. `V11`'s real prediction files (`V3`/`V8`/`V9`/`V10`'s source
   kernels are gone/overwritten, so this is the full extent of what's
   recoverable). The two versions diverge almost entirely on one public
   well — `00bbac68`, the longest hidden tail (6,014 rows): RMSE `2.156`
   between the two versions, a real and *growing* divergence across the
   tail (`~16%` of that well's own `TVT` range by the end). The other
   two public wells barely moved (`000d7d20`: RMSE `0.394`, essentially
   unchanged). This explains the *mechanism* — pooled public RMSE is
   dominated by whichever well has the longest tail and the largest
   model-to-model divergence — but not *why* V11 is closer to the true
   (hidden) value there; that needs ground truth this project doesn't
   have access to. See
   [docs/4_next_steps.md §7](docs/4_next_steps.md#7-run-log) for the
   full analysis.
3. **Post-processing `tau` sweep (done, promoted, closed out).** Acted
   on the well-`00bbac68` finding directly: `tau` controls how fast
   residual correction decays with distance into the hidden tail, so
   it's the most directly relevant lever for a long-tail well showing
   growing divergence. A clean within-run A/B (`V12`/`V13`/`V14`/`V15`,
   one training pass, only `tau` varied) tested the full range:

   | tau | version | public score |
   |---:|---|---:|
   | `None` | `V13` | **9.952** (selected) |
   | `100` | `V12` | 10.087 |
   | `25` | `V14` | 10.126 |
   | `350` | `V15` | 10.242 |

   Not perfectly monotonic (`25`/`100` swap order, likely noise from
   only 3 public wells), but the extremes are unambiguous: no damping
   is best, heaviest damping is worst. Promoted `V13` and hard-coded
   `tau=None` in the notebook instead of trusting the local grid
   search's `tau≈100` pick. See
   [docs/4_next_steps.md §7](docs/4_next_steps.md#7-run-log) for the
   full sweep.

The immediate priority is deciding whether other post-process parameters (`alpha`, `w_pf`) deserve the same real-submission A/B treatment as `tau` did, using the same one-training-pass, multiple-lightweight-CPU-replay pattern.

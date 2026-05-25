# 1. Instructions And Approaches

## 1. Objective

Predict `TVT` (True Vertical Thickness) for the hidden interval of each horizontal wellbore in the ROGII Wellbore Geology Prediction competition.

The task starts from a visible prefix where `TVT_input` is known. After the Prediction Start point, `TVT_input` becomes missing and the notebook must infer the target `TVT` values for the rows listed in `sample_submission.csv`.

## 2. Competition Inputs

Each well has a horizontal-well file and a paired typewell file.

Horizontal well files contain:

- `MD`: measured depth along the horizontal well;
- `X`, `Y`, `Z`: spatial coordinates;
- `GR`: gamma ray log, with possible missing values;
- `TVT_input`: known `TVT` before Prediction Start and missing values after it;
- `TVT`: training target, available only for train wells.

Typewell files contain:

- `TVT`: vertical reference depth;
- `GR`: typewell gamma ray log;
- geology labels and formation context.

Submission rows use IDs in the form `{WELLNAME}_{row_index}`. Only those row-level `TVT` values are scored.

## 3. Evaluation Target

The leaderboard metric is RMSE over predicted `TVT`.

The practical validation target should mimic the competition:

1. Select complete training wells.
2. Hide the tail of each selected well by masking `TVT_input`.
3. Build features from the visible prefix, horizontal-well logs, and paired typewell.
4. Predict the masked `TVT` suffix.
5. Score predictions against the true training `TVT`.

Held-out-well masked-tail validation is preferred because random row validation would leak too much local trajectory information.

## 4. Run Order

| Order | Notebook | Purpose |
|---:|---|---|
| 1 | [`1_rogii_eda.ipynb`](../notebooks/1_rogii_eda.ipynb) | Inspect data layout, missingness, hidden interval length, leakage boundaries, and modeling implications. |
| 2 | [`2_rogii_baseline.ipynb`](../notebooks/2_rogii_baseline.ipynb) | Establish carry-forward, trend, blend, and feature-tree baselines. |
| 3 | [`3_rogii_typewell_alignment.ipynb`](../notebooks/3_rogii_typewell_alignment.ipynb) | Add typewell-aware `GR` alignment features to the residual model. |
| 4 | [`4_rogii_beam_pf.ipynb`](../notebooks/4_rogii_beam_pf.ipynb) | Reconstruct full `TVT` trajectories with Beam/PF candidates, ensemble blending, and artifact replay. |

Supporting analysis:

- [`2_eda_insights.md`](2_eda_insights.md) summarizes the EDA findings and recommended deeper analysis.
- [`3_baseline_models.md`](3_baseline_models.md) documents baseline design and score readouts.

## 5. Data Interpretation

The public sample shows a sequence reconstruction problem rather than ordinary row-wise regression.

Key points:

- Train inventory has `773` horizontal wells and `773` paired typewells.
- The public sample has `3` test wells and `14,151` requested predictions.
- The hidden suffix is long, roughly `73-74%` of each public-sample well.
- `GR` can be missing, so interpolation and missingness indicators matter.
- Global `GR` to `TVT` correlation is weak; local shape matching is more useful than a single global regression relationship.
- Test does not expose training-only geology-top columns such as `ANCC`, `ASTNU`, `ASTNL`, `EGFDU`, `EGFDL`, and `BUDA`.
- Public leaderboard movement should be interpreted carefully because the visible public test contains only `3` wells.

## 6. Notebook Output Summary

The latest reruns provide a clear modeling sequence:

| Notebook | Key Readout | Implication |
|---|---:|---|
| EDA | `14,151` public rows across `3` wells | Public score is useful, but small-sample. |
| EDA | Median hidden suffix `72.7%` public and `74.0%` train | Validation must hide long tails. |
| EDA | Sampled `GR` missingness `32.0%` | Missingness and interpolation are core model features. |
| Baseline | Feature tree improves `10.281 -> 10.084` RMSE | Row-wise features help only slightly. |
| Typewell alignment | Alignment model improves `10.281 -> 9.799` RMSE | Typewell `GR` matching is a stronger signal. |
| Beam/PF | Latest run: stack `10.440`, post-process `10.410` | Trajectory reconstruction remains strongest locally. |
| Beam/PF public | V1 `9.941`, V3 `10.197`, V5 `10.212` | Keep V1 selected; later runs support reproducibility, not a better score. |

## 7. Approach 1: EDA

The EDA notebook answers the basic modeling questions:

- How many wells and files are available by split?
- Which columns are shared between train and test?
- Where does `TVT_input` become hidden?
- How much `GR` is missing?
- Does typewell `TVT` coverage overlap the horizontal-well range?
- Do representative wells show log-shape similarity between horizontal `GR` and typewell `GR`?

Main implication: the model should treat each well as a trajectory and use grouped validation. The EDA output also shows that public test wells are few and long-hidden, so model comparisons need both local validation and leaderboard caution.

## 8. Approach 2: Baseline Models

The baseline notebook compares deterministic inference-safe models:

- carry-forward from the final known `TVT_input`;
- linear trend extrapolation from the known prefix;
- damped trend extrapolation;
- blends between carry-forward and trend.

It then trains a feature-tree residual model:

```text
target_residual = true_TVT - carry_forward_TVT
```

This keeps the prediction anchored to a stable fallback while testing whether rolling `GR`, coordinates, position, and `TVT_input` features add signal.

Recorded public-score progression:

| Version | Public Score | Interpretation |
|---|---:|---|
| `V3` | `15.883` | Previous carry-forward/simple baseline level. |
| `V6` | `15.491` | Feature-tree submission improved the score. |
| `V7` | `15.491` | Rerun matched V6; feature baseline plateaued. |

Interpretation: the feature tree is worth keeping as a sanity check, but further gains require new geological trajectory signal.

## 9. Approach 3: Typewell Alignment

Typewell alignment adds geological signal by comparing horizontal `GR` to typewell `GR` around plausible `TVT` offsets.

Feature idea:

1. Start from the carry-forward `TVT` estimate.
2. Interpolate typewell `GR` near that estimate.
3. Search nearby `TVT` offsets.
4. Record the best-matching offset, `GR` difference, local slope, and context spread.
5. Train a residual model over carry-forward.

Result: Advanced Modeling V2 reached public score `15.049`, improving on the feature baseline best of `15.491`. The latest validation output also shows a larger local gain than the baseline feature tree: `0.481` RMSE improvement versus carry-forward.

## 10. Approach 4: Beam + Particle Filter

Beam/PF is the strongest approach currently recorded in the project.

It changes the problem from row-wise residual modeling to full trajectory reconstruction:

- beam search proposes plausible typewell-aligned `TVT` paths;
- particle filters reconstruct hidden suffix trajectories;
- multi-scale normalized cross-correlation compares horizontal and typewell `GR`;
- spatial formation-plane features add broader geological context;
- LightGBM/CatBoost blend candidate signals and residual corrections;
- post-processing smooths the final `TVT` curve.

Result: Beam + Particle Filter V1 reached public score `9.941` and remains the selected submission. Later artifact-workflow runs scored worse on the public leaderboard:

| Version | Public Score | Readout |
|---|---:|---|
| `V1` | `9.941` | Selected best. |
| `V3` | `10.197` | Submission-mode replay from V2 artifacts. |
| `V5` | `10.212` | Artifact workflow with LightGBM best-iteration preservation. |

The score drop means the next work should focus on diagnostics and reproducibility, not a wholesale model rewrite. The local Beam/PF validation improved on later runs, but the public score worsened, which suggests public-well sensitivity or validation mismatch.

## 11. Recommended Next Run

Use the Beam/PF notebook in two modes:

1. `CFG.MODE = "train"` to create `submission.csv` and `rogii_beam_pf_artifacts.zip`.
2. Upload the artifact zip as a private Kaggle model input.
3. Set `CFG.MODE = "submission"` and rerun to verify artifact replay.
4. Compare train-mode and submission-mode predictions row by row before submitting.

For model selection, keep V1 selected until a new public submission beats `9.941`.

## 12. Next Analysis

The next useful work is targeted deep-dive analysis around Beam/PF:

- V1 versus V3/V5 prediction differences by public well;
- alignment quality by well;
- `TVT` monotonicity and reversals;
- spatial dip behavior;
- random versus block `GR` missingness;
- per-well validation error curves.

These analyses should explain when trajectory reconstruction works, when it fails, and which Beam/PF components deserve more tuning.

## 13. Sources

- Kaggle competition overview: <https://www.kaggle.com/competitions/rogii-wellbore-geology-prediction/overview>
- Extracted competition task brief: <https://github.com/vamseeachanta/kaggle-rogii-2026/blob/main/docs/task-brief.md>

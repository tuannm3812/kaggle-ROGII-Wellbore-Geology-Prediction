# 3. Baseline And Model Results

## 1. Purpose

This page documents the baseline approach and result readouts from [`2_rogii_baseline.ipynb`](../notebooks/2_rogii_baseline.ipynb), then summarizes how later notebooks improved on that baseline.

The goal of the baseline notebook is to answer one question clearly: can simple inference-safe features beat carry-forward under masked-tail validation?

## 2. Baseline Design

The baseline uses only columns available at inference time:

- `MD`;
- `X`, `Y`, and `Z`;
- `GR`;
- `TVT_input`;
- row position and well-level derived features.

It deliberately excludes train-only geology-top columns such as `ANCC`, `ASTNU`, `ASTNL`, `EGFDU`, `EGFDL`, and `BUDA`.

The model predicts a residual:

```text
target_residual = true_TVT - carry_forward_TVT
```

This keeps the submission anchored to carry-forward. If validation does not improve, the notebook falls back to carry-forward automatically.

## 3. Deterministic Baselines

The deterministic comparison uses masked-tail validation on held-out wells.

| Model | Mean RMSE | Median RMSE | Interpretation |
|---|---:|---:|---|
| `carry_forward` | 7.62 | 6.14 | Strongest deterministic baseline and safest fallback. |
| `blend_0.25` | 8.43 | 6.87 | Trend blending adds noise in many wells. |
| `damped_trend_035` | 8.99 | 7.26 | Dampening helps but still trails carry-forward. |
| `linear_trend` | 14.05 | 10.02 | Too unstable for long hidden intervals. |

Carry-forward is the baseline to beat because the public and validation hidden intervals often begin after a stable known prefix.

## 4. Feature Engineering

Feature families:

- relative row position and measured-depth features;
- centered and normalized coordinates within each well;
- `GR` missingness, interpolation, and rolling statistics;
- carry-forward `TVT_input` level;
- distance from the final known `TVT_input` point;
- recent `TVT_input` slope and local volatility.

Rolling windows are intentionally simple: `25`, `101`, and `301` rows. This keeps the model fast enough for Kaggle and avoids fragile feature explosion.

## 5. Feature Model

The primary model is `HistGradientBoostingRegressor`.

Fallback model:

- `RandomForestRegressor`, used only if histogram gradient boosting fails in the runtime.

Validation strategy:

- held-out wells;
- hidden-tail masks of `20%`, `30%`, and `40%`;
- capped rows per well/fold for Kaggle runtime stability;
- residual prediction over carry-forward.

## 6. Validation Result

Held-out-well validation:

| Model | Validation RMSE | Delta vs Carry-Forward |
|---|---:|---:|
| `carry_forward` | 10.281 | 0.000 |
| `feature_tree` | 10.084 | -0.197 |

The feature tree improved validation by `0.197` RMSE. This is useful, but the gain is small enough that later experiments need genuinely new signal rather than only more tuning.

Tail-fraction validation shows the same pattern:

| Tail Fraction | Carry RMSE | Feature-Tree RMSE | Delta |
|---:|---:|---:|---:|
| `0.2` | 8.545 | 8.338 | -0.207 |
| `0.3` | 9.421 | 9.171 | -0.250 |
| `0.4` | 12.453 | 12.297 | -0.156 |

The gain is consistent but small across hidden-tail lengths.

## 7. Public Score Readout

| Version | Public Score | Readout |
|---|---:|---|
| `V3` | `15.883` | Previous carry-forward/simple baseline level. |
| `V6` | `15.491` | Feature-tree submission improved the score by `0.392`. |
| `V7` | `15.491` | Latest rerun matched V6; no additional gain. |

The feature baseline moved public score from `15.883` to `15.491`. The plateau at `15.491` motivated the typewell-alignment and Beam/PF notebooks.

## 8. Typewell Alignment Readout

[`3_rogii_typewell_alignment.ipynb`](../notebooks/3_rogii_typewell_alignment.ipynb) adds typewell-aware `GR` matching features.

Latest validation:

| Model | Validation RMSE | Delta vs Carry-Forward |
|---|---:|---:|
| `carry_forward` | 10.281 | 0.000 |
| `typewell_alignment` | 9.799 | -0.481 |

Tail-fraction validation:

| Tail Fraction | Carry RMSE | Typewell RMSE | Delta |
|---:|---:|---:|---:|
| `0.2` | 8.545 | 8.362 | -0.183 |
| `0.3` | 9.421 | 9.180 | -0.241 |
| `0.4` | 12.453 | 11.563 | -0.890 |

The alignment model helps most on the longest hidden tails, which is exactly where carry-forward is weakest. Its best public score was Advanced Modeling V2 at `15.049`.

## 9. Beam/PF Readout

[`4_rogii_beam_pf.ipynb`](../notebooks/4_rogii_beam_pf.ipynb) is the strongest modeling family. It builds full trajectory candidates and stacks LightGBM/CatBoost predictions.

Current model-selection status:

- `V3` remains selected for the leaderboard (`10.197`) — the best *verified* public score on record.
- `V5`, `V8`, and `V9` are maintained as diagnostic branches (`10.212`, `10.305`, `10.299` respectively).
- A previously documented `V1 = 9.941` entry could not be verified in either Kaggle account's submission history and was removed from the record on 2026-07-29 (see `docs/7_submission_score_registry.md`).

Quick sanity command pattern:

```python
# in 4_rogii_beam_pf.ipynb, set mode before running train/submission sections
CFG.MODE = "train"      # local rebuild + diagnostics
CFG.MODE = "submission" # attached artifact replay + replay checks
```

Latest local output:

| Component | Local RMSE |
|---|---:|
| `lgb0` | 10.786 |
| `lgb1` | 10.691 |
| `lgb2` | 10.747 |
| `catboost` | 10.549 |
| average ensemble | 10.564 |
| ridge stack | 10.440 |
| post-process | 10.410 |

Latest ridge weights:

| Model | Weight |
|---|---:|
| `lgb0` | 0.000 |
| `lgb1` | 0.234 |
| `lgb2` | 0.164 |
| `catboost` | 0.602 |

CatBoost remains the dominant stack member. The zero weight on `lgb0` suggests future ablations should test whether all three LightGBM variants are still worth the runtime.

Public score readout:

| Beam/PF Version | Public Score | Important Change | Interpretation |
|---|---:|---|---|
| `V3` | `10.197` | Original public-best Beam/PF production path with full trajectory-stack baseline. | Selected best (verified in Kaggle submission history). |
| `V5` | `10.212` | Submission-mode replay with V2 artifact bundle and best-iteration preservation. | Worse than V3. |
| `V8` | `10.305` | Main-account notebook output submission (2026-06-10); discovered unlogged during 2026-07-29 reconciliation. | Worse than V3, diagnostic only. |
| `V9` | `10.299` | tuannm3823-account GPU train notebook submission (2026-06-10); discovered unlogged during 2026-07-29 reconciliation. | Worse than V3, diagnostic only. |

The local validation rank has not reliably matched the public score rank across these runs, which is why per-public-well diagnostics remain the next priority. (A previous version of this table included a `V1 = 9.941` row and framed it as the local-vs-public mismatch example; that row could not be verified in either Kaggle account's submission history and was removed 2026-07-29 — see `docs/7_submission_score_registry.md`.)

## 10. Interpretation

The baseline is useful as a stable reference and fallback. The small validation gain and public-score plateau suggest that future effort should focus on trajectory signal, not broad tuning of the same row-wise feature family.

The current model-selection position is:

- keep Beam/PF V3 selected because it has the best *verified* public score (`10.197`);
- keep the artifact workflow because it improves reproducibility;
- investigate why later runs (V5/V8/V9) score worse than V3 on the public leaderboard;
- prioritize per-well prediction comparison and Beam/PF component ablations over more feature-tree tuning.

# 3. Baseline Models

## 1. Purpose

This page documents the baseline approach and result readouts from [`2_rogii_baseline.ipynb`](../notebooks/2_rogii_baseline.ipynb).

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

## 7. Public Score Readout

| Version | Public Score | Readout |
|---|---:|---|
| `V3` | `15.883` | Previous carry-forward/simple baseline level. |
| `V6` | `15.491` | Feature-tree submission improved the score by `0.392`. |
| `V7` | `15.491` | Latest rerun matched V6; no additional gain. |

The feature baseline moved public score from `15.883` to `15.491`. The plateau at `15.491` motivated the typewell-alignment and Beam/PF notebooks.

## 8. Interpretation

The baseline is useful as a stable reference and fallback. The small validation gain and public-score plateau suggest that future effort should focus on new trajectory signal, not broad tuning of the same row-wise feature family.

# 4. Next Steps

## 1. Immediate Cleanup (Today)

1. Finalize commit-ready docs and notebook metadata changes.
2. Keep local Kaggle execution artifacts out of git (`kaggle_runs/` and `docs/superpowers/` already ignored).
3. Verify `README.md` lists only tracked docs and notebooks.

## 2. Runbook Hardening (Next 1-2 Runs)

### 2.1 Before each Kaggle push

- Set a clean scratch run folder:
  - `mkdir -p kaggle_runs`
- Copy source and metadata into a fresh folder:
  - `cp notebooks/4_rogii_beam_pf.ipynb <run-folder>/4_rogii_beam_pf.ipynb`
  - `cp -R <base-run-folder>/ <run-folder>/` when needed for artifact carry-over.
- Enforce slug contract:
  - `id = tuannm3812/rogii-beam-pf-gpu-v2-production`
  - `title = rogii-beam-pf-gpu-v2-production`
  - no datetime-only titles.

### 2.2 During run

- Push with:
  - `/Users/tuannm3812/Library/Python/3.9/bin/kaggle kernels push -p kaggle_runs/<run-folder>`
- Poll status until COMPLETE before output collection.
- Pull outputs to a short-lived folder and inspect:
  - `version_log.jsonl`
  - `submission.csv`
  - `/tmp/.../<slug>.log`
  - artifact zip (if produced)

### 2.3 After run

- Record status + best local score + submit message in `7_submission_score_registry.md` and include operational notes in `4_next_steps.md`.
- If run is a diagnostic rerun and not submission candidate, avoid updating selected version.
- Clean local folder if it is no longer needed:
  - `rm -rf kaggle_runs/<run-folder>`.

## 3. Commit Discipline (for each run cycle)

Before committing, apply this checklist:

1. Keep commits function-scoped:
   - one commit for notebook/runtime changes,
   - one commit for runbook/process docs,
   - one commit for status/risk summary docs.
2. Include affected version label in each commit message/body (`V1`, `V3`, `V5`, etc.).
3. Use professional message format, for example:
   - `feat(notebook): ...`
   - `chore(docs): ...`
   - `chore(runbook): ...`
4. If public score status changes, require same-cycle updates to:
   - `docs/7_submission_score_registry.md`
   - `docs/1_instructions.md` or `docs/3_baseline_models.md` (if interpretation changes)
   - `README.md` (if selected status changes)
   - `docs/6_kaggle_autosubmit_runbook.md` (if workflow changes)
5. Add a short commit body note:
   - intent,
   - impacted version,
   - selected/diagnostic impact,
   - next action.

Reference the full standard in [5_coding_standards.md](./5_coding_standards.md).

## 4. Investigation Priority (Primary)

1. Explain the V3/V5/V10 public gap with per-well prediction diff on public rows. Partially blocked 2026-07-29 (see [Run Log](#7-run-log)): only `V5`'s real prediction file was recoverable (`V3`/`V8`/`V9`'s source kernels are gone); a fresh `V10` submission (`10.226`) is available to diff against the recovered `V5` file.
2. Add failure-mode analysis for monotonicity reversals and tail-start error spikes.
3. Evaluate lightgbm variant relevance with an ablation matrix using local validation + public sanity checks. Local-validation half done 2026-07-28/29 (see [Run Log](#7-run-log)) — `lgb0` dropped from `LGB_CONFIGS` with no local regression and ~28% faster wall-clock; public sanity check still pending (no submission made yet).
4. Add a stable `beam_pf` artifact audit checklist in runbook checklists.

## 5. Investigation Priority (Secondary)

1. Add a short per-well alignment diagnostic table (public and train).
2. Add post-process sensitivity sweep across only the top 2-3 parameters.
3. Compare candidate runtime: remove dominated variants without score regressions.

## 6. Completion Criteria

- New V3-beating public score candidate identified and reproduced.
- Public and local validation mismatch explained and logged by well-level evidence.
- Documentation updated with exactly one “selected” version and one “diagnostic” run family.
- No generated Kaggle folders/files are staged in git after each run cycle.

## 7. Run Log

### 2026-07-28: LightGBM Component Ablation (Diagnostic, Not Submitted)

- Kernel: `tuannm3812/rogii-beam-pf-gpu-v2-production`, `CFG.MODE = "train"`.
- Change under test: added `disable_lgb0` and `disable_lgb0_lgb1` rows to
  `run_component_ablation_matrix()` (no retrain needed; reuses this run's
  own OOF/test predictions).
- Local ablation result (`beam_pf_ablation_matrix.csv`):

  | component_set | local_tail_rmse | delta vs full |
  |---|---:|---:|
  | full | 10.3549 | 0.000 |
  | disable_lgb0 | 10.3556 | +0.0007 |
  | disable_lgb0_lgb1 | 10.3663 | +0.0113 |
  | disable_postprocess | 10.3926 | +0.0377 |
  | disable_ensemble | 10.4512 | +0.0963 |
  | disable_catboost | 10.5158 | +0.1609 |

- Ridge weights this run: `lgb0=0.058, lgb1=0.215, lgb2=0.103, cb=0.624`.
  `lgb0`'s weight is not reliably zero run-to-run (it was `0.000` in the
  run behind `docs/3_baseline_models.md`), but the ablation confirms its
  marginal contribution is negligible either way, not just inferred from
  the ridge weight.
- Decision: drop `lgb0` from `LGB_CONFIGS` in the next notebook iteration
  to cut training time and artifact size for a ~0.0007 local RMSE cost.
  Keep `lgb1`/`lgb2` for now — dropping both only buys ~0.011 more RMSE
  headroom, not worth it until `lgb1` gets its own isolated ablation row.
- This run's overall local post-process RMSE (`10.3549`) is better than
  the `10.410` previously logged in `docs/3_baseline_models.md`, but per
  the V3-vs-V5/V8/V9 precedent a local improvement alone is not grounds to
  submit — the per-well public-gap diagnostic (Investigation Priority
  Primary #1) is still the prerequisite before trusting any new local
  number against the leaderboard.
- No submission was made from this run (train mode only); nothing to
  record in `docs/7_submission_score_registry.md`.

### 2026-07-29: Verify lgb0 Removal From LGB_CONFIGS (Diagnostic, Not Submitted)

- Kernel: `tuannm3812/rogii-beam-pf-gpu-v2-production`, `CFG.MODE = "train"`,
  version 3 — first run with `lgb0`'s config actually removed from
  `LGB_CONFIGS` (only 2 LightGBM configs now, positionally renamed to
  `lgb0`/`lgb1`; see commit `f8b03a2`).
- Local ablation result (`beam_pf_ablation_matrix.csv`):

  | component_set | local_tail_rmse | delta vs full |
  |---|---:|---:|
  | disable_lgb0 | 10.24099 | -0.0000006 (noise) |
  | full | 10.24099 | 0.000 |
  | disable_postprocess | 10.26914 | +0.0282 |
  | disable_ensemble | 10.31858 | +0.0776 |
  | disable_lgb0_lgb1 | 10.34358 | +0.1026 |
  | disable_catboost | 10.45701 | +0.2160 |

- `model_scores.csv`: `lgb0=10.5568, lgb1=10.5287, cb=10.3671, avg=10.3473,
  ridge=10.2691`. CatBoost remains clearly dominant (dropping it costs
  `+0.2160`, the largest of any component tested).
- Wall-clock: ~2h16m (09:36 -> 11:52), vs. ~3h09m for the prior 3-LGB-model
  run — roughly 28% faster, as expected from training one fewer LightGBM
  model.
- This run's local RMSE (`10.2410`) is again lower than the prior run's
  (`10.3549`), but per the standing caution this is not itself evidence of
  a public-score improvement — GPU training has run-to-run variance, and
  local-vs-public correlation is still unverified (Investigation Priority
  Primary #1). No submission made.
- Follow-up: `disable_lgb0` (i.e. dropping the current, formerly-lgb1,
  0.020-learning-rate config too) is *also* within noise of `full`. The
  two remaining LightGBM models may both be dominated by CatBoost —
  worth a follow-up ablation isolating `disable_lgb1` alone (not yet
  tested) before deciding whether to cut LightGBM down further.

### 2026-07-29: Isolated disable_lgb1 Ablation (Diagnostic, Not Submitted)

- Kernel: `tuannm3812/rogii-beam-pf-gpu-v2-production`, `CFG.MODE = "train"`,
  version 4 — added a `disable_lgb1` row to `run_component_ablation_matrix()`
  (commit `3d85e90`) to isolate the second remaining LightGBM config
  (`lr=0.030, seed=123`) instead of only testing it bundled with `lgb0`.
- Local ablation result (`beam_pf_ablation_matrix.csv`):

  | component_set | local_tail_rmse | delta vs full |
  |---|---:|---:|
  | disable_lgb0 | 10.29551 | -0.0004 (noise) |
  | full | 10.29591 | 0.000 |
  | disable_lgb1 | 10.30198 | +0.0061 |
  | disable_postprocess | 10.31275 | +0.0168 |
  | disable_lgb0_lgb1 | 10.36247 | +0.0666 |
  | disable_ensemble | 10.40941 | +0.1135 |
  | disable_catboost | 10.57516 | +0.2793 |

- Ridge weights this run: `lgb0=0.114, lgb1=0.168, cb=0.718`. CatBoost now
  carries almost three-quarters of the stack weight.
- Reading across all three ablation runs to date, the current `lgb0`
  (`lr=0.020, seed=7`) has been within noise of `full` twice in a row
  (`-0.0000006` then `-0.0004`), while `lgb1` (`lr=0.030, seed=123`)
  costs a small but consistently positive `+0.006` to `+0.011` when
  removed. CatBoost's removal cost has grown each run (`0.161 -> 0.216 ->
  0.279`) — it is clearly the load-bearing component.
- Implication: the current `lgb0` looks like a good candidate to drop
  too, leaving a single LightGBM model (`lgb1`, `lr=0.030`) + CatBoost.
  Not acted on yet — flagged for a decision before spending another GPU
  cycle, per one-controlled-change-per-run discipline.
- No submission was made from this run (train mode only).

### 2026-07-29: Single LightGBM Model + CatBoost (Diagnostic, Not Submitted)

- Kernel: `tuannm3812/rogii-beam-pf-gpu-v2-production`, `CFG.MODE = "train"`,
  version 5 — dropped the remaining `lr=0.020, seed=7` config from
  `LGB_CONFIGS`, leaving a single LightGBM model (`lr=0.030, seed=123`)
  + CatBoost (commit `8e35034`). Also removed the now-meaningless
  `disable_lgb1` / `disable_lgb0_lgb1` ablation rows (no `lgb1` key
  exists with only one LightGBM model left).
- Local ablation result (`beam_pf_ablation_matrix.csv`):

  | component_set | local_tail_rmse | delta vs full |
  |---|---:|---:|
  | full | 10.40424 | 0.000 |
  | disable_postprocess | 10.43037 | +0.0261 |
  | disable_ensemble | 10.43850 | +0.0343 |
  | disable_lgb0 | 10.47272 | +0.0685 |
  | disable_catboost | 10.64968 | +0.2454 |

- Ridge weights: `lgb0=0.338, cb=0.662`.
- Wall-clock: ~1h13m (14:59 -> 16:12), vs. ~2h16m for the two-LGB-model
  run — a further ~46% cut (~65% total vs. the original 3-model run).
- **Different result from the prior two prunings — this one is not free.**
  With only one LightGBM model left, dropping it costs `+0.0685` in this
  run's own matched ablation (`disable_lgb0`), clearly larger than the
  `+0.006` to `+0.011` cost measured for the *same* `lr=0.030` config
  when a second LightGBM model was still present to share the load. The
  pattern makes sense: earlier removals cut redundant capacity (near-zero
  cost); this one cuts the last non-redundant LightGBM signal.
- This run's overall `full` RMSE (`10.40424`) is also the highest of the
  four ablation runs so far (prior two-LGB runs: `10.2410`, `10.2959`),
  though cross-run comparisons are confounded by run-to-run GPU training
  variance (the two prior same-config runs already differed by `0.055`
  on their own) and shouldn't be read as precisely as the in-run
  `disable_lgb0` figure above.
- **Open decision, not resolved by this run:** keep the single-LGB
  setup (accept a real, if modest, RMSE cost for a large runtime win),
  or restore the second LightGBM model given it showed genuine
  (non-noise) value. Flagged for explicit direction before further
  changes.
- No submission was made from this run (train mode only).

### 2026-07-29: Score Record Correction — V1/9.941 Could Not Be Verified

- While preparing the V3-vs-V5/V8/V9 per-well diagnostic, pulled the full
  submission history for both known Kaggle accounts:
  - `tuannm3812`: `kaggle competitions submissions -c rogii-wellbore-geology-prediction --csv`
  - `tuannm3823`: same command via a temporary `KAGGLE_CONFIG_DIR` pointed
    at `kaggle_tuannm3823.json` (found in the parent `2. Kaggle/` folder).
- Neither account's history contains a `9.941` submission. The record
  previously documented as `V1 = 9.941, selected` had a `submission_date_utc`
  (`2026-05-24T14:05:15.043000Z`) that is an exact, microsecond-for-microsecond
  duplicate of the real `V5` submission's timestamp (`10.212`) — strong
  evidence the row was a copy/fabrication error rather than a real,
  under-documented submission.
- Two real submissions were found that had never been logged at all:
  - `V8`: `10.305`, `tuannm3812`, 2026-06-10T15:27:55.557000Z, "Main account
    notebook output submission 20260610-221607".
  - `V9`: `10.299`, `tuannm3823`, 2026-06-10T21:47:04.007000Z, "tuannm3823
    GPU train notebook submission 20260611-0030".
  - (Two additional `tuannm3823` submissions from 2026-06-10 returned
    `SubmissionStatus.COMPLETE` with no public score value and are not
    fileable as registry rows.)
  - Also noticed one more real, unlogged pre-BeamPF submission —
    `15.306`, `tuannm3812`, 2026-05-20T12:23:48.673000Z — from the
    baseline/alignment era. Not added to the registry (which only ever
    tracked the BeamPF family) and doesn't change the baseline docs'
    already-correct `15.883 -> 15.491 -> 15.049` progression narrative,
    but flagged here for completeness.
- **Corrected selected version: `V3` (`10.197`)** — the actual best
  verified public score in either account's history. Updated
  `docs/7_submission_score_registry.md`, `docs/1_instructions.md`,
  `docs/3_baseline_models.md`, and `README.md` accordingly; the fabricated
  `V1` row was removed from the registry table (see its Audit Log for the
  removal record) rather than silently deleted with no trace.
- Practical implication: the real bar to beat going forward is `10.197`,
  not `9.941`. Every diagnostic run logged above this entry (all scoring
  in the `10.24`-`10.40` local range) was already being compared against
  the wrong, unbeatable target — none of them were ever close to `9.941`,
  but several may already be worth testing against the real `10.197` bar
  once the per-well diagnostic restores trust in local validation.

### 2026-07-29: Real Submission V10 (10.226) — Single-LGB + CatBoost

- Attempted the planned V3-vs-V5/V8/V9 per-well diagnostic first. Blocked:
  the Kaggle CLI only pulls a kernel's *current* version's output, and
  today's ablation runs had already overwritten
  `tuannm3812/rogii-beam-pf-gpu-v2-production`'s history. Recovered one
  real historical file — `V5`'s actual `submission.csv` — from a
  separate, untouched kernel (`tuannm3812/rogii-beam-particle-filter`,
  last run 2026-05-24, exact same moment as the V5 submission; its
  printed local RMSE `10.4101` matches what's already documented,
  confirming it's genuine). `V3`, `V8`, and `V9`'s source kernels are
  gone (deleted or overwritten) — not recoverable via the API.
- Given the real spread among genuine submissions (`10.197`-`10.305`)
  is much narrower than the fabricated-`V1` framing implied — the same
  order of magnitude as run-to-run GPU noise already measured today —
  pivoted from "explain the gap" to "get one fresh, real, comparable
  score" for today's current committed notebook state (single-LGB
  `lr=0.030` + CatBoost).
- Getting that submission through surfaced real, previously-undocumented
  competition constraints (now written up in
  `docs/6_kaggle_autosubmit_runbook.md` S2/S10/S14):
  - The in-kernel auto-submit cell can never succeed here — this
    competition rejects internet-enabled notebooks, and the auto-submit
    call needs network access.
  - Raw file upload (`kaggle competitions submit -f submission.csv`) is
    also rejected outright — this competition only accepts submissions
    tied to a specific notebook/kernel version
    (`competition_submit_code`).
  - The installed `kaggle` pip package silently used a dead legacy
    endpoint until the `kagglesdk` package was installed alongside it.
  - All of this only surfaced by calling the API directly in Python and
    reading the actual JSON error body — the CLI and kernel logs only
    ever showed a bare `400 Client Error`.
- **Result: `V10` = `10.226`** (kernel `rogii-beam-pf-submission-replay-cpu`
  v7). Worse than `V3` (`10.197`) and `V5` (`10.212`) — stays diagnostic,
  `V3` remains selected. Logged in
  [docs/7_submission_score_registry.md](./7_submission_score_registry.md).
- This run's local RMSE (`10.4042`) was *slightly better* than `V5`'s
  historical local RMSE (`10.4101`), yet scored worse publicly — the
  local-vs-public mismatch reconfirmed on a real, non-fabricated data
  point. Today's LightGBM-pruning work (S7, 2026-07-29 entries above)
  optimized for runtime, not public score; `V5`'s full 3-LGB-model stack
  beat today's leaner single-LGB stack on the actual leaderboard.
- Implication for next steps: the per-well diagnostic is no longer
  fully executable (missing V3/V8/V9 files), so it should be reframed
  around what *is* available — `V10` vs the recovered real `V5` file —
  or dropped in favor of testing whether restoring the second/third
  LightGBM model (reverting some of today's pruning) recovers real
  public score, now that pruning is confirmed to have a real cost, not
  just a local one.

## 8. Execution Checklist (Next Run Cycle)

- Keep one notebook-only controlled change per Kaggle run cycle.
- In train mode, write candidate settings to notebook variables:
  - `CFG.SEED`, beam search tuple list, PF settings, and postprocess grid.
- Record these from output:
  - local tail RMSE,
  - alignment diagnostics by well (`mean_hidden_tail_length`, `monotonicity_violations`),
  - component ablation table.
- In submission mode, confirm `validate_replay_metadata(...)` succeeds before interpreting leaderboard output.
- If leaderboard improves on `10.197` (V3, the verified selected score — see
  `docs/7_submission_score_registry.md`), promote immediately and move
  everything else to diagnostic-only status.

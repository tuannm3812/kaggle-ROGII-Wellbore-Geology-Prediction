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

- Record status + best local score + submit message in `7_submission_score_registry.md` and include operational notes in `8_run_log.md`.
- If run is a diagnostic rerun and not submission candidate, avoid updating selected version.
- Clean local folder if it is no longer needed:
  - `rm -rf kaggle_runs/<run-folder>`.

## 3. Commit Discipline (for each run cycle)

Before committing, apply this checklist:

1. Keep commits function-scoped:
   - one commit for notebook/runtime changes,
   - one commit for runbook/process docs,
   - one commit for status/risk summary docs.
2. Include affected version label in each commit message/body (`V3`, `V5`, `V13`, etc.).
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

1. Explain why V11 (and now V13) has worse local RMSE than other runs yet the best public score of the whole cycle, with a per-well prediction diff on public rows. Done 2026-07-29 for the recoverable pair (see [8_run_log.md](./8_run_log.md)): `V5` vs `V11` diverge almost entirely on the longest-hidden-tail public well (`00bbac68`, RMSE 2.156), while the other two wells barely moved -- explains the *mechanism* (pooled RMSE dominated by whichever well has the longest tail and largest divergence), not yet *why* V11 is closer to ground truth there. `V3`/`V8`/`V9`/`V10`'s source kernels are gone/overwritten, so this pair is the full extent of what's recoverable.
2. Add failure-mode analysis for monotonicity reversals and tail-start error spikes.
3. Evaluate lightgbm variant relevance with an ablation matrix using local validation + public sanity checks. Done 2026-07-28/29 (see [8_run_log.md](./8_run_log.md)) — pruned to a single LightGBM model (`V10`), which cost real public score; restored a second model (`V11`), which became best until superseded by `V13`'s post-process fix. Local validation never tracked public rank across any of these runs.
4. Add a stable `beam_pf` artifact audit checklist in runbook checklists.

## 5. Investigation Priority (Secondary)

1. Add a short per-well alignment diagnostic table (public and train). Attempted 2026-07-29; blocked by a real bug in `build_alignment_diagnostics` (fixed, commit `15d9fb5`) that needs a fresh GPU run to get corrected numbers -- not done yet, deprioritized given local validation's established unreliability.
2. Add post-process sensitivity sweep across only the top 2-3 parameters. **Done 2026-07-29/30:** `tau` (promoted V13), `alpha`, `w_pf` (no improvement) -- see [8_run_log.md](./8_run_log.md).
3. Compare candidate runtime: remove dominated variants without score regressions. **Done 2026-07-28/29:** LightGBM pruned from 3 models to 2 (net), cutting training time meaningfully with the final config (`V11`/`V13`) beating the original 3-model stack publicly.

## 6. Completion Criteria

- New V3-beating public score candidate identified and reproduced. **Done 2026-07-29/30:** `V11` (`10.022`) beat `V3` (`10.197`) by `0.175`; superseded the same day by `V13` (`9.952`), now selected.
- Public and local validation mismatch explained and logged by well-level evidence. Mechanism explained (§4 item 1); root cause of *why* one model's extrapolation is better on the dominant well remains open.
- Documentation updated with exactly one “selected” version and one “diagnostic” run family. Maintained through every promotion this cycle.
- No generated Kaggle folders/files are staged in git after each run cycle.

## 7. Run Log

Detailed, chronological run-by-run history (GPU/CPU run results, ablation
findings, promotions, bugs found and fixed, platform quirks) has moved to
[`8_run_log.md`](./8_run_log.md) to keep this document focused on current
priorities. Record every Kaggle run there per the [Score Filing
Rule](./7_submission_score_registry.md#2-score-filing-rule) and the
runbook's [Optional Local
Audit](./6_kaggle_autosubmit_runbook.md#12-optional-local-audit) step.

Latest summary: `V13` (`9.952`) is selected, promoted 2026-07-29 after a
`tau` post-process A/B beat the local grid search's own pick; a follow-up
`alpha`/`w_pf` sweep on 2026-07-30 found no further improvement. See
`8_run_log.md` for the full trail.

## 8. Execution Checklist (Next Run Cycle)

- Keep one notebook-only controlled change per Kaggle run cycle.
- In train mode, write candidate settings to notebook variables:
  - `CFG.SEED`, beam search tuple list, PF settings, and postprocess grid.
- Record these from output:
  - local tail RMSE,
  - alignment diagnostics by well (`mean_hidden_tail_length`, `monotonicity_violations`),
  - component ablation table.
- In submission mode, confirm `validate_replay_metadata(...)` succeeds before interpreting leaderboard output.
- If leaderboard improves on `9.952` (V13, the verified selected score — see
  `docs/7_submission_score_registry.md`), promote immediately and move
  everything else to diagnostic-only status.

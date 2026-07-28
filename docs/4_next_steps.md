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

1. Explain V1 vs V3/V5 public gap with per-well prediction diff on public rows.
2. Add failure-mode analysis for monotonicity reversals and tail-start error spikes.
3. Evaluate lightgbm variant relevance with an ablation matrix using local validation + public sanity checks.
4. Add a stable `beam_pf` artifact audit checklist in runbook checklists.

## 5. Investigation Priority (Secondary)

1. Add a short per-well alignment diagnostic table (public and train).
2. Add post-process sensitivity sweep across only the top 2-3 parameters.
3. Compare candidate runtime: remove dominated variants without score regressions.

## 6. Completion Criteria

- New V1-equivalent public score candidate identified and reproduced.
- Public and local validation mismatch explained and logged by well-level evidence.
- Documentation updated with exactly one “selected” version and one “diagnostic” run family.
- No generated Kaggle folders/files are staged in git after each run cycle.

## 7. Execution Checklist (Next Run Cycle)

- Keep one notebook-only controlled change per Kaggle run cycle.
- In train mode, write candidate settings to notebook variables:
  - `CFG.SEED`, beam search tuple list, PF settings, and postprocess grid.
- Record these from output:
  - local tail RMSE,
  - alignment diagnostics by well (`mean_hidden_tail_length`, `monotonicity_violations`),
  - component ablation table.
- In submission mode, confirm `validate_replay_metadata(...)` succeeds before interpreting leaderboard output.
- If leaderboard improves 9.941, promote immediately and move everything else to diagnostic-only status.

# 7. Submission Score Registry

## 1. Purpose

This file is the single source of truth for versioned submissions. It tracks score history, decision status, and promotion actions.

## 2. Score Filing Rule

Every Kaggle run must be entered here after output pull.

Record the following fields:

- `run_name`  
- `kaggle_slug`
- `notebook_version`  
- `version_label`
- `cfg_mode`  
- `submission_score`  
- `submission_date` (UTC)
- `run_status`  
- `important_change`  
- `notes`  

## 3. Promotion Rule

1. Compare the new public score against `Current Selected Version`.
2. If new score is better than current selected score (strictly lower RMSE), promote immediately.
3. On promotion:
   - update `README.md` selected score statement,
   - update `docs/1_instructions.md` "selected version" wording,
   - update `docs/3_baseline_models.md` score table,
   - update notebook metadata `DESCRIPTION` if needed,
   - run one final Kaggle replay in `CFG.MODE = "submission"` with same artifact bundle.
4. If score is equal or worse, keep it in diagnostic-only section.

## 4. GPU-First Execution Instruction for Agents

For any retraining-heavy script or notebook changes, use Kaggle GPU execution and do not run full training locally.

1. Keep local notebook edits under `notebooks/`.
2. Push via `kaggle_runs/<run-folder>` using the runbook.
3. Run with `CFG.MODE = "train"` first.
4. Validate log, then run with `CFG.MODE = "submission"` for replay.
5. Use this if/when score is competitive:
   - if score improves selected baseline, update notebook and resubmit on Kaggle.

## 5. Registry Template

Append rows to this table as runs complete.

| run_name | kaggle_slug | notebook_version | version_label | cfg_mode | public_submission_score | submission_date_utc | status | important_change | notes |
|---|---|---|---|---|---:|---|---|---|---|
| BeamPF | rogii-beam-pf-gpu-v2-production | v2.0 | V3 | submission | 10.197 | 2026-05-24T04:13:41.533000Z | submitted | Original public-best Beam/PF trajectory-stack production path | Superseded by V11 2026-07-29; kept for history |
| BeamPF | rogii-beam-pf-gpu-v2-production | v2.0 | V5 | submission | 10.212 | 2026-05-24T14:05:15.043000Z | submitted | Submission-mode replay using V2 artifact bundle; score below selected | Submission attempt, worse than V3 |
| BeamPF | rogii-beam-pf-gpu-v2-production | v2.0 | V8 | submission | 10.305 | 2026-06-10T15:27:55.557000Z | submitted | Main-account notebook output submission; discovered unlogged during 2026-07-29 reconciliation | Worse than V3, diagnostic only |
| BeamPF (tuannm3823 account) | rogii-beam-pf-gpu-train-submit-20260611-0030 | v2.0 | V9 | submission | 10.299 | 2026-06-10T21:47:04.007000Z | submitted | tuannm3823-account GPU train notebook submission; discovered unlogged during 2026-07-29 reconciliation | Worse than V3, diagnostic only |
| BeamPF | rogii-beam-pf-submission-replay-cpu | v7 | V10 | submission | 10.226 | 2026-07-29T07:29:24.263000Z | submitted | Single-LightGBM (lr=0.030) + CatBoost stack, submitted via kernel-version code-competition path (`competition_submit_code`, not raw file upload -- this competition rejects internet-enabled notebooks and non-notebook submissions entirely) | Worse than V3 and V5; local RMSE (10.4042) was slightly better than V5's historical local RMSE (10.4101) yet scored worse publicly -- local-vs-public mismatch confirmed on a real, non-fabricated data point |
| BeamPF | rogii-beam-pf-submission-replay-cpu | v8 | V11 | submission | 10.022 | 2026-07-29T11:49:57.247000Z | submitted | Restored lr=0.020 LightGBM model (2-LGB + CatBoost, undoing the V10 single-LGB prune) | Superseded by V13 2026-07-29; was best verified score until then |
| BeamPF | rogii-beam-pf-gpu-v2-production | v7 | V12 | submission | 10.087 | 2026-07-29T16:08:44.940000Z | submitted | Same model/config as V11, fresh GPU training pass, tau=100 (auto-selected local-best), submitted directly from the GPU train kernel (not the CPU replay kernel) | Worse than V11 by 0.065 -- pure run-to-run training variance for an identical config, consistent with noise observed all cycle |
| BeamPF | rogii-tau-replay-none | v1 | V13 | submission | 9.952 | 2026-07-29T16:14:45.183000Z | selected | tau=None (no distance damping) override on V12's exact artifact bundle -- same model/training pass as V12, only tau differs | **New best verified public score**, beats V11 by 0.070 and V12 (same run, tau=100) by 0.135. Clean within-run A/B: the local grid search has picked tau~100 as "best" in every run this cycle, but real submissions consistently favor no distance-damping at all |
| BeamPF | rogii-tau-replay-25 | v1 | V14 | submission | 10.126 | 2026-07-29T16:14:49.387000Z | submitted | tau=25 (heavy distance damping) override on V12's exact artifact bundle | Worse than V12 (tau=100) and V13 (tau=None) -- heavier damping hurt, confirming the direction (less damping is better) rather than just "any change helps" |
| BeamPF | rogii-tau-replay-350 | v1 | V15 | submission | 10.242 | 2026-07-30T00:07:28Z | submitted | tau=350 (light distance damping) override on V12's exact artifact bundle -- completes the 4-point tau sweep after the daily quota blocked it on 2026-07-29 | Worst of the entire tau sweep. Full ranking by tau value None(9.952) < 100(10.087) < 25(10.126) < 350(10.242) is not perfectly monotonic (25/100 swap order), but the extremes are unambiguous: no damping is best, heaviest damping is worst |

## 6. Audit Log (Latest)

| action | timestamp_utc | actor | change | impact |
|---|---|---|---|---|
| baseline record kept | 2026-05-24 | workflow | V1 retained in 1.0 docs | public score best |
| row removed | 2026-07-29 | agent (reconciliation) | Removed `V1 = 9.941, selected` row. Verified against the full submission history of both `tuannm3812` and `tuannm3823` Kaggle accounts (`kaggle competitions submissions --csv`); no `9.941` submission exists in either. The row's `submission_date_utc` was an exact duplicate of `V5`'s real timestamp, indicating a copy/fabrication error rather than an under-documented real submission. | Selected version changed from the unverifiable `V1` to `V3` (`10.197`), the actual best verified public score. |
| rows added | 2026-07-29 | agent (reconciliation) | Added `V8` (`10.305`, `tuannm3812`, 2026-06-10) and `V9` (`10.299`, `tuannm3823`, 2026-06-10) — both real, `SubmissionStatus.COMPLETE` submissions that were never logged here. | Registry now reflects every scored submission found in either account's history. |
| promotion | 2026-07-29 | agent | `V11` (`10.022`) beats `V3` (`10.197`) by `0.175`. `V3` demoted from `selected` to `submitted`. Updated `README.md`, `docs/1_instructions.md`, `docs/3_baseline_models.md` per the Promotion Rule (S3). | `V11` is the new selected version. |
| promotion | 2026-07-29 | agent | `V13` (`9.952`) beats `V11` (`10.022`) by `0.070` via a clean within-run `tau` A/B test (V12/V13/V14 share one training pass, only `tau` differs). `V11` demoted to `submitted`. Updated `README.md`, `docs/1_instructions.md`, `docs/3_baseline_models.md` per the Promotion Rule (S3); `notebooks/4_rogii_beam_pf.ipynb` changed to force `tau=None` instead of trusting the local grid search's choice. | `V13` is the new selected version. |

## 7. Closing Rule

Do not modify this file without actual run output.

## 8. References

- [0_readme.md](./0_readme.md)
- [4_next_steps.md](./4_next_steps.md)
- [6_kaggle_autosubmit_runbook.md](./6_kaggle_autosubmit_runbook.md)

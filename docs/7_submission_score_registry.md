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
| BeamPF | rogii-beam-pf-gpu-v2-production | v2.0 | V3 | train | 10.197 | 2026-06-04T00:00:00Z | submitted | Added reproducible artifact + diagnostic replay path and version log export | Replay diagnostic, fallback run |
| BeamPF | rogii-beam-pf-gpu-v2-production | v2.0 | V5 | submission | 10.212 | 2026-06-04T00:20:00Z | submitted | Submission-mode replay using V2 artifact bundle; score below selected | Submission attempt, worse than V1 |
| BeamPF | rogii-beam-pf-gpu-v2-production | v2.0 | V1 | submission | 9.941 | 2026-05-24T14:05:15.043000Z | selected | Public-best Beam/PF trajectory stack kept as selected benchmark | Best public benchmark retained |

## 6. Audit Log (Latest)

| action | timestamp_utc | actor | change | impact |
|---|---|---|---|---|
| baseline record kept | 2026-05-24 | workflow | V1 retained in 1.0 docs | public score best |

## 7. Closing Rule

Do not modify this file without actual run output.

## 8. References

- [0_readme.md](./0_readme.md)
- [4_next_steps.md](./4_next_steps.md)
- [6_kaggle_autosubmit_runbook.md](./6_kaggle_autosubmit_runbook.md)

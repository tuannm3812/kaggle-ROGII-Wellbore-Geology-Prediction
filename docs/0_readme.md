# Docs Index

## Purpose

This folder is the project documentation source of truth for analysis, results, run policies, and reproducibility operations. Code in `notebooks/` is the execution source; docs provide rationale and run strategy.

## Structure

- [1_instructions.md](./1_instructions.md)
  - Competition framing, run order, and approach narrative.
- [2_eda_insights.md](./2_eda_insights.md)
  - EDA findings and targeted follow-up analyses.
- [3_baseline_models.md](./3_baseline_models.md)
  - Baseline → alignment → Beam/PF progression and metric progression.
- [4_next_steps.md](./4_next_steps.md)
  - Current short-term execution plan and priority order.
- [5_coding_standards.md](./5_coding_standards.md)
  - Style and collaboration conventions for notebooks/docs; includes leakage-prevention rules, commit-by-function, and a pre-commit/pre-push checklist.
- [6_kaggle_autosubmit_runbook.md](./6_kaggle_autosubmit_runbook.md)
  - Kaggle GPU runbook and submission workflow.
- [7_submission_score_registry.md](./7_submission_score_registry.md)
  - Versioned submission score ledger and promotion policy.
- [8_run_log.md](./8_run_log.md)
  - Detailed chronological run-by-run history: diagnostics, ablations, promotions, bugs found and fixed.

## Documentation consistency rules

- Use numbered sections in all strategy docs (`## 1.`, `## 2.`, ...).
- Keep section titles and findings concise and decision-oriented.
- Put concrete metrics in tables with columns for metric and interpretation.
- Use relative links to notebooks, scripts, and sibling docs.
- Keep public-score claims tied to a version (for example V3, V5, V13).
- End each document with an explicit "Next Steps" block.

## Next-step ownership map

- `4_next_steps.md` drives execution priorities.
- `6_kaggle_autosubmit_runbook.md` owns operational execution (push/watch/submit).
- `1_instructions.md` owns project framing and baseline-to-endgame flow.
- `2_eda_insights.md` and `3_baseline_models.md` own empirical evidence.
- `8_run_log.md` owns the detailed run-by-run history; `4_next_steps.md` links to it rather than containing it.

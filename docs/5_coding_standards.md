# Coding Standards

## 1. Repository Scope

This repository is intentionally notebook-first. Kaggle notebooks are the executable source of truth, while `docs/` captures analysis, model results, and project decisions.

Keep the root small:

- `notebooks/` for Kaggle notebooks.
- `docs/` for reports, results, and supporting EDA artifacts.
- `README.md` for the high-level project overview.

Avoid adding local-only folders such as `data/`, `models/`, `outputs/`, `configs/`, or `scripts/` unless the project direction changes back to local training.

Large Kaggle artifacts, including trained model bundles, should stay on Kaggle as private model or dataset inputs. The repository should document them, not store them.

## 2. Notebook Naming

Use numbered, stable notebook names:

1. `1_rogii_eda.ipynb`
2. `2_rogii_baseline.ipynb`
3. `3_rogii_typewell_alignment.ipynb`
4. `4_rogii_beam_pf.ipynb`

Notebook names should describe the actual Kaggle workflow. Do not split training and submission into separate notebooks when the competition flow is meant to run end-to-end.

## 3. Code Style

Follow PEP 8 for Python code:

- Use 4 spaces for indentation.
- Keep lines to 79 characters or fewer where practical.
- Prefer concise, optimized syntax such as list comprehensions, f-strings, and small utility functions when they improve readability.
- Add type hints for functions and class methods when the type is clear.
- Group imports in this order:
  1. Standard library
  2. Third-party libraries
  3. Local modules
- Separate import groups with a blank line.

Use Google-style docstrings for reusable functions and classes:

```python
def func(x: int) -> int:
    """One-line summary.

    Args:
        x (int): Description.

    Returns:
        int: Description.
    """
```

Add short inline comments only when they explain why a decision was made. Avoid comments that restate what the code already says.

## 4. Notebook Style

Each notebook should include:

- a short purpose statement at the top;
- a clear configuration section near the top for tunable values;
- explicit mode flags when runtime behavior differs between training and submission;
- Kaggle path auto-detection where practical;
- Markdown insight cells after important plots or metrics;
- artifact-writing cells for reusable outputs such as `submission.csv`, histories, configs, model bundles, or plots.

Prefer readable, self-contained notebook code over imports from local project modules. Kaggle should be able to run the notebook after attaching only the required competition datasets and model inputs.

When notebook code changes without a fresh Kaggle rerun, clear outputs before committing. When the notebook has just been rerun on Kaggle and the outputs are being used as evidence in `docs/`, committed outputs are acceptable if they are lightweight and reviewer-relevant.

Competition notebooks should not depend on internet access during final reruns. For pretrained models or runtime packages that differ from the Kaggle image, attach local Kaggle input weights or wheelhouse datasets and load them explicitly. If exploratory installation is allowed, gate it behind an explicit config flag and keep the default offline-safe.

Submission notebooks must be optimized for Kaggle scoring limits. Do not run EDA, avoid unnecessary model training in scored submission paths, and keep submission mode focused on loading attached artifacts, running inference, and writing `submission.csv`.

For train/submission workflows:

- use `CFG.MODE = "train"` to build models and artifacts;
- use `CFG.MODE = "submission"` to load artifacts and write `submission.csv`;
- save any information needed for exact replay, such as feature columns, ensemble weights, post-processing parameters, and model best iterations;
- verify train-mode and submission-mode predictions match before treating an artifact bundle as reusable.

## 5. Plot Style

Use the Viridis palette as the default visual language across notebooks:

- Use `sns.color_palette("viridis", ...)` for categorical or sequential accents.
- Use `"viridis"` as the default colormap for heatmaps and spectrogram-like plots.
- Change color palettes only when a specific chart needs clearer contrast, semantic coloring, or accessibility improvement.
- Keep chart titles short and analytical; avoid decorative styling.

## 6. Documentation Style

Documentation should be written for a competition reviewer or teammate who wants the reasoning quickly:

- Use numbered sections.
- Lead with findings and implications.
- Include exact metrics when available.
- Link notebooks and docs with relative paths.
- Keep model result pages separate by model.
- Keep broad narrative in the root `README.md`; keep detailed evidence in focused docs.
- Separate selected-best leaderboard results from newer experimental reruns when they differ.
- Explain score drops plainly, especially when local validation and public leaderboard results disagree.
- Keep docs discoverable through `docs/0_readme.md`.
- Use a shared next-steps anchor document (`docs/4_next_steps.md`) for execution sequencing.

## 7. Git Hygiene

Do not commit:

- raw competition data;
- local checkpoints;
- Kaggle working directories;
- large embedding arrays or cached feature tables;
- large trained model artifact zips;
- Python caches or notebook checkpoints;
- ad hoc experiment dumps.

Commit lightweight artifacts only when they directly support the written analysis, such as figures used by EDA markdown and model result pages.

## 8. Commit by Function (Agent Standard)

Commit by functional scope, not mixed intent.

1. Keep each commit focused on one function domain:
   - `notebooks/`: modeling, feature logic, inference pipeline, mode flags, and Kaggle runtime behavior.
   - `docs/`: strategy, run policy, next steps, score registry, interpretation notes.
   - `README.md` and root metadata: public-facing status, index hygiene, and result summary.
   - `docs/6_kaggle_autosubmit_runbook.md`: files that directly affect Kaggle execution, submission process, and operational naming.
2. Use a professional commit message with explicit scope and version context, for example:
   - `chore(docs): add per-version important_change ledger notes for V3/V5`
   - `chore(runbook): update kaggle autosubmit checklist and GPU monitoring commands`
   - `feat(notebook): add reproducible artifact replay logging in submission mode`
3. Do not mix docs and notebook behavior changes in the same commit unless they are the same closed change.
4. For run cycles, include a short change summary in the commit body with:
   - what function changed,
   - which version label was targeted (for example `V1`, `V3`, `V5`),
   - expected impact (`selected` / `diagnostic`).
5. If a commit changes submission workflow or score records, require all of:
   - `6_kaggle_autosubmit_runbook.md` updated if execution steps changed.
   - `7_submission_score_registry.md` updated if score status changed.
   - `4_next_steps.md` updated if priorities changed.

## 9. Kaggle Submission Method

Prefer submitting via Kaggle's **notebook submission** ("Submit to
Competition" from within the notebook) over uploading a `submission.csv`
generated elsewhere. Kaggle re-executes the notebook end-to-end, which
verifies the leaderboard result actually matches the committed code. See the
shared `coding-standards/coding_standards.md` (§11) in the GitHub root for
the full rule.

Before submitting, confirm the notebook version matches what's recorded in
this project's results doc, and log the submission (version, score, date)
after it completes.

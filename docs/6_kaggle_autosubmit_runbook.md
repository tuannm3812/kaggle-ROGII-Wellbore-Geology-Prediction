# Kaggle Auto-Submit Runbook

## 1) Purpose

This runbook defines the production workflow for running 
[`notebooks/5_rogii_beam_pf_submission_replay.ipynb`](../notebooks/5_rogii_beam_pf_submission_replay.ipynb) on Kaggle with a stable notebook slug and in-notebook submission.

Use this whenever launching a new version of the Beam + Particle Filter notebook.

## 2) Current Run Policy

- Run in Kaggle notebook mode: `CFG.MODE = "train"` (artifact build) or `CFG.MODE = "submission"` (replay + submit path).
- Use in-notebook submission (Kaggle submission API endpoint is not used outside the notebook).
- Use `kaggle_kernel/` as the default clean push folder for submission replay updates.
- Keep `kaggle_runs/` as a local scratch workspace only (not committed to git).
- Keep notebook logging professional and non-datetime-only.

## 3) Required Environment

- Kaggle CLI authenticated and available:
  - `/Users/tuanm.nguyen/Library/Python/3.9/bin/kaggle`
- Active network.
- Kaggle username: `tuannm3812`
- Competition: `rogii-wellbore-geology-prediction`

## 4) Naming and Versioning Standards

Use a fixed professional slug for each production-style launch. Do not put datetimes in `id` or `title`.

Default clean submission replay kernel:

```text
run_name=rogii-beam-pf-submission-replay-cpu
kaggle_slug=${run_name}
kernel_id=tuannm3812/${kaggle_slug}
```

- `id` must be exactly: `tuannm3812/rogii-beam-pf-submission-replay-cpu`
- `title` must be exactly: `rogii-beam-pf-submission-replay-cpu`
- `code_file` must be `5_rogii_beam_pf_submission_replay.ipynb`
- Push folder should be `kaggle_kernel/`

GPU training kernel, when a fresh artifact build is needed:

```text
run_name=rogii-beam-pf-gpu-v2-production
kaggle_slug=${run_name}
kernel_id=tuannm3812/${kaggle_slug}
```

Notebook metadata and runtime logs must agree on this naming pattern.

- `id` must be exactly: `tuannm3812/rogii-beam-pf-gpu-v2-production`
- `title` must be exactly: `rogii-beam-pf-gpu-v2-production`
- `title` must have <=50 chars (safe on Kaggle)
- `code_file` must be `4_rogii_beam_pf.ipynb`
- `description` should summarize scope and risk controls

## 5) Scratch Workspace Layout

Stable push workspace:

```text
kaggle_kernel/
├── 5_rogii_beam_pf_submission_replay.ipynb
└── kernel-metadata.json
```

Local scratch workspaces:

```text
kaggle_runs/
└── rogii-beam-gpu-<run-label>/
    ├── 4_rogii_beam_pf.ipynb
    └── kernel-metadata.json
```

`<run-label>` can be a local label for tracking, e.g. `20260605-v2-prod-01`.
It does not need to match Kaggle slug as long as inside `kernel-metadata.json` the `id`/`title` are correct.

## 6) Pre-Run Checklist

1. Confirm notebook is ready:
   - `notebooks/5_rogii_beam_pf_submission_replay.ipynb` has final replay changes.
   - `professional-version-log` and `auto-submit-notebook` blocks updated.
   - `VERSION_RUN_NAME` is descriptive and non-datetime-only.
2. Confirm workspace:
   - `kaggle_kernel/5_rogii_beam_pf_submission_replay.ipynb` is synced from the clean replay notebook.
   - `kaggle_kernel/kernel-metadata.json` uses the stable replay slug.
   - `kaggle_runs/` is only used for temporary experiments.
3. Confirm metadata file exists and matches the naming policy.

## 7) Sync Clean Kaggle Kernel Folder

```bash
cd "/Users/tuanm.nguyen/Library/CloudStorage/GoogleDrive-tuannm3812@gmail.com/My Drive/10_Github/2. Kaggle/kaggle-ROGII-Wellbore-Geology-Prediction"

mkdir -p kaggle_kernel
cp notebooks/5_rogii_beam_pf_submission_replay.ipynb kaggle_kernel/5_rogii_beam_pf_submission_replay.ipynb

cat > kaggle_kernel/kernel-metadata.json <<'JSON'
{
  "id": "tuannm3812/rogii-beam-pf-submission-replay-cpu",
  "title": "rogii-beam-pf-submission-replay-cpu",
  "code_file": "5_rogii_beam_pf_submission_replay.ipynb",
  "language": "python",
  "kernel_type": "notebook",
  "is_private": true,
  "enable_gpu": false,
  "enable_tpu": false,
  "enable_internet": false,
  "dataset_sources": [],
  "competition_sources": [
    "rogii-wellbore-geology-prediction"
  ],
  "kernel_sources": [
    "tuannm3812/rogii-beam-gpu-v2-073300"
  ],
  "model_sources": [],
  "description": "ROGII Beam + PF submission replay CPU notebook. Stable clean Kaggle kernel for artifact replay and submission.csv generation."
}
JSON
```

## 8) Push to Kaggle

```bash
/Users/tuanm.nguyen/Library/Python/3.9/bin/kaggle kernels push -p kaggle_kernel
```

If output returns:
- `Maximum batch GPU session count of 2 reached.` → pause and retry after one run completes.
- 500/403 from status APIs later → retry after a short wait; continue with local output once available.

## 9) Monitor and Collect Outputs

```bash
RUN_SLUG=rogii-beam-pf-submission-replay-cpu
KERNEL_ID=tuannm3812/$RUN_SLUG
OUTPUT_DIR=/tmp/kaggle_output/$RUN_SLUG

/Users/tuanm.nguyen/Library/Python/3.9/bin/kaggle kernels status "$KERNEL_ID"
/Users/tuanm.nguyen/Library/Python/3.9/bin/kaggle kernels output "$KERNEL_ID" -p "$OUTPUT_DIR"
ls -R "$OUTPUT_DIR"
```

Then inspect logs:

```bash
tail -n 240 "$OUTPUT_DIR/${RUN_SLUG}.log"
```

Key success checks in the log:
- training mode prints artifact files and `submission.csv` path
- final section prints version log summary
- submission cell prints `Submission command invoked successfully from notebook.`

## 10) Validate Competition Submission

```bash
/Users/tuanm.nguyen/Library/Python/3.9/bin/kaggle competitions submissions -c rogii-wellbore-geology-prediction
```

Confirm latest submission status is `SubmissionStatus.COMPLETE` and compare description message to the run metadata.

## 11) Optional Local Audit

For each successful run, capture a lightweight run record:
- kernel slug
- version label (e.g., V1/V3/V5)
- run timestamp
- `train` vs `submission` mode
- local validation best score (if logged)
- public score after submit
- notable issues
- important change summary

If you are delegating to an agent, use this instruction block:
1. Run `notebooks/4_rogii_beam_pf.ipynb` on Kaggle GPU.
2. Execute `CFG.MODE = "train"` and collect `version_log.jsonl`.
3. Execute `CFG.MODE = "submission"` and run in-notebook submit.
4. Record the row in [docs/7_submission_score_registry.md](./7_submission_score_registry.md), including:
   - version label (e.g., V1/V3/V5)
   - important change summary
   - whether score improved selected benchmark
5. If public score improves current selected score, update notebook and immediately rerun submission path on Kaggle.

Append the run row to:

- [docs/7_submission_score_registry.md](./7_submission_score_registry.md)

Add operational note to:

- [docs/4_next_steps.md](./4_next_steps.md)

## 12) Post-Run Cleanup

```bash
# remove old temporary run folder after you copy the evidence you need
rm -rf "$RUN_DIR"

# optional cleanup of downloaded outputs
rm -rf /tmp/kaggle_output/$RUN_SLUG
```

## 13) Troubleshooting Notes

- `Kernel slug does not resolve` warning in push output:
  - indicates `id/title` in metadata may not match Kaggle slug normalization.
  - confirm values exactly equal.
- In-kernel submit missing:
  - check notebook environment variable `KAGGLE_URL_BASE` is present.
  - verify `submission.csv` exists before submit cell.
- Submission missing but log indicates success:
  - retry output pull and inspect full notebook log tail.
  - verify competition submission list includes the expected timestamp/description.

## 14) Commit-Back Discipline for Run Cycles

After each run cycle, commit only functionally coherent changes:

- Use the standard in [5_coding_standards.md](./5_coding_standards.md).
- At minimum, keep one clean commit for docs/runbook updates and one clean commit for notebook/runtime updates.
- For any submission-related change, include all impacted files in the same functional commit group:
  - `docs/6_kaggle_autosubmit_runbook.md` (execution changes),
  - `docs/7_submission_score_registry.md` (scores/history),
  - `docs/4_next_steps.md` (priority updates),
  - root `README.md` (if public status text changes),
  - `docs/1_instructions.md` or `docs/3_baseline_models.md` (if interpretation wording changes).

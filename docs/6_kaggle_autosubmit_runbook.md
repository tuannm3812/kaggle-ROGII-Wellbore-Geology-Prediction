# Kaggle Auto-Submit Runbook

## 1) Purpose

This runbook defines the production workflow for running
[`notebooks/4_rogii_beam_pf.ipynb`](../notebooks/4_rogii_beam_pf.ipynb) on Kaggle with a stable notebook slug, then submitting a specific completed kernel version to the competition from outside the kernel.

Use this whenever launching a new version of the Beam + Particle Filter notebook.

Note: `notebooks/4_rogii_beam_pf.ipynb` is the single source notebook for
both GPU training and CPU submission-replay pushes. Kaggle's
`kernel-metadata.json` has no field for setting environment variables, so
the `kaggle_kernel/` copy's `CFG.MODE` default (in the `class CFG` cell)
must be patched from `"train"` to `"submission"` as part of the sync step
below -- everything else about the two copies is identical (GPU on/off
and kernel slug come from `kernel-metadata.json` alone). A separate
`5_rogii_beam_pf_submission_replay.ipynb` twin previously existed and was
retired because it required manual, error-prone syncing with notebook 4.

## 2) Current Run Policy

- **This competition rejects internet-enabled notebooks and raw file
  uploads entirely.** Confirmed 2026-07-29 via the actual Kaggle API
  error body (`"This competition only accepts Submissions from
  Notebooks"`, then `"Your Notebook cannot use internet access in this
  competition"`) -- the CLI/kernel logs only ever show a generic `400
  Client Error` on their own, so don't stop at that; get the real
  message (see S13).
- Consequence: `enable_internet` must stay `false` in
  `kaggle_kernel/kernel-metadata.json`. The notebook's own in-kernel
  auto-submit cell (which calls `kaggle competitions submit` from
  inside the running kernel) **will always fail here** with a DNS
  resolution error, because it needs network access it isn't allowed to
  have. That failure is expected and harmless -- do not "fix" it by
  enabling internet, which would make the kernel version ineligible for
  submission instead.
- The actual submission step happens from **outside** the kernel, after
  a run completes, via `competition_submit_code` (kernel + version
  reference, not a file upload) -- see S10.
- Run in Kaggle notebook mode: `CFG.MODE = "train"` (artifact build) or `CFG.MODE = "submission"` (replay path, produces the `submission.csv` that gets submitted afterward).
- Use `kaggle_kernel/` as the default clean push folder for submission replay updates.
- Keep `kaggle_runs/` as a local scratch workspace only (not committed to git).
- Keep notebook logging professional and non-datetime-only.

## 3) Required Environment

- Kaggle CLI authenticated and available:
  - `/Users/tuannm3812/Library/Python/3.9/bin/kaggle`
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
- `code_file` must be `4_rogii_beam_pf.ipynb`
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
├── 4_rogii_beam_pf.ipynb
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
   - `notebooks/4_rogii_beam_pf.ipynb` has final replay changes.
   - `professional-version-log` and `auto-submit-notebook` blocks updated.
   - `VERSION_RUN_NAME` is descriptive and non-datetime-only.
2. Confirm workspace:
   - `kaggle_kernel/4_rogii_beam_pf.ipynb` is synced from `notebooks/4_rogii_beam_pf.ipynb`
     **with the `CFG.MODE` default patched to `"submission"`** (see step 7).
   - `kaggle_kernel/kernel-metadata.json` uses the stable replay slug, `enable_gpu: false`,
     and `kernel_sources` pointing at the GPU kernel whose latest output holds the
     artifact bundle to replay (currently `tuannm3812/rogii-beam-pf-gpu-v2-production`).
   - `kaggle_runs/` is only used for temporary experiments.
3. Confirm metadata file exists and matches the naming policy.

## 7) Sync Clean Kaggle Kernel Folder

```bash
cd "/Users/tuannm3812/Documents/GitHub/2. Kaggle/kaggle-ROGII-Wellbore-Geology-Prediction"

mkdir -p kaggle_kernel
cp notebooks/4_rogii_beam_pf.ipynb kaggle_kernel/4_rogii_beam_pf.ipynb

# Patch the CFG.MODE default for the replay copy only -- Kaggle's
# kernel-metadata.json has no environment-variable field, so this is the
# only way to make the pushed copy default to submission mode.
python3 - <<'PY'
import json
path = "kaggle_kernel/4_rogii_beam_pf.ipynb"
nb = json.load(open(path, encoding="utf-8"))
cell = nb["cells"][5]  # "class CFG" cell
src = "".join(cell["source"])
src = src.replace(
    'MODE = os.environ.get("ROGII_MODE", "train")',
    'MODE = os.environ.get("ROGII_MODE", "submission")',
)
cell["source"] = src.splitlines(keepends=True)
json.dump(nb, open(path, "w", encoding="utf-8"), indent=1, ensure_ascii=False)
PY

cat > kaggle_kernel/kernel-metadata.json <<'JSON'
{
  "id": "tuannm3812/rogii-beam-pf-submission-replay-cpu",
  "title": "rogii-beam-pf-submission-replay-cpu",
  "code_file": "4_rogii_beam_pf.ipynb",
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
    "tuannm3812/rogii-beam-pf-gpu-v2-production"
  ],
  "model_sources": [],
  "description": "ROGII Beam + PF submission replay CPU kernel. Runs notebooks/4_rogii_beam_pf.ipynb under a stable slug with CFG.MODE set to submission for artifact replay and submission.csv generation."
}
JSON
```

Verify the patch landed before pushing:

```bash
grep -o 'MODE = os.environ.get(\\"ROGII_MODE\\", \\"[a-z]*\\")' kaggle_kernel/4_rogii_beam_pf.ipynb
# expect: MODE = os.environ.get(\"ROGII_MODE\", \"submission\")
```

## 8) Push to Kaggle

```bash
/Users/tuannm3812/Library/Python/3.9/bin/kaggle kernels push -p kaggle_kernel
```

If output returns:
- `Maximum batch GPU session count of 2 reached.` → pause and retry after one run completes.
- 500/403 from status APIs later → retry after a short wait; continue with local output once available.

## 9) Monitor and Collect Outputs

```bash
RUN_SLUG=rogii-beam-pf-submission-replay-cpu
KERNEL_ID=tuannm3812/$RUN_SLUG
OUTPUT_DIR=/tmp/kaggle_output/$RUN_SLUG

/Users/tuannm3812/Library/Python/3.9/bin/kaggle kernels status "$KERNEL_ID"
/Users/tuannm3812/Library/Python/3.9/bin/kaggle kernels output "$KERNEL_ID" -p "$OUTPUT_DIR"
ls -R "$OUTPUT_DIR"
```

Then inspect logs:

```bash
tail -n 240 "$OUTPUT_DIR/${RUN_SLUG}.log"
```

Key success checks in the log:
- training mode prints artifact files and `submission.csv` path
- final section prints version log summary
- the in-kernel submission cell prints a DNS/connection failure and
  "Notebook submission command failed" -- **this is expected** (S2);
  it does not mean the run failed, only that the in-kernel submit
  attempt was correctly blocked from network access.

## 10) Submit The Kernel Version To The Competition

Do this from outside the kernel, after confirming the run completed and
its `submission.csv` looks right. Use the kernel **version number**
reported by the push in S8 (or check `kaggle kernels status` /
the kernel's Kaggle page for the latest version number).

```bash
/Users/tuannm3812/Library/Python/3.9/bin/kaggle competitions submit \
  rogii-wellbore-geology-prediction \
  -k tuannm3812/rogii-beam-pf-submission-replay-cpu \
  -v <version_number> \
  -f submission.csv \
  -m "<version label>: <what changed>; local RMSE <value>"
```

If this 400s with only a generic `Bad Request` message, get the real
error via a direct Python call instead of trusting the CLI's swallowed
error text:

```python
from kaggle.api.kaggle_api_extended import KaggleApi
api = KaggleApi()
api.authenticate()
try:
    print(api.competition_submit_code(
        "submission.csv", "<message>", "rogii-wellbore-geology-prediction",
        kernel="tuannm3812/rogii-beam-pf-submission-replay-cpu",
        kernel_version=<version_number>,
    ))
except Exception as e:
    print(e.response.status_code, e.response.text)
```

This requires the `kagglesdk` package to be installed locally
(`pip install kagglesdk`) -- without it, the `kaggle` CLI's submit path
silently falls back to a legacy endpoint that no longer works.

## 11) Validate Competition Submission

```bash
/Users/tuannm3812/Library/Python/3.9/bin/kaggle competitions submissions -c rogii-wellbore-geology-prediction
```

New submissions start as `SubmissionStatus.PENDING` and take a few
minutes to score -- poll until it flips to `COMPLETE` (or `ERROR`)
before reading `publicScore`. Compare the description message to the
run metadata.

## 12) Optional Local Audit

For each successful run, capture a lightweight run record:
- kernel slug
- version label (e.g., V3/V5/V10)
- run timestamp
- `train` vs `submission` mode
- local validation best score (if logged)
- public score after submit
- notable issues
- important change summary

If you are delegating to an agent, use this instruction block:
1. Run `notebooks/4_rogii_beam_pf.ipynb` on Kaggle GPU (`CFG.MODE = "train"`) and collect `version_log.jsonl`.
2. Push the CPU replay kernel (`CFG.MODE = "submission"`, S7-S8) and confirm it completes -- ignore the expected in-kernel submit failure (S2, S9).
3. Submit that kernel version to the competition from outside the kernel (S10), then poll until scored (S11).
4. Record the row in [docs/7_submission_score_registry.md](./7_submission_score_registry.md), including:
   - version label (e.g., V3/V5/V10)
   - important change summary
   - whether score improved selected benchmark
5. If public score improves current selected score, update notebook and immediately rerun submission path on Kaggle.

Append the run row to:

- [docs/7_submission_score_registry.md](./7_submission_score_registry.md)

Add a detailed operational note to:

- [docs/8_run_log.md](./8_run_log.md)

Update [docs/4_next_steps.md](./4_next_steps.md) only if the run changes
current priorities (it links to `8_run_log.md` rather than containing
run-by-run detail).

## 13) Post-Run Cleanup

```bash
# remove old temporary run folder after you copy the evidence you need
rm -rf "$RUN_DIR"

# optional cleanup of downloaded outputs
rm -rf /tmp/kaggle_output/$RUN_SLUG
```

## 14) Troubleshooting Notes

- `Kernel slug does not resolve` warning in push output:
  - indicates `id/title` in metadata may not match Kaggle slug normalization.
  - confirm values exactly equal.
- In-kernel submit fails with a DNS/`NameResolutionError`:
  - **expected** on this competition (S2) -- `enable_internet` must stay
    `false`. Do not try to fix this by enabling internet.
- `kaggle competitions submit` (or `competition_submit_code`) returns a
  bare `400 Client Error` with no useful detail:
  - the CLI and the in-kernel call both swallow the actual response
    body. Get it directly:
    ```python
    from kaggle.api.kaggle_api_extended import KaggleApi
    api = KaggleApi(); api.authenticate()
    try:
        api.competition_submit_code(...)
    except Exception as e:
        print(e.response.status_code, e.response.text)
    ```
  - Errors seen and resolved so far on this competition:
    - `"BlobFileTokens must be specified"` -- the installed `kaggle`
      package's submit path silently falls back to a legacy endpoint
      when the `kagglesdk` package isn't installed. Fix: `pip install
      kagglesdk`, retry.
    - `"This competition only accepts Submissions from Notebooks"` --
      raw file upload (`kaggle competitions submit -f submission.csv`
      with no `-k`/`-v`) is rejected outright. Use `-k`/`-v` (or
      `competition_submit_code`) to submit a specific kernel version
      instead.
    - `"Your Notebook cannot use internet access in this competition"`
      -- the kernel version being submitted had `enable_internet: true`.
      Push a fresh version with `enable_internet: false` and submit
      that version instead.
    - `"Submission files must be named \"submission.csv\""` -- this
      competition rejects any other output filename via
      `competition_submit_code`, even though the kernel can write and
      keep other named files as regular outputs. To A/B test several
      candidates (e.g. a post-process parameter sweep), you cannot
      submit multiple differently-named files from one kernel version
      -- push a separate kernel version per candidate, each producing
      its own `submission.csv` (see the 2026-07-29 tau-sweep entry in
      `docs/8_run_log.md` for a worked example using several
      lightweight CPU replay pushes, one per candidate).
    - `"Your team has used its daily Submission allowance (N) today"`
      -- resets at UTC midnight; the error message includes the exact
      wait time. Not retriable until then.
- Submission stuck at `SubmissionStatus.PENDING`:
  - normal for the first few minutes after submit; poll
    `kaggle competitions submissions` again rather than assuming failure.

## 15) Commit-Back Discipline for Run Cycles

After each run cycle, commit only functionally coherent changes:

- Use the standard in [5_coding_standards.md](./5_coding_standards.md).
- At minimum, keep one clean commit for docs/runbook updates and one clean commit for notebook/runtime updates.
- For any submission-related change, include all impacted files in the same functional commit group:
  - `docs/6_kaggle_autosubmit_runbook.md` (execution changes),
  - `docs/7_submission_score_registry.md` (scores/history),
  - `docs/8_run_log.md` (detailed run notes),
  - `docs/4_next_steps.md` (only if priorities changed),
  - root `README.md` (if public status text changes),
  - `docs/1_instructions.md` or `docs/3_baseline_models.md` (if interpretation wording changes).

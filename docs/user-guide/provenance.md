# Logs, warnings and exit codes

## What every run writes

| file | purpose |
|---|---|
| `SUMMARY.md` | what ran, counts per step, px/cm and where it came from, warnings, videos left out and why |
| `tmaze_run.log` / `gait_run.log` | everything printed during the run(s) |
| `tmaze_run_manifest.yaml` / `stride_gait_run_manifest.yaml` | the resolved settings (config file plus command-line overrides), models, code version, results |
| `<stage>_run_manifest.yaml` | one per stage, next to that stage's tables |
| `tmaze_state.json` / `gait_state.json` | what is finished, so a second run does not repeat it |

`stride status --output <dir>` lists the manifests under an output folder: stage, when, code
version, result counts.

## Run manifests

Every stage writes a `<stage>_run_manifest.yaml` next to its outputs, also when its functions
are called from Python. It records how the outputs were produced, so a table can be audited
later without the shell history.

```yaml
manifest_schema_version: 1
stage: decisions
timestamp_utc: '2026-08-07T17:06:46+00:00'
stride_version: 1.0.0
git:
  available: true
  commit_short: 46074bc85314
  branch: main
  dirty: false
  remote: https://github.com/talmolab/STRIDE.git
environment:
  python: '3.13.12'
  platform: Linux-6.8.0-x86_64-with-glibc2.39
  hostname: gpu-workstation
  user: researcher
  packages:
    stride: 1.0.0
    sleap-io: 0.7.1
    numpy: 2.4.6
command: stride tmaze --config my_tmaze.yaml
models:
  - path: /data/models/pose_model/centroid/best.ckpt
    exists: true
    size_bytes: 1507696075
    modified_utc: '2026-07-02T18:22:10+00:00'
params:    # the settings of the stage
inputs:    # what went in
outputs:   # what was written, with row counts
results:   # counts, videos left out, warnings
```

| field | content |
|---|---|
| `stage`, `timestamp_utc`, `stride_version` | which stage wrote it, when, with which version |
| `git` | commit, branch and whether the code tree had uncommitted edits (`dirty`); `available: false` when the code is not in a git checkout |
| `environment` | Python, platform, host, user, package versions |
| `command` | the command line |
| `models` | the model paths, fingerprinted by their checkpoint files (size, modification time) |
| `params` | the settings; in the pipeline manifests, the fully resolved settings |
| `inputs`, `outputs` | what went in and what was written, with row counts for tables |
| `results` | counts per stage, videos left out, warnings |

- `git.dirty: true` means the commit alone does not identify what ran.
- Manifests are best-effort. If one cannot be assembled the stage still succeeds and prints a
  warning; provenance never breaks a completed run.
- The manifest records the host and the user name of whoever ran the pipeline.
- An annotated example is in the package at `docs/example_run_manifest.yaml`.

From Python:

```python
from stride.run_manifest import write_run_manifest
write_run_manifest(output_dir, stage="gait", models=[...], params={...}, outputs=[...])
```

## Exit codes

| command | 0 | 1 | 2 |
|---|---|---|---|
| `stride tmaze`, `stride gait` | the run completed, also when some videos were left out (they are listed) | the run stopped, or `--strict` and a video was left out; the reason is printed and written to `SUMMARY.md` | a wrong option, an unknown step |
| `stride calibrate` | a value was accepted | no value was accepted; the reason is printed | a wrong option |
| `python run_all.py` | every stage completed | a stage failed; the run stops there and says which | no config given |
| `python -m stride.stages.safety_checks` | no FAIL (and no WARN with `--strict`) | a WARN with `--strict` | any FAIL |

Anything that stops a run exits non-zero with one line saying why.

## Videos that are left out

A video is left out, never guessed at, when it has no pose file, no ROI (and no consensus ROI
could be used), or its analysis raised an error. Left-out videos are counted and listed with the
reason on screen and in `SUMMARY.md`. `--strict` turns any left-out video into exit code 1.

## Warnings

Warnings never change a value. They are printed, collected in `SUMMARY.md` and in the manifest.

| warning | meaning |
|---|---|
| `N video(s) were left out` | see above; the reason is listed per video |
| `clip(s) use a consensus (borrowed) ROI` | a maze corner was hidden for the whole clip; open the montage under `roi/roi_consensus_qc/` and confirm the ROI lands on the maze |
| `scale below 20 px/cm` | paw tracking at this resolution is too coarse for the gait measures; decisions are unaffected |
| `fps < 100` | stride detection works in frames; timing and stride length are not comparable with recordings at 100 fps or more |
| the maze stem does not run left to right (`stride tmaze`); body axis within 45 degrees of the image x-axis in less than half of the walking frames (`stride gait`) | `stride_length_cm`, `step_length_cm` and the forward-direction filter assume the animal walks along the image x-axis |
| `NO usable calibration` / px/cm sentinel `1.0` | the scale of a clip could not be derived; its centimetre values are invalid |
| sampled videos show lens distortion above the thresholds | the ChArUco board in view shows distortion; STRIDE reports it and does not correct it |
| no video matched a row of the metadata; videos without a reward row | the metadata keys do not match the file names; `stride tmaze` then leaves the correctness empty |
| no stride was found in any video | the clips are too short or the animal does not walk |
| analysed videos have no arm entry (`ok = False`) | no arm dwell was found after the gate in those trials |

## A naming quirk to know

The `px_per_cm` column of the gait tables stores **centimetres per pixel**. The value used by a
run, in px/cm, is in `scale/px_per_cm.json` and in `SUMMARY.md`.

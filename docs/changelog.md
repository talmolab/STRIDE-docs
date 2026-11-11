# Changelog

## 1.0.0

First public release. Three functions for three audiences:

- **STRIDE T-maze** (`stride tmaze`): videos and two trained SLEAP models (pose, maze ROI) in;
  per-trial and per-stride CSV tables out. Finds the videos, measures the scale, runs both
  inferences, writes decisions (raw and after the two decision rules) and the gait tables.
- **STRIDE Gait** (`stride gait`): videos with pose tracking and a scale in; per-stride, per-step
  and per-video gait tables out. No maze ROI needed; skeletons with other node names are mapped.
- **STRIDE Lab** (`stride-lab`, separate package under `lab/`): the authors' own intake workflow
  (sort, whiteboard check, rename, per-day scale check), which then runs STRIDE T-maze.

Also new:

- **Scale calibration** (`stride calibrate`): px/cm from a ChArUco board in view. Scale precedence
  in the pipelines: a value given by the user, then the board, then (T-maze) the ROI corridor width.
- **Config, then command line**: every pipeline runs from one YAML file (`stride init-config`),
  with command-line flags overriding it; every step can be switched on or off; runs resume.
- **Run manifests and summaries**: every stage writes a manifest (settings, models, code version,
  inputs and outputs); the pipelines add `SUMMARY.md` and a run log.
- **Honest exits**: a failure exits non-zero with the reason; left-out videos are listed.
- A synthetic test set (`tests/mock/`) and a test suite; tests set up for Linux, macOS and Windows.
- One generic configuration file, `config.example.yaml`, for batch runs and the optional
  genotype statistics. The repository no longer ships the original study's own configuration.

Changes that alter numbers relative to 0.1.0:

- **Snout probes.** The snout position used for probe detection is now gap-filled
  (`interpolate_gaps`, `max_gap=7`) like the body keypoints. In 0.1.0 the raw track was used, so a
  short tracking dropout split one probe episode into several: probe counts were too high and probe
  frames too low. Arm choices, entry times and all gait measures are unaffected.
- **ROI selection.** The maze ROI is now chosen as in the authors' production scripts: frames from
  the middle of the video, every detected instance evaluated, all 16 corners required, a full-video
  pass when no sampled frame qualifies, and the folder's consensus ROI for a clip in which a corner
  stays hidden. In 0.1.0 only the first instance of the first frames was considered. This affects
  the ROI files written by the inference commands, not the analysis of existing ROI files.

No metric definition, threshold, filter or statistical recipe changed between 0.1.0 and 1.0.0.

## Between 0.1.0 and 1.0.0

Earlier merged pull requests on `talmolab/STRIDE`, newest first:

- **#8** Gait: the stride stem comes from the on-disk video path, not the name embedded in the
  `.slp`; the confidence merge fails loudly when stride and confidence stems disagree and reports
  unmatched strides instead of dropping them silently.
- **#7** Inference: recursive, case-insensitive video discovery, matching the decision stage.
- **#6** Gait: `px_per_cm_override` per batch in `config.yaml`, bypassing the per-video
  segment2 derivation when a verified board value exists.
- **#5** `run_all.py`: stage subprocesses run inside the repository (fixes the README quick start).
- **#4** Docs: installation clones `talmolab/STRIDE`.
- **#3** Docs: annotated example run manifest.
- **#2** Decisions: non-numeric mouse ids (cage-position, genotype-prefixed) are accepted.
- **#1** Run-manifest YAML written after each pipeline stage.

## 0.1.0

Initial internal version: the stage-by-stage T-maze pipeline (inference, decisions, gait
extraction and filtering, mixed models, classifier).

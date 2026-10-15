# Changelog

## main (after 0.1.0)

Merged pull requests on `talmolab/STRIDE`, newest first:

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

First packaged release of the pipeline used for the STRIDE manuscript.

- Config-driven batches; `run_all.py` orchestrator with resumable stages.
- Stages 1 to 10: distortion check, SLEAP pose and ROI inference, region conversion, T-maze
  decisions with depth-validated entry, stride detection and filtering, M1/M2 mixed models with
  FDR, WT-vs-Tau classification.
- Safety checks: ROI coverage, calibration, body length, frame rate, and the gate ledger.
- Run manifests for provenance on every stage.
- ROI best-frame selector evaluates every instance per frame (fixes whole days rejected as
  `no_passing_frame`).

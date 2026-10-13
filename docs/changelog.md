# Changelog

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

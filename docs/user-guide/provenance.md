# Provenance

Every stage writes `<stage>_run_manifest.yaml` next to its outputs. The manifest records how the
outputs were produced, so a table can be audited months later without the shell history.

```yaml
manifest_schema_version: 1
stage: decisions
timestamp_utc: '2026-08-07T17:06:46+00:00'
stride_version: 0.1.0
git: {commit_short: 46074bc85314, branch: main, dirty: false, remote: https://github.com/talmolab/STRIDE.git}
models:
  - {path: .../finetune_final_v5/centroid/best.ckpt, size_bytes: 1507696075, mtime: ...}
params: {min_depth_px: 50, strict_thr: 95.0, ...}
inputs: [{path: meta_trials.csv, rows: 1476}]
outputs: [{path: decisions.csv, rows: 1476}]
environment: {python: '3.13.12', packages: {sleap-nn: 0.2.0, sleap-io: 0.5.4, numpy: ...}, hostname: ..., command: ...}
results: {ok: 1330, total: 1476}
```

- `git.dirty: true` means the code tree had uncommitted edits at run time: the commit alone does
  not identify what ran.
- Manifests are best-effort. If one cannot be assembled the stage still succeeds and prints a
  warning; provenance never breaks a completed run.
- A full annotated example is in the package at `docs/example_run_manifest.yaml`.

Programmatic callers can write one too:

```python
from stride.run_manifest import write_run_manifest
write_run_manifest(output_dir, stage="gait", models=[...], params={...}, outputs=[...])
```

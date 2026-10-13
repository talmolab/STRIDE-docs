# Safety checks and gates

STRIDE grew its gates out of real incidents: a silent `px_per_cm = 1.0` once put 93% of a
cohort's strides into the figures uncalibrated. The checks exist so that a shell step can stop a
pipeline instead of trusting it.

## The checks

| check | what it tests | FAIL when |
|---|---|---|
| `--roi-coverage DIR` | every video has a region file | any video lacks a `.rois.yml` (or the count differs from `--expect-count`) |
| `--gait CSV` calibration | `px_per_cm` is uniform and, with `--expect-pxcm`, within 5% of the board value | mixed scales, `1.0` anywhere, or a uniform value that disagrees with the board |
| `--gait CSV` body length | median `body_length_cm` is plausible for a mouse | implausible median (a scale error shows up here first) |
| `--gait CSV` fps | frame rate is uniform across strides | mixed frame rates in one table |

A pinned override without `--expect-pxcm` is reported as a WARN: a uniform but wrong constant is
exactly the failure mode the gate is for.

## Gate ledger

Order is enforced by a ledger, not by memory. Each passing gate stamps `.stride_gates.json` in
the video directory, and later steps can require earlier stamps:

```bash
python -m stride.stages.safety_checks --stamp roi --vid "$VID"
python -m stride.stages.safety_checks --require rename,roi --vid "$VID"   # FAIL if either is missing
```

## Recommended order for one recording day

1. **Preflight**: number of clips equals the trial count from the recorder export.
2. **Rename** to `Day{N}_{mouse}_Trial{k}` and verify identities; stamp `rename`.
3. **ROI inference**, then `--roi-coverage`; stamp `roi`.
4. **Pose inference**: one `.slp` per clip; spot-check completeness.
5. **Decisions**: the run must report `skipped == 0`.
6. **Gait**, then the **calibration gate** with the board value.
7. Log data-quality flags for the day.

!!! tip "Rename before pose"
    The pose file embeds the video name, so renaming after inference breaks the join between
    pose, regions and metadata.

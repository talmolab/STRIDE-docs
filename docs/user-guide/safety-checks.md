# Safety checks

`python -m stride.stages.safety_checks` is a separate command that tests a finished table or
folder and returns an exit code, so a shell script can stop on it. The pipelines do not run it by
themselves: `stride tmaze` and `stride gait` print warnings and never change a value
([Logs, warnings and exit codes](provenance.md)).

## The checks

| check | what it tests | FAIL when |
|---|---|---|
| `--roi-coverage DIR` | every video under `DIR` has an ROI file, next to it or in `--yml-dir` | no video is found, any video lacks an ROI file, or the number of videos differs from `--expect-count` |
| `--gait CSV`: calibration | the `px_per_cm` column of a gait table, read as px/cm | the file is empty or has no usable `px_per_cm` column; any stride, or the median, is outside 5 to 40 px/cm (this catches the `1.0` sentinel); the 98th and 2nd percentile differ by more than a factor 1.4; more than 2 % of the strides sit near twice or half the median; or, with `--expect-pxcm`, the median differs from the expected value by more than 5 % |
| `--gait CSV`: body length | `body_length_cm` | the column is missing or empty; the median is outside 4 to 11 cm; or the table has a `camera` or `pose_model` column with more than one value |
| `--gait CSV`: fps | the `fps` column | more than one frame rate (rounded to 0.1) in the table; a missing column is a WARN |

A scale that is the same for every stride, checked without `--expect-pxcm`, is reported as a
WARN: a uniform but wrong value cannot be caught without the expected value.

```bash
python -m stride.stages.safety_checks --roi-coverage /data/videos --yml-dir /data/stride_out/roi
python -m stride.stages.safety_checks --gait /data/stride_out/gait/gait_per_stride_filtered.csv --expect-pxcm 26.1 --strict
```

Exit code 2 on any FAIL, 1 on a WARN when `--strict` is given, 0 otherwise.

## Gate ledger

Each passing step can stamp `.stride_gates.json` in a video folder, and a later step can require
earlier stamps:

```bash
python -m stride.stages.safety_checks --stamp roi --vid /data/videos
python -m stride.stages.safety_checks --require rename,roi --vid /data/videos   # FAIL if either is missing
```

The gate names are free text. The authors' own intake workflow runs the gait calibration check
after its gait stage ([Authors' lab workflow](../about/lab-workflow.md)).

# Inputs and file layout

A cohort is a directory of videos plus three small tables. Per-day subdirectories are supported;
discovery is recursive.

```
my_cohort/
  2026-09-29-DAY1/
    Day1_16676_Trial1.MP4
    Day1_16676_Trial1.slp        # pose, written by stage 2
    Day1_16676_Trial1.rois.yml   # maze regions, written by stage 2
    ...
  meta_trials.csv
  mice.csv
```

## Videos

`Day{N}_{mouse}_Trial{k}.MP4`, one trial per clip. The stem carries the identity used everywhere
downstream. Frame rate is read from the file; `fps` in the config is the fallback.

## Pose file (`.slp`)

A SLEAP predictions file with the same stem as the video, next to it (or with the suffix given by
`pose_slp_suffix`). One instance per frame with the 15-node skeleton:

`snout`, `mouth`, `forepawL1`, `forepawL2`, `forepawR1`, `forepawR2`, `hindpawL1`, `hindpawL2`,
`hindpawR1`, `hindpawR2`, `tailbase`, `tail1`, `tail2`, `tail3`, `tailtip`.

Gait uses the snout, tail base, tail tip and hind paws; the two candidate nodes per hind paw are
resolved per video by picking the better tracked one.

## Region file (`.rois.yml`)

Seven polygons per clip, resolved in this order: `{stem}.preds.v2.best1.yml`, `{stem}.rois.yml`,
`{stem}.yml`, first next to the video, then in `yml_dir`.

```yaml
image_file: /data/my_cohort/2026-09-29-DAY1/Day1_16676_Trial1.MP4
roi_count: 7
rois:
- id: 1
  name: arm_right          # arm_right, junction, arm_left, segment1 ... segment4
  type: polygon
  coordinates: [[x, y], [x, y], [x, y], [x, y]]
```

`segment1` to `segment4` tile the stem from the start box to the junction; `segment2` is also the
scale reference (its width is the 10 cm corridor) when no `px_per_cm_override` is set.

## Trial metadata (`meta_trials.csv`)

| column | meaning |
|---|---|
| `day` | `DAY1`, `DAY2`, ... |
| `mouse` | mouse id as in the file names |
| `reward` | rewarded arm for that mouse on that day, `L` or `R` |

## Mouse metadata (`metadata_csv`)

Genotype and sex per mouse. Column names are mapped in the batch's `column_map`, so an existing
spreadsheet with `Gender` or a two-column mouse id (`G` + `number`) works without editing.

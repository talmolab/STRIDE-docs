# STRIDE Gait (`stride gait`)

Videos with pose tracking go in; per-stride, per-step and per-video gait tables come out. No maze
and no ROI model are needed: you bring the pose tracking, STRIDE gets the scale (px/cm) and does
the gait analysis.

```
discover -> scale (px/cm) -> pose inference (only with a pose model) -> stride detection
         -> keypoint confidence -> stride filtering
```

The command adds no computation of its own: every step calls the same stage function the rest of
the package uses (`run_gait_extraction`, `compute_keypoint_confidence`, `filter_strides`), so the
tables are the same whichever way the stages are started.

## What you need

| | |
|---|---|
| Videos | `.mp4` (any letter case), any file names, in one folder or many. Names must be unique: every table is keyed by the file name. |
| Pose | Either a pose file per video (`<video name>.slp`, next to the video or in `--pose-dir`; another ending with `--pose-suffix`), or your trained SLEAP top-down pose model (`--pose-model`, one or two folders) to make them. One animal per video. |
| Keypoints | `snout`, `tailbase`, and at least one landmark per hind paw (`hindpawL1` / `hindpawL2`, `hindpawR1` / `hindpawR2`); `tailtip` is optional. Other node names are fine: map them (see [Keypoints](#keypoints)). |
| Scale | px/cm: a value you measured (`--px-per-cm`), or a ChArUco board in view. There is no other source: without one of the two the run stops. |
| Recording | The paws must be visible (camera below a transparent floor). The animal should walk left-right in the image, at 100 fps or more and 20 px/cm or more; the run warns otherwise (see [Warnings](#warnings-you-may-see)). |
| Runtime | The analysis runs on Windows, macOS and Linux with `pip install -e .`. Pose inference additionally needs [sleap-nn](https://github.com/talmolab/sleap-nn) and, in practice, a GPU. |

## Run it

```bash
# pose files exist, scale measured by you
stride gait /data/videos --px-per-cm 26.1 --out /data/gait_out

# pose files elsewhere, scale from the ChArUco board in view
stride gait /data/videos --pose-dir /data/pose --out /data/gait_out

# no pose files yet: your pose model makes them first
stride gait /data/videos --pose-model /models/pose/centroid --pose-model /models/pose/centered_instance \
    --px-per-cm 26.1 --out /data/gait_out

# or from a config file (any option on the command line overrides the file)
stride init-config gait -o my_gait.yaml        # writes a commented template
stride gait --config my_gait.yaml
stride gait --config my_gait.yaml --out /somewhere/else --workers 8
```

`VIDEOS` is a folder (searched recursively), one video, or a pattern such as `"D:/rec/*/*.mp4"`;
several may be given. Folders named `waste` are skipped (`exclude_dirs` in the config).

### Steps

| step | what it does | switch off with |
|---|---|---|
| discover | finds the videos, reads the metadata file | (always runs) |
| `calibrate` | px/cm from the ChArUco board in view; not used when `--px-per-cm` is given | `--skip calibrate` |
| `pose_inference` | runs your pose model on the videos that have no pose file; without a model those videos are left out and listed | `--skip pose_inference` |
| (keypoints) | checks the first pose file for the keypoints the analysis needs | (always runs) |
| `gait_extraction` | stride detection and the per-stride, per-step and per-video tables | `--skip gait_extraction` |
| `keypoint_confidence` | mean keypoint confidence per video (used by the confidence filter) | `--skip keypoint_confidence` |
| `stride_filtering` | writes `gait_per_stride_filtered.csv` | `--skip stride_filtering` |

In the config the same switches are the `steps:` block ([Configuration reference](../user-guide/configuration.md#stride-gait-config)). `--stop-after STEP` ends the run after a
step. A second run does not repeat finished work (`--force` redoes it). Exit code: 0 when the run
completed, 1 when it stopped (the reason is printed and written to `SUMMARY.md`), 2 for a wrong
option. Videos that had to be left out do not change the exit code unless `--strict` is given.

## Keypoints

The gait stage reads these keypoints by name:

| STRIDE name | needed | used for |
|---|---|---|
| `snout` | yes | body axis, body length, snout sway |
| `tailbase` | yes | body position and speed, body axis, origin of the body frame |
| `hindpawL1`, `hindpawL2` | at least one | left hind paw (swings, strides, step and stride measures) |
| `hindpawR1`, `hindpawR2` | at least one | right hind paw |
| `tailtip` | no | tail-tip sway; without it the two tail-tip columns stay empty |

If your skeleton uses other names, map them in the config (`their name: STRIDE name`) or on the
command line:

```yaml
keypoints:
  nose: snout
  tail_root: tailbase
  tail_end: tailtip
  left_hindpaw: hindpawL1
  right_hindpaw: hindpawR1
```

```bash
stride gait /data/videos --px-per-cm 26.1 -o out --keypoint nose=snout --keypoint tail_root=tailbase \
    --keypoint left_hindpaw=hindpawL1 --keypoint right_hindpaw=hindpawR1
```

- With two landmarks per hind paw, one is picked per side and per video (see `hindL_ind` below).
  With one landmark per hind paw that landmark is used.
- The check runs on the first pose file before any analysis. If a needed keypoint is missing the
  run stops and prints what is missing and which nodes the pose file has.
- The 15-node STRIDE skeleton is `snout, mouth, forepawR2, forepawR1, forepawL1, forepawL2,
  hindpawR2, hindpawR1, hindpawL2, hindpawL1, tailbase, tail1, tail2, tail3, tailtip`. For any
  other skeleton the per-video mean keypoint confidence (used by the confidence filter) is taken
  over the nodes that skeleton has.

## Scale (px/cm)

1. `--px-per-cm VALUE` (or `scale: px_per_cm:` in the config): used as given.
2. Otherwise the ChArUco board in view: the first video of every folder is measured (at least
   `min_clips`), the median is used when at least 3 clips agree within 1 %. Describe a board other
   than the default (14 x 9 squares, 20 mm squares, 15 mm markers, `DICT_5X5_250`) with
   `--board-squares-x`, `--board-squares-y`, `--board-square-mm`, `--board-marker-mm`, `--board-dict` or `scale: charuco:`. `scale: charuco: clips:` names the videos to measure. The
   measurement is also available on its own: `stride calibrate` ([Scale calibration](calibration.md)).
3. Otherwise the run stops with the reason. One value is used for every video of the run.

`scale/px_per_cm.json` records the value and where it came from.

## Metadata (optional)

A CSV with a `video` column holding the file name (with or without folder and extension) and any
other columns:

```csv
video,animal,sex,group,weight_g
runway A1.mp4,A1,F,control,21.5
```

- An animal id (`mouse_id`, `mouse`, `animal`, `animal_id`, `subject` or `id`) fills `mouse_id`;
  `sex` / `gender` fills `sex`; `genotype` fills `genotype`; `day` fills `day`.
- Every other column (and the id column under its own name) is added at the end of
  `gait_per_stride.csv`, `gait_per_step.csv`, `gait_per_video.csv` and `gait_per_stride_filtered.csv`.
- A file named `Day<N>_<animal>_Trial<T>.mp4` gets `day` and `mouse_id` from its name.
- Without metadata (or without a row for a video) these columns stay empty.

## Output folder

```
SUMMARY.md                       what ran, counts, px/cm and its source, warnings, videos left out and why
gait_run.log                     everything printed
stride_gait_run_manifest.yaml    resolved settings, model, code version, results
gait_state.json                  what is finished (for the next run)
scale/px_per_cm.json             the scale and its source (+ ChArUco files when the board was measured)
pose/<video name>.slp            only for videos whose pose file was made by this run
gait/gait_per_stride.csv         one row per stride
gait/gait_per_stride_filtered.csv   the strides that pass the filters
gait/gait_per_step.csv           one row per hind-paw swing
gait/gait_per_video.csv          one row per video
gait/pose_keypoint_confidence_by_video.csv
gait/videos.csv, run_params.json, *_run_manifest.yaml
```

No trial phase and no maze region is attached to a stride in this mode, and the trial columns of
`gait_per_video.csv` (`choice_LR`, `latency_choice_ms`, `stem_latency_ms`, `junction_explore_ms`,
`reward`, `correct_TF`, `correct_bin`, `probes_L`, `probes_R`) are empty.

## How the tables are computed

Every column of the gait tables is described, exactly as the code computes it, in the
[column glossary](../user-guide/outputs.md#gait-tables); the detection of walking, swings and
strides and the stride filters are described in
[Stride detection and gait measures](../how-it-works/gait.md). Three things to know before
reading the tables:

- `stride_length_cm` and `step_length_cm` are measured relative to the body, not on the ground,
  and `step_length_cm` depends on which hind-paw landmark was picked on each side;
- `duration_ms` is `(frame_end - frame_start) / fps`;
- the `px_per_cm` column stores **centimetres per pixel** (historical name).

## Warnings you may see

Warnings never change a value. They are printed, collected in `SUMMARY.md` and in the manifest.

| warning | meaning |
|---|---|
| body axis within 45 degrees of the image x-axis in less than half of the walking frames | `stride_length_cm`, `step_length_cm` and the forward-direction filter assume the animal walks left-right in the image. Rotate the camera or the videos so that the runway is horizontal. The share is always printed (`[direction]`). |
| fps below 100 | stride detection works in frames; timing and stride length are not comparable with recordings at 100 fps or more |
| scale below 20 px/cm | paw tracking at this resolution is too coarse for the gait measures |
| no stride was found in any video | the clips are too short or the animal does not walk |
| N video(s) were left out | no pose file and no pose model, or the pose file could not be analysed; `SUMMARY.md` lists each with its reason |

## Limits

- One scale for the whole run. Recordings with different scales are run separately.
- One animal per video; the first instance of each frame is used.
- The step and stride measures are relative to the body and assume walking along the image x-axis.
- No trial phase, no maze region, no decision table: those need the maze ROI (`stride tmaze`).
- The genotype statistics are not part of this command.

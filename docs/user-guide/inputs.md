# Inputs and file layout

What each pipeline reads, and where it looks for it. Nothing here has to be renamed or moved:
STRIDE writes only into its output folder.

## Videos

- `.mp4`, any letter case of the extension, in one folder or many. A folder is searched
  recursively; folders named `waste` are skipped (`exclude_dirs`).
- **Any file names.** Names must be unique within a run: every table is keyed by the file name.
- A name of the form `Day<N>_<mouse>_Trial<T>.mp4` is recognised; day, mouse and trial are then
  filled from it. The mouse part may be any text (`1042`, `M3-2`, `WT_1_1`).
- The frame rate is read from the video; `fps_default` (120) is used only when it cannot be read.

The stage-by-stage batch runner (`run_all.py`, `stride analyze-decisions`, `stride run`)
**requires** the `Day<N>_<mouse>_Trial<T>` form and a reward table; `stride tmaze` and
`stride gait` do not.

## Pose files

One SLEAP predictions file per video, `<video name>.slp` (another ending with `--pose-suffix` /
`pose_suffix`). The pipelines look next to the video first, then in the output folder's `pose/`
(files made by an earlier run), then in `--pose-dir` / `pose_dir`. One animal per video; the
first instance of each frame is used.

A pose file remembers the path of the video it was made from. When that path cannot be read on
the machine running the analysis, STRIDE uses the video on disk instead.

### Keypoints STRIDE reads

The 15-node mouse skeleton of the original study:

`snout, mouth, forepawR2, forepawR1, forepawL1, forepawL2, hindpawR2, hindpawR1, hindpawL2,
hindpawL1, tailbase, tail1, tail2, tail3, tailtip`

| step | keypoints it reads by name |
|---|---|
| decisions (`stride tmaze`) | `snout`, `forepawL2`, `forepawR2`, `hindpawL2`, `hindpawR2`, `tailbase` (at least three must exist) |
| gait | `snout`, `tailbase`, `tailtip`, `hindpawL1`, `hindpawL2`, `hindpawR1`, `hindpawR2` |

`stride gait` also accepts other skeletons: map their node names with `keypoints:`. It needs the
snout, the tail base and at least one landmark per hind paw; the tail tip is optional
([STRIDE Gait](../pipelines/gait.md#keypoints)).

## ROI files (T-maze only)

Seven polygons per video. `stride tmaze` looks in the output folder's `roi/` (files made by an
earlier run), then in `--roi-dir` / `roi_dir`, then next to the video, for these names, in this
order: `<video name>.preds.v2.best1.yml`, `<video name>.rois.yml`, `<video name>.yml`. It writes
them to the output folder's `roi/` when it runs your ROI model.

```yaml
image_file: /data/videos/clip_001.mp4
roi_count: 7
rois:
- id: 1
  name: arm_right          # arm_right, junction, arm_left, segment1 ... segment4
  type: polygon
  coordinates: [[x, y], [x, y], [x, y], [x, y]]
```

`segment1` to `segment4` tile the stem; `segment4` is the one next to the junction and `segment1`
the far end. When no scale is given and no ChArUco board is in view, `segment2` is the scale
reference: the mean length of its shorter pair of opposite edges is taken as the corridor width
(`maze_width_cm`, 10 cm).

### What the ROI model must provide

A SLEAP top-down model with the 23-node maze skeleton: 16 corners and 7 centroids.

| corners (16, all required) | centroids (7) |
|---|---|
| `arm_right.top_left`, `arm_right.top_right` | `arm_right.centroid` |
| `junction.top_left`, `junction.top_right`, `junction.bottom_left`, `junction.bottom_right` | `junction.centroid` |
| `arm_left.bottom_left`, `arm_left.bottom_right` | `arm_left.centroid` |
| `segment1.top_left`, `segment1.bottom_left` ... `segment4.top_left`, `segment4.bottom_left` | `segment1.centroid` ... `segment4.centroid` |

How a frame is chosen and what happens when a corner is hidden:
[Pose and ROI models](../how-it-works/models.md).

## Metadata (optional)

### `stride tmaze`

Without metadata the arm choice, the timing and the probe columns are written and the correctness
columns stay **empty**. A metadata CSV (several may be given) has trial rows and/or mouse rows.
Column names are case-insensitive.

| rows | key columns | value columns |
|---|---|---|
| trial rows | `video` (file name, with or without extension) **or** `day` + `mouse` (+ `trial`) | `reward` (`L` / `R`); optional `day`, `mouse`, `trial`, `sex`, `genotype` |
| mouse rows | `mouse` | `sex` (or `Gender`), `genotype` |

`day` + `mouse` rows are matched to videos named `Day<N>_<mouse>_Trial<T>` (the day may be written
`Day1`, `DAY1` or `1`); for any other file name use the `video` column. A video without a matching
row keeps empty correctness; the run says how many.

```csv
day,mouse,reward
Day1,101,R
Day2,101,L
```

```csv
Mouse,Gender,Genotype
101,F,WT
102,M,WT
```

### `stride gait`

A CSV with a `video` column holding the file name and any other columns. An animal id
(`mouse_id`, `mouse`, `animal`, `animal_id`, `subject` or `id`) fills `mouse_id`; `sex` / `gender`
fills `sex`; `genotype` fills `genotype`; `day` fills `day`. Every other column, and the id column
under its own name, is added at the end of the gait tables.

```csv
video,animal,sex,group,weight_g
runway A1.mp4,A1,F,control,21.5
```

### Batch runner (`run_all.py`)

`meta_trials_csv` with the columns `day,mouse,reward` and `metadata_csv` with one row per mouse
(`Mouse,Gender,Genotype`; the names are mapped in the batch's `column_map`). In these entry points
a video whose (day, mouse) has no reward row is written as **not correct** (`correct_bin = 0`)
and a warning names it; `stride tmaze` leaves the correctness empty instead.

## Scale

See [Scale calibration](../pipelines/calibration.md). In short: a value you give, else the
ChArUco board in view, else (T-maze only) the ROI corridor width.

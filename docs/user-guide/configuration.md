# Configuration reference

STRIDE has three configuration files, each for one way of running it.

| file | written by | used by |
|---|---|---|
| T-maze config | `stride init-config tmaze` | `stride tmaze --config` |
| Gait config | `stride init-config gait` | `stride gait --config` |
| Batch / statistics config | copy `config.example.yaml` | `python run_all.py --config`, the stage modules, and `stats_config` of `stride tmaze` |

## Config, then command line

```bash
stride init-config tmaze -o my_tmaze.yaml      # writes a commented template
stride tmaze --config my_tmaze.yaml
stride tmaze --config my_tmaze.yaml --out /somewhere/else --workers 8
```

- The order is **defaults, then the config file, then the command line**: an option given on the
  command line overrides the value in the file. Both pipelines also run from the command line alone.
- A key that is not listed below is refused, with the list of known keys. So is an unknown step
  under `steps:`.
- Relative paths are resolved from the folder the command is started in. Paths may contain spaces.
- The settings a run actually used (file plus overrides) are written into its run manifest.

## STRIDE T-maze config

| key | type | default | meaning |
|---|---|---|---|
| `videos` | path, pattern or list | none | a folder (searched recursively), one file, a pattern such as `"D:/recordings/*/*.mp4"`, or a list of these |
| `output` | path | none | everything STRIDE writes goes here |
| `pose_model` | list of 1 or 2 paths | `[]` | your trained SLEAP pose model (centroid, centered instance) |
| `roi_model` | list of 1 or 2 paths | `[]` | your trained SLEAP maze-ROI model |
| `pose_dir` | path | `null` | folder with existing pose files, when they are not next to the videos |
| `roi_dir` | path | `null` | folder with existing ROI files, when they are not next to the videos |
| `metadata` | path or list | `null` | optional CSV(s): trial rows and/or mouse rows ([Inputs](inputs.md#metadata-optional)) |
| `pose_suffix` | text | `.slp` | pose file = `<video name><suffix>` |
| `exclude_dirs` | list | `[waste]` | folder names skipped when searching for videos |
| `device` | text | `auto` | inference device: `auto`, `cuda`, `cuda:0`, `cpu` |
| `workers` | integer | `4` | CPU worker processes for the analysis |
| `batch_size` | integer | `16` | pose inference batch size |
| `fps_default` | number | `120` | used only when a video's frame rate cannot be read |
| `scale.px_per_cm` | number | `null` | a value you measured yourself; wins over everything else |
| `scale.maze_width_cm` | number | `10.0` | corridor width, for px/cm from the ROI |
| `scale.charuco.squares_x`, `squares_y`, `square_mm`, `marker_mm`, `dictionary` | | `null` | the board in view; `null` = the default board (14 x 9 squares, 20 mm squares, 15 mm markers, `DICT_5X5_250`) |
| `scale.charuco.min_clips` | integer | `6` | videos measured: the first of every folder, at least this many |
| `scale.charuco.max_clips` | integer | `null` | and at most this many |
| `scale.charuco.clips` | list | `null` | measure exactly these videos instead of that choice |
| `roi.session` | `folder` or `all` | `folder` | which clips share one ROI geometry for the consensus ROI |
| `roi.drift_max_px` | number | `25` | the consensus ROI is refused when the detected ROIs differ by more than this at any vertex |
| `roi.full_pass` | true / false | `true` | run the ROI model on the whole video when the sampled frames show no full maze |
| `roi.frames` | integer | `10` | frames sampled per video in the first ROI pass |
| `distortion.max_clips` | integer | `6` | videos sampled by the distortion check |
| `distortion.frames` | integer | `10` | frames examined per sampled video |
| `decision_rules.window_s` | number | `150` | trial length: an arm entry after this many seconds is no decision |
| `stride_detection` | mapping | `{}` | optional overrides of the stride detection thresholds ([names below](#analysis-settings)) |
| `stride_filtering` | mapping | `{}` | optional overrides of the stride filters |
| `steps` | mapping | see below | switch steps on or off |
| `stats_config` | path | `null` | only for `mixed_models` / `classifier`: a batch / statistics config (below) |
| `stop_after` | step name | `null` | stop after this step |
| `force` | true / false | `false` | redo the analysis steps even when their outputs are up to date |
| `strict` | true / false | `false` | exit with an error when any video was left out |

Steps and their defaults:

```yaml
steps:
  distortion_check: true     # report only
  calibrate: true            # px/cm from the ChArUco board
  pose_inference: true       # skipped per video when its pose file exists
  roi_inference: true        # skipped per video when its ROI file exists
  roi_consensus: true        # borrow the session's ROI for clips with a hidden maze corner
  decisions: true
  decision_rules: true
  gait: true
  mixed_models: false        # genotype statistics: off by default, need stats_config
  classifier: false
```

On the command line: `--skip STEP`, `--enable STEP` (both repeatable), `--skip-inference`
(both inference steps off), `--stop-after STEP`.

## STRIDE Gait config

| key | type | default | meaning |
|---|---|---|---|
| `videos` | path, pattern or list | none | as above |
| `output` | path | none | as above |
| `pose_dir` | path | `null` | folder with existing pose files, when they are not next to the videos |
| `pose_suffix` | text | `.slp` | pose file = `<video name><suffix>` |
| `pose_model` | list of 1 or 2 paths | `[]` | your trained pose model: pose inference then runs for the videos without a pose file |
| `metadata` | path | `null` | optional CSV with a `video` column; its other columns are added to the tables |
| `keypoints` | mapping | `{}` | node name in your pose files: STRIDE keypoint name |
| `exclude_dirs` | list | `[waste]` | folder names skipped when searching for videos |
| `device` | text | `auto` | pose inference device |
| `workers` | integer | `4` | CPU worker processes |
| `batch_size` | integer | `16` | pose inference batch size |
| `fps_default` | number | `120` | used only when a video's frame rate cannot be read |
| `scale.px_per_cm` | number | `null` | a value you measured yourself; wins over the board |
| `scale.charuco.*` | | as above | the board in view; without a value and without a board the run stops |
| `stride_detection`, `stride_filtering` | mapping | `{}` | optional overrides ([names below](#analysis-settings)) |
| `steps` | mapping | all `true` | `calibrate`, `pose_inference`, `gait_extraction`, `keypoint_confidence`, `stride_filtering` |
| `stop_after` | step name | `null` | stop after this step |
| `force`, `strict` | true / false | `false` | as above |

`pose_inference` only acts when a pose model is given and a video has no pose file.

```yaml
keypoints:                 # only when your pose skeleton uses other node names
  nose: snout
  tail_base: tailbase
  tail_end: tailtip
  left_hindpaw: hindpawL1  # one landmark per hind paw is enough
  right_hindpaw: hindpawR1
```

## Batch / statistics config

`config.example.yaml` in the repository root is the one configuration file for the
stage-by-stage runner and the optional statistics. Copy it and edit the `batches:` section.

```bash
cp config.example.yaml my_config.yaml
python run_all.py --config my_config.yaml --output output
```

### `batches`

A batch is one cohort recorded on one rig. The batch key is used for `--batch` and as the output
subfolder.

```yaml
batches:
  my_batch:
    name: "My batch"
    video_dir: "/path/to/videos/"                # searched recursively; names Day<N>_<mouse>_Trial<T>.mp4
    meta_trials_csv: "/path/to/meta_trials.csv"  # columns: day,mouse,reward
    metadata_csv: "/path/to/metadata.csv"        # columns: Mouse,Gender,Genotype (one row per mouse)
    fps: 120                                     # used only when the frame rate cannot be read
    column_map:
      mouse_id: ["mouse"]
      mouse_id_sep: ""
      sex: "Gender"
```

| optional key | meaning |
|---|---|
| `yml_dir` | ROI polygon files; leave out when stage 2 writes them or when they sit next to the videos |
| `pose_slp_suffix` | pose file next to each video: `<video name><suffix>` |
| `px_per_cm_override` | a calibration you measured, used for every video instead of the per-video value derived from the ROI |
| `has_sex` | add Sex as a covariate in the mixed models |
| `finalized` | never re-run inference or decisions for this batch; use the CSVs given below |
| `pose_model_paths`, `roi_model_paths` | SLEAP models for stages 1 to 2; leave out to use existing pose and ROI files |
| `device` | inference device for this batch |
| `decisions_csv`, `events_csv`, `metrics_meta_csv`, `stride_csv` | explicit inputs for the later stages; normally not needed, since each stage reads what the previous one wrote under `<output>/<batch>/` |
| `mouse_id_cols`, `mouse_id_sep` | only for stride or T-maze tables that do not come from STRIDE |
| `calibrate` | `true` or a block of calibration settings: `python -m stride.stages.run_calibration` then measures px/cm from the ChArUco board and writes `<output>/<batch>/calibration/px_per_cm.txt`. Put that value in `px_per_cm_override`; the batch gait stage does not read the file by itself ([Scale calibration](../pipelines/calibration.md)) |

A top-level `inference:` block (`device`, `batch_size`) sets the defaults for every batch.

### Analysis settings

These are the values used in the STRIDE paper; change them only if you mean to change the analysis.

```yaml
gait_extraction:
  n_workers: 16

stride_detection:
  min_walk_velocity: 5.0        # cm/s, tail-base speed to count as walking
  min_paw_velocity: 8.0         # cm/s, paw speed for swing candidacy
  min_peak_paw_velocity: 15.0   # cm/s, peak paw speed to keep a swing
  min_frames_swing: 3           # minimum swing duration, in frames
  smooth_median_window: 3       # median filter window, in frames
  smooth_moving_avg_window: 5   # moving average window, in frames
  phase_spline_k: 3             # spline order for phase interpolation
  phase_grid_points: 101        # interpolation grid (0 to 100 % of the stride cycle)
  maze_width_cm: 10.0           # corridor width, used to derive px/cm from the ROI

stride_filtering:
  confidence_threshold: 0.3     # minimum mean keypoint confidence per video
  angular_velocity_min: -20.0   # deg/s
  angular_velocity_max: 20.0    # deg/s
  remove_edge_strides: true     # drop the first and last stride of every video
  filter_forward_direction: true
```

The same `stride_detection` and `stride_filtering` keys are the optional overrides in the T-maze
and gait configs.

### Statistics settings

| section | content | read by the stages |
|---|---|---|
| `metrics` | the 14 columns that enter the mixed models, with label, category and units; `step_freq_hz` is derived as `1000 / duration_ms` | yes |
| `covariates` | `speed: stride_speed_cm_s_mean`, `body_length: body_length_cm` | yes |
| `models` | `M1` (covariate `body_length_cm`) and `M2` (covariates `stride_speed_cm_s_mean`, `body_length_cm`, `skip_outcome_if_covariate: true`) | `covariates` and `skip_outcome_if_covariate` are read; `description` and `formula` document the model |
| `statistics` | `reference_genotype: WT`, `genotype_order: [WT, APP, Tau]`, `optimizer: powell`, `fdr_alpha: 0.05`, `fdr_method: fdr_bh` | `reference_genotype`, `genotype_order` and `optimizer` are read; the FDR is Benjamini-Hochberg with stars at 0.05 in the code |
| `classification` | `merged_data_csv: null`, the optional p-value tables `gait_lmm_pvalues` and `tmaze_lmm_pvalues` used to rank the features, `n_splits: 5`, `random_state: 42`, `target_genotypes: [WT, Tau]` | `merged_data_csv` and the two p-value tables are read; 5 splits, seed 42 and WT vs Tau are fixed in the code |
| `trial_phase_tagging` | `task_window` = `gate_frame` to `entry_frame`, `poked_return_speed_threshold: 2.0`, `default_filter: task` | no: it documents what the stage does |
| `colors`, `significance_markers`, `regional` | figure colours, star thresholds, a regional (stem, junction, arm) analysis | no: not read by any stage of the package in 1.0.0 |

What the stages do with these: [Statistics](../how-it-works/statistics.md).

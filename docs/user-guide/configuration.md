# Configuration

`config.yaml` is the single source of truth: batches, thresholds, the metric list and the model
formulas. Nothing batch-specific lives in code.

## Batches

```yaml
batches:
  gopro_sept2025:
    name: "GoPro (Sept 2025)"
    video_dir: "/data/gopro/"
    yml_dir: "/data/gopro/roi_yml/"
    meta_trials_csv: "/data/gopro/meta_trials.csv"
    metadata_csv: "/data/mice.csv"
    fps: 120
    px_per_cm: 27.38            # reference only; see px_per_cm_override
    px_per_cm_override: 26.08   # optional: trusted board value, bypasses per-video derivation
    camera_type: "gopro"
    pose_slp_suffix: ".slp"
    has_sex: true               # add Sex to the mixed models
    pose_model_paths: [".../centroid", ".../centered_instance"]
    roi_model_paths:  [".../centroid", ".../centered_instance"]
    column_map:
      mouse_id: ["mouse"]       # or ["G", "number"] with mouse_id_sep: "_"
      mouse_id_sep: ""
      sex: "Gender"
    finalized: false            # true = skip inference, tables already exist
```

For a batch that is already processed, give `stride_csv`, `step_csv`, `events_csv` and
`decisions_csv` directly and set `finalized: true`; the statistics stages read the tables as-is.

## Stride detection

```yaml
stride_detection:
  min_walk_velocity: 5.0        # cm/s, tail base speed that counts as walking
  min_paw_velocity: 8.0         # cm/s, paw speed that starts a swing candidate
  min_peak_paw_velocity: 15.0   # cm/s, peak a swing must reach to be kept
  min_frames_swing: 3
  smooth_median_window: 3       # frames
  smooth_moving_avg_window: 5   # frames
  phase_spline_k: 3
  phase_grid_points: 101        # 0 to 100 % of the stride cycle
  maze_width_cm: 10.0           # corridor width used for the ROI-derived scale
```

## Stride filtering

```yaml
stride_filtering:
  confidence_threshold: 0.3     # minimum mean keypoint confidence per video
  remove_edge_strides: true     # drop the first and last stride of each run
  filter_forward_direction: true   # drop backward strides (hesitations, reversals)
  angular_velocity_min: -20.0   # deg/s, straight-walking band
  angular_velocity_max: 20.0
```

## Trial phase tagging

```yaml
trial_phase_tagging:
  task_window: { start: "gate_frame", end: "entry_frame" }   # columns of decisions.csv
  poked_return_speed_threshold: 2.0   # x the mouse's within-trial median speed
  default_filter: "task"              # which phase enters the mixed models
```

## Metrics and models

The `metrics:` list names the 14 columns that enter the mixed models and heatmaps, with label,
category and units; `step_freq_hz` is derived as `1000 / duration_ms`. The two model formulas:

```yaml
models:
  M1:
    formula: "{outcome} ~ C(Genotype, Treatment('WT')) + scale(body_length_cm)"
  M2:
    formula: "{outcome} ~ C(Genotype, Treatment('WT')) + scale(stride_speed_cm_s_mean) + scale(body_length_cm)"
```

`+ Sex` is appended when the batch has `has_sex: true`. Mouse is always the random intercept.

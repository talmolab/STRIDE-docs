# Outputs

Every stage writes its tables into the batch's output directory and a `<stage>_run_manifest.yaml`
beside them ([Provenance](provenance.md)).

## T-maze (stage 3)

**`decisions.csv`**, one row per clip:

| column | meaning |
|---|---|
| `day`, `mouse`, `trial`, `stem`, `video_path` | identity from the file name |
| `reward` | rewarded arm from `meta_trials.csv` |
| `fps` | frame rate used |
| `ok`, `reason` | `False` with a reason (for example `no_arm_dwell_after_gate`) when no decision could be scored |
| `gate_frame`, `gate_reason` | frame the mouse left segment 4 (`seg4_leave`) |
| `entry_side`, `entry_frame`, `entry_ms`, `entry_threshold` | first validated arm entry and the coverage rule that accepted it (`>=95_depth` or the soft fallback) |
| `correct_TF`, `correct_bin` | entry side equals reward side |

!!! warning "Filter on `ok`"
    Rows with `ok == False` still carry `correct_bin = 0`. Always subset on `ok` before computing
    accuracy, or the undecided trials count as errors.

**`events.csv`**: segment 1, segment 4 and junction entry frames and times, commit side and frame,
pre-commit junction exploration (frames and ms), snout probes per arm with their frame counts,
and reversals inside the task window.

**`metrics.csv`**: the per-trial summary used by the figures: `choice_LR`, `latency_choice_ms`,
`stem_latency_ms`, `junction_explore_ms`, `correct_TF`, `probes_L`, `probes_R`.

## Gait (stages 4 to 6)

**`gait_per_stride.csv`**, one row per detected stride (before filtering):

| group | columns |
|---|---|
| identity | `video_path`, `stem`, `day`, `mouse_id`, `sex`, `genotype`, `stride_id` |
| timing | `frame_start`, `frame_end`, `t_start_ms`, `t_end_ms`, `duration_ms`, `fps` |
| scale | `px_per_cm`, `body_length_cm`, `hindL_ind`, `hindR_ind` |
| spatial | `stride_length_cm`, `stride_length_euclid_cm`, `step_length_cm`, `step_width_cm` |
| temporal | `stride_speed_cm_s_mean`, `limb_duty_factor_mean`, `temporal_symmetry`, `angular_velocity_deg_s_mean` |
| body sway | `nose_lat_disp_norm`, `tailbase_lat_disp_norm`, `tailtip_lat_disp_norm` and the matching `*_lat_phase_pct` |
| flags | `valid_stride_TF`, `has_R_end_inside_TF`, `nans_in_stride_TF`, `comment` |

!!! warning "`px_per_cm` stores centimetres per pixel"
    The column holds the conversion factor applied to the stride (for example `0.0383` for a
    26.1 px/cm rig). A value of `1.0` means the stride was never calibrated; the calibration gate
    catches this.

**`gait_per_stride_filtered.csv`**: the same columns after confidence, edge, direction and
angular-velocity filtering. This is the file every downstream analysis reads.

**`gait_per_step.csv`**: one row per hind-paw swing: `paw_side`, swing frames `frame_a`,
`frame_b`, duration, peak paw speed, peak tail-base speed and the stride it belongs to.

**`gait_per_video.csv`**: per-clip summary joining the trial metrics with stride counts and the
per-video mean, SD and n of each gait metric.

**`pose_keypoint_confidence_by_video.csv`**: per-clip mean confidence per keypoint, used by the
confidence filter.

## Statistics (stages 7 to 10)

| file | content |
|---|---|
| `lmm_results_all.csv` | M1 and M2 coefficients, p-values and FDR for every metric and batch |
| `heatmap_*.png` | dual panels: `-log10(p)` and standardized effects, FDR-significant cells marked |
| `regression_roc_curves.png` | ROC curves for T-maze, gait and combined feature sets |
| `classifier_feature_weights.csv` | ranked features with p-values and significance |

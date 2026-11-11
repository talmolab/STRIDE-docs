# Outputs and column glossary

Every file each pipeline writes, and every column of every table, described exactly as the code
computes it. The method behind the numbers is under [How It Works](../how-it-works/decisions.md).

## Output folders

=== "stride tmaze"

    ```
    SUMMARY.md                     what ran, counts per step, px/cm and its source, warnings, videos left out
    tmaze_run.log                  everything printed during the run(s)
    tmaze_run_manifest.yaml        resolved settings (config + command line), models, code version, results
    tmaze_state.json               what is finished (for the next run)
    scale/px_per_cm.json           scale and its source
    scale/distortion_check.csv     the lens distortion report
    scale/px_per_cm.txt, calibration_summary.txt, calibration_readings.csv, *_checkerboard.png, <clip>.json
    pose/<video>.slp               pose files made by this run (videos that had none)
    roi/<video>.rois.yml           ROI polygons (7 regions)
    roi/<video>.preds.v2.best1.slp the kept ROI prediction
    roi/roi_sources.csv            detected | consensus | missing, per video
    roi/roi_consensus_qc/          overlays and montage sheets for clips with a borrowed ROI
    decisions/decisions.csv        one row per trial
    decisions/decisions_clean.csv  the same after the two decision rules
    decisions/events.csv           region entry times, commit, junction exploration, snout probes, reversals
    decisions/metrics.csv          per-trial summary
    gait/gait_per_stride.csv       one row per stride
    gait/gait_per_stride_filtered.csv   the strides that pass the filters
    gait/gait_per_step.csv         one row per hind-paw swing
    gait/gait_per_video.csv        one row per video
    gait/pose_keypoint_confidence_by_video.csv
    gait/metrics_meta.csv, run_params.json
    stats/                         only with mixed_models / classifier switched on
    ```

=== "stride gait"

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
    gait/videos.csv, run_params.json
    ```

=== "run_all.py"

    ```
    <output>/<batch>/decisions/decisions.csv, events.csv, metrics.csv
    <output>/<batch>/metrics_meta.csv, learning_cycle_1.csv, learning_cycle_2.csv
    <output>/<batch>/gait_per_stride.csv, gait_per_stride_filtered.csv, gait_per_step.csv, gait_per_video.csv
    <output>/<batch>/pose_keypoint_confidence_by_video.csv, run_params.json
    <output>/lmm_results_all.csv, lmm_fit_log.csv, heatmap_<model>_<batch>.png / .pdf
    <output>/regression_feature_ranking.csv, regression_auc_vs_k.csv / .png / .pdf, regression_roc_curves.png / .pdf
    ```

Each stage also writes a `<stage>_run_manifest.yaml` next to its tables
([Logs, warnings and exit codes](provenance.md)).

## Decision tables

"Coverage" of a region is the share of the body hull's area that lies inside that region, in
percent ([T-maze decisions](../how-it-works/decisions.md#coverage)).

### `decisions.csv`

One row per analysed video.

| column | computed as |
|---|---|
| `video_path`, `stem` | the video and its file name without extension |
| `day`, `mouse`, `trial` | from a file name of the form `Day<N>_<mouse>_Trial<T>`, else from the metadata, else empty |
| `reward` | the rewarded arm (`L` / `R`) from the metadata; empty when unknown |
| `fps` | the frame rate used |
| `ok` | `True` when an arm entry was found |
| `reason` | empty, or `no_arm_dwell_after_gate` when no arm dwell follows the gate |
| `gate_frame` | the later of frame `round(0.5 x fps)` and the first frame of a run of at least 5 frames in which the coverage of `segment4` is below 30 %. `segment4` is the stem segment next to the junction, so on a trial that starts at the far end of the stem this is in practice the 0.5 s mark |
| `gate_reason` | `seg4_leave` when such a run exists, else `warmup_only` |
| `entry_side` | `L` or `R`: the arm of the chosen arm dwell ([how it is chosen](../how-it-works/decisions.md#arm-entry)); empty when `ok` is `False` |
| `entry_frame`, `entry_ms` | first frame of that dwell, and `1000 x entry_frame / fps` |
| `entry_threshold` | which coverage threshold produced the arm dwells: `>=95% for ≥3f`, the fallback `>=60% for ≥3f`, or `none` |
| `correct_TF`, `correct_bin` | `T` / `1` when `entry_side` equals `reward`, else `F` / `0`. With `stride tmaze` both are **empty** when the reward is unknown. In the batch entry points (`run_all.py`, `stride analyze-decisions`, `stride run`) a trial without a reward row is written as `F` / `0` |

!!! warning "Rows with `ok == False`"
    A row with `ok == False` and a known reward carries `correct_TF = F` and `correct_bin = 0`.
    Subset on `ok` before computing accuracy, or the trials without an arm entry count as errors.

### `decisions_clean.csv`

The same columns after the two decision rules ([details](../how-it-works/decisions.md#decision-rules)).
A row is changed when its entry only cleared the 60 % fallback (`entry_threshold` contains `60%`)
or when `entry_ms` is above the decision window (`decision_rules.window_s`, 150 s). Such a row
gets `ok = False`, `reason = soft_entry_no_commit` or `entry_after_window`, empty `entry_side`,
`entry_frame` and `entry_ms`, `entry_threshold = none`, and `correct_TF = F`, `correct_bin = 0`
(both stay empty when they were empty). Every other cell is written back as it was read.
`decisions.csv`, `events.csv` and `metrics.csv` are not changed by the rules.

### `events.csv`

One row per analysed video.

| column | computed as |
|---|---|
| `day`, `mouse`, `trial`, `stem`, `video_path` | as in `decisions.csv` |
| `enter_seg1_frame`, `enter_seg4_frame`, `enter_junction_frame` | first frame in which the coverage of `segment1`, `segment4` or `junction` is at least 50 %; empty when it never is |
| `enter_seg1_ms`, `enter_seg4_ms`, `enter_junction_ms` | `1000 x frame / fps`; empty when the frame is empty or is frame 0 |
| `commit_side`, `commit_frame`, `commit_ms` | `entry_side`, `entry_frame`, `entry_ms` of `decisions.csv` |
| `ok` | as in `decisions.csv` |
| `junction_explore_frames_precommit` | frames before `commit_frame` that lie in junction dwells: runs of at least 3 frames (1-frame gaps bridged) with junction coverage of at least 95 %. 0 when there is no commit |
| `junction_explore_ms_precommit` | `1000 x` that count `/ fps` |
| `probes_L`, `probes_R` | number of snout-probe episodes into that arm before `commit_frame`. A probe frame has the gap-filled snout inside the arm polygon, coverage of that arm below 40 % and junction coverage of at least 30 %; an episode is a run of at least 2 consecutive probe frames. 0 when there is no commit |
| `probe_frames_L`, `probe_frames_R` | number of probe frames into that arm before `commit_frame` |
| `n_reversals`, `reversal_frames` | from a position score per frame: the region with the highest coverage among `segment4` (0), `segment3` (1), `segment2` (2), `segment1` (3) and `junction` (4), when that coverage is at least 30 %; gaps of up to 5 frames are interpolated and an 11-frame median filter is applied. Between `gate_frame` and `commit_frame` the highest score reached so far is tracked; a frame whose score is at least 1 below it is a reversal frame, and `n_reversals` counts the episodes of such frames (an episode ends when a new highest score is reached). With these scores a run from `segment1` through `segment2`, `segment3` and `segment4` to the junction passes through lower scores, so a straight run registers one reversal. 0 when there is no commit |
| `reversal_ms` | `1000 x reversal_frames / fps` |
| `coverage_thr_rule` | `entry_threshold` of `decisions.csv` |

### `metrics.csv`

One row per analysed video, built from `events.csv` and `decisions.csv`.

| column | computed as |
|---|---|
| `day`, `mouse`, `trial`, `stem`, `video_path` | as above |
| `choice_LR` | `commit_side` |
| `latency_choice_ms` | `commit_ms - enter_seg4_ms` |
| `stem_latency_ms` | `enter_junction_ms - enter_seg1_ms` |
| `junction_explore_ms` | `junction_explore_ms_precommit` |
| `reward`, `correct_TF`, `correct_bin` | from `decisions.csv` |
| `probes_L`, `probes_R`, `probe_frames_L`, `probe_frames_R` | from `events.csv` |

## Gait tables

Coordinates are divided by px/cm; missing values are filled per coordinate (nearest neighbour,
extrapolated at the ends); a median filter (3 frames) and a moving average (5 frames) follow. The
speed of a keypoint is its frame-to-frame displacement times the frame rate. The **body frame**
of a frame has the tail base at the origin and is rotated by the angle of the tail base to snout
axis in the image; the rotation puts the snout on the +x axis when the animal heads along the
image x-axis. Walking, swings and strides are defined in
[Stride detection and gait measures](../how-it-works/gait.md).

### `gait_per_stride.csv`

One row per stride. `s0` = `frame_start`, `s1` = `frame_end`.

| column | computed as |
|---|---|
| `video_path`, `stem` | the video and its file name without extension |
| `day`, `mouse_id`, `sex`, `genotype` | from the file name / metadata; empty when unknown |
| `stride_id` | index of the stride within the video, from 0 |
| `frame_start`, `frame_end` | `s0` = the frame after a left hind-paw swing ends; `s1` = the last frame of the next left swing |
| `t_start_ms`, `t_end_ms` | `s0 / fps` and `(s1 - 1) / fps`, in ms |
| `duration_ms` | `(s1 - s0) / fps`, in ms |
| `fps` | frame rate read from the video (the fallback value only when it cannot be read) |
| `px_per_cm` | **cm per pixel** (1 / scale), despite the name. `1.0` means the scale of that clip could not be derived |
| `body_length_cm` | mean snout to tail base distance over the whole video |
| `hindL_ind`, `hindR_ind` | node index of the hind-paw landmark used on each side. With two landmarks per side the one with the higher (share of walking frames with a speed) x (median of its 10 highest speeds while walking) is taken, per video and per side |
| `angular_velocity_deg_s_mean` | mean over the stride frames of the rate of change of the body-axis angle, deg/s |
| `stride_speed_cm_s_mean` | mean tail-base speed over the stride frames |
| `limb_duty_factor_mean` | mean of the left and right duty factor; a paw's duty factor is the share of the stride frames in which it is not in a swing |
| `temporal_symmetry` | (left duty - right duty) / (left duty + right duty) |
| `step_length_cm` | body-frame x of the right paw at the right strike inside the stride, minus body-frame x of the left paw at the left strike before the stride (each in the body frame of its own frame). It depends on which landmark was picked on each side |
| `step_width_cm` | distance of the right paw at that right strike from the segment between the left paw at toe-off and at strike of the left swing before the stride, in body-frame coordinates |
| `stride_length_cm` | body-frame x of the left paw at the strike that ends the stride, minus its body-frame x at toe-off of that swing. This is the paw's excursion relative to the body, not the distance covered on the ground, and it is signed |
| `stride_length_euclid_cm` | length of that body-frame displacement (unsigned) |
| `nose_lat_disp_norm`, `tailbase_lat_disp_norm`, `tailtip_lat_disp_norm` | range (max - min) over the stride frames of the keypoint's offset perpendicular to the straight line from the tail base at `s0` to the tail base at `s1 - 1`, divided by `body_length_cm` |
| `nose_lat_phase_pct`, `tailbase_lat_phase_pct`, `tailtip_lat_phase_pct` | where in the stride (0 to 100 %) that perpendicular offset is largest (cubic spline through the frames, 101 points) |
| `valid_stride_TF`, `has_R_end_inside_TF` | a right swing ends inside the stride (always true for the rows written) |
| `nans_in_stride_TF` | any keypoint coordinate of the stride frames is not finite after the filling step (true for every stride when a keypoint was never tracked or the skeleton has no tail tip) |
| `comment` | empty, or why the stride is not valid |

`step_length_cm`, `step_width_cm` and both stride lengths are measured relative to the body and
are sensitive to the body's yaw; `stride_length_cm` and `step_length_cm` assume the animal walks
along the image x-axis. With `stride gait`, metadata columns are appended at the end.

### `gait_per_stride_filtered.csv`

The same columns, for the strides that pass, in this order:

1. **Confidence**: the video's mean keypoint confidence (mean of the `avg_*` columns of the
   confidence table) is above 0.3.
2. **Edges**: the first and the last stride of every video are removed (a video with one or two
   strides keeps none).
3. **Turning**: `angular_velocity_deg_s_mean` within -20 to +20 deg/s.
4. **Forward direction**: the sign of the median `stride_length_cm` of the remaining strides of
   the run is taken as forward; strides with the other sign are removed. With the body frame as
   defined above, `stride_length_cm` is positive while the body axis is within 45 degrees of the
   image x-axis, so this filter keeps strides along the image x-axis (the maze stem, when it runs
   left to right) and removes backward steps and strides across it.

### `gait_per_step.csv`

One row per hind-paw swing.

| column | computed as |
|---|---|
| `video_path`, `stem`, `day`, `mouse_id`, `sex`, `genotype` | as above |
| `step_id` | index of the swing within the video |
| `paw_side` | `L` or `R` |
| `frame_a`, `frame_b` | first and last swing frame |
| `t_a_ms`, `t_b_ms` | their times in ms |
| `dur_ms` | `(frame_b - frame_a + 1) / fps`, in ms |
| `peak_speed_cm_s` | peak paw speed in the swing |
| `peak_tailbase_speed_cm_s` | peak tail-base speed in the same frames |
| `within_stride_id` | the stride in which the swing ends, -1 when none |

### `gait_per_video.csv`

One row per video, over all strides of the video, before the filters.

| column | computed as |
|---|---|
| `video_path`, `stem`, `day`, `mouse_id`, `sex`, `genotype` | as above |
| `choice_LR`, `latency_choice_ms`, `stem_latency_ms`, `junction_explore_ms`, `reward`, `correct_TF`, `correct_bin`, `probes_L`, `probes_R` | copied from the decision step's per-trial table; empty with `stride gait` |
| `hindL_ind`, `hindR_ind` | as in the per-stride table |
| `n_strides`, `n_valid_strides`, `frac_valid` | strides of the video, those with `valid_stride_TF`, and their ratio |
| `<column>_mean`, `<column>_sd`, `<column>_n` | mean, standard deviation and count over the video's strides, for `stride_speed_cm_s_mean`, `angular_velocity_deg_s_mean`, `limb_duty_factor_mean`, `temporal_symmetry`, `step_length_cm`, `step_width_cm`, `stride_length_cm`, the three `*_lat_disp_norm` and the three `*_lat_phase_pct` columns |

### `pose_keypoint_confidence_by_video.csv`

One row per video: `stem`, `video_path`, `frames_total`, `frames_kept`, and `avg_<keypoint>` = the
mean score of that keypoint over the frames that have an instance (0 when the keypoint has no
score). For the 15-node skeleton the columns are `avg_snout`, `avg_mouth`, `avg_forepawR2`,
`avg_forepawR1`, `avg_forepawL1`, `avg_forepawL2`, `avg_hindpawR2`, `avg_hindpawR1`,
`avg_hindpawL2`, `avg_hindpawL1`, `avg_tailbase`, `avg_tail1`, `avg_tail2`, `avg_tail3`,
`avg_tailtip`. For another skeleton (`stride gait` with `keypoints:`) the columns are that
skeleton's nodes.

### Other files under `gait/`

- `metrics_meta.csv` (`stride tmaze`) / `videos.csv` (`stride gait`): the list of videos the gait
  step ran on, with the id columns.
- `run_params.json`: the stride detection parameters used.

## Scale and ROI records

| file | content |
|---|---|
| `scale/px_per_cm.json` | `px_per_cm` (empty when derived per video), `source`, and, where relevant, `maze_width_cm` and why the ChArUco measurement gave no value |
| `scale/calibration_readings.csv` | one row per measured clip: `clip`, `px_per_cm`, `usable`, `corners` (ChArUco corners found), `anisotropy`, `reprojection_px`, `frame_time_s`, `px_per_square`, `rail_px_per_cm` (the optional cross-check) |
| `scale/distortion_check.csv` | one row per sampled video: `video`, `line_straightness` (RMS deviation of the board edges from straight lines, px), `spacing_uniformity` (coefficient of variation of the corner spacing), `reprojection_error` (px), `needs_undistortion` (any of the three above its threshold: 2 px, 0.05, 5 px; empty when no board was found), `frames_detected` |
| `roi/roi_sources.csv` | one row per video: `stem`, `video_path`, `roi_source` (`detected`, `consensus` or `missing`), `roi_file`, `note` |

## Statistics outputs (optional steps)

Written under `stats/` by `stride tmaze` and under `<output>/` by `run_all.py`.

| file | columns |
|---|---|
| `lmm_results_all.csv` | one row per metric, model and batch: `variable`, `model`, `n_obs`, `n_mice`, `converged`, and per non-reference genotype `<G>_coef` (in the metric's units), `<G>_se`, `<G>_pval`, `<G>_coef_z` (standardized), `<G>_fdr`; `batch`, `batch_key`, `label` |
| `lmm_fit_log.csv` | every fit, kept or not: `batch_key`, `variable`, `model`, `status`, `reason`, `optimizer_converged` |
| `heatmap_<model>_<batch>.png` / `.pdf` | `-log10(p)` and standardized coefficients per metric |
| `regression_feature_ranking.csv` | `feature`, `label`, `category`, `p_value`, `source` (`LMM`, `t-test` or `N/A`), `direction`, `significant` (p below 0.05) |
| `regression_auc_vs_k.csv` | `k`, `auc_mean`, `auc_std`, `top_feature` |
| `regression_auc_vs_k.png` / `.pdf`, `regression_roc_curves.png` / `.pdf` | the classifier figures |

`run_all.py` stage 3 additionally writes `metrics_meta.csv` (`metrics.csv` plus the sex and
genotype columns of `metadata_csv`) and `learning_cycle_1.csv` / `learning_cycle_2.csv`. `learn_1`
is the first day on which a mouse's rewarded arm differs from the day before. Cycle 1 holds the
rows of `metrics_meta.csv` from day 1 on that are before `learn_1` (all of them when the reward
never changes); cycle 2 holds the rows from `learn_1` on. Added columns: `mouseID`, `day_num`,
`reward_changed`, `learn_1`, `learn_1_num`, `learning_cycle`.

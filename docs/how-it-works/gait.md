# Stride detection and gait metrics

Stages 4 to 6 follow the stride-level approach of Sheppard et al. (2022) for open-field pose
data, adapted to a corridor. Every threshold is in `config.yaml` under `stride_detection` and
`stride_filtering`.

## Preprocessing

1. Keypoints are read from the `.slp`; missing values are interpolated linearly.
2. Coordinates are converted to centimetres with the batch scale: `px_per_cm_override` if set,
   otherwise the width of `segment2` in the region file divided by the 10 cm corridor width.
3. Traces are smoothed with a 3-frame median filter and a 5-frame moving average; velocities are
   central differences at the video frame rate.
4. **Body length** is the mean snout-to-tail-base distance over the clip. It scales the sway
   metrics and enters the mixed models as a covariate.
5. Of the two tracked nodes per hind paw, the better tracked one is chosen per video
   (`hindL_ind`, `hindR_ind` in the output).

## Walking and swings

A frame counts as **walking** when tail-base speed exceeds 5 cm/s. Within walking, a hind paw is
in **swing** while its speed exceeds 8 cm/s; a swing is kept when it lasts at least 3 frames and
its peak speed reaches 15 cm/s. The end of a swing is a **foot strike**.

## Strides

A stride runs from one foot strike of the reference hind paw to the next. Each stride yields one
row in `gait_per_stride.csv`; each swing of either hind paw yields one row in `gait_per_step.csv`.

| metric | definition |
|---|---|
| `duration_ms` | time between consecutive foot strikes of the same paw |
| `stride_length_cm` | tail-base displacement along the direction of travel over the stride (`_euclid` is the straight-line version) |
| `step_length_cm` | separation of the two hind-paw strikes along the direction of travel |
| `step_width_cm` | separation of the two hind-paw strikes across the direction of travel |
| `stride_speed_cm_s_mean` | mean tail-base speed within the stride |
| `limb_duty_factor_mean` | fraction of the stride the paw is in stance |
| `temporal_symmetry` | timing of the contralateral strike within the cycle, 0 = symmetric |
| `angular_velocity_deg_s_mean` | mean heading change rate, the straight-walking criterion |
| `nose_lat_disp_norm`, `tailbase_lat_disp_norm`, `tailtip_lat_disp_norm` | peak-to-peak lateral excursion of the point across the stride's travel axis, divided by body length |
| `*_lat_phase_pct` | where in the cycle (0 to 100%) that lateral excursion peaks, from a cubic spline on a 101-point grid |

Each row also records the scale used (`px_per_cm`, stored as centimetres per pixel) and three
validity flags: whether both strikes fell inside the region file's maze, whether the stride
contains NaN keypoints, and a free-text comment.

## Filtering (stage 6)

Applied in this order, each configurable:

1. **Confidence**: videos whose mean keypoint confidence is below 0.3 are dropped.
2. **Edges**: the first and last stride of every continuous walking run are removed; they are
   accelerations and decelerations, not steady gait.
3. **Direction**: backward strides are removed. Their kinematics are a different behaviour
   (about twice slower, two and a half times more sway, five times more turning); the forward
   direction is detected per batch from the sign of the median stride length.
4. **Turning**: strides with mean angular velocity outside −20 to +20 deg/s are removed for the
   straight-walking analyses.

The table that leaves this stage, `gait_per_stride_filtered.csv`, is the canonical gait file.

# Stride detection and gait measures

The gait step follows the stride-level approach of
[Sheppard et al. (2022)](https://doi.org/10.1016/j.celrep.2022.110231) and
[Bellardita & Kiehn (2015)](https://doi.org/10.1016/j.cub.2015.04.005). The thresholds are the
`stride_detection` and `stride_filtering` settings
([Configuration reference](../user-guide/configuration.md#analysis-settings)); every output
column is described in the [column glossary](../user-guide/outputs.md#gait-tables).

## Preprocessing

1. Keypoints are read from the pose file and divided by the scale (px/cm).
2. Missing values are filled per coordinate (nearest neighbour, extrapolated at the ends).
3. A 3-frame median filter and a 5-frame moving average follow.
4. The speed of a keypoint is its frame-to-frame displacement times the frame rate.
5. **Body length** is the mean snout to tail base distance over the video.
6. Of the two tracked landmarks per hind paw, one is picked per video and per side: the one with
   the higher (share of walking frames with a speed) x (median of its 10 highest speeds while
   walking). `hindL_ind` and `hindR_ind` record the choice.

## The body frame

The body frame of a frame has the tail base at the origin and is rotated by the angle of the
tail base to snout axis in the image. The rotation puts the snout on the +x axis when the animal
heads along the image x-axis. The step and stride measures below are taken in this frame, so they
are relative to the body, they assume walking along the image x-axis, and they are sensitive to
the body's yaw.

## Walking, swings and strides

- **Walking**: tail-base speed of at least 5 cm/s.
- **Swing** of a hind paw: consecutive walking frames with paw speed of at least 8 cm/s, at least
  3 frames long, with a peak paw speed of at least 15 cm/s that is not below the peak tail-base
  speed in the same frames.
- **Stride**: from the frame after a left hind-paw swing ends (`frame_start`) to the last frame
  of the next left swing (`frame_end`); kept when a right swing ends inside it.

Each stride is one row of `gait_per_stride.csv`; each swing of either hind paw is one row of
`gait_per_step.csv`.

## The measures, as computed

| measure | definition |
|---|---|
| `duration_ms` | `(frame_end - frame_start) / fps` |
| `stride_length_cm` | body-frame x of the left paw at the strike that ends the stride, minus its body-frame x at toe-off of that swing: the paw's excursion relative to the body, not the distance covered on the ground; signed |
| `stride_length_euclid_cm` | the length of that body-frame displacement |
| `step_length_cm` | body-frame x of the right paw at the right strike inside the stride, minus body-frame x of the left paw at the left strike before the stride; it depends on which landmark was picked on each side |
| `step_width_cm` | distance of the right paw at that strike from the segment between the left paw at toe-off and at strike of the left swing before the stride, in the body frame |
| `stride_speed_cm_s_mean` | mean tail-base speed over the stride frames |
| `limb_duty_factor_mean` | mean of the left and right duty factor: the share of the stride frames in which a paw is not in a swing |
| `temporal_symmetry` | (left duty - right duty) / (left duty + right duty) |
| `angular_velocity_deg_s_mean` | mean rate of change of the body-axis angle |
| `*_lat_disp_norm` | range of the keypoint's offset perpendicular to the straight line between the tail base at the first and the last stride frame, divided by body length |
| `*_lat_phase_pct` | where in the stride (0 to 100 %) that offset is largest |

The lateral displacement and phase measures use the line of travel of the stride, not the body
frame.

## Filtering

`gait_per_stride_filtered.csv` holds the strides that pass, in this order:

1. **Confidence**: the video's mean keypoint confidence is above 0.3.
2. **Edges**: the first and the last stride of every video are removed.
3. **Turning**: mean angular velocity within -20 to +20 deg/s.
4. **Forward direction**: the sign of the median `stride_length_cm` of the remaining strides of
   the run is taken as forward; strides with the other sign are removed. In the body frame as
   defined above `stride_length_cm` is positive while the body axis is within 45 degrees of the
   image x-axis, so this filter keeps strides along the image x-axis (the maze stem, when it runs
   left to right) and removes backward steps and strides across it.

`stride analyze-gait` is a quicker two-step filter (confidence and edges only) and keeps more
strides than the pipeline's filter.

## What the gait step warns about

Below 100 fps, below 20 px/cm, a scale that could not be derived for a clip, and a maze stem
(`stride tmaze`) or walking direction (`stride gait`) that does not run along the image x-axis.
The values are written unchanged ([Logs, warnings and exit codes](../user-guide/provenance.md#warnings)).

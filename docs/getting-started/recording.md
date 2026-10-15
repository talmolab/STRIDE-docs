# Recording requirements

STRIDE was validated on three rigs filming the same 10 cm corridor T-maze. The numbers below are
what separated usable gait from unusable gait; cognitive scoring is far less demanding.

| Requirement | Gait | T-maze decisions |
|---|---|---|
| Scale on the maze floor | **20 to 25 px/cm or more** | any resolution where the regions are visible |
| Frame rate | **100 fps or more**, 120 preferred | 30 fps and up |
| Compression | low, global shutter preferred | not critical |
| View | single camera, perpendicular to the floor (bottom-up through a transparent floor, or top-down) | same |
| Calibration | a ChArUco board in view, or a verified px/cm per rig | not required |

## Why these numbers

- **Scale.** At 10 px/cm a paw is a 3-pixel blob, and stride length and body length pick up systematic artefacts. The gait of the paper's 1024 x 1024 industrial-camera batch was excluded for that reason; its T-maze decisions were kept.
- **Frame rate.** Swing phases are short (10th percentile about 60 ms). A swing must span at least 3 frames to be detected and 5 frames to be measured well. At 120 fps 97% of swings clear the 5-frame bar; at 60 fps 27% are lost; at 30 fps gait cannot be resolved.
- **Compression.** Heavy H.265 compression raised the tracking noise floor of a 4K action camera to about 2 mm, against 0.2 mm for a less compressed 4K camera at the same scale. Sway metrics of a few millimetres need the lower floor.

## Calibration

Stage 1 looks for a ChArUco board (default 14 x 9 squares, 20 mm squares, 15 mm markers,
5x5 dictionary) in sampled frames to measure lens distortion and the scale. If your rig has a
pinned, externally verified px/cm, set `px_per_cm_override` for the batch; the calibration gate
then checks that every stride was converted with that constant.

## Naming

Clips are named `Day{N}_{mouse}_Trial{k}.MP4` (`.mp4` works too; discovery is recursive and
case-insensitive). The stem is parsed for day, mouse and trial, and the decision stage joins each
clip to `meta_trials.csv` on day and mouse. Mouse ids need not be numeric: cage-position ids such
as `M3-2` or genotype-prefixed ids such as `WT_1_1` are accepted.

Rename **before** pose inference when you can. The pose file embeds the video name from inference
time; since the fix merged as pull request #8 the gait stage takes the stem from the file on disk,
so a later rename no longer breaks gait, and a stem mismatch between the stride table and the
confidence table stops the run instead of silently dropping strides.

# Recording requirements

STRIDE was validated on three rigs filming the same 10 cm corridor T-maze. The numbers below are
what separated usable gait from unusable gait; cognitive scoring is far less demanding.

| Requirement | Gait | T-maze decisions |
|---|---|---|
| Paws visible | **yes**: camera below a transparent floor | not needed |
| Scale on the floor | **20 px/cm or more** | any resolution where the regions are visible |
| Frame rate | **100 fps or more** | 30 fps and up |
| Direction | **the maze stem (or the walking direction) runs left to right in the image** | any |
| Compression | low, global shutter preferred | not critical |
| Scale reference | a ChArUco board in view, or a px/cm you measured | not required |

The [rig checker](https://talmolab.github.io/STRIDE-page/tools/rig-checker/) on the project page
reads a video in the browser and reports whether a set-up meets these
([Web tools](../web-tools.md)).

## What STRIDE does when a recording does not meet them

It warns, and it never changes a value:

- below 100 fps or below 20 px/cm the gait step prints a warning and still writes the tables;
- when the maze stem (`stride tmaze`) or the body axis while walking (`stride gait`) does not run
  along the image x-axis it prints a warning. The step and stride measures and the
  forward-direction filter assume that direction; STRIDE does not correct for it
  ([Stride detection and gait measures](../how-it-works/gait.md));
- the lens distortion check is a report only. STRIDE does not undistort.

## Why these numbers

- **Scale.** At 10 px/cm a paw is a 3-pixel blob, and stride length and body length pick up systematic artefacts. The gait of the paper's 1024 x 1024 industrial-camera batch was excluded for that reason; its T-maze decisions were kept.
- **Frame rate.** Swing phases are short (10th percentile about 60 ms). A swing must span at least 3 frames to be detected and 5 frames to be measured well. At 120 fps 97% of swings clear the 5-frame bar; at 60 fps 27% are lost; at 30 fps gait cannot be resolved.
- **Compression.** Heavy H.265 compression raised the tracking noise floor of a 4K action camera to about 2 mm, against 0.2 mm for a less compressed 4K camera at the same scale. Sway metrics of a few millimetres need the lower floor.

## Scale reference

`stride calibrate`, and the `calibrate` step of both pipelines, measure px/cm from a ChArUco
board lying in view of the camera (default 14 x 9 squares, 20 mm squares, 15 mm markers,
`DICT_5X5_250`). The board must lie flat in the plane the animal walks on and be visible during
the first 10 s of at least three clips. Details: [Scale calibration](../pipelines/calibration.md).

Without a board, give the scale yourself (`--px-per-cm`). `stride tmaze` can also fall back on
the corridor width of each video's ROI; `stride gait` has no such fallback and stops.

## File names

`stride tmaze` and `stride gait` accept **any** video file names; they only have to be unique
within a run, because every table is keyed by the file name.

Names of the form `Day<N>_<mouse>_Trial<T>.mp4` are recognised: day, mouse and trial are then
filled from the name. The stage-by-stage batch runner (`run_all.py`, `stride analyze-decisions`,
`stride run`) requires that form and a reward table
([Stage-by-stage batch runs](../tutorials/batch-runs.md)).

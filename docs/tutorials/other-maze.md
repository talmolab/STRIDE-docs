# Gait on a different maze

`stride gait` does not know what a T-maze is. It needs pose tracking, the scale and the frame
rate.

## What to provide

| input | requirement |
|---|---|
| pose | one `.slp` per video, or your pose model. Needed keypoints: `snout`, `tailbase`, at least one landmark per hind paw; `tailtip` is optional. Other node names are mapped with `keypoints:` |
| scale | `--px-per-cm` from your own measurement, or a ChArUco board in view |
| recording | paws visible from below, the animal walking left-right in the image, 100 fps and 20 px/cm or more ([Recording requirements](../getting-started/recording.md)) |
| metadata | optional CSV keyed by video file name; its columns are carried into the tables |

## Steps

1. Train or fine-tune a SLEAP pose model for your rig, or use pose files you already have.
2. Write the config and map your node names:

    ```bash
    stride init-config gait -o my_gait.yaml
    ```

    ```yaml
    keypoints:
      nose: snout
      tail_base: tailbase
      left_hindpaw: hindpawL1
      right_hindpaw: hindpawR1
    ```

3. Run:

    ```bash
    stride gait --config my_gait.yaml
    ```

4. Read `SUMMARY.md`. The `[direction]` line gives the share of walking frames in which the body
   axis is within 45 degrees of the image x-axis; when it is below half, rotate the camera or the
   videos so that the runway is horizontal.

## What you do not get without the T

No trial phase, no maze region and no decision table: those need the maze ROI
([STRIDE T-maze](../pipelines/tmaze.md)). The genotype statistics are not part of `stride gait`.
The stride table, the filters and the measures are the same.

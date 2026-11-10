# Pose and ROI models

Two SLEAP top-down models (centroid + centered instance) run on every clip. STRIDE ships
neither: you train them for your rig and pass their folders (`--pose-model`, `--roi-model`).

## Pose model

Tracks one mouse with 15 keypoints at every frame: `snout`, `mouth`, two nodes per paw
(`forepawL1`, `forepawL2`, `forepawR1`, `forepawR2`, `hindpawL1`, `hindpawL2`, `hindpawR1`,
`hindpawR2`), `tailbase`, `tail1`, `tail2`, `tail3`, `tailtip`. Output is one `.slp` per video
with the same name. Which of them each step reads: [Inputs and file layout](../user-guide/inputs.md#keypoints-stride-reads).

## ROI model

Predicts the 23 maze keypoints: 16 corners and 7 centroids of the regions `segment1` to
`segment4`, `junction`, `arm_left`, `arm_right`
([node names](../user-guide/inputs.md#what-the-roi-model-must-provide)).

### Choosing the frame and the instance

1. **Slice pass.** The model is run on 10 frames sampled evenly from the middle of the video
   (20 % to 80 % of its length), not from its start, where a hand or the mouse being placed often
   hides a corner.
2. **Every instance is evaluated**, not only the first one of a frame. The model sometimes
   returns a second, shifted, low-score instance with missing corners, and it is not always
   listed second.
3. **A frame qualifies** when one instance has all 23 nodes, or, failing that, all 16 corners.
   A missing centroid is acceptable; a missing corner is not.
4. **The best is kept.** Frames with all 23 nodes take precedence over corner-only frames; within
   a class the choice is by mean point score over the 23 nodes, then mean point score over the
   required nodes, then instance score. The kept one-frame file (`<video>.preds.v2.best1.slp`)
   carries only the chosen instance.
5. **Full pass.** When none of the sampled frames qualifies, the model is run on the whole video
   and the same selection is applied (`roi.full_pass`).

## From keypoints to regions

The 16 corners are turned into seven polygons: `arm_right`, `junction`, `arm_left`, `segment1`
to `segment4`, written as `<video>.rois.yml`. Neighbouring regions share their corners. The
region file is what the decision step reads. `stride convert-roi` does this conversion on its
own.

## Consensus ROI

When no frame of a clip shows all 16 corners, something (a calibration board, a hand) covered a
corner for the whole clip. The maze and the camera do not move within a session, so the clip gets
the per-vertex median of the ROIs detected in the other clips of the same folder
(`roi.session: all` treats the whole run as one session).

- The consensus is refused, and the clip left out, when the detected ROIs of the session differ
  by more than `roi.drift_max_px` (25 px) at any vertex: the maze or the camera moved.
- Borrowed ROI files carry a `# consensus_roi_of: N clips` line and are never used as donors.
- `roi/roi_sources.csv` lists, per video, whether its ROI was `detected`, `consensus` or `missing`.
- `roi/roi_consensus_qc/` holds an overlay per recovered clip and montage sheets. Open them and
  confirm the ROI lands on the maze.

`stride propagate-roi` applies the same consensus to a folder on its own. There the guard is a
warning by default (`--refuse-on-drift` makes it a refusal).

## Training your own

A new camera, maze colour or floor material usually needs its own models, or fine-tuning on
labelled frames in SLEAP. The keypoint and region names must stay as above for `stride tmaze` to
find them; `stride gait` can map other pose node names
([STRIDE Gait](../pipelines/gait.md#keypoints)).

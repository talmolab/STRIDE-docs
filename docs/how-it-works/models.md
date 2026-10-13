# Pose and ROI models

Two SLEAP top-down models (centroid + centered-instance) run on every clip.

## Pose model

Tracks one mouse with 15 keypoints at every frame: `snout`, `mouth`, two nodes per paw
(`forepawL1`, `forepawL2`, `forepawR1`, `forepawR2`, `hindpawL1`, `hindpawL2`, `hindpawR1`,
`hindpawR2`), `tailbase`, `tail1`, `tail2`, `tail3`, `tailtip`. Output is one `.slp` per video
with the same stem. Typical completeness on a well-lit rig is about 14 of 15 nodes per frame.

## ROI model

Predicts 23 maze keypoints (corners and centroids of the stem segments, junction and arms). The
stage samples 10 frames from the middle of the clip, scores each frame, and keeps the best one
as a one-frame `.slp`. A frame passes when all 16 corner nodes are present; centroids are derived
downstream and are not required.

!!! note "Best instance, not first instance"
    The model sometimes returns a second, shifted low-score instance next to the real maze.
    The selector evaluates every instance in a frame and keeps the best passing one. Earlier
    versions looked only at the first instance and could reject whole days as "no passing
    frame"; use the current scripts.

## From keypoints to regions

`convert-roi` turns the 23 keypoints into seven polygons: `arm_right`, `junction`, `arm_left`,
`segment1` to `segment4`. The region file is what both analysis modules read.

## When a clip has no passing frame

The maze does not move within a session, so a clip whose ROI detection fails can take the
session's consensus: the per-vertex median of the clips that passed, written with a provenance
line (`consensus_roi_of: N clips; max per-vertex spread X px`). A large spread means the camera
or maze moved and the consensus is not safe. This propagation is a lab tool outside the package;
whatever you use, keep the provenance line so the ROI coverage gate can tell direct detections
from borrowed ones.

## Training your own

The deposited models were fine-tuned on the paper's rigs. A new camera, maze colour or floor
material usually needs fine-tuning on a few hundred labelled frames in SLEAP; the skeleton and
region names must stay as above for STRIDE to find them.

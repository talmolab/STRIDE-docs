# Process a recording set end to end

One recording set, from videos to tables, with `stride tmaze`. The paths are placeholders.

## 1. Check the recording

Paws visible from below, at least 100 fps and 20 px/cm for the gait tables, the maze stem running
left to right in the image ([Recording requirements](../getting-started/recording.md)). The
[rig checker](https://talmolab.github.io/STRIDE-page/tools/rig-checker/) reads a video in the
browser and reports this.

## 2. Write the config

```bash
stride init-config tmaze -o my_tmaze.yaml
```

Fill in `videos`, `output`, `pose_model`, `roi_model`, and, if you have it, `metadata` (the
rewarded arm per trial). Leave `scale.px_per_cm` empty when a ChArUco board is in view; set it
when you measured the scale yourself.

## 3. Run

```bash
stride tmaze --config my_tmaze.yaml
```

Inference is the slow part. To spread pose inference over several GPUs first, see
[Multi-GPU inference](multi-gpu.md); `stride tmaze` then finds the pose files and skips that step
for them.

## 4. Read `SUMMARY.md`

It says how many videos were found and analysed, which were left out and why, the px/cm used and
where it came from, and every warning.

- **Videos left out**: fix the cause (no pose file, no ROI) and run the same command again.
  Finished work is not repeated.
- **Consensus ROI**: open the montage under `roi/roi_consensus_qc/` and confirm each borrowed ROI
  lands on the maze.
- **Scale**: when the ChArUco measurement was refused, read `scale/calibration_summary.txt`.

## 5. Check the gait scale (optional)

```bash
python -m stride.stages.safety_checks --gait /data/stride_out/gait/gait_per_stride_filtered.csv --expect-pxcm 26.1
```

Uniform scale equal to the value you expect, plausible body length, uniform fps
([Safety checks](../user-guide/safety-checks.md)).

## 6. Use the tables

`decisions/decisions_clean.csv` for choices, `decisions/metrics.csv` for the per-trial summary,
`gait/gait_per_stride_filtered.csv` for gait
([column glossary](../user-guide/outputs.md)).

## 7. Record what happened

The run manifests record the code and the settings. Keep a short note next to the outputs for
the judgement calls: clips that use the consensus ROI, trials you excluded and why.

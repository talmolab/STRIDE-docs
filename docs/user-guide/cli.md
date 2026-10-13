# Command reference

## `stride` (Click CLI)

```bash
stride --help
```

| command | purpose | main options |
|---|---|---|
| `stride run` | distortion check, pose and ROI inference, then decisions, for one video directory | `-i`, `-o`, `--meta`, `--workers`, `--skip-distortion`, `--skip-pose`, `--skip-roi`, `--batch-size`, `--device` |
| `stride check-distortion` | ChArUco-based lens distortion report | `-i`, `-o`, `--num-frames` (10), `--squares-x` (14), `--squares-y` (9) |
| `stride pose-inference` | full-video 15-keypoint pose, one `.slp` per video | `-i`, `-o`, `--model` (twice: centroid, centered_instance), `--batch-size`, `--device` or `--gpu`, `--worker-id`, `--num-workers`, `--overwrite` |
| `stride roi-inference` | best-frame ROI keypoints, one one-frame `.slp` per video | `-i`, `-o`, `--model`, `--device` or `--gpu`, `--worker-id`, `--num-workers`, `--overwrite` |
| `stride convert-roi` | ROI `.slp` to 7-polygon `.yml` | `-i`, `-o`, `--overwrite` |
| `stride analyze-decisions` | T-maze decisions, events and metrics | `--videos`, `--yml`, `--meta`, `-o` (`decisions.csv`), `--workers` |
| `stride analyze-gait` | confidence and edge filtering of a per-stride table | `-i`, `--confidence`, `-o`, `--threshold` (0.3) |
| `stride count-frames` | frame counts per video | `-i` |
| `stride status` | progress from a checkpoint file | `--checkpoint` (`progress.log`) |

Workers skip videos whose output already exists unless `--overwrite` is given, so a crashed run
can be relaunched as-is.

## Config-driven stage runners

Each reads `config.yaml` and processes every batch, or one with `--batch`:

```bash
python -m stride.stages.run_inference        --config config.yaml [--batch KEY] [--skip-pose] [--skip-roi] [--skip-distortion]
python -m stride.stages.run_tmaze            --config config.yaml [--batch KEY] [--output DIR]
python -m stride.stages.run_gait_extraction  --config config.yaml [--batch KEY] [--output DIR]
python -m stride.stages.gait_lmm             --config config.yaml [--output DIR] [--trial-phase task|all]
python -m stride.stages.classification       --config config.yaml [--output DIR]
```

## Orchestrator

```bash
python run_all.py [--config config.yaml] [--batch KEY] [--stages 1-2,3,4-6,7-9,10]
```

Resumable: a stage whose outputs exist is skipped.

## Safety checks

```bash
python -m stride.stages.safety_checks --roi-coverage VIDEO_DIR [--expect-count N]
python -m stride.stages.safety_checks --gait gait_per_stride_filtered.csv [--expect-pxcm 26.08] [--strict]
python -m stride.stages.safety_checks --stamp GATE --vid VIDEO_DIR
python -m stride.stages.safety_checks --require GATE1,GATE2 --vid VIDEO_DIR
```

Exit code 2 on any FAIL, 1 on a WARN with `--strict`, 0 otherwise. Details in
[Safety checks and gates](safety-checks.md).

# Quickstart

## 1. Describe your batch

Add an entry under `batches:` in `config.yaml` (see [Configuration](../user-guide/configuration.md)):

```yaml
batches:
  my_cohort:
    name: "My cohort"
    video_dir: "/data/my_cohort/videos/"       # Day{N}_{mouse}_Trial{k}.MP4
    yml_dir: "/data/my_cohort/roi_yml/"        # or leave ROIs next to the videos
    meta_trials_csv: "/data/my_cohort/meta_trials.csv"   # day,mouse,reward
    metadata_csv: "/data/my_cohort/mice.csv"             # mouse, sex, genotype
    fps: 120
    pose_model_paths: ["/models/pose/centroid", "/models/pose/centered_instance"]
    roi_model_paths:  ["/models/roi/centroid",  "/models/roi/centered_instance"]
    column_map:
      mouse_id: ["mouse"]
      sex: "Gender"
```

## 2. Run everything

```bash
python run_all.py --config config.yaml --batch my_cohort
```

`run_all.py` runs the stages in order and skips any stage whose outputs already exist, so a
stopped run resumes where it left off. Use `--stages` to run a subset:

```bash
python run_all.py --stages 1-2        # inference only
python run_all.py --stages 3,4-6      # decisions, then stride detection and filtering
python run_all.py --stages 7-9        # mixed models and heatmaps
```

## 3. Or run stages one at a time

```bash
stride analyze-decisions --videos /data/my_cohort/videos --yml /data/my_cohort/roi_yml --meta meta_trials.csv
python -m stride.stages.run_gait_extraction --config config.yaml --batch my_cohort
python -m stride.stages.gait_lmm --config config.yaml
python -m stride.stages.classification --config config.yaml
```

## 4. Pass the gates before you trust a number

```bash
python -m stride.stages.safety_checks --roi-coverage /data/my_cohort/videos
python -m stride.stages.safety_checks --gait output/my_cohort/gait_per_stride_filtered.csv --expect-pxcm 26.1
```

The second command is the **calibration gate**: it fails if the strides are not in a uniform,
verified centimetre scale. See [Safety checks and gates](../user-guide/safety-checks.md).

## What you get

`decisions.csv`, `events.csv` and `metrics.csv` for the task; `gait_per_stride.csv`,
`gait_per_stride_filtered.csv`, `gait_per_step.csv` and `gait_per_video.csv` for gait;
`lmm_results_all.csv` with heatmaps; ROC curves for classification; and a
`<stage>_run_manifest.yaml` next to each. Column by column: [Outputs](../user-guide/outputs.md).

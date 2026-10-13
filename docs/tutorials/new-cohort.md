# Process a new cohort

End to end for one cohort recorded on a rig STRIDE already has models for.

## 1. Stage the videos

Sort clips into per-day folders and rename them `Day{N}_{mouse}_Trial{k}.MP4`. Reconcile the
number of clips per day against the recorder's trial counts before anything else; a mismatch
here becomes a wrong identity later. Write `meta_trials.csv` (`day,mouse,reward`) and the mouse
metadata table.

## 2. Add the batch to `config.yaml`

See [Configuration](../user-guide/configuration.md). Set `px_per_cm_override` if the rig has a
verified board value.

## 3. Inference (GPU environment)

```bash
python -m stride.stages.run_inference --config config.yaml --batch my_cohort
python -m stride.stages.safety_checks --roi-coverage /data/my_cohort/DAY1   # repeat per day
```

Large cohorts: run `stride pose-inference` with several workers per GPU
([Multi-GPU inference](multi-gpu.md)).

## 4. Decisions

```bash
python -m stride.stages.run_tmaze --config config.yaml --batch my_cohort
```

The summary must report `skipped: 0`. A skipped clip means it did not match a `day,mouse` row in
`meta_trials.csv` or has no region file.

## 5. Gait and the calibration gate

```bash
python -m stride.stages.run_gait_extraction --config config.yaml --batch my_cohort
python -m stride.stages.safety_checks --gait output/my_cohort/gait_per_stride_filtered.csv --expect-pxcm 26.08
```

Read the gate output: uniform scale equal to the board, plausible body length, uniform fps.

## 6. Statistics

```bash
python -m stride.stages.gait_lmm --config config.yaml
python -m stride.stages.classification --config config.yaml
```

## 7. Record what happened

Keep a per-cohort data-quality note next to the outputs: clips recovered by consensus ROI,
quarantined trials, identity corrections. The run manifests record the code and parameters; the
note records the judgement calls.

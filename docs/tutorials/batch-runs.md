# Stage-by-stage batch runs and statistics

`run_all.py` runs the numbered stages of the original study for the batches of one configuration
file. Use it when you want the genotype statistics, or several cohorts in one run. It expects
videos named `Day<N>_<mouse>_Trial<T>` and a reward table.

For a single recording set with no statistics, [`stride tmaze`](../pipelines/tmaze.md) is the
shorter way; it writes the same decision and gait tables.

## Try it on mock data

About a minute, from the repository root:

```bash
python tests/mock/make_mock_dataset.py --out tests/mock/_data --only core
python run_all.py --config tests/mock/_data/core/config.readme.yaml --output mock_output
stride status --output mock_output          # what was run, when, with which code
```

## On your own recordings

### 1. Write the config

```bash
cp config.example.yaml my_config.yaml         # edit the `batches:` section
```

```yaml
batches:
  my_batch:
    name: "My batch"
    video_dir: "/path/to/videos/"                # Day<N>_<mouse>_Trial<T>.mp4, searched recursively
    meta_trials_csv: "/path/to/meta_trials.csv"  # day,mouse,reward
    metadata_csv: "/path/to/metadata.csv"        # Mouse,Gender,Genotype
    fps: 120
    column_map:
      mouse_id: ["mouse"]
      mouse_id_sep: ""
      sex: "Gender"
```

Add `pose_model_paths` and `roi_model_paths` to run inference, or leave them out to use existing
pose and ROI files. Every key: [Configuration reference](../user-guide/configuration.md#batch-statistics-config).

### 2. Run

```bash
python run_all.py --config my_config.yaml --output output
python run_all.py --config my_config.yaml --output output --batch my_batch
python run_all.py --config my_config.yaml --output output --stages 3,4-6,7-9    # selected stages
```

| stage key | does |
|---|---|
| `1-2` | distortion check, pose and ROI inference, ROI region files |
| `3` | decisions, events, metrics, learning cycles |
| `4-6` | stride detection, keypoint confidence, stride filtering |
| `7-9` | trial-phase tagging, mixed models, heatmaps |
| `10` | WT vs Tau classifier |

Each stage reads what the previous one wrote under `<output>/<batch>/`; an explicit path in the
batch wins. A stage that fails stops the run with a non-zero exit code and says why.

### 3. Read the results

- Per batch: `<output>/<batch>/decisions/`, the gait tables, `learning_cycle_1.csv`.
- Across batches: `lmm_results_all.csv`, `lmm_fit_log.csv`, heatmaps, the classifier tables and
  figures ([Outputs](../user-guide/outputs.md#statistics-outputs-optional-steps)).
- `stride status --output output` lists the run manifests.

## Differences from `stride tmaze` to know

- A trial with no reward row is scored **not correct** here; `stride tmaze` leaves it empty.
- The decision rules are not applied by `run_all.py`. Run `stride apply-decision-rules` on a
  `decisions.csv` to get `decisions_clean.csv`.
- The scale is taken per video from the ROI corridor width unless the batch has
  `px_per_cm_override`.

## The statistics inside `stride tmaze`

```yaml
# in the T-maze config
steps:
  mixed_models: true
  classifier: true
stats_config: config.example.yaml
```

The metadata must carry a `genotype` column. Results are written under `stats/` of the output
folder.

# Quickstart

Both pipelines can be tried on synthetic ("mock") data that the repository generates: no real
videos, no models and no GPU are needed. Run the commands from the repository root after
`pip install -e ".[dev]"`.

## STRIDE T-maze on mock data

```bash
python tests/mock/make_mock_dataset.py --out tests/mock/_data --only core
stride tmaze tests/mock/_data/core/videos --roi-dir tests/mock/_data/core/roi_yml \
    --metadata tests/mock/_data/core/meta_trials.csv --metadata tests/mock/_data/core/metadata.csv \
    --out mock_output/tmaze --skip-inference
```

The mock set comes with its pose and ROI files, so the two inference steps are skipped
(`--skip-inference`). The run writes `mock_output/tmaze/SUMMARY.md`, the decision tables under
`decisions/` and the gait tables under `gait/`. The mock maze is drawn at 16 px/cm, so the gait
step prints its "below 20 px/cm" warning.

## STRIDE Gait on mock data

```bash
python tests/mock/make_mock_gait.py --out tests/mock/_data_gait --only runway
stride gait tests/mock/_data_gait/runway/videos --px-per-cm 24 \
    --metadata tests/mock/_data_gait/runway/metadata.csv --out mock_output/gait
```

A synthetic mouse walking a straight line, with its pose files. The tables are under
`mock_output/gait/gait/`.

## On your own recordings

Replace the placeholder paths with yours.

=== "STRIDE T-maze"

    ```bash
    stride tmaze /data/videos \
        --pose-model /models/pose/centroid --pose-model /models/pose/centered_instance \
        --roi-model  /models/roi/centroid  --roi-model  /models/roi/centered_instance \
        --out /data/stride_out
    ```

=== "STRIDE Gait"

    ```bash
    # pose files (<video name>.slp) next to the videos or in --pose-dir; scale measured by you
    stride gait /data/videos --px-per-cm 26.1 --out /data/gait_out

    # no pose files yet: your trained pose model makes them; scale from the ChArUco board in view
    stride gait /data/videos \
        --pose-model /models/pose/centroid --pose-model /models/pose/centered_instance \
        --out /data/gait_out
    ```

=== "Scale only"

    ```bash
    stride calibrate /data/videos --out /data/calibration
    ```

## Or from a config file

```bash
stride init-config tmaze -o my_tmaze.yaml     # commented template: videos, models, output, steps ...
stride tmaze --config my_tmaze.yaml

stride init-config gait -o my_gait.yaml
stride gait --config my_gait.yaml
```

Anything given on the command line overrides the file. Every key is described in the
[Configuration reference](../user-guide/configuration.md).

## What you get

`decisions.csv`, `decisions_clean.csv`, `events.csv` and `metrics.csv` for the task;
`gait_per_stride.csv`, `gait_per_stride_filtered.csv`, `gait_per_step.csv` and
`gait_per_video.csv` for gait; `SUMMARY.md`, a run log and run manifests. Column by column:
[Outputs and column glossary](../user-guide/outputs.md).

## Next

- [STRIDE T-maze](../pipelines/tmaze.md) and [STRIDE Gait](../pipelines/gait.md): every step and switch.
- [Recording requirements](recording.md): what a recording needs before the gait numbers mean anything.
- [Stage-by-stage batch runs and the optional statistics](../tutorials/batch-runs.md).

# Installation

STRIDE has two parts that live in two environments.

| Part | Stages | Environment |
|---|---|---|
| **Analysis package** (`stride`) | 3 to 10: decisions, gait, statistics, classification | plain Python, CPU only |
| **Inference runtime** | 1 to 2: calibration check, SLEAP pose and ROI inference | `sleap-nn` with PyTorch and a CUDA GPU |

## Analysis package

Requires Python 3.12 or newer.

```bash
git clone https://github.com/talmolab/STRIDE.git
cd STRIDE
pip install -e .        # or: uv pip install -e .
```

This installs the `stride` command and the stage modules. `sleap-io` is a dependency for reading
and writing `.slp` files only; it is not an inference engine.

## Inference runtime

Pose and ROI inference run on a separate GPU environment with
[sleap-nn](https://github.com/talmolab/sleap-nn). Install it following its own instructions,
then point `config.yaml` at that interpreter when running stages 1 to 2.

!!! note "Trained models are not bundled"
    STRIDE needs two SLEAP models: a 15-keypoint top-down **pose** model (centroid +
    centered-instance) and a 7-region **ROI** model. Set `pose_model_paths` and
    `roi_model_paths` in `config.yaml` to your local copies. The trained weights used in the
    paper will be deposited with the publication.

## Check the install

```bash
stride --help
python -m stride.stages.safety_checks --help
```

# Installation

## The package

Requires **Python 3.12 or newer**.

```bash
git clone https://github.com/talmolab/STRIDE.git
cd STRIDE
pip install -e .            # or: uv pip install -e .
pip install -e ".[dev]"     # adds pytest, needed for the mock-data quick starts
```

This installs the `stride` command and the stage modules. It is everything the analysis needs:
decisions, gait, scale calibration, and the optional statistics run on a plain CPU.

## The inference runtime (only when STRIDE runs your models)

STRIDE runs your SLEAP models through [sleap-nn](https://github.com/talmolab/sleap-nn) (PyTorch;
a GPU in practice). Install it separately, following its own instructions, in the same
environment as STRIDE. It is needed only for the two inference steps (pose, ROI); the analysis of
existing pose and ROI files does not use it. `sleap-io`, which is installed with STRIDE, only
reads and writes `.slp` files.

!!! note "Trained models are not bundled"
    STRIDE ships no trained models. Point it at your own SLEAP pose and ROI models
    (`--pose-model`, `--roi-model`, or the config). What the models must provide is listed in
    [Inputs and file layout](../user-guide/inputs.md). The trained weights of the original study
    are to be deposited (DOI to be added).

## OpenCV version

The package pins `opencv-python >= 4.8, < 4.14`. From 4.14 on (and in 5.x) the ChArUco detector
reads the same board about 0.1 % lower, so the pin keeps the scale measurement the same for
everyone ([Scale calibration](../pipelines/calibration.md)).

## Operating systems

| | Linux | macOS | Windows |
|---|---|---|---|
| Analysis on synthetic data (the test suite, run for every pull request) | tested | tested | tested |
| Inference with SLEAP models | run on NVIDIA GPUs | not run | not run |

The code uses no shell commands and no links, writes UTF-8 files, matches video extensions
case-insensitively and starts its workers under `spawn`.

## Check the install

```bash
stride --version
stride --help
pytest tests            # synthetic data only; no videos, models or GPU needed
```

## The authors' lab layer (optional)

`pip install -e lab/` adds the `stride-lab` command described in
[Authors' lab workflow](../about/lab-workflow.md). It is not needed to use STRIDE.

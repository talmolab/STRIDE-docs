# FAQ and troubleshooting

## Running

**Which command do I use?**
`stride tmaze` for T-maze recordings with a pose and an ROI model; `stride gait` for gait from
pose tracking alone ([Home](index.md#which-stride-do-i-need)).

**`unknown setting(s) [...]` or `unknown step(s) [...]`.**
A key in the config file, or a step name, is not one STRIDE knows. The message lists the valid
ones ([Configuration reference](user-guide/configuration.md)).

**`ERROR: config file not found: ... config.yaml` from `run_all.py`.**
The repository ships no `config.yaml`. Copy `config.example.yaml`, edit its `batches:` section
and pass it with `--config` ([Stage-by-stage batch runs](tutorials/batch-runs.md)).

**The run finished but `N video(s) were left out`.**
Each is listed with its reason in `SUMMARY.md`: no pose file, no ROI file, or an error in that
clip. Fix the cause and run the same command again; finished work is
not repeated. `--strict` makes a left-out video an error.

**A second run does nothing.**
Finished steps are skipped when their inputs and settings are unchanged. `--force` redoes the
analysis. To redo inference for a video, delete its pose or ROI file from the output folder.

**`N video(s) have no pose file: pose inference needs your trained pose model`.**
STRIDE ships no models. Pass `--pose-model` (and `--roi-model` for `stride tmaze`) or set them in
the config; or use `--skip-inference` to analyse only the videos that already have pose and ROI
files. `stride gait` stops in the same way when none of the videos has a pose file and no pose
model was given.

**`sleap-nn is not installed in this Python environment`.**
The two inference steps need [sleap-nn](https://github.com/talmolab/sleap-nn) installed in the
same environment as STRIDE; the analysis of existing pose and ROI files does not.

## Scale

**`stride gait` stops with `no scale`.**
There is no ROI to fall back on. Give `--px-per-cm`, or have a ChArUco board in view
([Scale calibration](pipelines/calibration.md)).

**`only N usable ChArUco reading(s)`.**
Is the board in view during the first 10 s of the clips, flat and unobstructed, and does the
board specification (squares, sizes, dictionary) match the print? `scale/calibration_summary.txt`
lists the clips in which no board was found.

**`the readings spread by X %`.**
The board or the camera moved between clips, or one reading is bad. Compare the
`_checkerboard.png` images.

**The `px_per_cm` column of the gait table is about 0.04.**
That column stores centimetres per pixel. The scale in px/cm is in `scale/px_per_cm.json`.

**`px_per_cm` is `1.0` in the gait table.**
The scale of that clip could not be derived (no ROI file, no value given). Its centimetre values
are invalid; the run warns about it. Fix the inputs and run again.

## Decisions

**`correct_TF` is empty.**
The rewarded arm of that trial is unknown: no metadata, or no row that matches the video. With
`day` + `mouse` rows the video must be named `Day<N>_<mouse>_Trial<T>`; for any other name use a
`video` column ([Inputs](user-guide/inputs.md#metadata-optional)).

**Accuracy looks too low.**
Rows with `ok == False` carry `correct_bin = 0` when the reward is known. Subset on `ok` first,
and use `decisions_clean.csv` for choices. In the batch entry points (`run_all.py`) a trial
without a reward row is also scored not correct.

**ROI inference finds no frame for most clips of a day.**
Something covers a maze corner in the sampled frames. STRIDE then runs the whole video, and
after that borrows the folder's consensus ROI. Check `roi/roi_sources.csv` and the montage under
`roi/roi_consensus_qc/` ([Pose and ROI models](how-it-works/models.md#consensus-roi)).

**The consensus ROI was refused.**
The detected ROIs of the folder differ by more than `roi.drift_max_px` (25 px): the maze or the
camera moved within the session. Split the folder at the move, or raise the limit after looking
at the overlays.

**`n_reversals` is 1 on a trial where the mouse ran straight.**
That is how the count is coded ([glossary](user-guide/outputs.md#eventscsv)).

## Gait

**Few strides survive the filters.**
The filters remove, in this order: every stride of a video whose mean keypoint confidence is not
above 0.3; the first and the last stride of every video (a video with one or two strides keeps
none); strides with a mean angular velocity outside -20 to +20 deg/s; and strides whose
`stride_length_cm` has the other sign than the run's median. The run log prints how many strides
each filter removed. The last filter assumes walking along the image x-axis; the run warns when
a recording does not meet that, and STRIDE does not rotate the data.

**Gait filtering stops with `confidence merge matched 0 stems`.**
The stride table and the per-video confidence table name the clips differently, so no stride
could be matched to a confidence value (for example tables from two different runs, or pose
files that carry another video name than the file on disk). Run the gait step again so that
both tables are written by the same run. When only some strides are unmatched the run warns,
counts them, and the confidence filter removes them.

**Stride length is shorter than the distance covered on the ground.**
`stride_length_cm` is the paw's excursion relative to the body
([Stride detection and gait measures](how-it-works/gait.md#the-measures-as-computed)).

**Stride length differs between two cameras for the same mice.**
Expected. Scale and tracking noise differ by rig. Compare within rig or z-score within cohort
([Statistics](how-it-works/statistics.md)).

**A keypoint is missing from my skeleton.**
`stride gait` stops before any analysis and prints what is missing and which nodes the pose file
has. Map other names with `keypoints:`; the tail tip is optional
([STRIDE Gait](pipelines/gait.md#keypoints)).

**Pose inference is slow or stalls on a network share.**
Decode is CPU-bound and network filesystems add latency. Keep videos on local disk during
inference, or at least run the workers on the machine that holds the data.

## Scope

**Can STRIDE score an elevated plus maze, Y-maze or open field?**
Gait: yes, with `stride gait` ([Gait on a different maze](tutorials/other-maze.md)).
Decisions: only the T geometry is implemented.

**Does STRIDE correct lens distortion?**
No. The distortion check reports it; undistort the videos first if it says they need it.

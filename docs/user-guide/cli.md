# Command reference

The usage text below is the output of `--help` of STRIDE 1.0.0.

| command | purpose |
|---|---|
| [`stride tmaze`](#stride-tmaze) | STRIDE T-maze: videos and your pose and ROI models in, per-trial and per-stride tables out |
| [`stride gait`](#stride-gait) | STRIDE Gait: videos with pose tracking in, gait tables out |
| [`stride calibrate`](#stride-calibrate) | px/cm from a ChArUco board in view |
| [`stride init-config`](#stride-init-config) | write a commented config template |
| [`stride status`](#stride-status) | list the run manifests under an output folder |
| [`stride pose-inference`](#stride-pose-inference), [`stride roi-inference`](#stride-roi-inference) | one inference step, with worker options for several GPUs |
| [`stride convert-roi`](#stride-convert-roi), [`stride propagate-roi`](#stride-propagate-roi) | ROI predictions to region files; consensus ROI for a folder |
| [`stride analyze-decisions`](#stride-analyze-decisions), [`stride apply-decision-rules`](#stride-apply-decision-rules) | the decision stage and the two decision rules on their own |
| [`stride analyze-gait`](#stride-analyze-gait) | the quick two-step stride filter |
| [`stride check-distortion`](#stride-check-distortion), [`stride count-frames`](#stride-count-frames) | lens distortion report; frame counts |
| [`stride run`](#stride-run) | older one-folder runner: distortion check, inference, decisions |
| [`python run_all.py`](#run_allpy) | stage-by-stage batch runner (stages 1 to 10) |
| [`python -m stride.stages.safety_checks`](#safety-checks) | calibration, ROI coverage and order checks |

```text
Usage: stride [OPTIONS] COMMAND [ARGS]...

  T-maze Pipeline - Automated video analysis for behavioral experiments.

Options:
  --version  Show the version and exit.
  --help     Show this message and exit.

Commands:
  analyze-decisions     Analyze T-maze arm entry decisions.
  analyze-gait          Filter gait strides by keypoint confidence and...
  apply-decision-rules  Apply the two protocol rules to a decisions.csv...
  calibrate             px/cm from a ChArUco board in view of the camera.
  check-distortion      Check if videos need undistortion using charuco...
  convert-roi           Convert ROI .slp files to polygon YAML format.
  count-frames          Count total frames across all videos in a directory.
  gait                  STRIDE Gait: videos with pose tracking in,...
  init-config           Write a commented config template for `stride...
  pose-inference        Run pose estimation inference on videos (mouse...
  propagate-roi         Give the folder's consensus ROI to the videos...
  roi-inference         Run ROI keypoint inference on videos.
  run                   Run the pipeline on one folder: distortion check,...
  status                Show what has been run under an output directory.
  tmaze                 STRIDE T-maze: videos + your pose and ROI models...
```

## Pipelines

### `stride tmaze`

```text
Usage: stride tmaze [OPTIONS] [VIDEOS]...

  STRIDE T-maze: videos + your pose and ROI models in, per-trial and per-
  stride tables out.

  VIDEOS is a folder (searched recursively), a video file, or a pattern;
  several may be given. Steps: discover, distortion check (report only), scale
  (px/cm), pose inference, ROI inference (+ consensus ROI), decisions,
  decision rules, gait. Finished work is not repeated on a second run.

  stride tmaze /data/videos --pose-model P1 --pose-model P2 --roi-model R1 --roi-model R2 -o out
  stride tmaze --config my_tmaze.yaml
  stride tmaze /data/videos -o out --skip-inference        # pose and ROI files already exist

Options:
  -c, --config PATH          YAML config (`stride init-config tmaze` writes a
                             template). Options given on the command line
                             override the config.
  -o, --out PATH             Output folder
  --pose-model PATH          Your trained pose model: give the option once or
                             twice (centroid, centered instance)
  --roi-model PATH           Your trained maze-ROI model: give the option once
                             or twice
  -m, --metadata PATH        Optional CSV (repeatable). Trial rows: `video`
                             (file name) or `day` + `mouse`, plus `reward`
                             (L/R); optional `trial`, `sex`, `genotype`. Mouse
                             rows: `mouse` + `sex` / `genotype`
  --pose-dir PATH            Folder with existing pose files, when they are
                             not next to the videos
  --roi-dir PATH             Folder with existing ROI files, when they are not
                             next to the videos
  --px-per-cm FLOAT          Scale you measured yourself (wins over the
                             ChArUco board and the ROI width)
  --maze-width-cm FLOAT      Corridor width for the ROI-based scale (default
                             10)
  --board-squares-x INTEGER  ChArUco board: squares along x
  --board-squares-y INTEGER  ChArUco board: squares along y
  --board-square-mm FLOAT    ChArUco board: square size in mm
  --board-marker-mm FLOAT    ChArUco board: marker size in mm
  --board-dict TEXT          ChArUco board: dictionary, e.g. DICT_5X5_250
  --device TEXT              Inference device: auto (default), cuda, cuda:0,
                             cpu
  -w, --workers INTEGER      CPU worker processes for the analysis (default 4)
  --batch-size INTEGER       Pose inference batch size (default 16)
  --pose-suffix TEXT         Pose file = <video name><suffix> (default .slp)
  --skip-inference           Do not run the two inferences: analyse the videos
                             that already have pose and ROI files
  --skip STEP                Switch a step off (repeatable): distortion_check,
                             calibrate, pose_inference, roi_inference,
                             roi_consensus, decisions, decision_rules, gait
  --enable STEP              Switch a step on (repeatable), e.g. mixed_models,
                             classifier (off by default)
  --stop-after STEP          Stop after this step
  --force                    Redo steps even when their outputs are up to date
  --strict                   Exit with an error when any video was left out
  --help                     Show this message and exit.
```

### `stride gait`

```text
Usage: stride gait [OPTIONS] [VIDEOS]...

  STRIDE Gait: videos with pose tracking in, per-stride and per-video gait
  tables out.

  No maze ROI is needed. VIDEOS is a folder (searched recursively), a video
  file, or a pattern; several may be given. Pose files (<video name>.slp) are
  taken from next to the videos or from --pose-dir; with --pose-model they are
  made first. The scale comes from --px-per-cm or from a ChArUco board in
  view. Steps: discover, scale, pose inference, stride detection, keypoint
  confidence, stride filtering. Finished work is not repeated on a second run.

  stride gait /data/videos --px-per-cm 26.1 -o out
  stride gait /data/videos --pose-model P1 --pose-model P2 -o out      # scale from the ChArUco board
  stride gait --config my_gait.yaml

Options:
  -c, --config PATH               YAML config (`stride init-config gait`
                                  writes a template). Options given on the
                                  command line override the config.
  -o, --out PATH                  Output folder
  --pose-dir PATH                 Folder with existing pose files, when they
                                  are not next to the videos
  --pose-suffix TEXT              Pose file = <video name><suffix> (default
                                  .slp)
  --pose-model PATH               Your trained pose model (once or twice):
                                  pose inference runs for videos without a
                                  pose file
  --px-per-cm FLOAT               Scale you measured yourself (wins over the
                                  ChArUco board)
  --board-squares-x INTEGER       ChArUco board: squares along x
  --board-squares-y INTEGER       ChArUco board: squares along y
  --board-square-mm FLOAT         ChArUco board: square size in mm
  --board-marker-mm FLOAT         ChArUco board: marker size in mm
  --board-dict TEXT               ChArUco board: dictionary, e.g. DICT_5X5_250
  --keypoint THEIR_NAME=STRIDE_NAME
                                  Node name in your pose files = STRIDE
                                  keypoint name (repeatable), e.g. nose=snout
  -m, --metadata PATH             Optional CSV with a `video` column (file
                                  name); its other columns are added to the
                                  tables
  --fps FLOAT                     Frame rate used only when it cannot be read
                                  from a video (default 120)
  --device TEXT                   Inference device: auto (default), cuda,
                                  cuda:0, cpu
  -w, --workers INTEGER           CPU worker processes (default 4)
  --batch-size INTEGER            Pose inference batch size (default 16)
  --skip STEP                     Switch a step off (repeatable): calibrate,
                                  pose_inference, gait_extraction,
                                  keypoint_confidence, stride_filtering
  --enable STEP                   Switch a step on (repeatable)
  --stop-after STEP               Stop after this step
  --force                         Redo steps even when their outputs are up to
                                  date
  --strict                        Exit with an error when any video was left
                                  out
  --help                          Show this message and exit.
```

### `stride calibrate`

```text
Usage: stride calibrate [OPTIONS] VIDEOS...

  px/cm from a ChArUco board in view of the camera.

  VIDEOS is a folder (clips are found recursively; the first clip of each sub-
  folder is measured) or one or more clip files. Writes px_per_cm.txt, a
  summary, a per-clip table and QC images, and prints the value. Use the value
  as the batch's `px_per_cm_override` in the gait stage.

Options:
  --out DIRECTORY            Output folder. Default: <folder>/calibration.
  --board-squares TEXT       ChArUco squares, X x Y.  [default: 14x9]
  --square-mm FLOAT          Square side on the print, mm.  [default: 20.0]
  --marker-mm FLOAT          Marker side on the print, mm.  [default: 15.0]
  --dictionary TEXT          ArUco dictionary of the board.  [default:
                             DICT_5X5_250]
  --clips INTEGER            For a folder: measure at least this many clips.
                             [default: 6]
  --max-clips INTEGER        Measure at most this many clips (spread over the
                             folders).
  --frames-per-clip INTEGER  Frames examined per clip over the first 10 s.
                             Default: one every 0.15 s (67).
  --min-readings INTEGER     Usable clip readings needed to accept a value.
                             [default: 3]
  --max-spread FLOAT         Largest accepted spread of the readings, %.
                             [default: 1.0]
  --rail-crop TEXT           Optional cross-check: y0,y1,x0,x1 of a straight
                             10-cm corridor in the frame.
  --workers INTEGER          Clips measured in parallel.  [default: 1]
  --help                     Show this message and exit.
```

### `stride init-config`

```text
Usage: stride init-config [OPTIONS] [tmaze|gait]

  Write a commented config template for `stride tmaze --config` or `stride
  gait --config`.

Options:
  -o, --output PATH  File to write (default: ./stride_<pipeline>.yaml)
  --overwrite        Replace the file when it exists
  --help             Show this message and exit.
```

### `stride status`

```text
Usage: stride status [OPTIONS]

  Show what has been run under an output directory.

  Every stage writes a <stage>_run_manifest.yaml next to its outputs; this
  lists them (stage, when, code version, result counts).

Options:
  -o, --output PATH      Output directory of a run (default: current
                         directory)
  -c, --checkpoint PATH  (legacy) a progress.log checkpoint file to read
  --help                 Show this message and exit.
```

## Single steps

### `stride pose-inference`

```text
Usage: stride pose-inference [OPTIONS]

  Run pose estimation inference on videos (mouse body keypoints).

  For multi-GPU parallelism, run multiple instances with different worker IDs:

      # Terminal 1 - GPU 0     stride pose-inference -i /videos -m <centroid>
      -m <centered_instance> --gpu 0 --worker-id 0 --num-workers 2

      # Terminal 2 - GPU 1     stride pose-inference -i /videos -m <centroid>
      -m <centered_instance> --gpu 1 --worker-id 1 --num-workers 2

Options:
  -i, --input PATH          Directory containing videos (searched recursively,
                            .mp4 / .MP4)  [required]
  -o, --output PATH         Output directory for .slp files (default: same as
                            video)
  -m, --model PATH          Model path; give twice (centroid and
                            centered_instance). Required
  -b, --batch-size INTEGER  Batch size for inference
  -d, --device TEXT         Device for inference (cuda, cuda:0, cuda:1, cpu)
  -g, --gpu INTEGER         GPU device ID (shorthand for --device cuda:N)
  --worker-id INTEGER       Worker ID for multi-GPU parallel processing
                            (0-indexed)
  --num-workers INTEGER     Total number of parallel workers (for multi-GPU)
  --overwrite               Overwrite existing outputs
  --help                    Show this message and exit.
```

### `stride roi-inference`

```text
Usage: stride roi-inference [OPTIONS]

  Run ROI keypoint inference on videos.

  For multi-GPU parallelism, run multiple instances with different worker IDs:

      # Terminal 1 - GPU 0     stride roi-inference -i /videos -o /output -m
      <centroid> -m <centered_instance> --gpu 0 --worker-id 0 --num-workers 2

      # Terminal 2 - GPU 1     stride roi-inference -i /videos -o /output -m
      <centroid> -m <centered_instance> --gpu 1 --worker-id 1 --num-workers 2

Options:
  -i, --input PATH       Directory containing videos (searched recursively,
                         .mp4 / .MP4)  [required]
  -o, --output PATH      Output directory for ROI .slp files  [required]
  -m, --model PATH       Model path; give twice (centroid and
                         centered_instance). Required
  -d, --device TEXT      Device for inference (cuda, cuda:0, cuda:1, cpu).
                         Default: chosen by sleap-nn
  -g, --gpu INTEGER      GPU device ID (shorthand for --device cuda:N)
  --worker-id INTEGER    Worker ID for multi-GPU parallel processing
                         (0-indexed)
  --num-workers INTEGER  Total number of parallel workers (for multi-GPU)
  --overwrite            Overwrite existing outputs
  --help                 Show this message and exit.
```

### `stride convert-roi`

```text
Usage: stride convert-roi [OPTIONS]

  Convert ROI .slp files to polygon YAML format.

Options:
  -i, --input PATH   Directory containing ROI .slp files  [required]
  -o, --output PATH  Output directory for .yml files  [required]
  --overwrite        Overwrite existing outputs
  --help             Show this message and exit.
```

### `stride propagate-roi`

```text
Usage: stride propagate-roi [OPTIONS] VIDEO_DIR

  Give the folder's consensus ROI to the videos without a <video
  name>.rois.yml.

  The consensus is the per-vertex median of the ROI files detected in the same
  folder (one recording session: maze and camera did not move). Existing ROI
  files are never overwritten; QC overlays and montage sheets are written for
  a person to check.

Options:
  --drift-warn FLOAT  Static-maze guard: max per-vertex spread across the
                      detected ROIs, in px  [default: 25.0]
  --refuse-on-drift   Write nothing when the guard is exceeded (default: write
                      with a warning)
  --qc-frame FLOAT    Fraction of the clip at which the QC frame is taken
                      [default: 0.5]
  --qc-out PATH       Folder for the QC overlays (default:
                      <VIDEO_DIR>/roi_consensus_qc)
  --help              Show this message and exit.
```

### `stride analyze-decisions`

```text
Usage: stride analyze-decisions [OPTIONS]

  Analyze T-maze arm entry decisions.

  Writes decisions.csv, events.csv and metrics.csv (plus
  decisions_run_manifest.yaml) into the output directory.

Options:
  -v, --videos PATH      Directory containing videos with pose files next to
                         them (searched recursively)  [required]
  -y, --yml PATH         Directory containing ROI .yml files (omit when the
                         ROI files sit next to the videos)
  -m, --meta PATH        Metadata CSV with day,mouse,reward columns
                         [required]
  -o, --output PATH      Output directory for decisions.csv, events.csv,
                         metrics.csv (default: ./decisions)
  --pose-suffix TEXT     Suffix of the pose file next to each video (default:
                         .slp)
  -w, --workers INTEGER  Number of parallel workers
  --help                 Show this message and exit.
```

### `stride apply-decision-rules`

```text
Usage: stride apply-decision-rules [OPTIONS] DECISIONS_CSV

  Apply the two protocol rules to a decisions.csv and write
  decisions_clean.csv.

  A trial stays a decision only if the arm entry cleared the strict 95 % body-
  in-arm threshold and happened within the decision window. The input file is
  not modified.

Options:
  --window-s FLOAT   Decision window: an arm entry after this many seconds is
                     no decision  [default: 150.0]
  -o, --output PATH  File to write (default: decisions_clean.csv next to the
                     input)
  --help             Show this message and exit.
```

### `stride analyze-gait`

```text
Usage: stride analyze-gait [OPTIONS]

  Filter gait strides by keypoint confidence and remove edge strides.

  This is the quick two-step filter. The pipeline's stage 6 (`python -m
  stride.stages.run_gait_extraction`) additionally applies the angular-
  velocity and forward-direction filters from the config and therefore keeps
  fewer strides.

Options:
  -i, --input PATH       Input gait per-stride CSV  [required]
  -c, --confidence PATH  Confidence CSV (optional)
  -o, --output PATH      Output filtered CSV
  -t, --threshold FLOAT  Confidence threshold (default: 0.3)
  --help                 Show this message and exit.
```

### `stride check-distortion`

```text
Usage: stride check-distortion [OPTIONS]

  Check if videos need undistortion using charuco board analysis.

Options:
  -i, --input PATH          Directory containing videos (searched recursively)
                            or single video file  [required]
  -o, --output PATH         Output file for results (CSV/JSON)
  -n, --num-frames INTEGER  Number of frames to sample per video
  --squares-x INTEGER       Charuco board squares in X
  --squares-y INTEGER       Charuco board squares in Y
  --help                    Show this message and exit.
```

### `stride count-frames`

```text
Usage: stride count-frames [OPTIONS]

  Count total frames across all videos in a directory.

Options:
  -i, --input PATH  Directory containing videos (searched recursively, .mp4 /
                    .MP4 / .avi)  [required]
  --help            Show this message and exit.
```

### `stride run`

```text
Usage: stride run [OPTIONS]

  Run the pipeline on one folder: distortion check, pose + ROI inference,
  decisions.

  Inference needs the trained SLEAP models (--pose-model / --roi-model, each
  given twice) and the sleap-nn runtime. With --skip-pose --skip-roi it
  analyses existing pose and ROI files. For the gait stages and the statistics
  use `python run_all.py --config <config>`.

Options:
  -i, --input PATH       Directory containing input videos (searched
                         recursively)  [required]
  -o, --output PATH      Output directory for results
  -m, --meta PATH        Metadata CSV with day,mouse,reward columns
  --pose-model PATH      Pose model path; give twice (centroid, centered-
                         instance). Needed unless --skip-pose
  --roi-model PATH       ROI model path; give twice (centroid, centered-
                         instance). Needed unless --skip-roi
  -y, --yml PATH         Existing ROI .yml folder to use with --skip-roi
                         (default: <output>/roi_yml, then ROI files next to
                         the videos)
  -w, --workers INTEGER  Number of parallel workers
  --skip-distortion      Skip distortion check
  --skip-pose            Skip pose inference (use existing .slp)
  --skip-roi             Skip ROI inference (use existing ROI files)
  --batch-size INTEGER   Batch size for inference
  --device TEXT          Device for inference (cuda, cuda:0, cpu)
  --help                 Show this message and exit.
```

## Stage-by-stage runner

Runs the numbered stages for the batches of a [batch config](configuration.md#batch-statistics-config). See [Stage-by-stage batch runs](../tutorials/batch-runs.md).

### `run_all.py`

```text
usage: run_all.py [-h] [--config CONFIG] [--output OUTPUT] [--batch BATCH] [--stages STAGES]
                  [--device DEVICE]

T-Maze Pipeline Orchestrator

options:
  -h, --help       show this help message and exit
  --config CONFIG  Pipeline config (default: config.yaml next to this script; copy
                   config.example.yaml to make one)
  --output OUTPUT  Output directory (default: ./output, resolved from CWD)
  --batch BATCH    Process only this batch key
  --stages STAGES  Comma-separated stage keys to run, from: 1-2, 3, 4-6, 7-9, 10 (e.g.
                   '3,4-6,7-9'). Default: all
  --device DEVICE  Inference device for stages 1-2 (cuda, cuda:0, cpu). Default: from the config,
                   else cuda
```

| stage key | module | does |
|---|---|---|
| `1-2` | `stride.stages.run_inference` | distortion check, pose inference, ROI inference, ROI region files |
| `3` | `stride.stages.run_tmaze` | decisions, events, metrics, learning cycles |
| `4-6` | `stride.stages.run_gait_extraction` | stride detection, keypoint confidence, stride filtering |
| `7-9` | `stride.stages.gait_lmm` | trial-phase tagging, mixed models, heatmaps |
| `10` | `stride.stages.classification` | WT vs Tau classifier |

Each stage module can also be started on its own:

### `python -m stride.stages.run_inference`

```text
usage: run_inference.py [-h] [--config CONFIG] [--output OUTPUT] [--batch BATCH] [--skip-pose]
                        [--skip-roi] [--skip-distortion] [--device DEVICE]

Config-driven SLEAP inference (Stages 1-2)

options:
  -h, --help         show this help message and exit
  --config CONFIG    Pipeline config (default: ./config.yaml)
  --output OUTPUT    Output directory (default: ./output)
  --batch BATCH      Process only this batch key
  --skip-pose        Skip pose inference
  --skip-roi         Skip ROI inference
  --skip-distortion  Skip distortion check
  --device DEVICE    Inference device (cuda, cuda:0, cpu). Default: `device` in the batch /
                     `inference.device` in the config, else cuda for pose
```

### `python -m stride.stages.run_tmaze`

```text
usage: run_tmaze.py [-h] [--config CONFIG] [--output OUTPUT] [--batch BATCH]

Config-driven T-maze behavioral analysis

options:
  -h, --help       show this help message and exit
  --config CONFIG  Pipeline config (default: ./config.yaml)
  --output OUTPUT  Output directory (default: ./output)
  --batch BATCH    Process only this batch key
```

### `python -m stride.stages.run_gait_extraction`

```text
usage: run_gait_extraction.py [-h] [--config CONFIG] [--output OUTPUT] [--batch BATCH]

Config-driven gait extraction (Stages 4-6)

options:
  -h, --help       show this help message and exit
  --config CONFIG  Pipeline config (default: ./config.yaml)
  --output OUTPUT  Output directory (default: ./output)
  --batch BATCH    Process only this batch key
```

### `python -m stride.stages.gait_lmm`

```text
usage: gait_lmm.py [-h] [--config CONFIG] [--output OUTPUT] [--batch BATCH]
                   [--trial-phase {task,all,return,poked_return}]

Config-driven gait LMM analysis

options:
  -h, --help            show this help message and exit
  --config CONFIG       Path to config.yaml (default: ./config.yaml)
  --output OUTPUT       Output directory (default: ./output)
  --batch BATCH         Process only this batch key
  --trial-phase {task,all,return,poked_return}
                        Which trial_phase to include (default: task)
```

### `python -m stride.stages.classification`

```text
usage: classification.py [-h] [--config CONFIG] [--output OUTPUT]

WT vs Tau logistic regression

options:
  -h, --help       show this help message and exit
  --config CONFIG  Pipeline config (default: ./config.yaml)
  --output OUTPUT  Output directory (default: ./output)
```

### `python -m stride.stages.run_calibration`

```text
usage: run_calibration.py [-h] [--config CONFIG] [--output OUTPUT] [--batch BATCH]

Config-driven ChArUco scale calibration (stage 0)

options:
  -h, --help       show this help message and exit
  --config CONFIG  Pipeline config
  --output OUTPUT  Output directory
  --batch BATCH    Process only this batch key
```

## Safety checks

```text
usage: safety_checks.py [-h] [--gait GAIT] [--roi-coverage ROI_COVERAGE]
                        [--expect-pxcm EXPECT_PXCM] [--expect-count EXPECT_COUNT]
                        [--yml-dir YML_DIR] [--vid VID] [--stamp STAMP] [--require REQUIRE]
                        [--strict]

STRIDE safety gates

options:
  -h, --help            show this help message and exit
  --gait GAIT
  --roi-coverage ROI_COVERAGE
  --expect-pxcm EXPECT_PXCM
  --expect-count EXPECT_COUNT
  --yml-dir YML_DIR     ROI yml folder for --roi-coverage when the ROI files are not next to the
                        videos
  --vid VID             video dir for the gate ledger (--stamp/--require)
  --stamp STAMP         stamp this gate name as passed (needs --vid)
  --require REQUIRE     comma-list of gate names that must already be stamped (needs --vid)
  --strict
```

Exit code 2 on any FAIL, 1 on a WARN when `--strict` is given, 0 otherwise. Details: [Safety checks](safety-checks.md).

## `stride-lab`

The authors' intake command is a separate package (`pip install -e lab/`); its commands are listed in [Authors' lab workflow](../about/lab-workflow.md).

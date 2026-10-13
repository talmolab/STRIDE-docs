# Pipeline overview

```mermaid
flowchart TB
  subgraph inference [Stages 1-2 · GPU environment]
    A[Calibration and distortion check] --> B[Pose inference: 15 keypoints, every frame]
    A --> C[ROI inference: best frame, 23 keypoints]
    C --> D[Convert to 7 polygons .rois.yml]
  end
  subgraph tmaze [Stage 3 · T-maze]
    B --> E[Coverage per region per frame]
    D --> E
    E --> F[Gate, arm entry, correctness, probes, reversals]
  end
  subgraph gait [Stages 4-6 · Gait]
    B --> G[Preprocess: interpolate, smooth, velocities]
    D --> G
    G --> H[Swing detection and stride segmentation]
    H --> I[Metrics per stride and per step]
    I --> J[Filter: confidence, edges, direction, turning]
  end
  subgraph stats [Stages 7-10]
    F --> K[Tag strides with trial phase]
    J --> K
    K --> L[Mixed models M1 and M2, FDR]
    L --> M[WT vs Tau classifier]
  end
```

| stage | module | reads | writes |
|---|---|---|---|
| 1 | `distortion_check` | sampled frames | distortion report, scale |
| 2 | `pose_inference`, `roi_inference`, `slp_to_yaml` | video, models | `.slp`, `.rois.yml` |
| 3 | `decision_analysis` | `.slp`, `.rois.yml`, `meta_trials.csv` | `decisions.csv`, `events.csv`, `metrics.csv` |
| 4 to 5 | `gait_extraction` | `.slp`, `.rois.yml` | `gait_per_stride.csv`, `gait_per_step.csv`, confidence table |
| 6 | `gait_filtering` | per-stride table | `gait_per_stride_filtered.csv` |
| 7 to 9 | `gait_lmm` | filtered strides, decisions | `lmm_results_all.csv`, heatmaps |
| 10 | `classification` | per-mouse features | ROC curves, feature weights |

Stages 1 to 2 need the GPU environment; everything after reads files and runs on any machine.

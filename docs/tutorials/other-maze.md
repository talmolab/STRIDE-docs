# Gait on a different maze

The gait module does not know what a T-maze is. It needs a pose file, one straight region to
measure in, the scale and the frame rate.

## What to provide

| input | requirement |
|---|---|
| pose `.slp` | the 15-node skeleton, or at minimum `snout`, `tailbase`, `tailtip` and the four hind-paw nodes with those names |
| region file | a `.rois.yml` whose `segment2` polygon spans a straight corridor of known width; the other six regions can be any polygons that tile your arena |
| scale | `px_per_cm_override` from a calibration target, or a corridor width in `maze_width_cm` that matches `segment2` |
| frame rate | at least 100 fps for full stride resolution ([Recording requirements](../getting-started/recording.md)) |
| metadata | `mouse`, `sex`, `genotype` columns mapped through `column_map` |

## Steps

1. Train or fine-tune a SLEAP pose model for your rig with the node names above.
2. Draw the regions once per session (the maze does not move) and save them as `.rois.yml`
   files named after each clip, or run the ROI model if your maze is a T.
3. Add a batch with `finalized: false`, a `video_dir`, the region directory and
   `px_per_cm_override`.
4. Run stages 4 to 6 and the calibration gate:

```bash
python -m stride.stages.run_gait_extraction --config config.yaml --batch my_arena
python -m stride.stages.safety_checks --gait output/my_arena/gait_per_stride_filtered.csv --expect-pxcm <value>
```

5. Run the mixed models. Without `decisions.csv` there is no trial phase, so pass
   `--trial-phase all`.

## What you lose without the T

Trial phase tagging, the regional (stem, junction, arm) analysis and every cognitive metric. The
stride table, the filters, the sway metrics and the M1/M2 models are unchanged.

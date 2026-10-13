# FAQ and troubleshooting

**A clip is missing from `decisions.csv` or `gait_per_stride.csv`.**
It has no region file, or its `day,mouse` did not match `meta_trials.csv`. Run
`safety_checks --roi-coverage` on the folder and check the stem.

**`px_per_cm` is `1.0` in the gait table.**
The stride was never calibrated: no region file was found to derive the scale and no override
was set. The calibration gate fails on this. Fix the inputs and re-run gait; do not divide by hand.

**The calibration gate says the scale is uniform but disagrees with the board.**
`px_per_cm_override` is set to a wrong constant. Re-measure the board and update the batch.

**ROI inference reports `no_passing_frame` for most of a day.**
Either the board or the operator occludes the maze in the sampled frames, or you are running an
old selector that evaluates only the first instance per frame. Use the current `roi-inference`,
then recover remaining clips with the session consensus
([Pose and ROI models](how-it-works/models.md)).

**Accuracy by day looks too low.**
You are counting `ok == False` rows as errors. Subset on `ok` first
([Outputs](user-guide/outputs.md)).

**Stride length differs between two cameras for the same mice.**
Expected. Scale and tracking noise differ by rig. Compare within rig or z-score within cohort
([Statistics](how-it-works/statistics.md)).

**Pose inference is slow or stalls on a network share.**
Decode is CPU-bound and network filesystems add latency. Keep videos on local disk during
inference, or at least run the workers on the machine that holds the data.

**Can STRIDE score an elevated plus maze, Y-maze or open field?**
Gait: yes, with a straight region and a scale ([Gait on a different maze](tutorials/other-maze.md)).
Decisions: only the T geometry is implemented.

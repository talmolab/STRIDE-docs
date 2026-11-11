# T-maze decisions

The decision step scores each trial from the pose file and the seven regions. The thresholds are
constants of `decision_analysis.py`; the columns they produce are listed in the
[column glossary](../user-guide/outputs.md#decision-tables).

## Coverage

For every frame the convex hull of six body keypoints (`snout`, `forepawL2`, `forepawR2`,
`hindpawL2`, `hindpawR2`, `tailbase`; gaps of up to 7 frames are interpolated first) is
intersected with each region. The **coverage** of a region is the share of the hull's area that
lies inside it, in percent. A frame whose hull is smaller than 1500 px² has no coverage.

## Gate

`gate_frame` is the later of two frames: the 0.5 s mark of the video, and the first frame of a
run of at least 5 frames in which the coverage of `segment4` is below 30 % (`gate_reason =
seg4_leave`; `warmup_only` when no such run exists). `segment4` is the stem segment next to the
junction, so on a trial that starts at the far end of the stem the gate is in practice the 0.5 s
mark. Arm dwells before the gate are ignored.

## Arm entry

1. **Arm dwells.** A dwell is a run of at least 3 frames (1-frame gaps bridged) in which the
   coverage of `arm_left` or `arm_right` is at least **95 %**. If no such dwell starts at or
   after the gate, the same search is repeated at **60 %**. `entry_threshold` records which
   threshold produced the dwells.
2. **Depth and hind end.** For the first frame of each dwell: the depth is the distance of the
   snout from the junction's boundary (negative while the snout is still inside the junction),
   and the hind end counts as in the arm when `hindpawL2`, `hindpawR2` or `tailbase` lies inside
   the arm polygon.
3. **Choice.** Among the dwells with a depth above 50 px and the hind end in the arm, the deepest
   is taken. If there is none, the earliest dwell with the hind end in the arm is taken; if there
   is none either, the first dwell found.
4. `entry_side`, `entry_frame` and `entry_ms` describe the first frame of the chosen dwell.

A trial with no arm dwell after the gate gets `ok = False` and `reason = no_arm_dwell_after_gate`.

## Correctness

`correct_TF` compares `entry_side` with the rewarded arm from the metadata. When the reward is
unknown, `stride tmaze` leaves `correct_TF` and `correct_bin` empty. In the batch entry points
(`run_all.py`, `stride analyze-decisions`, `stride run`) a trial with no reward row is scored not
correct (`F` / `0`), and a warning names it.

## Decision rules

The `decision_rules` step (also `stride apply-decision-rules`) writes `decisions_clean.csv` next
to `decisions.csv`. A trial stays a decision only if both hold:

- **Full commit**: the entry cleared the strict 95 % threshold, not only the 60 % fallback;
- **Within the window**: `entry_ms` is at most the decision window (`decision_rules.window_s`,
  150 s, the trial length of the published protocol).

A reclassified trial gets `ok = False`, `reason = soft_entry_no_commit` or `entry_after_window`,
and empty entry columns. `decisions.csv` is never modified, and `events.csv` and `metrics.csv`
are not changed by the rules. Use `decisions_clean.csv` for choices.

## Junction exploration and snout probes

- **Junction exploration** (`junction_explore_ms_precommit`): the frames before the commit that
  lie in junction dwells, where a junction dwell is a run of at least 3 frames with junction
  coverage of at least 95 %.
- **Snout probe**: a frame in which the snout lies inside an arm polygon while the coverage of
  that arm is below 40 % and the junction coverage is at least 30 %. An episode is a run of at
  least 2 consecutive such frames. Probes are counted per arm, before the commit, with their
  frame counts. The snout position is gap-filled (up to 7 frames) like the body keypoints.

## Latencies

`latency_choice_ms` is `commit_ms - enter_seg4_ms` and `stem_latency_ms` is
`enter_junction_ms - enter_seg1_ms`, where a region is entered at the first frame in which its
coverage is at least 50 %.

## Reversals

`n_reversals` and `reversal_frames` come from a position score per frame along the stem, as
described in the [glossary](../user-guide/outputs.md#eventscsv). With the scores as coded, a
straight run from `segment1` to the junction registers one reversal.

## Learning cycles (batch runner)

Stage 3 of `run_all.py` also splits the trials per mouse at the first day on which the rewarded
arm changes: trials before it form `learning_cycle_1.csv`, trials from it on
`learning_cycle_2.csv`.

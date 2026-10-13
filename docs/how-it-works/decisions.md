# T-maze decisions

Stage 3 scores each trial from the pose and the seven regions. All thresholds are defaults in
`decision_analysis.py`.

## Coverage

For every frame, the fraction of the mouse's keypoints inside each region gives a **body
coverage** per region. Decisions are made on these coverage traces plus the snout position.

## Gate

The trial clock starts when the mouse **leaves segment 4**: coverage of segment 4 drops below
30% for 5 consecutive frames (`gate_reason = seg4_leave`).

## Arm entry

The first arm dwell after the gate counts as the decision when it is a **deep entry**:

1. body coverage in the arm at or above **95%** (strict), with a **60%** soft fallback;
2. snout at least **50 px** beyond the junction boundary;
3. at least one hind paw inside the arm;
4. the dwell lasts at least 3 frames, allowing 1-frame gaps.

`entry_threshold` records which rule accepted the entry (`>=95_depth` or the soft rule).
`correct_TF` compares `entry_side` with the rewarded arm from `meta_trials.csv`. A trial with no
qualifying dwell gets `ok = False` and a `reason` such as `no_arm_dwell_after_gate`.

## Junction exploration and snout probes

Between the junction entry and the commit, the time the body spends in the junction is the
**pre-commit exploration time**. A **snout probe** is a snout excursion into an arm polygon while
the body stays in the junction: arm coverage below 40% and junction coverage above 30%. Probes
are counted per arm with their frame counts.

## Reversals

Inside the task window, a move toward one arm followed by a return and a commit to the other is
a **reversal**; counts and frames are written to `events.csv`.

## Learning cycles

Reward reversal across days is detected per mouse from `meta_trials.csv`. Trials before the
first reversal form the first learning cycle, the window used for days-to-criterion.

## Protocol rules

Lab protocols often add rules on top of the raw decisions, for example reclassifying soft entries
that never commit or entries after a time limit as undecided (`U`). Keep those in a separate
step that writes `decisions_clean.csv`, so the raw table stays reproducible.

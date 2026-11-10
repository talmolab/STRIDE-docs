# Statistics (optional)

The genotype comparison of the paper is not part of the automated run. It is two existing stages
that can be switched on:

- in `stride tmaze`: `steps: {mixed_models: true, classifier: true}` plus `stats_config`
  (results under `stats/`). The metadata needs a `genotype` column;
- with the stage-by-stage runner: stages `7-9` and `10` of `run_all.py`
  ([Stage-by-stage batch runs](../tutorials/batch-runs.md)).

Both read the batch / statistics config
([Configuration reference](../user-guide/configuration.md#statistics-settings)).

## Trial phase tagging

Each stride is matched to its trial by the video name, using the decision table. A stride whose
`frame_start` lies between `gate_frame` and `entry_frame` (inclusive) is a **task** stride. Every
other stride of a matched video is a **return** stride. A return stride faster than twice the
mouse's median task-stride speed is a **poked return**. Strides of videos without a decision row
are `unmatched`; without a decision table every stride is `untagged`. The mixed models use task
strides by default (`--trial-phase task`).

## Mixed models

Two linear mixed models per metric and batch (statsmodels `mixedlm`, REML), with a random
intercept per mouse. The metric and the covariates are z-scored.

| model | fixed effects |
|---|---|
| **M1** | genotype + body length [+ sex] |
| **M2** | genotype + stride speed + body length [+ sex] |

- Genotype is compared with the reference genotype (`statistics.reference_genotype`, `WT`). The
  labels in `statistics.genotype_order` are used; a batch may contain any subset that includes
  the reference, and other labels are left out.
- Sex is added when the batch has `has_sex: true`.
- The metrics are the `metrics:` list of the config (14 columns; `step_freq_hz` is derived as
  `1000 / duration_ms`). In M2 a metric that is itself a covariate is skipped.
- A fit is not produced when fewer than 100 strides have all values present, or when a standard
  error of the fit is above 10. `lmm_fit_log.csv` lists every fit, kept or not, and why.
- `lmm_results_all.csv` holds the coefficients (in the metric's units and standardized), standard
  errors and p-values, plus `<genotype>_fdr`: Benjamini-Hochberg over all genotype p-values of
  one batch and model. The heatmaps show `-log10(p)` next to the standardized coefficients and
  mark FDR below 0.05.

## Classifier

A WT-versus-Tau logistic regression on per-mouse features. It needs mice labelled `WT` and `Tau`.

1. **Features.** Gait (6): Euclidean stride length, step frequency, snout sway, temporal symmetry,
   stride duration and step width, from the filtered strides with a speed of 15 to 25 cm/s, median
   per mouse. T-maze (8): accuracy (the share of correct trials, a mean), junction exploration,
   number of probes, probe duration, probe bias, probe-choice correlation, and the log10 choice
   and stem latencies (per-mouse medians).
2. **Ranking.** Features are ranked by p-value: the values listed under
   `classification.gait_lmm_pvalues` / `tmaze_lmm_pvalues` in the config where given, otherwise a
   WT-versus-Tau t-test on the per-mouse values. The ranking uses all mice: it is done outside
   the cross-validation folds.
3. **Model.** Standardized inputs, logistic regression with balanced class weights, 5-fold
   stratified cross-validation (seed 42), evaluated for the top K features, K = 1 to N. The K
   with the highest mean AUC is reported as optimal.
4. **Outputs.** `regression_feature_ranking.csv`, `regression_auc_vs_k.csv` with its figure, and
   ROC curves for the T-maze, gait and combined feature sets.

The shipped `config.example.yaml` has empty p-value tables, so with it every feature is ranked by
the t-test.

!!! note "Never pool raw centimetres across cameras"
    Rigs differ by a few percent in scale and by more in tracking noise. Compare genotypes within
    a rig, or z-score within cohort before pooling. Camera differences never explain
    biology-sized effects, but they do add offsets.

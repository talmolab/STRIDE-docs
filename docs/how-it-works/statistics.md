# Statistics and classification

## Trial phase tagging (stage 7)

Each stride is placed in its trial using `decisions.csv`: strides between the gate frame and the
arm-entry frame are **task** strides; strides after the entry are the **return**; a return that
is faster than twice the mouse's within-trial median speed is a **poked return**. The mixed models
use task strides by default (`--trial-phase task`).

## Mixed models (stages 8 to 9)

Two linear mixed models per metric and batch, mouse as random intercept:

| model | fixed effects |
|---|---|
| **M1** | genotype (treatment-coded against WT) + standardized body length [+ sex] |
| **M2** | M1 + standardized stride speed |

M2 asks whether a genotype effect survives after speed is accounted for, since speed drives most
gait metrics. Fourteen metrics enter (the `metrics:` list in `config.yaml`); p-values are
corrected with Benjamini-Hochberg FDR across the metrics of each model. Results go to
`lmm_results_all.csv`, and heatmaps show `-log10(p)` next to standardized coefficients with
FDR-significant cells marked.

### Regional analysis

Strides are tagged by the region of their midpoint (stem, junction, arm) and the models are
re-run per region, which is how the junction-specific step-frequency effect in tau mice was found.

## Classification (stage 10)

Per-mouse averaged features from both modules are ranked by p-value (gait from the M2 models;
T-maze metrics from mixed models where available, t-tests otherwise). A logistic regression with
standardized inputs and balanced class weights separates WT from tau mice under 5-fold stratified
cross-validation, evaluated at every feature-set size K = 1 to N. Outputs: ROC curves for the
T-maze, gait and combined sets and the ranked feature table.

!!! note "Never pool raw centimetres across cameras"
    Rigs differ by a few percent in scale and by more in tracking noise. Compare genotypes within
    a rig, or z-score within cohort before pooling. Camera differences never explain
    biology-sized effects, but they do add offsets.

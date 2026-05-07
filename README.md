# Quasi-Experimental Causal Analysis: Depth Effects on Dungeness Crab Shell Condition
**WDFW Puget Sound Crab Survey, 2015–2024**

## Overview

This notebook is a principled follow-up to a predictive-modeling capstone (Random Forest, Logistic Regression, GAM on the same WDFW dataset). The original work achieved 88% predictive accuracy but flagged three structural limitations: (1) correlated predictors (depth, temperature, DO) made Partial Dependence Plots uninterpretable; (2) the 88% hard-shell majority class dominated accuracy metrics; (3) region-level confounding was visible but never formally adjusted for.

This analysis reframes the question causally: *Does water depth (shallow ≤ 30 ft vs. deep > 80 ft) affect the probability of hard-shell crab condition, after adjusting for region, season, year, sex, and soak time?*

## Key Findings

| Metric | Value |
|---|---|
| **Adjusted APE** (depth treatment on hard-shell prob.) | **+6.60 pp** |
| **Bootstrap 95% CI** (11 station-clusters, 490 reps) | **[−12.75, +17.59] pp** |
| **Cluster-robust Odds Ratio** (shallow vs. deep) | **2.08** [95% CI: 1.29, 3.35] |
| **Imbalanced covariates** before restriction (\|SMD\| > 0.1) | **11 of 16** |
| **E-value** (point estimate / CI bound) | **3.57 / 1.89** |
| **MDE** (80% power, pooled) | **< 0.5 pp** |

**Interpretation**: The analytic OR (2.08, CI excludes 1) is statistically significant with cluster-robust standard errors. However, the cluster bootstrap CI — which correctly reflects only 11 independent sampling stations — is wide and includes zero. This illustrates a critical design limitation: 163,758 crab observations reduce to just 11 independent station-level sampling units. The data is not as informative as its row count suggests. An E-value of 3.6 means an unmeasured confounder would need RR ≥ 3.6 associations with both depth assignment and shell condition to overturn the OR point estimate.

**Estimable population**: Port Townsend Bay and Hood Canal only. Marine Area 10 and Marine Area 13 have shallow n < 50 and are excluded from all causal inference.

## Analysis Structure

| Section | Content |
|---|---|
| **§1** | Data loading, feature engineering (season, depth_cat, hard_shell, treated) |
| **§2** | Coverage diagnostics — region × season heatmap, depth coverage by region |
| **§3** | Standardized Mean Differences — imbalance before restriction |
| **§4** | Common-support restriction to PTB + Hood Canal |
| **§5** | Retrospective power and Minimum Detectable Effect |
| **§6** | Primary logistic regression + cluster bootstrap for APE |
| **§7.1** | Specification curve (8 specs; bad-control demonstration) |
| **§7.2** | Heterogeneous treatment effects by region |
| **§7.3** | E-value for unmeasured confounding |
| **§8** | Decision framing: what we can/cannot claim, WDFW recommendations |

## Methodological Highlights

- **No temp/DO in primary model**: They are post-treatment mediators (depth causes temp and DO). Adding them is over-adjustment bias; §7.1 demonstrates this empirically via a specification curve.
- **Station-level cluster bootstrap**: With 11 unique stations, within-station observations are not independent. Resampling stations (not rows) produces correct finite-sample uncertainty.
- **E-value**: Quantifies robustness to unmeasured confounders without assuming their existence.
- **Narrow estimable population**: Causal claims are restricted to the two regions with adequate shallow coverage — this is a feature, not a bug.

## Figures

| File | Description |
|---|---|
| `fig_coverage_region_season.png` | Region × Season heatmap — structural gaps in data |
| `fig_smd.png` | SMD bar chart — covariate imbalance before restriction |
| `fig_bootstrap_ape.png` | Bootstrap distribution of APE (station-clustered) |
| `fig_spec_curve.png` | Specification curve — 8 models including bad-control specs |

## How to Re-Run

```bash
# From the project directory
source .venv/bin/activate
jupyter nbconvert --to notebook --execute --inplace \
    --ExecutePreprocessor.timeout=3600 \
    causal_analysis_dungeness.ipynb
```

Expected runtime: ~20 minutes (bootstrap is the bottleneck).

**Requirements**: numpy, pandas, matplotlib, seaborn, statsmodels, scipy, nbformat, jupyter/nbconvert. All available in the `.venv` environment.

**Data**: `final_merged_with_region.csv` must be in the working directory (341,202 rows, 17 columns).

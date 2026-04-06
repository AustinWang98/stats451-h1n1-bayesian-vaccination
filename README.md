# H1N1 vaccination: hierarchical Bayesian logistic regression

STATS 451 (University of Michigan) project. The code models **binary H1N1 vaccine uptake** (`h1n1_vaccine`) from National H1N1 Flu Survey–style individual records: behaviors, doctor recommendations, health indicators, demographics, and geography.

## Data handling (in code)

- Empty strings are treated as missing. **Numeric** columns get **median** imputation; listed **categorical** fields get **mode** imputation.
- Ordinal categories (`income_level`, `age_bracket`, `qualification`, `census_msa`) are mapped to **ordered numeric** scores; other categorical columns are **dummy-coded as numeric factors** for glmnet/Stan matrices.
- **70/30** train/test split with stratification on the outcome (`caret::createDataPartition`, fixed seed).

## Feature selection: two logistic lassos

Lasso is used separately for each hierarchical specification so the **grouping variable is not in the penalized predictor matrix** (it only enters as the hierarchical intercept index).

1. **Qualification model:** predictors exclude `qualification` (and `unique_id`, response). `glmnet::cv.glmnet`, `family = "binomial"`, `alpha = 1`, **5-fold CV**, `type.measure = "auc"`, coefficients at **`lambda.min`**. Nonzero columns define `x` for the first Stan model.
2. **Census MSA model:** same setup but the matrix excludes **`census_msa`** instead; a second feature set for the second Stan model.

Dropped variables differ between the two models (see the formal report table); the code prints selected and removed names per run.

## Stan models (RStan)

Both models are **Bernoulli logistic regression with random intercepts** and shared slopes:

\[
y_i \sim \mathrm{Bernoulli}(\mathrm{logit}^{-1}(\alpha_{g[i]} + x_i^\top \beta)),
\]

where \(g[i]\) is either **education level** (`qualification`, \(J\) levels) or **MSA category** (`census_msa`, \(J\) levels).

**Group intercepts:** \(\alpha_j \sim \mathcal{N}(\mu_\alpha, \sigma_\alpha)\) for \(j = 1,\ldots,J\) (partial pooling).

**Priors (same in both programs):**

| Parameter | Prior |
|-----------|--------|
| \(\beta\) | \(\mathcal{N}(0, 1)\) |
| \(\mu_\alpha\) | \(\mathcal{N}(0, 1)\) |
| \(\sigma_\alpha\) | **Half-Cauchy** implied by `sigma_alpha ~ cauchy(0, 1.5)` with `lower=0` |

**Sampling:** `sampling(..., iter = 2000, chains = 4, seed = 123)` for each model.

**Generated quantities:** per-row **probability** `theta` and **pointwise log-likelihood** `log_lik` for **PSIS-LOO** (`loo` / `loo_compare`).

## What the analysis does after fitting

- **`loo_compare`** on the two fits (qualification vs census MSA random intercepts).
- **`bayesplot::mcmc_areas`** on posterior samples for \(\beta\), \(\alpha\), \(\mu_\alpha\), \(\sigma_\alpha\) (census MSA model), with a mapping from `beta[k]` to predictor names.
- **Test-set prediction** using the **census MSA** fit: for each test row, linear predictors are built from posterior draws of \(\alpha\) and \(\beta\), then **posterior mean** inv-logit probabilities; **grouped means** by `census_msa` vs observed vaccination rates and a bar/point plot.

## Files (what they hold)

| File | Role |
|------|------|
| `analysis.Rmd` | Full pipeline: imputation, encoding, EDA plots, both lassos, both Stan strings, LOO, posterior plots, test predictions, grouped calibration plot. |
| `final-project-report.Rmd` | Long-form writeup (LaTeX sections) with embedded figures under `figures/`. |
| `model-specification.Rmd` | Compact math description of likelihood, priors, and posterior (variant priors on \(\beta\) / hyperparameters in that draft). |
| `analysis-early-draft.Rmd` | Earlier end-to-end draft of the same style of workflow. |
| `data/h1n1_vaccine_prediction.csv` | Row-level survey features and `h1n1_vaccine`. |
| `figures/` | Static figures for the report; `posterior-distribution.png` from the posterior visualization workflow. |
| `deliverables/` | Exported PDFs, slides, and documents from the course. |
| `archive/` | Alternate report source with different local figure paths. |

Core dependencies in the notebooks: **tidyverse**, **glmnet**, **rstan**, **loo**, **bayesplot**, **gridExtra**, **caret**, **knitr**.

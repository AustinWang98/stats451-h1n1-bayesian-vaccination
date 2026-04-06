# H1N1 vaccination prediction (hierarchical Bayes)

Course project for **STATS 451** (University of Michigan): binary prediction of H1N1 vaccine uptake using **lasso feature selection** and **hierarchical Bayesian logistic regression** fit with **RStan**. The analysis uses survey-style individual-level data with demographic, behavioral, and health-related predictors.

## Repository layout

| Path | Description |
|------|-------------|
| `data/h1n1_vaccine_prediction.csv` | Modeling dataset (one row per respondent; response `h1n1_vaccine`). |
| `analysis.Rmd` | Main pipeline: imputation, encoding, EDA plots, glmnet lasso, Stan models, evaluation. |
| `final-project-report.Rmd` | Written report (PDF via R Markdown / LaTeX sections). |
| `model-specification.Rmd` | Short math write-up of likelihood, priors, and posterior structure. |
| `analysis-early-draft.Rmd` | Earlier analysis draft (same general workflow as `analysis.Rmd`). |
| `figures/` | Report figures (`01.png`, `02.png`, `03.jpg`) plus `posterior-distribution.png`. |
| `deliverables/` | Exported PDFs, slides, and related documents from the course milestone. |
| `archive/` | Alternate report draft with machine-specific figure paths (not maintained for reproducibility). |

## Requirements

- **R** (recent 4.x recommended)
- R packages used in the notebooks: `tidyverse`, `glmnet`, `rstan`, `bayesplot`, `gridExtra`, `caret`, `knitr`

**RStan** needs a working C++ toolchain (see the [RStan getting started guide](https://mc-stan.org/users/interfaces/rstan)).

## How to run

1. Clone the repository and open the project folder as your working directory (in RStudio: *Session → Set Working Directory → To Source File Location* is **not** required if you knit from the project root with `.Rmd` files at the top level).
2. Install missing packages when R prompts you, or install them manually before knitting.
3. Knit `analysis.Rmd` to reproduce modeling, plots, and Stan fits (runtime depends on MCMC settings and hardware).

### Formal report figures

`final-project-report.Rmd` knits figures from `figures/01.png`, `figures/02.png`, and `figures/03.jpg` (included in this repository). To refresh them, regenerate plots from the EDA sections in `analysis.Rmd` and overwrite those files.

## Methods (summary)

- Missing values: numeric median imputation; categorical mode imputation.
- Train/test split: 70% / 30% (`caret::createDataPartition`).
- Feature screening: logistic **lasso** with 5-fold CV, `lambda.min`, AUC as the criterion.
- Hierarchical models: group-specific intercepts by **education (`qualification`)** or **MSA status (`census_msa`)**, with shared individual-level coefficients; compared in the report.

## Suggested GitHub repository name

Use a short, searchable name such as:

**`stats451-h1n1-bayesian-vaccination`**

(Alternatives: `umich-stats451-h1n1-hierarchical-bayes`, `h1n1-vaccine-hierarchical-bayes`.)

## License

Add a `LICENSE` file if you want others to reuse the code; course projects often use MIT or remain “all rights reserved” unless your instructor specifies otherwise.

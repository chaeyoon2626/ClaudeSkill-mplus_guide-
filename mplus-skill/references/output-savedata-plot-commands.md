# OUTPUT / SAVEDATA / PLOT Commands — Cross-Procedure Option Reference

> Source: Mplus User's Guide v8, Chapter 18 (OUTPUT, SAVEDATA, and PLOT commands)
> https://www.statmodel.com/HTML_UG/chapter18V8.htm

## One-line summary
Regardless of which procedure is used (regression / path analysis / EFA / mixture model, etc.), "I want to see one particular piece of information in the results" almost always comes down to adding a single OUTPUT/SAVEDATA/PLOT option from this file. Always consult it **alongside** the procedure-specific reference file.

## Usage logic
When the user says something like "I only want to see [X] in the results," "How do I test [Y]?", "I want to save [Z]," or "I'd like to see this as a graph," first find the matching option in the table below, then combine it with the `MODEL:`/`ANALYSIS:` syntax pulled from the relevant procedure-specific reference file.

## OUTPUT options

| Option | What it does | When to use it |
|---|---|---|
| `SAMPSTAT` | Sample statistics: means/variances/covariances/correlations, etc. | Check descriptive statistics |
| `CROSSTABS` | Cross-tabulations for categorical variables | Check associations between categorical variables |
| `STANDARDIZED` | All 3 types of standardized coefficients + SE | Compare effect sizes across variables on different scales |
| `STDYX` | Standardizes both predictor and outcome variables | The most commonly used standardized coefficient (for comparing regression coefficients) |
| `STDY` | Standardizes only the outcome variable (when a binary predictor is present) | When there's a dummy predictor variable |
| `STD` | Standardizes only latent variables | When you only care about latent-variable relationships, independent of observed-variable scale |
| `RESIDUAL` | Residuals: observed vs. model-implied values | Detailed fit diagnostics |
| `MODINDICES` | Modification indices & expected parameter change | How much fit would improve if a fixed parameter were freed |
| `CINTERVAL` | Confidence interval (frequentist) / credibility interval (Bayesian). `CINTERVAL (BOOTSTRAP)` gives an asymmetric bootstrap CI | Whenever a confidence interval is needed, e.g. for mediation effects |
| `SVALUES` | Prints the estimated parameters as starting-value syntax for the next analysis | Speeds up convergence for repeated analyses/large models |
| `PATTERNS` | Summary of missing-data patterns | Understanding the structure of missingness |
| `TECH1` | Parameter specification and starting values | Confirm the model is set up as intended |
| `TECH3` | Covariance/correlation matrix of the parameter estimates | Check relationships/precision among parameters |
| `TECH4` | Means, covariances, and correlations of latent variables (with SE, p-values) | Check the distribution/relationships of latent variables |
| `TECH8` | Iterative estimation progress (optimization history) | Check convergence status/speed, especially for slow-running models |
| `TECH10` | Univariate/bivariate/response-pattern fit for categorical outcomes (observed vs. estimated frequencies) | Detailed fit check for categorical indicators |
| **`TECH11`** | **Lo-Mendell-Rubin likelihood ratio test (comparing k vs. k-1 classes)** | **Essential for deciding "how many classes is right" in a mixture model (LPA/LCA)** |
| `TECH12` | Residuals for observed vs. model-estimated means/variances/covariances/skewness/kurtosis | Multi-angle fit diagnostics for mixture models |
| `TECH13` | Two-sided test comparing observed vs. model-generated skewness/kurtosis via 200 resampling replications | Detailed distributional fit test (e.g. normality) for mixture models — **not** for deciding the number of classes |
| **`TECH14`** | **Parametric bootstrap likelihood ratio test (k vs. k-1 classes)** | **A stricter alternative to TECH11 for deciding the number of classes (more computationally expensive)** |
| `TECH15` | Marginal/conditional probabilities across classes, latent transition probabilities | Interpreting a latent transition model (LTA) |
| `ENTROPY` | Per-indicator entropy contribution | Which indicator contributes most to distinguishing classes |

> **Note**: if a user asks specifically about "TECH13," check first whether they actually mean it — it's a distributional fit test (skewness/kurtosis), not a class-count decision statistic. For "how many classes is right" type questions, `TECH11` (LMR-LRT) or `TECH14` (bootstrap LRT) is the correct answer. Users often aren't sure of the exact TECH number, so always ask what result they actually want to see, then find the correct number in this table — never just go along with whatever number they happened to say.

## SAVEDATA options (most commonly used)

| Option | What it does |
|---|---|
| `SAVE = CPROBABILITIES;` | Saves each individual's posterior class-membership probabilities + most-likely-class assignment — essentially required for mixture models (TYPE=MIXTURE) |
| `SAVE = FSCORES;` | Saves factor scores (frequentist) / plausible values (Bayesian) |
| `FILE IS ...;` | Specifies the output file name |
| `SWMATRIX = ...;` | Saves within/between sample statistics, e.g. for two-level EFA (saves computation time on reanalysis) |
| `DIFFTEST = ...;` | Saves the H1-model derivatives needed for a WLSMV/MLMV chi-square difference test |

## PLOT options

| TYPE | Plots included |
|---|---|
| `PLOT1` | Histograms, scatterplots, time-series plots (ACF/PACF) |
| `PLOT2` | Estimated means/percentiles, sample/estimated probabilities, LOOP plots (e.g. moderation effects), bootstrap distributions, survival curves |
| `PLOT3` | PLOT1 + PLOT2 + histograms/scatterplots of factor scores/residuals |
| `SENSITIVITY` | Sensitivity-analysis plots for mediator-outcome confounding (used with MODEL INDIRECT) |

## Likely FAQ mapping
- "I ran LPA and want to see how many classes is right" → `TECH11` (+ `TECH14` if needed), decide together with BIC/per-class sample size
- "I want a confidence interval for the mediation effect" → `CINTERVAL (BOOTSTRAP)`
- "I want to see standardized coefficients" → `STDYX`
- "I want to save which class each person belongs to" → `SAVEDATA: SAVE = CPROBABILITIES;`
- "How do I check whether the model converged?" → `TECH8` (+ check for the "MODEL ESTIMATION TERMINATED NORMALLY" message in the results)

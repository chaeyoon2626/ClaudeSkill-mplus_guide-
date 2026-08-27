# Missing Data — Mechanisms and Multiple Imputation

> Source: Mplus User's Guide v8, Chapter 11, Examples 11.1–11.6
> Bundled source: references/source-pdfs/Chapter11.pdf

## One-line summary
Handle missing data with maximum-likelihood estimation under MCAR/MAR (the Mplus default), model non-ignorable missingness (NMAR) with a selection or pattern-mixture model, or generate multiply imputed data sets with the `DATA IMPUTATION` command for reuse across one or more later analyses.

## Prerequisite checklist
- [ ] What is the missing-value code/symbol in the raw data file (e.g. 999, -99)?
- [ ] Is missingness assumed MCAR/MAR (ignorable — can depend on observed covariates and observed outcomes), or is there reason to suspect NMAR (missingness related to the unobserved value itself, e.g. dropout tied to how a person would have scored)?
- [ ] If NMAR is suspected: is a selection model (dropout modeled via logistic regression on the outcome, needs numerical integration) or a pattern-mixture model (dropout pattern used as a covariate of the growth factors) preferred?
- [ ] Is there an extra variable outside the substantive model that predicts missingness and could be added as a "missing data correlate" to improve the plausibility of MAR?
- [ ] Is the end goal a single FIML analysis run, or a set of imputed data sets meant to be reused across several downstream analyses (multiple imputation)?
- [ ] If doing multiple imputation: how many imputed data sets are needed (default is five), and should any imputed variables be flagged as categorical?

## Option selection logic
| Situation | Choice |
|---|---|
| Continuous/censored/categorical/count outcomes with ignorable missingness (MCAR/MAR) | Just declare `MISSING = ALL (code);` — maximum-likelihood (FIML) estimation is used by default, no extra options needed |
| Want to improve the plausibility of the MAR assumption using an extra correlate that is not part of the substantive model | `AUXILIARY = (m) z;` — z is allowed to correlate with the outcome and corrects the number of parameters/chi-square test |
| Need descriptive statistics of the outcome by dropout status | `DATA MISSING:` with `TYPE = DDROPOUT;` (dummy dropout indicators) + `ANALYSIS: TYPE = BASIC;` + `DESCRIPTIVE =` |
| Want to plot dropout means vs. sample means over time | `PLOT: TYPE = PLOT2; SERIES = var-list(*);` |
| Suspect dropout depends on the current (missing) outcome value (NMAR) and want to jointly model the outcome and dropout process | Diggle-Kenward selection model: `DATA MISSING: TYPE = SDROPOUT;` + `ANALYSIS: ALGORITHM = INTEGRATION; INTEGRATION = MONTECARLO;` + dropout-indicator `ON` outcome statements in `MODEL:` |
| Suspect NMAR but prefer to let the dropout pattern act as a covariate of the growth factors, without numerical integration | Pattern-mixture model: `DATA MISSING: TYPE = DDROPOUT;` + `i ON d1-d5; s ON d3-d5;` (plus equality constraints for underidentified coefficients) |
| Need reusable imputed data sets to feed into one or more subsequent analyses | `DATA IMPUTATION: IMPUTE = ...; NDATASETS = ...; SAVE = name*.dat;`, then in later runs `DATA: TYPE = IMPUTATION;` |
| Want imputation and the substantive frequentist analysis done in one job | `DATA IMPUTATION:` block followed directly by `ANALYSIS: ESTIMATOR = ML;` and the substantive `MODEL:` |

## Menu path & screen fields
**[FIML with a missing data correlate — Ex 11.1]**
1. **DATA:** `FILE = ...;`
2. **VARIABLE:** `USEVARIABLES = y1-y4;` / `MISSING = ALL (999);` / `AUXILIARY = (m) z;`
3. **ANALYSIS:** `ESTIMATOR = ML;`
4. **MODEL:** substantive model, e.g. growth model `i s | y1@0 y2@1 y3@2 y4@3;`
5. **OUTPUT:** `TECH1;` (optional — check parameter specification)

**[Dropout descriptives/plots — Ex 11.2]**
1. **VARIABLE:** `USEVARIABLES = ... y0-y5 d1-d5;` / `MISSING = ALL (999);`
2. **DATA MISSING:** `NAMES = y0-y5; TYPE = DDROPOUT; BINARY = d1-d5; DESCRIPTIVE = y0-y5 | * z1-z5;`
3. **ANALYSIS:** `TYPE = BASIC;`
4. **PLOT:** `TYPE = PLOT2; SERIES = y0-y5(*);`

**[NMAR selection model — Ex 11.3]**
1. **VARIABLE:** `CATEGORICAL = d1-d5;`
2. **DATA MISSING:** `NAMES = y0-y5; TYPE = SDROPOUT; BINARY = d1-d5;`
3. **ANALYSIS:** `ESTIMATOR = ML; ALGORITHM = INTEGRATION; INTEGRATION = MONTECARLO; PROCESSORS = 2;`
4. **MODEL:** growth model + `d# ON` (current-time outcome, coefficient fixed label) and (previous-time outcome, different fixed label), equal across time

**[NMAR pattern-mixture model — Ex 11.4]**
1. **DATA MISSING:** `NAMES = y0-y5; TYPE = DDROPOUT; BINARY = d1-d5;`
2. **MODEL:** growth model + `i ON d1-d5; s ON d3-d5; s ON d1 (1); s ON d2 (1);`

**[Multiple imputation, standalone — Ex 11.5]**
1. **VARIABLE:** `USEVARIABLES = ...; AUXILIARY = v1-v10; MISSING = ALL (999);`
2. **DATA IMPUTATION:** `IMPUTE = y1-y4 x1 (c) x2; NDATASETS = 10; SAVE = missimp*.dat;`
3. **ANALYSIS:** `TYPE = BASIC;`
4. **OUTPUT:** `TECH8;`

**[Multiple imputation + downstream ML analysis in one job — Ex 11.6]**
1. **VARIABLE:** `USEVARIABLES = y1-y4 x1 x2; MISSING = ALL (999);`
2. **DATA IMPUTATION:** `IMPUTE = y1-y4 x1 (c) x2; NDATASETS = 10;`
3. **ANALYSIS:** `ESTIMATOR = ML;`
4. **MODEL:** substantive growth model, e.g. `i s | y1@0 y2@1 y3@2 y4@3; i s ON x1 x2;`
5. **OUTPUT:** `TECH1 TECH8;`

**[Reusing a previously saved imputed-data set in a later run]**
1. **DATA:** `FILE = missimplist.dat; TYPE = IMPUTATION;`
2. Continue as any normal analysis (VARIABLE/ANALYSIS/MODEL)

## Sub-option details
- `MISSING = ALL (999);` : declares which value/symbol in the raw data is treated as missing/invalid for every variable listed. Maximum-likelihood estimation (FIML) is applied by default and handles MCAR and MAR without any further options; standard errors under missing data are computed from the observed information matrix (Kenward & Molenberghs, 1998), and bootstrap standard errors/confidence intervals are also available with missing data.
- `AUXILIARY = (m) z;` : the `(m)` setting marks z as a *missing data correlate* — a variable outside the substantive model that is allowed to correlate with the outcome so as to improve the plausibility of the MAR assumption and to preserve the correct number of parameters and chi-square test for the analysis model (Asparouhov & Muthén, 2008b). Contrast with plain `AUXILIARY = v1-v10;` (no `(m)`), which simply carries variables through to be saved (e.g. alongside imputed data sets) without treating them as missing data correlates.
- `DATA MISSING:` : creates a set of binary variables that flag missing data/dropout for another set of variables.
  - `NAMES =` : the variables (must already be on the `VARIABLE: NAMES` list) for which missingness indicators are created.
  - `TYPE = DDROPOUT;` : binary dummy dropout indicators — one fewer indicator than there are time points.
  - `TYPE = SDROPOUT;` : binary discrete-time (event-history) survival dropout indicators, for use in a Diggle-Kenward-style selection model.
  - `BINARY =` : assigns names to the newly created binary indicator variables (e.g. d1-d5).
  - `DESCRIPTIVE =` : used together with `ANALYSIS: TYPE = BASIC;` and `TYPE = DDROPOUT;` to request the mean/SD for a set of variables, computed using all observations without missing data on the variable, and separately for those who drop out (or not) before the next time point.
- `PLOT: TYPE = PLOT2; SERIES = y0-y5(*);` : requests missing-data plots of dropout means vs. sample means, viewed after the run via the post-processing graphics module. `SERIES` lists the variables to connect with a line; `(*)` assigns sequential x-axis values (1, 2, 3, …) to those variables.
- Diggle-Kenward selection model (Ex 11.3): jointly estimates a growth model for the outcome and a discrete-time survival model for the dropout indicators (Diggle & Kenward, 1994). `ALGORITHM = INTEGRATION;` is required because latent variables representing the missing outcome influence the binary dropout indicators; `INTEGRATION = MONTECARLO;` is required because the dimensions of integration vary across observations. The `d# ON` outcome statements specify logistic regressions of a dropout indicator on the outcome at the previous time point and at the current time point (which is latent/missing for those already dropped out); coefficients are held equal across time via matching fixed labels. This example uses numerical integration and can be computationally demanding depending on problem size.
- Pattern-mixture model (Ex 11.4; Little, 1995; Hedeker & Gibbons, 1997; Demirtas & Schafer, 2003): the dropout pattern (dummy dropout indicators) is used as a covariate of the growth factors (`i ON d1-d5; s ON d3-d5;`). A coefficient can be unidentified when the outcome is observed at only one time point for that dropout pattern (e.g. `s ON d1`, since the pattern with d1=1 only has data at the first time point); it is then held equal to an identified coefficient (`s ON d2`) purely for identification.
- `DATA IMPUTATION:` : creates a set of imputed data sets via multiple imputation when the data set has missing values.
  - `IMPUTE = y1-y4 x1 (c) x2;` : lists the variables for which missing values are imputed; `(c)` after a variable marks it as categorical for imputation purposes.
  - `NDATASETS = 10;` : number of imputed data sets to create; the default is five.
  - `SAVE = missimp*.dat;` : saves the imputed data sets, one file per imputation with the `*` replaced by the imputation number (missimp1.dat, missimp2.dat, …); all variables on the `USEVARIABLES` and `AUXILIARY` lists are saved. A companion "list" file (asterisk replaced by the word `list`) is also produced, naming every imputed data set — this list file is what gets passed to `DATA: FILE =` for reuse.
  - When no `MODEL` command is present, imputation uses an unrestricted H1 model; several different algorithms exist for H1 imputation, including sequential (chained) regression (Raghunathan et al., 2001; van Buuren, 2007). Multiple imputation itself is carried out using Bayesian analysis (Rubin, 1987; Schafer, 1997), independent of whichever `ESTIMATOR` is used for a subsequent substantive analysis in the same run.
- `DATA: TYPE = IMPUTATION;` : tells Mplus that `DATA: FILE =` points to a list of already-imputed data sets, so the same model is fit to every imputed data set and the results combined automatically.
- Combining results across imputations: maximum-likelihood parameter estimates are averaged over the set of analyses; standard errors are computed from the average of the standard errors over the set of analyses plus the between-analysis parameter-estimate variation (Rubin, 1987; Schafer, 1997). A chi-square test of overall model fit is provided with maximum-likelihood estimation (Asparouhov & Muthén, 2008c; Enders, 2010).

## Post-run operations
- Check `TECH1` to confirm the intended parameters are free/fixed as expected, and `TECH8` to confirm the estimation (including any numerical integration) converged normally.
- For dropout descriptive/plot runs (Ex 11.2), inspect the graphical display (post-processing graphics module) of dropout means vs. overall sample means across time to visually assess whether dropout looks related to the outcome level.
- For NMAR models (Ex 11.3/11.4), compare substantive conclusions (e.g. growth factor means) against a plain FIML/MAR run — sensitivity of the results to the missingness assumption is itself worth reporting.
- For multiple imputation, verify the imputed-data "list" file was produced, and confirm the combined-analysis output reports parameter estimates averaged across imputations, standard errors reflecting between-imputation variation, and (for ML) a chi-square test of overall fit that accounts for all imputations rather than a single one.

## Likely FAQ mapping
- "Some of my outcome values are missing, what do I do?" → default FIML handling: `MISSING = ALL (code);` + `ESTIMATOR = ML;` (or whichever estimator fits the outcome type)
- "I have another variable that predicts missingness but isn't part of my substantive model" → `AUXILIARY = (m) z;` missing data correlate
- "I want to see whether dropout looks related to my outcome" → `DATA MISSING:` + `ANALYSIS: TYPE = BASIC;` + `PLOT: TYPE = PLOT2;`
- "I suspect people who would have scored higher/lower were the ones who dropped out (missing not at random)" → NMAR selection model (Ex 11.3) or pattern-mixture model (Ex 11.4)
- "I need to create multiply imputed data sets to hand off to another analysis" → `DATA IMPUTATION:` + `SAVE =`
- "How do I reuse the imputed data sets I already created?" → `DATA: FILE = ...list.dat; TYPE = IMPUTATION;`
- "How many imputed data sets should I create?" → default is five; the chapter's examples use 10 (Ex 11.5/11.6) or 20 (Ex 11.7, latent-variable case) for more complete coverage of variability

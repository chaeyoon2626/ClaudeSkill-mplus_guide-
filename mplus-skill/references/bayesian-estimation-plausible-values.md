# Bayesian Estimation — Model-Based Imputation and Plausible Values

> Source: Mplus User's Guide v8, Chapter 11, Examples 11.7–11.8
> Bundled source: references/source-pdfs/Chapter11.pdf

## One-line summary
Use `ESTIMATOR = BAYES` to impute missing data from the substantive (H0) model itself, generate multiple-imputation "plausible value" distributions for latent factors and latent response variables, and combine Bayesian imputation with a frequentist estimator (e.g. two-level WLSMV) that does not itself handle MAR missing data.

## Prerequisite checklist
- [ ] Does the substantive model include latent variables (factors, growth factors) whose score distributions ("plausible values") are needed, not just imputed values for observed variables?
- [ ] Should imputation follow the specific H0 (substantive) model given in `MODEL:`, rather than a generic unrestricted H1 model? (Requires `ESTIMATOR = BAYES` plus a `MODEL` command.)
- [ ] Is the intended downstream analysis estimator (e.g. two-level `WLSMV`) one that cannot itself handle missing data under MAR, making a Bayesian imputation pre-step necessary?
- [ ] How many posterior draws/imputed data sets are needed to adequately represent the variability of the latent variables (the default of five may be too few for latent-variable applications)?
- [ ] Are multiple processors available for parallel computation (`PROCESSORS =`)?

## Option selection logic
| Situation | Choice |
|---|---|
| Want imputation driven by the specific substantive model (H0), not a generic unrestricted model | `ANALYSIS: ESTIMATOR = BAYES;` + a `MODEL:` command + `DATA IMPUTATION:` |
| Want a distribution of factor scores ("plausible values") saved per observation, not just a single point estimate | `SAVEDATA: SAVE = FSCORES; FACTORS = f1 f2 ...;` (requires `ESTIMATOR = BAYES`) |
| Want a distribution of the continuous latent response variables underlying categorical outcomes saved per observation | `SAVEDATA: SAVE = LRESPONSES (ndraws); LRESPONSES = u1 u2 ...;` (requires `ESTIMATOR = BAYES`) |
| Only need imputed values for specific observed analysis variables | Add `IMPUTE = var1 var2 ...;` inside `DATA IMPUTATION:` |
| No `IMPUTE` option given at all | No imputation of missing data on the analysis variables is performed — only whatever else is requested (e.g. plausible values) |
| Downstream estimator (e.g. two-level `WLSMV`) can't handle MAR missing data | Run a Bayesian `DATA IMPUTATION:` pre-step (`ESTIMATOR = BAYES`), save the imputed-data list, then feed it via `DATA: TYPE = IMPUTATION;` into the `WLSMV` run |
| Two-level weighted least squares on saved imputations, want to avoid recomputing sample statistics for each one | `SAVEDATA: SWMATRIX = name*.dat;` in the WLS run on the imputed data, then reuse via `DATA: SWMATRIX = namelist.dat;` in later runs |

## Menu path & screen fields
**[Bayesian model-based imputation + plausible values — Ex 11.7]**
1. **VARIABLE:** `CATEGORICAL = u11-u33;`
2. **ANALYSIS:** `ESTIMATOR = BAYES; PROCESSORS = 2;`
3. **MODEL:** the substantive factor/growth model, e.g. `f1 BY u11 u21-u31 (1-2); ... i s | f1@0 f2@1 f3@2;`
4. **DATA IMPUTATION:** `NDATASETS = 20; SAVE = ex11.7imp*.dat;` (no `IMPUTE` option here — plausible values are the goal, not imputed observed variables)
5. **SAVEDATA:** `FILE = ex11.7plaus.dat; SAVE = FSCORES; FACTORS = f1-f3 i s; SAVE = LRESPONSES (20); LRESPONSES = u11-u33;`
6. **OUTPUT:** `TECH1 TECH8;`

**[Bayesian imputation as a pre-step for a two-level WLSMV analysis — Ex 11.8, part 1]**
1. **VARIABLE:** `CATEGORICAL = u11-u33; CLUSTER = clus; MISSING = ALL (999);`
2. **ANALYSIS:** `TYPE = TWOLEVEL; ESTIMATOR = BAYES; PROCESSORS = 2;`
3. **MODEL:** `%WITHIN%` / `%BETWEEN%` factor model
4. **DATA IMPUTATION:** `IMPUTE = u11-u33 (c); SAVE = ex11.8imp*.dat;`
5. **OUTPUT:** `TECH1 TECH8;`

**[Reusing the imputed data sets with WLSMV — Ex 11.8, part 2]**
1. **DATA:** `FILE = ex11.8implist.dat; TYPE = IMPUTATION;`
2. **VARIABLE:** `CATEGORICAL = u11-u33; CLUSTER = clus;`
3. **ANALYSIS:** `TYPE = TWOLEVEL; ESTIMATOR = WLSMV; PROCESSORS = 2;`
4. **MODEL:** two-level growth-on-factors model
5. **SAVEDATA:** `SWMATRIX = ex11.8sw*.dat;`
6. Later reuse: **DATA:** `FILE = ex11.8implist.dat; TYPE = IMPUTATION; SWMATRIX = ex11.8swlist.dat;`

## Sub-option details
- `ESTIMATOR = BAYES;` : selects Bayesian estimation. Used together with a `DATA IMPUTATION:` block and a `MODEL` command, missing data are imputed from the H0 (substantive) model specified in `MODEL:`, rather than from a generic unrestricted H1 model. Modeling with missing data under Bayesian analysis gives asymptotically the same results as maximum-likelihood estimation under MAR.
- `PROCESSORS = 2;` : requests parallel computation across the given number of processors (used in every Bayesian example in this chapter).
- `DATA IMPUTATION: IMPUTE = ...;` : with `ESTIMATOR = BAYES`, specifies which analysis variables get imputed missing values. If `IMPUTE` is omitted, no imputation of missing data for the analysis variables is performed at all — only whatever else is requested (e.g. plausible values) is produced.
- `DATA IMPUTATION: NDATASETS = 20;` : the number of imputed data sets/posterior draws to create; the default is five. A larger number (e.g. 20 here vs. 10 in the simpler frequentist multiple-imputation examples) is used to more fully represent the variability in the latent variables.
- `SAVEDATA: SAVE = FSCORES;` with `ESTIMATOR = BAYES` : produces a distribution of factor scores — "plausible values" (Mislevy et al., 1992; von Davier et al., 2009) — for each observation, drawn from the Bayesian posterior distribution. Saved per-observation summaries are: mean, median, standard deviation, lower 2.5% limit, and upper 97.5% limit.
- `FACTORS =` : names the latent factors (including growth factors) whose plausible-value distributions are saved alongside `SAVE = FSCORES`.
- `SAVEDATA: SAVE = LRESPONSES (20);` with `ESTIMATOR = BAYES` : produces a distribution of the continuous latent response variable scores underlying categorical outcomes, for each observation, using the same 5-number summary (mean, median, SD, 2.5%, 97.5%). The number in parentheses is the number of imputations/draws used from the Bayesian posterior distribution to compute that distribution.
- `LRESPONSES =` : names the latent response variables (underlying the categorical observed indicators) for which distributions are saved.
- `DATA IMPUTATION: SAVE = name*.dat;` : saves one imputed data set per draw (asterisk replaced by the imputation number) plus a companion "list" file (asterisk replaced by the word `list`), e.g. `ex11.7implist.dat`. Saved imputed data sets contain the observed variables, the continuous latent response variables for any categorical outcomes (suffixed with `*`), and any requested factor scores.
- `SAVEDATA: SWMATRIX =` (used with `TYPE = TWOLEVEL` and a weighted-least-squares estimator) : saves the within- and between-level sample statistics and their estimated asymptotic covariance (weight) matrix to an ASCII file per imputed data set, plus a companion list file — this lets a subsequent `WLSMV` run reuse saved statistics instead of recomputing them for each imputation.
- Rationale for the two-step Bayesian-impute-then-WLSMV workflow (Ex 11.8): the two-level weighted least squares estimator does not itself handle missing data using MAR; running a Bayesian multiple imputation step first removes all missing data before the weighted least squares analysis, sidestepping that limitation. To save computational time in subsequent analyses, the two-level weighted least squares sample statistics and weight matrix for each imputed data set are saved via `SWMATRIX`.

## Post-run operations
- Check `TECH1` for correct parameter specification and `TECH8` to confirm the Bayesian estimation ran without problems.
- Confirm the companion "list" data file (e.g. `ex11.7implist.dat`, `ex11.8implist.dat`) was created and lists every imputed data set — this is the file to pass into `DATA: FILE = ...; TYPE = IMPUTATION;` for any downstream analysis.
- For plausible values, treat the saved mean/median (point estimate), SD, and 2.5%/97.5% limits per observation/factor as the basis for any secondary analysis of the latent variable scores, rather than relying on a single point-estimated factor score.
- For the WLSMV-after-Bayesian-imputation workflow, reuse the saved `SWMATRIX` list together with the imputed-data list file in the follow-up `DATA:` command so sample statistics/weight matrices are not recomputed.

## Likely FAQ mapping
- "How do I use ESTIMATOR=BAYES to impute missing data based on my actual model, not a generic model?" → `ESTIMATOR = BAYES;` + `MODEL:` + `DATA IMPUTATION:`
- "I need plausible values / a distribution of factor scores for each person, not just one factor score" → `SAVEDATA: SAVE = FSCORES; FACTORS = ...;` with `ESTIMATOR = BAYES`
- "My weighted least squares (WLSMV) two-level model can't handle my missing data" → run a Bayesian multiple imputation pre-step, then feed the imputed list into the `WLSMV` run
- "How many posterior draws/imputations do I need for plausible values?" → default is five; the chapter's latent-variable example uses 20 to better capture posterior variability
- "Where do I find details on priors, MODEL PRIORS, BITERATIONS/FBITERATIONS, or convergence diagnostics like PSR (Gelman-Rubin)?" → not covered in this chapter — Chapter 11 only shows `ESTIMATOR = BAYES` in the context of model-based imputation and plausible values; the general technical controls for Bayesian estimation belong to the ANALYSIS command chapter (Ch.16), which is not yet available as a local reference file here — flag this to the user and do a live lookup of that chapter if needed

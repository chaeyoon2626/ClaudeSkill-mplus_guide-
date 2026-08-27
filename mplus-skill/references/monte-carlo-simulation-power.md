# Monte Carlo Simulation Studies

> Source: Mplus User's Guide v8, Chapter 12 (concepts) and Chapter 19 (MONTECARLO command), Example(s) 12.1, 12.2, 12.3, 12.4, 12.6 Step 1, 12.6 Step 2, 12.7 Step 1, 12.7 Step 2
> Bundled source: references/source-pdfs/Chapter12.pdf, references/source-pdfs/Chapter19.pdf

## One-line summary
A Monte Carlo simulation study generates many synthetic datasets from user-specified "true" (population) parameter values, re-fits the analysis model to each one, and summarizes the results across replications — this is how Mplus estimates statistical power, checks parameter/SE bias, and checks coverage for a planned study design before real data are collected.

## Prerequisite checklist
- [ ] The analysis model already worked out in ordinary Mplus `MODEL:` syntax (pull it from the relevant procedure-specific reference file) — it will be reused almost as-is
- [ ] Population ("true") values for every free parameter, i.e. the hypothesized effect sizes — from prior literature, pilot data, or a saved real-data analysis
- [ ] Planned sample size (`NOBSERVATIONS`) and, for multilevel designs, the planned cluster-size structure (`NCSIZES`/`CSIZES`)
- [ ] A large enough number of replications (`NREPS`) — the default is 1, which is unusable for a real study; Chapter 12 examples commonly use 100-500
- [ ] Decide whether the data-generation model and the analysis model are the same (**internal** Monte Carlo, one MONTECARLO run) or need to differ / be reused outside Mplus (**external** Monte Carlo, two-step)
- [ ] The scale of each dependent variable (continuous is the default; binary/ordinal, censored, count, nominal, or survival need the `GENERATE` option)

## Option selection logic
| Situation | Choice |
|---|---|
| Data-generation model = analysis model, single self-contained run | **Internal Monte Carlo**: one input with `MONTECARLO:`, `MODEL POPULATION:`, and `MODEL:` |
| Need to save the generated data sets and analyze them separately (e.g. generation model structurally differs from analysis model, or data were generated outside Mplus) | **External Monte Carlo**: step 1 uses `ANALYSIS: TYPE = BASIC;` + `MONTECARLO: REPSAVE = ALL; SAVE = name*.dat;` to only generate/save data; step 2 uses `DATA: TYPE = MONTECARLO; FILE = ...replist.dat;` to read them back in and analyze |
| Dependent variables continuous | leave `GENERATE` unspecified (default) |
| Dependent variables binary/ordinal | `GENERATE = u1-u4 (1);` + `CATEGORICAL = u1-u4;` |
| Dependent variables censored | `GENERATE = y1 (ca 1);` (ca/cb/cai/cbi for above/below, with/without inflation) + `CENSORED ARE y1 (a);` |
| Dependent variables counts | `GENERATE = u1-u4 (p);` (Poisson; also nb, ci/pi, nbi, nbt, nbh) + `COUNT ARE u1-u4;` |
| Need realistic missing data / attrition for a power study | single group: `PATMISS =` + `PATPROBS =`; multiple patterns predicted by covariates (MAR) or multi-group: `MISSING = y1-y4;` + `MODEL MISSING:` logistic-regression block |
| Clustered / multilevel design (e.g. students in schools) | `ANALYSIS: TYPE = TWOLEVEL;` + `MONTECARLO: NCSIZES = k; CSIZES = ...; WITHIN = ...; BETWEEN = ...;` + `%WITHIN%`/`%BETWEEN%` in both `MODEL POPULATION:` and `MODEL:` |
| Population values should come from a real fitted model instead of being hand-typed | fit the real model first, `SAVEDATA: ESTIMATES = est.dat;`, then in the simulation `MONTECARLO: POPULATION = est.dat; COVERAGE = est.dat;` |
| Goal is specifically a power estimate for one or more parameters | make sure the population value for that parameter is non-zero, set `NREPS` reasonably large (e.g. 500), and read the `% Sig Coeff` column in `MODEL RESULTS` |

## Menu path & screen fields
1. **MONTECARLO:**
   - `NAMES ARE y1-y4 x1 x2;` — names for the generated variables (required)
   - `NOBSERVATIONS = 500;` — sample size for data generation and analysis (required); for multiple groups, `NOBSERVATIONS = 500 1000;`
   - `NREPS = 500;` — number of replications/datasets to draw (default 1 — always raise this for a real study)
   - `SEED = 4533;` — random seed (default 0)
   - `GENERATE = ...;` / `CUTPOINTS = ...;` / `CATEGORICAL = ...;` / `CENSORED ARE ...;` / `COUNT ARE ...;` as needed for non-continuous variables
   - `MISSING = ...;` / `PATMISS = ...; PATPROBS = ...;` if missing data should be generated
   - `NCSIZES = ...; CSIZES = ...; WITHIN = ...; BETWEEN = ...;` if the design is clustered/multilevel
2. **MODEL POPULATION:** the "true" model — every parameter is given a population value, e.g. `f BY y1@1 y2-y4*1; f*.5; y1-y4*.5; f ON x1*1 x2*.3;`
3. **MODEL MISSING:** (only if `MISSING =` was used) logistic-regression specification of the missingness/attrition-generating process, e.g. `[y1-y4@-1]; y1 ON x1*.4 x2*.2;`
4. **ANALYSIS:** any options the analysis model needs, e.g. `TYPE = TWOLEVEL;`, `TYPE = MIXTURE;`, `TYPE = COMPLEX;`, `ESTIMATOR = MLR;`
5. **MODEL:** the analysis model actually fit to each generated dataset — normally mirrors `MODEL POPULATION:`, but can deliberately differ to study misspecification
6. **OUTPUT:** `TECH9;` — recommended; prints error/warning messages per replication (e.g. non-convergence) so problem replications can be diagnosed

## Sub-option details
- `NAMES` / `NOBSERVATIONS` / `NREPS` / `SEED` — basic setup; the hyphen list shortcut works the same as elsewhere (`y1-y4` expands to `y1 y2 y3 y4`)
- `GENERATE = u1-u2 (1) u3 (1 p) u5 u6 (2 p);` — sets the scale/model used to generate each dependent variable: number of thresholds + `l`(logistic, default)/`p`(probit) for categorical, `ca`/`cb`/`cai`/`cbi` for censored, `p`/`nb`/`ci`/`pi`/`nbi`/`nbt`/`nbh` for counts, `n`+count for nominal intercepts, `s`+intervals for survival
- `CUTPOINTS = x1 (0) x2 (1);` — dichotomizes generated continuous independent variables at the given value (values ≤ cutpoint become 0)
- `MODEL POPULATION:` — every parameter must be given a value after `@` or `*`; any parameter not given a value defaults to population value 0
- `POPULATION = est.dat;` / `COVERAGE = est.dat;` / `STARTING = est.dat;` — read population values for data generation / coverage computation / analysis starting values from a file previously written by `SAVEDATA: ESTIMATES = est.dat;`
- `PATMISS = y1(.1) y2(.2) y3(.3) y4(1) | y1(1) y2(.1) y3(.2) y4(.3);` + `PATPROBS = .4 | .6;` — defines missing-data patterns (per-variable missing proportions, patterns separated by `|`) and what proportion of generated cases falls in each pattern; not available for multiple-group analysis
- `MISSING = y1-y4;` + `MODEL MISSING:` — alternative missingness approach that supports MAR (missingness predicted by covariates/outcomes) and multiple groups; the intercept/slope values in `MODEL MISSING:` are logistic-regression coefficients for a binary "is-missing" indicator per variable
- `NCSIZES` / `CSIZES = 40 (5) 50 (10) 20 (15);` — number of unique cluster sizes and, for each, how many clusters of that size to generate (here: 40 clusters of size 5, 50 of size 10, 20 of size 15)
- `WITHIN = x;` / `BETWEEN = w;` — declares which generated variables are individual-level-only vs. cluster-level-only, mirroring the `VARIABLE:` command's role in a real two-level analysis
- `REPSAVE = ALL;` + `SAVE = rep*.dat;` — saves generated datasets to disk (asterisk replaced by replication number); a companion file with `*` replaced by `list` (e.g. `replist.dat`) lists all saved filenames for use as `DATA: FILE =` in a follow-up external Monte Carlo step
- `RESULTS = results.sav;` — saves per-replication parameter estimates, SEs, and fit statistics to an ASCII file

## Post-run operations
- Read the `MODEL RESULTS` output table, which reports per parameter: **Population** (the true value from `MODEL POPULATION:`/`MODEL:`/`COVERAGE`), **Average** (mean estimate across replications — compare to Population for bias), **Std. Dev.** (empirical SD of the estimates), **S.E. Average** (mean of the model-based standard errors — compare to Std. Dev. for SE bias), **M.S.E.** (mean squared error = variance + bias²), **95% Cover** (proportion of replications whose 95% CI contains the population value — should be close to 0.95), and **% Sig Coeff** (proportion of replications where the parameter was significant at .05).
- **This is how power is read**: for a parameter with a non-zero population value, `% Sig Coeff` is an estimate of statistical **power**; for a parameter fixed at population value 0, the same column is an estimate of **Type I error**.
- To answer "what sample size do I need for 80% power," rerun the simulation with different `NOBSERVATIONS` (or `CSIZES` for a clustered design) and compare `% Sig Coeff` for the parameter of interest across runs.
- The default output also gives a Chi-Square Test of Model Fit summary (expected vs. observed proportions/percentiles across replications) — a large mismatch between expected and observed proportions signals that the reference chi-square distribution is not well approximated for that model/sample size.
- Percentage bias for a parameter: `100 * (Average - Population) / Population`.
- Check `TECH9` for non-convergence or error messages on specific replications before trusting the summary — replications with problems can distort Average/Std. Dev./coverage.
- To reuse the study later or feed data to another program: `REPSAVE = ALL;` + `SAVE = name*.dat;` together with `ANALYSIS: TYPE = BASIC;` writes out the generated data without analyzing it.
- To resume an external analysis on previously saved data: `DATA: FILE = replist.dat; TYPE = MONTECARLO;` in a second Mplus run, then specify `VARIABLE:`, `ANALYSIS:`, and `MODEL:` as in a normal analysis (population values placed in `MODEL:` there are used only for coverage/printing, not generation).
- To ground the simulation's effect sizes in real data instead of guessing them: fit the model to the real dataset, `SAVEDATA: ESTIMATES = est.dat;`, then in the simulation input use `MONTECARLO: POPULATION = est.dat; COVERAGE = est.dat;`.

## Likely FAQ mapping
- "How do I do a power analysis for a planned study in Mplus before collecting data?" → internal Monte Carlo: put hypothesized effect sizes in `MODEL POPULATION:`, set `NOBSERVATIONS` to the planned N, `NREPS` ≥ ~500, and read `% Sig Coeff` for the key parameter(s) as power
- "What sample size gives me 80% power to detect this effect?" → rerun with a few candidate `NOBSERVATIONS`/`CSIZES` values and compare `% Sig Coeff`
- "I have pilot/real data — can I use its estimates as my simulated effect sizes instead of typing numbers by hand?" → fit the real model, `SAVEDATA: ESTIMATES = est.dat;`, then `MONTECARLO: POPULATION = est.dat; COVERAGE = est.dat;` (Example 12.7 Step 1/2)
- "I want to see how badly a misspecified model performs (fewer classes than truth, missing path, etc.)" → make `MODEL POPULATION:` and `MODEL:` differ, then inspect the bias/coverage columns in `MODEL RESULTS` (Example 12.3)
- "My study will have dropout/missing data — can the simulation reflect that?" → `PATMISS`/`PATPROBS` for fixed missing-data patterns, or `MISSING =` + `MODEL MISSING:` if missingness should depend on covariates (MAR/attrition) or the design has multiple groups
- "My design is students-in-schools (or similarly clustered) — how do I simulate that?" → `ANALYSIS: TYPE = TWOLEVEL;` + `NCSIZES`/`CSIZES` + `WITHIN`/`BETWEEN`, with `%WITHIN%`/`%BETWEEN%` blocks in both `MODEL POPULATION:` and `MODEL:`
- "I generated data with a different model than I want to analyze it with (or generated it outside Mplus)" → external Monte Carlo: save with `REPSAVE = ALL; SAVE = name*.dat;` under `ANALYSIS: TYPE = BASIC;`, then re-read with `DATA: TYPE = MONTECARLO; FILE = ...replist.dat;` in a second run
- "I only want to see one particular result from a Monte Carlo run (e.g. a specific TECH option)" → combine with `output-savedata-plot-commands.md`

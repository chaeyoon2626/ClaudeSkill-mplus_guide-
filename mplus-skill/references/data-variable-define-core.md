# TITLE, DATA, VARIABLE, and DEFINE Commands — Core Syntax Reference

> Source: Mplus User's Guide v8, Chapter 15
> Bundled source: references/source-pdfs/Chapter15.pdf

## One-line summary
These four commands appear at the top of essentially every Mplus .inp file: TITLE labels the run, DATA points to and describes the raw data file, VARIABLE names/selects/typess the variables (missing codes, categorical vs. continuous, clustering, weighting, grouping, number of latent classes), and DEFINE computes new variables or transforms existing ones (centering, standardizing, sums/means, recoding). Always consult this file **alongside** the procedure-specific reference file for the statistical model itself.

## Prerequisite checklist
Before writing these blocks, confirm with the user:
- [ ] **Data file location/format**: raw ASCII file path; fixed-format (needs a FORTRAN-like FORMAT statement) or free-format (default; comma/space/tab-delimited, no blanks allowed)
- [ ] **Full variable list and order** as columns appear in the file (needed for `NAMES ARE`)
- [ ] **Which variables are actually used in this analysis** (`USEVARIABLES`) vs. just present in the file
- [ ] **Missing-value coding** in the raw data (a number like 9/-99, or `.`/`*`/blank)
- [ ] **Measurement scale of each dependent variable**: continuous (default), censored, binary/ordinal categorical, unordered categorical (nominal), or count
- [ ] **Any survey/clustering structure**: multistage sampling (STRATIFICATION/CLUSTER/WEIGHT) or nested data (CLUSTER with TYPE=TWOLEVEL/THREELEVEL)
- [ ] **Any grouping variable** (multiple-group analysis) or **latent class count** (mixture models)
- [ ] **Any variables that must be computed** from existing ones (sum/mean scores, centering, standardization, recoding)

## Option selection logic (branching by data/situation)

| Situation | Choice |
|---|---|
| Data are individual-level rows × columns, no header row | `DATA: FILE IS ...;` with default `TYPE = INDIVIDUAL;` (no TYPE needed) |
| Data are a summary covariance/correlation matrix, not raw cases | `DATA: TYPE = CORRELATION MEANS STDEVIATIONS;` (or `COVARIANCE`, `FULLCOV`, `FULLCORR`) + `NOBSERVATIONS = ...;` |
| Data are already multiply-imputed data sets (list of file names) | `DATA: TYPE = IMPUTATION; FILE IS <file listing the imputed data set names>;` |
| Need to *create* the imputed data sets first | Separate `DATA IMPUTATION:` command (see below), then a later run uses `TYPE = IMPUTATION;` |
| Raw data file has no variable name header | `VARIABLE: NAMES ARE ...;` listing every column in order |
| Only some variables in NAMES are analyzed | `USEVARIABLES ARE ...;` |
| Missing values coded with a specific number/symbol | `MISSING ARE ...;` |
| DV piles up at a floor/ceiling | `CENSORED ARE y1 (b);` / `(a)` |
| DV is binary/ordinal (Likert, yes/no) | `CATEGORICAL ARE ...;` |
| DV is unordered categories (e.g. which brand chosen) | `NOMINAL ARE ...;` |
| DV is a count (0,1,2,3 events) | `COUNT ARE ...;` |
| Data for all groups are stacked in one file | `GROUPING IS ...;` |
| Data are nested (students in schools, repeated measures in persons) | `CLUSTER IS ...;` (+ `TYPE=TWOLEVEL`/`COMPLEX` in ANALYSIS) |
| Complex survey sample with strata | `STRATIFICATION IS ...;` (with `CLUSTER` and `WEIGHT`, `TYPE=COMPLEX`) |
| Unequal probability of selection | `WEIGHT IS ...;` |
| Mixture/latent class model | `CLASSES = c (2);` (number of classes in parentheses) |
| Need a composite/sum/mean score from items | `DEFINE: newvar = SUM(item1-item9);` or `MEAN(...)` |
| Need a predictor centered before an interaction term | `DEFINE: CENTER x1 x2 (GRANDMEAN);` (or `GROUPMEAN` for clustered/multilevel data) |
| Need variables on a common (z) scale | `DEFINE: STANDARDIZE y1 y2;` |

## Menu path & screen fields (categorized option list)
Mplus has no GUI menus — the .inp text file *is* the interface. The blocks below are the ordered "screens."

**TITLE:**
- `TITLE:` free-text description of the analysis; not required; printed in the output before the Summary of Analysis

**DATA:**
- `FILE IS` — path/name of the ASCII data file (required)
- `FORMAT IS` — `FREE` (default) or a fixed FORTRAN-like format statement
- `TYPE IS` — `INDIVIDUAL` (default) / `COVARIANCE` / `CORRELATION` / `FULLCOV` / `FULLCORR` / `MEANS` / `STDEVIATIONS` / `MONTECARLO` / `IMPUTATION`
- `NOBSERVATIONS ARE` — required for summary (matrix) data; optional cap on rows read for individual data
- `NGROUPS =` — number of groups for multiple-group summary-data analysis
- `LISTWISE =` — `ON`/`OFF` (default `OFF`) — listwise-delete cases with any missing analysis variable
- `SWMATRIX =` — saved within/between sample-statistics file for TYPE=TWOLEVEL WLS
- `VARIANCES =` — `CHECK` (default) / `NOCHECK` — whether to check for zero-variance variables

**DATA IMPUTATION:** (creates multiply-imputed data sets)
- `IMPUTE =` — variables to impute (append `(c)` for categorical)
- `NDATASETS =` — number of imputed data sets (default 5)
- `SAVE =` — output file-name pattern, e.g. `impute*.dat`
- `FORMAT =` — save format (default fixed, analysis variables)
- `MODEL =` — `COVARIANCE` (default) / `SEQUENTIAL` / `REGRESSION`
- `VALUES =` — restrict imputed values to a range
- `ROUNDING =` — decimal places for imputed continuous variables (default 3)
- `THIN =` — every k-th posterior draw used (default 100)

**Other DATA transformation commands** (rearranging/deriving variables before analysis — used less often; mentioned here for completeness): `DATA WIDETOLONG` / `DATA LONGTOWIDE` (reshape repeated-measures data), `DATA TWOPART` (splits a semicontinuous variable into a binary + continuous pair), `DATA MISSING` (creates missing/dropout indicator variables), `DATA SURVIVAL` (creates discrete-time event-history indicators), `DATA COHORT` (rearranges multiple-cohort data by age).

**VARIABLE:**
- `NAMES ARE` — names of every variable in the data file, in column order (required)
- `USEOBSERVATIONS ARE` — conditional statement selecting a subset of rows
- `USEVARIABLES ARE` — names of variables actually used in the analysis (default: all of NAMES)
- `MISSING ARE` — missing-value flag(s) per variable or `ALL`
- `CENSORED ARE` — censored dependent variables + direction
- `CATEGORICAL ARE` — binary/ordinal dependent variables
- `NOMINAL ARE` — unordered-categorical dependent variables
- `COUNT ARE` — count dependent variables
- `DSURVIVAL ARE` — discrete-time survival variables (for PLOT)
- `GROUPING IS` — multiple-group indicator variable + labels
- `IDVARIABLE IS` — case identifier (or `_RECNUM`)
- `FREQWEIGHT IS` — frequency/case-weight variable
- `TSCORES ARE` — individually-varying times of observation (growth models)
- `AUXILIARY =` — variables saved/plotted but not modeled; also missing-data correlates `(M)` or 3-step mixture settings (`R3STEP`, `BCH`, etc.)
- `CONSTRAINT =` — variables usable in `MODEL CONSTRAINT`
- `PATTERN IS` — pattern variable for data missing by design
- `STRATIFICATION IS` — sampling-stratum variable (complex survey data)
- `CLUSTER IS` — clustering variable(s) (multilevel / complex survey data)
- `WEIGHT IS` — sampling weight variable
- `WTSCALE IS` — `UNSCALED` / `CLUSTER` (default) / `ECLUSTER` — rescaling of the WEIGHT variable under TYPE=TWOLEVEL
- `BWEIGHT`, `B2WEIGHT`, `B3WEIGHT`, `BWTSCALE` — between-level sampling weights for TWOLEVEL/THREELEVEL
- `REPWEIGHTS ARE` — replicate weight variables
- `SUBPOPULATION IS` — conditional statement selecting a subpopulation/domain
- `FINITE =` — finite population correction variable (`FPC`/`SFRACTION`/`POPULATION`)
- `CLASSES =` — categorical latent variable name(s) + number of classes (required for TYPE=MIXTURE)
- `KNOWNCLASS =` — categorical latent variable with class membership known from an observed variable
- `TRAINING =` — variables carrying class-membership/probability/prior information
- `WITHIN` / `BETWEEN` — individual-level vs. cluster-level variables (multilevel models)
- `SURVIVAL` / `TIMECENSORED` — continuous-time survival variables and right-censoring indicator
- `LAGGED` — maximum lag for a time-series variable
- `TINTERVAL IS` — time-interval variable for misaligned time-series data

**DEFINE:**
- `variable = mathematical expression;` — compute a new/transformed variable
- `IF (conditional statement) THEN transformation statements;` — conditional recoding
- `_MISSING` — keyword referring to a missing value inside DEFINE
- `variable = MEAN(list);` / `variable = SUM(list);` — composite scores
- `CUT variable(s) (cutpoints);` — categorize a continuous variable
- `variable = CLUSTER_MEAN(variable);` — cluster-average of an individual-level variable
- `CENTER variable(s) (GRANDMEAN);` / `(GROUPMEAN)` — mean-centering
- `STANDARDIZE variable(s);` — z-standardize (mean 0, SD 1)
- `DO (start, end) expression;` / double `DO ($,..) DO (#,..) expression;` — loop to repeat a transformation across a numbered set of variables

## Sub-option details

### DATA command
- `FILE IS c:\analysis\data.dat;` — required; if the path contains blanks it must be quoted; if given without a path, the local directory (and then the directory of the .inp file) is searched
- `FORMAT IS 5F4.0, 10x, 6F1.0;` — FORTRAN-like descriptors: `F` = real number (`F5.3` = 5 columns, 3 decimals; a leading count like `5F5.3` repeats the descriptor), `x` = skip columns (`10x` skips 10), `t` = jump to a column (`t130`), `/` = go to next record. Free format (default) requires comma/space/tab delimiters and disallows blank fields.
- `TYPE IS CORRELATION MEANS STDEVIATIONS;` — for summary data, each type of statistic must start on its own record in the external file, means first, then SDs, then the lower-triangular correlation (or covariance) matrix row-wise; `NOBSERVATIONS` is required in this case. Binary/ordinal dependent variables in summary data restrict TYPE to a correlation matrix.
- `TYPE = IMPUTATION;` — `FILE IS` then names a file that itself lists the imputed data set file names (one per line), e.g. `imp1.dat` ... `imp5.dat`; estimates are averaged across the imputations and SEs use the Rubin (1987) formula.
- `NOBSERVATIONS = 1000;` — required for summary data; for individual data it can subset to the first N rows.
- `LISTWISE = ON;` — turns on listwise deletion of any case missing on an analysis variable; default is to use all available data under missing-data theory.
- `VARIANCES = NOCHECK;` — turns off the default check for zero-variance analysis variables.

### DATA IMPUTATION command
- `IMPUTE = y1-y4 u1-u4 (c) x1 x2;` — continuous variables listed plainly, categorical ones flagged `(c)`; `IMPUTE = (c) x1 x3 x5;` applies `(c)` to everything after the `=`; `IMPUTE = ALL (c);` imputes every variable as categorical.
- `NDATASETS = 20;` — how many imputed data sets to create (default 5).
- `SAVE = impute*.dat;` — the `*` is replaced by the imputation number; a companion file listing all data set names is also produced (`*` replaced by `list`).
- `MODEL = SEQUENTIAL;` — chained-equations/sequential-regression imputation instead of the default unrestricted means/variances/covariances model; `REGRESSION` regresses variables with missing data on variables without.
- `VALUES = y1-y4 (1-5);` — restricts imputed values to a set/range; the closest allowed value is used.
- `ROUNDING = y1-y10 (5);` — decimal places for imputed continuous values (0 = integers).
- `THIN = 200;` — use every 200th posterior draw instead of the default every 100th.

### VARIABLE command — naming, subsetting, missing values
- `NAMES ARE gender ethnic income educatn drink_st agedrink;` — required; names ≤ 8 characters, letters/numbers/underscore only, must start with a letter; case-insensitive; `y1-y5 x1-x3` expands to `y1 y2 y3 y4 y5 x1 x2 x3`.
- `USEOBSERVATIONS = ethnic EQ 1 AND gender EQ 2;` — logical operators only (`AND OR NOT EQ NE GE LE GT LT`, or symbolic `== /= >= <= > <`); only NAMES-list variables allowed; not available for summary data.
- `USEVARIABLES ARE gender income agefrst;` — original NAMES variables must be listed before any DEFINE-created/DATA-transformation-created variables; within each of those two groups, any order is fine. `USEVARIABLES = ALL hd1 hd2 hd3;` keeps every original variable plus the three new ones.
- `MISSING ARE . ;` — period as the missing flag for all variables; `MISSING = BLANK;` (fixed format only); `MISSING ARE ethnic (9 99) y1 (1);` — per-variable flags; `MISSING ARE ALL (9);` or with the list function `MISSING ARE ALL (9 99-102);` / `MISSING ARE gender-income (9 30 98-102);`. Negative flags need commas to separate them from a range, e.g. `MISSING ARE ethnic (-9, -99);`.

### VARIABLE command — measurement scale of dependent variables
- `CENSORED ARE y1 (a) y2 (b);` — `(a)` = censored from above, `(b)` = censored from below; add `i` for a censored-inflated model, e.g. `(bi)`; the inflation part is referred to in MODEL as `y1#1`.
- `CATEGORICAL ARE u2 u3 u7-u13;` — binary/ordinal dependent variables (max 10 categories, determined from the data by default); thresholds referenced in MODEL as `u1$1`, `u1$2`, etc. Variants: `(gpcm)` generalized partial credit, `(3pl)`/`(4pl)` IRT logistic models, `(*)` recode using the observed categories for the set as a whole (useful in multiple-group/growth models), or `(1-6)`/`(2 4 6)` to fix the allowed category set explicitly; different variables can get different settings separated by `|`.
- `NOMINAL ARE u1 u2 u3 u4;` — unordered categorical DVs (max 10 categories); categories referenced as `u1#1`, `u1#2`, ... (last category is the reference).
- `COUNT ARE u1-u4 (p);` — Poisson (`p`, default if no letter given); `(i)`/`(pi)` zero-inflated Poisson; `(nb)` negative binomial; `(nbi)` zero-inflated negative binomial; `(nbt)` zero-truncated negative binomial (values must be > 0); `(nbh)` negative binomial hurdle. Inflation/hurdle parts referenced as `u1#1`.
- `DSURVIVAL = u1-u4;` — flags discrete-time survival variables so PLOT can draw survival curves.

### VARIABLE command — special-function and design variables
- `GROUPING IS gender (1=male 2=female);` — one grouping variable, integer-valued; label(s) used in group-specific MODEL statements; a shorthand `GROUPING = country (101-200 225 350-360);` lists the values to include, or `GROUPING = country (34);` just gives the group count (labels default to `g1`, `g2`, ...).
- `IDVARIABLE = id;` — identifier saved with SAVEDATA output (max length 16); `IDVARIABLE = _RECNUM;` uses the data file's record number when there's no ID column.
- `FREQWEIGHT IS casewgt;` — case/frequency weights (integer); not available with TYPE=COMPLEX/TWOLEVEL/THREELEVEL/CROSSCLASSIFIED/EFA (or TYPE=RANDOM without numerical integration).
- `AUXILIARY = gender race educ;` — saved/plotted, not modeled; `AUXILIARY = z1-z4 (M);` adds them as missing-data correlates (ML, TYPE=GENERAL, continuous DVs only); `AUXILIARY = race (R3STEP) x1-x5 (R3STEP);` or `(BCH)`/`(DU3STEP)`/`(DCATEGORICAL)`/`(DE3STEP)`/`(DCONTINUOUS)`/`(E)` automate the 3-step mixture-model approach (R3STEP/BCH preferred; only one categorical latent variable and one of the 8 settings per analysis).
- `CONSTRAINT = y1 u1;` — variables usable inside `MODEL CONSTRAINT`; cannot include variables already used by GROUPING/PATTERN/COHORT/COPATTERN/CLUSTER/STRATIFICATION/AUXILIARY; not available for TYPE=RANDOM/TWOLEVEL/THREELEVEL/CROSSCLASSIFIED/COMPLEX or non-ML(R/F) estimators.
- `PATTERN IS design (1=y1 y3 y5 2=y2 y3 y4 3=y1 y4 y5);` — for data missing by design; each design value lists which variables should be non-missing for that pattern.
- `STRATIFICATION IS region;` — used with `TYPE=COMPLEX`; identifies the sampling strata.
- `CLUSTER IS school;` — one cluster variable for TYPE=TWOLEVEL/COMPLEX; two for TYPE=THREELEVEL/CROSSCLASSIFIED (`CLUSTER IS school class;`, highest level first for nested data); three for TYPE=COMPLEX THREELEVEL.
- `WEIGHT IS sampwgt;` — sampling weights (non-negative); rescaled to sum to N if they don't already; available for ESTIMATOR = MLR, MLM, MLMV, WLS, WLSM, WLSMV, ULS (with two exceptions: not with WLS when all DVs are continuous, and not with MLM/MLMV for EFA).
- `WTSCALE = ECLUSTER;` — for TYPE=TWOLEVEL, rescales the within-level WEIGHT variable to sum to the effective sample size per cluster (`CLUSTER` default rescales to the raw sample size per cluster; `UNSCALED` applies no adjustment).
- `BWEIGHT = bweight;` (TWOLEVEL) / `B2WEIGHT` + `B3WEIGHT` (THREELEVEL) — between-level sampling weight(s); `BWTSCALE = UNSCALED;` (default `SAMPLE`, which rescales so between × within weights sum to the total sample size).
- `REPWEIGHTS = rweight1-rweight80;` — replicate weight variables (used with `WEIGHT` and the ANALYSIS `REPSE` option); cannot be combined with `STRATIFICATION`/`CLUSTER`/`SUBPOPULATION`.
- `SUBPOPULATION = gender EQ 2;` — selects a domain for TYPE=COMPLEX analysis; all observations stay in the run but non-subpopulation cases get zero weight; cannot combine with `USEOBSERVATIONS` or multiple-group analysis.
- `FINITE IS sampfrac (SFRACTION);` — finite population correction input; settings `FPC` (default), `SFRACTION`, `POPULATION`.
- `CLASSES = c1 (2) c2 (2) c3 (3);` — required for TYPE=MIXTURE; assigns names and class counts; with more than one categorical latent variable, only "later" classes (per CLASSES order) can be regressed on "earlier" ones in MODEL (this ordering restriction doesn't apply to PARAMETERIZATION=LOGLINEAR).
- `KNOWNCLASS = c1 (gender = 0 1);` — multiple-group mixture analysis where class membership is known and equals an observed grouping variable; shorthand `KNOWNCLASS = c (country);` takes the class values straight from the data.
- `TRAINING = t1 t2 t3 (MEMBERSHIP);` — (default setting) hard class-membership indicators; `(PROBABILITIES)`/`(PRIORS)` supply fractional class-membership information that must sum to 1 per person.
- `WITHIN = y1 y2 x1;` / `BETWEEN = z1 z2 x1;` — individual-level vs. cluster-level variables for TYPE=TWOLEVEL/THREELEVEL/CROSSCLASSIFIED; label the cluster level for THREELEVEL/CROSSCLASSIFIED, e.g. `WITHIN = y1-y3 (class) y4-y6 (school) y7-y9;`.
- `SURVIVAL = t;` / `TIMECENSORED = tc;` — continuous-time survival outcome and its right-censoring indicator (0 = event, 1 = right-censored by default; recode with e.g. `TIMECENSORED = tc (1=NOT 999=RIGHT);`); `SURVIVAL = t (10);` requests 10 semi-parametric baseline-hazard intervals, `t (ALL)` a fully saturated (Cox-style) baseline hazard, `t (CONSTANT)` a constant hazard.
- `LAGGED = y (1);` — max lag 1 for time-series variable `y`, referenced in MODEL as `y&1`; `LAGGED = y1-y3 (2);` gives each of y1-y3 a max lag of 2 (`y1&1`, `y1&2`, ...).
- `TINTERVAL = time (1);` — builds a time variable from a raw time-interval variable when measurement occasions are misaligned (data must be sorted by the interval variable).

### DEFINE command
- Statements execute in order, one observation at a time, with one exception: `CLUSTER_MEAN`, `CENTER`, and `STANDARDIZE` are all executed together (in the order they appear) *after* every transformation that precedes them in DEFINE/DATA-transformation commands; anything written after them then runs using their new values.
- Non-conditional: `y = y/100;` transforms in place; `abuse = item1 + item2 + item8 + item9;` creates a new variable (missing on the right-hand side propagates to missing on the left).
- Conditional: `IF (gender EQ 1 AND ses EQ 1) THEN group = 1;` — missing input variables (or an unmatched condition with no covering ELSE-style branch) leave the target variable missing.
- `_MISSING` keyword: `IF (y EQ 0) THEN u = _MISSING;` (assign missing) or `IF (y EQ _MISSING) THEN u = 1;` (test for missing).
- Logical operators: `AND OR NOT EQ(==) NE(/=) GE(>=) LE(<=) GT(>) LT(<)`. Arithmetic: `+ - * / ** (exponent) % (remainder)`. Functions: `LOG LOG10 EXP SQRT ABS SIN COS TAN ASIN ACOS ATAN PHI` (standard normal CDF, e.g. `PHI(y)` or `PHI(#)`).
- `mean = MEAN(y1 y3 y5);` — average across the named variables (or `MEAN(y1-y10)` via the list function), based only on the non-missing subset; missing on *all* of them yields missing.
- `sum = SUM(y1 y3 y5);` — sum across the named variables; missing on *any one* of them yields missing on the sum (unlike MEAN). Both MEAN and SUM must use original NAMES-list or DEFINE-created variables (list-function order follows NAMES order, not USEVARIABLES).
- `CUT y1 y5-y7 (30 40);` — categorizes each named variable using the same cutpoints into 0/1/2 (≤30, >30 & ≤40, >40); missing input → missing output.
- `clusmean = CLUSTER_MEAN(x);` — cluster average of variable `x` (TYPE=TWOLEVEL/COMPLEX with CLUSTER); a cluster where everyone is missing on `x` gets a missing cluster mean. New DEFINE-created variables used with CLUSTER_MEAN must be added to USEVARIABLES after the originals.
- `CENTER x1-x4 (GRANDMEAN);` — subtracts the overall mean; available for every continuous observed analysis variable. `CENTER x1-x4 (GROUPMEAN);` subtracts the cluster mean (TWOLEVEL/THREELEVEL/CROSSCLASSIFIED/COMPLEX with CLUSTER); for THREELEVEL/CROSSCLASSIFIED you can target a specific cluster level, e.g. `CENTER x1-x2 (GROUPMEAN class); CENTER x3 x4 (GROUPMEAN school);` (a variable can only be group-mean-centered on one level).
- `STANDARDIZE y1 y5-y10 y14;` — subtracts the mean and divides by the SD (z-score); in multiple-group analysis each group is standardized using its own mean/SD.
- `DO (1, 5) diff# = y# - x#;` — do-loop; `#` is replaced by 1..5 in turn, producing `diff1 = y1 - x1;` ... `diff5 = y5 - x5;`. Double loop: `DO ($,1,2) DO (#,1,4) y$# = x$ * u#;` produces `y11 = 0.1*u1; ... y24 = 2*u4;` — note the numbers substituted for `$` and `#` are the literal values you supply, not indices.
- New variables created in DEFINE that will be used in the analysis must be listed on `USEVARIABLES` *after* the original NAMES variables.

## Post-run operations
- Verify the variable roster actually read matches expectations: check the "SUMMARY OF ANALYSIS" and variable list at the top of the .out file against `NAMES`/`USEVARIABLES`.
- Check "NUMBER OF OBSERVATIONS" and any "SAMPLE STATISTICS" section against the expected sample size — a mismatch usually means a `FORMAT`/delimiter problem or an unhandled missing-value code.
- If a censored/categorical/nominal/count DV was declared, confirm the output's "UNIVARIATE PROPORTIONS AND COUNTS" (or equivalent) section shows the expected number of categories/levels — an unexpected category count usually means the raw missing-value flag wasn't declared via `MISSING`.
- For `DEFINE`-created variables used in `MODEL`, double-check they were added to `USEVARIABLES` — omitting this is the most common reason a new variable "doesn't show up."
- For clustered/weighted designs, confirm the requested `TYPE=COMPLEX`/`TWOLEVEL` (in ANALYSIS) matches the `CLUSTER`/`STRATIFICATION`/`WEIGHT` variables declared here — VARIABLE alone does not turn on complex-survey or multilevel estimation.

## Likely FAQ mapping
- "My data file has no header row — how do I tell Mplus the column names?" → `VARIABLE: NAMES ARE ...;` listing every column in file order
- "I only want to analyze some of my variables" → `USEVARIABLES ARE ...;`
- "How do I code missing values?" → `VARIABLE: MISSING ARE <flag>;` (numeric flag, or `.`/`*`/`BLANK`; per-variable flags in parentheses after the variable name)
- "My outcome is a Likert/ordinal item" → `CATEGORICAL ARE ...;`
- "My outcome is which of several unordered options someone picked" → `NOMINAL ARE ...;`
- "My outcome is a count of events (0,1,2,3...)" → `COUNT ARE ...;` (add `(nb)` if overdispersed)
- "My data have students nested in schools" → `CLUSTER IS school;` + `ANALYSIS: TYPE = TWOLEVEL;` (or `COMPLEX` for SE/chi-square correction only)
- "I have survey weights / a stratified sample" → `STRATIFICATION IS ...; CLUSTER IS ...; WEIGHT IS ...;` with `ANALYSIS: TYPE = COMPLEX;`
- "I want a multiple-group model with one stacked data file" → `GROUPING IS group (1=grp1 2=grp2);`
- "I'm running an LPA/LCA — how do I set the number of classes?" → `CLASSES = c (3);` + `ANALYSIS: TYPE = MIXTURE;`
- "How do I make a new variable from existing ones (e.g. a total score)?" → `DEFINE: total = SUM(item1-item9);` (use `MEAN(...)` if you want the average instead, which tolerates some missing items)
- "How do I center a predictor before creating an interaction term?" → `DEFINE: CENTER x1 x2 (GRANDMEAN);` then build the product term afterward so it uses the centered values (use `GROUPMEAN` instead for a multilevel/clustered centering)
- "How do I standardize/z-score a variable?" → `DEFINE: STANDARDIZE y1 y2;`
- "My data are already multiply imputed — how do I analyze all the imputations at once?" → `DATA: TYPE = IMPUTATION; FILE IS <file listing each imputed data set's name>;`
- "My data are summary statistics (a correlation/covariance matrix), not raw cases" → `DATA: TYPE = CORRELATION MEANS STDEVIATIONS;` (or `COVARIANCE`) + `NOBSERVATIONS = ...;`

# Two-Level and Cross-Classified Time Series (Autoregressive / Cross-Lagged) Models

> Source: Mplus User's Guide v8, Chapter 9, Examples 9.30-9.40

## One-line summary
Analyzes intensive longitudinal data (many closely-spaced time points per subject, e.g. EMA/diary data) as a multivariate-across-time two-level or cross-classified model — using `LAGGED=` to create lagged variables and `TYPE = TWOLEVEL RANDOM` or `TYPE = CROSSCLASSIFIED RANDOM` to let AR slopes, cross-lagged slopes, trends, factor loadings, and residual variances vary randomly across subjects and/or across time.

## Prerequisite checklist
- [ ] Data are intensive longitudinal (many time points per subject, e.g. 20-200), long-format, one row per subject-time observation
- [ ] A subject ID variable always exists; decide whether time is also treated as its own crossed cluster (cross-classified, Ex 9.38-9.40) or only implicitly ordered within subject (two-level, Ex 9.30-9.37)
- [ ] Confirm this is genuinely multilevel time series (many subjects), not single-subject (N=1) time series — the N=1 case is covered in Chapter 6, not here
- [ ] Decide the autoregressive order needed: AR(1), AR(2), or a moving-average/ARMA(1,1) representation
- [ ] Decide whether you need: a random intercept only, a random AR slope, a random covariate slope, a random linear trend, a random residual variance, and/or random factor loadings (IRT/CFA case)
- [ ] For a multivariate process, decide whether you need a bivariate cross-lagged model (two observed variables, or two factors) instead of a univariate AR model
- [ ] Willing to use `ESTIMATOR = BAYES` — every worked example in this set uses Bayesian estimation
- [ ] If data are irregularly spaced in time with unflagged missing occasions, you will need `TINTERVAL=` and a sorted time variable
- [ ] For the cross-classified variant, confirm each subject is observed only once per time point (no nesting of subject within time or vice versa)

## Option selection logic
| Situation | Choice |
|---|---|
| Univariate continuous DV, AR(1) process, random intercept + random AR(1) slope + random residual variance across subjects | `VARIABLE: LAGGED = y(1);` `ANALYSIS: TYPE = TWOLEVEL RANDOM; ESTIMATOR = BAYES;` `%WITHIN% s \| y ON y&1; logv \| y;` (Example 9.30) |
| Same, but AR(2) needed instead of AR(1) | `LAGGED = y(2);` and add a second `\|` statement, `s2 \| y ON y&2;` (Example 9.30 variant) |
| Want only the lag-2 effect, not lag-1 | fix the lag-1 coefficient: `y ON y&1@0;` then `s2 \| y ON y&2;` (Example 9.30 variant) |
| AR(1) process plus a time-varying covariate `x` with its own random slope | add `sx \| y ON x;` on `%WITHIN%` alongside `s \| y ON y&1;` (Example 9.31) |
| Two continuous DVs that predict each other over time (cross-lagged panel style, but with many time points) | `LAGGED = y1(1) y2(1);` with four `\|` statements on `%WITHIN%`: `s1\|y1 ON y1&1; s2\|y2 ON y2&1; s12\|y1 ON y2&1; s21\|y2 ON y1&1;` (Example 9.32) |
| A single observed indicator with measurement error, want to separate true AR(1) signal from noise | latent-factor parameterization: `f BY y@1(&1); s \| f ON f&1;` (Example 9.33), or the algebraically equivalent ARMA(1,1)/moving-average parameterization: `s \| y ON y&1; e BY y@1(&1); y@.01; y ON e&1;` |
| Multiple-indicator (CFA) construct that is itself autoregressive, with random intercepts, random AR(1) slope, and random residual variance for the factor | `f BY y1-y4(&1); s \| f ON f&1; logv \| f;` on `%WITHIN%`; `fb BY y1-y4*; fb@1;` on `%BETWEEN%` (Example 9.34) |
| Same idea but with binary/ordinal indicators (an autoregressive IRT model) with random thresholds | `CATEGORICAL = u1-u4;` then `f BY u1-u4*(&1 1-4);` on `%WITHIN%` with matching loading labels `(1-4)` on `%BETWEEN%` `fb BY u1-u4*(1-4);` to equate loadings across levels (Example 9.35) |
| Two multiple-indicator factors that cross-lag each other over time | mirror the bivariate cross-lagged pattern at the factor level: `f1 BY y11-y14(&1); f2 BY y21-y24(&1);` then four `\|` statements for `s11,s22,s12,s21` (Example 9.36) |
| AR(1) process plus a linear time trend, a covariate, and a random residual variance, all with random slopes | add a third `\|` statement using an individual-level `time` variable: `s \| y ON time;` alongside `sy \| y ON y&1;` and `sx \| y ON x;` (Example 9.37) |
| Subjects and time points should be treated as **crossed** (not nested) random factors | `CLUSTER = subject time;` (subject listed first, data sorted by time within subject) and `ANALYSIS: TYPE = CROSSCLASSIFIED RANDOM;`; model has `%WITHIN%`, `%BETWEEN subject%`, and `%BETWEEN time%` blocks (Example 9.38) |
| Cross-classified AR(1) model, want to add a linear trend whose random effect varies across time as well as subject | `DEFINE` two copies of the time cluster variable (e.g. `timew`, `timet`) so the same clock can be used as an individual-level predictor and as a `%BETWEEN time%` predictor; use `s@0;` in `%BETWEEN time%` to pin a trend's variance to zero at that level if it should only vary by subject (Example 9.39) |
| Cross-classified multiple-indicator (CFA) construct that is autoregressive AND varies across both subjects and time simultaneously | `%WITHIN% f BY y1-y3*(&1 1-3); f@1; f ON f&1;` then mirror with `%BETWEEN subject% fsubj BY y1-y3*(1-3);` and `%BETWEEN time% ftime BY y1-y3*(1-3);`, using the same `(1-3)` labels to equate loadings across all three levels (Example 9.40); a second variant lets the factor loadings themselves be random via `s1-s3 \| f BY y1-y3(&1);` on `%WITHIN%`, with only `f;` (no `BY`) needed in each `%BETWEEN%` block |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE = ...;**
3. **VARIABLE:**
   - `NAMES ARE ...;`
   - `WITHIN = x;` (or `WITHIN = x timew;` etc.) — individual(-time)-level-only predictors; for cross-classified models, a `WITHIN` variable left unlabeled is modeled only at the within level (no variance at either between level), while one **not** mentioned on `WITHIN` is modeled at all levels
   - `BETWEEN = w xm;` (two-level) or `BETWEEN = (subject) w xm (time) timet;` (cross-classified) — for `TYPE=CROSSCLASSIFIED`, every `BETWEEN` variable must carry a `(subject)` or `(time)` label identifying which crossed level it belongs to
   - `CLUSTER = subject;` (two-level) or `CLUSTER = subject time;` (cross-classified — subject must be listed before time, and the data must be sorted by time within each subject)
   - `LAGGED = y(1);` — creates lagged version(s) of `y` up to the given order; use `y1(1) y2(1);` for a bivariate model, `y(2);` for AR(2)
   - `TINTERVAL = time(1);` — only needed when observations are unevenly spaced in time and missed occasions are not flagged as missing; requires the data sorted by the named time-interval variable
   - `CATEGORICAL = u1-u4;` — only if the repeated-measures indicators are binary/ordinal (IRT case)
   - `USEVARIABLES = ...;` — needed if `DEFINE`-created duplicate time variables (e.g. `timew`, `timet`) must be appended to the analysis variable list
4. **DEFINE:** e.g. `CENTER x (GROUPMEAN);` for a time-varying covariate; `timew = time; timet = time;` to create separate copies of a crossed time-cluster variable for use as a within-level predictor versus a `%BETWEEN time%` predictor
5. **ANALYSIS:**
   - `TYPE = TWOLEVEL RANDOM;` or `TYPE = CROSSCLASSIFIED RANDOM;`
   - `ESTIMATOR = BAYES;`
   - `PROCESSORS = 2;`
   - `BITERATIONS = (min);` — larger minimums (5,000-10,000) are used for the more demanding factor/IRT/cross-classified models versus simpler AR regressions (2,000)
6. **MODEL:**
   - `%WITHIN%` — lag structure and any within-level factor definition, e.g. `s | y ON y&1;`, `logv | y;`, `sx | y ON x;`, `f BY y1-y4(&1); f ON f&1;`
   - `%BETWEEN%` (two-level) or `%BETWEEN subject%` / `%BETWEEN time%` (cross-classified) — regress each named random effect (`y`, `s`, `sx`, `logv`, the between factor, etc.) on cluster covariates, and free correlations among them with `WITH`
7. **OUTPUT:** `TECH1 TECH8;`; add `FSCOMPARISON;` to compare between-level estimated factor scores (Example 9.30), or `STANDARDIZED (CLUSTER);` to get per-cluster standardized estimates for a model with random effects (Example 9.32)
8. **PLOT:** `TYPE = PLOT3;`; add `FACTORS = ALL;` to make estimated factor scores for all between-level random effects available for plotting

## Sub-option details
- `LAGGED = y(1);`: tells Mplus to build `y&1` (y at the previous time point) automatically for use in `MODEL`; the ampersand-plus-lag-number notation (`y&1`, `y&2`) refers to those lagged values inside `MODEL`
- `s | y ON y&1;`: the `|` symbol under `TYPE = ...RANDOM` names a random AR(1) slope `s` from the within-level regression of `y` on its own lag; the named slope then appears on `%BETWEEN%` exactly like a random intercept
- `logv | y;` (or `logv | f;`): names the log of a random residual variance, exactly as in the random-residual-variance models — see `two-level-random-residual-variance.md` for the full mechanics of this option
- Fixing a lag to zero, e.g. `y ON y&1@0;`, removes that lag from the model while keeping a higher-order lag (e.g. AR(2) only)
- Bivariate/cross-lagged slopes (`s12 | y1 ON y2&1;`, `s21 | y2 ON y1&1;`): each cross-lagged path gets its own name via `|` and its own random-effect distribution on `%BETWEEN%`
- Latent AR(1)-with-measurement-error factor, `f BY y@1(&1);`: fixes the single loading to 1 and permits the factor `f` to be referenced at its own lag (`f&1`) so that `f ON f&1;` expresses the true (error-free) autoregressive process, separately from the residual/measurement-error variance of `y`
- ARMA(1,1) alternative parameterization: `e BY y@1(&1); y@.01; y ON e&1;` fixes `y`'s own residual variance near zero (`.01`, not exactly 0, for estimation stability) and uses a lagged residual factor `e&1` as the moving-average component — algebraically equivalent to the measurement-error factor version but expressed without a separate latent AR process
- Matching numbers in parentheses after `BY`, e.g. `f BY y1-y4*(&1 1-4);` on `%WITHIN%` paired with `fb BY y1-y4*(1-4);` on `%BETWEEN%`: constrains the within-level and between-level factor loadings to be equal — required whenever the metric of the factor is set via a fixed factor variance (asterisk overriding the default fixed-first-loading) rather than the default loading-fixed-to-one metric
- `STANDARDIZED (CLUSTER)`: for models with random effects, each parameter's standardized value is itself standardized separately per cluster; the default reported value is the average across clusters, and the `(CLUSTER)` option additionally prints the per-cluster standardized values
- `FSCOMPARISON`: requests a comparison of the between-level estimated factor scores (i.e., the estimated random effects) across clusters
- Cross-classified `WITHIN=`/`BETWEEN=` semantics differ from plain two-level: on `WITHIN=`, an **unlabeled** variable is within-only (no variance at either between level); one **omitted** from `WITHIN=` entirely is modeled at all three levels (within, between subject, between time). On `BETWEEN=`, every variable needs an explicit `(subject)` or `(time)` label, and it is modeled only in the matching `%BETWEEN subject%` or `%BETWEEN time%` block
- `CLUSTER = subject time;`: for `TYPE = CROSSCLASSIFIED`, the two cluster variables are crossed, not nested (each subject is observed once per time point); the subject variable must be listed first, and the data must be sorted by time within subject
- A within-level random effect (e.g. an AR slope) can be given a distribution at only one of the two crossed levels by naming it in only one `%BETWEEN ...%` block, or at both by naming it in both — this lets you decide, per random effect, whether it varies across subjects only, across time only, or across both (Example 9.39 contrasts a slope that varies across both subject and time with a random residual variance that varies across subject only)
- `s@0;` inside a `%BETWEEN time%` (or `%BETWEEN subject%`) block: fixes a random effect's variance at that level to zero, i.e. tells Mplus the effect is not free to vary across that particular crossed dimension, even though it remains free (or random) at the other

## Post-run operations
- Check `TECH8`'s MCMC history and the PSR criterion against `BITERATIONS` to confirm convergence; the more complex factor/IRT/cross-classified models in this set typically need a larger minimum iteration count and closer scrutiny of convergence than a plain two-level regression
- With a `logv`-type random residual variance in the model, remember the reported value is on the log scale — see `two-level-random-residual-variance.md` for interpretation
- For a cross-lagged model, compare the two cross-lagged slopes (e.g. `s12` vs `s21`) to judge asymmetry in how the two processes drive each other over time, and check their `WITH` correlations with the AR slopes and intercepts on `%BETWEEN%`
- For the measurement-error/ARMA parameterizations (Ex 9.33), confirm whether the AR structural process and the measurement-error variance are separately identified and substantively distinguishable — this is the point of using a latent `f` rather than modeling `y` directly
- Use `PLOT: TYPE = PLOT3; FACTORS = ALL;` to inspect estimated factor-score plots for each random effect (random intercepts, AR slopes, trends, residual variances) and judge cluster-level (and, for cross-classified models, time-level) heterogeneity
- For a cross-classified model, separately examine the `%BETWEEN subject%` and `%BETWEEN time%` variance estimates for each random effect to see whether an effect genuinely differs more across people or more across occasions
- If the model is a plain two-level regression/CFA without any autoregressive or cross-classified structure, this file does not apply — see `two-level-regression-path-basics.md` or `two-level-cfa-sem.md` instead; for single-subject (N=1) time series, see Chapter 6 of the User's Guide (not yet covered in this skill)

## Likely FAQ mapping
- "I have ecological momentary assessment / diary data with many time points per person and want an AR(1) model" → Example 9.30 pattern (`LAGGED=`, `s | y ON y&1;`)
- "I want the autoregressive effect (and/or residual variance) to differ from person to person" → the `|` random-slope / `logv |` mechanics in Examples 9.30-9.31
- "I have a time-varying covariate in my intensive longitudinal model" → Example 9.31 pattern
- "Two variables seem to predict each other over time (a cross-lagged panel with many waves)" → Example 9.32 pattern
- "My repeated measure has measurement error and I want to separate it from the true autoregressive signal" → Example 9.33 (`f BY y@1(&1);`) or its ARMA(1,1) alternative
- "I have multiple indicators per time point and want an autoregressive factor model" → Example 9.34 (continuous) or 9.35 (binary/ordinal IRT)
- "I want two autoregressive factors that cross-lag each other" → Example 9.36
- "I want a linear time trend on top of the AR process" → Example 9.37
- "My data have both subjects and time points as crossed grouping factors, not nested" → Examples 9.38-9.40 (`TYPE = CROSSCLASSIFIED RANDOM;`, `CLUSTER = subject time;`)
- "Should a random effect vary across subjects, across time, or both?" → name it in the corresponding `%BETWEEN subject%` and/or `%BETWEEN time%` block(s), or fix it with `@0` to suppress variation at one level (Example 9.39)
- "I want a multiple-indicator factor that's autoregressive and varies across both people and time" → Example 9.40
- "This is a single-subject (N=1) time series, not multilevel" → not covered here; see Chapter 6 of the User's Guide

# Two-Level Growth Models for Longitudinal Data Nested in Clusters

> Source: Mplus User's Guide v8, Chapter 9, Examples 9.12, 9.13, 9.14, 9.15, 9.16, 9.17, 9.18, 9.19
> Bundled source: references/source-pdfs/Chapter9.pdf

## One-line summary
Combines Chapter 6-style growth curve modeling (repeated measures collapsed into intercept/slope growth factors via the `|` symbol) with the Chapter 9 `%WITHIN%`/`%BETWEEN%` framework, so that individually-varying growth trajectories (continuous, categorical, count, or multiple-indicator) can be estimated for people nested within clusters — plus two closely related two-level longitudinal extensions: continuous-time survival with a random intercept, and a MIMIC model with random factor loadings.

## Prerequisite checklist
- [ ] **Confirm the data has three levels conceptually** (time within person within cluster) but only **two** `CLUSTER=` variables are ever needed in Mplus, because Mplus treats repeated measures multivariately (as separate variables `y1 y2 y3 y4`, one column per occasion) rather than as a stacked "level 1" file — this collapses the time dimension into the within part of a `TYPE=TWOLEVEL` model instead of requiring `TYPE=THREELEVEL`
- [ ] Is the outcome continuous, binary/ordinal, or a count (possibly zero-inflated)?
- [ ] Is growth measured directly on one observed variable per occasion, or via a **factor with multiple indicators** at each occasion (multiple-indicator growth)?
- [ ] Are the time scores fixed/equidistant (e.g. `y1@0 y2@1 y3@2 y4@3`), or does the model need individually-varying/time-varying covariates?
- [ ] If there's a time-varying covariate, should its effect (slope) be fixed, random only across clusters (between-level variation), or random at **both** the within (occasion-to-occasion) and between (cluster) levels?
- [ ] Is the raw data file in **wide** format (one row per person, one column per occasion) and does it need reshaping to **long** format for a univariate two-level growth setup (`DATA WIDETOLONG`)?
- [ ] Is this actually a **time-to-event** outcome (survival) rather than a growth trajectory? See Example 9.18.
- [ ] Is this actually a **MIMIC model with a random factor loading** (a two-level factor model with a covariate, not a repeated-measures growth curve)? See Example 9.19 — for other two-level CFA/SEM variants without a growth component, see `two-level-cfa-sem.md`.

## Option selection logic
| Situation | Choice |
|---|---|
| Continuous outcome, linear growth, individual-level covariate `x` predicts within growth factors, cluster-level covariate `w` predicts between growth factors | `TYPE = TWOLEVEL;` with `iw sw \| y1@0 y2@1 y3@2 y4@3;` on `%WITHIN%` and `ib sb \| y1@0 y2@1 y3@2 y4@3;` on `%BETWEEN%` (Example 9.12) |
| Same, but outcome is binary/ordinal | add `CATEGORICAL = u1-u4;`; default ML uses numerical integration — use `ANALYSIS: INTEGRATION = n;` to reduce integration points per dimension if the default (15) is too slow (Example 9.13) |
| A time-varying covariate's slope should vary across **both** occasions-within-cluster and clusters | name the slope with a trailing asterisk, e.g. `s* \| y1 ON a1; s* \| y2 ON a2; ...;`, under `TYPE = TWOLEVEL RANDOM;` + `ALGORITHM = INTEGRATION;` (Example 9.14) — omitting the asterisk restricts that slope to between-level variation only |
| Growth factor is measured by **multiple categorical indicators** at each occasion (multiple-indicator growth) | define one within factor and one between factor per occasion via `BY`, then run the growth `\|` statement on the factors themselves (`iw sw \| f1w@0 f2w@1 f3w@2;`); use `ESTIMATOR = WLSM;` to avoid heavy numerical integration, and save sample statistics with `SAVEDATA: SWMATRIX=...;` (Example 9.15) |
| Raw data is in wide (multivariate) format and needs conversion to long (univariate) format before a two-level growth analysis | `DATA WIDETOLONG: WIDE = ...; LONG = ...; IDVARIABLE = ...; REPETITION = ...;` ahead of `VARIABLE: USEVARIABLES=...; CLUSTER = <id>;` (Example 9.16) |
| Count outcome, growth model needed for excess zeros | `COUNT = u1-u4 (i);` for zero-inflated Poisson; growth `\|` statements needed for both the count part and, using `#1` after the variable name, the inflation part (e.g. `u1#1@0 u2#1@1 ...`) (Example 9.17) |
| Outcome is time-to-event, not a repeated growth trajectory, with clustering | `SURVIVAL = t (ALL); TIMECENSORED = tc (0 = NOT 1 = RIGHT);` + `ANALYSIS: TYPE = TWOLEVEL; BASEHAZARD = OFF;` (non-parametric baseline hazard, the default) — this is Cox regression with a random intercept, not a growth model (Example 9.18) |
| Two-level MIMIC model where the factor loading itself is a random effect that can be constrained equal across levels | `TYPE = TWOLEVEL RANDOM;` with `s1-s4 \| f BY y1-y4;` on `%WITHIN%`, `f ON x1 x2;`; on `%BETWEEN%`, either build a separate factor from the random intercepts (`fb BY y1-y4;`) or label the between loadings with the same names as the within random-slope means (`fb BY y1-y4* (lam1-lam4); [s1-s4] (lam1-lam4);`) to force equality; `ESTIMATOR = BAYES;` is used for the Bayesian version (Example 9.19) |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;** — optionally followed by **DATA WIDETOLONG:** (only when reshaping is needed, Example 9.16):
   - `WIDE = y11-y14 | a31-a34;` — sets of wide-format variable groups to convert
   - `LONG = y | a3;` — new long-format variable names, one per WIDE group
   - `IDVARIABLE = person;` — name for the new unit/person identifier (default `id`)
   - `REPETITION = time;` — name for the new occasion-order variable (default `rep`)
3. **VARIABLE:**
   - `NAMES ARE ...;`
   - `CATEGORICAL = u1-u4;` — binary/ordinal outcome occasions
   - `COUNT = u1-u4 (i);` — count outcome; `(i)` requests zero-inflated Poisson for the growth model
   - `SURVIVAL = t (ALL);` / `TIMECENSORED = tc (0 = NOT 1 = RIGHT);` — survival-analysis variant only
   - `WITHIN = x;` — individual-level-only covariates (and, after `DATA WIDETOLONG`, the reshaped time-varying covariate/time score)
   - `BETWEEN = w;` — cluster-level-only covariates
   - `CLUSTER = clus;` (or the `IDVARIABLE` name from `DATA WIDETOLONG`, e.g. `person`)
   - `USEVARIABLES = ...;` — required after `DATA WIDETOLONG` to list original plus newly created variables
4. **ANALYSIS:**
   - `TYPE = TWOLEVEL;` — sufficient whenever growth/intercept factors are defined with the compact `\| y1@0 y2@1 ...;` batch notation, **even with random slopes on those growth factors** — the `RANDOM` keyword is not required for this notation
   - `TYPE = TWOLEVEL RANDOM;` — required as soon as a random effect is instead named via ordinary `\| DV ON covariate;` regression-style syntax (e.g. Examples 9.14, 9.16, 9.19)
   - `ALGORITHM = INTEGRATION;` with `INTEGRATION = n;` — numerical-integration ML; reduce `n` from the default 15 points/dimension when many growth factors make computation slow (e.g. `INTEGRATION=7` for a 4-occasion categorical growth model, `INTEGRATION=10` for a 4-dimension random-slope model)
   - `MCONVERGENCE = 0.01;` — loosens the EM algorithm's log-likelihood-derivative convergence criterion from the default .001 when high numerical precision is hard to achieve (e.g. zero-inflated Poisson growth)
   - `ESTIMATOR = WLSM;` — robust weighted least squares alternative for multiple-indicator categorical growth models
   - `ESTIMATOR = BAYES;` with `PROCESSORS = 2; BITERATIONS = (1000);` — Bayesian estimation; the number in parentheses on `BITERATIONS` sets the **minimum** MCMC iterations per chain (maximum stays at the default 50,000) until the potential scale reduction (PSR) criterion is met
   - `BASEHAZARD = OFF;` — survival-analysis variant only; specifies a non-parametric baseline hazard (the default)
5. **DEFINE:** (only if rescaling a time variable, not shown in these examples' equivalents but analogous to Example 9.27's `DEFINE: timescor = (time-1)/100;`)
6. **MODEL:**
   - `%WITHIN%` block: growth-factor definition `iw sw | y1@0 y2@1 y3@2 y3@3;` (names + fixed time scores); `y1-y4 (1);` to constrain residual variances equal over time (overridable); `iw sw ON x;` for covariate effects on the within growth factors
   - `%BETWEEN%` block: parallel growth-factor definition `ib sb | y1@0 y2@1 y3@2 y4@3;` referencing the **same outcome names**; `y1-y4@0;` to fix outcome residual variances to zero at the between level (the default, overridable); `ib sb ON w;` for cluster-level covariate effects
   - For multiple-indicator growth: within/between factor `BY` statements per occasion, then the growth `|` statement operates on the factor names instead of raw outcomes
   - For zero-inflated count growth: a second `|` statement for the inflation part, using `#1` after the count variable name (e.g. `u1#1@0 u2#1@1 ...`)
   - For survival: `%WITHIN% t ON x;` / `%BETWEEN% t ON w; t;` (freeing the between-level residual variance of the time-to-event random intercept)
7. **SAVEDATA:** `SWMATRIX = ex9.15sw.dat;` — only with `ESTIMATOR=WLSM`/`WLSMV`, to save within/between sample statistics for reuse
8. **PLOT:** `TYPE = PLOT2;` — used in the Bayesian MIMIC example to enable posterior distribution/trace/autocorrelation plots
9. **OUTPUT:** `TECH1 TECH8;` — recommended whenever numerical integration or Bayesian estimation is used

## Sub-option details
- **Growth-factor `|` batch notation vs. random-slope `|` regression notation**: `iw sw | y1@0 y2@1 y3@2 y4@3;` names growth factors directly from fixed time scores and does **not** require `TYPE=...RANDOM`; by contrast `s | y ON x;` (or `s* | y1 ON a1;`) names an arbitrary random slope from an ordinary regression and **does** require `TYPE=...RANDOM`. Confirm which notation a target model needs before deciding on the `ANALYSIS` line.
- `y1-y4 (1);` on `%WITHIN%`: the parenthesized label constrains the listed residual variances to be equal across time, matching conventional multilevel growth modeling; this is a convention, not a requirement, and can be dropped to let residual variances differ by occasion.
- `y1-y4@0;` on `%BETWEEN%`: fixes the outcome (random-intercept) residual variances to zero at the between level by default, since the growth factors themselves already absorb the between-cluster variance; can be freed.
- Growth factor residuals (`iw`/`sw`, `ib`/`sb`) are **correlated by default** at both levels, following the general Mplus convention that latent variables not predicting anything besides their own indicators have correlated residuals; use `WITH` to free/fix as needed.
- Growth-factor intercepts: the outcome variable intercepts at each occasion are fixed at zero by default (identification of the growth parameterization); the growth factor means/intercepts are freely estimated in the between part by default.
- `s* |` (trailing asterisk on a random-slope name): allows that random effect to have variance at **both** the within and between levels; without the asterisk, the same slope name only varies across clusters (between-level only). This distinction only applies to the `| DV ON covariate;` regression-style random-slope notation.
- `COUNT = u (i);`: `(i)` requests a **zero-inflated Poisson** model; the inflation (structural-zero) part of the growth model is addressed with the same outcome name plus `#1` (e.g. `u1#1@0`), and its own intercept/slope growth factors can be named separately (e.g. `iiw siw |`).
- `MCONVERGENCE`: an `ANALYSIS`-level EM convergence tolerance, useful specifically when a model (such as zero-inflated Poisson growth) is numerically difficult to converge to the default high precision.
- `SURVIVAL = t (ALL);`: `t` is the time-to-event variable; `(ALL)` takes the baseline-hazard time intervals directly from the observed data. Must be paired with `TIMECENSORED =` to flag right-censored cases (`0 = NOT 1 = RIGHT` is the default coding).
- `BASEHAZARD = OFF;`: chooses a non-parametric baseline hazard function for continuous-time Cox regression (the default); this is the only baseheight setting demonstrated in these examples.
- Two-level MIMIC random-loading notation (`s1-s4 | f BY y1-y4;`): names a **vector** of random loadings, one per indicator, in one statement; `f@1;` fixes the factor variance to set the metric (since the loadings, not the factor variance, are now free/random); on `%BETWEEN%`, either build a genuinely separate between factor from the random intercepts (`fb BY y1-y4;`) or force the between loadings equal to the within random-loading means using shared parameter labels (`fb BY y1-y4* (lam1-lam4); [s1-s4] (lam1-lam4);`).
- Default estimator is maximum likelihood with robust standard errors unless numerical-integration cost, categorical multiple-indicator complexity, or an explicit Bayesian choice dictates `WLSM`/`WLSMV`/`BAYES`.

## Post-run operations
- Interpret within-level growth factors (`iw`, `sw`) as the individual-level (occasion-to-occasion) trajectory and between-level growth factors (`ib`, `sb`) as how the average trajectory (and its individual variability) differs across clusters — these are two separate growth curves, not one.
- For a time-varying covariate slope defined with a trailing asterisk, check whether its within-level variance and between-level variance are both meaningfully nonzero before concluding the effect genuinely varies at both levels.
- For multiple-indicator growth models, first confirm the within- and between-level measurement models (factor loadings/thresholds) are adequate before interpreting the growth parameters built on top of the factors.
- For zero-inflated Poisson growth, interpret the count-part growth factors (`iw`, `sw`) and the inflation-part growth factors (`iiw`, `siw`) separately — one governs trajectory among those who can have nonzero counts, the other governs the probability of being a structural zero.
- For the survival variant, treat the within-level `t ON x;` coefficient as a (log-)hazard-ratio-scale effect and the between-level residual variance of `t` as evidence of cluster-level frailty.
- `OUTPUT: TECH1;` confirms parameter specification/starting values; `TECH8;` monitors optimization (or MCMC) progress — check both whenever `ALGORITHM=INTEGRATION` or `ESTIMATOR=BAYES` is used.
- If weighted least squares was used, keep the `SAVEDATA: SWMATRIX=...;` output file for faster re-estimation in follow-up runs on the same data.
- If sampling weights or a stratified/clustered survey design are also involved (in addition to genuine multilevel growth), see `complex-survey-design-multilevel.md`.

## Likely FAQ mapping
- "I have repeated measures nested within schools/clinics/etc. and want a growth curve" → Example 9.12 pattern (`TYPE=TWOLEVEL`, growth `|` on `%WITHIN%`/`%BETWEEN%`)
- "My repeated outcome is binary/ordinal, not continuous, and I want a multilevel growth model" → Example 9.13 pattern (`CATEGORICAL=`, `INTEGRATION=`)
- "I have a time-varying covariate and I think its effect on my outcome differs both occasion-to-occasion and cluster-to-cluster" → Example 9.14 pattern (`s* |`)
- "My growth factor is measured by several items at each wave, and the items are categorical" → Example 9.15 pattern (multiple-indicator growth, `ESTIMATOR=WLSM`)
- "My raw data is one row per person with separate columns per wave/covariate — how do I get it into shape for a multilevel growth model?" → Example 9.16 pattern (`DATA WIDETOLONG`)
- "My repeated outcome is a count with lots of zeros" → Example 9.17 pattern (`COUNT = ... (i);`, zero-inflated growth)
- "I actually have a time-to-event outcome with clustering, not a repeated-measures trajectory" → Example 9.18 pattern (`SURVIVAL=`, `TIMECENSORED=`) — this is Cox regression with a random intercept, not growth modeling
- "I want a two-level factor model (MIMIC) where the factor loadings themselves vary across clusters" → Example 9.19 pattern (`s1-s4 | f BY ...;`, `TYPE=TWOLEVEL RANDOM`) — for two-level CFA/SEM without a growth/longitudinal angle, see `two-level-cfa-sem.md` instead
- "Do I need TYPE=TWOLEVEL RANDOM for my growth model?" → only if a random slope is named with ordinary `| DV ON covariate;` syntax; the compact `| y1@0 y2@1 ...;` growth-factor notation does not require it
- "I also have survey weights or a stratified design on top of this longitudinal multilevel structure" → point to `complex-survey-design-multilevel.md`

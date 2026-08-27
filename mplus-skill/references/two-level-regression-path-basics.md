# Two-Level Regression and Path Analysis (Random Intercepts and Slopes)

> Source: Mplus User's Guide v8, Chapter 9, Examples 9.1, 9.2, 9.3, 9.4, 9.5
> Bundled source: references/source-pdfs/Chapter9.pdf

## One-line summary
Models individuals clustered within a higher-level unit (school, clinic, etc.) by splitting the model into a `%WITHIN%` (individual-level) part and a `%BETWEEN%` (cluster-level) part, with observed regression/path relationships among variables — no latent factors involved.

## Prerequisite checklist
- [ ] **Confirm a cluster ID variable exists in the data** (e.g. school/class ID) — required for `CLUSTER=`
- [ ] **Classify every variable as within-only, between-only, or both**: a variable listed on `WITHIN=` is measured only at the individual level (no between variance); a variable on `BETWEEN=` is measured only at the cluster level (constant within a cluster); a variable named in neither is measured at the individual level but can have both a within and a between part (e.g. a dependent variable with a random intercept)
- [ ] Is only the intercept assumed to vary randomly across clusters, or does a slope (the effect of a within-level predictor) also vary across clusters?
- [ ] If a slope is random, will centering be grand-mean or group(cluster)-mean? (group-mean centering is recommended when a random slope is estimated)
- [ ] Do you want an individual-level covariate treated as a plain observed covariate (using an observed cluster mean, e.g. `xm`), or decomposed into latent within/between parts (by leaving it off `WITHIN=`)?
- [ ] Are the dependent variable(s) continuous, categorical (binary/ordinal), or a mix — this affects the default estimator and whether `ALGORITHM=INTEGRATION` or `ESTIMATOR=WLSM` is needed
- [ ] Is there also a cluster-level (between-only) dependent/mediating variable in the path diagram, in addition to the individual-level ones?

## Option selection logic
| Situation | Choice |
|---|---|
| Continuous DV, only a random intercept, covariate treated as observed | `TYPE = TWOLEVEL;` with `%WITHIN% y ON x;` / `%BETWEEN% y ON w xm;` (Example 9.1) |
| Also want the slope of `x` on `y` to vary across clusters | `TYPE = TWOLEVEL RANDOM;` with `s \| y ON x;` on `%WITHIN%`, then `y s ON w xm;` on `%BETWEEN%` (Example 9.2) |
| Covariate `x` should be split into latent within + between parts instead of using an observed cluster mean | omit `x` from the `WITHIN=` statement — Mplus decomposes it automatically into an uncorrelated within-level latent covariate and a between-level latent covariate (Examples 9.1 second part, 9.2 third part) |
| Path model where a mediator `y` (continuous) feeds into a binary/ordinal DV `u` | `CATEGORICAL = u;` + `ANALYSIS: TYPE = TWOLEVEL; ALGORITHM = INTEGRATION;` (Example 9.3) |
| Path model that also has an observed cluster-level (between-only) mediator `z`, estimated with weighted least squares instead of ML | `WITHIN=`/`BETWEEN=` as usual + `ANALYSIS: ESTIMATOR = WLSM;` (Example 9.4) |
| Path model with several within-level equations, each having its own random slope | `TYPE = TWOLEVEL RANDOM;` with multiple `sK \| yK ON ...;` statements on `%WITHIN%`, all named random effects regressed on cluster covariates on `%BETWEEN%` (Example 9.5) |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE = ...;**
3. **VARIABLE:**
   - `NAMES ARE ...;`
   - `WITHIN = x;` — individual-level-only variables
   - `BETWEEN = w xm;` — cluster-level-only variables
   - `CLUSTER = clus;` — required cluster ID variable
   - `CATEGORICAL = u;` — only if a DV is binary/ordinal
4. **DEFINE:** `CENTER x (GRANDMEAN);` or `CENTER x (GROUPMEAN);` — optional but recommended, especially group-mean centering when a random slope is used
5. **ANALYSIS:**
   - `TYPE = TWOLEVEL;` — random intercept(s) only
   - `TYPE = TWOLEVEL RANDOM;` — needed as soon as any `|` random-slope statement is used in MODEL
   - `ALGORITHM = INTEGRATION;` — needed for maximum likelihood with a categorical outcome in the within part
   - `ESTIMATOR = WLSM;` — alternative to ML for continuous/categorical mixes (robust weighted least squares, diagonal weight matrix)
6. **MODEL:**
   - `%WITHIN%` block: observed-variable regressions among individual-level variables, e.g. `y ON x;`; random slopes via `s | y ON x;`
   - `%BETWEEN%` block: regressions of the random intercepts/slopes (referred to by the same DV name, e.g. `y`, and slope name, e.g. `s`) on cluster-level covariates, e.g. `y ON w xm;` or `y s ON w;`; use `y WITH s;` to free the intercept–slope residual covariance
7. **MODEL CONSTRAINT:** (optional) `NEW(betac); betac = gamma01 - gamma10;` — to compute a "contextual effect" (difference between the between- and within-level slope of the same covariate)
8. **OUTPUT:** `TECH1 TECH8;` (recommended when `ALGORITHM=INTEGRATION` is used, to check starting values and monitor optimization)

## Sub-option details
- `WITHIN=` / `BETWEEN=`: any variable not mentioned on either statement is treated as measured at the individual level but modeled on **both** the within and between parts (this is exactly how a dependent variable becomes a random intercept — it is automatically named the same on `%BETWEEN%`)
- Leaving a covariate off `WITHIN=` decomposes it into two uncorrelated latent variables, `x = x_within + x_between`; this differs from conventional multilevel software, which typically uses only the observed cluster mean (`xm`) as the between-level covariate — a latent decomposition can be preferable when the observed cluster mean has low reliability
- `s | y ON x;`: the `|` syntax names a random slope (`s`) via a within-level regression; requires `ANALYSIS: TYPE = TWOLEVEL RANDOM;`; the named random effect (`s`) is then referenced on `%BETWEEN%` exactly like a random intercept
- `CENTER (GROUPMEAN)`: recommended for the individual-level predictor when its slope is random (Raudenbush & Bryk, 2002, p. 143)
- `y WITH s;`: on `%BETWEEN%`, random intercepts and random slopes are **not** correlated with each other by default — add explicit `WITH` statements to free these residual covariances
- `MODEL CONSTRAINT: NEW(betac); betac = gamma01 - gamma10;`: computes the "contextual effect" — the difference between the between-level slope (e.g. `gamma01` from `y ON w`/`xm`) and the within-level slope (e.g. `gamma10` from `y ON x`) of the same variable, following Raudenbush & Bryk (2002, Table 5.11)
- `ALGORITHM = INTEGRATION;`: required maximum-likelihood numerical-integration algorithm when a categorical DV appears with random effects in the within part; integration cost grows quickly with the number of dimensions (random effects/factors), so watch computation time on larger models
- `ESTIMATOR = WLSM;`: a robust weighted least squares estimator using a diagonal weight matrix (Asparouhov & Muthén, 2007), an alternative to ML for models mixing continuous and categorical/cluster-level dependent variables
- Default estimator overall is maximum likelihood with robust standard errors; `ANALYSIS: ESTIMATOR = ...;` selects an alternative
- `PLOT: TYPE = PLOT2;` combined with `MODEL CONSTRAINT: PLOT(...); LOOP(level1, low, high, increment);` can graph a cross-level interaction (e.g. how a within-level slope changes across values of a between-level moderator) — see Example 9.2's third part

## Post-run operations
- Interpret `%WITHIN%` and `%BETWEEN%` coefficients as separate regression equations (level 1 vs. level 2 of a conventional multilevel model) — do not conflate them
- If a random slope was estimated, check its variance on `%BETWEEN%`: a significant variance means the slope genuinely differs across clusters, and the `s ON w` coefficient (if specified) tests whether a cluster-level covariate explains that variation
- If a contextual effect was requested via `MODEL CONSTRAINT`, check whether `betac` differs meaningfully (and significantly) from zero — a nonzero contextual effect means the within- and between-level slopes of the same variable genuinely differ
- Use `OUTPUT: TECH1;` to confirm parameter specification/starting values, and `TECH8;` to monitor the optimization history, especially whenever `ALGORITHM=INTEGRATION` is used
- If cluster-level sampling weights or a stratified/clustered survey design are also involved (as opposed to a genuinely multilevel model), see `complex-survey-design-multilevel.md`

## Likely FAQ mapping
- "My data has students nested in schools and I want a regression with a random intercept" → Example 9.1 pattern (`TYPE=TWOLEVEL`)
- "I think the slope of my predictor differs by cluster" → Example 9.2 pattern (`TYPE=TWOLEVEL RANDOM`, `s | y ON x;`)
- "Should I use the cluster mean of my covariate, or something more sophisticated?" → explain the observed cluster-mean (`xm`) vs. latent within/between decomposition options
- "One of my mediators is categorical/binary" → Example 9.3 pattern (`CATEGORICAL=`, `ALGORITHM=INTEGRATION`)
- "I have a mediator that's only measured at the cluster level" → Example 9.4 pattern (`BETWEEN=` mediator, `ESTIMATOR=WLSM`)
- "I have several within-level equations, each possibly with its own random slope" → Example 9.5 pattern
- "I want to know if the within- and between-level effects of the same variable are actually different" → contextual effect via `MODEL CONSTRAINT`
- "I also have survey weights/stratification on top of this multilevel structure" → point to `complex-survey-design-multilevel.md`
- "I want factors/latent variables in my multilevel model" → point to `two-level-cfa-sem.md`, not this file

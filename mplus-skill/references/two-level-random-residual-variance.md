# Two-Level Random Residual Variance (Heteroscedasticity) Models

> Source: Mplus User's Guide v8, Chapter 9, Examples 9.28, 9.29

## One-line summary
Lets the within-level residual variance of a continuous dependent variable (or of a within-level factor) vary randomly across clusters — modeling heteroscedasticity — by naming the log of that residual variance as an extra between-level random effect, estimated with Bayes.

## Prerequisite checklist
- [ ] Cluster ID variable exists in the data (`CLUSTER=`)
- [ ] The outcome whose variability you want to model is continuous
- [ ] Decide whether the random residual variance belongs to an **observed dependent variable** (regression case, Ex 9.28) or to a **within-level latent factor's structural residual** (CFA case, Ex 9.29)
- [ ] Willing to use `ESTIMATOR = BAYES` — both worked examples in this part of the chapter use Bayesian estimation for the random-residual-variance parameterization
- [ ] Any cluster-level covariates (`w`, `xm`) or cluster-level distal outcomes (`z`) that should be regressed on the random intercept and/or the random residual variance
- [ ] Comfortable interpreting the random residual variance on the **log** scale (Mplus models `logv`, the log of the variance, not the variance itself, so it stays positive across MCMC draws)

## Option selection logic
| Situation | Choice |
|---|---|
| Continuous DV `y` with a random intercept, and you suspect the within-level residual variance of `y` also differs by cluster | `TYPE = TWOLEVEL RANDOM;` `ESTIMATOR = BAYES;` with `logv \| y;` on `%WITHIN%` (Example 9.28) |
| Want to know whether a cluster-level covariate predicts that residual-variance heterogeneity | `logv ON w xm;` on `%BETWEEN%` |
| Want a cluster-level distal outcome `z` to depend on both the cluster mean of `y` and on how variable `y` is within that cluster | `z ON y logv;` on `%BETWEEN%` (Example 9.28) |
| Random intercept and random residual variance should be allowed to correlate across clusters | `y WITH logv;` on `%BETWEEN%` |
| Two-level CFA where a within-level factor's own structural residual variance (not just its indicators') varies across clusters | build a within factor `fw BY y1-y4;`, then `logv \| fw;` on `%WITHIN%`; mirror with a between factor `fb BY y1-y4;` (Example 9.29) |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE = ...;**
3. **VARIABLE:**
   - `NAMES ARE ...;`
   - `WITHIN = x1 x2;` — individual-level-only predictors
   - `BETWEEN = w xm z;` — cluster-level covariates and/or a cluster-level distal outcome `z`
   - `CLUSTER = clus;`
4. **ANALYSIS:**
   - `TYPE = TWOLEVEL RANDOM;` — required as soon as any `|` random-effect statement (including a random residual variance) is used
   - `ESTIMATOR = BAYES;`
   - `PROCESSORS = 2;` — speeds up the two parallel MCMC chains when available
   - `BITERATIONS = (min);` — sets the minimum (and, optionally, maximum) number of MCMC iterations per chain used with the PSR convergence criterion; the worked examples use a larger minimum (e.g. 10,000 for the CFA case, Ex 9.29) for models expected to converge more slowly than a simple regression case (e.g. 2,000, Ex 9.28)
5. **MODEL:**
   - `%WITHIN%` — for a regression outcome: `y ON x;` then `logv | y;`; for a factor: `fw BY y1-y4; fw ON x1 x2;` then `logv | fw;`
   - `%BETWEEN%` — regress the random intercept (and, for the CFA case, the between factor `fb BY y1-y4;`) on cluster covariates, e.g. `y ON w xm;` or `fb ON w;`; regress the random residual variance the same way, e.g. `logv ON w xm;`; free correlations among random effects with `WITH`, e.g. `y WITH logv;`; optionally regress a cluster-level distal outcome on the random effects, e.g. `z ON y logv;`
6. **OUTPUT:** `TECH1 TECH8;` — parameter specification/starting values and MCMC convergence history
7. **PLOT:** `TYPE = PLOT3;` — enables trace, autocorrelation, and posterior-distribution plots for Bayesian parameters, including the random residual variance

## Sub-option details
- `logv | y;`: the `|` symbol used with `TYPE = TWOLEVEL RANDOM` to name (left-hand side) the **log** of the random residual variance of the variable on the right-hand side (`y`); must be specified on `%WITHIN%`; the named parameter (`logv`) then behaves like any other between-level random effect and can be regressed on covariates, correlated with `WITH`, or used as a predictor on `%BETWEEN%`
- `logv | fw;`: the same mechanism applied to a within-level **factor's** structural residual variance instead of an observed dependent variable's residual variance — lets the residual variance of a latent construct (after accounting for its own predictors) vary across clusters
- Filled circles in the path diagrams distinguish random effects (intercept, residual variance) that vary across clusters — shown as circles on `%BETWEEN%` — from the fixed within-level structure
- `y WITH logv;` (or `fb WITH s logv;` etc.): random intercepts/factors and random residual variances are **not** correlated with each other by default on `%BETWEEN%`; add explicit `WITH` statements to free these covariances
- `z ON y logv;`: an observed cluster-level dependent variable can be regressed on both the random intercept and the (log) random residual variance simultaneously — useful when cluster-level heterogeneity in within-cluster variability is itself a substantively meaningful predictor of a cluster outcome
- In the CFA case (Ex 9.29), the between factor `fb BY y1-y4;` uses the random intercepts of the indicators (`y1`–`y4` on `%BETWEEN%`) as its own indicators, exactly as in the baseline two-level CFA framework — see `two-level-cfa-sem.md` for that unadorned pattern
- `BITERATIONS = (min);`: increasing the minimum iteration count is recommended for these random-residual-variance models because they can be slower to converge than simpler random-intercept-only models

## Post-run operations
- Because `logv` is the **log** of the residual variance, exponentiate reported values of `logv` (or use `MODEL CONSTRAINT`) to translate back to the variance scale before reporting; a `logv ON w` coefficient is interpreted as a proportional/multiplicative effect on the residual variance, not an additive one
- Use `TECH8`'s printed MCMC history and `TECH1`'s parameter specification to confirm the model is set up as intended and that the chains are converging (check the PSR values against the requested `BITERATIONS`)
- With `PLOT: TYPE = PLOT3;`, inspect the trace plot and autocorrelation plot for `logv` (and any other random effect) to judge MCMC mixing, and the posterior-distribution plot to see the full posterior shape rather than only a point estimate and interval
- If `z ON y logv;` was specified, check whether the residual-variance coefficient is credibly different from zero — that tells you whether within-cluster variability (not just the cluster mean) predicts the distal cluster outcome
- If instead you only need a plain random intercept (no heteroscedasticity modeling), use the simpler pattern in `two-level-regression-path-basics.md` or `two-level-cfa-sem.md`

## Likely FAQ mapping
- "I think the variability of my outcome — not just its mean — differs across schools/clinics/clusters" → Example 9.28 pattern (`logv | y;`)
- "How do I let a residual variance be random across level-2 units in Mplus?" → the `logv | variable;` syntax under `TYPE = TWOLEVEL RANDOM` with `ESTIMATOR = BAYES`
- "Can a cluster-level covariate explain why some clusters are more variable than others?" → `logv ON w xm;` on `%BETWEEN%`
- "I want a two-level CFA where the factor's own residual variance (not just indicator noise) differs by cluster" → Example 9.29 pattern
- "Can a cluster-level outcome depend on how spread out my level-1 outcome is within that cluster, not just its average?" → `z ON y logv;`
- "Should the random intercept and random residual variance be allowed to correlate?" → add `y WITH logv;` on `%BETWEEN%` (not automatic)
- "I actually just want a random intercept, no heteroscedasticity" → see `two-level-regression-path-basics.md` / `two-level-cfa-sem.md` instead

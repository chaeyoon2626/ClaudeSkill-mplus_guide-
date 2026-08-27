# Two-Level CFA and SEM (Random-Intercept and Random-Slope Factor Models)

> Source: Mplus User's Guide v8, Chapter 9, Examples 9.6, 9.7, 9.8, 9.9, 9.10, 9.11
> Bundled source: references/source-pdfs/Chapter9.pdf

## One-line summary
Extends the two-level `%WITHIN%`/`%BETWEEN%` framework to latent variables: a within-level factor is measured by individual-level indicators, and its per-cluster random intercepts can themselves load onto a between-level factor, optionally with covariates, random factor loadings/slopes, structural (SEM) paths, or multiple groups.

## Prerequisite checklist
- [ ] **Confirm the within-level measurement model**: how many factors, which individual-level indicators load on each, via `BY` statements on `%WITHIN%`
- [ ] **Decide whether a genuine between-level factor is wanted**, or whether the between part should just leave the indicators' random intercepts uncombined (no `BY` on `%BETWEEN%`)
- [ ] Are the within-level factor indicators continuous or categorical (binary/ordinal)? This changes the default estimator and computational cost
- [ ] Are there individual-level covariates predicting the within factor (a within-level MIMIC), and/or cluster-level covariates predicting the between factor?
- [ ] Do you need a **random factor loading or random regression slope** (varies across clusters), or are all loadings/slopes fixed across clusters?
- [ ] Is there a structural path from the factor to another (observed or latent) dependent variable, i.e. is this really an SEM rather than pure CFA?
- [ ] Is this a multiple-group analysis (a `GROUPING` variable), and if so, which parameters should be allowed to differ by group?

## Option selection logic
| Situation | Choice |
|---|---|
| Pure two-level CFA, continuous indicators, cluster-level covariate predicts the between factor | `%WITHIN% fw BY y1-y4; fw ON x1 x2;` / `%BETWEEN% fb BY y1-y4; y1-y4@0; fb ON w;` (Example 9.6) |
| Same, but the within-level indicators are binary/ordinal | add `CATEGORICAL = u1-u4;`; default is ML with numerical integration; between-level indicator (random intercept) residual variances are fixed at 0 by default (Example 9.7) |
| Loading of the within factor on a covariate should vary randomly by cluster (a random slope on the factor) | `TYPE = TWOLEVEL RANDOM;` with `s1 \| fw ON x1; s2 \| fw ON x2;` on `%WITHIN%`, then `fb s1 s2 ON w;` on `%BETWEEN%` (Example 9.8) |
| Within level has categorical factor indicators; between level has its own observed continuous factor indicators plus the random-intercept indicators of the within factor | two `BY` statements on `%BETWEEN%`: one for the random-intercept factor (`fb BY u1-u6;`), one for the genuinely between-level observed indicators (`f BY y1-y4;`); use `ESTIMATOR = WLSMV;` to avoid heavy numerical integration (Example 9.9) |
| SEM-style structural path from a within factor to a dependent variable, with the path (slope) random across clusters | `s | y5 ON fw;` on `%WITHIN%`; `y5 s ON fb w;` on `%BETWEEN%`; requires `ALGORITHM = INTEGRATION;` (numerical integration) (Example 9.10) |
| Two-level CFA compared across known groups | `GROUPING = g (1 = g1 2 = g2);` + a `MODEL g2:` block overriding only the parameters that differ in group 2 (Example 9.11) |
| Weighted least squares estimation is chosen and the same within/between sample statistics will be reused in later runs | `SAVEDATA: SWMATRIX = ...;` |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE:**
   - `NAMES ARE ...;`
   - `CATEGORICAL = u1-u4;` — only if within-level indicators are binary/ordinal
   - `WITHIN = x1 x2;` — individual-level-only covariates
   - `BETWEEN = w;` (and any purely between-level observed factor indicators, e.g. `y1-y4`)
   - `CLUSTER = clus;`
   - `GROUPING = g (1 = g1 2 = g2);` — only for multiple-group two-level CFA
4. **ANALYSIS:**
   - `TYPE = TWOLEVEL;` — random-intercept factor structure only
   - `TYPE = TWOLEVEL RANDOM;` — needed for any random loading/slope (`|` statement)
   - `ALGORITHM = INTEGRATION;` — numerical-integration ML, and optionally `INTEGRATION = 10;` to change the number of integration points per dimension from the default of 15
   - `ESTIMATOR = WLSMV;` (or `WLSM`) — robust weighted least squares alternative to ML, useful when categorical indicators + many integration dimensions make ML slow
5. **MODEL:**
   - `%WITHIN%` block: `fw BY y1-y4;` (metric set by fixing the first loading to 1, can be overridden); `fw ON x1 x2;` for a within-level MIMIC; `s1 | fw ON x1;` for a random loading/slope
   - `%BETWEEN%` block: `fb BY y1-y4;` where `y1-y4` here refer to the same names as the within indicators — Mplus interprets them as the random intercepts; `y1-y4@0;` fixes their between-level residual variances to zero (common default for a "random-intercept-as-indicator" between factor); `fb ON w;` for a between-level MIMIC
6. **MODEL g2:** (multiple-group only) repeat only the `%WITHIN%`/`%BETWEEN%` statements whose parameters should differ from group 1
7. **SAVEDATA:** `SWMATRIX = filename.dat;` — optional, saves within/between sample statistics + asymptotic covariance matrix for reuse with weighted least squares estimation
8. **OUTPUT:** `TECH1 TECH8;` — recommended whenever numerical integration is used

## Sub-option details
- `fw BY y1-y4;` on `%WITHIN%`: standard CFA `BY` syntax; the metric is set by fixing the first loading to 1 by default (overridable)
- `fb BY y1-y4;` on `%BETWEEN%`: here `y1-y4` are the same variable names as the within-level indicators, but on the between side they represent each indicator's random intercept (a continuous latent variable that varies across clusters) — this is how the chapter builds a "between factor of random intercepts"
- `y1-y4@0;`: fixes the residual (measurement error) variances of the between-level indicators to zero — the default choice in these examples when the between factor is built from random intercepts, since those residual variances are typically very small and, for categorical indicators, each would otherwise require its own numerical-integration dimension
- `s1 | fw ON x1;`: names a random loading/slope of the within factor on a covariate; requires `TYPE = TWOLEVEL RANDOM;`; the resulting random effect (`s1`) is referenced on `%BETWEEN%` just like a random intercept
- Between-level factor residual variances/covariances are estimated and uncorrelated by default; within-level factor residual correlations follow ordinary CFA rules (correlated by default when factors don't predict anything else in the model)
- `ALGORITHM = INTEGRATION;` with `INTEGRATION = n;`: numerical integration for ML; the number of integration points is roughly `points_per_dimension^(number of dimensions)`, so it grows quickly — e.g. a 2-dimension model with defaults used 225 points, a 4-dimension model used 10,000; reduce dimensions or switch estimator (`WLSMV`) if this becomes too slow
- `ESTIMATOR = WLSMV;` / `WLSM;`: robust weighted least squares using a diagonal weight matrix; for categorical outcomes and many-dimension models, this avoids numerical integration and can be faster; between-level residual variances under WLS do not require numerical integration (unlike ML)
- `SAVEDATA: SWMATRIX = ...;`: saves the within- and between-level sample statistics and their estimated asymptotic covariance matrix; recommended to reduce computation time in subsequent runs on the same data (weighted least squares + `TYPE=TWOLEVEL` only)
- `GROUPING = g (1 = g1 2 = g2);` + `MODEL g2:`: multiple-group two-level CFA; parameters not respecified in `MODEL g2:` are held equal to group 1's values (standard Mplus multiple-group convention); loadings, factor structure, etc. can be freed selectively in the group-specific block
- Default estimator is maximum likelihood with robust standard errors unless numerical integration cost or the WLS examples above dictate otherwise

## Post-run operations
- Check within-level and between-level model fit/loadings separately; a between factor built from random intercepts should be interpreted cautiously if the ICCs of its indicators are low
- If factor loadings are constrained equal across within and between levels (not the default — must be done explicitly), this implies the regression of the within factor on covariates has a random intercept interpretation; otherwise treat within and between loadings as distinct parameters
- For random loadings/slopes, check the `%BETWEEN%` variance of the random effect (e.g. `s1`) for evidence of true cross-cluster heterogeneity, and check `s1 ON w` (if specified) for what explains it
- `OUTPUT: TECH1;` confirms parameter specification and starting values; `TECH8;` shows optimization progress — check both whenever `ALGORITHM=INTEGRATION` is used, since these models can be slow
- For multiple-group models, compare loadings/intercepts across `MODEL g2:` overrides to assess measurement invariance across groups
- If sampling weights or a stratified/clustered survey design (rather than a genuinely multilevel structure) are also in play, see `complex-survey-design-multilevel.md`

## Likely FAQ mapping
- "I want a factor analysis but my data has students nested in schools" → Example 9.6/9.7 pattern (two-level CFA)
- "My factor indicators are binary/ordinal, not continuous, in a multilevel CFA" → Example 9.7 pattern (`CATEGORICAL=`)
- "Does a factor loading or covariate effect vary by cluster?" → Example 9.8/9.10 pattern (`TYPE=TWOLEVEL RANDOM`, `s | ... ON ...;`)
- "I have both within-level categorical items and separate between-level (school-level) continuous indicators for a school-level construct" → Example 9.9 pattern (two `BY` statements on `%BETWEEN%`)
- "I want a structural path from a within-level factor to an outcome, and that path might vary by cluster" → Example 9.10 pattern
- "I want to compare a two-level CFA model across known groups" → Example 9.11 pattern (`GROUPING=`, `MODEL g2:`)
- "ML with numerical integration is too slow" → consider `ESTIMATOR=WLSMV`/`WLSM`, and note `SWMATRIX` for reuse
- "I also have sampling weights or a stratified survey design on top of this" → point to `complex-survey-design-multilevel.md`

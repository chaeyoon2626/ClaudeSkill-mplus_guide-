# Multilevel Mixture Modeling — Two-Level CFA / IRT Mixture Models

> Source: Mplus User's Guide v8, Chapter 10, Example(s) 10.4, 10.5
> Bundled source: references/source-pdfs/Chapter10.pdf

## One-line summary
Extends single-level factor-mixture (CFA-mixture, Example 7.17) and IRT-mixture (Example 7.27) models to two-level clustered data under `TYPE = TWOLEVEL MIXTURE`: a within-level factor (continuous indicators) or IRT factor (binary/categorical indicators) is combined with a between-level factor built from the indicators' random intercepts (continuous case) or random thresholds (categorical case), optionally alongside a between-level categorical latent variable that classifies clusters.

## Prerequisite checklist
- [ ] Cluster-ID variable (`CLUSTER=`) present in the data
- [ ] Factor indicators identified: continuous (`y1-y5`, Ex 10.4) or binary/categorical (`u1-u8`, Ex 10.5, needs `CATEGORICAL=`)
- [ ] Decide whether the categorical latent variable belongs to individuals only (`c`, no `BETWEEN` listing, Ex 10.4), or to both individuals and clusters at once (`c` within + `cb` between, `BETWEEN = cb;`, Ex 10.5)
- [ ] For continuous indicators: understand that the between-level factor (`fb`) is built from the *random intercepts* of the indicators (`y1-y5` at between), not from the indicators' raw values
- [ ] For categorical indicators: understand that a between-level categorical latent variable's indicators are the *random thresholds* of the within-level items (e.g. `[u1$1-u8$1]` at between), not the items themselves
- [ ] Be ready for numerical integration; check how many dimensions your specification implies (each estimated between-level random-effect residual variance normally costs one dimension) — Ex 10.4 uses 2 dimensions/225 points, Ex 10.5 uses 1 dimension/15 points
- [ ] Decide `STARTS=` strategy (`STARTS = 0;` used for the chapter's simple illustrative runs) and whether `PROCESSORS=` is needed

## Option selection logic
| Situation | Choice |
|---|---|
| Continuous factor indicators, mixture class variable only at individual level | `%WITHIN% fw BY y1-y5;` + `%BETWEEN% fb BY y1-y5;` with `c#1` (random mean of `c`) left uncorrelated with `fb` by default (Example 10.4) |
| Binary/ordinal factor indicators, need classes of *both* individuals and clusters at once | `CLASSES = cb(2) c(2); BETWEEN = cb;` — within factor `f` under `c`, between-level random thresholds under `cb` (Example 10.5) |
| Between-level factor should summarize continuous indicators' cluster-level variability | `%BETWEEN% %OVERALL% fb BY y1-y5;` — loadings default to the metric-setting convention (first loading fixed to 1), residual variances of the indicators themselves at between default to 0 |
| Between-level categorical latent variable should summarize categorical indicators' cluster-level variability | `%BETWEEN% %OVERALL% [u1$1-u8$1];` inside class-specific blocks of the between-level categorical latent variable, so item thresholds are random and vary by cluster class |
| Model has more than one categorical latent variable (e.g. `c` and `cb` together) | route statements through `MODEL c:` / `MODEL cb:` labeled sections; a period-joined label like `%cb#1.c#1%` addresses the interaction cell of both categorical latent variables' classes (Example 10.5) |
| Want factor loadings held equal across the within and between parts (implies the within factor's own mean varies across clusters) | noted in the chapter text as a modeling option for Ex 10.4, without example syntax shown; use the general equality-label mechanism on the loadings at both levels, see `parameter-freeing-fixing-equality-constraints.md` |
| Want the within-level categorical latent variable's random mean (`c#1`) related to the between factor `fb` | correlate them (an available alternative to the uncorrelated default), or regress `fb ON c#1`; avoid regressing `c#1 ON fb`, which the chapter text calls internally inconsistent because `fb`'s mean already varies by class of `c` (Example 10.4) |
| Want a within-level factor's variance to differ across the within-level categorical latent variable's classes | repeat the factor name inside each class-specific block, e.g. `%c#1% f; %c#2% f;`, instead of leaving the variance fixed equal across classes (Example 10.5) |
| Need faster computation on a multi-core machine | `ANALYSIS: PROCESSORS = 2;` |

## Menu path & screen fields
1. **DATA:** `FILE IS ...;`
2. **VARIABLE:**
   - `NAMES ARE ...;` / `USEVARIABLES = ...;`
   - `CATEGORICAL = u1-u8;` — only for the IRT-mixture case with binary/ordinal indicators
   - `CLASSES = c (2);` (Ex 10.4, individual-level only) or `CLASSES = cb(2) c(2);` (Ex 10.5, cluster-level + individual-level)
   - `BETWEEN = cb;` — declares which categorical latent variable is cluster-level, when more than one is used
   - `CLUSTER = clus;`
3. **ANALYSIS:**
   - `TYPE = TWOLEVEL MIXTURE;`
   - `ALGORITHM = INTEGRATION;` — can be stated explicitly (it is the default estimation algorithm for these models regardless)
   - `PROCESSORS = 2;` — optional
   - `STARTS = 0;` — used for the chapter's demo runs; raise for a real analysis
4. **MODEL:**
   - `%WITHIN% %OVERALL% fw BY y1-y5;` (continuous case) or `f BY u1-u8; [f@0];` (categorical/IRT case)
   - `%BETWEEN% %OVERALL% fb BY y1-y5;` (continuous case, random-intercept factor) or class-specific `[u1$1-u8$1];` blocks under the between-level categorical latent variable (categorical/IRT case)
   - Class-specific blocks labeled `%c#1%`, `%cb#1%`, or combined `%cb#1.c#1%`
   - `MODEL c:` / `MODEL cb:` — required once more than one categorical latent variable exists
5. **OUTPUT:** `TECH1 TECH8;`

## Sub-option details
- `fw BY y1-y5;` (within, Ex 10.4) : defines the within-level factor from the continuous indicators; metric set by fixing the first loading to 1 (overridable); indicator residual variances estimated and uncorrelated by default; factor variance estimated by default.
- `fb BY y1-y5;` (between, Ex 10.4) : defines the between-level factor from the *random intercepts* named `y1-y5` at between (not the raw indicators) — these random intercepts are the filled-circle terms at the end of each within-level loading arrow; their residual variances default to 0 (each freed one costs one integration dimension); `fb`'s own variance is estimated by default.
- `c#1*1;` / `[fb*2];` (Ex 10.4) : starting values — `c#1*1` gives the random mean of `c` (i.e. `c#1`, the between-level intercept of the class-1 logit) a starting value of 1; `[fb*2]` inside `%c#1%` gives the mean of `fb` in class 1 a starting value of 2.
- `[f@0];` (within, Ex 10.5) : fixes the mean of the IRT factor `f` at 0 in `%OVERALL%` — a standard IRT identification choice.
- `%cb#1.c#1% [u1$1-u8$1];` (Ex 10.5) : sets the random thresholds of `u1`-`u8` for the combined cell of between-class 1 (`cb#1`) and within-class 1 (`c#1`) — the period-joined label lets the two categorical latent variables' class combinations jointly influence the thresholds.
- `MODEL c: %WITHIN% %c#1% f; %c#2% f;` (Ex 10.5) : because more than one categorical latent variable exists, `MODEL c:` routes this statement to `c`'s model; repeating `f;` (with no `BY`) inside each class-specific block lets the factor's *variance* differ by class of `c`.
- `PROCESSORS = 2;` : requests 2 processors for parallel computation.
- `ALGORITHM = INTEGRATION;` : numerical integration; the default estimator for these models is maximum likelihood with robust standard errors via numerical integration regardless of whether this line is written explicitly.
- `STARTS = 0;` : turns off random starts (chapter's simplified demo runs) — raise for a real analysis, especially with more classes/dimensions.

## Post-run operations
- Same estimation-cost profile as the rest of this chapter's two-level mixture models: numerical integration cost rises with the number of between-level factors/dimensions and sample size (Ex 10.4: 2 dimensions/225 points; Ex 10.5: 1 dimension/15 points). Use `ESTIMATOR=` to change the default estimator if needed.
- Raise `STARTS=` beyond a bare demo value for a real analysis; watch `TECH8` for loglikelihood replication across starts as evidence against a local optimum.
- `TECH1`/`TECH8` serve the same diagnostic role described in `multilevel-mixture-two-level-lca-basics.md` (parameter specification/starting values; optimization history) — see that file's Post-run section for the shared `PLOT` command and `TYPE=COMPLEX` complex-survey-correction notes, which apply equally here and are not repeated per example.
- For related single-level (non-multilevel) CFA-mixture and IRT-mixture syntax and class-enumeration guidance, see `cfa-irt-models.md` and `mixture-lpa-lca-cross-sectional.md`.

## Likely FAQ mapping
- "My CFA/IRT mixture data are clustered (e.g. students in schools) — how do I add the multilevel structure?" → split `VARIABLE`/`MODEL` into within/between parts and switch to `TYPE = TWOLEVEL MIXTURE;`; for continuous indicators build a between factor from the indicators' random intercepts (Ex 10.4), for binary/ordinal indicators build a between-level categorical latent variable from the indicators' random thresholds (Ex 10.5)
- "Do I need one categorical latent variable or two?" → one (`c` only) if classes describe individuals only; two (`c` + `cb`, with `BETWEEN = cb;`) if you also want classes of clusters, routed via `MODEL c:`/`MODEL cb:`
- "What does the period in a label like `cb#1.c#1` mean?" → it addresses the interaction cell formed by combining one class of the between-level categorical latent variable with one class of the within-level categorical latent variable, letting both jointly affect a between-level parameter (Example 10.5)
- "Can the within factor's mean/variance differ by mixture class?" → mean: yes, via class-specific `[fw*...]`; variance: yes, by repeating the factor name (no `BY`) inside each class-specific block, e.g. `%c#1% f; %c#2% f;` (Example 10.5)
- "Should I correlate my within-level class random mean with the between factor, or regress one on the other?" → correlation or a one-directional `fb ON c#1` regression are both valid modeling choices per the chapter text; regressing `c#1 ON fb` instead is called internally inconsistent, since `fb`'s mean already varies by class of `c` (Example 10.4)

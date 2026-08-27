# Multilevel Mixture Modeling — Two-Level LCA / Latent Class Basics

> Source: Mplus User's Guide v8, Chapter 10, Example(s) 10.1, 10.2, 10.3, 10.6, 10.7 (+ overview text at the start of the chapter)
> Bundled source: references/source-pdfs/Chapter10.pdf

## One-line summary
Extends cross-sectional mixture/latent-class modeling (`TYPE=MIXTURE`, see `mixture-lpa-lca-cross-sectional.md`) to clustered data by adding a between (cluster) level with `TYPE=TWOLEVEL MIXTURE`, so that the categorical latent class variable — and every other part of the model — can be specified separately for individuals (`%WITHIN%`) and for clusters (`%BETWEEN%`), including the case where class membership itself is a cluster-level (rather than individual-level) property.

## Prerequisite checklist
- [ ] Confirm the data really are clustered (e.g. individuals within schools/clinics) and a cluster-ID variable exists in the data set
- [ ] Decide whether the categorical latent variable(s) belong at the individual level (e.g. `c`, classes of persons), the cluster level (e.g. `cb`, classes of clusters/schools), or both at once (e.g. `cw` at individual level and `cb` at cluster level together)
- [ ] Identify which variables are individual-level only (`WITHIN=`), cluster-level only (`BETWEEN=`), or measured at both levels (mentioned on neither list)
- [ ] Be ready for numerical integration: this is the default estimation approach for these models and gets computationally demanding as the number of factors/dimensions and sample size grow
- [ ] For a first-pass/demo run, note that `STARTS = 0;` is acceptable, but a real analysis should use enough random starts (e.g. `STARTS = 100 10;`) to protect against local optima, as illustrated in the chapter's more complex example (10.7)

## Option selection logic
| Situation | Choice |
|---|---|
| Latent classes describe individuals (people have distinct within-cluster profiles) | Individual-level categorical latent variable: `CLASSES = c (2);` with no `BETWEEN` listing for `c` — model it in the `%WITHIN%` part (Examples 10.1, 10.6) |
| Latent classes describe clusters (e.g. whole schools/clinics differ qualitatively) | Between-level categorical latent variable: `CLASSES = cb (2);` + `BETWEEN = cb;` — model it in the `%BETWEEN%` part (Example 10.2) |
| Both individual-level and cluster-level classes are needed at once | `CLASSES = cb(k1) c(k2);` + `BETWEEN = cb;`; use `MODEL c:` and `MODEL cb:` (or `MODEL cw:`) labels to separate the two categorical latent variables' model statements (Examples 10.5, 10.7, 10.10, 10.12, 10.13) |
| LCA/LCGA class indicators are categorical (binary/ordinal) | `CATEGORICAL = u1-u6;` in `VARIABLE`, same as cross-sectional LCA |
| A cluster-level categorical latent variable has its own categorical indicators (rather than being predicted from a continuous individual-level intercept) | List those indicators on `BETWEEN=` too, e.g. `BETWEEN = cb w u1-u6;` (Example 10.3) |
| Individual-level class-varying random intercepts (e.g. `c#1`, `c#2`) need to vary across clusters and be regressed on a cluster covariate `w` | Represent the (often highly correlated) random intercepts with a single between-level factor, e.g. `f BY c#1 c#2; f ON w;`, instead of giving each one its own separate regression — this also cuts the number of numerical-integration dimensions (Example 10.6) |
| Want to speed up / simplify a model where random slopes have zero variance | Use the "class-varying slopes" alternative specification (repeat the `ON` statement inside each class-specific `%c#k%` block) instead of `|` random-slope syntax; avoids having to reference the slope's mean/variance on `%BETWEEN%` |
| Need faster computation on a multi-core machine | `ANALYSIS: PROCESSORS = 2;` (or more) to parallelize |
| Need to test relationships/means involving group membership known in advance (not inferred) rather than fitting a mixture | `KNOWNCLASS` option of `VARIABLE` for multiple-group analysis under `TYPE=MIXTURE` (general feature noted in the chapter introduction, not itself demonstrated with a two-level LCA example in this chapter) |

## Menu path & screen fields
1. **DATA:** `FILE IS ...;` (free format is the default, so a `FORMAT` statement is not required)
2. **VARIABLE:**
   - `NAMES ARE ...;` / `USEVARIABLES = ...;`
   - `CATEGORICAL = u1-u6;` — if the class indicators are categorical (LCA)
   - `CLASSES = c (3);` — individual-level categorical latent variable, and/or
   - `CLASSES = cb (5) cw (4);` — cluster-level and individual-level categorical latent variables together (any mix of names/numbers of classes)
   - `WITHIN = x;` — variables measured only at the individual level, modeled only in `%WITHIN%`
   - `BETWEEN = w;` (or `BETWEEN = cb w u1-u6;` if `cb` has its own between-level indicators) — variables measured only at the cluster level, and/or which between-level categorical latent variables exist
   - `CLUSTER = clus;` — the variable identifying cluster membership
3. **ANALYSIS:**
   - `TYPE = TWOLEVEL MIXTURE;` (add `RANDOM` after `MIXTURE` when random slopes are declared with `|`, e.g. `TYPE = TWOLEVEL MIXTURE RANDOM;`)
   - `STARTS = n m;` (e.g. `STARTS = 0;` to turn random starts off for a quick demo run, or `STARTS = 100 10;` for a real analysis with several classes)
   - `PROCESSORS = 2;` — optional, for parallel computation
   - `ALGORITHM = INTEGRATION;` — can be specified explicitly (numerical integration is the default estimation algorithm for these models regardless)
   - `ESTIMATOR = ...;` — to override the default estimator (maximum likelihood with robust standard errors via numerical integration)
4. **MODEL:**
   - `%WITHIN%` / `%BETWEEN%` to separate the individual-level and cluster-level parts of the model
   - `%OVERALL%` inside each part for the model pieces common to all classes
   - Class-specific pieces labeled by categorical-latent-variable name + `#` + class number, e.g. `%c#1%`, `%cb#2%`
   - When more than one categorical latent variable exists, `MODEL c:` / `MODEL cb:` / `MODEL cw:` labeled sections separate the model statements that belong to each one
5. **OUTPUT:** `TECH1 TECH8;` is the standard pairing used throughout this chapter's examples (parameter specification/starting values, and optimization history)

## Sub-option details
- `CLASSES = c (2);` vs `CLASSES = cb (2);` : the categorical latent variable named on `BETWEEN=` (like `cb`) forms classes of clusters (e.g. schools); one not named there (like `c`) forms classes of individuals. A model can have both simultaneously, e.g. `CLASSES = cb(2) c(2);`.
- `WITHIN = x1 x2;` : identifies variables measured on the individual level and modeled only in the within part; they are given no between-level variance.
- `BETWEEN = w;` : identifies variables measured on the cluster level and modeled only in the between part; also used to declare which categorical latent variables are between-level (e.g. `BETWEEN = cb w;`).
- `%OVERALL%` : the part of the model common across all classes of the categorical latent variable in that MODEL section.
- `%c#1%`, `%cb#2%`, `%cb#1.c#1%` : labels for class-specific parts of the model. A label combining two categorical latent variables' classes with a period (e.g. `cb#1.c#1`) refers to the parameter set specific to that combination — i.e. the interaction of both categorical latent variables' influence (Example 10.5).
- `f BY c#1 c#2;` : on the between level, this represents the (typically correlated) random intercepts of the individual-level class-specific logits with a single continuous factor `f`, which can then be regressed on a cluster covariate with `f ON w;`. Its residual variances are fixed at 0 by default (each one otherwise costs one dimension of numerical integration) and can be freed if needed.
- `c ON x;` (within) / `f ON w;` (between) : multinomial logistic regression of the categorical latent variable on covariates — within-level covariates predict individual class membership directly; a cluster covariate instead predicts the between-level factor that summarizes the random class-specific intercepts.
- `cw#1-cw#3 ON cb;` : linear regression of an individual-level categorical latent variable's (here `cw`) random-mean representation on a between-level categorical latent variable (`cb`), letting cluster class membership shift the prevalence of individual-level classes (Example 10.7).
- `PROCESSORS = 2;` : requests 2 processors for parallel computation.
- `STARTS = n m;` : `n` = number of random start sets, `m` = number carried to final-stage optimization; `STARTS = 0;` disables random starts (used for the chapter's simplest illustrative runs).
- `MODEL c:` / `MODEL cb:` / `MODEL cw:` : required once a model has more than one categorical latent variable, to route each set of model statements to the correct one.
- `TYPE = TWOLEVEL MIXTURE RANDOM;` : add `RANDOM` when the model also declares random slopes with the `|` symbol (e.g. `s1 | y ON x1;`) inside a two-level mixture model.

## Post-run operations
- The default estimator for these two-level mixture models is maximum likelihood with robust standard errors using numerical integration; this becomes increasingly demanding as the number of factors/dimensions and the sample size increase (noted repeatedly across the chapter's examples — e.g. models with 2 dimensions used 225 integration points, models with 1 dimension used 15). Use `ESTIMATOR` to change the estimator if needed, and keep an eye on runtime when adding random effects or extra classes.
- Increase `STARTS` beyond a bare-bones demo value (e.g. `STARTS = 0;`) for a real analysis, especially with more classes/dimensions — the chapter's most complex example (10.7, with `cb(5) cw(4)`) uses `STARTS = 100 10;`.
- `TECH8` (screen-printed by default during computation, and available in the output when requested) is useful for tracking how long the analysis takes and following the optimization history; `TECH1` shows parameter specifications and starting values for all free parameters.
- The `PLOT` command (used with a post-processing graphics module) can produce histograms, scatterplots, plots of individual observed/estimated values, and plots of sample/estimated means and proportions — for the total sample, by group, by class, and adjusted for covariates — plus a display of descriptive statistics per variable; graphics can be exported as DIB/EMF/JPEG, and the underlying plotted data can be saved to an external file for other graphics software.
- Corrections to standard errors and the chi-square test of model fit that account for stratification, clustering (non-independence), and unequal selection probability are obtained by adding `TYPE = COMPLEX` together with the `STRATIFICATION`, `CLUSTER`, `WEIGHT`, `WTSCALE`, and `BWEIGHT`/`BWTSCALE` options of `VARIABLE`, if the design calls for complex-survey corrections in addition to the multilevel structure.

## Likely FAQ mapping
- "My LCA data are students nested in schools — how do I extend a cross-sectional LCA to account for that?" → this file: add `CLUSTER=`, split `VARIABLE`/`MODEL` into `WITHIN=`/`BETWEEN=` and `%WITHIN%`/`%BETWEEN%`, and switch `ANALYSIS: TYPE = MIXTURE;` to `TYPE = TWOLEVEL MIXTURE;` (see Example 10.6 as the closest LCA-with-covariates template)
- "Are my classes classes of people, or classes of clusters/groups?" → see the Option selection logic table; individual-level classes stay off the `BETWEEN=` list, cluster-level classes go on it
- "I want both a school-level typology and a student-level typology in the same model" → two categorical latent variables at once (`CLASSES = cb(k1) c(k2);` with `BETWEEN=cb`), routed via `MODEL c:`/`MODEL cb:` labels (Examples 10.5, 10.7, 10.12, 10.13)
- "The model is extremely slow / won't finish" → numerical integration cost rises with the number of between-level factors/dimensions and sample size; consider whether random-intercept variances can stay at their (often) zero default, whether correlated random intercepts can be summarized by a single between-level factor (Example 10.6's `f BY c#1 c#2;` trick), and whether `STARTS` and `PROCESSORS` need adjusting
- "I only got LCA class-count guidance (BIC, TECH11/TECH14, entropy) for cross-sectional models — does that change here?" → this chapter does not add multilevel-specific class-count guidance; for that decision procedure, see `mixture-lpa-lca-cross-sectional.md` (its cross-sectional Post-run operations section) and treat it as the same procedure, just with `TYPE=TWOLEVEL MIXTURE` output
- "How do I bring in survey weights/stratification alongside a two-level mixture model?" → combine `TYPE = COMPLEX` with `TYPE = TWOLEVEL MIXTURE` design-related `VARIABLE` options (`STRATIFICATION`, `WEIGHT`, `WTSCALE`, `BWEIGHT`, `BWTSCALE`) as noted in the chapter introduction — not separately demonstrated with a worked two-level LCA example in this chapter, so double check the live guide for exact combined syntax if this is central to the analysis

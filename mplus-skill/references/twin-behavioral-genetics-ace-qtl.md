# Twin & Sibling Behavioral Genetics Models (ACE / QTL)

> Source: Mplus User's Guide v8, Chapter 5, Examples 5.18, 5.19, 5.21, 5.22, 5.23
> Bundled source: references/source-pdfs/Chapter5.pdf

## One-line summary
Fits classical behavioral-genetics variance-decomposition models for twin/sibling data — univariate ACE models (Additive genetic, Common/shared environment, unique Environment) for a continuous or categorical outcome, specified either as an explicit two-group factor model (Ex 5.18/5.19) or more compactly via `MODEL CONSTRAINT` algebra (Ex 5.21/5.22) — plus a sibling QTL (quantitative trait locus) model that adds a genotype-linked variance component moderated by measured allele sharing (Ex 5.23).

## Prerequisite checklist
- [ ] Data are structured one row per twin/sibling pair, with each member's outcome stored as a separate variable (e.g. `y1`, `y2`), not one row per individual
- [ ] A grouping variable identifies zygosity (e.g. MZ vs DZ twins) — required because monozygotic (identical) pairs share ~100% and dizygotic (fraternal) pairs share ~50% of segregating genes on average, which changes the genetic covariance assumed between the two members
- [ ] Confirm outcome type: continuous (Ex 5.18/5.21) or binary/ordered categorical (Ex 5.19/5.22) — categorical twin models are formulated on an underlying normal "liability" and use thresholds instead of intercepts/residual variances
- [ ] Decide specification style: explicit A/C/E factor model (more transparent/didactic, Ex 5.18/5.19) vs. algebraic `MODEL CONSTRAINT` decomposition (more compact, directly yields a labeled heritability estimate, Ex 5.21/5.22) — both fit the mathematically identical model
- [ ] For a QTL sibling model (Ex 5.23): is there a per-pair genotypic-similarity measure available, e.g. an estimated IBD (identity-by-descent) sharing probability, to moderate the genetic covariance at a specific locus?

## Option selection logic
| Situation | Choice |
|---|---|
| Continuous twin outcome, want an explicit/didactic A/C/E factor-model view | two-group factor-model ACE, Ex 5.18 pattern |
| Continuous twin outcome, want a compact algebraic decomposition (`a`, `c`, `e`; heritability formula) | `MODEL CONSTRAINT` parameter-constraint style, Ex 5.21 pattern |
| Binary/ordinal twin outcome, explicit factor-model view | two-group factor-model ACE with `CATEGORICAL`, Ex 5.19 pattern |
| Binary/ordinal twin outcome, compact algebraic decomposition | `MODEL CONSTRAINT` style with `e` as a remainder (unit total liability variance), Ex 5.22 pattern |
| Sibling (not necessarily twin) pairs with a measured degree of allele sharing at a marker, to test for a specific genetic locus | QTL sibling model, Ex 5.23 pattern — adds a `q` (QTL) component moderated by the per-pair sharing variable |
| How to tell Mplus which pair is which zygosity/group | `GROUPING = g (1 = mz 2 = dz);` on `VARIABLE` |

## Menu path & screen fields
**[Explicit ACE factor model, continuous — 5.18]**
1. **VARIABLE:** `NAMES = y1 y2 g;` `GROUPING = g (1 = mz 2 = dz);`
2. **ANALYSIS:** `MODEL = NOCOVARIANCES;` — fixes all covariances to 0 by default so only the `WITH` statements below reinstate the specific genetic/environmental covariances
3. **MODEL:**
   - `[y1-y2] (1);` — equal intercepts across twin 1 and twin 2
   - `y1-y2@0;` — residual variances of y1, y2 fixed to 0 (all variance instead carried by the A/C/E factors)
   - `a1 BY y1*(2); a2 BY y2*(2);` — A (additive genetic) factors, loadings freed then held equal across twins
   - `c1 BY y1*(3); c2 BY y2*(3);` — C (common/shared environment) factors
   - `e1 BY y1*(4); e2 BY y2*(4);` — E (unique environment) factors
   - `a1-e2@1; [a1-e2@0];` — all six factor variances fixed to 1, means fixed to 0
   - `a1 WITH a2@1; c1 WITH c2@1;` — genetic and shared-environment correlations for the default/overall group
4. **MODEL dz:** `a1 WITH a2@.5;` — override for the DZ group: siblings/dizygotic twins share ~50% of genes on average

**[Explicit ACE factor model, categorical — 5.19]**
1. **VARIABLE:** `NAMES = u1 u2 g;` `CATEGORICAL = u1-u2;` `GROUPING = g (1 = mz 2 = dz);`
2. **ANALYSIS:** `MODEL = NOCOVARIANCES;`
3. **MODEL:**
   - `[u1$1-u2$1] (1);` — equal thresholds across twin 1 and twin 2 (no E factor here — categorical outcomes have no freely estimated residual variance to reassign)
   - `a1 BY u1*(2); a2 BY u2*(2);` `c1 BY u1*(3); c2 BY u2*(3);`
   - `a1-c2@1; [a1-c2@0];`
   - `a1 WITH a2@1; c1 WITH c2@1;`
4. **MODEL dz:** `a1 WITH a2@.5;` `{u1-u2@1};` — DZ genetic correlation of 0.5, plus WLSMV scale factors fixed to 1 in both groups so A/C variance contributions are on a common metric

**[Compact parameter-constraint ACE, continuous — 5.21; equivalent to 5.18]**
1. **VARIABLE:** `NAMES = y1 y2 g;` `GROUPING = g(1 = mz 2 = dz);`
2. **MODEL:** `[y1-y2](1);` `y1-y2(var);` `y1 WITH y2(covmz);`
3. **MODEL dz:** `y1 WITH y2(covdz);`
4. **MODEL CONSTRAINT:**
   - `NEW(a c e h);`
   - `var = a**2 + c**2 + e**2;`
   - `covmz = a**2 + c**2;`
   - `covdz = 0.5*a**2 + c**2;`
   - `h = a**2/(a**2 + c**2 + e**2);` — heritability, reported with its own SE

**[Compact parameter-constraint ACE, categorical — 5.22; equivalent to 5.19]**
1. **VARIABLE:** `NAMES = u1 u2 g;` `CATEGORICAL = u1 u2;` `GROUPING = g(1=mz 2=dz);`
2. **MODEL:** `[u1$1-u2$1](1);` `u1 WITH u2(covmz);`
3. **MODEL dz:** `u1 WITH u2(covdz);`
4. **MODEL CONSTRAINT:**
   - `NEW(a c e h);`
   - `covmz = a**2 + c**2;` `covdz = 0.5*a**2 + c**2;`
   - `e = 1 - (a**2 + c**2);` — E as a remainder because the liability has unit variance
   - `h = a**2/1;`

**[QTL sibling model, continuous — 5.23]**
1. **VARIABLE:** `NAMES = y1 y2 pihat;` `USEVARIABLES = y1 y2;` `CONSTRAINT = pihat;` — `pihat` kept out of the analysis model itself but made available to `MODEL CONSTRAINT`
2. **MODEL:** `[y1-y2](1);` `y1-y2(var);` `y1 WITH y2(cov);`
3. **MODEL CONSTRAINT:**
   - `NEW(a e q);`
   - `var = a**2 + e**2 + q**2;`
   - `cov = 0.5*a**2 + pihat*q**2;` — additive genetic covariance fixed at the sibling average of 0.5, but the QTL covariance is moderated per-pair by the observed IBD-sharing proportion `pihat`

## Sub-option details
- `GROUPING = g (1 = mz 2 = dz);` : ordinary multiple-group syntax (see `multiple-group-analysis-mechanics.md`) repurposed so each zygosity group gets its own fixed genetic-covariance assumption
- `ANALYSIS: MODEL = NOCOVARIANCES;` : overrides the usual default that exogenous variables/factors are freely correlated. A twin model needs almost every A/C/E factor pair to have a *specific*, theory-fixed covariance (1, 0.5, or 0 depending on the pair and component), so it's cleaner to start from "nothing correlated" and add back only the needed `WITH` statements
- Labeling loadings identically across twin 1 and twin 2 (e.g. `a1 BY y1*(2); a2 BY y2*(2);`) encodes the assumption that the two members of a pair are exchangeable — the same genetic/environmental architecture applies to both
- `a1 WITH a2@1;` (overall/default group) vs. `MODEL dz: a1 WITH a2@.5;` (group-specific override) is the core of the ACE logic: the genetic correlation is fixed to 1.0 for identical (MZ) twins and 0.5 for fraternal (DZ) twins/full siblings, since DZ pairs share on average half their segregating genes. `c1 WITH c2@1;` (shared environment) stays fixed at 1.0 in *both* zygosity groups, since shared environment is assumed to be shared equally regardless of zygosity
- Continuous vs. categorical changes what carries "residual" variance: with continuous outcomes, `y1-y2@0;` fixes each indicator's own residual variance to zero so all variance is decomposed purely into A+C+E; with categorical outcomes there is no free residual variance to fix in the first place, so the E factor is simply dropped from the explicit model (Ex 5.19) and recovered afterward as a remainder — `e = 1 - (a**2+c**2)` in the constraint style (Ex 5.22), or via `STANDARDIZED` output for the factor-model style — because the underlying liability is scaled to unit variance
- `{u1-u2@1};` (curly braces) : fixes the WLSMV *scale factors* to 1 in both groups, which is needed so the A and C variance contributions for u1/u2 are estimated on a common metric across zygosity groups — a categorical-model identification requirement also seen in `cfa-multigroup-categorical-invariance.md`
- The parameter-constraint style (Ex 5.21/5.22) is mathematically identical to the explicit factor-model style (Ex 5.18/5.19), but re-derives the twin covariance structure directly in terms of three scalars `a`, `c`, `e` via `MODEL CONSTRAINT` rather than via six latent factors — more compact, and it directly yields a labeled heritability parameter `h` (and its SE) via `NEW(...)`. See `cfa-efa-parameter-constraints.md` for the general mechanics of `MODEL CONSTRAINT`/`NEW`
- `CONSTRAINT = pihat;` (on `VARIABLE`, Ex 5.23) : flags a variable as usable inside `MODEL CONSTRAINT` even though it is not part of the analysis model itself (not in `USEVARIABLES`, not appearing in any `BY`/`ON`/`WITH` statement) — this is how a per-pair covariate (here, a pre-estimated IBD-sharing probability) can moderate a variance/covariance expression
- QTL logic (Ex 5.23): the additive genetic covariance between siblings is always `0.5*a**2` (siblings share half their genes on average, same as DZ twins), but the QTL-specific covariance term is `pihat*q**2` rather than a fixed 0.5 — because IBD sharing at a *specific* genetic marker/locus varies pair-to-pair around the genome-wide average of 0.5, and the observed `pihat` (typically pre-estimated from genotype data outside Mplus) captures that pair-specific value. There is no `C` (shared-environment) term in this sibling QTL example, and `e` absorbs unique-environment variance plus any locus-specific variance not captured by `q`
- Default estimator: maximum likelihood for continuous outcomes (Ex 5.18, 5.21, 5.23); robust weighted least squares by default for categorical outcomes (Ex 5.19, 5.22), with ML available as an alternative requiring numerical integration

## Post-run operations
- For the explicit ACE models (Ex 5.18/5.19), use `OUTPUT: STANDARDIZED;` to read the proportion of variance attributable to A, C, and E directly from standardized factor-loading-squared terms
- For the parameter-constraint style (Ex 5.21/5.22/5.23), read `a`, `c`, `e` (and `h`, `q`) directly from the `NEW()` parameter block in the output, each with its own SE and confidence interval
- Compare an ACE model's fit against reduced AE-only or CE-only models (drop the A or C component and refit) to test whether shared environment or additive genetic variance is actually needed — a standard behavioral-genetics model-comparison step, not itself shown in these examples but a natural extension of them
- For the QTL model, examine the estimated `q` (and its SE/significance) as the test for linkage: a `q` reliably different from zero is evidence that the marker's IBD-sharing pattern predicts phenotypic similarity beyond the polygenic background captured by `a`
- Double check that MZ vs. DZ group labels in the data match the `GROUPING` label assignment — mislabeling zygosity silently swaps which genetic correlation (1.0 vs 0.5) is applied to which group, without producing an obvious error

## Likely FAQ mapping
- "How do I fit an ACE twin model in Mplus?" → this file; the same model can be written either as an explicit A/C/E factor model (Ex 5.18/5.19) or more compactly via `MODEL CONSTRAINT` algebra (Ex 5.21/5.22) — pick whichever is easier to extend for your design
- "My twin data has binary/ordinal outcomes, not continuous" → Ex 5.19/5.22 pattern; the E component isn't directly estimated, it's the remainder after A and C
- "How do I get heritability out of Mplus with a standard error?" → `MODEL CONSTRAINT: NEW(h); h = a**2/(a**2+c**2+e**2);` pattern from Ex 5.21/5.22
- "I have sibling (not twin) data plus a genotype-similarity measure — can I test for a specific gene/locus?" → QTL sibling model, Ex 5.23, using `CONSTRAINT` on `VARIABLE` to bring the per-pair IBD-sharing variable into `MODEL CONSTRAINT`
- "Why does the covariance between the A factors differ between the overall MODEL and the group-specific MODEL dz?" → that's exactly how zygosity differences (100% vs. ~50% average gene sharing) enter the model — fixed to 1 for MZ (default/overall group) and 0.5 for DZ (group-specific override)

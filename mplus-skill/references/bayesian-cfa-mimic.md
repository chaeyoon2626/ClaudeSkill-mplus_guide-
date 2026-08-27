# Bayesian Structural Equation Modeling (BSEM) — CFA, MIMIC & Multi-Group Approximate Invariance

> Source: Mplus User's Guide v8, Chapter 5, Examples 5.31, 5.32, 5.33
> Bundled source: references/source-pdfs/Chapter5.pdf

## One-line summary
Uses Bayesian estimation (`ESTIMATOR = BAYES`) with zero-mean, small-variance informative priors on cross-loadings, direct effects, or cross-group parameter differences — an alternative to hard-fixing parameters to exactly zero (or to exact equality across groups), letting the data pull a parameter slightly away from zero/equality only if there's real evidence for it (Muthén & Asparouhov, 2012; BSEM).

## Prerequisite checklist
- [ ] Confirm the base model (bi-factor CFA, MIMIC, or multi-group CFA) is otherwise settled — this file covers the Bayesian/small-variance-prior *extension* of those models, not the base specification itself (see `cfa-continuous.md`, `esem-bifactor.md`, `cfa-multigroup-invariance.md`)
- [ ] Confirm the specific goal:
  - Cross-loadings in a bi-factor CFA that should be "approximately zero" rather than exactly fixed to zero → Example 5.31 pattern
  - Cross-loadings and/or direct effects in a MIMIC model that should be "approximately zero" → Example 5.32 pattern
  - Approximate measurement invariance across many groups (loadings/intercepts allowed to differ slightly rather than forced exactly equal) → Example 5.33 pattern
- [ ] Multiple processors available? `PROCESSORS = 2;` speeds up the (by default, two independent) MCMC chains used in Bayesian estimation

## Option selection logic
| Situation | Choice |
|---|---|
| Want a bi-factor CFA where minor cross-loadings are allowed but shrunk toward zero | label the cross-loadings, then `MODEL PRIORS: <labels>~N(0,0.01);` |
| Want a MIMIC model where cross-loadings and/or direct effects are allowed but shrunk toward zero | label them, then `MODEL PRIORS: <labels>~N(0,0.01);` |
| Want approximate (not exact) measurement invariance across many known groups | `TYPE = MIXTURE;` + `KNOWNCLASS` + `MODEL = ALLFREE;` + `MODEL PRIORS` on the *differences* between each group's parameters and a reference group's, via `DO` + `DIFF` |
| Want ordinary exact-equality invariance instead | use `cfa-multigroup-invariance.md` / `cfa-multigroup-categorical-invariance.md` (ML/WLS, no priors needed) |

## Menu path & screen fields
**[Bayesian bi-factor CFA with cross-loadings — 5.31]**
1. **ANALYSIS:** `ESTIMATOR = BAYES;` `PROCESSORS = 2;`
2. **MODEL:**
   - `fg BY y1-y10*; fg@1;` — general factor, CFA-fixed, metric set via factor variance
   - `f1 BY y1-y4` then on a new line `y5-y10 (f1xlam5-f1xlam10);` — f1's main loadings (y1-y4) plus labeled cross-loadings (y5-y10)
   - `f2 BY y5-y8` then `y1-y4 y9-y10 (f2xlam1-f2xlam6);` — f2's main loadings plus labeled cross-loadings
   - `fg WITH f1-f2@0;` — general/specific factors uncorrelated
3. **MODEL PRIORS:** `f1xlam5-f2xlam6~N(0,0.01);` — zero-mean, small-variance prior on all the labeled cross-loadings
4. **PLOT:** `TYPE = PLOT2;`

**[Bayesian MIMIC with cross-loadings and direct effects — 5.32]**
1. **ANALYSIS:** `ESTIMATOR = BAYES;` `PROCESSORS = 2;`
2. **MODEL:**
   - `f1 BY y1-y3` then `y4-y6 (xload4-xload6);` — f1's main + cross-loadings
   - `f2 BY y4-y6` then `y1-y3 (xload1-xload3);` — f2's main + cross-loadings
   - `f1-f2 ON x1-x3;` — ordinary MIMIC paths
   - `y1-y6 ON x1-x3 (dir1-dir18);` — direct effects of covariates on every indicator, labeled
3. **MODEL PRIORS:** `xload1-xload6~N(0,0.01);` `dir1-dir18~N(0,0.01);`
4. **PLOT:** `TYPE = PLOT2;`

**[Bayesian multi-group approximate invariance — 5.33]**
1. **VARIABLE:**
   - `USEVARIABLES = y1-y6 group;`
   - `CLASSES = c(10);` — 10 known classes (groups)
   - `KNOWNCLASS = c(group = 1-10);` — the observed variable `group` defines known class membership
2. **ANALYSIS:** `TYPE = MIXTURE;` `ESTIMATOR = BAYES;` `PROCESSORS = 2;`
3. **MODEL:**
   - `MODEL = ALLFREE;` — frees factor means/variances/covariances and indicator intercepts/thresholds/loadings/residual variances across all groups (except factor means fixed at 0 in the last class)
   - `%OVERALL%` `f1 BY y1-y3* (lam#_1-lam#_3);` `f2 BY y4-y6* (lam#_4-lam#_6);` `[y1-y6] (nu#_1-nu#_6);` — automatic per-group labeling using `#` (group placeholder) `_` number
   - `%c#10%` `f1-f2@1;` `[f1-f2@0];` — reference-class (10th group) identification constraints
4. **MODEL PRIORS:**
   - `DO(1,6) DIFF(lam1_#-lam10_#)~N(0,0.01);`
   - `DO(1,6) DIFF(nu1_#-nu10_#)~N(0,0.01);`
5. **PLOT:** `TYPE = PLOT2;`
6. **OUTPUT:** `TECH1 TECH8;`

## Sub-option details
- `ESTIMATOR = BAYES;` : switches to Bayesian estimation; by default two independent MCMC chains are run; `PROCESSORS = 2;` parallelizes them for speed
- Labeling loadings/paths that would otherwise be fixed to zero (cross-loadings) or that would be freely estimated without shrinkage (direct effects), then applying `MODEL PRIORS: <labels>~N(0,0.01);`, is the core BSEM device — a zero-mean prior with a small variance (here 0.01) lets these parameters move away from zero only when the data strongly support it, rather than hard-fixing them at exactly zero (as ordinary ML CFA/MIMIC would) or leaving them completely free
- Multi-group approximate invariance (Ex 5.33) uses `TYPE = MIXTURE` + `KNOWNCLASS` purely as a mechanism to run a "many-groups" Bayesian model — `KNOWNCLASS` fixes class membership to the observed `group` variable (no actual mixture/latent-class estimation occurs). `MODEL = ALLFREE` frees essentially everything across groups by default (instead of Mplus's usual equal-loadings/equal-intercepts multi-group default), and approximate invariance is imposed afterward via priors, not via equality constraints
- The automatic group-labeling convention `lam#_1` (and similarly `nu#_1`) expands the `#` placeholder across all classes to give `lam1_1, lam2_1, ..., lam10_1`, etc. — one label per group per parameter
- `DO(1,6) DIFF(lam1_#-lam10_#)~N(0,0.01);` : the `DO` loop (ranging `#` from 1 to 6, one per factor indicator) combined with `DIFF()` assigns a zero-mean, small-variance prior to the *difference* between each group's loading and the reference group's (group 10) loading — this is what "approximate" (rather than exact) invariance means: each group's loading is allowed to differ, but is shrunk toward the reference group's value
- `PLOT: TYPE = PLOT2;` : in Bayesian analysis this requests posterior parameter distributions, posterior parameter trace plots, autocorrelation plots, and posterior predictive checking scatterplots/distribution plots — the standard Bayesian diagnostic set

## Post-run operations
- Inspect trace plots (`PLOT: TYPE = PLOT2;`) for each MCMC chain to check convergence (chains should mix well and show no trend)
- Check posterior distributions of the small-variance-prior parameters (cross-loadings, direct effects, or cross-group differences) — most should stay close to zero; any that shift meaningfully away from zero despite the shrinking prior indicate real cross-loadings/non-invariance the data actually support
- Use posterior predictive checking plots to assess overall model fit in the Bayesian framework
- For the multi-group approximate-invariance model, examine which specific groups/items show non-trivial `DIFF()` posteriors — these are the practical analogue of "partial invariance" items in the ML framework

## Likely FAQ mapping
- "I don't want to force cross-loadings to exactly zero, but I also don't want them totally free" → Bayesian small-variance ("approximate zero") priors, Examples 5.31/5.32 pattern
- "I have many groups and exact measurement invariance is too strict/unrealistic" → Bayesian approximate invariance via `KNOWNCLASS` + `MODEL=ALLFREE` + `DIFF()` priors, Example 5.33 pattern
- "What does N(0,0.01) mean as a prior?" → a normal prior centered at zero with variance 0.01 (SD = 0.1) — small enough to strongly favor near-zero values but not rigidly fix them, letting the data override it when there's strong evidence
- "How many MCMC chains does Mplus run by default, and can I speed it up?" → two independent chains by default; `PROCESSORS = 2;` (or more) parallelizes them

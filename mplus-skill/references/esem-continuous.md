# Exploratory Structural Equation Modeling (ESEM) — Continuous Indicators

> Source: Mplus User's Guide v8, Chapter 5, Examples 5.24, 5.25, 5.26, 5.27
> Bundled source: references/source-pdfs/Chapter5.pdf

## One-line summary
Combines EFA-style factors (identified by rotation, no loadings fixed to zero) with the full modeling machinery of SEM — covariates/MIMIC with direct effects, structural paths onto/from CFA factors, invariance across time, and multiple-group comparisons — using the `(*label)` notation on a `BY` statement to mark a set of factors as an EFA (rotated) block.

## Prerequisite checklist
- [ ] Confirm this really needs EFA-style (rotated) factors rather than a fixed CFA loading pattern — if the loading pattern is fully known in advance, use `cfa-continuous.md`/`sem-structural-paths.md` instead
- [ ] Decide the goal:
  - Covariates predicting EFA factors, possibly with direct effects on specific indicators (measurement non-invariance test) → Example 5.24 pattern
  - Structural paths from/to EFA factors combined with ordinary CFA factors in the same model → Example 5.25 pattern
  - The same EFA factors measured at two (or more) time points, testing loading invariance and allowing correlated residuals across time → Example 5.26 pattern
  - The same EFA factors compared across known groups, at varying degrees of invariance → Example 5.27 pattern
- [ ] Confirm which rotation criterion is wanted (default: oblique GEOMIN) — see `ANALYSIS: ROTATION = ...;`

## Option selection logic
| Situation | Choice |
|---|---|
| Want covariates to predict a set of EFA factors, and/or test direct effects of covariates on individual indicators | `f1-f2 BY y1-y8(*1); f1-f2 ON x1-x2; y1 ON x1;` (MIMIC + direct effect, Ex 5.24) |
| Want EFA factors to feed into (or be predicted by) separately-defined CFA factors in one model | define the EFA block with `(*1)`, the CFA factors with plain `BY` statements, then connect them with `ON` (Ex 5.25) |
| Want to test whether the same EFA loading pattern holds across two measurement occasions | give each occasion's `BY` statement a different label but the *same* trailing invariance number, e.g. `(*t1 1)` and `(*t2 1)`, and use `PWITH` to correlate same-indicator residuals across time (Ex 5.26) |
| Want to test invariance of an EFA measurement model across known groups | `GROUPING IS ...;` + progressively remove group-specific overrides to tighten invariance, exactly as with CFA multi-group testing (Ex 5.27) |

## Menu path & screen fields
**[EFA with covariates (MIMIC) + direct effects — 5.24]**
1. **VARIABLE:** `NAMES ARE y1-y8 x1 x2;`
2. **MODEL:**
   - `f1-f2 BY y1-y8(*1);` — f1, f2 form one EFA (rotated) set, all sharing indicators y1-y8
   - `f1-f2 ON x1-x2;` — covariates predict both EFA factors
   - `y1 ON x1;` / `y8 ON x2;` — direct effects of covariates on specific indicators, used to test measurement non-invariance
3. **OUTPUT:** `TECH1;`

**[SEM combining EFA and CFA factors — 5.25]**
1. **MODEL:**
   - `f1-f2 BY y1-y6 (*1);` — EFA set
   - `f3 BY y7-y9;` / `f4 BY y10-y12;` — ordinary CFA factors
   - `f3 ON f1-f2;` — CFA factor regressed on the EFA set
   - `f4 ON f3;` — further structural path

**[EFA at two time points with loading invariance + correlated residuals — 5.26]**
1. **MODEL:**
   - `f1-f2 BY y1-y6 (*t1 1);` — time-1 EFA set, labeled t1
   - `f3-f4 BY y7-y12 (*t2 1);` — time-2 EFA set, labeled t2; the shared trailing `1` constrains the two loading matrices to be equal (loading invariance across time)
   - `y1-y6 PWITH y7-y12;` — pairs up and correlates each indicator's residual with its time-2 counterpart
2. **OUTPUT:** `TECH1 STANDARDIZED;`

**[Multiple-group EFA — 5.27]**
1. **VARIABLE:** `GROUPING IS group (1 = g1 2 = g2);`
2. **MODEL:** `f1-f2 BY y1-y10 (*1); [f1-f2@0];` (factor means fixed to 0 in both groups, overriding the usual free-in-second-group default)
3. **MODEL g2:** progressively remove statements to tighten invariance (see Sub-option details)

## Sub-option details
- `f1-f2 BY y1-y8(*1);` : the `(*<number or label>)` immediately after a `BY` statement is what marks f1, f2 as an EFA (rotated) set sharing indicators — no loadings are fixed to zero; instead identification comes from the rotation itself. When no `ROTATION` option is specified, the default oblique GEOMIN rotation is used
- For EFA factors: intercepts and residual variances of the indicators are estimated, residuals are uncorrelated by default, and (unlike CFA) the **factor variances are fixed at 1 by default** rather than the first loading being fixed to 1; factors are correlated by default under oblique GEOMIN
- `f1-f2 ON x1-x2; y1 ON x1;` : covariate paths onto the EFA factors work exactly like MIMIC; adding a direct effect from a covariate to one specific indicator (bypassing the factor) tests whether that indicator is measurement-invariant with respect to the covariate
- Combining EFA and CFA factors (Ex 5.25) in one `MODEL:` is simply a matter of defining each block with its own syntax (`(*1)` for EFA, plain `BY` for CFA) and then connecting them with ordinary `ON` statements — the structural part behaves identically to `sem-structural-paths.md`
- `(*t1 1)` / `(*t2 1)` (Ex 5.26) : the label (`t1`, `t2`) distinguishes the two EFA sets; the number (`1`) that follows constrains the factor loading matrices of the two sets to be equal — this is how loading invariance across time is imposed for EFA/ESEM factors. `PWITH` (pairwise WITH) correlates same-indicator residuals across the two time points, analogous to autocorrelated residuals in a repeated-measures CFA. By default, when loadings are held equal across time, factor variances are fixed to 1 at the first time point and freely estimated at the other; factor means are fixed at 0 at both time points by default
- Multiple-group EFA (Ex 5.27) defaults: the four rotation-based identification restrictions (loadings, variances, covariances) are imposed via rotation with factor variances fixed at 1 in **all** groups; factor means are fixed at 0 in the first group and free in the others by default. Progressively removing statements from the group-specific `MODEL <group>:` block tightens invariance:
  - No constraints: repeat `f1-f2 BY y1-y10 (*1);` and `[y1-y10];` fully in the group-specific block
  - Equal loading matrices: drop the `BY` statement from the group-specific block (equal loadings become the default)
  - + equal intercepts: also drop the `[y1-y10];` bracket statement (equal intercepts become the default; this is the ordinary invariance default)
  - + equal factor variances/covariance: in the main `MODEL:`, add `f1 WITH f2 (1); f1-f2@1;` to force the covariance equal and variances fixed to 1 in both groups
  - + equal factor means: also add `[f1-f2@0];` in the main `MODEL:` to fix means to 0 in both groups (instead of free in the non-reference group)

## Post-run operations
- For Ex 5.24-style direct effects: a significant direct effect of a covariate on a specific indicator (beyond its path through the factor) signals non-invariance for that indicator with respect to that covariate
- For Ex 5.25-style combined EFA+CFA SEM: check the structural path from the EFA set to the CFA factor(s) the same way as any `ON` coefficient, after confirming the EFA measurement part rotates to a sensible, interpretable pattern
- For Ex 5.26-style longitudinal EFA: compare fit with and without the loading-invariance constraint (shared trailing label number) to test whether the EFA structure is stable over time; inspect the `PWITH` residual correlations for evidence of unmodeled occasion-specific dependency
- For Ex 5.27-style multi-group EFA: compare fit across the nested sequence (no constraints → equal loadings → equal loadings+intercepts → + equal variances/covariance → + equal means), the ESEM analogue of configural/metric/scalar invariance testing

## Likely FAQ mapping
- "I want to let the factor structure be exploratory but still add predictors/covariates" → ESEM MIMIC (Ex 5.24 pattern)
- "Some of my factors are exploratory and some are confirmatory, and I want structural paths between them" → combined EFA+CFA SEM (Ex 5.25 pattern)
- "I have the same exploratory factors measured twice and want to test if the loadings stayed the same" → longitudinal ESEM with the shared-label trick + `PWITH` (Ex 5.26 pattern)
- "I want to test measurement invariance but my factor structure is exploratory, not confirmatory" → multiple-group EFA/ESEM (Ex 5.27 pattern), same nested-model logic as CFA invariance testing

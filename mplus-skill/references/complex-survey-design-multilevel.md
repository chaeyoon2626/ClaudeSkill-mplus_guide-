# Complex Survey Design Corrections (STRATIFICATION, CLUSTER, WEIGHT, TYPE=COMPLEX)

> Source: Mplus User's Guide v8, Chapter 9, Introduction (no separately numbered worked example is dedicated to these options within Chapter 9 — the option names and their combination rules are documented in the chapter's introductory text, which precedes Examples 9.1 onward)
> Bundled source: references/source-pdfs/Chapter9.pdf

## One-line summary
Corrects standard errors and the model chi-square test for a complex sampling design — stratification, cluster (non-independent) sampling, and/or unequal probability of selection (sampling weights) — either as a stand-alone design-based correction on top of an otherwise single-level model (`TYPE=COMPLEX`), or combined with a genuinely modeled multilevel structure (`TYPE=COMPLEX TWOLEVEL`/`TYPE=COMPLEX THREELEVEL`, or sampling weights added directly inside `TYPE=TWOLEVEL`/`TYPE=THREELEVEL`).

## Prerequisite checklist
- [ ] **Clarify the goal**: do you want to *model* the hierarchical/multilevel structure itself (random intercepts/slopes across clusters — see `two-level-regression-path-basics.md` / `two-level-cfa-sem.md`), or do you just want *correct standard errors and chi-square* for a stratified/clustered/weighted sample while otherwise fitting a single-level model? These are two different approaches described in this chapter and can also be combined.
- [ ] Do you have a stratum variable (survey strata)?
- [ ] Do you have a cluster/primary-sampling-unit (PSU) variable identifying non-independent groups of observations?
- [ ] Do you have sampling weights? If so, are they individual-level only, or do you also have separate cluster-level weights (needed only when a genuine multilevel structure — `TYPE=TWOLEVEL`/`TYPE=THREELEVEL` — is also being modeled)?
- [ ] Do you need a subpopulation/domain-restricted analysis (estimating parameters for a subset of cases, e.g. a specific demographic group, while still using the full sample's design information for correct SEs)?
- [ ] If clustering exists at more levels than you intend to explicitly model (e.g., three levels of clustering but you only want a two-level model), you may need to combine `TYPE=COMPLEX` with `TYPE=TWOLEVEL`/`TYPE=THREELEVEL`
- [ ] Confirm outcome variable types: for `TYPE=COMPLEX` alone, continuous, censored, binary, ordered categorical, nominal, count, or combinations are all supported. For `TYPE=THREELEVEL` combined with complex survey features, only continuous outcomes are supported — categorical outcomes and `TYPE=CROSSCLASSIFIED` are estimated with Bayes, for which complex survey (weight/stratification) corrections have not been developed

## Option selection logic
| Situation | Choice |
|---|---|
| No genuine multilevel structure needed — just want SE/chi-square corrected for stratification, clustering, and/or unequal selection probability | `ANALYSIS: TYPE = COMPLEX;` + `VARIABLE: STRATIFICATION = strat; CLUSTER = clus; WEIGHT = wt;` |
| Also want the analysis restricted to a subpopulation/domain while keeping full-sample design info for SEs | add `VARIABLE: SUBPOPULATION = subpop;` |
| Want a genuinely modeled two-level structure (random intercepts/slopes) that also uses individual- and/or cluster-level sampling weights | `ANALYSIS: TYPE = TWOLEVEL;` + `VARIABLE: CLUSTER = clus; WEIGHT = wt;` (individual-level weight) and, if a cluster-level weight also exists, `WTSCALE = ...; BWEIGHT = bwt; BWTSCALE = ...;` |
| Want a genuinely modeled three-level structure with weights | `ANALYSIS: TYPE = THREELEVEL;` + `VARIABLE: CLUSTER = clus2 clus3; WEIGHT = wt; WTSCALE = ...; B2WEIGHT = bwt2; B3WEIGHT = bwt3; BWTSCALE = ...;` (continuous outcomes only) |
| There is clustering at a higher level than the multilevel model explicitly represents (e.g., two cluster variables exist but you're fitting `TYPE=TWOLEVEL`) | `ANALYSIS: TYPE = COMPLEX TWOLEVEL;` + `VARIABLE: STRATIFICATION = strat; CLUSTER = clus_hi clus_lo; WEIGHT = wt; ...` — the highest cluster level's SE/chi-square correction is handled design-based via `COMPLEX`, while the lower (modeled) cluster level is handled via `TWOLEVEL` |
| Same idea but for three explicitly modeled clustering levels plus one higher, unmodeled sampling-design level | `ANALYSIS: TYPE = COMPLEX THREELEVEL;` with the corresponding `STRATIFICATION=`, `CLUSTER=`, `WEIGHT=`, `WTSCALE=`, `B2WEIGHT=`, `B3WEIGHT=`, `BWTSCALE=` options |
| Categorical outcomes needed together with `TYPE=THREELEVEL`, or a `TYPE=CROSSCLASSIFIED` design | complex survey (stratification/weight) corrections are **not available** — these are estimated using Bayes, for which the corrections have not been generally developed; note this limitation to the user rather than attempting it |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE = ...;**
3. **VARIABLE:**
   - `NAMES ARE ...;`
   - `STRATIFICATION = strat;` — stratum identifier (used with `TYPE=COMPLEX`)
   - `CLUSTER = clus;` — sampling cluster/PSU identifier; also doubles as the multilevel cluster variable when combined with `TYPE=TWOLEVEL`/`TYPE=THREELEVEL`
   - `WEIGHT = wt;` — individual-level sampling weight
   - `WTSCALE = ...;` — rescaling option for the individual-level weight (multilevel weighted analyses)
   - `BWEIGHT = bwt;` — cluster-level (between) sampling weight, `TYPE=TWOLEVEL` only
   - `B2WEIGHT = ...;` / `B3WEIGHT = ...;` — level-2 / level-3 cluster weights, `TYPE=THREELEVEL` only
   - `BWTSCALE = ...;` — rescaling option for the cluster-level weight
   - `SUBPOPULATION = subpop;` — restricts analysis to a subpopulation/domain, used with `TYPE=COMPLEX`
4. **ANALYSIS:**
   - `TYPE = COMPLEX;` — design-based SE/chi-square correction only, no multilevel structure modeled
   - `TYPE = TWOLEVEL;` / `TYPE = THREELEVEL;` — genuine multilevel structure, optionally with weights via the `VARIABLE` options above
   - `TYPE = COMPLEX TWOLEVEL;` / `TYPE = COMPLEX THREELEVEL;` — both combined
5. **MODEL:** for `TYPE=COMPLEX` alone, an ordinary single-level model (no `%WITHIN%`/`%BETWEEN%` split); for `TYPE=TWOLEVEL`/`TYPE=THREELEVEL` (with or without `COMPLEX`), use the `%WITHIN%`/`%BETWEEN%` structure described in `two-level-regression-path-basics.md` / `two-level-cfa-sem.md`

## Sub-option details
- `TYPE = COMPLEX`: corrects standard errors and the model chi-square test of fit for stratification, non-independence of observations due to cluster sampling, and/or unequal probability of selection; with sampling weights, parameters are estimated by maximizing a **weighted loglikelihood function**, and standard errors use a **sandwich estimator**
- `STRATIFICATION=`, `CLUSTER=`, `WEIGHT=`, `SUBPOPULATION=`: the four `VARIABLE`-command options used with `TYPE=COMPLEX` for the design-based approach; use any subset relevant to the actual design (e.g. only `CLUSTER=` and `WEIGHT=` if there is no stratification)
- `TYPE = TWOLEVEL` / `TYPE = THREELEVEL` **with weights**: this is the "model each level explicitly" approach — sampling weights (individual- and/or cluster-level) are allowed in estimating parameters, standard errors, and the chi-square test; with sampling weights, parameters are again estimated via a weighted loglikelihood with sandwich-based SEs
- `WEIGHT=` vs `BWEIGHT=`/`B2WEIGHT=`/`B3WEIGHT=`: `WEIGHT=` is the individual (lowest-level) sampling weight; `BWEIGHT=` is the single cluster-level weight for `TYPE=TWOLEVEL`; `B2WEIGHT=`/`B3WEIGHT=` are the level-2/level-3 cluster weights for `TYPE=THREELEVEL`
- `WTSCALE=` / `BWTSCALE=`: named options for rescaling the individual-level and cluster-level weights respectively (used with the multilevel weighted approach)
- `TYPE = COMPLEX TWOLEVEL` / `TYPE = COMPLEX THREELEVEL`: combines both approaches. When there is clustering due to more cluster variables than are explicitly modeled, the standard errors and chi-square test of model fit account for the **highest** cluster level via the `COMPLEX` design-based correction, while the **lower**, explicitly modeled cluster level(s) are handled by `TWOLEVEL`/`THREELEVEL`
- Outcome variable type support: `TYPE=COMPLEX` alone supports continuous, censored, binary, ordered categorical, nominal, count, or combinations. `TYPE=TWOLEVEL` (with or without `COMPLEX`) also supports all these types. `TYPE=THREELEVEL` (with or without `COMPLEX`) supports **continuous only** — complex survey features are not available for `TYPE=THREELEVEL` with categorical variables, nor for `TYPE=CROSSCLASSIFIED`, because those are estimated with Bayes and complex survey corrections have not generally been developed for Bayesian estimation

## Post-run operations
- Compare weighted vs. unweighted (or design-corrected vs. uncorrected) results if design effects are of interest — standard errors under `TYPE=COMPLEX` (or a weighted `TYPE=TWOLEVEL`/`THREELEVEL`) use a sandwich (robust) estimator and will generally differ from a naive single-level, unweighted analysis
- If combining `TYPE=COMPLEX` with `TYPE=TWOLEVEL`/`THREELEVEL`, double check which cluster variable is being handled by which mechanism — the highest-level cluster (design-based, `COMPLEX`) vs. the lower, explicitly modeled cluster level(s)
- If `SUBPOPULATION=` was used, confirm the reported N and parameter estimates reflect only the subpopulation, while the SEs still reflect the full sample's design
- For an actual model to place inside `MODEL:`, combine this file's `VARIABLE`/`ANALYSIS` options with the modeling patterns in `two-level-regression-path-basics.md` or `two-level-cfa-sem.md`

## Likely FAQ mapping
- "My survey has stratified/clustered sampling and sampling weights, but I don't need a genuine multilevel model" → `TYPE = COMPLEX;` with `STRATIFICATION=`, `CLUSTER=`, `WEIGHT=`
- "I want to restrict the analysis to a subgroup but keep the full sample's design info for correct SEs" → `SUBPOPULATION=`
- "I have both an individual-level weight and a school-level (cluster) weight, and I'm fitting a real two-level model" → `TYPE = TWOLEVEL;` with `WEIGHT=`, `WTSCALE=`, `BWEIGHT=`, `BWTSCALE=`
- "My data has clustering at more levels than I want to model (e.g., students in classes in schools in districts, but I only want a two-level model)" → `TYPE = COMPLEX TWOLEVEL;` (or `COMPLEX THREELEVEL`)
- "Can I do this with categorical outcomes and three levels?" → not for `TYPE=THREELEVEL`'s complex-survey features (continuous only); not for `TYPE=CROSSCLASSIFIED` either (Bayes-only, no complex survey correction developed)
- "What's the actual MODEL syntax once I add these design/weight options?" → point to `two-level-regression-path-basics.md` / `two-level-cfa-sem.md` for the `%WITHIN%`/`%BETWEEN%` model statements, or an ordinary single-level model if using plain `TYPE=COMPLEX`

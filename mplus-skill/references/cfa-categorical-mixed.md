# Confirmatory Factor Analysis (CFA) — Categorical / Censored / Count Indicators

> Source: Mplus User's Guide v8, Chapter 5, Examples 5.2, 5.4
> https://www.statmodel.com/HTML_UG/chapter5V8.htm

## One-line summary
A CFA where the pre-specified factor indicators are binary/ordinal categorical, censored, or count variables (or a mix), rather than continuous.

## Prerequisite checklist
- [ ] Confirm the measurement structure is already decided (which items load on which factor) — if not, this is EFA, see `efa-categorical-mixed.md` instead
- [ ] Confirm the exact type of each indicator: categorical (binary/ordinal), censored, count, or continuous — if types differ across indicators, mixed-type handling applies
- [ ] Link function preference for categorical indicators: probit (default, robust WLS) vs. logistic (needs ML)
- [ ] Awareness that mixed-type / all-categorical CFA with ML uses numerical integration, which slows down as the number of factors grows

## Option selection logic
| Situation | Choice |
|---|---|
| All indicators binary/ordinal categorical | `CATEGORICAL ARE u1-u6;` (default robust WLS, fast) |
| Indicators are censored (floor/ceiling piled values) | `CENSORED ARE y1-y3 (a);` (`(a)`=above/ceiling, `(b)`=below/floor) |
| Indicators are count data | `COUNT ARE u4-u6;` |
| Mixed types across factors/indicators | specify each variable's type individually; ML with numerical integration is used automatically |

## Menu path & screen fields
**[Categorical — 5.2]**
1. **VARIABLE:** `NAMES ARE u1-u6;` / `CATEGORICAL ARE u1-u6;`
2. **MODEL:** `f1 BY u1-u3; f2 BY u4-u6;`

**[Censored + count mixed — 5.4]**
1. **VARIABLE:**
   - `NAMES ARE y1-y3 u4-u6;`
   - `CENSORED ARE y1-y3 (a);` — ceiling-censored
   - `COUNT ARE u4-u6;` — count
2. **MODEL:** `f1 BY y1-y3; f2 BY u4-u6;`
3. **OUTPUT:** `TECH1 TECH8;` (recommended — numerical integration models can be slow to converge)

## Sub-option details
- `CATEGORICAL ARE u1-u6;` : category counts auto-detected; default estimator = robust WLS (probit link)
- Default estimator for censored/count/mixed CFA = "maximum likelihood with robust standard errors using a numerical integration algorithm" — this is inherently more computationally intensive than the WLS default for pure categorical CFA
- The censoring bound (floor/ceiling value) is derived automatically from the observed data
- For count indicators, a Poisson regression links the factor to each count variable by default (no zero-inflation unless separately specified, following the same `(i)`/`(nb)` logic as regression — see `regression-categorical-count.md`)

## Post-run operations
- Standard fit indices (CFI/TLI/RMSEA) are less standardized for WLS-estimated categorical CFA than for continuous CFA — check which fit statistics the estimator actually reports (WLSMV-family estimators report a scaled/robust chi-square)
- For numerical-integration models (mixed types), watch `TECH8` for convergence and be prepared for longer run times as the number of factors grows
- Loadings on categorical/censored/count indicators are on a probability/rate scale — caution when comparing magnitudes directly against continuous-indicator loadings in the same model

## Likely FAQ mapping
- "My survey items are all Likert-scale and I already know the factor structure" → `CATEGORICAL=` CFA
- "Some indicators are counts, some are continuous, some are floor-censored" → mixed-type CFA, tag each variable's type
- "This is taking a long time to run" → note the numerical-integration cost, consider whether all indicators truly need to be modeled with the mixed-type approach

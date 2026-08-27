# Confirmatory Factor Analysis (CFA) — Continuous Indicators (+ MIMIC)

> Source: Mplus User's Guide v8, Chapter 5 (Confirmatory Factor Analysis and Structural Equation Modeling), Examples 5.1, 5.8
> https://www.statmodel.com/HTML_UG/chapter5V8.htm

## One-line summary
Fits a pre-specified (theory-driven) measurement model where each latent factor is measured by a fixed, known set of continuous indicators — unlike EFA, the loading pattern is fixed by the user, not estimated freely. Optionally, add covariates that predict the factors (a MIMIC model).

## Prerequisite checklist
- [ ] **Confirm the measurement structure**: how many factors, and exactly which items load on which factor (a CFA requires you to specify this in advance — if you don't know the structure yet, that's EFA, not CFA; point to `efa-continuous.md` instead)
- [ ] Confirm all indicators are continuous (or treated as continuous)
- [ ] Are there covariates meant to predict the factors (a MIMIC model), or is this a pure measurement model?
- [ ] Should the factors be allowed to correlate with each other (default: yes, for exogenous factors), or should independence be assumed?
- [ ] Are cross-loadings (an item loading on more than one factor) expected, or is each item assumed to load on exactly one factor?

## Option selection logic
| Situation | Choice |
|---|---|
| Pure measurement model, no predictors of the factors | plain CFA: `f1 BY y1-y3; f2 BY y4-y6;` |
| Want to test how covariates relate to/predict the factors | MIMIC: add `f1 f2 ON x1-x3;` |
| An item is expected to load on more than one factor | list it in more than one `BY` statement, or free it explicitly, e.g. `f1 BY y1-y3; f2 BY y4-y6 y3;` |
| Want the factors to be uncorrelated | fix the covariance to 0: `f1 WITH f2@0;` (default is freely estimated/correlated) |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE: NAMES ARE ...;**
4. **MODEL:**
   - `f1 BY y1-y3;` — defines factor f1, measured by y1, y2, y3
   - `f2 BY y4-y6;` — a second factor
   - (MIMIC only) `f1 f2 ON x1-x3;` — regress the factors on covariates
5. **OUTPUT:** (optional) `STDYX;`, `MODINDICES;`

## Sub-option details
- `f1 BY y1-y3;` : the `BY` statement is the measurement-model syntax — "f1 is measured by y1, y2, y3." The metric of f1 is set automatically by fixing the first loading (on y1) to 1
- Intercepts and residual variances of indicators are estimated by default; residuals are uncorrelated by default (add `y1 WITH y2;` explicitly if a residual correlation is theoretically expected)
- Factors are correlated by default when treated as exogenous (this is the standard CFA assumption, distinct from EFA where correlation is a rotation choice)
- `f1 f2 ON x1-x3;` : a MIMIC extension — studies how covariates relate to the latent factors, useful for exploring population heterogeneity or as a precursor to measurement-invariance testing
- Default estimator = ML

## Post-run operations
- Check standard CFA fit indices (CFI, TLI, RMSEA, SRMR) — unlike a saturated path model, a CFA with a fixed loading pattern is normally over-identified, so these are meaningful
- `OUTPUT: MODINDICES;` flags cross-loadings or residual correlations that, if freed, would most improve fit — treat these as hypotheses to consider, not automatic edits
- Standardized loadings: `OUTPUT: STDYX;` — loadings above ~.4-.5 are conventionally considered acceptable indicators of the factor

## Likely FAQ mapping
- "I already know which items belong to which factor — I just want to confirm the structure" → this file (CFA), not EFA
- "I want to see whether a covariate (e.g. gender, age) predicts the factor" → MIMIC extension
- "How do I know if my CFA model fits well?" → CFI/TLI/RMSEA/SRMR, point to modification indices if fit is poor
- "Can an item load on two factors?" → yes, list it in both `BY` statements

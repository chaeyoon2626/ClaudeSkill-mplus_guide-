# Regression — Continuous / Censored Dependent Variables

> Source: Mplus User's Guide v8, Chapter 3 (Regression and Path Analysis), Examples 3.1–3.3
> https://www.statmodel.com/HTML_UG/chapter3V8.htm

## One-line summary
Simple/multiple regression for a dependent variable that is either continuous, or piled up at a specific floor/ceiling value (censored).

## Prerequisite checklist
Always confirm these with the user (all must be settled before moving to code generation):

- [ ] **Number and type of dependent variable(s)**: Is it continuous? Or does it pile up at a specific lower/upper bound (censored)?
  - If censored: is it censored from above (ceiling effect) or from below (floor effect)?
  - Is there an unusually large cluster of observations exactly at the censoring point, such that you also want to separately model "the probability of being at that point"? (i.e., is censored-inflation needed?)
- [ ] **List of independent variables (predictors)**: how many, and is each continuous or categorical?
- [ ] **Data file format**: raw data ready (.dat/.csv, recommended with no variable-name header row), missing value code
- [ ] **Estimator preference**: default (ML, or auto-selected based on the situation) unless there's a specific reason not to; whether robust SE (MLR) is needed

## Option selection logic (branching by data/situation)

| Situation | Choice | Mplus option |
|---|---|---|
| DV is purely continuous, no censoring/piling | Linear regression | No option needed (continuous is the default) |
| DV is censored at a lower bound (e.g. values pile up at the minimum) | Censored regression (below) | `CENSORED = y1 (b);` |
| DV is censored at an upper bound (ceiling effect) | Censored regression (above) | `CENSORED = y1 (a);` |
| The probability of being exactly at the censoring point is itself of interest (e.g. people piled at 0 differ systematically) | Censored-inflated | `CENSORED = y1 (bi);` or `(ai)` |
| Large sample, concerned about non-normality | Use robust standard errors | `ANALYSIS: ESTIMATOR = MLR;` |

## Menu path & screen fields (Mplus input file blocks)
Mplus is configured through a text syntax file (.inp), not GUI menus. The following ordered "command blocks" are the equivalent of screens/input fields.

1. **TITLE:** — description of the analysis (free text, useful when interpreting results later)
2. **DATA: FILE IS ...;** — path to the data file
3. **VARIABLE:**
   - `NAMES ARE ...;` — list every variable name in the data file (must be in order, since the file itself has no header row)
   - `USEVARIABLES ARE ...;` — select only the variables actually used in this analysis
   - `CENSORED ARE y1 (b);` — designate the censored variable (censored models only)
4. **ANALYSIS:**
   - `ESTIMATOR = MLR;` — specify only when robust standard errors are needed (default is ML/WLSMV etc. depending on the situation)
5. **MODEL:**
   - `y1 ON x1 x3;` — DV ON predictor1 predictor2 ...; form
   - For censored-inflated, add `y1#1 ON x1 x3;` (models the inflation part)
6. **OUTPUT:** (optional) — `STDYX;`, `SAMPSTAT;`, etc.

## Sub-option details
- `CENSORED ARE y1 (b);` : `(b)` = below (lower-bound censoring), `(a)` = above (upper-bound censoring), `(bi)`/`(ai)` = with inflation
- `y1 ON x1 x3;` : regress the DV on multiple predictors simultaneously (multiple regression)
- For censored-inflated, `y1#1 ON x1 x3;` is the logistic part predicting "the probability of being at the censoring point"
- `ESTIMATOR = MLR;` : makes standard errors robust to heteroscedasticity/non-normality instead of the default WLS/ML

## Post-run operations
- What to check in the results (.out) file: model fit (a continuous multiple regression is saturated, so there's no separate fit index), coefficients/SE/p-values under `y1 ON`
- If standardized coefficients are needed, add `OUTPUT: STDYX;`
- For censored models, note to the user that the coefficients are interpreted in terms of an underlying continuous latent propensity, so care is needed when interpreting them on the original scale

## Likely FAQ mapping
- "There are a lot of zeros in my dependent variable — can I just run a regular regression?" → need to re-confirm whether censored-inflation applies → guide with this file
- "All the high values are piled up at 100 (ceiling effect)" → `CENSORED (a)` option
- "My standard errors look off" → recommend `ESTIMATOR=MLR`

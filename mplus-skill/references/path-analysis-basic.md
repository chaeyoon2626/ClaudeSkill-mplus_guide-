# Path Analysis — Basic Mediation Model (mixed DV types)

> Source: Mplus User's Guide v8, Chapter 3, Examples 3.11–3.15
> https://www.statmodel.com/HTML_UG/chapter3V8.htm

## One-line summary
Estimates, in a single model, a structure where several observed variables (no latent variables) flow through mediators to a final dependent variable — regardless of variable type (continuous / categorical / censored / nominal).

## Prerequisite checklist
- [ ] **Confirm the path diagram (in structural-equation form)**: clearly establish, with arrows, which variables are independent (x), which are mediators, and which is the final dependent variable (y)
- [ ] **Confirm the type of each mediator/dependent variable individually** (branch in the table below if even one differs):
  - continuous / binary-ordinal categorical / nominal (unordered) / censored
- [ ] If categorical variables are present, probit (default) vs. logistic (ML) interpretation preference
- [ ] For categorical variable estimation, whether Delta (default) or Theta parameterization is needed (Delta is usually sufficient — Theta is only for when you want the residual variances across categories to be free parameters)
- [ ] Whether any latent variables (factors) are involved at all (if so, this isn't path analysis but an SEM/CFA procedure — check that manual separately)

## Option selection logic
| Mediator/DV combination | Mplus handling |
|---|---|
| All continuous | Standard linear regression system, no special option |
| All binary/ordinal categorical | `CATEGORICAL ARE u1-u3;` (default probit, logistic if ML specified) |
| Binary/ordinal + Theta parameterization needed | above + `ANALYSIS: PARAMETERIZATION = THETA;` |
| Continuous mediator + categorical final DV mixed | tag only that variable with `CATEGORICAL=`, the rest are treated as continuous automatically |
| A complex mix of censored + ordinal + nominal | specify `CENSORED=`, `CATEGORICAL=`, `NOMINAL=` separately per variable |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE:**
   - `NAMES ARE ...;`, `USEVARIABLES ARE ...;`
   - Type designation per variable: `CENSORED IS y1 (a);` / `CATEGORICAL IS u1;` / `NOMINAL IS u2;` (omit if none apply — treated as continuous)
4. **ANALYSIS:** (if needed) `PARAMETERIZATION = THETA;`
5. **MODEL:**
   - Translate the path diagram directly into `DV ON predictors;` statements. If multiple DVs share the same predictor set, they can be combined on one line: `y1 y2 ON x1 x2 x3;`
   - A mediator acts as a predictor for the next stage: `y3 ON y1 y2 x2;`
   - For specific nominal categories only: `u2#1 u2#2 ON y1 u1 x2;`

## Sub-option details
- `y1 y2 ON x1 x2 x3;` : listing several DVs on the left generates a separate regression equation for each against the same predictor set (coefficients are estimated separately per DV)
- `PARAMETERIZATION = THETA;` : frees the residual variance of a categorical DV as a parameter while fixing the scale factor (the default, Delta, does the opposite) — rarely used except for special purposes like multi-group comparisons
- Only **one** type designation is allowed per variable (a variable can't be both CENSORED and CATEGORICAL at once)

## Post-run operations
- If you also need the numeric value of the mediation (indirect) effect, a `MODEL INDIRECT:` block is required → see `path-analysis-mediation-bootstrap-missing.md`
- Standardized coefficients: `OUTPUT: STDYX;`
- If the model is saturated, fit indices may be meaningless — only check CFI/TLI/RMSEA etc. when it's a "non-saturated" model with some paths among the mediators/DVs omitted

## Likely FAQ mapping
- "I have several mediators — can I fit them all in one model?" → this file (basic path analysis)
- "Each dependent variable has a different scale (a mix of continuous + categorical + nominal)" → see the table, tag each variable's type
- "I also want the size and significance of the indirect (mediation) effect" → point to `path-analysis-mediation-bootstrap-missing.md`
- "I also want a latent variable (factor) in this model" → that's an SEM procedure, not path analysis, and note that manual chapter isn't available yet

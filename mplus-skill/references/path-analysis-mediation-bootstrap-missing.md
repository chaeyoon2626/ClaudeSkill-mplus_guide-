# Path Analysis — Mediation Testing (Bootstrap) and Missing Data

> Source: Mplus User's Guide v8, Chapter 3, Examples 3.16–3.17
> https://www.statmodel.com/HTML_UG/chapter3V8.htm

## One-line summary
Test the size and statistical significance of an indirect (mediation) effect with a bootstrap confidence interval, or estimate a path model that accounts for missing data in the mediator/dependent variables.

## Prerequisite checklist
- [ ] Are the prerequisites from `path-analysis-basic.md` (path diagram, variable types) already confirmed?
- [ ] **Is testing the indirect effect the goal?** → get specific about which path (x→mediator→y) the indirect effect is for (e.g. x1 → y1 → y3)
  - [ ] Preferred number of bootstrap draws (1000 is the usual default; up to 5000 depending on journal/thesis requirements)
  - [ ] Confidence interval type: bias-corrected needed, or is a simple percentile (asymmetric) bootstrap CI enough?
- [ ] **Is there missing data?** → which variable(s) have missing values, and the missing-value code (e.g. 999, -99)
  - [ ] Is the variable with missing data categorical or continuous (numerical integration is needed if there's missingness in a continuous mediator together with a categorical DV)

## Option selection logic
| Situation | Choice |
|---|---|
| Need the size/significance of the indirect effect | `MODEL INDIRECT:` + `ANALYSIS: BOOTSTRAP = 1000;` + `OUTPUT: CINTERVAL (BOOTSTRAP);` |
| Missing data in mediator/DV, categorical DV present | `MISSING IS variable (code);` + `ESTIMATOR = MLR;` + `ANALYSIS: INTEGRATION = MONTECARLO;` |
| Missing data present, everything continuous | `MISSING IS variable (code);` alone is enough (FIML applies by default, no separate integration needed) |

## Menu path & screen fields
**[Bootstrapped indirect effect]**
1. **VARIABLE:** specify variable types the same way as in basic path analysis
2. **ANALYSIS:** `BOOTSTRAP = 1000;` — number of resampling draws
3. **MODEL:** the existing path statements
4. **MODEL INDIRECT:** in the form `y3 IND y1 x1;`, i.e. "finalDV IND mediator startingVariable;", listing whichever indirect paths are wanted (multiple lines allowed)
5. **OUTPUT:** `CINTERVAL (BOOTSTRAP);` — outputs the asymmetric bootstrap confidence interval

**[Missing data handling]**
1. **VARIABLE:** `MISSING IS y (999);` — specify the missing-value code
2. **ANALYSIS:** `ESTIMATOR = MLR;` (+ `INTEGRATION = MONTECARLO;` if a categorical DV is present)
3. **MODEL:** same as before
4. **OUTPUT:** `TECH1 TECH8;` — for checking parameter specification and estimation convergence (optional)

## Sub-option details
- `MODEL INDIRECT: y3 IND y1 x1;` : requests "the indirect effect of x1 on y3 through y1." If another path shares the same start/end variables, the total effect is computed automatically too
- `BOOTSTRAP = 1000;` : resamples the data 1000 times to compute standard errors/confidence intervals; recommend increasing the number of draws for smaller samples
- `CINTERVAL (BOOTSTRAP);` : an asymmetric confidence interval with no normality assumption (the standard recommended approach for mediation testing)
- `MISSING IS y (999);` : treats 999 as missing, handled via FIML (full-information maximum likelihood)
- `INTEGRATION = MONTECARLO;` : Monte Carlo integration (500 points by default) to reduce the computational burden when a categorical DV and a missing mediator occur together

## Post-run operations
- Check each path's coefficient/confidence interval in the "TOTAL, TOTAL INDIRECT, SPECIFIC INDIRECT EFFECTS" section of the results
- If the confidence interval doesn't include 0, that indirect effect is statistically significant
- For missing-data models, recommend checking `TECH8` to confirm the iterative estimation converged normally (i.e., the log-likelihood stabilized)

## Likely FAQ mapping
- "How do I check whether the mediation effect is significant?" → `MODEL INDIRECT` + bootstrap CI
- "How many bootstrap draws should I run?" → default 1000, recommend up to 5000 for small samples
- "Some respondents didn't answer the mediator question" → the missing-data handling section
- "There's missing data and also a categorical dependent variable" → explain why `INTEGRATION=MONTECARLO` is needed

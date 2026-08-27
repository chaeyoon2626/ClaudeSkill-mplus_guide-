# Growth Modeling — Basic Linear (and Quadratic) Growth Model for a Continuous Outcome

> Source: Mplus User's Guide v8, Chapter 6, Examples 6.1, 6.8, 6.9
> Bundled source: references/source-pdfs/Chapter6.pdf

## One-line summary
Estimates a latent growth curve — intercept and slope (and optionally quadratic) growth factors — for a continuous outcome measured at several occasions, using Mplus's multivariate (SEM-style) approach with the `|` symbol.

## Prerequisite checklist
- [ ] Outcome is continuous and measured at three or more occasions (the manual's examples use four: y11, y12, y13, y14)
- [ ] Decide whether the occasions are equidistant with known spacing (fixed time scores) or whether the spacing/shape should instead be estimated from the data (free/estimated time scores)
- [ ] Decide whether a straight-line (linear) trajectory is assumed, or whether curvature (quadratic) should be modeled
- [ ] Confirm no covariates are needed yet (if covariates, piecewise growth, or individually-varying observation times are needed, see `growth-covariates-piecewise.md`)

## Option selection logic
| Situation | Choice |
|---|---|
| Occasions equally spaced, spacing known in advance, straight-line growth assumed | Fix time scores at 0, 1, 2, 3 in the `\|` statement (Example 6.1) |
| Spacing/shape of growth over time is unknown and should be estimated from the data | Fix two time scores for identification, free the rest with `*` and a starting value (Example 6.8) |
| Growth is expected to accelerate or decelerate over time (curvature) | Add a third growth factor (quadratic slope) whose time scores are the squares of the linear time scores (Example 6.9) |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE:**
   - `NAMES ARE y11-y14 x1 x2 x31-x34;`
   - `USEVARIABLES ARE y11-y14;` (select just the four outcome occasions if the data set contains other variables)
4. **MODEL:**
   - Basic linear growth: `i s | y11@0 y12@1 y13@2 y14@3;`
   - Quadratic growth: `i s q | y11@0 y12@1 y13@2 y14@3;` (time scores for `q` are computed automatically as the squares of the linear time scores)
   - Estimated (free) time scores: `i s | y11@0 y12@1 y13*2 y14*3;` (the `*` frees a time score at the given starting value; the first two occasions stay fixed at 0 and 1 for identification)

## Sub-option details
- `i s | y11@0 y12@1 y13@2 y14@3;` — the `\|` symbol names and defines the growth factors. Names on the left (`i`, `s`) are the new latent intercept and slope growth factors; the statement on the right lists the outcome variables and their time scores. `@` fixes a time score to a specific value; `*` frees it (with the number given as a starting value).
- The time score of 0 for the slope factor at the first occasion defines the intercept factor `i` as an "initial status" factor (its mean/variance describe the outcome at time 0).
- The intercept growth factor's loadings on the outcomes are fixed at 1 as part of the growth model parameterization (not something you set yourself).
- By default, the intercepts of the outcome variables at each occasion are fixed at zero; the means and variances of the growth factors are estimated; and the growth factor covariance is estimated (because the growth factors are exogenous/independent variables).
- By default, residual variances of the outcome variables are estimated and allowed to differ across occasions, and the residuals are **not** correlated across occasions.
- Quadratic model (`i s q |`): requires three random effects/growth factors (intercept, linear slope, quadratic slope). All three growth factor means, variances, and covariances are estimated and correlated by default, same exogenous-variable logic as above.
- Estimated time scores: for identification of a two-factor (intercept + slope) growth model, at least two time scores must remain fixed; in the example the first two are fixed (0 and 1) and the last two are freed with starting values of 2 and 3.
- Default estimator for these continuous-outcome growth models is maximum likelihood; `ANALYSIS: ESTIMATOR = ...;` can select a different one.

## Post-run operations
- Check the mean of `s` (the slope factor) — a significant mean indicates average growth/decline over time.
- Check the variance of `s` — a significant variance indicates individual differences in the rate of change (a random slope).
- Check the `i` WITH `s` covariance (estimated by default) — indicates whether higher starting levels are associated with faster/slower subsequent growth.
- For the quadratic model, check the mean/variance of `q` to determine whether curvature is present and whether it differs across individuals.
- For the estimated-time-scores model, inspect the estimated free time scores to see the data-implied spacing/shape of the trajectory.
- Add `OUTPUT: STDYX;` for standardized growth factor loadings/coefficients if needed.

## Likely FAQ mapping
- "How do I fit a basic latent growth curve model in Mplus?" → this file (Example 6.1)
- "My measurement occasions aren't evenly spaced, or I don't want to assume a shape" → estimated time scores (Example 6.8)
- "The trajectory looks curved, not a straight line" → quadratic growth model (Example 6.9)
- "I want to add a predictor of the trajectory, or my covariate is measured every wave" → see `growth-covariates-piecewise.md`
- "Development seems to change phase partway through" → piecewise growth, see `growth-covariates-piecewise.md`
- "My outcome is categorical/censored/count instead of continuous" → not yet covered by a reference file; Chapter 6 Examples 6.2–6.7 and 6.15 address these variable types and can be added on request
- "My outcome is time until a single event" → see `survival-analysis-discrete-continuous-time.md`

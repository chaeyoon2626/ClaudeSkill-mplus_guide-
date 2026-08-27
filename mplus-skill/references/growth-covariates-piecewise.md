# Growth Modeling — Time-Invariant/Time-Varying Covariates, Piecewise Growth, and Individually-Varying Observation Times

> Source: Mplus User's Guide v8, Chapter 6, Examples 6.10, 6.11, 6.12
> Bundled source: references/source-pdfs/Chapter6.pdf

## One-line summary
Extends a basic linear growth model (see `growth-linear-basic.md`) by adding covariates that predict the trajectory, by splitting growth into two linear phases (piecewise), and/or by letting each individual's observation times and the effect of a time-varying covariate differ across people.

## Prerequisite checklist
- [ ] A baseline linear growth model (intercept `i`, slope `s`) is already specified — see `growth-linear-basic.md`
- [ ] Distinguish time-invariant covariates (TIC; measured once per person, e.g. `x1`, `x2`) from time-varying covariates (TVC; measured at every occasion, e.g. `a31`–`a34`)
- [ ] Decide whether development has two distinguishable linear phases (needs a piecewise model) or a single overall linear trend
- [ ] Determine whether individuals were actually measured at the same fixed occasions or at individually-varying times/ages (needs `TSCORES` + `AT`)
- [ ] If a time-varying covariate's effect on the outcome is believed to differ across individuals (a random slope), confirm `TYPE = RANDOM;` is acceptable for the analysis

## Option selection logic
| Situation | Choice |
|---|---|
| Covariate measured once per person, predicts overall level and/or rate of growth | Time-invariant covariate: `i s ON x1 x2;` (Example 6.10) |
| Covariate measured at every occasion, predicts the outcome at that same occasion | Time-varying covariate: one `ON` statement per occasion, e.g. `y11 ON a31;` (Example 6.10) |
| Growth has two distinguishable linear phases (e.g., before/after some turning point) | Piecewise growth: two `\|` statements that share one intercept factor and each define their own slope factor over the relevant occasions (Example 6.11) |
| Individuals were measured at different actual times, but the TVC's effect on the outcome is assumed constant across people | `AT` option in the `\|` statement plus `TSCORES` in VARIABLE; TVC effects specified with plain `ON` statements |
| Individuals were measured at different actual times, and the TVC's effect on the outcome is assumed to vary across people | Same as above, plus `ANALYSIS: TYPE = RANDOM;` and a `\|` statement using `ON` to define a random slope for the TVC effect (Example 6.12) |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE:**
   - `NAMES ARE ...;`
   - `TSCORES = a11-a14;` (only needed for individually-varying times of observation)
4. **ANALYSIS:** `TYPE = RANDOM;` (only if a random slope for a time-varying covariate is being defined)
5. **MODEL:**
   - Baseline growth: `i s | y11@0 y12@1 y13@2 y14@3;`
   - Time-invariant covariates: `i s ON x1 x2;`
   - Time-varying covariates: `y11 ON a31; y12 ON a32; y13 ON a33; y14 ON a34;`
   - Piecewise growth (two phases sharing one intercept `i`): `i s1 | y1@0 y2@1 y3@2 y4@2 y5@2; i s2 | y1@0 y2@0 y3@0 y4@1 y5@2;`
   - Individually-varying times + random TVC slope: `i s | y1-y4 AT a11-a14; st | y1 ON a21; st | y2 ON a22; st | y3 ON a23; st | y4 ON a24; i s st ON x;`

## Sub-option details
- `i s ON x1 x2;` regresses both growth factors on the time-invariant covariates in a single statement (equivalent to writing `i ON x1 x2; s ON x1 x2;` separately).
- Time-varying covariate effects must be written one occasion at a time: `y11 ON a31;`, `y12 ON a32;`, etc., each describing the linear regression of the outcome on the covariate measured at that same occasion.
- **Piecewise growth**: only one intercept factor `i` is named, and it must appear (identically) in both `\|` statements. The first `\|` statement defines the slope factor for phase 1 (time scores increase, e.g. 0,1,2, across the phase-1 occasions and stay flat afterward); the second `\|` statement defines the slope factor for phase 2 (time scores stay flat during phase 1 and increase, e.g. 0,1,2, during phase 2). By default the intercepts of the outcomes are fixed at zero; the means and variances of all growth factors (intercept + both slopes) are estimated and the growth factors are correlated with each other as default, since they are independent (exogenous) variables.
- `TSCORES = a11-a14;` (VARIABLE command) identifies the variables in the data set that hold each individual's actual observed time value at each occasion.
- `AT` on the right-hand side of a `\|` statement (e.g., `i s | y1-y4 AT a11-a14;`) replaces fixed/free numeric time scores with the individually-varying times taken from the `TSCORES` variables.
- `ANALYSIS: TYPE = RANDOM;` is required whenever a random slope (an effect that is allowed to vary across individuals) is being defined with a `\|`/`ON` combination.
- A random slope for a time-varying covariate's effect is defined by repeating the **same** slope name across occasion-specific `\|`/`ON` statements, e.g. `st | y1 ON a21; st | y2 ON a22; st | y3 ON a23; st | y4 ON a24;` — Mplus treats all four `st |` statements as defining one random-slope variable `st` (one regression coefficient per occasion is estimated as a realization of the same random effect).
- `i s st ON x;` regresses the intercept, slope, and the random TVC-slope on a time-invariant covariate `x` in one statement (this is how you explain individual differences in the random TVC effect itself).
- Default estimator for the covariate/piecewise growth models is maximum likelihood; for the individually-varying-times/random-slope model it is maximum likelihood with robust standard errors. `ANALYSIS: ESTIMATOR = ...;` can select a different one.

## Post-run operations
- A significant `i ON x1` or `s ON x1` path means the time-invariant covariate `x1` explains individual differences in initial level or rate of growth, respectively.
- A significant `y1t ON a3t` path means the occasion-specific covariate affects the outcome at that same occasion, controlling for the growth trajectory.
- For piecewise growth, compare the means/variances of `s1` and `s2` to characterize the two developmental phases separately (e.g., faster growth pre- vs. post-turning-point).
- For the random-slope TVC model, check the variance of `st` — a significant variance means the TVC's effect on the outcome differs across individuals; check `st ON x` to see whether `x` explains that variation.
- Add `OUTPUT: STDYX;` for standardized coefficients as usual.

## Likely FAQ mapping
- "I want to add a predictor that explains why people start higher or grow faster" → time-invariant covariate on `i` and/or `s`
- "I have a covariate that was measured at every wave" → time-varying covariate, one `ON` statement per occasion
- "Growth looks like it changes rate/direction partway through the study" → piecewise growth with a shared intercept factor
- "My participants weren't all measured at the same ages/times" → individually-varying times of observation (`AT` + `TSCORES`)
- "I think the effect of my time-varying covariate itself differs by person" → random slope + `TYPE = RANDOM;`
- "I just need a plain growth model with no covariates yet" → see `growth-linear-basic.md`
- "My outcome is time until a single event, not a repeated continuous measure" → see `survival-analysis-discrete-continuous-time.md`

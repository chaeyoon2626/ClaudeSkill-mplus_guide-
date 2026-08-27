# Survival Analysis — Discrete-Time and Continuous-Time (Cox and Parametric Proportional Hazards)

> Source: Mplus User's Guide v8, Chapter 6, Examples 6.19, 6.20, 6.21, 6.22
> Bundled source: references/source-pdfs/Chapter6.pdf

## One-line summary
Models the time until a single, non-repeatable event occurs, either as a sequence of period-specific 0/1 event indicators (discrete-time) or as a continuous time-to-event variable analyzed with a Cox regression or a parametric proportional hazards model (continuous-time), optionally with covariates and/or a latent factor predicting event timing.

## Prerequisite checklist
- [ ] Confirm the event of interest is single and non-repeatable (it happens once, or not at all, per individual)
- [ ] Decide whether the event timing was recorded infrequently at a small number of periods (e.g., monthly/annually) — discrete-time — or frequently/exactly (e.g., days, hours) — continuous-time
- [ ] Identify the right-censoring indicator variable and confirm its coding (which value means "censored")
- [ ] Identify any covariate(s) that should predict event timing
- [ ] For continuous-time analyses, decide between a semi-parametric Cox regression (no assumption about the shape of the baseline hazard) and a parametric proportional hazards model (baseline hazard itself estimated over specified intervals)

## Option selection logic
| Situation | Choice |
|---|---|
| Event status recorded at a small number of discrete periods (e.g., annually) as a 0/1 indicator per period, missing once the event has occurred or the person drops out | Discrete-time survival: binary period indicators, `CATEGORICAL =`, `f BY u1-u4@1;`, `f@0;` (Example 6.19) |
| Event time recorded finely (continuous time-to-event) and no assumption is wanted about the shape of the baseline hazard | Continuous-time Cox regression: `SURVIVAL =`, `TIMECENSORED =`, `t ON x;` (Example 6.20) |
| Continuous time-to-event, and the baseline hazard itself should be estimated as model parameters | Parametric proportional hazards: `SURVIVAL = t(#intervals*length);`, `ANALYSIS: BASEHAZARD = ON;`, `[t#1-t#K];` (Example 6.21) |
| Continuous-time survival, plus a latent factor (measured by other indicators) should also predict survival | Add `f BY u1-u4;`, `t ON x f;`, `f ON x;`, `ANALYSIS: ALGORITHM = INTEGRATION;` (Example 6.22) |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE:**
   - Discrete-time: `NAMES ARE u1-u4 x; CATEGORICAL = u1-u4; MISSING = ALL (999);`
   - Continuous-time: `NAMES = t x tc; SURVIVAL = t;` (or `SURVIVAL = t (20*1);` for a parametric model with 20 intervals of length 1) `TIMECENSORED = tc (0 = NOT 1 = RIGHT);`
4. **ANALYSIS:** (as needed) `ESTIMATOR = MLR;` / `BASEHAZARD = ON;` / `ALGORITHM = INTEGRATION;`
5. **MODEL:**
   - Discrete-time: `f BY u1-u4@1; f ON x; f@0;`
   - Cox regression: `t ON x;`
   - Parametric proportional hazards: `[t#1-t#21]; t ON x;`
   - Parametric proportional hazards with a factor: `f BY u1-u4; [t#1-t#21]; t ON x f; f ON x;`

## Sub-option details
- **Discrete-time (Example 6.19)**: each `u` variable indicates whether the event occurred in that specific time period (1 = occurred, 0 = did not occur); a missing value flag means the event occurred in a preceding period or the individual dropped out of the study. `f BY u1-u4@1;` fixes all factor loadings to 1, representing a proportional-odds assumption for the hazards across periods. `f ON x;` regresses the survival factor on the covariate. `f@0;` fixes the factor's residual variance to zero, which corresponds to a conventional discrete-time survival model. The default estimator for this type is a robust weighted least squares estimator; specifying `ESTIMATOR = MLR;` switches to maximum likelihood with robust standard errors using numerical integration.
- **`SURVIVAL` option (VARIABLE command)**: identifies the variable holding time-to-event information, and — for parametric models — the number and length of the time intervals used for the baseline hazard function, e.g. `t (20*1)` specifies 20 intervals of length 1.
- **`TIMECENSORED` option (VARIABLE command)**: identifies the variable containing right-censoring information; it must be used together with `SURVIVAL`. `(0 = NOT 1 = RIGHT)` specifies that 0 means no censoring and 1 means right censoring — this is the Mplus default coding.
- **Cox regression (Example 6.20)**: `t ON x;` describes the loglinear regression of the time-to-event variable on the covariate. Estimation uses the profile likelihood method. The default estimator is maximum likelihood with robust standard errors; `ANALYSIS: ESTIMATOR = ...;` can select a different one.
- **`BASEHAZARD = ON;` (ANALYSIS command, Example 6.21)**: used with continuous-time survival analysis to treat the baseline hazard parameters as explicit model parameters rather than auxiliary parameters. There are as many baseline hazard parameters as there are time intervals plus one. They are referred to in the MODEL command by appending `#` and a number to the time-to-event variable's name (e.g., `t#1`, `t#2`, ...), and must be listed in a bracket statement such as `[t#1-t#21];` to be included as free parameters.
- **`ALGORITHM = INTEGRATION;` (ANALYSIS command, Example 6.22)**: invokes a maximum likelihood estimator with robust standard errors using numerical integration, needed when a latent factor is also part of the survival model. Numerical integration becomes more computationally demanding as the number of factors and the sample size increase.
- **Factor influencing survival (Example 6.22)**: `f BY u1-u4;` measures a latent factor from indicators `u1`–`u4`; `t ON x f;` regresses the time-to-event variable on both the covariate `x` and the factor `f`; `f ON x;` regresses the factor on `x`, allowing `x`'s effect on survival to work partly through the factor.

## Post-run operations
- Discrete-time: examine the `f ON x` coefficient — describes the covariate's effect on the (proportional-odds) hazard across all periods.
- Continuous-time (Cox or parametric): examine the `t ON x` coefficient — its sign/magnitude describes how the covariate accelerates or decelerates the time to the event.
- Parametric proportional hazards: inspect the estimated baseline hazard parameters (`t#1`, `t#2`, ...) to see the shape of the hazard over the specified intervals.
- Parametric proportional hazards with a factor: check both `f ON x` and `t ON f` to determine whether `x`'s effect on survival is partly indirect, transmitted through the factor.
- `OUTPUT: TECH1 TECH8;` is commonly requested in these examples — `TECH1` prints parameter specifications/starting values, `TECH8` prints the optimization history (useful for gauging run time).

## Likely FAQ mapping
- "My outcome is the time until a single event happens (dropout, relapse, death, etc.)" → this file
- "I only have periodic (e.g., monthly/annual) check-ins for whether the event happened yet" → discrete-time survival (Example 6.19)
- "I have exact/continuous event times and don't want to assume a shape for the hazard" → Cox regression (Example 6.20)
- "I want the baseline hazard function itself estimated as parameters" → parametric proportional hazards (Example 6.21)
- "I also have a latent factor that should help explain survival" → parametric proportional hazards with a factor influencing survival (Example 6.22)
- "My outcome is a repeated continuous/categorical measurement over time, not a single event" → see `growth-linear-basic.md` and `growth-covariates-piecewise.md`

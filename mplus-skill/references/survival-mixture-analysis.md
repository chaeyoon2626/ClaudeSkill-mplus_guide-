# Survival Mixture Analysis — Discrete-Time and Continuous-Time (Cox) Survival Combined with Latent Classes

> Source: Mplus User's Guide v8, Chapter 8, Examples 8.16, 8.17

## One-line summary
Combines survival analysis (time to a single, non-repeatable event; see `survival-analysis-discrete-continuous-time.md`) with an unobserved categorical latent class variable under `TYPE = MIXTURE`, so that either survival is predicted by growth-trajectory class membership (discrete-time survival mixture analysis) or survival itself, together with a covariate, helps define the latent classes (continuous-time/Cox survival mixture analysis).

## Prerequisite checklist
- [ ] Confirm the event of interest is single and non-repeatable, exactly as in ordinary survival analysis
- [ ] Decide whether the survival mixture model should have the latent classes come from a separate growth process that survival is then regressed on (discrete-time case), or whether the latent classes are defined directly by the survival process and a covariate (continuous-time/Cox case)
- [ ] For the discrete-time case: identify the repeated growth outcome (e.g., `y1-y3`) that will define the trajectory classes, and the period-specific binary event indicators (e.g., `u1-u4`)
- [ ] For the continuous-time case: identify the time-to-event variable, its right-censoring indicator, and any covariate(s) that should predict both survival and class membership
- [ ] Familiarity with basic mixture-model mechanics (`TYPE = MIXTURE`, `CLASSES =`, `%OVERALL%` / class-specific blocks) — see `growth-mixture-modeling-lcga.md` or `mixture-lpa-lca-cross-sectional.md` for the general logic

## Option selection logic
| Situation | Choice |
|---|---|
| Repeated period-specific event indicators (discrete-time survival), and interest is in how latent growth-trajectory classes from a separate repeated outcome predict the timing of the event | Discrete-time survival mixture analysis: growth model (`i s |`) defines the classes, a separate `f BY u1-u4@1;` proportional-odds factor represents the discrete-time hazard, class membership predicts the growth factors (Example 8.16) |
| Continuous/exact time-to-event data, and the latent classes themselves should be shaped jointly by survival time and a covariate | Continuous-time survival mixture analysis: `SURVIVAL =`, `TIMECENSORED =`, class-varying `t ON x;` and class-varying category indicator thresholds under `TYPE = MIXTURE` (Example 8.17) |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE:**
   - Discrete-time: `NAMES ARE y1-y3 u1-u4; CLASSES = c (2); CATEGORICAL = u1-u4; MISSING = u1-u4 (999);`
   - Continuous-time (Cox): `NAMES = t u1-u5 x tc; CLASSES = c (2); CATEGORICAL = u1-u5; SURVIVAL = t (ALL); TIMECENSORED = tc (0 = NOT 1 = RIGHT);`
4. **ANALYSIS:** `TYPE = MIXTURE;`
5. **MODEL:**
   - Discrete-time: `%OVERALL% i s | y1@0 y2@1 y3@2; f BY u1-u4@1;`
   - Continuous-time: `%OVERALL% t ON x; c ON x; %c#1% [u1$1-u5$1]; t ON x; %c#2% [u1$1-u5$1]; t ON x;`
6. **OUTPUT:** `TECH1 TECH8;`

## Sub-option details
- **Discrete-time survival mixture analysis (Example 8.16)**: `i s | y1@0 y2@1 y3@2;` defines an ordinary linear growth model (as in `growth-mixture-modeling-lcga.md`) for the repeated outcome `y1-y3`, and its growth factors are allowed to differ across the classes of `c` by default (their means are not held equal across class as the default; their variances/covariance are held equal across class as the default). Separately, `f BY u1-u4@1;` defines a discrete-time survival factor exactly as in ordinary discrete-time survival analysis (see `survival-analysis-discrete-continuous-time.md`), with all loadings fixed at 1 to represent a proportional-odds assumption; each `u` variable is 1 if the event occurred in that period and 0 otherwise, with a missing-value flag once the event has occurred or the individual has dropped out. The mean of `f` is fixed at zero in the last class by default, and `f`'s variance is fixed at zero in every class, matching a conventional discrete-time survival model. Because the growth factors `i`/`s` differ by class while `f` does not, survival is effectively predicted by which trajectory class an individual belongs to.
- **`CATEGORICAL = u1-u4;` / `MISSING = u1-u4 (999);`**: as in ordinary discrete-time survival analysis, the event indicators must be declared categorical, and a missing-value flag is used to represent "event already occurred, or dropped out" in later periods.
- **Continuous-time (Cox) survival mixture analysis (Example 8.17)**: `SURVIVAL = t (ALL);` identifies the time-to-event variable and requests that the time intervals used for the (non-parametric) baseline hazard be taken directly from the data (the `ALL` keyword) rather than specified as a fixed count/length. `TIMECENSORED = tc (0 = NOT 1 = RIGHT);` identifies the right-censoring variable, same convention as in ordinary continuous-time survival analysis. In `%OVERALL%`, `t ON x;` regresses time-to-event on a covariate `x`, and `c ON x;` regresses class membership on the same covariate via multinomial logistic regression. Inside each class-specific block (`%c#1%`, `%c#2%`), the thresholds of the categorical indicators `u1-u5` (e.g., `[u1$1-u5$1];`) and the `t ON x;` regression are each re-specified, which lets both the measurement thresholds and the covariate's effect on survival vary freely across latent classes — this is what allows survival information itself to help define/separate the classes. The non-parametric baseline hazard function is allowed to vary across class by default. Estimation uses the profile likelihood method, matching plain Cox regression.
- Both examples use the standard mixture-model defaults for `%OVERALL%` vs. class-specific blocks: parameters placed in `%OVERALL%` are common across classes unless overridden inside a `%c#k%` block.
- Default estimator for both example types is maximum likelihood with robust standard errors; `ANALYSIS: ESTIMATOR = ...;` can select a different one.

## Post-run operations
- Confirm "MODEL ESTIMATION TERMINATED NORMALLY" and check the loglikelihood was replicated across random starts (TECH8), exactly as in other mixture models — survival mixture models are as prone to local optima as any mixture model.
- Discrete-time case: compare the growth factor means (`i`, `s`) across classes to characterize each trajectory class, then relate class membership to the discrete-time hazard factor `f` to describe how trajectory shape relates to event timing.
- Continuous-time (Cox) case: compare the class-specific `t ON x` coefficients to see whether the covariate's effect on survival differs by latent class, and compare class-specific thresholds to characterize how the indicators differ across classes.
- Apply the same class-enumeration workflow (BIC, entropy, class size/interpretability) used for other mixture models — see `mixture-lpa-lca-cross-sectional.md` and `growth-mixture-modeling-lcga.md` for the general logic, which applies here as well.
- `OUTPUT: TECH1 TECH8;` is the standard request pair in both examples.

## Likely FAQ mapping
- "I want to see whether latent trajectory classes from a growth model predict event timing (survival)" → discrete-time survival mixture analysis (Example 8.16)
- "I have continuous/exact time-to-event data and want unobserved classes that reflect differences in survival itself" → continuous-time survival mixture analysis using a Cox regression model (Example 8.17)
- "I just need plain (non-mixture) survival analysis" → see `survival-analysis-discrete-continuous-time.md`
- "I want a growth mixture model without any survival/event-time component" → see `growth-mixture-modeling-lcga.md`
- "How do I decide on the number of latent classes?" → same class-enumeration logic as any mixture model; see `mixture-lpa-lca-cross-sectional.md`

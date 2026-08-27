# Growth Modeling for Censored, Categorical, and Count Outcomes

> Source: Mplus User's Guide v8, Chapter 6, Examples 6.2, 6.3, 6.4, 6.5, 6.6, 6.7

## One-line summary
Extends the basic linear growth model (see `growth-linear-basic.md`) to outcomes with a floor/ceiling effect (censored, or censored-inflated), binary/ordinal outcomes (categorical, with a choice of Delta or Theta parameterization), and count outcomes (Poisson, or zero-inflated Poisson) — all using the same `|` growth-factor syntax, but declaring the outcome's measurement level in the VARIABLE command.

## Prerequisite checklist
- [ ] A baseline linear growth model (intercept `i`, slope `s`) already fits the general design — see `growth-linear-basic.md`
- [ ] Determine the outcome's true measurement type: continuous with a floor/ceiling (censored), binary/ordinal (categorical), or a count (Poisson-distributed)
- [ ] For a censored outcome, decide whether a plain censored (Tobit-type) model is enough, or whether there is also excess mass exactly at the censoring point that needs its own sub-model (censored-inflated)
- [ ] For a count outcome, decide whether there is excess mass at zero beyond what a Poisson distribution predicts (zero-inflated Poisson) or a plain Poisson model suffices
- [ ] For a categorical outcome, be aware numerical integration is required if a maximum-likelihood-based estimator is used instead of the weighted-least-squares default
- [ ] Confirm the repeated outcome variables are named consistently across occasions (e.g., `y11-y14` or `u11-u14`) so list shorthand (`-`) can be used

## Option selection logic
| Situation | Choice |
|---|---|
| Continuous outcome with a floor or ceiling effect (many scores piled at one boundary) | `CENSORED = y11-y14 (b);` for a floor effect (`a` for ceiling) — a censored regression/Tobit-type growth model (Example 6.2) |
| Floor/ceiling effect, and the probability of being stuck at the boundary is itself worth modeling separately from the continuous part | `CENSORED = y11-y14 (bi);` — a censored-inflated model with two growth processes, one for the continuous part and one for the binary "at-the-limit" part (Example 6.3) |
| Binary or ordered-categorical (ordinal) repeated outcome, default weighted least squares estimation acceptable | `CATEGORICAL = u11-u14;` with the default (Delta) parameterization (Example 6.4) |
| Binary/ordinal outcome, but residual variances of the latent response variable (rather than scale factors) are the quantities of interest | `CATEGORICAL = u11-u14;` plus `ANALYSIS: PARAMETERIZATION = THETA;` (Example 6.5) |
| Count outcome (e.g., number of events), no excess zeros beyond what Poisson predicts | `COUNT = u11-u14;` — Poisson growth model (Example 6.6) |
| Count outcome with more zeros than a Poisson distribution would predict | `COUNT = u11-u14 (i);` — zero-inflated Poisson growth model, with a second `\|` statement for the always-zero/inflation part (Example 6.7) |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE:**
   - `NAMES ARE y11-y14 x1 x2 x31-x34;`
   - `USEVARIABLES ARE y11-y14;`
   - Censored: `CENSORED = y11-y14 (b);` or `(bi)` for censored-inflated
   - Categorical: `CATEGORICAL = u11-u14;`
   - Count: `COUNT = u11-u14;` or `COUNT = u11-u14 (i);` for zero-inflated
4. **ANALYSIS:**
   - Censored: `ESTIMATOR = MLR;` (switches from the default robust weighted least squares to maximum likelihood with numerical integration)
   - Censored-inflated: `INTEGRATION = 7;` (reduces integration points per dimension from the default 15, since three factors already give many integration points)
   - Categorical/Theta: `PARAMETERIZATION = THETA;`
   - Count (Poisson or ZIP): numerical integration is used automatically under the default ML-type estimator for count outcomes
5. **MODEL:**
   - Censored/categorical/Poisson (single growth process): `i s | y11@0 y12@1 y13@2 y14@3;`
   - Censored-inflated: `i s | y11@0 y12@1 y13@2 y14@3; ii si | y11#1@0 y12#1@1 y13#1@2 y14#1@3; si@0;`
   - Zero-inflated Poisson: `i s | u11@0 u12@1 u13@2 u14@3; ii si | u11#1@0 u12#1@1 u13#1@2 u14#1@3; s@0; si@0;`
6. **OUTPUT:** `TECH1 TECH8;` (as in most Chapter 6 examples)

## Sub-option details
- `CENSORED = y11-y14 (b);` (VARIABLE command): declares the listed variables censored; `b` = censored from below (floor effect), `a` = censored from above (ceiling effect); the censoring limit is taken from the data. Default estimator is robust weighted least squares; `ESTIMATOR = MLR;` switches to maximum-likelihood-with-robust-SE using numerical integration (more computationally demanding as the number of growth factors/sample size grows).
- `CENSORED = y11-y14 (bi);`: `bi` requests a **censored-inflated** model — two growth processes are estimated simultaneously. The first `\|` statement models the continuous part of the outcome (values at or above the censoring point); the second `\|` statement models the binary inflation part (probability of being unable to take any value except the censoring point). The inflation part's variables are referenced by appending `#1` to the censored variable's name (e.g., `y11#1`). By default: for the continuous part, growth-factor means/variances/covariance are estimated as usual (exogenous variables); for the inflation part, the intercept growth factor's mean is fixed at zero, the slope factor's mean and both factors' variances are estimated, and their covariance is estimated. Fixing the inflation slope's variance (`si@0;`) is common for parsimony/stabilization — when done, all of `si`'s covariances with other growth factors are fixed at zero by default, while the remaining covariances stay free.
- `CATEGORICAL = u11-u14;`: declares the listed outcomes binary or ordinal; thresholds (not means/intercepts) are estimated and, in a growth model, are held equal across time by default. With the default (robust) weighted least squares estimator, a probit model with the Delta parameterization is used: the scale factor of the latent response variable is fixed at 1 for the first occasion and freely estimated at later occasions. Switching to a maximum-likelihood estimator instead uses a logistic model with numerical integration (Hedeker & Gibbons, 1994).
- `ANALYSIS: PARAMETERIZATION = THETA;`: switches from the default Delta parameterization to Theta. Under Theta, the *residual variance* of the latent response variable is fixed at 1 for the first occasion and free thereafter, while scale factors are not separately estimated at all (the reverse of Delta's approach of fixing/freeing scale factors while residual variances are not separate parameters).
- `COUNT = u11-u14;`: declares the listed outcomes as counts, modeled with a Poisson distribution; default estimator is maximum likelihood with robust SEs and numerical integration.
- `COUNT = u11-u14 (i);`: `i` requests a **zero-inflated Poisson** model. As with the censored-inflated model, two `\|` statements are needed — one for the count part (values ≥ 0), one for the inflation part (probability of being unable to take any value except zero), with the inflation part's variables referenced via `#1` (e.g., `u11#1`). In the chapter's example, both the count-part slope (`s@0;`) and inflation-part slope (`si@0;`) variances are fixed at zero for simplicity; when both are fixed, their mutual covariances and covariances with other growth factors are fixed at zero by default, but the covariance between the two intercept factors (`i` and `ii`) remains free by default.
- Growth-factor default logic (applies across all these outcome types unless noted): outcome intercepts at each occasion are fixed at zero; growth-factor means and variances are estimated; growth-factor covariances are estimated by default because growth factors are exogenous (independent) variables.

## Post-run operations
- Censored models: interpret `s` (slope) mean/variance as in a continuous growth model, but remember coefficients describe the underlying (uncensored) latent continuous process.
- Censored-inflated models: check both the continuous-part slope (`s`) and the inflation-part slope (`si`, if freed) — the latter describes whether the probability of being "stuck" at the floor/ceiling changes over time.
- Categorical growth models: examine the slope growth factor `s` (Delta or Theta parameterization) for average change in the underlying propensity over time; compare Delta vs. Theta results only within their own parameterization since scale/residual variance are fixed differently.
- Count/Poisson models: interpret `i`/`s` on the log-count (Poisson link) scale; for zero-inflated Poisson, separately interpret the count-part growth factors and the inflation-part growth factors (probability of "always zero").
- `OUTPUT: TECH1 TECH8;` is used throughout these examples — `TECH1` for parameter specification/starting values, `TECH8` for optimization history (useful to gauge run time, especially with numerical integration).

## Likely FAQ mapping
- "My repeated outcome has a floor or ceiling effect (lots of scores at the minimum/maximum)" → censored growth model, `CENSORED = ... (b)` or `(a)` (Example 6.2)
- "Besides the floor effect, I think being stuck at the floor is its own process" → censored-inflated model (Example 6.3)
- "My repeated outcome is binary or ordinal (Likert-type)" → categorical growth model with Delta (default) or Theta parameterization (Examples 6.4, 6.5)
- "My repeated outcome is a count (e.g., number of symptoms/events)" → Poisson growth model (Example 6.6)
- "My count outcome has way more zeros than Poisson predicts" → zero-inflated Poisson growth model (Example 6.7)
- "I need a growth model for a censored/categorical/count outcome but also want latent trajectory classes" → see `growth-mixture-modeling-lcga.md`, which covers the same outcome types combined with `TYPE = MIXTURE`
- "I just need a plain continuous growth model with no special outcome type" → see `growth-linear-basic.md`
- "I need covariates, piecewise growth, or individually-varying times of observation" → see `growth-covariates-piecewise.md`

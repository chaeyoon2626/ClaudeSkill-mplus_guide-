# N=1 Time Series — Dynamic Factor Analysis and IRT Models

> Source: Mplus User's Guide v8, Chapter 6, Examples 6.26, 6.27, 6.28

## One-line summary
Extends N=1 (single-subject) autoregressive time series modeling to latent variables — an autoregressive AR(1) confirmatory factor / dynamic factor analysis (DAFS) model for continuous indicators, an alternative white-noise-factor-score (WNFS) parameterization, an AR(1) IRT-style model for binary indicators, and a bivariate cross-lagged two-factor model — all via Bayesian estimation.

## Prerequisite checklist
- [ ] Multiple indicators are measured repeatedly on the **same single individual** at each time point (e.g., items y1-y4 collected at every occasion), not just one observed variable — if only one observed variable is involved, see `time-series-n1-autoregressive-crosslagged.md`
- [ ] Decide whether the latent factor's own true score should follow the AR(1) process (dynamic factor / DAFS) or whether the observed indicators themselves should be regressed on the lagged factor (white noise factor score / WNFS) — these are competing theoretical parameterizations, not a strict either/or default
- [ ] Check whether the indicators are continuous (CFA-style parameterization) or binary/ordinal (IRT-style parameterization with `CATEGORICAL =`)
- [ ] Decide whether one latent process, or two separate latent processes (each with its own indicator set) that may mutually influence each other over time, is needed
- [ ] Confirm the shared N=1 Bayesian setup — `ESTIMATOR = BAYES`, `PROCESSORS =`, `BITERATIONS =` — is in place (see `time-series-n1-autoregressive-crosslagged.md` for the rationale of each)

## Option selection logic
| Situation | Choice |
|---|---|
| Continuous indicators; the latent factor's own score should follow an AR(1) process over time | Dynamic factor / DAFS model: `f BY y1-y4 (&1); f ON f&1;` (Example 6.26) |
| Continuous indicators; want the observed indicators (not a lagged factor score) regressed on the previous factor | White noise factor score (WNFS) model: `f BY y1-y4 (&1); y1-y4 ON f&1;` (Example 6.26, alternative) |
| Binary indicators; want an AR(1) latent-trait (IRT-style) model | `CATEGORICAL = u1-u4; f BY u1-u4*(&1); f@1; f ON f&1;` (Example 6.27) |
| Two distinct latent processes, each with its own continuous indicators, that may predict each other over time | Bivariate cross-lagged two-factor model: `f1 BY y11-y14(&1); f2 BY y21-y24(&1); f1 ON f1&1 f2&1; f2 ON f2&1 f1&1;` (Example 6.28) |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE:**
   - Continuous indicators: `NAMES = y1-y4;`
   - Binary indicators: `NAMES = u1-u4; CATEGORICAL = u1-u4;`
   - Two-factor version: `NAMES = y11-y14 y21-y24;`
4. **ANALYSIS:** `ESTIMATOR = BAYES; PROCESSORS = 2; BITERATIONS = (2000);`
5. **MODEL:**
   - DAFS: `f BY y1-y4 (&1); f ON f&1;`
   - WNFS alternative: `f BY y1-y4 (&1); y1-y4 ON f&1;`
   - IRT AR(1): `f BY u1-u4*(&1); f@1; f ON f&1;`
   - Two-factor cross-lagged: `f1 BY y11-y14(&1); f2 BY y21-y24(&1); f1 ON f1&1 f2&1; f2 ON f2&1 f1&1;`
6. **OUTPUT: TECH1 TECH8;**
7. **PLOT: TYPE = PLOT3;**

## Sub-option details
- `f BY y1-y4 (&1);` — measures a common factor `f` from indicators `y1`-`y4` as in ordinary CFA. The `(&1)` placed after the whole `BY` statement (not on an individual indicator) tells Mplus to make the lag-1 version of the *factor itself* — `f&1`, i.e., f at the previous time point — available for use elsewhere in `MODEL:`. This is the factor-level counterpart of the `VARIABLE: LAGGED =` option used for observed variables.
- Default factor metric: the first indicator's loading is fixed to 1 automatically, as in standard CFA; this default can be overridden (see the IRT example below).
- `f ON f&1;` — the dynamic factor / DAFS specification (Example 6.26): an AR(1) regression of the factor on its own lag-1 value, estimating a regression coefficient and a factor residual variance; the factor intercept is fixed at zero by default. Indicator intercepts and residual variances are estimated, with indicator residuals uncorrelated by default.
- `y1-y4 ON f&1;` — the white noise factor score (WNFS) alternative (Zhang & Nesseroade, 2007): instead of regressing the current factor on its own lag, each observed indicator is regressed directly on the lagged factor. This reflects a different substantive account of how the prior latent state carries forward into the present observed indicators, rather than being a strictly preferred default over DAFS.
- IRT AR(1) model (Example 6.27): `CATEGORICAL = u1-u4;` declares binary indicators. `f BY u1-u4*(&1);` frees the first loading with `*` (overriding the usual fixed-at-1 default) so that the factor's metric can instead be set by fixing its residual variance to 1 via `f@1;`. Thresholds for the indicators are estimated by default. `f ON f&1;` specifies the same AR(1) latent regression as in the continuous case.
- Bivariate cross-lagged two-factor model (Example 6.28): two separate `BY` statements, each with its own four indicators and its own `(&1)` lag tag — `f1 BY y11-y14(&1);` and `f2 BY y21-y24(&1);`. Two `ON` statements mirror the observed-variable cross-lagged model: `f1 ON f1&1 f2&1;` and `f2 ON f2&1 f1&1;`, estimating four regression coefficients (two autoregressive, two cross-lagged), two factor residual variances, and one residual covariance between the two factors. Factor intercepts are fixed at zero by default, matching the single-factor case.
- Shared Bayesian setup (`ESTIMATOR = BAYES; PROCESSORS = 2; BITERATIONS = (...)`) and `PLOT: TYPE = PLOT3;` follow the same logic as the observed-variable AR/cross-lagged models — see `time-series-n1-autoregressive-crosslagged.md` for the full explanation of each.

## Post-run operations
- Check the `f ON f&1` coefficient (or, for the two-factor model, `f1 ON f1&1`/`f2&1` and `f2 ON f2&1`/`f1&1`) for the strength and direction of the latent autoregressive or cross-lagged process, interpreted the same way as the observed-variable AR/VAR coefficients.
- Before trusting the dynamic (lag) parameters, confirm the measurement part looks sound: factor loadings (or thresholds, for the IRT model) should be sensible in magnitude and sign, as in any standard CFA/IRT check.
- Use the `PLOT3` trace plots, autocorrelation plots, and posterior distribution plots per parameter to check MCMC convergence and posterior quality, exactly as for the observed-variable models.
- When choosing between the DAFS and WNFS parameterizations, compare model fit and substantive plausibility for the data at hand — they represent different theoretical claims about how the past factor state feeds into the present, not a case where one is always correct.
- For the two-factor cross-lagged model, compare `f1 ON f2&1` against `f2 ON f1&1` to judge which latent process leads the other, and inspect the estimated residual covariance between the factors for contemporaneous association not explained by the lagged effects.

## Likely FAQ mapping
- "I want an N=1/single-subject model with a latent factor from multiple indicators, not just one observed variable" → dynamic factor / DAFS model, Example 6.26
- "What's the difference between the dynamic factor (DAFS) and white noise factor score (WNFS) models?" → see the WNFS row in Option selection logic and Sub-option details above
- "My repeated indicators in an N=1 design are binary/ordinal" → AR(1) IRT-style model, Example 6.27
- "I want two latent processes from one person's repeated indicator data that might predict each other over time" → bivariate cross-lagged two-factor model, Example 6.28
- "My N=1 time series variable(s) are directly observed, not indicators of a latent factor" → see `time-series-n1-autoregressive-crosslagged.md`
- "How do PROCESSORS, BITERATIONS, TECH8, and PSR convergence work for this kind of Bayesian model?" → see `time-series-n1-autoregressive-crosslagged.md` for the shared explanation, or `analysis-command-options.md` (Ch.16) for the full Bayesian technical option set

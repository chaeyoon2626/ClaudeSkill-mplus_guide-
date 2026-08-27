# N=1 Time Series — Autoregressive and Cross-Lagged Models for Observed Variables

> Source: Mplus User's Guide v8, Chapter 6, Examples 6.23, 6.24, 6.25

## One-line summary
Models a single individual's repeatedly measured observed variable(s) over many time points as an autoregressive AR(1)/AR(2) process, optionally with a time-varying covariate, or as a bivariate cross-lagged (first-order vector autoregressive, VAR(1)) process between two variables, using Bayesian estimation.

## Prerequisite checklist
- [ ] The data are repeated measurements of **one** individual/unit over many occasions (N=1 time series), not many individuals measured a few times each (that is growth modeling — see `growth-linear-basic.md`)
- [ ] The data file is "long": one record per time point, in time order (records must be ordered by time — Mplus does not reorder them)
- [ ] Decide the maximum lag of interest for each variable (does only the immediately preceding time point matter, AR(1), or do the two preceding points matter, AR(2)?)
- [ ] Decide whether one variable's own history (univariate AR) or the mutual influence between two variables over time (bivariate cross-lagged/VAR) is the question
- [ ] If a covariate is involved, decide whether both its current value and its lagged value should predict the outcome
- [ ] Confirm Bayesian estimation (`ESTIMATOR = BAYES`) will be used, and check whether multiple processors are available to speed up the two MCMC chains

## Option selection logic
| Situation | Choice |
|---|---|
| One continuous variable, only the immediately preceding time point matters | AR(1) model: `LAGGED = y(1);` and `y ON y&1;` (Example 6.23) |
| One continuous variable, the two preceding time points both matter | AR(2) model: `LAGGED = y(2);` and `y ON y&1 y&2;` (Example 6.23 extension) |
| One continuous variable, want to test whether only lag-2 (skipping lag-1) matters | Fix the lag-1 coefficient to zero: `y ON y&1@0 y&2;` (Example 6.23 extension) |
| One variable's own history plus a time-varying covariate's current and lagged values | `LAGGED = y(1) x(1);` and `y ON y&1 x x&1;` (Example 6.24) |
| Two variables measured on the same individual that may each predict the other over time | Bivariate cross-lagged / VAR(1): `LAGGED = y1(1) y2(1);` and `y1 ON y1&1 y2&1; y2 ON y2&1 y1&1;` (Example 6.25) |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE:**
   - `NAMES ARE y;` (univariate) or `NAMES ARE y x;` (with covariate) or `NAMES = y1 y2;` (bivariate cross-lagged)
   - `LAGGED = y(1);` (AR1) or `LAGGED = y(2);` (AR2) or `LAGGED = y(1) x(1);` (with covariate) or `LAGGED = y1(1) y2(1);` (cross-lagged)
4. **ANALYSIS:**
   - `ESTIMATOR = BAYES; PROCESSORS = 2; BITERATIONS = (2000);` (minimum iterations shown varies by example: 2000 for AR1/AR2, 1000 with a covariate, 500 for the bivariate model)
5. **MODEL:**
   - AR(1): `y ON y&1;`
   - AR(2): `y ON y&1 y&2;`
   - Lag-2-only: `y ON y&1@0 y&2;`
   - With covariate: `y ON y&1 x x&1;`
   - Bivariate cross-lagged: `y1 ON y1&1 y2&1; y2 ON y2&1 y1&1;`
6. **OUTPUT: TECH1 TECH8;**
7. **PLOT: TYPE = PLOT3;**

## Sub-option details
- `VARIABLE: LAGGED = y(1);` — declares, per observed variable, the maximum lag Mplus should make available for use in `MODEL:`. The data set contains one row per time point for the single subject; the number of "observations" is the number of time points. A lagged variable is referred to elsewhere by appending an ampersand and the lag number to the variable name (`y&1` = y one time point back, `y&2` = y two time points back).
- `LAGGED = y(2);` — requests up to lag 2 be available, needed for an AR(2) model (`y ON y&1 y&2;`), which estimates an intercept, two autoregressive coefficients, and a residual variance.
- `y ON y&1@0 y&2;` — a restricted AR(2) specification where the lag-1 coefficient is fixed at zero, leaving only the lag-2 predictor plus an intercept and residual variance to be estimated (useful for testing whether the lag-1 effect is needed at all).
- `LAGGED = y(1) x(1);` with `y ON y&1 x x&1;` (Example 6.24) — models y's own lag-1 value together with the covariate's current value (`x`) and its lag-1 value (`x&1`); an intercept, three regression coefficients, and a residual variance are estimated.
- Bivariate cross-lagged / VAR(1) (Example 6.25) — two simultaneous `ON` statements, one per outcome, each regressed on its own lag-1 value and the other variable's lag-1 value: `y1 ON y1&1 y2&1;` and `y2 ON y2&1 y1&1;`. Two intercepts, four regression coefficients (two autoregressive, two cross-lagged), two residual variances, and one residual covariance between y1 and y2 are estimated.
- `ANALYSIS: ESTIMATOR = BAYES;` — required for this style of N=1 time series analysis in these examples. By default, Bayesian estimation runs two independent Markov chain Monte Carlo (MCMC) chains.
- `PROCESSORS = 2;` — runs the two MCMC chains in parallel when multiple processors are available, reducing run time.
- `BITERATIONS = (2000);` — sets the minimum (the number in parentheses) and maximum number of iterations per MCMC chain used with the potential scale reduction (PSR) convergence criterion (Gelman & Rubin, 1992); the maximum defaults to 50,000 if not otherwise specified.
- `OUTPUT: TECH1 TECH8;` — `TECH1` prints parameter specifications and starting values; `TECH8` prints the MCMC optimization/iteration history (by default also to the screen during estimation), useful for tracking run time and PSR-based convergence.
- `PLOT: TYPE = PLOT3;` — requests time-series-specific graphics viewable afterward in the post-processing graphics module: trace plots and autocorrelation plots of the MCMC draws per parameter (for judging chain convergence and posterior sample quality), full posterior distribution plots per parameter, plus time series plots of the observed data, and autocorrelation/partial autocorrelation plots of the observed data at different lags.

## Post-run operations
- Watch the `TECH8` screen output / saved output for PSR values stabilizing near 1 as an MCMC convergence check; raise the `BITERATIONS` minimum and rerun if PSR has not stabilized.
- Open the `PLOT3` trace plots for each estimated parameter to visually confirm that the two MCMC chains are well mixed and have converged to the same region.
- Open the `PLOT3` autocorrelation plots of the parameter draws to check how independent the posterior samples are (high autocorrelation across draws suggests needing more iterations or thinning).
- Inspect the `PLOT3` posterior distribution plot for each parameter's shape (e.g., is it roughly symmetric, or skewed/bounded).
- Use the `PLOT3` observed-data time series plot, plus its autocorrelation and partial autocorrelation functions at different lags, as an exploratory step to help choose an AR order before — or sanity-check the order after — fitting the model.
- Examine the `y ON y&1` coefficient's magnitude and credibility interval: values near 1 indicate a highly persistent, slowly decaying process, while values near 0 indicate weak carryover from one time point to the next.
- For the bivariate cross-lagged model, compare `y1 ON y2&1` against `y2 ON y1&1` to judge which variable's past more strongly predicts the other variable's present, and inspect the estimated residual covariance for contemporaneous (same-time-point) association not explained by the lagged terms.

## Likely FAQ mapping
- "I have one person/unit measured repeatedly over time and want an autoregressive model" → univariate AR(1)/AR(2), `LAGGED =` + `y ON y&1 (y&2);` (Example 6.23)
- "How do I add a time-varying predictor to my single-subject AR model?" → Example 6.24, `y ON y&1 x x&1;`
- "I have two variables from the same person over time and want to know if they predict each other (cross-lagged panel model for N=1)" → bivariate cross-lagged/VAR(1), Example 6.25
- "My repeated-measures data are from many different people, not a single individual" → not this file; see `growth-linear-basic.md` or `growth-covariates-piecewise.md`
- "I want a latent factor (not just an observed variable) to follow an autoregressive process over time" → see `time-series-n1-dynamic-factor-irt.md`
- "How do BITERATIONS, PROCESSORS, TECH8, and PSR convergence work in general for Bayesian estimation?" → this file covers the N=1 time series usage pattern; for the full technical option set see `analysis-command-options.md` (Ch.16) or `bayesian-estimation-plausible-values.md`

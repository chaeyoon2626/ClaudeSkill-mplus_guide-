# Categorical Mediating Variables & Converting Probit/Logistic Coefficients to Probabilities

> Source: Mplus User's Guide v8, Chapter 14 (Special Modeling Issues) — this chapter contains no numbered Examples; it is a technical-discussion chapter. Sections used: Categorical Mediating Variables, Calculating Probabilities from Probit Regression Coefficients, Calculating Probabilities from Logistic Regression Coefficients

## One-line summary
Explains how a categorical mediator `u` (in `x -> u -> y`) is actually represented differently depending on the estimator (WLS uses the latent response variable `u*` as the covariate for `y`, ML uses the observed `u`, Bayes lets you pick via the `MEDIATOR` option); and — as **post-run, by-hand interpretation guidance rather than Mplus syntax** — how to turn probit or logistic regression coefficients printed in Mplus output into probabilities, log odds, odds, odds ratios, and (for a nominal/multinomial outcome, including a categorical latent class variable) class probabilities.

## Prerequisite checklist
- [ ] For the mediator question: which estimator is being used (WLS, ML, or Bayes)? The treatment of `u` inside the `y ON u` regression depends on this.
- [ ] For the mediator question: is `u` binary or ordered categorical, and is it declared with `CATEGORICAL =` in `VARIABLE`?
- [ ] For the probability-conversion question: do you have the printed regression coefficient(s) and, for a probit or a categorical-DV model, the threshold(s)/intercept(s) from the Mplus output in hand?
- [ ] Is the coefficient from a probit-link model (typically WLS/WLSMV, or ML/Bayes with `LINK=PROBIT`) or a logit-link model (typically ML with the default logistic link)?
- [ ] Is the outcome binary, ordered categorical (ordinal, same slope across categories), or unordered categorical/nominal (including a categorical latent class variable `c`, where the last class is the customary reference category)?
- [ ] At what covariate value(s) do you want the probability evaluated — this needs to be decided before doing the hand calculation.

## Option selection logic
| Situation | Choice |
|---|---|
| Categorical mediator `u`, WLS/WLSMV estimation | `u ON x` is a probit regression coefficient; in `y ON u`, the continuous latent response variable `u*` (not the observed `u`) is used as the covariate |
| Categorical mediator `u`, maximum likelihood estimation | `u ON x` is estimated as either a logistic or a probit regression coefficient (depending on the model's link); in `y ON u`, the **observed** variable `u` (not a latent response variable) is used as the covariate |
| Categorical mediator `u`, Bayesian estimation (`ESTIMATOR=BAYES`), default treatment | `u ON x` is a probit regression coefficient; in `y ON u`, either the observed `u` or the latent response variable `u*` can be used as the covariate |
| Categorical mediator `u`, Bayesian estimation, want explicit control over which representation of `u` is used as the covariate in `y ON u` | `ANALYSIS: MEDIATOR = OBSERVED | LATENT;` (the `MEDIATOR` option of the `ANALYSIS` command) |
| Have a probit-model binary-outcome coefficient + threshold, want a probability at a specific covariate value | Post-run hand calculation: `P(u=1|x) = F(-t + b*x)` using the standard normal CDF `F` |
| Have a probit-model ordered-categorical (3-category) coefficient + two thresholds | Post-run hand calculation using the three-category probit formulas (see Sub-option details) |
| Have a logistic-model binary-outcome coefficient, want a probability at a specific covariate value | Post-run hand calculation: `P(u=1|x) = exp(a+b*x) / (1+exp(a+b*x))` |
| Have a logistic-model binary-outcome coefficient, want the log odds / odds / odds ratio interpretation instead of a probability | `b` itself is the log-odds increase per unit increase in `x`; `exp(b)` is the corresponding odds increase (or odds ratio, for a unit/group comparison) |
| Have a multinomial logistic model (nominal DV with 3+ categories, or a categorical latent class variable predicted by covariates) and want class/category probabilities at specific covariate values | Post-run hand calculation: exponentiate each category's log odds (intercept + slope×covariate, with the reference category fixed at intercept=slope=0), then divide each by the sum across all categories (softmax) |
| Want the class-probability-by-covariate relationship plotted rather than hand-calculated point-by-point | `PLOT:` command + Mplus's post-processing graphics module (exports e.g. an EMF file with probability curves by covariate) |

## Menu path & screen fields
**Categorical mediator representation** — this is not a separate procedure, just a consequence of estimator choice plus (for Bayes) one `ANALYSIS` option:
1. **VARIABLE:** `CATEGORICAL = u;`
2. **ANALYSIS:** `ESTIMATOR = WLS | WLSMV | ML | MLR | BAYES;` — determines the default treatment of `u` inside `y ON u`
3. **ANALYSIS (Bayes only, optional):** `MEDIATOR = OBSERVED;` or `MEDIATOR = LATENT;` — overrides the default choice of observed-`u` vs. latent-`u*` as the covariate in `y ON u`
4. **MODEL:** `u ON x; y ON u;` (plus any direct `y ON x` path for the non-mediated component)

**Probability/odds conversion** — a manual calculation performed after reading Mplus's `MODEL RESULTS` output, not an Mplus command sequence:
1. Run the model with `u ON x;` (or, for a categorical latent variable, `c#k ON x;` for each non-reference class `k`) and any needed thresholds/intercepts
2. Locate, in `MODEL RESULTS`, the regression coefficient(s) `b` and, for binary/ordinal probit models, the threshold(s) `t`, or for multinomial models, the intercept(s) `a`
3. Plug the values into the matching formula from Sub-option details, at whatever covariate value(s) `x` are of interest
4. (Optional) Use `PLOT:` + the post-processing graphics module to visualize the resulting probability curve across a range of covariate values instead of computing single points by hand

## Sub-option details
### Categorical mediating variable treatment (`x -> u -> y`)
- **WLS**: `u ON x` is a probit regression coefficient. In `y ON u`, the continuous **latent response variable** `u*` underlying `u` is used as the covariate for `y` — not the observed `u`.
- **ML**: `u ON x` is estimated as either a logistic or a probit regression coefficient (the link depends on the model specification). In `y ON u`, the **observed** variable `u` is used as the covariate for `y` — not a latent response variable.
- **Bayes**: `u ON x` is a probit regression coefficient. In `y ON u`, either the observed `u` or the latent response variable `u*` can be used as the covariate — this choice is controlled explicitly with the `MEDIATOR` option of the `ANALYSIS` command.

### Calculating probabilities from probit regression coefficients
- Binary DV: `P(u=1|x) = F(a + b*x) = F(-t + b*x)`, where `F` is the standard normal cumulative distribution function, `a` is the probit intercept, `b` is the probit slope, and `t` is the probit threshold, related by `t = -a`. `P(u=0|x) = 1 - P(u=1|x)`.
  - Worked example from the manual: output shows `u ON age` coefficient `0.055` and threshold `u$1 = 3.581`. At `age = 62`: `P(u=1|age=62) = F(-3.581 + 0.055*62) = F(-0.171)`. Reading `-0.171` off a standard normal (z) table gives approximately `0.43`, i.e. the probability of `u=1` at age 62 is about `0.43`.
- Ordered categorical (ordinal) DV with 3 categories, thresholds `t1` and `t2`, single probit slope `b` (same slope across categories, as is standard for an ordinal probit model):
  - `P(u=0|x) = F(t1 - b*x)`
  - `P(u=1|x) = F(t2 - b*x) - F(t1 - b*x)`
  - `P(u=2|x) = F(-t2 + b*x)`

### Calculating probabilities from logistic regression coefficients
- An odds is a ratio of two probabilities; a log odds is the log of that ratio; exponentiating a log odds gives an odds. A logistic regression coefficient **is** a log odds, also called a logit.
- Binary DV: `P(u=1|x) = exp(a+b*x) / (1+exp(a+b*x)) = 1 / (1+exp(-a-b*x))`, `P(u=0|x) = 1 - P(u=1|x)`. This is algebraically equivalent to the linear logit expression `log[P(u=1|x)/P(u=0|x)] = a + b*x`.
- Interpreting `b` for a continuous covariate: comparing `x0` and `x0+1`, the log odds at `x0+1` minus the log odds at `x0` equals `b` exactly — i.e., `b` is the increase in the log odds of `u=1` vs. `u=0` per one-unit increase in `x`, and the corresponding **odds increase** is `exp(b)`. Worked example: a logistic coefficient of `.75` for age predicting depression (`u=1`) means one additional year of age increases the log odds of being depressed by `.75`; the corresponding odds increase is `exp(.75) ≈ 2.12`.
- Interpreting `b` for a binary 0/1 covariate `x` (e.g., gender): the difference in log odds between `x=1` and `x=0` is again exactly `b`, and this quantity is the **log odds ratio** for `u=1` vs. `u=0` comparing `x=1` to `x=0`; `exp(b)` is the corresponding **odds ratio**. Worked example: a coefficient of `1.0` for gender (1=female, 0=male) predicting depression means the log odds for females is 1.0 higher than for males; the odds ratio is `exp(1.0) ≈ 2.72`, i.e., the odds of being depressed are 2.72 times higher for females than for males.
- **Reference category convention**: for a binary DV, `u=0` is customarily the reference category (as used above). For a DV with more than two categories (nominal/unordered, or an ordered categorical treated via the ordinal logit slope), the **last** category is customarily the reference category.
- **Multinomial logistic regression** (nominal DV, or a categorical latent variable `c`, with R categories): generalizes the binary formula to
  `P(u=r|x) = exp(a_r + b_r*x) / [exp(a_1+b_1*x) + ... + exp(a_R+b_R*x)]`,
  where the reference category `R` has `a_R = 0` and `b_R = 0` by construction (so `exp(a_R+b_R*x)=1`), and the log odds of category `r` vs. the reference category `R` is `log[P(u=r|x)/P(u=R|x)] = a_r + b_r*x`.
- **Ordered categorical logistic slopes**: for an ordinal DV, the logistic regression slopes `b_r` are constrained to be the same across the categories of `u` (a single slope, multiple thresholds — analogous to the ordinal probit case above).
- **Worked multinomial example** (categorical latent variable `c` with 4 classes, predicted by 3 covariates `age94`, `male`, `black`; class 4 is the reference class, so its intercept and all its slopes are 0):
  1. *Intercepts-only probabilities* (all covariates = 0): compute each class's log odds vs. class 4 from the printed intercepts (`C#1`, `C#2`, `C#3`), exponentiate each, sum the exponentials, then divide each exponentiated value by the sum. In the manual's example this yields class probabilities of approximately `.069`, `.201`, `.307`, and `.424` for classes 1–4 respectively (columns sum to ~1, allowing for rounding).
  2. *Probabilities at a specific covariate profile* (all three covariates = 1): compute each class's log odds as intercept + sum of (slope × 1) over the three covariates, exponentiate, sum, and normalize the same way. In the manual's example this yields probabilities of approximately `.200`, `.036`, `.657`, and `.107` for classes 1–4.
  3. *Interpreting one slope directly as an odds ratio*: e.g., the `male` slope of `2.578` for `C#1 ON male` means that, holding the other covariates constant, the log odds of being in class 1 vs. class 4 is 2.578 higher for males than for females; the corresponding odds ratio for class 1 vs. class 4, comparing males to females, is `exp(2.578) ≈ 13.17`.
- **Visualizing instead of hand-computing**: the estimated probability-by-covariate relationship (e.g., class probability curves across a covariate like age) can be produced automatically and exported (e.g., as an EMF file) using the `PLOT` command together with Mplus's post-processing graphics module, instead of repeating the hand calculation at many covariate values.

## Post-run operations
- After reading off coefficients/thresholds from `MODEL RESULTS`, decide which formula applies (probit vs. logit, binary vs. ordinal vs. multinomial) before doing the by-hand calculation — using the wrong formula (e.g., treating a probit threshold like a logit intercept) will silently give a wrong probability.
- For a multinomial/latent-class result, always renormalize by summing the exponentiated log odds across **all** categories (including the reference category's value of `exp(0)=1`) — a probability computed from only one category's exponentiated log odds without the sum will not be a valid probability.
- Cross-check a hand-computed probability against the `PLOT` command's automatically generated probability curve where possible, as a sanity check on the arithmetic.
- For a categorical mediator model, after estimation check which representation (`u` vs. `u*`) was actually used for `y ON u` (governed by the estimator, or by `MEDIATOR=` for Bayes) before interpreting the mediated-effect coefficient, since the two representations are not on the same numeric scale.

## Likely FAQ mapping
- "I have `x -> u -> y` with `u` categorical — how does Mplus actually treat `u` inside the `y ON u` regression?" → this file (Categorical Mediating Variables: WLS/ML/Bayes differ; `MEDIATOR` option for Bayes)
- "What does the `MEDIATOR` option in `ANALYSIS` do?" → this file (Categorical Mediating Variables — Bayes-only, `OBSERVED` vs. `LATENT`)
- "I have a probit regression coefficient and threshold from my output — how do I turn that into an actual probability?" → this file (Calculating Probabilities from Probit Regression Coefficients)
- "How do I convert a logistic regression coefficient to an odds ratio / probability?" → this file (Calculating Probabilities from Logistic Regression Coefficients — log odds, odds, odds ratio)
- "My categorical latent class variable `c` is regressed on covariates — how do I get the actual class probabilities for a given covariate profile?" → this file (worked multinomial-logit example with a categorical latent variable)
- "Which category is the reference category for my logistic/multinomial model?" → this file (binary DV: `u=0`; 3+ category DV: last category)

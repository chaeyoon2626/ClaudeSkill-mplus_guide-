# Mixture CFA and Structural Equation Mixture Modeling (Continuous Factors Combined with a Categorical Latent Class Variable)

> Source: Mplus User's Guide v8, Chapter 7, Example(s) 7.17, 7.19, 7.20

## One-line summary
Combines a continuous latent factor (measured by a CFA-style measurement model) with a categorical latent class variable c in the same model — either letting the factor's mean/distribution differ across latent classes (mixture CFA, 7.17), regressing the categorical latent variable c on a continuous factor f (7.19), or letting a full structural path between two continuous factors vary across latent classes (structural equation mixture modeling, 7.20).

## Prerequisite checklist
- [ ] Confirm whether the continuous latent variable is meant to be an outcome that differs by class (mixture CFA, factor mean varies across c), a predictor of class membership (c regressed on f), or a mediator/structural variable whose regression path on another factor varies by class (SEM mixture)
- [ ] Identify all indicators for each continuous factor and, separately, all indicators for the categorical latent class variable (they may be entirely different variable sets, e.g. 7.19's u1-u4 for f versus u5-u8 for c)
- [ ] Decide which model parameters should be allowed to vary across classes versus held equal as the default (Mplus holds everything in `%OVERALL%` equal across classes unless overridden in a class-specific `%c#k%` block)
- [ ] Be aware that models regressing a categorical latent variable on a continuous factor, or otherwise requiring integration over a continuous latent variable within a mixture, typically need `ALGORITHM = INTEGRATION;` (numerical integration), which grows more demanding as the number of factors/dimensions and sample size increase
- [ ] Confirm indicator types (continuous vs. categorical) per factor so `CATEGORICAL =` is set correctly for any binary/ordinal indicators

## Option selection logic
| Situation | Choice |
|---|---|
| Want a continuous factor's mean (or other distributional feature) to differ across latent classes, i.e. factor mixture / mixture CFA | `f BY y1-y5;` in `%OVERALL%`; add `[f*1];` inside each `%c#k%` block to free the factor mean in that class (the residual/variance of f is implicitly allowed to vary within class by default in this setup) |
| Want to regress a categorical latent class variable c on a continuous factor f (c depends on f) | `f BY <f-indicators>;` then `c ON f;` in `%OVERALL%` — produces a multinomial logistic regression of c on f; needs `ALGORITHM = INTEGRATION;` |
| Want a full structural path between two continuous factors (f2 ON f1) where the intercept/slope of that path differs by class | `f1 BY ...; f2 BY ...; f2 ON f1;` in `%OVERALL%`, then re-specify `[f1*1];` (class-varying factor mean), `[f2]` intercept, and/or `f2 ON f1;` again inside `%c#k%` to free those specific parameters by class |
| Only some structural/measurement parameters should vary by class, others should stay equal | Put class-invariant parts in `%OVERALL%` only; repeat just the parameters that must vary inside the relevant `%c#k%` block(s) — anything not repeated in a class block stays at its `%OVERALL%` (equal-across-class) value |
| Model includes any regression of a categorical latent variable on a continuous latent factor, or requires integrating over continuous latent variables inside a mixture | `ANALYSIS: ALGORITHM = INTEGRATION;` (adds numerical integration; default is 15 integration points per dimension) |

## Menu path & screen fields
1. **VARIABLE:**
   - `NAMES ARE y1-y5;` (mixture CFA) / `NAMES ARE u1-u8;` (SEM with categorical c on continuous f) / `NAMES ARE y1-y6;` (structural equation mixture)
   - `CLASSES = c (2);`
   - `CATEGORICAL = u1-u8;` — only when the relevant indicators are categorical (7.19's factor and class indicators are all binary in that example)
2. **ANALYSIS:**
   - `TYPE = MIXTURE;`
   - `ALGORITHM = INTEGRATION;` — needed for 7.19's c-on-f regression; not needed in 7.17/7.20 since those keep the factor(s) as outcomes/mediators rather than regressing c on a continuous factor
3. **MODEL:**
   - `%OVERALL%` block: `BY` statement(s) for the factor(s), any class-invariant `ON` structural paths, and (for 7.19) `c ON f;`
   - `%c#1%`, `%c#2%`, ... blocks: any parameter that should be freed to differ by class — factor mean `[f*1];`, intercepts `[f2];`, or structural slopes `f2 ON f1;`
4. **OUTPUT:** `TECH1 TECH8;` (as in the base LCA/LPA file)

## Sub-option details
- `f BY y1-y5;` in `%OVERALL%` (7.17) : specifies factor f is measured by y1-y5 with loadings held equal across classes as the default; the picture's residual arrow into f signals that f's own distribution (not just its mean) is allowed to vary freely within each class, so f need not be normally distributed overall
- `[f*1];` inside `%c#k%` (7.17) : frees the mean of factor f to be estimated separately (starting value 1) in that specific class, which is what actually creates class differences in the factor — all other CFA parameters (loadings, residual variances) remain equal across classes unless also repeated in a class block
- `f BY u1-u4; c ON f;` (7.19) : f is a continuous factor measured by binary indicators u1-u4; c (itself measured by a separate indicator set u5-u8) is regressed on f via a multinomial logistic regression — i.e., a continuous factor predicts which latent class an individual falls into
- `[u5$1-u8$1];` repeated inside each `%c#k%` block (7.19) : frees the thresholds of c's own indicators to differ by class, which is what actually distinguishes the classes of c (standard LCA-style class-specific thresholds), on top of the separate f→c regression
- `f1 BY y1-y3; f2 BY y4-y6; f2 ON f1;` (7.20) : two continuous factors in a simple mediation-style structural chain, with f2 regressed on f1, all specified in `%OVERALL%` as the class-invariant baseline
- `[f1*1]; f2 ON f1;` repeated inside `%c#1%` (7.20) : frees the mean of f1 and the slope of f2 on f1 (and, per the model picture, the intercept of f2) to differ in class 1 specifically — the dashed arrow in the diagram from c to the f1→f2 path denotes that this structural slope is what is being allowed to vary by class, over and above any class difference in f1's or f2's mean/intercept
- `ALGORITHM = INTEGRATION;` : switches estimation to maximum likelihood with robust standard errors using numerical integration, required whenever a continuous latent variable appears on the right-hand side of a regression involving a categorical latent variable (as in `c ON f;`); computational burden increases with the number of integration dimensions (factors requiring integration) and sample size, so keep the number of such factors as small as the model requires

## Post-run operations
- For mixture CFA (7.17), examine the class-specific factor means `[f]` to see how the underlying continuous trait differs across the latent classes, and treat any other freed parameters the same way; compare against a fully class-invariant CFA to judge whether allowing class differences meaningfully improves fit
- For c-on-f models (7.19), interpret the `c ON f` coefficient as a log-odds effect of the continuous factor on class membership (exponentiate for an odds ratio), and separately check whether c's own indicator thresholds differ substantially across classes to confirm the classes are well separated on their own indicators
- For structural equation mixture models (7.20), read the class-specific `f2 ON f1` slopes as evidence that the mediating/structural relationship itself differs by (unobserved) subgroup — this is the model's key novel feature versus a single-class SEM, so contrast the class-specific slopes directly
- In all three, confirm "MODEL ESTIMATION TERMINATED NORMALLY" and check that the best loglikelihood value was replicated across random starts (see mixture-lpa-lca-cross-sectional.md); with `ALGORITHM=INTEGRATION` models, also watch runtime/convergence since numerical integration is more demanding
- Decide up front (and document) which parameters were deliberately left in `%OVERALL%` (class-invariant) versus freed in `%c#k%` blocks, since that choice — not just the number of classes — determines what "the mixture" is actually capturing in these combined factor/class models

## Likely FAQ mapping
- "I want a CFA factor whose mean differs across some unobserved subgroup" → mixture CFA, Example 7.17 (`f BY ...;` in `%OVERALL%`, `[f*1];` in each `%c#k%`)
- "Can a continuous factor predict which latent class someone is in?" → yes, `c ON f;`, Example 7.19; needs `ALGORITHM = INTEGRATION;`
- "I have a mediation/structural model and suspect the path coefficient differs by an unobserved subgroup" → structural equation mixture modeling, Example 7.20 (`f2 ON f1;` repeated inside `%c#k%` to free the slope by class)
- "My model has a continuous latent factor together with a categorical latent class variable — is that even allowed?" → yes, Mplus supports combining continuous and categorical latent variables in one `TYPE = MIXTURE;` model, per Examples 7.17-7.20
- "Do I need special estimation settings for these combined models?" → only when a categorical latent variable is regressed on a continuous factor (or similar integration-requiring specification) — add `ALGORITHM = INTEGRATION;`, as in Example 7.19

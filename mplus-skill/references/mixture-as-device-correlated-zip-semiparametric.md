# Mixture Components as a Statistical Device: Correlated Indicators, Zero-Inflation, and Semiparametric Factor Distributions

> Source: Mplus User's Guide v8, Chapter 7, Examples 7.22, 7.25, 7.26
> https://www.statmodel.com/HTML_UG/chapter7V8.htm

## One-line summary
Covers three uses of `TYPE = MIXTURE` where the latent classes are not meant to represent substantively meaningful subgroups, but instead serve as a technical device: relaxing the default within-class conditional-independence assumption to allow correlated continuous indicators (multivariate normal mixture, Ex 7.22), representing a single count variable's zero-inflation as a 2-class split instead of a within-indicator inflation parameter (Ex 7.25), or approximating a non-normal continuous factor distribution with a histogram of class-specific factor means (semiparametric/non-parametric CFA, Ex 7.26).

## Prerequisite checklist
- [ ] Clarify for the user (or yourself) up front which use case applies, since the model syntax looks like ordinary class enumeration but the classes themselves should not be interpreted as real subgroups in cases 2 and 3 below
- [ ] For the correlated-indicators case (Ex 7.22): confirm the indicators are continuous and there's reason to think they correlate within class beyond what the class means already explain (the standard LPA default fixes these covariances to zero)
- [ ] For the zero-inflation-as-class case (Ex 7.25): confirm the outcome is a single count variable, and decide whether this whole-class approach (which also yields individual posterior probabilities of being in the "structural zero" group) is preferred over the simpler single-indicator zero-inflation option (`COUNT = u (i);`, covered in `mixture-lca-covariates-predictors.md` and `mixture-class-indicator-types-starting-values.md`)
- [ ] For the semiparametric factor case (Ex 7.26): confirm the goal is to represent a factor's non-normal shape using a set of latent classes standing in for scale steps, not to find genuinely distinct subpopulations

## Option selection logic
| Situation | Choice |
|---|---|
| Continuous indicators need to correlate within class instead of being conditionally independent | Add `WITH` statements among the indicators in `%OVERALL%`, e.g. `y1 WITH y2-y4; y2 WITH y3 y4; y3 WITH y4;` — relaxes the default zero-covariance-within-class assumption; the resulting covariances are held equal across classes by default |
| Want the excess-zero process modeled as its own latent class (rather than a within-indicator inflation parameter) and want individual posterior probabilities of "structural zero" membership | `COUNT = u1;` (plain Poisson, no `(i)`), `CLASSES = c (2);`, fix class 1's intercept at an extreme low value (e.g. `[u1@-15];`) and its regression slopes at zero, so class 1 represents individuals who can only ever be observed at zero |
| Want a continuous latent factor's distribution represented without assuming normality, using classes as scale steps | `f BY y1-y5; f@0;` inside `%OVERALL%` with `CLASSES = c (k);` — fixing the factor variance to zero within each class means each class becomes one "step" on the factor scale, and the class proportions/means jointly approximate the factor's shape |
| Want the same non-normal-factor idea but as a more general mixture (parametric normality restored within each class) | Do not fix `f@0;` — let the factor variance be freely estimated, reverting to an ordinary factor mixture within each class |

## Menu path & screen fields
1. **VARIABLE:**
   - Correlated-indicators MVN mixture: `NAMES ARE y1-y4; CLASSES = c (3);` (continuous indicators, no special declaration)
   - Zero-inflation-as-class: `NAMES ARE u1 x1 x3; COUNT IS u1; CLASSES = c (2);`
   - Semiparametric factor: `NAMES ARE y1-y5 c; USEVARIABLES = y1-y5; CLASSES = c (3);`
2. **ANALYSIS:** `TYPE = MIXTURE;` in all three cases
3. **MODEL:**
   - Correlated-indicators: `%OVERALL%` block with `WITH` statements among the indicators; class-specific blocks with starting values for the means (e.g. `[y1-y4*-1];` for class 2, `[y1-y4*1];` for class 3)
   - Zero-inflation-as-class: `%OVERALL%` with `u1 ON x1 x3; c ON x1 x3;`; `%c#1%` block with `[u1@-15]; u1 ON x1@0 x3@0;`
   - Semiparametric factor: `%OVERALL%` with `f BY y1-y5; f@0;`
4. **OUTPUT:** `TECH1 TECH8;`

## Sub-option details
- `y1 WITH y2-y4; y2 WITH y3 y4; y3 WITH y4;` (Ex 7.22) : relaxes the mixture-model default of fixing all within-class indicator covariances to zero; these covariance parameters are held equal across classes by default (that default can be overridden by repeating the `WITH` statement inside a class-specific block). Variances of the indicators are estimated and held equal across classes by default too. This is a conventional multivariate normal mixture model in the sense of Everitt & Hand (1981) and McLachlan & Peel (2000) — the classes here genuinely can represent substantive subgroups, since the extension over base LPA is only about permitting correlation, not about repurposing the classes as a device. It is grouped in this file because it is the model that makes correlated-indicator LPA/mixture models possible, contrasted with the two device-only cases below.
- `COUNT IS u1;` with no `(i)` (Ex 7.25) : declares u1 a plain (non-inflated) count/Poisson class indicator; the zero-inflation is instead produced entirely by mixing two classes together — one class in which every individual is fixed at zero (via `[u1@-15];` making the Poisson log-rate parameter effectively minus infinity, forcing all predicted counts to zero, and via `u1 ON x1@0 x3@0;` fixing that class's regression slopes at zero since there is no variability left to explain) and one ordinary Poisson class. `c ON x1 x3;` becomes the logistic part predicting membership in the "structural zero" class vs. the Poisson class — functionally equivalent to Example 3.8's single-indicator zero-inflated Poisson model, but reparameterized so that each individual gets an explicit posterior probability of belonging to the always-zero class, in addition to the usual model-implied probabilities.
- `f BY y1-y5; f@0;` (Ex 7.26) : `f@0;` fixes the within-class factor variance to zero, so all variation in the factor is carried entirely by which class an individual falls into (i.e., a discrete set of factor score "steps") rather than by continuous within-class variation. One class's factor mean is fixed at zero for identification; the other classes' factor means, together with the class proportions, jointly trace out a histogram-like (non-parametric/semiparametric) approximation to the factor's true, possibly non-normal, distribution (Aitkin, 1999; Muthén, 2002, 2004). Leaving the factor variance free instead reverts to an ordinary (parametric, within-class-normal) factor mixture model.

## Post-run operations
- Correlated-indicators MVN mixture: inspect the estimated within-class covariances/correlations directly — if they turn out negligible, the simpler conditional-independence LPA default (see `mixture-lpa-lca-cross-sectional.md`) may fit just as well with fewer parameters.
- Zero-inflation-as-class: use `SAVEDATA: SAVE = CPROBABILITIES;` to obtain each individual's posterior probability of belonging to the "always zero" class versus the Poisson class — this is the main practical advantage over the single-indicator `(i)` inflation approach, which does not directly hand back this posterior probability.
- Semiparametric factor: do not interpret the classes substantively (as if they were real subgroups) — they exist only to approximate the shape of a single continuous factor. Instead, examine the pattern of class-specific factor means and proportions together as a histogram describing the factor's distribution, and compare model fit against a standard single-class (parametric, assumed-normal) CFA to judge whether normality was a poor assumption.
- In all three cases, `TECH1` confirms which parameters ended up fixed/free/equal as intended (especially important for the deliberately-fixed parameters in the zero-inflation-as-class and semiparametric-factor cases) and `TECH8` shows the optimization history.

## Likely FAQ mapping
- "My LPA indicators seem correlated within class beyond what the class means explain — is that allowed?" → yes, add `WITH` statements in `%OVERALL%` (Ex 7.22); by default mixture/LPA models assume zero within-class covariance
- "I want to model excess zeros in a count variable but also get each person's probability of being a 'structural zero'" → use the 2-class device from Ex 7.25 (`COUNT IS u;` with no `(i)`, plus a fixed-at-zero class) rather than the single-indicator `(i)` inflation option
- "What's the difference between `COUNT = u (i);` and modeling zero-inflation as a 2-class mixture?" → both represent the same substantive zero-inflated Poisson process, but the class-based version (Ex 7.25) gives explicit posterior class probabilities per person, while `(i)` (see `mixture-class-indicator-types-starting-values.md`) is a more compact single-indicator specification
- "My factor doesn't look normally distributed — can I avoid assuming normality without going fully nonparametric?" → semiparametric/histogram representation via `f@0;` plus multiple classes (Ex 7.26); increasing the number of classes gives a finer-grained histogram approximation
- "Should I interpret these classes as real subgroups of people?" → only for the correlated-indicators MVN mixture (Ex 7.22); for the zero-inflation and semiparametric-factor uses, the classes are a computational device, not a substantive typology

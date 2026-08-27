# Growth Mixture Modeling (GMM) & Latent Class Growth Analysis (LCGA)

> Source: Mplus User's Guide v8, Chapter 8, Examples 8.1-8.11
> Bundled source: references/source-pdfs/Chapter8.pdf

## One-line summary
Models unobserved subgroups (latent trajectory classes) in a repeated-measures/longitudinal outcome: GMM allows within-class variability around each class's growth trajectory (random growth factors), while LCGA fixes the growth factor variances to zero so everyone in a class shares the same trajectory — both use `TYPE = MIXTURE` combined with a `|` growth-factor definition, as in ordinary growth modeling.

## Prerequisite checklist
- [ ] Outcome type: continuous, censored, categorical (binary/ordinal), or count (Poisson/zero-inflated Poisson/negative binomial) — determines the VARIABLE-command keyword (`CENSORED`, `CATEGORICAL`, `COUNT`) and whether numerical integration (`ALGORITHM = INTEGRATION;`) is needed
- [ ] Decide GMM vs LCGA: allow individual variation around each class's trajectory (growth factor variances free → GMM) or force everyone in a class onto the same trajectory (growth factor variances fixed at zero → LCGA)
- [ ] Number and spacing of repeated measurement occasions, and the fixed time scores to use in the `|` statement (equidistant occasions shown in all chapter examples)
- [ ] Whether a covariate predicts the growth factors and/or class membership
- [ ] Whether a distal outcome (measured outside the growth process) needs to be related to class membership
- [ ] Number of classes to compare (start small, increase) and a starting-value strategy (`STARTS =`, `STITERATIONS =`), given mixture models' sensitivity to local optima

## Option selection logic
| Situation | Choice |
|---|---|
| Continuous repeated outcome | `i s | y1@0 y2@1 y3@2 y4@3;` under `%OVERALL%`, no extra VARIABLE keyword needed |
| Censored (floor/ceiling) continuous outcome | `CENSORED = y1-y4 (b);` (`b` = censored from below/floor effect, limit taken from data) + `ANALYSIS: ALGORITHM = INTEGRATION;` |
| Binary/ordinal repeated outcome, want within-class variability (GMM) | `CATEGORICAL = u1-u4;` + `ANALYSIS: ALGORITHM = INTEGRATION;` (growth factor variances then estimated) |
| Binary/ordinal repeated outcome, no within-class variability (LCGA) | `CATEGORICAL = u1-u4;`, omit `ALGORITHM = INTEGRATION;` — growth factor variances are fixed at zero by default in this case |
| Count repeated outcome, zero-inflated | `COUNT = u1-u8 (i);` + `ALGORITHM = INTEGRATION;` + two `|` statements (count part, inflation part) |
| Count repeated outcome, overdispersed but not zero-inflated | `COUNT = u1-u8 (nb);` (negative binomial, one dispersion parameter per outcome) + `ALGORITHM = INTEGRATION;` |
| Covariate predicts growth factors and/or class membership | `i s ON x;` (growth factors on x) and `c ON x;` (multinomial logistic regression of class on x), both in `%OVERALL%` |
| Categorical distal outcome related to class membership | add it as an extra `CATEGORICAL` variable (e.g., `CATEGORICAL = u;`); its thresholds vary by class by default |
| Two sequential growth processes whose classes are related | two categorical latent variables, e.g. `CLASSES = c1 (3) c2 (2);`, with `c2 ON c1;` in `%OVERALL%` and separate `MODEL c1:` / `MODEL c2:` blocks — a simple form of latent transition analysis across two distinct constructs |
| Class membership is actually known/observed (multiple-group mixture) | `KNOWNCLASS = cg (g = 0 g = 1);` alongside a second, unknown categorical latent variable `c` |
| Concerned about local optima | increase `STARTS = n m;` beyond the default (e.g., `STARTS = 40 8;`) and/or `STITERATIONS =` beyond the default of 10 |

## Menu path & screen fields
1. **DATA:** `FILE IS ...;`
2. **VARIABLE:**
   - `NAMES ARE y1-y4 x;`
   - `CLASSES = c (2);` — categorical latent (trajectory-class) variable and number of classes
   - `CATEGORICAL = u1-u4;` (binary/ordinal outcomes only) / `CENSORED = y1-y4 (b);` (censored outcomes only) / `COUNT = u1-u8 (i);` or `(nb)` (count outcomes only)
   - `KNOWNCLASS = cg (g = 0 g = 1);` — only for known-class (multiple-group) mixture models
3. **ANALYSIS:**
   - `TYPE = MIXTURE;`
   - `ALGORITHM = INTEGRATION;` — needed for censored/categorical/count GMM with free growth-factor variances (numerical integration)
   - `STARTS = 40 8;` / `STITERATIONS = 20;` — random-start settings for avoiding local optima
4. **MODEL:**
   - `%OVERALL%` block: `i s | y1@0 y2@1 y3@2 y4@3;` (growth factor definition, common to all classes), `i s ON x;`, `c ON x;`
   - Class-specific blocks `%c#1%`, `%c#2%`, ... for starting values or class-specific restrictions (e.g., `[i*1 s*.5];`)
5. **OUTPUT:** `TECH1 TECH8;` (default recommended pair)

## Sub-option details
- `i s | y1@0 y2@1 y3@2 y4@3;` : names (`i`, `s`) and defines the intercept/slope growth factors; the right-hand side gives the outcome and fixed time scores for the slope factor (here a linear growth model with equidistant occasions 0, 1, 2, 3). Intercept-factor loadings are fixed at 1 as part of the growth model parameterization.
- `i s ON x;` : linear regression of the growth factors on covariate x, estimated in `%OVERALL%` (applies across classes unless overridden in a class-specific block)
- `c ON x;` (or `c#1 ON x;`) : multinomial logistic regression of class membership on covariate x; `c#1` refers to class 1 of c — this `#` notation lets individual class-specific parameters be referenced for starting values or restrictions
- `CENSORED = y1-y4 (b);` : `b` = censored from below (floor effect); the censoring limit is taken from the data; requires `ALGORITHM = INTEGRATION`
- `CATEGORICAL = u1-u4;` : declares binary/ordinal outcomes; thresholds (not intercepts) are estimated, held equal across time by default for LCGA
- `COUNT = u1-u8 (i);` : `(i)` requests a zero-inflated Poisson model — two `|` statements are then needed: one growth model for the count part (e.g., `i s q | u1@0 ...`), one for the inflation part (e.g., `ii si qi | u1#1@0 ...`), referencing the binary inflation part of each count variable via `#1`
- `COUNT = u1-u8 (nb);` : `(nb)` requests a negative binomial model, with one dispersion parameter estimated per outcome (held equal across classes by default)
- `s-qi@0;` : shorthand for fixing a consecutively-listed run of parameters (e.g., several growth factor variances) to 0 in one statement
- `MODEL c1: / MODEL c2:` : when there is more than one categorical latent variable, each gets its own MODEL command; the class-specific parts inside each use labels like `%c1#1%`
- `KNOWNCLASS = cg (g = 0 g = 1);` : identifies `cg` as a categorical latent variable whose class membership is known/observed, defined from the values of observed variable g — used for multiple-group mixture analysis under TYPE=MIXTURE
- `STARTS = 40 8;` : 40 initial-stage random start sets, best 8 carried to final-stage optimization
- `STITERATIONS = 20;` : number of iterations used in the initial-stage optimizations (default 10)

## Post-run operations
- Confirm "MODEL ESTIMATION TERMINATED NORMALLY" and that the best loglikelihood was replicated across starting values (via TECH8 output) — if not replicated, a local optimum is likely; increase STARTS/STITERATIONS and rerun
- For deciding GMM vs LCGA fit, and for deciding the number of classes, apply the same class-enumeration workflow as cross-sectional mixture models (BIC, TECH11/TECH14, entropy, class sizes, interpretability) — see `mixture-lpa-lca-cross-sectional.md` for the general logic; it applies equally here
- To save individual class assignments/probabilities for downstream use: `SAVEDATA: SAVE = CPROBABILITIES; FILE = classprobs.dat;` (see `output-savedata-plot-commands.md`)
- Graphical checks (individual/estimated trajectory plots by class) are available via the `PLOT` command, post-processed in the Mplus graphics module (histograms, scatterplots, observed/estimated means and probabilities by class, by group, or adjusted for covariates)

## Likely FAQ mapping
- "What's the difference between GMM and LCGA?" → GMM allows within-class variation in the trajectory (growth factor variances free, i.e., random effects); LCGA does not (variances fixed at zero) — same `TYPE=MIXTURE` + `|` syntax either way; for categorical/count outcomes, omitting `ALGORITHM=INTEGRATION` produces the LCGA (variances fixed at 0) default
- "My repeated outcome is a count (e.g., number of symptoms) — how do I do GMM/LCGA on it?" → `COUNT = u1-u8 (i);` for zero-inflated Poisson or `(nb)` for negative binomial, plus `ALGORITHM = INTEGRATION;` (Examples 8.5, 8.11)
- "I want a covariate that predicts both the trajectory shape and which class someone is in" → `i s ON x;` and `c ON x;` in `%OVERALL%`
- "I want to relate class membership to a categorical outcome measured later (distal outcome)" → add it as an extra `CATEGORICAL` variable; its thresholds vary by class by default (Example 8.6)
- "My class membership (e.g., treatment group) is actually known/observed, not latent — I want separate class enumeration within each group" → `KNOWNCLASS = cg (g = 0 g = 1);` combined with an additional unknown class variable `c` (Example 8.8)
- "I have two growth processes (e.g., two constructs or two phases) and want to see how classes at process 1 relate to classes at process 2" → two categorical latent variables (`CLASSES = c1(3) c2(2);`) with `c2 ON c1;` — a simple form of latent transition analysis; for the *same* indicator(s) repeated over time, see `latent-transition-analysis-hidden-markov.md` instead

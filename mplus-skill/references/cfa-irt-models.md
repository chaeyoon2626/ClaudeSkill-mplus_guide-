# Item Response Theory (IRT) Models via CFA

> Source: Mplus User's Guide v8, Chapter 5, Example 5.5
> Bundled source: references/source-pdfs/Chapter5.pdf

## One-line summary
Estimates a single-factor CFA with categorical (binary/ordinal) indicators, reparameterized and reported in conventional item response theory (IRT) form — covering the generalized partial credit model (GPCM), two-parameter logistic (2PL) / graded-response model, three-parameter logistic (3PL, with guessing), and four-parameter logistic (4PL, with guessing and an upper asymptote).

## Prerequisite checklist
- [ ] Confirm all items load on a single common factor (a one-factor CFA) — this is the classic unidimensional IRT setup shown here; multidimensional IRT would extend `cfa-categorical-mixed.md` logic but isn't demonstrated in this example
- [ ] Confirm item type: binary items only, or ordered categorical (polytomous) items
- [ ] Decide which IRT model matches the theoretical assumptions:
  - GPCM: partial-credit style, polytomous items, no guessing/asymptotes
  - 2PL (binary) / graded-response model (ordinal): discrimination + difficulty (threshold) parameters only
  - 3PL: adds a lower-asymptote (guessing) parameter — typically needs priors to converge
  - 4PL: adds both a lower (guessing) and upper asymptote parameter — typically needs priors to converge
- [ ] Since ML with categorical indicators requires numerical integration, be aware this gets slower as sample size/dimensions grow

## Option selection logic
| Situation | Choice |
|---|---|
| Polytomous items, partial-credit style | `CATEGORICAL = u1-u20 (gpcm);` |
| Binary items, standard discrimination+difficulty IRT | `CATEGORICAL = u1-u20;` (no parenthetical model keyword) → 2PL |
| Ordinal items, standard discrimination+difficulty IRT | `CATEGORICAL = u1-u20;` → graded-response model |
| Binary items with a guessing parameter | `CATEGORICAL = u1-u20 (3pl);` + `MODEL PRIORS` on the guessing-related threshold |
| Binary items with both guessing and an upper asymptote | `CATEGORICAL = u1-u20 (4pl);` + `MODEL PRIORS` on both extra thresholds |

## Menu path & screen fields
**[GPCM]**
1. **VARIABLE:** `NAMES ARE u1-u20;` / `CATEGORICAL ARE u1-u20 (gpcm);`
2. **ANALYSIS:** `ESTIMATOR = MLR;`
3. **MODEL:** `f BY u1-u20;` `f@1;`
4. **OUTPUT:** `TECH1 TECH8;`
5. **PLOT:** `TYPE = PLOT3;`

**[2PL / graded response]**
1. **VARIABLE:** `CATEGORICAL ARE u1-u20;` (no model keyword in parentheses)
2. **ANALYSIS:** `ESTIMATOR = MLR;`
3. **MODEL:** `f BY u1-u20*;` `f@1;`

**[3PL with guessing]**
1. **VARIABLE:** `CATEGORICAL = u1-u20 (3pl);`
2. **MODEL:** `f BY u1-u20*;` `f@1;` `[u1$2-u20$2] (a1-a20);`
3. **MODEL PRIORS:** `a1-a20~N(1.386,1);`

**[4PL with guessing + upper asymptote]**
1. **VARIABLE:** `CATEGORICAL = u1-u20 (4pl);`
2. **MODEL:** `f BY u1-u20*;` `f@1;` `[u1$2-u20$2] (a1-a20);` `[u1$3-u20$3] (b1-b20);`
3. **MODEL PRIORS:** `a1-a20~N(1.386,1);` `b1-b20~N(-2,1);`

## Sub-option details
- `CATEGORICAL = u1-u20 (gpcm);` : the model keyword in parentheses after the variable list selects which IRT model is estimated; omitting it (plain `CATEGORICAL ARE u1-u20;`) gives the standard 2PL/graded-response parameterization
- `f BY u1-u20*; f@1;` : the asterisk (`*`) frees the first factor loading, which is otherwise fixed to 1 by default to set the factor's metric; instead, the metric is set IRT-style by fixing the factor variance to 1 with `f@1`. For one-factor models with no covariates, Mplus reports results in *both* the factor-model parameterization and the conventional IRT parameterization automatically
- `ESTIMATOR = MLR;` : ML with robust standard errors using numerical integration — required because the model links categorical indicators to a continuous factor via logistic regressions; numerical integration becomes increasingly demanding as the number of factors and sample size grow
- Thresholds are referenced with `$` followed by a number: `u1$1` is the first threshold. Parameters that can't be referenced directly (the guessing and upper-asymptote parameters in 3PL/4PL) are accessed via the second (`$2`) and third (`$3`) thresholds, which must be given labels in the `MODEL` command (e.g. `(a1-a20)`) so `MODEL PRIORS` can target them
- 3PL/4PL convergence: because these models are notoriously hard to converge, `MODEL PRIORS` supplies informative priors on the guessing/asymptote-related thresholds. A prior mean of `1.386` on the second threshold corresponds to a guessing value of about 0.25; a prior mean of `-2` on the third threshold corresponds to an upper-asymptote value of about 0.88 (Asparouhov & Muthén, 2016)
- `PLOT: TYPE = PLOT3;` : requests item characteristic curves and information curves, viewable in a post-processing graphics module after the run; if covariates with direct effects on indicators are added, item characteristic curves can be plotted by covariate value to show differential item functioning (DIF)

## Post-run operations
- Compare the IRT-parameterization output (discrimination `a`, difficulty/location `b`) against the factor-model output (loadings, thresholds) — both describe the same fitted model
- For 3PL/4PL, check whether the guessing/upper-asymptote estimates are sensible (e.g., guessing near 1/(number of response options) for multiple-choice-style items) and whether `MODEL PRIORS` needed adjusting for convergence
- Use `PLOT: TYPE = PLOT3;` output to inspect item characteristic curves and test information curves for each item
- `TECH8` shows the optimization history — useful to gauge how long the (numerical-integration-based) run took and whether it converged cleanly

## Likely FAQ mapping
- "I want to run a 2PL/3PL/4PL IRT model" → this file; note IRT models here are just CFA/categorical-factor models with a particular parameterization and (for 3PL/4PL) priors
- "My items are partial-credit / polytomous with an ordered scoring rule" → GPCM, `(gpcm)` keyword
- "The 3PL model won't converge" → point to `MODEL PRIORS` on the labeled second threshold, following the pattern shown here
- "Can I see item characteristic curves?" → yes, `PLOT: TYPE = PLOT3;`, viewed in the post-processing graphics module

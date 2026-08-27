# Non-linear CFA and Latent Variable Interactions

> Source: Mplus User's Guide v8, Chapter 5, Examples 5.7, 5.13
> Bundled source: references/source-pdfs/Chapter5.pdf

## One-line summary
Adds a non-linear (quadratic) or interaction term built from latent factors themselves — e.g. a factor squared, or the product of two factors — as a predictor of observed indicators or of another factor, using the `|` symbol together with `XWITH` and maximum-likelihood numerical integration.

## Prerequisite checklist
- [ ] Confirm the base measurement model first (which factor(s), which indicators) — same as ordinary CFA/SEM
- [ ] Decide which non-linear term is needed:
  - A quadratic effect of a single factor on its own indicators (McDonald, 1967) — Example 5.7
  - An interaction between two distinct factors used to predict a third factor (Klein & Moosbrugger, 2000) — Example 5.13
- [ ] Confirm this needs `TYPE = RANDOM;` with `ALGORITHM = INTEGRATION;` — this is a numerical-integration ML model, which is more computationally demanding than a standard linear CFA/SEM, especially as the number of factors/dimensions of integration grows

## Option selection logic
| Situation | Choice |
|---|---|
| Indicators are theorized to be a quadratic (curvilinear) function of a single factor | quadratic factor term: `fxf \| f XWITH f;` then `y1-y5 ON fxf;` |
| One factor is theorized to moderate another factor's effect on a third factor (a latent interaction) | interaction term: `f1xf2 \| f1 XWITH f2;` then `f3 ON f1xf2;` |
| Effects among factors are all assumed linear | use plain `sem-structural-paths.md` (`ON` statements among factors), no `XWITH` needed |

## Menu path & screen fields
**[Quadratic factor term — 5.7]**
1. **VARIABLE:** `NAMES ARE y1-y5;`
2. **ANALYSIS:**
   - `TYPE = RANDOM;` — specifies a model with a random effect (needed to build the non-linear term)
   - `ALGORITHM = INTEGRATION;` — ML with robust SEs via numerical integration
3. **MODEL:**
   - `f BY y1-y5;` — defines the linear part of the quadratic function (ordinary factor definition)
   - `fxf | f XWITH f;` — defines the quadratic factor term fxf as `f` interacted with itself
   - `y1-y5 ON fxf;` — the quadratic part of the quadratic function: indicators regressed on fxf
4. **OUTPUT:** `TECH1 TECH8;`

**[Interaction between two factors — 5.13]**
1. **MODEL:** (on top of an ordinary multi-factor SEM measurement + structural model, e.g. `f1 BY y1-y3; f2 BY y4-y6; f3 BY y7-y9; f4 BY y10-y12; f4 ON f3; f3 ON f1 f2;`)
   - `f1xf2 | f1 XWITH f2;` — names the latent-variable interaction f1xf2 on the left of `|`, defines it as f1 interacted with f2 on the right via `XWITH`
   - `f3 ON f1xf2;` — uses the latent interaction as a predictor, exactly like any other `ON` statement
2. **ANALYSIS:** `TYPE = RANDOM;` `ALGORITHM = INTEGRATION;`
3. **OUTPUT:** `TECH1 TECH8;`

## Sub-option details
- `<name> | <factor> XWITH <factor>;` : the general syntax for building a latent product/interaction term. The name on the left of `|` becomes a new "variable" usable elsewhere in `MODEL` (as a predictor via `ON`); `XWITH` on the right defines what it's built from. `f XWITH f` gives a quadratic (self-interaction) term; `f1 XWITH f2` gives a genuine two-factor interaction
- `ANALYSIS: TYPE = RANDOM; ALGORITHM = INTEGRATION;` : required whenever a latent interaction/quadratic term is used — this activates the random-effect-based estimation with numerical integration that non-linear latent terms require; the ESTIMATOR option can select among available ML-family estimators
- Numerical integration cost scales with the number of integration dimensions (i.e., roughly the number of factors involved) — Example 5.7 uses one dimension (15 integration points), Example 5.13 uses two dimensions (225 integration points total) — expect run time to grow accordingly
- Default estimator underlying this feature is maximum likelihood; explanation of TITLE/DATA/VARIABLE/OUTPUT commands otherwise follows the same conventions as ordinary CFA (see `cfa-continuous.md`)

## Post-run operations
- Check the coefficient on the quadratic/interaction term (e.g. `y1-y5 ON fxf` or `f3 ON f1xf2`) for significance — this is the test of whether the non-linear/interactive effect exists
- Watch `TECH8` for convergence — numerical-integration models are more prone to convergence issues than standard linear CFA/SEM
- If probing a significant interaction, consider computing conditional effects at representative levels of the moderating factor (following the same logic as `path-analysis-moderated-mediation.md`, but with latent moderators)

## Likely FAQ mapping
- "I think the relationship between my factor and its indicators isn't linear — maybe curvilinear" → quadratic factor term via `XWITH` (Example 5.7 pattern)
- "I want to test whether one latent factor moderates another factor's effect" → latent interaction via `XWITH` (Example 5.13 pattern)
- "Why is this model so much slower than my regular SEM?" → non-linear/interaction latent terms require numerical integration (`TYPE=RANDOM; ALGORITHM=INTEGRATION;`), which is inherently more demanding than the closed-form linear case

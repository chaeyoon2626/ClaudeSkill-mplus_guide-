# Multilevel Mixture Modeling — Two-Level Growth Mixture, LCGA & LTA

> Source: Mplus User's Guide v8, Chapter 10, Example(s) 10.8, 10.9, 10.10, 10.11, 10.12, 10.13
> Bundled source: references/source-pdfs/Chapter10.pdf

## One-line summary
Extends growth modeling, growth mixture modeling (GMM), latent class growth analysis (LCGA), and latent transition analysis (LTA) to two-level clustered/longitudinal data under `TYPE = TWOLEVEL MIXTURE` (add `RANDOM` when a random slope is named with `|` on an `ON` statement), representing the repeated-measures growth or transition process within clusters (`%WITHIN%`) and a second growth or transition structure across clusters (`%BETWEEN%`), with class membership placed at the individual level, the cluster level, or both — including a "three-level via two-level" device that represents a third (cluster) level's own growth trajectory as a between-level growth model.

## Prerequisite checklist
- [ ] Repeated-measures outcome(s) present with a `CLUSTER=` variable identifying the (third-level / between) grouping unit
- [ ] Decide whether the growth process itself needs a random slope-on-intercept regression that varies across clusters (a "three-level growth model represented as two-level", Ex 10.8-10.10) — this needs `TYPE = TWOLEVEL MIXTURE RANDOM;` because of the `|`-named slope used on an `ON` statement
- [ ] Decide where the mixture classes live: cluster level only (`cb`, no individual-level classes, Ex 10.8), individual level only (`c`, Ex 10.9), or both simultaneously (`cb` + `c`, Ex 10.10)
- [ ] For LCGA (Ex 10.11): confirm growth-factor variances should be fixed at 0 within class (no within-class trajectory variability) and indicators are categorical (`CATEGORICAL=`)
- [ ] For LTA (Ex 10.12-10.13): confirm the same indicator set is measured at 2+ occasions (e.g. `u11-u14` at time 1, `u21-u24` at time 2), and decide whether transitions/thresholds should also have cluster-level (between) structure
- [ ] Anticipate numerical integration cost (2 dimensions/225 points is typical across these examples whenever growth-factor or transition random effects are estimated at between; Ex 10.11's simpler LCGA uses 1 dimension/15 points)
- [ ] Decide `STARTS=`/`PROCESSORS=` strategy

## Option selection logic
| Situation | Choice |
|---|---|
| Growth process needs a genuine third level (e.g. students in classrooms in schools) but the data/software structure only supports two levels | represent the third level's growth as a between-level growth model: `%BETWEEN% ib sb | y1@0 y2@1 y3@2 y4@3;`, alongside the ordinary within-level `iw sw | ...;` (Examples 10.8, 10.9, 10.10) |
| Within-level slope factor needs to be regressed on the within-level intercept factor, and that regression's slope should itself vary across clusters | name it with `|` on the `ON` statement, e.g. `s | sw ON iw;`, and use `TYPE = TWOLEVEL MIXTURE RANDOM;` (Example 10.8) |
| Simpler alternative when that random-slope's variance is expected to be zero | skip the `|`/`RANDOM` route; instead repeat `sw ON iw;` as a class-varying slope inside each between-level class-specific block (e.g. `%cb#1% sw ON iw; %cb#2% sw ON iw;`) — avoids referencing the slope's mean/variance in `%BETWEEN%` (Example 10.8, alternative spec) |
| Mixture classes describe clusters only (e.g. school types), not individuals | `CLASSES = cb(2);` + `BETWEEN = cb;`, no separate individual-level categorical latent variable (Example 10.8) |
| Mixture classes describe individuals only (ordinary two-level GMM) | `CLASSES = c(2);`, `c ON x;` at within, `ib sb ON w;` + `c#1 ON w;` at between (Example 10.9) |
| Need classes of individuals *and* classes of clusters together, with cluster classes able to shift the between-level growth intercept independently of individual-level classes | `CLASSES = cb(2) c(2);` + a second between-level intercept-only growth factor, e.g. `ib2 | y1-y4@1;`, whose mean is allowed to vary by `cb` class via `MODEL cb:` while `ib`'s mean varies by class of `c` via `MODEL c:` (Example 10.10) |
| Categorical repeated outcome, want no within-class trajectory variability (LCGA) | fix growth-factor variances at 0 (`i-s@0;`) inside `%WITHIN%`, keep `CATEGORICAL=` on the indicators, and hold indicator thresholds equal across time using shared equality-label numbers at between (Example 10.11) |
| Random mean of a within-level categorical latent variable needs a starting value at between | `c#1*1;` at `%BETWEEN% %OVERALL%` (Example 10.11) |
| Same indicator set repeated at 2+ occasions; want transition probabilities between latent states | LTA: one categorical latent variable per occasion (`CLASSES = c1(2) c2(2);`), `c2 ON c1 x;` and `c1 ON x;` at within (Example 10.12) |
| LTA with cluster-level random intercepts on the class variables | add random intercepts `c1#1`, `c2#1` at between, regress them on a cluster covariate `w` (`c1#1 ON w; c2#1 ON c1#1 w;`), freeing their between-level residual variances (`c1#1 c2#1;`) instead of leaving them at the default 0 (Example 10.12) |
| LTA where cluster-level classes should also exist and shift the transition structure | add a between-level categorical latent variable `cb`, regress the random intercepts on it (`c1#1 ON cb; c2#1 ON cb;`), and let the transition regression `c2 ON c1` itself vary by `cb` class via `MODEL cb: %WITHIN% %cb#1% c2 ON c1;` (Example 10.13) |
| Need faster computation on a multi-core machine | `ANALYSIS: PROCESSORS = 2;` |

## Menu path & screen fields
1. **DATA:** `FILE IS ...;`
2. **VARIABLE:**
   - `NAMES ARE y1-y4 x w clus;` (growth) or `NAMES ARE u11-u14 u21-u24 x w clus;` (LTA)
   - `CATEGORICAL = u1-u4;` — LCGA/LTA indicators
   - `CLASSES = cb(2);` / `CLASSES = c(2);` / `CLASSES = cb(2) c(2);` / `CLASSES = c1(2) c2(2);` / `CLASSES = cb(2) c1(2) c2(2);` — per the Option selection logic table
   - `WITHIN = x;` — individual-level covariate
   - `BETWEEN = w;` (add the between-level categorical latent variable name too if one is used, e.g. `BETWEEN = cb w;`)
   - `CLUSTER = clus;`
3. **ANALYSIS:**
   - `TYPE = TWOLEVEL MIXTURE;` or `TYPE = TWOLEVEL MIXTURE RANDOM;` (add `RANDOM` whenever a `|`-named random slope like `s | sw ON iw;` is used)
   - `STARTS = 0;` (demo runs) or higher for a real analysis
   - `PROCESSORS = 2;` — optional
4. **MODEL:**
   - `%WITHIN% %OVERALL%` : growth-factor `|` statement(s) (e.g. `iw sw | y1@0 y2@1 y3@2 y4@3;`), `ON` regressions of growth factors/classes on `x`, and (LTA) `c2 ON c1 x; c1 ON x;`
   - `%BETWEEN% %OVERALL%` : between-level growth-factor `|` statement(s) (e.g. `ib sb | y1@0 y2@1 y3@2 y4@3;`), `ON` regressions on `w`, and any between-level categorical latent variable's `ON w;` regression
   - Class-specific blocks: `%c#1%`, `%cb#1%`, `%c1#1%`, `%c2#1%`, or combinations, holding `[...]` intercepts/means/thresholds
   - `MODEL c:` / `MODEL cb:` / `MODEL c1:` / `MODEL c2:` — once more than one categorical latent variable is declared
5. **OUTPUT:** `TECH1 TECH8;`

## Sub-option details
- `iw sw | y1@0 y2@1 y3@2 y4@3;` (within) : names and defines the within-level intercept (`iw`) and linear slope (`sw`) growth factors from fixed time scores 0,1,2,3; intercept-factor loadings fixed at 1 by the growth parameterization; residual variances of the outcomes estimated (allowed to differ over time unless constrained) with uncorrelated residuals by default.
- `ib sb | y1@0 y2@1 y3@2 y4@3;` (between) : the analogous growth-factor definition on the between level, representing a third-level (cluster) growth trajectory; the outcome residual variances are fixed at 0 at between by default (conventional multilevel-growth choice, overridable); the between-level intercept factor's residual variance is free by default, while the slope factor's residual variance defaults to 0 (each freed random-effect variance costs one integration dimension).
- `y1-y4 (1);` : holds the four within-level residual variances equal to each other via a shared label (Example 10.8).
- `iw sw ON x;` : linear regression of the within-level growth factors on individual covariate `x`.
- `s | sw ON iw;` (Ex 10.8) : names the between-level random slope `s` of the within-level regression of `sw` on `iw`; `s` has no within-class variance and is a between-level continuous latent variable whose mean can differ by `cb` class (via `[ib sb s];` inside `%cb#1%`/`%cb#2%`) — this is how a third level's influence on the slope-on-intercept relationship gets represented within a two-level specification.
- `iw ON x; sw ON x iw;` then `%cb#1% sw ON iw; %cb#2% sw ON iw;` (Ex 10.8, alternative spec) : the "class-varying slopes" alternative to the `s |` route — the `iw` coefficient in the regression of `sw` is re-specified separately inside each between-level class-specific block, avoiding any reference to its mean/variance in `%BETWEEN%`.
- `cb ON w;` : multinomial logistic regression of the between-level categorical latent variable on cluster covariate `w`.
- `c ON x;` (within) : multinomial logistic regression of the individual-level categorical latent variable on `x` (Example 10.9).
- `c#1 ON w;` (between) : linear regression of the random intercept of the within-level class-1 logit on cluster covariate `w` (Example 10.9).
- `ib2 | y1-y4@1;` (Ex 10.10) : a second between-level intercept-only growth factor, all loadings fixed at 1 — added so the between-level intercept growth-factor mean can vary across classes of both the individual-level categorical latent variable `c` (through `ib`, in `MODEL c:`) and the between-level categorical latent variable `cb` (through `ib2`, in `MODEL cb:`) at once; `ib2`'s mean is fixed at 0 in one `cb` class and freely estimated in the other, for identification.
- `i s | u1@0 u2@1 u3@2 u4@3; i-s@0;` (LCGA, within, Ex 10.11) : defines the growth factors from categorical indicators, then fixes both growth-factor variances at 0 — no within-class trajectory variability, the defining feature of LCGA versus GMM.
- `[u1$1-u4$1*1] (1); [u1$2-u4$2*1.5] (2);` (LCGA, between, Ex 10.11) : holds each threshold equal across the four repeated occasions using a shared label per threshold number, with given starting values.
- `c#1*1;` (LCGA, between, Ex 10.11) : starting value of 1 for the random mean of the within-level categorical latent variable's class-1 logit.
- `CLASSES = c1(2) c2(2);` (LTA, Ex 10.12) : one categorical latent variable per measurement occasion; `c2 ON c1 x;` and `c1 ON x;` in `%WITHIN%` describe the transition from occasion 1 to occasion 2 and its dependence on covariate `x`.
- `c1#1 ON w; c2#1 ON c1#1 w; c1#1 c2#1;` (LTA, between, Ex 10.12) : regressions of the random intercepts of the class-1 logits of `c1` and `c2` on cluster covariate `w` (and, for `c2#1`, also on `c1#1`); listing `c1#1 c2#1;` frees their between-level residual variances instead of leaving them at the default 0.
- `MODEL c1: %BETWEEN% %c1#1% [u11$1-u14$1] (1-4); %c1#2% [u11$1-u14$1] (5-8);` (LTA, Ex 10.12) : sets class-specific thresholds for occasion 1's indicators at between, with equality labels holding the four thresholds equal within a class (repeated per occasion via `MODEL c2:`).
- `c1#1 ON cb; c2#1 ON cb;` (LTA + cluster classes, Ex 10.13) : regressions of the random class-1-logit intercepts of both occasions' categorical latent variables on the between-level categorical latent variable `cb`, letting cluster class shift baseline prevalence at both occasions.
- `MODEL cb: %WITHIN% %cb#1% c2 ON c1;` (Ex 10.13) : because the transition regression itself is meant to vary by cluster class `cb`, it is placed inside `cb`'s class-specific `%WITHIN%` block rather than `%OVERALL%`; the corresponding random slope has no within-class variance by default.
- `PROCESSORS = 2;` : requests 2 processors for parallel computation.
- `TYPE = TWOLEVEL MIXTURE RANDOM;` : the `RANDOM` keyword is required whenever a `|` statement names a random slope of one latent variable regressed on another (e.g. `s | sw ON iw;`), as distinct from an ordinary growth-factor `|` statement.

## Post-run operations
- Estimation cost profile matches the rest of the chapter: 2 integration dimensions/225 points is typical whenever growth-factor or transition random effects are freely estimated at between (Examples 10.8-10.10, 10.12-10.13); Example 10.11 (LCGA, no within-class growth variance) uses 1 dimension/15 points; use `ESTIMATOR=` to change the default estimator.
- Raise `STARTS=` beyond the demo value used in these examples (`STARTS = 0;`) for a real analysis, and check `TECH8` for loglikelihood replication across starts as evidence against a local optimum, especially with more classes or growth/transition parameters.
- For the shared `PLOT` command output and `TYPE=COMPLEX` complex-survey-design corrections that also apply to these models, see `multilevel-mixture-two-level-lca-basics.md`'s Post-run section (not repeated here to avoid duplication).
- For single-level (non-multilevel) GMM/LCGA and LTA class-enumeration workflow (BIC, TECH11/TECH14, entropy, TECH15 for transition tables), see `growth-mixture-modeling-lcga.md` and `latent-transition-analysis-hidden-markov.md` — the same logic applies once the two-level structure is added, evaluated on the `TYPE=TWOLEVEL MIXTURE` output.
- For LTA, `TECH15` (documented for single-level LTA as the way to get transition-probability tables rather than logit coefficients) is not shown in the chapter's two-level LTA examples here — verify it is still requested the same way under `TYPE=TWOLEVEL MIXTURE` in the live guide if the transition-probability table itself is the quantity of interest.

## Likely FAQ mapping
- "My growth data really have three levels (e.g. repeated measures in students in schools) but I only have two `%WITHIN%`/`%BETWEEN%` parts to work with — how do I fit a three-level growth model?" → represent the third level's growth as a between-level growth-factor `|` statement (`ib sb | y1@0 ...;`) alongside the ordinary within-level one (Examples 10.8-10.10)
- "I want a random slope in the regression of one within-level growth factor on another (e.g. `sw ON iw`) that itself varies across clusters" → name it with `|` (`s | sw ON iw;`) and use `TYPE = TWOLEVEL MIXTURE RANDOM;`, or use the simpler class-varying-slopes alternative if its variance is expected to be zero (Example 10.8)
- "Where do my mixture classes belong — clusters, individuals, or both?" → see the Option selection logic table; cluster-only (`cb`), individual-only (`c`), or both together with a second between-level growth factor to let the intercept mean vary by both categorical latent variables independently (Example 10.10)
- "How do I do a two-level LCGA (no within-class trajectory variability) instead of GMM?" → fix the within-level growth-factor variances at 0 (`i-s@0;`) while keeping everything else the same as two-level GMM (Example 10.11)
- "I have the same items measured at two occasions, nested in clusters — how do I do a two-level LTA?" → one categorical latent variable per occasion (`c1`, `c2`, ...) with `c2 ON c1 x;` at within; add cluster-level random intercepts on the class variables regressed on a cluster covariate if cluster effects on prevalence are of interest (Example 10.12)
- "Can cluster-level classes change the transition probabilities between occasions in my two-level LTA?" → yes: add a between-level categorical latent variable `cb`, regress the class-logit random intercepts on it, and move the transition regression `c2 ON c1` into a `cb`-class-specific block via `MODEL cb:` (Example 10.13)

# Three-Level Regression, Path Analysis, MIMIC, and Growth Models

> Source: Mplus User's Guide v8, Chapter 9, Examples 9.20, 9.21, 9.22, 9.23
> Bundled source: references/source-pdfs/Chapter9.pdf

## One-line summary
Extends the two-level `%WITHIN%`/`%BETWEEN%` framework to a genuinely three-level nested structure (e.g. students within classrooms within schools) using `TYPE=THREELEVEL`, with a separate `%BETWEEN <clusterlabel>%` block per intermediate and highest level, and supports random slopes whose own cluster-level variation can in turn be predicted ("slope of a slope").

## Prerequisite checklist
- [ ] **Confirm there really are two nesting cluster variables** (e.g. classroom ID nested in school ID) — three-level models need exactly two `CLUSTER=` variables, listed **highest level first**
- [ ] For every variable, decide which level(s) it is measured on and which level(s) it should vary at: individual-level variables go on `WITHIN=`; cluster-level variables go on `BETWEEN=` with a parenthesized level label (e.g. `(level2)`, `(level3)`) unless they should be modeled on more than one level
- [ ] Is the dependent variable continuous, categorical, or a mix — does the added complexity of 3 levels + categorical outcomes make numerical integration too slow, favoring `ESTIMATOR=BAYES` or `WLSMV`?
- [ ] Are any random slopes needed at level 1 (within), and if so, should their between-cluster variation (at level 2) itself be predicted by a level-3 covariate — i.e. is a "slope of a slope" needed?
- [ ] Is this a three-level **growth** model (repeated measures nested in individuals nested in a higher cluster), a three-level **path/MIMIC** model, or plain three-level regression?
- [ ] Does the model use the compact growth-factor `|` notation (`iw sw | y1@0 y2@1 ...;`) anywhere, which does not require `TYPE=THREELEVEL RANDOM`, or does it use the general `| DV ON covariate;` random-slope notation, which does?

## Option selection logic
| Situation | Choice |
|---|---|
| Continuous DV, random slope at level 1, and that slope's between-cluster (level 2) variation is itself predicted by a level-3 covariate | `TYPE = THREELEVEL RANDOM;` with `s1 \| y ON x;` on `%WITHIN%`, `s2 \| y ON w; s12 \| s1 ON w;` on `%BETWEEN level2%`, and `y ON z; s1 ON z; s12 ON z;` on `%BETWEEN level3%` (Example 9.20) |
| Path model mixing a continuous mediator and a categorical dependent variable, spread across 3 levels, where ML numerical integration would be too costly | `CATEGORICAL = u;` + `ESTIMATOR = BAYES;` with `PROCESSORS=2; BITERATIONS=(1000);` (Example 9.21) |
| Three-level MIMIC/factor model: within-level factors with covariates, between-level factors built from random intercepts, and random loadings/slopes whose own variation cascades from level 2 up to level 3 | `TYPE = THREELEVEL RANDOM;` with `BY` measurement models on `%WITHIN%`, `%BETWEEN level2%`, and `%BETWEEN level3%`, plus `s \| ...` and `sf2 \| ...`/`ss \| ...` random-slope chains across levels (Example 9.22) |
| Three-level **growth** model: repeated occasions nested in individuals nested in a higher cluster, with a covariate at each of the 3 levels | `TYPE = THREELEVEL;` (no `RANDOM` needed — growth-factor `\|` notation) with `iw sw \| y1@0 y2@1 y3@2 y4@3; iw sw ON x;` on `%WITHIN%`, `ib2 sb2 \| y1@0 ... ; ib2 sb2 ON w;` on `%BETWEEN level2%`, and `ib3 sb3 \| y1@0 ...; ib3 sb3 ON z;` on `%BETWEEN level3%` (Example 9.23) |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE = ...;**
3. **VARIABLE:**
   - `NAMES ARE ...;`
   - `CATEGORICAL = u;` — only if a DV is binary/ordinal
   - `CLUSTER = level3 level2;` — **highest cluster level listed first**, i.e. `level2` nested inside `level3`
   - `WITHIN = x;` — variables measured at level 1; an unlabeled entry is modeled only on level 1 (no variance on levels 2/3); a variable omitted from `WITHIN=` entirely is modeled on all three levels
   - `BETWEEN = (level2) w (level3) z;` — cluster-level variables, each tagged with the level it is measured/modeled on; a level-2 variable listed **without** a level label is modeled on both levels 2 and 3; a level-3 variable **must** carry a `(level3)` label
4. **ANALYSIS:**
   - `TYPE = THREELEVEL;` — sufficient when every random effect is defined via the compact growth-factor `|` notation
   - `TYPE = THREELEVEL RANDOM;` — required as soon as any random slope/loading is instead named with `| DV ON covariate;` or `| factor BY indicator;` regression-style syntax
   - `ESTIMATOR = BAYES;` with `PROCESSORS = 2; BITERATIONS = (1000);` — Bayesian alternative, useful when categorical outcomes plus 3 levels make ML numerical integration slow
   - Default estimator otherwise: maximum likelihood with robust standard errors
5. **MODEL:**
   - `%WITHIN%` block: level-1 regressions/measurement model; random slopes named `s1 | y ON x;` (requires `RANDOM`) or growth factors named `iw sw | y1@0 y2@1 ...;` (does not)
   - `%BETWEEN level2%` block: regressions of level-1 random effects (referenced by name, e.g. `y`, `s1`) on level-2 covariates; new level-2-only random slopes can themselves be named here (e.g. `s2 | y ON w;`), including slopes of level-1 random slopes (`s12 | s1 ON w;`)
   - `%BETWEEN level3%` block: regressions of level-2 random effects (including the "slope of a slope" `s12`) on level-3 covariates; `WITH` statements to free residual covariances among the level-3 random effects
6. **OUTPUT:** `TECH1 TECH8;` — recommended whenever `ESTIMATOR=BAYES` or numerical integration is used

## Sub-option details
- `CLUSTER = level3 level2;`: order matters — the cluster variable for the **highest** level is listed first, and the lower-level cluster variable must be nested within it.
- `WITHIN=` labeling rules: an individual-level variable mentioned **without** a label on `WITHIN=` is modeled only on level 1 (level 1 only, no random effect); a variable **not mentioned** on `WITHIN=` at all is modeled on all three levels (it automatically gets a random intercept at both level 2 and level 3, referenced by its own name).
- `BETWEEN=` labeling rules: a variable measured at level 2 that is listed **with** a `(level2)` label is modeled only at level 2 (no variance contributed at level 3); the same variable listed **without** a label is modeled at both level 2 and level 3; a variable measured at level 3 must always carry an explicit `(level3)` label.
- "Slope of a slope" pattern: a within-level random slope (e.g. `s1`) can itself be treated as a dependent variable at level 2 — `s12 | s1 ON w;` names a **new** random effect (`s12`) representing how much `s1` varies across level-3 units, driven by the level-2 covariate `w`; `s12` is then referenced again in the `%BETWEEN level3%` block just like any other random effect.
- Random-intercept/slope residual (co)variances are estimated and **uncorrelated by default** at level 2, and likewise at level 3 — use explicit `WITH` statements to free correlations (as done in Examples 9.20–9.22).
- For the three-level growth model (Example 9.23), the outcome intercepts at each occasion are fixed at zero as the default identification convention; within-level residual variances of `y1`–`y4` are free and allowed to differ across time by default; level-2 residual variances of `y1`–`y4` are likewise free and allowed to differ by default, but the level-3 residual variances of `y1`–`y4` are **fixed at zero by default** (an asymmetric default worth checking before interpreting level-3 output).
- Three-level MIMIC models (Example 9.22) can combine two within-level factors (`fw1`, `fw2`), each with its own indicator set, with between-level factors built from the random intercepts (`fb2` at level 2, `fb3` at level 3); a random loading (e.g. `s`) defined at level 1 can have its own level-2 variation (`ss | s ON w;`) whose level-3 variation is then predicted in turn — the "slope of a slope" pattern generalizes to loadings, not just regression slopes.
- Bayesian estimation (`ESTIMATOR=BAYES`) is chosen in the mixed continuous/categorical three-level path example specifically to avoid the heavy numerical-integration cost that categorical outcomes plus 3 levels of random effects would otherwise require; `PROCESSORS=2` speeds up computation when multiple processors are available, and `BITERATIONS=(1000)` sets the **minimum** number of MCMC iterations per chain (maximum remains the default 50,000) subject to the potential scale reduction (PSR) convergence criterion.

## Post-run operations
- Read `%WITHIN%`, `%BETWEEN level2%`, and `%BETWEEN level3%` output as three separate but linked sets of equations — level-2 coefficients describe how level-1 random effects vary across level-2 units, and level-3 coefficients describe how level-2 random effects (including any "slope of a slope") vary across level-3 units.
- For any "slope of a slope" (e.g. `s12`), check its variance at the level where it was defined for evidence of genuine cross-cluster heterogeneity, and check whatever covariate regresses on it for what explains that heterogeneity.
- For the three-level growth model, remember the asymmetric default residual-variance treatment (level-2 outcome residuals free, level-3 outcome residuals fixed at zero) — free the level-3 residuals explicitly if there is reason to think the growth model does not fully explain level-3 variability in the raw outcomes.
- `OUTPUT: TECH1;` confirms parameter specification/starting values; `TECH8;` shows optimization or MCMC progress — check both, especially with `ESTIMATOR=BAYES` or numerical integration.
- If sampling weights or a stratified/clustered survey design are also involved (three levels of sampling design rather than three genuinely nested substantive levels), see `complex-survey-design-multilevel.md`.
- If the two grouping structures are **not** nested (e.g. students cross-classified by both neighborhood and school), this is not a three-level model — see `cross-classified-models.md` instead.

## Likely FAQ mapping
- "I have students in classrooms in schools and want a regression with random effects at each level" → Example 9.20 pattern (`TYPE=THREELEVEL RANDOM`)
- "Does the between-cluster variation in my random slope depend on a higher-level covariate?" → Example 9.20's "slope of a slope" pattern (`s12 | s1 ON w;`)
- "I have a 3-level path model with a categorical dependent variable and ML is too slow" → Example 9.21 pattern (`ESTIMATOR=BAYES`)
- "I want a factor model (MIMIC/CFA) across three nested levels, possibly with random loadings" → Example 9.22 pattern
- "I have repeated measures nested in people nested in a higher-level cluster (3 conceptual levels) and want a growth curve" → Example 9.23 pattern (`TYPE=THREELEVEL`, growth `|` notation, no `RANDOM` needed)
- "Do I need TYPE=THREELEVEL RANDOM?" → only if a random effect is named via `| DV ON covariate;` or `| factor BY indicator;`; the compact growth-factor `| y1@0 y2@1 ...;` notation does not require it
- "My two grouping variables aren't nested in each other" → not a three-level model, see `cross-classified-models.md`
- "I also have survey weights or a stratified design layered on top of this three-level structure" → point to `complex-survey-design-multilevel.md`

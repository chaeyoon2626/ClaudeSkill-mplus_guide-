# MODEL Command — Core Syntax & Cross-Cutting Notation

> Source: Mplus User's Guide v8, Chapter 17
> Bundled source: references/source-pdfs/Chapter17.pdf

## One-line summary
The MODEL command's building blocks — BY, ON, WITH (and their paired forms PON/PWITH), bracket `[ ]` and `$` notation for means/intercepts/thresholds, the `*`/`@`/`(number)`/`(name)` symbols, list ("shorthand") notation like `y1-y5`, the `|` symbol for growth/random-effect/interaction language, MODEL INDIRECT, MODEL CONSTRAINT, MODEL TEST, MODEL PRIORS, and the `MODEL label:` / `%OVERALL%` / `%class%` / `%WITHIN%` / `%BETWEEN%` block variations — are the common syntax vocabulary used across every other Mplus procedure file in this skill. Consult this file whenever a question is about "how do I write this piece of MODEL syntax" rather than "which whole procedure do I need."

## Prerequisite checklist
- [ ] Know which variables are observed vs. latent, and dependent vs. independent, in the diagram/hypothesis (this determines whether BY, ON, or WITH is needed)
- [ ] Know the exact, in-order variable list from `VARIABLE: NAMES ARE ...;` / `USEVARIABLES ARE ...;` — the hyphen list function (`y1-y5`) depends entirely on this declared order, not on visual/alphabetical order
- [ ] Decide whether any parameter needs a **label** (needed for MODEL CONSTRAINT, MODEL TEST, ESTIMATOR=BAYES/MODEL PRIORS) — if so, plan parameter names (≤8 characters, start with a letter, letters/numbers/underscore only)
- [ ] For mixture/multilevel/multiple-group runs, know which parts of the model are common (`%OVERALL%`, `%WITHIN%`) vs. group/class-specific (`%class%`, `%BETWEEN%`, `MODEL label:`)
- [ ] For MODEL INDIRECT, confirm whether conventional (product-of-coefficients) or counterfactually-defined (causal, exposure/mediator/moderator) effects are wanted

## Option selection logic
| Situation | Choice |
|---|---|
| Define a continuous latent variable (factor) from its indicators | `f1 BY y1-y5;` |
| Regress one variable (observed or latent) on others | `y ON x;` (or `f1 f2 ON x1-x9;` for several DVs on the same predictors) |
| Regress a long, ordered chain of variables each on the next variable in a matched list | `PON` — pairs left-hand list with right-hand list element-by-element |
| Free a covariance/correlation between two variables | `f1 WITH f2;` |
| Free covariances between two matched lists element-by-element | `PWITH` |
| Refer to a variance or residual variance | plain variable list, no brackets: `y1 y2 y3;` |
| Refer to a mean, intercept, or threshold | bracket notation: `[y1 y2 y3];` or `[u1$1 u1$2];` |
| Free a parameter (from its fixed default) and/or give it a starting value | `*` : `y1* y2*.5;` |
| Fix a parameter at a specific value | `@` : `y1@0 f1@1;` |
| Constrain two or more parameters to be equal | `(number)` after each: `f1 ON x1 (1); f2 ON x1 (1);` |
| Give a parameter a name for later reference (MODEL CONSTRAINT/TEST/PRIORS) | `(name)`: `y ON x1 (p1);` |
| Refer to a scale factor (categorical-variable latent response scaling, TYPE=GENERAL) | `{list}`: `{u1 u2 u3};` |
| Specify a growth model, random slope, random factor loading, random variance, or latent-variable interaction | `\|` symbol |
| Request indirect/direct/total effects (mediation) | `MODEL INDIRECT:` with `IND` (no moderation) or `MOD` (moderation) |
| Impose linear/non-linear constraints across parameters, or derive new quantities from parameters | `MODEL CONSTRAINT:` with `NEW` |
| Run a Wald chi-square test of restrictions on labeled parameters | `MODEL TEST:` |
| Set a Bayesian prior distribution on a parameter (ESTIMATOR=BAYES) | `MODEL PRIORS:` |
| Give each group its own model deviations (multiple-group analysis) | `MODEL label:` (label from `GROUPING=` or per-group file) |
| Give each latent class its own model (mixture model) | `%OVERALL%` (shared part) + `%class label%` (e.g. `%c#1%`) blocks inside `MODEL:` |
| Specify individual- vs. cluster-level parts (multilevel) | `%WITHIN%` / `%BETWEEN%` (or `%BETWEEN label%` for 3-level/cross-classified) |
| Provide population values for Monte Carlo data generation | `MODEL POPULATION:` / `MODEL COVERAGE:` / `MODEL MISSING:` (parallel structure to MODEL) |

## Menu path & screen fields
Mplus is a text .inp syntax file, not a GUI. This section is organized as a categorized syntax reference for the pieces that recur across every procedure-specific file in this skill.

### 1. The three core relational keywords
- **BY** — "measured by"; defines continuous latent variables (factors) from their indicators. `f1 BY y1-y5;`
  - First variable's loading is fixed at 1 by default (sets the factor's metric). To free it instead and fix the factor variance, do `f1 BY y1* y2-y5; f1@1;`
  - Indicators can be continuous, censored, binary/ordinal, count, or the inflation part of censored/count variables, in any combination
  - Second-order factors: `f3 BY f1 f2;` is allowed only after f1 and f2 are themselves defined by earlier BY statements (a factor cannot appear on the right of BY before being defined on the left of one)
  - **ESEM (exploratory)**: `f1-f2 BY y1-y5 (*1);` defines a *set* of EFA factors (rotated together); the `(*1)` label after an asterisk marks the set. Same label across BY statements ties two sets to the same rotated matrix and factor-loading equality (`(*1 1)`). `TARGET` rotation values use `~`: `f1 BY y1-y5 y1~0 (*1);`
- **ON** — "regressed on"; defines regression relationships. `y ON x1 x3;` (multiple predictors = multiple regression). Works for observed-on-observed, latent-on-observed, latent-on-latent, and any combination.
  - For a categorical latent variable c with 3 classes: `c#1 c#2 ON x1-x3;` (or simply `c ON x1-x3;`) — one ON statement per class except the reference (last) class
  - For a nominal observed variable u with 3 categories: `u#1 u#2 ON x1-x3;` (or `u ON x1-x3;`) — same reference-category logic
  - **PON** (paired ON) — pairs left-hand list with right-hand list element-by-element: `y2 y3 y4 PON y1 y2 y3;` implies `y2 ON y1; y3 ON y2; y4 ON y3;`. Cannot be used with the simplified categorical-latent/nominal shorthand.
- **WITH** — "correlated with"; defines covariances/residual covariances. `f1 WITH f2;`
  - Works for continuous observed & latent variables; with WLS-family estimators, also for binary/ordinal/censored observed variables
  - Crossing: `y1 y2 y3 WITH y4 y5 y6;` frees **all 9** cross-covariances (3×3)
  - Two categorical latent variables: `c1 WITH c2;` — association coefficient for the last class of each is fixed at 0 by default (loglinear-style)
  - `NOCOVARIANCES` (ANALYSIS: MODEL option) fixes all covariances/residual covariances at 0 by default; WITH is then used to selectively free them
  - **PWITH** (paired WITH) — pairs left/right lists element-by-element: `y1 y2 y3 PWITH y4 y5 y6;` implies `y1 WITH y4; y2 WITH y5; y3 WITH y6;` (only 3 covariances, not 9). Cannot be used with categorical-latent-variable shorthand.

### 2. Variances, means/intercepts/thresholds
- A bare variable list (no brackets) refers to **variances** (independent variables) or **residual variances** (dependent variables): `y1 y2 y3;`
- A **bracketed** list `[ ... ];` refers to **means** (independent vars), **intercepts** (dependent vars), or **thresholds** (categorical dependent vars): `[y1 y2 y3];`
- Thresholds are labeled `variable$number`: a 4-category variable y1 has 3 thresholds, `y1$1 y1$2 y1$3` (lowest to highest). `[y1$1 y1$2 y1$3];`
- Threshold sign is opposite an intercept's: a threshold of -0.5 ≡ an intercept of 0.5 for the same variable
- Defaults: continuous/censored residual variances free; categorical observed variables have no variance parameter (fixed scale) except under Theta parameterization; nominal/count/categorical-latent variables have no variance parameters at all
- In single-group models, continuous latent variable means/intercepts are fixed at 0 by default. In multiple-group models they're fixed at 0 in the first group, free elsewhere. In mixture models they're fixed at 0 in the last class, free elsewhere (same rule for categorical latent variable means).

### 3. Freeing, fixing, constraining, labeling (the `*` `@` `(number)` `(name)` symbols)
- `*` — frees a parameter at its default starting value, or with a value attached (`y2*.5`) sets a custom starting value while freeing it. Can follow any parameter type (loadings, regressions, covariances, variances, means/intercepts/thresholds, scale factors).
- `@` — fixes a parameter at a specific value: `y1@0`, `f1@1`, `f1 WITH f2@0;`
- `(number)` — constrains all parameters sharing that number to be equal: `y1 ON x1 (1); y2 ON x2 (1);` ties those two slopes together. Only **one** number in parentheses is allowed per line; a statement continued across lines needs the number restated at the end of each line.
- `(name)` — assigns a text label (≤8 chars, starts with a letter, letters/digits/underscore) to a parameter, used later by MODEL CONSTRAINT, MODEL TEST, or MODEL PRIORS: `y ON x1 (p1) x2 (p2);`
- **Special "last mention wins" rule**: if a parameter is mentioned more than once on the right-hand side of a BY/ON/WITH statement (or in a variance/mean/scale-factor list), the program uses the **last** specification given — handy for setting one common value then overriding a few exceptions, e.g. `f1 BY y1-y6*0 y5*.5;` → all loadings start at 0 except y5, which starts at 0.5.

### 4. List (shorthand) notation
- A hyphen between two variable names lists every variable between them **in the order VARIABLE: NAMES/USEVARIABLES declared them** — e.g. `f1 BY y1-y4;` = `f1 BY y1 y2 y3 y4;`
- For **latent** variables, list order follows the order the BY/`|` statements defining them appear in the MODEL command (factors from BY first, then `|`-defined random effects, in the order they occur)
- Works on both sides of ON and WITH, and the right-hand side of BY; a list on the left implies multiple statements, a list on the right implies a list of variables
- List function + equality: `f1 BY y1-y5 (1) y6-y10 (2);` ties loadings within each group to a common value
- List function + individual labels: `[y1-y5] (p1-p5);` assigns p1→y1, p2→y2, ... p5→y5 (a label list, not a single equality)
- When lists appear on **both** sides of ON/WITH, Mplus needs a matching list of constraints/labels of size (n-left × n-right): `y1-y3 ON x1-x2 (1-2 3-4 5-6);` produces 6 separate equality-tagged regressions
- A list of equality constraints/labels cannot be used with a list of *individually specified* parameters — only with the shorthand list form

### 5. Labeling categorical entities & special variable types
- Categorical latent variable classes: `c#1`, `c#2`, ... (last class = reference, all parameters fixed at 0)
- Nominal observed variable categories: `u#1`, `u#2`, ... (last category = reference)
- Thresholds: `variable$number` (see above)
- Inflation part of a censored or count variable: `variable#1` (e.g. `y1#1` for the inflation part of censored y1)
- Baseline hazard parameters (continuous-time survival, `BASEHAZARD` ON): `t#1, t#2, ..., t#(k+1)` for k time intervals

### 6. Scale factors
- `{list}` refers to scale factors for latent-response-variable scaling under TYPE=GENERAL: `{u1 u2 u3};` — free parameters, default starting value 1. Used e.g. to relax across-time/across-group equality of categorical latent-response-variable variances in growth/multiple-group models, or to fit a correlation-structure model for continuous variables using SDs as scale factors.

### 7. The `|` symbol — growth factors, random slopes/loadings/variances, interactions
- **Growth models**: `i s | y1@0 y2@1 y3@2 y4@3;` names growth factors (i, s, ...) on the left and gives outcome + time scores on the right; equivalent to writing out BY/bracket statements manually (see table below).
- **AT** (with TYPE=RANDOM): individually-varying observation times — `i s | y1-y4 AT t1-t4;` where t1-t4 are data variables holding each person's timepoints. Number of names left of `|` (1–4) determines intercept-only / linear / quadratic / cubic growth.
- **Random slopes** (with TYPE=RANDOM): `s | y ON x;` names s as the random slope of y on x. An asterisk after the slope name (`s* | y ON x;`) allows variation on both within and between levels for TYPE=TWOLEVEL (otherwise between-level only). Lists work: `s1-s3 | y1-y3 PON x1-x3;`.
- **Random factor loadings** (TYPE=TWOLEVEL/CROSSCLASSIFIED with TYPE=RANDOM): `s1-s10 | f BY y1-y10;` — loadings on the right of BY cannot use `*`/`@` here.
- **Random variances** (TYPE=TWOLEVEL, ESTIMATOR=BAYES only): `logv | y;` names logv as the (log-scale) random residual variance of y.
- **XWITH** (TYPE=RANDOM): defines interactions between two continuous latent variables, or a continuous latent variable and an observed variable: `int | f1 XWITH f2;`. `fsq | f XWITH f;` gives the square of a latent variable. Interaction variables may appear only on the right-hand side of ON statements. Not available for TYPE=THREELEVEL/CROSSCLASSIFIED. See the interaction-options table below for other variable-type combinations (observed×observed → DEFINE; categorical latent × anything → MIXTURE).

| Types of variables interacting | How to obtain the interaction |
|---|---|
| observed continuous × observed continuous | `DEFINE` (VARIABLE command) |
| observed categorical × observed continuous | `DEFINE`, or multiple-group |
| observed continuous × continuous latent | `XWITH` |
| observed categorical × continuous latent | `XWITH`, or multiple-group |
| observed continuous/categorical × categorical latent | `MIXTURE` (KNOWNCLASS for observed-categorical case) |
| continuous latent × continuous latent | `XWITH` |
| continuous latent × categorical latent | `MIXTURE` |
| categorical latent × categorical latent | `MIXTURE` |

### 8. Growth-model quick table (continuous outcome unless noted)
| Growth type | `\|` language | Equivalent BY/bracket language |
|---|---|---|
| Intercept only | `i \| y1-y4@1;` | `i BY y1-y4@1; [y1-y4@0 i];` |
| Linear | `i s \| y1@0 y2@1 y3@2 y4@3;` | `i BY y1-y4@1; s BY y1@0 y2@1 y3@2 y4@3; [y1-y4@0 i s];` |
| Quadratic | `i s q \| y1@0 y2@1 y3@2 y4@3;` | adds `q BY y1@0 y2@1 y3@4 y4@9;` |
| Piecewise | `i s1 \| y1@0 y2@1 y3@2 y4@2 y5@2; i s2 \| y1@0 y2@0 y3@0 y4@1 y5@2;` | two slope BY statements with the piecewise loadings |
| Linear, binary outcome (Delta) | `i s \| u1@0 u2@1 u3@2 u4@3;` | adds `[u1$1-u4$1] (1); [i@0 s]; {u1@1 u2-u4};` |
| Linear, binary outcome (Theta) | same `\|` form | adds `u1@1 u2-u4;` instead of scale factors |
| Multiple-group | same `\|` form | add `MODEL g1: [i s];` for group-specific deviations |
| Mixture | `%OVERALL% i s \| ...;` | add `%c#1% [i s];` per class for class-specific growth factor means |
| Multilevel | `%WITHIN% iw sw \| ...; %BETWEEN% ib sb \| ...;` | within-level and between-level growth factors specified separately |

Defaults differ by outcome scale: for continuous/censored/count outcomes, growth-factor means/intercepts are free by default; for binary, ordinal, the inflation part of censored/count outcomes, and multiple-indicator growth models, the **intercept** growth factor's mean is fixed at 0 (first group / last class) while slope factor means are free. Growth-factor (residual) variances/covariances are free as the default for all outcome types.

### 9. MODEL INDIRECT — mediation and causal effects
- Not available for TYPE=RANDOM, the CONSTRAINT option of VARIABLE, or TYPE=EFA
- Default SEs are delta-method; bootstrap SEs require `ANALYSIS: BOOTSTRAP =` combined with `MODEL INDIRECT:`
- Combine with `OUTPUT: STANDARDIZED;` for standardized indirect/direct effects, and `OUTPUT: CINTERVAL;` for symmetric, bootstrap, or bias-corrected-bootstrap 95%/99% CIs
- **Conventional (product-of-coefficients) effects** — `IND` and `VIA`:
  - `IND`: `y3 IND y1;` (dependent variable on the left) requests **all** indirect effects from the last right-hand variable (independent) through any path to y3, plus total and total indirect effect. Add mediators explicitly to request one specific path: `y3 IND y1 x1;` = the specific x1→y1→y3 path.
  - `VIA`: `y3 VIA y1 x1;` requests **all** indirect effects from x1 to y3 that pass through mediator y1 (i.e., a set of paths sharing a mediator, rather than one fully specified path)
- **Counterfactually-defined (causal) effects** — available for a single mediator, continuous/binary/ordinal mediator & continuous/binary/ordinal/count outcome, binary or continuous exposure:
  - `IND` (no moderation): `y IND m x;` — for a continuous exposure, give two comparison values in parentheses after x, e.g. `y IND m x (1 -1);` (default is 1 vs. 0 for a binary exposure). Adding a value after the mediator, `y IND m (2) x;`, requests the **controlled** direct effect at that mediator value.
  - `MOD` (with moderation): 3, 4, or 5 arguments after MOD depending on which variables interact with the moderator z — `y MOD m mx x;` (interaction with mediator, no moderator range plot), `y MOD m z (-1 1 0.1) mz x;` (moderator range + interaction with mediator), `y MOD m z (-1 1 0.1) xz x;` (interaction with exposure), or all five arguments together for interactions with both. The `(lower upper increment)` triple after z drives moderation plots (`PLOT: TYPE=PLOT2;` or `PLOT3`). Continuous control variables should be centered for interpretable results (indirect/direct effects are then evaluated at their means; otherwise at zero).

### 10. MODEL CONSTRAINT — linear/non-linear constraints & derived parameters
- Not available for TYPE=EFA. Uses labels from MODEL (see §3) plus:
  - `NEW (name*startvalue)`: declares a brand-new parameter not in MODEL, default start value 0.5 if omitted — `NEW (c*.6); p2 = p1 + c; p3 = p1 + 2*c;`
  - Equal sign (=), >, <, and DEFINE-command arithmetic operators/functions (no absolute-value function) build explicit constraints (`p1 = p2**2 + p3**2;`) or implicit ones (`0 = m4 - m5;`)
  - `DO (start, end) expr#...;` — do-loop; `#` is replaced by each value in range. `DO ($low,high) DO (%low,high) ...;` gives a double do-loop (two replaced symbols, e.g. `$` and `%`)
  - `CONSTRAINT = varlist;` (VARIABLE command) makes named data variables usable inside MODEL CONSTRAINT (all treated as continuous); not available for TYPE=RANDOM/TWOLEVEL/THREELEVEL/CROSSCLASSIFIED/COMPLEX or non-ML/MLR/MLF estimators

### 11. MODEL TEST — Wald chi-square test
- Uses labels from MODEL and NEW parameters from MODEL CONSTRAINT (not CONSTRAINT-option variables)
- `0 = p2 - p1; 0 = p3 - p1;` jointly tests p1=p2=p3 (2 df) — do **not** also add a redundant `0 = p3 - p2;`
- Supports the same `DO`/double-`DO` loop syntax as MODEL CONSTRAINT for building many restrictions at once

### 12. MODEL PRIORS — Bayesian prior distributions (ESTIMATOR=BAYES)
- Default is diffuse/non-informative priors per parameter type (see table below)
- Assign with `~` and a distribution code + two parameters in parentheses: `p1-p10 ~ N (1, 0.5);`
  - Normal `N` / Lognormal `LN`: (mean, variance)
  - Uniform `U`: (lower, upper)
  - Inverse Gamma `IG`: (shape, scale)
  - Gamma `G`: (shape, inverse scale)
  - Inverse Wishart `IW`: (matrix-forming value, degrees of freedom)
  - Dirichlet `D`: (observations added to referenced class, observations added to last class)
- `COVARIANCE (p1, p2) = value;` sets a normal bivariate prior covariance between two parameters (factor loadings, regression coefficients, intercepts, or binary-variable thresholds only)
- `DIFFERENCE (p1, p2) ~ N (mean, variance);` sets a normal prior on the *difference* p1 − p2 (same restricted parameter types as COVARIANCE) — useful for group/class/time-point comparisons
- `DO` loops (and double-`DO`) work the same way as in MODEL CONSTRAINT to apply one prior specification to many parameters at once, including combined with DIFFERENCE

| Parameter type | Distributions available | Default prior |
|---|---|---|
| Observed continuous DV means/intercepts (nu) | normal | N(0, ∞) |
| Observed continuous DV variances/residual variances (theta) | inverse Gamma | IG(-1, 0) |
| Observed categorical DV thresholds (tau) | normal, uniform | N(0, ∞) |
| Factor loadings (lambda) | normal | N(0, ∞) continuous vars / N(0, 5) categorical vars |
| Regression coefficients (beta) | normal | N(0, ∞) continuous vars / N(0, 5) categorical vars |
| Continuous latent variable means/intercepts (alpha) | normal | N(0, ∞) |
| Continuous latent variable (residual) variances (psi) | IG / Gamma / uniform / lognormal / normal (1 var); inverse Wishart (>1 var) | IG(-1, 0); IW(0, -p-1) for >1 |
| Categorical latent variable parameters | Dirichlet | D(10, 10) |

### 13. MODEL command block variations
- **`MODEL:`** — the overall/single-group analysis model
- **`MODEL label:`** — group-specific deviations in multiple-group analysis (label from `GROUPING=`, per-group `FILE=`, or program-assigned for summary data); also used with `%WITHIN%`/`%BETWEEN%` for multiple-group multilevel analysis
- **`MODEL:` with `%OVERALL%` / `%class label%`** — mixture models; `%OVERALL%` = shared model, `%c#1%`, `%c#2%`, ... = class-specific deviations. With **more than one** categorical latent variable, class-specific parts go in a separate `MODEL c1:` block per categorical latent variable (no `%OVERALL%` inside it), and `MODEL c1.c2:` blocks (using `%c1#1.c2#1%`) specify parameters unique to a class-combination — allowed for combinations of all-but-one categorical latent variable
- **`MODEL:` with `%WITHIN%` / `%BETWEEN%` / `%BETWEEN label%`** — multilevel models; `%WITHIN%` = individual-level, `%BETWEEN%` = cluster-level (TYPE=TWOLEVEL) or, for THREELEVEL/CROSSCLASSIFIED, `%BETWEEN label%` where the label is a CLUSTER-option variable name (e.g. level2/level3, or level2a/level2b for cross-classified)
- **`MODEL POPULATION:` / `MODEL COVERAGE:` / `MODEL MISSING:`** — Monte Carlo simulation only (paired with the MONTECARLO command); mirror the same label/`%OVERALL%`/`%class%`/`%WITHIN%`/`%BETWEEN%` structure but supply **population** parameter values (each parameter followed by `@` or `*` and a numeric value) rather than an analysis model. MODEL MISSING additionally requires the `MISSING=` option of MONTECARLO and specifies a logistic-regression missing-data-generation model per dependent variable.
- The MODEL command is required for every analysis **except** EFA, exploratory LCA, a baseline model, and TYPE=BASIC

## Sub-option details
- **BY vs. ON**: a BY statement is conceptually a set of ON statements (indicators regressed on the factor), but a continuous latent variable's *definition* can only come from BY, never from writing out the implied ON statements manually.
- **CATEGORICAL LATENT VARIABLE / NOMINAL cross-reference table** (which relationships are `ON` vs. bracket-only, condensed from the chapter's table): continuous, censored/categorical/count, and continuous-latent mediators/DVs always use `ON`; nominal mediators are NA as predictors in this table's sense; relationships to a **categorical latent** mediator/DV are specified as means/thresholds varying by class (bracket, no ON) — except when the DV *is* itself a categorical latent variable, which uses `ON` (multinomial logistic regression, §"Categorical latent variables and nominal observed variables" above).
- **PON/PWITH size rule**: the number of variables on the left must exactly equal the number on the right; each is paired positionally, not crossed.
- **Equality-constraint list mechanics**: with lists on both sides of ON/WITH, `y1-y3 ON x1-x2 (1-2 3-4 5-6);` needs 3 sub-lists of 2 (one per left-hand variable) — a single flat list of 6 numbers cannot be used unless it is literally 3 pairs like this.
- **Parameter-label list order in MODEL PRIORS**: when a label list is used (e.g. `p1-p10`), the order is **alphabetical**, not MODEL-command appearance order — a common source of mismatches.
- **Simplified categorical/nominal shorthand**: `c ON x1-x3;` and `u ON x1-x3;` are shorthand for one ON statement per class/category except the reference; the list function works with this shorthand for both PON... except PON itself and PWITH cannot be used with categorical-latent/nominal shorthand.

## Post-run operations
- This file is pure MODEL syntax; there is no independent "post-run" step of its own — pair it with `output-savedata-plot-commands.md` for what to request in OUTPUT/SAVEDATA/PLOT once MODEL syntax is written (e.g. `STANDARDIZED`/`STDYX` for indirect effects from MODEL INDIRECT, `CINTERVAL (BOOTSTRAP)` for mediation CIs, `TECH1` to confirm a MODEL command's parameter specification matches intent).
- For MODEL INDIRECT with moderation (`MOD` with 4–5 arguments), request `PLOT: TYPE=PLOT2;` or `TYPE=PLOT3;` to get the moderation plot with confidence bands.
- For MODEL CONSTRAINT + `PLOT`/`LOOP`, request `TYPE=PLOT2` in the PLOT command and use the Mplus Editor's "Loop plots" menu to view computed-value-vs.-x-axis plots with 95% CIs (or Bayes credibility intervals under ESTIMATOR=BAYES).

## Likely FAQ mapping
- "How do I define a factor / latent variable from my items?" → BY, §1
- "What's the difference between BY and ON?" → §1 (BY only defines latent variables; conceptually a special-purpose ON)
- "How do I write y1 through y20 without typing every name?" → list/hyphen notation, §4 — but warn that order follows NAMES/USEVARIABLES declaration order, not visual order
- "How do I get Mplus to print/estimate the intercept or the threshold, not just the slope?" → bracket `[ ]` notation, §2
- "My model won't identify — how do I fix a loading/variance instead of freeing it?" → `@`, §3
- "How do I force two regression coefficients to be exactly equal across groups/models?" → `(number)` equality label, §3
- "How do I get a p-value on whether two effects differ, or a nonlinear combination of parameters?" → MODEL CONSTRAINT + MODEL TEST, §10–11
- "How do I get the indirect (mediation) effect and its significance?" → MODEL INDIRECT (`IND`), §9 — point to `path-analysis-mediation-bootstrap-missing.md` for the full worked procedure
- "I need a causal/counterfactual mediation effect with an exposure×mediator interaction" → MODEL INDIRECT `MOD` option, §9
- "How do I specify growth factors without typing out all the BY/bracket statements?" → the `|` symbol + growth table, §7–8
- "How do I let a slope vary randomly across people/clusters?" → `|` random slopes, §7
- "How do I add a latent interaction term (moderation) to my SEM?" → `XWITH`, §7 and the interaction-options table
- "This is a Bayesian model — how do I set an informative prior?" → MODEL PRIORS, §12
- "I have several latent classes — how do I give each class different parameters?" → `%OVERALL%` / `%class%` blocks, §13
- "This is a two-level/multilevel model — where do I put within- vs. between-level paths?" → `%WITHIN%` / `%BETWEEN%`, §13
- "I'm running a multiple-group model and need group-specific paths" → `MODEL label:`, §13

## Deliberately skipped
- The chapter's Monte Carlo `MODEL POPULATION:` / `MODEL COVERAGE:` / `MODEL MISSING:` blocks are summarized only briefly (§13) since they are simulation-only and not part of a real-data analysis model — a dedicated Monte Carlo procedure file would be the place for a full worked example.
- Continuous-time survival BASEHAZARD labeling is mentioned only in the variable-labeling table (§5); full survival-model MODEL syntax belongs in a dedicated survival-analysis procedure file.

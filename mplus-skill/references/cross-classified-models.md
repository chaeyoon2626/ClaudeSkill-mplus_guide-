# Cross-Classified Models (Non-Nested Grouping Structures)

> Source: Mplus User's Guide v8, Chapter 9, Examples 9.24, 9.25, 9.26, 9.27
> Bundled source: references/source-pdfs/Chapter9.pdf

## One-line summary
Models individual-level observations that are simultaneously grouped by **two grouping variables that are not nested in each other** (e.g. students cross-classified by both neighborhood and school, or item responses cross-classified by subject and item), using `TYPE=CROSSCLASSIFIED` with one `%BETWEEN <label>%` block per grouping variable and Bayesian estimation only.

## Prerequisite checklist
- [ ] **Confirm the two grouping variables are genuinely non-nested** (every combination of the two groupings is possible, e.g. a student's neighborhood does not determine their school) — if one grouping is nested inside the other, use `three-level-models.md` instead
- [ ] Identify which grouping variable is the "fastest moving" one relative to the individual-level rows (varies most often across consecutive rows) — this must be listed **first** on `CLUSTER=`
- [ ] Is each individual-level covariate/outcome measured once per row, or is this a **long-format** setup (e.g. one row per subject-by-item combination, or per subject-by-time combination) built specifically to exploit cross-classification for IRT- or growth-style models?
- [ ] Are any random slopes needed at the individual (within) level, and if so, should they vary across **one** of the two groupings, **both**, or be regressed on covariates from each grouping separately (not cascaded, since the two groupings are not nested)?
- [ ] **Accept that Bayesian estimation is required** — `ESTIMATOR=BAYES` is the only estimator available for `TYPE=CROSSCLASSIFIED`, so plan for MCMC diagnostics rather than a likelihood-ratio chi-square test of fit

## Option selection logic
| Situation | Choice |
|---|---|
| Continuous DV regressed on individual-level covariates, with each of the two non-nested groupings contributing its own random intercept/slope prediction | `TYPE = CROSSCLASSIFIED RANDOM;` with `%WITHIN%` regressions, then a separate `%BETWEEN level2a%` block and `%BETWEEN level2b%` block, each independently regressing the (same-named) random intercept/slope on its own covariate (Example 9.24) |
| Path model with two continuous DVs/mediators, no random slopes needed (only random intercepts) | `TYPE = CROSSCLASSIFIED;` (no `RANDOM`), `ESTIMATOR = BAYES;` — note this estimator is mandatory, not just a computational convenience (Example 9.25) |
| Item response data in long format (one row per subject-by-item pair) — cross-classified IRT | `CLUSTER = item subject;` (fastest-moving level first), `s \| f BY u;` on `%BETWEEN subject%` to define a subject-level factor with a random (item-varying) loading, and item difficulty/discrimination modeled as random effects on `%BETWEEN item%` (Example 9.26) |
| Growth/repeated-measures data reframed in long format with subject and time as two cross-classified levels, using multiple indicators per occasion | `CLUSTER = subject time;`, within-level multiple-indicator factor with random loadings (`s1-s3 \| f BY y1-y3;`), a random slope of the factor on a rescaled time score (`s \| f ON timescor;`), and separate `%BETWEEN time%` / `%BETWEEN subject%` blocks (Example 9.27) |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE = ...;**
3. **VARIABLE:**
   - `NAMES ARE ...;`
   - `CATEGORICAL = u;` — for a categorical/binary response (e.g. IRT item data)
   - `CLUSTER = level2b level2a;` (or e.g. `item subject;`, `subject time;`) — **two** non-nested cluster ID variables; per the manual, the **fastest-moving** level (varies most often row-to-row) is listed first
   - `WITHIN = x1 x2;` — individual-level-only covariates; a variable can also be given a level label in parentheses (e.g. `WITHIN = timescor (time) y1-y3;`) to indicate it varies at level 1 **and** at one specific cross-classified level but has no variance at the other
   - `BETWEEN = (level2a) w (level2b) z;` — each cluster-level covariate is tagged with exactly which of the two non-nested groupings it belongs to; a variable can be modeled on only one of the two classifications, never cascaded to the other
4. **DEFINE:** (optional) e.g. `timescor = (time-1)/100;` — to rescale/center a time or index variable used as a within-level covariate
5. **ANALYSIS:**
   - `TYPE = CROSSCLASSIFIED;` — sufficient for models using only random intercepts (no `| DV ON covariate;`/`| factor BY indicator;` random-slope syntax)
   - `TYPE = CROSSCLASSIFIED RANDOM;` — required as soon as any random slope/loading is named via `|`
   - `ESTIMATOR = BAYES;` — **mandatory**; no other estimator is available for `TYPE=CROSSCLASSIFIED`
   - `PROCESSORS = 2;` — speeds up MCMC computation across chains if multiple processors are available
   - `BITERATIONS = (2000);` — sets the minimum MCMC iterations per chain (maximum stays the default 50,000) under the PSR convergence criterion
6. **MODEL:**
   - `%WITHIN%` block: individual-level regressions/measurement model (can be empty, as in the IRT example, if all structure is at the two between levels)
   - `%BETWEEN <label1>%` block: regressions of random intercepts/slopes on covariates specific to grouping 1, plus `WITH` statements for residual correlations within that grouping
   - `%BETWEEN <label2>%` block: the parallel, independent block for grouping 2 — this is **not** nested inside grouping 1's block, unlike three-level `%BETWEEN level3%`
7. **OUTPUT:** `TECH1 TECH8;` — recommended to confirm parameter specification/starting values and monitor MCMC progress

## Sub-option details
- `CLUSTER=` ordering: the manual states the "fastest moving level" must come first (demonstrated explicitly for the item-response example, where `item` is listed before `subject` because items vary faster than subjects across consecutive rows in that data layout). Verify the intended row-to-row variation pattern of a new data set before deciding the order — the guide does not give a universal rule beyond this stated case.
- `WITHIN=` with a parenthesized level label (e.g. `timescor (time)`): declares that the variable is measured at level 1 **and** has variance at one specific cross-classified level, but not the other — distinct from the unlabeled case (level 1 only) and the omitted case (variance at both cross-classified levels).
- `BETWEEN = (level2a) w (level2b) z;`: parallels the three-level `BETWEEN` labeling syntax, but here each label refers to one of two **independent, non-nested** classifications rather than a hierarchy — a variable measured on classification A is modeled only within classification A's `%BETWEEN%` block and never cascades into classification B's block (there is no "slope of a slope" analog across the two classifications, since neither contains the other).
- Two `%BETWEEN%` blocks for the same random effect: in Example 9.24, the same random intercept/slope names (`y`, `s`) are regressed on covariates in **both** `%BETWEEN level2a%` and `%BETWEEN level2b%`, with `WITH` statements in each block to free their residual covariance within that classification — the two blocks' contributions to `y`/`s` are additive, not hierarchical.
- Estimator restriction: for `TYPE=CROSSCLASSIFIED`, `ESTIMATOR=BAYES` is not just the default but the **only** available estimator, per the manual's explicit statement in Example 9.25 — plan model checking around posterior summaries and MCMC diagnostics (`TECH8`) rather than a chi-square/robust-SE workflow.
- Cross-classified IRT layout (Example 9.26): the data is in long format with one row per subject-by-item combination and a single response variable `u`; the `%WITHIN%` block is empty; `s | f BY u;` on `%BETWEEN subject%` names a random factor loading (item-specific discrimination, viewed from the subject side) defining a subject-level ability factor `f`, with `f@1;` fixing the factor variance and `u@0;` fixing the residual variance of `u` (Theta parameterization convention for a binary/ordinal item); on `%BETWEEN item%`, `u; [u$1];` free the item's threshold (difficulty) and `s; [s];` free the mean and variance of the random loading, allowing item discrimination to vary by item.
- Cross-classified long-format growth (Example 9.27): reframes a multiple-indicator growth model as cross-classified data with `subject` and `time` as the two classifications instead of the usual two-level "occasions as separate variables" approach; `DEFINE: timescor = (time-1)/100;` rescales/centers the raw time index before it is used as a within-level covariate driving a random slope (`s | f ON timescor;`) of the growth factor on time.
- Bayesian iteration controls (`BITERATIONS=(n)`) and `PROCESSORS=` behave identically to their usage in two-level and three-level Bayesian examples elsewhere in this chapter — see `two-level-growth-longitudinal.md` and `three-level-models.md` for the same conventions.

## Post-run operations
- Because `ESTIMATOR=BAYES` is mandatory, evaluate convergence via `TECH8`'s MCMC history/PSR values rather than expecting a maximum-likelihood chi-square fit statistic; increase `BITERATIONS` if PSR has not stabilized.
- Interpret each `%BETWEEN <label>%` block's coefficients as the independent contribution of that grouping to the random intercept/slope — do not assume one classification's effect is conditional on or nested within the other's.
- For the cross-classified IRT setup, interpret the `%BETWEEN item%` estimates as item difficulty (`[u$1]`) and item discrimination (mean/variance of `s`), and the `%BETWEEN subject%` factor `f` as ability — a standard IRT interpretation implemented through cross-classification rather than a conventional single-level IRT command.
- For the cross-classified growth setup, interpret the random slope `s` (factor `f` regressed on `timescor`) as the growth rate, checking its variance at whichever level(s) it was allowed to vary for evidence of meaningful individual or occasion-level heterogeneity.
- Confirm `TECH1` shows the two `%BETWEEN%` blocks as separate, non-cascading sets of parameters — if a variable's placement in `WITHIN=`/`BETWEEN=` produces variance at the wrong classification, revisit the labeling rules before interpreting results.
- If sampling weights or a stratified survey design are also involved on top of this cross-classified structure, note that complex-survey corrections are demonstrated in this guide for `TYPE=COMPLEX`, `TYPE=TWOLEVEL`, and `TYPE=THREELEVEL`, not `TYPE=CROSSCLASSIFIED` — see `complex-survey-design-multilevel.md` for what is supported.

## Likely FAQ mapping
- "My two grouping variables aren't nested in each other (e.g. neighborhood and school)" → this file; `TYPE=CROSSCLASSIFIED`, one `%BETWEEN%` block per grouping
- "How do I order my two CLUSTER= variables for a cross-classified model?" → list the faster-moving grouping first (Example 9.26's explicit rule)
- "I want a regression/path model with two independent sources of clustering" → Example 9.24/9.25 patterns
- "I have item-response data with subjects crossed with items and want an IRT-style model" → Example 9.26 pattern (long-format cross-classified IRT)
- "I want a growth/repeated-measures model but my data is naturally long-format with subject and time as separate grouping variables" → Example 9.27 pattern
- "Can I use maximum likelihood for a cross-classified model?" → no, `ESTIMATOR=BAYES` is the only available estimator for `TYPE=CROSSCLASSIFIED`
- "My two levels ARE nested, not crossed" → not this file, see `three-level-models.md`
- "I also have survey weights or sampling stratification" → complex-survey corrections are not demonstrated for `TYPE=CROSSCLASSIFIED` in this chapter; see `complex-survey-design-multilevel.md` for what is covered

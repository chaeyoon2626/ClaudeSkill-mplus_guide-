# CFA with Mixed Indicator Types & Constrained Mean/Threshold Structure

> Source: Mplus User's Guide v8, Chapter 5, Examples 5.3, 5.9, 5.10
> Bundled source: references/source-pdfs/Chapter5.pdf

## One-line summary
Extends the basic CFA measurement model in two directions: (a) letting a single CFA mix continuous and categorical indicators in the same model (Ex 5.3), and (b) testing whether several indicators are "equivalent test forms" by constraining their loadings, and then their intercepts (continuous, Ex 5.9) or thresholds (categorical, Ex 5.10), to be equal.

## Prerequisite checklist
- [ ] Confirm indicator types: is this genuinely a *mix* of continuous and categorical indicators within one model? If all indicators are one type, use `cfa-continuous.md` or `cfa-categorical-mixed.md` instead
- [ ] Is the goal a plain measurement model, or specifically to test whether several indicators are equivalent/parallel measures of the same thing (an "equivalent test forms" hypothesis)?
- [ ] For an equivalence test: which loadings are hypothesized equal across forms, and should intercepts (continuous) or thresholds (categorical) also be forced equal?
- [ ] Indicator type determines the location parameter that carries the equivalence hypothesis — intercepts for continuous indicators, thresholds for categorical indicators

## Option selection logic
| Situation | Choice |
|---|---|
| Some indicators continuous, some categorical, same CFA | list only the categorical ones under `CATEGORICAL ARE`; any named variable left out of that list is treated as continuous by default (Ex 5.3) |
| Want to test whether 3 continuous indicators are equivalent/parallel forms (equal loadings, equal intercepts) | fix the 2nd/3rd loadings equal to the first with `@1`, then label the intercepts equal: `[y1a y1b y1c] (1);` (Ex 5.9) |
| Same equivalence question but the indicators are categorical | fix loadings equal with `@1`, then label the thresholds equal: `[u1a$1 u1b$1 u1c$1] (1);` (Ex 5.10) |
| Estimator for the mixed or equivalence-test model | WLSMV-style robust weighted least squares is the default whenever any categorical indicator is present; ML is available via `ESTIMATOR=` but needs numerical integration for the categorical part |

## Menu path & screen fields
**[Mixed continuous + categorical indicators — 5.3]**
1. **VARIABLE:** `NAMES ARE u1-u3 y4-y6;` `CATEGORICAL ARE u1 u2 u3;`
2. **MODEL:** `f1 BY u1-u3;` `f2 BY y4-y6;`

**[Mean structure equivalence, continuous — 5.9]**
1. **VARIABLE:** `NAMES ARE y1a-y1c y2a-y2c;`
2. **MODEL:**
   - `f1 BY y1a y1b@1 y1c@1;` — f1's three indicators, 2nd/3rd loadings fixed equal to the first
   - `f2 BY y2a y2b@1 y2c@1;` — same pattern for f2
   - `[y1a y1b y1c] (1);` — intercepts of f1's three forms held equal
   - `[y2a y2b y2c] (2);` — intercepts of f2's three forms held equal

**[Threshold structure equivalence, categorical — 5.10]**
1. **VARIABLE:** `NAMES ARE u1a-u1c u2a-u2c;` `CATEGORICAL ARE u1a-u1c u2a-u2c;`
2. **MODEL:**
   - `f1 BY u1a u1b@1 u1c@1;` `f2 BY u2a u2b@1 u2c@1;`
   - `[u1a$1 u1b$1 u1c$1] (1);` — thresholds of f1's three forms held equal
   - `[u2a$1 u2b$1 u2c$1] (2);` — thresholds of f2's three forms held equal

## Sub-option details
- `CATEGORICAL ARE u1 u2 u3;` with `y4-y6` left out: any variable in `NAMES` but not in `CATEGORICAL` defaults to continuous — this is the entire mechanism for mixing indicator types in one CFA (Ex 5.3). The default estimator runs probit regressions for the categorical indicators and linear regressions for the continuous ones simultaneously; `ESTIMATOR=MLR` (or similar) instead uses logistic regression via numerical integration for the categorical part, which gets more demanding as the number of factors/categorical indicators grows
- `y1b@1 y1c@1;` : fixing the 2nd and 3rd loadings to exactly 1 — the same value the first loading is fixed to by default — operationalizes "these three indicators are equivalent/parallel test forms" rather than just "these three items load on the same factor"
- `[y1a y1b y1c] (1);` : a bracket statement (intercepts) with a parenthetical label forces every listed intercept to the same value — the recurring Mplus convention that any set of parameters sharing an integer label in the same statement is held equal. The label `1` is local to that statement/model in this example, distinct from `2` used for the second factor's indicators
- `[u1a$1 u1b$1 u1c$1] (1);` : the categorical analogue — `$1` denotes the first (and, for a binary item, only) threshold; the same equal-label logic constrains thresholds instead of intercepts
- Factor means are fixed at 0 by default in both Ex 5.9 and Ex 5.10, because the intercepts/thresholds carry the location information the equivalence hypothesis is actually about
- Because the loadings are forced to an identical value, the loading pattern by itself cannot distinguish real differences between forms; only the intercept or threshold equality (or its rejection) formally tests the equivalent-forms hypothesis

## Post-run operations
- For Ex 5.9/5.10-style models, compare the fit of the equal-intercept/equal-threshold model against a version with the `(1)`/`(2)` labels removed (intercepts/thresholds freed) via a chi-square difference test — this is the actual test of the equivalent-forms hypothesis, not just eyeballing parameter estimates
- For the mixed-indicator model (Ex 5.3), confirm the categorical indicators produced threshold estimates and the continuous ones produced intercept/residual-variance estimates, matching their declared type
- Use standard `STDYX` output to interpret loadings on their usual standardized scale once the equivalence structure is set

## Likely FAQ mapping
- "My factor has some Likert items and some continuous items in the same model" → Ex 5.3 pattern: list only the categorical ones under `CATEGORICAL`
- "I want to test if three parallel test forms are truly interchangeable" → Ex 5.9 (continuous) / Ex 5.10 (categorical): fix loadings equal, then test intercept/threshold equality
- "How do I force two or more parameters to be equal in Mplus?" → give them the same integer label in parentheses within the same (or a related) statement
- "What's the difference between `@` and `*` after a loading?" → `@` fixes the parameter to the given value (e.g. `y1b@1`); `*` frees an otherwise-fixed parameter (e.g. the default-fixed first loading)

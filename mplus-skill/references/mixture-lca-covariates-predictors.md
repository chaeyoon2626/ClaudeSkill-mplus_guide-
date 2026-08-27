# LCA/LPA with Covariates Predicting Class Membership (and Direct Effects)

> Source: Mplus User's Guide v8, Chapter 7, Example(s) 7.1, 7.2, 7.12
> Bundled source: references/source-pdfs/Chapter7.pdf

## One-line summary
Adds one or more covariates (x) to a mixture/LCA/LPA model so that class membership itself is predicted by the covariate(s) through a multinomial logistic regression of C on the covariate(s), and/or so that a covariate has a "direct effect" on one specific indicator in addition to (or instead of) affecting class membership.

## Prerequisite checklist
- [ ] Decide first whether class enumeration (deciding k) should be done WITHOUT covariates, then covariates added only after k is fixed (recommended practice; see mixture-lpa-lca-cross-sectional.md for the enumeration step itself)
- [ ] Identify which covariates are hypothesized to predict class membership (goes into `c ON x...;`) versus which are hypothesized to have a direct effect on a specific indicator (goes into `indicator ON x;`)
- [ ] Confirm which class is the reference class for the multinomial logistic regression — Mplus estimates one `c ON x` equation per class comparison against the reference class
- [ ] For count outcomes with excess zeros, decide whether the "extra zero" process itself should be modeled as a 2-class mixture (zero-inflated Poisson mixture regression) rather than as a single-class zero-inflated Poisson model

## Option selection logic
| Situation | Choice |
|---|---|
| Want covariate(s) to predict which class a person falls into | `c ON x1 x2;` in `%OVERALL%` — multinomial logistic regression of `c` on the covariate(s) |
| Want to test whether one indicator's relationship with a covariate goes beyond what class membership already explains ("direct effect") | Add `indicator ON x;` in `%OVERALL%` (held equal across classes as the default) |
| Want an indicator's regression on a covariate to differ across classes | Add `indicator ON x;` inside the class-specific `%c#k%` statement instead of `%OVERALL%` |
| Outcome variable is a count with excess zeros and the "extra zero" process should be its own 2-class mixture | `COUNT = u (i);` (the `i` requests a zero-inflated Poisson) then `u ON x1 x2;` (Poisson part) and `u#1 ON x1 x2;` (logistic part for the inflation) in `%OVERALL%` |
| Want the regression of a continuous outcome on a covariate to vary by class | Put `y ON x2;` (or similar) inside `%c#2%` etc. instead of `%OVERALL%` |
| Want an alternative way to refer to one specific class-comparison coefficient (e.g. for starting values/constraints) | `c#1 ON x1;` refers to the class-1-vs-reference-class equation specifically |

## Menu path & screen fields
1. **VARIABLE:**
   - `NAMES ARE y x1 x2;` (or `u x1 x2` for count outcomes, or `u1-u4 x` for categorical LCA indicators plus one covariate)
   - `CLASSES = c (2);`
   - `COUNT = u (i);` — only for zero-inflated Poisson mixture regression (Example 7.2); `i` in parentheses requests zero-inflation
   - `CATEGORICAL = u1-u4;` — only if the class indicators themselves are categorical (LCA)
2. **ANALYSIS:** `TYPE = MIXTURE;`
3. **MODEL:**
   - `%OVERALL%` block: outcome-on-covariate regressions that hold across classes, plus `c ON x...;` for the covariate(s) predicting class membership, plus any direct effect such as `u4 ON x;`
   - `%c#2%` (etc.) block: any regression coefficients that should be freed to vary by class, e.g. `y ON x2;`
4. **OUTPUT:** `TECH1 TECH8;` (as in the base class-enumeration file)

## Sub-option details
- `c ON x1;` : multinomial logistic regression of the categorical latent variable c on covariate x1; the intercept in this regression is estimated as the default; the coefficient describes how a one-unit change in x1 changes the log-odds of being in one class versus the reference class
- `c#1 ON x1;` : alternative way to refer to the regression for a specific class comparison (class 1 vs. the reference class), useful for giving starting values or placing restrictions on individual class-comparison coefficients
- `u4 ON x;` (direct effect, Example 7.12): describes the logistic regression of the binary indicator u4 on covariate x, over and above the indirect relationship that already runs through c; this regression coefficient is held equal across classes as the default
- `COUNT = u (i);` : the `i` in parentheses requests a zero-inflated Poisson model for count variable u; `u ON x1 x2;` is then the Poisson-part regression (predicting counts for individuals able to assume values of zero and above), and `u#1 ON x1 x2;` is the logistic regression of the binary latent inflation part (probability of being unable to assume any value except zero) on the same covariates
- Class-varying vs. class-invariant regressions: putting a statement in `%OVERALL%` holds it equal across classes by default; repeating the same left-hand-side variable inside a `%c#k%` block relaxes that equality constraint for class k only (Example 7.1's `%c#2% y ON x2;`, which lets the slope of y on x2 differ in class 2 while the class-1 slope stays at the overall value)

## Post-run operations
- Read the `c ON x` (or `c#1 ON x1`) logistic regression coefficients as log-odds; exponentiate for an odds-ratio interpretation of how the covariate shifts the odds of one class versus the reference class
- If a direct effect (e.g., `u4 ON x`) is statistically significant, the indicator's relationship with the covariate is not fully explained by class membership alone — reconsider whether that indicator is behaving as a "pure" class indicator or partly measures something else related to the covariate
- For a zero-inflated Poisson mixture (Example 7.2), examine the two sets of coefficients separately: the Poisson-part `u ON x1 x2;` describes the count process among those who can have counts above zero, while `u#1 ON x1 x2;` describes what predicts being "stuck" at zero
- Compare a model with `c ON x` against the same model without it to check whether adding covariates changes the class solution's substantive meaning — best practice is to finalize the number of classes on the unconditional (covariate-free) model first, then add covariates afterward

## Likely FAQ mapping
- "I want to see whether a variable predicts which class someone is in" → `c ON x;` in `%OVERALL%`, interpret as a multinomial logistic regression
- "Should I add covariates before or after choosing the number of classes?" → recommend finalizing class count using the unconditional model first (see mixture-lpa-lca-cross-sectional.md), then add covariates
- "One of my indicators seems related to a covariate even after controlling for class" → that is a direct effect; specify `indicator ON x;` and test it; note it's held equal across classes as the default unless placed in a class-specific block
- "My count variable has too many zeros and I want the zero-generating process treated as its own class" → zero-inflated Poisson mixture regression, Example 7.2 (`COUNT = u (i);` plus `u ON ...;` and `u#1 ON ...;`)
- "I want a covariate's relationship with the outcome to differ between classes" → move that ON statement into the class-specific `%c#k%` block instead of `%OVERALL%`

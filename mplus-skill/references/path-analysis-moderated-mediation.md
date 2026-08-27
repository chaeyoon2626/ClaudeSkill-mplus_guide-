# Path Analysis — Moderated Mediation

> Source: Mplus User's Guide v8, Chapter 3, Example 3.18
> https://www.statmodel.com/HTML_UG/chapter3V8.htm

## One-line summary
Test whether the size of a mediation (indirect) effect itself changes depending on the level of a third moderator variable, and visualize how the indirect effect changes across values of the moderator.

## Prerequisite checklist
- [ ] Is the independent variable (x) → mediator (m) → dependent variable (y) path confirmed?
- [ ] Confirm exactly which path the moderator (z) moderates (typically first-stage moderated mediation on the x→m path; moderation of the m→y path can be extended with the same logic if needed)
- [ ] Since an interaction term (x*z) will be created, confirm x and z are continuous (if categorical, dummy coding plus an interaction term would be needed — this isn't covered in the manual example, so re-confirm with the user)
- [ ] What range of the moderator to show the indirect-effect change over (plot range; the manual's example default is -2 to 2 in steps of 0.1)
- [ ] Whether the user understands/is comfortable with Bayesian (MCMC) estimation — this procedure uses `ESTIMATOR=BAYES` by default

## Option selection logic
| Situation | Choice |
|---|---|
| Test whether the x→m path is moderated by z, and visualize the indirect effect across z values | create the interaction term with `DEFINE` + use LOOP/PLOT in `MODEL CONSTRAINT` |
| Prefer frequentist (ML) over Bayesian | possible, but the confidence-interval plot (PLOT) will be less smooth — switch to `ESTIMATOR=ML` and consider pairing with `CINTERVAL(BOOTSTRAP)` if needed (the manual's example is BAYES-based) |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE = ...;**
3. **VARIABLE:** `NAMES = y m x z;` / `USEVARIABLES = y m x z xz;` (the interaction term xz must be included in the variables actually used)
4. **DEFINE:** `xz = x*z;` — create the interaction term
5. **ANALYSIS:** `ESTIMATOR = BAYES;` / `PROCESSORS = 2;` / `BITERATIONS = (30000);` — specify the number of MCMC chains/iterations
6. **MODEL:**
   - `y ON m (b) x z;` — regression for the final DV, label the mediator's coefficient `(b)`
   - `m ON x (gamma1) z xz (gamma2);` — regression for the mediator, label x's coefficient `(gamma1)`, the interaction term's coefficient `(gamma2)`
7. **MODEL CONSTRAINT:**
   - `PLOT(indirect);` — designate what to plot
   - `LOOP(mod,-2,2,0.1);` — sweep the moderator value from -2 to 2 in steps of 0.1
   - `indirect = b*(gamma1+gamma2*mod);` — the formula computing the indirect effect at each moderator value (mod)
8. **PLOT:** `TYPE = PLOT2;`
9. **OUTPUT:** `TECH8;`

## Sub-option details
- `(b)`, `(gamma1)`, `(gamma2)` : parameter labels. Labels are required to reuse a parameter in a formula inside `MODEL CONSTRAINT`
- `LOOP(mod,-2,2,0.1);` : creates a new variable `mod`, sweeping it across the specified range/step and computing the `indirect` value at every point
- `PLOT(indirect);` : plots the `indirect` values computed via LOOP against the moderator value
- `BITERATIONS = (30000);` : minimum number of Bayesian MCMC iterations (can be increased after checking convergence)

## Post-run operations
- In the Mplus results viewer, go to the `PLOT` menu → `View graphs` → the loop plot shows the indirect effect and its credibility interval curve across moderator values
- If you want to separately confirm whether the `indirect` parameter is significant at a specific moderator value (e.g. the mean, ±1 SD) rather than the whole range, you can add a formula plugging in one specific value instead of using `LOOP`
- For Bayesian convergence diagnostics, recommend checking that `TECH8`'s PSR (potential scale reduction) value is close to 1

## Likely FAQ mapping
- "I want to see whether the mediation effect changes depending on a moderator" → this file
- "Can the indirect effect be shown as a graph?" → guide with the LOOP + PLOT combination
- "Why Bayesian? Can't I just use ML?" → explain the manual's example is BAYES-based as the choice for smooth confidence-interval plots, and mention the alternative if needed

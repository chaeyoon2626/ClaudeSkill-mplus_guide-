# Exploratory Factor Analysis (EFA) — Continuous Indicators

> Source: Mplus User's Guide v8, Chapter 4 (Exploratory Factor Analysis), Example 4.1
> https://www.statmodel.com/HTML_UG/chapter4V8.htm

## One-line summary
Exploratorily estimate, with no prior hypothesis, how many latent factors explain the correlations among a set of continuous observed variables, and how strongly each item loads on each factor.

## Prerequisite checklist
- [ ] Confirm the indicators (items) are all continuous (or a Likert scale that can be treated as continuous)
- [ ] What range of factor counts to explore (lower–upper bound, e.g. 1–4)
- [ ] Rotation preference: allow correlated factors (oblique, default GEOMIN) vs. assume independent factors (orthogonal)
- [ ] Is there a plan to feed this straight into a subsequent SEM model (ESEM)? (if so, write directly in ESEM syntax)
- [ ] Confirm the final desired model in structural-equation form: "how many factors, and which items are expected to group under each factor"

## Option selection logic
| Situation | Choice |
|---|---|
| Don't yet know how many factors is right, purely exploratory | `TYPE = EFA 1 4;` (specify a range, compare with chi-square difference tests) |
| Factor count already decided, and want to feed the factors straight into a structural model (regression/covariances) afterward | ESEM: `MODEL: f1-f4 BY y1-y12 (*1);` |
| Want to assume factors are uncorrelated | `ANALYSIS: ROTATION = ...(ORTHOGONAL);` (e.g. `GEOMIN(ORTHOGONAL)`) |
| Want to check residual correlations / possible extra loadings | `OUTPUT: MODINDICES;` |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE: NAMES ARE ...;**
4. **ANALYSIS:** `TYPE = EFA 1 4;` — estimates and compares 1-to-4-factor models (default estimator = ML, default rotation = oblique GEOMIN)
5. **MODEL:** (only if writing directly as ESEM) `f1-f4 BY y1-y12 (*1);`
6. **OUTPUT:** `MODINDICES;` — checks modification indices for the residual correlations that EFA fixes at zero

## Sub-option details
- `TYPE = EFA 1 4;` : sequentially estimates from a minimum of 1 up to a maximum of 4 factors. The output reports fit and chi-square difference tests (with scaling correction) across factor counts together
- `ROTATION = ...;` : defaults to GEOMIN (oblique) if unspecified. Add `(ORTHOGONAL)` for an orthogonal rotation
- ESEM (`f1-f4 BY y1-y12 (*1);`) : `(*1)` is the EFA-style notation specifying that every indicator freely loads on every factor before rotation
- `MODINDICES` : since EFA fixes residual correlations at zero by default, this shows for reference how much fit would improve if a particular correlation were freely estimated

## Post-run operations
- Deciding the factor count: guide the user to weigh chi-square difference tests, eigenvalues, and interpretability (whether the items grouped under each factor make theoretical sense) together
- In the rotated loadings table, note that items with a loading above roughly |.30–.40| are conventionally treated as representative of that factor
- If the user wants to lock this down further with CFA/SEM, mention converting to ESEM syntax or a constrained CFA — if the relevant manual (SEM/CFA chapter) isn't available yet, say so and request the PDF

## Likely FAQ mapping
- "I don't know how many factors my items group into" → EFA `TYPE=EFA 1 4`
- "Is it okay to assume the factors are correlated with each other?" → explain that oblique (allowing correlation) is the default
- "I want to use the EFA result directly in a regression" → guide with ESEM syntax

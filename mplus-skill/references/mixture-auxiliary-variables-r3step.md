# Auxiliary Variables: Testing Class Differences and the R3STEP Three-Step Approach

> Source: Mplus User's Guide v8, Chapter 7, Example 7.3 (AUXILIARY option) plus chapter overview text
> Bundled source: references/source-pdfs/Chapter7.pdf

## One-line summary
Lets you bring variables that are NOT part of the mixture measurement model into the analysis after the classes are formed, either to test whether their means differ across the classes (default `AUXILIARY` behavior, using posterior-probability-based multiple imputation) or to use them as covariates predicting class membership in a bias-corrected three-step multinomial logistic regression (`AUXILIARY` with the `R3STEP` keyword).

## Prerequisite checklist
- [ ] Finalize the number of classes first — auxiliary-variable analyses (both the default mean-equality test and R3STEP) are meant to be run after the class structure is settled, not to help choose the number of classes
- [ ] Decide the purpose: (a) test mean differences of a variable across classes → default `AUXILIARY = var;` (no keyword); (b) use variables as predictors of class membership while accounting for classification uncertainty → `AUXILIARY = var (R3STEP);`
- [ ] Confirm the auxiliary variable(s) are not also listed as class indicators (USEVARIABLES/CATEGORICAL) or already included as covariates via `c ON x` in the same MODEL
- [ ] List all auxiliary variables together in a single `AUXILIARY` statement in the VARIABLE command

## Option selection logic
| Situation | Choice |
|---|---|
| Want to test whether an external variable's mean differs across the already-decided latent classes | `AUXILIARY = var;` (no keyword) — uses posterior probability-based multiple imputation to test equality of means across classes |
| Want an external variable to serve as a predictor of class membership without letting it influence the measurement model / class formation | `AUXILIARY = var (R3STEP);` — three-step approach that accounts for classification uncertainty when estimating the multinomial logistic regression of class on the covariate |
| Want a covariate to directly influence how classes are formed (single-step, not post hoc) | Not an AUXILIARY use case — instead include it as `c ON x;` inside the model estimated jointly with the indicators (see mixture-lca-covariates-predictors.md) |
| Have several candidate predictors to screen post hoc without re-running the full mixture model for each one | List them all in one statement, e.g. `AUXILIARY = x1-x10 (R3STEP);` |

## Menu path & screen fields
1. **VARIABLE:**
   - `NAMES ARE u1-u4 x1-x10;`
   - `USEVARIABLES = u1-u4;` — restrict the indicators actually used to define the mixture model
   - `CLASSES = c (2);`
   - `CATEGORICAL = u1-u4;` — for LCA (binary/ordinal indicators); omit for LPA (continuous indicators)
   - `AUXILIARY = x1-x10 (R3STEP);` — or `AUXILIARY = x1-x10;` for the default mean-equality test
2. **ANALYSIS:** `TYPE = MIXTURE;`
3. **OUTPUT:** `TECH1 TECH8 TECH10;` (TECH10 is for categorical-indicator fit diagnostics, not specific to AUXILIARY)

## Sub-option details
- `AUXILIARY = x1-x10;` (no keyword) : the listed variables are not part of the class-formation model; Mplus tests the equality of their means across the latent classes using a posterior-probability-based multiple imputation approach
- `AUXILIARY = x1-x10 (R3STEP);` : the R3STEP keyword marks the listed variables to be used as covariates in a third-step multinomial logistic regression of the categorical latent variable on those covariates, following the bias-correcting three-step method (Vermunt, 2010; Asparouhov & Muthén, 2012b)
- USEVARIABLES vs. AUXILIARY: only the indicators named in USEVARIABLES (together with CATEGORICAL/COUNT/etc. designations) define the latent classes; AUXILIARY variables play no role in forming the classes themselves

## Post-run operations
- For the default `AUXILIARY` (mean-equality test): read the output section reporting equality tests of means across classes; a significant result indicates the auxiliary variable differs meaningfully by class
- For `AUXILIARY (R3STEP)`: read the resulting multinomial logistic regression coefficients (of c on the R3STEP covariates) in the output — these already correct for classification uncertainty in assigning individuals to classes, unlike naively exporting most-likely class and regressing on it separately
- Because both approaches are meant for a model whose class count is already decided, do not use AUXILIARY results to argue for a different number of classes — re-run the enumeration step (TECH11/TECH14/BIC, see mixture-lpa-lca-cross-sectional.md) instead if that decision needs revisiting

## Likely FAQ mapping
- "I want to know if my classes differ on an outcome I didn't use to form them" → `AUXILIARY = var;` (default, no keyword), read the equality-of-means test
- "I want to know what predicts class membership without disturbing my measurement model" → `AUXILIARY = var (R3STEP);`
- "What's the difference between putting x in `c ON x` versus `AUXILIARY = x (R3STEP)`?" → `c ON x` lets x directly help shape the classes as part of the single-step model; `AUXILIARY (R3STEP)` keeps the classes fixed as already estimated and only afterward relates x to class membership, correcting for classification uncertainty
- "Can I list many variables at once to auxiliary-test them?" → yes, `AUXILIARY = x1-x10 (R3STEP);` applies R3STEP to every listed variable in a single statement

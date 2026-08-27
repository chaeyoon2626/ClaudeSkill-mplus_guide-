# CFA/EFA with MODEL CONSTRAINT — Derived Parameters & Inequality Restrictions

> Source: Mplus User's Guide v8, Chapter 5, Examples 5.20, 5.28
> Bundled source: references/source-pdfs/Chapter5.pdf

## One-line summary
Uses `MODEL CONSTRAINT` on top of an ordinary measurement model to define new derived parameters as functions of labeled model parameters (e.g. reliability, hand-computed standardized coefficients, with their own standard errors), to force parameters equal via an explicit algebraic constraint, and to impose inequality restrictions (a residual variance must exceed zero, or exceed another named parameter) — demonstrated in a single-group CFA (Ex 5.20) and, via a `DO` loop, across every residual variance of a 10-indicator EFA/ESEM model (Ex 5.28).

## Prerequisite checklist
- [ ] Confirm the base measurement model (CFA or EFA/ESEM) is already specified — this file covers the `MODEL CONSTRAINT` layer added on top, not the base `BY` statements
- [ ] Is the goal to (a) compute a derived quantity with its own SE, (b) force two parameters equal via an algebraic constraint instead of a shared label, or (c) impose an inequality (stay positive, stay above another parameter)?
- [ ] Does the same constraint need to be applied repeatedly across many parameters (e.g. every residual variance in a 10-item EFA)? — if so, plan on a `DO` loop rather than writing it out N times
- [ ] Do any parameters referenced in `MODEL CONSTRAINT` need to come from outside the analysis model entirely (an external variable not in `USEVARIABLES`)? That specific case (a per-observation covariate feeding into a constraint) is covered separately in `twin-behavioral-genetics-ace-qtl.md`

## Option selection logic
| Situation | Choice |
|---|---|
| Compute a new quantity (e.g. reliability) as a function of labeled model parameters, with its own SE | `MODEL CONSTRAINT: NEW(rel2); rel2 = lam2**2*vf1/(lam2**2*vf1+ve2);` |
| Force two already-labeled/derived parameters to be exactly equal, as an explicit constraint rather than a shared label | `MODEL CONSTRAINT: 0 = stan6 - stan3;` (or equivalently `rel5 = rel2;`) |
| Keep a variance from crossing zero (avoid a Heywood case) | `MODEL CONSTRAINT: ve4 > 0;` |
| Keep one named parameter larger than another named parameter | `MODEL CONSTRAINT: ve2 > ve5;` |
| Apply the same inequality to every indicator's residual variance in a many-indicator EFA/ESEM | label all residual variances (`y1-y10 (v1-v10);` in MODEL), then `MODEL CONSTRAINT: DO(1,10) v#>0;` |

## Menu path & screen fields
**[CFA with parameter constraints — 5.20]**
1. **VARIABLE:** `NAMES ARE y1-y6;`
2. **MODEL:**
   - `f1 BY y1 y2-y3(lam2-lam3);` `f2 BY y4 y5-y6(lam5-lam6);` — labeled loadings
   - `f1(vf1);` `f2(vf2);` — labeled factor variances
   - `y1-y3(ve1-ve3);` `y4-y6(ve4-ve6);` — labeled residual variances
3. **MODEL CONSTRAINT:**
   - `NEW(rel2 rel5 stan3 stan6);` — declares 4 derived parameters not in the analysis model
   - `rel2 = lam2**2*vf1/(lam2**2*vf1 + ve2);` `rel5 = lam5**2*vf2/(lam5**2*vf2 + ve5);` — reliability of y2 and y5
   - `rel5 = rel2;` — forces the two reliabilities equal
   - `stan3 = lam3*SQRT(vf1)/SQRT(lam3**2*vf1 + ve3);` `stan6 = lam6*SQRT(vf2)/SQRT(lam6**2*vf2 + ve6);` — standardized loadings of y3 and y6
   - `0 = stan6 - stan3;` — equality via a difference-equals-zero statement
   - `ve2 > ve5;` `ve4 > 0;` — inequality constraints
4. **OUTPUT:** `STANDARDIZED;`

**[EFA/ESEM with residual variances constrained > 0 — 5.28]**
1. **VARIABLE:** `NAMES = y1-y10;`
2. **ANALYSIS:** `ROTATION = GEOMIN;`
3. **MODEL:** `f1-f2 BY y1-y10 (*1);` `y1-y10 (v1-v10);` — EFA factors plus labeled residual variances
4. **MODEL CONSTRAINT:** `DO(1,10) v#>0;` — expands to `v1>0; v2>0; ... v10>0;`
5. **OUTPUT:** `STDY;`

## Sub-option details
- `MODEL CONSTRAINT` can reference three kinds of things: labels defined on parameters inside `MODEL` (e.g. `lam2`, `ve1`), brand-new parameters declared via `NEW(...)` that exist only inside `MODEL CONSTRAINT` (e.g. `rel2`), and — in other examples, not this one — observed variables flagged with the `CONSTRAINT` option of `VARIABLE` for cases where an external covariate feeds into a constraint expression
- `NEW(rel2 rel5 stan3 stan6);` : declares parameters not otherwise part of the model; Mplus reports their point estimate and a delta-method standard error alongside ordinary model parameters — this is the standard way to get an SE for reliability, a hand-derived standardized coefficient, or any other function of the raw parameters
- `rel5 = rel2;` and `0 = stan6 - stan3;` are two equivalent syntaxes for asserting that one quantity equals another; either is accepted, and both differ from giving two *original* MODEL parameters the same label in that they can combine already-derived (`NEW`) quantities
- `ve2 > ve5;` and `ve4 > 0;` : inequality constraints — the `> 0` form is the standard device for keeping an estimated residual variance from crossing zero (a Heywood case) during estimation
- `DO(1,10) v#>0;` : the `DO` loop iterates the `#` placeholder across the given integer range, expanding one line into ten (`v1>0` through `v10>0`) — the labels being constrained (`v1`-`v10`) must already have been assigned in `MODEL` to the parameters of interest, here via `y1-y10 (v1-v10);`
- `OUTPUT: STANDARDIZED;` (5.20) is used as a cross-check: the R-square values and standardized loadings Mplus reports natively should exactly match the hand-derived `rel2`/`stan3`-style formulas, confirming the constraint formulas were written correctly
- `OUTPUT: STDY;` (5.28) requests standardization with respect to y only (not x), which is the metric conventionally reported for EFA results

## Post-run operations
- Read the point estimate, SE, and p-value for each `NEW()` parameter directly from the output, exactly like any other model parameter
- Confirm the equality/inequality constraints actually held at the converged solution — if an inequality can't be satisfied, that's a signal the underlying model (not just the constraint) has a real identification or Heywood-case problem the constraint is only masking
- Cross-check `STANDARDIZED`/`STDY` native output against the hand-computed constraint values to catch formula-writing errors
- In a large `DO`-loop model, check the parameter listing (or `TECH1`) to confirm the loop expanded to the expected number of individual constraints

## Likely FAQ mapping
- "How do I get Mplus to report reliability with a standard error?" → `MODEL CONSTRAINT: NEW(...)` with the reliability formula, Ex 5.20 pattern
- "How do I stop a residual variance from going negative during estimation?" → `MODEL CONSTRAINT: <label> > 0;`
- "I have 20 parameters that all need the identical constraint — do I have to write it 20 times?" → no, use `DO(start,end) <expression with #>;`
- "Can I hand-compute standardized coefficients and check them against Mplus's own STDYX/STDY output?" → yes, exactly what `stan3`/`stan6` (Ex 5.20) and `STDY` (Ex 5.28) demonstrate

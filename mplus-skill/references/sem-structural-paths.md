# Structural Equation Modeling (SEM) — Structural Paths Among Latent Factors

> Source: Mplus User's Guide v8, Chapter 5, Examples 5.11, 5.12
> https://www.statmodel.com/HTML_UG/chapter5V8.htm

## One-line summary
A full SEM: a CFA measurement model (factors measured by observed indicators) combined with a structural model (regression/mediation paths *among the latent factors themselves*) — this is the procedure to use once a model needs both a measurement part and a structural part with latent variables, as opposed to `path-analysis-basic.md` (no latent variables at all) or plain `cfa-continuous.md` (no structural paths among factors).

## Prerequisite checklist
- [ ] Confirm the measurement model first: how many factors, which observed items measure each (same as CFA)
- [ ] Confirm the structural model: which factors predict which other factors, and in what direction (draw this the same way as a path diagram, but with circles/ovals for the factors instead of rectangles)
- [ ] If mediation is of interest, confirm exactly which indirect path(s) through the factors are wanted
- [ ] Confirm none of the "predicted" factors are also indicators of another factor in a way that creates a logical inconsistency (structural paths follow the same DAG-like logic as path analysis, just among latent variables)

## Option selection logic
| Situation | Choice |
|---|---|
| Want to test a directional (regression) relationship between two or more factors | add `ON` statements among factor names, same syntax as observed-variable path analysis |
| Want the size/significance of an indirect effect that runs through a mediating factor | add `MODEL INDIRECT:` referencing factor names, exactly as in `path-analysis-mediation-bootstrap-missing.md` but with factors instead of observed variables |
| Want factor variances/covariances left as in ordinary CFA (no structural claim) | just use `cfa-continuous.md` instead — don't add `ON` statements |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE: NAMES ARE ...;**
4. **MODEL:**
   - Measurement part: `f1 BY y1-y3; f2 BY y4-y6; f3 BY y7-y9; f4 BY y10-y12;`
   - Structural part: `f4 ON f3; f3 ON f1 f2;` — same `ON` syntax as observed-variable regression, just with factor names on both sides
5. **MODEL INDIRECT:** (only if mediation through factors is wanted) `f4 IND f3 f1;` — same syntax as `path-analysis-mediation-bootstrap-missing.md`, with factor names

## Sub-option details
- `f4 ON f3; f3 ON f1 f2;` : structural paths among latent factors use identical `ON` syntax to observed-variable regression — the only difference is that the "variables" here are factors defined earlier by `BY` statements
- `MODEL INDIRECT: f4 IND f3 f1;` : requests the indirect effect of f1 on f4 through f3 — identical logic to path analysis's `MODEL INDIRECT`, just operating on latent factors; standard errors are computed automatically
- Bootstrap confidence intervals for the indirect effect follow the same pattern as `path-analysis-mediation-bootstrap-missing.md` (`ANALYSIS: BOOTSTRAP = 1000;` + `OUTPUT: CINTERVAL (BOOTSTRAP);`) — add these only if the user asks for significance testing of the indirect effect
- Default estimator = ML (or the appropriate default given the indicator types, following the same rules as CFA)

## Post-run operations
- Check the measurement part's fit the same way as CFA (CFI/TLI/RMSEA/SRMR) before trusting the structural paths — a poor-fitting measurement model undermines the structural results
- Check the structural path coefficients (`f4 ON f3`, `f3 ON f1 f2`) the same way as any regression coefficient
- If `MODEL INDIRECT` was used, check the "TOTAL, TOTAL INDIRECT, SPECIFIC INDIRECT EFFECTS" section, same as in path analysis

## Likely FAQ mapping
- "I want to test whether one latent construct predicts another" → this file
- "I have a mediation model but the variables are all measured by multiple items (factors), not single observed variables" → this file, not `path-analysis-basic.md`
- "Can I get the indirect effect through a factor, with a bootstrap CI?" → yes, same `MODEL INDIRECT` + `BOOTSTRAP` + `CINTERVAL(BOOTSTRAP)` pattern as path analysis, added only on request

# PWITH Residual Covariances and DIFFTEST Chi-Square Difference Testing

> Source: Mplus User's Guide v8, Chapter 13, Example(s) 13.11, 13.12

## One-line summary
Two unrelated "special feature" MODEL/ANALYSIS options grouped together because they both refine standard model output: `PWITH` adds a compact way to estimate residual covariances only between adjacent pairs of variables (e.g. repeated measures), and `DIFFTEST` runs a proper two-step nested-model chi-square difference test when the estimator is WLSMV or MLMV (for which the plain chi-square subtraction is invalid).

## Prerequisite checklist
- [ ] For PWITH: do you have an ordered set of variables (e.g. repeated measures y1...yT) where you want residual covariance estimated only between each variable and the *next* one in the list, not every possible pair?
- [ ] For DIFFTEST: are you comparing two nested models (a less restrictive H1 and a more restrictive H0, e.g. free vs. zero-constrained factor covariances)?
- [ ] For DIFFTEST: is the estimator `WLSMV` or `MLMV`? (For `ML`, `MLR`, `WLS`, etc., a plain chi-square/df subtraction across the two runs is valid instead and `DIFFTEST` is not needed.)
- [ ] For DIFFTEST: can you run the H1 (less restrictive) model first and keep its output file available before running H0?

## Option selection logic
| Situation | Choice |
|---|---|
| Want residual covariances between consecutive/adjacent variables only (not all pairs) | `y1-yT PWITH y2-y_{T};` (offset lists) in `MODEL:` |
| Want every pairwise residual covariance instead (not just adjacent) | ordinary `y1 WITH y2; y1 WITH y3; ...;` statements instead of `PWITH` |
| Need a nested chi-square difference test and estimator is WLSMV or MLMV | two-step `DIFFTEST` procedure (save derivatives from H1, feed into H0) |
| Need a nested chi-square difference test and estimator is ML/MLR/WLS/etc. | just subtract the two models' chi-square values and df directly; `DIFFTEST` is unnecessary |

## Menu path & screen fields
**[PWITH — Ex 13.11]**
1. **VARIABLE: NAMES ARE ...; USEVARIABLES ARE ...;**
2. **MODEL:** the substantive model (e.g. a growth model with `i s | y1@0 y2@1 y3@2 y4@3;`)
   - `y1-y3 PWITH y2-y4;` — adds adjacent residual covariances

**[DIFFTEST — Ex 13.12, two separate runs]**
- *Step 1 (H1, less restrictive model):*
  1. **VARIABLE:** as usual, including `CATEGORICAL = ...;` if applicable
  2. **MODEL:** the less restrictive model (e.g. factor covariances freely estimated)
  3. **SAVEDATA: DIFFTEST IS deriv.dat;**
- *Step 2 (H0, more restrictive model, nested within H1):*
  1. **VARIABLE:** identical variable setup to Step 1
  2. **ANALYSIS: DIFFTEST IS deriv.dat;** (the file saved in Step 1)
  3. **MODEL:** the more restrictive model (adds the constraint being tested, e.g. factor covariances fixed to 0)
  4. Output includes the chi-square difference test result comparing H0 to H1

## Sub-option details
- `y11-y13 PWITH y12-y14;`: pairs each variable on the left with the corresponding variable on the right of the statement, in list order, and estimates a residual covariance for each pair. In the chapter's example, `y11-y13 PWITH y12-y14;` produces residual covariances for (y11,y12), (y12,y13), and (y13,y14) — i.e., only between each variable and its immediate neighbor in the sequence — rather than every possible pair among y11-y14.
- `SAVEDATA: DIFFTEST IS deriv.dat;`: used in the **H1 (less restrictive) run**; saves the derivatives of the H1 model to the named file so they can be reused when fitting the nested H0 model. This is the same SAVEDATA option documented in `output-savedata-plot-commands.md`; this file covers the full two-run workflow it belongs to.
- `ANALYSIS: DIFFTEST IS deriv.dat;`: used in the **H0 (more restrictive) run**; points to the file created by the H1 run's `SAVEDATA: DIFFTEST`. Mplus uses these saved derivatives together with the H0 model's own results to compute a chi-square difference test that is valid for WLSMV/MLMV, where naively subtracting the two models' reported chi-square values and df is **not** distributed as chi-square and would give the wrong answer.
- The two runs must use the exact same data, sample, and variable list (including `CATEGORICAL =`) — only the MODEL constraint differs between H1 and H0 — since the saved H1 derivatives are matched against the H0 run's own model specification.

## Post-run operations
- For PWITH: check `TECH1` to confirm exactly the intended adjacent-pair residual covariances were freed (and no others), especially at the ends of the list where there is only one neighbor.
- For DIFFTEST: confirm the H1 run completed and actually wrote the derivative file (check the log/output for a note that the file was saved) before launching the H0 run.
- For DIFFTEST: the H0 run's output reports the chi-square difference test statistic and its degrees of freedom directly — do not additionally subtract the two runs' individually reported chi-square values, since that is precisely the invalid approach this procedure replaces.
- If the H0 (nested/constrained) model is rejected by the difference test, consider whether the constraint (e.g., zero factor covariance) is substantively too strong before dropping it.

## Likely FAQ mapping
- "I have repeated-measures residuals and only want adjacent ones correlated, not all pairs" → `PWITH`
- "My model uses WLSMV (or MLMV) — can I just subtract the two chi-squares for a difference test like I would with ML?" → no; use the two-step `SAVEDATA: DIFFTEST` (H1) + `ANALYSIS: DIFFTEST` (H0) procedure instead
- "Do I need DIFFTEST if I'm using ML or MLR?" → no, a plain chi-square/df subtraction across the two nested runs is valid for those estimators
- "What order do the two DIFFTEST runs go in?" → less restrictive (H1) first, saving derivatives via `SAVEDATA: DIFFTEST`; more restrictive (H0, the nested model) second, reading them via `ANALYSIS: DIFFTEST`

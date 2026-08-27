# Growth Modeling — Parallel-Process, Multiple-Indicator, Two-Part, Autocorrelated-Residual, and Multiple-Cohort Designs

> Source: Mplus User's Guide v8, Chapter 6, Examples 6.13, 6.14, 6.15, 6.16, 6.17, 6.18

## One-line summary
Covers advanced extensions of the basic growth model (see `growth-linear-basic.md`): two simultaneously-growing outcomes with cross-process regressions (parallel process), a growth model of a factor measured by multiple indicators at each occasion (continuous or categorical, i.e. a second-order/multiple-indicator growth model), a two-part model for a semicontinuous outcome with a floor of exact zeros, correlated (autocorrelated) residuals over time via nonlinear constraints, and a multiple-group approach to accelerated (cohort-sequential) longitudinal designs.

## Prerequisite checklist
- [ ] A baseline linear growth model (intercept `i`, slope `s`) is understood — see `growth-linear-basic.md`
- [ ] Parallel process: confirm there are two separate repeated outcomes, each with its own growth trajectory, and a hypothesis about how growth in one predicts growth in the other
- [ ] Multiple-indicator growth: confirm the construct at each occasion is itself measured by several indicators (not a single observed score), so within-occasion measurement error must be separated from true growth
- [ ] Multiple-indicator growth, categorical case: confirm the indicators are binary/ordinal rather than continuous
- [ ] Two-part growth: confirm the outcome has a floor of exact zeros (a "preponderance of zeroes") plus a continuous distribution above zero (semicontinuous outcome)
- [ ] Autocorrelated residuals: confirm there's reason to believe residuals close in time are more correlated than residuals far apart, beyond what the growth factors already explain
- [ ] Multiple-cohort/accelerated design: confirm different birth-year cohorts were measured at different calendar times such that age, not measurement occasion, is the natural time axis, and that a wider age range than any single cohort covers is of interest

## Option selection logic
| Situation | Choice |
|---|---|
| Two repeated outcomes, want to test whether growth in one predicts (or is predicted by) growth in the other | Parallel-process growth: two `\|` statements, one per process, plus `ON` regressions between the growth factors, e.g. `s1 ON i2; s2 ON i1;` (Example 6.13) |
| Repeated construct is measured by several continuous indicators per occasion (not a single score) | Multiple-indicator growth: one `BY` statement per occasion defining a factor, equality constraints for measurement invariance, then a `\|` statement on the occasion factors, e.g. `i s | f1@0 f2@1 f3@2;` (Example 6.14) |
| Same as above, but indicators are binary/ordinal | Add `CATEGORICAL = ...;`, use threshold bracket statements instead of intercepts, and (if WLS) a curly-bracket scale-factor statement (Example 6.15) |
| Outcome has a floor of exact zeros plus a continuous part above zero (semicontinuous) | Two-part growth: `DATA TWOPART:` command splits the outcome into a binary "any value vs. zero" part and a continuous "value given nonzero" part, each with its own `\|` growth model (Example 6.16) |
| Residuals of the repeated outcome are expected to be correlated over time in a systematic, decaying pattern | Autocorrelated residuals via `MODEL CONSTRAINT`: label the residual variance and adjacent-lag covariances, then define them as nonlinear functions of one autocorrelation parameter (Example 6.17) |
| Data come from multiple birth-year cohorts measured at different calendar times, and age (not wave) is the time axis of interest | Multiple-group multiple-cohort growth: `GROUPING =` by cohort, fixed time scores per group reflecting each cohort's ages, with equality constraints linking parameters for ages shared by more than one cohort (Example 6.18) |

## Menu path & screen fields
1. **TITLE:**
2. **DATA:**
   - Standard: `FILE IS ...;`
   - Two-part: `DATA TWOPART: NAMES = y1-y4; BINARY = bin1-bin4; CONTINUOUS = cont1-cont4;` (plus optional `CUTPOINT` / `TRANSFORM`)
3. **VARIABLE:**
   - `NAMES ARE ...;`
   - Multiple-indicator categorical: `CATEGORICAL = u11 u21 u31 u12 u22 u32 u13 u23 u33;`
   - Two-part: `USEVARIABLES = bin1-bin4 cont1-cont4; CATEGORICAL = bin1-bin4; MISSING = ALL(999);`
   - Multiple-cohort: `GROUPING = g (1 = 1990 2 = 1989 3 = 1988);`
4. **ANALYSIS:** (as needed) `ESTIMATOR = MLR;`
5. **MODEL:**
   - Parallel process: `i1 s1 | y11@0 y12@1 y13@2 y14@3; i2 s2 | y21@0 y22@1 y23@2 y24@3; s1 ON i2; s2 ON i1;`
   - Multiple-indicator (continuous): `f1 BY y11 y21-y31 (1-2); f2 BY y12 y22-y32 (1-2); f3 BY y13 y23-y33 (1-2); [y11 y12 y13](3); [y21 y22 y23](4); [y31 y32 y33](5); i s | f1@0 f2@1 f3@2;`
   - Multiple-indicator (categorical): same `BY`/threshold-bracket logic, plus a curly-bracket scale-factor statement, e.g. `{u11-u31@1 u12-u33};`
   - Two-part: `iu su | bin1@0 bin2@1 bin3@2 bin4@3; iy sy | cont1@0 cont2@1 cont3@2 cont4@3; su@0; iu WITH sy@0;`
   - Autocorrelated residuals: `i s | y1@0 y2@1 y3@2 y4@3; y1-y4 (resvar); y1-y3 PWITH y2-y4 (p1); y1-y2 PWITH y3-y4 (p2); y1 WITH y4 (p3);` then `MODEL CONSTRAINT: NEW (corr); p1 = resvar*corr; p2 = resvar*corr**2; p3 = resvar*corr**3;`
   - Multiple-cohort: overall `MODEL:` with the reference cohort's time scores plus group-specific `MODEL <label>:` blocks with each cohort's own time scores
6. **OUTPUT:** `TECH1 TECH8;` (or `TECH1 MODINDICES(3.84);` for the multiple-cohort model, to check whether relaxed equality constraints are needed)

## Sub-option details
- **Parallel process (Example 6.13)**: `i1 s1 | ...` and `i2 s2 | ...` each define an independent linear growth model for one process, using the usual fixed time scores (0,1,2,3) and default parameterization. `s1 ON i2;` and `s2 ON i1;` add cross-process regressions of one process's rate of change on the other's initial level. By default, both intercept factors' means/variances/covariance are estimated (exogenous), while the slope factors' intercepts and residual variances are estimated and their residuals are correlated by default (since neither slope influences anything besides its own indicators once the `ON` paths are added).
- **Multiple-indicator growth, continuous (Example 6.14)**: one `BY` statement per occasion defines an occasion-specific factor (e.g., `f1 BY y11 y21-y31 (1-2);`); the metric of each factor is set automatically by fixing its first loading to 1. The parenthetical list-function labels (e.g., `(1-2)`) hold factor loadings equal across occasions — required for **measurement invariance**, a prerequisite for a valid multiple-indicator growth model. Separate bracket statements with their own labels (e.g., `[y11 y12 y13](3);`) hold each indicator's intercept equal across occasions. The growth `\|` statement (`i s | f1@0 f2@1 f3@2;`) then models growth in the occasion factors exactly like growth in observed variables, with the intercepts of `f1`–`f3` fixed at zero by default, the intercept growth factor's mean fixed at zero (its overall level absorbed by the indicator intercepts), and the slope growth factor's mean estimated.
- **Multiple-indicator growth, categorical (Example 6.15)**: identical structure to 6.14, but `CATEGORICAL = ...;` is declared and threshold bracket statements (e.g., `[u11$1 u12$1 u13$1](3);`) replace intercept bracket statements, since categorical indicators are described by thresholds. A curly-bracket statement listing observed variables (e.g., `{u11-u31@1 u12-u33};`) sets/frees the scale factors of the underlying latent response variables — used with weighted least squares and the Delta parameterization; the first occasion's factor's scale factors are fixed at 1, others freed, matching the usual Delta convention.
- **Two-part / semicontinuous growth (Example 6.16)**: `DATA TWOPART:` derives, from each original outcome with a floor of zeros, a binary variable (1 = nonzero, 0 = zero) and a continuous variable (equal to the log of the original value by default when nonzero, missing when the original value is zero or below the cutpoint). `NAMES` identifies the source variables, `BINARY`/`CONTINUOUS` name the two derived sets, `CUTPOINT` (default 0) sets the split point, and `TRANSFORM` can turn off the default log transform. Both derived sets must appear in `USEVARIABLES`, and the binary set must be declared `CATEGORICAL`. Two separate growth models are then estimated: `iu su | bin1@0 ...;` for the binary ("any nonzero value") part, `iy sy | cont1@0 ...;` for the continuous ("value given nonzero") part. Fixing `su@0;` (binary-part slope variance) and `iu WITH sy@0;` (a cross-part covariance) are common stabilizing restrictions, since not all growth-factor covariances are typically significant in two-part models — these particular restrictions aren't defaults, they reflect a deliberate simplification made in the example.
- **Autocorrelated residuals (Example 6.17)**: a label placed after a list of residual variances (`y1-y4 (resvar);`) both holds them equal to each other and gives that shared value a name usable in `MODEL CONSTRAINT`. `PWITH` pairs up variables positionally for residual covariances (e.g., `y1-y3 PWITH y2-y4 (p1);` covaries y1 with y2, y2 with y3, y3 with y4 — adjacent-lag pairs — all sharing label `p1`). `MODEL CONSTRAINT: NEW (corr);` introduces a new autocorrelation parameter not otherwise in the model; the equations `p1 = resvar*corr; p2 = resvar*corr**2; p3 = resvar*corr**3;` impose a first-order autoregressive structure — the covariance between residuals k lags apart equals the residual variance times the autocorrelation raised to the k-th power.
- **Multiple-group multiple-cohort growth / accelerated longitudinal design (Example 6.18)**: `GROUPING = g (1 = 1990 2 = 1989 3 = 1988);` (VARIABLE command) splits a single stacked data set into cohort-specific groups and assigns readable labels. The overall `MODEL:` command applies to the reference group (whichever label is listed first is not automatically the reference — Mplus treats the unlabeled `MODEL:` as the overall/first-listed group and each `MODEL <label>:` as a difference from it); its `\|` statement uses that cohort's own time scores expressed as age (e.g., divided by 10 to keep the scores small and avoid convergence problems). Each additional `MODEL <label>:` block re-specifies the `\|` statement with that cohort's own time scores for age, since different cohorts were observed at different ages at each wave. Parenthetical numeric labels shared between the overall model and specific age-matched paths in different group blocks (e.g., matching label `(12)` on a `y2 ON a22` path in one cohort's block and a same-age path in another) constrain those parameters to be equal across cohorts **only where the cohorts share the same age** — this tests/imposes the assumption that development depends on age, not on cohort or calendar time. Growth-factor means/variances/covariances are, by default, constrained equal across all groups in the overall `MODEL:` command (unless overridden), representing the baseline assumption that all cohorts come from the same population.

## Post-run operations
- Parallel process: examine `s1 ON i2` and `s2 ON i1` — significant coefficients indicate the initial level of one process predicts the rate of change in the other.
- Multiple-indicator growth: first check that measurement invariance constraints (equal loadings/intercepts or thresholds over time) fit adequately before interpreting the growth factors; only then interpret `i`/`s` on the latent-factor metric as in a standard growth model.
- Two-part growth: interpret the binary-part growth factors (`iu`, `su`) as growth in the probability of a nonzero value, and the continuous-part growth factors (`iy`, `sy`) as growth in the (typically log) magnitude conditional on being nonzero — report both, since they answer different questions.
- Autocorrelated residuals: check the sign/magnitude of `corr` from `MODEL CONSTRAINT` output — a value away from zero indicates residual dependency beyond what the growth factors capture; compare model fit to a model without the autocorrelation constraints.
- Multiple-cohort design: inspect whether the equality-constrained (shared-age) parameters hold up (e.g., via `MODINDICES(3.84)` to flag constraints that don't fit) — if cohorts diverge at the same age, the "cohort effect" assumption (that age alone explains development) is undermined.
- `OUTPUT: TECH1 TECH8;` is the standard pairing across these examples; `MODINDICES(3.84)` is additionally useful for the multiple-cohort model to test whether specific cross-cohort equality constraints should be relaxed.

## Likely FAQ mapping
- "I have two outcomes measured repeatedly and want to see how their trajectories relate" → parallel-process growth model (Example 6.13)
- "My construct at each wave is measured by several items/indicators, not one score" → multiple-indicator (second-order) growth model, continuous (Example 6.14) or categorical (Example 6.15) indicators
- "My outcome has a lot of exact zeros plus a continuous distribution above zero" → two-part/semicontinuous growth model via `DATA TWOPART:` (Example 6.16)
- "I think residuals close together in time are more correlated than residuals far apart" → autocorrelated residuals with `MODEL CONSTRAINT` (Example 6.17)
- "My study combines several birth cohorts measured at different times to cover a wider age range (accelerated/cohort-sequential design)" → multiple-group multiple-cohort growth model (Example 6.18)
- "My outcome is a plain continuous/categorical/count/censored score with no multi-process or multi-cohort complications" → see `growth-linear-basic.md` or `growth-censored-categorical-count-outcomes.md`
- "I want covariates or piecewise growth, not these structural extensions" → see `growth-covariates-piecewise.md`

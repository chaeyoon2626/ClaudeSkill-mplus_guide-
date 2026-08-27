# Multiple Group Analysis — Mechanics & Measurement Invariance Testing

> Source: Mplus User's Guide v8, Chapter 14 (Special Modeling Issues) — this chapter contains no numbered Examples; it is a technical-discussion chapter. Sections used: Multiple Group Analysis (Requesting a Multiple Group Analysis, First Group, Defaults, MODEL Command, Equalities, Means/Intercepts/Thresholds, Scale Factors, Residual Variances, Data in Multiple Group Analysis, Testing for Measurement Invariance)
> Bundled source: references/source-pdfs/Chapter14.pdf

## One-line summary
General-purpose mechanics for setting up any multi-group Mplus run (single file with a grouping variable, separate files per group, or summary data per group), how Mplus's default cross-group equality constraints work, how to write overall vs. group-specific `MODEL` blocks and equality-constraint labels, and how to define/compare nested configural → metric → scalar measurement-invariance models for each observed-variable type and estimator. Applies to multi-group CFA, path analysis/SEM, and growth models alike; multiple group analysis is not available for `TYPE=MIXTURE` (use `KNOWNCLASS` instead) or for EFA.

## Prerequisite checklist
- [ ] How is the grouping information provided? Single data file with a group-membership column? Separate data file per group? Summary (means + covariance) data, one block per group in a single file?
- [ ] What is the grouping variable's coding, and which value should count as the reference ("first") group for interpretation?
- [ ] Which parameters, if any, are expected to differ across groups beyond what Mplus already holds free/equal by default?
- [ ] If the goal is measurement-invariance testing: what type are the factor indicators (continuous/censored/count vs. binary vs. ordered categorical with 3+ categories), and what estimator/parameterization is in use (ML; or WLS-family with Delta or Theta parameterization)? The exact configural/metric/scalar model definitions depend on this combination.
- [ ] Is comparing latent factor means across groups a goal? (this needs at least scalar, or partial-scalar, invariance and a mean structure)
- [ ] Which estimator will be used for nested-model comparisons (plain ML/WLS vs. MLR/MLM/WLSM vs. WLSMV/MLMV) — this determines how the difference test must be computed

## Option selection logic
| Situation | Choice |
|---|---|
| Individual-level data for all groups sit in one data file | `VARIABLE: GROUPING IS grp (1 = male 2 = female);` |
| Individual-level data for each group are already in separate files | Multiple `FILE (label) = ...;` statements in `DATA:`, one per group, each carrying the same variables in the same format |
| Only summary data (means + covariance matrix) available, one block per group in one file | `DATA: NGROUPS = 2;` plus `NOBSERVATIONS = n1 n2;` (no `GROUPING` needed; Mplus auto-labels groups g1, g2, ... in file order) |
| Grouping is really a combination of ≥2 variables (e.g. gender × site) | Create a single combined grouping variable first with `DEFINE:`, then use that in `GROUPING IS` |
| Want to relax/override specific parameters for one group only | Add a `MODEL <label>:` block listing just the parameters that should differ for that group |
| Want a cross-group equality constraint on a parameter that Mplus does not already hold equal by default (e.g. a regression coefficient) | Put the same `(n)` label after the parameter(s) in the overall `MODEL:` command |
| Want to relax a default cross-group equality for one group (e.g. release a loading in one group only) | Re-mention that parameter (without/with a different label) inside that group's `MODEL <label>:` block |
| Categorical indicators, weighted least squares, need to compare latent-response-variable variance across groups | Reference the scale factor with curly braces, e.g. `MODEL g2: {u1-u5*.5};` (Delta parameterization) |
| Categorical indicators, weighted least squares, Theta parameterization, need to free residual variances across groups | Reference by variable name, e.g. `MODEL g2: u1-u5*2;` |
| Testing measurement invariance | Fit a sequence of increasingly restrictive nested models — configural, then metric, then scalar — and compare each pair |
| Comparing nested models estimated with plain ML or WLS | Ordinary chi-square difference test |
| Comparing nested models estimated with MLR, MLM, or WLSM | Use the scaling correction factor printed in the output — do not just subtract raw chi-squares |
| Comparing nested models estimated with WLSMV or MLMV | Use the `DIFFTEST` procedure (save derivatives from the less-restrictive model, feed them into the more-restrictive model's `ANALYSIS`) |
| Full invariance doesn't hold for one or two items | Consider partial measurement invariance — relax that item's constraints while keeping the rest equal |

## Menu path & screen fields
1. **TITLE:**
2. **DATA:**
   - `FILE IS ...;` — single-file case
   - `FILE (male) = male.dat; FILE (female) = female.dat;` — separate-files case (label in parentheses is used later in group-specific `MODEL` blocks)
   - `NGROUPS = 2;` + `NOBSERVATIONS = 180 220;` — summary-data case
3. **VARIABLE:**
   - `GROUPING IS grp (1 = male 2 = female);` — single-file case only; declares the grouping variable and assigns labels used in later `MODEL <label>:` blocks
4. **MODEL:** — the overall model, applied to every group except where a group-specific block overrides it
5. **MODEL <label>:** (repeat once per group needing overrides, e.g. `MODEL male:`, `MODEL g2:`) — list only the parameters that differ from the overall `MODEL:` for that group; anything not repeated stays governed by the overall model
6. **OUTPUT:** (optional) `SAMPSTAT;`, `STDYX;`, etc.
7. For measurement-invariance testing specifically: run this as a *sequence* of separate analyses of increasing restrictiveness (configural, then metric, then scalar), each a full input file, and compare fit between adjacent pairs (see Post-run operations)

## Sub-option details
- **First group**: for a single file, the group with the *lowest* value on the grouping variable; for separate files, the group named in the *first* `FILE` statement; for summary data, the *first* block in the data set (auto-labeled g1)
- **Defaults for multi-group analysis**: intercepts/thresholds and factor loadings of observed dependent variables that are factor indicators are held equal across groups by default; residual variances of those indicators are *not* held equal by default. All structural parameters (factor means, variances, covariances, regression coefficients) are free and unconstrained by default. Factor means are fixed at 0 in the first group and free in the other groups (the customary way to set a reference group).
- **Scale factors** (WLS, Delta parameterization, categorical indicators): fixed at 1 in the first group, free (starting value 1) in the other groups by default — this reflects that latent response variables are not restricted to equal variances across groups
- **Residual variances of latent response variables** (WLS, Theta parameterization, categorical indicators): fixed at 1 in the first group, free (starting value 1) in the other groups by default
- **Equality-constraint labels `(n)`**: the same number/label placed after a parameter or list of parameters constrains them to be equal; only one label per line. A label used in the *overall* `MODEL:` command applies across all groups; a label used inside a *group-specific* `MODEL <label>:` block applies only within that group. Example — overall command `MODEL: f1 BY y1-y5; y1-y5 (1);` holds all five residual variances equal to each other and across all groups (1 parameter estimated instead of 15 for a 3-group, 5-indicator model)
- Re-mentioning a parameter in a group-specific block *without* reusing the overall label frees it from that overall constraint for that group only (e.g. `MODEL g3: y1-y5;` frees those five residual variances for group g3 specifically)
- **Measurement-invariance model definitions** (least → most restrictive), by indicator type/estimator:
  - *Continuous, censored, and count indicators (any estimator)*: **configural** = loadings, intercepts, and residual variances all free across groups, factor means fixed at 0 in all groups; **metric** = loadings held equal across groups, intercepts and residual variances free, factor means still fixed at 0 in all groups; **scalar** = loadings and intercepts held equal across groups, residual variances free, factor means fixed at 0 in one group and free in the rest
  - *Binary indicators, WLS estimation*: only **configural** and **scalar** are usable — the metric model is not identified because scale factors/residual variances are allowed to vary across groups
  - *Binary indicators, ML estimation*: configural, metric, and scalar are all usable, because residual variances are implicitly fixed at 1 in all groups, which identifies the metric model
  - *Ordered categorical (3+ category) indicators*: same three-level configural/metric/scalar scheme as above for the relevant estimator/parameterization, except the **metric** model additionally holds equal, across groups, the first threshold of every item and the second threshold of the item used to set the factor's metric (see Millsap, 2011, cited in the manual for the rationale). The metric model is **not allowed at all** for ordinal indicators when an indicator cross-loads on more than one factor, when a factor's metric is set by fixing its variance to 1, or under Exploratory Structural Equation Modeling (ESEM).
- **Partial measurement invariance**: for continuous variables, relax the constraint on an individual intercept, loading, or residual variance as needed. For categorical variables, a threshold and its corresponding loading must be relaxed *together* for an item — and, when relaxed, that item's scale factor (Delta parameterization) or residual variance (Theta parameterization) must also be fixed at 1 for that group.

## Post-run operations
- Compare adjacent models in the invariance sequence (configural vs. metric, then metric vs. scalar) with a chi-square difference test: subtract the chi-square and degrees of freedom of the less restrictive model from the more restrictive one, then compare to a chi-square table using the df difference. A significant difference means constraining those parameters significantly worsens fit (non-invariance); a non-significant difference supports invariance at that level.
- For MLR, MLM, and WLSM: do not subtract raw chi-squares — use the scaling correction factor printed in the output.
- For WLSMV and MLMV: use the `DIFFTEST` procedure — save derivative information from the less restrictive model with `SAVEDATA: DIFFTEST IS deriv.dat;`, then read it into the more restrictive model's run with `ANALYSIS: DIFFTEST = deriv.dat;`.
- If a model has no chi-square available at all, difference testing can instead use −2 × the loglikelihood difference between nested models.
- Once scalar (or partial-scalar) invariance is supported, factor means become meaningfully comparable across groups in the output.
- For a CFA-specific walkthrough of which parameters to check first and how to decide when to stop testing further constraints, see `cfa-multigroup-invariance.md` — this file covers the underlying general syntax/data mechanics and the full per-variable-type invariance-model definitions that file doesn't spell out in as much detail.

## Likely FAQ mapping
- "How do I run one linked analysis across two or more groups?" → this file (Requesting a Multiple Group Analysis: GROUPING vs. multiple FILE vs. NGROUPS)
- "My group data are in separate files / I only have means and a covariance matrix per group" → this file (Data in Multiple Group Analysis)
- "Why are my factor loadings/intercepts automatically the same across my groups?" → this file (Defaults for Multiple Group Analysis)
- "How do I free just one loading/intercept for one group?" → this file (MODEL command in multiple group analysis, group-specific `MODEL <label>:` blocks)
- "How exactly is the metric/scalar model defined for a binary or ordinal item, and why can't I run a metric model here?" → this file (Measurement-invariance model definitions by indicator type/estimator)
- "How do I formally compare my invariance models statistically, especially with WLSMV?" → this file (Model Difference Testing, DIFFTEST)

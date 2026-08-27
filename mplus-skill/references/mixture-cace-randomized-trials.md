# CACE Estimation for Randomized Trials (Complier-Average Causal Effect)

> Source: Mplus User's Guide v8, Chapter 7, Examples 7.23, 7.24
> https://www.statmodel.com/HTML_UG/chapter7V8.htm

## One-line summary
Uses a two-class mixture model to estimate the causal effect of a randomized treatment on an outcome specifically among "compliers" (the Complier-Average Causal Effect, or CACE; Little & Yau, 1998), where the latent class variable represents true compliance status — observed for individuals assigned to treatment but structurally unobserved (mixed) for individuals in the control group, since control-group members never have the opportunity to reveal whether they would have complied.

## Prerequisite checklist
- [ ] Confirm the data come from a randomized trial with a treatment/control assignment variable (a covariate, not the class variable itself)
- [ ] Confirm compliance status is observed only in the treatment group; individuals in the control group have unknown (structurally missing) compliance status
- [ ] Decide how compliance information will be supplied to Mplus: as separate `TRAINING` variables carrying prior class-membership information (Example 7.23), or as an explicit observed indicator variable with a missing-value code for the control group (Example 7.24)
- [ ] Identify the continuous outcome `y`, the treatment assignment dummy (e.g. `x2`), and any covariate(s) to control for (e.g. `x1`)

## Option selection logic
| Situation | Choice |
|---|---|
| Compliance status is available as separate 0/1 "training" columns already coding known vs. unknown class membership | `TRAINING = c1 c2;` in `VARIABLE` (Example 7.23) — one training variable per class |
| Compliance status is available as a single observed binary variable, missing for the control group | `CATEGORICAL = u;` for that variable, `MISSING = u (999);` (or the applicable missing-value code), and fix its class-specific thresholds at extreme logit values to make it a (near-)perfect indicator of the latent class (Example 7.24) |
| Want the treatment effect (the CACE) itself | Regress the outcome on the treatment dummy only within the complier class, and fix that same slope to zero in the non-complier class, since non-compliers by definition do not receive the treatment's active ingredient |
| Want compliance propensity to depend on a baseline covariate | `c ON x1;` in `%OVERALL%` — multinomial logistic regression of complier/non-complier class on the covariate |
| Want the outcome's baseline level and variance to differ between compliers and non-compliers regardless of treatment | Free the intercept `[y]` and residual variance `y` (declared with no `ON`) separately in each class-specific block |

## Menu path & screen fields
1. **VARIABLE:**
   - `NAMES ARE y x1 x2 c1 c2;` (Example 7.23, training-data approach) or `NAMES ARE u y x1 x2;` (Example 7.24, latent-indicator approach)
   - `CLASSES = c (2);` — class 1 = non-compliers, class 2 = compliers (or as coded by the training/indicator data)
   - `TRAINING = c1 c2;` — Example 7.23 only
   - `CATEGORICAL = u;` and `MISSING = u (999);` — Example 7.24 only
2. **ANALYSIS:** `TYPE = MIXTURE;`
3. **MODEL:**
   - `%OVERALL%`: `y ON x1 x2;` (baseline regression of outcome on covariate and treatment dummy), `c ON x1;` (compliance propensity regressed on the covariate)
   - `%c#1%` (non-compliers): `[y*value];` and `y;` (free the intercept/residual variance from the overall equality default), `y ON x2@0;` (treatment effect fixed to zero — non-compliers show no effect of treatment)
   - `%c#2%` (compliers): `[y*value];` and `y;` (free intercept/residual variance), the `y ON x2` slope is left free (not fixed) — this freely estimated slope is the CACE
   - Example 7.24 additionally needs, per class: `[u$1@15];` in the non-complier class and `[u$1@-15];` in the complier class
4. **OUTPUT:** `TECH1 TECH8;`

## Sub-option details
- `TRAINING = c1 c2;` : identifies variables that carry prior information about latent class membership for each of the two classes. Individuals in the treatment group are coded 1 on the training variable matching their observed compliance status and 0 on the other, fixing their class membership. Individuals in the control group are coded 1 on both training variables, signaling that either class is possible and their membership should be estimated from the model rather than fixed by the data.
- `c ON x1;` : the multinomial logistic regression of class membership on the covariate x1, comparing class 1 to class 2 (or vice versa depending on parameterization); the intercept is estimated by default.
- `y ON x1 x2;` in `%OVERALL%` : the baseline linear regression of the outcome on the covariate and the treatment dummy, held equal across classes unless overridden in a class-specific block.
- `y ON x2@0;` in `%c#1%` : fixes the treatment-dummy slope at exactly zero for the non-complier class, encoding the assumption that individuals who never comply are unaffected by nominal treatment assignment (the exclusion restriction underlying CACE).
- Freely estimated `y ON x2` in `%c#2%` : this class-specific slope is the CACE itself — the causal effect of treatment among those who would actually comply if assigned to treatment.
- `[u$1@15];` / `[u$1@-15];` (Example 7.24) : fixes (with `@`, not `*` — a hard fix, not a starting value) the threshold of the observed compliance indicator u at an extreme logit value (15 or -15) in each class, forcing the probability that u equals a particular value to be (numerically) exactly 0 or 1 in that class. This makes u function as a deterministic, perfectly known indicator of class membership wherever it is observed, while still letting the class be estimated probabilistically for control-group individuals whose u is missing.
- `MISSING = u (999);` : declares the numeric code used in the data for control-group individuals' structurally missing compliance status; Mplus treats this as missing data and estimates their class membership from the rest of the model rather than from u.

## Post-run operations
- Read the freely estimated `y ON x2` coefficient inside the complier class (`%c#2%`) as the CACE — the causal effect of treatment restricted to those who would comply.
- Compare this to a naive intention-to-treat estimate (a simple `y ON x2` regression ignoring compliance classes) to see how much diluting by non-compliers affects the estimated effect; CACE is typically larger in magnitude than an ITT estimate when compliance is imperfect.
- Check the estimated class proportions (from `c ON x1` intercepts, or overall class counts) against the observed compliance rate in the treatment group as a face-validity check — the model-implied complier proportion should be consistent with what was actually observed among treated individuals.
- If `c ON x1` is used, interpret it as which baseline characteristics predict a person's propensity to comply, independent of treatment assignment.

## Likely FAQ mapping
- "I ran a randomized trial with imperfect compliance and want the effect of treatment specifically among people who actually took it" → CACE estimation, this file; interpret the class-2 (complier) `y ON x2` slope as the causal effect
- "Compliance is only observed in my treatment group — how do I handle the control group?" → either supply `TRAINING` variables that mark control-group members as belonging to either class, or use a single compliance indicator with a `MISSING` code for the control group and extreme fixed thresholds (`@15`/`@-15`) per class
- "How is CACE different from just doing an intention-to-treat regression?" → CACE isolates the effect among compliers by fixing the treatment slope to zero in the non-complier class, rather than averaging the effect over everyone regardless of compliance
- "Can I let compliance depend on a baseline covariate?" → yes, `c ON x1;` in `%OVERALL%`
- "What's the difference between fixing a threshold with `@` versus giving it a starting value with `*`?" → `@` hard-fixes the parameter at that value permanently (as used here to force u into a near-perfect class indicator); `*` only supplies a starting value that the optimizer is free to move away from (see `mixture-class-indicator-types-starting-values.md`)

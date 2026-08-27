# Continuous-Time Survival Analysis (Cox Regression) with a Treatment/Control Mixture Class

> Source: Mplus User's Guide v8, Chapter 7, Example 7.30
> https://www.statmodel.com/HTML_UG/chapter7V8.htm

## One-line summary
Combines continuous-time Cox regression survival analysis with `TYPE = MIXTURE`, using a latent class variable whose classes are pinned to observed treatment/control assignment (via an extreme fixed threshold trick) so the treatment effect can be expressed as a class-specific intercept in the Cox regression, holding the baseline hazard shape equal across the two groups.

## Prerequisite checklist
- [ ] Confirm the outcome is a continuous (or finely measured) time-to-event variable suitable for Cox regression, and that a companion time-censoring variable exists (see `survival-analysis-discrete-continuous-time.md` for the general continuous-time survival setup)
- [ ] Confirm there is an observed binary treatment-assignment variable (0 = control, 1 = treatment) that will be used to pin the mixture classes to the known groups, rather than letting the classes be freely estimated
- [ ] Decide whether the baseline hazard should be held equal across the treatment and control groups (as in Ex 7.30) or allowed to differ — this example holds it equal and expresses the treatment effect purely through the Cox regression intercept
- [ ] Identify any covariate(s) predicting survival time within the Cox regression

## Option selection logic
| Situation | Choice |
|---|---|
| Want a mixture framework where "class" is actually a known treatment/control assignment, not something to be estimated | Declare the assignment variable `CATEGORICAL`, then fix its class-specific threshold at extreme logit values (`@15` in one class, `@-15` in the other) so class membership is deterministic given the observed assignment |
| Want the treatment effect expressed as a shift in the survival/hazard intercept, with the baseline hazard shape shared across groups | Fix the Cox regression intercept (via the survival variable's `[t]` bracket, or equivalently fixing the `t ON x` regression's baseline) to 0 in the control class and leave it free in the treatment class — the freely estimated value is the treatment effect |
| Want a formal test of whether the two groups' survival curves differ | `OUTPUT: LOGRANK;` — requests a logrank test (Mantel, 1966) comparing treatment vs. control survival curves |
| Want diagnostic/descriptive survival plots (Kaplan-Meier, hazard curves, etc.) | `PLOT: TYPE = PLOT2;` |

## Menu path & screen fields
1. **VARIABLE:**
   - `NAMES ARE t u x tcent class;`
   - `USEVARIABLES = t-tcent;` (restricts the variables actually entering the model)
   - `SURVIVAL = t;` — declares t as the continuous time-to-event variable
   - `TIMECENSORED = tcent;` — declares the companion right-censoring variable
   - `CATEGORICAL = u;` — u is the binary treatment/control assignment variable used as the "class indicator"
   - `CLASSES = c (2);`
2. **ANALYSIS:** `TYPE = MIXTURE;`
3. **MODEL:**
   - `%OVERALL%`: `t ON x;` — the Cox regression of survival time on the covariate, shared functional form across classes
   - `%c#1%` (control): `[u$1@15]; [t@0];` — fixes u's threshold so class 1 is deterministically the control group, and fixes the survival intercept at 0
   - `%c#2%` (treatment): `[u$1@-15]; [t];` — fixes u's threshold so class 2 is deterministically the treatment group, and leaves the survival intercept free (this free intercept is the treatment effect)
4. **OUTPUT:** `TECH1 LOGRANK;`
5. **PLOT:** `TYPE = PLOT2;`

## Sub-option details
- `SURVIVAL = t;` / `TIMECENSORED = tcent;` : identical role to the non-mixture continuous-time survival setup — see `survival-analysis-discrete-continuous-time.md` for full detail on these two VARIABLE-command options.
- `CATEGORICAL = u;` with `[u$1@15];` in class 1 and `[u$1@-15];` in class 2 : the `@` fixes (not just starts) u's threshold at an extreme logit value in each class, forcing the model-implied probability that u equals 1 to be numerically 0 in class 1 and 1 in class 2. Because u is the observed treatment dummy itself, this makes class membership perfectly determined by observed treatment assignment rather than something the model needs to infer — the same "extreme fixed threshold" trick used to build a perfectly known class from an observed indicator in CACE modeling (`mixture-cace-randomized-trials.md`, Example 7.24).
- `t ON x;` in `%OVERALL%` : the Cox (semi-parametric, profile-likelihood) regression of the survival time on the covariate x, with the same functional form and slope shared across both classes — only the intercept is allowed to differ, per the class-specific `[t]` statements below.
- `[t@0];` in class 1 (control) : fixes the Cox regression intercept for the survival variable at 0 in the control group, serving as the reference baseline hazard level.
- `[t];` in class 2 (treatment), no fixed value : leaves the survival intercept free in the treatment group; because the baseline hazard shape (via the shared `t ON x;` slope) is held equal across the two classes, this freely estimated intercept represents the treatment's shift in the hazard, i.e., the treatment effect.
- `OUTPUT: LOGRANK;` : requests a logrank test (Mantel, 1966) of the equality of the treatment and control survival curves — a nonparametric complement to the model-based Cox treatment-effect estimate.
- `PLOT: TYPE = PLOT2;` : produces, among other survival-related plots, the Kaplan-Meier curve, the sample log cumulative hazard curve, the estimated baseline hazard curve, the estimated baseline survival curve, the estimated log cumulative baseline curve, and overlays of the Kaplan-Meier/log-cumulative-hazard curves against their model-estimated counterparts.

## Post-run operations
- Read the freely estimated survival intercept in the treatment class (`[t]` in `%c#2%`) as the treatment effect on the Cox regression scale; because the baseline hazard is held equal across classes, this intercept isolates the shift attributable to treatment.
- Check the `LOGRANK` test result as an independent, assumption-light corroboration of whether the treatment and control survival curves actually differ.
- Use the `PLOT2` outputs (Kaplan-Meier vs. estimated baseline survival curve overlay in particular) to visually assess how well the fitted Cox model's baseline survival curve tracks the empirical Kaplan-Meier curve.
- Because class membership here is fixed to a known, observed treatment assignment (not estimated), do not evaluate this model using the usual class-enumeration diagnostics (BIC across k, entropy, TECH11/TECH14) — those apply when the number/composition of classes is itself being decided, which is not the case here.

## Likely FAQ mapping
- "I have a randomized/observed treatment-vs-control design and a time-to-event outcome — how do I get a Cox-regression treatment effect in Mplus?" → this file; use `TYPE = MIXTURE` with the treatment indicator's threshold fixed at extreme values per class, and read the treatment class's free survival intercept as the effect
- "How do I force a mixture class to equal a variable I already observe, instead of letting Mplus estimate it?" → fix that variable's class-specific threshold at an extreme logit value (e.g. `@15` / `@-15`) — the same technique used for known-class CACE modeling (`mixture-cace-randomized-trials.md`)
- "How do I test whether two groups' survival curves differ, aside from the regression coefficient?" → `OUTPUT: LOGRANK;` for a nonparametric logrank test
- "What survival-specific plots can I request?" → `PLOT: TYPE = PLOT2;` for Kaplan-Meier, hazard, and baseline survival/hazard curves (including overlays with the model-estimated counterparts)
- "My survival design doesn't involve a genuine unobserved subgroup — is TYPE=MIXTURE still the right tool?" → yes, when the "class" is actually a known group, mixture syntax with fixed extreme thresholds is the standard way to build a class-specific (here, group-specific) submodel while keeping a shared baseline hazard structure

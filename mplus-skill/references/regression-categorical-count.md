# Regression — Categorical / Ordinal / Nominal / Count Dependent Variables

> Source: Mplus User's Guide v8, Chapter 3, Examples 3.4–3.8, 3.10
> https://www.statmodel.com/HTML_UG/chapter3V8.htm

## One-line summary
The family of regressions used when the dependent variable is binary/ordinal categorical (0/1, Likert, etc.), nominal (unordered categories), or count (frequency) data.

## Prerequisite checklist
- [ ] Pin down the **exact type of the DV** as one of the following
  - Binary/ordinal categorical (e.g. 0/1, a 1-5 Likert item treated as ordinal) → probit or logistic
  - Unordered categorical / nominal (e.g. chosen major A/B/C) → multinomial (nominal)
  - Frequency/count data (0,1,2,3...) → Poisson family
  - Count data with an unusually large number of zeros → zero-inflated Poisson / negative binomial
- [ ] **Link function preference**: probit (default) vs. logistic (specify ML if odds-ratio interpretation is wanted) — which coefficient interpretation is desired
- [ ] **Overdispersion**: if the count data's variance is much larger than its mean, consider negative binomial
- [ ] (If nominal) whether a specific category needs to be set as the reference category
- [ ] (Whether this is a special case needing a theoretical constraint on specific multinomial-logit parameters — e.g. a nonlinear equality constraint between parameters)

## Option selection logic

| Situation | Choice | Mplus option |
|---|---|---|
| Binary/ordinal DV, want probit interpretation (default) | Probit regression | `CATEGORICAL = u1;` |
| Binary/ordinal DV, want odds-ratio (logistic) interpretation | Logistic regression | `CATEGORICAL = u1;` + `ANALYSIS: ESTIMATOR = ML;` |
| Unordered categorical (3+ options) | Multinomial logistic | `NOMINAL = u1;` |
| Frequency/count, no zero-inflation | Poisson regression | `COUNT = u1;` |
| Frequency data with excessive zeros (a structural zero group exists) | Zero-inflated Poisson | `COUNT = u1 (i);` + `u1#1 ON ...;` |
| Frequency data, variance ≫ mean (overdispersion) | Negative binomial | `COUNT = u1 (nb);` |
| Theoretical equality/nonlinear constraint needed among multinomial-logit parameters | Constrained multinomial model | `NOMINAL=` + `MODEL CONSTRAINT:` |

## Menu path & screen fields (input file blocks)
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE:**
   - `NAMES ARE ...;`, `USEVARIABLES ARE ...;`
   - Use exactly one of `CATEGORICAL = u1;` (binary/ordinal), `NOMINAL = u1;` (nominal), or `COUNT = u1;` (count), matching the DV type
4. **ANALYSIS:**
   - `ESTIMATOR = ML;` — specify only when logistic interpretation is wanted (the default for a categorical DV is robust WLS/probit)
5. **MODEL:**
   - `u1 ON x1 x3;` — basic form
   - Zero-inflated: `u1 ON x1 x3;` (count part) + `u1#1 ON x1 x3;` (inflation part)
   - For a specific nominal category only: `u1#1 u1#2 ON x1 x3;`
   - If a constraint is needed, label the parameters: `[u#1] (p1);`, then define the relationship in a `MODEL CONSTRAINT:` block
6. **MODEL CONSTRAINT:** (only when a special constraint is needed) — nonlinear relationship between labeled parameters

## Sub-option details
- `CATEGORICAL = u1;` : number of categories is auto-detected from the data; default estimator is robust WLS
- `ESTIMATOR = ML;` : when applied to a CATEGORICAL DV, switches from probit to logistic
- `NOMINAL = u1;` : the last category is the default reference category; use `u1#k` notation to access individual category parameters
- `COUNT = u1 (i);` : `(i)` = with zero-inflation, `(nb)` = negative binomial, neither = standard Poisson
- `u1#1 ON x1 x3;` : for zero-inflation, this is the logistic part predicting "probability of belonging to the structural-zero group"; for nominal, it's the logit for a specific category vs. the reference category
- `MODEL CONSTRAINT:` : e.g. `p2 = log((exp(p1)-1)/2 - 1);` — sharing a label imposes an equality constraint; a formula imposes a nonlinear constraint

## Post-run operations
- Probit/logistic: coefficients are in log-odds units (logistic) or z-score units (probit) — for `OR` (odds ratio), compute exp() or see `OUTPUT: STDYX;`
- **Getting Mplus to compute/test odds ratios directly**: logit coefficients themselves are in log-odds units, so getting an odds-ratio interpretation requires exponentiation (exp). You can compute exp() by hand (if you also add `OUTPUT: CINTERVAL;`, applying exp() to the lower/upper bounds of that confidence interval gives the OR's confidence interval too), but if you want Mplus to also compute the standard error and p-value directly, define a new parameter with `NEW()` inside `MODEL CONSTRAINT`:
  ```
  MODEL:     u1 ON x1 (p1)
                 x2 (p2);

  MODEL CONSTRAINT:
             NEW(or_x1 or_x2);
             or_x1 = exp(p1);
             or_x2 = exp(p2);
  ```
  This makes the odds ratio, along with its SE, z, and p-value, appear directly in the "New/Additional Parameters" section of the output. The `MODEL CONSTRAINT`/`NEW()`/parameter-labeling syntax is the same standard pattern already confirmed in Ch.3 Example 3.10 (nonlinear constraints) and Ch.3 Example 3.18 (moderated mediation), applied here. **However, don't add this block to the base code until the user actually asks for the odds-ratio calculation** — mention that the option exists, and add it to the code only once requested.
- Nominal: confirm that each `u1#k ON` block in the results is the logit for "category k relative to the reference category"
- Zero-inflated: interpret the two parts (count model vs. inflation model) separately
- If overdispersion is suspected, recommend comparing the log-likelihood/AIC/BIC of the Poisson vs. negative binomial results

## Likely FAQ mapping
- "My dependent variable is a 5-point Likert item — how do I run a regression?" → ordinal categorical → `CATEGORICAL=`
- "I want to interpret this as an odds ratio" → recommend `ESTIMATOR=ML` (coefficients are log-odds; for the odds ratio itself, offer the `MODEL CONSTRAINT`+`NEW()`+`exp()` method above only if requested separately)
- "Can Mplus directly compute the odds ratio and its confidence interval for me?" → guide with the `MODEL CONSTRAINT`+`NEW()`+`exp()` method (don't include it in the base code — add it only when requested)
- "It's an unordered category like a major" → `NOMINAL=`
- "I want a count of survey responses (0,1,2,3...) as the DV" → `COUNT=`
- "There are too many zeros and the regression looks off" → check whether zero-inflation applies

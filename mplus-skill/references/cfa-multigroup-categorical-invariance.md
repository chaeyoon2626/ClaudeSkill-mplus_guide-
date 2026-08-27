# Multiple-Group CFA & Measurement Invariance — Categorical Indicators

> Source: Mplus User's Guide v8, Chapter 5, Examples 5.16, 5.17
> Bundled source: references/source-pdfs/Chapter5.pdf

## One-line summary
Extends multiple-group CFA/MIMIC measurement invariance testing (see `cfa-multigroup-invariance.md`) to binary/ordinal (categorical) indicators, where thresholds replace intercepts/means and an extra identification device — a scale factor (Delta parameterization) or a residual variance (Theta parameterization) — must be managed across groups.

## Prerequisite checklist
- [ ] Confirm the base single-group categorical CFA/MIMIC model is settled first — see `cfa-categorical-mixed.md` prerequisites
- [ ] Confirm the grouping variable and its value labels
- [ ] Confirm which parameterization is wanted:
  - **Delta** (default): scale factors are model parameters, residual variances of the underlying latent response variables are not
  - **Theta**: residual variances of the underlying latent response variables are model parameters, scale factors are not
- [ ] Confirm which items (if any) are suspected of non-invariant loadings/thresholds (partial invariance), since freeing a loading+threshold for an item requires also fixing that item's scale factor (Delta) or residual variance (Theta) in the group where it's freed, for identification

## Option selection logic
| Situation | Choice |
|---|---|
| Default categorical multi-group CFA/MIMIC with thresholds | just declare `CATEGORICAL ARE ...;` + `GROUPING IS ...;` — Delta parameterization is the default |
| Want residual variances of the underlying continuous latent response variables to be explicit free parameters instead of scale factors | `ANALYSIS: PARAMETERIZATION = THETA;` |
| A specific item's loading/threshold isn't invariant across groups (partial invariance) | free the loading + threshold for that item in a group-specific `MODEL <groupname>:` block, **and** fix that item's scale factor (Delta: `{item@1};`) or residual variance (Theta: `item@1;`) in that same group |

## Menu path & screen fields
**[Delta parameterization (default) — 5.16]**
1. **VARIABLE:**
   - `NAMES ARE u1-u6 x1-x3 g;`
   - `CATEGORICAL ARE u1-u6;`
   - `GROUPING IS g (1 = male 2 = female);`
2. **MODEL:** `f1 BY u1-u3; f2 BY u4-u6; f1 f2 ON x1-x3;`
3. **MODEL female:** (partial invariance for u3)
   - `f1 BY u3;` — frees u3's loading for females
   - `[u3$1];` — frees u3's threshold for females
   - `{u3@1};` — fixes u3's scale factor to 1 for females (required for identification once loading+threshold are both freed)

**[Theta parameterization — 5.17]**
1. **ANALYSIS:** `PARAMETERIZATION = THETA;`
2. **VARIABLE / MODEL:** identical to the Delta version above
3. **MODEL female:**
   - `f1 BY u3;`
   - `[u3$1];`
   - `u3@1;` — fixes u3's *residual variance* (not a scale factor) to 1 for females

## Sub-option details
- `CATEGORICAL ARE u1-u6;` : thresholds (not intercepts/means) are modeled for binary/ordinal indicators; the number of thresholds per variable = number of categories minus 1 (a binary variable has one threshold, referenced as `u3$1`)
- When a mean structure is present (the norm in multi-group analysis), Mplus holds both factor loadings and thresholds equal across groups by default (metric + scalar invariance), exactly mirroring the continuous-indicator case in `cfa-multigroup-invariance.md`
- **Delta parameterization** (default): scale factors, referenced with curly braces `{}`, are fixed at 1 in the first group and freely estimated in other groups by default. If a specific item's loading and threshold are both freed across groups, that item's scale factor must be fixed at 1 in the groups where it's freed, for identification
- **Theta parameterization** (`PARAMETERIZATION = THETA;`): the residual variances of the latent response variables underlying the categorical indicators are the free parameters instead of scale factors — fixed at 1 in the first group, free in others by default, with the same "must fix to 1 when loading+threshold are freed" rule applying to the residual variance instead
- Default estimator = robust weighted least squares; with maximum likelihood, logistic regressions are estimated via numerical integration (increasingly demanding as the number of factors/sample size grow)

## Post-run operations
- Compare fit across configural → metric → scalar categorical models, same logic as the continuous case, using the appropriate WLSMV-family chi-square difference test or ML-based comparison
- Confirm that any partial-invariance freeing (loading + threshold for one item) was paired correctly with fixing that item's scale factor (Delta) or residual variance (Theta) in the same group — a common source of non-identification errors in categorical multi-group models
- Choose Delta vs. Theta based on which quantity you want directly interpretable in standardized output (Theta gives you residual variances directly; Delta gives you scale factors) — both represent the same underlying model, just reparameterized

## Likely FAQ mapping
- "My items are Likert-scale and I want to test measurement invariance across groups" → this file, not `cfa-multigroup-invariance.md` (continuous version)
- "What's a scale factor / why do I need curly braces `{}`?" → Delta-parameterization identification device for categorical multi-group models; explain the default-fix-first-group-free-elsewhere pattern
- "I freed one item's loading and threshold for partial invariance and now the model won't identify" → remind to fix that item's scale factor (`{item@1};`) or residual variance (`item@1;`, under Theta) in the same group
- "Should I use Delta or Theta parameterization?" → both are equivalent reparameterizations; Delta is the Mplus default, Theta is selected via `PARAMETERIZATION = THETA;` when residual variances (rather than scale factors) are the more natural quantity to report

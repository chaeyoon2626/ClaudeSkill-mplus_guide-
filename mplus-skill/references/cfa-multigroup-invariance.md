# Multiple-Group CFA & Measurement Invariance

> Source: Mplus User's Guide v8, Chapter 5, Examples 5.14, 5.15
> https://www.statmodel.com/HTML_UG/chapter5V8.htm

## One-line summary
Tests whether a CFA measurement model works the same way across two or more known groups (e.g. male/female, country, time point) — i.e. whether the factor structure, loadings, and intercepts are equivalent (invariant) across groups, which is a prerequisite for meaningfully comparing factor means/relationships across those groups.

## Prerequisite checklist
- [ ] Confirm the base (single-group) CFA measurement model is already settled — see `cfa-continuous.md` prerequisites first
- [ ] Confirm the grouping variable: which variable in the data identifies group membership, and what do its values mean (e.g. 1=male, 2=female)
- [ ] Confirm which level of invariance is the actual goal:
  - **Configural**: same factor structure (which items load on which factor) across groups, nothing else constrained
  - **Metric**: configural + loadings equal across groups
  - **Scalar**: metric + intercepts equal across groups (requires a mean structure)
  - **Partial invariance**: same as metric/scalar, but with a few specific items freed across groups because full invariance didn't hold
- [ ] Is comparing factor **means** across groups part of the goal? (this requires scalar invariance and a mean structure — i.e. can't use `NOMEANSTRUCTURE`)

## Option selection logic
| Situation | Choice |
|---|---|
| Just want to compare relationships (e.g. factor-covariate regressions) across groups, not factor means | `ANALYSIS: MODEL = NOMEANSTRUCTURE;` (no mean structure) |
| Want to compare factor means across groups (the more common goal) | mean structure included (default when `NOMEANSTRUCTURE` isn't specified), test scalar invariance |
| Testing whether loadings/intercepts are equal across groups | fit the model with default (equal) constraints first — Mplus holds loadings and intercepts equal across groups by default |
| A specific item doesn't hold to equal loadings/intercepts (partial invariance) | free just that item in a group-specific `MODEL <groupname>:` block |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE:**
   - `NAMES ARE y1-y6 x1-x3 g;`
   - `GROUPING IS g (1 = male 2 = female);` — declares the grouping variable and labels its values
4. **ANALYSIS:** (optional) `MODEL = NOMEANSTRUCTURE;` — only if factor means are not of interest
5. **MODEL:** the base measurement model, applied to all groups by default: `f1 BY y1-y3; f2 BY y4-y6;` (+ any structural/MIMIC paths)
6. **MODEL <groupname>:** (e.g. `MODEL female:`) — group-specific overrides, used to relax equality constraints for that group only:
   - `f1 BY y3;` — frees the y3 loading for this group (test/relax metric invariance for that item)
   - `[y3];` — frees the y3 intercept for this group (test/relax scalar invariance for that item; only meaningful with a mean structure)

## Sub-option details
- `GROUPING IS g (1 = male 2 = female);` : required to activate multiple-group analysis; the labels are just for readability in the output
- The top-level `MODEL:` block is the constrained (invariant) model applied to every group — Mplus's default behavior already holds factor loadings equal across groups (metric invariance) and, when a mean structure is present, intercepts equal too (scalar invariance), **without you needing to write anything extra**
- A `MODEL <groupname>:` block only needs to repeat the specific parameters you want to **free** for that group — anything not repeated stays at the equality-constrained default
- `[y3];` inside a group-specific block frees that indicator's intercept for that group only; without a mean structure (`NOMEANSTRUCTURE`), intercepts/means aren't modeled at all, so this only applies to the with-mean-structure setup (Example 5.15 pattern)

## Post-run operations
- Compare fit across nested models in sequence: configural → metric → scalar, using chi-square difference tests (or, for ML, comparing CFI/RMSEA changes) between each pair — a substantial fit drop when moving from configural to metric (or metric to scalar) suggests that level of invariance doesn't hold
- If full metric/scalar invariance is rejected, look at modification indices to identify which specific item(s) to free (partial invariance), then re-test
- Only once scalar (or partial scalar) invariance holds is it meaningful to compare factor means across groups in the output

## Likely FAQ mapping
- "I want to check if my scale works the same way for men and women" → this file (measurement invariance)
- "Can I compare the average factor score between groups?" → only after establishing scalar invariance; requires a mean structure (don't use `NOMEANSTRUCTURE`)
- "Metric invariance failed for one item — what now?" → free just that item's loading/intercept in a group-specific `MODEL <groupname>:` block (partial invariance), and note that the rest stays constrained

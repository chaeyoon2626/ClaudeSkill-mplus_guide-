# Bi-Factor Exploratory Factor Analysis

> Source: Mplus User's Guide v8, Chapter 4, Example 4.7
> https://www.statmodel.com/HTML_UG/chapter4V8.htm

## One-line summary
A factor analysis that simultaneously estimates a single general factor shared by all items, and specific factors shared only within subgroups of items.

## Prerequisite checklist
- [ ] Confirm the theoretical expectation is "a common construct shared by all items, plus item-subgroup-specific constructs" together (if a simple multi-factor structure is expected instead, `efa-continuous.md` may be more appropriate)
- [ ] Confirm the indicators are continuous
- [ ] The total factor-count range to explore (expressed as 1 general factor + n specific factors = total factor count, e.g. 2-3 = 1-2 specific factors)
- [ ] Whether to allow correlation among specific factors and with the general factor (oblique, default) or assume complete independence (orthogonal)

## Option selection logic
| Situation | Choice |
|---|---|
| Confirming a general + specific factor structure, allowing correlation (default) | `ROTATION = BI-GEOMIN;` |
| Specific factors must be completely independent from each other/the general factor | `ROTATION = BI-GEOMIN(ORTHOGONAL);` |
| Want to use a different bi-factor rotation criterion | `ROTATION = BI-CF-QUARTIMAX;` |

## Menu path & screen fields
1. **VARIABLE:** `NAMES = y1-y10;`
2. **ANALYSIS:**
   - `TYPE = EFA 2 3;` — total factor count 2-3 (= 1 general factor + 1-2 specific factors)
   - `ROTATION = BI-GEOMIN;` — bi-factor-specific rotation

## Sub-option details
- `TYPE = EFA 2 3;` : in bi-factor models, "total factor count = general factor (always 1) + number of specific factors," so 2 = 1 specific factor, 3 = 2 specific factors
- `ROTATION = BI-GEOMIN;` : defaults to oblique (specific factors allowed to correlate with the general factor and each other)
- `ROTATION = BI-GEOMIN(ORTHOGONAL);` : the fully independent version
- `ROTATION = BI-CF-QUARTIMAX;` : an alternative bi-factor rotation criterion

## Post-run operations
- In the results table, the loadings on the "General factor" should normally be significant fairly evenly across all items for it to be a well-formed bi-factor structure
- Check whether each specific factor's loadings show up clearly only within a particular subgroup of items
- If the general factor explains too little variance, note to the user that a simpler multi-factor model (`efa-continuous.md`) may fit better

## Likely FAQ mapping
- "It seems like all items measure something in common, plus each subdomain also measures something separately" → guide toward bi-factor EFA
- "Do the general and specific factors need to be independent?" → explain the default allows correlation (oblique); use the ORTHOGONAL option if needed

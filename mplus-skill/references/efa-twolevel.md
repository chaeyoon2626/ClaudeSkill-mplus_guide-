# Two-Level Exploratory Factor Analysis

> Source: Mplus User's Guide v8, Chapter 4, Examples 4.5–4.6
> https://www.statmodel.com/HTML_UG/chapter4V8.htm

## One-line summary
For data where individuals are clustered within a higher-level group (school, organization, etc.), separately explore the individual-level (within) and group-level (between) factor structures.

## Prerequisite checklist
- [ ] Confirm there is a variable identifying the cluster in the data (e.g. school ID, class ID)
- [ ] Whether the indicators are measured only at the individual level, or whether some are measured at the group level (e.g. school-level characteristic variables)
- [ ] Range of factor counts to explore, separately for the individual and group level
- [ ] Whether the indicator type (continuous/categorical) is the same or different across levels

## Option selection logic
| Situation | Choice |
|---|---|
| All indicators measured at the individual level only, continuous | `TYPE = TWOLEVEL EFA a b UW c d UB;` (individual-level factor count a-b, group-level c-d) |
| Individual level is categorical, and separate continuous variables exist at the group level | specify `CATEGORICAL=` (individual level) + `BETWEEN=` (group-level variables) together |
| Computational efficiency matters (large clustered dataset) | save sample statistics with `SAVEDATA: SWMATRIX = ...;` for reuse |

## Menu path & screen fields
**[4.5 — individual-level continuous only]**
1. **VARIABLE:** `NAMES ARE y1-y6 x1 x2 w clus;` / `USEVARIABLES = y1-y6;` / `CLUSTER = clus;`
2. **ANALYSIS:** `TYPE = TWOLEVEL EFA 1 2 UW 1 1 UB;` — 1-2 factors at the individual level (UW), 1 factor at the group level (UB)

**[4.6 — individual-level categorical + group-level continuous mixed]**
1. **VARIABLE:**
   - `NAMES = u1-u6 y1-y4 x1 x2 w clus;`
   - `USEVARIABLES = u1-u6 y1-y4;`
   - `CATEGORICAL = u1-u6;` — individual-level categorical indicators
   - `CLUSTER = clus;`
   - `BETWEEN = y1-y4;` — indicators measured only at the group level
2. **ANALYSIS:** `TYPE = TWOLEVEL EFA 1 2 UW 1 2 UB;`
3. **SAVEDATA:** `SWMATRIX = ex4.6sw.dat;` — save within/between sample statistics (optional, speeds up reanalysis)

## Sub-option details
- `CLUSTER = clus;` : the cluster (higher-level group) identifier variable, required
- `TYPE = TWOLEVEL EFA a b UW c d UB;` : `UW` = individual (within) level factor-count range, `UB` = between-level factor-count range
- `BETWEEN = ...;` : specifies that a variable only has values at the group level (doesn't vary across individuals)
- `SWMATRIX` : saves an intermediate product to save computation time in large, repeated analyses

## Post-run operations
- Interpret the within-level and between-level factor solutions separately in the results (careful not to conflate them)
- If the ICC (intraclass correlation) is low, caution that between-level factor structure should be interpreted carefully
- The number of factors is decided independently per level (e.g. within=2 factors, between=1 factor is fine — they can differ)

## Likely FAQ mapping
- "My survey data has students clustered by school — how do I run factor analysis?" → guide toward two-level EFA
- "I want to include a school-level characteristic (one value for the whole group) along with student responses" → guide toward using `BETWEEN=`

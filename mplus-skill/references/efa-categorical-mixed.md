# Exploratory Factor Analysis (EFA) — Categorical / Mixed Indicators

> Source: Mplus User's Guide v8, Chapter 4, Examples 4.2–4.3
> https://www.statmodel.com/HTML_UG/chapter4V8.htm

## One-line summary
Exploratory factor analysis when the indicators (items) are binary/ordinal categorical, or when continuous, censored, categorical, and count types are mixed together.

## Prerequisite checklist
- [ ] Confirm the type of every indicator: continuous / categorical (binary-ordinal) / censored / count — if even one differs, mixed-type handling is needed
- [ ] Range of factor counts to explore
- [ ] Awareness of computation time: as the number of categorical indicators and factors grows, the numerical-integration burden grows exponentially (e.g. 4 factors × 7 points = 2401 points) → decide in advance whether to reduce integration points (an accuracy-vs-speed tradeoff)
- [ ] Estimator preference: is the default robust WLS sufficient for categorical-only data, or is there a specific reason ML (numerical integration) is needed (e.g. mixed indicators)?

## Option selection logic
| Situation | Choice |
|---|---|
| Indicators are all binary/ordinal categorical | `CATEGORICAL ARE u1-u12;` (default robust WLS is sufficient and fast) |
| Indicators mix continuous, censored, categorical, count | specify the matching option per variable group + ML-based numerical integration required (slow) |
| Taking too long to compute | reduce integration points from the default 7 with `ANALYSIS: INTEGRATION = 3;` etc. (an approximate solution) |

## Menu path & screen fields
**[Categorical only — 4.2]**
1. **VARIABLE:** `NAMES ARE u1-u12;` / `CATEGORICAL ARE u1-u12;`
2. **ANALYSIS:** `TYPE = EFA 1 4;`

**[Mixed types — 4.3]**
1. **VARIABLE:**
   - `NAMES = u4-u6 y4-y6 u1-u3 y1-y3;`
   - `CENSORED = y4-y6(b);` — censored (from below)
   - `CATEGORICAL = u1-u3;` — binary/ordinal
   - `COUNT = u4-u6;` — count
   - (any remaining unlabeled variables are automatically continuous)
2. **ANALYSIS:** `TYPE = EFA 1 4;` (mixed types internally use ML + numerical integration)

## Sub-option details
- `CATEGORICAL ARE u1-u12;` : number of categories auto-detected from the data. Default estimator = robust WLS
- For mixed types, default estimator = ML with robust SE, numerical integration. The integration dimensionality grows with the number of factors
- Reducing integration points: lowering from the default 7 to as few as 3 can substantially cut computation time (at some cost to accuracy)

## Post-run operations
- Note to the user that categorical indicators' loadings are on a probability scale (probit/logit), so care is needed when directly comparing magnitudes with continuous indicators
- Mixed-type models can take a long time to converge — recommend using `TECH8` to monitor iterative estimation progress
- If computation takes too long, suggest reducing integration points or narrowing the factor-count range and retrying

## Likely FAQ mapping
- "All my survey items are 5-point scales (ordinal) — how do I run EFA?" → specify `CATEGORICAL=`
- "My item types are all different (a mix of continuous + categorical + count)" → see the mixed-type table
- "The analysis is taking too long" → guide toward reducing integration points

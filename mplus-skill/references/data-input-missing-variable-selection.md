# Data Input, Missing Values, and Variable Selection/Transformation

> Source: Mplus User's Guide v8, Chapter 13, Example(s) 13.1-13.7
> Bundled source: references/source-pdfs/Chapter13.pdf

## One-line summary
Covers how to feed Mplus data that isn't a plain raw-score rectangular file — summary statistics (covariance / means+covariance matrices), fixed-column data — plus non-numeric/numeric missing-value flags, and how to select/transform variables and observations before modeling.

## Prerequisite checklist
- [ ] Do you have raw individual-level data, or only a summary covariance matrix / means-covariance matrix (e.g., copied from a published table)?
- [ ] Is the raw data file free-format (space/comma separated) or fixed-column (each variable occupies specific character positions)?
- [ ] Does the raw data use a missing-value code (numeric, e.g. 9/99/-999, or a non-numeric flag such as a blank or asterisk)?
- [ ] Do you need only a subset of the named variables and/or a subset of the rows (e.g., one subgroup) for this particular run?
- [ ] Do any variables need to be rescaled, recoded, or computed from other variables before modeling?

## Option selection logic
| Situation | Choice |
|---|---|
| You only have a covariance matrix (no means) from a paper/report | `DATA: TYPE = COVARIANCE; NOBSERVATIONS = ...;` |
| You have both means and a covariance matrix as summary data | `DATA: TYPE IS MEANS COVARIANCE; NOBSERVATIONS = ...;` |
| Raw data has fixed-width columns instead of free delimiters | `DATA: FORMAT IS ...;` |
| Missing is marked with a symbol (e.g. blank/asterisk), not a number | `VARIABLE: MISSING = *;` (applies to all variables) |
| Missing is marked with specific numeric code(s), possibly different per variable | `VARIABLE: MISSING = v1-v2(9) v3(9 99);` |
| Every variable in the file shares one numeric missing code | `VARIABLE: MISSING = ALL (9);` |
| Only some named variables/observations are needed for this specific model | `VARIABLE: USEVARIABLES ARE ...; USEOBSERVATIONS ARE (condition);` |
| A variable must be rescaled/recoded/computed before use | `DEFINE:` statements |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
   - `TYPE = COVARIANCE;` or `TYPE IS MEANS COVARIANCE;` — only when the input file itself already contains a summary matrix, not raw scores
   - `NOBSERVATIONS = ...;` — required whenever summary data (TYPE=COVARIANCE / MEANS COVARIANCE) is used; gives the sample size behind the matrix
   - `FORMAT IS ...;` — only needed if the raw data file is fixed-format instead of free format
3. **VARIABLE: NAMES ARE ...;**
   - `MISSING = ...;` — declare missing-value flag(s), numeric or the `*` symbol
   - `USEVARIABLES ARE ...;` — restrict the analysis to a subset of the named variables
   - `USEOBSERVATIONS ARE (condition);` — restrict the analysis to rows meeting a logical condition
4. **DEFINE:** (only if transformation/creation of variables is needed) — the transformed/created names can then be referenced in `USEVARIABLES` and `MODEL`
5. **MODEL:** as usual for the chosen procedure

## Sub-option details
- `TYPE = COVARIANCE;` / `TYPE IS MEANS COVARIANCE;`: summary data must be in an external free-format file. For means+covariance data, the means come first (one record), and the covariances must start on a new record.
- `NOBSERVATIONS = 1000;`: the N behind the summary matrix; required whenever TYPE=COVARIANCE or MEANS COVARIANCE is used.
- `FORMAT IS 3f4.2 3f2 f1 2f2;`: a Fortran-like fixed-format descriptor — e.g. `3f4.2` means the next 3 variables each take 4 columns with 2 decimal digits; `f1` means the next variable takes 1 column with 0 decimals; `2f2` means the next 2 variables each take 2 columns with 0 decimals.
- `MISSING = *;`: a single non-numeric flag (e.g. asterisk) applied to all variables in the data set.
- `MISSING = y1-y3(9) y4(9 99) y5-y12(9-12);`: numeric flags can differ by variable or variable group — y1 through y3 use 9; y4 uses 9 or 99; y5 through y12 use any of 9, 10, 11, 12.
- `MISSING = ALL (9);`: shortcut when every variable in the data set shares the same missing-value code.
- `USEVARIABLES ARE y1-y3 x1-x3;`: narrows the working variable set from everything named in `NAMES ARE` down to only what the current MODEL needs.
- `USEOBSERVATIONS ARE (x4 EQ 2);`: keeps only rows where the stated logical condition on a named variable holds (here, only rows where x4 = 2).
- `DEFINE: y1 = y1/100; x3 = SQRT(x3);`: transforms variables in place (rescaling, function transforms) before they are used in `USEVARIABLES`/`MODEL`; the DEFINE command can also be used to create new variables.

## Post-run operations
- When using summary data (TYPE=COVARIANCE/MEANS COVARIANCE), there is no raw-data-based output such as individual factor scores or case-level residuals — only what can be computed from the matrix itself.
- Double-check that `NOBSERVATIONS` matches the true N; an incorrect value silently distorts standard errors and fit statistics.
- After adding `MISSING =`, check the output's sample statistics / reported N to confirm the flagged values were actually excluded rather than analyzed as real scores.
- After `USEVARIABLES`/`USEOBSERVATIONS`, confirm the reported sample size in the output matches the intended subset.

## Likely FAQ mapping
- "I only have a correlation/covariance table from a published paper — can I still run this in Mplus?" → `TYPE = COVARIANCE;` + `NOBSERVATIONS`
- "My missing data are coded as -99 / 999 — how do I tell Mplus?" → `MISSING = varname(-99);` or `MISSING = ALL (999);`
- "I want to run this model only on one subgroup (e.g., males)" → `USEOBSERVATIONS ARE (group EQ 1);`
- "I need to rescale/transform a variable before the model" → `DEFINE:`
- "My data file has no delimiters; columns are fixed width" → `FORMAT IS ...;`

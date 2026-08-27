# Merging Data Sets and Using/Generating Replicate Weights

> Source: Mplus User's Guide v8, Chapter 13, Example(s) 13.17, 13.18, 13.19

## One-line summary
Covers two data-preparation features that operate through `SAVEDATA`/`VARIABLE`/`ANALYSIS` rather than through `MODEL`: merging two separate data files into one by a common ID variable, and using (or generating) replicate weights — an alternative to the sandwich/Taylor-series approach for standard errors under a complex sampling design (see `complex-survey-design-multilevel.md` for the `STRATIFICATION`/`CLUSTER`/`WEIGHT`/`TYPE=COMPLEX` sandwich-estimator approach, which this file does not repeat).

## Prerequisite checklist
- [ ] **For merging:** do you have two separate data files that share a common ID variable, with each file contributing different variables for the same set of (or overlapping) cases?
- [ ] **For merging:** do you know which variables from each file are actually needed in the merged result, and what missing-value flag(s) each source file uses?
- [ ] **For replicate weights (using):** does your data file already contain pre-computed replicate weight variables (e.g. from a survey organization) along with a main sampling weight, and do you know which resampling method (jackknife, BRR, bootstrap) was used to create them?
- [ ] **For replicate weights (generating):** do you instead have raw stratification/cluster/weight variables and want Mplus itself to generate replicate weights from them?
- [ ] Confirm `TYPE = COMPLEX` is in play for any replicate-weights analysis — replicate weights are only available under `TYPE = COMPLEX`.

## Option selection logic
| Situation | Choice |
|---|---|
| Need to combine two data files sharing a common ID into one analysis/saved data set | `SAVEDATA: MFILE = ...; MNAMES ARE ...; ` + `VARIABLE: IDVARIABLE IS ...;` |
| Only some variables from the second file are needed in the merge | add `MSELECT ARE ...;` |
| Second file's missing-value flag needs to be recognized during merge | add `MMISSING = ...;` |
| Second file is fixed-format, not free format | add `MFORMAT IS ...;` |
| Want to control the merged output file's own format / missing flag | `SAVEDATA: FILE IS ...; FORMAT IS ...; MISSFLAG = ...;` |
| Data file already has pre-built replicate weight variables (e.g. from survey provider) | `VARIABLE: WEIGHT = ...; REPWEIGHTS = ...;` + `ANALYSIS: TYPE = COMPLEX; REPSE = JACKKNIFE1;` (or the method actually used) |
| Need Mplus to generate replicate weights from stratification/cluster/weight variables | `VARIABLE: WEIGHT=...; STRATIFICATION=...; CLUSTER=...;` + `ANALYSIS: TYPE = COMPLEX; REPSE = BOOTSTRAP; BOOTSTRAP = ...;` + `SAVEDATA: SAVE = REPWEIGHTS; FILE IS ...;` |

## Menu path & screen fields
**[Merging two data sets — Ex 13.17]**
1. **DATA: FILE IS data1.dat;** — the first (primary) data file
2. **VARIABLE:**
   - `NAMES ARE id y1-y4;` — variable names in the first file (must include the ID)
   - `IDVARIABLE IS id;` — the merge key; must appear in both `NAMES` and `MNAMES`
   - `USEVARIABLES = y1 y2;` — restricts the first file's variables used (independent of the merge)
   - `MISSING IS *;` — missing-value flag for the first file
3. **ANALYSIS: TYPE = BASIC;** — merging works with any analysis type; `BASIC` is used here since no model is being fit, just the merge
4. **SAVEDATA:**
   - `MFILE = data2.dat;` — the second data file to merge in
   - `MNAMES ARE id y5-y8;` — variable names in the second file (must include the ID)
   - `MFORMAT IS F6 4F2;` — fixed format for the second file, if not free format
   - `MSELECT ARE y5 y8;` — subset of the second file's variables to bring into the merge
   - `MMISSING = y5-y8 (99);` — missing-value flag(s) for the second file
   - `FILE IS data12.sav;` — name of the merged output file
   - `FORMAT IS FREE;` — format of the merged output file
   - `MISSFLAG = 999;` — missing-value flag to use in the merged output file

**[Using existing replicate weights — Ex 13.18]**
1. **DATA: FILE IS rweights.dat;**
2. **VARIABLE:**
   - `NAMES ARE y1-y4 weight r1-r80;`
   - `WEIGHT = weight;` — main sampling weight (required whenever `REPWEIGHTS` is used)
   - `REPWEIGHTS = r1-r80;` — the pre-existing replicate weight variables
3. **ANALYSIS:**
   - `TYPE = COMPLEX;` — required for replicate weights
   - `REPSE = JACKKNIFE1;` — resampling method that was used to build r1-r80
4. **MODEL:** the substantive model as usual

**[Generating, using, and saving replicate weights — Ex 13.19]**
1. **DATA: FILE IS ex13.19.dat;**
2. **VARIABLE:**
   - `NAMES ARE y1-y4 weight strat psu;`
   - `WEIGHT = weight;`
   - `STRATIFICATION = strat;`
   - `CLUSTER = psu;`
3. **ANALYSIS:**
   - `TYPE = COMPLEX;`
   - `REPSE = BOOTSTRAP;` — resampling method used to generate the weights
   - `BOOTSTRAP = 100;` — number of bootstrap draws to generate
4. **MODEL:** substantive model
5. **SAVEDATA:**
   - `FILE IS rweights.sav;`
   - `SAVE = REPWEIGHTS;` — saves the newly generated replicate weights alongside the other analysis variables

## Sub-option details
- `IDVARIABLE IS id;`: designates the merge key; this variable name must be present in both the primary `VARIABLE: NAMES` list and the `SAVEDATA: MNAMES` list, and merging matches records between the two files by this ID.
- `MFILE`, `MNAMES`, `MFORMAT`, `MSELECT`, `MMISSING`: the `M`-prefixed `SAVEDATA` options all describe the **second** file being merged in — its path (`MFILE`), its full variable list (`MNAMES`), its format if fixed (`MFORMAT`), which of its variables to keep (`MSELECT`), and its missing-value flag(s) (`MMISSING`) — paralleling `DATA:`/`VARIABLE:` options but for the file being merged rather than the primary file.
- `SAVEDATA: FILE IS ...; FORMAT IS ...; MISSFLAG = ...;`: control the merged output file itself. The default output format is free, and the default missing-value flag in the merged file is an asterisk (`*`); both can be overridden. Different missing-value codes used in the two source files (e.g. `*` in the first, `99` in the second) are unified into a single flag (e.g. `999`) in the merged file.
- Merging works with any `ANALYSIS: TYPE`; `TYPE = BASIC` is used purely because the example's goal is producing the merged data set, not fitting a model.
- `WEIGHT =` is **required** whenever `REPWEIGHTS =` is used — replicate weights are additional variables that quantify sampling variability around the main weight, not a replacement for it.
- `REPSE =`: identifies the resampling method behind the replicate weights, e.g. `JACKKNIFE1` (jackknife) when weights already exist, or `BOOTSTRAP` when Mplus itself is asked to generate the replicate weights via `BOOTSTRAP = n;` (number of bootstrap draws).
- Replicate weights (used or generated) are only available under `ANALYSIS: TYPE = COMPLEX`. When *generating* them, `STRATIFICATION =` and/or `CLUSTER =` supply the design information (subpopulation strata, primary sampling units) that the resampling procedure draws on; when *using* pre-built replicate weight variables instead, `STRATIFICATION`/`CLUSTER` are not needed since the design information is already baked into the supplied replicate weight columns.
- `SAVEDATA: SAVE = REPWEIGHTS; FILE IS ...;`: after Mplus generates replicate weights (Ex 13.19), this saves them — together with the other analysis variables — to a new file, so the same replicate weights can be reused directly (via the "using existing replicate weights" pattern in Ex 13.18) in later analyses without regenerating them.

## Post-run operations
- After a merge, open/inspect the merged output file (or check the run's log) to confirm the row count and variable set match expectations, and that the unified missing-value flag was applied correctly to both sources' originally-missing values.
- After using or generating replicate weights, compare standard errors against a plain `TYPE = COMPLEX` sandwich-estimator run (`STRATIFICATION`/`CLUSTER`/`WEIGHT` only, no `REPWEIGHTS`) if a sensitivity check on the SE method is wanted.
- After generating replicate weights (Ex 13.19), verify the saved replicate-weight file actually contains the expected number of replicate columns (matching `BOOTSTRAP = n;` or the jackknife/BRR count implied by the design) before reusing it.

## Likely FAQ mapping
- "I have two data files that share a person ID and I need them combined for one analysis" → `SAVEDATA: MFILE=...; MNAMES=...;` + `VARIABLE: IDVARIABLE=...;`
- "The two files use different missing-value codes — will merging handle that?" → yes, via `MISSING`/`MMISSING` on each source plus `MISSFLAG` on the merged output
- "My survey data already comes with replicate weight columns (e.g. jackknife weights) — how do I use them?" → `VARIABLE: WEIGHT=...; REPWEIGHTS=...;` + `ANALYSIS: TYPE=COMPLEX; REPSE=JACKKNIFE1;` (match `REPSE` to the actual method used)
- "I don't have replicate weights yet, only stratification/cluster/weight variables — can Mplus create them for me?" → `ANALYSIS: TYPE=COMPLEX; REPSE=BOOTSTRAP; BOOTSTRAP=n;` + `SAVEDATA: SAVE=REPWEIGHTS;` to save them for reuse
- "Can I use STRATIFICATION and CLUSTER together with REPWEIGHTS?" → no — `STRATIFICATION`/`CLUSTER` are not used in conjunction with `REPWEIGHTS` (they're for generating replicate weights or for the plain sandwich-estimator `TYPE=COMPLEX` approach, not for reading in ready-made replicate weight columns)

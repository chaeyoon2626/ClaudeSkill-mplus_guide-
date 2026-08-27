# DATA Reshape/Derivation Commands (WIDETOLONG, LONGTOWIDE, TWOPART, MISSING, SURVIVAL, COHORT) + General Abbreviation Rule

> Source: Mplus User's Guide v8, Chapter 20

## One-line summary
Six special-purpose `DATA` command variants reshape the raw data file or derive new variables from it *before* `VARIABLE:`/`DEFINE:`/`MODEL:` ever run — reshaping wide↔long repeated-measures data, splitting a semicontinuous variable into binary+continuous parts, building missing/dropout indicators, building discrete-time event-history indicators, and realigning multiple age/entry cohorts onto a common time axis; this file also captures the one genuinely new, cross-cutting syntax rule stated in Chapter 20's preamble and not written down elsewhere in this skill: every Mplus command and option keyword can be abbreviated to its first four (or more) letters.

## Prerequisite checklist
- [ ] Confirm which reshape/derivation is actually needed — these six commands solve different problems and are not interchangeable
- [ ] These commands run as part of `DATA:` processing, before `VARIABLE:`/`DEFINE:`/`MODEL:` — any variable they create must still be listed in `VARIABLE: NAMES ARE`/`USEVARIABLES ARE` like any other variable in the (possibly reshaped) data file
- [ ] Know whether the resulting variables feed into: a long-format growth/survival/time-series model (`WIDETOLONG`), a wide-format growth model (`LONGTOWIDE`), a two-part/semicontinuous model (`TWOPART`), a missing-data-mechanism/attrition analysis (`MISSING`), a discrete-time survival model built from raw event-time data (`SURVIVAL`), or a multiple-cohort/accelerated-longitudinal design (`COHORT`)
- [ ] These are all listed only as terse option/keyword tables in Chapter 20 with no explanatory prose — if the exact mechanics of a specific command matter for the analysis at hand, confirm behavior against the live chapter (e.g. via WebFetch on the Ch.13 "Special features" HTML page) before writing final syntax, since this file only documents the keyword names, groupings, and defaults actually shown in the PDF

## Option selection logic
| Situation | Choice |
|---|---|
| Data are wide format (one row per subject, separate columns per timepoint) but the model needs long format (one row per subject-occasion) | `DATA WIDETOLONG: WIDE = ...; LONG = ...; IDVARIABLE = ...; REPETITION = ...;` |
| Data are long format and the model needs wide format | `DATA LONGTOWIDE: LONG = ...; WIDE = ...; IDVARIABLE = ...; REPETITION = ...;` |
| A dependent variable is semicontinuous (a spike at a floor/cut value plus a continuous distribution above it) and needs splitting into a binary + continuous pair | `DATA TWOPART: NAMES = ...; CUTPOINT = ...; BINARY = ...; CONTINUOUS = ...; TRANSFORM = ...;` |
| Need binary indicator variables built from the pattern of missingness/dropout in the raw data | `DATA MISSING: NAMES = ...; BINARY = ...; TYPE = MISSING / SDROPOUT / DDROPOUT;` |
| Need discrete-time binary event-history variables built directly from raw survival-time/status variables (rather than declaring `DSURVIVAL` on already-built indicator columns) | `DATA SURVIVAL: NAMES = ...; CUTPOINT = ...; BINARY = ...;` |
| Data come from multiple age/entry cohorts that need realigning onto a common time axis (accelerated longitudinal design) | `DATA COHORT: COHORT IS ...; COPATTERN IS ...; COHRECODE = ...; TIMEMEASURES = ...; TNAMES = ...;` |
| Unsure how much of a keyword needs to be typed anywhere in Mplus syntax | any command/option keyword can be truncated to its first 4 (or more) letters — the bold portion shown in the official manual's option tables is the minimum abbreviation |

## Menu path & screen fields
Categorized option list for each command (Mplus has no GUI — these are `.inp` text blocks):

**DATA WIDETOLONG:**
- `WIDE =` — names of the existing wide-format variables (the repeated columns to be stacked)
- `LONG =` — names of the new long-format variable(s) to create
- `IDVARIABLE =` — variable holding each subject's ID, carried through into the long-format file
- `REPETITION =` — variable that will hold the repetition/occasion index in the long-format file

**DATA LONGTOWIDE:** (mirror image of WIDETOLONG)
- `LONG =` — names of the existing long-format variables
- `WIDE =` — names of the new wide-format variables to create
- `IDVARIABLE =` — variable holding each subject's ID
- `REPETITION =` — variable/values identifying each occasion; default `0, 1, 2, etc.` if not otherwise supplied

**DATA TWOPART:**
- `NAMES =` — variables used to build the binary/continuous pairs
- `CUTPOINT =` — value used to divide each original variable into its binary and continuous parts; default `0`
- `BINARY =` — names of the new binary (spike-indicator) variables
- `CONTINUOUS =` — names of the new continuous-part variables
- `TRANSFORM =` — function applied to the new continuous variables; default `LOG`

**DATA MISSING:**
- `NAMES =` — variables used to build the missingness indicator set
- `BINARY =` — names of the new binary (missing-pattern) variables
- `TYPE =` — `MISSING` (default) / `SDROPOUT` / `DDROPOUT`
- `DESCRIPTIVE =` — sets of variables for additional descriptive statistics, sets separated by the `|` symbol

**DATA SURVIVAL:**
- `NAMES =` — variables used to build the binary event-history variable set
- `CUTPOINT =` — value used to create the set of binary event-history variables from the original variables
- `BINARY =` — names of the new binary variables

**DATA COHORT:**
- `COHORT IS` — name of the cohort variable (its values identify each cohort)
- `COPATTERN IS` — name of a cohort/pattern variable (its patterns identify each cohort)
- `COHRECODE =` — `(old value = new value)` recoding pairs
- `TIMEMEASURES =` — sets of variables (one set per occasion/measure), sets separated by `|`
- `TNAMES =` — root names for the variable sets listed in `TIMEMEASURES`, separated by `|`

**General abbreviation rule** (Chapter 20 preamble; applies across every command in every other reference file in this skill):
- Any Mplus command name or option keyword can be shortened to its first four letters or more, e.g. `ANALYSIS:` → `ANAL:`, `ESTIMATOR` → `ESTIM`
- Within a keyword's set of allowed settings (e.g. `DATA: TYPE = CORRELATION;`), the bold-typed portion in the official manual's tables is the minimum abbreviation Mplus accepts (e.g. `CORR` for `CORRELATION`, `GRAND` for `GRANDMEAN`, `STAND` for `STANDARDIZED`) — spelling the full word out always also works, and this skill's other reference files consistently use full words for clarity

## Sub-option details
- All six commands are part of `DATA:` processing and run before variable typing/analysis — none of them is a substitute for `VARIABLE:`/`DEFINE:`; any variable a reshape/derivation command produces still needs to appear in `VARIABLE: NAMES ARE`/`USEVARIABLES ARE` to be used downstream.
- `WIDETOLONG`/`LONGTOWIDE` are inverses of each other; both need `IDVARIABLE` so records can be correctly matched back to the same subject, and `REPETITION` to track which occasion/column each stacked value came from.
- `TWOPART`'s `CUTPOINT` and `TRANSFORM` defaults (`0` and `LOG`) mean that, unless overridden, the binary part flags values above/below zero and the continuous part is log-transformed — override both explicitly if the semicontinuous variable's natural floor or desired transform differs.
- `MISSING`'s `TYPE=` offers three settings (`MISSING`, `SDROPOUT`, `DDROPOUT`) but Chapter 20 gives no further prose on how they differ — treat this as a menu of available keywords only, and confirm the intended one against the live Ch.13/Ch.15 text before use if the distinction matters for the analysis.
- `SURVIVAL`'s `CUTPOINT`/`BINARY`/`NAMES` options build binary event-history indicators from raw variables — this is a data-preparation alternative to declaring `DSURVIVAL ARE` (in `VARIABLE:`) on variables that already exist as indicators; see `data-variable-define-core.md` for `DSURVIVAL` itself.
- `COHORT`'s `COHRECODE=` uses the same `(old = new)` recoding syntax pattern seen elsewhere in Mplus; `TIMEMEASURES=`/`TNAMES=` both use `|` to separate parallel sets, matching the `|`-as-list-separator convention also used in `DATA MISSING: DESCRIPTIVE=`.
- None of these six commands appear in the DATA sub-option coverage of `data-variable-define-core.md`, which only names them in one summary sentence — this file is the first place their actual keyword/default tables are recorded in this skill.

## Post-run operations
- After a `WIDETOLONG`/`LONGTOWIDE` reshape, check the "SUMMARY OF ANALYSIS" and observation count in the `.out` file — long format should show more rows (one per subject-occasion) than the original wide file, and vice versa; a mismatch usually means `IDVARIABLE`/`REPETITION` weren't set correctly.
- After `TWOPART`, confirm both the new binary and new continuous variables appear as expected in the variable summary, and that the continuous part's distribution (after `TRANSFORM`) looks reasonable among non-floor cases.
- After `MISSING`/`SURVIVAL`, spot-check a few cases' new binary indicator values against the original raw data to confirm the intended coding direction.
- For all six, remember to add the newly created variable names to `VARIABLE: NAMES ARE`/`USEVARIABLES ARE` — a variable created by a `DATA` reshape/derivation command that's missing from `NAMES ARE` will not be readable downstream.

## Likely FAQ mapping
- "My repeated-measures data are one row per subject with separate columns per wave — how do I get long format for a survival or N=1 time-series model?" → `DATA WIDETOLONG`
- "My data are long format (one row per subject-occasion) but I need wide format for a growth model" → `DATA LONGTOWIDE`
- "My outcome has a big spike at a floor value plus a continuous distribution above it (semicontinuous)" → `DATA TWOPART`
- "I need indicator variables flagging which cases/occasions are missing or dropped out" → `DATA MISSING`
- "I need discrete-time event-history indicators built directly from raw event-time/status variables" → `DATA SURVIVAL`
- "My sample was recruited across multiple age/entry cohorts and I need to align them onto one common time axis" → `DATA COHORT`
- "How much of a command or option keyword do I actually have to type?" → first four (or more) letters of the keyword; the bold-typed portion in the official manual's tables is the minimum abbreviation, and spelling the whole word out always works too

# Saving the Analysis Data Set and PLOT SERIES Syntax (Gap-Fill)

> Source: Mplus User's Guide v8, Chapter 13, Example(s) 13.14, 13.15, 13.16

## One-line summary
A short supplement to `output-savedata-plot-commands.md` covering only what that cross-procedure reference doesn't already spell out: the plain `SAVEDATA: FILE IS ...;` behavior for saving the raw analysis data set itself (with or without factor scores added via `SAVE = FSCORES;`), and the `PLOT: SERIES = ...;` syntax that names which variables/time-scores go on a plotted line — most of Ex 13.14-13.16's content (what `SAVE = FSCORES` does, what `TYPE = PLOT1/PLOT2/PLOT3` show) is already covered there and is not repeated here.

## Prerequisite checklist
- [ ] Do you want to save the exact analysis data set (the variables actually used, after any `USEVARIABLES`/`USEOBSERVATIONS` restriction) as its own file for reuse, independent of any TECH-numbered output?
- [ ] Do you additionally want per-case factor scores appended to that saved file (see `output-savedata-plot-commands.md` for the `SAVE = FSCORES;` option itself)?
- [ ] Do you want a graphical display (e.g., observed/estimated trajectories) that needs `PLOT: SERIES = ...;` to know which variables form the line and what x-axis values to use?

## Option selection logic
| Situation | Choice |
|---|---|
| Just want to save the analysis-ready data set (after any variable/observation selection) as a plain file for later use | `SAVEDATA: FILE IS name.sav;` with no `SAVE =` option |
| Want that saved file to also include estimated factor scores per case | add `SAVE = FSCORES;` to the same `SAVEDATA:` command (see `output-savedata-plot-commands.md` for what FSCORES contains) |
| Need a line/trajectory plot (e.g. growth curve) and must tell Mplus which variables/time points define it | `PLOT: SERIES = varlist (x-axis spec);` combined with `TYPE = PLOT1/PLOT2/PLOT3` (see `output-savedata-plot-commands.md` for what each TYPE shows) |

## Menu path & screen fields
**[Saving the analysis data set — Ex 13.14]**
1. **VARIABLE:** `USEVARIABLES ARE ...; USEOBSERVATIONS ARE (...);` (any subsetting as needed)
2. **MODEL:** as usual
3. **SAVEDATA: FILE IS regress.sav;** — no `SAVE =` option; saves exactly the variables/observations used in the analysis

**[Saving the data set plus factor scores — Ex 13.15]**
1. **MODEL:** as usual (e.g. a CFA/MIMIC model)
2. **SAVEDATA: FILE IS mimic.sav; SAVE = FSCORES;**

**[PLOT SERIES for a growth model — Ex 13.16]**
1. **MODEL:** `i s | y11@0 y12@1 y13@2 y14@3;` (growth model defining the slope factor and time scores)
2. **PLOT:**
   - `SERIES = y11-y14 (s);` — the repeated-measures variables, with x-axis values taken from the slope growth factor's time scores
   - `TYPE = PLOT3;` (see `output-savedata-plot-commands.md` for what `PLOT1`/`PLOT2`/`PLOT3` each display)

## Sub-option details
- `SAVEDATA: FILE IS regress.sav;` (no `SAVE =`): saves the individual-level data actually used in the analysis (post `USEVARIABLES`/`USEOBSERVATIONS` filtering) to an ASCII file. The default save format is fixed; use `FORMAT IS ...;` on `SAVEDATA:` to override it.
- `SAVE = FSCORES;`: when added to `SAVEDATA:`, appends estimated factor scores to the same saved data file (rather than only saving raw variables) — the saved file's format defaults to fixed here as well unless `FORMAT IS ...;` is specified.
- `PLOT: SERIES = y11-y14 (s);`: `SERIES` lists a set of variables to be connected as a line in the plot (e.g. repeated measures of one outcome over time). The parenthetical term after the variable list gives the x-axis values: `(s)` means use the time-score values already defined for growth factor `s` in the `MODEL:` command's `|` statement (here, 0, 1, 2, 3 from `y11@0 y12@1 y13@2 y14@3`) rather than typing the x-axis values out again.
- `TYPE = PLOT3;`: requests the graphical output category to generate; the specific plots included under `PLOT1`/`PLOT2`/`PLOT3` are enumerated in `output-savedata-plot-commands.md` and are not repeated here. All PLOT output is viewed after the run completes, via Mplus's post-processing graphics module (not printed inline in the text output).

## Post-run operations
- Open the saved `SAVEDATA` file and confirm its column count/order matches expectations (raw variables only, or raw variables + factor scores if `SAVE = FSCORES;` was used) before feeding it into a downstream program.
- For `PLOT: SERIES`, confirm in the post-processing graphics viewer that the x-axis values shown match the intended time scores (e.g., 0/1/2/3), especially when `(s)` (growth-factor time scores) is used instead of manually listed values.

## Likely FAQ mapping
- "How do I save just the data set I actually analyzed (after selecting a subset of variables/cases)?" → `SAVEDATA: FILE IS name.sav;` with no `SAVE =` option
- "Can I save factor scores together with the raw data in the same file?" → yes, `SAVEDATA: FILE IS name.sav; SAVE = FSCORES;`
- "How do I tell Mplus what the x-axis should be for a growth curve trajectory plot?" → `PLOT: SERIES = varlist (s);` where `s` is the slope growth factor whose `|` statement already defines the time scores
- "What do PLOT1/PLOT2/PLOT3 actually show?" → see `output-savedata-plot-commands.md`'s PLOT options table (not repeated in this file)

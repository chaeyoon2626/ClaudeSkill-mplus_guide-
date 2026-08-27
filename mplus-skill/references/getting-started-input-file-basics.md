# Mplus Input File Basics — The Ten Commands & Minimal Templates

> Source: Mplus User's Guide v8, Chapter 1-2

## One-line summary
Chapters 1-2 are not a modeling procedure but orientation: Chapter 1 explains Mplus's unifying x/y/u/f/c variable framework (background, outcome, latent-continuous, latent-categorical) and lists the broad classes of models/features Mplus can fit; Chapter 2 lists the ten top-level commands that make up every `.inp` file, the mechanical syntax rules for writing them, and four fully-worked minimal templates. Use this file to sanity-check overall file structure and command syntax mechanics; use the procedure-specific reference files for what actually goes inside `MODEL:`.

## Prerequisite checklist
- [ ] Confirm which commands the analysis actually needs — only `DATA:` and `VARIABLE:` are required; everything else is added only as needed
- [ ] If a requested `TYPE=` (e.g. `MIXTURE`, `TWOLEVEL`, `CROSSCLASSIFIED`) seems to silently fail or be unavailable, check the user's Mplus product edition (see below) before assuming a syntax error
- [ ] Have a rough shape of the target model in mind so the closest minimal template (factor-with-covariates, growth, latent-class, or multilevel-regression) can be adapted rather than built from a blank file

## Option selection logic

| Situation | Choice |
|---|---|
| New to Mplus, unsure what commands even exist | Ten commands total; see the list below. Only `DATA` and `VARIABLE` are required |
| Unsure what order commands must appear in | Commands may appear in **any order**; conventional/readable order is `TITLE, DATA, VARIABLE, DEFINE, ANALYSIS, MODEL, OUTPUT, SAVEDATA, PLOT, MONTECARLO` |
| Need the smallest possible working file for a factor model with a covariate (MIMIC) | Adapt the MIMIC template below |
| Need the smallest possible working file for a linear growth model | Adapt the growth-model template below |
| Need the smallest possible working file for a 2-class LCA with a covariate + direct effect | Adapt the LCA template below |
| Need the smallest possible working file for a multilevel (random intercept + random slope) regression | Adapt the multilevel-regression template below |
| Unsure whether `IS`, `ARE`, `=` matter | Interchangeable everywhere **except** inside `DEFINE`, `MODEL CONSTRAINT`, and `MODEL TEST` |
| `TYPE = MIXTURE;` / `TWOLEVEL` / `THREELEVEL` / `CROSSCLASSIFIED` seems unavailable | Check the product edition — Mplus Base alone does not include mixture or multilevel analyses (see Sub-option details) |
| Need to shorten typing | Commands, options, and option settings can be abbreviated to their first 4+ letters (or the bolded portion shown in each command's manual entry) |
| Need to annotate the file | `!` comments out the rest of a line; `!*` ... `*!` comments out a multi-line block |
| A long `NAMES ARE` / variable list is causing problems | Each input record (line) is limited to 90 columns — wrap long lists onto additional lines |

## Menu path & screen fields
Mplus has no GUI menu system for building input — the `.inp` text file itself is the "screen," organized into the ten top-level commands, listed here in the conventional order:

1. **TITLE** (optional) — free-text label for the run, printed at the top of the output
2. **DATA** (**required**) — where/how the data file is read
3. **VARIABLE** (**required**) — variable names, selection, missing-value codes, measurement scale, clustering/weighting/grouping/class structure
4. **DEFINE** (optional) — transform existing variables / compute new ones
5. **ANALYSIS** (optional) — technical estimation details (`TYPE=`, `ESTIMATOR=`, etc.); omit entirely if defaults suffice
6. **MODEL** (optional but almost always present) — the model to be estimated
7. **OUTPUT** (optional) — additional output beyond the default
8. **SAVEDATA** (optional) — save analysis data, auxiliary results, factor scores, etc. to an external file
9. **PLOT** (optional) — request graphical displays of data/results
10. **MONTECARLO** (optional) — details of a Monte Carlo simulation study (data generation and/or analysis)

## Sub-option details

### General syntax mechanics (apply across every command)
- Each command name must start on a **new line** and be followed by a colon, e.g. `DATA:`
- A semicolon ends each option/statement; more than one option is allowed per physical line
- Input records (lines) are limited to **90 columns**
- Mplus is **case-insensitive**; upper case is used in the manual only by convention (commands/keywords upper case, user-supplied information lower case)
- Commands, options, and option settings can be abbreviated to their first **4 or more letters**, or to whatever portion is shown in bold in the manual's command-box listing for that option
- A hyphen (`-`) expands a list of consecutive variables or numbers, e.g. `y1-y5` = `y1 y2 y3 y4 y5`
- The keyword `ALL` can be used wherever the manual documents it, to mean "every variable" in that context
- `IS`, `ARE`, and `=` are interchangeable everywhere **except** inside `DEFINE`, `MODEL CONSTRAINT`, and `MODEL TEST`
- Comments: `!` comments out the remainder of a line; a block can be commented out by starting the first line with `!*` and ending the last line with `*!`

### The x/y/u/f/c variable framework (for reading the manual's own examples)
Mplus's own example input files use a consistent (but not mandatory) naming convention worth recognizing when reading manual examples or other chapters' reference files:
- `x` — observed background/independent variable
- `y` — observed continuous or censored outcome variable
- `u` — observed binary/ordinal (categorical), nominal, or count outcome variable
- `t` — continuous-time survival (time-to-event) variable
- `a` — observed time-varying background variable
- `w` — observed between-level background variable (multilevel)
- `f` — continuous latent variable (factor)
- `c` — categorical latent variable (latent class)
- `i` — intercept growth factor
- `s` or `q` — slope growth factor / random slope
These are illustrative only — actual variable names are never restricted to this scheme (and are still subject to the ≤8-character Latin-alphanumeric naming rule from the DATA/VARIABLE reference file).

### Four minimal worked templates (from Chapter 2, adapted for reference here)
These show the smallest complete file for four different model families — a useful starting skeleton before layering in procedure-specific options from the matching reference file.

**1. Factor analysis with covariates (MIMIC), two factors, six continuous indicators, three covariates:**
```
TITLE:      MIMIC model, two factors, six continuous indicators, three covariates
DATA:       FILE IS mimic.dat;
VARIABLE:   NAMES ARE y1-y6 x1-x3;
MODEL:      f1 BY y1-y3;
            f2 BY y4-y6;
            f1 f2 ON x1-x3;
```

**2. Linear growth model, continuous outcome at 4 time points, two time-invariant covariates:**
```
TITLE:      linear growth model with time-invariant covariates
DATA:       FILE IS growth.dat;
VARIABLE:   NAMES ARE y1-y4 x1 x2;
MODEL:      i s | y1@0 y2@1 y3@2 y4@3;
            i s ON x1 x2;
```

**3. Latent class analysis, two classes, one covariate, one direct effect:**
```
TITLE:      LCA, two classes, one covariate, one direct effect
DATA:       FILE IS lcax.dat;
VARIABLE:   NAMES ARE u1-u4 x;
            CLASSES = c (2);
            CATEGORICAL = u1-u4;
ANALYSIS:   TYPE = MIXTURE;
MODEL:
            %OVERALL%
            c ON x;
            u4 ON x;
```

**4. Two-level (multilevel) regression, one individual-level outcome regressed on an individual-level predictor, with the intercept and random slope regressed on a cluster-level predictor:**
```
TITLE:      multilevel regression with a random intercept and a random slope
DATA:       FILE IS reg.dat;
VARIABLE:   NAMES ARE clus y x w;
            CLUSTER = clus;
            WITHIN = x;
            BETWEEN = w;
            MISSING = .;
DEFINE:     CENTER x (GRANDMEAN);
ANALYSIS:   TYPE = TWOLEVEL RANDOM;
MODEL:
            %WITHIN%
            s | y ON x;
            %BETWEEN%
            y s ON w;
```

### Mplus product editions (why a `TYPE=` might be unavailable)
Mplus is sold in tiers that gate which `TYPE=` settings run, independent of syntax correctness:
- **Mplus Base** — regression/path/CFA/SEM/growth/survival-type analyses; **excludes** `TYPE=MIXTURE`, `TWOLEVEL`, `THREELEVEL`, `CROSSCLASSIFIED`
- **Base + Mixture Add-On** — adds mixture models; still excludes `TWOLEVEL`/`THREELEVEL`/`CROSSCLASSIFIED`
- **Base + Multilevel Add-On** — adds `TWOLEVEL`/`THREELEVEL`/`CROSSCLASSIFIED`; still excludes `TYPE=MIXTURE`
- **Base + Combination Add-On** — every analysis type, no restrictions
If a model with `TYPE=MIXTURE` or a multilevel `TYPE=` fails to run despite correct syntax, this is the first thing to check.

## Post-run operations
There is no single "run" for this orientation content, but two habits carry into every actual analysis:
- Before troubleshooting syntax, confirm the file actually contains the two required commands (`DATA:`, `VARIABLE:`) — everything else is optional and its absence is not itself an error
- If a `TYPE=` setting appears to be silently ignored or rejected, check the product-edition table above before assuming a typo

## Likely FAQ mapping
- "What are all the commands in Mplus, and do I need all of them?" → ten commands total; only `DATA` and `VARIABLE` are required, the rest are added as needed
- "What order do the commands need to go in?" → any order is technically accepted, but the conventional readable order is TITLE, DATA, VARIABLE, DEFINE, ANALYSIS, MODEL, OUTPUT, SAVEDATA, PLOT, MONTECARLO
- "Can I add comments to my .inp file?" → `!` for the rest of a line, `!* ... *!` to block-comment multiple lines
- "Does it matter whether I write IS, ARE, or =?" → no, they're interchangeable, except inside `DEFINE`, `MODEL CONSTRAINT`, and `MODEL TEST`
- "My variable list / a long line seems to break the file" → input records are limited to 90 columns; wrap onto additional lines
- "I asked for TYPE=MIXTURE (or TWOLEVEL) and it's not working" → check which Mplus product edition/add-ons are installed, not just the syntax
- "What's the bare-minimum file I need to get something running?" → adapt the closest of the four minimal templates above, then add procedure-specific detail from the matching reference file
- "What do the y/u/x/f/c/i/s letters mean in the manual's example variable names?" → naming convention only (not a requirement): x=background, y=continuous/censored outcome, u=categorical/nominal/count outcome, f=factor, c=latent class, i/s(or q)=growth intercept/slope factors

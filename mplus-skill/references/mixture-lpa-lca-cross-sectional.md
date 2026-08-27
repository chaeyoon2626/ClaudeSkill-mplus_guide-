# Latent Class / Latent Profile Analysis (LCA/LPA) — Cross-Sectional Mixture Modeling Basics

> Source: Mplus User's Guide v8, Chapter 7 (Mixture Modeling with Cross-Sectional Data), Examples 7.3, 7.9 (+ full table of contents 7.1-7.30)
> https://www.statmodel.com/HTML_UG/chapter7V8.htm
> **Note**: only the table of contents and Examples 7.3 (LCA) and 7.9 (LPA) have been detailed for this chapter. The remaining examples (7.1-7.2, 7.4-7.8, 7.10-7.30 — covariates, known class, CACE, etc.) haven't been written up yet, so if a related question comes in, re-check the relevant page live before answering (never guess).

## One-line summary
Explores how many unobserved subgroups (latent classes) exist based on patterns in observed indicators (items/scores), and how likely each individual is to belong to each group. When the indicators are categorical it's called LCA (latent class analysis); when continuous, LPA (latent profile analysis) — the Mplus syntax is identical either way (`TYPE=MIXTURE`).

## Prerequisite checklist
- [ ] Confirm whether the indicators are categorical (LCA) or continuous (LPA)
- [ ] Range of class counts to explore — usually starting at 2 and increasing sequentially for comparison
- [ ] Whether there's a covariate that's meant to predict class membership or show differences across classes (if so, this is outside this file's scope — needs a live check of 7.12-7.14)
- [ ] Awareness of starting-value sensitivity: mixture models are prone to local optima, so enough random starts are needed

## Option selection logic
| Situation | Choice |
|---|---|
| Indicators are categorical (binary/ordinal) | `CATEGORICAL = u1-u4;` + `CLASSES = c(k);` + `TYPE = MIXTURE;` |
| Indicators are continuous | `CLASSES = c(k);` + `TYPE = MIXTURE;` (no variable-type designation needed, continuous by default) |
| Need to decide the number of classes k | start at k=2 and fit incrementally increasing k → compare BIC, `TECH11`, `TECH14`, entropy, per-class sample size, and interpretability together |
| Concerned about local optima | increase beyond the default (20 4) with e.g. `ANALYSIS: STARTS = 40 8;` |

## Menu path & screen fields
1. **VARIABLE:**
   - `NAMES ARE ...;` / `USEVARIABLES = ...;`
   - `CLASSES = c (2);` — latent class variable name and number of classes
   - `CATEGORICAL = u1-u4;` — LCA only (categorical indicators)
   - `AUXILIARY = x1-x10 (R3STEP);` — auxiliary variables not used to predict class membership, but whose relationship to the classes you want to test afterward (the 3-step method)
2. **ANALYSIS:** `TYPE = MIXTURE;` (+ `STARTS = n m;` if needed)
3. **OUTPUT:** `TECH1 TECH8;` (default) + `TECH11 TECH14;` when deciding the number of classes + `TECH10;` for categorical-indicator fit diagnostics

## Sub-option details
- `CLASSES = c (2);` : `c` is the newly created latent class variable name (can be named freely), `(2)` is the number of classes
- `AUXILIARY = x (R3STEP);` : specifies the 3-step method, which accounts for classification uncertainty when testing the relationship between a covariate and class membership after the fact
- `STARTS = n m;` : n = number of initial random start sets, m = how many of those proceed to final optimization (default 20 4)

## Post-run operations
- Guide the user to decide the final number of classes by jointly weighing: **the point with the smallest BIC**, **the point where the TECH11/TECH14 p-value stops being significant** (meaning no further improvement over the previous k), **entropy close to 1**, **no class with too small a sample size**, and **whether the class profiles are theoretically interpretable**
- Check that the "MODEL ESTIMATION TERMINATED NORMALLY" message appears and that the best log-likelihood value is replicated across multiple starting values (`TECH8` / the "best loglikelihood value replicated" note near the top of the output) — if it isn't replicated, suspect a local optimum and retry with more STARTS
- To use individual class membership in downstream analysis: `SAVEDATA: SAVE = CPROBABILITIES; FILE = classprobs.dat;`

## Likely FAQ mapping
- "I ran LPA and want to decide the number of classes" → guide through the combined BIC + TECH11/TECH14 decision procedure
- "I just want to see the TECH13 results" → clarify that TECH13 is a distributional (skew/kurtosis) fit test, not a class-count decision statistic; if the goal is deciding class count, point to TECH11/TECH14 instead (→ see `output-savedata-plot-commands.md`)
- "I want to see how people differ across classes, together with their characteristics (covariates)" → outside this file's scope (the 7.12-7.14 covariate examples) — note that a live check of that chapter is needed

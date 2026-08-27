# Latent Transition Analysis (LTA) & Hidden Markov Modeling

> Source: Mplus User's Guide v8, Chapter 8, Examples 8.12-8.15
> Bundled source: references/source-pdfs/Chapter8.pdf

## One-line summary
Models how individuals move between unobserved categorical states (latent classes) across two or more measurement occasions using the same class indicators repeated over time; a first-order hidden Markov model estimates measurement error together with the latent state at each occasion, and LTA further characterizes the transition probabilities between occasions, optionally as a function of covariates or with a mover-stayer structure.

## Prerequisite checklist
- [ ] Same set of latent class indicators measured repeatedly at each occasion (e.g., `u11-u15` at time 1, `u21-u25` at time 2)
- [ ] Decide the number of latent classes per occasion (often held equal across occasions) and whether measurement parameters (thresholds) should be held invariant over time
- [ ] Decide whether transitions should be unrestricted (full LTA) or constrained to a mover-stayer structure (some individuals never change class)
- [ ] Whether a covariate (binary/grouping or continuous) is hypothesized to influence the transition probabilities
- [ ] Whether you want logit-based transition parameters (default) or the probability parameterization (`PARAMETERIZATION = PROBABILITY;`) for more directly interpretable transition probabilities
- [ ] Note: the underlying parameterization for models with multiple categorical latent variables is detailed in Chapter 14 — this file covers only the Chapter 8 example syntax

## Option selection logic
| Situation | Choice |
|---|---|
| Single indicator set repeated at every occasion; want to estimate measurement error + latent state jointly (hidden Markov) | one categorical latent variable per occasion (e.g., `CLASSES = c1(2) c2(2) c3(2) c4(2);`), each occasion's indicator regressed only on its own occasion's latent variable |
| Want transition matrices held equal across time (stationary Markov chain) | equality labels: e.g. `[c2#1-c4#1] (1);` on the class intercepts, and `c4 ON c3 (2); c3 ON c2 (2); c2 ON c1 (2);` on the transition regressions |
| Want measurement invariance across time (same indicator thresholds at every occasion) | equality labels on thresholds, e.g. `[u11$1] (1); [u21$1] (1);` — the same label number reused across occasions holds those thresholds equal |
| Two time points; want to see if a covariate shifts transition probabilities | `CLASSES = c1(3) c2(3);` with `c1 c2 ON x;` (continuous x), or via `KNOWNCLASS`/`MODEL cg:` (binary/group covariate) |
| Covariate is a binary/grouping variable observed in the data | `KNOWNCLASS = cg (g = 0 g = 1);` + `MODEL cg:` block with class-specific `c2 ON c1;` per group |
| Continuous covariate influencing transitions | `c1 ON x;` in `%OVERALL%`, plus class-specific `c2 ON x;` inside each `%c1#k%` block (this is "parameterization 2") |
| Want transition probabilities expressed directly as probabilities rather than logits | `ANALYSIS: PARAMETERIZATION = PROBABILITY;` |
| Some individuals never change class (movers vs. stayers) over ≥3 occasions | mover-stayer LTA: add a mover/stayer categorical latent variable `c (2)`; stayer-class transitions fixed at 0/1 via `@0`/`@1`, mover-class transitions freely estimated |
| Want the transition-probability table itself in the output | `OUTPUT: TECH15;` |

## Menu path & screen fields
1. **DATA:** `FILE = ...;`
2. **VARIABLE:**
   - `NAMES = u11-u15 u21-u25 g;` (repeated indicator sets per occasion, plus any covariate)
   - `CATEGORICAL = u11-u15 u21-u25;`
   - `CLASSES = cg (2) c1 (3) c2 (3);` — one categorical latent variable per occasion (`c1`, `c2`, ...), plus `cg` only if using KNOWNCLASS
   - `KNOWNCLASS = cg (g = 0 g = 1);` — only when the covariate is a known/observed group
3. **ANALYSIS:**
   - `TYPE = MIXTURE;`
   - `PARAMETERIZATION = PROBABILITY;` — optional, switches to probability-scale transition parameters
   - `STARTS = 100 20;` / `PROCESSORS = 8;` — as needed for larger LTA models
4. **MODEL:**
   - `%OVERALL%`: `c1 c2 ON cg;` (or `c1 ON x;` plus class-specific `c2 ON x;`) — the transition/covariate regressions
   - `MODEL cg:` (only if KNOWNCLASS used) — group-specific `c2 ON c1;` transition regressions
   - `MODEL c1:` / `MODEL c2:` / ... — class-specific measurement (threshold) parameters for each occasion's indicators, with equality-label numbers in parentheses to enforce measurement invariance
5. **OUTPUT:** `TECH1 TECH8 TECH15;` — TECH15 requests the transition-probability tables

## Sub-option details
- `CLASSES = c1 (3) c2 (3);` : each occasion gets its own categorical latent variable with its own number of classes; occasions are linked via an `ON` regression (`c2 ON c1;`), not via a `|` growth statement
- `[u11$1] (1); [u21$1] (1);` : the same equality-label number (here `1`) placed on thresholds from different occasions holds them equal, i.e., enforces measurement invariance over time; Mplus's list function lets a run of labels be assigned at once, e.g. `(1-5)` across five thresholds
- `c2 ON c1;` in `%OVERALL%` : multinomial logistic regression describing the (baseline) transition probabilities from the class of c1 to the class of c2
- `MODEL cg: %cg#1% c2 ON c1; %cg#2% c2 ON c1;` : lets the transition regression differ by known group `cg` (e.g., separate transition matrices for two observed groups)
- `c2#1 ON c1#1@1; c2#2 ON c1#1@0;` (mover-stayer, Example 8.15) : fixes specific transition-regression coefficients at 0 or 1 to force the stayer class to remain in the same state across occasions, while the mover class (no `@` fixing) has freely estimated transitions
- `PARAMETERIZATION = PROBABILITY;` : re-expresses the transition structure directly as transition probabilities instead of multinomial-logit coefficients; when used with a covariate, typically only the first `c(t) ON c(t-1)` regression is specified in `%OVERALL%`, with later-transition regressions specified inside class-specific blocks (Example 8.14, second part)
- `TECH15` : produces the estimated transition-probability tables (e.g., "probability of staying in class 2 from time 1 to time 2")
- `PROCESSORS = 8;` : parallelizes computation, useful for larger LTA/hidden-Markov models run with many starts

## Post-run operations
- Request `TECH15` to obtain the actual transition probability matrix per class/group — this is usually the main quantity of interest in LTA, more directly useful than raw logit coefficients
- Check "MODEL ESTIMATION TERMINATED NORMALLY" and loglikelihood replication across starts (TECH8), as with any mixture model; LTA/hidden Markov models with many classes and occasions are particularly prone to local optima — increase `STARTS =` and consider `PROCESSORS =` to speed up the search
- For continuous-covariate LTA, the Mplus Editor provides a built-in "LTA calculator" (Mplus menu) that computes latent transition probabilities at chosen covariate values from the parameterization-2 output
- Apply the class-enumeration logic from `mixture-lpa-lca-cross-sectional.md` (BIC, TECH11/TECH14, entropy) per occasion before committing to a fixed number of classes for the full LTA model
- For a mover-stayer model, examine the estimated class sizes of the mover vs. stayer latent class before interpreting transition probabilities, since the stayer class's "transitions" are fixed by definition, not estimated

## Likely FAQ mapping
- "I have the same set of items measured at multiple waves and want to allow for measurement error while tracking latent class over time" → hidden Markov model (Example 8.12): one categorical latent variable per occasion, each with its own indicator set, transition matrices optionally held equal over time
- "I want to know the probability someone moves from class 2 to class 1 between wave 1 and wave 2" → this is the LTA transition probability; request `OUTPUT: TECH15;`
- "Does a covariate (e.g., treatment group) change how likely people are to transition between classes?" → binary/group covariate: `KNOWNCLASS` + `MODEL cg:`; continuous covariate: `c1 ON x;` plus class-specific `c2 ON x;` (Examples 8.13, 8.14)
- "I think some people never change group membership while others do" → mover-stayer LTA (Example 8.15): add a mover/stayer categorical latent variable and fix the stayer class's transition parameters at 0/1
- "I want probabilities, not logit coefficients, in my transition equations" → `PARAMETERIZATION = PROBABILITY;`
- "Should I use LTA or the sequential-process GMM (two categorical latent variables linked by `c2 ON c1`) described in the GMM/LCGA file?" → sequential-process GMM links two *different* growth processes' classes (e.g., two separate outcome sets, each with its own growth factors); LTA/hidden Markov links the *same* repeated indicator(s) observed at multiple waves — pick based on whether you have two distinct constructs (sequential GMM, see `growth-mixture-modeling-lcga.md`) or one construct assessed repeatedly (LTA, this file)

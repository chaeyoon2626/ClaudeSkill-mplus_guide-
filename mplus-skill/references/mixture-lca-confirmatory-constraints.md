# Confirmatory LCA with Parameter Constraints, Multiple Categorical Latent Variables, and Loglinear/RESCOVARIANCES Parameterizations

> Source: Mplus User's Guide v8, Chapter 7, Example(s) 7.13, 7.14, 7.15, 7.16

## One-line summary
Moves LCA from purely exploratory class enumeration to a confirmatory framework: fixing or equating specific threshold parameters to test substantive measurement hypotheses (7.13), modeling two (or more) correlated categorical latent variables at once (7.14), re-expressing associations among categorical latent variables as a loglinear model (7.15), and relaxing local independence for a specific indicator pair within one class using RESCOVARIANCES (7.16).

## Prerequisite checklist
- [ ] A specific substantive hypothesis about the measurement model exists (e.g., two indicators are "parallel measures," an indicator is near-deterministic in one class, an error rate is equal across classes) rather than just wanting to enumerate classes
- [ ] Decide whether the analysis involves one categorical latent variable or more than one (multiple categorical latent variables require per-variable `MODEL <label>:` blocks)
- [ ] If constraining thresholds directly by equality across classes, know that this can be done inline with `(label)`; nonlinear or cross-parameter constraints instead require `MODEL CONSTRAINT`
- [ ] If suspecting a local dependence between two specific indicators within a class (conditional independence violated for that pair only), confirm which indicator pair and which class before adding a residual association
- [ ] Understand thresholds are on the logit scale; fixing a threshold to a large-magnitude value (e.g., ±15) forces the corresponding response probability to (near) 0 or 1, useful for "perfectly measured" categorical latent variables

## Option selection logic
| Situation | Choice |
|---|---|
| Want two class-indicators to behave as parallel measures (equal thresholds within a class) | Give both threshold statements the same label, e.g. `[u2$1-u3$1*-1] (1);` in `%c#1%` and `[u2$1-u3$1*1] (2);` in `%c#2%` |
| Want one indicator to have (near) probability 1 in a specific class | Fix its threshold to an extreme logit value, e.g. `[u1$1@-15];` (probability ≈ 1) or `[u1$1@15];` (probability ≈ 0) — sign convention: negative threshold ⇒ higher probability of scoring 1 |
| Want a parameter in one class to equal the negative of a parameter in another class (or any non-equality algebraic relationship) | Label both parameters via `(pname)`, then define the relationship in `MODEL CONSTRAINT: pname2 = - pname1;` |
| Have two (or more) categorical latent variables that are correlated, each with its own indicators | `CLASSES = cu (2) cy (3);` (name each latent class variable and its number of classes), then a separate `MODEL <label>:` block per latent variable, with `%OVERALL% cu WITH cy;` for the association |
| Want to specify the cu–cy association as a loglinear (log-odds) association instead of a simple correlation-style WITH | `ANALYSIS: PARAMETERIZATION = LOGLINEAR;` then use `WITH` (e.g. `c1 WITH c3; c2 WITH c3;`) to define two-/three-way loglinear association terms among the categorical latent variables |
| Want to refer to one specific pair of classes across two categorical latent variables (e.g., for starting values or restrictions) | Use the `#` notation, e.g. `cu#1 WITH cy#1 cy#2;` |
| Estimating a loglinear model for an observed multi-way frequency table (categorical latent variables perfectly measured by observed variables) | `PARAMETERIZATION = LOGLINEAR;`, one categorical latent variable per observed table variable, each with thresholds fixed to ±15 so the latent variable equals the observed variable, plus `WITH` statements among the categorical latent variables for the desired interaction terms; can add `FREQWEIGHT = w;` if data are entered as a frequency table |
| Suspect local dependence (conditional independence violated) between two specific indicators in one class only | `ANALYSIS: PARAMETERIZATION = RESCOVARIANCES;` then, inside the relevant class-specific block only, add `indicator1 WITH indicator2;` for the residual association; leave it out of the other class(es) to keep conditional independence there |

## Menu path & screen fields
1. **VARIABLE:**
   - `NAMES ARE u1-u4;` (single categorical latent variable) or `NAMES ARE u1-u4 y1-y4;` (two latent variables with separate indicator sets)
   - `CLASSES = c (2);` for one categorical latent variable, or `CLASSES = cu (2) cy (3);` for two (name and class count per variable)
   - `CATEGORICAL = u1-u4;` — lists all categorical indicators across all categorical latent variables
2. **ANALYSIS:**
   - `TYPE = MIXTURE;`
   - `PARAMETERIZATION = LOGLINEAR;` — for loglinear association terms among categorical latent variables (7.14 alternative spec, 7.15)
   - `PARAMETERIZATION = RESCOVARIANCES;` — for residual (local-dependence) associations between binary/ordered categorical indicators under ML (7.16)
   - `STARTS = 0;` — sometimes turned off when the model is fully constrained/deterministic (as in the perfectly-measured loglinear example, 7.15) since there is no class-enumeration search needed
3. **MODEL:**
   - `%OVERALL%` — for a single latent variable, class-invariant defaults plus any `WITH` between multiple categorical latent variables
   - `%c#1% ... %c#2% ...` — class-specific threshold/constraint statements for one latent variable
   - `MODEL cu: %cu#1% ... %cu#2% ...` and `MODEL cy: %cy#1% ... %cy#3% ...` — separate labeled blocks required once more than one categorical latent variable is in the model
4. **MODEL CONSTRAINT:** for algebraic (linear/non-linear) relationships between labeled parameters
5. **OUTPUT:** `TECH1 TECH8;` (as in the base LCA/LPA file)

## Sub-option details
- `[u2$1-u3$1*-1] (1);` : sets starting value -1 for the u2 and u3 thresholds in the current class and assigns them the shared label `(1)`, forcing equality between u2's and u3's thresholds within that class (parallel-measurement hypothesis); a different label in the other class allows the equated pair to still differ across classes
- `[u1$1@-15];` : fixes (not just starts) the threshold at -15 logits, effectively pinning the response probability near a boundary — used both for "this indicator behaves almost deterministically in this class" (7.13) and for "this categorical latent variable is perfectly measured by an observed variable" (7.15)
- `(p1)` / `(p2)` labels + `MODEL CONSTRAINT: p2 = - p1;` : gives two parameters (here, u4's threshold in class 1 and in class 2) labels in `MODEL`, then defines any algebraic relationship between them in `MODEL CONSTRAINT` — needed whenever the relationship is not a plain equality (plain equality just reuses the same label directly)
- `MODEL <latent-var-label>:` : once `CLASSES` lists more than one categorical latent variable, each one needs its own labeled `MODEL` block (label = the latent variable's name) containing that variable's own `%<var>#<class>%` statements; indicators are only allowed to load on their own latent variable's classes
- `cu WITH cy;` in `%OVERALL%` : requests the association between the two categorical latent variables; under the default parameterization this is a correlation-style association, under `PARAMETERIZATION = LOGLINEAR` the `WITH` statement instead defines loglinear (log-odds) association terms
- `cu#1 WITH cy#1 cy#2;` : the alternative, more granular way to specify the same cu–cy association one specific class-pair at a time; useful for supplying starting values or placing restrictions on individual association parameters
- `FREQWEIGHT = w;` : declares that variable `w` holds cell frequencies so the data can be read in as an aggregated multi-way table rather than one row per case (used in the pure loglinear example, 7.15)
- `PARAMETERIZATION = RESCOVARIANCES;` + `u2 WITH u3;` inside one class block only : adds a residual covariance between two categorical indicators for that class alone (partial conditional independence); omitting the WITH statement in other classes keeps the standard local-independence assumption there — this parameterization is required specifically to let WITH apply to binary/ordered categorical outcomes under maximum likelihood

## Post-run operations
- After constraining thresholds, check `TECH1` to confirm the intended parameters actually share the same parameter number (for equality labels) or were fixed as specified — this catches typos in class/threshold numbering before interpreting results
- For `MODEL CONSTRAINT`-based hypotheses, check whether the constrained model fits appreciably worse than an unconstrained version (compare loglikelihoods / use a likelihood-ratio-style comparison appropriate to the estimator) to judge whether the substantive hypothesis (e.g., equal error rates across classes) is tenable
- For the two-categorical-latent-variable models (7.14/7.15), read the `cu WITH cy` (or loglinear `WITH`) output as evidence of association between the two latent typologies — a non-significant association suggests the two classifications are essentially independent of one another
- For `RESCOVARIANCES` models (7.16), a significant residual `WITH` in one class but not fit in the other class means local independence is being selectively relaxed — do not add the same residual term to every class by default, since that would just be a different (larger) model
- Because 7.15's loglinear example fixes thresholds to ±15 and disables random starts (`STARTS = 0;`), there is no class-enumeration decision to make in that setup — it is a fully specified confirmatory loglinear model for an observed table, not an exploratory search

## Likely FAQ mapping
- "I want to force two of my LCA indicators to have identical measurement properties in a class" → equal-label threshold constraint, Example 7.13 (`[u2$1-u3$1*-1] (1);` style)
- "I want one indicator to basically define class membership by itself" → fix its threshold to an extreme value like `[u1$1@-15];`, Example 7.13
- "I have two related categorical outcomes and want to model both sets of classes together, correlated" → two `CLASSES=` entries + one `MODEL <label>:` block per latent variable + `WITH` in `%OVERALL%`, Example 7.14
- "I want to fit a loglinear model to a contingency table using Mplus" → `PARAMETERIZATION = LOGLINEAR;`, one categorical latent variable per table variable with thresholds fixed to ±15, `WITH` for the desired interaction terms, optionally `FREQWEIGHT=`, Example 7.15
- "My LCA has two indicators that seem related beyond what class membership explains, just in one class" → `PARAMETERIZATION = RESCOVARIANCES;` + a class-specific `WITH` for that indicator pair only, Example 7.16
- "How do I write a non-trivial linear constraint between two parameters in Mplus?" → label both parameters with `(name)` in `MODEL`, then define the equation in `MODEL CONSTRAINT`, Example 7.13

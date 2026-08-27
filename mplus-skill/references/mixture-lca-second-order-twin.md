# LCA with a Second-Order Continuous Factor for Paired (Twin) Data

> Source: Mplus User's Guide v8, Chapter 7, Example 7.18

## One-line summary
Specifies a second-order factor model in which two first-order categorical latent class variables (e.g., one per member of a twin pair) each have their own set of categorical indicators, and both are regressed on a single second-order continuous latent factor with an equality constraint forcing the factor to influence both first-order latent variables to the same degree — a design used for studies of twin/paired-unit associations on a latent typology.

## Prerequisite checklist
- [ ] Data must contain two parallel sets of latent-class indicators, one set per member of the pair (e.g., u11-u13 for twin 1 and u21-u23 for twin 2), coded so the two sets are directly comparable item-by-item
- [ ] Confirm the two first-order categorical latent variables should have the same number of classes and (typically) mirrored class-specific measurement structure, since the model treats the two "twins" as exchangeable copies of the same latent typology
- [ ] Understand this design requires numerical integration (`ALGORITHM = INTEGRATION;`), which becomes more computationally demanding as sample size grows — plan for longer run times than a standard LCA
- [ ] Decide whether the equal-influence assumption (the second-order factor affects both first-order latent variables identically) is the hypothesis being tested, since that is what the equality constraint on the two `ON` coefficients encodes by default in this example

## Option selection logic
| Situation | Choice |
|---|---|
| Want two categorical latent variables (one per pair member) linked through a shared continuous second-order factor | `CLASSES = c1(2) c2(2);` with separate indicator sets per latent variable, plus `f BY;` (empty BY list) and `f@1;` to define f as a factor with no first-order indicators of its own and a fixed variance, then `c1 c2 ON f;` |
| Want the second-order factor's influence to be constrained equal across both first-order latent variables (twin 1 and twin 2 treated symmetrically) | Give the two coefficients of `c1 c2 ON f*1 (1);` the same label, e.g. `(1)`, so both multinomial-regression slopes are equal |
| Need each first-order latent variable's own class-specific measurement parameters (thresholds) specified separately | Use labeled `MODEL c1:` and `MODEL c2:` blocks, each with their own `%c1#1%`/`%c1#2%` (or `%c2#1%`/`%c2#2%`) threshold statements |
| Model involves a continuous latent variable with no directly observed indicators, used purely to link other latent variables | `f BY;` (declares f as a latent variable with an empty indicator list) then `f@1;` (fixes its variance, typically to 1, for identification) |
| Need to select the estimator/algorithm appropriate for a categorical-on-continuous-factor regression inside a mixture | `ANALYSIS: ALGORITHM = INTEGRATION;` — maximum likelihood with robust standard errors via numerical integration (one dimension of integration here, using the default 15 integration points) |

## Menu path & screen fields
1. **VARIABLE:**
   - `NAMES ARE u11-u13 u21-u23;` — indicator sets for twin 1 (u11-u13) and twin 2 (u21-u23)
   - `CLASSES = c1(2) c2(2);` — one categorical latent variable per pair member, each with its own class count
   - `CATEGORICAL = u11-u23;` — all class indicators across both latent variables
2. **ANALYSIS:** `TYPE = MIXTURE;` + `ALGORITHM = INTEGRATION;`
3. **MODEL:**
   - `%OVERALL%` block: `f BY;` then `f@1;` (defines the second-order factor with fixed variance and no own indicators), then `c1 c2 ON f*1 (1);` (equality-constrained regression of both first-order latent variables on f)
   - `MODEL c1:` block: `%c1#1%` / `%c1#2%` threshold statements for twin 1's indicators (u11-u13)
   - `MODEL c2:` block: `%c2#1%` / `%c2#2%` threshold statements for twin 2's indicators (u21-u23)
4. **OUTPUT:** `TECH1 TECH8;`

## Sub-option details
- `f BY;` : declares f as a latent variable with an empty list of directly observed indicators — it exists in the model purely as a second-order factor linking c1 and c2, not as a first-order CFA factor with its own manifest indicators
- `f@1;` : fixes the variance of f (here to 1) since, with no indicators of its own, f's scale must be fixed for identification
- `c1 c2 ON f*1 (1);` : specifies the multinomial logistic regression of both first-order categorical latent variables on the continuous second-order factor f, giving both slopes the same starting value (1) and the same label `(1)`, which constrains them to be numerically equal — this equality is what makes f represent a single, symmetric "shared latent trait" influencing both twins identically rather than two separately estimated associations
- `MODEL c1:` / `MODEL c2:` labeled blocks : required once more than one categorical latent variable appears in `CLASSES=`; each block holds that latent variable's own `%c1#k%`/`%c2#k%` class-specific statements (here, threshold sets for the respective indicator group), analogous to the multi-categorical-latent-variable syntax in confirmatory LCA (see mixture-lca-confirmatory-constraints.md)
- `ALGORITHM = INTEGRATION;` : required because c1 and c2 are regressed on a continuous latent variable f; uses one dimension of integration (since there is a single continuous factor) with the default 15 integration points — more factors or larger samples increase computation time substantially

## Post-run operations
- Interpret the shared, equality-constrained `ON` slope from `c1 c2 ON f*1 (1);` as the strength of association between the two members of the pair on the underlying latent typology — a larger (in magnitude) slope indicates that twin 1's and twin 2's class memberships are more strongly linked through the shared factor f
- If the equal-influence assumption is itself the hypothesis under test, compare this constrained model against a version with separate (unlabeled) `c1 ON f;` and `c2 ON f;` slopes to see whether relaxing the equality materially changes fit — a large fit difference would suggest twin 1 and twin 2 are not symmetric in how f relates to their class membership
- Check `TECH1` to confirm both `c1 ON f` and `c2 ON f` slopes were assigned the same parameter number (evidence the equality label was applied as intended) before interpreting the single shared coefficient
- Because `ALGORITHM = INTEGRATION;` is used, confirm convergence and, if runtime is a concern, consider whether the number of integration points needs adjusting for stability versus speed
- Read each first-order latent variable's class-specific thresholds (from the separate `MODEL c1:`/`MODEL c2:` blocks) the same way as in a standard confirmatory LCA to check the two latent typologies are behaving as intended before drawing conclusions about the twin association itself

## Likely FAQ mapping
- "I have twin (or other paired-unit) data with a latent class outcome for each member and want to model their association" → second-order factor twin design, Example 7.18 (`f BY; f@1; c1 c2 ON f*1 (1);`)
- "How do I create a continuous latent variable that has no indicators of its own, just to link two other latent variables?" → `f BY;` (empty indicator list) plus `f@1;` to fix its variance for identification
- "How do I force two regression coefficients on a categorical latent variable to be exactly equal?" → give them the same parenthesized label, e.g. `c1 c2 ON f*1 (1);`, which equates the two slopes
- "Why does this model need ALGORITHM=INTEGRATION when my earlier LCA models didn't?" → because a categorical latent variable (c1, c2) is being regressed on a continuous latent factor (f), which requires numerical integration; plain LCA/LPA without such a regression does not need it
- "Can I let the twin association differ between twin 1 and twin 2 instead of assuming symmetry?" → drop the shared label and specify separate `c1 ON f;` / `c2 ON f;` statements (or distinct labels) instead of `c1 c2 ON f*1 (1);`

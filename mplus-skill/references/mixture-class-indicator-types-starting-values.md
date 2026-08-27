# LCA/LPA Class-Indicator Types and Starting-Value Strategy

> Source: Mplus User's Guide v8, Chapter 7, Examples 7.4-7.11
> https://www.statmodel.com/HTML_UG/chapter7V8.htm

## One-line summary
Covers two orthogonal setup decisions for any class-enumeration mixture model (LCA/LPA): (1) how to declare each class indicator's measurement level — binary, ordinal, unordered categorical (nominal), censored, or count — and (2) how to control the random-start/optimization search (`STARTS`, `STITERATIONS`) and, optionally, supply user-specified starting values for thresholds or means instead of relying on Mplus's automatic ones.

## Prerequisite checklist
- [ ] Confirm the class count `CLASSES = c (k);` is already tentatively fixed — this file is about how to specify indicators and starting values for a given k, not about choosing k itself (see `mixture-lpa-lca-cross-sectional.md`)
- [ ] For each indicator, decide its measurement level: binary/ordinal (`CATEGORICAL`), unordered categorical (`NOMINAL`), censored with a floor/ceiling (`CENSORED`), count (`COUNT`), or continuous (default, no declaration needed)
- [ ] Decide whether to rely on Mplus's automatic starting values (the default and usually sufficient) or supply user-specified starting values — typically only needed when automatic search keeps landing on a poor or uninterpretable local solution
- [ ] If using user-specified starting values, decide whether to also allow random perturbations around them (`STARTS > 0`) or fix the solution exactly (`STARTS = 0;`)

## Option selection logic
| Situation | Choice |
|---|---|
| Indicator is binary/ordinal | `CATEGORICAL = u1-u4;` — thresholds referenced as `u1$1`, `u1$2`, ... (number of thresholds = number of categories minus one) |
| Indicator is unordered categorical (no natural ordering among categories) | `NOMINAL = u1-u4;` — categories referenced as `u1#1`, `u1#2`, ... (one fewer parameter than categories; program infers category count from the data) |
| Indicator is continuous with a floor or ceiling effect | `CENSORED = y1 (b);` for censoring from below (floor), or the matching keyword for censoring from above; censoring limit is taken from the data |
| Indicator is a count variable | `COUNT = u3;` for a Poisson class indicator, or `COUNT = u3 (i);` to additionally estimate a zero-inflated Poisson part for that indicator (`u3#1` refers to the inflation part) |
| Indicator is continuous with no floor/ceiling/count structure | No declaration needed — continuous is the default |
| Model has a mix of indicator types | Combine the relevant options in one `VARIABLE` command, e.g. `CATEGORICAL = u1; CENSORED = y1 (b); NOMINAL = u2; COUNT = u3 (i);` (Example 7.11) |
| Automatic starting values are adequate (typical first attempt) | Omit `MODEL` thresholds/means entirely, or leave `STARTS` at its default (20 initial-stage sets, 4 final-stage optimizations) |
| Automatic search is landing on poor/uninterpretable solutions | Increase `STARTS = n m;` (more initial and/or final-stage sets) and/or `STITERATIONS = n;` (more initial-stage iterations per start) |
| Want to force the model toward a specific, theory-driven solution and skip random perturbation entirely | `STARTS = 0;` plus user-specified starting values (`*value`) on thresholds/means in `MODEL`, one class-specific block per class |
| Want user-specified starting values but still explore nearby random perturbations | Keep `STARTS = n m;` (n > 0) and give starting values with `*value` — Mplus randomly perturbs around the supplied values |

## Menu path & screen fields
1. **VARIABLE:**
   - `NAMES ARE ...;`
   - `CLASSES = c (k);`
   - Indicator-type declarations as needed: `CATEGORICAL = ...;`, `NOMINAL = ...;`, `CENSORED = ... (b);`, `COUNT = ... (i);` (combine as needed for mixed indicator sets)
2. **ANALYSIS:**
   - `TYPE = MIXTURE;`
   - `STARTS = n m;` — n = number of initial-stage random starts, m = number carried to final-stage optimization (default 20 4); `STARTS = 0;` disables random starts entirely
   - `STITERATIONS = n;` — maximum iterations allowed per start in the initial stage (default 10)
3. **MODEL:** (only needed for user-specified starting values)
   - `%OVERALL%`
   - `%c#1%`, `%c#2%`, ... class-specific blocks with bracket statements giving starting values, e.g. `[u1$1*1 u2$1*1 u3$1*-1 u4$1*-1];`
4. **OUTPUT:** `TECH1 TECH8;`

## Sub-option details
- `CATEGORICAL = u1-u4;` : each listed variable's thresholds are referenced as `varname$#`, e.g. `u1$1` for a binary indicator's single threshold. Thresholds and the mean of the latent class variable are estimated by default; thresholds are NOT held equal across classes by default.
- `[u1$1*1 u4$1*-1];` inside `%c#1%` : the asterisk assigns a starting value on the logit scale to the threshold that follows it — here u1's threshold starts at 1 and u4's at -1 for class 1. Different values are typically given per class to break symmetry and help the optimizer find distinct classes.
- Ordinal indicators with 3+ categories: each indicator needs one starting value per threshold, listed in increasing order within each class, e.g. `[u1$1*.5 u1$2*1];` (Example 7.6).
- `NOMINAL = u1-u4;` : declares unordered categorical indicators; categories are referenced as `varname#category`, e.g. `u1#1`, `u1#2` for a 3-category nominal indicator (one parameter per category beyond the reference category). Starting values for nominal-indicator means use the same bracket + `*value` syntax, e.g. `[u1#1-u4#1*0]; [u1#2-u4#2*1];` (Example 7.8).
- `CENSORED = y1 (b);` : the `b` (below) marks y1 as censored from below (a floor effect); the model estimated is a censored regression for that indicator. The censoring limit is determined from the data, not specified by the user.
- `COUNT = u3 (i);` : the `i` requests a zero-inflated Poisson model for count indicator u3; the inflation part is referenced as `u3#1`.
- `STARTS = 100 10;` `STITERATIONS = 20;` (Example 7.5) : random perturbations are generated around user-specified starting values rather than around Mplus's automatic ones; increasing both the number of starts and the per-start iteration cap improves the chance of finding the global optimum but increases run time proportionally.
- For continuous class indicators (LPA), starting values are given for means rather than thresholds, e.g. `[y1-y4*1];` for class 1 and `[y1-y4*-1];` for class 2 (Example 7.10) — means and variances of the indicators, plus the mean of the categorical latent variable, are estimated by default; variances are held equal across classes by default (covariances among indicators are fixed at zero by default regardless of indicator type).

## Post-run operations
- After any run using non-default `STARTS`, check that the best log-likelihood value is replicated across multiple random starts (reported near the top of the output and in `TECH8`) — non-replication signals a local optimum, and `STARTS` should be increased further.
- When user-specified starting values are used with `STARTS = 0;` (no random starts at all), there is no replication check available — treat the result as provisional and re-verify with `STARTS > 0` before finalizing, since a single fixed start can converge to a local optimum without any warning.
- For a censored indicator, check the reported floor (or ceiling) proportion in the sample statistics to confirm the censoring limit derived from the data matches expectations.
- For a nominal indicator, read the estimated `#`-labeled parameters as a multinomial logistic model within each class, referencing the same (implicit) last category as the baseline.
- For a zero-inflated count indicator (`COUNT = u (i);`), interpret `u#1` output as the logistic part governing whether an individual is forced to zero, and the main `u` parameters as the Poisson-part rate given not forced to zero — this is distinct from using a whole latent class to represent the inflation, which is covered as a special case in the mixture-as-device reference file.

## Likely FAQ mapping
- "My mixture model keeps giving me a different best solution every time I re-run it" → increase `STARTS = n m;` (and/or `STITERATIONS`) to search more starting points; check for log-likelihood replication in the output
- "How do I give Mplus my own starting values for a mixture model instead of the automatic ones?" → use bracket statements with `*value` inside class-specific `%c#k%` blocks; combine with `STARTS = 0;` to skip random starts entirely, or leave `STARTS` positive to perturb around the supplied values
- "One of my class indicators has categories with no natural order" → `NOMINAL = varname;` instead of `CATEGORICAL`
- "One of my indicators is a count that's also floor/ceiling limited or has excess zeros" → `COUNT = var;` for a plain Poisson indicator, `COUNT = var (i);` if that specific indicator's own zero-inflation should be modeled directly (distinct from using a whole class to represent excess zeros — see the mixture-as-device file for that alternative)
- "Can I mix binary, ordinal, nominal, censored, and count indicators in the same LCA?" → yes, declare each with its own VARIABLE-command keyword in a single model (Example 7.11)
- "What happens if I don't declare a class indicator's type at all?" → it's treated as continuous by default (an LPA indicator)

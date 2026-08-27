# Multiple Group Mixture Modeling with Known Classes (KNOWNCLASS)

> Source: Mplus User's Guide v8, Chapter 7, Example 7.21
> Bundled source: references/source-pdfs/Chapter7.pdf

## One-line summary
Runs a mixture model across two or more OBSERVED groups (e.g., site, gender, treatment arm) at the same time as estimating a genuinely UNOBSERVED latent class variable, by declaring one categorical latent variable's classes to be "known" and defined from an existing grouping variable in the data — this is TYPE=MIXTURE's equivalent of multiple-group analysis.

## Prerequisite checklist
- [ ] Have an observed grouping variable in the data set (e.g., g) whose values code known group membership
- [ ] Decide whether the model should also include a genuinely unobserved latent class variable (e.g., c) whose membership is predicted by group, or whether the known-class variable alone is what's needed
- [ ] Confirm the number of classes for BOTH the known-class variable and (if present) the unobserved latent class variable
- [ ] Decide which parameters should be allowed to differ across the known groups (analogous to deciding which parameters are group-varying in ordinary multiple-group SEM)

## Option selection logic
| Situation | Choice |
|---|---|
| Need a standard multiple-group mixture analysis where groups are observed, not latent | `CLASSES = cg (2) c (2);` + `KNOWNCLASS = cg (g = 0 g = 1);` where cg is the known-class variable and g is the observed grouping variable in the data |
| Want the unobserved latent class c's prevalence to differ by known group, or want to test whether group predicts latent class membership | Add `c ON cg;` in `%OVERALL%` |
| Want indicator means to vary across the unobserved latent classes c | Specify them inside `MODEL c: %c#1% ... %c#2% ...` |
| Want indicator variances to vary across the known groups cg instead | Specify them inside `MODEL cg: %cg#1% ... %cg#2% ...` |
| More than two known groups | List every value mapped to a class, e.g. `KNOWNCLASS = cg (g = 0 g = 1 g = 2);` for a 3-group cg variable |

## Menu path & screen fields
1. **VARIABLE:**
   - `NAMES = g y1-y4;` — g is the observed grouping/data variable
   - `CLASSES = cg (2) c (2);` — cg: known-class (group) variable; c: unobserved latent class variable
   - `KNOWNCLASS = cg (g = 0 g = 1);` — maps values of g to the classes of cg
2. **ANALYSIS:** `TYPE = MIXTURE;`
3. **MODEL:**
   - `%OVERALL%` : `c ON cg;` — multinomial logistic regression of the unobserved class c on the known group cg
   - `MODEL c:` block with `%c#1% ... %c#2% ...` — indicator means that vary across the unobserved classes
   - `MODEL cg:` block with `%cg#1% ... %cg#2% ...` — parameters (e.g., indicator variances) that vary across the known groups
4. **OUTPUT:** `TECH1 TECH8;`

## Sub-option details
- `KNOWNCLASS = cg (g = 0 g = 1);` : identifies cg as the categorical latent variable whose class membership is known and observed; the information in parentheses defines each known class from the values of the data variable g — here class 1 = individuals with g=0, class 2 = individuals with g=1
- `c ON cg;` : describes the multinomial logistic regression of the (still genuinely latent/unobserved) class variable c on the known group variable cg — lets you test whether the unobserved classes' prevalence differs by known group
- `MODEL c: %c#1% [y1-y4]; %c#2% [y1-y4];` : specifies that the means of y1-y4 vary across the classes of the unobserved latent variable c (the usual class-indicator mean specification, placed under a `MODEL c:` label because the model now has more than one categorical latent variable)
- `MODEL cg: %cg#1% y1-y4; %cg#2% y1-y4;` : specifies that the variances of y1-y4 vary across the classes of the known group variable cg (i.e., between-group heteroscedasticity); by default the means of y1-y4 vary across the classes of c and the variances vary across the classes of cg
- When a model contains more than one categorical latent variable, each needs its own `MODEL <name>:` label so Mplus knows which class-specific statements belong to which categorical latent variable

## Post-run operations
- Check the estimated `c ON cg` coefficients to see whether the observed grouping variable predicts unobserved class membership (e.g., whether one site has a higher prevalence of a given profile)
- Verify per-known-group per-latent-class sample sizes are large enough to support separately estimated parameters — known-class mixture models effectively multiply the number of class-specific parameter sets
- As with any mixture model, confirm "MODEL ESTIMATION TERMINATED NORMALLY" and loglikelihood replication across starts before trusting the solution
- If the substantive goal is really just testing measurement invariance of the indicators across an observed group (not adding a genuinely unobserved class), consider whether a simpler continuous-outcome multiple-group model (see cfa-multigroup-invariance.md) fits the question better than adding an unnecessary unobserved class

## Likely FAQ mapping
- "I have data from two known sites/arms/cohorts and want to run the same mixture model separately in each while comparing something across them" → `KNOWNCLASS` multiple-group mixture modeling, Example 7.21
- "Does my observed group variable predict which latent class people are in?" → `c ON cg;` after specifying `KNOWNCLASS`
- "I only have one categorical latent variable (the known group) and no separate unobserved classes" → still specify a `CLASSES=` entry for it and use `KNOWNCLASS`; the unobserved `c` latent variable is only needed if there's a genuinely unobserved classification to estimate
- "How do I let indicator variances differ by known group but indicator means differ by an unobserved class?" → split the specification across `MODEL cg:` (variances by known group) and `MODEL c:` (means by latent class), as in Example 7.21

# Exploratory Factor Mixture Analysis

> Source: Mplus User's Guide v8, Chapter 4, Example 4.4
> https://www.statmodel.com/HTML_UG/chapter4V8.htm

## One-line summary
Assumes the sample consists of unobserved latent classes with different factor structures (or the same structure but different parameters), and simultaneously explores class membership and each class's factor structure.

## Prerequisite checklist
- [ ] Confirm the indicators are continuous (per the manual's example)
- [ ] How many latent classes to explore (e.g. 2)
- [ ] Range of factor counts to explore within each class (e.g. 1-2)
- [ ] Confirm whether the user theoretically expects "classes truly differ" vs. "a single-group EFA would be enough" — the number of classes needs to be decided statistically too, not purely by prior assumption

## Option selection logic
| Situation | Choice |
|---|---|
| Expect the factor structure to differ across latent classes | `TYPE = MIXTURE EFA a b;` (a-b is the within-class factor-count range) |
| Need to explore the number of classes itself too | fit repeatedly while varying the class count (`CLASSES = c(k);`) and compare BIC/entropy |

## Menu path & screen fields
1. **VARIABLE:** `NAMES = y1-y8;` / `CLASSES = c(2);` — latent class variable c, 2 classes
2. **ANALYSIS:** `TYPE = MIXTURE EFA 1 2;` — mixture analysis + explore 1-2 factors within each class

## Sub-option details
- `CLASSES = c(2);` : creates the latent categorical variable c, fixed at 2 classes (can refit with a different number)
- `TYPE = MIXTURE EFA 1 2;` : combines MIXTURE and EFA, estimating a separately rotated factor solution per class
- Default rotation = oblique GEOMIN, default estimator = robust ML

## Post-run operations
- Deciding the number of classes: guide the user to jointly consider BIC (smaller is better), entropy (closer to 1 means clearer class separation), and whether any class's sample size is too small
- Check each class's factor-loading table separately to interpret whether the structure genuinely differs across classes
- Individual posterior class-membership probabilities can be saved with `SAVEDATA: SAVE = CPROBABILITIES;` if needed (explain further on request)

## Likely FAQ mapping
- "It seems like our sample splits into heterogeneous subgroups" → guide toward factor mixture analysis
- "How do I decide the number of classes?" → explain the BIC/entropy comparison procedure

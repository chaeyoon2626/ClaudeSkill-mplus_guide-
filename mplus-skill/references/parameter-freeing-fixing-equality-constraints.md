# Freeing, Fixing, and Equality-Constraining Parameters

> Source: Mplus User's Guide v8, Chapter 13, Example(s) 13.8-13.10
> Bundled source: references/source-pdfs/Chapter13.pdf

## One-line summary
Shows the MODEL-command syntax for overriding Mplus's default free/fixed parameter status, supplying starting values, and constraining parameters to be equal — within a single group or across groups in a multiple-group model.

## Prerequisite checklist
- [ ] Do you know which parameters Mplus fixes by default in your model (e.g., the first indicator's loading after `BY` is fixed to 1 to set the factor metric)?
- [ ] Do you need any parameter to differ from that default (free a fixed one, fix a free one, or set a specific starting value)?
- [ ] Do you want two or more parameters forced to be equal — within a single group, or across groups in a multiple-group model?
- [ ] If multiple groups: do you know which parameters should stay equal across ALL groups vs. which should be allowed to differ for specific groups?

## Option selection logic
| Situation | Choice |
|---|---|
| A factor loading Mplus fixed by default (e.g., first indicator) should instead be estimated | add `*` after that variable in the `BY` statement |
| A parameter that's free by default should instead be fixed to a specific value | add `@value` after it |
| You want to supply a custom starting value instead of the default | add `*value` after it |
| Two or more parameters (loadings, variances, residual variances, etc.) should be equal, single group | give them the same `(label)` in parentheses |
| Same idea, but across groups in a multiple-group model | put the equality `(label)` in the overall `MODEL:` command |
| One specific group should be allowed to deviate from an overall-model equality, or needs its own group-specific equality | override or add it in that group's `MODEL <groupname>:` command |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE: NAMES ARE ...;** (+ `GROUPING IS grp (1=g1 2=g2 3=g3);` for multiple-group models)
4. **MODEL:** (overall — applies to all groups if multiple-group)
   - `f1 BY y1* y2*.5 y3;` — free/fix loadings, optionally with starting values
   - `f1-f2@1;` — fix a list of parameters (e.g., factor variances) to a value
   - `y2-y3 (1-2);` — assign equality labels to a list of parameters using the list function
   - `y1-y3 (3);` — hold a list of parameters equal to each other with one shared label
5. **MODEL <groupname>:** (multiple-group only; one block per group that needs group-specific settings)
   - overrides/relaxes an overall-model equality, or adds a group-specific free parameter/loading/intercept

## Sub-option details
- `f1 BY y1* y2*.5 y3;`: the first indicator (y1) after `BY` is fixed to 1 by default to set the factor's metric; adding `*` after y1 frees it. `y2*.5` frees y2's loading and gives it a starting value of .5 (a bare `*` with no number just frees it, using the default starting value).
- `f1-f2@1;`: fixes the variances of f1 and f2 to 1 — an alternative way of setting the factor metric instead of fixing a loading to 1.
- `y2-y3 (1-2);`: uses the list function to assign consecutive equality labels 1 and 2 to y2's and y3's loadings respectively, so they can be paired with same-numbered loadings elsewhere (e.g., y5, y6) to hold them equal to each other.
- `y1-y3 (3);`: a single shared label (3) holds all three parameters (e.g., residual variances of y1, y2, y3) equal to one another.
- In a multiple-group model, an equality label placed in the overall `MODEL:` command holds that parameter equal **across all groups**. A `MODEL <groupname>:` command can then free a specific group's copy of that parameter (e.g., `f1 BY y3*;` in `MODEL g1:` to let y3's loading differ in group 1) or apply its own equality label to hold something equal across only some groups (e.g., a shared label used only in `MODEL g1:` and `MODEL g3:` to equate a factor variance between g1 and g3, leaving g2 free to differ).

## Post-run operations
- Check `OUTPUT: TECH1;` to confirm which parameters ended up free/fixed/labeled as intended before trusting the results.
- For equality-constrained models, compare fit against a freely estimated (unconstrained) version if a formal test of the constraint is desired.
- If starting values were supplied to aid convergence, confirm in the output that estimation still terminated normally rather than getting stuck near the starting value.

## Likely FAQ mapping
- "How do I free the factor loading of the first indicator instead of leaving it fixed to 1?" → add `*` after it in the `BY` statement
- "How do I force two loadings/paths to be equal?" → give them the same `(label)`
- "In my multi-group CFA, how do I hold a loading equal across groups vs. let it differ?" → label it in the overall `MODEL:` command for full equality, or override/relax it in group-specific `MODEL <g>:` blocks
- "My model won't converge — can I give it better starting values?" → `varname*startvalue`
- "How do I fix a parameter to a specific value instead of estimating it?" → `varname@value`

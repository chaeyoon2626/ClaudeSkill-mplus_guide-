# Second-Order (Hierarchical) Factor Analysis

> Source: Mplus User's Guide v8, Chapter 5, Example 5.6
> https://www.statmodel.com/HTML_UG/chapter5V8.htm

## One-line summary
A CFA where several first-order factors are themselves treated as indicators of a single higher-order (second-order) factor — used when a broad construct is theorized to be expressed through several narrower sub-constructs.

## Prerequisite checklist
- [ ] Confirm the first-order structure first: how many first-order factors, and which items measure each (same as basic CFA)
- [ ] Confirm the theoretical claim that these first-order factors are themselves explained by one common higher-order factor (if instead you just want the first-order factors to correlate freely with no higher-order structure, plain multi-factor CFA — `cfa-continuous.md` — is enough, and is a less restrictive model)
- [ ] At least 3 first-order factors are typically needed for the second-order factor to be identified (2 is only identified under extra constraints)

## Option selection logic
| Situation | Choice |
|---|---|
| Believe a single broad construct explains correlations among several first-order factors | second-order factor: add `f5 BY f1-f4;` on top of the first-order `BY` statements |
| Just want first-order factors to correlate, no higher-order claim | plain multi-factor CFA (`cfa-continuous.md`), don't add the second-order statement |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE: NAMES ARE ...;**
4. **MODEL:**
   - `f1 BY y1-y3;` / `f2 BY y4-y6;` / `f3 BY y7-y9;` / `f4 BY y10-y12;` — first-order factors, each measured by its own item set
   - `f5 BY f1-f4;` — the second-order factor, "measured by" the first-order factors themselves

## Sub-option details
- `f5 BY f1-f4;` : uses the identical `BY` syntax as a normal measurement statement, but the items on the right-hand side are latent factors instead of observed variables — this is what makes it "second-order"
- The metric of both levels is set the same way: fixing the first loading in each `BY` statement to 1 (so f1's metric is set via y1, and f5's metric is set via f1)
- First-order factor residual (disturbance) variances are freely estimated by default, with no assumed correlation among them; the second-order factor's own variance is also freely estimated

## Post-run operations
- Check whether the second-order factor loadings (f1-f4 on f5) are all substantial and significant — a weak loading from one first-order factor suggests it may not really belong under the proposed higher-order construct
- Compare fit against the plain (first-order-only, freely correlated) CFA — the second-order model is more restrictive (it constrains the first-order factor correlations to be fully explained by the single higher-order factor), so a notably worse fit than the unconstrained model is a sign the second-order claim doesn't hold
- Standardized second-order loadings (`OUTPUT: STDYX;`) are typically the main quantity of interest for interpreting a hierarchical structure

## Likely FAQ mapping
- "I think these sub-scales are all really measuring one bigger thing" → second-order factor model
- "Do I need at least a certain number of first-order factors for this to work?" → mention that 3+ is typical for identification; 2 needs extra constraints
- "How is this different from just letting the factors correlate?" → explain the second-order model is a constrained special case of the freely-correlated model, and can be compared to it

# Bi-Factor Models via ESEM Rotation

> Source: Mplus User's Guide v8, Chapter 5, Examples 5.29, 5.30
> Bundled source: references/source-pdfs/Chapter5.pdf

## One-line summary
Fits a bi-factor structure (one general factor shared by all items, plus specific/group factors) using exploratory structural equation modeling (ESEM) rotation — either rotating the general and specific factors together with a bi-factor rotation criterion, or fixing the general factor CFA-style and rotating only the specific factors.

## Prerequisite checklist
- [ ] Confirm a bi-factor structure is theoretically expected (a common construct spanning all items, plus item-subgroup-specific constructs) — if a simple multi-factor structure is expected instead, plain `esem-continuous.md` or `cfa-continuous.md` may be more appropriate
- [ ] Decide whether *all* factors (general + specific) should be rotated together (full bi-factor ESEM rotation), or whether the general factor should be fixed with a known/near-complete loading pattern (CFA-style) while only the specific factors are rotated
- [ ] Confirm which items, if any, are meant to load on the general factor only (no specific-factor loading)
- [ ] Note this local Ch.5 approach differs from — but is closely related to — the pure EFA bi-factor procedure in `efa-bifactor.md` (Ch.4); this file covers the ESEM/SEM-chapter variant, including the version where the general factor is CFA-fixed

## Option selection logic
| Situation | Choice |
|---|---|
| Rotate the general factor and all specific factors together | `ANALYSIS: ROTATION = BI-GEOMIN;` with all factors listed in one `BY (*1)` EFA block |
| Want the general and specific factors to be fully independent | `ROTATION = BI-GEOMIN(ORTHOGONAL);` |
| Want an alternative bi-factor rotation criterion | `ROTATION = BI-CF-QUARTIMAX;` |
| Some items should load only on the general factor, not on any specific factor | fix the general factor CFA-style (plain `BY` with metric set via `@1` on the variance) measured by *all* items, then define the specific factors as an EFA `(*1)` set measured only by the subset of items that should have specific loadings, with `ROTATION = GEOMIN;` (not `BI-GEOMIN`, since only the specific block is being rotated) |

## Menu path & screen fields
**[Bi-factor EFA using ESEM, all factors rotated together — 5.29]**
1. **VARIABLE:** `NAMES ARE y1-y10;`
2. **ANALYSIS:** `ROTATION = BI-GEOMIN;`
3. **MODEL:** `fg f1 f2 BY y1-y10 (*1);` — fg (general), f1, f2 (specific) all form one EFA set measured by all 10 items
4. **OUTPUT:** `STDY;`

**[Bi-factor EFA with two items loading only on the general factor — 5.30]**
1. **ANALYSIS:** `ROTATION = GEOMIN;`
2. **MODEL:**
   - `fg BY y1-y10*;` `fg@1;` — general factor fixed CFA-style, measured by all 10 items; asterisk frees the first loading (metric set instead by fixing the factor variance to 1)
   - `f1-f2 BY y1-y8 (*1);` — the two specific factors, an EFA set measured by only y1-y8 (leaving y9, y10 to load on the general factor only)
   - `fg WITH f1-f2@0;` — fixes the general/specific covariances to zero (specific factors remain oblique to each other under GEOMIN, but are orthogonal to the general factor)
3. **OUTPUT:** `STDY;`

## Sub-option details
- `ROTATION = BI-GEOMIN;` : a bi-factor-specific oblique rotation criterion — the default allows the specific factors to correlate with the general factor and with each other; `ROTATION = BI-GEOMIN(ORTHOGONAL);` forces full independence instead; `ROTATION = BI-CF-QUARTIMAX;` is an alternative bi-factor rotation criterion
- `fg f1 f2 BY y1-y10 (*1);` (Ex 5.29) : listing the general and specific factors together in one `(*1)`-labeled `BY` statement means *all* of them are rotated jointly using the bi-factor rotation criterion — none of the loadings are fixed to zero going in; the rotation itself produces the bi-factor pattern
- `fg BY y1-y10*; fg@1;` + `f1-f2 BY y1-y8 (*1);` (Ex 5.30) : here fg is specified CFA-style (a fixed, estimated loading on every item, metric set by fixing the factor variance), while only f1 and f2 form the rotated EFA block — and that block only includes y1-y8, which is what forces y9 and y10 to load exclusively on the general factor. `ROTATION = GEOMIN;` (not `BI-GEOMIN`) is used here because only the *specific* factors need rotating, not a full general+specific bi-factor rotation
- `fg WITH f1-f2@0;` : explicitly zeroes out the general-specific covariances — needed in the mixed CFA/EFA specification (5.30) since fg isn't part of the rotated block and wouldn't otherwise default to being uncorrelated with f1/f2
- `OUTPUT: STDY;` : requests standardization with respect to y (puts results in conventional EFA-loading metric) — the natural standardization choice for rotated bi-factor output

## Post-run operations
- Check that the general factor's loadings are reasonably even across all (or nearly all) items — a hallmark of a well-formed bi-factor structure
- Check that each specific factor's loadings concentrate within its intended item subgroup, and (for Ex 5.30-style models) confirm the items intended to load on the general factor only show negligible specific-factor loadings
- Compare the bi-factor solution's fit/interpretability against a simpler multi-factor EFA/ESEM model (`esem-continuous.md`) to judge whether the bi-factor structure is actually warranted
- If the bi-factor structure is unstable or hard to interpret, consider the small-variance-prior Bayesian approach in `bayesian-cfa-mimic.md` (Example 5.31), which handles bi-factor cross-loadings more flexibly

## Likely FAQ mapping
- "I want a bi-factor model but I'm not sure of the exact loading pattern" → full bi-factor ESEM rotation (`BI-GEOMIN`), Example 5.29 pattern
- "I want a bi-factor model where two items load only on the general factor" → mixed CFA (general) + EFA (specific) specification, Example 5.30 pattern
- "How is this different from the bi-factor EFA in the EFA chapter?" → conceptually the same rotation-based idea (`efa-bifactor.md`), but this Ch.5/ESEM version additionally supports fixing the general factor CFA-style and mixing it with a rotated specific-factor block
- "Do the specific factors have to be independent of each other and the general factor?" → default is oblique (correlated); use `ORTHOGONAL` or an explicit `WITH ... @0;` statement to force independence

# Regression — Random Coefficient Regression

> Source: Mplus User's Guide v8, Chapter 3, Example 3.9
> https://www.statmodel.com/HTML_UG/chapter3V8.htm

## One-line summary
A model used when you assume the relationship (slope) between a predictor and the dependent variable differs across individuals/groups, and want to explain that slope heterogeneity (a random effect) with another variable.

## Prerequisite checklist
- [ ] Are the dependent variable y and the predictor x1 (which will get the random slope) confirmed? (both assumed continuous)
- [ ] Is there a moderator variable x2 that's meant to explain "individual differences in the slope"?
- [ ] Whether to grand-mean center the predictors (usually recommended for ease of interpretation)
- [ ] Confirm the data isn't truly hierarchical/multilevel and that a single-level random coefficient is really what's needed (if it's genuinely clustered/multilevel data, a separate procedure like `TYPE=TWOLEVEL RANDOM` is needed — note this if that manual isn't available yet)

## Option selection logic
| Situation | Choice |
|---|---|
| The strength of the x1→y relationship differs across people, and you want to explain why with x2 | Random coefficient regression (`TYPE=RANDOM`) |
| Also interested in the covariance between the random slope and the intercept (residual) | Add `s WITH y;` |
| Just want to know whether heterogeneity exists at all, don't need to explain the cause | Omit x2 from `y s ON x2;`, just check the variance of `s` |

## Menu path & screen fields
1. **TITLE:**
2. **DATA: FILE IS ...;**
3. **VARIABLE: NAMES ARE ...;**
4. **DEFINE:** `CENTER x1 x2 (GRANDMEAN);` — center the predictors (optional but recommended)
5. **ANALYSIS:** `TYPE = RANDOM;` — required to estimate random coefficients
6. **MODEL:**
   - `s | y ON x1;` — defines the random slope `s` (regress y on x1, but let the slope be random)
   - `s WITH y;` — freely estimate the residual covariance between the slope and the intercept (y)
   - `y s ON x2;` — explain both the intercept and the slope with the moderator x2

## Sub-option details
- `s | y ON x1;` : the `s` before `|` is the name of the newly created latent random-coefficient variable
- `TYPE = RANDOM;` : the required ANALYSIS option for estimating random slopes
- `CENTER (GRANDMEAN)` : makes the intercept interpretable as "y at the average x value" (not required, but interpretation becomes harder without it)
- `y s ON x2;` : regresses both y and s on x2 at once — can also be written as two separate lines (`y ON x2; s ON x2;`)

## Post-run operations
- In the results, check the mean and variance of `S` (the random coefficient) — a significant variance means "individual differences in the slope exist"
- If the `S ON x2` coefficient is significant, "x2 explains those individual differences"
- Add `OUTPUT: STDYX;` if standardized coefficients are needed

## Likely FAQ mapping
- "It seems like the effect of x on y is different for different people" → guide toward random coefficient regression
- "Why does the intercept look strange?" → check whether centering was applied
- "Is this the same as a multilevel model (HLM)?" → similar, but genuinely clustered data needs a `TYPE=TWOLEVEL` family procedure — note that manual isn't currently available and request the relevant chapter PDF

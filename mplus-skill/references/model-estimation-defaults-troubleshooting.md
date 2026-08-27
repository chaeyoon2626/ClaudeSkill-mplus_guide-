# Model Estimation Defaults & Troubleshooting

> Source: Mplus User's Guide v8, Chapter 14 (Special Modeling Issues) — this chapter contains no numbered Examples; it is a technical-discussion chapter. Sections used: Parameter Default Settings, Parameter Default Starting Values, Multiple Solutions for Mixture Models, Convergence Problems, Model Identification, Numerical Integration
> Bundled source: references/source-pdfs/Chapter14.pdf

## One-line summary
Explains what Mplus fixes/frees by default and what starting values it assigns, then gives systematic diagnostic steps for the most common cross-cutting estimation problems — non-convergence, non-identification, multiple/local solutions in mixture models, and slow or unstable numerical integration — that can arise in almost any Mplus analysis (not tied to one specific model type).

## Prerequisite checklist
- [ ] What is the actual symptom? (a) unsure which parameters are free/fixed by default, (b) the run does not converge, (c) an error says the model is not identified, (d) a mixture-model run gives a different "best" solution on repeated runs, (e) an analysis with latent variables/random slopes/interactions is very slow or numerically unstable
- [ ] Does the model involve `TYPE=MIXTURE`, random slopes defined with the `|` symbol, latent-variable interactions, or multilevel data? (these are the situations most prone to convergence/integration issues)
- [ ] For non-convergence: has `OUTPUT: TECH1;` (parameter free/fixed status) and `TECH5`/`TECH8` (optimization history) already been requested and inspected?
- [ ] For mixture models: how many sets of starting values were used (`STARTS =`), and is the best (highest) loglikelihood value replicated across at least two final-stage solutions?
- [ ] Are the continuous observed variables on similar scales (sample variances roughly between 1 and 10)?

## Option selection logic
| Situation | Choice |
|---|---|
| Unsure which parameters are free/fixed, or what their starting values are | Request `OUTPUT: TECH1;` — lists every parameter's free/fixed status and starting value |
| Continuous observed variables have sample variances far outside the 1–10 range | Rescale with `DEFINE:` (divide by a constant if variance is large, multiply if small) before modeling |
| Run stops before max iterations reached, and preliminary estimates show no large negative variances/residual variances | Increase iterations, or re-run using the preliminary parameter estimates as new starting values |
| Run stops before max iterations reached, and large negative variances/residual variances appear in preliminary estimates | Try new starting values, focusing on variance/residual-variance parameters |
| Run reaches max iterations without converging (difficulties optimizing the fitting function) | New starting values are needed (again, variance/residual-variance parameters first); if that fails, reconsider the model |
| A random-effect variable (random slope via `ON` with `\|`, or a growth factor) has a variance close to zero and causes convergence trouble | If it's a random slope defined via `ON \|`, switch to a fixed effect using a plain `ON` statement instead; if it's a growth factor, fix its variance and related covariances to zero |
| Convergence is hard to achieve for a large, complex model | Build up the model in stages — estimate parts of it separately first to get good starting values for the full model |
| Mplus reports the model is not identified | Use `OUTPUT: TECH1;` to translate the reported parameter number to its name; add further restrictions to the model |
| A parameter's identification status is unclear even though the model overall is identified | Check `OUTPUT: TECH2;` (derivatives) and `MODINDICES` — a fixed parameter with a zero modification index/derivative would not be identified if freed |
| Mixture model: best loglikelihood not replicated by at least 2 final-stage solutions | Increase `STARTS = <#initial random sets> <#final-stage optimizations>;` (default `STARTS = 20 4;`), optionally with more `STITERATIONS` |
| Mixture model: several near-best final-stage solutions have similar loglikelihoods | Compare their parameter estimates (e.g. re-run a specific seed via `OPTSEED` in `ANALYSIS`); very similar estimates → trust the best-LL solution; substantially different estimates → the model may not be well defined for the data (e.g. too many classes) |
| Model requires ML estimation of a latent variable whose posterior has no closed form | Numerical integration is invoked automatically |
| More than 3 dimensions of numerical integration (many latent variables / random slopes / latent-variable interactions) | Reduce points per dimension to 10, or switch to Monte Carlo integration, e.g. `ANALYSIS: INTEGRATION = MONTECARLO(5000);` |
| Numerical instability with the default adaptive integration (outliers, non-normal latent-variable distribution, small cluster sizes) | Turn adaptive integration off: `ANALYSIS: ADAPTIVE = OFF;` |
| `TECH8` output shows large negative values in the ABS CHANGE column | Increase the number of integration points to improve numerical precision |
| Want to check timing/spec before a full run of a slow model | `ANALYSIS: MITERATIONS = 1; STARTS = 0;` together with `OUTPUT: TECH1; TECH8;` — reports per-iteration time and confirms specification without a full run |

## Menu path & screen fields
These are diagnostic ANALYSIS/OUTPUT/DEFINE options layered on top of whatever substantive model (regression, CFA, mixture, growth, etc.) is already being estimated — there is no separate "procedure" screen.

1. **DEFINE:** (optional, before ANALYSIS) — `newvar = var/10;` or `newvar = var*10;` to rescale a variable whose sample variance falls far outside 1–10
2. **ANALYSIS:**
   - `STARTS = 100 20;` — raise the number of initial-stage random starting-value sets (default 20) and final-stage optimizations (default 4) for mixture models
   - `STITERATIONS = 20;` — raise the number of iterations per initial-stage start (default 10)
   - `MITERATIONS = 1;` with `STARTS = 0;` — quick one-iteration check run
   - `ADAPTIVE = OFF;` — disable adaptive numerical integration
   - `INTEGRATION = MONTECARLO(5000);` — switch to Monte Carlo integration with a set number of total points (default 500 if MONTECARLO used without a number); or a plain number, e.g. `INTEGRATION = 10;`, to set rectangular/Gauss-Hermite points per dimension (default 15)
   - `OPTSEED = <seed>;` — re-estimate the model starting from one specific seed, to inspect that particular final-stage solution's parameter estimates in isolation
3. **OUTPUT:**
   - `TECH1;` — free/fixed status and starting value of every parameter (also used to translate a non-identification error's parameter number to its name)
   - `TECH2;` — derivatives (paired with `MODINDICES` to check identification of a fixed parameter)
   - `TECH5;` / `TECH8;` — optimization history; distinguishes the two types of non-convergence (stopped at max iterations vs. stopped due to optimization difficulty) and shows the ABS CHANGE column used to judge numerical-integration precision
   - `MODINDICES;` — modification indices, paired with `TECH2` for identification checks

## Sub-option details
- Defaults, in brief (see the manual for the complete list): observed-independent-variable means/variances/covariances are not modeled (model is conditional on them); in single-group analysis intercepts/thresholds of observed dependent variables are free; continuous-latent-variable means/intercepts are fixed at 0 in single-group analysis (and in the first group / last class of multi-group or multi-class analysis); regression coefficients are fixed at 0 unless mentioned in `MODEL`; residual covariances among dependent variables are fixed at 0 with a few named exceptions
- Starting-value defaults: means/intercepts of continuous & censored variables = 0 or the sample mean (depending on the analysis); count-variable means/intercepts = 0; thresholds = 0 or determined by sample proportions; latent-variable variances/residual variances = .05 or 1; continuous/censored observed-variable residual variances = .5 × sample variance; loadings for continuous-latent-variable indicators = 1; scale factors = 1; all other parameters = 0
- Threshold-to-probability translation for choosing mixture/LCA starting values (logit scale, note the sign is opposite of logit intercepts): very low probability ≈ threshold +3; low ≈ +1; high ≈ −1; very high ≈ −3
- Growth mixture starting-value strategies: (1) estimate a one-class or regular growth model first, then set multi-class starting values at the mean ± ½ SD of the growth factors; (2) estimate a multi-class model with growth-factor variances/covariances fixed at 0 first, then use its factor-mean estimates as starting values for the full (variances free) model
- Computational burden of numerical integration by number of dimensions: 1 = light, 2 = moderate, 3–4 = heavy, 5+ = very heavy; burden increases exponentially with dimensions (for rectangular/Gauss-Hermite) and linearly with the number of observations
- Adaptive integration (Mplus default) typically needs only ~15 points/dimension vs. 30–50 for non-adaptive; exploratory factor analysis may need as few as 3 points/dimension
- A low condition number (e.g. below 1.0E-6) can indicate non-identification that a singular-information-matrix check alone might miss

## Post-run operations
- After raising `STARTS`, check the Loglikelihood/Seed/Initial-Stage-Starts table: trust the result only if the best (highest) loglikelihood value is replicated across at least two (preferably more) final-stage solutions
- If several final solutions cluster near the best loglikelihood, compare their parameter estimates: near-identical estimates → pick the best-LL one; divergent estimates → treat as a signal that the model isn't well determined by the data (e.g. re-check the number of mixture classes)
- For non-convergence, use `TECH5`/`TECH8` to determine which of the two failure types occurred, then apply the matching remedy above
- For non-identification, map the reported parameter number to a name via `TECH1`, then add restrictions or reconsider the model's specification
- For numerical-integration runs, re-check `TECH8`'s ABS CHANGE column after any change to confirm the instability is resolved before trusting final estimates

## Likely FAQ mapping
- "My model won't converge, what do I do?" → this file (Convergence Problems / General Convergence Problems / random-effects-specific convergence)
- "Mplus says my model is not identified" → this file (Model Identification, using TECH1/TECH2/MODINDICES)
- "My mixture/LCA model gives a different 'best' solution depending on the seed" → this file (Multiple Solutions for Mixture Models, STARTS/STITERATIONS/OPTSEED)
- "My growth mixture / multilevel / interaction model is extremely slow or gives unstable results" → this file (Numerical Integration, ADAPTIVE, INTEGRATION, computational burden table)
- "What starting values / free-fixed defaults does Mplus use if I don't specify anything?" → this file (Parameter Default Settings, Parameter Default Starting Values)

# Mplus Procedure Index

Official Mplus User's Guide (v8) full table of contents: https://www.statmodel.com/html_ug.shtml

This table tracks "procedures written up as local reference files." All 20 chapters have now been read in full and written up — every numbered example across the guide is covered by one of the 79 files in `references/`. Match the question's keywords against the "Procedures — detailed" table below and read the matching `references/<file>.md` directly. Live lookup via `WebFetch` is now only a fallback for the two things that are genuinely outside the 20 chapters (the standalone online Index page and the separate Examples-collection page), or for a future manual revision this index hasn't been re-synced to yet.

**Rule: if a chapter/procedure isn't in this table at all, never make it up — say "I couldn't find this procedure in the Mplus User's Guide table of contents" and ask the user to confirm. Never fabricate Mplus options from general training knowledge alone, even for a "Done" chapter — if a follow-up question needs an option that genuinely isn't in the matched reference file, say so and do a live lookup rather than guessing.**

Note: every `references/*.md` file in this skill is an original written summary/how-to produced by reading the official guide — none of them, nor this skill package, includes or redistributes the manual's own text, figures, or PDF files. That's also why example coverage occasionally spans multiple files per chapter (procedures were split or grouped by what's actually a distinct technique, not by strict example-number order).

## Procedures — detailed (local reference file exists)

| Procedure | Matching keywords | Reference file | Source chapter (examples) | Status |
|---|---|---|---|---|
| Mplus input file basics (command order, syntax mechanics) | input file structure, TITLE DATA VARIABLE MODEL, command order, comments, abbreviation, minimal template, product editions | `getting-started-input-file-basics.md` | Ch.1-2 | Done |
| Continuous / censored regression | regression, linear regression, multiple regression, censored, floor effect, ceiling effect | `regression-continuous-censored.md` | Ch.3 Ex 3.1-3.3 | Done |
| Categorical / nominal / count regression | logistic, probit, odds ratio, ordinal, nominal, multinomial logit, Poisson, count, zero-inflated, overdispersion | `regression-categorical-count.md` | Ch.3 Ex 3.4-3.8, 3.10 | Done |
| Random coefficient regression | random coefficient, random slope | `regression-random-coefficient.md` | Ch.3 Ex 3.9 | Done |
| Basic path analysis | path analysis, mediator, mediation model | `path-analysis-basic.md` | Ch.3 Ex 3.11-3.15 | Done |
| Mediation bootstrap & missing data | mediation, indirect effect, bootstrap, missing data | `path-analysis-mediation-bootstrap-missing.md` | Ch.3 Ex 3.16-3.17 | Done |
| Moderated mediation | moderated mediation, conditional indirect effect | `path-analysis-moderated-mediation.md` | Ch.3 Ex 3.18 | Done |
| EFA with continuous indicators | exploratory factor analysis, EFA, factor analysis, rotation, geomin, ESEM | `efa-continuous.md` | Ch.4 Ex 4.1 | Done |
| EFA with categorical/mixed indicators | categorical EFA, Likert factor analysis, mixed indicators, numerical integration | `efa-categorical-mixed.md` | Ch.4 Ex 4.2-4.3 | Done |
| Exploratory factor mixture analysis | factor mixture, latent class EFA | `efa-mixture.md` | Ch.4 Ex 4.4 | Done |
| Two-level EFA | multilevel factor analysis, two-level EFA, clustering, hierarchical data | `efa-twolevel.md` | Ch.4 Ex 4.5-4.6 | Done |
| Bi-factor EFA | bi-factor, general factor, specific factor | `efa-bifactor.md` | Ch.4 Ex 4.7 | Done |
| CFA with continuous indicators (+ MIMIC) | CFA, confirmatory factor analysis, MIMIC, factor with covariates | `cfa-continuous.md` | Ch.5 Ex 5.1, 5.8 | Done |
| CFA with categorical/censored/count indicators | categorical CFA, censored CFA, count CFA, ordinal factor indicators | `cfa-categorical-mixed.md` | Ch.5 Ex 5.2, 5.4 | Done |
| CFA with mixed indicator types & constrained mean/threshold structure | mixed indicators, equivalent test forms, constrained intercepts, constrained thresholds, parallel forms | `cfa-mixed-mean-threshold-structure.md` | Ch.5 Ex 5.3, 5.9, 5.10 | Done |
| IRT models via CFA | IRT, item response theory, 2PL, 3PL, 4PL, graded response model, GPCM, discrimination, difficulty, guessing parameter | `cfa-irt-models.md` | Ch.5 Ex 5.5 | Done |
| Second-order (hierarchical) factor analysis | second-order factor, higher-order factor, hierarchical factor model | `cfa-second-order.md` | Ch.5 Ex 5.6 | Done |
| Non-linear CFA & latent variable interactions | non-linear CFA, quadratic factor, latent interaction, XWITH, latent moderation, curvilinear | `cfa-nonlinear-interactions.md` | Ch.5 Ex 5.7, 5.13 | Done |
| SEM — structural paths among latent factors | SEM, structural equation modeling, latent variable mediation, factor regression, indirect effect between factors | `sem-structural-paths.md` | Ch.5 Ex 5.11, 5.12 | Done |
| Multiple-group CFA & measurement invariance (continuous) | multiple group CFA, measurement invariance, configural, metric, scalar, partial invariance, GROUPING | `cfa-multigroup-invariance.md` | Ch.5 Ex 5.14, 5.15 | Done |
| Multiple-group CFA & measurement invariance (categorical) | categorical measurement invariance, threshold structure, scale factor, Delta parameterization, Theta parameterization | `cfa-multigroup-categorical-invariance.md` | Ch.5 Ex 5.16, 5.17 | Done |
| CFA/EFA parameter constraints via MODEL CONSTRAINT | MODEL CONSTRAINT, NEW option, reliability, inequality constraint, Heywood case, DO loop, residual variance > 0 | `cfa-efa-parameter-constraints.md` | Ch.5 Ex 5.20, 5.28 | Done |
| ESEM with continuous indicators | ESEM, exploratory structural equation modeling, EFA with covariates, EFA MIMIC, EFA longitudinal invariance, multiple-group EFA | `esem-continuous.md` | Ch.5 Ex 5.24-5.27 | Done |
| Bi-factor models via ESEM rotation | bifactor ESEM, BI-GEOMIN, general factor, specific factor, bifactor rotation | `esem-bifactor.md` | Ch.5 Ex 5.29, 5.30 | Done |
| Bayesian CFA / MIMIC (BSEM) | Bayesian CFA, BSEM, ESTIMATOR=BAYES, small-variance priors, approximate measurement invariance, MODEL PRIORS, cross-loadings | `bayesian-cfa-mimic.md` | Ch.5 Ex 5.31-5.33 | Done |
| Twin & sibling behavioral genetics models (ACE / QTL) | twin model, ACE model, additive genetic, heritability, QTL, sibling model, IBD sharing, liability threshold | `twin-behavioral-genetics-ace-qtl.md` | Ch.5 Ex 5.18, 5.19, 5.21-5.23 | Done |
| Basic linear/quadratic growth model | growth model, latent growth curve, intercept slope, linear growth, quadratic growth, estimated time scores | `growth-linear-basic.md` | Ch.6 Ex 6.1, 6.8, 6.9 | Done |
| Growth model with covariates / piecewise growth | time-invariant covariate, time-varying covariate, piecewise growth, TSCORES, random slope | `growth-covariates-piecewise.md` | Ch.6 Ex 6.10-6.12 | Done |
| Growth modeling for censored/categorical/count outcomes | censored growth, censored-inflated, categorical growth, Delta/Theta parameterization, count growth, Poisson, zero-inflated Poisson | `growth-censored-categorical-count-outcomes.md` | Ch.6 Ex 6.2-6.7 | Done |
| Growth modeling — parallel-process, multi-indicator, two-part, autocorrelated, multiple-cohort | parallel process growth, multiple indicator growth, second-order growth, two-part growth, semicontinuous, autocorrelated residuals, PWITH, multiple cohort, accelerated longitudinal design | `growth-parallel-multiindicator-twopart.md` | Ch.6 Ex 6.13-6.18 | Done |
| Survival analysis (discrete & continuous time) | survival analysis, discrete-time survival, Cox regression, proportional hazards, time-to-event | `survival-analysis-discrete-continuous-time.md` | Ch.6 Ex 6.19-6.22 | Done |
| N=1 time series — autoregressive & cross-lagged (observed variables) | N=1 time series, single-subject, autoregressive, AR(1), AR(2), cross-lagged panel, VAR(1), LAGGED | `time-series-n1-autoregressive-crosslagged.md` | Ch.6 Ex 6.23-6.25 | Done |
| N=1 time series — dynamic factor analysis & IRT models | dynamic factor analysis, DAFS, white noise factor score, N=1 IRT, autoregressive factor model | `time-series-n1-dynamic-factor-irt.md` | Ch.6 Ex 6.26-6.28 | Done |
| Latent class/profile analysis basics (LCA/LPA) | LCA, LPA, latent class analysis, latent profile analysis, mixture model, class enumeration | `mixture-lpa-lca-cross-sectional.md` | Ch.7 Ex 7.3, 7.9 | Done |
| LCA/LPA with covariates predicting class membership | c ON x, multinomial logistic regression, direct effect, class-varying regression | `mixture-lca-covariates-predictors.md` | Ch.7 Ex 7.1, 7.2, 7.12 | Done |
| Auxiliary variables & 3-step approach (R3STEP) | AUXILIARY, R3STEP, three-step approach, distal outcome, classification uncertainty | `mixture-auxiliary-variables-r3step.md` | Ch.7 Ex 7.3 (aux) | Done |
| LCA class-indicator types & starting-value strategy | STARTS, STITERATIONS, random starts, local optima, ordinal/nominal/censored/count class indicators | `mixture-class-indicator-types-starting-values.md` | Ch.7 Ex 7.4-7.11 | Done |
| Confirmatory LCA with parameter constraints & multiple categorical latent variables | confirmatory LCA, two correlated categorical latent variables, PARAMETERIZATION=LOGLINEAR, RESCOVARIANCES, local dependence | `mixture-lca-confirmatory-constraints.md` | Ch.7 Ex 7.13-7.16 | Done |
| Mixture CFA & structural equation mixture modeling | mixture CFA, factor mixture, class-varying structural path, structural equation mixture modeling | `mixture-cfa-sem-combined.md` | Ch.7 Ex 7.17, 7.19, 7.20 | Done |
| LCA with a second-order factor for paired (twin) data | twin analysis, second-order factor, paired data, equality-constrained regression | `mixture-lca-second-order-twin.md` | Ch.7 Ex 7.18 | Done |
| Mixture components as a statistical device (correlated indicators, ZIP-as-class, semiparametric) | multivariate normal mixture, correlated indicators, zero-inflated Poisson as class, semiparametric factor, non-parametric histogram | `mixture-as-device-correlated-zip-semiparametric.md` | Ch.7 Ex 7.22, 7.25, 7.26 | Done |
| Multiple-group mixture modeling (KNOWNCLASS) | KNOWNCLASS, multiple group mixture, known class vs unobserved latent class | `mixture-multiple-group-known-class.md` | Ch.7 Ex 7.21 | Done |
| CACE estimation for randomized trials | CACE, complier average causal effect, randomized trial, TRAINING, compliance, exclusion restriction | `mixture-cace-randomized-trials.md` | Ch.7 Ex 7.23-7.24 | Done |
| Factor (IRT) mixture & twin ACE models via numerical integration | factor mixture, IRT mixture, ALGORITHM=INTEGRATION, twin ACE with integration, 2PL IRT, zygosity | `mixture-factor-irt-twin-ace-integration.md` | Ch.7 Ex 7.27-7.29 | Done |
| Continuous-time survival (Cox) with a treatment/control mixture class | Cox regression, continuous-time survival, treatment effect known class, LOGRANK, baseline hazard | `mixture-survival-cox-treatment-class.md` | Ch.7 Ex 7.30 | Done |
| Growth mixture modeling (GMM) & LCGA | growth mixture model, latent class growth analysis, TYPE=MIXTURE, distal outcome, sequential process GMM | `growth-mixture-modeling-lcga.md` | Ch.8 Ex 8.1-8.11 | Done |
| Latent transition analysis (LTA) & hidden Markov | hidden Markov model, latent transition analysis, transition probabilities, mover-stayer | `latent-transition-analysis-hidden-markov.md` | Ch.8 Ex 8.12-8.15 | Done |
| Survival mixture analysis (discrete & continuous time) | survival mixture, discrete-time survival mixture, Cox regression mixture, TYPE=MIXTURE survival | `survival-mixture-analysis.md` | Ch.8 Ex 8.16-8.17 | Done |
| Two-level regression & path analysis | TYPE=TWOLEVEL, WITHIN, BETWEEN, CLUSTER, random intercept, random slope, contextual effect | `two-level-regression-path-basics.md` | Ch.9 Ex 9.1-9.5 | Done |
| Two-level CFA & SEM | two-level CFA, two-level SEM, random factor loading, multilevel MIMIC | `two-level-cfa-sem.md` | Ch.9 Ex 9.6-9.11 | Done |
| Complex survey design corrections | TYPE=COMPLEX, STRATIFICATION, WEIGHT, sampling weights, sandwich estimator, SUBPOPULATION | `complex-survey-design-multilevel.md` | Ch.9 (introductory syntax) | Done |
| Two-level growth models for longitudinal/clustered data | two-level growth, multilevel growth curve, random slope TVC, two-level survival, random-loading MIMIC | `two-level-growth-longitudinal.md` | Ch.9 Ex 9.12-9.19 | Done |
| Three-level models (regression, path, MIMIC, growth) | TYPE=THREELEVEL, three-level regression, three-level growth, slope of a slope, nested clustering | `three-level-models.md` | Ch.9 Ex 9.20-9.23 | Done |
| Cross-classified models (non-nested grouping) | TYPE=CROSSCLASSIFIED, cross-classified multilevel, non-nested clustering, cross-classified IRT | `cross-classified-models.md` | Ch.9 Ex 9.24-9.27 | Done |
| Two-level random residual variance (heteroscedasticity) | heteroscedasticity, random residual variance, logv, two-level regression/CFA variance modeling | `two-level-random-residual-variance.md` | Ch.9 Ex 9.28-9.29 | Done |
| Two-level & cross-classified time series (AR/cross-lagged) | multilevel time series, autoregressive, cross-lagged, cross-classified time series, IRT time series | `two-level-crossclassified-time-series-ar.md` | Ch.9 Ex 9.30-9.40 | Done |
| Multilevel mixture modeling / two-level LCA | TYPE=TWOLEVEL MIXTURE, two-level LCA, multilevel mixture, between-level class | `multilevel-mixture-two-level-lca-basics.md` | Ch.10 Ex 10.1-10.3, 10.6, 10.7 | Done |
| Two-level CFA / IRT mixture models | two-level factor mixture, two-level IRT mixture, random intercept/threshold mixture | `multilevel-mixture-two-level-cfa-irt.md` | Ch.10 Ex 10.4-10.5 | Done |
| Two-level growth mixture, LCGA & LTA | two-level GMM, two-level LCGA, two-level LTA, three-level growth mixture | `multilevel-mixture-two-level-growth-lta.md` | Ch.10 Ex 10.8-10.13 | Done |
| Missing data mechanisms & multiple imputation | missing data, MCAR, MAR, NMAR, FIML, dropout, DATA IMPUTATION, TYPE=IMPUTATION | `missing-data-mechanisms-imputation.md` | Ch.11 Ex 11.1-11.6 | Done |
| Bayesian estimation — imputation & plausible values | Bayesian estimation, ESTIMATOR=BAYES, plausible values, FSCORES, model-based imputation | `bayesian-estimation-plausible-values.md` | Ch.11 Ex 11.7-11.8 | Done |
| Monte Carlo simulation studies (power analysis) | MONTECARLO, MODEL POPULATION, NREPS, power, coverage, bias, simulation | `monte-carlo-simulation-power.md` | Ch.12 Ex 12.1-12.4, 12.6-12.7; Ch.19 (full) | Done |
| Monte Carlo — specialized data/model types | EFA Monte Carlo, survival Monte Carlo, two-part Monte Carlo, two-level mediation power, multiple-group EFA Monte Carlo | `monte-carlo-specialized-model-types.md` | Ch.12 Ex 12.5, 12.8-12.12 | Done |
| Data input, missing values & variable selection/transformation | summary data, covariance matrix input, fixed format, missing value flags, USEVARIABLES, DEFINE | `data-input-missing-variable-selection.md` | Ch.13 Ex 13.1-13.7 | Done |
| Freeing, fixing & equality-constraining parameters | freeing parameters, fixing parameters, starting values, equality constraints, parameter labels | `parameter-freeing-fixing-equality-constraints.md` | Ch.13 Ex 13.8-13.10 | Done |
| PWITH residual covariances & DIFFTEST chi-square difference testing | PWITH, adjacent residual covariance, DIFFTEST, WLSMV/MLMV nested model test | `pwith-difftest-advanced-model-testing.md` | Ch.13 Ex 13.11-13.12 | Done |
| Analyzing externally-imputed multiple imputation data sets | TYPE=IMPUTATION, external imputation, imputed data list file | `multiple-imputation-external-datasets.md` | Ch.13 Ex 13.13 | Done |
| Saving the analysis data set & PLOT SERIES syntax (gap-fill) | SAVEDATA FILE, SAVE=FSCORES, PLOT SERIES, growth factor time scores | `saving-data-and-plot-series-syntax.md` | Ch.13 Ex 13.14-13.16 | Done |
| Merging data sets & replicate weights | MFILE, MNAMES, IDVARIABLE, merge data sets, REPWEIGHTS, REPSE, JACKKNIFE, complex survey replicate weights | `data-merging-replicate-weights.md` | Ch.13 Ex 13.17-13.19 | Done |
| Model estimation defaults & troubleshooting | convergence problems, non-convergence, model identification, starting values, numerical integration | `model-estimation-defaults-troubleshooting.md` | Ch.14 (Model Estimation section) | Done |
| Multiple-group analysis mechanics | multiple group analysis, GROUPING, group-specific MODEL, equality constraints across groups | `multiple-group-analysis-mechanics.md` | Ch.14 (Multiple Group section) | Done |
| Missing data — general options, data missing by design, multiple cohort design | MCAR, MAR, coverage, data missing by design, multiple cohort design, DATA COHORT, accelerated cohort | `missing-data-special-designs.md` | Ch.14 (Missing Data section) | Done |
| Categorical mediators & probit/logistic-to-probability conversion | categorical mediator, MEDIATOR option, probit regression, log odds, odds ratio, multinomial logit | `categorical-mediators-probability-conversion.md` | Ch.14 (Categorical Mediators / Probability Conversion sections) | Done |
| Parameterization of models with 2+ categorical latent variables | PARAMETERIZATION=LOGIT/LOGLINEAR/PROBABILITY, TECH15, LTA calculator | `multiple-categorical-latent-variables-parameterization.md` | Ch.14 (final section) | Done |
| DATA/VARIABLE/DEFINE core syntax reference | DATA command, VARIABLE command, NAMES ARE, USEVARIABLES, MISSING IS, CLUSTER, WEIGHT, DEFINE, CENTER | `data-variable-define-core.md` | Ch.15 (full) | Done |
| DATA reshape/derivation commands (WIDETOLONG, LONGTOWIDE, TWOPART, MISSING, SURVIVAL, COHORT) | data reshaping, wide to long, long to wide, semicontinuous, dropout indicator, cohort-sequential design, abbreviation rule | `data-reshape-cohort-commands.md` | Ch.20 (gap-fill vs. Ch.15) | Done |
| ANALYSIS command core options | ANALYSIS command, TYPE=, ESTIMATOR=, ITERATIONS, CONVERGENCE, STARTS, PARAMETERIZATION, INTEGRATION | `analysis-command-options.md` | Ch.16 (full) | Done |
| MODEL command core syntax & cross-cutting notation | BY, ON, WITH, list shorthand, brackets, MODEL CONSTRAINT, MODEL INDIRECT, MODEL TEST, MODEL PRIORS | `model-command-core-syntax.md` | Ch.17 (full) | Done |
| OUTPUT/SAVEDATA/PLOT common options | TECH1-TECH16, output, standardized coefficients, STDYX, modification indices, modindices, confidence interval, cinterval, factor scores, savedata, plot | `output-savedata-plot-commands.md` | Ch.18 (full) | Done |

## Full chapter table of contents

| Ch | Title | URL | Status |
|---|---|---|---|
| 1 | Introduction | https://www.statmodel.com/HTML_UG/chapter1V8.htm | Done (see `getting-started-input-file-basics.md` above) |
| 2 | Getting started with Mplus | https://www.statmodel.com/HTML_UG/chapter2V8.htm | Done (see `getting-started-input-file-basics.md` above) |
| 3 | Regression and path analysis | https://www.statmodel.com/HTML_UG/chapter3V8.htm | Done (see above) |
| 4 | Exploratory factor analysis | https://www.statmodel.com/HTML_UG/chapter4V8.htm | Done (see above) |
| 5 | Confirmatory factor analysis and structural equation modeling (CFA/SEM) | https://www.statmodel.com/HTML_UG/chapter5V8.htm | Done (see above — all of Ex 5.1-5.33 covered) |
| 6 | Growth modeling and survival analysis | https://www.statmodel.com/HTML_UG/chapter6V8.htm | Done (see above — all of Ex 6.1-6.28 covered) |
| 7 | Mixture modeling with cross-sectional data (LCA/LPA) | https://www.statmodel.com/HTML_UG/chapter7V8.htm | Done (see above — all of Ex 7.1-7.30 covered) |
| 8 | Mixture modeling with longitudinal data (growth mixture models, LTA, etc.) | https://www.statmodel.com/HTML_UG/chapter8V8.htm | Done (see above — all of Ex 8.1-8.17 covered) |
| 9 | Multilevel modeling with complex survey data | https://www.statmodel.com/HTML_UG/chapter9V8.htm | Done (see above — all of Ex 9.1-9.40 covered) |
| 10 | Multilevel mixture modeling | https://www.statmodel.com/HTML_UG/chapter10V8.htm | Done (see above — all of Ex 10.1-10.13 covered) |
| 11 | Missing data modeling and Bayesian analysis | https://www.statmodel.com/HTML_UG/chapter11V8.htm | Done (see above — all 8 examples covered; note Ch.11 itself never covers MODEL PRIORS/BITERATIONS/PSR diagnostics — those live in Ch.16, `analysis-command-options.md`) |
| 12 | Monte Carlo simulation studies | https://www.statmodel.com/HTML_UG/chapter12V8.htm | Done (see above — all of Ex 12.1-12.12 covered) |
| 13 | Special features | https://www.statmodel.com/HTML_UG/chapter13V8.htm | Done (see above — all of Ex 13.1-13.19 covered) |
| 14 | Special modeling issues | https://www.statmodel.com/HTML_UG/chapter14V8.htm | Done (see above — all sections covered; this chapter has no numbered examples of its own) |
| 15 | TITLE, DATA, VARIABLE, DEFINE commands | https://www.statmodel.com/HTML_UG/chapter15V8.htm | Done (see above) |
| 16 | ANALYSIS command | https://www.statmodel.com/HTML_UG/chapter16V8.htm | Done (see above — core options covered in depth; ROTATION/ALIGNMENT/REPSE/BASEHAZARD and fine-grained Bayes/MCMC technical options are listed but not deeply expanded, since they're covered by procedure-specific files) |
| 17 | MODEL command | https://www.statmodel.com/HTML_UG/chapter17V8.htm | Done (see above) |
| 18 | OUTPUT, SAVEDATA, PLOT commands | https://www.statmodel.com/HTML_UG/chapter18V8.htm | Done (see above) |
| 19 | MONTECARLO command | https://www.statmodel.com/HTML_UG/chapter19V8.htm | Done (folded into `monte-carlo-simulation-power.md` with Ch.12) |
| 20 | Summary of the Mplus language | https://www.statmodel.com/HTML_UG/chapter20V8.htm | Done (mostly a condensed restatement of Ch.15-19, already covered there; the genuinely new content — DATA reshape commands and the abbreviation rule — is in `data-reshape-cohort-commands.md`) |
| — | Index | https://www.statmodel.com/HTML_UG/indexv8.htm | Index only (not a chapter; live lookup only) |
| — | User's Guide Examples (full example collection) | https://www.statmodel.com/ugexcerpts.shtml | Index only (not a chapter; live lookup only) |

## Expanding the index (for future manual revisions / edge cases)
1. Match keywords in `references/index.md` → use the matching `references/<file>.md` directly
2. If a genuinely new example or option surfaces that isn't in any matched file (e.g. a future Mplus version adds new syntax), look up the chapter URL live with `WebFetch`, use it to answer, and write it up as a new `references/<english-slug>.md` using the same 7-part structure as the other files, then add/update the matching row(s) in both tables above
3. If a procedure isn't in the table of contents at all, live lookup won't resolve it either — ask the user for the original source (PDF/link)

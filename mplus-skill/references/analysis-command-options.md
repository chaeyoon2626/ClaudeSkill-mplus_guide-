# ANALYSIS Command — Core Options Reference (TYPE, ESTIMATOR, Convergence, Starts, and More)

> Source: Mplus User's Guide v8, Chapter 16
> Bundled source: references/source-pdfs/Chapter16.pdf

## One-line summary
The `ANALYSIS:` command controls the technical details of estimation — analysis type (`TYPE=`), statistical estimator (`ESTIMATOR=`), iteration/convergence limits, random starts, parameterization, numerical integration, and parallel processing — and it is not required: if the defaults shown below fit the analysis, `ANALYSIS:` can be omitted entirely.

## Prerequisite checklist
- [ ] Know whether latent classes are involved (`TYPE=MIXTURE`) — this changes almost every other default in this file
- [ ] Know whether data are clustered/multilevel (`TYPE=TWOLEVEL`/`THREELEVEL`/`CROSSCLASSIFIED`)
- [ ] Know the outcome variable type(s) (continuous, binary/ordinal, censored, count, nominal) — this drives the `ESTIMATOR=` default and availability
- [ ] Know if the sample is a complex survey design (stratification/clustering/unequal selection probability) — needs `COMPLEX` added to `TYPE=`
- [ ] For mixture models, decide up front how much time can be spent on random starts (`STARTS=`) — under-investing here risks a local-optimum solution

## Option selection logic
| Situation | Choice |
|---|---|
| Standard SEM/CFA/regression/path/growth model, no latent classes, no clustering | `TYPE = GENERAL;` (default, can be omitted) |
| Model has random intercepts and/or random slopes but no latent classes | `TYPE = RANDOM;` (equivalently `TYPE = GENERAL RANDOM;`) |
| Complex survey data (stratification, clustering, unequal probability of selection) | add `COMPLEX` to TYPE, e.g. `TYPE = COMPLEX;` |
| Latent class / latent profile / mixture / growth mixture model | `TYPE = MIXTURE;` |
| Two-level (clustered) data | `TYPE = TWOLEVEL;` |
| Three-level data | `TYPE = THREELEVEL;` |
| Data are cross-classified (not strictly nested) | `TYPE = CROSSCLASSIFIED;` |
| Exploratory factor analysis | `TYPE = EFA # #;` (the two numbers are the lower/upper number of factors to extract) |
| Want only descriptive/sample statistics | add `BASIC` to TYPE |
| All-continuous outcomes, standard case | `ESTIMATOR = ML;` (usually the default) |
| Continuous outcomes, want robustness to non-normality and/or non-independence | `ESTIMATOR = MLR;` |
| Categorical outcomes, `TYPE=GENERAL`, want fast limited-information estimation | `ESTIMATOR = WLSMV;` (default in that cell of the table) |
| Want full Bayesian estimation | `ESTIMATOR = BAYES;` |
| Model won't converge / hits max iterations | raise `ITERATIONS=`, review `CONVERGENCE=`, inspect `TECH8`, improve starting values |
| Mixture model may have converged on a local optimum (best loglikelihood not replicated) | raise `STARTS = n m;` (e.g. `STARTS = 100 20;`, or `400 100;` for a thorough search) |
| Categorical outcomes + WLS-family estimator, need control over residual-variance hypotheses | `PARAMETERIZATION = THETA;` instead of the default `DELTA` |
| Model needs numerical integration (random slopes, categorical outcomes under ML, etc.) and is slow | adjust `INTEGRATION=` points/method, keep `ADAPTIVE=ON` (default), and/or raise `PROCESSORS=` |
| Multi-core machine available and the run is slow | `PROCESSORS = n;` (add a thread count for random-start or multi-chain Bayes models) |

## Menu path & screen fields
Categorized list of `ANALYSIS:` command options:

**1. Analysis type & estimator**
- `TYPE = GENERAL | MIXTURE | TWOLEVEL | THREELEVEL | CROSSCLASSIFIED | EFA # #;` each combinable with `BASIC`/`RANDOM`/`COMPLEX`/`MIXTURE`/`TWOLEVEL` as applicable (see Sub-option details)
- `ESTIMATOR = ML | MLM | MLMV | MLR | MLF | MUML | WLS | WLSM | WLSMV | ULS | ULSMV | GLS | BAYES;` — default depends on TYPE and outcome variable type(s)

**2. Measurement invariance / alignment**
- `MODEL = CONFIGURAL | METRIC | SCALAR;` — auto-builds multiple-group invariance models (with GROUPING or KNOWNCLASS)
- `MODEL = NOMEANSTRUCTURE | NOCOVARIANCES | ALLFREE;` — changes the defaults of the MODEL command
- `ALIGNMENT = FIXED | FREE (reference-group/model settings);` — alignment optimization for many-group factor mean/variance comparison

**3. Distribution / parameterization / link**
- `DISTRIBUTION = NORMAL (default) | SKEWNORMAL | TDISTRIBUTION | SKEWT;`
- `PARAMETERIZATION = DELTA (default) | THETA | LOGIT (default, TYPE=MIXTURE) | LOGLINEAR | PROBABILITY | RESCOVARIANCES;`
- `LINK = LOGIT (default) | PROBIT;`

**4. EFA rotation**
- `ROTATION = GEOMIN (default) | QUARTIMIN | CF-VARIMAX | CF-QUARTIMAX | CF-EQUAMAX | CF-PARSIMAX | CF-FACPARSIM | CRAWFER | OBLIMIN | VARIMAX | PROMAX | TARGET | BI-GEOMIN | BI-CF-QUARTIMAX;`
- `ROWSTANDARDIZATION = CORRELATION (default) | KAISER | COVARIANCE;`
- `PARALLEL = number;` — parallel-analysis draws for choosing the number of EFA factors

**5. Complex-survey resampling**
- `REPSE = BOOTSTRAP | JACKKNIFE | JACKKNIFE1 | JACKKNIFE2 | BRR | FAY (#);` — resampling method for replicate weights (no default)

**6. Survival-specific**
- `BASEHAZARD = ON | OFF (EQUAL | UNEQUAL);` — treat baseline hazard parameters as model vs. auxiliary parameters

**7. Optimization algorithm & numerical integration**
- `CHOLESKY = ON | OFF;`
- `ALGORITHM = EM | EMA | FS | ODLL | INTEGRATION;`
- `INTEGRATION = STANDARD (#) | GAUSSHERMITE (#) | MONTECARLO (#);`
- `MCSEED = #;`
- `ADAPTIVE = ON (default) | OFF;`
- `INFORMATION = OBSERVED | EXPECTED | COMBINATION;`

**8. Bootstrap / bootstrap LRT**
- `BOOTSTRAP = # (STANDARD | RESIDUAL);`
- `LRTBOOTSTRAP = #;`

**9. Random starts (mixture, EFA, TWOLEVEL categorical + WLS)**
- `STARTS = n m;`, `STITERATIONS=`, `STCONVERGENCE=`, `STSCALE=`, `STSEED=`, `OPTSEED=`, `K-1STARTS=`, `LRTSTARTS=`, `RSTARTS=`, `ASTARTS=`, `H1STARTS=`

**10. Difference testing / missing-data helpers**
- `DIFFTEST = filename;`
- `MULTIPLIER = filename;`
- `COVERAGE = value;` (default .10)
- `ADDFREQUENCY = value;` (default .5 / N)

**11. Iterations**
- `ITERATIONS=` (1000), `SDITERATIONS=` (20), `H1ITERATIONS=` (2000), `MITERATIONS=` (500), `MCITERATIONS=` (1), `MUITERATIONS=` (1), `RITERATIONS=` (10000), `AITERATIONS=` (5000)

**12. Convergence criteria**
- `CONVERGENCE=`, `H1CONVERGENCE=` (.0001), `LOGCRITERION=`, `RLOGCRITERION=`, `MCONVERGENCE=`, `MCCONVERGENCE=` (.000001), `MUCONVERGENCE=` (.000001), `RCONVERGENCE=` (.00001), `ACONVERGENCE=` (.001), `MIXC=`, `MIXU=`

**13. Latent class indicator technical limits**
- `LOGHIGH=` (+15), `LOGLOW=` (-15), `UCELLSIZE=` (.01), `VARIANCE=` (.0001)

**14. Alignment technical**
- `SIMPLICITY = SQRT (default) | FOURTHRT;`, `TOLERANCE=` (.01), `METRIC = REFGROUP (default) | PRODUCT;`

**15. Matrix analyzed**
- `MATRIX = COVARIANCE (default) | CORRELATION;`

**16. Bayes/MCMC & multiple imputation**
- `POINT = MEDIAN (default) | MEAN | MODE;`
- `CHAINS=` (2), `BSEED=` (0), `STVALUES = UNPERTURBED (default) | PERTURBED | ML;`, `PREDICTOR = LATENT (default) | OBSERVED;`
- `ALGORITHM = GIBBS (PX1|PX2|PX3|RW) | MH;` (Bayes context; GIBBS/PX1 is the default)
- `BCONVERGENCE=` (.05), `BITERATIONS=` (50000, min 0), `FBITERATIONS=` (no default), `THIN=` (1), `MDITERATIONS=` (10000), `KOLMOGOROV=` (100), `PRIOR=` (1000)

**17. Interactive control & parallel computing**
- `INTERACTIVE = filename;`
- `PROCESSORS = # of processors  # of threads;` (default 1 1)

## Sub-option details

### TYPE
- `GENERAL` (default): regression, path analysis, CFA, SEM, growth modeling, discrete-time and continuous-time survival, N=1 time series. Combinable with `BASIC` (sample statistics/descriptives only), `RANDOM` (random intercepts and slopes), `COMPLEX` (SEs and chi-square adjusted for stratification, clustering, unequal selection probability). Example: `TYPE = GENERAL RANDOM;` or simply `TYPE = RANDOM;` since GENERAL is the default.
- `MIXTURE`: any model with categorical latent variable(s) — LCA, LPA, mixture regression/path/SEM, latent class growth analysis, growth mixture modeling, latent transition/hidden Markov, CACE modeling, discrete/continuous-time survival mixture, loglinear modeling. Combinable with `BASIC`, `RANDOM`, `COMPLEX`.
- `TWOLEVEL`: random intercepts/slopes varying across clusters in hierarchical data. Combinable with `BASIC`, `RANDOM` (also random factor loadings and variances), `MIXTURE` (adds a categorical latent variable), `COMPLEX`.
- `THREELEVEL`: three-level clustered data; observed outcomes continuous, binary, or combinations. Combinable with `BASIC`, `RANDOM`, `COMPLEX` (continuous outcomes only).
- `CROSSCLASSIFIED`: clusters that are cross-classified rather than strictly nested; outcomes continuous, binary, ordered categorical, or combinations. Combinable with `RANDOM`.
- `EFA # #`: exploratory factor analysis; the two numbers give the lower and upper number of factors to extract, e.g. `TYPE = EFA 1 3;` produces one-, two-, and three-factor solutions. Combinable with `BASIC`, `MIXTURE`, `COMPLEX`, `TWOLEVEL` (multilevel EFA uses `TYPE = TWOLEVEL EFA lo hi UW* lo hi UB*;`, where `UW*`/`UB*` request an unrestricted within/between model be estimated; using `UW`/`UB` without the asterisk instead fixes the unrestricted model at sample-statistic values, which speeds up the analysis).

### ESTIMATOR
Default and availability depend jointly on TYPE and on whether the dependent variables are all continuous, include at least one binary/ordered-categorical variable, or include at least one censored/nominal/count variable (see the full estimator-availability table on Chapter 16 pp.666-667 for every TYPE x outcome-type combination). Meanings:
- `ML` — maximum likelihood, conventional standard errors and chi-square test statistic
- `MLM` — ML with standard errors and a mean-adjusted chi-square (the Satorra-Bentler chi-square), robust to non-normality
- `MLMV` — ML with standard errors and a mean- and variance-adjusted chi-square, robust to non-normality
- `MLR` — ML with standard errors (sandwich estimator) and a chi-square (when applicable) robust to non-normality and, with `TYPE=COMPLEX`, non-independence of observations; the MLR chi-square is asymptotically equivalent to the Yuan-Bentler T2* statistic
- `MLF` — ML with standard errors approximated by first-order derivatives and a conventional chi-square
- `MUML` — Muthen's limited-information estimator (`TYPE=TWOLEVEL`; maximum likelihood with balanced data, limited-information for unbalanced data, not available with missing data)
- `WLS` — weighted least squares, conventional SEs and chi-square using a full weight matrix (also called ADF when all outcomes are categorical)
- `WLSM` — WLS with a diagonal weight matrix; SEs and mean-adjusted chi-square use a full weight matrix
- `WLSMV` — WLS with a diagonal weight matrix; SEs and mean- and variance-adjusted chi-square use a full weight matrix
- `ULS` — unweighted least squares
- `ULSMV` — ULS with SEs and mean- and variance-adjusted chi-square using a full weight matrix
- `GLS` — generalized least squares, conventional SEs and chi-square using a normal-theory weight matrix
- `BAYES` — Bayesian posterior parameter estimates with credibility intervals and posterior predictive checking

All estimators require individual-level data except `ML` (for `TYPE=GENERAL` and `TYPE=EFA`), `GLS`, and `ULS`, which can also use summary (matrix) data.

### BAYES estimation (ESTIMATOR=BAYES)
- Available for continuous, binary, ordered categorical outcomes (or combinations) with `TYPE=GENERAL`, `TYPE=MIXTURE` (only one categorical latent variable), `TWOLEVEL`, `THREELEVEL`, `THREELEVEL RANDOM`, `CROSSCLASSIFIED`, `CROSSCLASSIFIED RANDOM`, and `EFA`.
- Uses MCMC: Gibbs sampling by default (`ALGORITHM = GIBBS (PX1|PX2|PX3|RW);`, default PX1) or Metropolis-Hastings (`ALGORITHM = MH;`, not available for `TYPE=MIXTURE`/`TWOLEVEL`). The first half of each chain is discarded as burn-in.
- Convergence assessed via the Gelman-Rubin potential scale reduction (PSR) criterion (`BCONVERGENCE=`, default .05) with `BITERATIONS = max (min);` (default 50000, 0), or via a `FBITERATIONS=` fixed iteration count if PSR is not used (then check convergence manually, e.g. via trace plots).
- Other controls: `CHAINS=` (default 2, one chain per processor when parallelized — pair with `PROCESSORS=` for speed), `BSEED=` (default 0), `THIN=` (default 1, keep every k-th draw), `POINT=` (MEDIAN default; MEAN; MODE, which uses `MDITERATIONS=`), `STVALUES=` (UNPERTURBED default; PERTURBED needs `BSEED`; ML runs ML first and uses those estimates as Bayes starting values), `PREDICTOR=` (LATENT default; OBSERVED treats a categorical mediator/exogenous predictor as continuous-observed rather than latent-response), `KOLMOGOROV=` (Kolmogorov-Smirnov cross-chain equality test draws, default 100), `PRIOR=` (draws for the prior-distribution plot, default 1000).

### PARAMETERIZATION
- `DELTA` (default when `TYPE=GENERAL`, >=1 categorical DV, WLS-family estimator): scale factors for latent response variables are free model parameters; residual variances are not.
- `THETA`: residual variances for latent response variables are free model parameters instead; needed when hypotheses about residual variances are of interest (e.g., multiple-group analysis, longitudinal data), and required for certain models where a categorical DV both influences and is influenced by another observed or latent variable (DELTA can impose improper constraints there).
- `LOGIT` (default when `TYPE=MIXTURE` with more than one categorical latent variable): categorical-latent-variable relationships specified as logistic regressions via ON/WITH.
- `LOGLINEAR`: loglinear model for the categorical latent variables, allowing two- and three-way interactions; only WITH (not ON) can be used.
- `PROBABILITY`: categorical-latent-variable regression coefficients expressed as probabilities rather than logits.
- `RESCOVARIANCES` (alias `RESCOV`): used with LCA/LTA under maximum likelihood to allow residual covariances for binary/ordered-categorical outcomes, specified with the WITH option; can be free across classes, held equal, or included in only certain classes.

### STARTS and related random-start options (critical for mixture models)
- `STARTS = n m;` — n = random start sets generated in the initial stage, m = number of those carried to final-stage optimization. Default for `TYPE=MIXTURE` is `20 4`. `STARTS = 0;` turns random starts off entirely. Example: `STARTS = 100 20;` (100 initial sets, 20 to final stage); for a more thorough search, `STARTS = 400 100;` or `STARTS = 1000 250;`. For `TYPE=EFA`, `TYPE=GENERAL`, and `TYPE=TWOLEVEL` with the WLS/WLSM/WLSMV/ULSMV estimators, the default is `STARTS = 10;` (10 initial sets, 10 final optimizations).
- `STITERATIONS=` — max iterations allowed in the initial stage (default 10; e.g. `STITERATIONS = 20;` for a more thorough search).
- `STCONVERGENCE=` — derivative convergence criterion for the initial-stage optimization (default 1).
- `STSCALE=` — scale of the random perturbation applied to starting values (default 5, a medium level).
- `STSEED=` — random seed used to generate the random starts (default 0).
- `OPTSEED=` — reuse the seed found in a previous run to give the highest loglikelihood; using it turns off random starts.
- `K-1STARTS = n m;` — starts for the k-1 class model used by `TECH11`; default 20 initial / 4 final optimizations when `OPTSEED` is not used (otherwise same as `STARTS`).
- `LRTSTARTS = n0 m0 n1 m1;` — starts for the `TECH14` bootstrap likelihood ratio test, given separately for the k-1 class model (n0 m0) and the k class model (n1 m1) fit to bootstrap-drawn data; default `0 0 40 8`.
- `RSTARTS = n m;` — random starts for the EFA rotation (GPA) algorithm: n = rotation starts (default 30), m = number of best rotated solutions printed.
- `ASTARTS=` — random starts for the alignment optimization (default 30).
- `H1STARTS = n m;` — random starts for the unrestricted H1 model used with `TYPE=GENERAL` plus the `DISTRIBUTION` option; default `0 0` (the H1 model typically needs several starts when requested).

### ITERATIONS and CONVERGENCE families
- `ITERATIONS=` — max Quasi-Newton iterations for continuous outcomes (default 1000).
- `SDITERATIONS=` — max steepest-descent iterations within the Quasi-Newton algorithm (default 20).
- `H1ITERATIONS=` — max EM iterations for the unrestricted H1 model with missing data (default 2000).
- `MITERATIONS=` — max EM algorithm iterations (default 500).
- `MCITERATIONS=` / `MUITERATIONS=` — M-step iterations for categorical latent variables / for censored, categorical, and count outcomes (default 1 each); `MIXC=`/`MIXU=` choose whether the M step of the EM algorithm stops based on `ITERATIONS` or `CONVERGENCE` for each case.
- `RITERATIONS=` — max iterations of the EFA rotation (GPA) algorithm (default 10000).
- `AITERATIONS=` — max iterations for the alignment optimization (default 5000).
- `CONVERGENCE=` — Quasi-Newton derivative convergence criterion for continuous outcomes; default .000001 for `TYPE=TWOLEVEL`/`MIXTURE`/`RANDOM` and `ALGORITHM=INTEGRATION`, .00005 for all other models.
- `H1CONVERGENCE=` — EM convergence criterion for the unrestricted H1 model with missing data (default .0001).
- `LOGCRITERION=` / `RLOGCRITERION=` — absolute / relative observed-data loglikelihood-change convergence criteria for the EM algorithm; defaults vary by type (e.g. `.001`/`.000001` for TWOLEVEL/RANDOM/ALGORITHM=INTEGRATION, `.0001` for MIXTURE with PARAMETERIZATION=PROBABILITY, else `.0000001`).
- `MCONVERGENCE=` — observed-data loglikelihood derivative convergence for the EM algorithm (defaults similarly vary by type).
- `MCCONVERGENCE=` / `MUCONVERGENCE=` — complete-data loglikelihood derivative convergence for the M step, for categorical latent variables / for censored, categorical, and count outcomes (default .000001 each).
- `RCONVERGENCE=` — convergence criterion for the EFA rotation algorithm (default .00001).
- `ACONVERGENCE=` — convergence criterion for the derivatives of the alignment optimization (default .001).

### INTEGRATION and related numerical-integration options
- `INTEGRATION = STANDARD (#) | GAUSSHERMITE (#) | MONTECARLO (#);` used when `ALGORITHM=INTEGRATION`. `STANDARD` (default) uses rectangular (trapezoid) numerical integration; default points per dimension is 7 for `TYPE=EFA` and `TYPE=TWOLEVEL` with weighted least squares, and 15 for all other analyses. `GAUSSHERMITE` also defaults to 15 points per dimension. `MONTECARLO` uses randomly generated integration points; the default number of points varies by analysis type, most commonly 500. Example: `INTEGRATION = 10;` or `INTEGRATION = STANDARD (10);`.
- `MCSEED=` — random seed for Monte Carlo integration (default 0).
- `ADAPTIVE = ON (default) | OFF;` — customizes the numerical integration points per observation during computation.
- `ALGORITHM = EM | EMA | FS | ODLL | INTEGRATION;` — optimization method. `EM` optimizes the complete-data loglikelihood via the EM algorithm; `EMA` is an accelerated EM procedure using Quasi-Newton/Fisher Scoring steps when needed; `FS` is Fisher Scoring; `ODLL` optimizes the observed-data loglikelihood directly; `INTEGRATION` indicates numerical integration is required and can be combined with an optimization setting, e.g. `ALGORITHM = INTEGRATION EM;`.
- `CHOLESKY = ON | OFF;` — used with `ALGORITHM=INTEGRATION` to decompose the latent-variable and residual covariance matrices into orthogonal components, improving optimization. Default is ON when all dependent variables are censored, categorical, and/or count (except categorical DVs with `LINK=PROBIT`); OFF otherwise.
- `INFORMATION = OBSERVED | EXPECTED | COMBINATION;` — which information matrix estimator (for ML/MLR) is used to compute standard errors. `OBSERVED` is used by default under missing-data theory; `EXPECTED` is also available for all-continuous models estimated without numerical integration; `COMBINATION` is also available for other outcome types / models needing numerical integration.

### PROCESSORS
- `PROCESSORS = # of processors  # of threads;` — default is 1 processor, 1 thread. Enables parallel computing to speed up estimation; available for `TYPE=MIXTURE`; Bayesian analysis with more than one chain (unless `STVALUES=ML`); models requiring numerical integration; all-continuous ML models with missing data; and `TYPE=TWOLEVEL` categorical outcomes with `ESTIMATOR=WLSMV`.
- With multiple Bayes chains, one chain runs per processor — pair `CHAINS=` with `PROCESSORS=` to realize the speedup, e.g. `CHAINS = 4; PROCESSORS = 4;`.
- With random starts, give two numbers: `PROCESSORS = 8 4;` means 8 processors distributed across 4 threads. The number of threads actually used is the smaller of the requested thread count and the number of final-stage optimizations in `STARTS=`. If only one number is given, the thread count equals the processor count.
- Prefer fewer threads than processors for large, memory-heavy models, because memory usage scales with the number of threads and can exceed available memory (making computation slower or impossible) if there are too many.

## Post-run operations
- If the output is missing "MODEL ESTIMATION TERMINATED NORMALLY," or the best loglikelihood is not replicated across starting-value sets (mixture models), raise `STARTS=` and re-run; check `TECH8` for the iteration history and any interim messages.
- For a correct chi-square difference test under `MLMV`/`WLSMV` estimation, first fit the less restrictive H1 model with `SAVEDATA: DIFFTEST = deriv.dat;`, then fit the H0 model with `ANALYSIS: DIFFTEST = deriv.dat;` to obtain the proper difference test (a raw MLMV/WLSMV chi-square difference is not itself chi-square distributed).
- For heavy/slow runs, consider `OUTPUT: SVALUES;` to save current parameter estimates as starting-value syntax for a follow-up run (see output-savedata-plot-commands.md), and/or set `OPTSEED=` to the seed that produced the best loglikelihood so reruns skip random starts.
- For Bayes runs, check convergence via the Gelman-Rubin PSR (`BCONVERGENCE=`), posterior trace plots, and the cross-chain Kolmogorov-Smirnov test (`KOLMOGOROV=`) before trusting posterior summaries; use posterior predictive checking (PPC) for overall fit.

## Likely FAQ mapping
- "My model won't converge" -> check `TECH8` for where estimation stalls; raise `ITERATIONS=`; for mixture models raise `STARTS=`; try better/perturbed starting values (`STVALUES`); confirm the `TYPE=`/`ESTIMATOR=` combination is actually identified for the model.
- "My mixture/LPA/LCA solution might be a local optimum instead of the global best" -> raise `STARTS = n m;` (e.g. `100 20` -> `400 100` -> `1000 250`) and confirm the best loglikelihood is replicated across starts.
- "How do I speed up a slow run?" -> set `PROCESSORS = n;` (add a thread count for random-start or multi-chain Bayes models, `PROCESSORS = n threads;`); reduce `INTEGRATION=` points if numerical integration is used (keep `ADAPTIVE=ON`); consider `ALGORITHM=EMA`; trim `STARTS=` final-stage optimizations once a stable solution is confirmed.
- "Which estimator should I use for my data?" -> continuous outcomes -> `ML` (or `MLR` for robustness to non-normality/non-independence); categorical outcomes with `TYPE=GENERAL` -> `WLSMV` (default) or `ML`/`MLR` with numerical integration; want a Bayesian analysis -> `ESTIMATOR = BAYES;`.
- "I have clustered/multilevel data" -> `TYPE = TWOLEVEL;` (or `THREELEVEL` / `CROSSCLASSIFIED`); add `COMPLEX` instead when it is a complex-survey design (stratification/weights) rather than a substantive multilevel model.
- "I have latent classes/profiles/mixture components" -> `TYPE = MIXTURE;`, and tune `STARTS=` for reliable convergence.
- "How many integration points do I need, and can I change them?" -> `INTEGRATION = STANDARD (#);` or `GAUSSHERMITE (#);`; more points is more accurate but slower; `ADAPTIVE=ON` (default) already customizes points per observation to reduce the burden.
- "How do I get a correct chi-square difference test with WLSMV/MLMV?" -> `SAVEDATA: DIFFTEST = file;` on the H1 run, then `ANALYSIS: DIFFTEST = file;` on the H0 run.
- "Should I use DELTA or THETA parameterization for categorical outcomes?" -> default is `DELTA`; switch to `PARAMETERIZATION = THETA;` when residual-variance hypotheses matter (multi-group or longitudinal models) or when a categorical DV both influences and is influenced by another variable.
- "How many processors/threads should I request for a Bayes run with multiple chains?" -> match `PROCESSORS=` to `CHAINS=` (one chain per processor) for full parallel speedup.

# Experiment Patterns

Common experiment types the auto-academic pipeline may need to implement. The experiment-planner agent uses these patterns to generate experiment specs from hypotheses.

## Pattern 1: Survey/Questionnaire Analysis

**When:** Hypothesis involves human perceptions, attitudes, behavioral intentions
**Typical stack:** Python + pandas + scipy/statsmodels
**Steps:** Generate/load survey data -> validate instrument (Cronbach's alpha, CFA) -> descriptive statistics -> inferential tests (t-test, ANOVA, regression)
**Success metrics:** p-values, effect sizes, model fit indices (CFI, RMSEA, SRMR)
**Key output:** Statistical tables, correlation matrices, path diagrams

### Program.md Example

**Scope — Tier 1 Tunable Parameters:**
- `significance_threshold`: p-value cutoff for hypothesis rejection (default: 0.05, range: 0.01–0.10)
- `control_variables`: list of covariates to include in regression (default: `["age", "gender"]`)
- `regression_model_type`: OLS, logistic, or ordinal (default: `"OLS"`)

**Scope — Tier 2 Code Change Boundaries:**
- May swap between t-test and Mann-Whitney U for non-normal data
- May add or remove mediator/moderator terms in regression model

**Evaluation — BETTER if:**
- Effect size (Cohen's d or η²) is larger while p < significance_threshold
- Model fit improved (CFI > 0.95, RMSEA < 0.06, SRMR < 0.08)
- Regression R² increased without overfitting (AIC/BIC decreased)

**Evaluation — WORSE if:**
- p-value crosses significance_threshold in wrong direction
- Model fit indices degrade
- Instrument reliability drops (Cronbach's alpha < 0.70)

## Pattern 2: Text/Content Analysis

**When:** Hypothesis involves patterns in textual data (policies, curricula, social media)
**Typical stack:** Python + nltk/spacy + scikit-learn
**Steps:** Collect/load corpus -> preprocess -> code/classify -> inter-rater reliability -> frequency/thematic analysis
**Success metrics:** Cohen's kappa (inter-rater), theme saturation, classification accuracy
**Key output:** Codebook, frequency tables, theme maps

### Program.md Example

**Scope — Tier 1 Tunable Parameters:**
- `n_topics`: number of topics for LDA/NMF topic modeling (default: 10, range: 5–30)
- `min_df`: minimum document frequency for vocabulary (default: 2, range: 1–5)
- `max_features`: maximum vocabulary size (default: 5000, range: 1000–20000)
- `classification_threshold`: probability cutoff for binary classification (default: 0.5, range: 0.3–0.7)

**Scope — Tier 2 Code Change Boundaries:**
- May swap between LDA and NMF for topic modeling
- May replace rule-based classifier with trained scikit-learn model
- May change tokenization strategy (word-level vs. n-gram)

**Evaluation — BETTER if:**
- Inter-rater reliability higher (Cohen's kappa > 0.80 is strong)
- Classification accuracy or F1 score improved on held-out test set
- Topic coherence score (NPMI) increased

**Evaluation — WORSE if:**
- Cohen's kappa drops below 0.60 (unacceptable agreement)
- Classification accuracy drops or precision/recall trade-off worsens
- Topic model produces incoherent or redundant topics

## Pattern 3: Machine Learning Classification/Regression

**When:** Hypothesis involves predictive modeling or pattern recognition
**Typical stack:** Python + scikit-learn/pytorch + pandas
**Steps:** Load data -> train/val/test split -> train model -> evaluate -> ablation studies -> compare baselines
**Success metrics:** Accuracy, F1, AUC-ROC, MSE, R-squared (with confidence intervals)
**Key output:** Performance tables, confusion matrices, learning curves, ablation results

### Program.md Example

**Scope — Tier 1 Tunable Parameters:**
- `learning_rate`: optimizer step size (default: 0.001, range: 0.0001–0.01)
- `batch_size`: training batch size (default: 32, range: 16–256)
- `hidden_dims`: list of hidden layer sizes (default: `[128, 64]`)
- `dropout`: dropout probability for regularization (default: 0.2, range: 0.0–0.5)
- `epochs`: maximum training epochs (default: 50, range: 10–200)

**Scope — Tier 2 Code Change Boundaries:**
- May swap model architecture (e.g., MLP to LSTM, random forest to gradient boosting)
- May change loss function (e.g., cross-entropy to focal loss)
- May add/remove regularization (L1/L2, dropout layers)

**Evaluation — BETTER if:**
- Test accuracy or F1 score improved (classification) / MSE/R² improved (regression)
- Train-test performance gap < 5% (not overfitting)
- Inference time does not increase disproportionately

**Evaluation — WORSE if:**
- Test metric degrades
- Train-test gap exceeds 10% (overfitting)
- Model fails to converge (loss does not decrease after 10 epochs)

## Pattern 4: Simulation/Agent-Based Modeling

**When:** Hypothesis involves system dynamics, emergent behavior, policy scenarios
**Typical stack:** Python + mesa/netlogo + numpy
**Steps:** Define model parameters -> implement agents/rules -> run simulation (N iterations) -> sensitivity analysis -> compare scenarios
**Success metrics:** Convergence of simulation outcomes, sensitivity coefficients, scenario comparisons
**Key output:** Time-series plots, parameter sweep heatmaps, scenario comparison tables

### Program.md Example

**Scope — Tier 1 Tunable Parameters:**
- `n_agents`: number of agents in the simulation (default: 100, range: 50–1000)
- `interaction_probability`: probability of agent-agent interaction per step (default: 0.1, range: 0.01–0.5)
- `learning_rate`: agent belief/behavior update rate (default: 0.05, range: 0.01–0.2)
- `n_iterations`: number of simulation time steps (default: 500, range: 100–2000)

**Scope — Tier 2 Code Change Boundaries:**
- May change agent decision rule (rational vs. bounded rationality)
- May add/remove network topology (random, small-world, scale-free)
- May swap between synchronous and asynchronous agent updating

**Evaluation — BETTER if:**
- Outcome distribution (e.g., adoption rate, equilibrium state) is closer to observed empirical data
- Sensitivity analysis shows stable results across parameter ranges (low coefficient of variation)
- Scenario comparisons show statistically distinguishable outcomes

**Evaluation — WORSE if:**
- Simulation fails to converge (outcomes still changing after n_iterations)
- Sensitivity analysis shows extreme dependence on a single parameter
- Outcomes diverge further from empirical benchmarks

## Pattern 5: Statistical Comparison/Meta-Analysis

**When:** Hypothesis involves comparing groups, treatments, or synthesizing existing effect sizes
**Typical stack:** Python/R + scipy/metafor
**Steps:** Collect effect sizes -> heterogeneity tests (Q, I-squared) -> random/fixed effects model -> forest plot -> publication bias tests
**Success metrics:** Pooled effect size + CI, I-squared, funnel plot symmetry
**Key output:** Forest plots, funnel plots, summary tables

### Program.md Example

**Scope — Tier 1 Tunable Parameters:**
- `effect_size_measure`: metric to synthesize (default: `"cohen_d"`, options: `"hedge_g"`, `"log_or"`, `"r"`)
- `model_type`: random or fixed effects (default: `"random"`)
- `moderators`: list of study-level covariates for meta-regression (default: `[]`)

**Scope — Tier 2 Code Change Boundaries:**
- May add moderator variables to meta-regression model
- May switch between DerSimonian-Laird and REML estimators for τ²
- May add trim-and-fill procedure for publication bias correction

**Evaluation — BETTER if:**
- Heterogeneity reduced (I² lower, Q test non-significant)
- Pooled effect size more precise (narrower confidence interval)
- Meta-regression explains significant variance (R² > 0, p < 0.05)

**Evaluation — WORSE if:**
- I² increases substantially (> 75% is high heterogeneity)
- Confidence interval widens
- Egger's test indicates significant publication bias without correction

## Pattern 6: Network/Graph Analysis

**When:** Hypothesis involves relational structures (citation networks, collaboration patterns, knowledge graphs)
**Typical stack:** Python + networkx/igraph + matplotlib
**Steps:** Build graph from data -> compute centrality metrics -> community detection -> visualize -> compare networks
**Success metrics:** Centrality distributions, modularity, clustering coefficients
**Key output:** Network visualizations, centrality tables, community maps

### Program.md Example

**Scope — Tier 1 Tunable Parameters:**
- `edge_threshold`: minimum weight for including an edge (default: 1, range: 0–10)
- `community_algorithm`: algorithm for community detection (default: `"louvain"`, options: `"girvan_newman"`, `"label_propagation"`)
- `centrality_measure`: primary centrality metric (default: `"betweenness"`, options: `"degree"`, `"eigenvector"`, `"pagerank"`)

**Scope — Tier 2 Code Change Boundaries:**
- May add multi-layer network analysis
- May switch between directed and undirected graph representations
- May add temporal slicing for longitudinal network comparison

**Evaluation — BETTER if:**
- Modularity higher (stronger community structure, Q > 0.3 is meaningful)
- Community structure more interpretable (fewer singleton communities)
- Centrality distribution fits expected scale-free or small-world properties

**Evaluation — WORSE if:**
- Modularity decreases
- Number of isolated nodes increases substantially
- Community assignments are unstable across algorithm runs (low NMI between runs)

## Pattern 7: Natural Experiment / Quasi-Experimental Design

**When:** Hypothesis involves causal claims without randomized control
**Typical stack:** Python/R + statsmodels/causalimpact
**Steps:** Define treatment/control -> propensity score matching or diff-in-diff -> estimate treatment effect -> robustness checks
**Success metrics:** ATT/ATE + CI, parallel trends test, placebo tests
**Key output:** Treatment effect estimates, parallel trends plots, robustness tables

### Program.md Example

**Scope — Tier 1 Tunable Parameters:**
- `matching_method`: propensity score matching strategy (default: `"nearest_neighbor"`, options: `"kernel"`, `"radius"`)
- `caliper`: maximum propensity score distance for matching (default: 0.05, range: 0.01–0.2)
- `bandwidth`: bandwidth for kernel matching or RDD (default: 0.1, range: 0.05–0.5)
- `n_bootstrap`: bootstrap replications for standard errors (default: 500, range: 200–2000)

**Scope — Tier 2 Code Change Boundaries:**
- May switch between propensity score matching and difference-in-differences
- May add regression discontinuity design if running variable is available
- May add instrumental variable approach if valid instrument exists

**Evaluation — BETTER if:**
- Covariate balance improved after matching (standardized mean differences < 0.1)
- Treatment effect estimate more precise (narrower CI) without bias increase
- Placebo tests non-significant (treatment effect in pre-period near zero)

**Evaluation — WORSE if:**
- Covariate imbalance worsens (standardized mean differences increase)
- Parallel trends assumption violated (pre-treatment trends diverge)
- Treatment effect estimate changes substantially across robustness checks (sensitivity)

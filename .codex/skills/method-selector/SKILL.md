---
name: method-selector
description: Compare 2-4 candidate modeling schemes for each subproblem and recommend one execution route based on task type, data, interpretability, literature analysis, and contest constraints.
license: MIT
---

# Purpose

Generate a candidate method pool for each classified subquestion in a mathematical modeling contest.

This skill converts validated problem parse, problem classification, and related-paper analysis into a structured method candidate pool. For each subquestion, it proposes 2-4 meaningfully different candidate modeling schemes, compares them on feasibility, interpretability, data fit, and implementation risk, and recommends a first-round execution priority. It specifies what each method expects as input, what it should output, and how its results should be evaluated.

This skill focuses on generating the candidate pool — the actual method selection happens AFTER multi-round experiments, when the modeler reviews experiment reports and locks in the final method.

This skill does not generate code, clean data, run experiments, create figures, or write paper sections.

# When to use

Use this skill:

- After `problem-parser`, `problem-classifier`, and `related-paper-analyzer` have produced validated artifacts.
- Before data cleaning, code generation, robustness checks, figure planning, or paper writing.
- When the modeler needs a structured set of candidate methods to consider for each subquestion.
- When the user says: "what methods should we try for Q1?", "generate candidate methods", "build the method pool for Q2".
- When the team needs to see multiple modeling approaches before committing to one.

# Preconditions

The following should already exist or be provided:

- A validated problem parse.
- A validated problem classification artifact.
- A related paper analysis artifact at `workspace/papers/related_paper_analysis.md` or an explicit decision to proceed without one.
- Subquestion IDs and dependencies.
- Required outputs for each subquestion.
- Data inventory and known data limitations.
- Contest constraints such as time limit, allowed tools, and submission expectations if available.

If the problem parse or classification is missing, hand back to `workflow-orchestrator`, `problem-parser`, or `problem-classifier`.

# Inputs

Use or request:

- `workspace/problem/problem-parser/problem_parse.json`, if available.
- `workspace/problem/problem-classifier/problem_classification.json`, if available.
- `workspace/papers/related_paper_analysis.md`, if available.
- Parsed subquestions, required outputs, constraints, and dependencies.
- Primary and secondary problem types from `problem-classifier`.
- Transferable ideas, cautions, and comparison notes from `workspace/papers/related_paper_analysis.md`.
- Data inventory, missing data list, units, and known data risks.
- User constraints, such as preferred implementation language, team skill level, contest deadline, or required interpretability.

# Workflow

1. Read the parsed problem, classification, and literature analysis artifacts.
   - Work subquestion by subquestion.
   - Respect dependencies between subquestions.
   - Do not collapse different subquestions into one generic method pool.

2. For each subquestion, identify the method requirements from:
   - Required output form (ranking, prediction, plan, mechanism explanation, etc.).
   - Available data and missing data.
   - Interpretability requirement.
   - Computational complexity and implementation time.
   - Literature cues (what worked in similar problems).
   - Validation and evaluation criteria.

3. Generate 2-4 candidate schemes per subquestion.
   - Each scheme must be meaningfully different — not cosmetic variants of the same idea.
   - Each scheme must have a clear mathematical idea (what it does, not just a name).
   - Each scheme must specify:
     - **Math idea**: The core mathematical approach in 2-3 sentences.
     - **Strengths**: Why this method fits this subquestion.
     - **Weaknesses**: Known limitations, risks, or assumptions.
     - **Expected output from programmer**: What result files, figures, and metrics the programmer should produce.
     - **Evaluation metric**: How to judge whether this method's output is good.
     - **Implementation difficulty**: easy / medium / hard.
     - **First-round priority**: high (try first) / medium (try if time) / low (backup).

4. Reject unsuitable methods explicitly.
   - For each subquestion, identify methods that look tempting but are unsuitable.
   - Give data-driven, time-driven, or interpretability-driven reasons for rejection.
   - This prevents the team from wasting time on obviously poor choices.

5. Define baseline requirement.
   - Every subquestion must have at least one baseline method in its candidate pool.
   - The baseline should be the simplest meaningful reference method.
   - State clearly which candidate is the baseline.

6. Recommend first-round execution strategy.
   - State which 1-2 methods should be implemented and tested first.
   - State under what conditions the team should try the other candidates.
   - State what the modeler should look for in round 1 experiment reports.

7. Define expected artifacts per method.
   - Required input data.
   - Expected output files (results, figures, metrics).
   - Expected validation checks.
   - How to compare this method against others.

8. Produce the candidate method pool.
   - Save per-subquestion files under `methods/Qx/`.
   - Each file is `methods/Qx/qx_method_candidates.md`.
   - Also produce a global pool summary at `methods/method_pool_summary.md` (optional, useful for overview).

# Outputs

Produce per-subquestion candidate method pool files:

- `methods/Q1/q1_method_candidates.md`
- `methods/Q2/q2_method_candidates.md`
- `methods/Q3/q3_method_candidates.md`
- `methods/Q4/q4_method_candidates.md` (if needed)
- `methods/method_pool_summary.md` (optional global overview)

Each file must include:

- Subquestion goal (from problem parse).
- Problem type (from classifier).
- Required output.
- Candidate method pool (2-4 methods).
- Rejected methods and reasons.
- Baseline designation.
- First-round execution recommendation.
- Data requirements.
- Evaluation criteria per method.

# Output format

Each `methods/Qx/qx_method_candidates.md` must follow this structure:

```markdown
# Qx Method Candidate Pool

## 1. Subquestion Summary
- **Goal**: [from problem parse]
- **Problem type**: [from classifier]
- **Required output**: [what must be delivered]
- **Input data available**: [data inventory relevant to this subquestion]
- **Dependencies**: [which subquestions this one depends on]
- **Constraints**: [key constraints affecting method choice]

## 2. Candidate Method Pool

### Candidate M1: [Short Descriptive Name]

- **Math idea**: [2-3 sentences explaining the core mathematical approach]
- **Why it fits this subquestion**: [1-2 sentences]
- **Strengths**:
  - [strength 1]
  - [strength 2]
- **Weaknesses**:
  - [weakness 1]
  - [weakness 2]
- **Expected programmer outputs**:
  - Result files: [list expected output files]
  - Figures: [list expected figures]
  - Metrics: [list expected metrics]
- **Evaluation criteria**: [how to judge if this method worked well]
- **Implementation difficulty**: easy / medium / hard
- **First-round priority**: high / medium / low
- **Data requirements**: [specific data fields needed]
- **Literature support**: [reference to paper analysis if applicable, or "no direct literature support"]
- **Estimated implementation time**: [rough estimate: hours]

### Candidate M2: [Short Descriptive Name]
[Same structure]

### Candidate M3: [Short Descriptive Name]
[Same structure]

### Candidate M4: [Short Descriptive Name] (if applicable)
[Same structure]

## 3. Rejected Methods

| Method | Reason for Rejection |
|--------|---------------------|
| [method name] | [specific data-driven, time-driven, or interpretability-driven reason] |

## 4. Baseline Designation

- **Baseline**: [which candidate serves as baseline]
- **Why this baseline**: [reason — simplest, most transparent, easiest to implement]

## 5. First-Round Execution Recommendation

- **Implement first**: [1-2 method names]
- **Reason**: [why these should be tried first]
- **When to try others**: [conditions under which remaining candidates should be tested]
- **What to look for in round 1**: [what the modeler should check in the experiment report]

## 6. Cross-Method Comparison Matrix

| Criterion | M1 | M2 | M3 | M4 |
|-----------|---|---|---|---|
| Interpretability | [rating] | [rating] | [rating] | [rating] |
| Data requirement satisfaction | [rating] | [rating] | [rating] | [rating] |
| Implementation complexity | [rating] | [rating] | [rating] | [rating] |
| Literature support | [rating] | [rating] | [rating] | [rating] |
| Expected accuracy/fit | [rating] | [rating] | [rating] | [rating] |
| Robustness potential | [rating] | [rating] | [rating] | [rating] |

## 7. Handoff

- **Next skill**: `model-code-analyzer`
- **Required inputs for next skill**:
  - This candidate pool document
  - Cleaned data (after `data-auditor-cleaner`)
  - Implementation target (python / matlab)
```

Also produce a `methods/method_pool_summary.md` if multiple subquestions are being planned:

```markdown
# Method Pool Summary (All Subquestions)

| Subquestion | Type | # Candidates | Baseline | First-Round Priority | Rejected |
|-------------|------|-------------|----------|---------------------|----------|
| Q1 | evaluation | 3 | M1 (equal-weight) | M2, M1 | Deep learning (data insufficient) |
| Q2 | prediction | 3 | M1 (moving average) | M2, M1 | LSTM (time series too short) |
| Q3 | optimization | 2 | M1 (greedy) | M1, M2 | Genetic algorithm (overkill) |
```

# Method family guide

Use this section to select candidate method families. These are guidelines, not mandatory choices.

## Evaluation

Candidate options:
- equal-weight normalized scoring (always consider as baseline)
- entropy-weight TOPSIS
- AHP + TOPSIS (if subjective weights are justifiable)
- PCA-assisted comprehensive evaluation (if indicators are highly correlated)
- grey relational analysis
- fuzzy comprehensive evaluation
- RSR (rank sum ratio) evaluation

Use when: output is score, ranking, grade, risk level, priority, or comparison.

Check: indicator justification, positive/negative direction, normalization, weight source, ranking stability.

Avoid: arbitrary weights with no explanation, black-box scoring without labeled data, repeated indicators double-counting.

## Prediction

Candidate options:
- mean or last-value baseline (always consider)
- linear regression (always consider as simple baseline)
- ARIMA / SARIMA (for time series with clear temporal structure)
- exponential smoothing
- grey prediction GM(1,1) (for small-sample monotonic sequences)
- random forest regression (for tabular nonlinear data)
- gradient boosting (XGBoost/LightGBM, if justified)
- ensemble (if multiple models are justified)

Use when: output is future value, trend, forecast interval, missing value estimate, or uncertainty estimate.

Check: train-test split or validation scheme, error metrics (MAE, RMSE, MAPE), residuals, extrapolation limits, feature leakage.

Avoid: high-capacity models with tiny data, prediction without error metrics, claiming future accuracy from training fit alone, deep learning when data < 1000 samples.

## Optimization

Candidate options:
- greedy heuristic (always consider as baseline)
- linear programming (for linear objectives and constraints)
- integer programming (for discrete decisions)
- nonlinear programming (for nonlinear objectives or constraints)
- dynamic programming (for sequential decisions)
- multi-objective optimization (for trade-off problems)
- network flow / shortest path (for graph-based allocation)
- simulated annealing or genetic algorithm (for hard combinatorial cases, use only if exact methods are infeasible)

Use when: output is a decision plan, allocation, route, schedule, assignment, maximum, minimum, or feasible strategy.

Check: decision variables, state variables, parameters, objective function, constraints, feasibility, implementable final plan, sensitivity to weights or constraints.

Avoid: objective functions with vague meaning, missing constraints, final result that is only an optimal value without a plan, heuristics without baseline comparison.

## Mechanism

Candidate options:
- simplified algebraic relationship (baseline)
- differential equations (ODE/PDE)
- difference equations (discrete time)
- compartment models (SIR, SEIR, etc.)
- conservation laws
- system dynamics
- mechanism + parameter fitting (hybrid)

Use when: output must explain a process, dynamic mechanism, physical relation, biological spread, flow, motion, growth, or decay.

Check: assumptions, units, parameter sources, validation against data or common sense, boundary conditions.

Avoid: equations with undefined variables, parameters with no source, mechanism claims unsupported by validation.

## Classification / Clustering

Candidate options:
- rule-based classification (baseline)
- k-means (baseline clustering)
- decision tree
- logistic regression
- random forest
- SVM
- clustering with validation index (silhouette, Davies-Bouldin)
- anomaly detection (isolation forest, LOF)

Use when: output is class label, cluster, segment, group, or anomaly.

Check: feature scaling, class balance, validation metric, cluster number selection, interpretability.

Avoid: arbitrary cluster counts, classification without labels, accuracy-only reporting under class imbalance.

## Graph / Routing

Candidate options:
- direct distance rule (baseline)
- greedy nearest-neighbor (baseline)
- Dijkstra shortest path
- A* search
- minimum spanning tree (Prim/Kruskal)
- max flow / min cut
- Hungarian algorithm (matching)
- vehicle routing heuristics (Clarke-Wright, sweep)
- TSP/VRP exact or heuristic

Use when: problem involves nodes, edges, paths, networks, flows, routes, matching, or connectivity.

Check: graph abstraction, edge weights, node definitions, feasibility constraints, static vs dynamic assumptions.

Avoid: graph models that ignore real constraints, unjustified edge weights, routes that are mathematically valid but practically infeasible.

## Simulation

Candidate options:
- deterministic scenario comparison (baseline)
- Monte Carlo simulation
- discrete-event simulation
- agent-based simulation
- stochastic process modeling
- scenario analysis

Use when: output is a stochastic distribution, scenario result, repeated process outcome, or policy comparison under uncertainty.

Check: random seed, number of trials, parameter distributions, confidence intervals or summary statistics, scenario assumptions.

Avoid: simulation without statistical summary, hidden random assumptions, too few trials, unverifiable scenario generation.

## Data Analysis

Candidate options:
- descriptive statistics (baseline)
- correlation analysis
- hypothesis testing
- factor analysis
- principal component analysis
- regression explanation
- exploratory visualization

Use when: output is pattern discovery, factor identification, descriptive insight, or relationship analysis.

Check: correlation vs causation, data quality, unit consistency, figure relevance, connection to later modeling steps.

Avoid: decorative figures without modeling purpose, causal claims from correlation alone, analysis not feeding any subquestion.

# Rules

- Generate 2-4 candidate methods per subquestion. If only 1 is feasible, explain why and flag the risk.
- Each candidate must have a distinct mathematical idea — not just a different library or parameter.
- Every subquestion must have a baseline candidate designated.
- Every candidate must specify what the programmer should output and how to evaluate it.
- Unsuitable methods must be explicitly rejected with reasons.
- First-round priority must be stated — the team should know what to implement first.
- Do not select the final method. This is the candidate pool, not the final choice.
- Final method selection happens AFTER experiments, by the modeler reviewing experiment reports.
- Do not generate code.
- Do not clean data.
- Do not write paper text.
- Do not fabricate data, metrics, or results.
- Mark high-risk methods as high-risk.
- Preserve uncertainty inherited from previous stages.

# Verification

Before handing off, verify:

- Every subquestion has 2-4 candidate methods or an explicit justification for fewer.
- Every subquestion has a baseline designated.
- Every candidate has: math idea, strengths, weaknesses, expected outputs, evaluation criteria, difficulty, priority.
- Unsuitable methods are explicitly listed and rejected with reasons.
- First-round execution recommendation is clear.
- Cross-method comparison matrix is filled for all criteria.
- The next skill is `data-auditor-cleaner` (or `workflow-orchestrator` if data is not needed).

# Failure modes

Stop and report a blocker if:

- No validated problem classification exists.
- Required outputs are unknown.
- Data availability is too unclear to select feasible methods.
- The only plausible methods require data that is unavailable or forbidden by contest rules.
- A subquestion's output requirements are so vague that no method can be meaningfully proposed.

# Stop conditions

This skill must stop instead of guessing when:

- Selecting a method would require inventing data fields, labels, constraints, or evaluation metrics.
- A subquestion cannot be linked to a required output.
- A method choice depends on an unavailable attachment.
- The user asks for final numerical results, code, or paper text before the candidate pool is complete.

When stopping, output:

- the blocker
- why it matters
- safe partial candidate pools if any
- missing information needed
- recommended next action

# Handoff

After producing the candidate method pool, hand off to:

`data-auditor-cleaner`
— to prepare data for the methods in the pool.

The handoff should include:

- Candidate method pool files (per subquestion).
- Required data fields per method.
- Baseline designations.
- First-round execution recommendations.
- Implementation difficulty notes.

If no external or tabular data is needed, the workflow owner may route next to `model-code-analyzer`.

# Examples

## Example 1: Evaluation problem candidate pool

Input state:
- Q1: evaluate and rank cities by resilience.
- Problem type: evaluation.
- Data: multiple indicators with mixed units.
- Literature: TOPSIS and entropy weighting commonly used in similar evaluation problems.

Output (`methods/Q1/q1_method_candidates.md`):

```markdown
# Q1 Method Candidate Pool

## 2. Candidate Method Pool

### Candidate M1: Equal-Weight Normalized Scoring

- **Math idea**: Normalize all indicators to [0,1] range (with direction handling), then compute a weighted sum with equal weights. Rank cities by total score descending.
- **Why it fits**: Simplest possible multi-indicator evaluation; transparent and easy to explain.
- **Strengths**:
  - No weight justification needed (all indicators treated equally)
  - Trivial to implement and verify
  - Serves as baseline for all other methods
- **Weaknesses**:
  - Cannot distinguish indicator importance
  - May overweight redundant indicators
- **Expected programmer outputs**:
  - Result files: `equal_weight_scores.csv`, `equal_weight_ranking.csv`
  - Figures: `indicator_distribution.png`, `score_bar_chart.png`
  - Metrics: score variance, ranking list
- **Evaluation criteria**: Ranking interpretability, score differentiation
- **Implementation difficulty**: easy
- **First-round priority**: high (baseline — always implement first)
- **Data requirements**: indicator matrix (all columns)
- **Estimated implementation time**: 0.5 hours

### Candidate M2: Entropy-Weight TOPSIS

- **Math idea**: Use information entropy of each indicator to compute objective weights (higher dispersion → higher weight). Then apply TOPSIS: find ideal best/worst virtual alternatives, compute each city's relative closeness to the ideal best, rank by closeness.
- **Why it fits**: Entropy provides objective data-driven weights (no subjective input needed). TOPSIS produces clear comparable scores. Widely used in mathematical modeling contests.
- **Strengths**:
  - Objective weights (no subjective bias)
  - Well-established in contest literature
  - Produces differentiable scores
- **Weaknesses**:
  - Entropy may overweight high-variance indicators that are not practically important
  - Sensitive to normalization method choice
- **Expected programmer outputs**:
  - Result files: `entropy_weights.csv`, `topsis_scores.csv`, `topsis_ranking.csv`
  - Figures: `weight_bar_chart.png`, `score_distribution.png`, `ranking_comparison.png` (vs baseline)
  - Metrics: weight values, relative closeness scores, ranking stability
- **Evaluation criteria**: Weight interpretability, ranking differentiation vs baseline, top-K stability
- **Implementation difficulty**: medium
- **First-round priority**: high (recommended main method)
- **Data requirements**: indicator matrix (all columns), indicator direction (positive/negative)
- **Estimated implementation time**: 2 hours

### Candidate M3: PCA-Assisted Evaluation

- **Math idea**: Apply PCA to reduce correlated indicators to independent principal components. Use component scores (weighted by explained variance) as composite evaluation scores.
- **Why it fits**: Handles indicator correlation automatically. Reduces dimensionality if many indicators exist.
- **Strengths**:
  - Handles multicollinearity well
  - Automatic dimensionality reduction
- **Weaknesses**:
  - Reduced interpretability (principal components are linear combinations)
  - May lose information from small-variance components that are practically important
  - Harder to explain to readers
- **Expected programmer outputs**:
  - Result files: `pca_loadings.csv`, `pca_scores.csv`, `pca_ranking.csv`
  - Figures: `scree_plot.png`, `biplot.png`, `pca_score_plot.png`
  - Metrics: explained variance ratio, component loadings, ranking vs TOPSIS comparison
- **Evaluation criteria**: Information retention (explained variance), consistency with M2 ranking
- **Implementation difficulty**: medium
- **First-round priority**: medium (try if M1/M2 results need validation)
- **Data requirements**: indicator matrix (all numeric columns)
- **Estimated implementation time**: 1.5 hours

## 3. Rejected Methods

| Method | Reason for Rejection |
|--------|---------------------|
| AHP + TOPSIS | Requires subjective pairwise comparison matrix; cannot be objectively justified without domain expert input |
| Deep neural network scoring | Insufficient labeled data; no ground-truth scores for training; weak interpretability |
| Fuzzy comprehensive evaluation | Membership function design is arbitrary without domain knowledge; similar to weighted scoring with extra complexity |

## 4. Baseline Designation

- **Baseline**: M1 (Equal-Weight Normalized Scoring)
- **Why**: Simplest transparent reference; any improvement over equal weights must be demonstrated

## 5. First-Round Execution Recommendation

- **Implement first**: M1 and M2
- **Reason**: M1 establishes the baseline; M2 is the recommended main method. M3 is a validation backup.
- **When to try M3**: If M2 ranking is suspicious (dominated by one or two high-variance indicators) or if indicators are highly correlated
- **What to look for in round 1**: Does M2 produce meaningfully different weights? Does the ranking differ from M1? Is the ranking stable?

## 6. Cross-Method Comparison Matrix

| Criterion | M1 Equal-Weight | M2 Entropy-TOPSIS | M3 PCA |
|-----------|---|---|---|
| Interpretability | ★★★★★ | ★★★★☆ | ★★★☆☆ |
| Data requirement satisfaction | ★★★★★ | ★★★★★ | ★★★★★ |
| Implementation complexity | ★★★★★ | ★★★☆☆ | ★★★☆☆ |
| Literature support | ★★★☆☆ | ★★★★★ | ★★★☆☆ |
| Expected differentiation | ★★☆☆☆ | ★★★★☆ | ★★★★☆ |
| Robustness potential | ★★★★★ | ★★★★☆ | ★★★☆☆ |
```

## Example 2: Prediction problem candidate pool

Input state:
- Q2: predict future demand.
- Problem type: prediction.
- Data: historical time series, 36 months.
- Literature: ARIMA and grey prediction common for short series.

Output (summary):
```markdown
### Candidate M1: Moving Average Baseline
- Math idea: Forecast = average of last K observations.
- Baseline: Yes.
- First-round priority: high.

### Candidate M2: ARIMA
- Math idea: Model time series as autoregressive integrated moving average process; fit order via AIC; forecast with confidence intervals.
- First-round priority: high.

### Candidate M3: Grey Prediction GM(1,1)
- Math idea: Accumulate data to reduce noise, fit first-order differential equation, inverse-accumulate to get predictions.
- First-round priority: medium (especially useful if data is short and monotonic).

### Rejected:
| LSTM | Data length (36 points) is insufficient for deep learning; high implementation risk |

### Baseline: M1 (Moving Average)
```

## Example 3: Optimization problem candidate pool

Input state:
- Q3: allocate limited resources to demand points.
- Problem type: optimization.
- Data: demand estimates, resource capacity, cost parameters.

Output (summary):
```markdown
### Candidate M1: Greedy Allocation (Baseline)
- Math idea: Sort demand points by priority, allocate resources greedily from highest to lowest until capacity exhausted.
- Baseline: Yes.

### Candidate M2: Integer Linear Programming
- Math idea: Define binary allocation variables, minimize total cost subject to capacity and demand constraints, solve with branch-and-bound.

### Candidate M3: Multi-Objective Optimization
- Math idea: Extend M2 with additional objectives (fairness, coverage) using weighted sum or epsilon-constraint method.

### Rejected:
| Genetic Algorithm | Overkill; problem size is manageable with exact methods; GA adds randomness and reduces reproducibility |

### Baseline: M1 (Greedy Allocation)
```

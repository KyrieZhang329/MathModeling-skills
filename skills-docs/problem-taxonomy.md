# Problem Taxonomy

This document defines the problem types used by `problem-classifier`.

The purpose is not to label an entire contest problem with one word. Most modeling contest problems are mixed. The useful unit is the subquestion: classify each subquestion by what it asks for and what output it requires.

## Why classification matters

Problem type affects:

- what model families are reasonable
- what data is required
- what result format is expected
- what validation is needed
- what risks should be checked later

A wrong type usually leads to the wrong model. For example, a task asking for a ranking should not be treated as a prediction task just because the dataset has time fields. A task asking for an allocation plan should not stop at descriptive analysis.

## Classification principle

Classify by the required output, not by the topic title.

Use these questions:

1. What does this subquestion ask the team to produce?
2. Is the output a score, forecast, decision, explanation, group, path, simulation result, or data insight?
3. What evidence would make the answer acceptable?
4. Does this subquestion depend on outputs from earlier subquestions?

If a problem has several subquestions, classify each one separately.

## Standard types

Use the following labels consistently:

```text
evaluation
prediction
optimization
mechanism
classification-clustering
graph-routing
simulation
data-analysis
hybrid
```

## **Type guide**

### **Evaluation**

Use this type when the main output is a score, ranking, grade, priority order, risk level, or comprehensive assessment.

Common wording:

- evaluate
- rank
- compare
- assess
- score
- determine importance
- build an index system
- comprehensive evaluation

Typical outputs:

- ranking table
- score table
- indicator weights
- evaluation level
- priority list

Common method families:

- indicator system construction
- normalization
- entropy weight
- AHP
- TOPSIS
- grey relational analysis
- fuzzy comprehensive evaluation
- PCA-based evaluation

Common risks:

- arbitrary indicators
- arbitrary weights
- mixing positive and negative indicators incorrectly
- repeated indicators double-counting the same factor
- ranking with no sensitivity check

Minimum checks:

- indicator direction check
- weight justification
- ranking stability
- comparison with a simple baseline such as equal-weight scoring

### **Prediction**

Use this type when the main output is a future value, unknown value estimate, trend, forecast interval, or uncertainty estimate.

Common wording:

- predict
- forecast
- estimate
- infer future
- trend
- extrapolate
- fill missing values

Typical outputs:

- predicted values
- trend curve
- error metrics
- forecast interval
- residual analysis

Common method families:

- naive baseline
- moving average
- linear or nonlinear regression
- time series models
- grey prediction
- machine learning regression
- ensemble prediction

Common risks:

- using training fit as future performance
- no validation split
- no error metric
- feature leakage
- overclaiming long-term forecasts
- using high-capacity models with small data

Minimum checks:

- baseline comparison
- train-test split or cross-validation when possible
- MAE, RMSE, MAPE, or task-appropriate metric
- residual or failure-case discussion

### **Optimization**

Use this type when the main output is a decision plan under constraints.

Common wording:

- optimize
- maximize
- minimize
- allocate
- schedule
- assign
- choose
- route
- design a plan
- determine the best strategy

Typical outputs:

- decision variables
- objective value
- feasible plan
- allocation table
- schedule
- route
- constraint satisfaction report

Common method families:

- linear programming
- integer programming
- nonlinear programming
- dynamic programming
- multi-objective optimization
- network optimization
- greedy baseline
- heuristic search
- genetic algorithm or simulated annealing for hard combinatorial cases

Common risks:

- unclear decision variables
- objective function has no real meaning
- missing constraints
- plan is mathematically optimal but not implementable
- only reporting the optimal value without the actual decision plan

Minimum checks:

- decision variable definition
- objective and constraints
- feasibility check
- baseline comparison
- sensitivity to key constraints or weights

### **Mechanism**

Use this type when the main output is an explanation of how a system works or evolves.

Common wording:

- explain mechanism
- describe process
- derive relationship
- model evolution
- simulate dynamics from rules
- analyze spread, growth, decay, flow, motion, or interaction

Typical outputs:

- equations
- state variables
- parameter estimates
- process explanation
- dynamic curves
- boundary conditions

Common method families:

- differential equations
- difference equations
- compartment models
- conservation laws
- physical or geometric constraints
- system dynamics
- mechanism plus parameter fitting

Common risks:

- undefined variables
- inconsistent units
- parameters with no source
- assumptions too strong
- no validation against data or common sense

Minimum checks:

- variable and unit consistency
- parameter source
- boundary condition check
- sensitivity to important parameters

### **Classification-clustering**

Use this type when the main output is a label, group, segment, class, or anomaly.

Common wording:

- classify
- cluster
- group
- segment
- identify category
- detect abnormal samples
- divide into types

Typical outputs:

- class labels
- cluster assignments
- anomaly list
- confusion matrix
- cluster validation metric
- feature importance or explanation

Common method families:

- rule-based baseline
- logistic regression
- decision tree
- random forest
- SVM
- k-means
- hierarchical clustering
- DBSCAN
- anomaly detection

Common risks:

- no labels for supervised classification
- arbitrary number of clusters
- feature scaling ignored
- class imbalance
- no validation metric
- clusters described after the fact without evidence

Minimum checks:

- label availability
- feature scaling
- class balance or cluster validity
- accuracy, F1, confusion matrix, silhouette score, or task-appropriate metric

### **Graph-routing**

Use this type when the problem naturally involves nodes, edges, paths, routes, networks, flows, or matching.

Common wording:

- network
- path
- route
- shortest path
- flow
- matching
- connectivity
- node importance
- transportation
- routing

Typical outputs:

- route plan
- shortest path
- flow allocation
- matching result
- network structure
- node ranking
- connectivity analysis

Common method families:

- shortest path
- minimum spanning tree
- maximum flow / minimum cost flow
- matching
- vehicle routing
- graph centrality
- graph search
- network optimization

Common risks:

- graph abstraction ignores real constraints
- edge weights are arbitrary
- static network assumption hides time-varying constraints
- route is mathematically valid but practically infeasible

Minimum checks:

- node and edge definition
- edge weight justification
- feasibility check
- comparison with a simple route or greedy baseline

### **Simulation**

Use this type when the main output is a scenario result, stochastic outcome distribution, or repeated process behavior.

Common wording:

- simulate
- scenario
- random
- probability distribution
- Monte Carlo
- repeated trials
- agent behavior
- stochastic process

Typical outputs:

- scenario table
- distribution summary
- confidence interval
- simulation curve
- policy comparison
- expected value and variance

Common method families:

- Monte Carlo simulation
- discrete-event simulation
- agent-based simulation
- stochastic process modeling
- scenario analysis

Common risks:

- hidden random assumptions
- random seed not fixed
- too few trials
- no statistical summary
- scenario design not justified

Minimum checks:

- random seed
- number of trials
- parameter distribution source
- confidence interval or summary statistics
- sensitivity to scenario assumptions

### **Data-analysis**

Use this type when the main output is a descriptive finding, pattern, correlation, factor relationship, or data insight before modeling.

Common wording:

- analyze data
- describe pattern
- identify influencing factors
- explore relationship
- summarize characteristics
- compare distributions

Typical outputs:

- descriptive statistics
- correlation table
- distribution plot
- feature relationship
- factor explanation
- data quality findings

Common method families:

- descriptive statistics
- correlation analysis
- hypothesis testing
- exploratory visualization
- PCA or factor analysis
- regression explanation

Common risks:

- correlation treated as causation
- decorative figures
- analysis not connected to later modeling
- ignoring missing values or outliers

Minimum checks:

- data quality summary
- unit and field meaning check
- relationship to later subquestions
- clear distinction between correlation and causation

### **Hybrid**

Use this type when different subquestions require different task types, or when one stage produces inputs for another stage.

Common patterns:

- evaluation → prediction → optimization
- data analysis → prediction → decision
- mechanism model → parameter fitting → simulation
- classification → evaluation → recommendation
- prediction → routing or allocation

Typical outputs:

- staged workflow
- dependency map
- intermediate artifacts
- final integrated recommendation

Common risks:

- treating the whole problem as one type
- using one model to answer unrelated subquestions
- skipping dependencies between subquestions
- inconsistent variables across stages
- paper sections not matching the actual workflow

Minimum checks:

- classify each subquestion separately
- identify dependencies
- define artifacts passed between stages
- check consistency of variables and outputs across stages

## **Common misclassification patterns**

### **Prediction vs evaluation**

If the output is a ranking or score, it is usually evaluation.

If the output is a future value or unknown value estimate, it is prediction.

A dataset with time fields does not automatically make the problem a prediction problem.

### **Data analysis vs modeling**

If the task only asks for patterns, relationships, or descriptive findings, it may be data-analysis.

If the task requires decisions, forecasts, rankings, or mechanisms, data analysis is only a supporting stage.

### **Optimization vs evaluation**

If the task asks “which is better,” it is usually evaluation.

If the task asks “what should we choose under constraints,” it is usually optimization.

### **Simulation vs prediction**

If the task asks for a future value from historical data, it is usually prediction.

If the task asks for behavior under scenarios or random processes, it is usually simulation.

### **Mechanism vs regression**

If the task needs an interpretable process or law-like relationship, mechanism modeling may be needed.

If the task only needs empirical mapping from inputs to outputs, regression or prediction may be enough.

## **Output format for classification**

A useful classification artifact should include:

```json
{
  "subquestion_classifications": [
    {
      "id": "Q1",
      "primary_type": "evaluation",
      "secondary_type": "data-analysis",
      "reason": "The required output is a ranking based on multiple indicators.",
      "expected_outputs": [
        "score table",
        "ranking table"
      ],
      "candidate_method_families": [
        "indicator system construction",
        "multi-criteria evaluation"
      ],
      "risks": [
        "indicator weights need justification",
        "ranking stability should be checked"
      ]
    }
  ],
  "global_pattern": "hybrid",
  "dependency_notes": [
    "Q2 uses Q1 scores as input."
  ]
}
```

## **Practical rule**

When unsure, write down the required output first.

The output usually tells you the type.

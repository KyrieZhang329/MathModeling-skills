# Method Selection Tree

This document explains how to choose baseline, main, and optional improved methods after each subquestion has been classified.

It supports `method-selector`. It is not meant to replace modeling judgment. It gives a practical decision path for common mathematical modeling contest tasks.

## Purpose

A method plan should answer four questions:

1. What is the simplest valid baseline?
2. What is the main model?
3. Is an improved model necessary?
4. What evidence is needed to justify the result?

Do not choose a method only because it looks advanced. Choose a method because it fits the task type, data, output requirement, time limit, and explainability requirement.

## General rules

- Every main model should have a baseline unless there is a clear reason not to.
- The baseline should be simple, runnable, and meaningful.
- The main model should directly produce the required output.
- An improved model is optional. Add it only if it solves a real weakness of the main model.
- Do not use a complex model when a simpler model already answers the subquestion well.
- Do not use deep learning unless the data size, labels, validation setup, and contest time support it.
- Do not change the problem type to fit a favorite method.
- Do not select a method before checking the available data.
- Do not plan paper claims before planning validation and robustness checks.

## Basic selection process

```text
parsed subquestion
→ problem type
→ required output
→ available data
→ baseline
→ main model
→ optional improvement
→ validation plan
→ robustness plan
```

For each subquestion, write down:

- task type
- required output
- available data
- missing data
- baseline method
- main method
- optional improvement
- rejected methods
- validation checks
- robustness checks

## **By problem type**

## **Evaluation**

Use when the required output is a score, ranking, grade, risk level, priority list, or comprehensive assessment.

### **Baseline options**

- equal-weight scoring
- simple normalized weighted sum
- manually defined rubric
- rank by one key indicator

### **Main model options**

- entropy-weight TOPSIS
- AHP + TOPSIS
- PCA-based comprehensive evaluation
- grey relational analysis
- fuzzy comprehensive evaluation
- rank-sum ratio method

### **Optional improvements**

Use optional improvements when needed:

- combine subjective and objective weights
- compare several weighting schemes
- add ranking stability analysis
- use PCA if indicators are highly correlated
- use fuzzy evaluation if criteria are qualitative or vague

### **Recommended choice**

For many contest tasks with tabular indicators:

```text
baseline: equal-weight scoring
main model: entropy-weight TOPSIS
improvement: weight perturbation / ranking stability check
```

Use AHP when expert preference or hierarchical criteria are important.

Use PCA when many indicators are correlated and dimensionality reduction is useful.

Use fuzzy evaluation when qualitative levels or vague membership are central.

### **Avoid**

- arbitrary weights with no explanation
- black-box scoring without labels
- repeated indicators that double-count the same factor
- ranking without sensitivity analysis

### **Validation and robustness**

Check:

- indicator direction
- normalization method
- weight source
- ranking stability
- sensitivity to weights
- comparison with equal-weight baseline

## **Prediction**

Use when the required output is a future value, unknown value, trend, forecast interval, or uncertainty estimate.

### **Baseline options**

- last-value baseline
- mean baseline
- moving average
- seasonal naive forecast
- simple linear regression

### **Main model options**

- linear or nonlinear regression
- ARIMA or exponential smoothing
- grey prediction for short monotonic series
- random forest or gradient boosting for tabular nonlinear data
- ensemble forecast if several models are justified

### **Optional improvements**

Use optional improvements when needed:

- feature engineering
- rolling validation
- ensemble averaging
- uncertainty interval
- residual correction
- model comparison table

### **Recommended choice**

For short time series:

```text
baseline: moving average or last-value baseline
main model: ARIMA / exponential smoothing / grey prediction
improvement: residual analysis and short-horizon validation
```

For tabular prediction with enough features:

```text
baseline: linear regression
main model: random forest or gradient boosting
improvement: feature importance and cross-validation
```

### **Avoid**

- deep learning on very small datasets
- claiming accuracy from training fit alone
- long-term extrapolation without uncertainty
- using future information as features
- prediction without error metrics

### **Validation and robustness**

Check:

- train-test split
- cross-validation when feasible
- MAE, RMSE, MAPE, accuracy, F1, or task-specific metric
- residual pattern
- baseline comparison
- feature leakage
- extrapolation boundary

## **Optimization**

Use when the required output is a decision plan, allocation, schedule, route, assignment, or best strategy under constraints.

### **Baseline options**

- greedy rule
- current-policy comparison
- equal allocation
- random feasible solution
- exhaustive search for small cases

### **Main model options**

- linear programming
- integer programming
- nonlinear programming
- dynamic programming
- multi-objective optimization
- network flow
- shortest path
- vehicle routing
- heuristic search

### **Optional improvements**

Use optional improvements when needed:

- multi-objective weighting
- Pareto frontier
- fairness constraint
- robust optimization
- scenario-based optimization
- heuristic improvement for large combinatorial cases

### **Recommended choice**

For allocation with linear objective and constraints:

```text
baseline: greedy allocation
main model: linear programming or integer programming
improvement: constraint sensitivity analysis
```

For discrete choices:

```text
baseline: greedy or simple rule
main model: integer programming
improvement: multi-objective extension or sensitivity to resource limits
```

For routing or network allocation:

```text
baseline: nearest-neighbor or direct route
main model: shortest path / network flow / vehicle routing
improvement: scenario or constraint perturbation
```

### **Avoid**

- objective functions with no real meaning
- missing constraints
- reporting only the optimal value without a plan
- heuristics without baseline comparison
- mathematically feasible but practically impossible plans

### **Validation and robustness**

Check:

- decision variables
- objective function
- constraints
- feasibility
- baseline comparison
- sensitivity to resource limits, weights, or costs
- whether the final plan is implementable

## **Mechanism**

Use when the required output is an explanation of process, dynamics, physical relationship, causal structure, growth, decay, spread, flow, or interaction.

### **Baseline options**

- simplified algebraic relationship
- steady-state model
- empirical fit
- discrete approximation

### **Main model options**

- differential equations
- difference equations
- compartment models
- conservation laws
- physical or geometric constraints
- system dynamics
- mechanism model with parameter fitting

### **Optional improvements**

Use optional improvements when needed:

- parameter calibration
- boundary condition analysis
- sensitivity to key parameters
- coupling with data-driven fitting
- scenario simulation

### **Recommended choice**

For spread or transition processes:

```text
baseline: simplified discrete model
main model: compartment or difference equation model
improvement: parameter sensitivity
```

For physical or geometric processes:

```text
baseline: simplified equation
main model: constraint-based mechanism model
improvement: boundary condition analysis
```

### **Avoid**

- undefined variables
- inconsistent units
- parameters with no source
- strong assumptions hidden as facts
- no validation against data or common sense

### **Validation and robustness**

Check:

- units
- parameter sources
- boundary conditions
- sensitivity to key parameters
- fit to observed data if available
- whether assumptions are realistic enough for the contest problem

## **Classification-clustering**

Use when the required output is a category label, group assignment, segment, type, or anomaly.

### **Baseline options**

- rule-based threshold
- majority class
- simple k-means
- simple decision tree
- manual grouping by key features

### **Main model options**

- logistic regression
- decision tree
- random forest
- SVM
- k-means
- hierarchical clustering
- DBSCAN
- anomaly detection

### **Optional improvements**

Use optional improvements when needed:

- feature selection
- dimensionality reduction
- class imbalance handling
- cluster number validation
- interpretability analysis
- anomaly threshold sensitivity

### **Recommended choice**

For supervised classification with labels:

```text
baseline: majority class or decision tree
main model: logistic regression / random forest / SVM
improvement: cross-validation and feature importance
```

For clustering without labels:

```text
baseline: k-means with simple features
main model: k-means / hierarchical clustering / DBSCAN
improvement: silhouette score and cluster stability
```

### **Avoid**

- supervised methods without labels
- arbitrary cluster count
- ignoring feature scaling
- reporting accuracy only under class imbalance
- naming clusters without evidence

### **Validation and robustness**

Check:

- label availability
- class balance
- feature scaling
- confusion matrix, F1, AUC, or task-specific metric
- silhouette score or clustering validity index
- cluster stability

## **Graph-routing**

Use when the problem involves nodes, edges, networks, paths, routes, flows, matching, or connectivity.

### **Baseline options**

- direct distance route
- greedy nearest-neighbor route
- simple connectivity check
- simple shortest path under one weight

### **Main model options**

- shortest path
- minimum spanning tree
- maximum flow
- minimum cost flow
- matching
- vehicle routing
- graph centrality
- graph search
- network optimization

### **Optional improvements**

Use optional improvements when needed:

- multi-weight edge cost
- time-varying network
- capacity constraint
- route robustness
- scenario comparison
- fairness or service constraint

### **Recommended choice**

For path planning:

```text
baseline: shortest path under distance only
main model: weighted shortest path
improvement: sensitivity to edge weights
```

For allocation over networks:

```text
baseline: simple proportional allocation
main model: network flow
improvement: capacity sensitivity
```

### **Avoid**

- arbitrary edge weights
- graph abstraction that ignores key real constraints
- static graph when time-dependence is essential
- route that is mathematically valid but practically infeasible

### **Validation and robustness**

Check:

- node definition
- edge definition
- edge weight source
- feasibility of paths or flows
- baseline route comparison
- sensitivity to edge weights or capacity limits

## **Simulation**

Use when the required output is a scenario result, stochastic distribution, repeated process behavior, or policy comparison under uncertainty.

### **Baseline options**

- deterministic scenario table
- expected-value calculation
- small manual scenario comparison
- simplified simulation with fixed parameters

### **Main model options**

- Monte Carlo simulation
- discrete-event simulation
- agent-based simulation
- stochastic process model
- scenario analysis

### **Optional improvements**

Use optional improvements when needed:

- more scenarios
- trial count sensitivity
- random seed stability
- confidence intervals
- policy comparison
- parameter distribution sensitivity

### **Recommended choice**

For uncertainty-driven results:

```text
baseline: deterministic expected-value calculation
main model: Monte Carlo simulation
improvement: trial count and seed stability
```

For process queues or events:

```text
baseline: simplified average-case calculation
main model: discrete-event simulation
improvement: scenario comparison
```

### **Avoid**

- hidden random assumptions
- no fixed random seed
- too few trials
- no statistical summary
- unsupported parameter distributions

### **Validation and robustness**

Check:

- random seed
- trial count
- parameter distribution source
- confidence interval or variance
- scenario assumptions
- comparison across policies or settings

## **Data-analysis**

Use when the required output is a descriptive finding, pattern, correlation, feature relationship, or data insight before deeper modeling.

### **Baseline options**

- descriptive statistics
- distribution summary
- simple correlation table
- missing value summary

### **Main model options**

- correlation analysis
- hypothesis testing
- exploratory visualization
- PCA or factor analysis
- regression explanation
- feature importance analysis

### **Optional improvements**

Use optional improvements when needed:

- subgroup analysis
- robustness to outliers
- alternative correlation measure
- dimensionality reduction
- visualization refinement

### **Recommended choice**

For early data exploration:

```text
baseline: descriptive statistics
main model: correlation or factor analysis
improvement: outlier and subgroup sensitivity
```

### **Avoid**

- treating correlation as causation
- decorative charts
- analysis that does not support later modeling
- ignoring missing values, units, or outliers

### **Validation and robustness**

Check:

- data quality
- units and field meanings
- correlation vs causation
- outlier effect
- link to later subquestions

## **Hybrid**

Use when different subquestions require different task types, or when one stage produces inputs for another stage.

### **Common patterns**

```text
evaluation → prediction → optimization
data-analysis → prediction → decision
mechanism → parameter fitting → simulation
classification → evaluation → recommendation
prediction → routing or allocation
```

### **Baseline options**

Baselines should be defined per subquestion, not for the whole problem.

Examples:

- equal-weight scoring for an evaluation stage
- moving average for a prediction stage
- greedy allocation for an optimization stage

### **Main model options**

Use a staged pipeline. Each stage should have:

- its own task type
- its own method
- its own output artifact
- a clear handoff to the next stage

### **Optional improvements**

Use optional improvements to improve handoff reliability:

- consistency check between stages
- sensitivity of later decisions to earlier outputs
- uncertainty propagation
- scenario comparison

### **Avoid**

- using one model to answer unrelated subquestions
- ignoring dependencies between subquestions
- changing variable definitions across stages
- writing the paper as if the workflow were simpler than it is

### **Validation and robustness**

Check:

- every subquestion has a method
- every handoff artifact is defined
- later-stage outputs remain stable when earlier-stage outputs change
- variables are consistent across stages

## **When not to use complex methods**

Avoid complex methods when:

- the dataset is small
- labels are missing
- the contest time is short
- the method is hard to explain
- the method does not produce the required output directly
- validation would be weak
- a simpler baseline already solves the task well
- the team cannot debug the method under time pressure

Complexity is useful only when it solves a real problem.

## **Baseline rules**

A baseline should be:

- simple
- explainable
- runnable
- comparable to the main model
- aligned with the required output

Bad baselines include:

- a method that cannot produce the same output as the main model
- a method that is too weak to be meaningful
- a method added only to satisfy the word “baseline”
- a method whose results are not saved

## **Robustness planning**

Plan robustness before paper writing.

Common checks:

| **Task type**             | **Useful checks**                                            |
| ------------------------- | ------------------------------------------------------------ |
| Evaluation                | weight perturbation, ranking stability, normalization comparison |
| Prediction                | error metrics, residuals, train-test split, baseline comparison |
| Optimization              | feasibility, constraint sensitivity, objective weight sensitivity |
| Mechanism                 | parameter sensitivity, boundary condition check, unit consistency |
| Classification-clustering | validation metric, class balance, cluster stability          |
| Graph-routing             | edge weight sensitivity, route feasibility, capacity changes |
| Simulation                | seed stability, trial count sensitivity, scenario comparison |
| Data-analysis             | outlier influence, correlation robustness, subgroup comparison |
| Hybrid                    | handoff consistency, uncertainty propagation, stage dependency checks |

## **Output format**

A method plan should look like this:

```json
{
  "subproblem_methods": [
    {
      "id": "Q1",
      "problem_type": "evaluation",
      "required_output": [
        "score table",
        "ranking table"
      ],
      "baseline_model": {
        "name": "equal-weight scoring",
        "reason": "simple reference ranking"
      },
      "main_model": {
        "name": "entropy-weight TOPSIS",
        "reason": "fits multi-indicator evaluation and produces comparable scores"
      },
      "improved_model": {
        "name": "weight perturbation stability check",
        "condition": "use to test whether rankings are sensitive to weights"
      },
      "rejected_methods": [
        {
          "name": "deep neural network scoring",
          "reason": "no labeled data and poor interpretability"
        }
      ],
      "validation_plan": [
        "compare with baseline",
        "check indicator directions",
        "verify ranking stability"
      ],
      "robustness_plan": [
        "weight perturbation",
        "normalization comparison"
      ]
    }
  ],
  "recommended_next_skill": "data-auditor-cleaner"
}
```

## **Practical rule**

Choose the simplest method that can produce the required output and survive basic validation.

Then add complexity only where the baseline fails for a clear reason.


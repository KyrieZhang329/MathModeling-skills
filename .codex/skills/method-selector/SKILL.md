---
name: method-selector
description: Compare 2-4 candidate modeling schemes for each subproblem and recommend one execution route based on task type, data, interpretability, literature analysis, and contest constraints.
license: MIT
---

# Purpose

Select feasible modeling routes for each classified subquestion in a mathematical modeling contest.

This skill converts a validated problem parse, problem classification, and related-paper analysis into a method plan. For each subquestion, it should usually compare 2-4 candidate modeling schemes before recommending one execution route. Within the recommended route, it may still define a baseline model, a main model, and, when justified, an improved model.

This skill does not generate code, clean data, run experiments, create figures, or write paper sections.

# When to use

Use this skill:

- After `problem-parser`, `problem-classifier`, and `related-paper-analyzer` have produced validated artifacts.
- Before data cleaning, code generation, robustness checks, figure planning, or paper writing.
- When the team needs several contest-feasible modeling routes for each subquestion before committing to one execution plan.
- When multiple plausible methods exist and the team needs a contest-feasible choice.
- When a model choice must be justified against task type, data availability, interpretability, and time constraints.

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
- Any known baseline requirements or scoring criteria.

# Workflow

1. Read the parsed problem and classification artifacts.
   - Work subquestion by subquestion.
   - Respect dependencies between subquestions.
   - Do not collapse different subquestions into one generic method.

2. For each subquestion, identify method requirements.
   - Required output form.
   - Available data and missing data.
   - Interpretability requirement.
   - Computational complexity.
   - Implementation risk.
   - Validation or evaluation requirement.

3. Generate candidate schemes.
   - Propose 2-4 candidate modeling schemes for each subquestion when feasible.
   - Keep schemes meaningfully different rather than cosmetic variants of the same idea.
   - Use task type, data conditions, contest limits, and literature cues to justify why each scheme is worth considering.

4. Compare and recommend one execution route.
   - Compare candidate schemes on feasibility, interpretability, implementation risk, data fit, and validation burden.
   - Recommend one scheme for execution.
   - Preserve the other schemes as explicit alternatives instead of collapsing them into a single hidden choice.

5. Define baseline, main model, and improved model inside the recommended route.
   - Choose the baseline as the simplest meaningful reference inside the recommended route.
   - Choose the main model as the preferred executable method for the contest.
   - Choose an improved model only if it addresses a specific weakness and remains contest-feasible.

6. Reject unsuitable methods.
   - Explain why common tempting methods are not recommended.
   - Reject methods unsupported by data size, data quality, interpretability needs, or implementation time.

7. Define expected artifacts.
   - Required input data.
   - Expected intermediate outputs.
   - Expected result tables.
   - Expected figures.
   - Required robustness or sensitivity checks.
   - Handoff requirements for `data-auditor-cleaner` and `model-code-analyzer`.

8. Produce a method plan.
   - Keep it structured.
   - Make every method traceable to a subquestion.
   - Save the overall JSON plan under `workspace/problem/method-selector/`.
   - Write one Markdown comparison document per subquestion under `workspace/problem/method-selector/`, such as `Q1.md`, `Q2.md`, or `Q3.md`.
   - Each subquestion Markdown document should compare 2-4 candidate schemes and identify the recommended route for that subquestion.
   - Do not start implementation inside this skill.

# Outputs

Produce a method plan as paired problem-stage artifacts:

- `workspace/problem/method-selector/method_plan.json`
- `workspace/problem/method-selector/Q1.md`
- `workspace/problem/method-selector/Q2.md`
- additional per-subquestion Markdown files such as `Q3.md` or `Q4.md` when needed

The artifacts should contain:

- `method_plan_summary`
- `subproblem_methods`
- `candidate_schemes`
- `recommended_scheme_id`
- `dependency_plan`
- `data_requirements`
- `expected_artifacts`
- `validation_plan`
- `robustness_plan`
- `rejected_methods`
- `implementation_notes`
- `recommended_next_skill`

# Output format

Prefer this JSON-compatible structure for `workspace/problem/method-selector/method_plan.json`:

```json
{
  "method_plan_summary": {
    "overall_strategy": "staged modeling workflow",
    "reason": "Different subquestions require evaluation, prediction, and optimization in sequence.",
    "implementation_priority": [
      "build baseline models first",
      "validate main models",
      "add improvements only if time and data allow"
    ]
  },
  "subproblem_methods": [
    {
      "id": "Q1",
      "problem_type": "evaluation",
      "candidate_schemes": [
        {
          "scheme_id": "Q1-S1",
          "name": "weighted scoring route",
          "core_methods": [
            "normalization",
            "weighted scoring"
          ],
          "strengths": [
            "simple",
            "easy to explain"
          ],
          "weaknesses": [
            "sensitive to weight design"
          ]
        },
        {
          "scheme_id": "Q1-S2",
          "name": "TOPSIS route",
          "core_methods": [
            "entropy weighting",
            "TOPSIS"
          ],
          "strengths": [
            "fits multi-indicator ranking",
            "clear score output"
          ],
          "weaknesses": [
            "indicator direction and normalization must be handled carefully"
          ]
        }
      ],
      "recommended_scheme_id": "Q1-S2",
      "required_output": [
        "score",
        "ranking",
        "interpretation"
      ],
      "baseline_model": {
        "name": "equal-weight scoring",
        "role": "provide a transparent reference ranking",
        "required_data": [
          "normalized indicators"
        ],
        "expected_output": [
          "baseline score table",
          "baseline ranking"
        ],
        "limitations": [
          "does not distinguish indicator importance"
        ]
      },
      "main_model": {
        "name": "entropy-weight TOPSIS",
        "role": "produce an objective weighted multi-indicator ranking",
        "selection_reason": [
          "matches evaluation task",
          "works with tabular indicator data",
          "more explainable than black-box scoring"
        ],
        "required_data": [
          "indicator matrix",
          "positive or negative direction for each indicator"
        ],
        "expected_output": [
          "indicator weights",
          "relative closeness scores",
          "final ranking"
        ],
        "validation_need": [
          "weight sensitivity check",
          "ranking stability check"
        ]
      },
      "improved_model": {
        "name": "PCA-assisted evaluation comparison",
        "role": "check whether dimensionality reduction changes the ranking materially",
        "condition": "use only if indicators are highly correlated",
        "risk": "reduced interpretability"
      },
      "rejected_methods": [
        {
          "name": "deep neural network scoring",
          "reason": "insufficient labeled data and weak interpretability"
        }
      ],
      "expected_figures_tables": [
        "indicator direction table",
        "weight table",
        "ranking table",
        "sensitivity plot"
      ],
      "implementation_difficulty": "medium"
    }
  ],
  "dependency_plan": [
    {
      "from": "Q1",
      "to": "Q2",
      "artifact_needed": "Q1 scores may be used as explanatory variables for Q2"
    }
  ],
  "data_requirements": [
    {
      "subquestion": "Q1",
      "required_data": [
        "indicator matrix"
      ],
      "missing_data": [],
      "data_risks": [
        "mixed units",
        "positive and negative indicators"
      ]
    }
  ],
  "validation_plan": [
    "compare baseline and main model outputs",
    "check whether results satisfy required output form",
    "verify units and variable definitions"
  ],
  "robustness_plan": [
    "parameter sensitivity",
    "weight perturbation",
    "ranking stability",
    "baseline comparison"
  ],
  "rejected_methods": [
    {
      "method": "black-box neural model",
      "reason": "not enough labeled data or interpretability support"
    }
  ],
  "implementation_notes": [
    "Generate code only after data auditing confirms the required fields."
  ],
  "recommended_next_skill": "data-auditor-cleaner"
}
```

Also produce one Markdown document per subquestion under `workspace/problem/method-selector/`.

Each Markdown document should summarize:

- the subquestion goal
- the 2-4 candidate schemes considered
- strengths and weaknesses of each scheme
- the recommended scheme for execution
- baseline, main model, and optional improved model inside the recommended route
- required data, expected artifacts, and validation notes

# Model selection rules

- Provide 2-4 candidate schemes for each subquestion when feasible.
- Do not treat baseline, main model, and improved model as substitutes for multiple candidate schemes.
- Recommend one execution route, but preserve the alternatives explicitly.
- Every main model must have a baseline.
- Every model must map to a specific subquestion.
- Every model must produce the required output form.
- Every model must state required data and expected artifacts.
- Every model must include a validation or checking plan.
- Prefer simple, explainable, contest-feasible methods over complex but fragile methods.
- Improved models are optional and must address a specific weakness.
- Reject methods that are unsupported by data, interpretability, time, or implementation constraints.
- Do not recommend deep learning unless data size, task structure, and validation conditions justify it.
- Do not choose a method only because it appears in an excellent paper.
- Do not use a single method to answer unrelated subquestions poorly.

# Method family guide

Use this section to select method families. These are guidelines, not mandatory choices.

## Evaluation

Baseline options:

- equal-weight scoring
- simple normalized weighted sum
- manual rubric with transparent weights

Main model options:

- entropy-weight TOPSIS
- AHP plus TOPSIS
- PCA-assisted comprehensive evaluation
- grey relational analysis
- fuzzy comprehensive evaluation
- RSR or rank-based evaluation

Use when:

- the output is score, ranking, grade, risk level, priority, or comparison.

Check:

- indicator justification
- positive and negative indicator direction
- normalization
- weight source
- ranking stability
- sensitivity to weights

Avoid:

- arbitrary weights with no explanation
- black-box scoring without labeled data
- repeated indicators that double-count the same factor

## Prediction

Baseline options:

- mean or last-value baseline
- linear regression
- simple moving average
- naive seasonal baseline

Main model options:

- regression with feature engineering
- ARIMA or other time series models
- grey prediction for small-sample monotonic sequences
- random forest or gradient boosting for tabular nonlinear prediction
- ensemble prediction if multiple models are justified

Use when:

- the output is future value, trend, forecast interval, missing value estimate, or uncertainty estimate.

Check:

- train-test split or validation scheme
- error metrics
- residuals or failure cases
- long-term extrapolation limits
- feature leakage

Avoid:

- high-capacity models with tiny data
- prediction without error metrics
- claiming future accuracy from training fit alone

## Optimization

Baseline options:

- greedy rule
- exhaustive search for small cases
- simple heuristic
- current-policy comparison

Main model options:

- linear programming
- integer programming
- nonlinear programming
- dynamic programming
- multi-objective optimization
- network flow
- shortest path or routing model
- simulated annealing or genetic algorithm for hard combinatorial cases

Use when:

- the output is a decision plan, allocation, route, schedule, assignment, maximum, minimum, or feasible strategy.

Check:

- decision variables
- state variables
- parameters
- objective function
- constraints
- feasibility
- implementable final plan
- sensitivity to weights or constraints

Avoid:

- objective functions with vague meaning
- missing constraints
- final result that gives only an optimal value but no plan
- heuristics without baseline comparison

## Mechanism

Baseline options:

- simplified algebraic relationship
- discrete approximation
- steady-state model
- empirical fit to a known equation

Main model options:

- differential equations
- difference equations
- compartment models
- conservation laws
- physical or geometric constraints
- system dynamics
- mechanism plus parameter fitting

Use when:

- the output must explain a process, dynamic mechanism, physical relation, biological spread, flow, motion, growth, or decay.

Check:

- assumptions
- units
- parameter sources
- validation against data or common sense
- boundary conditions

Avoid:

- equations with undefined variables
- parameters with no source
- mechanism claims unsupported by validation

## Classification or clustering

Baseline options:

- rule-based classification
- k-means with simple features
- majority-class baseline
- threshold rule

Main model options:

- decision tree
- logistic regression
- random forest
- SVM
- clustering with validation index
- anomaly detection

Use when:

- the output is class label, cluster, segment, group, or anomaly.

Check:

- feature scaling
- class balance
- validation metric
- cluster number selection
- interpretability

Avoid:

- arbitrary cluster counts
- classification without labels
- accuracy-only reporting under class imbalance

## Graph or routing

Baseline options:

- direct distance rule
- greedy nearest-neighbor route
- simple connectivity analysis

Main model options:

- shortest path
- minimum spanning tree
- network flow
- matching
- graph centrality
- vehicle routing
- graph search

Use when:

- the problem involves nodes, edges, paths, networks, flows, routes, matching, or connectivity.

Check:

- graph abstraction
- edge weights
- node definitions
- feasibility constraints
- static vs dynamic assumptions

Avoid:

- graph models that ignore real constraints
- unjustified edge weights
- routes that are mathematically valid but practically infeasible

## Simulation

Baseline options:

- deterministic scenario comparison
- small manual scenario table
- simplified expected-value calculation

Main model options:

- Monte Carlo simulation
- discrete-event simulation
- agent-based simulation
- stochastic process modeling
- scenario analysis

Use when:

- the output is a stochastic distribution, scenario result, repeated process outcome, or policy comparison under uncertainty.

Check:

- random seed
- number of trials
- parameter distributions
- confidence intervals or summary statistics
- scenario assumptions

Avoid:

- simulation without statistical summary
- hidden random assumptions
- too few trials
- unverifiable scenario generation

## Data analysis

Baseline options:

- descriptive statistics
- simple correlation table
- distribution summary

Main model options:

- correlation analysis
- hypothesis testing
- factor analysis
- principal component analysis
- regression explanation
- exploratory visualization

Use when:

- the output is pattern discovery, factor identification, descriptive insight, or relationship analysis.

Check:

- correlation vs causation
- data quality
- unit consistency
- figure relevance
- connection to later modeling steps

Avoid:

- decorative figures without modeling purpose
- causal claims from correlation alone
- analysis that does not feed any subquestion

# Rules

- Do not generate code.
- Do not clean data.
- Do not write paper text.
- Do not fabricate data, metrics, or results.
- Do not choose final methods without considering data availability.
- Do not skip baselines.
- Do not overfit the method plan to a famous excellent paper.
- Do not recommend a complex method unless it has a clear role.
- Do not ignore implementation time and team capability.
- Do not ignore robustness or sensitivity requirements.
- Mark optional methods as optional.
- Mark high-risk methods as high-risk.
- Preserve uncertainty inherited from previous stages.

# Verification

Before handing off, verify:

- Every subquestion has 2-4 candidate schemes or an explicit reason why fewer are defensible.
- Every subquestion has one recommended execution route.
- Every subquestion has a baseline model.
- Every subquestion has a main model.
- Improved models are justified or explicitly omitted.
- Model choices match problem type, required output, and data conditions.
- Required input data is listed for each model.
- Expected outputs, tables, and figures are listed.
- Validation checks are defined.
- Robustness or sensitivity checks are defined.
- Rejected methods are explained.
- The next skill is appropriate, usually `data-auditor-cleaner`.

# Failure modes

Stop and report a blocker if:

- No validated problem classification exists.
- Required outputs are unknown.
- Data availability is too unclear to select feasible methods.
- The user requests code generation before a method plan exists.
- Multiple method choices would lead to materially different workflows and the tradeoff cannot be resolved from available information.
- The only plausible methods require data that is unavailable or forbidden by contest rules.

# Stop conditions

This skill must stop instead of guessing when:

- Selecting a model would require inventing data fields, labels, constraints, or evaluation metrics.
- A subquestion cannot be linked to a required output.
- A model choice depends on an unavailable attachment.
- The user asks for final numerical results, code, or paper text before method planning is complete.

When stopping, output:

- the blocker
- why it matters
- safe partial method choices if any
- missing information needed
- recommended next action

# Handoff

After producing a validated method plan, hand off to:

`data-auditor-cleaner`

The handoff should include:

- candidate schemes
- recommended scheme for each subquestion
- method plan
- required data fields
- expected cleaned data outputs
- expected result tables
- expected figures
- validation checks
- robustness checks
- rejected methods and reasons
- implementation notes

If no external or tabular data is needed and the method plan is fully specified, the workflow owner may route next to `model-code-analyzer`, but only after confirming that data requirements are satisfied.

# Examples

## Example 1: Evaluation problem

Input state:

- Q1 is classified as `evaluation`.
- Output required: ranking of several alternatives.
- Data: multiple indicators with mixed units.

Output:

```json
{
  "subproblem_methods": [
    {
      "id": "Q1",
      "problem_type": "evaluation",
      "candidate_schemes": [
        {
          "scheme_id": "Q1-S1",
          "name": "equal-weight scoring route"
        },
        {
          "scheme_id": "Q1-S2",
          "name": "entropy-weight TOPSIS route"
        }
      ],
      "recommended_scheme_id": "Q1-S2",
      "baseline_model": {
        "name": "equal-weight normalized scoring",
        "role": "transparent baseline ranking"
      },
      "main_model": {
        "name": "entropy-weight TOPSIS",
        "selection_reason": [
          "fits multi-indicator evaluation",
          "handles objective weighting",
          "produces comparable scores and ranking"
        ]
      },
      "improved_model": {
        "name": "weight perturbation stability analysis",
        "role": "not a new scoring model, but a robustness improvement"
      },
      "expected_figures_tables": [
        "indicator direction table",
        "weight table",
        "ranking table",
        "ranking sensitivity plot"
      ],
      "recommended_next_skill": "data-auditor-cleaner"
    }
  ]
}
```

## Example 2: Prediction problem

Input state:

- Q2 is classified as `prediction`.
- Output required: future demand values.
- Data: historical time series with limited length.

Output:

```json
{
  "subproblem_methods": [
    {
      "id": "Q2",
      "problem_type": "prediction",
      "candidate_schemes": [
        {
          "scheme_id": "Q2-S1",
          "name": "moving-average route"
        },
        {
          "scheme_id": "Q2-S2",
          "name": "ARIMA route"
        },
        {
          "scheme_id": "Q2-S3",
          "name": "feature-augmented regression route"
        }
      ],
      "recommended_scheme_id": "Q2-S2",
      "baseline_model": {
        "name": "last-value or moving-average baseline",
        "role": "simple reference forecast"
      },
      "main_model": {
        "name": "ARIMA or exponential smoothing",
        "selection_reason": [
          "fits short historical time series",
          "more explainable than deep learning",
          "supports forecast error analysis"
        ]
      },
      "improved_model": {
        "name": "feature-augmented regression",
        "condition": "use only if reliable explanatory variables exist"
      },
      "rejected_methods": [
        {
          "name": "BiLSTM",
          "reason": "data length is likely insufficient and implementation risk is high"
        }
      ],
      "validation_plan": [
        "train-test split",
        "MAE or RMSE",
        "residual check"
      ],
      "recommended_next_skill": "data-auditor-cleaner"
    }
  ]
}
```

## Example 3: Optimization problem

Input state:

- Q3 is classified as `optimization`.
- Output required: allocation plan under limited resources.
- Data: resource limits and demand estimates.

Output:

```json
{
  "subproblem_methods": [
    {
      "id": "Q3",
      "problem_type": "optimization",
      "candidate_schemes": [
        {
          "scheme_id": "Q3-S1",
          "name": "greedy allocation route"
        },
        {
          "scheme_id": "Q3-S2",
          "name": "integer linear programming route"
        },
        {
          "scheme_id": "Q3-S3",
          "name": "multi-objective optimization route"
        }
      ],
      "recommended_scheme_id": "Q3-S2",
      "baseline_model": {
        "name": "greedy allocation by demand priority",
        "role": "simple feasible reference plan"
      },
      "main_model": {
        "name": "integer linear programming",
        "selection_reason": [
          "decision variables are discrete",
          "constraints can be written explicitly",
          "final output must be an implementable allocation plan"
        ]
      },
      "improved_model": {
        "name": "multi-objective weighted optimization",
        "condition": "use if fairness, cost, and efficiency must be balanced"
      },
      "expected_artifacts": [
        "decision variable table",
        "objective function definition",
        "constraint list",
        "optimal allocation table",
        "sensitivity analysis on weights or resource limits"
      ],
      "recommended_next_skill": "data-auditor-cleaner"
    }
  ]
}
```

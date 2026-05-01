---
name: robustness-checker
description: Design and run robustness, sensitivity, error, and baseline comparison checks.
license: MIT
---

# Purpose

Design and run robustness, sensitivity, error, and baseline comparison checks for mathematical modeling contest solutions.

This skill tests whether model conclusions are stable enough to support paper claims. It compares baseline and main model results, perturbs key parameters or inputs, checks error behavior, identifies fragile assumptions, and records the boundary conditions under which conclusions remain valid.

This skill does not choose the original modeling route, rewrite model code broadly, fabricate experiment results, or write final paper sections.

# When to use

Use this skill:

- After `code-reviewer` has confirmed that modeling code is runnable or has clear remaining risks.
- After baseline and main model outputs exist.
- Before `figure-table-planner`.
- Before paper writing uses model conclusions.
- When the solution needs sensitivity analysis, baseline comparison, error analysis, or stability evidence.
- When final claims depend on parameter choices, weights, random seeds, sampling, constraints, or uncertain assumptions.

# Preconditions

The following should already exist or be provided:

- A validated method plan.
- Reviewed modeling code.
- Run instructions.
- Baseline outputs where applicable.
- Main model outputs.
- Cleaned data or approved non-tabular inputs.
- Data audit report and code review report if available.
- Known parameters, weights, thresholds, assumptions, constraints, and random seeds.

If model outputs do not exist, hand back to `code-reviewer`, `python-model-code-generator`, or `matlab-model-code-generator`.

If the main model has no baseline and no justified exception, hand back to `method-selector`.

# Inputs

Use or request:

- Method plan from `workspace/problem/` or equivalent.
- Reviewed scripts under `workspace/code/`.
- Cleaned data under `workspace/data/data_clean/`.
- Model outputs under `workspace/results/`.
- Figures under `workspace/figures/`, if already generated.
- Code review report.
- Data audit report.
- Baseline result files.
- Main model result files.
- User-specified robustness requirements or contest scoring criteria if available.

# Workflow

1. Identify claims that need support.
   - Extract key conclusions from result summaries if available.
   - Identify which outputs will likely become paper claims.
   - Do not test irrelevant quantities.

2. Map checks to model type.
   - Evaluation models usually need weight sensitivity, normalization sensitivity, and ranking stability.
   - Prediction models usually need error metrics, split sensitivity, residual checks, and extrapolation limits.
   - Optimization models usually need constraint perturbation, parameter sensitivity, feasibility checks, and baseline comparison.
   - Simulation models usually need random seed stability, trial count sensitivity, and distribution summaries.
   - Mechanism models usually need parameter sensitivity, boundary condition checks, and unit or scale consistency.

3. Compare baseline and main model.
   - Confirm that the main model improves, clarifies, or complements the baseline.
   - If the main model does not outperform the baseline, report that honestly.
   - Do not hide baseline failure or main model weakness.

4. Design robustness checks.
   - Perturb key parameters, weights, thresholds, or assumptions.
   - Perturb data where meaningful, such as noise injection, sample deletion, resampling, or missing-value strategy comparison.
   - Test extreme but reasonable scenarios.
   - Test random seed stability when randomness exists.
   - Check feasibility under changed constraints for optimization tasks.

5. Run or specify checks.
   - Run checks when code and data are available.
   - If execution is not possible, produce an executable robustness plan and mark it as not yet run.
   - Do not invent numerical robustness results.

6. Summarize stability.
   - Separate stable findings from fragile findings.
   - State which conclusions are supported.
   - State which conclusions require caution.
   - State the boundary conditions under which conclusions are valid.

7. Save or recommend artifacts.
   - Save robustness reports under `workspace/results/`.
   - Save sensitivity tables under `workspace/results/`.
   - Save robustness figures under `workspace/figures/`.
   - Keep file names traceable to subquestions and checks.

8. Recommend next step.
   - If robustness evidence is sufficient, hand off to `figure-table-planner`.
   - If model or code issues are discovered, hand back to the relevant upstream skill.

# Outputs

Produce robustness artifacts such as:

- `workspace/results/robustness_report.md`
- `workspace/results/q1_sensitivity_table.csv`
- `workspace/results/q2_error_metrics.csv`
- `workspace/results/q3_constraint_sensitivity.csv`
- `workspace/figures/q1_weight_sensitivity.png`
- `workspace/figures/q2_residual_plot.png`
- `workspace/figures/q3_baseline_comparison.png`
- robustness summary
- supported conclusions
- fragile conclusions
- recommended next skill

# Output format

Prefer this JSON-compatible summary:

```json
{
  "robustness_summary": {
    "status": "passed_with_warnings",
    "checked_subquestions": [
      "Q1",
      "Q2",
      "Q3"
    ],
    "supported_conclusions": [
      "Q1 ranking remains stable under moderate weight perturbation."
    ],
    "fragile_conclusions": [
      "Q2 long-term forecast becomes unstable beyond the observed time range."
    ],
    "recommended_next_skill": "figure-table-planner"
  },
  "checks": [
    {
      "subquestion": "Q1",
      "check_type": "weight_sensitivity",
      "description": "Perturb indicator weights by plus or minus 10 percent.",
      "status": "completed",
      "artifact": "workspace/results/q1_weight_sensitivity.csv",
      "finding": "Top three alternatives remain unchanged."
    }
  ],
  "baseline_comparison": [
    {
      "subquestion": "Q1",
      "baseline": "equal-weight scoring",
      "main_model": "entropy-weight TOPSIS",
      "comparison_metric": "ranking stability and interpretability",
      "finding": "Main model provides differentiated weights while keeping major ranking patterns consistent."
    }
  ],
  "remaining_risks": [
    "Some indicator units still require final explanation in the paper."
  ],
  "generated_artifacts": [
    "workspace/results/robustness_report.md",
    "workspace/results/q1_weight_sensitivity.csv",
    "workspace/figures/q1_weight_sensitivity.png"
  ]
}
```

If a JSON block is too rigid for the situation, use a concise Markdown report with the same fields.

# Robustness check types

Use the following checks when appropriate. Select checks based on model type and claim risk.

## Baseline comparison

Use when:

- a baseline and main model both exist
- a paper claim says the main model is better, more stable, more accurate, or more useful

Check:

- whether outputs differ materially
- whether improvement is meaningful
- whether the main model adds interpretability, accuracy, feasibility, or stability
- whether the baseline already solves the task sufficiently

## Parameter sensitivity

Use when:

- results depend on parameters, thresholds, coefficients, decay rates, penalties, or hyperparameters

Check:

- small perturbations such as plus or minus 5 percent, 10 percent, or 20 percent
- monotonicity of output changes
- tipping points where conclusions change
- parameters that dominate results

## Weight sensitivity

Use when:

- evaluation, multi-objective optimization, or weighted scoring is used

Check:

- weight perturbation
- alternative weighting schemes
- ranking stability
- score stability
- whether top alternatives remain stable

## Data perturbation

Use when:

- results may depend on noisy, incomplete, or uncertain data

Check:

- noise injection
- missing-value strategy comparison
- sample deletion
- bootstrap or resampling if appropriate
- effect of outliers

## Error analysis

Use when:

- prediction, fitting, classification, or estimation is used

Check:

- MAE, RMSE, MAPE, accuracy, F1, residuals, confusion matrix, or task-appropriate metrics
- train-test split or cross-validation when feasible
- error distribution
- failure cases
- extrapolation limits

## Constraint perturbation

Use when:

- optimization constraints affect feasibility or final decisions

Check:

- resource limits
- budget limits
- capacity limits
- fairness or safety bounds
- feasibility under relaxed and tightened constraints

## Randomness stability

Use when:

- stochastic algorithms, simulations, random sampling, randomized initialization, or train-test splits are used

Check:

- fixed random seeds
- multiple seeds
- result variance
- trial count sensitivity
- confidence intervals or summary statistics

## Extreme scenario testing

Use when:

- the model may face boundary cases or policy scenarios

Check:

- high-demand case
- low-resource case
- worst-case cost
- best-case or optimistic case
- edge conditions allowed by the problem context

## Conclusion boundary

Use for all models.

Check:

- what conditions must hold for the conclusion to remain valid
- where the conclusion should not be generalized
- which assumptions are most important
- what data limitations constrain the interpretation

# Robustness rules

- Robustness checks must target claims, not random variables.
- Every major paper claim should have at least one supporting check or a stated limitation.
- Do not invent robustness results.
- Do not report a check as completed unless it was actually run or derived from existing artifacts.
- If a check is only planned, mark it as planned.
- Prefer simple, explainable checks over elaborate but fragile tests.
- Use baseline comparison whenever a main model claims improvement.
- For evaluation models, check ranking stability.
- For prediction models, report error metrics and residual or failure behavior.
- For optimization models, check feasibility and constraint sensitivity.
- For simulation models, check random seed and trial count stability.
- For mechanism models, check parameter and boundary-condition sensitivity.

# Rules

- Do not choose a new main model.
- Do not silently change the validated method plan.
- Do not fabricate numerical results.
- Do not hide unstable findings.
- Do not write final paper text.
- Do not perform final QA.
- Do not overwrite validated artifacts without permission.
- Do not treat decorative figures as robustness evidence.
- Do not claim robustness without checks.
- Mark incomplete checks explicitly.
- Mark fragile conclusions explicitly.
- Hand back to upstream skills when robustness failures reveal method, data, or code problems.

# Verification

Before handing off, verify:

- Baseline comparison exists where applicable.
- Major conclusions have supporting checks or stated limitations.
- Robustness checks match the model type.
- Completed checks have artifacts or clear derivations.
- Planned checks are not mislabeled as completed.
- Generated tables and figures are saved under correct workspace directories.
- Stable and fragile findings are separated.
- Remaining risks are listed.
- The next skill is `figure-table-planner` if robustness evidence is usable.

# Failure modes

Stop and report a blocker if:

- Model outputs do not exist.
- Baseline outputs are missing without justification.
- Reviewed code is unavailable or not runnable.
- Required data is missing.
- The method plan does not define the output being tested.
- Robustness checks would require fabricating data or results.
- Results cannot be traced to scripts, data, or method-plan items.
- The user asks for paper conclusions before robustness evidence exists.

# Stop conditions

This skill must stop instead of guessing when:

- A robustness result cannot be computed from available artifacts.
- The metric or claim to be tested is undefined.
- A required baseline is missing.
- A robustness failure reveals that the method plan is invalid.
- The code cannot be run safely or lacks required inputs.
- Continuing would require inventing numerical findings.

When stopping, output:

- the blocker
- why it matters
- affected subquestion or claim
- checks that can still be performed
- missing information or artifact needed
- recommended next action

# Handoff

If robustness evidence is usable, hand off to:

`figure-table-planner`

The handoff should include:

- robustness report path
- sensitivity tables
- baseline comparison results
- generated robustness figures
- supported conclusions
- fragile conclusions
- remaining risks
- recommended figures and tables

If robustness checks expose code issues, hand off to:

`code-reviewer`

If robustness checks expose method-plan issues, hand off to:

`method-selector`

If robustness checks expose data issues, hand off to:

`data-auditor-cleaner`

Do not hand off directly to `paper-section-writer` unless figure and table planning has already been completed.

# Examples

## Example 1: Evaluation robustness

Input state:

- Q1 uses entropy-weight TOPSIS.
- Equal-weight baseline exists.
- Ranking results exist.

Output:

```json
{
  "robustness_summary": {
    "status": "passed_with_warnings",
    "checked_subquestions": [
      "Q1"
    ],
    "supported_conclusions": [
      "The top-ranked alternatives remain stable under moderate weight perturbation."
    ],
    "fragile_conclusions": [
      "Middle-ranked alternatives are sensitive to weight changes."
    ],
    "recommended_next_skill": "figure-table-planner"
  },
  "checks": [
    {
      "subquestion": "Q1",
      "check_type": "weight_sensitivity",
      "description": "Perturb indicator weights by plus or minus 10 percent.",
      "status": "completed",
      "artifact": "workspace/results/q1_weight_sensitivity.csv",
      "finding": "Top three alternatives remain unchanged; ranks 5 to 8 change under several perturbations."
    }
  ]
}
```

## Example 2: Prediction robustness

Input state:

- Q2 predicts future demand.
- Baseline and main model forecasts exist.
- Train-test split results exist.

Output:

```json
{
  "robustness_summary": {
    "status": "needs_caution",
    "checked_subquestions": [
      "Q2"
    ],
    "supported_conclusions": [
      "The main model improves short-term prediction error over the moving-average baseline."
    ],
    "fragile_conclusions": [
      "Long-term predictions are unstable and should not be overclaimed."
    ],
    "recommended_next_skill": "figure-table-planner"
  },
  "checks": [
    {
      "subquestion": "Q2",
      "check_type": "error_analysis",
      "status": "completed",
      "artifact": "workspace/results/q2_error_metrics.csv",
      "finding": "Short-term RMSE improves over baseline, but residuals increase near the end of the forecast horizon."
    }
  ]
}
```

## Example 3: Blocked by missing baseline

Input state:

- Main optimization result exists.
- No greedy or current-policy baseline exists.
- Method plan required a baseline.

Output:

```json
{
  "blocked_items": [
    "Baseline output is missing, so the main optimization result cannot be compared against a reference plan."
  ],
  "affected_subquestion": "Q3",
  "recommended_next_skill": "python-model-code-generator or matlab-model-code-generator",
  "recommended_next_action": "Generate and review the planned baseline output before robustness comparison."
}
```

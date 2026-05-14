---
name: robustness-checker
description: Design and run robustness, sensitivity, error, and baseline comparison checks.
license: MIT
---

# Purpose

Design and run robustness, sensitivity, error, and baseline comparison checks for mathematical modeling contest solutions.

This skill tests whether model conclusions are stable enough to support paper claims. It compares baseline and main model results, perturbs key parameters or inputs, checks error behavior, identifies fragile assumptions, and records the boundary conditions under which conclusions remain valid.

A key output is the per-subquestion robustness report (`robustness/Qx/qx_robustness_report.md`), which serves the paper materials chain: the report's findings are referenced by the final result analysis, incorporated into the solution package, and ultimately written into the paper's robustness section.

This skill does not choose the original modeling route, rewrite model code broadly, fabricate experiment results, or write final paper sections.

# When to use

Use this skill:

- After `code-reviewer` has confirmed that modeling code is runnable or has clear remaining risks.
- After baseline and main model outputs exist.
- After the final method has been selected (not during early experimentation rounds — basic error checks during experimentation are done by the code or experiment report).
- Before `figure-table-planner`.
- Before paper writing uses model conclusions.
- When the solution needs sensitivity analysis, baseline comparison, error analysis, or stability evidence.
- When final claims depend on parameter choices, weights, random seeds, sampling, constraints, or uncertain assumptions.

# Preconditions

The following should already exist or be provided:

- A validated candidate method pool.
- Final method explanation for the target subquestion (if the final method is locked; otherwise, basic checks only).
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

- Candidate method pool files.
- Final method explanation files (if available).
- Reviewed scripts under `code/`.
- Cleaned data under `workspace/data/data_clean/`.
- Model outputs under `results/Qx/experiments/`.
- Experiment reports under `results/Qx/experiments/roundN/`.
- Figures under `results/Qx/experiments/*/figures/`.
- Code review report.
- Data audit report.
- Baseline result files.
- Main model result files.
- User-specified robustness requirements or contest scoring criteria if available.

# Workflow

1. Identify claims that need support.
   - Extract key conclusions from the final result analysis or experiment reports.
   - Identify which outputs will likely become paper claims.
   - Do not test irrelevant quantities.

2. Map checks to model type.
   - Evaluation models → weight sensitivity, normalization sensitivity, ranking stability.
   - Prediction models → error metrics, split sensitivity, residual checks, extrapolation limits.
   - Optimization models → constraint perturbation, parameter sensitivity, feasibility checks, baseline comparison.
   - Simulation models → random seed stability, trial count sensitivity, distribution summaries.
   - Mechanism models → parameter sensitivity, boundary condition checks, unit or scale consistency.

3. Compare baseline and main model.
   - Confirm that the main model improves, clarifies, or complements the baseline.
   - If the main model does not outperform the baseline, report that honestly.
   - Do not hide baseline failure or main model weakness.

4. Design robustness checks.
   - Perturb key parameters, weights, thresholds, or assumptions.
   - Perturb data where meaningful (noise injection, sample deletion, resampling, missing-value strategy comparison).
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

7. Determine figure placement.
   - For each robustness figure, recommend: main paper (Type 3 论文图) or appendix (Type 4 附录图).
   - Criteria: if the result materially affects the main conclusion → main paper. If supplementary detail → appendix.

8. Produce per-subquestion robustness report.
   - Save to `robustness/Qx/qx_robustness_report.md`.
   - Keep the report self-contained and citable from the final result analysis and solution package.

9. Recommend next step.
   - If robustness evidence is sufficient, hand off to `figure-table-planner`.
   - If model or code issues are discovered, hand back to the relevant upstream skill.

# Outputs

Produce per-subquestion robustness artifacts:

- `robustness/Q1/q1_robustness_report.md`
- `robustness/Q2/q2_robustness_report.md`
- `robustness/Q3/q3_robustness_report.md`
- `robustness/Q4/q4_robustness_report.md`
- Sensitivity data files: `robustness/Qx/weight_sensitivity.csv`, `robustness/Qx/error_metrics.csv`, `robustness/Qx/constraint_sensitivity.csv`, etc.
- Robustness figures: `robustness/Qx/figures/weight_sensitivity.png`, `robustness/Qx/figures/residual_plot.png`, `robustness/Qx/figures/baseline_comparison.png`, etc.

# Output format

Each `robustness/Qx/qx_robustness_report.md` must follow this structure:

```markdown
# Qx Robustness and Sensitivity Report

## 1. Summary

- **Status**: passed / passed_with_warnings / needs_caution / failed
- **Overall finding**: [one-sentence summary of stability]
- **Recommended next skill**: `figure-table-planner`

## 2. Claims Under Test

| # | Claim | Source | Importance |
|---|-------|--------|------------|
| 1 | [claim from final result analysis or experiment report] | [source artifact] | high / medium |

## 3. Baseline Comparison

### 3.1 Comparison Setup
- **Baseline**: [method name]
- **Main model**: [method name]
- **Comparison metric**: [metric name and formula]

### 3.2 Comparison Results

| Metric | Baseline | Main Model | Improvement | Meaningful? |
|--------|----------|------------|-------------|-------------|
| [metric] | [value] | [value] | [delta] | yes / no / marginal |

### 3.3 Baseline Comparison Verdict
- [Whether the main model meaningfully improves over baseline]
- [If not, honest assessment of why the main model is still chosen]

## 4. Robustness Checks

### 4.1 [Check Type: e.g., Weight Sensitivity]

- **Description**: [what was perturbed, how, and by how much]
- **Perturbation range**: [e.g., ±5%, ±10%, ±20%]
- **Status**: completed / planned / not applicable
- **Artifact**: `robustness/Qx/weight_sensitivity.csv`
- **Figure**: `robustness/Qx/figures/weight_sensitivity.png`
- **Finding**: [specific finding]
- **Conclusion stability**: stable under this perturbation / fragile / breaks
- **Figure recommendation**: main paper / appendix / not needed

### 4.2 [Next Check Type]
[same structure]

## 5. Supported Conclusions

These conclusions are supported by robustness evidence:

| # | Conclusion | Supporting Check | Confidence |
|---|-----------|-----------------|------------|
| 1 | [conclusion] | [check name + artifact] | high / medium |

## 6. Fragile Conclusions

These conclusions require caution or qualification:

| # | Conclusion | Why Fragile | Boundary Condition | Recommended Qualification |
|---|-----------|-------------|-------------------|--------------------------|
| 1 | [conclusion] | [which check revealed fragility] | [under what conditions it holds] | [how to qualify in the paper] |

## 7. Conclusion Boundaries

For each major conclusion, state the conditions under which it remains valid:

| Conclusion | Valid When | May Fail When | Paper Guidance |
|-----------|------------|--------------|----------------|
| [conclusion] | [conditions] | [conditions] | [how to frame in paper] |

## 8. Figure Placement Recommendations

| Figure | Content | Recommended Placement | Reason |
|--------|---------|----------------------|--------|
| `robustness/Qx/figures/weight_sensitivity.png` | Weight perturbation effect on ranking | Main paper (Type 3) | Directly supports key claim about ranking stability |
| `robustness/Qx/figures/full_sensitivity_curves.png` | All indicators, full perturbation | Appendix (Type 4) | Supplementary detail |

## 9. Remaining Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| [risk not covered by current checks] | [potential impact on conclusions] | [how to address or acknowledge in paper] |

## 10. Generated Artifacts

- `robustness/Qx/qx_robustness_report.md`
- `robustness/Qx/weight_sensitivity.csv`
- `robustness/Qx/figures/weight_sensitivity.png`

## 11. Handoff

- **Next skill**: `figure-table-planner`
- **Supported conclusions** (for solution package): [list]
- **Fragile conclusions** (for solution package): [list]
```

# Robustness check types

## Baseline comparison
Use when: a baseline and main model both exist; a paper claim says the main model is better.

Check:
- Whether outputs differ materially.
- Whether improvement is meaningful.
- Whether the main model adds interpretability, accuracy, feasibility, or stability.
- Whether the baseline already solves the task sufficiently.

## Parameter sensitivity
Use when: results depend on parameters, thresholds, coefficients, decay rates, penalties, or hyperparameters.

Check:
- Small perturbations (±5%, ±10%, ±20%).
- Monotonicity of output changes.
- Tipping points where conclusions change.
- Parameters that dominate results.

## Weight sensitivity
Use when: evaluation, multi-objective optimization, or weighted scoring is used.

Check:
- Weight perturbation.
- Alternative weighting schemes.
- Ranking stability (do top-K alternatives stay the same?).
- Score stability.

## Data perturbation
Use when: results may depend on noisy, incomplete, or uncertain data.

Check:
- Noise injection.
- Missing-value strategy comparison.
- Sample deletion.
- Bootstrap or resampling if appropriate.
- Effect of outliers.

## Error analysis
Use when: prediction, fitting, classification, or estimation is used.

Check:
- MAE, RMSE, MAPE, accuracy, F1, residuals, confusion matrix.
- Train-test split or cross-validation.
- Error distribution.
- Failure cases.
- Extrapolation limits.

## Constraint perturbation
Use when: optimization constraints affect feasibility or final decisions.

Check:
- Resource limits.
- Budget/capacity limits.
- Fairness or safety bounds.
- Feasibility under relaxed and tightened constraints.

## Randomness stability
Use when: stochastic algorithms, simulations, random sampling, or train-test splits are used.

Check:
- Fixed random seeds.
- Multiple seeds.
- Result variance.
- Trial count sensitivity.
- Confidence intervals or summary statistics.

## Extreme scenario testing
Use when: the model may face boundary cases or policy scenarios.

Check:
- High-demand case.
- Low-resource case.
- Worst-case cost.
- Best-case or optimistic case.
- Edge conditions allowed by the problem context.

## Conclusion boundary (always required)
Check for ALL models:
- What conditions must hold for the conclusion to remain valid.
- Where the conclusion should not be generalized.
- Which assumptions are most important.
- What data limitations constrain the interpretation.

# Rules

- Robustness checks must target claims, not random variables.
- Every major paper claim should have at least one supporting check or a stated limitation.
- Do not invent robustness results.
- Do not report a check as completed unless actually run or derived from existing artifacts.
- If a check is only planned, mark it as planned.
- Prefer simple, explainable checks over elaborate but fragile tests.
- Use baseline comparison whenever a main model claims improvement.
- For evaluation models, check ranking stability.
- For prediction models, report error metrics and residual or failure behavior.
- For optimization models, check feasibility and constraint sensitivity.
- For simulation models, check random seed and trial count stability.
- For mechanism models, check parameter and boundary-condition sensitivity.
- Always include a conclusion boundary section.
- Do not choose a new main model.
- Do not silently change the validated method plan.
- Do not fabricate numerical results or hide unstable findings.
- Do not write final paper text or perform final QA.
- Mark incomplete checks and fragile conclusions explicitly.
- Output per-subquestion reports (not one global report).

# Verification

Before handing off, verify:

- Per-subquestion robustness report exists for every subquestion that has a final method.
- Baseline comparison exists where applicable.
- Major conclusions have supporting checks or stated limitations.
- Robustness checks match the model type.
- Completed checks have artifacts or clear derivations.
- Planned checks are not mislabeled as completed.
- Generated tables and figures are saved under correct `robustness/Qx/` directories.
- Stable and fragile findings are separated.
- Conclusion boundaries are stated.
- Figure placement recommendations (main paper vs appendix) are provided.
- Remaining risks are listed.
- The next skill is `figure-table-planner`.

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

If robustness evidence is usable:
→ `figure-table-planner`
— with the per-subquestion robustness report, sensitivity tables, generated figures, supported conclusions, fragile conclusions, figure placement recommendations, and remaining risks.

If robustness checks expose code issues:
→ `code-reviewer`

If robustness checks expose method-plan issues:
→ `method-selector`

If robustness checks expose data issues:
→ `data-auditor-cleaner`

Do not hand off directly to `paper-section-writer` unless figure and table planning has already been completed.

# Relationship to the Paper Materials Chain

The robustness report is a link in the paper materials chain:

```
robustness/Qx/qx_robustness_report.md
    ↓ (referenced by)
results/Qx/reports/qx_final_result_analysis.md
    ↓ (incorporated into)
results/Qx/reports/qx_solution_package_for_writer.md
    ↓ (written into)
paper/sections/robustness.tex
```

The solution-package-builder should reference this report. The final result analysis should cite its findings. The paper robustness section should be based on it.

# Examples

## Example 1: Evaluation robustness (Q1)

Input state:
- Q1 final method: entropy-TOPSIS (M2).
- Equal-weight baseline (M1) exists.
- Ranking results exist.

Output (`robustness/Q1/q1_robustness_report.md` excerpt):

```markdown
# Q1 Robustness and Sensitivity Report

## 1. Summary
- **Status**: passed_with_warnings
- **Overall finding**: Top-3 ranking is stable under moderate (±10%) weight perturbation. Middle-ranked alternatives (ranks 4-7) show some sensitivity.
- **Recommended next skill**: `figure-table-planner`

## 3. Baseline Comparison

| Metric | M1 Equal-Weight | M2 Entropy-TOPSIS | Improvement | Meaningful? |
|--------|----------------|-------------------|-------------|-------------|
| Top-3 consistency | N/A (reference) | Same top 3 as M1 | — | M2 confirms M1's top picks |
| Score differentiation (std) | 0.08 | 0.15 | +88% | Yes — M2 differentiates better |

### 3.3 Baseline Comparison Verdict
M2 improves score differentiation over M1 while maintaining the same top-3 ranking, confirming that equal-weight ranking was reasonable but M2 adds useful discrimination for middle ranks.

## 6. Fragile Conclusions

| Conclusion | Why Fragile | Boundary Condition | Recommended Qualification |
|-----------|-------------|-------------------|--------------------------|
| "City D ranks 5th" | Rank changes under ±15% weight perturbation | Stable within ±10% perturbation | "City D ranks 5th under current weights; this position is moderately sensitive to weight assumptions." |

## 8. Figure Placement Recommendations

| Figure | Recommended Placement | Reason |
|--------|----------------------|--------|
| `weight_sensitivity.png` | Main paper (Type 3) | Supports key stability claim |
| `full_sensitivity_curves.png` | Appendix (Type 4) | Supplementary detail |
```

## Example 2: Prediction robustness (Q2)

```markdown
## 1. Summary
- **Status**: needs_caution
- **Overall finding**: Short-term predictions (1-3 months) show meaningful improvement over baseline. Long-term predictions (6+ months) become unstable.

## 5. Supported Conclusions
| Conclusion | Supporting Check | Confidence |
|-----------|-----------------|------------|
| "ARIMA improves short-term RMSE by 35% over moving average baseline." | Error analysis (1-3 month horizon) | High |

## 6. Fragile Conclusions
| Conclusion | Why Fragile | Recommended Qualification |
|-----------|-------------|--------------------------|
| "Future demand will be X in month 12." | Long-term forecast error grows exponentially | "The model projects X for month 12, but uncertainty increases substantially beyond a 3-month horizon." |
```

## Example 3: Blocked by missing baseline

```json
{
  "blocked_items": [
    "Baseline output is missing for Q3. The main optimization result cannot be compared against a reference plan."
  ],
  "affected_subquestion": "Q3",
  "recommended_next_skill": "python-model-code-generator or matlab-model-code-generator",
  "recommended_next_action": "Generate and review the planned baseline output before robustness comparison."
}
```

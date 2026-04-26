---
name: code-reviewer
description: Review, debug, simplify, and verify mathematical modeling code.
license: MIT
---

# Purpose

Review, debug, simplify, and verify code used in a mathematical modeling contest workflow.

This skill checks whether generated modeling code is runnable, reproducible, consistent with the validated method plan, and safe to use for producing result artifacts. It may make minimal fixes when needed, but it must not change the modeling route or silently alter assumptions.

This skill does not select models, clean raw data, invent results, perform full robustness analysis, write paper sections, or approve final submission.

# When to use

Use this skill:

- After `model-code-generator` has produced modeling scripts.
- When code fails to run or produces unexpected outputs.
- When generated scripts need to be checked before producing final results.
- When paths, data fields, dimensions, formulas, seeds, or output artifacts need verification.
- Before `robustness-checker`.

# Preconditions

The following should already exist or be provided:

- A validated method plan.
- Generated modeling scripts under `workspace/scripts/` or equivalent.
- Cleaned data under `workspace/data_clean/`, if the code requires data.
- Expected result artifacts from `model-code-generator`.
- Run instructions or intended execution command.
- Data audit report and field mapping if data is used.

If the generated code is missing, hand back to `model-code-generator`.

If cleaned data is missing, hand back to `data-auditor-cleaner`.

# Inputs

Use or request:

- Script paths under `workspace/scripts/`.
- Cleaned data paths under `workspace/data_clean/`.
- Method plan and expected outputs.
- Data field mapping and data audit report.
- Current error messages, stack traces, logs, or failed outputs if available.
- Expected result paths under `workspace/results/`.
- Expected figure paths under `workspace/figures/`.
- Preferred language or runtime environment if relevant.

# Workflow

1. Inspect the code purpose.
   - Identify which subquestion each script supports.
   - Identify whether it implements a baseline, main model, improved model, or utility step.
   - Compare the script intent with the method plan.

2. Check execution readiness.
   - Verify imports and dependencies.
   - Verify relative paths.
   - Verify cleaned data inputs.
   - Verify output directories and file names.
   - Verify run instructions.

3. Check data and shape consistency.
   - Confirm required fields exist in the cleaned data.
   - Check column names, data types, missing values, and expected units where relevant.
   - Check array, matrix, table, and vector dimensions.
   - Check train-test splits and feature-target separation for predictive tasks.

4. Check numerical and modeling safety.
   - Check division by zero.
   - Check `NaN`, `inf`, and invalid operations.
   - Check random seed usage.
   - Check parameter ranges.
   - Check convergence or stopping criteria where applicable.
   - Check whether formulas match the method plan.

5. Check artifact generation.
   - Verify result tables are saved under `workspace/results/`.
   - Verify figures are saved under `workspace/figures/`.
   - Verify intermediate outputs needed for explanation are saved.
   - Verify file names clearly indicate subquestion and model role.

6. Make minimal fixes if needed.
   - Fix syntax errors, path errors, import errors, field name mismatches, shape issues, and obvious runtime blockers.
   - Keep changes surgical and traceable.
   - Do not perform broad refactors unless necessary for correctness.
   - Do not change the model route.

7. Produce a review report.
   - State what was checked.
   - State what was changed.
   - State how to run the code.
   - State remaining risks.
   - Recommend the next skill.

# Outputs

Produce reviewed code artifacts and a review summary such as:

- corrected scripts under `workspace/scripts/`
- updated run instructions
- code review report
- list of fixed issues
- list of remaining risks
- expected result and figure paths
- recommended next skill

# Output format

Prefer this JSON-compatible summary:

```json
{
  "code_review_summary": {
    "reviewed_scripts": [
      "workspace/scripts/model_q1.py"
    ],
    "status": "passed_with_warnings",
    "fixed_issues": [
      {
        "type": "path_error",
        "file": "workspace/scripts/model_q1.py",
        "change": "Replaced absolute input path with workspace/data_clean/indicator_data.csv."
      }
    ],
    "remaining_risks": [
      "Indicator directions must still be verified against the method plan."
    ],
    "run_instructions": [
      "python workspace/scripts/model_q1.py"
    ],
    "expected_outputs": [
      "workspace/results/q1_main_results.csv",
      "workspace/figures/q1_ranking_comparison.png"
    ],
    "recommended_next_skill": "robustness-checker"
  }
}
```

If a JSON block is too rigid for the situation, use a concise Markdown report with the same fields.

# Review checklist

Check at least the following:

## Structural checks

- script maps to a specific subquestion
- script maps to baseline, main, or improved model
- code follows the validated method plan
- code does not silently change the selected model
- functions or sections are understandable
- unnecessary complexity is avoided

## Runtime checks

- imports are available or reasonable
- paths are relative where practical
- required input files exist or are requested
- output directories are used correctly
- run instructions are provided
- scripts can be executed in a clear order

## Data checks

- cleaned data is used instead of raw data
- required columns exist
- field names match the data audit mapping
- data types are compatible with operations
- missing values are handled or flagged
- array and matrix dimensions are valid
- train-test split avoids leakage where relevant

## Numerical checks

- no avoidable division by zero
- no uncontrolled `NaN` or `inf`
- random seeds are fixed when randomness is used
- parameter ranges are reasonable
- convergence or iteration limits are defined where needed
- objective functions and metrics are computed consistently

## Modeling consistency checks

- baseline output is generated when required
- main model implements the selected method
- improved model is optional and clearly separated
- formulas and variables correspond to the method plan
- validation outputs are available or planned
- generated figures support expected model outputs

## Artifact checks

- scripts are under `workspace/scripts/`
- result tables are under `workspace/results/`
- figures are under `workspace/figures/`
- important intermediate results are saved
- file names are traceable to subquestions and model roles
- no raw data files are overwritten

# Fixing rules

- Make the smallest necessary change.
- Preserve existing style where practical.
- Fix only issues connected to the requested code review.
- Remove only unused imports or variables introduced by the fix.
- Do not refactor unrelated code.
- Do not optimize for cleverness.
- Do not hide assumptions inside code.
- Do not change the mathematical model unless the method plan is impossible to implement.
- If a model-plan issue is found, report it and hand back to `method-selector`.
- If a data-readiness issue is found, report it and hand back to `data-auditor-cleaner`.

# Rules

- Do not fabricate data, labels, metrics, or outputs.
- Do not change the modeling route silently.
- Do not convert placeholder outputs into fake results.
- Do not write paper text.
- Do not perform final QA.
- Do not approve final submission.
- Do not overwrite raw data.
- Do not make broad style rewrites.
- Do not add heavy dependencies unless required and justified.
- Do not claim the code is correct unless run instructions and output expectations are clear.
- Mark remaining risks explicitly.

# Verification

Before handing off, verify:

- Reviewed scripts match the method plan.
- Scripts can be run or have clear blockers.
- Input paths point to cleaned data or approved non-tabular inputs.
- Output paths are under the correct workspace directories.
- Baseline outputs exist or are explicitly blocked.
- Main model outputs exist or are explicitly blocked.
- Randomness is controlled where relevant.
- The code does not write into `workspace/data_raw/`.
- Fixed issues are listed.
- Remaining risks are listed.
- The next skill is `robustness-checker` if code is usable.

# Failure modes

Stop and report a blocker if:

- No script is available to review.
- The method plan is unavailable.
- Required cleaned data is unavailable.
- Required fields are missing.
- Runtime errors cannot be fixed without changing the model route.
- The code depends on unavailable heavy dependencies.
- The code output cannot be linked to any subquestion.
- The user asks for paper claims before verified outputs exist.

# Stop conditions

This skill must stop instead of guessing when:

- A fix would change the mathematical meaning of the model.
- A fix requires inventing data, labels, parameters, or results.
- A script cannot be mapped to the method plan.
- The data schema is too ambiguous to repair safely.
- The code would overwrite validated artifacts without permission.
- The error indicates that the method plan is infeasible.

When stopping, output:

- the blocker
- why it matters
- affected script or subquestion
- safe partial fixes if any
- missing information needed
- recommended next action

# Handoff

If reviewed code is usable, hand off to:

`robustness-checker`

The handoff should include:

- reviewed script paths
- run instructions
- result output paths
- figure output paths
- fixed issue list
- remaining risk list
- validation outputs if available

If code is blocked by data issues, hand off to:

`data-auditor-cleaner`

If code is blocked by method-plan issues, hand off to:

`method-selector`

Do not hand off directly to `paper-section-writer` or `quality-assurance-auditor`.

# Examples

## Example 1: Path and output fix

Input state:

- `workspace/scripts/model_q1.py` uses an absolute local input path.
- Method plan expects output under `workspace/results/`.

Output:

```json
{
  "code_review_summary": {
    "status": "passed_with_warnings",
    "reviewed_scripts": [
      "workspace/scripts/model_q1.py"
    ],
    "fixed_issues": [
      {
        "type": "path_error",
        "change": "Replaced absolute local input path with a relative path under workspace/data_clean/."
      },
      {
        "type": "artifact_path",
        "change": "Changed result export path to workspace/results/q1_main_results.csv."
      }
    ],
    "remaining_risks": [
      "Input column units still need confirmation before paper interpretation."
    ],
    "recommended_next_skill": "robustness-checker"
  }
}
```

## Example 2: Blocked by method mismatch

Input state:

- Method plan specifies linear programming.
- Code implements a genetic algorithm without explanation.

Output:

```json
{
  "blocked_items": [
    "The script implements a genetic algorithm, but the validated method plan specifies linear programming."
  ],
  "affected_script": "workspace/scripts/model_q3.py",
  "recommended_next_skill": "method-selector",
  "recommended_next_action": "Confirm whether the method plan should be revised or the code should be regenerated according to the original linear programming plan."
}
```

## Example 3: Blocked by missing field

Input state:

- Script requires column `demand`.
- Cleaned data does not contain `demand`.

Output:

```json
{
  "blocked_items": [
    "The script requires the field demand, but the cleaned data does not contain it."
  ],
  "affected_script": "workspace/scripts/model_q2.py",
  "recommended_next_skill": "data-auditor-cleaner",
  "recommended_next_action": "Recheck field mapping and confirm whether a demand field exists under another name."
}
```

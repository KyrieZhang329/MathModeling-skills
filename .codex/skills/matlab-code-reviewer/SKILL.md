---
name: matlab-code-reviewer
description: Review, debug, and verify MATLAB or Beita Tianyuan compatible modeling code against the validated method plan and expected artifacts.
license: MIT
---

# Purpose

Review MATLAB / 北太天元 compatible modeling code after scripts have been generated and routed by `code-reviewer`.

This skill checks whether `.m` scripts are runnable, consistent with the validated method plan, compatible with MATLAB / 北太天元 constraints, and able to produce traceable result and figure artifacts.

This skill may make minimal code fixes when needed, but it must not change the modeling route or silently alter assumptions.

This skill does not select models, clean raw data, invent results, write paper sections, or approve final submission.

# When to use

Use this skill:

- After `code-reviewer` routes MATLAB scripts to `matlab-code-reviewer`.
- When `implementation.target = matlab`.
- When generated scripts are `.m` files.
- When the contest requires MATLAB / 北太天元 compatibility.
- Before using MATLAB-generated outputs for robustness checks, figures, or paper writing.

# Preconditions

The following should already exist or be provided:

- A validated method plan.
- `implementation.target = matlab`.
- MATLAB scripts under `workspace/code/matlab/`.
- Cleaned data under `workspace/data/data_clean/`, if data is required.
- Data audit report and field mapping if data is used.
- Expected result paths under `workspace/results/`.
- Expected figure paths under `workspace/figures/`.
- Run instructions or intended script order.
- Runtime notes, especially MATLAB / 北太天元 compatibility requirements.

If generated scripts are missing, hand back to `matlab-model-code-generator`.

If cleaned data is missing, hand back to `data-auditor-cleaner`.

If `implementation.target` is not `matlab`, hand back to `code-reviewer`.

# Inputs

Use or request:

- `workspace/problem/method-selector/method_plan.json`
- MATLAB scripts under `workspace/code/matlab/`
- cleaned data under `workspace/data/data_clean/`
- `workspace/data/data_report.md`, if available
- field mapping from `data-auditor-cleaner`
- expected output files
- expected figure files
- runtime notes such as:
  - `beita-tianyuan-compatible`
  - `avoid-heavy-toolboxes`
  - `prefer-basic-matrix-and-table-operations`
  - `avoid-live-scripts`
  - `avoid-app-designer`
  - `save-portable-artifacts`
- error messages, logs, screenshots, or failed outputs if available

# Workflow

1. Confirm MATLAB review target.
   - Confirm `implementation.target = matlab`.
   - Confirm scripts are `.m` files.
   - Confirm scripts are under `workspace/code/matlab/` or clearly documented otherwise.
   - Preserve MATLAB / 北太天元 runtime notes.

2. Map scripts to the method plan.
   - Identify which subquestion each script supports.
   - Identify whether each script implements a baseline, main model, improved model, or helper procedure.
   - Check that every required baseline and main model script exists.
   - Flag scripts that cannot be mapped to the method plan.

3. Check MATLAB script structure.
   - Prefer plain `.m` scripts.
   - Reject Live Script-only content.
   - If a file defines a primary function, verify the function name matches the file name.
   - Check that helper functions are either local functions or saved in clearly named `.m` files.
   - Check that script execution order is clear.

4. Check paths and artifacts.
   - Confirm cleaned data is read from `workspace/data/data_clean/`.
   - Confirm raw data under `workspace/data/data_raw/` is not modified.
   - Confirm results are saved under `workspace/results/`.
   - Confirm figures are saved under `workspace/figures/`.
   - Check that outputs are saved to portable formats such as `.csv`, `.mat`, and `.png`.
   - Flag scripts that only print or display results without saving them.

5. Check data-field consistency.
   - Verify `readtable`, `readmatrix`, `load`, or equivalent input operations use expected files.
   - Verify table variable names match the data audit field mapping.
   - Verify units and indicator directions are consistent with the method plan.
   - Verify missing values are handled or explicitly blocked.
   - Verify rows and columns are not silently dropped.

6. Check numerical correctness risks.
   - Check matrix and vector dimensions.
   - Check row-vector and column-vector consistency.
   - Check 1-based indexing.
   - Check division by zero.
   - Check `NaN`, `Inf`, or invalid numeric operations.
   - Check objective functions, constraints, weights, and formulas against the method plan.
   - Check convergence or stopping criteria where relevant.

7. Check randomness and reproducibility.
   - If randomness is used, verify `rng(seed)` is set.
   - Verify stochastic outputs are saved.
   - Verify repeated trials are documented if required by the method plan.

8. Check MATLAB / 北太天元 compatibility.
   - Prefer basic MATLAB-compatible syntax.
   - Flag heavy toolbox dependencies.
   - Flag unsupported or high-risk features such as App Designer, GUI code, Simulink dependencies, parallel toolbox, deep learning toolbox, symbolic toolbox, or version-specific functions.
   - If a function may not be available in 北太天元, recommend a simpler alternative or mark compatibility as a risk.

9. Make minimal fixes if needed.
   - Fix syntax errors, path errors, output saving errors, simple dimension mismatches, missing directory creation, and obvious variable-name mismatches.
   - Keep changes surgical and traceable.
   - Do not refactor broadly.
   - Do not change the mathematical model.

10. Produce a review report.
    - State reviewed scripts.
    - State fixed issues.
    - State unresolved risks.
    - State run instructions.
    - State expected outputs.
    - Recommend the next skill.

# Outputs

Produce reviewed MATLAB code artifacts and a review summary such as:

- corrected `.m` scripts under `workspace/code/matlab/`
- `workspace/code/code-review/matlab_review_summary.md`
- updated run instructions
- fixed issue list
- remaining compatibility risks
- expected result paths
- expected figure paths
- recommended next skill

# Output format

Prefer this JSON-compatible summary:

```json
{
  "matlab_code_review_summary": {
    "implementation_target": "matlab",
    "status": "passed_with_warnings",
    "reviewed_scripts": [
      "workspace/code/matlab/Q1/q1_baseline.m",
      "workspace/code/matlab/Q1/q1_main.m"
    ],
    "fixed_issues": [
      {
        "type": "path_error",
        "file": "workspace/code/matlab/Q1/q1_main.m",
        "change": "Replaced an absolute input path with fullfile('workspace','data_clean','clean_data.csv')."
      }
    ],
    "remaining_risks": [
      "北太天元 compatibility should be checked if the local environment lacks writematrix."
    ],
    "run_instructions": [
      "Run workspace/code/matlab/Q1/q1_baseline.m",
      "Run workspace/code/matlab/Q1/q1_main.m"
    ],
    "expected_outputs": [
      "workspace/results/q1_baseline_results.csv",
      "workspace/results/q1_main_results.csv",
      "workspace/figures/q1_ranking.png"
    ],
    "markdown_summary": "workspace/code/code-review/matlab_review_summary.md",
    "recommended_next_skill": "robustness-checker"
  }
}
```

If JSON is too rigid, use a concise Markdown report with the same fields.

# MATLAB / 北太天元 review checklist

Check at least the following.

## Script structure

- script is a plain `.m` file
- script is not a Live Script
- file name is clear and maps to a subquestion
- primary function name matches file name if a function is used
- helper functions are local or clearly named
- execution order is documented
- `run_all.m` exists only if useful

## Method consistency

- script maps to a method-plan item
- baseline script exists when required
- main model implements the selected method
- optional improved model is clearly separated
- formulas match the method plan
- variables match the method plan where practical
- objective functions and constraints are consistent
- rejected methods are not accidentally implemented

## Data and table handling

- cleaned data is used
- raw data is not overwritten
- `readtable`, `readmatrix`, `load`, or equivalent input calls use expected paths
- table field names match the data report
- missing values are handled or reported
- units and indicator directions are respected
- no rows or columns are silently dropped

## Matrix and numeric checks

- matrix dimensions are compatible
- row and column vectors are used consistently
- indexing uses MATLAB 1-based indexing correctly
- no avoidable division by zero
- no uncontrolled `NaN` or `Inf`
- weights sum or normalize correctly when required
- constraints are represented correctly
- convergence or iteration limits are defined where needed

## Reproducibility

- `rng(seed)` is used when randomness exists
- seed value is documented
- stochastic outputs are saved
- repeated simulations are summarized if required

## Paths and outputs

- relative paths are used where practical
- no hard-coded local absolute paths
- output directories are created if needed
- result tables are saved under `workspace/results/`
- figures are saved under `workspace/figures/`
- `.mat` files are saved when useful for later review
- outputs are not limited to command window display

## Compatibility

- code uses conservative MATLAB-compatible syntax
- heavy toolbox functions are avoided unless explicitly approved
- Live Scripts, App Designer, GUI code, and Simulink dependencies are avoided
- parallel toolbox, deep learning toolbox, and symbolic toolbox are avoided unless explicitly approved
- version-specific functions are flagged
- 北太天元 compatibility risks are listed

# Fixing rules

- Make the smallest necessary change.
- Preserve the approved modeling route.
- Preserve existing code style where practical.
- Fix only issues related to review goals.
- Remove only unused imports, variables, or code created by the fix.
- Add directory creation only when needed for saving outputs.
- Add result saving when missing.
- Add `rng(seed)` when randomness exists and no seed is set.
- Do not rewrite the whole script for style.
- Do not replace the model with a different method.
- Do not hide assumptions inside code.
- Do not fabricate outputs.

# Rules

- Do not select or change the model.
- Do not clean raw data.
- Do not fabricate data, labels, metrics, outputs, logs, or results.
- Do not write paper text.
- Do not perform final QA.
- Do not approve final submission.
- Do not overwrite raw data.
- Do not make broad style refactors.
- Do not add heavy toolbox dependencies.
- Do not ignore MATLAB / 北太天元 compatibility notes.
- Do not claim the code is correct if scripts are not runnable or outputs are not traceable.
- Mark remaining risks explicitly.

# Verification

Before handing off, verify:

- `implementation.target = matlab`.
- Reviewed scripts are `.m` files.
- Reviewed scripts map to the method plan.
- Scripts use cleaned data or approved non-tabular inputs.
- No raw data file is overwritten.
- Baseline outputs exist or are explicitly blocked.
- Main model outputs exist or are explicitly blocked.
- Randomness is controlled where relevant.
- Results are saved under `workspace/results/`.
- Figures are saved under `workspace/figures/`.
- MATLAB / 北太天元 compatibility risks are listed.
- Fixed issues are documented.
- Remaining risks are documented.
- The next skill is `robustness-checker` if code is usable.

# Failure modes

Stop and report a blocker if:

- No `.m` script is available to review.
- The method plan is missing.
- `implementation.target` is not `matlab`.
- Required cleaned data is unavailable.
- Required fields are missing.
- Runtime errors cannot be fixed without changing the model.
- The code depends on unavailable or unsupported toolbox functions.
- The script output cannot be linked to any subquestion.
- The script writes to `workspace/data/data_raw/`.
- The script cannot produce required result artifacts.
- Fixing the script would require inventing data, labels, parameters, or results.

# Stop conditions

This skill must stop instead of guessing when:

- A fix would change the mathematical meaning of the model.
- A fix requires inventing data, labels, parameters, metrics, outputs, or results.
- A script cannot be mapped to the method plan.
- The data schema is too ambiguous to repair safely.
- The script language or implementation target conflicts with the route.
- MATLAB / 北太天元 compatibility cannot be maintained without changing the method.
- Continuing would hide a method, data, or compatibility problem.

When stopping, output:

- blocker
- why it matters
- affected script or subquestion
- safe partial fixes if any
- missing information needed
- recommended repair skill
- next action

# Handoff

If reviewed MATLAB code is usable, hand off to:

```
robustness-checker
```

The handoff should include:

- reviewed `.m` script paths
- run instructions
- result output paths
- figure output paths
- fixed issue list
- remaining risk list
- MATLAB / 北太天元 compatibility notes
- validation outputs if available

If blocked by missing or wrong generated scripts, hand back to:

```
matlab-model-code-generator
```

If blocked by method-plan issues, hand back to:

```
method-selector
```

If blocked by data issues, hand back to:

```
data-auditor-cleaner
```

If blocked by target or routing conflicts, hand back to:

```
code-reviewer
```

# Examples

## Example 1: Passed with path and saving fixes

Input state:

- `implementation.target = matlab`
- `workspace/code/matlab/Q1/q1_main.m` uses an absolute local path
- script displays ranking but does not save it

Output:

```json
{
  "matlab_code_review_summary": {
    "implementation_target": "matlab",
    "status": "passed_with_warnings",
    "reviewed_scripts": [
      "workspace/code/matlab/Q1/q1_main.m"
    ],
    "fixed_issues": [
      {
        "type": "path_error",
        "change": "Replaced absolute local input path with a relative path under workspace/data/data_clean/."
      },
      {
        "type": "missing_result_artifact",
        "change": "Added writetable call to save ranking results under workspace/results/."
      }
    ],
    "remaining_risks": [
      "北太天元 compatibility should be verified if writetable behavior differs in the contest environment."
    ],
    "expected_outputs": [
      "workspace/results/q1_main_results.csv"
    ],
    "recommended_next_skill": "robustness-checker"
  }
}
```

## Example 2: Blocked by target mismatch

Input state:

- This reviewer is invoked
- method plan says `implementation.target = python`

Output:

```json
{
  "blocked_items": [
    "matlab-code-reviewer was invoked, but implementation.target is python."
  ],
  "recommended_next_skill": "code-reviewer",
  "recommended_next_action": "Route the scripts through code-reviewer again and use the Python review branch."
}
```

## Example 3: Blocked by heavy toolbox dependency

Input state:

- `implementation.target = matlab`
- runtime notes include `beita-tianyuan-compatible`
- script depends on a heavy toolbox-specific solver
- no approved fallback method exists

Output:

```json
{
  "blocked_items": [
    "The script depends on a toolbox-specific solver that conflicts with the 北太天元 compatibility requirement."
  ],
  "affected_script": "workspace/code/matlab/Q3/q3_main.m",
  "recommended_next_skill": "method-selector",
  "recommended_next_action": "Revise the method plan to use a basic MATLAB-compatible solver or explicitly approve the dependency."
}
```

## Example 4: Blocked by matrix dimension issue

Input state:

- script implements weighted scoring
- score matrix has size `n x m`
- weight vector is `1 x k`, where `k != m`

Output:

```json
{
  "blocked_items": [
    "Weight vector length does not match the number of indicators."
  ],
  "affected_script": "workspace/code/matlab/Q1/q1_main.m",
  "recommended_next_skill": "matlab-model-code-generator",
  "recommended_next_action": "Regenerate or repair the script using the indicator count from the cleaned data and method plan."
}
```

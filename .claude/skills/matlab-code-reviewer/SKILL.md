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

6.5. **Constraint-direction sanity check (P2 — list for human review, do NOT auto-correct).**
    Inequality direction errors (writing `allocated ≥ capacity` when it should be `≤`) are semantic, not syntactic — the agent cannot verify physical correctness. But it can **surface every constraint for the user to scan**.

    For every constraint in the code:
    - Extract the line number, the inequality direction (`<=` / `>=` / `==`), the left-hand side variable name, and the right-hand side expression.
    - Format as a compact table in the review file:

    | File:line | Direction | LHS | RHS | Expected physical meaning |
    |---|---|---|---|---|
    | `code/matlab/Q3/q3_main.m:42` | `<=` | `total_allocated(i)` | `capacity(i)` | allocated ≤ capacity (resource upper bound) |
    | `code/matlab/Q3/q3_main.m:58` | `>=` | `allocated(i)` | `demand_min(i)` | allocated ≥ minimum demand (demand floor) |

    - Do NOT change inequality direction. The reviewer MUST:
      - Flag constraints whose direction looks counterintuitive.
      - Ask the user to scan the "Expected physical meaning" column and confirm.
    - Document this table in `code/matlab/Qx/reviews/qx_matlab_review.md` under a `## Constraint Direction Review` section.

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

10. **Produce a review report — MANDATORY substantive file on disk.**
    - State reviewed scripts.
    - **List ≥ 5 explicit pass items** (concrete checks that ran and passed, each citing file:line and what was verified). Generic statements ("looks good", "no issues") do NOT count.
    - List failed checks with file:line and suggested repair.
    - State fixed issues (what this skill changed surgically).
    - State unresolved risks.
    - State run instructions.
    - State expected outputs.
    - Recommend the next skill.
    - **If fewer than 5 explicit pass items can be listed**, the review is not substantive — return `status: not_run`. Do NOT pad with generic statements.

# Outputs

Produce reviewed MATLAB code artifacts and a **mandatory review file**:

- corrected `.m` scripts under `code/matlab/Qx/` (matching CLAUDE.md workspace convention).
- **`code/matlab/Qx/reviews/qx_matlab_review.md`** — REQUIRED. This is the artifact `completeness-auditor` will check for existence and pass-item count. Create the `reviews/` directory if missing.
- updated run instructions
- fixed issue list
- remaining compatibility risks
- expected result paths
- expected figure paths
- recommended next skill

(Legacy path `workspace/code/code-review/matlab_review_summary.md` is deprecated. New runs must write to `code/matlab/Qx/reviews/qx_matlab_review.md`.)

# Output format

The mandatory `code/matlab/Qx/reviews/qx_matlab_review.md` file MUST follow this template:

```markdown
# Qx MATLAB Code Review

> **Status**: passed / passed_with_warnings / failed
> **Reviewer**: matlab-code-reviewer
> **Date**: [timestamp]
> **Scripts reviewed**: [list of .m paths]
> **Compatibility target**: MATLAB / 北太天元

## Pass Items (≥ 5 REQUIRED — concrete, file:line cited)

1. ✅ `code/matlab/Q1/q1_main.m:8` reads cleaned data via `readtable(fullfile('workspace','data_clean','clean_data.csv'))`.
2. ✅ `code/matlab/Q1/q1_main.m:24` weight vector `w (1×6)` matches indicator count from data audit.
3. ✅ `code/matlab/Q1/q1_main.m:53` calls `writetable` to save ranking → output saved, not just displayed.
4. ✅ `rng(2026)` is set at line 6 → reproducibility ensured.
5. ✅ No App Designer / Live Script / Simulink dependencies → 北太天元 compatible.
6. ✅ ...

## Failed / Repaired Items

| # | File:line | Issue | Action taken | Status |
|---|-----------|-------|--------------|--------|
| 1 | code/matlab/Q1/q1_main.m:17 | absolute path | replaced with relative path | fixed |
| 2 | code/matlab/Q3/q3_main.m:42 | uses `fmincon` (Optimization Toolbox) | NOT fixed — requires method change | blocked → method-selector |

## Remaining Risks

- [risk 1 with file:line]

## Run Instructions

```
Run code/matlab/Q1/q1_baseline.m
Run code/matlab/Q1/q1_main.m
```

## Expected Outputs

- `results/Q1/experiments/round1/tables/ranking.csv`
- `results/Q1/experiments/round1/figures/q1_ranking.png`
- `results/Q1/experiments/round1/run_summary.json`

## Recommended Next Skill

- `result-report-generator` or `robustness-checker`
```

JSON summary is acceptable for handoff but the markdown file at `code/matlab/Qx/reviews/qx_matlab_review.md` is still required.

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
- Do not declare "completion" without writing `code/matlab/Qx/reviews/qx_matlab_review.md` to disk with ≥ 5 explicit pass items. A verbal "passed" is not a review — `completeness-auditor` will flag it as MISSING.
- Do not pad the pass list with generic statements. If you cannot list 5 concrete checks, report `status: not_run`.
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
- Results are saved under `results/Qx/experiments/roundN/` (per CLAUDE.md workspace convention).
- Figures are saved under `results/Qx/experiments/roundN/figures/`.
- MATLAB / 北太天元 compatibility risks are listed.
- Fixed issues are documented.
- Remaining risks are documented.
- **`code/matlab/Qx/reviews/qx_matlab_review.md` exists on disk with ≥ 5 explicit pass items** (this is what `completeness-auditor` will check — verbal completion does not count).
- The next skill is `result-report-generator` or `robustness-checker` if code is usable.

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

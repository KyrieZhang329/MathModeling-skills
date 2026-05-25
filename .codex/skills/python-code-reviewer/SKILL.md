---
name: python-code-reviewer
description: Review, debug, and verify Python modeling code against the validated method plan and expected artifacts.
license: MIT
---

# Purpose

Review Python modeling code after scripts have been generated and routed by `code-reviewer`.

This skill checks whether `.py` scripts are runnable, consistent with the validated method plan, and able to produce traceable result and figure artifacts.

This skill may make minimal code fixes when needed, but it must not change the modeling route or silently alter assumptions.

This skill does not select models, clean raw data, invent results, write paper sections, or approve final submission.

# When to use

Use this skill:

- After `code-reviewer` routes Python scripts to `python-code-reviewer`.
- When `implementation.target = python`.
- When generated scripts are `.py` files.
- Before using Python-generated outputs for robustness checks, figures, or paper writing.
- When Python is used for official modeling, preprocessing, prototyping, validation, or auxiliary computation.

# Preconditions

The following should already exist or be provided:

- A validated method plan.
- `implementation.target = python`.
- Python scripts under `workspace/code/python/`.
- Cleaned data under `workspace/data/data_clean/`, if data is required.
- Data audit report and field mapping if data is used.
- Expected result paths under `workspace/results/`.
- Expected figure paths under `workspace/figures/`.
- Run instructions or intended script order.
- Runtime notes, especially dependency and portability requirements.

If generated scripts are missing, hand back to `python-model-code-generator`.

If cleaned data is missing, hand back to `data-auditor-cleaner`.

If `implementation.target` is not `python`, hand back to `code-reviewer`.

# Inputs

Use or request:

- `workspace/problem/method-selector/method_plan.json`
- Python scripts under `workspace/code/python/`
- cleaned data under `workspace/data/data_clean/`
- `workspace/data/data_report.md`, if available
- field mapping from `data-auditor-cleaner`
- expected output files
- expected figure files
- runtime notes such as:
  - `minimal-dependencies`
  - `portable-artifacts`
  - `avoid-notebook-only-code`
  - `save-csv-json-png`
  - `fixed-random-seed`
- error messages, logs, tracebacks, screenshots, or failed outputs if available

# Workflow

1. Confirm Python review target.
   - Confirm `implementation.target = python`.
   - Confirm scripts are `.py` files.
   - Confirm scripts are under `workspace/code/python/` or clearly documented otherwise.
   - Preserve dependency and portability runtime notes.

2. Map scripts to the method plan.
   - Identify which subquestion each script supports.
   - Identify whether each script implements a baseline, main model, improved model, helper procedure, or preprocessing stage.
   - Check that every required baseline and main model script exists.
   - Flag scripts that cannot be mapped to the method plan.

3. Check Python script structure.
   - Prefer plain `.py` scripts.
   - Reject notebook-only workflows when a runnable script is required.
   - Check that scripts can run from the project root.
   - Check imports and dependencies.
   - Check that helper functions are clear and not over-abstracted.
   - Check that execution order is documented.
   - Check that `run_all.py` exists only if useful.

4. Check paths and artifacts.
   - Confirm cleaned data is read from `workspace/data/data_clean/`.
   - Confirm raw data under `workspace/data/data_raw/` is not modified.
   - Confirm results are saved under `workspace/results/`.
   - Confirm figures are saved under `workspace/figures/`.
   - Check that outputs are saved to portable formats such as `.csv`, `.json`, and `.png`.
   - Flag scripts that only print or display results without saving them.

5. Check data-field consistency.
   - Verify `pandas.read_csv`, `read_excel`, `numpy.load`, or equivalent input operations use expected files.
   - Verify DataFrame column names match the data audit field mapping.
   - Verify units and indicator directions are consistent with the method plan.
   - Verify missing values are handled or explicitly blocked.
   - Verify rows and columns are not silently dropped.
   - Verify feature-target separation for prediction tasks.

6. Check numerical correctness risks.
   - Check array and DataFrame shapes.
   - Check matrix and vector dimensions.
   - Check broadcasting behavior.
   - Check index alignment in pandas operations.
   - Check division by zero.
   - Check `NaN`, `inf`, or invalid numeric operations.
   - Check objective functions, constraints, weights, and formulas against the method plan.
   - Check convergence or stopping criteria where relevant.

6.5. **Constraint-direction sanity check (P2 — list for human review, do NOT auto-correct).**
    Inequality direction errors (writing `allocated ≥ capacity` when it should be `≤`) are semantic, not syntactic — the agent cannot verify physical correctness. But it can **surface every constraint for the user to scan**.

    For every constraint in the code:
    - Extract the line number, the inequality direction (`≤` / `≥` / `==`), the left-hand side variable name, and the right-hand side expression.
    - Format as a compact table in the review file:

    | File:line | Direction | LHS | RHS | Expected physical meaning |
    |---|---|---|---|---|
    | `code/Q3/q3_main.py:42` | `≤` | `total_allocated[i]` | `capacity[i]` | allocated ≤ capacity (resource upper bound) |
    | `code/Q3/q3_main.py:58` | `≥` | `allocated[i]` | `demand_min[i]` | allocated ≥ minimum demand (demand floor) |
    | `code/Q3/q3_main.py:72` | `==` | `sum(allocated)` | `available` | total consumed = total available (conservation) |

    - Do NOT change inequality direction — the reviewer cannot know the modeler's intent. But the reviewer MUST:
      - Flag constraints whose direction looks counterintuitive (e.g., `allocated ≤ demand_min` is surprising — should it be `≥`?).
      - Ask the user to scan the "Expected physical meaning" column and confirm.
    - This is a **safety net for the user**, not a correctness guarantee. Document this table in `code/Qx/reviews/qx_python_review.md` under a `## Constraint Direction Review` section.

7. Check prediction and machine learning risks when applicable.
   - Check train-test split.
   - Check cross-validation setup when used.
   - Check feature leakage.
   - Check label leakage.
   - Check scaling is fit only on training data when needed.
   - Check random states are fixed.
   - Check metrics match the method plan and task type.

8. Check randomness and reproducibility.
   - If randomness is used, verify fixed seeds are set.
   - Verify stochastic outputs are saved.
   - Verify repeated trials are documented if required by the method plan.

9. Check dependency and portability risks.
   - Prefer common scientific Python dependencies.
   - Flag unnecessary heavy dependencies.
   - Flag notebook-only assumptions.
   - Flag local environment assumptions.
   - Flag Python-only binary artifacts if they are the only outputs.

10. Make minimal fixes if needed.
    - Fix syntax errors, import errors, path errors, output saving errors, simple shape mismatches, missing directory creation, and obvious column-name mismatches.
    - Keep changes surgical and traceable.
    - Do not refactor broadly.
    - Do not change the mathematical model.

11. **Produce a review report — MANDATORY substantive file on disk.**
    - State reviewed scripts.
    - **List ≥ 5 explicit pass items** (concrete checks that ran and passed, each citing the file/line and what was verified). Generic statements like "code is correct" or "all checks passed" do NOT count.
    - List failed checks with file:line and suggested repair.
    - State fixed issues (what this skill changed surgically).
    - State unresolved risks.
    - State run instructions.
    - State expected outputs.
    - Recommend the next skill.
    - **If fewer than 5 explicit pass items can be listed**, the review is not substantive — return `status: not_run` and report which checks could not be performed (missing data, missing method plan, etc.). Do NOT pad the list with generic statements.

# Outputs

Produce reviewed Python code artifacts and a **mandatory review file**:

- corrected `.py` scripts under `code/Qx/` (matching CLAUDE.md workspace convention).
- **`code/Qx/reviews/qx_python_review.md`** — REQUIRED. This is the artifact `completeness-auditor` will check for existence and pass-item count. Create the `reviews/` directory if missing.
- updated run instructions
- fixed issue list
- remaining dependency or portability risks
- expected result paths
- expected figure paths
- recommended next skill

(Legacy path `workspace/code/code-review/python_review_summary.md` is deprecated. New runs must write to `code/Qx/reviews/qx_python_review.md`.)

# Output format

The mandatory `code/Qx/reviews/qx_python_review.md` file MUST follow this template:

```markdown
# Qx Python Code Review

> **Status**: passed / passed_with_warnings / failed
> **Reviewer**: python-code-reviewer
> **Date**: [timestamp]
> **Scripts reviewed**: [list of .py paths]

## Pass Items (≥ 5 REQUIRED — concrete, file:line cited)

1. ✅ `code/Q1/q1_main.py:14` reads cleaned data from `workspace/data_clean/clean_data.csv` (matches data audit field mapping).
2. ✅ `code/Q1/q1_main.py:42` uses `K=30` matching `methods/Q1/q1_final_method_explanation.md`.
3. ✅ `code/Q1/q1_main.py:67` writes ranking to `results/Q1/experiments/round1/tables/ranking.csv` — output saved, not printed.
4. ✅ `code/Q1/q1_baseline.py` implements the designated baseline (equal-weight scoring) per method plan.
5. ✅ All scripts set `SEED = 2026` for reproducibility (`q1_main.py:8`, `q1_baseline.py:6`).
6. ✅ No raw data file under `workspace/data_raw/` is opened in write mode.
7. ✅ ...

## Failed / Repaired Items

| # | File:line | Issue | Action taken | Status |
|---|-----------|-------|--------------|--------|
| 1 | code/Q1/q1_main.py:23 | absolute path `/Users/zhnnky/...` | replaced with `DATA_DIR / 'clean_data.csv'` | fixed |
| 2 | code/Q2/q2_main.py:88 | scaler fit on full dataset before train-test split (leakage) | NOT fixed — requires method change | blocked → method-selector |

## Remaining Risks

- [risk 1 with file:line and mitigation suggestion]

## Run Instructions

```
python code/Q1/q1_baseline.py
python code/Q1/q1_main.py
```

## Expected Outputs

- `results/Q1/experiments/round1/tables/ranking.csv`
- `results/Q1/experiments/round1/figures/q1_ranking.png`
- `results/Q1/experiments/round1/run_summary.json`

## Recommended Next Skill

- `result-report-generator` (if scripts ran and produced outputs)
- OR `robustness-checker` (if results already reported)
```

JSON summary is acceptable for handoff but the markdown file at `code/Qx/reviews/qx_python_review.md` is still required.

# Python review checklist

Check at least the following.

## Script structure

- script is a plain `.py` file
- script is not notebook-only
- file name is clear and maps to a subquestion
- script can run from the project root
- helper functions are clear and limited
- execution order is documented
- `run_all.py` exists only if useful

## Method consistency

- script maps to a method-plan item
- baseline script exists when required
- main model implements the selected method
- optional improved model is clearly separated
- formulas match the method plan
- variables match the method plan where practical
- objective functions and constraints are consistent
- rejected methods are not accidentally implemented

## Data and DataFrame handling

- cleaned data is used
- raw data is not overwritten
- input calls use expected paths
- DataFrame column names match the data report
- missing values are handled or reported
- units and indicator directions are respected
- no rows or columns are silently dropped
- feature-target separation is explicit where relevant

## Numerical checks

- array dimensions are compatible
- DataFrame shapes are expected
- pandas index alignment is intentional
- broadcasting is intentional
- no avoidable division by zero
- no uncontrolled `NaN` or `inf`
- weights sum or normalize correctly when required
- constraints are represented correctly
- convergence or iteration limits are defined where needed

## Prediction and ML checks

- train-test split is valid
- validation setup matches the method plan
- no feature leakage
- no label leakage
- scalers or preprocessing are fit only on training data where applicable
- metrics match the task type
- random states are fixed where supported

## Reproducibility

- Python random seed is set when randomness exists
- NumPy random seed is set when randomness exists
- model `random_state` is set where supported
- seed value is documented
- stochastic outputs are saved
- repeated simulations are summarized if required

## Paths and outputs

- relative paths are used where practical
- no hard-coded local absolute paths
- output directories are created if needed
- result tables are saved under `workspace/results/`
- figures are saved under `workspace/figures/`
- `.json` summaries are saved when useful
- outputs are not limited to console printouts

## Dependencies

- dependencies are common and justified
- unnecessary heavy packages are avoided
- dependency risks are listed
- notebook-only assumptions are avoided
- Python-only binary artifacts are not the only saved outputs

# Fixing rules

- Make the smallest necessary change.
- Preserve the approved modeling route.
- Preserve existing code style where practical.
- Fix only issues related to review goals.
- Remove only unused imports, variables, or code created by the fix.
- Add directory creation only when needed for saving outputs.
- Add result saving when missing.
- Add fixed random seeds when randomness exists and no seed is set.
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
- Do not add heavy dependencies.
- Do not ignore dependency or portability notes.
- Do not claim the code is correct if scripts are not runnable or outputs are not traceable.
- Do not declare "completion" without writing `code/Qx/reviews/qx_python_review.md` to disk with ≥ 5 explicit pass items. A verbal "passed" is not a review — `completeness-auditor` will flag it as MISSING.
- Do not pad the pass list with generic statements. If you cannot list 5 concrete checks, report `status: not_run` instead of faking the list.
- Mark remaining risks explicitly.

# Verification

Before handing off, verify:

- `implementation.target = python`.
- Reviewed scripts are `.py` files.
- Reviewed scripts map to the method plan.
- Scripts use cleaned data or approved non-tabular inputs.
- No raw data file is overwritten.
- Baseline outputs exist or are explicitly blocked.
- Main model outputs exist or are explicitly blocked.
- Randomness is controlled where relevant.
- Results are saved under `results/Qx/experiments/roundN/` (per CLAUDE.md workspace convention).
- Figures are saved under `results/Qx/experiments/roundN/figures/`.
- Dependency and portability risks are listed.
- Fixed issues are documented.
- Remaining risks are documented.
- **`code/Qx/reviews/qx_python_review.md` exists on disk with ≥ 5 explicit pass items** (this is what `completeness-auditor` will check — verbal completion does not count).
- The next skill is `result-report-generator` or `robustness-checker` if code is usable.

# Failure modes

Stop and report a blocker if:

- No `.py` script is available to review.
- The method plan is missing.
- `implementation.target` is not `python`.
- Required cleaned data is unavailable.
- Required fields are missing.
- Runtime errors cannot be fixed without changing the model.
- The code depends on unavailable or unjustified heavy dependencies.
- The script output cannot be linked to any subquestion.
- The script writes to `workspace/data/data_raw/`.
- The script cannot produce required result artifacts.
- Fixing the script would require inventing data, labels, parameters, or results.
- Contest rules disallow Python for the intended stage and no mixed-language workflow is approved.

# Stop conditions

This skill must stop instead of guessing when:

- A fix would change the mathematical meaning of the model.
- A fix requires inventing data, labels, parameters, metrics, outputs, or results.
- A script cannot be mapped to the method plan.
- The data schema is too ambiguous to repair safely.
- The script language or implementation target conflicts with the route.
- Python is not approved for the intended contest computation stage.
- Continuing would hide a method, data, dependency, or portability problem.

When stopping, output:

- blocker
- why it matters
- affected script or subquestion
- safe partial fixes if any
- missing information needed
- recommended repair skill
- next action

# Handoff

If reviewed Python code is usable, hand off to:

```
robustness-checker
```

The handoff should include:

- reviewed `.py` script paths
- run instructions
- result output paths
- figure output paths
- fixed issue list
- remaining risk list
- dependency and portability notes
- validation outputs if available

If blocked by missing or wrong generated scripts, hand back to:

```
python-model-code-generator
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

- `implementation.target = python`
- `workspace/code/python/Q1/q1_main.py` uses an absolute local path
- script displays ranking but does not save it

Output:

```json
{
  "python_code_review_summary": {
    "implementation_target": "python",
    "status": "passed_with_warnings",
    "reviewed_scripts": [
      "workspace/code/python/Q1/q1_main.py"
    ],
    "fixed_issues": [
      {
        "type": "path_error",
        "change": "Replaced absolute local input path with DATA_DIR / 'clean_data.csv'."
      },
      {
        "type": "missing_result_artifact",
        "change": "Added DataFrame.to_csv call to save ranking results under workspace/results/."
      }
    ],
    "remaining_risks": [
      "Column names should be rechecked if the data audit report changes."
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
- method plan says `implementation.target = matlab`

Output:

```json
{
  "blocked_items": [
    "python-code-reviewer was invoked, but implementation.target is matlab."
  ],
  "recommended_next_skill": "code-reviewer",
  "recommended_next_action": "Route the scripts through code-reviewer again and use the MATLAB review branch."
}
```

## Example 3: Blocked by feature leakage

Input state:

- `implementation.target = python`
- prediction script scales the full dataset before train-test split
- method plan requires train-test validation

Output:

```json
{
  "blocked_items": [
    "Feature scaling is fitted on the full dataset before train-test split, causing leakage."
  ],
  "affected_script": "workspace/code/python/Q2/q2_main.py",
  "recommended_next_skill": "python-code-reviewer",
  "recommended_next_action": "Move scaler fitting after the train-test split and fit only on the training set."
}
```

## Example 4: Blocked by heavy dependency

Input state:

- `implementation.target = python`
- script imports a heavy package not justified in the method plan
- a simpler implementation is feasible

Output:

```json
{
  "blocked_items": [
    "The script depends on an unjustified heavy package that is not required by the approved method plan."
  ],
  "affected_script": "workspace/code/python/Q3/q3_main.py",
  "recommended_next_skill": "python-code-reviewer",
  "recommended_next_action": "Replace the dependency with a simpler numpy / scipy / scikit-learn implementation or update the method plan with a justification."
}
```
